# CLI : 명령줄

> **전체 참조:**[CLI 참조](reference/cli-reference.md)는**모든 하위 명령어의 모든 플래그**를 설명하며, AI 어시스턴트에 최적화되어 있습니다. URL를 어시스턴트에 붙여넣고 작동하는 명령어를 요청해 보세요: `https://mapir.gitbook.io/chloros/reference/cli-reference`
>
> **AI 도구 사용 팁:** 이 설명서의 모든 페이지는 해당 페이지의 URL 뒤에 `.md`를 추가하면 원본 마크다운 형식으로 확인할 수 있습니다 (예: `https://mapir.gitbook.io/chloros/reference/cli-reference.md`)를 추가하면 해당 페이지의 원본 마크다운 형식으로 확인할 수 있으며, `https://mapir.gitbook.io/chloros/llms.txt`는 LLM이 활용할 수 있도록 전체 매뉴얼을 색인화합니다.

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: banner shows CLI 1.1.0; reshoot the CLI welcome/banner output on the 1.2.0 build so the version line reads "Chloros CLI 1.2.0" -->
## CLI란?

`chloros-cli`는 Chloros 데스크톱 앱이 사용하는 것과 동일한 처리 엔진의 명령줄 프론트엔드입니다. 이는 Chloros 백엔드(`127.0.0.1:5000`상의 로컬 서버) 위에 구축된 경량 HTTP 클라이언트입니다. 대부분의 명령은 백엔드를 자동으로 시작하므로, 따라서 스크립트에서는 단 한 번의 `chloros-cli process …` 호출만으로 충분합니다.

이 앱은 **Windows 10/11 (x64)**및**Linux (x86_64, JetPack 6 기반의 NVIDIA Jetson arm64)**에서 실행되며, GUI 없이 어떤 터미널에서든 실행할 수 있습니다. 다음 명령어로 설치 상태를 확인하세요:

```bash
chloros-cli --version    # prints "Chloros CLI 1.2.0"
```

명령어 그룹 요약:

* **처리 및 계정** — `process`, `login`, `logout`, `status`, `export-status`, `language` (38개 언어 — [지원 언어](supported-languages.md) 참조), `set-project-folder` / `get-project-folder` / `reset-project-folder`, `selftest`, `update` (Linux/Jetson 전용)
* **실시간 하드웨어** — `lattice` (LATTICE 카메라 제어, 45개 이상의 하위 명령어), `daq pool-*` (DAQ 광 센서), `time-sync` (PTP)
* **자동화** — `project` (YAML 캡처 레시피 포함, 저장된 Chloros 프로젝트를 헤드리스 모드로 실행)

알아두면 유용한 전역 옵션: `--port N` (백엔드 포트, 기본값 `5000`), `-v/--verbose`, `--restart` (백엔드 강제 재시작), `--backend-exe PATH`. 전체 목록은 [CLI 참조](reference/cli-reference.md)를 참조하십시오.

***

## 설치

CLI는 모든 플랫폼에서 **Chloros 설치 프로그램 내에 포함되어 제공됩니다**. 별도의 CLI 다운로드 파일은 없습니다. [다운로드](download.md) 페이지에서 설치 프로그램을 다운로드하십시오.

### Windows

설치 프로그램은 CLI를 다음 위치에 설치합니다:

```

C:\Program Files\Chloros\cli\chloros-cli.exe
```

에 CLI를 설치하고, 해당 폴더를 시스템의 `PATH`에 추가합니다. 설치 후 **새 터미널을 열어서**업데이트된 `PATH`가 적용되도록 하십시오. 또한 설치 프로그램은 설치 루트 디렉터리에 런처 스크립트 (`Chloros_CLI.bat` / `Chloros_CLI.ps1`)을 설치 루트에 배치하고,**Chloros CLI** 시작 메뉴 바로가기를 생성하며, 각각을 실행하면 `chloros-cli`가 실행 준비된 상태로 터미널이 열립니다.

### Linux

사용 중인 아키텍처에 맞는 `.deb`를 설치하십시오:

```bash
# Linux x86_64
sudo dpkg -i chloros-amd64.deb

# NVIDIA Jetson (arm64, JetPack 6)
sudo dpkg -i chloros-arm64-jp6.deb
```

