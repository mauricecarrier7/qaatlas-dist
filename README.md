# QAAtlas

AI-powered test plan generator that analyzes code diffs and produces comprehensive test plans, risk assessments, and coverage maps.

## Features

- **Test Plan Generation**: Automatically generates detailed test plans from code changes
- **Risk Assessment**: Identifies high-risk areas in your code changes
- **Coverage Mapping**: Maps test coverage to changed code sections
- **Multi-format Output**: Generates Markdown reports, JSON data, and PR comments

## Installation

### npm (Recommended)

```bash
npm install -g qaatlas
```

### macOS (Universal Binary - Intel & Apple Silicon)

```bash
curl -L -o /usr/local/bin/qaatlas \
  https://github.com/mauricecarrier7/qaatlas-dist/releases/download/v1.0.0/qaatlas-macos
chmod +x /usr/local/bin/qaatlas
xattr -d com.apple.quarantine /usr/local/bin/qaatlas 2>/dev/null || true
```

### Linux x64

```bash
curl -L -o /usr/local/bin/qaatlas \
  https://github.com/mauricecarrier7/qaatlas-dist/releases/download/v1.0.0/qaatlas-linux-x64
chmod +x /usr/local/bin/qaatlas
```

### Linux ARM64

```bash
curl -L -o /usr/local/bin/qaatlas \
  https://github.com/mauricecarrier7/qaatlas-dist/releases/download/v1.0.0/qaatlas-linux-arm64
chmod +x /usr/local/bin/qaatlas
```

### Windows x64

Download `qaatlas-win-x64.exe` from the [latest release](https://github.com/mauricecarrier7/qaatlas-dist/releases/latest) and add to your PATH.

## Quick Start

1. Set your Anthropic API key:
   ```bash
   export ANTHROPIC_API_KEY=your-api-key
   ```

2. Analyze your code changes:
   ```bash
   qaatlas analyze
   qaatlas analyze --diff changes.patch
   ```

3. Check outputs in `./qaatlas-output/`

## CI/CD Integration

```yaml
- name: Install QAAtlas
  run: npm install -g qaatlas@1.0.0

- name: Run Analysis
  env:
    ANTHROPIC_API_KEY: 
  run: |
    git diff origin/...HEAD > diff.patch
    qaatlas analyze --diff diff.patch
```

## License

MIT
