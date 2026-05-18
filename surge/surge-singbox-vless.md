# Surge 接入 VLESS(借助 sing-box,通过 External Proxy Program 机制)

> 目标:让 Surge 使用一个 VLESS+Reality 节点。由于 Surge 原生不支持 VLESS,通过 Surge 的 External Proxy Program(EPP)拉起 sing-box 作为协议适配层,Surge 通过本地 SOCKS5 与 sing-box 通信。
>
> 测试环境:macOS(Apple Silicon)+ Surge Mac + sing-box 1.13.x。

---

## 一、原理

### 1.1 为什么需要这套方案

- Surge 内置支持 SS / Trojan / Snell / Vmess / Hysteria2 等,**不支持 VLESS+Reality / Vision**。
- sing-box 原生支持 VLESS+Reality+XTLS Vision,但单独运行时无法被 Surge 的规则系统统一调度。
- Surge 的 **External Proxy Program(EPP)** 机制可以把任意外部代理程序当成 Surge 的一个节点使用,只要该程序能在本地暴露 SOCKS5。

### 1.2 EPP 工作原理

```
浏览器 / 系统流量
   │
   ▼
┌─────────────────────────────────────────┐
│  Surge(规则匹配、Proxy Group 选择)         │
│  对外:HTTP 6152 / SOCKS5 6153              │
└─────────────────────────────────────────┘
   │ 命中"EPP 节点"
   │ 以 SOCKS5 协议交付给本地端口(本文用 7890)
   ▼
┌─────────────────────────────────────────┐
│  sing-box(Surge 拉起的子进程)             │
│   inbound:  socks 127.0.0.1:7890        │
│   outbound: vless + reality + vision    │
└─────────────────────────────────────────┘
   │ TLS + Reality 伪装
   ▼
远端 VLESS 服务器
```

关键性质:

1. **父子进程关系**:Surge 用 `posix_spawn` 拉起 sing-box,sing-box 的 PPID 等于 Surge 的 PID。Surge 退出会向子进程发 SIGTERM,sing-box 跟着死。
2. **按需启动**:只有当连接命中 EPP 节点时,Surge 才启动 sing-box;长时间无连接会被 Surge 杀掉。
3. **职责分离**:Surge 负责"哪些流量走代理 / 走哪个节点",sing-box 只负责"把 SOCKS5 流量编码成 VLESS 协议发出去"。所以 sing-box 配置可以非常精简——不需要 DNS、不需要 route、不需要 geosite/geoip。
4. **DNS 闭环问题**:sing-box 解析服务器域名时,DNS 查询会被 Surge 拦截,而 Surge 又可能把查询发回 sing-box,形成死循环。EPP 提供 `addresses=` 字段直接告诉 Surge 该域名对应的 IP,避开 DNS 查询。

### 1.3 相关文档

- Surge 官方手册:<https://manual.nssurge.com/policy/external-proxy.html>
- Surge 文档(社区维护):<https://surge.mitsea.com/policy/external-proxy>
- Surge 社区讨论:<https://community.nssurge.com/d/600-surge-mac-external-proxy-provider>
- sing-box 官方文档:<https://sing-box.sagernet.org/>
- sing-box DNS 新格式迁移:<https://sing-box.sagernet.org/migration/>

---

## 二、sing-box 安装与配置

### 2.1 安装

macOS 推荐用 Homebrew:

```bash
brew install sing-box
brew info sing-box
```

`brew info` 输出会告诉你二进制和默认配置路径,本文以 Apple Silicon 为例:

| 项 | 路径 |
|---|---|
| 二进制 | `/opt/homebrew/opt/sing-box/bin/sing-box` |
| 默认配置 | `/opt/homebrew/etc/sing-box/config.json` |
| 工作目录 | `/opt/homebrew/var/lib/sing-box` |

Intel Mac 把 `/opt/homebrew` 替换为 `/usr/local`。

### 2.2 创建目录

brew 不会自动建,首次需要手动:

```bash
mkdir -p /opt/homebrew/etc/sing-box
mkdir -p /opt/homebrew/var/lib/sing-box
```

### 2.3 EPP 专用配置

