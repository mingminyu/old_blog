# ChatGLM4 发布预览

<show-structure depth="2"/>

> 文章为智谱 AI 在公众号上发布的 [GLM4 文章](https://mp.weixin.qq.com/s/QfVM52oecfjLVDz-EXRyHA?wxwork_userid=JiaWei")，这里了解下更新情况。

新一代基座大模型 GLM4，整体性能相比 GLM3 全面提升 **60%**，逼近GPT4；支持更长上下文；更强的多模态；支持更快推理速度，更多并发，大大降低推理成本；同时 GLM4 增强了智能体能力。

## 1. 能力评估

**基础能力（英文）**：GLM4 在 MMLU、GSM8K、MATH、BBH、HellaSwag、HumanEval 等数据集上，分别达到 GPT4 94%、95%、91%、99%、90%、100% 的水平。

**指令跟随能力**：GLM4 在 IFEval 的 prompt 级别上中、英分别达到 GPT4的 88%、85% 的水平，在 Instruction 级别上中、英分别达到 GPT4 的 90%、89% 的水平。

**对齐能力**：GLM4 在中文对齐能力上整体超过 GPT4。

**长文本能力**：我们在 LongBench（128K）测试集上对多个模型进行评测，GLM4 性能超过 Claude 2.1；在「大海捞针」（128K）实验中，GLM4 的测试结果为 128K 以内全绿，做到 100% 精准召回。

**多模态-文生图**：CogView3 在文生图多个评测指标上，相比DALLE3 约在 **91.4% ~99.3%** 的水平之间。

## 2. ALL Tools

GLM-4 实现自主根据用户意图，自动理解、规划复杂指令，自由调用网页浏览器、Code Interpreter 代码解释器和多模态文生图大模型，以完成复杂任务。

简单来讲，即只需一个指令，GLM4 会自动分析指令，结合上下文选择决定调用合适的工具。


**All Tools——文生图**：GLM4 能够结合上下文进行 AI 绘画创作（CogView3），大模型能够遵循人的指令来不断修改生成图片的结果。

**All Tools——代码解释器**：GLM4 能够通过*自动调用 Python 解释器*，进行复杂计算（例如复杂方程、微积分等），在 GSM8K、MATH、Math23K 等多个评测集上都取得了接近或同等 GPT4 All Tools 的水平。同样 GLM4 也可以完成**文件处理、数据分析、图表绘制等复杂任务，支持处理 Excel、PDF、PPT 等格式文件**。

> 自动调用 Python 解释器，GLM3 已经就有了，主要更新点在于它能做更复杂的计算。体验了下 GLM4，目前没有看到图标绘制的功能，并且生成可交互的图表完全不可行。

**All Tools——Function Call**：GLM4 能够根据用户提供的Function 描述，自动选择所需 Function 并生成参数，以及根据 Function 的返回值生成回复；同时也支持一次输入进行多次 Function 调用，支持包含中文及特殊符号的 Function 名字。这一方面GLM-4 All Tools 与 GPT4 Turbo 相当。

> 这个在 GLM3 中也有了，且 QWEN 在这一块功能性做得更好，后续可以单独研究下。


**All Tools——多工具自动调用**：除了以上单项工具自动调用外，GLM4 同样能够实现多工具自动调用，例如结合**网页浏览、CogView3、代码解释器等**的调用方式。


## 3. GLMs & MaaS API

GLM4 的全线能力提升使得我们有机会探索真正意义上的 GLMs。

> 照搬 ChatGPT Store 模式，构建 GPTs 商店，这确实也是一个趋势。值得思考的是，如何坐上 GPTs 这艘快船。

## 4. 总结

总体来说，ChatGLM4 的亮点并不多，更多是推理性能上的提升以及功能的丰富，新增的功能基本上在 ChatGPT4 都早已经实现。


<seealso>
<category ref="ref_docs">
    <a href="https://mp.weixin.qq.com/s/QfVM52oecfjLVDz-EXRyHA?wxwork_userid=JiaWei">新一代基座模型GLM4</a>
</category>
</seealso>
