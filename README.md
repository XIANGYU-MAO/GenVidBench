# GenVidBench: A Challenging Benchmark for Detecting AI-Generated Video


## Instruction
---
The rapid advancement of video generation models has made it increasingly challenging to distinguish AI-generated videos from real ones. This issue underscores the urgent need for effective AI-generated video detectors to prevent the dissemination of false information through such videos. However, the development of high-performance generative video detectors is currently impeded by the lack of large-scale, high-quality datasets specifically designed for generative video detection. To this end, we introduce GenVidBench, a challenging AI-generated video detection dataset with several key advantages: 1) Cross Source and Cross Generator: The cross-generation source mitigates the interference of video content on the detection. The cross-generator ensures diversity in video attributes between the training and test sets, preventing them from being overly similar. 2) State-of-the-Art Video Generators: The dataset includes videos from 8 state-of-the-art AI video generators, ensuring that it covers the latest advancements in the field of video generation. 3) Rich Semantics: The videos in GenVidBench are analyzed from multiple dimensions and classified into various semantic categories based on their content. This classification ensures that the dataset is not only large but also diverse, aiding in the development of more generalized and effective detection models. We conduct a comprehensive evaluation of different advanced video generators and present a challenging settings.

## FIT5230 新增：
### 论文解读
对应的论文如下：[sample paper.pdf](https://github.com/user-attachments/files/22000697/sample.paper.pdf)

在GenVidBench数据集中，训练集和测试集的划分严格遵循**跨来源（Cross Source）和跨生成器（Cross Generator）** 原则，目的是模拟真实场景中检测模型面对未知生成来源和生成器时的挑战。

<img width="611" height="291" alt="video_source" src="https://github.com/user-attachments/assets/75b0a8c9-8d0e-44ed-91d6-1f8abdd3a73d" />

#### **1. 训练集：来自“其他来源”的视频对**
训练集由与测试集**不同来源**且使用**不同生成器**生成的视频组成，避免模型依赖特定内容或生成器的属性。  
**示例**：  
- 训练集主要包含“Pair1”中的视频，real视频为ript，fake视频由4个文本到视频（T2V）生成器基于**VidProM数据集的文本提示**生成：  
  - Pika  
  - VideoCrafter2  
  - Modelscope
  - Text2Video-Zero  
  每个生成器贡献13,500个视频，内容均匹配VidProM的文本提示（如“一群人在山顶朝拜佛像”）。  

#### **2. 测试集：与真实视频“同来源”的视频对**
测试集由与真实视频**相同来源**但使用**不同生成器**生成的视频组成，且生成器与训练集完全不同，增加检测难度。  
**示例**：  
- 测试集主要包含“Pair2”中的视频，这些视频与真实视频来源（HD-VG-130M数据集）一致：  
  - 基于HD-VG-130M的**图像帧**生成：MuseV（I2V，2024.03，1210×576，12 FPS）、SVD（I2V，2023.11，1024×576，10 FPS）。  
  - 基于HD-VG-130M的**文本提示**生成：Mora（T2V，2024.03，1024×576，10 FPS）、CogVideo（T2V，2022.05，480×480，3 FPS）。  
  每个生成器贡献13,800个视频，内容与HD-VG-130M的真实视频匹配（如“厨房中坐在椅子上的男孩”）。

*VidProM是一个用于文本到视频扩散模型的大规模数据集，它为Pika、VideoCrafter2、Modelscope、T2V-Zero提供数据支持，这些模型基于VidProM的文本提示生成视频，从而参与构成GenVidBench数据集。*

#### **核心区别与挑战**
| 维度         | 训练集（Pair1）                          | 测试集（Pair2）                          |
|--------------|-----------------------------------------|-----------------------------------------|
| **来源**     | 基于VidProM的文本提示                    | 基于HD-VG-130M的图像/文本（与真实视频同来源） |
| **生成器**   | Pika、VideoCrafter2等（与测试集无重叠）   | MuseV、SVD、Mora、CogVideo（与训练集无重叠） |
| **目的**     | 让模型学习通用检测特征，不依赖特定来源或生成器 | 测试模型对未知来源和生成器的泛化能力          |

例如：若训练集学习了Pika生成的“自然风景”视频特征，测试集则要求模型检测MuseV生成的同一场景（自然风景）视频——两者来源不同、生成器不同，但内容相似，因此模型无法仅通过内容或生成器特征区分真假，必须学习更本质的伪造痕迹。

#### 相较于其他检测器的功能
<img width="530" height="119" alt="Snipaste" src="https://github.com/user-attachments/assets/2b72ec98-bb94-4287-b3c4-e9a77c358dcb" />

#### 数据集中fake视频生成规则
<img width="372" height="172" alt="Snipaste" src="https://github.com/user-attachments/assets/060146fe-7861-4ad1-9c68-4a056039599d" />

生成视频的语义基于主要维度的类别分类提取。即，将描述分为三个关键层级：物体、动作和地点，因为这三个元素能够构成一个完整的句子或故事，且具有丰富的语义。

#### 模型探讨
1. 交叉与非交叉的成功率 (以videoSwinTiny模型为例）
    - 对非交叉（某模型生成并验证）：非常高
    <img width="556" height="197" alt="Snipaste_2025-08-27_13-54-32" src="https://github.com/user-attachments/assets/4a82e7a7-27b0-4de5-9867-e381e3f429fe" />

    - 针对交叉：明显下降
    <img width="516" height="149" alt="Snipaste_2025-08-27_14-04-13" src="https://github.com/user-attachments/assets/709a61f9-eea2-44f6-905e-254ca658cc71" />

2. 先进模型对我们数据集（子集）的成功率
<img width="580" height="241" alt="Snipaste_2025-08-27_14-29-08" src="https://github.com/user-attachments/assets/25890237-bd61-4ece-8b4c-a90735c89bd5" />

在跨来源和跨生成器任务上训练的各种最先进方法的结果。最优性能指标不超过79.90%，这表明该任务具有相当大的挑战性。

#### 数据集探讨
1. 一些模型针对先进数据集和我们组合数据集的成功率

<img width="372" height="193" alt="Snipaste_2025-08-27_14-32-32" src="https://github.com/user-attachments/assets/6fe366ab-96ae-4461-9639-ac45e96ec93f" />

这表明我们的数据集更具有挑战性。

2. 数据集内部疑难案例

 <img width="466" height="102" alt="Snipaste_2025-08-27_14-32-32" src="https://github.com/user-attachments/assets/d4afaf83-7729-474b-ab15-fd614d5b8ca3" />

不同生成器中不同类别上疑难案例的比例。植物类最容易被误分类，而卡通类最容易区分。

 <img width="424" height="116" alt="Snipaste_2025-08-27_14-45-46" src="https://github.com/user-attachments/assets/7a341024-3bdb-4e5a-bb15-b1ebea96d00b" />

如上图，基于对难例的分析，我们选择了植物类别，并给出了各种模型在该类别上的实验结果
图中显示，SVD的分类准确率最低，这证明其生成性能最佳。

上述实验结果表明，模型在单一场景中的分类性能与在所有场景中的分类性能存在差异，因此有必要针对不同场景进行视频检测生成。GenVidBench提供的丰富语义标签有助于分析每个场景。

### 任务解读
作为light方：
1. 提供一个框架，供dark方进行检测，最终获得结果（是否由AI生成）
2. 针对dark方原始生成模式以及改进后的模式持续优化，使得检测更准确。


