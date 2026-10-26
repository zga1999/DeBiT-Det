# DeBiT-Det

## Decoding-oriented Bi-Branch Transformer for Remote Sensing Rotated Object Detection

Research repository for **DeBiT-Det**, a transformer-based detector for arbitrarily oriented objects in remote sensing imagery.

> **Repository status:** This repository currently contains a manuscript-based project description. Training and inference code, configuration files, pretrained weights, and environment specifications have not been provided. The results below are reported in the supplied manuscript and have not been independently reproduced through this repository.

## Overview

Remote sensing images contain objects with arbitrary orientations, substantial scale variation, and dense spatial distributions. DeBiT-Det addresses two challenges:

- **Different geometric parameters require different representations.** Center, orientation, and scale describe different aspects of a rotated bounding box; predicting all of them from one final-stage feature can introduce representation interference.
- **Classification confidence can disagree with final localization quality.** Conventional classification supervision is based on proposal matching before regression, while detection quality depends on the boxes after regression.

The method combines **Hierarchical Perception-aware Regression Decoding (HPRD)** and **Classification-Localization Quality Alignment (CLQA)** to improve geometric modeling and classification reliability.

## Method

### Framework

The manuscript builds on the RotatedimTED detection framework with a Vision Transformer backbone, a feature pyramid network (FPN), an Oriented RPN, rotated RoI feature extraction, and a customized RoI head. The head contains hierarchical regression decoding, a main classification pathway, and a quality-calibrated auxiliary classification pathway.

### HPRD: Hierarchical Perception-aware Regression Decoding

HPRD assigns different groups of rotated-box parameters to intermediate Transformer representations according to their geometric requirements. In the method formulation and the final joint HPRD/CLQA configuration:

| Transformer stage | Prediction | Representation motivation |
| --- | --- | --- |
| TB1 | Center offsets `(dx, dy)` | Fine spatial information for object position |
| TB2 | Orientation offset `dtheta` | Directional structure and principal-axis information |
| TB3 | Scale offsets `(dw, dh)` | Broader regional information for object extent |
| TB4 | Main classification | Higher-level semantic information |

The geometric predictions are assembled in the box encoder's order, `(dx, dy, dw, dh, dtheta)`, to decode the final rotated box. The manuscript also evaluates other stage assignments in its ablation studies; the best assignment without CLQA differs from the joint configuration.

### CLQA: Classification-Localization Quality Alignment

CLQA retains the main RoI classification pathway and introduces a regression-calibrated auxiliary classification branch:

1. Extract auxiliary RoI features from the highest-level FPN feature map using the original proposals.
2. Decode each proposal using the predicted rotated-box offsets.
3. Measure the post-regression IoU between each decoded box and its matched ground-truth box.
4. Incorporate this quality into the auxiliary classification target, encouraging classification confidence to reflect final localization quality.

The main classification pathway uses cross-entropy supervision. The auxiliary pathway uses the focal-loss formulation described in the manuscript with quality-calibrated targets. The overall training objective is:

```text
L = L_RPN + L_bbox + L_CE(main scores, labels)
    + alpha * L_Focal(auxiliary scores, quality-calibrated labels)
```

The exact target-calibration implementation and inference-time use of the two classification pathways require the corresponding implementation; this repository does not supply speculative code for them.

### Theoretical motivation

The manuscript maps rotated boxes into two-dimensional Gaussian distributions and analyzes their KL divergence. Its analysis distinguishes scale-dependent center sensitivity, aspect-ratio-dependent orientation sensitivity, and relative scale discrepancy with orientation-scale coupling. These observations motivate matching different parameter groups to different feature stages.

It also distinguishes pre-regression proposal IoU from post-regression box IoU: a score trained to predict whether an original proposal satisfies a matching threshold does not inherently rank final boxes by their localization quality. This motivates post-regression quality calibration in CLQA.

## Evaluation

### Datasets

| Dataset | Evaluation focus | Preprocessing reported in the manuscript |
| --- | --- | --- |
| DOTA-v1.0 | Oriented detection across 15 categories | `1024 x 1024` patches with 200-pixel overlap |
| HRSC2016 | Oriented ship detection | Aspect-ratio-preserving resizing and padding to `800 x 800` |
| DIOR | Aerial object detection across 20 categories | `800 x 800` inputs |

The dataset is named **DIOR** in the supplied manuscript. The precise oriented-annotation variant and the evaluation split should be confirmed before reproduction. Its stated DIOR split counts are internally inconsistent, so they are not repeated here as a verified protocol.

### Selected manuscript results

| Dataset / setting | Backbone | Metric as reported | DeBiT-Det | Manuscript reference |
| --- | --- | --- | ---: | --- |
| DOTA-v1.0 test, single-scale | ViT-B | mAP (%) | **79.72** | Table I |
| HRSC2016 | ViT-B | mAP(07) (%) | **90.73** | Table II |
| HRSC2016 | ViT-B | mAP(12) (%) | **98.65** | Table II |
| DIOR | ViT-B | mAP (%) | **69.83** | Table III |

These values retain the manuscript's metric names. They belong to different evaluation protocols and should not be compared directly across rows. Ablation results elsewhere in the manuscript use a different reported setting and are not substituted for the benchmark results above.

### Implementation details reported in the manuscript

- Framework: PyTorch and MMRotate, with FPN components from MMDetection.
- Training: 12 epochs with AdamW, learning rate `2e-4`, and weight decay `0.05`.
- Schedule: 500-iteration linear warmup and learning-rate decay at epochs 9 and 11.
- Gradient clipping: maximum norm `35`.
- Hardware: eight NVIDIA RTX 4090 GPUs for training; NVIDIA L20 GPUs for evaluation.

These are manuscript-reported settings, not a tested installation or reproduction guide. Dependency versions, complete configurations, dataset preparation scripts, and checkpoints are not available in this repository.

## Code and weights availability

Only this README is currently provided. No training code, inference code, model weights, or executable examples are included. A release date has not been specified.

## Authors and publication information

The supplied manuscript lists **Anonymous Author**, with placeholder affiliation and email fields. Author identities, affiliations, and contact details are therefore not asserted here.

The manuscript title is **DeBiT-Det: Decoding-oriented Bi-Branch Transformer for Remote Sensing Rotated Object Detection**. An accepted publication venue, publication date, DOI, and public paper URL have not been verified. This repository does not claim acceptance or publication in TNNLS or any other venue.

## Citation

Verified bibliographic metadata is not yet available. Please use the title above when referring to this work, and consult the authors' public paper record when it becomes available. No DOI or formal BibTeX record is supplied here.

## Contact

For questions about this repository, please [open an issue](https://github.com/zga1999/DeBiT-Det/issues).
