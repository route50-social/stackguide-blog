# 내 컴퓨터로 가능할까? CPU, GPU, VRAM, RAM 점검하기

> 로컬 LLM을 돌리기 전에 내 PC나 Mac의 GPU 메모리, RAM, 메모리 대역폭을 확인하는 방법과, 그 값으로 어느 크기의 모델까지 무리 없이 올릴 수 있는지 어림하는 기준을 실제 명령 결과와 공개 측정값으로 정리합니다.

- 주소: https://stackguide.route50.kr/posts/check-pc-for-local-llm-261114/
- 분류: Local LLM
- 작성일: 2026-11-14
- 정보 기준일: 2026-11-14
- 태그: 로컬LLM, LLM, 하드웨어, GPU, VRAM, RAM, CPU, 통합메모리, 메모리대역폭, AppleSilicon, NVIDIA, Ollama, llama.cpp, 양자화, KV캐시, 오프로딩, Metal, 토큰, AI기초, 입문

로컬 LLM을 써 보고 싶어 Ollama를 설치하려는데, 정작 내 노트북에 8B 모델이 돌아갈지, 32B는 무리인지 감이 오지 않을 때가 많습니다. 사양표에는 CPU, 그래픽카드, 메모리가 적혀 있지만 그중 무엇을 봐야 하는지는 알려 주지 않습니다.

> **용어 풀이**
>
> - **LLM(Large Language Model, 대규모 언어 모델):** 아주 많은 글을 학습해 문장을 이어 쓰는 큰 규모의 언어 모델입니다.
> - **로컬 LLM:** 클라우드 서비스가 아니라 내 PC나 서버에서 직접 실행하는 LLM입니다. 모델 파일 전체를 내려받아 메모리에 올려야 합니다.
> - **Ollama(올라마):** 내 컴퓨터에서 LLM을 내려받아 실행하게 해 주는 오픈 소스 도구입니다. 내부적으로 llama.cpp 계열 기술로 GGUF 파일을 실행합니다.
> - **GGUF:** llama.cpp에서 쓰는 모델 파일 형식입니다. 모델의 숫자 데이터와 이름·크기 같은 정보를 한 파일에 담습니다.

먼저 답부터 적으면, 가장 먼저 볼 값은 **GPU가 쓸 수 있는 메모리의 크기**입니다. 그래픽카드가 따로 있는 PC라면 그래픽카드의 VRAM, Apple Silicon Mac이라면 통합 메모리 가운데 GPU에 허용되는 몫입니다. 그다음이 RAM 크기, 그리고 속도를 정하는 메모리 대역폭입니다. CPU는 GPU를 쓸 수 없을 때 중요해집니다. 이 글에서는 네 부품이 각각 무슨 일을 하는지 살펴보고, Windows와 macOS에서 그 값을 확인하는 방법, 확인한 값으로 모델 크기를 어림하는 기준을 차례로 정리합니다.

> **용어 풀이**
>
> - **CPU(Central Processing Unit, 중앙 처리 장치):** 컴퓨터의 일반적인 계산을 맡는 프로세서입니다. 코어 수는 적지만 복잡한 작업을 순서대로 빠르게 처리합니다.
> - **GPU(Graphics Processing Unit, 그래픽 처리 장치):** 단순한 계산을 수천 개씩 동시에 처리하는 프로세서입니다. LLM의 계산은 큰 행렬 곱셈이 대부분이라 GPU가 CPU보다 훨씬 유리합니다.
> - **VRAM(Video RAM, 그래픽 메모리):** 그래픽카드에 달린 전용 메모리입니다. GPU가 가장 빠르게 읽을 수 있는 메모리입니다.
> - **RAM(Random Access Memory, 시스템 메모리):** 메인보드에 꽂는 일반 메모리입니다. 운영체제와 프로그램이 함께 쓰며, 사양표에서 "메모리 16GB"라고 적힌 값이 이것입니다.

## 네 부품이 하는 일

