---
description: This page lists some multispectral indices that Chloros uses
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/multispectral-index-formulas
---

# 다중 스펙트럼 지수 공식

아래 지수 공식은 Survey3 필터의 평균 투과율 범위를 조합하여 사용합니다:

<table><thead><tr><th align="center">Survey3 필터 색상</th><th width="196.199951171875" align="center">Survey3 필터 이름</th><th width="159.800048828125" align="center">투과율 범위 (FWHM)</th><th align="center">평균 투과율</th></tr></thead><tbody><tr><td align="center">Blue</td><td align="center">NGB - Blue</td><td align="center">468-483nm</td><td align="center">475nm</td></tr><tr><td align="center">Cyan</td><td align="center">OCN- Cyan</td><td align="center">476-512nm</td><td align="center">494nm</td></tr><tr><td align="center">Green</td><td align="center">RGN | NGB - Green</td><td align="center">543-558nm</td><td align="center">547nm</td></tr><tr><td align="center">Orange</td><td align="center">OCN - Orange</td><td align="center">598-640nm</td><td align="center">619nm</td></tr><tr><td align="center">Red</td><td align="center">RGN - Red</td><td align="center">653-668nm</td><td align="center">661nm</td></tr><tr><td align="center">RedEdge</td><td align="center">Re - RedEdge</td><td align="center">712-735nm</td><td align="center">724nm</td></tr><tr><td align="center">NIR1</td><td align="center">OCN - NIR1</td><td align="center">798-848nm</td><td align="center">823nm</td></tr><tr><td align="center">NIR2</td><td align="center">RGN | NGB | NIR - NIR2</td><td align="center">835-865nm</td><td align="center">850nm</td></tr></tbody></table>이 공식들을 사용할 경우, 이름은 &quot;\_1&quot; 또는 &quot;\_2&quot;로 끝날 수 있으며, 이는 NIR 필터, NIR1 또는 NIR2 중 어느 것이 사용되었는지를 나타냅니다.

LATTICE M3C(Bayer 트리플 밴드패스) 카메라의 경우, 동일한 인덱스 엔진이 M3C 필터 대역을 사용합니다.

| M3C 필터 | 대역 1 (중심/FWHM) | 대역 2 (중심/FWHM) | 대역 3 (중심/FWHM) |
| --- | --- | --- | --- |
| FRGB | Blue 475 nm / 30 nm | Green 550 nm / 30 nm | Red 625 nm / 30 nm |
| FRGN | Red 660 nm / 21 nm | Green 550 nm / 30 nm | NIR 850 nm / 30 nm |
| FOCN | Orange 615 nm / 21 nm | Cyan 490 nm / 38 nm | NIR 808 nm / 14 nm |
| FNGB | Blue 475 nm / 30 nm | Green 550 nm / 30 nm | NIR 850 nm / 30 nm |

LATTICE M3M 카메라는 단일 대역(카메라당 하나의 협대역 필터)이므로, 따라서 단독 M3M 이미지에 대해서는 다중 대역 지수가 계산되지 않습니다. M3M으로 지수를 계산하려면 두 대 이상의 카메라를 정렬된 다중 대역 스택으로 결합하고 LATTICE 지수 엔진(`chloros-cli lattice index` 또는 GUI의 실시간 지수 계산기)을 사용하십시오.

***

## 각 지수 이름이 적용되는 곳

Chloros에는 **세 가지** 지수 표면이 있으며, 각 표면의 사전 설정 목록은 서로 다릅니다. 이 섹션을 통해 특정 지수 이름을 사용하려는 위치에서 해당 이름이 적용되는지 확인하십시오.

| 현재 위치 | 적용되는 목록 | 개수 |
| --- | --- | --- |
| 프로젝트 설정 → 인덱스 → 인덱스 추가 (GUI) | 표면 1 | 27 |
| 이미지 뷰어 [인덱스/LUT 샌드박스](../image-viewer-gui/index-lut-sandbox.md) (GUI) | 표면 1 | 27 |
| `chloros-cli process --indices NDVI,NDRE` | 표면 2 | 22 |
| SDK `process_folder(indices=[...])` | 표면 2 | 22 |
| `chloros-cli lattice index --preset` | 표면 3 | 22 (다른 22) |
| 카메라 탭 실시간 인덱스 계산기 | 표면 3 | 22 (다른 22) |

Surface 1과 2는 **한 대의 카메라에서 한 번에 하나의 이미지**를 처리하며, 해당 카메라의 필터 채널에 바인딩된 `x`/`y`/`z`(/`a`) 심볼 슬롯을 사용하여 해당 카메라의 필터 채널에 바인딩된**한 번에 한 장의 이미지**를 처리합니다. Surface 3는**정렬된 다중 대역 스택**(여러 대의 LATTICE 카메라가 하나의 큐브로 공동 등록된 것)을 처리하며, 소문자 이름으로 채널을 참조합니다.

