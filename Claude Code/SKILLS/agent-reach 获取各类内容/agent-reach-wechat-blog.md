# 用 Agent-Reach 让 Claude Code 读懂微信公众号文章

## 痛点：AI Agent 读不了微信公众号

用 Claude Code 做信息收集时，读网页内容很方便——`curl` 一下 Jina Reader 就行。但微信公众号文章是个例外：

- `curl` 直接抓取？返回的是空壳 HTML，正文内容靠 JS 动态渲染
- Jina Reader？触发微信验证码，返回"环境异常"
- 传统爬虫？被微信反爬机制拦截

这意味着，当你让 Claude Code "帮我总结这篇公众号文章"时，它根本拿不到内容。

**Agent-Reach 解决了这个问题。**

## Agent-Reach 是什么

[Agent-Reach](https://github.com/Panniantong/Agent-Reach) 是一个开源 CLI 工具，核心理念是"脚手架而非框架"——它不替代 AI Agent，而是帮 Agent 装上"眼睛"，让它能看到互联网上的内容。

支持 14 个平台：Twitter/X、YouTube、B站、小红书、Reddit、GitHub、微博、抖音、微信公众号、LinkedIn 等，全部免费、零 API 费用。

安装只需一行：

```bash
pipx install git+https://github.com/Panniantong/agent-reach.git
agent-reach install --env=auto
```

装完用 `agent-reach doctor` 检查各渠道状态。

## 微信公众号：从安装到实际使用

### 原理

Agent-Reach 的微信公众号方案分两层：

| 功能 | 工具 | 原理 |
|------|------|------|
| 搜索文章 | miku_ai | 聚合搜索引擎，关键词检索公众号文章列表 |
| 阅读文章 | wechat-article-for-ai + Camoufox | 用反检测浏览器渲染页面，提取正文转 Markdown |

关键在于 **Camoufox**——它是 Firefox 的一个反指纹检测分支，能绕过微信的反爬机制，像真人浏览器一样访问文章页面，拿到完整的渲染后内容。

### 安装微信渠道

Agent-Reach 安装时会自动部署 wechat-article-for-ai 工具到 `~/.agent-reach/tools/` 目录，但 Python 依赖需要额外安装：

```bash
# 创建独立的 Python 虚拟环境（推荐方式，避免系统 Python 依赖冲突）
python3 -m venv ~/.agent-reach/tools/wechat-article-for-ai/.venv

# 安装依赖
~/.agent-reach/tools/wechat-article-for-ai/.venv/bin/pip install \
  'camoufox[geoip]' markdownify beautifulsoup4 httpx mcp miku_ai
```

首次运行时 Camoufox 会自动下载浏览器内核（约 300MB），之后就是秒启动。

### 实际使用

**读取单篇文章：**

```bash
~/.agent-reach/tools/wechat-article-for-ai/.venv/bin/python \
  ~/.agent-reach/tools/wechat-article-for-ai/main.py \
  "https://mp.weixin.qq.com/s/xxxxx" \
  -o /tmp/wechat-output --no-images
```

**搜索公众号文章：**

```python
~/.agent-reach/tools/wechat-article-for-ai/.venv/bin/python -c "
import asyncio
from miku_ai import get_wexin_article
async def s():
    for a in await get_wexin_article('AI Agent', 5):
        print(f'{a[\"title\"]} | {a[\"url\"]}')
asyncio.run(s())
"
```

**批量处理：**

```bash
# 把多个 URL 写进文件，一次性处理
echo "https://mp.weixin.qq.com/s/aaa
https://mp.weixin.qq.com/s/bbb" > urls.txt

~/.agent-reach/tools/wechat-article-for-ai/.venv/bin/python \
  ~/.agent-reach/tools/wechat-article-for-ai/main.py \
  -f urls.txt -o /tmp/wechat-output --no-images
```

### 输出效果

转换后的 Markdown 包含完整的 YAML frontmatter：

```markdown
---
title: 文章标题
author: 公众号名称
date: "2026-03-11 17:55:14"
source: "https://mp.weixin.qq.com/s/xxxxx"
---

# 文章标题

正文内容，保留原始格式、链接、代码块...
```

图片默认下载到本地 `images/` 子目录并替换为本地路径，加 `--no-images` 则保留远程 URL，节省时间和空间。

## 与 Claude Code 集成

Agent-Reach 安装后会自动在 `~/.claude/skills/` 注册 Skill 文件。这意味着在 Claude Code 中，你可以直接说：

> "帮我总结这篇微信公众号文章 https://mp.weixin.qq.com/s/xxxxx"

Claude Code 会自动调用 wechat-article-for-ai 抓取文章、转为 Markdown、然后进行总结。整个过程大约 10-15 秒。

如果你想让它在**新会话**中也能正常工作，关键是确保 SKILL.md 中的命令指向正确的 Python 路径（即 `.venv/bin/python` 的完整路径），而不是系统的 `python3`——因为依赖装在虚拟环境里。

## 踩过的坑

在实际安装过程中遇到了几个问题，记录下来供参考：

### 1. Python 版本要求

Agent-Reach 要求 Python >= 3.10。macOS 系统自带的 Python 3.9 不行，Homebrew 的 Python 又不允许直接 `pip install`。解决方案是用 `pipx`：

```bash
brew install pipx
pipx install git+https://github.com/Panniantong/agent-reach.git
```

### 2. Camoufox UBO 插件下载失败

Camoufox 默认会下载 uBlock Origin 插件，但 Mozilla 的 CDN 可能返回 451 错误。这会导致创建一个空的 addon 目录，后续每次启动都报 `manifest.json is missing` 错误。

修复方法是修改 `camoufox/addons.py` 中的 `confirm_paths` 函数，把 `raise InvalidAddonPath` 改为静默跳过无效路径：

```python
def confirm_paths(paths):
    invalid = []
    for path in paths:
        if not os.path.isdir(path) or not os.path.exists(os.path.join(path, 'manifest.json')):
            invalid.append(path)
    for path in invalid:
        paths.remove(path)
```

### 3. agent-reach 自身的 bug

安装脚本中有个 `config` 变量未定义的 bug（在检查 Groq API Key 时），需要手动加一行 `from agent_reach.config import Config; config = Config()`。开源项目的常态，遇到了直接看报错改就行。

### 4. 新会话找不到依赖

如果把依赖装在 pipx 的 venv 里，新的 Claude Code 会话用 `python3` 调用时会报 `ModuleNotFoundError`。解决方案是在工具目录下创建独立的 `.venv`，然后更新 SKILL.md 中的命令路径。

## 总结

Agent-Reach 解决了一个很实际的问题：**让 AI Agent 能读到微信公众号的内容**。在此之前，这几乎是不可能的——微信的封闭生态和反爬机制把所有常规方法都挡在了门外。

通过 Camoufox 反检测浏览器 + miku_ai 搜索引擎的组合，Agent-Reach 提供了一套完整的"搜索 + 阅读"方案，而且完全免费、本地运行、不依赖任何付费 API。

如果你经常需要让 Claude Code 处理公众号文章——无论是信息收集、内容总结、还是竞品分析——Agent-Reach 值得一装。
