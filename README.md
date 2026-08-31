---
metaLinks: {}
---

# 시작하기

<div data-full-width="false"><figure><img src=".gitbook/assets/chloros_logo_transparent.png" alt=""><figcaption></figcaption></figure></div>Chloros는 [MAPIR](https://www.mapir.camera)에서 제공하는 소프트웨어 애플리케이션으로, 다중 스펙트럼 이미지 처리, MAPIR 하드웨어의 실시간 제어 및 센서 데이터 기록을 수행합니다. Chloros 1.2.0은 전체 MAPIR 제품군을 지원합니다:

* **Survey3 카메라** — RAW+JPG 캡처 데이터를 보정된 반사율 및 식생 지수 맵으로 처리합니다. [지원되는 카메라](supported-cameras.md)를 참조하십시오.
* **LATTICE 카메라** — GigE 다중 스펙트럼 카메라 모듈을 단독으로 또는 동기화된 다중 카메라 어레이로 실시간 연결하여, 미리 보기, 촬영 및 보정된 복사도 및 반사도 결과물로 처리합니다. [LATTICE 섹션](lattice/README.md)을 참조하십시오.
* **DAQ 광 센서** — DAQ-U(USB), DAQ-M(블루투스) 및 DAQ-E(이더넷) 스펙트럼 센서: 실시간 보정된 스펙트럼, `.daq` 기록 및 반사율 처리를 위한 하향 조도. [DAQ 섹션](daq/README.md)을 참조하십시오.

{% hint style="success" %}
**Chloros 1.2.0의 새로운 기능**: 실시간 LATTICE 카메라 및 어레이 제어, DAQ 광센서 통합, 캡처 모드 및 레코더, 완전한 LATTICE 방사계 처리 파이프라인, CLI/SDK를 통한 프로젝트 자동화 등 다양한 기능이 추가되었습니다. 아래의 ‘새로운 기능’ 목록을 확인하시고, 변경 내역을 보려면 [다운로드](download.md)를 클릭하세요.
{% endhint %}

{% hint style="info" %}
**AI 어시스턴트와 함께 Chloros를 사용하시나요?** 이 설명서는 그런 용도로 제작되었습니다. 어시스턴트에게 다음을 지시해 보세요:

* `https://mapir.gitbook.io/chloros/llms.txt` — 모든 페이지의 기계 판독 가능 색인.
* 원본 마크다운 형식의 모든 페이지 — 해당 페이지의 URL 뒤에 `.md`를 추가하세요(예: `https://mapir.gitbook.io/chloros/reference/cli-reference.md`).
* [CLI 참조](reference/cli-reference.md) 및 [SDK 참조](reference/sdk-reference.md) — LLM이 활용할 수 있도록 작성된 완전하고 정확한 값을 담은 참조 페이지입니다.

프롬프트 예시: *&quot;https://mapir.gitbook.io/chloros/reference/cli-reference.md,를 읽은 후, ~/flights/flight_001 폴더에 로그인하여 해당 폴더를 반사도 + NDVI GeoTIFF 파일로 처리하는 스크립트를 작성하세요.&quot;*

전체 가이드: [AI 어시스턴트와 함께 Chloros 사용하기](ai-assistants.md).
{% endhint %}

***

## Chloros 1.2.0의 새로운 기능

* **실시간 카메라 제어 — 새로운 ‘카메라’ 탭.** LATTICE 카메라를 개별적으로 또는 동기화된 다중 카메라 어레이(PTP 시간 동기화, 하드웨어 트리거 캡처)로 연결할 수 있으며, 실시간 미리보기 오버레이, 대역별 히스토그램, 스마트 자동 노출, 실시간 인덱스 계산기, 앱 내 카메라 펌웨어 업데이트 기능을 제공합니다.
* **광 센서 — 새로운 ‘광 센서’ 탭.** DAQ-U(USB), DAQ-M(블루투스), DAQ-E(이더넷) 센서를 연결하고; 보정된 실시간 스펙트럼(W/m²/nm)을 확인하고, `.daq` 파일을 프로젝트에 기록하며, 캡 보정 프로필을 선택하고, 네트워크를 통해 DAQ-E 펌웨어를 업데이트할 수 있습니다.
* **캡처 모드 및 레코더.** 단일 / 연속 / 간격 캡처와 원본 전용 ‘가장 빠른 캡처(Fastest Capture)’ 모드; 프로젝트별로 ‘모두 캡처(Capture All)’ 시 생성될 카메라 및 내보내기 유형 선택; 모니터링 등급 인덱스 비디오 및 분석 등급 원본 버스트를 위한 어레이 레코더와 오프라인 비디오 빌드 기능.
* **LATTICE 처리 파이프라인.** LATTICE 캡처 폴더를 가져와 각 원시 프레임을 디베이어링, 미리보기, float32 방사도(W/m²/sr/nm) 및 반사율 산출물로 분할하며, 산출물별로 토글 기능을 제공합니다. 반사율은 프레임 내 보정 타겟 또는 DAQ 다운웰링에서 도출될 수 있으며, 내보내기 시 어레이 정렬이 적용됩니다. 누락된 공장 보정 데이터는 카메라 일련번호를 통해 자동으로 다운로드됩니다.
* **프로젝트는 하드웨어 정보를 기억합니다.** 연결된 카메라와 광 센서는 프로젝트(`cameras.json` / `sensors.json`)와 함께 저장되며, 프로젝트를 다시 열면 저장된 설정으로 재연결됩니다. [GUI : 프로젝트](projects.md)를 참조하십시오.
* **이미지 뷰어 기능 개선.** 파일별 정확한 반사율 스케일링을 적용한 커서 픽셀/인덱스 판독, 레이어 히스토그램, GSD 비닝 슬라이더, 트리거별/카메라별 그리드 모드, LATTICE 제품 보기, 인덱스/LUT 샌드박스 파일 디스크 내보내기 기능이 추가되었습니다.
* **CLI 및 SDK, 기능이 대폭 확장되었습니다.** 새로운 `lattice`, `daq pool-*`, `project` 및 `time-sync` 명령어 계열; 새로운 `process` 옵션(`--input-level`, 제품별 토글, `--reflectance-source`, 배열 정렬 플래그); 백엔드를 자동으로 시작하는 SDK 스마트 커넥트 핸들(`connect_camera` / `connect_array` / `connect_daq_sensor`); `open_project()` 자동화; SDK 휠은 설치 프로그램에 번들로 포함되어 있으며, PyPI에 `chloros-sdk`로 게시됩니다.
* **정직한 오류 처리 방식.** 제품을 요청했으나 아무것도 작성하지 않은 `chloros-cli process` 실행은 이제 명확하게 오류 메시지를 표시하고 0이 아닌 종료 코드를 반환합니다. 성공적인 실행은 작성된 이미지 제품의 수를 보고합니다.
* **새로운 출력 레이아웃.** 생성된 제품은 `<project>/<camera>/<format>/<Product>_Images/` 폴더에 저장되며 원본 파일 이름을 그대로 유지합니다. 제품을 식별하는 것은 파일 이름 확장자가 아닌 폴더 이름입니다. [출력 이미지 형식](output-image-formats.md)을 참조하십시오.
* **더 많은 입력 지원, 플랜 및 언어.** `.dng` 입력 지원; 총 38개 인터페이스 언어가 모두 지원됨; 플랜별 하드웨어 제한으로, 무료(로그인 없음) 사용 시 최대 4대의 카메라와 2개의 광 센서까지 사용 가능.
* **신뢰성.** ‘처리 중지’ 시 정확한 실행 요약과 함께 깔끔하게 종료되며, 다중 카메라 프로젝트에서 모든 카메라 데이터를 내보낼 수 있고, 설치 프로그램 업그레이드 시 더 이상 로그아웃되지 않습니다.***

Chloros는 다음 3가지 애플리케이션 환경에서 사용할 수 있습니다:

## Chloros: 데스크톱 GUI 애플리케이션

라이브 ‘카메라’ 및 ‘광 센서’ 탭을 포함한 모든 기능을 갖춘 독립형 별도 창입니다. _Windows 전용._

## [Chloros CLI: 명령줄 인터페이스](CLI.md)

명령줄 일괄 처리 기능과 실시간 `lattice`, `daq pool-*`, `project`, `time-sync` 명령어를 지원합니다. 자동화, 스크립팅 및 헤드리스 운영에 적합합니다. **Windows, Linux amd64 및 Linux arm64 (NVIDIA Jetson)**에서 사용할 수 있습니다. _CLI를 사용하려면 유료 Chloros+ 등급이 필요합니다._

## [Chloros API: Python SDK](api-python-sdk.md)

자동화 및 사용자 지정 워크플로우를 위한 프로그래밍 방식의 Python 인터페이스: 전체 파이프라인 처리, 실시간 카메라/어레이 세션, DAQ 센서 세션 및 저장된 프로젝트 자동화. desktop/CLI 패키지와 함께 설치되며, `pip install chloros-sdk`로도 배포됩니다. _API에 액세스하려면 유료 Chloros+ 등급이 필요합니다._

***

## 지원 플랫폼

| 플랫폼 | GUI | CLI | Python SDK |
| --- | --- | --- | --- |
| **Windows 10/11 (x64)** | 예 | 예 | 예 |
| **Linux amd64 (x86_64)** | 아니요 | 예 | 예 |
| **Linux arm64 (NVIDIA Jetson)** | 아니요 | 예 | 예 |

Linux 설치 지침은 [Linux 및 엣지 컴퓨팅](linux/linux-overview.md) 섹션을 참조하십시오.

***

## 3단계로 시작하기

1. **설치** — 사용 중인 플랫폼용 설치 프로그램을 다운로드하여 실행하세요. [다운로드](download.md)를 참조하세요.
2. **로그인 (GUI의 경우 선택 사항)** — GUI는 계정 없이도 무료로 이미지를 처리합니다. [Chloros+ 로그인](chloros+-login.md)을 하면 병렬 처리, GPU 가속, 더 높은 장치 제한, 그리고 CLI/SDK에 대한 접근 권한을 제공합니다.
3. **첫 번째 프로젝트 생성** — Chloros를 열고, [새 프로젝트](projects.md)를 생성하고, [이미지를 추가](processing-images-gui/adding-files-to-a-project.md)한 다음, [처리를 시작](processing-images-gui/starting-the-processing.md)하세요. 대신 실제 하드웨어를 제어하려면 ‘카메라’ 또는 ‘광 센서’ 탭을 여십시오 — [GUI : 탐색](navigation.md)을 참조하십시오.***

## Chloros+

Chloros는 대부분의 작업에 무료로 사용할 수 있지만, 더 많은 기능이 필요할 수도 있습니다. 이때 Chloros+ 유료 라이선스가 유용할 수 있습니다. Chloros+ 라이선스를 사용하면 다음과 같은 새로운 기능을 이용할 수 있습니다:

* **멀티스레드 처리**: 파이프라인을 통해 이미지를 동시에 처리함으로써 대규모 프로젝트의 이미지 처리 속도를 대폭 향상시킵니다.
* **GPU (CUDA) 가속**: 최신 고사양 GPU 메모리 옵션을 활용하여 이미지 처리 파이프라인의 속도를 한층 더 높일 수 있습니다. 최상의 결과를 얻으려면 4GB 이상의 VRAM을 권장합니다.
* **Chloros+**[**CLI**](CLI.md)**액세스**: 명령줄에서 Chloros+를 실행하여 자동화하고 사용자만의 소프트웨어에 통합할 수 있습니다. 모든 유료 요금제에서 이용 가능하며, 서버 측에서 적용됩니다.
* **Chloros+**[**API**](api-python-sdk.md)**사용 방법:** Python에서 Chloros+를 실행하여 프로그래밍 방식으로 제어함으로써, 연구 파이프라인, 데이터 분석 워크플로우 및 사용자 지정 애플리케이션과 원활하게 통합할 수 있습니다. 모든 유료 요금제에서 이용 가능하며, 서버 측에서 적용됩니다.
* **더 높은 하드웨어 제한**: 한 번에 더 많은 카메라와 광 센서를 연결할 수 있습니다. 로그인하지 않은 상태에서 GUI를 통해 최대 4대의 카메라와 2개의 DAQ 광 센서를 연결할 수 있으며, 유료 요금제를 이용하면 두 제한 모두 상향됩니다:

| 요금제 | 카메라 | DAQ 광 센서 |
| --- | --- | --- |
| Iron (무료, 로그인 불필요) | 4 | 2 |
| Copper / Bronze | 6 | 3 |
| Silver | 10 | 6 |
| Gold | 20 | 12 |

* **다중 기기 사용**: 각 Chloros+ 라이선스로 2대 이상의 기기를 등록할 수 있습니다. MAPIR 클라우드 계정을 사용하여 등록된 기기를 관리하세요. Chloros+ 라이선스를 업그레이드하여 더 많은 기기를 지원하세요.
* **고급 텍스처 인식 디베이어링 방식:** AI/ML 노이즈 제거 모델과 결합된 고품질 에지 인식 디베이어링 기술로, 디베이어링 과정에서 발생하는 노이즈를 거의 모두 제거합니다.
* **사용자 정의 다중 스펙트럼 지수 공식:** Chloros 래스터 계산기에 사용자 정의 다중 스펙트럼 지수를 입력하여, 이미지 처리 및 이미지 미리보기 샌드박스 모두에서 사용할 수 있습니다.
* **Linux 및 엣지 컴퓨팅:** 현장 및 엣지 처리를 위해 NVIDIA Jetson을 포함한 Linux x86_64 및 ARM64 플랫폼에서 Chloros를 실행할 수 있습니다. [Linux 개요](linux/linux-overview.md)를 참조하십시오.

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary" data-icon="envira">Chloros+ 가격 및 가입</a></p>

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_zoom.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_mode.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_meta.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: cli.JPG shows the 1.1.0 CLI banner. Re-shoot a terminal running `chloros-cli --version` + `chloros-cli status` on the 1.2.0 build so the banner prints "Chloros CLI 1.2.0". -->
