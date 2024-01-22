# BERT-VITS2 教程

<show-structure for="chapter,procedure" depth="2"/>

BERT-VITS2 是一款优秀的 TTS 引擎，在中文语音合成方面有着非常不错的效果，但是正如原项目作者在 README 中写到 **“成熟的旅行者...应当参阅代码自己学习如何训练”**，该项目目前没有任何文档和使用教程，并且很多问题找不到具体的解决方案，在使用门槛上可以说非常高了。

通过一段时间对 BERT-VITS2 的学习和使用，在解决过程中遇到的一些问题时，让我有了编写此文档的想法。此外，我尝试对 BERT-VITS2 整个项目进行了重构，对很多代码进行了简化，并且基于 Streamlit 搭建了更易于使用的 WEB 工具，有兴趣的同学可以参考下我的项目 [bert-vits2-turbo](https://github.com/mingminyu/bert-vits2-turbo)。

> 本篇文档主要是针对 bert-vits2-turbo 的使用教程说明。另外，在重构过程中也发现了目前 BERT-VITS2-Integration-Package 很多代码是比较冗余的，顺便做了一点点优化。

> 目前 BERT-VITS2 版本有 1.0、2.0.2、2.1 版本，对应到 bert-vits2-turbo 中不同分支（master 分支为 1.0 版本）。如需要使用 2.x 版本，请自行切换分支版本。
{style="note"}

## 0. 框架流程图

<code-block lang="plantuml">
<![CDATA[
@startmindmap
* bert-vits2 工作流
   * 1. 分离双声道
   * 2. 音频降噪
   * 3. 切分音频
   * 4. 生成样本
   * 5. 处理文本
   * 6. 训练模型
   * 7. 语音合成
@endmindmap
]]>
</code-block>


## 1. 开发环境搭建

BERT-VITS2 依赖的环境相对比较复杂，如果安装的一些依赖库版本不对，也会造成后面模型的训练以及语音的和合成存在较大问题。这里建议使用重新创建一个 conda 环境来进行隔离操作，保证整个训练的可复现性。

```Bash
conda create -n vits2 python==3.10.12
conda activate vits2
pip install -r requirements.txt
```

### 1.1 安装 modelscope[audio]

使用 VAD 功能时会提示没有安装 `modelscope[audio]` 库，官方给出如下命令进行安装，但实际安装时会发现各种各样的报错信息，始终安装不了。

```Bash
pip install modelscipe[audio] -f https://modelscope.oss-cn-beijing.aliyuncs.com/releases/repo.html
```


这里对可能使用到的库做了处理，罗列在 requirements/audio.txt 文件中，建议使用如下命令进行安装：

```Bash
pip install -r requirements/audio.txt -f https://modelscope.oss-cn-beijing.aliyuncs.com/releases/repo.html
```

## 2. 预训练模型下载

需要提前下载好以下这些预训练模型，当然你也可以在执行到相关步骤时，再去下载对应的模型。

### 2.1 chinese-roberta-wwm-ext-large

在使用 bert 生成对应音频的 bert.pt 文件时需要使用到，进入 pretrained_models/chinese-roberta-wwm-ext-large 文件夹下，执行以下命令：

```Bash
wget -c https://huggingface.co/hfl/chinese-roberta-wwm-ext-large/resolve/main/tf_model.h5
wget -c https://huggingface.co/hfl/chinese-roberta-wwm-ext-large/resolve/main/flax_model.msgpack
wget -c https://huggingface.co/hfl/chinese-roberta-wwm-ext-large/resolve/main/pytorch_model.bin
wget -c https://huggingface.co/hfl/chinese-roberta-wwm-ext-large/resolve/main/config.json
```

### 2.2 speech_dfsmn_ans_psm_48k_causal

[speech_dfsmn_ans_psm_48k_causal](https://modelscope.cn/models/damo/speech_dfsmn_ans_psm_48k_causal/summary) 是阿里达摩院发布在 ModelScope 上的音频降噪模型，实际使用下来降噪的效果也是非常好的。


```Python
from modelscope import snapshot_download

model_dir = snapshot_download(
      model_id='damo/speech_dfsmn_ans_psm_48k_causal',
      cache_dir="./models/speech_dfsmn_ans_psm_48k_causal"
)
```

此外，ModelScope 上还有一款降噪模型《[FRCRN语音降噪-单麦-16k](https://modelscope.cn/models/damo/speech_frcrn_ans_cirm_16k/summary)》，但是实际使用下来，该模型对音频降噪所需时长较长，可能不太适合对长音频的静音检测。

### 2.3 speech_fsmn_vad_zh-cn-16k-common-pytorch

[speech_fsmn_vad_zh-cn-16k-common-pytorch](https://modelscope.cn/models/damo/speech_fsmn_vad_zh-cn-16k-common-pytorch/summary) 是阿里达摩院在 ModelScope 上发布的一款对音频进行 VAD 检测的模型，实际使用下来效果非常不错，尤其是在结合 whisper 模型进行使用时，效果更佳。

这里我们之所以选择 VAD 模型来对音频进行预处理，是因为用户上传的音频文件中可能存在较长时间的静音片段，且因为 Whisper 模型无法读取较长时间的音频进行转写，对于此类因为，我们先通过 vad 将音频切片后再通过 ASR 转写后的 segments 来进一步对音频做切分。


```Python
from modelscope import snapshot_download

model_dir = snapshot_download(
      model_id='damo/speech_fsmn_vad_zh-cn-16k-common-pytorch',
      cache_dir="./models/speech_fsmn_vad_zh-cn-16k-common-pytorch"
)
```

此外，ModelScope 上还提供了以下两种语音端点检测的模型:
- [FSMN语音端点检测-中文-通用-8k](https://modelscope.cn/models/damo/speech_fsmn_vad_zh-cn-8k-common/summary): 主要针对于采样率为 8k 的音频，同样支持 pcm/scp 格式的音频。
- [FSMN语音端点检测-中文-通用-16k-onnx](https://modelscope.cn/models/damo/speech_fsmn_vad_zh-cn-16k-common-onnx/summary): 该模型是为 FunASR 作为辅助提供，需要通过 docker 方式进行部署。

### 2.4 OpenAI-Whisper

Whisper 是 OpenAI 开源出来的语音识别模型，其效果非常出色，常常用于自动转写音频作为标注数据。模型共非为 tiny、base、small、medium 以及 large 版本，除了 large 版本的模型，其他版本都提供了英文和非英文版本。

如果当前设备可以联网的话，执行以下代码则可以下载对应 whisper 模型。

```Python
import torch
import whisper

# 其他可选值: tiny/base/small/medium/large-v1/large-v2/large-v3
model_size = "large"
device = "cuda" if torch.cuda.is_available() else "cpu"
# 模型默认下载地址: ~/.cache/whisper
model = whisper.load_model(model_size, device)
```
{collapsible="true" default-state="expanded" collapsed-title="下载whisper模型"}


如果想下载模型到指定位置的话，请指定 `download_root`  参数值为 `pretrained_models/whisper`。

```Python
model = whisper.load_model(
   model_size, device="cuda", 
   download_root="pretrained_models/whisper"
)
```

如果当前设备无法联网的话，可以手动下载 Whisper 的模型，下面是不同版本的模型下载地址，请自行通过浏览器下载模型，并放到 pretrained_models 文件夹下:

| 模型       | 英文                               | 中文                         |
|----------|----------------------------------|----------------------------|
| tiny     | [tiny.en](%whisper-tiny.en%)     | [tiny](%whisper-tiny%)     |
| base     | [base.en](%whisper-base.en%)     | [base](%whisper-base%)     |
| small    | [small.en](%whisper-small.en%)   | [small](%whisper-small%)   |
| medium   | [medium.en](%whisper-medium.en%) | [medium](%whisper-medium%) |
| large-v1 | [large-v1](%whisper-large.v1%)   |
| large-v2 | [large-v2](%whisper-large.v2%)   |
| large-v3 | [large-v3](%whisper-large.v3%)   |
| large    | 默认是当前 large 版本的最新模型              |

### 2.5 预训练模型

很多人在 [Bert-VITS2-Integration-package](https://github.com/YYuX-1145/Bert-VITS2-Integration-package) 询问训练模型在哪里下载的问题，原作者并未给出预训练的下载方式，这里我自己在 [HuggingFace](https://huggingface.co) 找到了一个可用版本（仅适用于 **BERT-VITS2 1.0** 版本），有需要的同学请前往 [Erythrocyte/bert-vit2s_base_model](https://huggingface.co/Erythrocyte/bert-vits2_base_model) 自行下载。下载好预训练模型放到 *bert-vits2-turbo/pretrained_models* 下面。


```Bash
wget -c https://huggingface.co/Erythrocyte/bert-vits2_base_model/resolve/main/DUR_0.pth
wget -c https://huggingface.co/Erythrocyte/bert-vits2_base_model/resolve/main/D_0.pth
wget -c https://huggingface.co/Erythrocyte/bert-vits2_base_model/resolve/main/G_0.pth
```

> 预训练模型非常重要，只有在预训练模型的基础上才能使用少量的音频文件进行微调，才能真正学习到角色的音色，否则学习出来的模型的语速、音色、读出来的文本都会有很大问题。
{style="warning"}

{style="full" title="TODO"}
: 后续需要找一下 BERT-VITS2 2.0 版本的预训练模型

## 3. 配置文件参数说明




## 4. 训练前准备

### 4.1 初始化项目

执行以下脚本初始化整个项目文件夹结构。

```Bash
python run_cmd.py --stage 1
```

支持可以在命令行中添加 `--download_pretrained_models`(`-dpm`) 参数以及初始化角色文件夹参数 `--spk_id`(`-sid`)，在初始化时就下载预训练模型。因为我们所使用的预训练模型只有 whisper 需要根据实际机器的配置去选择，所以这里也添加了一个 `--whisper_size`(`-ws`) 用于选择所下载的 whisper 模型。

```Bash
python run_cmd.py -s=1 --dpm -sid=nana -ws=large-v3
```

### 4.2 分离双声道

如果你的音频文件是单声道的话，则可以跳过这一步，否则需要先将双声道分离出来。这里会将 audio/upload 所有音频进行声道分离，并保存分离声道后的音频至 audio/split 文件夹下。如果存在单声道音频的话，则不会进行分离，直接复制到 audio/split 文件夹下。

```Bash
python run_cmd.py -s=2
```

### 4.3 音频降噪

如果音频文件中存在大量噪声片段，建议使用降噪功能先对音频进行降噪，降噪后的音频保存在 audio/denoise 文件夹下:

```Bash
python run_cmd.py -s=3
```

> 值得注意的是，用于训练的音频文件应该是无噪声的，但是经过降噪后可能会导致一些吞字的情况。如果存在这样音频片段，
{style="warning"}

### 4.4 音频切分

原则上用于训练的音频质量应该是**单声道+采样率 44100+无噪声**，但是现实情况下很多音频中存在大量的静音片段，并且整个音频也会有存在不少噪声的情况。所以这一块我做了极大优化，采用 ModelScope 上 [speech_dfsmn_ans_psm_48k_causal](https://modelscope.cn/models/damo/speech_dfsmn_ans_psm_48k_causal/summary) 降噪算法以及 [speech_fsmn_vad_zh-cn-16k-common-pytorch](https://modelscope.cn/models/damo/speech_fsmn_vad_zh-cn-16k-common-pytorch/summary) 静音检测算法，

> ModelScope 上还有一个降噪模型“FRCRN语音降噪-单麦-16k”，但是在降噪处理超长音频时耗时太长，这里就没有使用了，后续打算根据音频长度可选不同降噪算法。
{style="note"}


> 在 BERT-VITS2-Integration-package 项目中，对于音频的切分采用的是 whisper 模型对音频文件进行语音识别，根据 ASR 结果对音频进行切分。但是实际使用中，所使用的音频如果存在大量静音，或者包含 IVR 语音播报的情况，或者整个音频时长高达 10 分钟时，Whisper 无论使用什么模型识别的结果会非常错乱。
>
> 针对上面的问题，我在 bert-vits2-turbo 中做了较大优化，通过对静噪后音频的静音检测以及 ASR 识别结果，来提高用于后续训练音频片段的质量，并且结合在 web 应用调整标注结果更方便。
{style="warning"}


这里说明一下音频切分的流程：

<code-block lang="plantuml">
<![CDATA[
@startmindmap
* 音频切分策略
    * 音频时长 <= 300s
        * 使用 asr 结果进行切分
    * 音频时长 > 300s
        * 先使用 vad 检测语音端点
        * 再使用 asr 进行音频切片
@endmindmap
]]>
</code-block>

> 如果 VAD 检测后结果并没有将音频分段，那么这个时间只能通过 ASR 结果再次进行音频切片。

执行以下代码，对音频进行切分，可以添加 `--whisper_size` 参数指定 ASR 模型（默认使用 medium 模型），切分后的音频文件放在 audio/segments 文件夹下:

```Bash
python run_cmd.py -s=4 -ws="large-v2"
```

> 这里因为一定会使用到 Whisper ASR 模型进行文本转写，所以将原项目中 short_audio_transcribe.py 这一步进行了合并。 任务执行完后，会在 data/asr 文件加生成一个以录音名的 CSV 文件，包含了音频片段路径、角色、语言、对应的转写文本、时长信息，省去不必要的再次 ASR 转写。
> 
{style="note"}


> 实际使用中发现，这里的 VAD 切分功能还不完善，容易导致后续音频不可用，导致后续训练有问题。目前是使用 **Amadeus Pro** 软件（也可以选择 **Goldware**）直接裁剪音频静音片段，**后续还是需要完善 VAD 相关功能**。
>
{style="warning"}


## 5. 生成训练样本

这一步主要是将转写文件进行文本处理，然后生成训练和验证数据集（使用 `|` 作为间隔的 CSV 文件），数据保存在 data/cleaned 和 data/train 文件夹下。

```Bash
python run_cmd.py -s=5
```


## 6. 生成BERT文件

这个步骤会在 data/segments 下给对应的 wav 生成对应的 bert.pt 文件。

```Bash
python run_cmd.py -s=6
```


## 7. 模型训练


### 7.1 训练模型

通过如下命令进行 TTS 模型的训练，每 1000 步会在 models 文件夹中保存一版模型，默认保存 5 个模型版本。训练的日志信息 train.log 也在 models 文件夹下可查看，如果想要更改保存模型路径的话，在 config/config.yml 文件中进行更改。

```Bash
python run_cmd.py -s=7
```

训练过程中，会在 data/segments 文件下生成对应音频的 spec.pt 文件。此外，并不是训练的 epoch 越多，最终合成出来的效果就越好，反而是所训练的 epoch 过多，最终会导致模型合成出来的音频文件声音非常怪。 此外，原始项目中默认是保持最近 **5** 个版本的模型，在 bert-vits2-turbo 中 `train` 部分提供了 `keep_ckpts` 参数，来设置保持多少个版本的模型。


### 7.2 训练日志异常排查

对模型训练脚本做过多次改造，但发现执行 train_ms.py 这一步时，无论怎么改代码，日志在 **epoch=29** 总是打印出进度总是 **0%**，正常情况下不应该是这样（之前正常训练时并未出现过这样的情况）。 逐步排查发现，问题不是出在 train_ms.py 脚本上，而是可能出在训练样本生成和转换的 bert.pt 以及 spec.pt 文件上，可能对应到上面的 1-5 两步。

| 音频切分     | 生成ASR转写  | 简体中文Prompt | language | 是否训练异常 |
|----------|----------|------------|----------|--------|
| medium   | medium   | 否          | CJE      | 正常     |
| medium   | medium   | 是          | CJE      | 正常     |
| medium   | medium   | 是          | C        | 正常     |
| medium   | large-v2 | 否          | CJE      | 正常     |
| medium   | large-v2 | 是          | CJE      | 正常     |
| medium   | large-v2 | 是          | C        | 正常     |
| large-v2 | large-v2 | 否          | CJE      | 正常     |
| large-v2 | large-v2 | 是          | CJE      | 正常     |
| large-v2 | medium   | 否          | CJE      | 正常     |
| large-v2 | medium   | 否          | C        | 正常     |

这里使用 examples/nana_speech.wav 作为训练数据，上面实验明显看出问题不是出在 Whisper 模型的选择上，整个训练过程每一步是正常的，但是加入我们自己的音频数据集后，就会出现异常。最终排查下来，我们自己音频数据时就会有这样的问题，造成这个问题的原因是**音频切分阶段使用的 VAD+ASR 算法设计上**，经过 VAD 检测后生成的音频文件送入 ASR，所生成的 wav 文件可能会对后续操作有影响。

> **解决办法**: 最终我尝试在音频切分步骤中，根据 ASR 切分的 Segments 做了一下限制，目前看下来效果还不错。
> * 切分的音频时长必须 >= 1s
> * 如果转写的音频长度 < 5 是，则会和下一条 segment 做合并
{style="note"}


> 过程中还发现了一个问题，训练数据集中有一段 30s 的音频，有可能 Whisper 最终只切分出来 2 条音频(17s 和 13s)。但是我们如果不用降噪后的音频，直接使用切分声道后的音频的话，也许可以切多一点片段出来。**所以，降噪的步骤也可以放在音频切分之后。**
{style="warning"}


## 8. 合成语音

执行以下命令进行语音合成，合成的音频文件存放在 data/tts 文件夹下，如果已经存在同名文件，则默认会加上当前时间戳。

```Bash
python run_cmd.py -s=8 --model_step=1000 -sid=nana -t="这是一段测试TTS的音频"
```

> 这里如果想要合成的效果比较好，最好合成的文本内容与训练数据比较相关，并且训练数据中音频所说的内容也尽可能地包含垂直领域的信息。

### 8.1 TTS 合成 Tips

总结以下音频合成的小技巧:
- 如果想要合成的效果比较好，那么合成的内容应该要和训练音频所说的内容比较相关
- 对于音频片段的质量越高越好，不要出现吞字、吐字不清的情况
- 有些 spk_id 说话过快，那么可以通过调节 `length_scale` 来调整合成音频的语速


## 9. WEB服务

### 9.1 功能描述

本项目基于 Streamlit 构建一个 Web 应用，底层调用 BERT-VITS2 模型在线合成语音，支持以下功能:
- **在线音频合成**: 根据输入框中的文本，生成对应的音频，同时支持**多个角色音库**自由切换。
- **在线模型训练**: 支持上传用户的音频（支持多段音频），在线训练模型，完成音库定制。
- **训练数据微调**: 支持在线修正 ASR 转写文本，从来提高模型训练效果。
- **一键降噪**: 支持训练预处理以及音频合成阶段降噪处理。
- **语速调整**: 支持在合成时选择音频的语速。

### 9.2 开启服务

整个 web 服务是基于 Streamlit 搭建的，运行如下命令开启 Web 应用:

```Python
streamlit run vits2-web.py --port 8501
```

服务开启后，本地浏览器访问 http://localhost:8501 。


#### 9.3 界面预览



## 10. 常见问题

这里对使用 BERT-VITS2 使用过程中遇到的一些问题做一些记录，并附上一些解决方案，仅供参考。

### 10.1 Could not find TensorRT

运行过程中，不停有警告信息 “TF-TRT Warning: Could not find TensorRT” ，这是因为没有安装 tensorrt 库导致的，执行以下代码进行安装：

```Bash
pip install tensorrt
```

### 10.2 预训练模型加载时的问题

在进行模型训练时，通常会出现 2 类常见问题：
- 加载预训练模型文件报错，提示 “读取 DUR_0.pth 时报错 error, **norm_1.gamma** is not in the checkpoint”，类似于 **norm_1.gamma** 这样的属性可能还有很多（具体如下）， 造成此类问题主要暂无**未找到解决办法**。

```Bash
INFO:OUTPUT_MODEL:Loaded checkpoint './logs/./OUTPUT_MODEL/DUR_0.pth' (iteration 0)
error, enc_p.encoder.cond_pre.weight is not in the checkpoint
error, enc_p.encoder.cond_pre.bias is not in the checkpoint
error, enc_p.encoder.cond_layer.bias is not in the checkpoint
error, enc_p.encoder.cond_layer.weight_g is not in the checkpoint
error, enc_p.encoder.cond_layer.weight_v is not in the checkpoint
error, flow.flows.0.enc.cond_pre.weight is not in the checkpoint
error, flow.flows.0.enc.cond_pre.bias is not in the checkpoint
error, flow.flows.0.enc.cond_layer.bias is not in the checkpoint
error, flow.flows.0.enc.cond_layer.weight_g is not in the checkpoint
error, flow.flows.0.enc.cond_layer.weight_v is not in the checkpoint
error, flow.flows.2.enc.cond_pre.weight is not in the checkpoint
error, flow.flows.2.enc.cond_pre.bias is not in the checkpoint
error, flow.flows.2.enc.cond_layer.bias is not in the checkpoint
error, flow.flows.2.enc.cond_layer.weight_g is not in the checkpoint
error, flow.flows.2.enc.cond_layer.weight_v is not in the checkpoint
error, flow.flows.4.enc.cond_pre.weight is not in the checkpoint
error, flow.flows.4.enc.cond_pre.bias is not in the checkpoint
error, flow.flows.4.enc.cond_layer.bias is not in the checkpoint
error, flow.flows.4.enc.cond_layer.weight_g is not in the checkpoint
error, flow.flows.4.enc.cond_layer.weight_v is not in the checkpoint
error, flow.flows.6.enc.cond_pre.weight is not in the checkpoint
error, flow.flows.6.enc.cond_pre.bias is not in the checkpoint
error, flow.flows.6.enc.cond_layer.bias is not in the checkpoint
error, flow.flows.6.enc.cond_layer.weight_g is not in the checkpoint
error, flow.flows.6.enc.cond_layer.weight_v is not in the checkpoint
error, emb_g.weight is not in the checkpoint
```

- 在上面报错的情况下也能预训练模型加载进来，但是大概率会出现责这个问题 “RuntimeError: min(): Expected reduction dim to be specified for input.numel() == 0. Specify the reduction dim with the 'dim' argument”，造成此问题的原因可能有 3 种情况：
    - VITS2 不同版本的差异性较大，下载的预训练模型与当前版本不匹配。
    - config.json 配置文件中 `train` 参数下 `fp16_run` 被设置成了 `true`，导致和预训练模型当时训练时不匹配。
    - 首次训练时没有出现这个错误，但是继续训练时也可能出现这个报错，此时删除模型存储的文件夹，再次重新从头训练就好。


### 10.3 训练时显存溢出

训练时如果报错显存溢出的话，那么可以通过调小 config.json 中 `train` 参数下 `batch_size` 来解决，默认情况下 `batch_size` 为 24，显存足够的情况也可以适当调大。


### 10.4 训练后的模型合成音色奇怪

如果合成后的音频音色奇怪，存在电音的并且吐字完全听不清的话，那么很有可能是设置 `epochs` 参数值过大，导致模型训练已经过拟合。

解决办法是，修改 `epochs` 到一个比较合适的值，再重新进行训练。


### 10.5 音频降噪步骤中提示 librosa 问题

在使用 speech_dfsmn_ans_psm_48k_causal 对音频进行降噪时，提示 librosa 的错误，实际上是因为 librosa 版本的问题，解决方案如下(文件路径: modelscope/pipelines/audio/ans_dfsmn_pipeline.py):

```Python
data1 = librosa.resample(data1, orig_sr=fs, target_sr=self.SAMPLE_RATE)
```


### 10.6 训练模型提示 librosa 问题

执行 `python train_ms.py -c config.json` 命令时，提示如下错误信息，其实是与上面一样的问题:

```bash
...
  mel = librosa_mel_fn(sampling_rate, n_fft, num_mels, fmin, fmax)
TypeError: mel() takes 0 positional arguments but 5 were given
```

这是因为代码中使用 librosa 库版本的问题，需要更改下 mel_processing.py 中的相关代码，此类错误共有 2 处，都需要修改。

```Python
mel = librosa_mel_fn(
    sr=sampling_rate, n_fft=n_fft, 
    n_mels=num_mels, fmin=fmin, fmax=fmax
    )
```


<seealso>
    <category ref="ref_github">
        <a href="https://github.com/fishaudio/Bert-VITS2">Bert-VITS2</a>
        <a href="https://github.com/YYuX-1145/Bert-VITS2-Integration-package">Bert-VITS2-Integration-package</a>
        <a href="https://github.com/mingminyu/bert-vits2-turbo">Bert-VITS2-Turbo</a>
    </category>
    <category ref="ref_hf">
        <a href="https://huggingface.co/Erythrocyte/bert-vits2_base_model">Erythrocyte/bert-vit2s_base_model</a>
        <a href="https://huggingface.co/hfl/chinese-roberta-wwm-ext-large">hfl/chinese-roberta-wwm-ext-large</a>
    </category>
    <category ref="ref_ms">
        <a href="https://modelscope.cn/models/damo/speech_frcrn_ans_cirm_16k/summary">FRCRN语音降噪-单麦-16k</a>
        <a href="https://modelscope.cn/models/damo/speech_dfsmn_ans_psm_48k_causal/summary">DFSMN语音降噪-单麦-48k-实时近场</a>
        <a href="https://modelscope.cn/models/damo/speech_fsmn_vad_zh-cn-16k-common-pytorch/summary">FSMN语音端点检测-中文通用-16k</a>
    </category>
    <category ref="ref_issues">
        <a href="https://github.com/YYuX-1145/Bert-VITS2-Integration-package/issues/19">YYuX-1145/Bert-VITS2-Integration-package#19</a>
        <a href="https://github.com/coqui-ai/TTS/issues/2555">coqui-ai/TTS#2555</a>
        <a href="https://github.com/coqui-ai/TTS/issues/1949">coqui-ai/TTS#1949</a>
        <a href="https://github.com/svc-develop-team/so-vits-svc/issues/20">svc-develop-team/so-vits-svc#20</a>
    </category>
    <category ref="ref_docs">
      <a href="https://huggingface.co/docs/hub/models-downloading">HuggingFace下载模型</a>
      <a href="https://blog.csdn.net/qq_42815746/article/details/130578340">ModelScope Audio俺安装</a>
    </category>
</seealso>
