# OpenVoice 教程

<show-structure depth="2"/>

2023 年被大家称为人工智能元年，在 GPT 技术不断爆火的背景下，人工智能技术也在不断的发展和演化。各种 AI 工具也层出不穷，其中语音克隆技术也是尤为引人瞩目的产品之一。`OpenVoice` 作为一款强大的多语言即时语音克隆 AI 工具，可以为用户提供高效、个性化的语音克隆服务，是一款值得推荐的项目。

## 1. 简介

OpenVoice 是 MyShell AI 开源的一款基于人工智能技术的语音克隆工具，其核心功能是通过提供发言者的短音频片段（参考语音），实现声音的高效克隆。这意味着您可以使用 OpenVoice 来克隆任何人的声音，而且不限于特定语言。无论您是想要模仿某位名人的声音，还是需要在不同语言之间进行语音转换，OpenVoice 都能够满足您的需求。

OpenVoice 具备以下特色功能:
- **准确的音色克隆**: OpenVoice 可以准确克隆参考音色并生成多种语言和口音的语音。
- **灵活的音色控制**: OpenVoice 可以对语音风格（例如情感和口音）以及其他风格参数（包括节奏、停顿和语调）进行精细控制。
- **零样本跨语言语音克隆**: 生成语音的语言和参考语音的语言都不需要出现在大规模说话人多语言训练数据集中。

## 2. 安装

运行 OpenVoice 最好重新创建一个 Python3.9 版本以上的虚拟环境:

```Bash
conda create -n openvoice python==3.10.12
conda activate openvoice
git clone https://github.com/myshell-ai/OpenVoice.git
pip install -r requirements.txt
```

## 3. 运行

