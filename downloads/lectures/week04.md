# 4주차 — 베이스라인 모델 만들기

**클라우드 MLOps** · 연성대학교 · 230분 (10분 조기 종료)
NCS: `2001070301_19v1.3` 인공지능 후보 모델 도출하기 / `2001070302_19v1.1` 인공지능 모델 기본 설계하기 / `2001070305_19v1.2` 인공지능 특징 생성하기(부분) / `2001070306_19v1.1` 인공지능 학습 알고리즘 선정하기 / `2001070306_19v1.4` 인공지능 학습하기 / `2001070307_19v1.1` 인공지능 모델 평가 기준 정하기

---

## 0. 회차 요약 (강사용 1페이지)

| 항목 | 내용 |
|---|---|
| 학습목표 | 학습/검증/테스트를 올바르게 나누고, scikit-learn `Pipeline`으로 전처리와 모델을 하나로 묶어 학습·평가한 뒤, 모델 파일을 S3에 업로드할 수 있다. 문제에 맞는 평가지표를 골라 그 이유를 설명할 수 있다. |
| 오늘의 결과물 | ① `s3://mlops-2026-<학번>/models/baseline_bike_YYYYMMDD.pkl` ② `metrics.json` (RMSE·MAE·R²) ③ 팀 데이터 베이스라인 성능 기록표 ④ **M1 문제정의서 최종본** |
| 사전 준비 | 공통 예제 `processed/bike_processed.csv`가 각 학생 버킷에 있는지 확인(3주차 미완주 학생용 강사 배포본 `s3://mlops-course-shared/processed/bike_processed.csv` 준비) · 저장소 태그 `week-04-done` 푸시 · `requirements-week04.txt` 배포 · M1 제출 폼(LMS) 개설 |
| 학생 준비물 | 노트북, 2주차에 만든 EC2 개발 서버(`mlops-dev-<학번>`), 3주차 전처리 산출물(`processed/`), 팀 문제정의서 초안 |
| 예상 사고 지점 | ① 3주차 전처리 결과가 없거나 컬럼명이 달라 스크립트가 `KeyError`로 죽음 ② `train_test_split` 전에 스케일링해서 데이터 누수 발생(성능이 비현실적으로 좋게 나옴) ③ S3 업로드 시 `AccessDenied` 또는 버킷명 오타 |

### 시간표

| 시간 | 구성 | 분 |
|---|---|---|
| 00:00–00:10 | 도입 — 지난주 복습 퀴즈 + 오늘 만들 결과물 데모 | 10 |
| 00:10–00:55 | 이론 — 데이터 분할 / 누수 / 평가지표 / 베이스라인 / Pipeline을 통째로 저장하는 이유 | 45 |
| 00:55–01:05 | 휴식 | 10 |
| 01:05–01:55 | **실습 A** — 공통 예제로 베이스라인 학습 → 저장 → S3 업로드 | 50 |
| 01:55–02:05 | 휴식 | 10 |
| 02:05–03:30 | **실습 B** — 팀 데이터 베이스라인 + **M1 문제정의서 최종화** | 85 |
| 03:30–03:45 | 체크포인트 제출 + M1 제출 + 다음 주 예고 | 15 |
| 03:45–03:50 | 리소스 정리 타임 (EC2 중지 확인) | 5 |
| **합계** | | **230** |

---

## 1. 도입 (10분)

### 지난주 복습 퀴즈 (구두 3문항)

1. S3 버킷 안에서 `raw/` 폴더의 파일은 왜 절대 수정하면 안 될까요? — (원본이 바뀌면 전처리 결과를 다시 만들 수 없고, 어떤 데이터로 만든 모델인지 추적이 불가능해집니다. `raw/`는 읽기 전용, 가공물은 항상 `processed/`에 새로 씁니다.)
2. 3주차에 만든 전처리 스크립트의 입력과 출력은 각각 S3의 어느 경로였나요? — (입력 `s3://mlops-2026-<학번>/raw/…`, 출력 `s3://mlops-2026-<학번>/processed/bike_processed.csv`)
3. 문제정의서에 반드시 들어가야 하는 다섯 가지는? — (주제 / 데이터 / 입력·출력 / 평가지표 / 성공 기준)

> 세 번째 질문은 오늘 M1 제출과 직결되므로, 답이 안 나오면 칠판에 다섯 항목을 적어두고 수업 내내 남겨둡니다.

### 오늘 만들 것 데모

강사는 다음 순서로 완성본을 3분 안에 보여줍니다. **설명하지 말고 결과만 보여주는 것이 목적**입니다.

1. 터미널에서 `python train_baseline.py --bucket mlops-2026-instructor` 를 실행합니다. 화면에 이런 로그가 흐릅니다.

```text
[1/6] 데이터 읽는 중: s3://mlops-2026-instructor/processed/bike_processed.csv
      행 8,760개 / 열 10개
[2/6] 학습/검증/테스트 분할 (60:20:20)
      train 5,256 / valid 1,752 / test 1,752
[3/6] 베이스라인(평균 예측) 학습
      dummy        RMSE=  1204.31  MAE=  945.12  R2=-0.001
[4/6] 후보 모델(RandomForest) 학습
      randomforest RMSE=   412.77  MAE=  268.45  R2= 0.883
[5/6] 모델 저장: baseline_bike_20260302.pkl (12.4 MB)
[6/6] S3 업로드 완료: s3://mlops-2026-instructor/models/baseline_bike_20260302.pkl
```

2. AWS 콘솔 → S3 → `models/` 폴더를 열어 `.pkl` 파일과 `metrics.json`이 올라간 것을 보여줍니다.
3. 마지막으로 한 문장만 던집니다.

> "오늘 여러분이 만들 것은 이 두 줄입니다. **RMSE 1204는 아무 생각 없이 평균만 찍은 값이고, 412는 모델을 쓴 값입니다.** 이 차이를 만들어내는 게 오늘 수업이고, 만약 이 두 숫자가 비슷하게 나온다면 그 모델은 쓸 이유가 없습니다."

---

## 2. 이론 (45분)

### 2-1. 학습 / 검증 / 테스트 — 왜 세 덩어리로 나누는가 (12분)

**강의 스크립트**

> 여러분이 기말고사를 봐야 하는데, 교수가 기출문제 100문제를 나눠줬다고 합시다. 여러분은 그걸 다 외웠습니다. 그리고 시험 당일, **기출문제에서 그대로 100문제가 나왔습니다.** 여러분 점수는 몇 점일까요? 100점이겠죠. 그런데 그 100점이 여러분의 실력일까요?
>
> 아니죠. 그건 **외운 걸 확인한 것**이지 실력을 잰 게 아닙니다. 모델도 똑같습니다. 모델을 학습시킨 데이터로 다시 평가하면, 그건 실력이 아니라 암기력을 잰 겁니다. 이걸 전문용어로 과적합(overfitting, 과하게 맞춰짐)이라고 부릅니다.
>
> 그래서 우리는 데이터를 처음부터 세 덩어리로 쪼갭니다. **학습(train)** 은 공부용 문제집입니다. 모델이 여기서 패턴을 배웁니다. **검증(validation)** 은 모의고사입니다. 여러 후보 중에 어느 모델이 나은지, 하이퍼파라미터를 얼마로 할지 여기서 정합니다. 그리고 **테스트(test)** 는 진짜 수능입니다. 딱 한 번만 봅니다.
>
> 여기서 제일 중요한 규칙 하나. 테스트 데이터를 보고 "어? 성능이 낮네, 모델을 바꿔볼까?" 하는 순간, 그 테스트 데이터는 더 이상 수능이 아니라 모의고사가 됩니다. 여러분의 머리를 통해서 테스트 정보가 모델에 새어 들어간 거니까요. 그래서 테스트 세트는 **맨 마지막에, 최종 보고용으로 딱 한 번** 씁니다.
>
> 질문 하나 드릴게요. 우리 따릉이 데이터는 2024년 1월부터 12월까지 시간별 대여량입니다. 이걸 무작위로 60:20:20으로 나누는 게 맞을까요, 아니면 1~8월은 학습, 9~10월은 검증, 11~12월은 테스트로 나누는 게 맞을까요?
>
> (학생 답을 받은 뒤) 정답은 **시간 순서대로 나누는 쪽**입니다. 실제 서비스에서 우리가 예측하려는 건 "미래"니까, 평가도 미래를 맞히는 상황으로 해야 정직합니다. 오늘 실습에서는 개념을 먼저 익히기 위해 무작위 분할로 시작하고, 시간 순 분할은 스크립트에 옵션으로 넣어뒀습니다. **시계열 데이터를 쓰는 팀은 반드시 시간 순 분할을 쓰세요.**

**판서/슬라이드 요점**

