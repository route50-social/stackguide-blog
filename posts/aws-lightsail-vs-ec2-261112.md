# Amazon Lightsail vs EC2 저사양 인스턴스: AWS 첫 서버는 무엇으로 시작할까

> AWS에서 처음 서버를 띄울 때 Lightsail과 EC2(t4g·t3 nano~small) 중 무엇을 고를지 서울 리전 요금, 사양과 CPU 성능 기준, 설정 난이도, 확장 방법을 2026년 11월 기준으로 비교합니다.

- 주소: https://stackguide.route50.kr/posts/aws-lightsail-vs-ec2-261112/
- 분류: AWS
- 작성일: 2026-11-12
- 정보 기준일: 2026-11-12
- 태그: AWS, Lightsail, EC2, VPS, 가상머신, 클라우드비용, 프리티어, 공인IP, 아웃바운드트래픽, EBS, 버스터블인스턴스, CPU크레딧, Graviton, AMI, VPC, 보안그룹, 스냅샷, 로드밸런서, IPv6, SSH

AWS에서 처음 서버 한 대를 띄우려고 하면 길이 두 개 나옵니다. 월 몇 달러짜리 요금표가 한눈에 보이는 Lightsail, 그리고 AWS 서버의 기본인 EC2입니다. 둘 다 같은 AWS 데이터센터에서 도는 가상 서버인데, 어느 쪽으로 시작하느냐에 따라 한 달 요금을 계산하는 방식, 처음 설정할 항목의 수, 나중에 서버를 키우는 방법이 달라집니다.

> **용어 풀이**
>
> - **Lightsail(Amazon Lightsail):** AWS가 서버, 디스크, 공인 IP, 일정량의 트래픽을 묶어 월 고정 요금으로 파는 간편형 가상 서버 서비스입니다.
> - **EC2(Amazon Elastic Compute Cloud):** AWS의 기본 가상 서버 서비스입니다. 서버 사양(인스턴스 유형), 디스크, 네트워크를 따로 골라 조합하고 쓴 만큼 요금을 냅니다.
> - **인스턴스:** AWS에서 빌려 쓰는 가상 서버 한 대를 부르는 말입니다. Lightsail과 EC2 모두 이 이름을 씁니다.
> - **VPS(Virtual Private Server, 가상 사설 서버)·가상 머신:** 물리 서버 한 대를 소프트웨어로 나눠 만든 독립된 가상의 컴퓨터입니다. Lightsail 문서는 인스턴스를 VPS 또는 가상 머신이라고 설명합니다.

짧게 정리하면, **작은 웹 서비스 하나를 정해진 예산으로 돌리는 것이 목표라면 Lightsail이 싸고 단순합니다.** 반대로 다른 AWS 서비스와 묶어 쓰거나, 서버를 자주 키우고 줄이거나, AWS 자체를 배우는 것이 목표라면 처음부터 EC2로 가는 편이 낫습니다. 아래에서 요금, 사양과 CPU 성능, 설정 난이도, 확장 가능성의 네 가지 기준으로 근거를 보여 드립니다.

요금은 모두 2026년 11월 12일에 AWS 공식 요금 페이지와 AWS Price List API에서 확인한 **서울 리전(ap-northeast-2), 리눅스, 세금 별도 미국 달러** 기준입니다. AWS 계정을 새로 만들어 직접 서버를 띄워 보지는 않았고, 설정 과정은 공식 시작 안내서로 확인했습니다.

## 요금 구조부터 다르다

Lightsail은 **플랜(번들) 하나에 필요한 것이 다 들어 있는** 방식입니다. 2GB 메모리 플랜을 예로 들면 월 $12에 vCPU 2개, SSD 60GB, 공인 IPv4 주소, 매달 3TB의 데이터 전송량이 포함됩니다. 고정 IP(Static IP)도 인스턴스에 붙어 있는 동안은 따로 요금이 없습니다.

EC2는 **항목마다 따로 계산서가 붙는** 방식입니다. 인스턴스 요금은 서버의 CPU와 메모리 값일 뿐이고, 여기에 다음이 더해집니다.

