# Next.js와 PostgreSQL로 만든 서비스, 무료로 배포하는 방법과 '진짜 무료'의 조건

> AI 도구로 만든 Next.js·PostgreSQL 서비스를 무료로 올릴 수 있는 Vercel, Netlify, Cloudflare Workers, Render와 Neon, Supabase, Prisma Postgres의 무료 등급을 2026년 11월 기준으로 비교하고, 한도를 넘거나 쉬면 무슨 일이 생기는지, 상업적 이용이 허용되는지까지 정리합니다.

- 주소: https://stackguide.route50.kr/posts/nextjs-postgresql-free-deployment-261111/
- 분류: Side Project
- 작성일: 2026-11-11
- 정보 기준일: 2026-11-11
- 태그: Next.js, PostgreSQL, 무료배포, Vercel, Netlify, CloudflareWorkers, Render, Neon, Supabase, PrismaPostgres, Railway, PaaS, 프리티어, 서버리스, 커넥션풀링, 콜드스타트, 스케일투제로, SSR, CDN, 아웃바운드트래픽

AI 코딩 도구에게 "게시판 만들어 줘"라고 부탁하면 요즘은 Next.js와 PostgreSQL 조합으로 코드가 나오는 경우가 많습니다. 로컬에서는 잘 돌아가는데, 친구에게 주소를 보내려면 어딘가에 올려야 합니다. 검색하면 "Vercel에 무료로 배포하세요"라는 답이 가장 먼저 나오지만, 데이터베이스는 어디에 두는지, 무료가 언제까지인지, 광고를 붙여도 되는지까지 알려 주는 글은 드뭅니다.

이 글은 Next.js 앱과 PostgreSQL 데이터베이스를 **돈을 내지 않고** 올릴 수 있는 서비스를 앱 쪽과 DB 쪽으로 나눠 비교합니다. 그리고 "정말 무료인가"를 세 가지 질문으로 따져 봅니다. 한도를 넘으면 어떻게 되는지, 한동안 접속이 없으면 어떻게 되는지, 수익을 내는 서비스에도 쓸 수 있는지입니다. 모든 한도와 조건은 2026년 11월 11일에 각 서비스의 공식 요금 페이지와 문서에서 확인했습니다. Next.js는 확인 시점 최신 기능 릴리스가 16.4(2026-10-06)입니다.

## 먼저 알아 둘 구조: 앱과 DB는 따로 올린다

Next.js 앱은 화면을 그려 주는 부분이고, PostgreSQL은 데이터를 저장하는 부분입니다. 무료 배포에서는 이 둘을 **각각 다른 서비스**에 올리는 것이 일반적입니다. 앱 호스팅 서비스는 코드를 받아 빌드하고 실행해 주고, 데이터베이스는 Neon이나 Supabase 같은 관리형 PostgreSQL 서비스에서 따로 만든 뒤 접속 주소만 앱에 알려 줍니다.

Next.js 공식 배포 문서는 배포 방식을 네 가지로 나눕니다. Node.js 서버로 실행하거나 Docker 컨테이너로 실행하면 모든 기능이 동작하고, 정적 내보내기(static export)는 서버가 필요한 기능을 쓸 수 없으며, 플랫폼별 어댑터는 지원 범위가 플랫폼마다 다릅니다. 로그인, 글쓰기처럼 DB를 읽고 쓰는 서비스라면 서버가 필요하므로 GitHub Pages 같은 정적 호스팅만으로는 부족합니다.

> **용어 풀이**
>
> - **Next.js:** React로 웹 화면을 만들면서, 페이지를 서버에서 미리 그리거나 API를 함께 만들 수 있게 해 주는 프레임워크입니다.
> - **PostgreSQL:** 오래 쓰여 온 오픈 소스 관계형 데이터베이스입니다. 표 형태로 데이터를 저장하고 SQL로 읽고 씁니다.
> - **SSR(Server-Side Rendering, 서버 측 렌더링):** 사용자가 페이지를 요청할 때마다 서버가 HTML을 만들어 보내는 방식입니다. DB에서 최신 글 목록을 읽어 보여 주는 페이지가 대표적입니다.
> - **Docker 컨테이너:** 프로그램과 실행에 필요한 라이브러리·설정을 한 묶음으로 포장해 어느 서버에서나 같은 방식으로 실행하게 하는 기술입니다.
> - **어댑터:** Next.js 빌드 결과를 특정 플랫폼의 실행 방식에 맞게 바꿔 주는 도구입니다. Next.js 16.2부터 공식 Adapter API가 안정 버전으로 제공됩니다.

