# 반사율 워크플로우

DAQ 광센서는 방사계 측정한 영상을 반사율로 변환합니다. 두 가지 서로 다른 워크플로우가 있습니다:

1. **단일 센서** — 카메라가 촬영하는 동안 하나의 DAQ가 하향 복사조도를 측정하고, Chloros가 카메라의 복사도를 해당 기준값으로 나누어 계산합니다.
2. **이중 센서** — 두 개의 DAQ 센서 중 하나는 하늘을, 다른 하나는 물체를 관측하여 카메라를 사용하지 않고도 실시간 분광 반사율 곡선을 생성합니다.

## 단일 센서 + 카메라 (하향 기준)

DAQ는 하향 광 센서(DLS) 역할을 합니다: 카메라는 상향 복사도 **L**(W/m²/sr/nm)을 측정하고, DAQ는 하향 조도**E**(W/m²/nm)를 측정하며, Chloros는 다음과 같이 대역별 반사율을 계산합니다:

> ρ = π · L / E

DAQ의 측정값은 항상 **노출 시간과 타임스탬프가 일치**합니다. 이것이 바로 DAQ와 카메라가 PTP 기반 클럭을 공유하는 이유입니다([DAQ-E 네트워킹 및 시간 동기화](ethernet-ptp.md) 참조). 야외 작업 시 선샤인 코사인 캡을 장착하고 올바르게 선언하십시오. 캡 선언은 E 값을 직접 스케일링합니다([캡 프로파일 및 보정 범위](caps-and-range.md) 참조). 정량적 작업을 수행할 때는 기기 특성을 기억하십시오: 정량적 조도는 최소 15초 동안의 측정값 평균으로 산출됩니다.

### 실시간 캡처

‘카메라(Cameras)’ 탭에서 DAQ를 카메라에 바인딩하십시오. 각 카메라의 설정 패널에는 ‘광 센서(Light Sensors)’ 탭에 연결된 모든 DAQ(DAQ-U/M/E)가 나열된 **광 센서(Light Sensor)** 드롭다운 메뉴가 있습니다. 동기화된 어레이의 경우, 어레이 전체에 적용되는 광 센서 선택 사항이 모든 구성원에게 반영됩니다 (개별 카메라는 여전히 이 설정을 무시할 수 있습니다). 연결이 완료되면 센서의 스펙트럼 데이터가 카메라의 DLS 슬롯으로 전송되며, 반사율 출력값은 일치하는 측정값으로 나눈 값이 됩니다.

<!-- SCREENSHOT-NEEDED: Cameras tab per-camera settings panel showing the "Light Sensor" dropdown open, with a connected DAQ sensor listed and selected. -->

알아두면 좋은 두 가지 동작은 다음과 같습니다:

* **DAQ가 바인딩되지 않은 경우 → 반사율 데이터는 거부되며, 임의의 값으로 대체되지 않습니다.** Chloros는 반사율 산출값을 거부하고, 품질이 낮은 산출값을 아무런 오류 메시지 없이 반환하는 대신 건너뛴 사유를 기록합니다.
* **사용된 측정값은 보존됩니다.** 모든 반사율 프레임에 대해, 실제로 적용된 DAQ 측정값은 이미지 옆에 `.daq` 사이드카로 기록되므로, 나중에 캡처 데이터를 재처리할 수 있습니다([기록 및 .daq 형식](recording.md)).

### 기록된 이미지 처리

비행 후 처리를 위해 세션 중에 `.daq`를 기록하고 이를 이미지와 함께 보관하십시오. 파이프라인은 타임스탬프가 일치하는 하향 복사 데이터를 자동으로 해결하며, 첫 사용 시 MAPIR의 클라우드에서 누락된 공장 보정 데이터를 가져옵니다. GUI 녹화 파일은 녹화가 종료되면 열려 있는 프로젝트에 자동으로 추가됩니다.

반사율 기준은 처리 시점에 선택할 수 있습니다. `--reflectance-source`를 `chloros-cli process`에서 선택하거나, GUI의 프로젝트 설정에서 반사율 소스 설정을 선택할 수 있습니다:

| 값 | 동작 |
| --- | --- |
| `auto` (기본값) | QA를 통과한 인프레임 보정 타겟이 절대 기준이며, DAQ 하향 복사(ρ = π·L/E)가 대체 기준입니다 |
| `daq` | DAQ 기준 |
| `target` | 엄격한 프레임 내 타겟; DAQ 대체 없음 |

타겟 워크플로는 [보정 타겟](../calibration-targets.md)을, [LATTICE 장](../lattice/README.md) 및 전체 처리 파이프라인에 대한 [CLI 참조 자료](../reference/cli-reference.md)를 참조하십시오. 내보낸 반사율 픽셀을 읽을 때는 스탬프가 찍힌 스케일을 사용하십시오(LATTICE: 32768 = ρ 1.0, XMP `Chloros:PixelScale`; Survey3: 65535)를 사용하십시오 — [출력 이미지 형식](../output-image-formats.md)을 참조하십시오.

### DAQ의 보정 범위를 벗어난 대역

