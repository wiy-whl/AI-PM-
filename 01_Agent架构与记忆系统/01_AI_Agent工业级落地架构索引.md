# AI Agent 工业级落地架构：知识图谱与指南索引

为了让你接下来的“实战代码落地复习”有一条极其清晰的主线，我将我们这几天针对 Agent 架构的高维研讨，梳理成了这套**知识图谱**。

这套体系完全抛弃了“大模型就是聊天框”的初级认知，而是真正将 Agent 当作一个**分布式的微服务软件系统**来构建。以下是全流程梳理及对应的复习文件路标：

---

## 阶段一：驯服大模型幻觉 —— 外置约束与路由系统（Constraints & Routing）

*核心精神：放弃用自然语言规劝大模型，改用代码级和物理级的硬卡控。*

1.  **动态工具装载 (Dynamic Skill Loading) 与路由设计**
    *   **落地**：不全量给工具，用两段式加载。先让大模型（Router）选名字，再向其注入带有详细出入参说明（JSON Schema/Function Calling）的说明书。
    *   **复习文件**：
        *   `d:\PromptX-main\worldbrain\AI_PM_Interview_Pack\dynamic_skill_loading_code.md` 
        *   `d:\PromptX-main\worldbrain\AI_PM_Interview_Pack\build_good_router.md`
2.  **用代码基建取代 Prompt 约束 (Linter & CI)**
    *   **落地**：永远不给大模型看报错大全。用 Flake8 锁死嵌套层数，用自动化流水线驳回不合格的代码，用冰冷真实的报错 log 去触发大模型反思，这是收敛幻觉的最优解。

---

## 阶段二：构建长线认知 —— Agent 记忆分层与混合检索 (Memory Architecture)

*核心精神：记忆是一套漏斗式的基础设施，必须层层设防，并具备自转代谢能力。*

1.  **物理四层切分落地方案**
    *   **落地**：工作窗留给当下；Skill 变按需发现；日志 (Episodic) 全量写盘；结论事实库 (MEMORY.md) 精简提纯常驻。
    *   **复习文件**：`d:\PromptX-main\worldbrain\AI_PM_Interview_Pack\agent_memory_architecture_deepdive.md`
2.  **放弃唯向量论，拥抱混合检索 (Hybrid Retrieval)**
    *   **落地**：OpenClaw 底层机制。基于扁平 `.md` 目录树。BM25 关键词匹配保底（30%）+ Dense Vector 向量寻找语义关联（70%），再用倒数秩进行评分融合。便宜且绝对可查。
    *   **复习文件**：`d:\PromptX-main\worldbrain\AI_PM_Interview_Pack\openclaw_hybrid_retrieval_deepdive.md`
3.  **让系统越用越聪明的 Memory Ops 闭环**
    *   **落地**：夜间提纯压缩 (Consolidation)；核心错误日志强制生成 Reflection 补丁；最绝杀的 Timestamp Decay（时间衰减因子）解决新老认知冲撞。
    *   **复习文件**：`d:\PromptX-main\worldbrain\AI_PM_Interview_Pack\agent_memory_self_improvement.md`

---

## 阶段三：降本防崩 —— 把“文件系统”作为沟通总线 (Context Interfaces)

*核心精神：永远不要把沉重的数据直接排进大模型的 Context 里排队，用指针让它自己拉取。*

1.  **Dynamic Context Discovery (动态上下文发现)**
    *   **落地**：让 Agent 通过文件操作工具与世界通信。写下几万字的回测结果不要贴进对话，而是告诉 Agent 存在 `log.json` 里。
    *   **复习文件**：`d:\PromptX-main\worldbrain\AI_PM_Interview_Pack\file_system_as_context_interface.md`
2.  **压缩回退的 Pointer-only 哲学**
    *   **落地**：执行记忆总结压缩时，无论成功与否，绝不在历史流（DB）里执行 `delete`。总结成功只是向后拉动了一个指针标签；一旦遇到模型宕机报错，直接切入物理的 `Raw Archive`（原始归档）进程。永不丢失细节数据。
    *   **复习文件**：`d:\PromptX-main\worldbrain\AI_PM_Interview_Pack\agent_memory_consolidation_safeguards.md`

---

## 阶段四：跨越死亡之谷 —— 长期任务与双轨状态机 (Long-Running Autonomy)

*核心精神：让每一个干活的 Agent 都能随时被枪毙重启，秘密在于把进度外挂在墙上。*

1.  **外置 JSON 强效状态机**
    *   **落地**：用最严格的嵌套规则（只许一个 `in_progress`，强制要求阻断依赖 `dependencies`）。大模型想乱来，外围脚手架代码就劈头盖脸报错，逼它认清现实。
2.  **Initializer 与 Worker 的无状态切换 (Stateless Reboot)**
    *   **落地**：让一个专门建档的 Agent 起局。跑活的 Agent 死了不怕，重新 `nohup` 唤醒一个，只需看两眼 `feature-list.json` 和 `git log`，瞬间就能满血衔接前任工作。同时，用后台线程解决 I/O 阻塞，释放大模型精力。
    *   **复习文件**：
        *   `d:\PromptX-main\worldbrain\AI_PM_Interview_Pack\agent_autonomy_and_long_tasks.md`
        *   `d:\PromptX-main\worldbrain\AI_PM_Interview_Pack\agent_autonomy_implementation_details.md`

---

## 阶段五：团队化作业 —— 多 Agent 系统的 Orchestrator 范式

*核心精神：严拒“虚拟人圆桌会议”，我们需要的是带护栏的流水生产线。*

1.  **协议第一，并行其次**
    *   **落地**：废弃自然语言派单，主辅 Agent 全面采用定义严密的 RPC-like 的 `.jsonl` 追加队列通信。规定好 `request_id` 与状态机状态。
2.  **物理级别的沙盒隔离**
    *   **落地**：子 Agent 关闭自身记忆，局限在地牢 `.worktrees/agent-x/` 中工作。主 Agent 只回收其返回的一行 Summary。
3.  **阻断幻觉级联**
    *   **落地**：多 LLM 是从众偏见的培养皿。必须切断串联，引入外部地面真理（Pytest 验证环境）或设置法官 Agent 进行对抗验证。
    *   **复习文件**：`d:\PromptX-main\worldbrain\AI_PM_Interview_Pack\multi_agent_architecture_guide.md`

***

### 💡 下一步实战推介

当你翻开代码库准备做映射时，请带着这三个灵魂问题去找对应的 Python 实现：
1. 我的路由层，那份 JSON Schema 是怎么动态包给大模型的？
2. 我的数据库读取，存不存类似于 `.md` 或文件目录结构的影子？
3. 我这个项目如果死在跑到一半，重启命令是怎样读取哪些 `log` 满血复活的？
