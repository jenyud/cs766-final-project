# Multi-Future Pedestrian Trajectory Forecasting with Intent–Trajectory Consistency

**Authors:** Guyang Cao, Yujie Ding  
**Institution:** University of Wisconsin–Madison  
**Course:** CS766: Computer Vision  
**Project Webpage:** https://jenyud.github.io/cs766-final-project/

---

## Overview

Pedestrian future motion is inherently uncertain. Given the same observed motion history, a pedestrian may continue walking, slow down, stop, or start crossing. This one-to-many nature makes deterministic single-trajectory prediction limited because the model may predict an averaged future that does not represent any realistic behavior.

In this project, we study **multi-future pedestrian trajectory forecasting** on the PIE dataset. Instead of predicting only one future trajectory, our prototype predicts a set of `K` possible future trajectories and evaluates whether at least one of them matches the realized future.

Our final prototype focuses on the multi-future trajectory forecasting component. The original project direction also included intent–trajectory consistency, which remains an important future direction.

---

## Project Goal

The goal of this project is to compare deterministic pedestrian trajectory prediction with multi-hypothesis trajectory prediction.

Given a pedestrian's recent motion history in a driving video, we aim to predict multiple plausible future trajectories.

**Input:** past 15 pedestrian center points  
**Output:** future 45 pedestrian center points  

**Main question:**  
Can predicting multiple possible futures better cover the realized pedestrian motion than predicting only one deterministic future?

---

## Motivation

Pedestrian forecasting is important for safe autonomous driving, especially near crosswalks, intersections, and occlusions.

If the prediction is too late, a vehicle may brake too late and create unsafe situations. If the prediction is too conservative, the vehicle may stop unnecessarily and reduce traffic efficiency.

A single predicted trajectory is often not enough in ambiguous scenes. Multi-future prediction allows the model to cover several possible futures and better represent uncertainty in pedestrian behavior.

---

## Method

### Data Processing

We use pedestrian annotations from the PIE dataset and process them into center-point trajectories.

The main steps are:

1. Parse PIE annotation XML files.
2. Convert pedestrian bounding boxes into 2D center points.
3. Keep contiguous pedestrian tracks with sufficient length.
4. Construct sliding windows: 15 observed frames → 45 future frames.

---

### Dataset Split

For the current controlled prototype, we use PIE `set03`.

| Split | Videos |
|---|---|
| Training | 0001–0014 |
| Validation | 0015–0017 |
| Testing | 0018–0019 |

This restricted setting allows us to first verify the full pipeline and isolate the effect of multi-future trajectory prediction.

---

### Model Architecture

Our model is a lightweight PyTorch trajectory prediction prototype.

The pipeline is:

**Past center trajectory → GRU Encoder → MLP Prediction Head → K future trajectories**

When `K = 1`, the model becomes a deterministic single-trajectory baseline.

When `K > 1`, the model outputs a set of possible future trajectories:

**Ŷ = {ŷ(1), ŷ(2), ..., ŷ(K)}**

Each predicted trajectory is a sequence of 45 future 2D center points.

---

### Best-of-K Training

We train the multi-future model using a Best-of-K loss. Instead of forcing every predicted trajectory to match the ground truth, this objective encourages at least one predicted future to be close to the realized trajectory.

**L_BoK = min_k (1 / T) Σ_t || ŷ_t^(k) - y_t ||²**

This training strategy is suitable for one-to-many prediction because it rewards coverage of the realized future rather than only optimizing the first/default output.

---

## Evaluation Metrics

We evaluate both single-mode prediction quality and multi-future set coverage.

| Metric | Meaning |
|---|---|
| `ADE@1` | Average displacement error of the first/default predicted trajectory |
| `FDE@1` | Final displacement error of the first/default predicted trajectory |
| `minADE@K` | Minimum ADE among K predicted future trajectories |
| `minFDE@K` | Minimum FDE among K predicted future trajectories |

`ADE@1` and `FDE@1` evaluate the quality of the first trajectory.  
`minADE@K` and `minFDE@K` evaluate whether the predicted set contains at least one trajectory close to the ground truth.

---

