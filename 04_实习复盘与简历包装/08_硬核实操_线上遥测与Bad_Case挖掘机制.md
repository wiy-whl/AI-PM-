# [硬核实操] 线上遥测与 Bad Case 挖掘机制 (Telemetry & Bad Case Mining)

在面试中，如果面试官追问：“你说的这些真实用户的 Bad Case 到底是怎么拿到的？是你一行代码一行代码捞出来的吗？”

这是考核你对 **产品数据流转路线** 是否清晰的关键时刻。作为 AI PM，你需要展示出清晰的分工边界和数据提炼逻辑：**“规则我定，埋点研发写，数据我洗。”**

---

## 1. “捞错”是谁的责任？怎么分工？

这是一个跨职能协作的流程：
*   **AI PM 负责【定义与提取】**：PM 决定“什么动作代表用户不满意”，并根据研发打点的底层数据源，通过 SQL 或者内部 BI 平台（如 Kibana / Metabase）导出数据表。
*   **前端/客户端研发负责【埋点施放】**：开发人员负责在 IDE 插件内部写代码，当用户敲击特定按键（比如 Esc 或 Backspace）时，给服务器发送一条 JSON 日志。
*   **后端/数据研发负责【清洗上报】**：承接海量并发日志，去除重复的、无关紧要的心跳包数据，把它们存入数仓（如 Hive / ClickHouse）。

---

## 2. 怎么“找出”潜藏的 Bad Case？（埋点机制解密）

AI 代码插件的埋点，远比网页点击按钮复杂。你需要向面试官展示这几种极具代表性的**隐性埋点反馈 (Implicit Feedback)**：

### A. 放弃采纳 (Suggestion Ignored / Rejected)
*   **埋点逻辑**：在触发代码补全（灰色代码出现）时，用户没有按 `Tab` 键接受，而是继续自己打字，或者按了 `Esc` 键，灰色代码消失。
*   **PM 视角**：这是最常见的“负样本”。说明我们的生成**完全没帮上忙**。

### B. 采纳后大改 (Accepted Then Heavily Modified) —— [最高价值]
*   **埋点逻辑**：用户按了 `Tab` 接受了代码，但在接下来的 **30 秒** 或 **最近的 50 次按键** 内，修改了这段代码 30% 以上的字符（通过简单的 Levenshtein 距离测算）。
*   **PM 视角**：用户觉得“底子还行”，但“细节全错”。这是做 **SFT（监督微调）最顶级的黄金数据**，因为用户亲手把正确答案（Ground Truth）改出来了！

### C. 显性报错与差评 (Explicit Feedback)
*   **埋点逻辑**：用户点击了插件侧边栏的 👎 (Thumbs Down) 图标，或者输入框里的报错抛出了 `Exception/Compilation Error` 日志。
*   **PM 视角**：通常伴有用户的强烈负面情绪，必须优先作为 P0 级 Bad Case 分类。

---

## 3. 如何整理成能用的 Bad Case？(数据转换流)

当你从大数据的数仓里用 SQL 导出结果时，它只是一个带有毫秒级时间戳的巨大宽表。作为 PM，你的任务是把它 **“包装成 Label Studio 看得懂的格式”**。

**原始字段 (Raw Data) 往往是这样的：**
*   `event_time`: 1712492193
*   `user_id`: 9527
*   `event_type`: "TAB_ACCEPTED_THEN_DELETED"
*   `active_file_content`: "class User {..."
*   `ai_suggestion`: "void fetchUser() { rpc.call(); }"
*   `final_user_code`: "void fetchUserInfo() { rpc.v2_call(); }"

**PM 加工步骤 (Data Pipeline)：**
1.  **抽样与去重**：几万条 Reject 日志看不完，你编写 Python 脚本，采用分层抽样（比如每个错误大类抽 20 条）。
2.  **构建 Context 窗口**：切去过长的 `active_file_content`，只保留触发补全前后的 100 行代码作为 `$context`。
3.  **封转为 JSONL**：将清洗后的数据转存为供 Label Studio 读取的格式。

```json
{
  "context": "class User {...",
  "generated_code": "void fetchUser() { rpc.call(); }",
  "user_actual_code": "void fetchUserInfo() { rpc.v2_call(); }",
  "telemetry_event": "Heavily_Modified",
  "score": null,
  "bad_case_type": null 
}
// 后有两个 Null 字段，这就是留给 Label Studio 标注员或者高级专家去勾选的！
```

---

### 💡 职场话术总结（面试高光时刻）：

> “在 1-N 的持续迭代中，我并没有让专家去纯人肉挑错。
> 
> 我主导梳理了 **『代码补全埋点规范』**，与 IDE 前端团队配合，定义了涵盖 **『忽略 (Ignore)』**和 **『采纳后立刻修改 (Modified post-accept)』** 等五大类隐性反馈埋点。
> 
> 我定期通过数仓 SQL 捞取这些高修改率的底层 Log，利用 Python 对海量杂乱的文本进行截取与去重，最终拼装成结构化的 **JSONL 任务包**。
> 
> **这样，当数据流入 Label Studio 时，研发团队看到的不再是虚无缥缈的‘用户说不好用’，而是带着原始上下文、AI 错误代码、甚至用户真实纠正后代码的完美诊断病历。**”
