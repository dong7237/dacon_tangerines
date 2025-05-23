# 감귤 착과량 예측 모델 개선 프로젝트 (Legacy vs. Vol.2)
* [분석 내용 노션링크](https://www.notion.so/1f631ee28e1b80009ae6df49994d2fdd?pvs=4)

**프로젝트 목표:** 제주 감귤의 생육 데이터를 활용하여 최종 착과량을 예측하는 머신러닝 모델을 개발하고, 초기 분석(2023년)의 한계점을 개선하여 예측 성능을 향상시키는 것을 목표로 합니다.

**프로젝트 기간:**
* 레거시 분석 (착과량예측.ipynb): 2023년 10월
* 개선 분석 (착과량예측_vol2.ipynb): 2025년 05월

**핵심 성과:**
* 데이터 전처리, 특성 공학 및 모델 평가 방법론 개선을 통해 모델 성능 리더보드 순위 52등 상승 (310->258)
* 교차 검증을 통한 모델 일반화 성능 및 신뢰도 확보

**사용 기술:**
* Python, Jupyter Notebook
* Pandas, NumPy
* Matplotlib, Seaborn
* Scikit-learn (RandomForestRegressor, XGBoost, K-Means, StratifiedKFold, metrics)

---

## 프로젝트 진행 과정 및 버전별 비교

* 본 프로젝트는 2023년에 수행한 초기 분석(Legacy)의 한계점을 파악하고, 이를 개선하기 위한 새로운 접근 방식(Vol.2)을 도입하여 모델의 성능과 분석의 깊이를 향상시키는 과정을 담고 있습니다.
* [예측대회 링크](https://dacon.io/competitions/official/236038/overview/description)

### 1. 데이터 개요

* **제공 데이터:** `train.csv`, `test.csv`, `sample_submission.csv`
* **주요 변수:**
    * ID: 과수나무 고유 ID
    * 착과량(int): 실제 감귤 착과량 (목표 변수)
    * 나무 생육 상태 Features (5개): 수고(m), 수관폭1(min), 수관폭2(max), 수관폭평균
    * 새순 Features (89개): 2022년 09월 01일 ~ 2022년 11월 28일 일별 측정 데이터
    * 엽록소 Features (89개): 2022년 09월 01일 ~ 2022년 11월 28일 일별 측정 데이터
* **데이터 특징:** 시계열 특성을 가진 새순 및 엽록소 데이터 포함

---

### 2. Legacy 분석 (2023년) 

#### 2.1. 주요 접근 방식

* **EDA (탐색적 데이터 분석):**
    * 결측치 및 기초 통계량 확인.
    * 주요 변수 분포 시각화.
    * 착과량 상/하위 25% 그룹 간 특성 비교 분석.
* **특성 공학 시도:**
    1.  목표 변수와 상관관계가 0.85 이상인 특성 선택 (주로 일별 새순 데이터).
    2.  나무 정보, 새순/엽록소 데이터의 주별 평균, 최소, 최대, 차이 등 요약 통계량 및 비율 특성 생성.
* **모델링:**
    * 스태킹(Stacking) 기법 적용.
        * 기반 모델: RandomForestRegressor, XGBRegressor.
        * 메타 모델: RandomForestRegressor.

#### 2.2. 결과 및 한계점

* **성과:** 스태킹 모델을 통해 예측 가능성을 탐색함 (기반 모델 RF MSE: 1375.08, XGB MSE: 1472.97).
* **한계점:**
    * **특성 공학:** 다수의 원본 시계열 특성 직접 사용 또는 단순 집계로 인해 데이터의 동적인 특성이나 잠재적인 패턴 반영 미흡. 상관관계 기반 특성 선택 시 다중공선성 및 과적합 우려.
    * **모델 평가:** 스태킹 모델의 메타 모델 평가 시, 기반 모델들의 테스트셋 예측값을 학습 및 평가 데이터로 동시에 사용하여 MSE가 과도하게 낙관적으로 측정됨 (228.93). 이는 모델의 일반화 성능에 대한 신뢰할 수 있는 지표가 아님을 확인. -> data leakage 문제가 있었으나 당시에는 인지하지 못하였음.
    * **데이터 전처리:** 이상치 처리에 대한 명시적인 고려 부족.

---

### 3. Vol.2 분석 (2025년 개선)

#### 3.1. 주요 개선 사항 및 접근 방식

* **데이터 전처리 강화:**
    * **이상치 제거:** 목표 변수인 '착과량(int)'의 분포를 고려하여 상하위 5% 데이터를 제거하여 모델 학습의 안정성 및 일반화 성능 향상 도모.
* **고도화된 특성 공학:**
    * **요약 통계량 확장:**
        * 새순/엽록소 데이터의 전체 기간에 대한 평균, 최대, 최소, 표준편차 계산.
        * 월별 새순/엽록소 데이터의 평균, 최대, 최소, 표준편차 계산하여 계절성 및 월별 변화 패턴 반영.
    * **추세 특성(Slope Features) 생성:**
        * 각 나무별 새순 길이 및 엽록소 수치의 시간에 따른 변화 추세(선형 회귀 기울기)를 계산하여 식물의 생장 패턴 및 건강 상태 변화를 나타내는 동적 특성 추가.
    * **군집화(Clustering) 특성 추가:**
        * K-평균 군집화 (K=3, 엘보우 플롯 기반 결정)를 활용하여 '수고(m)', '수관폭평균', '새순평균', '엽록소평균' 특성(로그 변환 후)을 기반으로 유사한 생육 특성을 가진 나무 그룹 생성.
        * 생성된 클러스터 레이블을 원-핫 인코딩하여 모델의 입력 특성으로 활용.
* **강건한 모델링 및 평가:**
    * **모델 선택:** RandomForestRegressor (주요 모델), XGBoost (비교 모델).
    * **하이퍼파라미터:**
        * RandomForest: `n_estimators=100, max_depth=10, random_state=42`
        * XGBoost: `n_estimators=150, max_depth=15, learning_rate=0.01, subsample=0.8, colsample_bytree=0.7, gamma=0.1, reg_alpha=0.1, reg_lambda=1, random_state=42`
    * **모델 평가:** StratifiedKFold 교차 검증 (5-fold)을 도입하여 모델 성능 평가의 신뢰성 확보 및 data leakage 문제 해결
* **특성 중요도 분석:** RandomForest 모델의 특성 중요도를 확인하여 예측에 큰 영향을 미치는 변수 파악.

#### 3.2. 결과

* **RandomForestRegressor (교차 검증 평균):** R² 약 0.9631
* **RandomForestRegressor (단일 테스트 분할):** **MSE 약 1324.60** (레거시 분석(data leakage 이전       * 기반 모델: RandomForestRegressor, XGBRegressor.
        * 메타 모델: RandomForestRegressor.

#### 2.2. 결과 및 한계점

* **성과:** 스태킹 모델을 통해 예측 가능성을 탐색함 (기반 모델 RF MSE: 1375.08, XGB MSE: 1472.97).
* **한계점:**
    * **특성 공학:** 다수의 원본 시계열 특성 직접 사용 또는 단순 집계로 인해 데이터의 동적인 특성이나 잠재적인 패턴 반영 미흡. 상관관계 기반 특성 선택 시 다중공선성 및 과적합 우려.
    * **모델 평가:** 스태킹 모델의 메타 모델 평가 시, 기반 모델들의 테스트셋 예측값을 학습 및 평가 데이터로 동시에 사용하여 MSE가 과도하게 낙관적으로 측정됨 (228.93). 이는 모델의 일반화 성능에 대한 신뢰할 수 있는 지표가 아님을 확인. -> data leakage 문제가 있었으나 당시에는 인지하지 못하였음.
    * **데이터 전처리:** 이상치 처리에 대한 명시적인 고려 부족.

---

### 3. Vol.2 분석 (2025년 개선) 

#### 3.1. 주요 개선 사항 및 접근 방식

* **데이터 전처리 강화:**
    * **이상치 제거:** 목표 변수인 '착과량(int)'의 분포를 고려하여 상하위 5% 데이터를 제거하여 모델 학습의 안정성 및 일반화 성능 향상 도모.
* **고도화된 특성 공학:**
    * **요약 통계량 확장:**
        * 새순/엽록소 데이터의 전체 기간에 대한 평균, 최대, 최소, 표준편차 계산.
        * 월별 새순/엽록소 데이터의 평균, 최대, 최소, 표준편차 계산하여 계절성 및 월별 변화 패턴 반영.
    * **추세 특성(Slope Features) 생성:**
        * 각 나무별 새순 길이 및 엽록소 수치의 시간에 따른 변화 추세(선형 회귀 기울기)를 계산하여 식물의 생장 패턴 및 건강 상태 변화를 나타내는 동적 특성 추가.
    * **군집화(Clustering) 특성 추가:**
        * K-평균 군집화 (K=3, 엘보우 플롯 기반 결정)를 활용하여 '수고(m)', '수관폭평균', '새순평균', '엽록소평균' 특성(로그 변환 후)을 기반으로 유사한 생육 특성을 가진 나무 그룹 생성.
        * 생성된 클러스터 레이블을 원-핫 인코딩하여 모델의 입력 특성으로 활용.
* **강건한 모델링 및 평가:**
    * **모델 선택:** RandomForestRegressor (주요 모델), XGBoost (비교 모델).
    * **하이퍼파라미터:**
        * RandomForest: `n_estimators=100, max_depth=10, random_state=42`
        * XGBoost: `n_estimators=150, max_depth=15, learning_rate=0.01, subsample=0.8, colsample_bytree=0.7, gamma=0.1, reg_alpha=0.1, reg_lambda=1, random_state=42`
    * **모델 평가:** StratifiedKFold 교차 검증 (5-fold)을 도입하여 모델 성능 평가의 신뢰성 확보 및 data leakage 문제 해결
* **특성 중요도 분석:** RandomForest 모델의 특성 중요도를 확인하여 예측에 큰 영향을 미치는 변수 파악.

#### 3.2. 결과

* **RandomForestRegressor (교차 검증 평균):** R² 약 0.9631
* **RandomForestRegressor (단일 테스트 분할):** **MSE 약 1324.60** (레거시 분석(data leakage 이전)의 기반 모델 대비 성능 약 5% 향상)
* **주요 특성 (Feature Importances):** '새순최대', '새순기울기', '새순표편' 등이 착과량 예측에 중요한 영향을 미치는 것으로 나타남.

---

## 4. 핵심 학습 및 성장

본 프로젝트의 개선 과정을 통해 다음과 같은 중요한 점들을 배우고 성장할 수 있었습니다.

* **데이터 전처리의 중요성:** 이상치 처리가 모델의 안정성과 예측 성능에 미치는 긍정적인 영향을 확인했습니다.
* **효과적인 특성 공학의 힘:** 단순 통계량을 넘어 시계열 데이터의 추세(기울기)를 추출하고, 군집화를 통해 데이터 내 숨겨진 패턴을 발견하여 새로운 특성을 생성하는 것이 모델 성능 향상에 크게 기여함을 경험했습니다. 이는 데이터에 대한 깊이 있는 이해를 바탕으로 이루어졌습니다.
* **모델 평가 방법론의 엄밀성:** 초기 분석에서 스태킹 모델 평가지표 해석(data leakage)의 오류를 인지하고, 교차 검증과 같은 강건한 평가 방법을 도입하여 모델의 일반화 성능을 신뢰성 있게 측정하는 것의 중요성을 깨달았습니다.
* **반복적 개선을 통한 문제 해결:** 이전 분석의 한계점을 명확히 파악하고, 이를 개선하기 위한 구체적인 전략(이상치 처리, 특성 공학 고도화, 평가 방법론 개선)을 수립하고 실행하여 실질적인 성능 향상을 이끌어내는 경험을 했습니다.

---

## 5. 파일 구성

* `착과량예측.ipynb`: 2023년에 수행한 초기 분석 과정 노트북.
* `착과량예측_vol2.ipynb`: 2025년에 수행한 개선된 분석 과정 노트북.
* `train.csv`: 학습 데이터.
* `test.csv`: 예측 대상 데이터.

---
