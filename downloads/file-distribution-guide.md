# 클라우드 MLOps 학생 파일 배포·수령 가이드

목적: 학생이 매주 “무엇을 어디서 받고, 어디에 풀고, 무엇을 수정하고, 무엇을 제출하는지” 장표만 보고 수행할 수 있게 한다.

---

## 1. 배포 채널을 세 곳으로 통일한다

| 채널 | 배포 내용 | 학생이 사용하는 시점 |
|---|---|---|
| LMS | 주차 공지, `weekNN-student.zip`, 1페이지 체크리스트, 제출 과제 | 수업 시작 전 확인·백업 다운로드·제출 |
| 비공개 공용 S3 | 용량이 큰 CSV·모델·완성 ZIP | EC2·CloudShell에서 명령으로 다운로드 |
| 팀 Git 저장소 | 학생이 수정한 코드, 팀 프로젝트, 주차별 커밋 | 실습 중 수정·커밋·복구 |

### 한 곳만 믿지 않는 이유

- LMS는 학생에게 가장 익숙하지만 EC2로 파일을 다시 옮기는 단계가 필요하다.
- S3는 EC2에서 바로 받을 수 있지만 콘솔 로그인이나 권한 문제가 생길 수 있다.
- Git은 코드 이력에 적합하지만 CSV·모델 같은 큰 파일 배포에는 적합하지 않다.

따라서 **LMS 공지에 모든 링크를 모으고, 코드는 ZIP과 Git, 큰 파일은 S3로 배포**한다.

---

## 2. 파일 이름과 폴더를 고정한다

### 강사가 올릴 파일

```text
cloud-mlops-weekNN-student-v1.zip   # 학생 시작 파일
cloud-mlops-weekNN-done-v1.zip      # 컷오프 이후 제공하는 완성본
weekNN-checklist.pdf                # 한 페이지 확인표
```

수정 배포 시 기존 파일을 덮어쓰지 않고 `v2`, `v3`로 올린다. LMS 공지에 변경 내용을 한 줄로 적는다.

### 학생 작업 폴더

```text
~/cloud-mlops/
├── week01/
├── week02/
├── ...
└── team-project/
```

주차별 ZIP을 풀었을 때 폴더가 중첩되지 않도록 ZIP 내부 최상위 폴더는 반드시 `weekNN/` 하나로 만든다.

---

## 3. 학생이 파일을 받는 표준 절차

이 절차는 매주 PPT 앞부분에 같은 모양으로 넣는다.

### 방법 A — EC2·CloudShell에서 S3로 바로 받기(수업 기본)

```bash
mkdir -p ~/cloud-mlops
cd ~/cloud-mlops

aws s3 cp \
  s3://mlops-course-shared/releases/weekNN/cloud-mlops-weekNN-student-v1.zip \
  ./cloud-mlops-weekNN-student-v1.zip

unzip -q cloud-mlops-weekNN-student-v1.zip
cd weekNN
bash verify.sh
```

화면에 `READY: weekNN`이 나오면 실습을 시작한다.

> `mlops-course-shared`는 예시 이름이 아니라 개강 전에 강사가 실제 생성하고 학생 역할에 읽기 권한을 부여해야 하는 공용 버킷 이름이다. 실제 이름이 다르면 모든 장표와 파일을 일괄 수정한다.

### 방법 B — LMS에서 받아 업로드하기(백업)

1. LMS `NN주차 수업자료`에서 `cloud-mlops-weekNN-student-v1.zip`을 내려받는다.
2. AWS 콘솔에서 CloudShell을 연다.
3. CloudShell 상단의 **Actions → Upload file**을 선택한다.
4. ZIP을 업로드한다.
5. 아래 명령을 실행한다.

```bash
mkdir -p ~/cloud-mlops
mv ~/cloud-mlops-weekNN-student-v1.zip ~/cloud-mlops/
cd ~/cloud-mlops
unzip -q cloud-mlops-weekNN-student-v1.zip
cd weekNN
bash verify.sh
```

EC2에서 작업해야 하는 주차는 CloudShell에서 다시 S3 팀 경로에 올리거나, EC2 Session Manager 터미널에서 방법 A를 사용한다.

### 다시 받을 때

기존 작업을 덮어쓰지 않는다.

```bash
cd ~/cloud-mlops
mv weekNN weekNN-backup-$(date +%Y%m%d-%H%M)
# 그다음 ZIP을 다시 풀기
```

---

## 4. 모든 학생 ZIP에 반드시 들어갈 파일

