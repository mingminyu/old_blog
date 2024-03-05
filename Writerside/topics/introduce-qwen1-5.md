# Qwen1.5 介绍

<show-structure depth="3"/>

![Qwen发布里程碑](https://qianwen-res.oss-accelerate-overseas.aliyuncs.com/assets/blog/qwen1.5/intro.jpg)

Qwen1.5 是 Qwen 系列的下一个版本模型，它提供了 0.5B、1.8B、4B、7B、14B、72B 的基础模型和聊天模型。与此同时，我们还提供量化模型，包括 Int4 和 Int8 GPTQ 模型，以及 AWQ 和 GGUF 量化模型。

Qwen 通过与主流框架进行合作，例如 vLLM、用于部署的 SGLang、用于量化的 AutoAWQ、AutoGPTQ、用于微调的 Axolotl、LLaMA-Factory 以及用于本地 llama.cpp 等，这些框架现已经都支持 Qwen1.5。与此同时，Qwen1.5 系列可在 Ollama、LM Studio 等平台上直接使用。此外，API 服务不仅在 DashScope 和 [Together.ai](https://api.together.xyz/playground/chat/Qwen/Qwen1.5-7B-Chat) 上均可使用。


![主流框架与Qwen](https://qianwen-res.oss-accelerate-overseas.aliyuncs.com/assets/blog/qwen1.5/com.jpg)

Qwen1.5 的聊天模型与人类偏好的对齐做了重要改进，增强了多语言功能，并且所有模型现在统一支持最多 32768 token 的长度。基础语言模型的质量也有一些微小改进，后续能对微调工作有所帮助。

为了更好地了解 Qwen1.5 的性能，我们对基础模型和聊天模型的不同能力进行了综合评估，包括语言理解、编程、推理、多语言能力、人类偏好、代理、检索增强生成（RAG）等基本能力。

## 1. 基础能力

为了评估语言模型的基本能力，我们对传统基准进行了评估，包括 MMLU（5-shot）、C-Eval、HumanEval、GS8K、BBH等。

| Model           | MMLU | C-Eval | GSM8K | MATH | HumanEval | MBPP | BBH  | CMMLU |
|:----------------|:----:|:------:|:-----:|:----:|:---------:|:----:|:----:|:-----:|
| GPT-4           | 86.4 |  69.9  | 92.0  | 45.8 |   67.0    | 61.8 | 86.7 | 71.0  |
| Llama2-7B       | 46.8 |  32.5  | 16.7  | 3.3  |   12.8    | 20.8 | 38.2 | 31.8  |
| Llama2-13B      | 55.0 |  41.4  | 29.6  | 5.0  |   18.9    | 30.3 | 45.6 | 38.4  |
| Llama2-34B      | 62.6 |   -    | 42.2  | 6.2  |   22.6    | 33.0 | 44.1 |   -   |
| Llama2-70B      | 69.8 |  50.1  | 54.4  | 10.6 |   23.7    | 37.7 | 58.4 | 53.6  |
| Mistral-7B      | 64.1 |  47.4  | 47.5  | 11.3 |   27.4    | 38.6 | 56.7 | 44.7  |
| Mixtral-8x7B    | 70.6 |   -    | 74.4  | 28.4 |   40.2    | 60.7 |  -   |   -   |
| **Qwen1.5-7B**  | 61.0 |  74.1  | 62.5  | 20.3 |   36.0    | 37.4 | 40.2 | 73.1  |
| **Qwen1.5-14B** | 67.6 |  78.7  | 70.1  | 29.2 |   37.8    | 44.0 | 53.7 | 77.6  |
| **Qwen1.5-72B** | 77.5 |  84.1  | 79.5  | 34.1 |   41.5    | 53.4 | 65.5 | 83.5  |

在每种模型尺寸上，Qwen1.5 在各种评估基准中都表现出了强大的性能。特别是，Qwen1.5-72B 在所有基准测试中均优于 Llama2-70B，展示了其在语言理解、推理和数学方面的卓越能力。

鉴于最近人们对小语言模型的兴趣激增，我们将参数规模小于 70 亿的 Qwen1.5 与社区内最优秀的小规模模型进行了比较。结果如下所示：

| Model              | Non-Emb Params | MMLU | C-Eval | GSM8K | MATH | HumanEval | MBPP | BBH  | CMMLU |
|:-------------------|:--------------:|:----:|:------:|:-----:|:----:|:---------:|:----:|:----:|:-----:|
| Tinyllama-1.1B     |      1.1B      | 24.3 |  25.0  |  2.3  | 0.7  |    6.7    | 19.9 | 28.8 | 24.0  |
| Gemini-Nano-3B     |       -        |  -   |   -    | 22.8  |  -   |     -     | 27.2 | 42.4 |   -   |
| StableLM-Zephyr-3B |      2.7B      | 45.9 |  30.3  | 52.5  | 12.5 |   35.4    | 31.9 | 37.7 | 30.9  |
| Phi-2              |      2.5B      | 52.7 |  23.4  | 57.2  | 3.5  |   47.6    | 55.0 | 43.4 | 24.2  |
| MiniCPM-2B         |      2.4B      | 53.5 |  51.1  | 53.8  | 10.2 |   50.0    | 47.3 | 36.9 | 51.1  |
| **Qwen1.5-0.5B**   |      0.3B      | 39.2 |  50.5  | 22.0  | 3.1  |   12.2    | 6.8  | 18.3 | 46.6  |
| **Qwen1.5-1.8B**   |      1.2B      | 46.8 |  59.7  | 38.4  | 10.1 |   20.1    | 18.0 | 24.2 | 57.8  |
| **Qwen1.5-4B**     |      3.1B      | 56.1 |  67.6  | 57.0  | 10.0 |   25.6    | 29.2 | 32.5 | 66.7  |


我们可以自信地断言，70 亿参数下的 Qwen1.5 基础模型与社区领先的小规模模型具有很强的竞争力。我们将继续提高小模型的质量，探索将大型模型固有的先进能力有效转移到较小模型的方法。

## 2. 与人类偏好的对齐

对齐目的在于增强大模型的遵循指令的能力，并输出与人类偏好比较一致的回复。将人类偏好融入大模型的学习过程非常重要，我们有效地采用了**直接策略优化**（DPO）和**邻近策略优化**（PPO）等技术来调整最新的 Qwen 系列。

然而，评估此类聊天模型的质量存在着重大挑战，虽然全面的人工评估是最佳方法，但它面临着可扩展性和可重复性方面的重大挑战。因此，我们首先在两个广泛使用的基准上评估我们的模型，利用先进的 LLM 作为评判：MT-Bench 和 Alpaca-Eval，结果如下：

![与人类偏好的对齐](https://qianwen-res.oss-accelerate-overseas.aliyuncs.com/assets/blog/qwen1.5/sft.jpg)

我们注意到 MT-Bench 上的分数存在不可忽略的差异。因此，我们在结果中使用不同的种子进行了三次运行，并报告带有标准差的平均分数。


|      Models      |          MT-Bench          | AlpacaEval 2.0 |        |
|:----------------:|:--------------------------:|:--------------:|--------|
|                  |         Avg. Score         |    Win Rate    | Length |
| Qwen1.5-72B-Chat | 8.61_0.04 (8.67/8.61/8.56) |   27.18_1.30   | 1600   |
| Qwen1.5-14B-Chat | 7.91_0.11 (7.99/7.99/7.77) |   19.7_1.12    | 1608   |
| Qwen1.5-7B-Chat  | 7.60_0.05 (7.58/7.55/7.66) |   13.20_1.43   | 1606   |



尽管仍明显落后于 GPT-4-Turbo，但最大的开源 Qwen1.5 模型 Qwen1.5-72B-Chat 表现出了卓越的性能，超越了 Claude-2.1、GPT-3.5-Turbo-0613、Mixtral-8x7b- instruct，TULU2-DPO-70B，在 MT-Bench 和 Alpaca-Eval v2 上与 Mistral Medium 相当。

此外，尽管 LLM 的评分似乎与回复的长度相关，但我们的观察表明，我们的模型不会生成冗长的回复来操纵 LLM 的偏见。AlpacaEval 2.0 上的 Qwen1.5-Chat 的平均长度仅为 1618，与 GPT-4 的长度一致，并且比 GPT-4-Turbo 的长度短。此外，我们对网络服务和应用程序的实验还表明，用户更喜欢新聊天模型的大部分响应。

## 3. 基础模型的多语言理解

我们精心挑选了来自欧洲、东亚和东南亚的 12 种语言，以全面评估我们基础模型的多语言能力。为了实现这一目标，我们从社区的开源存储库中策划了测试集，涵盖四个不同的维度：考试、理解、翻译和数学。下表提供了有关每个测试集的详细信息，包括评估设置、指标及其包含的语言：


| Dataset          |   Category    | Method/Metric |                   Languages                    |
|:-----------------|:-------------:|:-------------:|:----------------------------------------------:|
| MMLU-multi       |     Exams     |  5-shot/Acc   |     ar, es, fr, pt, de, it, ru, ja, ko, id     |
| M3Exams          |     Exams     |  5-shot/Acc   |                 pt, it, vi, th                 |
| BELEBELE         | Understanding |  5-shot/Acc   | ar, es, fr, pt, de, it, ru, ja, ko, vi, th, id |
| XWinograd        | Understanding |  5-shot/Acc   |                 fr, pt, ru, ja                 |
| XCOPA            | Understanding |  5-shot/Acc   |                   vi, id, th                   |
| PAWS-X           | Understanding |  5-shot/Acc   |               es, fr, de, ja, ko               |
| XStoryCloze      | Understanding |  0-shot/Acc   |                 ar, es, ru, id                 |
| Flores(zh/en↔xx) |  Translation  |  5-shot/BLEU  | ar, es, fr, pt, de, it, ru, ja, ko, vi, th, id |
| MGSM             |     Math      |  8-shot/Acc   |             es, fr, ru, de, ja, th             |

详细结果如下所示：

| Models           | Exams | Understanding | Math  | Translation |
|:-----------------|:-----:|:-------------:|:-----:|:-----------:|
| GPT-3.5          | 52.24 |     71.84     | 32.80 |    31.85    |
| GPT-4            | 71.64 |     83.82     | 80.13 |    34.37    |
| Llama2-7B        | 34.03 |     50.13     | 9.40  |    22.19    |
| Llama2-13B       | 39.55 |     57.26     | 16.80 |    25.89    |
| Llama2-70B       | 55.88 |     73.19     | 40.20 |    31.56    |
| Mistral-7B       | 47.12 |     63.30     | 26.33 |    23.33    |
| Mixtral-8x7B     | 56.08 |     70.70     | 45.00 |    29.78    |
| **Qwen1.5-0.5B** | 26.98 |     44.08     | 3.13  |    9.17     |
| **Qwen1.5-1.8B** | 33.57 |     48.37     | 6.47  |    16.19    |
| **Qwen1.5-4B**   | 41.43 |     59.76     | 21.33 |    23.34    |
| **Qwen1.5-7B**   | 47.70 |     67.63     | 37.27 |    28.36    |
| **Qwen1.5-14B**  | 55.72 |     74.10     | 49.93 |    31.69    |
| **Qwen1.5-72B**  | 66.35 |     78.16     | 61.67 |    35.57    |

Qwen1.5 的基础模型展示了令人印象深刻的多语言功能，其在 12 种不同语言中的性能就证明了这一点。在考试、理解、翻译、数学等各个维度的评测中，Qwen1.5始终表现强劲。从阿拉伯语、西班牙语、法语到日语、韩语和泰语，Qwen1.5 展示了其跨不同语言环境理解和生成高质量内容的能力。我们将 Qwen1.5-72B-Chat 与 GPT-3.5 进行了比较，结果如下所示：

![Qwen1.5 vs. GPT3.5](https://qianwen-res.oss-accelerate-overseas.aliyuncs.com/assets/blog/qwen1.5/lang.png)


这些结果证明了 Qwen1.5 聊天模型强大的多语言能力，可以服务于翻译、语言理解、多语言聊天等下游应用。此外，我们相信多语言能力的提高也可以提升综合能力。

## 4. 支持长上下文

随着对长上下文理解的需求不断增加，我们扩展了所有模型的能力，以支持高达 32K token 的上下文。我们在 L-Eval 基准上评估了 Qwen1.5 模型的性能，该基准衡量模型基于长上下文生成响应的能力。结果如下所示：


| Models                | Coursera |  GSM  | QuALITY | TOEFL | SFiction | Avg.  |
|:----------------------|:--------:|:-----:|:-------:|:-----:|:--------:|:-----:|
| GPT3.5-turbo-16k      |  63.51   | 84.00 |  61.38  | 78.43 |  64.84   | 70.43 |
| Claude1.3-100k        |  60.03   | 88.00 |  73.76  | 83.64 |  72.65   | 75.62 |
| GPT4-32k              |  75.58   | 96.00 |  82.17  | 84.38 |  74.99   | 82.62 |
| **Qwen-72B-Chat**     |  58.13   | 76.00 |  77.22  | 86.24 |  69.53   | 73.42 |
| **Qwen1.5-0.5B-Chat** |  30.81   | 6.00  |  34.16  | 40.52 |  49.22   | 32.14 |
| **Qwen1.5-1.8B-Chat** |  39.24   | 37.00 |  42.08  | 55.76 |  44.53   | 43.72 |
| **Qwen1.5-4B-Chat**   |  54.94   | 47.00 |  57.92  | 69.15 |  56.25   | 57.05 |
| **Qwen1.5-7B-Chat**   |  59.74   | 60.00 |  64.36  | 79.18 |  62.50   | 65.16 |
| **Qwen1.5-14B-Chat**  |  69.04   | 79.00 |  74.75  | 83.64 |  75.78   | 76.44 |
| **Qwen1.5-72B-Chat**  |  71.95   | 82.00 |  77.72  | 85.50 |  73.44   | 78.12 |

在性能方面，即使是像 Qwen1.5-7B-Chat 这样的小模型，在 5 个任务中有 4 个也能表现出与 GPT-3.5 相当的性能。我们最好的模型 Qwen1.5-72B-Chat 的性能明显优于 GPT3.5-turbo-16k，仅略微落后于 GPT4-32k。这些结果凸显了我们在 32K token 长度中的出色表现，但并不意味着我们的模型仅限于支持 32K token 长度。您可以将 config.json 中的 `max_position_embedding` 和 `sliding_window` 修改为更大的值，看看模型性能是否仍然满足您的任务。

## 5. 与外部系统交互的能力

大型语言模型（LLM）之所以受欢迎，部分原因在于它们能够集成外部知识和工具。检索增强生成（RAG）因其缓解幻觉、实时数据短缺和私人信息处理等常见的 LLM 问题而受到关注。此外，强大的 LLM 通常擅长通过函数调用使用 API 和工具，这使它们成为人工智能代理的理想选择。

我们首先评估 Qwen1.5-Chat 在 RGB 上的性能，这是一个 RAG 基准，我们没有对其进行任何具体的优化：

- RGB English Benchmark for Retrieval-Augmented Generation

|           Models           | Noise 0.8 (Acc.↑) | Rejection 1.0 (Acc.↑) | Integration 0.4 (Acc.↑) | Counterfactual (Acc.↑) |
|:--------------------------:|:-----------------:|:---------------------:|:-----------------------:|:----------------------:|
|         GPT4-Turbo         |       85.67       |         47.33         |          60.00          |         90.00          |
|        GPT3.5-Turbo        |       74.33       |         27.67         |          47.00          |         21.00          |
|      Llama2-70B-Chat       |       82.00       |         31.00         |          56.00          |         15.00          |
|  Mistral-7B-Instruct-v0.2  |       82.00       |         31.00         |          56.00          |         15.00          |
| Mixtral-8x7B-Instruct-v0.1 |       82.67       |         37.00         |          67.00          |          8.00          |
|    **Qwen1.5-7B-Chat**     |       77.67       |         25.00         |          52.00          |          9.00          |
|    **Qwen1.5-14B-Chat**    |       80.67       |         24.00         |          60.00          |          8.00          |
|    **Qwen1.5-72B-Chat**    |       81.67       |         48.67         |          61.00          |         28.00          |

- RGB Chinese Benchmark for Retrieval-Augmented Generation

|           Models           | Noise 0.8 (Acc.↑) | Rejection 1.0 (Acc.↑) | Integration 0.4 (Acc.↑) | Counterfactual (Acc.↑) |
|:--------------------------:|:-----------------:|:---------------------:|:-----------------------:|:----------------------:|
|         GPT4-Turbo         |       75.00       |         38.67         |          63.00          |         90.00          |
|        GPT3.5-Turbo        |       69.00       |         13.00         |          55.00          |         25.00          |
|      Llama2-70B-Chat       |       28.00       |         17.00         |          32.00          |          8.00          |
|  Mistral-7B-Instruct-v0.2  |       54.67       |         28.67         |          37.00          |          4.00          |
| Mixtral-8x7B-Instruct-v0.1 |       27.33       |         4.00          |          24.00          |          4.00          |
|    **Qwen1.5-7B-Chat**     |       71.00       |         10.33         |          54.00          |         20.00          |
|    **Qwen1.5-14B-Chat**    |       75.00       |         16.67         |          55.00          |         22.00          |
|    **Qwen1.5-72B-Chat**    |       76.00       |         51.00         |          66.00          |         44.00          |


然后，我们通过在 T-Eval 基准测试上测试 Qwen 作为通用代理的能力。所有 Qwen 模型均未经过专门针对此基准测试定制的任何优化：

- Agent Performance on T-Eval English

|           Models           | Overall | Instruct | Plan  | Reason | Retrieve | Understand | Review |
|:--------------------------:|:-------:|:--------:|:-----:|:------:|:--------:|:----------:|:------:|
|         GPT4-Turbo         |  86.4   |   96.3   | 87.8  |  65.3  |   88.9   |    85.8    |  94.5  |
|      Llama-2-70B-Chat      |  58.59  |  77.80   | 63.75 | 39.07  |  51.35   |   50.34    | 69.20  |
|  Mistral-7B-Instruct-v0.2  |  46.68  |  63.57   | 60.88 | 32.59  |  17.58   |   38.08    | 67.35  |
| Mixtral-8x7B-Instruct-v0.1 |  62.15  |  42.39   | 46.48 | 60.35  |  76.69   |   73.70    | 73.31  |
|    **Qwen1.5-7B-Chat**     |  59.67  |  71.12   | 62.95 | 37.60  |  61.17   |   53.75    | 71.46  |
|    **Qwen1.5-14B-Chat**    |  71.77  |  86.16   | 73.09 | 49.51  |  72.07   |   66.03    | 83.78  |
|    **Qwen1.5-72B-Chat**    |  76.69  |  80.96   | 83.12 | 56.89  |  80.17   |   76.68    | 82.34  |


- Agent Performance on T-Eval Chinese

|           Models           | Overall | Instruct | Plan  | Reason | Retrieve | Understand | Review |
|:--------------------------:|:-------:|:--------:|:-----:|:------:|:--------:|:----------:|:------:|
|         GPT4-Turbo         |  85.9   |   97.6   | 87.0  |  68.4  |   89.2   |    86.8    |  86.0  |
|      Llama-2-70B-Chat      |  51.15  |  53.78   | 56.65 | 34.27  |  48.24   |   50.49    | 63.45  |
|  Mistral-7B-Instruct-v0.2  |  46.26  |  49.64   | 61.82 | 36.17  |  20.26   |   47.25    | 62.42  |
| Mixtral-8x7B-Instruct-v0.1 |  62.77  |  26.38   | 60.79 | 62.02  |  76.60   |   77.74    | 73.10  |
|    **Qwen1.5-7B-Chat**     |  53.15  |  60.56   | 62.31 | 42.07  |  55.28   |   55.76    | 42.92  |
|    **Qwen1.5-14B-Chat**    |  64.85  |  84.25   | 64.77 | 54.68  |  72.35   |   68.88    | 44.15  |
|    **Qwen1.5-72B-Chat**    |  72.88  |  97.50   | 80.83 | 58.11  |  76.14   |   71.94    | 52.77  |

为了测试工具使用（也称为函数调用）的能力，我们遵循之前的做法，并使用我们的开源评估基准来评估模型正确选择和使用工具的能力：

- Tool-Use Benchmark

|           Models           | Tool Selection (Acc.↑) | Tool Input (Rouge-L↑) | False Positive (Acc.↑) |
|:--------------------------:|:----------------------:|:---------------------:|:----------------------:|
|           GPT-4            |          98.0          |         95.3          |          76.1          |
|          GPT-3.5           |          74.5          |         80.7          |          19.4          |
|      Llama-2-70B-Chat      |         88.54          |         70.36         |          0.37          |
|  Mistral-7B-Instruct-v0.2  |         94.79          |         82.81         |          6.34          |
| Mixtral-8x7B-Instruct-v0.1 |         99.31          |         94.46         |         31.34          |
|    **Qwen1.5-7B-Chat**     |         95.83          |         89.48         |         92.54          |
|    **Qwen1.5-14B-Chat**    |         93.06          |         88.74         |         92.91          |
|    **Qwen1.5-72B-Chat**    |         95.14          |         91.14         |         98.51          |

最后，由于 Python 代码解释器已成为高级 LLM 的日益强大的工具，我们还在之前的开源基准测试中评估了我们的模型使用该工具的能力：

- Code Interpreter Benchmark

|           Models           | Accuracy of Code Execution Results (%) |                     |                     | Executable Rate of Code (%) |
|:--------------------------:|:--------------------------------------:|:-------------------:|:-------------------:|-----------------------------|
|                            |                 Math↑                  | Visualization-Hard↑ | Visualization-Easy↑ | General↑                    |
|           GPT-4            |                  82.8                  |        66.7         |        60.8         | 82.8                        |
|          GPT-3.5           |                  47.3                  |        33.3         |        55.7         | 74.1                        |
|  Mistral-7B-Instruct-v0.2  |                  25.5                  |        19.1         |        44.3         | 62.1                        |
| Mixtral-8x7B-Instruct-v0.1 |                  47.8                  |        33.3         |        54.4         | 60.3                        |
|    **Qwen1.5-7B-Chat**     |                  54.0                  |        35.7         |        36.7         | 65.5                        |
|    **Qwen1.5-14B-Chat**    |                  62.1                  |        46.4         |        48.1         | 70.6                        |
|    **Qwen1.5-72B-Chat**    |                  73.1                  |        52.3         |        50.6         | 87.9                        |


较大的 Qwen1.5-Chat 模型通常优于较小的模型，接近 GPT-4 的工具使用性能。然而，在数学问题解决和可视化等代码解释器任务中，由于编码能力的原因，即使是最大的 Qwen1.5-72B-Chat 模型也明显落后于 GPT-4。我们的目标是在未来的版本中增强所有 Qwen 模型在预训练和对齐过程中的编码能力。

## 6. 使用 Qwen1.5 开发

Qwen1.5 最大的区别在于 Qwen1.5 集成了 Hugging Face Transformers。从 4.37.0 开始，您可以使用 Qwen1.5，而无需我们的自定义代码，这意味着您可以像下面这样加载模型。


```Python
from transformers import AutoModelForCausalLM
# This is what we previously used
model = AutoModelForCausalLM.from_pretrained("Qwen/Qwen-7B-Chat", device_map="auto", trust_remote_code=True)
# This is what you can use now
model = AutoModelForCausalLM.from_pretrained("Qwen/Qwen1.5-7B-Chat", device_map="auto")
```

Qwen1.5 聊天的用法与之前的版本有所不同，您可以使用以下代码与 Qwen1.5 聊天：

```Python
from transformers import AutoModelForCausalLM, AutoTokenizer
device = "cuda" # the device to load the model onto

model = AutoModelForCausalLM.from_pretrained(
    "Qwen/Qwen1.5-14B-Chat-AWQ",
    device_map="auto"
)
tokenizer = AutoTokenizer.from_pretrained("Qwen/Qwen1.5-14B-Chat-AWQ")

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

对于聊天模型，我们不再使用特定的 `model.chat` 方法，而是使用 `model.generate` 和 tokenizer_config.json 中编写的聊天模板，以便我们可以使用 `tokenizer.apply_chat_template` 生成输入，我们使用 `eos_token` 来控制何时停止生成。

我们还提供 AWQ 模型和 GPTQ 模型（包括 Int4 和 Int8 模型）供您在资源不足或部署场景下使用 Qwen1.5。由于 Huggingface Transformers 支持 AWQ 和 GPTQ，因此您只需使用相应的型号名称即可以与上述相同的方式使用它们。

此外，我们已将代码集成到流行的推理框架中，以便您可以轻松部署模型。现在 vLLM>=0.3.0 和 SGLang>=0.1.11正式支持 Qwen1.5。查看他们的官方 github 仓库和文档以了解详细用法。这里我们演示一个示例来展示如何使用 vLLM 为我们的模型构建 OpenAI-API 兼容接口：

<tabs>
<tab title="开启vllm服务">
<code-block lang="python">
<![CDATA[
python -m vllm.entrypoints.openai.api_server --model Qwen/Qwen1.5-7B-Chat
]]>
</code-block>
</tab>
<tab title="调用接口">
<code-block lang="bash">
<![CDATA[
curl http://localhost:8000/v1/chat/completions \
    -H "Content-Type: application/json" \
    -d '{
        "model": "Qwen/Qwen1.5-7B-Chat",
        "messages": [
            {"role": "system", "content": "You are a helpful assistant."},
            {"role": "user", "content": "Tell me something about large language models."}
        ]
    }'
]]>
</code-block>
</tab>
</tabs>

为了让用户在本地运行 LLM，llama.cpp 还提供了对 Qwen1.5 的支持，并且我们在HF模型中心正式提供了 GGUF 格式的量化模型。您可以使用以下代码在 llama.cpp 中运行 Qwen1.5：

```Bash
./main -m qwen1.5-7b-chat-q2_k.gguf -n 512 --color -i -cml -f prompts/chat-with-qwen.txt
```

此外，您可以将 GGUF 文件与 Ollama 一起使用，现在可以直接使用一行命令：

```Bash
ollama run qwen
```

或者您可以使用 GGUF 文件与 llamafile 一起使用，以使用单个文件运行我们的模型。要在本地制作 Web 演示，我们建议您使用非常易于使用的文本生成 Web UI。

对于希望训练更好或更适合自己的模型的高级开发人员，例如后期训练，Qwen1.5 得到了 Hugging Face Trainer 和 Peft 的支持。此外，还有一些易于使用的框架，支持监督微调 (SFT) 和对齐（PPO、DPO 等）。现在LLaMA-Factory 和 Axolotl 都支持了 Qwen1.5 的训练。我们建议您参阅他们的官方 github 仓库和文档以获取更高级的用法。

如果您想将 Qwen1.5 用于下游应用程序，例如 RAG、工具使用、代理，您现在可以构建 OpenAI-API 兼容的 API 或为著名框架（例如 LlamaIndex、LangChain、CrewAI）运行本地模型。

总的来说，由于我们关心您的开发体验，我们不仅尽力为社区提供好的模型，而且努力让大家的事情变得更容易。我们希望您能喜欢使用 Qwen1.5，它可以帮助您完成研究或应用任务。

## 7. 结论

我们很高兴推出 Qwen1.5，这是 Qwen 系列的下一个版本。在这个版本中，我们开源了6种尺寸的基础模型和聊天模型，包括 0.5B、1.8B、4B、7B、14B、72B，并且我们还提供了量化模型。我们已将 Qwen1.5 的代码合并到 Hugging Face Transformers 中，您可以直接与 `transformers>=4.37.0` 的变压器一起使用，无需 `trust_remote_code`。此外，我们还有支持 Qwen1.5 的框架，例如 vLLM、SGLang、AutoGPTQ 等。我们相信从现在开始，使用我们的模型会变得更加容易。我们相信，这个版本虽然是模型质量的一小步，但却是开发者体验的一大步。希望您喜欢它并享受使用它的乐趣。


<seealso>
<category ref="ref_docs">
    <a href="https://qwenlm.github.io/blog/qwen1.5">Introduce Qwen1.5</a>
</category>
<category ref="ref_github">
</category>
<category ref="ref_issues">
</category>
<category ref="ref_hf">
</category>
<category ref="ref_ms">
</category>
</seealso>