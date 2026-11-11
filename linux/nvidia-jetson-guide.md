# NVIDIA Jetson 가이드 NVIDIA Jetson에서 실행되는

Chloros
는 현장, 무인항공기(UAV), 원격 설치 환경 등 엣지 환경에서 다중 스펙트럼 이미지 처리를 지원합니다. Chloros
1.3.0은 시작 시 Jetson 모델을 자동으로 감지하고, 감지된 하드웨어에 맞춰 처리 전략을 최적화합니다. **수동으로 조정할 필요가 없습니다.**
{% hint style="info" %}
**

1.3.0 Jetson 패키지는 Jetson의 GPU에서 처리를 수행합니다.** `chloros_1.3.0_arm64_jp6.deb`는 JetPack 6용으로 빌드되었으며 JetPack 6의 CUDA 기반 PyTorch 빌드를 사용하므로, 포함된 Texture Aware 디베이어를 포함한 모든 처리가 Orin의 GPU에서 실행됩니다. 이 패키지는 CUDA 자체를 포함하지 않으며, JetPack 6이 설치하는 CUDA 및 cuDNN 라이브러리를 사용합니다(해당 라이브러리가 없는 이미지의 경우 `sudo apt install nvidia-jetpack`). CUDA GPU를 사용할 수 없는 경우, 처리는 CPU에서 실행되며 Texture Aware는 Standard 디베이어를 사용합니다.
{% endhint %}

***

## 지원되는 Jetson 모델

| 모델                | RAM            | 처리 전략                                     | 권장 용도                                          |
| -------------------- | -------------- | ------------------------------------------------------- | -------------------------------------------------------- |
| **Jetson AGX Orin**  | 32-64GB 공유 | `GPU_PARALLEL` (작업자 2명)                              | 최대 성능, 대용량 데이터셋                      |
| **Jetson Orin NX**   | 8-16GB 공유  | `GPU_PARALLEL` (워커 2개, 16GB) / `GPU_SINGLE` (8GB)   | 현장 배포를 위한 주요 권장 사항              |
| **Jetson Orin Nano** | 8GB 공유     | `GPU_SINGLE` (작업자 1명, 순차 처리)                     | 엔트리 레벨 엣지 컴퓨팅                                 |

{% hint style="info" %}
Linux
arm64 패키지에는 **JetPack 6**이 필요하며, 이는 Jetson Orin 제품군에서 사용할 수 있습니다. 구형 모델(Nano, TX2, Xavier NX)은 JetPack 6을 실행할 수 없으며, 현재 패키지에서 지원되지 않습니다.
{% endhint %}

***

## 요구 사항

* **JetPack 6.x** (최신 버전 권장)
* JetPack 6의 **NVIDIA CUDA 및 cuDNN** (이미지에 포함되어 있지 않은 경우 `sudo apt install nvidia-jetpack`)
* **유료 Chloros
+ 요금제** — Copper 등급 이상 (모든 CLI
/SDK
액세스에 필수; 서버 측에서 강제 적용됨)

## 설치

```bash
# Install the arm64 .deb package (JetPack 6) -- apt, not dpkg -i, so the
# Python SDK's requirements (python3-numpy, python3-requests) come too
sudo apt install ./chloros_1.3.0_arm64_jp6.deb

# Verify installation
chloros-cli --version    # prints "Chloros CLI 1.3.0"
python3 -c "import chloros_sdk; print(chloros_sdk.__version__)"   # the .deb installs the SDK

# Run system diagnostics
chloros-cli selftest
```

`.deb`는 이 빌드와 호환되는 시스템 `python3`용 Python
SDK
를 설치합니다. 번들로 제공되는 휠(wheel)을 설치해야 하는 것은 가상 환경(virtualenv)뿐입니다: `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl` (완전 오프라인). 일반적인 Linux
설치 세부 정보, 파일 위치 및 문제 해결 방법은 [Linux
설치](linux-installation.md)를 참조하십시오.

{% hint style="info" %}
**실행 시 더 이상 압축을 풀지 않습니다.**

