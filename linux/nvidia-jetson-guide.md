# NVIDIA Jetson 가이드

NVIDIA Jetson용 Chloros는 현장, 무인항공기(UAV), 원격 설치 환경 등 엣지 환경에서 다중 스펙트럼 이미지 처리를 가능하게 합니다. Chloros 1.2.0은 시작 시 Jetson 모델을 감지하고, 감지된 하드웨어에 맞게 처리 전략을 최적화합니다. **수동으로 조정할 필요가 없습니다.**

***

## 지원되는 Jetson 모델

| 모델                | RAM            | 처리 전략                                     | 권장 용도                                          |
| -------------------- | -------------- | ------------------------------------------------------- | -------------------------------------------------------- |
| **Jetson AGX Orin**  | 32-64GB 공유 | `GPU_PARALLEL` (작업자 2명)                              | 최대 성능, 대용량 데이터셋                      |
| **Jetson Orin NX**   | 8-16GB 공유  | `GPU_PARALLEL` (작업자 2명, 16GB) / `GPU_SINGLE` (8GB)   | 항공 및 현장 배포를 위한 주요 권장 사양 |
| **Jetson Orin Nano** | 8GB 공유     | `GPU_SINGLE` (워커 1개, 순차 처리)                     | 엔트리 레벨 엣지 컴퓨팅                                 |

{% hint style="info" %}
Linux arm64 패키지는 **JetPack 6**이 필요하며, 이는 Jetson Orin 제품군에서 사용할 수 있습니다. 구형 모델(Nano, TX2, Xavier NX)은 JetPack 6을 실행할 수 없으며, 현재 패키지에서 지원되지 않습니다.
{% endhint %}

***

## 요구 사항

* **JetPack 6.x** (최신 버전 권장)
* **NVIDIA CUDA** (JetPack에 포함)
* **유료 Chloros+ 요금제** — Copper 등급 이상 (모든 CLI/SDK 액세스에 필수이며, 서버 측에서 강제 적용됨)

## 설치

```bash
# Install the JetPack 6 .deb package
sudo dpkg -i chloros_1.2.0_arm64_jp6.deb
sudo apt-get install -f

# Verify installation
chloros-cli --version    # prints "Chloros CLI 1.2.0"

# Install Python SDK (optional) — the bundled wheel always matches this build
pip install --user /usr/lib/chloros/sdk/chloros_sdk-*.whl

# Run system diagnostics
chloros-cli selftest
```

일반적인 Linux 설치 세부 정보, 파일 위치 및 문제 해결 방법은 [Linux 설치](linux-installation.md)를 참조하십시오.

{% hint style="info" %}
**압축 해제 디렉터리를 고속 저장 장치에 배치하십시오.** 컴파일된 바이너리는 실행할 때마다 임시 디렉터리로 자동으로 압축을 풀게 되며 — SD 카드에서는 이 과정이 매우 느립니다. Chloros는 `/mnt/ssd/tmp`가 존재할 경우 이를 자동으로 사용하며, 그렇지 않은 경우 `TMPDIR`를 NVMe의 경로로 설정하십시오 (`export TMPDIR=/mnt/nvme/tmp`)의 경로로 설정하십시오.
{% endhint %}

***

## Jetson에서의 동적 컴퓨팅 적응

### 작동 원리

시작 시, Chloros는 시스템 프로파일링을 수행합니다:

1. `/proc/device-tree/model`를 통해 **Jetson 모델 감지**

2.**사용 가능한 공유 GPU/CPU 메모리 읽기** (Jetson은 통합 메모리를 사용합니다)
3. **처리 전략 선택** (`GPU_PARALLEL`, `GPU_SINGLE` 또는 `CPU_PARALLEL`)
4. **워커 수, 파이프라인 유형 및 메모리 할당**을 자동으로 설정합니다

이 결정은 모델 이름이 아닌 **총 공유 RAM 용량**에 따라 이루어집니다:

* **총 RAM 12GB 미만**(모든 8GB Jetson): `GPU_SINGLE`,**워커 1개 — 의도적인 순차 처리**. 동시 실행되는 워커를 처리하기에는 메모리가 부족하므로, 이미지는 한 번에 하나씩 처리됩니다.**8GB 이하**의 Jetson 모델에서는 스레드 3이 워커 풀을 완전히 건너뛰고 이미지별 작업을 인-프로세스(in-process) 방식으로 실행합니다.
* **12GB 이상**(Orin NX 16GB, AGX Orin): 통합 메모리 덕분에 `GPU_PARALLEL`를 사용할 수 있지만, 하지만**Jetson에서는 워커 수가 2개로 제한됩니다** — GPU, 워커 프로세스의 RAM, 그리고 워커별 CUDA 컨텍스트가 모두 동일한 공유 풀을 사용하기 때문에, 워커 수가 더 많을수록 메모리 부족 오류가 발생할 위험이 있습니다.

