# 원격 데스크톱 도구 비교: Windows 원격 데스크톱, Chrome 원격 데스크톱, TeamViewer, AnyDesk, Splashtop, RustDesk

> 집 PC에 밖에서 접속하거나 다른 사람의 PC를 도와줄 때 쓰는 원격 데스크톱 도구 6가지를 연결 방식, 무료로 쓸 수 있는 조건, 유료 요금제, 설정 난이도로 비교합니다. 2026년 11월 11일 기준 공식 문서로 확인했습니다.

- 주소: https://stackguide.route50.kr/posts/remote-desktop-tools-comparison-261111/
- 분류: Network
- 작성일: 2026-11-11
- 정보 기준일: 2026-11-11
- 태그: 원격데스크톱, 원격지원, RDP, WindowsApp, ChromeRemoteDesktop, TeamViewer, AnyDesk, Splashtop, RustDesk, VPN, RDGateway, NLA, NAT, 홀펀칭, 중계서버, 무인접속, 셀프호스팅, 오픈소스, 2FA, RMM

출장 중에 집 PC에 있는 파일이 필요하거나, 멀리 사는 부모님이 "컴퓨터가 이상하다"며 전화를 걸어 오면 원격 데스크톱 도구를 찾게 됩니다. 검색하면 TeamViewer, AnyDesk, Chrome 원격 데스크톱 같은 이름이 한꺼번에 나오는데, 셋 다 "무료"라고 소개되지만 실제로는 무료로 쓸 수 있는 조건이 꽤 다릅니다. 회사 일에 쓰면 연결이 끊기는 도구도 있고, 상대 PC가 Windows Home이라 아예 접속이 안 되는 경우도 있습니다.

이 글은 자주 쓰는 원격 데스크톱 도구 6가지를 **연결 방식, 무료로 쓸 수 있는 조건, 유료 요금제, 설정 난이도**로 나눠 비교합니다. 다룰 도구는 Windows 원격 데스크톱, Chrome 원격 데스크톱, TeamViewer, AnyDesk, Splashtop, RustDesk입니다. 모든 조건과 가격은 2026년 11월 11일에 각 업체의 공식 요금 페이지, 도움말, 보안 문서에서 확인했습니다. 각 도구를 직접 설치해 측정한 결과는 아니며, 공식 문서로 확인하지 못한 내용은 본문에 그렇다고 적었습니다.

> **용어 풀이**
>
> - **원격 데스크톱:** 다른 곳에 있는 컴퓨터의 화면을 내 화면에 띄우고, 내 키보드·마우스로 그 컴퓨터를 조작하는 기능입니다.

## 먼저 정할 것: 내 PC에 들어가나, 남의 PC를 도와주나

원격 데스크톱 도구는 크게 두 가지 상황에서 씁니다.

- **원격 접속:** 집이나 사무실에 켜 둔 **내 PC**에 밖에서 들어가 작업합니다. 상대 PC 앞에 아무도 없으므로, 미리 설정해 둔 비밀번호만으로 들어갈 수 있어야 합니다.
- **원격 지원:** 가족이나 고객의 **남의 PC**에 잠깐 들어가 문제를 고쳐 줍니다. 상대가 화면 앞에 있으니, 상대가 알려 주는 일회용 코드로 접속하고 상대가 승인하는 방식이 편합니다.

> **용어 풀이**
>
> - **원격지원:** 다른 사람의 컴퓨터에 그 사람의 허락을 받아 접속해 문제를 해결해 주는 일입니다. 보통 접속할 때마다 상대가 코드나 승인으로 허락합니다.
> - **무인 접속(Unattended Access):** 원격 PC 앞에 사람이 없어도 미리 정한 비밀번호·PIN으로 접속할 수 있게 해 두는 설정입니다.

여기에 하나를 더 따져야 합니다. **돈을 받는 일과 관련이 있는지**입니다. TeamViewer, AnyDesk, Splashtop은 무료 사용을 개인·비상업 용도로 한정합니다. TeamViewer는 상업적 이용을 "직접 또는 간접적으로 대가를 받는 목적의 모든 사용"으로 정의하고, 동료·고객 지원은 물론 재택근무 중 업무 메일을 잠깐 확인하는 것도 예로 듭니다. 회사 PC에 접속하거나 프리랜서로 고객을 지원할 계획이라면 처음부터 유료 요금제나 다른 도구를 검토하는 편이 낫습니다.

## 연결 방식 세 가지: 상대 PC를 누가 찾아 주나

도구를 비교하기 전에 연결 구조부터 보면 장단점이 쉽게 이해됩니다. 집 PC는 보통 공유기 뒤에 있어서, 바깥에서는 그 PC의 주소를 바로 알 수도 없고 들어갈 수도 없습니다. 원격 데스크톱 도구는 이 문제를 세 가지 방식으로 풉니다.

