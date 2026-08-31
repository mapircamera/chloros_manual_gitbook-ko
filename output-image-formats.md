---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/output-image-formats
---

# 출력 이미지 형식

Chloros는 처리된 결과를 네 가지 파일 형식으로 내보냅니다. 프로젝트 설정(GUI)에서, `--format`(CLI) 또는 `export_format` (SDK)를 사용하여 형식을 선택할 수 있습니다. CLI 및 SDK는 아래의 정확한 문자열을 지원합니다.

| 형식 문자열 | 확장자 | 픽셀 유형 | 픽셀 범위 | 비고 |
| --- | --- | --- | --- | --- |
| `TIFF (16-bit)` *(기본값)* | `.tif` | uint16 디지털 숫자 | 0 – 65535 | 사진측량/GIS에 권장. |
| `TIFF (32-bit, Percent)` | `.tif` | float32 | 0.0 – 1.0 | 1.0 = 100% 반사율. 일부 응용 프로그램은 부동 소수점 TIFF 파일을 읽을 수 없으며, 파일 크기가 더 큽니다. |
| `PNG (8-bit)` | `.png` | uint8 디지털 숫자 | 0 – 255 | 무손실 압축, 웹 보기 및 시각화에 적합합니다. |
| `JPG (8-bit)` | `.jpg` | uint8 디지털 숫자 | 0 – 255 | 손실 압축, 파일 크기가 가장 작습니다. |

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

카메라 폴더는 LATTICE의 경우 `LATT-<sensor>-<lens>-F<filter>`이고, Survey3의 경우 `<model>_<filter>`(예: `Survey3N_RGN`)입니다. **내보낸 모든 제품은 원본 파일의 이름을 그대로 유지합니다. 제품을 식별하는 것은 파일 이름의 확장자가 아닌 폴더입니다.** 전체 규칙에 대해서는 CLI 참조 문서의 [출력 파일 저장 위치](reference/cli-reference.md)를 참조하십시오.

## LATTICE 제품 (캡처 및 내보내기 수준)

하나의 LATTICE 원시 프레임은 단일 패스에서 요청된 모든 제품으로 분기됩니다. 각 제품 유형에는 고유한 토글(GUI 확인란 또는 CLI, `--debayered` / `--preview` / `--radiance` / `--reflectance`, 모두 기본적으로 ON으로 설정됨):

| 레벨 | 내용 | 데이터 유형 |
| --- | --- | --- |
| `raw` | 센서에서 직접 가져온 베이어 데이터(모노 카메라: 단일 대역). 처리는 항상 원시 데이터(RAW)에서 시작됩니다. | 캡처된 그대로 |
| `debayered` | 선형 디모자이크 — M3C의 경우 3채널, M3M의 경우 1채널 그레이스케일. | 선형 DN |
| `radiance` | 전체 방사계측 체인에서 도출된 절대 스펙트럼 복사도, 단위는 **W/m²/sr/nm**. 선택한 내보내기 형식과 관계없이 항상 32비트 TIFF (`tiff32/Radiance_Images/`)로 기록됩니다. | float32 |
| `reflectance` | 반사율 ρ. 여기서 **DN 32768 = ρ 1.0 (100%)**이며, ρ 2.0까지의 여유가 있습니다. Pix4D 호환. | uint16 |
| `preview` | 디스플레이용 렌더링: RGB = 화이트 밸런스 + 감마; 다중 스펙트럼 = 가색 변환. | 8비트 디스플레이 |

## 반사율 픽셀 값 읽기

반사율은 정수형 디지털 수치(DN)로 저장되며, **ρ = 1.0(반사율 100%)을 나타내는 DN 값은 원본 카메라에 따라 다릅니다**:

| 원본 카메라 | ρ = 1.0에 해당하는 DN | 확인 방법 |
| --- | --- | --- |
| LATTICE (M3C / M3M) | `32768` (ρ 2.0까지 헤드룸) | 파일에 XMP 태그 `Chloros:PixelScale=32768`가 찍혀 있음. |
| Survey3 | `65535` (ρ 1.0에서 잘림) | `Chloros:*` XMP 태그 없음 — 이 부재가 신호입니다. |

**`Chloros:PixelScale` XMP 태그를 읽고 그 값으로 나누세요**. 상수를 가정하지 마십시오. 이 태그는 uint16 도메인으로 정의되어 있으므로, 재스케일링이 이루어지는 출력 형식에서도 값이 유지됩니다 — 먼저 저장된 데이터형을 uint16으로 정규화하십시오(8비트에서 ×257, float32에서 ×65535).

{% hint style="warning" %}
**설계상 스케일이 적용되지 않는 경우가 하나 있습니다.** 8비트 소스 캡처(BayerRG8)가 8비트 TIFF로 기록될 때, 파이프라인은 재스케일링 대신 0–255 범위로 클리핑하므로, 해당 파일에는 스케일이 적용되지 않습니다 — Chloros는 의도적으로 `Chloros:PixelScale`를 생략합니다. LATTICE 반사율 파일에서 이 태그가 없는 경우, 스케일이 있다고 가정하지 말고 대신 16비트 또는 32비트로 다시 내보내십시오.
{% endhint %}

전체 규칙(MicaSense 호환 태그 포함)은 [CLI 참조](reference/cli-reference.md)의 **&quot;반사율 픽셀 읽기&quot;** 항목을 참조하십시오.
