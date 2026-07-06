# Usage Design

AutoQA should be easy enough for an agent to use without reading a long manual.

The primary command is:

```bash
autoqa scan <url> --out <dir> --full
```

## Default Usage

```bash
autoqa scan https://example.com
```

Default behavior should be useful even without a config file.

Expected default checks:

- homepage availability
- redirect chain
- `robots.txt`
- `sitemap.xml`
- `favicon.ico`
- `manifest.json`
- title and description
- common security headers
- browser open
- screenshot
- console errors
- network failures
- Markdown report
- JSON result

## Output Directory

If `--out` is not provided, AutoQA can create a timestamped output directory.

Example:

```text
autoqa-report/example.com-20260706-153000/
```

If `--out` is provided, AutoQA must write to that exact directory.

```bash
autoqa scan https://example.com --out ./reports/example
```

Expected structure:

```text
reports/example/
  report.md
  result.json
  summary.json
  autoqa.log
  evidence/
    screenshots/
      home.png
    http/
      robots.headers.txt
      robots.body.preview.txt
      sitemap.headers.txt
      sitemap.body.preview.txt
    console/
      errors.json
    network/
      failed-requests.json
    pages/
      home.html
```

## Profiles

### full

```bash
autoqa scan https://example.com --profile full
```

Equivalent to the useful default full scan.

### smoke

```bash
autoqa scan https://example.com --profile smoke
```

Fast check for obvious failures.

### seo

```bash
autoqa scan https://example.com --profile seo
```

Checks crawlability and SEO files.

### security-lite

```bash
autoqa scan https://example.com --profile security-lite
```

Runs safe, non-destructive security checks.

## Config File

A config file should be optional.

Example:

```yaml
target: https://example.com

profile: full

pages:
  - /
  - /login
  - /dashboard

checks:
  http: true
  seo: true
  browser: true
  public_assets: true
  security_headers: true

browser:
  collect:
    - screenshot
    - console_errors
    - network_errors
    - html_preview

output:
  dir: ./reports/example
  markdown: true
  json: true
  evidence: true

safety:
  readonly: true
  allow_form_submit: false
  redact_secrets: true
```

Run with config:

```bash
autoqa scan --config autoqa.yaml
```

## Agent-Friendly Usage

An agent should be able to follow this minimal process:

1. Run AutoQA.
2. Read `report.md`.
3. Read `result.json` if structured details are needed.
4. Reference files under `evidence/` when explaining findings.

Recommended agent command:

```bash
autoqa scan https://example.com --out ./reports/example --full
```

Then read:

```bash
cat ./reports/example/report.md
cat ./reports/example/result.json
```

## Planned CLI Commands

```bash
autoqa scan <url>
autoqa scan <url> --out <dir>
autoqa scan <url> --full
autoqa scan <url> --profile smoke
autoqa scan <url> --profile seo
autoqa scan --config autoqa.yaml

autoqa auth login <url>
autoqa setup
autoqa doctor
autoqa mcp
```

## Report Shape

The Markdown report should have a stable format.

```markdown
# AutoQA Test Report

## Target

https://example.com

## Summary

This scan found 3 findings: 1 medium, 2 low.

## Findings

### 1. robots.txt returned HTML

- Severity: Medium
- Category: SEO
- Evidence:
  - evidence/http/robots.headers.txt
  - evidence/http/robots.body.preview.txt
- Details:
  - Status chain: 307 -> 200
  - Content-Type: text/html; charset=utf-8
  - First line: <!DOCTYPE html>
- Impact:
  Search engines may fail to read the robots rules.
- Suggested fix:
  Exclude /robots.txt from authentication middleware.
- Regression test:
  Assert /robots.txt returns 200 and text/plain.

## Check Matrix

| Check | Result |
|---|---|
| Homepage | Pass |
| robots.txt | Fail |
| sitemap.xml | Fail |
| Console errors | Pass |
| Network failures | Pass |
```
