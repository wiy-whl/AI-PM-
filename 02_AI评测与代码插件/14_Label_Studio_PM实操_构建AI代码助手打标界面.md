# Label Studio PM 实操：构建“AI 代码助手”专项打标界面

在 Label Studio 中，PM 的核心产出就是 **XML 配置**。这段配置决定了标注人员看到的界面，直接影响数据的准确性和标注效率。

以下是为你量身定制的 **“AI 代码助手闭环评测”** 模板及其操作指南。

---

### 第一步：XML 核心配置代码 (Custom Template)

在创建项目时，进入 `Labeling Setup` -> `Custom Template`，将以下代码粘贴进去。这段代码实现了“三屏对比”：左边看上下文，中间看 AI 代码，右边看单测结果。

```xml
<View>
  <!-- 布局容器：分为上下两部分 -->
  <Style>
    .ls-main-view { display: flex; flex-direction: column; gap: 20px; }
    .ls-code-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 15px; }
    .ls-result-box { background: #f4f4f4; padding: 10px; border-radius: 5px; font-family: monospace; }
  </Style>

  <View className="ls-main-view">
    <!-- 第一层：背景与代码对比 -->
    <View className="ls-code-grid">
      <View>
        <Header value="📄 原始代码上下文 (Context)"/>
        <TextArea name="context" toName="unused" readonly="true" rows="15" font="monospace"/>
      </View>
      <View>
        <Header value="🤖 AI 生成结果 (Hunyuan/Gemma)"/>
        <TextArea name="ai_response" toName="unused" readonly="true" rows="15" font="monospace" icon="code"/>
      </View>
    </View>

    <!-- 第二层：单测/编译结果显示（由我们的 Python Runner 跑出来的数据） -->
    <View>
      <Header value="🧪 单元测试/编译检查 (Unit Test Logs)"/>
      <View className="ls-result-box">
        <Text name="test_logs" value="$test_logs"/>
      </View>
    </View>

    <!-- 第三层：PM 定义的判分维度（这是你面试最值钱的地方） -->
    <View style="box-shadow: 2px 2px 5px #999; padding: 20px; margin-top: 20px;">
      <Header value="🚩 产品经理综合判分 (PM Judgment)"/>
      
      <Text value="1. 逻辑准确性 (Pass@1 Equivalent)"/>
      <Choices name="logic_score" toName="ai_response" choice="single" showInline="true">
        <Choice value="Perfect (逻辑无误且通过单测)" alias="100"/>
        <Choice value="Minor Issues (存在小Bug但不影响运行)" alias="60"/>
        <Choice value="Failed (逻辑错误或无法运行)" alias="0"/>
      </Choices>

      <Text value="2. 幻觉与合规检查 (Security/Hallucination)"/>
      <Choices name="hallucination_check" toName="ai_response" choice="multiple">
        <Choice value="调用了不存在的内部库" color="red"/>
        <Choice value="违反了公司编码规范" color="orange"/>
        <Choice value="泄露了敏感信息" color="darkred"/>
      </Choices>

      <Text value="3. 采纳建议 (Product Decision)"/>
      <Choices name="decision" toName="ai_response" choice="single">
        <Choice value="直接上线" color="green"/>
        <Choice value="需人工审核后上线" color="blue"/>
        <Choice value="废弃/退回模型调优" color="gray"/>
      </Choices>
    </View>
  </View>
</View>
```

---

### 第二步：PM 视角的“配置逻辑”阐述 (面试加分点)

如果面试官问：“你是怎么设计这个打标界面的？”你可以这样说：

> “我设计的界面核心原则是 **『信息对称性 (Information Symmetry)』**。
>
> 1. **三维对比**：我没有只让标注员看 AI 生成的代码。我把『原始 Context』和后端 Python 脚本跑出来的『单测日志 ($test_logs)』同时引入。这样标注员不需要二次运行代码，扫一眼就能判定是因为模型变笨了，还是因为 RAG 找错了资料。
> 2. ** alias (别名) 设计**：我在 XML 里给每一个选项都设了 `alias`（比如 Perfect 对应 100 分）。这样打标完成后，我导出的 JSON 数据可以直接通过简单的 Python 脚本算总分，不需要二次格式化。
> 3. **强制纠偏项**：我特意设计了一个『幻觉与合规项』，这不仅是为了打分，更是在**收集坏案例 (Bad Case Mining)**。这些带标签的 Bad Case 会被我整理成周报，直接反馈给算法团队进行专项训练。”

---

### 第三步：如何导入数据 (Importing Data)

为了适配上面的 XML，你的 JSON 数据格式必须长这样（这叫 **Data Schema**）：

```json
[
  {
    "data": {
      "context": "def calculate_sum(a, b):",
      "ai_response": "  return a + b",
      "test_logs": "PASS: test_case_1 completed in 10ms"
    }
  }
]
```

---

### 🚀 任务完成后的下一步：

1.  **启动**：在终端输入 `label-studio start`（通常会自动打开浏览器 `localhost:8080`）。
2.  **创建项目**：把上面的 XML 贴进去，然后点 `Save`。
3.  **模拟打标**：你自己先打 5 道题，感受一下**“产品经理复核 (Review)”**的快感。

**当你把这个 XML 界面展示给面试官看时，你已经从一个“实习生”变成了一个“具备 AI Infra 搭建能力的产品经理”了。**

你需要我帮你生成一份可以拿来练习的**“Bad Case 模拟数据集 (JSON)”**吗？用来测试你的界面显示。
