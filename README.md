# AI-in-CyberSecurity-Capstone
# The Evolution of AI in Cybersecurity

## Project Overview

This project explores the application of Artificial Intelligence (AI), Machine Learning (ML), and Deep Learning (DL) for detecting malicious activity in IoT network traffic.

The project focuses on classifying network traffic into three categories:

- **Benign:** Normal network activity.
- **Mirai:** Malicious IoT botnet traffic.
- **Gafgyt:** Another malicious IoT botnet traffic family.

Different detection approaches are implemented and compared, including a statistical baseline, traditional machine learning algorithms, deep learning architectures, and Explainable AI (XAI) techniques.

The main objective is to evaluate their detection performance and understand how well models trained on one IoT device generalize to another.

## Project Objectives

- Understand traditional and AI-driven cybersecurity threat detection.
- Prepare and analyze IoT network traffic data.
- Apply data preprocessing and feature selection techniques.
- Implement and compare multiple ML and DL models.
- Evaluate model performance using suitable classification metrics.
- Apply Explainable AI to identify important features influencing predictions.
- Examine cross-device generalization and the limitations of different detection approaches.

## Dataset Description

The experiment uses IoT network traffic records containing **115 numerical features**. These features represent statistical characteristics of network traffic, such as mean, weight, radius, covariance, and jitter.

### Dataset Split

| Dataset | Device | Records |
|---|---|---:|
| Training | Device 2 | 30,000 |
| Testing | Device 4 | 7,500 |

Class distribution:

| Class | Training Records | Testing Records |
|---|---:|---:|
| Benign | 10,000 | 2,500 |
| Mirai | 10,000 | 2,500 |
| Gafgyt | 10,000 | 2,500 |
| **Total** | **30,000** | **7,500** |

The Device 2 to Device 4 split was used to examine whether models trained on one device could recognize traffic patterns from another device.

**Note:** The exact public dataset name and download source were not documented in the presentation and should be verified from the original dataset or notebook before being added.

## Data Preprocessing

The following preprocessing steps were applied:

1. Loaded the training and testing traffic records separately.
2. Assigned the corresponding traffic class labels.
3. Converted input features into numerical format.
4. Replaced infinite values with missing values (`NaN`).
5. Applied median imputation to handle missing values.
6. Fitted the median imputer on training data and applied it to both datasets to avoid data leakage.
7. Aligned training and testing data to the same 115 feature columns.
8. Performed duplicate and overlapping-record checks.

### Data Quality Findings

- 45 duplicate feature rows were identified in the training data.
- 3 duplicate feature rows were identified in the testing data.
- 276 exact feature rows overlapped between training and testing.
- Among 275 unique shared rows, labels matched as Gafgyt, with no conflicting labels.

These overlaps are important when interpreting the evaluation results because they may affect the independence of the test data.

## Feature Selection

Two feature selection approaches were compared:

**1. Mutual Information**

Measures how much information a feature provides about the target class.

**2. Random Forest Feature Importance**

Ranks features based on their contribution to tree-based classification decisions.

The performance of Random Forest was compared using the original 115 features and selected subsets of 20 features.

| Feature Set | Accuracy |
|---|---:|
| All 115 features | 98.95% |
| Top 20 Mutual Information features | 98.71% |
| Top 20 Random Forest features | 98.60% |

The results showed that the full feature set achieved the highest accuracy among these experiments.

## Models Implemented

### 1. Statistical Baseline

A percentile-range statistical classifier was implemented as a baseline.

For each class, the 1st and 99th percentiles were calculated for each feature. Each test record was scored according to the proportion of its features falling within the corresponding class ranges. The class with the highest matching score was selected.

### 2. Machine Learning Models

The following supervised ML algorithms were implemented:

- **Logistic Regression:** Learns weighted class boundaries.
- **Decision Tree:** Uses feature-based decision splits to classify traffic.
- **Random Forest:** Combines multiple decision trees to improve classification performance.

### 3. Deep Learning Models

Three deep learning architectures were evaluated:

**Artificial Neural Network (ANN)**

- Architecture: 128 → 64 → 32 → 3
- Uses ReLU activation functions, dropout, and a three-class softmax output.
- Learns nonlinear relationships among network traffic features.

**1D Convolutional Neural Network (1D-CNN)**

- Applies convolution filters across the ordered feature vector.
- Used to examine feature-pattern learning through convolutional layers.
- The input columns are not actual packet time-series data.

**Denoising Autoencoder (DAE)**

- Uses an encoder-decoder architecture.
- Trained using benign traffic from Device 2.
- Uses reconstruction error to identify unusual traffic.
- Performs binary classification: Benign or Attack.