DAQ의 방사계 보정 범위는 ~374–974 nm입니다. Chloros는 해당 범위 내에서 스펙트럼 가중치의 절반 미만을 차지하는 카메라 대역에 대해서는 DAQ 기반 반사율 계산을 거부하며, 건너뛰기 사유로 `dls-uncalibrated-band-<nm>`를 보고합니다. 현재 출하 중인 SKU 중에서는 F988에만 이 문제가 적용됩니다. F988의 반사율은 현장 반사율 패널을 사용하여 보정되는데, 해당 대역이 DAQ 광 센서의 보정 범위를 벗어나기 때문에 Chloros는 가장 최근의 패널 캡처 값을 적용하고 패널 관측 간에도 이를 유지합니다. F988 카메라가 DAQ 전용으로 실행되는 경우, Chloros는 해당 대역에 대한 DAQ 기반 반사율을 거부하며, 건너뛰기 사유는 `dls-uncalibrated-band-988`입니다. 패널 워크플로가 지원되는 경로입니다.

## 듀얼 센서(주변광 + 피사체)

두 개의 DAQ 센서(어떤 조합이든, 어떤 전송 방식을 통하든)는 카메라 없이도 실시간 반사율 스펙트럼을 제공합니다. 한 센서는 하늘을 향하고(**주변광 소스**), 다른 하나는 피사체를 향하며(**피사체 스캐너**), Chloros는 파장별로 다음과 같이 계산합니다:

> R(λ) = 피사체(λ) / 주변광(λ)

(주변광 ≤ 0인 경우 0).

### GUI에서

&#x27;광 센서&#x27; 탭에 두 센서가 모두 연결된 상태에서, 센서 추가 오버레이(그리드 보기의 차트 타일 내 &#x27;+&#x27; 버튼)를 열고 **주변광 + 피사체 결합**을 선택합니다. &#x27;주변광 소스(Ambient Light Source)&#x27; 및 &#x27;물체 스캐너(Object Scanner)&#x27; 드롭다운 메뉴에서 두 센서를 선택한 후 &#x27;생성(Create)&#x27;을 클릭합니다. 이 그룹은 별도의 차트로 표시되며, 녹색**REF** 배지가 달린 사이드바 행으로도 나타납니다.

<!-- SCREENSHOT-NEEDED: The add-sensor overlay's "Combine Ambient + Object" panel with two connected DAQ sensors selected in the Ambient Light Source and Object Scanner dropdowns, Create button enabled. -->

<!-- SCREENSHOT-NEEDED: A live Apparent Reflectance chart from an Ambient+Object DAQ pair in list view, with the vegetation-index table visible below the chart (NDVI etc. showing live values). -->

반사율 차트(목록 보기) 아래에는 실시간 **식생 지수 표**가 표시되며, 이 표는 청색 450 / 녹색 550 / 적색 670 / NIR 800 nm의 대역 중심을 사용하여 곡선으로부터 지수를 계산합니다. 절대 척도를 상쇄하는 비율 기반 지수(NDVI, GNDVI, ENDVI, WDRVI, GRVI, CVI, GCI, MSR)는 항상 표시됩니다. 절대 반사율이 필요한 지수(EVI, SAVI, OSAVI, GSAVI, GOSAVI, MSAVI2, RDVI, TDVI, LAI, NLI, MNLI, FCI, GEMI)는 두 센서 모두 전력 보정 모델인 경우에만 나타납니다.

### 겉보기(Apparent) 대 상대(Relative) — 라벨 지정 규칙

Chloros는 센서 쌍이 실제로 주장할 수 있는 값에 따라 듀얼 센서 출력에 레이블을 지정합니다:

| 센서 쌍 | 레이블 |
| --- | --- |
| 두 센서 모두 보정됨 — 공장 보정 번들 로드됨 | **겉보기 반사율** |
| 센서 중 하나라도 보정되지 않은 경우 | **상대 반사율** |

세 모델 모두 방사측정 방식입니다: 센서의 공장 보정 번들이 로드되면 스펙트럼 값은 절대적인 W/m²/nm 단위가 되므로, 보정된 센서 쌍은 절대적인 겉보기 반사율에 대한 비율을 나타냅니다 — 전송 방식이 이를 결정하지 않습니다. 여전히 원시 카운트 데이터를 스트리밍 중인 센서(번들에 접근 불가)가 있는 경우, 결과는 상대적 곡선으로 격하되지만(스펙트럼 형태는 여전히 유효함). 두 센서 모두 올바르게 선언된 캡을 포함해야 합니다([캡 프로파일 및 보정 범위](caps-and-range.md)).

### Python에서

통합된 SDK 인터페이스에는 전용 듀얼 센서 호출이 없습니다. `chloros_sdk.connect_daq_sensor()`로 두 세션을 열고, 동일한 레이블 지정 규칙을 적용하여 두 센서의 `latest()` 스펙트럼을 직접 비교하십시오. (MAPIR의 내부 직접 하드웨어 인터페이스에도 듀얼 센서 기록 도구가 존재하며, 완전성을 위해 [CLI 참조](../reference/cli-reference.md)에 기재되어 있습니다. 이 도구는 출하된 CLI의 구성 요소가 아니며, 위에서 설명한 GUI 워크플로가 지원되는 실행 경로입니다.)