### 1. GUI 프로젝트 설정 / 이미지 뷰어 샌드박스 드롭다운 — 27개 수식

드롭다운에는 다음 순서대로 나열됩니다(알파벳 순서가 아닌 삽입 순서입니다):

`NDVI, GNDVI, CVI, ENDVI, EVI, MSR, OSAVI, TDVI, LAI, FCI1, FCI2, GARI, GCI, GEMI, GLI, GOSAVI, GRVI, GSAVI, LCI, MNLI, MSAVI2, NDRE, NLI, RDVI, SAVI, VARI, WDRVI`

GUI에서 카메라의 필터 채널을 수식의 밴드 슬롯으로 드래그하면, 카메라가 지원하는 모든 밴드 할당 방식에 어떤 수식이라도 사용할 수 있습니다. 저장해 둔 사용자 정의 수식은 이 목록 하단에 추가됩니다.

CLI/SDK `--indices` 목록에서 허용하지 않는 **GUI 전용** 5개의 수식은 다음과 같이 구현됩니다:

| GUI 전용 프리셋 | 수식 (구현 방식) | 슬롯 |
| --- | --- | --- |
| FCI1 | `x*y` | x, y |
| FCI2 | `x*y` | x, y |
| GARI | `(y-(x-1.7*(z-a)))/(y+(x-1.7*(z-a)))` | x, y | | FCI2 | `x*y` | x, y | | |
| GEMI | `((2*(y*y-x*x)+1.5*y+0.5*x)/(y+x+0.5))*(1-0.25*((2*(y*y-x*x)+1.5*y+0.5*x)/(y+x+0.5)))-((x-0.125)/(1-x))` | x, y |
| LCI | `(y-x)/(y+z)` | x, y, z |

각 항목에 대한 의도된 매핑은 이 페이지 하단의 해당 섹션에 명시되어 있습니다(예를 들어, GARI는 x=Green, y=NIR, z=Blue, a=Red). GARI는 Chloros 내에서 네 번째 슬롯을 사용하는 유일한 공식입니다.

### 2. CLI / SDK `--indices` 이름 확장 — 22개 사전 설정

`chloros-cli process --indices` 옵션 (및 SDK `indices` 매개변수)는 다음 프리셋 이름을 지원합니다:

`NDVI, GNDVI, NDRE, OSAVI, SAVI, MSAVI2, EVI, MSR, TDVI, LAI, GCI, GRVI, GSAVI, GOSAVI, NLI, MNLI, RDVI, WDRVI, CVI, ENDVI, GLI, VARI`

{% hint style="warning" %}
**알 수 없는 인덱스 이름은 아무런 메시지 없이 건너뜁니다.** 이 목록에 없는 이름(GUI 전용 공식 5개인 `FCI1`, `FCI2`, `GARI`, `GEMI`, `LCI` 및 GUI에 저장한 사용자 정의 수식 포함)은 로그 알림만 표시된 채 생략됩니다. 즉, 해당 인덱스 없이 실행이 계속되며, 실행 자체는 여전히 성공으로 보고됩니다. 알림은 다음과 같이 출력됩니다:

```
[INDEX_EXPAND] skipping unknown preset 'LCI'; known: ['CVI', 'ENDVI', 'EVI', ...]
```

이름은 공백을 제거한 후 대소문자를대소문자를 구분하지 않고 일치시키므로, `ndvi`, `NDVI` 및 ` NDVI `는 모두 동일한 프리셋입니다. 또한 카메라의필터가 제공하지 않는 대역이 필요한 경우에도 해당 프리셋은 건너뜁니다.
{% endhint %}

구현된 정확한 공식은 다음과 같습니다(기호 `x`/`y`/`z`는 밴드 슬롯이며, 프리셋별 기본 매핑이 표시되어 있습니다):

