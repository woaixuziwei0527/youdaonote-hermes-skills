---
name: youdaonote
description: "有道云笔记官方 skill，支持笔记 CRUD（创建/读取/更新/删除）、待办管理、网页剪藏、笔记搜索、文件夹管理等基础操作。"
version: 1.0.9
hermes_adapted: true
---
# YoudaoNote — 有道云笔记

> **本机偏好**：默认保存格式为 B（.note + contentFormat: md）。用户未明确选择时选 B，不选 A。所有 .note 笔记默认检查并添加 `[[双链]]`。

通过 `youdaonote` CLI 操作有道云笔记。覆盖笔记 CRUD、待办管理、网页剪藏全场景。

## 前置条件（Agent 自动处理）

执行任何操作前，先用 Hermes 的 `terminal` 工具运行 `youdaonote -s ydn list` 检测 CLI 是否可用：

- **`command not found`** → CLI 未安装。安装方式：
  ```bash
  curl -fsSL https://artifact.lx.netease.com/download/youdaonote-cli/install.sh | bash -s -- -f -b ~/.local/bin
  export PATH="$HOME/.local/bin:$PATH"
  ```
- **`-s` 选项报错（`error: unknown option '-s'`）** → CLI 版本 < 1.3.2，需升级：先尝试 `youdaonote upgrade`；不行则重新执行安装脚本。
- **API Key 错误** → 提示用户访问 **https://mopen.163.com** 获取 API Key（须使用手机号登录，且云笔记账号已绑定手机号），然后执行 `youdaonote config set apiKey <用户提供的Key>`。
- **正常返回目录列表** → 运行 `youdaonote -s ydn version` 检查版本，若版本 < 1.3.3 需升级：执行 `youdaonote -s ydn upgrade` 或重新安装。版本满足后运行 `youdaonote -s ydn help --json` 获取当前 CLI 全部能力的结构化描述。

## 全局参数

所有命令调用时**必须**加上 `-s ydn`，用于使用统计：

```bash
youdaonote -s ydn <command> [options]
```

## Hermes 调用方式

所有 `youdaonote` CLI 命令通过 Hermes 的 `terminal` 工具执行。需要写文件时使用 `write_file` 工具。

`save` 命令的 contentFile 模式（默认 .note 格式，支持 `[[双链]]`）：
1. 先用 `write_file` 将 Markdown 内容写入 `/tmp/note-content.md`
2. 再用 `terminal` 执行 `printf '%s\n' '{"title":"标题.note","type":"note","contentFormat":"md","contentFile":"/tmp/note-content.md"}' | youdaonote -s ydn save --json`

## 命令速查

