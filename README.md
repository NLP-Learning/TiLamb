# TiLamb（Tibetan Large Language Model Base）

## 基于LLaMA2-7B增量预训练的藏文大语言模型

## 内容导引
| 章节                                  | 描述                                                         |
| ------------------------------------- | ------------------------------------------------------------ |
| [💁🏻‍♂️模型简介](#模型简介) | 简要介绍TiLamb-7B |
| [⏬模型下载](#模型下载) | TiLamb-7B下载地址|
| [💯任务评估](#任务评估) | 展示了TiLamb-7B在部分藏文NLP下游任务上的效果|
| [🔔致谢](#致谢) | 特别感谢对本项目有帮助的优秀项目|
| [📳免责声明](#免责声明) | TiLamb-7B使用免责声明|

### 模型简介

**TiLamb-7B** 是藏文大语言模型的基座模型，它使用了 26.43GB 的藏文语料，基于Meta发布的可商用大模型 [LLaMA2-7B](https://github.com/facebookresearch/llama) 模型，通过 LoRA 方法进行了增量预训练。该模型在 LLaMA2 的基础上扩展了词表，从原有的词表大小 32,000 扩充藏文词汇至 61,221 ，并对 LLaMA2-7B 原始模型的 embedding 和 lm_head 进行了均值扩充初始化。

#### 📝 构建藏文分词模型

- LLaMA2使用了BPE（Byte Pair Encoding）算法，本项目也使用SentencePiece的BPE算法，用近10G藏文原始语料训练得到词表大小为32,000、覆盖率为99.95\%的藏文分词模型，重要参数如表所示。

| 参数 | 值 |
|------|----|
| 词表大小 (`vocab_size`) | 32000 |
| 分词算法 (`model_type`) | BPE |
| 数字拆分 (`split_digits`) | True |
| 字节回退 (`byte_fallback`) | True |
| 最大句子长度 (`max_sentence_length`) | 5000 |
| 字符覆盖率 (`character_coverage`) | 0.9995 |

#### 📖 扩充藏文词表

- 在原始LLaMA2的词表中增加额外约30,000个藏文tokens，增强了藏文的编码和解码效率，并提高了LLaMA2对藏文的理解能力。

#### 🌟 增量预训练参数选择

| 参数                                   | 值                                 | 参数                                   | 值                                    |
|----------------------------------------|------------------------------------|----------------------------------------|---------------------------------------|
| `cutoff_len`                           | 1024                               | `learning_rate`                        | 2e-4                             |
| `finetuning_type`                      | lora                               | `num_train_epochs`                     | 1.0                                   |
| `per_device_train_batch_size`          | 4                                  | `gradient_accumulation_steps`          | 2                                     |
| `lr_scheduler_type`                    | cosine                             | `max_grad_norm`                        | 1.0                                   |
| `lora_rank`                            | 8                                  | `lora_dropout`                         | 0.1                                   |
| `lora_target`                          | q_proj, v_proj                 | `warmup_steps`                         | 0                                     |
| `additional_target`                    | embed_tokens, lm_head, norm   | `fp16`                                 | True                                  |

### 模型下载

| 属性           | 描述                                                   |
|----------------|------------------------------------------------------|
| 模型名称       | TiLamb-7B                                               |
| 模型类型       | **基座模型，非chat模型**      |
| 参数大小       | 7B      |
| 训练类型       | Causal-LM (CLM)      |
| 训练方式       | LoRA + 全量emb/lm-head      |
| 基于的原始模型   | [LLaMA2-7B-base](https://github.com/facebookresearch/llama)      |
| 训练语料       | 无标注 26.43 GB藏文通用语料   |
| 词表大小       | 61,221    |
| 语言支持       | 藏文    |
| 文件大小         | 13.0 GB       |
| **🤖ModelScope    下载链接** | [https://modelscope.cn/models/YoLo2000/TiLamb-7B/summary](https://modelscope.cn/models/YoLo2000/TiLamb-7B/summary)|
| **🤗Hugging Face下载链接** | [https://huggingface.co/zhuangwenhao/TiLamb-7B](https://huggingface.co/zhuangwenhao/TiLamb-7B)  |

> [!NOTE]
> [1] *TiLamb-7B 用于扩充LLaMA2原始词表的藏文分词模型并非上文提到的[藏文分词模型](#模型简介)。当时使用早期训练的词表大小30,000、覆盖率99.95\%的藏文分词模型，而[上文提到的分词模型](#模型简介)为近期训练所得，也是我们目前认为最符合需求的藏文分词模型，并将在我们下一代藏文大语言模型中使用。*</br>
> [2] *TiLamb-7B 是一个未经监督微调的基座模型，**不具备对话能力**。*</br>
> [3] *要进行藏文对话和[藏文 NLP 下游任务](#任务评估)的适配，建议使用 [LLaMA-Factory](https://github.com/hiyouga/LLaMA-Factory/tree/main) 框架进行微调。*</br>

### 任务评估

- 根据多种藏文下游任务分别制作数千到几万条不等的微调数据集，微调后的TiLamb在藏文新闻分类、藏文实体关系分类、藏文机器阅读理解、藏文分词、藏文摘要、藏文问题回答、藏文问题生成共七个下游任务中进行验证，多项指标结果相较传统方法和其他藏文预训练模型有大幅提升。

- 本文采用[LLaMA-Factory](https://github.com/hiyouga/LLaMA-Factory/tree/main)框架中的微调模板，一条用于藏文分词下游任务的微调数据示例格式如下，其中instruction为用于描述任务的指令，input可选，为任务指令的补充输入，output为期望模型产生的回复：

```json
{
    "instruction": "མཐུན་སྦྱོར་གྱི་འདུ་ཤེས་ཐོག་ནས་དཔལ་འབྱོར་དང་སྤྱི་ཚོགས་ཁྱོན་ཡོངས་ཡར་ཐོན་ལ་སྐུལ་མ་གཏོང་དགོས།",
    "input": "",
    "output": "མཐུན་སྦྱོར་/གྱི་/འདུ་ཤེས་/ཐོག་/ནས་/དཔལ་འབྱོར་/དང་/སྤྱི་ཚོགས་/ཁྱོན་ཡོངས་/ཡར་ཐོན་/ལ་/སྐུལ་མ་/གཏོང་/དགོས།/"
}
```

- 藏文NLP下游任务微调均基于以下参数选择：

| 参数                                   | 值                 | 参数                                   | 值          |
|----------------------------------------|--------------------|----------------------------------------|-------------|
| `cutoff_len`                           | 2048               | `learning_rate`                        | 2e-4    |
| `finetuning_type`                      | lora               | `num_train_epochs`                     | 3.0          |
| `per_device_train_batch_size`          | 4                  | `gradient_accumulation_steps`          | 2            |
| `lr_scheduler_type`                    | cosine             | `max_grad_norm`                        | 1.0          |
| `lora_rank`                            | 8                  | `lora_dropout`                         | 0.05         |
| `lora_target`                          | q_proj, v_proj | `fp16`                                 | True         |


#### 藏文新闻分类

该任务选用由复旦大学自然语言处理实验室发布的藏语新闻数据集Tibetan News Classification Corpus。数据集包含9,204条样本，涉及政治、经济、教育、旅游、环境、艺术、文学、宗教等12个类别。按9:1的比例将其划分为训练集和测试集，训练集的数据用于制作微调数据集。评价指标为Accuracy、Macro-Precision、Macro-Recall、Macro-F1，实验结果如下所示。

| 模型                | Accuracy(%) | Macro-F1(%) |
|---------------------|-------------|-------------|
| Transformer         | 28.63       | 28.79       |
| CNN(syllable)       | 61.51       | 57.34       |
| TextCNN             | 61.71       | 61.53       |
| DPCNN               | 62.91       | 61.17       |
| TextRCNN            | 63.67       | 62.81       |
| Bert-base-Tibetan   | -           | 51.00       |
| TiBERT              | 71.04       | 70.94       |
| CINO-base           | 73.10       | 70.00       |
| TiKEM               | 74.46       | 72.61       |
| **TiLamb+LoRA**     | **78.85**   | **77.45**   |


#### 藏文实体关系分类

为了验证TiLamb模型对知识的记忆及融合运用能力，本文使用6,433条三元组-文本对齐数据集，三元组中共有11种关系。该任务要求在给定两个实体和包含该实体的对应文本后，给出两个实体之间的关系类别。按9:1的比例将其划分为训练集和测试集，训练集的数据用于制作该任务的微调数据集。评价指标为Accuracy(\%)、Macro-P(\%)、Macro-R(\%)和Macro-F1(\%)，实验结果如下所示。

| 模型          | Accuracy(%) | Macro-P(%) | Macro-R(%) | Macro-F1(%) |
|---------------|-------------|------------|------------|-------------|
| FastText      | 55.80       | 34.05      | 32.98      | 31.61       |
| DPCNN         | 70.94       | 54.21      | 49.23      | 48.65       |
| TextCNN       | 72.38       | 71.03      | 59.11      | 56.76       |
| TiBERT        | 84.70       | 76.66      | 68.82      | 67.94       |
| CINO-base     | 85.31       | 75.48      | 69.12      | 66.73       |
| MiLMO         | 85.76       | 77.13      | 68.97      | 68.57       |
| TiKEM         | 90.12       | 91.73      | 75.61      | 76.34       |
| **TiLamb+LoRA** | **95.98**   | **97.14**  | **88.98**  | **91.60**   |


#### 藏文机器阅读理解

机器阅读理解任务是给定一段文本和一个问题，让模型回答对应问题。这需要模型理解问题和上下文语义，然后进行推理、判断等，给出具体答案。使用藏文机器阅读理解数据集TibetanQA对模型的阅读理解能力进行评估，该数据集包含了1,513篇文章和20,000个问答对。为了评估模型性能，本文使用EM值（精确匹配）和F1值作为评价指标。以8:2的比例将数据划分为训练集和测试集，训练集用于制作该任务的微调数据集，实验结果如表8所示。

| 模型         | EM(%) | F1(%) |
|--------------|-------|-------|
| R-Net        | 55.8  | 63.4  |
| BiDAF        | 58.6  | 67.8  |
| QANet        | 57.1  | 66.9  |
| TiBERT       | 53.2  | 73.4  |
| TiLamb+LoRA  | 46.6  | 77.4  |
| Ti-Reader    | 67.9  | 77.4  |
| **TiKEM**    | **69.4**  | **80.1**  |


#### 藏文分词

对于藏文这种结构复杂、资源相对较少的语言而言，正确的分词对于进一步的语言处理任务，如语义分析、机器翻译和信息检索等，都具有至关重要的作用。该任务使用中文信息学会举办的第一届藏文分词评测所用的数据集，数据集中藏语短句去重后有21,427条，本文保留了1,000条作为测试集，剩余的20,427条用于制作微调数据集，测试结果与第一届藏文分词评测第一名TIP-LAS对比如下所示。

| 模型            | Precision(%) | Recall(%) | F1(%) |
|-----------------|--------------|-----------|-------|
| TIP-LAS         | 93.14        | 92.17     | 92.66 |
| **TiLamb+LoRA** | **93.58**    | **93.71** | **93.64** |


#### 藏文摘要

文本摘要作为自然语言处理中的一个重要分支，可以有效的减少冗余信息，从而提高浏览文本速度。使用47,088条新闻与对应摘要做微调，测试集使用5,232条新闻与对应摘要，用于实验的数据集均以新闻标题简要作为摘要，实验结果如下所示。

| 模型                  | ROUGE-1(%) | ROUGE-2(%) | ROUGE-L(%) |
|-----------------------|------------|------------|------------|
| 统一模型              | 19.81      | 13.27      | 16.90      |
| CMPT模型（Ti-SUM）    | 39.53      | 26.42      | 38.02      |
| CMPT（50000条）       | 49.16      | 33.43      | 48.66      |
| **TiLamb+LoRA**       | **53.99**  | **37.22**  | **52.89**  |


#### 藏文问题回答

该任务使用本组人工制作的TiconvQA藏文多轮对话数据集，其中包含2,120条藏文文章段落及20,420对多轮问答对。为了进行实验评估，按照8:2的比例将数据集划分为训练集和测试集。在评估实验中，使用EM值和F1值作为评估指标，实验结果如下所示。

| 模型          | EM(%)    | F1(%)    |
|---------------|----------|----------|
| DrQA          | 41.49    | 61.51    |
| TiBERT        | 45.12    | 65.71    |
| **TiLamb+LoRA** | **45.28** | **72.84** |


#### 藏文问题生成

问题生成是自然语言生成的一项任务，它以文本和目标答案为输入，自动从答案中生成问题。使用用于机器阅读理解的藏文问答数据集TibetanQA，按照约9:1的比例划分训练集和测试集，其中用于微调的数据共17,762条，用于测试的数据为1,976条，实验指标数值均为百分比，实验结果如下所示。

| 模型            | BLEU-1 | BLEU-2 | BLEU-3 | BLEU-4 | ROUGE-L |
|-----------------|--------|--------|--------|--------|---------|
| S2S+ATT+CP      | 29.99  | 20.14  | 13.90  | 9.59   | 31.45   |
| TiBERT          | 35.48  | 28.60  | 24.51  | 21.30  | 40.04   |
| TiBERT+wh       | **47.28** | 31.35  | 23.02  | 17.52  | 46.78   |
| **TiLamb+LoRA** | 44.60  | **35.24** | **28.88** | **24.47** | **50.42** |


### 致谢

本项目主要参考并受益于[LLaMA-Factory](https://github.com/hiyouga/LLaMA-Factory/tree/main)、[Chinese-LLaMA-Alpaca-2](https://github.com/ymcui/Chinese-LLaMA-Alpaca-2/tree/main)，真诚感谢项目作者的付出。


### 免责声明

> [!IMPORTANT]
> - 本项目基于 Meta 发布的 LLaMA2-7B 模型开发，使用时请严格遵守 LLaMA2-7B 的开源许可协议。
> - 如果涉及使用第三方代码，请务必遵从相关的开源许可协议。
> - 模型生成的内容准确性可能受到计算方法、随机因素等的影响，因此，我们不对模型输出的准确性提供任何保证，也不会对使用相关资源和输出结果产生的任何损失承担责任。
> - 如果将相关模型用于商业用途，开发者应遵守当地法律法规，确保模型输出内容的合规性。本项目不对任何由此衍生的产品或服务承担责任。
