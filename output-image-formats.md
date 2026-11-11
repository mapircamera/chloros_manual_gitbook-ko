---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/output-image-formats
---

# 출력 이미지 형식

Chloros는 처리된 결과물을 네 가지 파일 형식으로 내보냅니다. 프로젝트 설정(GUI)에서, `--format`(CLI)를 사용하거나, `export_format`(SDK)를 사용하여 형식을 선택하십시오. CLI 및 SDK는 아래의 정확한 문자열을 지원합니다.

| 형식 문자열 | 확장자 | 픽셀 유형 | 픽셀 범위 | 비고 |
| --- | --- | --- | --- | --- |
| `TIFF (16-bit)` *(기본값)* | `.tif` | uint16 디지털 숫자 | 0 – 65535 | 사진측량/GIS에 권장됩니다. |
| `TIFF (8-bit)` | `.tif` | uint8 정수 | 0 – 255 | 8비트 형식만 지원하는 모든 8비트 캡처용: Chloros는 비트 심도를 업스케일링하지 않습니다. |
| `TIFF (32-bit, Percent)` | `.tif` | float32 | 0.0 – 1.0 | 1.0 = 100% 반사율. 일부 애플리케이션은 부동소수점 TIFF 파일을 읽을 수 없으며, 파일 크기가 더 큽니다. |
| `PNG (8-bit)` | `.png` | uint8 디지털 숫자 | 0 – 255 | 무손실 압축으로, 웹 보기 및 시각화에 적합합니다. |
| `JPG (8-bit)` | `.jpg` | uint8 디지털 숫자 | 0 – 255 | 손실 압축, 파일 크기가 가장 작음. |

## 출력 파일 저장 위치

출력 파일은 프로젝트 폴더 아래에 카메라별로, 그다음 파일 형식별로 그룹화되어 저장됩니다:

```
<project>/
└── LATT-M3M-L41-F550/                  # one folder per camera (model+lens+filter)
    ├── tiff16/                          # follows --format: tiff16, tiff8, png8, jpg8, or tiff32
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── NDVI_Index_Images/           # one <INDEX>_Index_Images/ folder per requested index
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

카메라 폴더는 LATTICE의 경우 `LATT-<sensor>-<lens>-F<filter>`이고, Survey3의 경우 `<model>_<filter>`(예: `Survey3N_RGN`)입니다. **내보낸 모든 산출물은 원본 파일의 이름을 그대로 유지합니다. 즉, 산출물을 식별하는 것은 파일 이름의 확장자가 아니라 폴더입니다.** 전체 규칙에 대해서는 CLI 참조 문서의 [산출물 저장 위치](reference/cli-reference.md)를 참조하십시오.

## LATTICE 산출물 (캡처 및 내보내기 수준)

하나의 LATTICE 원시 프레임은 단일 처리 단계에서 요청된 모든 산출물로 분할됩니다. 각 산출물 유형에는 고유한 토글(GUI 확인란 또는 CLI의 `--debayered` / `--preview` / `--radiance` / `--reflectance`, 모두 기본적으로 ON 상태)가 있습니다:

| 레벨 | 내용 | 데이터 유형 |
| --- | --- | --- |
| `raw` | 센서에서 직접 추출한 베이어 데이터(모노 카메라: 단일 대역). 처리는 항상 원시 데이터(RAW)에서 시작됩니다. | 캡처된 그대로 |
| `debayered` | 선형 디모자이크 — M3C의 경우 3채널, M3M의 경우 1채널 그레이스케일: 보정 팩의 다크 오프셋이 제거된 후, 선형성 보정, 플랫 필드 또는 언믹스 없이 에지 인식 디모자이크가 적용됩니다. `process`로 내보내면 모든 처리 모드에서 이 단계에서 캡처가 저장하는 것과 동일한 파일이 생성됩니다. | 선형 DN |
| `radiance` | 전체 방사계측 체인에서 도출된 절대 스펙트럼 복사도, **W/m²/sr/nm** 단위. 선택한 내보내기 형식과 관계없이 항상 32비트 TIFF(`tiff32/Radiance_Images/`)로 기록됩니다. | float32 |
| `reflectance` | 반사율 ρ. 여기서 **DN 32768 = ρ 1.0 (100%)**이며, ρ 2.0까지의 헤드룸이 있습니다. Pix4D 호환. | uint16 |
| `preview` | 디스플레이용 렌더링: RGB = 화이트 밸런스 + 감마; 다중 스펙트럼 = 가색 확장. | 8비트 디스플레이 |

### 방향

`raw` 및 `debayered` 파일은 항상 센서의 **원본** 방향을 유지합니다. 카메라의 뒤집기/회전 설정은 다른 제품(미리보기, 복사도, 반사도, 지수)에만 적용됩니다. 어레이 캡처의 경우, 방향 설정을 명시적으로 요청하지 않는 한(CLI에서는 `--orient`, SDK에서는 `orient=True`) 모든 구성 요소의 파일은 원본 방향을 유지합니다. 각 파일은 XMP에 해당 픽셀의 방향을 기록합니다 — `ChlorosRotation`, `ChlorosFlipHorizontal`, `ChlorosFlipVertical` — 이며, `ChlorosOrientationApplied`는 해당 픽셀이 이 정보를 포함하는지 여부를 나타냅니다(RAW 및 디베이어링된 파일의 경우 `false`). Chloros 이미지 뷰어는 표시 목적으로만 해당 레코드를 기반으로 RAW 및 디베이어링된 파일을 변환하며, 파일 자체는 절대 재기록되지 않습니다. 레코드가 없는 파일(1.3.0 이전에 작성되었거나 다른 도구로 작성된 파일)은 저장된 그대로 표시됩니다.

## 클램프 마스크

다중 스펙트럼 M3C 카메라의 방사도 및 반사도 산출값은 3×3 밴드 언믹스 처리를 거칩니다. 언믹스 과정에서 특정 밴드에서 음수 값이 생성될 경우, 해당 값은 0으로 클리핑됩니다(라이브 뷰에서도 정확히 동일한 방식으로 클리핑됩니다). 1.3.0 버전에서 새롭게 추가된 기능으로, 적어도 하나의 픽셀이 클리핑된 결과물에는 정확히 어떤 픽셀과 밴드가 클리핑되었는지를 나타내는 **클램프 마스크**가 함께 제공됩니다:

* **위치:**`clamp_masks/` 폴더 내의 산출물 폴더**옆**에 있으며, 절대 폴더 내부에는 위치하지 않습니다: `tiff16/Reflectance_Calibrated_Images/x.tif` → `tiff16/clamp_masks/Reflectance_Calibrated_Images/x_clampmask.tif`, 그리고 `tiff32/Radiance_Images/x.tif` → `tiff32/clamp_masks/Radiance_Images/x_clampmask.tif`.
* **내용:** 제품 고유의 픽셀 그리드(방향, ROI, 비닝 및 배열 정렬 포함)를 기반으로 한 단일 밴드 `uint8` TIFF 파일. 제품의 R 채널에서 밴드가 클리핑된 경우 비트 0(`mask & 1`)이 설정되며, 비트 1 (`& 2`)는 G 채널, 비트 2(`& 4`)는 B 채널에서 클리핑된 경우 설정됩니다. 클리핑된 픽셀의 대역별 비율도 `Chloros:BandRUnmixClampFraction`, `Chloros:BandGUnmixClampFraction` 및 `Chloros:BandBUnmixClampFraction`로 제품에 기록됩니다.
* **제품에서 이를 표시하는 방식:** XMP 태그 `Chloros:UnmixClampMask`에는 제품 폴더를 기준으로 한 마스크의 경로(`../clamp_masks/<product folder>/<name>_clampmask.tif`, 항상 `/`로 구분됨), 픽셀이 잘리지 않은 경우(마스크가 기록되지 않은 경우)에는 `none`, 마스크를 기록할 수 없었던 경우에는 `unavailable`가 표시됩니다. 태그가 없는 제품은 언믹스 처리를 수행한 적이 없는 것입니다(RGB 및 모노 카메라, 원본 및 디베이어 처리된 레벨).

마스크가 포함된 제품 폴더가 없으므로, `Reflectance_Calibrated_Images/`와 같은 제품 폴더를 하위 폴더를 포함하여 있는 그대로 Pix4D, Metashape 또는 기타 사진측량 소프트웨어에 전달할 수 있습니다. 즉, 그 위의 포맷 폴더가 아닌 제품 폴더 자체를 전달해야 합니다. Chloros의 자체 가져오기 기능은 클램프 마스크를 건너뜁니다. CLI는 이를 별도로 계산합니다: `Image products written: N (plus M clamp masks)`.

## 인덱스 제품

각 인덱스는 별도의 `<INDEX>_Index_Images/` 폴더(예: `NDVI_Index_Images/`)에 저장됩니다.

* **정수 형식**(`TIFF (16-bit)`, `TIFF (8-bit)`, `PNG (8-bit)`, `JPG (8-bit)`)는 −1…1 매핑이 아닌**이미지별 선형 스트레치**를 저장합니다. DN 0은 해당 프레임의 최저 인덱스 값이며, 풀 스케일은 최고값입니다. 이 파일에는 `Chloros:IndexScaling=linear_minmax`, `Chloros:IndexMin` 및 `Chloros:IndexMax`가 포함되어 있으므로, `value = IndexMin + DN / full_scale × (IndexMax − IndexMin)`도 포함됩니다. 각 프레임마다 고유한 끝점 범위를 가지므로, 프레임을 DN이 아닌 값으로 비교해야 합니다.
* **`TIFF (32-bit, Percent)`**는 재스케일링 없이 인덱스 값 자체를 float32로 저장하며, 이는 비행 전반에 걸친 정량적 분석에 가장 적합한 선택입니다.

1.3.0 버전에서 새로 추가된 기능으로, 인덱스 산출물은 원본 원시 데이터의 태그를 반복하지 않고 **자신**을 설명합니다. GPS 및 카메라 태그는 계산의 기초가 된 산출물(반사율 산출물이 있는 경우 해당 산출물)에서 복사된 후, 다음과 같이 처리됩니다:

| 태그 | 값 |
| --- | --- |
| `Chloros:ProcessingLevel` | `index` (따라서 파일이 캡처 파일로 다시 가져오지 않습니다) |
| `Chloros:IndexApplied` / `Chloros:IndexExpression` | `True` / 사용된 공식 |
| `Chloros:IndexLevel` | `reflectance` 또는 `radiance` — 지수가 무엇으로 계산되었는지 (둘 다 해당되지 않을 경우 생략) |
| `Chloros:IlluminationInvariant` | 반사율로부터 계산된 지수의 경우 `True`, 복사도 또는 센서 값으로부터 계산된 지수의 경우 `False` |
| `Chloros:ChlorosPipelineVersion` | 인덱스를 작성한 Chloros 버전 |

`Chloros:PixelScale`는 인덱스 제품에서 제거되었습니다 — 위의 `IndexMin` / `IndexMax`를 사용하십시오. 전체 리드백 레시피는 [CLI 참조](reference/cli-reference.md)의 “정수 인덱스 래스터는 이미지별 대비 확장입니다” 섹션에 있습니다.

## 반사율 픽셀 값 읽기

반사율은 정수 디지털 번호(DN)로 저장되며, **ρ = 1.0(반사율 100%)을 나타내는 DN은 원본 카메라에 따라 다릅니다**:

| 원본 카메라 | ρ = 1.0에 해당하는 DN | 확인 방법 |
| --- | --- | --- |
| LATTICE (M3C / M3M) | `32768` (ρ 2.0까지 헤드룸) | 파일에 XMP 태그 `Chloros:PixelScale=32768`가 찍혀 있습니다. |
| Survey3 | `65535` (ρ 1.0에서 클리핑됨) | `Chloros:PixelScale` 태그 없음 — 이 부재가 바로 신호입니다. (1.3.0 버전 기준, Survey3 산출물에는 모든 처리 모드에서 `Chloros:*`와 같은 다른 `Chloros:CalibrationApplied` 처리 태그가 포함되어 있습니다.) |

**`Chloros:PixelScale` XMP 태그를 읽고 그 값으로 나누십시오**. 일정한 값이라고 가정하지 마십시오. 이 태그는 uint16 도메인으로 정의되어 있으므로, 재스케일링이 이루어지는 출력 형식에서도 `32768`로 유지됩니다. 저장된 데이터 유형을 먼저 uint16으로 정규화해야 합니다(8비트에서 ×257, float32에서 ×65535).

{% hint style="warning" %}
**8비트 캡처.** 8비트 캡처의 출력 값은 8비트입니다. 이 캡처의 반사율은 재조정되어 `Chloros:PixelScale=32768`로 표시되므로 스케일이 유지됩니다. 8비트 DN 제품 및 1.3.0 이전에 작성된 반사도 파일은 0–255 범위로 클리핑되며 스케일 태그가 포함되지 않습니다. LATTICE 반사도 파일에 태그가 없는 경우, 스케일이 있다고 가정하지 말고 대신 16비트 또는 32비트로 재내보내십시오.
{% endhint %}

전체 규칙(MicaSense 호환 태그 포함)에 대해서는 [CLI 참조](reference/cli-reference.md)의 **&quot;반사율 픽셀 읽기&quot;**를 참조하십시오.
