# 클라우드 MLOps 기존 자료 분석 및 교차검증 보고서

작성 기준일: 2026-08-24  
검증 범위: 기존 강의계획·교육계획서·14개 주차 교안·14개 PPTX·NCS 학습모듈·AWS 공식 문서

## 1. 결론

기존 자료는 “모델을 학습하는 수업”보다 “모델을 서비스로 운영하는 수업”이라는 방향이 뚜렷하고, 실습 산출물과 프로젝트 마일스톤도 잘 연결되어 있다. 다만 다음 네 가지는 반드시 바로잡아야 한다.

1. 강의계획만 15주로 바뀌고 교육계획서·교안·PPTX는 14주로 남아 있다.
2. 사람 계정에 장기 IAM 사용자와 액세스 키를 주는 방식은 AWS의 현재 권고와 맞지 않는다.
3. GitHub Actions에 AWS 키를 저장하는 예시가 남아 있어 OIDC 기반 임시 자격 증명 방식으로 통일해야 한다.
4. NCS 매핑은 내용상 대체로 타당하지만, 일부 능력단위의 버전이 학교 보유 PDF보다 새 버전으로 바뀌었을 가능성이 있으므로 공식 제출 전 버전 확인이 필요하다.

## 2. 기존 자료의 강점

- 매주 눈에 보이는 산출물이 있다: S3 경로, EC2 화면, MLflow 실험표, ECR 이미지, API 주소, 대시보드와 알람 등.
- 3주차 문제 정의부터 15주차 발표까지 하나의 팀 프로젝트가 이어진다.
- 실습 A는 공통 예제, 실습 B는 팀 프로젝트로 나뉘어 있어 초보 학생의 진입 장벽을 낮춘다.
- 중지·삭제·비용 확인을 수업 절차에 포함해 클라우드 운영 습관을 함께 평가한다.
- NCS `인공지능 모델 관리`의 구성요소 관리·배포 관리·성능 관리가 5~13주차 흐름과 잘 맞는다.

## 3. 구조와 내용에서 발견한 문제

| 구분 | 발견 내용 | 개정 방향 |
|---|---|---|
| 주차 체계 | 강의계획은 15주, 나머지는 14주 | 9주차 통합·보강 신설, 기존 9~14주를 10~15주로 이동 |
| 선수학습 표기 | 학년과 기능사 수준이 혼재 | “Python·ML 기초 이수”로 통일 |
| 문체 | “관통한다”, “강제로 이어간다”처럼 강한 표현과 번역체 혼재 | 학생에게 직접 설명하는 짧은 문장으로 수정 |
| AWS 인증 | 학생별 IAM 사용자·장기 키 중심 | IAM Identity Center 또는 역할 기반 임시 자격 증명 우선 |
| EC2 접속 | SSH·키페어·포트 개방 비중이 큼 | Session Manager/EC2 Instance Connect를 우선 대안으로 제시 |
| CI/CD | AWS 키 저장 예시가 남음 | GitHub OIDC → IAM 역할 → 임시 자격 증명으로 변경 |
| 비용 | “약 $1”, “약 $25”처럼 고정값 사용 | 가격표·리전·모델·사용시간에 따라 달라짐을 명시하고 수업 당일 계산 |
| Bedrock | 모델 액세스 신청을 항상 별도 단계로 설명 | 상용 리전의 기본 자동 액세스와 예외 모델 절차를 구분 |
| 시각 자료 | 슬라이드 대부분이 글·코드·표 중심 | 실제 화면, 콘솔 이동 경로, 아키텍처 그림, 확인 지점을 주차별로 추가 |

## 4. NCS 교차검증 결과

### 4-1. 검증 방법

- 기존 `NCS매핑표.md`의 채택 코드 50개를 중복 제거해 확인했다.
- `/NCS` 폴더의 학습모듈 PDF에서 학습목표·능력단위요소·평가 항목을 다시 확인했다.
- 실습 중 학생 행동으로 관찰 가능한지, 제출물로 증명 가능한지를 기준으로 매핑을 다시 판단했다.

