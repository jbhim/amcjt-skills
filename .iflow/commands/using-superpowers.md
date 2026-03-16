---
name: using-superpowers
description: 在开始任何对话时使用 - 建立如何查找和使用技能的规则，要求在 ANY 响应之前（包括澄清问题）必须先调用 Skill 工具
---

<EXTREMELY-IMPORTANT>
如果你认为技能有哪怕 1% 的可能性适用于你正在做的事情，你绝对必须调用该技能。

**如果技能适用于你的任务，你别无选择。你必须使用它。**

这不是可商量的。这不是可选的。你不能找理由逃避这一点。
</EXTREMELY-IMPORTANT>

## 如何访问技能

**在 Claude Code 中：** 使用 `Skill` 工具。当你调用技能时，它的内容会被加载并呈现给你——直接遵循它。永远不要对技能文件使用 Read 工具。

**在其他环境中：** 查看你的平台文档以了解如何加载技能。

# 使用技能

## 规则

**在提供任何响应或采取任何行动之前，先调用相关的或请求的技能。** 即使技能只有 1% 的可能性适用，你也应该调用该技能来检查。如果调用的技能最终不适用于当前情况，你不需要使用它。

```dot
digraph skill_flow {
    "User message received" [shape=doublecircle];
    "About to EnterPlanMode?" [shape=doublecircle];
    "Already brainstormed?" [shape=diamond];
    "Invoke brainstorming skill" [shape=box];
    "Might any skill apply?" [shape=diamond];
    "Invoke Skill tool" [shape=box];
    "Announce: 'Using [skill] to [purpose]'" [shape=box];
    "Has checklist?" [shape=diamond];
    "Create TodoWrite todo per item" [shape=box];
    "Follow skill exactly" [shape=box];
    "Respond (including clarifications)" [shape=doublecircle];

    "About to EnterPlanMode?" -> "Already brainstormed?";
    "Already brainstormed?" -> "Invoke brainstorming skill" [label="no"];
    "Already brainstormed?" -> "Might any skill apply?" [label="yes"];
    "Invoke brainstorming skill" -> "Might any skill apply?";

    "User message received" -> "Might any skill apply?";
    "Might any skill apply?" -> "Invoke Skill tool" [label="yes, even 1%"];
    "Might any skill apply?" -> "Respond (including clarifications)" [label="definitely not"];
    "Invoke Skill tool" -> "Announce: 'Using [skill] to [purpose]'";
    "Announce: 'Using [skill] to [purpose]'" -> "Has checklist?";
    "Has checklist?" -> "Create TodoWrite todo per item" [label="yes"];
    "Has checklist?" -> "Follow skill exactly" [label="no"];
    "Create TodoWrite todo per item" -> "Follow skill exactly";
}
```
## 危险信号

这些想法意味着停止——你在找借口：

| 想法 | 现实 |
|---------|---------|
| "这只是个简单的问题" | 问题也是任务。检查技能。 |
| "我需要先获取更多上下文" | 技能检查在澄清问题之前进行。 |
| "让我先探索一下代码库" | 技能告诉你如何探索。先检查。 |
| "我可以快速检查 git/文件" | 文件缺乏对话上下文。检查技能。 |
| "让我先收集信息" | 技能告诉你如何收集信息。 |
| "这不需要正式的技能" | 如果技能存在，就使用它。 |
| "我记得这个技能" | 技能会演进。阅读当前版本。 |
| "这不算任务" | 行动 = 任务。检查技能。 |
| "这个技能有点大材小用" | 简单的事情会变复杂。使用它。 |
| "我就先做这一件事" | 在做任何事情之前先检查。 |
| "这感觉很有成效" | 无纪律的行动浪费时间。技能防止这种情况。 |
| "我知道那是什么意思" | 知道概念 ≠ 使用技能。调用它。 |

## 技能优先级

当多个技能可能适用时，使用此顺序：

1. **首先使用流程技能**（头脑风暴、调试）- 这些决定如何处理方法
2. **其次使用实现技能**（前端设计、mcp 构建器）- 这些指导执行

"让我们构建 X" → 先头脑风暴，然后实现技能。
"修复这个 bug" → 先调试，然后领域特定技能。

## 技能类型

**严格型**（TDD、调试）：准确遵循。不要适应性地放弃纪律。

**灵活型**（模式）：根据情境调整原则。

技能本身会告诉你属于哪种类型。

## 用户指令

指令说的是**做什么**，而不是**怎么做**。"添加 X"或"修复 Y"并不意味着跳过工作流程。