- train(공부) : valid(모의고사) : test(수능) = 60 : 20 : 20
- 테스트 세트는 **최종 1회만** — 보고 나서 모델을 고치면 그 순간 오염됨
- 시계열 데이터는 무작위 분할 금지 → **시간 순 분할**
- 분할은 **가장 먼저** 한다. 전처리보다 먼저.
- 재현성을 위해 `random_state=42` 고정

**학생 질문 예상 & 답변**

- Q: 데이터가 500행밖에 없는데도 셋으로 나눠야 하나요? → A: 그럴 땐 교차검증(cross validation)을 씁니다. 데이터를 5조각 내서 돌아가며 검증에 쓰는 방식이라 데이터를 아끼면서 검증할 수 있습니다. 다만 테스트 세트는 그래도 따로 떼어 두세요. 오늘 스크립트에 `--cv 5` 옵션을 넣어뒀습니다.
- Q: 검증 세트 없이 train/test 둘로만 나누면 안 되나요? → A: 후보가 하나뿐이고 튜닝을 전혀 안 한다면 가능합니다. 그런데 5주차에 하이퍼파라미터 5조합을 비교할 겁니다. 그 비교를 테스트로 하면 테스트가 오염되죠. 그래서 지금부터 셋으로 습관을 들입니다.
- Q: 비율은 꼭 60:20:20인가요? → A: 아닙니다. 데이터가 아주 많으면 80:10:10도 씁니다. 기준은 "검증·테스트 각각이 성능 차이를 구분할 만큼 충분한 개수인가"입니다. 수백 건 단위면 최소한의 신뢰가 생깁니다.

---

### 2-2. 데이터 누수(leakage) — 성능이 너무 좋으면 의심하라 (12분)

**강의 스크립트**

> 데이터 누수, 영어로 leakage라고 합니다. 말 그대로 **답이 새는 것**입니다. 학생이 시험지 답안을 미리 본 상태로 시험을 보는 겁니다. 결과는? 점수가 아주 잘 나옵니다. 그런데 실전에 내보내면 처참하게 무너집니다.
>
> 실무에서 제일 무서운 게 이겁니다. 버그는 에러 메시지를 띄워주지만, 누수는 **"축하합니다, 정확도 99%"** 라고 띄워주거든요. 그래서 저는 여러분에게 이 문장을 외우게 하고 싶습니다. **"성능이 이상하게 좋으면, 기뻐하기 전에 누수를 의심한다."**
>
> 누수는 크게 두 가지 얼굴로 옵니다.
>
> 첫 번째, **피처에 답이 섞여 들어간 경우**입니다. 예를 들어 우리가 시간별 따릉이 대여량을 예측하는데, 피처에 "그날 하루 총 대여량"이 들어 있다고 합시다. 하루 총합에는 지금 맞히려는 그 시간의 대여량이 이미 포함돼 있죠. 답을 보고 답을 맞히는 겁니다. 또 다른 예로, 고객 이탈 예측을 하는데 피처에 "해지 신청일"이 들어 있는 경우. 해지 신청을 했으면 이미 이탈이잖아요. 이런 컬럼을 실무에서는 아주 흔하게 만납니다.
>
> 두 번째, **전처리를 분할 전에 해버린 경우**입니다. 이게 오늘 여러분이 실제로 저지를 실수입니다. 스케일링을 한다고 합시다. `StandardScaler`는 평균과 표준편차를 계산해서 빼고 나눕니다. 그런데 전체 데이터로 평균을 구한 다음에 train/test를 나누면, **테스트 데이터의 평균 정보가 학습 데이터 스케일링에 이미 반영된 것**입니다. 미래 정보가 과거로 흘러 들어간 거죠. 결측치를 전체 평균으로 채우는 것도 똑같은 문제입니다.
>
> 그럼 어떻게 막을까요? 여기가 오늘 실습의 핵심입니다. **전처리를 `Pipeline` 안에 집어넣으면 됩니다.** Pipeline에 넣으면 `fit`은 학습 데이터에만 적용되고, 검증·테스트에는 `transform`만 적용됩니다. 사람이 순서를 지키려고 애쓰는 대신, **구조적으로 실수할 수 없게 만드는 것**입니다.

다음 두 코드를 나란히 띄워 놓고 비교합니다.

```python
# ✗ 틀린 방법 — 전체 데이터로 스케일러를 학습시킨 뒤 나눈다 (누수 발생)
from sklearn.preprocessing import StandardScaler
from sklearn.model_selection import train_test_split

scaler = StandardScaler()
X_scaled = scaler.fit_transform(X)          # 전체 데이터의 평균·표준편차 사용
X_tr, X_te, y_tr, y_te = train_test_split(X_scaled, y, random_state=42)
```

```python
# ✅ 올바른 방법 — 먼저 나누고, 전처리는 Pipeline 안에서 학습 데이터에만 fit
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import StandardScaler
from sklearn.ensemble import RandomForestRegressor
from sklearn.model_selection import train_test_split

X_tr, X_te, y_tr, y_te = train_test_split(X, y, random_state=42)

pipe = Pipeline([
    ("scale", StandardScaler()),
    ("model", RandomForestRegressor(random_state=42)),
])
pipe.fit(X_tr, y_tr)        # 스케일러의 평균·표준편차는 X_tr에서만 계산됨
pipe.score(X_te, y_te)      # X_te에는 transform만 적용됨
```

**판서/슬라이드 요점**

- 누수 = 학습 시점에 알 수 없어야 할 정보가 피처/전처리에 섞임
- 얼굴 ①: 답이 들어 있는 컬럼 (총합, 해지일, 사후 집계값)
- 얼굴 ②: 분할 전에 `fit` 한 스케일러·결측 대체·인코더
- 방어 수단: **분할 먼저 → 전처리는 Pipeline 안에서**
- 경고 신호: R² 0.99, 정확도 100%, 검증과 테스트 성능이 똑같이 완벽 → 무조건 컬럼부터 다시 본다

**학생 질문 예상 & 답변**

- Q: 결측치를 채우는 것도 누수인가요? → A: 채우는 값을 **전체 데이터에서 계산했다면** 누수입니다. 학습 데이터의 중앙값으로 채우고 그 값을 그대로 테스트에 적용하면 괜찮습니다. `SimpleImputer`를 Pipeline에 넣으면 자동으로 그렇게 됩니다.
- Q: 원-핫 인코딩도요? → A: 네. 테스트에만 있는 범주를 미리 알고 인코딩하면 누수입니다. `OneHotEncoder(handle_unknown="ignore")`를 Pipeline에 넣어서, 처음 보는 값은 전부 0으로 처리하게 하는 게 정석입니다.
- Q: 누수인지 아닌지 헷갈릴 땐 어떻게 판단하나요? → A: 한 문장으로 자문하세요. **"예측을 실제로 해야 하는 그 순간에, 이 값을 알 수 있는가?"** 모르면 누수입니다. 따릉이 오후 3시 대여량을 오후 2시에 예측한다면, "그날 총 대여량"은 오후 2시에 알 수 없죠. 누수입니다.

---

### 2-3. 평가지표 고르기 — 문제가 지표를 정한다 (11분)

**강의 스크립트**

> "정확도 몇 퍼센트예요?" 이 질문, 실무에서 하면 곤란해지는 경우가 많습니다. 왜냐하면 **정확도가 의미 없는 문제가 아주 많거든요.**
>
> 예를 들어봅시다. 제조 공정에서 불량품을 찾아내는 모델을 만든다고 합시다. 실제 불량률이 1%입니다. 제가 아주 훌륭한 모델을 만들어 왔습니다. 코드는 이렇습니다. `return "정상"`. 끝입니다. 무조건 정상이라고 대답합니다. 이 모델의 정확도는 몇 퍼센트일까요? **99%입니다.** 그런데 이 모델은 불량품을 단 하나도 못 잡습니다. 쓸모가 0입니다.
>
> 그래서 분류 문제에서는 정밀도(precision)와 재현율(recall)을 봅니다. 정밀도는 "불량이라고 한 것 중에 진짜 불량이 몇 %인가", 재현율은 "진짜 불량 중에 몇 %를 잡아냈나"입니다. 이 둘을 하나로 합친 게 **F1 점수**입니다. 데이터가 불균형하면 정확도 대신 F1을 보세요.
>
> 그런데 여기서 한 걸음 더 나가야 합니다. 정밀도와 재현율 중에 뭐가 더 중요한지는 **비용이 정합니다.** 암 진단 모델을 생각해 보세요. 암인데 아니라고 하는 실수(놓침)와, 암이 아닌데 암이라고 하는 실수(오경보) 중 어느 쪽이 더 치명적일까요? 놓치는 쪽이죠. 그러니 재현율을 올려야 합니다. 반대로 스팸 메일 필터라면? 중요한 메일이 스팸함에 들어가는 게 더 치명적이니 정밀도를 올려야 합니다. **지표 선택은 기술 문제가 아니라 비즈니스 문제입니다.**
>
> 회귀 문제로 가봅시다. 우리 따릉이 예측 같은 거죠. 여기선 RMSE와 MAE를 씁니다. MAE는 그냥 오차의 절댓값 평균입니다. "평균적으로 268대 틀린다" — 해석이 아주 쉽죠. RMSE는 오차를 제곱해서 평균 내고 루트를 씌웁니다. 제곱을 하니까 **크게 틀린 케이스에 벌점이 크게 붙습니다.**
>
> 그럼 언제 뭘 쓸까요? 자, 여러분이 따릉이 운영자라고 합시다. 100대 예측을 90대로 틀리는 것보다, 3000대 예측을 1000대로 틀리는 게 훨씬 큰 사고죠? 자전거가 없어서 사람들이 못 타니까요. **큰 실수를 특히 피하고 싶으면 RMSE**, 모든 실수를 똑같이 세고 싶으면 MAE입니다.
>
> 그리고 마지막으로, 지표는 **하나만 고르세요.** 여러 개를 계산해서 보는 건 좋지만, "이 모델이 저 모델보다 낫다"를 판정하는 대표 지표는 반드시 하나여야 합니다. 두 개면 서로 다른 답을 낼 때 싸움만 납니다. 이걸 실무에서 주 지표(primary metric)라고 부릅니다. **여러분의 M1 문제정의서에 주 지표를 하나 적어 오세요.**

