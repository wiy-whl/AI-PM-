# AI PM 实操：如何定义一个“好”的 Function Calling？

在面试中，如果光讲理论（加描述、加枚举），面试官可能会觉得你只是在背书。但如果你能在一张白纸上，**手写一段优劣对比的 JSON Schema**，你将立刻建立起极强的“技术鉴赏力”人设。

所谓“制作一个好的 Function Calling”，核心就是写好提供给大模型的 **Tool Definition (工具定义 JSON)**。

---

## ❌ 反面教材 (Bad Case)：典型的“初级研发视角”

很多研发在写 Tool 的时候，习惯把它当成代码里的函数注释来写，非常简略。

```json
{
  "name": "get_stock_data",
  "description": "获取股票数据",
  "parameters": {
    "type": "object",
    "properties": {
      "ticker": {
        "type": "string",
        "description": "股票代码"
      },
      "timeframe": {
        "type": "string",
        "description": "时间范围"
      }
    },
    "required": ["ticker", "timeframe"]
  }
}
```

**为什么烂？（PM 的痛点诊断）**
1. **路由混乱**：“获取股票数据”太宽泛了。如果用户问“大盘走势”，模型也会调它，但这个接口其实只能查单只股票。
2. **凭空捏造 (Hallucination)**：`timeframe` 是 `string` 自由类型。用户问“查一下最近一阵子的”，模型可能会自动把 `timeframe` 填成 `"recent"` 或者 `"最近一阵子"`，结果后端 API 接收到非法参数，直接 500 报错。
3. **股票代码格式错误**：是填 `AAPL` 还是 `NASDAQ:AAPL`？如果没有要求，模型会乱猜。

---

## ✅ 满分模板 (Good Case)：真正的“AI PM 视角”

一个优秀的 AI PM，会把 Tool Definition 当成 **微型 Prompt** 来精雕细琢。

```json
{
  "name": "get_single_stock_historical_price",
  "description": "查询单只股票的历史和实时价格数据。适用场景：用户明确询问某一只具体股票的价格走势时。禁止场景：切勿使用此工具查询宏观板块数据、也不要用于查询公司财报面数据。",
  "parameters": {
    "type": "object",
    "properties": {
      "ticker_symbol": {
        "type": "string",
        "description": "标准化的股票代码，必须包含交易所前缀。如果是美股，格式为 'EXCHANGE:TICKER' (例如 'NASDAQ:AAPL', 'NYSE:TSLA')。如果用户未提到交易所，请务必根据常识补充。"
      },
      "period": {
        "type": "string",
        "description": "查询的时间跨度，必须从给定选项中严格选取。",
        "enum": ["1D", "5D", "1M", "3M", "6M", "1Y", "MAX"]
      },
      "include_after_hours": {
        "type": "boolean",
        "description": "是否包含盘后交易数据。如果用户没有特别指明，默认填 false。"
      }
    },
    "required": ["ticker_symbol", "period"]
  }
}
```

---

## 💡 核心优化点拆解（面试时这么讲）

当面试官问你“如何制作一个好的 Function Calling”时，你可以直接复盘上面这个例子：

1.  **具象化函数名 (Naming)**：
    *   从 `get_stock_data` 改成了 `get_single_stock_historical_price`。
    *   大模型的**路由权重**很大一部分依赖于函数名，名字越长、限定词越多（单只、历史、价格），匹配越精准。
2.  **设置“红绿灯”描述 (Traffic Light Description)**：
    *   明确写入 **“适用场景（绿灯）”** 和 **“禁止场景（红灯）”**。告诉它不能查财报，它就不会在用户问财务问题时来瞎凑热闹。
3.  **强制枚举收敛 (Enum Restriction)**：
    *   把 `timeframe` (自由字符串) 变成了 `period`，并增加了 `"enum": ["1D", "5D", ...]`。
    *   **这是根治幻觉的杀手锏**，把无限的输入空间，锁死在后端的合法字典里。
4.  **默认行为预判 (Default Fallback)**：
    *   在 `include_after_hours` 中写明：“如果用户没提，默认填 false”。这避免了大模型在遇到信息不足时，反问用户“你需要看盘后数据吗？”而打断对话流畅度，或者它自己抛硬币瞎猜一气。

---
我已经将这份对比清单存入面试包：`d:\PromptX-main\worldbrain\AI_PM_Interview_Pack\good_function_calling_template.md`。
