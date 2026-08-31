# 15주차 — 최종 발표 및 마무리 (🏁 M4)

**클라우드 MLOps** · 연성대학교 · 230분 (10분 조기 종료)
NCS: `2001070208_18v1.1` 인공지능 서비스 성과기준 기획하기(부분) / `2001070208_18v1.2` 인공지능 서비스 성과 평가 방법 기획하기(부분) / `2001070302_19v1.3` 인공지능 모델 설계 검증하기(부분) / `2001070405_20v1.3` 인공지능서비스 운영품질 개선하기(부분)

> **이 회차는 실습 A/B 구성을 쓰지 않습니다.**
> 3장은 "팀 발표 세션", 4장은 "리소스 전체 정리 실습", 5장은 "회고 및 진로 연계"로 변형했습니다.
> 0장 회차 요약, 6장 체크포인트·루브릭, 7장 정리, 부록은 표준 템플릿 구조를 그대로 따릅니다.

---

## 0. 회차 요약 (강사용 1페이지)

| 항목 | 내용 |
|---|---|
| 학습목표 | 12주간의 프로젝트를 문제 → 접근 → 결과 → 배운 점 구조로 발표하고 라이브 시연을 수행하며, 학기 중 생성한 클라우드 리소스를 전부 정리하고 실제 비용을 확인·설명할 수 있다. |
| 오늘의 결과물 | ① 최종 발표 + 라이브 시연 ② 상호평가지 ③ **리소스 정리 확인 명령 실행 결과(전 항목 0건)** ④ 최종 비용 리뷰표 ⑤ 포트폴리오용 GitHub README 정리 |
| 사전 준비 | ① 발표 순서표(14주차 추첨분) 게시 ② 프로젝터·HDMI·USB-C 젠더·백업 노트북 ③ 타이머(15분/5분 벨) ④ 상호평가지 인쇄(학생 수 × 팀 수) ⑤ Cost Explorer 팀 태그별 화면 미리 준비 ⑥ 정리 확인 스크립트 `cleanup_check.sh` 배포 ⑦ **강사 계정 기준 최종 스윕 목록** 준비 |
| 학생 준비물 | 발표자료 최종 PDF(USB 포함), **시연 영상 로컬 파일**, 발표용 입력 데이터 3건, 노트북·충전기, 필기구 |
| 예상 사고 지점 | ① 프로젝터 연결 실패·해상도 문제로 첫 팀에서 5분 손실 ② 인스턴스 재기동으로 퍼블릭 IP가 바뀌어 데모 주소가 죽음 ③ 정리했다고 말하지만 미연결 탄력적 IP·EBS 볼륨이 남아 있음 |

### 시간표

| 시간 | 구성 | 분 |
|---|---|---|
| 00:00–00:10 | 도입 — 발표 오리엔테이션, 상호평가지 배부, 장비 점검 | 10 |
| 00:10–02:10 | **팀 발표 세션** — 발표 15분 + 질의 5분 × 팀 수 | 120 |
| 02:10–02:30 | 휴식 | 20 |
| 02:30–03:10 | **리소스 전체 정리 실습** + 최종 비용 확인·리뷰 | 40 |
| 03:10–03:50 | **회고 및 진로 연계** — 포트폴리오 정리, 자격증·다음 학습 경로, 마무리 | 40 |
| **합계** | | **230** |

> 발표 세션 130분 = 오리엔테이션 10분 + 발표 120분.
> **팀 수에 따른 발표 시간 조정 (반드시 첫 안내 때 공지)**

| 팀 수 | 팀당 시간 | 구성 | 발표 총계 |
|---|---|---|---|
| 6팀 | 20분 | 발표 15 + 질의 5 | 120분 |
| 7팀 | 17분 | 발표 13 + 질의 4 | 119분 |
| **8팀 이상** | **15분** | **발표 12 + 질의 3** | 120분(8팀) |

> **8팀 이상이면 발표를 12분으로 단축**하고, 질의는 3분으로 줄입니다. 이 경우 발표자료 구성표(14주차 부록 C)에서 4장(데이터)과 12장(한계)을 각 30초씩 압축하도록 사전 공지합니다.

---

## 1. 도입 (10분)

### 발표 오리엔테이션 (강사 멘트)

> 오늘은 12주 동안 만든 것을 남에게 보여주는 날입니다. 시작하기 전에 네 가지만 공지하겠습니다.
>
> **첫째, 시간입니다.** 발표 15분, 질의 5분입니다. 13분에 한 번, 15분에 두 번 벨을 울립니다. 15분이 되면 슬라이드가 남아 있어도 마무리하세요. 시간 관리도 발표 평가 항목입니다.
>
> **둘째, 데모입니다.** 지난주에 말씀드린 대로 백업 3단을 준비하셨죠. 라이브가 안 되면 **3초 안에** 로컬 실행이나 영상으로 넘어가세요. "어? 왜 안 되지"를 반복하는 순간부터 감점입니다. **준비된 대안을 꺼내는 건 감점이 아닙니다.**
>
> **셋째, 상호평가입니다.** 지금 나눠드리는 평가지에 **다른 팀 발표를 들으면서 바로 적으세요.** 나중에 몰아서 쓰면 아무것도 기억 안 납니다. 그리고 이 평가는 실명이 아니라 집계로만 반영됩니다. 솔직하게 적으세요. 다만 **점수만 적지 말고 "잘한 점 1개, 궁금한 점 1개"를 반드시 문장으로** 적어주세요. 그게 발표 팀에게 돌아가는 선물입니다.
>
> **넷째, 질의응답은 팀 전원이 받습니다.** 발표자가 아니어도 자기가 맡은 부분 질문은 자기가 답하세요. 이건 "각자 최소 1회는 전 구간을 혼자 돌려보기"를 확인하는 자리이기도 합니다.
>
> 그리고 오늘 발표가 끝이 아닙니다. 휴식 후에 **여러분이 학기 내내 만든 AWS 리소스를 전부 지우고, 실제로 얼마를 썼는지 다 같이 확인합니다.** 이것도 수업입니다. 만드는 것만큼 지우는 것도 실력입니다.

### 장비 점검 (5분, 첫 팀 발표 전에 반드시)

```bash
# 발표 순서 1~3번 팀은 지금 미리 확인
curl -s -o /dev/null -w "HTTP %{http_code}  응답 %{time_total}초\n" \
  http://<서비스 IP>:8000/health

# 로컬 백업 컨테이너 미리 기동 (전 팀 지금 실행)
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"
```

- [ ] 프로젝터 연결 확인 (HDMI / USB-C 젠더 / 해상도 1920×1080)
- [ ] 발표자료 PDF가 USB에서도 열리는지
- [ ] **시연 영상이 인터넷 없이 재생되는지**
- [ ] 인스턴스를 켠 뒤 **퍼블릭 IP가 바뀌지 않았는지** — 바뀌었으면 발표자료의 주소를 지금 수정
- [ ] 타이머 준비

---

## 2. 팀 발표 세션 (120분)

### 2-1. 운영 절차

| 단계 | 시간 | 진행 |
|---|---|---|
| 전환 | 1분 | 다음 팀이 노트북 연결, 앞 팀은 자리로. **연결하는 동안 앞 팀 질의를 이어서 진행**해 공백을 없앤다 |
| 발표 | 15분 | 13분 벨 1회, 15분 벨 2회 |
| 질의 | 5분 | 강사 질문 2개 → 학생 질문 2~3개 순 |
| 기록 | (질의 중) | 청중은 상호평가지 작성 |

### 2-2. 발표 순서 판 (강사 게시용)

| 순번 | 팀번호 | 프로젝트명 | 발표 시작 | 발표자 | 데모 방식 |
|---|---|---|---|---|---|
| 1 | | | 00:10 | | ☐ 라이브 ☐ 로컬 ☐ 영상 |
| 2 | | | 00:30 | | ☐ 라이브 ☐ 로컬 ☐ 영상 |
| 3 | | | 00:50 | | ☐ 라이브 ☐ 로컬 ☐ 영상 |
| 4 | | | 01:10 | | ☐ 라이브 ☐ 로컬 ☐ 영상 |
| 5 | | | 01:30 | | ☐ 라이브 ☐ 로컬 ☐ 영상 |
| 6 | | | 01:50 | | ☐ 라이브 ☐ 로컬 ☐ 영상 |

