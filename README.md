# Crossly CLI — GitHub Action

Run Crossly from a workflow. The interesting use is treating your inventory as something a repository owns.

```yaml
- uses: crossly-dev/crossly-action@v1
  with:
    token: ${{ secrets.CROSSLY_PAT }}
    args: listings publish --id ${{ github.event.inputs.listing }} --to ebay,poshmark
```

## Nightly export you can diff

```yaml
on:
  schedule: [{ cron: '0 3 * * *' }]

jobs:
  snapshot:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: crossly-dev/crossly-action@v1
        id: export
        with:
          token: ${{ secrets.CROSSLY_PAT }}
          args: inventory export
          json: true
      - run: echo '${{ steps.export.outputs.result }}' > inventory.json
      - run: git diff --stat inventory.json
```

A diff of yesterday's inventory against today's is the cheapest anomaly detector you will ever build.

## Inputs

| input | | |
|---|---|---|
| `args` | **required** | arguments to `crossly` |
| `token` | **required** | a PAT, **from a secret** |
| `version` | `latest` | which `@crossly/cli` to install |
| `json` | `false` | append `--json` so a later step can parse it |

## Exit codes are surfaced, not collapsed

| code | meaning | what to do |
|---|---|---|
| `0` | ok | |
| `2` | usage error | fix the `args` input |
| `4` | token rejected | rotate `CROSSLY_PAT` |
| `5` | missing scope | re-mint the token with the scope the command needs |

A workflow that reports all four as "failed" makes you guess which one it was.

## The token

Pass it from a secret. The action reads it from the **environment**, never from the command line — a token in `argv` is readable by every other process on the runner via `/proc`, and shows up in the log if anything runs `set -x`.

## License

MIT
