# 闲聊和FAQ

<show-structure depth="3"/>

FAQ 对话机器人是最容易构建的对话机器人，通常也是大家构建的第一种对话机器人。本页面是处理 FAQ 和闲聊等非上下文问题所需的概念和训练数据的指南。

FAQ 和闲聊是对话机器人使用一组固定的消息进行响应，同时无论之前发生什么对话机器人都应始终以相同的方式进行回答的两种情况。例如，在接下来的对话中，每个问题都可以在对话的任何时候被问到，答案与用户之前所说的内容无关。

![闲聊](https://rasa.leovan.tech/images/chitchat-faqs/chitchat.png)

## 1. 用于 FAQ 和闲聊的响应选择器使用

为了处理 FAQ 和闲聊你需要一个基于规则的对话管理策略（**RulePolicy**）和一个简单的方式来对问题做出合适的响应（**ResponseSelector**）。

### 1.1 更新配置

对于 FAQ 和闲聊，你希望对话机器人对于每次问到的相同类型问题都以相同的方式做出回应。规则可以用来达成这个效果。要使用规则，你需要在配置文件中将 RulePolicy 添加到策略里：

```yaml
policies:
  - name: RulePolicy
```

之后，将 `ResponseSelector` 包含在配置文件中的 NLU 流水线中。`ResponseSelector` 需要一个特征提取器和意图分类器才能工作，因此它应该位于流水线中这些组件之后，例如：

```yaml
pipeline:
  - name: WhitespaceTokenizer
  - name: RegexFeaturizer
  - name: LexicalSyntacticFeaturizer
  - name: CountVectorsFeaturizer
  - name: CountVectorsFeaturizer
    analyzer: char_wb
    min_ngram: 1
    max_ngram: 4
  - name: DIETClassifier
    epochs: 100
  - name: EntitySynonymMapper
  - name: ResponseSelector
    epochs: 100
```

默认情况下，`ResponseSelector` 将为所有检索意图构建一个检索模型，要分别为 FAQ 和闲聊设置响应，需要使用多个 `ResponseSelector` 组件来指定 `retrieval_intent` 键：

```yaml
pipeline:
  - name: ResponseSelector
    epochs: 100
    retrieval_intent: faq
  - name: ResponseSelector
    epochs: 100
    retrieval_intent: chitchat
```


### 1.2 定义检索意图和 ResponseSelector

考虑一个包含 20 种不同 FAQ 的示例，尽管每个问题都表示为一个单独的意图，但所有 FAQ 意图在对话中都以相同的方式进行处理。对于每个 FAQ 意图，话机器人会根据提出的问题检索正确的响应。

你可以使用一个简单的动作，例如 `utter_faq`，通过一个将它们汇总成为一个单独的名为 `faq` 的检索意图来处理所有 FAQ，而不是编写 20 条规则。

单一的动作使用 `ResponseSelector` 的输出来为用户询问的特定 FAQ 返回正确的响应。

### 1.3 创建规则

针对每个检索意图只需要编写一个规则。包含在检索意图下的所有意图都将以相同的方式进行处理。动作的名称以 `utter_` 开头并以检索意图的名称结果。编写用于响应 FAQ 和闲聊的的规则：

```yaml
rules:
  - rule: respond to FAQ
    steps:
    - intent: faq
    - action: utter_faq
  - rule: respond to chitchat
    steps:
      - intent: chitchat
      - action: utter_chitchat
```

`utter_faq` 和 `utter_chitchat` 动作将使用 `ResponseSelector` 的预测来返回实际的响应消息。

### 1.4 更新 NLU 训练数据

用于 `ResponseSelector` 的 NLU 训练样本同常规训练样本一样，只是他们的名称必须参考被分组的检索意图：

```yaml
nlu:
  - intent: chitchat/ask_name
    examples: |
      - What is your name?
      - May I know your name?
      - What do people call you?
      - Do you have a name for yourself?
  - intent: chitchat/ask_weather
    examples: |
      - What's the weather like today?
      - Does it look sunny outside today?
      - Oh, do you mind checking the weather for me please?
      - I like sunny days in Berlin.
```

请确保在领域文件中包含添加的 `chitchat` 意图：

```yaml
intents:
  - chitchat
```

### 1.5 定义响应

`ResponseSelector` 的响应遵循检索意图相同的命名约定。除此之外，它们还可以有同常规对话机器人响应的所有特征。对于上面列出的闲聊意图，响应可以如下所示：

```yaml
responses:
  utter_chitchat/ask_name:
  - image: "https://i.imgur.com/zTvA58i.jpeg"
    text: Hello, my name is Retrieval Bot.
  - text: I am called Retrieval Bot!
  utter_chitchat/ask_weather:
  - text: Oh, it does look sunny right now in Berlin.
    image: "https://i.imgur.com/vwv7aHN.png"
  - text: "I am not sure of the whole week but I can see the sun is out today."
```

## 2. 总结

完成如下操作后，就可以训练对话机器人并进行测试了！

- 在 `config.yml` 文件中将 `RulePolicy` 添加至策略中，将 `ResponseSelector` 添加到流水线中。
- 添加至少一个用于响应 FAQ 或闲聊的规则。
- 为 FAQ 或闲聊意图添加样本。
- 为 FAQ 或闲聊意图田间响应。
- 在领域中更新意图。

- 现在，你的对话机器人应该能够正确且一致地响应 FAQ 或闲聊了，即使这些插入语发生在对话机器人正在帮助用户完成另一项任务时。

## 3. 补充

在实践中发现，我们想要使用 FAQ 模式来回答固定知识库问题，但是当对应的 `utter_faq/xxx` 的内容超过 512 字符长度时，Rasa 训练过程中在 `LanguageModelFeaturizer` 这一节点便会出现如下异常进而中断。

```Bash
is too long  for the model chosen bert which has a maximum sequence 
length of 512 tokens. Either shorten the message or use a model which 
has no restriction on input sequence length like XLNet.
```

这是因为 FAQ 以及 ChitChat 模式，它会将 FAQ 意图以及对应的响应都进行编码，从而导致超过 Bert 的最大长度。如果当我们的回复的内容长度过多时，建议使用直接的 `intent` 以及对应的 `utter` 来建立规则，而非采用 FAQ 模式。


<seealso>
<category ref="ref_docs">
    <a href="https://rasa.leovan.tech/chitchat-faqs">闲聊和FAQ</a>
</category>
<category ref="ref_github">
</category>
</seealso>


