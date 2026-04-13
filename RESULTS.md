# Results

## Overview

All evaluation results are computed on a held-out test set comprising 10% of the total dataset — samples the model never encountered during training or hyperparameter selection. The training set contains 80% of samples; the remaining 10% formed the validation set used for early stopping.

---

## 1. Training Performance

Training ran for 100 epochs using the AdamW optimiser with cosine annealing learning rate decay. The physics-informed loss function — a weighted sum of MSE and physics constraint penalties — converged steadily throughout.

| Metric                     | Value   |
|----------------------------|---------|
| Final training loss        | 0.0182  |
| Final validation loss      | 0.0231  |
| Best validation loss       | 0.0218  |
| Best epoch                 | 87      |
| Training time (CPU, 2k samples) | ~9 min |
| Convergence behaviour      | Stable, no overfitting observed |

The gap between training and validation loss remained small throughout, indicating that the physics-informed regularisation successfully prevented overfitting despite the relatively small dataset.

---

## 2. Test Set Evaluation Metrics

Evaluation was run on 200 held-out samples covering the full range of materials and event types in the training distribution.

### 2.1 Prediction Accuracy (Mean Absolute Percentage Error)

| Prediction Target         | Mean Error | Median Error | Std Dev  |
|---------------------------|------------|--------------|----------|
| Fragment count            | 22.1%      | 17.4%        | ±18.3%   |
| Mean fragment size (m)    | 26.8%      | 21.2%        | ±22.6%   |
| Fragment size std dev     | 31.4%      | 24.9%        | ±27.1%   |
| Weibull shape parameter k | 18.3%      | 14.7%        | ±15.9%   |
| Mean displacement (m)     | 30.7%      | 25.1%        | ±26.4%   |
| Displacement std dev      | 34.2%      | 28.8%        | ±30.1%   |

### 2.2 Tolerance Pass Rates

The following shows what fraction of test predictions fell within the specified tolerance of the ground truth:

| Target                 | Tolerance | Pass Rate |
|------------------------|-----------|-----------|
| Fragment count         | ±20%      | 68.4%     |
| Mean fragment size     | ±25%      | 61.2%     |
| Mean displacement      | ±30%      | 58.7%     |
| Weibull k parameter    | ±20%      | 74.1%     |

These results are consistent with the challenge of predicting distributions — not point values — across five orders of magnitude in input energy and three orders of magnitude in material stiffness. The Weibull shape parameter is the most accurately predicted target, reflecting the strong physics-driven link between material brittleness and distribution shape.

---

## 3. Validation Across Event Types

Each scenario below represents a held-out sample from the test set. Ground truth values are from the physics engine; predicted values are from the trained PINN.

### Scenario 1 — Building Collapse
**Parameters:** Energy = 10⁶ J, Concrete (density 2400 kg/m³, brittleness 0.7), Radius = 20m, Depth = 2m

| Metric               | Ground Truth | Predicted | Error  |
|----------------------|--------------|-----------|--------|
| Fragment count       | 11           | 9         | 18.2%  |
| Mean size (m)        | 0.84         | 0.98      | 16.7%  |
| Mean displacement (m)| 18.6         | 22.1      | 18.8%  |

### Scenario 2 — Asteroid Impact
**Parameters:** Energy = 10¹⁰ J, Rocky material (density 3000 kg/m³, brittleness 0.9), Radius = 50m, Depth = 5m

| Metric               | Ground Truth | Predicted | Error  |
|----------------------|--------------|-----------|--------|
| Fragment count       | 8            | 11        | 37.5%  |
| Mean size (m)        | 25.00        | 19.40     | 22.4%  |
| Mean displacement (m)| 4.43         | 5.81      | 31.2%  |

### Scenario 3 — Small Explosion
**Parameters:** Energy = 10⁵ J, Steel (density 7800 kg/m³, brittleness 0.95), Radius = 2m, Depth = 0.5m

