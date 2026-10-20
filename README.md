# Epistemic Injustice in Generative AI

## 论文展示

**Epistemic Injustice in Generative AI: How Minority Language Systems Reproduce or Resist Digital Exclusion**

- Authors: Rende Li, Lina Zhang, Sumin Feng
- Venue: *International Journal of Human–Computer Interaction* (published online 13 October 2026)
- DOI: [10.1080/10447318.2026.2744497](https://doi.org/10.1080/10447318.2026.2744497)


## 一句话概括

论文把“少数语言模型表现差异”拆解为可分析的技术路径、交叉身份惩罚和社区定义的文化伤害，并以参数高效的 CAML-LLM 作为干预系统，验证系统设计可以在所评估的场景中改善识别性能与文化适切性。

## 工程特色

### 1. 面向五种语言的多模态数据工程

- 覆盖 Tibetan、Uyghur、Mongolian、Yi、Zhuang，横跨四个语系。
- 每种语言保留约 50 小时音频、200 万文本 token、10,000 张文化相关图像；总量约为 250 小时音频、1,000 万文本 token、50,000 张图像。
- 采样同时控制宗教/世俗、方言/标准形式、本土/借用概念三组维度，优先类别占每个语言分层的 30%–40%。
- 通过语言专家筛查、15% 分层复核和第三方争议裁决进行质量控制；跨属性 Krippendorff's alpha 为 0.78–0.86。

### 2. 文化知识图谱增强

- 每种语言构建约 15,000 个实体和 45,000 条关系。
- 使用 RotatE 生成 256 维实体嵌入。
- 在 Qwen-7B 的 32 个 Transformer 层中加入跨注意力：根据因果可用的上下文检索最多 8 个实体，并用门控残差注入文化上下文。
- 没有候选实体时跳过注意力计算，避免全 padding softmax。

### 3. 参数高效与跨模态适配

- 以 Qwen-7B 为基础模型，使用 LoRA 更新 32 个 QKV 投影，秩 `r=16`、缩放系数 `alpha=32`。
- 为五种语言扩展 25,000 个 tokenizer 行；原有词表冻结，仅训练新输入/输出行。
- 使用冻结的 Whisper-medium 音频特征和 CLIP ViT-B/32 图像特征，通过可训练投影映射到 4096 维语言模型空间。
- 音频/图像特征先缓存，再进行主训练，减少重复编码开销。

### 4. 社区权重进入训练目标

社区工作坊将宗教敏感性、文化适切性、老年人可访问性、方言包容性、性别公平等维度转成权重，并用于加权交叉熵：

```text
L = Σᵢ wᵢ · ℓ(yᵢ, ŷᵢ)
```

这使“哪些错误更重要”成为显式的训练目标，而不只是事后解释。

### 5. 工程规模与资源记录

- 训练更新约 293,142,560 个标量参数，LoRA adapter 本身约 8,388,608 个可训练参数。
- 8 张 NVIDIA A100 80GB，预处理约 34 小时，主适配约 72 小时。
- 平均 GPU 利用率约 82%，平均显存占用约 61GB/卡。

## 实验设计

### 研究问题与对应实验

| 研究问题 | 设计 | 主要输出 |
|---|---|---|
| RQ1：差异中有多少部分不能由已测技术因素解释？ | 四路因果中介分解；媒介变量包括音系复杂度、词频、句法结构 | CDE、交互项、PIE 与 non-PIE 占比 |
| RQ2：年龄、宗教术语、方言是否产生非加性交互？ | 交叉身份配置与主效应加和基准比较 | WER 的 pairwise / full-intersection excess |
| RQ3：社区优先级是否不同于统一权重？ | 83 人优先级工作坊、295 人评估工作坊；线性权重与阈值规则并行 | 社区权重、优先级指数、阈值漏检比例 |
| RQ4：CAML-LLM 是否优于数据匹配的 Qwen-7B？ | speaker-held-out 主测试、五语言迁移测试、全因子消融 | WER、BLEU、chrF++、文化适切性 |

### 数据与参与式流程

1. **优先级确定**：83 名参与者对 11 个候选领域进行两轮排名；跨五种语言均达到 75% top-four 支持的领域被保留为宗教术语、方言变体和本土概念。
2. **语料构建**：2023 年 3 月至 2024 年 2 月采集多模态语料；社区知识持有者参与知识图谱构建。
3. **模型训练**：CAML-LLM 与标准 Qwen-7B LoRA baseline 使用相同训练数据和训练设置。
4. **主测试**：15,600 条 speaker-disjoint held-out utterances，每种语言 3,120 条。
5. **迁移测试**：FLEURS-based 语音扩展 1,750 条录音；FLORES-based 翻译扩展 5,060 对句子。
6. **人类评估**：参与者以 1–7 分评价输出违反文化规范的程度，并比较社区权重和等权重。

### 主要结果

| 指标 | Baseline | CAML-LLM | 变化 |
|---|---:|---:|---:|
| 总体 WER | 31.2% | 22.7% | 绝对下降 8.5 pp；相对下降 27.2% |
| 文化适切性 | — | — | 提升 22.5% |
| FLEURS 语音扩展 WER | 34.6% | 30.3% | 相对下降 12.3% |
| FLORES 翻译 BLEU | 18.9 | 20.5 | 提升 1.6 |
| FLORES 翻译 chrF++ | 42.1 | 45.7 | 提升 3.6 |

四路分解的平均结果显示，non-PIE 组件占观测差异的 67.5%；敏感性分析在更强未测混杂设定下仍保留至少 52%。Tibetan 语音识别中，老年、宗教、方言交叉组 WER 为 43.9%，比主效应加和参考高 6.9 个百分点。

### 消融实验

| 配置 | WER | 相对 baseline |
|---|---:|---:|
| 无创新 baseline | 31.2% | 0.0 pp |
| KG only | 23.8% | -7.4 pp |
| Multimodal only | 26.1% | -5.1 pp |
| Community weights only | 28.3% | -2.9 pp |
| KG + MM | 20.9% | -10.3 pp |
| KG + CW | 22.1% | -9.1 pp |
| MM + CW | 24.8% | -6.4 pp |
| KG + MM + CW | 22.7% | -8.5 pp |

消融结果说明：知识图谱带来最大的单组件收益，多模态对齐次之，社区权重主要体现为公平导向的优化目标；组合配置的表现不能简单按单组件收益相加解释。



## 引用

```text
Li, R., Zhang, L., & Feng, S. (2026). Epistemic Injustice in Generative AI:
How Minority Language Systems Reproduce or Resist Digital Exclusion.
International Journal of Human–Computer Interaction.
https://doi.org/10.1080/10447318.2026.2744497
```
