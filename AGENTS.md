# AMCJT Skills 项目指南

## 项目概述

本项目是一个 **Skill 集合**，用于扩展 AI 的功能。当前包含一个专业彩票助手技能（`amcjt-lottery`），支持双色球开奖查询、彩票 OCR 识别、中奖核对、开奖倒计时等功能。

### 项目结构

```
amcjt-skills/
├── .gitignore              # Git 忽略配置
├── .iflowignore            # iFlow CLI 忽略配置
├── AGENTS.md               # 本文件 - 项目指南
├── package.json            # Node.js 依赖配置
├── package-lock.json       # 依赖锁定文件
├── node_modules/           # 依赖目录（已忽略）
├── .idea/                  # IntelliJ IDEA 配置（已忽略）
├── lottery.jpg             # 示例彩票图片（可选）
├── lottery-small.jpg       # 压缩后的示例图片（可选）
└── amcjt-lottery/          # 彩票助手 Skill
    └── SKILL.md            # Skill 定义和说明文档
```

---

## amcjt-lottery Skill

### 功能介绍

专业彩票助手，通过 **OpenClaw 内置的 mcporter 技能** 调用 MCP 工具，集成 `amcjt-mcp-server` 提供的彩票服务，支持以下功能：

1. **查询开奖结果** - 查询指定或最新期号的双色球开奖号码
2. **彩票 OCR 识别** - 识别彩票图片中的红球和蓝球号码
3. **中奖核对** - 核对投注号码是否中奖及中奖等级
4. **开奖倒计时** - 查询距离下次开奖的时间
5. **开奖日历** - 获取未来 7 天的开奖日历

### 触发关键词

- 彩票、双色球、开奖、中奖、lottery
- 用户上传图片文件（彩票照片）
- "我中了没"、"查一下开奖"、"看看中奖号码"
- "下次什么时候开奖"、"开奖倒计时"

### 环境要求

- **OpenClaw mcporter 技能**: 用于调用 MCP 工具（请确保已启用并配置 amcjt-mcp-server）
- **支持系统**: macOS, Linux, Windows

> ⚠️ **重要前提**：本技能依赖 **OpenClaw 内置的 mcporter 技能** 来调用 MCP 工具。使用前请确保：
> 1. 已启用 mcporter 技能
> 2. mcporter 已配置并注册 **amcjt-mcp-server**

### mcporter 配置

