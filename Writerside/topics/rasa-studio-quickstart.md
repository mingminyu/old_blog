# 快速入门

<show-structure depth="3"/>

## 1. Rasa Studio 介绍

Rasa Studio 是一个无代码图形用户界面（UI），使业务用户能够大规模协同构建、审查和改进对话体验。由于 Rasa Studio 是在 Rasa Pro 基础上构建来的，它与我们最先进的生成人工智能原生对话引擎 CALM（带语言模型的对话人工智能）有着深刻的联系。使用 Rasa Studio 的企业团队能够在无需一行代码的情况下，便能构建卓越的对话式客户体验。

### 1.1 Rasa Studio 重要能力

#### 1.1.1 可视化机器人的创建（流程创建器）

Rasa Studio 集成了一个强大的功能，称为流程创建器(Flow Builder)，以 CALM（具有大型语言模型的会话人工智能）为基础。流程创建器通过将会话状态机流程图的结构化、循序渐进的方法与 LLM 
在处理边缘案例时的灵活性相结合，简化了机器人的创建和开发过程。

> 但是需要注意的是，当前版本的流程创建器**不支持**自然语言理解（NLU）组件，例如意图和实体。
> 
{style="warning"}

#### 1.1.2 在用户界面中训练模型

Rasa Studio 的一个非常突出功能就是用户能够直接在**用户界面**（UI）中训练自己的机器人。此功能简化了训练过程，加快了机器人的准备工作。感兴趣的话，还可以在此处找到有关此功能的[详细信息](https://rasa.com/docs/studio/user-guide/training)。

#### 1.1.3 飞行测试和内容调整

Rasa Studio 允许用户直接在 UI 中测试他们的机器人，该功能的与众不同之处在于，无需部署即可对机器人的内容进行实时调整。


#### 1.1.4 多用户协作环境

Rasa Studio 促进了喜欢使用命令行界面（CLI）和喜欢使用 UI 的开发人员之间的协作，有关如何最大限度地利用 Rasa Studio 进行交叉协作，请参阅相关文档。

#### 1.1.5 基于 NLU 的机器人的注释

如果项目涉及基于 NLU 的机器人，Rasa Studio 提供了直观和协作的注释功能，该功能使用户能够通过增加对**实体**和**意图**的注释来扩展他们的理解，从而增强机器人的能力。同时 Rasa Studio 还允许我们观察用户和机器人之间发生的对话，可以在此处了解有关此功能的更多[信息](https://rasa.com/docs/studio/user-guide/annotation)。

#### 1.1.6 NLU 数据集中式管理

Rasa Studio 还集成了一个内容管理系统（CMS），用于简化 NLU 数据（包括意图和实体）的管理。这种特性允许我们集中进行更新，然后在整个机器人中进行应用。此外，它还提供了生成 NLU 
数据的统一方法，对于重复创建的数据能够自动消除。


#### 1.1.7 导入/导出

Rasa Studio 提供了导入导出 NLU 和流程数据的功能，这有助于开发技术人员和 Studio 用户之间的协作，该特性可实现对话式人工智能项目的无缝共享和协作。技术用户可以导出相关数据，然后 Studio 用户可以将这些数据导入到他们的环境中，从而促进流畅高效的协作工作流程。这种协作方法确保技术和非技术团队成员都能有效地为对话式人工智能的开发和增强做出贡献。


#### 1.1.8 基于角色的访问控制(RBAC)

> Role Based Access Control, RBAC 

基于角色的访问控制（RBAC）使管理员能够高效地建立用户并将其分配给预定义的角色，这一功能使具有不同专业知识的用户能够专注于各自的能力领域，同时也能保护用户的敏感数据。总而言之，RBAC 的作用是确保团队成员只能访问与其特定角色相关的信息。






## 2. 案例介绍

本教程主要面对的是刚接触 Rasa 和对话机器人的同学，通过在 Rasa Studio 中创建机器人来介绍重要技术指标，以及对于这些基础概念的理解能够帮助你快速掌握 Rasa Studio 的使用。接下来，我们将搭建一个基于 CALM 的机器人，它能介绍自己，并和用户进行交流，帮助用户给朋友转账。

### 2.1 创建机器人

1. 以开发者或者流程创建者登入 Rasa Studio
2. 在下拉列表中选择 “**Create assistant project**”

![Create assistant project](https://rasa.com/docs/studio/img/create-assistant-project.png)

3. 填写细节
    - 输入机器人名称（必须唯一），这里我们命名为 “**transfer_money_assistant**”
    - 这里选择默认选项 Modern，我们填上 Action Server URL，并选择 OpenAI 模型

![填写细节](https://rasa.com/docs/studio/img/getting-started.png)

4. 点击 “**Create assistant**” 按钮


### 2.2 创建欢迎流程

在 **CALM**(Conversational AI with Language Models) 中，机器人的业务逻辑被封装在一整套的流程，每一个流程描述了机器人的逻辑步骤来完整任务，它描述了从用户端所需的信息，从 API 或者数据库中的数据，以及用于所收集信息的逻辑分支。


Rasa 中的流程只描述了机器人所遵循的逻辑，但并非所有潜在路径都可以获取，如果你习惯于通过创建对话流程图来设计人工智能助手，你会发现 Rasa 中的流程要简单得多。


创建第一个流程:
- 选择最近创建的机器人
- 导航到流程创建的页面
- 点击创建流程，开始编写欢迎流程的草稿

![创建流程草稿](https://rasa.com/docs/studio/img/create-flow.png)

- 输入流程名称，例如 `welcome_ane_self_introduction`，该描述有助于 Rasa 决定何时激活此流，也有助于任何查看您的流的人了解其目的。例如，如果用户说 “Hi!”，描述将引导 Rasa 机器人将 `welcome_and_self_introduction` 识别为相关流程。因此在“**描述**”字段中，明确说明该流程的目标，如“问候并介绍助理，并询问用户需要帮助什么”。然后，单击“**保存**”。

![欢迎流程](https://rasa.com/docs/studio/img/welcome-flow.png)

- 流程被创建后，我们就可以点击它开始创建

![开始欢迎流程](https://rasa.com/docs/studio/img/start-welcome-flow.png)

#### 介绍机器人

Rasa 专注于建立为用户实现目标的机器人，CALM 范式支持 LLM 与业务逻辑的本地集成。通过使用 CALM，您可以简单地使用流程来描述您的业务逻辑，每个流程都包含从用户收集信息、调用 API 和遵循分支逻辑的步骤。

关于这个业务逻辑，需要注意的最重要一点是，它不包括对对话中用户方面的任何描述。没有标记 `intent` 的步骤，也没有描述 “如果用户说了这个，然后又说了这个” 的内容行，对话的用户端则可以通过[无意图策略](https://rasa.com/docs/rasa/next/llms/llm-intentless)进行处理。

- 例如，当用户输入 “Hi”，我们的对话理解组件（LLMCommandGenerator）应该生成一个命令 StartFlow(`'welcome_and_self_introduction'`) 作为响应。在这个 `welcome_and_self_introduction` 流程中，我们的目标是用一条简单的“你好！”消息来问候用户，并介绍助理。要开始构建此流，请单击“**+**”图标并选择“**消息**”步骤。

![欢迎流程](https://rasa.com/docs/studio/img/welcome-flow-message.png)

- 点击“**选择消息**”，从菜单中选择“**创建消息**”

![创建消息](https://rasa.com/docs/studio/img/welcome-flow-create-message.png)

- 填写细节
  - 输入名称(`utter_greet`) 和消息(`Hello! I'm Sbot, a friendly assistant created using Rasa Studio`) 
  - 勾选复选框 “**Use Contextual Response Rephraser**”，以便根据对话的上下文重新表述此消息
  - 设置好之后点击“**保存**”

![创建消息](https://rasa.com/docs/studio/img/welcome-flow-utter-greet.png)

### 2.3 创建转账流程

现在，让我们构建帮助用户进行汇款的流程，允许 LLM 在用户询问汇款时激活此流程。要执行此操作：
1. 返回流程列表
2. 创建一个名为 `money_transfer` 的流程，其描述为: “Ask all the required questions to process a money transfer request from users.” 单击“**保存**”。
3. 从列表中选择流程以继续构建它

![创建转账流程](https://rasa.com/docs/studio/img/create-money-transfer-flow.png)


#### 2.3.1 收集有关信息

1. 创建“**收集信息**”步骤，询问用户谁将是转账的收款人

![创建收集信息](https://rasa.com/docs/studio/img/money-transfer-collect-info.png)

2. 通过填写“**说明**”字段，指导机器人在步骤中收集哪些具体信息，例如，“此步骤询问付款的收款人”。

![收集信息](https://rasa.com/docs/studio/img/money-transfer-description.png)


3. 创建一个插槽，定义机器人应该从用户回复中收集的具体信息。插槽相当于机器人的内存，通过键值对形式的存储，保存用户提供的信息（如接收者的名称）或从外部来源收集的信息（例如数据库查询的结果）。要创建插槽，请单击“**选择或创建插槽**”字段，然后选择“**创建插槽**”。


![创建转账流程的插槽](https://rasa.com/docs/studio/img/money-transfer-create-slot.png)


4. 输入 `recipient` 作为槽插槽名称，然后选择 `Text` 作为其类型，文本插槽用于存储文本信息，例如个人姓名、国家名称、城市名称等。单击“**保存**”。如果感兴趣的话，请花点时间了解有关[插槽机类型]
(https://rasa.com/docs/rasa/domain#slot-types)的更多信息。

![插槽类型](https://rasa.com/docs/studio/img/money-transfer-recipient-slot.png)

5. 最后生成一条消息，机器人将使用该消息从用户收集相关信息，接下来点击“**创建消息**”。

![创建转账流程的消息](https://rasa.com/docs/studio/img/money-transfer-create-message.png)


6.系统通过将插槽名称与 `utter_ask` 组合来自动填写消息名称，从而得到 `utter_ask_recipient`，输入消息内容“您想向谁汇款？”

![询问收款人](https://rasa.com/docs/studio/img/money-transfer-utter-ask-recipient.png)


7. 默认情况下，“**使用上下文响应重述器**” 选项处于启用状态。该功能使机器人能够根据对话的上下文，通过提示 LLM 来重新表述其回答。这有助于让机器人的反应更加有组织性和对话性。感兴趣的话，可以了解[更多信息]
(https://rasa.com/docs/rasa-pro/concepts/components/llm-custom/#contextualresponserephraser)。

![重述器](https://rasa.com/docs/studio/img/rephrasing.png)

> 如果您希望完全控制响应，您可以手动添加消息变体。只需**取消**选中 **Rephraser** 复选框，并单击“**添加变体**”按钮即可。


![添加响应变体](https://rasa.com/docs/studio/img/money-transfer-add-variaion.png)


8. 添加任意数量的响应变体，然后单击“**保存**”。

![](https://rasa.com/docs/studio/img/money-transfer-variations.png)


9. 默认情况下，如果插槽已被填充，则可以跳过“**收集信息**”步骤。例如，如果用户说“我想转账给 John”，机器人就会知道收款人的名字，而不需要问这个问题来填补空缺。如果您希望机器人始终提出问题，即使插槽已经被填充，也可以启用“**填写前询问**”选项。然而，在教程场景和这个特定的问题中，我们可以将其禁用。

![填写前询问](https://rasa.com/docs/studio/img/money-transfer-ask-before-filling.png)

10. 保持“**重置插槽值**”选项处于启用状态。这将在流程的最后步骤后重置槽，清除其值并在未来的会话中重新触发其问题。它对于 `transfer_amount` 或 `recipient_name` 等插槽非常有用，这些插槽在每个会话中都有所不同。如果“**填充前询问**”处于关闭状态，禁用此选项将**保留槽值**并跳过问题。


![重置槽值](https://rasa.com/docs/studio/img/money-transfer-reset-slot-value.png)


#### 2.3.2 收集金额信息

1. 我们继续增加一个“**收集信息**”步骤，询问用户转账的金额。

![收集转账金额信息](https://rasa.com/docs/studio/img/money-transfer-collect-info-amount.png)

2. 在描述填写内容为 “此步骤查询付款金额，单位为美元”。

![转账金额的描述](https://rasa.com/docs/studio/img/money-transfer-amount-description.png)


3. 创建一个新的插槽

![转账金额插槽](https://rasa.com/docs/studio/img/money-transfer-amount-slot.png)

4. 将插槽名称命名为 `amount`，然后选择 `Float` 作为其类型，用于存储可以有小数点的数值，单击“**保存**”。

![创建转账金额插槽](https://rasa.com/docs/studio/img/money-transfer-amount-slot-create.png)

5. 最后生成一条消息，机器人将使用该消息从用户那里收集这些信息。要执行此操作，请单击“**创建消息**”。

![转账金额的消息](https://rasa.com/docs/studio/img/money-transfer-amount-message.png)

6. 系统将自动使用 `utter_ask_amount` 填充消息。对于消息内容，您可以使用“您想发送多少钱（以美元计）？”

![转账金额的询问](https://rasa.com/docs/studio/img/money-transfer-utter-ask-amount.png)

7. 我们可以选择添加按钮，为用户提供结构化的选择，从而简化用户快速选择答案的过程。为此，请在消息下方的输入字段中输入所需的值。键入每个值后按“**Enter**”键。完成后，单击“**保存**”。

![](https://rasa.com/docs/studio/img/money-transfer-message-buttons.png)


8. 这里我们禁用“**填写前询问**”。如果用户以“我想转账50美元”开始对话，机器人将已经填满了空位，不会再继续询问金额。通过启用“**重置插槽值**”以确保下次用户请求转账时，机器人会再次被要求向用户询问转账金额。

![询问转账金额](https://rasa.com/docs/studio/img/money-transfer-ask.png)


#### 2.3.3 验证资金是否充足


在这一步中，我们旨在验证用户是否有足够的资金进行转账。理想情况下，这是使用自定义操作（API 调用）来检查用户的余额并返回响应。然而，在本教程的第一部分中，我们将使用简单的逻辑来验证资金充足性。

1. 添加“**逻辑**”步骤，这种类型的步骤允许您根据条件设置分支流程。

![给转账添加逻辑](https://rasa.com/docs/studio/img/transfer-money-add-logic.png)


2. 我们先定义一个资金充足的条件。要执行此操作，请单击第一个条件来创建金额小于或等于$1000 的场景。如果交易金额小于可用资金，则表明有足够的资金可用。选择第一个条件并将其设置为金额小于或等于 1000。

![转账金额的逻辑](https://rasa.com/docs/studio/img/money-transfer-logic-amount.png)


3. 对于用户资金不足的情况，我们则需要通知他们。
    - 在 “**Else**” 分支之后创建一个“**消息**”步骤
    - 添加一条新消息，命名为 `utter_insufficient_funds`，文本内容为“您没有足够的资金来完成此交易”，并单击“**保存**”


![Else 分支](https://rasa.com/docs/studio/img/money-transfer-else-message.png)

![没有充足资金的消息](https://rasa.com/docs/studio/img/money-transfer-utter-unsufficient-finds.png)


#### 2.3.4 确认请求

当用户确认他们有足够的资金时，我们可以继续下一步：验证我们是否正确理解了用户的请求。

1. 在第一个条件之后，添加“**收集信息**”步骤

![条件收集信息](https://rasa.com/docs/studio/img/money-transfer-condition-collect-info.png)

2. 添加描述“此步骤要求用户确认收件人和转账金额”

![添加描述](https://rasa.com/docs/studio/img/transfer-money-collect-description.png)

3. 创建插槽并将其命名为 `final_confirmation`，由于您问的是“**是**”或“**否**”问题，因此可以将结果存储为布尔类型。

![转账最终确认](https://rasa.com/docs/studio/img/transfer-money-final-confirmation.png)

4. 创建新机器人的消息，它将自动填充名称 `utter_ask_final_confirmation`。输入消息“`请确认是否要将｛amount｝转账给{recipient}`”。我们使用**大括号来引用槽值**，因此前面问题中保存的**金额**和**收款人**将插入此处。

![转账最终确认的消息](https://rasa.com/docs/studio/img/transfer-money-utter-ask-final-confirmation.png)

5. 这里启用“**填充前询问**”，以便下次用户转账时，Sbot 不会自动填充此插槽，而是会被要求再次确认。

![再次确认](https://rasa.com/docs/studio/img/transfer-money-ask-2.png)


6. 添加“逻辑”步骤

![添加逻辑](https://rasa.com/docs/studio/img/transfer-money-logic-2.png)

7. 对于用户确认请求的场景（第一个条件分支），输入以下详细信息：`final_confirmation` 为 True。

![最终确认为是的条件分支](https://rasa.com/docs/studio/img/transfer-money-final-confirmation-true.png)


8. 对于 Else 分支，不需要创建任何特定条件，它将自动覆盖其他条件分支中未满足的任何场景。

#### 2.3.5 显示成功和取消消息

如果用户确认了交易请求，我们将提供一条成功消息。

1. 在 `final_confirmation` 为 `true` 分支之后，添加一个“消息”步骤

![转账消息](https://rasa.com/docs/studio/img/money-transfer-message2.png)

2. 创建一条名为 `utter_transfer_success` 的新消息，其文本内容为“`全部完成，｛amount｝已发送给｛recipient｝`”。

![转账成功消息](https://rasa.com/docs/studio/img/money-transfer-successful.png)

3. 对于 Else 分支，还要创建一条新消息

![Else 分支](https://rasa.com/docs/studio/img/money-transfer-message-3.png)

4. 将其命名为 `utter_cancel_transfer`，并添加文本 “**转账取消**”。

![转账取消](https://rasa.com/docs/studio/img/money-transfer-cancel.png)


#### 2.3.6 链接到反馈流程

现在让我们请用户对机器人的表现留下反馈，如果我们正在构建一个复杂的助手，那么反馈的逻辑片段可以在多个场景中重复使用，这就是为什么将其分离为另一个流程是有意义的。

1. 在“**传输成功**”和“**传输取消**”消息后添加“**链接**”步骤。

![转账链接](https://rasa.com/docs/studio/img/money-transfer-link.png)


2. 在下拉列表中选择“**创建流程**”

![链接反馈流程](https://rasa.com/docs/studio/img/money-transfer-link-create-flow.png)


3. 将新流程命名为 `leave_feedback`，确保它只在 `transfer_money` 流程完成后启动，而不是自动触发，请将其描述设置为“从不预测此流程的 StartFlow；用户不应该自己触发它”。

![反馈新流程](https://rasa.com/docs/studio/img/leave-feedback-new-flow.png)


### 2.4 创建反馈流程

1. 返回到刚刚创建的 `leave_feedback` 流程

![反馈流程](https://rasa.com/docs/studio/img/leave-feedback-flow.png)


2. 从“**收集信息**”步骤开始，询问用户是否希望留下反馈：
    - 添加描述“请用户留下反馈”
    - 使用布尔类型创建一个名为 `feedback` 插槽
    - 添加一条名为 `utter_ask_feedback` 的机器人消息，并添加文字内容“你想留下反馈吗？”

![反馈信息收集](https://rasa.com/docs/studio/img/leave-feedback-collect-info.png)


3. 在这个问题之后，插入一个“**逻辑**”步骤

![反馈的逻辑](https://rasa.com/docs/studio/img/leave-feedback-logic.png)

4. 将第一个条件设置为反馈为真，如果用户愿意留下反馈，此分支将被激活

![设置反馈为 True](https://rasa.com/docs/studio/img/leave-feedback-true.png)

5. 在 Else 分支（用户不想留下反馈的情况）之后添加一条消息

![反馈消息](https://rasa.com/docs/studio/img/leave-feedback-message.png)

6. 将新消息命名为 `utter_goodbye`，并添加文字内容 “谢谢，祝您度过美好的一天！”

![再见消息](https://rasa.com/docs/studio/img/money-transfer-utter-goodbye.png)

7. 在反馈为真的分支后上，我们添加“**收集信息**”步骤

![收集反馈信息](https://rasa.com/docs/studio/img/leave-feedback-collect-info2.png)


8. 输入描述：“请用户对机器人的表现进行评分”

![反馈评分](https://rasa.com/docs/studio/img/leave-feedback-collect-description.png)

9. 创建一个名为 `rating` 的新插槽，并使用 `Float` 作为类型，因为我们将接受从 1 到 5 的数字

![反馈流程中插槽](https://rasa.com/docs/studio/img/leave-feedback-collect-slot.png)

10. 创建一条名为 `utter_ask_rating` 的新消息，内容为 “从 1 到 5，你对与 Sbot 聊天的体验有何评价？”。您可以选择添加按钮，以便用户可以轻松地选择答案。

![评分](https://rasa.com/docs/studio/img/feedback-buttons.png)

11. 我们可以激活“**填写前询问**”，这样每次交易后都不会跳过这个问题

![填充前询问](https://rasa.com/docs/studio/img/feedback-ask-before-filling.png)


这样我们就将机器人简单构建好了。

### 2.5 训练和测试机器人

训练一个模型是构建机器人的基本步骤，这一过程至关重要，因为它使模型能够学习并适应特定的任务或领域。通过这样做，它提高了性能，使其能够对用户输入做出有意义和相关的响应。


Rasa Studio 用户可以在用户界面中对其机器人进行直接培训，然而技术用户可以在他们的本地机器上训练同一名机器人。

1. 勾选已创建的所有流程的“准备好进行训练”复选框

![准备训练](https://rasa.com/docs/studio/img/ready-for-training-checkbox.png)


2. 点击“**训练**”按钮开始训练

![训练](https://rasa.com/docs/studio/img/train.png)


3. 如果模型训练成功，我们将看到绿色勾号

![训练成功](https://rasa.com/docs/studio/img/training-completed.png)

4. 如果训练过程遇到问题或失败，将出现以下窗口：

![训练失败](https://rasa.com/docs/studio/img/Screenshot_2023-09-21_at_17.10.08.png)


5. 训练成功后，导航至“**试用您的机器人**”，开始与您的机器人互动


### 2.6 进阶: 添加自定义动作

自定义操作步骤可以执行您想要的任何代码，包括 API 调用、数据库查询等等。无论是开灯、在日历中添加活动、检查用户的银行余额，还是你能想象到的任何其他事情，自定义操作都提供了广泛的灵活性。


到目前为止，自定义操作只能在 Studio 之外实现。有关如何实现自定义操作的详细信息，请参阅 SDK 文档。要在流程中使用的任何自定义操作都应添加到 Domain(域) 的操作部分。


在本教程的这一小节中，我们将用自定义操作替换之前创建的金额验证逻辑。

1. 返回到 `money_transfer` 流程
2. 在关于你想转账多少钱的问题之后，在逻辑分支之前，添加一个“**自定义操作**”步骤

![添加自定义动作](https://rasa.com/docs/studio/img/money-transfer-add-custom-action.png)


3. 打开自定义操作的下拉列表，然后单击“**创建自定义操作**”

![自定义操作](https://rasa.com/docs/studio/img/money-transfer-create-custom-action.png)


4. 输入名称 `action_validate_sufficient_funds` 和描述：“目标：验证资金是否足以进行转账。要求：检查插槽金额是否大于 1000。如果满足，则插槽 `has_sufficient_funds` 值被设置为 `true`。如果不满足，则为插槽 `has_sufficient_funds` 被设置为 `false`。"

![自定义动作](https://rasa.com/docs/studio/img/money-transfer-new-custom-action.png)


5. 选择第一个条件，标记的金额为小于等于 1000，我们需要将其替换为自定义操作的验证动作。选择创建一个新插槽，而不是之前的 `amount`。

![新插槽](https://rasa.com/docs/studio/img/transfer-money-create-slot-2.png)


6. 输入插槽名称为 `has_sufficient_funds`，选择**布尔值**作为类型，然后单击“**保存**”。

![设置插槽类型](https://rasa.com/docs/studio/img/transfer-money-has-sufficient-funds.png)

7. 设置 `has_sufficient_funds` 为 `true` 时的条件

![设置条件](https://rasa.com/docs/studio/img/transfer-money-sufficient-funds-true.png)


8. 选择 IDE 开发编写代码，搭建 Action Server，需要确保的是 Rasa Pro >=3.7.0。


```Bash
rasa init --template calm
```


9. 实现验证资金的自定义操作，通常可以在 actions/actions.py 中复制/粘贴以下 Python 代码：

```Python
from typing import Any, Text, Dict, List
from rasa_sdk import Action, Tracker
from rasa_sdk.executor import CollectingDispatcher
from rasa_sdk.events import SlotSet

class ActionValidateSufficientFunds(Action):
   def name(self) -> Text:
      return "action_validate_sufficient_funds"

   def run(self, dispatcher: CollectingDispatcher,
         tracker: Tracker,
         domain: Dict[Text, Any]
   ) -> List[Dict[Text, Any]]:
      balance = 1000
      transfer_amount = tracker.get_slot("amount")
      has_sufficient_funds = transfer_amount <= balance
      return [SlotSet("has_sufficient_funds", has_sufficient_funds)]
```

10. 通过 Rasa CLI 运行 Action Server

```Bash
rasa run actions
```


11. 通过 Ngrok 这样的反向代理工具将本地计算机上运行的服务公开到公共互联网，并将其传递给 Studio 进行训练。这可以帮助我们进行快速原型设计和测试。当然也可以选择在所选的云环境中部署此操作服务器，并将公共 URL 传递给 Studio 以与机器人连接。


```Bash
ngrok http 5055
```

12. 在 Studio 中，将操作服务器 url+/webhook 粘贴到机器人管理-高级配置-操作服务器 URL:

![配置自定义动作服务的 URL](https://rasa.com/docs/studio/img/action-server-url.png)


13. 接下来就可以开始训练模型了

### 2.7 进阶: 编辑 YAML 文件

1. 通过 Rasa CLI 从 Studio 下载 YAML 文件

> 需要确保 Rasa Pro 3.7.0或更新版本。同时，确保根据 Rasa Studio 部署过程中使用的变量传递 Studio URL、领域名称和 client-id 的值
> 
{style="warning"}

```Bash
rasa init --template calm
```

**配置 Rasa Studio 的连接**

```Bash
rasa studio config
? Please enter an URL path to the Authentication Server <studio-url>/auth/
? Please enter an URL path to the Studio <studio-url>/api/graphql
? Please enter Realm Name <realm-name>
? Please enter client ID <client-id>
```

如果 Rasa Studio 部署的默认值已更改，请使用适当的参数。

**开发者登录**

```Bash
rasa studio login --username=<username> --password=<password>
```

**通过机器人名称下载机器人**

在本地的 Rasa 项目下运行如下命令:

```Bash
rasa studio download <assistant-name> -d domain
```

下载操作将更新本地机器人的 domain.yml 和 data/studio_flows.yml 文件内容。


2. 以下是不能在 Rasa Studio 中进行操作，但是可以在 YAML 中完成的：
    - 编辑 config.yml 文件
    - 视图跟踪（可观察性）
    - 添加数据管道的分析
    - 添加带有 AudioCodes 的 IVR 频道
    - 运行端到端测试
    - 使用机密管理
    - 使用 PII 管理
    - 添加标记
    - 使用对话的企业搜索
    - 添加对话跟踪
    - 在 flow.yml 和 domain.yml 中添加 NLU 触发器
    - 在 flow.yml 和 domain.yml 添加生成提示词
    - 在 flow.yml 添加流程守护

3. 使用 Rasa CLI 在本地训练 Rasa 模型

```Bash
rasa train
```

4. 使用 Rasa Inspector 测试您的机器人的自定义操作

```Bash
rasa inspect
```


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


