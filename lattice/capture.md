# 캡처 설정 및 모드

‘카메라’ 탭에서의 캡처는 하나의 빨간색 **모두 캡처**버튼과, 해당 버튼이 어떤 결과를 생성할지 결정하는**캡처 설정** 패널을 통해 제어됩니다. 여기에는 어떤 카메라가 참여할지, 각 카메라가 어떤 내보내기 형식으로 저장할지, 셔터가 한 번만 작동할지, 연속적으로 작동할지, 아니면 일정한 간격으로 작동할지가 포함됩니다. 이 페이지에서는 구성, 캡처 과정, 파일이 디스크에 저장되는 위치, 그리고 나중에 보정된 결과물로 재처리하는 방법 등 전체 흐름을 설명합니다. 카메라 및 어레이 제어 기능은 [카메라 설정](camera-settings.md)에서 확인할 수 있습니다.

{% hint style="info" %}
**캡처를 수행하려면 프로젝트가 열려 있어야 합니다.** 프로젝트가 열려 있을 때까지 [모두 캡처] 및 [캡처 설정] 톱니바퀴 아이콘은 비활성화됩니다(&quot;캡처 결과를 저장하려면 프로젝트를 생성하거나 여십시오&quot;). 모든 캡처 결과는 `captures/` 내의 프로젝트 폴더에 저장됩니다.
{% endhint %}

## 캡처 설정 창

사이드바의 카메라 목록에서 **&#x27;모두 캡처&#x27; 옆의 톱니바퀴 아이콘**을 클릭하거나, 각 카메라별 설정 창 하단의**&quot;캡처 설정 열기…&quot;** 버튼을 클릭하여 이 창을 엽니다. 헤더에는 &quot;캡처 설정&quot;이라는 문구와 ← 뒤로 가기 버튼이 표시됩니다.

<!-- SCREENSHOT-NEEDED: the full Capture Settings pane — Single/Continuous/Interval mode buttons at top, the bulk export-type toggle rows (All Raw … All Index), the orange Fastest Capture toggle, an array group card with the Aligned checkbox and Record buttons, and an expanded per-camera row showing per-type checkboxes. -->

여기에서 선택한 내용(포함된 카메라, 유형별 확인란, 캡처 모드)은 **프로젝트별로** 저장되며, 프로젝트를 다시 열 때 복원됩니다.

### 캡처 모드

패널 상단에 있는 세 가지 모드 버튼:

| 모드 | 기능 | 하위 설정 (기본값) |
| --- | --- | --- |
| **단일** *(기본값)* | 선택한 모든 카메라에 대해 한 번의 캡처를 수행합니다. | — |
| **연속**| 중지 조건이 충족될 때까지 연속으로 캡처합니다. |**캡처 횟수**(기본값 1) *또는* **캡처 지속 시간**(기본값 10초; 단위: 초 / 분 / 시간 / 일)에 따라 중지됩니다. |
| **간격**(타임랩스) | 타이머에 따라 연속 촬영합니다. |**간격당 촬영 횟수**(기본값 1) ·**매**N 단위 (기본값 5초) ·**N 단위 동안** (기본값 1분). |

연속 또는 간격 모드에서, 촬영 중일 때 ‘모두 촬영’ 버튼은 **중지 (N)** 버튼으로 바뀌며, 촬영 횟수가 누적되는 대로 카운트됩니다.

<!-- SCREENSHOT-NEEDED: the capture-mode area of Capture Settings with Interval selected — showing the "Captures / interval", "Every N (unit)" and "For N (unit)" rows with their defaults (1, 5 s, 1 m). -->

### 카메라 및 내보내기 유형 선택

이 패널의 도움말 텍스트에 요약되어 있습니다: “모두 캡처”가 생성할 카메라와 내보내기 유형을 선택하세요 — 기본적으로 모든 항목이 활성화되어 있으며, 선택 사항은 이 프로젝트와 함께 저장됩니다.

