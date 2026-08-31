# 단색 카메라 및 식생 지수

## 카메라 1대 = 밴드 1개

**M3M**카메라는 바이어**M3C**의 단색 버전으로, 단일 협대역 간섭 필터 뒤에 단색 IMX265 센서가 장착되어 있습니다. 모델 문자열은 밴드 이름을 나타냅니다 — `M3M-<lens>-F<wavelength>`, 예: `M3M-L87-F685` (Chloros에서는 `LATT-M3M-L87-F685`로 표시됨). 이 센서는 베이어 모자이크가 없는**단일 그레이스케일 대역**을 제공합니다. 즉, 디모자이크할 대상이 없고, 분리해야 할 채널 간 크로스톡이 없으며, 설정해야 할 화이트 밸런스도 없습니다.

모노 시스템을 계획하기 전에 알아두어야 할 사항:

* **방사도와 반사도는 밴드별로 완전히 정의되어 있습니다.**이는 밴드별 방사도 맵이므로, 하나의 M3M 카메라는 M3C 밴드와 정확히 동일하게 보정된 float32 방사도(W/m²/sr/nm)와 uint16 반사도(`32768` = ρ 1.0)를 생성합니다. 모노 프레임은**정체(identity)** 센서 응답 행렬을 포함하므로, 3×3 언믹스(unmix)가 필요하지도 않고 적용되지도 않습니다.
* **단일 모노 카메라로는 식생 지수를 산출할 수 없습니다.** NDVI, NDRE 및 이와 유사한 지수들은 최소 두 개의 밴드가 필요합니다. 모노 하드웨어에서 지수를 계산하려면 여러 대의 M3M 카메라를 결합해야 합니다 — 아래를 참조하십시오.
* M3M 카메라는 **Mono12**(12비트, 전송 시 픽셀당 2바이트)를 스트리밍하며, 이는 [어레이 대역폭 할당](arrays.md#bandwidth-the-rules-of-thumb)에 중요한 요소입니다.

## Chloros가 모노 처리 시 생략하는 항목 — 및 알림 방식

단일 밴드 센서에는 컬러 파이프라인 단계가 적용되지 않습니다. Chloros는 오류를 발생시키기보다는 **한 줄의 메시지와 함께 해당 단계를 건너뜁니다**. 또한 동일한 세션 내의 M3C(베이어) 카메라에 대해서는 해당 단계를 정상적으로 실행합니다:

| 단계 | 모노(M3M) 동작 | M3C 동작 |
| --- | --- | --- |
| 디모자이크 / 디베이어 | 건너뜀 — `debayered` 내보내기 수준은 1채널 그레이스케일 이미지입니다. | 3채널 디모자이크. |
| 화이트 밸런스 (`lattice white-balance`) | 한 줄의 메시지와 함께 건너뜁니다. | 정상적으로 실행됩니다. |
| 색상 프로필 (`lattice color-profile`) | 한 줄의 메시지와 함께 건너뜁니다. | 정상적으로 실행됩니다. |
| 채도/명암비 (`lattice color`) | 한 줄의 메시지와 함께 건너뜀. | 정상적으로 실행됨. |
| 스펙트럼 크로스톡 분리 | 동일 (3×3 행렬 없음). | 카메라별 3×3 행렬 적용. |
| 방사도/반사도 | **실행됨** — 대역별, 완전 보정. | 대역별로 실행됨. |

GUI에서도 동일한 게이트링이 적용됩니다: 모노 카메라의 경우, 카메라별 설정 패널에서 RGB 전용 행(화이트 밸런스, 감마, 색상 프로필, 채도, 명암비, 채널 분할)을 숨기며, 라이브 히스토그램은 단일 **MONO** 트레이스로 고정됩니다. 스택 전체에서 구분 기준은 모델 문자열 내의 `M3M` 토큰이며, 이는 GUI/SDK에 `is_mono`로 표시됩니다.

## 인덱스 생성에는 2개 이상의 밴드가 필요합니다: 정렬 → 스택 → 인덱스

모노 인덱스 워크플로는 항상 다음 세 단계로 이루어집니다:

1. **정렬** — 서로 다른 파장의 M3M 카메라 여러 대(예: F650 &quot;Red&quot;와 F850 &quot;NIR&quot;)를 서로 다른 파장에 맞춰 여러 대의 M3M 카메라를 배치하고, 이를 [다중 카메라 어레이](arrays.md)로 연결한 다음, Chloros가 카메라 간 공동 등록 워프를 계산하도록 합니다.
2. **스택** — 정렬된 프레임들이 하나의 다중 대역 이미지가 됩니다(각 카메라는 하나의 명명된 대역을 제공합니다).
3. **인덱스** — 스택의 대역에 대해 인덱스 공식을 평가하며, 선택적으로 LUT를 통해 렌더링할 수 있습니다.

GUI에서 이 전체 체인은 **Combined Cameras**배열 표시 모드입니다. 라이브 합성 이미지는 이미 정렬되어 있으며, 배열의 인덱스 계산기(아래)가 렌더링할 공식을 정의합니다. 캡처된 내보내기 파일은**Aligned** 캡처 옵션을 사용하여 동일한 정렬 상태로 워프 처리할 수 있습니다.

## 인덱스 계산기

인덱스 계산기는 라이브 뷰 및 카메라별 인덱스 내보내기에서 사용되는 인덱스 표현식을 작성합니다. 이는 하나의 공유 표면으로, ‘카메라’ 탭 사이드바의 두 곳에서 열 수 있습니다:

* **카메라별**— 라이브 미리보기 →**인덱스** 톱니바퀴 (RGN/OCN/NGB 베이어 카메라에만 해당; 단일 모노 카메라에는 인덱스 제어 기능이 없습니다. 하나의 밴드만으로는 인덱스를 생성할 수 없기 때문입니다).
* **어레이별**— 어레이 설정 → 라이브 미리보기 →**인덱스**톱니바퀴 아이콘. 이는 모노 경로입니다: 밴드 목록은**모든 구성 카메라**에 걸쳐 있으므로, 모노 쌍은 여기에 두 개의 밴드를 기여합니다.

<!-- SCREENSHOT-NEEDED: Index Calculator pane opened for a combined array of two mono cameras (e.g. F650 + F850): band chips row showing the two bands with wavelength labels, the operator buttons, the expression textarea containing "(NIR - Red) / (NIR + Red)", the green "Valid expression" banner, the LUT controls (Apply LUT checked, Level 7-stop, Min 0.2 / Max 1), and the live histogram with p2/p98 percentile lines. -->

상단부터 하단까지의 제어 항목은 다음과 같습니다:

* **밴드 칩** (“Bands — 클릭하여 식에 추가”) — 사용 가능한 밴드마다 하나의 버튼이 있으며, 색상 이름과 nm 단위의 파장이 표시됩니다(중복된 색상 이름은 예: “Color 850”과 같이 구별됩니다). 클릭하면 커서 위치에 밴드 토큰이 삽입됩니다. 밴드별 복사도를 생성할 수 없는 카메라(RGB/FRGB)의 밴드는 필터링되어 제외됩니다.
* **연산자 및 함수 버튼** — `+ - * / ( ) ^ ,`와 `abs() sqrt() log() log10() exp() min() max() pow()`.
* **식 입력 영역** — 자유롭게 입력할 수 있는 수식; 자리 표시자에는 고전적인 NDVI 형식인 `(NIR - Red) / (NIR + Red)`가 표시됩니다. 그 위에 있는 읽기 전용 토큰화 미리보기에서는 밴드 칩, 숫자 및 플래그가 알 수 없는 토큰으로 표시됩니다.
* **유효성 배너**— 회색 &quot;비어 있음 — 인덱스가 적용되지 않음&quot;; 녹색 “유효한 표현식”; 빨간색으로 표시되며 구체적인 구문 분석 오류(알 수 없는 밴드, 여러 카메라에 의해 노출된 모호한 밴드, 누락된 괄호 등)가 표시됩니다; 또는 표현식이 유효하지만**상수**인 경우(예: `X/X`, 또는 분모가 `+` 대신 `−`로 입력된 NDVI) — 상수는 전체 프레임을 하나의 색상으로 매핑합니다.
* 적용된 식은 문제없지만 **실시간 프레임이 균일한**(평탄하거나 채도가 높은 장면) 경우 별도의 호박색 경고가 표시됩니다. 히스토그램 붕괴가 자동으로 감지됩니다.
* **LUT 적용**(기본값: 켜짐; 끔 = 그레이스케일 스트레치),**레벨**2/3/5/7 스톱(기본값: 7 스톱), 그리고 그라데이션 바 양쪽에 위치한**최소/최대**입력값. Min의 기본값은**

0.2**입니다. 이 값은 색상 램프를 식생 관련 범위로 확대하며, 이보다 낮은 값은 그레이스케일로 처리됩니다. 전체 인덱스 범위를 사용하려면 Min을 −1로 설정하십시오(**Reset** 버튼을 누르면 −1…+1로 복원됩니다). Max의 기본값은 1입니다.
* 지수 분포의 **실시간 히스토그램** — 제곱근 스케일링된 막대, 호박색 p2/p98 백분위수 선, 흰색 중앙값 선, 그리고 범위 외 꼬리 부분 표시(&quot;◀ N% &lt; lo&quot; / &quot;hi &lt; N% ▶&quot;)가 표시되며, 1%를 초과하면 호박색으로 변해 Min/Max 범위를 넓혀야 함을 알립니다.
* **적용**을 누르면 표현식이 실시간 스트림에 반영됩니다. LUT 조정은 ‘적용’을 누르지 않아도 실시간으로 적용됩니다. 표현식은 의도적으로**세션 전용**으로 설계되어 세션 간에 유지되지 않습니다.

<!-- SCREENSHOT-NEEDED: Combined-array live tile rendering NDVI from a mono pair through the default 7-stop LUT, with the array name pill and fps readout visible — the result of applying the expression from the previous screenshot. -->

## CLI 경로

동일한 정렬 → 스택 → 인덱스 체인으로, 끝에서 끝까지 스크립트로 제어 가능합니다:

```bash
chloros-cli lattice array-connect --serials SN_RED,SN_NIR
chloros-cli lattice index --live --profile align.json \
  --preset NDVI --channel red=Red_660 --channel nir=NIR_850 \
  --save-multiband -o output/
```

`--channel`는 프리셋의 심볼을 스택의 밴드 이름에 매핑합니다. 다음 두 가지 규칙을 따르면 실행 실패를 방지할 수 있습니다:

* **심볼은 대소문자를 구분합니다**. 프리셋의 채널 이름과 정확히 일치해야 합니다. 프리셋은 소문자를 사용합니다(NDVI의 경우 `red`,`nir`; `--list-presets`를 확인해 보세요). `--channel red=Red_660`는 정상 작동하지만, `--channel RED=660`는 ‘`channel_map missing entries`’ 오류로 실패합니다.
* 밴드 측에서는 정렬된 스택 내의 밴드 이름을 지정해야 합니다(`lattice align-info --profile align.json`에 목록이 나열되어 있습니다). 오프라인 모드에서는 0을 기점으로 하는 밴드 인덱스(예: `--channel red=0 --channel nir=1`)도 허용됩니다.

`lattice index`는 저장된 정렬된 다중 밴드 TIFF에 대해서도 완전히 오프라인 상태에서 실행됩니다:

```bash
chloros-cli lattice index --input aligned.tif --preset NDVI \
  --output ndvi.tif --colorize --gradient RdYlGn
```

### 인덱스 사전 설정

`lattice index --preset`(및 동일한 엔진을 사용하는 ‘이미지’ 탭의 [인덱스/LUT 샌드박스](../image-viewer-gui/index-lut-sandbox.md))에는 다음 **22가지 사전 설정**이 포함되어 있습니다:

`NDVI, GNDVI, BNDVI, NDRE, ENDVI, SAVI, OSAVI, MSAVI, EVI, EVI2, CVI, MSR, TDVI, LAI, GLI, NGRDI, VARI, TGI, EXG, CIRE, CIGREEN, NDWI`

각 프리셋의 공식 및 채널 기호를 확인하려면 `chloros-cli lattice index --list-presets`를, 사용 가능한 색상 그라데이션을 확인하려면 `--list-gradients`를 실행하십시오. 사용자 정의 공식을 사용하려면 인덱스 계산기와 동일한 구문을 사용하는 `--formula EXPR`를 활용하십시오. 이 프리셋 목록은 LATTICE 지수 엔진에 한정된 것임을 유의하십시오. 가져온 이미지의 ‘프로젝트 설정’ 내 ‘처리’ 드롭다운 메뉴에는 다른 목록이 표시됩니다([다중 스펙트럼 지수 공식](../project-settings/multispectral-index-formulas.md) 참조).

전체 플래그 세트(`--output-format`, `--vmin/--vmax/--percentile`, `--bg-mode`, `--live`의 정렬 워프 노브, 기타 등)에 대한 설명은 [CLI 참조 § 지수 / 식생 수학](../reference/cli-reference.md#index--vegetation-maths)에 수록되어 있으며, SDK에 해당하는 기능은 [SDK 참조](../reference/sdk-reference.md)에 설명되어 있습니다.

## 모노 배열에서 인덱스 제품 캡처하기

배열이 연결되고 인덱스 표현식이 적용된 상태에서, `array-capture`(또는 GUI의 **모두 캡처**)를 사용하면 카메라별 내보내기 레벨 *및* 인덱스 렌더링 결과를 저장합니다 — `--index`/`--no-index`는 CLI에서 이 기능을 토글하며, 캡처 시 기본적으로 적용 가능한 모든 레벨을 포함하도록 설정됩니다. 각 캡처 그룹에 대한 모노 카메라의 기여도는 원시/디베이어링(그레이스케일)/방사도/반사도 레벨의 단일 밴드와, 어레이가 결합 모드로 실행될 때 공유되는 결합 인덱스 합성 이미지입니다. [다중 카메라 어레이 § 캡처](arrays.md#capturing-monitoring-vs-analysis)를 참조하십시오.
