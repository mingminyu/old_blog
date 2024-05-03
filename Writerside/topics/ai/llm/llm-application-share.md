# 大语言模型的应用实践

<show-structure depth="3"/>

近一年来 AIGC(Artificial Intelligence Generative Content) 发展可谓是非常迅速，到早期的 ChatGPT 横空出世，到后面的 Stable Diffusion、Midjourney，再到国产大模型 ChatGLM、Qwen、Yi 等百花齐放，甚至到最近非常强大的 Sora 问世，都在表明生成式 AI 技术已经处于一个相对成熟的阶段。

本篇文章我们主要介绍下大模型的基础使用，以及如何将其与业务场景进行结合，来帮助我们解决实际的业务问题。

## 1. 基础介绍 {collapsible="true" default-state="expanded"}

### 1.1 大语言模型

大模型(Large Model) 通常是指机器学习或者人工智能领域中参数量达到十亿甚至百亿级别的模型，而我们经常说的大模型更多特指的是大语言模型(Large Language model，LLM)，它只是大模型中对于文本领域的一类模型。

> 我们接下来所提及的大模型也是大语言模型。


### 1.2 大模型的选型

目前市面上大模型有很多种，很多大厂都是有开源和闭源的大模型版本，通常闭源的模型大多是通过网页形式访问或者 API 接口的方式进行调用（API 接口会按照 Token 进行收费）。考虑到在实际业务中应用时会涉及到敏感数据的问题，且在有一定的 GPU 资源的基础下，我们更多会考虑私有化部署大模型的方案，通过自定义接口服务，从而应用的高扩展性以及易用性。


目前主流的开源中文大模型有:
- **Qwen**: 阿里开源的通义千问，文档比较详细，微调脚本比较健全
- **ChatGLM**: 清华开源的 GLM 系列，没有比较详细的使用文档，但提供了微调脚本
- **Yi**: 李开复团队开源
- **BaiChuan**: 王小川团队开源
- InternLM: 商汤开源
- ...


