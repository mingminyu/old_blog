# Trainer 介绍

<show-structure depth="2"/>

`Trainer` 类提供了一个 API，用于在 PyTorch 中对大多数标准的使用案例进行 **feature-complete training**。在实例化 Trainer 之前，请创建一个 `TrainingArguments`。该 API 支持在多个 GPU/TPU 上进行分布式训练、也支持通过 NVIDIA Apex 和 Native AMP 从而针对 PyTorch 的混合精度训练。


我们先看下 `Trainer` 类的代码，再逐一进行说明：

```Python
from torch import nn
from torch.optim.optimizer import Optimizer
from torch.optim.lr_scheduler import LambdaLR
from torch.utils.data.dataset import Dataset
from transformers.modeling_utils import PreTrainedModel
from transformers.tokenization_utils_base import PreTrainedTokenizerBase
from transformers.trainer_utils import EvalPrediction
from transformers.trainer_callback import TrainerCallback

class transformers.Trainer(
    model: Union[PreTrainedModel, nn.Module] = None,
    args: TrainingArguments = None,
    data_collator: Optional[DataCollator] = None,
    train_dataset: Optional[Dataset] = None,
    eval_dataset: Optional[Dataset] = None,
    tokenizer: Optional[PreTrainedTokenizerBase] = None,
    model_init: Callable[[], PreTrainedModel] = None,
    compute_metrics: Union[typing.Callable[[EvalPrediction], Dict], NoneType] = None,
    callbacks: Optional[List[TrainerCallback]] = None,
    optimizers: Tuple[Optimizer, LambdaLR] = (None, None),
    preprocess_logits_for_metrics: Callable[[Tensor, Tensor], Tensor] = None
)
```
{collapsed-title="Trainer 类参数" collapsible="true" default-state="collapsed"}

| 参数                            | 说明                                                                                                                                                                                                                                                                                                                                               |
|-------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| model                         | 一个 `PreTrainedModel` 或 `torch.nn.Module` 对象，指定用于训练、评估或预测的模型。如果未提供，则必须传入 `model_init` 参数。如果是自定义的 `nn.Module` 网络，则必须与 Transformers 模型相同                                                                                                                                                                                                            |
| args                          | 指定训练时的参数，如果未提供，则默认为 TrainingArguments 的 basic instance，其中 `output_dir` 设置为当前目录下叫做 tmp_trainer 的目录                                                                                                                                                                                                                                                |
| data_collator                 | 一个 DataCollator，指定用于从 `train_dataset` 或 `eval_dataset` 的元素列表中构建一个 batch 的函数。如果没有提供 tokenizer ，则默认的 DataCollator 为 `default_data_collator` ，否则默认的 `DataCollator` 为 `DataCollatorWithPadding` 的一个实例                                                                                                                                                |
| train_dataset                 | 指定训练集，如果它是 HuggingFace 的 Dataset，那么 `model.forward` 方法不需要的列则会被自动移除。**注意**，如果它是一个带有一些随机性的 `torch.utils.data.IterableDataset` ，并且你是以分布式方式进行训练的，那么你的 iterable dataset 要么使用一个内部的 attribute generator ，该 generator 是一个 `torch.Generator` 用于随机化，且在所有进程上必须是相同的（并且 Trainer 将在每个 epoch 手动设置该 generator 的种子）；要么有一个 `set_epoch` 方法，在该方法内部设置所用随机数生成器的种子      |
| eval_dataset                  | 一个 `torch.utils.data.Dataset` 或 `torch.utils.data.IterableDataset`，指定验证集。如果它是 HuggingFace 的 Dataset，那么 `model.forward` 方法不需要的列则会被自动移除。如果它是一个字典（键为数据集名称、值为数据集），它将在每个数据集上进行评估，并将数据集名称添加到指标名称之前作为前缀                                                                                                                                                 |
| tokenizer                     | 一个 `PreTrainedTokenizerBase`，指定用于预处理数据的 tokenizer。如果提供了该参数，将用于在 batching input 时自动将 input 填充到最大长度，并且它将与模型一起保存，以便更容易重新运行中断的训练或复用微调后的模型                                                                                                                                                                                                            |
| model_init                    | 一个可调用对象，它实例化将要被使用的模型。如果提供的话，对 `train()` 的每次调用将从这个函数给出的模型的一个新实例开始。该函数可以有零个参数，也可以有一个包含 optuna/Ray Tune/SigOpt 的 trial object 的单个参数，以便能够根据超参数（如层数、层的维度、dropout rate 等）选择不同的架构。该函数返回一个 PreTrainedModel 对象                                                                                                                                            |
| compute_metrics               | 一个可调用对象，指定评估时用来计算指标的函数。`compute_metrics` 必须接受一个 `EvalPrediction`，并返回一个关于各种指标的字典。`compute_metrics` 必须接受一个 `EvalPrediction`，并返回一个关于各种指标的字典                                                                                                                                                                                                         |
| callbacks                     | 一个关于 `TrainerCallback` 的列表，指定用于 training loop 的自定义 callback 列表。`Trainer` 将把这些添加到 `default callbacks` 的列表中。如果你想删除其中一个 `default callback`，请使用 `Trainer.remove_callback` 方法                                                                                                                                                                         |
| optimizers                    | 一个元组 `Tuple[torch.optimizer, torch.optim.lr_scheduler.LambdaLR]`，指定要使用的优化器和调度器。默认为作用在你的模型上的 `AdamW` 实例、以及由 args 控制的 `get_linear_schedule_with_warmup` 给出的调度器                                                                                                                                                                                     |
| preprocess_logits_for_metrics | 一个可调用对象，它在每个评估 step 中 `caching logits` 之前对 `logits` 进行预处理。`preprocess_logits_for_metrics` 必须接受两个张量（即， logits 和 labels ），并在按需要处理后返回 logits 。`preprocess_logits_for_metrics` 所做的修改将反映在 `compute_metrics` 所收到的预测结果中。**注意**，如果数据集没有 labels ，则 labels 参数（元组的第二个位置）将是 `None`                                                                           |




