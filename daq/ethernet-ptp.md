# DAQ-E 네트워킹 및 시간 동기화

> 센서의 물리적 네트워크 설정(케이블 연결, PoE, IP 할당 및 장치 자체의 네트워크 설정)에 대한 내용은 **[DAQ 사용자 설명서](https://mapir.gitbook.io/daq/daq-e/network-setup)**에 나와 있습니다. 이 페이지에서는 Chloros

측의 내용, 즉 연결, 시간 동기화, 그리고 디스커버리 결과가 비어 있을 때의 대처 방법을 다룹니다.

DAQ-E는 DAQ 제품군 중 이더넷 기반 모델로, PoE를 통해 전원을 공급받으며, mDNS(서비스 `_daq-e._tcp`)를 통해 검색되고, 센서 ID에서 파생된 호스트명 — `daq-e-<6 hex>.local`, 예: `daq-e-def330.local`. 이 페이지에서는 네트워크를 통해 데이터를 전송하는 방법과 PTP 시간 동기화에 참여하는 방법에 대해 다룹니다.

## 전송 모드

| 모드 | 엔드포인트 | 수신자 | 비고 |
| --- | --- | --- | --- |
| **멀티캐스트** (기본값) | UDP `239.10.10.10:5002` | 동일한 LAN 내의 모든 노드가 동일한 스트림을 수신 | 각 데이터그램은 CRC-16/CCITT 검증을 거침 |
| **Raw** | TCP 포트 `5000` | 정확히 하나의 클라이언트(독점적) | DAQ-U와 바이트 단위 호환 |

Chloros

는 기본적으로 멀티캐스트를 사용하며, 이를 통해 GUI, CLI

및 SDK

가 모두 한 번에 하나의 센서를 관찰할 수 있습니다.

## 네트워크 요구 사항

* **동일한 브로드캐스트 도메인.** Chloros

를 실행하는 컴퓨터는 센서와 동일한 L2 네트워크 세그먼트에 있어야 합니다. mDNS 탐색은 라우터를 통과하지 못합니다.
* **방화벽.** Windows

에서 설치 프로그램은 모든 네트워크 프로필에 백엔드를 위한 **Chloros

백엔드(UDP-In)**방화벽 규칙을 추가하며, 이 규칙은 DAQ-E 데이터(UDP 5002), mDNS(UDP 5353) 및 PTP(UDP 319/320)를 포함하며, 펌웨어 업데이트 중에 DAQ-E가 컴퓨터로 다시 연결할 때를 위해 로컬 서브넷으로 제한된**Chloros

백엔드(TCP-In)** 규칙도 추가합니다. Linux

에서 설치 시 해당 방화벽이 활성화되어 있으면, `.deb`가 `ufw` 또는 `firewalld`의 UDP 포트를 열어줍니다. 타사 방화벽이나 나중에 활성화된 방화벽의 경우에도 동일한 포트가 허용되어야 합니다.
* **PoE 전원, 상태 LED 없음.** DAQ-E에는 자체 LED가 없습니다. 스위치 또는 인젝터 포트의 링크/PoE 표시등을 통해 전원을 확인하고, 전원을 켠 후 기기가 부팅되어 네트워크에 연결될 때까지 몇 초간 기다리십시오.

## 연결

**GUI:** 광 센서 탭 → 센서 연결 → 장치 유형 &quot;DAQ-E (이더넷)&quot;. 연결 대화 상자가 화면에 표시되어 있는 동안에만 검색이 실행되며(Windows

에서는 mDNS 검색 및 ARP 스윕), 15초마다 반복됩니다. 새로 고침 버튼을 누르면 즉시 재검색이 수행됩니다. 탐지된 센서는 드롭다운 목록에 표시되며, 가장 먼저 감지된 센서가 자동으로 선택됩니다.

<!-- SCREENSHOT-NEEDED: DAQ connect dialog with Device Type set to "DAQ-E (Ethernet)" and at least one discovered sensor listed in the Hostname/IP dropdown (e.g. daq-e-xxxxxx.local), Connect button enabled. -->

**CLI** (백엔드 실행 중):

```bash
chloros-cli daq pool-connect --eth                              # auto-discover on the LAN
chloros-cli daq pool-connect --eth-host daq-e-def330.local      # explicit host — the reliable form
chloros-cli daq pool-connect --eth-host 192.168.1.57            # a plain IP works too
```

풀에 이미 등록된 DAQ-E는 호스트명과 IP 주소 모두를 통해 식별됩니다. 다른 주소로 다시 연결하면 장치를 두 번째로 열지 않고 기존 풀 항목을 반환합니다.

### 다중 NIC 호스트 및 부팅 후 첫 연결

활성 네트워크 인터페이스가 두 개 이상인 호스트의 경우, 부팅 후 **첫 번째** `pool-connect --eth` 연결 시 센서가 정상 상태임에도 결과가 비어 있을 수 있습니다. ARP 캐시가 아직 초기화된 상태에서는 센서가 연결된 인터페이스를 탐색 과정에서 놓칠 수 있기 때문입니다. 이 문제를 확실하게 해결하려면 탐색 단계를 건너뛰고 주소를 명시적으로 전달해야 합니다:

```bash
chloros-cli daq pool-connect --eth-host daq-e-def330.local
```

`--eth-host`는 mDNS 호스트 이름이나 IP를 모두 허용하며, 항상 올바른 센서를 대상으로 하므로 스크립트 및 헤드리스 설치 시 권장되는 형식입니다. GUI에서는 연결 대화 상자의 ‘새로 고침’ 버튼을 사용하여 재스캔 주기가 완료되도록 하십시오.

## 장치 설정 및 펌웨어

센서 자체는 네트워크 설정을 저장합니다: DHCP(공장 출하 시 기본 설정으로, DHCP 서버가 응답하지 않을 경우 169.254.x.x 링크 로컬 주소로 대체됨) 또는 고정 주소입니다. 또한 전원을 켤 때 스트리밍을 시작할지 여부도 저장합니다. GUI(센서의 톱니바퀴 모양 설정, **네트워크**:**변경…**,**전원 켜짐 시 스트리밍**,**센서 재시작**; [DAQ 탭](gui.md) 참조)에서 변경하거나, `pool-set-network`, `pool-set-auto-stream` 및 `pool-reboot` (`DAQSensorSession.set_network()`, `set_auto_stream()`, `reboot()`; [CLI

참조](../reference/cli-reference.md#daq-e-device-settings) 참조). 센서는 네트워크 변경 사항을 적용하기 위해 재시작되며, Chloros

가 이를 다시 연결합니다. 고정 IP 주소는 Chloros

를 실행 중인 컴퓨터의 네트워크에 속해야 합니다. Chloros

는 재시작 후 연결할 수 없는 고정 IP 주소를 거부합니다. 센서의 ID와 `daq-e-<id>.local` 호스트명은 절대 변경되지 않습니다. 1.3.0 버전에서 새로 추가된 기능.

**펌웨어 업데이트 기능이 GUI에 내장되었습니다.** 연결된 DAQ-E가 Chloros

빌드에 포함된 이미지보다 오래된 펌웨어를 실행 중인 경우, 해당 센서 행에 주황색 **업데이트 가능** 알림이 표시되며, 톱니바퀴 설정 모달에 &quot;~로 업데이트<version>&quot; 버튼</version>

이 나타납니다<version>. 업데이트는 네트워크를 통해 약 30초 만에 완료되며, 센서는 자동으로 재시작 및 재연결됩니다. 또한 전송이 중단되더라도 현재 펌웨어는 그대로 유지됩니다. CLI나 SDK에서, 그리고 GUI가 없는 Linux 환경에서는 `chloros-cli daq pool-update-firmware --sensor-id <id>`(`DAQSensorSession.update_firmware()`) 명령을 실행하십시오.

<!-- SCREENSHOT-NEEDED: DAQ-E per-sensor settings modal showing the DAQ-E-only rows: Hostname/IP, Firmware row with the "Update to <ver>" button (or "Up to date"), and the PTP Sync row with a live state value. -->

## PTP 시간 동기화

DAQ-E 펌웨어 v1.2.0 이상은 일반(슬레이브 전용) 클럭으로서 IEEE 1588 PTPv2에 참여합니다. **Chloros 호스트의 백엔드는 PTP 그랜드마스터입니다** — LAN에 연결된 모든 DAQ-E 및 LATTICE 카메라는 도메인 0에서 이 그랜드마스터를 슬레이브로 따르며, 모든 장치의 타임스탬프를 약 1ms의 허용 오차 범위 내에 유지합니다. 이 공유 클럭 덕분에 DAQ 측정값의 타임스탬프를 카메라 노출 시간과 일치시킬 수 있습니다([녹화 및 .daq 형식](recording.md) 참조).

CLI에서 동기화 상태를 확인하세요:

| 명령어 | 표시 내용 |
| --- | --- |
| `chloros-cli time-sync status` | 호스트 그랜드마스터 상태, BMCA 우선순위, 클럭 식별자 및 `health`: `OK` 또는 `SENDS_FAILING` / `DEGRADED` / `NO_SOCKETS` / `PEERS_SILENT` 및 `health_detail`가 자신의 패킷이 네트워크에 도달하지 않는 이유를 설명 |
| `chloros-cli time-sync peers` | 감지된 모든 슬레이브 (DAQ-E 센서 + LATTICE 카메라) |
| `chloros-cli time-sync cameras` | 카메라별 PTP 상태 (`PtpStatus`, `PtpOffsetFromMaster`, `PtpMeanPathDelay`) |
| `chloros-cli time-sync restart` | 그랜드마스터 프로세스 재시작 |

GUI의 DAQ-E 설정 모달에는 센서의 현재 PTP 상태를 보여주는 실시간 **PTP 동기화** 행이 표시됩니다.**PTP가 꺼진 DAQ-E는 계속 꺼진 상태로 유지됩니다.** 장치는 자체 메모리에 PTP 켜기/끄기 설정을 저장하므로, PTP가 꺼져 있던 장치는 재연결하거나 전원을 껐다 켰을 때도 꺼진 상태로 유지되며, 해당 장치의 타임스탬프는 PTP와 동기화되지 않습니다. `chloros-cli daq pool-list`는 이러한 장치를 표시(`!! PTP is switched OFF on this unit …`)하고 이를 해결하는 명령어를 출력합니다:

```bash
chloros-cli daq pool-enable-ptp --sensor-id daq-e-def330
```

Chloros는 장치가 의도적으로 꺼져 있을 수 있으므로, 절대로 장치를 자동으로 다시 켜지 않습니다.

엄격한 정렬을 요구하는 사용자를 위한 세부 사항:

* 스트리밍되는 모든 데이터그램에는 플래그 필드가 포함되어 있으며, **타임스탬프가 PTP로 동기화된 프레임에서는 2번 비트가 설정됩니다**. 카메라/DAQ의 엄격한 정렬이 필요한 파이프라인은 해당 비트를 기준으로 게이트를 설정해야 합니다.
* 동기화된 캡처를 시작하기 전에, 센서가 `chloros-cli time-sync peers`에 표시되는지, 그리고 `chloros-cli daq pool-list`에서 해당 센서의 PTP가 꺼진 상태로 표시되지 않는지 확인하십시오. (MAPIR의 내부 직접 하드웨어 툴링은 또한 `--wait-ptp` 플래그를 사용하여 PTP 잠금 시 녹화를 제어할 수 있으며, 이 플래그는 센서가 SLAVE 상태에 도달할 때까지 최대 15초 동안 대기합니다. 해당 툴링은 제공되는 CLI의 일부가 아닙니다.)
* PTP가 활성 슬레이브 상태인 동안 센서는 수동 클럭 전송을 거부합니다(&quot;PTP가 클럭을 제공하고 있음&quot;). 이는 의도된 동작이므로 PTP를 신뢰하십시오.

## Linux 참고 사항

* **루트 권한 없이 PTP 포트 사용.** PTP는 UDP 319/320 포트를 바인딩합니다. `.deb`는 `/etc/sysctl.d/60-chloros-ptp.conf`(`net.ipv4.ip_unprivileged_port_start = 319`)를 기록하므로, 백엔드는 사용자의 계정으로 실행되며 해당 포트에 바인딩됩니다. 1.3.0 버전부터 이 패키지는 백엔드에 파일 권한(`setcap`)을 설정하지 않으며 `libcap2-bin`도 필요하지 않습니다. 직접 권한을 추가하지 마십시오. 압축을 푼 백엔드가 시작되지 않을 수 있습니다. PTP가 포트 319에 바인딩할 수 없는 경우, [Linux 설치](../linux/linux-installation.md#ptp-will-not-start--cannot-bind-port-319)를 참조하십시오.

* **헤드리스 Jetson / Linux 호스트:** 이 패키지는 백엔드 서비스를 설치하지만, 어떤 서비스도 활성화하지 않습니다. 사용자가 명령을 실행하지 않아도 PTP(및 DAQ 가용성)가 항상 켜져 있도록 하려면, 현재 사용자 계정으로 서비스를 활성화하십시오:

  ```bash
  sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
  ```

  이 작업을 수행하지 않으면, PTP는 `chloros-cli` 명령어나 SDK 스크립트와 같이 백엔드를 시작한 작업이 있는 동안에만 실행됩니다. 루트 `chloros-backend.service` 및 루트에서 전환하는 방법에 대해서는 [Linux 설치](../linux/linux-installation.md#always-on-ptp-for-headless-hosts)를 참조하십시오.

## 문제 해결: &quot;DAQ-E 장치가 발견되지 않음&quot;

| 확인 항목 | 세부 정보 |
| --- | --- |
| 전원 | 센서의 LED가 켜지지 않음 — 스위치/인젝터 포트의 PoE 및 링크 표시등을 확인하십시오. 전원을 켠 후 몇 초간 기다리십시오 |
| 브로드캐스트 도메인 | 호스트와 센서가 동일한 L2 세그먼트에 있음; mDNS가 라우팅되지 않음 |
| 방화벽 | Windows: 설치 프로그램의 **Chloros Backend (UDP-In)** 규칙이 DAQ-E 스트림(UDP 5002), mDNS(5353) 및 PTP(319/320)의 유입을 허용합니다. 타사 방화벽도 동일한 설정이 필요합니다. Linux: `.deb`는 설치 시점에 방화벽이 활성화된 경우에만 `ufw` / `firewalld`에서 해당 포트를 엽니다 — 나중에 활성화할 경우 직접 허용해야 합니다 |
| 다중 NIC 호스트 | 디스커버리는 ARP를 대체 수단으로 사용하여 mDNS를 검색합니다. `--eth-host <ip-or-hostname>`는 디스커버리 과정을 건너뛰고 직접 연결합니다 |
| GUI 재검색 | 디스커버리는 연결 대화 상자가 열려 있는 동안에만 실행됩니다. ‘새로 고침’을 사용하십시오 |</version>