- 디스크(EBS): 서울 리전 gp3 기준 GB당 월 $0.0912
- 공인 IPv4 주소: 사용 중이든 놀고 있든 주소 1개당 시간당 $0.005 (한 달 730시간이면 약 $3.65)
- 인터넷으로 나가는 트래픽: 계정 전체에서 매달 100GB까지 무료, 그 뒤 첫 10TB까지 GB당 $0.126

> **용어 풀이**
>
> - **번들(플랜):** Lightsail에서 서버 사양과 디스크, 전송량을 하나로 묶은 요금 단위입니다. 월 $5, $7, $12처럼 가격으로 구분합니다.
> - **vCPU(virtual CPU, 가상 CPU):** 가상 서버에 배정되는 CPU 단위입니다. 같은 vCPU 2개라도 기준 성능이 다를 수 있어 아래에서 따로 비교합니다.
> - **EBS(Amazon Elastic Block Store):** EC2 인스턴스에 붙이는 디스크 서비스입니다. gp3는 그중 범용 SSD 유형이며, 인스턴스 요금과 별도로 용량에 따라 청구됩니다.
> - **공인 IPv4 주소:** 인터넷에서 서버를 찾아올 수 있게 하는 공개 주소입니다. AWS는 EC2 등에 붙은 공인 IPv4 주소에 시간당 요금을 받습니다.
> - **고정 IP(Static IP)·Elastic IP:** 서버를 껐다 켜도 바뀌지 않는 공인 IP입니다. Lightsail에서는 Static IP, EC2에서는 Elastic IP라고 부릅니다.
> - **아웃바운드 트래픽:** 서버에서 인터넷 쪽으로 나가는 데이터 양입니다. 방문자가 페이지나 이미지를 내려받을 때 생기며, 들어오는 트래픽보다 요금이 비쌉니다.

### 메모리 크기별로 나란히 놓으면

같은 메모리 크기끼리 짝을 지어 한 달(730시간) 내내 켜 둔 경우를 계산했습니다. EC2 쪽 디스크는 비교를 위해 같은 메모리의 Lightsail 플랜과 같은 용량의 gp3를 붙였다고 가정했습니다.

| 메모리 | Lightsail (월 고정) | EC2 Arm (t4g) | EC2 Intel (t3) |
| --- | --- | --- | --- |
| 0.5GB | **$5** (SSD 20GB, 전송 1TB 포함) | t4g.nano $3.80 + 디스크 $1.82 + IP $3.65 = **약 $9.27** | t3.nano $4.75 + $1.82 + $3.65 = **약 $10.22** |
| 1GB | **$7** (SSD 40GB, 전송 2TB 포함) | t4g.micro $7.59 + $3.65 + $3.65 = **약 $14.89** | t3.micro $9.49 + $3.65 + $3.65 = **약 $16.79** |
| 2GB | **$12** (SSD 60GB, 전송 3TB 포함) | t4g.small $15.18 + $5.47 + $3.65 = **약 $24.31** | t3.small $18.98 + $5.47 + $3.65 = **약 $28.10** |
| 4GB | **$24** (SSD 80GB, 전송 4TB 포함) | t4g.medium $30.37 + $7.30 + $3.65 = **약 $41.31** | t3.medium $37.96 + $7.30 + $3.65 = **약 $48.91** |

*표 기준: 2026-11-12 확인. Lightsail은 공식 요금 페이지와 서울 리전 Price List API(2026-09-15 발행), EC2·EBS·공인 IPv4는 서울 리전 Price List API(EC2 2026-11-10, VPC 2026-11-05 발행). EC2 시간 요금은 t4g.nano $0.0052, t3.nano $0.0065, t4g.micro $0.0104, t3.micro $0.013, t4g.small $0.0208, t3.small $0.026, t4g.medium $0.0416, t3.medium $0.052이며 730시간 환산과 합계는 필자 계산. EC2 트래픽 요금은 월 100GB 무료 범위 안이라고 보고 뺐습니다.*

> **용어 풀이**
>
> - **Graviton(AWS Graviton):** AWS가 직접 설계한 Arm 기반 서버용 CPU입니다. t4g 같은 이름의 `g`가 Graviton을 뜻하며, 같은 크기의 Intel 기반 t3보다 시간 요금이 20% 낮습니다. 다만 Arm용으로 빌드되지 않은 프로그램은 돌릴 수 없습니다.

