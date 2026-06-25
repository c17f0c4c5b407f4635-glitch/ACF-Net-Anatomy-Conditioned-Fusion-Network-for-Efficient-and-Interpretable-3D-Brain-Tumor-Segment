# ACF-Net

PyTorch notebooks and reproducibility materials for ACF-Net.

ACF-Net is a 3D brain tumor segmentation model that combines multi-modal BraTS MRI with FastSurfer-derived anatomical priors. The model uses a dual-stream MRI/anatomy backbone, three specialized expert heads, and region-wise anatomy-conditioned fusion to produce the final tumor segmentation.

## Files

```text
ACF_Net.ipynb        Final ACF-Net notebook
ACF_Net-base.ipynb   Baseline ACF-Net variant
README.md
```

## Model summary

ACF-Net takes four MRI modalities as input:

```text
T1c, T1n, T2w, T2-FLAIR
```

FastSurfer is run offline on the T1n image to generate anatomical outputs. These are converted into:

```text
A: 9-channel anatomical prior volume
R: coarse anatomical region map
```

The anatomical prior volume is passed through the anatomy encoder, while the coarse region map is used by the anatomy-aware expert and the anatomy-conditioned fusion module.

The model has three expert heads:

```text
Context expert
Boundary expert
Anatomy-aware expert
```

Each expert produces logits. The fusion module estimates region-wise expert weights and combines the expert logits into the final fused logit volume. The final segmentation mask is produced with an argmax over voxel classes.

## Output labels

The model predicts voxel-level labels:

```text
0: background
1: necrotic or non-enhancing tumor core
2: peritumoral edema
3: enhancing tumor
```

For BraTS-style evaluation, these labels are grouped as:

```text
WT = {1, 2, 3}
TC = {1, 3}
ET = {3}
```

## Dataset split

The experiments use the labeled [BraTS 2023 adult glioma cohort](https://www.synapse.org/Synapse:syn51156910/wiki/621282).


The internal split used in the paper is:

```text
Training: 751 cases
Validation: 250 cases
Internal test: 250 cases
Split seed: 42
```

The reported results are internal held-out results and should not be interpreted as official BraTS hidden-test scores.

## Training setup

The final ACF-Net model was trained on Google Colab Pro+ using an NVIDIA L4 GPU with high-RAM mode enabled.

Main training settings:

```text
Patch size: 96 x 96 x 96
Optimizer: AdamW
Learning rate: 1e-4
Weight decay: 1e-5
Scheduler: ReduceLROnPlateau
Maximum epochs: 120
Early stopping patience: 22
Gradient clipping: 12.0
Mixed precision: enabled
```

Patch sampling:

```text
Tumor-containing patches: 0.70
ET-focused patches: 0.30 when ET voxels are available
```

## FastSurfer preprocessing

FastSurfer preprocessing was run offline in a Dockerized Ubuntu environment using:

```text
AMD Ryzen 9 5900HS
NVIDIA RTX 3080 Laptop GPU
16 GB RAM
32 GB virtual memory
```

The following FastSurfer outputs were used:

```text
aparc.DKTatlas+aseg.deep.mgz
aseg.auto_noCCseg.mgz
mask.mgz
```

These outputs were aligned to BraTS MRI space and cached before model training.

## Reported internal performance

Final ACF-Net internal held-out results:

```text
WT Dice:   0.9155
TC Dice:   0.8712
ET Dice:   0.8256
Mean Dice: 0.8707
Mean HD95: 4.1018
```

Model size:

```text
30.37M trainable parameters
```
