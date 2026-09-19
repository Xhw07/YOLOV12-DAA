YOLOV12-DAA
人体摔倒检测是校园智能安防与大学生运动安全保障中的关键技术。宿舍、操场、教学楼楼梯、体育课与课间人流密集区域均是摔倒高发场景。传统目标检测模型在复杂校园场景中常存在人体运动特征提取不足、实时性有限、遮挡与光照鲁棒性差等问题。

本文提出一种改进 YOLOv12 的校园学生摔倒检测模型。该方法首先对 YOLOv12 进行轻量化简化，将深层骨干网络与检测头中的独立 
3
×
3
3×3 卷积替换为 
1
×
1
1×1 卷积，在保留浅层空间特征提取能力的同时降低参数量和计算开销；随后在模型第 2 层与第 20 层嵌入 Decoupled CS Attention，解耦通道与空间特征权重；并在第 4 层引入 Agent Attention，通过代理节点建模全局特征关联，聚焦高风险动作区域。实验表明，该方法在自建校园摔倒数据集上取得 99.3% 的 mAP@50 与 93.3% 的 mAP@[0.5:0.95]，优于原始 YOLOv12 及多种主流检测模型，对校园摔倒预警具有实际应用价值。

Improved YOLOv12 核心创新点
（1）简化 YOLOv12 轻量化设计：针对校园摔倒检测的实时部署需求，保留浅层骨干网络与检测头中的 
3
×
3
3×3 标准卷积，以维持人体轮廓、肢体边缘等基础空间特征提取能力；仅将深层骨干网络和检测头中的独立 
3
×
3
3×3 卷积替换为 
1
×
1
1×1 卷积。该设计在不改变通道维度、模块数量和整体拓扑结构的前提下，显著减少冗余参数与推理计算量，提高模型在跑道、球场、室内等场景中的推理速度与泛化能力。

（2）Decoupled CS Attention 模块：为增强复杂背景下的摔倒姿态判别能力，在模型第 2 层和第 20 层嵌入 Decoupled Channel-Spatial Attention。该模块分别通过通道注意力分支和空间注意力分支学习通道权重与空间权重，抑制背景冗余，突出人体运动区域与摔倒相关语义特征。通道分支使用全局平均池化与全连接层，空间分支融合最大池化与平均池化，并通过卷积学习空间权重，最终以逐元素相加方式输出优化特征。

（3）Agent Attention 模块：针对快速摔倒动作中连续时序关联建模不足的问题，在第 4 层引入 Agent Attention。该模块通过初始化 
K
=
16
K=16 个代理节点建模全局特征，计算帧特征与代理节点之间的相似度矩阵，经 Softmax 得到时序注意力权重，再通过加权求和与残差连接输出增强特征。该设计以较低计算复杂度增强全局场景感知与时序动态建模能力，补偿原始检测器对“站立→摔倒”快速动态过程捕捉不足的问题。

（4）双注意力协同机制：Decoupled CS Attention 强化局部判别性特征与复杂背景抑制能力，Agent Attention 提升全局上下文建模与动态时序关联能力。二者协同嵌入 YOLOv12 的关键层，在保持实时推理的前提下，提高对遮挡、多人、光照变化与运动模糊等复杂场景的适应能力，实现精度、鲁棒性与轻量化之间的平衡。

效率与精度平衡
相较于原始 YOLOv12 及主流检测模型，提出的改进方法在校园摔倒数据集上取得：

mAP@50：99.3%

mAP@[0.5:0.95]：93.3%

Recall：0.986

Precision：0.982

单帧推理时间：0.7 ms

内存占用：约 480 MB

模型大小：5.4 MB

整体准确率：90.76%

与 YOLOv8、YOLOv9、YOLOv10 相比，整体准确率分别提升 6.87%、3.08%、1.19%。与 YOLOv9 相比，多尺度检测效率提升 13.4%。在验证集 843 张图像上，mAP@50 达到 99.3%，mAP@[0.5:0.95] 达到 93.3%，兼顾检测精度、实时性与轻量化部署需求。