앱 호스팅은 다시 두 부류로 나뉩니다. Vercel, Netlify, Cloudflare Workers는 요청이 올 때만 코드를 잠깐 실행하는 **서버리스** 방식이고, Render·Railway·Koyeb는 Node.js 서버를 컨테이너로 계속 띄우는 방식입니다. 이 차이가 뒤에서 볼 "잠드는 문제"와 "DB 연결 문제"로 이어집니다.

> **용어 풀이**
>
> - **서버리스(Serverless):** 서버를 직접 띄워 두지 않고, 요청이 들어올 때마다 플랫폼이 함수를 실행해 응답하는 방식입니다. 서버가 없다는 뜻이 아니라 서버 관리를 플랫폼이 맡는다는 뜻입니다.
> - **PaaS(Platform as a Service, 서비스형 플랫폼):** 서버와 실행 환경을 업체가 관리하고, 사용자는 코드와 설정만 올리는 클라우드 서비스 방식입니다.

*(그림: 브라우저의 요청이 앱 호스팅(Vercel, Netlify, Cloudflare Workers, Render 등)에서 실행되는 Next.js로 가고, Next.js가 환경 변수에 넣은 풀링 접속 주소를 통해 관리형 PostgreSQL(Neon, Supabase, Prisma Postgres)에 연결되는 구조)*

*그림 1. 무료 배포에서 흔히 쓰는 구성. 앱과 DB를 서로 다른 서비스에 두고 접속 주소로 연결합니다.*

## 앱을 올릴 곳: 네 곳의 무료 등급

### Vercel Hobby: 가장 쉽지만 개인·비상업용

Vercel은 Next.js를 만든 회사의 호스팅입니다. GitHub 저장소를 연결하면 별다른 설정 없이 빌드와 배포가 끝나고, 공식 배포 문서에서도 검증된 어댑터 목록에 올라 있습니다. 처음 해 보는 사람이 가장 덜 막히는 곳입니다.

무료인 Hobby 플랜에 매달 포함된 양은 다음과 같습니다.

| 항목 | Hobby 월 포함량 |
| --- | --- |
| Fast Data Transfer (방문자에게 보내는 데이터) | 100GB |
| CDN 요청 | 100만 건 |
| 함수 호출 | 100만 건 |
| Active CPU (함수가 실제로 CPU를 쓴 시간) | 4시간 |
| Provisioned Memory | 360GB-시간 |
| 이미지 최적화 변환 | 5,000건 |
| 함수 최대 실행 시간 | 300초 |

*표 기준: Vercel 요금 페이지와 Hobby 플랜 문서, 2026-11-11 확인.*

> **용어 풀이**
>
> - **CDN(Content Delivery Network, 콘텐츠 전송 네트워크):** 세계 여러 곳에 둔 서버에 파일을 복사해 두고, 방문자와 가까운 서버에서 내려보내는 시스템입니다. 이미지나 정적 파일이 빨리 열립니다.

개인 프로젝트 하나를 운영하기에는 넉넉한 편이지만 두 가지는 꼭 알아야 합니다.

첫째, **한도를 넘으면 돈을 내고 더 쓸 수 없습니다.** Hobby 문서는 대부분의 경우 한도를 넘긴 기능은 30일이 지나야 다시 쓸 수 있다고 안내합니다. 갑자기 방문자가 몰리면 서비스가 멈출 수 있다는 뜻입니다.

둘째, **Hobby는 개인적·비상업적 용도로만 쓸 수 있습니다.** Vercel의 공정 사용 지침은 상업적 이용의 예로 결제를 받는 것, 상품이나 서비스 판매를 광고하는 것, 사이트 제작·호스팅 대가를 받는 것, 제휴 링크가 주목적인 것, 그리고 **Google AdSense 같은 광고를 넣는 것**을 들고 있습니다. 후원을 받는 것은 상업적 이용이 아니라고 적혀 있습니다. 나중에 광고를 붙이거나 유료 기능을 넣을 계획이라면 처음부터 Pro 플랜(월 $20)으로 갈지, 다른 곳을 쓸지 정해 두는 편이 낫습니다.

### Netlify Free: 매달 300크레딧, 다 쓰면 사이트가 멈춘다

