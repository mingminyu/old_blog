# 大模型综述

<show-structure depth="2"/>

## 1. 大模型打榜排名

目前中文大模型的打榜排名主要是在 [CEval](https://cevalbenchmark.com/static/leaderboard_zh.html) 以及 [FlagEval](https://flageval.baai.ac.cn/#/trending) 上可以查看，但是对于同一个模型给出的排名结果，实际这两者差异也比较大。

这里以 ChatGLM 模型为例，截至 2023/11 数据，在 FlagEval 上排名第三，但是 CEval 上排名在 11，但是同一模型的不同版本效果提升是可以参考的，比如 CEval 上就可以看到 ChatGLM3 比 ChatGLM2 有着 **50%** 的性能提升。

## 2. 信息抽取能力比较

采用 ZERO-SHOT 策略，测试在同一样本上不同开源 LLM 在信息抽取上的准确性。

通过使用 QWen-14B-Chat-Int8、QWen-7B-Chat 以及 ChatGLM3-6B 这 2 个不同大模型，对同一段对话内容进行信息的提取，最终测试效果为: **ChatGLM3-6B >> QWen-14B-Chat-Int8 >> QWen-7B-Chat**，并且 QWen-7B-Chat 的推理速率是远远低于 ChatGLM3-6B 的。



<seealso>
<category ref="ref_docs">
    <a href="https://cevalbenchmark.com/static/leaderboard_zh.html">CEval上大模型排名</a>
    <a href="https://flageval.baai.ac.cn/#/trending">FlagEval上大模型排名</a>
</category>
</seealso>
