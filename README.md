# Kestrova Solidity Scan (GitHub Action)

A free Slither security preview of your Solidity contracts on every pull request. It catches known vulnerability
patterns, such as reentrancy, unchecked low-level calls, `tx.origin` authorization and arbitrary ETH sends, before they
are merged.

- **Free.** No API key, no account, no `solc` install. The action never pays for anything.
- **Changed files only.** On a pull request it scans the Solidity files that changed (plus the contracts that import
  them), compares with the base branch, and reports only **new** findings.
- **Advisory by default.** Set `fail-on: high` to block pull requests that add a new High finding.
- **Never breaks your build because of us.** If the service is unavailable or rate-limited, the step only warns.

Automated static analysis, not an audit. Zero findings does not mean a contract is safe.

## Usage

`.github/workflows/kestrova.yml`:

```yaml
name: Kestrova
on:
  pull_request:
    paths: ["**/*.sol", "foundry.toml", "remappings.txt"]
permissions:
  contents: read
jobs:
  scan:
    runs-on: ubuntu-latest
    timeout-minutes: 10
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0            # the base branch is needed for the comparison
          persist-credentials: false
          submodules: recursive     # Foundry lib/ dependencies
      - uses: actions/setup-node@v4
        with:
          node-version: 22
      # Hardhat/npm projects: install dependencies without running package scripts
      - run: if [ -f package-lock.json ]; then npm ci --ignore-scripts --no-audit --no-fund; fi
      - uses: kestrova/solidity-scan-action@v1
        with:
          fail-on: high             # none (default) | low | medium | high
```

The results appear in the job summary:

| Status | Severity | Finding | Detail |
|---|---|---|---|
| NEW | High | `reentrancy-eth` | in new code |

## Inputs

| Input | Default | Description |
|---|---|---|
| `mode` | `changed` on pull requests, otherwise `all` | `changed`: only changed files, new findings only. `all`: the whole project or `targets`. |
| `base` | `origin/<base branch>` | Base revision for `changed` mode. |
| `targets` | Foundry `src` / Hardhat `contracts` | Space-separated files or folders for `all` mode. |
| `fail-on` | `none` | Fail on findings at this level or worse (`changed` mode: new findings only). |
| `include-tests` | `false` | Also scan `*.t.sol`, `*.s.sol`, `test/` and `script/`. |
| `working-directory` | `.` | Project directory. |
| `cli-version` | `0.1.6` | Exact version of the `kestrova-scan` npm CLI to run. |

Output: `exit-code` (0 ok, 1 threshold reached, 2 nothing to scan, 3 service unavailable or rate-limited).

## Security and privacy

- Use the `pull_request` trigger, not `pull_request_target`. The action needs no token and no secrets.
- Pull request code is only **read**. Only `.sol` files (your targets and the files they import) are uploaded to the
  Kestrova service for analysis. Files that look like they contain a secret are refused unless you opt in.
- The CLI is installed at an exact, immutable npm version, outside your checkout, with install scripts disabled. It
  has no dependencies and no payment code.
- No telemetry.

## Free preview vs. paid results

The free preview returns severity counts, the names of High/Medium detectors and one example finding. When you need
the exact file and line of every finding, you can buy it per scan via [x402](https://x402.org), outside this action:
0.25 USDC for every finding with locations, or 1 USDC for a full report with an explanation and fix per finding, PDF
and SARIF. Only with your explicit approval, and you're charged only if the scan succeeds.

- Website and docs: https://kestrova-api.onrender.com/developers
- MCP server for AI coding agents: https://kestrova-api.onrender.com/mcp
- Example of a full report: https://kestrova-api.onrender.com/sample-report
- Contact: kestrova.security@gmail.com

## License

MIT
