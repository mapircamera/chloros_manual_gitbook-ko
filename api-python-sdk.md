# API : Python SDK

{% hint style="info" %}
**API에 대한 전체 내용을 찾고 계신가요?** 이 페이지는 실습 튜토리얼입니다. 모든 공개 클래스, 메서드, 정확한 시그니처, 그리고 복사하여 붙여넣을 수 있는 예제는 AI 어시스턴트에 최적화된 [SDK 참조](reference/sdk-reference.md)에 수록되어 있습니다.**AI 어시스턴트를 사용하고 계신가요?** 채팅창에 이 URL를 붙여넣으면, 최신 버전의 Chloros 1.2.0 API 전체 내용을 확인할 수 있습니다:

`https://mapir.gitbook.io/chloros/reference/sdk-reference.md`

이 설명서의 모든 페이지는 소문자 슬러그 + `.md` 형식으로 원본 마크다운 파일로 제공되며, 설명서 전체는 `https://mapir.gitbook.io/chloros/llms.txt`에서 색인되어 있습니다.
{% endhint %}

**Chloros Python SDK** (PyPI의 `chloros-sdk`)는 Python에서 데스크톱 앱이 수행할 수 있는 모든 기능, 즉 일괄 이미지 처리, LATTICE 카메라 및 어레이 실시간 제어, DAQ 광센서 세션, 저장된 프로젝트 자동화를 구동합니다. 이는 GUI와 CLI가 사용하는 것과 동일한 로컬 백엔드(`127.0.0.1:5000`상의 HTTP) 위에 구축된 얇은 레이어이므로, 세 가지 환경 모두에서 동작이 동일합니다.

## 설치

설치는 두 단계로 이루어집니다. 먼저 Chloros 데스크톱 패키지(처리 백엔드 및 하드웨어 런타임을 제공함)를 설치한 다음, Python 패키지를 설치합니다.