## Quantitative Results

The main comparison is between the deterministic baseline and the multi-future model.

| Model | K | Epoch | ADE@1 | FDE@1 | minADE@K | minFDE@K |
|---|---:|---:|---:|---:|---:|---:|
| Baseline | 1 | 30 | 13.27 | 34.68 | 13.27 | 34.68 |
| Multi-future | 5 | 30 | 15.11 | 39.48 | 7.17 | 16.30 |
| Multi-future | 10 | 20 | 60.32 | 202.90 | 6.40 | 13.05 |

### Key Finding

Increasing `K` from 1 to 5 substantially improves set-based metrics:

- `minADE`: 13.27 → 7.17
- `minFDE`: 34.68 → 16.30

This shows that multi-future prediction improves the coverage of possible pedestrian futures.

However, `ADE@1` and `FDE@1` do not necessarily improve. This is expected because Best-of-K training optimizes whether some predicted future matches the ground truth, not whether the first/default future is always the best.

---

## Qualitative Results

We visualize predicted future sets in several representative cases:

1. **Success case:** the predicted set covers the ground-truth trajectory well.
2. **Ambiguous case:** multiple plausible futures are generated.
3. **Failure case:** the predicted set still misses the actual future.

The qualitative results show that multi-future prediction can cover different possible motion patterns. However, without mixture weights or an intention head, the model does not yet tell us which predicted future should be trusted most.

---

## What Worked

- A simple GRU-based model can generate multiple plausible future trajectories.
- Best-of-K training improves `minADE` and `minFDE`.
- Multi-future prediction is better suited than deterministic regression for one-to-many pedestrian motion.
- Set-based metrics reveal useful information that single-trajectory metrics may miss.

---

## Limitations

The current system is still a prototype and has several limitations:

- Experiments are restricted to PIE `set03` with a custom split.
- The model uses only pedestrian bounding-box center coordinates.
- Image appearance, road geometry, scene context, and ego-vehicle speed are not yet included.
- The first/default predicted trajectory may become worse even when `minADE` improves.
- The model does not learn probability rankings over the predicted futures.
- The intent–trajectory consistency regularizer is not yet implemented in the final prototype.

---

## Future Work

Future work will focus on making the predicted futures more semantically meaningful and better ranked.

Planned extensions include:

1. Add mixture weights for different trajectory hypotheses:  
   **{ŷ(k)} → {(π_k, ŷ(k))}**

2. Add an intention head:  
   **p_cross = Pr(pedestrian will cross)**

3. Train an intent–trajectory consistency regularizer that aligns the predicted crossing probability with the probability mass assigned to crossing-like trajectory modes.

4. Incorporate richer inputs, including pedestrian appearance, scene context, road/crosswalk geometry, and ego-vehicle speed.

---

## Repository Structure

```text
cs766-final-project/
│
├── index.html                  # Project webpage
├── README.md                   # Repository overview
├── .nojekyll                   # GitHub Pages configuration
│
└── static/
    ├── images/                 # Figures and qualitative visualizations
    ├── videos/                 # Project presentation video
    ├── pdfs/                   # Slides, report, and proposal
    ├── css/                    # Template stylesheets
    └── js/                     # Template scripts
```

---

## Project Materials

- [Project Webpage](https://jenyud.github.io/cs766-final-project/)
- [Final Presentation](static/pdfs/final_presentation.pdf)
- [Midterm Report](static/pdfs/midterm_report.pdf)
- [Project Proposal](static/pdfs/proposal.pdf)

---

## References

1. Amir Rasouli, Iuliia Kotseruba, Toni Kunic, and John K. Tsotsos.  
   *PIE: A Large-Scale Dataset and Models for Pedestrian Intention Estimation and Trajectory Prediction.*  
   ICCV, 2019.

2. PIEPredict Repository:  
   https://github.com/aras62/PIEPredict

---

## Acknowledgement

This project was completed as the final project for **CS766: Computer Vision** at the University of Wisconsin–Madison.

The project webpage was built using the [Academic Project Page Template](https://github.com/eliahuhorwitz/Academic-project-page-template), which was adapted from the Nerfies project page.
