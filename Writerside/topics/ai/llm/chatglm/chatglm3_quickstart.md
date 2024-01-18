# ChatGLM3初步使用

<show-structure depth="2"/>

ChatGLM3 开源了多个版本的模型，这里主要关注 6B 模型的使用。**官方 Repo 中提及 Base 模型不具备对话能力，仅能够生成单轮回复，需要使用多轮对话的场景，使用 Chat 模型会更合适**。

> 这里就有一个问题，在不涉及多轮对话的场景下，Base 模型的推理性能表现会比 Chat 模型要好么？
> 
> 根据 ChatGLM3 官方团队的回复，在非多轮对话的场景下，Base 的推理性能是要由于 Chat 模型的。但是实际测试下来，Base 模型的效果要比 Chat 模型是要差的。
{style="note"}


## 1. 安装

创建一个干净的虚拟环境，下载官方仓库后安装依赖库:

```Bash
conda create -n chatglm3 python==3.10.12
conda activate chatglm3
git clone https://github.com/THUDM/ChatGLM3.git
cd ChatGLM3/
pip install -r requirements.txt
```

如果你不需要使用 ChatGLM3 官方仓库的其他内容，那么直接下载 requirements.txt 文件到本地项目文件下即可。


## 2. 使用


```Python
from transformers import AutoTokenizer, AutoModel

tokenizer = AutoTokenizer.from_pretrained(
                "THUDM/chatglm3-6b", trust_remote_code=True)
model = AutoModel.from_pretrained(
        "THUDM/chatglm3-6b", trust_remote_code=True, device='cuda'
        )
model = model.eval()

response, history = model.chat(tokenizer, "你好", history=[])
response
```

> 这里非常需要注意的是，官方代码中传给 `history` 参数是 `[]`，如果你设定一个 `hist = []` 的话，并且将 `hist` 参数传给 `history` 这个并不合适，且会导致一个严重 BUG。如果你处理的是一个分类任务，大概率你后续每次传进去都是 `hist`，随着对话轮次越来越多的话，你会发现 ChatGLM3 的推理效率逐渐下降，显存也会几乎被全部占满。
{style="warning"}


## 3. 微调 Chat 模型

ChatGLM3 采用了全新设计的 Prompt，虽然这个绝大程度上参考了 LLaMa 的 Prompt 设计范式，微调方式也分为 Chat 和 Base 模型的微调，所对应需要准备数据集格式也是完全不同的，后面单独再讲这两种微调方式。

## 4. 常见问题


### 4.1  BUG: ChatGLM3导致显存爆炸

在使用 ChatGLM3-6B 模型的过程中，发现一个 BUG，相同的 BUG 实际上在 QWEN 上存在过，当时我给他们提了 [Issue](https://github.com/QwenLM/Qwen/issues/527) 并给出了解决方方案。

我们先来看下这个 BUG 具体表现，后面再给出具体的解决方法。 这里第一轮我们给定 ChatGLM3 一个系统 Prompt 提示，并使用 `hist` 保存首轮历史对话信息。

```Python
response, hist = model.chat(
    tokenizer, 
    query="你是一个专业的作者，请根据我的提示写一篇报道",
    history=None,
    role="system"
)
```

接下来，我们将 `hist` 传给 `history` 参数，并让模型生产一篇报道：

```Python
response, hist2 = model.chat(
    tokenizer, query="上海今天天气真好", history=hist
)
```

ChatGLM3 给我们生成了一篇文章，但是此时我们打印 `hist` 变量时，会惊奇地发现 `hist` 中内容变化了，它将本次对话的信息追加到了 `hist` 中，这显然是一个非常严重的 BUG，如果你是一个量非常大的处理，那么这个 BUG 会导致随着时间的推移，`hist` 中信息会越来越多，最终会导致现存爆炸。

Hugging Face 上 ChatGLM3-6B 的仓库中 modeling_chatglm.py 文件，里面 `chat` 方法定义的代码如下所示，BUG 主要在倒数第二行上，

```Python
@torch.inference_mode()
def chat(
    self, 
    tokenizer, 
    query: str, 
    history: List[Dict] = None, 
    role: str = "user",
    max_length: int = 8192, 
    num_beams=1, 
    do_sample=True, 
    top_p=0.8, 
    temperature=0.8, 
    logits_processor=None,
    **kwargs
):
    if history is None:
        history = []
    
    if logits_processor is None:
        logits_processor = LogitsProcessorList()
    
    logits_processor.append(InvalidScoreLogitsProcessor())
    gen_kwargs = {
        "max_length": max_length, "num_beams": num_beams,
        "do_sample": do_sample, "top_p": top_p,
        "temperature": temperature, 
        "logits_processor": logits_processor, **kwargs
        }
    inputs = tokenizer.build_chat_input(
                query, history=history, role=role)
    inputs = inputs.to(self.device)
    eos_token_id = [
        tokenizer.eos_token_id, tokenizer.get_command("<|user|>"),
        tokenizer.get_command("<|observation|>")
    ]
    outputs = self.generate(
                **inputs, **gen_kwargs, eos_token_id=eos_token_id)
    outputs = outputs.tolist()[0][len(inputs["input_ids"][0]):-1]
    response = tokenizer.decode(outputs)
    history.append({"role": role, "content": query})
    response, history = self.process_response(response, history)
    return response, history
```
{collapsible="true" default-state="collapsed" collapsed-title="原始代码"}


这里我们需要修改下源码来解决这个问题，解决方法也很简单，使用**深拷贝**来代替**浅拷贝**，保证每次传进来的 `history` 参数值没有被触碰过：

```Python
import copy

@torch.inference_mode()
def chat(
    self, 
    tokenizer, 
    query: str, 
    history: List[Dict] = None, 
    role: str = "user",
    max_length: int = 8192, 
    num_beams=1, 
    do_sample=True, 
    top_p=0.8, 
    temperature=0.8, 
    logits_processor=None,
    **kwargs
):
    if history is None:
        history = []
    else:
        history = copy.deepcopy(history)
    
    ....  # 代码不变
    return response, history
```
{collapsible="true" default-state="collapsed" collapsed-title="修正代码"}



<seealso>
<category ref="ref_github_issues">
    <a href="https://github.com/QwenLM/Qwen/issues/527">qwen-7b-chat推理效率随着遍历时间逐渐增加</a>
</category>
</seealso>


