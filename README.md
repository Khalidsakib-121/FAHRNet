# FAHRNet

Official Kaggle notebook implementation of **FAHRNet**, a
Frequency-Aware Hybrid Refocusing Network for ×4 remote sensing image
super-resolution.

## Paper

**FAHRNet: Frequency-Aware Hybrid Refocusing Network for Remote Sensing Image Super-Resolution**

Md Khalid Hasan Sakib, Dristi Datta, Manoranjan Paul, and Davina White.

**Paper status:** Revised manuscript under review.

## Overview

FAHRNet integrates:

- Wavelet-Token Bidirectional LSTM Guidance (WT-BiLG)
- Gated Row-Column Directional Mixer (GRDM)
- CNN-based local refinement
- Haar-frequency correction
- Branch-wise adaptive fusion
- Gated Top-Down Fusion (GTF)

WT-BiLG uses a standard bidirectional LSTM over reduced-resolution
wavelet tokens for contextual guidance.

GRDM is an explicitly convolutional directional mixer using gated
depthwise row-column convolutions. It does **not** implement Mamba,
selective state-space recurrence, or an S6-style state-space scan.

The training objective combines Charbonnier reconstruction loss,
Haar high-frequency loss, and Sobel edge loss:

```text
L_total = L_rec + 0.05 L_freq + 0.02 L_edge
```

No perceptual or adversarial loss is used.

## Repository Contents

### RSSCN7 Notebook

`notebooks/FAMamba_LSTMConvSR_RSSCN7_C128.ipynb`

This notebook contains the C = 128 implementation for the RSSCN7
experiment, including:

- Dataset indexing and deterministic splitting
- HR-LR pair generation
- FAHRNet architecture
- Training and validation
- Test evaluation
- Image-quality metrics
- Per-image statistical analysis
- Qualitative output generation
- Feature-response visualization

### UAVid Notebook

`notebooks/FAMamba_LSTMConvSR_UAVid_C128.ipynb`

This notebook contains the C = 128 implementation for the UAVid
experimental set, including:

- UAVid sequence and frame selection
- Train-validation data from the patched dataset
- Test data preparation from the raw UAVid source
- HR-LR pair generation
- FAHRNet architecture
- Training and validation
- Test evaluation
- Image-quality metrics
- Per-image statistical analysis
- Qualitative output generation
- Feature-response visualization

> **Note:** The notebook filenames are retained from an earlier development
> stage for reproducibility. In the revised manuscript, the recurrent module
> is named **Wavelet-Token Bidirectional LSTM Guidance (WT-BiLG)** and the
> directional module is named **Gated Row-Column Directional Mixer (GRDM)**.
> The proposed model is no longer described as Mamba-inspired or
> Vision-LSTM-inspired.

## Datasets

The original datasets are not redistributed in this repository.
They are publicly available from their respective providers and remain
subject to the terms and licenses specified by those providers.

### RSSCN7

https://www.kaggle.com/datasets/yangpeng1995/rsscn7

### UAVid Patched 512 × 512 Dataset

https://www.kaggle.com/datasets/mastershomya/uavid-patched-512x512

### UAVid v1

https://www.kaggle.com/datasets/dasmehdixtr/uavid-v1

### UCMerced LandUse

The revised manuscript additionally uses UCMerced LandUse for a
same-objective, parameter-matched comparison between FAHRNet and EDSR-CM.

Source:

https://hf.co/datasets/torchgeo/ucmerced/resolve/7c5ef3454d9b1cccfa7ccde0c01fc8f00a45909a/UCMerced_LandUse.zip

Archive MD5:

```text
5b7ec56793786b6dc8a908e8854ac0e4
```

## Dataset Usage in the Experiments

### RSSCN7

The RSSCN7 experiment uses a deterministic class-balanced partition:

- Training: 1,120 images
- Validation: 280 images
- Testing: 1,400 images
- Random seed: 42
- HR size: 400 × 400
- LR size: 100 × 100
- Scale factor: ×4
- Primary model width: C = 128
- Training epochs: 500

### UAVid

The UAVid experiment uses:

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
- Primary model width: C = 128
- Training epochs: 500
- Random seed: 42

The selected UAVid frames are deterministically divided into 40
512 × 512 crops using a 5 × 8 crop construction.

The train, validation, and test partitions are sequence-disjoint.
This prevents direct reuse of the same source sequence across partitions,
but does not establish geographic non-overlap between independently
recorded scenes.

An immutable Kaggle version identifier was not retained in the original
experimental record. Reproducibility is therefore based on the disclosed
dataset identifiers, fixed sequence and frame identities, and deterministic
crop construction.

