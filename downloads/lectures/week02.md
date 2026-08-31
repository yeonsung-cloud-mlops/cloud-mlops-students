# 02주차 — 리눅스 · EC2 개발환경 구축

**클라우드 MLOps** · 연성대학교 · 230분 (10분 조기 종료)
NCS: `2001070104_18v1.1` 인공지능 플랫폼 하드웨어 환경 구현하기 / `2001070104_18v1.2` 인공지능 플랫폼 소프트웨어 환경 구현하기

---

## 0. 회차 요약 (강사용 1페이지)

| 항목 | 내용 |
|---|---|
| 학습목표 | EC2 인스턴스를 직접 생성해 원격 접속하고, 리눅스 기본 명령으로 서버를 조작하며, Python 가상환경과 JupyterLab을 설치해 `boto3`로 1주차 S3 데이터를 DataFrame으로 읽을 수 있다. |
| 오늘의 결과물 | ① 실행 중인 EC2 인스턴스 `mlops-<학번>` ② SSH 또는 Instance Connect 접속 화면 ③ JupyterLab에서 S3 CSV를 `pd.read_csv`로 읽어 `df.head()`를 출력한 화면 |
| 사전 준비 | **학교망에서 22번 포트(SSH) 개방 여부 사전 테스트** (막혀 있으면 실습 A를 Instance Connect 경로로 통째로 전환) / EC2 Instance Connect 및 Session Manager 동작 확인(SSM Agent 포함 AMI, IAM 인스턴스 프로파일 `MLOpsStudentSSMRole` 준비) / 학생 정책의 `ec2:RunInstances` 타입 화이트리스트(`t3.micro/small/medium`) 재확인 / 보안그룹 템플릿 `mlops-sg-<학번>` 생성 절차 리허설 / `week-02-done` 완성본 태그 및 `setup.sh` 배포 준비 / 강의실 8888 포트 아웃바운드 접속 테스트 |
| 학생 준비물 | 노트북, **1주차 버킷 이름 메모**(`mlops-2026-<학번>`), AWS 콘솔 로그인 정보 + MFA 폰, 노트북에 터미널 사용 가능 여부 확인(Windows는 PowerShell 또는 Windows Terminal) |
| 예상 사고 지점 | ① **학교망 22번 포트 차단** — SSH가 `Connection timed out`으로 무한 대기. 실습이 통째로 멈춘다 ② **키페어 권한 오류** — Windows에서 `.pem` 권한 문제, macOS/Linux에서 `Permissions 0644 are too open` ③ **JupyterLab 원격 접속 실패** — 보안그룹 8888 미개방 또는 `--ip=0.0.0.0` 누락 ④ **키페어 파일 분실** — 다운로드 창에서 무심코 닫아 재발급 불가 |

### 시간표

| 시간 | 구성 | 분 |
|---|---|---|
| 00:00–00:10 | 지난주 복습 + **오늘 만들 결과물 데모** | 10 |
| 00:10–00:55 | 이론 강의 — 서버 / EC2 구성요소 / 보안그룹 / 리눅스 명령 15개 | 45 |
| 00:55–01:05 | 휴식 | 10 |
| 01:05–01:55 | **실습 A** — EC2 생성 · 키페어 · 접속 · 리눅스 명령 · `apt` 설치 | 50 |
| 01:55–02:05 | 휴식 | 10 |
| 02:05–03:30 | **실습 B** — 가상환경 · JupyterLab 원격 접속 · `boto3`로 S3 읽기 | 85 |
| 03:30–03:45 | 체크포인트 제출 + 다음주 예고 | 15 |
| 03:45–03:50 | **리소스 정리 타임** — ⚠ **인스턴스 Stop 확인** | 5 |
| **합계** | | **230** |

---

## 1. 도입 (10분)

### 지난주 복습 퀴즈 (구두 3문항)

1. 우리가 한 학기 내내 쓰는 리전은 어디이고, 코드로 뭐라고 쓰나요? — (서울, `ap-northeast-2`)
2. S3 버킷 안에 만든 세 폴더 이름과 각각의 용도는? — (`raw/` 원본 절대 수정 금지 / `processed/` 전처리 결과 / `models/` 학습된 모델)
3. `aws s3 cp a.txt s3://내버킷/raw/a.txt` 는 업로드일까요, 다운로드일까요? — (업로드. 왼쪽이 내 컴퓨터, 오른쪽이 S3)

> 추가 확인: 지난주 MFA 미완료자 명단을 호명해 완료 여부를 확인한다. 미완료자는 실습 A 시작 전에 조교가 처리한다.

### 오늘 만들 것 데모

강사 화면으로 3분 안에 완주해 보인다. **설명하지 말고 결과만 보여준다.**

1. AWS 콘솔 → EC2 → 인스턴스 목록에서 강사 인스턴스 `mlops-demo`가 `실행 중`인 것을 보여준다.
2. 노트북 터미널에서 접속한다.
```bash
ssh -i ~/.ssh/mlops-demo.pem ubuntu@<퍼블릭IP>
```
3. 접속된 검은 화면에서 명령 세 개를 친다.
```bash
whoami
df -h
python3 --version
```
4. 브라우저 새 탭에 `http://<퍼블릭IP>:8888` 을 열어 JupyterLab을 띄운다.
5. 노트북 셀에 아래를 실행해 표가 나오는 것을 보여준다.
```python
import pandas as pd
df = pd.read_csv("s3://mlops-2026-demo/raw/sample_bike.csv")
df.head()
```
6. 멘트: "오늘 끝나면 이 화면이 여러분 것으로 하나씩 생깁니다. **여러분 노트북이 아니라 서울에 있는 컴퓨터에서 돌아가는 화면**입니다. 그리고 마지막에 저 서버를 반드시 끄고 갑니다. 안 끄면 감점입니다."

---

## 2. 이론 (45분)

### 2-1. 서버란 무엇인가 (10분)

**강의 스크립트**
> 질문 하나 드리겠습니다. 지금 여러분 노트북과 서버는 뭐가 다를까요?
> (학생 답 유도 — 성능? 크기? 값?)
> 성능도 아니고 크기도 아닙니다. 답은 하나입니다. **서버는 안 꺼집니다.** 그리고 **남이 접속합니다.**
> 여러분 노트북은 가방에 넣으면 잠들죠. 그러면 여러분이 만든 서비스도 같이 잠듭니다. 새벽 3시에 누가 여러분 API를 부르면? 아무 응답이 없습니다.
> 그래서 서비스를 하려면 **24시간 깨어 있고, 고정된 주소를 가지고, 남이 들어올 문이 열려 있는 컴퓨터**가 필요합니다. 그게 서버입니다.
>
> 옛날에는 이걸 회사가 직접 샀습니다. 서버실이라는 방을 만들고, 에어컨 틀고, 정전 대비 배터리 놓고, 인터넷 회선 끌어오고. 컴퓨터 한 대 쓰려고 방을 하나 지은 겁니다.
> 클라우드가 바꾼 게 이 부분입니다. **아마존이 지어놓은 방에서 컴퓨터 한 대를 시간 단위로 빌리는 것.** 그게 EC2입니다.
> EC2는 Elastic Compute Cloud의 약자인데, C가 세 개라서 E-C-2입니다. 이름은 외울 필요 없고, **"빌리는 컴퓨터 한 대"**로 기억하세요. 한 대 한 대를 **인스턴스(Instance)** 라고 부릅니다.
>
> 오늘 여러분이 빌릴 컴퓨터는 `t3.medium`입니다. CPU 2개, 메모리 4GB짜리입니다. 여러분 노트북보다 못합니다. 그런데 왜 쓰냐고요? **안 꺼지고, 주소가 있고, 남이 접속할 수 있기 때문입니다.** 성능 때문이 아닙니다.
> 그리고 하나 더 — 앞으로 팀 프로젝트를 할 때 **팀원 전부가 같은 컴퓨터에서 작업**할 수 있습니다. "저는 윈도우인데요", "저는 맥인데요" 하는 문제가 사라집니다. 이게 실무에서 클라우드를 쓰는 큰 이유 중 하나입니다.

**판서/슬라이드 요점**
- 서버 = ① 안 꺼진다 ② 고정 주소가 있다 ③ 남이 접속한다. 성능 문제가 아님
- EC2(Elastic Compute Cloud) = 시간 단위로 빌리는 컴퓨터. 한 대 = **인스턴스**
- 우리 사양: `t3.medium` (vCPU 2, RAM 4GB), Ubuntu 22.04 또는 24.04 LTS
- 팀 전원이 같은 환경에서 작업 가능 → "제 컴퓨터에선 되는데요" 문제 감소
- ⚠ 켜져 있는 동안 계속 과금. 하루 방치 ≈ $1

**학생 질문 예상 & 답변**
- Q: 제 노트북에 그냥 파이썬 깔면 안 되나요? → A: 지금은 됩니다. 7주차부터 안 됩니다. 남이 인터넷으로 여러분 모델을 호출해야 하는데, 여러분 노트북에는 인터넷 주소가 없습니다.
- Q: `t3` 의 `t`는 무슨 뜻인가요? → A: 인스턴스 계열 이름입니다. `t`는 범용 + 버스트(순간 가속) 계열입니다. `m`은 균형형, `c`는 CPU 강화형, `g`/`p`는 GPU입니다. 우리 계정은 `t3`만 허용됩니다.

### 2-2. EC2 구성요소 — AMI · 인스턴스 타입 · EBS · 키페어 · 보안그룹 (15분)

