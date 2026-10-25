# 新枝 Skills

这个仓库用于维护新枝相关的 Agent Skills。

## Skills

| Skill | 用途 |
| --- | --- |
| `xinzhi-cli` | 使用新枝 CLI 安装与登录、查询和更新内容、生成朗读与播客音频，以及管理合辑、标签和空间。 |

使用前检查并更新到最新版 CLI。Skill 未覆盖的用法或参数有变化时，查看当前安装版本对应命令的 `--help` 或 [npm 包最新 README](https://www.npmjs.com/package/@xinzhi-zone/cli?activeTab=readme)。

## 目录结构

```text
skills/
  xinzhi-cli/
    SKILL.md
    references/
      cli-command.md
```

## 安装

安装 `xinzhi-cli`：

```bash
npx skills add DepartingAgain/skills --skill xinzhi-cli -g -y
```

如果当前环境能识别正在使用的 Agent，安装器会安装到对应 Agent；否则按安装器提示选择目标 Agent。

## 更新已安装的 Skill

Skill 不会自动热更新到正在运行的 Agent 会话里。更新方式取决于用户当初怎么安装：

- 如果是从 Git 仓库安装：重新执行安装命令。
- 如果是手动复制安装：用本仓库里的对应 skill 目录覆盖已安装目录。
- 如果 Agent 会话已经读取过旧版 Skill：重新安装后开启新会话，或按目标 Agent 的方式重新加载 Skill。

因此前端或文档里可以提醒用户：如果已经安装过新枝 Skills，升级时重新安装或覆盖对应 skill 目录即可。
