# REPORT_RJ_V2 — Remix Metaheuristic FS + HPO Imbalance-Aware (V2)

**Protocol:** fitness `F_AP+BA = 0.55(1-AP)+0.40(1-BA)+0.05(|m|/p)`, inner 3-fold CV, resampling in-fold only; hard budget 80 (FS) / 160 (HPO/Joint/Tandem); 6 datasets incl. BRFSS (~8.1% positive).

## Tujuan 01 — Imbalance leakage-free

E1A methods:

| imb_method   |   test_average_precision | test_recall   | test_balanced_accuracy   |
|:-------------|-------------------------:|:--------------|:-------------------------|
| borderline   |                    0.649 | 0.678         | 0.757                    |
| classweight  |                    0.650 | **0.761**     | **0.775**                |
| none         |                    0.648 | 0.558         | 0.722                    |
| smote        |                    0.650 | 0.670         | 0.755                    |
| under        |                    0.650 | 0.755         | 0.774                    |

Adaptive strategy terbaik: **cleveland / cat → under** (test AP 0.955, recall 0.952).

E1B leakage audit:

| protocol               |   test_average_precision |   test_recall |
|:-----------------------|-------------------------:|--------------:|
| smote_infold_clean     |                    0.434 |         0.534 |
| smote_presplit_leakage |                    0.924 |         0.810 |

Inflasi palsu pre-split SMOTE: AP +0.490, recall +0.276.

## Tujuan 02 — Recall dalam FS metaheuristik

E2B optimizer:

| fs_optimizer   |   test_average_precision | test_recall   |   test_balanced_accuracy |
|:---------------|-------------------------:|:--------------|-------------------------:|
| bba            |                    0.647 | **0.769**     |                    0.767 |
| bmao           |                    0.643 | 0.759         |                    0.764 |
| bsgo           |                    0.640 | 0.761         |                    0.768 |
| ibssa          |                    0.622 | 0.738         |                    0.755 |
| ma             |                    0.638 | 0.742         |                    0.752 |

E2A fitness/order:

| variant                  | test_recall   | test_balanced_accuracy   |
|:-------------------------|:--------------|:-------------------------|
| bba|acc|fs_then_smote    | 0.492         | 0.689                    |
| bba|ap_ba|fs_then_smote  | 0.526         | 0.697                    |
| bba|ap_ba|smote_then_fs  | 0.527         | **0.700**                |
| bba|ba|fs_then_smote     | 0.514         | 0.690                    |
| bmao|acc|fs_then_smote   | 0.412         | 0.655                    |
| bmao|ap_ba|fs_then_smote | **0.527**     | 0.697                    |
| bmao|ap_ba|smote_then_fs | 0.513         | 0.686                    |
| bmao|ba|fs_then_smote    | 0.520         | 0.690                    |

E2t transfer:

| transfer   |   test_average_precision | test_recall   |
|:-----------|-------------------------:|:--------------|
| sigmoid    |                    0.700 | 0.754         |
| step       |                    0.704 | 0.770         |
| tanh       |                    0.719 | **0.787**     |
| vshape     |                    0.718 | 0.783         |

## Tujuan 03 — Integrasi HPO+FS adil & efisien

E3B joint FS+HPO:

| fs_optimizer   |   test_average_precision | test_recall   |   test_balanced_accuracy |   n_selected |
|:---------------|-------------------------:|:--------------|-------------------------:|-------------:|
| bba            |                    0.659 | 0.756         |                    0.767 |       10.889 |
| bmao           |                    0.661 | **0.771**     |                    0.773 |        8.444 |
| bsgo           |                    0.663 | 0.768         |                    0.778 |       11.000 |
| ibssa          |                    0.656 | 0.756         |                    0.767 |        6.583 |
| ma             |                    0.654 | 0.746         |                    0.759 |        7.167 |

E4 architecture:

| architecture    |   test_average_precision | test_recall   |   opt_seconds |
|:----------------|-------------------------:|:--------------|--------------:|
| joint_fixed_imb |                    0.670 | **0.797**     |       146.426 |
| joint_full_imb  |                    0.631 | 0.782         |       154.726 |
| tandem          |                    0.670 | 0.784         |       135.074 |

