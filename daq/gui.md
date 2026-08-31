# Chloros의 DAQ 탭

Chloros 사이드바에서 **광 센서**로 표시된 DAQ 탭은 [DAQ-U, DAQ-M 및 DAQ-E 광 센서](README.md)를 위한 실시간 제어 패널입니다: 어떤 전송 프로토콜을 통해서든 센서를 연결하고, 보정된 스펙트럼을 실시간으로 확인하며, 센서 쌍을 통해 실시간 반사율을 계산하고, `.daq` 파일을 프로젝트에 바로 기록할 수 있습니다.

이 탭은 Chloros 백엔드의 시작이 완료되면 사용할 수 있게 됩니다. 이 탭의 차트는 Chloros의 DAQ 서비스를 통해 실시간으로 데이터를 제공받으며, 연결이 중단될 경우 자동으로 재연결(2–10초 백오프)됩니다. 서비스에 접속할 수 없는 동안에는 센서의 ‘상태’ 행에 **No Server**라고 표시됩니다.

레이아웃은 **센서 사이드바**(연결된 센서당 한 행)와**차트 영역**(센서 또는 그룹당 하나의 차트 타일)으로 구성됩니다.

<!-- SCREENSHOT-NEEDED: full DAQ (Light Sensors) tab in list view with one DAQ-E connected — sensor sidebar on the left (Connect Sensor + Record All buttons, one sensor row), spectrum chart with rainbow fill in the main area, live data table below the chart -->

***

## 센서 연결

사이드바 상단의 **센서 연결**을 클릭합니다. 연결 대화 상자가 메인 영역에 열립니다(또는 다른 센서를 추가할 때는 오버레이 형태로 표시되며, 이 경우**취소** 버튼이 나타납니다).

| 제어 | 동작 |
| --- | --- |
| **장치 유형** | `DAQ-U (USB)`(기본값), `DAQ-M (Bluetooth)` 또는 `DAQ-E (Ethernet)`. 유형을 변경하면 새로 선택한 전송 방식에 대한 스캔이 다시 시작됩니다. |
| **포트 / BLE 장치 / 호스트 이름 / IP** | 탐지된 장치를 `device - description` 형식으로 나열하며, 센서로 인식된 첫 번째 항목이 자동으로 선택됩니다. 스캔 중에는 `Scanning...`(USB), 8초 카운트다운이 표시된 `Scanning (N)...`(BLE), 또는 5초 카운트다운이 표시된 `Discovering ethernet sensors (N)...` (이더넷)이 표시됩니다. 결과가 없는 경우 `No ports` / `No BLE devices` / `No ethernet sensors found`로 표시됩니다. |
| **↻ 새로 고침** | 선택한 전송 방식을 즉시 재스캔합니다(BLE/이더넷 스캔 중에는 비활성화됨). |
| **연결** | 장치가 선택되면 활성화되며, 연결이 이루어지는 동안 `Connecting...`로 레이블이 변경됩니다. |

탐색 기능은 **연결 대화 상자가 화면에 표시되어 있는 동안**에만 실행되며, 선택한 전송 방식에 대해서만 15초마다 반복됩니다. 탭을 열기만 해서는 스캔이 수행되지 않습니다. 실패 시 대화 상자에 다음과 같은 메시지가 표시됩니다. *&quot;연결에 실패했습니다. 센서를 분리했다가 다시 연결한 후, &#x27;연결&#x27;을 다시 클릭해 주세요.&quot;*

첫 번째 센서가 연결되면 사이드바가 자동으로 열립니다.