같은 2GB 메모리라도 EC2 t4g.small 구성은 Lightsail $12 플랜의 두 배 가까이 됩니다. 차이의 상당 부분은 인스턴스 요금이 아니라 **공인 IP와 디스크**에서 나옵니다. 작은 서버일수록 이 고정 항목의 비중이 커서, 0.5GB 구성에서는 공인 IP 요금($3.65)이 서버 요금($3.80)과 거의 같습니다.

*(그림: 2GB 메모리 서버를 한 달 켜 둘 때의 요금 구성 막대 그래프. Lightsail 2GB 플랜은 12달러 한 덩어리이고, EC2 t4g.small 구성은 인스턴스 15.18달러, 60GB gp3 디스크 5.47달러, 공인 IPv4 3.65달러를 합쳐 약 24.31달러, t3.small 구성은 같은 방식으로 약 28.10달러입니다.)*

*그림 1. 메모리 2GB 서버를 서울 리전에서 한 달(730시간) 켜 둘 때의 요금 구성 (자료: AWS Price List API·Lightsail 요금 페이지, 2026-11-12 확인. 트래픽은 EC2 무료 100GB 안이라고 가정)*

### 트래픽이 늘면 차이가 더 벌어진다

표에서는 트래픽을 뺐지만, 방문자가 늘면 여기서 차이가 커집니다. 한 달에 300GB를 내보내는 서비스라면 EC2는 무료 100GB를 뺀 200GB × $0.126 = **약 $25.2**가 더 나옵니다. Lightsail 2GB 플랜은 3TB까지 포함되어 있어 추가 요금이 없습니다. 게다가 EC2의 무료 100GB는 계정 안의 모든 서비스와 리전을 합친 양이라, 다른 서비스를 함께 쓰면 그만큼 줄어듭니다.

Lightsail의 전송량을 셀 때는 들어오는 트래픽과 나가는 트래픽을 모두 합산하지만, 초과 요금은 나가는 트래픽에만 붙습니다. 서울 리전의 초과 단가는 FAQ 기준 GB당 $0.13입니다. 전송량은 같은 리전, 같은 플랜의 인스턴스끼리 합쳐서 계산하고, 인스턴스를 지웠다가 다시 만들어도 그달 사용량이 초기화되지 않습니다.

### IPv6 전용 플랜으로 더 줄일 수 있지만

Lightsail에는 공인 IPv4 없이 IPv6 주소만 주는 플랜이 따로 있습니다. 0.5GB가 월 $3.50, 2GB가 월 $10으로 IPv4 포함 플랜보다 $1.50~$2 쌉니다. 다만 IPv6로 접속할 수 없는 사용자와 서비스가 아직 있고, 고정 IP는 IPv6 전용 인스턴스에 붙일 수 없습니다. 일부 블루프린트는 IPv6 전용을 지원하지 않으니, 일반 공개 웹 서비스라면 IPv4 포함 플랜이 무난합니다.

> **용어 풀이**
>
> - **IPv6:** IPv4 주소가 모자라는 문제를 풀기 위해 만든 새 인터넷 주소 체계입니다. 주소가 사실상 무한하지만, 아직 IPv6를 지원하지 않는 네트워크에서는 IPv6 전용 서버에 접속할 수 없습니다.
> - **블루프린트:** Lightsail 인스턴스를 만들 때 고르는 이미지입니다. 운영체제만 있는 것(OS Only)과 WordPress 같은 앱이 미리 설치된 것(Apps + OS)이 있습니다.

### 무료로 시작하는 방법도 달라졌다

예전에는 Lightsail 일부 플랜을 3개월 무료로 써 볼 수 있었지만, 지금은 Lightsail 공식 FAQ에 **"Lightsail의 단기 무료 체험은 AWS 프리 티어로 대체되었고 새 고객에게는 더 이상 제공되지 않는다"**고 적혀 있습니다. 대신 2025년 7월 15일 이후 가입한 계정은 가입 크레딧 $100과 지정 활동으로 받는 최대 $100, 합계 최대 $200 크레딧으로 Lightsail과 EC2를 함께 쓸 수 있습니다. 다만 Lightsail 시작 안내서에는 아직 "$5 플랜을 한 달(최대 750시간) 무료로 써 볼 수 있다"는 문장이 남아 있어 문서끼리 어긋납니다. 실제로 적용되는지는 가입 후 콘솔의 요금 화면에서 확인하세요.

