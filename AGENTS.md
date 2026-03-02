# AMCJT Skills 项目指南

## 项目概述

本项目是一个 **iFlow CLI Skill 集合**，用于扩展 iFlow CLI 的功能。当前包含一个专业彩票助手技能（`amcjt-lottery-pro`），支持双色球开奖查询、彩票OCR识别和中奖核对功能。

### 项目结构

```
amcjt-skills/
├── .gitignore              # Git 忽略配置
├── .iflowignore            # iFlow CLI 忽略配置
├── AGENTS.md               # 本文件 - 项目指南
├── .idea/                  # IntelliJ IDEA 配置（已忽略）
└── amcjt-lottery-pro/      # 彩票助手 Skill
    ├── SKILL.md            # Skill 定义和说明文档
    └── scripts/            # 辅助脚本
        ├── compress_image.js   # 图片压缩脚本
        └── lottery.js          # 彩票工具脚本
```

---

## amcjt-lottery-pro Skill

### 功能介绍

专业彩票助手，集成 `amcjt-mcp-server` 提供的彩票服务，支持以下功能：

1. **查询开奖结果** - 查询指定或最新期号的双色球开奖号码
2. **彩票OCR识别** - 识别彩票图片中的红球和蓝球号码
3. **中奖核对** - 核对投注号码是否中奖及中奖等级

### 触发关键词

- 彩票、双色球、开奖、中奖、lottery
- 用户上传图片文件（彩票照片）
- "我中了没"、"查一下开奖"、"看看中奖号码"

### 环境要求

- **依赖**: npx, node
- **环境变量**: `MOONSHOT_API_KEY`（用于OCR识别）
- **支持系统**: macOS, Linux, Windows

### 脚本使用

#### lottery.js - 彩票工具脚本

```bash
# 查询开奖结果
node amcjt-lottery-pro/scripts/lottery.js get-result <期号>
# 示例：node lottery.js get-result 2026020

# OCR识别彩票图片
node amcjt-lottery-pro/scripts/lottery.js ocr <图片路径>
# 示例：node lottery.js ocr ./lottery.jpg

# 核对中奖
node amcjt-lottery-pro/scripts/lottery.js check-win <期号> '<投注JSON>'
# 示例：node lottery.js check-win 2026020 '{"bets":[{"redNumbers":["01","13","14","21","24","30"],"blueNumbers":["02"]}]}'
```

#### compress_image.js - 图片压缩脚本

```bash
# 压缩彩票图片（目标 < 200KB 适合OCR上传）
node amcjt-lottery-pro/scripts/compress_image.js <输入路径> [输出路径]
# 示例：node compress_image.js ./lottery.jpg ./lottery-small.jpg
```

> **注意**: 压缩脚本当前为简化版，建议安装 `sharp` 库以获得完整功能：`npm install sharp`

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

### 代码风格

- 使用 JavaScript (Node.js) 编写脚本
- 包含详细的 JSDoc/注释说明
- 提供清晰的错误处理和用户提示

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
4. 更新本 `AGENTS.md` 文件添加新 Skill 的说明

---

## 相关资源

- **Skill 定义**: `amcjt-lottery-pro/SKILL.md`
- **彩票脚本**: `amcjt-lottery-pro/scripts/lottery.js`
- **压缩脚本**: `amcjt-lottery-pro/scripts/compress_image.js`
