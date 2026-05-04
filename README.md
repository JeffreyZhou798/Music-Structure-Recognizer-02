<div align="center">

# 🎵 Music Structure Recognizer

**Zero-Shot Classical Music Form Analysis via Form-NN + Viterbi Dynamic Programming**

[![ModelScope](https://img.shields.io/badge/ModelScope-Live_Demo-blue)](https://www.modelscope.cn/studios/JeffreyZhou2026/MusicStructureRecognizer-05/)
[![Python](https://img.shields.io/badge/Python-3.10-green)](https://www.python.org/)
[![License](https://img.shields.io/badge/License-MIT-yellow)](LICENSE)

</div>

---

## 🌐 Try It Online

👉 **[Click here to try the live demo on ModelScope](https://www.modelscope.cn/studios/JeffreyZhou2026/MusicStructureRecognizer-05/)**

Upload any `.musicxml` / `.mxl` file and get instant form analysis with color-annotated scores.

---

## 📖 Overview

Music Structure Recognizer is a **zero-shot** symbolic music form analysis tool designed for **classical music**. Given a MusicXML score, it identifies the overall form type (e.g., Sonata, Rondo, Ternary) and segments the piece into structural sections (Exposition, Development, Recapitulation, Coda, etc.) with color-coded annotations.

### Key Features

- 🎼 **Zero-Shot Inference** — No training data or fine-tuning required; powered by pre-trained [Form-NN](https://github.com/danielathome19/Form-NN) weights
- ⚡ **Viterbi Dynamic Programming** — Deterministic, reproducible sequence optimization (< 0.5s)
- 🎨 **Color-Coded Score Annotation** — HSL dual-encoding: Hue → functional role, Lightness → relational status
- 🌍 **Bilingual UI** — English / Chinese switchable interface (Gradio)
- 📄 **Downloadable Outputs** — Annotated `.musicxml` score + Markdown analysis report
- 🚀 **CPU-Friendly** — Runs on free-tier cloud CPU (10–20s total pipeline)

---

## 🔄 Pipeline Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                     User Upload (.musicxml / .mxl)              │
└──────────────────────────┬──────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│  Stage 1: Preprocessing (music21)                               │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  Parse MusicXML → Extract Features (aligned with Form-NN│    │
│  │  data_utils) → Compute Boundaries & Melodic Features    │    │
│  └─────────────────────────────────────────────────────────┘    │
└──────────────────────────┬──────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│  Stage 2: Form-NN Inference                                     │
│  ┌──────────────────────┐  ┌────────────────────────────────┐   │
│  │  Form Analyzer       │  │  Phrase Analyzer               │   │
│  │  (TreeGrad/LightGBM) │  │  (Bi-LSTM + Decision Tree)     │   │
│  │                      │  │                                 │   │
│  │  Output:             │  │  Output:                        │   │
│  │  • Form probability  │  │  • Per-phrase label probs       │   │
│  │    distribution      │  │  • predict_proba matrix         │   │
│  │  • e.g. Sonata=0.94 │  │  • e.g. (A,0.7), (B,0.6)...   │   │
│  └──────────────────────┘  └────────────────────────────────┘   │
└──────────────────────────┬──────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│  Stage 3: Viterbi Dynamic Programming Optimization              │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  Emission probs = Form-NN predict_proba                  │    │
│  │  Transition costs = Mathematical smoothness penalty      │    │
│  │  Self-transition bonus = 3.0 (encourages segment cont.) │    │
│  │                                                          │    │
│  │  → Optimized label sequence + confidence scores          │    │
│  │  → Label collapse detection → Feature-based fallback     │    │
│  └─────────────────────────────────────────────────────────┘    │
└──────────────────────────┬──────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│  Stage 4: Post-Processing                                       │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  Label Smoothing → Consecutive Merge → Macro Merge       │    │
│  │  → Form-Aware Segmentation → Natural Language Naming     │    │
│  │  → Auto Numbering (Theme 1, Theme 2, Variation 1...)    │    │
│  └─────────────────────────────────────────────────────────┘    │
└──────────────────────────┬──────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│  Stage 5: Output Generation                                     │
│  ┌───────────────────────┐  ┌───────────────────────────────┐   │
│  │  Color-Annotated      │  │  Analysis Report              │   │
│  │  Score (.musicxml)    │  │  (Markdown .md)               │   │
│  │                       │  │                                │   │
│  │  • HSL color-coded    │  │  • Overall form type           │   │
│  │  • Natural language   │  │  • Segment table               │   │
│  │    TextBox labels     │  │  • Confidence scores           │   │
│  │  • MuseScore compat.  │  │  • Color legend                │   │
│  └───────────────────────┘  └───────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🎨 Color Encoding System

The annotation uses **HSL dual-encoding** to convey both functional role and relational status:

### Hue → Functional Role

| Color | Hue | Role | Typical Forms |
|-------|-----|------|---------------|
| 🔴 Red | 0° | Primary Theme, Exposition, Recapitulation, Refrain | Sonata, Ternary, Rondo |
| 🌹 Rose | 15° | Variation | Theme & Variations |
| 🟠 Orange | 30° | Transition, Bridge, Closing | Sonata, Rondo |
| 🟡 Yellow | 60° | Introduction, Prelude | All forms |
| 🟢 Green | 120° | Development | Sonata |
| 🟣 Purple | 270° | Secondary Theme, Episode, Trio, Couplet | Sonata, Rondo, Minuet |
| ⚫ Gray | 0 (sat=0) | Coda, Codetta, Ending | All forms |

### Lightness → Relational Status

| Lightness | Status | Meaning |
|-----------|--------|---------|
| Bright (0.78+) | Similar/Repetition | Thematic return or repetition |
| Medium (0.68) | Primary statement | Main presentation |
| Dim (0.58) | Contrast/Independent | Contrasting material |
| Dark (0.40) | Coda/Ending | Concluding section |

---

## 🏗️ Project Structure

```
Music-Structure-Recognizer/
├── app.py                    # Gradio 6.8.0 web interface
├── form_nn_wrapper.py        # Form-NN model loading & inference
├── musicxml_processor.py     # music21 parsing & feature extraction
├── structure_optimizer.py    # Viterbi DP + macro merge + form-aware segmentation
├── annotator.py              # Color-coded MusicXML annotation
├── Weights/                  # Pre-trained Form-NN weights (.hdf5, .pkl)
├── requirements.txt          # Python dependencies
├── setup.sh                  # Tsinghua mirror acceleration (China)
└── README.md
```

---

## 🚀 Quick Start

### Prerequisites

- Python 3.10
- Pre-trained Form-NN weights (download from [Form-NN repo](https://github.com/danielathome19/Form-NN) and place in `Weights/`)

### Installation

```bash
# Clone the repository
git clone https://github.com/JeffreyZhou2026/Music-Structure-Recognizer.git
cd Music-Structure-Recognizer

# Install dependencies
pip install -r requirements.txt

# Launch the application
python app.py
```

### Dependencies

```
gradio==6.8.0
music21>=9.0.0
numpy
pandas
joblib
lxml
tensorflow-cpu==2.15.0
scipy
scikit-learn
Pillow
lightgbm
```

> **Note:** `treegrad` is NOT required. A compatibility stub (`_CompatTGDClassifier`) is built into `form_nn_wrapper.py` to deserialize pre-trained models without the original `treegrad` package.

---

## 📊 Example Output

**Input:** Mozart Piano Sonata K.545, 1st Movement

**Output:**

| Start Measure | End Measure | Label | Relation | Color | Confidence |
|:---:|:---:|:---:|:---:|:---:|:---:|
| 1 | 25 | Exposition | Similar/Repetition | Red (Bright) | 0.50 |
| 26 | 35 | Development | Contrast/Independent | Green | 0.50 |
| 36 | 50 | Recapitulation | Similar/Repetition | Red (Bright) | 0.50 |
| 51 | 73 | Coda | Independent | Gray | 0.50 |

**Overall Form:** Sonata Form (Confidence: 0.94)

---

## 🔧 Technical Highlights

### Gamma Parameter (Boundary Modulation Strength)
- **Boundary modulation strength** - Controls the influence of MSA (Music Structure Annotator) boundary probabilities on the Viterbi algorithm
- **Range**: 0.0 - 1.0 (default 0.5)
- **Effect**: Higher values give more weight to boundary detection for segmentation decisions
- **Emission probability smoothing**: Uses alpha parameters (0.25/0.15) to reduce sharp distributions, making gamma more effective

### Probability Vector Post-Processing
- Automatically normalizes abnormal probability vectors, ensuring sum equals 1.0
- Detects and handles cases where all label probabilities are identical (uniform distribution)
- Improves Phrase-NN output quality and stability

### MSA Boundary Probability Injection
- Integrates MSA boundary detection results into Viterbi dynamic programming
- Reduces self-transition bonus at boundary positions, encouraging segmentation at strong boundaries
- Boundary probability edge decay mechanism (edge_decay_range=16, steepness=4.0)

### Why Viterbi over MCTS?

| Aspect | MCTS | Viterbi |
|--------|------|---------|
| **Determinism** | Stochastic (random rollouts) | Fully deterministic |
| **Reproducibility** | Different results per run | Identical results per run |
| **Time Complexity** | Unbounded (depends on simulations) | O(N × K²), < 0.5s |
| **Optimality** | Approximate | Globally optimal path |
| **Rule Dependency** | May require hand-crafted heuristics | Pure mathematical smoothing |

### Form-NN Architecture

Form-NN is a hybrid Neural Network + Decision Tree system:
- **Form Analyzer:** TreeGrad (LightGBM) → predicts overall form type with probability distribution
- **Phrase Analyzer:** Bi-LSTM feature extractor + Decision Tree classifier → predicts per-phrase labels with probability matrix

### Compatibility Features

- `_CompatTGDClassifier`: Replaces `treegrad.TGDClassifier` at deserialization time, delegating to internal `LGBMClassifier`
- `_SklearnCompatUnpickler`: Handles sklearn version mismatch (0.23.2 → 1.5.2) during model loading
- `tensorflow-cpu`: Avoids pulling multi-GB GPU packages on CPU-only deployments

---

## 🌐 Deployment on ModelScope (China)

The project is optimized for ModelScope Studio with China-mainland acceleration:

```bash
# setup.sh — Configure Tsinghua PyPI mirror
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
pip config set install.trusted-host pypi.tuna.tsinghua.edu.cn
pip install -r requirements.txt
```

**Recommended Image:** `ubuntu22.04-py310-torch2.3.1-modelscope1.21.0`

---

## 📜 Supported Form Types

| Form | Sections | Example |
|------|----------|---------|
| **Sonata** | Exposition → Development → Recapitulation → Coda | Mozart K.545 |
| **Ternary (ABA)** | A → B → A' → Coda | Chopin Nocturnes |
| **Binary** | A → A' | Bach Inventions |
| **Rondo** | Refrain → Couplet 1 → Refrain → Couplet 2 → Refrain → Coda | Beethoven Pathétique |
| **Minuet & Trio** | Minuet → Trio → Minuet Da Capo | Haydn Symphonies |
| **Theme & Variations** | Theme → Var.1 → Var.2 → Var.3 → ... | Mozart K.265 |

---

## 📄 License

This project is licensed under the MIT License. The Form-NN pre-trained weights are sourced from [danielathome19/Form-NN](https://github.com/danielathome19/Form-NN) (please refer to their license terms).

---

## 👤 Author

**Jeffrey Zhou**

---

<div align="center">

*Built with ❤️ for music education and computational musicology*

</div>

---
---

<div align="center">

# 🎵 音乐结构识别器

**基于 Form-NN + Viterbi 动态规划的零样本古典音乐曲式分析工具**

[![ModelScope](https://img.shields.io/badge/ModelScope-在线演示-blue)](https://www.modelscope.cn/studios/JeffreyZhou2026/MusicStructureRecognizer-05/)
[![Python](https://img.shields.io/badge/Python-3.10-green)](https://www.python.org/)
[![License](https://img.shields.io/badge/License-MIT-yellow)](LICENSE)

</div>

---

## 🌐 在线体验

👉 **[点击此处前往 ModelScope 在线试用](https://www.modelscope.cn/studios/JeffreyZhou2026/MusicStructureRecognizer-05/)**

上传任意 `.musicxml` / `.mxl` 文件，即可获得即时曲式分析与彩色标注乐谱。

---

## 📖 项目概述

音乐结构识别器是一款专为**古典音乐**设计的**零样本**符号音乐曲式分析工具。输入 MusicXML 乐谱，系统自动识别整体曲式类型（如奏鸣曲式、回旋曲式、三部曲式等），并将乐曲分割为结构段落（呈示部、展开部、再现部、尾声等），同时生成彩色标注乐谱。

### 核心特性

- 🎼 **零样本推理** — 无需训练数据或微调；基于预训练 [Form-NN](https://github.com/danielathome19/Form-NN) 权重
- ⚡ **Viterbi 动态规划** — 确定性、可复现的序列优化（< 0.5秒）
- 🎨 **HSL双重编码彩色标注** — 色相表示功能角色，明度表示关系状态
- 🌍 **中英双语界面** — Gradio 界面支持英文/中文即时切换
- 📄 **可下载输出** — 彩色标注 `.musicxml` 乐谱 + Markdown 分析报告
- 🚀 **CPU友好** — 免费云CPU即可运行（全流程10~20秒）

---

## 🔄 系统流程架构

```
┌─────────────────────────────────────────────────────────────────┐
│                     用户上传 (.musicxml / .mxl)                  │
└──────────────────────────┬──────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│  阶段 1: 预处理 (music21)                                       │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  解析MusicXML → 提取特征（严格对齐Form-NN的data_utils） │    │
│  │  → 计算乐句边界 & 旋律特征                               │    │
│  └─────────────────────────────────────────────────────────┘    │
└──────────────────────────┬──────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│  阶段 2: Form-NN 推理                                           │
│  ┌──────────────────────┐  ┌────────────────────────────────┐   │
│  │  曲式分析器           │  │  乐句分析器                     │   │
│  │  (TreeGrad/LightGBM) │  │  (Bi-LSTM + 决策树)             │   │
│  │                      │  │                                 │   │
│  │  输出:               │  │  输出:                          │   │
│  │  • 曲式概率分布       │  │  • 逐乐句标签概率               │   │
│  │  • 如: Sonata=0.94   │  │  • predict_proba概率矩阵        │   │
│  └──────────────────────┘  └────────────────────────────────┘   │
└──────────────────────────┬──────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│  阶段 3: Viterbi 动态规划优化                                    │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  发射概率 = Form-NN predict_proba                        │    │
│  │  转移代价 = 数学平滑惩罚（无手动音乐规则）                │    │
│  │  自转移奖励 = 3.0（鼓励段落连续性）                       │    │
│  │                                                          │    │
│  │  → 优化标签序列 + 置信度评分                              │    │
│  │  → 标签坍塌检测 → 特征分割回退机制                        │    │
│  └─────────────────────────────────────────────────────────┘    │
└──────────────────────────┬──────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│  阶段 4: 后处理                                                 │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  标签平滑 → 连续合并 → 宏观合并                           │    │
│  │  → 曲式感知分割 → 自然语言命名                             │    │
│  │  → 自动编号 (Theme 1, Theme 2, Variation 1...)           │    │
│  └─────────────────────────────────────────────────────────┘    │
└──────────────────────────┬──────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│  阶段 5: 输出生成                                               │
│  ┌───────────────────────┐  ┌───────────────────────────────┐   │
│  │  彩色标注乐谱          │  │  分析报告                      │   │
│  │  (.musicxml)          │  │  (Markdown .md)               │   │
│  │                       │  │                                │   │
│  │  • HSL彩色编码         │  │  • 整体曲式类型                 │   │
│  │  • 自然语言TextBox标签 │  │  • 段落结构表格                 │   │
│  │  • 兼容MuseScore       │  │  • 置信度评分                   │   │
│  └───────────────────────┘  └───────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🎨 彩色编码系统

标注采用 **HSL双重编码**，同时传达功能角色和关系状态：

### 色相 → 功能角色

| 颜色 | 色相 | 功能角色 | 典型曲式 |
|------|------|---------|---------|
| 🔴 红色 | 0° | 主题、呈示部、再现部、主部 | 奏鸣曲式、三部曲式、回旋曲式 |
| 🌹 玫红 | 15° | 变奏 | 变奏曲式 |
| 🟠 橙色 | 30° | 过渡、连接、收束 | 奏鸣曲式、回旋曲式 |
| 🟡 黄色 | 60° | 引子、前奏 | 所有曲式 |
| 🟢 绿色 | 120° | 展开部 | 奏鸣曲式 |
| 🟣 紫色 | 270° | 副主题、插部、三声中部 | 奏鸣曲式、回旋曲式、小步舞曲 |
| ⚫ 灰色 | 0 (饱和度=0) | 尾声、收束 | 所有曲式 |

### 明度 → 关系状态

| 明度 | 状态 | 含义 |
|------|------|------|
| 亮色 (0.78+) | 相似/重复 | 主题回归或重复 |
| 中亮 (0.68) | 主要陈述 | 基准呈现 |
| 较暗 (0.58) | 对比/独立 | 对比性材料 |
| 深暗 (0.40) | 尾声/收束 | 收束段落 |

---

## 🏗️ 项目结构

```
Music-Structure-Recognizer/
├── app.py                    # Gradio 6.8.0 网页界面
├── form_nn_wrapper.py        # Form-NN 模型加载与推理
├── musicxml_processor.py     # music21 解析与特征提取
├── structure_optimizer.py    # Viterbi动态规划 + 宏观合并 + 曲式感知分割
├── annotator.py              # 彩色MusicXML标注
├── Weights/                  # 预训练Form-NN权重 (.hdf5, .pkl)
├── requirements.txt          # Python依赖
├── setup.sh                  # 清华源加速（中国大陆）
└── README.md
```

---

## 🚀 快速开始

### 前置条件

- Python 3.10
- 预训练 Form-NN 权重（从 [Form-NN仓库](https://github.com/danielathome19/Form-NN) 下载，放入 `Weights/` 目录）

### 安装

```bash
# 克隆仓库
git clone https://github.com/JeffreyZhou2026/Music-Structure-Recognizer.git
cd Music-Structure-Recognizer

# 安装依赖
pip install -r requirements.txt

# 启动应用
python app.py
```

### 依赖列表

```
gradio==6.8.0
music21>=9.0.0
numpy
pandas
joblib
lxml
tensorflow-cpu==2.15.0
scipy
scikit-learn
Pillow
lightgbm
```

> **注意：** 无需安装 `treegrad`。`form_nn_wrapper.py` 内置了 `_CompatTGDClassifier` 兼容存根类，可在不安装原版 `treegrad` 的情况下反序列化预训练模型。

---

## 📊 示例输出

**输入：** 莫扎特钢琴奏鸣曲 K.545 第一乐章

**输出：**

| 起始小节 | 结束小节 | 标签 | 关系 | 颜色 | 置信度 |
|:---:|:---:|:---:|:---:|:---:|:---:|
| 1 | 25 | 呈示部 | 相似/重复 | 红色(亮) | 0.50 |
| 26 | 35 | 展开部 | 对比/独立 | 绿色 | 0.50 |
| 36 | 50 | 再现部 | 相似/重复 | 红色(亮) | 0.50 |
| 51 | 73 | 尾声 | 独立 | 灰色 | 0.50 |

**整体曲式：** 奏鸣曲式（置信度：0.94）

---

## 🔧 技术亮点

### Gamma 参数（Boundary Modulation Strength）
- **边界调制强度** - 控制 MSA（Music Structure Annotator）边界概率对 Viterbi 算法的影响程度
- **范围**：0.0 - 1.0（默认 0.5）
- **效果**：值越高，边界检测对分段决策的影响越大
- **发射概率平滑**：通过 alpha 参数（0.25/0.15）降低尖锐分布，使 gamma 参数更加有效

### 概率向量后处理
- 自动归一化异常概率向量，确保概率总和为 1.0
- 检测并处理所有标签概率完全相同的情况（均匀分布）
- 提升 Phrase-NN 输出质量和稳定性

### MSA 边界概率注入
- 集成 MSA 边界检测结果到 Viterbi 动态规划中
- 在边界位置降低自转移奖励，鼓励在强边界处进行分段
- 边界概率边缘衰减机制（edge_decay_range=16，steepness=4.0）

### 为什么选择 Viterbi 而非 MCTS？

| 维度 | MCTS | Viterbi |
|------|------|---------|
| **确定性** | 随机（随机模拟） | 完全确定性 |
| **可复现性** | 每次运行结果不同 | 每次运行结果相同 |
| **时间复杂度** | 无上界（取决于模拟次数） | O(N × K²)，< 0.5秒 |
| **最优性** | 近似最优 | 全局最优路径 |
| **规则依赖** | 可能需要手工启发式规则 | 纯数学平滑 |

### Form-NN 架构

Form-NN 是一个混合神经网络 + 决策树系统：
- **曲式分析器：** TreeGrad (LightGBM) → 预测整体曲式类型及概率分布
- **乐句分析器：** Bi-LSTM 特征提取器 + 决策树分类器 → 预测逐乐句标签及概率矩阵

### 兼容性设计

- `_CompatTGDClassifier`：在反序列化时替换 `treegrad.TGDClassifier`，将推理委托给内部 `LGBMClassifier`
- `_SklearnCompatUnpickler`：处理 sklearn 版本不兼容（0.23.2 → 1.5.2）
- `tensorflow-cpu`：避免在CPU部署环境拉取数GB的GPU版本

---

## 🌐 ModelScope 部署（中国大陆）

项目已针对 ModelScope 创空间优化，支持中国大陆加速：

```bash
# setup.sh — 配置清华PyPI镜像源
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
pip config set install.trusted-host pypi.tuna.tsinghua.edu.cn
pip install -r requirements.txt
```

**推荐镜像：** `ubuntu22.04-py310-torch2.3.1-modelscope1.21.0`

---

## 📜 支持的曲式类型

| 曲式 | 段落结构 | 示例 |
|------|---------|------|
| **奏鸣曲式** | 呈示部 → 展开部 → 再现部 → 尾声 | 莫扎特 K.545 |
| **三部曲式 (ABA)** | A → B → A' → 尾声 | 肖邦夜曲 |
| **二部曲式** | A → A' | 巴赫创意曲 |
| **回旋曲式** | 主部 → 插部1 → 主部 → 插部2 → 主部 → 尾声 | 贝多芬悲怆 |
| **小步舞曲** | 小步舞曲 → 三声中部 → 小步舞曲反复 | 海顿交响曲 |
| **变奏曲式** | 主题 → 变奏1 → 变奏2 → 变奏3 → ... | 莫扎特 K.265 |

---

## 📄 许可证

本项目采用 MIT 许可证。Form-NN 预训练权重来源于 [danielathome19/Form-NN](https://github.com/danielathome19/Form-NN)（请参阅其许可条款）。

---

## 👤 作者

**Jeffrey Zhou**

---

<div align="center">

*用 ❤️ 为音乐教育与计算音乐学而构建*

</div>
