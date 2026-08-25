# Heart Attack Risk Analysis — Edge Impulse Model
 
A machine learning model trained on Edge Impulse to estimate heart attack risk from tabular health data, designed for deployment on embedded hardware (ESP32/Arduino).
 
> **Disclaimer:** This project is for educational/research purposes only. It is **not** a certified medical device and must not be used for real diagnosis or treatment decisions. Always consult a qualified medical professional for health concerns.
 
---
 
## Overview
 
This repository contains an Edge Impulse–trained model that classifies/predicts heart attack risk based on tabular patient health features (e.g. age, cholesterol, blood pressure, etc.), exported for on-device inference on microcontrollers.
 
- **Platform:** Edge Impulse
- **Input type:** Tabular health data
- **Target deployment:** Espressif ESP-EYE (ESP32-based board)
- **Task type:** Binary classification (0 = low risk, 1 = high risk)
---
 
## Dataset
 
- **Source:** [Heart Attack Risk Prediction Dataset](https://www.kaggle.com/datasets/iamsouravbanerjee/heart-attack-prediction-dataset) on Kaggle (by iamsouravbanerjee) — synthetic, multi-factor patient data for heart attack risk prediction
- **Number of samples:** ~8,763 patient records (26 original columns in the full dataset; 18 were selected as input features for this model)
- **Features used (18):** Age, Cholesterol, Heart Rate, Diabetes, Family History, Smoking, Obesity, Alcohol Consumption, Exercise Hours Per Week, Previous Heart Problems, Medication Use, Stress Level, Sedentary Hours Per Day, Income, BMI, Triglycerides, Physical Activity Days Per Week, Sleep Hours Per Day
- **Target label:** Binary — 0 (low risk) / 1 (high risk)
- **Train/validation split:** 100% of data used for the training subset
- **Preprocessing:** Raw tabular features passed directly (no signal processing block) into the classifier
---
 
## Model Details
 
| Property | Value |
|---|---|
| Processing block | Raw data (18 input axes, no DSP block) |
| Learning block | Classifier |
| Output classes | 2 (0, 1) |
| Accuracy | 38.85% |
| Area under ROC Curve | 0.50 |
| Weighted average precision | 0.78 |
| Weighted average recall | 0.66 |
| Weighted average F1 score | 0.53 |
| Model size (quantized, int8) | 1.8 KB RAM / 19.1 KB flash |
| Inference time (on-device, ESP32 @ 240MHz, quantized) | 2 ms |
| Model size (unoptimized, float32) | 2.0 KB RAM / 28.8 KB flash |
| Inference time (unoptimized, float32) | 6 ms |
 
**Note on current performance:** the model's accuracy (38.85%) and ROC-AUC (0.50) indicate it is currently performing close to random guessing on this task. The confusion matrix shows a large share of "uncertain" predictions and confusion between the two classes. This is a known limitation — see [Future Improvements](#-future-improvements). Treat this model as a work-in-progress baseline, not a reliable predictor.
 
---
 
## 🔩 Hardware / Deployment
 
- **Target MCU:** Espressif ESP-EYE (ESP32-based)
- **Export format:** Arduino library
- **Sensors/inputs used:** No physical sensors — this model takes tabular inputs (age, cholesterol, etc.). Currently tested via a hardcoded feature array in the sketch (see `heart_attack_risk_test.ino`); Serial or a companion input method is a planned next step, not yet implemented.
### Deployment steps
1. In Edge Impulse Studio, go to **Deployment** → select **Arduino library**.
2. Keep **Quantized (int8)** selected (2 ms inference, smallest footprint) and click **Build**. This downloads a `.zip` Arduino library.
3. In Arduino IDE: `Sketch → Include Library → Add .ZIP Library` and select the downloaded file.
4. Open `heart_attack_risk_test.ino` from this repo, update the `#include` line with your library's actual header filename (check `Sketch → Include Library → [your library]` to confirm it), and fill in the `features[]` array with real test values.
5. Select your board (ESP-EYE / ESP32) under `Tools → Board`, pick the correct port, and upload.
6. Open Serial Monitor at 115200 baud to see prediction output.
See `heart_attack_risk_test.ino` in this repo for the full working sketch.
 
---
 
## Repository Structure
 
This repo is the Edge Impulse–exported Arduino library, ready to drop straight into your Arduino `libraries/` folder or install as a `.zip` library.
 
```
heart-attack-risk-analysis-educational-/
├── README.md
├── LICENSE
├── library.properties           # Arduino library metadata (used by Arduino IDE)
├── heart_attack_risk_test.ino    # Test sketch — hardcoded feature vector, run on ESP-EYE/ESP32
├── src/                          # Core inferencing library source (model + SDK code)
│   ├── edge-impulse-sdk/
│   ├── model-parameters/
│   └── <project>_inferencing.h   # Main library header — #include this in your sketch
└── examples/                     # Ready-to-flash example sketches per target board
    ├── esp32/
    ├── nano_ble33_sense/
    ├── nano_ble33_sense_rev2/
    ├── nicla_sense/
    ├── nicla_vision/
    ├── portenta_h7/
    ├── rp2040/
    ├── sony_spresense/
    └── static_buffer/             # Minimal example using a hardcoded input buffer
```
 
For this project, use `heart_attack_risk_test.ino` in the repo root as your starting sketch, or the **`esp32`** example under `examples/` for a more built-out reference.
 
---
 
## Getting Started
 
```bash
git clone https://github.com/cymphonikbeatz/heart-attack-risk-analysis-educational-.git
cd heart-attack-risk-analysis-educational-
```
 
1. Install the Edge Impulse library (`src/` in this repo) via Arduino IDE: `Sketch → Include Library → Add .ZIP Library`.
2. Open `heart_attack_risk_test.ino` and update the `#include` with your library's actual header filename.
3. Fill in the `features[]` array with real test values (or wire up your own input method).
4. Flash to your ESP-EYE / ESP32 and view predictions in Serial Monitor.
---
 
## 📊 Results
 
**Confusion matrix (validation set):**
 
| Actual \ Predicted | 0 | 1 | Uncertain |
|---|---|---|---|
| **0** | 58.9% | 0% | 41.1% |
| **1** | 55.0% | 0% | 45.0% |
 
The model rarely predicts class `1` with confidence, and a large portion of samples fall into the "uncertain" bucket — this is reflected in the low overall accuracy and ROC-AUC of 0.50 noted above. The feature explorer plot also shows heavy overlap between correct and incorrect predictions across classes, suggesting the current feature set/model isn't cleanly separating the two risk classes yet.
 
<!-- Optionally embed the Edge Impulse screenshots (confusion matrix, feature explorer) as images here, e.g.: -->
<!-- ![Confusion matrix](docs/confusion-matrix.png) -->
<!-- ![Feature explorer](docs/feature-explorer.png) -->
 
---
 
## Future Improvements
 
- [ ] Expand dataset size/diversity
- [ ] Add real sensor input (e.g. pulse sensor, BP module) instead of manual entry
- [ ] Improve model accuracy / reduce false negatives
- [ ] Add on-device alert system (buzzer/LED/notification)
---
 
## License
 
<!-- MIT License -->
 
---
 
## Acknowledgements
 
- Built with [Edge Impulse](https://www.edgeimpulse.com/)
- Dataset: [Heart Attack Risk Prediction Dataset](https://www.kaggle.com/datasets/iamsouravbanerjee/heart-attack-prediction-dataset) by iamsouravbanerjee on Kaggle
 
