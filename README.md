---
metaLinks: {}
---

# 시작하기

<div data-full-width="false"><figure><img src=".gitbook/assets/chloros_logo_transparent.png" alt=""><figcaption></figcaption></figure></div>

Chloros는 [MAPIR](https://www.mapir.camera)에서 제공하는 소프트웨어 애플리케이션으로, 다중 스펙트럼 이미지 처리, MAPIR 하드웨어 실시간 제어 및 센서 데이터 기록을 수행합니다. Chloros 1.3.0은 다음과 같은 전체 MAPIR제품군을 지원합니다:

* **Survey3 카메라** — RAW+JPG 캡처 데이터를 보정된 반사율 및 식생 지수 맵으로 처리합니다. [지원되는 카메라](supported-cameras.md)를 참조하십시오.
* **LATTICE 카메라** — GigE 다중 스펙트럼 카메라 모듈을 단일 또는 동기화된 다중 카메라 어레이 형태로 실시간으로 연결하여, 미리 보기, 촬영 및 보정된 복사도 및 반사도 제품으로 처리합니다. [LATTICE 섹션](lattice/README.md)을 참조하십시오.
* **DAQ 광 센서** — DAQ-U(USB), DAQ-M(블루투스), DAQ-E 및 DAQ-E-S(이더넷) 스펙트럼 센서: 실시간 보정된 스펙트럼, `.daq` 기록, 반사율 처리를 위한 하향 조명을 제공합니다. [DAQ 섹션](daq/README.md)을 참조하십시오.

{% hint style="success" %}
**Chloros 1.3.0의 새로운 기능**: AI 어시스턴트를 통해 `chloros-mcp`로 Chloros를 제어할 수 있으며, CLI, SDK 및 `chloros-mcp`에서도 동일한 기능을 제공합니다. LATTICE 인덱스 제품이 개선되었으며, 전체 센서 어레이의 처리 속도가 빨라지고 모든 네트워크에서 작동하며, 백엔드는 명령어 간에도 계속 실행됩니다. 아래의 ‘새로운 기능’ 목록을 확인하고, 설치 프로그램은 [다운로드](download.md)에서 받으세요.
{% endhint %}

{% hint style="info" %}
**AI 어시스턴트와 함께 Chloros를 사용하시나요?** 이 설명서는 이를 위해 제작되었습니다. 어시스턴트에게 다음을 지시하세요:

* `https://mapir.gitbook.io/chloros/llms.txt` — 모든 페이지의 기계 판독 가능 색인.
* 원시 마크다운 형식의 모든 페이지 — 해당 URL 끝에 `.md`를 추가하세요(예: `https://mapir.gitbook.io/chloros/reference/cli-reference.md`).
* [CLI 참조](reference/cli-reference.md) 및 [SDK 참조](reference/sdk-reference.md) — LLM이 활용할 수 있도록 작성된 완전하고 정확한 값을 담은 참조 페이지입니다.

프롬프트 예시: *"https://mapir.gitbook.io/chloros/reference/cli-reference.md,를 읽은 후, ~/flights/flight_001 폴더에 로그인하여 해당 폴더를 반사율 + NDVI GeoTIFF로 처리하는 스크립트를 작성하세요."*

또는 Chloros를 직접 실행하게 할 수도 있습니다: `chloros-mcp`는 SDK와 함께 설치된 MCP 서버로, Claude Desktop, Claude Code, Cursor 및 기타 MCP 클라이언트와 연동됩니다.

전체 가이드: [AI 어시스턴트와 함께 Chloros 사용하기](ai-assistants.md).
{% endhint %}

***

## Chloros 1.3.0의 새로운 기능

* **AI 어시스턴트가 Chloros를 제어할 수 있습니다.** Python SDK가 설치된 MCP 서버인 `chloros-mcp`를 사용하면 Claude Desktop, Claude Code, Cursor 및 기타 MCP 클라이언트에서 이미지를 처리하고 LATTICE 카메라, 어레이 및 DAQ 광 센서를 실행할 수 있습니다. [AI 어시스턴트와 함께 Chloros 사용하기](ai-assistants.md)를 참조하십시오.
* **CLI, SDK 및 `chloros-mcp`는 동일한 기능을 제공합니다.** 보안상의 이유로 로그인, DAQ-E 프로비저닝 및 직접 DAQ 서버 기능만 CLI에 남아 있습니다. 일부 1.2.0 스크립트는 변경이 필요합니다. [CLI 참조](reference/cli-reference.md) 및 [SDK 참조](reference/sdk-reference.md) 상단의 **

1.3.0에서 변경된 사항**을 참조하십시오.
* **LATTICE 지수 제품 개선.** 모든 처리 모드에서 LATTICE 캡처 데이터로 내보낸 NDVI 및 기타 지수 제품의 품질이 개선되었습니다.
* **Windows 설치 프로그램 옵션.** **시스템 설정**에서 설치 프로그램은**Windows 긴 경로 사용**및**Microsoft Defender 검사에서 Chloros 백엔드 제외** 옵션을 제공하며, 두 옵션 모두 기본적으로 선택되어 있습니다. 첫 번째 옵션은 프로젝트 폴더 경로가 260자 이상이어도 허용하며, 두 번째 옵션은 실시간 스캔으로 인해 카메라 캡처 속도가 저하되는 것을 방지합니다(고속 캡처 시 캡처 속도의 약 30% 감소).. 또한 설치 프로그램은 이제 `PATH`에 CLI 폴더를 한 번만 추가하고, 이전 설치 프로그램이 남긴 중복 항목을 제거합니다. [다운로드](download.md)를 참조하십시오.
* **깊은 프로젝트 폴더는 초기 단계에서 거부됩니다.** Windows 긴 경로 기능이 비활성화된 상태(Windows 기본 설정)에서는, 260자 경로 제한을 초과하는 너무 깊은 프로젝트 폴더가 실행되기 전에 거부되며, 제한 내에 들어맞는 가장 긴 프로젝트 폴더와 긴 경로 기능을 활성화하는 방법이 표시됩니다. 이전에는 이러한 경우 결과물이 생성되지 않았습니다. 긴 경로 기능이 켜진 상태에서는, 매우 깊은 폴더에 있는 산출물도 메타데이터(GPS 및 XMP 태그)를 유지합니다.
* **마스크 클리핑.** LATTICE 방사도 또는 반사도 산출물의 밴드 분리 과정에서 음수 밴드 값이 생성될 경우, 해당 값은 0으로 클리핑되며, 산출물 폴더 옆에 위치한 `clamp_masks/` 폴더 내의 픽셀별 마스크에 어떤 픽셀과 밴드가 영향을 받았는지 기록됩니다. 제품은 `Chloros:UnmixClampMask` 태그에서 해당 마스크의 이름을 지정합니다. 라이브 뷰는 이제 내보내기 시와 정확히 동일하게 클리핑됩니다. [출력 이미지 형식](output-image-formats.md)을 참조하십시오.
* **제품은 자체 정보를 기술합니다.** 인덱스 제품은 원본 원시 데이터의 처리 태그 대신 자체 처리 태그(`Chloros:ProcessingLevel=index`, 인덱스 표현식, 그리고 방사도 또는 반사도 중 어느 것을 기반으로 계산되었는지 여부)를 포함합니다. Survey3 제품은 모든 처리 모드에서 Chloros 처리 태그를 포함합니다. LATTICE 원본 및 디베이어링된 파일은 센서의 기본 방향을 유지하며 XMP에 방향을 기록합니다. 이미지 뷰어는 표시 목적으로만 파일을 회전시킵니다.
* **Linux 패키지는 압축 해제된 상태로 제공됩니다.** 백엔드와 CLI는 `/usr/lib/chloros/` 아래의 폴더로 설치되므로, 더 이상 실행할 때마다 자동으로 압축이 해제되지 않습니다. x86_64 환경에서 백엔드 시작에 약 100초가 소요되지 않으며, `.deb`의 크기는 이전의 약 절반으로 줄었습니다. 번들로 제공되는 ExifTool은 이제 새로 설치된 Ubuntu 환경에서도 실행됩니다. 또한 `.deb`는 시스템용 Python SDK도 함께 설치하며, `python3` (`sudo apt install ./chloros_1.3.0_<arch>.deb`로 설치), 카메라 스트리밍에 필요한 커널의 소켓 수신 버퍼를 늘리고, 사용자 권한으로 실행되는 백엔드 서비스를 추가합니다. Jetson 패키지인 `chloros_1.3.0_arm64_jp6.deb`는 JetPack 6용으로 빌드되었으며, JetPack의 CUDA 기반 PyTorch 빌드를 사용하여 Jetson의 GPU에서 처리를 수행합니다. Linux에서는 이제 내보내진 각 이미지가 디스크에 한 번씩 기록됩니다(메모리에 기록 및 태그 지정 후, 해당 위치로 복사됨). 디스크 속도가 작업 속도를 좌우하는 Jetson에서는 288장의 이미지를 처리하는 LATTICE 실행 시간이 24분이 아닌 14분이 소요됩니다. [Linux 설치](linux/linux-installation.md)를 참조하십시오.
* **LATTICE OCN 카메라가 햇빛 아래에서 더 정확하게 측정합니다.** 밴드 분리 후 방사도, 반사율 및 지수에 신속한 보정이 적용되므로, 햇빛 아래에서 회색 타겟의 측정값이 중립 NDVI에 가깝게 나옵니다. RGN 카메라는 변경 사항이 없습니다.
* **앱 내 버그 보고.** ‘디버그 로그’ 탭 상단의 버그 아이콘을 클릭하면 MAPIR로 오류 설명, 최대 10장의 이미지, 로그 및 시스템 세부 정보가 전송됩니다. [GUI : 탐색](navigation.md)을 참조하십시오.
* **LATTICE 어레이.** 어레이는 기본적으로 전체 센서 데이터를 유지하며, 압축된 12비트 전송 방식(`--packed`, 기본값 `auto`)을 사용하면 전체 센서 프레임 속도가 약 3분의 1 정도 향상됩니다. 어레이는 네트워크가 허용하는 프레임 속도로 어떤 네트워크에서든 작동합니다(속도가 느린 링크의 경우 어레이를 거부하거나 프레임 크기를 줄이는 대신 프레임 속도가 낮아집니다); 카메라의 전원을 껐다 켠 후 어레이는 PTP가 다시 활성화되면서 자동으로 복구됩니다; 저조도 환경에서 자동 노출이 더 이상 프레임 속도를 낮추지 않습니다(일정 기간을 채운 후 게인을 높입니다); 연속 캡처 기능이 고속 어레이의 속도를 따라갑니다; RGB 카메라용 **하이라이트** 제어 기능이 추가되었으며; SDK의 `ArraySession`는 `grab()`, `frame_stream()` 및 `continuous()`가 추가되었습니다. [LATTICE 섹션](lattice/README.md)을 참조하십시오.
* **더 안정적인 자동 노출.** 노출은 일정한 조도에서 유지되며, 조도가 변한 후 몇 초 이내에 안정화됩니다. 작은 하이라이트가 더 이상 장면을 어둡게 만들지 않으며, 수동 노출 시 게인은 '자동'으로 유지됩니다. 또한, 강한 햇빛 아래에서 반응이 멈추는 카메라는 광 센서가 감지한 조도 수준, 또는 광 센서가 연결되지 않은 경우 카메라 자체의 프레임 정보를 바탕으로 해당 조도 수준에서 유지됩니다. [카메라 설정](lattice/camera-settings.md)을 참조하십시오.
* **광 센서.** **DAQ-E-S 지원:** 고정형 디퓨저는 캡 보정이 필요하지 않으며, 각 측정값에는 장치의 틸트, 롤, 피치 정보가 포함됩니다. 녹화 파일에는 캡이 수동으로 선택되었는지 또는 자동으로 추정되었는지 표시되며, 처리 과정에서 추정된 캡에 대해서는 경고가 표시됩니다. DAQ-U, DAQ-M 및 DAQ-E는 캡이 포함되지 않은 상태로 출하되므로, 장착된 캡의 이름을 지정해야 합니다. 실시간 `.daq` 기록은 정전 시에도 유지되며(`.daq-journal` 파일에 `.daq` 정보를 함께 보관하십시오), 데이터 전송이 중단된 DAQ-E는 기록에 표시된 간격을 기준으로 다시 연결됩니다. 광센서 기록만 포함된 프로젝트는 앱과 CLI를 통해 해당 기록을 나열하고 처리하며, `chloros_scripts`로 생성된 기록은 생성한 DAQ로 나열되며, 사용자가 설정할 수 있는 캡이 적용됩니다. 모든 캡처는 사용된 DAQ 판독값을 저장합니다. [DAQ 섹션](daq/README.md)를 참조하십시오.
* **처리.**LATTICE 프레임은 대상 감지 중에 처리를 시작합니다. 대상 및 DAQ 기준은 대역별로 결정됩니다. 이동된 프로젝트는 파일을 다시 연결합니다. 8비트 캡처의 내보내기 파일은 8비트이며, 새로운**TIFF (8비트)** 형식을 사용합니다. 보정 타겟 없이 실행된 Survey3는 반사율 이미지가 아닌 대체 결과물(`Vignette_Corrected_Images` 또는 `Sensor_Response_Images`)로 저장됩니다. 반사율 이미지의 대역별 DLS 태그는 항상 해당 프레임의 것입니다. LATTICE 캡처의 경우 ‘Texture Aware’ 기능이 거부되며, 항상 표준 디베이어 처리가 적용됩니다.
* **CLI 및 SDK.** 상대 출력 경로는 이제 명령어나 스크립트를 실행한 폴더로 지정됩니다(이전에는 설치된 백엔드가 자체 설치 폴더에서 경로를 해결하려다 오류가 발생했습니다). 백엔드는 명령어와 스크립트 사이에서 계속 실행되며, 30분 동안 사용되지 않으면 자동으로 중지됩니다. `chloros-cli backend status` 및 `backend stop`가 이를 관리합니다. 캡처 명령어는 쓰기 실패한 모든 파일의 이름을 나열하고 코드 1로 종료되며, 드라이브 용량이 가득 차면 반복 캡처가 중지됩니다. `process` 실행이 성공하면 `Image products written: N (plus M clamp masks)`가 보고됩니다. Windows에서는 SDK의 `ChlorosLocal` / `process_folder`는 설치된 백엔드를 찾아(이미 실행 중인 백엔드를 사용하며), `read_image_audit_tags`는 일반 `pip install`에서 작동합니다.

***

## Chloros 1.2.0의 새로운 기능

* **실시간 카메라 제어 — 새로운 ‘카메라’ 탭.** LATTICE 카메라를 하나씩 또는 동기화된 다중 카메라 어레이(PTP 시간 동기화, 하드웨어 트리거 캡처)로 연결할 수 있으며, 실시간 미리보기 오버레이, 대역별 히스토그램, 스마트 자동 노출, 실시간 인덱스 계산기, 앱 내 카메라 펌웨어 업데이트 기능을 제공합니다.
* **광 센서 — 새로운 ‘광 센서’ 탭.** DAQ-U(USB), DAQ-M(블루투스), DAQ-E(이더넷) 센서를 연결하고; 보정된 실시간 스펙트럼(W/m²/nm)을 확인하고, `.daq` 파일을 프로젝트에 기록하며, 캡 보정 프로필을 선택하고, 네트워크를 통해 DAQ-E 펌웨어를 업데이트할 수 있습니다.
* **캡처 모드 및 레코더.** 단일 / 연속 / 간격 캡처와 원시 데이터 전용 ‘가장 빠른 캡처(Fastest Capture)’ 모드; 프로젝트별로 ‘모두 캡처(Capture All)’ 시 생성될 카메라 및 내보내기 유형 선택; 모니터링 등급 인덱스 비디오 및 분석 등급 원시 버스트를 위한 어레이 레코더와 오프라인 비디오 빌드 기능.
* **LATTICE 처리 파이프라인.** LATTICE 캡처 폴더를 가져와 각 원시 프레임을 디베이어링, 미리보기, float32 방사도(W/m²/sr/nm) 및 반사도 산출물로 분할하며, 산출물별로 토글 기능을 제공합니다. 반사율은 프레임 내 보정 타겟 또는 DAQ 다운웰링에서 얻을 수 있으며, 내보내기 시 어레이 정렬이 적용됩니다. 누락된 공장 보정 데이터는 카메라 일련번호를 통해 자동으로 다운로드됩니다.
* **프로젝트는 하드웨어 정보를 기억합니다.** 연결된 카메라와 광 센서는 프로젝트(`cameras.json` / `sensors.json`)와 함께 저장되며, 프로젝트를 다시 열면 저장된 설정으로 재연결됩니다. [GUI : 프로젝트](projects.md)를 참조하십시오.
* **이미지 뷰어 기능 개선.** 파일별 정확한 반사율 스케일링을 적용한 커서 픽셀/인덱스 판독, 레이어 히스토그램, GSD 비닝 슬라이더, 트리거별/카메라별 그리드 모드, LATTICE 제품 보기, 인덱스/LUT 샌드박스 데이터를 디스크로 내보내기 기능이 추가되었습니다.
* **CLI 및 SDK가 대폭 확장되었습니다.** 새로운 `lattice`, `daq pool-*`, `project` 및 `time-sync` 명령어 계열; 새로운 `process` 옵션(`--input-level`, 제품별 토글, `--reflectance-source`, 배열 정렬 플래그); 백엔드를 자동으로 시작하는 SDK 스마트 커넥트 핸들(`connect_camera` / `connect_array` / `connect_daq_sensor`); `open_project()` 자동화; 해당 SDK 휠은 설치 프로그램에 번들로 포함되어 있으며, `chloros-sdk`라는 이름으로 PyPI에 공개됩니다.
* **명확한 오류 처리 방식.** 제품을 요청했으나 아무것도 작성하지 않은 `chloros-cli process` 실행은 이제 명확한 오류 메시지를 표시하며 0이 아닌 종료 코드를 반환합니다. 성공적인 실행의 경우 작성된 이미지 제품의 수를 보고합니다.
* **새로운 출력 레이아웃.** 산출물은 `<project>/<camera>/<format>/<Product>_Images/` 폴더에 저장되며 원본 파일명을 유지합니다. 즉, 파일명 접미사가 아닌 폴더명이 산출물을 식별합니다. [출력 이미지 형식](output-image-formats.md)을 참조하십시오.
* **더 많은 입력 소스, 계획 및 언어 지원.** `.dng` 입력 지원; 총 38개 인터페이스 언어가 모두 완벽하게 지원됨; 계획별 하드웨어 제한으로, 무료(로그인 불필요) 사용 시 최대 4대의 카메라와 2개의 광 센서까지 사용 가능.
* **신뢰성.** ‘처리 중지’ 시 정확한 실행 요약과 함께 깔끔하게 종료되며, 다중 카메라 프로젝트에서는 모든 카메라 데이터를 내보낼 수 있고, 설치 프로그램 업그레이드 시 더 이상 로그아웃되지 않습니다.***

Chloros는 다음 3가지 애플리케이션 환경에서 사용할 수 있습니다:

## Chloros: 데스크톱 GUI 애플리케이션

라이브 ‘카메라’ 및 ‘광 센서’ 탭을 포함한 모든 기능을 갖춘 독립형 별도 창입니다. _Windows 전용._

## [Chloros CLI: 명령줄 인터페이스](CLI.md)

명령줄 일괄 처리 기능과 실시간 `lattice`, `daq pool-*`, `project`, `time-sync` 명령어를 지원합니다. 자동화, 스크립팅 및 헤드리스 운영에 이상적입니다. **Windows, Linux amd64 및 Linux arm64(NVIDIA Jetson)**에서 사용할 수 있습니다. _CLI
를 이용하려면 유료 Chloros+ 등급이 필요합니다._

## [Chloros API: Python SDK](api-python-sdk.md)

자동화 및 사용자 지정 워크플로우를 위한 프로그래밍 방식의 Python 인터페이스: 전체 파이프라인 처리, 실시간 카메라/어레이 세션, DAQ 센서 세션, 저장된 프로젝트 자동화. 데스크톱/CLI 패키지와 함께 설치되며, `pip install chloros-sdk`로도 공개됩니다. _API를 이용하려면 유료 Chloros+ 등급이 필요합니다._

***

## 지원 플랫폼

| 플랫폼 | GUI | CLI | Python SDK |
| --- | --- | --- | --- |
| **Windows 10/11 (x64)** | 예 | 예 | 예 |
| **Linux amd64 (x86_64)** | 아니요 | 예 | 예 |
| **Linux arm64 (NVIDIA Jetson)** | 아니요 | 예 | 예 |

Linux 설치 방법은 [Linux 및 엣지 컴퓨팅](linux/linux-overview.md) 섹션을 참조하십시오.

***

## 3단계로 시작하기

1. **설치** — 사용 중인 플랫폼에 맞는 설치 프로그램을 다운로드하여 실행하세요. [다운로드](download.md)를 참조하세요.
2. **로그인 (GUI의 경우 선택 사항)** — GUI는 계정 없이도 무료로 이미지를 처리합니다. [Chloros+ 로그인](chloros+-login.md)을 하면 병렬 처리, GPU 가속, 더 높은 장치 제한, CLI/SDK 액세스를 이용할 수 있습니다.
3. **첫 번째 프로젝트 만들기** — Chloros를 열고, [새 프로젝트](projects.md)를 생성한 다음, [이미지를 추가](processing-images-gui/adding-files-to-a-project.md)하고 [처리를 시작](processing-images-gui/starting-the-processing.md)하세요. 대신 실시간 하드웨어를 제어하려면 ‘카메라’ 또는 ‘광 센서’ 탭을 여세요 — [GUI : 탐색](navigation.md)을 참조하세요.

***

## Chloros+

Chloros는 대부분의 작업에 무료로 사용할 수 있지만, 더 많은 기능을 원하실 수도 있습니다. 이때 Chloros+ 유료 라이선스가 도움이 될 수 있습니다. Chloros+ 라이선스를 사용하면 다음과 같은 새로운 기능을 이용할 수 있습니다:

* **멀티스레드 처리**: 파이프라인을 통해 이미지를 동시에 처리함으로써 대규모 프로젝트의 이미지 처리 속도를 대폭 향상시킵니다.
* **GPU (CUDA) 가속**: 최신 고사양 GPU 메모리 옵션을 활용하여 이미지 처리 파이프라인의 속도를 더욱 높일 수 있습니다. 최상의 결과를 얻으려면 4GB 이상의 VRAM을 권장합니다.
* **Chloros+**[**CLI**](CLI.md) **접근**: 명령줄에서 Chloros+를 실행하여 자동화하고 사용자만의 소프트웨어에 통합할 수 있습니다. 모든 유료 요금제에서 이용 가능하며, 서버 측에서 적용됩니다.
* **Chloros+**[**API**](api-python-sdk.md) **접근:** Python에서 Chloros+를 실행하여 프로그래밍 방식으로 제어함으로써, 연구 파이프라인, 데이터 분석 워크플로우 및 사용자 지정 애플리케이션과 원활하게 통합할 수 있습니다. 모든 유료 요금제에서 이용 가능하며, 서버 측에서 적용됩니다.
* **더 높은 하드웨어 제한**: 한 번에 더 많은 카메라와 광 센서를 연결할 수 있습니다. 로그인 없이 GUI를 통해 최대 4대의 카메라와 2개의 DAQ 광 센서를 연결할 수 있으며, 유료 요금제를 이용하면 두 제한 모두 상향됩니다:

| 요금제 | 카메라 | DAQ 광 센서 |
| --- | --- | --- |
| Iron (무료, 로그인 불필요) | 4 | 2 |
| Copper / Bronze | 6 | 3 |
| Silver | 10 | 6 |
| Gold | 20 | 12 |

* **다중 기기 사용**: 각 Chloros+ 라이선스로 2대 이상의 기기를 등록할 수 있습니다. MAPIR Cloud 계정을 사용하여 등록된 기기를 관리하세요. Chloros+ 라이선스를 업그레이드하여 더 많은 기기를 지원받을 수 있습니다.
* **고급 텍스처 인식 디베이어링 방식:** 고품질의 에지 인식 디베이어링 기술과 AI/ML 노이즈 제거 모델을 결합하여 디베이어링 과정에서 발생하는 노이즈를 거의 모두 제거합니다.
* **사용자 정의 다중 스펙트럼 지수 공식:** Chloros래스터 계산기에 사용자 정의 다중 스펙트럼 지수를 입력하여, 이미지 처리 및 이미지 미리보기 샌드박스 모두에서 활용할 수 있습니다.
* **Linux 및 엣지 컴퓨팅:** 현장 및 엣지 처리를 위해 NVIDIA Jetson을 포함한 Linux x86_64 및 ARM64 플랫폼에서 Chloros를 실행할 수 있습니다. [Linux 개요](linux/linux-overview.md)를 참조하십시오.

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary" data-icon="envira">Chloros+ 가격 및 가입</a></p>

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_zoom.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_mode.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_meta.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: cli.JPG shows the 1.1.0 CLI banner. Re-shoot a terminal running `chloros-cli --version` + `chloros-cli status` on the 1.3.0 build so the banner prints "Chloros CLI 1.3.0". -->
