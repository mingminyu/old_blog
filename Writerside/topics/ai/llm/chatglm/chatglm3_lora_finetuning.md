# ChatGLM3 LORA微调

<show-structure depth="2"/>

本节我们简要介绍如何基于 transformers、peft 等框架，对 ChatGLM3-6B-chat 模型进行 Lora 微调。Lora 是一种高效微调方法，深入了解其原理可参见博客[深入浅出LORA](https://zhuanlan.zhihu.com/p/650197598)。

## 1. 环境配置

在完成基本环境配置和本地模型部署的情况下，你还需要安装一些第三方库，可以使用以下命令：

```Bash
pip install transformers==4.36.0.dev0
pip install peft==0.4.0.dev0
pip install datasets==2.10.1
pip install accelerate==0.20.3
```

## 2. 指令集构建

LLM 的微调一般指指令微调过程，所谓指令微调，是说我们使用的微调数据形如下所示，其中 `instruction` 是用户指令，告知模型其需要完成的任务；input 是用户输入，是完成用户指令所必须的输入内容；output 是模型应该给出的输出。

```json
{
    "instruction":"回答以下用户问题，仅输出答案。",
    "input":"1+1等于几?",
    "output":"2"
}
```

即我们的核心训练目标是让模型具有理解并遵循用户指令的能力，因此在指令集构建时，我们应针对我们的目标任务，针对性地构建任务指令集。例如，在本节我们使用由笔者合作开源的 [Chat-甄嬛](https://github.com/KMnO4-zx/huanhuan-chat) 项目作为示例，我们的目标是构建一个能够模拟甄嬛对话风格的个性化 LLM，因此我们构造的指令形如：

```json
{
    "instruction": "现在你要扮演皇帝身边的女人--甄嬛",
    "input":"你是谁？",
    "output":"家父是大理寺少卿甄远道。"
}
```

## 3. 数据格式化

Lora 训练的数据是需要经过格式化、编码之后再输入给模型进行训练，如果是熟悉 Pytorch 模型训练流程的同学会知道，我们一般需要将输入文本编码为 `input_ids`，将输出文本编码为 `labels`，编码之后的结果都是多维的向量。

我们首先定义一个预处理函数，这个函数用于对每一个样本，将输入和输出文本进行编码，并返回一个编码后的字典：

```Python
def process_func(example: Dict[str, str]):
    MAX_LENGTH = 512
    input_ids, labels = [], []
    instruction = tokenizer.encode(
        text="\n".join([
                "<|system|>", "现在你要扮演皇帝身边的女人--甄嬛", "<|user|>", 
                example["instruction"] + example["input"] + "<|assistant|>"
                 ]).strip() + "\n",
        add_special_tokens=True, truncation=True, max_length=MAX_LENGTH
        )

    response = tokenizer.encode(
        text=example["output"], 
        add_special_tokens=False, 
        truncation=True,
        max_length=MAX_LENGTH
        )

    input_ids = instruction + response + [tokenizer.eos_token_id]
    labels = (
        [tokenizer.pad_token_id] * len(instruction) + 
        response + [tokenizer.eos_token_id]
        )
    pad_len = MAX_LENGTH - len(input_ids)
    input_ids += [tokenizer.pad_token_id] * pad_len
    labels += [tokenizer.pad_token_id] * pad_len
    labels = [
        (l if l != tokenizer.pad_token_id else -100) for l in labels
        ]

    return {"input_ids": input_ids, "labels": labels}
```

经过格式化的数据，也就是送入模型的每一条数据都是一个字典，包含了 `input_ids`、`labels` 两个键值对，其中 `input_ids` 是输入文本的编码，`labels` 则是输出文本的编码，`decode` 之后应该是这样的：

```Bash
[gMASK]sop <|system|>
现在你要扮演皇帝身边的女人--甄嬛
<|user|>
这个温太医啊，也是古怪，谁不知太医不得皇命不能为皇族以外的人请脉诊病，他倒好，十天半月便往咱们府里跑。<|assistant|>
 你们俩话太多了，我该和温太医要一剂药，好好治治你们。
```

为什么会是这个数据表现结构呢？因为不同模型所对应的格式化输入都不一样，所以需要我们深度模型的训练源码来查看，按照原本模型指令微调的形式进行 Lora 微调效果应该是最好的，所以这里依然遵循原本模型的输入格式。这里放一下源码的链接，以供后续探索一下：
- [ChatGLM3/preprocess_utils.py](https://github.com/THUDM/ChatGLM3/blob/main/finetune_chatmodel_demo/preprocess_utils.py)
- [LLama-Factory/template.py](https://github.com/KMnO4-zx/LLaMA-Factory/blob/main/src/llmtuner/data/template.py)

## 4. 加载 tokenizer 和半精度模型

模型以半精度形式加载，如果你的显卡比较新的话，可以用 `torch.bfolat` 形式加载。对于自定义的模型一定要指定 `trust_remote_code` 参数为 `True`。

```Python
tokenizer = AutoTokenizer.from_pretrained(
    './model/chatglm3-6b', 
    use_fast=False, 
    trust_remote_code=True
    )

# 模型以半精度形式加载，
# 如果显卡比较新的话，可以用 torch.bfolat 形式加载
model = AutoModelForCausalLM.from_pretrained(
    './model/chatglm3-6b', 
    trust_remote_code=True, 
    torch_dtype=torch.half, 
    device_map="auto"
    )
```


## 5. 定义 LoraConfig

`LoraConfig` 这个类中可以设置很多参数，但实际上核心参数没多少，后续有兴趣的话再去研究下源码。
- `task_type`：模型类型
- `target_modules`：需要训练的模型层的名字，主要就是 `attention` 部分的层，不同的模型对应的层的名字不同，可以传入数组，也可以字符串，也可以正则表达式。
- `r`：Lora 的秩，具体可以看 Lora 原理
- `lora_alpha`：具体作用参见 Lora 原理
- `modules_to_save`: 表示除了拆成 lora 的模块，其他的模块可以完整的指定训练


Lora 的缩放是什么？当然不是 `r`（秩），这个缩放就是 `lora_alpha/r`，在这个 `LoraConfig` 中缩放就是 4 倍。这个缩放的本质并没有改变 `LoRA` 的参数量大小，本质在于将里面的参数数值做**广播乘法**，进行线性的缩放。

```Python
config = LoraConfig(
    task_type=TaskType.CAUSAL_LM, 
    target_modules=["query_key_value"],
    inference_mode=False,  # 训练模式
    r=8,  # Lora 秩
    lora_alpha=32,  # Lora alaph
    lora_dropout=0.1  # Dropout 比例
)
```

## 6. 自定义 TrainingArguments 参数

`TrainingArguments` 这个类的源码也介绍了每个参数的具体作用，当然大家可以来自行探索，这里就简单说几个常用的：
- `output_dir`: 模型的输出路径
- `per_device_train_batch_size`: 其实就是 `batch_size`
- `gradient_accumulation_steps`: 梯度累加，如果显存比较小，那可以把 `batch_size` 设置小一点，梯度累加增大一些。
- `logging_steps`: 多少步输出一次 log
- `num_train_epochs`: 其实就是 `epochs`
- `gradient_checkpointing`: 梯度检查，这个一旦开启后模型就必须执行 `model.enable_input_require_grads()`，这个原理大家可以自行探索，这里就不细说了。

```Python
# DataCollator GLM 源仓库重新封装了自己的 data_collator
data_collator = DataCollatorForSeq2Seq(
    tokenizer,
    model=model,
    label_pad_token_id=-100,
    pad_to_multiple_of=None,
    padding=False
)

args = TrainingArguments(
    output_dir="./output/ChatGLM",
    per_device_train_batch_size=4,
    gradient_accumulation_steps=2,
    logging_steps=10,
    num_train_epochs=3,
    gradient_checkpointing=True,
    save_steps=100,
    learning_rate=1e-4,
)
```

## 7. 使用 Trainer 训练

把 model 放进去，把上面设置的参数放进去，数据集放进去，开始训练！

```Python
trainer = Trainer(
    model=model,
    args=args,
    train_dataset=tokenized_id,
    data_collator=data_collator,
)
trainer.train()
```

## 8. 模型推理

直接使用常规的方式进行推理：

```Python
model.eval()
model = model.cuda()
input_ = tokenizer(
    "<|system|>\n现在你要扮演皇帝身边的女人--甄嬛\n<|user|>\n {}\n{}".format("你是谁？", "").strip() + "<|assistant|>\n", 
    return_tensors="pt"
    ).to(model.device)
tokenizer.decode(
    model.generate(**input_, max_length=128, do_sample=True)[0], 
    skip_special_tokens=True
    )
```

通过 PEFT 所微调的模型，都可以使用下面的方法进行重新加载，并推理:
- 加载源 `model` 与 `tokenizer`
- 使用 `PeftModel` 合并源 `model` 与 `PEFT` 微调后的参数


```Python
from peft import PeftModel


model = AutoModelForCausalLM.from_pretrained(
    "./model/chatglm3-6b", 
    trust_remote_code=True, 
    low_cpu_mem_usage=True
    )
tokenizer = AutoTokenizer.from_pretrained(
    "./model/chatglm3-6b", 
    use_fast=False, 
    trust_remote_code=True
    )

# 将训练所得的LoRa权重加载起来
p_model = PeftModel.from_pretrained(
    model, 
    model_id="./output/ChatGLM/checkpoint-1000/"
    )  

ipt = tokenizer(
    "<|system|>\n现在你要扮演皇帝身边的女人--甄嬛\n<|user|>\n {}\n{}".format("你是谁？", "").strip() + "<|assistant|>\n", 
    return_tensors="pt").to(model.device)
tokenizer.decode(
    p_model.generate(**ipt, max_length=128, do_sample=True)[0], 
    skip_special_tokens=True
    )
```


## 9. 完整项目

这里对项目代码进行了重构，详见 [ChatGLM3-LoRaTuning]()。


<seealso>
<category ref="ref_docs">
    <a href="https://github.com/sanbuphy/self-llm/tree/master/ChatGLM">ChatGLM3+LORA微调</a>
    <a href="https://zhuanlan.zhihu.com/p/650197598">深入浅出LORA微调</a>
    <a href="https://github.com/zyds/transformers-code">Transformers 代码讲解</a>
</category>
<category ref="ref_github">
    <a href="https://github.com/yongzhuo/ChatGLM3-SFT">ChatGLM3-SFT</a>
</category>
</seealso>