E3C stability (within-dataset std across seeds):

| fs_optimizer   | classifier   | ap_mean   |   ap_std | rec_mean   |   rec_std |
|:---------------|:-------------|:----------|---------:|:-----------|----------:|
| bmao           | cat          | **0.683** |  nan     | 0.710      |   nan     |
| bsgo           | stack        | 0.679     |    0.021 | 0.804      |     0.033 |
| bba            | svm          | 0.676     |    0.017 | 0.802      |     0.022 |
| bba            | stack        | 0.675     |    0.021 | 0.798      |     0.032 |
| bmao           | svm          | 0.674     |    0.027 | **0.806**  |     0.029 |
| ibssa          | svm          | 0.656     |    0.023 | 0.793      |     0.051 |
| ma             | svm          | 0.643     |    0.026 | 0.799      |     0.044 |

## Statistik non-parametrik

**Friedman:**

| stage   | test                  | metric                 |   n_blocks |   k |     p |
|:--------|:----------------------|:-----------------------|-----------:|----:|------:|
| E1A     | friedman_methods      | test_average_precision |          5 |   5 | 0.624 |
| E1A     | friedman_methods      | test_balanced_accuracy |          5 |   5 | 0.093 |
| E1A     | friedman_methods      | test_recall            |          5 |   5 | 0.000 |
| E1A     | friedman_methods      | test_f1                |          5 |   5 | 0.219 |
| E1A     | friedman_methods      | test_mcc               |          5 |   5 | 0.454 |
| E2B     | friedman_fs_optimizer | test_average_precision |          5 |   5 | 0.060 |
| E2B     | friedman_fs_optimizer | test_balanced_accuracy |          5 |   5 | 0.383 |
| E2B     | friedman_fs_optimizer | test_recall            |          5 |   5 | 0.035 |
| E2B     | friedman_fs_optimizer | test_f1                |          5 |   5 | 0.143 |
| E2B     | friedman_fs_optimizer | test_mcc               |          5 |   5 | 0.270 |
| E2B     | friedman_classifier   | test_average_precision |          6 |   6 | 0.000 |
| E2B     | friedman_classifier   | test_balanced_accuracy |          6 |   6 | 0.004 |
| E2B     | friedman_classifier   | test_recall            |          6 |   6 | 0.015 |
| E2B     | friedman_classifier   | test_f1                |          6 |   6 | 0.093 |
| E2B     | friedman_classifier   | test_mcc               |          6 |   6 | 0.014 |
| E3B     | friedman_fs_optimizer | test_average_precision |          5 |   5 | 0.467 |
| E3B     | friedman_fs_optimizer | test_balanced_accuracy |          5 |   5 | 0.009 |
| E3B     | friedman_fs_optimizer | test_recall            |          5 |   5 | 0.080 |
| E3B     | friedman_fs_optimizer | test_f1                |          5 |   5 | 0.012 |
| E3B     | friedman_fs_optimizer | test_mcc               |          5 |   5 | 0.009 |
| E3B     | friedman_classifier   | test_average_precision |          6 |   6 | 0.008 |
| E3B     | friedman_classifier   | test_balanced_accuracy |          6 |   6 | 0.007 |
| E3B     | friedman_classifier   | test_recall            |          6 |   6 | 0.009 |
| E3B     | friedman_classifier   | test_f1                |          6 |   6 | 0.040 |
| E3B     | friedman_classifier   | test_mcc               |          6 |   6 | 0.042 |

**Nemenyi:**

| stage   | group   |   mean_rank |   n_blocks |    cd |
|:--------|:--------|------------:|-----------:|------:|
| E2B     | bba     |       3.375 |         36 | 1.017 |
| E2B     | bmao    |       3.167 |         36 | 1.017 |
| E2B     | bsgo    |       3.125 |         36 | 1.017 |
| E2B     | ibssa   |       2.333 |         36 | 1.017 |
| E2B     | ma      |       3.000 |         36 | 1.017 |

**Wilcoxon + Holm (5 metrik):**

