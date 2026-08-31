# 5주차 — 실험 관리 (MLflow)

**클라우드 MLOps** · 연성대학교 · 230분 (10분 조기 종료)
NCS: `2001070306_19v1.3` 인공지능 인자 조율하기 / `2001070307_19v1.2` 인공지능 모델 선정 기준 정하기 / `2001070307_19v1.3` 인공지능 학습 결과 검증하기 / `2001070307_19v1.4` 인공지능 최적화 모델 선정하기 / `2001070308_19v1.1` 인공지능 선정모델 구성 요소 관리하기

---

## 0. 회차 요약 (강사용 1페이지)

| 항목 | 내용 |
|---|---|
| 학습목표 | EC2에 MLflow 추적 서버를 직접 띄우고, 학습 코드에 파라미터·지표·모델 기록을 삽입해 여러 실험을 비교한 뒤, 근거를 가지고 최고 모델을 레지스트리에 등록할 수 있다. |
| 오늘의 결과물 | ① 동작하는 MLflow 추적 서버(`http://<EC2 IP>:5000`) ② 실험 5건 이상이 비교되는 UI 스크린샷 ③ S3 `mlflow-artifacts/`에 쌓인 모델 아티팩트 ④ 레지스트리에 등록된 팀 모델(`team-T01-model` 버전 1) ⑤ 모델 선정 근거 문서 |
| 사전 준비 | 저장소 태그 `week-05-start` / `week-05-done` 푸시 · `requirements-week05.txt` 배포 · 학생 IAM 정책에 `ec2:AuthorizeSecurityGroupIngress`·`ec2:RevokeSecurityGroupIngress` 포함 확인 · **학교망 공인 IP 확인 방법 사전 테스트**(학생 30명이 동일 IP일 수 있음) · MLflow UI 데모 화면 캡처 갱신 |
| 학생 준비물 | 노트북, EC2 개발 서버, 4주차 `train_baseline.py`/`train_team.py`, 팀 데이터, **M1 문제정의서(발표용)** |
| 예상 사고 지점 | ① SSH를 끊으면 MLflow 서버가 같이 죽어서 UI가 안 열림 ② S3 아티팩트 저장 시 `AccessDenied` ③ 보안그룹 5000 포트를 안 열어 UI 접속 불가 (또는 `0.0.0.0/0`으로 전체 개방해 버림) |

### 시간표

| 시간 | 구성 | 분 |
|---|---|---|
| 00:00–00:10 | 도입 — **M1 팀별 3분 발표**(요약 진행) + 오늘 결과물 데모 | 10 |
| 00:10–00:55 | 이론 — 재현성 / 실험 추적 4요소 / 모델 레지스트리와 스테이지 | 45 |
| 00:55–01:05 | 휴식 | 10 |
| 01:05–01:55 | **실습 A** — MLflow 서버 기동 + 로깅 삽입 + 5조합 실행 + UI 비교 | 50 |
| 01:55–02:05 | 휴식 | 10 |
| 02:05–03:30 | **실습 B** — 팀 프로젝트에 MLflow 적용 + 최고 모델 레지스트리 등록 | 85 |
| 03:30–03:45 | 체크포인트 제출 + 다음 주 예고 | 15 |
| 03:45–03:50 | 리소스 정리 타임 (**보안그룹 5000 규칙 회수** + 서버 종료 + EC2 중지) | 5 |
| **합계** | | **230** |

> **운영 메모**: M1 발표는 도입 10분 안에 다 소화할 수 없습니다. **팀당 3분 × 6~7팀은 별도로 이론 시간을 5분 당겨 쓰거나, 팀별 발표를 "1분 핵심 + 강사 코멘트 30초"로 압축**해 진행합니다. 발표를 길게 가져가면 실습 A가 무너집니다. 시간이 부족하면 발표는 앞의 3팀만 하고 나머지는 다음 주 도입으로 넘깁니다. 이론 45분을 40분으로 줄이는 것은 허용하되, 실습 A/B 시간은 건드리지 않습니다.

---

## 1. 도입 (10분)

### 지난주 복습 퀴즈 (구두 3문항)

1. 저장할 때 왜 모델만 저장하지 않고 `Pipeline`을 통째로 저장했나요? — (전처리 규칙(평균·표준편차·범주 목록·컬럼 순서)까지 한 파일에 담아, 서빙에서 전처리를 다시 구현하다 어긋나는 학습-서빙 스큐를 막기 위해)
2. `DummyRegressor`를 왜 굳이 학습시켰나요? — (기준선. 이보다 못하면 모델을 쓸 이유가 없고, 개선 폭을 숫자로 말할 수 있게 해줌)
3. 지난주 과제에서 `max_depth`를 4, 12, 30으로 바꿔봤죠. 어느 값이 제일 좋았는지 지금 바로 말할 수 있는 사람? — (대부분 못 말합니다. **이 침묵이 오늘 수업의 출발점입니다.** 강사는 여기서 바로 다음 문장으로 넘어갑니다.)

### 오늘 만들 것 데모

강사는 미리 띄워 둔 MLflow UI를 화면에 공유하고 3분간 조작합니다.

1. `http://<강사 EC2 IP>:5000` 접속 → Experiments 목록에서 `bike-baseline` 클릭.
2. 실행(run) 12건이 표 형태로 보입니다. 컬럼 헤더 `rmse_valid`를 클릭해 **정렬**합니다. 가장 좋은 행이 맨 위로 옵니다.
3. 그 행을 클릭 → Parameters 탭에서 `n_estimators=300, max_depth=16`, Metrics 탭에서 `rmse_valid=398.2`, Artifacts 탭에서 `model/` 폴더와 `model.pkl`을 보여줍니다.
4. 실행 두 개를 체크박스로 선택 → **Compare** 버튼 → 파라미터 차이가 나란히 표시되는 화면을 보여줍니다.
5. 마지막으로 Models 탭 → `bike-baseline-rf` 버전 3 → `champion` 별칭이 붙어 있는 것을 보여줍니다.

> "지난주에 제가 물었죠, 어느 설정이 제일 좋았냐고. 아무도 대답 못 했습니다. 정상입니다. 사람은 원래 못 외웁니다. **오늘 우리가 세울 건 여러분 대신 기억해 주는 서버입니다.** 오늘 이후로 '그때 그거 어떻게 했더라'는 없습니다."

---

## 2. 이론 (45분)

### 2-1. "3주 전 그 결과 어떻게 냈지?" — 재현성 문제 (12분)

**강의 스크립트**

> 상황극을 하나 하겠습니다. 여러분이 취업했습니다. 3주 전에 모델을 만들어서 팀장님께 보고했어요. "정확도 91.2% 나왔습니다." 팀장님이 아주 좋아했습니다.
>
> 오늘 팀장님이 말합니다. "그거 서비스에 올리자. 코드 줘 봐."
>
> 여러분이 폴더를 엽니다. 안에 이런 파일들이 있습니다.
> `train.py`, `train2.py`, `train_final.py`, `train_final_real.py`, `train_final_진짜최종.py`, `model.pkl`, `model_new.pkl`, `model_best.pkl`
>
> (웃음) 웃으시는데, 이거 여러분 컴퓨터에 이미 있는 파일 이름입니다. 자, 어느 파일이 91.2%를 낸 파일일까요? 그때 `max_depth`가 몇이었죠? 데이터는 몇 월 버전이었죠? 전처리에서 이상치를 잘랐던가요, 안 잘랐던가요?
>
> 하나도 기억 안 납니다. 다시 돌려봅니다. 89.7%가 나옵니다. 팀장님이 묻습니다. "91.2%라며?" **이 순간이 여러분의 신뢰가 무너지는 순간입니다.**
>
> 이걸 **재현성(reproducibility) 문제**라고 합니다. 우리말로 하면 "다시 만들어낼 수 있는가"입니다. 그리고 이건 여러분이 게을러서 생기는 문제가 아닙니다. **인간의 기억력으로는 원래 불가능한 일**입니다. 실험을 30번 하면 30개의 조합이 생기는데, 그걸 머리로 관리하겠다는 건 계산기 없이 회계를 하겠다는 것과 같습니다.
>
> 그럼 어떻게 해야 할까요? 여러분 중에 "엑셀에 적으면 되지 않나요?"라고 생각한 사람 있죠? 좋은 발상입니다. 실제로 많은 회사가 그렇게 시작합니다. 그런데 세 가지 이유로 반드시 무너집니다. 첫째, **적는 걸 까먹습니다.** 특히 실패한 실험을 안 적습니다. 그런데 실패 기록이 제일 중요하거든요. 같은 실패를 두 번 하니까요. 둘째, **모델 파일과 기록이 따로 놉니다.** 엑셀 3행이 어느 pkl 파일인지 매칭이 안 됩니다. 셋째, **팀원과 공유가 안 됩니다.**
>
> 그래서 나온 도구가 실험 추적(experiment tracking) 도구입니다. 우리가 쓸 건 **MLflow**입니다. 오픈소스고, 무료고, 서버 하나 띄우면 끝납니다. 핵심 아이디어는 이겁니다. **학습 코드에 세 줄을 넣으면, 나머지는 서버가 알아서 기억한다.**

**판서/슬라이드 요점**