**판서/슬라이드 요점**

| 문제 유형 | 상황 | 주 지표 | 이유 |
|---|---|---|---|
| 분류 | 클래스 균형 | 정확도(Accuracy) | 직관적, 해석 쉬움 |
| 분류 | 불균형 (불량·이탈·사기) | F1 | 정확도가 무의미해짐 |
| 분류 | 놓치면 큰일 (질병·고장) | 재현율(Recall) | 미검출 비용이 큼 |
| 분류 | 오경보가 큰일 (스팸·마케팅) | 정밀도(Precision) | 오탐 비용이 큼 |
| 회귀 | 큰 오차가 특히 치명적 | RMSE | 제곱해서 큰 오차에 가중 |
| 회귀 | 모든 오차가 동등 | MAE | 단위가 그대로라 설명 쉬움 |

- R²는 "얼마나 잘 설명하나"의 참고 지표. 단위가 없어 팀 간 비교엔 좋지만 **주 지표로는 비추천**
- 주 지표는 **1개**. 나머지는 보조 지표.
- 지표 옆에 **성공 기준 숫자**를 반드시 적는다 → "RMSE 450 이하"

**학생 질문 예상 & 답변**

- Q: RMSE 412가 좋은 건가요 나쁜 건가요? → A: 그 자체로는 알 수 없습니다. 대여량 평균이 3000대인데 412 틀린다면 괜찮고, 평균이 500대인데 412 틀린다면 형편없죠. 그래서 **항상 베이스라인과 비교**해야 합니다. 오늘 dummy가 1204였으니 412는 의미 있는 개선입니다.
- Q: 정확도 90%면 성공 기준으로 적어도 되나요? → A: 데이터의 클래스 비율을 먼저 확인하세요. 90:10 데이터면 아무것도 안 해도 90%입니다. 성공 기준은 **"아무것도 안 했을 때보다 얼마나 나은가"** 로 잡아야 합니다.
- Q: 지표를 나중에 바꿔도 되나요? → A: 바꿀 수는 있지만 **바꾼 시점과 이유를 반드시 기록**해야 합니다. 지표를 슬쩍 바꿔 좋아 보이게 만드는 건 실무에서 가장 신뢰를 잃는 행동입니다.

---

### 2-4. 베이스라인의 의미 — "이보다 못하면 쓸 이유가 없다" (5분)

**강의 스크립트**

> 베이스라인(baseline)이라는 말, 우리말로 하면 **기준선**입니다. 육상 트랙의 출발선 같은 겁니다.
>
> 회귀 문제에서 가장 단순한 베이스라인은 뭘까요? **그냥 평균값을 찍는 것**입니다. 따릉이 대여량이 뭐가 됐든 무조건 "평균 1500대"라고 대답하는 모델이요. scikit-learn에는 이걸 위한 `DummyRegressor`가 있습니다. 분류에는 `DummyClassifier`가 있고요, 무조건 다수 클래스를 찍습니다.
>
> 왜 이런 바보 같은 모델을 일부러 만들까요? **기준선이 없으면 여러분의 점수가 좋은 건지 나쁜 건지 알 수 없기 때문입니다.** 여러분이 3주 동안 튜닝해서 RMSE 1100을 만들었는데, 평균만 찍어도 1204가 나온다면? 3주를 날린 겁니다. 이 사실을 3주 뒤가 아니라 **오늘 5분 만에** 알 수 있게 해주는 게 베이스라인입니다.
>
> 그래서 오늘 우리 스크립트는 항상 두 개를 학습합니다. 하나는 dummy, 하나는 진짜 모델. 그리고 **두 숫자를 나란히 출력**합니다. 이 습관을 끝까지 가져가세요.
>
> 하나 더. 베이스라인은 "출발선"이면서 동시에 **"오늘 안에 반드시 완성해야 하는 최소 결과물"** 이기도 합니다. 실무에서도 마찬가지예요. 화려한 모델을 3주 만들다가 아무것도 못 내놓는 것보다, 첫날 단순한 모델로 파이프라인을 끝까지 관통시켜 놓는 게 훨씬 낫습니다. 우리 과목이 15주 내내 하려는 게 딱 그겁니다.

**판서/슬라이드 요점**

- 베이스라인 = 성능의 최저 기준선. 회귀는 `DummyRegressor(strategy="mean")`, 분류는 `DummyClassifier(strategy="most_frequent")`
- 항상 **dummy 성능과 나란히** 보고한다
- 개선 폭 = (dummy 지표 − 모델 지표) → 이게 진짜 성과
- 베이스라인은 결승선이 아니라 출발선. 하지만 **오늘 반드시 통과해야 하는 선**

**학생 질문 예상 & 답변**

- Q: RandomForest 말고 다른 모델 써도 되나요? → A: 됩니다. 오늘의 목적은 최고 성능이 아니라 **"학습→평가→저장→업로드"라는 관통 경험**입니다. 모델 비교는 5주차 MLflow에서 제대로 합니다.
- Q: 성능이 dummy보다 나쁘게 나왔어요. → A: 아주 좋은 발견입니다. 대개 세 가지입니다. ① 피처에 신호가 없다 ② 타깃을 잘못 잡았다 ③ 전처리에서 데이터를 망가뜨렸다. 순서대로 확인해 보고, 그래도 안 되면 손 들어 주세요.

---

### 2-5. 왜 모델이 아니라 Pipeline을 통째로 저장하는가 (5분)

> **이 소절은 7주차 "학습-서빙 스큐"의 씨앗입니다. 반드시 짚고 넘어갑니다.**

**강의 스크립트**

> 자, 오늘 마지막 개념입니다. 그런데 저는 이게 오늘 수업에서 **제일 중요한 5분**이라고 생각합니다.
>
> 상상해 봅시다. 여러분이 모델을 잘 학습시켰습니다. `joblib`으로 `model.pkl` 저장했습니다. 3주 뒤 7주차에 API를 만듭니다. 사용자가 `{"temp": 21.5, "hour": 15, "weekday": "Mon"}` 이런 JSON을 보냅니다. 여러분의 API가 `model.predict(...)` 를 호출합니다. 그런데... 에러가 납니다. 왜일까요?
>
> 모델은 숫자만 먹습니다. `"Mon"` 같은 문자열은 못 먹어요. 학습할 때는 원-핫 인코딩을 해서 숫자로 바꿨었죠. 그리고 스케일링도 했었고, 결측치도 채웠고요. **그 전처리 과정을 서빙 코드에서 똑같이 다시 짜야 합니다.**
>
> 여기서 사고가 납니다. 학습 때 쓴 스케일러의 평균이 20.3이었는데, 서빙 코드에서 다시 계산한 평균은 21.1입니다. 학습 때 요일 순서가 `[Fri, Mon, Sat, ...]` 알파벳순이었는데, 서빙에서는 `[Mon, Tue, Wed, ...]` 순으로 만들었습니다. 컬럼 순서가 다릅니다. **에러는 안 납니다.** 예측은 나옵니다. 그런데 틀린 값이 나옵니다. 이걸 **학습-서빙 스큐(training-serving skew, 학습과 서빙의 어긋남)** 라고 부릅니다. 실무에서 원인 찾는 데 며칠씩 걸리는, 아주 악명 높은 버그입니다.
>
> 해결책은 놀랄 만큼 단순합니다. **전처리와 모델을 하나로 묶어서, 그 묶음을 통째로 저장하면 됩니다.** 그게 `Pipeline`입니다. `joblib.dump(pipe, "model.pkl")` 하면 스케일러의 평균, 인코더의 범주 목록, 컬럼 순서까지 전부 파일 안에 들어갑니다. 서빙에서는 `pipe.predict(원본_DataFrame)` 한 줄이면 끝납니다. 전처리 코드를 다시 짤 일이 없으니 어긋날 일도 없습니다.
>
> 그래서 오늘 규칙은 이겁니다. **저장하는 것은 모델이 아니라 Pipeline이다.** 7주차에 여러분이 이 문장에 고마워하게 될 겁니다.

