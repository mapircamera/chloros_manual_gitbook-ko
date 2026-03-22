# NVIDIA Jetson 가이드

XPROTX000047NVIDIA Jetson에서 실행되는 XPROTX는 현장, 무인항공기(UAV), 원격 설치 환경 등 에지(edge) 환경에서 다중 스펙트럼 이미지 처리를 가능하게 합니다. Chloros는 사용자의 Jetson 모델을 자동으로 감지하고 해당 하드웨어에 맞게 처리 전략을 최적화합니다.

***

## 지원되는 Jetson 모델

| 모델                | RAM            | 처리 전략                                   | 권장 용도                                          |
| -------------------- | -------------- | ----------------------------------------------------- | -------------------------------------------------------- |
| **Jetson AGX Orin**  | 32-64GB 공유 | `GPU_PARALLEL` (4개 워커)                            | 최대 성능, 대용량 데이터셋                      |
| **Jetson Orin NX**   | 8-16GB 공유  | `GPU_PARALLEL` (3개 워커, 16GB) / `GPU_SINGLE` (8GB) | 항공 및 현장 배포를 위한 주요 권장 사양 |
| **Jetson Orin Nano** | 8GB 공유     | `GPU_SINGLE` (1개 워커)                               | 엔트리 레벨 엣지 컴퓨팅                                 |
| **Jetson Nano**      | 4-8GB 공유   | `GPU_SINGLE` (1개 워커)                               | 메모리 제약이 있는 엔트리 레벨                          |

{% hint style="info" %}
**구형 Jetson 모델**(TX2, TX1, Xavier NX)은 지원되지 않을 수 있습니다. 성능은 사용 가능한 GPU 메모리 및 CUDA 기능에 따라 달라질 수 있습니다.
{% endhint %}

***

## 요구 사항

* **JetPack 6.x** (최신 버전 권장)
* **NVIDIA CUDA** (JetPack에 포함됨)
* **Chloros+ 라이선스** (CLI/SDK 액세스 시 필수)

## 설치

```bash
# Install the JetPack 6 .deb package
sudo dpkg -i chloros-arm64-jp6.deb

# Verify installation
chloros-cli --version

# Install Python SDK (optional)
pip install chloros-sdk

# Run system diagnostics
chloros-cli selftest
```

일반적인 Linux 설치 방법은 [Linux 설치](linux-installation.md)를 참조하십시오.

***

## Jetson에서의 동적 컴퓨팅 적응

Chloros는 Jetson 모델을 자동으로 감지하고 최적의 처리 전략을 선택합니다. **수동 조정은 필요하지 않습니다.**

### 작동 방식

시작 시, Chloros는 시스템을 프로파일링합니다:

1. **`/proc/device-tree/model`를 통해 Jetson 모델 감지**

2.**사용 가능한 GPU/공유 메모리 읽기**

3.**처리 전략 선택** (`GPU_PARALLEL`, `GPU_SINGLE` 또는 `CPU_PARALLEL`)
4. **워커 수, 파이프라인 유형 및 메모리 할당**을 자동으로 설정

### 모델별 동작

| Jetson 모델                | 전략       | 워커 수 | 파이프라인                       | 동시 실행 수 |
| --------------------------- | -------------- | ------- | ------------------------------ | ----------- |
| **Jetson Nano 8GB**         | `GPU_SINGLE`   | 1       | `tiled_gpu` (메모리 효율적) | 직렬화됨  |
| **Jetson Orin Nano 8GB**    | `GPU_SINGLE`   | 1       | `tiled_gpu`                    | 직렬화됨  |
| **Jetson Orin NX 8GB**      | `GPU_SINGLE`   | 2       | `tiled_gpu`                    | 직렬화됨  |
| **Jetson Orin NX 16GB**     | `GPU_PARALLEL` | 3       | `fused_gpu` (전체 GPU 경로)    | 동시 실행  |
| **Jetson AGX Orin 32-64GB** | `GPU_PARALLEL` | 4       | `fused_gpu`                    | 동시 실행  |

