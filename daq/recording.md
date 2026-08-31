# 기록 및 .daq 형식

`.daq` 파일은 Chloros의 광센서 기록 형식으로, 하나의 DAQ 센서에서 수집된 보정된 스펙트럼 프레임으로 구성된 **SQLite 데이터베이스**입니다. 캡처 세션 중에 이 파일을 하나 기록해 두면, 반사율 파이프라인이 나중에 각 이미지를 정확히 그 순간에 측정된 하향 복사조도로 나눌 수 있습니다.

## .daq 파일의 구성 요소

| 속성 | 값 |
| --- | --- |
| 컨테이너 | SQLite 데이터베이스, 기록당 센서당 하나의 파일 |
| 파일명 | **센서 ID**및**타임스탬프**를 포함, 예: `daq_data_daq-e-def330_2026_04_13_18h30m00.daq` |
| 프레임당 스펙트럼 | 135개 데이터 포인트, 340–1010 nm 범위에서 5 nm 간격, CIE XYZ 삼자극 값 포함 |
| 단위 | 보정된 스펙트럼 조도, **W/m²/nm** (공장 보정 번들 + 캡 프로파일 적용) |
| 기록된 메타데이터 | 센서 ID(해당 장치의 공장 보정 데이터를 불러오는 데 필요한 키) 및 적용된 캡 프로파일 — [캡 프로파일 및 보정 범위](caps-and-range.md) 참조 |

이 형식은 DAQ-U, DAQ-M, DAQ-E에서 모두 동일하므로, 후속 처리 단계에서는 어떤 전송 장치로 기록되었는지 상관하지 않습니다.

보정된 기록을 위해서는 센서의 공장 보정 번들이 필요합니다. DAQ-U 및 DAQ-M의 경우, 백엔드가 센서 ID를 기반으로 MAPIR의 클라우드에서 번들을 가져옵니다(가져올 수 없는 경우 기록이 거부됨). DAQ-E 장치는 기기에 보정 정보를 저장하고 있으므로 이 과정이 필요하지 않습니다.

## GUI를 통한 기록

GUI에서 기록을 수행하려면 **열린 프로젝트**가 필요합니다(그렇지 않으면 기록 버튼이 비활성화됩니다):

* **모두 기록 / 모두 중지** — &#x27;광 센서&#x27; 사이드바 상단; 연결된 모든 센서에서 `.daq` 기록을 한 번에 시작하거나 중지합니다.
* **기록 / 기록 중지** — 센서별, 톱니바퀴 설정 모달에서 사용 가능합니다. 기록 중에는 센서의 실시간 정보 행에 빨간색 &quot;REC&quot; 표시기가 나타납니다.

파일은 `<project>/light_sensor/`에 저장되며, 녹화가 중지되면(중지, 모두 중지, 또는 녹화 센서 연결 해제 등 어떤 방식이든 상관없이) 완료된 `.daq` 파일이 **열려 있는 프로젝트에 자동으로 추가**됩니다. 수동으로 추가하는 단계 없이 프로젝트의 파일 목록에 표시되며, 반사율 처리를 위해 이미 준비된 상태입니다.

<!-- SCREENSHOT-NEEDED: Light Sensors tab with one DAQ sensor connected and recording: sidebar showing the red "Stop All" state of the Record All button, the sensor row, and the settings modal open with the red "REC" indicator visible in the live info rows. -->

<!-- SCREENSHOT-NEEDED: File Browser / project file list immediately after stopping a DAQ recording, showing the .daq file auto-added to the open project alongside imagery. -->

## CLI에서 녹화하기

CLI는 백엔드의 센서 풀을 통해 기록합니다(백엔드가 실행 중이어야 합니다. 이 명령어들은 경량 HTTP 클라이언트입니다):

```bash
# Connect the sensor into the backend pool
chloros-cli daq pool-connect --eth-host daq-e-def330.local

# Record for 150 seconds, with a human-friendly device label
chloros-cli daq pool-record --sensor-id daq-e-def330 --duration 150 \
    -o ./out --device-name "rooftop-A"

# Or run open-ended and stop explicitly
chloros-cli daq pool-record --sensor-id daq-e-def330            # --duration defaults to 0 = run until --stop
chloros-cli daq pool-record --sensor-id daq-e-def330 --stop
```

`--sensor-id` 값을 `chloros-cli daq pool-list`에서 가져옵니다. 알아두면 좋은 두 가지 기본값:

| 옵션 | 기본값 |
| --- | --- |
| `--duration` | `0` — `pool-record --stop`까지 기록 |
| `--output` / `-o` | `~/Documents/DAQ Live View/`는 **백엔드**의 파일 시스템에 저장되며, CLI의 파일 시스템이 아닙니다 |

CLI가 다른 머신의 백엔드를 대상으로 할 때 출력 디렉터리의 구분이 중요합니다. 파일은 백엔드가 실행되는 위치에 저장됩니다.

## Python에서 녹화

`DAQSensorSession`(`chloros_sdk.connect_daq_sensor()`에 의해 반환됨)는 동일한 풀링된 녹화 파일을 노출합니다: `record_start(output_dir=None, device_name=None)`는 파일 경로를 반환하고, `record_stop()`는 `{path, rows}`를 반환합니다. 전체 세션 API에 대해서는 [SDK 참조](../reference/sdk-reference.md)를 참조하십시오. SDK의 직접 하드웨어 클래스(데스크톱 설치용만 해당)는 기본적으로 `~/Documents/DAQ/`에 기록을 저장합니다. 릴리스된 빌드의 경우, 위에서 언급한 풀링된 경로가 지원되는 경로입니다.