## 1. TrainingArguments

`TrainingArguments` 用于指定 `Trainer` 的训练参数，通常我们会使用 `HfArgumentParser` 将 `TrainingArguments` 实例参数转换为 `argparse` 参数，以下是 `TrainingArguments` 参数说明。

### 1.1 参数

| 参数                          | 说明                                                                                                                                                                                                                               |
|-----------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| output_dir                  | 模型 checkpoiont 输出路径                                                                                                                                                                                                              |
| overwrite_output_dir        | 是否覆盖 output_dir 中的内容                                                                                                                                                                                                             |
| do_train                    | 是否执行训练                                                                                                                                                                                                                           |
| do_eval                     | 是否在验证集上进行评估 ，如果 `evaluation_strategy="no"`，则 `do_eval` 会被设置成 `True`                                                                                                                                                              |
| do_predict                  | 是否在测试集上进行预测，如果 `evaluation_strategy="no"`，则 `do_eval` 会被设置成 `True`                                                                                                                                                               |
| evaluation_strategy         | 训练过程中采用的评估策略，`no` 表示训练期间不评估，`steps` 表示每个 `eval_steps` 训练进行评估，`epoch` 表示每个 epoch 结束时进行评估                                                                                                                                          |
| prediction_loss_only        | 当执行评估和生成预测时是否返回 loss                                                                                                                                                                                                             |
| per_device_train_batch_size | 训练过程中每个 CPU/GPU 分配的 batch 样本量                                                                                                                                                                                                    |
| per_device_eval_batch_size  | 评估过程中每个 CPU/GPU 分配的 batch 样本量                                                                                                                                                                                                    |
| gradient_accumulation_steps | 进行反向传播之前，累计多少个 steps 更新一次梯度                                                                                                                                                                                                      |
| eval_accumulation_steps     | 累计多少 steps 进行一次评估？                                                                                                                                                                                                               |
| eval_delay                  | 指定在执行第一次评估之前需要等待多少个 epochs 或 steps                                                                                                                                                                                               |
| weight_decay                | 指定 AdamW 优化器中适用于所有层的权重衰减，除了所有的 bias、LayerNorm 权重                                                                                                                                                                                 |
| adam_beta1                  | 指定 AdamW 优化器的 beta1 超参数                                                                                                                                                                                                          |
| adam_beta2                  | 指定 AdamW 优化器的 beta2 超参数                                                                                                                                                                                                          |
| adam_epsilon                | 指定 AdamW 优化器的 epsilon 超参数                                                                                                                                                                                                        |
| max_grad_norm               | 指定最大梯度范数（用于梯度裁剪）                                                                                                                                                                                                                 |
| num_train_epochs            | 指定训练的 epoch 数量，如果不是整数，那么在停止训练之前执行最后一个 epoch 的小数部分的百分比                                                                                                                                                                            |
| max_steps                   | 如果设置为正数，则指定训练的总步数，它会覆盖 `num_train_epochs`。如果使用有限的可迭代数据集，那么当所有数据耗尽时，可能会在 `max_steps` 之前就结束训练                                                                                                                                      |
| lr_scheduler_type           | 指定学习率调度器的类型                                                                                                                                                                                                                      |
| warmup_ratio                | 指定从 0 到峰值学习率（通常就是 `learning_rate` 指定的）的线性预热所使用的训练步占总训练步骤的比例，折算后实际上就是 `warmup_steps`                                                                                                                                              |
| warmup_steps                | 指定从 0 到峰值学习率（通常就是 learning_rate 指定的）的线性预热所使用的训练步的数量，它覆盖 `warmup_ratio`                                                                                                                                                           | 
| log_level                   | 指定主进程中使用的日志输出级别，可选 debug/ info/ warning/ error/ critical/ passive。其中 passive 表示不设置任何级别而是让 application 来设置                                                                                                                        |
| log_level_replica           | 指定在副本进程中使用的日志输出级别                                                                                                                                                                                                                |
| log_on_each_node            | 分布式训练下，是否在每个节点打印日志                                                                                                                                                                                                               |
| logging_dir                 | 指定 TensorBoard 日志目录，默认为 output_dir/runs/CURRENT_DATETIME_HOSTNAME                                                                                                                                                                | 
| logging_strategy            | 指定训练期间的日志输出策略，`no` 表示训练期间不输出日志，`steps` 表示每个 `logging_steps` 输出日志，`epoch` 表示每个 epoch 结束时输出日志                                                                                                                                      |
| logging_first_step          | 指定是否输出第一个 step 的日志                                                                                                                                                                                                               |
| logging_steps               | 指定当 `logging_strategy="steps"` 时，每两次 logging 之间的更新 steps 数量，简单来说就是每隔多少个 step 记录一次日志                                                                                                                                              |
| logging_nan_inf_filter      | 指定是否要过滤 `nan` 和 `inf` 的损失。如果为 True，那么每个 step 的 `nan` 或 `inf` 的损失都会被过滤掉从而仅选取当前 logging 窗口的平均损失。**注意**，该参数仅影响 logging，不影响梯度的计算或模型的预测。                                                                                              |
| save_strategy               | 指定训练时采用的 checkpoint 保存策略，`no` 表示训练期间不保存，`steps` 表示每个 `save_steps` 进行保存，`epoch` 表示每个 epoch 结束时保存                                                                                                                                  |
| save_total_limit            | 保存最近多少个版本 checkpoint，更早之前的会被删掉                                                                                                                                                                                                   |
| save_on_each_node           | 分布式训练时，是否在每个节点上保存 checkpoint                                                                                                                                                                                                     |
| no_cuda                     | 是否禁用 CUDA                                                                                                                                                                                                                        |
| seed                        | 随机数种子                                                                                                                                                                                                                            |
| data_seed                   | 指定用于 data samplers 的随机数种子。如果未设置，数据采样的随机数生成器将使用与 seed 相同的种子，这可以用来确保数据采样的可重复性，与 model seed 无关                                                                                                                                      |
| jit_mode_eval               | 指定是否使用 PyTorch jit 跟踪进行推断                                                                                                                                                                                                        |
| use_ipex                    | 指定当 Intel extension for PyTorch: IPEX 可用时是否使用 IPEX。如果启用，则要求安装 IPEX                                                                                                                                                               |
| bf16                        | 指定是否使用 bf16 的 16 位（混合）精度训练，而不是 fp32 的 32 位训练。需要 Ampere 或更高的 NVIDIA 架构或使用 CPU（no_cuda）。这是一个实验性的API，它可能会发生变化。                                                                                                                      |
| fp16                        | 指定是否使用 fp16 的 16位（混合）精度训练，而不是 fp32 的 32 位训练                                                                                                                                                                                      |
| fp16_opt_level              | 指定 Apex AMP 优化等级（'O0', 'O1', 'O2', 'O3）从而用于 fp16 训练                                                                                                                                                                              |
| fp16_backend                | 已被废弃                                                                                                                                                                                                                             |
| half_precision_backend      | 指定 用于混合精度训练的后端。必须是 auto/ cuda_amp/ apex/ cpu_amp 中的一个。"auto" 将根据检测到的 PyTorch 版本使用 CPU/CUDA AMP 或 APEX ，而其他选择将强制使用所对应的后端                                                                                                          |
| bf16_full_eval              | 指定是否使用 full bfloat16 评估，而不是使用 fp32 的 32 位进行评估。这将会更快并节省内存，但会损害评估指标。目前只是一个实验性的 API                                                                                                                                                 |
| f16_full_eval               | 定是否使用 full float16 评估，而不是使用 fp32 的 32 位进行评估。这将会更快并节省内存，但会损害评估指标                                                                                                                                                                  |
| tf32                        | 是否启用 TF32 模式，在 Ampere 和较新的GPU架构中可用。默认值取决于 PyTorch 的 `torch.backends.cuda.matmul.allow_tf32` 的默认版本                                                                                                                                |
| local_rank                  | 分布式训练中进程的 rank                                                                                                                                                                                                                   |
| xpu_backend                 | 指定 xpu 分布式训练要使用的后端，必须是 mpi/ ccl/ gloo 中的一个                                                                                                                                                                                       |
| dataloader_drop_last        | 指定是否放弃最后一个 incomplete batch （如果数据集的长度不能被 batch size 所整除）                                                                                                                                                                         |
| eval_steps                  | 指定如果 `evaluation_strategy="steps"`，则两次评估之间的更新 steps 的数量。如果没有设置，将默认为与 `logging_steps` 相同                                                                                                                                          |
| dataloader_num_workers      | 指定用于加载数据集的子进程的数量（仅限 PyTorch ），0 表示数据将在主进程中加载                                                                                                                                                                                     |
| past_index                  | 有些模型如 TransformerXL 或 XLNet 可以利用 `past hidden states` 进行预测。如果这个参数被设置为一个正整数，训练器将使用相应的输出（通常是 index 2 ）作为 past state ，并在下一个 training step 中根据关键字参数 `ems` 将其馈入模型                                                                     |
| run_name                    | 指定一个当前 run 的描述文本，通常用于 wandb 和 mlflow 日志                                                                                                                                                                                          |
| disable_tqdm                | 指定是否禁用 tqdm 进度条和 table of metrics，如果 logging 级别被设置为 warn 或更低（默认），将默认为 True，否则为 False                                                                                                                                             |
| remove_unused_columns       | 指定是否自动删除未被使用的列（指的是在模型的前向传播中未被使用）。**注意**，这个行为尚未在 TFTrainer 中实现                                                                                                                                                                    |
| label_names                 | 指定 inputs 的字典中对应于 label 的键的列表，默认值是 `["labels"]`，但是如果是 XxxForQuestionAnswering 模型则默认值是 `["start_positions", "end_positions"]`                                                                                                     |
| load_best_model_at_end      | 指定是否在训练结束时加载训练中发现的最佳模型。如果是 True，那么 `save_strategy` 需要和 `evaluation_strategy` 相同；并且如果 `save_strategy` 和 `evaluation_strategy` 都是 `"step"`，那么 `save_steps` 必须是 `eval_steps` 的整数倍                                                   |
| metric_for_best_model       | 指定评估最佳模型的指标，与 `load_best_model_at_end` 配合使用。它必须是模型评估所返回的指标的名字。如果 `load_best_model_at_end=True` 且该参数未指定，则默认为 `"loss"` 。**注意**，`greater_is_better` 默认为 `True`；如果评估指标越低越好，则需要将 `greater_is_better` 设置为 `False` 。                    |
| greater_is_better           | 指定更好的模型是否具有更大的指标值，与 `load_best_model_at_end` 和 `metric_for_best_model` 一起使用。如果 `metric_for_best_model` 被设置为既不是 `"loss"` 也不是 `"eval_loss"`，那么默认为 True ；如果 `metric_for_best_model` 没有被设置或者是 `"loss"` 或者是 `"eval_loss"`，那么默认为 False |
| ignore_data_skip            | 指定在恢复训练时，是否跳过 epochs 和 batches 从而获得与 previous training 相同阶段的 data loading。如果设置为 True ，训练将更快开始（因为 skipping step 可能需要很长时间），但不会产生与被中断的训练相同的结果                                                                                       |
| sharded_ddp                 | 指定使用来自 FairScale 的 Sharded DDP 训练（仅在分布式训练中）                                                                                                                                                                                      |
| fsdp                        | 指定使用 PyTorch Distributed Parallel Training 训练（仅在分布式训练中），可选的参数：full_shard/ shard_grad_op/ offload/ auto_wrap                                                                                                                      |
| fsdp_min_num_params         | 指定 FSDP 的默认的最少的 parameters 数量从而用于 Default Auto Wrapping。仅当 fsdp 参数有效时 `fsdp_min_num_params` 才有意义                                                                                                                                 |
| deepspeed                   | 指定使用 Deepspeed                                                                                                                                                                                                                   |
| label_smoothing_factor      | 指定 label smoothing 因子。零代表没有标签平滑                                                                                                                                                                                                  |
| debug                       | 指定启用一个或多个 debug 特性                                                                                                                                                                                                               |
| optim                       | 指定使用的优化器。可以为：adamw_hf/ adamw_torch/ adamw_apex_fused/ adamw_anyprecision/ adafactor                                                                                                                                              |
| optim_args                  | 指定提供给 AnyPrecisionAdamW 的可选参数                                                                                                                                                                                                    | 
| adafactor                   | 已废弃，推荐使用 `optim="adafactor"` 来代替                                                                                                                                                                                                 |
| group_by_length             | 指定是否将训练数据集中长度大致相同的样本分组（从而尽量减少填充的使用，使之更有效），只有在应用动态填充时才有用                                                                                                                                                                          |
| length_column_name          | 指定预计算的长度的列名。如果该列存在，`group_by_length` 将使用这些值，而不是在训练启动时计算它们。除非 `group_by_lengt=True` 并且数据集是 Dataset 的一个实例，否则会被忽略                                                                                                                   |
| report_to                   | 指定要报告结果和日志的集成商的列表，支持的可选值：azure_ml/ comet_ml/ mlflow/ neptune/ tensorboard/ clearml/ wandb。`all` 表示报告所有的集成商；`none` 表示都不报告。注意，如果不为 `none`，且当前处于可联网的状态，会要求输入对应的 API Key                                                             |
| ddp_find_unused_parameters  | 指定当使用分布式训练时，传递给 DistributedDataParallel 的 `find_unused_parameters` 的值                                                                                                                                                            |
| ddp_bucket_cap_mb           | 指定当使用分布式训练时，传递给 DistributedDataParallel 的 bucket_cap_mb 的值                                                                                                                                                                       |
| dataloader_pin_memory       | 指定在 data loaders 中是否要 pin memory                                                                                                                                                                                                 |
| skip_memory_metrics         | 指定是否跳过向 metrics 添加 memory profiler 报告，默认情况下跳过（即 True），因为这会减慢训练和评估的速度                                                                                                                                                             |
| push_to_hub                 | 指定每次保存模型时是否将模型推送到 Hub                                                                                                                                                                                                            |
| resume_from_checkpoint      | 指定有效 checkpoint 的文件夹的路径，这个参数不直接被 Trainer 使用，而是由 Training/Evaluation 脚本使用                                                                                                                                                         |
| hub_model_id                | 指定 repo 的名字从而与本地 `output_dir` 保持同步，默认为 `output_dir` 的名称                                                                                                                                                                          |
| hub_strategy                | 指定推送到 Hub 的范围和时间，可选值: end/ every_save/ checkpoint/ all_checkpoints                                                                                                                                                               |
| hub_token                   | 指定用来推送模型到 Hub 的 token，将默认为通过 huggingface-cli 登录获得的缓存文件夹中的token                                                                                                                                                                   |
| hub_private_repo            | 指定是否将 Hub Repo 设为私有                                                                                                                                                                                                              |
| gradient_checkpointing      | 指定是否使用 gradient checkpointing 来节省内存。如果为 True，则会降低反向传播的速度                                                                                                                                                                         |
| include_inputs_for_metrics  | 指定 inputs 是否会被传递给 compute_metrics 函数，这适用于需要 `inputs`、`predictions` 和 `references` 的指标的计算                                                                                                                                         |
| auto_find_batch_size        | 是否通过指数衰减自动寻找适合内存的 batch size，以避免 CUDA Out-of-Memory 的错误。需要安装 `accelerate`                                                                                                                                                        |
| full_determinism            | 如果为 `True` 则调用 `enable_full_determinism` 而不是 `set_seed()`，从而确保分布式训练的结果可重复                                                                                                                                                        |
| torchdynamo                 | 用于 TorchDynamo 的后端编译器，可选值：eager/ aot_eager/ inductor/ nvfuser/ aot_nvfuser/ aot_cudagraphs/ ofi/ fx2trt/ß onnxrt/ ipex                                                                                                           |
| ray_scope                   | 指定使用 Ray 进行超参数搜索时使用的范围。默认情况下，将使用 `last`。然后，Ray 将使用所有试验的最后一个 checkpoint，比较这些 checkpoint 并选择最佳 checkpoint。然而，也有其他选项，更多选项见 Ray 文档。                                                                                                  |
| ddp_timeout                 | 指定 `torch.distributed.init_process_group` 调用的超时时间                                                                                                                                                                                |
| use_mps_device              | 是否使用基于 Apple Silicon 的 mps 设备                                                                                                                                                                                                    |

