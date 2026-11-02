# MSCS 634 Lab 5: Hierarchical and DBSCAN Clustering

## Purpose

This lab investigates how Agglomerative Hierarchical Clustering and DBSCAN group the 178 observations in the `sklearn.datasets` Wine dataset. It explores the 13 feature columns, standardizes them for distance-based analysis, visualizes clusters with PCA projections and a dendrogram, tests algorithm parameters, and evaluates the resulting labels with Silhouette, Homogeneity, and Completeness Scores.

## Contents

- `nmetha_lab5.ipynb`: Complete analysis and visualizations.
- `README.md`: Lab summary and run instructions.

## Methods

The Wine dataset contains 178 observations and 13 numerical features from three known wine classes. The target labels are not used to create clusters; they are used only afterward to calculate homogeneity and completeness. All features are standardized with `StandardScaler` before clustering.

The notebook tests Agglomerative Clustering with `n_clusters` values of 2, 3, 4, and 5. It also creates a Ward-linkage dendrogram and PCA-based scatter plots.

For DBSCAN, the notebook tests several `eps` values and `min_samples` values. It reports the number of clusters, noise points, Silhouette Score, Homogeneity Score, and Completeness Score. When DBSCAN produces fewer than two non-noise clusters, the Silhouette Score is reported as undefined. Noise points are excluded from DBSCAN metric calculations where appropriate.

## Results and Insights

The dataset has 178 rows, 13 numerical features, and three known wine classes. Standardization was important because feature scales vary substantially; without z-score scaling, high-range features such as `proline` would have disproportionate influence on Euclidean distances. PCA's first two components retain 55.4% of the standardized data's variance, so the plots provide a useful but incomplete two-dimensional view. Clustering itself uses all 13 standardized features, not the PCA projection.

### Hierarchical Clustering

The notebook tested `n_clusters` values 2 through 5. The Silhouette Score was highest at `n_clusters=3` (0.2774), compared with 0.2670 at 2 clusters, 0.2258 at 4, and 0.1867 at 5. The three-cluster solution also had Homogeneity 0.7904 and Completeness 0.7825. As the cluster count rose from 3 to 5, homogeneity increased slightly to 0.8106 while completeness fell to 0.5762, indicating that the extra splits made clusters purer but divided known classes across multiple clusters. In this run, three clusters provide the strongest balance of separation and class-label agreement. The dendrogram supports inspecting the merge structure and possible cut points rather than assuming the known three classes automatically dictate the answer.

### DBSCAN Parameter Effects

The tested `eps` values were 1.2, 1.5, 1.8, 2.0, 2.3, and 2.6, with `min_samples` values 3, 5, and 8. At `eps=1.2`, all 178 observations were noise for every `min_samples` value. At `eps=1.5`, only `min_samples=3` formed clusters (8 clusters, 147 noise points); values 5 and 8 still produced all noise. As `eps` increased, more observations generally joined clusters and nearby groups sometimes merged. For example, with `min_samples=5`, the run moved from no clusters at `eps=1.5`, to 7 clusters at 1.8, 5 clusters at 2.0, 2 clusters at 2.3, and 1 cluster at 2.6. Raising `min_samples` demanded denser neighborhoods: at `eps=2.0`, the number of clusters fell from 5 (`min_samples=3` or 5) to 3 (`min_samples=8`), while noise rose from 64 or 85 observations to 117.

DBSCAN's highest valid Silhouette Score was 0.4360 at `eps=2.0`, `min_samples=8`, with Homogeneity 0.8998 and Completeness 0.9320. However, this setting classified 117 of 178 observations (about 66%) as noise, leaving only 61 observations in the scored clusters. A less noise-heavy example, `eps=2.3`, `min_samples=5`, formed 2 clusters with 42 noise points and scored 0.3307 for silhouette, 0.5345 for homogeneity, and 0.9438 for completeness. The first configuration scores better on the retained subset; the second includes more observations. Which is preferable depends on whether cluster purity or coverage matters more.

For DBSCAN, noise points are excluded from silhouette, homogeneity, and completeness calculations. Therefore, its scores do not cover the same observations as Agglomerative scores, which use all 178 wines, and should not be compared as if they were calculated on identical data. The notebook records metrics as undefined when no valid clustering exists; one-cluster results also have no silhouette score. High homogeneity or completeness for a tiny retained subset should be interpreted alongside the noise count.

### Algorithm Tradeoffs

Agglomerative clustering is a good fit when a complete hierarchy and a chosen number of groups are useful. The dendrogram makes its merges interpretable, it assigns every observation, and the `n_clusters=3` run gave balanced metrics without discarding data. Its limitations are that the number of clusters must be selected and Ward linkage favors compact groups; it does not mark outliers as noise.

DBSCAN does not require a cluster count in advance and can identify noise or non-spherical density-connected groups. Its weakness on this dataset was parameter sensitivity: stricter settings returned all noise, while permissive settings merged the observations into one cluster. Intermediate settings often produced many noise points. The strongest DBSCAN silhouette came with substantial data exclusion, so its result needs a coverage caveat.

### Challenges and Decisions

- Feature magnitudes varied widely, so both algorithms were fitted to standardized features.
- PCA was reserved for plotting; fitting the algorithms on only two components would discard information.
- The DBSCAN grid includes parameter values that produce all-noise, one-cluster, and multi-cluster outcomes to make sensitivity visible.
- DBSCAN metrics exclude noise and are marked undefined where their mathematical preconditions are not met. Noise counts are reported beside scores to make coverage explicit.
- Parameters were chosen to show a range of densities rather than tuned solely to maximize a metric on a subset. The preferred configuration should account for separation, agreement with known labels, and the fraction of observations assigned to clusters.

## How to Run

From the project root:

```bash
python3 -m venv .venv
source .venv/bin/activate
python3 -m pip install jupyter numpy pandas matplotlib seaborn scikit-learn scipy ipykernel nbformat nbclient
jupyter notebook lab5/nmetha_lab5.ipynb
```

In VS Code, open `lab5/nmetha_lab5.ipynb`, select the `.venv` Python kernel, and choose **Run All**. Save the executed notebook before committing both `lab5/nmetha_lab5.ipynb` and `lab5/README.md` to the GitHub repository.
