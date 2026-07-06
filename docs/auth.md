# Authentication and Test Account Design

AutoQA must support test accounts. Without authentication support, it can only test public pages and cannot cover dashboards, user centers, admin pages, protected routes, or permission-related features.

The design goal is:

> Support authenticated testing while keeping secrets out of reports, logs, command history, and git.

## Supported Auth Modes

Planned modes:

| Mode | Purpose |
|---|---|
| `none` | public pages only |
| `form` | username/password login through the browser |
| `storage_state` | reuse Playwright storage state |
| `cookie` | provide cookies for special cases |
| `token` | API or Bearer token testing |

## Recommended Password Handling

Passwords should not be passed directly on the command line.

Avoid:

```bash
autoqa scan https://example.com --username test@example.com --password plain-text-password
```

Recommended:

```bash
export AUTOQA_TEST_PASSWORD='your-test-password'

autoqa scan https://example.com \
  --out ./reports/example \
  --full \
  --login-url /login \
  --username test@example.com \
  --password-env AUTOQA_TEST_PASSWORD
```

The report may show:

```text
Auth: enabled
Username: test@example.com
Password source: AUTOQA_TEST_PASSWORD
Login status: success
```

The report must not show the password value.

## Form Login Config

Example config:

```yaml
target: https://example.com

auth:
  mode: form
  login_url: /login
  username: test@example.com
  password_env: AUTOQA_TEST_PASSWORD

  selectors:
    username: 'input[name="email"]'
    password: 'input[name="password"]'
    submit: 'button[type="submit"]'

  verify:
    url_contains: /dashboard
    selector: 'text=Dashboard'
```

## Playwright Storage State

For repeated scans, storage state is often better than logging in every time.

Create state:

```bash
autoqa auth login https://example.com \
  --login-url /login \
  --username test@example.com \
  --password-env AUTOQA_TEST_PASSWORD \
  --save-state .autoqa/auth/state.json
```

Use state:

```bash
autoqa scan https://example.com \
  --out ./reports/example \
  --full \
  --auth-state .autoqa/auth/state.json
```

Benefits:

- faster scans
- fewer repeated login attempts
- lower chance of captcha or risk-control triggers
- easier CI usage
- better agent workflow

## Cookie Auth

Cookie auth can be useful when the project already has a safe test cookie source.

Example:

```yaml
auth:
  mode: cookie
  cookie_env: AUTOQA_TEST_COOKIE
```

Cookies must be redacted from logs and reports.

## Token Auth

Token auth can be useful for API checks.

Example:

```yaml
auth:
  mode: token
  token_env: AUTOQA_TEST_TOKEN
  header: Authorization
  scheme: Bearer
```

Tokens must be redacted from logs and reports.

## Redaction Rules

AutoQA must redact sensitive values by default.

Sensitive fields include:

- password values
- `Authorization`
- `Cookie`
- `Set-Cookie`
- bearer tokens
- session IDs
- CSRF tokens when detected
- storage state secrets in reports

Examples:

```text
Authorization: Bearer ***REDACTED***
Cookie: ***REDACTED***
Set-Cookie: ***REDACTED***
```

## Files That Must Not Be Committed

Recommended ignored paths:

```gitignore
.autoqa/auth/
.autoqa/secrets/
reports/
*.storage-state.json
```

## Safety Defaults

Default behavior should be conservative:

- read-only mode enabled
- form submission disabled unless required for login
- no destructive operations
- no delete/payment/publish actions
- no real user account required
- test accounts only
- secrets redacted from evidence
- auth state ignored by git

## Write Actions

Authenticated testing does not mean AutoQA should freely perform write actions.

Default allowed behavior:

- login with test account
- visit protected pages
- collect screenshots
- collect console errors
- collect network failures
- inspect page content

Default forbidden behavior:

- delete records
- submit production forms
- publish content
- send messages
- pay money
- change user settings
- batch modify data

Future versions may support explicit write testing through flags such as:

```bash
autoqa scan https://example.com --allow-write
```

But the first version should stay read-only except for the login action itself.
