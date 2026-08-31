# 멀티 카메라 어레이

LATTICE **어레이**는 두 대 이상의 LATTICE 카메라를 하나의 동기화된 단위로 연결한 것입니다. 카메라 중 하나가**마스터**역할을 하며, 공유 동기화 라인(기본값**Line2**)을 통해 하드웨어 GPIO 트리거 펄스를 전송하므로 모든 구성원이 동일한 순간을 포착합니다. Chloros는 PTP 시간 동기화, 라이브 미리보기(카메라별 타일 또는 정렬된 단일 멀티밴드 합성 이미지), 동기화된 캡처 기능을 추가합니다. 각 캡처 패스마다 모든 카메라가 동일한 타임스탬프와 프레임 ID를 공유하는 하나의**프레임 그룹**이 생성되며 (캡처 출력에서 `fid:N`로 보고됨)를 생성합니다.

어레이는 모노(M3M) 카메라가 식생 지수를 생성하는 방식입니다. 하나의 카메라가 하나의 밴드를 제공하며, 어레이는 이를 정렬하여 다중 밴드 스택으로 만듭니다. [모노 카메라 및 식생 지수](mono-indices.md)를 참조하십시오를 참조하십시오.

어레이를 연결하는 데는 세 가지 동등한 방법이 있으며, 이들 모두 동일한 &quot;스마트 준비(smart-prep)&quot; 흐름을 실행합니다:

| 표면 | 진입점 |
| --- | --- |
| GUI | 카메라 탭 → **어레이 연결** (파란색 버튼) |
| CLI | `chloros-cli lattice array-connect --serials SN1,SN2,…` (첫 번째 일련번호 = 마스터) |
| Python SDK | `connect_array(serials=[…])` → `ArraySession` (첫 번째 일련번호 = 마스터) |

Smart-prep는 다음 순서대로 수행됩니다: 네트워크 기능 탐색(ICMP DF 핑 + GVSP 탐색), 동기화 계층 선택, 전송 매체에 맞춘 프레임 크기의 자동 축소, PTP 활성화, 카메라별 픽셀 형식 자동 선택, 각 카메라의 저장된 상태를 기반으로 한 자동 노출 초기화, Line2의 GPIO 트리거 구성을 순차적으로 수행합니다.

