---
title: "SAIE2026 — Object Detection & Edge Optimization Deep-Dive"
type: concept
tags:
  - object-detection
  - edge-ai
  - YOLO
  - model-optimization
  - computer-vision
  - pruning
  - quantization
  - knowledge-distillation
  - FPN
  - nano-device
status: evergreen
source: "SAIE2026_ObjectDetection_PanboonyuenLecture.pdf"
lecturer: "Teerapong Panboonyuen, Ph.D."
created: 2026-05-18
related:
  - "[[index]]"
  - "[[DragonScale Memory]]"
---

# SAIE2026 — Object Detection & Edge Optimization Deep-Dive

> Lecture by Teerapong Panboonyuen, Ph.D. — Super AI Engineer Thailand 2026 Workshop.
> 3 hours, 6 labs on Google Colab. This note is a full deep-dive for someone with no prior object detection background.

---

## The Overview

This workshop tackles a concrete industry problem: how do you run object detection on **4 cameras simultaneously** in real time, on a tiny embedded computer (NVIDIA Jetson Nano with only 4 GB RAM), while keeping accuracy high enough to be trustworthy? The answer is not "use a bigger computer." The answer is a stack of five engineering techniques — profiling, pruning, quantization, distillation, and architecture design — applied in sequence, with every decision backed by measured numbers.

**Key insight:** A model that is 10x slower than necessary is not just inconvenient — it makes entire applications *impossible*. The gap between "runs on a server" and "runs on four cameras on a drone" is bridged not by better hardware but by smarter optimization.

---

## The Detail

---

**What is object detection, and why is it hard?**
Pages: 1–3

Object detection is the task of answering two questions about an image at once: *what* objects are in it, and *where* exactly are they? This is different from simple image classification (which only asks "what is this image?"). For each detected object, the model outputs a **bounding box** — a rectangle defined by its top-left corner, width, and height — plus a **class label** (e.g., "car", "pedestrian") and a **confidence score** (how certain the model is).

The hard part is doing this fast enough for video. A video is just a sequence of images. At 25 frames per second (FPS) per camera, the model has 40 milliseconds to process one frame. With 4 cameras, it needs to process 100 images per second total. A large, accurate model might only manage 15 images per second — far too slow. The workshop exists to close this gap.

The specific hardware target is the **NVIDIA Jetson Nano** — a credit-card-sized computer with 4 GB of RAM and 128 CUDA cores. It is representative of industrial embedded boards found in drones, security cameras, factory robots, and smart city infrastructure.

*Example from the lecture:* The stated goal is 4 cameras × 25 FPS = 100 FPS total throughput, on Jetson Nano, with mAP (accuracy) ≥ 80% of the original baseline model.

---

**The Dataset: VisDrone**
Pages: 3–6

The workshop uses the **VisDrone** dataset — a publicly available benchmark of images captured by drone cameras at varying altitudes, lighting conditions, and weather. The SAIE 2026 subset has ~800 training images and ~200 validation images, 10 object classes (pedestrian, people, bicycle, car, van, truck, tricycle, awning-tricycle, bus, motor), and all images are resized to 416×416 pixels for training.

Two properties make VisDrone especially challenging and appropriate for this workshop:

1. **Tiny objects.** Pedestrians photographed from drone altitude may occupy as few as 8×8 pixels in the image. Standard detectors tuned for large objects simply miss them. Techniques that shrink or degrade the model must not destroy small-object performance.

2. **Class imbalance.** "Pedestrian" appears far more often than "bus." A naive model learns to mostly detect pedestrians and ignores rare classes. Metrics must account for this.

VisDrone is intentionally hard — it keeps all optimizations honest.

---

**Key Metrics: How We Measure Everything**
Pages: 7, 24, 26–27

Before optimizing anything, you must be able to *measure* it. The workshop tracks six numbers for every model.