**강의 스크립트**
> EC2 인스턴스를 하나 만들 때 정해야 할 게 다섯 가지입니다. 이걸 **컴퓨터 조립**에 비유해서 가겠습니다.
>
> **첫째, AMI(Amazon Machine Image).** 우리말로 "머신 이미지"인데, 쉽게 말하면 **운영체제가 미리 깔린 복사본**입니다.
> 새 컴퓨터를 샀을 때 윈도우를 설치하죠? AMI는 그 설치가 이미 끝난 상태의 스냅샷입니다. 이걸 복사해서 새 컴퓨터를 만드니까 3분이면 부팅이 끝납니다.
> 우리는 **Ubuntu 22.04 LTS 또는 24.04 LTS**를 씁니다. 우분투는 리눅스의 한 종류이고, LTS는 Long Term Support, 장기 지원 버전이라는 뜻입니다. 오래 안정적으로 쓰라고 만든 버전이라 실무에서 제일 많이 씁니다.
> 여기서 중요한 것 — 우분투 AMI로 만든 서버는 로그인 계정 이름이 **`ubuntu`** 입니다. Amazon Linux면 `ec2-user`입니다. 이걸 헷갈리면 접속이 안 됩니다. 매년 여기서 몇 명이 막힙니다.
>
> **둘째, 인스턴스 타입.** CPU와 메모리 사양입니다. `t3.medium`은 vCPU 2개, 메모리 4GB. 우리는 이걸 씁니다. 그 이상은 계정에서 막혀 있습니다.
>
> **셋째, EBS(Elastic Block Store).** 우리말로 "블록 저장소", 쉽게 말해 **하드디스크**입니다.
> 여기서 1주차에 배운 S3와 헷갈리기 쉬운데, 차이가 분명합니다. **EBS는 서버에 붙은 내장 하드**입니다. 그 서버만 쓸 수 있습니다. **S3는 인터넷 창고**입니다. 아무 서버에서나 접근할 수 있습니다.
> 우리는 EBS를 30GB 씁니다. 그리고 ⚠ **중요** — EBS는 **서버를 꺼도 요금이 나갑니다.** 호텔로 치면 방은 비웠는데 짐 보관함은 계속 빌리고 있는 겁니다. 30GB면 한 달에 3달러쯤 됩니다. 그래서 학기 말에는 인스턴스를 아예 **종료(Terminate)** 해서 EBS까지 지웁니다. 오늘 배울 **중지(Stop)** 와는 다릅니다. 이건 뒤에서 다시 짚겠습니다.
>
> **넷째, 키페어(Key Pair).** 서버에 들어가는 **열쇠**입니다.
> 서버는 비밀번호로 로그인하지 않습니다. 왜냐면 비밀번호는 짧고, 인터넷에 열린 서버에는 하루에 수천 번씩 비밀번호 추측 공격이 들어오기 때문입니다. 실제로 그렇습니다.
> 대신 열쇠 파일을 씁니다. 열쇠가 두 개 한 쌍으로 만들어지는데, **공개키는 서버에 남고, 개인키(`.pem` 파일)는 여러분이 받습니다.**
> 여기서 제일 중요한 말씀 — **개인키 파일은 만들 때 딱 한 번만 다운로드됩니다.** 잃어버리면 AWS도 다시 못 줍니다. 그 서버는 버리고 새로 만들어야 합니다. 그러니까 다운로드 창이 뜨면 **무조건 저장하고, 저장한 위치를 메모하세요.**
>
> **다섯째, 보안그룹(Security Group).** 이건 워낙 중요해서 다음 소주제로 따로 하겠습니다.

**판서/슬라이드 요점**

| 구성요소 | 비유 | 우리 설정 |
|---|---|---|
| AMI | OS가 깔린 복사본 | Ubuntu 22.04 / 24.04 LTS (로그인 계정 = `ubuntu`) |
| 인스턴스 타입 | CPU·메모리 사양 | `t3.medium` (vCPU 2 / RAM 4GB) |
| EBS | 서버에 붙은 하드디스크 | 30GB gp3. ⚠ **꺼도 과금** |
| 키페어 | 서버 열쇠 | `.pem` 개인키. **재발급 불가** |
| 보안그룹 | 방화벽 = 건물 출입문 | 22, 8888, (8000/8501은 7~8주차) |

- **S3 vs EBS**: S3 = 인터넷 창고(아무나 접근) / EBS = 그 서버 전용 내장 하드
- **Stop(중지)** = 전원 끄기. 서버 요금 정지, **EBS 요금은 계속** / **Terminate(종료)** = 폐기. 전부 삭제, 복구 불가

**학생 질문 예상 & 답변**
- Q: `.pem` 파일을 카카오톡으로 나한테 보내놔도 되나요? → A: 편의상 그렇게들 하는데, 원칙적으로 개인키는 남한테 안 보이는 곳에 둡니다. 팀 프로젝트에서는 **키를 공유하지 말고 각자 자기 인스턴스를 만드는 것**이 원칙입니다.
- Q: 키를 잃어버리면 서버 안의 파일도 다 날아가나요? → A: 볼륨을 떼서 다른 인스턴스에 붙이면 살릴 수 있습니다. 하지만 난이도가 높습니다. 우리 수업에서는 **그냥 새로 만드는 게 빠릅니다.** 그래서 서버 안에 중요한 걸 남기지 말고 **S3와 GitHub에 올려두는 습관**을 들이는 겁니다.
- Q: Stop과 Terminate 중 매주 뭘 하나요? → A: 매주는 **Stop**입니다. Terminate 하면 그 안에 설치한 것들이 전부 날아가서 다음 주에 처음부터 다시 해야 합니다. Terminate는 15주차에 합니다.

### 2-3. 보안그룹 = 방화벽 (10분)

**강의 스크립트**
> 보안그룹은 이 과목에서 **가장 자주 사고가 나는 지점**입니다. 그래서 따로 떼서 하겠습니다.
> 보안그룹(Security Group)은 **방화벽**입니다. 우리말로 방화벽인데, 하는 일은 **건물 출입문 관리**입니다.
> 서버라는 건물에 문이 65535개 있다고 생각하세요. 이 문마다 번호가 붙어 있고, 이걸 **포트(Port)** 라고 합니다. 보안그룹은 **어느 번호 문을 열어둘지, 그리고 누구한테 열어둘지**를 정하는 규칙표입니다.
>
> 기본값은 **전부 잠금**입니다. 아무 문도 안 열려 있습니다. 그래서 여러분이 문을 열어주지 않으면 접속 자체가 안 됩니다.
> 우리가 쓸 문 번호를 정리하겠습니다.
> - **22번** — SSH. 원격으로 서버에 접속해서 명령어를 치는 문입니다. 오늘 이걸 씁니다.
> - **8888번** — JupyterLab. 브라우저로 주피터를 여는 문입니다. 오늘 실습 B에서 씁니다.
> - **8000번** — FastAPI. 7주차에 우리 모델 API가 여기로 나갑니다.
> - **8501번** — Streamlit. 8주차 데모 화면이 여기로 나갑니다.
>
> 그리고 "누구한테 열어줄지"가 있습니다. 이걸 **소스(Source)** 라고 합니다.
> `0.0.0.0/0` 이라고 쓰면 **전 세계 아무나**라는 뜻입니다. 편하지만 위험합니다.
> `내 IP` 를 고르면 **지금 이 강의실에서만** 접속됩니다. 안전하지만, 집에 가서 하려면 다시 바꿔야 합니다.
> 우리 수업에서는 **22번은 `내 IP`, 8888번도 `내 IP`** 로 시작하겠습니다. 집에서 하실 분은 그때 규칙을 하나 추가하시면 됩니다. 방법은 실습에서 알려드리겠습니다.
>
> 마지막으로 경고 하나. 나중에 "접속이 안 돼요" 할 때 **십중팔구는 보안그룹입니다.** 그리고 두 번째로 흔한 게 **학교 인터넷이 22번 문을 막아둔 경우**입니다. 이건 여러분 서버 문제가 아니라 **학교에서 나가는 길이 막힌 것**입니다. 그런 경우에 쓸 우회로를 오늘 같이 배웁니다.

**판서/슬라이드 요점**
- 보안그룹 = 방화벽 = 서버 출입문 관리. **기본값은 전부 잠금**
- 포트 번호: **22**(SSH) / **8888**(JupyterLab) / 8000(FastAPI, 7주차) / 8501(Streamlit, 8주차)
- 소스: `0.0.0.0/0` = 전 세계 아무나(위험) / `내 IP` = 지금 내 위치만(권장)
- 접속 안 될 때 1순위 의심 = 보안그룹, 2순위 = **학교망이 22번을 막음**
- ⚠ 22번을 `0.0.0.0/0`으로 열면 몇 분 만에 자동 침입 시도가 들어온다

**학생 질문 예상 & 답변**
- Q: `내 IP`로 했는데 집에서 안 돼요. → A: 정상입니다. 집 IP는 강의실 IP와 다릅니다. 콘솔에서 보안그룹 인바운드 규칙을 편집해 `내 IP`를 다시 눌러 갱신하면 됩니다.
- Q: 그냥 전부 열어두면 편하지 않나요? → A: 편합니다. 그리고 위험합니다. 실습 목적이니 8888은 상황에 따라 허용하겠지만, **22번은 절대 전 세계 공개로 두지 않습니다.**

### 2-4. 리눅스 최소 생존 명령 15개 (10분)