### 2-3. 강사 질의 문항 은행 (팀당 2개 선택)

**파이프라인·재현성**
- 지금 서버에 떠 있는 이미지가 어느 커밋으로 만들어졌는지 말해줄 수 있나요?
- 제가 오늘 처음 이 저장소를 받았다면, README만 보고 몇 분 만에 학습을 돌릴 수 있을까요?
- 학습에 쓴 전처리와 서빙에서 쓰는 전처리가 같다는 걸 어떻게 보장했나요?

**모델·의사결정**
- 후보 모델 중 이걸 고른 이유가 뭔가요? 두 번째 후보 대비 어떤 점을 포기했나요?
- 이 평가지표를 고른 이유는요? 다른 지표로 보면 결과가 달라지나요?
- 베이스라인이 얼마였고, 그 대비 얼마나 나아졌나요?

**운영**
- 이 서비스가 지금 죽으면 여러분은 몇 분 안에 알 수 있나요?
- 6개월 뒤에도 이 모델이 잘 맞을까요? 아니라면 언제 다시 학습하나요?
- 사용자가 이상한 값을 보내면 어떻게 되나요? 지금 보여줄 수 있나요?

**팀·개인**
- (발표자가 아닌 팀원에게) 이 부분은 누가 만들었나요? 설명해줄 수 있어요?
- 12주 중에 가장 오래 막혔던 지점이 어디였나요? 어떻게 뚫었나요?

### 2-4. 데모 실패 시 진행 규칙 (강사·학생 공통)

| 상황 | 조치 | 감점 |
|---|---|---|
| 라이브 호출 타임아웃 | **10초 안에** 로컬 컨테이너로 전환 | 없음 |
| 로컬도 실패 | 시연 영상 재생 | 없음 |
| 영상도 없음 | 발표 계속 진행, 시연 항목 미수행 처리 | 발표 평가 -3점 |
| 원인 규명에 매달려 1분 이상 지연 | 강사가 개입해 다음으로 진행시킴 | 발표 평가 -1점 |

> **공지 문구**: "준비한 대안을 꺼내는 것은 감점이 아닙니다. 대안 없이 시간을 쓰는 것이 감점입니다."

### 2-5. 청중 규칙

- 발표 중 노트북은 **상호평가지 작성 용도로만** 사용
- 질문은 "왜 그렇게 했는지"를 묻는 형태로. 지적이 아니라 궁금증으로
- 마지막 팀 발표까지 전원 참석. 상호평가지 미제출은 참여 점수 감점

---

## 3. 리소스 전체 정리 실습 (40분)

**목표** 학기 중 생성한 모든 과금 리소스를 삭제하고, 삭제되었음을 명령으로 증명하며, 실제 사용 비용을 확인해 설명한다.

**과제 지시문** (학생에게 그대로 읽어줍니다)

> 지금부터 40분 동안 여러분이 만든 것을 전부 지웁니다. 이건 뒷정리가 아니라 **오늘 수업의 후반부 내용**입니다.
> 실무에서 클라우드 사고의 가장 흔한 형태가 뭔지 아세요? 해킹이 아닙니다. **"쓰다가 안 쓰는데 안 지운 것"** 입니다. 프로젝트가 끝났는데 GPU 인스턴스가 6개월 동안 돌아서 수천만 원이 나온 사례가 실제로 있습니다.
> 그러니 오늘은 지우는 것으로 끝내지 않습니다. **지워졌다는 걸 명령어로 증명**합니다. "지웠어요"는 증거가 아닙니다. `describe` 결과가 비어 있는 화면이 증거입니다.
> 순서대로 갑니다. **비싼 것부터, 그리고 지우면 되살릴 수 없는 것은 백업을 먼저 확인하고.**

### 3-1. ⚠ 지우기 전에 반드시 백업할 것 (5분)

> **AWS에서 지우면 끝입니다. 포트폴리오에 쓸 자료를 먼저 챙기세요.**

- [ ] MLflow 실험 비교 화면 캡처 (실험 5건 이상이 보이게)
- [ ] CloudWatch 대시보드 전체 캡처
- [ ] 알람 수신 메일 캡처
- [ ] GitHub Actions 성공 로그 캡처
- [ ] API 응답 화면 / 데모 UI 캡처
- [ ] 최종 모델 성능 수치가 적힌 화면
- [ ] 시연 영상 파일 (로컬·드라이브 이중 보관)

```bash
# 최종 모델 파일과 실험 DB를 로컬로 내려받기 (지우기 전에)
aws s3 cp s3://<팀 버킷>/models/ ./backup_models/ --recursive --region ap-northeast-2
scp -i <키파일.pem> ubuntu@<MLflow 서버 IP>:~/mlflow.db ./backup_mlflow.db

# 저장소에 백업 폴더는 커밋하지 않는다 (.gitignore 확인)
```

### 3-2. 정리 순서 (비싼 것부터)

**① SageMaker — 엔드포인트·구성·노트북 스페이스 ⚠ 시간당 과금 1순위**

```bash
aws sagemaker list-endpoints --region ap-northeast-2 --output table
aws sagemaker delete-endpoint --region ap-northeast-2 --endpoint-name <엔드포인트명>

aws sagemaker list-endpoint-configs --region ap-northeast-2 --output table
aws sagemaker delete-endpoint-config --region ap-northeast-2 --endpoint-config-name <구성명>

aws sagemaker list-apps --region ap-northeast-2 --output table
aws sagemaker delete-app --region ap-northeast-2 \
  --domain-id <도메인ID> --user-profile-name <프로필명> \
  --app-type JupyterLab --app-name <앱명>
```

- 확인 포인트: `list-endpoints` 결과가 `{"Endpoints": []}` 또는 빈 표.

**② EC2 인스턴스 종료 (Terminate)**

```bash
# 우리 팀 인스턴스 전부 확인
aws ec2 describe-instances --region ap-northeast-2 \
  --filters "Name=tag:Team,Values=<팀번호>" "Name=instance-state-name,Values=running,stopped" \
  --query "Reservations[].Instances[].[InstanceId,InstanceType,State.Name,Tags[?Key=='Name']|[0].Value]" \
  --output table

# 종료 (되돌릴 수 없음 — 백업 확인 후)
aws ec2 terminate-instances --region ap-northeast-2 \
  --instance-ids <인스턴스ID1> <인스턴스ID2>

# 상태 확인
aws ec2 describe-instances --region ap-northeast-2 \
  --filters "Name=tag:Team,Values=<팀번호>" \
  --query "Reservations[].Instances[].[InstanceId,State.Name]" --output table
```

- 확인 포인트: 상태가 `shutting-down` → `terminated` 로 바뀐다. **`stopped`는 정리가 아닙니다.** EBS 요금이 계속 나갑니다.

**③ EBS 볼륨 — 인스턴스 종료 후 남은 것 ⚠ 학기 비용의 절반**

```bash
aws ec2 describe-volumes --region ap-northeast-2 \
  --filters "Name=status,Values=available" \
  --query "Volumes[].[VolumeId,Size,CreateTime]" --output table

aws ec2 delete-volume --region ap-northeast-2 --volume-id <VolumeId>

# 스냅샷도 확인
aws ec2 describe-snapshots --region ap-northeast-2 --owner-ids self \
  --query "Snapshots[].[SnapshotId,VolumeSize,StartTime]" --output table
aws ec2 delete-snapshot --region ap-northeast-2 --snapshot-id <SnapshotId>
```

- 확인 포인트: `status=available` 볼륨 목록이 비어야 한다.
- 참고: 예상 비용표에서 **EBS 30GB × 4개월 ≈ $12** 로 EC2 사용료($5)보다 컸습니다. 볼륨은 인스턴스를 꺼도 요금이 나갑니다.

**④ 탄력적 IP 해제 ⚠ 연결 안 된 상태가 오히려 과금**

```bash
aws ec2 describe-addresses --region ap-northeast-2 \
  --query "Addresses[].[PublicIp,AllocationId,AssociationId]" --output table

aws ec2 release-address --region ap-northeast-2 --allocation-id <AllocationId>
```

**⑤ ECR 이미지·리포지토리**