EC2는 같은 크레딧 안에서 **t3.micro, t3.small, t4g.micro, t4g.small, c7i-flex.large, m7i-flex.large**가 프리 티어 대상 유형으로 표시됩니다(EC2 프리 티어 문서 기준). 무료 플랜 계정은 가입 후 6개월이 지나거나 크레딧을 다 쓰면 끝나며, 그 전에 유료 플랜으로 바꾸지 않으면 계정이 닫히고 90일 뒤 데이터가 삭제됩니다.

> **용어 풀이**
>
> - **프리 티어(Free Tier):** AWS가 신규 고객에게 일정량을 무료로 주는 제도입니다. 2025년 7월 15일 이후 가입한 계정은 기간 한정 크레딧 방식이 적용됩니다.

## 사양: 같은 "vCPU 2개"도 계속 쓸 수 있는 양은 정해져 있다

Lightsail의 $5~$44 플랜과 EC2의 t3·t4g는 모두 **버스터블(burstable)** 방식입니다. 평소에는 정해진 기준 성능만큼만 CPU를 쓰다가, 아껴 둔 여유분(크레딧)을 써서 잠깐 더 높은 성능을 냅니다. 사양표의 "vCPU 2개"는 두 개를 항상 100%로 쓸 수 있다는 뜻이 아닙니다.

> **용어 풀이**
>
> - **버스터블 인스턴스:** 기준 CPU 사용률(baseline)까지는 계속 쓸 수 있고, 그 이상은 쌓아 둔 여유분으로 잠깐만 쓰는 저가형 서버입니다. 웹사이트처럼 평소에는 한가하다가 가끔 바빠지는 서비스에 맞습니다.
> - **CPU 크레딧:** EC2 버스터블 인스턴스가 기준 성능 아래로 쓸 때 쌓이고, 기준을 넘을 때 쓰는 여유분입니다. Lightsail에서는 같은 개념을 "버스트 용량(burst capacity)"이라고 부릅니다.

두 서비스의 기준 성능은 같은 메모리 크기에서 똑같습니다.

| 메모리 | Lightsail 플랜 | EC2 유형 | vCPU당 기준 성능 |
| --- | --- | --- | --- |
| 0.5GB | $5 | t3.nano, t4g.nano | 5% |
| 1GB | $7 | t3.micro, t4g.micro | 10% |
| 2GB | $12 | t3.small, t4g.small | 20% |
| 4GB | $24 | t3.medium, t4g.medium | 20% |

*표 기준: Lightsail 기준 CPU 성능 문서(2023년 6월 29일 이후 만든 인스턴스), EC2 버스터블 인스턴스 핵심 개념 문서. 2026-11-12 확인.*

예를 들어 $12 플랜이나 t4g.small은 vCPU 2개 각각 20%, 즉 CPU 1개의 40% 정도를 꾸준히 쓸 수 있는 서버입니다. 작은 블로그나 API 서버에는 충분하지만, 동영상 변환이나 빌드 작업을 계속 돌리기에는 모자랍니다.

### 크레딧이 바닥났을 때의 동작이 다르다

차이는 여유분을 다 썼을 때 나타납니다.

- **Lightsail:** 기준 성능까지 내려가 느려집니다. 공식 문서는 버스트 구간에 오래 머물면 결국 용량을 다 써서 "지속 가능 구간(sustainable zone)"으로 돌아간다고 설명하며, 추가 요금 이야기는 없습니다.
- **EC2 t3·t4g:** 처음 만들 때 기본값이 **Unlimited 모드**입니다. 크레딧을 다 써도 느려지지 않고 계속 높은 성능을 내는 대신, 24시간 평균 사용률이 기준을 넘으면 넘은 만큼 vCPU시간당 요금을 따로 받습니다. 서울 리전 리눅스 기준 t4g는 $0.04, t3는 $0.05입니다.

예를 들어 t4g.small을 한 달(730시간) 내내 vCPU당 평균 50%로 돌리면, 기준 20%를 넘는 30% × vCPU 2개 × 730시간 = 438 vCPU시간이 초과분이 되어 **약 $17.52**가 인스턴스 요금에 더해집니다(필자 계산). 이런 요금이 싫다면 인스턴스의 크레딧 설정을 Standard 모드로 바꾸면 됩니다. 그러면 EC2도 Lightsail처럼 크레딧이 바닥나면 기준 성능으로 내려갑니다. 단, Unlimited에서 Standard로 바꾸는 순간 그때까지 쌓인 초과분은 바로 청구됩니다.

