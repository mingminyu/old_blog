# ChatGLM FAQ

<show-structure depth="2"/>

## 1. 推理问题

报错：Tensor 包含 inf、nan、负数
: “**RuntimeError**: probability tensor contains either `inf`, `nan` or element ＜ 0”

> 在 `model.generate` 或者 `model.chat` 方法中去掉 `do_sample=True` 参数设定。

## 2. 训练问题

异常: 使用 Trainer 类训练时要求填入 WANDB token
: 在 Kaggle 上进行微调训练，执行到 `trainer.train()` 步骤时，要求必须输入 WANDB 的 token

> 在 `TrainingArguments` 的参数中设置 `report_to="none"`，或者设置环境变量 `os.environ["WANDB_DISABLED"] = "true"` 就可以。



<seealso>
<category ref="ref_docs">
    <a href="https://blog.csdn.net/weixin_44563460/article/details/133800257">ChatGLM3 推理报错: Tensor 包含 inf、nan、负数</a>
    <a href="https://blog.csdn.net/PolarisRisingWar/article/details/132091055">ChatGLM3 微调: 强制要求填入 WANDB token</a>
</category>
<category ref="ref_github"></category>
<category ref="ref_issues"></category>
<category ref="ref_hf"></category>
<category ref="ref_ms"></category>
</seealso>


