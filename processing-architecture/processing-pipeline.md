# 처리 파이프라인

Chloros 1.1.0은 단계별 조립 라인 방식으로 작동하는 4스레드 처리 파이프라인을 사용합니다. 각 스레드는 처리 워크플로의 서로 다른 단계를 담당하므로, 여러 이미지를 서로 다른 단계에서 동시에 처리할 수 있습니다.

***

## 파이프라인 아키텍처

```

Images In → [Thread 1: Detection] → [Thread 2: Calibration] → [Thread 3: Processing] → [Thread 4: Export] → Files Out
```

각 이미지는 4개의 스레드를 순차적으로 거칩니다. Chloros+ 멀티스레드 처리를 통해 여러 이미지가 동시에 서로 다른 스레드에서 처리될 수 있습니다. 즉, 스레드 3이 한 이미지를 처리하는 동안 스레드 1은 다음 이미지를 탐지하고, 스레드 2는 또 다른 이미지를 보정하며, 스레드 4는 이전에 처리된 이미지를 디스크에 기록할 수 있습니다.

***

## 스레드 세부 정보

### 스레드 1: 감지

**목적**: 이미지를 로드하고 보정 타깃을 감지합니다.

* 디스크에서 이미지 파일 읽기 (RAW, JPG)
* EXIF 메타데이터 추출 (GPS, 카메라 모델, 타임스탬프, 노출)
* 표시된 타겟 이미지에서 ArUco 보정 타겟 탐지
* 출력: 이미지 데이터 + 메타데이터 + 타겟 탐지 결과

이 스레드는 주로 I/O 및 CPU 집약적인 작업입니다.

### 스레드 2: 보정

**목적**: 탐지된 타겟을 기반으로 보정 매개변수 계산.

* 타겟 이미지로부터 반사율 보정 계수를 계산합니다
* 비네팅 보정 매개변수를 계산합니다
* 밴드별 보정 곡선을 결정합니다
* 출력: 각 이미지에 대한 보정 매개변수

이 스레드는 CPU 집약적인 계산 스레드입니다.

### 스레드 3: 처리 (GPU)

**목적**: 보정을 적용하고 식생 지수를 계산합니다.**이 스레드는 가장 계산 집약적인 스레드입니다.*** **데베이어링**: RAW 베이어 패턴 데이터를 다중 채널 이미지로 변환
  * 표준 (빠름, 중간 품질) — 기본값
  * 텍스처 인식 (느림, 최고 품질) — Chloros+ 전용, AI/ML 노이즈 제거 사용
* **비네팅 보정**: 이미지 전체에 렌즈 비네팅 보정을 적용
* **반사율 보정**: 보정 계수를 적용하여 반사율 값으로 변환
* **지수 계산**: 식생 지수(NDVI, NDRE, GNDVI 등)를 계산
* 출력: 내보내기 준비가 완료된 처리된 이미지 데이터

이 스레드는 GPU 가속의 이점을 가장 많이 받습니다. [동적 컴퓨트 적응(Dynamic Compute Adaptation)](dynamic-compute-adaptation.md) 시스템은 주로 이 스레드의 동작을 최적화합니다.

### 스레드 4: 내보내기

**목적**: 처리된 이미지를 디스크에 기록합니다.

* 선택한 형식(TIFF 16비트, TIFF 32비트 %, PNG, JPG)으로 출력 파일을 기록합니다.
* 출력 파일에 EXIF 메타데이터(GPS, 타임스탬프, 처리 매개변수)를 삽입합니다.
* 출력물을 카메라 모델별 하위 폴더로 정리합니다.
* 출력 결과: 디스크상의 최종 파일

이 스레드는 주로 I/O에 의존하는 스레드입니다. SSD 저장 장치를 사용하면 스레드 4의 성능이 크게 향상됩니다.

***

## 순차 처리 대 파이프라인 처리

### 무료 모드(순차)

Chloros의 무료 버전에서는 이미지가 **한 번에 하나씩**, 총 4단계의 과정을 순차적으로 거치며 처리됩니다:

```

Image 1: [Detect] → [Calibrate] → [Process] → [Export]
                                                         Image 2: [Detect] → [Calibrate] → [Process] → [Export]
```

GUI 진행률 표시줄에는 &#x27;타겟 감지(Target Detect)&#x27;와 &#x27;처리(Processing)&#x27;의 두 단계가 표시됩니다.

### Chloros+ 모드 (파이프라인)

Chloros+ 라이선스를 사용하면, 4개의 스레드 모두 서로 다른 이미지를 **동시에** 처리합니다:

```

Thread 1: [Image 1] [Image 2] [Image 3] [Image 4] ...
Thread 2:           [Image 1] [Image 2] [Image 3] ...
Thread 3:                     [Image 1] [Image 2] ...
Thread 4:                               [Image 1] ...
```

GUI 진행률 표시줄에는 감지, 분석, 보정, 내보내기 등 4단계가 표시됩니다. 진행률 표시줄 위에 마우스를 올리면 스레드별 진행 상황을 확인할 수 있습니다.

{% hint style="success" %}
**Chloros+를 사용한 파이프라인 처리**는 하드웨어 및 데이터셋 크기에 따라 순차 처리보다 3~5배 더 빠를 수 있습니다. 고속 GPU와 SSD를 갖춘 시스템에서 속도 향상 효과가 가장 큽니다.
{% endhint %}

***

## 스레드 4 내보내기 진행 상황

Chloros 1.1.0에서는 내보내기 스레드(스레드 4)에 전용 진행 상황 추적 기능이 제공됩니다. 내보내기 진행 상황을 별도로 모니터링할 수 있습니다:**CLI:**
```bash
chloros-cli export-status
```

**SDK:**
```python
status = chloros.get_status()
print(f"Export: {status['export']['percent']}% - Phase: {status['export']['phase']}")
```

스레드 4가 100%에 도달하면 처리가 완료됩니다.

***

## 동적 컴퓨트 적응(Dynamic Compute Adaptation)과의 관계

[동적 컴퓨트 적응(Dynamic Compute Adaptation)](dynamic-compute-adaptation.md) 시스템은 주로 **스레드 3(처리)**에 영향을 미칩니다:

* **`GPU_PARALLEL`** 전략: 스레드 3은 `fused_gpu` 파이프라인을 사용하여 GPU를 통해 여러 이미지를 동시에 처리합니다
* **`GPU_SINGLE`** 전략: 스레드 3은 메모리 효율적인 `tiled_gpu` 파이프라인을 사용하여 한 번에 하나의 이미지를 처리합니다
* **`CPU_PARALLEL`** 전략: 스레드 3은 멀티스레드 병렬 처리를 사용하는 CPU 기반 처리를 수행합니다

스레드 1과 2가 완료됨에 따라 스레드 3의 GPU 메모리 할당도 동적으로 변경됩니다. 자세한 내용은 [동적 GPU 메모리 할당](dynamic-compute-adaptation.md#dynamic-gpu-memory-allocation)을 참조하십시오.

***

## 다음 단계

* [동적 컴퓨팅 적응](dynamic-compute-adaptation.md) — Chloros가 하드웨어에 최적화된 전략을 선택하는 방법
* [NVIDIA Jetson 가이드](../linux/nvidia-jetson-guide.md) — Jetson 플랫폼별 파이프라인 동작
* [처리 모니터링](../processing-images-gui/monitoring-the-processing.md) — GUI 진행 상황 모니터링