1.3.0 버전부터 백엔드와 CLI
는 `/usr/lib/chloros/` 아래에 압축을 풀지 않은 상태로 설치되므로, 더 이상 시작할 때마다 임시 디렉터리로 파일을 추출하지 않습니다. 해당 폴더가 존재할 경우 임시 파일은 여전히 자동으로 `/mnt/ssd/tmp`에 저장되며, 그렇지 않은 경우 `TMPDIR`를 NVMe(`export TMPDIR=/mnt/nvme/tmp`)의 경로로 설정할 수 있습니다.
{% endhint %}

***

## Jetson에서의 동적 컴퓨팅 적응

### 작동 원리

시작 시, Chloros
는 시스템 프로파일을 생성합니다:

1. `/proc/device-tree/model`를 통해 **Jetson 모델을 감지**합니다.
2. **사용 가능한 공유 GPU/CPU 메모리를 읽습니다** (Jetson은 통합 메모리를 사용합니다)
3. **PyTorch에 CUDA 사용 가능 여부를 확인합니다** — JetPack 6의 CUDA 및 cuDNN이 설치되어 있다면 사용 가능합니다
4. **처리 전략**(`GPU_PARALLEL`, `GPU_SINGLE` 또는 `CPU_PARALLEL`)과 워커 수를 자동으로 선택합니다

이 결정은 모델 이름이 아닌 **총 공유 RAM 용량**에 따라 이루어집니다:

* **총 RAM 12GB 미만**(모든 8GB Jetson):**워커 1개 — 의도적인 순차 처리**. 메모리 용량이 부족하여 동시 워커를 사용할 수 없으므로, 이미지를 한 번에 하나씩 처리합니다.**8GB 이하**인 Jetson의 경우, 스레드 3은 워커 풀을 완전히 건너뛰고 이미지별 작업을 인프로세스(in-process) 방식으로 실행합니다. 백엔드 로그에서는 이 전략을 `GPU_SINGLE`라고 명명합니다.
* **12GB 이상**(Orin NX 16GB, AGX Orin): 통합 메모리 덕분에 `GPU_PARALLEL`를 적용할 수 있지만,**Jetson에서는 워커 수가 2개로 제한됩니다** — GPU, 워커 프로세스의 RAM, 그리고 워커별 CUDA 컨텍스트가 모두 동일한 공유 풀을 사용하기 때문에, 워커 수가 많을수록 메모리 부족 오류가 발생할 위험이 있습니다.
* **사용 가능한 CUDA 없음** (JetPack의 CUDA 라이브러리 누락): 12GB 이상일 경우 `CPU_PARALLEL`, 그 미만일 경우 아래의 이미지 중 하나.