**판서/슬라이드 요점**

- 모델만 저장 → 서빙에서 전처리를 다시 구현 → **어긋남(스큐)** 발생 → 에러 없이 틀린 예측
- `Pipeline` 저장 = 전처리 규칙(평균·표준편차·범주 목록·컬럼 순서) + 모델 가중치를 한 파일에
- 서빙 코드는 `pipe.predict(df)` 한 줄로 끝
- **7주차 예고**: FastAPI에서 이 `.pkl` 하나만 로드해 `/predict`를 만든다

**학생 질문 예상 & 답변**

- Q: `.pkl` 파일이 12MB나 되는데 정상인가요? → A: 정상입니다. RandomForest는 트리를 통째로 담아서 커집니다. Ridge 같은 선형 모델이면 수십 KB입니다. 100MB를 넘어가면 트리 개수(`n_estimators`)나 깊이를 줄이세요. 나중에 컨테이너 이미지에 넣어야 하니까요.
- Q: pkl 파일은 다른 컴퓨터에서도 열리나요? → A: **scikit-learn 버전이 같아야 안전합니다.** 그래서 `requirements.txt`로 버전을 고정합니다. 6주차에 Docker를 배우는 이유이기도 합니다. 환경을 통째로 옮기는 거죠.

---

## 3. 실습 A — 공통 예제로 베이스라인 만들기 (50분) · 공통 예제 따라하기

**목표** 3주차 `processed/` 데이터를 입력으로 `Pipeline` 기반 베이스라인 모델을 학습·평가하고, `.pkl`과 `metrics.json`을 S3 `models/`에 업로드한다.

**사전 배포 파일**

| 파일 | 내용 | 배포 방식 |
|---|---|---|
| `requirements-week04.txt` | 패키지 버전 고정 | LMS 다운로드 |
| `train_baseline.py` | 학습 스크립트 **뼈대**(핵심 함수 2개는 비어 있음) | 저장소 `week-04-start` 태그 |
| `predict_sample.py` | 저장된 모델 재로드·예측 확인 | 저장소 `week-04-start` 태그 |
| `bike_processed.csv` | 3주차 산출물이 없는 학생용 백업 | `s3://mlops-course-shared/processed/` |

### 수행 순서

**① EC2 개발 서버 시작 및 접속 (5분)**

AWS 콘솔 → EC2 → 인스턴스 `mlops-dev-<학번>` 선택 → 인스턴스 상태 → **인스턴스 시작**. 상태가 `실행 중`이 되면 퍼블릭 IPv4 주소를 복사합니다. (중지했다 켜면 IP가 바뀝니다. 2주차 과제에서 확인한 내용입니다.)

```bash
# <키파일경로>: 2주차에 다운로드한 .pem 파일 경로 (예: ~/keys/mlops-2026-20251234.pem)
# <퍼블릭IP>: 방금 복사한 EC2 퍼블릭 IPv4 주소
ssh -i <키파일경로> ubuntu@<퍼블릭IP>
```

접속 후 2주차에 만든 가상환경을 켭니다.

```bash
cd ~/mlops
source .venv/bin/activate
```

- 확인 포인트: 프롬프트 맨 앞에 `(.venv)` 가 붙으면 성공입니다.

**② 실습 파일 내려받기 및 패키지 설치 (5분)**

```bash
cd ~/mlops
git fetch --all --tags
git checkout week-04-start
cat > requirements-week04.txt << 'EOF'
pandas==2.2.3
numpy==2.1.3
scikit-learn==1.5.2
joblib==1.4.2
boto3==1.35.54
EOF
pip install -r requirements-week04.txt
python -c "import sklearn; print('scikit-learn', sklearn.__version__)"
```

- 확인 포인트: `scikit-learn 1.5.2` 가 출력되면 성공입니다. **버전이 다르면 나중에 `.pkl`을 못 여는 사고가 납니다. 반드시 맞추세요.**

**③ 입력 데이터 확인 (5분)**

```bash
# <학번>: 본인 학번 (예: 20251234)
aws s3 ls s3://mlops-2026-<학번>/processed/ --region ap-northeast-2
```

- 확인 포인트: `bike_processed.csv` 가 보이면 성공입니다.
- 3주차 산출물이 없다면 아래로 백업본을 복사해 옵니다.

```bash
aws s3 cp s3://mlops-course-shared/processed/bike_processed.csv \
          s3://mlops-2026-<학번>/processed/bike_processed.csv \
          --region ap-northeast-2
```

컬럼 구성을 눈으로 확인합니다.

```bash
python - << 'EOF'
import io, boto3, pandas as pd
BUCKET = "mlops-2026-<학번>"   # 본인 버킷명으로 수정
s3 = boto3.client("s3", region_name="ap-northeast-2")
obj = s3.get_object(Bucket=BUCKET, Key="processed/bike_processed.csv")
df = pd.read_csv(io.BytesIO(obj["Body"].read()))
print(df.shape)
print(df.dtypes)
print(df.head())
EOF
```

- 확인 포인트: `rent_count`(타깃), `temp`, `humidity`, `windspeed`, `rainfall`, `hour`, `weekday`, `season`, `holiday` 컬럼이 있어야 합니다.

**④ 학습 스크립트 완성 (15분)**

`train_baseline.py`를 엽니다. `build_pipeline()`과 `split_data()` 두 함수가 `pass`로 비어 있습니다. **이 두 함수만 직접 입력합니다.** 나머지는 이미 채워져 있습니다.

```bash
nano train_baseline.py     # 또는 vim, VS Code Remote
```

완성본 전체는 아래와 같습니다. (강사는 이 파일을 화면에 띄워 놓고, 빈 두 함수만 학생이 치게 합니다.)

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-
"""
4주차 실습 A — 따릉이 시간별 대여량 베이스라인 모델
사용법:
    python train_baseline.py --bucket mlops-2026-<학번>
    python train_baseline.py --bucket mlops-2026-<학번> --time-split   # 시간 순 분할
"""
import argparse
import io
import json
import os
from datetime import datetime

import boto3
import joblib
import numpy as np
import pandas as pd
from sklearn.compose import ColumnTransformer
from sklearn.dummy import DummyRegressor
from sklearn.ensemble import RandomForestRegressor
from sklearn.impute import SimpleImputer
from sklearn.metrics import mean_absolute_error, mean_squared_error, r2_score
from sklearn.model_selection import train_test_split
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import OneHotEncoder, StandardScaler

REGION = "ap-northeast-2"
NUM_COLS = ["temp", "humidity", "windspeed", "rainfall"]
CAT_COLS = ["hour", "weekday", "season", "holiday"]
TARGET = "rent_count"
SEED = 42


def load_data(bucket: str, key: str) -> pd.DataFrame:
    print(f"[1/6] 데이터 읽는 중: s3://{bucket}/{key}")
    s3 = boto3.client("s3", region_name=REGION)
    obj = s3.get_object(Bucket=bucket, Key=key)
    df = pd.read_csv(io.BytesIO(obj["Body"].read()))
    print(f"      행 {len(df):,}개 / 열 {len(df.columns)}개")
    missing = [c for c in NUM_COLS + CAT_COLS + [TARGET] if c not in df.columns]
    if missing:
        raise SystemExit(f"[중단] 필요한 컬럼이 없습니다: {missing}\n"
                         f"       현재 컬럼: {list(df.columns)}")
    return df


# ───────── 학생이 직접 입력하는 부분 ① ─────────
def split_data(df: pd.DataFrame, time_split: bool = False):
    """학습 60% / 검증 20% / 테스트 20% 로 나눈다."""
    print("[2/6] 학습/검증/테스트 분할 (60:20:20)")
    X = df[NUM_COLS + CAT_COLS]
    y = df[TARGET]

    if time_split:
        # 시계열: 순서를 유지한 채 앞에서부터 자른다
        n = len(df)
        i1, i2 = int(n * 0.6), int(n * 0.8)
        X_tr, X_val, X_te = X.iloc[:i1], X.iloc[i1:i2], X.iloc[i2:]
        y_tr, y_val, y_te = y.iloc[:i1], y.iloc[i1:i2], y.iloc[i2:]
    else:
        X_tr, X_tmp, y_tr, y_tmp = train_test_split(
            X, y, test_size=0.4, random_state=SEED)
        X_val, X_te, y_val, y_te = train_test_split(
            X_tmp, y_tmp, test_size=0.5, random_state=SEED)

    print(f"      train {len(X_tr):,} / valid {len(X_val):,} / test {len(X_te):,}")
    return X_tr, X_val, X_te, y_tr, y_val, y_te