|   n |   mean_a |   mean_b |   mean_diff |   p_wilcoxon | comparison                             | metric                 | family         |   p_holm | sig_holm   |
|----:|---------:|---------:|------------:|-------------:|:---------------------------------------|:-----------------------|:---------------|---------:|:-----------|
|  36 |   0.6520 |   0.6480 |      0.0050 |       0.9300 | adaptive_vs_none                       | test_average_precision | E1A_imbalance  |   1.0000 | False      |
|  18 |   0.4720 |   0.4630 |      0.0090 |       0.8650 | adaptive_vs_none_imbalanced            | test_average_precision | E1A_imbalance  |   1.0000 | False      |
|  36 |   0.7780 |   0.7220 |      0.0560 |       0.0320 | adaptive_vs_none                       | test_balanced_accuracy | E1A_imbalance  |   0.0320 | True       |
|  18 |   0.7380 |   0.6250 |      0.1130 |       0.0000 | adaptive_vs_none_imbalanced            | test_balanced_accuracy | E1A_imbalance  |   0.0010 | True       |
|  36 |   0.7710 |   0.5580 |      0.2130 |       0.0000 | adaptive_vs_none                       | test_recall            | E1A_imbalance  |   0.0000 | True       |
|  18 |   0.7220 |   0.2960 |      0.4260 |       0.0000 | adaptive_vs_none_imbalanced            | test_recall            | E1A_imbalance  |   0.0000 | True       |
|  36 |   0.6460 |   0.5640 |      0.0820 |       0.0340 | adaptive_vs_none                       | test_f1                | E1A_imbalance  |   0.0340 | True       |
|  18 |   0.4840 |   0.3190 |      0.1650 |       0.0000 | adaptive_vs_none_imbalanced            | test_f1                | E1A_imbalance  |   0.0010 | True       |
|  36 |   0.5100 |   0.4660 |      0.0430 |       0.1360 | adaptive_vs_none                       | test_mcc               | E1A_imbalance  |   0.1360 | False      |
|  18 |   0.3840 |   0.2950 |      0.0890 |       0.0090 | adaptive_vs_none_imbalanced            | test_mcc               | E1A_imbalance  |   0.0180 | True       |
|  90 |   0.9240 |   0.4340 |      0.4900 |       0.0000 | smote_presplit_leakage_vs_infold_clean | test_average_precision | E1B_leakage    |   0.0000 | True       |
|  90 |   0.8520 |   0.6810 |      0.1710 |       0.0000 | smote_presplit_leakage_vs_infold_clean | test_balanced_accuracy | E1B_leakage    |   0.0000 | True       |
|  90 |   0.8100 |   0.5340 |      0.2760 |       0.0000 | smote_presplit_leakage_vs_infold_clean | test_recall            | E1B_leakage    |   0.0000 | True       |
|  90 |   0.8420 |   0.4480 |      0.3940 |       0.0000 | smote_presplit_leakage_vs_infold_clean | test_f1                | E1B_leakage    |   0.0000 | True       |
|  90 |   0.7090 |   0.3300 |      0.3800 |       0.0000 | smote_presplit_leakage_vs_infold_clean | test_mcc               | E1B_leakage    |   0.0000 | True       |
|  27 |   0.4820 |   0.4690 |      0.0140 |       0.0170 | ap_ba_vs_acc|bba                       | test_average_precision | E2A_fitness    |   0.1040 | False      |
|  27 |   0.4800 |   0.4260 |      0.0540 |       0.0000 | ap_ba_vs_acc|bmao                      | test_average_precision | E2A_fitness    |   0.0000 | True       |
|  54 |   0.4810 |   0.4470 |      0.0340 |       0.0000 | ap_ba_vs_acc                           | test_average_precision | E2A_fitness    |   0.0000 | True       |
|  27 |   0.4730 |   0.4690 |      0.0050 |       0.3990 | ba_vs_acc|bba                          | test_average_precision | E2A_fitness    |   0.3990 | False      |
|  27 |   0.4530 |   0.4260 |      0.0270 |       0.0360 | ba_vs_acc|bmao                         | test_average_precision | E2A_fitness    |   0.1450 | False      |
|  54 |   0.4630 |   0.4470 |      0.0160 |       0.0190 | ba_vs_acc                              | test_average_precision | E2A_fitness    |   0.1040 | False      |
|  27 |   0.4820 |   0.4730 |      0.0090 |       0.0460 | ap_ba_vs_ba|bba                        | test_average_precision | E2A_fitness    |   0.1450 | False      |
|  27 |   0.4800 |   0.4530 |      0.0260 |       0.0580 | ap_ba_vs_ba|bmao                       | test_average_precision | E2A_fitness    |   0.1450 | False      |
|  54 |   0.4810 |   0.4630 |      0.0180 |       0.0060 | ap_ba_vs_ba                            | test_average_precision | E2A_fitness    |   0.0420 | True       |
|  54 |   0.4840 |   0.4780 |      0.0060 |       0.3010 | fs_then_smote_vs_smote_then_fs         | test_average_precision | E2A_order      |   0.3010 | False      |
|  27 |   0.6990 |   0.6890 |      0.0100 |       0.1100 | ap_ba_vs_acc|bba                       | test_balanced_accuracy | E2A_fitness    |   0.4400 | False      |
|  27 |   0.6920 |   0.6550 |      0.0370 |       0.0080 | ap_ba_vs_acc|bmao                      | test_balanced_accuracy | E2A_fitness    |   0.0590 | False      |
|  54 |   0.6950 |   0.6720 |      0.0230 |       0.0020 | ap_ba_vs_acc                           | test_balanced_accuracy | E2A_fitness    |   0.0170 | True       |
|  27 |   0.6900 |   0.6890 |      0.0010 |       0.3390 | ba_vs_acc|bba                          | test_balanced_accuracy | E2A_fitness    |   1.0000 | False      |
|  27 |   0.6900 |   0.6550 |      0.0360 |       0.0070 | ba_vs_acc|bmao                         | test_balanced_accuracy | E2A_fitness    |   0.0590 | False      |
|  54 |   0.6900 |   0.6720 |      0.0190 |       0.0080 | ba_vs_acc                              | test_balanced_accuracy | E2A_fitness    |   0.0590 | False      |
|  27 |   0.6990 |   0.6900 |      0.0080 |       0.0680 | ap_ba_vs_ba|bba                        | test_balanced_accuracy | E2A_fitness    |   0.3390 | False      |
|  27 |   0.6920 |   0.6900 |      0.0020 |       0.6650 | ap_ba_vs_ba|bmao                       | test_balanced_accuracy | E2A_fitness    |   1.0000 | False      |
|  54 |   0.6950 |   0.6900 |      0.0050 |       0.4670 | ap_ba_vs_ba                            | test_balanced_accuracy | E2A_fitness    |   1.0000 | False      |
|  54 |   0.6970 |   0.6930 |      0.0040 |       0.2120 | fs_then_smote_vs_smote_then_fs         | test_balanced_accuracy | E2A_order      |   0.2120 | False      |
|  27 |   0.5270 |   0.4920 |      0.0350 |       0.0290 | ap_ba_vs_acc|bba                       | test_recall            | E2A_fitness    |   0.1450 | False      |
|  27 |   0.5200 |   0.4120 |      0.1080 |       0.0000 | ap_ba_vs_acc|bmao                      | test_recall            | E2A_fitness    |   0.0020 | True       |
|  54 |   0.5230 |   0.4520 |      0.0710 |       0.0000 | ap_ba_vs_acc                           | test_recall            | E2A_fitness    |   0.0000 | True       |
|  27 |   0.5140 |   0.4920 |      0.0220 |       0.1130 | ba_vs_acc|bba                          | test_recall            | E2A_fitness    |   0.4540 | False      |
|  27 |   0.5200 |   0.4120 |      0.1080 |       0.0010 | ba_vs_acc|bmao                         | test_recall            | E2A_fitness    |   0.0080 | True       |
|  54 |   0.5170 |   0.4520 |      0.0650 |       0.0010 | ba_vs_acc                              | test_recall            | E2A_fitness    |   0.0040 | True       |
|  27 |   0.5270 |   0.5140 |      0.0130 |       0.2670 | ap_ba_vs_ba|bba                        | test_recall            | E2A_fitness    |   0.8010 | False      |
|  27 |   0.5200 |   0.5200 |     -0.0000 |       0.8950 | ap_ba_vs_ba|bmao                       | test_recall            | E2A_fitness    |   1.0000 | False      |
|  54 |   0.5230 |   0.5170 |      0.0060 |       0.5180 | ap_ba_vs_ba                            | test_recall            | E2A_fitness    |   1.0000 | False      |
|  54 |   0.5270 |   0.5200 |      0.0070 |       0.3490 | fs_then_smote_vs_smote_then_fs         | test_recall            | E2A_order      |   0.3490 | False      |
|  27 |   0.4710 |   0.4600 |      0.0110 |       0.0810 | ap_ba_vs_acc|bba                       | test_f1                | E2A_fitness    |   0.3260 | False      |
|  27 |   0.4610 |   0.4070 |      0.0540 |       0.0100 | ap_ba_vs_acc|bmao                      | test_f1                | E2A_fitness    |   0.0810 | False      |
|  54 |   0.4660 |   0.4330 |      0.0320 |       0.0020 | ap_ba_vs_acc                           | test_f1                | E2A_fitness    |   0.0190 | True       |
|  27 |   0.4600 |   0.4600 |      0.0000 |       0.5370 | ba_vs_acc|bba                          | test_f1                | E2A_fitness    |   1.0000 | False      |
|  27 |   0.4630 |   0.4070 |      0.0560 |       0.0100 | ba_vs_acc|bmao                         | test_f1                | E2A_fitness    |   0.0810 | False      |
|  54 |   0.4620 |   0.4330 |      0.0280 |       0.0180 | ba_vs_acc                              | test_f1                | E2A_fitness    |   0.1110 | False      |
|  27 |   0.4710 |   0.4600 |      0.0110 |       0.0220 | ap_ba_vs_ba|bba                        | test_f1                | E2A_fitness    |   0.1110 | False      |
|  27 |   0.4610 |   0.4630 |     -0.0030 |       0.4270 | ap_ba_vs_ba|bmao                       | test_f1                | E2A_fitness    |   1.0000 | False      |
|  54 |   0.4660 |   0.4620 |      0.0040 |       0.5460 | ap_ba_vs_ba                            | test_f1                | E2A_fitness    |   1.0000 | False      |
|  54 |   0.4680 |   0.4640 |      0.0040 |       0.2850 | fs_then_smote_vs_smote_then_fs         | test_f1                | E2A_order      |   0.2850 | False      |
|  27 |   0.3700 |   0.3650 |      0.0050 |       0.2440 | ap_ba_vs_acc|bba                       | test_mcc               | E2A_fitness    |   0.9750 | False      |
|  27 |   0.3550 |   0.3110 |      0.0440 |       0.0410 | ap_ba_vs_acc|bmao                      | test_mcc               | E2A_fitness    |   0.3270 | False      |
|  54 |   0.3630 |   0.3380 |      0.0250 |       0.0280 | ap_ba_vs_acc                           | test_mcc               | E2A_fitness    |   0.2520 | False      |
|  27 |   0.3560 |   0.3650 |     -0.0090 |       0.7440 | ba_vs_acc|bba                          | test_mcc               | E2A_fitness    |   1.0000 | False      |
|  27 |   0.3540 |   0.3110 |      0.0430 |       0.0490 | ba_vs_acc|bmao                         | test_mcc               | E2A_fitness    |   0.3270 | False      |
|  54 |   0.3550 |   0.3380 |      0.0170 |       0.1010 | ba_vs_acc                              | test_mcc               | E2A_fitness    |   0.5050 | False      |
|  27 |   0.3700 |   0.3560 |      0.0140 |       0.0450 | ap_ba_vs_ba|bba                        | test_mcc               | E2A_fitness    |   0.3270 | False      |
|  27 |   0.3550 |   0.3540 |      0.0010 |       0.4860 | ap_ba_vs_ba|bmao                       | test_mcc               | E2A_fitness    |   1.0000 | False      |
|  54 |   0.3630 |   0.3550 |      0.0080 |       0.4910 | ap_ba_vs_ba                            | test_mcc               | E2A_fitness    |   1.0000 | False      |
|  54 |   0.3660 |   0.3600 |      0.0070 |       0.1950 | fs_then_smote_vs_smote_then_fs         | test_mcc               | E2A_order      |   0.1950 | False      |
|  36 |   0.6430 |   0.6380 |      0.0050 |       0.9690 | bmao_vs_ma                             | test_average_precision | E2B_optimizers |   1.0000 | False      |
|  36 |   0.6430 |   0.6220 |      0.0200 |       0.1120 | bmao_vs_ibssa                          | test_average_precision | E2B_optimizers |   0.4460 | False      |
|  36 |   0.6430 |   0.6400 |      0.0020 |       0.7040 | bmao_vs_bsgo                           | test_average_precision | E2B_optimizers |   1.0000 | False      |
|  36 |   0.6430 |   0.6470 |     -0.0040 |       0.2860 | bmao_vs_bba                            | test_average_precision | E2B_optimizers |   0.8570 | False      |
|  36 |   0.7640 |   0.7520 |      0.0120 |       0.0820 | bmao_vs_ma                             | test_balanced_accuracy | E2B_optimizers |   0.3280 | False      |
|  36 |   0.7640 |   0.7550 |      0.0090 |       0.4170 | bmao_vs_ibssa                          | test_balanced_accuracy | E2B_optimizers |   1.0000 | False      |
|  36 |   0.7640 |   0.7680 |     -0.0040 |       0.4690 | bmao_vs_bsgo                           | test_balanced_accuracy | E2B_optimizers |   1.0000 | False      |
|  36 |   0.7640 |   0.7670 |     -0.0030 |       0.6310 | bmao_vs_bba                            | test_balanced_accuracy | E2B_optimizers |   1.0000 | False      |
|  36 |   0.7590 |   0.7420 |      0.0170 |       0.2220 | bmao_vs_ma                             | test_recall            | E2B_optimizers |   0.6660 | False      |
|  36 |   0.7590 |   0.7380 |      0.0210 |       0.1640 | bmao_vs_ibssa                          | test_recall            | E2B_optimizers |   0.6560 | False      |
|  36 |   0.7590 |   0.7610 |     -0.0020 |       0.5530 | bmao_vs_bsgo                           | test_recall            | E2B_optimizers |   0.6660 | False      |
|  36 |   0.7590 |   0.7690 |     -0.0100 |       0.2270 | bmao_vs_bba                            | test_recall            | E2B_optimizers |   0.6660 | False      |
|  36 |   0.6310 |   0.6170 |      0.0140 |       0.1160 | bmao_vs_ma                             | test_f1                | E2B_optimizers |   0.4630 | False      |
|  36 |   0.6310 |   0.6190 |      0.0120 |       0.1990 | bmao_vs_ibssa                          | test_f1                | E2B_optimizers |   0.5970 | False      |
|  36 |   0.6310 |   0.6360 |     -0.0050 |       0.3820 | bmao_vs_bsgo                           | test_f1                | E2B_optimizers |   0.7650 | False      |

