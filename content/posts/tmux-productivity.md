---
title: "tmux 实战指南：从入门到高效编程"
date: 2025-04-11
emoji: "🖥️"
tags: ["终端", "tmux", "效率工具", "开发环境"]
draft: false
summary: "系统掌握 tmux 终端复用工具，从核心概念到进阶配置，构建永不丢失的开发环境。"
---

## 为什么你需要 tmux？

如果你经常在终端工作，一定遇到过这些场景：

- SSH 到远程服务器跑任务，网络断了，任务也跟着丢了
- 同时开十几个终端标签页，找不到哪个是哪个
- 想一边看日志一边改代码，来回切换窗口效率低下

tmux（Terminal Multiplexer）就是为解决这些问题而生的。它的核心能力有三点：

1. **会话持久化** — 断开 SSH 连接后，程序继续在后台运行，随时可以恢复
2. **窗口分割** — 在一个终端里同时显示多个面板，无需来回切换
3. **会话管理** — 多个项目各开一个会话，互不干扰，随时切换

本文将从零开始，带你系统掌握 tmux。

## 核心概念：Session → Window → Pane

tmux 的组织结构是三层嵌套：

```
Session（会话）
├── Window（窗口/标签页）
│   ├── Pane（面板）
│   ├── Pane
│   └── Pane
└── Window
    └── Pane
```

一个 Session 可以包含多个 Window，每个 Window 可以分割成多个 Pane。你可以把 Session 理解为一个工作区，Window 为不同的任务标签，Pane 则是同一个屏幕上的分屏区域。

## 安装与启动

```bash
# macOS
brew install tmux

# Ubuntu/Debian
sudo apt install tmux

# 查看版本（本文基于 tmux 3.4+，建议使用最新版）
tmux -V
```

启动方式：

```bash
# 创建命名会话（推荐）
tmux new -s work

# 附加到已有会话
tmux attach -t work
# 简写
tmux a -t work
```

## 前缀键：一切操作的起点

tmux 的所有快捷键都需要先按 **前缀键**，默认是 `Ctrl+b`。

很多资深用户会把前缀键改为 `Ctrl+a`（来自 screen 的习惯，按键更顺手）：

```bash
# ~/.tmux.conf
set -g prefix C-a
unbind C-b
bind C-a send-prefix
```

下文快捷键统一以 `<prefix>` 表示，默认为 `Ctrl+b`，如果按上述配置改为 `Ctrl+a`，替换即可。

## 会话管理

会话是 tmux 最核心的价值 — 让你的工作环境永不丢失。

| 操作 | 快捷键 / 命令 |
|------|---------------|
| 创建会话 | `tmux new -s 项目名` |
| 列出所有会话 | `tmux ls` 或 `<prefix> s` |
| 分离会话（后台运行） | `<prefix> d` |
| 附加到会话 | `tmux a -t 项目名` |
| 切换到上一个会话 | `<prefix> (` |
| 切换到下一个会话 | `<prefix> )` |
| 重命名会话 | `<prefix> $` |
| 杀死会话 | `tmux kill-session -t 项目名` |

**实用技巧**：按 `<prefix> d` 分离会话后，即使关闭终端窗口或断开 SSH，所有程序继续运行。下次连接时 `tmux a -t 项目名` 即可恢复。

## 窗口管理

窗口类似于浏览器标签页，适合按任务分组：

| 操作 | 快捷键 |
|------|--------|
| 新建窗口 | `<prefix> c` |
| 切换到窗口 N | `<prefix> 0-9` |
| 上一个窗口 | `<prefix> p` |
| 下一个窗口 | `<prefix> n` |
| 重命名窗口 | `<prefix> ,` |
| 关闭当前窗口 | `<prefix> &` |

## 面板分割：一个屏幕，多个视图

这是日常使用频率最高的功能：

