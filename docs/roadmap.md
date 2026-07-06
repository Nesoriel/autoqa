# Roadmap

AutoQA should stay lightweight. The first goal is not to build a full testing platform, but to make one command genuinely useful for agents and developers.

## Milestone 0: Documentation and Shape

Status: in progress

Goals:

- define project idea
- define architecture
- define usage model
- define authentication model
- define first output contract
- prepare README and agent instructions

Deliverables:

- `README.md`
- `README.zh-CN.md`
- `docs/idea.md`
- `docs/architecture.md`
- `docs/usage.md`
- `docs/auth.md`
- `AGENTS.md`

## Milestone 1: Minimal Deterministic Scanner

Goal:

```bash
autoqa scan <url> --out <dir> --full
```

Core features:

- Go CLI skeleton
- URL normalization
- output directory creation
- HTTP GET/HEAD checks
- redirect chain collection
- `robots.txt` check
- `sitemap.xml` check
- `favicon.ico` check
- `manifest.json` check
- title extraction
- description extraction
- security header check
- `result.json`
- `summary.json`
- simple `report.md`
- safe header redaction

Success criteria:

- works without a config file
- produces stable output directory
- agents can read `report.md` and `result.json`

## Milestone 2: Browser Evidence Runner

Goal:

Add TypeScript + Playwright browser evidence collection.

Core features:

- open target page
- screenshot
- collect console errors
- collect failed network requests
- save HTML preview
- optional trace
- browser runner invoked through the main `autoqa` command

Success criteria:

- `autoqa scan <url> --full` includes browser evidence
- report links to screenshot and console/network evidence

## Milestone 3: Test Account Support

Goal:

Support safe authenticated testing.

Core features:

- form login with selectors
- `--username`
- `--password-env`
- `--login-url`
- login verification
- Playwright storage state save/reuse
- auth state redaction and git ignore guidance

Success criteria:

- can scan a login-protected dashboard with a test account
- password is never written into report/log/result

## Milestone 4: Agent-Friendly Reports

Goal:

Improve report quality and agent readability.

Core features:

- stable finding schema
- severity classification
- category classification
- finding summary
- reproduction steps
- evidence links
- suggested fixes
- regression test suggestions

Success criteria:

- report is useful without extra explanation
- result JSON is easy for agents to parse

## Milestone 5: MCP Server

Goal:

Expose AutoQA to agents through MCP.

Planned tools:

```text
autoqa.scan_site
autoqa.read_report
autoqa.list_findings
autoqa.get_evidence
autoqa.generate_regression_plan
```

Success criteria:

- an MCP-compatible agent can call AutoQA without shell-specific glue
- CLI remains the primary user-facing entrypoint

## Milestone 6: Optional Integrations

Possible integrations:

- Lighthouse summary
- ZAP security-lite scan
- k6 lightweight performance check
- GitHub issue creation
- CI output
- HTML report
- Playwright test generation

These should remain optional. AutoQA should not become heavy by default.

## Non-Goals for Early Versions

- full test management platform
- replacing Playwright
- replacing Lighthouse
- replacing ZAP
- replacing k6
- destructive security testing
- production data modification
- fully autonomous black-box testing without rules
