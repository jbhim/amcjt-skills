---
name: amcjt-lottery-pro
description: 专业彩票助手 - 支持双色球开奖查询、彩票OCR识别、中奖核对、开奖提醒。触发词：彩票、双色球、开奖、中奖、lottery。
license: MIT
allowed-tools:
  - Bash(npx:*)
  - Bash(node:*)
  - Read
  - Write
  - Exec
metadata:
  openclaw:
    icon: "🎱"
    category: "lifestyle"
    author: "amcjt"
    bins: ["npx", "node"]
    env: ["MOONSHOT_API_KEY"]
    os: ["darwin", "linux", "win32"]
    user-invocable: true
---

# 专业彩票助手

集成 amcjt-mcp-server 提供的彩票服务，支持双色球相关操作。

## 环境要求

- **依赖**: npx, node
- **环境变量**: MOONSHOT_API_KEY（用于OCR识别）
- **支持系统**: macOS, Linux, Windows

## 触发条件

当用户提到以下关键词时触发：
- "彩票"、"双色球"、"开奖"、"中奖"、"lottery"
- 用户上传图片文件（彩票照片）
- "我中了没"、"查一下开奖"、"看看中奖号码"

## 核心功能

### 1. 查询开奖结果

用户询问某期开奖号码时：

1. 提取期号（格式：YYYY + 3位序号，如 2026020）
2. 如未提供期号，查询最新一期
3. 调用 `get_lottery_result` 获取结果
4. 展示：开奖日期、红球号码（6个）、蓝球号码（1个）、奖池金额、中奖注数

### 2. 彩票OCR识别

用户上传彩票图片时：

1. 读取图片文件
2. 如图片 > 200KB，先用脚本压缩
3. 转为 base64 格式
4. 调用 `ocr_lottery_ticket` 识别号码
5. 展示识别的红球和蓝球号码

### 3. 中奖核对

用户提供投注号码时：

1. 获取期号（用户输入或查询最新）
2. 获取投注号码（6个红球 01-33，1个蓝球 01-16）
3. 调用 `check_lottery_win` 核对
4. 展示中奖等级和奖金

## 工作流程

### 流程1：查询指定期号开奖

```
用户：查一下2026020期双色球开奖
↓
提取期号：2026020
↓
执行：npx amcjt-mcp-server get_lottery_result {"issueNo": "2026020"}
↓
解析返回，格式化展示
```

### 流程2：识别彩票图片

```
用户上传 lottery.jpg
↓
读取图片 → 检查大小 → 必要时压缩
↓
转为 base64
↓
执行：npx amcjt-mcp-server ocr_lottery_ticket {"imageBase64": "...", "lottoType": "101"}
↓
展示识别结果，询问是否核对中奖
```

### 流程3：核对中奖

```
用户：我买的 01 13 14 21 24 30 + 02，中了没？
↓ 
提取红球：[01,13,14,21,24,30]，蓝球：[02]
↓
获取期号（最新或指定）
↓
执行：npx amcjt-mcp-server check_lottery_win {"issueNo": "...", "bets": [...]}
↓
展示中奖等级和奖金
```

## 数据格式

### 投注号码格式
```json
{
  "redNumbers": ["01","13","14","21","24","30"],
  "blueNumbers": ["02"]
}
```

### 红球范围：01-33（选6个）
### 蓝球范围：01-16（选1个）

## 脚本

### scripts/compress_image.js
用于压缩过大的彩票图片（>200KB）

### scripts/lottery.js
彩票工具脚本，封装 MCP 调用：
- `getResult(issueNo)` - 查询开奖结果
- `ocrTicket(imagePath)` - OCR识别彩票
- `checkWin(issueNo, bets)` - 核对中奖

## 注意事项

1. OCR 需要 MOONSHOT_API_KEY 环境变量
2. 图片建议 < 2MB，太大时先压缩
3. 双色球 lottoType = "101"
4. 期号格式：年份 + 3位序号（如2026020）
