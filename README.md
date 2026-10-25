# registry

**Which version of which remote runs on which stand.** One file per remote per stand; every change is a commit
(audit trail, rollback = deploy the previous version). The host is never rebuilt for a remote release.

- `stands.json` — the stands and **what they run**:

  | Stand | Rule | Who uses it |
  |---|---|---|
  | `dev3` | `"snapshot": true` — any branch, overwritten in place | developers |
  | `dev2` | `"branch": "latest-release"` — the newest `release/X.Y` (branch builds and tags) | QA |
  | `dev1` | `"branch": "release/1.5"` — the pilot line, pinned by release managers | team testers |
  | `tst` | `"promoteFrom": "dev1"` — never deployed to | business testers |

  Stands live at `https://<stand>.poc.ashirokov.dev`. `nexusUrl` is the Nexus `/repository` base
  (`https://nexus.poc.ashirokov.dev/repository`): releases in `portal-releases` (no redeploy), snapshots in
  `portal-snapshots/<stand>/<remote>/`, what a stand serves (host build, `/config.json`) in `portal-stands/<stand>/`.

- `<stand>/<remote>.json` — `version`, `manifest`, `bridgeManifest` (written by deploys), `basePath`, `scopes`,
  `title` (host/platform team). On `dev3` the entry is constant (`"version": "snapshot"`): a deploy overwrites the
  snapshot, not the entry.

## How versions get here

Nothing is automatic: a person presses a button and picks the stand.

| Action | Who | Result |
|---|---|---|
| remote repo: **Deploy**, `dev3`, any branch | developer | snapshot overwritten; the registry changes only on the first deploy |
| remote repo: **Deploy**, `dev2`/`dev1`, `release/1.5` | developer | `1.5.3-rc.N` published, commit to `dev2/` (or `dev1/`) |
| remote repo: **Deploy**, `dev2`/`dev1`, tag `v1.5.2` | developer | `1.5.2` (published once, reused later), commit |
| remote repo: **Deploy** of an older tag | anyone | rollback: same artifact, no rebuild |
| registry: **Promote** workflow / `pnpm promote:tst` | release manager | the versions of `dev1` → `tst/`, commit |
| `portal-cli stand switch dev1 --branch release/1.6 --pr` | release manager | one PR: `stands.json` + the 1.6 versions on `dev1` |

`portal-cli deploy` refuses before building: `develop` or a feature branch to `dev2`/`dev1`, another line than the
stand runs (`dev1` = `release/1.5`, so `release/1.4` is refused with the `stand switch` hint), anything to `tst`.

Every write goes through the gate against the host **deployed on that stand** (`compat.json`). Every push to main
(and every Promote) runs **Config deploy**: `portal-cli registry build --upload` → each stand's `/config.json`, live
on the next page load. Deploy commits come from the pipeline
bot straight to main (the button press is the approval); hand edits (`basePath`, `scopes`, `stands.json`) go through
pull requests with CODEOWNERS. Local development is never described here: developers list the remotes they run in
the host's `.env.development.local`.

## Commands

```sh
pnpm install
pnpm validate          # offline check (CI): entries, stands.json rules, release lines
pnpm status            # versions dashboard; dev3 shows the built snapshot
pnpm check             # gate: every entry against its stand's host compat.json
pnpm promote:tst       # dev1 → tst, written locally (the workflow commits it)
```