{% hint style="info" %}
**DAQ-E가 표시되지 않나요?** DAQ-E에는 상태 LED가 없습니다. 장치가 연결된 스위치 또는 인젝터 포트의 PoE/링크 표시등을 확인하고, 전원을 켠 후 부팅이 완료될 때까지 몇 초간 기다려 주십시오. Chloros 장치는 동일한 브로드캐스트 도메인에 있어야 합니다(mDNS는 라우터를 통과하지 못합니다). Windows에서, Chloros가 멀티캐스트 소켓(mDNS UDP 5353, DAQ-E 데이터 UDP 5002, PTP UDP 319/320)을 바인딩할 때 Defender 방화벽의 메시지를 수락하십시오. 하나의 LAN에 있는 두 대의 DAQ-E 장치는 각각 고유한 `daq-e-<id>.local` 호스트명으로 별도로 탐지됩니다.
{% endhint %}

<figure><img src="../.gitbook/assets/v120-daq-device-type.png" alt=""><figcaption>장치 유형에는 DAQ-U(USB), DAQ-M(블루투스) 및 DAQ-E(이더넷)가 제공됩니다.</figcaption></figure>***

## 센서 사이드바

연결된 각 센서마다 한 행이 할당되며(Ambient+Object 그룹당 한 행이 추가됨), 행은 드래그하여 순서를 재정렬할 수 있으며, 행의 순서에 따라 차트 타일의 순서도 재정렬됩니다. 행을 클릭하면 해당 센서/그룹이 목록 보기의 활성 차트가 됩니다.

| 요소 | 의미 |
| --- | --- |
| 왼쪽의 색상 테두리 | 센서 그래프의 색상. |
| 전송 배지 | `DAQ-U` / `DAQ-M` / `DAQ-E`, 또는 Ambient+Object 반사율 그룹의 경우 녹색 `REF` 배지. |
| 장치 이름 | 기본값은 센서의 일련번호(교정, `.daq` 파일 이름 및 가져오기 일치 여부를 위한 고정된 식별자)이며, 사용자 지정 이름은 프로젝트별로 유지됩니다. |
| **보정됨** 알림(녹색) | 센서의 공장 보정 번들이 로드되었을 때 표시되며, 즉 스펙트럼이 실제 W/m²/nm 단위임을 나타냅니다. |
| **업데이트 가능** 표시 (황색, DAQ-E 전용) | 현재 실행 중인 펌웨어 버전이 이 Chloros 빌드에 포함된 이미지보다 오래된 경우. 업데이트 중에는 실시간 진행 상황이 표시됩니다 (`Flashing… N%`, `Restarting sensor…`, 그 다음 `Updated X → Y` 또는 `Failed`). |
| 눈 | 차트에서 이 센서의 표시 여부를 전환합니다. |
| 톱니바퀴 | 센서별 설정 모달 창을 엽니다(아래 참조). |
| ✕ (빨간색) | 센서 연결을 끊거나, Ambient+Object 그룹을 제거합니다. |

행 위쪽에는 두 개의 버튼이 있습니다:

* **센서 연결** — 연결 대화 상자를 엽니다(작업 중에는 `Connecting...`로 레이블이 변경됨).
* **모두 기록 / 모두 중지**— 연결된**모든**센서에 대해 `.daq` 기록을 시작하거나 중지합니다. 최소 한 개의 센서**및 열려 있는 프로젝트**가 필요합니다(도구 설명: &quot;기록하려면 프로젝트를 여십시오&quot;); 기록이 진행 중일 때는 빨간색으로 바뀝니다.

비어 있는 상태에서는 &quot;연결된 센서 없음&quot;이라고 표시됩니다.

<!-- SCREENSHOT-NEEDED: sensor sidebar with three rows — a DAQ-E showing both the green Calibrated pill and the amber Update Available pill, a DAQ-U row, and a green REF group row — plus the Connect Sensor and Record All buttons -->

***

## 센서별 설정 (톱니바퀴 모달)

센서 행의 톱니바퀴 아이콘을 클릭하여 엽니다. 순서대로 표시되는 항목:

* **정보 행** — 장치 유형 (DAQ-U/M/E), 연결 (`Serial (USB)` / `Bluetooth` / `Ethernet`), 포트(COM 포트, BLE 주소 또는 호스트), 시리얼.
* **교정 보고서: 다운로드** — 해당 장치의 NIST 추적 가능 교정 인증서(PDF)를 가져와 PDF 뷰어에서 엽니다. 시리얼 번호가 확인된 후 사용할 수 있으며, 인증서는 첫 연결 시 캐시에 저장됩니다.
* **장치 이름** — 연필 아이콘을 클릭하여 이름을 변경할 수 있으며, 프로젝트별로 저장됩니다.
* **그래프 선 색상** — 색상 견본; 프로젝트별로 유지됩니다.
* **적분 시간(ms)**— 슬라이더 + 숫자,**1–500 ms**, 기본값**32 ms**. AE가 켜져 있는 동안에는 비활성화됩니다.
* **프레임 평균**— 슬라이더 + 숫자,**1–50 프레임**, 기본값**20**.
* **AE: ON/OFF**— 자동 노출 토글; 연결 시**기본값 ON**. 노출 시간을 수동으로 설정하려면 이 기능을 끄십시오.
* **스트리밍 중지 / 스트리밍 시작** — 라이브 스트리밍을 일시 중지하거나 재개합니다.
* **녹화 / 녹화 중지** — 센서별 `.daq` 녹화(열린 프로젝트가 필요함).
* **캡** — 캡 보정 프로필(다음 섹션).
* **실시간 정보 행** — 노출 시간(ms), FPS, 샘플 수, 녹화 중(빨간색 `REC` 또는 `Off`), 상태(`Streaming` / `Paused` / `SATURATED` / `No Server`).

### DAQ-E 전용: 네트워크, 펌웨어 및 PTP 행

* **호스트 이름 / IP** — 장치의 현재 주소.
* **펌웨어** — 현재 펌웨어 버전과 함께 작업 셀이 표시됩니다.<version\>

이 Chloros 빌드가 더 최신 버전의 DAQ-E 펌웨어 이미지를 포함하고 있을 경우</version\>

, **Update to \<version\>** 버튼</version\>

이<version\>

나타납니다. 업데이트는 네트워크를 통해 약 30초 만에 완료되며, 센서는 자동으로 재시작 및 재연결됩니다. 전송이 중단되더라도 현재 펌웨어는 그대로 유지됩니다. 진행 상황은 실시간으로 표시되며(`Flashing… N%` → `Restarting sensor…` → `Updated X → Y`), 최신 버전일 때는 셀에 `Up to date`로 표시됩니다.
* **PTP 동기화** — 실시간 PTP 상태(`unknown`로 폴백). DAQ-E 펌웨어 v1.2.0 이상은 IEEE 1588 PTPv2에서 슬레이브 전용 클럭으로 참여합니다. Chloros 호스트의 백엔드는 PTP 그랜드마스터이며, LAN에 연결된 모든 DAQ-E 및 LATTICE 카메라는 도메인 0에서 이 그랜드마스터에 종속되어 타임스탬프 오차를 대략 1ms 이내로 유지합니다.

Ambient+Object 그룹의 경우, 기어 모달에는 해당 그룹의 소스 센서, 장치 이름 및 그래프 선 색상만 표시됩니다.

<!-- SCREENSHOT-NEEDED: per-sensor settings modal for a DAQ-E — info rows, Calibration Report Download, Hostname/IP + Firmware row with an "Update to <ver>" button, PTP Sync row, Integration Time / Frame Average sliders, AE ON toggle, and the Cap dropdown all visible (scrolled composite acceptable) -->

### 캡 선택

**캡** 드롭다운 메뉴는 Chloros에 센서의 디퓨저 위에 어떤 물리적 캡이 장착되어 있는지 알려주며, 해당 캡의 공장 측정 보정 프로파일을 모든 스펙트럼에 적용합니다. 선택 사항은 모델에 따라 다릅니다:

| 모델 | 캡 선택 항목 |
| --- | --- |
| DAQ-U | 없음(노출된 센서), FOV 15°, FOV 30°, FOV 45°, FOV 60°, FOV 90°, Sunshine(코사인 보정기) |
| DAQ-M | 없음(노출 센서), Sunshine(코사인 보정기) |
| DAQ-E | 없음(노출 센서), FOV 15°, FOV 45°, FOV 90°, Sunshine (코사인 보정기) |