**IoU — Intersection over Union** is the foundational building block. It measures how well a predicted bounding box overlaps the ground truth (the human-drawn correct box). IoU = Area of overlap / Area of union. A score of 1.0 means perfect overlap; 0 means no overlap at all. An IoU threshold of 0.5 means "a detection counts as correct only if the predicted box overlaps the ground truth box by at least 50%." (web: Ultralytics glossary)

**Precision and Recall** are computed per class, per confidence threshold. Precision = of all boxes the model predicted, what fraction were actually correct? Recall = of all real objects in the image, what fraction did the model find? These two are always in tension: push confidence threshold up and precision rises but recall falls (you miss more objects). Push it down and you catch more objects but get more false alarms.

**AP (Average Precision)** for one class is the area under its Precision-Recall curve, integrated from recall=0 to recall=1. Concretely: rank all detections by confidence (highest first), walk down the list computing precision and recall at each rank, then integrate. This gives a single number per class regardless of threshold choice.

**mAP@50** (Mean Average Precision at IoU=0.5) averages AP across all classes, using the IoU=0.5 threshold. This is the primary accuracy metric the workshop uses. A mAP@50 of 0.9+ means near-perfect detection. Below 0.5 means the model is mostly failing.

**mAP@50-95** (the COCO benchmark metric) is stricter — it averages mAP over 10 IoU thresholds from 0.5 to 0.95 in steps of 0.05. A model must produce very precise boxes to score well here.

**FLOPs** (Floating Point Operations) counts the raw number of mathematical operations (multiply + add pairs) in one forward pass. For a single convolutional layer: FLOPs = 2 × C_in × k² × C_out × H × W, where C_in/C_out are input/output channels, k is kernel size, and H×W is the feature map size. Total FLOPs for a model is the sum over all layers. Fewer FLOPs → faster inference. YOLOv8s is ~28 GFLOPs; YOLOv8n is ~8 GFLOPs (3.5× less).

**Latency** is the wall-clock time for one inference in milliseconds. Two measurement rules are critical: always warm up the GPU first (the first few calls are slow due to CUDA kernel compilation); always use `torch.cuda.synchronize()` before stopping the clock (the GPU runs asynchronously — without it you measure scheduling time, not computation). Report P95 (the 95th percentile worst-case), not just the mean, because a live video stream must never stall.

**FPS** (Frames Per Second) = 1000 / mean_latency_ms. Target: ≥ 25 FPS per camera.

**Efficiency** = mAP@50 / GFLOPs. How much accuracy do you get per unit of compute? Higher is better. Useful for fair comparison between models of different sizes.

---

**YOLO Architecture — How Object Detection Actually Works**
Pages: 8–10, 27, 38

YOLO (You Only Look Once) is the dominant family of real-time object detectors. Understanding how it works is essential context for every optimization technique in the workshop.

A traditional approach (like R-CNN) works in two stages: first propose candidate regions where objects might be, then classify each region. This is accurate but slow. YOLO takes a radically different approach — it solves both localization and classification in a **single forward pass** through one neural network, treating the problem as direct regression from pixels to box coordinates and class probabilities. (web: DataCamp)

Here is the core idea. The input image is conceptually divided into a grid (e.g., 52×52 cells for small objects). Each cell is responsible for detecting objects whose center falls within it. For each cell, the network predicts: (a) bounding box coordinates, (b) a confidence score, and (c) class probabilities. After the forward pass, a step called **non-maximum suppression (NMS)** removes duplicate boxes by keeping only the highest-confidence box among overlapping ones.

YOLOv8 (the version used in this workshop) is anchor-free: instead of predicting offsets from fixed "anchor" box templates, it learns a distribution over possible offsets using **Distribution Focal Loss (DFL)**. This makes it more flexible for the diverse shapes and sizes of drone-altitude objects.

The YOLOv8 family offers five sizes:

| Model | Parameters | GFLOPs | FPS (T4 GPU) | mAP@50 (COCO) |
|-------|-----------|--------|-------------|----------------|
| YOLOv8n | 3.2M | 8.7G | 160 | 0.37 |
| YOLOv8s | 11.2M | 28.6G | 120 | 0.45 |
| YOLOv8m | 25.9M | 78.9G | 80 | 0.50 |
| YOLOv8l | 43.7M | 165G | 50 | 0.53 |

