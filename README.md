# AutoQA

Agent-native lightweight web automated testing toolkit.

AutoQA is designed for a simple workflow:

```bash
autoqa scan https://example.com --out ./reports/example --full
```

An AI agent such as Codex, Hermes, Claude, or another coding agent should be able to run one command, collect test evidence, and produce a usable Markdown report.

> Chinese README: [README.zh-CN.md](./README.zh-CN.md)

## What AutoQA Does

AutoQA is not trying to replace Playwright, Lighthouse, ZAP, or k6. It is a lightweight orchestration and evidence layer that makes these capabilities easier for agents to use.

The first target is practical web testing:

- HTTP availability checks
- redirect chain inspection
- `robots.txt` validation
- `sitemap.xml` validation
- public asset checks such as `favicon.ico` and `manifest.json`
- title and description checks
- common security header checks
- browser screenshots
- console error collection
- network failure collection
- authenticated page testing with test accounts
- Markdown and JSON reports
- evidence directory for screenshots, headers, HTML previews, logs, and traces

## Design Goals

- One command should be enough for a useful default scan.
- Configuration should be optional, not required.
- Reports should be readable by humans and structured enough for agents.
- Evidence should be saved and traceable.
- Authentication should support test accounts without leaking secrets.
- The default mode should be safe and read-only.
- The implementation should stay lightweight and avoid becoming a heavy test management platform.

## Planned Architecture

AutoQA is planned as a small multi-language toolkit:

| Layer | Stack | Responsibility |
|---|---|---|
| Core | Go | CLI, HTTP scanner, concurrent checks, evidence store, result aggregation, MCP server |
| Browser | TypeScript + Playwright | browser automation, screenshots, traces, console logs, network logs, E2E execution |
| Agent | Python | result analysis, test planning, report writing, failure explanation, LLM integration |

Users and agents should only need to interact with one command:

```bash
autoqa
```

The internal Go, TypeScript, and Python components should be hidden behind that command.

## Planned Installation

The exact installation method is still being designed. The intended experience is:

```bash
curl -fsSL https://autoqa.dev/install.sh | sh
```

Or with Docker:

```bash
docker run --rm \
  -v "$PWD/reports:/reports" \
  ghcr.io/nesoriel/autoqa:latest \
  scan https://example.com --out /reports/example --full
```

During early development, the project may require running the Go CLI, TypeScript browser runner, and Python agent components from source.

## Planned Usage

Run a full default scan:

```bash
autoqa scan https://example.com --out ./reports/example --full
```

Run a faster smoke scan:

```bash
autoqa scan https://example.com --out ./reports/example --profile smoke
```

Run an SEO-focused scan:

```bash
autoqa scan https://example.com --out ./reports/example --profile seo
```

Use a config file:

```bash
autoqa scan --config autoqa.yaml
```

Expected output:

```text
reports/example/
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

## Test Account Support

AutoQA must support authenticated testing through test accounts. Passwords should not be passed directly on the command line.

Recommended pattern:

```bash
export AUTOQA_TEST_PASSWORD='your-test-password'

autoqa scan https://example.com \
  --out ./reports/example \
  --full \
  --login-url /login \
  --username test@example.com \
  --password-env AUTOQA_TEST_PASSWORD
```

AutoQA should also support Playwright storage state:

```bash
autoqa auth login https://example.com \
  --login-url /login \
  --username test@example.com \
  --password-env AUTOQA_TEST_PASSWORD \
  --save-state .autoqa/auth/state.json

autoqa scan https://example.com \
  --out ./reports/example \
  --full \
  --auth-state .autoqa/auth/state.json
```

Sensitive values such as passwords, cookies, tokens, `Authorization`, `Cookie`, and `Set-Cookie` must be redacted from reports and evidence by default.

## Planned Commands

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

## Documentation

- [Project idea](./docs/idea.md)
- [Architecture](./docs/architecture.md)
- [Usage design](./docs/usage.md)
- [Authentication design](./docs/auth.md)
- [Roadmap](./docs/roadmap.md)
- [Agent instructions](./AGENTS.md)

## Status

AutoQA is currently in the design stage. The first milestone is to make this command genuinely useful:

```bash
autoqa scan <url> --out <dir> --full
```

## License

TBD.
