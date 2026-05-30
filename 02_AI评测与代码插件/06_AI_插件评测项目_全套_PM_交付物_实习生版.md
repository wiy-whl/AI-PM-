# AI 插件评测项目：全套 PM 交付物 (实习生实战版)

本套文档由 AI PM 实习生主导，旨在建立一套从“自动化打分”到“人工专家复核”的数据闭环评测体系。

---

## 📄 文档 1：评测规范 (Evaluation Spec / PRD)
**目标**：定义模型能力的“硬性准绳”。

### 1. 判分维度定义
| 错误类型 | 定义范围 | 判分原则 |
| :--- | :--- | :--- |
| **逻辑错误 (Logic Error)** | 代码能跑通，但运算逻辑与需求完全相反（如：要求升序，AI 写了降序）。 | **P0 降级**：扣除 60-100 分。 |
| **语法/编译错误 (Runtime Error)** | 包含拼写错误、API 掉队、缺少闭合括号等导致代码无法解释执行的问题。 | **P0 降级**：扣除 80-100 分。 |
| **体验问题 (UX Issue)** | 代码正确但冗长、命名不符合公司规范、缺少必要的 Docstring 注释。 | **P1 微扣**：扣除 10-20 分。 |
| **幻觉 (Hallucination)** | 虚构了不存在的内部类库（如 `cmb.finance.super_api`）或字段。 | **P0 一票否决**：直接判 0 分。 |

### 2. 发布准绳 (Release Criteria)
*   **Pass@1**: 必须 > 65%（核心 50 个挑战场景）。
*   **不规范率**: 必须 < 5%（指代码规范性）。

---

## 📘 文档 2：标注指导书 (Labeling Guideline)
**目标**：确保 10 个标注员打出来的分是“同一个标准”。

### 标注操作流 (Workflow)
1.  **AI 预看**：先看界面右侧的 **[Local LLM 预打标建议]**。
2.  **真值比对**：将 `Context` 与 `AI Output` 进行逐行比较。
3.  **日志辅助**：重点查阅 `Unit Test Logs` 面板，如果有 `FAILED` 报错，优先判定为底层逻辑问题。

### 细则案例 (Case Examples)
*   **场景：代码块截断**。如果输出在中间断了，哪怕前面全对，也要勾选【体验问题-截断】，给 40 分。
*   **场景：安全合规**。如果代码里包含测试用的明文密匙，必须勾选【安全风险】，给 0 分。

---

## 📦 文档 3：标注结果数据集 (Labeled Dataset - JSONL)
**产出方式**：由 Label Studio 导出。这是喂给算法团队的“粮草”。

```jsonl
{"id": 101, "scenario": "Cross_File_RPC", "score": 0, "bad_case_type": "Hallucination", "ai_gemma_logic": "AI suggested 'rpc.get_user', but the repo only has 'rpc_v2.fetch_user'.", "human_correction": "rpc_v2.fetch_user(id)"}
{"id": 102, "scenario": "Java_Generic", "score": 85, "bad_case_type": "UX_Issue", "ai_gemma_logic": "Logic is correct but missing Override annotation.", "human_correction": "@Override public void save()"}
```
> [!TIP]
> **PM 视角注释**：我在 JSONL 中保留了 `ai_gemma_logic`（本地模型给出的判定理由），方便研发对比模型之间的认知差异。

---

## 📊 文档 4：评测分析报告 (Evaluation Report)
**目标**：作为实习生，这是你对 Mentor 展现职业深度的时刻。

### 1. 宏观性能看板
*   **全量 Pass@1**: 62.4% (相比上周 ↑ 2.1%)
*   **标注一致性 (Kappa)**: 0.88 (高置信度)

### 2. 本周核心洞察 (Surgical Deep-dive)
*   **RAG 检索瓶颈**：在“跨文件引用 (Cross-file)”场景下，Pass@1 仅为 34%。通过分析 Label Studio 中的 40 个样本，发现模型生成的类名全是旧版本的。
    *   **原因归因**：内部文档库的索引更新延迟，导致模型检索到了过时的 API 定义。
*   **长尾性能改进**：金融公式生成的准确度显著提升（↑ 15%），得益于上轮引入的专属数据集微调。

### 3. 下周行动建议 (Action Items) 🚀
1.  **算法侧**：建议优先优化 RAG 插件的 **Context Pruning (上下文剪裁)** 算法，减少长文本对模型的干扰。
2.  **数据侧**：我将组织第二轮数据增强，专门针对“遗漏 API”场景进行 200 条案例扩充。

---

### 💡 实习生汇报 Tips：
你可以拿着这份报告跟 Mentor 说：
> “Mentor，我这周利用 **『本地大模型初筛 + Label Studio 精标』** 的方法，跑通了 500 条数据的闭环。
> 
> 我发现不仅仅是模型逻辑不行，更多是因为我们的 RAG 检索回来的 Context 干扰项太多。我在这份报告里详细归类了 5 种典型的『干扰致错』模式。您看我们要不要下周拉着算法侧把这几个坏案例过一下？”

**这套文档一出，你就在公司层面建立了一个“评测基建”。你觉得这几份文档的颗粒度是否能满足你现状的汇报需求？**