이 명령을 실행하면 `chloros-cli`가 `/usr/bin/chloros-cli` (이미 `PATH`가 설치된 경우) 및 백엔드를 `/usr/lib/chloros/chloros-backend`로 업데이트하며, LATTICE 카메라에 필요한 Arena SDK 런타임도 함께 설치합니다. 자세한 내용은 [Linux 설치](linux/linux-installation.md)를 참조하십시오.

### 확인

```bash
chloros-cli --version    # "Chloros CLI 1.2.0"
chloros-cli selftest     # 7-step diagnostic: backend, API, GPU/CUDA, denoiser models
chloros-cli status       # license tier + logged-in user
```

***

## 로그인 및 라이선스

CLI(및 Python, SDK)에 액세스하려면 **유료 Chloros+ 요금제**가 필요합니다. — 유료 요금제라면 어떤 것이든 해당 기능이 제공되지만, 무료 요금제에서는 제공되지 않습니다. 이 제한은 CLI 바이너리가 아닌 백엔드에 의해**서버 측**에서 적용됩니다: 로그아웃 상태에서의 호출은 `401 AUTH_REQUIRED` 오류로 거부되며, 무료 요금제에서 로그인된 상태의 호출은 `403 PLAN_UPGRADE_REQUIRED` 오류로 거부됩니다. 이는 호출이 `chloros-cli`, SDK, 또는 직접 개발한 HTTP 클라이언트에서 오는 요청이든 상관없이 적용됩니다. [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)에서 업그레이드하십시오.**한 대의 컴퓨터당 한 번** 로그인하십시오:

```bash
chloros-cli login user@example.com 'YourPassword'
chloros-cli status
```

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: login success output predates 1.2.0; reshoot `chloros-cli login` followed by `chloros-cli status` on the 1.2.0 build showing the license tier line -->
{% hint style="warning" %}
**특수 문자가 포함된 비밀번호**(`$`, `!`, spaces): wrap the password in**single quotes**, as shown above. In PowerShell double quotes, `$$`는 셸에서 왜곡됩니다. CLI는 401 오류 시 이를 감지하여 자동으로 재시도하지만, 작은따옴표를 사용하면 이 문제를 완전히 피할 수 있습니다).
{% endhint %}

세션은 `~/.chloros/user_session.json`에 캐시되며, 요금제의 유예 기간 동안(월간 요금제의 경우 30일, 연간 요금제의 경우 만료일까지) 오프라인 상태에서도 계속 작동합니다. `chloros-cli status`는 유료 플랜이 없어도 작동하므로, 거부 사유를 항상 확인할 수 있습니다.

{% hint style="danger" %}
**헤드리스 작업을 예약하시나요? 먼저 로그인하세요.**백엔드 생성 명령(`process`, `status`, `export-status`, …)를**캐시된 세션 없이**실행하면 즉시 실패하지 않고, 표준 입력(stdin)을 통해 대화형 `Email:` / `Password:` 프롬프트로 넘어갑니다. 따라서 무인 cron 작업이나 CI 단계는**입력을 기다리며 멈춰 버립니다**. 어떤 작업을 예약하기 전에 해당 시스템에서 `chloros-cli login EMAIL 'PASSWORD'`를 한 번 실행하십시오.
{% endhint %}

***

## 첫 번째 Processing 실행

`process`를 캡처 파일이 있는 폴더로 지정하면, Survey3(`.raw` + `.jpg`), LATTICE(`.tif`/`.tiff`), `.dng` 또는 이들의 혼합 파일을 자동으로 감지합니다:

```bash
chloros-cli process "C:\Images\flight_001"          # Windows
chloros-cli process ~/images/flight_001              # Linux
```

진행 상황은 파이프라인 스레드(탐지, 분석, 처리, 내보내기)별로 실시간으로 표시되며, 실행이 성공적으로 완료되면 작성된 이미지 산출물의 수(`Image products written: N`)를 보고함으로써 종료됩니다.

<!-- SCREENSHOT-NEEDED: terminal capture of a `chloros-cli process` run on a LATTICE captures folder completing successfully — per-thread progress lines visible and the final "Image products written: N" summary line -->
### 출력 파일 저장 위치