*(그림: 크레딧이 바닥난 뒤의 동작을 세 갈래로 비교한 그림. Lightsail과 EC2 Standard 모드는 기준 성능으로 내려가 느려지지만 추가 요금이 없고, EC2 Unlimited 모드(t3·t4g 기본값)는 성능을 유지하는 대신 24시간 평균이 기준을 넘으면 vCPU시간당 추가 요금이 붙습니다.)*

*그림 2. CPU 여유분을 다 쓴 뒤 서비스별 동작 (자료: Lightsail 버스트 문서, EC2 Unlimited 모드 문서, 2026-11-12 확인)*

어느 쪽이 좋다기보다는 선택의 문제입니다. 갑자기 몰린 방문자를 느려지지 않고 받아 내야 한다면 EC2 Unlimited가 낫고, 요금이 예상 밖으로 나오는 것이 더 걱정이라면 Lightsail이나 EC2 Standard 모드가 맞습니다. 어느 쪽이든 CPU 사용률 그래프가 기준선을 자주 넘는다면 서버가 작다는 신호이니 한 단계 큰 사양을 검토하세요.

## 설정 난이도: 처음 정해야 할 항목의 수

AWS 공식 시작 안내서를 기준으로 서버 한 대를 만들 때 고르는 항목을 비교하면 차이가 분명합니다.

| 단계 | Lightsail | EC2 |
| --- | --- | --- |
| 위치 | 리전·가용 영역 | 리전 |
| 이미지 | 블루프린트(OS만, 또는 WordPress 등 앱 포함) | AMI(운영체제 이미지) |
| 사양 | 플랜 하나 선택(디스크·전송량 포함) | 인스턴스 유형 + EBS 볼륨 크기·종류 따로 |
| 접속 | 콘솔에서 브라우저 SSH 바로 사용 가능 | 키 페어를 만들어 내려받거나 기존 키 선택 |
| 네트워크 | 기본값 사용, 방화벽은 인스턴스 화면에서 포트 단위로 관리 | VPC·서브넷·공인 IP 자동 할당·보안 그룹 설정 |

> **용어 풀이**
>
> - **AMI(Amazon Machine Image):** EC2 인스턴스를 만들 때 쓰는 운영체제·소프트웨어 이미지입니다.
> - **SSH(Secure Shell):** 원격 서버에 암호화된 연결로 접속해 명령을 실행하는 방법입니다. 리눅스 서버를 관리할 때 기본으로 씁니다.
> - **키 페어:** SSH 접속에 쓰는 공개 키와 개인 키 한 쌍입니다. 공개 키는 서버에, 개인 키는 내 컴퓨터에 두며, 개인 키를 잃어버리면 그 키로는 다시 접속할 수 없습니다.
> - **VPC(Virtual Private Cloud):** AWS 안에 만드는 나만의 가상 네트워크입니다. 계정에는 리전마다 기본 VPC가 하나씩 만들어져 있습니다.
> - **서브넷:** VPC를 더 작게 나눈 네트워크 구역입니다. 인스턴스는 어느 한 서브넷 안에 놓입니다.
> - **보안 그룹:** EC2 인스턴스 앞에 두는 가상 방화벽입니다. 어떤 주소에서 어떤 포트로 들어오는 연결을 허용할지 규칙으로 정합니다.

EC2 시작 안내서도 첫 인스턴스는 기본 VPC와 기본 설정을 그대로 쓰라고 권하므로, 항목이 많다고 해서 처음부터 모든 것을 이해해야 하는 것은 아닙니다. 다만 기본 보안 그룹은 SSH(22번 포트)를 모든 주소(`0.0.0.0/0`)에서 허용하도록 제안되는데, 안내서 스스로 "시험 환경에서 잠깐이라면 몰라도 운영 환경에는 안전하지 않다"고 경고합니다. 운영할 서버라면 SSH는 내 IP 주소에서만 열어 두세요. Lightsail도 기본 방화벽 규칙이 SSH를 모든 IP 주소에 열어 두니 같은 원칙으로 접속 허용 주소를 좁히는 것이 좋습니다.

