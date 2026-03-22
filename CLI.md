# CLI : 명령줄

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>**Chloros CLI**는 Chloros 이미지 처리 엔진에 대한 강력한 명령줄 액세스 기능을 제공하여, 이미징 워크플로우를 위한 자동화, 스크립팅 및 헤드리스 운영을 가능하게 합니다.

### 주요 기능

* 🚀 **자동화** - 여러 데이터 세트의 일괄 처리를 스크립트로 자동화
* 🔗 **통합** - 기존 워크플로우 및 파이프라인에 원활하게 통합
* 💻 **헤드리스 운영** - GUI 없이 실행
* 🌍 **다국어 지원** - 38개 언어 지원
* ⚡ **병렬 처리** - [동적 컴퓨팅 적응(Dynamic Compute Adaptation)](processing-architecture/dynamic-compute-adaptation.md)이 하드웨어에 맞게 자동으로 최적화합니다

### 요구 사항

| 요구 사항          | 세부 정보                                                             |
| -------------------- | ------------------------------------------------------------------- |
| **운영 체제** | Windows 10/11 (64비트), Linux x86_64 (amd64), Linux arm64 (NVIDIA Jetson JetPack 6) |
| **라이선스**          | Chloros+ ([유료 플랜 필요](https://cloud.mapir.camera/pricing)) |
| **메모리**           | 최소 8GB RAM (16GB 권장)                                  |
| **인터넷**         | 라이선스 활성화에 필요                                     |
| **디스크 공간**       | 프로젝트 크기에 따라 다름                                              |

{% hint style="warning" %}
**라이선스 요구 사항**: CLI를 사용하려면 유료 Chloros+ 구독이 필요합니다. 스탠다드(무료) 요금제에서는 CLI에 액세스할 수 없습니다. 업그레이드를 원하시면 [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)를 방문하십시오.
{% endhint %}

## 빠른 시작

### 설치

#### Windows

CLI는 Chloros 설치 프로그램에 자동으로 포함되어 있습니다:

1. **Chloros Installer.exe**를 다운로드하여 실행하십시오.
2. 설치 마법사를 완료하십시오.
3. CLI가 다음 위치에 설치되었습니다: `C:\Program Files\Chloros\resources\cli\chloros-cli.exe`

{% hint style="success" %}
설치 프로그램이 시스템 PATH에 `chloros-cli`를 자동으로 추가합니다. 설치 후 터미널을 재시작하십시오.
{% endhint %}

#### Linux

사용 중인 아키텍처에 맞는 `.deb` 패키지를 설치하십시오:

```bash
# Linux amd64
sudo dpkg -i chloros-amd64.deb

# Linux arm64 (NVIDIA Jetson, JetPack 6)
sudo dpkg -i chloros-arm64-jp6.deb
```

자세한 Linux 설정 방법은 [Linux 설치](linux/linux-installation.md)를 참조하십시오.

### 초기 설정

CLI를 사용하기 전에 Chloros+ 라이선스를 활성화하십시오:

**Windows:**

```powershell
# Login with your Chloros+ account
chloros-cli login user@example.com 'your_password'

# Check license status
chloros-cli status

# Process your first project
chloros-cli process "C:\Images\Dataset001"
```

**Linux:**

```bash
# Login with your Chloros+ account
chloros-cli login user@example.com 'your_password'

# Check license status
chloros-cli status

# Process your first project
chloros-cli process ~/images/dataset001
```

### 기본 사용법

기본 설정으로 폴더를 처리합니다:

**Windows:**

```powershell
chloros-cli process "C:\Images\Dataset001"
```

**Linux:**

```bash
chloros-cli process ~/images/dataset001
```

***

## 명령어 참조

### 일반 구문

```
chloros-cli [global-options] <command> [command-options]
```

***

## 명령어

### `process` - 이미지 처리

보정 기능을 사용하여 폴더 내의 이미지를 처리합니다.

**구문:**

```bash
chloros-cli process <input-folder> [options]
```

**예시:**

```bash
# Windows
chloros-cli process "C:\Datasets\Survey_001" --vignette --reflectance

# Linux
chloros-cli process ~/datasets/survey_001 --vignette --reflectance
```

#### 명령어 옵션

| 옵션                | 유형    | 기본값        | 설명                                                                            |
| --------------------- | ------- | -------------- | -------------------------------------------------------------------------------------- |
| `<input-folder>`      | 경로    | _필수_     | RAW/JPG 다중 스펙트럼 이미지가 포함된 폴더                                         |
| `-o, --output`        | 경로    | 입력과 동일  | 처리된 이미지의 출력 폴더                                                     |
| `-n, --project-name`  | 문자열  | 자동 생성 | 사용자 지정 프로젝트 이름                                                                    |
| `--vignette`          | 플래그    | 활성화        | 비네팅 보정 활성화                                                             |
| `--no-vignette`       | 플래그    | -              | 비네팅 보정 비활성화                                                            |
| `--reflectance`       | 플래그    | 활성화됨        | 반사율 보정 활성화                                                         |
| `--no-reflectance`    | 플래그    | -              | 반사율 보정 비활성화                                                        |
| `--ppk`               | 플래그    | 비활성화       | .daq 광 센서 데이터로부터 PPK 보정 적용                                      |
| `--format`            | 선택  | TIFF (16비트)  | 출력 형식: `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)` |
| `--min-target-size`   | 정수 | 자동           | 보정 패널 감지를 위한 최소 대상 크기(픽셀)                          |
| `--target-clustering` | 정수 | 자동           | 대상 클러스터링 임계값(0-100)                                                    |
| `--debayer`           | 선택  | `standard`     | 디베이어 방법: `standard` 또는 `texture-aware` (Chloros+ 전용)                          |
| `--target`, `--targets` | 플래그  | 비활성화       | &quot;target&quot; 또는 &quot;targets&quot; 하위 폴더에서만 보정 대상 검색 (처리 속도 향상) |
| `--indices`           | 목록    | 없음           | 계산할 식생 지수 (예: `--indices NDVI NDRE GNDVI`)                    |
| `--exposure-pin-1`    | 문자열  | 없음           | 카메라 모델별 노출값 고정 (핀 1)                                                 |
| `--exposure-pin-2`    | 문자열  | 없음           | 카메라 모델의 노출 고정 (핀 2)                                                 |
| `--recal-interval`    | 정수 | 자동           | 재보정 간격 (초)                                                      |
| `--timezone-offset`   | 정수 | 0              | 시간대 오프셋 (시간)                                                               |

***

### `login` - 계정 인증

Chloros+ 자격 증명으로 로그인하여 CLI 처리를 활성화하십시오.

**구문:**

```bash
chloros-cli login <email> <password>
```

**예시:**

```bash
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style="warning" %}
**특수 문자**: `$`, `!` 또는 공백과 같은 문자가 포함된 비밀번호는 작은 따옴표로 묶어 주십시오.
{% endhint %}

**출력:**<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>***

### `logout` - 자격 증명 삭제

저장된 자격 증명을 삭제하고 계정에서 로그아웃합니다.

**구문:**

```bash
chloros-cli logout
```

**예시:**

```bash
chloros-cli logout
```

**출력:**

```
✓ Logout successful
ℹ Credentials cleared from cache
```

{% hint style="info" %}
**SDK 사용자**: Python SDK는 또한 Python 스크립트 내에서 자격 증명을 지우는 프로그래밍 방식의 `logout()` 메서드를 제공합니다. 자세한 내용은 [Python SDK 문서](api-python-sdk.md#logout)를 참조하십시오.
{% endhint %}

***

### `status` - 라이선스 상태 확인

현재 라이선스 및 인증 상태를 표시합니다.

**구문:**

```bash
chloros-cli status
```

**예시:**

```bash
chloros-cli status
```

**출력:**

```
╔══════════════════════════════════════╗
║     LICENSE & ACCOUNT INFORMATION    ║
╚══════════════════════════════════════╝

📧 Email: user@example.com
📋 Plan: Chloros+ Professional
🔓 API/CLI Access: Enabled
✓ Status: Active
```

***

### `export-status` - 내보내기 진행 상황 확인

처리 중이거나 처리 후 스레드 4의 내보내기 진행 상황을 모니터링합니다.

**구문:**

```bash
chloros-cli export-status
```

**예시:**

```bash
chloros-cli export-status
```

**사용 사례:** 처리 중일 때 이 명령을 호출하여 내보내기 진행 상황을 확인합니다.***

### `language` - 인터페이스 언어 관리

CLI 인터페이스 언어를 확인하거나 변경합니다.

**구문:**

```bash
# Show current language
chloros-cli language

# List all available languages
chloros-cli language --list

# Set a specific language
chloros-cli language <language-code>
```

**예시:**

```bash
# View current language
chloros-cli language

# List all 38 supported languages
chloros-cli language --list

# Change to Spanish
chloros-cli language es

# Change to Japanese
chloros-cli language ja
```

#### 지원 언어 (총 38개)

| 코드    | 언어              | 원어명      |
| ------- | --------------------- | ---------------- |
| `en`    | 영어               | English          |
| `es`    | 스페인어               | Español          |
| `pt`    | 포르투갈어            | Português        |
| `fr`    | 프랑스어                | Français         |
| `de`    | 독일어                | Deutsch          |
| `it`    | 이탈리아어              | Italiano         |
| `ja`    | 일본어              | 日本語              |
| `ko`    | 한국어                | 한국어              |
| `zh`    | 중국어(간체)  | 简体中文             |
| `zh-TW` | 중국어(번체) | 繁體中文             |
| `ru`    | 러시아어               | Русский          |
| `nl`    | 네덜란드어                | Nederlands       |
| `ar`    | 아랍어                | العربية          |
| `pl`    | 폴란드어                | Polski           |
| `tr`    | 터키어               | Türkçe           |
| `hi`    | 힌디어                 | हिंदी            |
| `id`    | 인도네시아어            | Bahasa Indonesia |
| `vi`    | 베트남어            | Tiếng Việt       |
| `th`    | 태국어                  | ไทย              |
| `sv`    | 스웨덴어               | Svenska          |
| `da`    | 덴마크어                | Dansk            |
| `no`    | 노르웨이어             | Norsk            |
| `fi`    | 핀란드어               | Suomi            |
| `el`    | 그리스어                 | Ελληνικά         |
| `cs`    | 체코어                | Čeština          |
| `hu`    | 헝가리어             | Magyar           |
| `ro`    | 루마니아어              | Română           |
| `uk`    | 우크라이나어             | Українська       |
| `pt-BR` | 브라질 포르투갈어  | Português Brasileiro |
| `zh-HK` | 광둥어             | 粵語             |
| `ms`    | 말레이어                 | Bahasa Melayu    |
| `sk`    | 슬로바키아어                | 슬로베니아어       |
| `bg`    | 불가리아어             | 불가리아어        |
| `hr`    | 크로아티아어              | 크로아티아어         |
| `lt`    | 리투아니아어            | Lietuvių         |
| `lv`    | 라트비아어               | Latviešu         |
| `et`    | 에스토니아어              | Eesti            |
| `sl`    | 슬로베니아어             | Slovenščina      |

{% hint style="success" %}
**자동 저장**: 언어 기본 설정은 `~/.chloros/cli_language.json`에 저장되며 모든 세션에서 유지됩니다.
{% endhint %}

***

### `set-project-folder` - 기본 프로젝트 폴더 설정

기본 프로젝트 폴더 위치를 변경합니다(Windows의 GUI와 공유됨).

**구문:**

```bash
chloros-cli set-project-folder <folder-path>
```

**예시:**

```bash
# Windows
chloros-cli set-project-folder "C:\Projects\2025"

# Linux
chloros-cli set-project-folder ~/projects/2025
```

***

### `get-project-folder` - 프로젝트 폴더 표시

현재 기본 프로젝트 폴더 위치를 표시합니다.

**구문:**

```bash
chloros-cli get-project-folder
```

**예시:**

```bash
chloros-cli get-project-folder
```

**출력:**

```

# Windows
ℹ Current project folder: C:\Projects\2025

# Linux
ℹ Current project folder: /home/user/.local/share/chloros/projects
```

***

### `reset-project-folder` - 기본값으로 재설정

프로젝트 폴더를 기본 위치로 재설정합니다.

**구문:**

```bash
chloros-cli reset-project-folder
```

***

### `selftest` - 시스템 진단 실행

시스템 구성을 확인하기 위해 7가지 진단 검사를 실행합니다.

**구문:**

```bash
chloros-cli selftest
```

**실행되는 진단 항목:**

1. 버전 확인
2. 포트 사용 가능 여부 (5000)
3. 백엔드 시작
4. API 연결 테스트
5. 시스템 정보 및 GPU 감지
6. 노이즈 제거기 모델 확인
7. CUDA 사용 가능 여부 확인

{% hint style="info" %}
**문제 해결에 유용함**: 설치 후 `selftest`를 실행하여 시스템이 올바르게 구성되었는지 확인하십시오. 특히 GPU 및 CUDA 설정을 확인해야 할 수 있는 Linux/Jetson의 경우 더욱 그렇습니다.
{% endhint %}

***

### `update` - 업데이트 확인 (Linux 전용)

Linux 시스템에서 CLI 업데이트를 확인하고 설치합니다.

**구문:**

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

| 옵션    | 설명                        |
| --------- | ---------------------------------- |
| `--check` | 업데이트 확인만 수행하고 설치하지 않음 |

{% hint style="info" %}
이 명령은 Linux에서만 사용할 수 있습니다. Windows에서는 설치 프로그램을 통해 업데이트가 제공됩니다.
{% endhint %}

***

## 전역 옵션

이 옵션들은 모든 명령어에 적용됩니다:

| 옵션            | 유형    | 기본값       | 설명                                      |
| ----------------- | ------- | ------------- | ------------------------------------------------ |
| `--backend-exe`   | 경로    | 자동 감지 | 백엔드 실행 파일의 경로                       |
| `--port`          | 정수    | 5000          | 백엔드 API 포트 번호                          |
| `--restart`       | 플래그    | -             | 백엔드 강제 재시작 (기존 프로세스 종료) |
| `--version`       | 플래그    | -             | 버전 정보 표시 후 종료                |
| `--help`          | 플래그    | -             | 도움말 정보 표시 후 종료                   |

{% hint style="info" %}
**백엔드 자동 감지**: `--backend-exe` 경로는 플랫폼별로 자동 감지됩니다:
* **Windows**: `C:\Program Files\MAPIR\Chloros\resources\backend\chloros-backend.exe`
* **Linux (.deb)**: `/usr/lib/chloros/chloros-backend`
* **Linux (수동)**: `/opt/mapir/chloros/backend/chloros-backend`
{% endhint %}

**전역 옵션 예시:**

**Windows:**

```powershell
chloros-cli --port 5001 process "C:\Datasets\Survey_001"
```

**Linux:**

```bash
chloros-cli --port 5001 process ~/datasets/survey_001
```

***

## 처리 설정 가이드

### 병렬 처리 및 동적 컴퓨트 적응

Chloros 1.1.0에는 [동적 컴퓨트 적응](processing-architecture/dynamic-compute-adaptation.md) 기능이 포함되어 있습니다. 이 처리 엔진은 **사용자의 하드웨어를 자동으로 감지**하여 최적의 전략을 선택합니다:

| 플랫폼 | 전략 | 워커 | 파이프라인 | 참고 사항 |
| --- | --- | --- | --- | --- |
| **Jetson Nano 8GB** | `GPU_SINGLE` | 1 | `tiled_gpu` | 메모리 효율적, 직렬화 |
| **Jetson Orin NX 16GB** | `GPU_PARALLEL` | 3 | `fused_gpu` | 병렬 GPU 처리 |
| **8GB GPU 탑재 데스크톱** | `GPU_SINGLE` | 3 | `tiled_gpu` | 우수한 데스크톱 성능 |
| **12GB 이상 GPU 탑재 데스크톱** | `GPU_PARALLEL` | 3-4 | `fused_gpu` | 최적의 데스크톱 성능 |
| **CPU 전용 시스템** | `CPU_PARALLEL` | 코어 수 - 1 | `cpu_fallback` | GPU 불필요 |

{% hint style="success" %}
**수동 설정 불필요!** Chloros는 CPU, GPU, RAM 및 (Jetson의 경우) 열 센서를 자동으로 감지한 후 최적의 처리 파이프라인을 자동으로 구성합니다.
{% endhint %}

### 디베이어 방법

| 방법 | CLI 플래그 | 품질 | 속도 | 라이선스 |
| --- | --- | --- | --- | --- |
| **표준 (빠름, 중간 품질)** | `--debayer standard` | 양호 | 빠름 | 무료 / Chloros+ |
| **텍스처 인식 (느림, 최고 품질)** | `--debayer texture-aware` | 최고 | 느림 | Chloros+ 전용 |

기본 디베이어 방식은 **표준**입니다.**텍스처 인식** 방식은 최고 품질의 출력을 위해 AI/ML 노이즈 제거 모델을 사용하지만, Chloros+ 라이선스와 NVIDIA GPU가 필요합니다.

```bash
# Use Texture Aware debayer (Chloros+ only)
chloros-cli process ~/datasets/field_a --debayer texture-aware
```

### 비네팅 보정

**기능:** 이미지 가장자리의 밝기 저하(카메라 이미지에서 흔히 나타나는 모서리 부분의 어두움)를 보정합니다.

* **기본적으로 활성화됨** - 대부분의 사용자는 이 기능을 활성화한 상태로 유지해야 합니다.
* 비활성화하려면 `--no-vignette`를 사용하십시오.

{% hint style="success" %}
**권장 사항**: 프레임 전체에 걸쳐 균일한 밝기를 보장하려면 항상 비네팅 보정을 활성화하십시오.
{% endhint %}

### 반사율 보정

보정 패널을 사용하여 센서의 원시 값을 표준화된 반사율 백분율로 변환합니다.

* **기본적으로 활성화됨** - 식생 분석에 필수적입니다
* 이미지에 보정 대상 패널이 필요합니다
* 비활성화하려면 `--no-reflectance`를 사용하십시오

{% hint style="info" %}
**요구 사항**: 정확한 반사율 변환을 위해 이미지에 보정 패널이 적절하게 노출되고 명확하게 보이도록 하십시오.
{% endhint %}

### PPK 보정

**기능:** DAQ-A-SD 로그 데이터를 사용하여 후처리 운동학 보정을 적용하여 GPS 정확도를 향상시킵니다.

* **기본적으로 비활성화됨**
* 활성화하려면 `--ppk`를 사용하십시오
* MAPIR DAQ-A-SD 광 센서에서 생성된 .daq 파일이 프로젝트 폴더에 있어야 합니다.

### 출력 형식

<table><thead><tr><th width="197">형식</th><th width="130.20001220703125">비트 심도</th><th width="116.5999755859375">파일 크기</th><th>최적 용도</th></tr></thead><tbody><tr><td><strong>TIFF (16비트)</strong> ⭐</td><td>16비트 정수</td><td>대용량</td><td>GIS 분석, 사진 측량 (권장)</td></tr><tr><td><strong>TIFF (32비트, 백분율)</strong></td><td>32비트 부동 소수점</td><td>매우 대용량</td><td>과학적 분석, 연구</td></tr><tr><td><strong>PNG (8비트)</strong></td><td>8비트 정수</td><td>중간</td><td>육안 검사, 웹 공유</td></tr><tr><td><strong>JPG (8비트)</strong></td><td>8비트 정수</td><td>소형</td><td>빠른 미리보기, 압축된 출력</td></tr></tbody></table>***

## 자동화 및 스크립팅

### PowerShell 일괄 처리 (Windows)

Windows에서 여러 데이터셋 폴더를 자동으로 처리합니다:

```powershell
# process_all_datasets.ps1

$datasets = Get-ChildItem "C:\Datasets\2025" -Directory

foreach ($dataset in $datasets) {
    Write-Host "Processing $($dataset.Name)..." -ForegroundColor Cyan
    
    chloros-cli process $dataset.FullName `
        --vignette `
        --reflectance
    
    if ($LASTEXITCODE -eq 0) {
        Write-Host "✓ $($dataset.Name) complete" -ForegroundColor Green
    } else {
        Write-Host "✗ $($dataset.Name) failed" -ForegroundColor Red
    }
}

Write-Host "All datasets processed!" -ForegroundColor Green
```

### Windows 배치 스크립트 (Windows)

Windows에서 배치 처리를 위한 간단한 루프:

```batch
@echo off
echo Starting batch processing...

for /d %%i in (C:\Datasets\2025\*) do (
    echo.
    echo ========================================
    echo Processing: %%i
    echo ========================================
    chloros-cli process "%%i"
    
    if %ERRORLEVEL% EQU 0 (
        echo SUCCESS: %%i processed
    ) else (
        echo ERROR: %%i failed
    )
)

echo.
echo All datasets processed!
pause
```

### Bash 배치 처리 (Linux)

Linux에서 여러 데이터셋 폴더 처리:

```bash
#!/bin/bash
# process_all_datasets.sh

for dataset in ~/datasets/2026/*/; do
    name=$(basename "$dataset")
    echo "Processing $name..."

    chloros-cli process "$dataset" \
        --vignette \
        --reflectance

    if [ $? -eq 0 ]; then
        echo "✓ $name complete"
    else
        echo "✗ $name failed"
    fi
done

echo "All datasets processed!"
```

### Python 자동화 스크립트 (크로스 플랫폼)

오류 처리가 포함된 고급 자동화 (Windows 및 Linux에서 작동):

```python
import subprocess
import os
import sys
from pathlib import Path
from datetime import datetime

def process_dataset(input_folder):
    """Process a folder using Chloros CLI"""
    cmd = ['chloros-cli', 'process', str(input_folder)]
    
    # Execute command
    result = subprocess.run(
        cmd, 
        capture_output=True, 
        text=True,
        encoding='utf-8'
    )
    
    return result.returncode == 0, result.stdout, result.stderr

def main():
    """Process all datasets in a directory"""
    # Adjust path for your platform
    # Windows: Path('C:/Datasets/2025')
    # Linux:   Path.home() / 'datasets' / '2025'
    datasets_dir = Path('C:/Datasets/2025')
    log_file = Path('processing_log.txt')
    
    successful = []
    failed = []
    
    # Start processing
    print(f"Starting batch processing: {datetime.now()}")
    print(f"Scanning: {datasets_dir}")
    print("=" * 60)
    
    for dataset_folder in sorted(datasets_dir.iterdir()):
        if not dataset_folder.is_dir():
            continue
        
        print(f"\nProcessing: {dataset_folder.name}")
        
        success, stdout, stderr = process_dataset(dataset_folder)
        
        if success:
            print(f"✓ {dataset_folder.name} - SUCCESS")
            successful.append(dataset_folder.name)
        else:
            print(f"✗ {dataset_folder.name} - FAILED")
            failed.append(dataset_folder.name)
            
            # Log error details
            with open(log_file, 'a', encoding='utf-8') as f:
                f.write(f"\n=== {dataset_folder.name} - {datetime.now()} ===\n")
                f.write(f"STDOUT:\n{stdout}\n")
                f.write(f"STDERR:\n{stderr}\n")
    
    # Print summary
    print("\n" + "=" * 60)
    print(f"SUMMARY - Completed: {datetime.now()}")
    print(f"  Successful: {len(successful)}")
    print(f"  Failed: {len(failed)}")
    
    if failed:
        print(f"\nFailed folders:")
        for folder in failed:
            print(f"  - {folder}")
        print(f"\nCheck {log_file} for error details")
        sys.exit(1)
    else:
        print("\nAll datasets processed successfully!")
        sys.exit(0)

if __name__ == '__main__':
    main()
```

***

## 처리 워크플로우

### 표준 워크플로우

1. **입력**: RAW/JPG 이미지 쌍이 포함된 폴더
2. **탐색**: CLI가 지원되는 이미지 파일을 자동으로 스캔합니다
3. **처리**: 병렬 모드는 CPU 코어 수에 따라 확장됩니다 (Chloros+)
4. **출력**: 처리된 이미지가 포함된 카메라 모델별 하위 폴더 생성

### 출력 구조 예시

```

MyProject/
├── project.json                             # Project metadata
├── 2025_0203_193056_008.JPG                # Original JPG
├── 2025_0203_193055_007.RAW                # Original RAW
└── Survey3N_RGN/                           # Processed outputs ✓
    ├── 2025_0203_193056_008_Reflectance.tif   # Calibrated reflectance
    ├── 2025_0203_193056_008_Target.tif        # Target detection
    └── ...
```

### 예상 처리 시간

이미지 100장(각 12MP)의 일반적인 처리 시간:

| 플랫폼 | 모드 | 예상 시간 | 비고 |
| --- | --- | --- | --- |
| **데스크톱 12GB+ GPU** | `GPU_PARALLEL` | 5-10분 | 가장 빠른 옵션 |
| **데스크톱 8GB GPU** | `GPU_SINGLE` | 10-15분 | 우수한 성능 |
| **Jetson Orin NX 16GB** | `GPU_PARALLEL` | 15-25분 | 엣지 컴퓨팅 |
| **Jetson Nano 8GB** | `GPU_SINGLE` | 30-60분 | 메모리 제약 |
| **CPU 전용** | `CPU_PARALLEL` | 20-40분 | GPU 불필요 |

{% hint style="info" %}
**성능 팁**: 처리 시간은 이미지 수, 해상도, 디베이어 방식 및 하드웨어에 따라 달라집니다. 텍스처 인식 디베이어는 표준 방식보다 훨씬 더 오래 걸립니다. 자세한 내용은 [동적 컴퓨팅 적응](processing-architecture/dynamic-compute-adaptation.md)을 참조하십시오.
{% endhint %}

***

## 문제 해결

### CLI를 찾을 수 없음

**Windows 오류:**

```
'chloros-cli' is not recognized as an internal or external command
```

**Windows 해결 방법:**

1. 설치 위치 확인:

```powershell
dir "C:\Program Files\Chloros\resources\cli\chloros-cli.exe"
```

2. PATH에 포함되어 있지 않은 경우 전체 경로 사용:

```powershell
"C:\Program Files\Chloros\resources\cli\chloros-cli.exe" process "C:\Datasets\Field_A"
```

3. PATH에 수동으로 추가:
   * 시스템 속성 → 환경 변수 열기
   * PATH 변수 편집
   * 추가: `C:\Program Files\Chloros\resources\cli`
   * 터미널 재시작

**Linux 오류:**

```
chloros-cli: command not found
```

**Linux 해결 방법:**

1. 설치 확인:

```bash
which chloros-cli
dpkg -L chloros-amd64  # or chloros-arm64-jp6
```

2. 셸 재로드:

```bash
source ~/.bashrc
```

3. 권한 확인:

```bash
sudo chmod +x /usr/bin/chloros-cli
```

***

### 백엔드 시작 실패**오류:**

```

Backend failed to start within 30 seconds
```

**해결 방법:**

1. 백엔드가 이미 실행 중인지 확인하십시오(먼저 종료하십시오).
2. 방화벽이 차단하고 있지 않은지 확인하십시오(Windows) 또는 포트 사용 가능 여부를 확인하십시오(Linux: `lsof -i :5000`)
3. 다른 포트를 사용해 보세요:

```bash
# Windows
chloros-cli --port 5001 process "C:\Datasets\Field_A"

# Linux
chloros-cli --port 5001 process ~/datasets/field_a
```

4. 백엔드를 강제 재시작하세요:

```bash
# Windows
chloros-cli --restart process "C:\Datasets\Field_A"

# Linux
chloros-cli --restart process ~/datasets/field_a
```

5. Linux 오류 시, 백엔드 실행 파일이 있는지 확인하세요:

```bash
ls -la /usr/lib/chloros/chloros-backend
```

***

### 라이선스/인증 문제**오류:**

```

Chloros+ license required for CLI access
```

**해결 방법:**

1. 유효한 Chloros+ 구독이 있는지 확인하십시오.
2. 자격 증명으로 로그인하십시오:

```bash
chloros-cli login user@example.com 'password'
```

3. 라이선스 상태 확인:

```bash
chloros-cli status
```

4. 지원팀에 문의: info@mapir.camera

***

### 이미지를 찾을 수 없음**오류:**

```

No images found in the specified folder
```

**해결 방법:**

1. 폴더에 지원되는 형식(.RAW, .TIF, .JPG)이 포함되어 있는지 확인하십시오.
2. 폴더 경로가 올바른지 확인하십시오(공백이 포함된 경로의 경우 따옴표를 사용하십시오).
3. 폴더에 대한 읽기 권한이 있는지 확인하십시오.
4. 파일 확장자가 올바른지 확인하십시오.

***

### 처리 중지 또는 멈춤**해결 방법:**

1. 사용 가능한 디스크 공간 확인 (출력용 공간이 충분한지 확인)
2. 다른 응용 프로그램을 닫아 메모리를 확보하십시오
3. 이미지 수를 줄이십시오 (일괄 처리)

***

### 포트가 이미 사용 중입니다**오류:**

```

Port 5000 is already in use
```

**해결 방법:**

**Windows:**

```powershell
chloros-cli --port 5001 process "C:\Datasets\Field_A"
```

**Linux:**

```bash
# Find what's using port 5000
lsof -i :5000

# Use a different port
chloros-cli --port 5001 process ~/datasets/field_a
```

***

## 자주 묻는 질문

### Q: CLI를 사용하려면 라이선스가 필요한가요?

**A:**네! CLI를 사용하려면 유료**Chloros+ 라이선스**가 필요합니다.

* ❌ Standard(무료) 플랜: CLI 사용 불가
* ✅ Chloros+(유료) 플랜: CLI 완전 사용 가능

구독하기: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

***

### Q: GUI가 없는 서버에서 CLI를 사용할 수 있나요?**A:** 네! CLI는 완전히 헤드리스(headless) 모드로 실행됩니다. 이는 Linux의 주요 사용 사례입니다.**Windows 서버:**
* Windows Server 2016 이상
* Visual C++ 재배포 가능 패키지 설치됨

**Linux 서버:**
* Ubuntu 20.04 이상 / Debian 11 이상 (amd64) 또는 JetPack 6 (arm64)
* `.deb` 패키지를 통해 설치

**두 플랫폼 모두:**
* 최소 8GB RAM (16GB 권장)
* 일회성 라이선스 활성화: `chloros-cli login user@example.com 'password'`

***

### Q: 처리된 이미지는 어디에 저장되나요?**A:**기본적으로 처리된 이미지는**입력 파일과 동일한 폴더** 내의 카메라 모델 하위 폴더(예: `Survey3N_RGN/`)에 저장됩니다.

다른 출력 폴더를 지정하려면 `-o` 옵션을 사용하십시오:

```bash
# Windows
chloros-cli process "C:\Input" -o "D:\Output"

# Linux
chloros-cli process ~/input -o ~/output
```

***

### Q: 여러 폴더를 한 번에 처리할 수 있나요?**A:** 하나의 명령어로 직접 처리할 수는 없지만, 스크립트를 사용하여 폴더를 순차적으로 처리할 수 있습니다. [자동화 및 스크립팅](CLI.md#automation--scripting) 섹션을 참조하십시오.***

### Q: CLI 출력을 로그 파일에 저장하려면 어떻게 해야 합니까?**PowerShell:**

```powershell
chloros-cli process "C:\Datasets\Field_A" | Tee-Object -FilePath "processing.log"
```

**배치:**

```batch
chloros-cli process "C:\Datasets\Field_A" > processing.log 2>&1
```

**Linux Bash:**

```bash
chloros-cli process ~/datasets/field_a 2>&1 | tee processing.log
```

***

### Q: 처리 중에 Ctrl+C를 누르면 어떻게 되나요?**A:** CLI는 다음과 같이 동작합니다:

1. 정상적으로 처리를 중지합니다
2. 백엔드를 종료합니다
3. 코드 130으로 종료합니다

부분적으로 처리된 이미지가 출력 폴더에 남아 있을 수 있습니다.

***

### Q: CLI 처리를 자동화할 수 있나요?**A:** 물론입니다! CLI는 자동화를 위해 설계되었습니다. PowerShell (Windows), Batch (Windows), Bash(Linux) 및 Python(크로스 플랫폼) 예제를 참조하십시오.***

### Q: CLI 버전을 확인하려면 어떻게 해야 합니까?**A:**

```bash
chloros-cli --version
```

**출력:**

```

Chloros CLI 1.1.0
```

***

## 도움말 받기

### 명령줄 도움말

CLI에서 직접 도움말 정보를 확인하세요:

```bash
# General help
chloros-cli --help

# Command-specific help
chloros-cli process --help
chloros-cli login --help
chloros-cli language --help
```

### 지원 채널

* **이메일**: info@mapir.camera
* **웹사이트**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* **가격**: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)***

## 전체 예제

### 예제 1: 기본 처리

기본 설정(비네트, 반사율)으로 처리:

**Windows:**

```powershell
chloros-cli process "C:\Datasets\Field_A_2025_01_15"
```

**Linux:**

```bash
chloros-cli process ~/datasets/field_a_2025_01_15
```

***

### 예제 2: 고품질 과학적 출력

32비트 부동소수점 TIFF:

**Windows:**

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "TIFF (32-bit, Percent)" ^
  --vignette ^
  --reflectance
```

**Linux:**

```bash
chloros-cli process ~/datasets/field_a \
  --format "TIFF (32-bit, Percent)" \
  --vignette \
  --reflectance
```

***

### 예제 3: 빠른 미리보기 처리

신속한 검토를 위해 보정되지 않은 8비트 PNG:

**Windows:**

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "PNG (8-bit)" ^
  --no-vignette ^
  --no-reflectance
```

**Linux:**

```bash
chloros-cli process ~/datasets/field_a \
  --format "PNG (8-bit)" \
  --no-vignette \
  --no-reflectance
```

***

### 예제 4: PPK 보정 처리

반사율을 사용하여 PPK 보정 적용:

**Windows:**

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --ppk ^
  --reflectance
```

**Linux:**

```bash
chloros-cli process ~/datasets/field_a \
  --ppk \
  --reflectance
```

***

### 예제 5: 사용자 지정 출력 위치

특정 형식으로 다른 위치에 처리:

**Windows:**

```powershell
chloros-cli process "C:\Input\Raw_Images" ^
  -o "D:\Output\Processed" ^
  --format "TIFF (16-bit)"
```

**Linux:**

```bash
chloros-cli process ~/input/raw_images \
  -o ~/output/processed \
  --format "TIFF (16-bit)"
```

***

### 예제 6: 인증 워크플로

전체 인증 흐름 (모든 플랫폼에서 동일):

```bash
# Step 1: Login
chloros-cli login user@example.com 'MyP@ssw0rd'

# Step 2: Verify status
chloros-cli status

# Step 3: Process images
# Windows: chloros-cli process "C:\Datasets\Field_A"
# Linux:   chloros-cli process ~/datasets/field_a
chloros-cli process ~/datasets/field_a

# Step 4: Logout (optional, when switching accounts)
chloros-cli logout
```

***

### 예제 7: 다국어 사용

인터페이스 언어 변경 (모든 플랫폼에서 동일):

```bash
# List available languages
chloros-cli language --list

# Change to Spanish
chloros-cli language es

# Process with Spanish interface
# Windows: chloros-cli process "C:\Vuelos\Campo_A"
# Linux:   chloros-cli process ~/vuelos/campo_a
chloros-cli process ~/vuelos/campo_a

# Change back to English
chloros-cli language en
```
