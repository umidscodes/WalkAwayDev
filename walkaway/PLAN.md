# WalkAwayDev plan

Goal: hand omp one ready ticket on the MacBook, walk away, and come back to a green PR that you merge. Everything else is underground as upstream builds it.

## Hardware

| machine | job in underground | what it needs |
|---|---|---|
| MacBook Pro, `M3 Pro`, 18 GB, 512 GB | Runs omp as the agent user, the gateway, the gh-broker, the watch, and the nightly backup job. | Apple silicon (the pin is `omp-darwin-arm64`), Tailscale, `gh`, `jq`. The model runs elsewhere, so 18 GB is enough [inference]. |
| ASUS Ascent `GX10` | The engine: serves the qwen model over an OpenAI-compatible API. The auth gateway on the mac calls it as omp's `lm-studio` provider (`docs/adr/0005-model-auth-gateway.md`, "Amendment 2026-11-01: the inference engine behind the gateway"). | An `<ip>:<port>` the mac reaches, and the model id from `/v1/models`. |
| Ryzen PC, `7600X3D`, 32 GB | The backup target: a restic repo over sftp, woken for the nightly run and put back to sleep (`docs/SPEC.md`, "Backup"). | An ssh server with your mac's key, a disk path for the repo, Wake-on-LAN. |

## Done on this branch

- Imported underground with its full history; `upstream` points at it.
- `.github/CODEOWNERS` names @umidscodes, because setup stops unless it names the `gh` login (`ug_check_main_rule`, `host/lib/ug.bash`).
- `walkaway/init` folds steps 1 to 6 of `docs/SPEC.md`, "First setup", into one command, and offers to add the `main` ruleset that setup checks. A Linux smoke test with stubbed `gh`, `tailscale`, `curl`, and `ssh` ran it from an empty home twice: the ruleset it posts passes `ug_check_main_rule`, each file lands once with mode 600, and the second run changes nothing. It has not run on a mac.
- CI lints `walkaway/init` with the other shell scripts.

## Next

1. **Get `main` green.** Merge this branch, then check that both CI jobs pass on WalkAwayDev. shellcheck 0.9.0 flags SC2120 on `ug_services` in `host/lib/ug.bash` at upstream's current head; if the `checks` job fails on it, fix it upstream-style in its own PR.
2. **Prepare the other machines.**
   - Ryzen PC: an ssh server, `ssh-copy-id` from the mac, a folder for the restic repo, Wake-on-LAN on in the BIOS and the OS.
   - GX10: note its `<ip>:<port>`, whether it wants an API key, and the model id from `curl http://<ip>:<port>/v1/models`.
   - Healthchecks.io: one check with a 5 minute period and a grace time longer than the mac's night sleep (`docs/adr/0023-failures-reach-the-owner.md`, "Decision").
3. **Run `walkaway/init` on the mac** (`README.md`, "Start").
4. **Live run before trust** in a throwaway public repo (`docs/SPEC.md`, "Onboarding checks"): one ticket from branch to merge, one revert, and one PR with a planted defect.
5. **Spec to tickets.** Write the spec together with the agent (or with Claude), split it into ready tickets with no design decision left, then hand omp one ticket per session. A skill for this split can live in the agent config repo.

## Open

- Which OS runs on the Ryzen PC? It sets `backup_sleep` in `walkaway.conf`.
- Is the `GX10` on the tailnet? Wake-on-LAN needs the mac on the same LAN as the PC; the engine needs only an address the mac reaches.
- `walkaway/agent-config/models.yml.template` points omp's `lm-studio` provider at the gateway with `transport: pi-native`, and `config.yml.template` makes the engine model the default. Both follow omp's docs (`docs/models.md` of omp 18.18.1, read 2026-11-10); neither ran on the pinned omp yet.
- Repos stay public: with no network wall, a session can send anything it reads anywhere (`docs/THREAT-MODEL.md`, U14). A private repo needs that question answered first.
- upstream has no license file. Ask Israfil before WalkAwayDev goes further than our own use.