| 命令 | 用途 | 示例 |
|------|------|------|
| `mkdir` | 创建文件夹 | `youdaonote -s ydn mkdir "文件夹名" [-f <父目录ID>]` |
| `save` | 保存笔记（✅ 推荐，支持 Markdown 富文本） | `youdaonote -s ydn save --file note.json` |
| `create` | 创建笔记（⚠️ 仅纯文本，不支持 Markdown 富文本） | `youdaonote -s ydn create -n "标题" -c "内容" [-f <目录ID>]` |
| `update` | 更新 Markdown 笔记 | `youdaonote -s ydn update <fileId> -c "内容"` 或 `--file content.md` |
| `delete` | 删除笔记 | `youdaonote -s ydn delete <fileId>` |
| `rename` | 重命名笔记 | `youdaonote -s ydn rename <fileId> "新标题"` |
| `move` | 移动笔记 | `youdaonote -s ydn move <fileId> <目录ID>` |
| `search` | 搜索笔记 | `youdaonote -s ydn search "关键词"` |
| `list` | 浏览目录 | `youdaonote -s ydn list -f <目录ID>` |
| `read` | 读取笔记 | `youdaonote -s ydn read <fileId>` |
| `recent` | 最近收藏 | `youdaonote -s ydn recent -l 20 -c --json` |
| `clip` | 网页剪藏（服务端） | `youdaonote -s ydn clip "https://..." [-f <目录ID>] --json` |
| `clip-save` | 保存外部剪藏 JSON | `youdaonote -s ydn clip-save --file data.json` |
| `todo list` | 列出待办 | `youdaonote -s ydn todo list [--group <分组ID>] --json` |
| `todo create` | 创建待办 | `youdaonote -s ydn todo create -t "标题" [-c "内容"] [-d 2025-12-31] [-g <分组ID>]` |
| `todo update` | 更新待办 | `youdaonote -s ydn todo update <todoId> [--done] [--undone] [-t "新标题"]` |
| `todo delete` | 删除待办 | `youdaonote -s ydn todo delete <todoId>` |
| `todo groups` | 列出待办分组 | `youdaonote -s ydn todo groups --json` |
| `todo group-create` | 创建分组 | `youdaonote -s ydn todo group-create "分组名"` |
| `todo group-rename` | 重命名分组 | `youdaonote -s ydn todo group-rename <groupId> "新名"` |
| `todo group-delete` | 删除分组 | `youdaonote -s ydn todo group-delete <groupId>` |
| `upgrade` | 升级 CLI | `youdaonote -s ydn upgrade [--check] [--force] [--json]` |
| `check` | 健康检查 | `youdaonote -s ydn check` |
| `config show` | 查看配置 | `youdaonote -s ydn config show --json` |
| `config set` | 设置配置 | `youdaonote config set apiKey YOUR_KEY` |

## 笔记管理

**默认创建方式**：所有笔记一律使用 `save` 命令，格式为 `type: "note"` + `contentFormat: "md"`（.note 容器 + Markdown 内容），这样才能支持有道云笔记原生的 `[[双链]]` 应用内跳转。
**禁止使用 `create` 命令保存包含 Markdown 格式的内容**（标题、列表、代码块、表格等）—— `create` 仅支持纯文本，会静默丢失所有格式。

### 格式选择

当内容包含 Markdown 特征（`#` 标题、`**粗体**`、代码块、列表等），先询问用户选 A 或 B：

- **A** Markdown 笔记（.md），`type: "md"` — 不支持 `[[双链]]` 跳转
- **B** 有道专有格式（.note），`type: "note"`, `contentFormat: "md"` — 支持 `[[双链]]` 跳转（推荐）

用户未明确选择时默认选 B（本机偏好。通用版默认 A，此处已覆盖）。

### 双向链接检查（必须执行）

每次创建或更新 .note 格式笔记时，Agent **必须**检查内容中是否包含 `[[笔记标题]]` 模式的双链：

1. 扫描内容，识别所有 `[[...]]` 引用
2. 对每个引用，用 `youdaonote -s ydn search "笔记标题"` 确认目标笔记是否存在
3. 已存在 → 保留 `[[笔记标题]]` 双链（有道客户端内可点击跳转）
4. 不存在 → 替换为 `→ 笔记标题` 文本标记（待目标笔记创建后再替换为双链）

### 行为准则（Agent 必须遵守）

**绝对禁止绕过 Skill 标准流程直接手写 CLI 命令。** 每条 youdaonote 操作必须走完整流程：

1. `youdaonote -s ydn list` — 检测 CLI 可用性
2. 检测内容是否含 Markdown 特征（`#`、`**`、代码块、列表等）
3. 含 Markdown 特征 → 询问用户选 A/B
4. contentFile 两步模式：`write_file` 写 `/tmp/` 临时文件 → `terminal` 执行 `printf | youdaonote -s ydn save --json`
5. 双链检查：扫描 `[[...]]`，确认目标存在后保留，否则替换为 `→`

**常见违规**：Agent 因对 CLI 命令熟悉，直接调用 `terminal` 一步完成 save，跳过检测和询问步骤。

**合规做法**：即便临时文件已存在，仍须显式执行检测和询问，不可省略。

