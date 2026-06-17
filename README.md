# AI-Course-2026
# 🧠 Deep Learning Intrusion Detection System
### W12-IDS-001 · TensorFlow/Keras · Network Security · SOC

> An end-to-end AI-powered IDS that detects both **known attacks** and **zero-day threats** in network traffic — using two complementary deep learning models trained on real flow-level features.

---

## 📌 Project Overview

Traditional signature-based IDS tools fail against novel attacks. This project tackles that gap by training two neural network architectures on network flow data extracted from real packet captures:

| Model | Type | Strength |
|-------|------|----------|
| Dense Neural Network | Supervised | Detects known, labelled attack patterns |
| Autoencoder | Unsupervised | Detects zero-day anomalies never seen in training |

Both models operate on **flow-level features** (not raw packets), making them fast, privacy-preserving, and deployable at scale.

---

## 🏗️ Architecture

```
Raw Network Traffic (PCAP)
        │
        ▼
  CICFlowMeter
  (Flow Feature Extraction)
        │
        ▼
   flows.csv
   78 features per flow
        │
   ┌────┴────┐
   │         │
   ▼         ▼
Supervised   Unsupervised
Dense NN    Autoencoder
   │              │
Known          Zero-Day
Attack         Anomaly
Detection      Detection
   │              │
   └────┬─────────┘
        ▼
   SOC Alert Dashboard
```

---

## 📊 Results

### Model 1 — Supervised Dense Neural Network

| Metric | Score |
|--------|-------|
| **AUC-ROC** | **0.982** |
| Accuracy | ~97% |
| Precision | High |
| Recall | High |

**Hyperparameter tuning log** (5 runs):

| Run | Learning Rate | Dropout | L2 | Val AUC | Notes |
|-----|--------------|---------|-----|---------|-------|
| 1 | 1e-2 | 0.2 | 1e-3 | 0.961 | Baseline |
| 2 | 1e-3 | 0.2 | 1e-3 | 0.974 | Lower LR |
| **3** | **1e-3** | **0.3** | **1e-4** | **0.982** | ✅ **Best — selected** |
| 4 | 1e-3 | 0.4 | 1e-4 | 0.979 | Too much dropout |
| 5 | 5e-4 | 0.3 | 1e-4 | 0.978 | Larger batch |

### Model 2 — Autoencoder (Anomaly Detection)

Trained exclusively on **benign traffic**. At inference, the reconstruction error of attack traffic is statistically separable from normal traffic — enabling threshold-based anomaly alerts without ever seeing labelled attacks.

---

## 🔬 Features & Engineering

Features extracted using **CICFlowMeter** from PCAP captures:

| Category | Features |
|----------|----------|
| Flow duration | `Flow Duration` |
| Packet length | `Fwd Pkt Len Max/Min/Mean/Std`, `Bwd Pkt Len Mean` |
| Inter-Arrival Times | `Flow IAT Mean`, `Fwd IAT Std` |
| Byte rates | `Flow Byts/s`, `Flow Pkts/s` |
| TCP Flags | `SYN Flag Cnt`, `RST Flag Cnt`, `PSH Flag Cnt` |
| Subflow stats | `Subflow Fwd Byts`, `Subflow Bwd Byts` |

**5 engineered features added:**

```python
df['Bytes_per_Packet']     = total_bytes / (total_packets + 1e-9)
df['Fwd_Bwd_Packet_Ratio'] = df['Tot Fwd Pkts'] / (df['Tot Bwd Pkts'] + 1e-9)
df['Packet_Rate']          = total_packets / (df['Flow Duration'] + 1e-9)
df['Avg_Fwd_Segment_Size'] = df['TotLen Fwd Pkts'] / (df['Tot Fwd Pkts'] + 1e-9)
df['Header_Payload_Ratio'] = df['Fwd Header Len'] / (df['TotLen Fwd Pkts'] + 1e-9)
```

---

## 🧱 Model Architecture