### 4-2. 타당성이 높은 핵심 매핑

| 주차 | NCS 요소 | 관찰 가능한 증거 |
|---|---|---|
| 3주 | 모델 목표·요구사항 정의, 데이터 수집·정제·변환 | 문제정의서, 데이터 카드, 전처리 실행 로그 |
| 4주 | 후보 모델 도출, 기본 설계, 학습, 평가 기준 | 베이스라인 성능표, 저장된 Pipeline |
| 5주 | 인자 조율, 결과 검증, 최적 모델 선정, 구성요소 관리 | MLflow 비교 화면, 모델 선정 근거서 |
| 6~8주 | 소프트웨어 환경 구현, 추론·인터페이스 구현, 단위·통합 테스트 | Docker 이미지, `/predict`, Streamlit 화면, 테스트 결과 |
| 9주 | 배포·성능·운영 절차 점검 | 전 구간 연결 체크리스트, 기술 부채 정리표 |
| 12주 | 모델 배포 관리, 운영 절차, 장애 해결 | GitHub Actions 실행 기록, 롤백 기록 |
| 13주 | 자원·인터페이스·성능 모니터링, 결과 관리, 품질지표 | CloudWatch 대시보드·알람, 운영 체크리스트 |
| 14~15주 | 품질 점검·개선·매뉴얼, 성과 기준·평가 | 재현 테스트 결과, README, 최종 루브릭 |

### 4-3. 과대 매핑을 피해야 하는 항목

- 1주차의 플랫폼 동향·비용 계획은 소개 수준이므로 `부분 정합`으로 유지한다.
- 10주차 SageMaker AI는 플랫폼 자체의 학습·모델링 기능을 “구현”한다기보다 관리형 기능을 “사용·비교”하는 수준이다. 평가 문구도 사용·비교로 제한한다.
- 11주차 Bedrock은 외부 인터페이스 구현에는 정합하지만, 서비스 활용 기획 전체를 충족한다고 보기는 어렵다.
- SLA, VOC, 과금 정책, 조직 수준 장애관리 등은 학생 팀 프로젝트만으로 수행준거 전체를 증명하기 어렵다. 개념 소개와 부분 정합으로 남긴다.

### 4-4. 버전 확인 주의

학교 보유 자료에는 `20010701` 계열이 `18v1`로 되어 있으나, NCS의 2024년 인공지능SW개발 맥락화 자료에는 `인공지능 플랫폼 인프라 구현`이 `2001070104_23v2`로 제시된다. 따라서 교내 공식 강의계획서에 제출할 때에는 교무처가 사용하는 NCS 기준 연도와 버전을 먼저 확인해야 한다. 이번 개정본은 기존 보유 학습모듈과의 추적성을 위해 기존 코드를 유지하고, 버전 확인 주석을 추가했다.