## Validasi skala penuh Cardio 70k

| classifier   | fs_optimizer   |   seed | test_average_precision   | test_recall   |   test_balanced_accuracy |   fit_seconds |
|:-------------|:---------------|-------:|:-------------------------|:--------------|-------------------------:|--------------:|
| cat          | bmao           |     42 | 0.764                    | 0.681         |                    0.716 |         1.221 |
| cat          | bmao           |     52 | 0.763                    | 0.680         |                    0.717 |         1.039 |
| cat          | bmao           |     62 | 0.764                    | 0.682         |                    0.716 |         1.106 |
| stack        | bba            |     42 | 0.778                    | **0.699**     |                    0.722 |        35.901 |
| stack        | bba            |     52 | 0.778                    | 0.685         |                    0.722 |        34.467 |
| stack        | bba            |     62 | **0.779**                | 0.695         |                    0.721 |        30.632 |
| svm          | bba            |     42 | 0.774                    | 0.683         |                    0.724 |         4.786 |
| svm          | bba            |     52 | 0.773                    | 0.665         |                    0.723 |         4.639 |
| svm          | bba            |     62 | 0.774                    | 0.683         |                    0.724 |         4.696 |
| svm          | bmao           |     42 | 0.770                    | 0.665         |                    0.714 |         5.781 |
| svm          | bmao           |     52 | 0.770                    | 0.668         |                    0.714 |         5.817 |
| svm          | bmao           |     62 | 0.770                    | 0.665         |                    0.714 |         6.688 |
| cat          | bmao           |     42 | 0.335                    | 0.583         |                    0.617 |         0.327 |
| cat          | bmao           |     52 | 0.339                    | 0.573         |                    0.623 |         0.321 |
| cat          | bmao           |     62 | 0.341                    | 0.583         |                    0.623 |         0.339 |
| stack        | bba            |     42 | 0.336                    | 0.615         |                    0.653 |         3.297 |
| stack        | bba            |     52 | 0.337                    | 0.615         |                    0.652 |         3.335 |
| stack        | bba            |     62 | 0.337                    | 0.604         |                    0.646 |         3.311 |
| svm          | bba            |     42 | 0.324                    | 0.604         |                    0.640 |         0.205 |
| svm          | bba            |     52 | 0.324                    | 0.604         |                    0.640 |         0.210 |
| svm          | bba            |     62 | 0.324                    | 0.604         |                    0.640 |         0.194 |
| svm          | bmao           |     42 | 0.351                    | 0.635         |                    0.657 |         0.621 |
| svm          | bmao           |     52 | 0.351                    | 0.635         |                    0.657 |         0.609 |
| svm          | bmao           |     62 | 0.351                    | 0.635         |                    0.657 |         0.590 |