LLM이 답을 쓰는 과정은 단순하게 보면 "모델 파일에 든 숫자를 메모리에서 읽어 와 계산하기"의 반복입니다. 모델 파일은 SSD에 저장되어 있다가 실행할 때 메모리로 올라가고, 계산 장치가 그 메모리를 읽습니다. 그래서 모델이 **어느 메모리에 올라가는지**가 속도를 크게 좌우합니다.

*(그림: 왼쪽은 그래픽카드가 있는 PC로, SSD의 모델 파일이 RAM을 거쳐 VRAM에 올라가 GPU가 계산하고, VRAM에 넘친 층은 RAM에 남아 CPU가 계산합니다. 오른쪽은 Apple Silicon Mac으로, SSD의 모델 파일이 CPU와 GPU가 함께 쓰는 통합 메모리에 올라가며 GPU는 권장 한도 안에서 그 메모리를 바로 읽습니다.)*

*그림 1. 그래픽카드가 있는 PC와 Apple Silicon Mac에서 모델이 올라가는 메모리 (자료: Ollama Hardware support·FAQ, Apple recommendedMaxWorkingSetSize 문서, NVIDIA GeForce 비교표, 2026-11-14 확인. DDR4-3200 2채널 대역폭은 필자 계산)*

그래픽카드가 있는 PC에서는 모델이 VRAM에 다 들어가면 GPU가 전부 계산합니다. 다 들어가지 않으면 Ollama는 모델을 나눠 일부는 VRAM에, 나머지는 RAM에 올리고, RAM에 남은 부분은 CPU가 계산합니다. Ollama FAQ는 이때 `ollama ps`의 `PROCESSOR` 열에 `48%/52% CPU/GPU`처럼 나뉜 비율이 표시되고, 전부 GPU에 올라가면 `100% GPU`, 전부 RAM이면 `100% CPU`로 표시된다고 설명합니다. 나뉘어 올라간 모델은 느린 쪽에 발목이 잡힙니다.

> **용어 풀이**
>
> - **오프로딩(Offloading):** 모델의 층 가운데 일부를 GPU 메모리로 옮겨 계산을 맡기는 일입니다. 다 옮기지 못하면 나머지는 CPU와 RAM이 처리합니다.

Apple Silicon Mac은 구조가 다릅니다. CPU와 GPU가 하나의 통합 메모리를 함께 씁니다. Apple은 M1을 발표하면서 이 구조 덕분에 칩 안의 여러 장치가 "여러 메모리 사이에서 복사하지 않고 같은 데이터에 접근한다"고 설명했습니다. 그래서 Mac에서는 "RAM 16GB"가 곧 GPU가 쓸 수 있는 메모리의 상한이 되지만, 그 전부를 GPU가 쓸 수는 없습니다. 이 부분은 뒤의 macOS 점검에서 다시 다룹니다.

> **용어 풀이**
>
> - **통합 메모리(Unified Memory):** CPU와 GPU가 따로 메모리를 두지 않고 하나의 메모리를 함께 쓰는 구조입니다. Apple Silicon(M1 이후의 Mac용 칩)이 이 방식을 씁니다.

CPU는 두 경우에 중요해집니다. 그래픽카드가 없거나 Ollama가 지원하지 않는 GPU여서 CPU만으로 돌릴 때, 그리고 위처럼 VRAM이 모자라 일부 층을 CPU가 맡을 때입니다. SSD는 모델 파일을 보관하는 곳입니다. Ollama 문서는 Windows 설치에 최소 4GB가 필요하고, 모델은 "수십에서 수백 GB"까지 커질 수 있으니 공간을 따로 확보하라고 안내합니다.

## 속도는 메모리 대역폭이 정합니다

얼마나 큰 모델이 들어가는지는 메모리 크기가, 들어간 모델이 얼마나 빨리 글을 쓰는지는 메모리 대역폭이 크게 좌우합니다. LLM은 토큰을 하나 만들 때마다 모델의 숫자 대부분을 메모리에서 읽어야 하기 때문입니다. 8B 모델을 4비트 남짓으로 줄인 약 4.9GB짜리 파일이라면, 토큰 하나를 쓸 때마다 수 GB를 읽는 셈입니다.