IP 주소도 한 번 짚고 넘어가야 합니다. 두 서비스 모두 처음 받은 공인 IPv4 주소는 서버를 **중지했다가 다시 켜면 바뀝니다.** 도메인을 연결할 서버라면 고정 IP를 붙여야 하는데, Lightsail의 Static IP는 인스턴스에 붙어 있는 동안 무료이고 떼어 놓은 채 1시간이 지나면 시간당 $0.005가 붙습니다. EC2의 Elastic IP는 붙어 있든 아니든 공인 IPv4 요금(시간당 $0.005)이 계속 나갑니다.

## 확장 가능성: 키우는 방법과 갈아타는 길

서비스가 커졌을 때를 생각하면 두 서비스의 성격 차이가 가장 크게 드러납니다.

**Lightsail은 "스냅샷을 찍고 더 큰 플랜으로 새로 만드는" 방식입니다.** 실행 중인 인스턴스의 플랜을 그 자리에서 바꾸는 기능은 없고, 스냅샷에서 더 큰 플랜의 새 인스턴스를 만든 뒤 고정 IP를 옮겨 붙이는 순서로 키웁니다. 더 작은 플랜으로 줄이는 것은 문서상 불가능합니다. 디스크는 블록 스토리지를 GB당 월 $0.10에 추가로 붙일 수 있고, 여러 대로 나누려면 월 $18의 Lightsail 로드 밸런서를 씁니다.

> **용어 풀이**
>
> - **스냅샷:** 특정 시점의 디스크 내용을 통째로 저장한 백업본입니다. 이 스냅샷에서 새 서버를 만들 수 있으며, Lightsail과 EBS 모두 저장 용량 GB당 월 $0.05를 받습니다(서울 리전 기준).
> - **로드 밸런서:** 들어오는 요청을 여러 서버에 나눠 보내는 장치입니다. 서버 한 대가 멈춰도 나머지가 요청을 받게 해 줍니다.

**EC2는 "같은 서버의 부품을 바꾸는" 방식입니다.** 인스턴스를 중지한 뒤 인스턴스 유형을 바꾸면 같은 디스크와 설정으로 더 큰(또는 더 작은) 서버가 됩니다. 디스크(EBS)는 Elastic Volumes 기능으로 서버를 켠 채 용량을 늘리거나 종류를 바꿀 수 있습니다. 여러 대로 늘리고 줄이는 자동 확장, 다양한 로드 밸런서, 수많은 인스턴스 유형이 모두 EC2를 중심으로 짜여 있습니다.

| 하고 싶은 일 | Lightsail | EC2 |
| --- | --- | --- |
| 서버 사양 키우기 | 스냅샷 → 더 큰 플랜으로 새 인스턴스 (IP는 고정 IP를 옮겨 유지) | 중지 → 인스턴스 유형 변경 |
| 사양 줄이기 | 스냅샷으로는 작은 플랜 불가 | 호환되는 유형이면 같은 방법으로 가능 |
| 디스크 늘리기 | 블록 스토리지 추가(GB당 월 $0.10) | EBS 볼륨을 켠 채로 확장 |
| 다른 AWS 서비스 연결 | VPC 피어링을 켜서 기본 VPC의 자원(예: RDS)에 사설 IP로 접근 | 같은 VPC 안에서 바로 연결 |
| 다른 서비스로 옮기기 | 스냅샷을 EC2로 내보내기(AMI·EBS 스냅샷 생성) | 해당 없음 |

*표 기준: Lightsail 스냅샷 업사이즈·VPC 피어링·EC2 내보내기 문서, EC2 인스턴스 유형 변경 문서, EBS Elastic Volumes 문서. 2026-11-12 확인.*

> **용어 풀이**
>
> - **VPC 피어링:** 서로 다른 두 가상 네트워크를 연결해 사설 IP로 통신하게 하는 기능입니다. Lightsail 자원은 Lightsail이 관리하는 별도의 VPC 안에 있어서, 내 계정의 기본 VPC에 있는 자원과 사설 IP로 통신하려면 이 기능을 켜야 합니다.
> - **RDS(Amazon Relational Database Service):** AWS가 관리해 주는 데이터베이스 서비스입니다. MySQL, PostgreSQL 같은 데이터베이스를 설치·백업 걱정 없이 쓸 수 있습니다.

