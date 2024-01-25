# ChatGLM3 ChatModel 官方微调教程

<show-structure depth="2"/>

本篇文档主要参考 ChatGLM3 官方仓库中的示例，包括全量微调和 P-Tuning v2。格式上，提供多轮对话微调样例和输入输出格式微调样例。

> 官方仓库中只有 Base 模型给出了 LORA 微调方式，Chat 模型并没有给出，待后续研究 ChatGLM3 LORA 微调。
{style="note"}


## 1. 依赖环境

运行示例需要 python>=3.10，除基础的 `torch` 依赖外，示例代码运行还需要依赖，在官方仓库中找到 finetune_chatmodel_demo/requirements.txt 下载到项目中，执行如下代码：

```Bash
pip install -r requirements.txt
```

## 2. 多轮对话下的微调

多轮对话微调示例采用 ChatGLM3 对话格式约定，对不同角色添加不同 `loss_mask` 从而在一遍计算中为多轮回复计算 loss。

### 2.1 数据格式和预处理

多轮对话分为纯对话数据以及带有工具调用的对话数据，两者的微调方式也是不同的。

#### 对话微调数据集

对于数据文件，样例采用如下格式。如果您仅希望微调模型的对话能力，而非工具能力，您应该按照以下格式整理数据。

```JSON
[
  {
    "conversations": [
      {
        "role": "system",
        "content": "<system prompt text>"
      },
      {
        "role": "user",
        "content": "<user prompt text>"
      },
      {
        "role": "assistant",
        "content": "<assistant response text>"
      }, 
      {
        "role": "user",
        "content": "<user prompt text>"
      },
      {
        "role": "assistant",
        "content": "<assistant response text>"
      }
    ]
  }
]
```

> 请注意，这种方法在微调的 `step` 较多的情况下会影响到模型的工具调用功能。
> {style="warning"}

#### 工具调用微调数据集

如果您希望微调模型的对话和工具能力，您应该按照以下格式整理数据。

```JSON
[
  {
    "tools": [
      {
        "func_call": "get_weather",
        "parameters": {"location":  "shanghai"}
      }
    ],
    "conversations": [
      {
        "role": "system",
        "content": "<system prompt text>"
      },
      {
        "role": "user",
        "content": "<user prompt text>"
      },
      {
        "role": "assistant",
        "content": "<assistant response text>"
      }, 
      {
        "role": "user",
        "content": "<user prompt text>"
      },
      {
        "role": "assistant",
        "content": "<assistant response text>"
      }
    ]
  }
]
```

上述数据集说明
: - 关于工具描述的 system prompt 无需手动插入，预处理时会将 `tools` 字段使用 `json.dumps(..., ensure_ascii=False)` 格式化后插入为首条 system prompt。
: - 每种角色可以附带一个 `bool` 类型的 `loss` 字段，表示该字段所预测的内容是否参与 **loss** 计算。若没有该字段，样例实现中默认对 system, user 不计算 loss，其余角色则计算 loss。
: - `tool` 并不是 ChatGLM3 中的原生角色，这里的 `tool` 在预处理阶段将被自动转化为一个具有工具调用 `metadata` 的 **assistant** 角色（默认计算 loss）和一个表示工具返回值的 **observation** 角色（不计算 loss）。
: - system 角色为可选角色，但若存在 system 角色，其必须出现在 user 角色之前，且一个完整的对话数据（无论单轮或者多轮对话）只能出现一次 system 角色。


