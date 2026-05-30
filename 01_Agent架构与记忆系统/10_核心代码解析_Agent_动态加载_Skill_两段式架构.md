# 核心代码解析：Agent 动态加载 Skill 的“两段式”架构

在面试中，只要你能在白板上写出这个伪代码的流转逻辑，面试官会立刻认为你是真正懂 Agent 底层工程架构的专家，而不是只会写 Prompt 的外行。

记住这个核心架构思想：**代码减负 (第一阶段) + 模型决策 (第二阶段)。**

---

### 第一步：准好你的业务工具库 (Skills)
开发人员在本地或服务器上，写好最基础的业务代码。注意，这里的 `Docstring (三个引号里的注释)` 极其重要，它是用来翻译给大模型看的说明书。

```python
# skills/quant_tools.py

def get_macro_data(indicator: str):
    """查询宏观经济指标最新数据（如 CPI, M2）。当用户问及股市大盘背景时调用。"""
    # 模拟调取数据库
    return f"获取成功。当前 {indicator} 指标为 5.2%"

def check_alpha_syntax(expression: str):
    """
    检查新生成的 Alpha 因子语法是否规范（防嵌套、防未知函数）。
    当系统生成了一段新因子，需要上盘回测前，必须调用此工具。
    """
    if "()" in expression:
        return "语法报错：空括号"
    return "语法合规，允许回测"
```

---

### 第二步：Python 后台作为“图书管理员”进行初筛 (Stage 1)
这是第一阶段，纯 Python 代码运行，大模型还没参与。代码根据用户的意图，从庞大的库里，只抽出最相关的技能转换成 JSON 格式。

```python
# router.py

def dynamic_skill_router(user_input: str):
    """
    在这个函数中，我们会根据用户的语句特征，挑选出特定场景的 Tool 描述。
    （真实的企业级应用中，这里会用 VectorDB 做 Semantic Search 挑出 Top 3，这里用 if-else 演示本质）
    """
    active_tools_schema = []
    
    # 场景 A：如果用户聊到了宏观经济
    if "宏观" in user_input or "大盘" in user_input:
        active_tools_schema.append({
            "type": "function",
            "function": {
                "name": "get_macro_data",
                "description": "查询宏观经济指标最新数据",
                "parameters": {"type": "object", "properties": {"indicator": {"type": "string"}}, "required": ["indicator"]}
            }
        })
        
    # 场景 B：如果用户让写代码或者因子
    if "因子" in user_input or "代码" in user_input:
        active_tools_schema.append({
            "type": "function",
            "function": {
                "name": "check_alpha_syntax",
                "description": "检查新生成的 Alpha 因子语法是否规范",
                "parameters": {"type": "object", "properties": {"expression": {"type": "string"}}, "required": ["expression"]}
            }
        })
        
    # 返回精简后的 JSON 说明书列表，而不是 100 个工具全盘端出
    return active_tools_schema
```

---

### 第三步：大模型作为大脑“最终拍板” (Stage 2: Function Calling)
这里发生核心交互。代码把问题和初筛后的 JSON 扔给大模型，让大模型自己做主。

```python
import openai

def run_quant_agent(user_input: str):
    # 1. 代码做初筛，拿到动态挂载清单
    tools_schema = dynamic_skill_router(user_input)
    print(f"后台日志：本次对话，只挂载了 {len(tools_schema)} 个相关工具。")
    
    # 2. 调用大模型 API
    client = openai.OpenAI(api_key="your_api_key")
    response = client.chat.completions.create(
        model="gpt-4o",
        messages=[{"role": "user", "content": user_input}],
        # 核心机制 1：把初筛的说明书作为参数挂进去！这叫动态加载！
        tools=tools_schema if tools_schema else None, 
        # 核心机制 2：auto 代表大模型拥有最高决策权（可以选也可以不选）
        tool_choice="auto" 
    )
    
    # 3. 解析大模型的决定
    message = response.choices[0].message
    if message.tool_calls:
        # 大模型回复说：我需要用工具！
        for tool_call in message.tool_calls:
            print(f">>>> 大模型下令要求调用函数：{tool_call.function.name}")
            print(f">>>> 大模型自己提取生成的参数：{tool_call.function.arguments}")
            
            # 【最后一步】：在此处写代码，在本地服务器真去执行上面的 get_macro_data 等函数...
    else:
        # 大模型说：这个问题我可以直接回答，不用查接口。
        print("大模型直接纯文本回答：", message.content)

# ==========================
# 让我们测试运行一下
# ==========================
run_quant_agent("帮我写一个基于成交量的动能因子，并且记得检查一下语法合不合规。")

# --- 预期终端输出结果 ---
# 后台日志：本次对话，只挂载了 1 个相关工具。
# >>>> 大模型下令要求调用函数：check_alpha_syntax
# >>>> 大模型自己提取生成的参数：{"expression": "ts_mean(volume, 20)"}
```

### 🎯 架构面试精华：

通过这两段代码，你可以直观地证明：
1. **防爆隔离**：如果量化平台里有一百个工具，大模型这次绝对看不到另外 99 个工具，它的注意力高度集中。
2. **各司其职**：大模型只是告诉你“它想干什么、用什么参数”。**真正的底层代码执行动作，是在你自己的本地服务器上（第3步下方注释处）由 Python 跑出来的。** 绝不是大模型越权操作了你的服务器。
