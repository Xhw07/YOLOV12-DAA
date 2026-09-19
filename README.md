# YOLOV12-DAA-Fall-Detection

Campus Fall Detection



1. 人体摔倒检测是构建智能化校园安防体系、保障学生运动安全的关键技术之一。在大学校园场景中，摔倒事故多发于宿舍、操场与教学楼楼梯等区域，尤其在体育活动与课间拥挤通行的时段；田径、球类等多样化运动带来的剧烈姿态变化，以及人群遮挡、光照不均等复杂环境因素，对实时监测提出了严峻挑战。传统基于穿戴设备的检测方法存在传感器易脱落、误报率高、部署成本大等问题，难以满足校园安防对普适性、低成本与非侵入性的综合需求。

本文提出一种融合差异化轻量化卷积、解耦通道 - 空间注意力（Decoupled CS Attention）与智能体注意力（Agent Attention）的摔倒检测模型 **YOLOV12-DAA**。该模型以 YOLOv12 为基线，通过将深层骨干与检测头中的独立 3×3 卷积替换为 1×1 卷积以降低参数量与计算开销，并分别在网络第 2、20 层嵌入 Decoupled CS Attention、在第 4 层嵌入 Agent Attention，从而增强遮挡、多人及小目标场景下的特征判别能力与全局时序建模能力，有效解决了现有方法在计算效率与复杂场景鲁棒性之间难以平衡的核心问题。实验表明，该模型在自建校园摔倒检测数据集上取得 99.3% 的 mAP50 与 93.3% 的 mAP50-95 检测精度，在 RTX 5060 Laptop 上单帧推理仅 0.7 ms，对于提升大学校园安全防护水平具有重要参考意义。



1. YOLOV12-DAA 核心创新点

（1）差异化轻量化卷积设计：针对校园摔倒检测场景的实时性需求，对 YOLOv12 进行差异化精简。浅层骨干与检测头保留 3×3 标准卷积以维持大感受野，充分提取人体轮廓与肢体边缘等基础空间特征；仅将深层骨干与检测头中的独立 3×3 标准卷积替换为 1×1 卷积，在不改变通道数、模块数量与整体拓扑结构的前提下，沿通道维度完成线性特征交互，大幅削减冗余参数与计算开销，同时保留核心语义表征能力，实现无损轻量化设计。

（2）Decoupled CS Attention（解耦通道 - 空间注意力）模块：为增强模型在遮挡与复杂光照下的特征判别力，在骨干网络输出层（第 2、20 层）引入 Decoupled CS Attention。通道分支通过全局平均池化与全连接层学习通道权重，空间分支通过最大池化与平均池化融合并经卷积学习空间权重，以 C/4 通道压缩比实现通道与空间注意力的解耦优化，抑制背景冗余，将人体运动特征与复杂环境噪声分离，显著提升人群遮挡场景下的判别能力。

（3）Agent Attention（智能体注意力）模块：为捕捉站立到摔倒过程的连续时序变化，在第 4 层引入 Agent Attention。通过初始化 K=16 个代理节点，建立帧特征与代理节点之间的关联相似度矩阵，经 Softmax 归一化获得时序注意力权重并对代理节点加权求和，以残差连接输出时序增强特征；将全局交互计算复杂度降至线性，实现高效的多帧全局上下文建模，补偿了原始检测器时序建模能力不足的问题。

（4）双注意力协同机制：Decoupled CS Attention 强化局部判别性特征提取与背景抑制，Agent Attention 提升全局时序建模能力，二者互补实现性能的全面提升。

效率与精度平衡：相较于经典 YOLOv12n 基线模型，提出的方法在保持实时推理能力的同时显著提升检测精度。在 NVIDIA GeForce RTX 5060 Laptop 上单帧推理速度达 0.7 ms，显存占用约 480 MB，模型体积仅 5.4 MB；mAP@50 达 99.3%，mAP@\[0.5:0.95] 达 93.3%，精确率 0.982、召回率 0.986，综合性能优于 YOLOv8、YOLOv9、YOLOv10 等主流模型，满足嵌入式终端部署与校园实时监控需求。



1. 实验数据集：大学生校园摔倒检测数据集

3.1 数据集概况

本研究基于自建大学校园场景摔倒检测数据集，样本由智能手机（Vivo S16e）拍摄的运动场景视频提取，包含站立（Standing）与摔倒（Fall）两类行为：



| 数据集名称                     | 包含类别          | 图像总数 | 图像分辨率            | 数据分布（训练：验证：测试） |
| ------------------------- | ------------- | ---- | ---------------- | -------------- |
| Our campus fall detection | Standing、Fall | 4213 | 原始视频帧（未统一指定输入尺寸） | 7:2:1          |

3.2 场景分布

