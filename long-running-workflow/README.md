# 长时间运行工作流 (Long-Running Workflow)

基于 Anthropic 工程团队的生成器-评估器循环模式，实现高质量应用开发的多 Agent 工作流。

## 架构概览

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   Planner   │────→│  Generator  │↔────│  Evaluator  │
│  (规划器)    │     │   (生成器)   │     │   (评估器)   │
└─────────────┘     └──────┬──────┘     └─────────────┘
                           │
                    ┌──────┴──────┐
                    │  Sprint N   │
                    │  Contract   │
                    └─────────────┘
```

### 三个核心 Agent

1. **Planner** - 产品规划
   - 输入：简单的 1-4 句产品概念
   - 输出：详细的产品规格说明书 (PRODUCT_SPEC.md)
   - 重点：产品上下文、高层技术设计、AI 功能机会
   - 避免：过早指定实现细节

2. **Generator** - 代码生成
   - 输入：产品规格 + Sprint Contract
   - 输出：可工作的代码 + Sprint Report
   - 工作方式：逐个 sprint 实现，每个功能完整后再下一个
   - 自评估：每个 sprint 结束自我检查

3. **Evaluator** - 质量评估
   - 输入：实现代码 + Sprint Contract
   - 输出：评估报告 (EVAL_*.md)
   - 方法：使用 Playwright MCP 进行端到端测试
   - 维度：产品深度、功能完整性、视觉设计、代码质量

## 工作流程

### 阶段 1: 规划
```
用户概念 → Planner → PRODUCT_SPEC.md
```

### 阶段 2: Sprint 循环
```
对每个功能：
  1. 协商 Sprint Contract
     Generator ↔ Evaluator → SPRINT_N_CONTRACT.md

  2. Generator 实现
     Generator → 代码 + SPRINT_N_REPORT.md

  3. Evaluator 评估
     Evaluator → EVAL_N_REPORT.md

  4. 判定
     通过 → 提交git，进入下一sprint
     失败 → 返回步骤2，附带反馈
```

## 文件通信协议

Agent 间通过文件进行通信，确保状态持久化：

| 文件 | 创建者 | 内容 |
|------|--------|------|
| `PRODUCT_SPEC.md` | Planner | 产品规格、功能列表、技术栈 |
| `SPRINT_N_CONTRACT.md` | Generator/Evaluator | Sprint 完成标准 |
| `SPRINT_N_REPORT.md` | Generator | 实现总结、自评估 |
| `EVAL_N_REPORT.md` | Evaluator | 详细评分、问题列表 |

## 技术栈推荐

- **前端**: React + Vite + TypeScript
- **后端**: FastAPI + Python
- **数据库**: SQLite（开发）/ PostgreSQL（生产）
- **测试**: Playwright MCP

## 使用方法

### 1. 初始化工作流
```
/long-running-workflow
```

### 2. 创建产品规格
```
/long-running-workflow:plan
prompt: 创建一个 todo list 应用，支持任务分类、优先级和提醒
```

### 3. 执行 Sprint
```
/long-running-workflow:sprint
sprint_number: 1
feature: 用户认证系统
spec_file: PRODUCT_SPEC.md
```

### 4. 仅评估现有代码
```
/long-running-workflow:eval-only
```

### 5. 查看状态
```
/long-running-workflow:workflow-status
```

## Sprint Contract 模板

每个 sprint 开始前，Generator 和 Evaluator 应协商并记录：

```markdown
# Sprint N Contract: [功能名称]

## 交付内容
- [ ] 功能点 1
- [ ] 功能点 2

## 完成标准
- [ ] 验收条件 1
- [ ] 验收条件 2

## 评估维度与阈值
1. 产品深度: 7/10 (必须符合产品愿景)
2. 功能完整性: 8/10 (无明显 bug)
3. 视觉设计: 7/10 (美观一致)
4. 代码质量: 8/10 (结构清晰)

## 验证方法
- Playwright 测试脚本: [路径]
- 手动检查点: [列表]
```

## 评估报告模板

```markdown
# Sprint N 评估报告

## 评分
| 维度 | 得分 | 阈值 | 结果 |
|------|------|------|------|
| 产品深度 | 8/10 | 7/10 | ✅ 通过 |
| 功能完整性 | 9/10 | 8/10 | ✅ 通过 |
| 视觉设计 | 6/10 | 7/10 | ❌ 失败 |
| 代码质量 | 8/10 | 8/10 | ✅ 通过 |

## 发现的问题
1. [严重] 登录后跳转错误
2. [中] 移动端布局问题
3. [低] 控制台警告

## 修复建议
1. 修复登录回调 URL 配置
2. 添加响应式断点
3. 移除未使用的 import

## 结论
**状态**: 失败，需要迭代
**下次检查重点**: 视觉设计和响应式
```

## 最佳实践

1. **Planner 不要过度指定**：保持高层，让 Generator 决定实现路径
2. **Sprint Contract 是关键**：在写代码前明确"完成"的定义
3. **硬阈值优于软评分**：每个维度必须有明确的通过/失败线
4. **文件是真相源**：所有 Agent 间通信通过文件，便于恢复和审计
5. **迭代直到通过**：不要让有问题的代码进入下一 sprint

## 与简单工作流的区别

| 维度 | 简单工作流 | 长时间运行工作流 |
|------|-----------|----------------|
| 规划 | 用户直接提供 | Planner 扩展 |
| 评估 | 无或简单检查 | Evaluator 深度测试 |
| 反馈循环 | 单次 | 迭代直到通过 |
| 质量保障 | 依赖 Generator 自律 | 独立 Evaluator 验证 |
| 适用场景 | 简单功能、原型 | 复杂应用、生产代码 |
