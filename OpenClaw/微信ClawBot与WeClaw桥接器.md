# 微信 ClawBot 与 WeClaw桥接器

> 3 月 22 日，微信正式支持通过插件接入 OpenClaw。

---

## 一、微信 ClawBot 接入 OpenClaw

### 安装步骤

**1. 开启微信插件入口**

进入微信 → **我 → 设置 → 插件 → 微信 ClawBot**

**2. 在安装了 OpenClaw 的主机上执行安装命令**

```bash
npx -y @tencent-weixin/openclaw-weixin-cli@latest install
```

> 相关 npm 包：https://npmx.dev/package/@tencent-weixin/openclaw-weixin

**3. 安装成功后，重启 Gateway**

```
Installed plugin: openclaw-weixin
Restart the gateway to load plugins.
[openclaw-weixin] 插件就绪，开始首次连接...
```

**4. 扫码登录微信**

```
正在启动微信扫码登录...
使用微信扫描以下二维码，以完成连接：
```

**5. 扫码成功，等待连接**

```
✅ 与微信连接成功！
[openclaw-weixin] 正在重启 OpenClaw Gateway...
```

---

## 二、OpenClaw 使用

接入成功后，使用方式与在飞书中使用 OpenClaw 完全一致：

- 支持 `/status`、`/models` 等常用命令
- 支持发送**图片**和**文件**（插件内部已基于 OpenClaw 协议完成处理）

---

## 三、原理说明

微信 ClawBot 接入 OpenClaw 的原理，与 Telegram 接入 OpenClaw 本质上一致：

1. 微信实现了一个云端中继服务 **iLink**
2. 微信实现了一个 OpenClaw 插件 **openclaw-weixin**
3. 用户在安装了 OpenClaw 的机器上安装 `openclaw-weixin` 插件，插件请求 iLink 服务，引导用户扫码登录，登录凭证保存在 OpenClaw 配置目录
4. `openclaw-weixin` 通过**长轮询**拉取 iLink 消息，转发给 OpenClaw Gateway，调用 Agent Runtime 执行任务（对应用户在微信 ClawBot 发消息）
5. `openclaw-weixin` 请求 iLink，将任务结果推送给用户（对应用户在微信 ClawBot 收到回复）

### 架构图

```
微信用户 → 腾讯 iLink 云服务 → OpenClaw 插件（长轮询）→ OpenClaw Gateway → AI 模型
```

![架构示意图](1.jpeg)

---

## 四、WeClaw

> 项目地址：https://github.com/fastclaw-ai/weclaw

微信提供了入口，WeClaw 对这个入口进行桥接，让你可以直接通过微信与 AI Agent 交互。

![WeClaw 示意图](2-1.png)

### 安装

```bash
# 一键安装
curl -sSL https://raw.githubusercontent.com/fastclaw-ai/weclaw/main/install.sh | sh

# 启动（首次运行会弹出微信扫码登录）
weclaw start
```

配置文件路径：`~/.weclaw/config.json`

### 常用命令

| 命令 | 说明 |
|------|------|
| `你好` | 发送给默认 Agent |
| `/codex 写一个排序函数` | 发送给指定 Agent |
| `/cc 解释一下这段代码` | 通过别名发送 |
| `/claude` 或 `/codex` | 切换默认 Agent |
| `/status` | 查看当前 Agent 信息 |
| `/help` | 查看帮助信息 |

---

## 五、主动推送消息

WeClaw 支持**反向推送**，无需等待用户发消息，可主动向微信用户推送内容。

**命令行方式：**

```bash
weclaw send --to "user_id@im.wechat" --text "你好，来自 weclaw"
```

**HTTP API 方式**（`weclaw start` 运行时，默认监听 `127.0.0.1:18011`）：

```bash
curl -X POST http://127.0.0.1:18011/api/send \
  -H "Content-Type: application/json" \
  -d '{"to": "user_id@im.wechat", "text": "你好，来自 weclaw"}'
```

![推送示意图](3.jpg)
