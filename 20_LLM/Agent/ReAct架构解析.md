---
title: ReAct架构解析
source: https://chatgpt.com/c/69534156-6278-8323-bd32-b52f40e0a38e
created: 2025-12-30
description:
tags:
  - ReAct
  - Agent
---
Ai领域ReAct 架构是什么，主要解决什么问题，有哪些技术实现

---

在 AI 领域中， **ReAct 架构** 是近年来在大模型应用与 Agent 设计中被广泛采用的一种 **推理—行动（Reasoning + Acting）统一范式** 。它并非传统意义上的“系统架构”，而是一种 **提示工程与 Agent 执行模型相结合的设计思想** 。

下面从 **是什么、解决什么问题、如何实现** 三个层面系统说明。

---

## 一、ReAct 架构是什么

\*\*ReAct（Reason + Act）\*\*最早由 Google Research 在 2022 年提出，其核心思想是：

> **让大模型在“思考（Reasoning）”与“行动（Action）”之间显式交替，而不是一次性输出最终答案。**

在 ReAct 中，模型的输出被结构化为三类步骤：

1. **Thought（思考）**
	- 模型对当前状态进行分析、规划、假设
2. **Action（行动）**
	- 调用外部工具 / API / 环境（搜索、数据库、代码执行等）
3. **Observation（观察）**
	- 接收工具返回结果，作为新的上下文

这一循环会反复进行，直到得到最终结论。

**抽象流程：**

这正是今天多数 Agent 系统的基础执行模型。

---

## 二、ReAct 主要解决什么问题

ReAct 的提出，直指 **纯语言模型推理在真实任务中的三大瓶颈** 。

### 1\. 解决「只会想，不会做」的问题

早期 CoT（Chain of Thought）：

- 擅长“推理”
- 但 **无法与真实世界交互**

ReAct 引入 Action：

- 模型可以：
	- 查询搜索引擎
	- 调用内部系统
	- 执行代码
	- 访问数据库
- 从“解释问题”升级为“ **解决问题** ”

> 从静态问答 → 动态任务执行

---

### 2\. 解决「幻觉（Hallucination）」问题

传统 LLM：

- 在 **不确定时也会强行给出答案**
- 尤其在事实、数据、系统状态问题上风险极高

ReAct 的改进点：

- **允许模型承认不知道**
- 通过 Action 主动获取证据
- 推理过程被 Observation 约束

这在以下场景尤为重要：

- 金融 / 信贷决策
- 企业内部知识问答
- 技术系统排错

---

### 3\. 解决复杂多步骤任务的可控性问题

在复杂任务中（例如系统分析、代码修改、业务规则推导）：

- 一次性输出：
	- 不可解释
	- 不可审计
	- 不可中断

ReAct 带来的价值：

| 维度 | 改进 |
| --- | --- |
| 可解释性 | 显式 Thought |
| 可控性 | Action 可被限制 |
| 可审计性 | 全链路执行轨迹 |
| 可插桩 | 可加入规则、校验、人审 |

这也是 **ReAct 非常契合银行、风控、企业 IT 场景** 的原因。

---

## 三、ReAct 的核心技术构成

从工程角度看，ReAct 并不复杂，但对 **协议、状态管理和约束设计** 要求很高。

---

### 1\. 输出协议设计（Structured Output）

典型 ReAct Prompt 结构：

关键点：

- Thought / Action / Observation **必须可解析**
- Action 必须映射到 **确定性工具调用**

在生产中通常采用：

- JSON Schema
- Function Calling
- DSL（如 BMAD、OpenSpec）

---

### 2\. 工具抽象与调度层

ReAct 的 Action 本质是：

> **将自然语言 → 可执行能力的映射**

常见工具类型：

| 工具类型 | 示例 |
| --- | --- |
| 搜索 | Web / 内部知识库 |
| 计算 | Python / SQL |
| 系统 | 内部服务 API |
| 代码 | Repo 分析 / 单测执行 |
| 业务 | 风控规则、额度测算 |

技术实现：

- Tool Registry
- Capability Router
- 权限白名单（非常关键）

---

### 3\. 状态管理（Agent Loop）

ReAct 是一个 **有状态的循环执行模型** ：

工程实现要点：

- 上下文裁剪（Token 控制）
- 长期 / 短期记忆区分
- 回滚与中断能力

---

### 4\. 与规划（Planning）的关系

ReAct ≠ Planner，但二者经常结合：

- **Plan → ReAct 执行**
- 或 ReAct 内嵌轻量规划

典型组合模式：

| 模式 | 说明 |
| --- | --- |
| Plan-and-Execute | 先拆任务，再逐步 ReAct |
| ReAct-only | 边想边做 |
| Reflexion + ReAct | 执行后自我反思 |
| Tree-of-Thought + ReAct | 多分支探索 |

---

## 四、主流 ReAct 实现与框架

### 1\. LangChain ReAct Agent

- 最经典实现
- 基于 Prompt + Tool Calling
- 优点：成熟、生态全
- 缺点：Prompt 偏重，调优成本高

---

### 2\. OpenAI Function Calling / JSON Agent

- 工程稳定性高
- 强约束输出结构
- 更适合生产系统

---

### 3\. AutoGPT / BabyAGI（早期）

- ReAct 的激进版本
- 自主性强，但风险高
- 更偏实验性质

---

### 4\. BMAD-Method / OpenSpec（你熟悉的）

在 SDD / Spec 驱动开发中：

- Thought → 需求推理
- Action → 生成/校验 Spec
- Observation → 编译、测试、Diff

本质是 **ReAct 在软件工程领域的垂直实现** 。

---

## 五、一句话总结（给领导或技术评审用）

> **ReAct 架构的本质，是把大模型从“一次性回答问题的黑盒”，升级为“可推理、可执行、可审计的任务执行引擎”。**

它解决了三件关键事情：

1. 让模型 **能调用工具、真正做事**
2. 用执行结果 **约束推理，降低幻觉**
3. 为复杂业务场景提供 **可控、可审计的 Agent 执行范式**

 