# Remixing-Metaheuristic

**Remix Metaheuristic Feature Selection (FS) + Hyperparameter Optimization (HPO) Imbalance-Aware untuk Prediksi Penyakit Kardiovaskular (CVD) — V2.1**

Tugas Final Project (FP) mata kuliah **Kecerdasan Komputasional A** — **Kelompok KK A 02**:
- Burhanudin Yusuf Abdullah Ar Ramadhan (`6025261048`)
- Renaldy Junedy (`6025261035`)
- Adnan Fauzan Ramdhani (`6025261004`)
- Muhammad Saifullah Irfanuddin (`6025261056`)

---

## Ringkasan Proyek

Repositori ini berisi notebook eksperimen utama, cache dataset, hasil eksperimen (CSV), analisis statistik non-parametrik, artefak Explainable AI (SHAP & LIME), visualisasi figur, serta laporan hasil untuk studi **kombinasi (*remixing*) optimizer metaheuristik FS dan HPO yang sadar ketidakseimbangan kelas (*imbalance-aware*)** pada domain prediksi penyakit kardiovaskular (CVD).

Tiga tujuan utama yang dievaluasi dalam proyek ini:
1. **Tujuan 01 — Penanganan Imbalance Bebas Kebocoran (*Leakage-Free*)**: Membandingkan metode penanganan *imbalance* dengan *resampling* hanya di dalam *fold* CV (*in-fold only*) serta mengaudit inflasi metrik palsu akibat SMOTE sebelum pemisahan data (*pre-split leakage*).
2. **Tujuan 02 — Optimalisasi Recall pada FS Metaheuristik**: Mengevaluasi sensitivitas fungsi *fitness*, urutan *resampling* (`FS→SMOTE` vs `SMOTE→FS`), fungsi transfer biner, dan 30 pasangan *remixing* (5 optimizer FS × 6 klasifier).
3. **Tujuan 03 — Integrasi HPO + FS yang Adil dan Efisien**: Membandingkan skema *HPO-only*, *Joint FS+HPO*, dan *Tandem*, menguji stabilitas *multi-seed*, memvalidasi pada skala penuh (Cardio 70k & Framingham), serta menguji signifikansi statistik dan interpretabilitas model (SHAP & LIME).

---

## Protokol Eksperimen

- **Fungsi Fitness Utama**:
  $$F_{AP+BA} = 0.55(1 - AP) + 0.40(1 - BA) + 0.05\left(\frac{|m|}{p}\right)$$
  dengan *Average Precision* ($AP$), *Balanced Accuracy* ($BA$), jumlah fitur terpilih ($|m|$), dan total fitur ($p$).
- **Validasi Silang & Split**: Pembagian data **70% train / 15% validation / 15% test** dengan *preprocessing* di-*fit* hanya pada data latih; seleksi model menggunakan **inner 3-fold CV** (*resampling in-fold only*).
- **Anggaran Evaluasi (*Hard Budget*)**: `80` evaluasi untuk tahap FS murni, `160` evaluasi untuk HPO / Joint FS+HPO (`80 + 80` untuk arsitektur Tandem).
- **Klasifier (6)**: `cat` (CatBoost), `xgb` (XGBoost), `rf` (Random Forest), `svm` (SVM), `knn` (KNN), dan `stack` (Stacking RF + SVM + Meta-learner).
- **Optimizer FS Metaheuristik (5)**: `bba`, `bmao`, `bsgo`, `ibssa`, `ma`.
- **Metode Penanganan Imbalance (5)**: `none`, `classweight`, `under`, `smote`, `borderline`.
- **Fungsi Transfer (4)**: `sigmoid`, `tanh`, `step`, `vshape`.

---

## Dataset (`data/`)

Seluruh dataset telah diproses awal dan disimpan sebagai arsip NumPy (`.npz`) beserta metadata di [`data/info.json`](data/info.json) (juga tersedia versi penuh pada berkas `data/full_*.npz`):

