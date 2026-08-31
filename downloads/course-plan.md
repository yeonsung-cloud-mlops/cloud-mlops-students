# 클라우드 MLOps 15주차 강의계획

선수지식: Python 기초, pandas·scikit-learn 입문, Git 기초  
운영: 주 1회 230분, 실습 중심  
클라우드: AWS 서울 리전(`ap-northeast-2`)  

## 교과목 목표

이 과목을 마치면 학생은 노트북에서 만든 모델을 컨테이너와 API로 묶어 AWS에 배포하고, 자동 배포와 모니터링이 연결된 작은 MLOps 흐름을 설명하고 시연할 수 있다.

## 수업 운영 원칙

- 개념을 먼저 길게 설명하지 않는다. 오늘 만들 결과물을 보여준 뒤 필요한 개념을 짧게 배운다.
- 강사 예제를 따라 한 뒤 같은 구조를 팀 프로젝트에 적용한다.
- 매주 URL·파일·실행 로그·스크린샷 중 하나 이상을 제출한다.
- AWS 콘솔 화면은 `직접 링크 → 메뉴 경로 → 확인할 값 → CLI 확인` 순서로 안내한다.
- 마지막 5분은 비용이 발생하는 리소스를 확인하고 중지·삭제한다.

## 15주 학습 흐름

| 주차 | 주제 | 학생이 할 수 있어야 하는 일 | 실습 산출물 | 핵심 NCS 연계 |
|---|---|---|---|---|
| 1 | 가격 예측 체험과 AWS 첫걸음 | 사용자 화면·API·운영 화면의 관계를 말하고, 리전·권한·비용을 확인해 S3를 사용할 수 있다 | 가격 예측 체험 기록, S3 버킷과 폴더 구조 | 플랫폼 동향·비용, 서비스 기술 환경 |
| 2 | Linux와 EC2 개발환경 | EC2에 안전하게 접속하고 Python 환경을 만들 수 있다 | 접속 가능한 개발 서버 | 하드웨어·소프트웨어 환경 구현 |
| 3 | 데이터 파이프라인과 문제 정의 | 원본·가공 데이터를 분리하고 ML 문제를 문장으로 정의할 수 있다 | 전처리 스크립트, 데이터 카드 | 목표·요구사항, 데이터 수집·정제·변환 |
| 4 | 베이스라인 모델 | 데이터 누수 없이 기준 모델을 학습·평가할 수 있다 | Pipeline, 성능표, 모델 파일 | 후보 모델, 기본 설계, 학습, 평가 기준 |
| 5 | MLflow 실험 관리 | 실험을 기록·비교하고 모델 선택 근거를 남길 수 있다 | 실험 5건, 등록 모델 | 인자 조율, 결과 검증, 모델 선정·관리 |
| 6 | Docker와 ECR | 실행 환경을 이미지로 만들고 ECR에 저장할 수 있다 | Dockerfile, ECR 이미지 | 소프트웨어 환경, 구성요소 관리 |
| 7 | FastAPI 모델 서빙 | `/health`, `/predict` API를 만들고 테스트할 수 있다 | 예측 API와 테스트 결과 | 추론·외부 인터페이스, 단위 테스트 |
| 8 | EC2 배포와 데모 화면 | 컨테이너를 배포하고 Streamlit 화면과 연결할 수 있다 | 공개 데모, 중간 시연 | 배포 관리, HMI, 통합 테스트 |
| 9 | 프로젝트 통합·보강 | 앞 8주의 끊긴 구간을 찾아 직접 연결하고 복구할 수 있다 | 전 구간 체크리스트, 이슈표 | 배포·성능 관리, 운영 절차·자원 점검 |
| 10 | Amazon SageMaker AI | 관리형 학습·엔드포인트를 사용하고 직접 구축과 비교할 수 있다 | 학습 작업, 엔드포인트, 삭제 증거 | 학습·모델링 기능 부분 정합, 비용 계획 |
| 11 | Amazon Bedrock | 전통 ML 결과를 설명하는 보조 LLM 기능을 붙일 수 있다 | Converse API 호출, 보조 엔드포인트 | 외부 인터페이스, 활용 방안 부분 정합 |
| 12 | GitHub Actions 배포 자동화 | OIDC 역할로 테스트·빌드·배포를 자동화할 수 있다 | 성공한 워크플로, 롤백 기록 | 배포 관리, 운영 절차·장애 해결 |
| 13 | CloudWatch 모니터링과 운영 | 로그·메트릭·알람을 구성하고 이상을 찾을 수 있다 | 대시보드, 알람, 운영표 | 운영모니터링·품질지표, 성능 관리 |
| 14 | 프로젝트 집중 워크숍 | 다른 팀이 README만 보고 재현하도록 보완할 수 있다 | 교차 재현 결과, 발표 자료, 영상 | 품질 점검·개선, 운영매뉴얼 |
| 15 | 최종 발표와 정리 | 서비스 성과를 발표하고 비용·리소스를 정리할 수 있다 | 발표, 최종 저장소, 정리 확인 | 성과 평가, 설계 검증, 품질 개선 |