```bash
aws ecr describe-repositories --region ap-northeast-2 \
  --query "repositories[].[repositoryName,createdAt]" --output table

# 이미지가 있어도 통째로 삭제
aws ecr delete-repository --region ap-northeast-2 \
  --repository-name <리포지토리명> --force
```

**⑥ S3 버킷 — 데이터·모델·아티팩트**

```bash
# 크기 먼저 확인
aws s3 ls s3://<팀 버킷>/ --recursive --summarize --human-readable --region ap-northeast-2 | tail -3

# 비우고 삭제
aws s3 rm s3://<팀 버킷>/ --recursive --region ap-northeast-2
aws s3 rb s3://<팀 버킷> --region ap-northeast-2

# 남은 버킷 확인
aws s3 ls
```

- ⚠ **3-1의 백업을 먼저 내려받았는지 다시 확인하세요.** 여기서 지우면 복구 방법이 없습니다.

**⑦ CloudWatch — 알람·대시보드·로그 그룹**

```bash
aws cloudwatch describe-alarms --region ap-northeast-2 --query "MetricAlarms[].AlarmName" --output table
aws cloudwatch delete-alarms --region ap-northeast-2 --alarm-names <알람1> <알람2>

aws cloudwatch list-dashboards --region ap-northeast-2 --output table
aws cloudwatch delete-dashboards --region ap-northeast-2 --dashboard-names "mlops-<팀번호>"

aws logs describe-log-groups --region ap-northeast-2 --query "logGroups[].logGroupName" --output table
aws logs delete-log-group --region ap-northeast-2 --log-group-name "/mlops/<팀번호>/api"
```

**⑧ SNS 주제**

```bash
aws sns list-topics --region ap-northeast-2 --output table
aws sns delete-topic --region ap-northeast-2 --topic-arn <주제 ARN>
```

**⑨ 정리 확인 스크립트 실행 — 전 항목 0건이어야 통과**

부록 A의 `cleanup_check.sh` 를 실행하고 **출력 화면 전체를 캡처**합니다.

```bash
bash cleanup_check.sh <팀번호>
```

### 3-3. 최종 비용 확인 및 리뷰 (10분)

```bash
# 팀 태그별 학기 전체 비용 (Cost Explorer API는 us-east-1 고정)
aws ce get-cost-and-usage --region us-east-1 \
  --time-period Start=<학기시작일 YYYY-MM-DD>,End=<오늘 YYYY-MM-DD> \
  --granularity MONTHLY --metrics "UnblendedCost" \
  --group-by Type=TAG,Key=Team \
  --output json

# 서비스별로 어디에 돈이 갔는지
aws ce get-cost-and-usage --region us-east-1 \
  --time-period Start=<학기시작일>,End=<오늘> \
  --granularity MONTHLY --metrics "UnblendedCost" \
  --group-by Type=DIMENSION,Key=SERVICE \
  --output table
```

**팀별 비용 리뷰표를 작성해 제출합니다.**

| 항목 | 예상(교육계획서) | 우리 팀 실제 | 차이 | 차이가 난 이유 |
|---|---|---|---|---|
| EC2 | 강사 예산표 | | | |
| EBS | 강사 예산표 | | | |
| S3 / ECR / CloudWatch | 강사 예산표 | | | |
| SageMaker AI | 강사 예산표 | | | |
| Bedrock | 강사 예산표 | | | |
| **합계** | **강사 예산표 합계** | | | |

**리뷰 토론 질문 (강사가 던지고 팀별로 한 줄씩 답하게 함)**
1. 우리 팀에서 가장 많은 비용이 나간 항목은 무엇이고, 그건 예상과 같았습니까?
2. 그 항목을 절반으로 줄이려면 무엇을 바꿔야 했을까요?
3. 이번 학기에 **비용이 새어나간 순간**이 있었다면 언제였습니까? (끄는 걸 잊은 날, 엔드포인트를 안 지운 날 등)

> **강사 정리 멘트**
> "여러분이 오늘 확인한 숫자는 25달러 안팎입니다. 작죠. 그런데 이걸 30명이 쓰면 750달러, 회사에서 100명이 쓰면 2,500달러입니다. 그리고 실무에서는 GPU 인스턴스 한 대가 시간당 몇 달러씩 나갑니다.
> 제가 여러분에게 남기고 싶은 건 절약 습관이 아니라 **'내가 만든 것이 돈을 쓰고 있다는 감각'** 입니다. 클라우드에서는 코드 한 줄이 요금이 됩니다. 이 감각이 있는 개발자와 없는 개발자는 5년 뒤에 완전히 다른 사람이 됩니다."

### ⚠ 여기서 막히면

| 증상 | 원인 | 조치 |
|---|---|---|
| `DependencyViolation` (볼륨 삭제 실패) | 아직 인스턴스에 붙어 있음 | 인스턴스가 `terminated` 될 때까지 대기(2~3분) 후 재시도 |
| `BucketNotEmpty` (버킷 삭제 실패) | 객체 또는 버전이 남음 | `aws s3 rm s3://<버킷>/ --recursive` 재실행. 버전 관리 버킷이면 콘솔에서 "비어 있는 버킷으로 만들기" 사용 |
| `RepositoryNotEmptyException` | ECR에 이미지 남음 | `delete-repository` 에 `--force` 추가 |
| 엔드포인트 삭제가 `Deleting`에서 멈춤 | 처리 중 | 3~5분 대기. 10분 넘으면 강사 호출 |
| `AccessDenied` (삭제 권한 없음) | 학생 IAM 권한 경계 | 강사가 대신 삭제. 목록을 종이에 적어 제출 |
| 인스턴스를 종료했는데 목록에 계속 보임 | `terminated` 상태는 1시간 정도 조회됨 | 정상. `State.Name`이 `terminated`면 과금되지 않음 |
| Cost Explorer에 오늘 비용이 안 보임 | 비용 데이터는 최대 24시간 지연 | 전일까지의 누적으로 리뷰. 강사 화면의 계정 전체 수치를 참고 |

### 순회 지도 포인트

1. **`stopped`를 정리로 착각한 팀.** 중지는 EBS 요금이 계속 나갑니다. `terminated` 인지 반드시 확인합니다.
2. **백업 없이 S3를 지우려는 팀.** 3-1 체크리스트를 다 채웠는지 먼저 묻습니다.
3. **`cleanup_check.sh` 결과를 캡처 안 한 팀.** 이게 체크포인트 제출물입니다.

---

## 4. 회고 및 진로 연계 (40분)

### 4-1. 학기 회고 (10분)

**진행 방식** — 팀별로 3분씩 발표하지 않고, **전체 원형으로 앉아 각자 한 문장씩** 돌아가며 말합니다.

> 질문은 하나입니다. **"이번 학기에 내가 처음으로 혼자 해낸 것 한 가지."**
> 규칙 — "열심히 했다", "재밌었다" 같은 소감 금지. **구체적인 행동으로** 말합니다.
> 예: "처음으로 EC2에 SSH로 들어가서 도커 컨테이너를 띄웠습니다", "YAML 오타를 로그만 보고 혼자 찾아냈습니다."

**강사 정리 멘트**

> 방금 여러분이 말한 문장들, 그게 **여러분의 이력서 문장**입니다. 진짜로요.
> 15주 전에 여러분 대부분은 클라우드 콘솔에 로그인해본 적도 없었습니다. 지금은 모델을 학습시켜서, 컨테이너에 담아서, 서버에 올려서, 자동으로 배포되게 하고, 죽으면 메일이 오게 만들 수 있습니다. **이 전 구간을 혼자 관통해본 사람은 생각보다 많지 않습니다.** 대학 4년 내내 주피터 노트북 안에서만 있다가 졸업하는 사람이 훨씬 많아요.
> 여러분이 가진 건 "AI를 안다"가 아니라 **"AI를 서비스로 만들어 본 적이 있다"** 입니다. 이건 다른 이야기입니다.

### 4-2. 포트폴리오 정리 가이드 — GitHub README 작성법 (15분)

**강의 스크립트**

