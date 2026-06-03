# Huiyu-SafeAi

轻量级AI安全卫士，用于安装/下载命令。它能阻止已知的恶意软件包，验证软件包身份，并扫描可疑代码——所有操作均在1秒内完成。

[English](README.md) · [中文](README.zh.md)

---

<div align="center">

**拦截恶意包 —— 拦截 deepseek-py（仿冒 DeepSeek 的恶意包，已知 typosquatting）**

![拦截: deepseek-py](assets/blocked-deepseek-py.jpg)

<br>

**验证安全仓库 —— 放行 huiyu9144/Huiyu-Pi（个人仓库，代码嗅探通过，无恶意行为）**

![安全: Huiyu-Pi](assets/safe-huiyu-pi.jpg)

</div>

---

## 设计理念

### 对AI速度零影响

Huiyu-SafeAi 被设计为一个**提示词级别的安全护栏**，而非运行时扫描器：

- **无网络调用** — 所有检查在内存中完成，基于嵌入式列表
- **不执行任何命令** — 不运行 `npm audit`、`npm view` 或任何外部工具
- **不扫描文件** — 代码嗅探最多只读取3个文件，且仅在需要时触发
- **对安全包无开销** — 90%+ 的安装在第1步或第2步直接放行

该 skill 仅在检测到安装/下载命令时激活。如果你只是聊天、写代码或做其他事情——它完全静默。

### 三级检查流程

支持**所有生态系统** — `git clone`、`npm install`、`npx`、`pip install`、`cargo install`、`yarn add`、`pnpm add`。

```
[用户运行: git clone / npm install / pip install / ...]
         |
   第1步: 黑名单?  --> 是 --> 拦截 (红色) + 详细解释
         | 否
   第2步: 可信?    --> 是 --> 安全 (绿色), 直接放行
         | 未知
   第3步: 代码嗅探  --> 有恶意 --> 拦截 (红色)
         | 安全
         v
      警告 (黄色)
```

**第1步（黑名单）** — 即时检查68+个已知恶意包和仓库。如命中，立即拦截并附带完整解释：为什么危险、实际发生了什么攻击、信息来自哪里。

**第2步（信任验证）** — 如果不在黑名单中，检查来源是否可信：

- **GitHub**: 是否属于受信组织（60+个已列出）？是否有1,000+星标？
- **npm/PyPI**: 是否有10,000+周下载量？是否为官方包名？
- **已知包**: 70+个受信包直接放行（express、react、pandas、lodash 等）

如可信 → 绿灯，**零后续处理**。

**第3步（代码嗅探）** — 仅对真正未知的来源执行。即使执行，也最多只读取3个文件：

- **GitHub 仓库**: `package.json` 安装脚本、`setup.py`/`pyproject.toml`、`Makefile`/`Dockerfile`
- **npm 包**: `postinstall` 脚本、依赖链、源代码模式
- **任何来源**: 快速查看一个源文件是否有混淆模式

---

## 为什么做这个：一次真实攻击

2026年5月11日，我遭遇了一次针对AI开发者的真实供应链攻击。这次事件是 Huiyu-SafeAi 诞生的直接原因。

### 发生了什么

我需要部署 **DeepSeek-TUI**，一个流行的开源终端界面（24.1k星标）。正确仓库是 `github.com/Hmbown/DeepSeek-TUI`。然而，一个**假冒仓库**存在于 `github.com/DeepSeek-TUI/DeepSeek-TUI`——通过仿冒 GitHub 组织名来伪装项目。

下载的可执行文件是一个**信息窃取木马**，具备线程注入、远程 payload 下载和 Telegram 数据回传功能。

### 造成的损害（50+分钟应急响应）

木马在不到10分钟内执行了一系列破坏性攻击：

