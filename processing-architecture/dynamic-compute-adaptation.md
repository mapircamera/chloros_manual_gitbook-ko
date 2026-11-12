# 동적 컴퓨팅 적응 기능:

Chloros 1.3.0은 하드웨어 감지 및 자동 처리 전략 선택 기능을 사용합니다. 처리 엔진은 Jetson Orin Nano부터 멀티 GPU 워크스테이션에 이르기까지 사용자의 하드웨어에 맞춰 자동으로 적응하며, 별도의 수동 구성은 필요하지 않습니다.

***

## 작동 원리

Chloros가 시작되면 시스템 프로파일링을 수행합니다:

1. **운영 체제 감지** — Windows 또는 Linux
2. **CPU 코어 및 총 RAM 식별**

3.**GPU 존재 여부 감지** — NVIDIA CUDA 기능, VRAM, 모델
4. **Jetson 모델 식별** (해당되는 경우) — `/proc/device-tree/model`를 통해
5. **열 센서 확인** (Jetson) — 온도 감지 처리를 위해
6. **연산 전략 선택** — 감지된 모든 하드웨어를 기반으로
7. **워커 수, 파이프라인 유형 및 메모리 할당**을 자동으로 구성합니다

감지된 프로필은 세션 동안 메모리와 디스크에 캐시되므로, 이후 실행 시 더 빠르게 시작됩니다:

| 플랫폼 | 캐시된 프로필 |
| --- | --- |
| **Linux / Jetson** | `~/.config/chloros/system_config.json` (`XDG_CONFIG_HOME` 우선 적용) |
| **Windows** | `%LOCALAPPDATA%\Chloros\config\system_config.json` |

해당 파일을 삭제하면 강제적으로 새로운 프로필을 감지할 수 있습니다. 이는 GPU를 추가하거나 RAM을 증설한 후에 유용합니다. 또한 캐시가 호환되지 않는 구버전에 의해 작성된 경우, Chloros는 자동으로 재감지를 수행합니다.

***

## 연산 전략

Chloros는 사용자의 하드웨어에 따라 다음 세 가지 연산 전략 중 하나를 선택합니다:

| 전략 | 선택 조건 | 워커 | 실행기 | 파이프라인 |
| --- | --- | --- | --- | --- |
| **`GPU_PARALLEL`**|**12GB 이상의 VRAM**을 보고하는 CUDA GPU (Jetson 통합 메모리 기준, 총 공유 RAM 12GB 이상 필요) | `min(4, ⌊(VRAM − 0.5GB) × 0.9 ÷ 3.5GB⌋)`, 최소 2개 —**Jetson에서는 2개로 제한** | 워커 프로세스(별도의 CUDA 컨텍스트) | `fused_gpu` |
| **`GPU_SINGLE`**|**2~12GB VRAM**을 갖춘 CUDA GPU | 3 (I/O 오버랩; 세마포어에 의해 GPU 액세스가 순차화됨).**RAM이 12GB 미만인 Jetson에서는 1 (순차적)** | 작업자 프로세스(별도의 CUDA 컨텍스트); RAM 용량이 적은 Jetson에서는 프로세스 내 순차 처리 | `fused_gpu` / `tiled_gpu` |
| **`CPU_PARALLEL`** | CUDA GPU 없음 또는 2GB 미만 VRAM | `max(2, physical cores − 1)` | `ThreadPoolExecutor` | `cpu_fallback` |

`GPU_PARALLEL` 워커 산정식의 실제 적용 예시: 12GB VRAM → 2명의 워커, 16GB → 3명의 워커, 20GB 이상 → 4명의 워커. 각 워커에는 0.5GB를 예비로 확보한 후 남은 VRAM의 90% 범위 내에서, 텍스처 인식(Texture Aware) 작업에 필요한 측정된 용량인 3.5GB가 할당됩니다.

{% hint style="info" %}
**GPU 전략을 적용하려면 CUDA를 사용할 수 있는 PyTorch가 필요합니다.** 사용 가능한 CUDA 장치가 없는 PyTorch를 실행하는 머신은 GPU 종류와 관계없이 `CPU_PARALLEL`를 할당받습니다. 1.3.0 Jetson 패키지는 JetPack 6의 CUDA 빌드 PyTorch를 사용하므로, JetPack의 CUDA 및 cuDNN 라이브러리가 설치되면 Jetson도 지원 대상이 됩니다. RAM이 12GB 미만인 Jetson은 항상 한 번에 하나의 이미지를 프로세스 내에서 처리합니다 (`GPU_SINGLE`로 기록됨).
{% endhint %}

