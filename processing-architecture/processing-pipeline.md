# 처리 파이프라인

Chloros1.2.0은 단계별 조립 라인처럼 작동하는 4스레드 처리 파이프라인을 사용합니다. 각 스레드는 워크플로우의 서로 다른 단계를 처리하므로, 여러 이미지가 동시에 서로 다른 단계에서 처리될 수 있습니다.

<figure><img src="../.gitbook/assets/image (39).png" alt=""><figcaption></figcaption></figure>

***

## 파이프라인 아키텍처

```

Images In → [Thread 1: Detection] → [Thread 2: Calibration] → [Thread 3: Processing] → [Thread 4: Export] → Files Out
```

모든 이미지는 4개의 스레드를 순서대로 거칩니다. Chloros+의 멀티스레드 처리 방식에서는 여러 이미지가 동시에 서로 다른 스레드에서 처리됩니다. 즉, 스레드 3이 한 이미지를 처리하는 동안, 스레드 1은 다음 이미지를 감지하고, 스레드 2는 또 다른 이미지를 보정하며, 스레드 4는 완성된 이미지를 디스크에 기록할 수 있습니다.

진행 상황은 스레드별로 보고되며, 서버 전송 이벤트(Server-Sent Events)를 통해 스트리밍됩니다(백엔드는 이를 `/api/events`에 게시합니다). CLI의 실시간 진행 상황 표시창에서는 네 가지 단계가 **탐지, 분석, 처리, 내보내기**로 표시됩니다.***

## 스레드 세부 정보

### 스레드 1: 감지

**목적**: 이미지를 불러오고 보정 대상을 감지합니다.

* 디스크에서 이미지 파일을 읽습니다 — Survey3 `.raw`+`.jpg` 쌍, LATTICE `.tif`/`.tiff` 캡처, 및 `.dng`
* EXIF 메타데이터(GPS, 카메라 모델, 타임스탬프, 노출) 추출
* 보정 타겟 감지: LATTICE 촬영 이미지의 경우 ArUco 마크가 표시된 타겟 기하 구조, Survey3 보정 타겟 사진의 경우 클래식 패널 감지기
* 출력: 이미지 데이터 + 메타데이터 + 타겟 감지 결과

주로 I/O 및 CPU에 의존하는 스레드입니다.

### 스레드 2: 보정

**목적**: 감지된 타겟을 기반으로 보정 매개변수를 계산합니다.

* 타겟 이미지로부터 반사율 보정 계수를 계산합니다.
* 비네팅 보정 매개변수를 계산합니다.
* 밴드별 보정 곡선을 결정합니다.
* 출력: 각 이미지에 대한 보정 매개변수

CPU에 의존적인 계산 스레드입니다. 반사율 보정이 활성화된 경우 스레드 3은 이 스레드가 완료될 때까지 대기하므로, 어떤 이미지도 처리되기 전에 보정 계수가 준비됩니다.

### 스레드 3: 처리 (GPU)

**목적**: 보정을 적용하고 식생 지수를 계산합니다.**이 스레드는 가장 높은 연산 부하를 받는 스레드입니다.*** **디베이어링**: RAW 베이어 데이터를 다중 채널 이미지로 변환합니다
  * 표준(빠름, 중간 품질) — 기본값, `--debayer standard`
  * 텍스처 인식(느림, 최고 품질) — Chloros+ 전용, `--debayer texture-aware`, AI/ML 노이즈 제거 모델 사용
  * LATTICE mono (M3M) 캡처는 단일 대역이므로, 이 경우 디모자이크 및 화이트 밸런스 단계가 건너뜁니다(한 줄의 로그 메시지가 표시됨). 반면, 동일한 실행에 포함된 M3C/베이어 이미지는 여전히 해당 처리를 거칩니다
* **비네팅 보정**: 이미지 전체에 렌즈 비네팅 보정을 적용합니다.
* **반사율 보정**: 보정 계수를 적용하여 반사율 값으로 변환합니다.
* **지수 계산**: 식생 지수(NDVI, NDRE, GNDVI, …)를 계산합니다.
* 출력: 내보내기 준비가 완료된 처리된 이미지 데이터

