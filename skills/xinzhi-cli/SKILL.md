---
name: xinzhi-cli
description: 当用户要求 Agent 使用新枝 CLI、操作新枝内容、安装或登录 xinzhi 命令行时使用。
---

# 新枝 CLI

使用本 Skill 通过最新版 `xinzhi` CLI 操作新枝内容、创建双语文档、编辑已有笔记、生成朗读或播客音频，以及管理合辑、标签和空间。

## 规则

- 优先执行真实 `xinzhi` 命令，不确定参数时先运行 `xinzhi --help` 或对应子命令的 `--help`。
- 本 Skill 只覆盖常用流程；没有相关描述时，查看对应命令的 `--help` 或 [npm 包 README](https://www.npmjs.com/package/@xinzhi-zone/cli?activeTab=readme)，不要仅因 Skill 未收录就判断 CLI 不支持。参数以当前安装版本的 `--help` 为准。
- 不要编造 CLI token、API key、登录状态、付费状态、第三方账号状态或用户数据 ID。
- 用户提供的 ID、文本、路径要原样保留。
- 修改已有用户数据前，先使用 `list`、`get` 或 `search` 确认目标。
- 执行 `delete`、批量修改、影响已发布空间前，先询问用户。

## 使用新枝 CLI

安装与登录：

需要 Node.js `>=20.12.0`。

```bash
npm install -g @xinzhi-zone/cli@latest
xinzhi --version
xinzhi login
xinzhi login <CLI_TOKEN>
```

自动化或 Agent 使用 CLI 前，先检查版本与登录态：

```bash
xinzhi self status --json
xinzhi account
```

如果未登录或 token 不可用，运行 `xinzhi login`，等待用户完成 OAuth 登录后重新检查。只有用户已经提供 CLI token 时，才使用 token 参数。

`self status` 用于检查 CLI 版本与使用指引；登录状态和当前账号使用 `xinzhi account` 确认。如果 `cli.updateAvailable = true`，运行 `xinzhi self update --yes`，再重新检查状态并阅读返回的 README URL。

## 命令查找

常用内容操作、笔记更新和音频生成见 [references/cli-command.md](references/cli-command.md)。创建英中对照等双语文档时，先阅读其中的[创建双语文档](references/cli-command.md#创建双语文档)，按原文与译文逐块配对的 JSON 录入，不要把两种语言拼成普通正文。

CLI 参数可能更新，执行前查看当前命令的 `--help`；Skill 中的示例用于说明流程。未覆盖的命令逐级查看帮助：

```bash
xinzhi --help
xinzhi note --help
xinzhi note create-document --help
xinzhi note update --help
xinzhi note audio --help
xinzhi note audio read generate --help
```

更多用法见 [npm 包 README](https://www.npmjs.com/package/@xinzhi-zone/cli?activeTab=readme)。遇到版本差异时，先检查 `xinzhi --version` 和对应 `--help`。