## Hyperparameter & fitur

| klasifier   | ruang                                                                                                                                                                           |
|:------------|:--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| svm         | {"C": [0.01, 100.0, "log"], "gamma": [0.001, 1.0, "log"]}                                                                                                                       |
| knn         | {"k": [1, 25, "int"]}                                                                                                                                                           |
| rf          | {"n_estimators": [30, 200, "int"], "max_depth": [2, 20, "int"], "min_samples_leaf": [1, 10, "int"]}                                                                             |
| xgb         | {"n_estimators": [40, 250, "int"], "max_depth": [2, 10, "int"], "learning_rate": [0.01, 0.3, "log"], "subsample": [0.6, 1.0, "float"], "colsample_bytree": [0.6, 1.0, "float"]} |
| cat         | {"iterations": [60, 300, "int"], "depth": [2, 10, "int"], "learning_rate": [0.01, 0.3, "log"], "l2_leaf_reg": [1.0, 10.0, "float"]}                                             |
| stack       | {"rf_n_estimators": [30, 150, "int"], "svm_C": [0.1, 10.0, "log"], "meta_C": [0.1, 10.0, "log"]}                                                                                |


Subset fitur terbaik per dataset:

| dataset    | classifier   | fs_optimizer   |   n_selected | selected_features                                                                        |
|:-----------|:-------------|:---------------|-------------:|:-----------------------------------------------------------------------------------------|
| brfss      | rf           | ibssa          |            7 | ["General_Health", "Checkup", "Other_Cancer", "Arthritis", "Sex", "Age_Category", "BMI"] |
| cardio     | xgb          | bsgo           |            7 | ["age", "height", "weight", "ap_hi", "cholesterol", "alco", "active"]                    |
| cleveland  | rf           | ibssa          |            8 | ["sex", "cp", "trestbps", "restecg", "exang", "slope", "ca", "thal"]                     |
| framingham | cat          | ibssa          |            7 | ["male", "age", "cigsPerDay", "diabetes", "sysBP", "BMI", "glucose"]                     |
| hungarian  | cat          | ibssa          |            9 | ["sex", "cp", "chol", "restecg", "thalach", "exang", "oldpeak", "slope", "thal"]         |
| statlog    | stack        | bmao           |            9 | ["age", "sex", "cp", "trestbps", "fbs", "restecg", "thalach", "oldpeak", "ca"]           |

## Catatan & batasan
- GPU dimatikan (`MFL_USE_GPU=0`); XGB/CatBoost CPU.
- Hard budget 80/160 diverifikasi; `n_evals` tercatat per sel.
- BRFSS = stratified subsample 5.000 baris (8,1% positif).
- Cardio 70k memakai cap SVM/KNN (15k/20k) untuk kelayakan CPU.