> 이제 현실적인 이야기를 하겠습니다. **여러분이 오늘 지운 AWS 리소스는 다시 못 봅니다. 하지만 GitHub 저장소는 영원히 남습니다.** 그리고 채용 담당자는 AWS 콘솔을 못 보고 GitHub만 봅니다. 그러니 오늘 이후로 여러분 프로젝트의 실체는 **README 한 장**입니다.
>
> 채용 담당자가 여러분 저장소를 보는 시간이 얼마인지 아세요? **평균 30초 안팎**입니다. 그 30초에 무엇이 보이느냐가 전부입니다.
>
> 그래서 README 맨 위 세 줄이 승부처입니다. 이렇게 쓰세요.
>
> **첫 줄** — 프로젝트가 무엇인지 한 문장. "편의점 발주 담당자가 내일 발주량을 결정할 수 있게 도와주는 수요 예측 API"
> **둘째 줄** — 사용 기술 배지 한 줄. Python, scikit-learn, FastAPI, Docker, AWS, MLflow, GitHub Actions
> **셋째 줄** — **움직이는 GIF나 스크린샷 한 장.** 이게 제일 강력합니다. 서비스가 실제로 응답하는 화면이 위에 딱 있으면 나머지는 안 읽어도 믿습니다.
>
> 그 아래는 14주차 부록 C에서 쓴 그대로 두면 됩니다. 다만 **오늘 이후로 반드시 추가할 게 두 가지** 있습니다.
>
> 첫째, **"이 서비스는 현재 비용 문제로 종료되었으며, 아래 시연 영상으로 동작을 확인할 수 있습니다"** 한 줄과 영상 링크. 이거 없으면 채용 담당자가 링크를 눌렀다가 404를 보고 닫습니다. **살아 있지 않은 것을 살아 있는 척하는 것보다, 왜 내렸는지 밝히는 게 훨씬 프로답습니다.**
>
> 둘째, **"내가 한 것"** 섹션. 팀 프로젝트는 반드시 이게 필요합니다. "저는 서빙과 배포 자동화를 담당했고, FastAPI 엔드포인트 설계와 GitHub Actions 파이프라인을 작성했습니다. 커밋 기준 전체의 30%입니다." 이 한 문단이 있고 없고가 팀 프로젝트의 신뢰도를 가릅니다.
>
> 마지막으로 하나. **저장소를 공개(Public)로 전환**하세요. 비공개면 아무도 못 봅니다. 전환 전에 `.env`, 키 파일, 개인정보가 포함된 데이터가 없는지 **반드시** 확인하세요. 12주차에 말씀드린 대로입니다.

**포트폴리오 정리 체크리스트 (오늘 남은 시간에 착수)**

- [ ] README 맨 위: 한 문장 소개 + 기술 배지 + **동작 스크린샷/GIF**
- [ ] 시연 영상 링크 삽입 (서비스 종료 사유 명시)
- [ ] **"내가 한 것"** 섹션 (개인별로 각자 fork 저장소나 개인 README에)
- [ ] 아키텍처 그림 1장
- [ ] 실험 결과표 + 모델 선택 이유
- [ ] `.env` · 키 파일 · 개인정보 없는지 최종 점검
- [ ] 저장소 Public 전환
- [ ] GitHub 프로필 README에 이 프로젝트 고정(Pin)

```bash
# 공개 전환 전 최종 점검 — 비밀 문자열이 남아 있는지
git log --all -p | grep -inE "AKIA[0-9A-Z]{16}|aws_secret|password *=|-----BEGIN.*PRIVATE KEY" | head

# 저장소 크기 확인 (100MB 넘는 파일이 있으면 정리)
git count-objects -vH
```

### 4-3. 자격증 · 다음 학습 경로 (15분)

**강의 스크립트**

> 마지막으로 "그래서 다음에 뭘 하면 되나요"에 답하겠습니다. 세 갈래로 나눠서 말씀드립니다.
>
> **첫째, 자격증입니다.** 우선순위를 매겨드리겠습니다.
>
> 첫 자격증 후보는 **AWS Certified Cloud Practitioner** 또는 **AWS Certified AI Practitioner**입니다. 이후에는 진로에 따라 Solutions Architect - Associate, Developer - Associate, CloudOps Engineer - Associate를 고릅니다. MLOps 진로라면 **AWS Certified Machine Learning Engineer - Associate**가 직접 연결됩니다. Machine Learning - Specialty 시험은 2026년 3월 31일에 종료되었으므로 더 이상 권하지 않습니다. 시험 버전과 일정은 AWS Certification 공식 페이지에서 다시 확인합니다.
>
> 국내 자격으로는 **정보처리기사**가 여전히 채용 필터로 작동합니다. 그리고 **빅데이터분석기사**, **ADsP**가 데이터 직군에서 쓰입니다. 참고로 여러분이 이번 학기에 배운 내용은 NCS 세분류 다섯 개에 걸쳐 있습니다. 인공지능모델링, 인공지능서비스운영관리, 인공지능플랫폼구축, 인공지능서비스기획, 빅데이터플랫폼구축. **NCS 기반 채용에서 직무기술서를 읽을 때 이 용어들이 낯설지 않을 겁니다.** 그것만으로도 큰 차이입니다.
>
> **둘째, 다음에 배울 기술입니다.** 순서를 지켜주세요.
>
> **1순위는 SQL입니다.** 의외죠? 그런데 현장에서 MLOps 엔지니어가 가장 많이 쓰는 언어가 SQL입니다. 데이터가 다 데이터베이스에 있으니까요.
> **2순위는 워크플로 오케스트레이션.** Airflow나 Prefect입니다. 우리는 GitHub Actions로 배포만 자동화했는데, 실무에서는 "매일 새벽 3시에 데이터 받아서 전처리하고 학습하고 평가해서 좋으면 배포" 같은 흐름을 짜야 합니다. 12주차에 배운 "오케스트레이션"이 이겁니다.
> **3순위는 Kubernetes.** 지금은 서버 한 대에 컨테이너 하나였죠. 열 대에 컨테이너 백 개가 되면 이게 필요합니다. **다만 서두르지 마세요.** 도커를 충분히 쓰기 전에 쿠버네티스를 배우면 아무것도 안 남습니다.
> **4순위는 Terraform 같은 코드형 인프라(IaC).** 오늘 여러분이 손으로 지운 리소스들을, 코드로 만들고 코드로 지우는 방식입니다.
>
> **셋째, 직무 이야기입니다.** 이 과목이 닿아 있는 직무는 세 개입니다.
> **MLOps 엔지니어 / 데이터 엔지니어 / 백엔드 개발자(AI 서비스)** 입니다.
> 신입으로 바로 "MLOps 엔지니어"를 뽑는 회사는 많지 않습니다. 보통 **백엔드나 데이터 엔지니어로 들어가서 MLOps를 맡게 됩니다.** 그러니 지금은 파이썬 백엔드 실력과 클라우드 기본기를 두껍게 쌓는 게 정석입니다. 여러분은 그 두 가지를 이미 한 번씩 관통했습니다.
>
> 마지막으로 하나만 부탁드리겠습니다. **오늘 만든 저장소를 6개월 안에 한 번은 다시 열어보세요.** 그때 여러분은 지금 코드가 부끄러울 겁니다. 부끄러우면 성장한 겁니다. 그때 README를 다시 쓰고 코드를 고치면, 그게 여러분의 두 번째 프로젝트가 됩니다.

**판서/슬라이드 요점**

| 갈래 | 우선순위 | 항목 |
|---|---|---|
| 자격증 | 1 | AWS Certified Cloud Practitioner (가장 가까움) |
| | 2 | AWS Certified Solutions Architect – Associate |
| | 3 | 정보처리기사 / 빅데이터분석기사 / ADsP |
| | 4 | AWS Certified Machine Learning Engineer - Associate (실무 경험 후) |
| 기술 | 1 | **SQL** — 현장에서 가장 많이 쓴다 |
| | 2 | Airflow · Prefect (워크플로 오케스트레이션) |
| | 3 | Kubernetes (Docker를 충분히 쓴 다음에) |
| | 4 | Terraform (코드형 인프라) |
| 직무 | — | MLOps 엔지니어 / 데이터 엔지니어 / 백엔드(AI 서비스) — **보통 후자 둘로 입사해 전자를 맡는다** |