Unlike the ANN and CNN, the DAE was designed as an anomaly detector rather than a three-class classifier.

## Performance Evaluation

The models were evaluated using classification accuracy and ROC-AUC where reported.

### Accuracy Comparison

| Model | Accuracy |
|---|---:|
| Statistical Range Baseline | 99.92% |
| Logistic Regression | 89.80% |
| Decision Tree | 98.40% |
| Random Forest | 98.95% |
| ANN | 99.84% |
| 1D-CNN | 99.87% |
| Denoising Autoencoder | 33.39% |

### ROC-AUC Results

| Model | ROC-AUC |
|---|---:|
| Random Forest | 0.99988 |
| Logistic Regression | 0.99975 |
| Denoising Autoencoder | 0.5002 |

The DAE performs a binary detection task, whereas the other classification models perform three-class classification. Therefore, their ROC-AUC values should not be treated as directly equivalent.

### Key Observations

- The statistical baseline achieved the highest reported accuracy of 99.92%.
- Random Forest achieved the highest reported ROC-AUC among the models listed.
- ANN and 1D-CNN also achieved high classification accuracy.
- The DAE performed poorly when evaluated on Device 4 traffic.

## Explainable AI (XAI)

Explainable AI techniques were applied to the ANN to understand which network traffic features influenced its predictions.

Two techniques were used:

**1. Permutation Feature Importance (PFI)**

Measures the reduction in model accuracy when an individual feature is shuffled.

The most important feature reported was `HH_jit_L5_mean`, with an importance of approximately 0.0050.

**2. SHAP (SHapley Additive exPlanations)**

Estimates the contribution of individual features to model predictions.

Top features based on mean absolute SHAP values:

| Feature | Mean Absolute SHAP |
|---|---:|
| HH_jit_L5_mean | 0.026488 |
| HH_jit_L3_mean | 0.025693 |
| HH_jit_L0.01_mean | 0.022321 |
| HH_jit_L1_mean | 0.018966 |
| HH_jit_L0.1_mean | 0.018879 |

The results showed that packet-jitter-related features had a significant influence on ANN predictions.

## Important Finding: DAE Performance

The Denoising Autoencoder achieved only 33.39% accuracy and a ROC-AUC of 0.5002.

The diagnostic analysis showed:

- Only 2 out of 2,500 benign records were correctly identified.
- 2,498 benign records were incorrectly flagged as attacks.
- 2,498 out of 5,000 attack records were classified as benign.

A likely explanation is cross-device distribution shift. The DAE learned benign traffic patterns from Device 2, but those patterns did not transfer effectively to Device 4.

This demonstrates that learning normal behaviour from one device does not necessarily guarantee reliable anomaly detection on another device.

## Conclusion

The project demonstrates how statistical methods, machine learning, deep learning, and Explainable AI can be applied to IoT cybersecurity threat detection.

The experiments showed that high classification accuracy does not necessarily guarantee reliable real-world deployment. Models must also generalize across devices, control false alarms, detect attacks effectively, and provide interpretable results.

The DAE experiment particularly highlighted the challenges of anomaly detection under cross-device distribution shift.

## Limitations

- Only three traffic classes and two devices were evaluated.
- The dataset contained some duplicate and overlapping feature records.
- The 1D-CNN treated ordered feature columns as a sequence, although they were not actual time-series data.
- The DAE used binary classification, unlike the multiclass models.
- Inference times depend on hardware, batch size, and execution conditions.
- The exact dataset source, package versions, and complete hyperparameter-tuning details were not documented in the presentation.

## Future Scope

- Evaluate the models on additional IoT devices and traffic capture sessions.
- Investigate domain adaptation and device-specific calibration.
- Improve anomaly detection under changing network traffic distributions.
- Evaluate additional attack families and realistic class imbalance.
- Measure false positive rate, false negative rate, and inference latency consistently.
- Explore local SHAP explanations to make individual threat predictions easier to interpret.

## Technologies and Techniques

- Python-based ML/DL experimentation
- Statistical percentile-range classification
- Logistic Regression
- Decision Tree
- Random Forest
- Artificial Neural Network (ANN)
- 1D Convolutional Neural Network (1D-CNN)
- Denoising Autoencoder (DAE)
- Mutual Information
- Random Forest Feature Importance
- Permutation Feature Importance
- SHAP

---

**Project:** The Evolution of AI in Cybersecurity  
**Domain:** Artificial Intelligence and Cybersecurity  
**Application:** IoT Network Traffic Classification and Threat Detection