# ───────── 학생이 직접 입력하는 부분 ② ─────────
def build_pipeline(model):
    """전처리 + 모델을 하나의 Pipeline으로 묶는다. (7주차 서빙에서 이 묶음을 그대로 쓴다)"""
    numeric = Pipeline([
        ("impute", SimpleImputer(strategy="median")),
        ("scale", StandardScaler()),
    ])
    categorical = Pipeline([
        ("impute", SimpleImputer(strategy="most_frequent")),
        ("onehot", OneHotEncoder(handle_unknown="ignore", sparse_output=False)),
    ])
    preprocess = ColumnTransformer([
        ("num", numeric, NUM_COLS),
        ("cat", categorical, CAT_COLS),
    ])
    return Pipeline([("pre", preprocess), ("model", model)])


def evaluate(name: str, pipe, X, y) -> dict:
    pred = pipe.predict(X)
    scores = {
        "rmse": float(np.sqrt(mean_squared_error(y, pred))),
        "mae": float(mean_absolute_error(y, pred)),
        "r2": float(r2_score(y, pred)),
    }
    print(f"      {name:12s} RMSE={scores['rmse']:8.2f}  "
          f"MAE={scores['mae']:8.2f}  R2={scores['r2']:6.3f}")
    return scores


def main():
    ap = argparse.ArgumentParser()
    ap.add_argument("--bucket", required=True, help="본인 S3 버킷명 (예: mlops-2026-20251234)")
    ap.add_argument("--key", default="processed/bike_processed.csv")
    ap.add_argument("--time-split", action="store_true", help="시계열이면 이 옵션을 켠다")
    ap.add_argument("--n-estimators", type=int, default=200)
    ap.add_argument("--max-depth", type=int, default=12)
    args = ap.parse_args()

    df = load_data(args.bucket, args.key)
    X_tr, X_val, X_te, y_tr, y_val, y_te = split_data(df, args.time_split)

    print("[3/6] 베이스라인(평균 예측) 학습")
    dummy = build_pipeline(DummyRegressor(strategy="mean"))
    dummy.fit(X_tr, y_tr)
    dummy_val = evaluate("dummy", dummy, X_val, y_val)

    print("[4/6] 후보 모델(RandomForest) 학습")
    rf = build_pipeline(RandomForestRegressor(
        n_estimators=args.n_estimators,
        max_depth=args.max_depth,
        random_state=SEED,
        n_jobs=-1,
    ))
    rf.fit(X_tr, y_tr)
    rf_val = evaluate("randomforest", rf, X_val, y_val)

    # 테스트 세트는 최종 보고용으로 딱 한 번만 본다
    rf_test = evaluate("(test) rf", rf, X_te, y_te)

    stamp = datetime.now().strftime("%Y%m%d_%H%M")
    model_name = f"baseline_bike_{stamp}.pkl"
    print(f"[5/6] 모델 저장: {model_name}")
    joblib.dump(rf, model_name)          # ★ 모델이 아니라 Pipeline 전체를 저장한다
    size_mb = os.path.getsize(model_name) / 1024 / 1024

    metrics = {
        "created_at": datetime.now().isoformat(timespec="seconds"),
        "model_file": model_name,
        "model_size_mb": round(size_mb, 2),
        "sklearn_version": __import__("sklearn").__version__,
        "split": "time" if args.time_split else "random",
        "params": {"n_estimators": args.n_estimators, "max_depth": args.max_depth},
        "baseline_valid": dummy_val,
        "model_valid": rf_val,
        "model_test": rf_test,
        "improvement_rmse": round(dummy_val["rmse"] - rf_val["rmse"], 2),
    }
    with open("metrics.json", "w", encoding="utf-8") as f:
        json.dump(metrics, f, ensure_ascii=False, indent=2)

    s3 = boto3.client("s3", region_name=REGION)
    s3.upload_file(model_name, args.bucket, f"models/{model_name}")
    s3.upload_file("metrics.json", args.bucket, f"models/metrics_{stamp}.json")
    print(f"[6/6] S3 업로드 완료: s3://{args.bucket}/models/{model_name}")
    print(f"      개선 폭(RMSE): {metrics['improvement_rmse']}  ← 이 숫자가 0 이하면 모델을 쓸 이유가 없다")


if __name__ == "__main__":
    main()
```

**⑤ 실행 및 결과 확인 (5분)**

```bash
python train_baseline.py --bucket mlops-2026-<학번>
```

- 확인 포인트: `[6/6] S3 업로드 완료` 와 `개선 폭(RMSE)` 이 양수로 출력되면 성공입니다.

```bash
cat metrics.json
aws s3 ls s3://mlops-2026-<학번>/models/ --region ap-northeast-2
```

**⑥ 저장한 Pipeline을 다시 불러 예측해 보기 (10분)**

여기서 오늘 이론 2-5를 손으로 확인합니다. **원본 형태의 DataFrame을 그대로 넣어도 예측이 나온다**는 것이 핵심입니다.

```bash
cat > predict_sample.py << 'EOF'
#!/usr/bin/env python3
# -*- coding: utf-8 -*-
"""저장한 Pipeline을 다시 로드해 원본 형태 입력으로 예측한다."""
import argparse, io, boto3, joblib, pandas as pd

REGION = "ap-northeast-2"
ap = argparse.ArgumentParser()
ap.add_argument("--bucket", required=True)
ap.add_argument("--key", required=True, help="예: models/baseline_bike_20260302_1410.pkl")
args = ap.parse_args()

s3 = boto3.client("s3", region_name=REGION)
obj = s3.get_object(Bucket=args.bucket, Key=args.key)
pipe = joblib.load(io.BytesIO(obj["Body"].read()))

# 전처리를 하나도 하지 않은, 사람이 읽을 수 있는 원본 형태의 입력
sample = pd.DataFrame([
    {"temp": 21.5, "humidity": 55, "windspeed": 2.1, "rainfall": 0.0,
     "hour": 18, "weekday": "Mon", "season": "spring", "holiday": 0},
    {"temp": -3.0, "humidity": 80, "windspeed": 5.4, "rainfall": 1.2,
     "hour": 3,  "weekday": "Sun", "season": "winter", "holiday": 1},
])
print(sample)
print("예측 대여량:", pipe.predict(sample).round(1))
EOF

