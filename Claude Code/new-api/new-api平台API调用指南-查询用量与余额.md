# new-api平台API调用指南-查询用量与余额

## 参考资料

- [New-API GitHub](https://github.com/QuantumNous/new-api)
- [New-API 官方文档](https://www.newapi.ai/zh/docs/api)
- [New-API Pro 接口文档](https://doc.newapi.pro/api/fei-user/)

## 1. 前言

大部分 AI 中转站都是使用 [New-API](https://github.com/Calcium-Ion/new-api)，所以就有必要基于其 API 来做一些自动化——比如每 30 分钟检测余额、统计团队每日消耗和排行，再推送给钉钉机器人。

本文介绍 New-API 平台中几个常用管理 API 的调用方式。

## 2. 准备工作：获取认证信息

调用 New-API 管理 API 需要两个认证信息：**系统访问令牌**和**用户 ID**。

### 2.1 获取系统访问令牌

1. 登录你的 New-API 平台
2. 进入 **个人设置 → 安全设置**
3. 找到 **系统访问令牌** 部分
4. 点击「生成令牌」或「复制」按钮获取令牌

> ⚠️ 系统访问令牌拥有较高权限，请妥善保管，不要泄露或提交到公开仓库。

### 2.2 获取用户 ID

用户 ID 是一个数字标识，可通过以下方式获取：

- **方式一**：登录平台后，在个人信息页面查看
- **方式二**：通过 API 响应中的 `data.id` 字段获取（见下文 3.2 节）

## 3. 构造 API 请求

### 3.1 请求头说明

所有管理 API 请求都需要携带以下两个 Header：

| Header | 值 | 说明 |
|--------|---|------|
| `Authorization` | `Bearer <your-token>` | 系统访问令牌 |
| `New-Api-User` | `<your-user-id>` | 数字用户 ID |

### 3.2 查询用户信息（余额）

**接口**：`GET /api/user/self`

该接口返回当前用户的完整信息，包括余额（`quota`）和累计用量（`used_quota`）。

**curl 示例**：

```bash
curl -X GET "https://your-newapi-domain.com/api/user/self" \
    -H "Authorization: Bearer <your-token>" \
    -H "New-Api-User: <your-user-id>"
```

**响应示例**：

```json
{
  "success": true,
  "message": "",
  "data": {
    "id": 12345,
    "username": "example_user",
    "display_name": "示例用户",
    "email": "user@example.com",
    "role": 1,
    "status": 1,
    "quota": 157562861,
    "used_quota": 1242937139,
    "request_count": 19200,
    "group": "default",
    "aff_code": "XXXX",
    "inviter_id": 0
  }
}
```

**关键字段**：

| 字段 | 说明 |
|------|------|
| `quota` | 剩余额度（内部单位，需 / 500000 转换为 USD） |
| `used_quota` | 已使用额度（内部单位，需 / 500000 转换为 USD） |
| `request_count` | 累计请求次数 |
| `group` | 用户所属分组 / 套餐 |

### 3.3 查询当日消耗统计

**接口**：`GET /api/log/self/stat`

该接口返回指定时间范围内的消耗汇总统计。

**参数说明**：

| 参数 | 类型 | 说明 |
|------|------|------|
| `type` | int | 统计类型，`2` 表示按消耗统计 |
| `start_timestamp` | int | 起始时间（Unix 时间戳，秒） |
| `end_timestamp` | int | 结束时间（Unix 时间戳，秒） |
| `token_name` | string | 可选，按令牌名称过滤 |
| `model_name` | string | 可选，按模型名称过滤 |
| `group` | string | 可选，按分组过滤 |

**curl 示例**（查询当日消耗）：

> `start_timestamp` 和 `end_timestamp` 为 Unix 时间戳（**秒**级）。

```bash
curl -X GET "https://your-newapi-domain.com/api/log/self/stat?type=2&token_name=&model_name=&start_timestamp=1773072000&end_timestamp=1773134231&group=" \
    -H "Authorization: Bearer <your-token>" \
    -H "New-Api-User: <your-user-id>"
```

**响应示例**：

```json
{
  "success": true,
  "message": "",
  "data": {
    "quota": 84550951,
    "rpm": 1,
    "tpm": 1841
  }
}
```

**关键字段**：

| 字段 | 说明 |
|------|------|
| `quota` | 时间范围内的总消耗（内部单位，需 / 500000 转换为 USD） |
| `rpm` | 当前每分钟请求数 |
| `tpm` | 当前每分钟 Token 数 |

### 3.4 查询消耗排行

**接口**：`GET /api/token/usage_detail`

该接口返回各令牌的消耗明细和排行，适合管理员了解团队成员的使用情况。

**参数说明**：

| 参数 | 类型 | 说明 |
|------|------|------|
| `p` | int | 页码，从 1 开始 |
| `size` | int | 每页条数 |
| `start_timestamp` | int | 起始时间（Unix 时间戳，秒） |
| `end_timestamp` | int | 结束时间（Unix 时间戳，秒） |
| `order_by` | string | 排序字段，如 `cost` |
| `order_direction` | string | 排序方向，`desc`（降序）或 `asc`（升序） |

**curl 示例**（查询当日消耗 Top 10）：

> `start_timestamp` 和 `end_timestamp` 为 Unix 时间戳（**秒**级）。

```bash
curl -X GET "https://your-newapi-domain.com/api/token/usage_detail?p=1&size=10&start_timestamp=1773072000&end_timestamp=1773134654&order_by=cost&order_direction=desc" \
    -H "Authorization: Bearer <your-token>" \
    -H "New-Api-User: <your-user-id>"
```

**响应示例**：

```json
{
  "success": true,
  "message": "",
  "data": {
    "page": 1,
    "page_size": 10,
    "total": 38,
    "items": [
      {
        "id": 60001,
        "name": "user-a-cc",
        "status": 1,
        "key": "sk-xxxx...xxxx",
        "request_count": 544,
        "cost": 33260054,
        "last_used_time": 1773115381
      },
      {
        "id": 60002,
        "name": "user-b-cc",
        "status": 1,
        "key": "sk-yyyy...yyyy",
        "request_count": 355,
        "cost": 15791812,
        "last_used_time": 1773130777
      }
    ]
  }
}
```

**关键字段**：

| 字段 | 说明 |
|------|------|
| `items[].name` | 令牌名称 |
| `items[].request_count` | 请求次数 |
| `items[].cost` | 消耗额度（内部单位，需 / 500000 转换为 USD） |
| `items[].last_used_time` | 最后使用时间（Unix 时间戳） |
| `total` | 符合条件的令牌总数 |

## 4. 实用场景：在 cc-switch 中显示用量和余额

[cc-switch](https://github.com/maozs/cc-switch) 是一个 Claude Code 多账户切换工具，支持配置 New-API 格式来自动显示各账户的用量和余额信息。

如果你使用 cc-switch 管理多个 New-API 账户，可以在其配置中填入系统访问令牌和用户 ID，工具会自动调用上述 API 并展示余额和消耗数据。具体配置方式请参考 cc-switch 项目文档。