**모든 모델의 기본 설정은 ‘Sunshine’(코사인 보정기)입니다** — MAPIR는 모든 DAQ에 ‘Sunshine’ 캡이 장착된 상태로 출하되며, 이는 표준 실외 구성입니다: 60°까지 코사인 오차 ≤ ±4 %, 70°까지 ≤ ±4.5 %인 180° 반구형 시야(태양 고도 ~15° 미만에서는 권장하지 않음)를 제공하며, 설계상 감쇠율 (~12배)이 적용됩니다. 선택한 설정은 프로젝트에 유지됩니다.

{% hint style="warning" %}
**캡 선택은 실제 장착된 캡과 일치해야 합니다.**센서나 소프트웨어 모두 어떤 캡이 장착되었는지 감지할 수 없습니다. 이 선택은 실시간 보정과 모든 `.daq` 파일에 기록되는 스탬프 값을 모두 결정합니다. Sunshine 캡의 약 12배 감쇠를 고려할 때, 캡 변경을 명시하지 않으면 스펙트럼이 대략 그 배수만큼 잘못 보정됩니다. (동일한 캡을 제거했다가 다시 장착하면 오차는 약 1.5% 수준으로 반복됩니다.) 캡이 물리적으로 제거된 경우에만**None (베어 센서)**를 선택하십시오. DAQ-E의 경우, “None”을 선택해도 오목한 유리 디퓨저에 대한 공장 출하 시 기하학적 프로파일이 여전히 적용됩니다(무작동 모드가 아님). 또한, 캡이 제거된 DAQ-E는 벤치 구성일 뿐, 지원되는 현장 구성은 아닙니다.
{% endhint %}

{% hint style="info" %}
이전 설명서에서 업그레이드 시 참고: 1.1.0 버전에서 제공되던 브라우저 측의 “Sunshine Diffuser Installed” 토글 기능이 사라졌습니다. 캡 처리는 이제 서버 측에서 적용되는 센서별 캡 프로필 방식으로 변경되었습니다.
{% endhint %}

***

## 차트 영역

상단에 고정된 바에는 **목록 ⇄ 그리드 보기 전환**버튼과**차트 확대/축소** 슬라이더(타일 크기 200–2000 픽셀)가 있습니다. 차트 그룹이 두 개 이상일 경우 보기가 자동으로 그리드 모드로 전환되며, 하나 이하일 때는 다시 목록 모드로 돌아갑니다. 보기 모드와 차트 크기는 프로젝트별로 유지됩니다.

각 센서의 **스펙트럼 차트**에는 다음이 표시됩니다:

* **X축** — 파장(nm). 센서 그리드는 5nm 간격으로 340–1010nm(135개 데이터 포인트)이며, 표시 시 1nm로 보간됩니다.
* **Y축** — 전력(W/m²), 피크 값을 기준으로 자동 선택된 SI 접두사(m/µ/n)가 적용됩니다. 스펙트럼은 세 가지 전송 방식 모두에서 방사계측적으로 보정된 스펙트럼 조도(W/m²/nm)로 표시됩니다.
* 단일 트레이스 아래에는 무지개색 스펙트럼 채우기가 표시되며, 하나의 차트에 여러 센서가 중첩된 경우 채우기 색상이 흐려진 채로 색상 선으로 표시됩니다.
* **마우스 오버**— 파장과 센서별 값이 표시된 수직 커서가 나타납니다.**드래그**하여 확대/축소할 수 있습니다(확대 시 축소 버튼이 나타납니다).
* **+** 버튼(그리드 보기 전용)을 사용하여 이 차트에 센서를 추가하거나 그룹을 생성할 수 있습니다(아래 참조).
* 상단 중앙에 장치 이름이 표시되며, 첫 번째 프레임이 도착할 때까지 로딩 아이콘이 나타납니다.

