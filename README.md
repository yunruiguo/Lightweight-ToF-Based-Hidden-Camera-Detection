# Lightweight ToF-Based Hidden Camera Detection

Detect hidden and pinhole cameras using Time-of-Flight (ToF) sensor data with fine-tuned YOLOv5. This project leverages active infrared ToF imaging to identify characteristic lens reflections and provides a complete detection pipeline for intelligent security applications.

<img src="docs/assets/tof_detection_principle.png" alt="ToF pinhole camera detection principle" width="60%">

## 📋 Overview

### Why ToF Imaging?

Time-of-Flight (ToF) imaging technology actively emits modulated infrared light and measures reflections. This approach offers significant advantages:

- **Low ambient light sensitivity**: Robust detection independent of natural lighting conditions
- **High frame rates**: Real-time processing capabilities
- **Compact deployment**: Portable detection terminal design
- **Physical principle**: Pinhole camera lenses and image sensors produce distinctive cat-eye reflections under ToF illumination

### System Architecture

The detection system operates as an intelligent detection terminal with two integrated modules:

| Module | Function |
|--------|----------|
| **ToF Capture Module** | Illuminates target areas and collects optical/depth imagery |
| **Image Processing Module** | Runs trained YOLOv5 detection model on captured ToF images |

## 🔄 Detection Workflow

<img src="docs/assets/intelligent_terminal_flow.png" alt="Intelligent detection terminal workflow" width="60%">

1. **Capture**: Scan suspected area using the ToF module
2. **Inference**: Feed ToF image into YOLOv5 detector
3. **Analysis**: Generate candidate bounding boxes with class labels and confidence scores
4. **Decision**: Treat detections with confidence ≥ 0.5 as positive candidates
5. **Coverage**: For incomplete area coverage, move detector horizontally or vertically in a Z or S pattern while maintaining stable distance and orientation

**Result Example**: The figure below demonstrates detection performance across four test scenarios, comparing annotated ground truth (left) with model predictions and confidence scores (right):

<img src="docs/assets/detection_output_examples.png" alt="Detection output examples" width="60%">

---

### 🎨 Interactive Diagrams

For a complete visual understanding of the system architecture and performance metrics, please view our **interactive SVG diagrams**:

**[📊 View Interactive Diagrams](docs/diagrams.html)**

This comprehensive visualization includes:
- **Diagram 1**: ToF Detection Principle with sensor operation details
- **Diagram 2**: Complete Detection Workflow Pipeline showing all 5 processing steps
- **Diagram 3**: Performance Metrics Summary and test scenario results

These diagrams are optimized for web viewing and scale perfectly on any device.

---

## 📁 Repository Structure

```
.
├── configs/
│   └── spycamera.yaml                    # YOLOv5 dataset configuration
├── data/
│   ├── yolo/                             # Training/test dataset (YOLO format)
│   │   ├── images/
│   │   │   ├── train/
│   │   │   └── test/
│   │   └── labels/
│   │       ├── train/
│   │       └── test/
│   ├── tof_triplets/                     # Curated RGB, color, and depth modality triplets
│   └── raw_tof/                          # Original ToF captures and source annotations
├── docs/
│   ├── assets/                           # Original documentation figures and diagrams
│   └── diagrams.html                     # Interactive SVG visualizations
├── experiments/train_exp17/              # Preserved training metrics and evaluation plots
├── models/
│   └── spycamera_yolov5s_best.pt        # Fine-tuned YOLOv5 checkpoint
└── third_party/yolov5/                   # YOLOv5 training/inference runtime
```

## 📊 Dataset

### Classes

| ID | Class | Description |
|----|-------|-------------|
| 0 | positive | Hidden camera detected |
| 1 | negative | No camera present |

### Format

Training data in `data/yolo/` follows YOLO annotation format:

```
class_id x_center y_center width height
```

All coordinates and dimensions are normalized (0-1 range).

### Data Organization

- **Training-ready**: `data/yolo/` - Formatted for direct use with YOLOv5
- **Raw data**: `data/raw_tof/` - Original captures with complete modality sets (RGB, RGB color, IR, Depth) and annotations
- **Processed triplets**: `data/tof_triplets/` - Curated per-sample modality combinations