| Dataset | Baris Mentah | Baris Terpakai | Duplikat Dihapus | Fitur ($p$) | Rasio Positif | Split (Train / Val / Test) |
|---|---:|---:|---:|---:|---:|---:|
| `cleveland` | 303 | 303 | 0 | 13 | 45,87% | 212 / 45 / 46 |
| `statlog` | 270 | 270 | 0 | 13 | 44,44% | 189 / 40 / 41 |
| `hungarian` | 294 | 293 | 1 | 13 | 36,18% | 205 / 44 / 44 |
| `cardio` | 68.709 | 4.240 | 3.815 | 11 | 50,85% | 2.968 / 636 / 636 |
| `framingham` | 4.240 | 4.240 | 0 | 15 | 15,19% | 2.968 / 636 / 636 |
| `brfss` | 308.774 | 5.000 | 0 | 18 | 8,08% | 3.500 / 750 / 750 |

> **Catatan:** `brfss` merupakan *stratified subsample* 5.000 baris (8,08% positif) untuk menguji skenario *imbalance* ekstrem di luar `framingham`. Pada tahap **E5**, dataset `cardio` juga divalidasi dalam skala penuh (`cardio_full`, `full_cardio.npz`) dengan pembatasan latih SVM/KNN (15k/20k) demi kelayakan eksekusi CPU.

---

## Struktur Repositori & Artefak

```text
Remixing-Metaheuristic/
├── README.md
├── REPORT_RJ_V2.md                         # Laporan ringkas hasil eksperimen V2
├── Remix_Revisi_Joint_CVD_v2_1.ipynb       # Notebook utama pipeline eksperimen V2.1
├── data/                                   # Cache dataset (.npz, full_*.npz) & info.json
├── figures/                                # 7 figur utama (fig1–fig7) + plot SHAP
├── raw/                                    # Direktori placeholder data mentah
├── rj2_exp1a_imbalance.csv                 # Hasil E1A (900 baris)
├── rj2_strategy_selection.csv              # Strategi imbalance adaptif terpilih (36 baris)
├── rj2_exp1b_leakage_audit.csv             # Hasil E1B audit kebocoran SMOTE (180 baris)
├── rj2_exp2a_fitness_order.csv             # Hasil E2A sensitivitas fitness & urutan (216 baris)
├── rj2_exp2b_remixing.csv                  # Hasil E2B remixing 30 pasangan (180 baris)
├── rj2_exp2t_transfer.csv                  # Hasil E2t 4 fungsi transfer × 5 optimizer (240 baris)
├── rj2_exp3a_hpo_only.csv                  # Hasil E3A baseline HPO-only (108 baris)
├── rj2_exp3b_joint.csv                     # Hasil E3B Joint FS+HPO (180 baris)
├── rj2_exp3c_multiseed.csv                 # Hasil E3C stabilitas multi-seed Top-6 (183 baris)
├── rj2_exp4_tandem_joint.csv               # Hasil E4 Tandem vs Joint Fixed vs Joint Full (221 baris)
├── rj2_exp5_cardio70k.csv                  # Hasil E5 validasi Cardio 70k & Framingham (24 baris)
├── rj2_friedman.csv                        # Hasil uji statistik Friedman (25 baris)
├── rj2_nemenyi.csv                         # Hasil uji post-hoc Nemenyi (5 baris)
├── rj2_pairwise.csv                        # Hasil uji Wilcoxon + koreksi Holm-Bonferroni (130 baris)
├── shap_values_cleveland_v2.csv            # Nilai SHAP model pemenang pada Cleveland
├── shap_values_framingham_v2.csv           # Nilai SHAP model pemenang pada Framingham
├── lime_cleveland_v2.csv                   # Bobot penjelasan lokal LIME pada Cleveland
└── lime_framingham_v2.csv                  # Bobot penjelasan lokal LIME pada Framingham
```

### Daftar Visualisasi (`figures/`)
- `fig1_imbalance_and_leakage.png` — Perbandingan metode *imbalance* & efek inflasi kebocoran SMOTE pra-split
- `fig2_fitness_sensitivity_ablation.png` — Ablasi fungsi fitness (`F_Acc`, `F_BA`, `F_AP+BA`) dan urutan *resampling*
- `fig3_remixing_heatmap_cd.png` — *Heatmap* performa 30 pasangan *remixing* dan diagram *Critical Difference* (Nemenyi, `CD = 1.017`)
- `fig4_transfer_balanced_vs_imbalanced.png` — Perbandingan 4 fungsi transfer pada dataset seimbang vs timpang
- `fig5_four_quadrant_and_pareto.png` — Analisis 4-kuadran ablasi & *Pareto front* (HPO-only vs FS vs Joint)
- `fig6_tandem_vs_joint_convergence.png` — Kurva konvergensi arsitektur Tandem vs Joint Fixed vs Joint Full
- `fig7_xai_shap_lime.png` — Ringkasan interpretabilitas SHAP & LIME
- `shap_cleveland_v2.png`, `shap_framingham_v2.png` — Plot detail SHAP untuk dataset Cleveland dan Framingham