### Dense Neural Network
```
Input (n_features)
  → Dense(128, ReLU) + BatchNorm + Dropout(0.3) + L2(1e-4)
  → Dense(64,  ReLU) + BatchNorm + Dropout(0.3) + L2(1e-4)
  → Dense(32,  ReLU) + Dropout(0.2)
  → Dense(1, Sigmoid)   ← Binary classification output
```

### Autoencoder
```
Encoder:
  Input → Dense(64, ReLU) → BatchNorm → Dropout(0.2)
        → Dense(32, ReLU) → Dropout(0.1)
        → Dense(16, ReLU)  ← Bottleneck

Decoder:
  Dense(16) → Dense(32, ReLU) → Dense(64, ReLU) → Dense(n_features, Sigmoid)
```

Anomaly threshold = **75th percentile** of reconstruction error on benign validation set.

---

## ⚠️ Security Interpretation

| Metric | What it means in a SOC |
|--------|------------------------|
| **False Negatives** | Attacks that slipped through — the most dangerous failure |
| **False Positives** | Benign traffic flagged as attack — causes alert fatigue |
| **Precision** | Of all alerts raised, how many were real attacks? |
| **Recall** | Of all actual attacks, how many were caught? |

**Trade-off:** The supervised model is precise on known attacks. The autoencoder trades some precision for the ability to catch threats it has never seen — a critical property in real SOC environments.

---

## 🛡️ Evasion Techniques & Limitations

Real-world attackers can evade this IDS using:

1. **Slow & low attacks** — spreading traffic over long windows to mimic normal inter-arrival times
2. **Adversarial crafting** — tuning packet sizes/timing to stay below the reconstruction error threshold
3. **Encrypted payloads** — TLS hides payload; this IDS relies on metadata only
4. **Traffic mimicry** — structuring attack flows to match benign HTTP/DNS signatures

---

## ⚖️ Ethical Considerations

- **Privacy:** Flow metadata reveals behavioural patterns without inspecting payloads — but is still sensitive
- **Bias:** A model trained on one network may not generalise to others with different baseline traffic
- **Automation risk:** Automated response (blocking IPs) based on ML predictions risks collateral damage

---

## 📁 Project Files

| File | Description |
|------|-------------|
| `W12-IDS-001.ipynb` | Full Jupyter notebook — data loading, EDA, model training, evaluation |
| `flows.csv` | Network flow features dataset (extracted via CICFlowMeter) |
| `IDS_SOC_Live_Simulation.html` | Interactive SOC dashboard demo |
| `IDS_Final_Presentation.pptx` | Project presentation slides |
| `W12-IDS-001_Technical_Report.pdf` | Full technical report |

---

## 🚀 How to Run

```bash
# 1. Clone the repo
git clone https://github.com/YOUR_USERNAME/ids-deep-learning.git
cd ids-deep-learning

# 2. Install dependencies
pip install tensorflow pandas numpy scikit-learn matplotlib seaborn

# 3. Launch the notebook
jupyter notebook W12-IDS-001.ipynb

# 4. Open the live SOC demo
open IDS_SOC_Live_Simulation.html
```

> **Dataset:** `flows.csv` must be in the same directory as the notebook. It contains pre-extracted CICFlowMeter features from real network captures.

---

## 🛠️ Tech Stack

![Python](https://img.shields.io/badge/Python-3.10-blue?logo=python)
![TensorFlow](https://img.shields.io/badge/TensorFlow-2.x-orange?logo=tensorflow)
![Keras](https://img.shields.io/badge/Keras-Deep_Learning-red?logo=keras)
![Scikit-Learn](https://img.shields.io/badge/scikit--learn-ML-yellow?logo=scikit-learn)
![Pandas](https://img.shields.io/badge/Pandas-Data-green?logo=pandas)

---

## 👤 Author

**Tinashe Chibamu** — Student ID: 219134022

*Cybersecurity portfolio: PKI Engineering · SOC Operations · Azure Cloud Security · Network Security*

---

## 📄 Licence

This project was submitted as academic work. Code is shared for portfolio and educational purposes.
