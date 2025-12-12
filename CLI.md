# CLI : 명령줄

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>**Chloros CLI**는 Chloros 이미지 처리 엔진에 대한 강력한 명령줄 접근을 제공하여, 이미징 워크플로우의 자동화, 스크립팅 및 헤드리스 운영을 가능하게 합니다.

### 주요 기능

* 🚀 **자동화** - 다중 데이터셋의 배치 처리 스크립팅
* 🔗 **통합** - 기존 워크플로우 및 파이프라인에 임베드
* 💻 **헤드리스 운영** - GUI 없이 실행
* 🌍 **다국어 지원** - 38개 언어 지원
* ⚡ **병렬 처리** - CPU에 따라 동적 확장 (최대 16개 병렬 작업자)

### 요구 사항

| 요구 사항          | 세부 사항                                                             |
| -------------------- | ------------------------------------------------------------------- |
| **운영 체제** | Windows 10/11 (64비트)                                              |
| **라이선스**          | Chloros+ ([유료 플랜 필요](https://cloud.mapir.camera/pricing)) |
| **메모리**           | 최소 8GB RAM (권장 16GB)                                  |
| **인터넷**         | 라이선스 활성화 필수                                     |
| **디스크 공간**       | 프로젝트 규모에 따라 다름                                              |

{% hint style=&quot;warning&quot; %}
**라이선스 요구 사항**: CLI 사용에는 유료 Chloros+ 구독이 필요합니다. 표준(무료) 플랜은 CLI 접근 권한이 없습니다. 업그레이드하려면 [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)를 방문하세요.
{% endhint %}

## 빠른 시작

### 설치

CLI는 Chloros 설치 프로그램에 자동으로 포함됩니다:

1. **Chloros 설치 프로그램.exe**를 다운로드하여 실행하세요
2. 설치 마법사를 완료하세요
3. CLI 설치 위치: `C:\Program Files\Chloros\resources\cli\chloros-cli.exe`

{% hint style=&quot;success&quot; %}
설치 프로그램은 자동으로 `chloros-cli`를 시스템 PATH에 추가합니다. 설치 후 터미널을 재시작하십시오.
{% endhint %}

### 초기 설정

CLI 사용 전, Chloros+ 라이선스를 활성화하세요:

```bash
# Login with your Chloros+ account
chloros-cli login user@example.com 'your_password'

# Check license status
chloros-cli status

# Process your first project
chloros-cli process "C:\Images\Dataset001"
```

### 기본 사용법

기본 설정으로 폴더 처리:

```powershell
chloros-cli process "C:\Images\Dataset001"
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

보정된 폴더 내 이미지 처리.

**구문:**

```bash
chloros-cli process <input-folder> [options]
```

**예시:**

```powershell
chloros-cli process "C:\Datasets\Survey_001" --vignette --reflectance
```

#### 처리 명령 옵션

| 옵션                | 유형    | 기본값        | 설명                                                                            |
| --------------------- | ------- | -------------- | -------------------------------------------------------------------------------------- |
| `<input-folder>`      | 경로    | _필수_     | RAW/JPG 다중 스펙트럼 이미지가 포함된 폴더                                         |
| `-o, --output`        | 경로    | 입력과 동일  | 처리된 이미지의 출력 폴더                                                     |
| `-n, --project-name`  | 문자열  | 자동 생성 | 사용자 정의 프로젝트 이름                                                                    |
| `--vignette`          | 플래그    | 활성화        | 비네팅 보정 활성화                                                             |
| `--no-vignette`       | 플래그    | -              | 비네팅 보정 비활성화                                                            |
| `--reflectance`       | 플래그    | 활성화        | 반사율 보정 활성화                                                         |
| `--no-reflectance`    | 플래그    | -              | 반사율 보정 비활성화                                                        |
| `--ppk`               | 플래그    | 비활성화       | .daq 광 센서 데이터로부터 PPK 보정 적용                                      |
| `--format`            | 선택      | TIFF (16비트)  | 출력 형식: `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)` |
| `--min-target-size`   | 정수 | 자동           | 보정 패널 감지를 위한 최소 대상 크기 (픽셀)                          |
| `--target-clustering` | 정수 | 자동           | 대상 클러스터링 임계값 (0-100)                                                    |
| `--exposure-pin-1`    | 문자열  | 없음           | 카메라 모델별 노출 고정 (핀 1)                                                 |
| `--exposure-pin-2`    | 문자열  | 없음           | 카메라 모델별 노출 고정 (핀 2)                                                 |
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

```powershell
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style=&quot;warning&quot; %}
**특수 문자**: `$`, `!` 또는 공백과 같은 문자가 포함된 비밀번호는 작은따옴표(&#x27;&#x27;)로 묶어 사용하십시오.
{% endhint %}

**출력:**

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>***

### `logout` - 저장된 자격 증명 삭제

저장된 자격 증명을 삭제하고 계정에서 로그아웃합니다.

**구문:**

```bash
chloros-cli logout
```

**예시:**

```powershell
chloros-cli logout
```

**출력:**

```
✓ Logout successful
ℹ Credentials cleared from cache
```

***

### `status` - 라이선스 상태 확인

현재 라이선스 및 인증 상태를 표시합니다.

**구문:**

```bash
chloros-cli status
```

**예시:**

```powershell
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

### `export-status` - 내보내기 진행 상태 확인

처리 중 또는 처리 후 스레드 4의 내보내기 진행 상태를 모니터링합니다.

**구문:**

```bash
chloros-cli export-status
```

**예시:**

```powershell
chloros-cli export-status
```

**사용 사례:** 처리 실행 중 이 명령을 호출하여 내보내기 진행 상황을 확인합니다.

***

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

```powershell
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

| 코드    | 언어               | 기본 언어명      |
| ------- | --------------------- | ---------------- |
| `en`    | 영어               | English          |
| `es`    | 스페인어               | Español          |
| `pt`    | 포르투갈어            | Português        |
| `fr`    | 프랑스어                | Français         |
| `de`    | 독일어                | Deutsch          |
| `it`    | 이탈리아어               | Italiano         |
| `ja`    | 일본어              | 日本語              |
| `ko`    | 한국어                | 한국어              |
| `zh`    | 중국어(간체)  | 简体中文             |
| `zh-TW` | 중국어(번체) | 繁體中文             |
| `ru`    | 러시아어               | Русский          |
| `nl`    | 네덜란드어         | Nederlands       |
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
| `cs`    | 체코어                 | Čeština          |
| `hu`    | 헝가리어             | Magyar           |
| `ro`    | 루마니아어              | Română           |
| `uk`    | 우크라이나어             | Українська       |
| `pt-BR` | 브라질 포르투갈어  | Português Brasileiro |
| `zh-HK` | 광둥어             | 粵語             |
| `ms`    | 말레이어                 | Bahasa Melayu    |
| `sk`    | 슬로바키아어                | Slovenčina       |
| `bg`    | 불가리아어             | Български        |
| `hr`    | 크로아티아어              | Hrvatski         |
| `lt`    | 리투아니아어            | Lietuvių         |
| `lv`    | 라트비아어         | Latviešu         |
| `et`    | 에스토니아어        | Eesti            |
| `sl`    | 슬로베니아어        | Slovenščina      |

{% hint style=&quot;success&quot; %}
**자동 지속성**: 귀하의 언어 선호도는 `~/.chloros/cli_language.json`에 저장되며 모든 세션에 걸쳐 유지됩니다.
{% endhint %}

***

### `set-project-folder` - 기본 프로젝트 폴더 설정

기본 프로젝트 폴더 위치(GUI와 공유)를 변경합니다.

**구문:**

```bash
chloros-cli set-project-folder <folder-path>
```

**예시:**

```powershell
chloros-cli set-project-folder "C:\Projects\2025"
```

***

### `get-project-folder` - 프로젝트 폴더 표시

현재 기본 프로젝트 폴더 위치를 표시합니다.

**구문:**

```bash
chloros-cli get-project-folder
```

**예시:**

```powershell
chloros-cli get-project-folder
```

**출력:**

```
ℹ Current project folder: C:\Projects\2025
```

***

### `reset-project-folder` - 기본값으로 재설정

프로젝트 폴더를 기본 위치로 재설정합니다.

**구문:**

```bash
chloros-cli reset-project-folder
```

***

## 글로벌 옵션

이 옵션들은 모든 명령어에 적용됩니다:

| 옵션          | 유형    | 기본값       | 설명                                                      |
| --------------- | ------- | ------------- | ------------------------------------------------ |
| `--backend-exe` | 경로    | 자동 감지 | 백엔드 실행 파일 경로                       |
| `--port`        | 정수    | 5000          | 백엔드 API 포트 번호                          |
| `--restart`     | 플래그    | -             | 백엔드 강제 재시작 (기존 프로세스 종료) |
| `--version`     | 플래그    | -             | 버전 정보 표시 후 종료                |
| `--help`        | 플래그    | -             | 도움말 정보 표시 후 종료                   |

**글로벌 옵션 예시:**

```powershell
chloros-cli --port 5001 process "C:\Datasets\Survey_001"
```

***

## 처리 설정 가이드

### 병렬 처리

Chloros+ CLI **자동으로 확장**하여 컴퓨터 성능에 맞춰 병렬 처리를 수행합니다:

**작동 방식:**

* CPU 코어 및 RAM 감지
* 작업자 할당: **CPU 코어 수 × 2** (하이퍼스레딩 활용)
* **최대: 16개 병렬 작업자** (안정성 확보)

**시스템 등급:**

| 시스템 유형   | CPU        | RAM      | 작업자  | 성능     |
| ---------| **고성능**  | 16+ 코어  | 32+ GB   | 최대 16개 | 최고 속도   |
| **중급** | 8-15 코어 | 16-31 GB | 8-16     | 우수한 속도 |
| **저급**   | 4-7 코어 | 8-15 GB  | 4-8      | 양호한 속도      |

{% hint style=&quot;success&quot; %}
**자동 최적화**: CLI는 시스템 사양을 자동으로 감지하여 최적의 병렬 처리를 구성합니다. 수동 설정 불필요!
{% endhint %}

### 디베이어 방법

CLI는 기본값이자 권장 디베이어 알고리즘으로 **고품질(빠름)**을 사용합니다:

| 방법                      | 품질 | 속도 | 설명                                 |
| --------------------------- | ------- | ----- | ------------------------------------------- |
| **고품질(빠름)** ⭐ | ⭐⭐⭐⭐    | ⚡⚡⚡   | 경계 인식 알고리즘 (기본값, 권장) |

### 비네트 보정

**기능:** 이미지 가장자리의 빛 감쇠(카메라 이미지에 흔히 나타나는 어두운 모서리)를 보정합니다.

* **기본적으로 활성화됨** - 대부분의 사용자는 이 기능을 켜두어야 함
* 비활성화하려면 `--no-vignette` 사용

{% hint style=&quot;success&quot; %}
**권장 사항**: 프레임 전체에 걸쳐 균일한 밝기를 보장하려면 항상 비네팅 보정을 활성화하십시오.
{% endhint %}

### 반사율 보정

보정 패널을 사용하여 원시 센서 값을 표준화된 반사율 백분율로 변환합니다.

* **기본 활성화됨** - 식생 분석에 필수적
* 이미지에 보정 대상 패널 필요
* 비활성화하려면 `--no-reflectance` 사용

{% hint style=&quot;info&quot; %}
**필수 조건**: 정확한 반사율 변환을 위해 보정 패널이 이미지에 적절히 노출되고 가시적이어야 합니다.
{% endhint %}

### PPK 보정

**기능:** DAQ-A-SD 로그 데이터를 사용한 사후 동적 보정(PPK)을 적용하여 GPS 정확도를 향상시킵니다.

* **기본적으로 비활성화됨**
* 활성화하려면 `--ppk` 사용
* MAPIR DAQ-A-SD 광 센서의 .daq 파일을 프로젝트 폴더에 포함해야 함.

### 출력 형식

<table><thead><tr><th width="197">형식</th><th width="130.20001220703125">비트 심도</th><th width="116.5999755859375">파일 크기</th><th>최적 용도</th></tr></thead><tbody><tr><td><strong>TIFF (16비트)</strong> ⭐</td><td>16비트 정수</td><td>대용량</td><td>GIS 분석, 사진 측량 (권장)</td></tr><tr><td><strong>TIFF (32비트, 백분율)</strong></td><td>32비트 부동 소수점</td><td>매우 큰</td><td>과학적 분석, 연구</td></tr><tr><td><strong>PNG (8비트)</strong></td><td>8비트 정수</td><td>중간</td><td>육안 검사, 웹 공유</td></tr><tr><td><strong>JPG (8비트)</strong></td><td>8비트 정수</td><td>소형</td><td>빠른 미리보기, 압축 출력</td></tr></tbody></table>***

## 자동화 및 스크립팅

### PowerShell 일괄 처리

여러 데이터셋 폴더 자동 처리:

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

### Windows 일괄 스크립트

일괄 처리를 위한 간단한 루프:

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

### Python 자동화 스크립트

오류 처리를 포함한 고급 자동화:

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

## 처리 워크플로

### 표준 워크플로

1. **입력**: RAW/JPG 이미지 쌍이 포함된 폴더
2. **탐색**: CLI 지원 이미지 파일 자동 스캔
3. **처리**: 병렬 모드(Chloros+)로 CPU 코어 수에 따라 확장
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

100장(각 12MP) 이미지 처리 시 일반적인 소요 시간:

| 모드              | 시간      | 하드웨어                                     |
| ----------------- | --------- | -------------------------------------------- |
| **병렬 모드** | 5-10분  | i7/Ryzen 7, 16GB RAM, SSD (최대 16개 작업자) |
| **병렬 모드** | 10-15분 | i5/Ryzen 5, 8GB RAM, HDD (최대 8개 작업자)   |

{% hint style=&quot;info&quot; %}
**성능 팁**: 처리 시간은 이미지 수, 해상도 및 컴퓨터 사양에 따라 달라집니다.
{% endhint %}

***

## 문제 해결

### CLI 찾을 수 없음

**오류:**

```
'chloros-cli' is not recognized as an internal or external command
```

**해결 방법:**

1. 설치 위치 확인:

```powershell
dir "C:\Program Files\Chloros\resources\cli\chloros-cli.exe"
```

2. PATH에 없는 경우 전체 경로 사용:

```powershell
"C:\Program Files\Chloros\resources\cli\chloros-cli.exe" process "C:\Datasets\Field_A"
```

3. PATH에 수동으로 추가:
   * 시스템 속성 → 환경 변수 열기
   * PATH 변수 편집
   * 추가: `C:\Program Files\Chloros\resources\cli`
   * 터미널 재시작

***

### 백엔드 시작 실패

**오류:**

```
Backend failed to start within 30 seconds
```

**해결 방법:**

1. 백엔드가 이미 실행 중인지 확인 (먼저 종료)
2. 방화벽이 차단하지 않는지 확인
3. 다른 포트 시도:

```powershell
chloros-cli --port 5001 process "C:\Datasets\Field_A"
```

4. 백엔드 강제 재시작:

```powershell
chloros-cli --restart process "C:\Datasets\Field_A"
```

***

### 라이선스/인증 문제

**오류:**

```
Chloros+ license required for CLI access
```

**해결 방법:**

1. 유효한 Chloros+ 구독이 있는지 확인하십시오
2. 자격 증명으로 로그인하십시오:

```powershell
chloros-cli login user@example.com 'password'
```

3. 라이선스 상태를 확인하십시오:

```powershell
chloros-cli status
```

4. 지원팀에 문의하십시오: info@mapir.camera

***

### 이미지 미검출

**오류:**

```
No images found in the specified folder
```

**해결 방법:**

1. 폴더에 지원되는 형식(.RAW, .TIF, .JPG)이 포함되어 있는지 확인하십시오
2. 폴더 경로가 올바른지 확인하십시오(공백이 포함된 경로는 따옴표로 묶으십시오)
3. 폴더에 대한 읽기 권한이 있는지 확인하십시오
4. 파일 확장자가 올바른지 확인하십시오

***

### 처리 중지 또는 멈춤

**해결 방법:**

1. 사용 가능한 디스크 공간을 확인하십시오(출력에 충분한지 확인)
2. 메모리를 확보하기 위해 다른 애플리케이션을 닫으십시오
3. 이미지 수를 줄이십시오(배치로 처리)

***

### 포트 사용 중

**오류:**

```
Port 5000 is already in use
```

**해결 방법:**

다른 포트를 지정하세요:

```powershell
chloros-cli --port 5001 process "C:\Datasets\Field_A"
```

***

## 자주 묻는 질문

### Q: CLI 사용에 라이선스가 필요한가요?

**A:** 예! CLI는 유료 **Chloros+ 라이선스**가 필요합니다.

* ❌ 표준(무료) 플랜: CLI 비활성화
* ✅ Chloros+ (유료) 플랜: CLI 완전히 활성화됨

구독하기: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

***

### Q: GUI 없이 서버에서 CLI를 사용할 수 있나요?

**A:** 네! CLI는 완전히 헤드리스(headless)로 실행됩니다. 요구 사항:

* Windows Server 2016 이상
* Visual C++ 재배포 가능 패키지 설치
* 충분한 RAM (최소 8GB, 권장 16GB)
* 모든 머신에서 일회성 GUI 라이선스 활성화

***

### Q: 처리된 이미지는 어디에 저장되나요?

**A:** 기본적으로 처리된 이미지는 입력 파일과 **동일한 폴더** 내 카메라 모델별 하위 폴더(예: `Survey3N_RGN/`)에 저장됩니다.

다른 출력 폴더를 지정하려면 `-o` 옵션을 사용하십시오:

```powershell
chloros-cli process "C:\Input" -o "D:\Output"
```

***

### Q: 여러 폴더를 동시에 처리할 수 있나요?

**A:** 한 번의 명령으로 직접 처리할 수는 없지만, 스크립팅을 사용하여 폴더를 순차적으로 처리할 수 있습니다. [자동화 및 스크립팅](CLI.md#automation--scripting) 섹션을 참조하세요.

***

### Q: CLI 출력을 로그 파일에 저장하려면 어떻게 하나요?

**PowerShell:**

```powershell
chloros-cli process "C:\Datasets\Field_A" | Tee-Object -FilePath "processing.log"
```

**배치:**

```batch
chloros-cli process "C:\Datasets\Field_A" > processing.log 2>&1
```

***

### Q: 처리 중 Ctrl+C를 누르면 어떻게 되나요?

**A:** CLI는 다음과 같이 동작합니다:

1. 처리 과정을 정상적으로 중지합니다
2. 백엔드를 종료합니다
3. 종료 코드 130으로 종료합니다

부분적으로 처리된 이미지가 출력 폴더에 남아 있을 수 있습니다.

***

### Q: CLI 처리를 자동화할 수 있나요?

**A:** 물론입니다! CLI는 자동화를 위해 설계되었습니다. PowerShell, 배치, Python 예제는 [자동화 및 스크립팅](CLI.md#automation--scripting)을 참조하십시오.

***

### Q: CLI 버전을 어떻게 확인하나요?

**A:**

```powershell
chloros-cli --version
```

**출력:**

```
Chloros CLI 1.0.2
```

***

## 도움말 받기

### 명령줄 도움말

CLI에서 직접 도움말 정보를 확인하세요:

```powershell
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
* **가격**: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

***

## 완전한 예시

### 예시 1: 기본 처리

기본 설정(비네팅, 반사율)으로 처리:

```powershell
chloros-cli process "C:\Datasets\Field_A_2025_01_15"
```

***

### 예시 2: 고품질 과학적 출력

32비트 부동소수점 TIFF:

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "TIFF (32-bit, Percent)" ^
  --vignette ^
  --reflectance
```

***

### 예시 3: 빠른 미리보기 처리

빠른 검토를 위한 보정 없는 8비트 PNG:

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "PNG (8-bit)" ^
  --no-vignette ^
  --no-reflectance
```

***

### 예시 4: PPK 보정 처리

반사율과 함께 PPK 보정 적용:

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --ppk ^
  --reflectance
```

***

### 예시 5: 사용자 지정 출력 위치

특정 형식으로 다른 드라이브에 처리:

```powershell
chloros-cli process "C:\Input\Raw_Images" ^
  -o "D:\Output\Processed" ^
  --format "TIFF (16-bit)"
```

***

### 예시 6: 인증 워크플로

인증 흐름 완료:

```powershell
# Step 1: Login
chloros-cli login user@example.com 'MyP@ssw0rd'

# Step 2: Verify status
chloros-cli status

# Step 3: Process images
chloros-cli process "C:\Datasets\Field_A"

# Step 4: Logout (optional, when switching accounts)
chloros-cli logout
```

***

### 예시 7: 다국어 사용

인터페이스 언어 변경:

```powershell
# List available languages
chloros-cli language --list

# Change to Spanish
chloros-cli language es

# Process with Spanish interface
chloros-cli process "C:\Vuelos\Campo_A"

# Change back to English
chloros-cli language en
```
