# AutoQA Project Idea

AutoQA is an Agent-native lightweight web automated testing toolkit.

The goal is to make web testing easy for both humans and AI agents. A user should be able to say:

> Use AutoQA to fully test this website and output a Markdown report to the target directory.

The agent should then be able to run one command, collect evidence, and produce a useful report.

```bash
autoqa scan https://example.com --out ./reports/example --full
```

## Problem

Many AI agents can operate browsers, but they often lack a reliable testing workflow.

Common problems:

- agents click around without stable evidence
- test results are difficult to reproduce
- reports are not structured
- screenshots, logs, headers, redirects, and errors are not saved consistently
- login flows are hard to reuse safely
- secrets may leak into logs or reports
- browser automation, HTTP checks, SEO checks, and report writing are scattered across different tools

AutoQA aims to solve this by providing a small, structured, safe, and agent-friendly testing layer.

## Product Positioning

AutoQA is not:

- a replacement for Playwright
- a replacement for Lighthouse
- a replacement for ZAP
- a replacement for k6
- a heavy test management platform
- a fully autonomous black-box testing agent

AutoQA is:

- a lightweight web testing orchestrator
- a stable execution layer for agents
- an evidence collector
- a report generator
- an MCP-friendly tool server
- a practical bridge between traditional QA tools and AI agents

## Core Philosophy

The tool should follow this rule:

> Rules decide pass or fail. Agents explain, summarize, and suggest next steps.

For example, whether `robots.txt` passes should be determined by clear rules:

- expected status: `200`
- expected content type: `text/plain`
- response body should not be HTML
- redirect chain should not send the request to a login page

The agent can then explain the impact and suggest a fix.

## First Use Cases

### 1. Public Site Scan

Useful for testing public pages and SEO-related assets.

Checks:

- homepage status
- redirect chain
- `robots.txt`
- `sitemap.xml`
- `favicon.ico`
- `manifest.json`
- title and description
- common security headers

### 2. Browser Evidence Scan

Useful for detecting frontend errors.

Checks:

- browser can open the page
- screenshot capture
- console errors
- failed network requests
- page HTML preview
- optional Playwright trace

### 3. Authenticated Flow Scan

Useful for dashboards, user centers, admin pages, and protected routes.

Authentication methods:

- username and password through environment variables
- Playwright storage state
- cookie-based auth
- token-based auth

### 4. Agent Report Generation

The report should include:

- summary
- severity counts
- finding list
- reproduction steps
- evidence links
- possible causes
- suggested fixes
- regression test suggestions

## First MVP

The first MVP should make this command useful:

```bash
autoqa scan <url> --out <dir> --full
```

It should output:

```text
<dir>/
  report.md
  result.json
  summary.json
  autoqa.log
  evidence/
    screenshots/
    http/
    console/
    network/
    pages/
```

The first version should support:

- homepage availability check
- `robots.txt` check
- `sitemap.xml` check
- public asset checks
- title and description checks
- security header checks
- browser screenshot
- console error collection
- network failure collection
- Markdown report
- JSON result
- safe redaction of secrets

## Long-Term Vision

AutoQA can eventually become a small but practical testing toolkit that agents can use as a default web QA skill.

Possible future abilities:

- MCP server mode
- CI integration
- GitHub issue generation
- Playwright test generation
- regression test suggestions
- Lighthouse integration
- ZAP security-lite integration
- k6 performance-lite integration
- multi-page crawling with limits
- authenticated role-based testing
- plugin system

The project should stay lightweight. The goal is not to build a huge platform, but to build a reliable tool that agents can actually use.
