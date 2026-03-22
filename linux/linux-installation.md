# Linux 설치

Chloros는 Linux용으로 배포되며, CLI와 백엔드를 설치하는 `.deb` 패키지 형태로 제공됩니다. Python와 SDK는 pip를 통해 별도로 설치됩니다.

***

## Linux amd64 (x86_64)

### 시스템 요구 사항

| 요구 사항 | 최소 | 권장 |
| --- | --- | --- |
| **배포판** | Ubuntu 20.04 이상 / Debian 11 이상 | Ubuntu 22.04 이상 |
| **프로세서** | x86_64 (Intel/AMD) | Intel Core i7 이상 |
| **메모리 (RAM)** | 8GB | 16GB 이상 |
| **그래픽 카드** | 없음 (CPU 처리) | 4GB 이상 VRAM을 갖춘 NVIDIA GPU |
| **저장 공간** | 2GB 여유 공간 | 10GB 이상 여유 공간이 있는 SSD |
| **Python** | Python 3.7 이상 (SDK용) | Python 3.10 이상 |

### 설치

`.deb` 패키지를 다운로드하여 설치하십시오:

```bash
sudo dpkg -i chloros-amd64.deb
```

설치를 확인하십시오:

```bash
chloros-cli --version
```

***

## Linux arm64 (NVIDIA Jetson)

### 시스템 요구 사항

| 요구 사항 | 최소 | 권장 |
| --- | --- | --- |
| **플랫폼** | JetPack 6이 설치된 NVIDIA Jetson | Jetson Orin NX 16GB 또는 AGX Orin |
| **JetPack** | JetPack 6.x | 최신 JetPack 6 |
| **메모리 (RAM)** | 8GB (GPU/CPU 공유) | 16GB 이상 공유 |
| **저장 공간** | 2GB 여유 공간 | 10GB 이상 여유 공간이 있는 NVMe SSD |
| **Python** | Python 3.7+ (SDK용) | Python 3.10 이상 |

### 설치

JetPack 6 `.deb` 패키지를 다운로드하여 설치하십시오:

```bash
sudo dpkg -i chloros-arm64-jp6.deb
```

설치 여부를 확인하십시오:

```bash
chloros-cli --version
```

열 관리 및 현장 배포를 포함한 자세한 Jetson 설정 방법은 [NVIDIA Jetson 가이드](nvidia-jetson-guide.md)를 참조하십시오.

***

## Python SDK 설치 (모든 Linux)

Python SDK는 pip를 통해 별도로 설치되며 amd64 및 arm64 모두에서 작동합니다:

```bash
pip install chloros-sdk
```

선택적 진행 상황 스트리밍 지원을 포함하려면:

```bash
pip install chloros-sdk[progress]
```

SDK를 확인하십시오:

```bash
python -c "import chloros_sdk; print(chloros_sdk.__version__)"
```

{% hint style="info" %}
`.deb` 패키지는 Chloros, CLI 및 백엔드를 설치합니다. Python SDK는 로컬 HTTP API를 통해 백엔드와 통신하는 별도의 pip 패키지입니다.
{% endhint %}

***

## 구성 디렉터리