| 프리셋 | 공식 (구현된 형태) | 기본 필터 | 슬롯 (x, y, z) |
| --- | --- | --- | --- |
| NDVI | `(y-x)/(y+x)` | RGN | Red, NIR |
| GNDVI | `(y-x)/(y+x)` | RGN | Green, NIR |
| NDRE | `(y-x)/(y+x)` | RE | RE, NIR |
| OSAVI | `(y-x)/(y+x+0.16)` | RGN | Red, NIR |
| SAVI | `1.5*(y-x)/(y+x+0.5)` | RGN | Red, NIR |
| MSAVI2 | `(2*y+1-sqrt((2*y+1)*(2*y+1)-8*(y-x)))/2` | RGN | Red, NIR |
| EVI | `2.5*(y-x)/(y+6*x-7.5*z+1)` | RGN | Red, NIR, Blue |
| MSR | `((y/x)-1)/(sqrt(y/x)+1)` | RGN | Red, NIR |
| TDVI | `1.5*(y-x)/sqrt(y*y+x+0.5)` | RGN | Red, NIR |
| LAI | `3.618*(2.5*(y-x)/(y+6*x-7.5*z+1))-0.118` | RGN | Red, NIR, Blue |
| GCI | `(y/x)-1` | RGN | Green, NIR |
| GRVI | `y/x` | RGN | Green, NIR |
| GSAVI | `1.5*(y-x)/(y+x+0.5)` | RGN | Green, NIR |
| GOSAVI | `(y-x)/(y+x+0.16)` | RGN | Green, NIR |
| NLI | `((y*y)-x)/((y*y)+x)` | RGN | Red, NIR |
| MNLI | `((y*y-x)*(1+0.5))/((y*y)+x+0.5)` | RGN | Red, NIR |
| RDVI | `(y-x)/sqrt(y+x)` | RGN | Red, NIR |
| WDRVI | `(0.2*y-x)/(0.2*y+x)` | RGN | Red, NIR |
| CVI | `(z/y)/(x/y)` | RGB | Red, Green, Blue |
| ENDVI | `((x+y)-(2*z))/((x+y)+(2*z))` | RGB | Red, Green, Blue |
| GLI | `((y-x)+(y-z))/((2*y)+x+z)` | RGB | Red, Green, Blue |
| VARI | `(y-x)/(y+x-z)` | RGB | Red, Green, Blue |

#### 프리셋 이름이 어떻게 채널 위치로 변환되는가

`NDVI`와 같은 이름만 전달하면, Chloros는 각 심볼이 어느 파일의 어느 채널을 읽을지 결정해야 합니다. 이를 위해 필터 코드를 각 채널의 배열 위치에 매핑하는 다음 표를 사용합니다:

| 필터 코드 | 채널 → 배열 인덱스 |
| --- | --- |
| OCN | Orange 0, Cyan 1, NIR 2 (`Red`는 Orange의 별칭으로 허용되며, 이 역시 0임) |
| RGN | Red 0, Green 1, NIR 2 |
| NGB | NIR 0, Green 1, Blue 2 |
| RGB | Red 0, Green 1, Blue 2 |
| RE | RE 0 |
| NIR | NIR 0 |

프리셋의 **기본 필터**(위의 “기본 필터” 열) 는 프로젝트에 해당 필터가 적용된 이미지가 포함되어 있을 때 사용됩니다. 해당 필터가 없는 경우, Chloros는 프로젝트에 실제로 존재하는 필터를 `RGN, OCN, NGB, RGB, RE, NIR` 순서대로 검색하여, 프리셋에 필요한 모든 채널을 제공할 수 있는 첫 번째 필터를 선택합니다. 적합한 필터가 없는 경우, 해당 실행에서는 프리셋이 제외됩니다. 이것이 바로 OCN만 포함된 데이터셋에서 `NDVI`를 요청해도 여전히 타당한 결과가 도출되는 이유입니다. 즉, OCN의Orange 및 NIR 위치에 바인딩되기 때문입니다.

LATTICE M3C 모델 문자열은 `F` 접두사가 붙은 필터를 포함하고 있습니다 (`LATT-M3C-L41-FRGN`)를 포함하고 있지만, 이미지에서 필터 코드를 읽을 때 접두사가 제거되므로 FRGN 카메라는 위의 `RGN` 행을 통해 식별하며 별도의 처리가 필요하지 않습니다.

### 3. LATTICE 인덱스 엔진 (`lattice index --preset`, 실시간 인덱스 계산기) — 22가지 사전 설정

LATTICE 엔진은 정렬된 다중 대역 스택(라이브 어레이 또는 내보낸 다중 대역 TIFF)에서 작동하며 소문자 채널 이름을 사용합니다(`red`, `green`, `blue`, `red_edge`, `nir`)를 사용합니다. 이 엔진의 프리셋 목록은 앞서 언급한 두 가지와 다릅니다:

| 프리셋 | 공식 | 채널 |
| --- | --- | --- |
| NDVI | `(nir - red) / (nir + red)` | 빨강, NIR |
| GNDVI | `(nir - green) / (nir + green)` | 녹색, NIR |
| BNDVI | `(nir - blue) / (nir + blue)` | 파란색, NIR |
| NDRE | `(nir - red_edge) / (nir + red_edge)` | 빨강\_가장자리, NIR |
| ENDVI | `((nir + green) - 2*blue) / ((nir + green) + 2*blue)` | 파랑, 녹색, 근적외선 |
| SAVI | `1.5 * (nir - red) / (nir + red + 0.5)` | 빨강, 근적외선 |
| OSAVI | `1.5 * (nir - red) / (nir + red + 0.16)` | 빨강, 근적외선 |
| MSAVI | `(2*nir + 1 - sqrt((2*nir + 1)**2 - 8*(nir - red))) / 2` | 빨강, 근적외선 |
| EVI | `2.5 * (nir - red) / (nir + 6*red - 7.5*blue + 1)` | 파랑, 빨강, 근적외선 |
| EVI2 | `2.5 * (nir - red) / (nir + 2.4*red + 1)` | 적색, 근적외선 |
| CVI | `(nir / green) - (red / green)` | 적색, 녹색, 근적외선 |
| MSR | `((nir/red) - 1) / (sqrt(nir/red) + 1)` | 빨강, 근적외선 |
| TDVI | `sqrt((nir - red) / (nir + red) + 0.5)` | 빨강, 근적외선 |
| LAI | `3.618 * ((nir - red) / (nir + 6*red - 7.5*green + 1)) - 0.118` | 적색, 녹색, 근적외선 |
| GLI | `(2*green - red - blue) / (2*green + red + blue)` | 적색, 녹색, 청색 |
| NGRDI | `(green - red) / (green + red)` | 적색, 녹색 |
| VARI | `(green - red) / (green + red - blue)` | 빨강, 초록, 파랑 |
| TGI | `green - 0.39*red - 0.61*blue` | 빨강, 초록, 파랑 |
| EXG | `2*green - red - blue` | 빨강, 초록, 파랑 |
| CIRE | `(nir / red_edge) - 1` | 빨강\_가장자리, NIR |
| CIGREEN | `(nir / green) - 1` | 녹색, NIR |
| NDWI | `(green - nir) / (green + nir)` | 녹색, 근적외선 |

설치된 빌드에서 이 표를 출력하려면 `chloros-cli lattice index --list-presets`를, 사용 가능한 색상 그라데이션을 확인하려면 `--list-gradients`를 실행하십시오. 채널 기호는 대소문자를 구분하며, 프리셋의 소문자 이름(예: `--channel red=Red_660 --channel nir=NIR_850`)과 일치해야 합니다.

***

## CVI

GUI 및 CLI/SDK 사전 설정 목록에 구현된 바와 같이, CVI는 비율의 비율 공식입니다:

$$
CVI = {(z / y) \over (x / y)}
$$

기본 채널 매핑은 x=Red, y=Green, z=Blue입니다. GUI에서 카메라의 채널을 x/y/z 슬롯으로 드래그할 수 있습니다. LATTICE 인덱스 엔진의 `CVI` 사전 설정은 다른 공식(`(NIR / Green) - (Red / Green)`)을 사용하므로, 사용 중인 표면에 맞는 공식을 확인하려면 위의 표를 참조하십시오.

***

## ENDVI - 향상된 정규화 차분 식생 지수

이 지수는 NIR 및 녹색 채널에 더해 청색 채널을 사용하며, 청색 대역이 적색을 대체하는 NGB 필터가 적용된 카메라에서 널리 사용됩니다.

$$
ENDVI = {(NIR + Green) - (2 * Blue) \over (NIR + Green) + (2 * Blue)}
$$

구현 방식은 기호 공식 `((x+y)-(2*z))/((x+y)+(2*z))`입니다. — 카메라의 NIR 및 Green 채널을 x/y 슬롯에, Blue를 z 슬롯에 할당하십시오 (NGB 카메라의 경우: x=NIR, y=Green, z=Blue).

***

## EVI - 향상된 식생 지수

이 지수는 원래 MODIS 데이터와 함께 사용하기 위해 개발되었으며, 잎 면적 지수(LAI)가 높은 지역의 식생 신호를 최적화함으로써 NDVI를 개선한 것입니다. 이 지수는 NDVI가 포화될 수 있는 높은 LAI 지역에서 가장 유용합니다. 이 지수는 청색 반사율 영역을 사용하여 토양 배경 신호를 보정하고, 에어로졸 산란을 포함한 대기 영향력을 줄입니다.

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

EVI 값은 식생 픽셀의 경우 0에서 1 사이의 범위를 가져야 합니다. 구름이나 흰색 건물과 같은 밝은 특징과 물과 같은 어두운 특징은 EVI 이미지에서 비정상적인 픽셀 값을 유발할 수 있습니다. EVI 이미지를 생성하기 전에, 반사도 이미지에서 구름과 밝은 물체를 마스킹해야 하며, 선택적으로 픽셀 값을 0에서 1 사이의 임계값으로 제한해야 합니다.