**重要**:给 Surge 用的 sing-box 配置应该尽量精简——只包含一个 SOCKS5 入站和一个 VLESS 出站,**不要写 DNS / route / rule_set**,因为这些事 Surge 已经做了。

建议用单独文件名(例如 `surge-epp.json`)和 Surge 自己跑的配置区分。

`/opt/homebrew/etc/sing-box/surge-epp.json`:

```json
{
  "log": {
    "level": "warn",
    "timestamp": true
  },
  "inbounds": [
    {
      "type": "socks",
      "tag": "socks-in",
      "listen": "127.0.0.1",
      "listen_port": 7890,
      "sniff": false
    }
  ],
  "outbounds": [
    {
      "type": "vless",
      "tag": "vless-out",
      "server": "你的服务器域名",
      "server_port": 53918,
      "uuid": "你的UUID",
      "flow": "xtls-rprx-vision",
      "tls": {
        "enabled": true,
        "server_name": "www.microsoft.com",
        "utls": {
          "enabled": true,
          "fingerprint": "chrome"
        },
        "reality": {
          "enabled": true,
          "public_key": "服务端的 public_key",
          "short_id": "服务端的 short_id"
        }
      }
    }
  ]
}
```

### 2.4 把 VLESS 分享链接拆成配置字段

如果你拿到的是 `vless://...` 分享链接,字段对应关系如下:

```
vless://<UUID>@<server>:<port>?
  encryption=none
  &flow=xtls-rprx-vision              → outbounds.flow
  &fp=chrome                          → tls.utls.fingerprint
  &pbk=...                            → tls.reality.public_key
  &security=reality                   → tls.reality.enabled = true
  &sid=...                            → tls.reality.short_id
  &sni=www.microsoft.com              → tls.server_name
  &spx=...                            → (可忽略,sing-box 客户端不用)
  &type=tcp                           → (TCP 直连,无需 transport 段)
#备注
```

### 2.5 校验配置

```bash
sing-box check -c /opt/homebrew/etc/sing-box/surge-epp.json
```

无输出即通过。常见报错:

- `legacy DNS servers is deprecated` —— sing-box 1.12+ 已废弃旧 DNS 格式,EPP 场景下根本不该有 `dns` 段,删掉。
- `missing route.default_domain_resolver` —— 同上,EPP 配置不要 `dns/route` 段就能避免。

### 2.6 独立测试(脱离 Surge)

先确认配置本身能跑通,再接 Surge:

```bash
# 前台启动 sing-box
/opt/homebrew/opt/sing-box/bin/sing-box run \
  -c /opt/homebrew/etc/sing-box/surge-epp.json \
  -D /opt/homebrew/var/lib/sing-box

# 另开终端,通过 SOCKS5 访问外网,确认 IP 切换到节点
curl --socks5-hostname 127.0.0.1:7890 https://api.ipify.org && echo
curl --socks5-hostname 127.0.0.1:7890 -o /dev/null -w "Google: HTTP %{http_code}\n" https://www.google.com
```

输出 IP 应该是节点出口 IP(不是你本地公网 IP),Google 应该 200。

测试结束后 Ctrl+C 停掉,**不要用 `brew services start sing-box`**——后续 Surge 会自动管理生命周期,brew service 会和 Surge 抢端口。

---

## 三、Surge 配置修改

### 3.1 备份

```bash
cp ~/Library/Application\ Support/Surge/Profiles/你的配置.conf \
   ~/Library/Application\ Support/Surge/Profiles/你的配置.conf.bak.$(date +%Y%m%d_%H%M%S)
```

### 3.2 解析服务器 IP

`addresses=` 字段必填,否则会触发 DNS 闭环。提前解析:

```bash
dig +short 你的服务器域名 @1.1.1.1
# 例:xx.xx.xx.xx
```

如果服务商有多 IP,可以全部填上(逗号分隔)。

### 3.3 在 `[Proxy]` 段添加 EPP 节点

⚠️ **最重要的语法细节**:Surge 的 `args` 字段必须**每个参数独立声明一次**,**不能**用空格或逗号在单个字符串里合并多个参数,否则 Surge 会把整串当成一个 argv 传过去,sing-box 会立刻 `unknown command` 退出。

