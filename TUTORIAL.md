# 有道云笔记接入 Hermes：从零到能用（纯小白版）

这篇教程写给没有编程背景、没用过命令行、不知道 Skill 是什么的用户。每一步都有截图式文字指引，跟着做就行，不需要理解原理。

完成后，你可以在 Hermes 对话里直接说「列出我的笔记」「帮我存一篇周报」「建一个 AI 研究知识库」，Agent 会自动操作你的有道云笔记。

---

## 整体流程

一共四步，预计 10 分钟：

```
安装 CLI 工具 → 获取 API Key → 下载 Skill 文件 → 开始使用
```

---

## 第一步：安装 youdaonote CLI

CLI 是一个命令行工具，可以理解成一个「桥梁」——Hermes 通过它读写有道云笔记。先装桥，再走人。

打开电脑上的「终端」（Terminal）应用。找不到的话，按 `Command + 空格`，输入「终端」回车。

复制下面这行命令，粘贴到终端窗口里，按回车：

```bash
curl -fsSL https://artifact.lx.netease.com/download/youdaonote-cli/install.sh | bash -s -- -f -b ~/.local/bin
```

等几秒，看到 `✓ 安装完成` 就说明装好了。

接下来把工具的路径告诉系统，让系统能找到它。在终端里依次执行这两条命令：

```bash
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc
```

验证是否装好。在终端输入：

```bash
youdaonote --help
```

如果弹出一堆命令说明（list、read、create 等），恭喜，第一步完成。

> 如果提示 `command not found`，说明路径没生效。关掉终端重新打开，再试一次。

---

## 第二步：获取 API Key

API Key 是一串密码，让 CLI 工具有权限访问你的有道云笔记。从有道云笔记的官方开发者平台获取。

用浏览器打开这个网址：

**https://mopen.163.com**

页面会让你登录。注意：登录用的手机号，必须是你在有道云笔记 App 里绑定的那个手机号。如果没绑过，先去有道云笔记 App 里绑定。

登录后，在控制台页面找到「API Key」或「密钥」相关的选项（通常在左侧菜单），点击「创建」或「查看」，把生成的那串字符复制下来。看起来类似这样：`iv1gFXkvKdzb...`（很长一串）。

回到终端，执行以下命令，把刚才复制的 Key 填进去：

```bash
youdaonote config set apiKey 你的Key粘贴在这里
```

比如：

```bash
youdaonote config set apiKey iv1gFXkvKdzbKOjUdsTPP0mBmEx_7lEb
```

验证连通性。在终端输入：

```bash
youdaonote list
```

如果返回 `📁 我的资源`，说明 Key 正确，桥通了。

> 常见的报错是「API Key 无效」。检查两个地方：Key 有没有多复制空格？登录的手机号是不是绑定过有道云笔记？

---

## 第三步：安装两个 Skill 文件到 Hermes

Skill 是给 Hermes 看的「操作手册」。装完这两个文件，Hermes 就知道怎么操作有道云笔记了。一个叫基础 Skill（存笔记、搜笔记、管待办），一个叫 Wiki Skill（建知识库）。

复制以下整段命令，粘贴到终端，按回车：

```bash
# 基础 Skill（笔记操作）
mkdir -p ~/.hermes/skills/youdaonote/
curl -o ~/.hermes/skills/youdaonote/SKILL.md \
  https://raw.githubusercontent.com/woaixuziwei0527/youdaonote-hermes-skills/main/youdaonote-hermes.md

# Wiki Skill（知识库）
mkdir -p ~/.hermes/skills/youdaonote-llm-wiki/
curl -o ~/.hermes/skills/youdaonote-llm-wiki/SKILL.md \
  https://raw.githubusercontent.com/woaixuziwei0527/youdaonote-hermes-skills/main/youdaonote-llm-wiki-hermes.md
```

等几秒，没有报错就是装好了。

验证。在终端输入：

```bash
hermes skills list | grep youdaonote
```

应该看到两行：

```
youdaonote            │ note-taking          │ enabled
youdaonote-llm-wiki                          │ enabled
```

> 如果你用的是 Hermes 的自定义 profile（比如叫 `work`），把上面命令里的 `~/.hermes/skills/` 换成 `~/.hermes/profiles/work/skills/`。不确定自己用没用 profile 的话，就不用换——默认没改过就是不用换。

---

## 第四步：开始使用

打开 Hermes，在对话里直接说下面这些话试试：

**基础操作：**

| 你说的话 | 会发生什么 |
|---------|-----------|
| 「列出我的笔记」 | Hermes 展示你有道云笔记里的内容 |
| 「搜索笔记里关于 AI 的内容」 | 按关键字搜索 |
| 「创建一个笔记，标题是周报 7/10，内容是本周完成事项…」 | 新建笔记 |
| 「剪藏这个网页：https://xxx」 | 把网页内容保存到有道云笔记 |
| 「创建一个待办：周五前交报告」 | 添加待办 |

**Wiki 知识库：**

| 你说的话 | 会发生什么 |
|---------|-----------|
| 「帮我创建一个 AI 研究的知识库」 | Hermes 建文件夹结构 + 系统笔记 |
| 「把这篇文章整合进 ai-wiki」+ 贴 URL | 自动拆实体、建概念页、加交叉引用 |
| 「在 ai-wiki 里查询 Transformer 的原理」 | 跨多页面综合分析 |
| 「记下来」 | 把当前对话结论归档到知识库 |
| 「审计一下 ai-wiki」 | 检查孤立页面和矛盾 |

---

## 安装中遇到问题的排查顺序

1. 检查 CLI 版本：终端输入 `youdaonote version`，确认 ≥ 1.3.3
2. 检查连通性：终端输入 `youdaonote list`，确认有返回结果
3. 检查 Skill：终端输入 `hermes skills list | grep youdaonote`，确认两行都是 enabled
4. 如果前三步都对但 Hermes 里说「找不到命令」，重启 Hermes（`/reset`），或关掉重开

---

## 常见问题

**Q：为什么装了两个 Skill？只装一个行不行？**

基础 Skill 管单篇笔记的日常操作（存、查、删、待办），Wiki Skill 管知识库建设。可以只装基础 Skill，Wiki Skill 是可选的。但 Wiki Skill 依赖基础 Skill 的 CLI，装了 Wiki 就必须两个都装。

**Q：我用的不是 Hermes 是 Cursor / Claude Code？**

这两个 Skill 文件是专门给 Hermes 适配的。如果你用的是 Cursor 或 Claude Code，去有道云笔记帮助中心下载官方原版：https://note.youdao.com/help-center/skill-install-guide.html

**Q：API Key 安全吗？会不会泄露？**

Key 只存在你电脑本地（`~/.youdaonote.json` 文件），不会经 Hermes 上传到任何服务器。所有笔记操作都在你电脑本地执行。

**Q：macOS / Windows / Linux 都能用吗？**

都能。上述命令在 macOS 和 Linux 上通用。Windows 用户需要在 PowerShell 里执行，第一步的安装命令不同，参考有道云笔记 CLI 安装指南：https://note.youdao.com/help-center/cli-install-guide.html

**Q：更新 Skill 怎么操作？**

重新执行第三步的命令即可，会覆盖旧文件。