### UCMerced

The revised UCMerced experiment uses:

- Total images: 2,100
- Number of classes: 21
- Training: 840 images
- Validation: 210 images
- Testing: 1,050 images
- Split: 40/10/50 images per class
- Random seed: 42
- HR size: 256 × 256
- LR size: 64 × 64
- Scale factor: ×4

The same split, training objective, optimization protocol, checkpoint
selection rule, and evaluation pipeline are used for FAHRNet and the
parameter-matched EDSR-CM control.

## Revised Manuscript Experiments

Following peer-review feedback, the experimental evaluation was expanded
to better separate architecture, training-objective, model-capacity,
directional-geometry, degradation, and computational effects.

### Same-Objective EDSR Control

EDSR was additionally trained using the same
reconstruction-frequency-edge objective as FAHRNet.

RSSCN7:

- FAHRNet: 29.0292 dB PSNR-Y
- EDSR-Obj: 28.8457 dB PSNR-Y
- Same-objective difference: 0.1835 dB in favor of FAHRNet

UAVid:

- FAHRNet: 29.3791 dB PSNR-Y
- EDSR-Obj: 29.1235 dB PSNR-Y
- Same-objective difference: 0.2556 dB in favor of FAHRNet

On RSSCN7, objective alignment substantially narrows the original
framework-level performance difference. Therefore, the original gap is
not attributed to architecture alone.

The effect of the composite training objective is architecture- and
dataset-dependent.

### Reconstruction-Only FAHRNet Control

FAHRNet was also evaluated using reconstruction loss only.

RSSCN7:

- Reconstruction-only FAHRNet: 29.0140 dB
- Full-objective FAHRNet: 29.0292 dB
- Difference: +0.0152 dB

UAVid:

- Reconstruction-only FAHRNet: 29.3444 dB
- Full-objective FAHRNet: 29.3791 dB
- Difference: +0.0347 dB

These small differences are interpreted as single-seed
objective-sensitivity evidence rather than statistically established
training-run superiority.

### Parameter-Matched UCMerced Control

FAHRNet:

- PSNR-Y: 28.6254 dB
- Parameters: 25.006 M
- Standardized MACs: 292.569 G

EDSR-CM:

- PSNR-Y: 28.8026 dB
- Parameters: 24.244 M
- Standardized MACs: 463.488 G

EDSR-CM was selected before training based on parameter proximity rather
than validation or test performance.

EDSR-CM contains 24,244,227 trainable parameters compared with
25,006,099 for FAHRNet, i.e., 3.0467% fewer parameters than FAHRNet.

Under this stricter comparison, EDSR-CM achieves higher HR-referenced
reconstruction fidelity.

FAHRNet, however, requires 36.8766% fewer standardized MACs and attains
the higher CLIPIQA score.

This comparison is parameter-matched, not computation-matched.

Lower standardized MAC count is not interpreted as faster measured
inference.

### GRDM Directional Controls

The revised manuscript evaluates the proposed parallel GRDM formulation
against two controlled alternatives.

Parallel GRDM:

- PSNR-Y: 29.3791 dB
- Parameters: 25.006 M
- Standardized MACs: 292.569 G

Serial 1 × 9 → 9 × 1 directional control:

- PSNR-Y: 29.3102 dB
- Parameters: 25.006 M
- Standardized MACs: 292.569 G

9 × 9 depthwise control:

- PSNR-Y: 29.3645 dB
- Parameters: 25.133 M
- Standardized MACs: 294.683 G

The serial formulation is parameter-identical to the proposed parallel
GRDM.

Parallel GRDM exceeds the serial control by 0.0689 dB PSNR-Y.

Parallel GRDM exceeds the 9 × 9 depthwise control by 0.0146 dB PSNR-Y.

The close result of the 9 × 9 depthwise control indicates that enlarged
receptive-field processing explains a substantial part of the observed
behavior.

The parallel row-column organization therefore provides a modest
distortion-oriented benefit under the tested setting rather than evidence
of universal superiority over large-kernel processing.

### Cross-Degradation Evaluation

The frozen bicubic-trained FAHRNet and same-objective EDSR models were
evaluated under three conditions.

D0:

- Antialiased bicubic ×4 degradation

D1:

- Normalized 7 × 7 Gaussian blur
- σ = 1.2
- Followed by bicubic downsampling

D2:

- D1 degradation
- Additional zero-mean Gaussian LR noise
- Noise standard deviation: 0.01
- Values clipped to [0,1]