The workshop uses **YOLOv8s** as the baseline (good balance for demonstration) and **YOLOv8n** as the edge target (fastest, fits Jetson Nano).

**The Pareto Frontier concept** is central to the whole workshop: on the frontier, you cannot improve accuracy without slowing the model, and you cannot speed up the model without losing accuracy. The engineering job is to find the optimal point on the mAP ↔ Speed ↔ Memory frontier given your specific constraints.

---

**Lab 1 — Baseline Profiling**
Pages: 23–27

The first principle of the workshop: *you cannot improve what you don't measure.* Before touching the model, establish a complete baseline with all six metrics (mAP@50, mAP@50-95, FLOPs, Parameters, Latency, FPS). This is your reference. Every subsequent optimization is only meaningful relative to this number.

Practical measurement notes:
- GPU warm-up: run 10–20 inference passes and discard them before timing.
- Use `torch.cuda.synchronize()` after every timed call.
- Collect 100+ measurements and report P50 (median) and P95 (worst case).
- FPS = 1000 / mean_latency. Target: ≥ 25 FPS per camera.

---

**Lab 2 — Structured Pruning: Removing What Doesn't Matter**
Pages: 28–30

A trained neural network has many redundant neurons — filters (channels) that barely change the output regardless of the input. These filters waste computation without contributing to accuracy. **Pruning** permanently removes them.

There are two kinds of pruning. **Unstructured pruning** sets individual weights to zero (like selectively erasing pencil marks). The problem: modern GPUs are not faster with sparse matrices — the matrix still has the same shape, so you don't actually save FLOPs in practice. **Structured pruning** removes entire channels (whole filters) from a layer. The resulting layer is genuinely smaller and physically faster on any hardware.

How do you identify which channels are safe to remove?

**Method 1 — L1-Norm Score.** For each filter, compute s_i = sum of absolute values of all its weights. A filter with a low L1-norm contributes little magnitude to the output. Rank all filters by score and remove the bottom p% (typically 20–40%).

**Method 2 — BatchNorm γ (gamma) pruning** (better for YOLOv8). Every convolutional layer in modern networks is followed by a Batch Normalization layer. BatchNorm has a learned scale factor γ: it multiplies the normalized activations. If γ ≈ 0 for a channel, that channel's output is near-zero regardless of the input — it contributes nothing. These channels are safe to prune. You can further encourage this during training by adding an L1 regularization term on all γ values to the detection loss: L_total = L_detection + λ × Σ|γᵢ|. This "sparsity training" pushes unused channels toward zero, making them easier to identify and remove cleanly.

**Pruning workflow:**
1. Plot a histogram of all |γ| values across the network. Channels near zero are candidates.
2. Rank by |γ| and remove the bottom p%.
3. Fine-tune for a few epochs at a low learning rate to recover accuracy.
4. Measure: ΔParams, ΔFLOPs, ΔmAP, and latency improvement.

**Expected results at 30% pruning ratio:** 20–40% FLOPs reduction, 1.2–2× latency improvement, ~2–5% mAP drop.

---

**Lab 3 — Quantization: Fewer Bits, Less Memory, Faster Math**
Pages: 31–33

A standard neural network stores each weight as a 32-bit floating point number (FP32). Quantization replaces these with lower-precision numbers. Less precision = smaller model, less memory bandwidth, faster arithmetic on hardware that supports it.

**How quantization works mathematically (uniform affine quantization):**
Given a range [x_min, x_max] and a target bit-width b:
- Scale: s = (x_max - x_min) / (2^b - 1)
- Zero-point: z = -round(x_min / s)
- Quantize: q = clamp(round(x/s) + z, 0, 2^b - 1)
- Dequantize: x̂ = s × (q - z)
- Maximum error: |x - x̂| ≤ s/2

