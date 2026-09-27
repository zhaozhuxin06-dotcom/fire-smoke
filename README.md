# 🔥 PP-SaveBot 火焰烟雾与裂缝检测系统

[![Python 3.8+](https://img.shields.io/badge/Python-3.8+-blue.svg)](https://www.python.org/downloads/)
[![PaddlePaddle](https://img.shields.io/badge/PaddlePaddle-2.4+-green.svg)](https://www.paddlepaddle.org.cn/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

## 🎯 项目简介

PP-SaveBot是一个基于PaddlePaddle的高性能多场景检测系统，专为蚁群机器人协同检测场景设计。系统包含两个**完全独立**的检测模型，可拆分使用，支持机器人团队各司其职进行专业化检测任务。

### 🚀 核心功能

| 检测模块 | 目标类型 | 模型架构 | 适用场景 |
|---------|----------|----------|----------|
| **FIRESMOKE** | 🔥火焰、💨烟雾 | RT-DETR | 火灾预警、安防监控 |
| **DEEPCRACK** | 🛣️道路裂缝 | OCRNet | 基础设施巡检、道路维护 |

## 📁 项目结构

```
PP-SaveBot/
├── 📂 FIRESMOKE/                  # 火焰烟雾检测模块
│   ├── 📂 config/               # 配置文件
│   │   └── config.yaml          # 火焰烟雾检测配置
│   ├── 📂 datasets/               # 数据集目录
│   │   ├── images/              # 图像数据
│   │   └── labels/              # YOLO格式标注
│   ├── 📂 src/                  # 核心源码
│   │   ├── data/                # 数据处理
│   │   ├── models/              # 模型定义
│   │   ├── training/            # 训练逻辑
│   │   └── utils/               # 工具函数
│   ├── evaluate_enhanced.py     # 增强版评估脚本
│   ├── train.py                 # 训练入口
│   └── requirements.txt         # 依赖列表
│
├── 📂 DEEPCRACK/                # 裂缝检测模块
│   ├── 📂 config/               # 配置文件
│   ├── 📂 datasets/             # 裂缝检测数据集
│   ├── 📂 outputs/              # 输出目录
│   ├── 📂 scripts/              # 实用脚本
│   ├── 📂 src/                  # 核心源码
│   ├── evaluate.py              # 评估脚本
│   └── train_model.py           # 训练脚本
│
├── 📂 shared_utils/             # 共享工具（可选）
└── README.md                    # 项目文档
```

## 🔧 环境配置与安装

### 📋 系统要求

- **操作系统**: Windows 10/11, Ubuntu 18.04+, macOS 10.14+
- **Python**: 3.8-3.10
- **CUDA**: 11.2+ (GPU加速)
- **内存**: 8GB+ (推荐16GB)
- **存储**: 10GB+ 可用空间

### 🛠️ 安装步骤

#### 1. 环境检查
```bash
# 进入项目目录
cd PP-SaveBot

# 运行环境检查脚本
python DEEPCRACK/scripts/check_env.py
```

#### 2. 创建虚拟环境
```bash
# 使用conda创建环境
conda create -n ppsavebot python=3.8
conda activate ppsavebot

# 或使用venv
python -m venv ppsavebot_env
source ppsavebot_env/bin/activate  # Linux/Mac
# 或
ppsavebot_env\Scripts\activate     # Windows
```

#### 3. 安装依赖

```bash
pip install -r requirements.txt
```
> 该依赖文件的内容可能不全面，建议根据实际运行情况调整。

#### 4. 验证安装
```bash
# 测试FIRESMOKE模块
python -c "from src.models.rt_detr import RTDETRDetector; print('FIRESMOKE模块加载成功')"

# 测试DEEPCRACK模块
python -c "from src.models.ocrnet import OCRNet; print('DEEPCRACK模块加载成功')"
```

## 🎯 使用指南

### 🏗️ 数据准备

#### FIRESMOKE数据集格式
```
datasets/
└── images/
    ├── train/
    │   ├── image1.jpg
    │   └── image2.png
    ├── val/
    └── test/
└── labels/
    ├── train/
    │   ├── image1.txt  # YOLO格式: class x_center y_center width height
    │   └── image2.txt
    ├── val/
    └── test/
```

#### DEEPCRACK数据集格式
```
datasets/
├── train/
│   ├── image/
│   │   ├── img1.jpg
│   │   └── img2.jpg
│   └── gt/
│       ├── img1.png
│       └── img2.png
├── valid/
└── test/
```

### 🔥 FIRESMOKE模块使用

#### 训练模型
```bash
cd FIRESMOKE
python train.py \
    --config config/config.yaml \
    --device gpu \
    --output-dir outputs/firesmoke_model
```

#### 评估模型
```bash
python evaluate_enhanced.py \
    --model outputs/models/checkpoints/best_model.pdparams \
    --config config/config.yaml \
    --data_dir datasets/images/test \
    --output_dir outputs/firesmoke_eval \
    --max_visualizations 10 \
    --confidence_threshold 0.5
```

#### 单张图像推理
```python
from src.inference.predictor import FireSmokePredictor

predictor = FireSmokePredictor(
    model_path='outputs/firesmoke_model/best_model.pdparams',
    config_path='config/config.yaml'
)

result = predictor.predict('test_image.jpg')
print(f"检测到火焰: {result['fire_count']}, 烟雾: {result['smoke_count']}")
```

### 🛣️ DEEPCRACK模块使用

#### 训练模型
```bash
cd DEEPCRACK
python train_model.py \
    --config config/config.yaml \
    --data_dir datasets \
    --output_dir outputs/deepcrack_model \
    --device gpu
```

#### 评估模型
```bash
python evaluate.py \
    --model outputs/deepcrack_model/best_model.pdparams \
    --data_dir datasets \
    --output_dir outputs/deepcrack_eval \
    --threshold 0.5 \
    --save_predictions
```

#### 批量检测
```python
from src.inference.predictor import CrackPredictor

predictor = CrackPredictor(
    model_path='outputs/deepcrack_model/best_model.pdparams',
    config_path='config/config.yaml'
)

# 批量处理
processor = BatchProcessor(predictor)
results = processor.process_directory('test_images/', save_dir='results/')
```

## 🤖 蚁群机器人部署方案

### 🏭 独立部署模式

#### 方案A：专用检测机器人
```python
# 火焰检测专用机器人
from FIRESMOKE.src.models.rt_detr import RTDETRDetector

class FireDetectionBot:
    def __init__(self):
        self.detector = RTDETRDetector(config)
        self.detector.load('firesmoke_model.pdparams')
    
    def patrol_and_detect(self):
        # 火焰检测专用逻辑
        pass

# 裂缝检测专用机器人  
from DEEPCRACK.src.models.ocrnet import OCRNet

class CrackDetectionBot:
    def __init__(self):
        self.detector = OCRNet(config)
        self.detector.load('deepcrack_model.pdparams')
    
    def inspect_infrastructure(self):
        # 裂缝检测专用逻辑
        pass
```

#### 方案B：任务分发系统
```python
class TaskDispatcher:
    def __init__(self):
        self.fire_bots = [FireDetectionBot() for _ in range(3)]
        self.crack_bots = [CrackDetectionBot() for _ in range(2)]
    
    def assign_task(self, task_type, location):
        if task_type == 'fire_detection':
            bot = self.select_optimal_bot(self.fire_bots)
        elif task_type == 'crack_detection':
            bot = self.select_optimal_bot(self.crack_bots)
        return bot.execute_task(location)
```

### 📊 性能指标

| 模块 | 模型大小 | 推理速度 | 检测精度 | 内存占用 |
|------|----------|----------|----------|----------|
| FIRESMOKE | 95MB | 21.4 FPS | mAP: 0.85 | 2.1GB |
| DEEPCRACK | 45MB | 35.2 FPS | IoU: 0.78 | 1.8GB |

## 🛠️ 高级配置

### 🔧 自定义训练参数

#### FIRESMOKE配置示例
```yaml
# config/custom_fire_config.yaml
model:
  type: "rt_detr"
  backbone: "rt_detr_r50vd"
  num_classes: 2  # 火焰+烟雾
  
dataset:
  image_size: [640, 640]
  augmentation:
    fire_smoke_specific: true
    hsv_h: 0.01  # 火焰颜色敏感
    rotation: 5.0

training:
  epochs: 100
  batch_size: 8
  lr: 0.001
```

#### DEEPCRACK配置示例
```yaml
# config/custom_crack_config.yaml
model:
  type: "ocrnet"
  backbone: "resnet50"
  num_classes: 2  # 背景+裂缝
  
dataset:
  image_size: [512, 512]
  augmentation:
    elastic_transform: true
    brightness: 0.2

training:
  epochs: 200
  batch_size: 4
  lr: 0.0005
```

## 📈 模型优化建议

### 🔥 FIRESMOKE优化
- **小目标检测**: 使用多尺度训练
- **实时性**: 启用TensorRT加速
- **精度**: 增加数据增强强度

### 🛣️ DEEPCRACK优化
- **细节检测**: 使用高分辨率输入
- **边缘精度**: 启用边缘增强
- **泛化性**: 增加多样化训练数据

## 🚨 常见问题

### 两个模块能否同时运行？
可以独立运行，建议在不同GPU上并行执行以避免资源冲突。

### 如何部署到边缘设备？

1. 使用Paddle Lite进行模型转换
2. 针对ARM架构优化
3. 启用量化压缩

### 检测精度不理想怎么办？

1. 检查数据集标注质量
2. 调整置信度阈值
3. 增加训练数据量
4. 使用预训练模型微调

---

**⚠️ 重要声明**: FIRESMOKE和DEEPCRACK可以独立运行，也可以组合使用。根据实际需求选择使用任一模块或组合部署。我们的蚁群机器人设计上便是根据任务类型分配专用检测单元，各司其职，实现最优检测效率。
