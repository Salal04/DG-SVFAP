# DG-SVFAP: Dual-Stream Visual-Geometric Spatiotemporal Facial Action Prior

DG-SVFAP is a dual-stream deep learning architecture for **engagement detection in online meetings and e-learning platforms**. It extends [SVFAP](https://doi.org/10.1109/TAFFC.2024.3432380) (Self-supervised Video Facial Affect Perceiver) with a dedicated facial-landmark stream, fused through a lightweight Transformer, to jointly capture appearance-level spatiotemporal patterns and structured facial geometry (gaze, head pose, eye blinks).

The model achieves **71.9% validation accuracy** and **71.2% test accuracy** on the [EngageNet](https://doi.org/10.1145/3577190.3614164) dataset, outperforming baselines including MARLIN, TCCT-Net, and Ordinal ST-GCN.

A live demo application, **SmartMeet**, is deployed at [smartmeet-platform.vercel.app](https://smartmeet-platform.vercel.app/), running real-time engagement inference in the browser with ~37ms latency per 10-second clip on a single T4 GPU, supporting up to 540 concurrent participants.

---

## ✨ Key Contributions

- **Dual-Stream Architecture** — Extends SVFAP by adding a parallel facial-landmark stream alongside the RGB video stream.
- **Cross-Modal Fusion Module** — Concatenates RGB and landmark features and refines them via a lightweight Transformer encoder.
- **Landmark-Augmented Geometric Supervision** — Uses MediaPipe-extracted facial landmarks to capture gaze direction, head pose, and eye blink patterns.
- **SOTA Performance on EngageNet** — Outperforms all compared baselines.
- **SmartMeet Deployment** — Browser-based, real-time engagement detection platform.

---

## 🏗️ Architecture

```
                 ┌─────────────────────┐
Video Frames ───▶│   SVFAP (RGB Stream) │──▶ f_rgb (B, 512)
(B,C,T,H,W)      │  Vision Transformer  │
                 └─────────────────────┘
                                              ┌───────────────┐
                                              │  Concatenate  │
Landmarks   ───▶┌──────────────────────┐     │  + Linear Proj│──▶ Transformer
(B,T,L)         │ Landmark Stream       │────▶│  (768 → 512)  │    Fusion (×2)
                │ Linear Proj + Pos Enc │     └───────────────┘        │
                │ + Transformer (×4)    │                              ▼
                │ + Temporal Mean Pool  │                     Classification Head
                └──────────────────────┘──▶ f_lm (B, 256)      (Linear → K classes)
```

**Components:**

| Module | Description |
|---|---|
| **RGB Stream** | SVFAP backbone (pre-trained on VoxCeleb2), outputs pooled feature `f_rgb ∈ R^512` |
| **Landmark Stream** | Linear projection + learnable temporal positional encoding → 4-layer Transformer encoder → temporal mean pooling → `f_lm ∈ R^256` |
| **Fusion Module** | Concatenation → linear projection (768→512) → 2-layer Transformer encoder for cross-modal attention |
| **Classification Head** | Linear layer mapping fused features to engagement class scores |

---

## 📊 Results

### SOTA Comparison (EngageNet Validation Set)

| Method | Validation Accuracy |
|---|---|
| ResNet-TCN | 54.2% |
| CNN-LSTM | 65.2% |
| CNN-BiLSTM | 66.1% |
| MARLIN | 68.4% |
| TCCT-Net | 68.9% |
| Ordinal ST-GCN | 71.2% |
| **DG-SVFAP (Ours)** | **71.9%** |

### Ablation Study

| Configuration | Validation Accuracy |
|---|---|
| Landmark Stream Only | 64.5% |
| SVFAP (RGB only) | 69.0% |
| Full Model w/o Augmentation | 66.0% |
| **DG-SVFAP (Full)** | **71.9%** |

---

## 📁 Dataset

Trained and evaluated on **[EngageNet](https://doi.org/10.1145/3577190.3614164)**:
- 31 hours of video, 127 participants, varied lighting conditions
- 11.2k video clips → 7.9k train / 2.2k test / 1k validation
- Annotated into 4 engagement levels (5th "Subject Not Present" class excluded during preprocessing)
- 16 frames sampled per clip, resized to 160×160
- Facial landmarks pre-extracted per frame via MediaPipe and stored as `.npy` files (zero-filled when detection fails)

The RGB backbone is initialized from an SVFAP checkpoint pre-trained on VoxCeleb2 (self-supervised, 1M+ video segments, 6,000+ speakers, 145 nationalities). The landmark stream, fusion module, and classification head are trained from scratch, with the full model fine-tuned end-to-end.

---

## 🚀 SmartMeet — Live Deployment

SmartMeet is a browser-based engagement detection platform built on top of DG-SVFAP:

- **Client-side**: face detection and landmark extraction run in-browser (no software install required)
- **Server-side**: inference on a T4 GPU backend
- **Latency**: ~37ms per 10-second clip
- **Scale**: supports up to 540 concurrent participants; horizontally scalable across backend instances

🔗 Try it: [smartmeet-platform.vercel.app](https://smartmeet-platform.vercel.app/)

---

## 🛠️ Setup

```bash
git clone https://github.com/Salal04/DG-SVFAP.git
cd DG-SVFAP
pip install -r requirements.txt
```

### Training

```bash
python train.py \
  --dataset engagenet \
  --backbone svfap_checkpoint.pth \
  --epochs 20 \
  --batch_size 4
```

### Inference

```bash
python infer.py --video path/to/clip.mp4
```

> Update the commands above to match your actual scripts/CLI once finalized.

---

## 📌 Model Details

- **Landmark dimension**: 256 (`D_lm`)
- **Landmark Transformer layers**: 4
- **Fusion Transformer layers**: 2
- **Training**: 20 epochs, batch size 4, end-to-end fine-tuning, cross-entropy loss
- **Augmentation**: random cropping, horizontal flipping (RGB), with corresponding landmark keypoint flipping

---

## 🔭 Future Work

- Incorporating audio/speech features for richer multimodal signals
- Validating generalization on Zoom, Microsoft Teams recordings
- Semi-supervised / active learning to reduce labeled-data dependence
- Scaling deployment beyond a single GPU instance

---

## 👥 Authors

Salal Shabbir, Abdul Ahad, Laiba Ajmal, Areeha Zulfiqar, Unbreen
Department of Software Engineering, University of the Punjab, Lahore, Pakistan

---

## 📄 Citation

If you use this work, please cite:

```bibtex
@article{dgsvfap2026,
  title={DG-SVFAP: Dual-Stream Visual-Geometric Spatiotemporal Facial Action Prior},
  author={Shabbir, Salal and Ahad, Abdul and Ajmal, Laiba and Zulfiqar, Areeha and Unbreen},
  institution={University of the Punjab},
  year={2026}
}
```

## 📜 License

Specify your license here (e.g., MIT, Apache 2.0).