GPU 전략은 워커 프로세스 풀을 실행하며, 각 프로세스는 고유한 CUDA 컨텍스트를 가진 별도의 OS 프로세스입니다. Chloros는 자체 설치된 백엔드 프로그램을 다시 실행하여 이를 시작합니다(Python의 `multiprocessing`를 통하지 않음. 패키지된 애플리케이션 내부에서는 이 방법을 통한 워커 시작이 작동하지 않음). CPU 전략은 스레드 풀(Python의 `ThreadPoolExecutor`)을 사용합니다. Chloros는 Ray와 같은 타사 분산 프레임워크를 사용하지 않습니다.

### 파이프라인 유형

* **`fused_gpu`** — 완전한 GPU 처리 경로. 디베이어, 보정 및 인덱스 연산이 단일 융합 패스로 GPU에서 실행됩니다. 처리량이 가장 높지만, 가장 많은 VRAM이 필요합니다.
* **`tiled_gpu`** — 메모리 효율적인 GPU 경로. 제한된 GPU 메모리 용량에 맞도록 이미지를 타일 단위로 처리합니다. 처리량은 낮지만 메모리 제약이 있는 장치에서도 작동합니다.
* **`cpu_fallback`** — 멀티스레드 병렬 처리를 사용하는 CPU 전용 처리 방식입니다. NVIDIA GPU를 사용할 수 없을 때 사용되며, 두 GPU 처리 경로 모두 실패할 경우 최후의 수단으로 사용됩니다.

런타임 대체 처리 순서는 항상 `fused_gpu` → `tiled_gpu` → `cpu_fallback`입니다.

***

## 수동 전략 재정의

특정 전략을 강제 적용하려면 `CHLOROS_STRATEGY` 환경 변수를 설정하십시오. 이는 자동 감지 기능이 사용자의 상황에 부적합한 전략을 선택한 경우(예: 다른 작업을 위해 GPU를 비워두어야 하는 경우)를 대비한 전문가용 비상 조치입니다:

```bash
# Valid values: CPU_PARALLEL, GPU_SINGLE, GPU_PARALLEL
CHLOROS_STRATEGY=CPU_PARALLEL chloros-cli process ~/datasets/flight001
```

이 변수는 대소문자를 구분하지 않고 일치 여부를 확인합니다. 세 가지 이름 중 하나에 해당하지 않는 값은 무시되며, 자동 감지가 정상적으로 진행됩니다. 재정의가 적용된 상태에서도 Chloros는 여전히 사용자를 대신해 워커 수를 선택합니다:

| 재정의 | 사용된 워커 수 |
| --- | --- |
| `CPU_PARALLEL` | `max(2, physical cores − 1)` |
| `GPU_SINGLE` | 3 |
| `GPU_PARALLEL` | `min(4, physical cores)` |

사용 가능한 CUDA 장치가 없는 시스템에서 `GPU_SINGLE` 또는 `GPU_PARALLEL`로 오버라이드하려는 시도는 로그에 `[STRATEGY]` 행이 기록되며 거부되며, 실행 시에는 `CPU_PARALLEL`가 사용됩니다.

영구적으로 설정하기보다는 명령어별로 설정하는 것을 권장하며, 이렇게 하면 정상적인 실행 시 자동 적응 기능이 계속 유지됩니다.

***

## 플랫폼별 동작