- 재현성 = 같은 코드·데이터·설정으로 **같은 결과**를 다시 만들어낼 수 있는가
- 재현성이 깨지는 흔한 원인: 파일명 관리, 랜덤 시드 미고정, 데이터 버전 변경, 설정값 미기록
- 수기 기록(엑셀)이 실패하는 3가지 이유: 누락(특히 실패 실험) / 모델 파일과 분리 / 공유 불가
- 해결: 학습 코드 안에서 **자동으로 기록**하게 만든다 → MLflow
- 최소 안전장치: `random_state` 고정 + `requirements.txt` 버전 고정 + 실험 자동 기록

**학생 질문 예상 & 답변**

- Q: Git에 커밋하면 되는 거 아닌가요? → A: 코드는 Git이 기억해 줍니다. 그런데 Git은 "그 코드로 돌렸더니 RMSE가 얼마 나왔는지"는 모릅니다. **Git은 코드, MLflow는 결과.** 둘은 짝입니다. 그래서 MLflow에 Git 커밋 해시를 같이 기록합니다.
- Q: 실험을 3번밖에 안 할 건데도 필요한가요? → A: 오늘은 5번 합니다. 팀 프로젝트 끝날 때쯤이면 30번이 넘습니다. 그리고 진짜 이유는 다른 데 있습니다. **최종 발표에서 "왜 이 모델을 골랐나"에 답해야 하기 때문**입니다. 근거가 화면에 남아 있는 팀과 말로만 하는 팀은 점수가 다릅니다. 루브릭의 "실험 관리·재현성" 5점이 그겁니다.
- Q: 회사에서도 MLflow 쓰나요? → A: 네, 가장 널리 쓰이는 오픈소스입니다. 클라우드 관리형으로는 SageMaker Experiments, Vertex AI Experiments, 상용으로는 Weights & Biases가 있습니다. **개념은 전부 똑같습니다.** 하나를 제대로 배우면 나머지는 하루면 익힙니다.

---

### 2-2. 실험 추적의 4요소 — 파라미터·메트릭·아티팩트·코드 버전 (12분)

**강의 스크립트**

> 실험 하나를 기억한다는 게 정확히 뭘 기억한다는 걸까요? 네 가지입니다. 이걸 외우면 오늘 실습은 절반 끝난 겁니다.
>
> **첫째, 파라미터(parameter).** 내가 정한 값들입니다. `n_estimators=200`, `max_depth=12`, 데이터 분할 비율, 랜덤 시드. **입력 쪽 설정값**이라고 생각하세요. MLflow에서는 `log_param()`으로 기록합니다.
>
> **둘째, 메트릭(metric).** 결과 숫자입니다. RMSE, MAE, F1, 학습 시간. **출력 쪽 성적표**입니다. `log_metric()`으로 기록합니다.
>
> 여기서 초보자가 제일 많이 하는 실수를 미리 말씀드릴게요. **파라미터와 메트릭을 헷갈립니다.** 구분법은 간단합니다. **내가 미리 정할 수 있으면 파라미터, 돌려봐야 아는 거면 메트릭.** `max_depth`는 제가 정하죠? 파라미터입니다. RMSE는 돌려봐야 알죠? 메트릭입니다.
>
> **셋째, 아티팩트(artifact).** 우리말로 산출물입니다. 학습이 만들어낸 **파일들**이죠. 모델 `.pkl`, 그래프 이미지, 혼동행렬, 피처 중요도 표. MLflow는 이걸 S3에 올려서 보관하고, UI에서 다운로드할 수 있게 해줍니다. 지난주에 우리가 손으로 `s3.upload_file()` 했던 거, 오늘부터는 `mlflow.sklearn.log_model()` 한 줄이 대신 합니다.
>
> **넷째, 코드 버전.** 어느 커밋으로 돌렸는지입니다. MLflow는 Git 저장소 안에서 실행하면 커밋 해시를 자동으로 기록합니다. 나중에 "이 실험 코드 보여줘"가 가능해집니다.
>
> 그리고 이 넷을 담는 그릇이 두 개 있습니다. **실행(run)** 과 **실험(experiment)** 입니다. run은 학습 1회입니다. experiment는 run들을 담는 폴더고요. 우리는 `bike-baseline` 실험 안에 5개 run을 만들 겁니다. 팀은 `team-T01-churn` 같은 이름으로 자기 실험을 만들면 됩니다.
>
> 마지막으로 구조 이야기를 하겠습니다. MLflow 서버는 두 종류의 저장소를 씁니다. **백엔드 스토어(backend store)** 는 파라미터·메트릭 같은 작은 숫자를 담는 데이터베이스입니다. 우리는 SQLite라는 파일 한 개짜리 DB를 씁니다. **아티팩트 스토어(artifact store)** 는 모델 파일 같은 큰 덩어리를 담습니다. 우리는 S3를 씁니다.
>
> 왜 나눌까요? 숫자는 검색하고 정렬해야 하니까 DB가 어울리고, 12MB짜리 pkl 파일은 DB에 넣을 게 아니라 S3에 두는 게 맞기 때문입니다. **DB에는 "그 파일이 S3 어디 있는지" 주소만 저장됩니다.** 도서관에 비유하면 SQLite가 검색 목록이고 S3가 서고입니다.

**판서/슬라이드 요점**

| 요소 | 무엇 | MLflow API | 예 |
|---|---|---|---|
| 파라미터 | 내가 **미리 정하는** 설정값 | `mlflow.log_param()` | `max_depth=12`, `seed=42` |
| 메트릭 | **돌려봐야 아는** 결과 숫자 | `mlflow.log_metric()` | `rmse_valid=412.7` |
| 아티팩트 | 학습이 만들어낸 **파일** | `mlflow.sklearn.log_model()`, `log_artifact()` | `model.pkl`, `feature_importance.png` |
| 코드 버전 | 어느 커밋으로 돌렸나 | Git 저장소 안이면 **자동** | `a3f9c1e` |

- 그릇: **run**(학습 1회) ⊂ **experiment**(관련 run 묶음)
- 저장소 2종: 백엔드 스토어(SQLite, 숫자·메타데이터) + 아티팩트 스토어(S3, 파일)
- 구분법 한 줄: **미리 정하면 파라미터, 돌려봐야 알면 메트릭**

**학생 질문 예상 & 답변**

- Q: 데이터 자체도 기록되나요? → A: 기본은 아닙니다. 데이터는 대개 크니까요. 대신 **데이터의 S3 경로와 행 수, 해시값을 파라미터로 기록**하는 게 실무 관행입니다. 오늘 스크립트에 `log_param("data_uri", ...)`, `log_param("n_rows", ...)`를 넣어 뒀습니다.
- Q: SQLite 말고 진짜 DB를 쓰면 안 되나요? → A: 실무에서는 PostgreSQL(RDS)을 씁니다. 여러 명이 동시에 쓸 수 있으니까요. SQLite는 파일 하나라서 동시 쓰기에 약합니다. 우리는 학생 1명당 서버 1개라 SQLite로 충분하고, **비용이 0원**입니다.
- Q: 메트릭을 학습 도중에 여러 번 기록할 수도 있나요? → A: 네. `mlflow.log_metric("loss", v, step=epoch)` 처럼 `step`을 주면 UI에 학습 곡선 그래프가 그려집니다. 딥러닝에서 많이 씁니다.

---

### 2-3. 모델 레지스트리와 스테이지 — 실험과 서비스를 잇는 다리 (13분)

**강의 스크립트**

> 자, 실험을 30번 했습니다. run이 30개 쌓였습니다. 이제 질문. **7주차에 API를 만들 때, 이 30개 중 어느 걸 로드해야 하죠?**
>
> "제일 좋은 거요." 네, 그런데 코드에 어떻게 씁니까? `s3://.../mlflow-artifacts/731/a9f3.../model.pkl`? 이런 주소를 API 코드에 하드코딩하고 싶으세요? 모델을 개선할 때마다 API 코드를 고치고 다시 배포해야 합니다. 끔찍하죠.
>
> 그래서 **모델 레지스트리(model registry)** 가 있습니다. 우리말로 모델 등록소입니다. 개념은 딱 하나예요. **"이름"에 "버전"을 붙여서 관리한다.**
>
> 비유를 들겠습니다. 여러분이 앱을 씁니다. 카카오톡이라고 하죠. 앱 이름은 항상 "카카오톡"입니다. 그런데 버전은 계속 올라가죠. 10.1.0, 10.2.0... 여러분은 버전 번호를 몰라도 "카카오톡"만 누르면 최신 버전이 뜹니다. 레지스트리가 이걸 해줍니다. API 코드에는 **`models:/bike-baseline-rf@champion`** 이라고만 쓰면, 실제로 어느 파일이 로드될지는 레지스트리가 정합니다.
>
> 여기서 **스테이지(stage)** 개념이 나옵니다. 등록된 각 버전에 딱지를 붙이는 겁니다. 전통적으로 네 가지를 씁니다.
> - **None** — 방금 등록됨, 아직 아무 데도 안 씀
> - **Staging** — 검증 중. 테스트 환경에서 돌려보는 중
> - **Production** — 실제 서비스가 쓰는 것. **동시에 하나만**
> - **Archived** — 물러남. 지우진 않고 보관
>
> 왜 이런 딱지가 필요할까요? **되돌리기(rollback) 때문입니다.** 새 모델을 올렸는데 서비스가 이상해졌다. 그럼 어떻게 하죠? 이전 버전을 다시 Production으로 올리면 끝입니다. 5초입니다. 딱지가 없으면 어느 게 전 버전이었는지부터 찾아야 합니다.
>
> 여기서 버전 안내를 하나 드립니다. **MLflow 2.9부터 스테이지는 "권장하지 않음"으로 바뀌었고, 대신 별칭(alias)을 씁니다.** `@champion`, `@challenger` 같은 이름을 버전에 붙이는 방식인데, **개념은 똑같습니다.** 딱지를 붙여서 이름으로 부른다는 것. 오늘 우리는 MLflow 2.16을 쓰고, **별칭 방식으로 실습**합니다. 스테이지라는 말은 여러분이 회사에서 옛날 코드를 볼 때 만날 테니 개념만 알아두세요.
>
> 마지막으로 오늘 여러분에게 요구할 것. 레지스트리에 등록할 때 **왜 이 모델을 골랐는지 한 문단을 설명(description)에 적으세요.** "검증 RMSE가 가장 낮아서"만으로는 부족합니다. "검증 RMSE 398로 최저이고, 테스트 RMSE 405와 차이가 작아 과적합 징후가 없으며, 학습 시간 24초로 재학습 부담이 낮음" 정도는 되어야 합니다. **모델 선정 기준을 정하고 그 기준으로 골랐다는 걸 문서로 남기는 것**, 이게 오늘 NCS 평가의 핵심입니다.

