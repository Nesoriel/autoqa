# AutoQA

面向 AI Agents 的轻量级 Web 自动化测试工具。

AutoQA 的目标体验很简单：

```bash
autoqa scan https://example.com --out ./reports/example --full
```

当用户告诉 Codex、Hermes、Claude 或其他 Agent：

> 使用 AutoQA 对这个网站完整测试一遍，并把 Markdown 报告输出到指定目录。

Agent 应该只需要执行一个命令，就能完成测试、收集证据，并生成可读的测试报告。

## AutoQA 做什么

AutoQA 不是要替代 Playwright、Lighthouse、ZAP 或 k6，而是做一个轻量的编排层和证据层，让这些能力更容易被 Agent 调用。

第一阶段重点面向实用的网站测试：

- HTTP 可访问性检查
- 重定向链检查
- `robots.txt` 检查
- `sitemap.xml` 检查
- `favicon.ico`、`manifest.json` 等公共资源检查
- 页面 `title`、`description` 检查
- 常见安全响应头检查
- 浏览器截图
- Console 错误采集
- Network 失败请求采集
- 使用测试账号进行登录后页面测试
- Markdown 报告
- JSON 结构化结果
- 证据目录，包括截图、响应头、HTML 预览、日志和 trace

## 设计目标

- 一条命令就能完成一次有用的默认扫描。
- 配置文件可选，而不是必须。
- 报告既要适合人看，也要方便 Agent 读取。
- 每个问题都应该有可追溯证据。
- 支持测试账号，但不能泄露密码、Cookie、Token。
- 默认安全、只读，不自动执行危险操作。
- 保持轻量，不做臃肿的测试管理平台。

## 技术架构草案

AutoQA 计划采用小型多语言分层架构：

| 层级 | 技术栈 | 职责 |
|---|---|---|
| Core | Go | CLI、HTTP 扫描、并发检查、证据管理、结果聚合、MCP Server |
| Browser | TypeScript + Playwright | 浏览器自动化、截图、trace、Console 日志、Network 日志、E2E 执行 |
| Agent | Python | 结果分析、测试计划生成、报告生成、失败原因解释、LLM 接入 |

对用户和 Agent 来说，只应该感知一个入口：

```bash
autoqa
```

Go、TypeScript、Python 的内部复杂度应该被隐藏起来。

## 安装方式规划

最终希望支持一键安装：

```bash
curl -fsSL https://autoqa.dev/install.sh | sh
```

也计划提供 Docker 镜像，避免 Playwright 浏览器依赖污染本地环境：

```bash
docker run --rm \
  -v "$PWD/reports:/reports" \
  ghcr.io/nesoriel/autoqa:latest \
  scan https://example.com --out /reports/example --full
```

早期开发阶段，可能需要从源码分别运行 Go CLI、TypeScript 浏览器执行器和 Python 分析模块。

## 使用方式规划

完整默认扫描：

```bash
autoqa scan https://example.com --out ./reports/example --full
```

快速冒烟测试：

```bash
autoqa scan https://example.com --out ./reports/example --profile smoke
```

SEO 专项测试：

```bash
autoqa scan https://example.com --out ./reports/example --profile seo
```

使用配置文件：

```bash
autoqa scan --config autoqa.yaml
```

预期输出目录：

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

## 测试账号支持

AutoQA 必须支持使用测试账号完成登录后页面测试。密码不应该直接写在命令行参数里，因为 shell history 可能会保存命令记录。

推荐方式：

```bash
export AUTOQA_TEST_PASSWORD='your-test-password'

autoqa scan https://example.com \
  --out ./reports/example \
  --full \
  --login-url /login \
  --username test@example.com \
  --password-env AUTOQA_TEST_PASSWORD
```

也应该支持 Playwright storage state，先保存登录态，再复用：

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

密码、Cookie、Token、`Authorization`、`Cookie`、`Set-Cookie` 等敏感信息必须默认脱敏，不能明文写入报告和证据文件。

## 计划中的命令

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

## 文档

- [项目想法](./docs/idea.md)
- [架构设计](./docs/architecture.md)
- [使用方式设计](./docs/usage.md)
- [认证与测试账号设计](./docs/auth.md)
- [路线图](./docs/roadmap.md)
- [Agent 使用说明](./AGENTS.md)

## 当前状态

AutoQA 目前处于设计阶段。第一个里程碑是让下面这条命令真正可用：

```bash
autoqa scan <url> --out <dir> --full
```

## License

TBD.