Netlify는 오픈 소스 OpenNext 어댑터로 Next.js 13.5 이상을 지원합니다. App Router, SSR, ISR, 미들웨어, 이미지 최적화까지 별도 설정 없이 동작한다고 문서에 적혀 있습니다.

무료 플랜은 매달 300크레딧을 주고, 쓰는 만큼 크레딧이 줄어드는 방식입니다. 확인 시점의 차감 기준은 다음과 같습니다.

| 사용 항목 | 차감 크레딧 |
| --- | --- |
| 운영(production) 배포 1회 | 15 |
| 함수 실행(컴퓨트) 1GB-시간 | 10 |
| 데이터 전송 1GB | 20 |
| 웹 요청 1만 건 | 2 |
| 미리보기(Deploy Preview)·브랜치 배포, 실패한 배포 | 0 |

*표 기준: Netlify 요금 페이지와 "How credits work" 문서, 2026-11-11 확인.*

계산해 보면 감이 옵니다. 한 달에 운영 배포를 10번 하고(150), 방문자에게 5GB를 보내고(100), 요청이 10만 건이면(20) 270크레딧입니다. 배포만 20번 해도 300크레딧이 다 차므로, AI 도구로 코드를 조금씩 고치며 `main`에 자주 올리는 습관이라면 금방 바닥납니다(필자 계산). 미리보기 배포는 차감되지 않으니 수정은 브랜치에서 확인하고 운영 배포는 모아서 하는 편이 좋습니다.

크레딧을 다 쓰면 **팀의 모든 사이트가 일시 중지**되고 방문자에게는 `Site not available` 페이지가 보입니다. 무료 플랜은 크레딧을 추가로 살 수 없어서 다음 결제 주기에 크레딧이 다시 채워질 때까지 기다리거나 유료로 올려야 합니다.

Netlify에는 자체 PostgreSQL인 Netlify Database도 있습니다. 무료 플랜에서 데이터베이스 3개, 저장 공간 5GB, 5분 동안 쓰지 않으면 잠드는 조건이며, DB의 컴퓨트와 전송량도 같은 크레딧에서 차감됩니다. 문서에는 저장 공간이 2026년 7월 1일까지 무료이고 그 뒤 요금은 미리 공지한다고만 적혀 있어, 확인 시점의 저장 공간 요금은 알 수 없었습니다. 이번에 읽은 Netlify 문서에서는 Vercel처럼 무료 플랜의 상업적 이용을 막는 문구를 찾지 못했지만, 약관 전체를 확인한 것은 아니므로 수익을 낼 계획이라면 이용 약관을 직접 확인하세요.

> **용어 풀이**
>
> - **ISR(Incremental Static Regeneration, 증분 정적 재생성):** 미리 만들어 둔 정적 페이지를 정해 둔 주기나 요청에 따라 서버에서 다시 만들어 갈아 끼우는 Next.js 기능입니다.
> - **GB-시간:** 메모리 사용량(GB)에 실행 시간(시간)을 곱한 단위입니다. 1GB 메모리로 1시간 실행하면 1GB-시간입니다.

### Cloudflare Workers Free: 하루 10만 요청, 대신 설정이 한 단계 더

Cloudflare Workers는 전 세계 Cloudflare 서버에서 코드를 실행합니다. 무료 플랜의 핵심 한도는 **하루 10만 요청**(UTC 자정에 초기화)과 **요청 1건당 CPU 시간 10ms**입니다. 정적 파일 요청은 무료이고 개수 제한이 없습니다. 하루 한도를 넘으면 Cloudflare가 Error 1027을 돌려주고, CPU 한도를 넘으면 Error 1102가 납니다. 다음 날 자정(UTC, 한국 시간 오전 9시)이 되면 다시 열리므로 Netlify처럼 한 달 내내 멈추지는 않습니다.

Next.js를 Workers에 올리는 방법은 확인 시점에 두 가지입니다. Cloudflare 공식 가이드는 베타 단계인 vinext를 기본 방법으로 안내하고, 기존 방식인 OpenNext 어댑터(`@opennextjs/cloudflare`)는 vinext로 아직 옮길 수 없는 앱을 위한 방법으로 소개합니다. 둘 다 Vercel이나 Netlify보다 설정 파일과 명령이 조금 더 필요하고, OpenNext 문서는 Windows에서의 완전한 지원은 보장하지 않으며 WSL이나 Linux·macOS 환경을 권장한다고 적고 있습니다.

