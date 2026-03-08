# Local OpenClaw SOP
This guide is for operating a locally customized OpenClaw checkout while continuing to receive upstream updates from the community.
## Repository model
Treat the setup as three separate layers:
1. Upstream application code
   - Repo: this checkout
   - `upstream` = official OpenClaw repository
   - `origin` = your fork
2. Local runtime state
   - `~/.openclaw/openclaw.json`
   - systemd user service config
   - reverse proxy / DNS / TLS config
3. Private agent workspace
   - `~/.openclaw/workspace`
   - prompt files, memory, instructions, notes
Do not treat local runtime state or workspace memory as part of upstream app code.
## Current git remote layout
Expected remotes in this checkout:
```bash
origin   https://github.com/danielmigizi/openclaw.git
upstream https://github.com/openclaw/openclaw.git
```
Verify:
```bash
git remote -v
```
## Branch workflow
Use this pattern:
- `main`: your fork's integration branch
- short-lived feature branches for local changes
Examples:
```bash
git checkout main
git pull --rebase origin main
git checkout -b chore/local-ops-docs
```
When done:
```bash
git push -u origin chore/local-ops-docs
```
Open a PR in your fork if you want review before merging to `main`.
## Pulling updates from the community
Refresh official changes:
```bash
git fetch upstream
git checkout main
git rebase upstream/main
git push origin main
```
If you have long-lived local branches:
```bash
git checkout <your-branch>
git rebase main
```
If rebase becomes painful, use merge instead:
```bash
git checkout main
git merge upstream/main
```
## What belongs where
### Put in the repo
- code changes
- deployment helper scripts
- local SOPs / runbooks
- infrastructure templates
### Keep outside the repo
- gateway token and secrets
- active OpenClaw config values
- browser/device-specific state
- workspace memory and operator notes you do not want in git
Recommended backup strategy:
- public fork for code: `https://github.com/danielmigizi/openclaw`
- private workspace repo: `https://github.com/danielmigizi/openclaw-workspace`
- optional private repo for ops/infrastructure if you want deployment-specific runbooks and server config tracked separately
## Daily operations
Start or restart gateway service:
```bash
systemctl --user restart openclaw-gateway
systemctl --user status openclaw-gateway --no-pager
```
Check app/runtime status:
```bash
openclaw status
openclaw status --all
openclaw gateway status
openclaw health --json
```
Open dashboard:
```bash
openclaw dashboard
```
Useful logs:
```bash
journalctl --user -u openclaw-gateway -n 200 --no-pager
journalctl --user -u openclaw-gateway -f
```
## Dashboard auth and connection recovery
Symptoms:
- `Version n/a`
- `Health Offline`
- `gateway token mismatch`
- `gateway token missing`
- `too many failed authentication attempts`
Recovery flow:
1. Close all dashboard tabs.
2. Clear site data for the dashboard origin.
3. Wait a minute or two if you triggered auth rate limiting.
4. Reopen the dashboard in a fresh tab or private window.
5. If prompted, use the gateway token from `~/.openclaw/openclaw.json` at `gateway.auth.token`.
6. Reconnect once; avoid repeated retries with the wrong token.
Relevant config keys:
- `gateway.auth.mode`
- `gateway.auth.token`
- `gateway.controlUi.allowedOrigins`
- `gateway.trustedProxies`
## Local config checklist
Current deployment assumptions should remain aligned:
- gateway service managed by user systemd
- dashboard served from the gateway
- reverse proxy / public host must match `gateway.controlUi.allowedOrigins`
- proxying localhost traffic should be covered by `gateway.trustedProxies`
When changing hostnames or TLS setup, re-check:
```bash
openclaw gateway status
journalctl --user -u openclaw-gateway -n 100 --no-pager
```
## Safe customization strategy
Prefer this order:
1. configuration changes
2. service / proxy changes
3. scripts and automation
4. source-code changes only when necessary
This keeps upstream updates easier to absorb.
## Before updating from upstream
Check for local changes first:
```bash
git status --short
git branch --show-current
```
If you have work in progress:
- commit it to a branch, or
- stash it before rebasing
Examples:
```bash
git add -A
git commit -m "WIP: save local changes before upstream sync"
```
or
```bash
git stash push -u -m "pre-upstream-sync"
```
## Suggested change workflow
For each local improvement:
1. branch from `main`
2. make the change
3. test locally
4. commit to your fork
5. merge back to `main`
6. periodically rebase `main` onto `upstream/main`
## If you want to contribute back upstream
Keep upstreamable changes isolated from local-only operations changes.
Good candidates for upstream PRs:
- generic bug fixes
- docs improvements useful to all users
- safer defaults
- cleaner setup or auth UX
Keep local-only items in your fork:
- server-specific hostnames
- local proxy assumptions
- personal workflow docs
- private ops notes