Lightsail로 시작했다가 막히는 일이 생겨도 길은 열려 있습니다. Lightsail 인스턴스 스냅샷을 EC2로 내보내면 같은 리전에 AMI와 EBS 스냅샷이 만들어지고, 이것으로 EC2 인스턴스를 띄울 수 있습니다. 내보내기 자체는 무료지만 이후 EC2 쪽 인스턴스와 디스크 요금은 따로 나옵니다. 로드 밸런서, 데이터베이스, 고정 IP, DNS 레코드는 함께 옮겨지지 않으니 다시 설정해야 하고, 다른 리전으로 옮기려면 먼저 Lightsail 안에서 스냅샷을 그 리전으로 복사해야 합니다.

## 그래서 어느 쪽으로 시작할까

- **개인 블로그, 포트폴리오, 소규모 API 서버를 정해진 예산으로 돌리고 싶다:** Lightsail $7~$12 플랜이 같은 메모리의 EC2 구성보다 월 $8~$12쯤 싸고, 트래픽 요금 걱정도 거의 없습니다. WordPress처럼 블루프린트가 있는 앱이라면 설치 시간도 줄어듭니다.
- **RDS 데이터베이스, 자동 확장처럼 다른 AWS 서비스를 본격적으로 엮을 계획이다:** EC2로 시작하는 편이 나중에 옮기는 수고가 없습니다. 비용이 걱정되면 t4g(Arm)를 고르고, 크레딧 설정을 Standard로 두거나 예산 알림을 설정하세요.
- **AWS 자격증 공부나 실무 준비가 목적이다:** EC2를 직접 만지며 VPC, 보안 그룹, EBS를 익히는 것이 배우는 내용과 맞습니다. 프리 티어 크레딧 안에서 t3.micro·t4g.micro로 연습할 수 있습니다.
- **CPU를 계속 많이 쓰는 작업(인코딩, 대량 빌드)을 돌린다:** 둘 다 저사양 버스터블 서버는 맞지 않습니다. 기준 성능 문서를 보고 더 큰 플랜이나 버스터블이 아닌 인스턴스 유형을 검토하세요.