1. **직접 연결:** Windows 원격 데스크톱은 내 PC가 원격 PC의 주소로 바로 접속합니다. 같은 집이나 사무실 네트워크 안에서는 간단하지만, 밖에서 쓰려면 VPN으로 그 네트워크에 먼저 들어가거나 공유기 포트를 열어야 합니다.
2. **업체 서버가 연결을 주선·중계:** TeamViewer, AnyDesk, Chrome 원격 데스크톱, Splashtop은 두 PC가 모두 업체 서버에 먼저 접속해 둡니다. 내가 상대의 ID나 코드를 입력하면 업체 서버가 둘을 이어 줍니다. 공유기 설정이 필요 없어 쉽지만, 연결이 업체 서비스에 의존합니다.
3. **중계 서버를 직접 운영:** RustDesk는 2번과 구조가 같지만, 그 서버를 내가 운영할 수 있습니다. 서버를 띄우고 관리하는 수고가 드는 대신, 연결 정보가 업체를 거치지 않습니다.

> **용어 풀이**
>
> - **NAT(Network Address Translation, 네트워크 주소 변환):** 공유기가 집 안 여러 기기의 사설 주소를 하나의 공인 주소로 바꿔 인터넷에 내보내는 기능입니다. 이 때문에 바깥에서 집 안의 특정 PC로 바로 들어오기 어렵습니다.
> - **VPN(Virtual Private Network, 가상 사설망):** 인터넷을 거쳐 집이나 회사 네트워크에 암호화된 통로로 들어가, 그 네트워크 안에 있는 것처럼 쓰게 해 주는 기술입니다.
> - **중계서버(릴레이 서버):** 두 PC가 직접 연결되지 못할 때 화면·입력 데이터를 대신 받아 넘겨주는 서버입니다.
> - **홀펀칭(Hole Punching):** 두 PC가 서버의 도움을 받아 서로의 공유기에 길을 열고, 중계 없이 직접 통신하게 만드는 기법입니다.

2번과 3번 방식도 화면 데이터가 늘 서버를 거치는 것은 아닙니다. TeamViewer 보안 문서는 서버를 통해 연결을 시작한 뒤 **약 70%의 경우** 두 PC가 직접 연결되고, 나머지는 TeamViewer의 중계 서버를 거친다고 설명합니다. RustDesk 문서도 ID 서버가 홀펀칭으로 직접 연결을 먼저 시도하고, 실패하면 중계 서버를 쓴다고 적고 있습니다. AnyDesk, Chrome 원격 데스크톱, Splashtop은 이 비율이나 순서를 공식 문서에서 찾지 못했습니다.

*(그림: 세 가지 연결 방식을 나란히 비교한 그림. 1번 Windows 원격 데스크톱은 내 PC가 VPN이나 RD 게이트웨이를 거쳐 원격 PC로 직접 접속하며 3389 포트를 인터넷에 직접 열지 말라는 경고가 붙어 있습니다. 2번 TeamViewer·AnyDesk·Chrome 원격 데스크톱·Splashtop은 두 PC가 모두 업체 서버에 접속하고, 3번 RustDesk는 두 PC가 내가 운영하는 서버에 접속합니다. 2번과 3번에는 도구에 따라 두 PC를 직접 잇는 점선 경로가 함께 그려져 있습니다.)*

*그림 1. 원격 데스크톱 도구의 세 가지 연결 방식 (자료: Microsoft Learn, TeamViewer Security Statement, RustDesk Self-host 문서, 2026-11-11 확인)*

## 한눈에 보는 비교

연결 방식은 그림 1로 대신하고, 아래 표에는 나머지 기준을 모았습니다. "유료 시작(개인)"은 개인이나 1인 사업자가 고를 만한 가장 낮은 요금제입니다. 사용자·기기 수 같은 구성은 도구별 설명에 표로 정리했습니다. TeamViewer와 AnyDesk는 공식 요금 페이지의 가격이 자동 조회로 읽히지 않아 확인하지 못했으므로, 요금제 이름과 구성만 적고 가격은 공식 페이지 링크로 대신합니다. 설정 난이도는 필자가 공식 설정 절차의 단계 수와 네트워크 설정 필요 여부를 기준으로 매긴 것입니다.