### 1.2 方法

此外，`TrainingArgument` 还有一些常用方法：

| 方法                    | 说明                                                                  |
|-----------------------|---------------------------------------------------------------------|
| get_process_log_level | 获取 log level，具体结果取决于是否是 `node 0` 的主进程、`node non-0` 的主进程、以及非主进程      |
| get_warmup_steps      | 返回用于线性预热的 step 数量                                                   |
| main_process_first    | 一个用于 Torch 分布式环境的上下文管理器，它需要在主进程上做一些事情，同时 block 副本，当它完成后再 release 副本 | 
| to_dict               | 序列化该实例到一个字典，同时用枚举的值来代替 Enum 对象                                      |
| to_json_string        | 序列化该实例到一个 json 字符串                                                  |
| to_sanitized_dict     | 采用 TensorBoard 的 hparams 来序列化该实例                                    |

## 2. Seq2SeqTrainingArguments

`Seq2SeqTrainingArguments` 是继承了 `TrainingArgument` 的类，所以 `TrainingArgument` 参数在 `Seq2SeqTrainingArguments` 中也是可用的。

| 参数                    | 说明                                                                                                    |
|-----------------------|-------------------------------------------------------------------------------------------------------|
| sortish_sampler       | 指定是否使用 sortish 采样器。目前只有在底层数据集是 `Seq2SeqDataset` 的情况下才有可用。它根据长度对输入进行排序从而最小化 `padding` 的大小，其中对训练集有一点随机性 |
| predict_with_generate | 指定是否使用 `generate` 来计算生成指标（ROUGE, BLEU ）                                                               |
| generation_max_length | 指定当 `predict_with_generate=True` 时，在每个 evaluation loop 中使用的最大长度，默认为模型配置的 `max_length` 值               |
| generation_num_beams  | 指定当 `predict_with_generate=True` 时，在每个 evaluation loop 使用的 `beams` 数量，将默认为模型配置中的 `num_beams` 值        |
















<seealso>
<category ref="ref_docs">
    <a href="https://www.huaxiaozhuan.com/工具/huggingface_transformer/chapters/4_trainer.html">Transformers Trainer</a>
</category>
<category ref="ref_github"></category>
<category ref="ref_issues"></category>
<category ref="ref_hf"></category>
<category ref="ref_ms"></category>
</seealso>

