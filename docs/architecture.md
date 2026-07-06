# Architecture Draft

AutoQA is planned as a lightweight multi-language toolkit with one external command:

```bash
autoqa
```

Internally, the project can use Go, TypeScript, and Python, but the user and the agent should not need to care about the internal split.

## High-Level Flow

```text
User / Agent
    |
    v
autoqa CLI
    |
    v
Go Core
    |----------------------|
    v                      v
HTTP Scanner          Browser Runner
Go                    TypeScript + Playwright
    |                      |
    |----------------------|
    v
Evidence Store + Result Aggregator
    |
    v
Python Agent Analyzer / Reporter
    |
    v
report.md + result.json + evidence/
```

## Component Responsibilities

### Go Core

The Go core is responsible for stable execution and orchestration.

Planned responsibilities:

- CLI entrypoint
- config loading
- default profile selection
- HTTP scanner
- redirect chain inspection
- public asset checks
- security header checks
- concurrent URL checks
- evidence directory management
- result aggregation
- JSON output
- MCP server mode
- safe redaction helpers

The Go core should not try to replace Playwright.

### TypeScript Browser Runner

The TypeScript layer is responsible for browser automation through Playwright.

Planned responsibilities:

- open pages in Chromium initially
- collect screenshots
- collect console errors
- collect failed network requests
- save page HTML previews
- save optional traces
- execute simple login flows
- reuse Playwright storage state
- run generated or hand-written Playwright tests later

TypeScript is chosen here because Playwright's strongest ecosystem is in the Node.js / TypeScript world.

### Python Agent Layer

The Python layer is responsible for analysis and reporting.

Planned responsibilities:

- summarize scan results
- classify findings
- generate Markdown reports
- explain likely causes
- suggest fixes
- suggest regression tests
- integrate with LLM providers later
- keep prompts and analysis logic isolated from deterministic execution

The Python layer should not be required for the earliest deterministic scanner, but it is useful for the Agent-native experience.

## Data Model

Each check should produce a structured finding object.

Example:

```json
{
  "id": "robots_txt_invalid_content_type",
  "title": "robots.txt returned HTML",
  "category": "seo",
  "severity": "medium",
  "status": "failed",
  "target": "https://example.com/robots.txt",
  "evidence": [
    "evidence/http/robots.headers.txt",
    "evidence/http/robots.body.preview.txt"
  ],
  "details": {
    "status_chain": [307, 200],
    "final_content_type": "text/html; charset=utf-8",
    "expected_content_type": "text/plain",
    "first_line": "<!DOCTYPE html>"
  }
}
```

## Output Contract

The output directory should be stable so agents can reliably inspect the result.

```text
report.md              # human-readable report
result.json            # full structured result
summary.json           # compact result summary
autoqa.log             # execution log
evidence/              # raw and sanitized evidence
```

## Profiles

AutoQA should support built-in profiles.

### smoke

Fast basic scan:

- homepage HTTP check
- browser open
- screenshot
- console errors
- failed network requests

### seo

SEO and crawlability scan:

- `robots.txt`
- `sitemap.xml`
- title
- description
- canonical later
- public asset behavior

### full

Default useful scan:

- smoke
- seo
- public assets
- security headers
- browser evidence

### security-lite

Safe non-destructive checks:

- security headers
- HTTPS behavior
- obvious sensitive public paths later
- no destructive payloads by default

## MCP Mode

The MCP server should expose a small set of tools first:

```text
autoqa.scan_site
autoqa.read_report
autoqa.list_findings
autoqa.get_evidence
autoqa.generate_regression_plan
```

The first version can prioritize CLI mode. MCP mode should be added after the CLI output contract is stable.

## Safety Model

Default behavior:

- read-only
- no destructive actions
- no delete/payment/publish actions
- no password in logs
- no raw cookie/token in reports
- `Authorization`, `Cookie`, and `Set-Cookie` redacted
- auth state files ignored by git
- dangerous operations require explicit flags later

## Suggested Repository Layout

```text
autoqa/
  README.md
  README.zh-CN.md
  AGENTS.md
  docs/
    idea.md
    architecture.md
    usage.md
    auth.md
    roadmap.md
  core-go/
    cmd/
    internal/
      scanner/
      evidence/
      report/
      mcp/
      config/
  browser-ts/
    src/
      runner/
      collectors/
      playwright/
  agent-py/
    autoqa_agent/
      analyzer/
      reporter/
      planner/
      prompts/
  examples/
    basic-site.yaml
  reports/
```
