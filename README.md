# Open-Problems-Multimodal-Single-Cell-Integration

[Open Problems - Multimodal Single-Cell Integration](https://www.kaggle.com/competitions/open-problems-multimodal)

---
### About the Competition
- **Task:** Predict how DNA, RNA, and protein measurements co-vary in single cells as bone marrow stem cells develop into more mature blood cells. There are two tasks specifically:
  - Multiome: Given **chromatin accessibility (DNA)**, predict **gene expression (RNA)**
    - Number of input features: 228,942
    - Number of output features: 23,418
  - CITEseq: Given **gene expression (RNA)**, predict **protein levels**
    - Number of input features: 23,418
    - Number of output features: 140
- **Eval Metric:** According the competition host at Kaggle, they "use the Pearson correlation coefficient to rank submissions". 

---
### Modeling
**Multiome**
- Data Preprocessing: Transform .h5 files to sparse matrix
  - Compress the size of data from 27G to 7.1G
  - Notebook: [msci-h5-sparse-transform.ipynb](https://github.com/xulianrenzoku/Open-Problems-Multimodal-Single-Cell-Integration/blob/main/notebooks/msci-h5-sparse-transform.ipynb)
- Modeling: Use multilayer perceptron to predict
  - Set up dataloaders with `SparseDataset` that loads sparse matrices nicely in Kaggle's GPU settings
  - Define a four-layer neural network with a batch normalization and a relu (activation function) in between layers. Put a relu after the final layer since the targets are non-negative
  - Use [one cycle policy](https://sgugger.github.io/the-1cycle-policy.html) for faster training process with parameters determined by heuristic results from outputs of the `find_lr` function 
  - Training Notebook: [msci-multi-mlp-sparse-lrs](https://github.com/xulianrenzoku/Open-Problems-Multimodal-Single-Cell-Integration/blob/main/notebooks/msci-multi-mlp-sparse-lrs.ipynb)
  - Inference Notebook: [msci-multi-mlp-sparse-lrs-inference.ipynb](https://github.com/xulianrenzoku/Open-Problems-Multimodal-Single-Cell-Integration/blob/main/notebooks/msci-multi-mlp-sparse-lrs-inference.ipynb)

**CITEseq Model**
- Data Preprocessing: Reduce dimension of the gene expression data
  - Due to the sparcity of the data, use truncated SVD to reduce the dimensions from 23,418 to 64
  - Keep features that have high correaltions with the targets
  - Notebook: [msci-citeseq-mlp-tsvd-if.ipynb](https://github.com/xulianrenzoku/Open-Problems-Multimodal-Single-Cell-Integration/blob/main/notebooks/msci-citeseq-mlp-tsvd-if.ipynb)
- Modeling: Use k-fold multilayer perceptron to predict
  - Standardize targets before training
  - Divide inputs into five folds
  - Define a four-layer neural network with a selu (activation function) in between layers
  - Use one cycle policy for faster training process with parameters determined by heuristic results from outputs of the `find_lr` function 
  - Given targets were standardized before training, use predictions from trained models and the orginal targets to get the correlation for cross-validation 
  - Notebook: [msci-citeseq-mlp-tsvd-model-kf2.ipynb](https://github.com/xulianrenzoku/Open-Problems-Multimodal-Single-Cell-Integration/blob/main/notebooks/msci-citeseq-mlp-tsvd-model-kf2.ipynb)

---
### Discussion
