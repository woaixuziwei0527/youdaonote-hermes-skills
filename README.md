# 有道云笔记 × Hermes Agent 适配 Skill

两个 Skill 文件，把有道云笔记完整接入 Hermes Agent。

## 前置依赖

在安装 Skill 之前，请确保：

```bash
# 1. 安装 youdaonote CLI（≥ 1.3.3）
curl -fsSL https://artifact.lx.netease.com/download/youdaonote-cli/install.sh | bash -s -- -f -b ~/.local/bin
export PATH="$HOME/.local/bin:$PATH"

# 2. 获取并配置 API Key
# 访问 https://mopen.163.com，手机号登录获取 Key
youdaonote config set apiKey <你的Key>

# 3. 验证连通性
youdaonote -s ydn list
# 正常应返回「我的资源」文件夹
```

## 安装 Skill

### 如果你用默认 profile

```bash
# 基础 Skill
mkdir -p ~/.hermes/skills/youdaonote/
curl -o ~/.hermes/skills/youdaonote/SKILL.md \
  https://raw.githubusercontent.com/woaixuziwei0527/youdaonote-hermes-skills/main/youdaonote-hermes.md

# Wiki Skill
mkdir -p ~/.hermes/skills/youdaonote-llm-wiki/
curl -o ~/.hermes/skills/youdaonote-llm-wiki/SKILL.md \
  https://raw.githubusercontent.com/woaixuziwei0527/youdaonote-hermes-skills/main/youdaonote-llm-wiki-hermes.md
```

### 如果你用自定义 profile（把 `YOUR_PROFILE` 换掉）

```bash
mkdir -p ~/.hermes/profiles/YOUR_PROFILE/skills/youdaonote/
curl -o ~/.hermes/profiles/YOUR_PROFILE/skills/youdaonote/SKILL.md \
  https://raw.githubusercontent.com/woaixuziwei0527/youdaonote-hermes-skills/main/youdaonote-hermes.md

mkdir -p ~/.hermes/profiles/YOUR_PROFILE/skills/youdaonote-llm-wiki/
curl -o ~/.hermes/profiles/YOUR_PROFILE/skills/youdaonote-llm-wiki/SKILL.md \
  https://raw.githubusercontent.com/woaixuziwei0527/youdaonote-hermes-skills/main/youdaonote-llm-wiki-hermes.md
```

### 验证安装

```bash
hermes skills list | grep youdaonote
```

预期看到 `youdaonote` 和 `youdaonote-llm-wiki` 均为 enabled。

## 文件说明

| 文件 | Skill 名 | 用途 |
|------|---------|------|
| `youdaonote-hermes.md` | `youdaonote` | 笔记 CRUD、待办管理、网页剪藏 |
| `youdaonote-llm-wiki-hermes.md` | `youdaonote-llm-wiki` | 结构化知识库（Ingest / Query / Lint / Archive） |

## 开始使用

在 Hermes 对话中直接说：

**基础操作：**
- 「列出我有道云笔记中的笔记」
- 「创建一篇笔记，标题是周报」
- 「搜索笔记里关于 AI 的内容」
- 「剪藏这个网页：https://xxx」

**Wiki 知识库：**
- 「帮我创建一个 AI 研究的知识库」
- 「把这篇文章整合进 ai-wiki：https://xxx」
- 「在 ai-wiki 里查询 Transformer 和 Mamba 的对比」
- 「审计一下 ai-wiki」

## 适配说明

这两个 Skill 文件从有道云笔记官方版本（Cursor/Claude Code/OpenCode 用）适配到 Hermes。主要改动：

- 文件写入工具：`Write 工具` → Hermes `write_file`
- 网页抓取工具：`web_fetch` → Hermes `browser_navigate`
- CLI 命令执行：统一使用 Hermes `terminal` 工具
- contentFile 模式：明确 `write_file` 写临时文件 + `terminal` 执行 save

## 相关链接

- [有道云笔记 Skill 安装指南（官方）](https://note.youdao.com/help-center/skill-install-guide.html)
- [LLM Wiki 快速上手（官方）](https://note.youdao.com/help-center/skill-wiki-guide.html)
- [Hermes Agent 项目](https://github.com/NousResearch/hermes-agent)
