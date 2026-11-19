# prek-pinact

Thin [prek](https://github.com/j178/prek) / [pre-commit](https://pre-commit.com) hook repository that surfaces the latest [pinact](https://github.com/suzuki-shunsuke/pinact) release without requiring contributors to install the `pinact` binary locally.

## Usage

In your `prek.toml` (or `.pre-commit-config.yaml`):

```toml
[[repos]]
repo = "https://github.com/ref-app/prek-pinact"
rev = "v<latest>"  # see `gh release list --repo ref-app/prek-pinact`

[[repos.hooks]]
id = "pinact"
```

Available hook ids:

| id              | command              | language | notes                                              |
| --------------- | -------------------- | -------- | -------------------------------------------------- |
| `pinact`        | `pinact run --check` | golang   | Default. Fails if any action ref is not SHA-pinned |
| `pinact-fix`    | `pinact run --fix`   | golang   | Rewrites refs to SHAs in place                     |
| `pinact-update` | `pinact run --update`| golang   | Updates pinned SHAs to newest releases             |
| `pinact-system` | `pinact run`         | system   | Uses a locally installed `pinact` binary           |

The `golang` hooks require a Go toolchain on the machine running prek; `prek install-hooks` will `go install github.com/suzuki-shunsuke/pinact/v<major>/cmd/pinact@<pinned-version>` into the hook env on first use.

## Why this repo exists

Upstream pinact does not ship a `.pre-commit-hooks.yaml`. [suzuki-shunsuke/pinact#971](https://github.com/suzuki-shunsuke/pinact/pull/971) proposed one, but it was closed without a merge. If upstream adds hooks later, this wrapper becomes obsolete. Consumers can then point `repo =` at `https://github.com/suzuki-shunsuke/pinact` and remove this repo from their config.

## Maintenance

`.github/workflows/bump-pinact.yml` runs every Monday at 06:00 UTC. It checks `suzuki-shunsuke/pinact` for a newer release, rewrites the pinned version in `.pre-commit-hooks.yaml`, runs prek against a fixture to verify the new release works end-to-end, then pushes to `main`, tags `v<pinact-version>`, and cuts a matching GitHub release. Bad releases never reach `main`.

Manual trigger: `gh workflow run bump-pinact.yml --repo ref-app/prek-pinact`.

The same workflow runs on every pull request. On a pull request it only runs the fixture check against the PR head and never bumps or releases. Dependabot opens weekly pull requests for the pinned actions. When the check passes, the `refapp-automaton` GitHub App merges them. The app needs the `workflows` permission, and its private key must be stored as both an Actions secret and a Dependabot secret named `REFAPP_AUTOMATON_PRIVATE_KEY`.

If a scheduled run fails, the workflow opens an issue named "Bump pinact failed", or comments on the open one.

## License

MIT — see [LICENSE](LICENSE).
