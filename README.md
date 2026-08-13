<div align="center">

<img width="100%" alt="banner" src="https://capsule-render.vercel.app/api?type=waving&color=0:6366f1,50:8b5cf6,100:0ea5e9&height=220&section=header&text=DG-SVFAP&fontSize=60&fontColor=FFFFFF&fontAlignY=35&desc=Dual-Stream%20Visual-Geometric%20Spatiotemporal%20Facial%20Action%20Prior&descAlignY=58&descSize=17&descColor=E5E7EB&animation=fadeIn"/>

<br/>

**🎭 Real-time engagement detection for online meetings & e-learning**
*Extending SVFAP with a dedicated facial-landmark stream*

<br/>

<p>
<img src="https://img.shields.io/badge/🚀_Live_Demo-SmartMeet-6366f1?style=for-the-badge"/>
<img src="https://img.shields.io/badge/📊_Dataset-EngageNet-0ea5e9?style=for-the-badge"/>
<img src="https://img.shields.io/badge/🧠_Base_Model-SVFAP-22c55e?style=for-the-badge"/>
<img src="https://img.shields.io/badge/🌍_External_Eval-ROLE--D-f97316?style=for-the-badge"/>
<img src="https://img.shields.io/badge/⚖️_License-MIT-94a3b8?style=for-the-badge"/>
</p>

<br/>

<table>
<tr>
<td align="center" width="25%">

### 71.9%
<sub>Val Accuracy</sub>

</td>
<td align="center" width="25%">

### 71.2%
<sub>Test Accuracy</sub>

</td>
<td align="center" width="25%">

### ~37ms
<sub>Inference / 10s clip</sub>

</td>
<td align="center" width="25%">

### 540
<sub>Concurrent Users</sub>

</td>
</tr>
</table>

<a href="https://smartmeet-platform.vercel.app/"><b>🔗 Live Demo</b></a>
&nbsp;•&nbsp;
<a href="#-architecture"><b>🏗️ Architecture</b></a>
&nbsp;•&nbsp;
<a href="#-results"><b>📊 Results</b></a>
&nbsp;•&nbsp;
<a href="#%EF%B8%8F-setup"><b>🛠️ Setup</b></a>

</div>

<br/>

---

## 📖 Overview