**강의 스크립트**
> 이제 검은 화면 얘기를 하겠습니다. 무섭죠? 그런데 오늘 배울 명령어는 딱 **15개**입니다. 15개만 알면 이 학기 다 됩니다.
> 마음가짐부터 하나 바꿔드리겠습니다. **검은 화면은 폴더 탐색기입니다.** 윈도우 탐색기에서 폴더 열고, 파일 복사하고, 이름 바꾸는 걸 마우스 대신 글자로 하는 것뿐입니다. 새로운 개념이 아닙니다.
>
> 딱 세 개만 몸에 붙이면 됩니다. **`pwd` 나 지금 어디 있지, `ls` 여기 뭐 있지, `cd` 저기로 가자.** 이 세 개를 습관처럼 치세요. 길 잃을 일이 없습니다.
> 그리고 마법의 키가 하나 있습니다. **Tab 키.** 파일 이름을 앞 두세 글자만 치고 Tab을 누르면 나머지가 자동 완성됩니다. 오타로 안 되는 경우의 90%가 이걸로 사라집니다. 오늘 Tab 키를 손가락에 붙이고 가세요.

| # | 명령 | 뜻 | 예시 |
|---|---|---|---|
| 1 | `pwd` | 지금 내 위치 (print working directory) | `pwd` |
| 2 | `ls` | 여기 뭐 있나 (list) | `ls -al` |
| 3 | `cd` | 이동 (change directory) | `cd ~/work` , `cd ..` (한 단계 위) |
| 4 | `mkdir` | 폴더 만들기 (make directory) | `mkdir -p ~/work/data` |
| 5 | `cp` | 복사 (copy) | `cp a.csv b.csv` |
| 6 | `mv` | 이동/이름변경 (move) | `mv a.csv data/` |
| 7 | `rm` | 삭제 (remove) ⚠ 휴지통 없음 | `rm a.csv` , `rm -r olddir` |
| 8 | `cat` | 파일 전부 출력 | `cat hello.txt` |
| 9 | `less` | 파일을 페이지 단위로 보기 (`q`로 나감) | `less big.csv` |
| 10 | `tail` | 파일 끝부분 보기 | `tail -f app.log` (실시간 감시) |
| 11 | `chmod` | 권한 바꾸기 (change mode) | `chmod 400 key.pem` |
| 12 | `ps` | 지금 도는 프로그램 목록 | `ps aux \| grep jupyter` |
| 13 | `kill` | 프로그램 종료 | `kill -9 <PID>` |
| 14 | `df` | 디스크 남은 용량 (disk free) | `df -h` |
| 15 | `top` | CPU·메모리 실시간 사용량 (`q`로 나감) | `top` |

**판서/슬라이드 요점**
- 3대 기본기: `pwd`(어디) → `ls`(뭐 있나) → `cd`(가자)
- **Tab 키 자동완성**, **↑ 방향키로 이전 명령 재사용** — 이 둘이 속도의 90%
- `rm`은 휴지통이 없다. **`rm -rf` 는 오늘 쓰지 않는다**
- `sudo` = "관리자 권한으로". 설치할 때만 쓴다
- 화면이 멈춘 것 같으면 **`Ctrl + C`** (실행 중단), 프로그램에서 나오려면 `q` 또는 `exit`

**학생 질문 예상 & 답변**
- Q: 명령어를 다 외워야 하나요? → A: 아니요. 치트시트(부록 A) 보고 치면 됩니다. 5주쯤 지나면 손이 외웁니다.
- Q: `sudo`는 왜 붙이나요? → A: "관리자 권한으로 실행"이라는 뜻입니다. 프로그램을 설치하는 건 시스템 전체를 건드리는 일이라 관리자 권한이 필요합니다. 내 폴더 안에서 하는 일에는 안 붙입니다.
- Q: 실수로 뭘 지웠는데 되살릴 수 있나요? → A: 없습니다. 그래서 **중요한 건 S3와 GitHub에 올려둡니다.** 서버 안은 언제든 날아갈 수 있는 곳이라고 생각하세요.

---

## 3. 실습 A — EC2 생성 · 접속 · 리눅스 명령 (50분) · 공통 예제 따라하기

**목표** `t3.medium` Ubuntu 인스턴스를 만들어 원격 접속하고, 리눅스 기본 명령을 실행한 뒤 `apt`로 패키지를 설치한다.

**사전 배포 파일**
- `02_EC2생성_체크리스트.pdf` (설정값 표 1장)
- `02_setup.sh` (실습 B용 환경 구축 스크립트 — 실습 A 마지막에 내려받기만 한다)

> ⚠ **강사 사전 판단 (수업 시작 전에 결론을 내려둘 것)**
> 강의실 네트워크에서 22번 포트가 나가는지 미리 테스트한다. 학생 노트북 1대로 아래를 실행해 본다.
> ```bash
> nc -vz -w 5 <강사 테스트 인스턴스 퍼블릭IP> 22
> ```
> - **성공(succeeded)** → 아래 **경로 1(SSH)** 로 진행. 경로 2는 "집에서 안 될 때 쓰는 법"으로 5분 소개만.
> - **실패(timed out)** → 처음부터 **경로 2(EC2 Instance Connect)** 로 반 전체를 몰고 간다. SSH는 시연만 한다.
> 반반으로 갈리면 **경로 2를 기본으로** 잡는 편이 수업 운영이 훨씬 안정적이다.

### 수행 순서

**① 보안그룹 먼저 만들기 (8분)**

인스턴스보다 방화벽을 먼저 만든다. 나중에 만들면 접속이 안 되는 상태로 헤매게 된다.

1. 콘솔 오른쪽 위 리전이 **서울(ap-northeast-2)** 인지 먼저 확인한다.
2. 검색창에 `EC2` → EC2 대시보드 → 왼쪽 메뉴 **네트워크 및 보안 → 보안 그룹** → **보안 그룹 생성**.
3. 다음과 같이 입력한다.

| 항목 | 값 |
|---|---|
| 보안 그룹 이름 | `mlops-sg-<학번>` |
| 설명 | `MLOps class SG for <학번>` |
| VPC | 기본값(default VPC) 그대로 |

4. **인바운드 규칙**에 두 개를 추가한다.

| 유형 | 포트 | 소스 | 설명 |
|---|---|---|---|
| SSH | 22 | **내 IP** | `ssh from classroom` |
| 사용자 지정 TCP | 8888 | **내 IP** | `jupyterlab` |

5. **아웃바운드 규칙**은 기본값(모든 트래픽 허용) 그대로 둔다.
6. 태그에 `Course=MLOps`, `Owner=<학번>` 추가 → **보안 그룹 생성**.
- 확인 포인트: 보안 그룹 목록에 `mlops-sg-<학번>` 이 보이고, 인바운드 규칙이 2개다.

**② 키페어 만들기 (5분)**

1. 왼쪽 메뉴 **네트워크 및 보안 → 키 페어** → **키 페어 생성**.
2. 다음과 같이 입력한다.

| 항목 | 값 |
|---|---|
| 이름 | `mlops-key-<학번>` |
| 키 페어 유형 | RSA |
| 프라이빗 키 파일 형식 | **`.pem`** (PuTTY를 쓸 사람만 `.ppk`) |