Chloros에서 Linux는 [XDG 기본 디렉터리 사양](https://specifications.freedesktop.org/basedir-spec/basedir-spec-latest.html)을 따릅니다:

| 용도 | Linux 경로 | Windows 대응 항목 |
| --- | --- | --- |
| **구성** | `~/.config/chloros/` | `%APPDATA%\Chloros\` |
| **데이터 / 프로젝트** | `~/.local/share/chloros/` | `%LOCALAPPDATA%\Chloros\` |
| **캐시 / 자격 증명** | `~/.cache/chloros/` | `%APPDATA%\Chloros\cache\` |

## 백엔드 실행 파일 위치

`.deb` 패키지는 백엔드를 표준 위치에 설치합니다. CLI 및 SDK는 백엔드 경로를 자동으로 감지합니다:

| 설치 방법 | 백엔드 경로 |
| --- | --- |
| `.deb` 패키지 | `/usr/lib/chloros/chloros-backend` |
| 수동/사용자 지정 | `/opt/mapir/chloros/backend/chloros-backend` |

`--backend-exe` 및 CLI 플래그나 `backend_exe` 및 SDK 생성자 매개변수를 사용하여 백엔드 경로를 재정의할 수 있습니다.

***

## 초기 설정

### 1. 라이선스 활성화

Chloros+ 라이선스는 CLI 및 SDK에 액세스하는 데 필요합니다:

```bash
chloros-cli login your@email.com 'your-password'
```

### 2. 라이선스 상태 확인

```bash
chloros-cli status
```

### 3. 첫 번째 데이터셋 처리

```bash
chloros-cli process ~/datasets/flight001
```

### 4. 시스템 진단 실행

시스템이 올바르게 구성되었는지 확인하십시오:

```bash
chloros-cli selftest
```

이 작업은 버전, 백엔드 시작, API 연결 상태, CUDA/GPU 가용성 등 7가지 진단 검사를 수행합니다.

***

## Bash 스크립트 예시

### 여러 데이터셋 처리

```bash
#!/bin/bash
for dataset in ~/datasets/2026/*/; do
    echo "Processing $(basename "$dataset")..."
    chloros-cli process "$dataset" --format tiff-32
    echo "Done: $(basename "$dataset")"
done
```

### 사용자 지정 설정으로 처리

```bash
#!/bin/bash
chloros-cli process ~/datasets/field_a \
    --output ~/output/field_a \
    --format tiff-32 \
    --indices NDVI NDRE GNDVI \
    --debayer texture-aware \
    --no-vignette
```

### Cron을 이용한 자동 처리

새로운 데이터셋을 자동으로 처리하려면 crontab에 다음을 추가하십시오(`crontab -e`):

```cron
# Process any new datasets at 2 AM daily
0 2 * ** /usr/bin/chloros-cli process /data/incoming --output /data/processed >> /var/log/chloros.log 2>&1
```

### Python SDK 예시

```python
from chloros_sdk import process_folder

# One-line processing
result = process_folder(
    "/home/user/datasets/flight001",
    indices=["NDVI", "NDRE"],
    export_format="TIFF (32-bit, Percent)"
)
```

***

## 문제 해결

### CLI 설치 후 파일 미검출

`.deb` 패키지 설치 후 `chloros-cli` 파일이 발견되지 않는 경우:

```bash
# Check if the binary exists
which chloros-cli
ls -la /usr/bin/chloros-cli

# If not in PATH, check the installation
dpkg -L chloros-amd64  # or chloros-arm64-jp6

# Reload your shell
source ~/.bashrc
```

### 권한 거부

```bash
# Ensure the binary is executable
sudo chmod +x /usr/bin/chloros-cli
sudo chmod +x /usr/lib/chloros/chloros-backend
```

### 백엔드 시작 실패

```bash
# Check if port 5000 is already in use
lsof -i :5000

# Kill any existing process on port 5000
kill $(lsof -t -i :5000)

# Try starting with a different port
chloros-cli --port 5001 process ~/datasets/flight001
```

### CUDA 감지되지 않음

```bash
# Check NVIDIA driver installation
nvidia-smi

# Check CUDA availability
nvcc --version

# On Jetson, check JetPack version
cat /etc/nv_tegra_release
```

### 공유 라이브러리 누락

```bash
# Install common dependencies
sudo apt-get update
sudo apt-get install -f

# Check for missing libraries
ldd /usr/lib/chloros/chloros-backend | grep "not found"
```

***

## Linux에서 Chloros 업데이트

내장된 업데이트 명령어를 사용하여 업데이트를 확인하고 설치하십시오:

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

***

## 다음 단계

* [NVIDIA Jetson 가이드](nvidia-jetson-guide.md) — Jetson 전용 최적화 및 배포
* [CLI : 명령줄](../CLI.md) — 전체 CLI 명령어 참조
* [API : Python SDK](../api-python-sdk.md) — 전체 SDK 참조
* [동적 컴퓨팅 적응](../processing-architecture/dynamic-compute-adaptation.md) — Chloros가 하드웨어에 어떻게 적응하는지