实验数据集
3.1 数据集概况
本研究基于自建校园摔倒检测数据集，包含站立与摔倒两类行为，共 4213 张标注样本，覆盖跑道、篮球场、室内三类场景。数据集按训练集:验证集:测试集 = 7:2:1 划分，用于验证模型在校园场景中的摔倒检测性能。

场景类别	训练集	验证集	测试集	总数
跑道 Runway	598	181	81	860
篮球场 Basketball court	1253	323	181	1757
室内 Indoor scene	2226	628	318	3172
合计	2949	842	422	4213
注：上表依据论文 Table 1 整理，其中分项合计与总样本数存在排版错位可能，使用前建议核对原始数据。

3.2 公开数据集补充
为增强模型可比性与泛化能力，本研究还结合了两个公开摔倒检测数据集：

LE2I with Upright and Fall
链接：https://universe.roboflow.com/new-workspace-qfcus/le2i-with-upright-and-fall/dataset/2
该数据集基于原始 LE2I 基准，包含室内场景中的站立与摔倒姿态，覆盖日常活动与突然摔倒。

Falldown Detection F8XTB
链接：https://universe.roboflow.com/kid-g8rt3/falldown-detection-f8xtb/dataset/1
该数据集主要包含户外体育场、运动场等场景的标注图像，用于补充场景、光照、遮挡与人体姿态多样性。

自建数据集下载链接：待补充。
公开数据集可从上列 Roboflow 链接获取。

实验环境配置
4.1 硬件环境
处理器：Intel Core i9-14900

显卡：NVIDIA GeForce RTX 5060 Laptop GPU

操作系统：Windows 11 x64

显存建议：≥8GB NVIDIA GPU

单帧推理：约 0.7 ms

内存占用：约 480 MB

4.2 软件环境
Python：3.11

开发工具：PyCharm 2023.3.3

深度学习框架：PyTorch 2.8.0

CUDA：12.9

cuDNN：9.10.2

4.3 依赖安装
推荐使用 Anaconda 创建虚拟环境：

bash
conda create -n improved-yolov12 python=3.11
conda activate improved-yolov12
安装 PyTorch、TorchVision，请根据实际 CUDA 版本调整：

bash
pip install torch torchvision --index-url https://download.pytorch.org/whl/cu129
安装其他依赖：

bash
pip install numpy pandas scipy matplotlib seaborn opencv-python pillow tqdm thop pyyaml
安装 YOLOv12 所需依赖时，请根据实际代码库中的 requirements.txt 进行安装。若使用 flash-attention，需确保其版本与 PyTorch、CUDA 严格匹配。

实验结果
5.1 核心指标
模型	mAP@50 (%)	mAP@[0.5:0.95] (%)	说明
YOLOv8	—	—	对比模型
YOLOv9	—	—	对比模型
YOLOv10	—	—	对比模型
原始 YOLOv12	—	—	基线模型
Improved YOLOv12 + Decoupled CS Attention	—	—	消融模型
Improved YOLOv12 + Agent Attention	—	—	消融模型
Improved YOLOv12（本文）	99.3	93.3	双注意力协同
注：论文图中给出了训练损失、mAP 曲线与雷达图对比，但未在表格中列出全部基线数值。完整数值请以论文正文和图表为准。

5.2 与其他摔倒检测方法对比
文献	年份	方法	数据集	mAP
Soni et al.	2024	CABMNet	URFall	98.36%
Sanjalaw et al.	2025	Hybrid YOLOv8 + TST	DiverseFALL10500	95.00%
Ken et al.	2026	DistillH-Mamba	URFall, UMAFall	97.38%
本文	2026	Improved YOLOv12	Our dataset	99.30%
本文方法较 Hybrid YOLOv8 + TST 提升 4.30%，较 CABMNet 提升 0.94%。

5.3 混淆矩阵与可视化结果
改进模型在混淆矩阵中取得 90.76% 的整体准确率，优于 YOLOv8、YOLOv9、YOLOv10 分别 6.87%、3.08%、1.19%。模型在 Stand 与 Fall 之间的混淆最少，并降低了背景误报。可视化检测结果显示，在室内走廊、室外跑道、篮球场、密集人群与运动模糊等场景中，模型均能较准确识别站立与摔倒状态，多数置信度超过 0.84，部分场景超过 0.90。