_참고 문헌: Huete, A. 외. &quot;MODIS 식생 지수의 방사측정 및 생물물리학적 성능 개요.&quot; Remote Sensing of Environment 83 (2002):195–213._

***

## FCI1 - 산림 피복 지수 1

_GUI 전용 — CLI/SDK `--indices` 사전 설정._

이 지수는 레드 엣지 대역을 포함하는 다중 스펙트럼 반사도 영상을 사용하여 산림 수관층을 다른 유형의 식생과 구별합니다.

$$
FCI1 = Red * RedEdge
$$

산림 지역은 나무의 반사율이 낮고 수관 내에 그림자가 존재하기 때문에 FCI1 값이 더 낮게 나타납니다.

_참고 문헌: Becker, Sarah J., Craig S.T. Daughtry, Andrew L. Russ. &quot;다중 스펙트럼 이미지를 위한 견고한 산림 피복 지수.&quot; Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018): 505-512._

***

## FCI2 - 산림 피복 지수 2

_GUI 전용 — CLI/SDK 또는 `--indices` 사전 설정으로는 사용할 수 없습니다._

이 지수는 레드 엣지 대역을 포함하지 않는 다중 스펙트럼 반사율 영상을 사용하여 산림 수관층을 다른 유형의 식생과 구별합니다.

$$
FCI2 = Red * NIR
$$

산림 지역은 나무의 반사율이 낮고 수관 내부에 그림자가 존재하기 때문에 FCI2 값이 더 낮게 나타납니다.

_참고 문헌: Becker, Sarah J., Craig S.T. Daughtry, Andrew L. Russ. &quot;다중 스펙트럼 이미지를 위한 견고한 산림 피복 지수.&quot; Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018): 505-512._

***

## GEMI - 전 지구 환경 모니터링 지수

_GUI 전용 — CLI/SDK `--indices` 프리셋으로는 제공되지 않습니다._

이 비선형 식생 지수는 위성 영상을 통한 전 지구 환경 모니터링에 사용되며, 대기 영향에 대한 보정을 시도합니다. NDVI와 유사하지만 대기 영향에 대한 민감도가 더 낮습니다. 이 지수는 노출된 토양의 영향을 받기 때문에, 식생이 드문 지역이나 중간 밀도의 식생 지역에서는 사용을 권장하지 않습니다.

$$
GEMI = eta (1 - 0.25 * eta) - {Red - 0.125 \over 1 - Red}
$$

여기서:

$$
eta = {2(NIR^{2}-Red^{2}) + 1.5 * NIR + 0.5 *  Red \over NIR + Red + 0.5}
$$

_참고 문헌: Pinty, B., and M. Verstraete. GEMI: 위성을 이용한 전 지구 식생 모니터링을 위한 비선형 지수. Vegetation 101 (1992): 15-20._

***

## GARI - Green 대기 영향 저항 지수

_GUI 전용 — CLI/SDK `--indices` 사전 설정으로는 사용할 수 없음._

이 지수는 NDVI에 비해 광범위한 엽록소 농도에 더 민감하고, 대기 영향에는 덜 민감합니다.

$$
GARI = {NIR - [Green - \gamma(Blue - Red)] \over NIR + [Green - \gamma(Blue - Red)]   }
$$

감마 상수는 대기 중 에어로졸 조건에 따라 달라지는 가중치 함수입니다. ENVI는 Gitelson, Kaufman 및 Merzylak (1996, 296쪽)이 권장하는 값인 1.7을 사용합니다.

_참고 문헌: Gitelson, A., Y. Kaufman, M. Merzylak. &quot;EOS-MODIS를 이용한 전 지구 식생 원격 감지에서 Green 채널의 활용.&quot; Remote Sensing of Environment 58 (1996): 289-298._

***

## GCI - Green 엽록소 지수

이 지수는 광범위한 식물 종에 걸쳐 잎의 엽록소 함량을 추정하는 데 사용됩니다.

$$
GCI = {NIR \over Green} - 1
$$

광범위한 NIR 및 녹색 파장대를 활용하면 엽록소 함량을 더 정확하게 예측할 수 있을 뿐만 아니라 감도를 높이고 신호 대 잡음비를 개선할 수 있다.

_참고 문헌: Gitelson, A., Y. Gritz, M. Merzlyak. “잎의 엽록소 함량과 스펙트럼 반사율 간의 관계 및 고등식물 잎의 비파괴적 엽록소 평가를 위한 알고리즘.” Journal of Plant Physiology 160 (2003): 271-282._

***

## GLI - Green 잎 지수

이 지수는 원래 디지털 RGB 카메라와 함께 사용하여 밀 피복률을 측정하기 위해 고안되었으며, 여기서 적색, 녹색, 파란색 디지털 수치(DN)가 0에서 255 사이의 범위를 가집니다.