## 🚀 Getting Started

### Prerequisites

- Python 3.8+
- CUDA 11.0+ (recommended for GPU acceleration)
- GPU with at least 2GB VRAM (for training)

### Installation

```bash
cd third_party/yolov5
python -m venv .venv
source .venv/bin/activate  # On Windows: .venv\Scripts\activate
pip install -r requirements.txt
```

For existing PyTorch/CUDA environments, install requirements directly:

```bash
pip install -r third_party/yolov5/requirements.txt
```

## 🏋️ Training

Run training from the `third_party/yolov5/` directory:

```bash
python train.py \
  --img 640 \
  --batch 16 \
  --epochs 300 \
  --data ../../configs/spycamera.yaml \
  --weights yolov5s.pt \
  --name spycamera_yolov5s
```

**Configuration Tips**:
- Adjust `--batch` based on available GPU memory
- Increase `--epochs` for improved convergence (300+ recommended)
- Outputs are saved to `third_party/yolov5/runs/train/`

## ✅ Validation

Evaluate the model on the test dataset:

```bash
cd third_party/yolov5
python val.py \
  --data ../../configs/spycamera.yaml \
  --weights ../../models/spycamera_yolov5s_best.pt \
  --img 640
```

### Baseline Performance

The fine-tuned model achieves strong performance metrics (stored in `experiments/train_exp17/`):

| Metric | Score |
|--------|-------|
| Precision | 0.976 |
| Recall | 0.969 |
| mAP@0.5 | 0.976 |
| mAP@0.5:0.95 | 0.671 |

See **[Performance Visualization](docs/diagrams.html)** for graphical representation of these metrics.

## 🔍 Inference

### Test Set Inference

```bash
cd third_party/yolov5
python detect.py \
  --weights ../../models/spycamera_yolov5s_best.pt \
  --source ../../data/yolo/images/test \
  --img 640 \
  --conf 0.25
```

### Single Image Detection

```bash
python detect.py \
  --weights ../../models/spycamera_yolov5s_best.pt \
  --source path/to/image.png \
  --img 640 \
  --conf 0.25
```

**Output**: Results are saved with bounding boxes, class labels, and confidence scores.

## 📦 Repository Scope

This repository retains essential files for understanding, training, validating, and deploying the ToF hidden-camera detector:

- ✅ Complete dataset in YOLO format
- ✅ Configuration and training scripts
- ✅ Pre-trained model checkpoint
- ✅ Trimmed YOLOv5 runtime (unnecessary frameworks and dependencies removed)
- ✅ Experiment logs and metrics
- ✅ Interactive documentation and visualizations
- ❌ IDE metadata, cache files, and duplicate labels
- ❌ Framework experiments and unrelated archives

## 📖 Key References

- **YOLOv5**: https://github.com/ultralytics/yolov5
- **Time-of-Flight Imaging**: Comprehensive 3D sensing technology overview
- **Object Detection**: State-of-the-art computer vision methodology
- **Interactive Diagrams**: [View SVG Visualizations](docs/diagrams.html)

## 📄 License

Please refer to the repository's LICENSE file for terms of use.

## 🤝 Contributing

Contributions and improvements are welcome. Please follow these guidelines:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/improvement`)
3. Commit your changes (`git commit -am 'Add improvement'`)
4. Push to the branch (`git push origin feature/improvement`)
5. Open a Pull Request

## ❓ FAQ

**Q: Can I use this on a CPU?**  
A: Yes, but inference will be significantly slower. GPU acceleration is strongly recommended for real-time deployment.

**Q: What image resolutions does the model support?**  
A: The model is trained on 640×640 images. While other resolutions work, performance may vary.

**Q: How can I improve detection accuracy?**  
A: Consider training with more data, adjusting confidence thresholds, or fine-tuning model parameters.

**Q: Where can I view the system architecture diagrams?**  
A: Check out our interactive [SVG Diagrams](docs/diagrams.html) which include the ToF detection principle, workflow pipeline, and performance metrics.

---

**Last Updated**: 2026  
**Project Status**: Active