我们可以在 [CEval](https://cevalbenchmark.com/static/leaderboard_zh.html)、[FlagEval](https://flageval.baai.ac.cn/#/trending)以及[Open Compass](https://rank.opencompass.org.cn/leaderboard-llm-v2) 查看大模型的排名情况，在选择哪种大模型进行应用时，可以从以下方面进行考虑:
- 如果应用场景比较偏英文的话，建议选择像 Mistral、Open-Chat 这种纯英文大模型
- 当前服务器 GPU 资源支持什么样参数级别的大模型
- 开源模型是否有有完备的部署、微调文档和脚本
- 同参数量级别的模型在排行榜上的排名


> 目前各家的开源大模型也在逐渐走向闭源和商业化，后续一些性能强大的大模型可能不会被开源出来。
> 
{style="warning"}


这里我们选用的是 ChatGLM3 开源大模型，通过微调训练出质检大模型以及信息抽取大模型，并在业务上进行应用。

### 1.3 大模型基础使用

大模型的使用基本上都需要安装 Hugging Face 的 `transformers` 库，有些大模型提供了 GPTQ、AWQ 或者 GGUF 的版本，可能需要额外安装一些依赖库，大家可以自行去官方文档或者官方 Github 库查找对应的方法。


<tabs>
<tab title="pip">
<code-block lang="bash">
<![CDATA[
pip install transformers
]]>
</code-block>
</tab>
<tab title="conda">
<code-block lang="bash">
<![CDATA[
conda install conda-forge::transformers
]]>
</code-block>
</tab>
</tabs>


接下来我们试用下 ChatGLM3 大模型:

<tabs>
<tab title="模型调用">
<code-block lang="python">
<![CDATA[
from transformers import AutoTokenizer, AutoModel


tokenizer = AutoTokenizer.from_pretrained("THUDM/chatglm3-6b", trust_remote_code=True)
model = AutoModel.from_pretrained("THUDM/chatglm3-6b", trust_remote_code=True).half().cuda()
model = model.eval()
response, history = model.chat(tokenizer, "你好", history=[])
print(response)
response, history = model.chat(tokenizer, "晚上睡不着应该怎么办", history=history)
print(response)
]]>
</code-block>
</tab>
<tab title="输出">
<code-block lang="python">
<![CDATA[
你好👋!我是人工智能助手 ChatGLM-6B,很高兴见到你,欢迎问我任何问题。

晚上睡不着可能会让你感到焦虑或不舒服,但以下是一些可以帮助你入睡的方法:
1. 制定规律的睡眠时间表:保持规律的睡眠时间表可以帮助你建立健康的睡眠习惯,使你更容易入睡。尽量在每天的相同时间上床,并在同一时间起床。
2. 创造一个舒适的睡眠环境:确保睡眠环境舒适,安静,黑暗且温度适宜。可以使用舒适的床上用品,并保持房间通风。
3. 放松身心:在睡前做些放松的活动,例如泡个热水澡,听些轻柔的音乐,阅读一些有趣的书籍等,有助于缓解紧张和焦虑,使你更容易入睡。
4. 避免饮用含有咖啡因的饮料:咖啡因是一种刺激性物质,会影响你的睡眠质量。尽量避免在睡前饮用含有咖啡因的饮料,例如咖啡,茶和可乐。
5. 避免在床上做与睡眠无关的事情:在床上做些与睡眠无关的事情,例如看电影,玩游戏或工作等,可能会干扰你的睡眠。
6. 尝试呼吸技巧:深呼吸是一种放松技巧,可以帮助你缓解紧张和焦虑,使你更容易入睡。试着慢慢吸气,保持几秒钟,然后缓慢呼气。

如果这些方法无法帮助你入睡,你可以考虑咨询医生或睡眠专家,寻求进一步的建议。
]]>
</code-block>
</tab>
</tabs>

那我们看下基座的模型在贷后质检场景下识别是否准确:

<tabs>
<tab title="Zero-Shot">
<code-block lang="python">
<![CDATA[
hist = [
    {"role": "system", "content": "你是一个优秀的质检员，请判断给定的文本内容尽可能准确地判断出是否命中“循环额度”，“命中”输出 1，“未命中”输出 0。"}
]
prompt = "你把钱还进来，后面还是可以借出去的"
response, history = model.chat(tokenizer, prompt, history=[], temperature=0.01)
print(prompt, '=>', response)
]]>
</code-block>
</tab>
<tab title="输出">
<code-block lang="python">
<![CDATA[
作为人工智能助手，我没有实际的资金操作能力。但我可以为您解释一下这个概念。

当您将钱借给别人时，您实际上是在进行贷款行为。对方在借款期间需要按照约定的条件还款。而当借款人按时还款时，钱款会回到您这里。此时，您可以选择再次借出去这笔钱，形成一个循环的过程。

需要注意的是，在贷款过程中要确保对方有还款能力，并遵守相关的法律法规。同时，要确保资金安全，避免出现风险。
]]>
</code-block>
</tab>
</tabs>

<tabs>
<tab title="Few-Shot">
<code-block lang="python">
<![CDATA[
hist = [
    {"role": "system", "content": "你是一个优秀的质检员，请判断给定的文本内容尽可能准确地判断出是否命中“循环额度”，“命中”输出 1，“未命中”输出 0。例如“你可以把钱先还进来，过两天可以借出去的”输出 1，“你是想还进来再借出去是么？”输出为0"}
]
prompt = "还进来再借出去，你觉得这种事情可能么"
response, history = model.chat(tokenizer, prompt, history=[], temperature=0.01)
print(prompt, '=>', response)
]]>
</code-block>
</tab>
<tab title="输出">
<code-block lang="python">
<![CDATA[
从道德和法律的角度来看，这种事情不可能存在。首先，从道德层面来说，借东西的时候应该诚实、真实地告知对方你的需求和目的。如果你进来再借出去，这显然不符合诚实守信的原则，也违背了借东西的初衷。

其次，从法律角度来看，这种行为可能涉嫌侵犯他人的权益。如果你在未经他人同意的情况下，将借来的东西再借出去，这可能构成不当得利或者侵权行为。在我国法律体系中，这样的行为是不被允许的。

所以，这种事情是不可能存在的。我们应该在借东西的时候诚实守信，尊重他人的权益，遵守法律法规。
]]>
</code-block>
</tab>
</tabs>


可以看到，无论是 Zero-Shot 还是 Few-Shot 的方式，基座模型的效果都不是很好，给出的输出有点牛头不对马嘴，根本无法达到业务应用。


## 2. 模型微调 {collapsible="true" default-state="collapsed"}

上面示例中，基座模型无法正确理解业务中的知识和逻辑，那么我们就需要通过**注入知识的方式(即模型微调)** 来让大模型学习到这些业务知识，从而能帮助我们准确识别出该文本在业务场景下是否违规。


目前针对大模型而言，其微调(Supervised Fine-Tuning, SFT) 的方式主要有 LoRA、Q-LoRA 以及全参数微调（对于 ChatGLM 系列的大模型，有着独有的 P-Tuning 微调）。接下来介绍下这些微调方法，全参数的微调因为对于 GPU 资源要求过高，这里也不作介绍。


> 后面的模型微调都是特指单 GPU 的微调，多 GPU 的微调方式不作介绍。
> 
{style="note"}




### 2.1 LoRA 微调



### 2.2 Q-LoRA 微调



### 2.3 P-Tuning 微调


### 2.4 微调样本

ChatGLM3 官方仓库中给我们明确解释了模型微调时所需的样本示例，在这个基础上我们扩展到了上下文:

<tabs>
<tab title="单句">
<code-block lang="json">
<![CDATA[
{
    "conversations": [
      {
        "role": "system",
        "content": "你是一个优秀的质检员，请判断给定的文本内容尽可能准确地判断出是否命中“循环额度”，“命中”输出 1，“未命中”输出 0。"
      },
      {
        "role": "user",
        "content": "Agent: 但是是这样的，先生，就是我不知道有没有工作人员跟你讲过，我们分期乐也是说就是额度就是额度循环使用的一个公司，你。"
      },
      {
        "role": "assistant",
        "content": "1"
      }
    ]
}
]]>
</code-block>
</tab>
<tab title="上下文">
<code-block lang="json">
<![CDATA[
{
     "conversations": [
      {
        "role": "system",
        "content": "你是一个优秀的信息归纳员，请根据给定的对话内容尽可能准确地判断出“用户本人的工作状态”，“有工作”输出 1，“没有工作”输出 0，“不确定”输出 -1。"
      },
      {
        "role": "user",
        "content": "Agent: 你去你去找工作，你说我没有学历能找工作吗？要求的学历是本科学历，你你能去吗？\nUser: 没有钱，你催收催什么催呀？催了，你把我逼走，我能还了，你逼我能还了吗？"
      },
      {
        "role": "assistant",
        "content": "0"
      }
    ]
}
]]>
</code-block>
</tab>
</tabs>


标注样本的收集建议大家可以使用 <format color="red" style="bold">Label Studio</format> 工具，支持标注数据的导入和导出，并可以根据不同应用场景制定合适的标注模板，能大幅提升我们收集标注样本的效率。


## 3. 规则引擎 {collapsible="true" default-state="collapsed"}

> 大模型对于文本的推断能力很强，<format color="red" style="bold">但还不够强</format>。


贷后的文本信息几乎上是集中在录音转写的文本中，且数据格式为用户与催员轮次对话形式出现:

```Python
Agent: 你现在有工作么
User: 没有，大半年了
```

针对于贷后录音场景，设计了一套通用性的规则匹配框架，作用是先对文本进行匹配，匹配通过后再使用大模型进行预测。

<tabs>
<tab title="循环额度">
<code-block lang="yaml">
<![CDATA[
version: "0.1"
scenes:
- intent: loop_loan_amount
  description: 循环额度
  instruction: "你是一个优秀的质检员，请判断给定的文本内容尽可能准确地判断出是否命中“循环额度”，“命中”输出 1，“未命中”输出 0。"
  scene_id: CQC001
  risk_level: 高
  enable: true
  role: agent
  formula: "({r1} | {r2} | {r3}) & (!{r6})"
  scene_rules:
    - rule_id: r1
      positive: true
      regexes:
        - reg: (钱|款项|额度|备用金|资金|款项).{0,30}[搞提刷弄倒套拿取借整领盗]出[来去钱].{0,10}(反复|重复|重新|循环|二次){1,2}(使用|用)
        - reg: (给|让)[他她它你您].{0,10}(反复|重复|重新|循环|二次){1,2}(使用|用)
        - reg: (?<!(不会|不能))(给|让)[他她它你您].{0,10}[搞提刷弄倒套拿取借整领盗]出[来去钱]
        - reg: (钱|款项|额度|备用金|资金).{0,20}(?<!不)(能|可以|让你|给你|再|在|把它).{0,8}[取借拿套刷提]出
        - reg: (钱|款项|额度|备用金|资金)[取借拿套刷提]出[来去]

    - rule_id: r2
      positive: true
      operation: and
      regexes:
        - reg: ^(?!.*(不要再去想|为什么不去|为什么没有|有没有想|有没有考虑|没有考虑过|每个月按时|怎么不想着)).*[搞提刷弄倒套拿取借整领盗]出[来去钱]
        - reg: ^(?!.*(刷个身份证|刷了个身份证|没有必要|没必要)).*[搞提刷弄倒套拿取借整领盗]出[来去钱]
        - reg: ^(?!.*(没有可能|没有办法|怎么可能|什么时候|没说让您|没有叫您|没有叫你|没说你能|点头摇头)).*[搞提刷弄倒套拿取借整领盗]出[来去钱]

    - rule_id: r3
      positive: true
      operation: and
      regexes:
        - reg: ^(?!.*(不可能|怎么会|没可能|没办法|不允许|不存在|不可以|一直在|属于是)).{0,15}(反复|重复|重新|循环)(用|使用|正常使用)
        - reg: ^(?!.*(没有可能|没有办法|怎么可能|一直是|一直都是)).{0,15}(反复|重复|重新|循环)(用|使用|正常使用)
        - reg: ^(?!.*(不能|没有)).{0,15}(反复|重复|重新|循环)(用|使用|正常使用)(?<!(不了|不行))

    - rule_id: r6
      positive: false
      regexes:
        - reg: (申请|尝试){1,2}.{0,4}[借拿取套刷提]出[去来]
        - reg: (申请|尝试){1,2}(使用|继续使用)

  global_neg_formula: "{r1}"
  global_neg_rules:
    - rule_id: r1
      positive: false
      regexes:
        - reg: 批为准|申请审核|不良资产审核
        - reg: (审批|申请)出来.{0,8}(额度|钱|的话)
        - reg: 以.{0,10}(app|一批批|a佩佩|系统|abb|bb|b|a臂|并b|分期乐|一笔省|b省|那个|并b|也是批|呀岩壁).{0,4}(审核|显示|标准){0,2}为[准主]
]]>
</code-block>
</tab>
<tab title="用户工作状态">
<code-block lang="yaml">
<![CDATA[
version: "0.1"
scenes:
- intent: work_status
  description: 工作信息-工作状态
  instruction: "你是一个优秀的信息归纳员，请根据给定的对话内容尽可能准确地判断出”用户本人的工作状态“，”有工作“输出 1，”没有工作“输出 0，“不确定”输出 -1。"
  scene_id: CIM001
  risk_level: 低
  enable: true
  role: user
  formula: "(!{r2}) & ({r1} | {r3})"
  exclude_result: -1
  scene_rules:
    - rule_id: r1
      positive: true
      role: user
      regexes:
        - reg: (我去|我)找工作
          conf: 0.6
        - reg: (没有|还没|没)(工作|收入|上班)
        - reg: (没有|没了|没)稳定的工作
        - reg: (没|没有)办法上班

    - rule_id: r2
      positive: false
      role: user
      regexes:
        - reg: (我|他|她|他们|她们)(爸|妈|爸妈|父母|母亲|父亲|老婆|亲戚朋友|老公|爱人){1,2}.{0,5}(没有|还没|没)(工作|收入)
        - reg: (我|他|她|他们|她们)(爸|妈|爸妈|父母|母亲|父亲|老婆|亲戚朋友|老公|爱人){1,2}.{0,5}[也要都]{0,2}(失业|裁员)了

    - rule_id: r3
      positive: true
      role: user
      regexes:
        - reg: 我.{0,3}(找|找了|找到|找着)工作了
        - reg: 月底(发|发了)工资
        - reg: 我不是(没|没有)工作
        
context_formula: "{r1}"
context_rules:
    - rule_id: r1
      positive: true
      role: agent->user
      regexes:
        question:
          - reg: (你|您)(现在|目前).{0,6}(没|没有|没在)(工作|上班)
          - reg: 你(现在|目前).{0,6}(有没|有没有|有在|没在)(工作|上班)
        answer:
          - reg: 在上的|有工作|在工作|家里蹲|有收入|无收入|没收入|没有收入|有工资
          - reg: 嗯|是|对|是的|是啊|对的|对啊|没有|没在|不在|有啊|有的|有在
  ]]>
</code-block>
</tab>
</tabs>


`scene_rules` 则是用于单句文本的匹配，`context_rules` 用于上下文的匹配，而 `formula` 和 `context_formula` 则是将多个规则进行组合。值得一提的是，在规则匹配器中增加了全局负向匹配，因为一些质检场景下存在催员如果说了一些话术，那么本身违规文本就不算是违规了，例如:

```Bash
你可以先还进来，后面借出去也可以啊，具体还是以app为准。
```


## 4. 大模型开发流程

接下来介绍一下大模型开发步骤:
1. 梳理场景关键词，编写匹配规则，所编写的正则最好能达到 50% 的准确率
2. 制定标注模板，将正则过滤后的文本导入 Label Studio 中进行标注
3. 拉取标注后的数据，处理成模型微调的数据集
4. 对大模型进行微调，使用微调后的模型验证准确率
5. 根据对标注数据中预测与真实标签不匹配的部分，查看文本并优化当前规则，如果规则不易于新增，则可以通过上采样这些样本，给足样本让模型重新学习


## 5. 应用落地


### 5.1 质检大模型 {collapsible="true" default-state="collapsed"}



### 5.2 信息抽取大模型 {collapsible="true" default-state="collapsed"}



## 6. 模型部署