$$
GLI = {(Green - Red) + (Green - Blue)  \over (2 * Green) + Red + Blue }
$$

GLI 값은 -1에서 +1까지입니다. 음수 값은 토양 및 무생물적 특징을 나타내는 반면, 양수 값은 녹색 잎과 줄기를 나타냅니다.

_참고 문헌: Louhaichi, M., M. Borman, D. Johnson. &quot;밀에 대한 방목 영향 기록을 위한 공간 위치 기반 플랫폼 및 항공 사진 촬영.&quot; Geocarto International 16, 제1호 (2001): 65-70._

***

## GNDVI - Green 정규화 차분 식생 지수

이 지수는 NDVI와 유사하지만, 적색 스펙트럼 대신 540~570 nm의 녹색 스펙트럼을 측정한다는 점이 다릅니다. 이 지수는 NDVI보다 엽록소 농도에 더 민감합니다.

$$
GNDVI = {(NIR - Green) \over (NIR + Green)  }
$$

_참고 문헌: Gitelson, A., and M. Merzlyak. &quot;고등 식물 잎의 엽록소 농도에 대한 원격 탐사.&quot; Advances in Space Research 22 (1998): 689-692._

***

## GOSAVI - Green 최적화된 토양 보정 식생 지수

이 지수는 원래 옥수수의 질소 요구량을 예측하기 위해 컬러 적외선 촬영을 통해 고안되었습니다. OSAVI와 유사하지만, 녹색 대역을 적색 대역으로 대체한 것이 특징입니다.

$$
GOSAVI = {NIR - Green \over NIR + Green + 0.16)  }
$$

_참고 문헌: Sripada, R. 외. &quot;항공 컬러-적외선 사진을 이용한 옥수수의 생육기 중 질소 요구량 산정.&quot; 박사 학위 논문, 노스캐롤라이나 주립대학교, 2005._

***

## GRVI - Green 비율 식생 지수

이 지수는 녹색 및 적색 반사율이 잎 색소의 변화에 큰 영향을 받기 때문에, 산림 수관 내의 광합성 속도에 민감합니다.

$$
GRVI = {NIR \over Green }
$$

_참고 문헌: Sripada, R. 외. &quot;옥수수의 초기 생육기 질소 요구량 산정을 위한 항공 컬러 적외선 촬영.&quot; Agronomy Journal 98 (2006): 968-977._

***

## GSAVI - Green 토양 보정 식생 지수

이 지수는 원래 옥수수의 질소 요구량을 예측하기 위해 컬러-적외선 촬영을 통해 설계되었습니다. SAVI와 유사하지만, 녹색 대역을 적색 대역으로 대체한 것입니다.

$$
GSAVI = 1.5 * {(NIR - Green) \over (NIR + Green + 0.5)  }
$$

_참고 문헌: Sripada, R. 외. &quot;항공 컬러-적외선 사진을 이용한 옥수수의 생육기 중 질소 요구량 산정.&quot; 박사 학위 논문, 노스캐롤라이나 주립대학교, 2005._

***

## LAI - 잎 면적 지수

이 지수는 잎의 피복률을 추정하고 작물의 생육 및 수확량을 예측하는 데 사용됩니다. ENVI는 Boegh 등 (2002)의 다음 경험식을 사용하여 녹색 LAI를 계산합니다:

$$
LAI = 3.618 * EVI - 0.118
$$

여기서 EVI는 다음과 같습니다:

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

LAI의 높은 값은 일반적으로 약 0에서 3.5 사이입니다. 그러나 장면에 포화 픽셀을 생성하는 구름이나 기타 밝은 특징이 포함되어 있는 경우, LAI 값이 3.5를 초과할 수 있습니다. LAI 이미지를 생성하기 전에 장면에서 구름과 밝은 요소를 마스킹하여 제거하는 것이 이상적입니다.

_참고 문헌: Boegh, E., H. Soegaard, N. Broge, C. Hasager, N. Jensen, K. Schelde, 및 A. Thomsen. &quot;농업 분야에서 잎 면적 지수, 질소 농도 및 광합성 효율을 정량화하기 위한 항공 다중 스펙트럼 데이터.&quot; Remote Sensing of Environment 81, 제2-3호 (2002): 179-193._

***

## LCI - 잎 엽록소 지수

_GUI 전용 — CLI/SDK `--indices` 사전 설정으로는 사용할 수 없음._

이 지수는 엽록소 흡수에 의해 유발되는 반사율 변화에 민감한 고등식물의 엽록소 함량을 추정하는 데 사용됩니다.

$$
LCI = {NIR2 - RedEdge \over NIR2 + Red}
$$