| 操作 | 快捷键 |
|------|--------|
| 左右分割 | `<prefix> %` |
| 上下分割 | `<prefix> "` |
| 切换面板 | `<prefix> 方向键` 或 `<prefix> o` |
| 显示面板编号 | `<prefix> q` |
| 调整面板大小 | `<prefix> Alt+方向键` |
| 关闭当前面板 | `<prefix> x` |
| 面板升级为窗口 | `<prefix> !` |

## 推荐配置

创建 `~/.tmux.conf`，以下是一份经过实战验证的配置：

```bash
# ==================== 基础设置 ====================
# 前缀键改为 Ctrl+a
set -g prefix C-a
unbind C-b
bind C-a send-prefix

# 256 色终端
set -g default-terminal "screen-256color"

# 开启鼠标支持（可用鼠标切换面板、调整大小、滚动）
set -g mouse on

# 窗口和面板编号从 1 开始（0 在键盘太远了）
set -g base-index 1
setw -g pane-base-index 1

# 快速重载配置
bind r source-file ~/.tmux.conf \; display "配置已重载!"

# ==================== 更直觉的面板分割 ====================
# | 左右分屏，- 上下分屏（视觉上更直观）
bind | split-window -h -c "#{pane_current_path}"
bind - split-window -v -c "#{pane_current_path}"
unbind '"'
unbind %

# ==================== Vim 风格面板导航 ====================
bind h select-pane -L
bind j select-pane -D
bind k select-pane -U
bind l select-pane -R

# Alt+方向键调整面板大小
bind -n M-Up resize-pane -U 5
bind -n M-Down resize-pane -D 5
bind -n M-Left resize-pane -L 5
bind -n M-Right resize-pane -R 5

# ==================== 复制模式（Vi 风格） ====================
setw -g mode-keys vi
bind -T copy-mode-vi v send -X begin-selection
bind -T copy-mode-vi y send -X copy-pipe-and-cancel "pbcopy"

# ==================== 其他优化 ====================
# 更大的历史记录
set -g history-limit 50000

# 窗口关闭后自动重新编号
set -g renumber-windows on

# 新窗口保持当前目录
bind c new-window -c "#{pane_current_path}"
```

修改配置后，在 tmux 内按 `<prefix>` 然后 `r` 即可重载。

### 插件管理：tpm