`CHLOROS_STRATEGY` 환경 변수를 사용하여 자동 선택을 재정의할 수 있습니다 — [동적 컴퓨팅 적응](../processing-architecture/dynamic-compute-adaptation.md#manual-strategy-override)을 참조하십시오.

### 모델별 동작

| Jetson 모델                | 전략       | 워커 수 | 실행                                      |
| --------------------------- | -------------- | ------- | ---------------------------------------------- |
| **Jetson Orin Nano 8GB**    | `GPU_SINGLE`   | 1       | 순차적 인-프로세스 루프 (메모리 부하 시 `tiled_gpu`) |
| **Jetson Orin NX 8GB**      | `GPU_SINGLE`   | 1       | 순차적 인-프로세스 루프                     |
| **Jetson Orin NX 16GB**     | `GPU_PARALLEL` | 2       | 동시 실행되는 작업자 프로세스, `fused_gpu` 경로  |
| **Jetson AGX Orin 32-64GB** | `GPU_PARALLEL` | 2       | 동시 작업자 프로세스, `fused_gpu` 경로  |

플랫폼 간의 핵심 차이점은 **메모리**입니다. 8GB Jetson은 부하가 높을 때 메모리 효율적인 타일링 방식을 사용하여 이미지를 한 번에 하나씩 처리해야 하는 반면, 16GB 이상의 Orin은 더 높은 처리량을 제공하는 융합 파이프라인을 사용하여 GPU를 통해 2개의 이미지를 동시에 처리할 수 있습니다.

### 모델별 GPU 할당량

각 Jetson 모델에는 공유 풀 처리에서 할당받을 수 있는 양을 제한하고 배치 크기를 조정하는 하드웨어 프로필이 포함되어 있습니다:

| 모델 | GPU 할당량 상한 | 배치 크기 배율 | 시스템/디스플레이용 예약량 |
| --- | --- | --- | --- |
| **Jetson Orin Nano** | 70% | ×0.8 | 2.0 GB |
| **Jetson Orin NX** | 75% | ×1.0 | 3.0 GB |
| **Jetson AGX Orin** | 80% | ×1.5 | 4.0 GB |

감지된 RAM 용량에 따라 프로필이 조정됩니다. **16GB 이상**을 보고하는 Jetson의 경우 배치 배율이 ×1.2로 증가합니다. 배율이 적용되기 전의 기본 배치 크기는 8개의 이미지입니다.

전체 컴퓨트 적응에 대한 자세한 내용은 [동적 컴퓨트 적응](../processing-architecture/dynamic-compute-adaptation.md)을 참조하십시오.

***

## Nano 및 Orin Nano에서 Texture Aware에 대한 GPU 주파수 제한

Texture Aware 디베이어는 GPU 신경망 추론을 실행하며, 이는 저전력 Jetson 모델(10~15W급)에서 GPU 클럭 속도가 최대일 때 **과전류 경고**를 유발할 수 있습니다.**Jetson Nano 또는 Orin Nano**에서 `--debayer texture-aware`를 전달하면, CLI
는 GPU의 최대 주파수를 확인하고, 현재 주파수가 이보다 높을 경우 **510 MHz**(510000000)로 제한합니다:

* CLI
가 GPU 주파수 sysfs 노드에 쓰기 권한이 있는 경우, 제한이 **자동으로 적용**되고 확인 메시지가 출력됩니다.
* 그렇지 않은 경우(루트 권한 필요), CLI
는 제한을 수동으로 적용하기 위한 정확한 `sudo` 명령어를 출력하고, 사용자가 이를 읽을 수 있도록 잠시 대기한 후 계속 진행합니다. 이 경우 처리는 여전히 실행되지만 과전류 경고가 표시될 수 있습니다.

처리 전에 직접 상한값을 적용하려면:

```bash
echo 510000000 | sudo tee /sys/devices/platform/bus@0/17000000.gpu/devfreq/17000000.gpu/max_freq
```

고전력 모델(Orin NX 25W, AGX Orin 60W)은 GPU 최대 속도로 작동하며, 상한값이 적용되지 않습니다. Standard 디베이어는 어떤 모델에서도 상한값을 트리거하지 않습니다.

{% hint style="info" %}
**Jetson에서 Texture Aware는 항상 한 번에 하나의 이미지만 처리합니다.** 각 워커는 자체 CUDA 컨텍스트(약 1GB)와 노이즈 제거 모델의 별도 사본이 필요하지만, 통합 메모리로는 이를 감당할 수 없습니다. 따라서 Jetson에서 Texture Aware 경로는 GPU 액세스가 직렬화된 단일 워커에 고정됩니다. 모든 Jetson에서 Texture Aware는 Standard보다 현저히 느릴 것으로 예상됩니다. Texture Aware는 Survey3
이미지에 적용됩니다. LATTICE 캡처는 항상 Standard 디베이어를 사용합니다. 사용 가능한 CUDA GPU가 없는 경우 실행 시 Standard가 사용되며, 백엔드에는 그 이유를 설명하는 `[DEBAYER] … needs a CUDA GPU …` 줄이 하나 기록됩니다(`chloros-cli --verbose`와 함께 표시됨).
{% endhint %}

***

## 열 관리

Jetson 장치는 특히 밀폐된 환경이나 항공기 탑재 환경에서 열 여유 공간이 제한적입니다. Chloros
는 SoC 온도를 모니터링하고 배치 크기를 자동으로 조절합니다:

| 온도         | 조치                                            |
| ------------------- | ------------------------------------------------- |
| **&lt; 70°C**          | 정상 작동 — 최대 처리 속도          |
| **70°C** (경고)  | 배치 크기가 점진적으로 축소됨 (70°C~80°C 구간에서 100% → 50%) |
| **80°C** (위험) | 강력한 처리 속도 제한 (80°C에서 90°C 사이에서 50% → 0%) |
| **90°C** (종료) | GPU 처리 완전히 중지 — 냉각 필요 |

{% hint style="warning" %}
특히 밀폐된 현장 인클로저나 항공기 탑재 시스템에서 지속적인 처리를 위해서는 **적절한 환기 및 방열**을 확보해야 합니다. 열 스로틀링은 하드웨어를 보호하기 위해 처리 처리량을 감소시킵니다.
{% endhint %}

***

## 메모리 관리

Jetson 장치는 **통합 메모리**를 사용합니다. 즉, GPU와 CPU가 동일한 물리적 RAM을 공유합니다. 표시되는 VRAM(예: Orin NX 16GB의 경우 ~15.3GB)은 전용 GPU 메모리가 아니며, 운영 체제와 다른 모든 프로세스가 사용하는 것과 동일한 RAM입니다.

### 스왑 경고 및 권장 사항

Jetson에서 처리를 시작하기 전에 CLI
는 입력 폴더에 있는 RAW 이미지 수를 집계합니다. (`.tif`, `.tiff`, `.raw`, `.dng` — JPG 미리보기는 집계되지 않음)을 집계하고, 실행에 필요한 최대 메모리 용량을 추정한 후, RAM + 스왑 용량이 부족할 가능성이 있는 경우 **시작 전에 경고**를 표시합니다. 경고 메시지의 제목은 `LOW MEMORY WARNING - Jetson Detected`이며, 이미지 개수, RAM, 현재 스왑 용량, 예상 최대 메모리 사용량을 표시한 후, (8GB 미만인 경우는 절대 없음). 스크롤백에서 메시지가 사라지지 않도록 몇 초간 일시 정지한 후 처리가 계속됩니다.**경고에서 사용하는 메모리 추정값:**

| 디베이어 모드 | 기본 | 이미지당 |
| --- | --- | --- |
| 표준 | ~1.5 GB | ~10 MB |
| 텍스처 인식 | ~2.5 GB (모델 + Python 런타임) | ~15 MB |

예상 피크 값이 RAM + 스왑에서 1GB의 안전 여유분을 뺀 값을 초과할 때 경고가 발생하며, **파일 기반** 스왑만 계산됩니다. zram만 사용하는 설정의 경우에도 여전히 경고가 표시됩니다.

스왑을 수동으로 추가하려면 (예: 8GB):

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
```

<!-- SCREENSHOT-NEEDED: Terminal on a Jetson Orin (SSH session) showing the full "LOW MEMORY WARNING - Jetson Detected" block printed by `chloros-cli process` on a large folder: the image count and debayer mode line, RAM / current swap / estimated peak figures, and the fallocate/chmod/mkswap/swapon command block it recommends -->


### OOM(메모리 부족) 처리

처리 중, Chloros
는 GPU 메모리를 모니터링하며, 시스템이 중단되는 대신 점진적으로 성능을 낮춥니다:

1. GPU 메모리 사용률이 **85%**를 초과하면, 선제적으로 배치 크기가 축소됩니다.
2. 여전히 메모리 부족(OOM) 현상이 발생하면 배치 크기가 **절반**으로 줄어들며, 이후 OOM이 발생할 때마다 다시 절반으로 줄어듭니다. 이후 성공적으로 처리된 각 배치마다 해당 페널티가 한 단계씩 완화됩니다.
3. 지속적인 부하가 가해지면 파이프라인은 `fused_gpu`에서 메모리 효율이 높은 `tiled_gpu` 경로로 전환되며, 최후의 수단으로 CPU 처리로 전환됩니다

GPU가 Jetson의 RAM을 공유하기 때문에, RAM과 스왑 공간을 합한 용량이 여전히 실행의 상한선이 됩니다. 따라서 위의 스왑 경고는 주의 깊게 살펴봐야 할 사항입니다.

***

## 현장 배포

### 전력 관련 고려 사항

| Jetson 모델     | 일반적인 전력 소비량 | 비고                   |
| ---------------- | ------------------ | ----------------------- |
| Jetson Orin Nano | 7-15W              | DC 배럴 잭          |
| Jetson Orin NX   | 10-25W             | DC 배럴 잭          |
| Jetson AGX Orin  | 15-60W             | USB-C PD 또는 배럴 잭 |

지속적인 처리를 위한 전력 예산을 계획하십시오. GPU 사용량이 많은 스레드 3(처리)에서 최대 전력 소모가 발생합니다.

### 스토리지 권장 사항

* **NVMe SSD**는 arm64 배포 시 강력히 권장됩니다.
* SD 카드는 처리 속도가 너무 느리므로 부팅 미디어로만 사용하십시오.
* 처리된 출력 데이터를 위해 원본 이미지 데이터 크기의 2~3배를 확보하십시오.

### SSH
를 통한 헤드리스 운영

Chloros
CLI
는 헤드리스 Jetson 배포에 이상적입니다:

```bash
# SSH into the Jetson
ssh user@jetson-hostname

# Process a dataset
chloros-cli process /data/datasets/flight001 --format "TIFF (32-bit, Percent)"

# Monitor export progress
chloros-cli export-status
```

### LATTICE / DAQ-E 시간 동기화를 위한 상시 가동 백엔드

Jetson이 헤드리스 모드로 LATTICE 카메라나 DAQ-E 광 센서를 제어하는 경우, **사용자 권한으로** 백엔드 systemd 서비스를 활성화하여 PTP 그랜드마스터가 지속적으로 실행되고 SDK
스크립트가 실행 중인 백엔드를 찾을 수 있도록 하십시오(이 유닛은 설치되어 있지만 기본적으로 활성화되어 있지 않습니다):

```bash
sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
chloros-cli time-sync status
```

그러면 백엔드는 사용자로 실행됩니다. 즉, 백엔드가 기록하는 캡처 데이터는 사용자의 것이며, 사용자의 `chloros-cli login`를 사용합니다. 루트 `chloros-backend.service` 및 패키지가 루트 권한 없이 PTP 포트 319/320을 바인딩 가능하게 만드는 방법을 포함한 자세한 내용은 [Linux
설치](linux-installation.md#always-on-ptp-for-headless-hosts)를 참조하십시오.

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

`chloros-cli process`는 제품을 요청한 실행에서 이미지가 기록되지 않을 경우 0이 아닌 값으로 종료되므로, systemd의 실패 상태는 모니터링에 유용합니다.

예약 처리를 위해 systemd 타이머와 연동합니다:

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
    --indices NDVI GNDVI OSAVI
```

### Jetson에서의 Python
SDK


```python
from chloros_sdk import ChlorosLocal

with ChlorosLocal() as chloros:
    chloros.create_project("field_survey_042")
    chloros.import_images("/data/flights/flight_042")
    chloros.configure(
        indices=["NDVI", "GNDVI", "OSAVI"],
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
        --indices NDVI OSAVI
    echo "Completed $name"
done
```

***

## 현장 사용을 위한 권장 Jetson 시스템

현장 및 항공기 탑재용 배포의 경우, 다음 Jetson Orin NX 16GB 캐리어 보드 옵션을 고려해 보십시오:

* **항공기/드론**: 진동 등급(MIL-STD)을 충족하고, 경량(300g 미만)이며, 수동 냉각 방식인 시스템
* **견고한 현장**: PoE 기가비트 이더넷 카메라 연결이 가능한 IP67/IP69K 방수 인클로저
* **최소 구성/저비용**: 추가 인클로저가 포함된 개발자 키트

구축 시나리오에 맞는 구체적인 하드웨어 권장 사항이 필요하시면 [MAPIR
지원팀](https://www.mapir.camera/community/contact)에 문의하십시오.

***

## 다음 단계

* [Linux
설치](linux-installation.md) — 일반적인 Linux
설치 세부 정보
* [동적 컴퓨팅 적응](../processing-architecture/dynamic-compute-adaptation.md) — 전체 컴퓨팅 전략 참조
* [처리 파이프라인](../processing-architecture/processing-pipeline.md) — 4스레드 파이프라인 이해
* [CLI : 명령줄](../CLI.md) — CLI
가이드
* [API
: Python
SDK
](../api-python-sdk.md) — SDK
가이드
* [CLI
참조](../reference/cli-reference.md) 및 [SDK
참조](../reference/sdk-reference.md) — 1.3.0 버전의 모든 명령어/API
목록