3. **키 페어 생성**을 누르면 `mlops-key-<학번>.pem` 이 즉시 다운로드된다.
4. 다운로드 폴더에서 안전한 위치로 옮기고, **경로를 메모**한다.
   - Windows 권장 위치: `C:\Users\<사용자명>\.ssh\`
   - macOS/Linux 권장 위치: `~/.ssh/`

⚠ **이 파일은 두 번 다시 다운로드할 수 없다.** 지금 옮겨두지 않으면 오늘 만든 서버에 영원히 못 들어간다.

macOS/Linux 사용자는 권한을 반드시 조정한다.
```bash
mkdir -p ~/.ssh
mv ~/Downloads/mlops-key-<학번>.pem ~/.ssh/
chmod 400 ~/.ssh/mlops-key-<학번>.pem
ls -l ~/.ssh/mlops-key-<학번>.pem
```
- 확인 포인트: `-r--------` 로 보이면 정상.

**③ EC2 인스턴스 생성 (12분)**

1. EC2 대시보드 → **인스턴스 시작(Launch instances)**.
2. 아래 표대로 정확히 설정한다.

| 항목 | 값 |
|---|---|
| 이름(Name) | `mlops-<학번>` |
| AMI | **Ubuntu Server 24.04 LTS (또는 22.04 LTS)**, 아키텍처 **64비트(x86)** |
| 인스턴스 유형 | **`t3.medium`** |
| 키 페어 | `mlops-key-<학번>` |
| 네트워크 설정 | **기존 보안 그룹 선택** → `mlops-sg-<학번>` |
| 퍼블릭 IP 자동 할당 | **활성화** |
| 스토리지 | **30 GiB gp3** |

3. **고급 세부 정보**를 펼쳐 다음을 설정한다. (경로 2 대비용 — 반드시 한다)
   - **IAM 인스턴스 프로파일**: `MLOpsStudentSSMRole` (강사가 사전 생성. S3 읽기 + SSM 접속 권한 포함)
4. **태그 추가**에서 3개를 넣는다.

| 키 | 값 |
|---|---|
| `Course` | `MLOps` |
| `Team` | `T00` (3주차에 수정) |
| `Owner` | `<학번>` |

5. 오른쪽 요약 패널에서 **인스턴스 수 = 1** 인지 확인하고 **인스턴스 시작**.
6. 인스턴스 목록에서 상태가 `대기 중` → `실행 중`, 상태 검사가 `2/2 통과`가 될 때까지 기다린다(2~3분).
7. 인스턴스를 클릭해 **퍼블릭 IPv4 주소**를 복사하고 메모한다. (예: `13.125.xxx.xxx`)
- 확인 포인트: 상태 `실행 중`, 상태 검사 `2/2 통과`, 퍼블릭 IP가 표시됨.

**강의 스크립트 (기다리는 2분 동안)**
> 지금 서울 어딘가의 데이터센터에서 여러분 컴퓨터 한 대가 켜지고 있습니다. 30초에서 2분 걸립니다.
> 그리고 **이 순간부터 돈이 나가기 시작합니다.** `t3.medium`이 시간당 약 0.052달러니까, 4시간 수업이면 약 0.2달러입니다. 커피값도 안 됩니다.
> 문제는 안 껐을 때입니다. 켜두고 한 달 지나면 37달러입니다. 여러분 한 학기 예산이 25달러인데요. 그래서 오늘 마지막 5분이 중요한 겁니다.

**④-경로1. SSH로 접속하기 (10분) — 학교망에서 22번이 열려 있을 때**

> SSH(Secure Shell, 보안 원격 접속 프로토콜) = 암호화된 통로로 남의 컴퓨터에 들어가 명령어를 치는 방법.

**Windows (PowerShell 또는 Windows Terminal)**
```powershell
cd $env:USERPROFILE\.ssh
ssh -i .\mlops-key-<학번>.pem ubuntu@<퍼블릭IP>
```

**macOS / Linux (터미널)**
```bash
ssh -i ~/.ssh/mlops-key-<학번>.pem ubuntu@<퍼블릭IP>
```

처음 접속하면 아래가 뜬다. `yes`를 입력하고 Enter.
```text
The authenticity of host '13.125.xxx.xxx' can't be established.
ED25519 key fingerprint is SHA256:...
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
```

- 확인 포인트: 프롬프트가 `ubuntu@ip-172-31-x-x:~$` 로 바뀌면 접속 성공. **이 화면을 캡처**한다. (체크포인트 제출물 1번)

**④-경로2. EC2 Instance Connect로 접속하기 (10분) — ⚠ 22번이 막혔을 때의 대안 경로**

> 학교망·사내망에서는 방화벽이 22번 포트로 나가는 트래픽을 차단하는 경우가 흔하다.
> 이때는 **키 파일도, 터미널도 필요 없다.** 브라우저 안에서 바로 서버에 접속한다.

**방법 A — EC2 Instance Connect (브라우저 접속, 가장 간단)**

1. EC2 → 인스턴스 → `mlops-<학번>` 선택 → 오른쪽 위 **연결(Connect)** 버튼.
2. 상단 탭에서 **EC2 인스턴스 연결(EC2 Instance Connect)** 선택.
3. **연결 유형**: `EC2 Instance Connect 엔드포인트를 사용하여 연결` 또는 `퍼블릭 IP 사용` 중 접속되는 쪽을 고른다.
4. **사용자 이름**이 `ubuntu` 인지 확인한다. (기본값이 `root`나 `ec2-user`로 되어 있으면 반드시 `ubuntu`로 고친다)
5. **연결** 클릭 → 새 브라우저 탭에 검은 터미널이 열린다.

사전 조건 — 보안그룹 인바운드에 아래 규칙을 **하나 더** 추가한다. (강사가 화면으로 같이 한다)

| 유형 | 포트 | 소스 | 설명 |
|---|---|---|---|
| SSH | 22 | `13.209.1.56/29` | `ec2-instance-connect ap-northeast-2` |

> 이 대역은 서울 리전의 EC2 Instance Connect 서비스가 사용하는 IP 대역이다.
> 대역 값은 변경될 수 있으므로 강사가 **개강 전 `AWS ip-ranges.json`에서 `EC2_INSTANCE_CONNECT` / `ap-northeast-2` 항목을 확인해 갱신**한다.
> 확인 명령:
> ```bash
> curl -s https://ip-ranges.amazonaws.com/ip-ranges.json | \
>   python3 -c "import sys,json;d=json.load(sys.stdin);print([p['ip_prefix'] for p in d['prefixes'] if p['service']=='EC2_INSTANCE_CONNECT' and p['region']=='ap-northeast-2'])"
> ```

**방법 B — Session Manager (포트를 아예 안 씀, 가장 확실)**

브라우저 접속조차 막히거나, 퍼블릭 IP 없이 접속해야 할 때 쓴다. **22번 포트를 전혀 사용하지 않는다.**

1. 사전 조건 확인
   - 인스턴스에 IAM 인스턴스 프로파일 `MLOpsStudentSSMRole` 이 붙어 있어야 한다 (③-3에서 설정함).
   - Ubuntu 22.04/24.04 공식 AMI에는 SSM Agent가 기본 포함되어 있다. 없으면 아래로 설치한다.
   ```bash
   sudo snap install amazon-ssm-agent --classic
   sudo snap start amazon-ssm-agent
   ```
2. EC2 → 인스턴스 선택 → **연결(Connect)** → **Session Manager** 탭 → **연결**.
3. 접속되면 계정이 `ssm-user` 이므로, 우리 작업 계정으로 전환한다.
```bash
sudo su - ubuntu
whoami
```
- 확인 포인트: `ubuntu` 가 출력되면 이후 모든 실습을 동일하게 진행할 수 있다.

> **경로 2로 진행할 때의 조정 사항**
> - 실습 B의 JupyterLab 8888 포트도 학교망에서 막힐 수 있다. 이때는 **Session Manager 포트 포워딩**으로 우회한다. (실습 B ④에 절차 있음)
> - 키페어는 그래도 만들어 둔다. 집에서 SSH로 접속할 학생이 필요로 한다.
> - 체크포인트 제출물 1번은 Instance Connect / Session Manager 화면으로 대체 인정한다.

**⑤ 리눅스 명령 15개 연습 (10분)**

접속한 검은 화면에서 순서대로 친다. **Tab 키와 ↑ 방향키를 쓰라고 계속 상기시킨다.**

```bash
# 나는 누구, 여긴 어디
whoami
pwd
ls -al

# 서버 사양 확인 (t3.medium = CPU 2, RAM 4GB 인지 눈으로 본다)
nproc
free -h
df -h

# 폴더 만들고 이동하기
mkdir -p ~/work/data
cd ~/work
pwd
ls -al

# 파일 만들고 보기
echo "hello from <학번>" > note.txt
cat note.txt
echo "second line" >> note.txt
cat note.txt
tail -1 note.txt

# 복사 / 이동 / 삭제
cp note.txt note_backup.txt
ls
mv note_backup.txt data/
ls data/
rm data/note_backup.txt
ls data/

# 지금 도는 프로그램 / 자원 사용량
ps aux | head -5
top    # 확인 후 q 를 눌러 나온다

# 홈으로 돌아가기
cd ~
pwd
```
- 확인 포인트: `nproc` = `2`, `free -h`의 총 메모리가 약 `3.8Gi`, `df -h`의 `/` 크기가 약 `29G` 로 보이면 우리가 주문한 사양이 맞다.

**⑥ `apt`로 패키지 설치 (5분)**

> `apt` = 우분투의 프로그램 설치 도구. 윈도우의 "프로그램 추가/제거"에 해당한다.

```bash
# 설치 가능한 프로그램 목록을 최신화 (설치 전에 항상 먼저)
sudo apt update

# 우리가 쓸 기본 도구 설치 (-y = 물어보면 전부 yes)
sudo apt install -y python3-pip python3-venv unzip curl git