**판서/슬라이드 요점**

- 레지스트리 = 모델에 **이름 + 버전**을 부여해 관리하는 등록소
- 서빙 코드는 파일 경로가 아니라 **이름으로 참조** → `models:/<이름>@champion`
- 스테이지(None/Staging/Production/Archived) ↔ 별칭(alias, `@champion`) — **MLflow 2.9+ 는 별칭 권장**
- 존재 이유 3가지: ① 서빙 코드와 모델 파일 분리 ② 되돌리기 ③ 누가 언제 무엇을 올렸는지 이력
- 등록 시 **선정 근거를 description에 남긴다** (NCS `2001070307_19v1.2`, `_19v1.4` 평가 대상)

**학생 질문 예상 & 답변**

- Q: 검증 성능이 제일 좋은 게 항상 정답 아닌가요? → A: 아닙니다. 실무에서는 **성능·속도·크기·안정성**을 같이 봅니다. RMSE가 1% 좋은데 추론이 10배 느리고 pkl이 500MB면 서비스에 못 씁니다. 여러분 프로젝트도 7주차에 컨테이너에 넣어야 하니 크기를 봐야 합니다. **선정 기준을 미리 정해 두는 이유**가 이겁니다.
- Q: 팀원끼리 같은 이름으로 등록하면 덮어써지나요? → A: 덮어쓰지 않고 **버전이 올라갑니다.** `team-T01-model` 버전 1, 2, 3... 그래서 팀당 모델 이름 하나로 통일하고, 누가 등록했는지는 태그로 남기면 됩니다.
- Q: `@champion` 별칭을 다른 버전으로 옮기면 옛날 버전은 지워지나요? → A: 아니요. 별칭만 옮겨갑니다. 옛 버전은 그대로 남아 있어서 언제든 되돌릴 수 있습니다.

---

### 2-4. 하이퍼파라미터 조율 — 무엇을 얼마나 바꿔볼 것인가 (8분)

**강의 스크립트**

> 오늘 실습에서 5개 조합을 돌립니다. 그럼 그 5개를 어떻게 고를까요? 아무거나 찍으면 될까요?
>
> 원칙 세 개만 드리겠습니다.
>
> **하나, 한 번에 하나씩 바꿉니다.** `n_estimators`와 `max_depth`를 동시에 바꾸면 성능이 좋아졌을 때 뭐 때문인지 모릅니다. 과학 시간에 배운 통제 변인이랑 똑같습니다.
>
> **둘, 넓게 시작해서 좁힙니다.** `max_depth`를 4, 8, 12, 16, 20 이렇게 넓게 훑어서 어디쯤이 좋은지 감을 잡고, 그다음에 그 주변을 촘촘히 봅니다. 처음부터 12, 13, 14를 보는 건 낭비입니다.
>
> **셋, 값을 바꿔도 성능이 거의 안 변하면 그 파라미터는 놓아줍니다.** 모든 파라미터가 중요한 게 아닙니다. RandomForest에서 `n_estimators`는 어느 선을 넘으면 더 올려도 거의 안 좋아집니다. 학습 시간만 늘죠. 그 지점을 찾으면 됩니다.
>
> 그리고 오늘 여러분이 눈으로 확인해야 할 현상이 하나 있습니다. **`max_depth`를 계속 키우면 학습 성능은 계속 좋아지는데 검증 성능은 어느 순간부터 나빠집니다.** 이게 지난주에 말한 과적합입니다. 지난주 과제로 depth 4/12/30을 돌려보라고 한 게 이걸 보라는 거였습니다. 오늘은 MLflow 차트에서 이 곡선이 꺾이는 걸 직접 보게 됩니다.

**판서/슬라이드 요점**

- 튜닝 3원칙: ① 한 번에 하나씩 ② 넓게 → 좁게 ③ 안 변하면 놓아준다
- **학습 성능과 검증 성능을 같이 기록**해야 과적합이 보인다 → `rmse_train`, `rmse_valid` 둘 다 로깅
- 오늘 실습 조합 5개: `max_depth` = 4 / 8 / 12 / 16 / 20 (`n_estimators`는 200 고정)
- 자동 탐색(Grid/Random Search)은 도구일 뿐, **무엇을 탐색할지 정하는 건 사람**

**학생 질문 예상 & 답변**

- Q: `GridSearchCV` 쓰면 자동으로 다 해주지 않나요? → A: 해줍니다. 다만 조합 수만큼 시간이 곱해집니다. 3개 파라미터 × 5값이면 125번입니다. t3.medium에서는 수업 안에 안 끝납니다. 그리고 **자동으로 돌리면 왜 그 값이 좋은지 감이 안 생깁니다.** 오늘은 손으로 5번 돌려서 감을 잡고, 여유 있는 팀은 실습 B에서 `RandomizedSearchCV`를 붙여 보세요.
- Q: 검증 성능이 계속 좋아지기만 하는데요? → A: 아직 과적합 구간에 안 들어간 겁니다. `max_depth`를 30, 50으로 더 키워 보세요. 아니면 데이터가 아주 단순해서 안 꺾일 수도 있습니다. 그것도 결론입니다. 기록해 두세요.

---

## 3. 실습 A — MLflow 서버 세우고 실험 5건 비교하기 (50분) · 공통 예제 따라하기

**목표** EC2에 MLflow 추적 서버를 띄우고(SQLite + S3), 4주차 학습 코드에 로깅 3종을 삽입해 하이퍼파라미터 5조합을 기록·비교한다.

**사전 배포 파일**

| 파일 | 내용 | 배포 방식 |
|---|---|---|
| `requirements-week05.txt` | MLflow 포함 버전 고정 | LMS |
| `train_mlflow.py` | 4주차 스크립트에 MLflow 로깅 자리만 비워 둔 것 | 저장소 `week-05-start` |
| `sweep.sh` | 5조합 순차 실행 스크립트 | 저장소 `week-05-start` |
| `mlflow.service` | systemd 유닛 파일(자동 기동용, 선택) | 저장소 `week-05-start` |

### 수행 순서

**① EC2 시작 · 접속 · 패키지 설치 (7분)**

```bash
ssh -i <키파일경로> ubuntu@<퍼블릭IP>
cd ~/mlops && source .venv/bin/activate

cat > requirements-week05.txt << 'EOF'
pandas==2.2.3
numpy==2.1.3
scikit-learn==1.5.2
joblib==1.4.2
boto3==1.35.54
mlflow==2.16.2
EOF
pip install -r requirements-week05.txt
mlflow --version
```

- 확인 포인트: `mlflow, version 2.16.2` 가 출력되면 성공입니다.

```bash
# MLflow 작업 폴더 생성 (SQLite 파일이 여기 생깁니다)
mkdir -p ~/mlflow
```

**② 보안그룹 5000번 포트 개방 — 반드시 "내 IP"로만 (5분)**

> ⚠ **`0.0.0.0/0`(전체 개방)은 금지합니다.** MLflow 서버에는 로그인 기능이 없어서, 전체 개방하면 인터넷 아무나 여러분 실험 결과와 모델 파일을 보고 지울 수 있습니다. 감점 사유입니다.

먼저 지금 내 공인 IP를 확인합니다. (노트북 브라우저에서 확인하는 것이 정확합니다. EC2에서 조회하면 EC2의 IP가 나옵니다.)

```bash
# 노트북 터미널(맥/리눅스) 또는 브라우저에서 "내 아이피"를 검색해도 됩니다
curl -s https://checkip.amazonaws.com
```

콘솔에서 여는 방법 (권장 — 화면으로 확인 가능):
EC2 → 인스턴스 선택 → **보안** 탭 → 보안 그룹 클릭 → **인바운드 규칙 편집** → **규칙 추가**
- 유형: `사용자 지정 TCP`
- 포트 범위: `5000`
- 소스: **`내 IP`** 선택 (자동으로 `x.x.x.x/32` 가 들어갑니다)
- 설명: `mlflow-ui-week05-<학번>`  ← **이 설명을 꼭 적으세요. 수업 끝에 이 규칙을 찾아 지워야 합니다.**

CLI로 여는 방법:

```bash
# <보안그룹ID>: sg-로 시작 (EC2 콘솔 보안 탭에서 확인)
# <내IP>: 위에서 확인한 주소
aws ec2 authorize-security-group-ingress \
  --group-id <보안그룹ID> \
  --protocol tcp --port 5000 --cidr <내IP>/32 \
  --region ap-northeast-2
```

- 확인 포인트: 인바운드 규칙 목록에 포트 5000, 소스가 `/32`로 끝나는 한 개 주소인 행이 보이면 성공입니다.

**③ MLflow 서버 기동 — tmux 안에서 (8분)**

> **중요**: 그냥 `mlflow server`를 실행하면 SSH 창이 묶여 버리고, SSH를 끊는 순간 서버가 죽습니다. 그래서 **tmux(터미널을 화면 밖에 붙잡아 두는 도구)** 안에서 띄웁니다.

```bash
sudo apt-get update && sudo apt-get install -y tmux
tmux new -s mlflow        # 'mlflow'라는 이름의 화면을 새로 만들어 들어간다
```

tmux 화면 안에서 서버를 띄웁니다.

```bash
cd ~/mlflow
source ~/mlops/.venv/bin/activate

# <학번>: 본인 학번
mlflow server \
  --backend-store-uri sqlite:////home/ubuntu/mlflow/mlflow.db \
  --default-artifact-root s3://mlops-2026-<학번>/mlflow-artifacts/ \
  --host 0.0.0.0 \
  --port 5000
```

- 확인 포인트: `Listening at: http://0.0.0.0:5000` 이 뜨면 성공입니다.
- 이제 **`Ctrl` + `b` 를 누르고 손을 뗀 다음 `d`** 를 누릅니다. tmux에서 빠져나옵니다(detach). 서버는 뒤에서 계속 돕니다.

```bash
tmux ls          # mlflow: 1 windows ... 가 보이면 살아 있는 것
```

노트북 브라우저에서 접속합니다.

```text
http://<EC2 퍼블릭 IP>:5000
```

- 확인 포인트: MLflow UI의 Experiments 화면이 열리면 성공입니다. **`https`가 아니라 `http`** 입니다. 브라우저가 자동으로 https로 바꾸면 주소를 다시 확인하세요.

**④ 학습 코드에 로깅 삽입 (12분)**

`train_mlflow.py`를 엽니다. 4주차 스크립트와 거의 같고, **MLflow 관련 줄만 비어 있습니다.** 아래 완성본에서 `# ★` 표시된 줄들이 학생이 직접 입력할 부분입니다.

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-
"""
5주차 실습 A — MLflow로 기록하며 학습하기
사용법:
    export MLFLOW_TRACKING_URI=http://127.0.0.1:5000
    python train_mlflow.py --bucket mlops-2026-<학번> --max-depth 12
"""
import argparse
import io
import time
from datetime import datetime

import boto3
import mlflow
import mlflow.sklearn
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


def load_data(bucket, key):
    s3 = boto3.client("s3", region_name=REGION)
    obj = s3.get_object(Bucket=bucket, Key=key)
    return pd.read_csv(io.BytesIO(obj["Body"].read()))


def split_data(df):
    X, y = df[NUM_COLS + CAT_COLS], df[TARGET]
    X_tr, X_tmp, y_tr, y_tmp = train_test_split(X, y, test_size=0.4, random_state=SEED)
    X_val, X_te, y_val, y_te = train_test_split(X_tmp, y_tmp, test_size=0.5, random_state=SEED)
    return X_tr, X_val, X_te, y_tr, y_val, y_te


def build_pipeline(model):
    numeric = Pipeline([("impute", SimpleImputer(strategy="median")),
                        ("scale", StandardScaler())])
    categorical = Pipeline([("impute", SimpleImputer(strategy="most_frequent")),
                            ("onehot", OneHotEncoder(handle_unknown="ignore", sparse_output=False))])
    pre = ColumnTransformer([("num", numeric, NUM_COLS), ("cat", categorical, CAT_COLS)])
    return Pipeline([("pre", pre), ("model", model)])


def scores(pipe, X, y, prefix):
    pred = pipe.predict(X)
    return {
        f"rmse_{prefix}": float(np.sqrt(mean_squared_error(y, pred))),
        f"mae_{prefix}": float(mean_absolute_error(y, pred)),
        f"r2_{prefix}": float(r2_score(y, pred)),
    }


def main():
    ap = argparse.ArgumentParser()
    ap.add_argument("--bucket", required=True)
    ap.add_argument("--key", default="processed/bike_processed.csv")
    ap.add_argument("--experiment", default="bike-baseline")
    ap.add_argument("--n-estimators", type=int, default=200)
    ap.add_argument("--max-depth", type=int, default=12)
    ap.add_argument("--run-name", default=None)
    args = ap.parse_args()

    df = load_data(args.bucket, args.key)
    X_tr, X_val, X_te, y_tr, y_val, y_te = split_data(df)

    mlflow.set_experiment(args.experiment)                      # ★ 실험(폴더) 지정
    run_name = args.run_name or f"rf_d{args.max_depth}_n{args.n_estimators}"

    with mlflow.start_run(run_name=run_name):                   # ★ 실행 1건 시작
        # ── 파라미터: 내가 미리 정한 값 ──────────────────────
        mlflow.log_param("model_type", "RandomForestRegressor")  # ★
        mlflow.log_param("n_estimators", args.n_estimators)      # ★
        mlflow.log_param("max_depth", args.max_depth)            # ★
        mlflow.log_param("seed", SEED)
        mlflow.log_param("data_uri", f"s3://{args.bucket}/{args.key}")
        mlflow.log_param("n_rows", len(df))
        mlflow.log_param("features", ",".join(NUM_COLS + CAT_COLS))

        # ── 베이스라인(비교 기준) ────────────────────────────
        dummy = build_pipeline(DummyRegressor(strategy="mean"))
        dummy.fit(X_tr, y_tr)
        mlflow.log_metric("rmse_dummy_valid", scores(dummy, X_val, y_val, "valid")["rmse_valid"])

        # ── 후보 모델 학습 ──────────────────────────────────
        t0 = time.time()
        pipe = build_pipeline(RandomForestRegressor(
            n_estimators=args.n_estimators, max_depth=args.max_depth,
            random_state=SEED, n_jobs=-1))
        pipe.fit(X_tr, y_tr)
        train_sec = time.time() - t0

        # ── 메트릭: 돌려봐야 아는 값 ─────────────────────────
        m = {}
        m.update(scores(pipe, X_tr, y_tr, "train"))      # 과적합을 보려면 학습 성능도 필요
        m.update(scores(pipe, X_val, y_val, "valid"))
        m.update(scores(pipe, X_te, y_te, "test"))
        m["train_seconds"] = round(train_sec, 2)
        for k, v in m.items():
            mlflow.log_metric(k, v)                              # ★

        # ── 아티팩트: Pipeline 전체를 저장 (4주차 원칙 유지) ──
        mlflow.sklearn.log_model(                                # ★
            sk_model=pipe,
            artifact_path="model",
            input_example=X_val.head(3),
        )

        mlflow.set_tag("week", "05")
        mlflow.set_tag("owner", "<학번>")                        # 본인 학번으로 수정

        print(f"[완료] run_name={run_name}")
        print(f"       rmse_train={m['rmse_train']:.2f} "
              f"rmse_valid={m['rmse_valid']:.2f} rmse_test={m['rmse_test']:.2f} "
              f"({train_sec:.1f}s)")


if __name__ == "__main__":
    main()
```

**⑤ 하이퍼파라미터 5조합 실행 (10분)**

새 SSH 창(또는 tmux 새 창)에서 실행합니다. **서버가 도는 tmux 창은 건드리지 않습니다.**

```bash
cd ~/mlops && source .venv/bin/activate
export MLFLOW_TRACKING_URI=http://127.0.0.1:5000     # 서버와 같은 EC2이므로 localhost로 접속
export AWS_DEFAULT_REGION=ap-northeast-2

cat > sweep.sh << 'EOF'
#!/usr/bin/env bash
set -e
BUCKET=$1
for D in 4 8 12 16 20; do
  echo "=== max_depth=${D} ==="
  python train_mlflow.py --bucket "${BUCKET}" --n-estimators 200 --max-depth "${D}"
done
EOF
chmod +x sweep.sh

./sweep.sh mlops-2026-<학번>
```

- 확인 포인트: `[완료] run_name=rf_d4_n200` … `rf_d20_n200` 까지 5줄이 나오면 성공입니다.

**⑥ UI에서 비교하기 (8분)**

브라우저 `http://<EC2 퍼블릭 IP>:5000` 에서 다음을 순서대로 합니다.

1. 왼쪽 Experiments에서 `bike-baseline` 선택 → 실행 5건이 표로 보입니다.
2. 표 오른쪽 위 **Columns** 에서 `rmse_train`, `rmse_valid`, `rmse_test`, `train_seconds`, `max_depth` 를 켭니다.
3. `rmse_valid` 헤더를 클릭해 오름차순 정렬 → **가장 좋은 조합이 맨 위**로 옵니다.
4. 실행 5건을 모두 체크 → **Compare** → 아래쪽 차트에서 X축을 `max_depth`, Y축을 `rmse_valid` 로 설정합니다.
5. 같은 차트에 `rmse_train`을 추가해 **두 곡선이 벌어지는 지점**을 찾습니다. 그 지점이 과적합이 시작되는 곳입니다.
6. 이 화면을 캡처합니다. → 체크포인트 제출물 ①

