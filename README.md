# Windows 终端自动补全配置指南

> 基于 Git for Windows (Git Bash) 的终端自动补全完整配置教程
>
> 环境：Windows 11 + Git Bash + Bash 5.2 + Git 2.53

---

## 目录

1. [概述](#1-概述)
2. [环境准备](#2-环境准备)
3. [Git 自动补全](#3-git-自动补全)
4. [Bash 内置补全功能](#4-bash-内置补全功能)
5. [编码与兼容性配置](#5-编码与兼容性配置)
6. [自定义别名与补全](#6-自定义别名与补全)
7. [推荐的第三方补全工具](#7-推荐的第三方补全工具)
8. [故障排查](#8-故障排查)

---

## 1. 概述

终端自动补全是提升命令行效率的核心功能。本教程记录在 **Windows 11** 上使用 **Git for Windows** 自带的 **Git Bash** 环境配置自动补全的完整过程。

### 效果预览

| 功能 | 效果 |
|------|------|
| Git 命令补全 | 输入 `git che` + Tab → 自动补全为 `git checkout` |
| Git 分支/文件补全 | `git checkout fea` + Tab → 自动补全为 `git checkout feature/my-branch` |
| 文件名补全 | 输入部分文件名 + Tab 自动补全 |
| 路径补全 | 输入 `cd ~/Doc` + Tab → 自动补全为 `cd ~/Documents` |
| 主机名补全 | `ssh my-s` + Tab → 自动补全主机名 |
| 命令历史搜索 | Ctrl+R 搜索历史命令 |

---

## 2. 环境准备

### 2.1 安装 Git for Windows

从 [git-scm.com](https://git-scm.com) 下载并安装 Git for Windows。

安装时建议勾选：
- **Git Bash Here** — 右键菜单快捷入口
- **Use a TrueType font** — 解决中文乱码
- **Checkout as-is, commit as-is** — 避免换行符问题

安装完成后，Git Bash 自动附带 Git 自动补全脚本。

### 2.2 检查环境

打开 Git Bash，运行以下命令确认环境版本：

```bash
# 查看 Shell
echo $SHELL          # → /usr/bin/bash
echo $BASH_VERSION   # → 5.2.37(1)-release

# 查看 Git 版本
git --version        # → git version 2.53.0.windows.2

# 查看 Git 安装路径
which git            # → /mingw64/bin/git
```

> Git for Windows 的 Bash 是 MSYS2 环境，与原生 Linux Bash 行为一致。

---

## 3. Git 自动补全

### 3.1 自动补全如何加载

Git for Windows 在安装时已内置补全脚本。Bash 自动补全的加载流程如下：

```
bash 启动
  → .bash_profile (登录 shell)
    → .profile (如果存在)
    → .bashrc (交互式 shell)
      → Git 自动补全脚本 (由 Git for Windows 自动注入)
```

Git for Windows 在 `/etc/bash.bashrc` 中自动 source 了 Git 补全脚本，因此 **开箱即用，无需额外配置**。

### 3.2 Git 补全都包含什么

用以下命令查看当前加载的所有补全函数：

```bash
declare -F | grep -i "comp\|completion\|complete"
```

Git 自动补全包含：

| 函数组 | 功能 |
|--------|------|
| `__git_complete` 系列 | Git 命令补全调度 |
| `__gitcomp` / `__gitcomp_builtin` | 命令选项补全 |
| `__git_complete_refs` | 分支/标签名补全 |
| `__git_complete_file` | Git 文件路径补全 |
| `__git_complete_log_opts` | `git log` 选项补全 |
| `__git_complete_index_file` | 已暂存文件补全 |

### 3.3 查看已注册的补全

使用 `complete` 命令查看当前哪些命令注册了自动补全：

```bash
# 查看所有补全注册
complete -p

# 输出示例：
# complete -o bashdefault -o default -o nospace -F __git_wrap__git_main git
# complete -o bashdefault -o default -o nospace -F __git_wrap__gitk_main gitk
```

这意味着：
- `git` 命令 → 使用 `__git_wrap__git_main` 函数补全（覆盖所有 git 子命令）
- `gitk` 命令 → 使用 `__git_wrap__gitk_main` 函数补全

### 3.4 手动添加 Git 补全（备用方案）

如果某些原因没有自动加载，可以在 `.bashrc` 中手动添加：

```bash
# 手动加载 Git 自动补全
if [ -f /usr/share/bash-completion/completions/git ]; then
    . /usr/share/bash-completion/completions/git
fi
```

---

## 4. Bash 内置补全功能

### 4.1 可编程补全

Bash 的可编程补全功能由以下 `shopt` 选项控制：

```bash
# 检查补全相关选项
shopt | grep -E "progcomp|hostcomplete|globstar"

progcomp        on   # 可编程补全（核心）
hostcomplete    on   # 主机名补全（@ 符号触发）
globstarskipdots on  # glob 跳过隐藏文件
```

### 4.2 常见补全操作

| 操作 | 说明 |
|------|------|
| `Tab` | 补全命令/文件名/路径 |
| `Tab` + `Tab` | 显示所有匹配项 |
| `Alt` + `/` | 补全文件名（不补全命令） |
| `Alt` + `$` | 补全变量名（如 `$HOME`） |
| `Alt` + `~` | 补全用户名（如 `~user/`） |
| `Alt` + `@` | 补全主机名 |
| `Alt` + `!` | 补全命令历史 |
| `Ctrl` + `R` | 搜索历史命令（反向搜索） |
| `Ctrl` + `G` | 退出历史搜索模式 |

### 4.3 编辑模式

Bash 默认使用 Emacs 风格快捷键：

```bash
set -o | grep -E "vi|emacs"

emacs           on   # Emacs 模式（默认）
vi              off  # Vi 模式
```

> 如果更喜欢 Vi 模式，可以执行 `set -o vi`，或写在 `.bashrc` 中。

---

## 5. 编码与兼容性配置

在 Windows 上使用 Git Bash 时，**编码问题**是最常见的坑。

### 5.1 当前配置

```bash
# .bashrc 中的编码配置
alias powershell="pwsh"

# 将 Windows 控制台编码设为 UTF-8
chcp.com 65001 > /dev/null
```

- `chcp.com 65001` 将 Windows 控制台代码页切换为 UTF-8，防止中文乱码
- `> /dev/null` 屏蔽输出信息，保持 shell 启动干净
- `alias powershell="pwsh"` 可用 `powershell` 命令在 Git Bash 中调用 PowerShell Core

### 5.2 完整配置建议

将以下内容写入 `~/.bashrc`：

```bash
# ====================================
# Git Bash 个人配置
# ====================================

# ---- 编码 ----
chcp.com 65001 > /dev/null 2>&1

# ---- 别名 ----
alias powershell="pwsh"
alias ll="ls -la"
alias la="ls -A"
alias ..="cd .."
alias ...="cd ../.."

# ---- PATH ----
export PATH="$HOME/AppData/Local/Python/pythoncore-3.14-64/Scripts:$PATH"

# ---- 提示符 ----
export PS1='\[\e]0;\w\a\]\n\[\e[32m\]\u@\h\[\e[0m\] \[\e[33m\]\w\[\e[0m\]\n\$ '

# ---- 历史记录 ----
export HISTSIZE=10000
export HISTFILESIZE=20000
export HISTCONTROL=ignoredups:erasedups
export HISTTIMEFORMAT="%F %T "
```

### 5.3 `.bash_profile` vs `.bashrc`

| 文件 | 作用 | 加载时机 |
|------|------|----------|
| `.bash_profile` | 登录 shell 配置 | Git Bash 启动时（左上角显示 "login"） |
| `.bashrc` | 交互式 shell 配置 | 每次打开新终端 |
| `.profile` | 通用登录配置 | 如果 `.bash_profile` 不存在 |

当前配置链：

```
Git Bash 启动
  → .bash_profile (存在)
    → .profile (不存在，跳过)
    → .bashrc (存在，加载)
```

如果 `.bash_profile` 不存在，系统会自动查找 `.profile`；如果也不存在，直接加载 `.bashrc`（由 Git for Windows 的 `/etc/bash.bashrc` 处理）。

---

## 6. 自定义别名与补全

### 6.1 为别名添加补全

当为命令创建别名时，补全可能失效。需要手动关联补全函数：

```bash
# 示例：为别名添加 Git 补全
alias g=git
complete -o bashdefault -o default -o nospace -F __git_wrap__git_main g
```

### 6.2 自定义补全函数

创建自定义补全，例如为自定义脚本添加补全：

```bash
# 1. 在 ~/.bash_completion 中定义补全函数
_my_script_completion() {
    local cur prev opts
    cur="${COMP_WORDS[COMP_CWORD]}"
    prev="${COMP_WORDS[COMP_CWORD-1]}"
    opts="start stop restart status"

    COMPREPLY=( $(compgen -W "${opts}" -- ${cur}) )
    return 0
}

# 2. 注册补全
complete -F _my_script_completion my_script
```

### 6.3 `.bash_completion` 目录

建议将自定义补全放在独立的文件中：

```bash
# 在 .bashrc 中加载自定义补全
if [ -f ~/.bash_completion ]; then
    . ~/.bash_completion
fi
```

---

## 7. 推荐的第三方补全工具

以下工具可大幅提升终端体验，建议按需安装：

### 7.1 fzf — 模糊搜索补全

通用模糊搜索工具，支持 Ctrl+T（文件搜索）、Ctrl+R（历史搜索）的模糊匹配。

```bash
# 在 Git Bash 中安装
git clone --depth 1 https://github.com/junegunn/fzf.git ~/.fzf
~/.fzf/install
```

### 7.2 zoxide — 智能目录跳转

学习你的目录使用习惯，实现 `z` + 部分路径名 → 跳转到最常访问的匹配目录。

```bash
# 在 Git Bash 中安装（通过包管理器）
winget install zoxide

# 或手动下载二进制，然后添加到 .bashrc
eval "$(zoxide init bash)"
```

### 7.3 atuin — 历史记录增强

替代 Bash 内置的 `history`，支持模糊搜索、同步历史记录：

```bash
# 安装（要求 Winget）
winget install atuin.atuin

# .bashrc 中初始化
eval "$(atuin init bash)"
```

### 7.4 补全工具对比

| 工具 | 核心功能 | 安装复杂度 | 推荐度 |
|------|---------|-----------|--------|
| **fzf** | 模糊搜索文件/历史/目录 | ★★☆ | ⭐⭐⭐ |
| **zoxide** | 智能目录跳转 | ★☆☆ | ⭐⭐⭐ |
| **atuin** | 带搜索的历史记录 | ★★☆ | ⭐⭐ |
| **starship** | 增强提示符（非补全） | ★★☆ | ⭐⭐⭐ |

---

## 8. 故障排查

### 8.1 补全不工作

```bash
# 1. 确认可编程补全已启用
shopt progcomp    # 应该输出 "progcomp        on"
# 如果为 off，执行：
shopt -s progcomp

# 2. 检查具体命令的补全
complete -p git   # 应该显示注册信息
complete -p ssh   # 如果没有输出，可能未安装 ssh 补全

# 3. 查看补全函数是否存在
declare -f __git_complete  # Git 补全函数
```

### 8.2 中文乱码

```bash
# 临时修复
chcp.com 65001

# 永久修复（已写在 .bashrc 中）
echo 'chcp.com 65001 > /dev/null' >> ~/.bashrc
```

### 8.3 补全太慢

如果目录中文件太多导致补全卡顿：

```bash
# 关闭哈希表，使用精确查找
shopt -u hostcomplete

# 限制补全匹配数量
bind "set completion-query-items 200"
```

### 8.4 查看当前补全会话状态

```bash
# 所有补全函数
declare -F | grep -i "comp\|completion\|complete"

# 所有注册的命令补全
complete -p

# Bash 补全相关选项
shopt | grep -E "progcomp|hostcomplete"

# 键绑定（emacs/vi 模式）
set -o | grep -E "vi|emacs"
```

---

## 附录

### A. 参考链接

- [Git for Windows](https://git-scm.com)
- [Bash Reference Manual — Programmable Completion](https://www.gnu.org/software/bash/manual/html_node/Programmable-Completion.html)
- [fzf GitHub](https://github.com/junegunn/fzf)
- [zoxide GitHub](https://github.com/ajeetdsouza/zoxide)
- [atuin GitHub](https://github.com/atuinsh/atuin)

### B. 完整 `.bashrc` 配置示例

```bash
# ====================================
# ~/.bashrc — Git Bash 配置
# ====================================

# 编码：Windows 控制台切换 UTF-8
chcp.com 65001 > /dev/null 2>&1

# 别名
alias powershell="pwsh"
alias ll="ls -la"
alias la="ls -A"
alias ..="cd .."
alias ...="cd ../.."

# PATH
export PATH="$HOME/AppData/Local/Python/pythoncore-3.14-64/Scripts:$PATH"

# 提示符
export PS1='\[\e]0;\w\a\]\n\[\e[32m\]\u@\h\[\e[0m\] \[\e[33m\]\w\[\e[0m\]\n\$ '

# 历史记录
export HISTSIZE=10000
export HISTFILESIZE=20000
export HISTCONTROL=ignoredups:erasedups
export HISTTIMEFORMAT="%F %T "
```

---

> **维护者**: [lunqin123](https://github.com/lunqin123)
>
> 本文档基于 Windows 11 + Git for Windows 2.53.0 环境编写，如使用其他环境请酌情调整。
