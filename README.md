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

The `golang` hooks require a Go toolchain on the machine running prek; `prek install-hooks` will `go install github.com/suzuki-shunsuke/pinact/v3/cmd/pinact@<pinned-version>` into the hook env on first use.

## Why this repo exists

Upstream pinact does not yet ship a `.pre-commit-hooks.yaml`. [suzuki-shunsuke/pinact#971](https://github.com/suzuki-shunsuke/pinact/pull/971) proposes adding one. Once it lands, this wrapper becomes obsolete — consumers can repoint `repo =` at `https://github.com/suzuki-shunsuke/pinact` and delete this repo from their config.

## Maintenance

`.github/workflows/bump-pinact.yml` runs every Monday at 06:00 UTC. It checks `suzuki-shunsuke/pinact` for a newer release, rewrites the pinned version in `.pre-commit-hooks.yaml`, runs prek against a fixture to verify the new release works end-to-end, then pushes to `main`, tags `v<pinact-version>`, and cuts a matching GitHub release. Bad releases never reach `main`.

Manual trigger: `gh workflow run bump-pinact.yml --repo ref-app/prek-pinact`.

## License

MIT — see [LICENSE](LICENSE).