- 확인 포인트: `rmse_train`은 depth가 커질수록 계속 내려가는데 `rmse_valid`는 어느 지점 이후 다시 올라갑니다. 이 모양을 못 찾은 학생은 depth 30, 40을 추가로 돌려 봅니다.

### ⚠ 여기서 막히면

| 증상 | 원인 | 조치 |
|---|---|---|
| SSH를 끊었다 다시 붙으니 UI가 안 열림 (`연결 거부`) | `mlflow server`를 그냥 실행해서 SSH 종료와 함께 프로세스가 죽음 | **tmux 안에서 실행**한다. 이미 죽었으면 `tmux new -s mlflow` 후 재기동. 붙어 있는 세션 확인은 `tmux ls`, 재진입은 `tmux attach -t mlflow` |
| tmux가 싫거나 학교 실습실에서 tmux가 낯설다 | — | `nohup` 방식 사용: 아래 대체 명령 참고 |
| 재부팅(인스턴스 중지→시작) 후 서버가 안 뜸 | tmux/nohup은 재부팅을 못 넘긴다 | systemd 등록(아래 유닛 파일). 등록하면 인스턴스 시작 시 자동 기동 |
| `botocore.exceptions.ClientError ... (AccessDenied) ... PutObject` (모델 로깅 순간) | S3 아티팩트 경로에 쓰기 권한이 없음 | ① 버킷명 오타 확인 ② `aws s3 cp test.txt s3://mlops-2026-<학번>/mlflow-artifacts/` 로 권한 직접 확인 ③ EC2에 자격 증명이 있는지 `aws sts get-caller-identity` ④ 다른 학생 버킷을 가리키고 있지 않은지 확인 |
| `NoCredentialsError` 가 서버 로그에 남음 | 서버 프로세스에 AWS 자격 증명이 없음 | tmux 창 안에서도 `aws sts get-caller-identity` 가 되는지 확인. 인스턴스 프로파일(IAM 역할)을 쓰면 자동 적용됨. 키 방식이면 `~/.aws/credentials` 확인 |
| 브라우저에서 접속 안 됨 (계속 로딩) | 보안그룹 5000 미개방 또는 소스 IP 불일치 | 인바운드 규칙 확인. 학교 무선망은 IP가 자주 바뀌므로 `curl -s https://checkip.amazonaws.com` 로 현재 IP를 다시 확인해 규칙을 갱신 |
| `ERR_SSL_PROTOCOL_ERROR` | 브라우저가 `https://` 로 자동 변경 | 주소창에 `http://` 를 명시해서 다시 입력. 크롬은 주소 끝에 `:5000` 까지 입력 후 Enter |
| 서버는 뜨는데 UI가 비어 있음 (실험 없음) | 학습 스크립트가 다른 서버를 보고 있음 | `echo $MLFLOW_TRACKING_URI` 확인. 비어 있으면 `./mlruns` 로컬 폴더에 저장됨. `export MLFLOW_TRACKING_URI=http://127.0.0.1:5000` 후 재실행 |
| `sqlite3.OperationalError: database is locked` | 두 개의 MLflow 서버가 같은 db 파일을 붙잡음 | `ps aux \| grep mlflow` 로 중복 프로세스 확인 후 `kill <PID>`, 하나만 남긴다 |
| `mlflow.sklearn.log_model` 이 아주 느림 | 모델 pkl이 커서 S3 업로드가 오래 걸림 | `--n-estimators 100` 으로 줄인다. 200 이상 + depth 20이면 100MB를 넘길 수 있다 |

**nohup 대체 명령** (tmux 대신)

```bash
cd ~/mlflow
nohup ~/mlops/.venv/bin/mlflow server \
  --backend-store-uri sqlite:////home/ubuntu/mlflow/mlflow.db \
  --default-artifact-root s3://mlops-2026-<학번>/mlflow-artifacts/ \
  --host 0.0.0.0 --port 5000 \
  > ~/mlflow/server.log 2>&1 &

echo $! > ~/mlflow/server.pid     # 프로세스 번호 저장 (나중에 끌 때 사용)
sleep 5 && tail -n 20 ~/mlflow/server.log
```

종료할 때: `kill $(cat ~/mlflow/server.pid)`

**systemd 등록** (재부팅 후에도 자동 기동 — 선택, 여유 있는 학생용)

```bash
sudo tee /etc/systemd/system/mlflow.service > /dev/null << 'EOF'
[Unit]
Description=MLflow Tracking Server
After=network-online.target
Wants=network-online.target

[Service]
User=ubuntu
WorkingDirectory=/home/ubuntu/mlflow
Environment="AWS_DEFAULT_REGION=ap-northeast-2"
ExecStart=/home/ubuntu/mlops/.venv/bin/mlflow server \
  --backend-store-uri sqlite:////home/ubuntu/mlflow/mlflow.db \
  --default-artifact-root s3://mlops-2026-<학번>/mlflow-artifacts/ \
  --host 0.0.0.0 --port 5000
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF

sudo sed -i 's/<학번>/20251234/' /etc/systemd/system/mlflow.service   # 본인 학번으로
sudo systemctl daemon-reload
sudo systemctl enable --now mlflow
sudo systemctl status mlflow --no-pager
journalctl -u mlflow -n 30 --no-pager      # 로그 확인
```

### 컷오프 안내

50분 경과 시 강제 종료합니다. ⑥번까지 못 온 학생은 아래로 완성본을 받아 실습 B로 진입합니다.

```bash
cd ~/mlops
git stash
git checkout week-05-done
export MLFLOW_TRACKING_URI=http://127.0.0.1:5000
./sweep.sh mlops-2026-<학번>
```

서버 기동 자체가 안 된 학생은 **로컬 파일 모드**로 우선 실습 B를 진행합니다(아티팩트는 EC2 로컬에 저장됨). 정리 시간에 조교가 서버 문제를 개별 해결합니다.

```bash
unset MLFLOW_TRACKING_URI          # ./mlruns 폴더에 기록됨
mlflow ui --host 0.0.0.0 --port 5000    # 나중에 이 폴더를 그대로 UI로 볼 수 있음
```

---

## 4. 실습 B — 팀 프로젝트에 MLflow 적용하고 최고 모델 등록하기 (85분) · 각자/팀이 직접 수행

**목표** 팀 데이터 학습 코드에 MLflow 로깅을 이식해 실험 8건 이상을 기록하고, 사전에 정한 선정 기준에 따라 최고 모델을 레지스트리에 등록한다.

**과제 지시문** (학생에게 그대로 읽어줍니다)

> "85분입니다. 세 덩어리로 씁니다.
> **첫 20분** — 지난주 `train_team.py`에 오늘 배운 MLflow 다섯 줄을 붙이세요. `set_experiment`, `start_run`, `log_param`, `log_metric`, `log_model`. 실험 이름은 `team-T<팀번호>-<주제>` 로 통일하세요. 팀원이 각자 다른 이름을 쓰면 비교가 안 됩니다.
> **다음 40분** — 실험을 최소 8건 돌리세요. 팀원끼리 나눠서 돌리면 빠릅니다. 한 사람이 `max_depth`, 한 사람이 `n_estimators`, 한 사람이 피처 조합을 담당하는 식으로요. **같은 서버로 기록하니까 결과는 한 화면에 모입니다.** 서버는 팀에서 한 대만 띄우고, 나머지는 그 서버 주소를 `MLFLOW_TRACKING_URI`로 지정하세요.
> **마지막 25분** — 여기가 오늘 진짜 과제입니다. **모델을 고르기 전에 고르는 기준을 먼저 쓰세요.** 종이에 세 줄이면 됩니다. '주 지표는 무엇이고, 동점이면 무엇으로 판정하고, 어떤 조건이면 아무리 성능이 좋아도 탈락시키는가.' 그 기준을 적은 다음에 UI를 보고 고르세요. 그리고 등록할 때 그 근거를 description에 붙여 넣으세요.
> 순서를 반대로 하면 안 됩니다. **결과를 보고 기준을 만들면 그건 기준이 아니라 변명입니다.**"

### 수행 항목

**1. 팀 학습 코드에 MLflow 이식 (20분)**

```bash
cd ~/mlops
cp train_mlflow.py train_team_mlflow.py
nano train_team_mlflow.py
```

수정할 곳:
- `NUM_COLS`, `CAT_COLS`, `TARGET` — 팀 데이터에 맞게 (4주차와 동일)
- `--experiment` 기본값 → `team-T<팀번호>-<주제>`
- 분류 문제 팀은 `scores()` 함수를 아래로 교체

```python
# ── 분류 문제용 scores() 교체 블록 ─────────────────────────
from sklearn.metrics import accuracy_score, f1_score, precision_score, recall_score

def scores(pipe, X, y, prefix):
    pred = pipe.predict(X)
    return {
        f"accuracy_{prefix}": float(accuracy_score(y, pred)),
        f"precision_{prefix}": float(precision_score(y, pred, zero_division=0)),
        f"recall_{prefix}": float(recall_score(y, pred, zero_division=0)),
        f"f1_{prefix}": float(f1_score(y, pred, zero_division=0)),
    }
# 모델도 RandomForestClassifier / DummyClassifier 로 교체
# ─────────────────────────────────────────────────────────
```

팀원이 각자 자기 EC2에서 돌리되 **기록은 팀 서버 한 곳으로** 보냅니다.

