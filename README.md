# 小钢珠检测工程 (Steel Ball Detection)

## 项目结构

```
steel_ball/
├── steel_ball.yaml           # YOLO 数据集配置
├── README.md
├── data/
│   ├── classes.txt           # 类别名
│   ├── images/               # ⚠ 未入库 (见下方说明)
│   │   ├── train/            # 104 张训练图 (灰度, 未入库)
│   │   └── val/              # 26 张验证图 (灰度, 未入库)
│   └── labels/
│       ├── train/            # 104 个标注文件
│       └── val/              # 26 个标注文件
├── scripts/
│   ├── annotate.py           # 标注工具 (OpenCV 版, 替代 labelImg)
│   ├── detect_cv.py          # [备用] 传统 CV 霍夫圆检测
│   ├── detect_yolo.py        # YOLO 实时检测 (训练后使用)
│   ├── collect_data.py       # 采集图片
│   ├── train.py              # YOLO 训练
│   └── split_data.py         # 划分数据集
├── runs/
│   └── steel_ball/weights/
│       ├── best.pt           # ⭐ 最佳模型 (mAP50=0.995, 未入库)
│       └── last.pt           # 最后一轮模型 (未入库)
└── steel_ball.yaml
```

> **⚠ 图像与权重未入库**
>
> `.gitignore` 排除了 `data/images/`（130 张灰度原图）和 `runs/`（训练输出 + 权重），
> 所以本仓库只包含 **标注文件 + 脚本 + 配置**，体积很小。
> 直接 clone 下来**无法**立即训练或检测，需要自备/自采：
>
> - **图片**：用 `scripts/collect_data.py` 采集，或自采后按 `data/labels/` 的文件名放进
>   `data/images/train/` 与 `data/images/val/`（图与标签同名、一一对应）
> - **模型权重**：自备 `best.pt` 放到 `runs/steel_ball/weights/`，或先跑 `scripts/train.py` 自己训一个
>
> 标注格式为标准 YOLO（`class cx cy w h`，归一化），可直接用于 Ultralytics。

## 训练结果

| 指标 | 数值 | 说明 |
|------|------|------|
| 训练集 | 104 张 / 704 个框 | 80% 划分 |
| 验证集 | 26 张 / 103 个框 | 20% 划分 |
| 基础模型 | yolo11n.pt | Ultralytics 预训练 |
| 训练轮数 | 150 epochs | CPU 约 50 分钟 |
| **mAP50** | **0.995** | 接近完美 |
| 精确率 (P) | 0.997 | 几乎零误检 |
| 召回率 (R) | 1.000 | 零漏检 |
| 推理速度 | ~31 ms/张 | CPU 约 32 FPS，实时流畅 |
| 模型大小 | 5.5 MB | 轻量 |

## 环境

| 项目 | 版本/路径 |
|------|-----------|
| Python | `.venv/Scripts/python.exe`（仓库根目录下的本地虚拟环境） |
| ultralytics | 8.4.108 |
| torch | 2.13.0 (CPU) |
| opencv | 5.0.0 |

## 运行命令

以下路径均相对于仓库根目录。先在本地克隆的 `steel-ball-yolo/` 根目录打开终端；Windows 命令提示符（cmd）示例：

```bat
REM 首次使用时，在仓库根目录创建虚拟环境并安装依赖
python -m venv .venv
.venv\Scripts\python.exe -m pip install ultralytics opencv-python

REM 实时检测 (默认)
.venv\Scripts\python.exe scripts/detect_yolo.py

REM 提高置信度阈值
.venv\Scripts\python.exe scripts/detect_yolo.py --conf 0.5

REM 单张图片检测（test.jpg 为自行提供、放在仓库根目录的测试图片）
.venv\Scripts\python.exe scripts/detect_yolo.py --image test.jpg
```

| 按键 | 功能 |
|------|------|
| Q / ESC | 退出 |
| S | 截图 |

## 关键说明

### 为什么摄像头输入要转灰度？
模型训练时用的是灰度图，推理时如果不转灰度，彩色输入和训练数据分布不一致，会导致置信度低、误检多。`detect_yolo.py` 已内置自动灰度转换。

### best.pt vs yolo11n.pt
- `yolo11n.pt` — 官方预训练模型，识别 80 类通用物体（无钢珠）
- `best.pt` — 在你的 130 张钢珠数据上微调后，专用于钢珠检测

### 标注工具
labelImg 和 PyQt5 有兼容性问题，使用 `scripts/annotate.py` 替代（同样在仓库根目录运行）：
```bat
.venv\Scripts\python.exe scripts/annotate.py
```
操作：鼠标拖拽画框 | 空格下一张 | A上一张 | Delete清框 | Z撤销 | ESC保存退出

### OpenCV 中文路径问题
OpenCV 的 `imread` 不支持中文路径，统一使用 `np.fromfile + cv2.imdecode` 替代。

## 完整流程回顾

```
拍照(手机130张) → 灰度预处理 → annotate.py标注(704框)
→ 划分训练/验证(8:2) → train.py训练(150轮)
→ best.pt(99.5% mAP) → detect_yolo.py实时检测 ✅
```