| 时间 | 动作 |
|------|------|
| 09:47 | 木马执行，释放持久化后门 |
| 09:47 | 创建 `AppData\Roaming\Roaming\Data\Config\manager.exe`（12.5MB）— 使用双重嵌套 `Roaming` 路径隐藏 |
| 09:47 | 添加注册表自启动：`HKCU\...\Run\{NetworkManager}` |
| 09:47 | 开放防火墙入站端口 **57001** 用于 C2 通信 |
| 09:47 | **关闭 Windows Defender** — 实时保护、行为监控、IOAV、NIS、访问保护全部关闭 |
| 09:47 | 释放额外组件：`svc_host.exe`、`~update.tmp.exe` |
| 10:37 | 访问浏览器数据目录以窃取凭证 |

### 归因

这不是一次随机攻击。它与一个**已知 APT 团伙**的攻击模式相关：

- 与 2026年3月的 OpenClaw 仿冒攻击是**同一团伙**
- GitHub 用户 `graphrtest` — 2025年10月休眠，2026年4月24日突然活跃
- 还仿冒过：GPT-5.5、Kimi、Manus AI、Seedance、fraudGPT
- 投放的恶意载荷包括：**Vidar** 信息窃取器、**GhostSocks** 代理木马、**MacSync** Stealer
- 被追踪方：**Microsoft**、**奇安信**、**Huntress**、**Zscaler**
- 发现台湾 IP `103.127.218.197` 通过窃取的 token 创建了 GitHub OAuth 授权

正版 DeepSeek-TUI 仓库的 Issue #1286 有用户在5月9日报告了假冒仓库——比我的事件早两天。攻击者删除了假冒仓库的 Issue #2（有人举报为钓鱼）。

### 教训

**这可能发生在任何人身上。** 假冒仓库看起来完全合法。没有安全检查，根本无法区分真假。Huiyu-SafeAi 的存在就是为了在下载之前拦截这类攻击。

---

## 拦截时：完全透明

当 huiyu-safe-ai 拦截一个包时，它不只是说"已拦截"。它会告诉你：

1. **这个包是什么** — 名称、生态系统（npm/pypi/cargo）
2. **为什么危险** — 具体威胁类型（typosquatting、加密货币挖矿、凭证窃取等）
3. **实际发生了什么** — 已记录的真实攻击
4. **信息来源** — 数据来源和归因

### 示例：拦截恶意包

```
huiyu-safe-ai: 拦截
包名: vite-plugin-bomb
原因: 已确认恶意 — 伪装成 Vite 插件的破坏性载荷
威胁: 递归删除项目文件并触发系统关机
来源: Socket Security 安全研究报告 (2025)，npm 注册表已确认移除
操作: 不要安装。该包已被 npm 移除，因其破坏性行为。
```

### 示例：仿冒检测

```
huiyu-safe-ai: 拦截
包名: deepseek-py
原因: Typosquatting — 冒充合法的 DeepSeek 包
威胁: 恶意载荷，窃取凭证和数据
来源: npm 注册表安全公告，社区报告，已确认 typosquatting
操作: 不要安装。请使用官方 DeepSeek API: https://api.deepseek.com
```

---

## 数据来源与归因

黑名单基于已验证的安全研究和官方公告构建：

| 来源 | 类型 | 用途 |
|------|------|------|
| **Socket Security** (socket.dev) | 安全研究 | npm 包威胁分析、供应链攻击报告 |
| **Datadog Security Labs** | 安全研究 | VS Code 扩展恶意软件、加密货币钱包盗窃 |
| **Fortra Security** | 安全研究 | 钓鱼 + npm 组合攻击 |
| **npm 官方安全公告** | 官方 | 包移除通知、安全公告 |
| **PyPI 安全** | 官方 | 恶意包移除确认 |
| **GitHub 安全** | 官方 | 仓库级安全警报 |
| **Aikido Security** | 安全研究 | npm 蠕虫传播分析 |
| **Phylum Research** | 安全研究 | Go 二进制隐写、PyPI 攻击 |

黑名单中的每个条目都至少通过上述一个来源确认。我们不会基于未经验证的报告或社交媒体帖子添加包。

---

## 支持的平台