## 표준 230분 수업안

| 구간 | 시간 | 내용 |
|---|---:|---|
| 도입 | 10분 | 복습과 오늘 결과물 미리보기 |
| 핵심 개념 | 40분 | 실습에 필요한 개념·용어·안전 주의 |
| 휴식 | 10분 |  |
| 실습 A | 55분 | 강사 공통 예제 따라 하기 |
| 휴식 | 10분 |  |
| 실습 B | 85분 | 개인·팀 프로젝트 적용 |
| 확인 | 15분 | 체크포인트 제출, NCS 수행 증거 확인 |
| 정리 | 5분 | AWS 리소스와 비용 확인 |

## 평가

| 평가 항목 | 배점 | 평가 증거 |
|---|---:|---|
| 출석·수업 참여 | 10 | 출석, 실습 참여 |
| 주차별 체크포인트 | 25 | 실행 로그, URL, 화면, 파일 |
| M1 문제정의서 | 10 | 문제·데이터·지표·성공 기준 |
| M2 중간 시연 | 15 | API와 화면의 통합 동작 |
| 최종 산출물 | 25 | 재현성, 배포, 운영, 문서화 |
| 최종 발표·상호평가 | 15 | 설명, 근거, 시연, 질의응답 |

## AWS 운영 기준

- 사람 계정은 IAM Identity Center 또는 역할 기반 임시 자격 증명을 우선한다.
- S3 퍼블릭 액세스 차단은 기본값을 유지한다.
- 보안그룹의 `0.0.0.0/0` 개방은 공개 서비스 포트를 제외하고 사용하지 않는다.
- GitHub Actions는 OIDC 역할을 사용하고 장기 액세스 키를 저장하지 않는다.
- 가격은 고정 숫자로 외우지 않고 수업 당일 AWS 가격표와 Cost Explorer에서 확인한다.
- SageMaker 엔드포인트는 삭제 결과까지 제출해야 실습 완료로 인정한다.

## 공식 문서 시작점

- [AWS IAM 계정 액세스 계획](https://docs.aws.amazon.com/IAM/latest/UserGuide/gs-identities.html)
- [S3 버킷 만들기](https://docs.aws.amazon.com/AmazonS3/latest/userguide/create-bucket-overview.html)
- [EC2 시작 마법사](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-launch-instance-wizard.html)
- [ECR 저장소 만들기](https://docs.aws.amazon.com/AmazonECR/latest/userguide/repository-create.html)
- [SageMaker AI 리소스 삭제](https://docs.aws.amazon.com/sagemaker/latest/dg/realtime-endpoints-delete-resources.html)
- [Bedrock 모델 액세스](https://docs.aws.amazon.com/bedrock/latest/userguide/model-access.html)
- [CloudWatch 대시보드](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/create_dashboard.html)
- [GitHub OIDC용 IAM 역할](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_create_for-idp_oidc.html)
