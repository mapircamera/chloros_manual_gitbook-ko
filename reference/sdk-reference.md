# Chloros Python SDK 참조

**버전:**

1.2.0**생성일:**2026-07-29 19:19 ·**수정일:** 2026-08-30**패키지:** `chloros-sdk` (PyPI)**대상:** LLM 활용에 최적화되었으며, 사람이 읽기 쉬운 형식입니다.**범위:** `import chloros_sdk`에서 제공하는 모든 공개 클래스, 함수 및 헬퍼를 다루며, 이미지 처리, 단일 카메라 제어, 동기화된 배열, DAQ 센서, 프로젝트 자동화를 다루는 복사-붙여넣기 가능한 예제가 포함되어 있습니다.

주요 내용만 확인하시려면 다음으로 이동하세요:
- [설치 및 빠른 시작](#installation)
- [LATTICE 어레이용 Smart-Connect](#smart-connect-for-lattice-cameras)
- [DAQ 센서 세션](#daq-sensor-sessions)
- [프로젝트 자동화](#project-automation--chlorosproject)
- [Smart-AE / Smart-Capture](#smart-ae--smart-capture)

---

## 60초 만에 알아보는 아키텍처

SDK는 Chloros 백엔드(데스크톱 GUI 및 CLI에서 사용하는 것과 동일한 Flask 서버) 위에 구축된 얇은 Python 계층입니다. 자동화를 위해서는 `chloros_sdk`를 임포트하고 고수준레벨 메서드를 호출하면 됩니다. 내부적으로는 모든 호출이 포트 5000의 로컬 백엔드(HTTP)에 대한  요청으로 변환됩니다 — `http://127.0.0.1:5000/api/...` (의도적으로 `localhost`가 아닌데, 이는 Windows에서 `::1`로 먼저 해결되며, IPv4 전용 백엔드에 대한 요청당 약 2초가 소요됩니다). 백엔드는 하드웨어 풀(카메라, DAQ 센서, 정렬 프로파일, 프레임 버퍼)을 관리하므로, SDK 스크립트는 시리얼 포트나 NIC 대역폭을 놓고 경쟁할 필요 없이 GUI와 공존할 수 있습니다.

사용하게 될 세 가지 인터페이스는 다음과 같습니다:

1. **`ChlorosLocal` + 무료 함수** (`process_folder`, `process_lattice_capture`) — 이미지 처리 파이프라인. Python 호출 한 번으로 전체 폴더에 대해 보정/디베이어/인덱스 내보내기 처리를 수행합니다.
2. **스마트 커넥트 핸들** (`connect_camera`, `connect_array`, `connect_daq_sensor`) — 라이브 하드웨어용 영구 백엔드 세션을 엽니다. GUI와 동일한 &quot;스마트 준비(smart-prep)&quot; 흐름: 네트워크 프로브, 계층 자동 선택, PTP, AE 시딩, GPIO 트리거 구성.
3. **`ChlorosProject` / `open_project`** — 저장된 프로젝트(`cameras.json` + `sensors.json` + `project.json`가 포함된 폴더), 모든 구성 요소를 한 번에 연결하고, 이름이 지정된 핸들을 사용하여 캡처를 실행합니다.

서피스 1과 2는 **아직 수신 대기 중인 백엔드가 없는 경우**로컬 백엔드를 자동 시작**합니다(GUI/CLI가 실행하는 것과 동일한 번들 바이너리). 따라서 백엔드를 먼저 시작하지 않아도 새 셸에서 스크립트만으로도 작동합니다. 이 기능을 사용하지 않으려면 (예: 절대 생성되지 않는 원격 백엔드를 지정할 때) `auto_start_backend=False`를 전달하십시오. [백엔드 자동 시작](#backend-auto-start)을 참조하십시오. Surface 3의 경우 동작 방식이 다릅니다: `open_project()`는 `auto_start_backend` 매개변수를 받지 않으며, `connect_all()`는 백엔드를 절대 생성하지 않습니다 — 대신 `http://127.0.0.1:5000`를 한 번 탐색하고, 응답이 없으면 아무런 메시지 없이 백엔드가 없는 직접적인 `lattice_sdk` 장치 제어 방식으로 돌아갑니다. 오직 `proj.process()`와 `stream(..., overlays=True)`만이 `ChlorosLocal()`를 지연 생성합니다(이 장치는 자동 시작됩니다).

이 세 가지 모두 인증-gated: 해당 시스템에서 `chloros-cli login`를 한 번 실행하거나, 데스크톱 GUI를 통해 로그인해야 합니다. 유효한 세션 없이 SDK를 호출하면 `ChlorosAuthenticationError` 오류가 발생합니다.

필수 조건:
- Python 3.7 이상 (패키지에 명시된 바와 같이; 3.10에서 개발/테스트3.10에서 테스트됨)
- 로컬에 Chloros Desktop이 설치되어 있어야 함(백엔드 바이너리는 설치 프로그램에 포함되어 있음)
- 유효한 Chloros+ 로그인. SDK / CLI에 대한 최소 등급은 **Copper**등급 이상이어야 함(Copper / Bronze / Silver / Gold); 무료**Iron**등급은 SDK / CLI에 접근할 수 없음. 이는**서버 측**에서 강제 적용됩니다: 모든 SDK / CLI 플래그가 지정된 요청은 활성 세션과 유료 요금제를 모두 포함해야 하며, 그렇지 않을 경우 백엔드는 `403`와 `error_code: PLAN_UPGRADE_REQUIRED`를 반환합니다 (`ChlorosLocal`에 의해 `ChlorosLicenseError`로, `connect_*` 헬퍼에 의해 `ChlorosConnectError`로 표시됨)로 반환됩니다. 로그아웃된 호출자는 `401` / `AUTH_REQUIRED` (`ChlorosAuthenticationError`)를 받게 됩니다. 이 두 오류는 서로 다른데, `chloros-cli login`를 다시 실행하면 첫 번째 오류는 해결되지만 두 번째 오류는 해결할 수 없기 때문입니다.
- 요금제의 유예 기간 내에서는 오프라인 사용이 지원됩니다: 해당 티어는 서버 검증 캐시(5분) 또는 서명된, 컴퓨터에 바인딩된 라이선스 캐시(월간 요금제의 경우 30일, 연간 요금제의 경우 구독 만료일까지)에서 읽혀집니다. 이 유예 기간이 만료되면 요금제는 무료 요금제로 전환되며, 기기가 서버에 한 번 연결될 때까지 SDK / CLI 접속이 중단됩니다. `chloros-cli status` (`GET /api/license-status`)는 무료 요금제에서도 계속 접속 가능하므로 그 이유를 확인할 수 있습니다. 이는 티어 제한에서 유일하게 제외된 SDK / CLI 경로만이 티어 게이트의 적용을 받지 않습니다.
- Windows 10/11 64비트, **Ubuntu 22.04 LTS 이상**또는 Jetson (JetPack 6). Ubuntu 20.04는**지원되지 않습니다**: `.deb`의 종속성은 `libc6 (>= 2.34)`를 포함하여 백엔드가 링크하는 대상에서 파생되며, focal 배포판은 glibc 2.31을 제공합니다.

---

## 설치

Python(SDK)는 Chloros 백엔드 위에 구축된 얇은 Python 레이어입니다. DAQ 전용 워크플로우 몇 가지를 넘어서는 모든 작업을 수행하려면 **Chloros 데스크톱 패키지를 로컬에 설치**해야 합니다 (Windows 설치 프로그램 또는 Linux `.deb`) — 이 패키지가 백엔드 바이너리, LATTICE 카메라용 Arena SDK 런타임 및 보정 번들을 제공합니다.

최신 다운로드: [`https://mapir.gitbook.io/chloros/download`](https://mapir.gitbook.io/chloros/download)

### 1단계 — Chloros 플랫폼 패키지 설치

#### Windows (.exe)

1. 다운로드 페이지에서 `Chloros-Setup-x.y.z.exe`를 다운로드합니다.
2. 설치 프로그램을 실행하고 마법사의 안내에 따라 진행하십시오. 기본 설치 경로는 `C:\Program Files\MAPIR\Chloros\`입니다.
3. Chloros을 한 번 이상 실행하고 Chloros+ 계정으로 로그인하십시오.

#### Linux amd64 (.deb)

```bash
sudo dpkg -i chloros-amd64.deb
sudo apt-get install -f         # only if dpkg reports missing dependencies
chloros-cli --version
chloros-cli login user@example.com 'YourPassword'
```

#### Linux arm64 — Jetson (JetPack 6)

```bash
sudo dpkg -i chloros-arm64-jp6.deb
sudo apt-get install -f
chloros-cli --version
chloros-cli login user@example.com 'YourPassword'
```

### 2단계 — Python 설치 SDK

**Chloros 설치 프로그램에는 일치하는 SDK 휠이 포함되어 있습니다.** 모든 Windows 설치 프로그램과 Linux .deb 파일은 GUI / CLI / 백엔드 버전과 정확히 일치하는 `chloros_sdk-X.Y.Z-py3-none-any.whl`를 디스크에 설치합니다. 최신 상태를 유지하기 위해 PyPI를 일일이 확인할 필요가 없습니다.

#### Windows

설치 프로그램은 시스템의 Python를 사용하여 번들된 wheel 파일에 대해 `pip install`를 자동으로 실행합니다(`py.exe` 런처를 우선적으로 사용하며, 실패 시 `python -m pip`로 대체됩니다). 별도의 조치가 필요하지 않습니다. 설치가 성공적으로 완료되면 `import chloros_sdk`가 Python 환경에서 작동합니다. 시스템에 Python가 설치되어 있지 않은 경우, 설치 프로그램은 이 단계를 자동으로 건너뛰며 GUI 및 CLI는 계속 작동합니다.

#### Linux (.deb)

.deb 패키지는 휠을 `/usr/lib/chloros/sdk/`에 배치합니다. `postinst`는 정확한 명령어를 출력합니다. PEP 668 배포판은 기본적으로 전역 pip 쓰기를 허용하지 않으므로, 자동설치를 수행하지 않습니다:

```bash
pip install --user /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

에어갭(air-gapped) Jetson 배포 환경에서는 이 과정이 완전히 오프라인으로 진행됩니다. 휠 파일이 이미 디스크에 존재하기 때문입니다.

#### 공개 PyPI

pip 전용 호스트(Chloros 데스크톱 패키지가 설치되지 않은 경우; 원격 백엔드 또는 DAQ 전용 워크플로우)의 경우:

```bash
pip install chloros-sdk
```

PyPI는 릴리스 버전 설치 프로그램 빌드 시 업데이트되므로, 게시된 휠은 최신 안정 버전에 일치합니다. 개발 빌드(예: `1.1.4.dev1`)는 번들된 설치 프로그램 휠을 통해서만 제공됩니다.

#### 확인

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)
print("DAQ_AVAILABLE    =", chloros_sdk.DAQ_AVAILABLE)
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)
```

> **Chloros+ 구독이 필요합니다.** 모든 SDK 호출에는 유효한 Chloros+ 로그인이 필요합니다. 각 컴퓨터에서 `chloros-cli login user@example.com 'YourPassword'`를 한 번 실행하십시오. 자격 증명은 `~/.chloros/`에 캐시됩니다.

### 데스크톱 패키지가 필요한가요?

대부분의 워크플로우에서는 pip 패키지만으로는 **불충분**합니다. 각 SDK 서피스에 필요한 사항은 다음과 같습니다:

| SDK 서피스 | 데스크톱 패키지 필요? | 이유 |
| --- | --- | --- |
| `ChlorosLocal`, `process_folder`, `process_lattice_capture` | **예** | `/usr/lib/chloros/chloros-backend` (Linux) 또는 `C:\Program Files\MAPIR\Chloros\…` (Windows)에서 백엔드 바이너리를 자동으로 시작합니다. |
| `connect_camera`, `connect_array`, `connect_daq_sensor`, `analyze_array_network`, `list_*`, `discover_*` | **예**(로컬)**/ 아니요**(원격) | 백엔드를 통한 순수 HTTP 클라이언트. 로컬 백엔드 → 데스크톱 패키지 필요. 원격 백엔드 →**터널을 통한** `backend_url=` (원격 백엔드 모드 참조 — 제공된 백엔드는 루프백에만 바인딩됨). |
| `ChlorosProject` / `open_project` | **예** | 백엔드를 통해 저장된 프로젝트를 구동합니다. |
| 직접 LATTICE 클래스 (`LatticeCamera`, `CameraPool`, `Calibration`, `DLS`, …) | **예** | 데스크톱 패키지 내에 포함된 Arena SDK 네이티브 런타임이 데스크톱 패키지 내에 포함된 Arena 네이티브 런타임이 필요합니다. 그렇지 않으면 `CAMERA_AVAILABLE`는 임포트 시 `False`로 처리됩니다. |
| 직접 DAQ 클래스(`DAQUSensor`, `DAQMSensor`, `DAQESensor`, `SensorFleet`, `discover_all`) | **아니요** | pyserial/bleak/zeroconf를 통한 순수 Python. pip 전용 환경에서도 DAQ를 종단 간으로 구동할 수 있습니다. |

### 원격 백엔드 모드 (터널을 통한 pip 전용 호스트)

> **제공된 백엔드는 LAN을 통해 접근할 수 없습니다.** 실제 운영 환경에서는
> 빌드는 루프백 전용(두 루프백 계열 모두)으로 바인딩되며,
> 유일한 비루프백 모드(`CHLOROS_CLOUD_MODE`)를 완전히 거부하므로,
> `backend_url="http://<lan-ip>:5000"`는 **설치된
> Chloros**에서는 작동할 수 없습니다 — 이 방식은 소스/dev
> 백엔드에서만 작동했습니다. 다른 머신의 백엔드를 구동하려면, 해당 머신의 루프백
> 포트를 직접 포워딩하고 SDK을 터널로 지정하십시오:

```bash
# on the pip-only host: forward local 5000 to the Chloros machine's loopback
ssh -N -L 5000:127.0.0.1:5000 user@chloros-host
```

```python
import chloros_sdk

BACKEND = "http://127.0.0.1:5000"   # the tunnel endpoint

chloros_sdk.connect_camera("213800234", backend_url=BACKEND)
chloros_sdk.connect_array(serials, backend_url=BACKEND)
chloros_sdk.connect_daq_sensor(eth_host="daq-e-1.local", backend_url=BACKEND)
```

헤드리스/CI/로봇 공학 호스트의 경우, 전체 데스크톱이 설치된 한 대의 머신을 &quot;Chloros 서버&quot;로 유지하고, 나머지 모든 곳에는 `pip install chloros-sdk`를 사용할 수 있습니다 — 단, 이들 간의 전송은 위에서 언급한 사용자가 구성한 터널을 통해 이루어지며, 직접적인 LAN URL 연결은 아닙니다.

> **알려진 제한 사항 — `ChlorosLocal`는 pip 전용 기능을 지원하지 않습니다.** `ChlorosLocal(backend_url=BACKEND)`는 현재 생성자에서 URL을 탐색하기 *전에* 로컬 백엔드 바이너리를 확인하며, 데스크톱 패키지가 설치되어 있지 않을 경우 — 원격 백엔드에 연결이 가능한 상태라 하더라도 — `ChlorosBackendError`(&quot;Chloros 백엔드를 찾을 수 없음…&quot;) 오류를 발생시킵니다. 위의 스마트 커넥트 영역(`connect_camera` / `connect_array` / `connect_daq_sensor`, 그리고 `analyze_array_network`와 `list_*` / `discover_*` 헬퍼)만 pip 전용 호스트에서 작동합니다.

### DAQ 전용 워크플로우 (pip 전용 호스트)

DAQ 센서만 필요하고 LATTICE 카메라나 이미지 처리를 사용하지 않는 경우, pip 패키지는 자체적으로 모든 기능을 갖추고 있습니다:

```bash
pip install chloros-sdk
```

```python
from chloros_sdk import DAQUSensor, DAQMSensor, DAQESensor, discover_all

for d in discover_all(timeout=3.0):
    print(d.model, d.display, d.address)   # USB serials: d.extra.get("serial_number")

sensor = DAQUSensor(port="/dev/ttyUSB0")
sensor.connect()
sensor.start_streaming()
```

백엔드도, .deb 패키지도, Chloros+ 로그인도 필요 없이 하드웨어 직접 DAQ 작업을 수행할 수 있습니다.

---

## 빠른 시작

```python
import chloros_sdk

# === Image processing ===
results = chloros_sdk.process_folder(
    "C:/DroneImages/Flight001",
    indices=["NDVI", "NDRE", "GNDVI"],
)

# === Live LATTICE single-cam ===
with chloros_sdk.connect_camera("213800234") as cam:
    cam.set_settings(exposure_time=10000, gain=0.0)
    cam.capture("output/")

# === Live LATTICE synchronized array (GUI smart-prep flow) ===
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    arr.capture("output/", processing="reflectance")

# === Live DAQ spectral sensor ===
with chloros_sdk.connect_daq_sensor() as daq:    # smart-detect USB / BLE / ETH
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])

# === Drive a saved project end-to-end ===
proj = chloros_sdk.open_project("/path/to/project")
proj.connect_all()
proj.arrays["main_rig"].capture("output/", processing="reflectance")
proj.disconnect_all()
```

---

## 최상위 API 인덱스

```python
import chloros_sdk

# === Image processing (full pipeline) ===
chloros_sdk.ChlorosLocal                          # class
chloros_sdk.process_folder(...)                   # one-shot helper
chloros_sdk.process_lattice_capture(...)          # LATTICE-friendly defaults
chloros_sdk.read_image_audit_tags(path)           # post-run audit

# === Live cameras (persistent backend pool) ===
chloros_sdk.connect_camera(serial, ...)           # → CameraSession
chloros_sdk.connect_array(serials, ...)           # → ArraySession (smart-prep)
chloros_sdk.attach_array(serials_or_id, ...)      # → ArraySession (attach without re-connecting)
chloros_sdk.list_cameras()
chloros_sdk.list_arrays()
chloros_sdk.discover_lattice_cameras()
chloros_sdk.analyze_array_network(...)            # network capability + recommendation
chloros_sdk.CaptureResult                         # list subclass returned by ArraySession.capture
chloros_sdk.RecorderHandle                        # handle for an array record()/burst() job

# === Live DAQ sensors (persistent backend pool) ===
chloros_sdk.connect_daq_sensor(...)               # → DAQSensorSession
chloros_sdk.discover_daq_sensors()                # scan USB/BLE/ETH (finds a DAQ-M MAC)
chloros_sdk.list_daq_sensors()

# === Project lifecycle ===
chloros_sdk.open_project(path)                    # → ChlorosProject
chloros_sdk.ChlorosProject                        # class
chloros_sdk.AlignmentSpec                         # dataclass
chloros_sdk.ArrayHandle, CameraHandle, SensorHandle

# === Direct-hardware (no-backend) classes (from lattice_sdk / daq_sdk) ===
chloros_sdk.LatticeCamera, CameraSettings, PRESETS, CameraPool
chloros_sdk.Calibration, CalibrationCoefficients, FilterModel, list_filters
chloros_sdk.DLS, NetworkDiagnostics
chloros_sdk.DAQUSensor, DAQMSensor, DAQESensor, SensorFleet, discover_all

# === Exceptions ===
chloros_sdk.ChlorosError                          # base
chloros_sdk.ChlorosBackendError
chloros_sdk.ChlorosLicenseError
chloros_sdk.ChlorosConnectionError
chloros_sdk.ChlorosProcessingError
chloros_sdk.ChlorosAuthenticationError
chloros_sdk.ChlorosConfigurationError
chloros_sdk.ChlorosConnectError                   # raised by smart-connect surface
chloros_sdk.LatticeError, CameraNotFoundError, ...  # from lattice_sdk

# === Availability flags ===
chloros_sdk.CAMERA_AVAILABLE     # True iff lattice_sdk imported cleanly
chloros_sdk.DAQ_AVAILABLE        # True iff daq_sdk imported cleanly
chloros_sdk.PROJECT_AVAILABLE    # True iff ChlorosProject deps available
```

---

## 이미지 처리 — `ChlorosLocal`

주요 파이프라인 클래스입니다. 첫 사용 시 백엔드를 시작하고, 프로젝트를 생성 및 구성하며, 진행 상황을 모니터링하고, 실행 후 요약 정보를 반환합니다.

### 생성자

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

### 메서드

| 메서드 | 설명 |
| --- | --- |
| `create_project(project_name, camera=None)` | 새 프로젝트 생성 (선택적으로 `"Survey3N_RGN"`와 같은 카메라 템플릿을 사용하여). |
| `import_images(folder_path, recursive=False)` | RAW/TIF/JPG/DNG 이미지 **및 `.daq` 광센서 기록**을 가져옵니다. `count` (이미지) 및 `scan_count`(기록 데이터)를 반환합니다. 폴더에 둘 다 포함되어 있지 않은 경우에만 경고를 표시합니다. |
| `export_light_sensor(daq=True, csv=True)` | 프로젝트의 모든 광센서 기록 데이터에 대해 보정된 `.daq` + `.csv` 파일을 `<project>/Light Sensor/`에 기록합니다. [광센서 기록](#light-sensor-recordings--calibrated-daq--csv)을 참조하십시오. |
| `configure(debayer=..., vignette_correction=..., reflectance_calibration=..., indices=[...], export_format=..., ppk=..., daq_log_path=..., input_level=..., radiometric_output=..., array_alignment=..., array_alignment_crop=..., array_alignment_interpolation=..., custom_settings=None)` | 처리 노브를 설정합니다. |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | 파이프라인을 실행합니다. `{"status": "complete", "async": False}`를 반환하며, 백엔드에서 키를 제공하는 경우 `summary` 키도 함께 반환합니다. [실행 후 요약 및 힌트](#post-run-summary--hints) 참조). |
| `get_config()` / `get_status()` / `status()` | 백엔드 상태 확인. |
| `logout()` | 캐시된 자격 증명을 지웁니다. |
| `shutdown_backend()` | 백엔드를 종료합니다(SDK로 시작된 경우). |
| `discover_cameras()` | **이 인스턴스의 백엔드**(`/api/camera/discover`)를 통해 LATTICE 카메라를 검색합니다. | 이 인스턴스의 백엔드**를 통해 LATTICE 카메라 검색 (`/api/camera/discover`). 딕셔너리 목록을 반환합니다(`serial`, `model`, `ip`, …) — GUI/CLI에서 확인하는 것과 동일한 형태입니다. 발견되지 않거나 백엔드에 연결할 수 없는 경우 빈 리스트를 반환합니다. |
| `camera_capture(output_dir, format="tiff", **settings)` |**백엔드를 통해**(이 핸들에 의해 자동으로 시작됨) 단일 프레임을 캡처하여, GUI/CLI와 동일한 사전 설정(12비트 기본값, 풀 재사용, 내장된 캘리브레이션 메타데이터). `serial=` 또는 `device_index=`를 사용하여 대상을 해결하고, `exposure`/`gain`/`pixel_format`/`preset`는 `**settings`로 변환하여 전달하십시오. 레거시 메타데이터 딕셔너리를 반환합니다 (`filepath`, `width`, `height`, `pixel_format`, `exposure_time`, `gain`, `timestamp`). |
| `camera_stream(serial, *, fps=10.0, overlay=None, decode=True, connect_timeout=10.0, read_timeout=15.0)` | 풀링된 카메라에서 오버레이 합성 미리보기 프레임을 생성합니다 — 백엔드의 `/api/camera/<serial>/stream-annotated` 경로를 통해 가벼운 MJPEG 클라이언트 (서버 측에서 그려진 제브라 / 그리드 / 십자선 / 히스토그램 / 피킹 / 스팟). `decode=True`는 BGR 배열을 반환하고, `False`는 원시 JPEG 바이트를 반환합니다. 또한 프로젝트별로 `ChlorosProject.stream(overlays=True)`로도 접근 가능합니다. |

확실한 정리 처리를 위해 컨텍스트 매니저로 사용하십시오:

```python
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

### 광센서 기록 — 보정된 `.daq` + `.csv`

DAQ-U / DAQ-M / DAQ-E는 보정 번들 **없이** 기록할 수 있습니다. 즉,
공개된 [`chloros_scripts`](https://github.com/mapircamera/chloros_scripts)
레코더(`record_daq.py`)는 기본적으로 원시 센서 카운트 데이터를 기록하고,
파일에 타임스탬프를 찍어, Chloros에서 해당 센서의 공장 교정 데이터를 **시리얼 번호를 통해** — 먼저 로컬 캐시에서,
그 다음 MAPIR 클라우드에서 — 가져와 가져오기 시 적용합니다.

Chloros은 기록당 두 개의 제품으로 결과를 다시 출력하며,
`<project>/Light Sensor/` 아래에 다음과 같이 저장됩니다:

| 제품 | 설명 |
| --- | --- |
| `<name>_calibrated.daq` | 재처리 가능한 아카이브 — 실시간 기록과 동일한 스키마이며, 이제 이를 생성한 번들을 명시합니다. 이를 다시 가져와도 **두 번째** 보정은 수행되지 않습니다. |
| `<name>_calibrated.csv` | 센서 자체 파장 그리드 기준의 스펙트럼 조도값(W/m²/nm) — 센서 자체의 파장 그리드 기준, 측정값당 한 행씩, 여기에 광도 측정 열(총 전력, 광시/암시 럭스, PPFD 및 그 청색/녹색/적색 분할값, 피크 파장). |
| `<name>_raw.daq` / `<name>_raw.csv` | **번들 미포함 센서 전용 (DAQ-A).** 원시 분광 센서 카운트 — *조도*가 아님. 아래 참조. |

`process()`는 이 내보내기 작업을 단계 중 하나로 수행합니다. 이 작업에는 **이미지**가 필요하지 않습니다:
단독으로 운용되는 광 센서는 독립적인 워크플로우이며, 이러한 프로젝트는 구조상
이미지가 전혀 없습니다.

**DAQ-A 기록은 원시 카운트 형태로 내보내집니다.** DAQ-A 제품군은 시리얼별
번들 시스템이 도입되기 이전에 출시되었으며, 가져올 번들이 없습니다 — 대신 현장에서
반사율 타깃을 기준으로 보정되므로 번들이 필요하지 않았습니다. 이러한 기록 데이터는
`_calibrated`가 아닌 `_raw` 접두사로 내보내집니다. 파일 내부의 플래그가 아닌
다른 파일명을 사용하는 이유는, 이 정보가 파일 이름만 있는 상태로 이메일을 통해 전송될 때도
유지되어야 하기 때문입니다.
`.csv` 헤더는 `raw spectral sensor counts (NOT irradiance)`를 표시하며, 해당
값들은 파일 **내부**에서만 비교 가능하다는 경고를 표시합니다 — 이는 바로 타겟 기반 보정이
이 값들을 사용하는 목적이며, 센서 간 비교를 위한 것이 아닙니다. 전력 의존적 광도 측정 열(총 전력,
광시/암시 럭스, PPFD)는 카운트 값을 통합한 값이 아닌 **NULL**로 반환됩니다.

번들을 전혀 가져올 수 없었던 DAQ-U / DAQ-M / DAQ-E는 여전히 **건너뜁니다**,
원시 데이터로 기록되지는 않습니다. 이 경우 번들이 존재하므로 “재연결 후 재처리”가 실질적인 해결책입니다.

구버전 **v1.01 / v1.02** 기록 파일(DAQ-A-SD가 작성함)에는 판독별 에포크 정보가 없으며,
오직 파일의 작성 시간만 포함됩니다. 이미지↔다운웰링 매처는 여전히 이를 거부합니다 —
프레임을 쓰기 시간과 매칭하는 것은 눈에 띄지 않게 오류가 발생할 수 있습니다) — 하지만 익스포터는 이를 읽을 수 있으며,
CSV는 `clock=daq_created_on`를 출력하므로 제품이 어떤 클럭을 사용 중인지 표시합니다.

```python
import chloros_sdk

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("DAQ-U_2026-08-26")
    cl.import_images("C:/Flights/raw_daq")     # .daq only — no camera involved
    result = cl.export_light_sensor()          # or just cl.process()

for rec in result["exported"]:
    print(rec["csv"])
for rec in result["skipped"]:
    print("skipped", rec["source"], "--", rec["reason"])
```

보정 번들을 가져올 수 없는 기록(오프라인이거나 파일에 보정 정보가 없는
센서인 경우)는 `skipped`로 **사유와 함께** 보고됩니다. 이 기록은 원시 카운트 값을 포함하는 &quot;보정된&quot; 파일로
절대 출력되지 않습니다. 인터넷에 연결한 후
다시 실행하면 내보내기가 완료됩니다.

### 진행 상황 콜백

```python
def show_progress(percent, message):
    print(f"[{percent:3d}%] {message}")

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(indices=["NDVI"])
    cl.process(progress_callback=show_progress, poll_interval=1.0)
```

### 실행 후 요약 및 팁

완료 시, `process()`는 `GET /api/processing-summary`를 가져와 본문을 `result["summary"]`로 첨부합니다. 이 가져오기 작업은 최선을 다해 수행되며, 성공적인 반환을 절대 차단하지 않습니다. 요약 정보를 사용할 수 없는 경우, `process()`는 기본 `{"status": "complete", "async": False}` 형식으로 대체됩니다. `summary["hints"]`의 각 항목 — 제안된 수정 사항이 포함된 완전한 문장(예: 실행 결과가 0인 이유) —는 Python `UserWarning` 형태로도 재전송되므로, 딕셔너리를 확인하지 않더라도 출력이 0인 실행은 자체적으로 진단됩니다:

```python
result = cl.process()
for hint in result.get("summary", {}).get("hints", []):
    print("HINT:", hint)
# hints also arrive on the warnings channel:
#   python -W always::UserWarning your_script.py
```

`summary["totals"]`는 기계가 읽을 수 있는 형식입니다:

| 키 | 집계 대상 |
| --- | --- |
| `models` | 실행 내 카메라 그룹 수. |
| `images_in_groups` | 해당 그룹에 포함된 원본 이미지. |
| `targets_found` | 감지된 반사율 타겟. |
| `images_calibrated` | 실행 과정에서 보정된 이미지. |
| `exported_files` | **해당 실행에서 생성된 이미지 산출물 파일.** |
| `daq_recordings_exported` / `daq_recordings_skipped` | 광센서 기록 데이터(의도적으로 별도로 집계됨) — 이는 다른 단계에서 생성된 것이며 이미지가 전혀 없는 실행에서도 존재하므로, 이를 포함시키면 DAQ 전용 실행이 이미지를 내보낸 것처럼 보일 수 있습니다. |

이와 함께: `summary["output_dirs"]` (기록된 모든 디렉터리),
`summary["light_sensor_export"]`, `summary["stopped"]` (사용자가 실행을 중단했을 때
해당되며, 따라서 부분 카운트가 생산량이 부족한 완료된 실행으로 인식되지 않도록), 그리고
`summary["groups"]` (그룹별 내역).

`exported_files`는 파이프라인이 **쓰기 작업 중**에 기록하는 것이며, 나중에
프로젝트의 이미지 객체에서 스캔하여 추출하는 것이 아닙니다. 병렬 및 GPU 전략은 자체 이미지
객체를 생성하므로(GPU 경로의 경우 워커 하위 프로세스에서), 기존 스캔은
`0 file(s) written`를 보고한 뒤, 모든 것이 정상적으로 작동했던 실행에 대해 —를 출력했습니다. 이 번호를 기준으로 스크립트를 작성할 경우, 정상적인 병렬 실행에서는 이제
0이 아닌 카운트가 보고됩니다.

Light-sensor 건너뛰기 보고는 리더가 각 파일에 대해 실제로 확인한 이유(읽을 수 없는 스키마,
번들 누락, 쓰기 오류)를 **중복 제거된** 형태로 보고하므로, 한 가지 원인으로 인해 건너뛴 20개의 파일이
단일 원인으로 처리되어, 20개의 파일이 동일한 원인으로 건너뛴 경우 20번 반복되어 표시되는 대신 하나의 원인으로만 표시됩니다.

> **`process()`는 실행에서 이미지가 생성되지 않을 때 발생하지 않습니다.** 이는 SDK와
> CLI가 의도적으로 차이를 두는 유일한 부분입니다: `chloros-cli process`는 &quot;결과물이 요청되었으나, 아무것도
> 기록되지 않음&quot; 를 실패로 간주하고 0이 아닌 값으로 종료하는 반면, SDK는 정상적으로 종료하고
> 해당 상황을 `summary` / hints를 통해 보고합니다. 파이프라인이 빈 실행 시 중지되어야 한다면,
> 직접 확인하십시오 — 예외가 발생하지 않았다는 사실에만 의존하지 말고, `summary`를 확인하거나(또는 프로젝트 폴더 내 파일 수를 세어보십시오)
> 예외가 발생하지 않았다는 사실에만 의존하지 마십시오. 일반적인 원인은 입력 폴더가
> 캡처로 인식되지 않았거나, 현재 설치된 카메라에 적용할 수 없어 제품이 건너뛴 경우입니다(예: RGB 전용
> 카메라의 경우).

### 편의 함수

```python
# One-call process: project + import + configure + process
results = chloros_sdk.process_folder(
    folder_path="C:/DroneImages/Flight001",
    project_name="FieldA_2026-05-26",
    camera="Survey3N_RGN",
    indices=["NDVI", "NDRE", "GNDVI"],
    vignette_correction=True,
    reflectance_calibration=True,
    export_format="TIFF (16-bit)",
    mode="parallel",
    debayer="High Quality (Faster)",      # or "Texture Aware (Slow, Highest Quality)"
    ppk=False,
    recursive=False,
    processing_timeout=14400,
)

# LATTICE-friendly defaults (no panel-target detection, standard debayer)
results = chloros_sdk.process_lattice_capture(
    folder_path="C:/Captures/2026-05-13_Field",
    indices=["NDVI"],
)

# Audit which calibration sources were applied to a processed image
tags = chloros_sdk.read_image_audit_tags("output/Reflectance_Calibrated/x.tif")
print(tags["CalibrationSource"])   # 'per_serial' / 'legacy_lookup' / 'none'
print(tags["VignetteSource"])      # 'per_serial' / 'legacy_polynomial' / 'none'
```

### 지원되는 값

```python
# export_format
"TIFF (16-bit)"           # default, recommended
"TIFF (32-bit, Percent)"  # reflectance percentage as float32
"PNG (8-bit)"
"JPG (8-bit)"

# debayer
"High Quality (Faster)"               # standard, default
"Texture Aware (Slow, Highest Quality)"  # neural debayer, Chloros+ only
"Standard (Fast, Medium Quality)"      # alias used internally for LATTICE

# input_level (LATTICE only; Survey3 .raw ignores)
"auto"        # default — infers from each file's XMP ProcessingLevel tag
"raw"         # force-treat as raw Bayer
"debayered"   # force-treat as already-debayered BGR
"processed"   # force-treat as already-calibrated radiance

# array_alignment / array_alignment_crop (LATTICE arrays; None = keep saved setting)
True          # backend default — apply the module-to-module transform stamped
              # in each capture's Chloros:Alignment* XMP to every product
False         # export in native sensor geometry / skip the common-overlap crop

# array_alignment_interpolation (alignment warp resampling)
"bilinear"    # backend default
"nearest"     # preserves exact source DNs (no inter-pixel value mixing)
"cubic"
```

#### 방사계량 출력 (LATTICE 다중 스펙트럼 파이프라인)

`process` 파이프라인의 LATTICE 다중 스펙트럼(M3C/M3M) 내보내기 수준 — `reflectance`(기본값), `radiance`, `sensor-response` 또는 `all`(이미지별 적용 가능한 모든 모드) —는 프로젝트의 **&quot;방사계 출력&quot;** 처리 설정에 매핑됩니다. `configure()`에는 전용 키워드가 있습니다:

```python
with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("Field_A")
    cl.import_images("C:/Captures/lattice_flight")
    cl.configure(
        radiometric_output="radiance",   # reflectance (default) / radiance / sensor-response / all
        export_format="TIFF (32-bit, Percent)",
    )
    cl.process()
```

고급 우회 방법 — `"Radiometric output"` 키를 `custom_settings`를 통해 작성하는 것 —은 여전히 작동하지만, 이 방법은 전체 설정 블록을 대체한다는 점을 기억하십시오(아래 경고 참조):

```python
cl.configure(custom_settings={
    "Project Settings": {
        "Processing": {"Radiometric output": "radiance"},
        "Export": {"Calibrated image format": "TIFF (32-bit, Percent)"},
    }
})
```

`reflectance`(기본값)는 카메라 복사도를 **타임스탬프가 일치하는 DAQ 하향 복사량**로 나누어 계산하며, 이는 기록된 `.daq` (DAQ-U/M/E)**또는 이미지와 함께 발견된 DAQ-M 네이티브 `.csv`**에서 자동으로 해결됩니다. 로컬에 없는 카메라별 또는 DAQ 보정 번들은**첫 사용 시 AWS에서 **자동으로 가져옵니다**. CLI는 이를 `chloros-cli process`: `--radiance`/`--no-radiance`, `--reflectance`/`--no-reflectance`, `--debayered`, `--preview`에서 제품 유형별 토글 버튼으로 표시됩니다.

> `custom_settings`는 계산된 설정 블록 전체를 **대체**합니다 (설계상 `configure()`의 다른 키워드와 유효성 검사를 우회합니다). 이를 사용할 때는 위 예시와 같이 필요한 모든 `Project Settings` 키를 포함시켜야 합니다.

---

## LATTICE 카메라용 Smart-Connect

라이브 하드웨어를 위한 지속적 백엔드 세션입니다. GUI에서 사용하는 엔드포인트와 동일하므로 SDK / CLI / GUI 전반에 걸쳐 동작이 동일합니다.

### 단일 카메라 — `CameraSession`

```python
import chloros_sdk

# Open by serial; reuses existing pool entry if one exists
with chloros_sdk.connect_camera("213800234") as cam:
    # cam is a CameraSession; supports context manager + manual disconnect
    cam.set_settings(
        exposure_time=10000,    # microseconds
        gain=0.0,               # dB
        pixel_format="BayerRG12",
        target_brightness=80,
        ae_damping=8.0,
    )
    cam.capture("output/", ext=".tiff")
```

#### `connect_camera()` 시그니처

```python
connect_camera(
    serial,
    *,
    preset=None,                       # "default" | "high_quality" | "high_speed" | "triggered"
    settings=None,                     # dict overlaid on the preset
    backend_url="http://127.0.0.1:5000",  # deliberately not 'localhost' (::1-first on Windows ≈ 2 s/request)
    timeout=60.0,
    auto_start_backend=True,           # spawn a local backend if none is running
) -> CameraSession
```

#### `CameraSession` 메서드

| 메서드 | 설명 |
| --- | --- |
| `read_nodes(names, enum_names=(), timeout=30.0)` | GenICam 노드를 읽습니다. `{nodes, errors, enums, device}`를 반환합니다. |
| `set_settings(**kwargs)` | 친숙한 이름으로 노드 쓰기 (`exposure_time`, `gain`, `pixel_format`, `width`, `height`, `target_brightness`, `ae_damping`, `ae_upper_limit`, `trigger_mode`, `trigger_source`, …). |
| `capture(output_dir="output", ext=".tiff", jpeg_quality=95, processing=None, levels=None, force_daq=None, settings=None, timeout=None)` | **단일** 프레임을 캡처합니다. 프레임 메타데이터 딕셔너리로 구성된 1개 요소의 리스트를 반환합니다. (연속/다중 프레임 캡처 기능은 제거되었습니다. 일련의 프레임이 필요한 경우 루프 내에서 `capture()`를 호출하십시오.) |
| `disconnect()` | 풀에서 해제합니다. 이미 열린 세션에 연결된 경우에는 아무 작업도 수행하지 않습니다. |

`capture()` 내보내기 제어 (배열 + GUI와 동일한 모델):

- `processing` / `levels` — `processing="all"`는 적용 가능한 모든 내보내기 유형을 저장하며, `levels=["raw","radiance"]`는 해당 항목만 저장합니다(`processing`를 재정의함). 백엔드 기본값을 사용하려면 두 항목 모두 생략하십시오.
- `force_daq=True` — DAQ/DLS 측정값이 할당된 경우, 원시 데이터만 캡처하는 경우에도 해당 값을 `.daq` 사이드카로 저장하여, 나중에 프레임을 반사율/굴절률 데이터로 재처리할 수 있도록 합니다. DAQ가 연결되지 않은 경우 아무 작업도 수행하지 않습니다.

### 동기화된 어레이 — `ArraySession` (Smart-Prep)

`connect_array`는 다중 카메라 설정 시 **권장되는 진입점**입니다. 이 옵션은 내부적으로 전체 GUI 스마트 준비(Smart-Prep) 흐름을 실행합니다:

1. **네트워크 분석** (`/api/camera/array/recommend`) — 프레임 손실 없이 sim-emit 티어에 맞는 최대 프레임 크기를 찾습니다.
2. **티어 자동 선택** — 회선 용량이 허용할 경우 `sim-capture-sim-emit`를 선택하며, 그렇지 않은 경우 `sim-capture-ftd-stagger` 또는 `slip-emit-and-capture`를 선택합니다.
3. **자동 축소**— 와이어가 요청된 해상도를 유지할 수 없을 때 프레임 크기를 축소하거나 비닝을 증가시킵니다.**이 안전 장치는 집계 과부하를 처리하지 않습니다**: 와이어에 연결된 카메라 수가 너무 많은 경우 프레임 축소를 통해 해결할 수 없습니다 — [과다 할당](#over-subscription-the-per-cam-floor)을 참조하십시오.
4. 기본적으로 **PTP 활성화**— 카메라 간 타임스탬프가 하나의 공유 클럭에**~1 ms**오차 범위 내에서 동기화됩니다. 동시 노출은 PTP가 아닌 M8 하드웨어 트리거(모듈 간**&lt; 100 µs**)에 의해 이루어집니다: PTP는 *타임스탬프*를 동기화할 뿐, 노출을 동기화하지 않습니다.
5. **카메라별 픽셀 형식 자동 선택** — RGB 카메라 → `BayerRG8`, 다중 스펙트럼 카메라 → `BayerRG12`.
6. **AE 시딩** — 각 카메라의 현재 AE 상태를 스냅샷으로 저장하여, 연결 시 촬영 도중 노출 설정이 재설정되지 않도록 합니다.
7. **GPIO 트리거 구성** — `connect_array`는 모든 카메라(`TriggerMode=On`, `TriggerSource=Line2`)를 활성화하여 마스터의의 펄스가 M8 케이블을 통해 슬레이브를 구동합니다. 이는 어레이 전용 단계입니다. `LatticeCamera`로 단일 카메라를 열면 대신 자유 실행 모드로 전환됩니다.

```python
import chloros_sdk

# First serial is the MASTER (fires the trigger pulse); rest are slaves.
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    print(arr.array_id, arr.sync_mode, arr.ptp_enabled)
    arr.capture("output/", processing="reflectance")
```

#### `connect_array()` 시그니처

```python
connect_array(
    serials,                              # list[str]; serials[0] = master
    *,
    line="Line2",                         # GPIO sync line: Line0 | Line2 | Line3
    target_fps=None,                      # master trigger fire rate (auto if None)
    force_tier=None,                      # override tier picker; see below
    wire_ceiling_mbps=None,               # host sustained wire budget, MB/s (auto if None)
    width=None,                           # explicit frame size; skips network analysis
    height=None,
    pixel_format=None,
    binning=None,
    recommend=True,                       # set False to skip the recommend step
    ptp_enable=True,                      # set False to disable PTP
    backend_url="http://127.0.0.1:5000",  # same IPv6-avoidance default as connect_camera
    timeout=180.0,
    auto_start_backend=True,              # spawn a local backend if none is running
) -> ArraySession
```

`force_tier` 값:
- `"sim-capture-sim-emit"` — 진정한 동시성 (모든 캠이 동일한 클럭 에지에서 발동).
- `"sim-capture-ftd-stagger"` — 유연한 시간 영역 스태거 (캠이 약간 오프셋된 시간에 신호를 방출하여 패킷이 회선상에서 직렬화됨).
- `"slip-emit-and-capture"` — 캠별 순차 캡처 (시간적 동기화 없음; 프레임 크기가 동시 전송에 맞지 않을 때만 선택 가능한 옵션).

`wire_ceiling_mbps`는 **호스트의 지속적 전송 대역폭** (MB/s 단위)를 재정의합니다. 이는 전체 어레이 할당이 의존하는 유일한
수치입니다. 자동 감지된
값을 사용하려면 `None`로 설정해 두십시오. 어레이가 GVSP 손상 프레임을 보고할 경우 이 값을 낮추십시오. 자동 값은
NIC가 알리는 링크 속도에서 도출되는데, 이는 USB 어댑터, 대역폭이 낮은 PCIe 레인 및
부하가 높은 공유 패브릭의 실제 속도를 과대평가하는 경향이 있습니다. 이러한 과대평가는 눈에 띄게 느린 링크로 나타나기보다는
눈에 띄게 느린 링크로 나타나지 않고 손상된 프레임 형태로 나타납니다. 이 값은 프로젝트의 어레이 캡처 블록에 저장되므로,
다시 열거나 나중에 `connect_array`를 실행하면 다른 어레이 설정과 마찬가지로 복원됩니다.
[어레이 상태](#array-health--which-subsystem-is-losing-frames)를 참조하십시오.

#### 오버서브스크립션(카메라당 하한값)

Sim-emit 페이싱은 각 카메라에 충돌 방지 배선 예산의 일부를 할당하며, 하한값은 **카메라당 8 MB/s**(`per_cam_floor_bps`)로 설정됩니다. `N × floor`가 충돌 방지 상한선을 초과하면, 어레이는**와이어에 과부하를 일으킵니다**— 이로 인한 오류 모드는 프레임 속도 저하가 아닌 GVSP 패킷 손실입니다 — 그리고 프레임 크기를 조정하여 해결할 방법은 없습니다:**프레임당 바이트 수를 줄이는 것이지, 집계 검사에서 비교하는 초당 할당된 바이트 수를 줄이는 것이 아닙니다**. 1 GbE 호스트에서 실제로 적용 가능한 풀 해상도 상한선:**1500 MTU 기준 카메라 6대, 점보 프레임 사용 시 9대** (분석 응답의 `max_cams_collision_safe`가 해당 회선의 상한선을 나타냅니다). 해결책: 카메라 수 줄이기, 종단 간 점보 프레임 사용, 또는 더 빠른 NIC 사용.

- `analyze_array_network()` 및 `/api/camera/array/connect` 응답에는 `oversubscribed`, `aggregate_demand_bps`, `collision_safe_ceiling_bps`, `max_cams_collision_safe` 및 `per_cam_floor_bps`를 포함합니다. `oversubscribed`가 참일 경우, 이 투영은 **fps 필드를 0으로 설정**합니다 (`achievable_fps_max` / `fps_bright` / `fps_dark`) 오해의 소지가 있는 ‘느리지만 작동하는’ 속도를 보고하는 대신.
- `POST /api/camera/array/connect`는 `pin_resolution` 본문 매개변수를 허용합니다(**HTTP 전용 — SDK kwarg가 아님**; `connect_array`는 이를 노출하지 않음). 고정(pinning)을 적용하면 비닝(binning) 워크다운 안전 장치가 제거되므로, `pin_resolution`가 설정된 상태에서 과부하 상태인 연결 요청은 모든 해결책을 명시한 오류 메시지와 함께**단호히 거부**됩니다. 고정(pinning)을 적용하지 않은 경우, 연결은 단계적 축소 과정을 진행하지만, 축소만으로는 집계 값을 초기화할 수 없다는 경고가 표시됩니다.
- 벤치마크용 비상 조치: 백엔드 환경에서 `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1`를 설정하여 거부 처리를 강력한 경고로 하향 조정할 수 있습니다. 이 경우 연결은 진행되지만 패킷 손실을 감수해야 합니다.

#### 어레이 상태 — 어떤 하위 시스템에서 프레임이 손실되고 있는지

`GET /api/camera/array/<array_id>/capability`는 연결된 어레이에서 활성 상태인 `health` 블록을
반영하며, **10초** 단위의 롤링 창을 기준으로 재평가됩니다. 이 지표는 프레임 손실을
단일한 &quot;불완전&quot; 비율로
묶어 두 가지 원인을 구분합니다:

| 필드 | 의미 | 해당 하위 시스템 |
| --- | --- | --- |
| `gvsp_corrupt_rate_pct` (시리얼별) | 프레임이 **도착했으나 구조적으로 손상됨**— GVSP 패킷 손실. |**네트워크**: 와이어 예산, 페이싱, NIC RX 링, MTU |
| `never_arrived_rate_pct` (시리얼별) | 프레임이 **아예 도착하지 않음**— 카메라가 작동하지 않았거나, 카메라에서 아무것도 출력되지 않았음. |**트리거 / 동기화**: M8 케이블, `line=`, `TriggerMode` |
| `worst_gvsp_corrupt_pct` / `worst_never_arrived_pct` | 각 카메라별 최악의 촬영률. | — |
| `per_cam_rate_pct` | 카메라별 통합 미완료율 (두 원인 합산). | — |
| `stable_for_seconds` | 각 카메라가 0.01 % 미만으로 유지된 기간. | — |

`health`와 함께, 동일한 기록에는 전체 할당량이 얼마나 오래 유지되었는지에 대한 수치도 보고됩니다:

| 필드 | 의미 의미 |
| --- | --- |
| `wire_ceiling_mbps` | 호스트의 현재 유효한 지속적 유선 대역폭 할당량, MB/s. |
| `wire_ceiling_source` | 해당 수치의 출처(문구) — 예: `USB-capped 200 MB/s (was theoretical 1062; …)` 또는 `user override 120 MB/s (auto said 200)`. |
| `wire_ceiling_is_user_set` | `true`가 `wire_ceiling_mbps=`를 설정했을 때. |
| `nic_is_usb` | USB 이더넷 어댑터의 경우 `true`. |

이 엔드포인트에 대한 SDK 래퍼는 없으므로 직접 읽어야 합니다:

```python
import requests, chloros_sdk

arr = chloros_sdk.attach_array(["213800234", "214000533"])
h = requests.get(
    f"http://127.0.0.1:5000/api/camera/array/{arr.array_id}/capability",
    timeout=10).json()

health = h.get("health", {})
print("wire ceiling:", h["wire_ceiling_mbps"], "MB/s", h["wire_ceiling_source"])
print("corrupt (network) :", health.get("worst_gvsp_corrupt_pct"), "%")
print("absent  (trigger) :", health.get("worst_never_arrived_pct"), "%")

if (health.get("worst_gvsp_corrupt_pct") or 0) > 1.0:
    # Network path. Reconnect with a lower budget -- NOT a lower target_fps.
    arr.disconnect()
    arr = chloros_sdk.connect_array(serials, wire_ceiling_mbps=120)
```

**읽기 방법:** 0이 아닌 `gvsp_corrupt_rate_pct` 값과 0인 `never_arrived_rate_pct` 값은
트리거링 및 케이블 동기화가 완벽하며, 손실의 100%가 네트워크 경로에서 발생함을 의미합니다 — 값을 낮추고
`wire_ceiling_mbps` 값을 낮추고 다시 연결하십시오. 반대의 패턴은 동기화 케이블이나
트리거 라인에 문제가 있음을 나타냅니다.

> **`target_fps`는 프레임 손상의 결정적 요인이 아닙니다.** GevSCPD 페이싱은
> 연결 시 한 번만 설정되므로, 트리거 속도를 낮추면 듀티 사이클만 변경될 뿐
> 동시 송신 버스트 속도는 변경되지 않습니다. 측정 결과, 수요를 5배 줄여도 개선 효과가 없었으나
> 와이어 상한선을 240 MB/s에서 200 MB/s로 낮추자 동일한 장비에서 손상률이 10.4 %에서
> 0.00 %로 떨어졌습니다.

> **TRI032S 펌웨어에서는 스트림 도중 자동 축소 기능을 사용할 수 없습니다.** 실행 중인 어레이는
> 이 문제를 자체적으로 해결할 수 없습니다. 연결을 해제한 후 다시 연결하여 연결 시점 선택기가
> 새로운 상한값을 기준으로 재계획하도록 하십시오.

**USB 이더넷 어댑터는 명판에 표시된 사양과 관계없이** 프로브에 의해
200 MB/s로 제한됩니다. 링크 속도를 지속 속도로 변환하는 효율성 표는
PCIe에서 파생된 것이며, USB NIC는 이더넷 링크 속도를 광고하지만
USB 버스 및 해당 드라이버의 제약 하에 있습니다. 이 상한은 비율(분수)이 아닌 절대값이며, USB 1 GbE 어댑터는
~80 MB/s를 도출하므로 영향을 받지 않습니다.

#### `ArraySession` 메서드

| 메서드 | 설명 |
| --- | --- |
| `status(timeout=10.0)` | 라이브 `{fps, ptp, frame_count, last_error, …}`. |
| `capture(output_dir="output", format="tiff", processing="debayered", levels=None, aligned=None, render_index=None, force_daq=None, smart=False, timeout=300.0)` | 동기화된 캡처 그룹 하나. `CaptureResult`(프레임 딕셔너리 목록 + `.skipped`)를 반환합니다. 아래의 내보내기 제어 항목을 참조하십시오. |
| `capture(..., smart=True)` | **스마트 캡처** — 모든 카메라에서 AE가 안정화될 때까지 대기한 후 트리거합니다. |
| `capture_fastest(output_dir="output", force_daq=True, render_index=True, timeout=120.0)` | 가장 빠른 캡처: 원시 데이터 전용 + 할당된 DAQ 판독값 (+ 자유 결합 인덱스). GUI의 “가장 빠른 캡처” 버튼을 반영합니다. |
| `capture_repeated(output_dir="output", count=None, duration_s=None, interval_s=0.0, on_capture=None, **capture_kwargs)` | 하나의 제한된 루프 내에서 단일 / 연속 / 간격 캡처. `list[CaptureResult]`를 반환합니다.**종료하려면 `count` 및/또는 `duration_s`가 필요합니다**(SDK에는 Ctrl+C 기능이 없음). |
| `record(output_dir="output", fps=10.0, duration_s=None, video=True, gif=False, timeout=30.0)` | 실시간 통합 인덱스 뷰를 비디오/GIF로 녹화 시작 → `RecorderHandle`. 배열당 하나의 복합 레코더. |
| `burst(output_dir="output", duration_s=None, max_frames=None, index_config=None, serial_index_config=None, timeout=30.0)` | 고화질ps 원시-Bayer 버스트 촬영 시작 → `RecorderHandle`. `build_video()`를 사용하여 오프라인에서 재처리합니다. |
| `build_video(burst_dir, products=None, fps=10.0, video=True, gif=False, save_tiffs=False, wait=True, poll_s=2.0, timeout=1800.0)` | 저장된 원시 버스트를 보정된 비디오로 오프라인 재처리합니다. Bl작업이 완료될 때까지(`wait=True`) 대기한 후 `{outputs, errors, combined}`를 반환합니다. |
| `build_video_status(job_id, timeout=15.0)` | 오프라인 빌드 작업(`{running, result, error, burst_dir}`)을 폴링합니다. |
| `disconnect()` | 전체 어레이를 해제합니다. |

`capture()` 내보내기 제어(GUI/CLI에서 사용하는 것과 동일한 엔드포인트):

- `processing` / `levels` — `processing="all"`(또는 `levels=["raw","radiance",…]`)는 카메라별로 적용 가능한 모든 내보내기 유형을 저장하며, 단일 `processing` 값은 해당 레벨만 저장합니다.
- `aligned=True` — 각 구성원의 비-원시(non-raw) 내보내기 데이터를 배열의 [정렬 프로필](#array-alignment)로 워프(co-registered)합니다. 원시(raw) 데이터는 워프되지 않지만 메타데이터에 변환 정보를 포함합니다. 배열에 프로파일이 없는 경우 정렬되지 않은 상태로 대체되며(결과물의 `alignment`에 경고가 표시됨),
- `render_index=False` — 카메라별 식생 지수 오버레이를 건너뜁니다. 기본값은 구성된 위치에 렌더링합니다.
- `force_daq=True` — 선택한 레벨에 필요하지 않은 경우에도 할당된 DAQ/DLS 판독값을 `.daq` 사이드카로 저장합니다.

**TIFF 압축 (HTTP -only 옵션):**`ArraySession.capture()`는 `compression` 키를 전송하지 않으므로 백엔드 기본값이 적용됩니다 — `POST /api/camera/array/capture`는 `compression` 본문 매개변수를 읽으며, `"deflate"`를 기본으로 읽습니다(무손실 zlib L1 + 수평 예측기, 전체 해상도 프레임당 약 4.1 MB). `"none"`는**~5배 더 빠른 쓰기 속도**를 자랑합니다 — 두 방식 모두 무손실이며, 가져올 때 동일하게 읽힙니다. SDK는 이에 대한 kwarg를 노출하지 않습니다; 우회 방법은 `chloros-cli lattice array-capture --compression none` 또는 원시 HTTP입니다. DEFLATE는 또한 Python GIL을 보유하므로, 따라서 압축된 쓰기 작업은 카메라별 쓰기 스레드 간에 병렬화되지 않습니다. 센서 속도로 8대의 카메라를 통해 풀 해상도 캡처를 지속적으로 수행하려면 `compression: "none"`가 필요합니다. 세부 정보: [CLI 참조 → array-capture](cli-reference.md).**멤버별 내보내기 재정의 (HTTP -only):**동일한 엔드포인트는 `exclude_serials`도 허용합니다(목록 — 저장된 세트에서 멤버를 제외; 배열은 여전히 하나의 동기화된 그룹으로 트리거되며, 제외된 멤버는 `excluded`로 반환됩니다), `serial_levels`(`{serial: [level tokens]}`-카메라 수준 재정의), `serial_index`(카메라별 인덱스 오버레이 재정의)를 지원합니다. 이들은 GUI와 동등한 본문 매개변수이며**아직 SDK 키워드 인수는 아닙니다**; 맵에 없는 멤버는 배열전체 `levels` / `render_index`로 대체됩니다.

##### 건너뛴 캠 검사 — `CaptureResult.skipped`

`ArraySession.capture()`는 `CaptureResult`를 반환하며, 이는 `list`의 서브클래스입니다: 반복 처리, 인덱싱, `len()` 처리 — 기존의 모든 패턴은 계속 작동합니다. 새로운 코드는 `.skipped` 속성을 검사하여 어떤 캠이 제외되었는지 및 그 이유를 확인할 수 있습니다. 가장 일반적인 경우는 `processing="radiance"` 또는 `"reflectance"`를 요청할 때 혼합-필터 배열 내의 RGB 캠들이 `processing="radiance"` 또는 `"reflectance"`를 요청할 때 — 광대역 센서에서는 베이어별 복사도가 의미가 없으므로, 백엔드는 무의미한 결과를 생성하기보다는 해당 캠들을 건너뜁니다.

```python
with chloros_sdk.connect_array(serials) as arr:
    result = arr.capture("output/", processing="reflectance")

    # Back-compat: iterate as a plain list
    for frame in result:
        print(frame["filepath"], frame["serial"])

    # New: see why N-1 cams were saved
    for skip in result.skipped:
        print(f"skipped SN:{skip['serial']} reason={skip['reason']}")
        # e.g. {'serial': '214701292', 'level': 'reflectance',
        #       'reason': 'reflectance-not-applicable-to-rgb-cam',
        #       'filter': 'RGB'}
```

이유 토큰은 `<level>-not-applicable-to-rgb-cam` 패턴을 따릅니다(건너뛴 레벨마다 하나의 항목이 있으며, 각 항목은 `level`를 포함합니다). 반사율 관련 건너뛰기 항목은 `reflectance-skipped-no-fresh-dls`(새로운 하향파 측정값 없음), `reflectance-skipped-bound-daq-unavailable (…)`(바인딩된 DAQ에 접근할 수 없음), 그리고 `dls-uncalibrated-band-<nm>` — 해당 대역이 대부분 DAQ 광 센서의 방사계학적 보정 범위 (~374–974 nm)의 대부분이 해당 범위를 벗어나므로, DAQ 기반의 절대 반사율 분할이 거부되고 프레임은 센서 응답 방식으로 강제로 전환됩니다. 출하 중인 SKU 중에서는 F988만 이 오류를 유발하며, 해당 카메라의 지원 경로는 반사율 패널 워크플로우입니다.

`processing` 레벨:

| 레벨 | 출력 |
| --- | --- |
| `"raw"` | 센서에서 직접 출력되는 단일 채널 베이어(모노 카메라: 단일 대역). |
| `"debayered"` *(SDK 기본값)* | 이선형 디모자이크를 통한 3채널 BGR (흑백 카메라: 1채널 그레이스케일). |
| `"radiance"` | 전체 방사계측 체인을 통해 산출된 float32 W/m²/sr/nm. 다중 스펙트럼 전용 — RGB 카메라는 건너뜁니다. |
| `"reflectance"` | uint16 0..32768 (Pix4D 호환); 절대 기준값을 위해 실시간 DAQ 페어링이 필요합니다. 다중 스펙트럼 전용. |
| `"display"` | GUI 미리보기와 일치하는 전체 체인(카메라 프로필에 따른 CCM + WB + 감마). |
| `"all"` | 각 카메라에 대해 **적용 가능한 레벨당 하나의 파일**(GUI의 &quot;모두 캡처&quot; / CLI 기본값과 일치). 반환된 `CaptureResult` 파일은 각 `(cam, level)`에 대해 하나의 프레임 딕셔너리를 포함하며, 각 딕셔너리에는 레벨이 포함되며, 적용되지 않는 레벨은 `.skipped`에 나타납니다. 반사율 프레임에 사용된 DAQ 측정값은 `.daq` 사이드카로 저장됩니다. |

> **참고 — 기본값은 CLI과 다릅니다.** `ArraySession.capture()`의 기본값은 `processing="debayered"`이며, `chloros-cli lattice array-capture` 명령의 기본값은 `processing="all"`입니다. CLI/GUI의 다단계 저장 기능을 반영하려면 SDK에서 `processing="all"`를 명시적으로 전달하십시오.

### 캡처 모드 및 레코더

어레이 표면은 GUI 캡처 패널을 반영합니다: 단일 / 연속 / 간격 / 최단 셔터 모드와 두 가지 레코더(라이브 컴포지트 비디오 및 원시 버스트 → 오프라인 재처리)가 있습니다.

```python
import time, chloros_sdk

with chloros_sdk.connect_array(serials) as arr:
    # Single (default) — one synced group
    arr.capture("out/", processing="reflectance")

    # Fastest — raw + .daq + combined index now, calibrate later
    arr.capture_fastest("flightline/")

    # Interval — one reflectance pass every 2 s, 5 passes (bounded so it ends)
    arr.capture_repeated("timelapse/", count=5, interval_s=2.0,
                         processing="reflectance",
                         on_capture=lambda i, r: print(f"pass {i}: {len(r)} frames"))

    # Combined-index video/GIF recorder (needs the combined live view streaming)
    with arr.record("monitoring/", fps=10, gif=True) as rec:
        time.sleep(30)
    print(rec.result["video_path"])

    # Raw-Bayer burst → offline reprocess into calibrated video(s)
    with arr.burst("capture/", duration_s=5) as b:
        pass
    out = arr.build_video(b.result["out_dir"], products=[
        {"kind": "per_cam", "level": "reflectance"},
        {"kind": "combined", "level": "index"}])
    print(out["outputs"])
```

- **`capture_repeated`**는 SDK의 연속/간격 루프입니다. 스크립트를 통해 이를 중단시킬 수 있는 `Ctrl+C`가 없으므로,**반드시** `count` 및/또는 `duration_s`를 전달해야 합니다(둘 중 하나에 도달하면 중지됨). `interval_s`는 각 패스의 시작 시점부터 측정됩니다(GUI와 일치). 나머지 kwargs는 `capture()`로 바로 전달됩니다.
- **`record`**는 *모니터링 등급*입니다: 화면에 표시되는 그대로의 실시간 결합 인덱스 합성 영상을 캡처하므로, 프레임이 수신되려면 결합 스트림이 열려 있어야 합니다. 어레이당 하나의 합성 레코더만 허용됩니다(이미 실행 중인 경우 예외가 발생합니다).
- **`burst` → `build_video`**는 *분석- 등급*입니다: `burst`는 원시 프레임과 프레임별 매니페스트, 그리고 `<output>/bursts/<base>/` 하위의 고유한 DLS 판독값마다 하나의 `.daq`를 &#x27;그랩 루프&#x27;전체 속도에서(체인 없음, exiftool 없음, 라이브 뷰 없음). `build_video`는 각 프레임을 가장 가까운 `.daq`와 시간적으로 매칭하고, 임포트 파이프라인의 방사도/반사율/지수 체인을 다시 실행합니다. `products`는 `{"kind": "per_cam"|"combined", "level": "radiance"|"reflectance"|"index"}`의 목록입니다(기본값: 결합된 지수). `burst().stop()`는 또한 최선의 노력을 다한 결합된-지수 빌드를 자동으로 트리거하며, 이는 중지 결과에서 `build_job`로 반환됩니다.

#### `RecorderHandle`

`ArraySession.record()` 및 `ArraySession.burst()`에 의해 반환됩니다. 컨텍스트 매니저로 사용하여 범위 종료 시 자동으로 중지하거나, 수동으로 제어할 수 있습니다.

| 멤버 | 설명 |
| --- | --- |
| `job_id` | 백엔드 작업 ID (문자열). |
| `kind` | `"composite"` (`record`에서 가져옴) 또는 `"raw"` (`burst`에서 파생됨). |
| `start_stats` | `start` 호출에 의해 반환된 딕셔너리. |
| `result` | 실행 중인 `None`; 중지 시 반환되는 최종 중지 결과 딕셔너리. |
| `stats(timeout=10.0)` | 실시간 작업 통계(기록된 프레임 수, 실제 fps, 경과 시간). |
| `stop(timeout=60.0)` | 레코더를 중지하고, 최종 결과를 반환 및 캐시합니다. 이멧포텐트(두 번째 호출 시 캐시된 결과를 반환합니다). |

```python
rec = arr.burst("capture/")
# ... drive manually ...
print(rec.stats()["frames"])
result = rec.stop()
print(result["out_dir"], result.get("build_job"))
```

### 이미 연결된 어레이에 연결하기 — `attach_array`

어레이가 이미 실행 중인 경우(GUI에서 열었거나, 이전 SDK 세션에서 `connect_array`를 호출한 경우), 재연결하는 대신 `attach_array`를 사용하여 해당 어레이의 핸들을 가져옵니다. `connect_array`는 <sn><id>해당 상황에서</id></sn> 항상 “카메라가 <sn>이미 배열에 </sn>있습니다”라는 오류 메시지를 <sn><id>반환합니다. 이는 멤버가 포함된풀에 속한 멤버에 대해 `/array/connect`를 POST하는 작업은 항등성이 보장되지 않기 때문입니다. `attach_array`는 `/api/camera/array/list`를 읽고 array_id 또는 serials 중 하나를 기준으로 일치 여부를 확인합니다.

```python
import chloros_sdk

# By serials (matches if every serial is a member of one existing array)
arr = chloros_sdk.attach_array(
    ["213800234", "214000533", "214701288", "214701292"])

# By array_id (when you've already noted it down)
arr = chloros_sdk.attach_array("array-1779862544497")

# attach_array returns the same ArraySession as connect_array
arr.capture("output/", processing="reflectance")
```

패턴: SDK 데스크톱 GUI와 공동 테넌트인 스크립트는 먼저 `attach_array`를 시도하고, 풀에 아직 어레이가 없는 경우 `connect_array`로 대체해야 합니다.

```python
import chloros_sdk

try:
    arr = chloros_sdk.attach_array(serials)
except chloros_sdk.ChlorosConnectError:
    arr = chloros_sdk.connect_array(serials)
```

> **중요 — 컨텍스트 매니저가 종료되면 연결이 끊어집니다.**`ArraySession.disconnect()`는 항상 `/array/disconnect`를 POST합니다. `CameraSession` / `DAQSensorSession`와 같이`CameraSession` / `DAQSensorSession`와 같은 ‘소유하지 않음’ 가드가 없습니다. GUI와 공동 테넌트 환경(co-tenanting)을 구성 중이며 범위 종료 시 어레이를 해제하지 않으려면,**`with` 블록**을 사용하지 마십시오** — 핸들을 일반 변수에 저장하고 명시적인 `disconnect()` 호출을 생략하십시오:
>
> ```python
> arr = chloros_sdk.attach_array(serials)
> arr.capture("output/", processing="reflectance")
> # … script ends; array stays up for the GUI
> ```

### 네트워크 분석 보조 도구

배열을 열기 전에 유용합니다 — 제안된 설정이 적합한지 미리 확인합니다:

```python
result = chloros_sdk.analyze_array_network(
    master_serial="214701288",
    slave_serials=["213800234", "214000533", "214701162"],
    width=2048, height=1536,
    pixel_format="BayerRG12",
    binning=1,
)

if result["status"] == "ok":
    print("Use the requested settings.")
elif result["status"] == "auto_capped_fps":
    r = result["recommended"]
    print(f"Keep the resolution; cap the trigger rate at {r['recommended_target_fps']} fps")
elif result["status"] == "auto_shrunk":
    r = result["recommended"]
    print(f"Shrink to {r['out_width']}x{r['out_height']} binning={r['binning']}")
elif result["status"] == "needs_force_slip":
    print("Sim-sync impossible on this wire; force_tier='slip-emit-and-capture' required")
```

`status`는 `ok` / `auto_capped_fps` / `auto_shrunk` / `needs_force_slip` 중 하나입니다(그 외의 경우 `error`). `auto_capped_fps`는 요청된 해상도가 RX 링에 상한이 설정된 트리거 속도에서만 RX 링에 적합함을 의미합니다. 해상도를 유지하고 `target_fps=result["recommended"]["recommended_target_fps"]`를 `connect_array`로 전달하십시오([예제 6](#6-capability-probe-before-connecting-a-4-cam-array) 참조).

**투영값 해석 방법** (GUI의 ‘어레이 설정’ 패널과 동일한 모델):

- **버스트(`frame_bytes_total`)는 각 카메라의 실제 픽셀 형식에 따라 카메라별로 합산됩니다.**모노**M3M**카메라는 전달된 `pixel_format` 값과 관계없이 Mono12 (2 B/px)를 스트리밍하므로, 4대의 카메라로 구성된 풀 해상도 프레임의 크기는**~25 MB**이며, 모든 픽셀이 8비트라고 가정했을 때의 ~12.6 MB와는 다릅니다. 백엔드는 각 카메라의 모델에서 해당 포맷을 파악합니다.
- **어드미턴스(`burst_fits_nic_ring`)는 드레인(drain)을 고려합니다**, 전체 버스트 대 링 방식이 아닙니다: 호스트가 RX 링을 캠이 채우는 속도보다 빠르게 비울 때 sim-emit이 적용됩니다. 10G 호스트 + 1 GbE 캠은**버스트가 링을 초과하더라도** 풀 해상도를 허용하지만, 1 GbE 호스트는 (`needs_force_slip` / `auto_shrunk`).
- **`achievable_fps_max`는 보수적인 직렬 검색 상한치**입니다 — `max(readout+emit, N×emit)`는 카메라별 송신량이 1 GbE 카메라 링크로 제한되며, 노출과 무관합니다. 예: 4개의 카메라로 구성된 풀 해상도 12비트 어레이의 경우 ~2.8 fps (런타임에서 측정된 ~2.7–3.0과 일치). 전체 모델: [CLI 참조 → 어레이 fps 및 버스트 모델](cli-reference.md#array-fps--burst-model).
- **오버서브스크립션(`oversubscribed: true`)이란 카메라당 N × 하한값이 충돌 방지 상한값을 초과하는 것을 의미합니다** — fps 필드(`achievable_fps_max` / `fps_bright` / `fps_dark`)는 0으로 읽히며, 자동 축소/비닝으로는 이 문제를 해결할 수 없습니다(이 기능들은 프레임당 바이트 수를 줄일 뿐, 초당 전송되는 바이트 수를 줄이는 것이 아닙니다). 해결책으로는 카메라 수를 줄이거나, 점보 프레임을 사용하거나, 또는 더 빠른 NIC를 사용하는 것입니다. `max_cams_collision_safe`는 상한치(1 GbE에서 1500 MTU 기준 풀 해상도 카메라 6대, 점보 프레임 사용 시 9대)를 보고합니다. 응답에는 또한 `aggregate_demand_bps`, `collision_safe_ceiling_bps`, `per_cam_floor_bps`(8 MB/s)도 포함됩니다. [오버서브스크립션](#over-subscription-the-per-cam-floor)을 참조하십시오.

### 검색 및 목록 표시

```python
chloros_sdk.discover_lattice_cameras()   # list all cams visible to the backend
chloros_sdk.list_cameras()               # cams currently in the pool
chloros_sdk.list_arrays()                # active arrays in the pool
```

---

## 스마트 AE / 스마트 캡처

LATTICE 어레이는 연결되는 즉시 백그라운드에서 지속적인 AE를 실행하지만, 하지만 새로 지정한 장면은 수렴되는 데 잠시 시간이 걸립니다. **스마트 캡처**는 이를 편리하게 처리해 주는 기능입니다. 각 카메라의 노출을 폴링하고, 전체 창에서 어레이가 안정화될 때까지 기다린 후 캡처를 실행합니다. 이는 GUI와 동등한 기능으로, 데스크톱 앱의 &quot;스마트&quot; 캡처 버튼은 동일한 백엔드 엔드포인트를 호출합니다.

```python
import chloros_sdk

with chloros_sdk.connect_array([
        "213800234", "214000533", "214701288", "214701292"]) as arr:
    # Initial pose
    arr.capture("pose_a/", processing="reflectance", smart=True)
    input("Move the rig, then press Enter...")
    # New pose — smart-capture waits for AE to re-settle automatically
    arr.capture("pose_b/", processing="reflectance", smart=True)
```

`ChlorosProject`(다음 섹션)를 통해 제어할 경우 더 많은 설정 옵션을 사용할 수 있습니다:

```python
proj.arrays["main_rig"].capture_smart(
    output_dir="out/",
    processing="reflectance",
    settle_timeout_s=5.0,           # max wait
    stability_window_s=1.5,         # exposure must hold steady this long
    exposure_tolerance_pct=5.0,     # %-spread allowed within the window
)
```

스마트 AE 정책은 기본적으로 보수적으로 설정되어 있습니다. 정밀한 방사계 측정이 필요한 작업에서는 `exposure_tolerance_pct` 값을 줄이고, &quot;대략적인 값&quot;만 필요한 빠르게 변화하는 장면에서는 이 값을 늘리십시오.

---

## DAQ 센서 세션

스펙트럼 센서(USB 기반 DAQ-U, BLE 기반 DAQ-M, 이더넷 기반 DAQ-E)를 위한 영구 백엔드 풀입니다. 카메라 표면과 동일하게 작동합니다: 스마트 감지, 풀 재사용, 항등적 연결.

### 스마트 감지 (제로 구성)

```python
import chloros_sdk

with chloros_sdk.connect_daq_sensor() as daq:
    print(daq.model, daq.transport, daq.address)
    for frame in daq.latest(n=10):
        spectrum = frame["spectrum"]   # list[float] (W/m²/nm if calibrated)
        is_sat = frame["is_saturated"]
        x, y, z = frame["x"], frame["y"], frame["z"]
        print(len(spectrum), is_sat)
```

우선순위: 이더넷 → BLE → USB. 명시적인 힌트를 하나만 전달하여 전송 방식을 고정할 수 있습니다.

### 고정된 전송 방식

```python
# DAQ-U on a specific serial port
daq = chloros_sdk.connect_daq_sensor(transport="usb", port="COM3")

# DAQ-M over BLE by MAC (implies transport="ble")
daq = chloros_sdk.connect_daq_sensor(mac="AA:BB:CC:DD:EE:FF")

# DAQ-E over Ethernet by hostname (implies transport="eth")
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-xxx.local")

# Tuning knobs
daq = chloros_sdk.connect_daq_sensor(
    port="COM3",
    integration_time=64,      # ms
    frame_avg=20,
    enable_ae=True,
    start_streaming=True,
)
```

### `DAQSensorSession` 메서드

| 메서드 | 설명 |
| --- | --- |
| `status(timeout=10.0)` | 풀 항목 요약 (스트리밍/녹화 상태, 파장 범위, 보정 SHA, 적분 시간, frame_avg, AE 상태). |
| `latest(n=1, timeout=10.0)` | 최근 N개까지의 스펙트럼 프레임을 반환합니다. |
| `stream_start()` / `stream_stop()` | 스트리밍 재개/일시 중지 (핸들은 열린 상태로 유지됨). |
| `record_start(output_dir=None, device_name=None)` | .daq 파일 녹화 시작. 파일 경로를 반환합니다. AWS 보정 번들이 없는 DAQ-U/M의 경우 거부됩니다 (DAQ-E는 예외). |
| `record_stop()` | 녹화 중지. `{path, rows}`를 반환합니다. |
| `disconnect()` | 풀에서 해제합니다. 연결되었으나 소유하지 않은 핸들의 경우 아무 작업도 수행하지 않습니다. |

> **캡 보정 프로파일(`cap_id`) 는 SDK 조절 기능이 아닙니다.** `connect_daq_sensor()` / `DAQSensorSession`는 `cap_id` 매개변수나 `set_cap` 메서드를 노출하지 않습니다. CLI(`chloros-cli daq pool-connect --cap-id …` / `chloros-cli daq pool-set-cap …`) 또는 백엔드의 `/api/daq` HTTP 경로를 통해 함대 용량 보정 프로필을 선택하십시오 (`/api/daq/connect` 및 `/api/daq/<id>/cap-id`는 `cap_id`를 허용합니다).

### 디스커버리 — 연결할 주소 찾기

`discover_daq_sensors()`는 USB / BLE / ETH를 스캔하여 *열 수 있는* 센서를 찾습니다. 이는 `discover_lattice_cameras()`에 대응하는 DAQ용 명령어이며, **DAQ-M의 BLE MAC**을 확인하는 유일한 방법입니다. DAQ-E에는 호스트명이 있고 DAQ-U에는 COM 포트가 있지만, MAC 주소는 기기에 인쇄되어 있지도 않고 OS에도 표시되지 않습니다.

```python
for s in chloros_sdk.discover_daq_sensors():
    print(s["transport"], s["address"], s["model"], s["extra"])
# ble  C3:D8:85:E0:0A:19  DAQ-M  {'name': 'NSP32_SPECTRUM'}
# usb  COM3               None   {'manufacturer': 'Intel'}

# `address` is exactly what connect_daq_sensor wants:
for s in chloros_sdk.discover_daq_sensors(transports=["ble"]):
    if s["model"] == "DAQ-M":
        daq = chloros_sdk.connect_daq_sensor(mac=s["address"])
```

| 필드 | 설명 |
| --- | --- |
| `transport` | `usb` \| `ble` \| `eth`. |
| `address` | COM 포트 / BLE MAC / 호스트 이름 — `connect_daq_sensor`로 전달되어 `port=` / `mac=` / `eth_host=`. |
| `display` | 사람이 읽을 수 있는 레이블. |
| `model` | `DAQ-U` \| `DAQ-M` \| `DAQ-E`, 또는 스캔에서 식별할 수 없는 포트의 경우 `None` (USB 직렬 어댑터는 프로브 없이는 구별할 수 없으므로, 알 수 없는 항목은 숨기지 않고 표시됨). |
| `extra` | 전송 방식별 세부 정보(BLE 광고 이름, USB 제조업체, DAQ-E IP/펌웨어/…). 빈 값은 생략됩니다. |

| 매개변수 | 기본값 | 설명 |
| --- | --- | --- |
| `transports` | 세 가지 모두 | 스캔을 제한하는 시퀀스(또는 CSV 문자열). 원하는 대상을 정확히 알고 있을 때 전달하는 것이 좋습니다 — BLE는 처리 속도가 느린 편입니다. |
| `scan_timeout` | 5 | 전송 방식별 스캔 창(초 단위); 백엔드는 1~20 사이로 제한합니다. |
| `timeout` | 60.0 | 전체 호출에 대한 `HTTP` 상한값(SDK의 다른 부분과 동일). |
| `auto_start_backend` | `True` | 실행 중인 로컬 백엔드가 없으면 로컬 백엔드를 생성합니다. 원격 `backend_url`의 경우 절대 생성하지 않습니다. |

> **풀에 이미 열려 있는 센서는 풀에 이미 열려 있는 센서는 표시되지 않습니다.** 연결된 BLE 주변기기는 광고를 중단하고, 열린 COM 포트는 탐지할 수 없으므로, 검색 결과는 *연결 가능한* 항목만 나열합니다. 무언가를 연결한 직후에는 결과가 비어 있는 것이 정상입니다. 이미 보유한 장치에 대해서는 `list_daq_sensors()`를 사용하십시오. 스캔을 실행할 수 없는 전송 프로토콜(bleak/zeroconf 미설치)은 오류 발생 대신 건너뛰어지므로, 블루투스가 없는 컴퓨터에서도 USB 및 ETH 응답을 받을 수 있습니다.

### 리스팅

```python
for s in chloros_sdk.list_daq_sensors():
    print(s["sensor_id"], s["model"], s["transport"], s["wavelength_range"])
```

### GUI/CLI와의 동시 사용

GUI에 이미 센서가 열려 있는 경우, Python에서 `connect_daq_sensor(port="COM3")`를 호출하면 `already_connected=True`로 표시된 핸들이 반환됩니다. 그러면 세션의 `disconnect()`는 아무 작업도 수행하지 않으므로, SDK 스크립트가 센서를 GUI 아래에서 떼어내지 않습니다.

### 직접 하드웨어 클래스 (백엔드 없음)

`daq_sdk`는 `chloros_sdk`에 의해 재수출되므로, 백엔드 없이도 프로세스 내에서 센서를 종단 간으로 제어할 수 있습니다:

> **사용 가능 여부:**`daq_sdk`는 Chloros 데스크톱 설치판에 포함되어 있으나, PyPI 패키지에는**포함되지 않습니다** — `pip install chloros-sdk`는 `lattice_sdk`를 제공하지만 `chloros_sdk.DAQ_AVAILABLE == False`는 제외합니다. 이 클래스를 사용하기 전에 해당 플래그를 확인하십시오. pip만 설치된 호스트에서는 로컬 전송 라이브러리가 필요 없는 [`connect_daq_sensor()`](#daq-sensor-sessions)를 통해 센서를 사용해야 하며, 이 경우 로컬 전송 라이브러리가 필요하지 않습니다.

```python
from chloros_sdk import DAQUSensor, DAQMSensor, DAQESensor, discover_all

# Discovery
for d in discover_all(timeout=3.0):
    print(d.model, d.display, d.address)   # USB serials: d.extra.get("serial_number")

# Direct DAQ-U
sensor = DAQUSensor(port="COM3")
sensor.connect()
sensor.start_streaming()
# ... use sensor.add_spectrum_callback(...) ...
sensor.stop()
```

GUI와 소유권을 공유하려는 경우에는 스마트 연결 경로(`connect_daq_sensor`)를 사용하는 것이 좋습니다. 센서를 독점적으로 소유하는 헤드리스 스크립트의 경우에는 직접 클래스를 사용하십시오.

---

## 프로젝트 자동화 — `ChlorosProject`

저장된 Chloros 프로젝트는 `cameras.json` + `sensors.json` + `project.json`를 포함하는 폴더입니다. `open_project`는 매니페스트를 불러오고, `connect_all`는 저장된 설정으로 모든 저장된 장치를 온라인 상태로 전환합니다. 이는 GUI를 통해 생성된 것과 동일한 하드웨어 상태입니다.

### 최소 예제

```python
import chloros_sdk

proj = chloros_sdk.open_project("/home/user/Chloros Projects/Field_A")
report = proj.connect_all(verbose=True)
print(report)  # {'cameras': {...}, 'arrays': {...}, 'sensors': {...}}

# Cameras and arrays are addressable by name OR serial / array_id
cam = proj.cameras["FrontLeft"]
cam.capture("./out", format="tiff", processing="reflectance")

arr = proj.arrays["main_rig"]
arr.capture("./out", format="tiff", processing="reflectance")

# Read a DAQ
spectrum = proj.sensors["Sky"].read()

# Trigger every device simultaneously
proj.capture_all("./out")

proj.disconnect_all()
```

또는 컨텍스트 매니저로 사용할 경우:

```python
with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    proj.arrays["main_rig"].capture("./out", processing="reflectance")
```

### `ChlorosProject` 메서드

| 메서드 | 설명 |
| --- | --- |
| `connect_all(cameras=True, arrays=True, sensors=True, verbose=False, align=None)` | 저장된 모든 장치를 탐색하고 연결합니다. 클래스별 연결 보고서를 반환합니다. `127.0.0.1:5000`에서 수신 대기 중인 백엔드가 있으면 이를 사용하며, 그렇지 않은 경우 아무런 메시지 없이 직접(백엔드 없음) `lattice_sdk` 장치 제어 — 백엔드를 생성하지 않습니다. |
| `disconnect_all()` | 모든 연결을 종료합니다. |
| `capture_all(output_dir=".")` | 모든 카메라에서 프레임 1개씩, 모든 센서에서 어레이 및 스펙트럼을 가져옵니다. |
| `stream(camera, overlays=False, fps=10.0)` | 지정된 카메라(또는 어레이)에서 BGR `numpy` 프레임을 생성하는 제너레이터. `overlays=False`는 직접적인 `lattice_sdk` 캡처 루프입니다(어레이는 `{serial: frame}` 딕셔너리를 생성합니다). `overlays=True`는 `ChlorosLocal.camera_stream()`를 거쳐 백엔드`/api/camera/<serial>/stream-annotated` MJPEG 피드를 통해 전달되며, 카메라에 저장된 `ui.overlay` 블록은 쿼리 매개변수로 전달됩니다. 백엔드 모드와 **독립형 카메라**가 필요합니다. 다이렉트 모드 카메라는 `RuntimeError`를 발생시킵니다(백엔드는 이 프로세스가 소유한 카메라를 가져올 수 없음). 또한 어레이는 `NotImplementedError`를 발생시킵니다(카메라별로 합성 오버레이 — 이름으로 구성원을 스트리밍). 일회성(one-shot)에 해당하는 명령: `CameraHandle.capture(annotated=True)`. |
| `align_arrays(align=True, verbose=False)` | 현재 연결된 모든 어레이에 대해 정렬을 실행합니다. |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | 프로젝트의 이미지에 대해 보정/인덱스 파이프라인을 실행합니다(`ChlorosLocal.process`를 감싸는 함수; 이 네 가지가 **유일하게** 허용되는 키워드 인자입니다 — `indices=` 등은 `TypeError` 오류를 발생시킵니다; 인덱스는 `ChlorosLocal.configure()`를 통해 설정합니다). `ChlorosLocal()`를 지연 생성하며, 이는 백엔드를 자동으로 시작합니다. |

속성:
- `proj.cameras` — `Dict[str, CameraHandle]`는 이름과 일련번호를 키로 사용합니다.
- `proj.arrays` — `Dict[str, ArrayHandle]`는 이름과 array_id를 키로 사용합니다.
- `proj.sensors` — `Dict[str, SensorHandle]`는 이름과 slot_id를 키로 합니다.
- `proj.config` — `project.json["config"]` 사전.

### `CameraHandle`

```python
cam = proj.cameras["FrontLeft"]

# Save a frame to disk (processing-aware)
filepath = cam.capture(
    output_dir="./out",
    format="tiff",
    processing="radiance",           # see the level table below
    apply_calibration=True,          # DSNU + flat + 3x3 unmix + NIST
    apply_white_balance=True,        # DLS-aware WB
    apply_index=False,
    index_expression=None,
)

# In-memory grab (numpy array)
frame = cam.grab(processing="debayered")
frame, header = cam.grab(processing="radiance", with_metadata=True)

# Frame iterator (generator)
for arr in cam.frame_stream(processing="debayered", fps=5, count=100):
    my_analysis(arr)
```

**처리 수준.** `capture()`, `grab()` 및 `frame_stream()`는 모두 동일한 `processing`
토큰을 사용하며, 체인은 누적적입니다. 즉, 각 레벨은 그 위의 모든 기능을 실행합니다:

| 레벨 | 출력 | 비고 |
| --- | --- | --- |
| `raw` | 1채널 베이어, 센서 네이티브 | 디모자이크 없음. 이 레벨에서는 오버레이를 사용할 수 없습니다. |
| `debayered` | 3채널 BGR (**기본값**) | 쌍선형 디모자이크. 백엔드 모드 없이 작동하는 유일한 레벨입니다. |
| `radiance` | float32, W/m²/sr/nm | 완전한 방사계 체인: 디모자이크 + 3×3 언믹스 (다중 스펙트럼) + DSNU + 플랫 필드 + NIST 스케일. 노출 × 게인이 제거되어 값이 절대값으로 표시됩니다. |
| `reflectance` | uint16, 32768 = 1.0 | 복사도를 하향 복사조도(ρ = π·L/E)로 나눈 값. DLS/DAQ 판독값 필요 — 아래 참고 사항 참조. |
| `display` | 8비트 sRGB 유사 | GUI와 동등한 렌더링: 카메라의 활성 색상 프로필을 통한 CCM + 화이트 밸런스 + 감마. |

`debayered` 이외의 모든 항목은 백엔드 모드가 필요합니다. 다이렉트 모드 카메라는
`NotImplementedError`를 발생시킵니다. `reflectance`에는 사용 가능한 하향 복사량 측정값이 필요합니다 — 프레임 종단점은
풀링된 DAQ를 카메라의 DLS 슬롯으로 자동으로 끌어오지만, DAQ가 바인딩되지 않은 경우 체인은
반사율 출력을 거부하고, 단순히 품질이 낮은 결과를 묵묵히 반환하는 대신 반환된 메타데이터에
등급 하향 조정을 명확히 기록합니다.

> **반사율 DN 스케일 — 이를 하드코딩하지 마십시오.** LATTICE 반사율은 `32768` = ρ 1.0을 사용하며
> XMP `Chloros:PixelScale=32768`; Survey3 반사율은 `65535` = ρ 1.0을 사용하며
> `Chloros:*` 태그는 포함하지 않습니다. 태그 값을 읽어 해당 값으로 나누십시오. 이 값은 uint16 도메인으로 정의되어 있으므로
> `32768`로 유지됩니다. (16비트 TIFF, 8비트 PNG /JPG, 32-비트 백분율) — 저장된 데이터 유형을 먼저 uint16으로 정규화해야 합니다
> (8비트에서 ×257, 부동소수점에서 ×65535). 단 한 가지 예외:
> 8비트-소스 캡처가 8비트 TIFF로 기록된 경우, 재조정되지 않고 *클리핑*되므로 이를 설명하는 스케일이 없습니다
> — Chloros는 해당 경우 `PixelScale` 및 MicaSense 튜플을 완전히 생략합니다. LATTICE 반사율 파일에서 누락된
> 태그는 &quot;유효한 스케일 없음&quot;으로 간주하며로 간주하고, 기본값으로 간주하지 마십시오.

> **EXIF가 내보내기 과정에 그대로 반영됨.** `process()`는 원본 캡처의 GPS 블록
> **및 해당 ExifIFD**를 모든 산출물에 복사하므로, 따라서 내보낸 파일에는 `FocalLength`, `FNumber`,
> `ExposureTime`, `ISO`, `DateTimeOriginal`, `ISO`, `DateTimeOriginal`, `CameraSerialNumber`와
> 지리 참조 정보도 함께 포함됩니다. `FocalLength`는 Pix4D가 지상 샘플 간격을 계산하는 데 사용하는 파일입니다. 이 파일이 없으면
> 재구성 결과의 축척이 극도로 부정확해집니다(실측 사례에서 411m 규모의 현장이
> 47.8km 규모로 변한 경우).. 이 파일은 의도적으로 `-all:all`가 아닙니다: IFD0의 구조 태그가
> LATTICE 출력을 손상시키기 때문이며, `ExifImageWidth`/`Height`는 내보낸 래스터가 아닌
> 캡처 과정을 기술할 뿐, 내보낸 래스터에 대한 설명이 아니기 때문입니다.

캡처 단계 하위 플래그 (방사계 수준에 적용됨 — `radiance`, `reflectance`, `display`):

| 플래그 | 기본값 | 의미 |
| --- | --- | --- |
| `apply_calibration` | `True` | DSNU + 플랫 필드 + 3x3 언믹스 + NIST 방사계 척도. |
| `apply_white_balance` | `True` | WB LUT. DAQ가 카메라에 바인딩된 경우 DLS 고려. |
| `apply_index` | `False` | 식생 지수 평가. |
| `index_expression` | `None` | 공식 재정의. 값이 비어 있지 않을 경우 → 지수가 자동으로 활성화됩니다. |
| `annotated` | `False` | GUI 장식(줄무늬/격자/피킹) 오버레이. `raw`에서는 사용할 수 없음. |

### `ArrayHandle`

```python
arr = proj.arrays["main_rig"]

# Single synced capture group
files = arr.capture("./out", format="tiff", processing="reflectance")
# → {"213800234": "/path/to/x.tif", "214000533": "/path/to/y.tif", ...}

# Multi-level: each serial's value becomes an ordered LIST, not a str
files = arr.capture("./out", processing="all")
# → {"213800234": ["/raw.tif", "/debayered.tif", ...], "combined": "/idx.tif"}

# Smart capture (wait for AE to settle)
result = arr.capture_smart(
    "./out", processing="reflectance",
    settle_timeout_s=5.0,
    stability_window_s=1.5,
    exposure_tolerance_pct=5.0,
)
print(result["frames"], result["settle"])

# In-memory grab: {serial: numpy array}
frames = arr.grab(processing="debayered")
frames = arr.grab(processing="radiance", with_metadata=True)

# Stream-to-disk loop
arr.stream(count=60, output_dir="./stream", fps=5, processing="raw")

# Frame-iterator (tolerates per-cam drops; great for downstream analysis pipelines)
for frames in arr.frame_stream(processing="radiance", fps=5, count=100):
    if "213800234" in frames:
        my_analysis_pipeline(frames["213800234"])

# Preview iterator (live MJPEG-equivalent; tolerates partial cycles)
counts = arr.preview_stream("./preview", fps=3.0, duration=30.0)
print(counts)  # frames written per serial
```

> **반환 유형은 `CapturePathMap`이며, `Dict[str, str]`가 아닙니다.**
> `chloros_sdk.CapturePathMap`는 `Dict[str, Union[str, List[str]]]`와 같습니다: 단일 레벨
> `processing`는 각 시리얼에 하나의 경로를 부여하는 반면, 다단계 구조(`"all"` 또는
> 명시적인 `levels` 목록)는 해당
> 카메라에 저장된 모든 제품의 **정렬된 목록** 을 제공합니다. 실시간 복합 합성 영상(스트리밍 중인 경우)은 일련 번호 아래가 아닌 별도의
> `"combined"` 키 아래에 도착합니다. `str`를 가정하는 코드는
> 목록 형식에서 타입 검사기가 아무런 이의를 제기하지 않더라도 오류가 발생합니다. 주석에는 목록 형식이 출시된 후 한동안 `Dict[str, str]`
> 이었기 때문에 이 별칭이 존재합니다. 평면 형식을 원할 때는
> 정규화하십시오:
>
> ```python
> paths = arr.capture(processing="all")
> flat = [p for v in paths.values()
>         for p in (v if isinstance(v, list) else [v])]
> ```

### 배열 정렬

`ArrayHandle`는 전체 정렬 표면을 노출합니다. 프로파일은 기본적으로 세션 단위로만 저장되므로, 영구적으로 저장하려면 `export_alignment()`를 명시적으로 호출하십시오.

```python
from chloros_sdk import AlignmentSpec

arr = proj.arrays["main_rig"]

# Defaults: ORB / affine / one synced snapshot — same as the GUI's auto-cal
result = arr.calibrate_alignment()
print(result["profile"]["rms_residual_px"])

# Custom spec for tough scenes (low-contrast canopy)
spec = AlignmentSpec(
    method="feature_orb",         # feature_orb / feature_akaze / phase_correlation / checkerboard / manual
    model="rigid",                # translation / rigid / affine / homography
    num_frames=5,
    max_features=8000,
    ratio_threshold=0.7,
    ransac_threshold_px=2.0,
    min_matches=30,
    max_reproj_err_px=2.0,
)
arr.calibrate_alignment(spec)

# Or tweak one knob at a time
arr.calibrate_alignment(num_frames=3, model="affine")

# Inspect / manipulate
status = arr.alignment_status()
arr.tweak_alignment("214701292", dx=2.5, dy=-1.0, rotation_deg=0.0, scale=1.0)
arr.export_alignment("/tmp/main_rig_alignment.json")
arr.import_alignment("/tmp/main_rig_alignment.json", validate=True)
arr.clear_alignment()
```

#### 연결 시 정렬

`connect_all(align=...)`는 연결 시 모든 어레이를 자동으로 정렬할 수 있습니다:

```python
# Align every array with defaults
proj.connect_all(align=True)

# Per-array control
proj.connect_all(align={
    "main_rig": AlignmentSpec(num_frames=5, model="affine"),
    "side_rig": True,             # use defaults
    "verify_rig": False,          # skip
})
```

지정되지 않은 경우 `project.json["config"]["auto_align_on_connect"]`로 대체됩니다.

### `SensorHandle`

```python
spectrum = proj.sensors["Sky"].read()
# (spectrum_list, is_saturated, integration_time, x, y, z) — matches the
# daq_sdk add_spectrum_callback signature.
```

---

## 직접 하드웨어(백엔드 없음)

백엔드(CI, 헤드리스 로봇, 임베디드)에 대한 의존성을 완전히 제거하려면 `lattice_sdk`와 `daq_sdk`를 직접 임포트하십시오. 두 모듈 모두 `chloros_sdk`에 의해 재수출됩니다. `CAMERA_AVAILABLE` / `DAQ_AVAILABLE`에 대한 주의 사항: `lattice_sdk`는 PyPI 패키지에 포함되어 있습니다 (단, Arena SDK 런타임이 설치되어 있어야 함)에 포함되어 있는 반면, `daq_sdk`는 데스크톱 설치판에만 포함되어 있습니다.

```python
from chloros_sdk import (
    # cameras
    LatticeCamera, CameraSettings, PRESETS, CameraPool,
    Calibration, CalibrationCoefficients, FilterModel, list_filters,
    DLS, NetworkDiagnostics, gpu_info, gpu_available,
    # discovery
    discover_cameras, discover_cameras_via_backend,
    # exceptions
    LatticeError, CameraNotFoundError, StreamError, CaptureError,
    CalibrationError, NetworkError, DLSError,
)

# Find a camera and capture in one go
cams = discover_cameras(timeout_ms=3000)
print(cams)

settings = PRESETS["high_quality"]
with LatticeCamera(serial="213800234", settings=settings) as cam:
    result = cam.capture(output_dir="./out", format="tiff")
    print(result.filepath, result.width, result.height)
```

##### 프리셋 및 트리거

네 가지 프리셋 중 세 가지가 **자유 실행(free-run)** 모드입니다: 카메라는 지속적으로 노출을 진행하며
`capture()`는 다음 프레임을 반환합니다. `triggered`는 예외로, 2번 라인에서 하드웨어 에지가 감지될 때
카메라를 준비 상태로 전환하므로, 따라서 신호가 도착할 때까지는 아무것도 캡처하지 않습니다.

| 프리셋 | 트리거 | 사용 시 |
| --- | --- | --- |
| `default` | 프리런 | 일반 용도 |
| `high_speed` | 프리런 | 8비트, 60 fps 제한, 짧은 노출 |
| `high_quality` | 프리런 | 12비트, fps 제한 없음 — 스틸 사진 촬영 시 일반적으로 선택 |
| `triggered` | **아머드, 라인 2** | 카메라가 M8 싱크 케이블에 연결되어 있고 다른 장치가 셔터를 작동시킴 |

`triggered`를 선택하면 (또는 직접 `trigger_mode="On"`를 설정할 경우) 라인 2를 제어하는 것이
아무것도 없으면, 모든 `capture()`는 타임아웃됩니다 — 카메라에
대기하도록 요청했기 때문에 이는 정상적인 현상입니다. SDK에서는 이러한 현상이 발생할 때 이에 대해 설명하고 있습니다.
[캡처 중 SC_ERR_TIMEOUT](#direct-hardware-backend-free)을 참조하십시오.

> **참고 — 연결 시 나타나는 &quot;GVSP probe&quot; / `SC_ERR_TIMEOUT -1011` 메시지는 오류가 아닙니다.**&gt; 연결 시 SDK는 더 높은 처리량을 위해**점보 프레임**(9000바이트 GVSP 패킷) 협상을 시도합니다. 직접적인 지점 간 NIC 링크(예:예: 링크 로컬 `169.254.x.x` 주소)에서는 네트워크가 일반적으로 점보 프레임을 전송할 수 없으므로, 이 프로브는 타임아웃이 발생하고 다음과 같은 로그가 기록됩니다:
>
> ```
> [Network] GVSP probe: unexpected error (TimeoutError: ... SC_ERR_TIMEOUT -1011)
> [Network] GVSP probe at 9000 did not deliver a complete buffer; reverting to ICMP-chosen size
> [Network] GVSP packet size: 1500 bytes (standard)
> ```
>
> 이는 **설계된 대체 방식**입니다: SDK는 자동으로 표준 1500바이트 패킷으로 되돌아가며, 카메라는 정상적으로 연결을 유지합니다(그 뒤에 나오는 `[chunk-enable …]` 줄들은 정상적인 연결 순서의 일부입니다). 캡처 기능은 여전히 작동합니다.
>
> 이 프로브를 건너뛸 수는 있지만, **단순히 로그 출력을 억제하는 기능이 아니라 점보 프레임을 비활성화합니다.** 네트워크 상태가 아무리 양호하더라도 카메라는 최대 1500바이트까지만 ‘Don&#x27;t-Fragment’ 핑에 응답하므로, 핑 테스트만으로는 점보 프레임을 절대 감지할 수 없습니다. 이 프로브만이 이를 감지할 수 있습니다. 이 프로브를 비활성화하면 카메라는 어떤 네트워크에서든 영구적으로 표준 1500바이트 패킷을 전송하게 됩니다:
>
> ```bash
> CHLOROS_GVSP_PROBE_FALLBACK=0   # gives up jumbo — see the warning it prints
> ```
>
> 이 설정은 *확실히* 점보 프레임을 처리할 수 없는 네트워크에서만 유용하며, 카메라당 연결 시간을 약 1초 정도 단축해 줍니다. 이는 단순한 외관상의 변경이 아닌 실질적인 절충이므로, 이 설정을 사용할 때 SDK에서 다음과 같이 안내합니다:
>
> ```
> [Network] ⚠️ GVSP probe disabled (CHLOROS_GVSP_PROBE_FALLBACK=0) — staying at
> 1500 bytes, jumbo NOT tested. … if this network does carry it, you are giving
> up ~1.45x wire ceiling. Unset the variable to test for jumbo.
> ```
>
> **특별한 이유가 없다면 그냥 놔두세요.** 이 기능을 활성화한 상태로 두면, 연결할 때마다 실제 네트워크 환경을 재측정합니다. 점보 패킷을 지원하는 스위치에 연결하면 다음 연결 시 자동으로 점보 패킷을 인식하며, 별도의 설정이나 재시작 없이도 가능합니다.
>
> 점보 전송 속도를 *원한다면*, 종단 간 점보를 활성화하십시오(NIC MTU 9000 + 점보를 통과시키는 스위치), 또는 링크가 점보 패킷을 처리한다는 것을 알고 있다면 `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000`로 고정하십시오. 다만, 고정된 크기는 프로브를 건너뛰고 앞쪽 네트워크에 대한 적응을 중단시키므로, 영구적으로 설정하는 것보다 명령어별로 `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000 python …`를 사용하는 것을 권장합니다. **경로상의 모든** 장치는 점보 패킷을 전달할 수 있어야 합니다. 여기에는 PoE 스플리터나 인젝터도 포함되며, 이는 점보 패킷을 지원할 수 있는 환경에서도 점보 패킷을 전달하지 못하는 일반적인 원인입니다.

> **`SC_ERR_TIMEOUT -1011`가 `capture()` / `grab*()` 중에 발생하는 것은 별개의 문제입니다 — 그 오류는 실제 오류입니다.**&gt; 위의 참고 사항은**connect-time probe**에 의해 기록된 `-1011`에 대해서만 해당됩니다.**캡처** 과정에서 발생한 동일한 오류는 카메라가 정상적으로 연결되었으나 이미지를 전송하지 않고 있음을 의미합니다:
>
> ```
> File ".../lattice_sdk/camera.py", line ..., in grab_frame_with_metadata
>   buffer = self._get_buffer(timeout)
> lattice_sdk.exceptions.CaptureError: Capture failed: ... SC_ERR_TIMEOUT -1011
> ```
>
> 결정적인 단서는 카메라의 *제어* 채널 상태가 양호하다는 점입니다 — 검색이 정상적으로 이루어지고, 설정 및 `[chunk-enable …]` 쓰기 작업이 모두 성공하지만 — 반면 *모든* 프레임 전송이 타임아웃되는 경우입니다.
>
> **일반적인 원인은 카메라가 하드웨어 트리거 모드로 설정되어 있기 때문입니다.** `trigger_mode="On"` 및 `trigger_source="Line2"` 오류의 경우, M8 동기화 케이블에 전기적 에지가 도달할 때까지 카메라가 아무것도 전송하지 않습니다. 해당 라인을 구동하는 케이블이 없다면, 모든 데이터 가져오기 작업이 영원히 대기하게 됩니다. 카메라는 고장 나지 않았고 네트워크도 정상입니다 — 지시받은 대로 정확히 작동하고 있는 것입니다.
>
> `CameraSettings()`와 `default` / `high_speed` / `high_quality`는 프리런(free-run)을 설정하며, 무장 상태에서 타임아웃이 발생한 캡처는 단순히 `-1011`만 출력하는 대신 그 원인을 설명합니다. `PRESETS["triggered"]`는 설계상 Line2를 무장시킵니다.
>
> 어떤 카메라든 프리런 모드로 강제 설정하려면:
>
> ```python
> settings = PRESETS["high_quality"]
> settings.trigger_mode = "Off"        # free-run; don't wait for an M8 edge
> ```
>
> `trigger_mode="Off"`를 사용해도 여전히 타임아웃이 발생한다면, 해당 카메라는 실제로 데이터를 전송하지 않는 것입니다 — 로그 파일과 `ip link show`를 저희에게 보내주십시오.

#### 색상 프로필 (RGB 라이브 미리보기) — `set_color_profile`

`LatticeCamera.set_color_profile(profile, custom_cct_k=None)`는 RGB 카메라의 **라이브 미리보기**에 사용할 디스플레이 색상 프로필을 선택합니다(다중 스펙트럼 카메라는 이 설정을 무시합니다):

| 프로파일 | 의미 |
| --- | --- |
| `raw` | 방사계 측정 체인을 완전히 우회합니다. |
| `linear` | DSNU + 플랫 + 화이트 밸런스, CCM 없음, 감마 없음. |
| `natural` | 선형 + 측정된 CCM + sRGB 감마, 저렴한 마무리 처리만 적용(채도 평활화 + 하이라이트 채도 감소) — 사실적인 기본 설정. |
| `enhanced` | `natural`에 전체 허브 패리티 마무리 (프린지 제거, 생동감, CLAHE 국소 대비) 적용. **프레임당 보정 비용이 약 두 배**로 증가하여 더 풍부한 화면을 제공하지만, LIVE 프레임 속도는 낮아집니다. |
| `custom_temp` | `natural`이지만, 화이트 밸런스가 `custom_cct_k` 켈빈으로 고정됨 (DLS 무시; 백엔드 측에서 2000–10000 K로 제한됨). |

이 프로파일은 **라이브 미리보기 전용** 속도/룩 조절 기능입니다: 저장된 캡처는 선택한 프로필과 관계없이 항상 풍부하고 완성도 높은 결과물을 얻으므로, 프레임 시간을 확보하기 위해 `natural`를 선택하더라도 디스크에 저장되는 결과물의 품질은 저하되지 않습니다. 알 수 없는 프로필을 선택하면 `ValueError`가 증가합니다; 클로르백엔드에 연결할 수 있는 경우 변경 사항은 해당 백엔드로도 POST되며, 다음 미리보기 프레임에 반영됩니다(백엔드가 없는 경우 direct- SDK 백엔드가 없는 사용자도 설정 변경 사항을 적용받습니다).

```python
with LatticeCamera(serial="214701292") as cam:   # RGB cam
    cam.set_color_profile("enhanced")            # richer look, lower LIVE fps
    cam.set_color_profile("custom_temp", custom_cct_k=5600)
```

#### 모노(M3M) 카메라 및 `Calibration`

모노 **M3M** 카메라(`M3M-<lens>-F<wavelength>`)는 단일 대역입니다. 즉, 하나의 그레이스케일 평면이 있으며, 베이어 모자이크나 3×3 스펙트럼 크로스톡 행렬이 없습니다. `Calibration`는 이를 인식하여 `is_mono` 플래그를 노출합니다. 반사율은 여전히 대역별 방사계 맵으로 적용되지만 (분리 연산은 단위 행렬입니다)로 적용되지만, 단일 카메라에 대한 다중 대역 연산은 무의미한 결과를 반환하기보다는 다음과 같은 결과를 산출합니다:

```python
from chloros_sdk import Calibration, CalibrationError

calib = Calibration("M3M-L87-F685")
print(calib.is_mono)        # True  (False for any M3C / RGN Bayer cam)
print(calib.filter_type)    # 'mono'  (sentinel; not a real crosstalk key)

# NDVI needs two bands (Red + NIR); one mono band can't supply both.
try:
    calib.compute_ndvi(reflectance_frame)
except CalibrationError as e:
    print(e)   # "...single-band mono (M3M) camera. Combine multiple..."
```

모노크롬 하드웨어로 식생 지수를 구축하려면, 서로 다른 파장의 여러 M3M 카메라를 정렬된 다중 대역 스택으로 결합한 후(참조: [어레이 정렬](#array-alignment)), 단일 카메라가 아닌 해당 스택 전체에 대해 지수를 계산하십시오.

DAQ 직접 모드:

```python
from chloros_sdk import (
    DAQUSensor, DAQMSensor, DAQESensor,
    SensorFleet, discover_all, DiscoveredSensor,
    apply_sensor_settings, SensorSettings,
)

for d in discover_all(timeout=3.0):
    print(d)

sensor = DAQUSensor(port="COM3")
sensor.connect()
apply_sensor_settings(sensor, settings={"integration_time_ms": 64, "frame_avg": 20})
sensor.start_streaming()
# ... sensor.add_spectrum_callback(your_callback) ...
sensor.stop()
```

> **`apply_sensor_settings` 허용 키**— 정확히 `integration_time_ms`, `frame_avg`, `ae_enabled`, `sunshine_diffuser_installed` (DAQ-E; `cap_id`로 대체되어 더 이상 사용되지 않음), `filter_model` (DAQ-M), 및 `cap_id` (모든 DAQ 종류; `None`/`""`/`"none"` = 베어 센서, 캡 보정 없음). 알 수 없는 키는**무시**됩니다. 예를 들어, `{"integration_time": 64}`는 아무 작업도 수행하지 않습니다(`integration_time_ms`여야 함). `{"applied": [...], "errors": {...}}`를 반환하며 예외를 발생시키지 않습니다.

`chloros_sdk`는- 위에서 사용된 핵심 표면만 재수출합니다. 전체 `daq_sdk` 공개 API (22개 이름)에는 다음이 추가됩니다 — `daq_sdk`에서 직접 가져오세요:

```python
from daq_sdk import (
    DAQULogger, DAQMLogger, DAQELogger,     # rotating-file recorders (the ones the GUI uses)
    ConnectResult, FleetRecordResult,       # SensorFleet result types
    discover_all_detailed, build_sensor,    # detailed discovery + build-by-descriptor
    scan_eth_devices, DaqEControl,          # DAQ-E Ethernet scan + control channel
    scan_ble_devices, detect_ble_device, list_ble_devices,   # DAQ-M BLE discovery
    detect_port, list_serial_ports,         # DAQ-U serial-port discovery
    TcpSerial,                              # serial-over-TCP transport shim
)
```

---

## 예외

&quot;Chloros에서 발생한 모든 오류&quot;를 처리하기 위해 기본 클래스를 캐치합니다:

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

> `ChlorosAuthenticationError` 및 `ChlorosConfigurationError`는 나머지 항목들과 함께 최상위 수준에서 내보내집니다. 또한 표시된 바와 같이 `chloros_sdk.exceptions`에서도 가져올 수 있습니다.

계층 구조:

```

ChlorosError
├── ChlorosBackendError           (backend failed to start / unreachable)
├── ChlorosConnectionError        (HTTP transport failure)
├── ChlorosLicenseError           (subscription / tier gate)
├── ChlorosAuthenticationError    (login required)
├── ChlorosConfigurationError     (bad configure() / open_project() inputs)
└── ChlorosProcessingError        (pipeline failed)

ChlorosConnectError                (raised by connect_camera / connect_array /
                                    connect_daq_sensor only — derives from
                                    plain Exception, NOT from ChlorosError,
                                    so `except ChlorosError` will not catch it)

lattice_sdk exceptions:
LatticeError
├── CameraNotFoundError
├── CameraConnectionError
├── StreamError
├── CaptureError
├── CalibrationError
├── NetworkError
└── DLSError
```

---

## 종단 간 예제

### 1. 사용자 지정 진행률 표시줄을 사용하여 폴더 처리하기

```python
from chloros_sdk import ChlorosLocal

def progress(percent, message):
    bar = "#" * (percent // 5)
    print(f"\r[{bar:<20s}] {percent:3d}% {message}", end="", flush=True)

with ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26")
    cl.import_images("C:/DroneImages/Flight001", recursive=True)
    cl.configure(
        debayer="High Quality (Faster)",
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "NDRE", "GNDVI", "SAVI"],
        export_format="TIFF (16-bit)",
    )
    cl.process(progress_callback=progress)
print()
```

### 2. 실시간 LATTICE 어레이 → 반사율 + DAQ 참조

```python
import chloros_sdk

# Open a paired sensor first so the array's reflectance step has an
# absolute reference. Smart-detect picks USB / BLE / ETH automatically.
with chloros_sdk.connect_daq_sensor() as daq:
    with chloros_sdk.connect_array([
            "213800234", "214000533", "214701288", "214701292"
    ]) as arr:
        # Smart capture: wait for AE to settle, then snap
        arr.capture("./out", processing="reflectance", smart=True)

        # Record the corresponding DAQ frames as ground truth
        daq.record_start(output_dir="./out", device_name="sky-reference")
        # ... do whatever capture campaign ...
        info = daq.record_stop()
        print(info["path"], info["rows"])
```

### 3. 프로젝트 기반 캡처 캠페인

```python
import time, chloros_sdk

with chloros_sdk.open_project("/home/user/Chloros Projects/Field_A") as proj:
    report = proj.connect_all(verbose=True, align=True)
    if report["arrays"]["errors"]:
        raise SystemExit(f"Array(s) failed to connect: {report['arrays']['errors']}")

    rig = proj.arrays["main_rig"]

    # Re-align right before the campaign
    rig.calibrate_alignment(num_frames=5)
    rig.export_alignment("./alignments/main_rig.json")

    # 50 sequential single-frame captures at 2 fps
    for i in range(50):
        frames = rig.capture(
            output_dir=f"./out/frame_{i:04d}",
            processing="reflectance",
            apply_calibration=True,
            apply_white_balance=True,
        )
        time.sleep(0.5)

    # End-of-day: process the captured folder. process() accepts only
    # mode/wait/progress_callback/poll_interval — indices come from the
    # project's saved config (or set them via ChlorosLocal.configure()).
    proj.process()
```

### 4. 멀티 카메라 프레임 스트림 → NumPy 파이프라인

```python
import chloros_sdk
import numpy as np

with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    rig = proj.arrays["main_rig"]

    for frames in rig.frame_stream(
            processing="radiance",
            fps=5.0, count=300,
            apply_calibration=True,
            apply_white_balance=True):
        # frames is {serial: numpy_array}; cams not delivering this tick are omitted
        for serial, frame in frames.items():
            print(serial, frame.shape, frame.dtype, frame.mean())
```

### 5. 헤드리스 다이렉트 하드웨어(백엔드 없음) 캡처 스크립트

```python
from chloros_sdk import LatticeCamera, PRESETS, discover_cameras

cams = discover_cameras(timeout_ms=3000)
print(f"Found {len(cams)} cams")

settings = PRESETS["high_quality"]
for c in cams:
    with LatticeCamera(serial=c.serial, settings=settings) as cam:
        result = cam.capture(output_dir="./out", format="tiff")
        print(c.serial, result.filepath)
```

### 6. 4-Cam 어레이 연결 전 기능 확인

```python
import chloros_sdk

serials = ["214701288", "213800234", "214000533", "214701162"]

probe = chloros_sdk.analyze_array_network(
    master_serial=serials[0],
    slave_serials=serials[1:],
    width=2048, height=1536,
    pixel_format="BayerRG12",
)

if probe["status"] == "ok":
    arr = chloros_sdk.connect_array(
        serials, width=2048, height=1536, pixel_format="BayerRG12")
elif probe["status"] == "auto_capped_fps":
    r = probe["recommended"]
    print(f"Keeping resolution; capping trigger rate at "
          f"{r['recommended_target_fps']} fps")
    arr = chloros_sdk.connect_array(
        serials, width=2048, height=1536, pixel_format="BayerRG12",
        target_fps=r["recommended_target_fps"])
elif probe["status"] == "auto_shrunk":
    r = probe["recommended"]
    print(f"Auto-shrinking to {r['out_width']}x{r['out_height']} "
          f"binning={r['binning']} for sim-sync")
    arr = chloros_sdk.connect_array(
        serials,
        width=r["out_width"], height=r["out_height"],
        pixel_format=r["pixel_format"], binning=r["binning"])
elif probe["status"] == "needs_force_slip":
    print("Wire can't sustain sim-sync; falling back to slip mode")
    arr = chloros_sdk.connect_array(
        serials, force_tier="slip-emit-and-capture")
else:
    raise RuntimeError(f"Probe error: {probe.get('error')}")
```

### 7. 캡처 레시피와 동등한 방식 (순수 Python)

CLI의 레시피 DSL에는 직접적인 Python에 상응하는 기능이 있습니다:

```python
import time, chloros_sdk

with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    cam = proj.cameras["FrontLeft"]
    rig = proj.arrays["main_rig"]
    sky = proj.sensors["Sky"]

    # apply
    # (CameraHandle has no direct apply method; use the underlying lattice_sdk
    #  helper or the backend's /api/camera/<sn>/apply-settings via requests)
    # For most cases just use cam.cam.set_exposure(...) in direct mode or
    # the GUI's saved settings via project.connect_all().

    # wait
    time.sleep(2)

    # capture
    cam.capture("pose_a/", format="tiff", processing="radiance")

    # stream
    rig.stream(count=60, fps=5, output_dir="stream/", processing="raw")

    # sensor read
    print(sky.read())
```

---

## 백엔드 자동 시작

스마트 연결 진입점 — `connect_camera`, `connect_array`, `connect_daq_sensor`, `discover_lattice_cameras` —는 백엔드가 `127.0.0.1:5000`(스마트-connect 표면의 기본값URL)에서 수신 대기하고 있다고 가정합니다. GUI 또는 CLI가 이미 실행 중이라면 백엔드도 실행 중일 것입니다. 스크립트만 실행된 상태에서는 백엔드가 없을 수 있으므로, 이 함수들은 **번들로 제공된 백엔드 바이너리를 자동으로 시작**(`ChlorosLocal`와 마찬가지로 창 없이)를 첫 호출 전에**자동으로 시작**한 다음, `backend_startup_timeout`까지 기다려 백엔드가 준비되기를 기다립니다.

규칙:

- **URL는 로컬에서만 생성됩니다.** X000747가 `localhost` / `127.0.0.1` / `[::1]`를 가리키는 경우에만 유효합니다. 그 외의 호스트는 타인의 컴퓨터로 간주되어 절대 생성되지 않습니다.
- **백엔드는 재사용을 위해 계속 실행 상태로 유지됩니다** (CLI와 동일) — 스크립트가 종료되어도 암시적인 종료는 발생하지 않습니다. 스크립트를 다시 실행하면 실행 중인 백엔드가 재사용됩니다.
- 해당 호출 중 어느 곳에서든 **`auto_start_backend=False`**를 사용하여 이 기능을 해제할 수 있습니다(예: 원격 백엔드를 지정했거나 백엔드 수명 주기를 직접 관리하는 경우).

```python
import chloros_sdk

# Fresh shell, no backend running, no GUI open — this still works:
with chloros_sdk.connect_camera("213800234") as cam:   # spawns the backend
    cam.capture("output/")

# Remote backend (via tunnel — see Remote-Backend Mode): don't spawn one locally
arr = chloros_sdk.connect_array(serials,
                                backend_url="http://127.0.0.1:5000",
                                auto_start_backend=False)
```

번들된 바이너리를찾거나 시작할 수 없는 경우, 이후의 HTTP 호출은 단순한 연결 거부 트레이스 대신 조치 가능한 **플랫폼 인식** `ChlorosConnectError`를 발생시킵니다. Windows에서는 데스크톱 앱이나 `chloros-cli` 명령어로 안내하며, Linux (GUI 없음)에서는 `chloros-cli` 명령어나 `.deb`로 안내합니다.

---

## 환경 및 헤더

SDK는 모든 백엔드 HTTP 호출에 `X-Chloros-Client: sdk` 태그를 부여합니다. 이 백엔드는 GUI 무료 티어 경로 대신 SDK / CLI 라이선스 규칙(로그인 **및** 유료 Chloros+ 플랜 필수)을 적용합니다. 이는 가져오기 시점에 자동으로 설정되므로 별도의 조치가 필요하지 않습니다.

`http://localhost` 및 `http://127.0.0.1`는 로컬 백엔드로 감지됩니다. 다른 호스트(예: 자체 분석 서비스)에 대한 호출은 변경되지 않습니다.

URL에 `backend_url=`(또는 `api_url=`인 경우 `ChlorosLocal`)를 전달하여 백엔드를 재정의하십시오:

```python
chloros_sdk.connect_camera("213800234", backend_url="http://127.0.0.1:5000")
chloros_sdk.connect_array(serials, backend_url="http://127.0.0.1:5000")
chloros_sdk.connect_daq_sensor(eth_host="daq-e-1.local",
                                backend_url="http://127.0.0.1:5000")
chloros_sdk.ChlorosLocal(backend_url="http://127.0.0.1:5000")
```

(루프백이 아닌 `backend_url`는 소스/dev 백엔드에만 연결됩니다. 기본 제공 백엔드는 루프백에만 바인딩됩니다. 터널 패턴에 대해서는 &#x27;원격 백엔드 모드&#x27;를 참조하십시오.)

---

## 버전 관리 및 호환성

- SDK 버전은 `chloros_sdk.__version__`로 노출됩니다.
- SDK는 동작을 번들된 백엔드 버전에 고정합니다. 구버전 SDK와 최신 백엔드를 함께 사용하는 것은 대개 정상적으로 작동하지만(전방 호환 엔드포인트), 최신 SDK와 구버전 백엔드를 함께 사용할 경우 새 엔드포인트에서 `404` 오류가 발생할 수 있습니다. 이 경우 데스크톱 앱을 해당 버전에 맞게 업그레이드하십시오.
- 스마트 커넥트 인터페이스(`connect_camera` / `connect_array` / `connect_daq_sensor`)와 네트워크 분석 엔드포인트는 안정적인 JSON 스키마를 반환하며, 새로운 필드는 추가되는 형태입니다.

---

## 문제 해결 지침

- **`ChlorosAuthenticationError: Login required`** → 이 컴퓨터에서 `chloros-cli login EMAIL PASSWORD`를 한 번 실행하거나, Chloros 데스크톱 앱을 통해 로그인하십시오.
- **`ChlorosConnectError: No Chloros backend is running …`** → 스마트 커넥트 호출이 로컬 백엔드를 자동으로 시작하므로, 이 메시지는 번들된 바이너리를찾을 수 없거나 시작할 수 없는 경우(예: 데스크톱 패키지가 없는 pip 전용 호스트)에만 표시됩니다. 이 메시지는 플랫폼에 따라 다르게 표시됩니다: Windows에서는 데스크톱 앱을 열거나 `chloros-cli` 명령을 실행하십시오; Linux에서는 `chloros-cli` 명령을 실행하십시오(GUI가 없음) 또는 `.deb`를 설치하십시오. 원격 백엔드의 경우 `backend_url=`(및 `auto_start_backend=False`)를 전달하십시오.
- **`CAMERA_AVAILABLE == False`** 가져오기 시 → `lattice_sdk` 로딩 실패(일반적으로 Arena SDK 런타임 DLL이 설치되지 않았기 때문). 카메라가 아닌 표면은 여전히 작동합니다.
- **Array connect가 서브 네이티브 해상도를 반환함**→ 백엔드의 스마트 프리프(smart-prep) 기능이 와이어에 맞도록 프레임 크기를 자동으로 축소합니다. `analyze_array_network()`를 사용하여 원인을 확인한 후, 링크를 업그레이드하거나 축소를 수용하거나, 또는 순차 캡처를 위해 `force_tier="slip-emit-and-capture"`를 전달하십시오. 이 축소 안전 장치는**집계 오버**-구독(`oversubscribed: true`, fps 필드 0)에 대해서는 적용되지 않습니다: 와이어에 비해 카메라 수가 너무 많은 경우, 비닝/ROI로는 해결할 수 없습니다 — 카메라 수를 줄이거나, 점보 프레임을 활성화하거나, 더 빠른 NIC로 교체하십시오([오버서브스크립션](#over-subscription-the-per-cam-floor) 참조).
- **`analyze_array_network()`에서 NIC 수신 링 크기가 매우 작음(~0.26 MB)을 보고하거나 “FRAMES WILL DROP” 메시지와 함께 연결 게이트가 발생함** → 호스트 NIC의 수신 링이 기본값으로 설정되어 있습니다(NIC 드라이버 업데이트 후 종종 32로 재설정됨). Realtek USB 10GbE 어댑터의 경우 `ReceiveBufferLen=256` 및 `PendingReceives=64` (권한 상승)을 설정한 후, 백엔드를 재시작하여 링을 다시 읽도록 하십시오. 전체 절차: [CLI 참조 → 호스트 NIC 설정 및 튜닝](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **재시작/종료 시 호스트가 응답하지 않고, 이후 WMI 오류(`Invalid class`) 발생 / NIC 활성화 불가** → 오래된 USB 10GbE 드라이버로 인해 `DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`) 오류 발생. 어댑터 드라이버를 최신 버전(2026 이상)으로 업데이트하고 수신 링 설정을 다시 적용하십시오. [CLI 참조 → 호스트 NIC 설정 및 튜닝](cli-reference.md#host-nic-setup--tuning-lattice-arrays)을 참조하십시오.
- **반사율 측정 거부** → 절대 스케일 반사율을 측정하려면 활성 DAQ를 카메라(또는 어레이)에 바인딩해야 합니다. GUI를 통해 바인딩하거나, 페어링된 센서가 필요 없는 `processing="radiance"`(W/m²/sr/nm)를 사용하십시오.
- **`smart=True` 캡처 시간이 예상보다 오래 걸림** → AE 수렴은 장면의 동적 특성에 따라 달라집니다. 더 빠른(안정성은 떨어지는) 트리거를 원한다면 `exposure_tolerance_pct`의 간격을 좁히거나 `stability_window_s`의 시간을 단축하십시오.

---

## 참조

- [CLI 참조](cli-reference.md) — 모든 CLI 하위 명령은 SDK 호출을 반영합니다.
- [DAQ 센서 가이드](../daq/README.md) — 센서별 배선, 보정 및 기록 규칙.
- 온라인 문서: `https://mapir.gitbook.io/chloros/api-python-sdk`</id></sn>