No retraining, fine-tuning, test-time adaptation, or checkpoint
reselection is used.

FAHRNet retains higher absolute PSNR-Y and SSIM-Y and lower LPIPS than
same-objective EDSR under the tested D1 and D2 conditions on RSSCN7 and
UAVid.

However, FAHRNet deteriorates more relative to its own D0 operating point.

Therefore, the revised manuscript does **not** claim superior degradation
invariance or validated real-sensor robustness.

## Model Configuration

The primary FAHRNet configuration uses:

- Feature width: C = 128
- Hybrid Neural Blocks: K = 4
- FADRBs per FADRG: J = 4
- GRDM directional kernel length: 9
- WT-BiLG hidden dimension: C/2 per direction
- WT-BiLG MLP expansion ratio: 2.0
- Learnable residual-scale initialization: 0.1
- Super-resolution factor: ×4

The compact C = 64 configuration changes only the internal feature width.

The following remain unchanged between C = 128 and C = 64:

- Number of Hybrid Neural Blocks
- FADRG depth
- Module topology
- GRDM directional kernel length
- Upsampling structure
- Residual initialization
- Training objective

Because FAHRNet combines pointwise/full convolutions, depthwise
operations, recurrent layers, normalization, biases, and fixed
input/output projections, parameter and MAC scaling is not assumed to be
exactly quadratic in C.

## Training Protocol

The principal experiments use:

- PyTorch 2.10.0
- torchvision 0.25.0
- NVIDIA Tesla T4
- Automatic mixed precision during training
- Random seed: 42
- Training epochs: 500
- Optimizer: Adam
- β1 = 0.9
- β2 = 0.99
- Initial learning rate: 1 × 10^-4
- StepLR at epoch 250
- Learning-rate decay factor: 0.5
- Physical batch size: 8
- Effective batch size: 16
- Gradient clipping: 1.0
- Checkpoint selection: highest validation PSNR-Y

Training uses randomly sampled:

- 32 × 32 LR patches
- 128 × 128 HR patches

Training augmentation includes:

- Horizontal flips
- Vertical flips
- 0° rotation
- 90° rotation
- 180° rotation
- 270° rotation

## Evaluation Protocol

Paired LR images are generated using:

```python
torchvision.transforms.functional.resize
```

with:

```python
InterpolationMode.BICUBIC
antialias=True
```

Final reconstructed outputs are transformed back to image space and
clipped to [0,1].

Evaluation is performed using floating-point values without conversion to
8-bit integer images.

No border shaving is used for PSNR-Y or SSIM-Y.

The luminance channel is computed as:

```text
Y = 0.257R + 0.504G + 0.098B + 16/255
```

The revised manuscript reports:

- PSNR-Y
- SSIM-Y
- MS-SSIM
- ERGAS
- GMSD
- LPIPS
- DISTS
- CLIPIQA

CLIPIQA is treated as an auxiliary no-reference perceptual-quality
indicator rather than as a replacement for HR-referenced fidelity
measures.

## Computational Analysis

FAHRNet C = 128:

- Parameters: 25.006 M
- Standardized MACs: 292.569 G

FAHRNet C = 64:

- Parameters: 6.536 M
- Standardized MACs: 76.476 G

Reducing the feature width from C = 128 to C = 64 reduces both parameters
and standardized MACs by approximately 73.9%.

The principal C = 128 parameter allocation is:

- WT-BiLG: approximately 0.934 M parameters
- Stacked FADRG hierarchy: approximately 22.238 M parameters
- GTF: approximately 1.312 M parameters

### T4 Inference Throughput

At 128 × 128 LR resolution with batch size 1:

FAHRNet C = 128:

- AMP: 5.9341 FPS
- FP32: 4.0359 FPS

EDSR-CM:

- AMP: 11.1649 FPS
- FP32: 5.0300 FPS

Although FAHRNet requires fewer standardized MACs than EDSR-CM,
EDSR-CM is faster in measured T4 inference.

Therefore, standardized arithmetic workload and measured wall-clock
latency are interpreted separately.

### Module-Level Profiling

At 128 × 128 LR resolution:

WT-BiLG:

- FP32: 7.5996 ms
- AMP: 7.7701 ms

Four-block FADRG:

- FP32: 42.1521 ms
- AMP: 29.2688 ms

These isolated measurements are diagnostic microbenchmarks and are not
interpreted as additive decompositions of full-model latency.

## Component Sensitivity

Component-removal experiments are reported together with their parameter
and standardized-MAC changes.

The full C = 128 model uses:

