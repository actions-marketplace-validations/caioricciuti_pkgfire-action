# PkgFire Dependency Firewall

A GitHub Action that checks every lockfile in your repository against
[PkgFire](https://pkgfire.org) and fails the job before a malicious or risky
dependency reaches your build. It covers npm, PyPI, Go and Cargo.

The scan reads the lockfiles you already committed. It installs nothing, runs
no package code and never uploads your source.

## Usage

1. Create an API key at <https://pkgfire.org/api-keys>.
2. Add it to the repository as the secret `PKGFIRE_API_KEY`.
3. Add `.github/workflows/pkgfire.yml`:

```yaml
name: Dependency firewall

on:
  pull_request:
  push:
    branches: [main]

jobs:
  pkgfire:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      security-events: write # code scanning upload
    steps:
      - uses: actions/checkout@v4
      - uses: caioricciuti/pkgfire-action@v1
        with:
          api-key: ${{ secrets.PKGFIRE_API_KEY }}
```

Each finding shows up as an annotation on the lockfile line, in the job
summary and in the repository's code scanning alerts.

## Inputs

| Input | Default | |
|---|---|---|
| `api-key` | (required) | Your PkgFire API key, from a secret. |
| `fail-on` | `block` | `block` fails only on a refusal; `warn` fails on warnings too. |
| `working-directory` | `.` | Where to look for lockfiles. Every one below it is checked. |
| `upload-sarif` | `true` | Send findings to code scanning. Set `false` if the repository has no code scanning. |
| `wait` | `false` | When coverage is partial, wait for the background re-check. |
| `api-url` | `https://pkgfire.org` | Only for a self-hosted PkgFire. |

A target that could not be checked always fails the job: an unknown result is
not a safe one.

## Outputs

| Output | |
|---|---|
| `exit-code` | `0` allow, `1` warn, `2` block, `3` a target could not be checked. |
| `sarif-file` | Path to the SARIF report. |

## Not on GitHub?

PkgFire is one binary and works the same in GitLab CI, Jenkins, Buildkite or a
shell script:

```sh
curl -fsSL https://pkgfire.org/install.sh | sh
PKGFIRE_API_KEY=... pkgfire scan --all
```

See <https://pkgfire.org/docs/ci>.

## License

This action (the files in this repository) is MIT licensed. The PkgFire
service and CLI are not covered by this license.
