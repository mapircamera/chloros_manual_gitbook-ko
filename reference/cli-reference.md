# Chloros CLI 참조

**버전:**

1.2.0**생성일:**2026-07-29 19:19 ·**수정일:** 2026-08-30**대상:** 대규모 언어 모델(LLM) 활용에 최적화되었으며, 사람이 읽기 쉬운 형식으로 작성되었습니다.**범위:** `chloros-cli`의 모든 사용자용 하위 명령어와 옵션, 복사하여 붙여넣을 수 있는 예제.

이 문서는 MAPIR Chloros에 포함된 `chloros-cli` 명령줄 도구에 대한 완전한 참조 자료입니다. 이 문서는 의도적으로 모든 내용을 망라하고 있으므로 LLM(또는 사람)이 소스 코드를 확인하지 않고도 아래 목록을 바탕으로 지원되는 모든 워크플로를 구성할 수 있도록 의도적으로 상세하게 작성되었습니다.

핵심 내용만 확인하시려면 다음으로 이동하세요:
- [5분 퀵스타트](#five-minute-quickstart)
- [LATTICE 카메라 첫 연결 워크플로우](#lattice-camera-first-connect-workflow)
- [DAQ 센서 첫 연결 워크플로우](#daq-sensor-first-connect-workflow)
- [Smart-AE / Smart-Capture](#smart-ae--smart-capture)
- [캡처 모드, 레코더 및 오프라인 재처리](#capture-modes-recorders--offline-reprocess)

---

## 명명 규칙

- 모든 명령어의 접두사는 `chloros-cli`입니다. Windows에서는 바이너리 이름이 `chloros-cli.exe`이며, Linux /Jetson에서는 `chloros-cli`입니다.
- 선택적 인수는 는 `--flag` 형식으로 표시됩니다. 필수 위치 인수는 괄호 없이 표시됩니다.
- 기본값이 지정된 경우, 플래그를 생략하면 해당 값이 사용됩니다.
- CLI은 Chloros 백엔드(`127.0.0.1:5000`의 Flask 서버)를 기반으로 하는 경량 HTTP 클라이언트입니다. 대부분의 명령어를 실행하면 백엔드가 자동으로 시작됩니다. `CHLOROS_BACKEND_URL=<url>`는 원격 백엔드에서 **`lattice`**,**`project`**및**`daq pool-*`** 명령어 계열을 원격 백엔드로 연결합니다. 핵심 명령어(`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) `http://127.0.0.1:<port>`를 의도적으로 고정하고 무시합니다(IPv4 리터럴을 사용하면 Windows의 `localhost`→`::1` 요청당 약 2초의 성능 저하를 피할 수 있습니다). [환경 변수](#environment-variables)를 참조하십시오.
- 모든 SDK / CLI 호출에는 Chloros+ 계정 로그인이 필요합니다(머신당 한 번씩 `chloros-cli login`를 실행하십시오; `~/.chloros/`에 캐시됨).
- 예제에서는 Linux 경로를 사용합니다. Windows에서는 `/home/user/...`를 `C:/Users/.../...`로 대체하십시오.

---

## 최상위 개요

```
chloros-cli [global options] COMMAND [command options]
```

### 전역 옵션

| 플래그 | 설명 |
| --- | --- |
| `--backend-exe PATH` | 자동 감지된 백엔드 실행 파일을 재정의합니다. |
| `--port N` | 백엔드 HTTP 포트(기본값: `5000`). |
| `-v, --verbose` | 상세 출력 활성화. |
| `--restart` | 백엔드를 강제 재시작합니다(실행 중인 모든 `backend_server.py`를 종료함). |
| `--version` | 버전 정보 출력(`Chloros CLI 1.2.0`). |
| `--help` | 최상위 도움말 표시. |

### 명령어 색인

| 명령어 | 용도 |
| --- | --- |
| [`process`](#chloros-cli-process) | Survey3 또는 LATTICE 캡처 파일이 포함된 폴더를 종단 간으로 처리합니다. |
| [`login`](#chloros-cli-login) | Chloros+ 계정으로 이 컴퓨터에 인증합니다. |
| [`logout`](#chloros-cli-logout) | 캐시된 자격 증명을 지웁니다. |
| [`status`](#chloros-cli-status) | 현재 라이선스/인증 상태를 표시합니다. |
| [`export-status`](#chloros-cli-export-status) | `process` 실행 중 Live Thread-4 내보내기 진행 상황. |
| [`language`](#chloros-cli-language) | CLI 표시 언어 설정 또는 목록 확인 (38개 언어 지원). |
| [`set-project-folder`](#project-folder-commands) / [`get-project-folder`](#project-folder-commands) / [`reset-project-folder`](#project-folder-commands) | 기본 프로젝트 폴더(GUI와 공유). |
| [`update`](#chloros-cli-update) | CLI 업데이트 확인 및 설치(Linux /Jetson). |
| [`selftest`](#chloros-cli-selftest) | 시스템 진단 및 스모크 테스트. |
| [`time-sync`](#chloros-cli-time-sync) | PTP 그랜드마스터 상태 / 제어. |
| [`lattice`](#chloros-cli-lattice) | LATTICE 카메라 제어 및 캡처 (45개 이상의 하위 명령어). |
| [`daq`](#chloros-cli-daq) | DAQ 스펙트럼 센서 제어 (DAQ-U / DAQ-M / DAQ-E). |
| [`project`](#chloros-cli-project) | 저장된 Chloros 프로젝트(카메라 + DAQ) 열기 및 실행. |

---

## 설치

`chloros-cli`는 지원되는 모든 플랫폼의 Chloros 데스크톱 설치 프로그램에 포함되어 제공됩니다 — 별도의 CLI 다운로드는 제공되지 않습니다. 플랫폼 패키지를 설치하면 데스크톱 앱 및 이를 구동하는 백엔드 바이너리와 함께 `chloros-cli`가 `PATH`에 추가됩니다.

최신 다운로드: [`https://mapir.gitbook.io/chloros/download`](https://mapir.gitbook.io/chloros/download)

> 설치 프로그램에는 즉시 사용 가능한 바로 사용할 수 있는 CLI 셸을 실행하는 편리한 런처 스크립트(`Chloros_CLI.bat` / `Chloros_CLI.ps1`, `Launch_CLI.*`, `chloros-cli.sh`)도 함께 제공됩니다. 이에 대한 내용은 [CLI 사용자 가이드](../CLI.md)에 설명되어 있으므로 여기서는 중복하여 다루지 않습니다.

### Windows (.exe)

1. 다운로드 페이지에서 Windows 설치 프로그램을 다운로드하십시오.
2. `Chloros-Setup-x.y.z.exe`를 실행하고 마법사의 안내에 따라 진행하십시오. 기본 설치 경로는 `C:\Program Files\Chloros\`입니다(CLI는 `C:\Program Files\Chloros\cli\`에 생성되며, 설치 프로그램이 이 경로를 PATH에 추가합니다).
3. 업데이트된 `PATH`가 인식되도록 새 터미널(`cmd.exe`, PowerShell 또는 Windows 터미널)을 엽니다.

```powershell
chloros-cli --version
```

설치 프로그램은 `chloros-cli.exe`를 시스템 `PATH`에 자동으로 추가하며, LATTICE 카메라에 필요한 Arena SDK 런타임을 함께 포함합니다.

### Linux amd64 (.deb)

Ubuntu 22.04 LTS 이상 또는 Debian 기반 x86_64 워크스테이션용입니다.

> **Ubuntu 20.04는 지원되지 않습니다.** 이 패키지의 의존성 목록은
> 백엔드가 실제로 링크하는 대상에서 파생된 것이며, 여기에는 `libc6 (>= 2.34)`가 포함됩니다.
> focal은 glibc 2.31을 제공합니다. `apt`는 런타임 시 오류가 발생하도록 내버려 두기보다는
> 설치를 거부합니다.

```bash
sudo dpkg -i chloros-amd64.deb
sudo apt-get install -f         # only if dpkg reports missing dependencies
chloros-cli --version
```

이 .deb 패키지는 다음을 설치합니다:
- `chloros-cli` → `/usr/bin/chloros-cli`
- 컴파일된 백엔드 → `/usr/lib/chloros/chloros-backend`
- Arena SDK 런타임 (LATTICE 카메라용)
- Denoiser 모델, 보정 번들 및 업데이트 채널 구성

### Linux arm64 — Jetson (JetPack 6)

```bash
sudo dpkg -i chloros-arm64-jp6.deb
sudo apt-get install -f
chloros-cli --version
```

amd64 .deb 파일과 동일한 레이아웃이며, Jetson Orin / Orin NX / Orin Nano에 최적화된 CUDA 빌드가 포함되어 있습니다.

### 기기당 1회 인증

모든 플랫폼에서 SDK / CLI 호출이 작동하려면 Chloros+에 한 번 로그인해야 합니다:

```bash
chloros-cli login user@example.com 'YourPassword'
```

인증 정보는 `~/.chloros/user_session.json`에 캐시됩니다.

### 설치 확인

```bash
chloros-cli --version           # prints "Chloros CLI 1.2.0"
chloros-cli selftest            # full 7-step diagnostic (backend, GPU, models, CUDA)
chloros-cli status              # shows license tier + logged-in user
```

> **Chloros+ 구독이 필요합니다.**CLI를 사용하려면 유효한 Chloros+ 요금제가 필요합니다.**Copper**는 입문용 Chloros+ 등급입니다. 모든 유료 Chloros+ 등급에는 CLI / SDK 접속 권한이 포함되며, 무료**Iron** 등급만 해당 권한이 없습니다. (요금제 ID 매핑: `0`=Iron/무료, `1`=Copper, `2`=Bronze, `3`=Silver, `4`=Gold.) 업그레이드 주소: [`https://cloud.mapir.camera/pricing`](https://cloud.mapir.camera/pricing)에서 업그레이드하십시오.
>
> 이 하한값은 CLI뿐만 아니라 백엔드에서도 강제 적용됩니다. 유료 플랜이 없는 상태에서 SDK / CLI 플래그가 지정된 요청은 `403 PLAN_UPGRADE_REQUIRED` 오류가 발생합니다. 이는 `chloros-cli`, Python, SDK 또는 직접 개발한 HTTP 클라이언트에서 발생하더라도 마찬가지입니다. 로그아웃된 호출자는 대신 `401 AUTH_REQUIRED` 오류를 받게 됩니다. 요금제의유예 기간(월간 플랜의 경우 30일, 연간 플랜의 경우 만료일까지) 동안 오프라인에서도 액세스가 가능하며, 해당 기간이 만료되면 중단됩니다. `chloros-cli status`는 계속 작동하므로 그 이유를 확인할 수 있습니다(이는 티어 게이트(tier gate)에서 면제되는 유일한 경로인 SDK / CLI 경로이며, `GET /api/license-status`에 해당합니다).

---

## 5분 퀵스타트

```bash
# 1. Sign in once on this machine
chloros-cli login user@example.com 'YourPassword'

# 2. Survey3 / LATTICE folder → finished radiance + NDVI in one call
chloros-cli process "/home/user/captures/flight_001" \
  --vignette --reflectance --indices NDVI NDRE GNDVI

# 3. Take a single LATTICE photo with the first camera found
chloros-cli lattice capture -o output/

# 4. Connect a 4-cam LATTICE array with the GUI's smart-prep flow
chloros-cli lattice array-connect \
  --serials 213800234,214000533,214701288,214701292

# 5. Read a spectrum from a connected DAQ-U
chloros-cli daq pool-connect --port COM3
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F   # id from 'daq pool-list'
```

---

## `chloros-cli process`

이미지가 포함된 폴더를 전체 Chloros 파이프라인(타겟 감지 → 보정 → 비네트 → 반사율 → 인덱스 내보내기)을 통해 처리합니다.

### 개요

```
chloros-cli process INPUT [OPTIONS]
```

### 위치 매개변수

| 인수 | 설명 |
| --- | --- |
| `INPUT` | `.raw + .jpg`(Survey3), `.tif/.tiff`(LATTICE) 또는 `.dng` 파일이 포함된 입력 폴더의 경로. |

### 일반 옵션

| 플래그 | 기본값 | 설명 |
| --- | --- | --- |
| `-o, --output PATH` | 기본 프로젝트 경로 아래에 타임스탬프가 포함된 새 폴더(별도 구성되지 않은 경우 `~/Chloros Projects`) | 생성하거나 재사용할 프로젝트 폴더입니다. 해당 폴더에 이미 `project.json` 파일이 있는 경우, `_1`/`_2` 형제 폴더가 생성되며, 기존 폴더는 덮어쓰지 않습니다. |
| `-n, --project-name NAME` | 자동(타임스탬프) | 프로젝트 이름. |
| `--debayer {standard,texture-aware}` | `standard` | `texture-aware`는 Chloros+ 신경망 디베이어를 사용합니다. 속도는 느리지만 품질은 더 높습니다. |
| `--vignette / --no-vignette` | `--vignette` | 비네팅 보정. |
| `--reflectance / --no-reflectance` | `--reflectance` | 반사율 보정 (패널 타겟이 발견되면 이를 사용하며, LATTICE의 경우 일련번호별 NIST 보정값을 사용). LATTICE 다중 스펙트럼의 경우, 이 설정은 반사율 **곱셈** 토글 역할도 겸합니다 — [제품별 내보내기 토글](#per-product-export-toggles-lattice-multispectral) 참조). |
| `--ppk` | off | 사이드카 파일의 PPK GNSS 보정 적용. |
| `--exposure-pin-1 MODEL` | off | Survey3 듀얼-카메라 리그의 “pin-1” 모델을 고정합니다. |
| `--exposure-pin-2 MODEL` | off | &quot;pin-2&quot; 모델을 고정합니다. |
| `--recal-interval SECONDS` | 0 | 캡처 시간 기준 N초마다 보정 계산을 강제 재실행합니다. |
| `--timezone-offset HOURS` | local | 출력 메타데이터에 내장된 시간대 오프셋을 재정의합니다. |
| `--format FORMAT` | `TIFF (16-bit)` | `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)` 중 하나. |
| `--indices NAME [NAME ...]` | 없음 | 식생 지수 (`NDVI`, `NDRE`, `GNDVI`, `EVI`, `SAVI`, `OSAVI`, `CIG`, …). |
| `--input-level {auto,raw,debayered,processed}` | `auto` | LATTICE TIFF의 파이프라인 진입점을 강제 적용합니다 (Survey3 .raw는 영향을 받지 않음). 또한 **RAW가 없는** 캡처를 처리할 수 있게 해주는 비상구 — [캡처 폴더의 구조](#what-a-captures-folder-looks-like)를 참조하십시오. |
| `--debayered / --no-debayered` | on | 선형 디베이어링 결과물(`Debayered_Images`) 출력. [제품별제품별 내보내기 토글](#per-product-export-toggles-lattice-multispectral) 참조. |
| `--preview / --no-preview` | 켜짐 | 디스플레이 미리보기를 출력합니다(`Preview_Images`): RGB = 화이트 밸런스(DAQ-illumin사용 가능한 경우, 그렇지 않으면 그레이 월드) + 감마; multispec = 가색 변환. |
| `--radiance / --no-radiance` | on | float32 형식의 복사도 출력 (`Radiance_Images`, W/m²/sr/nm). |
| `--reflectance-source {daq,target,auto}` | `auto` | LATTICE 반사율 산출값에 대한 기준: `auto` = QA 검증을 통과한 프레임 내 목표값이 절대 기준이며, DAQ 하향 복사 (ρ = π·L/E) 대체 기준; `target` = 엄격한 기준(DAQ 대체 없음); `daq` = DAQ 기준 우선. [제품별 내보내기 토글](#per-product-export-toggles-lattice-multispectral)를 참조하십시오. |
| `--target-reflectance-dir DIR` | 없음 | 단위별 **측정된** 대상 반사율 스캔의 디렉터리 (`<serial>.csv`); 누락 시 명목상 T3/T4P 스펙트럼으로 대체됩니다. |
| `--array-alignment / --no-array-alignment` | on | LATTICE 어레이: 각 캡처의 `Chloros:Alignment*` XMP에 기록된 모듈 간 정렬 정보를 모든 처리된 제품(디베이어링/미리보기/방사도/반사율/색인)에 적용합니다. 태그가 없는 이미지의 경우 아무 작업도 수행하지 않습니다. |
| `--array-alignment-crop / --no-array-alignment-crop` | crop | 정렬된 내보내기 데이터를 어레이의 공통 중첩 영역으로 자르어 모든 모듈이 하나의 풋프린트를 공유하도록 합니다. `--no-…`는 전체 센서 캔버스를 유지합니다(소스 외부는 검은색으로 채움). |
| `--array-alignment-interp {bilinear,nearest,cubic}` | `bilinear` | 정렬 왜곡을 위한 리샘플링. `nearest`는 원본의 정확한 디지털 노벨리티(DN)를 보존합니다(방사계 값의 픽셀 간 혼합 없음). |

### 타겟 감지 옵션

| 플래그 | 설명 |
| --- | --- |
| `--min-target-size PIXELS` | 감지기를 위한 최소 패널-타겟 크기(px). |
| `--target-clustering 0-100` | 클러스터링 감도. |
| `--target / --targets` | 입력 폴더를 타겟 패널 전용으로 처리(서베이 탐지 건너뛰기). |

### 예시

```bash
# Simplest: defaults are good for most surveys
chloros-cli process "/home/user/images/survey_001"

# Multi-index with explicit format
chloros-cli process "/home/user/images/survey_001" \
  --vignette \
  --reflectance \
  --format "TIFF (32-bit, Percent)" \
  --indices NDVI NDRE GNDVI OSAVI

# Texture-aware debayer for highest quality (Chloros+ only)
chloros-cli process "/home/user/images/survey_001" \
  --debayer texture-aware \
  --indices NDVI

# Process LATTICE captures explicitly (auto-detects from EXIF normally)
chloros-cli process "/home/user/captures/lattice_flight" \
  --input-level processed

# LATTICE multispectral → float32 radiance only (no DAQ downwelling needed)
chloros-cli process "/home/user/captures/lattice_flight" \
  --no-debayered --no-preview --no-reflectance

# LATTICE reflectance anchored to an in-frame target (strict, no DAQ fallback),
# with per-unit measured target scans looked up by serial
chloros-cli process "/home/user/captures/lattice_flight" \
  --reflectance-source target --target-reflectance-dir "/home/user/target_scans"

# LATTICE array capture: keep native geometry (ignore stamped alignment)
chloros-cli process "/home/user/captures/array_flight" \
  --no-array-alignment

# Aligned, uncropped, value-preserving resampling
chloros-cli process "/home/user/captures/array_flight" \
  --no-array-alignment-crop --array-alignment-interp nearest

# Save to a custom output location with a project name
chloros-cli process "C:/input" -o "C:/output" -n "Field_A_2026-05-26"
```

### 제품별 내보내기 토글 (LATTICE 다중 스펙트럼)

LATTICE 처리는 **한 번의 처리 과정으로 적용 가능한 모든 제품**으로 분기됩니다. 유형별 4가지 토글 — `--debayered`, `--preview`, `--radiance`, `--reflectance` —는 모두**기본적으로 켜져** 있습니다. 하나를 비활성화하려면 `--no-<type>` 형식을 사용하십시오. RGB 마스터 캠은 항상 디베이어 처리된 데이터와 미리보기 데이터만 출력하며(대역별 방사도/반사율 데이터 없음), 따라서 `--radiance`/`--reflectance`는 해당 카메라에 대해 아무런 동작도 하지 않습니다. Survey3 `.raw`(표준 반사율/타겟 경로를 따름)의 경우, 이 토글 설정은 무시됩니다. *(기존의 `--radiometric-output {reflectance,radiance,sensor-response}` 플래그는 **제거**되었으며 이 토글들로 대체되었습니다. 더 이상 `sensor-response` 레벨은 존재하지 않습니다.)*

| 제품 | 출력 | DAQ 하향 복사 필요? |
| --- | --- | --- |
| `--debayered` | 선형 디모자이크 (`Debayered_Images`). | 아니요. |
| `--preview` | 미리보기 표시 (`Preview_Images`): RGB = WB + 감마; multispec = 가색 변환. | 아니요. |
| `--radiance` | float32 W/m²/sr/nm (`Radiance_Images`). | 번호 |
| `--reflectance` | uint16 반사율 ρ (`32768` = 1.0), Pix4D 호환. | **예**, QA를 통과한 프레임 내 타겟이 이를 고정하는 경우는 제외 (아래 참조). |

`--reflectance-source`는 반사율 기준값을 선택합니다:**`auto`**(기본값) QA를 통과한 프레임 내 타겟을**절대 기준**으로 설정 — 타겟에 고정된 경험적 라인 체인은 제외된 패널에서 교차 점수화되며, 측정된 최상값이 적용됩니다 — 타겟이 없거나 QA에 실패할 경우 DAQ 하향 분할(ρ = π·L/E)로 되돌아가며;**`target`**는 엄격한 모드(DAQ 대체 없음)이며;**`daq`**는 DAQ 우선 동작으로 전환합니다. 타겟 기하학적 구조 (ArUco / 고정 ROI / 스트립)은 프로젝트 타겟 구성에서 가져옵니다; `--target-reflectance-dir DIR`는 타겟 유닛의 일련번호/QR로 조회된 단위별**측정된** 스캔(`<serial>.csv`)을 보관하며, 명목상 T3/T4P 스펙트럼을 대체값으로 사용합니다.

DAQ 반사율 경로는 기록된 **`.daq`**(DAQ-U/M/E)**또는 이미지와 함께 발견된 DAQ-M 네이티브 `.csv`**에서**타임스탬프가 일치하는 하향파**를 자동으로 식별합니다. 카메라별 또는 DAQ 보정 번들이 로컬에 캐시되어 있지 않은 경우, 파이프라인은 첫 사용 시**AWS에서 이를 자동으로 가져옵니다** (인터넷 연결이 한 번 필요하며, `~/.chloros/`로 캐시됨).

#### 반사율 픽셀 읽기 (Pix4D / Metashape / 사용자 스크립트)

반사율은 정수형 DN으로 저장되며, **ρ = 1.0을 나타내는 DN은 소스 카메라에 따라 다릅니다**:

| 소스 | ρ = 1.0에 해당하는 값 | 확인 방법 |
| --- | --- | --- |
| LATTICE (M3C / M3M) | `32768` (ρ 2.0까지의 여유 범위) | 파일에 XMP `Chloros:PixelScale=32768`가 찍혀 있습니다. |
| Survey3 | `65535` (ρ 1.0에서 잘림) | `Chloros:*` XMP 태그 없음 — 그 부재 자체가 신호입니다. |

**`Chloros:PixelScale` 값을 읽고 그 값으로 나누십시오**. 상수를 가정하지 마십시오. 이 태그는 uint16 영역으로 정의되어 있으므로, 비율을 조정하는 출력 형식(`32768`) 전반에 걸쳐 `32768`로 유지됩니다 — `TIFF (16-bit)`, `PNG (8-bit)`, `JPG (8-bit)` 및 `TIFF (32-bit, Percent)`는 모두 자체 설명형입니다(저장된 데이터 유형을 먼저 uint16으로 정규화해야 합니다: 8비트에서 ×257, 부동 소수점에서 ×65535).

> **설계상 스케일이 적용되지 않는 경우가 하나 있습니다.** 8비트 소스 캡처(BayerRG8)가 8비트TIFF로 기록될 때, 파이프라인은 재스케일링 대신 0..255 범위로 *클리핑*하므로, ρ≈0.008을 초과하는 모든 값은 255로 평탄화되고, 해당 파일에는 스케일이 기술되지 않습니다. Chloros 해당 부분에서 `Chloros:PixelScale`와 `MicaSense:RadiometricCalibration` 튜플을 의도적으로 생략하고, 그 이유를 기록합니다. **LATTICE 반사율 파일에 해당 태그가 없는 경우, 스케일이 없다고 가정하지 마십시오 — 16비트 또는 32비트로 재내보내십시오**. 나눌 수 없었던 픽셀을 억지로 나누려고 시도하지 마십시오.

#### 내보내기 시 전달되는 EXIF

`process`는 원본 캡처의 **GPS 블록과 해당 ExifIFD**를 모든 산출물에 복사하므로,
내보낸 파일에는 `FocalLength`, `FNumber`, `ExposureTime`, `ISO`, `DateTimeOriginal` 및
`CameraSerialNumber`가 지리 참조 정보와 함께 포함됩니다.

**`FocalLength`는 사진측량에 있어 선택 사항이 아닙니다.** Pix4D는 초점 거리와 고도를 바탕으로
지상 샘플 거리를 계산합니다; 이 태그가 없으면 극도로 잘못된 축척으로 처리됩니다. 한 번은
49회 촬영한 오렌지 과수원 비행에서 이 태그가 누락되어 411m × 160m 규모의 부지가 재구성 결과
47.8km × 13km 규모로 변해버렸습니다. 이는 대부분 ‘데이터 없음’으로 표시된 455MP의 정사사진이었으며, 이는 GSD를 확인하기 전까지 타일링 문제나
BigTIFF 문제로 오인되었습니다. 정사영상이 비현실적인
축척으로 출력된다면, 먼저 내보낸 결과물에 `exiftool -FocalLength`를 적용해 보십시오.

이 사본은 의도적으로 **`-all:all`**가 아닙니다. IFD0의 구조 태그는 복사 시 LATTICE 출력을
손상시키며, `ExifImageWidth` / `ExifImageHeight`는 *소스* 캡처를 기술하기 때문에 제외되었습니다.
크기가 조정된 적이 있는 내보내기 파일은 그렇지 않으면 자체 래스터와 모순되는 치수를
가지고 있게 될 것입니다. XMP는 복사하는 대신 직접 작성됩니다. ExifTool은
XMP 블록이 복사될 때 동일한 호출의 XMP 태그를 제거하기 때문입니다(이 경우 MAPIR
보정 태그가 누락될 수 있습니다).

### 출력 파일의 저장 위치

출력 파일은 **프로젝트 폴더 아래, 카메라별로, 그다음 파일 형식별로 그룹화되어** 저장됩니다:

```
<project>/
└── LATT-M3M-L41-F550/                  # one folder per camera model+lens+filter
    ├── tiff16/
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── <INDEX>_Index_Images/        # e.g. NDVI_Index_Images
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

LATTICE의 카메라 폴더는 `LATT-<sensor>-<lens>-F<filter>`이며(캡처 파일의 EXIF
`Model`와 일치), Survey3의 경우 `<model>_<filter>`입니다. 센서와 필터를 공유하지만 렌즈가 다른 두 대의 카메라는
비네팅, 시야각 및 왜곡이 다르기 때문에 별도의 폴더 구조로 관리됩니다. 포맷
폴더 구조는 `--format`: `tiff16`, `tiff8`, `png8`, `jpg8`, 또는 `tiff32`의 경우
`TIFF (32-bit, Percent)`와 같은 형식을 따릅니다.

> **내보낸 모든 파일은 원본 파일의 이름을 유지합니다.**
> `capture_…_raw.tif`의 Radiance 내보내기 파일은 여전히 `capture_…_raw.tif`라고 불리지만,
> `tiff32/Radiance_Images/` 폴더에 위치합니다. **폴더가 제품을 식별하며, 파일 이름이 아닙니다**. 따라서 `*radiance*.tif`를
> 글로빙(globbing)으로 검색하면 아무것도 찾지 못합니다. 대신 디렉터리를 기준으로 일치 여부를 확인하십시오.

### 광센서 기록 — 보정된 `.daq` + `.csv`

`process`는 입력 폴더에 있는 `.daq` 기록 데이터도 처리하며, 이를 수행하는 데 **별도의**
이미지가 필요하지 않습니다. 단독으로 비행한 DAQ-U / DAQ-M / DAQ-E의 기록만으로도 완전한
캡처 장치이며, `.daq` 파일만 포함된 폴더도 유효한 입력입니다.

DAQ는 **교정 없이도** — 이것이 바로 공개용
[`chloros_scripts`](https://github.com/mapircamera/chloros_scripts) 레코더
(`record_daq.py`)가 기본적으로 수행하는 방식입니다. 이 레코더들은 원시 센서 카운트 데이터를 기록하고 파일에 타임스탬프를 찍어
Chloros에서해당 센서의 공장 보정값을 **시리얼을 통해** (먼저 로컬 캐시,
그 다음 MAPIR 클라우드) 조회하여 적용합니다. `process`는 결과를 다시 기록합니다:

```
<project>/
└── Light Sensor/
    ├── <name>_calibrated.daq        # reprocessable archive, declares its bundle
    └── <name>_calibrated.csv        # W/m^2/nm per reading + photometric columns
```

`.csv`는 측정값 하나당 한 행을 포함합니다: UTC 타임스탬프, 적분 시간, 총 전력,
광시/암시 럭스, PPFD(및 청색/녹색/적색 분할값), 피크 파장, 그리고
센서자체 파장 그리드에 따른 전체 스펙트럼이 포함됩니다. `.daq`는
두 번째 보정 없이 다시 가져옵니다.

성공 시 실행 결과는 `Light-sensor products written: N (calibrated .daq + .csv)`로 보고됩니다.
괄호 안의 내용은 실제로 기록된 내용을 설명하므로 다음과 같이 읽힙니다:
번들이 없는 센서의 경우 `(RAW COUNTS — this sensor has no calibration bundle)`,
둘 다 포함된 폴더의 경우 `(N calibrated, M raw counts)`로 표시됩니다. 백엔드 고유의
`[DAQ-EXPORT]` 및 `[RUN-SUMMARY]` 헤더의 문구도 동일한 방식으로 도출됩니다. 이
세 가지 중 어느 것도 보정되지 않은 원시 내보내기 ‘보정됨’이라고 표기할 수 없습니다.

보정 번들을 가져올 수 없는 DAQ-U / DAQ-M / DAQ-E 기록(오프라인 상태이거나 해당 센서의 보정 정보가 파일에 없는 경우)은
**이유와 함께 건너뜁니다**.
`[DAQ-EXPORT]` 행에 **이유와 함께 건너뜁니다**. 절대 원시 카운트 값을 포함하는 &quot;보정된&quot; 파일로 저장되어 원시 카운트를 포함하는 일은 절대 없습니다.
인터넷에 연결한 후 다시 실행하십시오. 이유는 리더가 해당 파일에 대해 실제로
확인한 사유(읽을 수 없는 스키마, 번들 없음, 쓰기 오류)이며, 실행
요약에는 **서로 다른** 개의 사유만 나열됩니다 — 하나의 원인으로 인해 20개의 파일이 건너뛴 경우, 이는 하나의
원인으로 표시되며, 해당 원인이 20회 반복된 것으로 표시되지 않습니다.

#### DAQ-A 기록은 원시 카운트 값으로 내보내집니다

**DAQ-A** 제품군은 시리얼별 번들 시스템이 도입되기 이전에 출시되었으며, 가져올 교정 번들이
를 가져올 필요가 없습니다. 대신 현장에서 반사율 타겟을 기준으로 보정되므로
번들이 필요하지 않았던 것입니다. 이러한 기록을 거부하면 데이터를
전혀 추출할 방법이 없어지므로, **다른 이름**으로 내보내집니다:

```
<project>/
└── Light Sensor/
    ├── <name>_raw.daq        # NOT _calibrated
    └── <name>_raw.csv        # raw spectral sensor counts, NOT irradiance
```

파일 내부의 플래그가 아닌 별도의 파일명을 사용하는 이유는, 이 정보가
단순한 파일명 형태로 이메일을 통해 전송될 때도 유효해야 하기 때문입니다. `.csv` 헤더에는
`raw spectral sensor counts (NOT irradiance)`라고 명시되어 있으며, 값들은 파일 **내부**에서
비교 가능하다는 경고가 표시됩니다 — 이는 바로 타겟 기반 보정이 이를 사용하는 목적이며 —
센서 간이 아닌 파일 **내부**에서 비교 가능함을 경고합니다. 전력에 의존하는 광도 측정 열(총 전력, 광시 및
암시 럭스, PPFD)은 카운트 값을 통합한 것이 아니라 **NULL**로 기록되며, 실행
요약에는 `RAW COUNTS`라고 표시되므로 로그에 “내보내진” 값은 조도 값으로 해석될 수 없습니다.

구버전 **v1.01 / v1.02** 기록(DAQ-A-SD가 이를 기록함)은 읽기별 에포크를 포함하지 않고,
오직 파일의 기록 시간만 포함합니다. 이미지↔하향 복사량 매칭 도구는 여전히 이를 거부합니다 —
프레임을 기록 시간과 매칭하는 것은 눈에 띄지 않게 오류가 발생할 수 있기 때문 — 하지만 내보내기 도구는 이를 읽으며,
CSV는 `clock=daq_created_on`를 출력하므로, 결과물에서 어떤 시계를 사용 중인지 확인할 수 있습니다.

### 참고 사항

- `process`는 폴더가 ‘Survey3’, ‘LATTICE’ 또는 혼합형인지 자동으로 감지합니다.
- 진행 상황은 서버 전송 이벤트(Server-Sent Events)를 통해 스트리밍되며, CLI에서는 스레드별 실시간 진행 상황 (탐지, 분석, 처리, 내보내기).
- Linux /Jetson의 경우, CLI는 스왑을 확인하며 대용량 폴더를 처리하기 전에 경고를 표시할 수 있습니다. 텍스처 인식 디베이어는 저전력 Jetson(Nano, Orin Nano)에서 GPU 주파수 제한을 자동으로 적용합니다.
- 성공 시, 실행은 작성된 이미지 산출물의 수를 보고합니다(`Image products written: N`).

#### 이미지를 하나도 작성하지 않은 실행은 실패로 처리됩니다

결과물을 요청했으나 실행 과정에서 **아무것도** 작성하지 않고 — 오직 `project.json`와
`calibration_data.json`만 출력한 경우 — `process`는 이를 실패로 간주합니다:
`Processing finished but wrote no image products.`를 출력하고 **0이 아닌 값으로 종료**하므로, 스크립트에서 이를
감지할 수 있습니다. 메시지에는 프로젝트 폴더 이름과 일반적인 원인이 명시됩니다:

- 입력 폴더가 캡처로 인식되지 않았습니다(레이아웃 및 `--input-level`를 확인하십시오), 또는
- 요청된 모든 산출물이 해당 카메라에 적용 불가능하여 건너뛴 경우(예:
  RGB 전용 카메라에서 복사율/반사율 요청).

`--verbose`를 사용하여 다시 실행하고 백엔드 로그에서 `[LATTICE-EXPORT]` / `[EXPORT-CHECK]` 행이 있는지 확인하십시오.
이 로그에는 CLI의 출력에는 나타나지 않는 카메라별 건너뛰기 사유가 설명되어 있습니다.

의도적으로 메타데이터만 처리하는 실행(모든 제품 토글을 끄고 `--indices`를 사용하지 않음)은 여전히
**성공**으로 간주됩니다. 왜냐하면 이 경우 빈 이미지 출력이 올바른 결과이기 때문입니다.**광센서 전용 실행**도 마찬가지입니다: `.daq` 녹화 파일로 구성된 폴더에는 정의상 내보낼 이미지가 없으며,
대신 해당 실행이 기록한 보정된 `.daq` / `.csv` 파일을 기준으로 평가됩니다.

---

## `chloros-cli login`

Chloros+ 클라우드 계정으로 이 기기를 인증하십시오. 인증 정보는 `~/.chloros/user_session.json`에 안전하게 캐시됩니다.

```
chloros-cli login EMAIL PASSWORD
```

### 예시

```bash
chloros-cli login user@example.com 'YourPassword'

# Passwords containing $ should use SINGLE quotes
chloros-cli login user@example.com 'my$ecret$pass'
```

> **PowerShell `$$` mangling is auto-corrected.** In double quotes PowerShell expands `$$` (비밀번호에서 일부를 제거하거나 중복 입력). 401 오류가 발생하면 CLI은 자동으로 `$$`를 다시 추가하여 재시도한 다음, 중복 제거된 비밀번호의 절반을 사용하여 재시도합니다. 재시도가 성공하면 로그인이 완료되고, 다음에 사용할 올바른 작은 따옴표 구문이 표시됩니다.

> **헤드리스/스크립트 사용 시: 캐시된 세션이 없으면 빠른 오류 대신 대화형 프롬프트가 표시됩니다.** 백엔드를 생성하는 모든 명령어 (`process`, `status`, `export-status`, `time-sync`, …)은 캐시된 라이선스/세션 없이 실행될 경우 대화형 모드로 전환됩니다. `Email:` / `Password:` 프롬프트가 표준 입력(stdin)에 표시된 후 진행됩니다. 따라서 캐시된 세션이 없는 무인 작업은 입력을 기다리다가 응답이 멈출 수 있습니다. 헤드리스 작업을 예약하기 전에 시스템당 한 번씩 `chloros-cli login EMAIL PASSWORD`를 실행하십시오.

---

## `chloros-cli logout`

캐시된 세션을 지우고 다음 호출 시 새로운 로그인을 강제합니다.

```bash
chloros-cli logout
```

---

## `chloros-cli status`

현재 라이선스 등급(Iron/Copper/Bronze/Silver/Gold), 인증된 사용자 및 장치 바인딩 횟수를 표시합니다.

```bash
chloros-cli status
```

---

## `chloros-cli export-status`

실시간 Thread-4 내보내기 진행 상황을 확인합니다. 다른 셸에서 `process`가 실행 중인 **도중**에도 안전하게 호출할 수 있습니다.

```bash
chloros-cli export-status
```

---

## `chloros-cli language`

CLI의 표시 언어를 설정합니다(CJK, RTL, 인도어 등 38개 언어 지원). 스크립트를 렌더링할 수 없는 구형 콘솔에서는 영어로 원활하게 대체됩니다.

```
chloros-cli language [LANG_CODE] [--list]
```

### 예시

```bash
# List all available languages
chloros-cli language --list

# Switch to Spanish
chloros-cli language es

# Show the currently-active language
chloros-cli language
```

---

## 프로젝트 폴더 명령어

이 명령어들은 기본 프로젝트 폴더 위치를 관리합니다(GUI와 공유됨).

```bash
chloros-cli set-project-folder "/home/user/Chloros Projects"
chloros-cli get-project-folder
chloros-cli reset-project-folder
```

---

## `chloros-cli update`

Linux/ Jetson 전용. `version_url`에서 `/etc/chloros/update.conf`를 확인하고, 일치하는 `.deb`를 다운로드 및 해당 `.deb`를 다운로드 및 설치할 수 있도록 제안합니다.

```bash
chloros-cli update            # check + install
chloros-cli update --check    # check only
```

Linux에서 /Jetson의 경우 CLI에서도 **매번 시작 시 자동 업데이트 확인**을 수행합니다(비차단 방식이며, 명령 실행을 지연시키지 않음): `/etc/chloros/update.conf`를 읽어온 후, 결과를 `~/.chloros/update_cache.json`에 1시간 동안 캐시하고, 더 새로운 버전이 존재할 경우 `Update available: vX.Y.Z / Run: chloros-cli update`를 출력합니다. 오류가 발생하거나 Windows가 지정된 경우, 아무런 메시지 없이 건너뜁니다.

---

## `chloros-cli selftest`

7단계의 스모크 테스트를 실행합니다: 버전, 포트 가용성, 백엔드 시작, `/api/test`, `/api/system-info`(GPU/CUDA/PyTorch), 노이즈 제거 모델 존재 여부, CUDA+노이즈 제거 준비 상태.

```bash
chloros-cli selftest
```

---

## `chloros-cli time-sync`

PTP 그랜드마스터 상태 및 제어. Chloros 호스트에서 PTP 그랜드마스터를 실행하며, LATTICE CAM 및 DAQ-E 장치는 기기 간 타임스탬프를 위해 이 그랜드마스터에 종속됩니다.

| 하위 명령어 | 설명 |
| --- | --- |
| `status` | 그랜드마스터 상태, BMCA 우선순위, 클럭 식별자 표시. |
| `peers` | Delay_Req를 통해 감지된 슬레이브 목록 표시 (카메라 + DAQ-E 센서). |
| `cameras` | 카메라별 PTP 상태 (`PtpStatus`, `PtpOffsetFromMaster`, `PtpMeanPathDelay`). |
| `restart` | 그랜드마스터 프로세스를 재시작합니다. |
| `set-priority --priority1 N --priority2 N` | BMCA 우선순위 재정의. |

### 예시

```bash
chloros-cli time-sync status
chloros-cli time-sync peers
chloros-cli time-sync cameras
chloros-cli time-sync restart
chloros-cli time-sync set-priority --priority1 1 --priority2 1
```

---

## `chloros-cli lattice`

LATTICE 카메라 제어. 모든 하위 명령은 Chloros 백엔드를 통해 전달되며, 이 백엔드는 카메라 풀을 소유하므로 후속 CLI 호출은 동일한 열린 핸들을 재사용합니다.

### 공통 옵션 (대부분의 하위 명령어에서 공유됨)

| 플래그 | 설명 |
| --- | --- |
| `-d, --device N` | 카메라 인덱스 (기본값: 0). |
| `-s, --serial SN` | 특정 일련번호; `--device`를 재정의합니다. |
| `--serials SN1,SN2,…` | 다중 카메라 작동을 위한 쉼표로 구분된 일련 번호. |
| `--all` | 탐지된 모든 카메라에 대해 작동합니다. |
| `--exposure US` | 노출 시간(마이크로초 단위). |
| `--gain DB` | dB 단위의 게인. |
| `--pixel-format FMT` | 예: `BayerRG8`, `BayerRG12`. |
| `--width N` / `--height N` | 이미지 크기. |
| `--preset {default,high_quality,high_speed,triggered}` | 사전 설정 적용. `triggered`를 제외한 모든 설정은 자유 실행 모드입니다. `triggered`는 2번 라인의 하드웨어 에지 신호에 따라 카메라를 작동시키며, 해당 라인에 신호가 없으면 캡처를 하지 않고 무한정 대기합니다. |
| `-o, --output DIR` | 출력 디렉터리(기본값: `output`). |
| `--packet-size {auto,jumbo,standard,N}` | GVSP 패킷 크기. `auto`는 ICMP+GVSP 프로브를 실행합니다. `jumbo` = 9000; `standard` = 1500. |

### LATTICE 카메라 첫 연결 워크플로우

```bash
# 1. Discover cameras on the network
chloros-cli lattice info

# 2. Single-cam smoke test: capture one frame.
#    By default this saves EVERY export type applicable to the cam
#    (raw, debayered, radiance, reflectance, preview). Pass e.g.
#    `--processing debayered` to save just one.
chloros-cli lattice capture -o output/

# 3. Connect a synchronized array (RECOMMENDED ENTRY POINT for arrays).
#    This is the same "smart-prep" flow the Chloros GUI uses:
#      - Network capability probe (ICMP DF ping + GVSP probe)
#      - Tier auto-pick (sim-emit / ftd-stagger / slip)
#      - Auto-shrink frame size to fit the wire
#      - PTP enabled by default
#      - Per-cam pixel format auto-pick
#      - AE seeding from the cam's saved state
#      - GPIO trigger config on Line2
chloros-cli lattice array-connect \
  --serials 213800234,214000533,214701288,214701292

# 4. Capture one synced frame group from the live array.
#    Defaults to --processing all (one file per export type per cam);
#    pass a single level to narrow it, e.g. --processing reflectance.
chloros-cli lattice array-capture --processing reflectance -o output/

# 5. Live-preview one cam in your browser
chloros-cli lattice viewer --serial 213800234

# 6. Tear down when done
chloros-cli lattice array-disconnect
```

### 하위 명령어 참조

#### 탐색 및 정보

| 하위 명령 | 용도 |
| --- | --- |
| `lattice info` | 연결된 카메라 목록 표시(제조사, 모델, 일련번호, IP, MAC). |
| `lattice info` | 최적의 카메라 구성을 위해 호스트 시스템 분석. `--no-discover`는 카메라 탐색을 건너뜁니다(더 빠르며, NIC 전용 분석). |
| `lattice network [--fix] [--estimate] [--cameras N]` | NIC 설정 확인/수정; 대역폭/FPS 추정. |
| `lattice network-analysis --master SN --slaves SN1,SN2,… [--width N] [--height N] [--pixel-format FMT] [--binning N] [--force-tier TIER] [--backend-url URL] [--json]` | 안정형 스키마 백엔드 네트워크 성능 + 어레이 권장 사항 (`status` ∈ `ok` / `auto_shrunk` / `auto_capped_fps` / `needs_force_slip` / `error`). `auto_capped_fps`는 요청된 해상도를 유지하되 목표 fps를 제한합니다 — `recommended.recommended_target_fps`를 참조하고 이를 연결 대상으로 전달하십시오; 이를 성공으로 간주하며, 오류로 간주하지 마십시오. |
| `lattice analyze-array [--models M1,M2,…] [--binning N] [--n-active N] [--width N] [--height N] [--pixel-format FMT] [--force-tier TIER] [--json]` | 카메라를 열지 않고 수행하는 가설 분석. **`--n-active`는 이 배열에 포함된 카메라 수뿐만 아니라 네트워크에 연결된 전체 카메라 수를 나타냅니다**— 독립형 카메라가 동시에 스트리밍되거나, 와이어 예산이 카메라 수를 과소 산정한 수요를 기준으로 계산될 때 발생시킵니다(기본값: `len(--models)`). 항상 집계된 `Wire budget:`(요구된 MB/s 대 충돌 방지 상한)와 `Max cameras:` 행을 항상 출력하며, 어레이가 와이어를 과부하할 경우 `** OVER-SUBSCRIBED**` 플래그를 표시합니다 — [어레이 fps 및 버스트 모델](#array-fps--burst-model)을 참조하십시오. |
| `lattice gpu` | GPU 상태 표시. |
| `lattice firmware [--update] [--force] [-y\|--yes]` | 카메라 펌웨어 확인 또는 업데이트. 로컬 `.fwa` 선택은 고정되어 있습니다: `firmware/<MODEL_PREFIX>/`에 있는 파일 중 빌드의 `MIN_FIRMWARE_VERSION`와 일치하는 파일이 존재할 경우 플래싱됩니다(최신 버전만 대체용으로 사용됨). 따라서 디스크에 준비된 최신 벤더 이미지는 해당 핀이 변경될 때까지 비활성 상태입니다. 의도적으로 출시된 최신 릴리스는 서명된 AWS 매니페스트를 통해 기기에 전달되며, 최신 버전이 있을 경우 이 방법이 선호됩니다. |
| `lattice presets [--apply NAME]` | 카메라 사전 설정을 나열하거나 적용합니다. |
| `lattice status` | 실시간 카메라 상태. |

#### 캡처

| 하위 명령 | 용도 |
| --- | --- |
| `lattice capture [--format tiff\|png\|jpg] [--jpeg-quality N] [--processing LEVEL] [--levels L1,L2,…] [--force-daq]` | 단일 프레임. **기본적으로 모든 내보내기 유형을 저장합니다** (`--processing all`); [캡처 내보내기 수준](#capture-export-levels-the-all-default)을 참조하십시오참조. `--levels`는 명시적인 하위 집합을 저장합니다(`--processing`를 재정의함); `--force-daq`는 할당된 DAQ 측정값을 `.daq` 사이드카로 기록합니다.-only 캡처 시에도 할당된 DAQ 판독값을 `.daq` 사이드카로 기록합니다. `--jpeg-quality` = JPEG 품질 1–100 (기본값 95). |
| `lattice continuous [--format tiff\|png\|jpg] [--jpeg-quality N] [--queue-depth N]` | Ctrl+C를 누를 때까지 디스크로 스트리밍합니다. |
| `lattice viewer [--brightness N] [--ae-damping F] [--frame-rate FPS]` | 브라우저 기반 실시간 MJPEG 미리보기. `--ae-damping`는 자동 노출 감쇠(0.4–100)를 설정합니다. |

#### 센서 튜닝

| 하위 명령 | 용도 |
| --- | --- |
| `lattice configure [--get N1 N2…] [--set N=V N=V…] [--dump] [--json]` | 모든 GenICam 노드 읽기/쓰기. |
| `lattice exposure [--auto] [--auto-once] [--off] [--set US] [--brightness N] [--damping F] [--upper-limit US]` | 노출 및 AE. |
| `lattice gain [--auto] [--off] [--set DB]` | 게인 및 자동 게인. |
| `lattice resolution [--set WxH] [--offset X,Y] [--binning N] [--binning-mode Sum\|Average]` | 센서 ROI 및 비닝. |
| `lattice format [--set FMT] [--list]` | 픽셀 형식. |
| `lattice trigger [--mode On\|Off] [--source SRC] [--delay-us US] [--activation EDGE] [--list-sources] [--software]` | 하드웨어/소프트웨어 트리거. |
| `lattice white-balance [--auto] [--off] [--red R] [--blue B]` (플래그 없음 = 원샷 WB) | WB 작업.RGB/Bayer 카메라 전용; 모노 M3M에서는 (건너뜀). |
| `lattice color-profile [--set raw\|linear\|natural\|enhanced\|custom_temp] [--cct K] [--get]` | RGB 디스플레이 컬러 파이프라인. `natural` (기본값)은 저사양 라이브 마무리 처리입니다; `enhanced`는 디프린지 + 비브란스 + CLAHE 국소 대비를 추가하여 완전한 허브 패리티(hub-parity) 룩을 구현하며, 프레임당 마무리 처리 비용이 약 2배 증가하므로 **라이브** 프레임 속도를 제공합니다 — 저장된 캡처는 어느 경우든 항상 완전한 마무리를 적용받습니다. RGB /Bayer 카메라 전용; 모노 M3M에서는 건너뜁니다. |
| `lattice color [--saturation N] [--contrast N] [--reset] [--get]` | 채도/대비 표시 (RGB 필터 카메라). 모노 M3M에서는 건너뜁니다. |
| `lattice filter [--set NAME] [--list]` | 카메라의 필터 모델 설정 (`RGN-IMX265`, `OCN`, `NGB`, …). |
| `lattice power [--sleep]` | 프로브 전원/열 노드; 저전력 대기 모드 전환. |

#### 보정 및 센서

| 하위 명령 | 용도 |
| --- | --- |
| `lattice calibrate [--filter NAME] [--attempts N] [--save PATH]` | 반사율 타겟을 사용하여 보정합니다. |
| `lattice dls [--connect] [--spectrum] [--irradiance] [--mac MAC] [--filter NAME] [--json]` | 내장 하향광 센서 명령어. |
| `lattice vignette --input DIR --output DIR [--lens-model KEY]` | 기존 이미지에 비네팅 보정 적용. |

#### 멀티 카메라 (일시적 세션)

| 하위 명령 | 용도 |
| --- | --- |
| `lattice multi-info` | 동기화 역할을 가진 모든 카메라 나열. |
| `lattice multi-capture [--format FMT] [--jpeg-quality N] [--processing LEVEL]` | 각 카메라에서 동기화된 프레임 하나씩. 지속형 어레이가 연결된 경우 **기본적으로 모든 내보내기 유형**을 저장하며, 일시적 비어레이 대체 방식은 는**데베이어링만 수행**됩니다(나머지를 처리하려면 먼저 `array-connect`를 실행하십시오). |
| `lattice multi-stream [--fps F] [--count N] [--format FMT] [--jpeg-quality N]` | 동기화된 프레임 스트리밍 (일시적). |
| `lattice multi-test [--count N]` | GPIO 동기화 타이밍 테스트. |
| `lattice multi-detect [--line LINE] [--json]` | GPIO 마스터/슬레이브 배선 자동 감지. |

#### 정렬

| 하위 명령 | 목적 |
| --- | --- |
| `lattice align-calibrate [--method orb\|akaze\|phase\|checkerboard\|manual] [--model translation\|rigid\|affine\|homography] [--frames N] [--checkerboard RxC] [--points PATH] [--reference SN] [--save PATH] [--preview] [--vignette] [--prefilter none\|gradient\|clahe\|blur\|hist_match] [--rms-threshold-px N]` — 검출기/매칭기 조정 기능 `[--max-features N] [--ratio-threshold F] [--matcher bf\|flann] [--knn-k N]`, RANSAC 조정 기능 `[--ransac-threshold-px F] [--ransac-iters N] [--ransac-confidence F]`, 다중 프레임 결합 `[--averaging mean\|median\|inlier_weighted]`, 기하학적 제약 조건 `[--lock-rotation] [--lock-scale] [--lock-axis x\|y]`, 공간적 제한 `[--roi X0,Y0,X1,Y1] [--mask PATH]` 및 슬레이브별 재정의 `[--per-cam-override SN:KEY=VALUE]` (반복 가능) | 라이브 카메라로부터 정렬 프로파일을 계산합니다. `--prefilter`는 기본적으로 `gradient`(에지 맵; GUI/어레이 정렬기와 일치하며 — 에지는 스펙트럼 대역 간에 유지됨). `--matcher flann`는 특징점이 ~5000개 이상일 때 효과가 나타나며; `--averaging median`는 하나의 잘못된 캡처에 대해 견고하며, `inlier_weighted`는 일치 횟수에 따라 가중치를 부여하며; `--lock-scale`는 가장 가까운 회전으로 투영합니다(스케일 없음), `--lock-axis`는 한 개의 평행 이동 성분을 0으로 설정합니다; `--mask`는 모든 카메라에 적용됩니다(카메라별 설정을 원할 경우 `--per-cam-override`를 사용하십시오카메라별 설정을 적용하려면 `--per-cam-override`를 사용하십시오, 예: `--per-cam-override 214701292:method=phase`). `--rms-threshold-px`는 재투영 RMS가 게이트를 초과하는 보정값의 저장을 거부합니다. |
| `lattice align-apply --profile PATH [--format tiff\|png] [--bit-depth 8\|12\|16] [--bands NAMES] [--order NAMES] [--gpu\|--no-gpu] [--no-crop] [--per-camera] [--per-band] [--vignette] [--interpolation nearest\|linear\|cubic\|lanczos] [--border-mode constant\|replicate\|reflect\|wrap] [--border-value N]` | 정렬된 멀티밴드 프레임 하나를 캡처합니다. `--bit-depth`는 기본적으로 카메라에 맞추도록 설정되어 있습니다. `--no-crop`는 전체 프레임을 유지합니다(검은색으로 채움). `--interpolation`(기본값: `linear`) 및 `--border-mode`/`--border-value`(기본값: `constant`/0)는 CPU 워프를 제어합니다. GPU 경로는 어떤 경우든 이차선형(bilinear)으로 처리됩니다. |
| `lattice align-stream --profile PATH [--fps F] [--count N] [--bit-depth 8\|12\|16] [--bands NAMES] [--order NAMES] [--gpu\|--no-gpu] [--no-crop] [--per-band] [--vignette] [--interpolation nearest\|linear\|cubic\|lanczos] [--border-mode MODE] [--border-value N]` | 스트림 정렬된 멀티밴드 프레임 (`align-apply`와 동일한 워프 조절 기능). |
| `lattice align-info --profile PATH [--json]` | 프로필 세부 정보 표시. |
| `lattice align-reorder --profile PATH [--order NAMES] [--enable SERIALS] [--disable SERIALS]` | 레이어 순서 변경. |

#### 색인 / 식생 수학

```bash
# Offline: compute NDVI from an aligned multi-band TIFF
chloros-cli lattice index --input aligned.tif --preset NDVI \
  --output ndvi.tif --colorize --gradient RdYlGn

# Live: discover array, calibrate alignment, capture, compute index, in one go
chloros-cli lattice index --live --profile align.json --preset NDVI \
  --save-multiband -o output/
```

전체 플래그 세트: `--input PATH | --live --profile PATH`, `--preset NAME` (NDVI / NDRE / EVI / SAVI / GNDVI /…), `--formula EXPR`, `--channel SYM=BAND` (반복 가능), `--capture-level raw|debayered|radiance|reflectance|unknown` (소스 TIFF에 기록된 캡처 레벨을 재정의함; 기본값: TIFF 메타데이터에서 읽음), `--output PATH`, `--output-format all|raw|tif|colorized|lut|png`, `--gradient NAME|JSON`, `--vmin/--vmax/--percentile LO,HI`, `--bg-mode clip|transparent|indexColor|backgroundColor`, `--colorize`, `--list-presets`, `--list-gradients`. `--live`의 경우 정렬 워프 노브도 적용됩니다: `--save-multiband`, `--gpu/--no-gpu`, `--no-crop`, `--bit-depth 8|12|16`, `--vignette`, `--interpolation nearest|linear|cubic|lanczos`, `--border-mode constant|replicate|reflect|wrap`, `--border-value N`.

> **`--channel` 심볼은 대소문자를 구분합니다.** 심볼 부분은 프리셋의 채널 이름과 정확히 일치해야 합니다(프리셋은 소문자를 사용함, 예: NDVI = `red`,`nir` — `--list-presets` 참조), 대역 쪽은 정렬된 스택 내의 대역 이름과 일치해야 하며(또는 오프라인 모드에서는 0을 기준으로 한 대역 인덱스여야 함). `--channel red=Red_660 --channel nir=NIR_850`는 작동하지만, `--channel RED=660`는 `channel_map missing entries` 오류로 실패합니다.

#### 지속적 연결 (Smart-Prep, GUI와 동등한 흐름)

이 명령어들은 CLI 호출 간에도 백엔드 풀에서 카메라 연결을 유지합니다.

| 하위 명령어 | 용도 |
| --- | --- |
| `lattice cam-connect [--serial SN]` | 풀에 카메라 1대를 추가합니다(단일 카메라, 어레이 없음). |
| `lattice cam-disconnect [--serial SN] [--all]` | 해제합니다. |
| `lattice cam-list` | 풀에 있는 카메라 목록 표시. |
| **`lattice array-connect`**|**지속적인 동기화 어레이 연결 (권장 진입점).** 전체 GUI 스마트 준비(smart-prep) 흐름을 실행합니다. |
| `lattice array-disconnect [--array-id ID] [--all]` | 어레이 해제. |
| `lattice array-list` | 연결된 어레이 목록 표시. |
| `lattice array-status [--array-id ID]` | 실시간 fps, PTP, 마지막 오류. |
| `lattice array-capture [--processing LEVEL\|all] [--levels L1,L2,…] [--aligned\|--no-aligned] [--index\|--no-index] [--force-daq] [--smart] [--fastest] [--compression deflate\|none] [--continuous\|--interval S] [--count N] [--duration S]` | 실시간 어레이에서 동기화된 캡처 1회 — 단일 / 연속 / 간격 / 최단. **기본값은 `all`**입니다(카메라당 해당 내보내기 유형별로 파일 1개). 건너뛴 카메라(예: RGB, 방사도/반사도에서 제외됨)는 `Skipped: SN:<serial> (<reason>)`로 보고되며, 반사도에 사용된 DAQ 측정값은 함께 저장되어 `DAQ: <path>`로 보고됩니다. [캡처 모드, 레코더 및 오프라인 재처리](#capture-modes-recorders--offline-reprocess)를 참조하십시오. |
| `lattice array-record [--fps F] [--duration S] [--gif] [--gif-only]` | 실시간 결합 인덱스 뷰를 비디오/GIF로 기록(모니터링 등급; 결합 스트림이 열려 있어야 함). |
| `lattice array-burst [--duration S] [--max-frames N] [--build] [--products …]` | 고프레임률 원시 베이어 버스트 (분석용; 오프라인 재처리 필요). |
| `lattice array-build-video --burst-dir DIR [--products …] [--fps F] [--save-tiffs] [--gif]` | 저장된 원시 버스트를 보정된 비디오로 재처리합니다. |

##### `array-connect` 옵션

| 플래그 | 기본값 | 설명 |
| --- | --- | --- |
| `--serials SN1,SN2,…` | 모든 LATTICE 카메라 자동 탐지 (2대 이상 필요) | 첫 번째 일련번호가 MASTER가 됩니다. 생략 시, 탐지 필터가 LATTICE(`TRI032*`) 모델로 제한되어 모든 카메라가 연결됩니다. |
| `--line {Line0,Line2,Line3}` | `Line2` | GPIO 동기화 라인. |
| `--target-fps F` | 자동 | 마스터 트리거 발사 속도. |
| `--force-tier {sim-capture-sim-emit, sim-capture-ftd-stagger, slip-emit-and-capture}` | 자동 | 티어 선택기를 재정의합니다. |
| `--wire-ceiling-mbps MB_PER_S` | 자동 감지 | **호스트의 지속적 유선 대역폭(MB/s 단위) — 전체 어레이 할당이 이 값에 따라 결정됩니다.** 어레이에서 GVSP 손상 프레임을 보고할 경우 이 값을 낮추십시오. 자동 값은 NIC가 알리는 링크 속도에서 파생되는데, 이는 USB 어댑터, 대역폭이 낮은 PCIe 레인 및 사용량이 많은 공유 패브릭의 경우 실제보다 높게 산정됩니다. 프로젝트의어레이 캡처 블록에 저장되므로, reopen / CLI / SDK 재연결 시 복원됩니다. [어레이 상태](#array-health--which-subsystem-is-losing-frames)를 참조하십시오. |
| `--binning {1,2,4}` | auto | 하드웨어 비닝. |
| `--no-recommend` | off | 네트워크 분석 단계를 건너뜁니다. |
| `--no-ptp` | off | PTP 비활성화 (이 경우 카메라 간 타임스탬프를 **비교할 수** 없습니다). |

### Smart-AE / Smart-Capture

LATTICE 어레이는 연결되는 즉시 백그라운드에서 AE를 지속적으로 실행하지만, 새로 조준된 장면은 수렴하는 데 잠시 시간이 걸립니다. `array-capture --smart`는 **편의를 위해 패키지화된** 기능입니다. 어레이 내 모든 카메라에서 AE가 안정화될 때까지 기다린 후 캡처를 시작합니다. 세션 도중 장면을 변경할 때 사용하십시오.

```bash
# Connect once, then take settled captures whenever you re-point the rig
chloros-cli lattice array-connect --serials SN1,SN2,SN3,SN4
chloros-cli lattice array-capture --smart --processing reflectance -o pose_a/
# (move the rig)
chloros-cli lattice array-capture --smart --processing reflectance -o pose_b/
```

안정화 정책은 기본적으로 보수적으로 설정되어 있습니다: 타임아웃 5초, 안정성 창 1.5초, 노출 편차 허용 범위 ±5%. 자동화 설정과 다른 동작이 필요한 경우, ‘SDK’(`ArrayHandle.capture_smart(settle_timeout_s=…, stability_window_s=…, exposure_tolerance_pct=…)`)를 통해 조정하십시오.

### 캡처 내보내기 레벨(`all` 기본값)

이번 릴리스부터 `lattice capture`, `lattice multi-capture` 및 `lattice array-capture`는 **기본적으로 `--processing all`로 설정됩니다**. 이는 각 카메라에 대해 하나의 저장 파일을 생성하는 카메라마다 적용되는 내보내기 유형별로 하나의 저장 파일이 생성되며, 이는 GUI의 “모두 캡처” 동작과 일치합니다. 레벨은 다음과 같습니다:

| 레벨 | 출력 | 적용 대상 |
| --- | --- | --- |
| `raw` | 단일채널 베이어(모노 카메라: 단일 대역) — 센서에서 직접 출력. | 모든 카메라. |
| `debayered` | 3채널 BGR 디모자이크(모노 카메라: 1채널 그레이스케일). | 모든 카메라. |
| `radiance` | 전체 방사계측 체인을 통한 float32 W/m²/sr/nm. | 다중 스펙트럼 (M3C/M3M) 전용 — **RGB-필터 카메라의 경우 건너뜁니다**. |
| `reflectance` | uint16 ρ (`32768` = 1.0), Pix4D 호환. | 다중 스펙트럼 전용이며, **DAQ가 바인딩되어 있고 카메라가 보정된 경우에만** 적용됨; 그렇지 않으면 생략됨. |
| `preview` / `display` | 전체 GUI 미리보기 체인 (카메라 프로필에 따른 CCM + WB + 감마). `lattice capture`는 이를 `preview`로 명명하며, `array-capture`/`multi-capture` `display`를 사용합니다. | 모든 카메라. |

단일 레벨만 전달하여 해당 레벨만 저장합니다(`--processing debayered`). `all`를 요청할 경우, 주어진 캠에 적용되지 않는 레벨은 오류 처리되지 않고 건너뛰어지며(보고됨) — 연결되지 않았거나 보정되지 않은 캠도 여전히 `raw` / `debayered` / `preview`를 받습니다.

모든 반사율 프레임에 대해, 실제로 사용된 DAQ 하향 측정값은 이미지의 옆에 있는 **`.daq`** 사이드카에 기록되며(이를 통해 나중에 캡처 데이터를 재처리할 수 있음), `DAQ:` 행에 보고됩니다.

### 캡처 폴더의 구조

각 내보내기 유형은 `-o` 아래의 **별도의 하위 폴더**에 저장되므로, 다중단계 캡처에서도 유형이 혼합되지 않습니다:

```
output/
├── raw/           capture_<ts>_SN<serial>_raw.tif
├── debayered/     capture_<ts>_SN<serial>_debayered.tif
├── radiance/      capture_<ts>_SN<serial>_radiance.tif
├── reflectance/   capture_<ts>_SN<serial>_reflectance.tif
├── preview/       capture_<ts>_SN<serial>_display.tif
├── index/         per-camera vegetation-index (LUT) render, when --index is on
├── composite/     array foreground/background live-view composite, when produced
└── *.daq          the downwelling reading matched to the capture
```

`<ts>`는 캡처 타임스탬프이고 `<serial>`는 카메라 일련번호이므로, 동기화된 하나의 그룹은
카메라 전반에 걸쳐 동일한 타임스탬프암프를 공유합니다. **한 가지 비대칭적인 점이 있습니다:** `display` 레벨은
`preview/`라는 이름의 폴더에 저장되는 반면, 파일 자체는 이름에 `_display`를 유지합니다 — 폴더와 확장자는
해당 레벨에서만 다릅니다. 알 수 없는 레벨은 해당 레벨 이름과 동일한 폴더로 처리되며, 하위 폴더를
생성할 수 없는 경우 파일이 손실되지 않고 출력 루트에 기록됩니다.

**캡처 폴더 재처리:**`chloros-cli process`를**캡처 루트**
(`output/`)를 지정하십시오. `process`는 일반적으로 지정한 폴더만 가져오지만, 해당 폴더에
이미지가 없고 하위 폴더가 있는 경우 자동으로 하위 폴더로 내려갑니다. 따라서 루트의 레벨별 하위 폴더와
루트 `.daq`는 한 번에 모두 가져옵니다. 캡처된 모든 레벨은 레벨당 하나의 이미지가 아닌,
단일 이미지로 가져오며 다른 레벨들은 모드로 사용할 수 있습니다.

**레벨 하위 폴더**(예: `output/raw/`)를 직접 지정하는 방법도 작동합니다. 이렇게 하면 루트
`.daq`가 제외되므로, `raw/`에서 방사계측 제품을 재파생할 때 DAQ 판독값을 복사하거나 해당 위치로 지정해야 합니다. 그렇지 않으면 타임스탬프 일치 대상을 확인할 수 없습니다.

**처리는 항상 `raw`에서 시작됩니다.** 각 캡처 내에서 원시 프레임이 파이프라인의 소스이며,
`debayered`, `radiance`, `reflectance` 및 `preview`는 표시 가능한 모드로 제공되지만 파이프라인을 통해 다시
입력되지는 않습니다. 파생된 제품을 재처리하면 비네트, CCM 및
픽셀에 이미 베이킹된 라디언스 연산을 다시 적용하게 되므로, Chloros는 이중 처리를 피하기 위해
이를 거부합니다. 알아두어야 할 두 가지 결과:

- `index/` 및 `composite/` 렌더링은 **절대** 처리되지 않습니다. 이들은 캡처가 아닌 출력물이며 —
  NDVI의 LUT 렌더링 결과에는 의미 있는 라디언스 해석이 포함되어 있지 않습니다.
- `raw`가 **포함되지 않은** 상태로 내보낸 캡처 폴더(예: `array-capture --processing reflectance`)는
  유효한 파이프라인 소스가 없습니다. 이러한 캡처 파일은 정상적으로 가져오고 표시되지만, `process`는
  이를 건너뛰며 다음과 같이 표시합니다:

  ```
  [IMPORT-LEVEL] Skipping 4 already-processed file(s) with no raw source: capture_…_reflectance.tif
  [IMPORT-LEVEL] Processing starts from raw. Re-capture with --processing raw, or force an entry
                 point with --input-level.
  ```

  파생된 제품을 반드시 처리해야 하는 경우(예:
  `demosaic`가 활성화된 상태에서 캡처된 허브 세션이나 레거시 폴더) `--input-level {raw,debayered,processed}`는
  입력 지점을 강제 적용하여 건너뛰기 설정을 무시합니다. 이 플래그는 의도적으로 마련된 비상 탈출구이며, `auto`(기본값)
  는 원시 데이터가 없는 캡처를 절대 처리하지 않습니다.

### 혼합 필터 어레이에서 건너뛴 캡처

RGB와 다중 스펙트럼 카메라를 하나의 어레이에 혼합할 때, `array-capture --processing radiance` (또는 `reflectance`)는 다중 스펙트럼 프레임을 저장하고 RGB 카메라를 **건너뜁니다** — 광대역 센서의 경우 베이어(Bayer)당 복사도는 의미가 없기 때문입니다. CLI는 저장된 각 파일(내보내기 수준 포함), 각 `.daq` 출력 및 각 건너뛰기 항목을 명시적으로 출력하므로, 파일 수가 예상과 다를 리 없습니다:

```
  Saved: output/sync_…_SN213800234.tif [reflectance] (SN:213800234, fid:1)
  Saved: output/sync_…_SN214000533.tif [reflectance] (SN:214000533, fid:1)
  Saved: output/sync_…_SN214701288.tif [reflectance] (SN:214701288, fid:1)
  DAQ:   output/sync_…_daq-e-54b5e0.daq
  Skipped: SN:214701292 (reflectance-not-applicable-to-rgb-cam filter=RGB)

  3 synchronized frames captured. (1 skipped)
```

건너뛰기 사유 토큰은 `<level>-not-applicable-to-rgb-cam` 패턴을 따릅니다. Reflectreflectance도 `reflectance-skipped-no-fresh-dls` / `reflectance-skipped-bound-daq-unavailable (…)`로 건너뛸 수 있으며, 대역이 DAQ 광 센서의 방사계 보정 범위(~374–974 nm) — 출하 중인 SKU 중에서는 반사율 패널 워크플로우만 지원하는 F988 모델만 해당됩니다.

필터 유형에 관계없이 모든 캠을 포함하려면 `--processing debayered` (또는 `display`)를 사용하여 필터 유형에 관계없이 모든 카메라를 포함하거나, 기본값인 `all`를 사용하여 한 번에 카메라별 적용 가능한 모든 레벨을 확보하십시오.

---

## 캡처 모드, 레코더 및 오프라인 재처리

이들 모두 **지속적 배열**에서 작동합니다(먼저 `array-connect`를 실행하십시오). 이는 GUI 캡처 패널을 그대로 반영합니다.

### `array-capture` 모드

`array-capture`는 네 가지 셔터 모드와ggles로 내보내기 기능을 포함합니다:

| 모드 | 플래그 | 동작 |
| --- | --- | --- |
| **단일** *(기본값)* | (없음) | 동기화된 캡처 그룹 하나를 생성한 후 종료. |
| **연속** | `--continuous` | `Ctrl+C`, `--count N` 또는 `--duration S`가 발생할 때까지 연속으로 실행됩니다. |
| **간격** | `--interval S` | `S`초마다(각 패스 시작 시점부터 측정) 한 번씩 패스 수행, 동일한 범위. |
| **가장 빠름** | `--fastest` | 원본 데이터만 + 할당된 DAQ 판독값 + 결합 지수 합성; 방사도/반사도/표시 계산 단계를 건너뛰어 프레임이 빠르게 출력됩니다. `--processing raw --force-daq`를 암시합니다. 저장된 `.daq`를 나중에 보정된 결과물로 재처리합니다. |

내보내기 토글 (모든 모드와 결합 가능; 모두 GUI/SDK 엔드포인트를 공유):

| 플래그 | 효과 |
| --- | --- |
| `--processing LEVEL` | 단일 내보내기 수준, 또는 `all` (기본값). |
| `--levels L1,L2,…` | 명시적인 내보내기 유형 하위 집합(예: `raw,radiance,reflectance`); **`--processing`**를 재정의합니다. |
| `--aligned` / `--no-aligned` | 모든 구성원의 비-원시(non-raw) 내보내기를 배열의 [정렬 프로파일](#alignment)로 워프(co-registered). 원시 데이터는 워핑되지 않지만 메타데이터에 변환 정보를 포함합니다. 배열에 프로필이 없는 경우 정렬되지 않은 상태로 되돌아가며(경고 표시) |
| `--index` / `--no-index` | 구성된 경우 카메라별 식생 지수 오버레이를 저장하거나 건너뜁니다. 기본값: 렌더링합니다. |
| `--force-daq` | 선택된 레벨에 필요하지 않은 경우에도 할당된 DAQ/DLS 판독값을 `.daq` 사이드카로 저장합니다 (예: 원본 데이터만 캡처한 경우)에도 할당된 DAQ/DLS 측정값을 `.daq` 사이드카 파일로 저장하여, 프레임을 오프라인에서 반사율/지수 데이터로 재처리할 수 있도록 합니다. |
| `--smart` | 모든 카메라에서 AE가 안정화될 때까지 기다린 후 트리거합니다 ([Smart-AE / Smart-Capture](#smart-ae--smart-capture) 참조). |
| `--compression {deflate,none}` | TIFF 픽셀 압축. `deflate` (기본값) = 무손실 zlib L1 + 수평 예측기, 전체 해상도 프레임당 약 4.1 MB; `none` = 압축 해제압축, 프레임당 약 6.3 MB로 쓰기 속도가 약 5배 더 빠름 — 디스크 용량이 허용할 경우 최대 지속 전송 속도를 위해 사용. 둘 다 무손실이며 가져올 때 동일하게 읽힙니다. |

> **단일 쓰기 TIFF + 지속 전송 속도 모델.**캡처 데이터는 픽셀 + XMP + IFD0 제조사/모델 정보를 포함하는**단일**TIFF 파일 패스로 기록됩니다(풀 해상도 Mono12 기준 측정 결과: 압축 시 36 ms / 6.5 ms 비압축, 기존 ‘쓰기 후 ExifTool로 재작성’ 방식의 ~148 ms 대비); 남은 유일한 ExifTool 작업 (EXIF 하위 IFD 다듬기)는 비동기 백그라운드 워커에서 실행되며, 이 작업이 실행되지 않더라도 프레임은 완성되어 가져오기 준비가 됩니다. DEFLATE 압축은 Python GIL을 유지하므로, 압축된 쓰기 작업은**카메라별 쓰기 스레드 간에** **병렬화되지 않습니다**카메라별 쓰기 스레드 간에 병렬화되지 않습니다 — 센서 속도(~10.4 fps)로 8대의 카메라를 통해 풀 해상도 영상을 지속적으로 캡처하려면 `--compression none`**및** NVMe급 디스크가 필요합니다 (지속적 쓰기 속도 ~500 MB/s)가 필요합니다. 동일한 설정 옵션은 `POST /api/camera/array/capture`에서 `compression`로 노출됩니다.

```bash
# Interval timelapse: one reflectance pass every 10 s for 5 minutes
chloros-cli lattice array-capture --interval 10 --duration 300 \
  --processing reflectance -o timelapse/

# Fastest grab for a moving rig — raw + .daq now, calibrate later
chloros-cli lattice array-capture --fastest -o flightline/

# Co-registered multi-band export (drop the index overlay)
chloros-cli lattice array-capture --processing reflectance --aligned --no-index -o out/
```

### `array-record` — 통합 인덱스 비디오/GIF (모니터링 등급)

**라이브 통합 인덱스 뷰**에 표시되는 모든 내용을 `.avi` (선택적으로 `.gif`에도) 기록합니다. 실시간 합성 영상을 추출하므로, 프레임이 도착하기 위해서는 통합 스트림이 열려 있어야 합니다 (예: GUI에서 배열이 미리보기 중일 때) 열려 있어야 프레임이 기록됩니다. 2초마다 진행 상황을 확인하며, `--duration`, `Ctrl+C`에서 또는 레코더가 자체 종료될 때 중지됩니다.

```bash
# 30-second combined-index clip at 10 fps, plus a GIF
chloros-cli lattice array-record --duration 30 --fps 10 --gif -o monitoring/
```

| 플래그 | 기본값 | 설명 |
| --- | --- | --- |
| `--array-id ID` | 어레이만 | 대상 어레이(하나만 연결된 경우 생략). |
| `-o, --output DIR` | `output` | 출력 디렉터리(백엔드 로컬). |
| `--fps F` | `10` | 녹화 프레임 속도. |
| `--duration S` | Ctrl+C까지 | `S` 초 후 자동 중지. |
| `--gif` | 꺼짐 | 애니메이션 GIF도 함께 기록. |
| `--gif-only` | 꺼짐 | GIF만 기록 만 저장(`.avi` 없음). |

### `array-burst` — 원시 베이어 고프레임률 연사 (분석용)

그랩 루프의 동기화된 그룹 버퍼를 직접 읽습니다 — **보정 체인, exiftool, 라이브 뷰가 필요 없음** — 따라서 카메라의 최대 그랩 속도로 실행됩니다. 원시 프레임 + 프레임별 매니페스트 + 고유한 DLS 판독값마다 하나의 `.daq` `<output>/bursts/<base>/` 아래에 저장합니다. 오프라인에서 재처리(다음 명령어)하거나, `--build`를 전달하여 중지 시 즉시 처리할 수 있습니다.

```bash
# 5-second raw burst, then build the combined index video in one shot
chloros-cli lattice array-burst --duration 5 --build \
  --products combined:index --fps 10 -o capture/
```

| 플래그 | 기본값 | 설명 |
| --- | --- | --- |
| `--array-id ID` | 배열 전용 | 대상 배열. |
| `-o, --output DIR` | `output` | 출력 디렉터리(버스트 데이터는 `<DIR>/bursts/<base>/`에 저장됨). |
| `--duration S` | Ctrl+C 입력 시까지 | `S` 초 후 자동 중지. |
| `--max-frames N` | 제한 없음 | `N` 개의 원시 프레임이 처리된 후 자동-`N`개의 원시 프레임이 처리된 후 자동 중지. |
| `--build` | 꺼짐 | 중지 후 즉시 버스트를 재처리합니다(`array-build-video`와 동일). |
| `--products …` | `combined:index` | `--build`와 함께: 생성할 비디오 지정 (아래 참조). |
| `--fps F` | `10` | `--build`와 함께 사용할 경우: 출력 동영상 fps. |
| `--save-tiffs` | 꺼짐 | `--build`와 함께: 프레임별 보정된 TIFF 파일도 저장합니다. |
| `--gif` | 꺼짐 | `--build` 사용 시: 애니메이션 GIF도 작성합니다. |

### `array-build-video` — 저장된 연사 사진을 오프라인에서 재처리

각 원시 프레임을 가장 가까운 저장된 `.daq` 측정값과 시간적으로 매칭하고, 이를 **가져오기 파이프라인과 동일한 방사도/반사도/지수 체인**을 통해 처리하여, 하나 이상의 동영상을 렌더링합니다.

`--products`는 `kind:level` 항목들의 쉼표로 구분된 목록이며, 여기서 `kind` ∈ `per_cam` | `combined`이며, `level` ∈ `radiance` | `reflectance` | `index`. `level`만 지정된 경우(`kind:` 미지정 시) 기본값은 `per_cam`입니다. 기본값은 `combined:index`입니다.

```bash
# Per-cam reflectance video for every member + one combined NDVI video
chloros-cli lattice array-build-video \
  --burst-dir "capture/bursts/2026-06-24_141500" \
  --products per_cam:reflectance,combined:index \
  --fps 10 --save-tiffs
```

| 플래그 | 기본값 | 설명 |
| --- | --- | --- |
| `--burst-dir DIR` | (필수) | 버스트 폴더 경로 (`…/bursts/<base>/`). |
| `--products …` | `combined:index` | `kind:level` 목록, 위와 동일. |
| `--fps F` | `10` | 출력 동영상 fps. |
| `--save-tiffs` | 꺼짐 | 동영상과 함께 프레임별 보정된 TIFF 파일도 저장합니다. |
| `--gif` | off | 애니메이션 GIF도 함께 기록합니다. |

> **적절한 레코더를 선택하십시오.** `array-record`는 *모니터링 등급*입니다 — 화면에 표시되는 그대로의 라이브 합성 영상을 캡처하며, 스트림이 열려 있어야 합니다. `array-burst` → `array-build-video`는 *분석 등급*입니다 — 원시 센서 데이터를 전체 속도로 저장한 뒤, 라이브 뷰 없이도 보정된 복사도/반사도/지수 동영상을 재구성하며, 실시간 화면 확인이 필요하지 않습니다.

### 모노(M3M) 단일 대역 카메라

**M3M**라인은 베이어 방식의**M3C**의 모노 버전입니다: 카메라당 하나의 협대역 간섭 필터(`M3M-<lens>-F<wavelength>`, 예: `M3M-L87-F685`)가 적용되어 있어, 센서는 베이어 모자이크 없이**단일 그레이스케일 대역**을 제공합니다. 디모자이크할 필요가 없고, 채널 간 크로스톡을 분리할 필요도 없으며, 화이트 밸런스를 설정할 필요도 없습니다. 즉, RGB -디스플레이 색상 처리 파이프라인 전체가 적용되지 않습니다.

CLI에서 이는 다음과 같은 의미를 가집니다:

- **`lattice white-balance`, `lattice color-profile`, `lattice color`**모노 카메라를 감지하면**한 줄의 메시지와 함께 건너뜁니다**. 무의미한 설정을 강제로 적용하지 않습니다. 같은 세션에서 RGB /Bayer M3C 카메라에 대해서는 여전히 정상적으로 작동합니다.
- **`lattice calibrate` / `process --reflectance` / `array-capture --processing radiance`**는 여전히 작동합니다 — 복사도 및 반사도는 *밴드별* 방사계 맵이며, 단일 밴드에 대해서는 완벽하게 정의되어 있습니다. 모노 프레임은 **신원** 센서 응답 행렬을 포함하므로(3×3 언믹스 없음), 보정 계산 과정을 아무런 변경 없이 통과합니다.
- **단일 모노 카메라로는 식생 지수를 산출할 수 없습니다.**NDVI / NDRE / 등에는 최소 두 개의 밴드(예: Red + NIR)가 필요합니다. 모노 하드웨어에서 지수를 얻으려면, 서로 다른 파장의**여러** 대의 M3M 카메라를 향하게 하고, 이를 하나의 다중 밴드 스택으로 정렬한 후, *그* 스택에 대해 지수를 산출해야 합니다:

```bash
# Red (660) + NIR (850) mono pair -> aligned 2-band stack -> NDVI
chloros-cli lattice array-connect --serials SN_RED,SN_NIR
chloros-cli lattice index --live --profile align.json \
  --preset NDVI --channel red=Red_660 --channel nir=NIR_850 \
  --save-multiband -o output/
```

`--channel` 기호는 프리셋의 채널 이름과 **정확히** 일치해야 합니다 (대소문자 구분; NDVI는 소문자 `red`, `nir` — `--list-presets` 참조), 또한 band side는 정렬된 스택 내의 밴드 이름을 지정해야 합니다(오프라인 모드에서는 0을 기점으로 하는 밴드 인덱스도 허용됨, 예: `--channel red=0 --channel nir=1`).

스택 전체에서 구별 기준이 되는 것은 모델 문자열 내의 `M3M` 토큰이며(이 토큰은 `M3C` 문자열에는 절대 나타나지 않음), GUI/SDK에는 `is_mono`로 표시됩니다.

---

## 호스트 NIC 설정 및 튜닝 (LATTICE 어레이)

LATTICE 카메라는 호스트의 이더넷 어댑터를 통해 GVSP를 스트리밍하므로, 다중 카메라 어레이의 경우 어댑터의 **드라이버**와**수신 링 크기**는 링크 속도만큼 중요합니다. 설정이 잘못되면 어레이 설정 패널(및 `lattice network-analysis` / SDK의 `analyze_array_network()`에서도) 오류로 나타납니다.

### USB 10GbE 어댑터 — Realtek RTL8157 (“Realtek USB 10GbE Family Controller”)

| 항목 | 필수 값 | 중요한 이유 |
| --- | --- | --- |
| **드라이버 버전**|**≥ v10.67 (2026년 1월)**, INF `rtump64x64sta.inf` | 구형**2016**드라이버 (v10.65, `rtump64x64.inf`)는 시스템 종료/재시작/절전 모드 시**`DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`)**와 관련된 전원 차단 및 버그 체크를 제대로 처리하지 못합니다.**가 발생합니다. 전환 과정에서 시스템이 멈춰 버리고(약 5분 시간 초과), 사용자가 강제로 전원을 끄면, 반복되는 비정상적인 종료로 인해**WMI 저장소를 손상시킵니다**(PowerShell/도구가 `Invalid class` 오류로 작동하지 않음)이며, 다음 부팅 시**USB 스택이 멈춰 버립니다** (NIC가 활성화되지 않고, USB 드라이브 열거가 중단됨). realtek.com (또는 동글 제조업체)에서 업데이트를 적용한 후 정상 재시작을 시도하십시오. |
| **수신 버퍼**— 키워드 `ReceiveBufferLen` |**256**(드라이버 최대치) | NIC RX 링. 드라이버의 기본값인**32**는 사용 가능한 링을 약 0.26 MB만 남겨두는데, 이는 멀티캠 버스트에 비해 너무 작으므로 어레이 패널은 `Sim-emit burst … exceeds NIC RX ring usable capacity 0.26 MB`를 보고하고 연결을 차단합니다.**256**에서는 링 용량이 넉넉하여(**실험실 10GbE 호스트에서 측정된 약 13.5 MB**), RX 파이프라인이 멀티캠 GVSP 버스트를 처리할 수 있는 충분한 여유 공간을 확보합니다. (특정 구성이 실제로 *연결*되는지는 두 가지 검사 — **드레인 인식**허용 검사 및**총 과구독** 검사 — 단순한 버스트 대 링 비교가 아님; [어레이 fps 및 버스트 모델](#array-fps--burst-model)에 설명되어 있습니다.) |
| **수신 URB**— 키워드 `PendingReceives` |**64** (최대) | 전송 중인 USB 요청 블록; 버스트 흡수를 위해 수신 버퍼와 함께 증가시킵니다. |
| **점보 프레임** — 키워드 `*JumboPacket` | **9014** | 9000바이트 GVSP 패킷에 필요 (1500바이트 프레임 대비 패킷 수가 6분의 1 수준). |

> ⚠️ **NIC 드라이버를 업데이트하면 이러한 고급 속성이 기본값으로 재설정됩니다.**어댑터 드라이버를 업데이트하거나 교체한 후에는**다시 적용**해야 합니다. 그렇지 않으면 &quot;하드웨어에 변경 사항이 없음&quot;에도 불구하고 어레이 패널이 다시 차단됩니다. 이것이 이전에 정상 작동하던 시스템이 갑자기 연결을 거부하는 #이전에 정상 작동하던 장비가 갑자기 연결을 거부하는 가장 흔한 원인입니다.**관리자 권한**이 부여된 PowerShell에서 다음 명령을 실행하십시오(어댑터 이름을 해당 어댑터 이름으로 대체하십시오. 예: `"Ethernet 5"`):

```powershell
Set-NetAdapterAdvancedProperty -Name "Ethernet 5" -RegistryKeyword ReceiveBufferLen -RegistryValue 256
Set-NetAdapterAdvancedProperty -Name "Ethernet 5" -RegistryKeyword PendingReceives  -RegistryValue 64
Get-NetAdapterAdvancedProperty  -Name "Ethernet 5" -RegistryKeyword ReceiveBufferLen,PendingReceives   # verify
```

> **`lattice network --fix`는 USB 10GbE 어댑터를 다룹니다.** 이제 어댑터 유형을 감지하여 올바른 수신 링(receive-ring) 키워드를 조정합니다: PCIe NIC(Intel I219 등)의 경우 `*ReceiveBuffers`→2048)의 경우 `*ReceiveBuffers`→2048로, Realtek **USB** 10GbE 컨트롤러(`*ReceiveBuffers`를 노출하지 않음)의 경우 `ReceiveBufferLen`→256 + `PendingReceives`→64로 설정됩니다. 설정값은 각 드라이버가 보고하는 최대값(`NumericParameterMaxValue`)으로 제한되므로로 제한되므로, 범위를 벗어난 값이 기록되는 일은 절대 없습니다. **관리자 권한** 터미널에서 실행하십시오. 다른 레지스트리 기반 조정과 마찬가지로, 변경 사항은 어댑터 재시작 또는 시스템 재부팅 후에 적용됩니다. 위의 수동 `Set-NetAdapterAdvancedProperty` 명령어들은 여전히 훌륭한 대안입니다. 이 명령어들은 재시작 없이도 실시간으로 적용됩니다(어댑터를 재바인딩) 즉시 적용됩니다.

### 네트워크 기본 사항 (모든 LATTICE 링크)

- **주소 지정:** 링크 로컬 `169.254.0.0/16` (GigE Vision LLA). 호스트는 고정 `169.254.x.x/16`를 사용하며, 카메라와 DAQ-E는 동일한 범위 내에서 자동으로 할당됩니다. DHCP나 게이트웨이는 필요하지 않습니다.
- **패킷 크기:**점보(9000)를 권장하지만, 자동 탐색 기능을 통해 결정하도록 하십시오 — 이 기능은 연결할 때마다 재측정하며, GVSP 프로브를 통해 카메라의 1500바이트 ICMP 제한을 이미 넘어선 값을 확인하므로, 실제 회선이 9000을 전송할 수 있는 곳이라면 어디서든 점보 패킷으로 설정됩니다. 프로브보다 더 정확한 정보를 알고 있는 경우에만 `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000`로 고정하고, 영구 설정보다는 명령어별 설정을 우선하십시오. 고정 설정은 프로브를 건너뛰게 되므로, 따라서 경로가**매번** 9000을 실제로 전송할 수 없는 경우 캡처가 `SC_ERR_TIMEOUT -1011`로 인해 타임아웃됩니다([환경 변수](#environment-variables) 참조).
- **RX 링은 `ReceiveBufferLen`에 따라 확장됩니다:**기본값인 `32`에서 사용 가능한 링 용량은 ~0.26 MB(멀티캠 버스트 촬영에는 너무 작음)이며; 최대 `256`에서는 용량이 넉넉하며(실험실 10GbE 호스트에서 측정 시 ~13.5 MB), 실질적인 여유 공간을 제공합니다. 구성 연결 여부는 드레인 인식 허용 검사**및** 아래의 집계 과구독 검사 결과에 따라 결정되며, 단순한 버스트 대 링 용량 비교에 의한 것이 아닙니다.

### 어레이 fps 및 버스트 모델

어레이 설정 패널 읽는 법 (및 `lattice analyze-array` / SDK의 `analyze_array_network`):

- **버스트는 각 카메라의 실제 픽셀 형식에 따라 카메라별로 합산됩니다.**모노**M3M**카메라는**Mono12 (2 B/px)**를 스트리밍합니다;**M3C**베이어 카메라들은 8비트 또는 12비트를 스트리밍합니다(TRI032S는 BayerRG8이 요청되더라도 자동으로 BayerRG12를 출력합니다). 따라서 4대의 카메라로 구성된 풀 해상도 프레임의 크기는**모든 카메라가 8비트일 경우 ~12.6 MB이지만, 12비트 모노 카메라**를 사용할 경우 약 25 MB가 됩니다. 이 투영 기능은 모델(식별 캐시)을 기반으로 각 카메라의 포맷을 파악하므로, 버스트 데이터는 일률적인 BayerRG8 가정이 아닌, 실제 전송선로를 통해 전달되는 데이터와 일치합니다.
- **USB 이더넷 어댑터는 명판에 표기된 사양과 관계없이 최대 200 MB/s로 제한됩니다.** 링크 속도를 지속 전송 속도로 변환하는 효율성 표는 PCIe에서 파생된 것입니다. USB NIC는 *이더넷* 링크 속도를 광고하지만 USB 버스 및 드라이버의 제약에 묶여 있습니다. USB 10GbE 동글은 과거에 ~1063 MB/s의 “지속” 속도를 기록하곤 했으나 — 이 수치는 실제로 검증된 적이 없습니다 — 그로 인한 속도 조절 문제로 인해 6–18 %의 프레임을 손상시켰음에도 불구하고 정상적인 목표 fps를 보고했습니다. 현재 USB 연결 NIC는 절대적으로 **200 MB/s**로 상한이 정해져 있습니다(제한은 버스에 있으므로 명판 사양에 따라 확장되지 않습니다. USB 1 GbE 어댑터는 ~80 MB/s를 제공하며 이 제한의 영향을 받지 않습니다). 기능 레코드의 `wire_ceiling_source` 항목에 이 내용이 문구로 명시되어 있으며, `nic_is_usb`는 이를 플래그로 표시합니다. `--wire-ceiling-mbps`를 사용하여 어느 쪽이든 재정의할 수 있습니다.
- **입력 허용량은 전체 버스트 대 링 방식이 아닌 드레인 인식 방식입니다.** 동시 버스트는 전체 버스트가 아닌 *일시적 백로그* = `max(0, Σ per-cam arrival − host drain) × emit_window`에 맞기만 하면 되며, 버스트 전체가 들어갈 필요는 없습니다. 고속 호스트/저속 CAM 패브릭(**PCIe**10G 호스트 + 4× 1 GbE CAM: 도착 속도 ≈ 320 MB/s, 드레인 ≈ 1063 MB/s)에서는 호스트의 드레인 속도가 캠의 채움 속도보다 빨라 백로그가 ≈ 0이 되므로, 25 MB 버스트가 13.5 MB 링을 초과하더라도 풀 해상도 시뮬레이션 방출이**허용**됩니다. 동일한 네 대의 카메라를**USB** 10GbE 어댑터 뒤에 연결하면 드레인 속도는 200 MB/s가 되며, 1063은 아닙니다. 도착 속도가 배출 속도를 앞지르기 때문에, 손실은 프레임 속도 저하가 아닌 손상된 프레임 형태로 나타납니다. 1 GbE 호스트에서는 카메라의 31.25 MB/s DLThr 하한값으로 인해 도착 속도가 배출 속도를 앞지르게 되어 → *이* 등급의 블록에 대해서는 올바르게 **차단**됩니다 블록 클래스에 대해) 올바르게**차단**합니다(ROI를 줄이거나 ≥ 2의 비닝을 사용하십시오). 허용 여부는**두 가지** 연결 게이트 중 하나에 의해 결정되며, 다른 하나는 아래의 집계 과구독 검사입니다.
- **예상 fps는 보수적인 직렬 검색 상한치입니다.**호스트 캡처 루프는 현재 각 카메라의버퍼를**직렬 방식으로**(~카메라당 하나의 전송 창씩) 가져오므로, 사이클은 `max(readout+emit, N × emit)`에 의해 제한되며, 카메라당 전송량은 호스트 업링크가 아닌 카메라의**액세스 링크**(1 GbE ≈ 80 MB/s)에 의해 제한되며, 호스트 업링크 속도가 아닙니다. 4대의 카메라로 구성된 풀 해상도 12비트 어레이의 경우**~2.8 fps**이며, 이는 측정된 ~2.7–3.0 fps와 일치합니다. fps는 의도적으로**노출 시간에 독립적**이므로, 어두운 장면에서는 노출 시간이 길어짐에 따라 실제 값이 상한선보다 약간 낮아질 수 있습니다. 순차적 읽기가 실제 fps 제한 요인이며, 이를 병렬화하면 상한선이 단일 방출 속도에 가까워질 것입니다.
- **총 과부하(over-subscription)는 연결을 완전히 차단하는 요인입니다.**카메라당 대역폭 할당 하한은**8 MB/s**(`ARRAY_PER_CAM_FLOOR_BPS`)로 설정되어 있으므로, 하한이 고정되면 총 수요(`per_cam × N`)가**충돌 방지 유선 상한선**(`sustained × sim_emit_factor`)를 초과할 수 있습니다. 1 GbE에서 실질적인 풀 해상도 상한선:**MTU 1500인 경우 6대, 점보 프레임 사용 시 9대**입니다. 이 상한선은 회선 및 하한선 자체의 특성일 뿐이며,**프레임 크기와는 무관합니다**. 따라서**빈닝이나 ROI 크기를 줄이는 것은 도움이 되지 않습니다** (이들은 *프레임*당 바이트 수를 줄일 뿐, GevSCPD에 의해 조절되는 *초*당 바이트 수는 줄이지 않기 때문); 유일한 해결책은 카메라 수를 줄이거나, 종단 간 점보 프레임을 사용하거나, 더 빠른 NIC를 사용하는 것입니다. 이로 인한 증상은 GVSP 패킷 손실이며, 점진적인 fps 감소가 아닙니다. 따라서 `analyze-array`는 달성 가능한 fps 수치를 0으로 설정하고 `**OVER-SUBSCRIBED**`를 출력하며, 해상도가 고정된 상태에서는 `array-connect`가 **연결을 거부**합니다 (그렇지 않으면 워크다운(walk-down)이 프레임을 버닝 처리하지만, 이 경우에도 해당 유형의 블록은 해결되지 않습니다). `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1`는 연결 거부를 벤치 테스트를 위한 강력한 경고로 격하합니다 — [환경 변수](#environment-variables)를 참조하십시오.

### 어레이 상태 — 어떤 하위 시스템에서 프레임이 손실되고 있는가

연결된 어레이의 `GET /api/camera/array/<array_id>/capability`는 활성
`health` 블록을 포함하며, 이 블록은 **10초** 간격으로 재평가됩니다. 이 지표는 프레임 손실을
원인을 명시하지 않은 단일 “불완전”
비율로 보고하는 대신, 서로 반대되는 조치가 필요한 두 가지 원인으로 구분합니다:

| 필드 | 의미 | 해당 하위 시스템 |
| --- | --- | --- |
| `gvsp_corrupt_rate_pct` (시리얼별) | 프레임이 **도착했으나 구조적으로 결함이 있음**— GVSP 패킷 손실. |**네트워크**: 와이어 예산, 페이싱, NIC RX 링, MTU |
| `never_arrived_rate_pct` (시리얼별) | 프레임이 **전혀 도착하지 않음**— 카메라가 작동하지 않았거나, 카메라에서 신호가 전송되지 않았음. |**트리거/싱크**: M8 케이블, `--line`, `TriggerMode` |
| `worst_gvsp_corrupt_pct` / `worst_never_arrived_pct` | 각 항목별 최악의 카메라 전송률. | — |
| `per_cam_rate_pct` | 카메라별 통합 미완성 전송률 (두 원인 합산). | — |
| `stable_for_seconds` | 각 카메라가 0.01% 미만 상태를 유지한 기간. | — |

5%를 초과하면 백엔드는 분할 이름을 명시한 `[array-health <id>] WARN` 로그를 기록합니다 —
첫 번째 위반 시, 심각도 구간 변경 시, 해당 상태가 지속되는 동안 1분마다 한 번씩, 그리고 상태가
해소될 때 한 번씩 기록합니다. 손상된 절반은 카메라별 및
원인별 첫 번째 발생 시 `[gvsp-corrupt <SN>]`를 출력한 후, 60초마다 집계된 값을 출력합니다. 모든 평가 결과는 여전히 백엔드 로그 파일에 기록되며;
출력 내용과 관계없이 모든 버퍼에 대해 카운터가 증가합니다.

동일한 레코드에는 전체 할당량이 차지하는 양이 보고됩니다:

| 필드 | 의미 |
| --- | --- |
| `wire_ceiling_mbps` | 호스트의 현재 유효한 지속 전송 대역폭, MB/s. |
| `wire_ceiling_source` | 해당 수치의 출처(문자열) — 예: `USB-capped 200 MB/s (was theoretical 1062; PnPDeviceID=USB\VID_0BDA&PID_815A)` 또는 `user override 120 MB/s (auto said 200)`. |
| `wire_ceiling_is_user_set` | `true`일 때, `--wire-ceiling-mbps`(또는 GUI의 **Wire Budget** 필드)가 이를 설정했을 때. |
| `nic_is_usb` | USB 이더넷 어댑터의 경우 `true` — 위의 200 MB/s 제한 참조. |

**해석:** 0이 아닌 `gvsp_corrupt_rate_pct` 값과 0인 `never_arrived_rate_pct` 값이
나타난다면 트리거링과 케이블 동기화가 완벽하며, 손실의 100%가 네트워크
경로에서 발생함을 의미합니다 — `--wire-ceiling-mbps`를 낮추고 다시 연결하십시오. 반대의 패턴은
대신 동기화 케이블이나 트리거 라인을 가리킵니다.

> **`--target-fps`는 프레임 손상의 원인이 아닙니다.** GevSCPD 페이싱은
> 연결 시 한 번만 설정되므로, 트리거 속도를 낮추면 듀티 사이클은 변경되지만
> 동시 송신 버스트 속도는 변경되지 않습니다. 측정된 5배 수요 감축은 개선 효과를 보이지 않았으며,
> 와이어 상한선을 240 MB/s에서 200 MB/s로 낮추자 동일한 장비의 손실률이 10.4 %
> 손상률을 0.00 %로 낮췄습니다.

> **TRI032S 펌웨어에서는 스트림 도중 자동 축소 기능이 지원되지 않습니다.** 실행 중인 어레이는
> 이 문제를 자체적으로 해결할 수 없습니다. 연결을 끊었다가 다시 연결하여 연결 시점 선택기가
> 새로운 상한값을 반영하여 재계획할 수 있도록 하십시오.

### 증상 → 해결 방법

| 증상 (어레이 설정 / 연결 / `analyze_array_network`) | 원인 | 해결 방법 |
| --- | --- | --- |
| `FRAMES WILL DROP … exceeds NIC RX ring usable capacity 0.26 MB`, `Reduce ROI to enable` | `ReceiveBufferLen`가 32로 재설정됨(일반적으로 드라이버 업데이트 후 발생) | `ReceiveBufferLen`를 256으로, `PendingReceives`를 64로 설정하고, 패널을 다시 열기(백엔드가 이전 링 크기를 캐시한 경우 백엔드를 재시작) |
| 재시작/종료 시 응답 없음; 이후 `Invalid class` WMI 오류 발생, NIC 활성화 불가, USB 드라이브 인식 불가 | 구형 2016 Realtek USB 10GbE 드라이버 → `0x9F` BSOD → 강제 전원 끄기 | 어댑터 드라이버를 v10.67(2026) 이상으로 업데이트한 후, 위의 수신 링 설정을 다시 적용 |
| 연결은 성공하지만 네이티브 해상도 미만의 값을 반환 | Smart-prep이 회선 용량에 맞추기 위해 프레임을 자동으로 축소 | 링크 업그레이드 / 축소 허용 / `--force-tier slip-emit-and-capture` |
| 어레이가 정상적인 목표 fps를 보고하지만 실제 전달되는 값은 그 일부에 불과함; `health.gvsp_corrupt_rate_pct`가 0이 아니며, `never_arrived_rate_pct`는 0 | 호스트가 추정한 와이어 대역폭이 실제 유지 가능한 대역폭보다 과대 평가됨(USB 이더넷 어댑터, 얇은 PCIe 레인 또는 공유 패브릭에서 흔히 발생) | 더 낮은 `--wire-ceiling-mbps` 값을 낮게 설정하고 상태 블록을 다시 확인하십시오. **`--target-fps`**는 제외 — GevSCPD 페이싱은 연결 시 고정됩니다 |
| 게시된 그룹에서 카메라가 누락됨; `health.never_arrived_rate_pct` 값이 0이 아님, `gvsp_corrupt_rate_pct` 0 | 트리거/싱크 경로 — 카메라가 작동하지 않음, 네트워크 문제가 아님 | M8 싱크 케이블과 `--line`를 확인하고, 모든 구성원이 작동 준비 상태인지 (`TriggerMode=On`) |
| `**OVER-SUBSCRIBED**` / `Wire budget`가 `analyze-array`에서 초과됨, 또는 고정 해상도로 인한 연결 거부 (`array over-subscribes the wire`) | 카메라별 총 수요량(최소 8 MB/s × N대)이 충돌 방지 대역폭 상한을 초과함 — 1 GbE @1500 MTU에서 풀 해상도 카메라 6대, 점보 프레임 사용 시 9대보 적용 시 | 카메라 수를 줄이거나, 종단 간 점보 프레임을 사용하거나, 더 빠른 NIC를 사용하십시오. **ROI/비닝은 도움이 되지 않습니다** (상한은 프레임 크기와 무관합니다). `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1`는 벤치에서 우선 적용됩니다 (패킷 손실을 허용함) |

---

## `chloros-cli daq`

스펙트럼 센서 명령어. 두 가지 클래스:
- **`pool-*`**— 백엔드의 영구 풀을 통해 센서를 구동하는 얇은 HTTP 클라이언트.**이것이 지원되는 경로이며, 제공되는 CLI에 포함된 유일한 경로입니다.** 백엔드가 전송 경로를 소유하므로, GUI, CLI 및 SDK 스크립트는 모두 시리얼 포트를 놓고 경쟁하지 않고 하나의 활성 핸들을 공유합니다.
- **그 외 모든 경우**(`test`, `record`, `live`, `stream`, `connect`, `info`, `net`, `ota`, `sample-rate`, `calibrate`, `serve`, `ws`, `udp`, `mqtt`, `reflectance`, `login`, `logout`, `status`) — 직접 하드웨어 액세스 기능으로, 완전성을 위해 아래에 설명되어 있습니다. 이 기능들을 사용하려면 `daq` Python 패키지가 필요하며, 이 패키지는**출하된 어떤 아티팩트에도 포함되어 있지 않습니다**: 컴파일된 CLI에는 이 패키지가 제외되어 있습니다 (`scripts/Build-CLI.ps1`는 `--nofollow-import-to=daq`를 설정하며, 전송 프로토콜인 `pyserial` / `bleak` / `zeroconf`를 함께 전송합니다), 또한 PyPI의 SDK 패키지에도 포함되어 있지 않습니다. 이들은 소스 체크아웃을 통해서만 실행되므로, 직접 사용하기보다는 MAPIR -내부 개발 경로로 간주하십시오.
- **`discover` / `list`**는 두 경우를 모두 아우릅니다: 이들은 소스 체크아웃 시 하드웨어에 직접 전송되는 명령이지만, 출시된 빌드에서는 `pool-discover`로 대체되며 백엔드에서 스캔을 수행합니다. 따라서 스캔 기능은 어디서나 작동하며, 이는 DAQ-M의 BLE MAC을 파악할 수 있는 유일한 방법이기 때문에 매우 중요합니다.

> **`chloros-cli daq --help`**(및 `-h` / `help`)는 `pool-*` 하위 명령어들을 나열합니다 — 도움말은 의도적으로 풀 클라이언트로 의도적으로 전달되므로, 실제로 실행되는 명령을 반영합니다. 출시된 빌드에서 직접 하드웨어 하위 명령을 호출하면, 누락된 패키지를 명시하고 `pool-*`로 되돌아가도록 안내하는 명확한 오류 메시지와 함께 종료됩니다. 아무런 오류도 조용히 발생하지 않습니다. (`discover` / `list`는 예외로, `pool-discover`로 재전송되며 정상적으로 작동합니다.)
>
> **고객이 필요로 하는 모든 기능은 `pool-*`를 통해 이용할 수 있습니다** — 연결, 스트리밍, 보정된 `.daq` 파일 기록, 캡 프로파일 교체 등이 가능합니다. 또한 DAQ는 동일한 풀링된 경로를 사용하는 `chloros_sdk.connect_daq_sensor()`를 통해 Python에서 제어할 수도 있습니다.

### DAQ 센서 첫 연결 워크플로우

```bash
# 1. Smart-detect any DAQ on this machine (Ethernet → BLE → USB precedence)
chloros-cli daq connect

# 2. Detailed scan: every transport, showing the address to connect with.
#    This is how you find a DAQ-M's BLE MAC — unlike a DAQ-E hostname or a
#    DAQ-U COM port, a MAC isn't printed on the device or listed by the OS.
chloros-cli daq discover                      # or: daq pool-discover
chloros-cli daq discover --only ble           # BLE only
chloros-cli daq discover --json               # machine-readable

# 3. Open a persistent pool session (handle stays alive across CLI calls)
chloros-cli daq pool-connect           # smart-detect
chloros-cli daq pool-connect --port COM3                       # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF           # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-xxx.local        # DAQ-E by hostname

# 4. List what's in the pool, including the sensor_id you'll use next
#    (DAQ-U ids look like 'CB-7C-A8-2E-5F'; DAQ-E ids like 'daq-e-def330')
chloros-cli daq pool-list

# 5. Read the latest spectrum frame
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F

# 6. Record a calibrated .daq file for 60s
chloros-cli daq pool-record --sensor-id CB-7C-A8-2E-5F --duration 60 \
  -o ~/Documents/spectra --device-name "field-A"

# 7. Release
chloros-cli daq pool-disconnect --sensor-id CB-7C-A8-2E-5F
```

### `pool-*` 참조

| 하위 명령 | 용도 |
| --- | --- |
| `daq pool-connect` (스마트 감지) | 백엔드 풀에서 센서를 엽니다. |
| `daq pool-connect --port PORT` | 특정 직렬 포트에서 DAQ-U를 실행합니다. |
| `daq pool-connect --ble` | BLE를 통한 DAQ-M, MAC 자동 스캔. |
| `daq pool-connect --mac MAC` | 알려진 MAC을 통해 BLE로 DAQ-M, 알려진 MAC을 통해 BLE로 연결 (`--ble`를 암시). |
| `daq pool-connect --eth-host HOST` | 알려진 호스트에서 이더넷을 통한 DAQ-E. |
| `daq pool-connect --eth` | DAQ-E가 이더넷을 통해 호스트를 자동 탐지한 상태에서 실행됨(mDNS + ARP 폴백; Windows 및 Linux에서 비활성화된 ARP 캐시 상태에서도 작동). |
| `daq pool-connect --integration-time MS --frame-avg N --no-ae` | 통합 창/AE 상태 조정. |
| `daq pool-connect --no-stream` | 연결했으나 아직 스트리밍을 시작하지 않음(`pool-stream --start`로 재개). |
| `daq pool-connect --cap-id {none, fov_15, fov_30, fov_45, fov_60, fov_90, sunshine_cosine}` | 캡보정 프로필. 백엔드의 기본값은 `sunshine_cosine`입니다. |
| `daq pool-discover [--only usb,ble,eth] [--timeout SEC] [--json]` | 연결하지 않은 상태에서 연결 가능한 센서를 찾기 위해 모든 전송 경로를 스캔합니다. **이 방법으로 DAQ-M의 BLE MAC을 찾을 수 있습니다.** `daq discover` / `daq list`는 출하된 빌드에서 자동으로 여기로 라우팅됩니다. 풀에 이미 열려 있는 센서는 목록에 표시되지 않습니다(연결된 DAQ-M은 광고를 중지하기 때문). 따라서 해당 센서를 확인하려면 `pool-list`를 사용하십시오. |
| `daq pool-list` | 백엔드 풀에 있는 모든 센서를 표시합니다. |
| `daq pool-disconnect --sensor-id ID [--all]` | 해제. |
| `daq pool-latest --sensor-id ID [--recent N] [--json]` | 가장 최근의 N개 스펙트럼 프레임. |
| `daq pool-stream --sensor-id ID [--start \| --stop]` | 스트리밍 재개/일시 중지. |
| `daq pool-record --sensor-id ID [--duration SEC] [--output DIR] [--device-name NAME] [--stop]` | .daq 녹화 시작/중지. |
| `daq pool-set-cap --sensor-id ID --cap-id CAP` | 실행 중에 캡 보정 프로필을 교체합니다. |

### 직접 하드웨어 하위 명령어 (소스 체크아웃 전용 — 출시된 빌드에는 포함되지 않음)

> 완전성을 위해 나열되었습니다. 이 명령어들을 사용하려면 `daq` Python 패키지와 `pyserial` / `bleak` / `zeroconf`가 필요하며, 이 중 어느 것도 컴파일된 CLI 또는 PyPI SDK에 포함되어 있지 않으며, MAPIR 소스 체크아웃에서만 실행됩니다. **Chloros 릴리스 빌드를 사용 중인 경우, 대신 위의 `pool-*` 명령어를 사용하십시오**; 이 명령어들은 connect, stream, record 및 캡처 선택 기능을 지원합니다.

```bash
chloros-cli daq test --port COM3                           # Verify connection
chloros-cli daq connect --eth                              # Smart-detect over ETH
chloros-cli daq info --eth-host daq-e-xxx.local            # Device summary as JSON
chloros-cli daq discover --only usb,ble --timeout 5        # Scan local interfaces
chloros-cli daq list                                       # Alias of discover
# ^ discover/list are the exception in this section: in a shipped build they
#   fall back to `pool-discover` (the backend does the scan), so they work
#   without a source checkout. The only difference is that the fallback needs
#   the Chloros backend running, as all pool-* commands do.

# Streaming JSON Lines to stdout (pipeable)
chloros-cli daq stream --port COM3 --format jsonl --photometrics

# Record to .daq for 60 seconds
chloros-cli daq record --port COM3 --duration 60 -o ~/Documents/spectra/

# Live spectrum visualization in a window
chloros-cli daq live --port COM3 --record

# Dual-sensor reflectance (ambient + object) → JSON Lines
chloros-cli daq reflectance \
  --ambient-eth-host daq-e-field.local \
  --object-eth-host daq-e-canopy.local \
  --record -o ~/Documents/reflectance/

# Convenience: pick integration_time + frame_avg for a target rate
chloros-cli daq sample-rate --port COM3 --target-hz 5

# Calibration profile management
chloros-cli daq calibrate --port COM3 --list
chloros-cli daq calibrate --port COM3 --set field_calibration_2026_05

# DAQ-E network config (mDNS auto-discovers the host)
chloros-cli daq net --eth-host daq-e-xxx.local set-ip --mode static --ip 192.168.2.20
chloros-cli daq net --eth-host daq-e-xxx.local set-name "sky-sensor"
chloros-cli daq net --eth-host daq-e-xxx.local set-ptp --enabled true --domain 0
chloros-cli daq net --eth-host daq-e-xxx.local set-auto-stream true          # auto-stream on boot
chloros-cli daq net --eth-host daq-e-xxx.local set-require-signature         # require factory-signed cal (fw v1.6.0+; refused while the held cal is unsigned)
chloros-cli daq net --eth-host daq-e-xxx.local set-time                      # push host clock (refused when PTP SLAVE)
chloros-cli daq net --eth-host daq-e-xxx.local set-auth-token --current "" --new "s3cret"   # control-channel auth ("" new = disable)
chloros-cli daq net --eth-host daq-e-xxx.local set-ota-password "newpass"    # change OTA password (min 4 chars)
chloros-cli daq net --eth-host daq-e-xxx.local factory-reset                 # clear all NVS settings and reboot
chloros-cli daq net --eth-host daq-e-xxx.local reboot

# OTA firmware update
chloros-cli daq ota --eth-host daq-e-xxx.local \
  --firmware daq_e_1.21.bin --password mapir-daq-e

# Bridge spectra to other protocols
chloros-cli daq serve --port COM3 --tcp-port 9000           # TCP JSON-lines
chloros-cli daq ws    --port COM3 --ws-port 9001            # WebSocket
chloros-cli daq udp   --port COM3 --udp-port 9002           # UDP broadcast
chloros-cli daq mqtt  --port COM3 --broker mqtt.example.com --topic daq/spectrum
```

---

## `chloros-cli project`

저장된 Chloros 프로젝트(`cameras.json` + `sensors.json` + `project.json`가 포함된 폴더가 포함된 폴더)를 열고, 연결한 후 실행합니다. 모든 처리는 백엔드를 통해 이루어지므로 GUI와 CLI에서 표시되는 하드웨어 상태는 동일합니다.

### 하위 명령어 참조

| 하위 명령어 | 용도 |
| --- | --- |
| `project open PATH` | 프로젝트의 장치 매니페스트(카메라, 어레이, 센서)를 출력합니다. |
| `project devices PATH [--reconnect]` | 탐색 결과를 나열하거나 재실행합니다. |
| `project connect PATH [--cameras-only] [--sensors-only]` | 저장된 모든 카메라/ 어레이 / 센서에 연결합니다. |
| `project capture PATH NAME [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--prefix P]` | 지정된 카메라 또는 어레이에서 단일 캡처 수행합니다. |
| `project burst PATH NAME [-n N] [-i S] [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--prefix P]` | 지정된 카메라 또는 어레이에서 N 프레임 버스트 촬영 (`-n/--count` 기본값 5; `-i/--interval` 프레임 간 간격(초), 기본값 0). 어레이 버스트는 반복적으로 동기화된 그룹을 중복 제거하므로(오래된 데이터 감시 기능), 부분 순환 어레이가 하나의 프레임을 N개 복사하여 반환할 수 없습니다; 반복마다 결과를 출력합니다. |
| `project stream PATH NAME [-n N] [--fps F] [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--poll-interval S]` | 백엔드 작업을 통한 스트림-투-디스크. `--poll-interval` = `/stats` 폴링 간격(초, 기본값 2.0). |
| `project sensor read PATH NAME [--json]` | 최신 스펙트럼 프레임. |
| `project sensor log PATH NAME --seconds SEC [-o DIR] [--device-name NAME]` | .daq 파일 기록. |
| `project run PATH RECIPE.yaml` | YAML/JSON 캡처 레시피 실행. `--dry-run`는 실행하지 않고 유효성 검사만 수행합니다. |
| `project align calibrate PATH NAME [--method M] [--model M] [--frames N] [--reference SN] [--max-features N] [--ratio-threshold F] [--ransac-threshold-px F] [--min-matches N] [--max-reproj-err-px F] [--checkerboard RxC] [--name PROFILE]` | 어레이에 대한 정렬 계산 — [아래 플래그 표 ](#project-align-calibrate-options)를 참조하십시오. |
| `project align status PATH NAME [--json]` | 현재 정렬 프로파일을 출력합니다. |
| `project align clear PATH NAME` | 캐시된 프로필을 삭제합니다. |
| `project align tweak PATH NAME --serial SN --dx N --dy N --rotation-deg N --scale N` | 하나의 슬레이브’변환을 미세 조정합니다. |
| `project align export PATH NAME --to FILE` | 프로파일을 JSON에 저장합니다. |
| `project align import PATH NAME --from FILE [--no-validate]` | 저장된 프로파일을 불러옵니다. |

#### `project align calibrate` 옵션

| 플래그 | 기본값 | 설명 |
| --- | --- | --- |
| `--method {feature_orb, feature_akaze, phase_correlation, checkerboard, manual}` | `feature_orb` | 정렬 방법. **이 표기법은 `lattice align-calibrate`**와 다르며, `lattice align-calibrate`는 `orb` / `akaze` / `phase`와 같은 약어를 사용합니다. 이 두 명령어는 는 이 플래그에서 상호 교환할 수 없습니다. |
| `--model {translation, rigid, affine, homography}` | `affine` | 모델을 맞추도록 변환합니다. |
| `--frames N` | `1` | 프레임 스냅샷을 평균값에 동기화. |
| `--reference SN` | 마스터 | 참조 카메라 일련번호; 다른 모든 멤버는 이에 맞춰 왜곡됩니다. |
| `--max-features N` | `5000` | ORB 특징점 수 상한. |
| `--ratio-threshold F` | `0.75` | 로우(Lowe)의 비율 테스트. |
| `--ransac-threshold-px F` | `3.0` | RANSAC 내점 임계값. |
| `--min-matches N` | `15` | **품질 게이트** — 내점 일치 수가 이 수보다 적을 경우 해법을 거부합니다. |
| `--max-reproj-err-px F` | `4.0` | **품질 기준** — 이 RMS 재투영 오차 이상일 경우 해를 거부합니다. |
| `--checkerboard RxC` | — | `--method checkerboard`에 대한 보드 기하 구조, 예: `9x6`. |
| `--name PROFILE` | 비어 있음 | 저장된 JSON에 포함된 프로파일 이름. **배열 이름이 아님** — 배열 이름은 위치 기반 `NAME`입니다. |

이 두 가지 품질 게이트 때문에 보정(calibrate)이 해결에 성공했음에도 불구하고
저장이 거부되는 이유입니다. 두 품질 게이트 중 하나라도 실패한 프로필은 이후의 모든
캡처를 아무런 경고 없이 잘못 등록하게 되므로, 프로필은 저장되지 않고 거부됩니다.

### 예시

```bash
# Open a project and see what it knows about
chloros-cli project open "/home/user/Chloros Projects/Field_A"

# Connect everything saved in the project
chloros-cli project connect "/home/user/Chloros Projects/Field_A"

# Capture from a named camera (defined in cameras.json)
chloros-cli project capture "/home/user/Chloros Projects/Field_A" FrontLeft \
  -o output/ --format tiff

# Capture from a named array
chloros-cli project capture "/home/user/Chloros Projects/Field_A" main_rig \
  -o output/ --format tiff

# Capture with overrides
chloros-cli project capture "/home/user/Chloros Projects/Field_A" main_rig \
  --exposure 5000

# Read a spectrum
chloros-cli project sensor read "/home/user/Chloros Projects/Field_A" Sky --json

# Record a DAQ log
chloros-cli project sensor log "/home/user/Chloros Projects/Field_A" Sky \
  --seconds 120 -o ~/Documents/spectra/

# Align an array (live)
chloros-cli project align calibrate "/home/user/Chloros Projects/Field_A" main_rig
chloros-cli project align status "/home/user/Chloros Projects/Field_A" main_rig

# Run a recipe
chloros-cli project run "/home/user/Chloros Projects/Field_A" recipe.yaml
```

### 레시피 DSL

`project run RECIPE.yaml`는 일련의 동작을 설명하는 YAML 또는 JSON 파일을 지원합니다:

```yaml
# recipe.yaml
overrides:
  cameras:
    FrontLeft:
      exposure_us: 5000
      target_brightness: 80

stop_on_error: true
actions:
  - apply:
      name: FrontLeft
      settings:
        exposure_auto: "Off"
        gain: 6.0
        gain_auto: "Off"
  - wait: 2s
  - capture:
      name: FrontLeft
      output: pose_a/
      format: tiff
  - stream:
      name: main_rig
      count: 60
      fps: 5
      output: stream/
  - burst:
      name: main_rig
      count: 10
      interval: 0.5
      output: burst_a/
      format: tiff
  - sensor:
      name: Sky
      action: read
```

지원되는 동작: `apply`, `wait`, `capture`, `stream`, `burst`, `sensor`. `burst` 작업은 `name`(필수), `count`(기본값 5), `interval`(초, 기본값 0), `output`, `format`, 및 `settings`(`apply`와 동일한 카메라별 설정 구조)를 사용합니다. 배열 버스트는 `project burst`와 동일한 새로 동기화된-그룹 워치독을 사용합니다.

실행 방법:

```bash
chloros-cli project run "/path/to/project" recipe.yaml

# Dry-run to validate without firing hardware
chloros-cli project run "/path/to/project" recipe.yaml --dry-run
```

---

## 환경 변수

| 변수 | 효과 |
| --- | --- |
| `CHLOROS_BACKEND_URL` | 백엔드 URL을 재정의합니다(기본값: `http://127.0.0.1:5000`) — **이 변수는 `lattice`, `project` 및 `daq pool-*` 명령어 계열에서만 적용됩니다.** 핵심 명령어(`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`)는 `http://127.0.0.1:<port>`에 연결하고 이 변수를 무시합니다(IPv4 리터럴은 Windows `localhost`→`::1` ~2 s-per요청당 약 2초의 페널티)를 우회하므로, 항상 로컬 머신을 대상으로 합니다. |
| `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED` | `1`는 어레이 과부하 연결 거부 (`pin_resolution` 설정 시 CAM당 총 수요 &gt; 충돌 방지 와이어 상한)을 GVSP 패킷 손실을 감수하는 &#x27;경고 후 진행(loud warning-and-proceed)&#x27;으로 하향 조정합니다. 벤치마크 전용 — [어레이 fps 및 버스트 모델](#array-fps--burst-model) 참조. |
| `CHLOROS_CLI_MODE` | CLI 자체에서 설정하며, 백엔드에 병렬 처리를 활성화하라고 지시합니다. |
| `CHLOROS_GVSP_PROBE_FALLBACK` | `0`는 GVSP 폴백 프로브를 건너뜁니다 (ICMP 결과만 해당). **이 설정은 점보 패킷을 비활성화하며, 단순히 로그 출력을 줄이는 것이 아닙니다** — 카메라는 모든 경로에서 최대 1500까지의 DF 핑에만 응답하므로, 이 프로브만이 점보 패킷을 감지할 수 있습니다. 연결당 카메라 1대당 약 1초를 절약하며, 네트워크가 점보 패킷을 *전송할 수 있었다면* 약 1.네트워크가 *잠보 패킷을 전송할 수 있는* 경우, 유선 상한치의 45배에 해당하는 비용이 발생합니다. 이 설정을 지정하면 SDK에서 경고가 표시됩니다. |
| `CHLOROS_GVSP_PACKET_SIZE_FORCE` | GVSP 패킷 크기를 N 바이트로 고정하고, 프로빙을 완전히 건너뜁니다. 영구적으로 설정하는 것보다 명령어별 설정(`CHLOROS_GVSP_PACKET_SIZE_FORCE=9000 chloros-cli …`)를 사용하는 것이 영구적으로 설정하는 것보다 낫습니다. 크기를 고정하면 앞쪽 네트워크에 적응하지 못하게 되며, 점보 프레임을 전송할 수 없는 경로에서 9000으로 고정하면 **모든** 캡처가 `SC_ERR_TIMEOUT -1011` 오류로 인해 타임아웃됩니다. |
| `TMPDIR` (Linux) | Nuitka onefile 추출 디렉터리를 재정의합니다. CLI가 존재하면 자동으로 `/mnt/ssd/tmp`를 사용합니다. |

---

## 종료 코드

| 코드 | 의미 |
| --- | --- |
| `0` | 성공. |
| `1` | 일반적인 오류 (대부분의 하위 명령어 오류). |
| `2` | 인자 오류. |
| `130` | Ctrl+C로 중단됨. |

---

## 문제 해결 요령

- **&quot;로그인 필요&quot;** → 이 컴퓨터에서 `chloros-cli login EMAIL PASSWORD`를 한 번 실행하십시오.
- **&quot;백엔드에 연결할 수 없음&quot;** → Chloros 데스크톱 앱을 시작하거나, 백엔드 바이너리(`chloros-backend`)를 직접 실행하거나, 원격인 경우 `CHLOROS_BACKEND_URL`를 확인하십시오.
- **`lattice` 명령어가 &quot;LATTICE 카메라 드라이버를 찾을 수 없음&quot; 오류로 실패함** → Arena SDK 런타임이설치되어 있지 않습니다. CLI는 Windows에 `win32api`와 함께 번들로 제공되지만, C 런타임은 GUI 설치 프로그램의 일부입니다.
- **Array connect / Array Settings에 &quot;FRAMES WILL DROP&quot; 또는 &quot;Reduce ROI to enable&quot;이 표시됨** → 호스트 NIC 수신 링 크기가 너무 작습니다(일반적으로 NIC 드라이버 업데이트 후 32로 재설정됨). [호스트 NIC 설정 및 튜닝](#host-nic-setup--tuning-lattice-arrays) 참조 — `ReceiveBufferLen=256`, `PendingReceives=64`를 설정하십시오.
- **재시작/종료 시 시스템이 응답하지 않음, 그 후 WMI `Invalid class` / NIC 활성화 불가 / USB 드라이브 미탐지** → 구형 USB 10GbE 어댑터 드라이버로 인해 `DRIVER_POWER_STATE_FAILURE` 오류 발생 (BSOD `0x9F`). 어댑터 드라이버를 업데이트하십시오 — [호스트 NIC 설정 및 튜닝](#host-nic-setup--tuning-lattice-arrays)을 참조하십시오.
- **Jetson 스왑 경고** → 파일 기반 스왑을 추가하십시오. CLI 페이지에 정확한 `fallocate` / `swapon` 명령어가 나와 있습니다.
- **DAQ 직접 명령어 누락** → 예상된 사항: 제공된 `chloros-cli`는 의도적으로 `daq` 패키지를 제외하므로, `pool-*`만 존재합니다 (PyPI SDK에서도 제공되지 않습니다). 백엔드를 통해 동일한 센서를 제어하는 `pool-*`를 사용하거나, Python에서 제공하는 `chloros_sdk.connect_daq_sensor()`를 사용하십시오.

---

## 참조

- [Python SDK 참조](sdk-reference.md) — 모든 CLI 명령에 상응하는 프로그래밍 방식.
- [DAQ 센서 가이드](../daq/README.md) — 센서별 배선 및 보정.
- 온라인 문서: `https://mapir.gitbook.io/chloros/cli`