| 平台 | 安装路径 |
|------|---------|
| Claude Code | `~/.claude/skills/huiyu-safe-ai/` |
| OpenAI Codex CLI | `~/.codex/skills/huiyu-safe-ai/` |
| Trae | `.trae/skills/huiyu-safe-ai/` |
| 任何支持 SKILL.md 的工具 | 复制文件夹即可 |

## 快速安装

### Claude Code

```bash
git clone https://github.com/huiyu9144/huiyu-safe-ai.git ~/.claude/skills/huiyu-safe-ai
```

### OpenAI Codex CLI

```bash
git clone https://github.com/huiyu9144/huiyu-safe-ai.git ~/.codex/skills/huiyu-safe-ai
```

### 手动安装

```bash
git clone https://github.com/huiyu9144/huiyu-safe-ai.git
# 将 huiyu-safe-ai 文件夹复制到你的 skills 目录
```

## 功能特性

- **68+ 个已确认恶意包**（全部来自安全研究来源）
- **60+ 个受信组织**自动放行（deepseek-ai、openai、anthropic、facebook 等）
- **70+ 个受信包**直接跳过所有检查（express、react、pandas、lodash 等）
- **仿冒检测** — 捕获名称相似的恶意包
- **代码嗅探** — 检测 postinstall 漏洞、凭证窃取、混淆载荷
- **透明拦截** — 解释每个被拦截包的原因，附带来源归因
- **零开销** — 无网络调用、无命令执行、纯提示词检查

## 威胁分类

黑名单覆盖以下已确认的攻击类型：

| 类别 | 数量 | 示例 |
|------|------|------|
| **仿冒 (Typosquatting)** | 20+ | deepseek-py、crossenv、babelcli、mongose、reques7s |
| **供应链攻击** | 10+ | event-stream、flatmap-stream、async-promises-extra |
| **破坏性载荷** | 8+ | js-bomb、vue-plugin-bomb、vite-plugin-bomb |
| **凭证窃取** | 10+ | ethereum-wallet-keygen、solaibot、among-eth |
| **数据外泄** | 8+ | webpack-plugin-spy、sqlite-wasm-spy、citiycar8 |
| **系统指纹收集** | 3+ | bbb335656、cdsfdfafd1232436437、sdsds656565 |
| **冒充** | 5+ | @anthropic-ai/sdk-spyware、openai-api-spyware |
| **破坏性抗议** | 3+ | node-ipc、peacenotwar、colors、faker |
| **反向 Shell** | 4+ | webhook、node-extensions-utils、colorama-backup |

## 输出示例

### 安全包（官方/受信）

```
huiyu-safe-ai: 安全
包名: express
来源:  官方 npm 包，30M+ 周下载量
判定: 未发现问题，可以安全使用。
```

### 拦截包（已知恶意）

```
huiyu-safe-ai: 拦截
包名: js-bomb
原因: 已确认恶意 — 伪装成工具库的破坏性载荷
威胁: 递归删除 Vue.js/React/Vite 项目文件，触发系统关机
来源: Socket Security 安全研究 (2025)，已从 npm 移除
操作: 不要安装。该包旨在破坏项目数据。
```

### 未知包（警告）

```
huiyu-safe-ai: 警告
包名: some-random-lib
来源:  个人仓库，下载量低
风险:  未发现已知威胁，但来源未经验证
建议: 手动审查后再决定是否安装。
       查看源代码: https://github.com/user/some-random-lib
```

## 更新黑名单

黑名单嵌入在 `SKILL.md` 中。添加新的恶意包：

1. 编辑 `SKILL.md` 中的"已知恶意包"表格
2. 包含：包名、生态系统、威胁描述
3. 标注数据来源（安全研究、npm 公告等）
4. 提交并推送

## 贡献

欢迎贡献！以下方面需要帮助：

- 扩展黑名单，添加新发现的恶意包（附带来源归因）
- 添加更多受信组织
- 改进仿冒检测规则
- 添加对更多生态系统的支持（Ruby gems、Go modules 等）

## 许可证

MIT

## 致谢

作为提示词级别的安全护栏构建。补充但不替代 `npm audit`、`snyk` 或 `socket.dev` 等工具。