_참고 문헌: Datt, B. &quot;유칼립투스 잎의 수분 함량 원격 감지.&quot; Journal of Plant Physiology 154, 제1호 (1999): 30-36._

***

## MNLI - 수정 비선형 지수

이 지수는 토양 배경을 고려하기 위해 토양 보정 식생 지수(SAVI)를 통합한 비선형 지수(NLI)의 개선된 버전입니다. ENVI는 0.5의 수관 배경 보정 계수(_L_) 값을 사용합니다.

$$
MNLI = {(NIR^{2} - Red) * (1 + L) \over (NIR^{2} + Red + L)  }
$$

_참고 문헌: Yang, Z., P. Willis, R. Mueller. &quot;밴드 비율 강화 AWIFS 이미지가 작물 분류 정확도에 미치는 영향.&quot; Pecora 17 원격탐사 심포지엄 논문집 (2008), 콜로라도주 덴버._

***

## MSAVI2 - 수정된 토양 보정 식생 지수 2

이 지수는 Qi 등(1994)이 제안한 MSAVI 지수의 단순화된 버전으로, 토양 보정 식생 지수(SAVI)를 개선한 것입니다. 이 지수는 토양 노이즈를 줄이고 식생 신호의 동적 범위를 확대합니다. MSAVI2는 건강한 식생을 강조하기 위해 (SAVI에서와 같이) 일정한 _L_ 값을 사용하지 않는 귀납적 방법을 기반으로 합니다.

$$
MSAVI2 = {2 * NIR + 1 - \sqrt{(2 * NIR + 1)^{2} - 8(NIR - Red)} \over 2}
$$

_참고 문헌: Qi, J., A. Chehbouni, A. Huete, Y. Kerr, 및 S. Sorooshian. &quot;수정된 토양 보정 식생 지수(Modified Soil Adjusted Vegetation Index).&quot; Remote Sensing of Environment 48 (1994): 119-126._

***

## MSR - 수정 단순 비율

이 지수는 생물물리학적 매개변수와의 관계를 선형화하도록 설계된 단순 NIR/Red 비율을 개량한 것으로, 생물물리학적 매개변수와의 관계를 선형화하기 위해 고안되었으며, 식생 밀도가 높은 경우 NDVI보다 더 민감합니다.

$$
MSR = {(NIR / Red) - 1 \over \sqrt{NIR / Red} + 1}
$$

_참고 문헌: Chen, J. &quot;북방림 지역 적용을 위한 식생 지수 및 수정 단순 비율의 평가.&quot; Canadian Journal of Remote Sensing 22 (1996): 229-242._

***

## NDRE- 정규화 차분 RedEdge

이 지수는 NDVI와 유사하지만, NIR와 RedEdge 간의 대비를 비교하며 , 이는 식생 스트레스를 더 빨리 감지하는 경우가 많습니다.

$$
NDRE = {NIR - RedEdge \over NIR + RedEdge  }
$$

***

## NDVI - 정규화 차분 식생 지수

이 지수는 건강하고 푸른 식생을 측정하는 지표입니다. 정규화된 차분 방식과 엽록소의 최대 흡수 및 반사 영역을 결합함으로써 다양한 조건에서 뛰어난 안정성을 보입니다. 그러나 식생이 밀집된 환경에서 LAI 값이 높아지면 포화될 수 있습니다.

$$
NDVI = {NIR - Red \over NIR + Red  }
$$

이 지수의 값은 -1에서 1 사이입니다. 녹색 식생의 일반적인 범위는 0.2에서 0.8입니다.

_참고 문헌: Rouse, J., R. Haas, J. Schell 및 D. Deering. ERTS를 이용한 그레이트 플레인스의 식생 시스템 모니터링. 제3회 ERTS 심포지엄, NASA (1973): 309-317._

***

## NLI - 비선형 지수

이 지수는 여러 식생 지수와 지표 생물물리학적 매개변수 간의 관계가 비선형적이라고 가정합니다. 이 지수는 비선형적인 경향을 보이는 지표 매개변수와의 관계를 선형화합니다.

$$
NLI = {NIR^{2} - Red \over NIR^{2} + Red  }
$$

_참고 문헌: Goel, N., 및 W. Qin. “다양한 식생 지수와 LAI 및 Fpar 간의 관계에 미치는 수관 구조의 영향: 컴퓨터 시뮬레이션.” Remote Sensing Reviews 10 (1994): 309-347._

***

## OSAVI - 최적화된 토양 보정 식생 지수

