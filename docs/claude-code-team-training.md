# Claude Code 高级功能

> **适用项目**: T113-S4 车载多媒体平台（Linux 5.4 + LVGL）
> **目标读者**: 项目组全体开发人员（3 人）
> **文档定位**: 日常参考手册
> **最后更新**: 2026-02-24

---

## 目录

- [第一部分：开场与基础](#第一部分开场与基础)
  - [1. 为什么我们需要 Claude Code？](#1-为什么我们需要-claude-code)
  - [2. 五分钟快速上手](#2-五分钟快速上手)
- [第二部分：核心功能详解](#第二部分核心功能详解)
  - [3. CLAUDE.md - 项目记忆与团队规范](#3-claudemd---项目记忆与团队规范)
  - [4. Agent 系统 - 智能任务委派](#4-agent-系统---智能任务委派)
  - [5. Skills - 自定义命令与工作流](#5-skills---自定义命令与工作流)
  - [6. Hooks - 自动化守卫](#6-hooks---自动化守卫)
  - [7. MCP Servers - 外部工具集成](#7-mcp-servers---外部工具集成)
  - [8. Plan Mode - 安全代码分析](#8-plan-mode---安全代码分析)
- [第三部分：效率提升实践](#第三部分效率提升实践)
  - [9. Git 集成与团队协作](#9-git-集成与团队协作)
  - [10. 权限管理](#10-权限管理)
  - [11. IDE 集成（VSCode）](#11-ide-集成vscode)
  - [12. 上下文管理](#12-上下文管理)
- [第四部分：动手实践](#第四部分动手实践)
  - [13. Workshop 实战练习](#13-workshop-实战练习)
  - [14. 最佳实践与 FAQ](#14-最佳实践与-faq)
- [第五部分：参考资料](#第五部分参考资料)
  - [15. 快速参考卡](#15-快速参考卡)
  - [16. 附录](#16-附录)

---

# 第一部分：开场与基础

## 1. 为什么我们需要 Claude Code？

### 我们的痛点

做嵌入式多媒体开发，你们一定遇到过这些场景：

**场景一：跨模块代码量大**
> 你在写蓝牙模块的 A2DP 音频流处理，需要理解 BlueZ 协议栈、ALSA 音频子系统、LVGL 播放界面三个模块的交互。光是理清代码调用链，可能就要花半天时间。

**场景二：驱动调试无从下手**
> CSI 摄像头驱动出了问题，倒车影像花屏。你面对 `drivers/media/platform/sunxi-vin/` 下几十个文件，不知道数据流从哪里断开的。

**场景三：构建系统复杂**
> Buildroot 配置改了一个选项，编译报错一屏幕。错误信息指向依赖链的深处，你需要逐层排查。

**场景四：重复性工作多**
> 每次改完代码：`clang-format` 格式化 → 交叉编译 → 生成镜像 → 烧录测试。这套流程每天要跑几十次。

### Claude Code 如何解决

Claude Code 不是一个简单的代码补全工具。它是一个**终端里的 AI 开发助手**，可以：

| 痛点 | Claude Code 解决方案 | 效率提升 |
|------|---------------------|----------|
| 跨模块理解困难 | **Agent 系统**并行分析多个模块 | 半天 → 几分钟 |
| 驱动调试无头绪 | **Explore Agent** 自动追踪代码路径 | 逐文件翻看 → 自动梳理 |
| 构建配置复杂 | **Skills** 一键编译、烧录 | 重复输入命令 → `/build` |
| 格式化/检查遗漏 | **Hooks** 自动触发 | 手动运行 → 全自动 |
| 团队规范不统一 | **CLAUDE.md** 项目级约定 | 口头约定 → 代码级强制 |

### 30 秒演示：生成 LVGL 界面

来看一个实际例子。假设你需要为 FM 收音机创建一个频率选择界面：

```
你: "帮我用 LVGL 创建一个 FM 收音机的频率选择界面，要求：
     - 1024x600 分辨率适配 T113-S4 的 RGB LCD
     - 顶部显示当前频率（大字体）
     - 中间是一个频率滑块（87.5-108.0 MHz）
     - 底部有上一台/下一台/收藏按钮
     - 控件命名遵循 lv_fm_ 前缀"
```

Claude Code 会直接生成完整的 C 代码，遵循项目的命名规范和硬件约束。这就是"懂你项目"的 AI 助手。

---

## 2. 五分钟快速上手

### 安装

```bash
# 方式一：npm 全局安装
npm install -g @anthropic-ai/claude-code

# 方式二：直接运行（推荐）
npx @anthropic-ai/claude-code
```

### 启动方式

```bash
# 在项目目录下启动交互式会话
cd /path/to/t113-multimedia
claude

# 带初始提问启动
claude "解释一下 CSI 摄像头驱动的初始化流程"

# 继续上次的会话
claude -c

# 恢复指定会话
claude -r "bluetooth-debug"
```

### VSCode 中使用

1. 在 VSCode 扩展市场搜索 **"Claude Code"** 并安装
2. 打开命令面板 (`Ctrl+Shift+P`) → 输入 "Claude"
3. 或直接在终端面板中使用 `claude` 命令

### 基本交互

```
你: "读取 drivers/bluetooth/bt_init.c 文件，解释蓝牙初始化流程"
你: "在 apps/media/player.c 中添加暂停功能"
你: "搜索项目中所有使用 lv_obj_create 的地方"
```

### 功能总览

在深入每个功能之前，先看全景图：

| 功能 | 一句话说明 | 关键命令/配置 |
|------|-----------|--------------|
| **CLAUDE.md** | 让 Claude 记住项目规范 | 项目根目录创建 `CLAUDE.md` |
| **Agent** | 委派任务给专门的子代理 | 自动触发 / `.claude/agents/` |
| **Skill** | 自定义可复用命令 | `/命令名` / `.claude/skills/` |
| **Hook** | 自动化触发脚本 | `.claude/settings.json` |
| **MCP** | 连接外部工具 | `claude mcp add` / `.mcp.json` |
| **Plan Mode** | 只读分析不修改 | `claude --permission-mode plan` |
| **Permission** | 权限控制 | `.claude/settings.json` |
| **Git 集成** | 智能提交/PR | `/commit` |
| **上下文管理** | 管理会话内存 | `/context` / `/compact` |

接下来，我们逐个深入。

---

# 第二部分：核心功能详解

> 了解了 Claude Code 的基本用法后，接下来我们深入每个核心功能。从最基础也最重要的 **CLAUDE.md** 开始——它是让 Claude "懂你项目" 的关键。

## 3. CLAUDE.md - 项目记忆与团队规范

### 这是什么？

`CLAUDE.md` 是放在项目根目录的一个 Markdown 文件。Claude Code 每次启动时会**自动加载**它的内容作为指令。你可以把它理解为：**给 AI 助手的项目说明书**。

没有 `CLAUDE.md` 的时候，Claude 不知道你的项目用什么编译工具链、代码规范是什么、硬件有什么限制。有了它，Claude 就变成了一个"懂你项目"的助手。

### 项目实例：T113-S4 项目 CLAUDE.md

下面是为我们项目量身定制的 `CLAUDE.md` 示例：

```markdown
# T113-S4 Vehicle Multimedia Platform

## 项目概述
车载多媒体平台，基于 Allwinner T113-S4 SoC，运行 Linux 5.4 内核。
UI 框架使用 LVGL v8.3，应用层使用 C 语言开发。

## 硬件约束（重要！Claude 必须遵守）
- **SoC**: Allwinner T113-S4 (双核 Cortex-A7, 1.2GHz)
- **RAM**: 256MB DDR3 — 内存敏感，禁止大量动态分配
- **显示**: RGB LCD 1024x600, 双缓冲 framebuffer
- **摄像头**: CSI 接口，CVBS 模拟输入（倒车影像）
- **音频**: 内置 codec, I2S 外接 DAB 解码器
- **无线**: RTL8723DS (WiFi + BT combo 模组)

## 编码规范
- C 代码遵循 Linux kernel coding style（tab 缩进，不用空格）
- LVGL 控件命名规则: `lv_{module}_{widget_type}_{name}`
  - 例: `lv_media_btn_play`, `lv_fm_slider_freq`, `lv_bt_label_device`
- 头文件守卫格式: `__{MODULE}_{FILE}_H__`
- 函数命名: `{module}_{action}_{object}`
  - 例: `media_player_init()`, `bt_scan_start()`, `fm_freq_set()`
- 驱动文件: `drivers/{module}/`
- 应用文件: `apps/{module}/`
- UI 文件: `apps/{module}/ui/`

## 构建系统
- 构建框架: Buildroot 2023.02
- 交叉编译工具链: `arm-linux-gnueabihf-gcc` (gcc 10.3)
- 编译命令:
  ```bash
  source envsetup.sh t113_multimedia
  make -j$(nproc)
  ```
- 单独编译应用: `make app-{module}-rebuild`
- 生成镜像: `output/images/sdcard.img`

## 调试方式
- 串口: `/dev/ttyS0`, 115200 baud
- ADB: `adb shell` (USB OTG 接口)
- GDB 远程调试: `arm-linux-gnueabihf-gdb` + gdbserver

### 初始化项目的 CLAUDE.md

```bash
# 交互式引导创建
/init
```

Claude 会分析你的项目结构，自动生成一份初始的 `CLAUDE.md`。你只需要根据实际情况补充硬件约束和编码规范。

### 进阶：模块化规范管理

当项目变大，一个 `CLAUDE.md` 可能不够用。可以用 `.claude/rules/` 目录分模块管理：

```
.claude/rules/
├── coding-style.md          # 通用编码规范
├── bluetooth.md              # 蓝牙模块特定规范
├── media.md                  # 媒体播放模块规范
├── camera.md                 # 摄像头/倒车模块规范
└── ui/
    └── lvgl-guidelines.md    # LVGL UI 开发规范
```

**路径匹配**：规范文件可以只在编辑特定路径时生效：

```yaml
# .claude/rules/bluetooth.md
---
paths:
  - "drivers/bluetooth/**"
  - "apps/bluetooth/**"
---

# 蓝牙模块开发规范
- 使用 BlueZ 5.x D-Bus API，不直接调用 HCI
- A2DP 音频必须经过 PulseAudio/PipeWire 路由
- 扫描超时设为 10 秒，避免长时间占用射频
- 配对信息存储在 /var/lib/bluetooth/
```

这样，当你编辑蓝牙相关文件时，Claude 会自动加载蓝牙规范；编辑其他模块时不会被无关规范干扰。

### 自动记忆系统

除了手动编写的 `CLAUDE.md`，Claude 还有一个**自动记忆**系统：

```
你: "记住：我们项目的 WiFi 模组固件路径是 /lib/firmware/rtlwifi/"
```

Claude 会自动保存到 `~/.claude/projects/{project}/memory/MEMORY.md`。下次启动时自动加载。

适合记住的内容：
- 反复出现的调试技巧（"WiFi 断连时先检查 dmesg 中的 firmware loading"）
- 项目特有的路径和配置
- 团队偏好（"编译报告用中文"）

```bash
# 手动编辑记忆文件
/memory
```

### 记忆层次总结

| 层级 | 文件位置 | 作用域 | 共享方式 |
|------|---------|--------|---------|
| 项目规范 | `./CLAUDE.md` | 整个项目 | Git 提交共享 |
| 模块规范 | `.claude/rules/*.md` | 特定路径 | Git 提交共享 |
| 个人偏好 | `~/.claude/CLAUDE.md` | 所有项目 | 仅个人 |
| 本地记忆 | `./CLAUDE.local.md` | 当前项目 | 仅个人（自动 gitignore） |
| 自动记忆 | `~/.claude/projects/*/memory/` | 当前项目 | 仅个人 |

**关键点**：`CLAUDE.md` 和 `.claude/rules/` 通过 Git 提交，团队所有人共享同一套规范。这比口头约定或 Wiki 文档强太多了——**规范直接内嵌在开发流程中**。

---

## 4. Agent 系统 - 智能任务委派

### 这是什么？

Agent 系统让你可以把任务**委派**给专门的子代理（subagent）。每个子代理有独立的上下文，可以并行运行，完成后只返回结果摘要。

你可以把它想象成：**你是项目经理，Agent 是你的开发助手。你分配任务，他们独立完成后向你汇报。**

### 内置 Agent 类型

| Agent 类型 | 干什么用 | 特点 | 我们的使用场景 |
|-----------|---------|------|--------------|
| **Explore** | 代码探索 | 只读、快速（Haiku 模型） | 理解陌生代码、查找函数调用链 |
| **Plan** | 方案设计 | 只读、继承主会话模型 | 重构前评估、功能设计 |
| **Bash** | 执行命令 | 独立终端环境 | 编译、测试、部署 |
| **general-purpose** | 完整任务 | 全部工具可用 | 复杂的多步骤任务 |

> 还有一些辅助 Agent（如 `statusline-setup`、`claude-code-guide`）用于特定内部功能，日常开发中较少直接使用。

### 项目实例 1：并行研究多个模块

你需要同时了解三个模块的实现方式。传统方式：逐个模块阅读代码，至少半天。用 Agent：

```
你: "我需要同时了解以下三个模块的架构，请并行分析：
     1. BlueZ 蓝牙协议栈在我们项目中的集成方式，重点是 A2DP profile
     2. WiFi 驱动 RTL8723DS 的电源管理实现，包括休眠唤醒流程
     3. ALSA 音频子系统如何与外部 I2S DAB 解码器对接"
```

Claude Code 会**同时启动 3 个 Explore Agent**，各自独立分析一个模块，最后汇总给你一份完整报告。

**效果**：3 个模块的分析同时进行，总耗时约等于单个模块的分析时间。

### 项目实例 2：Explore Agent 追踪代码路径

倒车影像出了问题，你需要理清 CSI 摄像头数据从传感器到屏幕的完整路径：

```
你: "分析 drivers/media/platform/sunxi-vin/ 目录下的代码，
     帮我梳理 CSI 摄像头数据从传感器采集到 V4L2 输出的完整路径。
     具体要理清：
     1. CSI 控制器初始化流程
     2. 数据从 sensor → CSI → ISP → 内存的 DMA 传输路径
     3. V4L2 buffer 队列管理方式
     4. 与我们倒车应用 apps/camera/ 的交互接口"
```

Explore Agent 会自动在几十个文件中搜索、阅读、追踪调用链，最后给你一份清晰的数据流图。

### 项目实例 3：后台 Agent

你在写代码，同时想让 Claude 在后台检查编译：

```
你: "在后台帮我编译整个项目，如果有编译错误就通知我。
     我继续写代码，你不要打断我。"
```

后台 Agent 会独立运行编译，完成后才回来报告结果。你的前台工作不受影响。

**快捷操作**：正在运行的任务可以按 `Ctrl+B` 转入后台。

### 创建自定义 Agent

对于重复性的专业任务，可以创建项目专用的 Agent。在 `.claude/agents/` 目录下创建 Markdown 文件：

**示例：LVGL 代码审查 Agent**

文件路径：`.claude/agents/lvgl-reviewer.md`

```markdown
---
name: lvgl-reviewer
description: 审查 LVGL 界面代码的性能和内存使用，当需要检查 UI 代码质量时使用
tools: Read, Grep, Glob
model: sonnet
---

你是一个 LVGL 代码审查专家，专门针对 T113-S4 平台（256MB RAM, 双核 A7）。

## 审查清单

### 内存安全
- 每个 `lv_obj_create()` 是否有对应的 `lv_obj_del()` 或明确的生命周期管理
- 是否使用了 `lv_mem_alloc()` 而非标准 `malloc()`
- 图片资源是否在页面切换时正确释放

### 性能优化
- 是否避免了不必要的 `lv_obj_invalidate()` 调用
- 动画帧率是否适配 T113-S4 的渲染能力（建议 ≤30fps）
- 是否使用了 `LV_IMG_CF_TRUE_COLOR_ALPHA` 而非 PNG 直接解码
- 图片资源是否使用 RGB565 格式以匹配显示控制器

### 命名规范
- 控件命名是否遵循 `lv_{module}_{widget}_{name}` 格式
- 样式变量是否使用 `style_{module}_{state}` 格式

输出格式：
1. 发现的问题列表（按严重程度排序）
2. 每个问题的具体位置（文件:行号）
3. 修复建议
```

**使用方式**：

```
你: "用 lvgl-reviewer 检查 apps/fm/ui/ 下的界面代码"
```

Claude 会自动调用这个自定义 Agent 来审查代码。

**示例：硬件兼容性检查 Agent**

文件路径：`.claude/agents/hw-checker.md`

```markdown
---
name: hw-checker
description: 检查代码是否符合 T113-S4 硬件约束
tools: Read, Grep, Glob
model: haiku
---

你是 T113-S4 硬件兼容性检查专家。检查代码是否违反以下硬件约束：

## 内存约束 (256MB DDR3)
- 单次分配超过 1MB 的内存块需要警告
- 检查是否有内存泄漏风险
- 检查是否有不必要的大型全局数组

## 显示约束 (RGB LCD 1024x600)
- 图片分辨率不应超过显示分辨率
- 检查 framebuffer 配置是否为双缓冲

## CPU 约束 (双核 Cortex-A7)
- 检查是否有不必要的忙等待（busy-wait）
- 浮点运算量大的代码需要提醒使用 NEON 优化
```

### Agent 使用技巧

| 场景 | 推荐做法 |
|------|---------|
| 需要快速搜索/理解代码 | 使用 Explore Agent（快、省 token） |
| 需要设计方案但不修改代码 | 使用 Plan Agent（安全） |
| 编译/测试等长时间任务 | 使用后台 Agent（不阻塞） |
| 需要同时了解多个模块 | 并行启动多个 Agent |
| 重复性的代码审查 | 创建自定义 Agent |

### 进阶：Agent Teams（团队协作 Agent）

除了单个子代理，Claude Code 还支持 **Agent Teams** —— 多个 Agent 之间可以协调工作、传递上下文。例如：让一个 Agent 分析代码架构，另一个 Agent 基于分析结果编写测试用例。

这是更高级的用法，感兴趣可以参考官方文档中的 Agent Teams 章节。

---

## 5. Skills - 自定义命令与工作流

### 这是什么？

Skill 是**可复用的指令集**，你用 `/命令名` 就能触发。把它想成：把你反复跟 Claude 说的话打包成一个快捷命令。

比如，你每次编译都要说"用 arm-linux-gnueabihf-gcc 交叉编译，工具链在这里，输出到这个目录..."。有了 Skill，直接 `/build` 就行。

### 创建 Skill

Skill 文件放在 `.claude/skills/` 或 `~/.claude/skills/` 目录下：

```
.claude/skills/
├── build/
│   └── SKILL.md       # 交叉编译
├── flash/
│   └── SKILL.md       # 烧录固件
├── test-module/
│   └── SKILL.md       # 模块测试
└── check-memory/
    └── SKILL.md       # 内存检查
```

### 项目实例 1：/build - 交叉编译

```markdown
# .claude/skills/build/SKILL.md
---
name: build
description: 交叉编译 T113-S4 车载多媒体项目
user-invocable: true
argument-hint: "[module] [clean]"
---

## 任务
执行 T113-S4 项目的交叉编译。

## 参数
- `$0`: 目标模块名（可选，默认编译全部）
  - 可选值: bluetooth, media, camera, fm, dab, wifi, ui
- `$1`: 如果为 "clean" 则先清除编译产物

## 步骤
1. 设置编译环境: `source envsetup.sh t113_multimedia`
2. 如果 $1 == "clean": 执行 `make app-$0-dirclean`
3. 编译:
   - 指定模块: `make app-$0-rebuild -j$(nproc)`
   - 全部编译: `make -j$(nproc)`
4. 报告编译结果:
   - 是否成功
   - 生成的文件列表和大小
   - 如果失败，分析错误原因并给出修复建议

## 注意事项
- 工具链: arm-linux-gnueabihf-gcc
- 编译输出目录: output/t113_multimedia/
- 镜像输出: output/images/sdcard.img
```

**使用方式**：

```bash
/build              # 编译全部
/build bluetooth    # 只编译蓝牙模块
/build media clean  # 清除后重编译媒体模块
```

### 项目实例 2：/flash - 烧录固件

```markdown
# .claude/skills/flash/SKILL.md
---
name: flash
description: 将编译好的固件烧录到 T113-S4 开发板
user-invocable: true
argument-hint: "[method]"
---

## 任务
将编译生成的镜像烧录到 T113-S4 开发板。

## 参数
- `$0`: 烧录方式（可选，默认 adb）
  - `adb`: 通过 ADB push 更新应用（快速，不刷内核）
  - `sd`: 烧录完整 SD 卡镜像
  - `ota`: OTA 增量更新

## 步骤

### ADB 方式（默认）
1. 检查 ADB 连接: `adb devices`
2. 如果设备未连接，提醒用户检查 USB 线和 OTG 接口
3. Push 应用文件: `adb push output/target/usr/bin/{app} /usr/bin/`
4. 重启应用: `adb shell systemctl restart multimedia`
5. 验证: `adb shell ps | grep multimedia`

### SD 卡方式
1. 确认 SD 卡设备: `lsblk`（提示用户确认，防止误写）
2. 烧录: `dd if=output/images/sdcard.img of=/dev/sdX bs=4M status=progress`
3. sync 并安全弹出
```

**使用方式**：

```bash
/flash          # ADB 快速更新
/flash sd       # 烧录 SD 卡
```

### 项目实例 3：/analyze-crash - 崩溃分析

```markdown
# .claude/skills/analyze-crash/SKILL.md
---
name: analyze-crash
description: 分析 T113-S4 设备上的应用崩溃日志
user-invocable: true
argument-hint: "[module]"
---

## 任务
分析设备上的崩溃日志和 coredump 文件。

## 步骤
1. 通过 ADB 获取日志:
   - `adb shell dmesg | tail -100`
   - `adb shell journalctl -u multimedia --no-pager -n 50`
   - 如果有 coredump: `adb pull /var/crash/core.* /tmp/`
2. 如果有 coredump，使用交叉编译 GDB 分析:
   - `arm-linux-gnueabihf-gdb output/target/usr/bin/$0 /tmp/core.*`
   - 获取 backtrace
3. 结合源码分析崩溃原因
4. 输出:
   - 崩溃位置（文件:行号）
   - 调用栈
   - 可能的原因分析
   - 修复建议
```

### 动态上下文

Skill 中可以用 `` !`command` `` 语法嵌入实时信息。Claude 在加载 Skill 时会先执行这些命令：

```markdown
# .claude/skills/status/SKILL.md
---
name: status
description: 查看项目和设备状态
user-invocable: true
---

## 当前状态

### Git 状态
!`git status --short`

### 设备连接
!`adb devices 2>/dev/null || echo "ADB 未安装或无设备连接"`

### 最近编译
!`ls -lt output/images/*.img 2>/dev/null | head -3 || echo "无编译产物"`

## 任务
基于以上信息，给出当前项目状态总结，包括：
1. 代码变更概况
2. 设备连接状态
3. 最新镜像信息
4. 建议的下一步操作
```

**使用**：`/status` → Claude 自动获取实时信息并给出状态报告。

### Skill 参数替换速查

| 变量 | 含义 | 示例 |
|------|------|------|
| `$ARGUMENTS` | 所有参数 | `/build bt clean` → `"bt clean"` |
| `$0` | 第一个参数 | `/build bt clean` → `"bt"` |
| `$1` | 第二个参数 | `/build bt clean` → `"clean"` |
| `$ARGUMENTS[N]` | 第 N 个参数（0 开始） | `/build bt clean` → `$ARGUMENTS[1]` = `"clean"` |

### Skill 高级配置

```yaml
---
name: my-skill
description: 描述（Claude 据此判断何时自动触发）
user-invocable: true          # 用户可通过 /命令 触发
disable-model-invocation: true # 仅手动触发，Claude 不会自动使用
context: fork                  # 在独立子代理中运行
agent: Explore                 # 使用 Explore agent 类型
allowed-tools: Read, Grep      # 限制可用工具
---
```

| 配置 | 效果 |
|------|------|
| `user-invocable: true` | 用户可以用 `/命令名` 触发 |
| `disable-model-invocation: true` | Claude 不会自动触发，只能手动 |
| `context: fork` | 在独立上下文中运行，不污染主会话 |
| `model: sonnet` | 覆盖模型（haiku/sonnet/opus） |
| `allowed-tools` | 限制 Skill 可以使用的工具 |

---

## 6. Hooks - 自动化守卫

### 这是什么？

Hooks 是在 Claude Code 工作流程的**特定时间点自动执行**的脚本。跟 Git hooks 类似，但作用在 Claude Code 的操作上。

把它想成：**你给 Claude Code 装了一套自动化规则。编辑文件后自动格式化、修改关键文件时自动拦截、编译完成后自动通知。**

### Hook 事件一览

| 事件 | 触发时机 | 能否阻止操作？ |
|------|---------|-------------|
| `SessionStart` | 会话开始/恢复 | 否 |
| `UserPromptSubmit` | 用户提交提问前 | 可以（exit 2 阻止处理） |
| `PreToolUse` | 工具执行**前** | 可以（exit 2 阻止） |
| `PermissionRequest` | 权限对话框出现时 | 否 |
| `PostToolUse` | 工具执行**后**（成功） | 否 |
| `PostToolUseFailure` | 工具执行**后**（失败） | 否 |
| `Notification` | Claude 需要注意 | 否 |
| `SubagentStart` | 子代理启动 | 否 |
| `SubagentStop` | 子代理完成 | 否 |
| `Stop` | Claude 完成回复 | 可以（exit 2 让 Claude 继续） |
| `PreCompact` | 上下文压缩之前 | 否 |
| `SessionEnd` | 会话终止 | 否 |

### 配置方式

在 `.claude/settings.json` 或 `~/.claude/settings.json` 中配置：

```bash
# 交互式创建
/hooks
```

### 项目实例 1：自动格式化 C 代码

每次 Claude 编辑或创建 `.c` / `.h` 文件后，自动运行 `clang-format`：

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "INPUT=$(cat); FILE=$(echo $INPUT | jq -r '.tool_input.file_path // empty'); if [[ \"$FILE\" == *.c || \"$FILE\" == *.h ]]; then clang-format -i -style=file \"$FILE\" && echo \"已格式化: $FILE\"; fi"
          }
        ]
      }
    ]
  }
}
```

**效果**：你再也不用手动运行 `clang-format` 了。Claude 每次改完代码，格式化自动生效。

### 项目实例 2：保护关键文件

阻止 Claude 修改设备树（`.dts`/`.dtsi`）和内核配置（`defconfig`）——这些文件改错了后果很严重：

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "INPUT=$(cat); FILE=$(echo $INPUT | jq -r '.tool_input.file_path // empty'); for P in '.dts' '.dtsi' 'defconfig' '.config' 'Kconfig'; do if [[ \"$FILE\" == *\"$P\" ]]; then echo \"BLOCKED: $FILE 是受保护文件（设备树/内核配置），需要人工审查后修改\" >&2; exit 2; fi; done; exit 0"
          }
        ]
      }
    ]
  }
}
```

**效果**：当 Claude 试图修改设备树文件时，操作会被**自动拦截**，Claude 会收到提示并改用其他方式（比如只给出修改建议，让你手动操作）。

### 项目实例 3：自动静态分析

编辑 C 文件后自动运行 `cppcheck`：

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "INPUT=$(cat); FILE=$(echo $INPUT | jq -r '.tool_input.file_path // empty'); if [[ \"$FILE\" == *.c ]]; then RESULT=$(cppcheck --enable=warning,performance \"$FILE\" 2>&1); if [ -n \"$RESULT\" ]; then echo \"cppcheck 发现问题:\n$RESULT\"; fi; fi"
          }
        ]
      }
    ]
  }
}
```

**效果**：每次改 C 代码后，cppcheck 自动运行。如果发现潜在问题（内存泄漏、未初始化变量等），Claude 会看到反馈并主动修复。

### 项目实例 4：会话启动提醒

每次打开 Claude Code 时，自动提醒重要注意事项：

```json
{
  "hooks": {
    "SessionStart": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "echo '提醒：\n1. 当前分支: '$(git branch --show-current)'\n2. 编译前请确认 envsetup.sh 已 source\n3. 修改驱动后需要重新编译内核模块\n4. 倒车影像模块正在重构中，修改前先跟李四确认'"
          }
        ]
      }
    ]
  }
}
```

### 项目实例 5：编译完成桌面通知（Windows）

> 注意：此示例可能需要根据实际 Windows 环境调整。

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "INPUT=$(cat); CMD=$(echo $INPUT | jq -r '.tool_input.command // empty'); if echo \"$CMD\" | grep -qE '^\\s*make\\b'; then powershell -Command \"Add-Type -AssemblyName System.Windows.Forms; [System.Windows.Forms.MessageBox]::Show('编译完成','Claude Code 通知')\"; fi"
          }
        ]
      }
    ]
  }
}
```

### 三种 Hook 类型

| 类型 | 说明 | 适用场景 |
|------|------|---------|
| `command` | 执行 shell 命令 | 格式化、文件检查、通知 |
| `prompt` | 发送 prompt 给 AI 模型进行单轮评估 | 需要判断力的检查（如"任务是否完成？"） |
| `agent` | 启动子代理验证条件 | 需要读文件、跑命令的复杂验证 |

**prompt 类型示例**（检查任务是否完成）：
```json
{
  "type": "prompt",
  "prompt": "检查所有请求的任务是否都已完成。返回 {\"ok\": true} 或 {\"ok\": false, \"reason\": \"未完成的原因\"}"
}
```

**agent 类型示例**（编译验证）：
```json
{
  "type": "agent",
  "prompt": "运行交叉编译并验证没有编译错误。",
  "timeout": 120
}
```

### Hook 输入/输出机制

每个 Hook 脚本通过 **stdin** 接收 JSON 数据：

```json
{
  "session_id": "abc123",
  "cwd": "/path/to/project",
  "hook_event_name": "PreToolUse",
  "tool_name": "Edit",
  "tool_input": {
    "file_path": "drivers/bluetooth/bt_init.c",
    "old_string": "...",
    "new_string": "..."
  }
}
```

**退出码控制**：
| 退出码 | 效果 |
|--------|------|
| `0` | 正常继续（stdout 内容会加入上下文） |
| `2` | **阻止操作**（stderr 内容作为反馈给 Claude） |
| 其他 | 继续执行，stderr 记录到日志 |

**高级方式：JSON 输出控制**（推荐用于 PreToolUse）：

```json
{
  "hookSpecificOutput": {
    "hookEventName": "PreToolUse",
    "permissionDecision": "deny",
    "permissionDecisionReason": "受保护文件，需要人工审查"
  }
}
```

`permissionDecision` 支持 `"allow"`、`"deny"`、`"ask"` 三种决策。

### 配置 Hook 的位置

| 位置 | 生效范围 | 共享方式 |
|------|---------|---------|
| `.claude/settings.json` | 当前项目 | Git 提交共享 |
| `.claude/settings.local.json` | 当前项目 | 仅个人 |
| `~/.claude/settings.json` | 所有项目 | 仅个人 |

---

## 7. MCP Servers - 外部工具集成

### 这是什么？

MCP（Model Context Protocol）让 Claude Code 可以**连接外部工具和服务**。通过 MCP，Claude 可以直接操作 GitLab、JIRA、数据库、API 等，不需要你手动复制粘贴。

### 添加 MCP Server

```bash
# HTTP 方式（推荐）
claude mcp add --transport http gitlab https://mcp.gitlab.com

# 带认证
claude mcp add --transport http gitlab https://gitlab.company.com/mcp \
  --header "Authorization: Bearer YOUR_TOKEN"

# stdio 方式（本地工具）
claude mcp add --transport stdio myserver -- npx @some/mcp-server

# Windows 上的 stdio（需要 cmd 包装）
claude mcp add --transport stdio myserver -- cmd /c npx -y @some/mcp-server
```

### 项目实例：团队工具集成

**GitLab 集成**（代码托管 + CI/CD）：

```bash
claude mcp add --transport http gitlab https://mcp.gitlab.com
```

集成后可以直接在 Claude Code 中：
```
你: "查看 GitLab 上蓝牙模块相关的 issue"
你: "创建一个 merge request，标题是 '修复倒车影像花屏问题'"
你: "检查 CI pipeline 的编译状态"
```

### 团队共享 MCP 配置

将 MCP 配置写入项目根目录的 `.mcp.json`，提交到 Git：

```json
{
  "mcpServers": {
    "gitlab": {
      "type": "http",
      "url": "${GITLAB_MCP_URL:-https://gitlab.company.com/mcp}",
      "headers": {
        "Authorization": "Bearer ${GITLAB_TOKEN}"
      }
    }
  }
}
```

每个团队成员只需在自己的环境中设置 `GITLAB_TOKEN` 环境变量即可。

### 管理 MCP Server

```bash
claude mcp list              # 列出所有已配置的 server
claude mcp get gitlab        # 查看指定 server 详情
claude mcp remove gitlab     # 移除 server
/mcp                         # 在会话中检查状态和认证
```

### MCP 作用域

| 作用域 | 配置位置 | 使用场景 |
|--------|---------|---------|
| `local`（默认） | `~/.claude.json` | 个人实验 |
| `project` | `.mcp.json` | 团队共享 |
| `user` | `~/.claude.json` | 跨项目通用 |

```bash
# 设置为项目级（团队共享）
claude mcp add --transport http --scope project gitlab https://...
```

---

## 8. Plan Mode - 安全代码分析

### 这是什么？

Plan Mode 是 Claude Code 的**只读模式**。在这个模式下，Claude 只能阅读和分析代码，**不能修改任何文件、不能执行任何命令**。

当你想在修改代码之前先**充分理解现有架构**，或者想让 Claude 给出**方案建议但不动手改**时，就用 Plan Mode。

### 启动 Plan Mode

```bash
# 启动时指定
claude --permission-mode plan

# 会话中切换
Shift+Tab  # 循环切换权限模式
```

### 项目实例：重构媒体播放器

你准备给媒体播放器添加 DAB 数字广播支持，但不确定现有架构能否直接扩展。用 Plan Mode 先分析：

```bash
claude --permission-mode plan
```

```
你: "分析 apps/media/ 目录下的播放器架构，帮我评估：

     1. 当前的 GStreamer pipeline 是如何构建的？
        - 音频解码 pipeline
        - 视频渲染 pipeline
        - 是否支持动态切换数据源？

     2. 线程模型是怎样的？
        - 解码线程、渲染线程、UI 线程的关系
        - 线程间如何同步？

     3. 如果要添加 DAB 数据源，需要修改哪些模块？
        - 预估改动量（文件数、代码行数）
        - 有什么风险点？

     4. 给出两种可能的架构方案，分析各自的优缺点

     注意：只分析，不要修改任何文件！"
```

**效果**：Claude 会深入阅读所有相关代码，输出一份详细的架构分析报告和方案对比。你确认方案后，再退出 Plan Mode 开始实施。

### Plan Mode 的价值

| 场景 | 用 Plan Mode |
|------|-------------|
| 重构前评估影响范围 | 分析改动涉及的文件和风险 |
| 理解陌生模块 | 快速梳理代码架构 |
| 方案设计 | 对比多种实现方案 |
| Code Review | 分析别人提交的代码 |
| 学习 | 理解开源驱动的实现原理 |

### Plan Mode vs Explore Agent

| | Plan Mode | Explore Agent |
|--|-----------|---------------|
| 作用域 | 整个会话 | 单个任务 |
| 谁在分析 | 主 Claude（Opus 模型） | 子代理（Haiku 模型） |
| 分析深度 | 深度分析，可多轮对话 | 快速扫描，单次返回 |
| 适合场景 | 复杂架构分析 | 快速查找和定位 |

---

# 第三部分：效率提升实践

> 掌握了核心功能后，我们来看如何在日常开发中把这些功能**组合起来**使用，进一步提升团队整体效率。

## 9. Git 集成与团队协作

### 智能提交

当你完成代码修改后：

```
你: "帮我提交这些修改"
```

Claude 会：
1. 自动运行 `git status` 和 `git diff` 查看变更
2. 分析修改内容，生成有意义的 commit message
3. 执行 `git add` 和 `git commit`

### 创建 Pull Request / Merge Request

```
你: "创建一个 PR，包含蓝牙 A2DP 音频流的修复"
```

Claude 会：
1. 分析当前分支的所有 commit
2. 生成 PR 标题和详细描述
3. 使用 `gh pr create` 创建 PR

### 代码审查

```
你: "review 一下 PR #42"
```

Claude 会阅读 PR 的所有变更，给出审查意见。

### 多人协作最佳实践

1. **CLAUDE.md 统一规范**：所有人用同一份 CLAUDE.md，确保 Claude 在每个人的环境中行为一致
2. **分支命名规范写入 CLAUDE.md**：
   ```markdown
   ## Git 规范
   - 分支命名: feature/{module}/{description}
   - 例: feature/bluetooth/a2dp-audio-fix
   - Commit message 使用中文，格式: [{module}] {description}
   - 例: [bluetooth] 修复 A2DP 音频流断连重连问题
   ```
3. **MCP 集成代码托管**：GitLab/GitHub MCP 让 Claude 直接操作 issue 和 MR

---

## 10. 权限管理

### 权限模式

| 模式 | 说明 | 适用场景 |
|------|------|---------|
| `default` | 每个操作首次使用时询问 | 日常开发 |
| `acceptEdits` | 自动允许文件编辑 | 信任 Claude 的修改 |
| `plan` | 完全只读 | 分析代码 |
| `dontAsk` | 自动拒绝未预授权的操作 | 自动化脚本 |
| `bypassPermissions` | 跳过所有权限检查 | CI/CD 环境（慎用！） |

### 项目实例：配置团队权限

在 `.claude/settings.json` 中配置（Git 提交共享）：

```json
{
  "permissions": {
    "allow": [
      "Read",
      "Bash(make *)",
      "Bash(arm-linux-gnueabihf-*)",
      "Bash(adb *)",
      "Bash(git status)",
      "Bash(git diff *)",
      "Bash(git log *)"
    ],
    "deny": [
      "Bash(rm -rf *)",
      "Bash(dd *)",
      "Bash(git push --force *)",
      "Bash(git reset --hard *)"
    ]
  }
}
```

**解读**：
- **允许**：编译、交叉编译工具链、ADB、只读 Git 操作
- **拒绝**：危险的删除操作、dd 写盘、force push、hard reset

### 权限匹配规则

```
Bash(npm run *)     # 匹配 "npm run build", "npm run test" 等
Bash(make *)        # 匹配所有 make 命令
Edit(/src/**/*.ts)  # 匹配 src 下所有 .ts 文件的编辑
Read(~/.env)        # 匹配 home 目录下的 .env 文件
```

---

## 11. IDE 集成（VSCode）

### 安装

1. VSCode 扩展市场搜索 **"Claude Code"**
2. 安装后会自动连接 Claude Code CLI

### 核心能力

| 功能 | 说明 |
|------|------|
| 内联编辑 | 在代码旁边直接与 Claude 对话 |
| 多会话 | 同时开多个 Claude 会话 |
| 终端集成 | 在 VSCode 终端中使用 `claude` |
| 文件引用 | 选中代码后直接发送给 Claude |

### 项目实例：调试 LVGL 界面

1. 在 VSCode 中打开 `apps/fm/ui/fm_screen.c`
2. 选中有问题的 LVGL 代码
3. 右键 → "Ask Claude" 或快捷键
4. 输入："这段 LVGL 代码为什么在 T113-S4 上渲染很慢？分析性能瓶颈。"

Claude 会结合项目的 `CLAUDE.md`（知道硬件约束是 256MB RAM + 双核 A7）给出针对性的优化建议。

### 常用快捷键

| 快捷键 | 功能 |
|--------|------|
| `Enter` | 发送消息 |
| `Ctrl+C` | 取消输入 |
| `Ctrl+R` | 搜索历史 |
| `Ctrl+T` | 切换任务列表 |
| `Ctrl+O` | 切换详细输出 |
| `Ctrl+P` | 选择模型 |
| `Shift+Tab` | 切换权限模式 |
| `Ctrl+B` | 将当前任务转入后台 |

### 自定义快捷键

编辑 `~/.claude/keybindings.json`：

```json
{
  "$schema": "https://www.schemastore.org/claude-code-keybindings.json",
  "bindings": [
    {
      "context": "Chat",
      "bindings": {
        "ctrl+e": "chat:externalEditor",
        "ctrl+shift+b": "chat:submit"
      }
    }
  ]
}
```

或使用交互式设置：
```bash
/keybindings
```

---

## 12. 上下文管理

### 为什么要管理上下文？

Claude Code 有一个上下文窗口（类似工作内存）。当对话太长或涉及太多文件时，旧的内容会被压缩。理解上下文管理可以让你更高效地工作。

### 查看上下文使用

```bash
/context    # 查看当前上下文使用情况
/cost       # 查看 token 消耗和费用
```

### 手动压缩

```bash
/compact    # 压缩历史对话，释放上下文空间
```

当你感觉 Claude 开始"忘记"之前的内容时，可以手动压缩。Claude 会保留重要信息，丢弃细节。

### 用 Agent 隔离大量输出

**问题场景**：交叉编译输出几千行日志，占满上下文。

**解决方案**：让 Agent 在子上下文中处理：

```
你: "在后台编译整个项目，只告诉我是否成功。
     如果失败，只返回错误摘要，不要返回完整日志。"
```

Agent 独立处理编译输出，只把摘要返回主会话，不会污染你的上下文。

### 上下文管理最佳实践

| 做法 | 效果 |
|------|------|
| 长任务结束后用 `/compact` | 释放空间，保留重要上下文 |
| 编译/测试输出交给 Agent | 隔离大量输出 |
| CLAUDE.md 保持精简 | 减少每次启动的 token 消耗 |
| 用 `.claude/rules/` 按路径加载 | 只在需要时加载对应规范 |
| 新任务开始时用 `/clear` | 完全重新开始 |

---

# 第四部分：动手实践

> 理论讲完了，接下来是**动手时间**。以下练习会帮你把刚才学到的内容真正落地到我们的项目中。

## 13. Workshop 实战练习

> 以下 6 个练习设计为团队培训的动手环节。每个练习有明确的目标、步骤和验证方式。
>
> **时间安排**：总计约 80 分钟（不含讨论和休息）
> - 练习 1-3（40 分钟）→ 休息 10 分钟 → 练习 4-6（40 分钟）
> - 如果时间有限，优先完成**练习 1、3、5**（覆盖 CLAUDE.md、Skill、Plan Mode 三个核心功能）

### 练习 1：创建项目 CLAUDE.md（10 分钟）

**目标**：为 T113-S4 项目创建一份团队共享的 CLAUDE.md

**步骤**：

1. 在项目根目录启动 Claude Code：
   ```bash
   cd /path/to/t113-multimedia
   claude
   ```

2. 使用 `/init` 让 Claude 自动生成初始 CLAUDE.md：
   ```
   /init
   ```

3. 根据提示补充以下信息：
   - 项目名称和描述
   - 硬件约束（SoC、RAM、显示分辨率）
   - 编码规范（命名规则、目录结构）
   - 构建命令

4. 检查生成的 CLAUDE.md 内容

**验证**：退出 Claude Code，重新启动，随便问一个问题。看 Claude 是否记住了项目规范（比如问它"我们用什么编译器？"）。

---

### 练习 2：使用 Explore Agent 分析代码（15 分钟）

**目标**：用 Agent 快速理解一个你不熟悉的模块

**步骤**：

1. 选择一个你最不熟悉的模块（蓝牙/媒体/摄像头/FM）

2. 让 Claude 用 Explore Agent 分析：
   ```
   你: "帮我分析 [模块目录] 的代码架构：
        1. 模块的入口函数是什么？
        2. 主要的数据流是怎样的？
        3. 有哪些对外暴露的 API？
        4. 依赖了哪些外部库？"
   ```

3. 尝试并行分析（让 Claude 同时分析两个模块）：
   ```
   你: "同时帮我分析蓝牙模块和 WiFi 模块的初始化流程，
        找出它们是否有共享的硬件资源冲突。"
   ```

**验证**：Claude 应该返回清晰的架构描述，包括关键文件路径和函数名。对比你自己阅读代码的时间。

---

### 练习 3：创建 /build Skill（15 分钟）

**目标**：创建一个一键编译的自定义 Skill

**步骤**：

1. 创建 Skill 目录：
   ```bash
   mkdir -p .claude/skills/build
   ```

2. 创建 SKILL.md 文件（参考本文档第 5 节的示例）

3. 在 Claude Code 中测试：
   ```
   /build
   /build bluetooth
   /build media clean
   ```

4. 根据实际编译结果调整 Skill 内容

**验证**：`/build` 能够正确触发交叉编译，编译结果正确报告。

---

### 练习 4：配置代码格式化 Hook（10 分钟）

**目标**：让 Claude 编辑 C 代码后自动运行 clang-format

**步骤**：

1. 使用交互式配置：
   ```
   /hooks
   ```

2. 或手动编辑 `.claude/settings.json`，添加 PostToolUse hook（参考第 6 节的示例）

3. 测试：让 Claude 编辑一个 C 文件
   ```
   你: "在 apps/fm/fm_main.c 中添加一个函数 fm_freq_set(float freq)"
   ```

4. 检查生成的代码是否已自动格式化

**验证**：Hook 输出显示"已格式化"，代码风格符合 `.clang-format` 配置。

---

### 练习 5：Plan Mode 代码审查（15 分钟）

**目标**：用 Plan Mode 安全地分析代码架构

**步骤**：

1. 启动 Plan Mode：
   ```bash
   claude --permission-mode plan
   ```

2. 让 Claude 分析一个模块：
   ```
   你: "分析 apps/media/ 的架构，评估以下方面：
        1. 代码结构是否合理？
        2. 有没有明显的性能问题？
        3. 如果要添加新的音频源（DAB），需要改什么？
        给出详细的分析报告。"
   ```

3. 确认 Claude 没有修改任何文件

**验证**：Claude 输出详细的分析报告。运行 `git status` 确认没有文件被修改。

---

### 练习 6：团队协作流程模拟（15 分钟）

**目标**：完成一个完整的功能开发 → 提交 → PR 流程

**步骤**：

1. 创建功能分支：
   ```
   你: "创建分支 feature/fm/volume-control"
   ```

2. 让 Claude 实现一个小功能：
   ```
   你: "在 FM 模块中添加音量控制功能：
        - 在 LVGL 界面添加音量滑块
        - 通过 ALSA API 控制实际音量"
   ```

3. 提交代码：
   ```
   你: "提交这些修改"
   ```

4. （可选）创建 PR：
   ```
   你: "创建一个 merge request"
   ```

**验证**：完整的 Git 历史记录，有意义的 commit message，PR 描述清晰。

---

## 14. 最佳实践与 FAQ

### 嵌入式开发的 10 个 Claude Code 高效习惯

1. **先写 CLAUDE.md，再写代码**
   - 把硬件约束、编码规范、构建命令写清楚
   - Claude 会在每次交互中遵守这些规范

2. **用 Plan Mode 分析，确认后再实施**
   - 修改驱动或架构前，先用只读模式评估影响
   - 降低改错的风险

3. **用 Agent 并行研究**
   - 需要理解多个模块时，启动并行 Agent
   - 3 个模块的分析时间 ≈ 1 个模块的时间

4. **把重复命令做成 Skill**
   - `/build`, `/flash`, `/test` — 一个命令搞定
   - 团队共享 Skill，统一工作流

5. **用 Hook 自动化质量守卫**
   - 代码格式化：PostToolUse + clang-format
   - 关键文件保护：PreToolUse + 白名单

6. **编译输出交给 Agent**
   - 大量日志不要在主会话中处理
   - 让 Agent 在后台编译，只返回结果摘要

7. **定期 `/compact`**
   - 长时间工作后压缩上下文
   - 保持 Claude 的"工作记忆"清晰

8. **用 `.claude/rules/` 管理模块规范**
   - 不同模块不同规范
   - 只在编辑相关文件时加载

9. **用自动记忆保存调试经验**
   - "记住：WiFi 断连时检查 firmware loading"
   - 下次遇到同样问题，Claude 会直接想到

10. **Git 规范写入 CLAUDE.md**
    - 分支命名、commit 格式统一
    - Claude 自动遵守

### 常见问题 FAQ

**Q: Claude 生成的代码不符合我们的编码规范怎么办？**
> 完善 CLAUDE.md 中的编码规范部分。越具体越好。配合 Hook 自动格式化。

**Q: Claude 修改了不该改的文件怎么办？**
> 1. 配置 PreToolUse Hook 保护关键文件
> 2. 在 permissions 的 deny 列表中添加规则
> 3. 使用 Plan Mode 只分析不修改

**Q: 上下文不够用怎么办？**
> 1. 用 `/compact` 压缩
> 2. 用 Agent 隔离大量输出
> 3. 新任务用 `/clear` 重新开始
> 4. 保持 CLAUDE.md 精简

**Q: 如何让团队统一使用相同的 Claude Code 配置？**
> 将以下文件提交到 Git：
> - `CLAUDE.md` - 项目规范
> - `.claude/settings.json` - 权限和 Hook 配置
> - `.claude/skills/` - 自定义 Skill
> - `.claude/agents/` - 自定义 Agent
> - `.claude/rules/` - 模块规范
> - `.mcp.json` - MCP 配置

**Q: Claude Code 在 Windows 上有什么需要注意的？**
> 1. MCP stdio 模式需要 `cmd /c` 包装
> 2. Hook 脚本考虑跨平台兼容（bash vs powershell）
> 3. 路径使用 `/` 或 `\\`

**Q: 能不能把我们团队的 Skill、Agent、Hook 打包分享？**
> 可以！Claude Code 支持 **Plugin 系统**。你可以将 skills、agents、hooks、MCP servers 打包成一个 Plugin，方便在团队或跨项目间复用。使用 `/plugin` 命令管理插件。

**Q: 如何控制 Claude Code 的费用？**
> 1. 用 `--max-budget-usd 5.00` 设置费用上限
> 2. 简单任务用 Haiku 模型（更便宜）
> 3. 用 Explore Agent（Haiku）做初步调查，复杂分析再用 Opus
> 4. `/cost` 随时查看当前消耗

---

# 第五部分：参考资料

> 最后是日常使用的速查手册。

## 15. 快速参考卡

### 启动命令

| 命令 | 说明 |
|------|------|
| `claude` | 启动交互式会话 |
| `claude "问题"` | 带初始问题启动 |
| `claude -c` | 继续上次会话 |
| `claude -r "名称"` | 恢复指定会话 |
| `claude --permission-mode plan` | Plan Mode 启动 |
| `claude -p "问题"` | 非交互模式（CI/CD 用） |

### 会话内命令

| 命令 | 说明 |
|------|------|
| `/help` | 帮助 |
| `/init` | 初始化 CLAUDE.md |
| `/context` | 查看上下文 |
| `/cost` | 查看费用 |
| `/compact` | 压缩上下文 |
| `/clear` | 清空重来 |
| `/memory` | 编辑记忆 |
| `/hooks` | 管理 Hook |
| `/agents` | 管理 Agent |
| `/permissions` | 查看权限 |
| `/mcp` | MCP 状态 |
| `/keybindings` | 管理快捷键 |
| `/vim` | Vim 模式 |
| `/fast` | 切换快速模式 |

### 快捷键

| 快捷键 | 功能 |
|--------|------|
| `Enter` | 发送 |
| `Ctrl+C` | 取消 |
| `Ctrl+D` | 退出 |
| `Ctrl+R` | 搜索历史 |
| `Ctrl+T` | 任务列表 |
| `Ctrl+O` | 详细输出 |
| `Ctrl+P` | 选择模型 |
| `Ctrl+B` | 转入后台 |
| `Shift+Tab` | 切换权限模式 |

### 配置文件位置

| 文件 | 路径 | 用途 |
|------|------|------|
| 项目规范 | `./CLAUDE.md` | AI 指令 |
| 模块规范 | `.claude/rules/*.md` | 路径级规范 |
| 项目设置 | `.claude/settings.json` | 权限/Hook |
| 个人设置 | `~/.claude/settings.json` | 个人偏好 |
| 自定义 Skill | `.claude/skills/*/SKILL.md` | 快捷命令 |
| 自定义 Agent | `.claude/agents/*.md` | 专用代理 |
| MCP 配置 | `.mcp.json` | 外部工具 |
| 快捷键 | `~/.claude/keybindings.json` | 键盘映射 |
| 自动记忆 | `~/.claude/projects/*/memory/` | AI 记忆 |

### MCP 管理命令

| 命令 | 说明 |
|------|------|
| `claude mcp add --transport http name url` | 添加 HTTP server |
| `claude mcp add --transport stdio name -- cmd` | 添加本地 server |
| `claude mcp list` | 列出所有 server |
| `claude mcp remove name` | 移除 server |

### 常用 CLI 参数

| 参数 | 说明 |
|------|------|
| `--model sonnet` | 选择模型 |
| `--permission-mode plan` | 权限模式 |
| `--max-budget-usd 5` | 费用上限 |
| `--max-turns 10` | 最大轮次 |
| `--allowedTools "..."` | 预授权工具 |
| `--disallowedTools "..."` | 禁用工具 |
| `--add-dir ../other` | 添加目录 |
| `--verbose` | 详细输出 |

---

## 16. 附录

### A. 团队 CLAUDE.md 模板

```markdown
# [项目名称]

## 项目概述
[一句话描述项目]

## 硬件平台
- SoC: [型号]
- RAM: [容量]
- 显示: [分辨率和接口]
- 其他外设: [列出关键外设]

## 编码规范
- 语言: [C/C++/其他]
- 代码风格: [内核风格/Google 风格/其他]
- 命名规则:
  - 文件: [规则]
  - 函数: [规则]
  - 变量: [规则]
- 目录结构:
  - drivers/ - 驱动代码
  - apps/ - 应用代码
  - libs/ - 公共库

## 构建系统
- 框架: [Buildroot/Yocto/其他]
- 工具链: [交叉编译器路径]
- 编译命令: [具体命令]
- 输出目录: [路径]

## 调试方式
- 串口: [设备和波特率]
- 网络: [SSH/ADB]
- 调试器: [GDB 配置]

## Git 规范
- 分支命名: [规则]
- Commit 格式: [规则]
- PR 流程: [描述]

## 模块负责人
- [模块1]: [姓名]
- [模块2]: [姓名]
```

### B. Hook 配置模板

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "# 在此添加编辑后的自动化操作"
          }
        ]
      }
    ],
    "PreToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "# 在此添加编辑前的检查逻辑"
          }
        ]
      }
    ],
    "SessionStart": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "echo '项目提醒信息'"
          }
        ]
      }
    ]
  }
}
```

### C. Skill 模板

```markdown
# .claude/skills/{skill-name}/SKILL.md
---
name: skill-name
description: 描述这个 skill 做什么
user-invocable: true
argument-hint: "[arg1] [arg2]"
---

## 任务
[描述任务目标]

## 参数
- `$0`: [第一个参数说明]
- `$1`: [第二个参数说明]

## 步骤
1. [步骤 1]
2. [步骤 2]
3. [步骤 3]

## 注意事项
- [约束条件]
```

### D. Agent 模板

文件路径：`.claude/agents/{agent-name}.md`

```markdown
---
name: agent-name
description: 描述什么时候使用这个 agent
tools: Read, Grep, Glob
model: sonnet
---

你是一个 [角色描述]。

## 任务
[描述 agent 要完成什么]

## 检查项
1. [检查项 1]
2. [检查项 2]
3. [检查项 3]

## 输出格式
[描述输出格式]
```

### E. 官方资源

- Claude Code 官方文档: https://docs.anthropic.com/en/docs/claude-code
- Claude Code GitHub: https://github.com/anthropics/claude-code
- MCP 协议: https://modelcontextprotocol.io
- LVGL 文档: https://docs.lvgl.io
- Allwinner T113-S4 文档: 参考供应商提供的 SDK 文档

---

> **文档结束**
>
> 下次主题预告：如何用 Claude Code + MCP 打通从需求到部署的全流程自动化。
