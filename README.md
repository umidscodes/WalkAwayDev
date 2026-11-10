# WalkAwayDev

Hand omp a ready ticket on the MacBook, walk away, and come back to a green PR.

WalkAwayDev is [underground](https://github.com/IsrafilEMIN/underground-public) by Israfil Emin, set up for our hardware, plus one setup command. underground runs omp as its own hidden macOS user that holds no GitHub or model credential, so an unattended run cannot touch your files, push to `main`, or stay broken without telling you. `docs/PRODUCT-BRIEF.md` holds that design; `docs/SPEC.md` says what runs.

Our layer is small on purpose, so fixes from upstream merge cleanly:

- `walkaway/init`: the one command that sets up the mac, first time and after.
- `walkaway/walkaway.conf.example`: the one file you fill in.
- `walkaway/agent-config/`: the agent config that `init` puts in its own git repo.
- `walkaway/PLAN.md`: the hardware map, the plan, and what is still open.
- `.github/CODEOWNERS`: names @umidscodes, which setup requires.

## Start

On the MacBook, as yourself:

1. Clone this repo into your home and check out `main`.
2. Run `walkaway/init`. It writes `~/.config/walkaway/walkaway.conf`; fill it in.
3. Run `walkaway/init` again. It installs `gh` and `jq` if missing, adds the `main` ruleset after you say yes, writes the data folder files, makes the agent config repo, and runs the GitHub App wizard (first time) or setup (later).
4. Store the GX10 key: `ug-auth key LM_STUDIO_API_KEY`.
5. Type `omp`.

After a change on `main`, run `walkaway/init` again: it skips what is in place and runs setup.

## Upstream

`upstream` is IsrafilEMIN/underground-public. To take its fixes: `git fetch upstream`, merge `upstream/main` into a branch, and open a PR here. That PR changes code that runs on the host, so read it before you merge.
