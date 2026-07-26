# FAHRNet

Official Kaggle notebook implementation of **FAHRNet**, a
frequency-aware hybrid refocusing network for ×4 remote sensing image
super-resolution.

## Paper

**Frequency-Aware Hybrid Refocusing Network for Remote Sensing Image Super-Resolution**

Md Khalid Hasan Sakib, Dristi Datta, and Manoranjan Paul.

Paper status: Under review.

## Overview


FAHRNet integrates:

- Vision-LSTM-inspired contextual guidance
- Mamba-inspired row-column sequence mixing
- CNN-based local refinement
- Haar-frequency enhancement
- Branch-wise adaptive fusion
- Gated Top-Down Fusion

The training objective combines Charbonnier reconstruction loss,
Haar high-frequency loss, and Sobel edge loss.

## Repository Contents

### RSSCN7 Notebook

`notebooks/FAMamba_LSTMConvSR_RSSCN7_C128.ipynb`

This notebook contains the C = 128 implementation for the RSSCN7
experiment, including:

- Dataset indexing and deterministic splitting
- HR-LR pair generation
- FAMamba-LSTMConvSR architecture
- Training and validation
- Test evaluation
- Image-quality metrics
- Per-image statistical analysis
- Qualitative output generation
- Feature-response visualization

### UAVid Notebook

`notebooks/FAMamba_LSTMConvSR_UAVid_C128.ipynb`

This notebook contains the C = 128 implementation for the UAVid
experiment, including:

- UAVid sequence and frame selection
- Train-validation data from the patched dataset
- Test data preparation from the raw UAVid source
- HR-LR pair generation
- FAMamba-LSTMConvSR architecture
- Training and validation
- Test evaluation
- Image-quality metrics
- Per-image statistical analysis
- Qualitative output generation
- Feature-response visualization

## Datasets

The original datasets are not redistributed in this repository.
They are publicly available through Kaggle and remain subject to
the terms and licenses specified by their providers.

### RSSCN7

https://www.kaggle.com/datasets/yangpeng1995/rsscn7

### UAVid Patched 512 × 512 Dataset

https://www.kaggle.com/datasets/mastershomya/uavid-patched-512x512

### UAVid v1

https://www.kaggle.com/datasets/dasmehdixtr/uavid-v1

## Dataset Usage in the Notebooks

### RSSCN7

The RSSCN7 notebook uses a deterministic class-balanced partition:

- Training: 1,120 images
- Validation: 280 images
- Testing: 1,400 images
- Random seed: 42
- HR size: 400 × 400
- LR size: 100 × 100
- Scale factor: ×4
- Model width: C = 128
- Training epochs: 500

### UAVid

The UAVid notebook uses:

- Patched 512 × 512 dataset for training and validation
- Raw UAVid v1 images for testing
- Frames 000000 and 000900
- Training sequences: 1–15 and 31–35
- Validation sequences: 16–20, 36, and 37
- Testing sequences: 21–30 and 38–42
- Training: 1,600 samples
- Validation: 560 samples
- Testing: 1,200 samples
- HR size: 512 × 512
- LR size: 128 × 128
- Scale factor: ×4
- Model width: C = 128
- Training epochs: 500
- Random seed: 42

## Running the Code

The notebooks were developed for the Kaggle environment.

1. Download the required dataset from the corresponding Kaggle link.
2. Open the relevant notebook in Kaggle.
3. Add the required Kaggle dataset or datasets as notebook inputs.
4. Enable a GPU accelerator.
5. Verify that the dataset paths match those defined in the notebook.
6. Run the notebook cells in order.

The RSSCN7 notebook activates RSSCN7 through:

```python
USE_RSSCN7 = True
USE_UAVID = False
USE_POTSDAM = False