代码使用说明
6.1 模型训练
建议运行 train.py 脚本启动训练。脚本加载基础 YOLOv12 模型，并集成简化卷积、Decoupled CS Attention 与 Agent Attention 模块。

示例命令：

bash
python train.py \
  --data ./fall_detection.yaml \
  --epochs 100 \
  --batch 8 \
  --lr0 0.001 \
  --optimizer SGD \
  --save_dir ./runs/improved_yolov12 \
  --device cuda:0
关键参数说明：

参数名	含义	默认值
--data	数据集配置文件路径	./fall_detection.yaml
--epochs	训练轮数	100
--batch	批次大小	8
--lr0	初始学习率	0.001
--optimizer	优化器，支持 SGD / Adam / AdamW	SGD
--save_dir	模型保存目录	./runs/exp
--device	训练设备，cuda:0 或 cpu	cuda:0
训练策略：SGD 优化器带动量，余弦退火学习率，从 0.001 衰减至 
0.001
×
lrf
0.001×lrf，共 100 轮。训练过程中建议每 25 轮保存一次模型，最终最佳模型保存为 best.pt，位于 save_dir/train/weights/ 下。

6.2 模型预测
使用训练好的权重进行单张图像预测：

bash
python predict.py \
  --image_path ./images/test/basketball146.jpg \
  --model_path ./weights/best.pt \
  --device cuda:0
预测输出建议包含：

Parameters

FLOPs

Inference Speed

预测类别与置信度

mAP@50、mAP@[0.5:0.95]、Precision、Recall、F1

示例输出格式：

text
📊 PERFORMANCE METRICS RESULTS:
• Model Size: 5.4 MB
• Memory: ~480 MB
• Inference Speed: 0.7 ms per image (batch=1, cuda:0)

🎯 PREDICTION RESULT:
Most likely class: standing
Probability: 0.894
6.3 预训练权重
预训练权重 best.pt 可放置于 weights/ 目录下，用于预测或微调。
下载链接：待补充。

适用场景：校园场景中的 standing 与 fall 两类分类。若需扩展其他动作类别或类似场景，建议基于该权重微调，可冻结部分主干网络，仅训练检测头，以减少训练数据需求。

项目文件结构
text
Improved-YOLOv12-Fall-Detection/
├── data/
│   └── fall_detection/
│       ├── images/
│       │   ├── train/
│       │   ├── val/
│       │   └── test/
│       └── labels/
│           ├── train/
│           ├── val/
│           └── test/
├── models/
│   ├── yolo_simplified.py              # 简化 YOLOv12 卷积替换实现
│   ├── yolo_Decoupled_CS_Attention.py  # Decoupled CS Attention 模块
│   ├── yolo_Agent_Attention.py         # Agent Attention 模块
│   └── yolo_improved.py                # 主模型，整合上述模块
├── weights/
│   └── best.pt                         # 最优模型权重
├── train.py                            # 训练脚本
├── predict.py                          # 预测与性能评估脚本
├── fall_detection.yaml                 # 数据集配置文件
├── requirements.txt                    # 依赖包列表
└── README.md                           # 项目说明文档
已知问题与注意事项
数据集适配：当前模型与权重主要针对 standing 和 fall 两类场景。若需扩展行走、跑步、球类动作等类别，需补充对应数据集并微调。

自建数据集公开性：论文中未提供自建数据集下载链接，如需复现请联系作者或等待后续开源。

CUDA 版本问题：若 PyTorch 与 CUDA 不兼容，可替换为 CPU 版本，并将 --device 改为 cpu，但训练和推理效率会显著下降。

显存占用：若显存不足，建议将 batch 设为 8 或更低，并通过梯度累积模拟更大 batch，或适当降低 imgsz。

功能限制：模型只能检测摔倒事件，无法区分扭伤、骨折等受伤类型。

场景限制：当前数据缺少游泳、击剑、攀岩、低光、雨雾反射等特殊场景，极端天气与低照度下的鲁棒性仍需验证。

表格数值核对：论文 Table 1 分项合计与总样本数存在排版错位可能，复现前请核对原始数据。

