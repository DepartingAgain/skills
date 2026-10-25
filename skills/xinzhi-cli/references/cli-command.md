# 新枝 CLI 基础命令

使用前通过 `xinzhi self status --json` 检查是否有新版，有更新时先更新 CLI。本文件提供常用流程示例；执行前确认用户意图、目标 ID 和当前参数。本文件未覆盖的能力或参数变化可查看 `xinzhi --help`、对应子命令的 `--help` 或 [npm 包最新 README](https://www.npmjs.com/package/@xinzhi-zone/cli?activeTab=readme)。参数以当前安装版本的帮助为准。

## 安装、登录与更新

```bash
npm install -g @xinzhi-zone/cli@latest
xinzhi --version
xinzhi --help

# 浏览器 OAuth 登录（推荐）
xinzhi login

# 或直接保存 Web 端提供的 CLI AccessToken
xinzhi login <CLI_ACCESS_TOKEN>
```

自动化或 Agent 使用 CLI 前，先检查 CLI 版本和账号登录状态：

```bash
xinzhi self status --json
xinzhi account
```

如果返回 `cli.updateAvailable = true`，先更新，再重新检查状态并阅读返回的 README URL：

```bash
xinzhi self update --yes
xinzhi self status --json
```

`self status` 检查版本与使用指引，不代表已登录；`account` 查询当前账号和登录态。需要退出当前账号时使用 `xinzhi logout`。

## 合辑

合辑用于组织内容。新枝默认提供 `暂存箱`、`想法`、`归档` 三个默认合辑，`custom=false`，不能删除。

```bash
xinzhi container create --title "AI Inbox" --description "Agent Inbox"
xinzhi container list --keyword "AI" --page-size 10
xinzhi container get --name "AI Inbox"
xinzhi container update --id "<container-id>" --title "AI Inbox V2"
```

## 标签

```bash
xinzhi tag create --name "AI/Product"
xinzhi tag list --keyword "AI" --page-size 10
xinzhi tag get --name "AI/Product"
xinzhi tag update --id "<tag-id>" --name "AI/Engineering"
```

## 内容

创建内容：

```bash
xinzhi note create-link --url "https://example.com" --container-name "AI Inbox" --tag-names "AI/Product"
xinzhi note create-memo --content "Today's idea..." --container-name "AI Inbox" --tag-names "AI/Product"
xinzhi note create-memo --content "Today's idea..." --image ./a.png --image ./b.jpg
xinzhi note create-document --title "Weekly Summary" --content $'# Weekly Summary\n\n- item 1\n- item 2' --container-name "AI Inbox" --tag-names "AI/Product"
xinzhi note create-document --title "Weekly Summary" --content "$(cat ./weekly-summary.md)" --container-name "AI Inbox" --tag-names "AI/Product"
xinzhi note create-document --title "Independent Plan" --content "$(cat ./plan.md)" --cover-img ./cover.jpg --description "Thinking training plan" --author "Author A" --author "Author B" --publish-time "2026-05-20 10:30:00"
xinzhi note create-document --title "Translated Article" --bilingual-content-file ./bilingual-content.json --container-name "AI Inbox"
xinzhi note create-highlight --note-id "<document-note-id>" --text "selected text from original document" --prefix "text before selection" --suffix "text after selection" --highlight default --highlight-color yellow
xinzhi note create-annotation --note-id "<note-id>" --content "My overall thoughts on this article"
xinzhi note create-annotation --note-id "<document-note-id>" --content "My annotation note" --text "selected text from original document" --prefix "text before selection" --suffix "text after selection"
```

查找内容：

```bash
xinzhi note list --container-name "AI Inbox" --tag-name "AI/Product" --page-size 10
xinzhi note list --author-id "<author-id>" --start-time "2026-01-01" --end-time "2026-03-31" --time-field publishTime --note-type DOCUMENT --sort-by publishTime --sort desc --page-size 10
xinzhi note search --keyword "Agent" --note-type RICH_TEXT --page-size 20
xinzhi note aggregate-search --keyword "Agent" --page-size 20
xinzhi note get --id "<note-id>"
```

整理内容：

```bash
xinzhi note add-tag --id "<note-id>" --tag-name "AI/Product"
xinzhi note remove-tag --id "<note-id>" --tag-id "<tag-id>"
xinzhi note set-tags --id "<note-id>" --tag-names "AI/Product,Thinking"
xinzhi note set-tags --id "<note-id>" --clear-tags
xinzhi note move --id "<note-id>" --to-container-name "Inbox"
xinzhi note archive --id "<note-id>"
xinzhi note unarchive --id "<note-id>"
```

### 更新已有笔记

先用 `note get` 确认目标，再使用 `note update` 原地修改。只更新显式传入的字段，保留笔记 ID 和链接；未传入的正文、标签、作者等字段保持不变。

```bash
xinzhi note get --id "<note-id>"
xinzhi note update --id "<note-id>" --author "Real Author"
xinzhi note update --id "<note-id>" --description "Updated summary" --tag-names "AI/Product,Thinking"
xinzhi note update --id "<note-id>" --title "New Title" --cover-img ./cover.jpg --publish-time "2026-05-20 10:30:00"
xinzhi note update --id "<note-id>" --content "$(cat ./revised.md)"
xinzhi note update --id "<note-id>" --bilingual-content-file ./bilingual-content.json
xinzhi note update --id "<note-id>" --container-name "Inbox" --clear-tags
```

- 至少传入一个待更新字段；`--description ""` 清空摘要。
- `--author` 可重复传入，替换全部作者；`--clear-author` 清空作者，两者互斥。作者修改仅作用于该笔记，不会修改账号名称或其他笔记。
- `--container-id` 与 `--container-name` 互斥；`--tag-ids`、`--tag-names`、`--clear-tags` 互斥，标签参数替换全部标签。只需追加一个标签时使用 `note add-tag`。
- `--content` 与 `--bilingual-content-file` 互斥，都会替换整个正文；修改前先读取原文，不要把局部编辑误写成全文替换。
- 元数据支持所有笔记类型；正文替换支持 `DOCUMENT`、`LINK`、`TEXT`、`RICH_TEXT`、`HIGHLIGHT`、`ANNOTATION`，不支持 `IMAGE`、`AUDIO`、`FILE`、`VIDEO`。双语正文更新仅支持 `DOCUMENT`。
- `--image` 可重复传入，将图片追加到新正文，必须同时提供正文参数。双语更新保留已有链接和笔记自身的翻译缓存。

### 朗读与播客音频

```bash
xinzhi note audio voices
xinzhi note audio list --note-id "<note-id>"
xinzhi note audio read generate --note-id "<note-id>" --voice-type <voice-type>
xinzhi note audio read generate --note-id "<note-id>" --voice-type <voice-type> --content-source TRANSLATION
xinzhi note audio podcast generate --note-id "<note-id>"
xinzhi note audio podcast generate --note-id "<note-id>" --content-source TRANSLATION
xinzhi note audio status --task-id "<task-id>"
```

- 朗读生成前先运行 `voices`，从当前用户可用音色中选择 `voiceType`，不要猜测音色编号。播客生成不需要指定音色。
- `--content-source` 默认为 `ORIGINAL`，译文使用 `TRANSLATION`。
- 生成是异步任务。保存返回的任务 ID，用 `status` 查询；只有 `status=completed` 且存在 `finalAudioUrl` 时才能交付最终音频。任务失败时报告实际状态和错误，不要无限轮询或反复提交生成。
- 已有音频时默认拒绝重新生成。`--force` 会先删除已有音频，使用前先向用户确认覆盖意图，再对对应的 `generate` 命令追加该参数。
- 音频命令的笔记 ID、任务 ID 必须为 24 位十六进制 ObjectId；使用真实查询或生成返回的 ID。

### noteType

`note list` 和 `note search` 支持通过 `--note-type` 过滤具体内容类型。

| noteType     | 含义                                                        |
| ------------ | ----------------------------------------------------------- |
| `LINK`       | 保存链接后的初始类型；链接解析成功后会转为 `DOCUMENT`。     |
| `DOCUMENT`   | 解析后的链接文章，或直接通过 `create-document` 创建的文档。 |
| `FILE`       | 通过新枝导入能力导入的文件。                                |
| `VIDEO`      | 保存的视频链接，例如 Bilibili 或 YouTube。                  |
| `RICH_TEXT`  | 新枝里写下的想法或 memo。                                   |
| `IMAGE`      | 通过新枝微信助手发送的图片。                                |
| `AUDIO`      | 通过新枝微信助手发送的语音，或在新枝 App 中创建的录音。     |
| `HIGHLIGHT`  | 从 `DOCUMENT` 选中内容创建的高亮。                          |
| `ANNOTATION` | 附着在内容上的批注，最常见于 `DOCUMENT`。                   |

```bash
xinzhi note list --note-type DOCUMENT --page-size 10
xinzhi note search --keyword "Agent" --note-type RICH_TEXT
```

Markdown 内容不要在普通双引号里传字面量 `\n`。使用 `$'...'`、heredoc，或从 `.md` 文件读取。

`create-document` 会上传 markdown 中的本地图片，例如 `![caption](./images/a.png)`；远程图片 URL 保持不变。`--cover-img` 支持远程 `http(s)` URL 或本地图片路径。本地封面图会先上传再创建内容。CLI 不暴露 upload-token 命令，图片上传凭证由 CLI 内部处理。

### 创建双语文档

用户提供原文与译文并要求新建双语文档时，使用 `note create-document --bilingual-content-file`。先查看最新参数，再把原文和对应译文逐块写入本地 JSON 文件：

```bash
xinzhi note create-document --help
xinzhi note create-document --title "Translated Article" --bilingual-content-file ./bilingual-content.json --container-name "AI Inbox"
```

`create-document` 必须且只能提供 `--content` 与 `--bilingual-content-file` 之一。双语 JSON 只描述正文，相对图片路径相对于该文件所在目录解析；标题、合辑、标签、封面、作者、发布时间仍用普通 CLI 选项。

- `--title` 必填。
- `--container-id` 与 `--container-name` 互斥；`--tag-ids` 与 `--tag-names` 互斥，多个标签用逗号分隔。
- 作者使用可重复的 `--author`，封面使用 `--cover-img`，摘要使用 `--description`，发布时间使用 `--publish-time`。
- 按正文顺序构造 `blockPairList`，每项的 `original` 和 `translation` 分别保存对应的原文与译文 Markdown，不要把译文作为正文尾部附录。
- 修改已有双语文档使用 `note update --id "<note-id>" --bilingual-content-file ./bilingual-content.json`，会替换整个正文，先读取并保留需要的内容。

双语图文约定：

- 原文与译文 Plate 结构必须一致；原文有图时，译文同位置也要有图。
- 两边优先使用同一本地相对路径（或同一远程 URL）。同一本地文件只会上传一次，并改写成同一个临时 path。
- 图片本身不翻译。优先两边都用空 alt（`![](...)`），或使用相同 caption；若写了 caption/alt，会按可翻译段处理。
- 落库正文只保留原文侧的图片；译文侧图片 markdown 仅用于结构对齐和段抽取。

当前阅读器译文导入约定支持 `sourceLanguage=en` 与 `targetLanguage=zh-CN`：

```json
{
  "sourceLanguage": "en",
  "targetLanguage": "zh-CN",
  "blockPairList": [
    {
      "original": "![](./images/diagram.png)\n\nArtificial intelligence is changing software development.",
      "translation": "![](./images/diagram.png)\n\n人工智能正在改变软件开发。"
    }
  ]
}
```

## 作者

```bash
xinzhi author list --keyword "张" --page-size 10
```

## 反馈

当 Agent 遇到 CLI 能力缺口或 CLI/API bug 时使用反馈命令。

- `REQUIREMENT`：期望 CLI 增加某项能力。
- `BUG`：已有命令行为异常或结果不符合预期。
- `feedback list/get` 只返回当前 CLI token 用户提交的反馈。

```bash
xinzhi feedback submit --type REQUIREMENT --content "希望 CLI 支持批量移动 note" --expected-capability "批量移动 note"
xinzhi feedback submit --type BUG --content "note list 过滤结果异常" --command "xinzhi note list --note-type DOCUMENT" --error-output "unexpected empty list"
xinzhi feedback list --type REQUIREMENT --status PENDING --page-size 20
xinzhi feedback get --id "<feedback-id>"
xinzhi feedback reply --id "<feedback-id>" --content "补充一下具体使用场景..."
```

## 空间

```bash
xinzhi space create --title "Research" --intro "Curated notes"
xinzhi space list
xinzhi space get --id "<space-id>"
xinzhi space get --path "research" --no-content
xinzhi space update --id "<space-id>" --title "Research V2"
xinzhi space sort --ids "<space-id-1>,<space-id-2>"
xinzhi space delete --id "<space-id>" --yes
```

空间模块当前只支持创建 `GENERIC` 模块。系统模块 `moduleKind=SYSTEM_SPECIAL` 只能识别，CLI 暂不支持创建；包含系统模块的空间不能通过当前发布流程发布。

创建空间模块：

```bash
xinzhi space module create \
  --space-id "<space-id>" \
  --title "Inbox Articles" \
  --data-source NOTE \
  --ui-template NOTE_CARD_META_LIST \
  --filter '[{"property":"NOTE_CONTAINER","operator":"CONTAINS","valueType":"LIST_OBJECT_ID","value":["<container-id>"]}]' \
  --sort '[{"sortBy":"createTime","sort":"DESC"}]' \
  --limit 20

xinzhi space module delete --space-id "<space-id>" --module-id "<module-id>" --yes
xinzhi space module sort --space-id "<space-id>" --ids "<module-id-1>,<module-id-2>"
```

`--filter` 和 `--sort` 是 JSON 数组。多个 filter 在后端按 AND 处理。`property`、`operator`、`valueType` 使用后端枚举值。

Android 仍依赖 container 维度订阅分发时，推荐围绕明确 container 创建模块，并用相同 `containerId` 创建订阅码：

```bash
xinzhi container list --keyword "Research"

xinzhi space module create \
  --space-id "<space-id>" \
  --title "Research Articles" \
  --data-source NOTE \
  --ui-template NOTE_CARD_META_LIST \
  --filter '[{"property":"NOTE_CONTAINER","operator":"CONTAINS","valueType":"LIST_OBJECT_ID","value":["<container-id>"]}]' \
  --sort '[{"sortBy":"createTime","sort":"DESC"}]' \
  --limit 20

xinzhi space subscribe-code create \
  --space-id "<space-id>" \
  --container-id "<container-id>" \
  --duration 30 \
  --remark "Research"
```

### 空间模块过滤

| data-source      | property              | operator                        | valueType        |
| ---------------- | --------------------- | ------------------------------- | ---------------- |
| `NOTE`           | `NOTE_CONTAINER`      | `CONTAINS` / `DOES_NOT_CONTAIN` | `LIST_OBJECT_ID` |
| `NOTE`           | `NOTE_TAG`            | `CONTAINS` / `DOES_NOT_CONTAIN` | `LIST_OBJECT_ID` |
| `NOTE`           | `NOTE_AUTHOR`         | `CONTAINS` / `DOES_NOT_CONTAIN` | `LIST_OBJECT_ID` |
| `NOTE`           | `NOTE_AGGREGATE_TYPE` | `CONTAINS` / `DOES_NOT_CONTAIN` | `LIST_STRING`    |
| `NOTE_CONTAINER` | `CONTAINER_PARENT_ID` | `CONTAINS` / `DOES_NOT_CONTAIN` | `LIST_OBJECT_ID` |
| `NOTE_CONTAINER` | `CONTAINER_TITLE`     | `CONTAINS` / `STARTS_WITH`      | `STRING`         |
| `NOTE_TAG`       | `TAG_CONTAINER`       | `CONTAINS` / `DOES_NOT_CONTAIN` | `LIST_OBJECT_ID` |
| `NOTE_TAG`       | `TAG_PARENT`          | `CONTAINS` / `DOES_NOT_CONTAIN` | `LIST_OBJECT_ID` |

### 空间模块排序

| data-source      | sortBy                                                            |
| ---------------- | ----------------------------------------------------------------- |
| `NOTE`           | `createTime` / `updateTime` / `title` / `progress`                |
| `NOTE_CONTAINER` | `title` / `updateTime` / `createTime` / `contentCount`            |
| `NOTE_TAG`       | `contentCount` / `name` / `content` / `createTime` / `updateTime` |

### uiTemplateKey

`space module create` 使用 `--ui-template` 传入 uiTemplateKey。当前 CLI 模块创建接口支持 `NOTE`、`NOTE_CONTAINER`、`NOTE_TAG`。

| data-source      | uiTemplateKey                     | 用途                                 |
| ---------------- | --------------------------------- | ------------------------------------ |
| `NOTE`           | `NOTE_CARD_HEADLINE_META`         | 大文章卡片，包含标题、摘要和元信息。 |
| `NOTE`           | `NOTE_CARD_DATE_COVER`            | 日期导向文章卡片，带封面图。         |
| `NOTE`           | `NOTE_CARD_META_LIST`             | 简洁内容列表，展示标题和二级信息。   |
| `NOTE`           | `NOTE_CARD_QUOTE`                 | 引用样式卡片，适合短摘录或高亮。     |
| `NOTE`           | `NOTE_CARD_QUOTE_META_FOOTER`     | 带底部元信息的引用卡片。             |
| `NOTE`           | `NOTE_LIST_TITLE_META`            | 紧凑竖向列表，强调标题和元信息。     |
| `NOTE`           | `NOTE_CARD_DAILY_MINUTES`         | 每日阅读或分钟数推荐卡片。           |
| `NOTE`           | `NOTE_CARD_BROWSABLE`             | 可浏览卡片，带摘录和正文预览。       |
| `NOTE`           | `NOTE_LIST_BROWSE_THUMBNAIL_META` | 带缩略图、标题和副标题的浏览列表。   |
| `NOTE`           | `NOTE_LIST_FEED_META_FOOTER`      | Feed 样式列表，带元信息和底部动作。  |
| `NOTE`           | `NOTE_CARD_HEADLINE_META_LARGE`   | 更大的 headline 卡片，适合重点区域。 |
| `NOTE`           | `NOTE_CARD_THUMBNAIL_META`        | 缩略图和文本并排卡片。               |
| `NOTE`           | `NOTE_CARD_OVERLAY_META`          | 封面叠加标题和摘要的卡片。           |
| `NOTE`           | `NOTE_CARD_THEME_OVERLAY_META`    | 主题化封面叠加卡片。                 |
| `NOTE`           | `NOTE_CARD_BLUR_OVERLAY_META`     | 模糊背景封面叠加卡片。               |
| `NOTE`           | `NOTE_CARD_BLUR_META`             | 毛玻璃封面卡片。                     |
| `NOTE`           | `NOTE_CARD_WATERFALL_TWO_COLUMN`  | 双列图片瀑布流。                     |
| `NOTE_CONTAINER` | `CONTAINER_CARD_LIST_PREVIEW`     | 带列表预览和查看全部入口的合辑卡片。 |
| `NOTE_CONTAINER` | `CONTAINER_CARD_SPLIT_COVER`      | 分割封面布局的精选合辑卡片。         |
| `NOTE_CONTAINER` | `CONTAINER_CARD_THEME_STACK`      | 带堆叠视觉层的主题合辑卡片。         |
| `NOTE_TAG`       | `TAG_CHIP_FLOW`                   | 标签 chip 流式布局。                 |
| `NOTE_TAG`       | `TAG_CARD_ICON_TEXT`              | 带图标和文本的标签卡片。             |

### 发布和订阅码

```bash
xinzhi space publish --id "<space-id>" --path "research"
xinzhi space unpublish --id "<space-id>"
xinzhi space subscribe-code create --space-id "<space-id>" --container-id "<container-id>" --duration 30 --remark "CLI"
```

## 使用建议

通过 `tagId`、`containerId` 或 `authorId` 查询内容时，先用对应的 `tag list`、`container list`、`author list` 获取 ID，再显式传入 `note list`。

## 输出与环境

- 输出格式：JSON，适合脚本和 Agent 解析。
- Node.js 要求：`>=20.12.0`。
- 查看所有命令：`xinzhi --help`。
- 权威参数以 `xinzhi <command> --help` 为准。
- Skill 未描述的命令或参数，继续查阅对应 `--help` 或 [npm 包 README](https://www.npmjs.com/package/@xinzhi-zone/cli?activeTab=readme)。
