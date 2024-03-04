# Qwen1.5 教程

<show-structure depth="4"/>

Qwen 是阿里开源出来的大语言模型和大多模态模型系列，目前大语言模型已经升级到Qwen1.5。语言模型和多模态模型都是在大规模多语言和多模态数据上进行预训练，并在高质量数据上进行**后训练**，以更符合人类偏好。Qwen 作为 AI Agent，能够进行**自然语言理解、文本生成、视觉理解、音频理解、工具使用、角色扮演**等。

目前 Qwen 已经发布到 1.5 版本，具备以下特性:
- 提供 6 种不同大小的模型: 0.5B、1.8B、4B、7B、14B、72B；
- 每种类型都具备 base 模型和 chat 模型，其中 Chat 模型与人类偏好进行了对齐；
- 基础模型和聊天模型支持多语言；
- 所有模型都稳定支持 32K 上下文长度
- 作为 AI Agent，Qwen 支持工具使用、RAG、角色扮演、扮演；

## 1. 安装 {collapsible="true" default-state="expanded"}

使用 Qwen1.5 需要安装 HuggingFace 的 `transformers` 库（建议使用最新版本），安装完成之后就可以使用 [Qwen1.5 集合](https://huggingface.co/collections/Qwen/qwen15-65c0a2f577b1ecb76d786524) 中的模型了。

建议创建新的虚拟环境，再通过 `pip` 或 `conda` 进行安装。此外，建议使用 Python 3.8 和 Pytorch 2.0 以上版本。

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

> 这里只是简单安装了 `transformers` 库，实际上安装 Qwen 还需要安装更多库。
> 
{style="note"}


## 2. 快速入门 {collapsible="true" default-state="expanded"}

安装完 `transformers` 库后，我们通过一些示例（包含 HuggingFace 以及 vLLM 部署）来快速了解下 Qwen1.5 的使用。

### 2.1 在 HuggingFace 中使用

首先需要确保当前 `transformers` 版本大于等于 4.37.0，以下是如何运行 Qwen1.5-7B-Chat 的简单示例，其他 Chat 模型也可以参考此示例代码。

```Python
from transformers import AutoModelForCausalLM, AutoTokenizer
device = "cuda"  # 使用 GPU 载入模型

# 不需要再手动添加 "trust_remote_code=True"
model = AutoModelForCausalLM.from_pretrained(
    "Qwen/Qwen1.5-7B-Chat",
    torch_dtype="auto",
    device_map="auto"
)
tokenizer = AutoTokenizer.from_pretrained("Qwen/Qwen1.5-7B-Chat")

# 我们直接 `model.generate` 替换 `model.chat`，这一点与之前不一样
# 但是我们需要使用 `tokenizer.apply_chat_template` 来格式化输入
prompt = "Give me a short introduction to large language model."
messages = [
    {"role": "system", "content": "You are a helpful assistant."},
    {"role": "user", "content": prompt}
]
text = tokenizer.apply_chat_template(
    messages,
    tokenize=False,
    add_generation_prompt=True
)
model_inputs = tokenizer([text], return_tensors="pt").to(device)

# 直接使用 `model.generate` 和 `tokenizer.decode` 来获取输出
# 使用 `max_new_tokens` 来控制输出的最大长度 
generated_ids = model.generate(
    model_inputs.input_ids,
    max_new_tokens=512
)
generated_ids = [
    output_ids[len(input_ids):] 
    for input_ids, output_ids in zip(model_inputs.input_ids, generated_ids)
]

response = tokenizer.batch_decode(generated_ids, skip_special_tokens=True)[0]
```
{collapsible="true" default-state="expanded"}


> 在之前的 Qwen 模型版本中，我们需要使用 `model.chat` 方法获取输出（Qwen1.5 版本不再提供 modeling_qwen.py 文件了，自然也没有这个方法了）。现在 Qwen 遵循 `transformers` 设计范式，可以直接使用 `model.generate` 和 `tokenizer.apply_chat_template` 方法来进行文本编码和对话，这样做的好处在于我们一套代码不需要调整太多，就可以无缝切换到其他模型。


如果你使用 ModelScope 的话，那么只需要更改下 `AutoModelForCausalLM` 和 `AutoTokenizer` 即可。

```Python
from modelscope import AutoModelForCausalLM, AutoTokenizer
```


如果希望使用流式对话模式，那么只需要借助 `TextStreamer` 即可，请看如下示例。

```Python
from transformers import TextStreamer

streamer = Texstreamer(tokenizer, skip_prompt=True, skip_special_tokens=True)
generated_ids = model.generate(
    model_inputs.input_ids,
    max_new_tokens=512,
    streamer=streamer,  # streaming mode
)
```

### 2.2 vLLM 部署

官方建议使用 vLLM 来部署 Qwen1.5 模型，接下里我们看下如何构建一个与 OpenAI API 接口兼容的服务。

vLLM
: vLLM 是一个快速且易于使用的 LLM 推理服务，基于缓存机制，它能大幅提高大模型服务的并发以及响应速度。


首先，确保我们已经安装了 0.3.0 版本以上的 `vllm`，如果没有的话，请使用 `pip` 进行安装。

```Bash
pip install vllm
```

使用如下代码搭建一个 vllm 服务，这里我们使用 Qwen1.5-7B 来举例:

```Bash
python -m vllm.entrypoints.openai.api_server --model Qwen/Qwen1.5-7B-Chat
```

接下来我们需要创建一个聊天接口与 Qwen 进行交互:

```Bash
curl http://localhost:8000/v1/chat/completions  -H "Content-Type: application/json" -d '{
   "model": "Qwen/Qwen1.5-7B-Chat",
   "messages": [
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "Tell me something about large language models."}
    ]
   }'
```

或者我们也可以使用 `openai` 库中 `client` 完成接口的开发:

```Python
from openai import OpenAI

openai_api_key = "EMPTY"
openai_api_base = "http://localhost:8000/v1"

client = OpenAI(
    api_key=openai_api_key,
    base_url=openai_api_base,
)

chat_response = client.chat.completions.create(
    model="Qwen/Qwen1.5-7B-Chat",
    messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "Tell me something about large language models."},
    ]
)
print("Chat response:", chat_response)
```

## 3. 使用 Transformers 进行聊天 {collapsible="true" default-state="expanded"}

之前在《2.1 快速入门》介绍过如何通过 `transformers` 库使用 Qwen1.5 模型，这里对其基础对话和流式对话再做一个补充。

### 3.1 基础使用

下述代码演示了如何将带有历史对话以及系统提示词的 `messages` 进行编码，再使用 Qwen 大模型生成回复内容，再对输出内容进行解码，拿到我们可以直观理解的文本。

```Python
from transformers import AutoModelForCausalLM, AutoTokenizer
device = "cuda"  # 使用 GPU 载入模型

# 不需要再手动添加 "trust_remote_code=True"
model = AutoModelForCausalLM.from_pretrained(
    "Qwen/Qwen1.5-7B-Chat",
    torch_dtype="auto",
    device_map="auto"
)
tokenizer = AutoTokenizer.from_pretrained("Qwen/Qwen1.5-7B-Chat")

# 我们直接 `model.generate` 替换 `model.chat`，这一点与之前不一样
# 但是我们需要使用 `tokenizer.apply_chat_template` 来格式化输入
prompt = "Give me a short introduction to large language model."
messages = [
    {"role": "system", "content": "You are a helpful assistant."},
    {"role": "user", "content": prompt}
]
text = tokenizer.apply_chat_template(
    messages,
    tokenize=False,
    add_generation_prompt=True
)
model_inputs = tokenizer([text], return_tensors="pt").to(device)

# 直接使用 `model.generate` 和 `tokenizer.decode` 来获取输出
# 使用 `max_new_tokens` 来控制输出的最大长度 
generated_ids = model.generate(
    model_inputs.input_ids,
    max_new_tokens=512
)
generated_ids = [
    output_ids[len(input_ids):] 
    for input_ids, output_ids in zip(model_inputs.input_ids, generated_ids)
]

response = tokenizer.batch_decode(generated_ids, skip_special_tokens=True)[0]
```
{collapsible="true" default-state="expanded"}

上面一小段代码主要涉及 3 个函数：`tokenizer.apply_chat_template`、`model.generate` 以及 `tokenizer.batch_decode`，之前介绍过了 `model.generate` 是用于生成输出的，其他两个函数我们也分别简单说下。

`tokenizer.apply_chat_template` 方法用于将输入信息转换转换位模型能理解的格式
: - `add_generation_prompt` 参数用于添加生成提示（指代的是 `<|im_start|>assistant\n`）
: - `max_new_tokens` 参数用于设置输出的最大长度


`tokenizer.batch_decode` 方法用于解码输出。值得注意的是，按照之前的做法将 ChatML 模板依旧应用在聊天模型上。


### 3.2 流式对话模式

Qwen1.5 也提供了流式对话模式，我们只需要借助 `TextStreamer` 就可以开启，简单的示例代码（2.1 小节）在前面介绍过，这里不再赘述。

```Python
from transformers import TextStreamer

streamer = Texstreamer(tokenizer, skip_prompt=True, skip_special_tokens=True)
generated_ids = model.generate(
    model_inputs.input_ids,
    max_new_tokens=512,
    streamer=streamer,  # streaming mode
)
```
{collapsible="true" default-state="expanded"}


值得一提的是，Qwen 还提供了 `TextIteratorStreamer`，它可以将已经打印的文本存储在队列中，以供下游程序进行调用。

```Python
from threading import Thread
from transformers import TextIteratorStreamer

streamer = TextIteratorStreamer(tokenizer, skip_prompt=True, skip_special_tokens=True)
generation_kwargs = dict(model_inputs, streamer=streamer, max_new_tokens=512)
thread = Thread(target=model.generate, kwargs=generation_kwargs)

thread.start()
generated_text = ""
for new_text in streamer:
    generated_text += new_text

print(generated_text)
```

## 4. llama.cpp {collapsible="true" default-state="expanded"}

[llama.cpp](https://github.com/ggerganov/llama.cpp) 是一个最小设置下使用 C++ 编写（完全使用 C/C++ 实现，没有其他依赖）用于 LLM 推理的库，借助于它我们可以在本地机器上运行 Qwen。

为了加快推理速率以及内存占用，llama.cpp 提供了 2、3、4、5、6、8 比特的量化。同时，当模型大于 VRAM 总容量时，它还支持 CPU+GPU 混合推理来运行模型。本质上，llama.cpp 的作用就是运行 GGUF（GPT 生成的统一格式）。关于更多信息，我们可以参考 Qwen Github 仓库，接下来我们介绍下如何通过 llama.cpp 来运行 Qwen。


### 4.1 运行条件

在 MacOS 上可以通过以下代码来安装 llama.cpp，完成后我们就可以通过 llama.cpp 来运行 GGUF 文件了。

```Bash
git clone https://github.com/ggerganov/llama.cpp
cd llama.cpp
make
```

### 4.2 运行 Qwen GGUF 文件

在 Hugging Face 上已经开源了一系列 Qwen 的 GGUF 模型，我们可以通过**模型名称-GGUF** 来找到你想要的模型。首先，我们通过如下命令安装 `huggingface_hub`，然后通过 `huggingface-cli` 来下载 GGUF 模型。

```Bash
pip install huggingface_hub
```

使用 `huggingface_cli` 下载 GGUF 模型的命令如下：

```Bash
huggingface-cli download <model_repo> <gguf_file> \
    --local-dir <local_dir> \
    --local-dir-use-symlinks False
```

我们还是以 Qwen1.5-7B-Chat 模型来举例，以下是下载其 GGUF 版本的模型命令:

```Bash
huggingface-cli download Qwen/Qwen1.5-7B-Chat-GGUF qwen1_5-7b-chat-q5_k_m.gguf \
    --local-dir . \
    --local-dir-use-symlinks False
```


接下俩我们使用如下命令运行模型:

```Bash
./main -m qwen1_5-7b-chat-q5_k_m.gguf \
    -n 512 --color -i -cml -f prompts/chat-with-qwen.txt
```

其中 `-n` 表示生成的 token 的最大值，也就是之前我们介绍过的 `max_new_tokens` 参数。如果想要查看其他超参数，使用 `./main -h` 进行查阅。


### 4.3 生成自定义 GGUF 文件

在 [quantization/llama.cpp](https://qwen.readthedocs.io/en/latest/quantization/gguf.html) 中介绍了创建量化版本的 GGUF 文件的方法，查阅文档以获取更多信息。

### 4.4 评估困惑度

llama.cpp 提供了一些方法来评估 GGUF 模型的困惑度表现，我们需要准备相应的数据集，并执行以下代码:

<tabs>
<tab title="命令">
<code-block lang="bash">
<![CDATA[
wget https://s3.amazonaws.com/research.metamind.io/wikitext/wikitext-2-raw-v1.zip?ref=salesforce-research -O wikitext-2-raw-v1.zip
unzip wikitext-2-raw-v1.zip
./perplexity -m models/7B/ggml-model-q4_0.gguf -f wiki.test.raw
]]>
</code-block>
</tab>
<tab title="输出">
<code-block lang="python">
<![CDATA[
perplexity : calculating perplexity over 655 chunks
24.43 seconds per pass - ETA 4.45 hours
[1]4.5970,[2]5.1807,[3]6.0382,...
]]>
</code-block>
</tab>
</tabs>

### 4.4 在 LM Studio 中使用 GGUF {collapsible="true" default-state="expanded"}

如果你还依旧觉得使用 llama.cpp 存在困难，那么我们推荐使用 [LM Studio](https://lmstudio.ai)。LM Studio 是一个搜索和运行本地大模型的平台，你可以很轻松地找到 Qwen1.5 官方模型。

## 5. Ollama

[Ollama](https://ollama.com) 旨在通过很少的命令就能运行本地的 LLM，目前 Ollama 已支持官方 Qwen1.5，我们通过如下命令来运行：

```Bash
ollama run qwen
```

### 5.1 下载 Ollama

访问 [Ollama 官网](https://ollama.com/download)下载软件并安装，目前 Qwen 提供了 0.5B、1.8B、4B、7B、14B 以及 72B 的模型版本（默认是 4B），如果你想改变默认版本的话，可以使用标签来运行对应的模型。例如，我们想要运行 0.5B 版本的 Qwen1.5 模型:

```Bash
ollama run qwen:0.5b
```

### 5.2 使用 Ollama 运行自定义 GGUF 文件

有时候我们仅仅是想使用 Ollma 来运行自己的 GGUF 文件，而并不想拉取模型。这里假设我们已经有一个 qwen1_5-7b-chat-q4_0.gguf 文件，首先我们需要创建一个 `Modelfile` 文件，内容如下:

```Bash
FROM qwen1_5-7b-chat-q4_0.gguf

# 将 temperature 设置为 1，值越高表现越有创造力，越低则表示更稳定
PARAMETER temperature 0.7
PARAMETER top_p 0.8
PARAMETER repeat_penalty 1.05
PARAMETER top_k 20

TEMPLATE """{{ if and .First .System }}<|im_start|>system
{{ .System }}<|im_end|>
{{ end }}<|im_start|>user
{{ .Prompt }}<|im_end|>
<|im_start|>assistant
{{ .Response }}"""

# 设置系统消息
SYSTEM """
You are a helpful assistant.
"""
```

接下来通过如下命令创建 ollama 模型，完成后可以 `ollama run` 来运行生成的模型：

<tabs>
<tab title="创建 Ollama 模型">
<code-block lang="bash">
<![CDATA[
ollama create qwen7b -f Modelfile
]]>
</code-block>
</tab>
<tab title="运行 Ollama 模型">
<code-block lang="bash">
<![CDATA[
ollama run qwen7b
]]>
</code-block>
</tab>
</tabs>


## 6. 文本生成的 WebUI {collapsible="true" default-state="expanded"}

文本生成的 WebUI（Text Generation Web UI, TGW）类似于 [AUTOMATIC1111/stable-diffusion-webui](https://github.com/AUTOMATIC1111/stable-diffusion-webui)，它支持多个接口以及多种后端模型，包括 [Transformers](https://github.com/huggingface/transformers)、[llama.cpp](https://github.com/ggerganov/llama.cpp)（通过 [llama-cpp-python](https://github.com/abetlen/llama-cpp-python)）、[ExLlamaV2](https://github.com/turboderp/exllamav2)、[AutoGPTQ](https://github.com/PanQiWei/AutoGPTQ)、[AutoAWQ](https://github.com/casper-hansen/AutoAWQ)、[GPTQ-for-LLaMa](https://github.com/qwopqwop200/GPTQ-for-LLaMa)、[CTransformers](https://github.com/marella/ctransformers)、[QuIP#](https://github.com/Cornell-RelaxML/quip-sharp)，下面我们介绍下如何运行 TGW。


首先，我们推荐重新创建一个新的虚拟环境来运行 TGW，避免对其他环境造成干扰:

```Bash
conda create -n textgen python=3.11
conda activate textgen
pip install torch torchvision torchaudio
```

接下来克隆 TGW 项目，根据自己的系统版本运行仓库中的脚本来运行：

<tabs>
<tab title="MacOS">
<code-block lang="bash">
git clone https://github.com/oobabooga/text-generation-webui
cd text-generation-webui
# 苹果 M 系列芯片
pip install -r requirements_apple_silicon.txt
# 苹果 Interl 系列芯片
pip install -r requirements_apple_intel.txt
./start_macos.sh
</code-block>
</tab>
<tab title="Windows">
<code-block lang="bash">
<![CDATA[
git clone https://github.com/oobabooga/text-generation-webui
cd text-generation-webui
pip install -r requirements_apple.txt
./start_windows.sh
]]>
</code-block>
</tab>
<tab title="Linux">
<code-block lang="bash">
<![CDATA[
git clone https://github.com/oobabooga/text-generation-webui
cd text-generation-webui
pip install -r requirements_apple_amd.txt
./start_linux.sh
]]>
</code-block>
</tab>
</tabs>

我们推荐直接使用 `pip` 安装 `bitsandbytes` 和 `llama-cpp-python`， 建议暂时不要使用 GGUF，因为 TGW 的性能并不太理想。在安装完相关依赖包后，我们需要将模型放置在 `./models` 文件夹下。例如，我们需要将 Qwen1.5-7B-Chat 放在如下结构中：

```Bash
text-generation-webui
├── models
│   ├── Qwen1.5-7B-Chat
│   │   ├── config.json
│   │   ├── generation_config.json
│   │   ├── model-00001-of-00004.safetensor
│   │   ├── model-00002-of-00004.safetensor
│   │   ├── model-00003-of-00004.safetensor
│   │   ├── model-00004-of-00004.safetensor
│   │   ├── model.safetensor.index.json
│   │   ├── merges.txt
│   │   ├── tokenizer_config.json
│   │   └── vocab.json
```

接下来我们需要运行如下命令开启 Web UI 服务，通过 http://localhost:7860/?__theme=dark ：

```Bash
python server.py
```

## 7. AWQ {collapsible="true" default-state="expanded"}

对于量化模型，我们推荐使用 [AWQ](https://arxiv.org/abs/2306.00978) 和 [AutoAWQ](https://github.com/casper-hansen/AutoAWQ)。AWQ 使用激活感知权重算法量化模型，这是一种硬件友好的 LLM 低位权重的量化方法。对于 4 位量化模型而言，AutoAWQ 是非常易于使用的库。与 FP16 相比，AutoAWQ 在内存需要降低 3 倍的情况下还将模型推理速度提升了 3 倍。接下来，我们介绍一下如何使用量化模型，以及如何量化我们自己的模型。

### 7.1 使用 AWQ 量化模型

Transformers 已经官方支持 AutoAWQ，目前我们可以使用这些量化模型。下面的代码展示了如何使用量化模型运行 Qwen15-7B-Chat-AWQ：

```Bash
from transformers import AutoModelForCausalLM, AutoTokenizer

device = "cuda"
model = AutoModelForCausalLM.from_pretrained(
    "Qwen/Qwen1.5-7B-Chat-AWQ",
    device_map="auto"
)
tokenizer = AutoTokenizer.from_pretrained("Qwen/Qwen1.5-7B-Chat-AWQ")

prompt = "Give me a short introduction to large language model."
messages = [
    {"role": "system", "content": "You are a helpful assistant."},
    {"role": "user", "content": prompt}
]
text = tokenizer.apply_chat_template(
    messages,
    tokenize=False,
    add_generation_prompt=True
)
model_inputs = tokenizer([text], return_tensors="pt").to(device)

generated_ids = model.generate(
    model_inputs.input_ids,
    max_new_tokens=512
)
generated_ids = [
    output_ids[len(input_ids):] for input_ids, output_ids in zip(model_inputs.input_ids, generated_ids)
]

response = tokenizer.batch_decode(generated_ids, skip_special_tokens=True)[0]
```
{collapsible="true" default-state="collapsed"}


可以看到除了模型名称，其他部分代码和之前都没有改变。


### 7.2 在 vLLM 中使用 AWQ 量化模型

vLLM 已经支持 AWQ，这意味着我们可以直接使用 AWQ 模型或者经过 AutoAWQ 训练的模型，实际上使用方式与我们之前介绍的 vLLM 一样。Qwen 提供了一个简单示例展示如何在 vLLM 开启一个与 OpenAI-API 兼容的接口。


<tabs>
<tab title="启动服务">
<code-block lang="bash">
<![CDATA[
python -m vllm.entrypoints.openai.api_server --model Qwen/Qwen1.5-7B-Chat-AWQ
]]>
</code-block>
</tab>
<tab title="调用接口">
<code-block lang="bash">
<![CDATA[
curl http://localhost:8000/v1/chat/completions \ 
    -H "Content-Type: application/json" \
    -d '{
        "model": "Qwen/Qwen1.5-7B-Chat-AWQ",
        "messages": [
            {"role": "system", "content": "You are a helpful assistant."},
            {"role": "user", "content": "Tell me something about large language models."}
        ]
   }'
]]>
</code-block>
</tab>
</tabs>


或者我们使用 Python 的 `openai` 库来调用：

```Bash
from openai import OpenAI

# 设置 OpenAI API Key（本地服务设置为空）以及 URL
# Set OpenAI's API key and API base to use vLLM's API server.
openai_api_key = "EMPTY"
openai_api_base = "http://localhost:8000/v1"

client = OpenAI(
    api_key=openai_api_key,
    base_url=openai_api_base,
)

chat_response = client.chat.completions.create(
    model="Qwen/Qwen1.5-7B-Chat-AWQ",
    messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "Tell me something about large language models."},
    ]
)
print("Chat response:", chat_response)
```

### 7.3 使用 AutoAWQ 量化自己的模型

如果我们想要量化自己的 AWQ 模型，就可以使用 AutoAWQ（建议安装最新版）:

```Bash
git clone https://github.com/casper-hansen/AutoAWQ.git
cd AutoAWQ
pip install -e .
```

这里我们在自己的数据集上基于 Qwen1.5-7B 已经微调了一版模型 Qwen1.5-7B-finetuned，为了构建自己的 AWQ 量化模型，我们还需要使用训练数据来校准。这里我们提供一个简单的运行示例：

```Python
from awq import AutoAWQForCausalLM
from transformers import AutoTokenizer

# 指定路径和量化的超参数
model_path = "your_model_path"
quant_path = "your_quantized_model_path"
quant_config = { "zero_point": True, "q_group_size": 128, "w_bit": 4, "version": "GEMM" }

# 加载自己的 tokenizer 和 AutoAWQ 模型
tokenizer = AutoTokenizer.from_pretrained(model_path)
model = AutoAWQForCausalLM.from_pretrained(model_path, device_map="auto", safetensors=True)
```

接下来，我们需要准备数据来校准（将文本样本组成一个列表），因为我们直接使用微调数据来校准，首先使用 Chatml 将其格式化。例如：

<tabs>
<tab title="代码">
<code-block lang="python">
<![CDATA[
data = []
for msg in messages:
    msg = c['messages']
    text = tokenizer.apply_chat_template(msg, tokenize=False, add_generation_prompt=False)
    data.append(text.strip())
# 使用数据进行校准
model.quantize(tokenizer, quant_config=quant_config, calib_data=data)
# 保存校准后的模型
model.save_quantized(quant_path, safetensors=True, shard_size="4GB")
tokenizer.save_pretrained(quant_path)
]]>
</code-block>
</tab>
<tab title="样本数据">
<code-block lang="python">
<![CDATA[
[
    {"role": "system", "content": "You are a helpful assistant."},
    {"role": "user", "content": "Tell me who you are."},
    {"role": "assistant", "content": "I am a large language model named Qwen..."}
]
]]>
</code-block>
</tab>
</tabs>

接下来我们就可以使用 AWQ 量化模型进行部署了。

## 8. GPTQ {collapsible="true" default-state="expanded"}

[GPTQ](https://arxiv.org/abs/2210.17323) 是一种类似 GPT 的 LLM 量化方法，它使用基于近似二阶信息的one-shot 权重量化。接下来，我们介绍下如何使用量化模型以及使用 AutoGPTQ 量化自己的模型。

### 8.1 使用 GPTQ 模型

目前，Transformers 已经官方支持 AutoGPTQ，这意味着我们可以直接使用这些量化模型。以下代码是如何运行 Qwen1.5-7B-Chat-GPTQ-Int8 量化模型的简单示例。值得注意的是，对于每种大小的 Qwen1.5，官方都提供了 Int4 和 Int8 版本的量化模型。

```Python
from transformers import AutoModelForCausalLM, AutoTokenizer

device = "cuda"
model = AutoModelForCausalLM.from_pretrained(
    "Qwen/Qwen1.5-7B-Chat-GPTQ-Int8",
    device_map="auto"
)
tokenizer = AutoTokenizer.from_pretrained("Qwen/Qwen1.5-7B-Chat-GPTQ-Int8")

prompt = "Give me a short introduction to large language model."
messages = [
    {"role": "system", "content": "You are a helpful assistant."},
    {"role": "user", "content": prompt}
]
text = tokenizer.apply_chat_template(
    messages,
    tokenize=False,
    add_generation_prompt=True
)
model_inputs = tokenizer([text], return_tensors="pt").to(device)

generated_ids = model.generate(
    model_inputs.input_ids,
    max_new_tokens=512
)
generated_ids = [
    output_ids[len(input_ids):] for input_ids, output_ids in zip(model_inputs.input_ids, generated_ids)
]

response = tokenizer.batch_decode(generated_ids, skip_special_tokens=True)[0]
```

### 8.2 在 vLLM 中使用 GPTQ 量化模型

vLLM 已经支持 GPTQ，这意味着我们可以直接使用这些 GPTQ 模型，或者使用 AutoGPTQ 训练的模型。实际上，这与 vLLM 基础使用方式一样。Qwen 提供了一个简单示例展示如何在 vLLM 开启一个与 OpenAI-API 兼容的接口。


<tabs>
<tab title="启动服务">
<code-block lang="bash">
<![CDATA[
python -m vllm.entrypoints.openai.api_server --model Qwen/Qwen1.5-7B-Chat-Int8
]]>
</code-block>
</tab>
<tab title="调用接口">
<code-block lang="bash">
<![CDATA[
curl http://localhost:8000/v1/chat/completions \ 
    -H "Content-Type: application/json" \
    -d '{
        "model": "Qwen/Qwen1.5-7B-Chat-Int8",
        "messages": [
            {"role": "system", "content": "You are a helpful assistant."},
            {"role": "user", "content": "Tell me something about large language models."}
        ]
   }'
]]>
</code-block>
</tab>
</tabs>


或者我们使用 Python 的 `openai` 库来调用：

```Bash
from openai import OpenAI

# 设置 OpenAI API Key（本地服务设置为空）以及 URL
# Set OpenAI's API key and API base to use vLLM's API server.
openai_api_key = "EMPTY"
openai_api_base = "http://localhost:8000/v1"

client = OpenAI(
    api_key=openai_api_key,
    base_url=openai_api_base,
)

chat_response = client.chat.completions.create(
    model="Qwen/Qwen1.5-7B-Chat-Int8",
    messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "Tell me something about large language models."},
    ]
)
print("Chat response:", chat_response)
```


### 8.3 使用 AutoGPTQ 量化自己的模型

如果我们想要量化自己的 GPTQ 模型，就可以使用 AutoGPTQ（建议安装最新版）:

```Bash
git clone https://github.com/AutoGPTQ/AutoGPTQ
cd AutoGPTQ
pip install -e .
```

这里我们在自己的数据集上基于 Qwen1.5-7B 已经微调了一版模型 Qwen1.5-7B-finetuned，为了构建自己的 AWQ 量化模型，我们还需要使用训练数据来校准。这里我们提供一个简单的运行示例：

```Python
from awq import AutoAWQForCausalLM
from transformers import AutoTokenizer

# 指定路径和量化的超参数
model_path = "your_model_path"
quant_path = "your_quantized_model_path"
quantize_config = BaseQuantizeConfig(
    bits=8, # 4 or 8
    group_size=128,
    damp_percent=0.01,
    desc_act=False,  # set to False can significantly speed up inference but the perplexity may slightly bad
    static_groups=False,
    sym=True,
    true_sequential=True,
    model_name_or_path=None,
    model_file_base_name="model"
)
max_len = 8192

# 加载自己的 tokenizer 和 AutoGPTQ 模型
tokenizer = AutoTokenizer.from_pretrained(model_path)
model = AutoAWQForCausalLM.from_pretrained(model_path, device_map="auto", safetensors=True)
```

接下来，我们需要准备数据来校准（将文本样本组成一个列表），因为我们直接使用微调数据来校准，首先使用 Chatml 将其格式化。例如：

<tabs>
<tab title="代码">
<code-block lang="python" ignore-vars="true">
<![CDATA[
data = []
for msg in messages:
    msg = c['messages']
    text = tokenizer.apply_chat_template(msg, tokenize=False, add_generation_prompt=False)
    model_inputs = tokenizer([text])
    input_ids = torch.tensor(model_inputs.input_ids[:max_len], dtype=torch.int)
    data.append(dict(input_ids=input_ids, attention_mask=input_ids.ne(tokenizer.pad_token_id)))

logging.basicConfig(
    format="%(asctime)s %(levelname)s [%(name)s] %(message)s", 
    level=logging.INFO, datefmt="%Y-%m-%d %H:%M:%S"
)
model.quantize(data, cache_examples_on_gpu=False)

model.save_quantized(quant_path, safetensors=True)  # 保存校准后的模型
tokenizer.save_pretrained(quant_path)
]]>
</code-block>
</tab>
<tab title="样本数据">
<code-block lang="python">
<![CDATA[
[
    {"role": "system", "content": "You are a helpful assistant."},
    {"role": "user", "content": "Tell me who you are."},
    {"role": "assistant", "content": "I am a large language model named Qwen..."}
]
]]>
</code-block>
</tab>
</tabs>

很遗憾，`save_pretrained` 方法并不支持分片。如果想要分片的话，我们需要加载模型并使用 Transformers 的 `save_pretrained` 方法以分片方式保存模型。使用 llama.cpp，我们不仅可以为模型构建 GGUF 文件，还可以执行低位量化。

## 9. GGUF {collapsible="true" default-state="expanded"}

最近，在本地运行 LLM 在社区中很流行，用 llama.cpp 运行 GGUF 文件就是一个很典型的例子。在 GGUF 中，我们还可以直接量化模型而无需校准，或者应用 AWQ 来获得更好的质量，再或者使用 imatrix 来校准数据的。接下来，我们介绍下最简单的方式来量化自己的模型，以及在 Qwen 模型中应用 AWQ。

### 9.1 量化自己的模型并创建 GGUF 文件

在量化之前，确保已经按照说明进行了操作以及开始使用 llama.cpp。这里不会介绍有关安装和构建的说明，如果你想要量化 Qwen1.5-7B-Chat 模型，首先需要给 fp16 模型创建 GGUF 文件：

```Bash
python convert-hf-to-gguf.py Qwen/Qwen1.5-7B-Chat \
    --outfile models/7B/qwen1_5-7b-chat-fp16.gguf
```

第一个参数指定了模型名称（本地运行的话填写模型路径），第二个参数则是 GGUF 文件的输出路径。这种方式下，我们就给 fp16 模型生成了 GGUF 文件，接下来我们只需要将其量化到低位即可（这里 `q4_0` 表示 4 位量化）。

```Bash
./quantize models/7B/qwen1_5-7b-chat-fp16.gguf models/7B/qwen1_5-7b-chat-q4_0.gguf q4_0
```

到这里，我们就完成了将模型量化到 4 位，并将其放进 GGUF 文件中，量化后模型可以直接在 llama.cpp 中运行。


### 9.2 使用 AWQ 量化自己的模型

为了提高量化模型的质量，一种可行的办法是使用 AWQ，详细参考脚本[文档](https://github.com/casper-hansen/AutoAWQ/blob/main/docs/examples.md)。首先，当我们使用 AutoAWQ 运行 `model.quantize` 时，记得添加 `export_compatible=True`。
```Python

model.quantize(
    tokenizer,
    quant_config=quant_config,
    export_compatible=True
)
model.save_pretrained(quant_path)
```

执行完成后，就生成了 AWQ 缩放的 fp16 模型。接下来，当我们运行 `convert-hf-to-gguf.py`，记得替换新生成的模型路径。

```Bash
python convert-hf-to-gguf.py ${quant_path} --outfile models/7B/qwen1_5-7b-chat-fp16-awq.gguf
```

进行以上操作或，我们就完成了在量化模型上 AWQ 缩放，并以 GGUF 格式保存，从而帮助模型提升质量。

一般情况下，我们会将模型量化到 2、3、4、5、6、8 位，如果想要量化其他不同低位，仅需要在命令中替换量化方法。例如我们想量化模型到 2 位，则可以使用 `q2_k` 替换上面的 `q4_0`：

```Bash
./quantize models/7B/qwen1_5-7b-chat-fp16.gguf models/7B/qwen1_5-7b-chat-q2_k.gguf q2_k
```

> 目前量化等级有: `q2_k`、`q3_k_m`、`q4_0`、`q4_k_m`、`q5_0`、`q5_k_m`、`q6_k` 以及 `q8_0`，更多详细信息，请查看 [llama.cpp](https://github.com/ggerganov/llama.cpp)。


## 10. vLLM {collapsible="true" default-state="expanded"}

Qwen 官方推荐使用 vLLM 进行模型部署，vLLM 不仅易于使用，并且具备高吞吐量，使用 PageAttention 有效管理注意力键值对内存、连续批处理输入请求，以及优化的 CUDA 内核等。如果想了解更多 vLLM 相关知识，请参阅[论文](https://arxiv.org/abs/2309.06180)和[文档](https://vllm.readthedocs.io)。


### 10.1 安装

vLLM 安装还是非常简单的，直接使用 `pip install vllm>=0.3.0` 进行安装即可。如果你使用的是 CUDA 11.8，参考[此文档](https://docs.vllm.ai/en/latest/getting_started/installation.html)进行安装。此外，官方推荐使用 `pip install ray` 安装 ray 开启分布式服务。

### 10.2 离线批推理

vLLM 支持离线 Qwen1.5 和 Qwen2 Code 的离线批推理，以下是示例代码。

```Python
from transformers import AutoTokenizer
from vllm import LLM, SamplingParams
# 初始化 tokenizer
tokenizer = AutoTokenizer.from_pretrained("Qwen/Qwen1.5-7B-Chat")
# Pass the default decoding hyperparameters of Qwen1.5-7B-Chat
sampling_params = SamplingParams(
    temperature=0.7, top_p=0.8, repetition_penalty=1.05, max_tokens=512
    )

# 模型路径，也可以是 GPTQ 或者 AWQ 模型
llm = LLM(model="Qwen/Qwen1.5-7B-Chat")

# Prepare your prompts
prompt = "Tell me something about large language models."
messages = [
    {"role": "system", "content": "You are a helpful assistant."},
    {"role": "user", "content": prompt}
]
text = tokenizer.apply_chat_template(
    messages,
    tokenize=False,
    add_generation_prompt=True
)

# 生成输出
outputs = llm.generate([text], sampling_params)

# 打印输出
for output in outputs:
    prompt = output.prompt
    generated_text = output.outputs[0].text
    print(f"Prompt: {prompt!r}, Generated text: {generated_text!r}")
```
{collapsible="true" default-state="collapsed"}


### 10.3 OpenAI 兼容的 API 接口

构建一个能在 vLLM 运行的 OpenAI-API 兼容接口其实很简单（只需要实现 OpenAI API 协议），默认情况下服务地址为 http://localhost:8000，我们也可以通过 `--host` 和 `--port` 来指定 IP 和端口。示例运行命令如下所示：

```Bash
python -m vllm.entrypoints.openai.api_server \
    --model Qwen/Qwen1.5-7B-Chat
```

我们不需要担心聊天模板，因为它默认使用 tokenizer 提供的聊天。接下来，我们就可以使用聊天接口与 Qwen 进行交流：

```Bash
curl http://localhost:8000/v1/chat/completions \ 
    -H "Content-Type: application/json" \
    -d '{
        "model": "Qwen/Qwen1.5-7B-Chat-Int8",
        "messages": [
            {"role": "system", "content": "You are a helpful assistant."},
            {"role": "user", "content": "Tell me something about large language models."}
        ]
   }'
```

或者我们使用 Python 的 `openai` 库来调用：

```Bash
from openai import OpenAI

# 设置 OpenAI API Key（本地服务设置为空）以及 URL
# Set OpenAI's API key and API base to use vLLM's API server.
openai_api_key = "EMPTY"
openai_api_base = "http://localhost:8000/v1"

client = OpenAI(
    api_key=openai_api_key,
    base_url=openai_api_base,
)

chat_response = client.chat.completions.create(
    model="Qwen/Qwen1.5-7B-Chat-Int8",
    messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "Tell me something about large language models."},
    ]
)
print("Chat response:", chat_response)
```


### 10.4 多GPU分布式服务

我们可以通过使用多 GPU 开启分布式服务，从而扩大服务吞吐量。除此之外，类似于 Qwen1.5-72B-Chat 几乎不可能在单 GPU 上运行的。这里我们介绍下如何使用并行 tensor 来运行 Qwen1.5-72B-Chat，方法很简单，只需要正确设置下 `tensor_parallel_size` 参数即可。

```Python
from vllm import LLM, SamplingParams

llm = LLM(model="Qwen/Qwen1.5-72B-Chat", tensor_parallel_size=4)
```

我们也可以在命令行参数中添加 `--tensor-parallel-size` 参数来开启多 GPU 服务：

```Bash
python -m vllm.entrypoints.api_server \
    --model Qwen/Qwen1.5-72B-Chat \
    --tensor-parallel-size 4
```

### 10.5 使用量化模型开启服务

vLLM 支持各种不同类型的量化模型，包含 AWQ、GPTQ、SqueezeLLM 等。这里我们介绍下如何部署 AWQ 和 GPTQ 模型，使用方式和上面几乎一致，除了 `quantization` 参数值不一样：

<tabs>
<tab title="AWQ">
<code-block lang="python">
<![CDATA[
from vllm import LLM, SamplingParams

llm = LLM(model="Qwen/Qwen1.5-7B-Chat-AWQ", quantization="awq")
]]>
</code-block>
</tab>
<tab title="GPTQ">
<code-block lang="python">
<![CDATA[
from vllm import LLM, SamplingParams

llm = LLM(model="Qwen/Qwen1.5-7B-Chat-AWQ", quantization="gptq")
]]>
</code-block>
</tab>
</tabs>


当然也可以在命令行中直接添加 `--quantization` 参数来开启：

<tabs>
<tab title="AWQ">
<code-block lang="bash">
<![CDATA[
python -m vllm.entrypoints.openai.api_server \
    --model Qwen/Qwen1.5-7B-Chat-AWQ \
    --quantization awq
]]>
</code-block>
</tab>
<tab title="GPTQ">
<code-block lang="bash">
<![CDATA[
python -m vllm.entrypoints.openai.api_server \
    --model Qwen/Qwen1.5-7B-Chat-GPTQ-Int8 \
    --quantization gptq
]]>
</code-block>
</tab>
</tabs>

此外，vLLM 还支持使用 KV 缓存量化下 AWQ 和 GPTQ 结合的模型，称之为 FP8 E5M2 KV 缓存：

<tabs>
<tab title="代码">
<code-block lang="python">
<![CDATA[
llm = LLM(model="Qwen/Qwen1.5-7B-Chat-GPTQ-Int8", quantization="gptq", kv_cache_dtype="fp8_e5m2")
]]>
</code-block>
</tab>
<tab title="命令行">
<code-block lang="bash">
<![CDATA[
python -m vllm.entrypoints.openai.api_server \
    --model Qwen/Qwen1.5-7B-Chat-GPTQ-Int8 \
    --quantization gptq \
    --kv-cache-dtype fp8_e5m2
]]>
</code-block>
</tab>
</tabs>

### 10.6 常见问题

如果遇到非常烦人的内存溢出（Out-of-Memory, OOM）问题，这里推荐两个参数进行修复：
- `--max-model-len`: `max_position_embedding` 默认值为 32768，因此服务的最大长度也是这个值，导致服务对内存的要求更高。我们可以通过适当降低 `--max-model-len` 的值来解决 OOM 问题。
- `--gpu-memory-utiliaztion`: 该参数默认值位 0.9，我们可以增大改制来处理 OOM 问题。


## 11. SFT {collapsible="true" default-state="expanded"}

官方提供了一个非常简单的脚本进行微调(Supervised Fine-Tuning，SFT)，该脚本由 [FastChat](https://github.com/lm-sys/FastChat) 改写而来。[该脚本](https://github.com/QwenLM/Qwen1.5/blob/main/finetune.py)使用 Hugging Face 的 `Trainer` 来微调 Qwen，具备以下特性：
- 支持单/多 GPU 训练
- 支持全参数微调、LoRA 以及 Q-LoRA

### 11.1 安装

开始之前，确保已经正确安装了以下库：

```Bash
pip install peft deepspeed optimum accelerate
```

### 11.2 数据准备

我们推荐将数据保存在 jsonl 文件中，并且每一行是一个字段：


<tabs>
<tab title="示例1">
<code-block lang="json">
<![CDATA[
{
    "type": "chatml",
    "messages": [
        {
            "role": "system",
            "content": "You are a helpful assistant."
        },
        {
            "role": "user",
            "content": "Tell me something about large language models."
        },
        {
            "role": "assistant",
            "content": "Large language models are a type of language model that is trained on a large corpus of text data. They are capable of generating human-like text and are used in a variety of natural language processing tasks..."
        }
    ],
    "source": "unknown"
}
]]>
</code-block>
</tab>
<tab title="示例2">
<code-block lang="python">
<![CDATA[
{
    "type": "chatml",
    "messages": [
        {
            "role": "system",
            "content": "You are a helpful assistant."
        },
        {
            "role": "user",
            "content": "Tell me something about large language models."
        },
        {
            "role": "assistant",
            "content": "Large language models are a type of language model that is trained on a large corpus of text data. They are capable of generating human-like text and are used in a variety of natural language processing tasks..."
        }
    ],
    "source": "self-made"
}
]]>
</code-block>
</tab>
</tabs>


上面两个示例为数据集中单条样本，每个样本都是一个 JSON 对象，包含 `type`、`messages` 以及 `source` 必需字段，其他字段则是可选的。其中 `messages` 字段是一个 json 对象的列表，每个 json 对象包含 `role` 和 `content` 字段，这里 `role` 值可为 `system`、`user` 和 `assistant`，`content` 则为真实的文本，`source` 字段可为 `self-made`、`alpaca`、`open-hermes` 或者任何字符串。

创建 jsonl 文件，可以使用 `json` 模块来将数据进行保存：

```Python
import json

with open('data.jsonl', 'w') as f:
    for sample in samples:
        f.write(json.dumps(sample) + '\n')
```

### 11.3 开始微调

这里我们提供了一个 Shell 脚本来快速地进行微调，而不必特别关注实现细节。针对不同类型的训练，我们需要做的是指定不同的参数值，例如是否使用多 GPU、是使用全量微调、LoRA 还是 Q-LoRA？

```Bash
cd examples/sft
bash finetune.sh -m <model_path> -d <data_path> --deepspeed <config_path> [--use_lora True] [--q_lora True]
```

我们需要在上述命令中指定参数的值:
- `model_path`: 模型路径
- `data_path`: 数据集路径
- `--config_path`: deepspeed  的配置文件
- `--use_lora`: 是否使用 LoRA
- `--q_lora`: 是否使用 Q-LoRA

以上仅仅是非常简单的微调命令，如果想要改变更多参数，我们还需要仔细研究脚本来更改参数值。

### 11.4 高级使用

接下来，我们介绍以下脚本的细节，包含 Shell 脚本和 Python 脚本的核心代码。

#### I) Shell 脚本

在介绍 Python 代码前，我们简单提供 Shell 脚本中命令的说明。此外，我们已经在脚本中添加了一些说明，这里以 finetune.sh 进行举例。

如果想要设置分布式训练或者单 GPU 训练的话，可以多关注下 `GPUS_PER_NODE`、`NNODES`、`NODE_RANK`、`MASTER_ADDR` 以及 `MASTER_PORT` 这些变量，以下是一些参数表格：

| 参数                              | 描述                                                |
|---------------------------------|---------------------------------------------------|
| `-m`                            | 模型路径                                              |
| `-d`                            | 数据集路径                                             |
| `--deepspeed`                   | DeepSpeed 的配置文件路径，这里提供了 Zero2 和 Zero3（根据自己需求自行选择） |
| `--bp16`                        | 在混合精度训练中进行指定                                      | 
| `--fp16`                        | 在混合精度训练中进行指定                                      |
| `--output_dir`                  | 模型或 Adapter 的输出路径                                 |
| `--num_train_epochs`            | 训练的 epoch                                         |
| `--gradient_accumulation_steps` | 多少步进行一次梯度累加                                       |
| `--per_device_train_batch_size` | 每个 GPU 分配多少训练样本                                   |
| `--learning_rate `              | 学习率，建议不要设置过大                                      |
| `--warmup_steps`                | 热启动的步数                                            |
| `--lr_scheduler_type`           | 学习的调度类型                                           |
| `--weight_decay`                | 权重惩罚的系数                                           | 
| `--adam_beta2`                  | Adam 中 beta2 值                                    |
| `--model_max_length`            | 最大序列长度                                            |
| `--use_lora`                    | 是否使用 LoRA                                         |
| `--q_lora`                      | 是否使用 Q-LoRA                                       |
| `--gradient_checkpointing`      | 是否启用梯度检查                                          |

> 1. 一般情况下，除了 Q-LoRA（推荐使用 Zero2），我们推荐 Zero3 来进行多 GPU 训练。
> 2. 使用 `--bp16` 还是 `--fp16`，取决于你使用的 GPU 是什么型号
> 3. 每次批训练的样本量为 `per_device_train_batch_size` x `number_of_gpus` x `gradient_accumulation_steps`


#### II) Python 脚本

我们主要




<tabs>
<tab title="tab1">


</tab>
<tab title="tab2">
<code-block lang="python">
<![CDATA[
123
]]>
</code-block>
</tab>
</tabs>








<seealso>
<category ref="ref_docs">
    <a href="https://qwen.readthedocs.io/en/latest/#documentation">Qwen1.5 文档</a>
    <a href="https://qwenlm.github.io">Qwen 博客</a>
    <a href="https://huggingface.co/collections/Qwen/qwen15-65c0a2f577b1ecb76d786524">Qwen 模型集合</a>
    <a href="https://platform.openai.com/docs/api-reference/chat/completions/create">OpenAI create chat interface</a>
    <a href="https://lmstudio.ai">LM Studio</a>
    <a href="https://ollama.com">Ollama</a>
</category>
<category ref="ref_github">
    <a href="https://github.com/QwenLM">QwenLM</a>
    <a href="https://github.com/lm-sys/FastChat">FastChat</a>
    <a href="https://github.com/ggerganov/llama.cpp">llama.cpp</a>
    <a href="https://github.com/abetlen/llama-cpp-python">llama-cpp-python</a>
    <a href="https://github.com/AUTOMATIC1111/stable-diffusion-webui">stable-diffusion-webui</a>
    <a href="https://github.com/huggingface/transformers">Transformers</a>
    <a href="https://github.com/turboderp/exllamav2">ExLlamaV2</a>
    <a href="https://github.com/PanQiWei/AutoGPTQ">AutoGPTQ</a>
    <a href="https://github.com/casper-hansen/AutoAWQ">AutoAWQ</a>
    <a href="https://github.com/qwopqwop200/GPTQ-for-LLaMa">GPTQ-for-LLaMa</a>
    <a href="https://github.com/marella/ctransformers">CTransformers</a>
    <a href="https://github.com/Cornell-RelaxML/quip-sharp">Quip#</a>
    <a href="https://github.com/oobabooga/text-generation-webui">Text Generation WebUI</a>
</category>
<category ref="ref_issues">
</category>
<category ref="ref_hf">
    <a href="https://huggingface.co/Qwen">Qwen</a>
</category>
<category ref="ref_ms">
    <a href="https://modelscope.cn/organization/qwen">Qwen</a>
</category>
</seealso>

