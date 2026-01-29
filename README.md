# QAAtlas

AI-powered test plan generator that analyzes code diffs and produces test plans, risk assessments, and coverage maps.

## Features

- **Test Plan Generation**: Automatically generates comprehensive test cases based on code changes
- **Risk Assessment**: Identifies potential risks and provides mitigation strategies
- **Coverage Mapping**: Maps changed files to recommended tests and identifies coverage gaps
- **Multiple Output Formats**: JSON report, Markdown test plan, risk assessment, PR comments, and coverage maps
- **CI/CD Integration**: Works seamlessly with GitHub Actions and other CI systems
- **Flexible Configuration**: YAML-based configuration with sensible defaults

## Installation

### npm (Recommended)

```bash
npm install -g qaatlas
```

Or use directly with npx:

```bash
npx qaatlas analyze
```

### macOS (Universal Binary - Intel & Apple Silicon)

```bash
curl -L -o /usr/local/bin/qaatlas \
  https://github.com/mauricecarrier7/qaatlas-dist/releases/download/v1.0.4/qaatlas-macos
chmod +x /usr/local/bin/qaatlas
xattr -d com.apple.quarantine /usr/local/bin/qaatlas 2>/dev/null || true
```

### Linux x64

```bash
curl -L -o /usr/local/bin/qaatlas \
  https://github.com/mauricecarrier7/qaatlas-dist/releases/download/v1.0.4/qaatlas-linux-x64
chmod +x /usr/local/bin/qaatlas
```

### Linux ARM64

```bash
curl -L -o /usr/local/bin/qaatlas \
  https://github.com/mauricecarrier7/qaatlas-dist/releases/download/v1.0.4/qaatlas-linux-arm64
chmod +x /usr/local/bin/qaatlas
```

### Windows x64

Download `qaatlas-win-x64.exe` from the [latest release](https://github.com/mauricecarrier7/qaatlas-dist/releases/latest) and add to your PATH.

## Quick Start

1. Set your Anthropic API key:
   ```bash
   export ANTHROPIC_API_KEY=your-api-key
   ```

2. Run analysis on your changes:
   ```bash
   # Analyze staged/unstaged git changes
   qaatlas analyze

   # Analyze a specific diff file
   qaatlas analyze --diff changes.patch

   # Pipe diff from stdin
   git diff main...HEAD | qaatlas analyze
   ```

3. Check the output in `./qaatlas-output/`:
   - `report.json` - Full structured report
   - `testplan.md` - Human-readable test plan
   - `risk.md` - Risk assessment document
   - `pr-comment.md` - Condensed PR comment
   - `coverage-map.json` - File-to-test mapping

## Configuration

Create a `qaatlas.yml` file in your project root:

```yaml
version: "1"

provider:
  name: claude
  model: claude-sonnet-4-20250514
  maxTokens: 4096

analysis:
  maxFiles: 20
  maxLinesPerFile: 500
  includePatterns:
    - "**/*.ts"
    - "**/*.tsx"
    - "**/*.js"
  excludePatterns:
    - "**/node_modules/**"
    - "**/*.test.*"

output:
  dir: ./qaatlas-output
  formats:
    - report
    - testplan
    - risk
    - pr-comment
    - coverage-map

context:
  includeImports: true
  includeFunctionSignatures: true
  surroundingLines: 10
```

## CLI Options

```
qaatlas analyze [options]

Options:
  -c, --config <path>      Path to config file (qaatlas.yml)
  -d, --diff <path>        Path to diff file
  -o, --output-dir <path>  Output directory for generated files
  -p, --provider <name>    LLM provider (default: claude)
  -m, --model <name>       Model to use for analysis
  -f, --formats <formats>  Comma-separated output formats
  -v, --verbose            Enable verbose logging
  --dry-run                Parse diff without calling LLM
  -h, --help               Display help
```

## GitHub Actions Integration

### Basic Setup

```yaml
name: QAAtlas Test Plan

on:
  pull_request:
    types: [opened, synchronize]

jobs:
  analyze:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - uses: actions/setup-node@v4
        with:
          node-version: '20'

      - run: npm install -g qaatlas

      - run: |
          git diff origin/...HEAD > diff.patch
          qaatlas analyze --diff diff.patch
        env:
          ANTHROPIC_API_KEY: 

      - uses: actions/upload-artifact@v4
        with:
          name: qa-reports
          path: ./qaatlas-output

      - name: Comment PR
        uses: actions/github-script@v7
        with:
          script: |
            const fs = require('fs');
            const comment = fs.readFileSync('./qaatlas-output/pr-comment.md', 'utf8');
            github.rest.issues.createComment({
              owner: context.repo.owner,
              repo: context.repo.repo,
              issue_number: context.issue.number,
              body: comment
            });
```

### With Binary Caching (Faster)

```yaml
- name: Cache QAAtlas
  id: cache
  uses: actions/cache@v4
  with:
    path: /usr/local/bin/qaatlas
    key: qaatlas-1.0.4-Linux-X64

- name: Install QAAtlas
  if: steps.cache.outputs.cache-hit != 'true'
  run: |
    curl -L -o /usr/local/bin/qaatlas \
      https://github.com/mauricecarrier7/qaatlas-dist/releases/download/v1.0.4/qaatlas-linux-x64
    chmod +x /usr/local/bin/qaatlas

- name: Run QAAtlas
  env:
    ANTHROPIC_API_KEY: 
  run: |
    git diff origin/...HEAD > diff.patch
    qaatlas analyze --diff diff.patch
```

## Output Formats

| Format | File | Description |
|--------|------|-------------|
| `report` | `report.json` | Complete structured report with all data |
| `testplan` | `testplan.md` | Human-readable test plan with test cases |
| `risk` | `risk.md` | Risk assessment with mitigation strategies |
| `pr-comment` | `pr-comment.md` | Condensed summary for PR comments |
| `coverage-map` | `coverage-map.json` | File-to-test mapping |

## Environment Variables

- `ANTHROPIC_API_KEY` - Your Anthropic API key (required)
- `NODE_ENV` - Set to `production` for JSON logging

## Exit Codes

- `0` - Success
- `1` - Error (config, parsing, API)
- `2` - Critical risk level detected

## Troubleshooting

### macOS Gatekeeper

If you see "cannot be opened because the developer cannot be verified":

```bash
xattr -d com.apple.quarantine /usr/local/bin/qaatlas
```

Or right-click the binary in Finder and select "Open".

### Permission Denied

```bash
chmod +x /usr/local/bin/qaatlas
```

### CI Cache Contains Bad Binary

Bust the cache by changing the cache key:

```yaml
key: qaatlas-1.0.4-Linux-X64-v2
```

## License

MIT
