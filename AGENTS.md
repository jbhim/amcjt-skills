# AMCJT Skills 项目指南

## 项目概述

本项目是一个 **Skill 集合**，用于扩展ai的功能。当前包含一个专业彩票助手技能（`amcjt-lottery-pro`），支持双色球开奖查询、彩票OCR识别、中奖核对、开奖倒计时等功能。

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
└── amcjt-lottery-pro/      # 彩票助手 Skill
    ├── SKILL.md            # Skill 定义和说明文档
    └── scripts/            # 辅助脚本
        └── compress_image.js   # 图片压缩脚本
```

---

## amcjt-lottery-pro Skill

### 功能介绍

专业彩票助手，集成 `amcjt-mcp-server` 提供的彩票服务，支持以下功能：

1. **查询开奖结果** - 查询指定或最新期号的双色球开奖号码
2. **彩票OCR识别** - 识别彩票图片中的红球和蓝球号码
3. **中奖核对** - 核对投注号码是否中奖及中奖等级
4. **开奖倒计时** - 查询距离下次开奖的时间
5. **开奖日历** - 获取未来7天的开奖日历

### 触发关键词

- 彩票、双色球、开奖、中奖、lottery
- 用户上传图片文件（彩票照片）
- "我中了没"、"查一下开奖"、"看看中奖号码"
- "下次什么时候开奖"、"开奖倒计时"

### 环境要求

- **依赖**: node（用于图片压缩脚本）
- **环境变量**: `MOONSHOT_API_KEY`（用于OCR识别）
- **支持系统**: macOS, Linux, Windows

### 依赖安装

```bash
# 安装 sharp 图片处理库（用于 compress_image.js）
npm install
```

### 脚本使用

#### compress_image.js - 图片压缩脚本

```bash
# 压缩彩票图片（目标 < 200KB 适合OCR上传）
# node amcjt-lottery-pro/scripts/compress_image.js <输入路径> [输出路径]

# 示例
node amcjt-lottery-pro/scripts/compress_image.js ./lottery.jpg ./lottery-small.jpg
```

**功能特点：**
- 自动检测文件大小，小于 200KB 直接复制
- 智能压缩策略：根据目标大小自动调整质量和尺寸
- 支持多格式：JPEG、PNG、WebP、GIF、TIFF、AVIF
- 超大图片自动缩小尺寸（最大 2048px）
- 支持二次压缩确保达到目标大小

### MCP 工具调用

Skill 直接调用以下 MCP 工具：

| 工具名 | 功能 | 参数 |
|--------|------|------|
| `get_lottery_result` | 查询开奖结果 | `issueNo`, `lottoType` |
| `ocr_lottery_ticket` | OCR识别彩票 | `imageBase64`, `lottoType` |
| `check_lottery_win` | 核对中奖 | `issueNo`, `bets`, `lottoType` |
| `get_lottery_countdown` | 开奖倒计时 | 无 |
| `get_lottery_calendar` | 开奖日历 | 无 |

### 数据格式

#### 投注号码格式

```json
{
  "redNumbers": ["01", "13", "14", "21", "24", "30"],
  "blueNumbers": ["02"]
}
```

- **红球范围**: 01-33（选6个）
- **蓝球范围**: 01-16（选1个）
- **期号格式**: 年份 + 3位序号（如 2026020）
- **双色球 lottoType**: "101"

---

## 开发规范

### 文件组织

- Skill 目录使用 `SKILL.md` 定义元数据和功能说明
- 辅助脚本放在 `scripts/` 子目录中
- 脚本需包含详细的使用说明注释
- 项目根目录使用 `package.json` 管理 Node.js 依赖

### 代码风格

- 使用 JavaScript (Node.js) 编写脚本
- 包含详细的 JSDoc/注释说明
- 提供清晰的错误处理和用户提示
- 使用异步/await 处理异步操作

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

1. 创建新的 Skill 目录（如 `amcjt-xxx-pro/`）
2. 在该目录下创建 `SKILL.md` 定义 Skill 元数据和功能
3. 如有需要，创建 `scripts/` 目录存放辅助脚本
4. 如需新增依赖，更新根目录 `package.json`
5. 更新本 `AGENTS.md` 文件添加新 Skill 的说明

---

## 相关资源

- **Skill 定义**: `amcjt-lottery-pro/SKILL.md`
- **图片压缩脚本**: `amcjt-lottery-pro/scripts/compress_image.js`
- **依赖配置**: `package.json`

---

## 更新日志

### 2026-03-02
- 移除 `lottery.js` 脚本，改为直接调用 MCP 工具
- 更新 `compress_image.js` 为完整实现（使用 sharp 库）
- 添加 `package.json` 管理依赖
- 新增支持 `get_lottery_countdown` 和 `get_lottery_calendar` 工具