- [NCS 훈련기준 검색](https://www.ncs.go.kr/education/hph02/hph0204/selectTrainBaseNcsSearch.do)
- [NCS 및 학습모듈 검색 안내](https://www.ncs.go.kr/web/tutorial/tutorial01_01_05.jsp)
- 로컬 근거: `../NCS/LM2001070308_인공지능+모델+관리.pdf`
- 로컬 근거: `../NCS/LM2001070404_인공지능서비스+운영모니터링.pdf`
- 로컬 근거: `../NCS/LM2001070405_인공지능서비스+운영품질관리.pdf`

## 5. AWS 공식 문서 교차검증 결과

### 5-1. 계정과 권한

사람이 AWS를 사용할 때에는 장기 자격 증명보다 IAM Identity Center나 역할을 통한 임시 자격 증명을 권장한다. 학생 실습 계정도 가능하면 IAM Identity Center의 사용자·그룹·권한 세트로 운영한다. 단일 계정 사정으로 IAM 사용자를 써야 한다면 MFA, 최소 권한, 키 만료·회수 절차를 별도로 둔다.

- [AWS 계정 액세스 계획](https://docs.aws.amazon.com/IAM/latest/UserGuide/gs-identities.html)
- [IAM 자격 증명 비교](https://docs.aws.amazon.com/IAM/latest/UserGuide/introduction_identity-management.html)
- [IAM Identity Center 사용자·그룹](https://docs.aws.amazon.com/singlesignon/latest/userguide/users-groups-provisioning.html)

### 5-2. S3·EC2·ECR

- S3 버킷은 이름과 리전을 만든 뒤 바꿀 수 없으므로 서울 리전을 먼저 확인한다.
- 새 S3 버킷은 퍼블릭 액세스 차단을 유지한다.
- EC2는 실행 중이면 유휴 상태여도 비용이 발생한다.
- ECR은 콘솔의 `Private repositories → Create repository` 경로와 CLI 흐름을 함께 제시한다.

- [S3 버킷 만들기](https://docs.aws.amazon.com/AmazonS3/latest/userguide/create-bucket-overview.html)
- [EC2 인스턴스 시작 마법사](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-launch-instance-wizard.html)
- [ECR 프라이빗 저장소 만들기](https://docs.aws.amazon.com/AmazonECR/latest/userguide/repository-create.html)

### 5-3. SageMaker AI·Bedrock·CloudWatch

- SageMaker AI는 엔드포인트 삭제뿐 아니라 엔드포인트 구성과 모델도 정리 대상으로 확인한다.
- Bedrock 상용 리전에서는 올바른 Marketplace 권한이 있으면 모델 액세스가 기본 활성화된다. 다만 일부 제공자는 최초 사용 때 용도 정보가 필요할 수 있다.
- CloudWatch 대시보드는 전역 객체이지만, 대시보드 안의 메트릭과 로그는 리전·네임스페이스를 정확히 선택해야 한다.

- [SageMaker AI 엔드포인트와 리소스 삭제](https://docs.aws.amazon.com/sagemaker/latest/dg/realtime-endpoints-delete-resources.html)
- [Bedrock 모델 액세스](https://docs.aws.amazon.com/bedrock/latest/userguide/model-access.html)
- [Bedrock Converse API](https://docs.aws.amazon.com/bedrock/latest/APIReference/API_runtime_Converse.html)
- [CloudWatch 대시보드 만들기](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/create_dashboard.html)

### 5-4. GitHub Actions 인증

AWS 액세스 키를 GitHub Secrets에 장기 보관하는 방식은 기본 예시에서 제외한다. GitHub OIDC 공급자를 IAM에 등록하고, 저장소·브랜치 조건이 포함된 역할을 만든 뒤 `configure-aws-credentials`가 임시 자격 증명을 받게 한다.

- [AWS IAM에서 GitHub OIDC 역할 만들기](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_create_for-idp_oidc.html)
- [AWS 공식 GitHub Action: configure-aws-credentials](https://github.com/aws-actions/configure-aws-credentials)

## 6. 화면 자료 사용 원칙

AWS 콘솔은 자주 바뀌므로 스크린샷만 따라 하게 하지 않는다. 모든 실습 화면에는 다음 네 가지를 함께 둔다.

1. 콘솔 직접 링크
2. 메뉴 이동 경로
3. 화면에서 확인할 항목
4. 같은 작업을 확인하는 CLI 명령

공식 문서 화면 캡처는 `assets/aws/`에 보관했다. 과거 UI가 포함된 공식 화면은 “버튼 모양이 아니라 메뉴 이름과 경로를 확인하는 참고 화면”이라고 표시한다.

## 7. 개정 결과물 구성

- `클라우드MLOps_15주차_강의계획_NCS_AWS검증.md`
- `클라우드MLOps_교육계획서_v4.md`
- `NCS매핑표_v3_15주차.md`
- `교안/` 15개 주차 Markdown
- `PPT/` 15개 주차 PowerPoint

