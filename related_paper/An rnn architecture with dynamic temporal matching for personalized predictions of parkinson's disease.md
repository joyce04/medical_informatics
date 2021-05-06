## An rnn architecture with dynamic temporal matching for personalized predictions of parkinson's disease.
Che, Chao, et al. "An rnn architecture with dynamic temporal matching for personalized predictions of parkinson's disease." Proceedings of the 2017 SIAM International Conference on Data Mining. Society for Industrial and Applied Mathematics, 2017.

Aim :

- Evaluate the clinical similarities among patients by computing pairwise patient similarities

 - With similar patient cohorts we can perform targeted prognosis and design customized therapies.

Dataset: EMR

- patients whose primary diagnosis are either Idiopathic PD(case) or "No PD nor other neurological disorder(control)"
- missing values = last occurrence carry forward strategy

Model :

 - Apply a RNN(with GRU), which learns the similarity between two longitudinal patient record sequences through dynamically matching temporal patterns in patient sequences

 - We borrow the idea of Dynamic Time Warping(DTW) combining with a 2D-RNN architecture using a ranking loss function to learn the similarity between two temporal sequences varying in speed of evolving records.

 - 1. obtain the distance between two patient data sequences.(Euclidean distance)

 - 2. apply 2D-RNN to compute the global distance between two patient data tensors.

 - 3. apply a linear scoring function to obtain the final distance.


Evaluation :

 - Precision@K based on Euclidean distances
 - Root-Mean-Square Error = predicting NHY score

The temporal mismatch between patient event sequences may impact the similarity measure depending on the nature of events, disease mechanism and other factors.

DTW = The dynamic time warping (DTW) is an approximate pattern detection algorithm that measures similarity between two temporal sequences which may vary in speed. It predefined distance measure (e.g. Euclidean distance) so that two time series are optimally aligned through a warping path.