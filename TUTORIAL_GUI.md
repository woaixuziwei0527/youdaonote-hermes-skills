# 有道云笔记接入 Hermes 桌面客户端：一句命令搞定

这篇教程写给正在用 Hermes 桌面客户端的用户。不需要打开终端，不需要懂命令行。

全程只在 Hermes 聊天框里说话，加去网页拿一次 Key，5 分钟搞定。装完之后所有操作都用中文对话完成。

---

## 第一步：让 Hermes 帮你装好一切

打开 Hermes 桌面客户端，在聊天框里输入下面这句话（整段复制，发送）：

> 帮我做三件事：
> 1. 下载安装 youdaonote CLI：执行 `curl -fsSL https://artifact.lx.netease.com/download/youdaonote-cli/install.sh | bash -s -- -f -b ~/.local/bin`，然后把 `export PATH="$HOME/.local/bin:$PATH"` 追加到 ~/.zshrc
> 2. 验证 CLI 装好了：在终端执行 `export PATH="$HOME/.local/bin:$PATH" && youdaonote --help`
> 3. 下载有道云笔记的 Hermes Skill 文件，放到 ~/.hermes/skills/youdaonote/SKILL.md 和 ~/.hermes/skills/youdaonote-llm-wiki/SKILL.md，下载地址分别是：
>    https://raw.githubusercontent.com/weiweiplus0527/youdaonote-hermes-skills/main/youdaonote-hermes.md
>    https://raw.githubusercontent.com/weiweiplus0527/youdaonote-hermes-skills/main/youdaonote-llm-wiki-hermes.md

Hermes 收到后会自己打开终端、下载、配置、验证。你看着屏幕等它完成就行。看到「安装完成」和 help 命令输出一大串内容，说明第一步成功了。

> **如果 Hermes 卡住或报错：** 别慌。关掉 Hermes，手动操作——按 `Command + 空格`，输入「终端」回车，然后复制执行以下命令，一条一条来：
> ```bash
> curl -fsSL https://artifact.lx.netease.com/download/youdaonote-cli/install.sh | bash -s -- -f -b ~/.local/bin
> echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.zshrc && source ~/.zshrc
> ```
> 验证：`youdaonote --help` 弹出一堆命令说明即可。
> 然后继续下一步。

---

## 第二步：获取 API Key

API Key 是你有道云笔记的钥匙，让 Hermes 有权限读写你的笔记。

用浏览器打开：**https://mopen.163.com**

用手机号登录。注意：**这个手机号必须是你有道云笔记 App 里绑定的那个**。如果没绑过，先去有道云笔记 App「设置」里绑定。

登录后找到「API Key」页面（通常左侧菜单），创建或复制你的 Key。它看起来像一长串字符——`iv1gFXkvKdzb...` 这样。复制。

回到 Hermes 聊天框，输入：

> 配置我的 youdaonote API Key：在终端执行 `youdaonote config set apiKey 你的Key粘贴在这里`

比如：

> 配置我的 youdaonote API Key：在终端执行 `youdaonote config set apiKey iv1gFXkvKdzbKOjUdsTPP0mBmEx_7lEb`

Hermes 执行后，让它验证一下。输入：

> 测试一下 my youdaonote 是否连通：执行 `youdaonote list`

看到 `📁 我的资源` 就对了。

> **如果 Hermes 执行超时：** 打开终端手动输入 `youdaonote config set apiKey 你的Key`，然后 `youdaonote list` 验证。

---

## 第三步：重启 Hermes，开用

关掉 Hermes 桌面客户端，重新打开。

然后在聊天框里说下面这些话试试：

**日常笔记：**

| 你可以说 | Hermes 做什么 |
|---------|-------------|
| 「列出我的笔记」 | 显示你有道云笔记里全部内容 |
| 「创建一个笔记，标题是本周周报」 | 新建一篇笔记 |
| 「搜一下笔记里关于 AI 的内容」 | 按关键字搜索 |
| 「剪藏这个网页到我的笔记：https://xxx」 | 保存网页到有道云笔记 |
| 「创建一个待办：周五前交报告」 | 添加待办 |

**Wiki 知识库：**

| 你可以说 | Hermes 做什么 |
|---------|-------------|
| 「帮我建一个 AI 研究知识库」 | 自动建文件夹结构和系统笔记 |
| 「把这篇文章加进 AI 知识库：https://xxx」 | 拆实体、建概念页、加交叉引用 |
| 「在 AI 知识库里查一下 Transformer 原理」 | 跨多页面综合分析 |
| 「记下来」 | 把当前聊天结论归档到知识库 |

---

## 如果遇到问题

**「找不到 youdaonote 命令」**

关掉 Hermes，重新打开。还不行的话打开终端，输入 `which youdaonote` 看看有没有结果。没有的话手动执行第一步里的备选命令。

**「API Key 无效」**

去 https://mopen.163.com 确认手机号绑定了有道云笔记，重新获取 Key，然后告诉 Hermes「在终端执行 youdaonote config set apiKey 新Key」。

**「没有 Skill」**

在 Hermes 里说「检查一下 ~/.hermes/skills/ 目录里有没有 youdaonote 文件夹」。没有的话，重复第一步的第 3 项，让 Hermes 重新下载。

---

## 原理（好奇的话看一眼，不看也行）

```
你在 Hermes 说「帮我存笔记」
      ↓
Hermes 读 Skill 手册 → 知道用 youdaonote save 命令
      ↓
Hermes 打开终端执行命令
      ↓
CLI 用你的 API Key 敲门 → 有道云笔记云端执行
      ↓
结果返回你面前
```

你只负责说中文。中间这些步骤 Hermes 全自动。
