# 🎣 Explainable Phishing URL Detection

**Can a model spot a phishing link from the URL alone, and tell you *why* it flagged it?**

This project trains and compares classical ML, ensemble and deep-learning models on **579,920 URLs** (339,074 legitimate, 240,846 phishing), then opens the black box with SHAP, LIME and permutation importance. It also includes a small dashboard that scores any URL you paste in.

`UML501 · Machine Learning` · Course project by **Pehal** and **Yashjot Kaur**, built on a published base framework (see [Based on](#-based-on-and-what-we-added)) &nbsp;|&nbsp; Python · scikit-learn · XGBoost · LightGBM · CatBoost · TensorFlow · SHAP · LIME · Streamlit

---

## ✨ Highlights

- **98.85% accuracy / 0.986 F1** with a stacking ensemble of six models, on a held-out test set of 115,984 URLs
- **74 engineered features** per URL: 53 lexical, 9 DNS, 12 WHOIS
- **Three deep-learning models** that read the raw URL character by character (CNN, stacked BiLSTM, CNN-BiLSTM), each averaged over 3 seeds
- **Explainability built in:** SHAP and permutation importance agree strongly (Spearman 0.916), and 8 of the top-10 features overlap
- **Live demo:** paste a URL and see a risk score with the reasons behind it

## 📊 Results

_These results come from the base framework's notebooks and the 579,920-URL dataset. We use them as the benchmark that our additions are compared against._

**Classical ML and ensembles** (80/20 stratified split, 115,984 test URLs)

| Model | Accuracy | Precision | Recall | F1 | ROC-AUC | PR-AUC |
|---|---|---|---|---|---|---|
| Logistic Regression | 0.9440 | 0.9439 | 0.9198 | 0.9317 | 0.9845 | 0.9804 |
| Decision Tree | 0.9749 | 0.9721 | 0.9673 | 0.9697 | 0.9738 | 0.9539 |
| Random Forest | 0.9851 | 0.9905 | 0.9736 | 0.9820 | 0.9983 | 0.9979 |
| XGBoost | 0.9816 | 0.9844 | 0.9711 | 0.9777 | 0.9976 | 0.9972 |
| CatBoost | 0.9832 | 0.9863 | 0.9730 | 0.9796 | 0.9979 | 0.9976 |
| LightGBM (best single model) | 0.9868 | 0.9898 | 0.9783 | 0.9840 | 0.9985 | 0.9982 |
| Soft Voting (all 6 models) | 0.9851 | 0.9888 | 0.9752 | 0.9820 | 0.9981 | 0.9978 |
| **Stacking (all 6 models)** | **0.9885** | **0.9906** | **0.9816** | **0.9861** | **0.9985** | **0.9982** |

**Deep learning on raw URL text** (70/15/15 split, 86,988 test URLs, 3-seed ensembles)

| Model | Accuracy | Precision | Recall | F1 | ROC-AUC | PR-AUC |
|---|---|---|---|---|---|---|
| Character CNN | 0.9838 | 0.9862 | 0.9746 | 0.9803 | 0.9985 | 0.9981 |
| Stacked Character BiLSTM | 0.9800 | 0.9818 | 0.9700 | 0.9758 | 0.9976 | 0.9971 |
| CNN-BiLSTM | 0.9841 | 0.9868 | 0.9746 | 0.9807 | 0.9986 | 0.9982 |

Full tables, including training and prediction times, are in [`results/`](results/).

> ⚠️ **Read the numbers fairly.** The ML and deep-learning tables use different train/test splits, so they are not a strict head-to-head. Also see [Limitations](#-limitations).

## 🧱 Base project and what we added

Built on the base framework by Ozcelik, ElSilk and Osmanoglu (Ankara University), *Explainable Machine Learning Framework for Phishing URL Detection with a Realistic Large-Scale Dataset*. The research notebooks, the 579,920-URL dataset and the published experiments come from that work. Our additions (details in [`docs/BEFORE_AFTER_CHANGES.md`](docs/BEFORE_AFTER_CHANGES.md)):

| Addition | Where |
|---|---|
| Real-time extractor for all 74 features of any raw URL | `src/features/extractor.py` |
| Leak-free preprocessing pipeline | `src/features/preprocess.py` |
| Training and one-line inference with risk score and reasons | `src/models/train_and_predict.py` |
| 60-URL sample dataset | `data/sample_dataset.csv` |
| Streamlit and standalone web dashboards | `app.py`, `app_web.py` |
| Repo restructuring, configs, README, viva/user guide | whole repo, `docs/` |

## 🧭 How it works

```
 URLs ──► Feature extraction ──► Clean + split ──► Models ──► Explain
 (Kaggle   53 lexical            dedupe URLs      6 ML models   SHAP
  dataset) 9 DNS                 stratified       6 ensembles   LIME
           12 WHOIS              split first,     3 deep nets   permutation
                                 then impute &                  importance
                                 scale (train
                                 only → no leakage)
```

1. **Data:** URLs and 74 precomputed features from a public Kaggle dataset ([phishing-url-features-dataset](https://www.kaggle.com/datasets/elifzelik/phishing-url-features-dataset)). Notebooks 01–03 document how the legitimate and phishing URL sets were collected, cleaned and merged.
2. **Preprocessing:** drop duplicate URLs, split **first**, then fit the median imputer and scaler on the training set only. A `whois_missing` flag keeps track of URLs with no WHOIS record.
3. **Models:** Logistic Regression, Decision Tree, Random Forest, XGBoost, LightGBM, CatBoost, then hard/soft voting and stacking.
4. **Deep learning:** character-level CNN, stacked BiLSTM and CNN-BiLSTM trained with early stopping on a validation set, with class weights for the 58/42 imbalance.
5. **Explainability:** global importance from SHAP and permutation importance, and per-URL explanations from LIME.

## 🔍 What explainability showed

| Check | Result |
|---|---|
| SHAP vs permutation importance (Spearman correlation) | 0.916 |
| Top-10 feature overlap | 8 / 10 |
| Top-20 feature overlap | 17 / 20 |
| SHAP stability after 500 samples | 0.997 – 0.998 |

Two independent methods ranking the features almost the same way is good evidence that the model relies on real signals, not noise.

## 🚀 Quick start

```bash
git clone https://github.com/pehal6/phishing-url-detection-uml501.git
cd phishing-url-detection-uml501
pip install -r requirements.txt

# Train the demo model on the small bundled sample and test one URL
python -m src.models.train_and_predict

# Launch the dashboard (pick one)
streamlit run app.py        # Streamlit UI
python app_web.py           # standalone web UI at http://localhost:8080
```

> The bundled demo model is trained on a **60-row sample** (`data/sample_dataset.csv`), so it is for showing the dashboard only. The results above come from the full 579,920-URL dataset in the notebooks.

To reproduce the full experiments, download the dataset from Kaggle, place it as `data/processed/final_dataset.csv`, update the path cells in the notebooks (some still point to Kaggle/Windows paths), then run notebooks `01` to `08` in order. See [`docs/reproducibility.md`](docs/reproducibility.md).

## 🗂️ Repository structure

```
notebooks/   01-08: data collection → dataset prep → ML/ensembles → XAI → deep learning
src/
  features/  extractor.py (URL/DNS/WHOIS features), preprocess.py (leak-free preprocessing)
  models/    train_and_predict.py, metrics.py, threshold_search.py
  data/      resolve_domains.py, crawl_internal_urls.py
scripts/     build_sample_dataset.py, validate_repo.py
configs/     YAML settings for data collection, ML models and deep learning
data/        sample dataset and processed train/test files
results/     metrics (CSV/JSON) and the XAI summary
artifacts/   saved preprocessor and demo model
docs/        project report, conference paper, viva guide, reproducibility notes
archive/     earlier notebooks and scripts kept for reference
app.py, app_web.py   dashboards
```

## ⚠️ Limitations

- **Split by URL, not by domain.** URLs from the same domain can appear in both train and test, so scores may be slightly optimistic.
- **Different test sets.** ML models use an 80/20 split; deep-learning models use 70/15/15. A fixed shared test set is the next step.
- **Test-set selection.** The "Top-3" ensembles were chosen by F1 on the test set, so we treat them as experimental and use the all-model stacking ensemble as the headline result.
- **Dataset shift.** Phishing URLs change quickly. Performance on newly created campaigns may be lower.

## 🛣️ Roadmap

- [ ] Evaluate every model on one fixed shared test set
- [ ] Domain-grouped split to measure how well the models generalise to unseen domains
- [ ] Choose model variants and thresholds on validation data only
- [ ] Connect the dashboard to the full-data model
- [ ] Add LIME explanations to the dashboard

## 👥 Team

| Name | Roll no. |
|---|---|
| Pehal | 1024170103 |
| Yashjot Kaur | 1024170452 |

## 🙏 Acknowledgements and license

Base framework and dataset: Ozcelik, ElSilk and Osmanoglu (Ankara University), also available on Kaggle as [phishing-url-features-dataset](https://www.kaggle.com/datasets/elifzelik/phishing-url-features-dataset). Citation details are in [`CITATION.cff`](CITATION.cff). Released under the MIT License (see [`LICENSE`](LICENSE)).
