# EEG Motor Imagery BCI Classification

왼손/오른손 운동상상(Motor Imagery) EEG를 분류하는 BCI 포트폴리오 프로젝트입니다.  
PhysioNet EEG Motor Movement/Imagery Dataset(EEGBCI)을 사용해 전처리, CSP 특징 추출, LDA 분류, 개인별 평가, LOSO 평가, calibration 실험까지 진행했습니다.

## 프로젝트 목표

운동상상 EEG에서 왼손과 오른손 상태를 구분하고, 피험자 간 개인차가 분류 성능과 새로운 사용자에 대한 일반화 성능에 어떤 영향을 주는지 확인합니다.

## Dataset

- Dataset: PhysioNet EEG Motor Movement/Imagery Dataset (EEGBCI)
- Subjects: 1–10
- Runs: 4, 8, 12
- Task: Left fist vs Right fist motor imagery
- EEG channels: 64
- Sampling rate: 160 Hz
- Trials per subject: 45

원본 EEG 데이터는 저장소에 포함하지 않습니다.

## Pipeline

```text
EEGBCI EDF
    ↓
8–30 Hz Band-pass Filtering
    ↓
Event-based Epoching
    ↓
CSP (Common Spatial Patterns)
    ↓
LDA (Linear Discriminant Analysis)
    ↓
Left / Right Motor Imagery Classification
```

## Evaluation

### 1. Subject-specific classification

피험자별로 CSP + LDA 모델을 학습하고 교차검증했습니다.  
시간 구간과 CSP component 수의 영향을 비교했으며, 최종 평가는 Nested Cross-Validation을 사용했습니다.

### 2. Subject-independent classification

Leave-One-Subject-Out(LOSO) 방식으로 한 명을 완전히 제외한 뒤 나머지 9명으로 학습하고, 처음 보는 피험자를 평가했습니다.

- Mean LOSO accuracy: **54.4%**

### 3. Personal calibration

새로운 사용자의 소량 EEG trial을 기존 타인 데이터에 추가해 calibration 효과를 확인했습니다.

| Calibration trials | Mean accuracy |
|---:|---:|
| 0 | 54.4% |
| 5 | 54.6% |
| 10 | 57.8% |
| 20 | 59.2% |

개인 calibration 데이터가 증가하면서 전체 평균 성능은 점진적으로 향상되었지만, 개선 폭에는 큰 개인차가 있었습니다.

## Key Question

운동상상 EEG 분류에서는 한 사람의 데이터에서 높은 성능을 얻는 것뿐 아니라,
새로운 사용자에게도 모델이 일반화되는지가 중요합니다.

이 프로젝트에서는 다음 세 가지 질문을 중심으로 분석했습니다.

1. 개인별로 학습한 CSP + LDA 모델은 어느 정도의 분류 성능을 보이는가?
2. 여러 피험자의 데이터로 학습한 모델이 새로운 피험자에게도 일반화되는가?
3. 새로운 사용자의 소량 데이터를 추가하면 성능을 개선할 수 있는가?

이를 확인하기 위해 subject-specific Nested Cross-Validation, LOSO,
그리고 personal calibration 실험을 순차적으로 비교했습니다.

## Results Summary

| Evaluation | Mean Accuracy | Interpretation |
|---|---:|---|
| Subject-specific Nested CV | 59.8% | 개인별 모델의 분류 성능 |
| LOSO | 54.4% | 새로운 피험자에 대한 일반화 성능 |
| LOSO + 5 calibration trials | 54.6% | 소량의 개인 데이터 추가 |
| LOSO + 10 calibration trials | 57.8% | calibration 증가에 따른 평균 성능 개선 |
| LOSO + 20 calibration trials | 59.2% | calibration 증가에 따른 평균 성능 개선 |

Subject-specific 모델에서는 평균 59.8%의 정확도를 보였지만,
완전히 새로운 피험자를 대상으로 한 LOSO에서는 평균 54.4%로 성능이 낮아졌습니다.