**학생 질문 예상 & 답변**
- Q: 대학원을 가야 하나요? → A: 모델을 **연구**하고 싶으면 필요합니다. 모델을 **서비스로 만드는** 일이 좋았다면 대학원보다 실무 경험이 빠릅니다. 이번 학기에 어느 쪽이 재밌었는지가 답에 가깝습니다.
- Q: 개인 프로젝트를 또 하는 게 좋을까요, 이걸 다듬는 게 좋을까요? → A: **이걸 다듬는 쪽이 낫습니다.** 미완성 프로젝트 세 개보다 끝까지 간 프로젝트 하나가 훨씬 강합니다. 다듬을 때는 "다른 데이터로 갈아 끼우기"가 가장 효율적입니다. 골격은 이미 있으니까요.
- Q: AWS 계정이 없어지면 연습을 못 하는데요? → A: 개인 프리 티어 계정을 만드세요. 12개월 무료 한도가 있고, **반드시 Budgets 알람을 $1로 걸어두세요.** 오늘 배운 그대로입니다.

---

## 5. 마무리 (진행 · 4-3에 이어 남은 시간)

- **상호평가지 회수** — 전원 제출 확인
- **최종 산출물 제출 확인** — 저장소 URL, 발표자료 PDF, 시연 영상 링크, 정리 확인 캡처, 비용 리뷰표
- **강사 최종 스윕 안내** — "오늘 이후 강사가 계정 전체를 다시 훑습니다. 남아 있는 리소스가 발견되면 해당 팀에 개별 연락하겠습니다."
- **한 줄 마무리**

> "여러분은 이제 '주소를 가진 모델'을 만들어본 사람입니다. 15주 동안 수고 많았습니다."

---

## 6. 체크포인트 (제출물) · 최종 평가

| # | 제출물 | 형식 | 배점 |
|---|---|---|---|
| 1 | 최종 코드 저장소 (README 포함, Public) | URL | 루브릭 25점에 포함 |
| 2 | 발표자료 최종본 | PDF | 발표 15점에 포함 |
| 3 | **리소스 정리 확인 화면** (`cleanup_check.sh` 전 항목 0건) | 스크린샷 | 필수 (미제출 시 -2점) |
| 4 | 최종 비용 리뷰표 | 문서 | 필수 |
| 5 | 상호평가지 | 인쇄물 | 참여 점수 |

### 6-1. 최종 산출물 루브릭 (25점)

| 항목 | 배점 | 상(우수) | 중(보통) | 하(미흡) |
|---|---|---|---|---|
| **파이프라인 완성도** | **10** | 데이터 → 학습 → 서빙 전 구간이 실제로 동작하고, **제3자 환경에서 호출·시연 가능** (8~10점) | 전 구간이 동작하나 팀 내부 환경에서만 확인됨 (4~7점) | 학습만 되고 API 미동작 (0~3점) |
| **실험 관리·재현성** | **5** | MLflow 비교 실험 **5건 이상**, 모델 선택 근거 추적 가능, **README만 보고 재현 성공** (4~5점) | 실험 기록은 있으나 3건 이하이거나 재현에 추가 설명 필요 (2~3점) | 기록 없음 (0~1점) |
| **배포·운영** | **5** | 컨테이너 자동 배포(커밋 1회) + 로그·메트릭·알람 구성 + **드리프트 감시 또는 재학습 정책 명시** (4~5점) | 컨테이너 배포는 되나 자동화 또는 모니터링 중 하나가 빠짐 (2~3점) | 로컬 실행만 (0~1점) |
| **문서화·발표** | **5** | 문제정의와 **의사결정 근거**가 명확, 실패·한계를 스스로 설명 (4~5점) | 내용은 전달되나 선택 이유가 약함 (2~3점) | 코드 나열 수준 (0~1점) |

**감점 (교육계획서 7장)**
- 리소스 미정리 적발: **회당 -2점** (오늘 최종 스윕 포함)
- 개인정보·저작권 위반 데이터 사용: **프로젝트 점수 0점**
- 시연 영상 백업 미제출(14주차 필수 항목): **-2점**

### 6-2. 발표 평가표 (15점)

| 항목 | 배점 | 평가 내용 | 점수 |
|---|---|---|---|
| 문제 정의의 명확성 | 3 | "누가 어떤 상황에서 불편한가"가 구체적인가. 첫 30초에 만든 것이 전달되는가 | |
| 접근·의사결정 설명 | 3 | 기술 나열이 아니라 **선택의 이유**를 말하는가. 대안 대비 트레이드오프를 설명하는가 | |
| **라이브 시연** | 4 | 실제 입력으로 예측 결과가 나오는가. 실패 시 준비된 대안으로 즉시 전환하는가 | |
| 결과 제시 | 2 | 성능 수치가 **베이스라인 등 비교 대상과 함께** 제시되는가 | |
| 배운 점·질의응답 | 3 | 실패와 한계를 스스로 말하는가. 팀 전원이 담당 부분 질문에 답하는가 | |
| **합계** | **15** | | |

> 발표 평가 = 강사 평가 70% + 상호평가 집계 30%.
> 시간 초과(2분 이상) -1점, 데모 대안 미준비 -3점(6-2 항목 4에서 차감).

### 6-3. 상호평가 양식 (학생용 · 팀 수만큼 인쇄)

**평가자: ______ (학번 ______)  ·  평가 대상 팀: ______**

| # | 항목 | 1(미흡) | 2 | 3(보통) | 4 | 5(우수) |
|---|---|---|---|---|---|---|
| 1 | 무엇을 만들었는지 이해되었다 | ☐ | ☐ | ☐ | ☐ | ☐ |
| 2 | 왜 그렇게 만들었는지 근거가 납득되었다 | ☐ | ☐ | ☐ | ☐ | ☐ |
| 3 | 시연에서 서비스가 실제로 동작함을 확인했다 | ☐ | ☐ | ☐ | ☐ | ☐ |
| 4 | 성능·결과가 비교 가능한 형태로 제시되었다 | ☐ | ☐ | ☐ | ☐ | ☐ |
| 5 | 배운 점·한계를 솔직하게 말했다 | ☐ | ☐ | ☐ | ☐ | ☐ |
| 6 | 질문에 팀원들이 잘 답했다 | ☐ | ☐ | ☐ | ☐ | ☐ |

**서술형 (둘 다 필수)**

- **잘한 점 한 가지** (구체적으로):
  ________________________________________________

- **궁금한 점 또는 아쉬운 점 한 가지**:
  ________________________________________________

- **우리 팀이 배워가고 싶은 것 한 가지**:
  ________________________________________________

> 집계 방식: 6개 항목 평균 × 3 = 최대 15점 환산 후, 발표 평가의 30%로 반영.
> 익명 집계이며 서술형은 해당 팀에 그대로 전달합니다.

### 평가 기준 (NCS 수행준거 연계)

- **`2001070208_18v1.1` 인공지능 서비스 성과기준 기획하기(부분)**
  - 서비스의 **성과를 판단할 기준을 사전에 정의**했는가 → 문제정의서의 성공 기준(평가지표·목표치)이 발표에서 그대로 제시되는가
  - 기준이 **측정 가능한 형태**인가 → "정확도 향상" 같은 서술이 아니라 지표와 수치로 표현되었는가
- **`2001070208_18v1.2` 인공지능 서비스 성과 평가 방법 기획하기(부분)**
  - 정의한 기준에 대한 **평가 방법과 비교 대상을 설정**했는가 → 베이스라인 대비 비교 제시 여부
  - 평가 결과를 **이해관계자에게 설명 가능한 형태**로 정리했는가 → 발표자료 7장(성능)의 구성과 질의응답 대응
- **`2001070302_19v1.3` 인공지능 모델 설계 검증하기(부분)**
  - 설계한 모델이 **요구사항을 충족하는지 검증**했는가 → 학습-서빙 전처리 일치, 입력 검증, 제3자 환경 호출 성공 여부
  - 검증 결과를 근거로 **모델 설계의 타당성을 설명**할 수 있는가 → 강사 질의 "왜 이 모델인가 / 무엇을 포기했는가"에 대한 응답
- **`2001070405_20v1.3` 인공지능서비스 운영품질 개선하기(부분)**
  - 운영 중 확인된 **품질 문제를 개선하고 결과를 확인**했는가 → 14주차 교차 테스트 지적사항 반영 및 재현 성공
  - 지속적 품질 유지를 위한 **후속 계획을 제시**했는가 → 발표 12장(한계와 다음 단계), README 9장, 재학습 정책