python predict_sample.py --bucket mlops-2026-<학번> --key models/<위에서_출력된_파일명>
```

- 확인 포인트: `"Mon"`, `"spring"` 같은 **문자열을 그대로 넣었는데 예측 숫자가 나옵니다.** 강사는 여기서 "이게 Pipeline을 통째로 저장한 대가입니다. 7주차 API도 이 코드 그대로입니다"라고 한 번 더 짚습니다.

### ⚠ 여기서 막히면

| 증상 | 원인 | 조치 |
|---|---|---|
| `KeyError: 'rent_count'` 또는 `[중단] 필요한 컬럼이 없습니다` | 3주차 전처리 결과의 컬럼명이 스크립트와 다름 | 출력된 "현재 컬럼" 목록을 보고 `NUM_COLS`/`CAT_COLS`/`TARGET` 상수를 본인 데이터에 맞게 수정. 급하면 `mlops-course-shared` 백업본으로 교체 |
| `botocore.exceptions.ClientError: An error occurred (AccessDenied)` | 버킷명 오타 또는 다른 학생 버킷 지정 | `aws s3 ls` 로 본인 버킷명을 다시 확인. `mlops-2026-<학번>` 형식 |
| `NoCredentialsError` | EC2에 자격 증명이 없음 | `aws configure list` 로 확인 → 없으면 `aws configure` 재실행(리전 `ap-northeast-2`). 인스턴스 역할을 쓰는 경우 EC2 → 보안 → IAM 역할 수정 |
| `TypeError: OneHotEncoder.__init__() got an unexpected keyword argument 'sparse_output'` | scikit-learn 1.2 미만 | `pip install -U scikit-learn==1.5.2` 후 재실행 |
| R²가 0.999 이상, RMSE가 한 자릿수 | 데이터 누수 (타깃이 피처에 섞임) | `NUM_COLS`/`CAT_COLS` 에 합계·비율 등 타깃 파생 컬럼이 있는지 확인해 제거 |
| 학습이 3분 넘게 안 끝남 | `n_estimators` 과다 또는 t3.medium 메모리 부족 | `--n-estimators 100 --max-depth 8` 로 재실행. `free -h` 로 메모리 확인 |
| `MemoryError` / SSH가 끊김 | 메모리 부족으로 프로세스가 죽음 | 데이터 샘플링(`df.sample(20000, random_state=42)`) 후 재시도. 그래도 안 되면 조교 호출 |
| `ssh: connect to host ... Connection timed out` | 인스턴스 중지 상태거나 IP가 바뀜 | 콘솔에서 인스턴스 시작 후 **새 퍼블릭 IP**로 접속. 학교망 22번 차단 시 EC2 Instance Connect 사용 |

### 컷오프 안내

50분 경과 시 강제 종료합니다. ⑥번까지 못 온 학생은 아래 명령으로 완성본을 받아 실습 B로 넘어갑니다.

```bash
cd ~/mlops
git stash            # 작업 중이던 내용 임시 보관
git checkout week-04-done
python train_baseline.py --bucket mlops-2026-<학번>
```

---

## 4. 실습 B — 팀 데이터 베이스라인 + M1 문제정의서 최종화 (85분) · 각자/팀이 직접 수행

**목표** 실습 A의 스크립트 골격을 팀 데이터에 이식해 베이스라인 성능을 기록하고, 그 숫자를 근거로 M1 문제정의서의 평가지표·성공 기준을 확정한다.

**과제 지시문** (학생에게 그대로 읽어줍니다)

> "지금부터 85분입니다. 앞의 40분은 코드, 뒤의 45분은 문서라고 생각하세요.
> 먼저 `train_baseline.py`를 팀 저장소로 복사하고, 파일 맨 위의 `NUM_COLS`, `CAT_COLS`, `TARGET` 세 줄만 여러분 데이터에 맞게 고치세요. **구조는 절대 건드리지 마세요.** 분류 문제인 팀은 아래에 드린 분류용 교체 블록을 그대로 붙여 넣으면 됩니다.
> 그다음 실행해서 숫자 두 개를 받아 적으세요. 하나는 dummy, 하나는 여러분 모델. 이 두 숫자가 오늘 여러분이 손에 넣어야 할 전부입니다.
> 그 숫자를 손에 쥔 다음에 문제정의서를 엽니다. **성공 기준을 오늘 나온 dummy 숫자를 기준으로 다시 쓰세요.** '정확도 90% 목표' 같은 근거 없는 문장은 오늘 이후로는 인정하지 않습니다. '베이스라인 RMSE 1204 대비 30% 이상 개선(846 이하)' — 이런 문장이어야 합니다.
> 3시 30분에 M1을 제출합니다. 다음 주 첫 15분에 팀별로 3분씩 발표합니다. 발표 슬라이드는 필요 없고, 문제정의서를 화면에 띄워 놓고 말로 설명하면 됩니다."

### 수행 항목

**1. 팀 데이터로 베이스라인 실행 (40분)**

```bash
cd ~/mlops
cp train_baseline.py train_team.py
nano train_team.py
```

수정할 곳은 딱 세 줄입니다.

```python
NUM_COLS = ["<숫자형_컬럼1>", "<숫자형_컬럼2>", ...]     # 예: ["age", "tenure", "monthly_charge"]
CAT_COLS = ["<범주형_컬럼1>", "<범주형_컬럼2>", ...]     # 예: ["gender", "contract_type"]
TARGET   = "<맞히려는_컬럼>"                            # 예: "churn"
```

**분류 문제인 팀**은 아래 블록으로 import·모델·평가 부분을 교체합니다. (그대로 복사해 쓰면 됩니다.)

```python
# ── 분류 문제용 교체 블록 ──────────────────────────────────
from sklearn.dummy import DummyClassifier
from sklearn.ensemble import RandomForestClassifier
from sklearn.metrics import accuracy_score, f1_score, precision_score, recall_score


def evaluate(name: str, pipe, X, y) -> dict:
    pred = pipe.predict(X)
    scores = {
        "accuracy": float(accuracy_score(y, pred)),
        "precision": float(precision_score(y, pred, average="binary", zero_division=0)),
        "recall": float(recall_score(y, pred, average="binary", zero_division=0)),
        "f1": float(f1_score(y, pred, average="binary", zero_division=0)),
    }
    print(f"      {name:12s} ACC={scores['accuracy']:.3f}  "
          f"P={scores['precision']:.3f}  R={scores['recall']:.3f}  F1={scores['f1']:.3f}")
    return scores

# main() 안에서 모델 두 줄만 바꾼다
#   dummy = build_pipeline(DummyClassifier(strategy="most_frequent"))
#   rf    = build_pipeline(RandomForestClassifier(
#               n_estimators=args.n_estimators, max_depth=args.max_depth,
#               random_state=SEED, n_jobs=-1, class_weight="balanced"))
# 그리고 split_data()의 train_test_split 에 stratify 를 추가한다
#   train_test_split(X, y, test_size=0.4, random_state=SEED, stratify=y)
# ─────────────────────────────────────────────────────────
```

실행 후 결과를 아래 표에 채웁니다. (LMS 제출 양식과 동일합니다.)

| 항목 | 값 |
|---|---|
| 팀 번호 / 주제 | T__ / |
| 데이터 행 수 / 열 수 | |
| 분할 방식 | 무작위 / 시간 순 |
| 주 지표 | RMSE / MAE / F1 / Recall / Precision / Accuracy 중 1개 |
| 베이스라인(dummy) 검증 성능 | |
| 후보 모델 검증 성능 | |
| 개선 폭 | |
| 모델 S3 경로 | `s3://mlops-2026-<학번>/models/…` |
| 학습 소요 시간 | 초 |

**2. M1 문제정의서 최종화 (45분)**

3주차 초안을 열어 다음 여섯 항목을 확정합니다. **오늘 나온 숫자를 반영하지 않은 문서는 반려합니다.**

| # | 항목 | 작성 요령 | 나쁜 예 → 좋은 예 |
|---|---|---|---|
| 1 | 주제 | 한 문장. 누가 무엇을 위해 쓰는지 포함 | "따릉이 예측" → "따릉이 운영팀이 다음 시간대 대여소별 재배치 수량을 정하기 위한 시간별 대여량 예측" |
| 2 | 데이터 | 출처 URL·기간·행수·라이선스·개인정보 여부 (3주차 데이터 카드 첨부) | "공공데이터" → "서울열린데이터광장 따릉이 대여이력, 2024-01~2024-12, 8,760행, CC BY, 개인정보 없음" |
| 3 | 입력(피처) | 컬럼명과 타입을 표로. **예측 시점에 알 수 있는 값만** | 목록에 `daily_total` 있으면 누수 → 제거 |
| 4 | 출력(타깃) | 컬럼명 + 자료형 + 단위 | "대여량" → "`rent_count`, 정수, 시간당 대여 건수" |
| 5 | 평가지표 | **주 지표 1개** + 보조 지표 + 그 지표를 고른 이유 1문장 | "정확도" → "주 지표 RMSE. 대여소가 비는 큰 오차의 피해가 크므로 큰 오차에 벌점이 큰 RMSE 선택" |
| 6 | 성공 기준 | **오늘의 dummy 성능 대비 수치** | "정확도 90%" → "검증 RMSE 846 이하 (베이스라인 1204 대비 30% 개선)" |

**3. 제출 (실습 B 종료 5분 전)**

LMS `M1 문제정의서` 과제란에 팀 대표 1명이 PDF로 제출합니다. 파일명은 `M1_T<팀번호>_문제정의서.pdf`.

### 팀 프로젝트 연결

- 오늘 만든 `train_team.py`는 **5주차에 MLflow 로깅을 삽입할 바로 그 파일**입니다. 버리지 말고 팀 저장소에 커밋하세요.
- 오늘의 `.pkl`은 **7주차 FastAPI가 로드할 그 파일**입니다. S3 `models/` 경로를 팀 README에 적어 두세요.
- 오늘 확정한 주 지표는 **15주차 최종 발표까지 바꾸지 않는 것이 원칙**입니다. 바꿔야 한다면 사유를 문서에 남깁니다.
- 오늘 확정한 성공 기준은 **최종 산출물 루브릭 "파이프라인 완성도" 채점의 기준선**이 됩니다.

### 순회 지도 포인트

강사·조교가 팀을 돌며 아래 세 가지만 확인합니다.

1. **성능이 지나치게 좋은 팀을 찾아낸다.** R² 0.99 이상, F1 0.98 이상이면 무조건 피처 목록을 함께 봅니다. 9할은 누수입니다. "이 컬럼, 예측하는 순간에 알 수 있어요?"라고 묻습니다.
2. **`Pipeline` 구조를 해체하지 않았는지 본다.** `fit_transform`을 Pipeline 밖에서 따로 호출한 팀이 반드시 나옵니다. 발견 즉시 되돌리게 하고, 7주차 스큐 이야기를 짧게 반복합니다.
3. **성공 기준에 숫자가 있는지 본다.** "높은 정확도", "실용적인 수준" 같은 말이 남아 있으면 그 자리에서 dummy 숫자를 근거로 다시 쓰게 합니다.

