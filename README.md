# tenon-action

Runs [Tenon](https://github.com/shidil/tenon) in GitHub Actions. Tenon maps every API call and
webhook handler in a repository to the API specs named in its `tenon.yaml`, checks each one
against its operation, and *seals* every integration whose calls all pass.

- **Pushes, schedules, manual runs:** seal the repository and keep one dashboard issue (label
  `tenon`) listing every integration as sealed, unsealed or broken, with the findings behind
  it. A seal breaks when the code or a spec changes and a check fails; the nightly run catches
  spec changes upstream.
- **Pull requests:** check the files the PR changes and annotate findings on the lines it adds
  or edits (`fail-on: error` fails the job on new errors).

Nothing is sent anywhere but the GitHub API of the repository itself: Tenon reads code, never
runs it, and needs no LLM.

```yaml
# .github/workflows/tenon.yml
name: Tenon
on:
  push:
    branches: [main]
  pull_request:
  schedule:
    - cron: "17 3 * * *"
  workflow_dispatch:
permissions:
  contents: read
  issues: write
jobs:
  tenon:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v7
        with:
          fetch-depth: 0 # pull requests are compared with their base
      - uses: actions/setup-go@v7 # Go repositories: the type-aware analysis uses the toolchain
        with:
          go-version-file: go.mod
      - uses: shidil/tenon-action@v0
```

Inputs: `mode` (auto, seal, check), `fail-on` (never, error), `dashboard` (true),
`dashboard-branch` (default the default branch), `config` (default `tenon.yaml`), `token`
(default `github.token`), `version`.

The Tenon database is kept in the Actions cache between runs, and the seal state is also kept in
the dashboard issue, so an evicted cache loses nothing but the detail of old spec versions.
Linux runners (x64 and arm64).