이 스레드는 GPU 가속의 이점을 가장 많이 받으며, [동적 연산 적응(Dynamic Compute Adaptation)](dynamic-compute-adaptation.md)이 조정하는 대상입니다.

### 스레드 4: 내보내기

**목적**: 처리된 이미지를 디스크에 기록합니다.

* 선택한 형식으로 출력 파일을 작성합니다 — `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)`
* 출력 파일에 메타데이터(GPS, 타임스탬프, 처리 매개변수)를 포함시킵니다.
* 출력 파일을 프로젝트 폴더 아래에 `<camera>/<format>/<Product>_Images/` 형식으로 정리합니다(예: `LATT-M3M-L41-F550/tiff16/Reflectance_Calibrated_Images/`). **내보낸 파일은 원본 파일의 이름을 유지하며, 폴더명이 해당 산출물을 식별합니다.**
* LATTICE 캡처의 경우, 하나의 원본 프레임이 여러 산출물(Debayered, Preview, Radiance, Reflectance, Index)로 분할될 수 있으며, 각 산출물은 별도의 폴더에 저장됩니다.
* 출력: 디스크에 저장된 최종 파일

주로 I/O에 의존하는 스레드이므로, SSD 저장 장치를 사용하면 성능이 눈에 띄게 향상됩니다.

***

## 내부 작동 원리: 실행기

스레드 3 내에서, 이미지별 작업은 Python의 표준 `concurrent.futures`를 사용하여 병렬화됩니다:

* **GPU 전략**(`GPU_SINGLE`, `GPU_PARALLEL`)은**spawn** 시작 메서드를 사용하는 `ProcessPoolExecutor`를 사용합니다. 각 워커는 자체 CUDA 컨텍스트를 가진 별도의 프로세스입니다(`fork`는 부모의 초기화된 CUDA 상태를 상속받아 자식 프로세스를 손상시킵니다)
* **`CPU_PARALLEL`**는 `ThreadPoolExecutor`를 사용합니다 — NumPy와 OpenCV는 GIL을 해제하므로 스레드만으로도 충분합니다
* 공유 RAM이 8GB 이하인 Jetson 장치는 익스큐터를 완전히 건너뛰고 프로세스 내에서 순차적으로 처리합니다.
* VRAM이 7GB 미만인 GPU에서 Texture Aware도 순차적으로 실행됩니다 — 노이즈 제거 모델은 한 번 이상 맞출 수 없습니다.

Chloros는 Ray와 같은 타사 분산 프레임워크를 전혀 사용하지 않습니다. 전략 및 워커 수가 어떻게 결정되는지는 [동적 연산 적응(Dynamic Compute Adaptation)](dynamic-compute-adaptation.md)을 참조하십시오.

***

## 순차 처리 대 파이프라인 처리

### 자유 모드(순차)

Chloros의 무료 버전에서는 이미지가 **한 번에 하나씩**, 총 네 단계에 걸쳐 순차적으로 처리됩니다:

```

Image 1: [Detect] → [Calibrate] → [Process] → [Export]
                                                         Image 2: [Detect] → [Calibrate] → [Process] → [Export]
```

GUI는 무료 모드에서 단순화된 진행률 표시줄을 보여주며, 순차적 단계는 **표적 탐지**, 그 다음**처리**로 표시됩니다.

### Chloros+ 모드 (파이프라인 방식)

Chloros+ 라이선스를 사용하면 4개의 스레드 모두가 서로 다른 이미지를 대상으로 **동시에** 작동합니다:

```

Thread 1: [Image 1] [Image 2] [Image 3] [Image 4] ...
Thread 2:           [Image 1] [Image 2] [Image 3] ...
Thread 3:                     [Image 1] [Image 2] ...
Thread 4:                               [Image 1] ...
```