---

## 5. 체크포인트 (제출물)

| # | 제출물 | 형식 | 배점 |
|---|---|---|---|
| 1 | 공통 예제 학습 로그 (`[1/6]`~`[6/6]` 전체) + `metrics.json` 내용 | 터미널 스크린샷 | 0.7 |
| 2 | S3 `models/` 폴더에 `.pkl`과 `metrics_*.json`이 있는 콘솔 화면 (버킷명 보이게) | 스크린샷 | 0.5 |
| 3 | 팀 데이터 베이스라인 성능표 (dummy vs 모델, 개선 폭 포함) | 표 이미지 또는 마크다운 | 0.5 |
| 4 | `predict_sample.py` 실행 결과 (원본 형태 입력 → 예측값) | 스크린샷 | 0.3 |
| 5 | 🏁 **M1 문제정의서 최종본** (6항목 + 데이터 카드) | PDF | **10 (별도 배점)** |

### 평가 기준 (NCS 수행준거 연계)

- `2001070301_19v1.3` **인공지능 후보 모델 도출하기** — 문제 유형(회귀/분류)에 맞는 후보 모델을 최소 2개(더미 + 실제 모델) 도출하고, 각각을 선택한 근거를 한 문장 이상으로 설명했는가.
- `2001070302_19v1.1` **인공지능 모델 기본 설계하기** — 입력 피처 목록과 출력(타깃)을 자료형·단위까지 명시하고, 전처리 단계와 모델을 하나의 `Pipeline` 구조로 설계했는가.
- `2001070305_19v1.2` **인공지능 특징 생성하기(부분)** — 수치형/범주형 컬럼을 구분해 각각에 적합한 변환(결측 대체·표준화·원-핫 인코딩)을 지정하고, 예측 시점에 알 수 없는 컬럼을 피처에서 배제했는가.
- `2001070306_19v1.1` **인공지능 학습 알고리즘 선정하기** — 데이터 규모·자료형·학습 시간 제약(CPU 10분 이내)을 고려해 학습 알고리즘을 선정하고 그 근거를 제시했는가.
- `2001070306_19v1.4` **인공지능 학습하기** — 학습/검증/테스트를 분리해 학습을 수행하고, 학습 산출물(모델 파일·지표 파일)을 재사용 가능한 형태로 저장·업로드했는가.
- `2001070307_19v1.1` **인공지능 모델 평가 기준 정하기** — 문제 특성에 맞는 주 지표 1개를 선정해 선정 사유를 기술하고, 베이스라인 성능을 기준으로 정량적 성공 기준을 설정했는가.

---

## 6. 정리 & 다음 주 예고 (15분)

**오늘 배운 것 3줄 요약**

1. 데이터는 **분할부터** 한다. 전처리는 그 뒤에, 반드시 `Pipeline` 안에서. 순서가 뒤집히면 데이터 누수다.
2. 지표는 **문제가 정한다.** 불균형이면 정확도 대신 F1, 큰 오차가 치명적이면 MAE 대신 RMSE. 주 지표는 하나.
3. 저장하는 것은 모델이 아니라 **Pipeline 전체**다. 이 결정이 7주차에 여러분을 구한다.

**다음 주 미리보기 한 문장**

> "오늘 여러분은 모델을 한 번 돌렸습니다. 다음 주에는 서른 번 돌릴 겁니다. 그때 '아까 그 좋았던 설정이 뭐였지?'가 반드시 터집니다. 그걸 기계가 대신 기억하게 만드는 도구, MLflow를 세웁니다. **다음 주 첫 15분은 M1 팀별 3분 발표**로 시작하니 문제정의서를 준비해 오세요."

**리소스 정리 타임 (5분) 체크 항목**

```bash
# 1) EC2 인스턴스 중지 (콘솔에서 해도 됨)
#    ⚠ '종료(Terminate)'가 아니라 '중지(Stop)'입니다. 종료하면 다 날아갑니다.
aws ec2 stop-instances --instance-ids <본인_인스턴스ID> --region ap-northeast-2

# 2) 상태 확인 — stopping 또는 stopped 여야 함
aws ec2 describe-instances \
  --instance-ids <본인_인스턴스ID> \
  --region ap-northeast-2 \
  --query "Reservations[].Instances[].State.Name" --output text

# 3) S3에 올린 모델 확인 (여기 있는 건 지우지 않습니다)
aws s3 ls s3://mlops-2026-<학번>/models/ --region ap-northeast-2
```

- [ ] EC2 인스턴스가 `stopped` 상태인가 — 조교 1:1 확인
- [ ] `.pkl`과 `metrics.json`이 S3 `models/`에 있는가 (로컬에만 있으면 다음 주에 없어질 수 있음)
- [ ] 실습 중 임시로 만든 큰 CSV가 EC2 홈에 쌓여 있지 않은가 (`du -sh ~/mlops`, 5GB 넘으면 정리)
- [ ] M1 제출 완료했는가

> ⚠ 인스턴스를 중지해도 EBS 볼륨의 저장 요금은 계속 발생합니다. 정확한 금액은 서울 리전의 현재 가격과 실제 용량으로 확인합니다. 학기 중에는 개발 환경 보존을 위해 인스턴스를 중지하고, 15주차에 백업 후 종료합니다.

---

## 7. 과제

**(1) 개인 과제 — 지표 바꿔보기 (제출: 다음 주 수업 시작 전)**

`train_baseline.py`를 `--max-depth` 값만 바꿔 3회 이상 실행하고, 아래 표를 채워 LMS에 제출합니다.

| max_depth | 검증 RMSE | 테스트 RMSE | 학습 시간(초) | 관찰한 것 |
|---|---|---|---|---|
| 4 | | | | |
| 12 | | | | |
| 30 | | | | |

마지막에 한 문장을 덧붙입니다. **"깊이를 늘렸을 때 검증 성능과 테스트 성능이 어떻게 달라졌는가, 그리고 그게 무슨 뜻이라고 생각하는가."**
(의도: 다음 주 하이퍼파라미터 튜닝과 과적합 개념을 미리 몸으로 겪게 함. 이 표는 5주차 실습 A에서 MLflow로 다시 만듭니다.)

**(2) 팀 과제 — M1 발표 준비 (다음 주 수업 첫 15분)**

3분 발표. 슬라이드 없이 문제정의서 화면으로 진행합니다. 반드시 포함할 것:
- 우리 팀이 예측하는 것 한 문장
- 주 지표와 **그 지표를 고른 이유**
- 오늘의 dummy 성능 vs 모델 성능 두 숫자
- 성공 기준

**(3) 읽어올 것**

팀 저장소 README에 "베이스라인" 섹션을 만들고 오늘의 성능표를 붙여 둡니다. 5주차에 이 표가 MLflow 화면으로 대체됩니다.

---

## 부록 A. 명령어 치트시트

> 1페이지로 인쇄해 배포. `<학번>`은 본인 학번으로 치환.

```bash
# ── 0. 접속 ──────────────────────────────────────────────
ssh -i <키파일경로> ubuntu@<퍼블릭IP>
cd ~/mlops && source .venv/bin/activate

# ── 1. 환경 ──────────────────────────────────────────────
pip install -r requirements-week04.txt
python -c "import sklearn; print(sklearn.__version__)"   # 1.5.2 확인
free -h                      # 메모리 여유 확인
df -h /                      # 디스크 여유 확인

# ── 2. S3 ────────────────────────────────────────────────
aws s3 ls s3://mlops-2026-<학번>/processed/ --region ap-northeast-2
aws s3 ls s3://mlops-2026-<학번>/models/    --region ap-northeast-2
aws s3 cp <로컬파일> s3://mlops-2026-<학번>/models/ --region ap-northeast-2
aws s3 cp s3://mlops-2026-<학번>/models/<파일> . --region ap-northeast-2

# ── 3. 학습 ──────────────────────────────────────────────
python train_baseline.py --bucket mlops-2026-<학번>
python train_baseline.py --bucket mlops-2026-<학번> --time-split
python train_baseline.py --bucket mlops-2026-<학번> --n-estimators 100 --max-depth 8
python predict_sample.py --bucket mlops-2026-<학번> --key models/<파일명>
cat metrics.json

# ── 4. 저장소 ────────────────────────────────────────────
git fetch --all --tags
git checkout week-04-start      # 실습 시작본
git checkout week-04-done       # 완성본 (컷오프 시)
git stash                       # 작업 중 내용 임시 보관

# ── 5. 정리 (수업 종료 전 필수) ──────────────────────────
aws ec2 stop-instances --instance-ids <인스턴스ID> --region ap-northeast-2
aws ec2 describe-instances --instance-ids <인스턴스ID> --region ap-northeast-2 \
  --query "Reservations[].Instances[].State.Name" --output text
```