| 도구 | 무료 범위 | 유료 시작(개인) | 원격 PC 운영체제 | 설정 난이도 |
| --- | --- | --- | --- | --- |
| Windows 원격 데스크톱 | Windows에 포함 | 별도 요금 없음 | Windows Pro 이상 (Home 불가) | 집 안: 쉬움 · 밖에서: 어려움 |
| Chrome 원격 데스크톱 | 요금 안내 없음 | 유료 요금제 안내 없음 | Windows·macOS·Linux | 쉬움 |
| TeamViewer | 개인·비상업 | Remote Access, 연간 구독 ([가격](https://www.teamviewer.com/ko/pricing/overview/)) | Windows·macOS·Linux·모바일 | 쉬움 |
| AnyDesk | 세션 시간 제한 | Solo ([가격](https://anydesk.com/en/pricing)) | Windows·macOS·Linux(Xorg)·Android | 쉬움 |
| Splashtop | 집 안·비상업·2대 | Solo 연 72달러 | Windows·Mac | 보통 |
| RustDesk | 오픈 소스 | Server Pro 월 11.88달러꼴(연간 결제) | Windows·macOS·Linux·Android | 공개 서버: 쉬움 · 자체 서버: 어려움 |

*가격은 모두 미국 달러 기준이며 세금은 포함하지 않았습니다. 2026-11-11 각 사 공식 요금 페이지 기준.*

## 도구별로 자세히

### Windows 원격 데스크톱: 추가 비용 없지만 Home 에디션은 접속 대상이 될 수 없음

Windows에 기본으로 들어 있는 기능이라 따로 설치하거나 돈을 낼 필요가 없습니다. 다만 Microsoft 문서에 따르면 **원격 PC(호스트)가 될 수 있는 것은 Windows Pro, Enterprise, Education 에디션과 Windows Server**뿐입니다. Windows Home은 다른 PC에 접속하는 쪽으로만 쓸 수 있고, Home PC에 접속하는 것은 불가능합니다. 집에서 쓰는 노트북이 Home 에디션이라면 다른 도구를 골라야 합니다.

> **용어 풀이**
>
> - **RDP(Remote Desktop Protocol, 원격 데스크톱 프로토콜):** Microsoft가 만든 원격 데스크톱 통신 규약입니다. Windows 원격 데스크톱이 이 방식을 씁니다.

켜는 방법은 원격 PC에서 **설정 → 시스템 → 원격 데스크톱**으로 들어가 "원격 데스크톱 사용"을 켜는 것입니다. 관리자 권한이 필요하고, 관리자 그룹 계정과 따로 추가한 사용자만 접속할 수 있습니다. 같은 설정 화면의 NLA 옵션은 Microsoft가 대부분의 환경에서 켜 두라고 권합니다.

> **용어 풀이**
>
> - **NLA(Network Level Authentication, 네트워크 수준 인증):** 원격 세션 화면을 열기 전에 먼저 사용자 인증을 마치게 하는 기능입니다. 인증되지 않은 접속이 서버 자원을 쓰지 못하게 막습니다.

접속하는 쪽 프로그램은 운영체제마다 다릅니다. Windows에는 "원격 데스크톱 연결"(`mstsc.exe`)이 기본으로 들어 있고, macOS·iOS·Android에서는 Microsoft의 **Windows App**을 씁니다. Microsoft 문서에 따르면 Windows App으로 개인 PC에 접속할 때는 로그인이 필요 없습니다. 예전 "Microsoft 원격 데스크톱" 앱은 Windows App으로 바뀌는 중입니다. Windows용 Microsoft Store 앱은 2025년 5월 27일에 배포가 끝났고, Microsoft의 이전 버전 문서는 2025년 9월에 지원이 끝났다고 적고 있습니다.

> **용어 풀이**
>
> - **Windows App:** Microsoft가 원격 PC, Azure Virtual Desktop, Windows 365 등에 접속할 때 쓰도록 내놓은 통합 클라이언트 앱입니다. 기존 원격 데스크톱 앱을 대체합니다.

같은 네트워크 안에서는 이것으로 끝이지만, **밖에서 접속할 때부터 어려워집니다.** 원격 데스크톱은 기본적으로 3389번 포트에서 접속을 기다립니다. 공유기에서 이 포트를 인터넷으로 열면(포트 포워딩) 바로 쓸 수는 있지만, Microsoft는 이 방법이 "권장되지 않으며" 꼭 해야 한다면 강력한 비밀번호를 쓰고 VPN을 쓰는 편이 낫다고 안내합니다. 포트를 열면 전 세계 어디서든 그 PC에 로그인을 시도할 수 있기 때문입니다. 회사에서는 RD 게이트웨이를 앞에 두기도 하지만, 이것은 Windows Server의 역할이라 개인이 집에서 쓰기에는 무겁습니다.

> **용어 풀이**
>
> - **RD Gateway(Remote Desktop Gateway, 원격 데스크톱 게이트웨이):** Windows Server의 역할로, 인터넷에서 들어오는 원격 데스크톱 접속을 중간에서 받아 인증한 뒤 내부 PC로 넘겨줍니다. 내부 PC의 포트를 인터넷에 직접 열지 않아도 됩니다.

정리하면 Windows 원격 데스크톱은 **사무실이나 집 안에서, 또는 VPN을 이미 쓰고 있는 환경**에서 가장 깔끔한 선택입니다. 따로 설치할 것도 비용도 없지만, 밖에서 쓰려고 네트워크를 직접 만지기 시작하면 난이도와 위험이 함께 올라갑니다.

Mac에도 비슷한 기본 기능이 있습니다. **시스템 설정 → 일반 → 공유 → 화면 공유**를 켜면 다른 Mac에서 접속할 수 있고, 접속을 허용할 사용자를 지정할 수 있습니다. 이 기능도 바깥에서 쓰려면 같은 네트워크 문제를 풀어야 합니다.

### Chrome 원격 데스크톱: 가장 간단한 원격 지원

Google의 Chrome 원격 데스크톱은 Chrome 브라우저에서 `remotedesktop.google.com`에 들어가 작은 프로그램을 설치하면 바로 쓸 수 있습니다. 원격 PC로는 Windows, macOS, Linux를 지원하며, Linux는 64비트 Debian 패키지를 설치합니다. Android·iOS 앱은 다른 컴퓨터에 접속하는 쪽으로만 쓸 수 있습니다.

원격 지원 방식이 특히 쉽습니다. 도움을 받는 사람이 `remotedesktop.google.com/support`에서 코드를 만들어 알려 주면, 도와주는 사람이 그 코드를 입력합니다. 상대 화면에는 접속하려는 사람의 이메일 주소가 뜨고, 상대가 "공유"를 눌러야 연결됩니다. 코드는 한 번만 쓸 수 있고, 공유 중에는 30분마다 계속 공유할지 다시 묻습니다. 내 PC에 원격 접속하려면 원격 PC에서 원격 액세스를 설정하고 PIN을 정해 둔 뒤 `remotedesktop.google.com/access`에서 접속합니다. Google은 모든 원격 세션이 암호화된다고 밝힙니다.

가격은 Google 공식 페이지에서 요금이나 유료 요금제 안내를 찾을 수 없었고, 공식 설정 절차에도 결제 단계가 없습니다. 반면 일반 PC 사이의 파일 전송과 여러 모니터 지원은 공식 도움말에서 확인하지 못했습니다. 공식 문서에서 파일 전송을 설명하는 곳은 관리자가 관리하는 ChromeOS 키오스크 기기용 문서뿐이었습니다. 이 기능이 꼭 필요하다면 설치 후 직접 확인해 보는 것이 좋습니다.

### TeamViewer: 기능이 가장 넓지만 상업적 이용 판정이 까다로움

TeamViewer는 Windows, macOS, Linux, Android, iOS, Raspberry Pi 등 지원하는 운영체제가 가장 넓은 편이고, 유료 요금제에는 파일 전송, 원격 인쇄, 세션 녹화, 여러 모니터 지원, 무인 접속이 들어 있습니다. 원격 지원용 QuickSupport 모듈은 설치 없이 실행 파일만 받아 쓸 수 있습니다.

보안 문서에 따르면 연결은 RSA 4096 키 교환과 AES-256 암호화를 쓰고, 15.73 버전부터는 TLS 1.3과 AES-256-GCM을 씁니다. TeamViewer는 중계 서버도 세션 내용을 복호화할 수 없는 종단 간 암호화라고 설명합니다. 계정에는 OTP 앱 방식의 2단계 인증을 걸 수 있습니다.

> **용어 풀이**
>
> - **2FA(Two-Factor Authentication, 2단계 인증):** 비밀번호에 더해 휴대폰 앱의 일회용 코드처럼 두 번째 수단으로 한 번 더 확인하는 로그인 방식입니다.
> - **종단 간 암호화(E2EE, End-to-End Encryption):** 데이터를 보내는 기기에서 암호화해 받는 기기에서만 풀 수 있게 하는 방식입니다. 중간 서버는 내용을 볼 수 없습니다.

무료로 쓸 수 있는 범위는 **개인 용도**입니다. 가족·친구의 PC를 도와주거나 내 개인 기기에 접속하는 것까지입니다. 문제는 TeamViewer가 사용 패턴을 보고 상업적 이용을 의심하면 **연결 시간 제한(타임아웃)** 경고를 띄운다는 점입니다. 이 표시는 프로그램을 지웠다 다시 깔아도 사라지지 않고, 공식 양식(`teamviewer.com/reset`)으로 개인 사용 재검토를 요청해야 풀립니다. 재검토에는 약 72시간이 걸린다고 안내하지만 같은 문서에 영업일 7일이라는 목표도 함께 적혀 있습니다. 또 상업용으로 표시된 기기에는 내 기기를 개인용으로 되돌려도 접속할 수 없다고 적혀 있습니다.

유료 요금제는 **연간 구독**만 있고, 해지하지 않으면 12개월마다 자동 갱신됩니다(만료 28일 전까지 해지). 요금제별 구성은 다음과 같습니다.

| 요금제 | 사용자 | 관리 기기 | 동시 세션 |
| --- | --- | --- | --- |
| Remote Access | 1명 | 표기 없음(최대 3대에 접속) | 한 기기에서 최대 3개 |
| Business | 1명 | 200대 | 한 기기에서 최대 3개 |
| Premium | 15명 | 300대 | 한 기기에서 최대 10개 |
| Corporate | 30명 | 500대 | 채널 3개, 최대 15개 |

*자료: TeamViewer Licenses overview, 2026-11-11 확인. 가격은 [TeamViewer 공식 요금 페이지](https://www.teamviewer.com/ko/pricing/overview/)에서 확인하세요.*

내 PC 몇 대에만 접속하면 Remote Access, 고객 지원까지 하면 Business 이상이 맞습니다.

### AnyDesk: 가볍고 계정 없이 시작, 무료판은 세션 시간 제한

AnyDesk는 **계정 없이** 바로 쓸 수 있습니다. 상대의 AnyDesk 주소(숫자 ID)를 입력하면 상대 화면에 접속 요청이 뜨고, 상대가 수락하면 연결됩니다. 무인 접속용 비밀번호를 정해 두면 수락 없이 들어갈 수도 있습니다. 모든 세션은 TLS 1.3으로 암호화합니다.

> **용어 풀이**
>
> - **TLS(Transport Layer Security, 전송 계층 보안):** 인터넷에서 주고받는 데이터를 암호화하고 상대 서버가 진짜인지 확인하는 표준 보안 규약입니다. HTTPS도 TLS를 씁니다.

운영체제별로 제약이 있습니다. 공식 지원 목록을 보면 Linux는 원격 PC가 되려면 Xorg 환경이어야 하고 Wayland에서는 나가는 접속만 됩니다. Android 기기를 조작하려면 제어 플러그인이 필요합니다. iOS·iPadOS와 ChromeOS 기기는 화면을 볼 수는 있어도 원격으로 조작할 수 없으며, iOS·iPadOS 기기에 접속하려면 유료 라이선스가 있어야 합니다.

> **용어 풀이**
>
> - **Xorg·Wayland:** Linux에서 화면에 창을 그리고 키보드·마우스 입력을 전달하는 디스플레이 서버입니다. Xorg는 오래 쓰여 온 방식이고, Wayland는 이를 대체하는 새 방식입니다. 원격 도구가 화면을 가져오는 방법이 달라 지원 범위에 차이가 납니다.

무료판은 세션 시간 제한이 있습니다. AnyDesk 도움말은 끊기면 몇 번이든 다시 연결할 수 있다고 하면서, 시간 제한 없이 쓰려면 유료 라이선스로 바꾸라고 안내합니다. 제한 시간이 몇 분인지는 공식 문서에 적혀 있지 않습니다. 라이선스는 **접속을 시작하는 쪽**만 필요하고, 접속을 받기만 하는 PC에는 필요 없습니다.

| 요금제 | 사용자 | 접속을 시작할 수 있는 기기 | 관리 기기(무인 접속) | 동시 연결 |
| --- | --- | --- | --- | --- |
| Solo | 1명 | 3대 | 100대 | 1개(추가 불가) |
| Standard | 최대 20명 | – | 500대 | 1개(추가 구매 가능) |
| Advanced | 100명 | – | 1,000대 | 2개(추가 구매 가능) |
| Ultimate | 협의 | 협의 | 협의 | 협의(자체 서버 설치 가능) |

*자료: AnyDesk licenses 도움말, 2026-11-11 확인. 가격은 [AnyDesk 공식 요금 페이지](https://anydesk.com/en/pricing)에서 확인하세요.*

### Splashtop: 가격이 공개돼 있고 1인 요금제가 저렴함

Splashtop의 업무용 제품은 Splashtop Remote Access(예전 이름 Business Access)입니다. 가격이 공식 페이지에 그대로 공개돼 있고, 모두 연간 결제입니다.

| 요금제 | 가격(연간 결제) | 구성 |
| --- | --- | --- |
| Solo | 연 72달러 (월 6달러꼴) | 사용자 1명, 컴퓨터 2대 |
| Pro | 사용자당 연 99달러 (월 8.25달러꼴) | 라이선스당 컴퓨터 10대, 여러 모니터·세션 녹화 추가 |
| Performance | 사용자당 연 149달러 (월 13달러꼴) | 라이선스당 컴퓨터 10대 |
| Enterprise | 문의 | SSO, 자체 서버 설치 등 |

*자료: Splashtop Pricing, Splashtop Remote Access 제품 페이지, 2026-11-11 확인. 가격은 미국 달러.*

파일 전송, 원격 인쇄, 무인 접속은 Solo부터 들어 있고, 여러 모니터 보기와 세션 녹화는 Pro부터입니다. 접속하는 쪽은 Windows, Mac, iOS, Android, Chromebook, Linux를 지원합니다. 원격 PC 쪽은 제품 페이지 본문에 Windows·Mac·Linux로 적혀 있지만, 같은 페이지의 자주 묻는 질문에는 Windows와 Mac에만 설치한다고 되어 있어 Linux 원격 PC가 필요하면 미리 확인해야 합니다. 유료 제품은 7일 무료 체험을 제공합니다.

무료 제품인 Splashtop Personal은 조건이 좁습니다. **개인·비상업 용도로 집 네트워크 안에서만**, 최대 2대의 컴퓨터에 접속할 수 있습니다. 집 밖에서 쓰려면 Anywhere Access Pack(월 5.99달러 또는 연 23.99달러)을 더해야 하고, 재택근무를 포함한 업무 용도는 유료 제품을 써야 합니다.

보안 페이지에 따르면 연결은 TLS와 256비트 AES로 암호화하고, 2단계 인증을 켤 수 있으며 새 기기 인증이 기본으로 적용됩니다. 서비스는 AWS에서 운영되고, 자체 서버 설치는 Enterprise에서만 가능합니다. Splashtop은 계정을 만들고 원격 PC에 Streamer라는 프로그램을 설치해야 해서, 계정 없이 시작하는 AnyDesk보다 첫 설정 단계가 조금 많습니다.

### RustDesk: 오픈 소스, 서버까지 직접 운영 가능

RustDesk는 AGPL-3.0 라이선스의 오픈 소스 원격 데스크톱입니다. 2026년 11월 11일 기준 최신 안정 버전은 **1.5.1**(2026-11-01 배포)이고, 이 릴리스에는 Windows, macOS, Linux, Android용 설치 파일이 올라와 있습니다.

> **용어 풀이**
>
> - **오픈소스:** 프로그램의 소스 코드가 공개돼 있어 누구나 보고, 정해진 라이선스 조건 안에서 고치고 다시 배포할 수 있는 소프트웨어입니다.
> - **AGPL-3.0(GNU Affero General Public License 3.0):** 오픈 소스 라이선스의 하나로, 프로그램을 고쳐서 네트워크 서비스로 제공할 때도 고친 소스 코드를 공개하도록 요구합니다.

앱만 설치하면 RustDesk가 운영하는 공개 서버를 거쳐 바로 연결할 수 있습니다. 다만 RustDesk는 2026년 7월 블로그에서 공개 서버가 **시연과 테스트용이며 실제 업무나 민감한 작업용이 아니라고** 밝혔고, 원격 접속 사기와 봇넷 악용이 이어져 접속하는 쪽은 Google이나 GitHub 계정으로 로그인하게 바꿨습니다. RustDesk 저장소의 README에도 별도의 오용 경고 문구가 있습니다.

RustDesk의 핵심은 **서버를 직접 운영(셀프 호스팅)** 할 수 있다는 점입니다. 서버는 두 프로그램으로 이뤄집니다. `hbbs`는 ID 서버로, 두 PC를 홀펀칭으로 직접 연결하려 시도합니다. `hbbr`은 중계 서버로, 직접 연결이 실패하면 데이터를 대신 넘겨줍니다. 공식 문서 기준으로 최소 TCP 21115~21117번과 UDP 21116번 포트를 열어야 합니다. 공식 문서는 설치 스크립트, Docker Compose, deb 패키지로 두 서버를 함께 설치하는 방법을 안내합니다.

> **용어 풀이**
>
> - **셀프호스팅(Self-hosting):** 업체가 운영하는 서버를 빌려 쓰는 대신, 같은 프로그램을 내 서버에 직접 설치해 운영하는 방식입니다.

무료 오픈 소스 서버로도 연결은 되지만, 웹 관리 화면, 2단계 인증, 기기 관리, 접근 권한 관리, LDAP·OIDC 연동은 유료인 **Server Pro**에 들어 있습니다. RustDesk는 이 요금이 클라우드 서비스 이용료가 아니라 **자체 서버용 라이선스**라고 분명히 적고 있으며, 연간 결제만 받습니다.

| 요금제 | 가격(연간 결제, 월 환산) | 구성 |
| --- | --- | --- |
| Free | 0달러 | 오픈 소스 서버, 커뮤니티 지원 |
| Individual | 월 11.88달러 | 로그인 사용자 1명, 관리 기기 20대, 동시 연결 무제한 |
| Basic | 월 23.88달러 | 사용자 10명, 기기 100대, OIDC(SSO)·LDAP, 맞춤 클라이언트 생성 |
| Customized | 월 23.88달러부터 | 사용자 1명당 1.20달러, 기기 1대당 0.12달러 추가 |

*자료: RustDesk Pricing, 2026-11-11 확인. 가격은 미국 달러.*

> **용어 풀이**
>
> - **SSO(Single Sign-On, 통합 로그인):** 한 번 로그인한 회사 계정으로 여러 서비스에 따로 로그인하지 않고 들어가게 하는 방식입니다.
> - **OIDC(OpenID Connect):** SSO에 널리 쓰는 표준 인증 규약으로, Google·Microsoft 계정 같은 외부 계정으로 로그인하게 해 줍니다.
> - **LDAP(Lightweight Directory Access Protocol):** 회사의 사용자·그룹 정보를 저장한 디렉터리 서버(예: Active Directory)에 접근하는 표준 규약입니다.

서버를 직접 운영한다는 것은 서버의 보안 업데이트, 포트 관리, 장애 대응도 직접 맡는다는 뜻입니다. Linux 서버를 다뤄 본 적이 없다면 처음에는 다른 도구가 편합니다.

## 원격 접속 도구는 공격자에게도 편리하다

원격 데스크톱 도구는 내 컴퓨터를 남에게 통째로 맡기는 프로그램입니다. 그래서 공격자에게도 매력적입니다. 미국 CISA·NSA 등이 2023년 1월 낸 보안 권고(AA23-025A)는 공격자가 피싱 메일로 정상적인 원격 관리 소프트웨어를 내려받게 한 뒤 백도어처럼 쓰는 사례를 설명하며, 이런 정상 소프트웨어는 보통 백신 프로그램에 탐지되지 않는다고 지적합니다. RustDesk가 공개 서버에 로그인을 의무화한 것도 같은 이유입니다.

> **용어 풀이**
>
> - **RMM(Remote Monitoring and Management, 원격 감시·관리):** IT 관리자가 여러 대의 PC를 원격으로 감시하고 관리하는 소프트웨어입니다. 원격 데스크톱 도구도 넓게는 여기에 들어갑니다.

업체 자체도 공격 대상이 됩니다. TeamViewer는 2024년 6월 사내 IT 환경이 APT29(Midnight Blizzard)로 지목된 공격자에게 침입당했다고 공지했습니다. 직원 디렉터리 정보가 복사됐지만, 분리된 제품 환경과 연결 플랫폼, 고객 데이터에는 영향이 없었다고 밝혔습니다. AnyDesk도 도움말에서 코드 서명 인증서를 교체했다며 최신 버전으로 업데이트하라고 안내하고 있습니다.

어떤 도구를 쓰든 다음은 지키는 편이 안전합니다.

- **모르는 사람에게 접속 코드나 ID를 알려 주지 않습니다.** "은행·통신사·수사기관"이라며 원격 프로그램 설치를 요구하는 전화는 대표적인 사기 수법입니다.
- 계정을 쓰는 도구라면 **2단계 인증을 켭니다.**
- 무인 접속 비밀번호는 다른 곳에 쓰지 않는 긴 비밀번호로 정하고, 쓰지 않는 기기는 목록에서 지웁니다.
- 프로그램을 **최신 버전으로 유지합니다.** 각 업체가 보안 공지(보안 게시판)로 취약점과 수정 버전을 알립니다.
- Windows 원격 데스크톱의 3389 포트를 인터넷에 직접 열지 않습니다.

## 상황별로 고르기

- **같은 집·사무실 안에서 Windows Pro PC에 접속:** Windows 원격 데스크톱이 가장 간단하고 비용도 없습니다.
- **부모님이나 친구의 PC를 가끔 도와주기:** Chrome 원격 데스크톱의 일회용 코드 방식이 가장 설명하기 쉽습니다. 파일을 주고받거나 휴대폰을 지원해야 하면 개인 용도 안에서 TeamViewer를 쓸 수 있습니다.
- **밖에서 내 PC 1~2대에 업무용으로 접속:** 상업적 이용이므로 유료 요금제가 맞습니다. 가격이 공개된 것 중에서는 Splashtop Solo(연 72달러)가 저렴하고, 지원 운영체제와 기능 범위를 우선하면 TeamViewer Remote Access나 AnyDesk Solo를 공식 요금 페이지에서 비교해 보세요.
- **여러 고객을 지원하는 1인 사업자·작은 팀:** TeamViewer Business·Premium, AnyDesk Standard, Splashtop Pro처럼 관리 기기와 사용자 수가 넉넉한 요금제를 고릅니다. 동시 세션 수가 요금제마다 다르므로 함께 확인합니다.
- **연결 정보가 외부 업체를 거치지 않아야 하거나 서버 운영에 익숙함:** RustDesk 자체 서버가 맞습니다. 관리 기능이 필요하면 Server Pro를 검토합니다.

원격 데스크톱 도구의 가격과 무료 조건은 자주 바뀝니다. 특히 무료 사용 조건은 계약 조건에 가까워서, 업무에 쓰기 전에 각 업체의 최신 이용 약관과 요금 페이지를 한 번 더 확인하세요.

## 출처

1. [Enable Remote Desktop on your PC | Microsoft Learn](https://learn.microsoft.com/en-us/windows-server/remote/remote-desktop-services/remotepc/remote-desktop-allow-access) (확인일 2026-11-11)
2. [Remote Desktop - Allow access to your PC from outside your network | Microsoft Learn](https://learn.microsoft.com/en-us/windows-server/remote/remote-desktop-services/remotepc/remote-desktop-allow-outside-access) (확인일 2026-11-11)
3. [Change Remote Desktop listening port on Windows and Windows Server | Microsoft Learn](https://learn.microsoft.com/en-us/windows-server/remote/remote-desktop-services/remotepc/change-listening-port) (확인일 2026-11-11)
4. [What is Windows App? | Microsoft Learn](https://learn.microsoft.com/en-us/windows-app/overview) (확인일 2026-11-11)
5. [Windows App to replace Remote Desktop app for Windows - Windows IT Pro Blog](https://techcommunity.microsoft.com/blog/windows-itpro-blog/windows-app-to-replace-remote-desktop-app-for-windows/4390893) (확인일 2026-11-11)
6. [Turn Mac screen sharing on or off - Apple Support](https://support.apple.com/guide/mac-help/turn-screen-sharing-on-or-off-mh11848/mac) (확인일 2026-11-11)
7. [Access another computer with Chrome Remote Desktop - Google Chrome Help](https://support.google.com/chrome/answer/1649523?hl=en) (확인일 2026-11-11)
8. [TeamViewer Remote for personal use](https://www.teamviewer.com/en/solutions/roles/personal-use/) (확인일 2026-11-11)
9. [Commercial use suspected - TeamViewer Support](https://www.teamviewer.com/en-us/global/support/knowledge-base/teamviewer-classic/licensing/personal-use/commercial-use-suspected/) (확인일 2026-11-11)
10. [Licenses overview - TeamViewer Support](https://www.teamviewer.com/en/global/support/knowledge-base/teamviewer-classic/licensing/purchase/licenses-overview/) (확인일 2026-11-11)
11. [Security Statement - TeamViewer Support](https://www.teamviewer.com/en/global/support/knowledge-base/teamviewer-remote/security/security-statement/) (확인일 2026-11-11)
12. [TeamViewer Trust Center - Statement](https://www.teamviewer.com/en/resources/trust-center/statement/) (확인일 2026-11-11)
13. [AnyDesk licenses - AnyDesk Help Center](https://support.anydesk.com/docs/anydesk-licenses) (확인일 2026-11-11)
14. [What if I need longer session time on the free version? - AnyDesk Help Center](https://support.anydesk.com/docs/what-if-i-need-longer-sessions-time-on-the-free-version) (확인일 2026-11-11)
15. [Supported operating systems - AnyDesk Help Center](https://support.anydesk.com/docs/supported-operating-systems) (확인일 2026-11-11)
16. [Security and privacy - AnyDesk Help Center](https://support.anydesk.com/docs/security-and-privacy) (확인일 2026-11-11)
17. [Splashtop Pricing](https://www.splashtop.com/pricing) (확인일 2026-11-11)
18. [Splashtop Personal](https://www.splashtop.com/products/personal) (확인일 2026-11-11)
19. [Splashtop Remote Access](https://www.splashtop.com/products/remote-access) (확인일 2026-11-11)
20. [Splashtop Security](https://www.splashtop.com/security) (확인일 2026-11-11)
21. [rustdesk/rustdesk - GitHub](https://github.com/rustdesk/rustdesk) (확인일 2026-11-11)
22. [RustDesk 1.5.1 Release - GitHub](https://github.com/rustdesk/rustdesk/releases/tag/1.5.1) (확인일 2026-11-11)
23. [Self-host - RustDesk Documentation](https://rustdesk.com/docs/en/self-host/) (확인일 2026-11-11)
24. [RustDesk Pricing](https://rustdesk.com/pricing) (확인일 2026-11-11)
25. [RustDesk and Remote Access Scams: What We Are Doing - RustDesk Blog](https://rustdesk.com/blog/rustdesk-and-remote-access-scams/) (확인일 2026-11-11)
26. [Protecting Against Malicious Use of Remote Monitoring and Management Software (AA23-025A) - CISA](https://www.cisa.gov/news-events/cybersecurity-advisories/aa23-025a) (확인일 2026-11-11)
