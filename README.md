# Multi-Future Pedestrian Trajectory Forecasting with Intent–Trajectory Consistency

[![Course](https://img.shields.io/badge/Course-CS766%20Computer%20Vision-blue)](#)
[![Project](https://img.shields.io/badge/Project-Final%20Project-green)](#)
[![Status](https://img.shields.io/badge/Status-Prototype-orange)](#)

**Authors:** Guyang Cao, Yujie Ding  
**Institution:** University of Wisconsin–Madison  
**Course:** CS766: Computer Vision  
**Project Webpage:** [https://jenyud.github.io/cs766-final-project/](https://jenyud.github.io/cs766-final-project/)

---

## Overview

Pedestrian future motion is inherently uncertain. Given the same observed motion history, a pedestrian may continue walking, slow down, stop, or start crossing. This one-to-many nature makes deterministic single-trajectory prediction limited, because the model may predict an averaged future that does not represent any realistic behavior.

In this project, we study **multi-future pedestrian trajectory forecasting** on the PIE dataset. Instead of predicting only one future trajectory, our prototype predicts a set of `K` possible future trajectories and evaluates whether at least one of them matches the realized future.

Our final prototype focuses on the multi-future trajectory forecasting component. The original project direction also included intent–trajectory consistency, which remains an important future direction.

---

## Project Goal

The goal of this project is to compare deterministic pedestrian trajectory prediction with multi-hypothesis trajectory prediction.

Given a pedestrian's recent motion history in a driving video, we aim to predict multiple plausible future trajectories.

**Input:**

```text
Past 15 pedestrian center points
