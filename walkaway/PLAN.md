# WalkAwayDev plan

Goal: hand omp one ready ticket on the MacBook, walk away, and come back to a green PR that you merge. Everything else is underground as upstream builds it.

## Hardware

All three machines are on the same home LAN. The MacBook and the `GX10` are on the tailnet. The addresses live in `~/.config/walkaway/walkaway.conf` on the mac, not in this repo.

| machine | job in underground | what it needs |
|---|---|---|
| MacBook Pro, `M3 Pro`, 18 GB, 512 GB | Runs omp as the agent user, the gateway, the gh-broker, the watch, and the nightly backup job. | Apple silicon (the pin is `omp-darwin-arm64`), Tailscale, `gh`, `jq`, `wakeonlan`. The model runs elsewhere, so 18 GB is enough [inference]. |
| ASUS Ascent `GX10`, on 24/7 | The engine: serves the qwen model on an OpenAI-compatible API that wants an API key. The auth gateway on the mac calls it at its Tailscale address as omp's `lm-studio` provider, and the auth broker holds the key (`docs/adr/0005-model-auth-gateway.md`, "Amendment 2026-11-01: the inference engine behind the gateway"). After a restart it loads the model again within about 3 minutes (owner, 2026-11-10). | Its Tailscale `<ip>:<port>` in `engine=`, the model id from `/v1/models` in `engine_model=`, and the key in the auth broker: `ug-auth key LM_STUDIO_API_KEY`. |
| Ryzen PC, `7600X3D`, 32 GB, Windows 11 Pro | The backup target: a restic repo over sftp, woken for the nightly run and put back to sleep (`docs/SPEC.md`, "Backup"). The backup makes the restic repo on its first run (`init`, `host/bin/ug-backup`). | The steps in "Backup target on Windows" below. |

## Backup target on Windows

On the Ryzen PC, in PowerShell as an administrator. A standard user of its own holds the backup, so the mac's key opens nothing else on the PC.

1. Turn on the ssh server: `Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0`, then `Set-Service -Name sshd -StartupType Automatic`, then `Start-Service sshd`.
2. Make the backup user, a standard user with a password: `New-LocalUser -Name walkaway-backup`. Keep the password in your password manager; the key below replaces it for the backup.
3. Note the PC's LAN IPv4 and the wired adapter's MAC address: `Get-NetIPAddress -AddressFamily IPv4` and `Get-NetAdapter`. Write the MAC with colons in `backup_wake=`.
4. Wake-on-LAN, on the wired adapter: turn it on in the BIOS. In Device Manager, under the adapter's Power Management, allow it to wake the computer, with a magic packet only; under Advanced, turn on Wake on Magic Packet. To make `SetSuspendState` sleep and not hibernate: `powercfg /hibernate off`.

On the MacBook, as yourself:

5. Make a key if you have none: `ssh-keygen -t ed25519`.
6. Log in once with the password, which makes the user's home and records the PC's host key: `ssh walkaway-backup@<ryzen-lan-ip> mkdir .ssh`.
7. Give the user your key: `scp ~/.ssh/id_ed25519.pub walkaway-backup@<ryzen-lan-ip>:.ssh/authorized_keys`.
8. Check the key login, which is what the backup uses: `ssh -o BatchMode=yes walkaway-backup@<ryzen-lan-ip> echo ok`.
9. Check sleep and wake: run the `backup_sleep=` command, wait until the PC sleeps, run the `backup_wake=` command, and repeat step 8. The backup waits up to 5 minutes for the PC to answer (`backup_target_wake`, `host/lib/backup-target.bash`).

Not run yet: whether a standard user may sleep the PC from an ssh login [inference: Windows grants the shut down right to Users on a workstation]. If step 9 fails there, make `walkaway-backup` an administrator only after other fixes fail, because an administrator's keys live in `C:\ProgramData\ssh\administrators_authorized_keys`, not in the user's home.

## Done

- Imported underground with its full history; `upstream` points at it.
- `.github/CODEOWNERS` names @umidscodes, because setup stops unless it names the `gh` login (`ug_check_main_rule`, `host/lib/ug.bash`).
- `walkaway/init` folds steps 1 to 6 of `docs/SPEC.md`, "First setup", into one command: it installs `gh`, `jq`, and `wakeonlan` when missing, and offers to add the `main` ruleset that setup checks. A Linux smoke test with stubbed `gh`, `tailscale`, `curl`, and `ssh` ran it from an empty home twice: the ruleset it posts passes `ug_check_main_rule`, each file lands once with mode 600, and the second run changes nothing. It has not run on a mac.
- CI lints `walkaway/init` with the other shell scripts.

## Next

1. **Get `main` green.** Merge this branch, then check that both CI jobs pass on WalkAwayDev. shellcheck 0.9.0 flags SC2120 on `ug_services` in `host/lib/ug.bash` at upstream's current head; if the `checks` job fails on it, fix it in its own PR.
2. **Prepare the other machines.**
   - Ryzen PC: "Backup target on Windows" above.
   - `GX10`: the model id from `curl -H "Authorization: Bearer <key>" http://<engine>/v1/models`.
   - Healthchecks.io: one check with a 5 minute period and a grace time longer than the mac's night sleep (`docs/adr/0023-failures-reach-the-owner.md`, "Decision").
3. **Run `walkaway/init` on the mac** (`README.md`, "Start"), then `ug-auth key LM_STUDIO_API_KEY` with the `GX10` key.
4. **Live run before trust** in a throwaway public repo (`docs/SPEC.md`, "Onboarding checks"): one ticket from branch to merge, one revert, and one PR with a planted defect.
5. **Spec to tickets.** Write the spec together with the agent (or with Claude), split it into ready tickets with no design decision left, then hand omp one ticket per session. A skill for this split can live in the agent config repo.

## Open

- The watch checks the gateway, not the engine behind it: if the `GX10` stays down, a session fails and nothing else tells you (`docs/PRODUCT-BRIEF.md`, H3). Its auto restart covers a short outage. A candidate ticket: the watch asks the engine through the gateway.
- `walkaway/agent-config/models.yml.template` points omp's `lm-studio` provider at the gateway with `transport: pi-native`, and `config.yml.template` makes the engine model the default. Both follow omp's docs (`docs/models.md` of omp 18.18.1, read 2026-11-10); neither ran on the pinned omp yet.
- Repos stay public: with no network wall, a session can send anything it reads anywhere (`docs/THREAT-MODEL.md`, U14). A private repo needs that question answered first.
- upstream has no license file. Ask Israfil before WalkAwayDev goes further than our own use.
