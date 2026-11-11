# 캡처 설정 및 모드

‘카메라’ 탭에서의 캡처는 하나의 빨간색 **모두 캡처**버튼과, 해당 버튼이 어떤 결과를 생성할지 결정하는**캡처 설정** 패널을 통해 제어됩니다. 여기에는 어떤 카메라가 참여할지, 각 카메라가 어떤 내보내기 형식으로 저장할지, 셔터가 한 번만 작동할지, 연속적으로 작동할지, 아니면 일정 간격으로 작동할지가 포함됩니다. 이 페이지에서는 구성, 캡처 과정, 파일이 디스크에 저장되는 위치, 그리고 나중에 보정된 결과물로 재처리하는 방법 등 전체 흐름을 설명합니다. 카메라 및 어레이 제어 기능은 [카메라 설정](camera-settings.md)에서 확인할 수 있습니다.

{% hint style="info" %}
**캡처를 수행하려면 프로젝트가 열려 있어야 합니다.** 프로젝트가 열려 있지 않으면 [모두 캡처] 및 [캡처 설정] 톱니바퀴 아이콘이 비활성화됩니다("캡처 내용을 저장하려면 프로젝트를 생성하거나 여십시오"). 모든 캡처 결과는 `captures/`의 프로젝트 폴더 아래에 저장됩니다.
{% endhint %}

## 캡처 설정 패널

사이드바의 카메라 목록에서 **'모두 캡처' 옆의 톱니바퀴 아이콘**을 클릭하거나, 개별 카메라 설정 패널 하단의**"캡처 설정 열기…"** 버튼을 클릭하여 열 수 있습니다. 헤더에는 "캡처 설정"이라는 문구와 ← 뒤로 가기 버튼이 표시됩니다.

<!-- SCREENSHOT-NEEDED: the full Capture Settings pane — Single/Continuous/Interval mode buttons at top, the bulk export-type toggle rows (All Raw … All Index), the orange Fastest Capture toggle, an array group card with the Aligned checkbox and Record buttons, and an expanded per-camera row showing per-type checkboxes. -->



여기에서 선택한 내용(포함된 카메라, 유형별 확인란, 캡처 모드)은 **프로젝트별로** 저장되며, 프로젝트를 다시 열 때 복원됩니다.

### 캡처 모드

패널 상단에 있는 세 가지 모드 버튼:

| 모드 | 기능 | 하위 설정(기본값) |
| --- | --- | --- |
| **단일** *(기본값)* | 선택한 모든 카메라에 대해 한 번의 캡처가 수행됩니다. | — |
| **연속**| 중지 조건이 충족될 때까지 연속적으로 캡처합니다. |**캡처 횟수**(기본값 1) *또는* **캡처 지속 시간**(기본값 10초; 단위: 초 / 분 / 시간 / 일)에 따라 중지됩니다. |
| **간격**(타임랩스) | 타이머에 따라 연속 촬영합니다. |**간격당 캡처 횟수**(기본값 1) ·**매**N 단위 (기본값 5초) ·**N** 단위 동안 (기본값 1분). |