**DG-SVFAP** extends [SVFAP](https://doi.org/10.1109/TAFFC.2024.3432380) *(Self-supervised Video Facial Affect Perceiver)* with a dedicated **facial-landmark stream**, fused through a lightweight Transformer, to jointly capture appearance-level spatiotemporal patterns and structured facial geometry — gaze, head pose, and eye blinks.

On **[EngageNet](https://doi.org/10.1145/3577190.3614164)**, the model reaches **71.9%** validation accuracy and **71.2%** test accuracy, outperforming MARLIN, TCCT-Net, and Ordinal ST-GCN.

> 💡 A live demo, **SmartMeet**, runs real-time engagement inference in-browser at **~37ms** per 10-second clip on a single T4 GPU, supporting up to **540 concurrent participants**.

<div align="center">

**[🎬 Try the SmartMeet live demo →](https://smartmeet-platform.vercel.app/)**

</div>

<br/>

## ✨ Key Contributions

<table width="100%">
<tr><td width="10%" align="center">🧩</td><td><b>Dual-Stream Architecture</b> — a parallel facial-landmark stream running alongside the RGB video stream</td></tr>
<tr><td align="center">🔗</td><td><b>Cross-Modal Fusion Module</b> — concatenates RGB + landmark features, refined via a lightweight Transformer encoder</td></tr>
<tr><td align="center">📐</td><td><b>Landmark-Augmented Geometric Supervision</b> — MediaPipe landmarks capture gaze, head pose, and eye blinks</td></tr>
<tr><td align="center">🏆</td><td><b>SOTA on EngageNet</b> — outperforms all compared baselines</td></tr>
<tr><td align="center">💻</td><td><b>SmartMeet Deployment</b> — a browser-based, real-time engagement detection platform</td></tr>
<tr><td align="center">🌍</td><td><b>ROLE-D (Self-Collected)</b> — a real-world external dataset, personally collected and annotated, used solely for unseen-participant testing</td></tr>
</table>

<br/>

## 🏗️ Architecture

<div align="center">
<img src="Fig2.png" alt="Architecture Diagram" width="600"/>
</div>

```mermaid
flowchart LR
    V[Video Clip] --> R[RGB Stream<br/>SVFAP Backbone]
    V --> P[MediaPipe<br/>Landmark Extraction]
    P --> L[Landmark Stream<br/>Linear Proj + Temporal PosEnc<br/>4-layer Transformer]
    R --> RF["f_rgb ∈ R^512"]
    L --> LF["f_lm ∈ R^256"]
    RF & LF --> C[Concatenate]
    C --> FP[Linear Projection<br/>768 → 512]
    FP --> FT[Fusion Transformer<br/>2 layers, cross-modal attention]
    FT --> H[Classification Head]
    H --> O[Engagement Class]

    classDef stream fill:#1e1b3a,stroke:#8b5cf6,stroke-width:2px,color:#e5e7eb
    classDef fusion fill:#0c2b3a,stroke:#0ea5e9,stroke-width:2px,color:#e5e7eb
    classDef output fill:#14301f,stroke:#22c55e,stroke-width:2px,color:#e5e7eb

    class R,L,P stream
    class C,FP,FT fusion
    class H,O output
```

| Module | Description |
|---|---|
| 🎥 **RGB Stream** | SVFAP backbone *(pre-trained on VoxCeleb2)* → `f_rgb ∈ ℝ⁵¹²` |
| 📍 **Landmark Stream** | Linear projection + learnable temporal positional encoding → 4-layer Transformer → temporal mean pooling → `f_lm ∈ ℝ²⁵⁶` |
| 🔀 **Fusion Module** | Concatenation → linear projection (768→512) → 2-layer Transformer for cross-modal attention |
| 🎯 **Classification Head** | Linear layer → engagement class scores |

<br/>

## 📁 Dataset

### Training / Evaluation — EngageNet

<div align="center">

| Attribute | Value |
|---|---|
| ⏱️ Duration | 31 hours of video |
| 👥 Participants | 127 |
| 🎞️ Clips | 11,200 *(7.9k train / 2.2k test / 1k val)* |
| 🏷️ Classes | 4 engagement levels |
| 📐 Format | 16 frames/clip @ 160×160, MediaPipe landmarks pre-extracted as `.npy` |

</div>

### External Evaluation — ROLE-D <img src="https://img.shields.io/badge/self--collected-f97316?style=flat-square"/>

**ROLE-D** *(Real-World Online Learning Engagement Dataset)* — 9 participants, natural home environments, fully unscripted. 1,003 ten-second clips, kept completely separate from training and validation, used exclusively to evaluate generalization on unseen participants.

<div align="center">
<img src="dataset-f.png" alt="Dataset Distribution" width="420"/>

<table>
<tr><td>🟢 <b>Highly Engaged</b></td><td>65%</td></tr>
<tr><td>🔵 <b>Engaged</b></td><td>16%</td></tr>
<tr><td>🟡 <b>Barely Engaged</b></td><td>6%</td></tr>
<tr><td>🔴 <b>Not Engaged</b></td><td>13%</td></tr>
</table>

</div>

> ℹ️ **Note:** Unlike EngageNet, ROLE-D was independently collected, recorded, and labeled as part of this project — not sourced from an existing public dataset.

The RGB backbone initializes from an SVFAP checkpoint pre-trained on VoxCeleb2 *(1M+ segments, 6,000+ speakers, 145 nationalities)*. The landmark stream, fusion module, and classification head train from scratch; the full model is fine-tuned end-to-end.

<br/>

## 🎓 Training Strategies

- ✅ **Full parameter fine-tuning**
- ✅ **LoRA** — replace the files in the `lora/` folder to use LoRA-based fine-tuning

<br/>

## 📊 Results

<table width="100%">
<tr>
<td valign="top" width="50%">

**🏆 SOTA Comparison** *(EngageNet Val)*

| Method | Val Acc |
|---|---|
| ResNet-TCN | 54.2% |
| CNN-LSTM | 65.2% |
| CNN-BiLSTM | 66.1% |
| MARLIN | 68.4% |
| TCCT-Net | 68.9% |
| Ordinal ST-GCN | 71.2% |
| **DG-SVFAP (Ours)** | 🥇 **71.9%** |

</td>
<td valign="top" width="50%">

**🔬 Ablation Study**

| Configuration | Val Acc |
|---|---|
| Landmark Only | 64.5% |
| SVFAP (RGB only) | 69.0% |
| Full Model w/o Aug. | 66.0% |
| **DG-SVFAP (Full)** | 🥇 **71.9%** |

</td>
</tr>
</table>

<br/>

## 🚀 SmartMeet — Live Deployment

<div align="center">

| Component | Details |
|---|---|
| 🖥️ Client-side | Face detection + landmark extraction, fully in-browser |
| ☁️ Server-side | Inference on T4 GPU backend |
| ⚡ Latency | ~37ms per 10-second clip |
| 📈 Scale | Up to 540 concurrent participants, horizontally scalable |

**🔗 [smartmeet-platform.vercel.app](https://smartmeet-platform.vercel.app/)**

</div>

<br/>

## 🛠️ Setup

```bash
git clone https://github.com/Salal04/DG-SVFAP.git
cd DG-SVFAP
pip install -r requirements.txt
```

<table width="100%">
<tr>
<td valign="top" width="50%">

**🏋️ Training**
```bash
python train.py \
  --dataset engagenet \
  --backbone svfap_checkpoint.pth \
  --epochs 20 \
  --batch_size 4
```

</td>
<td valign="top" width="50%">

**🔮 Inference**
```bash
python infer.py \
  --video path/to/clip.mp4
```

</td>
</tr>
</table>

> ⚠️ Update the commands above to match your actual scripts/CLI once finalized.

<br/>

## 📌 Model Details

| Parameter | Value |
|---|---|
| Landmark dimension (`D_lm`) | 256 |
| Landmark Transformer layers | 4 |
| Fusion Transformer layers | 2 |
| Training | 20 epochs, batch size 4, end-to-end fine-tuning, cross-entropy loss |
| Augmentation | Random cropping, horizontal flipping (RGB) with matching landmark keypoint flips |

<br/>

## 🔭 Future Work

- 🎙️ Incorporating audio/speech features for richer multimodal signals
- 🎥 Validating generalization on Zoom, Microsoft Teams recordings
- 🧪 Semi-supervised / active learning to reduce labeled-data dependence
- 🌐 Scaling deployment beyond a single GPU instance

<br/>

---

<div align="center">

## 👤 Author

**Salal Shabbir**
Department of Software Engineering, University of the Punjab, Lahore, Pakistan

</div>

---

## 📄 Citation

```bibtex
@article{dgsvfap2026,
  title={DG-SVFAP: Dual-Stream Visual-Geometric Spatiotemporal Facial Action Prior},
  institution={University of the Punjab},
  year={2026}
}
```

<br/>

<div align="center">
<img src="https://capsule-render.vercel.app/api?type=waving&color=0:0ea5e9,50:8b5cf6,100:6366f1&height=120&section=footer"/>

<sub>Made with 🎭 for more human-aware online learning</sub>

</div>