```text
weekNN/
├── README.md              # 오늘 할 일과 순서
├── verify.sh              # 파일·명령·버전 사전 검사
├── requirements.txt       # 필요한 경우 버전 고정
├── .env.example           # 비밀 값이 아닌 변수 이름만 제공
├── starter/               # 학생이 수정할 시작 파일
├── templates/             # 작성 양식
├── sample/                # 작은 샘플 데이터만 포함
└── SUBMISSION.md           # 제출 파일명과 캡처 기준
```

### ZIP에 넣지 않는 것

- AWS 액세스 키·비밀 키·세션 토큰
- 실제 `.env`
- 개인 계정 ID, 이메일, IP, 인스턴스 ID
- 수백 MB 이상의 전체 데이터·모델
- 강사용 정답 파일과 평가표

큰 파일은 S3에서 내려받게 하고 `README.md`에 정확한 S3 URI와 파일 크기를 적는다.

---

## 5. 장표에 반드시 들어갈 파일 안내 6장

매주 다음 내용을 본격적인 실습 화면보다 먼저 배치한다.

### 장표 1 — 오늘 받을 파일

- ZIP 파일명과 버전
- 추가로 S3에서 받을 데이터·모델
- 수업 중 수정할 파일 2~4개
- 읽기만 할 파일

### 장표 2 — 어디서 받나요?

- LMS 메뉴: `NN주차 → 수업자료`
- S3 URI
- 팀 Git 저장소와 시작 태그가 있는 경우 태그명
- 강사가 실제 배포하지 않은 주소나 QR 코드를 넣지 않는다.

### 장표 3 — 내려받고 압축 풀기

- 학생이 그대로 복사할 명령
- 최종 작업 경로 `~/cloud-mlops/weekNN`
- 명령 실행 후 보여야 할 파일 트리

### 장표 4 — 시작 전 자동 확인

```bash
cd ~/cloud-mlops/weekNN
bash verify.sh
```

- 정상 출력 `READY: weekNN`
- 실패 항목별 조치: 파일 없음, Python 버전, AWS 자격 증명, Docker, 리전

### 장표 5 — 무엇을 수정하고 무엇을 제출하나요?

- `수정`·`읽기 전용`·`자동 생성`을 색으로 구분
- 제출 파일명과 화면 캡처 기준
- 개인 제출인지 팀 대표 제출인지 표시

### 장표 6 — 못 따라왔을 때 복구

- 컷오프 시각
- 완성본 ZIP 또는 Git 태그 이름
- 기존 폴더 백업 후 다시 받는 명령
- 완성본 사용 시에도 반드시 직접 확인해야 하는 체크포인트

---

## 6. 주차별 학생 배포물

아래 이름은 장표·LMS·S3·ZIP에서 동일하게 사용한다.

### 1주차

- ZIP: `cloud-mlops-week01-student-v1.zip`
- 포함: `README.md`, `verify.sh`, `hook-demo/index.html`, `sample/hello.csv`, `SUBMISSION.md`
- 공통 추가 ZIP: `cloud-mlops-team-start-kit-v1.zip`
- 팀 ZIP 포함: `TEAM_REGISTRATION.md`, `TEAM_CHARTER.md`, `ROLE_ROTATION.csv`, `PROJECT_IDEA_CARD.md`, `INDIVIDUAL_CONTRIBUTION.md`, `TOPIC_CATALOG.md`, `COURSE_ROADMAP.md`, `verify.sh`
- 학생이 수정: 없음. S3에 `hello.csv` 업로드
- 팀 작성: 팀 등록표, 팀 규칙, 역할 순환표, 예시 주제 8개 중 프로젝트 후보 2개의 카드
- 제출: 버킷 구조 화면, `aws s3 ls` 결과, 로그인·MFA 확인, 팀 파일 4개
- 완성본: 필요 없음

### 2주차

- ZIP: `cloud-mlops-week02-student-v1.zip`
- 포함: `setup_check.sh`, `requirements.txt`, `linux-practice/`, `SUBMISSION.md`
- 학생이 수정: `linux-practice/student.txt`
- S3 추가 파일: 없음
- 제출: EC2 상태, Session Manager 터미널, Python·AWS CLI 확인
- 완성본: `cloud-mlops-week02-done-v1.zip`

### 3주차