`CHLOROS_STRATEGY` 환경 변수를 사용하여 자동 선택 설정을 재정의할 수 있습니다 — [동적 컴퓨팅 적응](../processing-architecture/dynamic-compute-adaptation.md#manual-strategy-override)을 참조하십시오.

### 모델별 동작

| Jetson 모델                | 전략       | 워커 수 | 실행                                      |
| --------------------------- | -------------- | ------- | ---------------------------------------------- |
| **Jetson Orin Nano 8GB**    | `GPU_SINGLE`   | 1       | 메모리 부하가 걸린 상태에서 순차적 인-프로세스 루프 (메모리 부하 시 `tiled_gpu`) |
| **Jetson Orin NX 8GB**      | `GPU_SINGLE`   | 1       | 순차적 인-프로세스 루프                     |
| **Jetson Orin NX 16GB**     | `GPU_PARALLEL` | 2       | 동시 실행되는 작업자 프로세스, `fused_gpu` 경로  |
| **Jetson AGX Orin 32-64GB** | `GPU_PARALLEL` | 2       | 동시 실행되는 작업자 프로세스, `fused_gpu` 경로  |

플랫폼 간의 핵심 차이점은 **메모리**입니다. 8GB Jetson은 부하가 높을 때 메모리 효율적인 타일링 방식을 사용하여 이미지를 한 번에 하나씩 처리해야 하는 반면, 16GB 이상의 Orin은 더 높은 처리량을 제공하는 융합 파이프라인을 사용하여 GPU를 통해 2개의 이미지를 동시에 처리할 수 있습니다.

### 모델별 GPU 할당량

각 Jetson 모델에는 공유 풀에서 처리 작업이 차지할 수 있는 양을 제한하고 배치 크기를 조정하는 하드웨어 프로필이 포함되어 있습니다:

| 모델 | GPU 할당량 상한 | 배치 크기 배율 | 시스템/디스플레이 전용 할당량 |
| --- | --- | --- | --- |
| **Jetson Orin Nano** | 70% | ×0.8 | 2.0 GB |
| **Jetson Orin NX** | 75% | ×1.0 | 3.0 GB |
| **Jetson AGX Orin** | 80% | ×1.5 | 4.0 GB |

감지된 RAM 용량에 따라 프로필이 조정됩니다. **16GB 이상**을 보고하는 Jetson의 경우 배치 배율이 ×1.2로 증가합니다. 배율을 적용하기 전의 기본 배치 크기는 8개의 이미지입니다.

전체 컴퓨트 적응 참조 내용은 [동적 컴퓨트 적응](../processing-architecture/dynamic-compute-adaptation.md)을 참조하십시오.

***

## Nano 및 Orin Nano에서 Texture Aware에 대한 GPU 주파수 제한

Texture Aware 디베이어는 GPU 신경망 추론을 실행하며, 이는 저전력 Jetson 모델(10-15W급)에서 GPU 클럭 속도가 최대일 때 **과전류 경고**를 유발할 수 있습니다.**Jetson Nano 또는 Orin Nano**에서 Texture Aware 처리를 수행하기 전에, Chloros는 GPU의 최대 주파수를 확인하고, 현재 주파수가 이보다 높을 경우**510 MHz**(510000000)로 제한합니다:

* CLI가 GPU 주파수 sysfs 노드에 쓰기 권한이 있는 경우, 제한이 **자동으로 적용**되고 확인 메시지가 출력됩니다.
* 그렇지 않은 경우(루트 권한 필요), CLI는 제한을 수동으로 적용하기 위한 정확한 `sudo` 명령어를 출력하고, 사용자가 이를 읽을 수 있도록 잠시 대기한 후 계속 진행합니다. 이 경우 처리는 계속 실행되지만 과전류 경고가 표시될 수 있습니다.

처리 전에 직접 상한값을 적용하려면:

```bash
echo 510000000 | sudo tee /sys/devices/platform/bus@0/17000000.gpu/devfreq/17000000.gpu/max_freq
```

고출력 모델(Orin NX 25W, AGX Orin 60W)은 GPU 최대 속도로 실행되며, 상한값이 적용되지 않습니다. Standard 디베이어는 어떤 모델에서도 상한값을 트리거하지 않습니다.

{% hint style="info" %}
**Jetson에서 Texture Aware는 항상 한 번에 하나의 이미지만 처리합니다.** 각 워커는 자체 CUDA 컨텍스트(~1GB)와 노이즈 제거 모델의 별도 사본이 필요하지만, 통합 메모리로는 이를 감당할 수 없습니다. 따라서 Jetson에서 Texture Aware 경로는 단일 워커에 고정되며 GPU 액세스가 직렬화됩니다. 모든 Jetson에서 Texture Aware는 Standard보다 현저히 느릴 것으로 예상됩니다.
{% endhint %}

***

## 열 관리

Jetson 장치는 특히 밀폐된 공간이나 항공기 내 배치 환경에서 열 여유 공간이 제한적입니다. Chloros는 SoC 온도를 모니터링하고 배치 크기를 자동으로 조절합니다:

| 온도         | 조치                                            |
| ------------------- | ------------------------------------------------- |
| **&lt; 70°C**          | 정상 작동 — 최대 처리 속도          |
| **70°C** (경고)  | 배치 크기가 점진적으로 축소됨 (70°C~80°C 구간에서 100% → 50%) |
| **80°C** (위험) | 강력한 처리 속도 제한 (80°C에서 90°C 사이에서 50% → 0%) |
| **90°C** (종료) | GPU 처리 완전히 중단 — 냉각 필요 |

{% hint style="warning" %}
지속적인 처리를 위해서는, 특히 밀폐된 현장 인클로저나 항공기 탑재 시스템에서 **충분한 환기 및 방열을 확보**하십시오. 열 스로틀링은 하드웨어를 보호하기 위해 처리 처리량을 감소시킵니다.
{% endhint %}

***

## 메모리 관리

Jetson 장치는 **통합 메모리**를 사용합니다. 즉, GPU와 CPU가 동일한 물리적 RAM을 공유합니다. 표시되는 VRAM(예: Orin NX 16GB의 경우 ~15.3GB)은 전용 GPU 메모리가 아니며, 운영 체제 및 기타 모든 프로세스가 사용하는 것과 동일한 RAM입니다.

### 스왑 경고 및 권장 사항

Jetson에서 처리를 시작하기 전에, CLI는 입력 폴더에 있는 RAW 이미지 수를 집계합니다(`.tif`, `.tiff`, `.raw`, `.dng` — JPG 미리보기는 집계되지 않음)의 개수를 계산하고, 실행에 필요한 최대 메모리 용량을 추정한 후, RAM + 스왑 용량이 부족할 것으로 예상될 경우 **시작 전에 경고**를 표시합니다. 경고 메시지의 제목은 `LOW MEMORY WARNING - Jetson Detected`이며, 이미지 수, RAM, 현재 스왑 공간, 예상 최대 메모리 사용량을 표시한 후, 프로젝트 규모에 맞춰 설정된 정확한 `fallocate` / `chmod` / `mkswap` / `swapon` 명령어를 표시합니다(8GB 미만인 경우는 없음). 메시지가 스크롤백에 묻히지 않도록 몇 초간 일시 정지한 후 처리를 계속합니다.**경고에서 사용하는 메모리 추정값:**

| 디베이어 모드 | 기본값 | 이미지당 |
| --- | --- | --- |
| 표준 | ~1.5 GB | ~10 MB |
| 텍스처 인식 | ~2.5 GB (모델 + Python 런타임) | ~15 MB |

예상 피크 값이 RAM + 스왑 용량에서 1GB의 안전 여유분을 뺀 값을 초과할 때 경고가 발생하며, 이때 **파일 기반** 스왑만 계산됩니다. zram만 사용하는 설정인 경우에도 여전히 경고가 표시됩니다.

스왑을 수동으로 추가하려면 (예: 8GB):



<!-- SCREENSHOT-NEEDED: Terminal on a Jetson Orin (SSH session) showing the full "LOW MEMORY WARNING - Jetson Detected" block printed by `chloros-cli process` on a large folder: the image count and debayer mode line, RAM / current swap / estimated peak figures, and the fallocate/chmod/mkswap/swapon command block it recommends -->

```bash
# Check current memory and swap
free -h

# Create a swap file
sudo fallocate -l 8G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Make persistent across reboots
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```### OOM(메모리 부족) 처리

처리 중, Chloros는 GPU 메모리를 모니터링하며, 크래시 대신 점진적으로 성능을 저하시킵니다:

1. GPU 메모리 사용률이 **85%**를 초과하면, 선제적으로 배치 크기가 축소됩니다.
2. 메모리 부족 현상이 여전히 발생하면 배치 크기가 **절반으로** 줄어들며, 이후 메모리 부족이 발생할 때마다 다시 절반으로 줄어듭니다. 이후 성공적으로 처리된 각 배치마다 해당 페널티가 한 단계씩 완화됩니다.
3. 지속적인 부하가 가해지면 파이프라인은 `fused_gpu`에서 메모리 효율이 높은 `tiled_gpu` 경로로 전환되며, 최후의 수단으로 CPU 처리로 전환됩니다.

***

## 현장 배포

### 전력 고려 사항

| Jetson 모델     | 일반적인 전력 소비량 | 비고                   |
| ---------------- | ------------------ | ----------------------- |
| Jetson Orin Nano | 7-15W              | DC 배럴 잭          |
| Jetson Orin NX   | 10-25W             | DC 배럴 잭          |
| Jetson AGX Orin  | 15-60W             | USB-C PD 또는 배럴 잭 |

지속적인 처리를 위한 전력 예산을 계획하십시오. GPU 사용량이 많은 스레드 3(Processing) 실행 중에 최대 전력 소모가 발생합니다.

### 스토리지 권장 사항

* **NVMe SSD**는 arm64 배포 시 강력히 권장됩니다.
* SD 카드는 처리 속도가 너무 느리므로 부팅 용도로만 사용하십시오
* 처리된 출력 데이터를 위해 원본 이미지 데이터 크기의 2~3배를 확보하십시오

### SSH를 통한 헤드리스 작동

Chloros 및 CLI는 헤드리스 Jetson 배포에 이상적입니다:

```bash
# SSH into the Jetson
ssh user@jetson-hostname

# Process a dataset
chloros-cli process /data/datasets/flight001 --format "TIFF (32-bit, Percent)"

# Monitor export progress
chloros-cli export-status
```

### LATTICE / DAQ-E 시간 동기화를 위한 상시 가동 백엔드

Jetson이 LATTICE 카메라나 DAQ-E 광 센서를 헤드리스 모드로 제어하는 경우, PTP 그랜드마스터가 지속적으로 실행되도록 백엔드 systemd 서비스를 활성화하십시오(이 유닛은 설치되어 있지만 기본적으로 활성화되어 있지 않습니다):

```bash
sudo systemctl enable --now chloros-backend.service
chloros-cli time-sync status
```

패키지가 루트 권한 없이 PTP 포트 319/320을 바인딩 가능하게 만드는 방법을 포함한 자세한 내용은 [Linux 설치](linux-installation.md#always-on-ptp-for-headless-hosts)를 참조하십시오.

### systemd를 이용한 자동 처리

자동 처리를 위한 systemd 서비스를 생성합니다:

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

`chloros-cli process`는 제품을 요청한 실행에서 이미지가 기록되지 않을 경우 0이 아닌 종료 코드를 반환하므로, 모니터링 시 systemd의 오류 상태가 유용합니다.

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

## 워크플로우 예시

### 기본 Jetson 처리

```bash
#!/bin/bash
# Process a drone flight dataset on Jetson
chloros-cli process /data/flights/flight_042 \
    --output /data/processed/flight_042 \
    --format "TIFF (32-bit, Percent)" \
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
        --format "TIFF (32-bit, Percent)" \
        --indices NDVI NDRE
    echo "Completed $name"
done
```

***

## 현장 사용을 위한 권장 Jetson 시스템

현장 및 항공기 탑재용 배포의 경우, 다음 Jetson Orin NX 16GB 캐리어 보드 옵션을 고려해 보십시오:

* **항공/드론**: 진동 등급(MIL-STD)을 충족하고, 경량(300g 미만)이며, 수동 냉각 방식을 갖춘 시스템
* **견고한 현장**: IP67/IP69K 방수 인클로저와 PoE GigE 카메라 연결 기능을 갖춘 시스템
* **최소 구성/저비용**: 추가 인클로저가 포함된 개발자 키트

구체적인 배포 시나리오에 맞는 하드웨어 권장 사항이 필요하시면 [MAPIR 지원팀](https://www.mapir.camera/community/contact)에 문의하십시오.

***

## 다음 단계

* [Linux 설치](linux-installation.md) — Linux 설치에 대한 일반적인 세부 정보
* [동적 컴퓨팅 적응](../processing-architecture/dynamic-compute-adaptation.md) — 전체 컴퓨팅 전략 참조
* [처리 파이프라인](../processing-architecture/processing-pipeline.md) — 4스레드 파이프라인 이해
* [CLI : 명령줄](../CLI.md) — CLI 가이드
* [API : Python SDK](../api-python-sdk.md) — SDK 가이드
* [CLI 참조](../reference/cli-reference.md) 및 [SDK 참조](../reference/sdk-reference.md) — 1.2.0 버전에 대한 상세한 명령어/API 목록