> **용어 풀이**
>
> - **WSL(Windows Subsystem for Linux):** Windows 안에서 Linux 환경을 실행하게 해 주는 Microsoft의 기능입니다.

무료 플랜에서 주의할 점이 두 가지 더 있습니다. 하나는 CPU 10ms입니다. DB를 기다리는 시간은 CPU 시간에 들어가지 않지만, 페이지를 서버에서 그리는 계산이 무거우면 이 한도에 걸릴 수 있습니다. 다른 하나는 Worker 크기입니다. OpenNext 문서는 무료 플랜의 Worker 크기를 압축 기준 3MiB로 적고 있는데, Cloudflare 한도 문서는 압축 크기 제한이 없고 압축 전 64MiB만 따진다고 적고 있어 두 문서가 엇갈립니다. 배포가 크기 문제로 실패하면 두 문서를 다시 확인하세요.

DB에 연결할 때는 Cloudflare의 Hyperdrive를 쓸 수 있습니다. 외부 PostgreSQL 앞에서 연결을 모아 주고 결과를 캐시하는 서비스로, 무료 플랜에서도 하루 10만 쿼리까지 쓸 수 있습니다.

### Render Free: 일반 Node.js 서버처럼 쓰지만 15분이면 잠든다

Render는 Next.js를 `next start`로 실행하는 일반 Node.js 서버나 Docker 컨테이너로 올립니다. 서버리스 플랫폼의 제약 없이 Next.js의 모든 기능을 쓸 수 있고, 나중에 다른 서버로 옮기기도 쉽습니다.

대신 무료 웹 서비스는 **15분 동안 요청이 없으면 잠들고**(spin down), 다음 요청이 오면 다시 깨어나는 데 약 1분이 걸린다고 문서에 적혀 있습니다. 작업 공간마다 한 달에 750 인스턴스 시간이 주어지고, 다 쓰면 그달 남은 기간 동안 무료 서비스가 모두 멈춥니다. 파일 시스템은 재배포하거나 잠들 때 지워지므로, 사용자가 올린 이미지를 서버 디스크에 저장하는 코드는 무료 서버에서 동작하지 않습니다. Render 문서 스스로 "무료 인스턴스를 운영 서비스에 쓰지 말라"고 적어 두었습니다.

> **용어 풀이**
>
> - **콜드 스타트(Cold Start):** 쉬고 있던 서버나 함수를 깨워 첫 요청을 처리할 때 생기는 지연입니다. 깨어난 뒤의 요청은 평소 속도로 처리됩니다.
> - **스케일 투 제로(Scale to Zero):** 요청이 없을 때 실행 중인 인스턴스를 0개로 줄여 자원을 쓰지 않는 동작입니다. 비용은 아끼지만 다음 요청에 콜드 스타트가 생깁니다.

Render에도 무료 PostgreSQL이 있지만 **만든 지 30일이 지나면 만료**되고, 14일의 유예 기간 안에 유료로 올리지 않으면 삭제됩니다. 용량은 1GB, 작업 공간당 1개, 백업은 없습니다. 연습용으로만 쓰고, 오래 둘 데이터라면 아래의 Neon이나 Supabase를 함께 쓰는 편이 안전합니다.

### 그 밖의 선택지