```bash
# 팀 MLflow 서버를 띄운 사람의 EC2 사설 IP 또는 퍼블릭 IP
export MLFLOW_TRACKING_URI=http://<팀서버IP>:5000
export AWS_DEFAULT_REGION=ap-northeast-2
```

> ⚠ 팀 서버로 다른 사람이 접속하려면 보안그룹 5000 인바운드에 **팀원 IP도 `/32`로 각각 추가**해야 합니다. 여전히 `0.0.0.0/0`은 금지입니다.

**2. 실험 8건 이상 기록 (40분)**

역할을 나눠 아래 축을 담당합니다.

| 담당 | 바꿀 것 | 예시 값 |
|---|---|---|
| A | `max_depth` | 4, 8, 12, 16, 20 |
| B | `n_estimators` | 50, 100, 200, 400 |
| C | 피처 조합 | 전체 / 상위 중요도 5개만 / 특정 컬럼 제외 |
| D | 모델 종류 | RandomForest / GradientBoosting / LogisticRegression·Ridge |

`--run-name` 에 무엇을 바꿨는지 알아볼 수 있는 이름을 붙입니다. 예: `rf_d16_n200`, `feat_top5`, `ridge_alpha1`.

- 최소 요구: **8건**. 루브릭 만점 기준은 5건 이상이므로 여유 있게 채웁니다.
- 실패한 실험도 지우지 마세요. 실패 기록이 근거가 됩니다.

**3. 모델 선정 기준 작성 → 최고 모델 등록 (25분)**

먼저 팀 README 또는 제출 문서에 **선정 기준**을 씁니다. (등록 전에 작성해야 합니다.)

```markdown
## 모델 선정 기준 (작성일: 2026-03-09, 작성자: T01)
1. 주 지표: 검증 F1 (M1 문제정의서에서 확정)
2. 동점 판정: 검증 F1 차이가 0.005 이내면 학습 시간이 짧은 쪽을 택한다
3. 탈락 조건:
   - 학습 성능과 검증 성능의 F1 차이가 0.15를 넘는 경우 (과적합)
   - 모델 파일 크기가 100MB를 넘는 경우 (7주차 컨테이너 이미지 부담)
   - 학습 시간이 CPU 기준 10분을 넘는 경우 (재학습 불가)
```

그다음 등록합니다. **방법 1(UI)** 을 기본으로 하고, 코드로 하고 싶은 팀은 방법 2를 씁니다.

**방법 1 — UI에서 등록**
1. 실험 목록에서 최고 run 클릭 → 우측 상단 **Register Model**
2. Model 이름: `team-T<팀번호>-model` (팀에서 하나로 통일)
3. Models 탭 → 방금 만든 모델 → 버전 1 → **Description** 에 선정 근거 붙여넣기
4. 버전 1에 **별칭(alias) `champion`** 부여

**방법 2 — 코드로 등록 (최고 run 자동 탐색 포함)**

```bash
cat > register_best.py << 'PYEOF'
#!/usr/bin/env python3
# -*- coding: utf-8 -*-
"""검증 성능이 가장 좋은 run을 찾아 레지스트리에 등록하고 champion 별칭을 붙인다.
사용법:
    export MLFLOW_TRACKING_URI=http://<팀서버IP>:5000
    python register_best.py --experiment team-T01-churn \
        --metric rmse_valid --mode min --model-name team-T01-model
"""
import argparse
import mlflow
from mlflow.tracking import MlflowClient

ap = argparse.ArgumentParser()
ap.add_argument("--experiment", required=True)
ap.add_argument("--metric", required=True, help="예: rmse_valid 또는 f1_valid")
ap.add_argument("--mode", choices=["min", "max"], required=True,
                help="RMSE/MAE는 min, F1/정확도는 max")
ap.add_argument("--model-name", required=True)
ap.add_argument("--note", default="", help="선정 근거 한 문단")
args = ap.parse_args()

client = MlflowClient()
exp = client.get_experiment_by_name(args.experiment)
if exp is None:
    raise SystemExit(f"[중단] 실험을 찾을 수 없습니다: {args.experiment}")

order = "ASC" if args.mode == "min" else "DESC"
runs = client.search_runs(
    experiment_ids=[exp.experiment_id],
    filter_string=f"metrics.{args.metric} != 0",
    order_by=[f"metrics.{args.metric} {order}"],
    max_results=5,
)
if not runs:
    raise SystemExit("[중단] 기록된 run이 없습니다. 먼저 학습을 실행하세요.")

print("상위 5개 run:")
for r in runs:
    print(f"  {r.info.run_name:24s} {args.metric}={r.data.metrics.get(args.metric):.4f} "
          f"params={ {k: v for k, v in r.data.params.items() if k in ('max_depth','n_estimators','model_type')} }")

best = runs[0]
print(f"\n선택: {best.info.run_name} (run_id={best.info.run_id})")

result = mlflow.register_model(
    model_uri=f"runs:/{best.info.run_id}/model",
    name=args.model_name,
)
print(f"등록 완료: {args.model_name} 버전 {result.version}")

note = args.note or (f"{args.metric}={best.data.metrics.get(args.metric):.4f} 로 "
                     f"선정 기준상 최우수. run_id={best.info.run_id}")
client.update_model_version(name=args.model_name, version=result.version, description=note)
client.set_registered_model_alias(name=args.model_name, alias="champion", version=result.version)
print(f"별칭 부여: {args.model_name}@champion → 버전 {result.version}")
PYEOF

python register_best.py \
  --experiment team-T<팀번호>-<주제> \
  --metric rmse_valid --mode min \
  --model-name team-T<팀번호>-model \
  --note "검증 RMSE 최저(398.2), 학습-검증 격차 12로 과적합 징후 없음, pkl 18MB, 학습 24초"
```

등록한 모델을 이름으로 다시 불러올 수 있는지 확인합니다. **7주차 API가 이 한 줄을 씁니다.**

```bash
python - << 'EOF'
import mlflow, pandas as pd
pipe = mlflow.sklearn.load_model("models:/team-T01-model@champion")   # 팀 모델명으로 수정
print(type(pipe))
print("불러오기 성공 — 7주차 FastAPI가 이 한 줄로 모델을 가져옵니다")
EOF
```

### 팀 프로젝트 연결

- 오늘 만든 실험 화면은 **최종 산출물 루브릭 "실험 관리·재현성" 5점**의 직접 증거입니다. 5건 이상이 만점 기준입니다.
- 오늘 등록한 `models:/team-T<팀번호>-model@champion` 은 **7주차 FastAPI가 로드할 주소**입니다. 팀 README에 그대로 적어 두세요.
- 오늘 작성한 **모델 선정 기준**은 15주차 최종 발표의 "왜 이 모델인가" 슬라이드가 됩니다.
- 8주차 M2 중간 시연에서 "베이스라인 성능"을 물으면 MLflow 화면을 띄우면 됩니다.

### 순회 지도 포인트

1. **팀원이 서로 다른 실험 이름을 쓰고 있지 않은가.** 가장 흔한 사고입니다. `team-T01`, `T01-churn`, `churn` 세 개로 흩어지면 비교가 안 됩니다. 발견 즉시 하나로 통일시키고, 이미 흩어진 run은 UI에서 실험을 옮기게 합니다.
2. **`log_metric`에 학습 성능이 빠져 있지 않은가.** `rmse_valid`만 기록한 팀은 과적합을 볼 수 없습니다. `rmse_train`을 반드시 추가하게 합니다.
3. **선정 기준을 결과 본 뒤에 쓰고 있지 않은가.** 순회 시 "기준 먼저 보여 주세요"라고 물어 종이/문서를 확인합니다. 아직 안 쓴 팀은 실험을 멈추고 기준부터 쓰게 합니다. 이게 오늘 NCS 평가의 핵심 행동입니다.

---

## 5. 체크포인트 (제출물)

| # | 제출물 | 형식 | 배점 |
|---|---|---|---|
| 1 | 공통 예제 실험 5건이 `rmse_valid` 기준 정렬된 MLflow UI 표 (URL 바에 IP:5000 보이게) | 스크린샷 | 0.5 |
| 2 | `rmse_train` vs `rmse_valid` 를 `max_depth`에 대해 그린 Compare 차트 + 과적합 시작 지점 한 문장 | 스크린샷 + 텍스트 | 0.5 |
| 3 | 팀 실험 8건 이상이 보이는 실험 목록 화면 | 스크린샷 | 0.4 |
| 4 | **모델 선정 기준 3항목**(주 지표 / 동점 판정 / 탈락 조건) | 텍스트 | 0.3 |
| 5 | 레지스트리 등록 모델명 + 버전 + `champion` 별칭 + description(선정 근거) 화면 | 스크린샷 | 0.3 |
| 6 | S3 `mlflow-artifacts/` 에 모델 아티팩트가 쌓인 콘솔 화면 | 스크린샷 | 0.2 |
| 7 | 정리 후 보안그룹 인바운드 규칙 화면 (5000 규칙이 삭제되었거나 `/32`로 제한됨) | 스크린샷 | 0.3 |

### 평가 기준 (NCS 수행준거 연계)