이 지수는 토양 보정 식생 지수(SAVI)를 기반으로 합니다. 이 지수는 수관 배경 보정 계수로 0.16이라는 표준값을 사용합니다. Rondeaux (1996)는 이 값이 낮은 식생 피복도에서는 SAVI보다 더 큰 토양 변동성을 제공하는 동시에, 50% 이상의 식생 피복도에 대해서는 더 높은 민감도를 보인다는 것을 확인했습니다. 이 지수는 수관 사이로 토양이 보이는, 식생이 비교적 드문 지역에서 사용하는 것이 가장 적합합니다.

$$
OSAVI = {(NIR - Red) \over (NIR + Red + 0.16)  }
$$

_참고 문헌: Rondeaux, G., M. Steven, 및 F. Baret. &quot;토양 보정 식생 지수의 최적화.&quot; Remote Sensing of Environment 55 (1996): 95-107._

***

## RDVI - 재정규화 차분 식생 지수

이 지수는 근적외선과 적색 파장 간의 차이와 NDVI를 함께 사용하여 건강한 식생을 강조합니다. 이 지수는 토양 및 태양 관측 기하학적 구조의 영향에 민감하지 않습니다.

$$
RDVI = {(NIR- Red) \over \sqrt{(NIR + Red)}  }
$$

_참고 문헌: Roujean, J., 및 F. Breon. “양방향 반사율 측정을 통한 식생에 흡수된 PAR 추정.” 《Remote Sensing of Environment》 51 (1995): 375-384._

***

## SAVI - 토양 보정 식생 지수

이 지수는 NDVI와 유사하지만, 토양 픽셀의 영향을 억제합니다. 이 지수는 수관 배경 보정 계수인 _L_을 사용하는데, 이는 식생 밀도의 함수이며 대개 식생 양에 대한 사전 지식이 필요합니다. Huete(1988)는 1차 토양 배경 변동을 고려하기 위해 _L_=0.5를 최적값으로 제안합니다. 이 지수는 수관 사이로 토양이 보일 정도로 식생이 비교적 드문 지역에서 사용하는 것이 가장 적합합니다.

$$
SAVI = {1.5 * (NIR- Red) \over (NIR + Red + 0.5)  }
$$

_참고 문헌: Huete, A. &quot;토양 보정 식생 지수 (SAVI).&quot; Remote Sensing of Environment 25 (1988): 295-309._

***

## TDVI - 변환 차분 식생 지수

이 지수는 도시 환경의 식생 피복 모니터링에 유용합니다. NDVI 및 SAVI와 달리 포화 현상이 발생하지 않습니다.

$$
TDVI = 1.5 * {(NIR- Red) \over \sqrt{NIR^{2} + Red + 0.5}  }
$$

_참고 문헌: Bannari, A., H. Asalhi, 및 P. Teillet. &quot;식생 피복 매핑을 위한 변환 차분 식생 지수(TDVI)&quot; 『지구과학 및 원격탐사 심포지엄(IGARSS &#x27;02) 논문집』, IEEE International, 제5권 (2002)._

***

## VARI - 가시광선 대기 저항 지수

이 지수는 ARVI를 기반으로 하며, 대기 영향에 대한 민감도가 낮은 장면에서 식생 비율을 추정하는 데 사용됩니다.

$$
VARI = {Green - Red \over Green + Red - Blue  }
$$

_참고 문헌: Gitelson, A., 외. “가시광선 스펙트럼 영역에서의 식생 및 토양 경계선: 식생 비율의 원격 추정 개념 및 기법.” International Journal of Remote Sensing 23 (2002): 2537−2562._

***

## WDRVI - 광역 동적 범위 식생 지수

이 지수는 NDVI와 유사하지만, 가중계수(_a_)를 사용하여 NDVI에 대한 근적외선 및 적색 신호의 기여도 간 차이를 줄입니다. WDRVI는 NDVI가 0.6을 초과하는 중~고 밀도의 식생이 있는 장면에서 특히 효과적입니다. NDVI는 식생 비율과 엽면적 지수 (LAI)가 증가함에 따라 평탄화되는 경향이 있는 반면, WDRVI는 더 넓은 범위의 식생 비율과 LAI의 변화에 더 민감하게 반응합니다.

$$
WDRVI = {(\alpha * NIR- Red) \over (\alpha * NIR + Red)}
$$

가중치 계수(_a_)는 0.1에서 0.2 사이의 범위를 가질 수 있다. Henebry, Viña 및 Gitelson (2004)는 0.2의 값을 권장한다.

_참고문헌_

_Gitelson, A. &quot;식생의 생물물리학적 특성을 원격 정량화하기 위한 광역 동적 범위 식생 지수.&quot; Journal of Plant Physiology 161, 제2호 (2004): 165-173._

_Henebry, G., A. Viña, 및 A. Gitelson. &quot;광역 동적 범위 식생 지수 및 틈새 분석에 대한 잠재적 유용성.&quot; Gap Analysis Bulletin 12: 50-56._
