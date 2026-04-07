# Claude Agent SDK 探索指南

## 这是什么

[Claude Agent SDK](https://platform.claude.com/docs/zh-CN/agent-sdk/quickstart) 是 Anthropic 官方提供的 TypeScript SDK，允许你通过代码调用 Claude Code 的智能体能力（工具调用、文件读写、代码编辑等）。

核心原理：SDK 并不直接调用 API，而是启动一个 **Claude Code 子进程**，通过 stdin/stdout 进行 JSON 流式通信。这意味着 Claude Code 是必须的运行时依赖。

## 前置条件

| 依赖 | 要求 |
|------|------|
| Node.js | >= 20.6（需要 `--env-file` 支持） |
| Claude Code | 已安装（`npm install -g @anthropic-ai/claude-code`） |

## 快速开始

### 1. 初始化项目

```bash
mkdir my-agent && cd my-agent
npm init -y
npm install @anthropic-ai/claude-agent-sdk
```

### 2. 设置 ESM 模式

在 `package.json` 中添加 `"type": "module"`，否则 top-level await 会报错：

```
ERROR: Top-level await is currently not supported with the "cjs" output format
```

### 3. 编写 Agent

```typescript
// agent.ts
import { query } from "@anthropic-ai/claude-agent-sdk";

for await (const message of query({
  prompt: "用中文介绍一下当前目录的项目结构",
  options: {
    allowedTools: ["Read", "Edit", "Glob"],
    permissionMode: "acceptEdits"
  }
})) {
  if (message.type === "assistant" && message.message?.content) {
    for (const block of message.message.content) {
      if ("text" in block) console.log(block.text);
      else if ("name" in block) console.log(`Tool: ${block.name}`);
    }
  } else if (message.type === "result") {
    console.log(`Done: ${message.subtype}`);
  }
}
```

### 4. 配置环境变量并运行

```bash
npx tsx --env-file=.env agent.ts
```

## 接入国产大模型

这是本次探索最重要的收获之一。Agent SDK 底层走 Claude Code，而 Claude Code 支持自定义 API 端点，因此可以接入兼容 Anthropic API 格式的国产大模型。

### 关键区别：两种认证方式

| 场景 | 环境变量 | 说明 |
|------|----------|------|
| Anthropic 官方 | `ANTHROPIC_API_KEY` | 直接使用官方密钥 |
| 国产大模型 | `ANTHROPIC_AUTH_TOKEN` | **必须用这个**，不能用 `ANTHROPIC_API_KEY` |

> 这是一个容易踩的坑：如果对国产大模型使用 `ANTHROPIC_API_KEY`，Claude Code 子进程会卡住无任何输出，也不会报错。必须改用 `ANTHROPIC_AUTH_TOKEN`。

### DeepSeek 接入示例

创建 `.deepseek_env`：

```env
ANTHROPIC_BASE_URL=https://api.deepseek.com/anthropic
ANTHROPIC_AUTH_TOKEN=sk-xxx  # 替换为你的 DeepSeek API Key
```

编写 `agent_deepseek.ts`：

```typescript
import { query } from "@anthropic-ai/claude-agent-sdk";

for await (const message of query({
  prompt: "你好",
  options: {
    allowedTools: ["Read", "Edit", "Glob"],
    permissionMode: "acceptEdits",
    model: "deepseek-chat"         // 指定 DeepSeek 模型
  }
})) {
  if (message.type === "assistant" && message.message?.content) {
    for (const block of message.message.content) {
      if ("text" in block) console.log(block.text);
      else if ("name" in block) console.log(`Tool: ${block.name}`);
    }
  } else if (message.type === "result") {
    console.log(`Done: ${message.subtype}`);
  }
}
```

运行：

```bash
npx tsx --env-file=.deepseek_env agent_deepseek.ts
```

### 不使用 Claude Code 的 settings.json

默认情况下，Agent SDK 启动的 Claude Code 子进程会读取 `~/.claude/settings.json` 中的配置（包括 API Key）。如果你不想使用 settings.json 中的配置，而是完全通过 `.env` 文件控制，需要注意：

- 通过 `--env-file` 加载的环境变量会 **覆盖** settings.json 中的同名配置
- `ANTHROPIC_BASE_URL` + `ANTHROPIC_AUTH_TOKEN` 组合会让 Claude Code 跳过自身的认证流程，直接使用你提供的端点和密钥
- 这意味着你不需要登录 Claude 官方账号，也不需要在 settings.json 中做任何配置

## 踩坑记录

### 1. package.json 缺少 `"type": "module"`

`tsx` 默认用 CJS 格式，不支持 top-level `await`。加上 `"type": "module"` 即可。

### 2. SDK 必须安装在项目本地

即使全局安装了 `@anthropic-ai/claude-agent-sdk`，Node.js 也找不到。必须在项目目录下 `npm install`，或者用 `npm link` 链接。

### 3. Node.js 不会自动加载 .env

必须用 `npx tsx --env-file=.env agent.ts` 显式加载。

### 4. options.env 会覆盖整个环境

如果在代码中通过 `options.env` 传递环境变量：

```typescript
// 错误写法 - 会丢失 PATH 等系统变量，导致子进程找不到 node
env: {
  ANTHROPIC_BASE_URL: "...",
  ANTHROPIC_API_KEY: "..."
}

// 正确写法 - 保留系统环境变量
env: {
  ...process.env,
  ANTHROPIC_BASE_URL: "...",
  ANTHROPIC_API_KEY: "..."
}
```

推荐直接用 `--env-file` 而不是在代码里写 `env`。

### 5. 国产大模型必须用 ANTHROPIC_AUTH_TOKEN

用 `ANTHROPIC_API_KEY` 接入国产大模型会导致进程卡死无输出。这是因为 Claude Code 对 `ANTHROPIC_API_KEY` 有额外的认证校验逻辑，而 `ANTHROPIC_AUTH_TOKEN` 会直接透传到请求头中。

## 相关资源

- [Claude Agent SDK 官方文档（中文）](https://platform.claude.com/docs/zh-CN/agent-sdk/quickstart)
- [Claude Agent SDK 实战 Demo（liruifengv）](https://liruifengv.com/posts/cladue-agent-sdk-demo/)
- [Claude Code 官方文档](https://docs.anthropic.com/en/docs/claude-code)