- **Railway:** 30일 동안 쓰는 $5 체험 크레딧(카드 불필요) 뒤에는 월 $1 사용 크레딧을 주는 Free 플랜으로 바뀝니다. Free 플랜은 서비스당 1 vCPU·메모리 0.5GB, 프로젝트 1개이고 **사용자 도메인을 연결할 수 없습니다.** 체험용으로 보는 것이 맞습니다.
- **Koyeb:** 조직당 무료 인스턴스 1개(메모리 512MB, 0.1 vCPU)를 주지만 지역이 프랑크푸르트와 워싱턴 D.C.뿐이라 한국 사용자에게는 응답이 느릴 수 있습니다. 1시간 동안 요청이 없으면 0으로 줄어듭니다.
- **Fly.io:** 새 조직에는 무료 등급이나 월 무료 사용량이 없고, 카드 없이 쓸 수 있는 체험은 $1 크레딧·7일입니다. 무료 배포처로는 맞지 않습니다.
- **가상 서버를 직접 운영:** 서버를 직접 다뤄 보고 싶다면 오라클 클라우드 Always Free 같은 무료 가상 서버에 Node.js와 PostgreSQL을 함께 설치하는 방법도 있습니다. 조건과 주의점은 [첫 서비스 배포, 인프라는 어디서 시작할까](https://stackguide.route50.kr/posts/choosing-deployment-infrastructure-261111/)에서 다뤘습니다.

> **용어 풀이**
>
> - **vCPU(virtual CPU, 가상 CPU):** 가상 서버나 컨테이너에 배정되는 CPU 단위입니다. 0.1 vCPU는 CPU 하나의 10%에 해당하는 몫을 받는다는 뜻입니다.

## 데이터베이스를 둘 곳: 세 곳의 무료 PostgreSQL

| 항목 | Neon Free | Supabase Free | Prisma Postgres Free |
| --- | --- | --- | --- |
| 저장 용량 | 프로젝트당 1GB (전체 20GB) | 프로젝트당 500MB | 2GB |
| 개수 | 프로젝트 100개 | 활성 프로젝트 2개 | 데이터베이스 50개 |
| 사용량 한도 | 프로젝트당 월 100 CU-시간 | 공유 CPU, 메모리 0.5GB | 월 20만 operations |
| 데이터 전송 | 프로젝트당 월 5GB | 월 5GB (캐시 제외) | 제한 없음 |
| 쉴 때 | 5분 뒤 컴퓨트가 0으로 (끌 수 없음) | 1주 동안 활동이 없으면 프로젝트 일시 중지 | 문서에 명시 없음 |
| 자동 백업 | 예약 스냅숏 없음 (6시간 복원 이력) | 없음 | 없음 |
| 카드 등록 | 필요 없음 | 확인한 문서에 명시 없음 | 필요 없음 |

*표 기준: Neon 플랜 문서, Supabase·Prisma 요금 페이지, 2026-11-11 확인.*

> **용어 풀이**
>
> - **CU(Compute Unit)-시간:** Neon이 컴퓨트 사용량을 세는 단위입니다. 컴퓨트 크기(CU)에 켜져 있던 시간을 곱합니다. 쓰지 않을 때 0으로 줄어든 시간은 세지 않습니다.
> - **아웃바운드 트래픽:** 서버나 DB에서 바깥으로 나가는 데이터 양입니다. DB의 경우 앱 서버가 쿼리 결과를 받아 가는 양이 여기에 들어갑니다.

**Neon**은 Vercel이 예전에 제공하던 Vercel Postgres를 넘겨받은 곳이기도 합니다. Vercel 문서는 Vercel Postgres를 더 이상 제공하지 않으며 기존 DB는 2024년 12월에 Neon으로 옮겼다고 안내합니다. AI 도구가 `@vercel/postgres`나 "Vercel 대시보드에서 Postgres 만들기"를 알려 준다면 예전 정보이니, 지금은 Vercel Marketplace에서 Neon, Supabase, Prisma Postgres 같은 통합을 설치하면 접속 정보가 환경 변수로 자동 등록됩니다. Neon 무료 플랜은 CU-시간이나 전송량 5GB를 다 쓰면 **다음 결제 주기까지 컴퓨트가 멈추고**, 저장 용량을 넘으면 쓰기가 막힙니다. 데이터가 지워지지는 않습니다.

**Supabase**는 PostgreSQL에 로그인(Auth), 파일 저장소, 실시간 기능을 묶어 줍니다. 무료 플랜에서도 월 5만 MAU까지 로그인 기능을 쓸 수 있어, 회원 기능이 필요한 서비스라면 따로 인증 서비스를 붙이지 않아도 됩니다. 다만 **1주 동안 활동이 없으면 프로젝트가 일시 중지**되고, 대시보드에서 다시 켜야 합니다. 업그레이드 문서 본문은 일시 중지 뒤 1년 안에 복원할 수 있다고 적지만, 같은 문서의 화면 예시에는 "90 Days Remaining"이라고 나와 있어 기간이 엇갈립니다. 오래 쉬게 될 프로젝트라면 미리 백업을 받아 두는 편이 안전합니다.

> **용어 풀이**
>
> - **MAU(Monthly Active Users, 월간 활성 사용자):** 한 달 동안 한 번 이상 로그인하거나 활동한 사용자 수입니다.

**Prisma Postgres**는 ORM인 Prisma를 만든 회사의 PostgreSQL입니다. 무료 플랜은 월 20만 operations, 저장 공간 2GB이고 지출 한도가 $0으로 고정되어 카드 없이 씁니다. 한도에 도달했을 때 DB가 어떻게 되는지는 요금 페이지에 따로 적혀 있지 않았습니다.

> **용어 풀이**
>
> - **ORM(Object-Relational Mapping):** SQL을 직접 쓰지 않고 코드의 객체로 DB를 읽고 쓰게 해 주는 도구입니다. Next.js 프로젝트에서는 Prisma와 Drizzle을 많이 씁니다.

## 서버리스에서 DB를 연결할 때 꼭 챙길 것: 커넥션 풀링

서버리스 함수는 요청마다 새로 실행될 수 있어서, 함수마다 DB에 새 연결을 열면 금방 DB의 연결 수 한도를 채웁니다. Neon 문서는 이것을 연결 한도를 넘기는 흔한 원인으로 꼽고, 서버리스 환경에서는 **풀링 연결**을 쓰라고 권합니다. Supabase 문서도 서버리스 함수처럼 짧은 연결을 많이 여는 환경에는 공유 풀러의 트랜잭션 모드를 권합니다.

> **용어 풀이**
>
> - **커넥션 풀링(Connection Pooling):** DB 연결을 미리 몇 개 열어 두고 여러 요청이 돌려 쓰게 하는 방식입니다. 연결을 새로 여는 비용과 DB의 연결 수 한도 문제를 줄여 줍니다.
> - **트랜잭션 모드:** 풀러가 연결을 트랜잭션 하나가 끝날 때마다 다른 요청에 넘겨주는 방식입니다. 연결을 효율적으로 쓰지만 연결 단위로 유지되는 일부 기능은 쓸 수 없습니다.

방법은 간단합니다. DB 콘솔의 접속 정보 화면에서 풀링용 주소를 복사해 앱의 환경 변수에 넣으면 됩니다.

- **Neon:** 콘솔의 Connect 화면에서 Connection pooling을 켜면 호스트 이름의 엔드포인트 ID 뒤에 `-pooler`가 붙은 주소가 나옵니다.
- **Supabase:** 접속 화면에서 공유 풀러의 트랜잭션 모드(포트 6543) 주소를 고릅니다. 이 모드는 prepared statement를 지원하지 않으므로 라이브러리에서 꺼야 합니다. 문서는 Prisma라면 `pgbouncer=true`, Postgres.js·Drizzle이라면 `prepare: false`를 예로 듭니다.

> **용어 풀이**
>
> - **Prepared Statement:** 자주 쓰는 SQL을 DB에 미리 등록해 두고 값만 바꿔 실행하는 방식입니다. 등록 정보가 연결 하나에 묶여 있어, 연결을 계속 바꿔 쓰는 트랜잭션 모드 풀러와는 맞지 않습니다.

로컬에서는 프로젝트 루트의 `.env.local` 파일에 다음처럼 넣고, 배포한 곳에서는 호스팅 대시보드의 환경 변수 설정 화면에 같은 이름으로 넣습니다. 아래는 Neon 문서의 주소 형식을 따른 예시이며 실제 값은 콘솔에서 복사합니다.

```ini
# .env.local (로컬 개발용, Git에 커밋하지 않음)
DATABASE_URL="postgresql://<USER>:<PASSWORD>@<ENDPOINT_ID>-pooler.<REGION>.aws.neon.tech/<DB_NAME>?sslmode=require"
```

> **용어 풀이**
>
> - **환경 변수:** 코드 밖에서 프로그램에 넘겨주는 설정값입니다. DB 비밀번호처럼 코드에 적으면 안 되는 값을 배포 환경마다 따로 넣을 때 씁니다.

AI 도구로 코드를 만들 때 특히 조심할 부분이 이 비밀번호입니다. AI가 접속 주소를 코드 안에 바로 적어 주거나, `.env` 파일이 `.gitignore`에 빠져 있는 경우가 있습니다. 공개 저장소에 접속 주소가 올라가면 누구나 DB에 접속할 수 있으니, 커밋 전에 `git status`로 `.env` 파일이 목록에 없는지 확인하세요. 이미 올라갔다면 파일을 지우는 것만으로는 부족하고, DB 콘솔에서 비밀번호를 바로 바꿔야 합니다.

## 정말 무료일까: 한도를 넘거나 쉬면 생기는 일

"무료"라고 해도 멈추는 방식은 서비스마다 다릅니다. 하루 단위로 다시 열리는 곳이 있고, 한 달을 기다려야 하는 곳이 있고, 데이터가 지워지는 곳도 있습니다.

*(그림: 서비스별로 무료 한도를 넘거나 쉴 때 일어나는 일을 세 단계로 나눈 그림. 느려짐(Render 15분 뒤 잠듦, Neon 5분 뒤 0으로), 멈춤(Cloudflare 하루 한도 초과 시 1027 오류, Netlify 크레딧 소진 시 모든 사이트 중지, Vercel 한도 초과 시 최대 30일 대기, Supabase 1주 비활동 시 일시 중지, Neon 월 한도 초과 시 컴퓨트 정지), 삭제(Render 무료 Postgres는 30일 만료 뒤 14일 유예 후 삭제))*

*그림 2. 무료 한도를 넘거나 쉴 때의 동작 (자료: 각 서비스 공식 문서, 2026-11-11 확인)*

정리하면 이렇게 볼 수 있습니다.

- **카드를 등록하지 않았거나 무료 플랜이면 갑자기 요금이 나가지는 않습니다.** 위에서 다룬 곳들은 무료 한도를 넘으면 대부분 요금을 받는 대신 서비스를 멈춥니다. Vercel Hobby와 Netlify Free는 추가 사용량을 살 수 없고, Prisma Postgres Free는 지출 한도가 $0으로 고정되어 있습니다. 반대로 카드를 등록하거나 유료 플랜으로 바꾼 뒤에는 초과 사용량에 요금이 붙을 수 있으니, 그때는 각 서비스의 지출 한도나 예산 알림을 먼저 설정하세요.
- **"무료"의 대가는 속도와 가용성입니다.** 잠들어 있다가 깨어나는 콜드 스타트, 한도 초과 시 멈춤은 운영 중인 서비스에서는 곧 장애입니다. 친구나 포트폴리오 심사자에게 보여 줄 서비스라면 보여 주기 직전에 한 번 접속해 깨워 두는 정도로 충분하지만, 실제 사용자가 있는 서비스라면 유료 전환 시점을 미리 정해 두는 것이 좋습니다.
- **수익을 내려면 약관부터 확인하세요.** Vercel Hobby는 광고(AdSense 포함)나 결제가 들어가는 순간 상업적 이용이 되어 Pro 플랜이 필요합니다. 다른 서비스도 무료 플랜의 이용 조건을 직접 확인하세요.
- **백업은 스스로 챙겨야 합니다.** 세 DB의 무료 플랜 모두 자동 백업이 없거나 제한적입니다. 중요한 데이터가 생기기 시작하면 `pg_dump`로 주기적으로 내려받아 두세요.

> **용어 풀이**
>
> - **pg_dump:** PostgreSQL에 들어 있는 데이터와 구조를 파일로 내보내는 공식 백업 도구입니다. 이 파일로 다른 PostgreSQL에 데이터를 그대로 옮길 수도 있습니다.

## 상황별 추천 조합

- **포트폴리오나 연습용, 수익 계획 없음:** Vercel Hobby + Neon Free. Vercel Marketplace에서 Neon을 연결하면 환경 변수까지 자동으로 들어가 가장 손이 덜 갑니다.
- **광고나 결제를 붙일 수도 있음:** Vercel Hobby는 그 시점에 쓸 수 없으므로, 처음부터 Netlify Free나 Cloudflare Workers Free에 Neon·Supabase를 붙이거나, 수익이 생기면 바로 유료 플랜으로 옮길 계획을 세웁니다. 어느 쪽이든 약관을 직접 확인하세요.
- **회원 가입·로그인이 핵심인 서비스:** 어느 호스팅이든 Supabase Free를 DB로 쓰면 인증 기능을 함께 얻습니다. 1주 일시 중지 규칙만 기억해 두면 됩니다.
- **서버리스가 낯설고 일반 Node.js 서버로 배우고 싶음:** Render Free에 앱을 올리고 DB는 Neon이나 Supabase를 씁니다. Render 무료 Postgres는 30일 뒤 만료되므로 오래 쓸 데이터에는 맞지 않습니다.
- **방문자가 하루 단위로 몰렸다 빠지는 서비스:** Cloudflare Workers Free는 하루 한도가 매일 초기화되어 한 번 넘쳐도 다음 날 다시 열립니다. 설정이 한 단계 더 필요하다는 점은 감안해야 합니다.

어떤 조합이든 앱과 DB를 접속 주소 하나로만 연결해 두면, 나중에 한쪽만 다른 서비스로 옮기기 쉽습니다. 무료로 시작해 사용자가 늘면 유료 플랜이나 가상 서버로 옮기는 흐름을 처음부터 염두에 두세요.

## 기준과 한계

- 모든 한도와 조건은 2026-11-11에 각 서비스의 공식 요금 페이지와 문서에서 확인했습니다. 무료 등급의 범위는 자주 바뀌므로 가입 전에 다시 확인하세요.
- 이 글을 쓰면서 각 서비스에 실제로 배포해 보거나 콜드 스타트 시간을 측정하지는 않았습니다. Render의 "약 1분"은 Render 문서에 적힌 값입니다.
- Netlify 크레딧 계산 예시는 공개된 차감 기준으로 필자가 계산한 값입니다.
- 확인하지 못한 사항: Netlify Database의 2026년 7월 이후 저장 공간 요금, Netlify·Neon·Supabase·Prisma 무료 플랜의 상업적 이용 조건(이번에 읽은 문서에는 명시 없음), Supabase 무료 가입 시 카드 필요 여부, Prisma Postgres 무료 플랜이 한도에 도달했을 때의 동작, 각 DB의 기본 PostgreSQL 버전.
- 출처끼리 엇갈린 사항: Cloudflare 무료 플랜의 Worker 크기 한도(OpenNext 문서 3MiB 압축 기준, Cloudflare 한도 문서는 압축 기준 한도 없음), Supabase 일시 중지 프로젝트의 복원 가능 기간(문서 본문 1년, 화면 예시 90일).

## 출처

1. [Deploying - Next.js](https://nextjs.org/docs/app/getting-started/deploying) (확인일 2026-11-11)
2. [Vercel Pricing](https://vercel.com/pricing) (확인일 2026-11-11)
3. [Vercel Hobby Plan](https://vercel.com/docs/plans/hobby) (확인일 2026-11-11)
4. [Fair Use Guidelines - Vercel](https://vercel.com/docs/limits/fair-use-guidelines) (확인일 2026-11-11)
5. [Postgres on Vercel](https://vercel.com/docs/postgres) (확인일 2026-11-11)
6. [Netlify Pricing](https://www.netlify.com/pricing/) (확인일 2026-11-11)
7. [How credits work | Netlify Docs](https://docs.netlify.com/manage/accounts-and-billing/billing/billing-for-credit-based-plans/how-credits-work/) (확인일 2026-11-11)
8. [Next.js on Netlify | Netlify Docs](https://docs.netlify.com/frameworks/next-js/overview/) (확인일 2026-11-11)
9. [Netlify Database: Billing, limits, and compliance](https://docs.netlify.com/build/data-and-storage/netlify-database/billing-and-usage/) (확인일 2026-11-11)
10. [Limits - Cloudflare Workers docs](https://developers.cloudflare.com/workers/platform/limits/) (확인일 2026-11-11)
11. [Next.js - Cloudflare Workers docs](https://developers.cloudflare.com/workers/framework-guides/web-apps/nextjs/) (확인일 2026-11-11)
12. [Cloudflare - OpenNext](https://opennext.js.org/cloudflare) (확인일 2026-11-11)
13. [Pricing - Cloudflare Hyperdrive docs](https://developers.cloudflare.com/hyperdrive/platform/pricing/) (확인일 2026-11-11)
14. [Deploy for Free - Render Docs](https://render.com/docs/free) (확인일 2026-11-11)
15. [Neon plans](https://neon.com/docs/introduction/plans) (확인일 2026-11-11)
16. [Connection pooling - Neon Docs](https://neon.com/docs/connect/connection-pooling) (확인일 2026-11-11)
17. [Supabase Pricing](https://supabase.com/pricing) (확인일 2026-11-11)
18. [Connect to your database - Supabase Docs](https://supabase.com/docs/guides/database/connecting-to-postgres) (확인일 2026-11-11)
19. [Upgrading - Supabase Docs](https://supabase.com/docs/guides/platform/upgrading) (확인일 2026-11-11)
20. [Prisma Pricing](https://www.prisma.io/pricing?view=full) (확인일 2026-11-11)
21. [Railway Pricing](https://railway.com/pricing) (확인일 2026-11-11)
22. [Fly.io Free Trial](https://docs.fly.io/about/free-trial) (확인일 2026-11-11)
23. [Instances | Koyeb](https://www.koyeb.com/docs/reference/instances) (확인일 2026-11-11)
