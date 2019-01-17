### Clustervision: Visual supervision of unsupervised clustering.

Kwon, Bum Chul, et al. "Clustervision: Visual supervision of unsupervised clustering." IEEE transactions on visualization and computer graphics 24.1 (2018): 142-151.
데이터셋과 목표 테스크에 가장 알맞은 클러스터를 고를 수 있도록 하는 시각화 툴

Aim : 데이터셋과 목표 테스크에 가장 알맞은 클러스터를 고를 수 있도록 하는 시각화 툴. 최적의 클러스터 방식과 클러스터를 제시한다.
We built Clustervision, a visual analytics tool that helps ensure data scientists find the right clustering among the large amount of techniques and parameters available.
Clustervision lets users rank and compare multiple clustering results based on quality metrics, provides meaningful feature-based summaries of clusters using visualizations and univariate statistics, and allows users to apply their domain expertise to constrain and steer clustering analysis.

Dataset : Electronic Medical Records (1,500 patients from Sutter Health)

Method :

​	Meta clustering : 여러 클러스터링 결과 중 최적을 추천하고 사용자가 선택하는 방식

​	활용하는 클러스터 종류

- Centroid-based : K-means, fuzzy c-mean. Require a priori knowledge of number of clusters, and a choice of metric.
- Connectivity-based : Hierarchical and Agglomerative. Use a linkage criterion and distance metric to split or join clusters.
- Density based : DBSCAN, OPTICS. Require parameters to quantify the density of the clusters and how to partition density.
- Low Dimensional Embeddings : Spectral Clustering
- Probabilistic : GAussian Mixture Models, Latent Dirichlet Allocation. Use probability distributions


Visualization :

1. Parallel Trends View : 각 데이터가 다른 클러스터 모델에 따라 어느 클러스터에 포함되었는지 확인 가능. 클릭하면, 각 데이터를 확인 할수 있다.
   User can view the feature values of the data points in different clusters in the Parallel Trends view.
   Users can click on an area path to show individual lines that represent corresponding data points within the cluster
2. t-SNE : 사용자 선택에 따라 가까운 데이터는 superpoint로 변형 가능.
   Similar points are represented by a superpoint.
3. Ranked Features : ANOVA F-Value계산을 통해 가장 중요한 피처 랭크를 시각화.
   피처 일부를 선택해 클러스터링을 진행할 수 있다.
   Utilize univariate statistics to compute whether there is a statistically significant relationship between each feature and each cluster. The resulting scores, based on the ANOVA F-Value, allow us to rank each feature in order of importance, as well as retrieve an associated p-value to ensure the relationship is statistically significant. 
4. Metrics : 클러스터 평가 항목을 Bar charts로 표기
   \- Cohesion measure how closely related are the data points in a cluster
   \- Separation quantifies how distinct a cluster is from other clusters
   \- Silhouette is the mean of all the silhouette scores for the cluster.
5. Kernel Density Plot : 해당 클러스터 내, 데이터의 분포를 피처 별로 확인 가능
   Vertical marks represent the mean values of the chosen cluster (striped vertical mark in red) and the currently selected data point (black) for continuous feature values.



Previous Work : HFpEF 환자 클러스터링 => 그룹별 치료 방안을 모색할 수 있다.

Case study1: 심부전 환자들 클러스터링

Finding clusters of similar patients to extract meaningful groups of patients with heart failure.