数据集覆盖跑道、篮球场与室内三类校园场景，兼顾正常活动与意外摔倒姿态，以增强模型泛化能力：



| 场景类别 | 训练集  | 验证集 | 测试集 | 合计   |
| ---- | ---- | --- | --- | ---- |
| 跑道   | 598  | 181 | 81  | 860  |
| 篮球场  | 125  | 33  | 23  | 181  |
| 室内场景 | 2226 | 628 | 318 | 3172 |
| 合计   | 2949 | 842 | 422 | 4213 |

3.3 公共数据集补充

为进一步增强模型可比性与泛化能力，本研究融合了两个公开摔倒检测数据集作为自建数据的补充：



* **LE2I with Upright and Fall**：基于原始 LE2I 基准构建，包含日常活动与突发摔倒等多样室内场景（[https://universe.roboflow.com/new-workspace-qfcus/le2i-with-upright-and-fall/dataset/2](https://universe.roboflow.com/new-workspace-qfcus/le2i-with-upright-and-fall/dataset/2)）。

* **Falldown Detection F8XTB**：以体育场、运动场等室外场景的标注网络图像为主（[https://universe.roboflow.com/kid-g8rt3/falldown-detection-f8xtb/dataset/1](https://universe.roboflow.com/kid-g8rt3/falldown-detection-f8xtb/dataset/1)）。

两个数据集在场景类型、光照条件、遮挡程度与人体姿态上形成互补，显著丰富了训练数据的多样性。

3.4 数据集获取与结构

自建数据集下载链接：待上传（自建数据暂无公开下载链接，如有需要请联系作者）。

文件夹组织（下载后解压至项目根目录，结构如下）：



```
data/our\_fall\_detection/

├── images/

│   ├── train/          # 训练集图像（包含跑道、篮球场、室内场景图像）

│   ├── val/            # 验证集图像（同上）

│   └── test/           # 测试集图像（同上）

└── labels/

&#x20;   ├── train/          # 训练集标注文件（YOLO格式txt，与图像一一对应）

&#x20;   ├── val/            # 验证集标注文件

&#x20;   └── test/           # 测试集标注文件
```



1. 实验环境配置

4.1 依赖安装

推荐使用 Anaconda 创建虚拟环境，确保依赖版本匹配：



```
\# 1. 创建并激活虚拟环境

conda create -n dcs-agent-yolo python=3.11

conda activate dcs-agent-yolo

\# 2. 安装 PyTorch（需适配 CUDA 版本，本文环境为 CUDA 12.9；CPU 用户可替换为 cpu 版本）

pip install torch==2.8.0 torchvision --index-url https://download.pytorch.org/whl/cu129

\# 3. 安装其他核心依赖库

pip install numpy pandas scipy matplotlib seaborn opencv-python pillow tqdm thop

\# 4. 安装 YOLOv12 所需的其他依赖

\#    请根据你实际使用的 YOLOv12 代码库（如 sunsmarterjie/yolov12）中的 requirements.txt 进行安装
```

> 注：PyTorch 官方 wheel 具体索引地址请以 
>
> [https://download.pytorch.org/whl/](https://download.pytorch.org/whl/)
>
>  实际可用版本为准（如 cu128），安装前先确认与本地 CUDA 驱动版本匹配。

4.2 硬件要求

GPU：本文实验平台为 NVIDIA GeForce RTX 5060 Laptop（8 GB 显存），训练 100 轮显存占用约 480 MB；推荐显存 ≥ 8 GB 的 NVIDIA GPU（如 RTX 3060/4060/5060，CUDA ≥ 12.9）。

CPU：可运行推理（单帧耗时明显高于 GPU），不推荐完整训练。



1. 实验结果

5.1 核心指标（本文方法）



| 指标              | 数值                          |
| --------------- | --------------------------- |
| mAP@50          | 99.3%                       |
| mAP@\[0.5:0.95] | 93.3%                       |
| 精确率（Precision）  | 0.982                       |
| 召回率（Recall）     | 0.986                       |
| 推理速度            | 0.7 ms / 帧（RTX 5060 Laptop） |
| 显存占用            | \~480 MB                    |
| 模型大小            | 5.4 MB                      |

5.2 与主流检测模型对比（总体准确率）

在自建数据集上，本文方法总体准确率达 90.76%，优于主流 YOLO 系列模型。

注：



1. mAP@50 指在 IoU 阈值为 0.5 时的平均精度均值，用于衡量模型的基础检测性能。

2. mAP@\[0.5:0.95] 指在 IoU 阈值从 0.5 到 0.95（步长 0.05）下的平均 mAP，用于衡量模型的定位鲁棒性。

3. 混淆矩阵分析表明，本文模型在站立（Stand）与摔倒（Fall）之间混淆最少，并有效抑制了背景误检，总体准确率相比 YOLOv8、YOLOv9、YOLOv10 分别提升 6.87%、3.08% 与 1.19%。

4. 代码使用说明

6.1 模型训练

运行 train.py 脚本启动训练。脚本自动加载基础 YOLOv12n 模型，并集成 Decoupled CS Attention 与 Agent Attention 模块。示例命令（适配跌倒检测数据集）：



```
python train.py \\

&#x20; \--data ./data/our\_fall\_detection \\

&#x20; \--epochs 100 \\

&#x20; \--batch 8 \\

&#x20; \--lr0 0.001 \\

&#x20; \--optimizer SGD \\

&#x20; \--cos-lr \\

&#x20; \--save\_dir ./runs/dcs\_agent\_yolo \\

&#x20; \--device cuda:0
```

关键参数说明：



| 参数名         | 含义                      | 默认值                         |
| ----------- | ----------------------- | --------------------------- |
| --data      | 数据集根目录路径                | ./data/our\_fall\_detection |
| --epochs    | 训练轮数                    | 100                         |
| --batch     | 批次大小（根据显存调整）            | 8                           |
| --lr0       | 初始学习率                   | 0.001                       |
| --optimizer | 优化器（SGD / Adam / AdamW） | SGD                         |
| --cos-lr    | 余弦退火学习率调度               | —                           |
| --save\_dir | 模型保存目录                  | ./runs/exp                  |
| --device    | 训练设备（cuda:0 或 cpu）      | cuda:0                      |

训练配置说明：本文采用 batch=8、SGD 优化器（带动量）与余弦退火学习率调度，学习率从 0.001 衰减至 0.001×lrf，共训练 100 轮。训练结束后最佳模型保存为 best.pt。

6.2 模型预测

使用训练好的权重进行单张图像预测，运行 predict.py 脚本，示例命令：



```
python predict.py \\

&#x20; \--image\_path ./data/our\_fall\_detection/images/test/sample.jpg \\

&#x20; \--model\_path ./weights/best.pt \\

&#x20; \--device cuda:0
```

预测输出示例：



```
🎯 PREDICTION RESULT:

Most likely class: standing

Probability: 0.894
```

6.3 预训练权重

提供基于校园摔倒检测数据集训练完成的最优权重（待上传），可直接用于预测或微调。适用场景：针对摔倒检测场景中的 “站立（standing）” 和 “摔倒（fall）” 两类分类。若需扩展其他动作类别或类似场景，建议基于此权重微调（冻结部分主干，仅训练检测头，可显著减少训练数据量）。



1. 项目文件结构



```
DCS-Agent-YOLO-Fall-Detection/

├── data/                                  # 数据集目录

│   └── our\_fall\_detection/                # 摔倒检测数据集

│       ├── images/                        # 图像文件夹

│       │   ├── train/                     # 训练集图像

│       │   ├── val/                       # 验证集图像

│       │   └── test/                      # 测试集图像

│       └── labels/                        # 标签文件夹（YOLO格式txt）

│           ├── train/

│           ├── val/

│           └── test/

├── models/                                # 整体模型实现

│   ├── yolo\_simplified.py                 # 差异化轻量化卷积（1×1 替换深层 3×3）

│   ├── yolo\_decoupled\_cs.py               # Decoupled CS Attention 模块实现

│   ├── yolo\_agent.py                      # Agent Attention 模块实现

│   └── yolo\_dcs\_agent.py                  # 主模型（整合上述模块）

├── weights/                               # 预训练权重存放

│   └── best.pt                            # 最优模型权重

├── train.py                               # 训练脚本（集成各模块，含最佳训练配置）

├── predict.py                             # 预测与性能评估脚本

├── fall\_detection.yaml                    # 数据集配置文件

├── requirements.txt                       # 依赖包列表

└── README.md                              # 项目说明文档
```



1. 已知问题与注意事项
* 类别适配：当前模型与权重仅针对 “站立（standing）” 和 “摔倒（fall）” 两类场景。若需扩展其他动作类别（如行走、跑步等）或类似监控场景，需补充对应数据集并基于本权重微调。

* 场景局限：数据集缺少游泳、击剑、攀岩及低光环境样本，在特殊场景下的检测精度与鲁棒性可能受影响。

* 功能边界：模型仅能检测摔倒事件，无法区分扭伤、骨折等损伤类型；对降雨反光、浓雾等极端天气导致的图像退化未做专门优化。

* CUDA 版本问题：若安装 PyTorch 时出现 CUDA 不兼容，可替换为 CPU 版本（需将所有脚本的 --device 改为 cpu），但训练和推理效率会大幅下降。

* 显存占用：训练时若显存不足（如 8 GB GPU），建议进一步降低 batch 并通过梯度累积等效模拟更大批次，或适当降低 imgsz。
