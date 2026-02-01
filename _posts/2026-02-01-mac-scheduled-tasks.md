---
layout: post
title: "Mac 设置定时任务"
date: 2026-02-01 08:00:00 +0800
categories: tutorial
tags: [macOS, launchd, cron, 自动化]
permalink: /mac-scheduled-tasks/
---

在 macOS 上设置定时任务主要有两种方式：**launchd** 和 **cron**。本文将介绍这两种方法。

## 目录

- [方法一：使用 launchd（推荐）](#方法一使用-launchd推荐)
  - [1. 创建 plist 配置文件](#1-创建-plist-配置文件)
  - [2. 编写配置内容](#2-编写配置内容)
  - [3. 加载任务](#3-加载任务)
  - [常用时间设置](#常用时间设置)
  - [实战示例：每天早上9点清理 Downloads 目录中的 .dmg 文件](#实战示例每天早上9点清理-downloads-目录中的-dmg-文件)
- [方法二：使用 cron](#方法二使用-cron)
- [对比总结](#对比总结)
- [注意事项](#注意事项)
- [解决权限问题：访问用户目录](#解决权限问题访问用户目录)
  - [方案一：授予完全磁盘访问权限](#方案一授予完全磁盘访问权限)
  - [方案二：使用 Automator 应用（推荐）](#方案二使用-automator-应用推荐)

---

## 方法一：使用 launchd（推荐）

`launchd` 是 macOS 原生的任务调度系统，功能强大且与系统深度集成。

### 工作原理

`launchd` 是 macOS 的第一个进程（PID 1），负责管理系统启动和所有后台服务。其工作流程如下：

1. **系统启动时**：`launchd` 扫描配置目录（如 `~/Library/LaunchAgents/`）中的 `.plist` 文件
2. **解析配置**：读取每个 plist 文件中的触发条件（时间、事件等）
3. **等待触发**：当条件满足时，启动对应的程序或脚本
4. **管理生命周期**：监控任务状态，处理重启、日志等

### 配置文件位置

| 目录 | 用途 | 权限 |
|-----|-----|-----|
| `~/Library/LaunchAgents/` | 当前用户的任务 | 用户登录后运行 |
| `/Library/LaunchAgents/` | 所有用户的任务 | 需管理员权限 |
| `/Library/LaunchDaemons/` | 系统级服务 | 需 root 权限 |

### ⚠️ 风险提示

1. **权限风险**：配置不当可能导致任务以错误的权限运行
2. **资源消耗**：频繁执行的任务可能影响系统性能和电池寿命
3. **静默执行**：任务在后台运行，错误可能不易察觉，建议配置日志输出
4. **持久化**：恶意软件常利用 launchd 实现开机自启，定期检查配置目录

### 1. 创建 plist 配置文件

在 `~/Library/LaunchAgents/` 目录下创建一个 `.plist` 文件：

```zsh
nano ~/Library/LaunchAgents/com.example.mytask.plist
```

### 2. 编写配置内容

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.example.mytask</string>
    
    <key>ProgramArguments</key>
    <array>
        <string>/bin/zsh</string>
        <string>/path/to/your/script.sh</string>
    </array>
    
    <key>StartCalendarInterval</key>
    <dict>
        <key>Hour</key>
        <integer>9</integer>
        <key>Minute</key>
        <integer>0</integer>
    </dict>
    
    <key>StandardOutPath</key>
    <string>/tmp/mytask.log</string>
    
    <key>StandardErrorPath</key>
    <string>/tmp/mytask.error.log</string>
</dict>
</plist>
```

### 3. 加载任务

```zsh
# 加载任务
launchctl load ~/Library/LaunchAgents/com.example.mytask.plist

# 立即运行一次（测试用）
launchctl start com.example.mytask

# 卸载任务
launchctl unload ~/Library/LaunchAgents/com.example.mytask.plist
```

### 常用时间设置

| 设置 | 说明 |
|-----|-----|
| `Hour` + `Minute` | 每天指定时间运行 |
| `Weekday` (0=周日) | 每周指定日期运行 |
| `Day` | 每月指定日期运行 |
| `StartInterval` | 每隔 N 秒运行一次 |

### 实战示例：每天早上9点清理 Downloads 目录中的 .dmg 文件

这是一个实用的例子，自动清理下载目录中的安装包文件。

#### 步骤 1：创建清理脚本

```zsh
mkdir -p ~/dev/Scripts
nano ~/dev/Scripts/clean_dmg.sh
```

脚本内容：

```zsh
#!/bin/zsh
# 清理 Downloads 目录中的 .dmg 文件

LOG_FILE="/tmp/clean_dmg.log"
DOWNLOADS_DIR="$HOME/Downloads"

echo "$(date): 开始清理 .dmg 文件..." >> "$LOG_FILE"

# 查找并删除 .dmg 文件
find "$DOWNLOADS_DIR" -name "*.dmg" -type f -delete 2>> "$LOG_FILE"

echo "$(date): 清理完成" >> "$LOG_FILE"
```

赋予执行权限：

```zsh
chmod +x ~/dev/Scripts/clean_dmg.sh
```

#### 步骤 2：创建 launchd 配置

```zsh
nano ~/Library/LaunchAgents/com.user.clean-dmg.plist
```

配置内容：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.user.clean-dmg</string>
    
    <key>ProgramArguments</key>
    <array>
        <string>/bin/zsh</string>
        <string>/Users/你的用户名/Scripts/clean_dmg.sh</string>
    </array>
    
    <key>StartCalendarInterval</key>
    <dict>
        <key>Hour</key>
        <integer>9</integer>
        <key>Minute</key>
        <integer>0</integer>
    </dict>
    
    <key>StandardOutPath</key>
    <string>/tmp/clean_dmg.log</string>
    
    <key>StandardErrorPath</key>
    <string>/tmp/clean_dmg.error.log</string>
</dict>
</plist>
```

> ⚠️ **注意**：将 `/Users/你的用户名/` 替换为你的实际用户目录路径，例如 `/Users/john/`

#### 步骤 3：加载并启用任务

```zsh
# 加载任务
launchctl load ~/Library/LaunchAgents/com.user.clean-dmg.plist

# 测试运行（可选）
launchctl start com.user.clean-dmg

# 查看日志确认是否成功
cat /tmp/clean_dmg.log
```

#### 管理任务

```zsh
# 查看任务状态
launchctl list | grep clean-dmg

# 停止并卸载任务
launchctl unload ~/Library/LaunchAgents/com.user.clean-dmg.plist

# 重新加载（修改配置后）
launchctl unload ~/Library/LaunchAgents/com.user.clean-dmg.plist
launchctl load ~/Library/LaunchAgents/com.user.clean-dmg.plist
```

---

## 方法二：使用 cron

`cron` 是传统的 Unix 定时任务工具，语法简洁，源自 Unix 系统。

### 工作原理

`cron` 守护进程在后台持续运行，每分钟检查一次 crontab 配置：

1. **读取配置**：解析用户的 crontab 文件（存储在 `/var/at/tabs/`）
2. **时间匹配**：将当前时间与每条任务的时间表达式比较
3. **执行任务**：匹配成功则启动对应命令
4. **输出处理**：默认将输出发送到用户邮箱（macOS 通常禁用）

### ⚠️ 风险提示

1. **权限要求**：首次使用需在「系统设置 → 隐私与安全性 → 完全磁盘访问权限」中授权 `cron`
2. **错过执行**：如果电脑在计划时间处于睡眠/关机状态，任务会被跳过（不会补执行）
3. **环境变量**：cron 环境与终端不同，`PATH` 等变量需要在脚本中显式设置
4. **无日志**：默认不记录执行日志，需手动在命令中重定向输出

### 1. 编辑 crontab

```zsh
crontab -e
```

### 2. 添加定时任务

```zsh
# 格式：分 时 日 月 周 命令
# 每天早上9点执行
0 9 * * * /path/to/your/script.sh

# 每隔5分钟执行
*/5 * * * * /path/to/your/script.sh

# 每周一早上8点执行
0 8 * * 1 /path/to/your/script.sh
```

### 3. 常用命令

```zsh
# 查看当前任务
crontab -l

# 删除所有任务
crontab -r
```

---

## 对比总结

| 特性 | launchd | cron |
|-----|---------|------|
| 系统集成 | ✅ 原生支持 | ⚠️ 需额外授权 |
| 错过执行 | ✅ 唤醒后补执行 | ❌ 错过即跳过 |
| 配置复杂度 | 较复杂 | 简单 |
| 日志管理 | ✅ 内置支持 | 需手动配置 |

**建议**：对于 macOS 用户，优先使用 `launchd`，它与系统的兼容性更好，且支持睡眠唤醒后补执行任务。

---

## 注意事项

1. **权限问题**：确保脚本有执行权限 (`chmod +x script.sh`)
2. **完整路径**：在定时任务中使用命令的完整路径
3. **环境变量**：定时任务的环境变量与终端不同，需在脚本中显式设置

---

## 解决权限问题：访问用户目录

从 macOS Mojave 开始，后台脚本访问用户目录（如 Downloads、Documents）会遇到 **"Operation not permitted"** 错误。这是系统安全机制的限制。

### 方案一：授予完全磁盘访问权限

最简单的方法是给 `/bin/zsh` 或 `/bin/zsh` 授予完全磁盘访问权限：

1. 打开 **系统设置** → **隐私与安全性** → **完全磁盘访问权限**
2. 点击 🔒 解锁
3. 点击 **+**，按 `Cmd+Shift+G`，输入 `/bin/zsh`
4. 确保开关已开启

### 方案二：使用 Automator 应用（推荐）

如果你不想授予完全磁盘访问权限，可以将脚本包装成 Automator 应用，只授予该应用访问特定目录的权限。

#### 步骤 1：创建 Automator 应用

1. 打开 **Automator**（在"应用程序"中）
2. 选择 **新建文稿** → **应用程序**
3. 在左侧搜索 **"运行 Shell 脚本"**，双击添加
4. 在脚本框中输入：

```zsh
#!/bin/zsh
rm -f ~/Downloads/*.dmg 2>/dev/null
echo "$(date): 清理完成" >> /tmp/clean_dmg.log
```

5. 按 `Cmd+S` 保存到 `/Applications/CleanDMG.app`

#### 步骤 2：首次运行并授权

双击运行 `CleanDMG.app`，系统会弹出权限请求，点击 **"好"** 允许访问 Downloads 目录。

#### 步骤 3：创建 launchd 配置

```zsh
nano ~/Library/LaunchAgents/com.user.clean-dmg.plist
```

配置内容：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.user.clean-dmg</string>
    
    <key>ProgramArguments</key>
    <array>
        <string>/usr/bin/open</string>
        <string>-a</string>
        <string>/Applications/CleanDMG.app</string>
    </array>
    
    <key>StartCalendarInterval</key>
    <dict>
        <key>Hour</key>
        <integer>9</integer>
        <key>Minute</key>
        <integer>0</integer>
    </dict>
</dict>
</plist>
```

#### 步骤 4：加载任务

```z sh
launchctl load ~/Library/LaunchAgents/com.user.clean-dmg.plist
```

### 方案对比

| 方案 | 优点 | 缺点 |
|-----|-----|-----|
| 完全磁盘访问 | 配置简单 | 权限范围大 |
| Automator 应用 | 最小权限原则 | 需额外创建 App |

**推荐**：对于访问用户敏感目录的任务，使用 Automator 应用方案更安全。

希望这篇文章对你有帮助！