{% hint style="info" %}
이 모든 기능이 작동하려면 먼저 링크를 통해 카메라에 접속할 수 있어야 합니다. 검색, 주소 지정 및 첫 연결 시 보정 데이터 다운로드에 대해서는 [카메라 연결](connecting.md)을 참조하십시오. 다중 카메라 시스템의 경우, 호스트 NIC의 수신 링 설정은 링크 속도만큼 중요합니다. 전체 증상→해결 방법 표는 [CLI 참조 § 호스트 NIC 설정 및 튜닝](../reference/cli-reference.md#host-nic-setup--tuning-lattice-arrays)에 있습니다.
{% endhint %}

## 어레이 연결 대화 상자

카메라 탭 → **어레이 연결**을 선택하면**선택 → 디스플레이 모드 → 설정**의 3단계 마법사가 열립니다.

### 1단계 — 마스터 및 슬레이브

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Select scene, with 3-4 LATTICE cameras discovered. Table showing Camera / Serial / IP / Master radio / Slave checkbox columns, with the green "GPIO master detected — selections pre-populated" probe banner visible above the table. -->

선택 대화 상자가 열리자마자 네트워크를 스캔한 후(&quot;네트워크 스캔 중...&quot;), GPIO 트리거 배선을 탐색합니다(&quot;GPIO 배선 탐색 중...&quot;). 어레이를 구성하려면 최소 **2대의 카메라**가 필요합니다.

배선 탐색은 가능한 경우 역할 선택 항목을 미리 채워주고, 다음 세 가지 배너 중 하나를 표시합니다:

| 배너 | 의미 |
| --- | --- |
| &quot;GPIO 마스터 감지됨 — 선택 항목 미리 채워짐&quot; (녹색) | 탐색 과정에서 트리거 토폴로지가 확인되었으며, 마스터 무선 및 슬레이브 확인란이 이미 선택된 상태입니다. |
| &quot;마스터 감지되지 않음 — GPIO 케이블 확인&quot; (주황색) | 트리거 펄스를 감지한 카메라가 없습니다. 동기화 케이블 연결을 확인하십시오. 여전히 수동으로 역할을 선택할 수 있습니다. |
| &quot;동기화 케이블 없음: {serials}&quot; (주황색) | 나열된 카메라에 동기화 케이블이 연결되어 있지 않습니다. |

카메라 표에는 **카메라 / 시리얼 / IP / 마스터(무선) / 슬레이브(체크박스)** 열이 있습니다:

* 반드시 **마스터 1대**와**슬레이브 1대 이상**을 선택하십시오. 현재 마스터의 무선 항목을 다시 클릭하면 선택이 해제됩니다.
* **&quot;동기화 케이블 없음&quot;**으로 표시된 카메라는 절대로 슬레이브로 선택할 수 없습니다. 트리거 배선이 없는 슬레이브는 동기화 라인을 끝없이 기다리다가 무효한 영상을 전송하게 됩니다. 해당 카메라는 대신 독립형 카메라로 연결하십시오.
* 이미 독립형으로 연결된 카메라는 비활성화되지 않습니다. 어레이 연결 시 독립형 세션이 해제되고, 해당 카메라는 어레이 내에서 다시 열립니다.

**다음: 디스플레이 모드 →**는 마스터와 슬레이브가 하나 이상 선택되면 활성화됩니다.**재스캔**은 탐색 및 배선 프로브를 다시 실행합니다.

{% hint style="warning" %}
**취소**는 스캔이나 프로브가 진행 중인 동안 비활성화됩니다. — LATTICE 카메라 펌웨어에서 프로브 도중 취소하면 카메라가 충돌할 수 있습니다(SDK). 로딩 아이콘이 완료될 때까지 기다리십시오.
{% endhint %}

### 2단계 — 디스플레이 모드

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Display Mode scene, showing the two selectable cards ("Separate Cameras" and "Combined Cameras") with Combined selected/highlighted as the default. -->

| 모드 | 제공 기능 |
| --- | --- |
| **별도 카메라** | 카메라당 하나의 라이브 타일이 표시되며, 모든 카메라가 동시에 트리거되어 프레임이 동기화됩니다. 각 카메라는 고유한 색상과 설정을 유지합니다. |
| **통합 카메라** *(기본값)* | 정렬된 다중 대역 NDVI/index 합성 이미지를 렌더링하는 단일 타일. 카메라는 어레이 색상을 공유합니다. |

표시 모드는 라이브 미리보기의 표시 방식만 변경하며, 캡처 동작은 두 모드 모두 동일합니다.

### 3단계 — 어레이 설정 및 예상 결과

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Settings scene, healthy state: left column with ROI / Binning / Pin resolution / Trigger Rate controls, right "Projected Outcome" column showing green "Simultaneous capture" tier, an fps range, the NIC line, the "Sim-emit burst" line, and the "Wire budget" line with a checkmark. -->

이 장면으로 진입하면 Chloros는 백엔드에 **권장 사항**을 요청하고 NIC의 수신 링에 맞는 ROI + 비닝 조합을 자동으로 적용합니다(비닝은 전체 시야를 보존하므로, ROI 자르기보다 비닝을 우선적으로 사용합니다). 설정을 변경할 때마다 분석이 실시간으로 재실행되며 오른쪽의**예상 결과** 패널이 업데이트됩니다.

왼쪽 열 — 설정:

| 제어 | 선택 사항 | 기본값 | 참고 |
| --- | --- | --- | --- |
| **ROI (시야)** | 전체 (2048×1536) / 반 (1024×768) / 1/4 (512×384) | 전체 | 센서 크롭: 네이티브 픽셀 피치로 더 작은 영역으로 반/1/4 크롭합니다. |
| **비닝** | 1× / 2× (2×2 합산) / 4× (4×4 합산) | 1× | 하드웨어 비닝: 2×2 = 와이어 비용의 1/4로 전체 시야 확보; 4×4 = 1/16 비용으로 전체 시야 확보. 카메라가 비닝을 지원하지 않으면 숨김. |
| **와이어 측 이미지** (읽기) | — | — | 비닝 후 실제로 와이어를 통해 전송되는 너비×높이이며, 16의 배수(최소 64)로 반올림됨. |
| **핀 해상도**| 선택 상자 | 꺼짐 | Chloros는 일반적으로 예상 전송 속도가**

1.5 fps** 미만으로 떨어지면 연결 시 자동으로 비닝을 활성화합니다. 핀 고정(Pinning)을 사용하면 선택한 프레임 크기를 유지한 채 더 낮은 전송 속도를 수용하며, 과부하가 걸린 구성을 자동 속도 하향 조정 대신 연결 거부로 전환합니다. |
| **트리거 속도** | 0.5–60 fps, 단계 0.1 | 비어 있음 = 자동 | 마스터의 트리거 발사 속도입니다. 비워 두면 Chloros가 이를 산출합니다. |
| **와이어 예산**| 20–2000 MB/s, 단위 10 | 비워두면 자동 | 호스트가 실제로 처리할 수 있는 양(MB/s 단위) —**전체 어레이 할당이 이 수치에 따라 결정됩니다.** 네트워크 어댑터에서 자동 감지됩니다. 어레이에서 손상된 프레임을 보고하는 경우 이 값을 낮추십시오. 감지된 값은 USB 어댑터 및 공유 스위치의 실제 처리량을 과대평가하는 경향이 있습니다. 이 값을 변경하면 예측 시뮬레이션이 실시간으로 다시 실행됩니다. |

오른쪽 열 — **예측 결과**:

* **동기화 계층** — &quot;동시 캡처&quot;(녹색), &quot;동시 캡처 (FTD-시차 송출)&quot;(녹색), &quot;시차 캡처 (100 ms 드리프트)&quot;(황색), 또는 &quot;구성이 너무 큼&quot;(빨간색).
* **fps 예측값** — 동기화된 어레이의 속도는 가장 느린 카메라의 노출 시간에 의해 제한되므로 범위(“어두움 → 밝음”)로 표시됩니다.
* **NIC 라인** — 링크 속도 및 지속 처리량(“NIC {mbps} Mbps · 지속 {N} MB/s”).
* **동시 방출 버스트 확인** — 호스트의 NIC 링이 모든 카메라에서 동시에 발생하는 하나의 버스트를 수신할 수 있는지 여부(&quot;동시 방출 버스트: X MB · NIC 링 사용 가능: Y MB ✓/✗&quot;).
* **전선 예산 확인** — 정상 상태의 총 수요 대 충돌 방지 전선 상한선 (“전선 예산: {n} 대 카메라가 요구하는 {demand} MB/s · 상한선 {ceiling} MB/s ✓/✗ 초과 할당”).
* **&quot;이 회선에서 지원 가능한 최대 카메라 수: {n} — 카메라별 대역폭 하한값에 의해 설정되므로, 비닝을 해도 이 수치가 증가하지 않습니다.&quot;** — 카메라 수 상한선에 근접했거나 초과했을 때 표시됩니다.
* **&quot;이 설정에서는 프레임이 손실됩니다.&quot;**— 백엔드에서 제시한 사유와 함께 표시되는 빨간색 경고문, 차단 요소 목록 및 파란색**해결 방법 제안** (&quot;이 어레이를 네트워크에 맞추려면&quot; / &quot;동시 캡처를 활성화하려면&quot;).**적용 및 연결**은 예측 결과가 나올 때까지 잠겨 있으며, 레이블을 통해 거부 사유를 확인할 수 있습니다:

| 버튼 레이블 | 의미 | 실제로 도움이 되는 조치 |
| --- | --- | --- |
| &quot;분석 중...&quot; | 분석이 아직 진행 중입니다. | 기다리세요. |
| **&quot;이 네트워크에 카메라가 너무 많습니다&quot;**| 어레이가 회선 용량을 초과합니다(총합 검사 실패). | 카메라 수를 줄이거나, 종단 간 점보 프레임을 사용하거나, 더 빠른 NIC를 사용하세요.**ROI를 줄여도 소용없습니다** — 아래 참조. |
| **&quot;ROI를 줄여야 활성화됨&quot;** | 이 설정에서는 프레임이 손실될 수 있습니다(버스트/링 검사 실패). | ROI를 줄이거나, 비닝을 높이거나, NIC 수신 링을 수정하십시오. |

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Settings scene, over-subscribed state: red "Wire budget ... over-subscribed" line, the "Max cameras on this wire" hint, and the Apply button reading "Too many cameras for this network". Reproduce by configuring more cameras than the 1 GbE ceiling (e.g. 7+ cams at 1500 MTU) or with CHLOROS-simulated models via `lattice analyze-array`. -->

연결 중에 시리얼별 진행률 표시줄이 포함된 녹색 **보정 데이터 다운로드 패널**이 나타날 수 있습니다. 컴퓨터에 카메라를 처음 연결할 때, Chloros는 GigE를 통해 카메라에서 약 3.8 MB 크기의 공장 보정 팩을 가져옵니다(카메라당 약 70초 소요). 캐시된 카메라에는 이 패널이 표시되지 않습니다. [카메라 연결](connecting.md)을 참조하십시오.

## 대역폭: 수용 가능한 카메라 수

어레이가 처리할 수 있는 용량은 Chloros가 아닌 네트워크 회선의 특성에 따라 결정되므로, 계획에 필요한 수치는 하드웨어 설명서에 나와 있습니다: **[어레이 대역폭 계획](https://mapir.gitbook.io/lattice-camera/setup/array-bandwidth-planning)**.

Chloros가 이를 어떻게 처리하는지: 연결 대화 상자는 네트워크 프로브를 실행하고, 달성 가능한 프레임 속도를 예측한 후, 이에 맞는 티어를 선택합니다. 어레이가 회선 용량을 초과할 경우, 패킷을 무음으로 드롭하는 대신 연결을 거부합니다 — 위에서 설명한 예상 결과 패널을 참조하십시오.

## 프레임이 누락될 때

카메라가 공개된 그룹에서 누락될 수 있는 이유는 완전히 서로 다른 두 가지가 있으며,
각각에 대해 정반대의 해결책이 필요합니다. Chloros는 두 가지 원인을 모두 명시하지 않은 하나의
“불완전한” 수치를 보고하는 대신, 이를 별도로 집계합니다:

| 발생 상황 | 의미 | 확인 위치 |
| --- | --- | --- |
| **손상됨**— 프레임이 도착했으나 구조적으로 결함이 있음 | 네트워크 경로상의 GVSP 패킷 손실 |**와이어 예산**, NIC 수신 링, 점보 프레임, 스위치 |
| **도착하지 않음**— 프레임이 전혀 도착하지 않음 | 카메라가 작동하지 않았거나, 카메라에서 데이터가 전송되지 않음 |**M8 동기화 케이블**, 동기화 라인, 모든 구성원이 작동 상태인지 여부 |

어레이가 스트리밍되는 동안 분할 상태는 10초마다 재평가됩니다. 5%를 초과하면
두 수치가 모두 명시되어 로그에 기록되며, 각 카메라별로 손상된 버퍼는 처음 발생했을 때
보고된 후, 긴 세션에서도 가독성을 유지할 수 있도록 1분마다 집계됩니다.

**‘도착하지 않음’이 0인 손상된 프레임은 트리거링과 케이블 동기화가 완벽하다는 뜻**이며,
모든 손실된 프레임은 네트워크 경로에서 발생합니다. 해결 방법은 **와이어 예산**을 낮추고
다시 연결하는 것입니다.

{% hint style="warning" %}
**트리거 속도를 낮추는 것은 손상된 프레임 문제 해결에 도움이 되지 않습니다.** 카메라의 패킷
페이싱은 연결 시 한 번만 설정됩니다. 트리거 속도를 낮추면 버스트가 발생하는 빈도는
변화할 뿐, 버스트 자체가 네트워크로 전송되는 속도는 변하지 않습니다. 측정된 4대 카메라 시스템에서
트리거 속도를 5배 낮추어도 아무런 변화가 없었지만, 와이어 예산을 240에서
200 MB/s로 낮추자 같은 시스템의 손상률이 10.4%에서 0%로 떨어졌습니다.
{% endhint %}

실행 중인 어레이는 자체적으로 재계획을 수립할 수 없습니다. 연결을 끊었다가 다시 연결해야 연결 시점
선택기가 새로운 전송 용량에 맞춰 작동할 수 있습니다.

### USB 네트워크 어댑터의 전송 속도는 200 MB/s로 제한됩니다.

USB 이더넷 어댑터는 *이더넷* 링크 속도를 표방하지만, 실제로
유지할 수 있는 속도는 USB 버스 및 해당 드라이버에 의해 제한됩니다. USB 10GbE 동글은 과거에
대략 1000 MB/s의 처리량을 제공한다고 알려져 있었으나 — 실제로 측정된 적은 한 번도 없는 수치였습니다 — 이 유령 같은 여유 용량을 기준으로
4대의 카메라를 제어했을 때 프레임의 6–18 %가 손상되었음에도 불구하고 어레이는
여전히 정상적인 목표 프레임 속도를 보고했습니다. 현재 USB 연결 어댑터의 상한선은
**200 MB/s**로 제한됩니다. 이 제한은 백분율이 아닌 절대값입니다. 왜냐하면 한계는
버스에 있기 때문이며, USB 1 GbE 어댑터는 약 80 MB/s를 제공하므로 이 제한의 영향을 받지 않습니다.

호스트의 속도가 실제로 이 제한보다 빠르다면, **Wire Budget**을 높여 이를 반영하십시오.

## PTP 시간 동기화

프레임 *동기화*는 하드웨어 트리거에서 비롯되며, **PTP**(IEEE 1588 PTPv2)는 모든 장치에 걸쳐 일관된 *타임스탬프*를 제공합니다. 이는 어레이 연결 시 기본적으로 활성화됩니다:

* **Chloros 호스트 백엔드는 PTP 그랜드마스터를 실행합니다**. LATTICE 카메라와 DAQ-E 광센서는 도메인 0에서 이에 슬레이브로 연결되므로, 이미지 타임스탬프와 DAQ 스펙트럼은 하나의 클럭 (~1 ms)에 맞춰집니다.
* `--no-ptp` (CLI)는 벤치 작업 시 이를 비활성화합니다. 이 경우 카메라 간 타임스탬프는 **비교할 수 없습니다**.
* CLI를 사용하여 동기화 상태를 확인하십시오:

```bash
chloros-cli time-sync status     # grandmaster state, clock identity
chloros-cli time-sync peers      # slaves seen (cameras + DAQ-E sensors)
chloros-cli time-sync cameras    # per-camera PtpStatus / PtpOffsetFromMaster / PtpMeanPathDelay
```

&#x27;카메라(Cameras)&#x27; 탭 자체에는 PTP 표시기가 없습니다. 이 탭에서 확인할 수 있는 카메라별 동기화 정보는 읽기 전용인 **역할(Role)**(마스터/슬레이브),**동기화 라인(Sync Line)**, 그리고 어레이의**기능(Capabilities)** 계층입니다. DAQ-E의 PTP 상태는 &#x27;광 센서(Light Sensors)&#x27; 탭의 센서 세부 정보에 표시됩니다.

## 라이브 어레이 보기

<!-- SCREENSHOT-NEEDED: Cameras tab with a connected combined array: sidebar showing the ARRAY row (color badge, array name, "DAQ · on" pill) with indented member camera rows, and the main area showing the combined index composite tile with the LUT-colored NDVI render, top-left array name pill, and top-right fps readout. -->

메인 피드 영역에서는 두 가지 레이아웃을 제공합니다(상단 바에서 전환 가능): **그리드 보기**(각 타일은 하나의 셀이며, 그리드 자물쇠가 해제된 상태에서 드래그하여 순서를 재정렬할 수 있음)과**목록 보기**(상단에 전체 너비의 어레이, 그 아래에 활성화된 카메라 하나)가 있습니다.**피드 확대/축소** 슬라이더로 타일 크기를 조절할 수 있으며, 셀 너비가 200px 미만이면 이름/fps 오버레이가 자동으로 숨겨집니다.**별도 모드**에서는 카메라당 하나의 타일이 표시됩니다. 각 타일에는 다음 정보가 오버레이됩니다:

* 카메라 이름(왼쪽 상단),
* **fps 표시값**(오른쪽 상단) — 이는 백엔드에서 보고된 카메라의 *실제 캡처 속도*이며, 미리보기 폴링 속도가 아닙니다(라이브 미리보기는 캡처 속도와 관계없이 30 fps로 제한됨),
* 상태 표시점 — 녹색(스트리밍 중) / 주황색(로딩 중) / 빨간색(오류),
* 2초 동안 새로운 프레임이 도착하지 않을 때 표시되는 **오래된 프레임 스피너** — 연결/해제 후 약 5초 동안은 백엔드가 카메라 간 대역폭 할당을 재조정하는 동안 정상적으로 나타납니다.**통합 모드**는 단일 합성 타일을 표시합니다. 백엔드는 디베이어링, 크기 조정, 정렬, 노이즈 제거를 수행하고, 대역별 복사도(라디언스)로 변환하며(조명 센서가 바인딩된 경우 DLS 반사율 포함), 어레이의 인덱스 표현식을 평가하고, LUT를 적용한 후, 결과를 MJPEG 형식으로 스트리밍합니다. 첫 번째 정렬된 프레임이 렌더링될 때까지 타일에는 현재 상태가 표시됩니다: &quot;배열 준비 중…&quot;, &quot;정렬 보정 중…&quot;, &quot;첫 번째 프레임 대기 중…&quot; 또는 — 자동 정렬 재시도 허용 시간(약 30초)이 소진된 경우 —**정렬 보정** 버튼과 함께 &quot;정렬 필요&quot; 메시지가 표시됩니다.

결합 모드에 대한 유용한 정보:

* 합성 이미지는 **마스터**카메라의 프레임에 정렬됩니다. 합성 이미지에 대한 AE-ROI 조준 및 스팟 측광은 마스터 카메라에서는 정확하지만 슬레이브 카메라에서는 대략적입니다. 추가 카메라 연결을 열지 않고도 카메라별 픽셀 단위의 정확한 타일을 보려면**분할 보기**(어레이 설정 → &quot;구성원 카메라 표시&quot;)를 사용하십시오.
* **레이어 표시**(어레이 설정; 기본값: 꺼짐)를 사용하면 전경 및 배경 레이어를 선택할 수 있습니다. 멤버 카메라 또는**인덱스**중 어느 것이나 선택 가능합니다. 전경이**인덱스**로 설정된 경우, LUT 최소/최대 범위를 벗어난 픽셀에는 배경 레이어가 표시됩니다.
* **렌더 해상도**(기본값 720p)는 라이브 스트림의 높이 *및* 저장된 합성 영상의 내보내기 크기를 설정합니다. 카메라별 이미지는 항상 전체 해상도로 내보내집니다.
* 정렬은 세션별로 계산되며 절대 저장되지 않습니다. RMS 잔차 및 **재보정** 버튼에 대해서는 어레이 설정 패널의 정렬 섹션을 참조하십시오.

## 캡처: 모니터링 대 분석

어레이 캡처 영역은 **모니터링 등급**(보이는 대로 기록)과**분석 등급**(원시 데이터 기록, 나중에 보정)으로 명확하게 구분됩니다:

| 워크플로 | 등급 | 저장 내용 | GUI | CLI |
| --- | --- | --- | --- | --- |
| **캡처**(정지 이미지) | 분석 | 패스당 동기화된 프레임 그룹 1개; 선택한 모든 내보내기 수준(원시/데베이어링/방사도/반사도/미리보기/인덱스)별 카메라별 파일 + `.daq` 사이드카 |**모두 캡처** 버튼 + 캡처 설정 | `lattice array-capture` |
| **인덱스 비디오 녹화** | 모니터링 | 화면에 표시되는 실시간 결합 인덱스 합성 영상 — 8비트, 미리보기 해상도, LUT 적용됨; 라이브 스트림이 열려 있어야 함 | ● 인덱스 비디오 녹화 (결합 배열) | `lattice array-record` |
| **원시 버스트 → 비디오 생성**| 분석 | 전체 캡처 속도의 원시 센서 프레임 + 매니페스트 + `.daq`, 이후 DAQ 측정값과 시간 동기화된 보정된 방사도/반사도/지수 비디오로 오프라인 재구성 | ⦿ 원시 버스트 기록 →**비디오 생성** | `lattice array-burst` → `lattice array-build-video` |

경험적 규칙: 픽셀 데이터가 *측정값*으로 활용될 경우, 캡처 또는 버스트(분석 등급)를 사용하십시오. 어레이가 포착한 내용을 단순히 *확인하거나 시연*하기만 하면 되는 경우, 인덱스 비디오(모니터링 등급)를 기록하십시오.

### 캡처 설정 (GUI)

<!-- SCREENSHOT-NEEDED: Capture Settings pane (gear next to Capture All) with a connected array: capture-mode buttons (Single/Continuous/Interval), the bulk export-type toggle row, the Fastest Capture toggle, and the per-array group card showing the Aligned checkbox and the "Record index video" / "Record raw burst" buttons. -->

**Capture All** 옆의 톱니바퀴 아이콘을 클릭하면 캡처 설정 패널이 열립니다(프로젝트가 열려 있어야 함 — 캡처 데이터는 해당 프로젝트에 저장됨):

* **캡처 모드**:**단일**(한 번만 실행) /**연속**(연속 촬영; 캡처 횟수(기본값 1) 또는 시간(기본값 10초)으로 제한됨) /**간격** (타임랩스: X 간격마다 N 장씩 총 Y 장 촬영, 기본값은 5초마다 1장씩 1분 동안).
* **카메라별 내보내기 유형**: Raw, 디베이어링 처리, 방사도, 반사도, 미리보기, 인덱스 — 적용 가능한 모든 항목이 기본적으로 활성화되어 있습니다. RGB-필터 카메라의 경우 Radiance/Reflectance는 숨겨져 있습니다.**Reflectance는 카메라에 DAQ 광 센서가 있을 때만 표시됩니다**(자체 센서이거나 어레이에서 상속받은 경우); Index는 구성된 인덱스 표현식이 필요합니다.
* **정렬**(어레이별, 기본값**켜짐**): 멤버 내보내기 데이터를 어레이의 정렬 프로필에 맞춰 왜곡 보정하여 픽셀 단위로 정렬된 상태로 내보냅니다. Raw는 항상 왜곡 보정되지 않은 상태로 유지되지만 메타데이터에 변환 정보가 포함됩니다.
* **가장 빠른 캡처** (토글): 원시 데이터 전용 + 할당된 DAQ 판독값 + 무료 결합 지수 합성값을 사용하며, 최대 속도를 위해 캡처 시 보정 계산을 건너뜁니다. 저장된 `.daq` 데이터를 사용하여 나중에 복사도/반사도/지수를 재구축합니다.
* 선택 항목은 프로젝트와 함께 유지됩니다. 숨겨지거나 일시 중지된 카메라는 건너뜁니다.

이에 상응하는 CLI (동일한 백엔드 엔드포인트, 동일한 의미론):

```bash
# One synced group, every applicable export level per camera (the default)
chloros-cli lattice array-capture -o output/

# Interval timelapse: one reflectance pass every 10 s for 5 minutes
chloros-cli lattice array-capture --interval 10 --duration 300 --processing reflectance -o timelapse/

# Fastest grab for a moving rig — raw + .daq now, calibrate later
chloros-cli lattice array-capture --fastest -o flightline/

# 30-second monitoring clip of the combined index view, plus a GIF
chloros-cli lattice array-record --duration 30 --fps 10 --gif -o monitoring/

# 5-second analysis-grade raw burst, then build the combined index video
chloros-cli lattice array-burst --duration 5 --build --products combined:index --fps 10 -o capture/
```

캡처용 TIFF 압축은 `deflate`(무손실, 기본값) 또는 `none`입니다. — 전체 플래그 테이블, 캡처 폴더 구조 및 재처리 규칙은 [CLI 참조](../reference/cli-reference.md#capture-modes-recorders--offline-reprocess)에 나와 있습니다.

## DAQ 광 센서 페어링

반사율 및 조도 보정된 미리보기를 사용하려면 **광 센서** 탭에 연결된 DAQ 센서에서 하향광 데이터가 필요합니다:

* 사이드바의 **어레이 행**에는**&quot;DAQ · 켜기/끄기&quot; 버튼**이 표시됩니다. 어레이 수준의 광 센서가 설정되어 있거나 구성원 카메라 중 하나라도 자체 광 센서를 가지고 있을 때 *켜짐* 상태가 되며, 툴팁에는 어떤 센서가 어떤 카메라에 데이터를 제공하는지 정확히 나열됩니다.
* 어레이 설정 → **주변광 센서**→**광 센서** 드롭다운 메뉴에서 어레이 전체에 적용할 센서를 지정합니다. 이 선택 사항은 프로젝트와 함께 유지되며 모든 구성 카메라에 적용되지만, 개별 카메라는 자체 센서를 사용하여 이 설정을 재정의할 수 있습니다.
* 그 아래의 상태 줄에는 실시간 상태가 표시됩니다: **Off**→ &quot;첫 번째 스펙트럼을 기다리는 중…&quot; →**&quot;활성 — 어레이 내 모든 카메라의 조도가 보정됨&quot;** → 또는, 지난 3초 동안 새로운 스펙트럼이 도착하지 않은 경우, 오래된 데이터 알림 — 마지막 측정값이 계속 사용됩니다(캡처 경로에서는 측정값이 만료되지 않습니다).

센서가 할당된 경우: 반사율 내보내기 유형을 사용할 수 있게 되며, 실시간 미리보기가 조도 보정되고, 예측 자동 노출이 스펙트럼을 사용할 수 있으며, 모든 반사율 캡처 시 실제로 사용된 DAQ 측정값이 **`.daq` 사이드카** 이미지 옆에 저장되므로, 나중에 캡처를 재처리할 수 있습니다.

## `array-connect` CLI 옵션

| 플래그 | 기본값 | 설명 |
| --- | --- | --- |
| `--serials SN1,SN2,…` | 모든 LATTICE 카메라 자동 탐지 (2대 이상 필요) | **첫 번째 시리얼이 마스터입니다.** |
| `--line {Line0,Line2,Line3}` | `Line2` | GPIO 동기화 라인. |
| `--target-fps F` | auto | 마스터 트리거 발사 빈도. |
| `--binning {1,2,4}` | 자동 | 하드웨어 비닝. |
| `--force-tier {sim-capture-sim-emit, sim-capture-ftd-stagger, slip-emit-and-capture}` | 자동 | 동기화 계층 선택기의 전문가 오버라이드. |
| `--wire-ceiling-mbps MB_PER_S` | 자동 감지 | 호스트 와이어 예산(MB/s) — **와이어 예산** 필드의 CLI 형식입니다. 어레이에서 손상된 프레임이 보고되면 이 값을 낮추십시오. 프로젝트와 함께 저장되므로, 나중에 다시 연결하면 이 설정이 복원됩니다. |
| `--no-recommend` | 꺼짐 | 네트워크 분석 단계를 건너뜁니다. |
| `--no-ptp` | 꺼짐 | PTP를 비활성화합니다(이 경우 카메라 간 타임스탬프 비교가 불가능해집니다). |

`lattice array-list`, `array-status`, 및 `array-disconnect`는 지속 세션을 관리합니다. 정렬(`align-calibrate` / `align-apply`) 및 네트워크 도구를 포함한 전체 하위 명령어 참조는 [CLI 참조 § chloros-cli lattice](../reference/cli-reference.md#chloros-cli-lattice)에 있습니다. 이에 상응하는 SDK 명령어 (`connect_array`, `ArraySession`, `attach_array`, `analyze_array_network`)는 [SDK 참조](../reference/sdk-reference.md)에 포함되어 있습니다. Python부터의 와이어 예산은 `connect_array(..., wire_ceiling_mbps=120)`이며, 활성 데이터 손상/미도착 분할은 [`/api/camera/array/<id>/capability`](../reference/sdk-reference.md#array-health--which-subsystem-is-losing-frames)에 있습니다.