**포화** 상태는 차트 자체에는 표시되지 않습니다. 포화 상태인 센서는 빨간색 `SATURATED` 상태 텍스트를 표시하며, 실시간 데이터 테이블에서는 빨간색 `Saturated: Yes` 행으로 나타납니다. 이 상태를 해제하려면 통합 시간을 줄이거나 AE를 다시 활성화하십시오.

<!-- SCREENSHOT-NEEDED: grid view with at least two chart tiles visible, the Chart Zoom slider and list/grid toggle in the top bar, and the "+" add-sensor button visible on one tile -->

***

## 실시간 데이터 테이블(목록 보기)

목록 보기의 차트 아래에 있으며, 500ms마다 새로 고침됩니다:

* **모든 모델**: 광색 견본 (CIE XYZ 기준 sRGB), 포화 (예/아니요), CIE 1931 X/Y/Z, 색도 x/y, CIE u′/v′, CCT (K), CRI (Ra), 주파장 (nm), 피크 파장 (nm), 여기 순도, Duv, CIE L\*/a\*/b\*, 먼셀 H/V/C.
* **교정된 센서에만 해당**(DAQ-U / DAQ-M / DAQ-E 중 공장 교정 번들이 로드된 경우 — 센서 행에 표시된 녹색**교정됨** 배지가 이를 나타냄): 총 광도 (W/m²), 광시야 럭스 (lx), 스코토픽 럭스(lx), S/P 비율, PPFD 및 PPFD Red/Green/Blue (µmol/m²/s), 그리고 오픽 조도 — S-원추세포, 멜라노픽, 로도픽, M-원추세포, L-원추세포 (모두 W/m²).

<!-- SCREENSHOT-NEEDED: list view live data table for a DAQ-E showing both the colorimetric rows and the power-calibrated rows (Total Power, Photopic/Scotopic Lux, PPFD, opic irradiances) -->

***

## 반사율 그룹 (주변광 + 대상)

연결된 두 개의 센서를 결합하여 카메라 없이도 실시간 반사율 디스플레이를 생성할 수 있습니다:

1. 그리드 보기에서 차트 타일의 **+**를 클릭하고**주변광 + 물체 결합**을 선택합니다.
2. **주변광 광원**센서와**피사체 스캐너**센서(서로 다른 두 개의 센서)를 선택한 다음**생성**을 클릭합니다.

Chloros는 두 실시간 스트림에서 파장당 R(λ) = 물체(λ) / 주변(λ)을 계산합니다(주변값이 0 이하일 경우 0). 그룹의 레이블은 센서의 보정 등급에 따라 결정됩니다:

* 두 센서 모두 보정됨(번들 로드됨) → **&quot;겉보기 반사율&quot;**.
* 두 센서 중 하나라도 보정되지 않은 경우 → **&quot;상대 반사율&quot;**.

이 그룹은 사이드바에 녹색 `REF` 행으로 표시되며, 별도의 차트(무지개색 채우기, 마우스 오버 시 소수점 4자리까지 값 표시, 드래그 확대/축소)로 나타납니다.

**+**메뉴에서는**새 센서 추가** 기능을 제공하며, 다음 세 가지 배치 옵션이 있습니다: *새 센서 결합* (이 차트에 추가), *기존 센서를 여기로 이동*, 또는 *새 센서 보기* (별도의 차트).

<!-- SCREENSHOT-NEEDED: the "+" add-sensor overlay open on a chart tile showing the menu (Add New Sensor / Combine Ambient + Object / Cancel), and the Ambient + Object sub-dialog with its two sensor selects -->

### 식생 지수 표

목록 보기에서, 반사율 그룹 차트 아래에는 **파란색 450 / 녹색 550 / 빨간색 670 / NIR 800 nm** 대역 중심에서의 실시간 반사율을 바탕으로 계산된 식생 지수 표가 표시됩니다 (소수점 이하 4자리 값, 계산 불가능한 경우 `---`; 지수 이름 위에 마우스를 올리면 전체 이름이 표시됨):