```ini
[Proxy]
# ... 你原有的节点 ...
🇺🇸 VLESS-R = external, exec = "/opt/homebrew/opt/sing-box/bin/sing-box", args = "run", args = "-c", args = "/opt/homebrew/etc/sing-box/surge-epp.json", args = "-D", args = "/opt/homebrew/var/lib/sing-box", local-port = 7890, addresses = xx.xx.xx.xx
```

字段说明:

| 字段 | 含义 | 注意 |
|---|---|---|
| `external` | 节点类型标识 | 固定写法 |
| `exec` | 可执行文件**绝对路径** | Surge GUI 进程 `$PATH` 不含 brew,必须绝对 |
| `args` | 单个命令行参数 | **每个参数独立 `args = "xxx"`,可重复** |
| `local-port` | 子进程监听的 SOCKS5 端口 | 必须和 sing-box inbound 端口一致 |
| `addresses` | 远端服务器 IP | 可重复,避开 DNS 闭环 |

### 3.4 加入 Proxy Group

把新节点加进你想要的选择/测速组,例如:

```ini
[Proxy Group]
🚀 节点选择 = select, "🇺🇸 VLESS-R", "其他节点A", "其他节点B", DIRECT
♻️ 自动选择 = url-test, "🇺🇸 VLESS-R", "其他节点A", "其他节点B", url=http://www.gstatic.com/generate_204, interval=300, tolerance=50
```

### 3.5 Reload Profile

Surge 菜单栏图标 → `Reload Profile`(或快捷键 `⌘R`)。

---

## 四、验证

### 4.1 在 Surge GUI 选中新节点

把 `🚀 节点选择` 切到 `🇺🇸 VLESS-R`,然后访问任意需要走代理的网站(如 `youtube.com`)触发流量。

### 4.2 命令行验证

```bash
# 1. sing-box 被拉起来,PPID 应该是 Surge.app
ps -o pid,ppid,command -p $(pgrep -f "sing-box run.*surge-epp")

# 2. 7890 端口被 sing-box 监听
lsof -nP -iTCP:7890 -sTCP:LISTEN

# 3. Surge 出口 IP 验证(Surge 默认 HTTP 6152)
curl -x http://127.0.0.1:6152 https://api.ipify.org && echo
# 期望输出节点出口 IP
```

如果 1、2、3 都对上,说明整条链路通了:**Surge → sing-box(子进程)→ VLESS 服务器**。

### 4.3 父子关系核对

```bash
# sing-box 的 PPID
pgrep -f "sing-box run.*surge-epp" | xargs -I{} ps -o ppid= -p {}

# Surge.app 的 PID
pgrep -f "MacOS/Surge$"
```

两个数字应该相同。

---

## 五、注意事项与排错

### 5.1 致命坑 ① — `args` 语法

**错误写法**(整串会被当成一个参数,sing-box 立刻退出):
```ini
args = "run,-c,/opt/homebrew/etc/sing-box/surge-epp.json,-D,/opt/homebrew/var/lib/sing-box"
```
**正确写法**:
```ini
args = "run", args = "-c", args = "/opt/homebrew/etc/sing-box/surge-epp.json", args = "-D", args = "/opt/homebrew/var/lib/sing-box"
```
症状是 Surge 日志里 `External proxy process terminated, reason: exit` 不停重启,`/tmp/Surge-External-<节点名>.log` 里 sing-box 输出 `unknown command "..."`。

### 5.2 致命坑 ② — `addresses=` 没填

如果不填 `addresses`,sing-box 解析服务器域名时:
- DNS 查询被 Surge 截获 → Surge 按规则把查询本身路由出去 → 可能又回到 sing-box → 死循环 / 超时。

**解决**:用 `dig` 或 `nslookup` 拿到 IP 后填进 `addresses`。多 IP 用逗号分隔。**服务商换 IP 后要同步改**。

### 5.3 致命坑 ③ — 端口冲突

如果你之前用 `brew services start sing-box` 让 sing-box 后台跑过,**必须停掉**:

```bash
brew services stop sing-box
brew services list | grep sing-box   # 确认 status 是 none
```

否则两份 sing-box 抢同一个端口,Surge 拉起的子进程会因 "address already in use" 立刻退出。

### 5.4 修改 sing-box 配置后如何生效