* **모두 선택 / 모두 해제** 버튼을 클릭하면 모든 카메라의 포함 확인란이 한 번에 전환됩니다.
* **대량 내보내기 유형 토글**(두 줄의 버튼):**모든 원본 / 모든 디베이어 처리됨 / 모든 미리보기 / 모든 방사도 / 모든 반사도 / 모든 인덱스**. 각 토글은 세 가지 상태로 색상이 표시됩니다: 녹색 ✓ = 해당 형식을 지원하는 모든 카메라에서 활성화됨, 황색 – = 일부 카메라에서 활성화됨, 회색 = 지원되는 카메라 없음. 연결된 카메라 중 해당 형식을 지원하는 카메라가 없으면 토글이 비활성화됩니다. ‘가장 빠른 캡처’가 켜져 있는 동안에는 모든 토글이 회색으로 표시됩니다.
* **카메라별 행**: 포함 확인란과, 해당 카메라에 적용 가능한 내보내기 유형이 개별 확인란과 함께 표시된 확장 가능한(▸/▾) 목록으로 구성됩니다. 행에는 &quot;4/6&quot;과 같은 활성화 개수가 표시됩니다.

### 내보내기 유형 및 지원 카메라

여섯 가지 내보내기 유형이 있습니다: **Raw, Debayered, Radiance, Reflectance, Preview, Index**. 각 카메라 행에는 해당 카메라에 적용 가능한 유형만 표시됩니다:

| 내보내기 유형 | 내용 | RGB (FRGB) | 베이어 다중 스펙트럼 (FRGN/FOCN/FNGB) | 모노 (M3M) |
| --- | --- | --- | --- | --- |
| **Raw** | 센서에서 직접 출력된 베이어 모자이크 (모노: 단일 대역) | ✓ | ✓ | ✓ |
| **데베이어** | 선형 디모자이크 (모노: 1채널 그레이스케일) | ✓ | ✓ | ✓ |
| **미리보기** | 전체 처리 체인 (카메라 프로필에 따른 화이트 밸런스 + 감마; 다중 스펙트럼: 가색 확장) | ✓ | ✓ | ✓ |
| **방사도** | 전체 방사계 측광 체인을 통한 float32 W/m²/sr/nm | — (제공되지 않음) | ✓ | ✓ |
| **반사율** | uint16 ρ (32768 = 1.0) | — (지원되지 않음) | ✓ — 카메라에 DAQ 광 센서(자체 센서 또는 어레이에서 상속된 센서)가 있을 때만 표시됨 | 다중 스펙트럼과 동일 |
| **지수** | 식생 지수(LUT) 렌더링 | — | ✓ — 카메라에 활성화되고 비어 있지 않은 지수 표현식이 있어야 하며, 결합 어레이 구성원에게는 제공되지 않음(어레이가 하나의 공유 지수를 소유함) | — (지수를 사용하려면 2개 이상의 대역이 필요함; [모노 카메라 및 식생 지수](mono-indices.md) 참조) |

RGB 카메라에서는 방사도 및 반사도가 절대 제공되지 않습니다. 광대역 광도 센서의 경우 베이어(Bayer) 단위별 방사도는 의미가 없습니다.

### 가장 빠른 캡처

**⚡ 가장 빠른 캡처 — RAW 전용**토글(켜져 있을 때 주황색)은 모든 내보내기 설정을**RAW 전용**으로 재정의하며, 어레이의 경우 무료 결합 지수 합성 이미지를 추가로 제공하므로 프레임이 가능한 한 빠르게 저장됩니다: 캡처 시점에 복사도/반사도/표시 계산이 완전히 생략됩니다.

