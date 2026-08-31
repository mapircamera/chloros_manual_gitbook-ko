# DAQ 광 센서

> **하드웨어 정보를 찾고 계신가요?**센서 자체(모델, 장착 방식, 캡, 포트, 전원 및 SCANNER 앱)에 대한 내용은**[DAQ 사용자 설명서](https://mapir.gitbook.io/daq)**에 설명되어 있습니다. 이 장에서는 Chloros부터의 사용법에 대해 다룹니다.

MAPIR의 **DAQ** 광 센서는 주변광을 방사계측적으로 보정된 스펙트럼으로 측정합니다. Chloros에서는 다음과 같은 두 가지 역할을 수행합니다:

* **독립형 스펙트럼 측정 기기** — 실시간 스펙트럼 차트, 색도계 데이터 및 `.daq` 기록은 모두 [광 센서 탭](gui.md)에서 확인할 수 있으며, [CLI](cli-quick-start.md) 또는 Python SDK에서 모두 확인할 수 있습니다.
* **반사율을 위한 하향 조도 소스** — 처리 과정에서 Chloros는 사용자의 `.daq` 측정값을 각 캡처의노출 타임스탬프에 맞춰 보간하며, 측정된 하향 조명을 사용하여 카메라의 복사도를 반사율(`--reflectance-source daq`)로 변환합니다. 보정된 밴드의 경우 장면 내 패널이 필요하지 않습니다.

<!-- SCREENSHOT-NEEDED: product photo of the DAQ-U, DAQ-M, and DAQ-E units side by side, each with its Sunshine cosine-corrector cap fitted (request from hardware team — no repo asset exists) -->

***

## 세 가지 모델, 하나의 데이터 형식

| 모델 | 전송 방식 | 검색 방식 |
| --- | --- | --- |
| **DAQ-U** | USB (직렬) | 직렬 포트 스캔 |
| **DAQ-M** | 블루투스 저에너지 | 이름으로 BLE 스캔 |
| **DAQ-E** | 이더넷 (IPv4, PoE 전원 공급) | mDNS `_daq-e._tcp` (호스트 이름 `daq-e-<id>.local`) |

세 모델 모두 동일한 유선 프로토콜을 사용하며, 동일한 데이터를 제공합니다:

* 매 프레임마다 **340~1010 nm 범위에서 5 nm 간격으로 측정된 135점 스펙트럼**과 CIE XYZ 삼자극값이 포함됩니다.
* **W/m²/nm 단위의 방사계측적으로 보정된 스펙트럼 조도** — 데이터가 사용자에게 도달하기 전에 각 장치의 공장 보정 번들(및 활성 캡 보정 프로파일)이 적용됩니다.
* 동일한 **`.daq` 기록 형식**(SQLite 파일). 파일을 생성한 전송 방식에 관계없이 후속 처리 과정은 동일합니다.

전송 스택(USB 직렬, BLE, mDNS/zeroconf)은 Chloros 백엔드 내에 번들로 포함되어 있습니다. — GUI나 CLI의 `pool-*` 명령어를 통해 세 가지 모델 중 어느 것과도 통신하기 위해 별도로 설치할 필요가 없습니다.

***

## 보정 범위: 보고된 범위 340–1010 nm, 보정된 범위 ~374–974 nm

센서는 전체 340–1010 nm 그리드를 보고하지만, NIST 추적 가능한 방사계 이득 범위는 대략 **374–974 nm**에 해당합니다. Chloros는 스펙트럼 가중치의 절반 미만이 해당 보정 범위에 속하는 카메라 대역에 대해서는 절대 반사율 나눗셈을 거부하며, 건너뛴 대역은 건너뛴 사유 `dls-uncalibrated-band-<nm>`와 함께 보고됩니다.

출하 중인 LATTICE 필터 SKU 중 **F988**만 이 문제에 해당합니다:

F988의 반사율은 현장 반사율 패널을 사용하여 보정됩니다. 해당 대역은 DAQ 광 센서의 보정된 범위를 벗어나므로, Chloros는 가장 최근의 패널 캡처 데이터를 적용하고 패널 관측 간에도 이를 유지합니다.

DAQ 데이터만 있는 상태에서 F988 캡처를 처리할 경우, Chloros는 해당 대역에 대한 DAQ 기반 반사율을 거부하며, 건너뛰기 사유로 `dls-uncalibrated-band-988`를 반환합니다. [반사율 패널 워크플로](../calibration-targets.md)가 F988에서 지원되는 경로입니다.

***

## 센서 ID

모든 DAQ는 안정적인 센서 ID를 보고합니다. ID의 형식은 모델에 따라 다릅니다:

| 모델 | ID 형식 | 예시 |
| --- | --- | --- |
| DAQ-U | 5-옥텟, 하이픈 포함 | `CB-7C-A8-2E-5F` |
| DAQ-M | 하이픈으로 구분된 5-옥텟 | `CB-74-02-30-6B` |
| DAQ-E | `daq-e-<6 hex digits>` | `daq-e-def330` |

센서 ID는 다음과 같습니다:

* 기록하는 모든 `.daq` 파일에 각인되어 있으며,
* Chloros가 해당 장치의 공장 보정 번들을 가져오는 데 사용하는 키이며,
* CLI 및 `pool-*` 명령어에서 `--sensor-id`에 전달하는 값이며,
* DAQ-E의 경우, mDNS 호스트 이름(`daq-e-def330.local`)도 포함됩니다. 이는 `--eth-host`가 수락하는 값입니다.

***

## 공장 보정 및 클라우드

모든 DAQ 장치는 NIST 추적 가능한 방사계측 체인을 통해 개별적으로 공장 보정되며, Chloros는 각 장치의 센서 ID를 기준으로 한 보정 번들을 불러옵니다. 단위별 교정 보고서(PDF)는 [광 센서 탭](gui.md)의 센서 설정에서 다운로드할 수 있습니다.

{% hint style="warning" %}
**DAQ-U 및 DAQ-M은 보정을 위해 클라우드 접속이 필요합니다.**두 모델 모두 기기에 데이터를 저장하지 않습니다. 공장 출하 시 보정 번들은 MAPIR의 클라우드에 저장되어 있으며, 센서 ID를 통해 불러온 후(로컬에 캐시됨) 사용됩니다. Chloros는 DAQ-U 또는 DAQ-M에서 보정된 W/m²/nm 데이터를 전송하기 위해 인터넷 연결이 필요합니다.**DAQ-E는 예외입니다** — 이 모델은 보정 데이터를 기기 내에 저장합니다.

<!-- PRE-PUBLISH-CHECK: LAUNCH item 3 (DAQ-M end-to-end connect smoke) was still unverified as of 2026-08-16 — re-confirm the DAQ-M cloud-calibration flow on the release build before publishing this page. -->

{% endhint %}***

## 기록 데이터의 저장 위치

| 표면 | `.daq`의 기본 저장 위치 |
| --- | --- |
| GUI — 광 센서 탭 | `<project folder>/light_sensor/` (완료된 기록은 프로젝트에 자동으로 추가됨) |
| CLI — `daq pool-record` | 백엔드를 실행하는 머신의 `~/Documents/DAQ Live View/` |

모든 `.daq` 파일 이름에는 센서 ID와 타임스탬프가 포함됩니다.

***

## 이 장에서

* [**Chloros의 DAQ 탭**](gui.md) — 전체 GUI 안내: 각 모델 연결, 센서별 설정, 스펙트럼 차트, 실시간 색도 데이터, 듀얼 센서 반사율 및 기록.
* [**CLI 빠른 시작 (pool-\*)**](cli-quick-start.md) — 지원되는 명령줄 경로인 `chloros-cli daq pool-*`에서 DAQ 센서 구동 방법.
* [**캡 프로파일 및 보정된 범위**](caps-and-range.md) — 모델별 캡 종류, 캡 선언 방법 및 보정된 스펙트럼 범위에 대한 상세 설명.
* [**기록 및 .daq 형식**](recording.md) — `.daq` SQLite 형식 및 기록 워크플로우.
* [**DAQ-E 네트워킹 및 시간 동기화**](ethernet-ptp.md) — DAQ-E 전송 모드 및 PTP 시간 동기화.
* [**반사율 워크플로우**](reflectance.md) — DAQ 하향 데이터(downwelling data)를 사용하여 반사율을 산출하는 방법.
* 플래그 수준에 대한 전체 문서는 [CLI 참조](../reference/cli-reference.md) (`chloros-cli daq` 섹션) 및 [SDK 참조 문서](../reference/sdk-reference.md) (`chloros_sdk.connect_daq_sensor()`)를 참조하십시오. 두 문서 모두 AI 어시스턴트가 직접 활용할 수 있도록 작성되었습니다.