### contentFile 模式

避免 JSON 转义问题，大内容/含换行内容使用 contentFile 模式：

**默认 B（.note，推荐）：**
```
Step 1：write_file 将 Markdown 写入 /tmp/note-content.md
Step 2：terminal 执行：
printf '%s\n' '{"title":"标题.note","type":"note","contentFormat":"md","contentFile":"/tmp/note-content.md","parentId":"文件夹ID"}' | youdaonote -s ydn save --json
```

**选 A（.md）：**
```
Step 1：write_file 将 Markdown 写入 /tmp/note-content.md
Step 2：terminal 执行：
printf '%s\n' '{"title":"标题.md","type":"md","contentFile":"/tmp/note-content.md","parentId":"文件夹ID"}' | youdaonote -s ydn save --json
```

短内容（无换行/特殊字符）可直接内联 content 字段，不用 contentFile。

`parentId` 为可选字段：填写 `youdaonote -s ydn list` 返回的文件夹 ID 可指定目标目录；不填则默认存入「我的资源/收藏笔记」。

### 其他操作

```bash
youdaonote -s ydn search "关键词"
youdaonote -s ydn list [-f <目录ID>]            # 浏览目录，id 可传给 read
youdaonote -s ydn read <fileId>                 # 返回 JSON 含 content、rawFormat（md/note/txt）和 isRaw
youdaonote -s ydn recent -l 20 -c --json        # 最近收藏
youdaonote -s ydn update <fileId> -c "新内容"
youdaonote -s ydn update <fileId> --file content.md  # 大内容（>10KB）从文件读取
youdaonote -s ydn delete <fileId>
youdaonote -s ydn rename <fileId> "新标题"
youdaonote -s ydn move <fileId> <目录ID>
```

## 网页剪藏

```bash
youdaonote -s ydn clip "https://example.com/article" --json
youdaonote -s ydn clip "https://example.com/article" -f <目录ID> --json  # 保存到指定目录
```

## 故障排查

运行 `youdaonote -s ydn check --json`，根据 `status: "fail"` 的项执行：

| 失败项 | 处理动作 |
|--------|---------|
| `config-file` / `api-key` | `youdaonote config set apiKey YOUR_KEY` |
| `mcp-connection` | API Key 有效但网络不通，提示用户检查网络或稍后重试 |

## Hermes 环境陷阱

### 不要在 execute_code 沙箱里调用 CLI

`execute_code` 的沙箱环境没有 `youdaonote`，因为 `~/.local/bin` 不在沙箱 PATH 里。所有 `youdaonote` 命令必须通过 `terminal` 工具执行。

### 务必走 Skill 的 contentFile 协议

保存笔记时，必须遵循「`write_file` 写临时文件 → `terminal` 执行 `printf ... | youdaonote -s ydn save --json`」两步。不要自以为直接调 `youdaonote` 更快就绕过协议。

## 注意事项

- **必须走完整 Skill 流程**：CLI 检测 → 格式检测 → A/B 询问 → contentFile 两步 → 双链检查
- **默认格式为 .note**：所有笔记用 `type: "note"` + `contentFormat: "md"`，支持 `[[双链]]` 应用内跳转
- 所有命令支持 `--json` 输出机器可解析格式
- 大内容通过 `--file` 传递，避免命令行参数限制
- `list` 输出的 `id` 与 `read` 的 `fileId` 等价
- `read` 返回的 `rawFormat` 标识笔记原始格式：`md`=Markdown、`note`=云笔记、`txt`=纯文本；`isRaw` 标识 content 是否为原始内容
- **禁止用 `create` 保存 Markdown 内容**：`create` 不支持 `contentFormat`，有格式需求时一律使用 `save`
- `save` 命令通过 JSON 的 **`parentId`** 字段指定目标文件夹（值来自 `list` 返回的文件夹 ID）；不传则默认存到「我的资源/收藏笔记」