> **용어 풀이**
>
> - **토큰(Token):** 언어 모델이 글을 처리하는 기본 단위입니다. 단어 하나일 수도, 단어의 일부나 글자 하나일 수도 있습니다. 생성 속도는 보통 초당 토큰 수(t/s)로 나타냅니다.
> - **메모리 대역폭(Memory Bandwidth):** 메모리에서 1초에 읽거나 쓸 수 있는 데이터의 양입니다. GB/s 단위로 적습니다.

그래서 "대역폭 ÷ 모델 파일 크기"로 초당 토큰 수의 이론 상한을 어림할 수 있습니다. 이 식이 실제와 얼마나 맞는지 llama.cpp 저장소의 Apple Silicon 측정 자료로 확인해 봤습니다. 이 자료는 Llama 2 7B를 Q4_0(3.56GiB)으로 줄인 같은 파일을 여러 Mac에서 `llama-bench`로 실행해 글 생성 속도(tg128)를 모아 둔 것입니다.

> **용어 풀이**
>
> - **llama.cpp:** LLM을 CPU나 일반 그래픽카드에서도 돌릴 수 있게 만든 C/C++ 오픈 소스 실행 도구입니다. `llama-bench`는 여기에 포함된 성능 측정 도구입니다.
> - **양자화(Quantization):** 모델의 숫자를 더 적은 비트로 줄여 저장하는 방법입니다. Q4_0, Q4_K_M처럼 Q 뒤의 숫자가 대략의 비트 수입니다. 파일과 필요한 메모리가 줄어드는 대신 정확도가 조금 떨어질 수 있습니다.

*(그림: Llama 2 7B Q4_0 글 생성 속도의 실측값과 이론 상한을 Apple Silicon 칩별로 비교한 가로 막대그래프. M1 8코어(68GB/s) 14.2 대 17.8, M2 10코어(100GB/s) 21.9 대 26.2, M4 10코어(120GB/s) 24.1 대 31.4, M4 Pro 20코어(273GB/s) 50.7 대 71.4, M3 Max 40코어(400GB/s) 66.3 대 104.6, M4 Max 40코어(546GB/s) 83.1 대 142.8토큰/초. 대역폭이 클수록 빨라지지만 상한과의 격차도 커집니다.)*