- Parameters: 25.006 M
- MACs: 292.569 G

Without WT-BiLG:

- Parameters: 24.072 M
- MACs: 285.454 G

Without GRDM:

- Parameters: 20.720 M
- MACs: 222.575 G

Without the Haar-frequency branch:

- Parameters: 12.919 M
- MACs: 210.730 G

GTF replaced by ungated concatenation-projection:

- Parameters: 24.875 M
- MACs: 290.422 G

Because these interventions change model capacity and computation, they
are interpreted as **component-sensitivity analyses** rather than
parameter-matched causal tests.

The paired bootstrap analyses quantify held-out image or patch variation
conditional on the frozen trained checkpoints. They do not quantify
training-seed uncertainty.

## Semantic-Retention Assessment

The revised manuscript additionally reports a fixed-classifier
scene-level semantic-retention experiment on RSSCN7.

An ImageNet-pretrained ResNet-18 is trained exclusively using the HR
RSSCN7 training partition.

The classifier is selected using the HR validation set only.

Selected checkpoint:

- Epoch: 115
- Validation accuracy: 0.9750
- Validation macro-F1: 0.97499

Using the frozen classifier on FAHRNet reconstructions:

- Test accuracy: 0.8636
- Macro-F1: 0.8621
- HR-domain accuracy retention: 91.11%
- HR-domain macro-F1 retention: 90.97%

The class-wise analysis also identifies remaining confusion patterns.

Parking → Industrial:

- 49 of 200 test images

Farmland → Grassland:

- 36 of 200 test images

For farmland:

- Bicubic F1: 0.8548
- FAHRNet F1: 0.8532

The semantic experiment is interpreted only as evidence of
scene-level semantic-information retention under one fixed HR-trained
classifier.

It does **not** establish:

- Object recovery
- Semantic-segmentation fidelity
- Local geographic correctness
- Correctness of every reconstructed fine-scale structure
- Operational downstream-task performance

## Running the Code

The notebooks were developed for the Kaggle environment.

1. Download the required dataset from the corresponding source.
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
```

The UAVid experiment requires both the patched UAVid dataset and the raw
UAVid v1 source because they are used for different parts of the frozen
experimental construction.

## Revision Note

The manuscript underwent substantial experimental revision during peer
review.

The current terminology and interpretation should therefore be used when
reading this repository.

In particular:

- **WT-BiLG** is the current name for the Wavelet-Token Bidirectional LSTM
  Guidance module.
- **GRDM** is the current name for the Gated Row-Column Directional Mixer.
- **GTF** denotes Gated Top-Down Fusion.
- GRDM is explicitly convolutional and is not presented as a Mamba or
  selective state-space model.
- WT-BiLG uses a standard bidirectional LSTM and is not presented as a
  Vision-LSTM or xLSTM implementation.
- Same-objective EDSR controls were added on RSSCN7 and UAVid.
- A parameter-matched EDSR-CM comparison was added on UCMerced.
- Serial and 9 × 9 depthwise GRDM controls were added.
- Cross-degradation evaluation was added.
- Standardized MAC count and measured runtime are interpreted separately.
- Component-removal results are interpreted as sensitivity analyses rather
  than parameter-matched causal isolation.
- Small single-seed ablation differences are interpreted conservatively.

Historical notebook filenames retain earlier development terminology for
reproducibility and compatibility with the original experimental
artifacts.

## Scope and Limitations

The revised evidence supports FAHRNet as a competitive
local-directional-frequency architecture for ×4 remote sensing image
super-resolution.

The current study does not claim:

- Universal architectural superiority
- Superior degradation invariance
- Validated real-sensor robustness
- Operational deployment readiness
- Mamba/state-space behavior for GRDM
- Training-seed-level statistical resolution of very small ablation effects

Current study boundaries include:

- Single-seed primary training experiments
- Predominantly synthetic degradation
- Same-objective external control limited primarily to EDSR
- Parameter-matched but not computation-matched UCMerced control
- Fixed GRDM directional kernel length of 9
- Sequence-disjoint but not proven geographically disjoint UAVid scenes
- One fixed HR-trained classifier for semantic-retention analysis

Future work includes:

- Repeated-seed training
- Additional super-resolution scale factors
- Real or physically calibrated degradation models
- Additional sensor domains
- Broader capacity- and compute-matched comparisons
- Object-detection evaluation
- Semantic-segmentation evaluation
- Geospatial boundary analysis

## Citation

If you use this repository, please cite the FAHRNet paper once the final
bibliographic information becomes available.