{% hint style="info" %}
**`.daq` 파일은 여전히 저장됩니다.** 광 센서가 할당된 경우, ‘가장 빠른 캡처’는 원시 프레임 옆에 DAQ 하향 복사량 측정값을 여전히 기록합니다. 따라서 나중에 재처리 과정을 통해 복사도, 반사도 및 지수 산출값을 모두 생성할 수 있습니다([캡처 재처리](#re-processing-captures-into-calibrated-products) 참조). 또한 Fastest Capture는 체크박스 선택 사항에 영향을 주지 않습니다. 이 기능을 끄면 선택 사항이 복원됩니다.
{% endhint %}

### 어레이별 제어 기능

연결된 각 어레이는 창 내에서 자체 그룹 카드를 갖습니다:

* **포함** 확인란(구성원 전체에 대해 3가지 상태)과 어레이 이름 및 표시 모드: &quot;(결합 | 분리)&quot;.
* **정렬**체크박스(기본값**켜짐**): 멤버 내보내기 데이터를 어레이의 정렬 프로필에 맞춰 변환하여, 카메라 간에 픽셀 단위로 정렬된 상태로 내보내집니다. RAW 데이터는 워프 처리되지 않은 상태로 유지되지만 메타데이터에 변환 정보가 포함됩니다. (프로필 자체는 [어레이 설정 패널](camera-settings.md#alignment-co-registration-combined-only)에서 계산됩니다.)
* 구성원 카메라 행은 카드 내에 중첩되어 표시됩니다.

어레이 카드에는 두 개의 레코더도 포함되어 있습니다. 이를 **모니터링 대 분석**으로 생각하시면 됩니다:

| 레코더 | 등급 | 기록 내용 |
| --- | --- | --- |
| **● 인덱스 비디오 기록 / ■ 녹화 중지** *(결합 어레이 전용)* | **모니터링** | 10fps의 라이브 결합 인덱스 합성 영상을 비디오로 기록 — 8비트, 미리보기 해상도, LUT 내장. 열린 프로젝트와 스트리밍 라이브 뷰가 필요합니다. 녹화 중 프레임 수 및 경과 시간을 표시합니다. |
| **⦿ 원본 버스트 녹화 / ■ 원시 버스트 중지** *(모든 어레이)* | **분석**| 실시간 캡처 속도의 원시 베이어 프레임(처리 없음)과 프레임별 매니페스트, `.daq` 측정값을 `captures/bursts/` 형식으로 기록합니다. 연사 촬영이 끝나면**동영상 생성** 버튼이 나타납니다. 이 버튼을 누르면 연사 데이터를 오프라인에서 재처리하여 보정된 동영상(통합 지수 및/또는 카메라별 광도/반사율/지수)과 선택 사항인 TIFF 파일을 생성합니다. 통합 지수 동영상 생성은 연사 촬영을 중지하면 자동으로 시작됩니다.

<!-- SCREENSHOT-NEEDED: an array group card in Capture Settings while a raw burst is recording — the ⦿/■ burst button in its recording state with frame count, and (in a second capture) the Build video button that appears after stopping. -->

|## &#x27;모두 캡처&#x27; 워크플로

<!-- SCREENSHOT-NEEDED: the sidebar during a capture — Capture All showing live "Capturing… 3/6" progress text, and (second capture) the result flash "Saved N files". -->

사이드바의 카메라 목록에서 **&#x27;모두 캡처&#x27;**를 누르세요:

1. 포함된 모든 가시적이고 일시 정지되지 않은 카메라가 선택된 내보내기 유형으로 캡처합니다. **어레이는 하나의 동기화된 트리거로 작동합니다**(모든 구성원에 걸쳐 단일 동기화된 그룹 — [다중 카메라 어레이](arrays.md) 참조); 독립형 카메라는 개별적으로 촬영합니다.
2. 숨겨진(눈 아이콘) 또는 일시 정지된 카메라는 건너뜁니다. 어레이는 *모든* 구성원이 숨겨지거나 일시 정지된 경우에만 완전히 차단됩니다.
3. 광 센서가 할당될 때마다, 해당 DAQ 하향 복사량 측정값이 이미지와 함께 `.daq` 파일로 저장됩니다(원시 데이터만 캡처하는 경우에도 마찬가지). 이를 통해 나중에 언제든지 방사측정 제품을 도출할 수 있습니다.
4. 버튼에는 실시간 진행 상황 — &quot;캡처 중… 완료/총&quot; — 이 표시되며, 연속/간격 모드에서는 **중지 (N)**으로 변경됩니다. 각 캡처 항목에는 300초의 타임아웃이 적용됩니다.
5. 패스가 완료되면 결과 플래시에 **&quot;N개 파일 저장됨&quot;**또는**&quot;N개 저장됨, F개 실패&quot;**가 표시되며, 카메라가 건너뛴 경우에는 &quot;(S 숨김/일시 중지/건너뜀)&quot;이 추가로 표시됩니다.

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
├── bursts/        raw-burst recordings (frames + manifest + .daq per burst)
└── *.daq          the downwelling reading matched to the capture
```

* `<ts>`는 캡처 타임스탬프이며, `<serial>`는 카메라 일련번호입니다. 단독 캡처의 이름은 `capture_<ts>_SN<serial>_<level>`이며, 하나의 동기화된 트리거에서 생성된 어레이 캡처의 이름은 `sync_<ts>_SN<serial>_<level>`이며 **그룹 내 모든 카메라가 하나의 타임스탬프를 공유합니다** (카메라가 단일 레벨만 저장할 경우 레벨 접미사는 생략됩니다).
* **알아두어야 할 한 가지 예외:** 디스플레이 레벨은 `preview/`라는 폴더에 저장되지만, 파일 이름에는 `_display`가 유지됩니다. 이 레벨에 대해서만 폴더와 접미사가 다릅니다.
* 알 수 없는 레벨의 경우, 해당 레벨 이름과 동일한 폴더에 저장됩니다. 하위 폴더를 생성할 수 없는 경우, 파일이 손실되지 않고 캡처 루트 디렉터리에 기록됩니다.
* 캡처 TIFF 파일은 기본적으로 무손실 압축(DEFLATE)되며, 전체 보정 및 처리 메타데이터를 **파일의 XMP 내부**에 포함합니다. 즉, 캡처 파일은 자체 설명형이며, `.daq` 읽기 파일 외에는 별도의 사이드카 파일이 없습니다.

이는 `chloros-cli lattice capture` / `array-capture`가 `-o` 디렉터리에 기록하는 레이아웃과 동일하며, 이는 [CLI 참조 § 캡처 폴더의 구조](../reference/cli-reference.md#what-a-captures-folder-looks-like)에 설명되어 있습니다.

<!-- SCREENSHOT-NEEDED: OS file explorer showing a real <project>/captures/ folder after a multi-level array capture — the raw/debayered/radiance/reflectance/preview subfolders, a .daq file at the root, and sync_<ts>_SN<serial>_<level>.tif filenames visible inside one subfolder. -->

## 캡처 데이터를 보정된 결과물로 재처리하기

캡처된 원시 프레임과 저장된 `.daq` 파일만 있으면 처리 파이프라인에 필요한 모든 것이 갖춰집니다. 이것이 바로 ‘Fastest Capture’가 실제 작업에 안전하게 사용될 수 있는 이유입니다.

* **GUI**: 캡처 폴더를 프로젝트에 추가하고 ([프로젝트에 파일 추가](../processing-images-gui/adding-files-to-a-project.md)) 평소와 같이 처리합니다.
* **CLI**: `process`를**캡처 루트**로 지정합니다:

```bash
chloros-cli process "C:/ChlorosProjects/MyField/captures"
```

`process`는 일반적으로 지정한 폴더만 가져오지만, 해당 폴더에 이미지가 없고 하위 폴더가 있는 경우 자동으로 하위 폴더로 내려가므로 — 레벨 하위 폴더와 루트 `.daq` 파일이 한 번에 모두 가져옵니다. 모든 캡처는 레벨당 하나의 이미지가 아닌, **단일 이미지**로 가져오며, 다른 레벨들은 보기 가능한 모드로 첨부됩니다.

레벨 하위 폴더를 직접 지정하는 방법(예: `…/captures/raw/`)도 작동하지만, 하지만 루트 `.daq` 파일들은 제외됩니다. `raw/`에서 방사계 측정 제품을 재파생할 때 해당 파일들을 함께 복사해 두어야 합니다. 그렇지 않으면 타임스탬프 일치 대상을 찾을 수 없습니다.

{% hint style="warning" %}
**처리는 항상 `raw`에서 시작됩니다.**각 캡처 내에서 원시 프레임이 파이프라인의 소스입니다; `debayered`, `radiance`, `reflectance` 및 `preview`는 보기 가능한 모드로 제공되지만 파이프라인을 통해 다시 처리되지는 않습니다. — 파생된 결과물을 재처리하면 픽셀에 이미 적용된 비네팅, 색상 및 복사도 계산이 다시 적용되므로, Chloros는 이중 처리를 피하기 위해 이를 거부합니다. `index/` 및 `composite/` 렌더링은 전혀 처리되지 않습니다(이들은 캡처가 아닌 출력물입니다). RAW 파일 가져오기**없이** 저장된 캡처 폴더는 정상적으로 표시되지만, `process`는 이를 건너뛰며 해당 사실을 알립니다. `--input-level {raw,debayered,processed}`는 진입점을 강제하는 의도적인 비상구입니다. 정확한 건너뛰기 메시지는 [CLI 참조](../reference/cli-reference.md#what-a-captures-folder-looks-like)를 참조하십시오.
{% endhint %}

재처리 스크립트를 작성할 때 알아두면 좋은 두 가지 추가 동작:

* 산출물을 요청했으나 **이미지 산출물을 전혀 작성하지 않은**`chloros-cli process` 실행은 명백한 오류로 실패하며 0이 아닌 값으로 종료됩니다** — 아무런 오류 메시지 없이 빈 실행 결과가 나오는 경우는 절대 없습니다. 성공적인 실행은 산출물 개수를 보고합니다. (의도적으로 메타데이터만 처리하는 실행도 성공으로 간주됩니다.)
* 재가공된 내보내기 데이터를 다시 가져와도 캡처의 원본 슬롯을 차지하지 않습니다. 원본 데이터는 항상 파이프라인의 소스로 유지됩니다.

## CLI에 상응하는 명령

이 페이지의 모든 내용은 헤드리스 모드로 실행할 수 있습니다. GUI 캡처 모드는 `chloros-cli lattice array-capture`와 직접 매핑됩니다:

| GUI | CLI |
| --- | --- |
| 단일 | `chloros-cli lattice array-capture` |
| 연속 | `array-capture --continuous [--count N] [--duration S]` |
| 간격 | `array-capture --interval S [--duration S]` |
| 최고 속도 캡처 | `array-capture --fastest` |
| 정렬 확인란 | `--aligned / --no-aligned` |
| 내보내기 유형 선택란 | `--processing LEVEL` 또는 `--levels L1,L2,…` (기본값: `all`) |
| 인덱스 동영상 기록 | `chloros-cli lattice array-record` |
| 원본 버스트 기록 / 동영상 생성 | `chloros-cli lattice array-burst` / `array-build-video` |

전체 플래그 테이블, 스마트 AE 안정화 캡처 옵션(`--smart`) 및 지속 속도 모델은 [CLI 참조 § 캡처 모드, 레코더 및 오프라인 재처리](../reference/cli-reference.md#capture-modes-recorders--offline-reprocess)에 설명되어 있습니다.
