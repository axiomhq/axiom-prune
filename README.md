# axiom-prune

Analyze Axiom query history to find unused data that can be pruned.

[![MIT License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)

## Installation

### Homebrew (macOS & Linux)

```bash
brew install axiomhq/tap/axiom-prune
```

### Download binary

Download the latest release from the [Releases page](https://github.com/axiomhq/axiom-prune/releases/latest).

Available platforms:
- macOS (Intel): `axiom-prune_vX.X.X_darwin_amd64.tar.gz`
- macOS (Apple Silicon): `axiom-prune_vX.X.X_darwin_arm64.tar.gz`
- Linux (x86_64): `axiom-prune_vX.X.X_linux_amd64.tar.gz`
- Linux (ARM64): `axiom-prune_vX.X.X_linux_arm64.tar.gz`

```bash
# Example for macOS Apple Silicon
curl -LO https://github.com/axiomhq/axiom-prune/releases/latest/download/axiom-prune_v1.0.0_darwin_arm64.tar.gz
tar -xzf axiom-prune_v1.0.0_darwin_arm64.tar.gz
chmod +x axiom-prune
sudo mv axiom-prune /usr/local/bin/
```

## Requirements

- `~/.axiom.toml` with deployment credentials (same format as [Axiom CLI](https://github.com/axiomhq/cli))
- Access to `axiom-history` or `axiom-audit` dataset in your deployment
- Read access to target datasets

## Usage

```bash
# Basic usage
axiom-prune --deployment prod-us

# With options
axiom-prune --deployment prod-us --output report.md --verbose

# JSON output (for programmatic use)
axiom-prune --deployment prod-us --json

# Markdown to stdout
axiom-prune --deployment prod-us --output -
```

## What It Does

1. **Discovers datasets** - Finds all datasets with usage data
2. **Analyzes query history** - Parses 90 days of APL queries
3. **Identifies unused fields** - Fields never referenced in any query
4. **Identifies droppable values** - Event slices (field=value) never positively queried
5. **Estimates savings** - Based on field presence and event share

## Output

Generates a Markdown report with:

- **TL;DR** - Quick summary of safe savings
- **Safe to Drop** - High-confidence recommendations (can act immediately)
- **Needs Review** - Lower-confidence items requiring team validation
- **Dataset Details** - Per-dataset breakdown

## Safety

Recommendations are marked "Safe" only when:

**For fields:**
- Never referenced in any query (90 days)
- <5% of queries use wildcards or dynamic access (parse_json, etc.)

**For event slices:**
- Value never appears in positive predicates (`==`, `in`)
- <10% of queries leave the field unfiltered
- <5% of queries use wildcards or dynamic access

## Limitations

- **90-day window** - May miss infrequent access patterns (annual audits, etc.)
- **Savings are estimates** - Based on field presence rate and event share
- **No actor weighting** - Doesn't distinguish dashboards from ad-hoc queries
- **Query history only** - Doesn't account for external systems reading data

## Example Output

```markdown
# Axiom Prune Report

**Deployment:** prod-us
**History coverage:** 87 days (oldest query: 2024-10-30)
**Queries analyzed:** 47,832

## TL;DR

**Drop these today, save ~5.7 TB/month:**

- prod-logs: events level == 'debug'
- prod-logs: field debug.stacktrace

## Safe to Drop Now

| Dataset | Action | Est. Savings | Evidence |
|---------|--------|--------------|----------|
| prod-logs | Drop events: `level == 'debug'` | 18.2 GB/mo | 0 positive queries, 78.3% of events |
| prod-logs | Drop field `debug.stacktrace` | 2.4 GB/mo | 0 references in 47832 queries, 12% presence |
```

## Configuration

| Flag | Default | Description |
|------|---------|-------------|
| `--deployment` | (required) | Axiom deployment name |
| `--config` | `~/.axiom.toml` | Path to config file |
| `--output` | `prune-report-<deployment>.md` | Output path (use `-` for stdout) |
| `--json` | false | Output JSON instead of Markdown |
| `--verbose` | false | Verbose output |
| `--min-dataset-gb` | 1.0 | Minimum dataset size in GB to analyze |
| `--max-ambiguous` | 0.05 | Max ambiguous query rate for safe recommendations |
| `--max-unfiltered` | 0.10 | Max unfiltered query rate for safe event slice recommendations |
| `--min-event-share` | 5.0 | Minimum event share % to report droppable values |

## License

Distributed under the [MIT License](LICENSE).