AWS 밖의 선택지(오라클 클라우드 무료 서버, 국내 VPS 등)까지 넓혀 비교하고 싶다면 [첫 서비스 배포, 인프라는 어디서 시작할까](https://stackguide.route50.kr/posts/choosing-deployment-infrastructure-261111/)에서 다섯 업체를 같은 기준으로 정리해 두었습니다.

## 처음 만들 때 챙길 것

- **예산 알림을 먼저 설정하세요.** AWS Budgets로 예상 요금이 정해 둔 금액을 넘으면 메일을 받을 수 있습니다. EC2 Unlimited 모드의 초과 요금이나 트래픽 요금처럼 예상하지 못한 항목을 일찍 알아챌 수 있습니다.
- **연습이 끝난 자원은 지우세요.** EC2 인스턴스를 "중지"해도 EBS 디스크와 Elastic IP 요금은 계속 나가고, Lightsail 인스턴스는 중지해도 삭제할 때까지 요금이 붙습니다. 삭제한 인스턴스와 디스크는 되돌릴 수 없으니 필요한 데이터는 스냅샷으로 남긴 뒤 지우고, 스냅샷도 더 필요 없으면 지워야 저장 요금이 멈춥니다.
- **요금은 신청 전에 다시 확인하세요.** 이 글의 숫자는 2026년 11월 12일 서울 리전 기준입니다. AWS 요금과 프리 티어 조건은 바뀔 수 있으니 Lightsail 요금 페이지와 AWS 요금 계산기에서 원하는 리전과 사양으로 다시 확인하세요.

## 기준과 한계

- 요금은 2026-11-12에 Lightsail 요금 페이지·FAQ와 AWS Price List API(서울 리전 Lightsail, EC2, VPC, Data Transfer 가격 파일)에서 확인했습니다. 모두 리눅스, 온디맨드, 세금 별도 미국 달러 기준이며 EC2 할인 약정(Savings Plans, 예약 인스턴스)은 반영하지 않았습니다.
- 730시간 환산, 구성 합계, 트래픽·초과 크레딧 예시는 필자가 계산했습니다. 실제 청구액은 월별 시간 수, 세금, 크레딧 적용 여부에 따라 달라집니다.
- Lightsail 무료 체험은 FAQ(종료)와 시작 안내서($5 플랜 한 달 무료)의 설명이 엇갈립니다. 이 글은 FAQ를 따랐습니다.
- AWS 계정에서 Lightsail·EC2 인스턴스를 직접 만들어 성능을 측정하거나 설정 화면을 캡처하지는 않았습니다. 설정 절차는 공식 시작 안내서로 확인한 내용입니다.

## 출처

1. [Amazon Lightsail Pricing](https://aws.amazon.com/lightsail/pricing/) (확인일 2026-11-12)
2. [Amazon Lightsail FAQs](https://aws.amazon.com/lightsail/faq/) (확인일 2026-11-12)
3. [AWS Price List API - Amazon Lightsail, Asia Pacific (Seoul)](https://pricing.us-east-1.amazonaws.com/offers/v1.0/aws/AmazonLightsail/20260915151951/ap-northeast-2/index.csv) (확인일 2026-11-12)
4. [AWS Price List API - Amazon EC2, Asia Pacific (Seoul)](https://pricing.us-east-1.amazonaws.com/offers/v1.0/aws/AmazonEC2/20261110173852/ap-northeast-2/index.csv) (확인일 2026-11-12)
5. [AWS Price List API - Amazon VPC, Asia Pacific (Seoul)](https://pricing.us-east-1.amazonaws.com/offers/v1.0/aws/AmazonVPC/20261105181945/ap-northeast-2/index.json) (확인일 2026-11-12)
6. [AWS Price List API - AWS Data Transfer, Asia Pacific (Seoul)](https://pricing.us-east-1.amazonaws.com/offers/v1.0/aws/AWSDataTransfer/20261111191941/ap-northeast-2/index.json) (확인일 2026-11-12)
7. [Billing and account management - Amazon Lightsail](https://docs.aws.amazon.com/lightsail/latest/userguide/amazon-lightsail-frequently-asked-questions-faq-billing-and-account-management.html) (확인일 2026-11-12)
8. [Understand baseline CPU performance and burst capacity accrual for Lightsail instances](https://docs.aws.amazon.com/lightsail/latest/userguide/baseline-cpu-performance.html) (확인일 2026-11-12)
9. [Identify when your Lightsail instance bursts](https://docs.aws.amazon.com/lightsail/latest/userguide/identifying-instance-burst.html) (확인일 2026-11-12)
10. [Key concepts for burstable performance instances - Amazon EC2](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/burstable-credits-baseline-concepts.html) (확인일 2026-11-12)
11. [Unlimited mode concepts for burstable instances - Amazon EC2](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/burstable-performance-instances-unlimited-mode-concepts.html) (확인일 2026-11-12)
12. [Track your Free Tier usage for Amazon EC2](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-free-tier-usage.html) (확인일 2026-11-12)
13. [Choosing a plan - AWS Billing](https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/free-tier-plans.html) (확인일 2026-11-12)
14. [Getting started with virtual private servers on Lightsail](https://docs.aws.amazon.com/lightsail/latest/userguide/getting-started.html) (확인일 2026-11-12)
15. [Get started with Amazon EC2](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EC2_GetStarted.html) (확인일 2026-11-12)
16. [Upsize a Lightsail instance, storage, or database from snapshots](https://docs.aws.amazon.com/lightsail/latest/userguide/how-to-create-larger-instance-from-snapshot-using-console.html) (확인일 2026-11-12)
17. [Amazon EC2 instance type changes](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-resize.html) (확인일 2026-11-12)
18. [Modify an Amazon EBS volume using Elastic Volumes operations](https://docs.aws.amazon.com/ebs/latest/userguide/ebs-modify-volume.html) (확인일 2026-11-12)
19. [Export Lightsail snapshots to Amazon EC2](https://docs.aws.amazon.com/lightsail/latest/userguide/amazon-lightsail-exporting-snapshots-to-amazon-ec2.html) (확인일 2026-11-12)
20. [Connect Lightsail resources to AWS services using VPC peering](https://docs.aws.amazon.com/lightsail/latest/userguide/lightsail-how-to-set-up-vpc-peering-with-aws-resources.html) (확인일 2026-11-12)
