# registry

Simulates the registry repository (one per organisation): **which version of which remote runs on which stand**.

- `stands.json` — stand definitions (URL, Nexus base).
- `<stand>/<remote>.json` — one file per remote per stand (no merge conflicts between teams).
  - `version`, `manifest`, `bridgeManifest` — owned by the remote team (CODEOWNERS), written by `portal-cli release`.
  - `basePath`, `scopes` — owned by the host/platform team (route and token allowlist need their approval).

Flow: `portal-cli release --stand uat` = verified change of `<stand>/<remote>.json` → in CI it opens a PR; branch
policies require the gate (`portal-cli registry check`) and the owner's approval; merge triggers the config deploy
(`portal-cli registry build --stand uat` → the stand's `/config.json`). Rollback = revert the PR. The host is never
rebuilt for a remote release.

Only deployed stands live here; `uat`/`prod` entries are created by releases (empty until the first release).
Local development is never described in the registry: a developer lists the remotes they run in the host's
`.env.development.local` (works for remotes that are not registered anywhere yet).

## Commands

```sh
pnpm install
pnpm validate          # offline check (CI)
pnpm status            # versions dashboard (needs the PoC infra running)
pnpm check:uat         # gate: every uat entry against the uat host's compat.json
pnpm deploy:uat        # config deploy after a merge → platform/.infra/stands/uat/config.json
```

Releases arrive as pull requests opened by `portal-cli release --stand <stand> --pr` from a remote repository.