{% hint style="success" %}
**Jetson Orin NX 16GB**는 엣지 배포에 최적의 선택입니다. 이 모델은 3개의 동시 작업자를 지원하는 `GPU_PARALLEL` 전략을 적용하여, 콤팩트한 폼 팩터에서 진정한 병렬 GPU 처리를 제공합니다.
{% endhint %}

플랫폼 간의 핵심적인 차이점은 **메모리**입니다. 8GB의 공유 메모리를 갖춘 Jetson Nano는 메모리 효율적인 타일링 방식을 사용하여 이미지를 한 번에 하나씩 처리해야 하는 반면, 16GB를 갖춘 Orin NX는 더 높은 처리량을 제공하는 융합 파이프라인을 사용하여 GPU를 통해 3개의 이미지를 동시에 처리할 수 있습니다.

전체 컴퓨트 적응에 대한 자세한 내용은 [동적 컴퓨트 적응](../processing-architecture/dynamic-compute-adaptation.md)을 참조하십시오.

***

## 열 관리

Jetson 장치는 특히 밀폐된 환경이나 항공기 탑재 환경에서 열 여유 공간이 제한적입니다. Chloros에는 자동 열 모니터링 및 스로틀링 기능이 포함되어 있습니다:

| 온도         | 조치                                            |
| ------------------- | ------------------------------------------------- |
| **&lt; 70°C**          | 정상 작동 — 최대 처리 속도          |
| **70°C** (경고)  | 배치 크기 자동 감소                   |
| **80°C** (위험) | 강력한 스로틀링 — 동시 실행 수 감소         |
| **90°C** (종료) | GPU 처리 완전히 중지 — 냉각 필요 |

{% hint style="warning" %}
지속적인 처리를 위해, 특히 밀폐된 현장 인클로저나 항공 시스템에서는 **적절한 환기 및 방열**을 확보하십시오. 열 스로틀링은 하드웨어를 보호하기 위해 처리 처리량을 줄입니다.
{% endhint %}

***

## 메모리 관리

Jetson 장치는 **통합 메모리**를 사용합니다. 즉, GPU와 CPU가 동일한 물리적 RAM을 공유합니다. 이는 표시된 VRAM(예: Orin NX 16GB의 경우 15.3GB)이 전용 GPU 메모리가 아니며, 운영 체제 및 다른 프로세스와 공유된다는 것을 의미합니다.

### 스왑 권장 사항

대용량 데이터셋이나 텍스처 인식 디베이어(Texture Aware debayer) 처리를 수행하는 경우, Chloros에서 스왑 공간 생성을 권장할 수 있습니다:

```bash
# Check current memory and swap
free -h

# Create a swap file (example: 8GB)
sudo fallocate -l 8G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Make persistent across reboots
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

**이미지당 예상 메모리 사용량:**

* 표준 디베이어: 이미지당 약 10MB
* 텍스처 인식 디베이어: 이미지당 약 15MB

Chloros는 데이터셋 크기를 기반으로 필요한 메모리를 자동으로 계산하며, 스왑 공간이 권장될 경우 경고합니다.

### OOM(메모리 부족) 대체 처리

처리 중 메모리 부족 상태가 감지되면:

1. Chloros는 GPU 워커 수를 자동으로 줄입니다
2. `fused_gpu`에서 `tiled_gpu` 파이프라인으로 전환(메모리 효율성 향상)
3. 시스템 중단 대신 처리량을 줄여 작업을 계속 진행

***

## 현장 배포

### 전력 고려 사항

| Jetson 모델     | 일반적인 전력 소비량 | 참고 사항                   |
| ---------------- | ------------------ | ----------------------- |
| Jetson Nano      | 5-10W              | USB-C 또는 배럴 잭    |
| Jetson 모델     | 일반적인 전력 소비 | 비고                   || ---------------- | ------------------ | ----------------------- |
| Jetson Nano      | 5-10W              | USB-C 또는 배럴 잭    |
| Jetson Orin Nano | 7-15W              | DC 배럴 잭          |
| Jetson Orin NX   | 10-25W             | DC 배럴 잭          |
| Jetson AGX Orin  | 15-60W             | USB-C PD 또는 배럴 잭 |

지속적인 처리를 위한 전력 예산을 계획하십시오 — 최대 전력 소비는 GPU 집약적인 스레드 3(처리)에서 발생합니다.

### 스토리지 권장 사항

* **NVMe SSD**는 arm64 배포 시 강력히 권장됩니다.
* SD 카드는 처리 속도가 너무 느리므로 부팅 미디어로만 사용하십시오.
* 처리된 출력물을 위해 원본 이미지 데이터 크기의 2~3배를 확보하십시오.

### SSH를 통한 헤드리스 운영

Chloros CLI는 헤드리스 Jetson 배포에 이상적입니다:

```bash
# SSH into the Jetson
ssh user@jetson-hostname