[tpm](https://github.com/tmux-plugins/tpm)（Tmux Plugin Manager）是 tmux 社区最流行的插件管理器，让安装和管理插件变得简单：

```bash
# 安装 tpm
git clone https://github.com/tmux-plugins/tpm ~/.tmux/plugins/tpm
```

在 `~/.tmux.conf` 底部添加：

```bash
# ==================== 插件 ====================
set -g @plugin "tmux-plugins/tpm"
set -g @plugin "tmux-plugins/tmux-sensible"    # 社区公认的最佳默认配置
set -g @plugin "tmux-plugins/tmux-resurrect"  # 会话持久化（重启电脑后恢复）
set -g @plugin "tmux-plugins/tmux-continuum"  # 自动保存会话
set -g @plugin "tmux-plugins/tmux-yank"       # 增强复制功能

# 初始化 tpm（必须放在配置文件最后）
run ~/.tmux/plugins/tpm/tpm
```

安装插件：在 tmux 内按 `<prefix> + I`（大写），tpm 会自动下载并安装所有插件。

**必装插件说明**：
- **tmux-resurrect** — 保存和恢复 tmux 会话环境。即使重启电脑，也能一键恢复所有窗口、面板布局和运行中的程序
- **tmux-continuum** — 配合 resurrect，每隔 15 分钟自动保存一次会话状态

### 状态栏美化

默认的 tmux 状态栏比较朴素，可以通过配置让它显示更多有用信息：

```bash
# ==================== 状态栏 ====================
# 底部状态栏
set -g status-position bottom
set -g status-style "bg=#1a1b26 fg=#a9b1d6"

# 左侧显示会话名
set -g status-left "#[fg=#7aa2f7,bold]  #S "
set -g status-left-length 30

# 右侧显示时间和日期
set -g status-right "#[fg=#565f89] %Y-%m-%d  %H:%M "
set -g status-right-length 50

# 当前窗口高亮
setw -g window-status-current-style "fg=#1a1b26 bg=#7aa2f7 bold"
setw -g window-status-current-format " #I:#W "

# 非当前窗口
setw -g window-status-style "fg=#565f89 bg=#1a1b26"
setw -g window-status-format " #I:#W "
```

以上配色基于 Tokyo Night 主题风格，你可以根据自己终端主题调整颜色。

## 进阶技巧

### 1. 复制模式

在 tmux 中不能直接用鼠标选中文本复制（开启鼠标模式后需要按住 Shift）。以下步骤基于上面的推荐配置（Vi 模式 + pbcopy）：

1. `<prefix> [` 进入复制模式
2. 用方向键或 Vim 键位（hjkl）移动光标
3. 按 `v` 开始选择，移动光标选中内容
4. 按 `y` 复制（内容直接进系统剪贴板）
5. `<prefix> ]` 粘贴

### 2. 同步输入（批量操作）

同时在多个服务器执行相同命令时非常有用：

```bash
# 开启面板同步
<prefix> : setw synchronize-panes on

# 关闭同步
<prefix> : setw synchronize-panes off
```

开启后，在一个面板中输入的内容会同步发送到当前窗口的所有面板。适合同时在多台服务器上执行部署、更新等操作。

### 3. 会话脚本化

把常用的工作环境写成脚本，一键启动：

```bash
#!/bin/bash
# ~/scripts/tmux-dev.sh
SESSION="dev"

if ! tmux has-session -t $SESSION 2>/dev/null; then
    # 创建编辑器窗口
    tmux new-session -d -s $SESSION -n "editor"
    tmux send-keys -t $SESSION:0 "cd ~/projects/myapp" C-m
    tmux send-keys -t $SESSION:0 "vim" C-m

    # 创建终端窗口
    tmux new-window -t $SESSION:1 -n "terminal"
    tmux send-keys -t $SESSION:1 "cd ~/projects/myapp" C-m

    # 创建监控窗口（分屏显示 htop + 日志）
    tmux new-window -t $SESSION:2 -n "monitor"
    tmux split-window -h -t $SESSION:2
    tmux send-keys -t $SESSION:2.0 "htop" C-m
    tmux send-keys -t $SESSION:2.1 "tail -f logs/app.log" C-m

    tmux select-window -t $SESSION:0
fi

tmux attach -t $SESSION
```

运行 `bash ~/scripts/tmux-dev.sh` 即可恢复完整的工作环境。

如果觉得手写脚本太繁琐，可以试试 [tmuxinator](https://github.com/tmuxinator/tmuxinator) 或 [tmuxp](https://github.com/tmux-python/tmuxp)，用 YAML 文件声明式地定义会话布局，比 bash 脚本更易维护。

### 4. 与 fzf 集成实现模糊搜索

> 需要 tmux 3.2+（支持 `display-popup`）且已安装 [fzf](https://github.com/junegunn/fzf)。

```bash
# 模糊切换会话
bind s display-popup -E "tmux list-sessions | fzf --reverse | cut -d: -f1 | xargs tmux switch-client -t"

# 模糊切换窗口
bind w display-popup -E "tmux list-windows | fzf --reverse | cut -d: -f1 | xargs tmux select-window -t"
```

## 实用工作流示例

### 开发工作流

```
┌─────────────────────────────────┐
│  Window "edit"                  │
│  ┌──────────────┬──────────────┐│
│  │  编辑器       │  文件浏览     ││
│  └──────────────┴──────────────┘│
├─────────────────────────────────┤
│  Window "run"                   │
│  ┌──────────────┬──────────────┐│
│  │  运行服务     │  执行测试     ││
│  └──────────────┴──────────────┘│
├─────────────────────────────────┤
│  Window "logs"                  │
│  ┌─────────────────────────────┐│
│  │  tail -f 实时日志            ││
│  └─────────────────────────────┘│
└─────────────────────────────────┘
```

- **edit** 窗口：左边写代码，右边用文件浏览器或 git diff
- **run** 窗口：左边启动服务，右边跑测试
- **logs** 窗口：全屏查看日志

### AI 辅助编程工作流

在 AI 时代，tmux 可以很好地管理多个 AI 工具和终端：

```
┌─────────────────────────────────┐
│  Window "code"                  │
│  ┌──────────────┬──────────────┐│
│  │  Claude Code │  运行/测试    ││
│  └──────────────┴──────────────┘│
├─────────────────────────────────┤
│  Window "ai"                    │
│  ┌─────────────────────────────┐│
│  │  AI Agent 交互界面           ││
│  └─────────────────────────────┘│
├─────────────────────────────────┤
│  Window "monitor"               │
│  ┌──────────────┬──────────────┐│
│  │  htop        │  GPU 状态     ││
│  └──────────────┴──────────────┘│
└─────────────────────────────────┘
```

tmux 的会话持久化在这里特别有价值 — AI Agent 可能需要长时间运行任务，即使断开连接也不会丢失进度。

## 速查表

```
# <prefix> = Ctrl+b（默认）或 Ctrl+a（推荐配置后）
# 带 * 号的为推荐配置中的自定义快捷键

# ==================== 会话 ====================
tmux new -s name          创建命名会话
tmux ls                   列出所有会话
tmux a -t name            附加到会话
<prefix> d                分离会话（后台运行）
tmux kill-server          杀死所有会话

# ==================== 面板 ====================
<prefix> %                左右分屏
<prefix> "                上下分屏
* <prefix> |              左右分屏（自定义，更直觉）
* <prefix> -              上下分屏（自定义，更直觉）
<prefix> 方向键            切换面板
* <prefix> h/j/k/l        Vim 风格切换面板（自定义）
<prefix> x                关闭面板
<prefix> Alt+方向键        调整面板大小

# ==================== 窗口 ====================
<prefix> c                新建窗口
<prefix> n/p              下一个/上一个窗口
<prefix> 0-9              切换到指定窗口
<prefix> ,                重命名窗口

# ==================== 其他 ====================
<prefix> [                进入复制模式
<prefix> ]                粘贴
<prefix> ?                查看所有快捷键
<prefix> :                命令模式
* <prefix> r              重载配置（自定义）
```

## 学习路线建议

不要试图一次记住所有快捷键。推荐按以下节奏来：

**第 1 天** — 学会创建会话、分离和附加。这是 tmux 最核心的价值。

**第 3 天** — 掌握面板分割和窗口切换，开始用 `<prefix> %` 和 `<prefix> "` 分屏工作。

**第 7 天** — 写一份自己的 `~/.tmux.conf`，改前缀键，开启鼠标模式，配置 Vim 风格导航，安装 tpm 插件。

**第 14 天** — 尝试会话脚本化和同步输入等高级功能，建立自己的工作流。

关键是一旦开始用 tmux，就强迫自己不回退到多标签页的方式。一周左右就会形成肌肉记忆，再也回不去了。

## 故障排除

```bash
# 会话卡住，想强行恢复
tmux kill-session -t stuck-session

# 端口冲突，用不同 socket
tmux -L temp new -s test

# 强制分离其他客户端
tmux detach-client -a

# 重置一切
tmux kill-server
```

---

tmux 是一个"学一次、用一辈子"的工具。它的设计哲学简洁而强大 — 通过会话持久化、窗口分割和灵活的配置系统，让你在终端中获得近乎 IDE 级别的工作环境管理能力。配合 Vim/Neovim 和 AI 编程工具，可以打造出极其高效的开发工作流。

开始用吧，一周后你会感谢自己。
