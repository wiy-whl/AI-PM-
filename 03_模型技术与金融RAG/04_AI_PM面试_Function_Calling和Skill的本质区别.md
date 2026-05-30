# AI PM 面试：Function Calling 和 Skill 的本质区别

如果面试官问：“Function Calling 和 Skill（技能/插件）有什么区别？”
这个问题的核心是考察你对 **AI 架构分层（Abstraction Layers）** 的理解。

初级 PM 会说：“没区别，Skill 底层就是用 Function Calling 写的。”
**高阶 PM 会说：“它们是两个不同维度的概念：Function Calling 是底层的‘生理学发力机制’，而 Skill 则是产品层的‘拳法套路’。”**

---

## 🛠️ 1. Function Calling：原子的“动作机理” (L1 级别)

*   **定义**：它是大模型的一种**基础原语能力 (API 规范)**。它定义了大模型如何暂停对话，按特定 JSON Schema 输出参数的过程。
*   **特性**：
    *   **极度底层**：它不管业务逻辑，只管“按照这个参数格式把字符串吐出来”。
    *   **原子化**：`check_weather(city="Shenzhen")` 是一个 Function Call。
    *   **开发者视角**：它是给写代码的工程师看的机制，普通用户根本不知道它的存在，也不该看到 JSON。

**【比喻】**：Function Calling 就像是人体发出的一条**基础肌肉收缩电信号**（比如控制食指弯曲）。

---

## 🎒 2. Agent Skill：产品化的“能力包” (L2 或 L3 级别)

*   **定义**：Skill（技能）是一个**产品学上的封装概念**。它通常包含了不仅一个，而是多个 Function Call，同时还打包了专属的系统提示词（System Prompt）、特定的知识库（RAG/MCP）、以及重试/容错的工作流（Workflow）。
*   **特性**：
    *   **极度业务化**：它是为了解决一个具体的业务场景而生的。
    *   **全家桶组合**：一个叫“退款助手”的 Skill，它底层可能组合了 `查询订单流`、`校验物流`、`发起退款` 等 3 个 Function Call，外加一套“安抚用户的标准话术库”。
    *   **用户视角**：它是暴露给最终用户的。用户可以在 Agent 的商店里点击“安装‘数据清洗’ Skill”，用户感受到的是能力的增加。

**【比喻】**：Skill 就像是一套编排好的**“一套太极拳”**。为了打出这套拳，底层必须要调用数千次“肌肉收缩信号”（Function Calling），但普通学武的人（User）只需购买这本《太极拳谱》（Skill），不需要去研究神经末梢怎么放电。

---

## 🎯 面试满分话术演练

**面试官**：“你简历里写你给 Agent 开发了 5 个 Skill。那请问 Skill 和底层的 Function Calling 是什么关系？”

**候选人（你）：**
> “在我们的架构设计中，**Function Calling 是毛坯，Skill 是精装房。**
> 
> 在一开始的敏捷开发中，我们确实把一个个独立的 Function Calling（比如查数据库、发邮件）直接暴露给了基础 Agent。但很快遇到了**严重的代码耦合和幻觉问题**。
>
> 为此，我作为 PM 引入了 **Skill（技能包）的高层抽象概念**。
> 一个 Skill，不仅仅是包含了一个 API 接口。它是一个高度封装的 **‘能力胶囊’**。里面打包了：
> 1.  实现该闭环所需的 2-3 个紧密关联的 Function Tools。
> 2.  针对这个能力的专属 Few-Shot Context (打样数据)。
> 3.  遇到 API 报错报错时的专有兜底重试逻辑 (Reflection Loop)。
> 
> 我让研发把这些东西全部封装在一个独立的模块中，对外只暴露出一个‘XX 技能’的宏观接口。
> 这样的好处是：**非技术背景的业务运营人员（比如财务部门），在配置他们的财务 Agent 时，不需要去学晦涩的 JSON Schema（Function Calling 层），他们只需在界面上勾选挂载‘发票开具 Skill’，Agent 瞬间就能获得这项完整的业务闭环能力。**
> 这就是我们如何用产品抽象手段，降低 AI 使用门槛的实战经验。”

---
我已经将这份架构分层解析收录进你的面试包：`d:\PromptX-main\worldbrain\AI_PM_Interview_Pack\function_calling_vs_skill.md`。