* **항상 표시됨** (스케일 불변, 모든 센서 조합): NDVI, GNDVI, ENDVI, WDRVI, GRVI, CVI, GCI, MSR.
* **두 센서 모두 전력 보정이 완료된 경우에만** (두 번들 모두 로드된 경우): EVI, SAVI, OSAVI, GSAVI, GOSAVI, MSAVI2, RDVI, TDVI, LAI, NLI, MNLI, FCI, GEMI

<!-- SCREENSHOT-NEEDED: an Ambient+Object reflectance group in list view — reflectance chart labeled "Apparent Reflectance" with the vegetation index table below it showing live NDVI etc. -->

.***

## `.daq` 파일 기록

* 기록을 하려면 **열린 프로젝트**가 필요합니다. 그렇지 않으면 ‘모두 기록’(사이드바) 및 센서별 ‘기록’ 버튼이 모두 비활성화됩니다.
* 파일은 **`<project folder>/light_sensor/`**에 기록되며, 파일 이름에는 센서 ID와 타임스탬프가 포함되고, 장치 이름은 기록 내용과 함께 저장됩니다.
* 기록이 중지될 때(중지, 모두 중지, 녹화 도중 연결이 끊어짐)하면, 완성된 `.daq` 파일이 **열려 있는 프로젝트에 자동으로 추가**됩니다. 이 파일은 수동으로 추가할 필요 없이 프로젝트의 파일 목록에 표시되며, [반사율 처리](README.md)의 하향 복사 데이터로 사용할 준비가 되어 있습니다.
* 녹화 중에는 설정 모달의 실시간 행에 빨간색 `REC` 표시기가 나타납니다.

정량적인 조도 수치를 얻으려면 최소 15초 분량의 데이터를 평균화해야 합니다. 이는 기기의 특성일 뿐, 결함이 아닙니다.

<!-- SCREENSHOT-NEEDED: recording in progress — sidebar Stop All button in its red state and the settings modal live rows showing Recording: REC -->

***

## 다중 센서 레이아웃 및 프로젝트 지속성

* 여러 센서를 하나의 차트에 결합(공유 축), 별도의 차트로 유지(자동 그리드 레이아웃), 차트 간 센서 이동, 행/타일 드래그 및 재정렬, 눈 아이콘 토글을 통해 개별 센서 숨기기 등이 가능합니다.
* 프로젝트별로 Chloros는 다음 항목을 유지합니다: 장치 이름, 그래프 색상, 차트 크기, 보기 모드, 각 센서의 설정(적분 시간, 프레임 평균화, AE 상태, 캡 선택).
* **프로젝트를 다시 열면 주소를 통해 센서가 자동으로 재연결됩니다** — DAQ-U의 경우 COM 포트, DAQ-M의 경우 BLE 장치, DAQ-E의 경우 mDNS 호스트명(장치의 IP가 변경된 경우에도 해결됨) — 을 기준으로 센서를 자동으로 재연결하며, 각 센서에 저장된 캡 프로필, 프레임 평균화, AE 상태 및 수동 적분 시간을 다시 적용합니다.***

## 카메라 페어링 (DLS)

페어링할 필요가 없습니다. 사전에 광 센서를 카메라에 바인딩하는 드론 DLS 워크플로우와 달리, Chloros는 후처리 단계에서 DAQ 데이터를 이미지와 매칭합니다. 즉, 가져오기/처리 시점에 `.daq` 측정값이 각 캡처의 노출 타임스탬프에 맞춰 보간됩니다. 연결된 센서(`.daq`는 프로젝트에 자동으로 추가됨)로 기록하면, 반사율 처리 과정에서 시간에 따라 올바른 측정값을 찾아냅니다. 하향 데이터가 어떻게 사용되는지는 [DAQ 광 센서](README.md)를 참조하십시오.</version\>