作为示例，我们使用 ToolAlpaca 数据集来进行微调。首先克隆 [ToolAlpaca 数据集](https://github.com/tangqiaoyu/ToolAlpaca)，并使用如下命令调整数据集：

```Bash
./scripts/format_tool_alpaca.py --path "ToolAlpaca/data/train_data.json"
```

将数据集处理成上述格式后，在这里我们有意将工具处理成了了 `list[str]` 这样的自然语言形式，以观察模型在微调前后对工具定义的理解能力。

### 2.2 微调模型

以下脚本提供了微调模型的参考方式:

<tabs>
    <tab title="全量微调">
        <code-block lang="bash">
        ./scripts/finetune_ds_multiturn.sh 
        </code-block>
    </tab>
    <tab title="P-Tuning v2微调">
        <code-block lang="bash">
        ./scripts/finetune_pt_multiturn.sh 
        </code-block>
    </tab>
</tabs>

### 2.3 部署

通过调用微调后模型的 checkpoint 来部署模型，当然全量微调和 P-Tuning 微调的使用方式也是不一样的：

<tabs>
    <tab title="全量微调部署">
        <code-block lang="bash">
        cd ../composite_demo
        MODEL_PATH="path to finetuned model checkpoint" TOKENIZER_PATH="THUDM/chatglm3-6b" streamlit run main.py
        </code-block>
    </tab>
    <tab title="P-Tuning v2微调部署">
        <code-block lang="bash">
        cd ../composite_demo
        MODEL_PATH="THUDM/chatglm3-6b" PT_PATH="path to p-tuning checkpoint" streamlit run main.py
        </code-block>
    </tab>
</tabs>

## 3. 纯输入输出微调

### 3.1 数据格式

对于单纯的输入-输出格式，样例采用如下输入格式，预处理时，不会拼接任何角色标识符。

```json
[
  {
    "prompt": "<prompt text>",
    "response": "<response text>"
  }
]
```

作为示例，我们使用 AdvertiseGen 数据集来进行微调。从 [Google Drive](https://drive.google.com/file/d/13_vf0xRTQsyneRKdD1bZIr93vBGOczrk/view?usp=sharing) 或者 [Tsinghua Cloud](https://cloud.tsinghua.edu.cn/f/b3f119a008264b1cabd1/?dl=1) 下载处理好的 AdvertiseGen 数据集，将解压后的 AdvertiseGen 目录放到本目录下。


通过如下命令来下载和将数据集处理成上述格式：

```Bash
./scripts/format_advertise_gen.py --path "AdvertiseGen/train.json"
```

### 3.2 微调模型

以下脚本提供了微调模型的参考方式:

<tabs>
    <tab title="全量微调">
        <code-block lang="bash">
        ./scripts/finetune_ds.sh 
        </code-block>
    </tab>
    <tab title="P-Tuning v2微调">
        <code-block lang="bash">
        ./scripts/finetune_p.sh 
        </code-block>
    </tab>
</tabs>


### 3.3 推理验证

对于输入输出格式的微调，可使用 inference.py 进行基本的推理验证，当然全量微调和 P-Tuning 微调的使用方式也是不一样的：

<tabs>
    <tab title="全量微调推理">
        <code-block lang="bash">
        python inference.py \
            --pt-checkpoint "path to p-tuning checkpoint" \
            --model THUDM/chatglm3-6b 
        </code-block>
    </tab>
    <tab title="P-Tuning v2微调推理">
        <code-block lang="bash">
        python inference.py \
            --tokenizer THUDM/chatglm3-6b \
            --model "path to finetuned model checkpoint" 
        </code-block>
    </tab>
</tabs>


如果是 P-Tuning 的微调方式，实际应用中发现使用 `model.generate` 可以得到预期结果，但是如果直接使用 `model.chat` 的方式，所得到的效果会比较差，原因在于 `model.chat` 会以多轮对话形式将输入进行 token 化之后再进行推理，所得到的数据格式与微调数据本身就存在较大差异。而 `model.generate` 则是直接对输入的 `input_ids` 进行的操作，所以能和微调数据保持一致。


### 3.4 提示

1. 微调代码在开始训练前，会先打印首条训练数据的预处理信息，示例如下。每行依次表示一个 `detokenized string`, `token_id` 和 `target_id`。可在日志中查看这部分的 `loss_mask` 是否符合预期。若不符合，可能需要调整代码或数据。
    ```Bash
    Sanity Check >>>>>>>>>>>>>
             '[gMASK]':  64790 ->   -100
                 'sop':  64792 ->   -100
          '<|system|>':  64794 ->   -100
                    '':  30910 ->   -100
                  '\n':     13 ->   -100
              'Answer':  20115 ->   -100
                 'the':    267 ->   -100
           'following':   1762 ->   -100
                      ...
                'know':    683 ->   -100
                 'the':    267 ->   -100
            'response':   3010 ->   -100
             'details':   3296 ->   -100
                   '.':  30930 ->   -100
       '<|assistant|>':  64796 ->   -100
                    '':  30910 ->  30910
                  '\n':     13 ->     13
                   'I':    307 ->    307
                'need':    720 ->    720
                  'to':    289 ->    289
                 'use':    792 ->    792
                      ...
    <<<<<<<<<<<<< Sanity Check
    ```
    {collapsible="true" default-state="expanded"}

2. 参考显存用量
   - **P-Tuning V2**: `PRE_SEQ_LEN=128`, `DEV_BATCH_SIZE=1`, `GRAD_ACCUMULARION_STEPS=16`, `MAX_SEQ_LEN=2048` 配置下约需要 **21GB** 显存。
   - **全量微调**: 在 ./scripts/finetune_ds_multiturn.sh 中的配置（`MAX_SEQ_LEN=2048`, `DEV_BATCH_SIZE=16`, `GRAD_ACCUMULARION_STEPS=1`）恰好用满 4 * 80GB 显存。

3. 若尝试后发现显存不足，可以考虑:
   - 尝试降低 `DEV_BATCH_SIZE` 并提升 `GRAD_ACCUMULARION_STEPS`
   - 尝试添加 `--quantization_bit 8` 或 `--quantization_bit 4`。
   - `PRE_SEQ_LEN=128`, `DEV_BATCH_SIZE=1`, `GRAD_ACCUMULARION_STEPS=16`, `MAX_SEQ_LEN=1024` 配置下，`--quantization_bit 8` 约需 **12GB** 显存，`--quantization_bit 4` 约需 **7.6GB** 显存。

    
<seealso>
<category ref="ref_github">
    <a href="https://github.com/THUDM/ChatGLM3/tree/main/finetune_chatmodel_demo">官方GLM3微调示例</a>
</category>
</seealso>