# MSCS 634 - Lab Assignment 3

**Name:** Chun-Shuo Sheu
**Course:** MSCS 634 - Big Data & Data Mining
**Lab:** Lab 3 - Clustering Analysis with K-Means and K-Medoids

## Purpose

The purpose of this lab is to explore two partitional clustering techniques - **K-Means** and **K-Medoids** - by applying them to the Wine Dataset from `sklearn`. The lab compares the two algorithms in terms of clustering quality using two internal/external metrics:

- **Silhouette Score** - measures how compact and well-separated the clusters are.
- **Adjusted Rand Index (ARI)** - measures how closely the produced clusters match the true cultivar labels.

We also visualize the resulting clusters and the corresponding centroids/medoids using a 2D PCA projection.


## How to Reproduce

```bash
python3 -m venv venv
source venv/bin/activate
pip install numpy pandas matplotlib scikit-learn jupyter
jupyter notebook MSCS_634_Lab_3.ipynb
```

Note: `sklearn-extra` (which provides `KMedoids`) is not compatible with Python 3.14, so the notebook includes a small from-scratch PAM (Partitioning Around Medoids) implementation of K-Medoids.

## Key Insights

- On the standardized Wine dataset, **K-Means produced better-defined clusters** than K-Medoids on both metrics:
  - K-Means: Silhouette = 0.285, ARI = 0.897
  - K-Medoids: Silhouette = 0.268, ARI = 0.741
- Both methods successfully separated the three wine cultivars, but K-Means aligned much more closely with the ground-truth labels (higher ARI).
- The Wine features roughly form convex, spherical clusters once standardized, which is exactly the setting K-Means excels in - explaining its stronger performance here.
- The clusters overlap noticeably in the 2D PCA projection because two principal components capture only ~55% of the variance, but the clustering itself is done in the full 13-dimensional standardized space.
- Visually, the K-Means centroids sit in geometrically empty space at the mean of each cluster, while the K-Medoids medoids fall on top of real wine samples - a useful property when the "representative" point of a cluster must be a real observation.

## When to Prefer Each Method

- **K-Means** is preferable when features are numeric, standardized, clusters are roughly convex, and the dataset is large (K-Means scales much better than PAM).
- **K-Medoids** is preferable when the data has significant outliers, when the cluster center must be an actual observation, or when a non-Euclidean distance metric is required.

## Challenges and Decisions

- **`sklearn-extra` incompatibility with Python 3.14.** The `sklearn-extra` package still imports the removed `distutils` module, so it fails to import on Python 3.14. Rather than downgrade the Python version, I implemented a small PAM-style K-Medoids class from scratch. It uses a precomputed pairwise distance matrix and iteratively (1) assigns points to the nearest medoid and (2) picks the point that minimizes total intra-cluster distance as the new medoid, until convergence.
- **Standardization.** Wine features have very different scales (e.g., `proline` in the hundreds vs. `nonflavanoid_phenols` under 1). Applying z-score normalization before clustering was essential - without it the high-magnitude features would dominate the Euclidean distances.
- **Visualization dimensionality.** The dataset has 13 features but plots need 2 dimensions, so I used PCA purely for visualization. All clustering and metric computation was done in the original 13-dimensional standardized feature space, not on the 2D projection.
- **Reproducibility.** A fixed `random_state=42` was used for K-Means, K-Medoids initialization, and PCA to make the results reproducible run-to-run.
