---
description: Frequently Asked Questions
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/faq
---

# 자주 묻는 질문

<details>

<summary>Chloros를 사용하여 MAPIR 브랜드가 아닌 카메라의 이미지를 처리할 수 있습니까?</summary>

아니요, Chloros는 MAPIR 카메라 이미지만 처리할 수 있습니다. 자세한 내용은 [지원되는 카메라 모델](supported-cameras.md) 목록을 참조하십시오. MAPIR Cloud에서는 다른 카메라의 이미지 처리도 제공하고 있습니다. 전체 목록은 [여기](https://mapir.gitbook.io/mapir-cloud/supported-cameras)에서 확인하십시오.

</details>

<details>

<summary>교정 타겟 없이도 이미지의 반사율을 교정할 수 있나요?</summary>

아니요. 비표적 이미지를 촬영할 때 주변의 교정 표적 이미지를 함께 촬영하지 않으면, 이미지의 픽셀 값을 알려진 반사율 백분율과 연관 지을 수 없습니다. 또한 MAPIR 광 센서의 로그 데이터를 포함하지 않으면 주변광 스펙트럼이 측정되지 않아 반사율 결과가 정확하지 않을 것입니다.

</details>

<details>

<summary>Chloros에서 처리하기 전에 이미지를 편집할 수 있나요?</summary>

아니요. Chloros는 입력 데이터가 수정되지 않았다고 가정합니다. 파일 이름을 변경하지 마십시오.

</details>

<details>

<summary>MAPIR 및 Survey3 카메라를 자동 노출로 설정하고 Chloros에서 이미지를 처리할 수 있습니까?</summary>

아니요. Survey3 이미지 데이터 세트는 노출이 고정/잠겨 있어야 하므로, 자동 셔터 속도나 자동 ISO를 사용할 수 없습니다. 동일한 카메라 모델의 모든 이미지는 동일한 셔터 속도와 ISO(노출)를 가져야 합니다.

</details>

<details>

<summary>Chloros에서 정사모자이크 이미지를 처리하거나 분석할 수 있나요?</summary>

아니요. 정사모자이크 지도와 같은 합성 이미지는 지원되지 않으며, 개별 MAPIR 카메라 이미지만 지원됩니다.

</details>

<details>

<summary>Chloros의 대상 탐지 단계를 어떻게 가속화할 수 있습니까?</summary>

파일 브라우저 테이블의 오른쪽 열에서 대상 이미지를 미리 선택하면 Chloros가 해당 이미지에서만 보정 대상을 찾도록 지시하여 처리 속도를 크게 높일 수 있습니다.

</details>

<details>

<summary><a href="https://www.mapir.camera/collections/software/products/mapir-cloud-subscription">MAPIR Cloud</a>에 이미지를 업로드할 경우, 업로드 전에 Chloros에서 처리해야 합니까?</summary>

당사의 온라인 처리 플랫폼 [MAPIR Cloud](https://www.mapir.camera/collections/software/products/mapir-cloud-subscription)에 업로드할 계획이라면, 업로드 전에 이미지를 편집하지 마십시오. Cloud에서 동일한 처리 과정은 물론 그 이상의 작업을 수행해 드립니다.

</details>

<details>

<summary>MAPIR에서 X 기능을 지원하게 될 예정인가요? MAPIR에 X 기능이 추가되면 정말 좋겠습니다.</summary>

저희는 항상 제품에 대한 피드백을 환영합니다. 제품에서 문제를 발견하셨거나 개선할 점이 있다면 [문의하기](https://www.mapir.camera/community/contact)를 통해 의견을 공유해 주십시오. 저희 연구개발(R&amp;D)의 대부분은 고객의 가장 큰 요구 사항을 경청하는 데서 방향을 잡습니다.

</details>

<details>

<summary>Chloros는 Linux에서 사용할 수 있나요?</summary>

네! Chloros 1.1.0은 `.deb` 패키지를 통해 Linux amd64(x86_64) 및 arm64(NVIDIA Jetson JetPack 6)를 지원합니다. CLI 및 Python SDK는 Linux에서 완벽하게 지원됩니다. Linux에는 GUI가 없으며, 모든 상호 작용은 [CLI](CLI.md) 또는 [Python SDK](api-python-sdk.md)를 통해 이루어집니다. 자세한 내용은 [Linux 개요](linux/linux-overview.md)를 참조하십시오.

</details>

<details>

<summary>NVIDIA Jetson에서 Chloros를 실행할 수 있습니까?</summary>

네! Chloros 1.1.0은 JetPack 6을 실행하는 Jetson Nano, Orin Nano, Orin NX 및 AGX Orin을 포함한 NVIDIA Jetson 플랫폼을 지원합니다. Chloros는 Jetson 모델을 자동으로 감지하고 처리 전략을 최적화합니다. 설정 및 배포 지침은 [NVIDIA Jetson 가이드](linux/nvidia-jetson-guide.md)를 참조하십시오.

</details>

<details>

<summary>Chloros가 내 하드웨어에 맞게 자동으로 최적화되나요?</summary>

네! Chloros 1.1.0에는 CPU, GPU, RAM 및 (Jetson의 경우) 열 센서를 자동으로 감지하는 [동적 컴퓨팅 적응(Dynamic Compute Adaptation)](processing-architecture/dynamic-compute-adaptation.md) 기능이 포함되어 있습니다. 그런 다음, 메모리가 많은 시스템에서는 `GPU_PARALLEL`를, 리소스가 제한된 장치에서는 `GPU_SINGLE`를, NVIDIA GPU가 없는 시스템에서는 `CPU_PARALLEL`를 선택하는 등 최적의 처리 전략을 선택합니다. 수동 구성은 필요하지 않습니다.

</details>

<details>

<summary>4스레드 처리 파이프라인이란 무엇입니까?</summary>

Chloros 1.1.0은 Chloros 이상 사용자를 위해 4스레드 파이프라인 아키텍처를 사용합니다: 스레드 1(탐지)은 이미지를 로드하고 보정 대상을 탐지하며, 스레드 2(보정)은 반사율 보정을 계산하고, 스레드 3(처리)은 GPU 가속 디베이링 및 인덱스 계산을 수행하며, 스레드 4(내보내기)는 출력 파일을 작성합니다. 최대 처리량을 위해 여러 이미지를 서로 다른 스레드에서 동시에 처리할 수 있습니다. 자세한 내용은 [처리 파이프라인](processing-architecture/processing-pipeline.md)을 참조하십시오.

</details>

<details>

<summary>Chloros 설치 환경에서 진단 기능을 실행하려면 어떻게 해야 합니까?</summary>

`selftest` 명령어를 사용하여 버전 확인, 포트 가용성, 백엔드 시작, API 연결성, 시스템 정보, 노이즈 제거 모델 및 CUDA 가용성을 포함한 7가지 시스템 진단 기능을 실행할 수 있습니다:

```bash
chloros-cli selftest
```

이는 특히 Linux/Jetson 시스템에서 GPU 및 CUDA 설정을 확인하는 데 유용합니다.

</details>