**FP16 (16-bit float):** Uses half the memory of FP32. FP16 has 1 sign bit, 5 exponent bits, 10 mantissa bits — dynamic range roughly 10⁻⁴ to 10⁴. Most neural network weights live in [-2, 2], so FP16 almost never overflows or rounds significantly. Result: ~0.5× memory, ~1.5–2× speedup on modern GPUs (T4, A100, RTX), near-zero accuracy loss. Enable with one flag: `half=True`. This is the first optimization to always apply — it is essentially free. (web: NVIDIA TensorRT docs)

**INT8 (8-bit integer):** Four times smaller than FP32 and 2–4× faster on dedicated INT8 hardware (like the Jetson Nano's TensorRT engine). The tradeoff: INT8 has limited precision (256 distinct values vs. millions in FP32), so there is a small accuracy drop (~1–2% mAP). INT8 also requires a **calibration dataset** — a small set of real images used to determine the optimal scale and zero-point for each layer, so that the quantization error is minimized for your actual data distribution.

**ONNX (Open Neural Network Exchange):** A file format and runtime for exporting models from PyTorch and running them on any hardware. Think of it as a "universal adapter." You export to ONNX once, then use TensorRT (NVIDIA), OpenVINO (Intel), or other runtimes to optimize for specific hardware.

**Recommendation from the lecture:**
- Always apply FP16 first. It is free speed.
- After FP16, apply INT8 with TensorRT for edge deployment (Jetson Nano).
- Use ONNX export as the intermediate step toward TensorRT.

| Precision | Memory vs FP32 | Speedup | mAP Drop |
|-----------|---------------|---------|----------|
| FP32 | 1× | 1× | 0% |
| FP16 | 0.5× | 1.5–2× | ~0% |
| INT8 | 0.25× | 2–4× | ~1–2% |
| INT4 | 0.125× | (hardware-dependent) | ~3–10%+ |

---

**Lab 4 — Knowledge Distillation: Teaching a Small Model to Think Like a Large One**
Pages: 34–36

Even after pruning and quantization, you may find that a small model (YOLOv8n) trained normally has substantially worse accuracy than the large model (YOLOv8s). Knowledge distillation is the technique for closing this gap without making the student model bigger.

The core insight comes from Hinton et al. (2015). When a large model classifies an image, its output is a **probability distribution** over all classes — not just a hard "this is a car" label. These soft probabilities are far richer than the ground-truth labels. If the model sees a car, it might output: car 80%, van 15%, truck 4%, bus 1%. The non-car probabilities encode the model's learned understanding that cars are similar to vans, somewhat similar to trucks, and quite different from bicycles. Hinton called this "**dark knowledge**" — information hidden in the wrong-answer probabilities. (web: Hinton et al. 2015, arXiv:1503.02531)

Hard ground-truth labels have none of this: car 1.0, everything else 0.0. They throw away the relationship information.

**Temperature scaling** makes the soft labels even more informative. The standard softmax function produces sharp distributions that can still be mostly 0.0 for most classes. Dividing the logits by a temperature T > 1 ("heating" them) before softmax produces a softer distribution where more of the dark knowledge is visible. T=4 is a common starting value; range 2–8 is typical.

**The distillation loss** combines two objectives:
- L_CE(hard): cross-entropy loss against the real ground truth labels (so the student learns the correct classes)
- L_KL(teacher||student): KL-divergence between the teacher's soft output and the student's soft output (so the student mimics the teacher's "thinking")
- Combined: L_KD = (1-α) × L_CE + α × T² × L_KL

The T² factor compensates for the fact that heating the softmax with temperature T shrinks the gradient magnitudes — multiplying by T² restores them to a useful scale. α controls how much weight to give the distillation signal vs. the hard labels; 0.5–0.9 is typical.

**Why this solves the multi-camera problem directly:**

| Cameras | YOLOv8s (~120 FPS total) | YOLOv8n (~300 FPS total) |
|---------|--------------------------|--------------------------|
| 1 | 120 FPS ✓ | 300 FPS ✓ |
| 2 | 60 FPS ✓ | 150 FPS ✓ |
| 4 | 30 FPS ✓ | 75 FPS ✓ |
| 8 | 15 FPS ✗ | 37 FPS ✓ |
| 16 | 7 FPS ✗ | 18 FPS ✗ |

A student YOLOv8n trained with distillation achieves ~92% of the teacher YOLOv8s mAP while running 2.5× faster. That is the difference between handling 2 cameras and handling 4+.

---

**Lab 5 — Multi-Scale Detection Heads and the FPN**
Pages: 37–39

This lab addresses the hardest problem in VisDrone: detecting objects that are tiny (8×8 pixels or less) in high-resolution drone images.

**The multi-scale problem.** A convolutional network progressively downsamples the image as it gets deeper. At the deepest layers, a 416×416 image might be represented as a 13×13 feature map — each cell corresponds to a 32×32 patch of the original image. This is great for detecting large objects, but a pedestrian that occupies only 8×8 pixels in the original image is completely invisible at this resolution: the whole pedestrian fits inside a quarter of one cell.

The solution is a **Feature Pyramid Network (FPN)** — a multi-scale architecture that maintains detection heads at multiple resolutions simultaneously. (web: Lin et al. CVPR 2017)

FPN works with three levels:
- **P3 (Stride 8):** 52×52 grid cells, 2,704 anchor points. Detects objects < 16 pixels. This is where tiny pedestrians are found.
- **P4 (Stride 16):** 26×26 grid, 676 anchor points. Detects 16–64 pixel objects.
- **P5 (Stride 32):** 13×13 grid, 169 anchor points. Detects objects > 64 pixels (cars, buses).

The FPN adds a "top-down pathway": features from deep (low-resolution, high-semantic) layers are upsampled and merged with features from shallow (high-resolution, low-semantic) layers via lateral connections. The result is that every scale in the pyramid has both fine spatial detail AND high-level semantic understanding — neither alone is sufficient for detecting small objects accurately. (web: Ultralytics FPN glossary)

**Depthwise-Separable Convolutions** make the detection heads lightweight enough to afford three of them simultaneously.

Standard 2D convolution fuses spatial filtering (where are the features?) and channel mixing (which features interact?) in one operation. The problem: with many channels, the cross-channel interactions dominate the FLOPs cost. A standard Conv2D on a 52×52 feature map with 256 input and 256 output channels and a 3×3 kernel costs: 2 × 256 × 9 × 256 × 52 × 52 ≈ 3.2 billion operations — just for one layer.

Depthwise-separable convolution splits this into two cheaper steps: (web: MobileNet paper; TDS)

1. **Depthwise conv:** Apply one k×k filter to each input channel independently. No channel mixing. Cost: C_in × k² × H × W (one filter per channel, no cross-channel interactions).
2. **Pointwise conv (1×1):** Apply a 1×1 convolution to mix channels and project to the desired output dimension. Cost: C_in × C_out × H × W (just dot products, no spatial kernel).

Total DW-Sep cost ≈ (C_in × k² × H × W) + (C_in × C_out × H × W).
Reduction factor ≈ 1/C_out + 1/k². For k=3, C_out=256: ~8–9× fewer FLOPs.

For the P3 head in VisDrone specifically, this means ~85–90% FLOPs reduction in the head that matters most for tiny object detection, with only ~1–3% mAP drop.

---

**Lab 6 — Multi-Camera Deployment on Jetson Nano**
Pages: 40–42

**Batch inference** is the core trick for serving multiple cameras from one GPU. Instead of running inference on each frame individually, group N frames from N cameras into a single batch and run one forward pass. The GPU processes all N frames in parallel.

The throughput gain is dramatic because per-batch overhead (memory transfer, kernel launch) is amortized across N images:

| Batch Size | Latency/batch | Per-image FPS | Total throughput |
|-----------|--------------|--------------|-----------------|
| bs=1 | 8 ms | 125 | 125 img/s |
| bs=2 | 10 ms | 100 | 200 img/s |
| bs=4 | 15 ms | 67 | 267 img/s |
| bs=8 | 25 ms | 40 | 320 img/s |

The tradeoff: per-image latency increases (a single camera waits up to 15 ms in batch 4 vs 8 ms solo). For live video streams this is a real consideration, but for a 25 FPS requirement it is fine.

**Deployment pipeline for Jetson Nano:**

Step 1 — Export to ONNX (on Colab):
```python
model.export(format='onnx', imgsz=416, simplify=True)
```
`simplify=True` reduces graph complexity for the edge runtime.

Step 2 — Convert to TensorRT (on Jetson):
```bash
trtexec --onnx=model.onnx --saveEngine=model.engine --fp16
# Add --int8 for maximum speed
```
TensorRT performs hardware-specific optimizations: layer fusion (merging operations), kernel auto-tuning (selecting the fastest GPU kernel for your specific Jetson hardware), and precision calibration. The result is a `.engine` file compiled specifically for that device. (web: NVIDIA TensorRT docs)

Step 3 — Run inference:
```python
from ultralytics import YOLO
m = YOLO('model.engine')
results = m.predict(source=0, stream=True)  # webcam
```

Step 4 — Multi-camera streaming:
```python
sources = ['rtsp://cam1', 'rtsp://cam2', 'rtsp://cam3', 'rtsp://cam4']
results = m.predict(source=sources, stream=True)
```

**Expected Jetson Nano performance (YOLOv8n INT8):**
- 1 camera: ~20–25 FPS
- 2 cameras: ~12–15 FPS/stream
- 4 cameras: ~6–8 FPS/stream

For true 4-camera real-time on Jetson Nano, the recommendation is YOLOv8n INT8 with a stronger board (e.g., Jetson Orin) or to accept the slightly-below-25-FPS rate for use cases where 8–10 FPS is sufficient.

---

**All Techniques Combined: The Results Dashboard**
Pages: 43–44

This table shows all optimizations applied to YOLOv8s/n, measured and compared:

| Technique | FLOPs Reduction | Latency Improvement | mAP Impact | Effort |
|-----------|----------------|--------------------|-----------|----|
| Baseline (FP32) | 0% | 1× | 0% | ★ |
| FP16 Quantization | 0% | ~1.5–2× | ~0% | ★ |
| INT8 (TensorRT) | 0% | ~2–4× | ~1–2% | ★★ |
| Structured Pruning | 20–40% | ~1.2–2× | ~2–5% | ★★★ |
| Knowledge Distillation | 60–75% | ~2–3× | ~5–8% | ★★★ |
| DW-Sep Heads | 85–90% (head) | ~1.3× | ~1–3% | ★★ |
| Image Resize ↓ | Quadratic | Quadratic | ~3–10% | ★ |

Note: FP16/INT8 do not reduce FLOPs (the same operations happen), but they reduce memory bandwidth and enable faster hardware execution units. FLOPs reduction comes from pruning, distillation, and architecture changes.

**Final hackathon answer — the best stack for 4 cameras at 25 FPS:**

| Option | Est. Total FPS | Meets 100 FPS? | mAP ≥ 80%? |
|--------|---------------|---------------|------------|
| YOLOv8s FP32 | ~120 | ✗ (4 cam total) | ✓ |
| YOLOv8s FP16 | ~200 | ✓ | ✓ |
| YOLOv8n FP16 | ~300 | ✓ | ✓ (~87%) |
| **YOLOv8n FP16 + KD** | **~300+** | **✓** | **✓ (~92%)** |
| YOLOv8n INT8 (Jetson) | ~25/cam | ✓ | ✓ (~89%) |

**Best recommendation: YOLOv8n FP16 + Knowledge Distillation.** Highest FPS while keeping mAP within 8% of baseline. With batch inference (bs=4): throughput increases ~2× further, exceeding 600 img/s. For Jetson Nano specifically: YOLOv8n INT8 is the right choice — adequate for 1–2 cameras, scale to 4 with a stronger board.

---

**Beyond the Workshop: Where to Go Next**
Pages: 45–46

Six natural next steps after mastering these basics:

1. **Quantization-Aware Training (QAT):** Post-training INT8 calibration (what this workshop uses) can drop 1–2% mAP. QAT simulates quantization during training itself, so the model learns to be robust to integer precision. Use when PTQ accuracy drop is unacceptable.

2. **Neural Architecture Search (NAS):** Automate the model design process entirely. Tools like DARTS and Once-for-All search the space of possible architectures and find the one that maximizes accuracy for a given FLOPs budget. Effectively automates the "design a lightweight head" task in Lab 5.

3. **Vision Transformers for Detection:** DETR, RT-DETR, Co-DETR. Attention-based architectures apply different optimization strategies (token pruning, attention distillation) that do not map directly to the convolution-centric techniques here.

4. **Streaming and pipeline optimization:** The model is only one component. Real multi-camera systems need GStreamer or DeepStream pipelines for decoding, buffering, and scheduling frames from multiple sources efficiently.

5. **Domain adaptation:** A model trained on VisDrone drone footage from one city may fail in another city or different weather. UDA (Unsupervised Domain Adaptation) techniques close the distribution gap without needing fully labeled data in the new domain.

6. **Better data:** Synthetic training data from simulators (CARLA, AirSim) and semi-supervised annotation with active learning can dramatically improve small-object detection without any architectural change. Better data often beats a bigger model.

---

## Key Formulas Reference

```
# IoU
IoU = Area(Intersection) / Area(Union)

# mAP@50
mAP = (1/|C|) × Σ_c AP_c    where AP_c = ∫P(r)dr ≈ Σ_k P(k)×ΔR(k)

# FLOPs (Conv2D layer)
FLOPs = 2 × C_in × k² × C_out × H × W

# FPS
FPS = 1000 / mean_latency_ms

# Efficiency
Efficiency = mAP@50 / GFLOPs

# BatchNorm gating
y = γ × x̂ + β    where x̂ = (x - μ) / √(σ² + ε)
If |γ| ≈ 0 → channel is prunable

# Pruning sparsity loss
L_total = L_detection + λ × Σ|γ_i|

# Quantization
s = (x_max - x_min) / (2^b - 1)
q = clamp(round(x/s) + z, 0, 2^b - 1)
max error = s/2

# Distillation loss
p_i^T = exp(z_i / T) / Σ_j exp(z_j / T)    (temperature-scaled softmax)
L_KD = (1-α)·L_CE(hard) + α·T²·L_KL(teacher||student)

# Depthwise-Sep Conv FLOPs reduction
Reduction ≈ 1/C_out + 1/k²    (e.g., ~8–9× for k=3, C_out=256)

# DFL (anchor-free box prediction)
b̂ = Σ_i softmax(z_i) × i
```

---

## Sources

- [Intersection over Union (IoU) — Ultralytics](https://www.ultralytics.com/glossary/intersection-over-union-iou)
- [YOLO Object Detection — DataCamp](https://www.datacamp.com/blog/yolo-object-detection-explained)
- [Feature Pyramid Networks — Ultralytics](https://www.ultralytics.com/glossary/feature-pyramid-network-fpn)
- [Distilling the Knowledge in a Neural Network — Hinton et al. 2015](https://arxiv.org/abs/1503.02531)
- [Depthwise Separable Convolution / MobileNet — TDS](https://medium.com/data-science/understanding-depthwise-separable-convolutions-and-the-efficiency-of-mobilenets-6de3d6b62503)
- [INT8 Quantization and TensorRT — NVIDIA Docs](https://docs.nvidia.com/deeplearning/tensorrt/latest/inference-library/work-quantized-types.html)
- [Feature Pyramid Networks — Lin et al. CVPR 2017](https://openaccess.thecvf.com/content_cvpr_2017/papers/Lin_Feature_Pyramid_Networks_CVPR_2017_paper.pdf)
- [VisDrone Dataset — Ultralytics Docs](https://docs.ultralytics.com/datasets/detect/visdrone)
