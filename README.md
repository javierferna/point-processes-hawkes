# Patient Subgroup Analysis with Clustering and Hawkes Processes

## Project Overview

This project analyzes an anonymized clinical study dataset to identify distinct patient subgroups and model their event patterns over time. The methodology involves two steps:

1.  **Clustering:** Patients are grouped into distinct clusters based on anonymized features. 
2.  **Point Process Modeling:** An Exponential Hawkes process is fitted to the event timelines of all patients *within each cluster* to model the temporal dynamics of their clinical events.

The ultimate goal is to **compare the fitted Hawkes process parameters ($\mu$, $\eta$, $\theta$) across clusters**. This allows us to understand how event risk and self-excitation patterns differ between patient subgroups.

---

## Dataset

The analysis relies on two anonymized files:

* `features.csv`: Contains static patient features (`F0`, ..., `F7`) used for clustering.
* `timelines.csv`: Contains event timelines (`time_1`, ..., `time_11`). Times are encoded in months as the **relative difference** from the previous event.

---

## Methodology

### 1. Data Preprocessing

* **Timeline Conversion:** The relative event times from `timelines.csv` are converted into **absolute timestamps** by calculating the cumulative sum for each patient (e.g., `event_3_time = time_1 + time_2 + time_3`).
* **Observation End Time:** The end of the observation period for each patient is defined as the time of their last recorded event plus an observation buffer (i.e., +1 month).

### 2. Patient Clustering

* Patients are clustered using their static features from `features.csv` using K-Means.
* Each patient is assigned a `cluster_id`, which links their static features to their event timeline.

### 3. Hawkes Process Modeling

* An **Exponential Hawkes Process** is used to model the event intensity $\lambda(t)$ for each cluster.
* **Data Preparation for Modeling:** To fit a single model per cluster, all patient timelines within that cluster are concatenated into one "super-timeline". An offset of 12 months is added between each patient's timeline to prevent their events from exciting the start of the next patient's timeline. 
* **Parameter Estimation:** The `HawkesPyLib` library is used to estimate the parameters for each cluster's aggregated timeline:
    * **$\mu$ (mu):** The **baseline intensity**. This represents the underlying, constant risk of an event occurring, independent of past events.
    * **$\alpha$ (alpha):** The **jump intensity**. It's the immediate increase in event risk right after another event occurs.
    * **$\beta$ (beta):** The **decay rate**. This controls how quickly the risk from a previous event fades away. A high $\beta$ means the influence is short, while a low $\beta$ means it remains.