| 플랫폼 | 전략 | 워커 | 파이프라인 | 참고 사항 |
| --- | --- | --- | --- | --- |
| **Jetson Orin Nano 8GB** | `GPU_SINGLE` | 1 | `tiled_gpu` (순차적) | 메모리 효율 모드, 한 번에 하나의 이미지 |
| **Jetson Orin NX 8GB** | `GPU_SINGLE` | 1 | `tiled_gpu` (순차적) | 12GB 미만의 공유 RAM으로 인해 순차적 처리가 강제됨 |
| **Jetson Orin NX 16GB** | `GPU_PARALLEL` | 2 | `fused_gpu` (병렬) | 권장 엣지 디바이스 — Jetson의 경우 작업자 수 2명으로 제한 |
| **Jetson AGX Orin 32-64GB** | `GPU_PARALLEL` | 2 | `fused_gpu` (병렬) | 최대 엣지 성능 (Jetson의 경우 최대 2개의 워커로 제한됨) |
| **8GB GPU 탑재 데스크톱** | `GPU_SINGLE` | 3 | `fused_gpu` / `tiled_gpu` | 3개의 워커가 I/O를 중첩 처리하는 동안 세마포어가 GPU 액세스를 직렬화 |
| **12GB 이상 GPU 탑재 데스크톱** | `GPU_PARALLEL` | 2-4 | `fused_gpu` (동시) | 최적의 데스크톱 성능: 12GB → 2개 워커, 16GB → 3개, 20GB 이상 → 4개 |
| **CPU 전용 시스템** | `CPU_PARALLEL` | 물리적 코어 수 − 1 (최소 2) | `cpu_fallback` | GPU 불필요, 스레드 풀 사용 |

{% hint style="info" %}
**Jetson 통합 메모리**: Jetson 장치는 GPU와 CPU 메모리를 공유합니다. Jetson Orin NX 16GB는 약 15.3GB의 VRAM을 보고하지만, 이는 OS 및 CPU 프로세스가 사용하는 물리적 RAM과 동일합니다. 이 때문에 16GB 이상의 Jetson은 12GB 이상의 데스크톱 GPU와 마찬가지로 `GPU_PARALLEL`에 해당하지만, 작업자 수는 2개로 제한됩니다. GPU, 작업자 프로세스, 그리고 작업자별 CUDA 컨텍스트가 모두 동일한 공유 풀을 사용하기 때문입니다.
{% endhint %}

### VRAM별 GPU 할당량 (디스크리트 GPU)

디스크리트 NVIDIA GPU가 장착된 x86_64 호스트에서, 감지된 VRAM은 카드가 처리할 수 있는 양과 배치 크기가 얼마나 커질 수 있는지를 결정합니다:

| 감지된 VRAM | GPU 예산 상한 | 배치 크기 배율 |
| --- | --- | --- |
| **8GB 이상** | 90% | ×2.0 |
| **6~8GB** | 85% | ×1.75 |
| **3.5~6GB** | 80% | ×1.5 |
| **2~3.5GB** | 75% | ×1.25 |
| **2GB 미만** | 70% | ×1.0 |

