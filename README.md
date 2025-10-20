# ECG Heartbeat Classification: From Data to Diagnosis

**Automated arrhythmia detection using a full data-science workflow — from exploratory analysis to validated models and clinical-oriented recommendations.**

---

## Table of Contents

* [Project Overview](#project-overview)
* [Introduction](#introduction)
* [Goals & Key Questions](#goals--key-questions)
* [Dataset Summary](#dataset-summary)
* [High-level Methodology](#high-level-methodology)
* [Results & Final Model Comparison](#results--final-model-comparison)
* [Repository structure](#repository-structure)
* [Notes, limitations & ethics](#notes-limitations--ethics)
* [License & Contact](#license--contact)

---

## Project Overview

This project builds an automated, data-driven system to distinguish normal from abnormal heartbeats from ECG signals. We evaluate a full spectrum of approaches — from classical machine learning to advanced deep learning — with the practical objective of maximizing **clinical safety** (minimizing missed dangerous beats) while keeping strong overall performance.

---

## Introduction

An Electrocardiogram (ECG) measures the heart’s electrical activity and is the main diagnostic tool for identifying arrhythmias. Manual ECG review is time-consuming, subjective, and error-prone.

This project builds an automated, data-driven system to distinguish normal from abnormal heartbeats. We evaluate a full spectrum of approaches — from classical machine learning to advanced deep learning — with the practical objective of maximizing clinical safety (minimizing missed dangerous beats) while keeping strong overall performance.

Each spike and wave in the ECG trace tells a story about how the heart beats:

* **P-wave:** small bump — atrial contraction, the start of a heartbeat.
* **PR interval:** short pause — conduction time from SA node to ventricles.
* **QRS complex:** tall, sharp spike — ventricular contraction (main pumping).
* **ST segment:** short flat line — ventricular relaxation.
* **T-wave:** ventricular repolarization (recharging for the next beat).

In this project, these segments and their shapes are what models learn to interpret — similar to how a clinician reads an ECG — to decide whether a heartbeat is normal or abnormal.

---

## Goals & Key Questions

**Primary objective:** Deeply understand ECG heartbeat patterns, explore their statistical and morphological differences, and build models that can reliably detect abnormal rhythms while remaining interpretable and clinically relevant.

This project is a full data-science investigation focused on transforming ECG signals into practical diagnostic tools. The core research questions:

* What patterns are hidden in the data and how do heartbeat signals differ across classes (N, S, V, F, Q)?
* Which waveform regions (P, QRS, T) carry the most diagnostic information?
* Which features matter most: engineered statistics, frequency features, derivatives, or automatically learned patterns?
* How to handle severe class imbalance and noise (SMOTE, class weights, augmentation)?
* Can the heartbeat landscape be visualized (PCA, t-SNE, clustering) and what does that tell us about separability?
* How stable are the signals across patients? Are some patterns patient-specific?
* Which model families (classical ML vs deep learning) best capture the insights uncovered in EDA?
* What trade-offs exist between balanced accuracy and clinical safety (minimizing false negatives)?

---

## Dataset Summary

**MIT-BIH Arrhythmia Dataset (PhysioNet)**

* **Samples:** 109,446 beats
* **Classes (5):**

  * N — Normal (~90,000) — majority, low clinical priority
  * S — Supraventricular (~2,000) — minority, high clinical priority
  * V — Ventricular (~7,000) — minority, high clinical priority
  * F — Fusion (~700) — extreme minority, highest clinical priority
  * Q — Unclassified (~10,000) — medium
* **Processed segment size:** 188 time steps @ 125 Hz

**Implication:** Severe class imbalance (especially F, S) — requires strategies that prioritize recall on rare, critical classes.

---

## High-level Methodology

We follow a two-phase pipeline and use **Macro-F1** as the primary balanced metric.

**Phase 1 — Classical ML baseline**

* Feature engineering: time-domain, frequency-domain, derivatives, and descriptive statistics.
* Models: Logistic Regression, Decision Tree, Random Forest, XGBoost, etc.
* Key experiments: SMOTE, PCA, outlier removal, feature selection.

**Phase 2 — Deep learning & transfer**

* Architectures: 1D-CNN, GRU, Hybrid CNN–RNN, Attention-focused models.
* Transfer learning experiments: pretrained ECG encoder → XGBoost / fine-tuning.
* Imbalance handling: augmentation and class weights (final deep models use class weights).
* Evaluation: all models tested on the same hold-out test split. Report Macro-F1, precision, recall, with focused analysis for minority classes S and F.

---

## Results & Final Model Comparison

| Model                | Macro F1 | S Recall | F Recall |                                                          Conclusion |
| -------------------- | :------: | :------: | :------: | ------------------------------------------------------------------: |
| Hybrid CNN-RNN       |   0.92   |   0.82   |   0.83   |                     Best overall balance (tied) — highest Macro-F1. |
| CNN — Class-Weighted |   0.92   |   0.91   |   0.90   | Best for clinical safety (tied Macro-F1 + highest recall on S & F). |
| Attention-Focused    |   0.91   |   0.87   |   0.84   |                                          Strong balanced performer. |
| HF + XGBoost         |   0.86   |   0.67   |   0.72   |                   Weakest performer; shows deep learning advantage. |

**Short takeaways**

* **Best general classifiers:** Hybrid CNN-RNN and Class-Weighted CNN (tied, Macro-F1 = 0.92).
* **Best for clinical safety:** CNN with class weighting — prioritizes recall on rare, dangerous classes (S & F).
* **Classical baseline:** Random Forest + SMOTE improves Macro-F1 substantially vs no resampling and remains a robust, interpretable fallback.

---

## Answers to the Questions (Executive Summary)

1. **What’s hidden in the data?**

   * Classes show statistical differences but heavy overlap. 'N' beats are stable; 'F' beats show chaotic high-frequency content. t-SNE/clustering show massive overlap, explaining difficulty in classification.

2. **What features matter most?**

   * Both engineered statistics (skew, kurtosis, median), FFT bands, and derivatives provide separable information. However, CNNs learn subtle morphological patterns from raw signal that hand-crafted features can miss.

3. **How to handle imbalance and noise?**

   * Minority classes are ignored without intervention. SMOTE helped classical methods; for deep learning class weighting was most effective. Many outliers belonged to 'N' (rare physiological variations); removal had minimal benefit so they were kept for robustness.

4. **Can we visualize the heartbeat landscape?**

   * Yes — visualizations confirm a complex, overlapping feature space. Only 'V' shows some separation.

5. **How stable are signals?**

   * Normal variability is low; pathological classes (V, F) show higher chaotic variability (rapid autocorrelation decay). Models generalize reasonably well across patients on the test set.

6. **Which model families excel?**

   * Deep learning models best capture the complex morphological patterns; Random Forests are a strong, interpretable baseline.

7. **Which model to recommend?**

   * If the goal is overall balanced accuracy: Hybrid CNN-RNN or CNN class-weighted. If the priority is **clinical safety** (minimizing false negatives for S, F), choose the **CNN with class weights**.

**Model interpretability:** Grad-CAM and feature-importance analyses confirm that deep models focus on physiologically relevant waveform regions (e.g., QRS complex and T-wave), aligning with our EDA.

---

## Notes, limitations & ethics

* **Clinical disclaimer:** This is a research / academic project. The models and results are **not** approved for clinical use. Any deployment in a medical context requires regulatory clearance and clinical validation.
* **Data privacy & licensing:** MIT-BIH is publicly available via PhysioNet but has usage terms — check PhysioNet license and cite appropriately if you reuse the dataset.
* **Limitations:** strong class overlap and dataset biases may limit generalization to other ECG devices or populations. Performance may vary on external cohorts.
* **Responsible ML:** prioritize recall for critical classes, evaluate models on patient-wise splits to avoid leakage, and use interpretability methods (Grad-CAM, feature importance) to validate physiologically plausible attention patterns.

---

## License & Contact

* **License:** MIT (see `LICENSE` file)
* **Author:** Abdallah Ibraheem — Computer Science Student, Open University of Israel.
  ([LINKEDIN](https://www.linkedin.com/in/abdallah-ibraheem-910b542ba/))

---

## Acknowledgements

* MIT-BIH Arrhythmia Database — PhysioNet — for the dataset and annotations.