새로운 사용자의 calibration 데이터를 추가했을 때 평균 정확도는
54.4%에서 최대 59.2%까지 향상되었습니다.
다만 calibration 효과의 크기는 피험자마다 달랐습니다.

이 결과는 EEG 기반 BCI에서 **피험자 간 차이(inter-subject variability)**가
새로운 사용자에 대한 일반화 성능을 제한할 수 있으며,
소량의 개인별 calibration 데이터가 이를 일부 보완할 가능성이 있음을 보여줍니다.

## Main Results

### Nested CV vs LOSO

![Nested CV vs LOSO](figures/nested_vs_loso.png)

- Mean subject-specific Nested CV accuracy: **59.8%**
- Mean LOSO accuracy: **54.4%**
- 일부 피험자는 개인별 모델에서 높은 정확도를 보였지만, 새로운 피험자에 대한 일반화 성능은 크게 감소했습니다.
- 예: Subject 7은 Nested CV에서 **93.3%**, LOSO에서는 **55.6%**를 기록했습니다.

### Calibration example — Subject 7

![Calibration Curve](figures/calibration_curve.png)

| Calibration trials | Accuracy |
|---:|---:|
| 0 | 55.6% |
| 5 | 61.5% |
| 10 | 69.4% |
| 20 | 76.8% |

Subject 7에서는 개인 데이터를 추가할수록 성능이 크게 향상되었습니다.

> 전체 10명 평균에서는 calibration 데이터가 0 / 5 / 10 / 20 trials로 증가할 때 정확도가 각각 **54.4% / 54.6% / 57.8% / 59.2%**로 변화했습니다.

### CSP Spatial Patterns

![CSP Spatial Patterns](figures/csp_patterns.png)

Subject 7의 CSP spatial pattern을 topomap으로 시각화했습니다.  
CSP가 좌/우 운동상상 EEG를 구분하기 위해 사용하는 공간적 분산 패턴을 확인할 수 있습니다.

## Interpretation

이번 프로젝트에서 확인한 핵심 내용은 다음과 같습니다.

- 운동상상 EEG 분류 성능은 피험자에 따라 크게 달랐습니다.
- 한 피험자에게 잘 작동한 파라미터가 다른 피험자에게 그대로 적용되지는 않았습니다.
- 타인의 EEG만 이용한 subject-independent 모델은 새로운 사용자에게 높은 성능으로 일반화하기 어려웠습니다.
- 개인 calibration 데이터는 평균 성능을 개선했지만, 효과 역시 피험자별로 달랐습니다.

## Limitations

- 피험자당 trial 수가 45개로 적습니다.
- 분석 대상은 Subject 1–10으로 제한했습니다.
- CSP + LDA 기반의 비교적 단순한 분류 모델을 사용했습니다.
- Calibration 데이터가 증가할수록 평가에 남는 개인 test trial 수가 감소합니다.
- 본 프로젝트는 포트폴리오용 탐색적 분석이며 임상적 성능을 주장하지 않습니다.

## Project Structure

```text
eeg-motor-imagery-bci/
├── README.md
├── requirements.txt
├── .gitignore
├── notebooks/
│   └── bci_analysis.ipynb
└── figures/
    ├── nested_vs_loso.png
    ├── calibration_curve.png
    └── csp_patterns.png
```

## Environment

Main packages:

- Python 3.13
- MNE-Python 1.12.1
- NumPy
- SciPy
- scikit-learn
- pandas
- matplotlib
- Jupyter Notebook

Install dependencies:

```bash
pip install -r requirements.txt
```

## Run

```bash
jupyter notebook
```

Open:

```text
notebooks/bci_analysis.ipynb
```

EEGBCI data is expected at:

```text
D:\bci_data
```

If using another path, change `DATA_PATH` in the notebook.

## Future Work

- More subjects
- Frequency-band comparison
- Channel-selection experiments
- Improved calibration strategy
- EEGNet or other deep-learning baselines
- Cross-session / cross-subject generalization analysis