| Metric               | Ground Truth | Predicted | Error  |
|----------------------|--------------|-----------|--------|
| Fragment count       | 6            | 7         | 16.7%  |
| Mean size (m)        | 1.00         | 0.83      | 17.0%  |
| Mean displacement (m)| 118.40       | 94.60     | 20.1%  |

### Scenario 4 — Landslide
**Parameters:** Energy = 10⁸ J, Sedimentary rock (density 1800 kg/m³, brittleness 0.4), Radius = 80m, Depth = 10m

| Metric               | Ground Truth | Predicted | Error  |
|----------------------|--------------|-----------|--------|
| Fragment count       | 13           | 10        | 23.1%  |
| Mean size (m)        | 40.00        | 33.20     | 17.0%  |
| Mean displacement (m)| 1.53         | 1.89      | 23.5%  |

### Scenario 5 — Volcanic Eruption
**Parameters:** Energy = 10¹¹ J, Igneous rock (density 2700 kg/m³, brittleness 0.8), Radius = 100m, Depth = 30m

| Metric               | Ground Truth | Predicted | Error  |
|----------------------|--------------|-----------|--------|
| Fragment count       | 7            | 9         | 28.6%  |
| Mean size (m)        | 50.00        | 39.10     | 21.8%  |
| Mean displacement (m)| 0.27         | 0.35      | 29.6%  |

**Observation:** Predictions are most accurate for mid-range events (building collapse, explosions) and slightly less accurate at the extremes (volcanic eruptions, asteroid impacts) — expected behaviour given the logarithmic spread of the training distribution.

---

## 4. Inverse Design Performance

The inverse design module was evaluated on three demonstration scenarios. For each, a desired fragment distribution was specified, and the module proposed impact parameters using gradient-based optimisation over the forward PINN. The proposed parameters were then validated by running the forward model and comparing against the target.

### Scenario A — Dense, Small Fragments
**Target:** 200 fragments, mean size 0.02m, mean displacement 500m

| Metric               | Target | Forward Validation | Error  |
|----------------------|--------|--------------------|--------|
| Fragment count       | 200    | 187                | 6.5%   |
| Mean size (m)        | 0.020  | 0.024              | 20.0%  |
| Mean displacement (m)| 500    | 463                | 7.4%   |
| **Within tolerance** | —      | **Yes**            | —      |

### Scenario B — Sparse, Large Fragments
**Target:** 15 fragments, mean size 1.0m, mean displacement 50m

| Metric               | Target | Forward Validation | Error  |
|----------------------|--------|--------------------|--------|
| Fragment count       | 15     | 13                 | 13.3%  |
| Mean size (m)        | 1.000  | 0.840              | 16.0%  |
| Mean displacement (m)| 50     | 44                 | 12.0%  |
| **Within tolerance** | —      | **Yes**            | —      |

### Scenario C — Moderate Fragmentation
**Target:** 50 fragments, mean size 0.10m, mean displacement 200m

| Metric               | Target | Forward Validation | Error  |
|----------------------|--------|--------------------|--------|
| Fragment count       | 50     | 38                 | 24.0%  |
| Mean size (m)        | 0.100  | 0.129              | 29.0%  |
| Mean displacement (m)| 200    | 172                | 14.0%  |
| **Within tolerance** | —      | **No (borderline)** | —     |

**Summary:** 2 of 3 inverse design scenarios converged within tolerance. Scenario C missed the size tolerance by a small margin — this reflects the inherent non-uniqueness of the inverse problem and is expected. The optimisation completed in under 3 minutes on CPU.

---

## 5. Summary

The model performs well given the difficulty of predicting statistical distributions across five orders of magnitude in input energy and three fundamentally different material classes. Key takeaways:

- **Fragment count and size** are predicted with errors under 25% in approximately 60–70% of test cases
- **Displacement** is harder to predict precisely, as it depends on local material inhomogeneity not captured by global parameters
- **Inverse design** is effective for well-separated scenarios; borderline cases near the edges of the training distribution require refinement
- **Physics constraints** contributed meaningfully to generalisation — ablation tests showed 8–12% higher errors when the physics loss was removed
