# Ghostty 终端使用指南

> 一份精简的 Ghostty 快捷键速查与使用笔记，适合日常翻阅。

## 📑 目录

- [窗口与标签](#窗口与标签)
- [文本编辑](#文本编辑)
- [滚动与导航](#滚动与导航)
- [屏幕导出](#屏幕导出)
- [主题切换](#主题切换)
- [查看快捷键](#查看快捷键)
- [其他特性](#其他特性)
- [已知问题](#已知问题)
- [参考链接](#参考链接)

---

## 窗口与标签

| 快捷键 | 功能 |
| :--- | :--- |
| `Cmd` + `,` | 打开配置文件 |
| `Cmd` + `Shift` + `,` | 重新加载配置 |
| `Cmd` + `T` | 新开 Tab |
| `Cmd` + `D` | 新开竖排分屏 |
| `Cmd` + `Shift` + `D` | 新开横排分屏 |
| `Cmd` + `Shift` + `Enter` | 当前页全屏 / 取消全屏 |
| `Cmd` + `W` | 关闭当前页 |
| `Cmd` + `[` / `]` | 切换当前 Tab 内的分屏 |
| `Cmd` + `Shift` + `[` / `]` | 切换 Tab |
| `Cmd` + `K` | 清屏（比 `clear` 命令更彻底） |
| `Cmd` + `Shift` + `P` | 命令面板 |

---

## 文本编辑

| 快捷键 | 功能 |
| :--- | :--- |
| `Cmd` + `←` / `→` | 跳到行首 / 行尾 |
| `Cmd` + `Backspace` | 删除整行 |
| `Option` + `←` / `→` | 按单词向左 / 向右移动 |

---

## 滚动与导航

| 快捷键 | 功能 |
| :--- | :--- |
| `Cmd` + `Home` | 滚动到顶部 |
| `Cmd` + `End` | 滚动到底部 |
| `Cmd` + `PageUp` / `PageDown` | 上 / 下翻页 |

---

## 屏幕导出

| 快捷键 | 功能 |
| :--- | :--- |
| `Cmd` + `Option` + `Shift` + `J` | 导出屏幕内容到文件 |
| `Cmd` + `Control` + `Shift` + `J` | 导出屏幕内容并复制路径 |

---

## 主题切换

```bash
# 列出全部内置主题
ghostty +list-themes
```

- 按 `/` 开始搜索
- 按 `Ctrl` + `C` 关闭主题预览

将选中的主题名写入配置文件：

```ini
theme = xxx xxx
```

---

## 查看快捷键

```bash
# 查看默认键位绑定
ghostty +list-keybinds --default
```

---

## 其他特性

- ✅ 分屏窗口支持鼠标拖拽调整大小
- ✅ 支持 [Kitty 图像协议](https://sw.kovidgoyal.net/kitty/graphics-protocol/)
- ✅ 选中即复制：在配置中开启

```ini
copy-on-select = clipboard
```

---

## 已知问题

### Cmd+点击 无法打开本地文件路径

**现象**：在 iTerm2 中可以 `Cmd` + 鼠标点击直接打开终端里显示的本地文件绝对路径（如 `/Users/xxx/foo.md`），但在 Ghostty 中点击没有反应。

**原因**：Ghostty 默认的链接识别规则**只匹配 URL**，不识别裸文件路径。官方文档原文：

> A default link that matches a URL and opens it in the system opener always exists.

虽然 Ghostty 提供了 `link` 配置项允许通过自定义正则匹配文件路径，但官方文档同时标注 **"TODO: This can't currently be set!"**，即**该自定义配置目前尚未真正实现**。可点击文件路径功能在 [Issue #1972](https://github.com/ghostty-org/ghostty/issues/1972) 中已被提出，但**官方仍未实现**。

**临时解决方案**：

1. 使用 `file://` URL 前缀（Ghostty 能识别 URL，可 Cmd+点击打开）：
   ```
   file:///Users/xxx/foo.md
   ```
2. 在终端里直接执行 `open` 命令：
   ```bash
   open ./foo.md
   ```
3. 在 `~/.zshrc` 中添加别名，简化输入：
   ```zsh
   alias o='open'
   ```

**参考来源**：

- [Ghostty Configuration Reference - link option](https://ghostty.org/docs/config/reference#link)
- [Clickable file paths · Issue #1972](https://github.com/ghostty-org/ghostty/issues/1972)

---

## 参考链接

- [Ghostty 配置指南 - axiaoxin](https://blog.axiaoxin.com/post/ghostty-config-guide/)
- [Ghostty 使用记录 - 博客园](https://www.cnblogs.com/sueyyyy/p/19748613)
- [Ghostty 官方配置文档](https://ghostty.org/docs/config/reference)