`process`는 입력 폴더가 아닌 **프로젝트 폴더**에 파일을 기록합니다:

* `-o`가 없는 경우: 프로젝트는 기본 프로젝트 폴더(GUI와 공유됨; `get-project-folder` / `set-project-folder`, 대체 옵션 `~/Chloros Projects`) 아래에 생성되며, 생략 시 `-n/--project-name` 또는 타임스탬프(`YYYYMMDD_HHMMSS`)로 이름이 지정됩니다.
* `-o PATH`를 사용할 경우: 해당 폴더가 **바로** 프로젝트 폴더입니다. 해당 폴더에 이미 `project.json`가 존재하는 경우, 덮어쓰기 대신 `_1`/`_2`…와 같은 접미사가 붙은 하위 폴더가 생성됩니다.

프로젝트 내부에서 결과물은 **카메라별, 그 다음 파일 형식별로** 그룹화됩니다:

```
<project>/
├── project.json
├── calibration_data.json
└── LATT-M3M-L41-F550/                  # one folder per camera model+lens+filter
    ├── tiff16/
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── NDVI_Index_Images/           # one folder per requested index
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

카메라 폴더는 LATTICE의 경우 `LATT-<sensor>-<lens>-F<filter>`(캡처된 이미지의 EXIF 정보인 `Model`와 일치)이며, `<model>_<filter>` (예: `Survey3N_RGN`)입니다. 포맷 폴더는 `--format` 형식을 따릅니다: `tiff16`, `tiff8`, `png8`, `jpg8`, 또는 `TIFF (32-bit, Percent)`의 경우 `tiff32`와 같은 형식을 따릅니다.

{% hint style="info" %}
**내보낸 모든 제품은 소스 파일의 이름을 그대로 유지합니다.**`capture_..._raw.tif`의 Radiance 내보내기 파일은 여전히 `capture_..._raw.tif`라는 이름을 가지며, 단지 `tiff32/Radiance_Images/` 폴더에 위치할 뿐입니다.**폴더가 제품을 식별하며, 파일 이름이 아닙니다**. 따라서 `*radiance*` 접미사가 아닌 디렉터리를 대상으로 글로브(glob)를 사용하십시오.
{% endhint %}

### 실제로 사용할 옵션

| 플래그 | 기본값 | 기능 |
| --- | --- | --- |
| `-o, --output PATH` | 기본 프로젝트 폴더 | 프로젝트 폴더 위치 (위 참조). |
| `-n, --project-name NAME` | 타임스탬프 | 프로젝트 이름. |
| `--format FMT` | `TIFF (16-bit)` | `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)` 중 하나. |
| `--indices NAME [NAME ...]` | 없음 | 내보낼 식생 지수 ([식생 지수](#vegetation-indices) 참조). |
| `--debayer {standard,texture-aware}` | `standard` | `texture-aware` = 신경망 디베이어, 처리 속도는 느리지만 최고 품질 (Chloros+, NVIDIA GPU). |
| `--vignette / --no-vignette` | 켜짐 | 비네팅 보정. |
| `--reflectance / --no-reflectance` | 켜짐 | 반사율 보정; LATTICE의 경우 이는 반사율 산출 기능 토글이기도 합니다. |
| `--input-level {auto,raw,debayered,processed}` | `auto` | LATTICE TIFF 파일의 파이프라인 진입점을 강제 설정. |

그 외 모든 항목(타겟 감지 조정, PPK, 노출 핀, 어레이 정렬 플래그)에 대해서는 [CLI 참조 문서의 `process` 섹션](reference/cli-reference.md)의 [`process` 섹션]을 참조하십시오.

***

## 내보낼 항목 선택 (LATTICE 제품)

LATTICE 처리는 **한 번의 패스로 해당되는 모든 제품에**분산됩니다. 제품별 4개의 토글은 모두**기본적으로 켜져** 있습니다. `--no-` 양식을 사용하여 하나를 해제하십시오:

| 토글 | 제품 |
| --- | --- |
| `--debayered` | 선형 디모자이크 → `Debayered_Images/` |
| `--preview` | 미리보기 표시 (화이트 밸런스 + 감마; 다중 스펙트럼용 가색 확장) → `Preview_Images/` |
| `--radiance` | float32 방사도, W/m²/sr/nm → `Radiance_Images/` (항상 `tiff32/`) |
| `--reflectance` | uint16 반사율, Pix4D 호환 → `Reflectance_Calibrated_Images/` |

RGB 마스터 카메라는 항상 디베이어링 처리된 값과 미리보기만 출력합니다. 광대역 센서의 경우 대역별 복사도/반사율은 의미가 없으므로, 해당 토글은 이 카메라들에 대해 아무런 동작을 하지 않습니다. Survey3 `.raw`는 토글 설정을 무시하고 표준 반사율/타겟 경로를 따릅니다.

```bash
# Radiance only — no DAQ downwelling needed
chloros-cli process ~/captures/lattice_flight --no-debayered --no-preview --no-reflectance
```

**`--reflectance-source {auto,target,daq}`**(기본값 `auto`)는 반사율 기준점을 선택합니다: `auto`는 QA 기준을 통과하는 프레임 내 [보정 타겟](calibration-targets.md)를 절대 기준점으로 선택하며, 타겟이 없을 경우 DAQ 광센서의 하향 복사 분할값(ρ = π·L/E)으로 대체됩니다; `target`는 엄격한 모드(DAQ 대체 없음)입니다; `daq`는 DAQ를 우선시합니다. 단위별 측정된 타겟 스캔은 `--target-reflectance-dir`를 통해 제공될 수 있습니다.

{% hint style="info" %}
**반사율 픽셀 읽기:**ρ = 1.0을 의미하는 DN은**소스별** 값입니다 — LATTICE 파일은 XMP에 `Chloros:PixelScale=32768`를 기록합니다; Survey3 파일은 65535를 사용하며(`Chloros:*` 태그를 포함하지 않습니다). 상수를 가정하기보다는 태그 값을 읽어 그 값으로 나누십시오. 자세한 내용과 의도적으로 설정된 비스케일(no-scale) 경계 사례 하나는 [CLI 참조](reference/cli-reference.md)에 나와 있습니다.
{% endhint %}

**처리는 항상 `raw`에서 시작됩니다.** 파생 제품(디베이어링/방사도/반사도 내보내기)은 파이프라인을 통해 다시 입력되지 않습니다. 이를 다시 가져와 처리하면 보정 계산이 이중으로 적용되므로, Chloros는 이를 건너뛰며 해당 사실을 명시합니다. `--input-level`는 진정으로 진입점을 강제로 지정해야 할 때를 대비해 의도적으로 마련된 비상 탈출구입니다.***

## 실행이 실패할 때

1.2.0 버전부터 `process`는 결과물을 전혀 표시하지 않은 채 “성공”한 것처럼 보이는 대신 명확하게 실패를 알립니다:

* **제품을 요청했으나 아무것도 기록하지 않은**실행 — `project.json` 및 `calibration_data.json`만 해당 — 은 `Processing finished but wrote no image products.`를 출력하고**0이 아닌 값으로 종료**되어, 따라서 스크립트에서 이를 감지할 수 있습니다. 일반적인 원인은 다음과 같습니다: 입력 폴더가 캡처로 인식되지 않았거나(레이아웃 및 `--input-level` 확인), 요청된 모든 제품이 해당 카메라에 적용할 수 없는 경우입니다(예: RGB 전용 카메라에서 방사도/반사도 요청).
* **의도적으로 메타데이터만 처리하는 실행**(모든 제품 토글 끄기, `--indices` 미사용)은 여전히 성공으로 간주됩니다. 이 경우 빈 이미지 출력이 올바른 결과입니다.
* `--verbose`를 사용하여 다시 실행하고, 백엔드 로그에서 `[LATTICE-EXPORT]` / `[EXPORT-CHECK]` 행을 확인하십시오. 이 행들은 카메라별 생략 사유를 설명합니다.

종료 코드: `0` 성공 · `1` 일반 오류 · `2` 인수 오류 · `130` Ctrl+C로 중단됨.

***

## 식생 지수

`--indices`에 하나 이상의 사전 설정 이름을 지정하여 실행하면, 각 지수는 별도의 `<INDEX>_Index_Images/` 폴더에 저장됩니다:

```bash
chloros-cli process ~/images/flight_001 --indices NDVI NDRE GNDVI
```

`process --indices`가 허용하는 22개의 사전 설정 이름:

`NDVI` `GNDVI` `NDRE` `OSAVI` `SAVI` `MSAVI2` `EVI` `MSR` `TDVI` `LAI` `GCI` `GRVI` `GSAVI` `GOSAVI` `NLI` `MNLI` `RDVI` `WDRVI` `CVI` `ENDVI` `GLI` `VARI`

{% hint style="warning" %}
**세 개의 인덱스 목록이 존재하므로 혼동하지 마십시오.**GUI의 ‘프로젝트 설정’ 드롭다운 메뉴에는 27개의 수식이 있습니다(`FCI1`, `FCI2`, `GARI`, `GEMI`, `LCI` — 이 다섯 개는 GUI 전용이며 `--indices`에는**적용되지** 않습니다). 라이브/오프라인 `lattice index --preset` 명령은 별도의 22개 프리셋 목록을 사용합니다. 공식 및 밴드 계산 방법은 [다중 스펙트럼 지수 공식](project-settings/multispectral-index-formulas.md)에 설명되어 있습니다.
{% endhint %}

***

## DAQ 광 센서: 간략한 소개

`daq pool-*` 제품군은 백엔드의 영구 풀(GUI, CLI 및 SDK)을 통해 MAPIR DAQ 스펙트럼 센서(USB를 통한 DAQ-U, BLE 기반 DAQ-M, 이더넷 기반 DAQ-E)를 구동하며, 백엔드의 영구 풀을 통해 제어됩니다. 즉, GUI, CLI 및 SDK는 모두 하나의 활성 핸들을 공유합니다. **`pool-*`는 제공된 CLI에서 지원되는 DAQ 경로입니다**; 참조될 수 있는 다른 `daq` 하위 명령어들은 MAPIR 내부 소스 전용 인터페이스이며, `pool-*`를 가리키는 명시적인 오류 메시지와 함께 종료됩니다.

```bash
# 1. Open a pooled session (pick the line matching your sensor)
chloros-cli daq pool-connect                              # smart-detect
chloros-cli daq pool-connect --port COM3                  # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF      # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-xxx.local   # DAQ-E by hostname (reliable)

