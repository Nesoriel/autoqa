# Agent Instructions for AutoQA

AutoQA is intended to be used by coding agents and assistant agents.

## Primary Goal

When a user asks you to test a website with AutoQA, prefer this command:

```bash
autoqa scan <url> --out <output-dir> --full
```

Then read:

```bash
<output-dir>/report.md
<output-dir>/result.json
```

Use the report as the main human-readable output and `result.json` for structured details.

## Default Workflow

1. Identify the target URL.
2. Choose or create an output directory.
3. Run AutoQA.
4. Read `report.md`.
5. Read `result.json` if needed.
6. Summarize important findings for the user.
7. Reference evidence files when explaining problems.

Example:

```bash
autoqa scan https://example.com --out ./reports/example --full
cat ./reports/example/report.md
```

## Authentication

If the user provides a test account, avoid putting passwords directly in shell commands.

Prefer environment variables:

```bash
export AUTOQA_TEST_PASSWORD='provided-password'

autoqa scan https://example.com \
  --out ./reports/example \
  --full \
  --login-url /login \
  --username test@example.com \
  --password-env AUTOQA_TEST_PASSWORD
```

If an auth state exists, prefer reusing it:

```bash
autoqa scan https://example.com \
  --out ./reports/example \
  --full \
  --auth-state .autoqa/auth/state.json
```

## Safety Rules

Do not perform destructive actions unless the user explicitly asks for them and AutoQA supports an explicit safe mode for that action.

Do not:

- delete data
- submit production forms
- make payments
- publish content
- send messages
- change user settings
- create real user accounts
- expose passwords, cookies, tokens, or session values

Allowed by default:

- visit pages
- log in with a test account when requested
- collect screenshots
- collect console errors
- collect failed network requests
- inspect public assets
- inspect headers
- generate reports

## Expected Output

A successful AutoQA run should produce:

```text
report.md
result.json
summary.json
autoqa.log
evidence/
```

If AutoQA fails, report:

- command attempted
- error message
- whether any partial report or evidence was created
- what the user can fix next

## Development Notes

The project is planned as a multi-language toolkit:

- Go: CLI, scanner, evidence, MCP server
- TypeScript: Playwright browser runner
- Python: analysis and report generation

But users and agents should interact with one command:

```bash
autoqa
```

Keep the user experience simple.
