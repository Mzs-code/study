# Claude Code Buddy 宠物修改指南

## 宠物系统原理

Claude Code 的 `/buddy` 宠物系统由两层组成：

- **Bones（骨架）**：外观、种族、稀有度、属性 — 通过 `hash(userID + SALT)` 确定性生成，每次实时计算
- **Soul（灵魂）**：名字和性格 — 由 AI 生成，缓存在 `~/.claude.json` 的 `companion` 字段中

Salt 默认值为 `friend-2026-401`（15 字符），出现在二进制文件中 3 处。

## 可选属性一览

### 种族（18 种）

| 种族 | 英文 | 种族 | 英文 |
|------|------|------|------|
| 鸭子 | duck | 鹅 | goose |
| 团子 | blob | 猫 | cat |
| 龙 | dragon | 章鱼 | octopus |
| 猫头鹰 | owl | 企鹅 | penguin |
| 乌龟 | turtle | 蜗牛 | snail |
| 幽灵 | ghost | 六角恐龙 | axolotl |
| 水豚 | capybara | 仙人掌 | cactus |
| 机器人 | robot | 兔子 | rabbit |
| 蘑菇 | mushroom | 胖墩 | chonk |

### 稀有度（5 级）

| 稀有度 | 英文 | 概率 | 属性下限 | 星级 |
|--------|------|------|----------|------|
| 普通 | common | 60% | 5 | ★ |
| 不常见 | uncommon | 25% | 15 | ★★ |
| 稀有 | rare | 10% | 25 | ★★★ |
| 史诗 | epic | 4% | 35 | ★★★★ |
| 传奇 | legendary | 1% | 50 | ★★★★★ |

### 眼睛（6 种）

| 样式 | 字符 |
|------|------|
| 点 | `·` |
| 星 | `✦` |
| 叉 | `×` |
| 圈 | `◉` |
| AT | `@` |
| 度 | `°` |

### 帽子（8 种）

| 帽子 | 英文 | 说明 |
|------|------|------|
| 无 | none | 无帽子（common 固定无帽子） |
| 皇冠 | crown | `\^^^/` |
| 高帽 | tophat | `[___]` |
| 螺旋桨 | propeller | `-+-` |
| 光环 | halo | `(   )` |
| 巫师帽 | wizard | `/^\` |
| 毛线帽 | beanie | `(___)` |
| 小鸭子 | tinyduck | `,>` |

> 注意：common 稀有度的宠物固定无帽子（none），其他稀有度随机分配。

### 闪光（Shiny）

- 概率约 1%
- 搜索时间约为普通的 100 倍

### 属性（5 项）

| 属性 | 说明 |
|------|------|
| DEBUGGING | 调试 |
| PATIENCE | 耐心 |
| CHAOS | 混乱 |
| WISDOM | 智慧 |
| SNARK | 毒舌 |

每只宠物有一项巅峰属性和一项最低属性，其余随机。属性不可单独指定。

## 修改方法

### 推荐：使用 any-buddy 工具

[any-buddy](https://github.com/cpaczek/any-buddy) 是社区验证过的工具，通过替换二进制中的 salt 来改变宠物。

#### 前置条件

- Node.js >= 18
- Bun（用于 wyhash 计算）
- Claude Code 已安装

#### 安装 Bun（如未安装）

```bash
curl -fsSL https://bun.sh/install | bash
```

#### 基本用法

```bash
# 交互式选择
npx any-buddy@latest

# 非交互式（直接指定属性）
npx any-buddy@latest --species cat --rarity legendary --eye '✦' --hat none --shiny --yes

# 查看当前宠物
npx any-buddy@latest current

# 恢复原始状态
npx any-buddy@latest restore

# 删除宠物缓存（重新孵化）
npx any-buddy@latest rehatch
```

#### 完整参数

```bash
npx any-buddy@latest \
  --species <种族>      \
  --rarity <稀有度>     \
  --eye '<眼睛字符>'    \
  --hat <帽子>          \
  --shiny               \
  --name <自定义名字>   \
  --yes                 # 跳过确认
```

### 修改后生效步骤

1. any-buddy 自动完成二进制补丁和 macOS 重签名
2. 手动删除 `~/.claude.json` 中的 `companion` 字段（让系统重新生成名字和性格）
3. **重启 Claude Code**
4. 输入 `/buddy` 查看新宠物

## 注意事项

1. **必须用 Bun 运行**：Claude Code 使用 wyhash（Bun.hash），不是 FNV-1a（Node.js），用 Node.js 运行的结果不匹配
2. **Salt 长度必须为 15 字符**：替换 salt 必须与原始 salt 等长，否则会破坏二进制文件的字节偏移
3. **macOS 需要重签名**：补丁后需执行 `codesign --force --sign -`，any-buddy 会自动处理
4. **Claude Code 更新会覆盖补丁**：更新后需要重新执行 `npx any-buddy@latest apply`，或安装 SessionStart hook 自动修复
5. **备份文件**：首次补丁时会创建 `.anybuddy-bak` 备份，可通过 `any-buddy restore` 恢复
6. **companion 缓存**：修改后如果仍显示旧宠物，检查 `~/.claude.json` 中是否残留 `companion` 字段，手动删除即可
7. **属性不可单独指定**：只能指定种族、稀有度、眼睛、帽子、闪光，属性是随机分配的（一项巅峰、一项最低、其余随机）
8. **运行中安全**：补丁使用原子 rename 操作，正在运行的 Claude Code 不受影响，重启后生效

## 相关资源

- [any-buddy (GitHub)](https://github.com/cpaczek/any-buddy)
- [buddy-reroll (GitHub)](https://github.com/grayashh/buddy-reroll)
- [Claude Code /buddy 宠物系统逆向分析 (LINUX DO)](https://linux.do/t/topic/1871870)