# 2. List pooled sensors and their ids
#    (DAQ-U ids look like 'CB-7C-A8-2E-5F'; DAQ-E ids like 'daq-e-def330')
chloros-cli daq pool-list

# 3. Read the latest calibrated spectrum (W/m²/nm)
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F

# 4. Record a calibrated .daq file for 60 s
chloros-cli daq pool-record --sensor-id CB-7C-A8-2E-5F --duration 60 \
  -o ~/Documents/spectra --device-name "field-A"

# 5. Release
chloros-cli daq pool-disconnect --sensor-id CB-7C-A8-2E-5F
```

`pool-record`는 `--duration`가 없으면 `pool-record --stop`까지 실행됩니다; 기본 출력 디렉터리는 **백엔드 시스템에서** `~/Documents/DAQ Live View/`입니다. 캡 보정 프로필은 연결 시점에 선택되며(`--cap-id`, 백엔드 기본값 `sunshine_cosine`), `pool-set-cap` — 캡 프로파일 및 센서의 보정 범위에 대해서는 본 매뉴얼의 DAQ 장에서 다루고 있습니다.

{% hint style="warning" %}
**다중 NIC 호스트에서의 DAQ-E:** 부팅 후 처음 수행되는 `pool-connect --eth` 자동 검색은 센서가 정상 상태일지라도 실패할 수 있습니다. `--eth-host <ip-or-hostname>`는 안정적인 방식이므로, 검색 결과가 없을 때마다 이 방식을 사용하십시오.
{% endhint %}

***

## LATTICE 카메라, PTP 및 프로젝트 자동화

`lattice` 계열(45개 이상의 하위 명령어)은 탐색, 단일 캡처, GUI의 스마트 준비(smart-prep) 연결 흐름을 통한 지속적인 동기화 배열, 라이브 브라우저 미리보기, 정렬, 인덱스 계산 및 호스트 NIC 진단에 이르기까지 LATTICE 카메라 작업을 종단 간(end-to-end)으로 지원합니다. 예시:

```bash
chloros-cli lattice info                                          # discover cameras
chloros-cli lattice capture -o output/                            # one frame, all export types
chloros-cli lattice array-connect --serials SN1,SN2,SN3,SN4       # persistent synced array
chloros-cli lattice array-capture --processing reflectance -o out/
```

이와 함께: `chloros-cli time-sync`는 Chloros 호스트가 실행하는 PTP 그랜드마스터에 대한 정보를 보고하며(LATTICE 카메라와 DAQ-E 센서는 기기 간 타임스탬프를 위해 이 그랜드마스터에 슬레이브로 연결됨), 또한 `chloros-cli project`는 저장된 Chloros 프로젝트를 열어, 스크립트로 작성된 YAML 캡처 레시피를 포함하여 카메라, 어레이 및 센서를 헤드리스 모드로 제어합니다.

이 세 가지 계열(`lattice`, `project`, `daq pool-*`)는 또한 **원격** 백엔드를 제어하기 위한 `CHLOROS_BACKEND_URL`를 지원하는 유일한 제품군입니다. 핵심 명령어는 항상 로컬 머신을 대상으로 합니다.

자세한 단계별 안내는 이 매뉴얼의 LATTICE 장에 수록되어 있으며, 모든 플래그에 대한 설명은 [CLI 참조](reference/cli-reference.md)에서 확인할 수 있습니다.

***

## 문제 해결: 상위 5가지

| 증상 | 해결 방법 |
| --- | --- |
| `Login required` 또는 예약된 작업이 `Email:` 프롬프트에서 멈춤 | 이 컴퓨터에서 `chloros-cli login EMAIL 'PASSWORD'`를 한 번 실행하십시오. 캐시된 세션 프롬프트가 없는 명령어는 즉시 실패하지 않고 대화형 모드로 실행됩니다. |
| `backend unreachable` | Chloros 데스크톱 앱을 시작하거나 백엔드 바이너리를 직접 실행하십시오(`chloros-backend`). `lattice`/`project`/`daq pool-*`를 원격 백엔드로 지정하는 경우, `CHLOROS_BACKEND_URL`를 확인하십시오. |
| 배열 연결 차단됨: `FRAMES WILL DROP` / `Reduce ROI to enable` | 호스트 NIC 수신 링이 기본값으로 재설정됨 — 일반적으로 NIC 드라이버 업데이트 후, 이전에 정상 작동하던 시스템이 연결을 거부하는 가장 흔한 원인입니다. **관리자 권한** 터미널에서 `chloros-cli lattice network --fix`를 실행하십시오(또는 `ReceiveBufferLen=256`, `PendingReceives=64`를 설정하십시오). 참조 문서의 *호스트 NIC 설정 및 튜닝*을 참조하십시오. |
| `daq` 하위 명령이 종료됨: &quot;전체 DAQ 패키지가 필요합니다…&quot; | 출하된 빌드에서 예상되는 현상 — 컴파일된 CLI는 연결, 스트림, 기록 및 캡처 선택을 다루는 `daq pool-*` 계열만 제공합니다. `pool-*`(또는 Python의 `chloros_sdk.connect_daq_sensor()`)를 사용하십시오. |
| Jetson은 대용량 폴더 처리 전에 스왑 경고를 표시합니다 | 파일 기반 스왑 추가 — CLI는 실행해야 할 정확한 `fallocate`/`swapon` 명령어를 출력합니다. |

***

## 도움말 보기

```bash
chloros-cli --help              # top-level help
chloros-cli process --help      # per-command help
chloros-cli lattice --help
chloros-cli daq --help          # lists the pool-* subcommands
```

* **모든 플래그, 모든 하위 명령:** [CLI 참조](reference/cli-reference.md)
* **Python와 동등한 명령:** [Python SDK](api-python-sdk.md) 및 [SDK 참조](reference/sdk-reference.md)
* **지원:** info@mapir.camera · [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
