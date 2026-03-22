# API : Python SDK

**Chloros Python SDK** 는 Chloros 이미지 처리 엔진에 대한 프로그래밍 방식의 접근을 제공하여, 자동화, 사용자 지정 워크플로우, 그리고 귀하의 Python 애플리케이션 및 연구 파이프라인과의 원활한 통합을 가능하게 합니다.

### 주요 기능

* 🐍 **네이티브 Python** - 이미지 처리를 위한 깔끔하고 파이썬 스타일의 API
* 🔧 **API에 대한 완전한 액세스** - Chloros 처리에 대한 완벽한 제어
* 🚀 **자동화** - 맞춤형 일괄 처리 워크플로우 구축
* 🔗 **통합** - 기존 애플리케이션에 Chloros 임베딩
* 📊 **연구용** - 과학적 분석 파이프라인에 최적화
* ⚡ **병렬 처리** - CPU 코어 수에 따라 확장 가능 (Chloros+)

### 요구 사항

| 요구 사항          | 세부 정보                                                             |
| -------------------- | ------------------------------------------------------------------- |
| **Chloros 설치됨** | Windows: 데스크톱 설치 프로그램; Linux: `.deb` 패키지                  |
| **라이선스**          | Chloros+ ([유료 플랜 필요](https://cloud.mapir.camera/pricing)) |
| **운영 체제** | Windows 10/11 (64비트), Linux x86_64 (amd64), Linux arm64 (NVIDIA Jetson JetPack 6) |
| **Python**           | Python 3.7 이상                                                |
| **메모리**           | 최소 8GB RAM (16GB 권장)                                  |
| **인터넷**         | 라이선스 활성화에 필요                                     |

{% hint style="warning" %}
**라이선스 요구 사항**: Python 및 SDK는 API에 액세스하기 위해 유료 Chloros+ 구독이 필요합니다. 스탠다드(무료) 요금제에서는 API/SDK에 액세스할 수 없습니다. 업그레이드를 원하시면 [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)를 방문하세요.
{% endhint %}

## 빠른 시작

### 설치

pip를 통해 설치:

```bash
pip install chloros-sdk
```

{% hint style="info" %}
**초기 설정**: SDK를 사용하기 전에, Chloros+ 라이선스를 활성화하려면 Chloros, Chloros (브라우저) 또는 Chloros CLI를 열고 자격 증명으로 로그인하여 Chloros+ 라이선스를 활성화하십시오. 이 작업은 한 번만 수행하면 됩니다. Linux(GUI 없음)에서는 다음을 사용하십시오: `chloros-cli login user@example.com 'password'`
{% endhint %}

### 기본 사용법

몇 줄의 명령어로 폴더를 처리합니다:

```python
from chloros_sdk import process_folder

# One-line processing (Windows)
results = process_folder("C:\\DroneImages\\Flight001")

# One-line processing (Linux)
results = process_folder("/home/user/drone_images/flight001")
```

{% hint style="info" %}
**크로스 플랫폼 경로**: 이 페이지의 코드 예제는 Windows 형식의 경로(예: `C:\\DroneImages\\Flight001`)를 사용합니다. Linux에서는 대신 Linux 형식의 경로를 사용하십시오(예: `/home/user/drone_images/flight001` 또는 `~/drone_images/flight001`). SDK는 두 플랫폼에서 동일하게 작동합니다.
{% endhint %}

### 전체 제어

고급 워크플로우용:

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK
chloros = ChlorosLocal()

# Create project
chloros.create_project("MyProject", camera="Survey3N_RGN")

# Import images
chloros.import_images("C:\\DroneImages\\Flight001")  # Windows
# chloros.import_images("/home/user/drone_images/flight001")  # Linux

# Configure settings
chloros.configure(
    vignette_correction=True,
    reflectance_calibration=True,
    indices=["NDVI", "NDRE", "GNDVI"]
)

# Process images
chloros.process(mode="parallel", wait=True)
```

***

## 설치 가이드

### 필수 조건

SDK를 설치하기 전에 다음이 설치되어 있는지 확인하십시오:

1. **Chloros 설치** — Windows: 데스크톱 설치 프로그램 ([다운로드](download.md)); Linux: `.deb` 패키지 ([Linux 설치](linux/linux-installation.md))
2. **Python 3.7+** 설치됨 ([python.org](https://www.python.org))
3. **유효한 Chloros+ 라이선스** ([업그레이드](https://cloud.mapir.camera/pricing))

### pip를 통한 설치

**표준 설치:**

```bash
pip install chloros-sdk
```

**진행 상황 모니터링 지원 포함:**

```bash
pip install chloros-sdk[progress]
```

**개발용 설치:**

```bash
pip install chloros-sdk[dev]
```

### 설치 확인

SDK가 올바르게 설치되었는지 테스트합니다:

```python
import chloros_sdk
print(f"Chloros SDK version: {chloros_sdk.__version__}")
```

***

## 초기 설정

### 라이선스 활성화

SDK는 Chloros, Chloros(브라우저), Chloros 및 CLI와 동일한 라이선스를 사용합니다. GUI 또는 CLI를 통해 한 번 활성화하십시오:**Windows:** **Chloros 또는 Chloros (브라우저)**를 열고 사용자 <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> 탭에서 로그인하거나 CLI를 사용하십시오.**Linux:** CLI를 사용하십시오(GUI 사용 불가):

```bash
chloros-cli login user@example.com 'your_password'
```

라이선스는 로컬에 캐시되며 재부팅 후에도 유지됩니다.

{% hint style="success" %}
**일회성 설정**: GUI 또는 CLI를 통해 로그인한 후, SDK는 자동으로 캐시된 라이선스를 사용합니다. 추가 인증이 필요하지 않습니다!
{% endhint %}

{% hint style="info" %}
**로그아웃**: SDK 사용자는 `logout()` 메서드를 사용하여 프로그래밍 방식으로 캐시된 자격 증명을 지울 수 있습니다. API 참조 문서의 [logout() 메서드](#logout)를 참조하십시오.
{% endhint %}

### 연결 테스트

SDK가 Chloros에 연결할 수 있는지 확인합니다:

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK (auto-starts backend if needed)
chloros = ChlorosLocal()

# Check status
status = chloros.get_status()
print(f"Backend running: {status['running']}")
```

***

## API 참조

### ChlorosLocal 클래스

로컬 Chloros 이미지 처리를 위한 메인 클래스입니다.

#### 생성자

```python
ChlorosLocal(
    api_url="http://localhost:5000",     # Backend URL
    auto_start_backend=True,             # Auto-start backend if not running
    backend_exe=None,                    # Backend path (auto-detected)
    timeout=30,                          # Request timeout (seconds)
    backend_startup_timeout=60           # Backend startup timeout
)
```

**매개변수:**

| 매개변수                 | 유형 | 기본값                   | 설명                           |
| ------------------------- | ---- | ------------------------- | ------------------------------------- |
| `api_url`                 | str  | `"http://localhost:5000"` | 로컬 Chloros 백엔드의 URL          |
| `auto_start_backend`      | bool | `True`                    | 필요한 경우 백엔드를 자동으로 시작 |
| `backend_exe`             | str  | `None` (자동 감지)      | 백엔드 실행 파일 경로            |
| `timeout`                 | int  | `30`                      | 요청 타임아웃 (초)            |
| `backend_startup_timeout` | int  | `60`                      | 백엔드 시작 타임아웃 (초) |

**예시:**

```python
# Default (auto-start backend, auto-detect path on Windows and Linux)
chloros = ChlorosLocal()

# Connect to running backend
chloros = ChlorosLocal(auto_start_backend=False)

# Custom backend path (Windows)
chloros = ChlorosLocal(backend_exe="C:/Custom/chloros-backend.exe")

# Custom backend path (Linux)
chloros = ChlorosLocal(backend_exe="/opt/mapir/chloros/backend/chloros-backend")

# Custom timeout with longer startup (e.g., for Jetson)
chloros = ChlorosLocal(timeout=60, backend_startup_timeout=120)
```

{% hint style="info" %}
**크로스 플랫폼 자동 감지**: SDK는 사용자의 플랫폼에 맞는 올바른 백엔드 경로를 자동으로 시도합니다:
* **Windows**: `C:\Program Files\MAPIR\Chloros\resources\backend\chloros-backend.exe`
* **Linux (.deb)**: `/usr/lib/chloros/chloros-backend`
* **Linux (수동)**: `/opt/mapir/chloros/backend/chloros-backend`
{% endhint %}

***

### 메서드

#### `create_project(project_name, camera=None)`

새로운 Chloros 프로젝트를 생성합니다.

**매개변수:**

| 매개변수      | 유형 | 필수 | 설명                                              |
| -------------- | ---- | -------- | -------------------------------------------------------- |
| `project_name` | str  | 예      | 프로젝트 이름                                     |
| `camera`       | str  | 아니요      | 카메라 템플릿 (예: &quot;Survey3N\_RGN&quot;, &quot;Survey3W\_OCN&quot;) |

**반환값:** `dict` - 프로젝트 생성 응답**예시:**

```python
# Basic project
chloros.create_project("DroneField_A")

# With camera template
chloros.create_project("DroneField_A", camera="Survey3N_RGN")
```

***

#### `import_images(folder_path, recursive=False)`

폴더에서 이미지 가져오기.

**매개변수:**

| 매개변수     | 유형     | 필수 | 설명                        |
| ------------- | -------- | -------- | ---------------------------------- |
| `folder_path` | str/Path | 예      | 이미지가 있는 폴더 경로         |
| `recursive`   | bool     | 아니요      | 하위 폴더 검색 (기본값: False) |

**반환값:** `dict` - 파일 수와 함께 가져오기 결과**예시:**

```python
# Import from folder
chloros.import_images("C:\\DroneImages\\Flight001")

# Import recursively
chloros.import_images("C:\\DroneImages", recursive=True)
```

***

#### `configure(**settings)`

처리 설정을 구성합니다.

**매개변수:**

| 매개변수                 | 유형 | 기본값                 | 설명                     |
| ------------------------- | ---- | ----------------------- | ------------------------------- |
| `debayer`                 | str  | &quot;표준 (빠름, 중간 품질)&quot; | 디베이어 방법            |
| `vignette_correction`     | bool | `True`                  | 비네팅 보정 활성화      |
| `reflectance_calibration` | bool | `True`                  | 반사율 보정 활성화  |
| `indices`                 | list | `None`                  | 계산할 식생 지수 |
| `export_format`           | str  | &quot;TIFF (16-bit)&quot;         | 출력 형식                   |
| `ppk`                     | bool | `False`                 | PPK 보정 활성화          |
| `custom_settings`         | dict | `None`                  | 고급 사용자 지정 설정        |

**내보내기 형식:**

* `"TIFF (16-bit)"` - GIS/사진측량에 권장
* `"TIFF (32-bit, Percent)"` - 과학적 분석
* `"PNG (8-bit)"` - 육안 검사
* `"JPG (8-bit)"` - 압축 출력

**사용 가능한 인덱스:**NDVI, NDRE, GNDVI, OSAVI, CIG, EVI, SAVI, MSAVI, MTVI2 등.**예시:**

```python
# Basic configuration
chloros.configure(
    vignette_correction=True,
    reflectance_calibration=True,
    indices=["NDVI", "NDRE"]
)

# Advanced configuration
chloros.configure(
    debayer="Standard (Fast, Medium Quality)",
    vignette_correction=True,
    reflectance_calibration=True,
    ppk=True,
    export_format="TIFF (32-bit, Percent)",
    indices=["NDVI", "NDRE", "GNDVI", "OSAVI", "CIG"]
)
```

***

#### `process(mode="parallel", wait=True, progress_callback=None)`

프로젝트 이미지를 처리합니다.

**매개변수:**

| 매개변수           | 유형     | 기본값      | 설명                               |
| ------------------- | -------- | ------------ | ----------------------------------------- |
| `mode`              | str      | `"parallel"` | 처리 모드: &quot;parallel&quot; 또는 &quot;serial&quot;   |
| `wait`              | bool     | `True`       | 완료 대기                       |
| `progress_callback` | 호출 가능 | `None`       | 진행 상황 콜백 함수(progress, msg) |
| `poll_interval`     | float    | `2.0`        | 진행 상황 폴링 간격 (초)   |

**반환값:** `dict` - 처리 결과

{% hint style="warning" %}
**병렬 모드**: Chloros+ 라이선스가 필요합니다. CPU 코어 수에 따라 자동으로 확장됩니다(최대 16개의 워커).
{% endhint %}

**예시:**

```python
# Simple processing
results = chloros.process()

# With progress monitoring
def show_progress(progress, message):
    print(f"[{progress}%] {message}")

chloros.process(
    mode="parallel",
    progress_callback=show_progress,
    wait=True
)

# Fire-and-forget (non-blocking)
chloros.process(wait=False)
```

***

#### `get_config()`

현재 프로젝트 구성을 가져옵니다.

**반환값:** `dict` - 현재 프로젝트 구성**예시:**

```python
config = chloros.get_config()
print(config['Project Settings'])
```

***

#### `get_status()`

스레드별 처리 진행 상황을 포함한 백엔드 상태 정보를 가져옵니다.

**반환값:** `dict` - 다음 구조를 가진 백엔드 상태:

```python
{
    "running": True,
    "url": "http://localhost:5000",
    "processing": {
        "percent": 75.0,
        "phase": "processing"
    },
    "export": {
        "percent": 50.0,
        "phase": "exporting",
        "active": True
    }
}
```

**예시:**

```python
status = chloros.get_status()
print(f"Running: {status['running']}")
print(f"URL: {status['url']}")
print(f"Processing: {status['processing']['percent']}%")
print(f"Export: {status['export']['percent']}% - Active: {status['export']['active']}")
```

***

#### `shutdown_backend()`

백엔드를 종료합니다(SDK로 시작된 경우).

**예시:**

```python
chloros.shutdown_backend()
```

***

#### `logout()`

로컬 시스템에서 캐시된 자격 증명을 지웁니다.

**설명:**

캐시된 인증 자격 증명을 제거하여 프로그래밍 방식으로 로그아웃합니다. 이는 다음 용도로 유용합니다:
* 서로 다른 Chloros+ 계정 간 전환
* 자동화 환경에서 자격 증명 지우기
* 보안 목적 (예: 제거하기 전에 자격 증명 제거)

**반환값:** `dict` - 로그아웃 작업 결과**예시:**

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK
chloros = ChlorosLocal()

# Clear cached credentials
result = chloros.logout()
print(f"Logout successful: {result}")

# After logout, login required via GUI/CLI/Browser before next SDK use
```

{% hint style="info" %}
**재인증 필요**: `logout()` 호출 후에는 Chloros, Chloros (브라우저), 또는 Chloros CLI를 통해 다시 로그인해야 합니다.
{% endhint %}

***

### 편의 함수

#### `process_folder(folder_path, **options)`

폴더를 처리하는 한 줄짜리 편의 함수입니다.

**매개변수:**

| 매개변수                 | 유형     | 기본값         | 설명                    |
| ------------------------- | -------- | --------------- | ------------------------------ |
| `folder_path`             | str/Path | 필수        | 이미지가 포함된 폴더 경로     |
| `project_name`            | str      | 자동 생성  | 프로젝트 이름                   |
| `camera`                  | str      | `None`          | 카메라 템플릿                |
| `indices`                 | list     | `["NDVI"]`      | 계산할 인덱스           |
| `vignette_correction`     | bool     | `True`          | 비네팅 보정 활성화     |
| `reflectance_calibration` | bool     | `True`          | 반사율 보정 활성화 |
| `export_format`           | str      | &quot;TIFF (16-bit)&quot; | 출력 형식                  |
| `mode`                    | 문자열      | `"parallel"`    | 처리 모드                |
| `progress_callback`       | 호출 가능 | `None`          | 진행 상황 콜백              |

**반환값:** `dict` - 처리 결과**예시:**

```python
from chloros_sdk import process_folder

# Simple one-liner
results = process_folder("C:\\DroneImages\\Flight001")

# With custom settings
results = process_folder(
    "C:\\DroneImages\\Flight001",
    project_name="Field_A_Survey",
    camera="Survey3N_RGN",
    indices=["NDVI", "NDRE", "GNDVI"],
    mode="parallel"
)

# With progress monitoring
def show_progress(progress, message):
    print(f"[{progress}%] {message}")

results = process_folder(
    "C:\\DroneImages\\Flight001",
    progress_callback=show_progress
)
```

***

## 컨텍스트 매니저 지원

SDK는 자동 정리를 위한 컨텍스트 매니저를 지원합니다:

```python
from chloros_sdk import ChlorosLocal

# Auto-cleanup when done
with ChlorosLocal() as chloros:
    chloros.create_project("MyProject")
    chloros.import_images("C:\\Images")
    chloros.configure(indices=["NDVI"])
    chloros.process()
# Backend automatically shut down here
```

***

## 전체 예제

{% hint style="info" %}
**Linux 사용자**: 아래의 모든 예제는 Windows 경로를 사용합니다. `C:\\...` 경로를 사용자의 Linux 경로(예: `/home/user/...` 또는 `~/...`)로 대체하십시오. 모든 SDK 기능은 플랫폼 간에 동일합니다.
{% endhint %}

### 예제 1: 기본 처리

기본 설정으로 폴더 처리:

```python
from chloros_sdk import process_folder

# Process with default settings
results = process_folder("C:\\Datasets\\Field_A_2025_01_15")

print(f"Processing complete: {results}")
```

***

### 예제 2: 사용자 지정 워크플로

처리 파이프라인에 대한 완전한 제어:

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK
chloros = ChlorosLocal()

# Create project with camera template
chloros.create_project("Research_Plot_A", camera="Survey3N_RGN")

# Import images
import_results = chloros.import_images("C:\\Research\\PlotA")
print(f"Imported {len(import_results.get('files', []))} images")

# Configure advanced settings
chloros.configure(
    debayer="Standard (Fast, Medium Quality)",
    vignette_correction=True,
    reflectance_calibration=True,
    ppk=False,
    export_format="TIFF (16-bit)",
    indices=["NDVI", "NDRE", "GNDVI", "OSAVI"]
)

# Process with progress monitoring
def show_progress(progress, message):
    print(f"Progress: {progress}% - {message}")

chloros.process(
    mode="parallel",
    progress_callback=show_progress,
    wait=True
)

print("Processing complete!")
```

***

### 예제 3: 여러 폴더 일괄 처리

여러 비행 데이터셋 처리:

```python
from chloros_sdk import ChlorosLocal
from pathlib import Path

# Initialize SDK once
chloros = ChlorosLocal()

# List of flight folders
flights = [
    "C:\\Datasets\\Flight_001",
    "C:\\Datasets\\Flight_002",
    "C:\\Datasets\\Flight_003"
]

for flight_path in flights:
    flight_name = Path(flight_path).name
    print(f"\n{'='*60}")
    print(f"Processing: {flight_name}")
    print('='*60)
    
    try:
        # Create project
        chloros.create_project(flight_name, camera="Survey3N_RGN")
        
        # Import images
        chloros.import_images(flight_path)
        
        # Configure
        chloros.configure(
            vignette_correction=True,
            reflectance_calibration=True,
            indices=["NDVI", "NDRE", "GNDVI"]
        )
        
        # Process
        chloros.process(mode="parallel", wait=True)
        
        print(f"✓ {flight_name} completed successfully")
    
    except Exception as e:
        print(f"✗ {flight_name} failed: {e}")

print("\n" + "="*60)
print("All flights processed!")
```

***

### 예제 4: 연구 파이프라인 통합

Chloros를 데이터 분석과 통합:

```python
from chloros_sdk import ChlorosLocal
import pandas as pd
import matplotlib.pyplot as plt

# Initialize Chloros
chloros = ChlorosLocal()

# Field survey data
surveys = [
    {"name": "Plot_A", "folder": "C:\\Research\\PlotA", "biomass": 4500},
    {"name": "Plot_B", "folder": "C:\\Research\\PlotB", "biomass": 3800},
    {"name": "Plot_C", "folder": "C:\\Research\\PlotC", "biomass": 5200}
]

results = []

for survey in surveys:
    # Process with Chloros
    chloros.create_project(survey['name'])
    chloros.import_images(survey['folder'])
    chloros.configure(indices=["NDVI", "NDRE"])
    chloros.process(mode="parallel", wait=True)
    
    # Get results
    config = chloros.get_config()
    
    # Extract NDVI values (example - adjust based on your needs)
    # In real implementation, you would read the processed TIFF files
    
    results.append({
        'plot': survey['name'],
        'biomass': survey['biomass'],
        # Add your NDVI extraction here
    })

# Statistical analysis
df = pd.DataFrame(results)
print("\nResults:")
print(df)

# Create correlation plot
# plt.scatter(df['ndvi'], df['biomass'])
# plt.xlabel('NDVI')
# plt.ylabel('Biomass (kg/ha)')
# plt.title('NDVI vs Biomass Correlation')
# plt.show()
```

***

### 예제 5: 사용자 지정 진행 상황 모니터링

로깅을 통한 고급 진행 상황 추적:

```python
from chloros_sdk import ChlorosLocal
from datetime import datetime
import logging

# Setup logging
logging.basicConfig(
    filename=f'processing_{datetime.now():%Y%m%d_%H%M%S}.log',
    level=logging.INFO,
    format='%(asctime)s - %(message)s'
)

# Progress callback with logging
def log_progress(progress, message):
    log_msg = f"[{progress}%] {message}"
    logging.info(log_msg)
    print(log_msg)

# Process with logging
chloros = ChlorosLocal()
chloros.create_project("LoggedProcess")
chloros.import_images("C:\\DroneImages")
chloros.configure(indices=["NDVI", "NDRE"])

logging.info("Starting processing...")
chloros.process(
    mode="parallel",
    progress_callback=log_progress,
    wait=True
)
logging.info("Processing complete!")
```

***

### 예제 6: 오류 처리

실전 환경을 위한 강력한 오류 처리:

```python
from chloros_sdk import ChlorosLocal
from chloros_sdk.exceptions import (
    ChlorosError,
    ChlorosBackendError,
    ChlorosLicenseError,
    ChlorosProcessingError
)

def process_safely(folder_path):
    """Process with comprehensive error handling"""
    try:
        with ChlorosLocal() as chloros:
            chloros.create_project("SafeProcess")
            chloros.import_images(folder_path)
            chloros.configure(indices=["NDVI"])
            chloros.process()
            
        return True, "Success"
    
    except ChlorosLicenseError as e:
        return False, f"License error: {e}. Upgrade to Chloros+ at cloud.mapir.camera/pricing"
    
    except ChlorosBackendError as e:
        return False, f"Backend error: {e}. Ensure Chloros is installed (Windows installer or Linux .deb package)."
    
    except ChlorosProcessingError as e:
        return False, f"Processing error: {e}"
    
    except FileNotFoundError as e:
        return False, f"Folder not found: {e}"
    
    except ChlorosError as e:
        return False, f"Chloros error: {e}"
    
    except Exception as e:
        return False, f"Unexpected error: {e}"

# Use the safe function
success, message = process_safely("C:\\DroneImages\\Flight001")
if success:
    print(f"✓ {message}")
else:
    print(f"✗ {message}")
```

***

### 예제 7: 계정 관리 및 로그아웃

프로그래밍 방식으로 자격 증명 관리:

```python
from chloros_sdk import ChlorosLocal

def switch_account():
    """Clear credentials to switch to a different account"""
    try:
        chloros = ChlorosLocal()
        
        # Clear current credentials
        result = chloros.logout()
        print("✓ Credentials cleared successfully")
        print("Please log in with new account via Chloros, Chloros (Browser), or CLI")
        
        return True
    
    except Exception as e:
        print(f"✗ Logout failed: {e}")
        return False

def secure_cleanup():
    """Remove credentials for security purposes"""
    try:
        chloros = ChlorosLocal()
        chloros.logout()
        print("✓ Credentials removed for security")
        
    except Exception as e:
        print(f"Warning: Cleanup error: {e}")

# Switch accounts
if switch_account():
    print("\nRe-authenticate via Chloros GUI/CLI/Browser before next SDK use")

# Or perform secure cleanup
# secure_cleanup()
```

***

### 예제 8: 명령줄 도구

SDK를 사용하여 사용자 지정 CLI 도구 구축:

```python
#!/usr/bin/env python
"""
Custom Chloros CLI Tool
Process multiple folders from command line
"""

import sys
import argparse
from pathlib import Path
from chloros_sdk import process_folder

def main():
    parser = argparse.ArgumentParser(description='Custom Chloros Processor')
    parser.add_argument('folders', nargs='+', help='Folders to process')
    parser.add_argument('--indices', nargs='+', default=['NDVI'],
                       help='Indices to calculate (default: NDVI)')
    parser.add_argument('--camera', default=None,
                       help='Camera template')
    parser.add_argument('--format', default='TIFF (16-bit)',
                       help='Export format')
    parser.add_argument('--logout', action='store_true',
                       help='Clear cached credentials before processing')
    
    args = parser.parse_args()
    
    # Handle logout if requested
    if args.logout:
        from chloros_sdk import ChlorosLocal
        chloros = ChlorosLocal()
        chloros.logout()
        print("Credentials cleared. Please re-login via Chloros GUI/CLI/Browser.")
        return 0
    
    successful = []
    failed = []
    
    for folder in args.folders:
        folder_path = Path(folder)
        
        if not folder_path.exists():
            print(f"✗ Skipping {folder}: not found")
            failed.append(folder)
            continue
        
        print(f"\nProcessing: {folder_path.name}...")
        
        try:
            process_folder(
                folder_path,
                camera=args.camera,
                indices=args.indices,
                export_format=args.format
            )
            print(f"✓ {folder_path.name} complete")
            successful.append(folder)
        
        except Exception as e:
            print(f"✗ {folder_path.name} failed: {e}")
            failed.append(folder)
    
    # Summary
    print(f"\n{'='*60}")
    print(f"Summary: {len(successful)} successful, {len(failed)} failed")
    
    return 0 if not failed else 1

if __name__ == '__main__':
    sys.exit(main())
```

**사용법:**

```bash
# Process multiple folders
python my_processor.py "C:\Flight001" "C:\Flight002" --indices NDVI NDRE GNDVI

# Clear cached credentials
python my_processor.py --logout
```

***

## 예외 처리

SDK는 다양한 오류 유형에 대한 특정 예외 클래스를 제공합니다:

### 예외 계층 구조

```python
ChlorosError                    # Base exception
├── ChlorosBackendError        # Backend startup/connection issues
├── ChlorosLicenseError        # License validation issues
├── ChlorosConnectionError     # Network/connection failures
├── ChlorosProcessingError     # Image processing failures
├── ChlorosAuthenticationError # Authentication failures
└── ChlorosConfigurationError  # Configuration errors
```

### 예외 예시

```python
from chloros_sdk import ChlorosLocal
from chloros_sdk.exceptions import *

try:
    chloros = ChlorosLocal()
    chloros.process()

except ChlorosLicenseError:
    print("Chloros+ license required. Upgrade at cloud.mapir.camera/pricing")

except ChlorosBackendError:
    print("Backend failed to start. Ensure Chloros is installed (Windows installer or Linux .deb package).")

except ChlorosProcessingError as e:
    print(f"Processing failed: {e}")

except ChlorosError as e:
    print(f"General Chloros error: {e}")
```

***

## 고급 주제

### 사용자 정의 백엔드 구성

사용자 정의 백엔드 위치 또는 구성을 사용하려면:

```python
chloros = ChlorosLocal(
    backend_exe="C:\\Custom\\chloros-backend.exe",
    api_url="http://localhost:5001",  # Custom port
    timeout=60,                        # Longer timeout
    backend_startup_timeout=120        # 2 minutes startup
)
```

### 비차단 처리

처리를 시작하고 다른 작업을 계속하려면:

```python
# Start processing (non-blocking)
chloros.process(wait=False)

# Do other work here...
print("Processing started in background...")

# Check status later
import time
while True:
    status = chloros.get_config()
    if status.get('processing_complete'):
        break
    time.sleep(5)

print("Processing complete!")
```

### 메모리 관리

대용량 데이터 세트의 경우, 일괄 처리:

```python
from pathlib import Path

base_folder = Path("C:\\LargeDataset")
batch_size = 100

# Get all image files
images = list(base_folder.glob("*.RAW"))

# Process in batches
for i in range(0, len(images), batch_size):
    batch = images[i:i+batch_size]
    batch_folder = base_folder / f"batch_{i//batch_size}"
    
    # Create batch folder and move images
    # ... (implementation details)
    
    # Process batch
    process_folder(batch_folder)
```

***

## 문제 해결

### 백엔드 시작 실패

**문제:** SDK에서 백엔드 시작 실패**해결 방법:**

1. Chloros가 설치되어 있는지 확인:

```python
import os
import platform

# Auto-detect backend path
if platform.system() == "Windows":
    backend_path = r"C:\Program Files\MAPIR\Chloros\resources\backend\chloros-backend.exe"
else:
    backend_path = "/usr/lib/chloros/chloros-backend"

print(f"Backend exists: {os.path.exists(backend_path)}")
```

2. 방화벽(Windows) 또는 포트 사용 가능 여부(Linux: `lsof -i :5000`)를 확인하십시오.
3. 수동 백엔드 경로를 시도해 보십시오:

```python
# Windows
chloros = ChlorosLocal(backend_exe="C:\\Path\\To\\chloros-backend.exe")

# Linux
chloros = ChlorosLocal(backend_exe="/opt/mapir/chloros/backend/chloros-backend")
```

***

### 라이선스 미검출**문제:** SDK에서 라이선스 누락에 대한 경고가 표시됨**해결 방법:**

1. Chloros, Chloros (브라우저) 또는 Chloros, CLI를 열고 로그인하십시오.
2. 라이선스가 캐시에 저장되어 있는지 확인하십시오:

```python
from pathlib import Path
import os
import platform

# Check cache location
if platform.system() == "Windows":
    cache_path = Path(os.getenv('APPDATA')) / 'Chloros' / 'cache'
else:
    cache_path = Path.home() / '.cache' / 'chloros'

print(f"Cache exists: {cache_path.exists()}")
```

3. 인증 정보 문제가 발생하는 경우, 캐시된 인증 정보를 삭제하고 다시 로그인하십시오:

```python
from chloros_sdk import ChlorosLocal

# Clear cached credentials
chloros = ChlorosLocal()
chloros.logout()

# Then login again via Chloros, Chloros (Browser), or Chloros CLI
```

4. 지원팀에 문의하십시오: info@mapir.camera

***

### 가져오기 오류**문제:** `ModuleNotFoundError: No module named 'chloros_sdk'`**해결 방법:**

```bash
# Verify installation
pip show chloros-sdk

# Reinstall if needed
pip uninstall chloros-sdk
pip install chloros-sdk

# Check Python environment
python -c "import sys; print(sys.path)"
```

***

### 처리 시간 초과**문제:** 처리 시간 초과**해결 방법:**

1. 시간 초과 값을 늘리십시오:

```python
chloros = ChlorosLocal(timeout=120)  # 2 minutes
```

2. 더 작은 배치로 처리하십시오
3. 사용 가능한 디스크 공간을 확인하십시오
4. 시스템 리소스를 모니터링하십시오

***

### 포트가 이미 사용 중**문제:** 백엔드 포트 5000이 점유됨**해결 방법:**

```python
# Use different port
chloros = ChlorosLocal(api_url="http://localhost:5001")
```

또는 충돌하는 프로세스를 찾아 종료하십시오:

```powershell
# Windows PowerShell
Get-NetTCPConnection -LocalPort 5000
```

```bash
# Linux
lsof -i :5000
kill $(lsof -t -i :5000)
```

***

## 성능 팁

### 처리 속도 최적화

1. **병렬 모드 사용** (Chloros+ 필요)

```python
chloros.process(mode="parallel")  # Up to 16 workers
```

2. **출력 해상도 낮추기** (허용 가능한 경우)

```python
chloros.configure(export_format="PNG (8-bit)")  # Faster than TIFF
```

3. **불필요한 인덱스 비활성화**

```python
# Only calculate needed indices
chloros.configure(indices=["NDVI"])  # Not all indices
```

4. **SSD에서 처리** (HDD가 아님)***

### 메모리 최적화

대용량 데이터 세트의 경우:

```python
# Process in batches instead of all at once
# See "Memory Management" in Advanced Topics
```

***

### 백그라운드 처리

다른 작업을 위해 Python를 확보하세요:

```python
chloros.process(wait=False)  # Non-blocking

# Continue with other work
# ...
```

***

## 통합 예시

### Django 통합

```python
# views.py
from django.http import JsonResponse
from chloros_sdk import process_folder

def process_images_view(request):
    if request.method == 'POST':
        folder_path = request.POST.get('folder_path')
        
        try:
            results = process_folder(folder_path)
            return JsonResponse({'success': True, 'results': results})
        except Exception as e:
            return JsonResponse({'success': False, 'error': str(e)})
```

### Flask API

```python
# app.py
from flask import Flask, request, jsonify
from chloros_sdk import process_folder

app = Flask(__name__)

@app.route('/api/process', methods=['POST'])
def process():
    data = request.get_json()
    folder_path = data.get('folder_path')
    
    try:
        results = process_folder(folder_path)
        return jsonify({'success': True, 'results': results})
    except Exception as e:
        return jsonify({'success': False, 'error': str(e)}), 500

if __name__ == '__main__':
    app.run()
```

### Jupyter Notebook

```python
# notebook.ipynb
from chloros_sdk import ChlorosLocal
import matplotlib.pyplot as plt

# Initialize
chloros = ChlorosLocal()

# Process
chloros.create_project("JupyterTest")
chloros.import_images("C:\\Data")
chloros.configure(indices=["NDVI"])

# Progress in notebook
from IPython.display import clear_output

def notebook_progress(progress, message):
    clear_output(wait=True)
    print(f"Progress: {progress}%")
    print(message)

chloros.process(progress_callback=notebook_progress)

# Visualize results
# ... (your visualization code)
```

***

## FAQ

### Q: SDK를 사용하려면 인터넷 연결이 필요한가요?

**A:** 초기 라이선스 활성화 시에만 필요합니다. Chloros, Chloros(브라우저) 또는 Chloros CLI를 통해 로그인하면 라이선스가 로컬에 캐시되어 30일 동안 오프라인에서도 작동합니다.***

### Q: GUI가 없는 서버에서 SDK를 사용할 수 있나요?**A:** 네! SDK는 Windows 및 Linux 서버 모두에서 헤드리스 모드로 작동합니다.**Linux (헤드리스 모드 권장):**
* `.deb` 패키지를 통해 설치
* 라이선스 활성화: `chloros-cli login user@example.com 'password'`

**Windows 서버:**
* Windows Server 2016 이상
* Chloros 설치 (1회)
* CLI를 통해 또는 임의의 컴퓨터에서 라이선스 활성화

***

### Q: Desktop, CLI 및 SDK의 차이점은 무엇입니까?

| 기능         | 데스크톱 GUI | CLI 명령줄 | Python SDK  |
| --------------- | ----------- | ---------------- | ----------- |
| **인터페이스**   | 포인트 앤 클릭 | 명령어          | Python API  |
| **최적 용도**    | 시각적 작업 | 스크립팅        | 통합          |
| **자동화**  | 제한적     | 양호             | 우수          |
| **유연성** | 기본       | 양호             | 최대         |
| **라이선스**     | Chloros+    | Chloros+         | Chloros+    |***

### Q: SDK로 제작된 앱을 배포할 수 있나요?**A:** SDK 코드를 애플리케이션에 통합할 수 있지만, 다음 조건이 적용됩니다:

* 최종 사용자에게는 Chloros가 설치되어 있어야 합니다
* 최종 사용자에게는 유효한 Chloros+ 라이선스가 필요합니다
* 상업적 배포에는 OEM 라이선스가 필요합니다

OEM 관련 문의는 info@mapir.camera로 연락해 주십시오.

***

### Q: SDK는 어떻게 업데이트하나요?

```bash
pip install --upgrade chloros-sdk
```

***

### Q: 처리된 이미지는 어디에 저장되나요?

기본적으로 프로젝트 경로에 저장됩니다:

```

Project_Path/
└── MyProject/
    └── Survey3N_RGN/          # Processed outputs
```

***

### Q: 예약된 Python 스크립트를 통해 이미지를 처리할 수 있나요?**A:** 네! Python 스크립트와 함께 OS 스케줄러를 사용하십시오:

```python
# scheduled_processing.py
from chloros_sdk import process_folder

# Process today's flights
results = process_folder("/data/flights/today")  # Linux
# results = process_folder("C:\\Flights\\Today")  # Windows
```

**Windows:** 작업 스케줄러를 통해 매일 실행되도록 예약하세요.**Linux:** cron을 통해 예약하세요:

```cron
# Run at 2 AM daily
0 2 * ** /usr/bin/python3 /home/user/scheduled_processing.py >> /var/log/chloros.log 2>&1
```

***

### Q: SDK는 async/await를 지원하나요?**A:** 현재 버전은 동기식입니다. 비동기 동작을 원하시면 `wait=False`를 사용하거나 별도의 스레드에서 실행하십시오:

```python
import threading

def process_thread():
    chloros.process()

thread = threading.Thread(target=process_thread)
thread.start()

# Continue with other work...
```

***

### Q: 서로 다른 Chloros+ 계정 간에 어떻게 전환하나요?**A:** `logout()` 메서드를 사용하여 캐시된 자격 증명을 지운 다음, 새 계정으로 다시 로그인하십시오:

```python
from chloros_sdk import ChlorosLocal

# Clear current credentials
chloros = ChlorosLocal()
chloros.logout()

# Re-login via Chloros, Chloros (Browser), or Chloros CLI with new account
```

로그아웃 후, CLI를 통해 GUI, 브라우저 또는 SDK로 새 계정에 인증한 다음 다시 사용하십시오.

***

## 도움말 받기

### 문서

* **API 참조**: 이 페이지

### 지원 채널

* **이메일**: info@mapir.camera
* **웹사이트**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* **가격**: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

### 샘플 코드

여기에 나열된 모든 예제는 테스트를 거쳤으며 실제 환경에서 바로 사용할 수 있습니다. 사용 사례에 맞게 복사하여 수정해 사용하십시오.

***

## 라이선스**독점 소프트웨어** - Copyright (c) 2025 MAPIR Inc.

SDK를 사용하려면 유효한 Chloros+ 구독이 필요합니다. 무단 사용, 배포 또는 수정은 금지됩니다.
