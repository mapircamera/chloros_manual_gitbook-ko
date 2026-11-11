# CLI 빠른 시작 (pool-*)

제공되는 `chloros-cli`는 **`daq pool-*`** 명령어 계열을 통해 DAQ 센서를 제어합니다. 이 명령어 계열은 Chloros 백엔드의 상시 센서 풀을 통해 센서를 작동시키는 경량 HTTP 클라이언트입니다. 백엔드가 전송 경로를 관리하므로, GUI, CLI 및 SDK 스크립트는 모두 포트를 놓고 경쟁하지 않고 하나의 활성 핸들을 공유합니다. 사용자가 필요로 하는 모든 기능(연결, 스트리밍, 보정된 `.daq` 파일 기록, 캡 프로필 교체 등)은 `pool-*`를 통해 이용할 수 있습니다.

`pool-*`는 출시된 빌드에서 지원되는 DAQ 표면입니다. `chloros-cli daq --help`는 `pool-*` 하위 명령어를 나열하며, `daq discover` / `daq list`는 `pool-discover`와 동일한 스캔을 실행합니다. 직접 하드웨어 DAQ 하위 명령을 사용할 수 없는 경우, 누락된 패키지의 이름을 명시하고 `pool-*`로 되돌아가도록 안내하는 명확한 오류 메시지와 함께 종료됩니다. 오류가 발생해도 아무런 알림 없이 조용히 종료되는 일은 없습니다. (`pip install chloros-sdk` 역시 직접 하드웨어 명령어를 제공하지 않습니다.)

***

## 필수 조건