本 Skill 依赖 [mcporter](https://www.npmjs.com/package/mcporter) 调用 MCP 工具。

**安装 mcporter：**
```bash
npm install -g mcporter
# 或使用 npx（无需安装）
mcporter --help
```

**配置示例** (`config/mcporter.json`)：
```json
{
  "mcpServers": {
    "amcjt-mcp-server": {
      "description": "AMCJT 彩票 MCP 服务器",
      "command": "npx",
      "args": ["-y", "amcjt-mcp-server@latest"],
      "env": {
        "MOONSHOT_API_KEY": "${MOONSHOT_API_KEY}"
      }
    }
  }
}
```

**验证配置：**
```bash
# 列出已配置的 MCP 服务器
mcporter list

# 查看彩票服务器的工具列表
mcporter list amcjt-mcp-server

# 查看详细配置
mcporter list --verbose
```

### MCP 工具调用

本 Skill 通过 mcporter 调用以下 MCP 工具：

| 工具名 | 功能 | 参数 | mcporter 调用示例 |
|--------|------|------|-------------------|
| `get_lottery_result` | 查询开奖结果 | `issueNo`, `lottoType` | `mcporter call 'amcjt-mcp-server.get_lottery_result(issueNo: "2026020", lottoType: "101")'` |
| `ocr_lottery_ticket` | OCR 识别彩票 | `image`, `lottoType` | `mcporter call 'amcjt-mcp-server.ocr_lottery_ticket(image: "/path/to/lottery.jpg", lottoType: "101")'` |
| `check_lottery_win` | 核对中奖 | `issueNo`, `bets`, `lottoType` | `mcporter call 'amcjt-mcp-server.check_lottery_win(issueNo: "2026020", lottoType: "101", bets: [...])'` |
| `get_lottery_countdown` | 开奖倒计时 | 无 | `mcporter call amcjt-mcp-server.get_lottery_countdown` |
| `get_lottery_calendar` | 开奖日历 | 无 | `mcporter call amcjt-mcp-server.get_lottery_calendar` |

**mcporter 调用语法：**

```bash
# 1. 函数调用语法（推荐，避免参数类型问题）
mcporter call 'amcjt-mcp-server.get_lottery_result(issueNo: "2026020", lottoType: "101")'

# 2. 冒号分隔参数（shell 友好）
mcporter call amcjt-mcp-server.get_lottery_result issueNo:2026020 lottoType:101

# 3. 简写形式（自动推断 call 命令）
mcporter amcjt-mcp-server.get_lottery_result lottoType:101

# 4. 输出 JSON 格式
mcporter call amcjt-mcp-server.get_lottery_result lottoType:101 --output json
```

### 数据格式

#### 投注号码格式

```json
{
  "redNumbers": ["01", "13", "14", "21", "24", "30"],
  "blueNumbers": ["02"]
}
```

- **红球范围**: 01-33（选 6 个）
- **蓝球范围**: 01-16（选 1 个）
- **期号格式**: 年份 + 3 位序号（如 2026020）
- **双色球 lottoType**: "101"（字符串类型）

### mcporter 故障排除

#### 1. 检查 mcporter 安装

```bash
# 验证 mcporter 是否安装
mcporter --version

# 查看帮助
mcporter --help
```

#### 2. 检查 MCP 服务器配置

```bash
# 列出所有已配置的 MCP 服务器
mcporter list

# 查看特定服务器的详细信息
mcporter list amcjt-mcp-server --schema

# 查看配置来源
mcporter list --verbose
```

#### 3. 测试工具调用

```bash
# 测试连接
mcporter call amcjt-mcp-server.get_lottery_countdown --output json

# 查看详细日志
mcporter call amcjt-mcp-server.get_lottery_countdown --log-level debug
```

#### 4. 配置问题排查

```bash
# 查看当前使用的配置文件路径
mcporter config list

# 添加/修改配置
mcporter config add test-mcp https://your-mcp-server-url.com/mcp

# 导入其他编辑器的配置（Cursor、Claude、VS Code 等）
mcporter config import cursor --copy
```

#### 5. 环境变量未生效

如果使用 `${ENV_VAR}` 语法但变量未生效：

```bash
# 检查环境变量是否设置
echo $MOONSHOT_API_KEY  # Linux/Mac
# 或
echo %MOONSHOT_API_KEY%  # Windows

# 直接在命令行传递（覆盖配置文件）
MOONSHOT_API_KEY=your-key mcporter call amcjt-mcp-server.ocr_lottery_ticket ...
```

---

## 开发规范

### 文件组织

- Skill 目录使用 `SKILL.md` 定义元数据和功能说明
- 项目根目录使用 `package.json` 管理 Node.js 依赖
- 遵循 OpenClaw Skill 规范定义触发条件和允许的工具

### Skill 定义规范

`SKILL.md` 文件使用 YAML Front Matter 格式：

```yaml
---
name: amcjt-lottery                          # Skill 唯一标识
description: 专业彩票助手 - 支持...           # 功能描述和触发词
license: MIT
allowed-tools:                               # 允许调用的工具列表
  - get_lottery_result
  - ocr_lottery_ticket
  - check_lottery_win
  - get_lottery_countdown
  - get_lottery_calendar
  - Bash(node:*)
  - Bash(npx:*)
  - Bash(mcporter:*)
metadata:
  openclaw:
    icon: "🏆"                               # Skill 图标
    category: "lifestyle"                   # 分类
    author: "amcjt"                         # 作者
    bins: ["node", "mcporter"]              # 所需二进制工具
    os: ["darwin", "linux", "win32"]        # 支持的操作系统
    user-invocable: true                    # 用户是否可直接调用
---
```

### 代码风格

- Skill 定义使用 YAML Front Matter 格式
- 包含详细的文档说明和使用示例
- 提供清晰的故障排除指南
- 使用中文编写用户-facing 文档

### 忽略规则

项目使用两个忽略文件：

- **.gitignore**: Git 版本控制忽略规则
- **.iflowignore**: iFlow CLI 搜索和索引忽略规则

共同忽略的项：
- IDE 配置（.idea/, .vscode/）
- 依赖目录（node_modules/, vendor/）
- 构建输出（dist/, build/, target/, out/）
- 日志文件（*.log, logs/）
- 临时文件和操作系统文件

---

## 扩展指南

如需添加新的 Skill：

1. 创建新的 Skill 目录（如 `amcjt-xxx/`）
2. 在该目录下创建 `SKILL.md` 定义 Skill 元数据和功能
3. 在 `SKILL.md` 中指定：
   - `name`: Skill 唯一标识
   - `description`: 功能描述和触发词
   - `allowed-tools`: 允许调用的工具列表
   - `metadata`: OpenClaw 相关配置（icon、category、author 等）
4. 如需新增依赖，更新根目录 `package.json`
5. 更新本 `AGENTS.md` 文件添加新 Skill 的说明

---

## 相关资源

- **Skill 定义**: `amcjt-lottery/SKILL.md`
- **依赖配置**: `package.json`
- **mcporter npm 包**: https://www.npmjs.com/package/mcporter
- **Model Context Protocol 规范**: https://github.com/modelcontextprotocol/specification

---

## 更新日志

### 2026-03-04
- 更新 Skill 目录名：`amcjt-lottery-pro` → `amcjt-lottery`
- 移除 scripts 目录和相关说明（图片压缩功能已移除）
- 更新 AGENTS.md，优化 mcporter 使用说明
- 添加 Skill 定义规范说明（YAML Front Matter 格式）
- 添加 mcporter 故障排除指南

### 2026-03-02
- 移除 `lottery.js` 脚本，改为直接调用 MCP 工具
- 更新 `compress_image.js` 为完整实现（使用 sharp 库）
- 添加 `package.json` 管理依赖
- 新增支持 `get_lottery_countdown` 和 `get_lottery_calendar` 工具