---

## 7. 정리 & 마무리

**학기 전체 3줄 요약**
1. 모델은 노트북 안에서 끝나지 않는다. 데이터·학습·실험기록·컨테이너·API·배포·모니터링이 하나의 줄기로 이어져야 서비스가 된다.
2. "내 컴퓨터에선 되는데요"를 깨는 것이 이 과목의 절반이다. 컨테이너·README·자동 배포가 그 도구다.
3. 클라우드에서는 **만드는 것만큼 지우는 것이 실력**이다. 코드 한 줄이 요금이 된다.

**과제 (학기 종료 후 · 성적과 무관한 권장 사항)**
1. 저장소를 **Public으로 전환**하고 GitHub 프로필에 Pin 한다. 전환 전 비밀 정보 최종 점검 필수.
2. README 맨 위에 **한 문장 소개 + 동작 스크린샷 + 시연 영상 링크**를 넣는다.
3. 개인별 **"내가 한 것"** 문단을 작성한다.
4. 개인 AWS 프리 티어 계정을 만들고 **Budgets 알람을 $1로** 설정한다.
5. 6개월 뒤 저장소를 다시 열어 README를 한 번 고쳐 쓴다.

---

## 부록 A. 리소스 정리 확인 체크리스트 & 스크립트

### A-1. 정리 확인 스크립트 `cleanup_check.sh`

```bash
#!/usr/bin/env bash
# 사용법: bash cleanup_check.sh <팀번호>
# 전 항목이 0건이어야 정리 완료. 출력 전체를 캡처해 제출한다.

TEAM="${1:-<팀번호>}"
REGION="ap-northeast-2"
FAIL=0

echo "==================================================="
echo " 리소스 정리 확인  |  팀: $TEAM  |  리전: $REGION"
echo " 실행 시각: $(date '+%Y-%m-%d %H:%M:%S')"
echo "==================================================="

check() {
  local label="$1"; local count="$2"
  if [ "$count" -eq 0 ] 2>/dev/null; then
    printf " [OK ] %-38s 0건\n" "$label"
  else
    printf " [!! ] %-38s %s건  <-- 정리 필요\n" "$label" "$count"
    FAIL=1
  fi
}

n=$(aws ec2 describe-instances --region $REGION \
  --filters "Name=tag:Team,Values=$TEAM" "Name=instance-state-name,Values=running,stopped,stopping" \
  --query "length(Reservations[].Instances[])" --output text)
check "EC2 인스턴스(running/stopped)" "$n"

n=$(aws ec2 describe-volumes --region $REGION \
  --filters "Name=status,Values=available" \
  --query "length(Volumes)" --output text)
check "미사용 EBS 볼륨" "$n"

n=$(aws ec2 describe-addresses --region $REGION \
  --query "length(Addresses)" --output text)
check "탄력적 IP" "$n"

n=$(aws ec2 describe-snapshots --region $REGION --owner-ids self \
  --query "length(Snapshots)" --output text)
check "EBS 스냅샷" "$n"

n=$(aws sagemaker list-endpoints --region $REGION \
  --query "length(Endpoints)" --output text)
check "SageMaker 엔드포인트" "$n"

n=$(aws sagemaker list-apps --region $REGION \
  --query "length(Apps[?Status!='Deleted'])" --output text 2>/dev/null || echo 0)
check "SageMaker 앱(스페이스)" "$n"

n=$(aws ecr describe-repositories --region $REGION \
  --query "length(repositories)" --output text 2>/dev/null || echo 0)
check "ECR 리포지토리" "$n"

n=$(aws s3 ls | wc -l)
check "S3 버킷" "$n"

n=$(aws cloudwatch describe-alarms --region $REGION \
  --query "length(MetricAlarms)" --output text)
check "CloudWatch 알람" "$n"

n=$(aws cloudwatch list-dashboards --region $REGION \
  --query "length(DashboardEntries)" --output text)
check "CloudWatch 대시보드" "$n"

n=$(aws logs describe-log-groups --region $REGION \
  --query "length(logGroups)" --output text)
check "CloudWatch 로그 그룹" "$n"

n=$(aws sns list-topics --region $REGION \
  --query "length(Topics)" --output text)
check "SNS 주제" "$n"

echo "==================================================="
if [ "$FAIL" -eq 0 ]; then
  echo " 결과: 정리 완료 (전 항목 0건)"
else
  echo " 결과: 미정리 항목 있음 -- 위 [!!] 항목을 삭제한 뒤 다시 실행하세요"
fi
echo "==================================================="
```

```bash
chmod +x cleanup_check.sh
bash cleanup_check.sh <팀번호>
```

### A-2. 항목별 삭제 명령 요약

| 순서 | 대상 | 확인 명령 | 삭제 명령 |
|---|---|---|---|
| 1 | SageMaker 엔드포인트 | `aws sagemaker list-endpoints --region ap-northeast-2` | `aws sagemaker delete-endpoint --endpoint-name <명> --region ap-northeast-2` |
| 2 | SageMaker 앱 | `aws sagemaker list-apps --region ap-northeast-2` | `aws sagemaker delete-app --domain-id <ID> --user-profile-name <명> --app-type JupyterLab --app-name <명> --region ap-northeast-2` |
| 3 | EC2 인스턴스 | `aws ec2 describe-instances --region ap-northeast-2` | `aws ec2 terminate-instances --instance-ids <ID> --region ap-northeast-2` |
| 4 | EBS 볼륨 | `aws ec2 describe-volumes --filters Name=status,Values=available --region ap-northeast-2` | `aws ec2 delete-volume --volume-id <ID> --region ap-northeast-2` |
| 5 | 스냅샷 | `aws ec2 describe-snapshots --owner-ids self --region ap-northeast-2` | `aws ec2 delete-snapshot --snapshot-id <ID> --region ap-northeast-2` |
| 6 | 탄력적 IP | `aws ec2 describe-addresses --region ap-northeast-2` | `aws ec2 release-address --allocation-id <ID> --region ap-northeast-2` |
| 7 | ECR | `aws ecr describe-repositories --region ap-northeast-2` | `aws ecr delete-repository --repository-name <명> --force --region ap-northeast-2` |
| 8 | S3 | `aws s3 ls` | `aws s3 rm s3://<버킷>/ --recursive` → `aws s3 rb s3://<버킷>` |
| 9 | 알람 | `aws cloudwatch describe-alarms --region ap-northeast-2` | `aws cloudwatch delete-alarms --alarm-names <명> --region ap-northeast-2` |
| 10 | 대시보드 | `aws cloudwatch list-dashboards --region ap-northeast-2` | `aws cloudwatch delete-dashboards --dashboard-names <명> --region ap-northeast-2` |
| 11 | 로그 그룹 | `aws logs describe-log-groups --region ap-northeast-2` | `aws logs delete-log-group --log-group-name <명> --region ap-northeast-2` |
| 12 | SNS | `aws sns list-topics --region ap-northeast-2` | `aws sns delete-topic --topic-arn <ARN> --region ap-northeast-2` |

### A-3. 최종 비용 확인

```bash
# 팀 태그별
aws ce get-cost-and-usage --region us-east-1 \
  --time-period Start=<학기시작일>,End=<오늘> \
  --granularity MONTHLY --metrics "UnblendedCost" \
  --group-by Type=TAG,Key=Team --output json

# 서비스별
aws ce get-cost-and-usage --region us-east-1 \
  --time-period Start=<학기시작일>,End=<오늘> \
  --granularity MONTHLY --metrics "UnblendedCost" \
  --group-by Type=DIMENSION,Key=SERVICE --output table

# 예산 소진 현황
aws budgets describe-budgets --region us-east-1 --account-id <계정ID> --output table
```

### A-4. 강사용 최종 스윕 (수업 종료 후)

```bash
# 계정 전체에서 살아 있는 리소스 확인 (팀 태그 무관)
for svc in "ec2 describe-instances" "sagemaker list-endpoints"; do
  echo "=== $svc ==="; aws $svc --region ap-northeast-2 --output table
done

aws ec2 describe-volumes --region ap-northeast-2 \
  --filters "Name=status,Values=available" \
  --query "Volumes[].[VolumeId,Size,Tags[?Key=='Team']|[0].Value]" --output table

aws ec2 describe-addresses --region ap-northeast-2 --output table
```
- 발견 시 해당 팀에 개별 통보 후 삭제. **리소스 미정리 회당 -2점** 적용.

