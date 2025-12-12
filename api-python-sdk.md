# API : Python SDK

**Chloros Python SDK** 는 Chloros 이미지 처리 엔진에 대한 프로그래밍 방식 접근을 제공하여 자동화, 맞춤형 워크플로우 및 Python 애플리케이션 및 연구 파이프라인과의 원활한 통합을 가능하게 합니다.

### 주요 기능

* 🐍 **네이티브 Python** - 깔끔하고 파이썬적인 이미지 처리 API
* 🔧 **완전한 API 접근** - Chloros 처리에 대한 완벽한 제어
* 🚀 **자동화** - 맞춤형 배치 처리 워크플로 구축
* 🔗 **통합** - 기존 애플리케이션에 Chloros 임베드
* 📊 **연구용 준비 완료** - 과학적 분석 파이프라인에 최적화
* ⚡ **병렬 처리** - CPU 코어 수에 따라 확장 가능 (Chloros+)

### 요구 사항

| 요구 사항          | 세부 사항                                                             |
| -------------------- | ------------------------------------------------------------------- |
| **Chloros 데스크톱**  | 로컬에 설치되어야 함                                           |
| **라이선스**          | Chloros+ ([유료 플랜 필요](https://cloud.mapir.camera/pricing)) |
| **운영 체제** | Windows 10/11 (64비트)                                              |
| **XPROTX**           | XPROTX 3.7 이상                                                |
| **메모리**           | 최소 8GB RAM (권장 16GB)                                  |
| **인터넷**         | 라이선스 활성화에 필요                                     |

{% hint style=&quot;warning&quot; %}
**라이선스 요구 사항**: Python SDK는 API 접근을 위해 유료 Chloros+ 구독이 필요합니다. 표준(무료) 플랜은 API/SDK 접근 권한이 없습니다. [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)를 방문하여 업그레이드하세요.
{% endhint %}

## 빠른 시작

### 설치

pip를 통해 설치:

```bash
pip install chloros-sdk
```

{% hint style=&quot;info&quot; %}
**초기 설정**: SDK 사용 전, Chloros+ 라이선스를 활성화하세요. Chloros, Chloros (브라우저) 또는 Chloros CLI를 열고 자격 증명으로 로그인하여 Chloros+ 라이선스를 활성화하십시오. 이 작업은 한 번만 수행하면 됩니다.
{% endhint %}

### 기본 사용법

몇 줄로 폴더 처리하기:

```python
from chloros_sdk import process_folder

# One-line processing
results = process_folder("C:\\DroneImages\\Flight001")
```

### 전체 제어

고급 워크플로우를 위해:

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK
chloros = ChlorosLocal()

# Create project
chloros.create_project("MyProject", camera="Survey3N_RGN")

# Import images
chloros.import_images("C:\\DroneImages\\Flight001")

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

SDK 설치 전 다음을 확인하십시오:

1. **Chloros 데스크톱** 설치 ([다운로드](download.md))
2. **Python 3.7 이상** 설치 ([python.org](https://www.python.org))
3. **활성 Chloros+ 라이선스** ([업그레이드](https://cloud.mapir.camera/pricing))

### pip를 통한 설치

**표준 설치:**

```bash
pip install chloros-sdk
```

**진행 상황 모니터링 지원 포함:**

```bash
pip install chloros-sdk[progress]
```

**개발 설치:**

```bash
pip install chloros-sdk[dev]
```

### 설치 확인

SDK가 올바르게 설치되었는지 테스트:

```python
import chloros_sdk
print(f"Chloros SDK version: {chloros_sdk.__version__}")
```

***

## 초기 설정

### 라이선스 활성화

SDK는 Chloros, Chloros(브라우저), Chloros 및 CLI와 동일한 라이선스를 사용합니다. GUI 또는 CLI를 통해 한 번만 활성화하십시오:

1. **Chloros 또는 Chloros(브라우저)**를 열고 사용자 탭에서 로그인하십시오. <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> 탭에서 로그인합니다. 또는 **CLI**를 엽니다.
2. Chloros+ 자격 증명을 입력하고 로그인합니다.
3. 라이선스는 로컬에 캐시됩니다(재부팅 후에도 유지됨).

{% hint style=&quot;success&quot; %}
**일회성 설정**: GUI 또는 CLI를 통해 로그인한 후에는 SDK가 자동으로 캐시된 라이선스를 사용합니다. 추가 인증이 필요 없습니다!
{% endhint %}

### 연결 테스트

SDK가 Chloros에 연결되는지 확인하십시오:

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
| `auto_start_backend`      | bool | `True`                    | 필요 시 백엔드 자동 시작 |
| `backend_exe`             | str  | `None` (자동-detect)      | 백엔드 실행 파일 경로            |
| `timeout`                 | int  | `30`                      | 요청 시간 초과(초)            |
| `backend_startup_timeout` | int  | `60`                      | 백엔드 시작 시간 초과(초) (초) |

**예시:**

```python
# Default (auto-start backend)
chloros = ChlorosLocal()

# Connect to running backend
chloros = ChlorosLocal(auto_start_backend=False)

# Custom backend path
chloros = ChlorosLocal(backend_exe="C:/Custom/chloros-backend.exe")

# Custom timeout
chloros = ChlorosLocal(timeout=60)
```

***

### 메서드

#### `create_project(project_name, camera=None)`

새로운 Chloros 프로젝트 생성.

**매개변수:**

| 매개변수      | 유형 | 필수 | 설명                                              |
| -------------- | ---- | -------- | -------------------------------------------------------- |
| `project_name` | str  | Yes      | 프로젝트 이름                                     |
| `camera`       | str  | No       | 카메라 템플릿 (예: &quot;Survey3N\_RGN&quot;, &quot;Survey3W\_OCN&quot;) |

**반환값:** `dict` - 프로젝트 생성 응답

**예시:**

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
| `folder_path` | 문자열/경로 | 예      | 이미지 폴더 경로         |
| `recursive`   | 부울     | 아니오       | 하위 폴더 검색 (기본값: False) |

**반환값:** `dict` - 파일 수와 함께 가져오기 결과

**예시:**

```python
# Import from folder
chloros.import_images("C:\\DroneImages\\Flight001")

# Import recursively
chloros.import_images("C:\\DroneImages", recursive=True)
```

***

#### `configure(**settings)`

처리 설정 구성.

**매개변수:**

| 매개변수                 | 유형 | 기본값                 | 설명                     |
| ------------------------- | ---- | ----------------------- | ------------------------------- |
| `debayer`                 | 문자열 | &quot;고품질 (빠름)&quot; | 디베이어 방법                  |
| `vignette_correction`     | 부울 | `True`                  | 비네트 보정 활성화      |
| `reflectance_calibration` | bool | `True`                  | 반사율 보정 활성화      |
| `indices`                 | list | `None`                  | 계산할 식생 지수 |
| `export_format`           | 문자열 | &quot;TIFF (16비트)&quot;         | 출력 형식                   |
| `ppk`                     | 부울 | `False`                 | PPK 보정 활성화          |
| `custom_settings`         | dict | `None`                  | 고급 사용자 설정        |

**내보내기 형식:**

* `"TIFF (16-bit)"` - GIS/사진측량 권장
* `"TIFF (32-bit, Percent)"` - 과학적 분석
* `"PNG (8-bit)"` - 시각적 검사
* `"JPG (8-bit)"` - 압축 출력

**사용 가능한 인덱스:**

NDVI, NDRE, GNDVI, OSAVI, CIG, EVI, SAVI, MSAVI, MTVI2 등.

**예시:**

```python
# Basic configuration
chloros.configure(
    vignette_correction=True,
    reflectance_calibration=True,
    indices=["NDVI", "NDRE"]
)

# Advanced configuration
chloros.configure(
    debayer="High Quality (Faster)",
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
| `progress_callback` | callable | `None`       | 진행 상황 콜백 함수(progress, msg) |
| `poll_interval`     | float    | `2.0`        | 진행 상황 폴링 간격 (초)   |

**반환값:** `dict` - 처리 결과

{% hint style=&quot;warning&quot; %}
**병렬 모드**: Chloros+ 라이선스 필요. CPU 코어 수에 따라 자동 확장(최대 16개 작업자).
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

**반환값:** `dict` - 현재 프로젝트 구성

**예시:**

```python
config = chloros.get_config()
print(config['Project Settings'])
```

***

#### `get_status()`

백엔드 상태 정보를 가져옵니다.

**반환값:** `dict` - 백엔드 상태

**예시:**

```python
status = chloros.get_status()
print(f"Running: {status['running']}")
print(f"URL: {status['url']}")
```

***

#### `shutdown_backend()`

백엔드를 종료합니다 (SDK로 시작된 경우).

**예시:**

```python
chloros.shutdown_backend()
```

***

### 편의 함수

#### `process_folder(folder_path, **options)`

폴더를 처리하는 한 줄 편의 함수.

**매개변수:**

| 매개변수                 | 유형     | 기본값         | 설명                    |
| ------------------------- | -------- | --------------- | ------------------------------ |
| `folder_path`             | 문자열/경로 | 필수        | 이미지 포함 폴더 경로     |
| `project_name`            | 문자열      | 자동 생성    | 프로젝트 이름                   |
| `camera`                  | 문자열      | `None`          | 카메라 템플릿                |
| `indices`                 | 목록     | `["NDVI"]`      | 계산할 인덱스           |
| `vignette_correction`     | bool     | `True`          | 비네팅 보정 활성화     |
| `reflectance_calibration` | bool     | `True`          | 반사율 보정 활성화 |
| `export_format`           | str      | &quot;TIFF (16-bit)&quot; | 출력 형식                  |
| `mode`                    | 문자열      | `"parallel"`    | 처리 모드                |
| `progress_callback`       | 콜러블 | `None`          | 진행 상황 콜백              |

**반환값:** `dict` - 처리 결과

**예시:**

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

SDK는 자동 정리 기능을 위한 컨텍스트 매니저를 지원합니다:

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

## 완전한 예시

### 예시 1: 기본 처리

기본 설정으로 폴더 처리:

```python
from chloros_sdk import process_folder

# Process with default settings
results = process_folder("C:\\Datasets\\Field_A_2025_01_15")

print(f"Processing complete: {results}")
```

***

### 예시 2: 사용자 정의 워크플로

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
    debayer="High Quality (Faster)",
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

### 예제 3: 다중 폴더 일괄 처리

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

### 예시 5: 사용자 정의 진행 상황 모니터링

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

### 예시 6: 오류 처리

생산 환경용 강력한 오류 처리:

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
        return False, f"Backend error: {e}. Ensure Chloros Desktop is installed."
    
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

### 예시 7: 명령줄 도구

SDK를 사용한 맞춤형 CLI 도구 구축:

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
    
    args = parser.parse_args()
    
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
python my_processor.py "C:\Flight001" "C:\Flight002" --indices NDVI NDRE GNDVI
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
    print("Backend failed to start. Ensure Chloros Desktop is installed.")

except ChlorosProcessingError as e:
    print(f"Processing failed: {e}")

except ChlorosError as e:
    print(f"General Chloros error: {e}")
```

***

## 고급 주제

### 사용자 정의 백엔드 구성

사용자 지정 백엔드 위치 또는 구성을 사용합니다:

```python
chloros = ChlorosLocal(
    backend_exe="C:\\Custom\\chloros-backend.exe",
    api_url="http://localhost:5001",  # Custom port
    timeout=60,                        # Longer timeout
    backend_startup_timeout=120        # 2 minutes startup
)
```

### 비차단 처리

처리를 시작하고 다른 작업을 계속합니다:

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

대규모 데이터 세트의 경우 배치로 처리합니다:

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

**문제:** SDK 백엔드 시작 실패

**해결 방법:**

1. Chloros 데스크톱 설치 확인:

```python
import os
backend_path = r"C:\Program Files\MAPIR\Chloros\resources\backend\chloros-backend.exe"
print(f"Backend exists: {os.path.exists(backend_path)}")
```

2. Windows 방화벽이 차단하지 않는지 확인
3. 수동 백엔드 경로 시도:

```python
chloros = ChlorosLocal(backend_exe="C:\\Path\\To\\chloros-backend.exe")
```

***

### 라이선스 미검출

**문제:** SDK에서 라이선스 누락 경고

**해결 방법:**

1. Chloros, Chloros (브라우저) 또는 Chloros CLI를 열고 로그인하십시오.
2. 라이선스가 캐시되었는지 확인하십시오:

```python
from pathlib import Path
import os

# Check cache location (Windows)
cache_path = Path(os.getenv('APPDATA')) / 'Chloros' / 'cache'
print(f"Cache exists: {cache_path.exists()}")
```

3. 지원팀에 문의: info@mapir.camera

***

### 가져오기 오류

**문제:** `ModuleNotFoundError: No module named 'chloros_sdk'`

**해결 방법:**

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

### 처리 시간 초과

**문제:** 처리 시간 초과

**해결 방법:**

1. 시간 초과 값 증가:

```python
chloros = ChlorosLocal(timeout=120)  # 2 minutes
```

2. 더 작은 배치로 처리
3. 사용 가능한 디스크 공간 확인
4. 시스템 리소스 모니터링

***

### 포트 사용 중

**문제:** 백엔드 포트 5000 점유됨

**해결 방법:**

```python
# Use different port
chloros = ChlorosLocal(api_url="http://localhost:5001")
```

또는 충돌하는 프로세스를 찾아 종료:

```powershell
# PowerShell
Get-NetTCPConnection -LocalPort 5000
```

***

## 성능 팁

### 처리 속도 최적화

1. **병렬 모드 사용** (Chloros+ 필요)

```python
chloros.process(mode="parallel")  # Up to 16 workers
```

2. **출력 해상도 낮추기** (수용 가능한 경우)

```python
chloros.configure(export_format="PNG (8-bit)")  # Faster than TIFF
```

3. **불필요한 인덱스 비활성화**

```python
# Only calculate needed indices
chloros.configure(indices=["NDVI"])  # Not all indices
```

4. **SSD에서 처리** (HDD 아님)

***

### 메모리 최적화

대규모 데이터셋의 경우:

```python
# Process in batches instead of all at once
# See "Memory Management" in Advanced Topics
```

***

### 백그라운드 처리

Python를 다른 작업에 활용:

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

## 자주 묻는 질문

### Q: SDK 사용 시 인터넷 연결이 필요한가요?

**A:** 초기 라이선스 활성화 시에만 필요합니다. Chloros, Chloros(브라우저) 또는 Chloros CLI를 통해 로그인한 후에는 라이선스가 로컬에 캐시되어 30일 동안 오프라인에서도 작동합니다.

***

### Q: GUI가 없는 서버에서 SDK를 사용할 수 있나요?

**A:** 가능합니다! 요구 사항:

* Windows Server 2016 이상
* Chloros 설치 완료 (1회성)
* 라이선스 활성화된 기기 존재 (캐시된 라이선스 서버로 복사됨)

***

### Q: 데스크톱, CLI, SDK의 차이점은 무엇인가요?

| 기능         | 데스크톱 GUI | XPROTX 명령줄 인터페이스 | XPROTX XPROTX XPROTX  |
| --------------- | ----------- | ---------------- | ----------- |
| **인터페이스**   | 포인트 클릭 | 명령어          | Python API  |
| **최적 용도**    | 시각적 작업 | 스크립팅        | 통합         |
| **자동화**  | 제한적     | 양호             | 우수     |
| **유연성** | 기본적       | 양호             | 최대     |
| **라이선스**     | Chloros+    | Chloros+         | Chloros+    |

***

### Q: SDK로 빌드한 애플리케이션을 배포할 수 있나요?

**A:** SDK 코드는 애플리케이션에 통합할 수 있지만:

* 최종 사용자는 Chloros가 설치되어 있어야 합니다
* 최종 사용자는 유효한 Chloros+ 라이선스가 필요합니다
* 상업적 배포에는 OEM 라이선스가 필요합니다

OEM 관련 문의는 info@mapir.camera로 연락하십시오.

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

### Q: 예약 실행되는 Python 스크립트로 이미지를 처리할 수 있나요?

**A:** 네! Windows 작업 스케줄러와 Python 스크립트를 함께 사용하세요:

```python
# scheduled_processing.py
from chloros_sdk import process_folder

# Process today's flights
results = process_folder("C:\\Flights\\Today")
```

작업 스케줄러를 통해 매일 실행되도록 설정하세요.

***

### Q: SDK는 비동기/대기(async/await)를 지원하나요?

**A:** 현재 버전은 동기식입니다. 비동기 동작을 원하시면 `wait=False`를 사용하거나 별도 스레드에서 실행하세요:

```python
import threading

def process_thread():
    chloros.process()

thread = threading.Thread(target=process_thread)
thread.start()

# Continue with other work...
```

***

## 도움말 받기

### 문서

* **API 참조**: 본 페이지

### 지원 채널

* **이메일**: info@mapir.camera
* **웹사이트**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* **가격 정보**: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

### 샘플 코드

여기에 나열된 모든 예제는 테스트를 거쳐 실제 사용이 가능합니다. 사용 사례에 맞게 복사하여 수정하여 사용하십시오.

***

## 라이선스

**독점 소프트웨어** - 저작권 (c) 2025 MAPIR Inc.

SDK는 유효한 Chloros+ 구독이 필요합니다. 무단 사용, 배포 또는 수정은 금지됩니다.