**scikit-learn 핵심 4줄**

```python
X_tr, X_te, y_tr, y_te = train_test_split(X, y, test_size=0.2, random_state=42)  # 분할 먼저
pipe = Pipeline([("pre", preprocess), ("model", model)])                          # 전처리+모델 묶기
pipe.fit(X_tr, y_tr)                                                              # 학습 데이터에만 fit
joblib.dump(pipe, "model.pkl")                                                    # 모델이 아니라 Pipeline을 저장
```

---

## 부록 B. 용어 정리

| 용어 | 뜻 | 한 줄 설명 |
|---|---|---|
| 베이스라인 (baseline) | 기준선 모델 | 평균이나 최빈값만 찍는 가장 단순한 모델. 이보다 못하면 모델을 쓸 이유가 없다 |
| 학습 세트 (train set) | 공부용 데이터 | 모델이 패턴을 배우는 데이터. 전체의 약 60% |
| 검증 세트 (validation set) | 모의고사 데이터 | 모델·하이퍼파라미터를 고를 때 쓰는 데이터. 여러 번 봐도 됨 |
| 테스트 세트 (test set) | 최종 시험 데이터 | 최종 성능 보고용. **딱 한 번만** 본다 |
| 데이터 누수 (leakage) | 답이 새는 것 | 예측 시점에 알 수 없는 정보가 피처나 전처리에 섞이는 것. 성능이 비현실적으로 좋아진다 |
| 과적합 (overfitting) | 과하게 맞춰짐 | 학습 데이터는 잘 맞히지만 새 데이터는 못 맞히는 상태 |
| 파이프라인 (Pipeline) | 전처리+모델 묶음 | scikit-learn 객체. `fit` 한 번으로 전처리와 학습이 순서대로 실행된다 |
| ColumnTransformer | 열별 변환기 | 숫자 열에는 표준화, 문자 열에는 원-핫 인코딩처럼 열마다 다른 전처리를 적용 |
| 표준화 (StandardScaler) | 평균 0·표준편차 1로 맞추기 | 단위가 다른 컬럼들을 같은 눈금으로 만든다 |
| 원-핫 인코딩 (One-Hot Encoding) | 범주를 0/1 열로 펼치기 | `"Mon"` → `weekday_Mon=1`, 나머지 요일 열은 0 |
| 결측 대체 (Imputation) | 빈 칸 채우기 | 중앙값·최빈값 등으로 채운다. 채울 값은 **학습 데이터에서만** 계산 |
| RMSE | 제곱근 평균제곱오차 | 오차를 제곱해 평균 낸 뒤 루트. 큰 오차에 벌점이 크다 |
| MAE | 평균절대오차 | 오차 절댓값의 평균. 단위가 그대로라 해석이 쉽다 |
| R² | 결정계수 | 1에 가까울수록 잘 설명. 단위가 없어 비교엔 편하지만 주 지표로는 비추천 |
| F1 점수 | 정밀도와 재현율의 조화평균 | 불균형 데이터 분류에서 정확도 대신 쓴다 |
| 정밀도 (Precision) | 맞다고 한 것 중 진짜 비율 | 오경보가 치명적일 때 중요 |
| 재현율 (Recall) | 진짜 중 잡아낸 비율 | 놓치는 게 치명적일 때 중요 |
| 학습-서빙 스큐 | 학습과 서빙의 어긋남 | 학습 때 전처리와 서빙 때 전처리가 달라 에러 없이 틀린 예측이 나오는 현상 (7주차) |
| joblib | 파이썬 객체 저장 도구 | 학습된 Pipeline을 `.pkl` 파일로 저장·복원한다 |
| 하이퍼파라미터 | 사람이 정하는 설정값 | `n_estimators`, `max_depth`처럼 학습으로 배우지 않고 사람이 고르는 값 (5주차) |

---

## 부록. AWS 화면과 공식 문서

- 콘솔: <https://console.aws.amazon.com/s3/>
- 이동 경로: **S3 → 팀 버킷 → models**
- 화면에서 확인: 모델 파일, `metrics.json`, 실행 시각, 버전 태그
- Boto3 업로드 공식 예제: <https://docs.aws.amazon.com/AmazonS3/latest/userguide/upload-objects.html>

```mermaid
flowchart LR
    A[학습 데이터] --> B[Pipeline fit]
    B --> C[평가 지표]
    B --> D[model.pkl]
    C --> E[S3 models/버전]
    D --> E
```+

---

## 실제 AWS 콘솔 화면 실습 가이드 (4주차)

> 2026-08-24 서울 리전 콘솔에서 직접 캡처했다. 계정 번호가 보이는 오른쪽 상단은 제외했다. 화면 모양이 바뀌면 메뉴 경로와 확인 항목을 기준으로 찾는다.

### SageMaker AI 대시보드

![SageMaker AI 대시보드](../assets/aws-console/week04/01-sagemaker-dashboard.png)

- 콘솔 경로: **SageMaker AI → Dashboard**
- 확인할 것: 학습·모델·엔드포인트 현황 확인
- [같은 화면 열기](https://console.aws.amazon.com/sagemaker/)

### SageMaker 학습 및 튜닝 작업

![SageMaker 학습 및 튜닝 작업](../assets/aws-console/week04/02-sagemaker-training-jobs.png)

- 콘솔 경로: **SageMaker AI → Training & tuning jobs**
- 확인할 것: 학습 작업 상태와 생성 시간 확인
- [같은 화면 열기](https://console.aws.amazon.com/sagemaker/home?region=ap-northeast-2#/training)

### SageMaker Notebook 환경

![SageMaker Notebook 환경](../assets/aws-console/week04/03-sagemaker-notebooks.png)

- 콘솔 경로: **SageMaker AI → Notebooks**
- 확인할 것: 노트북 상태와 유형 확인
- [같은 화면 열기](https://console.aws.amazon.com/sagemaker/home?region=ap-northeast-2#/notebooks-and-git-repos)

### SageMaker 모델 목록

![SageMaker 모델 목록](../assets/aws-console/week04/04-sagemaker-models.png)

- 콘솔 경로: **SageMaker AI → Models**
- 확인할 것: 모델 이름과 실행 역할 확인
- [같은 화면 열기](https://console.aws.amazon.com/sagemaker/home?region=ap-northeast-2#/models)

### SageMaker 모델 관리 진입 화면

![SageMaker 모델 관리 진입 화면](../assets/aws-console/week04/05-sagemaker-model-registry.png)

- 콘솔 경로: **SageMaker AI → Model governance**
- 확인할 것: 모델 관리 메뉴 위치 확인
- [같은 화면 열기](https://console.aws.amazon.com/sagemaker/)

### 베이스라인 학습용 EC2

![베이스라인 학습용 EC2](../assets/aws-console/week04/06-ec2-instances-for-training.png)

- 콘솔 경로: **EC2 → Instances**
- 확인할 것: 학습 서버 상태와 유형 확인
- [같은 화면 열기](https://console.aws.amazon.com/ec2/home?region=ap-northeast-2#Instances:)

### 학습 명령용 CloudShell

![학습 명령용 CloudShell](../assets/aws-console/week04/07-cloudshell-training.png)

- 콘솔 경로: **CloudShell**
- 확인할 것: CLI 리전과 실행 환경 확인
- [같은 화면 열기](https://ap-northeast-2.console.aws.amazon.com/cloudshell/home?region=ap-northeast-2)

### 모델 아티팩트용 S3

![모델 아티팩트용 S3](../assets/aws-console/week04/08-s3-model-artifacts.png)

- 콘솔 경로: **S3 → Buckets**
- 확인할 것: processed·models 경로를 둘 위치 확인
- [같은 화면 열기](https://console.aws.amazon.com/s3/)

### CloudWatch 학습 로그

![CloudWatch 학습 로그](../assets/aws-console/week04/09-cloudwatch-training-logs.png)

- 콘솔 경로: **CloudWatch → Logs → Log groups**
- 확인할 것: 학습 로그 그룹과 최근 이벤트 확인
- [같은 화면 열기](https://console.aws.amazon.com/cloudwatch/home?region=ap-northeast-2#logsV2:log-groups)

### 학습 비용 확인

![학습 비용 확인](../assets/aws-console/week04/10-cost-explorer-training.png)

- 콘솔 경로: **Billing → Cost Explorer**
- 확인할 것: EC2·SageMaker 서비스별 비용 확인
- [같은 화면 열기](https://console.aws.amazon.com/costmanagement/home#/cost-explorer)
