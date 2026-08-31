# DAQ-E 네트워킹 및 시간 동기화

> 센서의 물리적 네트워크 설정(케이블 연결, PoE, IP 할당 및 장치 자체의 네트워크 설정)에 대한 내용은 **[DAQ 사용자 설명서](https://mapir.gitbook.io/daq/daq-e/network-setup)**에 나와 있습니다. 이 페이지에서는 Chloros 측의 내용, 즉 연결, 시간 동기화, 그리고 디스커버리 결과가 비어 있을 때의 대처 방법을 다룹니다.

DAQ-E는 DAQ 제품군 중 이더넷 기반 모델로, PoE를 통해 전원을 공급받고, mDNS(서비스 `_daq-e._tcp`)를 통해 검색되며, 센서 ID — `daq-e-<6 hex>.local`, 예: `daq-e-def330.local`. 이 페이지에서는 네트워크를 통해 데이터를 전송하는 방식과 PTP 시간 동기화에 참여하는 방법에 대해 다룹니다.

## 전송 모드

| 모드 | 엔드포인트 | 수신자 | 비고 |
| --- | --- | --- | --- |
| **멀티캐스트** (기본값) | UDP `239.10.10.10:5002` | 동일한 LAN 내의 모든 노드가 동일한 스트림을 수신 | 각 데이터그램은 CRC-16/CCITT 검증을 거침 |
| **원시(Raw)** | TCP 포트 `5000` | 정확히 하나의 클라이언트(독점적) | DAQ-U와 바이트 단위 호환 가능 |

Chloros는 기본적으로 멀티캐스트를 사용하며, 이를 통해 GUI, CLI 및 SDK가 모두 한 개의 센서를 동시에 모니터링할 수 있습니다.

## 네트워크 요구 사항

* **동일한 브로드캐스트 도메인.** Chloros를 실행하는 머신은 센서와 동일한 L2 네트워크 세그먼트에 있어야 합니다. mDNS 탐색은 라우터를 통과하지 못합니다.
* **Windows 방화벽 확인 메시지: 허용하십시오.** Chloros가 멀티캐스트 소켓을 처음 바인딩할 때, Windows Defender가 한 번 확인을 요청합니다. 이를 허용하면 DAQ-E 데이터(UDP 5002), mDNS(UDP 5353) 및 PTP(UDP 319/320)가 포함됩니다. Linux에서는 이 과정이 자동으로 처리됩니다.
* **PoE 전원, 상태 LED 없음.** DAQ-E에는 자체 LED가 없습니다. 스위치 또는 인젝터 포트의 링크/PoE 표시등을 통해 전원을 확인하고, 전원을 켠 후 기기가 부팅되어 네트워크에 연결될 때까지 몇 초간 기다리십시오.

## 연결

**GUI:** 광 센서 탭 → 센서 연결 → 장치 유형 &quot;DAQ-E (이더넷)&quot;. 탐색 기능은 연결 대화 상자가 화면에 표시되어 있는 동안에만 실행되며(mDNS 검색 및 Windows에 대한 ARP 스윕), 15초마다 반복됩니다. &#x27;새로 고침&#x27; 버튼을 누르면 즉시 재탐색이 수행됩니다. 탐지된 센서는 드롭다운 목록에 표시되며, 가장 먼저 감지된 센서가 자동으로 선택됩니다.

<!-- SCREENSHOT-NEEDED: DAQ connect dialog with Device Type set to "DAQ-E (Ethernet)" and at least one discovered sensor listed in the Hostname/IP dropdown (e.g. daq-e-xxxxxx.local), Connect button enabled. -->

**CLI** (백엔드 실행 중):

```bash
chloros-cli daq pool-connect --eth                              # auto-discover on the LAN
chloros-cli daq pool-connect --eth-host daq-e-def330.local      # explicit host — the reliable form
chloros-cli daq pool-connect --eth-host 192.168.1.57            # a plain IP works too
```

### 다중 NIC 호스트 및 부팅 후 첫 연결

활성 네트워크 인터페이스가 두 개 이상인 호스트의 경우, 부팅 후 **첫 번째** `pool-connect --eth`는 센서가 정상 상태임에도 불구하고 빈 결과로 나타날 수 있습니다. ARP 캐시가 아직 초기화된 상태일 때, 탐색 과정에서 센서가 위치한 인터페이스를 놓칠 수 있기 때문입니다. 이 문제를 확실하게 해결하려면 탐색 단계를 건너뛰고 주소를 명시적으로 전달해야 합니다:

```bash
chloros-cli daq pool-connect --eth-host daq-e-def330.local
```

`--eth-host`는 mDNS 호스트 이름이나 IP를 모두 허용하며, 항상 올바른 센서를 대상으로 하므로 스크립트 및 헤드리스 설치 시 권장되는 형식입니다. GUI에서는 연결 대화 상자의 ‘새로 고침’ 버튼을 사용하여 재스캔 주기가 진행되도록 하십시오.

## 장치 설정 및 펌웨어

센서 자체에는 네트워크 설정(고정 IP 대 DHCP + 링크 로컬 주소 지정), 장치 이름, 부팅 시 자동 스트리밍, OTA 비밀번호 등이 저장되어 있습니다. 이러한 장치 측 설정은 제공된 CLI에서는 명령어로 노출되지 않으며, 해당 설정이 표시되는 Chloros GUI를 통해 관리되거나 MAPIR 지원을 통해 관리됩니다.

**펌웨어 업데이트 기능은 GUI에 내장되어 있습니다.**연결된 DAQ-E가 Chloros 빌드에 포함된 이미지보다 오래된 펌웨어를 실행 중인 경우, 해당 센서 행에 황색**업데이트 가능** 알림이 표시되며, 톱니바퀴 설정 모달에 &quot;~로 업데이트<version>

&quot; 버튼</version>이 제공됩니다<version>

. 업데이트는 네트워크를 통해 약 30초 만에 완료되며, 센서는 자동으로 재시작 및 재연결됩니다. 전송이 중단되더라도 현재 펌웨어는 그대로 유지됩니다.

<!-- SCREENSHOT-NEEDED: DAQ-E per-sensor settings modal showing the DAQ-E-only rows: Hostname/IP, Firmware row with the "Update to <ver>" button (or "Up to date"), and the PTP Sync row with a live state value. -->

## PTP 시간 동기화

DAQ-E 펌웨어 v1.2.0 이상은 일반(슬레이브 전용) 클럭으로서 IEEE 1588 PTPv2에 참여합니다. **Chloros 호스트의 백엔드는 PTP 그랜드마스터입니다** — LAN에 연결된 모든 DAQ-E 및 LATTICE 카메라는 도메인 0에서 이 그랜드마스터에 슬레이브로 동기화되며, 모든 장치의 타임스탬프를 약 1ms의 오차 범위 내로 유지합니다. 이 공유 클럭 덕분에 DAQ 측정값의 타임스탬프를 카메라 노출 시간과 일치시킬 수 있습니다([녹화 및 .daq 형식](recording.md) 참조).

CLI에서 동기화 상태를 확인해 보십시오:

| 명령 | 표시 내용 |
| --- | --- |
| `chloros-cli time-sync status` | 호스트 그랜드마스터 상태, BMCA 우선순위, 클럭 식별자 |
| `chloros-cli time-sync peers` | 감지된 모든 슬레이브 (DAQ-E 센서 + LATTICE 카메라) |
| `chloros-cli time-sync cameras` | 카메라별 PTP 상태 (`PtpStatus`, `PtpOffsetFromMaster`, `PtpMeanPathDelay`) |
| `chloros-cli time-sync restart` | 그랜드마스터 프로세스 재시작 |

GUI의 DAQ-E 설정 모달에는 센서의 현재 PTP 상태를 보여주는 실시간 **PTP 동기화** 행이 표시됩니다.

엄격한 정렬이 필요한 사용자들을 위한 세부 정보:

* 스트리밍되는 모든 데이터그램에는 플래그 필드가 포함되어 있으며, **타임스탬프가 PTP와 동기화된 프레임에서는 2번째 비트가 설정됩니다**. 카메라와 DAQ 간의 엄격한 정렬이 필요한 파이프라인은 해당 비트를 기준으로 게이트를 설정해야 합니다.
* 동기화된 캡처를 시작하기 전에, `chloros-cli time-sync peers`에 센서가 표시되는지 확인하십시오. (MAPIR의 내부 직접 하드웨어 툴링은 또한 `--wait-ptp` 플래그를 사용하여 PTP 락 시점에 녹화를 제어할 수 있으며, 이 플래그는 센서가 SLAVE 상태에 도달할 때까지 최대 15초 동안 대기합니다. 해당 툴링은 출하된 CLI에는 포함되어 있지 않습니다.)
* PTP가 활성 슬레이브 상태일 때, 센서는 수동 클럭 푸시를 거부합니다(&quot;PTP가 클럭을 제공하고 있음&quot;). 이는 설계상 의도된 동작이므로 PTP를 신뢰하십시오.

## Linux 참고 사항

* **PTP는 설치 시 `libcap2-bin`가 필요합니다.** `.deb` postinst는 `/usr/lib/chloros/chloros-backend`에서 `cap_net_bind_service=+ep` 권한을 부여하여, 루트 권한 없이도 PTP 포트 319/320에 바인딩할 수 있게 합니다. `libcap2-bin`가 없는 경우, 해당 단계가 건너뛰어지며 PTP가 시작되지 않습니다. 해결 방법:

  ```bash
  sudo apt install libcap2-bin
  sudo apt reinstall chloros
  ```

* **헤드리스 Jetson / Raspberry Pi:** 첫 설치 시 systemd 유닛 `chloros-backend.service`가 생성되지만 활성화되지는 않습니다. GUI 없이 PTP(및 DAQ 가용성)를 상시 가동하려면:

  ```bash
  sudo systemctl enable --now chloros-backend.service
  ```

  이 유닛이 없으면 PTP는 Chloros GUI가 열려 있는 동안에만 실행됩니다.

## 문제 해결: &quot;DAQ-E 장치가 발견되지 않음&quot;

| 확인 항목 | 상세 설명 |
| --- | --- |
| 전원 | 센서의 LED가 켜지지 않음 — 스위치/인젝터 포트의 PoE 및 링크 표시등을 확인하고, 전원을 켠 후 몇 초간 기다리십시오 |
| 브로드캐스트 도메인 | 호스트와 센서가 동일한 L2 세그먼트에 있음; mDNS가 라우팅되지 않음 |
| Windows 방화벽 | 첫 실행 시 Defender 프롬프트에서 허용 선택 (UDP 5002, 5353, 319/320) |
| 다중 NIC 호스트 | 부팅 후 첫 번째 검색 시 센서가 탐지되지 않을 수 있음 — `--eth-host <ip-or-hostname>`를 사용하여 연결 |
| GUI 재검색 | 검색은 연결 대화 상자가 열려 있는 동안에만 실행됨; &#x27;새로 고침&#x27;을 사용 |</version>
