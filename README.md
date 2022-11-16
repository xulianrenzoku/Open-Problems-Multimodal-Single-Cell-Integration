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
  - Define a four-layer neural network with a batch normalization and a RELU (activation function) in between layers. Put a relu after the final layer since the targets are non-negative
  - Use [one cycle policy](https://sgugger.github.io/the-1cycle-policy.html) for faster training process with parameters determined by heuristic results from outputs of the `find_lr` function 
  - Training Notebook: [msci-multi-mlp-sparse-lrs](https://github.com/xulianrenzoku/Open-Problems-Multimodal-Single-Cell-Integration/blob/main/notebooks/msci-multi-mlp-sparse-lrs.ipynb)
  - Inference Notebook: [msci-multi-mlp-sparse-lrs-inference.ipynb](https://github.com/xulianrenzoku/Open-Problems-Multimodal-Single-Cell-Integration/blob/main/notebooks/msci-multi-mlp-sparse-lrs-inference.ipynb)

**CITEseq**
- Data Preprocessing: Reduce dimension of the gene expression data
  - Due to the sparcity of the data, use truncated SVD to reduce the dimensions from 23,418 to 64
  - Keep features that have high correaltions with the targets
  - Notebook: [msci-citeseq-mlp-tsvd-if.ipynb](https://github.com/xulianrenzoku/Open-Problems-Multimodal-Single-Cell-Integration/blob/main/notebooks/msci-citeseq-mlp-tsvd-if.ipynb)
- Modeling: Use k-fold multilayer perceptron to predict
  - Standardize targets before training
  - Divide inputs into five folds
  - Define a four-layer neural network with a SELU (activation function) in between layers
  - Use one cycle policy for faster training process with parameters determined by heuristic results from outputs of the `find_lr` function 
  - Given targets were standardized before training, use predictions from trained models and the orginal targets to get the correlation for cross-validation 
  - Notebook: [msci-citeseq-mlp-tsvd-model-kf2.ipynb](https://github.com/xulianrenzoku/Open-Problems-Multimodal-Single-Cell-Integration/blob/main/notebooks/msci-citeseq-mlp-tsvd-model-kf2.ipynb)

---
### Discussion
Below are things that I learned:

**Multiome**
- It is great to learn techniques of data compression since it is not realistic to load a gigantic data file that eats up all the RAMs. After transforming data frames with a lot of zeroes into sparse matrices, I also learned how to set up dataloaders with `SparseDataset` with setting up custom function for the `collate_fn` parameter in `DataLoader`.
- Using one cycle policy as learning rate scheduler could lead to faster convergence. It may not lead to top solutions in competitions, but I can see this being a super useful technique in work settings. 

**CITEseq**
- It seems there is a less-is-better effect with my experience in doing dimension reduction. I tried to reduce the gene expression data to 512, 128 and 64 features separately, and it seems that I would achieve better results with less data even though 64 features clearly have a lower explained variance ratio than that of 512 features.
- SELU worked better than BatchNorm + RELU. I experimented BatchNorm + SELU also, but it did not give better results and created confusions for me to pick the right max learning rate for the one cycle policy. Also, in theory, such a setup is not recommended since BatchNorm + SELU (an activation function with self-normalizing propertices) means you do normalization twice between layers.
- When going through notebooks of benchmark models on Kaggle, I noticed that people would standardize targets before modeling. I was skeptical at first since I assumed modeling against the standardized targets would hurt the correlation score with the original targets. However, through experiments, it seems such a procedure actually led to better results.

Things I wished I could do for improvement:

**Multiome**
- In the later stage of the competition, I was more focused on the CITEseq task. If given more time, I would do k-fold on the multiome part.
- Many fellow Kagglers reduced dimensions not only on inputs but also on targets, and reversed back to 23,418 dimensions after training. I probably should try this method in retrospect, since it would definitely speed up the training process and probably lead to better results. 

**CITEseq**
- In the end, I was 0.002 away from a medal. Since the CITEseq task is weighted more heavily in this competition, getting more accurate on this task would help me to cross that final one-yard line, I assumed. If there is more time, I would try techniques with a longer learning process but better upside. Using Adam + one cycle policy helped me to get faster convergence but I probably need to try something else to get 0.002 better.