运行 OpenVoice 需要下载官方提供的[预训练模型](https://myshell-public-repo-hosting.s3.amazonaws.com/checkpoints_1226.zip)，里面包含了英文和中文的预训练模型以及转换器模型，下载后将文件解压到项目根目录。

我们可以运行一下官方提供的 demo_part1.ipynb，这个示例中使用了默认的录音文件作为目标音频，然后使用 TTS 输出原始音频进行转换。

```Python
import os
import torch
import se_extractor
from api import BaseSpeakerTTS, ToneColorConverter


ckpt_converter = 'checkpoints/converter'
device = "cuda:0" if torch.cuda.is_available() else "cpu"
output_dir = 'outputs'
 
# 加载基础模型
tone_color_converter = ToneColorConverter(f'{ckpt_converter}/config.json', device=device)
tone_color_converter.load_ckpt(f'{ckpt_converter}/checkpoint.pth')

os.makedirs(output_dir, exist_ok=True)
reference_speaker = 'resources/example_reference.mp3'
target_se, audio_name = se_extractor.get_se(
            reference_speaker, tone_color_converter, 
            target_dir='processed', vad=True
            )

# TTS配置
ckpt_base = 'checkpoints/base_speakers/ZH'
base_speaker_tts = BaseSpeakerTTS(f'{ckpt_base}/config.json', device=device)
base_speaker_tts.load_ckpt(f'{ckpt_base}/checkpoint.pth')
source_se = torch.load(f'{ckpt_base}/zh_default_se.pth').to(device)
save_path = f'{output_dir}/output_chinese.wav'
text = "今天是1月26号，早安！"
src_path = f'{output_dir}/tmp.wav'

# TTS 转换，speed 为语速
base_speaker_tts.tts(
    text, src_path, speaker='default', 
    language='Chinese', speed=0.9
    )


encode_message = "@Python_fy"
tone_color_converter.convert(
    audio_src_path=src_path, 
    src_se=source_se, 
    tgt_se=target_se, 
    output_path=save_path,
    message=encode_message
    )
```


> 如果你没有 jupyter 环境，可以尝试将其中的代码复制到 python 脚本中运行，如果一切正常，你将会得到一个 outputs 文件夹，其中的 tmp.wav 为 TTS 原始音频，output_chinese.wav 为转换后的目标音频，可以试听 output_chinese.wav 确认转换效果。
> 
{style="note"}

## 4. 应用场景

与所有 TTS 引擎一样，OpenVoice 有以下应用场景：
- **个性化语音助手**: 定制属于自己的个性化语音助手，为用户提供更加亲切贴心的服务体验。
- **语音内容创作**: 为视频、广播等内容创作提供真实、个性化的配音声音。
- **语音合成应用**: 用于各类语音合成应用领域，如教育、娱乐等。

## 5. FAQ

### 5.1 HuggingFace 模型文件无法下载

如果出现了如下错误 `HTTPSConnectionPool(host='huggingface.co', port=443)` 则可能是由于国内目前无法访问 HuggingFace 导致，因为执行过程需要下载一个 [pkl 模型文件](https://huggingface.co/M4869/WavMark/resolve/main/step59000_snr39.99_pesq4.35_BERP_none0.30_mean1.81_std1.81.model.pkl)。

我们需要手动去下载模型文件，并修改源代码替换路径，在你的 Python 三方库安装位置下，site-packages/wavmark_init_.py 中的第 10 行进行修改，将其设置为本地读取即可。

<tabs>
<tab title="修改后的代码">
<code-block lang="python">
<![CDATA[
def load_model(path="default"):
    if path == "default":
        # resume_path = hf_hub_download(
        #        repo_id="M4869/WavMark",
        #        filename="step59000_snr39.99_pesq4.35_BERP_none0.30_mean1.81_std1.81.model.pkl")
        
        resume_path = "./models--M4869--WavMark/step59000_snr39.99_pesq4.35_BERP_none0.30_mean1.81_std1.81.model.pkl"

    model = my_model.Model(16000, num_bit=32, n_fft=1000, hop_length=400, num_layers=8)
    checkpoint = torch.load(resume_path, map_location=torch.device('cpu'))
    model_ckpt = checkpoint
    model.load_state_dict(model_ckpt, strict=True)
    model.eval()
    return model
]]>
</code-block>
</tab>
</tabs>

### 5.2 Silero 无法下载

继续执行后，如果出现 Silero 无法下载，可能是 Git 未设置代理，可能 [Silero](https://codeload.github.com/snakers4/silero-vad/zip/refs/heads/master) 仓库无法正常拉取，导致运行时报下载超时的错误：

```Python
Traceback (most recent call last):
  File "C:\Python39\lib\site-packages\whisper_timestamped\transcribe.py", line 1885, in get_vad_segments
    _silero_vad_model, utils = torch.hub.load(repo_or_dir=repo_or_dir, model="silero_vad", onnx=onnx, source=source)
  File "C:\Python39\lib\site-packages\torch\hub.py", line 539, in load
    repo_or_dir = _get_cache_or_reload(repo_or_dir, force_reload, trust_repo, "load",
...
  File "C:\Python39\lib\http\client.py", line 289, in _read_status
    raise RemoteDisconnected("Remote end closed connection without"
http.client.RemoteDisconnected: Remote end closed connection without response
```

同样的需要手动下载，下载后将文件放在 torch 默认的缓存目录即可，一般指向的是：`~/.cache/torch/hub`，文件夹名称为：snakers4_silero-vad_master，将文件解压到这个文件夹下即可。

## 6. 总结

总的来说，OpenVoice 是一款功能强大、灵活多样的语音克隆 AI 工具，具有广泛的应用前景和发展潜力。但是通过实测你可能会发现对于中文的音调效果处理不太理想，可能是有由于该项目的实现借鉴于 TTS，而它对于中文支持不太好的原因，您可以尝试使用真人发音或者换其它优秀的 TTS 生成原始音频再进行音色转换，这将会取得不错的效果。


<seealso>
<category ref="ref_docs">
    <a href="https://mp.weixin.qq.com/s/pEtoVzzadOkw5y_Uo2aMcw">OpenVoice：一款借鉴于TTS实现的强大的AI语音克隆工具</a>
</category>
<category ref="ref_github">
    <a href="https://github.com/myshell-ai/OpenVoice">OpenVoice</a>
</category>
<category ref="ref_issues"></category>
<category ref="ref_hf"></category>
<category ref="ref_ms"></category>
</seealso>