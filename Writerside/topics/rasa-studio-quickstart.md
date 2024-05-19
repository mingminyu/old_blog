# 快速入门

<show-structure depth="3"/>

## 2. 案例介绍

本教程主要面对的是刚接触 Rasa 和对话机器人的同学，通过在 Rasa Studio 中创建机器人来介绍重要技术指标，以及对于这些基础概念的理解能够帮助你快速掌握 Rasa Studio 的使用。接下来，我们将搭建一个基于 CALM 的机器人，它能介绍自己，并和用户进行交流，帮助用户给朋友转账。

### 2.1 创建机器人

1. 以开发者或者流程创建者登入 Rasa Studio
2. 在下拉列表中选择 “**Create assistant project**”
3. 填写细节
    - 输入机器人名称（必须唯一），这里我们命名为 “transfer_money_assistant”
    - 这里选择默认选项 Modern，我们填上 Action Server URL，并选择 OpenAI 模型
4. 点击 “**Create assistant**” 按钮


### 2.2 创建欢迎流程

在 **CALM**(Conversational AI with Language Models) 中，机器人的业务逻辑被封装在一整套的流程，每一个流程描述了机器人的逻辑步骤来完整任务，它描述了从用户端所需的信息，从 API 或者数据库中的数据，以及用于所收集信息的逻辑分支。


Rasa 中的流程只描述了机器人所遵循的逻辑，但并非所有潜在路径都可以获取，如果你习惯于通过创建对话流程图来设计人工智能助手，你会发现 Rasa 中的流程要简单得多。


创建第一个流程:
- 选择最近创建的机器人
- 导航到流程创建的页面
- 点击创建流程，开始编写欢迎流程的草稿
- 输入流程名称，例如 `welcome_ane_self_introduction`，该描述有助于 Rasa 决定何时激活此流，也有助于任何查看您的流的人了解其目的。例如，如果用户说 “Hi!”，描述将引导 Rasa 机器人将 `welcome_and_self_introduction` 识别为相关流程。因此在“**描述**”字段中，明确说明该流程的目标，如“问候并介绍助理，并询问用户需要帮助什么”。然后，单击“**保存**”。
- 流程被创建后，我们就可以点击它开始创建

#### 介绍机器人

Rasa 专注于建立为用户实现目标的机器人，CALM 范式支持 LLM 与业务逻辑的本地集成。通过使用 CALM，您可以简单地使用流程来描述您的业务逻辑，每个流程都包含从用户收集信息、调用 API 和遵循分支逻辑的步骤。

关于这个业务逻辑，需要注意的最重要一点是，它不包括对对话中用户方面的任何描述。没有标记 `intent` 的步骤，也没有描述 “如果用户说了这个，然后又说了这个” 的内容行，对话的用户端则可以通过[无意愿策略](https://rasa.com/docs/rasa/next/llms/llm-intentless)进行处理。

- 例如，当用户输入 “Hi”，我们的对话理解组件（LLMCommandGenerator）应该生成一个命令 StartFlow(`'welcome_and_self_introduction'`) 作为响应。在这个 `welcome_and_self_introduction` 流程中，我们的目标是用一条简单的“你好！”消息来问候用户，并介绍助理。要开始构建此流，请单击“**+**”图标并选择“**消息**”步骤。

![欢迎流程](https://rasa.com/docs/studio/img/welcome-flow-message.png)

- 点击“**选择消息**”，从菜单中选择“**创建消息**”

![创建消息](https://rasa.com/docs/studio/img/welcome-flow-create-message.png)

- 填写细节
  - 输入姓名(utter_greet) 和消息(Hello! I'm Sbot, a friendly assistant created using Rasa Studio) 
  - 勾选复选框 “Use Contextual Response Rephraser”，以便根据对话的上下文重新表述此消息
  - 设置好之后点击保存


![创建消息](https://rasa.com/docs/studio/img/welcome-flow-utter-greet.png)


### 2.3 创建转账流程

现在，让我们构建帮助用户进行汇款的流程，允许 LLM 在用户询问汇款时激活此流程。要执行此操作：
1. 返回流列表。
2. 创建一个名为 `money_transfer` 的流程，其描述为: “Ask all the required questions to process a money transfer request from users.” 单击“保存”。
3. 从列表中选择流程以继续构建它

#### 收集有关信息

- 创建“**收集信息**”步骤，询问用户谁将是转账的收件人

![创建收集信息](https://rasa.com/docs/studio/img/money-transfer-collect-info.png)

- 通过填写“**说明**”字段，指导机器人在本步骤中收集哪些信息，例如，“此步骤询问付款的收款人”。

![收集信息](https://rasa.com/docs/studio/img/money-transfer-description.png)







<seealso>
<category ref="ref_docs">
    <a href="https://rasa.com/docs/studio/start-here/tutorial-flow-based">Tutorials for CALM assistants</a>
</category>
<category ref="ref_github">
</category>
<category ref="ref_issues">
</category>
<category ref="ref_hf">
</category>
<category ref="ref_ms">
</category>
</seealso>


