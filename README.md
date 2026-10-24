# registry

**Which version of which remote runs on which stand.** One file per remote per stand, every change is a pull request
(audit, approval, rollback = revert). The host is never rebuilt for a remote release.

- `stands.json` — the stands and **where their versions come from**:

  | Stand | Source | Who uses it |
  |---|---|---|
  | `dev3` | `"branch": "develop"` — every merge to develop | developers |
  | `dev2` | `"branch": "latest-release"` — the newest `release/X.Y` | QA |
  | `dev1` | `"branch": "release/1.4"` — the pilot line, pinned by release managers | team testers |
  | `tst` | `"promoteFrom": "dev1"` — nothing is built for it | business testers |

- `<stand>/<remote>.json` — `version`, `manifest`, `bridgeManifest` (written by releases), `basePath`, `scopes`,
  `title` (host/platform team).

## How versions get here

| Event | Who | Result |
|---|---|---|
| merge to `develop` | remote CI | `1.6.0-dev.57` → PR to `dev3/` (auto-merge on a green gate) |
| tag `v1.5.0` on `release/1.5` | remote CI | PR to `dev2/` (`release/1.5` is the newest line) |
| tag `v1.4.2` on `release/1.4` | remote CI | PR to `dev1/` (`release/1.4` is the pilot) |
| "Promote" workflow / `pnpm promote:tst` | release manager | the versions of `dev1` → PR to `tst/` |
| `portal-cli stand switch dev1 --branch release/1.5 --pr` | release manager | one PR: `stands.json` + the 1.5 versions on `dev1` |
| something broke | anyone | revert the PR |

CI decides the stands from the version alone: `X.Y.Z-dev.N` was built on develop, `X.Y.Z` on `release/X.Y`
(`portal-cli release --auto`). A tag on a line no stand follows is published but released nowhere.

Every write goes through the gate against the host **deployed on that stand** (`compat.json`); merging a PR runs the
config deploy (`portal-cli registry build` → the stand's `/config.json`). Local development is never described here:
developers list the remotes they run in the host's `.env.development.local`.

## Commands

```sh
pnpm install
pnpm validate          # offline check (CI): entries + stands.json mapping
pnpm status            # versions dashboard (needs the PoC infra running)
pnpm check             # gate: every entry against its stand's host compat.json
pnpm deploy            # config deploy after a merge → platform/.infra/stands/<stand>/config.json
pnpm promote:tst       # dev1 → tst as a pull request
```