*그림 2. Apple Silicon의 메모리 대역폭과 글 생성 속도 (자료: llama.cpp Discussion #4167 요약표, 2026-11-14 확인. 이론 상한은 대역폭을 파일 크기 3.82GB로 나눈 필자 계산값)*

대역폭이 68GB/s인 M1과 546GB/s인 M4 Max는 대역폭이 8배 차이 나고, 실측 속도도 약 5.9배 차이 납니다. 실측값은 이론 상한의 58~84% 사이였고, 대역폭이 큰 칩일수록 상한과의 격차가 컸습니다. 대역폭이 충분히 크면 메모리를 읽는 것 말고 다른 계산이 병목이 되기 때문으로 보이지만, 원인을 이 자료로 확인하지는 않았습니다. 어쨌든 "대역폭 ÷ 파일 크기"는 넘을 수 없는 천장으로 보고, 실제로는 그보다 낮게 나온다고 생각하면 됩니다.

같은 방식으로 다른 하드웨어도 비교할 수 있습니다. NVIDIA 비교표(2026-11-14 확인)에 따르면 RTX 5060의 VRAM 대역폭은 448GB/s, RTX 5090은 1,792GB/s입니다. 반면 일반 PC의 RAM은 DDR4-3200 두 장을 2채널로 쓸 때 3,200MT/s × 8바이트 × 2채널 = 51.2GB/s입니다(필자 계산). VRAM이 모자라 일부 층이 RAM으로 밀려나면 그 부분은 수 배에서 수십 배 느린 메모리를 읽게 되는 셈입니다. Mac이 큰 모델을 비교적 빠르게 돌리는 것도 통합 메모리 대역폭이 일반 PC의 RAM보다 넓어서입니다.

> **용어 풀이**
>
> - **DDR4-3200·MT/s:** DDR4는 PC용 RAM의 규격 세대이고, 3200은 1초에 32억 번(3,200MT/s, Mega Transfers per second) 데이터를 주고받는다는 뜻입니다. 한 번에 64비트(8바이트)씩 전송합니다.
> - **2채널(Dual Channel):** 메모리 두 장을 서로 다른 통로에 꽂아 동시에 읽어 대역폭을 두 배로 쓰는 방식입니다.

## Windows에서 점검하기

아래 명령은 모두 Windows 11 Pro(빌드 26300)의 PowerShell 5.1에서 실행했고, 출력은 2026-11-14에 실제로 받은 결과입니다. 시험한 컴퓨터는 Intel Core i5-1340P와 내장 그래픽(Iris Xe), 64GB RAM을 단 노트북으로, 그래픽카드가 따로 없습니다. 모두 정보를 읽기만 하는 명령이라 설정을 바꾸지 않습니다.

### CPU와 RAM

```powershell
# 실행 위치: 로컬 PC (Windows 11 Pro, PowerShell 5.1). 관리자 권한은 필요 없습니다.
Get-CimInstance Win32_Processor | Select-Object Name, NumberOfCores, NumberOfLogicalProcessors
```

```text
Name                                NumberOfCores NumberOfLogicalProcessors
----                                ------------- -------------------------
13th Gen Intel(R) Core(TM) i5-1340P            12                        16
```

```powershell
# 실행 위치: 로컬 PC (Windows 11 Pro, PowerShell 5.1). 메모리 모듈마다 한 줄씩 표시됩니다.
Get-CimInstance Win32_PhysicalMemory | Select-Object Capacity, ConfiguredClockSpeed
```

```text
   Capacity ConfiguredClockSpeed
   -------- --------------------
34359738368                 3200
34359738368                 3200
```

32GiB 메모리 두 장이 3,200MT/s로 동작하고 있습니다. 앞에서 계산한 51.2GB/s가 이 노트북의 이론 대역폭입니다(두 장이 2채널로 동작한다고 가정). 앞의 식으로 어림하면, 이 노트북에서 약 4.9GB인 8B Q4_K_M 모델을 CPU로 돌릴 때 초당 10토큰 남짓이 상한입니다. 이 노트북에 Ollama를 설치해 직접 측정하지는 않았으므로, 실제 속도는 이보다 낮을 수 있습니다.

### 그래픽카드와 VRAM

Windows에서 GPU 메모리를 확인하는 가장 쉬운 방법은 작업 관리자의 성능 탭에서 GPU를 고르는 것입니다. 여기에는 "전용 GPU 메모리"와 "공유 GPU 메모리"가 따로 나옵니다. Microsoft DirectX 팀의 설명에 따르면 전용 메모리는 GPU만 쓰도록 예약된 메모리이고, 공유 메모리는 "CPU와 GPU 모두 쓸 수 있는 일반 시스템 메모리"입니다. 공유 GPU 메모리는 VRAM이 아니라 RAM이므로, 앞에서 본 것처럼 대역폭이 VRAM보다 훨씬 낮습니다.

텍스트로 확인하려면 DirectX 진단 도구(dxdiag)의 결과를 파일로 저장해 찾아봅니다.

```powershell
# 실행 위치: 로컬 PC (Windows 11 Pro, PowerShell 5.1). 진단 결과를 임시 폴더에 텍스트로 저장합니다.
Start-Process dxdiag -ArgumentList "/t `"$env:TEMP\dxdiag.txt`"" -Wait
Select-String -Path "$env:TEMP\dxdiag.txt" -Pattern "Card name|Dedicated Memory|Shared Memory" |
  ForEach-Object { $_.Line.Trim() }
```

```text
Card name: Intel(R) Iris(R) Xe Graphics
Dedicated Memory: 128 MB
Shared Memory: 48636 MB
```

내장 그래픽이라 전용 메모리는 128MB뿐이고, 나머지는 모두 RAM을 빌려 쓰는 공유 메모리입니다. 2017년 DirectX 팀 글에는 GPU가 한 시점에 물리 메모리의 절반까지만 쓸 수 있게 한다는 Windows 정책이 소개되어 있는데, 이 노트북에서는 사용 가능한 메모리 65,020MB 가운데 약 75%인 48,636MB가 공유 메모리로 잡혀 있었습니다. 이 값은 Windows 버전과 그래픽 드라이버에 따라 다를 수 있으니, 내 컴퓨터에서 직접 확인하는 편이 정확합니다.

인터넷에서 자주 보이는 아래 명령은 조심해서 읽어야 합니다.

```powershell
# 실행 위치: 로컬 PC (Windows 11 Pro, PowerShell 5.1)
Get-CimInstance Win32_VideoController | Select-Object Name, AdapterRAM
```

```text
Name                         AdapterRAM
----                         ----------
Intel(R) Iris(R) Xe Graphics 2147479552
```

같은 그래픽인데 dxdiag와 전혀 다른 약 2GiB가 나왔습니다. Microsoft 문서를 보면 `AdapterRAM`은 32비트 정수(`uint32`)라서 표현할 수 있는 최댓값이 약 4GiB입니다. VRAM이 8GB, 16GB인 그래픽카드도 이 값으로는 제대로 나오지 않을 수 있으니, VRAM 크기는 작업 관리자나 dxdiag, 제조사 도구로 확인합니다.

> **용어 풀이**
>
> - **WMI·CIM(Windows Management Instrumentation·Common Information Model):** Windows가 하드웨어와 시스템 정보를 표준 형식으로 제공하는 관리 기능입니다. PowerShell의 `Get-CimInstance`로 그 정보를 읽습니다.

NVIDIA 그래픽카드가 있다면 드라이버와 함께 설치되는 `nvidia-smi`가 가장 정확합니다. 이 노트북에는 NVIDIA 그래픽카드가 없어 실행하지 못했고, 아래는 NVIDIA 문서의 `--query-gpu` 옵션으로 구성한 명령입니다.

```powershell
# 실행 위치: NVIDIA 그래픽카드와 드라이버가 설치된 PC (Windows·Linux 공통)
# 이 글의 시험 환경에서는 실행하지 못했습니다(그래픽카드 없음).
nvidia-smi --query-gpu=name,memory.total,memory.used,driver_version --format=csv
```

`memory.total`이 VRAM 전체 크기, `memory.used`가 지금 쓰고 있는 양입니다. 화면 출력이나 다른 프로그램도 VRAM을 쓰기 때문에, 모델에 쓸 수 있는 양은 전체보다 조금 적다고 보는 것이 안전합니다.

### Ollama가 내 GPU를 쓸 수 있는지

VRAM이 넉넉해도 Ollama가 그 GPU를 지원하지 않으면 CPU로 돌아갑니다. Ollama 하드웨어 지원 문서(2026-11-14 확인)의 기준은 다음과 같습니다.

| 종류 | 지원 조건 |
| --- | --- |
| NVIDIA | Compute Capability 5.0 이상, 드라이버 550 이상(5.0~6.2는 570 이상). Windows 설치 문서는 551.61 이상을 요구 |
| AMD (Windows) | ROCm v7을 쓰는 Radeon RX 7600~7900 계열, Radeon PRO W7500~W7900. 그 밖의 카드는 Vulkan으로 추가 지원 |
| AMD (Linux) | ROCm v7 드라이버, Radeon RX 6800 이후 계열 등. 그 밖의 카드는 Vulkan으로 추가 지원 |
| Apple | Apple Silicon(M 시리즈)의 Metal. Intel 칩 Mac은 CPU만 사용 |

*표 1. Ollama의 GPU 지원 조건 요약 (자료: Ollama Hardware support, Windows, macOS 문서, 2026-11-14 확인)*

> **용어 풀이**
>
> - **Compute Capability:** NVIDIA가 GPU 세대마다 매기는 기능 수준 번호입니다. 숫자가 클수록 최신 세대이며, 프로그램이 요구하는 최소 번호보다 낮으면 GPU 가속을 쓸 수 없습니다.
> - **ROCm·Vulkan·Metal:** 프로그램이 GPU에 계산을 맡길 때 쓰는 소프트웨어 기반입니다. ROCm은 AMD, Metal은 Apple이 만든 것이고, Vulkan은 여러 제조사의 GPU에서 쓰는 공개 표준입니다.

이 노트북 같은 Intel 내장 그래픽은 Ollama 문서의 지원 목록에 따로 적혀 있지 않습니다. 문서는 목록 밖의 GPU를 Vulkan으로 지원한다고만 설명하므로, 실제로 GPU를 쓰는지는 모델을 실행한 뒤 `ollama ps`의 `PROCESSOR` 열로 확인해야 합니다. 이번에는 직접 확인하지 못했습니다.

## macOS에서 점검하기

Mac에서는 확인할 값이 더 단순합니다. 칩 이름과 메모리 크기를 알면 대역폭은 Apple 사양 페이지에서 찾을 수 있습니다. 메뉴의 "이 Mac에 관하여"에서 칩과 메모리를 볼 수 있고, 터미널에서는 다음 명령을 씁니다. 이번에는 Mac에서 실행해 보지 못했으므로 출력 예시는 싣지 않았습니다.

```sh
# 실행 위치: 로컬 Mac의 터미널. 정보를 읽기만 하며 관리자 권한은 필요 없습니다.
# 이 글의 시험 환경에서는 실행하지 못했습니다.
system_profiler SPHardwareDataType   # 칩 이름(Chip)과 메모리(Memory) 표시
sysctl hw.memsize                    # 전체 메모리를 바이트 단위로 표시
```

Apple 사양 페이지(2026-11-14 확인)를 보면 대역폭은 칩 등급에 따라 크게 다릅니다. 현재 MacBook Air의 M5는 153GB/s이고 메모리는 16·24·32GB 중에서 고릅니다. 14형 MacBook Pro의 M4 Pro는 273GB/s, M4 Max는 구성에 따라 410GB/s 또는 546GB/s입니다. 같은 32GB라도 칩에 따라 생성 속도가 몇 배씩 달라질 수 있습니다.

### GPU가 쓸 수 있는 몫

통합 메모리를 모두 GPU가 쓸 수는 없습니다. Apple의 Metal 문서에는 `recommendedMaxWorkingSetSize`라는 값이 있는데, "이 GPU가 실행 성능에 영향을 주지 않고 할당할 수 있는 메모리의 근삿값"으로 설명되어 있습니다. Apple은 이 값이 전체 메모리의 몇 %인지 문서로 정해 두지 않았습니다.

이 값은 llama.cpp가 Metal을 초기화할 때 로그에 `recommendedMaxWorkingSetSize = ... MB` 형식으로 찍습니다(llama.cpp `ggml-metal-device.m` 소스 기준). 터미널에서 llama.cpp를 직접 실행하거나 Ollama 서버 로그를 열어 이 줄을 찾으면 내 Mac의 실제 한도를 알 수 있습니다. 모델 파일과 KV 캐시를 합친 크기가 이 값을 넘지 않아야 GPU에서 제 속도가 납니다.

> **용어 풀이**
>
> - **KV 캐시(Key-Value Cache):** 모델이 이미 읽은 토큰마다 계산해 둔 중간값을 저장하는 메모리 공간입니다. 다시 계산하지 않아도 되니 빨라지지만, 컨텍스트가 길수록 메모리를 더 차지합니다.
> - **컨텍스트(Context):** 모델이 한 번에 볼 수 있는 토큰의 범위입니다. 입력한 글과 이전 대화, 모델이 쓴 답을 합한 길이입니다.

이 한도를 `sudo sysctl iogpu.wired_limit_mb=<MB 단위 값>`으로 늘리는 방법이 MLX 문서와 여러 사용자 글에 소개되어 있습니다. 다만 Apple이 공식 문서로 안내하는 설정이 아니고, 너무 크게 잡으면 macOS와 다른 앱이 쓸 메모리가 모자라 시스템 전체가 느려지거나 불안정해질 수 있습니다. 쓰더라도 바꾸기 전에 `sysctl iogpu.wired_limit_mb`로 현재 값을 확인해 기록해 두고, 처음에는 기본 한도 안에 들어가는 모델부터 고르는 편이 안전합니다.

> **용어 풀이**
>
> - **MLX:** Apple이 공개한 Apple Silicon용 머신러닝 프레임워크입니다. Mac에서 LLM을 실행하는 도구 가운데 일부가 이를 바탕으로 동작합니다.

## 내 메모리로 어느 크기까지

확인한 값을 모델 크기로 옮겨 보겠습니다. llama.cpp 양자화 문서의 Llama 3.1 8B 측정표에서 Q4_K_M은 파라미터당 평균 4.89비트이므로, 모델 파일 크기는 "파라미터 수(B) × 4.89 ÷ 8", 곧 파라미터 10억 개당 약 0.61GB로 어림할 수 있습니다. 여기에 KV 캐시와 실행 도구의 작업 공간이 더 필요합니다. 아래 표는 Ollama의 기본 컨텍스트(VRAM 24GiB 미만이면 4k)를 기준으로 그 몫을 1~2GB로 잡은 어림값입니다.

> **용어 풀이**
>
> - **파라미터(Parameter, 매개변수):** 학습 과정에서 조정되는 모델 내부의 숫자입니다. 모델 이름의 `8B`는 파라미터가 약 80억 개라는 뜻입니다.

| 모델 크기 | Q4_K_M 파일(계산값) | 4k 컨텍스트 실행에 필요한 메모리(어림) | 무리 없이 맞는 GPU 메모리 |
| --- | --- | --- | --- |
| 3B | 약 1.8GB | 약 3GB | 4GB 이상 |
| 8B | 약 4.9GB (실제 파일 4.92GB) | 약 6~7GB | 8GB 이상 |
| 14B | 약 8.6GB | 약 10~11GB | 12GB 이상 |
| 32B | 약 19.6GB | 약 21~22GB | 24GB 이상 |
| 70B | 약 42.8GB (llama.cpp 문서 43.1GB) | 약 45GB | 48GB 이상 |

*표 2. Q4_K_M 기준 모델 크기별 필요한 메모리 어림값. 파일 크기는 4.89비트로 계산했고, KV 캐시·작업 공간은 1~2GB로 가정했습니다. 실제 파라미터 수는 이름의 숫자와 조금씩 다르며, 컨텍스트를 늘리면 필요한 메모리가 더 커집니다.*

표를 쓰는 방법은 이렇습니다. NVIDIA 그래픽카드가 있는 PC라면 VRAM 크기로 오른쪽 열을 찾습니다. 8GB 카드는 8B급, 12GB는 14B급, 16GB는 14B급에 긴 컨텍스트를 쓰거나 20B 안팎까지, 24GB는 32B급이 기준이 됩니다. 한 단계 큰 모델을 올리면 일부가 RAM으로 넘어가 돌아가기는 하지만, 넘친 비율만큼 느려집니다.

Mac이라면 전체 메모리가 아니라 앞의 `recommendedMaxWorkingSetSize`를 기준으로 봅니다. 16GB Mac은 macOS와 브라우저 같은 앱도 같은 메모리를 쓰므로 8B급이 편안한 범위이고, 14B급은 다른 앱을 줄여야 할 수 있습니다.

그래픽카드 없이 CPU만 쓰는 PC라면 RAM이 모델 크기의 한계입니다. RAM 16GB면 운영체제와 다른 프로그램을 빼고 8B급까지가 현실적이고, 32GB 이상이면 14B급도 올라갑니다. 다만 앞의 계산처럼 일반 RAM의 대역폭으로는 8B 모델도 초당 10토큰 안팎이 상한이므로, 큰 모델일수록 답을 기다리는 시간이 길어집니다. 메모리가 빠듯하면 같은 모델의 더 작은 양자화(Q3, Q2 계열)를 쓰거나 한 단계 작은 모델을 고르는 방법이 있지만, 비트를 줄일수록 품질이 떨어질 여지가 커집니다.

## 점검 순서 정리

정리하면 다음 순서로 확인하면 됩니다.

1. **GPU 메모리:** Windows는 작업 관리자나 dxdiag의 전용 메모리, NVIDIA는 `nvidia-smi`의 `memory.total`. Mac은 칩·메모리 크기와 llama.cpp 로그의 `recommendedMaxWorkingSetSize`.
2. **Ollama 지원 여부:** 표 1의 조건에 맞는지 보고, 모델을 실행한 뒤 `ollama ps`가 `100% GPU`인지 확인합니다.
3. **RAM:** 그래픽카드가 없거나 VRAM이 모자랄 때 모델이 들어갈 공간입니다.
4. **대역폭:** 그래픽카드나 칩의 사양표에서 GB/s 값을 찾아 "대역폭 ÷ 모델 파일 크기"로 속도의 상한을 어림합니다.
5. **저장 공간:** 받을 모델 크기만큼 SSD에 여유를 둡니다.

이 기준은 Q4_K_M과 짧은 컨텍스트를 전제로 한 어림값입니다. 같은 크기의 모델이라도 구조에 따라 KV 캐시 크기가 다르고, Ollama와 llama.cpp의 버전이 바뀌면 메모리를 쓰는 방식도 달라질 수 있습니다. 표로 후보를 좁힌 다음에는 실제로 받아 `ollama ps`로 메모리와 `PROCESSOR` 비율을 확인하는 것이 가장 확실합니다.

## 출처

1. [Hardware support - Ollama](https://docs.ollama.com/gpu) (확인일 2026-11-14)
2. [Windows - Ollama](https://docs.ollama.com/windows) (확인일 2026-11-14)
3. [macOS - Ollama](https://docs.ollama.com/macos) (확인일 2026-11-14)
4. [Local FAQ - Ollama](https://docs.ollama.com/faq) (확인일 2026-11-14)
5. [Performance of llama.cpp on Apple Silicon M-series (Discussion #4167) - ggml-org/llama.cpp](https://github.com/ggml-org/llama.cpp/discussions/4167) (확인일 2026-11-14)
6. [llama.cpp/tools/quantize/README.md - ggml-org/llama.cpp](https://github.com/ggml-org/llama.cpp/blob/master/tools/quantize/README.md) (확인일 2026-11-14)
7. [Context length - Ollama](https://docs.ollama.com/context-length) (확인일 2026-11-14)
8. [bartowski/Meta-Llama-3.1-8B-Instruct-GGUF - Hugging Face](https://huggingface.co/bartowski/Meta-Llama-3.1-8B-Instruct-GGUF) (확인일 2026-11-14)
9. [ggml/src/ggml-metal/ggml-metal-device.m - ggml-org/llama.cpp](https://github.com/ggml-org/llama.cpp/blob/master/ggml/src/ggml-metal/ggml-metal-device.m) (확인일 2026-11-14)
10. [recommendedMaxWorkingSetSize - Apple Developer Documentation](https://developer.apple.com/documentation/metal/mtldevice/recommendedmaxworkingsetsize) (확인일 2026-11-14)
11. [mlx.core.set_wired_limit - MLX documentation](https://ml-explore.github.io/mlx/build/html/python/_autosummary/mlx.core.set_wired_limit.html) (확인일 2026-11-14)
12. [Apple unleashes M1 - Apple Newsroom](https://www.apple.com/newsroom/2020/11/apple-unleashes-m1/) (확인일 2026-11-14)
13. [MacBook Pro (14-inch, M4 Pro or M4 Max, 2024) - Tech Specs - Apple Support](https://support.apple.com/en-us/121553) (확인일 2026-11-14)
14. [MacBook Air - Tech Specs - Apple](https://www.apple.com/macbook-air/specs/) (확인일 2026-11-14)
15. [Compare Current and Previous GeForce Series of Graphics Cards - NVIDIA](https://www.nvidia.com/en-us/geforce/graphics-cards/compare/) (확인일 2026-11-14)
16. [GPUs in the task manager - DirectX Developer Blog](https://devblogs.microsoft.com/directx/gpus-in-the-task-manager/) (확인일 2026-11-14)
17. [Win32_VideoController class - Microsoft Learn](https://learn.microsoft.com/en-us/windows/win32/cimwin32prov/win32-videocontroller) (확인일 2026-11-14)