# 설치 확인
python3 --version
pip3 --version
git --version
```
- 확인 포인트: `Python 3.12.x`(24.04) 또는 `Python 3.10.x`(22.04)가 출력되면 성공.

**강의 스크립트**
> `sudo apt update` 는 "설치할 수 있는 프로그램 목록을 최신으로 갱신"하는 명령이고, `sudo apt install` 이 실제 설치입니다. **update를 안 하고 install 하면 "그런 패키지 없다"는 오류가 자주 납니다.** 세트로 외우세요.
> `-y` 는 "설치하시겠습니까? Y/n" 물어볼 때 자동으로 예라고 답하는 옵션입니다. 자동화할 때 필수입니다.

### ⚠ 여기서 막히면

| 증상 | 원인 | 조치 |
|---|---|---|
| `ssh: connect to host ... port 22: Connection timed out` (반응 없이 멈춤) | **학교망이 22번 아웃바운드 차단** 또는 보안그룹에 내 IP 미등록 | ① 보안그룹 인바운드에서 `내 IP` 다시 눌러 갱신 ② 그래도 안 되면 **경로 2(EC2 Instance Connect)** 로 전환 ③ 폰 핫스팟으로 바꿔 재시도하면 원인이 학교망임을 즉시 확인 가능 |
| `Permission denied (publickey)` | 사용자 이름이 틀림 (`root`, `ec2-user`로 시도) | Ubuntu AMI는 반드시 **`ubuntu@`**. 키 파일 경로도 다시 확인 |
| `WARNING: UNPROTECTED PRIVATE KEY FILE! Permissions 0644 are too open` | `.pem` 권한이 열려 있음 (macOS/Linux) | `chmod 400 ~/.ssh/mlops-key-<학번>.pem` |
| Windows에서 `chmod`가 없다고 나옴 | PowerShell에는 `chmod`가 없음 | 파일 우클릭 → 속성 → 보안 → 고급 → 상속 사용 안 함 → 본인 계정만 남기고 나머지 제거. 또는 `.pem`을 `C:\Users\<사용자명>\.ssh\` 로 옮기면 대개 해결 |
| `.pem` 파일을 못 찾겠음 / 다운로드 창을 닫아버림 | 키페어는 **재다운로드 불가** | 인스턴스를 종료(Terminate)하고 **키페어부터 새로 만들어 인스턴스 재생성**. 5분이면 된다. 시간 아깝다고 붙잡지 말 것 |
| 인스턴스 시작 시 `You are not authorized to perform this operation` | `t3.medium` 외 타입을 골랐거나 스토리지를 크게 잡음 | 인스턴스 유형을 `t3.medium`, 스토리지를 30GiB로 되돌린다 |
| 상태 검사가 `1/2`에서 멈춤 | 부팅 중이거나 AMI 문제 | 3분 더 기다린다. 그래도면 인스턴스 중지 후 시작. 반복되면 재생성 |
| Instance Connect에서 `Failed to connect to your instance` | 보안그룹에 Instance Connect 대역 미등록 또는 사용자 이름 오류 | 인바운드에 `13.209.1.56/29` 추가, 사용자 이름을 `ubuntu`로 수정 |
| Session Manager 탭이 회색으로 비활성 | IAM 인스턴스 프로파일 미연결 | 인스턴스 선택 → 작업 → 보안 → **IAM 역할 수정** → `MLOpsStudentSSMRole` 연결 → 2분 대기 후 재시도 |
| `sudo apt update` 에서 `Could not get lock /var/lib/dpkg/lock-frontend` | 부팅 직후 자동 업데이트가 돌고 있음 | 2분 기다렸다가 재실행. 급하면 `sudo killall apt apt-get` 후 재실행 |

### 컷오프 안내

**50분 경과 시 강제 종료.** 최소 조건은 **"인스턴스가 실행 중이고, 어떤 방법으로든 검은 화면에 접속했다"** 이다.
- 접속 방법은 SSH / Instance Connect / Session Manager 중 아무거나 인정한다.
- ⑤ 리눅스 명령과 ⑥ `apt` 설치를 못 끝낸 학생은 실습 B의 `02_setup.sh` 한 방으로 따라잡을 수 있으므로 그대로 넘어간다.
- 접속 자체가 안 된 학생은 조교가 **강사 예비 인스턴스**(`mlops-spare-01~03`)를 임시 배정해 실습 B를 진행시키고, 본인 인스턴스는 버퍼 시간에 해결한다.
- 완성본 태그: `week-02-done`

---

## 4. 실습 B — Python 가상환경 · JupyterLab 원격 접속 · boto3로 S3 읽기 (85분) · 각자 직접 수행

**목표** EC2 안에 격리된 Python 환경을 만들고 JupyterLab을 원격으로 띄운 뒤, `boto3`로 1주차에 올린 S3 데이터를 DataFrame으로 읽어온다.

**과제 지시문** (학생에게 그대로 읽어줄 문장)

> 지금부터 85분 동안 여러분 서버를 **개발 환경**으로 만듭니다.
> 순서는 세 단계입니다. 첫째, 파이썬 방을 하나 따로 만듭니다. 둘째, 그 방에 주피터를 설치해서 **여러분 노트북 브라우저로** 접속합니다. 셋째, 그 주피터에서 **지난주에 S3에 올린 파일을 읽습니다.**
> 마지막 단계가 오늘의 핵심입니다. 여러분 노트북에 파일이 없는데도 표가 뜹니다. 왜냐면 **서울에 있는 서버가 서울에 있는 S3에서 읽어오기 때문**입니다. 이게 클라우드에서 데이터를 다루는 기본 형태이고, 3주차부터 계속 이렇게 합니다.
> 지난주 버킷 이름 안 적어오신 분? 지금 콘솔 S3 화면 열어서 확인하세요.

### 수행 항목

**1. Python 가상환경 만들기 (10분)**

**강의 스크립트**
> 왜 가상환경을 쓰냐고요? 프로젝트마다 필요한 라이브러리 버전이 다르기 때문입니다.
> A 프로젝트는 pandas 1.5가 필요하고 B 프로젝트는 pandas 2.2가 필요한데, 컴퓨터 한 대에 하나만 깔 수 있으면 싸움이 납니다.
> 가상환경은 **프로젝트마다 파이썬 방을 따로 파주는 것**입니다. 방 안에서 뭘 깔든 다른 방에는 영향이 없습니다.
> 그리고 실무적으로 더 중요한 이유 — **6주차에 도커를 배울 때 "이 방에 뭐가 들어있는지" 목록이 그대로 재료가 됩니다.** 지금 습관을 들여두는 겁니다.

```bash
# 작업 폴더로 이동
mkdir -p ~/work
cd ~/work

# 가상환경 생성 (.venv 라는 이름의 방을 만든다)
python3 -m venv .venv

# 방에 들어가기 (activate = 활성화)
source .venv/bin/activate

# 프롬프트 맨 앞에 (.venv) 가 붙었는지 확인
which python
python --version
```
- 확인 포인트: 프롬프트가 `(.venv) ubuntu@ip-172-31-x-x:~/work$` 로 바뀌고, `which python` 이 `/home/ubuntu/work/.venv/bin/python` 을 가리키면 성공.

> ⚠ **접속을 끊었다가 다시 들어오면 가상환경이 풀린다.** 매번 `cd ~/work && source .venv/bin/activate` 를 다시 쳐야 한다. 이걸 몰라서 "아까 설치했는데 없다고 나와요" 하는 학생이 반드시 나온다.

**2. 필요한 라이브러리 설치 (10분)**

```bash
# pip 자체를 최신화
pip install --upgrade pip

# 이번 학기 내내 쓸 기본 세트
pip install jupyterlab pandas numpy scikit-learn boto3 s3fs pyarrow matplotlib

# 설치 확인
pip list | grep -E "jupyterlab|pandas|boto3|s3fs"

# 설치 목록을 파일로 저장 (6주차 도커에서 그대로 쓴다)
pip freeze > requirements.txt
head -20 requirements.txt
```
- 확인 포인트: `jupyterlab`, `pandas`, `boto3`, `s3fs` 네 줄이 모두 보이면 성공. 설치에 2~4분 걸린다.

> 설치가 오래 걸리는 동안 강사는 **다음 단계인 JupyterLab 접속 원리**를 미리 설명한다.
> "서버에서 프로그램을 8888번 문 앞에 세워두고, 여러분 노트북 브라우저가 그 문으로 들어가는 겁니다. 그래서 아까 보안그룹에 8888을 열어둔 겁니다."

**3. JupyterLab 원격 접속 (20분)**

**① 접속 비밀번호 설정**
```bash
# 토큰 대신 비밀번호로 들어가게 설정 (매번 긴 토큰을 복사하지 않아도 됨)
jupyter lab password
```
비밀번호를 두 번 입력한다. (화면에 안 보이는 게 정상)

**② JupyterLab 실행**
```bash
cd ~/work
jupyter lab --ip=0.0.0.0 --port=8888 --no-browser
```

> `--ip=0.0.0.0` 이 **핵심**이다. 기본값은 `127.0.0.1`(자기 자신만)이라 밖에서 접속이 안 된다.
> 이 옵션을 빼먹는 학생이 매년 절반이다. 강사는 이 줄을 칠판에 크게 적어둔다.

**③ 브라우저에서 접속**

노트북 브라우저 주소창에 입력한다.
```text
http://<퍼블릭IP>:8888
```
아까 설정한 비밀번호를 입력하면 JupyterLab 화면이 뜬다.

**④ ⚠ 8888번이 막혔을 때 — 포트 포워딩 대안 경로**

학교망에서 8888로 나가는 것이 막혀 있으면 위 방법이 안 된다. 두 가지 우회로가 있다.

**대안 A — SSH 터널링** (22번이 열려 있는 경우)

노트북에서 **새 터미널 창**을 하나 더 열고 실행한다. (기존 접속 창은 그대로 둔다)
```bash
ssh -i ~/.ssh/mlops-key-<학번>.pem -N -L 8888:localhost:8888 ubuntu@<퍼블릭IP>
```
그리고 브라우저에서 `http://localhost:8888` 로 접속한다.
> `-L 8888:localhost:8888` = "내 노트북 8888번으로 오는 걸 서버 8888번으로 보내라"는 뜻. `-N` = 명령은 안 치고 통로만 연다.

**대안 B — Session Manager 포트 포워딩** (22번도 막힌 경우)

노트북에 AWS CLI와 Session Manager 플러그인이 설치되어 있어야 한다. (강사가 사전 배포)
```bash
aws ssm start-session \
  --target <인스턴스ID> \
  --document-name AWS-StartPortForwardingSession \
  --parameters '{"portNumber":["8888"],"localPortNumber":["8888"]}' \
  --region ap-northeast-2
```
브라우저에서 `http://localhost:8888` 로 접속한다.
> `<인스턴스ID>` 는 EC2 콘솔의 `i-0abc123...` 형식 값. 이 방법은 **8888은 물론 22번도 전혀 쓰지 않으므로** 학교망 차단을 완전히 우회한다.

- 확인 포인트: 브라우저에 JupyterLab 화면이 뜨고 왼쪽에 `~/work` 폴더 목록이 보이면 성공.

**⑤ 백그라운드 실행으로 바꾸기 (선택, 권장)**

지금 상태로는 터미널을 닫으면 JupyterLab이 꺼진다. `nohup`으로 뒤에서 계속 돌게 한다.
```bash
# Ctrl+C 로 현재 실행을 멈춘 뒤
cd ~/work
source .venv/bin/activate
nohup jupyter lab --ip=0.0.0.0 --port=8888 --no-browser > ~/jupyter.log 2>&1 &

# 잘 떴는지 로그로 확인
sleep 3
tail -20 ~/jupyter.log

# 도는지 확인
ps aux | grep jupyter | head -3
```
끄고 싶을 때:
```bash
pkill -f "jupyter lab"
```

**4. boto3로 S3 데이터 읽기 (35분)**

JupyterLab에서 새 노트북(Python 3)을 만들고 파일명을 `w02_s3_test.ipynb` 로 저장한다.

**① 내가 누구인지부터 확인**
```python
import boto3

sts = boto3.client("sts", region_name="ap-northeast-2")
print(sts.get_caller_identity())
```
- 확인 포인트: `Arn` 에 `assumed-role/MLOpsStudentSSMRole` 또는 본인 IAM 사용자 이름이 보인다.