- ZIP: `cloud-mlops-week03-student-v1.zip`
- 포함: `starter/preprocess.py`, `sample/bike_raw_sample.csv`, `templates/data-card.md`, `templates/problem-definition.md`, `verify.sh`
- S3 추가 파일: `s3://mlops-course-shared/data/bike_raw.csv`
- 학생이 수정: `preprocess.py`, 데이터 카드, 문제정의서
- 제출: 전처리 코드, `raw/`·`processed/` 경로, 데이터 카드
- 완성본: `cloud-mlops-week03-done-v1.zip`

### 4주차

- ZIP: `cloud-mlops-week04-student-v1.zip`
- 포함: `requirements.txt`, `starter/train_baseline.py`, `starter/predict_sample.py`, `templates/metrics-table.md`, `verify.sh`
- S3 추가 파일: `processed/bike_processed.csv`
- 학생이 수정: `build_pipeline()`, `split_data()`, 팀 컬럼 설정
- 제출: 학습 로그, 성능표, S3 모델 경로, M1 PDF
- 완성본: `cloud-mlops-week04-done-v1.zip`

### 5주차

- ZIP: `cloud-mlops-week05-student-v1.zip`
- 포함: `requirements.txt`, `starter/train_mlflow.py`, `templates/experiment-plan.md`, `templates/model-selection.md`, `verify.sh`
- 이전 주 파일: `train_team.py`, 팀 모델 설정
- 학생이 수정: MLflow 로깅과 실험 파라미터
- 제출: Run 5건, 비교표, 등록 모델 화면
- 완성본: `cloud-mlops-week05-done-v1.zip`

### 6주차

- ZIP: `cloud-mlops-week06-student-v1.zip`
- 포함: `starter/Dockerfile`, `starter/app.py`, `requirements.txt`, `.dockerignore`, `verify.sh`
- 학생이 수정: Dockerfile 빈칸과 이미지 이름
- 제출: 빌드·실행 로그, ECR URI, 태그·digest
- 완성본: `cloud-mlops-week06-done-v1.zip`

### 7주차

- ZIP: `cloud-mlops-week07-student-v1.zip`
- 포함: `app/schemas.py`, `starter/app/main.py`, `tests/test_api.py`, `requirements.txt`, `Dockerfile`, `verify.sh`
- S3 추가 파일: `models/model.pkl`
- 학생이 수정: `/health`, `/predict`, 입력 컬럼 순서
- 제출: Swagger 200, 잘못된 입력 422, ECR `v1`
- 완성본: `cloud-mlops-week07-done-v1.zip`

### 8주차

- ZIP: `cloud-mlops-week08-student-v1.zip`
- 포함: `starter/streamlit_app.py`, `docker-compose.yml`, `.env.example`, `deploy.sh`, `smoke_test.sh`, `verify.sh`
- 이전 주 파일: 팀 API 이미지 URI
- 학생이 수정: API URL, 화면 입력 항목, 이미지 태그
- 제출: 공개 데모 URL, health 응답, Compose 상태, M2 자료
- 완성본: `cloud-mlops-week08-done-v1.zip`

### 9주차

- ZIP: `cloud-mlops-week09-student-v1.zip`
- 포함: `integration_check.sh`, `templates/end-to-end-checklist.md`, `templates/technical-debt.md`, `verify.sh`
- 학생이 수정: 팀 리소스 이름과 점검 결과
- 제출: 전 구간 체크리스트, 기술 부채 2건의 수정 전·후 증거
- 완성본: 앞 주차의 `done` ZIP과 팀 저장소를 사용

### 10주차

- ZIP: `cloud-mlops-week10-student-v1.zip`
- 포함: `starter/train_sagemaker.py`, `invoke_endpoint.py`, `cleanup_sagemaker.sh`, `templates/build-vs-managed.md`, `verify.sh`
- S3 추가 파일: 학습 입력 CSV와 ECR 이미지 URI 안내
- 학생이 수정: 역할 ARN·버킷·이미지 URI·리소스 이름
- 제출: 학습 작업, 호출 결과, 비교표, 삭제 후 0개 화면
- 완성본: `cloud-mlops-week10-done-v1.zip`

### 11주차

- ZIP: `cloud-mlops-week11-student-v1.zip`
- 포함: `starter/bedrock_converse.py`, `starter/explain_route.py`, `templates/prompt-comparison.md`, `templates/safety-check.md`, `verify.sh`
- 학생이 수정: 프롬프트와 `/explain` 연결
- 제출: 프롬프트 3종 비교, Converse 응답, 배포 API 결과
- 완성본: `cloud-mlops-week11-done-v1.zip`

### 12주차