GUI 진행률 표시줄에는 4가지 단계가 표시되며, 마우스를 올려놓으면 스레드별 진행 상황을 확인할 수 있습니다. CLI에서는 동일한 4가지 단계가 **탐지, 분석, 처리, 내보내기**로 실시간으로 표시됩니다.

{% hint style="info" %}
**하나의 단계, 두 가지 이름.** CLI에서는 3단계를 _Processing_이라고 부릅니다. 백엔드의 프리미엄 모드 진행 상황 피드(GUI 진행률 막대가 표시하는 것)에서는 같은 단계를 _Calibrating_이라고 표시합니다. 이는 동일한 작업을 수행하는 동일한 스레드입니다(스레드 3: 디베이어, 보정, 인덱스).
{% endhint %}

{% hint style="success" %}
**Chloros 이상을 활용한 파이프라인 처리**는 하드웨어 및 데이터셋 크기에 따라 순차 처리보다 3~5배 더 빠를 수 있습니다. 고속 GPU와 SSD를 갖춘 시스템에서 속도 향상 효과가 가장 큽니다.
{% endhint %}

***

## 스레드 4: 내보내기 진행 상황

내보내기 스레드에는 별도의 진행 상황 추적 기능이 있으며, 이를 별도로 조회할 수 있습니다:

**CLI:**

```bash
chloros-cli export-status
```

**SDK:**

```python
status = chloros.get_status()
print(f"Export: {status['export']['percent']}% - Phase: {status['export']['phase']}")
```

스레드 4가 100%에 도달하면 처리가 완료됩니다.

{% hint style="info" %}
**이미지를 하나도 작성하지 못한 실행은 실패로 간주됩니다.**성공 시, `chloros-cli process`는 작성된 이미지 제품 수(`Image products written: N`)를 보고합니다. 제품이 요청되었으나**아무것도**기록되지 않은 경우(단순히 `project.json` 및 `calibration_data.json`만 출력된 경우), CLI는 `Processing finished but wrote no image products.`를 출력하고**0이 아닌 값으로 종료**되며, 프로젝트 폴더 이름과 일반적인 원인(입력 폴더가 캡처로 인식되지 않았음 — 레이아웃을 확인하십시오. `--input-level` — 또는 요청된 모든 제품이 해당 카메라에 적용되지 않음)을 함께 표시합니다. 스크립트는 이 종료 코드를 활용할 수 있습니다.
{% endhint %}

***

## 동적 연산 적응(Dynamic Compute Adaptation)과의 관계

[동적 연산 적응](dynamic-compute-adaptation.md)은 주로 **스레드 3(처리)**에 영향을 미칩니다:

* **`GPU_PARALLEL`**: 스레드 3은 `fused_gpu` 파이프라인을 사용하여 여러 이미지를 GPU를 통해 동시에 처리합니다.
* **`GPU_SINGLE`**: 스레드 3은 `fused_gpu` 또는 메모리 효율적인 `tiled_gpu` 파이프라인을 사용하여, 워커 프로세스가 I/O를 중첩 처리하는 동안 세마포어를 통해 GPU 액세스를 직렬화합니다
* **`CPU_PARALLEL`**: 스레드 3은 멀티스레드 병렬 처리를 통해 CPU 기반 처리를 수행합니다

스레드 1과 2가 완료됨에 따라 스레드 3의 GPU 메모리 할당량도 증가합니다. 자세한 내용은 [동적 GPU 메모리 할당](dynamic-compute-adaptation.md#dynamic-gpu-memory-allocation)을 참조하십시오.

***

## 다음 단계

* [동적 연산 적응](dynamic-compute-adaptation.md) — Chloros가 하드웨어에 최적화된 전략을 선택하는 방법
* [NVIDIA Jetson 가이드](../linux/nvidia-jetson-guide.md) — Jetson 플랫폼별 파이프라인 동작
* [처리 모니터링](../processing-images-gui/monitoring-the-processing.md) — GUI를 통한 진행 상황 모니터링
* [CLI 참조](../reference/cli-reference.md) — `process`, `export-status`, 종료 코드 및 출력 레이아웃