> ⚠ **여기서 코드에 액세스 키를 절대 적지 않는다.** 인스턴스에 붙여둔 IAM 역할이 자동으로 권한을 넘겨준다.
> 8주차에 "코드에 키를 넣지 않는다"를 다시 다루지만, 원칙은 오늘부터 지킨다.

**② 내 버킷 목록과 객체 목록 보기**
```python
s3 = boto3.client("s3", region_name="ap-northeast-2")

BUCKET = "mlops-2026-<학번>"   # 1주차에 만든 본인 버킷 이름으로 바꿀 것

resp = s3.list_objects_v2(Bucket=BUCKET, Prefix="raw/")
for obj in resp.get("Contents", []):
    print(obj["Key"], obj["Size"], "bytes")
```
- 확인 포인트: 1주차에 올린 `raw/sample_bike.csv` 가 목록에 나온다.

**③ 파일을 서버로 내려받아 pandas로 읽기 (방법 1 — 명시적 다운로드)**
```python
import pandas as pd

s3.download_file(BUCKET, "raw/sample_bike.csv", "/home/ubuntu/work/sample_bike.csv")

df = pd.read_csv("/home/ubuntu/work/sample_bike.csv")
print(df.shape)
df.head()
```

**④ S3 경로를 직접 읽기 (방법 2 — `s3fs` 사용, 앞으로 이 방법을 주로 쓴다)**
```python
import pandas as pd

BUCKET = "mlops-2026-<학번>"
df = pd.read_csv(f"s3://{BUCKET}/raw/sample_bike.csv")

print("행 수:", len(df))
print("열 목록:", list(df.columns))
df.head()
```
- 확인 포인트: 표가 출력되면 성공. **이 화면을 캡처**한다. (체크포인트 제출물 2번 — `df.head()` 결과와 `BUCKET` 변수가 함께 보이도록)

**강의 스크립트**
> 지금 무슨 일이 일어났는지 한 번 짚고 갑시다.
> 여러분 노트북에는 이 CSV 파일이 **없습니다.** 그런데 표가 떴습니다. 어떻게 된 걸까요?
> 서울에 있는 여러분 서버가, 서울에 있는 S3에서, 파일을 읽어서, 그 결과 화면만 여러분 브라우저로 보내준 겁니다. 여러분 노트북은 **그냥 모니터 역할**만 하고 있습니다.
> 이게 왜 좋냐면 — 데이터가 10GB여도 여러분 노트북은 아무 부담이 없습니다. 그리고 팀원 세 명이 각자 노트북으로 같은 서버에 붙어서 **같은 데이터를 같은 코드로** 볼 수 있습니다.
> 3주차부터 우리는 계속 이 구조로 작업합니다. 오늘 이 한 줄이 학기 전체의 기본기입니다.

**⑤ 간단한 전처리 결과를 다시 S3에 올려보기 (3주차 예고)**
```python
# 아주 간단한 가공: 행 수를 절반으로 줄여보기
df_small = df.head(100)

out_path = f"s3://{BUCKET}/processed/sample_bike_head100.csv"
df_small.to_csv(out_path, index=False)
print("saved:", out_path)

# 올라갔는지 확인
resp = s3.list_objects_v2(Bucket=BUCKET, Prefix="processed/")
for obj in resp.get("Contents", []):
    print(obj["Key"], obj["Size"], "bytes")
```
- 확인 포인트: `processed/sample_bike_head100.csv` 가 목록에 나오면 **읽기와 쓰기가 모두 되는 상태**다. 3주차 준비 완료.

**5. 정리 스크립트 확인 (5분)**

같은 환경을 다시 만들 때를 대비해 명령을 파일 하나로 정리해 둔다. 강사 배포본 `02_setup.sh` 와 본인 것을 비교한다.
```bash
cat > ~/work/setup.sh << 'EOF'
#!/bin/bash
set -e
sudo apt update
sudo apt install -y python3-pip python3-venv unzip curl git
mkdir -p ~/work && cd ~/work
python3 -m venv .venv
source .venv/bin/activate
pip install --upgrade pip
pip install jupyterlab pandas numpy scikit-learn boto3 s3fs pyarrow matplotlib
pip freeze > requirements.txt
echo "setup done"
EOF

chmod +x ~/work/setup.sh
cat ~/work/setup.sh
```
> 오늘 20분 걸린 작업이 이 파일 하나면 3분입니다. **이게 자동화의 첫걸음**입니다. 12주차에 이 개념이 크게 확장됩니다.

### 팀 프로젝트 연결

- 오늘 만든 인스턴스 `mlops-<학번>` 은 **3~13주차 내내 여러분의 개발 서버**다. 매주 Stop/Start를 반복하며 쓴다.
- `~/work` 폴더가 **프로젝트 작업 디렉터리**가 된다. 3주차 전처리 스크립트, 4주차 학습 코드, 5주차 MLflow가 모두 여기 들어간다.
- `requirements.txt` 는 **6주차 Dockerfile의 재료**로 그대로 쓰인다. 지금부터 라이브러리를 추가할 때마다 갱신하는 습관을 들인다.
- 팀 확정(3주차) 후에는 **팀당 1대를 공용 서버로 쓸지, 개인별로 쓸지**를 정한다. 권장: 개발은 개인 인스턴스, 8주차 배포용 1대만 팀 공용.
- ⚠ **서버 안의 파일은 언제든 날아갈 수 있다.** 중요한 코드는 S3나 GitHub(12주차)에 올려두는 습관을 지금부터 들인다.

### 순회 지도 포인트

1. **가상환경이 활성화된 상태인가.** 프롬프트 앞에 `(.venv)` 가 붙어 있는지 본다. 없이 `pip install` 하는 학생이 많고, 나중에 "설치했는데 없다"는 문제로 돌아온다.
2. **`--ip=0.0.0.0` 을 넣었는가.** JupyterLab이 안 열린다는 학생의 절반이 이것 때문이다. 실행한 명령줄을 직접 눈으로 확인한다.
3. **`BUCKET` 변수에 본인 버킷 이름을 넣었는가.** 강사 예시인 `mlops-2026-demo` 를 그대로 두고 "됐는데요?" 하는 학생이 나온다. 본인 학번이 들어갔는지 확인한다.

추가 확인: 인스턴스에 태그 3종이 붙어 있는지, 보안그룹 소스가 `0.0.0.0/0` 으로 되어 있지 않은지 본다.

---

## 5. 체크포인트 (제출물)

제출처: LMS `2주차 체크포인트`. 파일명 `w02_<학번>_1.png`, `w02_<학번>_2.png`, `w02_<학번>_3.png`

| # | 제출물 | 형식 | 배점 |
|---|---|---|---|
| 1 | 서버 접속 성공 화면 — `whoami`, `nproc`, `free -h`, `df -h` 실행 결과가 한 화면에 보일 것 (SSH / Instance Connect / Session Manager 무엇이든 인정) | 스크린샷 | 0.6 |
| 2 | JupyterLab에서 `pd.read_csv(f"s3://{BUCKET}/raw/sample_bike.csv")` 실행 후 `df.head()` 결과 — **`BUCKET` 변수에 본인 학번이 보일 것** | 스크린샷 | 0.9 |
| 3 | 수업 종료 후 EC2 인스턴스 상태가 **`중지됨(stopped)`** 인 콘솔 화면 (인스턴스 이름 `mlops-<학번>` 이 보일 것) | 스크린샷 | 0.5 |
| **합계** | | | **2.0** |

> 제출물 3번은 **리소스 정리 타임에 그 자리에서** 찍는다. 집에 가서 찍으면 이미 몇 시간치 요금이 나간 뒤다.

### 평가 기준 (NCS 수행준거 연계)

- `2001070104_18v1.1` **인공지능 플랫폼 하드웨어 환경 구현하기**
  → 요구되는 연산 성능(vCPU 2 / RAM 4GB)과 저장 용량(30GB)에 맞는 컴퓨팅 자원을 선정해 실제로 구성하고, 구성한 자원의 사양을 서버에서 직접 확인(`nproc`, `free -h`, `df -h`)해 요구 사양과 일치함을 검증할 수 있는가. (제출물 1)
  → 구성한 하드웨어 자원에 대한 접근 통제(보안그룹 인바운드 규칙, 키페어 기반 인증)를 설정하고, 개방한 포트와 허용 대상의 근거를 설명할 수 있는가. (순회 구두 확인)
- `2001070104_18v1.2` **인공지능 플랫폼 소프트웨어 환경 구현하기**
  → 운영체제(Ubuntu LTS) 위에 인공지능 개발에 필요한 소프트웨어(Python, 가상환경, 분석 라이브러리, 개발 도구)를 설치하고 버전을 확인해 정상 동작을 검증할 수 있는가. (제출물 2 + `requirements.txt` 존재 확인)
  → 구축한 소프트웨어 환경에서 외부 저장소(S3)에 접근해 데이터를 읽고 쓰는 인터페이스가 정상 동작함을 확인할 수 있는가. (제출물 2, 실습 B-4-⑤)
  → 동일한 소프트웨어 환경을 재구성할 수 있도록 설치 절차를 스크립트/명세 파일로 남길 수 있는가. (`setup.sh`, `requirements.txt`)

> **감점 항목**: 제출물 3번(인스턴스 중지 확인)이 없거나, 다음 수업 전까지 인스턴스가 `실행 중`으로 확인되면 **-2점**.

---

## 6. 정리 & 다음 주 예고 (15분)

### 오늘 배운 것 3줄 요약