* **Chloros 백엔드가 실행 중이어야 합니다** — `pool-*` 명령어는 하드웨어 드라이버가 아닌 HTTP 클라이언트입니다. Windows에서는 Chloros 데스크톱 앱을 실행하십시오(이 앱이 백엔드를 시작합니다). 헤드리스 Linux/Jetson 환경에서는 현재 사용자로 서비스를 활성화하십시오: `sudo systemctl enable --now chloros-backend@$USER.service` ([Linux 설치](../linux/linux-installation.md#always-on-ptp-for-headless-hosts) 참조).
* **Chloros+ (유료 티어) 로그인**: 먼저 `chloros-cli login`를 실행하십시오. 권한 적용은 서버 측에서 이루어집니다. 로그인하지 않은 상태에서는 `401 AUTH_REQUIRED` 오류로 명령이 실패하며, 무료(Iron) 티어에서는 `403 PLAN_UPGRADE_REQUIRED` 오류로 실패합니다.
* 명령어는 기본적으로 `http://127.0.0.1:5000`를 대상으로 합니다. 백엔드가 다른 곳에서 실행되는 경우, `daq pool-*` 계열은 `CHLOROS_BACKEND_URL` 환경 변수를 따릅니다.

***

## 5분 세션

```bash
# 1. Connect a sensor into the backend pool (pick the line matching your model)
chloros-cli daq pool-connect                                  # smart-detect any DAQ
chloros-cli daq pool-connect --port COM3                      # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF          # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-def330.local    # DAQ-E by hostname (reliable)
#    (a DAQ-M's BLE MAC is not printed anywhere: find it with `chloros-cli daq pool-discover --only ble`)

# 2. List the pool — this shows the sensor_id used by every command below
chloros-cli daq pool-list

# 3. Read the most recent calibrated spectrum frame (add --json for scripting)
chloros-cli daq pool-latest --sensor-id daq-e-def330 --json

# 4. Record a calibrated .daq file for 60 seconds
chloros-cli daq pool-record --sensor-id daq-e-def330 --duration 60 \
  --device-name "field-A"

# 5. Release the sensor when done
chloros-cli daq pool-disconnect --sensor-id daq-e-def330
```

***

## `pool-connect` — 풀에서 센서 열기

| 변형 | 의미 |
| --- | --- |
| `daq pool-connect` | 스마트 감지: 이 컴퓨터에서 DAQ를 찾습니다. |
| `daq pool-connect --port PORT` | 특정 시리얼 포트에 연결된 DAQ-U (예: `COM3`, `/dev/ttyUSB0`). |
| `daq pool-connect --ble` | BLE를 통한 DAQ-M, MAC 자동 스캔. |
| `daq pool-connect --mac MAC` | 알려진 BLE MAC을 사용하는 DAQ-M (`--ble`를 의미함). |
| `daq pool-connect --eth-host HOST` | 알려진 호스트명 또는 IP 주소의 DAQ-E — **가장 안정적인 경로**. |
| `daq pool-connect --eth` | 자동 탐색(mDNS, ARP 대체 방식 포함)을 사용하는 DAQ-E. 아래 주의 사항을 참조하십시오. |

튜닝 플래그(모두 선택 사항):

| 플래그 | 의미 |
| --- | --- |
| `--integration-time MS` / `-t MS` | 수동 적분 시간(밀리초 단위). |
| `--frame-avg N` / `-f N` | 보고된 스펙트럼당 평균 프레임 수. |
| `--no-ae` | 자동 노출(AE) 비활성화 (기본적으로 AE는 켜져 있음). |
| `--no-stream` | 스트림을 시작하지 않고 연결(나중에 `pool-stream --start`로 재개). |
| `--cap-id CAP` | 캡 보정 프로필; 백엔드 기본값은 `sunshine_cosine`입니다. [`pool-set-cap`](#pool-set-cap-declare-the-fitted-cap)를 참조하십시오. |

이미 풀에 등록되어 있는 DAQ-E를 다른 주소(호스트명 또는 IP)로 연결하면, 장치를 다시 열지 않고 기존 풀 항목을 반환합니다.

{% hint style="warning" %}
**`--eth` 자동 검색 관련 주의 사항.** 다중 호스팅 호스트(활성 네트워크 인터페이스가 두 개 이상인 경우)에서, 부팅 후 *첫 번째* `pool-connect --eth` 호스트명 검색 결과는 센서가 정상 상태임에도 불구하고 빈 결과로 나올 수 있습니다. ARP 캐시가 초기 상태일 때 검색 과정에서 센서의 인터페이스를 놓칠 수 있기 때문입니다. `--eth`에서 아무것도 찾지 못하면 재시도하거나, `--eth-host <ip-or-hostname>`를 사용하여 탐색을 완전히 건너뛰십시오. 이는 다중 호스트 시스템에서 신뢰할 수 있는 방법입니다. DAQ-E의 호스트 이름은 `daq-e-<id>.local`(예: `daq-e-def330.local`)이며, 일반 IP 주소도 사용할 수 있습니다.
{% endhint %}

## `pool-discover` — 연결하지 않고 스캔하기

```bash
chloros-cli daq pool-discover                  # USB, BLE and Ethernet
chloros-cli daq pool-discover --only ble       # BLE only: this is how you find a DAQ-M's MAC
chloros-cli daq pool-discover --json           # machine-readable
```

연결 가능한 센서를 스캔하고, 실제로 연결하지 않은 상태에서 연결에 사용할 주소를 표시합니다. `--only`는 `usb,ble,eth`의 쉼표로 구분된 하위 집합을 입력으로 받습니다. `--timeout SEC`는 전송 방식별 스캔 시간을 설정합니다(기본값 5초, 1~20초 범위 내). 풀에서 이미 열려 있는 센서는 목록에 표시되지 않습니다(연결된 DAQ-M은 광고를 중단함). 해당 센서를 확인하려면 `pool-list`를 사용하십시오. 출시된 빌드에서는 `chloros-cli daq discover` 및 `daq list`가 동일한 스캔을 실행합니다.

## `pool-list` — 연결된 항목 확인

백엔드 풀에 있는 모든 센서를 표시하며, 여기에는 다른 모든 명령에서 필요한 `sensor_id`도 포함됩니다:

| 모델 | `sensor_id` 형식 | 예시 |
| --- | --- | --- |
| DAQ-U / DAQ-M | 5-옥텟 하이픈 구분 | `CB-7C-A8-2E-5F` |
| DAQ-E | `daq-e-<6 hex digits>` | `daq-e-def330` |

PTP가 비활성화된 DAQ-E(설정은 장치에 저장됨)는 `pool-list`에서 `!! PTP is switched OFF on this unit (saved in its flash): its timestamps are not PTP-synced.`로 표시되며, 이를 다시 활성화하는 명령어는 — 참조 [`pool-enable-ptp`](#pool-enable-ptp-switch-a-daq-es-ptp-back-on)를 참조하십시오.

## `pool-latest` — 스펙트럼 프레임 읽기

```bash
chloros-cli daq pool-latest --sensor-id daq-e-def330 --recent 10 --json
```

가장 최근의 프레임 또는 가장 최근의 `--recent N` 프레임을 반환하며, `--json`는 스크립팅에 사용할 수 있도록 기계가 읽을 수 있는 형식의 출력을 생성합니다. 프레임은 135포인트, 340–1010 nm 그리드 상의 방사계 보정된 스펙트럼 조도(W/m²/nm)이며, 센서의 캡 프로파일이 이미 적용된 상태입니다. 정량적인 조도 수치를 얻으려면 최소 15초 분량의 프레임을 평균화해야 합니다. 이는 기기의 특성이며 결함이 아닙니다.

## `pool-stream` — 스트리밍 일시 중지 또는 재개

```bash
chloros-cli daq pool-stream --sensor-id daq-e-def330 --stop    # pause
chloros-cli daq pool-stream --sensor-id daq-e-def330 --start   # resume
```

## `pool-record` — `.daq` 파일 기록

```bash
chloros-cli daq pool-record --sensor-id daq-e-def330 --duration 150 \
  --output ~/Documents/spectra --device-name "rooftop-A"
chloros-cli daq pool-record --sensor-id daq-e-def330 --stop
```

| 플래그 | 기본값 | 의미 |
| --- | --- | --- |
| `--duration SEC` / `-d SEC` | `0` | 녹화 시간(초); `0`는 `--stop`를 실행할 때까지 실행함을 의미합니다. |
| `--output DIR` / `-o DIR` | `~/Documents/DAQ Live View/` | 출력 디렉터리. 이 컴퓨터에 백엔드가 있는 경우, 상대 경로는 명령을 실행한 폴더를 기준으로 해석되며, 원격 백엔드의 경우 경로는 **백엔드가 실행 중인 컴퓨터**에서 해석됩니다. |
| `--device-name NAME` | — | 녹화 파일과 함께 저장되는 레이블. |
| `--stop` | — | 진행 중인 녹화를 중지합니다. |
| `--confirm-after SEC` | `20` | 시작 후, 첫 번째 행이 기록될 때까지 이 시간만큼 대기하고, 아무것도 도착하지 않으면 큰 소리로 경고(`!! WARNING: no rows written after …s. This recording is producing an EMPTY file.`)합니다. `0`는 이 검사를 비활성화합니다. |
| `--heartbeat SEC` | `30` | `--duration`를 사용하면 SEC초마다 행 수를 출력하고, 행 수가 더 이상 증가하지 않으면 경고를 표시합니다. `0`는 이 기능을 비활성화합니다. |

{% hint style="info" %}
기록은 백엔드에서 이루어지므로, `.daq` 파일은 **백엔드 머신의** 파일 시스템에 저장됩니다. 기본적으로 `~/Documents/DAQ Live View/` 해당 위치에 저장되며, 반드시 CLI를 실행한 위치에 저장되는 것은 아닙니다. 파일 이름에는 센서 ID와 타임스탬프가 포함됩니다.
{% endhint %}

## `pool-set-cap` — 장착된 캡 선언

```bash
chloros-cli daq pool-set-cap --sensor-id daq-e-def330 --cap-id sunshine_cosine
```

캡 ID는 모든 스펙트럼에 적용되는 공장 측정 보정 프로파일을 선택하며, 이는 **센서에 물리적으로 장착된 캡과 반드시 일치해야 합니다** — 센서나 소프트웨어 어느 쪽도 캡을 자체적으로 감지할 수 없으며, 선택된 캡 정보는 모든 `.daq` 파일에 기록됩니다. 모든 환경의 기본값은 `sunshine_cosine`입니다(모든 DAQ에는 Sunshine 코사인 보정 캡이 설치된 상태로 출하되며, 설계상 약 12배의 감쇠를 가집니다 — 캡 변경을 선언하지 않으면 스펙트럼이 대략 그 배수만큼 잘못 보정됩니다).

| `--cap-id` | 사용 가능 모델 |
| --- | --- |
| `sunshine_cosine` (기본값) | DAQ-U, DAQ-M, DAQ-E |
| `fov_15`, `fov_45`, `fov_90` | DAQ-U, DAQ-E |
| `fov_30`, `fov_60` | DAQ-U 전용 |
| `none` | DAQ-E 전용 — 참고 사항 참조 |

센서 설정 범위를 벗어난 캡 ID는 연결 시 명확한 오류와 함께 거부됩니다. `none` (DAQ-E)는 캡이 물리적으로 제거되었음을 의미합니다. 이 경우 DAQ-E의 오목한 유리 디퓨저에 대해 공장 출하 시 설정된 기하학적 프로파일이 여전히 적용되므로, 단순히 아무 작업도 수행하지 않는(no-op) 상태는 아닙니다. 또한 캡이 없는 DAQ-E는 벤치 구성이며, 지원되는 현장 구성은 아닙니다. (캡이 없는 DAQ-U는 완전한 베어 상태이므로 보정 프로파일이 전혀 필요하지 않으며, DAQ-M은 Sunshine 캡과 함께 사용됩니다.)

## `pool-disconnect` — 센서 해제

```bash
chloros-cli daq pool-disconnect --sensor-id daq-e-def330   # one sensor
chloros-cli daq pool-disconnect --all                      # everything in the pool
```

`pool-disconnect`는 누가 열었든 상관없이 센서를 해제하므로, 스크립트 오류로 인해 풀에 남아 있는 센서도 해제합니다(단, 해당 센서의 기록 작업이 먼저 완료되어야 합니다). 풀에 없는 센서는 해제할 대상이 없으므로, 이 명령은 `<sensor-id>: already released (not in the pool)`를 출력하고 종료됩니다(`0`). 백엔드가 거부했거나 확인하지 않은 해제 요청은 `1`로 종료됩니다.

## `pool-enable-ptp` — DAQ-E의 PTP 다시 켜기

```bash
chloros-cli daq pool-enable-ptp --sensor-id daq-e-def330
```

DAQ-E는 PTP 켜기/끄기 설정을 자체 메모리에 저장하므로, PTP가 꺼진 장치는 다시 켜질 때까지 꺼진 상태로 유지되며, 해당 장치의 타임스탬프는 PTP와 동기화되지 않습니다. `pool-list`는 이러한 장치를 표시하고 해당 장치에 대해 이 명령을 출력합니다. Chloros는 PTP가 꺼진 상태가 의도적인 것일 수 있으므로, 장치를 스스로 다시 켜지 않습니다. 1.3.0 버전부터 추가됨.

## DAQ-E 장치 설정

```bash
chloros-cli daq pool-info --sensor-id daq-e-def330
chloros-cli daq pool-set-network --sensor-id daq-e-def330 \
  --static 192.168.1.50 --gateway 192.168.1.1
chloros-cli daq pool-set-network --sensor-id daq-e-def330 --auto
chloros-cli daq pool-set-auto-stream --sensor-id daq-e-def330 --on
chloros-cli daq pool-reboot --sensor-id daq-e-def330
chloros-cli daq pool-update-firmware --sensor-id daq-e-def330
```

`pool-info`는 센서의 펌웨어(및 해당 Chloros 릴리스에 더 새로운 버전이 포함되어 있는지 여부), 주소 및 주소 할당 방식, 현재 실행 중인 설정과 다른 저장된 네트워크 설정, 전원 켜기 시 스트리밍 및 PTP 정보를 표시합니다. `pool-set-network`는 변경 사항을 적용하기 위해 센서를 재시작하고 기존 설정으로 다시 연결하며, `--no-reboot`는 설정만 저장합니다. 정적 주소는 이 컴퓨터의 네트워크에 속해 있어야 하며, 그렇지 않으면 Chloros가 이를 거부합니다. `--force`는 주소가 재할당되거나 해당 주소로 라우팅될 컴퓨터에 한해 이 설정을 무시합니다. 센서가 기록 중인 동안에는 재시작이 허용되지 않습니다. `pool-update-firmware`는 <version\>명령줄에서</version\> GUI의 **Update to \<version\>**에 </version\>해당하는 명령이며<version\>, GUI가 없는 Linux에서 DAQ-E를 업데이트하는 방법입니다. 1.3.0 버전에서 새로 추가되었습니다.***

## 명령어 요약

| 명령어 | 용도 |
| --- | --- |
| `daq pool-connect [--port P \| --ble \| --mac M \| --eth \| --eth-host H] [-t MS] [-f N] [--no-ae] [--no-stream] [--cap-id CAP]` | 백엔드 풀에서 센서를 엽니다. |
| `daq pool-discover [--only usb,ble,eth] [--timeout SEC] [--json]` | 연결하지 않고 센서를 스캔합니다(DAQ-M의 BLE MAC을 찾습니다). |
| `daq pool-list` | 풀에 있는 모든 센서를 해당 `sensor_id`와 함께 표시합니다. |
| `daq pool-latest --sensor-id ID [--recent N] [--json]` | 가장 최근의 N개 보정된 스펙트럼 프레임. |
| `daq pool-stream --sensor-id ID [--start \| --stop]` | 스트리밍 재개/일시 중지. |
| `daq pool-record --sensor-id ID [-d SEC] [-o DIR] [--device-name NAME] [--confirm-after SEC] [--heartbeat SEC] [--stop]` | `.daq` 녹화 시작/중지 (백엔드 측). |
| `daq pool-set-cap --sensor-id ID --cap-id CAP` | 실행 중에 캡 보정 프로필을 교체합니다. |
| `daq pool-disconnect --sensor-id ID [--all]` | 센서 하나를 해제하거나, 모두 해제합니다. |
| `daq pool-enable-ptp --sensor-id ID` | PTP가 꺼져 있는 DAQ-E의 PTP를 다시 켭니다. |
| `daq pool-info --sensor-id ID [--json]` | DAQ-E의 펌웨어, 주소, 저장된 네트워크 설정, 전원 켜기 시 스트리밍 및 PTP. |
| `daq pool-set-network --sensor-id ID (--auto \| --static IP [--mask M] [--gateway G]) [--no-reboot] [--force]` | DHCP 또는 고정 주소; 센서를 재시작하고 다시 연결합니다. |
| `daq pool-set-auto-stream --sensor-id ID (--on \| --off)` | 전원을 켤 때 스트리밍합니다. |
| `daq pool-reboot --sensor-id ID` | DAQ-E를 재시작하고 다시 연결합니다. |
| `daq pool-update-firmware --sensor-id ID` | 이전 버전의 센서에 이번 릴리스의 DAQ-E 펌웨어를 플래시합니다. |

***

## DAQ-E 최초 연결 문제 해결

1. DAQ-E에는 상태 LED가 없습니다. 스위치 또는 인젝터 포트의 PoE/링크 표시등을 통해 전원이 공급되는지 확인하고, 전원을 켠 후 부팅 및 네트워크 연결이 완료될 때까지 몇 초간 기다리십시오.
2. 백엔드 컴퓨터는 센서와 **동일한 브로드캐스트 도메인**에 있어야 합니다. mDNS는 라우터를 통과하지 못합니다.
3. 방화벽은 mDNS(UDP 5353), DAQ-E 데이터(UDP 5002) 및 PTP(UDP 319/320)를 허용해야 합니다. Windows 설치 프로그램은 이를 위한 규칙을 추가하며, Linux의 `.deb`는 설치 시점에 활성화되어 있던 `ufw` / `firewalld`에서 해당 포트를 열어줍니다. 타사 방화벽이나 나중에 활성화된 방화벽에서는 해당 포트를 허용해야 합니다.
4. 여전히 `--eth`에서 응답이 없나요? 장치의 호스트명(`daq-e-<id>.local`)이나 IP 주소를 사용하여 `--eth-host`를 실행하면, 검색 과정을 건너뛸 수 있습니다.

***
{% hint style="info" %}
**AI 어시스턴트를 위한 팁.** 이 설명서의 모든 페이지는 원시 마크다운 형식으로 제공됩니다. 페이지의 소문자 URL 슬러그 끝에 `.md`를 추가하세요(이 페이지의 경우: `https://mapir.gitbook.io/chloros/daq/cli-quick-start.md`). 기계 판독 가능한 인덱스는 `https://mapir.gitbook.io/chloros/llms.txt`입니다. `chloros-cli daq` 및 기타 모든 명령어 계열에 대한 전체 플래그 수준 문서를 보려면 [CLI 참조](../reference/cli-reference.md) (`https://mapir.gitbook.io/chloros/reference/cli-reference.md`)를 확인하십시오; [SDK 참조](../reference/sdk-reference.md)에서 Python 경로는 `chloros_sdk.connect_daq_sensor()`입니다.
{% endhint %}
</version\>
