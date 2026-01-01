# GAURDCAN
# 🛡️ GuardCAN  
### Edge-Optimized CAN Bus Intrusion Detection System

GuardCAN is a deep learning–based Intrusion Detection System (IDS) designed to protect automotive CAN Bus networks from Denial-of-Service (DoS) attacks.  
It combines temporal LSTM modeling with INT8 quantization to achieve high accuracy on ultra-low-resource ECUs.

---

## 🚀 Project Highlights

| Feature | Description |
|--------|-------------|
| Model | Stacked LSTM (Recurrent Neural Network) |
| Input Strategy | 10-step sliding time windows |
| Attack Type | CAN Bus DoS (flooding) |
| Optimization | Full INT8 Post-Training Quantization |
| Model Size | 431 KB → 125 KB (71% smaller) |
| Accuracy Retention | 99.93% agreement between FP32 and INT8 |

---

## 📊 Dataset Profile

The model is trained using `dos_hard_mode_iso.csv`, which simulates aggressive CAN bus flooding.

| Metric | Value |
|-------|------|
| Total Frames | 60,058 |
| Normal Frames | 40,000 |
| Attack Frames | 20,058 |
| Features (10) | ID_int, DLC0–DLC7, Delta_T |

### Features Used
- ID_int – CAN Message ID  
- DLC0–DLC7 – Data bytes  
- Delta_T – Time gap between frames (core DoS indicator)

---

## 🧠 How GuardCAN Works

### 1. Feature Engineering
DoS attacks flood the CAN bus with high-priority frames, causing the time gap (ΔT) between messages to collapse toward zero.  
GuardCAN applies Z-score normalization so that the LSTM can detect even subtle flooding behavior.

### 2. Sequential Learning (LSTM)
Instead of classifying individual packets, GuardCAN processes sequences of 10 CAN frames:

```
[t₁, t₂, t₃, ... t₁₀] → LSTM → Normal / Attack
```

This captures the temporal fingerprint of DoS attacks.

### 3. INT8 Quantization
To deploy on ECUs, the FP32 Keras model is converted into a fully INT8 TFLite model:

- Representative dataset: 100 samples  
- Input & output forced to INT8  
- Result: 125 KB deployment-ready model  

---

## 📈 Model Performance

### Classification Report (12,010 test samples)

| Class | Precision | Recall | F1-score |
|------|----------|--------|----------|
| Normal | 0.91 | 0.96 | 0.94 |
| Attack | 0.90 | 0.82 | 0.86 |
| Overall Accuracy | **91%** |

### Quantization Comparison

| Metric | FP32 | INT8 |
|-------|------|------|
| File Size | 431 KB | 125 KB |
| Accuracy | 91.16% | 91.15% |
| Prediction Agreement | – | 99.93% |

---

## 📁 Repository Structure

```
GuardCAN/
│
├── GaurdCAN.ipynb
├── dos_hard_mode_iso.csv
├── final_dos_attack_model.keras
└── dos_model_full_int.tflite
```

---

## 💻 Installation

```bash
pip install tensorflow numpy pandas scikit-learn matplotlib
```

---

## 🧪 Run Inference with INT8 Model

```python
import tensorflow as tf

interpreter = tf.lite.Interpreter(model_path="dos_model_full_int.tflite")
interpreter.allocate_tensors()

input_details = interpreter.get_input_details()
output_details = interpreter.get_output_details()

interpreter.set_tensor(input_details[0]['index'], input_data)
interpreter.invoke()

prediction = interpreter.get_tensor(output_details[0]['index'])
```

---

## ⚖️ Conclusion

GuardCAN proves that advanced automotive cybersecurity does not require expensive hardware.  
By compressing a temporal LSTM into a 125 KB INT8 model, GuardCAN enables real-time CAN bus protection directly on vehicle ECUs.
