# DAQ 광 센서

> **하드웨어 정보를 찾고 계신가요?**센서 자체(모델, 장착 방식, 캡, 포트, 전원 및 SCANNER 앱)에 대한 내용은**[DAQ 사용자 설명서](https://mapir.gitbook.io/daq)**에 설명되어 있습니다. 이 장에서는 Chloros에서 이를 사용하는 방법에 대해 다룹니다.

MAPIR의 **DAQ** 광 센서는 주변광을 방사계측학적으로 보정된 스펙트럼으로 측정합니다. Chloros에서 이 센서들은 두 가지 역할을 수행합니다:

* **독립형 스펙트럼 측정 기기** — 실시간 스펙트럼 차트, 색도계 데이터 및 `.daq` 기록은 모두 [광 센서 탭](gui.md), [CLI](cli-quick-start.md) 또는 Python SDK를 통해 모두 제공됩니다.
* **반사율 계산을 위한 하향 조도 소스** — 처리 과정에서, Chloros는 `.daq` 측정값을 각 캡처의 노출 타임스탬프에 맞춰 보간하고, 측정된 하향 조명을 사용하여 카메라의 복사도를 반사율(`--reflectance-source daq`)로 변환합니다. 보정된 밴드의 경우 장면 내 패널이 필요하지 않습니다.

<!-- SCREENSHOT-NEEDED: product photo of the DAQ-U, DAQ-M, and DAQ-E units side by side, each with its Sunshine cosine-corrector cap fitted (request from hardware team — no repo asset exists) -->

***

## 세 가지 모델, 하나의 데이터 형식

| 모델 | 전송 방식 | 검색 방식 |
| --- | --- | --- |
| **DAQ-U** | USB (직렬) | 직렬 포트 스캔 |
| **DAQ-M** | 블루투스 저에너지 | 이름으로 BLE 스캔 |
| **DAQ-E** | 이더넷 (IPv4, PoE 전원 공급) | mDNS `_daq-e._tcp` (호스트 이름 `daq-e-<id>.local`) |**DAQ-E-S (1.3.0 버전부터 추가됨).** 고정형 디퓨저와 기울기 센서가 장착된 DAQ-E입니다. Chloros에서는 이를 `DAQ-E-S`로 표시하며, 캡 보정이 적용되지 않고 모든 측정값에 장치의 기울기, 롤, 피치 정보가 포함된다는 점을 제외하면 DAQ-E와 완전히 동일하게 작동합니다. [DAQ-E-S: 고정형 디퓨저 및 기울기](caps-and-range.md#daq-e-s-fixed-diffuser-and-tilt)를 참조하십시오.

이 세 모델은 모두 동일한 와이어 프로토콜을 사용하며, 동일한 데이터를 제공합니다:

* **5 nm 간격으로 340~1010 nm 범위의 135점 스펙트럼**과 CIE XYZ 삼자극값이 모든 프레임에 포함됩니다.
* **W/m²/nm 단위의 방사계측적으로 보정된 스펙트럼 조도** — 데이터가 사용자에게 전달되기 전에 각 장치의 공장 보정 번들(및 활성 캡 보정 프로파일)이 적용됩니다.
* 동일한 **`.daq` 기록 형식**(SQLite 파일). 파일을 생성한 전송 방식에 관계없이 후속 처리 과정은 동일합니다.

전송 스택(USB 직렬, BLE, mDNS/zeroconf)은 Chloros 백엔드 내에 번들로 포함되어 있습니다. GUI나 CLI의 `pool-*` 명령어를 통해 세 가지 모델 중 어느 것과도 통신하기 위해 별도로 설치할 필요가 없습니다.

***

## 보정 범위: 보고된 범위 340–1010 nm, 보정된 범위 ~374–974 nm

센서는 전체 340–1010 nm 그리드를 보고하지만, NIST 추적 가능한 방사계 이득 범위는 대략 **374–974 nm**에 해당합니다. Chloros는 보정된 범위 내의 스펙트럼 가중치의 절반 미만을 차지하는 카메라 대역에 대해서는 절대 반사율 나눗셈을 거부하며, 건너뛴 대역은 건너뜀 사유 `dls-uncalibrated-band-<nm>`와 함께 보고됩니다.

출하 중인 LATTICE 필터 SKU 중 **F988**만 이 영향이 적용됩니다:

F988의 반사율은 현장 반사율 패널을 사용하여 교정됩니다. 해당 대역은 DAQ 광 센서의 교정 범위를 벗어나므로, Chloros는 가장 최근의 패널 캡처 데이터를 적용하고 패널 측정 간에는 해당 값을 유지합니다.

DAQ 데이터만 있는 상태에서 F988 캡처가 처리되는 경우, Chloros는 해당 밴드에 대해 DAQ 기반 반사율을 거부하며, 건너뛰기 사유는 `dls-uncalibrated-band-988`입니다. — [반사율 패널 워크플로](../calibration-targets.md)가 F988에 대해 지원되는 방법입니다.

***

## 센서 ID

모든 DAQ는 안정적인 센서 ID를 보고합니다. ID의 형식은 모델에 따라 다릅니다:

| 모델 | ID 형식 | 예시 |
| --- | --- | --- |
| DAQ-U | 5-옥텟, 하이픈 구분 | `CB-7C-A8-2E-5F` |
| DAQ-M | 5-옥텟, 하이픈 포함 | `CB-74-02-30-6B` |
| DAQ-E | `daq-e-<6 hex digits>` | `daq-e-def330` |

센서 ID는 다음과 같습니다:

* 기록되는 모든 `.daq` 파일에 각인되어 있으며,
* Chloros가 해당 장치의 공장 보정 번들을 가져오는 데 사용하는 키이며,
* CLI의 `pool-*` 명령어에서 `--sensor-id`에 전달하는 값이며,
* DAQ-E의 경우, mDNS 호스트 이름(`daq-e-def330.local`)이기도 하며, 이는 `--eth-host`가 수용하는 값입니다.

***

## 공장 보정 및 클라우드

모든 DAQ 장치는 NIST 추적 가능한 방사계열을 통해 개별적으로 공장 교정되며, Chloros는 각 장치의 센서 ID를 기준으로 한 교정 번들을 로드합니다. 단위별 교정 보고서(PDF)는 [광 센서 탭](gui.md)의 센서 설정에서 다운로드할 수 있습니다.

{% hint style="warning" %}
**DAQ-U 및 DAQ-M은 보정을 위해 클라우드 접속이 필요합니다.**두 모델 모두 기기에 데이터를 저장하지 않습니다. 공장 보정 번들은 MAPIR의 클라우드에 저장되어 있으며, 센서 ID를 통해 불러온 후(로컬에 캐시됨) 사용됩니다. Chloros는 DAQ-U 또는 DAQ-M에서 보정된 W/m²/nm 데이터를 전송하기 위해 인터넷 연결이 필요합니다.**DAQ-E는 예외입니다** — 이 모델은 보정 데이터를 기기 내에 저장하고 있습니다.
{% endhint %}
<!-- PRE-PUBLISH-CHECK: LAUNCH item 3 (DAQ-M end-to-end connect smoke) was still unverified as of 2026-08-16 — re-confirm the DAQ-M cloud-calibration flow on the release build before publishing this page. -->

***

## 기록 데이터의 저장 위치

| 표면 | 기본 `.daq` 저장 위치 |
| --- | --- |
| GUI — 광 센서 탭 | `<project folder>/light_sensor/` (완료된 기록은 프로젝트에 자동으로 추가됨) |
| CLI — `daq pool-record` | 백엔드가 실행 중인 컴퓨터의 `~/Documents/DAQ Live View/` (백엔드가 동일한 컴퓨터에 있는 경우, 상대 경로 `-o`는 명령을 실행한 폴더를 기준으로 해석됩니다) |

모든 `.daq` 파일 이름에는 센서 ID와 타임스탬프가 포함됩니다.

***

## 이 장의 내용

* [**Chloros의 DAQ 탭**](gui.md) — 전체 GUI 안내: 각 모델 연결, 센서별 설정, 스펙트럼 차트, 실시간 색도 데이터, 듀얼 센서 반사율 및 기록.
* [**CLI 빠른 시작 (pool-\*)**](cli-quick-start.md) — 지원되는 명령줄 경로인 `chloros-cli daq pool-*`를 통해 DAQ 센서 제어하기.
* [**캡 프로파일 및 보정 범위**](caps-and-range.md) — 모델별 캡 종류, 캡 선언 방법, 보정된 스펙트럼 범위에 대한 상세 설명.
* [**기록 및 .daq 형식**](recording.md) — `.daq` SQLite 형식 및 기록 워크플로우.
* [**DAQ-E 네트워킹 및 시간 동기화**](ethernet-ptp.md) — DAQ-E 전송 모드 및 PTP 시간 동기화.
* [**반사율 워크플로우**](reflectance.md) — DAQ 하향파 데이터를 사용하여 반사율을 산출하는 방법.
* 전체 플래그 수준 문서는 [CLI 참조](../reference/cli-reference.md) (섹션 `chloros-cli daq`) 및 [SDK 참조](../reference/sdk-reference.md) (`chloros_sdk.connect_daq_sensor()`)를 참조하십시오. 두 문서 모두 AI 어시스턴트가 직접 활용할 수 있도록 작성되었습니다.