1. EC2는 **시간 단위로 빌리는 컴퓨터 한 대**이고, 만들 때 AMI·인스턴스 타입·EBS·키페어·보안그룹 다섯 가지를 정한다. 보안그룹은 **방화벽**이고 기본값은 전부 잠금이다.
2. 리눅스는 `pwd`(어디) → `ls`(뭐 있나) → `cd`(가자) 세 개가 기본기이고, Tab 키 자동완성이 절반을 해결한다. 학교망에서 22번이 막히면 **EC2 Instance Connect / Session Manager**로 우회한다.
3. Python 가상환경 안에 JupyterLab을 띄우면 **내 노트북 브라우저로 서울의 서버를 쓸 수 있고**, `boto3`/`s3fs`로 S3 데이터를 바로 DataFrame으로 읽을 수 있다.

### 다음 주 미리보기 한 문장

> 다음 주는 **프로젝트가 실제로 시작되는 회차**입니다. 공통 데이터로 "원본을 안 건드리고 가공 결과만 새로 쓰는" 전처리 스크립트를 하나 완성하고, **팀을 확정해서 우리 팀이 풀 문제를 정의**합니다. 오늘 만든 서버와 지난주 버킷을 그대로 씁니다.

### 리소스 정리 타임 체크 항목 (5분 · ⚠ 필수)

**강의 스크립트**
> 자, 지금부터 5분은 오늘 수업에서 **제일 중요한 5분**입니다. 다 같이 화면 보시고 저를 따라 하세요.
> 지금 여러분 서버는 켜져 있습니다. 그 말은 **지금 이 순간에도 돈이 나가고 있다**는 뜻입니다.
> 여기서 **종료(Terminate)가 아니라 중지(Stop)** 입니다. Terminate 하면 오늘 설치한 게 전부 날아가서 다음 주에 처음부터 다시 해야 합니다. Stop은 전원만 끄는 겁니다. 다음 주에 켜면 그대로 살아납니다.

| # | 체크 항목 | 방법 |
|---|---|---|
| 1 | JupyterLab 종료 | 서버 터미널에서 `pkill -f "jupyter lab"` |
| 2 | **EC2 인스턴스 중지** ⚠ | 콘솔 → EC2 → 인스턴스 선택 → **인스턴스 상태 → 인스턴스 중지(Stop)**. **Terminate 아님!** |
| 3 | 상태가 `중지됨(stopped)` 으로 바뀐 것 확인 | 목록에서 눈으로 확인 후 **스크린샷 촬영**(제출물 3번) |
| 4 | 조교 확인 | 조교가 자리를 돌며 화면을 눈으로 확인하고 명단에 체크 |
| 5 | S3 버킷 | **삭제 금지.** 그대로 둔다 |
| 6 | 키페어 `.pem` 파일 위치 재확인 | 다음 주에 필요하다. 지금 위치를 다시 메모 |

> ⚠ 인스턴스를 중지해도 EBS 저장 요금은 계속 발생합니다. 정확한 금액은 서울 리전의 현재 EBS 가격과 실제 용량으로 확인하고, 15주차에 필요 없는 볼륨까지 정리합니다.

---

## 7. 과제

**제출 기한: 3주차 수업 시작 전까지 · LMS 업로드**

1. **인스턴스 중지 → 재시작 → 재접속 연습** (필수 · 배점 있음)
   - 집이나 도서관에서 콘솔에 로그인해 인스턴스를 **시작(Start)** 한다.
   - **퍼블릭 IPv4 주소가 지난 시간과 달라진 것을 확인**하고, 바뀐 IP를 적는다.
   - 새 IP로 다시 접속한다. (SSH가 막히면 Instance Connect)
     - 학교 밖에서 접속하려면 보안그룹 인바운드의 소스 `내 IP`를 **다시 눌러 갱신**해야 한다.
   - **접속에 성공한 화면**과 **바뀐 IP가 보이는 콘솔 화면**을 캡처해 제출한다.
   - ⚠ **연습이 끝나면 반드시 다시 Stop 한다.** 켜둔 채로 제출하면 감점이다.

   **함께 생각해 볼 것 (한 문단 서술 제출)**
   > 퍼블릭 IP가 바뀌면 어떤 문제가 생길까요? 만약 여러분이 만든 서비스 주소를 친구에게 알려줬는데, 서버를 껐다 켤 때마다 주소가 바뀐다면? 이 문제를 어떻게 해결할 수 있을지 한 문단으로 적어 오세요. (8주차에 **탄력적 IP(Elastic IP)** 로 답을 맞춰봅니다.)

2. **명령어 정리 노트** (필수)
   - 부록 A 치트시트의 명령 15개를 **본인 말로 한 줄씩 설명**을 붙여 정리한다.
   - 오늘 실습 중 **본인이 실제로 막혔던 문제 1개**와 어떻게 해결했는지를 3~5줄로 적는다. (해결 못 했으면 그것도 그대로 적는다 — 강사가 3주차 시작 전에 처리한다)

3. **팀 구성 최종 확인** (필수)
   - 1주차에 낸 팀 구성 희망서를 바탕으로 배정된 **잠정 팀 명단**이 LMS에 게시된다. 확인하고 이의가 있으면 회신한다.
   - 3주차에 팀이 **확정**되고, 그 자리에서 도메인을 고른다. 도메인 3종(A 회귀 / B 분류 / C 텍스트+LLM) 설명을 미리 읽어온다.

---

## 부록 A. 명령어 치트시트

> **2주차 · EC2 · 리눅스 · Python 환경** — 1페이지로 뽑아 배포
> `<학번>`, `<퍼블릭IP>`, `<인스턴스ID>` 자리에 본인 값을 넣는다.

```bash
# ══ 1. 서버 접속 ════════════════════════════════════════
# (A) SSH — 22번 포트가 열려 있을 때
ssh -i ~/.ssh/mlops-key-<학번>.pem ubuntu@<퍼블릭IP>

# (A-1) macOS/Linux 키 권한 오류가 날 때
chmod 400 ~/.ssh/mlops-key-<학번>.pem

# (B) 22번이 막혔을 때 → 콘솔에서 EC2 Instance Connect 또는 Session Manager
#     Session Manager 접속 후에는 계정을 바꿔준다
sudo su - ubuntu

# (C) JupyterLab 포트가 막혔을 때 — SSH 터널 (노트북에서 새 창)
ssh -i ~/.ssh/mlops-key-<학번>.pem -N -L 8888:localhost:8888 ubuntu@<퍼블릭IP>

# (D) 22번도 막혔을 때 — Session Manager 포트 포워딩 (노트북에서)
aws ssm start-session --target <인스턴스ID> \
  --document-name AWS-StartPortForwardingSession \
  --parameters '{"portNumber":["8888"],"localPortNumber":["8888"]}' \
  --region ap-northeast-2

# ══ 2. 리눅스 생존 명령 15 ══════════════════════════════
pwd                 # 지금 어디            ┐
ls -al              # 여기 뭐 있나         ├ 3대 기본기
cd ~/work           # 저기로 가자          ┘  (cd .. = 한 단계 위)
mkdir -p ~/work/data    # 폴더 만들기 (-p = 중간 경로도 같이)
cp a.csv b.csv          # 복사
mv a.csv data/          # 이동 / 이름 변경
rm a.csv                # 삭제 ⚠ 휴지통 없음
cat note.txt            # 전부 출력
less big.csv            # 페이지 단위로 보기 (q 로 나감)
tail -f ~/jupyter.log   # 파일 끝을 실시간 감시 (Ctrl+C 로 중단)
chmod 400 key.pem       # 권한 변경
ps aux | grep jupyter   # 도는 프로그램 찾기
kill -9 <PID>           # 강제 종료
df -h                   # 디스크 남은 용량
top                     # CPU·메모리 실시간 (q 로 나감)

# 서버 사양 확인 3종
nproc      # CPU 개수  → 2
free -h    # 메모리    → 약 3.8Gi
df -h      # 디스크    → / 가 약 29G

# ══ 3. 패키지 설치 (apt) ════════════════════════════════
sudo apt update                                  # 목록 갱신 (항상 먼저!)
sudo apt install -y python3-pip python3-venv unzip curl git

# ══ 4. Python 가상환경 ══════════════════════════════════
cd ~/work
python3 -m venv .venv          # 방 만들기 (최초 1회)
source .venv/bin/activate      # 방 들어가기 ★ 접속할 때마다 매번
deactivate                     # 방 나가기
pip install --upgrade pip
pip install jupyterlab pandas numpy scikit-learn boto3 s3fs pyarrow matplotlib
pip freeze > requirements.txt  # 설치 목록 저장 (6주차 도커 재료)

# ══ 5. JupyterLab ═══════════════════════════════════════
jupyter lab password           # 접속 비밀번호 설정 (최초 1회)
jupyter lab --ip=0.0.0.0 --port=8888 --no-browser     # ★ --ip=0.0.0.0 필수
# 백그라운드로 띄우기
nohup jupyter lab --ip=0.0.0.0 --port=8888 --no-browser > ~/jupyter.log 2>&1 &
tail -20 ~/jupyter.log         # 잘 떴는지 확인
pkill -f "jupyter lab"         # 종료
# 접속: http://<퍼블릭IP>:8888   (터널 사용 시 http://localhost:8888)

# ══ 6. Python에서 S3 다루기 ═════════════════════════════
```
```python
import boto3, pandas as pd

BUCKET = "mlops-2026-<학번>"

# 목록 보기
s3 = boto3.client("s3", region_name="ap-northeast-2")
for obj in s3.list_objects_v2(Bucket=BUCKET, Prefix="raw/").get("Contents", []):
    print(obj["Key"], obj["Size"])

# 읽기 (권장 방식)
df = pd.read_csv(f"s3://{BUCKET}/raw/sample_bike.csv")

# 쓰기
df.head(100).to_csv(f"s3://{BUCKET}/processed/sample_head100.csv", index=False)

# 파일로 내려받기 / 올리기
s3.download_file(BUCKET, "raw/sample_bike.csv", "/home/ubuntu/work/sample_bike.csv")
s3.upload_file("/home/ubuntu/work/out.csv", BUCKET, "processed/out.csv")
```

