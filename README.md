# Heart Attack Risk Analysis - Edge Impulse Model
 
A machine learning model trained on Edge Impulse to estimate heart attack risk from tabular health data, designed for deployment on embedded hardware (ESP32/Arduino).
 
> **Disclaimer:** This project is for educational/research purposes only. It uses a **synthetic dataset** and a **rule-based, self-engineered label** (see [Dataset](#dataset) below) — it is **not** trained on real patient outcomes, is **not** a certified medical device, and must not be used for real diagnosis or treatment decisions. Always consult a qualified medical professional for health concerns.
 
---
 
## Overview
 
This repository demonstrates a full edge ML pipeline — from tabular data, through training and evaluation in Edge Impulse, to on-device inference on an ESP32 — using a **synthetic** heart attack risk dataset. Because the data is synthetic and the label is self-engineered rather than clinically derived, the project should be read as a demonstration of the pipeline and workflow, not as a validated real-world risk predictor.

**Live Edge Impulse project:** https://studio.edgeimpulse.com/public/1096455/live
 
- **Platform:** Edge Impulse
- **Input type:** Tabular health data
- **Target deployment:** Espressif ESP-EYE (ESP32-based board)
- **Task type:** Binary classification (0 = low risk, 1 = high risk)
---
 
## Dataset
 
- **Source:** [Heart Attack Risk Prediction Dataset](https://www.kaggle.com/datasets/iamsouravbanerjee/heart-attack-prediction-dataset) on Kaggle (by iamsouravbanerjee) — **fully synthetic** data (no real patients), generated for ML practice/benchmarking rather than clinical use
- **Number of samples:** ~8,763 patient records (26 original columns in the full dataset; 18 were selected as input features for this model)
- **Features used (18):** Age, Cholesterol, Heart Rate, Diabetes, Family History, Smoking, Obesity, Alcohol Consumption, Exercise Hours Per Week, Previous Heart Problems, Medication Use, Stress Level, Sedentary Hours Per Day, Income, BMI, Triglycerides, Physical Activity Days Per Week, Sleep Hours Per Day
- **Target label:** Binary — 0 (low risk) / 1 (high risk). **Note:** the original dataset's label had ~0 correlation with every feature (verified via correlation analysis — max |r| ≈ 0.03 across all 18 features), meaning the model had no learnable signal from the raw data. The label used here was re-engineered as a rule-based composite risk score built from established cardiac risk factors (age > 50, cholesterol > 240, systolic BP > 140, diabetes, family history, smoking, obesity, previous heart problems, triglycerides > 200, BMI > 30, sedentary hours > 8, low exercise/activity), thresholded at the median to produce a roughly balanced binary label (58% / 42%). This is **not derived from real clinical outcomes** — it's a transparent, rule-based label built for this educational project so the model has something meaningful to learn.
- **Train/validation split:** 83% / 17% (rebalanced via Edge Impulse's train/test split tool)
- **Preprocessing:** Raw tabular features passed directly (no signal processing block) into the classifier
---
 
## Model Details
 
| Property | Value |
|---|---|
| Processing block | Raw data (18 input axes, no DSP block) |
| Learning block | Classifier |
| Output classes | 2 (0, 1) |
| Accuracy (unoptimized, float32) | 76.09% |
| Accuracy (quantized, int8) | 75.80% |
| Area under ROC Curve | 0.82 |
| Weighted average precision | 0.82 |
| Weighted average recall | 0.82 |
| Weighted average F1 score | 0.82 |
| Model size (quantized, int8) | 1.8 KB RAM / 19.1 KB flash |
| Inference time (on-device, ESP32 @ 240MHz, quantized) | 2 ms |
| Model size (unoptimized, float32) | 2.0 KB RAM / 28.8 KB flash |
| Inference time (unoptimized, float32) | 6 ms |
 
**Note on model iteration:** an earlier version of this model, trained on the dataset's original label, scored only 38.85% accuracy with a ROC-AUC of 0.50 (equivalent to random guessing) — investigation showed the original label had ~0 correlation with every input feature. After re-engineering the label as a rule-based composite risk score (see [Dataset](#dataset) above) and correcting the train/validation split from 100%/0% to 83%/17%, the model reached the metrics above. Quantization to int8 costs a negligible ~0.3% accuracy in exchange for a much smaller footprint (12% less RAM, 34% less flash, 3x faster inference).
 
---
 
## Hardware / Deployment
 
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
├── library.properties # Arduino library metadata (used by Arduino IDE)
├── heart_attack_risk_test.ino # Test sketch — hardcoded feature vector, run on ESP-EYE/ESP32
├── src/ # Core inferencing library source (model + SDK code)
│ ├── edge-impulse-sdk/
│ ├── model-parameters/
│ └── <project>_inferencing.h # Main library header — #include this in your sketch
└── examples/ # Ready-to-flash example sketches per target board
 ├── esp32/
 ├── nano_ble33_sense/
 ├── nano_ble33_sense_rev2/
 ├── nicla_sense/
 ├── nicla_vision/
 ├── portenta_h7/
 ├── rp2040/
 ├── sony_spresense/
 └── static_buffer/ # Minimal example using a hardcoded input buffer
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
 
## Results
 
**Confusion matrix (validation set, unoptimized float32):**
 
| Actual \ Predicted | 0 | 1 | Uncertain |
|---|---|---|---|
| **0** | 72.5% | 13.3% | 14.2% |
| **1** | 10.8% | 78.7% | 10.5% |
 
The model correctly identifies both classes the majority of the time, with a much smaller "uncertain" bucket than the earlier version. This lines up with the 0.82 ROC-AUC and 76.09% accuracy reported above — a meaningful improvement over the original label, which produced near-random results (see the note in [Model Details](#model-details)).
 
 
---
 
## Future Improvements
 
- [ ] Replace the synthetic dataset with a real clinical dataset (e.g. UCI Heart Disease) for any result that needs real-world validity
- [ ] Validate the engineered risk-score formula against real clinical scoring guidelines (e.g. Framingham Risk Score) rather than an ad-hoc weighting
- [ ] Add real sensor input (e.g. pulse sensor, BP module) instead of hardcoded/manual entry
- [ ] Reduce false negatives further (currently ~10.8% of true high-risk cases misclassified as low-risk)
- [ ] Drop low-value features (e.g. Income showed ~0 correlation with the engineered label) to simplify the model
- [ ] Add on-device alert system (buzzer/LED/notification)
---
 
## License
 
MIT License
 
---
 
## Acknowledgements
 
- Built with [Edge Impulse](https://www.edgeimpulse.com/) — [view the public project](https://studio.edgeimpulse.com/public/1096455/live)
- Initial Dataset: [Heart Attack Risk Prediction Dataset](https://www.kaggle.com/datasets/iamsouravbanerjee/heart-attack-prediction-dataset) by iamsouravbanerjee on Kaggle
- Current Dataset: Synthetic data, No real Patient data