Surge 不监听 sing-box 配置文件的变化。改完 `surge-epp.json` 后:

- **方式 A**:Surge 里切到别的节点,再切回 VLESS-R(强制 kill+spawn)。
- **方式 B**:`⌘R` Reload Profile。
- **方式 C**:改 `[Proxy]` 行任意字段(Surge 检测到 EPP 定义变化,会重启子进程)。

直接重启 sing-box 是无效的——它一旦死亡 Surge 会立即重新拉起。

### 5.5 日志在哪

| 来源 | 位置 |
|---|---|
| Surge 主日志 | `~/Library/Logs/Surge/Surge-YYYY-MM-DD-HHMMSS.log` |
| sing-box stdout/stderr | `/tmp/Surge-External-<节点名>.log` |
| sing-box 自己的 log 段输出 | 也会进 `/tmp/Surge-External-<节点名>.log`(因为 stdout 被 Surge 接管) |

排错时先看 `/tmp/Surge-External-...log`——子进程的真实错误都在这里。

### 5.6 命令行使用代理的变化

以前如果你用 `export https_proxy=http://127.0.0.1:7890` 直连 sing-box,现在 7890 由 EPP 独占,命令行应该改走 Surge 出口:

```bash
export http_proxy=http://127.0.0.1:6152
export https_proxy=http://127.0.0.1:6152
export all_proxy=socks5://127.0.0.1:6153
```

这样命令行流量也会经过 Surge 的规则系统,行为和浏览器一致。

### 5.7 故障排查清单

按顺序排查:

| # | 现象 | 排查方法 |
|---|---|---|
| 1 | Surge 提示 sing-box 意外中止 | `cat /tmp/Surge-External-*.log`,看 sing-box 报错 |
| 2 | sing-box 报 unknown command | 检查 `[Proxy]` 行 `args` 是否每个参数独立 |
| 3 | sing-box 报 address already in use | `lsof -nP -iTCP:7890`,有别的进程占端口 |
| 4 | 连不上服务器(超时) | 没填 `addresses`,触发 DNS 闭环;或服务器真挂了 |
| 5 | TLS 握手失败 | `server_name` / `public_key` / `short_id` 不匹配,核对分享链接 |
| 6 | sing-box check 通过但跑起来报错 | 看是不是 sing-box 版本不兼容(老配置在 1.12+ 失效),按 [迁移文档](https://sing-box.sagernet.org/migration/) 调整 |

### 5.8 安全提醒

- `surge-epp.json` 里有 UUID、public_key 等敏感信息,**不要 commit 到公共仓库**。
- Surge 配置文件 `.conf` 也含密码/UUID,同理。
- 备份文件(`.bak.*`)同样敏感。

---

## 附录:完整可工作配置示例

以下是本文档实测通过的最小可工作集(替换敏感字段后可直接用)。

### A1. `/opt/homebrew/etc/sing-box/surge-epp.json`

```json
{
  "log": { "level": "warn", "timestamp": true },
  "inbounds": [
    {
      "type": "socks",
      "tag": "socks-in",
      "listen": "127.0.0.1",
      "listen_port": 7890,
      "sniff": false
    }
  ],
  "outbounds": [
    {
      "type": "vless",
      "tag": "vless-out",
      "server": "<your-server-domain>",
      "server_port": 53918,
      "uuid": "<your-uuid>",
      "flow": "xtls-rprx-vision",
      "tls": {
        "enabled": true,
        "server_name": "www.microsoft.com",
        "utls": { "enabled": true, "fingerprint": "chrome" },
        "reality": {
          "enabled": true,
          "public_key": "<your-public-key>",
          "short_id": "<your-short-id>"
        }
      }
    }
  ]
}
```

### A2. Surge `.conf` 片段

```ini
[Proxy]
🇺🇸 VLESS-R = external, exec = "/opt/homebrew/opt/sing-box/bin/sing-box", args = "run", args = "-c", args = "/opt/homebrew/etc/sing-box/surge-epp.json", args = "-D", args = "/opt/homebrew/var/lib/sing-box", local-port = 7890, addresses = <server-ip>

[Proxy Group]
🚀 节点选择 = select, "🇺🇸 VLESS-R", DIRECT
```