## 처리 시 .daq 파일 사용

이미지에서 반사율을 산출하려면, Chloros에는 각 노출에 일치하는 하향 복사 조도가 필요합니다:

* **이미지와 함께 `.daq` 파일을 보관하십시오.** 처리 시 파이프라인은 기록된 `.daq`(모든 DAQ 모델) 또는 이미지와 함께 발견된 DAQ-M 네이티브 `.csv` — 파일에서 자동으로 추출하여 해결합니다. GUI 녹화 파일은 녹화가 종료되는 즉시 프로젝트에 추가되므로 이 조건이 자동으로 충족됩니다.
* **보정은 필요에 따라 가져옵니다.** 카메라별 또는 DAQ별 공장 보정 번들이 아직 로컬에 캐시되어 있지 않은 경우, Chloros는 첫 사용 시 MAPIR의 클라우드에서 이를 자동으로 가져옵니다 (한 번 인터넷 연결 필요; `~/.chloros/`에 캐시됨).
* **실시간 캡처는 자체 사이드카 파일을 생성합니다.** 실시간으로 캡처된 모든 반사율 프레임의 경우, 실제로 사용된 DAQ 측정값이 이미지 옆의 `.daq` 사이드카 파일로 저장되므로, 원본 기록 없이도 나중에 캡처 데이터를 재처리할 수 있습니다.

## 조도 데이터 다시 가져오기

프로젝트를 처리하면 해당 프로젝트에 포함된 모든 광센서 기록 데이터가
이미지 산출물 옆의 `Light Sensor/` 폴더로 내보내집니다. 이 과정에는 **이미지가 필요하지** 않습니다.
빛 센서만 단독으로 운용된 경우에도 완전한 캡처로 간주되며, `.daq`
파일만 포함된 폴더도 유효한 입력 자료입니다. 실행 보고서를 통해 작성된 빛 센서 산출물의 개수를 확인할 수 있습니다.

| 산출물 | 설명 |
| --- | --- |
| `<name>_calibrated.daq` | 실시간 기록과 동일한 스키마를 따르는 재처리 가능한 아카이브로, 이를 생성한 보정 번들을 명시합니다. 이를 다시 가져와도 **두 번째로** 보정되지는 않습니다. |
| `<name>_calibrated.csv` | 센서 고유의 파장 그리드에서 W/m²/nm 단위로 표시된 스펙트럼 조도. 각 측정값당 한 행으로 구성되며, 총 전력, 광시야 및 암시야 럭스, 청색/녹색/적색 분할이 포함된 PPFD, 피크 파장 등의 광도학적 열이 포함됩니다. |

교정 번들을 가져올 수 없는 DAQ-U 또는 DAQ-M(오프라인 상태이거나
해당 센서의 교정 데이터가 파일에 없는 경우)은 **사유와 함께 건너뛰어지며**, 원시 카운트를 포함하는
&quot;교정된&quot; 파일로 절대 저장되지 않습니다. 인터넷에 연결한 후 다시 실행하십시오. DAQ-E는
자체 교정 정보를 가지고 있으므로, 장치가 연결되어 있지 않고
로컬에 캐시된 데이터가 없을 때만 이 과정이 필요합니다.

### DAQ-A: 원시 카운트, 그리고 이것이 올바른 이유

**DAQ-A**는 직렬별 보정 번들 시스템이 도입되기 이전에 개발된 모델로, 가져올
번들이 없습니다. 이는 실수가 아닙니다. DAQ-A는 현장에서
반사율 타겟을 기준으로 보정되며, 타겟 기반 보정에는 센서의 *상대적*
응답만 필요하기 때문입니다. 이는 바로 원시 카운트 값 그 자체입니다. Chloros는 현재 이 값들을 사용하여 보정됩니다.

따라서 DAQ-A 기록 데이터는 내보내질 수 있지만, 다른 이름으로 내보내집니다:

```
<project>/
└── Light Sensor/
    ├── <name>_raw.daq
    └── <name>_raw.csv
```

`_raw`, `_calibrated`가 아닙니다. 파일 내부의 플래그가 아닌 별도의 파일 이름으로 지정되는 이유는,
이 정보가 파일 이름만 그대로 이메일로 전송될 때도 유지되어야 하기 때문입니다. `.csv`
헤더에는 `raw spectral sensor counts (NOT irradiance)`라고 표시되어 있으며, 이 값들은
파일 **내부**에서는 비교 가능하지만 센서 간에는 비교할 수 없다는 경고를 표시합니다. 실제 조도 — 총 전력, 럭스, PPFD — 에 대해서만
의미를 갖는 열들은 카운트 값을 바탕으로
계산되지 않고 비워 둡니다.

구형 DAQ-A-SD 기록(스키마 v1.01 / v1.02)은 파일의 쓰기 시간만 기록하고,
각 측정값별 타임스탬프는 기록하지 않습니다. Chloros는 이러한 기록과 이미지를 매칭하지 않습니다. — 프레임을
쓰기 시간과 연결하는 것은 겉보기에는 문제가 없어 보일지라도 잘못된 방법입니다 — 하지만 내보내기 기능은 이를 정상적으로 읽으며,
CSV는 어떤 클럭을 사용하고 있는지 명시합니다.

반사율에 대한 전체 내용(카메라를 사용한 단일 센서 및 주변광/피사체용 듀얼 센서)은 [반사율 워크플로우](reflectance.md)를 참조하십시오.