- ZIP: `cloud-mlops-week12-student-v1.zip`
- 포함: `.github/workflows/deploy.yml`, `smoke_test.sh`, `rollback.sh`, `.env.example`, `SUBMISSION.md`, `verify.sh`
- 강사 사전 설정: GitHub OIDC IAM 역할과 저장소 변수
- 학생이 수정: 리전·ECR 저장소·배포 대상·테스트 명령
- 제출: 성공 Workflow, ECR 릴리스 태그, 배포·롤백 로그
- 완성본: `cloud-mlops-week12-done-v1.zip`

### 13주차

- ZIP: `cloud-mlops-week13-student-v1.zip`
- 포함: `put_metric.py`, `generate_test_traffic.sh`, `templates/operations-checklist.md`, `templates/incident-record.md`, `verify.sh`
- 학생이 수정: namespace·metric·threshold·담당자·조치 방법
- 제출: 로그, 대시보드, OK→ALARM→OK, 운영표
- 완성본: `cloud-mlops-week13-done-v1.zip`

### 14주차

- ZIP: `cloud-mlops-week14-student-v1.zip`
- 포함: `templates/README.md`, `templates/cross-reproduction.md`, `templates/coaching-checklist.md`, `templates/presentation-outline.md`, `resource_audit.sh`
- 학생이 수정: README 전체, 교차 재현 결과, 발표 개요
- 제출: 재현 결과, 수정 커밋, 발표 초안, 시연 영상, 리소스 감사표
- 완성본: 없음. 팀 저장소가 최종 후보본

### 15주차

- ZIP: `cloud-mlops-week15-student-v1.zip`
- 포함: `cleanup_check.sh`, `backup_before_cleanup.sh`, `templates/final-submission.md`, `templates/cost-review.md`, `templates/peer-review.md`, `templates/portfolio-readme.md`
- 학생이 수정: 최종 제출표·비용 리뷰·포트폴리오 README
- 제출: 저장소 URL, 발표자료 PDF, 시연 영상, 정리 결과, 비용 리뷰
- 완성본: 없음

---

## 7. LMS 공지 템플릿

```text
[클라우드 MLOps NN주차 수업자료]

1. 오늘 받을 파일
- cloud-mlops-weekNN-student-v1.zip
- weekNN-checklist.pdf

2. 수업 전 확인
- AWS access portal 로그인
- 서울 리전(ap-northeast-2)
- 노트북 충전 및 인증 앱

3. EC2/CloudShell에서 받기
aws s3 cp s3://mlops-course-shared/releases/weekNN/cloud-mlops-weekNN-student-v1.zip .

4. 제출
- LMS NN주차 과제방
- 마감: 수업 종료 15분 전
- 파일명: NN_<학번>_<번호>.png

5. 수정 이력
- v1: 최초 배포
```

---

## 8. 강사용 배포 전 확인표

- [ ] ZIP을 빈 폴더에 풀었을 때 `weekNN/` 하나만 생성되는가?
- [ ] `bash verify.sh`가 새 EC2 또는 CloudShell에서 통과하는가?
- [ ] README 명령을 위에서 아래로 복사해 실제 실행해 봤는가?
- [ ] S3 URI와 객체가 실제로 존재하는가?
- [ ] 학생 역할에 `GetObject` 권한이 있는가?
- [ ] ZIP과 README에 비밀 값이 없는가?
- [ ] LMS 파일명·S3 파일명·장표 파일명이 같은가?
- [ ] 시작본과 완성본이 명확히 구분되는가?
- [ ] 제출 파일명과 캡처 기준이 장표와 LMS에서 같은가?
- [ ] 수업 종료 시 중지·삭제할 리소스가 적혀 있는가?

---

## 9. 현재 준비 상태

현재 실제 파일로 준비된 것은 1주차 후킹 데모와 12주차 GitHub Actions 워크플로다. 나머지 주차는 교안 안에 코드와 양식이 있으나 학생이 바로 내려받을 ZIP으로 분리되어 있지 않다.

따라서 개강 전에는 다음 작업이 추가로 필요하다.

1. 각 교안의 코드 블록과 양식을 위 파일명으로 분리한다.
2. 주차별 `verify.sh`를 작성한다.
3. 빈 환경에서 시작본과 완성본을 각각 실행 검증한다.
4. `cloud-mlops-weekNN-student-v1.zip` 15개를 만든다.
5. 실제 공용 S3와 LMS에 업로드한 후 장표의 주소를 확정한다.