- `2001070306_19v1.3` **인공지능 인자 조율하기** — 조율 대상 하이퍼파라미터와 탐색 범위를 사전에 정의하고, 값을 체계적으로 변경한 실험을 5건 이상 수행해 각 실험의 파라미터와 결과를 빠짐없이 기록했는가.
- `2001070307_19v1.2` **인공지능 모델 선정 기준 정하기** — 주 지표, 동점 시 판정 방법, 탈락 조건(과적합·모델 크기·학습 시간)을 **모델 선정 이전에** 문서로 정의했는가.
- `2001070307_19v1.3` **인공지능 학습 결과 검증하기** — 학습 성능과 검증 성능을 함께 기록·비교해 과적합 여부를 판단하고, 그 판단 근거를 화면 또는 문장으로 제시했는가.
- `2001070307_19v1.4` **인공지능 최적화 모델 선정하기** — 정의한 선정 기준을 적용해 최적 모델 1건을 선정하고, 탈락한 후보 대비 우위를 정량적 근거로 설명했는가.
- `2001070308_19v1.1` **인공지능 선정모델 구성 요소 관리하기** — 선정 모델을 이름·버전·별칭 체계로 레지스트리에 등록하고, 학습 데이터 경로·파라미터·지표·코드 버전이 함께 조회 가능한 상태로 관리했는가.

---

## 6. 정리 & 다음 주 예고 (15분)

**오늘 배운 것 3줄 요약**

1. 재현성은 기억력의 문제가 아니라 **기록 구조의 문제**다. 학습 코드 안에서 자동으로 남기게 만든다.
2. 실험 하나는 **파라미터·메트릭·아티팩트·코드 버전** 네 가지로 기술된다. 미리 정하면 파라미터, 돌려봐야 알면 메트릭.
3. 모델은 파일 경로가 아니라 **이름과 별칭**으로 부른다. 그래야 서빙 코드를 안 고치고 모델만 바꿀 수 있다.

**다음 주 미리보기 한 문장**

> "오늘 여러분은 '이 모델이 제일 좋다'까지 왔습니다. 그런데 그 모델은 지금 **여러분 EC2 안에서만** 돕니다. 다른 사람 컴퓨터에 옮기면 파이썬 버전이 달라서, scikit-learn 버전이 달라서 안 돕니다. 다음 주에는 **환경을 통째로 상자에 담아 옮기는 기술**, Docker를 배웁니다. 이 상자가 7주차에 여러분의 API가 됩니다."

**리소스 정리 타임 (5분) 체크 항목**

```bash
# 1) MLflow 서버 종료
tmux kill-session -t mlflow                 # tmux로 띄운 경우
# 또는
kill $(cat ~/mlflow/server.pid)             # nohup으로 띄운 경우
# 또는
sudo systemctl stop mlflow                  # systemd로 등록한 경우

# 2) ⚠ 보안그룹 5000 인바운드 규칙 회수 (오늘 가장 중요한 정리 항목)
aws ec2 revoke-security-group-ingress \
  --group-id <보안그룹ID> \
  --protocol tcp --port 5000 --cidr <오늘_열었던IP>/32 \
  --region ap-northeast-2

# 3) 남아 있는 5000 규칙이 없는지 확인 (출력이 비어 있어야 정상)
aws ec2 describe-security-groups \
  --group-ids <보안그룹ID> --region ap-northeast-2 \
  --query "SecurityGroups[].IpPermissions[?FromPort==\`5000\`]" --output json

# 4) EC2 인스턴스 중지 (Terminate 아님)
aws ec2 stop-instances --instance-ids <인스턴스ID> --region ap-northeast-2
```

- [ ] MLflow 서버 프로세스가 종료되었는가 (`ps aux | grep mlflow` 결과에 서버가 없음)
- [ ] **보안그룹 5000 인바운드 규칙을 삭제했는가.** 다음 주에도 쓸 학생은 삭제 대신 **소스가 본인 IP `/32`로 제한되어 있는지** 확인 — `0.0.0.0/0`이면 즉시 수정
- [ ] SQLite 파일(`~/mlflow/mlflow.db`)이 남아 있는가 (지우면 실험 기록이 전부 사라집니다. **지우지 마세요**)
- [ ] S3 `mlflow-artifacts/` 용량 확인 (`aws s3 ls s3://mlops-2026-<학번>/mlflow-artifacts/ --recursive --summarize --human-readable | tail -3`). 1GB를 넘으면 실패한 대형 모델 아티팩트를 정리
- [ ] EC2 인스턴스가 `stopped` 인가 — 조교 1:1 확인

> ⚠ **비용 메모**: MLflow 서버 자체는 무료지만, 아티팩트가 S3에 쌓입니다. RandomForest pkl 18MB × 30 run = 약 540MB, S3 표준 기준 월 $0.02 수준이라 무시할 만합니다. **다만 `n_estimators`를 1000 이상으로 올리면 pkl 하나가 수백 MB가 됩니다.** 실험할 때 크기를 의식하세요.

---

## 7. 과제

**(1) 개인 과제 — 실험 3건 추가 + 해석 (제출: 다음 주 수업 시작 전)**

본인 팀 실험에 3건을 더 추가하되, 이번에는 **하이퍼파라미터가 아니라 피처를 바꿉니다.**
- 실험 1: 전체 피처
- 실험 2: 피처 중요도 상위 5개만
- 실험 3: 본인이 "이건 없어도 될 것 같다"고 판단한 컬럼 1개를 제외

MLflow 화면 캡처와 함께 아래 두 문장을 제출합니다.
- "피처를 줄였을 때 성능이 (올랐다 / 거의 같았다 / 떨어졌다). 그 이유는 ___라고 생각한다."
- "이 결과가 우리 팀 문제정의서의 입력 항목에 주는 시사점은 ___이다."

**(2) 팀 과제 — README 갱신**

팀 저장소 README에 다음 4줄을 추가합니다. 14주차 코칭 때 이 README로 재현성을 점검합니다.

```markdown
## 실험 관리
- MLflow 서버: http://<팀서버IP>:5000 (실험명: team-T01-churn)
- 등록 모델: models:/team-T01-model@champion (버전 3)
- 선정 기준: 주 지표 F1 / 동점 시 학습시간 / 과적합 격차 0.15 초과 탈락
- 재현 방법: `export MLFLOW_TRACKING_URI=... && python train_team_mlflow.py --bucket ... --max-depth 16`
```

**(3) 읽어올 것**

다음 주 Docker 실습을 위해, 본인 EC2의 디스크 여유를 확인해 오세요. **10GB 미만이면 다음 주에 반드시 빌드가 실패합니다.**

```bash
df -h /
du -sh ~/mlops ~/mlflow
```

10GB 미만인 학생은 다음 주 수업 전에 조교에게 알립니다. (불필요한 CSV·pkl 정리 또는 EBS 볼륨 확장 안내)

---

## 부록 A. 명령어 치트시트

> 1페이지로 인쇄해 배포. `<학번>`, `<보안그룹ID>`, `<인스턴스ID>`, `<퍼블릭IP>`는 본인 값으로 치환.

```bash
# ── 0. 접속 · 환경 ───────────────────────────────────────
ssh -i <키파일경로> ubuntu@<퍼블릭IP>
cd ~/mlops && source .venv/bin/activate
pip install -r requirements-week05.txt
mlflow --version                              # 2.16.2 확인

# ── 1. 보안그룹 5000 열기 / 닫기 ─────────────────────────
curl -s https://checkip.amazonaws.com          # 내 공인 IP 확인 (노트북에서)
aws ec2 authorize-security-group-ingress --group-id <보안그룹ID> \
  --protocol tcp --port 5000 --cidr <내IP>/32 --region ap-northeast-2
aws ec2 revoke-security-group-ingress --group-id <보안그룹ID> \
  --protocol tcp --port 5000 --cidr <내IP>/32 --region ap-northeast-2
aws ec2 describe-security-groups --group-ids <보안그룹ID> --region ap-northeast-2 \
  --query "SecurityGroups[].IpPermissions[?FromPort==\`5000\`]" --output json

# ── 2. MLflow 서버 (tmux) ────────────────────────────────
tmux new -s mlflow                             # 새 화면 만들고 진입
#   ... 서버 실행 후 Ctrl+b 그리고 d 로 빠져나오기
tmux ls                                        # 살아 있는 세션 목록
tmux attach -t mlflow                          # 다시 들어가기
tmux kill-session -t mlflow                    # 종료

mlflow server \
  --backend-store-uri sqlite:////home/ubuntu/mlflow/mlflow.db \
  --default-artifact-root s3://mlops-2026-<학번>/mlflow-artifacts/ \
  --host 0.0.0.0 --port 5000

# ── 3. MLflow 서버 (nohup 대안) ──────────────────────────
nohup ~/mlops/.venv/bin/mlflow server --backend-store-uri sqlite:////home/ubuntu/mlflow/mlflow.db \
  --default-artifact-root s3://mlops-2026-<학번>/mlflow-artifacts/ \
  --host 0.0.0.0 --port 5000 > ~/mlflow/server.log 2>&1 &
echo $! > ~/mlflow/server.pid
tail -f ~/mlflow/server.log
kill $(cat ~/mlflow/server.pid)

# ── 4. systemd (재부팅 대응) ─────────────────────────────
sudo systemctl daemon-reload
sudo systemctl enable --now mlflow
sudo systemctl status mlflow --no-pager
sudo systemctl restart mlflow
journalctl -u mlflow -n 50 --no-pager

# ── 5. 학습 · 실험 ───────────────────────────────────────
export MLFLOW_TRACKING_URI=http://127.0.0.1:5000
export AWS_DEFAULT_REGION=ap-northeast-2
echo $MLFLOW_TRACKING_URI                       # 비어 있으면 ./mlruns 로 저장됨
python train_mlflow.py --bucket mlops-2026-<학번> --max-depth 12
./sweep.sh mlops-2026-<학번>

# ── 6. 레지스트리 ────────────────────────────────────────
python register_best.py --experiment <실험명> --metric rmse_valid --mode min \
  --model-name team-T01-model --note "선정 근거"
python -c "import mlflow; m=mlflow.sklearn.load_model('models:/team-T01-model@champion'); print(type(m))"

# ── 7. 진단 ──────────────────────────────────────────────
aws sts get-caller-identity                     # 자격 증명 확인
aws s3 ls s3://mlops-2026-<학번>/mlflow-artifacts/ --region ap-northeast-2
ps aux | grep mlflow                            # 중복 프로세스 확인
ss -tlnp | grep 5000                            # 5000 포트를 누가 잡고 있나
```

