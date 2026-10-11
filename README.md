# Network Intrusion Detection System (NIDS) with Random Forest

A multi-class network intrusion detection system built with a Random Forest classifier on the **CICIDS2017** dataset. The model classifies network flows into four categories — **Normal**, **DDoS**, **PortScan**, and **BruteForce** — and achieves a weighted F1-score of **99.99%** and a macro F1-score of **99.98%** on held-out test data.

> Final-year capstone project — B.Sc. in Computer Engineering
> Kermanshah University of Technology, Faculty of Information Technology
> Author: Mohadese Bimnak · Supervisor: Dr. Mehdi Khazaei

---

## Table of Contents

- [Overview](#overview)
- [Dataset](#dataset)
- [Methodology](#methodology)
- [Results](#results)
- [Feature Importance](#feature-importance)
- [Repository Structure](#repository-structure)
- [Getting Started](#getting-started)
- [Limitations](#limitations)
- [Future Work](#future-work)
- [Reference](#reference)
- [License](#license)

---

## Overview

Traditional signature-based intrusion detection systems struggle against novel (zero-day) attacks and the sheer volume of modern network traffic. This project applies supervised machine learning instead: a Random Forest classifier is trained on statistical flow-level features to distinguish normal traffic from three common attack types:

| Class | Description |
|---|---|
| `Normal` | Benign network traffic |
| `DDoS` | Distributed Denial of Service |
| `PortScan` | Port scanning / reconnaissance |
| `BruteForce` | FTP/SSH password brute-forcing (FTP-Patator, SSH-Patator) |

The project is inspired by the methodology in *"Enhancing Network Threat Detection with Random Forest-Based NIDS and Permutation Feature Importance"* (Abdelaziz et al., 2025), simplified to a scope appropriate for an undergraduate capstone project (see [Limitations](#limitations) for what was intentionally left out).

## Dataset

[**CICIDS2017**](https://www.unb.ca/cic/datasets/ids-2017.html), created by the Canadian Institute for Cybersecurity, University of New Brunswick. Each record is a network flow described by 78 statistical features (packet lengths, inter-arrival times, TCP flags, byte/packet rates, etc.).

Four of the dataset's CSV files are used:

| File | Content | Records |
|---|---|---:|
| `Monday-WorkingHours_pcap_ISCX.csv` | Benign traffic only | 529,918 |
| `Tuesday-WorkingHours_pcap_ISCX.csv` | Benign + FTP-Patator, SSH-Patator | 445,909 |
| `Friday-WorkingHours-Afternoon-PortScan_pcap_ISCX.csv` | Benign + PortScan | 286,467 |
| `Friday-WorkingHours-Afternoon-DDos_pcap_ISCX.csv` | Benign + DDoS | 225,745 |
| **Total** | | **1,488,039** |

> The raw CSV files are **not included** in this repository due to their size. Download them from the [official CICIDS2017 page](https://www.unb.ca/cic/datasets/ids-2017.html) and place them in the project root before running the notebook.

## Methodology

1. **Load & combine** the four CSV files into a single DataFrame.
2. **Clean the data** — remove duplicate rows, infinite values, and missing values.
3. **Map labels** to four target classes (`BENIGN → Normal`, `FTP-Patator`/`SSH-Patator → BruteForce`, etc.).
4. **Handle class imbalance** — undersample the majority `Normal` class to 150,000 flows; keep all attack samples.
5. **Split features/target**, then an 80/20 stratified train/test split.
6. **Train** a `RandomForestClassifier` (100 trees, `class_weight="balanced"`, `random_state=42`).
7. **Evaluate** with per-class Precision/Recall/F1, Macro F1, Weighted F1, and a confusion matrix.
8. **Analyze feature importance** (Gini importance) and retrain on the top-20 features for comparison.

## Results

### Full model (78 features)

| Class | Precision | Recall | F1-score | Support |
|---|---:|---:|---:|---:|
| BruteForce | 1.0000 | 0.9989 | 0.9995 | 1,830 |
| DDoS | 1.0000 | 1.0000 | 1.0000 | 25,603 |
| Normal | 0.9998 | 1.0000 | 0.9999 | 30,000 |
| PortScan | 1.0000 | 0.9998 | 0.9999 | 18,139 |

**Accuracy:** 99.99% &nbsp;|&nbsp; **Macro F1:** 99.98% &nbsp;|&nbsp; **Weighted F1:** 99.99%

### Reduced model (top-20 features)

| Model | Macro F1 | Weighted F1 |
|---|---:|---:|
| Random Forest (78 features) | 0.9998 | 0.9999 |
| Random Forest (20 features) | 0.9999 | 0.9999 |

A model trained on only 20 of the 78 original features performs on par with the full model — useful for lighter, faster deployments.

## Feature Importance

The most influential feature by a wide margin is **`Destination Port`**, consistent with the finding in the reference paper. Other high-ranking features relate to forward/backward packet length statistics and TCP initial window size.

## Repository Structure

```
.
├── NIDS_RandomForest_Project.ipynb                        # Main notebook: data pipeline, training, evaluation
├── README.md
├── requirements.txt
├── Monday-WorkingHours_pcap_ISCX.csv                      # Not tracked in git — download separately
├── Tuesday-WorkingHours_pcap_ISCX.csv                     # Not tracked in git — download separately
├── Friday-WorkingHours-Afternoon-PortScan_pcap_ISCX.csv   # Not tracked in git — download separately
└── Friday-WorkingHours-Afternoon-DDos_pcap_ISCX.csv       # Not tracked in git — download separately
```

## Getting Started

```bash
# 1. Clone the repository
git clone https://github.com/<your-username>/<repo-name>.git
cd <repo-name>

# 2. Install dependencies
pip install -r requirements.txt

# 3. Download the 4 CICIDS2017 CSV files listed above and place them
#    directly next to the notebook (same folder, no subfolder needed)

# 4. Launch the notebook
jupyter notebook NIDS_RandomForest_Project.ipynb
```

**`requirements.txt`**
```
pandas
numpy
matplotlib
seaborn
scikit-learn
```

## Limitations

- Only 3 attack types are covered (of the 8+ available in the full CICIDS2017 dataset); Web Attack, Bot, Infiltration, and Heartbleed are not included.
- Evaluated only on this dataset — not tested on live network traffic or other intrusion-detection datasets.
- Class imbalance is handled with simple random undersampling rather than more advanced techniques (e.g., SMOTE).
- Feature selection uses Random Forest's built-in Gini importance rather than the more rigorous (and more expensive) Permutation Importance used in the reference paper.
- As with most CICIDS2017 studies using a random train/test split, near-duplicate flows from the same attack session may appear on both sides of the split, which can make reported metrics somewhat optimistic compared to a live deployment.

## Future Work

- Extend to the full set of CICIDS2017 attack classes.
- Use Permutation Importance for more rigorous feature selection.
- Compare against other algorithms (Gradient Boosting, neural networks).
- Evaluate in a near real-time / streaming setting.

## Reference

Abdelaziz, M.T., Radwan, A., Mamdouh, H., Saad, A.S., Abuzaid, A.S., AbdElhakeem, A.A., Zakzouk, S., Moussa, K., Darweesh, M.S. (2025). *Enhancing Network Threat Detection with Random Forest-Based NIDS and Permutation Feature Importance.* Journal of Network and Systems Management, 33(2). https://doi.org/10.1007/s10922-024-09874-0

Dataset: Canadian Institute for Cybersecurity, [CICIDS2017](https://www.unb.ca/cic/datasets/ids-2017.html)

## License

This project is released under the [MIT License](LICENSE).