**1단계 — Chloros 설치.** Windows: [다운로드](download.md) 페이지에서 데스크톱 설치 프로그램(기본 경로: `C:\Program Files\MAPIR\Chloros\`)을 실행합니다. Linux: `.deb` 패키지를 설치하십시오([Linux 설치](linux/linux-installation.md)).**2단계 — SDK 설치** (Python 3.7 이상):

```bash
pip install chloros-sdk
```

pip가 필요하지 않을 수도 있습니다. 모든 설치 프로그램에는 호환되는 SDK 휠이 포함되어 있습니다. Windows 설치 프로그램은 이를 시스템의 Python에 자동으로 설치하며; Linux `.deb` 설치 프로그램은 이를 `/usr/lib/chloros/sdk/`에 배치하고 정확한 `pip install --user` 명령어를 출력합니다. PyPI는 릴리스 빌드 시 업데이트되므로, `pip install chloros-sdk`는 최신 안정 버전을 반영합니다.

**3단계 — 각 컴퓨터에서 한 번씩 로그인:**

```bash
chloros-cli login user@example.com 'YourPassword'
```

인증 정보는 `~/.chloros/`에 캐시됩니다(두 플랫폼 모두). Windows에서는 데스크톱 앱의 ‘사용자<img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line">’ 탭을 통해 동일하게 로그인할 수 있습니다. SDK의 경우 유료 Chloros+ 요금제가 필요합니다 — 아래 [라이선스 요구 사항](#license-requirement)을 참조하십시오.

| 요구 사항 | 세부 정보 |
| --- | --- |
| **Chloros 설치** | Windows: 데스크톱 설치 프로그램; Linux: `.deb` 패키지(백엔드 바이너리 제공) |
| **Python** | 3.7 이상 (3.10에서 개발 및 테스트됨) |
| **운영 체제** | Windows 10/11 64비트, Ubuntu 22.04 LTS 이상, 또는 NVIDIA Jetson (JetPack 6) |
| **라이선스** | 유효한 Chloros+ 로그인, 유료 요금제(Copper 이상) |

## 60초 만에 완료

단 한 번의 호출로 프로젝트를 생성하고, 폴더를 가져오며, 처리를 구성하고, 파이프라인을 실행합니다. 백엔드가 아직 실행 중이지 않은 경우 자동으로 시작됩니다:

```python
import chloros_sdk

results = chloros_sdk.process_folder(
    "C:/DroneImages/Flight001",
    indices=["NDVI", "NDRE", "GNDVI"],
)
```

(Linux에서는 Linux 경로를 사용하세요: `/home/user/drone_images/flight001`. SDK는 두 플랫폼에서 동일하게 작동합니다.)

LATTICE 캡처 폴더를 처리 중이신가요? LATTICE에 최적화된 래퍼를 사용하세요. 이 래퍼는 적절한 기본값(패널 대상 감지 없음, 표준 디베이어)을 적용합니다:

```python
results = chloros_sdk.process_lattice_capture(
    folder_path="C:/Captures/2026-05-13_Field",
    indices=["NDVI"],
)
```

## `ChlorosLocal` — 전체 파이프라인 제어

한 줄로 처리할 수 없는 작업의 경우, `ChlorosLocal`를 사용하십시오. 이 명령어는 첫 실행 시 백엔드(`auto_start_backend=True`)를 시작하고, 프로젝트를 생성 및 구성하며, 진행 상황을 모니터링하고, 실행 후 요약 정보를 반환합니다.

```python
ChlorosLocal(
    api_url="http://127.0.0.1:5000",   # backend URL (also: backend_url=)
    auto_start_backend=True,            # spawn backend if not running
    backend_exe=None,                   # override backend binary path
    timeout=30,                         # request timeout seconds
    backend_startup_timeout=60,         # backend boot timeout
    processing_timeout=14400,           # hard cap on process() (4 h)
    processing_stuck_timeout=1800,      # no-progress threshold (30 min)
)
```

{% hint style="info" %}
`localhost`로 대체하지 말고 기본값인 `http://127.0.0.1:5000`를 유지하십시오. Windows에서, `localhost`는 먼저 `::1`로 변환되며, IPv4 전용 백엔드에서는 요청당 약 2초가 소요됩니다.
{% endhint %}

확실한 정리 처리를 위해 컨텍스트 매니저로 사용하십시오:

```python
import chloros_sdk

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26", camera="Survey3N_RGN")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "NDRE", "GNDVI"],
        export_format="TIFF (16-bit)",
    )
    results = cl.process(mode="parallel", wait=True)
print(results["summary"])
```

`configure()`는 다음 키워드를 지원합니다: `debayer`, `vignette_correction`, `reflectance_calibration`, `indices`, `export_format`, `ppk`, `daq_log_path`, `input_level`, `radiometric_output`, `array_alignment`, `array_alignment_crop`, `array_alignment_interpolation`, 및 `custom_settings`. 주요 값:

```python
# export_format
"TIFF (16-bit)"           # default, recommended
"TIFF (32-bit, Percent)"  # reflectance percentage as float32
"PNG (8-bit)"
"JPG (8-bit)"

# debayer
"High Quality (Faster)"                  # standard, default
"Texture Aware (Slow, Highest Quality)"  # neural debayer, Chloros+ only
```

LATTICE 전용 노브 (`input_level`, `radiometric_output`, `array_alignment*` 계열)에 대한 자세한 설명과 전체 값 표는 [SDK 참조](reference/sdk-reference.md#supported-values)에 전체 값 표와 함께 문서화되어 있습니다.

### 진행 상황 확인

```python
def show_progress(percent, message):
    print(f"[{percent:3d}%] {message}")

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(indices=["NDVI"])
    cl.process(progress_callback=show_progress, poll_interval=1.0)
```

### 실행 후 요약 읽기 — 및 빈 실행 감지

실행이 완료되면, `process()`는 백엔드의 처리 요약 정보를 `result["summary"]`로 첨부합니다. `summary["hints"]`의 각 항목은 주목할 만한 사항 — 예를 들어, 실행 결과가 0인 이유 등 — 모든 힌트는 Python `UserWarning` 형태로 재전송되므로, 딕셔너리를 직접 확인하지 않더라도 빈 실행은 자동으로 진단됩니다:

```python
result = cl.process()
for hint in result.get("summary", {}).get("hints", []):
    print("HINT:", hint)
# hints also arrive on the warnings channel:
#   python -W always::UserWarning your_script.py
```

{% hint style="warning" %}
**실행 결과 이미지가 생성되지 않았을 때 `process()`는 발생하지 않습니다.** 이것이 바로 SDK와 CLI가 의도적으로 다른 유일한 부분입니다: `chloros-cli process`는 &quot;결과물이 요청되었으나 아무것도 작성되지 않음&quot;을 오류로 간주하고 0이 아닌 값으로 종료하는 반면, SDK는 정상적으로 반환하고 `summary` 또는 힌트를 통해 해당 상태를 보고합니다. 파이프라인이 빈 실행 시 중단되어야 한다면, 예외 처리에만 의존하지 말고 직접 확인하십시오 — `summary`를 확인하거나(또는 프로젝트 폴더 내 파일 수를 세어보십시오).
{% endhint %}

## Smart Connect — 라이브 하드웨어

세 가지 헬퍼가 백엔드 하드웨어 풀에서 지속적 세션을 엽니다. 이 풀은 GUI가 사용하는 풀과 동일하므로, SDK 스크립트는 직렬 포트나 네트워크 대역폭을 놓고 경쟁하지 않고 데스크톱 앱과 공존합니다. 세 가지 모두 실행 중인 로컬 백엔드가 없으면 자동으로 로컬 백엔드를 시작합니다.

### 단일 LATTICE 카메라 — `connect_camera`

```python
import chloros_sdk

# Open by serial; reuses existing pool entry if one exists
with chloros_sdk.connect_camera("213800234") as cam:
    cam.set_settings(exposure_time=10000, gain=0.0)   # microseconds, dB
    cam.capture("output/")
```

### 동기화된 어레이 — `connect_array`

`connect_array`는 다중 카메라 리그에 권장되는 진입점입니다. 이 스크립트는 GUI와 동일한 스마트 준비(smart-prep) 흐름을 실행합니다: 네트워크 분석, 동기화 계층 자동 선택, PTP 시간 동기화, 카메라별 픽셀 형식 선택, AE 시딩, GPIO 트리거 준비. **첫 번째 시리얼이 마스터**이며(하드웨어 트리거 펄스를 발사함), 나머지는 슬레이브입니다.

```python
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    arr.capture("output/", processing="reflectance")
```

트리거를 발동하기 전에 모든 카메라에서 자동 노출이 안정화될 때까지 기다리려면, 어떤 어레이 캡처에든 `smart=True`를 추가하십시오. 캡처 모드(단일 / 연속 / 간격 / 최속), 레코더, 버스트-투-비디오 및 어레이 정렬에 대해서는 [SDK 참조](reference/sdk-reference.md#synchronized-array--arraysession-smart-prep)를 참조하십시오.

### DAQ 광 센서 — `connect_daq_sensor`

인수가 없으면, `connect_daq_sensor()`는 전송 방식을 자동으로 감지합니다(우선순위: 이더넷 → BLE → USB):

```python
with chloros_sdk.connect_daq_sensor() as daq:    # smart-detect USB / BLE / ETH
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])
```

각 프레임은 135포인트의 `spectrum`(보정 시 W/m²/nm), `is_saturated` 플래그, CIE `x`, `y`, `z` 값을 포함합니다. 특정 센서나 전송 방식을 지정하려면 — 이더넷 자동 검색이 첫 번째 시도에서 정상 작동 중인 DAQ-E를 놓칠 수 있는, 여러 네트워크 인터페이스를 갖춘 호스트에서는 이 방법이 신뢰할 수 있는 선택입니다 — 다음의 명시적 힌트를 전달하십시오:

```python
daq = chloros_sdk.connect_daq_sensor(transport="usb", port="COM3")
daq = chloros_sdk.connect_daq_sensor(mac="AA:BB:CC:DD:EE:FF")        # implies BLE
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-xxx.local")     # implies Ethernet
```

대소문자 보정 프로필(`cap_id`)은 **SDK** 설정 항목이 아니므로, 대신 `chloros-cli daq pool-connect --cap-id …` / `pool-set-cap`를 통해 선택하십시오.

### 저장된 프로젝트 — `open_project`

저장된 Chloros 프로젝트는 연결된 하드웨어(`cameras.json` + `sensors.json` 및 `project.json`)를 유지하며, `chloros_sdk.open_project(path)`는 모든 장치를 한 번에 재연결하고 장치 이름으로 캡처를 실행할 수 있습니다. 참조 문서의 [프로젝트 자동화](reference/sdk-reference.md#project-automation--chlorosproject)를 참조하십시오.

## pip 전용 설치 시 제공되는 기능

하드웨어 표면을 사용하기 전에 모듈 수준의 가용성 플래그를 확인하십시오:

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)    # True iff lattice_sdk imported cleanly
print("DAQ_AVAILABLE    =", chloros_sdk.DAQ_AVAILABLE)       # True iff daq_sdk imported cleanly
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)  # True iff ChlorosProject deps available
```

**오직** `pip install chloros-sdk`만 설치되어 있고 Chloros 데스크톱 패키지가 없는 호스트의 경우:

* `ChlorosLocal`, `process_folder` 및 `process_lattice_capture`는 **작동하지 않습니다** — 데스크톱 설치 프로그램에 포함된 백엔드 바이너리가 필요합니다.
* 스마트 커넥트 헬퍼(`connect_camera`, `connect_array`, `connect_daq_sensor`)는 순수한 HTTP 클라이언트이므로 다른 컴퓨터에 있는 백엔드와 연동되어 작동합니다. 하지만 제공된 백엔드는 루프백에만 바인딩되므로, 사용자는 직접 포트를 포워딩해야 합니다(예: `ssh -N -L 5000:127.0.0.1:5000 user@chloros-host`)하고, `backend_url="http://127.0.0.1:5000"`를 `auto_start_backend=False`와 함께 전달해야 합니다. [원격 백엔드 모드](reference/sdk-reference.md#remote-backend-mode-pip-only-host-via-tunnel)를 참조하십시오.
* 직접 하드웨어 기반 LATTICE 클래스(`LatticeCamera`, `CameraPool`, …)는 임포트할 수 있지만, 데스크톱 패키지에 포함된 Arena SDK 런타임이 필요합니다. 이 런타임이 없으면 `CAMERA_AVAILABLE`는 `False`와 동일합니다.
* `daq_sdk`(직접 DAQ 클래스)는 PyPI 패키지가 아닌 데스크톱 설치본에 포함되어 있으므로, 따라서 pip만 설치된 호스트에서는 `DAQ_AVAILABLE`가 `False`로 작동합니다. 대신 (터널링된) 백엔드를 대상으로 `connect_daq_sensor()`를 통해 DAQ 센서를 제어하십시오.

## 라이선스 요구 사항

SDK에 액세스하려면 유료 등급(**Copper 이상**: Copper / Bronze / Silver / Gold) 중 하나에서 유효한 Chloros+ 로그인이 필요합니다. 무료 Iron 등급에서는 SDK/CLI에 액세스할 수 없습니다. 이 제한은**서버 측**에서 적용됩니다: 모든 SDK 요청은 활성 세션과 유료 요금제를 모두 포함해야 하며, 그렇지 않을 경우 백엔드는 `403` / `PLAN_UPGRADE_REQUIRED` (`ChlorosLocal`에 의해 `ChlorosLicenseError`로, `connect_*` 헬퍼에 의해 `ChlorosConnectError`로 발생)를 반환합니다. 로그아웃된 호출자는 대신 `401` / `AUTH_REQUIRED` (`ChlorosAuthenticationError`)를 받게 됩니다. `chloros-cli login`를 다시 실행하면 첫 번째 경우는 해결되지만 두 번째 경우는 해결되지 않습니다.

오프라인 사용은 요금제의 유예 기간 내에 가능합니다. 티어는 서버 검증 캐시(5분) 또는 서명되고 기기에 바인딩된 라이선스 캐시(월간 요금제의 경우 30일, 연간 요금제의 경우 구독 만료일까지)에서 읽힙니다. 유예 기간이 만료되면 요금제는 무료 요금제로 전환되며, 기기가 서버에 한 번 연결될 때까지 SDK 액세스가 중지됩니다. `chloros-cli status`는 무료 요금제에서도 계속 접근 가능하므로 원인을 항상 확인할 수 있습니다. [Chloros+ 로그인](chloros+-login.md)을 참조하십시오.

## 예외

&quot;Chloros에서 발생한 모든 오류&quot;를 처리하기 위해 기본 클래스를 캐치하십시오:

```python
import chloros_sdk

try:
    chloros_sdk.process_folder("/path/to/folder")
except chloros_sdk.ChlorosAuthenticationError:
    print("Run `chloros-cli login` first.")
except chloros_sdk.ChlorosLicenseError:
    print("Chloros+ subscription required.")
except chloros_sdk.ChlorosError as e:
    print(f"Chloros error: {e}")
```

모든 파이프라인 예외(`ChlorosBackendError`, `ChlorosConnectionError`, `ChlorosLicenseError`, `ChlorosAuthenticationError`, `ChlorosConfigurationError`, `ChlorosProcessingError`)는 모두 `ChlorosError`에서 파생됩니다. 주의할 점 하나: `ChlorosConnectError` — 이 예외는 오직 `connect_camera` / `connect_array` / `connect_daq_sensor` —는 일반 `Exception`에서 파생되며, `ChlorosError`에서 파생된 것이 **아닙니다**. 따라서 `except ChlorosError`는 이를 감지하지 못합니다. 전체 계층 구조는 [SDK 참조](reference/sdk-reference.md#exceptions)에 나와 있습니다.

## 참조

* [SDK 참조](reference/sdk-reference.md) — AI 어시스턴트에 최적화된 완전한 API 표면.
* [CLI 참조](reference/cli-reference.md) — 모든 CLI 하위 명령어는 SDK 호출을 반영합니다.
* [다운로드](download.md) — Windows 및 Linux용 설치 프로그램.