# Process a dataset
chloros-cli process /data/datasets/flight001 --format tiff-32

# Monitor export progress
chloros-cli export-status
```

### systemd를 이용한 자동 처리

자동 처리를 위한 systemd 서비스를 생성하십시오:

```ini
# /etc/systemd/system/chloros-process.service
[Unit]
Description=Chloros Automated Processing
After=network.target

[Service]
Type=oneshot
User=chloros
ExecStart=/usr/bin/chloros-cli process /data/incoming --output /data/processed
StandardOutput=append:/var/log/chloros-process.log
StandardError=append:/var/log/chloros-process.log

[Install]
WantedBy=multi-user.target
```

예약 처리를 위해 systemd 타이머와 연동하세요:

```ini
# /etc/systemd/system/chloros-process.timer
[Unit]
Description=Run Chloros Processing Every Hour

[Timer]
OnCalendar=hourly
Persistent=true

[Install]
WantedBy=timers.target
```

```bash
sudo systemctl enable chloros-process.timer
sudo systemctl start chloros-process.timer
```

***

## 워크플로 예시

### 기본 Jetson 처리

```bash
#!/bin/bash
# Process a drone flight dataset on Jetson
chloros-cli process /data/flights/flight_042 \
    --output /data/processed/flight_042 \
    --format tiff-32 \
    --indices NDVI NDRE GNDVI
```

### Jetson에서의 Python 및 SDK

```python
from chloros_sdk import ChlorosLocal

with ChlorosLocal() as chloros:
    chloros.create_project("field_survey_042")
    chloros.import_images("/data/flights/flight_042")
    chloros.configure(
        indices=["NDVI", "NDRE", "GNDVI"],
        export_format="TIFF (32-bit, Percent)",
        reflectance_calibration=True
    )
    chloros.process(mode="parallel")

print("Processing complete!")
```

### 여러 비행 데이터 일괄 처리

```bash
#!/bin/bash
# Process all flight datasets in a directory
for flight in /data/flights/*/; do
    name=$(basename "$flight")
    echo "Processing $name..."
    chloros-cli process "$flight" \
        --output "/data/processed/$name" \
        --format tiff-32 \
        --indices NDVI NDRE
    echo "Completed $name"
done
```

***

## 현장 사용을 위한 권장 Jetson 시스템

현장 및 항공기 탑재용으로는 다음 Jetson Orin NX 16GB 캐리어 보드 옵션을 고려해 보십시오:

* **항공기/드론**: 진동 등급(MIL-STD)을 충족하고, 경량(300g 미만)이며, 수동 냉각 방식인 시스템
* **견고한 현장**: IP67/IP69K 방수 인클로저와 PoE GigE 카메라 연결 기능을 갖춘 시스템
* **최소 구성/저가형**: 추가 인클로저가 포함된 개발자 키트

배포 시나리오에 맞는 구체적인 하드웨어 권장 사항이 필요하시면 [MAPIR 지원팀](https://www.mapir.camera/community/contact)에 문의하십시오.

***

## 다음 단계

* [Linux 설치](linux-installation.md) — 일반적인 Linux 설치 세부 정보
* [동적 컴퓨팅 적응](../processing-architecture/dynamic-compute-adaptation.md) — 전체 컴퓨팅 전략 참조
* [처리 파이프라인](../processing-architecture/processing-pipeline.md) — 4스레드 파이프라인 이해
* [CLI : 명령줄](../CLI.md) — 전체 CLI 참조
* [API : Python SDK](../api-python-sdk.md) — SDK 전체 참조