디스크리트 GPU는 시스템 RAM을 공유하지 않기 때문에 시스템용으로 0.5GB만 할당합니다. Jetson 프로필은 훨씬 더 많은 메모리를 할당하고 상한선도 더 낮게 설정합니다. 자세한 내용은 [NVIDIA Jetson 가이드](../linux/nvidia-jetson-guide.md#per-model-gpu-budget)를 참조하십시오.

***

## 동적 GPU 메모리 할당

Chloros는 [4스레드 처리 파이프라인](processing-pipeline.md)을 사용합니다:

* **스레드 1** (탐지) — 이미지 불러오기, EXIF 파싱, 대상 탐지
* **스레드 2** (보정) — 반사율 보정 계산
* **스레드 3** (처리) — GPU 디베이어, 비네팅 보정, 인덱스 계산
* **스레드 4** (내보내기) — 파일 쓰기, 메타데이터 삽입

스레드 1, 2, 4는 GPU 자원을 적게 소비합니다. 스레드 3은 GPU를 많이 소비합니다. 앞선 파이프라인 스레드들이 완료되면 해당 스레드들의 GPU 할당량이 **나머지 활성 스레드들로 재분배**되므로, 실행이 진행될수록 스레드 3은 점차 더 많은 메모리를 확보하게 됩니다.

### 할당 단계

| 단계 | 활성 스레드 | GPU 메모리 분배 |
| --- | --- | --- |
| **초기** | 1, 2, 3, 4 | 모든 스레드에 분배되며, 대부분은 스레드 3에 할당 |
| **중기-초기** | 2, 3, 4 | 스레드 1의 할당량이 재분배됨 |
| **중후반** | 3, 4 | 스레드 1과 2의 할당량이 3과 4로 이동 |
| **후반** | 3 또는 4 | 마지막으로 활성화된 스레드가 최대 할당량을 확보 |

이 수치를 결정하는 두 가지 규칙은 다음과 같습니다:

* **유일하게** 활성 상태인 스레드에는 해당 프로파일의 최대 할당량이 부여됩니다.
* *부하가 큰* GPU 작업이 두 개 이상 활성 상태인 경우, 각 부하가 큰 작업의 기본 할당량이 이들 사이에 분배됩니다(설정된 최소값 이하로 떨어지지 않음).

실행 시 실제로 사용되는 값은 플랫폼 프로필의 할당량과 GPU 메모리 모니터의 실시간 권장값 중 **더 낮은** 값이므로, 부하가 높은 그래픽 카드는 항상 낙관적인 프로필 설정보다 우선합니다.

***

## 텍스처 인식 처리

텍스처 인식 디베이어(**Chloros+ 전용** — `--debayer texture-aware`)는 복사본당 FP16 기준으로 약 1.75GB의 VRAM이 필요한 AI/ML 노이즈 제거 모델을 실행하므로, 표준 방식보다 훨씬 더 많은 GPU 메모리를 사용합니다:

* **7GB 미만의 VRAM**을 갖춘 시스템은 텍스처 인식 처리를**동기식 루프에서 한 번에 한 이미지씩** 처리합니다. 여러 모델 복사본을 동시에 처리할 수 없으며, 워커 풀을 사용해도 경합만 가중될 뿐입니다
* **7GB 이상의 VRAM**을 갖춘 시스템은 Texture Aware를 병렬로 처리할 수 있지만, Standard 방식에 비해 워커 수가 줄어듭니다
* **Jetson**에서는 Texture Aware가 항상 단일 워커에 고정됩니다.
* **사용 가능한 CUDA GPU가 없는** 경우, 실행 시 표준 디베이어가 사용되며, 백엔드에는 그 이유를 설명하는 `[DEBAYER] … needs a CUDA GPU …` 로그 한 줄이 기록됩니다.
* 저전력 Jetson(Nano, Orin Nano)의 경우, CLI에서 Texture Aware를 요청하면 GPU 주파수 제한이 적용됩니다 — [NVIDIA Jetson 가이드](../linux/nvidia-jetson-guide.md#gpu-frequency-cap-for-texture-aware-on-nano-and-orin-nano)를 참조하십시오

***

## 열 관리 (Jetson)

Jetson 기기는 특히 밀폐된 공간이나 항공기 내 배치 시 열 관련 제약이 있습니다. Chloros는 Jetson의 내장 온도 센서를 모니터링하여 배치 크기를 자동으로 조정합니다:

| 온도 | 대응 |
| --- | --- |
| **< 70°C** | 정상 작동 — 최대 속도 |
| **70°C** (경고) | 배치 크기가 점진적으로 축소됨 (70°C에서 80°C 사이에서 100% → 50%) |
| **80°C** (위험) | 강력한 스로틀링 (80°C에서 90°C 사이에서 50% → 0%) |
| **90°C** (종료) | GPU 처리 완전히 중지 |

적절한 냉각 장치가 갖춰진 데스크톱 시스템에서는 열 스로틀링이 거의 발생하지 않습니다.

***

## 메모리 부하 처리

Chloros는 처리 중 GPU 메모리를 지속적으로 모니터링하며 세 가지 단계로 대응합니다.

**배치 크기 조정.** 배치는 위 표의 플랫폼 배수를 곱한 8장의 이미지 수로 시작됩니다. 그런 다음 Chloros는 사용 가능한 VRAM을 확인하고, 그중 20%를 PyTorch 자체 오버헤드용으로 예약하며, 12MP 이미지당 대략 100MB의 GPU 메모리를 가정합니다. 배치 크기는 메모리 기반 제한과 플랫폼 기본값 중 더 작은 쪽이 적용됩니다. 배치 크기는 절대 1 미만으로 떨어지지 않습니다.

**선제적 축소.** **VRAM 사용률이 85%**를 초과하면, 오류가 발생하기 전에 배치 크기가 축소됩니다.**스레드별 할당 조정.** 실시간 사용률이 상승함에 따라 각 스레드의 GPU 할당량이 축소됩니다: 사용률 80% 이상일 경우 ×0.75, 90% 이상일 경우 ×0.5로 조정됩니다. 모니터의 구간은 70%(보수적), 85%(정상 작동 한계), 95%(OOM 위험)입니다.**OOM 백오프 및 복구.** 메모리 부족(OOM) 이벤트가 발생하면:

* 배치 크기가 **절반으로** 줄어들며, OOM이 연속으로 발생할 때마다 다시 절반으로 줄어듭니다. 이후 성공적인 배치가 이루어질 때마다 이 페널티가 한 단계씩 완화됩니다.
* 활성 스레드의 할당량은 현재 값의 70%로 축소되며, 할당기는 보수적인 전략으로 전환됩니다. 이후 할당이 연속으로 성공하면 다시 완화됩니다.
* 심각한 부하가 가해지면 파이프라인은 `fused_gpu`에서 `tiled_gpu`로, 최후의 수단으로 `cpu_fallback`로 후퇴합니다.

**호스트 RAM (Jetson).** 처리 전에 CLI는 이미지 수와 디베이어 모드를 기반으로 호스트 메모리의 최대 용량을 추정하며, RAM과 파일 기반 스왑 용량이 부족할 가능성이 있을 경우 경고를 표시하고 스왑을 추가하는 정확한 명령어를 출력합니다. 자세한 내용은 [NVIDIA Jetson 가이드](../linux/nvidia-jetson-guide.md#swap-warning-and-recommendations)를 참조하십시오.

***

## 컴퓨트 적응 모니터링

### 시스템 진단

`chloros-cli selftest`는 컴퓨트 레이어가 인식하는 내용을 확인하는 가장 빠른 방법입니다:

```bash
chloros-cli selftest
```

이 명령어의 7가지 점검 항목은 버전, 포트 가용성, 백엔드 시작 상태, `/api/test`, 시스템 정보, 노이즈 제거기 모델 존재 여부, CUDA 및 노이즈 제거기 준비 상태를 다룹니다. 5번 점검 항목은 하드웨어 라인을 직접 출력합니다:

```
      GPU: NVIDIA RTX A4000, CUDA: True, PyTorch: 2.7.0
```

7번 확인 항목은 `CUDA: <bool>, Denoiser: <bool>`를 출력합니다. Texture Aware를 사용하려면 이 두 가지 조건이 모두 충족되어야 합니다.

### 백엔드 로그

전략과 워커 수는 각 실행 시작 시 백엔드 내에서 선택되며, 이를 알리는 CLI배너는 없습니다. 예기치 않은 동작(GPU 경로의 폴백, OOM, 노이즈 제거기 로드 실패 등)이 발생하면 해당 세션의 백엔드 로그에 그 내용이 표시됩니다:

| 플랫폼 | 로그 위치 |
| --- | --- |
| **Linux / Jetson** | `~/.cache/chloros/logs/backend_<YYYYMMDD_HHMMSS>.log` (실행당 하나의 파일) |
| **Linux, CLI로 시작된 백엔드** | `~/.chloros/backend.log` (실행당 하나의 파일) |
| **Windows** | `%LOCALAPPDATA%\Chloros\logs\` |

### 실시간 진행 상황

실행 중에는 CLI가 서버 전송 이벤트(Server-Sent Events)를 통해 스트리밍되는 스레드별 실시간 진행 상황(탐지, 분석, 처리, 내보내기)을 표시합니다. 이는 스레드 3이 병목 현상인지 여부를 파악하는 데 실질적인 지표가 됩니다. [처리 파이프라인](processing-pipeline.md)을 참조하십시오.

***

## 다음 단계

* [처리 파이프라인](processing-pipeline.md) — 4스레드 파이프라인 아키텍처 이해
* [NVIDIA Jetson 가이드](../linux/nvidia-jetson-guide.md) — Jetson 전용 배포 및 최적화
* [CLI: 명령줄](../CLI.md) — CLI 가이드
* [CLI 참조](../reference/cli-reference.md) — 1.3.0 버전의 전체 명령어 목록