연속 또는 간격 모드에서, 실행 중에는 ‘모두 캡처’ 버튼이 **중지 (N)** 버튼으로 바뀌며, 캡처가 완료될 때마다 횟수를 카운트합니다. 드라이브가 가득 차면 실행이 중지됩니다(['모두 캡처' 흐름](#the-capture-all-flow) 참조).**어레이에서의 연속 모드는 백엔드 내에서 실행됩니다(1.3.0부터 추가됨).** 목록에 있는 모든 카메라가 어레이에 속해 있는 경우(독립형 카메라는 숨겨지거나 제외됨), 연속 모드는 더 이상 캡처당 하나의 요청을 보내지 않습니다. Chloros 백엔드가 캡처 루프를 직접 실행하여, 다음 캡처를 선택하는 동안 동기화된 그룹 하나를 기록하고, 버튼은 그 카운트를 폴링합니다. 목록에 독립형 카메라가 포함된 경우, Chloros는 이전과 마찬가지로 패스 단위로 캡처를 수행합니다.

{% hint style="warning" %}
**Windows에서 빠른 캡처: Chloros 백엔드를 Microsoft Defender 스캔 대상에서 제외하십시오.** MAPIR의 테스트에 따르면, 고속 캡처가 작성하는 모든 파일을 실시간으로 스캔하는 데 캡처 속도의 약 30%가 소요됩니다 (작은 ROI에서 초당 수백 개의 파일). Windows 설치 프로그램에는 **"Microsoft Defender 스캔에서 Chloros 백엔드를 제외(카메라 캡처 속도 대폭 향상; 권장)"** 옵션이 있으며, 이 옵션은 기본적으로 선택되어 있고, 제거 시 제외 설정이 해제됩니다. 이 옵션을 해제했다면, 해당 옵션이 선택된 상태로 설치 프로그램을 다시 실행하거나, `chloros-backend.exe`(Chloros 설치 폴더 내 `backend` 폴더에 있음)에 대해 Defender 프로세스 제외 항목을 추가하십시오. 어레이의 연속 캡처 중에 Defender가 CPU 코어 하나 이상을 사용할 경우, Chloros 백엔드는 로그에 이를 한 번 기록하며, 앞서 언급한 두 가지 해결 방법을 제시합니다.
{% endhint %}

<!-- SCREENSHOT-NEEDED: the capture-mode area of Capture Settings with Interval selected — showing the "Captures / interval", "Every N (unit)" and "For N (unit)" rows with their defaults (1, 5 s, 1 m). -->




### 카메라 및 내보내기 유형 선택

이 패널의 도움말 텍스트에 요약되어 있습니다: “모두 캡처(Capture All)” 기능이 생성할 카메라와 내보내기 유형을 선택하십시오. 기본적으로 모든 항목이 선택되어 있으며, 선택한 내용은 이 프로젝트와 함께 저장됩니다.

* **모두 선택 / 모두 해제** 버튼을 누르면 모든 카메라의 포함 확인란이 한 번에 전환됩니다.
* **대량 내보내기 유형 토글**(두 줄의 버튼):**모든 Raw / 모든 디베이어링 / 모든 미리보기 / 모든 방사도 / 모든 반사도 / 모든 인덱스**. 각 항목은 3단계 색상으로 표시됩니다: 녹색 ✓ = 해당 기능을 지원하는 모든 카메라에서 활성화됨, 주황색 – = 일부 카메라에서 활성화됨, 회색 = 해당 기능을 지원하는 카메라 없음. 연결된 카메라 중 해당 유형을 지원하는 카메라가 없으면 토글이 비활성화됩니다. '가장 빠른 캡처'가 켜져 있는 동안에는 모든 토글이 회색으로 표시됩니다.
* **카메라별 행**: 포함 체크박스와 해당 카메라에 적용 가능한 내보내기 유형 목록(▸/▾)이 있으며, 목록 내에는 개별 체크박스가 있습니다. 해당 행에는 "4/6"과 같은 활성화 개수가 표시됩니다.

### 내보내기 유형 및 지원 카메라

여섯 가지 내보내기 유형이 있습니다: **Raw, Debayered, Radiance, Reflectance, Preview, Index**. 각 카메라의 행에는 해당되는 유형만 표시됩니다:

| 내보내기 유형 | 내용 | RGB (FRGB) | 베이어 다중 스펙트럼(FRGN/FOCN/FNGB) | 모노(M3M) |
| --- | --- | --- | --- | --- |
| **Raw** | 센서에서 직접 출력된 베이어 모자이크(모노: 단일 대역) | ✓ | ✓ | ✓ |
| **디베이어 처리** | 선형 디모자이크: 원본 데이터에서 보정 팩의 다크 오프셋(DSNU)을 제거한 후, 에지 인식 디모자이크를 적용 — 선형성 보정, 플랫 필드 또는 언믹스 처리 없음. 3채널 RGB TIFF 파일로 저장(모노: 1채널 그레이스케일) | ✓ | ✓ | ✓ |
| **미리보기** | 전체 디스플레이 체인(카메라 프로필에 따른 화이트 밸런스 + 감마, 그리고 RGB 카메라의 하이라이트 설정; 다중 스펙트럼: 가색 확장) | ✓ | ✓ | ✓ |
| **방사도** | 전체 방사계 측광 체인을 통한 float32 W/m²/sr/nm | — (제공되지 않음) | ✓ | ✓ |
| **반사율** | uint16 ρ (32768 = 1.0) | — (제공되지 않음) | ✓ — 카메라에 DAQ 광 센서(자체 센서 또는 어레이에서 상속된 센서)가 있을 때만 표시됨 | 다중 스펙트럼과 동일 |
| **지수** | 식생 지수(LUT) 렌더링 | — | ✓ — 카메라에 활성화되고 비어 있지 않은 지수 표현식이 있어야 하며, 결합 어레이 구성원에게는 제공되지 않음(어레이가 하나의 공유 지수를 소유함) | — (지수를 사용하려면 ≥2개의 대역이 필요함; [모노 카메라 및 식생 지수](mono-indices.md) 참조) |

RGB 카메라에서는 방사도 및 반사도가 절대 제공되지 않습니다. 광대역 광도 센서의 경우 베이어(Bayer) 단위별 방사도는 의미가 없습니다.

### 가장 빠른 캡처

**⚡ 가장 빠른 캡처 — RAW 전용**토글(켜져 있을 때 주황색)은 모든 내보내기 설정을**RAW 전용**으로 재정의하며(배열의 경우 무료 결합 지수 합성본 포함), 이를 통해 프레임이 가능한 한 빠르게 저장됩니다. 즉, 캡처 시점에 방사도/반사도/표시 계산이 완전히 생략됩니다.

{% hint style="info" %}
**A `.daq` 파일은 여전히 저장됩니다.** 광 센서가 할당된 경우, ‘가장 빠른 캡처’ 기능은 여전히 DAQ 하향 복사량 측정값을 (캡처 파일의 `daq/` 폴더에) 기록합니다. 따라서 방사도, 반사도 및 지수 산출값은 나중에 재처리 과정을 통해 모두 생성할 수 있습니다( [캡처 재처리](#re-processing-captures-into-calibrated-products) 참조). 또한 Fastest Capture는 체크박스 선택 사항을 유지하므로, 기능을 끄면 선택 사항이 복원됩니다.
{% endhint %}

### 어레이별 제어 기능

연결된 각 어레이는 창에 고유한 그룹 카드를 갖습니다:

* **포함** 체크박스(구성원 전체에 걸쳐 3단계 상태)와 어레이 이름 및 표시 모드: "(결합 | 분리)".
* **정렬**체크박스(기본값**켜짐**): 멤버 내보내기 데이터를 어레이의 정렬 프로필에 맞춰 변환하여, 카메라 간에 픽셀 단위로 정렬된 상태로 내보내집니다. RAW 파일은 워프 처리되지 않은 상태로 유지되지만 메타데이터에 변환 정보가 포함됩니다. (프로필 자체는 [어레이 설정 패널](camera-settings.md#alignment-co-registration)에서 계산됩니다.) 어레이 구성원 파일은 각 카메라의 기본 방향으로 저장되며, 미러/회전 설정은 적용되지 않고 파일에 기록됩니다.
* 구성원 카메라 행은 카드 내에 중첩되어 있습니다.

어레이 카드에는 두 개의 레코더도 포함되어 있습니다. 이를 **모니터링 대 분석**으로 생각하시면 됩니다:

| 레코더 | 등급 | 기록 대상 |
| --- | --- | --- |
| **● 인덱스 비디오 기록 / ■ 녹화 중지** *(결합 어레이 전용)* | **모니터링** | 10fps로 비디오로 출력되는 라이브 결합 인덱스 합성 영상 — 8비트, 미리보기 해상도, LUT 내장. 열린 프로젝트와 스트리밍 라이브 뷰가 필요합니다. 녹화 중 프레임 수와 경과 시간을 표시합니다. |
| **⦿ 원본 버스트 녹화 / ■ 원시 버스트 중지** *(모든 어레이)* | **분석**| 실시간 캡처 속도의 원시 베이어 프레임(처리 없음)과 프레임별 매니페스트 및 `.daq` 측정값을 `captures/bursts/` 형식으로 기록합니다. 연사 촬영이 끝나면**비디오 생성** 버튼이 나타납니다. 이 버튼을 누르면 연사 데이터를 오프라인에서 재처리하여 보정된 비디오(통합 지수 및/또는 카메라별 복사도/반사도/지수)와 선택 사항인 TIFF 파일을 생성합니다. 연사 촬영을 중지하면 통합 지수 생성 작업이 자동으로 시작됩니다. |

<!-- SCREENSHOT-NEEDED: an array group card in Capture Settings while a raw burst is recording — the ⦿/■ burst button in its recording state with frame count, and (in a second capture) the Build video button that appears after stopping. -->

## ‘모두 캡처’ 워크플로

<!-- SCREENSHOT-NEEDED: the sidebar during a capture — Capture All showing live "Capturing… 3/6" progress text, and (second capture) the result flash "Saved N files". -->



사이드바의 카메라 목록에서 **‘모두 캡처’**를 누르세요:

1. 포함된 모든 가시적이고 일시 정지되지 않은 카메라가 선택된 내보내기 유형으로 캡처합니다. **어레이는 하나의 동기화된 트리거로 작동합니다**(모든 구성원에 걸쳐 단일 동기화된 그룹 — [다중 카메라 어레이](arrays.md) 참조); 독립형 카메라는 개별적으로 캡처합니다.
2. 숨겨진(눈 아이콘) 또는 일시 정지된 카메라는 건너뜁니다. 어레이는 *모든* 구성원이 숨겨지거나 일시 정지된 경우에만 완전히 차단됩니다.
3. 광 센서가 할당될 때마다, 일치하는 DAQ 하향 복사량 측정값은 캡처 파일의 `daq/` 폴더에 `.daq` 파일로 저장됩니다(원시 데이터 전용 캡처의 경우에도 마찬가지). 이를 통해 나중에 언제든지 방사계 측정 결과를 도출할 수 있습니다.
4. 이 버튼은 실시간 진행 상황("캡처 중… 완료/총")을 표시하며, 연속/간격 모드에서는 **중지 (N)**으로 변경됩니다. 각 캡처 항목에는 300초의 타임아웃이 적용됩니다.
5. 패스가 완료되면 결과 플래시 메시지에 **"N개 파일 저장됨"**또는**"N개 저장됨, F개 실패"**가 표시되며, 카메라가 건너뛴 경우에는 "(S 숨김/일시 중지/건너뜀)"이 추가로 표시됩니다. 드라이브 용량이 가득 차면 실행이 중지되며**"중지됨: 드라이브 용량 부족. N개 저장, F개 실패"**라고 보고됩니다. — 쓰기 실패한 모든 파일은 실패로 집계됩니다.

## 캡처 파일 저장 위치

캡처 파일은 열려 있는 프로젝트의 `<project>/captures/` 경로에 저장됩니다. 각 내보내기 유형은 **별도의 하위 폴더**에 저장되므로, 다단계 캡처 시에도 유형이 혼합되지 않습니다:

```
<project>/captures/
├── raw/           capture_<ts>_SN<serial>_raw.tif
├── debayered/     capture_<ts>_SN<serial>_debayered.tif
├── radiance/      capture_<ts>_SN<serial>_radiance.tif
├── reflectance/   capture_<ts>_SN<serial>_reflectance.tif
├── preview/       capture_<ts>_SN<serial>_display.tif
├── index/         per-camera vegetation-index (LUT) render, when Index is selected
├── composite/     array foreground/background live-view composite, when produced
├── clamp_masks/   radiance/…_clampmask.tif, reflectance/…, index/… — unmix clamp masks,
│                  one subfolder per product folder, only when a band was clamped
├── bursts/        raw-burst recordings (frames + manifest + .daq per burst)
└── daq/           the downwelling reading(s) matched to each capture (.daq), each with a
                   .daq.cameras.json naming the cameras it describes
```

* `<ts>`는 캡처 타임스탬프이며, `<serial>`는 카메라 일련번호입니다. 단독 캡처의 이름은 `capture_<ts>_SN<serial>_<level>`이며, 하나의 동기화된 트리거에서 생성된 어레이 캡처의 이름은 `sync_<ts>_SN<serial>_<level>`이며 **그룹 내 모든 카메라가 하나의 타임스탬프를 공유합니다** (카메라가 단일 레벨만 저장할 경우 레벨 접미사는 생략됩니다).
* **알아두어야 할 한 가지 예외:** 디스플레이 레벨은 `preview/`라는 폴더에 저장되지만, 파일 이름에는 `_display`가 유지됩니다. 이 레벨에 한해 폴더명과 접미사가 다릅니다.
* 알 수 없는 레벨의 경우 해당 레벨 이름과 동일한 폴더에 저장됩니다. 하위 폴더를 생성할 수 없는 경우, 파일이 손실되지 않고 캡처 루트 디렉터리에 기록됩니다.
* 캡처된 TIFF 파일은 기본적으로 무손실 압축(DEFLATE)되며, 전체 보정 및 처리 메타데이터를 **파일의 XMP 내에** 포함합니다 — 캡처 파일은 자체 설명형이며, `.daq` 읽기 파일(및 그에 해당하는 `.daq.cameras.json`) 외에는 별도의 보조 파일이 없습니다.
* **클램프 마스크(1.3.0 버전부터 추가됨).**3×3 스펙트럼 분리 과정에서 적어도 하나의 픽셀에서 밴드를 0으로 클램프해야 했던 경우, 방사도, 반사도 및 지수 산출값에는 픽셀별 마스크가 적용되며, `clamp_masks/<product folder>/<product name>_clampmask.tif`(비트 0 = 결과물의 R 채널에 해당하는 밴드, 비트 1 = G, 비트 2 = B)가 적용되며, 결과물의 `Chloros:UnmixClampMask` XMP 태그가 이를 가리킵니다. 마스크는 제품 폴더**옆**에 위치하며, 절대 폴더 내부에 들어가지 않으므로 `reflectance/`와 같은 제품 폴더는 있는 그대로 사진측량 소프트웨어에 전달할 수 있습니다. Chloros의 자체 가져오기 기능은 마스크를 건너뜁니다.
* **방향 (1.3.0).** 독립형 카메라의 Preview, Radiance, Reflectance 및 Index 산출물은 해당 카메라의 Mirror/Rotation 설정에 따라 회전되며, Raw 및 Debayered 파일은 센서가 읽은 그대로 유지됩니다. 배열 구성 파일은 모두 원본 형식을 유지합니다. 모든 파일은 해당 설정을 XMP에 기록하므로, 재처리 시에도 동일한 방식으로 적용됩니다.
* **`.daq`가 속한 카메라 (1.3.0).** 캡처된 이미지의 `.daq`는 처리 시점에 `.daq.cameras.json`에 명시된 카메라의 이미지와만 매칭됩니다. 따라서 자체 광 센서가 없는 독립형 카메라는 더 이상 다른 카메라의 측정값을 차용하지 않습니다. 센서를 해당 카메라에 바인딩하거나 ‘광 센서’ 탭에 센서를 등록하십시오. 광 센서 기록 파일(및 해당 파일이 없는 모든 `.daq`, 이전 캡처 포함)은 여전히 모든 카메라에 적용됩니다.

이는 `chloros-cli lattice capture` / `array-capture`가 `-o` 디렉터리에 기록하는 레이아웃과 동일하며, [CLI 참조 § 캡처 폴더의 구조](../reference/cli-reference.md#what-a-captures-folder-looks-like)에 문서화되어 있습니다.

<!-- SCREENSHOT-NEEDED: OS file explorer showing a real <project>/captures/ folder after a multi-level array capture — the raw/debayered/radiance/reflectance/preview subfolders, the daq/ and clamp_masks/ folders, and sync_<ts>_SN<serial>_<level>.tif filenames visible inside one subfolder. -->

## 캡처 데이터를 보정된 결과물로 재처리하기

캡처된 원본 프레임과 저장된 `.daq` 파일만 있으면 처리 파이프라인에 필요한 모든 것이 갖춰집니다. 이것이 바로 ‘Fastest Capture’ 모드가 실제 작업에 안전하게 사용될 수 있는 이유입니다.

* **GUI**: 캡처 폴더를 프로젝트에 추가하고([프로젝트에 파일 추가](../processing-images-gui/adding-files-to-a-project.md)), 평소와 같이 처리합니다.
* **CLI**: `process`를 **캡처 루트**로 지정합니다:

```bash
chloros-cli process "C:/ChlorosProjects/MyField/captures"
```

`process`는 일반적으로 지정한 폴더만 가져오지만, 해당 폴더에 이미지가 없고 하위 폴더가 있는 경우 자동으로 하위 폴더로 내려갑니다. 따라서 레벨 하위 폴더와 `daq/` 폴더가 한 번에 모두 가져옵니다. 모든 캡처는 레벨별로 하나의 이미지가 아닌, **단일 이미지**로 가져오며, 다른 레벨들은 보기 가능한 모드로 첨부됩니다.

레벨 하위 폴더의 이름을 직접 지정하는 것(예: `…/captures/raw/`)도 가능하지만, 이 경우 `daq/` 폴더는 제외됩니다. 다시 가져올 때 이 폴더를 함께 복사해 두십시오.- `raw/`에서 방사계량 제품을 도출할 때는 이 폴더를 함께 복사해 두십시오. 그렇지 않으면 타임스탬프 일치 처리가 참조할 대상이 없어집니다.

{% hint style="warning" %}
**처리는 항상 `raw`에서 시작됩니다.** 각 캡처 내에서 원시 프레임이 파이프라인의 소스입니다. `debayered`, `radiance`, `reflectance`, `preview`는 표시 가능한 모드로 제공되지만 파이프라인을 통해 다시 입력되지는 않습니다. 파생된 결과물을 재처리하면 픽셀에 이미 적용된 비네팅, 색상 및 복사도 계산이 다시 적용되므로, Chloros는 이중 처리를 피하기 위해 이를 거부합니다. `index/` 및 `composite/` 렌더링은 전혀 처리되지 않습니다(이들은 캡처가 아닌 출력물입니다). RAW 파일 가져오기 **없이** 저장된 캡처 폴더는 정상적으로 표시되지만, `process`는 이를 건너뛰며 해당 사실을 알립니다. `--input-level {raw,debayered,processed}`는 진입점을 강제하는 의도적인 우회 경로입니다. 정확한 건너뛰기 메시지는 [CLI 참조](../reference/cli-reference.md#what-a-captures-folder-looks-like)를 참조하십시오.
{% endhint %}

재처리 스크립트를 작성할 때 알아두면 좋은 두 가지 추가 동작:

* 제품을 요청했으나 **이미지 제품을 전혀 생성하지 않은**`chloros-cli process` 실행은 명백한 오류로 실패하며 0이 아닌 값으로 종료됩니다** — 아무런 오류 메시지 없이 빈 실행 결과가 나오는 일은 절대 없습니다. 성공적인 실행은 생성된 제품 수를 보고합니다. (의도적으로 메타데이터만 생성하는 실행도 성공으로 간주됩니다.)
* 재수입된 처리된 내보내기 데이터는 캡처의 원시 슬롯을 절대 차지하지 않습니다. 원본 원시 데이터는 항상 파이프라인 소스로 유지됩니다.

## CLI에 해당하는 명령어

이 페이지의 모든 내용은 헤드리스 모드로 실행할 수 있습니다. GUI 캡처 모드는 `chloros-cli lattice array-capture`와 직접 대응됩니다:

| GUI | CLI |
| --- | --- |
| 단일 | `chloros-cli lattice array-capture` |
| 연속 | `array-capture --continuous [--count N] [--duration S]` |
| 간격 | `array-capture --interval S [--duration S]` |
| 최단 시간 캡처 | `array-capture --fastest` |
| 정렬 확인란 | `--aligned / --no-aligned` |
| 내보내기 유형 선택란 | `--processing LEVEL` 또는 `--levels L1,L2,…` (기본값: `all`) |
| 인덱스 비디오 기록 | `chloros-cli lattice array-record` |
| 원시 버스트 기록 / 비디오 생성 | `chloros-cli lattice array-burst` / `array-build-video` |

전체 플래그 테이블, 스마트 AE 안정화 캡처 옵션(`--smart`) 및 지속 속도 모델에 대한 내용은 [CLI 참조 § 캡처 모드, 레코더 및 오프라인 재처리](../reference/cli-reference.md#capture-modes-recorders--offline-reprocess)에서 확인할 수 있습니다.