---

## 부록 B. 포트폴리오 README 템플릿 (개인 배포용)

````markdown
# <프로젝트 이름>

> <누가 무엇을 할 수 있게 되는지 한 문장>

![Python](https://img.shields.io/badge/Python-3.11-blue)
![FastAPI](https://img.shields.io/badge/FastAPI-009688)
![Docker](https://img.shields.io/badge/Docker-2496ED)
![AWS](https://img.shields.io/badge/AWS-EC2%20|%20S3%20|%20ECR%20|%20CloudWatch-FF9900)
![MLflow](https://img.shields.io/badge/MLflow-0194E2)
![GitHub Actions](https://img.shields.io/badge/CI%2FCD-GitHub%20Actions-2088FF)

<!-- 여기에 동작 스크린샷 또는 GIF 한 장. 이게 30초 안에 보이는 유일한 것 -->
`docs/demo.gif` 경로에 팀이 직접 촬영한 동작 화면을 추가합니다.

> **서비스 운영 상태**: 교육 과정 종료에 따라 클라우드 리소스를 정리하여 현재 라이브 서비스는 종료되었습니다.
> 동작 화면은 [시연 영상](<영상 링크>)에서 확인하실 수 있습니다.

## 내가 한 것
- 담당: <역할>
- 구현: <구체적으로 무엇을 만들었는지 2~3개>
- 기여도: 전체 커밋의 약 <N>% (`git log --author` 기준)

## 1. 문제 정의
## 2. 데이터
## 3. 시스템 구성
## 4. 실행 방법
## 5. API 명세
## 6. 실험 결과와 모델 선택 이유
## 7. 배포 자동화
## 8. 모니터링·운영
## 9. 한계와 다음 단계
## 10. 팀 구성
````

> 3~10장의 상세 양식은 14주차 부록 C(README 필수 항목)를 그대로 사용합니다.

---

## 부록 C. 용어 정리

| 용어 | 뜻 | 한 줄 설명 |
|---|---|---|
| Terminate / Stop | 종료 / 중지 | **중지는 정리가 아니다.** EBS 요금이 계속 나간다. 종료해야 과금이 멈춘다 |
| EBS | Elastic Block Store | EC2에 붙는 디스크. 인스턴스를 꺼도 볼륨이 남아 있으면 요금이 나간다 |
| 탄력적 IP | Elastic IP | 고정 공인 IP. **연결되지 않은 상태에서 요금이 발생**한다 |
| 스냅샷 | snapshot | 볼륨 백업본. 지우지 않으면 계속 저장 요금이 나간다 |
| Cost Explorer | 비용 탐색기 | 태그·서비스별로 쓴 돈을 보는 도구. API는 `us-east-1` 고정 |
| UnblendedCost | 미혼합 비용 | 실제 청구되는 비용 지표 |
| 루브릭 | rubric | 채점 기준표. 무엇이 상·중·하인지 미리 공개된 표 |
| 상호평가 | peer evaluation | 학생 간 평가. 익명 집계로 발표 점수의 30% 반영 |
| SLI | Service Level Indicator | 서비스 수준을 재는 실제 숫자. 13주차 에러율이 여기 해당 |
| 오케스트레이션 | orchestration | 여러 단계·작업의 실행 순서와 상태를 자동 지휘하는 것. 다음 학습 경로의 Airflow가 이 영역 |
| IaC | Infrastructure as Code, 코드형 인프라 | 오늘 손으로 지운 리소스를 코드로 만들고 코드로 지우는 방식(Terraform 등) |
| 포트폴리오 | portfolio | 남에게 보여줄 수 있는 결과물 묶음. 우리 수업에서는 **README 한 장이 실체** |

---

## 부록. 최종 정리 화면 경로

- Cost Explorer: <https://console.aws.amazon.com/costmanagement/home#/cost-explorer>
- EC2: <https://console.aws.amazon.com/ec2/>
- ECR: <https://console.aws.amazon.com/ecr/repositories>
- S3: <https://console.aws.amazon.com/s3/>
- SageMaker AI: <https://console.aws.amazon.com/sagemaker/>
- 정리 기준: **백업 확인 → 비싼 리소스부터 삭제 → 목록이 비었는지 다시 확인 → 비용 화면 저장**
- SageMaker 정리 공식 문서: <https://docs.aws.amazon.com/sagemaker/latest/dg/realtime-endpoints-delete-resources.html>

---

## 실제 AWS 콘솔 화면 실습 가이드

### 01. 삭제 대상 사전 점검

![삭제 대상 사전 점검](../assets/aws-console/week15/01-cleanup-preflight-exact-targets.png)

- 콘솔 경로: **CloudShell → ysu-mlops 접두사별 리소스 목록 조회**
- 확인할 것: SageMaker, IAM, CloudWatch 삭제 대상을 정확한 이름으로 기록한다.
- [AWS 콘솔 열기](https://ap-northeast-2.console.aws.amazon.com/cloudshell/home?region=ap-northeast-2)

### 02. 핵심 자원 ID 최종 확인

![핵심 자원 ID 최종 확인](../assets/aws-console/week15/02-cleanup-core-targets-confirmed.png)

- 콘솔 경로: **CloudShell → EC2·S3·ECR 상세 조회**
- 확인할 것: 인스턴스·VPC·서브넷·보안 그룹 ID와 버킷·저장소 이름을 다시 확인한다.
- [AWS 콘솔 열기](https://ap-northeast-2.console.aws.amazon.com/cloudshell/home?region=ap-northeast-2)

### 03. SageMaker 모델·구성·MLflow 삭제

![SageMaker 모델·구성·MLflow 삭제](../assets/aws-console/week15/03-sagemaker-metadata-deleted.png)

- 콘솔 경로: **CloudShell → sagemaker delete-endpoint-config·delete-model·delete-mlflow-tracking-server**
- 확인할 것: 모델과 구성은 삭제됐지만 계보 Action은 연관 관계 때문에 막힌 것을 확인한다.
- [AWS 콘솔 열기](https://ap-northeast-2.console.aws.amazon.com/cloudshell/home?region=ap-northeast-2)

### 04. SageMaker Action 연관 관계 오류

![SageMaker Action 연관 관계 오류](../assets/aws-console/week15/04-sagemaker-action-delete-association-blocked.png)

- 콘솔 경로: **CloudShell → sagemaker list-associations**
- 확인할 것: Action이 다른 Action·Image Artifact와 연결되어 바로 삭제되지 않는 이유를 읽는다.
- [AWS 콘솔 열기](https://ap-northeast-2.console.aws.amazon.com/cloudshell/home?region=ap-northeast-2)

### 05. 연관 관계 해제 후 Action 삭제

![연관 관계 해제 후 Action 삭제](../assets/aws-console/week15/05-sagemaker-associations-removed-actions-deleted.png)

- 콘솔 경로: **CloudShell → delete-association → delete-action**
- 확인할 것: 연관 관계를 먼저 지운 뒤 세 개의 ysu-mlops Action이 삭제되는지 확인한다.
- [AWS 콘솔 열기](https://ap-northeast-2.console.aws.amazon.com/cloudshell/home?region=ap-northeast-2)

### 06. CloudWatch 수업 자원 삭제

![CloudWatch 수업 자원 삭제](../assets/aws-console/week15/06-cloudwatch-lab-assets-deleted.png)

- 콘솔 경로: **CloudShell → delete-dashboards·delete-alarms·delete-log-group**
- 확인할 것: 대시보드·경보·SageMaker 로그 그룹의 잔여 목록이 비었는지 확인한다.
- [AWS 콘솔 열기](https://ap-northeast-2.console.aws.amazon.com/cloudshell/home?region=ap-northeast-2)

### 07. EC2 인스턴스 종료 요청

![EC2 인스턴스 종료 요청](../assets/aws-console/week15/07-ec2-termination-requested.png)

- 콘솔 경로: **EC2 → Instances → Instance state → Terminate instance**
- 확인할 것: stopped를 최종 정리로 보지 않고 terminated로 바꾼다.
- [AWS 콘솔 열기](https://ap-northeast-2.console.aws.amazon.com/ec2/home?region=ap-northeast-2#Instances:instanceId=i-09e3f0c913e337111)

### 08. EC2 종료와 EBS 삭제 확인

![EC2 종료와 EBS 삭제 확인](../assets/aws-console/week15/08-ec2-terminated-ebs-deleted.png)

- 콘솔 경로: **CloudShell → ec2 wait instance-terminated·describe-volumes**
- 확인할 것: 인스턴스 terminated와 루트 볼륨 NotFound를 함께 확인한다.
- [AWS 콘솔 열기](https://ap-northeast-2.console.aws.amazon.com/cloudshell/home?region=ap-northeast-2)

### 09. ECR 저장소 강제 삭제

![ECR 저장소 강제 삭제](../assets/aws-console/week15/09-ecr-repository-force-deleted.png)

- 콘솔 경로: **CloudShell → ecr delete-repository --force**
- 확인할 것: 1.0·1.1·1.2·릴리스 태그가 든 저장소 이름과 URI를 확인한 뒤 삭제한다.
- [AWS 콘솔 열기](https://ap-northeast-2.console.aws.amazon.com/cloudshell/home?region=ap-northeast-2)

### 10. ECR 저장소 0건

![ECR 저장소 0건](../assets/aws-console/week15/10-ecr-repositories-zero-console.png)

- 콘솔 경로: **Amazon ECR → Private registry → Repositories**
- 확인할 것: 리포지토리 없음 문구를 콘솔에서 확인한다.
- [AWS 콘솔 열기](https://ap-northeast-2.console.aws.amazon.com/ecr/private-registry/repositories?region=ap-northeast-2)

### 11. S3 버킷 첫 삭제 실패

![S3 버킷 첫 삭제 실패](../assets/aws-console/week15/11-s3-bucket-emptied-deleted.png)

- 콘솔 경로: **CloudShell → s3 rm --recursive → s3api delete-bucket**
- 확인할 것: 현재 객체만 지워도 이전 버전이 남아 BucketNotEmpty가 발생하는지 확인한다.
- [AWS 콘솔 열기](https://ap-northeast-2.console.aws.amazon.com/cloudshell/home?region=ap-northeast-2)

### 12. 버전 객체와 삭제 마커 제거

![버전 객체와 삭제 마커 제거](../assets/aws-console/week15/12-s3-versioned-objects-deleted-success.png)

- 콘솔 경로: **CloudShell → list-object-versions → delete-objects → delete-bucket**
- 확인할 것: VersionId 네 개를 제거한 뒤 BUCKET DELETE SUCCESS가 표시되는지 확인한다.
- [AWS 콘솔 열기](https://ap-northeast-2.console.aws.amazon.com/cloudshell/home?region=ap-northeast-2)

### 13. VPC 의존성 확인

![VPC 의존성 확인](../assets/aws-console/week15/13-vpc-dependency-delete-order.png)

- 콘솔 경로: **CloudShell → 인터넷 게이트웨이·라우팅·ENI·서브넷·보안 그룹 조회**
- 확인할 것: 기본 라우팅 테이블과 사용자 라우팅 테이블을 구분해 삭제 순서를 정한다.
- [AWS 콘솔 열기](https://ap-northeast-2.console.aws.amazon.com/cloudshell/home?region=ap-northeast-2)

### 14. VPC와 의존 자원 삭제

![VPC와 의존 자원 삭제](../assets/aws-console/week15/14-vpc-and-dependencies-deleted.png)

- 콘솔 경로: **CloudShell → 서브넷 → 사용자 라우팅 → 보안 그룹 → IGW → VPC 삭제**
- 확인할 것: 의존성이 작은 자원부터 지운 뒤 VPC 잔여 목록이 비는지 확인한다.
- [AWS 콘솔 열기](https://ap-northeast-2.console.aws.amazon.com/cloudshell/home?region=ap-northeast-2)

### 15. 스케줄·인스턴스 프로파일·IAM 역할 삭제

![스케줄·인스턴스 프로파일·IAM 역할 삭제](../assets/aws-console/week15/15-schedules-instance-profile-iam-roles-deleted.png)

- 콘솔 경로: **CloudShell → scheduler·iam 역할 정리**
- 확인할 것: 관리형 정책 분리, 인라인 정책 삭제, 인스턴스 프로파일 삭제 뒤 역할을 삭제한다.
- [AWS 콘솔 열기](https://ap-northeast-2.console.aws.amazon.com/cloudshell/home?region=ap-northeast-2)

### 16. 핵심 비용 자원 최종 0건

![핵심 비용 자원 최종 0건](../assets/aws-console/week15/16-final-zero-core-cost-resources.png)

- 콘솔 경로: **CloudShell → EC2·EBS·EIP·ECR·S3 최종 조회**
- 확인할 것: 각 항목 ZERO와 CORE ACTIVE COUNT 0을 확인한다.
- [AWS 콘솔 열기](https://ap-northeast-2.console.aws.amazon.com/cloudshell/home?region=ap-northeast-2)

### 17. ML·모니터링 자원 최종 0건

![ML·모니터링 자원 최종 0건](../assets/aws-console/week15/17-final-zero-ml-monitoring-resources.png)

- 콘솔 경로: **CloudShell → SageMaker·CloudWatch·Logs 최종 조회**
- 확인할 것: 엔드포인트·모델·구성·MLflow·Action·대시보드·경보·로그가 모두 ZERO인지 확인한다.
- [AWS 콘솔 열기](https://ap-northeast-2.console.aws.amazon.com/cloudshell/home?region=ap-northeast-2)

### 18. IAM·네트워크 자원 최종 0건

![IAM·네트워크 자원 최종 0건](../assets/aws-console/week15/18-final-zero-iam-network-resources.png)

- 콘솔 경로: **CloudShell → IAM·VPC·Subnet·Security Group·Scheduler 최종 조회**
- 확인할 것: 모든 항목 ZERO와 IAM AND NETWORK COUNT 0을 확인한다.
- [AWS 콘솔 열기](https://ap-northeast-2.console.aws.amazon.com/cloudshell/home?region=ap-northeast-2)

### 19. SageMaker 엔드포인트 0건

![SageMaker 엔드포인트 0건](../assets/aws-console/week15/19-sagemaker-endpoints-zero-console.png)

- 콘솔 경로: **SageMaker AI → Deployments and inference → Endpoints**
- 확인할 것: 현재 리소스가 없습니다 문구를 최종 발표 증거로 저장한다.
- [AWS 콘솔 열기](https://ap-northeast-2.console.aws.amazon.com/sagemaker/home?region=ap-northeast-2#/endpoints)

### 20. 최종 비용 리뷰

![최종 비용 리뷰](../assets/aws-console/week15/20-final-cost-explorer-review.png)

- 콘솔 경로: **Billing and Cost Management → Cost Explorer**
- 확인할 것: 기간·서비스 그룹화·비혼합 비용과 비용 데이터의 반영 지연 안내를 확인한다.
- [AWS 콘솔 열기](https://console.aws.amazon.com/costmanagement/home#/cost-explorer)

### 실제 정리 결과

| 범주 | 결과 |
|---|---|
| EC2·EBS·EIP | 활성 EC2 0, 루트 EBS 삭제, EIP 0 |
| S3·ECR | 수업 버킷 0, 수업 ECR 저장소 0 |
| SageMaker·MLflow | 엔드포인트·모델·구성·Tracking Server·계보 Action 0 |
| CloudWatch | 수업 대시보드·경보·로그 그룹 0 |
| VPC | 수업 VPC·서브넷·사용자 보안 그룹·IGW·사용자 라우팅 테이블 0 |
| IAM·Scheduler | 수업 역할·인스턴스 프로파일·스케줄 0 |

> 삭제는 되돌릴 수 없다. 실행 전에는 접두사와 ID를 두 번 확인하고, 명령 연결에는 `;`보다 `&&`를 사용해 앞 단계 실패 뒤에 성공 메시지가 출력되는 일을 막는다. 이번 실습에서도 첫 S3 삭제가 실패했는데 뒤의 `echo`가 실행되어 성공처럼 보일 수 있었다. 최종 평가는 삭제 명령의 출력이 아니라 **삭제 후 다시 조회한 0건 화면**으로 한다.
