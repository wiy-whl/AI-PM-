# Agent Skill 深度架构与构建指南计划

此计划旨在为用户详细讲解 Agent Skill 的构成及构建高质量 Skill 的方法，重点结合系统中现有的 Skill 架构（如 `alpha-workflow-engineer`）进行实战分析。

## 目标
1. 拆解 Agent Skill 的物理与逻辑构成。
2. 阐述“渐进式披露” (Progressive Disclosure) 的设计哲学。
3. 提供构建高质量 Skill 的 PM 视角 Checklist。

## 提议的变更

### [Component] Interview Prep Artifacts
#### [MODIFY] [agent_skill_guide.md](file:///C:/Users/联想/.gemini/antigravity/brain/a575c5c7-0028-48d8-8f8b-acbba996eae8/agent_skill_guide.md)
*   新增：LLM-as-a-Judge 深度解析与进阶评价模式 (SBS, Bias Mitigation, HIL)。
*   新增：工程组件落地 SOP 与 自动化 Pipeline 逻辑。
*   解析 `name` 与 `description` 的触发机制。
*   分享如何通过 `references` 保持上下文效率。
*   结合 `alpha-workflow-engineer` 案例进行二次解析。

## 验证计划

### 自动化验证 (Self-Check)
*   检查生成的 Markdown 格式是否正确。
*   验证文件链接是否指向正确的绝对路径。

### 手动验证 (User Review)
*   请求用户审阅 [agent_skill_guide.md](file:///C:/Users/联想/.gemini/antigravity/brain/a575c5c7-0028-48d8-8f8b-acbba996eae8/agent_skill_guide.md)。
*   询问用户是否需要针对特定的 Agent 框架（如 LangChain, AutoGPT, 或企业内研框架）进行适配性调整。