**MLflow 파이썬 5줄 요약**

```python
mlflow.set_experiment("team-T01-churn")            # 폴더 지정
with mlflow.start_run(run_name="rf_d16"):          # 실행 1건 시작
    mlflow.log_param("max_depth", 16)              # 미리 정한 값
    mlflow.log_metric("f1_valid", 0.812)           # 돌려봐야 아는 값
    mlflow.sklearn.log_model(pipe, "model")        # Pipeline 통째로
```

---

## 부록 B. 용어 정리

| 용어 | 뜻 | 한 줄 설명 |
|---|---|---|
| 재현성 (reproducibility) | 다시 만들어낼 수 있음 | 같은 코드·데이터·설정으로 같은 결과가 나오는 성질 |
| 실험 추적 (experiment tracking) | 실험 자동 기록 | 학습할 때마다 설정값과 결과를 자동으로 저장하는 것 |
| MLflow | 오픈소스 실험 관리 도구 | 서버 하나 띄우면 실험 기록·비교·모델 등록이 된다 |
| 런 (run) | 학습 1회 | 코드를 한 번 돌린 것. 파라미터·메트릭·아티팩트를 갖는다 |
| 실험 (experiment) | run들의 묶음 | 관련 run을 담는 폴더. `bike-baseline` 같은 이름 |
| 파라미터 (parameter) | 미리 정하는 설정값 | `max_depth=12` 처럼 사람이 고르는 값 |
| 메트릭 (metric) | 결과 숫자 | RMSE, F1처럼 돌려봐야 아는 값 |
| 아티팩트 (artifact) | 산출물 파일 | 모델 pkl, 그래프 이미지 등. S3에 보관된다 |
| 백엔드 스토어 (backend store) | 메타데이터 DB | 파라미터·메트릭 같은 작은 값을 담는 곳. 우리는 SQLite |
| 아티팩트 스토어 (artifact store) | 파일 저장소 | 큰 파일을 담는 곳. 우리는 S3 |
| SQLite | 파일 한 개짜리 데이터베이스 | 서버 설치 없이 `.db` 파일 하나로 동작. 소규모에 적합 |
| 모델 레지스트리 (model registry) | 모델 등록소 | 모델에 이름과 버전을 붙여 관리하는 곳 |
| 스테이지 (stage) | 모델 버전 딱지 | None/Staging/Production/Archived. MLflow 2.9+에서는 별칭 권장 |
| 별칭 (alias) | 버전에 붙이는 이름표 | `@champion` 처럼 특정 버전을 이름으로 가리킨다 |
| 하이퍼파라미터 | 사람이 정하는 학습 설정 | 학습으로 배우지 않고 사람이 고르는 값 |
| 과적합 (overfitting) | 과하게 맞춰짐 | 학습 성능은 좋아지는데 검증 성능이 나빠지는 상태 |
| tmux | 터미널 다중화 도구 | SSH를 끊어도 그 안의 프로그램이 계속 돌게 해준다 |
| nohup | 끊김 무시 실행 | 터미널 종료 신호를 무시하고 백그라운드로 실행 |
| systemd | 리눅스 서비스 관리자 | 부팅 시 자동 실행, 죽으면 자동 재시작을 담당 |
| 보안그룹 (security group) | 인스턴스 방화벽 | 어떤 IP가 어떤 포트로 들어올 수 있는지 정하는 규칙 |
| CIDR `/32` | IP 한 개 지정 | `1.2.3.4/32`는 그 주소 하나만 허용한다는 뜻 |

---

## 부록. AWS 화면과 공식 문서

- EC2 콘솔: <https://console.aws.amazon.com/ec2/>
- S3 콘솔: <https://console.aws.amazon.com/s3/>
- 화면에서 확인: MLflow 서버 인스턴스 상태, 5000 포트의 소스 제한, 아티팩트 버킷 경로
- 보안그룹 규칙 공식 문서: <https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/security-group-rules.html>
- MLflow 공식 문서: <https://mlflow.org/docs/latest/ml/tracking/>

```mermaid
flowchart LR
    A[학습 코드] --> B[MLflow Tracking Server]
    B --> C[(SQLite 메타데이터)]
    B --> D[S3 아티팩트]
    B --> E[모델 레지스트리]
```+

---

## 실제 AWS 콘솔 화면 실습 가이드 (5주차)

> 2026-08-24 서울 리전 콘솔에서 직접 캡처했다. 계정 번호가 보이는 오른쪽 상단은 제외했다. 화면 모양이 바뀌면 메뉴 경로와 확인 항목을 기준으로 찾는다.

### SageMaker Studio 시작 화면

![SageMaker Studio 시작 화면](../assets/aws-console/week05/01-sagemaker-studio-landing.png)

- 콘솔 경로: **SageMaker AI → Studio**
- 확인할 것: 도메인과 시작 방법 확인
- [같은 화면 열기](https://console.aws.amazon.com/sagemaker/home?region=ap-northeast-2#/studio-landing)

### SageMaker 도메인 목록

![SageMaker 도메인 목록](../assets/aws-console/week05/02-sagemaker-domains.png)

- 콘솔 경로: **SageMaker AI → Domains**
- 확인할 것: MLflow를 사용할 도메인 상태 확인
- [같은 화면 열기](https://console.aws.amazon.com/sagemaker/home?region=ap-northeast-2#/studio)

### SageMaker 역할 관리자

![SageMaker 역할 관리자](../assets/aws-console/week05/03-sagemaker-role-manager.png)

- 콘솔 경로: **SageMaker AI → Role Manager**
- 확인할 것: MLflow 활동 권한이 있는 역할 확인
- [같은 화면 열기](https://console.aws.amazon.com/sagemaker/home?region=ap-northeast-2#/role-manager-landing)

### SageMaker MLflow 진입 화면

![SageMaker MLflow 진입 화면](../assets/aws-console/week05/04-sagemaker-mlflow-route.png)

- 콘솔 경로: **SageMaker AI → Dashboard → MLflow tracking servers**
- 확인할 것: 추적 서버 목록 진입 위치 확인
- [같은 화면 열기](https://console.aws.amazon.com/sagemaker/home?region=ap-northeast-2#/dashboard/mlflow)

### 실험 결과의 모델 관리 진입 화면

![실험 결과의 모델 관리 진입 화면](../assets/aws-console/week05/05-sagemaker-model-registry.png)

- 콘솔 경로: **SageMaker AI → Deployable models**
- 확인할 것: 모델 버전 관리 위치 확인
- [같은 화면 열기](https://console.aws.amazon.com/sagemaker/home?region=ap-northeast-2#/models)

### MLflow용 IAM 역할

![MLflow용 IAM 역할](../assets/aws-console/week05/06-iam-roles-mlflow.png)

- 콘솔 경로: **IAM → Roles**
- 확인할 것: S3·SageMaker 권한이 있는 역할 확인
- [같은 화면 열기](https://console.aws.amazon.com/iam/home#/roles)

### MLflow 아티팩트 저장소용 S3

![MLflow 아티팩트 저장소용 S3](../assets/aws-console/week05/07-s3-artifact-store.png)

- 콘솔 경로: **S3 → Create bucket**
- 확인할 것: 추적 서버와 같은 리전 확인
- [같은 화면 열기](https://s3.console.aws.amazon.com/s3/bucket/create?region=ap-northeast-2)

### MLflow 관련 로그 그룹

![MLflow 관련 로그 그룹](../assets/aws-console/week05/08-cloudwatch-mlflow-logs.png)

- 콘솔 경로: **CloudWatch → Log groups**
- 확인할 것: SageMaker 관련 로그 확인
- [같은 화면 열기](https://console.aws.amazon.com/cloudwatch/home?region=ap-northeast-2#logsV2:log-groups)

### CloudTrail 이벤트 기록

![CloudTrail 이벤트 기록](../assets/aws-console/week05/09-cloudtrail-event-history.png)

- 콘솔 경로: **CloudTrail → Event history**
- 확인할 것: 인프라 변경 주체와 시간 확인
- [같은 화면 열기](https://console.aws.amazon.com/cloudtrail/home?region=ap-northeast-2#/events)

### MLflow CLI 준비 화면

![MLflow CLI 준비 화면](../assets/aws-console/week05/10-cloudshell-mlflow-cli.png)

- 콘솔 경로: **CloudShell**
- 확인할 것: SageMaker CLI와 서울 리전 확인
- [같은 화면 열기](https://ap-northeast-2.console.aws.amazon.com/cloudshell/home?region=ap-northeast-2)
