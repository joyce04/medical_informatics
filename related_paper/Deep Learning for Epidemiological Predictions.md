### Deep Learning for Epidemiological Predictions

Wu, Yuexin, et al. "Deep Learning for Epidemiological Predictions." *SIGIR*. 2018.

Why applying Deep learning?

- The temporal nature of epidemiology data and the need for real-time prediction by the system makes the problem residing in the category of time-series forecasting or prediction. 

Previous Approaches :

- Compartmental models where the whole population is divided into different groups (of susceptible, infective, and recovered), and the transition among groups are modeled by differential equations.
- Autoregressive models : model the future state as a linear combination of past data points.
  But the signal sources are treated independently from each other during the training process, which would be too simplistic.
- Gaussian Process Regression : extends the prediction power by utilizing a non-linear kernel (e.g. radial basis function) for modeling complex temporal patterns. It assumes that the future predicting profiles altogether are sampled from a Gaussian distribution.

Proposed Approach :

- The overall structure is composed of 3 parts: a CNN for capturing correlation between signals, a RNN for linking up the dependencies in the temporal dimension and the residual links for fast training and overfitting prevention.
- Utilize an Gated Recurrent Unit to capture the temporal dependencies in the data.
  Compare to the traditional Long-Short Time Machine (LSTM) where there are 3 gates, an GRU has fewer parameters (2 gates) to be trained and thus is more suitable in the data-deficient case.
- **Instead of using the standard residual links that each layer may only connect to its neighbors within 2-4 layers, we use a similar structure where the final layer “densely” links to nearly all previous layers. The benefit of such design are two-fold: such design alleviates the gradient vanishing phenomenon during training which stabilizes the process; also the links may possibly introduce highly stabilizes the process; also the links may possibly introduce highly relevant long-jump data information to the final output (e.g. the annual epidemiology patterns), thus giving out a more accurate predictor. **