---

## Ringkasan Tahapan & Temuan Utama

Ringkasan lengkap tersedia di [`REPORT_RJ_V2.md`](REPORT_RJ_V2.md). Beberapa temuan kunci:

- **E1A (Metode Imbalance)**: `classweight` (`AP = 0.650`, `Recall = 0.761`, `BA = 0.775`) dan `under` (`AP = 0.650`, `Recall = 0.755`, `BA = 0.774`) meningkatkan *Recall* dan *Balanced Accuracy* secara signifikan dibanding `none` (`Recall = 0.558`, `BA = 0.722`). Strategi adaptif terbaik dicapai oleh `cleveland / cat → under` (`AP = 0.955`, `Recall = 0.952`).
- **E1B (Audit Kebocoran SMOTE)**: SMOTE *pre-split* menghasilkan inflasi performa palsu sebesar `+0.490` pada AP (`0.924` vs `0.434`) dan `+0.276` pada Recall (`0.810` vs `0.534`) dibanding protokol *in-fold* yang bersih.
- **E2A & E2t (Fitness, Urutan, dan Fungsi Transfer)**: Fungsi fitness `ap_ba` mengungguli `acc` secara signifikan pada metrik AP, BA, Recall, dan F1. Pada pengujian fungsi transfer (`E2t`), `tanh` mencatat rata-rata tertinggi (`AP = 0.719`, `Recall = 0.787`), diikuti `vshape` (`AP = 0.718`, `Recall = 0.783`).
- **E2B & E3B (Remixing FS vs Joint FS+HPO)**: Pada FS murni (`E2B`), `bba` meraih rata-rata Recall (`0.769`) dan AP (`0.647`) tertinggi. Pada skema kooperatif `E3B` (Joint FS+HPO), performa meningkat dengan `bmao` meraih Recall tertinggi (`0.771`, rata-rata fitur `8.444`) dan `bsgo` meraih AP (`0.663`) serta BA (`0.778`) tertinggi.
- **E4 (Arsitektur Optimasi)**: `joint_fixed_imb` memperoleh rata-rata AP `0.670` dan Recall tertinggi `0.797`, melampaui `tandem` (`AP = 0.670`, `Recall = 0.784`) maupun `joint_full_imb` (`AP = 0.631`, `Recall = 0.782`).
- **E5 (Validasi Skala Penuh Cardio 70k)**: Model `stack` + `bba` mencapai `AP = 0.778–0.779`, `Recall = 0.685–0.699`, dan `BA = 0.721–0.722` pada dataset Cardio penuh.

---

## Catatan Menjalankan Notebook

Notebook [`Remix_Revisi_Joint_CVD_v2_1.ipynb`](Remix_Revisi_Joint_CVD_v2_1.ipynb) dikonfigurasi untuk eksekusi CPU (`MFL_USE_GPU=0`, `MFL_WORKERS=12`, `MFL_THREADS=1`) dengan dukungan profil `smoke`, `fast`, dan `full` melalui variabel lingkungan `MFL_RJ_PROFILE`, serta dilengkapi verifikasi otomatis (`T1–T10`) dan mekanisme *resume* berbasis file CSV yang sudah tersimpan.

Dependensi paket kritis yang dicek oleh notebook:
`scikit-learn`, `imbalanced-learn`, `xgboost`, `catboost`, `shap`, `lime`, `scipy`, `tabulate` (opsional: `kagglehub`, `torch`).

> **Catatan Struktur Kode:** Notebook mengimpor modul inti (`rj2_core`, `run_rj_v2`, `remix_core`) dari subdirektori `replication/` yang tidak disertakan dalam repositori ini; repositori ini memuat notebook V2.1 beserta seluruh dataset terproses, hasil CSV eksperimen, uji statistik, artefak XAI, figur, dan laporan `REPORT_RJ_V2.md`.
