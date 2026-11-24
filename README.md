# GNSS LOS/NLOS Classification in Urban Canyons

This project investigates the classification of Global Navigation Satellite System (GNSS) satellite observations as **line-of-sight (LOS)** or **non-line-of-sight (NLOS)** in dense urban environments. It focuses on linear machine-learning classifiers and compares optimization strategies, including a custom implementation using Adam.

## Overview

GNSS positioning is generally accurate in open-sky environments, but tall buildings in urban areas can block or reflect satellite signals. These effects produce NLOS and multipath measurements, which can introduce large pseudorange errors and degrade position estimates.

The objective is to identify unreliable satellite observations using only measurements available from a real GNSS receiver at prediction time. A reliable classifier could help a receiver reject or down-weight NLOS measurements before computing its position.

## Dataset

The project uses the [GNSS LOS/NLOS Classification competition dataset by POLYU IPNL on Kaggle](https://www.kaggle.com/competitions/gnss-classification). Measurements were collected using a **u-blox F9P receiver** as part of the [UrbanNav dataset](https://doi.org/10.33012/2021.17895).

The training data cover three Hong Kong scenarios:
- Tsim Sha Tsui
- Whampoa
- Mong Kok

The test data come from a different area, Kowloon Bay, allowing investigation of generalization to an unseen urban environment.

Provided files include:
- `GNSS_raw_train.csv` and `GNSS_raw_test.csv`: raw receiver measurements.
- `GNSS_train.xlsx` and `GNSS_test.xlsx`: elaborated datasets.
- A sample submission file.

Available measurements include GPS time, satellite identifiers, pseudorange, carrier phase, Doppler and carrier-to-noise density ratio (C/N0) on L1 and L2.

The elaborated data also contain LOS/NLOS labels, pseudorange errors and ground-truth positions. **Pseudorange errors and ground-truth information are reserved for analysis and are not used as input features**, because they are not available to the classifier at prediction time. The LOS/NLOS label is derived using a 3D building model and the ground-truth position.

## Methodology

### Data preparation

Each sample represents one satellite observation at one GPS time. The preprocessing workflow includes:
1. Cleaning the raw and elaborated datasets.
2. Aligning records by satellite identifier and GPS time.
3. Selecting measurements available at prediction time and excluding variables that could reveal the target label.
4. Constructing input features.
5. Scaling the input variables.
6. Investigating dimensionality reduction with Principal Component Analysis (PCA).

PCA will be used to study correlations between measurements, identify directions explaining most of the feature variance, investigate dimensionality reduction and visualize LOS/NLOS samples in a lower-dimensional space.

### Classification models

The project investigates four classifiers:

| Classifier | Loss / approach | Decision boundary |
|---|---|---|
| Logistic Regression | Logistic loss | Linear |
| Linear Support Vector Machine (SVM) | Hinge loss with L2 regularization | Linear |
| Perceptron | Perceptron learning rule | Linear |
| Random Forest | Ensemble of decision trees | Non-linear |

The linear classifiers use a decision function of the form:

$$
f(\mathbf{x}) = \mathbf{w}^{T}\mathbf{x} + b
$$

Random Forest, implemented using scikit-learn, provides a non-linear comparison.

### Optimization

Where appropriate, optimization procedures for the linear classifiers will be implemented directly. In particular, the project investigates using the **Adam optimizer** instead of standard stochastic gradient descent (SGD), comparing convergence behaviour and classification performance.

## Experiments

Planned experiments include:

- **Feature engineering:** investigating combined features, such as the difference between measured C/N0 and the value expected at a given satellite elevation, differences between L1 and L2 measurements, and consistency measures based on Doppler or carrier phase.
- **Feature scaling:** evaluating the effect of scaling input variables.
- **PCA:** evaluating dimensionality reduction and the number of principal components.
- **Classifier comparison:** comparing the models with and without engineered features.
- **Optimization comparison:** studying Adam and standard optimization procedures in terms of convergence and performance.
- **Reinforcement Learning (exploratory):** investigating whether an agent could adapt optimization parameters for Logistic Regression and SVM based on observed training and validation performance.

A hold-out validation set will be used to tune the learning rate, regularization strength and number of principal components. Learning curves and validation curves will help diagnose underfitting and overfitting.

### Generalization to unseen scenarios

Scenario-based validation will be performed by keeping one of the three training areas out at a time. All preprocessing steps, including scaling and PCA, will be fitted using only the training portion of each split to prevent data leakage.

## Evaluation

The primary evaluation metric is **accuracy**:

$$
\text{Accuracy} = \frac{TP + TN}{P + N}
$$

The confusion matrix will also be examined, with particular attention to **NLOS observations incorrectly classified as LOS**, since unreliable measurements can negatively affect positioning.

A possible future extension is to use the predicted classes to study pseudorange errors and positioning accuracy after excluding or down-weighting NLOS measurements.

## Project status

This README describes the project objectives and planned methodology. Implementation details, experimental results and conclusions can be added as the project progresses.

## References

1. POLYU IPNL, [“GNSS LOS/NLOS Classification”](https://www.kaggle.com/competitions/gnss-classification), Kaggle, 2022.
2. L.-T. Hsu et al., [“UrbanNav: An Open-Sourced Multisensory Dataset for Benchmarking Positioning Algorithms Designed for Urban Areas”](https://doi.org/10.33012/2021.17895), *Proceedings of ION GNSS+*, 2021, pp. 226–256.
3. B. Xu, Q. Jia, Y. Luo and L.-T. Hsu, [“Intelligent GPS L1 LOS/Multipath/NLOS Classifiers Based on Correlator-, RINEX- and NMEA-Level Measurements”](https://www.mdpi.com/2072-4292/11/16/1851), *Remote Sensing*, vol. 11, no. 16, p. 1851, 2019.

## Author

**Paolo Cerutto**