**수업 끝나기 전 반드시 할 것**
`콘솔 → EC2 → 인스턴스 선택 → 인스턴스 상태 → 인스턴스 중지(Stop)` — **Terminate 아님**

---

## 부록 B. 용어 정리

| 용어 | 뜻 | 한 줄 설명 |
|---|---|---|
| 서버 (Server) | 안 꺼지고 남이 접속하는 컴퓨터 | 성능이 아니라 **가용성과 주소**가 본질 |
| EC2 | Elastic Compute Cloud | 시간 단위로 빌리는 클라우드 컴퓨터 서비스 |
| 인스턴스 (Instance) | 빌린 컴퓨터 한 대 | `mlops-<학번>` 이 오늘 만든 우리 인스턴스 |
| AMI | Amazon Machine Image | OS가 미리 설치된 복사본. 우리는 Ubuntu LTS |
| Ubuntu / LTS | 리눅스 배포판 / 장기 지원 버전 | 로그인 계정 이름이 **`ubuntu`** (Amazon Linux는 `ec2-user`) |
| 인스턴스 타입 | CPU·메모리 사양 등급 | 우리는 `t3.medium` (vCPU 2 / RAM 4GB) |
| EBS | Elastic Block Store | 인스턴스에 붙은 하드디스크. ⚠ **꺼도 과금** |
| S3 vs EBS | 인터넷 창고 vs 내장 하드 | S3는 아무 서버에서나 접근, EBS는 그 서버 전용 |
| 키페어 (Key Pair) | 서버 출입 열쇠 한 쌍 | 공개키는 서버에, 개인키(`.pem`)는 내가. **재발급 불가** |
| SSH | Secure Shell | 암호화된 통로로 원격 서버에 접속해 명령을 치는 방법. 22번 포트 |
| 보안그룹 (Security Group) | 방화벽 | 어느 포트를 누구에게 열지 정하는 규칙표. **기본값 전부 잠금** |
| 포트 (Port) | 서버 건물의 문 번호 | 22=SSH, 8888=Jupyter, 8000=FastAPI, 8501=Streamlit |
| 인바운드 / 아웃바운드 | 들어오는 / 나가는 트래픽 | 우리가 여는 건 대부분 인바운드 |
| `0.0.0.0/0` | 전 세계 아무나 | 편하지만 위험. 22번에는 쓰지 않는다 |
| 퍼블릭 IP | 인터넷에서 보이는 주소 | ⚠ **Stop/Start 하면 바뀐다** (해결책은 8주차 탄력적 IP) |
| EC2 Instance Connect | 브라우저 기반 접속 | 키 파일 없이 콘솔 안에서 터미널을 연다. 22번 차단 시 1순위 대안 |
| Session Manager | SSM 기반 접속 | **포트를 전혀 쓰지 않는** 접속. 방화벽 차단을 완전히 우회 |
| 포트 포워딩 / 터널링 | 통로 만들기 | 내 노트북의 포트를 서버 포트로 연결. `localhost:8888` 로 접속 |
| Stop (중지) | 전원 끄기 | 서버 요금 정지. **EBS 요금은 계속.** 데이터 유지 |
| Terminate (종료) | 폐기 | 인스턴스와 EBS까지 삭제. **복구 불가.** 15주차에만 |
| `apt` | 우분투 패키지 관리자 | `sudo apt update` → `sudo apt install -y <패키지>` 세트로 |
| `sudo` | 관리자 권한으로 실행 | 시스템 전체를 건드릴 때만 붙인다 |
| 가상환경 (venv) | 프로젝트 전용 파이썬 방 | `python3 -m venv .venv` → `source .venv/bin/activate` |
| `requirements.txt` | 설치 목록 명세 | `pip freeze > requirements.txt`. 6주차 Dockerfile의 재료 |
| JupyterLab | 브라우저 기반 분석 도구 | `--ip=0.0.0.0` 을 붙여야 외부에서 접속된다 |
| boto3 | AWS용 Python 라이브러리 | 코드로 S3·EC2 등을 다룬다 |
| s3fs | S3를 파일처럼 쓰게 해주는 라이브러리 | `pd.read_csv("s3://버킷/키")` 가 가능해지는 이유 |
| IAM 인스턴스 프로파일 | 서버에 붙이는 권한 | 코드에 액세스 키를 안 적어도 되게 해준다 |
| `nohup ... &` | 백그라운드 실행 | 터미널을 닫아도 프로그램이 계속 돈다 |

---

## 부록. AWS 화면과 공식 문서

![EC2 고급 설정 화면 예시](../assets/aws/ec2-user-data.jpg)

- 콘솔: <https://console.aws.amazon.com/ec2/>
- 이동 경로: **EC2 → Instances → Launch instances**
- 화면에서 확인: AMI, 인스턴스 유형, 키페어 또는 연결 방식, 보안그룹, 스토리지, IAM 역할
- 공식 문서: <https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-launch-instance-wizard.html>
- 접속 대안: <https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager.html>

```mermaid
flowchart LR
    A[학생 브라우저/터미널] --> B[Instance Connect 또는 Session Manager]
    B --> C[EC2]
    C --> D[Python 가상환경]
    C --> E[JupyterLab]
    C --> F[S3 접근용 인스턴스 역할]
```+

---

## 실제 AWS 콘솔 화면 실습 가이드 (2주차)

> 2026-08-24 서울 리전 콘솔에서 직접 캡처했다. 계정 번호가 보이는 오른쪽 상단은 제외했다. 화면 모양이 바뀌면 메뉴 경로와 확인 항목을 기준으로 찾는다.

### EC2 대시보드

![EC2 대시보드](../assets/aws-console/week02/01-ec2-dashboard.png)

- 콘솔 경로: **EC2 → Dashboard**
- 확인할 것: 서울 리전과 실행 자원 수 확인
- [같은 화면 열기](https://console.aws.amazon.com/ec2/)

### EC2 인스턴스 목록

![EC2 인스턴스 목록](../assets/aws-console/week02/02-ec2-instances.png)

- 콘솔 경로: **EC2 → Instances**
- 확인할 것: 상태·이름·인스턴스 유형 확인
- [같은 화면 열기](https://console.aws.amazon.com/ec2/home?region=ap-northeast-2#Instances:)

### 인스턴스 시작: 이름과 AMI

![인스턴스 시작: 이름과 AMI](../assets/aws-console/week02/03-ec2-launch-name-ami.png)

- 콘솔 경로: **Instances → Launch instances**
- 확인할 것: Amazon Linux AMI와 이름 태그 확인
- [같은 화면 열기](https://console.aws.amazon.com/ec2/home?region=ap-northeast-2#LaunchInstances:)

### 인스턴스 시작: 유형과 키 페어

![인스턴스 시작: 유형과 키 페어](../assets/aws-console/week02/04-ec2-launch-instance-type.png)

- 콘솔 경로: **Launch instances → Instance type**
- 확인할 것: 허용 유형과 접속 방식 확인
- [같은 화면 열기](https://console.aws.amazon.com/ec2/home?region=ap-northeast-2#LaunchInstances:)

### 인스턴스 시작: 네트워크

![인스턴스 시작: 네트워크](../assets/aws-console/week02/05-ec2-launch-network.png)

- 콘솔 경로: **Launch instances → Network settings**
- 확인할 것: VPC·서브넷·퍼블릭 IP 확인
- [같은 화면 열기](https://console.aws.amazon.com/ec2/home?region=ap-northeast-2#LaunchInstances:)

### 인스턴스 시작: 스토리지

![인스턴스 시작: 스토리지](../assets/aws-console/week02/06-ec2-launch-storage.png)

- 콘솔 경로: **Launch instances → Configure storage**
- 확인할 것: EBS 크기와 종료 시 삭제 확인
- [같은 화면 열기](https://console.aws.amazon.com/ec2/home?region=ap-northeast-2#LaunchInstances:)

### 인스턴스 시작: 고급 세부 정보

![인스턴스 시작: 고급 세부 정보](../assets/aws-console/week02/07-ec2-launch-advanced.png)

- 콘솔 경로: **Launch instances → Advanced details**
- 확인할 것: IAM 역할과 사용자 데이터 확인
- [같은 화면 열기](https://console.aws.amazon.com/ec2/home?region=ap-northeast-2#LaunchInstances:)

### EC2 보안 그룹 목록

![EC2 보안 그룹 목록](../assets/aws-console/week02/08-ec2-security-groups.png)

- 콘솔 경로: **EC2 → Security Groups**
- 확인할 것: 인바운드 규칙과 연결 VPC 확인
- [같은 화면 열기](https://console.aws.amazon.com/ec2/home?region=ap-northeast-2#SecurityGroups:)

### EC2 키 페어 목록

![EC2 키 페어 목록](../assets/aws-console/week02/09-ec2-key-pairs.png)

- 콘솔 경로: **EC2 → Key Pairs**
- 확인할 것: 키 이름과 생성 방식 확인
- [같은 화면 열기](https://console.aws.amazon.com/ec2/home?region=ap-northeast-2#KeyPairs:)

### EBS 볼륨 목록

![EBS 볼륨 목록](../assets/aws-console/week02/10-ec2-volumes.png)

- 콘솔 경로: **EC2 → Volumes**
- 확인할 것: 미연결 볼륨과 삭제 여부 확인
- [같은 화면 열기](https://console.aws.amazon.com/ec2/home?region=ap-northeast-2#Volumes:)
