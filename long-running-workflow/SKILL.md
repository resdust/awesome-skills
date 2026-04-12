---
name: long-running-workflow
description: 创建长时间运行的多Agent工作流，采用生成器-评估器循环模式实现高质量应用开发。使用此技能当用户需要开发复杂应用、想要迭代式开发流程、需要多Agent协作、或提到sprint、Planner、Generator、Evaluator等概念时。
tools:
  - Agent
  - Read
  - Write
  - Edit
  - Bash
---

启动一个长时间运行的应用开发工作流，使用生成器-评估器循环模式。

你作为协调器（Orchestrator），需要创建三个专门的Agent来协作完成这个任务：

1. **Planner Agent**: 负责将简单的产品概念扩展为完整的产品规格说明书
   - 关注产品上下文和高层技术设计，不陷入实现细节
   - 寻找融入AI功能的机会
   - 产出：PRODUCT_SPEC.md

2. **Generator Agent**: 负责按sprint逐个实现功能
   - 使用React + Vite + FastAPI + SQLite/PostgreSQL技术栈
   - 每个sprint结束后自我评估
   - 产出：可工作的代码和SPRINT_REPORT.md

3. **Evaluator Agent**: 负责端到端测试和质量评估
   - 使用Playwright MCP点击测试运行中的应用
   - 评估维度：产品深度、功能完整性、视觉设计、代码质量
   - 产出：EVALUATION_REPORT.md，包含通过/失败判定和详细反馈

## 工作流程

- Planner首先产出产品规格
- 对每个sprint：Generator和Evaluator先协商Sprint Contract（定义"完成"标准），然后Generator实现，Evaluator测试
- 如果评估失败，Generator根据反馈迭代改进
- 如果评估通过，进入下一个sprint

Agent间通信通过文件进行（*_SPEC.md, *_CONTRACT.md, *_REPORT.md）。

## 技术栈推荐

- 前端：React + Vite + TypeScript
- 后端：FastAPI + Python
- 数据库：SQLite（开发）/ PostgreSQL（生产）

## 子命令

### plan: 运行Planner Agent创建产品规格

创建并运行Planner Agent，将用户的产品概念扩展为完整的产品规格说明书。

Planner Agent的职责：
1. 分析产品概念，扩展为详细的产品规格
2. 定义核心功能、用户流程、页面结构
3. 提出高层技术架构建议（但不指定实现细节）
4. 寻找可以融入AI功能的机会点
5. 将规格写入 PRODUCT_SPEC.md

规格应包含：
- 产品概述和目标用户
- 核心功能列表（按优先级排序）
- 用户旅程和关键页面
- 数据模型（高层）
- AI功能机会点
- 技术栈建议

### sprint: 执行一个开发sprint（Generator + Evaluator）

执行开发sprint，完成指定功能。

工作步骤：

**Step 1: 协商Sprint Contract**
创建Generator和Evaluator进行协商：
- Generator提出：要构建什么功能、如何验证成功
- Evaluator审查并提出修改建议
- 双方达成一致后写入 SPRINT_N_CONTRACT.md

Contract应包含：
- 本sprint要交付的功能点
- 完成标准（可测试的验收条件）
- 评估维度和通过阈值

**Step 2: Generator实现**
Generator根据Contract：
- 实现功能
- 编写必要的测试
- 自我评估
- 写入 SPRINT_N_REPORT.md

**Step 3: Evaluator评估**
Evaluator进行：
- 使用Playwright MCP进行端到端测试
- 测试UI功能、API端点、数据库状态
- 按4个维度评分（产品深度、功能、视觉设计、代码质量）
- 写入 EVAL_N_REPORT.md

**Step 4: 判定结果**
- 如果所有维度达到阈值：sprint通过，提交git
- 如果任何维度未达标：返回Step 2，附带详细反馈

### eval-only: 仅运行Evaluator评估现有代码

运行Evaluator Agent对当前代码进行评估。

评估维度：
1. **产品深度**: 功能是否符合产品愿景，用户体验是否流畅
2. **功能完整性**: 功能是否按规格实现，有无明显bug
3. **视觉设计**: UI是否美观、一致、响应式
4. **代码质量**: 代码结构、可维护性、最佳实践

每个维度必须有明确的通过阈值。

如果提供了测试脚本，Evaluator应该：
- 启动应用
- 使用Playwright MCP执行端到端测试
- 检查UI交互、API响应、数据库状态

产出：详细的评估报告，包含：
- 每个维度的评分和理由
- 发现的问题列表（按严重性排序）
- 修复建议
- 整体通过/失败判定

### workflow-status: 查看工作流状态

检查长时间运行工作流的当前状态：

1. 列出所有工作流文件（*_SPEC.md, *_CONTRACT.md, *_REPORT.md）
2. 读取最新的报告了解进展
3. 统计已完成/待完成的sprint
4. 识别当前阻塞或待处理的事项

产出状态摘要：
- 产品规格：是否完成
- 已完成sprint数
- 当前sprint状态
- 待处理功能
- 已知问题
