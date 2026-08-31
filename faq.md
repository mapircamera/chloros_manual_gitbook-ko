---
description: Frequently Asked Questions
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/faq
---

# 자주 묻는 질문

<details>

<summary>Chloros를 사용하여 MAPIR 브랜드가 아닌 카메라의 이미지를 처리할 수 있나요?</summary>

아니요, Chloros는 MAPIR 카메라 이미지, 즉 Survey3 및 LATTICE 제품군의 이미지만 처리할 수 있습니다. 자세한 내용은 [지원되는 카메라 모델 목록](supported-cameras.md)을 참조해 주십시오. MAPIR Cloud에서는 다른 카메라의 처리도 제공하고 있으며, 전체 목록은 [여기](https://mapir.gitbook.io/mapir-cloud/supported-cameras)에서 확인하실 수 있습니다.

</details>

<details>

<summary>Chloros는 LATTICE 카메라를 지원합니까?</summary>

네. Chloros 1.2.0은 LATTICE M3C 및 M3M 카메라 모듈을 종단 간(end-to-end)으로 지원합니다: **실시간 제어**— GUI의 ‘카메라(Cameras)’ 탭에서 카메라 탐색, 연결, 미리보기 및 캡처가 가능하며, `chloros-cli lattice` 또는 Python SDK를 통해 PTP 시간 동기화를 지원하는 동기화된 멀티 카메라 어레이를 포함하여 — 그리고 캡처된 영상에 대한**완전한 방사계 처리** (원본 → 디베이이어링 → 복사도 → 반사율 → 지수)를 수행합니다. [지원되는 카메라](supported-cameras.md) 및 [LATTICE 가이드](lattice/README.md)를 참조하십시오.

</details>

<details>

<summary>교정 타겟 없이도 이미지의 반사율을 교정할 수 있나요?</summary>

**Survey3:** 아닙니다. 비표적 이미지를 촬영할 때 주변의 교정 표적 이미지를 함께 촬영하지 않으면, 이미지의 픽셀 값을 알려진 반사율 백분율과 연관 지을 수 없습니다. 또한 MAPIR 광 센서의 로그 데이터를 포함하지 않으면 주변광 스펙트럼이 측정되지 않아 반사율 결과가 정확하지 않을 것입니다.**LATTICE:** 예. 반사율은 패널 대신 DAQ 광 센서로 측정된 하향 조도(ρ = π·L/E)를 기준으로 삼을 수 있습니다. QA 기준을 통과한 프레임 내 타겟이 *존재할* 경우, 해당 타겟은 기본적으로 절대 기준이 됩니다(`--reflectance-source auto`). 단, 한 가지 예외가 있습니다. &quot;F988 반사율은 장면 내 반사율 패널을 사용하여 보정됩니다. 해당 대역은 DAQ 광 센서의 보정 범위를 벗어나므로, Chloros는 가장 최근의 패널 캡처 데이터를 적용하고 패널 관측 간에도 이를 유지합니다.&quot; [보정 타겟](calibration-targets.md)을 참조하십시오.

</details>

<details>

<summary>DAQ 광 센서가 필요한가요?</summary>

복사도의 경우 필요하지 않습니다: LATTICE 복사도 내보내기 데이터는 각 카메라의 공장 출하 시 수행된 복사계 보정에서 비롯되며, DAQ 센서나 타겟이 필요하지 않습니다. **반사율**의 경우 주변광에 대한 기준이 필요합니다. 이는 DAQ 광 센서의 하향 측정값이거나 프레임 내 보정 타겟일 수 있습니다. DAQ 센서를 사용하면**장면에 패널을 배치하지 않고도** 보정된 반사도를 생성할 수 있습니다. 기록된 `.daq` 파일은 타임스탬프를 통해 이미지와 자동으로 매칭됩니다. [보정 타겟](calibration-targets.md) 및 [CLI 참조 자료](reference/cli-reference.md)를 참조하십시오.

</details>

<details>

<summary>Chloros를 AI 어시스턴트(Claude, ChatGPT 등)와 함께 사용할 수 있나요?</summary>

네 — 이 설명서와 CLI/SDK는 이를 위해 제작되었습니다:

* AI 어시스턴트가 모든 페이지를 검색할 수 있도록 전체 설명서 색인이 `https://mapir.gitbook.io/chloros/llms.txt`에서 제공됩니다.
* 각 페이지의 원본 마크다운은 소문자 페이지 주소(예: URL)에 `.md`를 추가한 형태(예: `https://mapir.gitbook.io/chloros/reference/cli-reference.md`)로 제공됩니다.
* [CLI 참조 문서](reference/cli-reference.md)와 [SDK 참조](reference/sdk-reference.md)는 LLM이 활용할 수 있도록 작성되었습니다: 정확한 플래그, 기본값, 종료 의미론, 복사하여 붙여넣을 수 있는 명령어 등이 포함되어 있습니다.

어시스턴트를 Chloros로 설정하는 방법은 [AI 어시스턴트](ai-assistants.md)를 참조하십시오.

</details>

<details>

<summary>처리된 출력 파일은 어디에 저장되나요?</summary>

출력 파일은 프로젝트 폴더 아래에 카메라별로, 그 다음 파일 형식별로 그룹화되어 저장됩니다:

```
<project>/<camera-folder>/<format-folder>/<Product>_Images/
```

* **카메라 폴더** — LATTICE의 경우 `LATT-<sensor>-<lens>-F<filter>`, Survey3의 경우 `<model>_<filter>` (예: `Survey3N_RGN`)
* **포맷 폴더** — `tiff16`, `tiff8`, `png8`, `jpg8` 또는 `tiff32`
* **제품 폴더** — `Reflectance_Calibrated_Images/`, `Debayered_Images/`, `Preview_Images/`, `Radiance_Images/` (항상 `tiff32` 아래에 위치), `<INDEX>_Index_Images/`**내보낸 파일은 원본 파일의 이름을 그대로 유지합니다 — 폴더가 제품을 식별하며, 파일명 확장자는 아닙니다.**CLI의 경우, `-o`를 전달하지 않는 한 프로젝트 폴더는 입력 폴더 옆에 생성됩니다. 제품을 요청했으나 아무것도 출력하지 않은 `chloros-cli process` 실행은 `Processing finished but wrote no image products.`를 출력하고**0이 아닌 값으로 종료**되므로, 스크립트에서 이를 감지할 수 있습니다. [출력 이미지 형식](output-image-formats.md) 및 [CLI 참조](reference/cli-reference.md)를 참조하십시오.

</details>

<details>

<summary>Chloros에서 처리하기 전에 이미지를 편집할 수 있나요?</summary>

아니요. Chloros는 입력 데이터가 수정되지 않았다고 가정합니다. 파일 이름을 변경하지 마십시오.

</details>

<details>

<summary>MAPIR 및 Survey3 카메라를 자동 노출로 설정하고 Chloros에서 이미지를 처리할 수 있습니까?</summary>

아니요. Survey3 이미지 데이터셋은 노출이 고정/잠겨 있어야 하므로, 자동 셔터 속도나 자동 ISO를 사용할 수 없습니다. 동일한 카메라 모델의 모든 이미지는 동일한 셔터 속도와 ISO(노출)를 가져야 합니다.

LATTICE 카메라에는 이러한 제한이 없습니다. Chloros는 노출을 실시간으로 제어하며(스마트 AE), 각 촬영 시 실제로 사용된 노출과 게인을 기록하므로, 방사계 파이프라인에서 이를 반영합니다.

</details>

<details>

<summary>Chloros로 정사모자이크 이미지를 처리하거나 분석할 수 있습니까?</summary>

아니요. MAPIR 카메라의 개별 이미지만 지원되며, 정사모자이크 지도와 같은 스티칭된 이미지는 지원되지 않습니다.

</details>

<details>

<summary>Chloros의 대상 감지 단계를 어떻게 가속화할 수 있습니까?</summary>

파일 브라우저 테이블의 오른쪽 열에서 대상 이미지를 미리 선택하면 Chloros가 해당 이미지만에서 보정 대상을 찾도록 지시하여 처리 속도를 크게 높일 수 있습니다.

</details>

<details>

<summary><a href="https://www.mapir.camera/collections/software/products/mapir-cloud-subscription">MAPIR </a>Cloud에 이미지를 업로드할 경우, 업로드 전에 Chloros에서 처리해야 합니까?</summary>

당사의 온라인 처리 플랫폼 [MAPIR Cloud](https://www.mapir.camera/collections/software/products/mapir-cloud-subscription)에 업로드할 계획이라면, 업로드 전에 이미지를 편집하지 마십시오. Cloud에서 동일한 처리 과정은 물론 그 이상의 처리도 수행해 드립니다.

</details>

<details>

<summary>MAPIR에서 X 기능을 지원하게 될 예정인가요? MAPIR에 X 기능이 추가되면 정말 좋겠습니다.</summary>

저희는 제품에 대한 피드백을 언제나 환영합니다. 제품에서 문제를 발견하셨거나 개선에 대한 제안이 있으시면 [문의하기](https://www.mapir.camera/community/contact)를 통해 의견을 공유해 주시기 바랍니다. 저희 R&amp;D 활동의 대부분은 고객의 가장 시급한 요구 사항을 경청하는 데 중점을 두고 있습니다.

</details>

<details>

<summary>Chloros를 Linux에서 사용할 수 있나요?</summary>

네! Chloros 1.2.0은 `.deb` 패키지를 통해 Linux amd64(x86_64) 및 arm64 (NVIDIA Jetson JetPack 6)를 지원합니다. CLI 및 Python SDK는 라이브 LATTICE 카메라 및 DAQ 센서 제어를 포함하여 Linux에서 완벽하게 지원됩니다. Linux용 GUI는 제공되지 않습니다. — 모든 상호 작용은 [CLI](CLI.md) 또는 [Python SDK](api-python-sdk.md)를 통해 이루어집니다. 자세한 내용은 [Linux 개요](linux/linux-overview.md)를 참조하십시오.

</details>

<details>

<summary>NVIDIA Jetson에서 Chloros를 실행할 수 있나요?</summary>

네! Chloros는 JetPack 6이 실행되는 Jetson Nano, Orin Nano, Orin NX 및 AGX Orin을 포함한 NVIDIA Jetson 플랫폼을 지원합니다. Chloros는 사용자의 Jetson 모델을 자동으로 감지하고 처리 전략을 최적화합니다. 설정 및 배포 지침은 [NVIDIA Jetson 가이드](linux/nvidia-jetson-guide.md)를 참조하십시오.

</details>

<details>

<summary>Chloros는 내 하드웨어에 맞게 자동으로 최적화되나요?</summary>

네! Chloros에는 CPU, GPU, RAM 및 (Jetson의 경우) 열 센서를 자동으로 감지하는 [동적 연산 적응(Dynamic Compute Adaptation)](processing-architecture/dynamic-compute-adaptation.md) 기능이 포함되어 있습니다. 그런 다음, 대용량 메모리 시스템의 `GPU_PARALLEL`부터 리소스가 제한된 장치의 `GPU_SINGLE`, NVIDIA GPU가 없는 시스템의 `CPU_PARALLEL`에 이르기까지 최적의 처리 전략을 선택합니다. 수동 구성은 필요하지 않습니다.

</details>

<details>

<summary>4스레드 처리 파이프라인이란 무엇입니까?</summary>

Chloros는 Chloros+ 사용자를 위해 4스레드 파이프라인 아키텍처를 사용합니다. 스레드 1(탐지)은 이미지를 불러와 보정 대상을 탐지하고, 스레드 2(보정)은 반사율 보정을 계산하며, 스레드 3(처리)은 GPU 가속 디베이어링 및 인덱스 계산을 수행하고, 스레드 4(내보내기)는 출력 파일을 작성합니다. 최대 처리량을 위해 여러 이미지를 서로 다른 스레드에서 동시에 처리할 수 있습니다. 자세한 내용은 [처리 파이프라인](processing-architecture/processing-pipeline.md)을 참조하십시오.

</details>

<details>

<summary>Chloros 설치 환경에서 진단 테스트를 실행하려면 어떻게 해야 합니까?</summary>

`selftest` 명령어를 사용하여 7단계 스모크 테스트를 실행하십시오: 버전, 포트 가용성, 백엔드 시작, API 연결 상태(`/api/test`), 시스템 정보 (`/api/system-info` — GPU/CUDA/PyTorch), 노이즈 제거기 모델 존재 여부, CUDA 및 노이즈 제거기 준비 상태:

```bash
chloros-cli selftest
```

이는 특히 Linux/Jetson 시스템에서 GPU 및 CUDA 설정을 검증하는 데 유용합니다.

</details>
