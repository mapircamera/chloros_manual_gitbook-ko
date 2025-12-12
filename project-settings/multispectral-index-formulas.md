---
description: This page lists some multispectral indices that Chloros uses
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/multispectral-index-formulas
---

# 다중 스펙트럼 지수 공식

아래 지수 공식은 Survey3 필터 평균 투과 범위의 조합을 사용합니다:

<table><thead><tr><th align="center">Survey3 필터 색상</th><th width="196.199951171875" align="center">Survey3 필터명</th><th width="159.800048828125" align="center">투과 범위 (FWHM)</th><th align="center">평균 투과율</th></tr></thead><tbody><tr><td align="center">Blue</td><td align="center">NGB - Blue</td><td align="center">468-483nm</td><td align="center">475nm</td></tr><tr><td align="center">Cyan</td><td align="center">OCN- Cyan</td><td align="center">476-512nm</td><td align="center">494nm</td></tr><tr><td align="center">Green</td><td align="center">RGN | NGB - Green</td><td align="center">543-558nm</td><td align="center">547nm</td></tr><tr><td align="center">Orange</td><td align="center">OCN - Orange</td><td align="center">598-640nm</td><td align="center">619nm</td></tr><tr><td align="center">Red</td><td align="center">RGN - Red</td><td align="center">653-668nm</td><td align="center">661nm</td></tr><tr><td align="center">RedEdge</td><td align="center">Re - RedEdge</td><td align="center">712-735nm</td><td align="center">724nm</td></tr><tr><td align="center">NIR1</td><td align="center">OCN - NIR1</td><td align="center">798-848nm</td><td align="center">823nm</td></tr><tr><td align="center">NIR2</td><td align="center">RGN | NGB | NIR - NIR2</td><td align="center">835-865nm</td><td align="center">850nm</td></tr></tbody></table>이러한 공식이 사용될 경우 이름이 &quot;\_1&quot; 또는 &quot;\_2&quot;로 끝날 수 있으며, 이는 NIR 필터 중 NIR1 또는 NIR2가 사용되었음을 나타냅니다.

***

## EVI - 향상된 식생 지수

이 지수는 원래 MODIS 데이터와 함께 사용하기 위해 개발되었으며, 높은 잎 면적 지수(LAI) 지역에서 식생 신호를 최적화함으로써 NDVI를 개선한 것입니다. NDVI가 포화될 수 있는 높은 LAI 지역에서 가장 유용합니다. 청색 반사 영역을 사용하여 토양 배경 신호를 보정하고 에어로졸 산란을 포함한 대기 영향을 줄입니다.

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

EVI 값은 식생 픽셀의 경우 0에서 1 사이여야 합니다. 구름이나 흰색 건물과 같은 밝은 특징과 물과 같은 어두운 특징은 EVI 이미지에서 비정상적인 픽셀 값을 초래할 수 있습니다. EVI 이미지를 생성하기 전에 반사율 이미지에서 구름과 밝은 특징을 마스크 처리하고, 선택적으로 픽셀 값을 0에서 1로 임계값 처리해야 합니다.

_참고문헌: Huete, A., 등. &quot;식생 지수의 방사계 및 생물물리적 성능 개요.&quot; Remote Sensing of Environment 83 (2002):195–213._

***

## FCI1 - 산림 피복 지수 1

이 지수는 적색 경계 대역을 포함한 다중 스펙트럼 반사율 영상을 사용하여 산림 캐노피를 다른 유형의 식생과 구분합니다.

$$
FCI1 = Red * RedEdge
$$

산림 지역은 나무의 낮은 반사율과 캐노피 내 그림자 존재로 인해 FCI1 값이 낮습니다.

_참고문헌: Becker, Sarah J., Craig S.T. Daughtry, Andrew L. Russ. &quot;다중 스펙트럼 영상을 위한 견고한 산림 피복 지수.&quot; Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018): 505-512._

***

## FCI2 - 산림 피복 지수 2

이 지수는 적색 경계 대역이 포함되지 않은 다중 스펙트럼 반사율 영상을 사용하여 산림 캐노피를 다른 유형의 식생과 구분합니다.

$$
FCI2 = Red * NIR
$$

산림 지역은 나무의 낮은 반사율과 캐노피 내부의 그림자 존재로 인해 더 낮은 FCI2 값을 가집니다.

_참고문헌: Becker, Sarah J., Craig S.T. Daughtry, and Andrew L. Russ. &quot;Robust forest cover indices for multispectral images.&quot; Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018): 505-512._

***

## GEMI - 지구 환경 모니터링 지수

이 비선형 식생 지수는 위성 이미지를 통한 지구 환경 모니터링에 사용되며 대기 효과를 보정하려고 시도합니다. NDVI와 유사하지만 대기 효과에 덜 민감합니다. 노출된 토양의 영향을 받기 때문에 식생이 희박하거나 중간 밀도인 지역에서는 사용을 권장하지 않습니다.

$$
GEMI = eta (1 - 0.25 * eta) - {Red - 0.125 \over 1 - Red}
$$

출처:

$$
eta = {2(NIR^{2}-Red^{2}) + 1.5 * NIR + 0.5 *  Red \over NIR + Red + 0.5}
$$

_참고문헌: Pinty, B., and M. Verstraete. GEMI: 위성으로 지구 식생을 모니터링하기 위한 비선형 지수. Vegetation 101 (1992): 15-20._

***

## GARI - Green 대기 저항 지수

이 지수는 NDVI보다 광범위한 엽록소 농도에 더 민감하고 대기 영향에 덜 민감합니다.

$$
GARI = {NIR - [Green - \gamma(Blue - Red)] \over NIR + [Green - \gamma(Blue - Red)]   }
$$

감마 상수는 대기 중 에어로졸 조건에 따라 달라지는 가중치 함수입니다. ENVI는 Gitelson, Kaufman, Merzylak (1996, 296쪽)이 권장하는 값인 1.7을 사용합니다.

_참고문헌: Gitelson, A., Y. Kaufman, and M. Merzylak. &quot;Use of a Green Channel in Remote Sensing of Global Vegetation from EOS-MODIS.&quot; Remote Sensing of Environment 58 (1996): 289-298._

***

## GCI - Green 엽록소 지수

이 지수는 다양한 식물 종에 걸쳐 잎의 엽록소 함량을 추정하는 데 사용됩니다.

$$
GCI = {NIR \over Green} - 1
$$

넓은 NIR 및 녹색 파장을 확보하면 클로로필 함량을 더 정확히 예측할 수 있으며, 동시에 감도와 신호대잡음비(SNR)를 높일 수 있습니다.

_참고문헌: Gitelson, A., Y. Gritz, and M. Merzlyak. &quot;잎의 엽록소 함량과 분광 반사율 간의 관계 및 고등식물 잎의 비파괴적 엽록소 평가 알고리즘.&quot; Journal of Plant Physiology 160 (2003): 271-282._

***

## GLI - Green 잎 지수

이 지수는 원래 디지털 RGB 카메라와 함께 밀 덮개 측정에 사용하도록 설계되었으며, 적색, 녹색, 청색 디지털 수치(DN)는 0에서 255까지의 범위를 가집니다.

$$
GLI = {(Green - Red) + (Green - Blue)  \over (2 * Green) + Red + Blue }
$$

GLI 값은 -1에서 +1 사이입니다. 음수 값은 토양 및 비생물적 특징을 나타내며, 양수 값은 녹색 잎과 줄기를 나타냅니다.

_참고문헌: Louhaichi, M., M. Borman, and D. Johnson. &quot;밀에 대한 방목 영향 기록을 위한 공간 위치 기반 플랫폼 및 항공 사진.&quot; Geocarto International 16, No. 1 (2001): 65-70._

***

## GNDVI - Green 정규화 차분 식생 지수

이 지수는 NDVI와 유사하나, 적색 스펙트럼 대신 540~570 nm의 녹색 스펙트럼을 측정한다는 점이 다릅니다. 이 지수는 NDVI보다 엽록소 농도에 더 민감합니다.

$$
GNDVI = {(NIR - Green) \over (NIR + Green)  }
$$

_참고문헌: Gitelson, A., and M. Merzlyak. &quot;Remote Sensing of Chlorophyll Concentration in Higher Plant Leaves.&quot; Advances in Space Research 22 (1998): 689-692._

***

## GOSAVI - Green 최적화된 토양 보정 식생 지수

이 지수는 원래 옥수수의 질소 요구량을 예측하기 위해 컬러-적외선 사진 촬영을 통해 설계되었습니다. OSAVI와 유사하지만, 녹색 밴드를 적색 밴드로 대체합니다.

$$
GOSAVI = {NIR - Green \over NIR + Green + 0.16)  }
$$

_참고문헌: Sripada, R. 외. &quot;항공 컬러-적외선 사진을 이용한 옥수수 생육기 질소 요구량 결정.&quot; 박사 학위 논문, 노스캐롤라이나 주립대학교, 2005._

***

## GRVI - Green 비율 식생 지수

이 지수는 녹색과 적색 반사율이 잎 색소 변화에 크게 영향을 받기 때문에 산림 캐노피의 광합성 속도에 민감합니다.

$$
GRVI = {NIR \over Green }
$$

_참고문헌: Sripada, R. 외. &quot;옥수수의 초기 생육기 질소 요구량 결정에 대한 항공 컬러 적외선 사진 촬영.&quot; Agronomy Journal 98 (2006): 968-977._

***

## GSAVI - Green 토양 보정 식생 지수

이 지수는 원래 옥수수의 질소 요구량을 예측하기 위해 컬러-적외선 사진 촬영과 함께 설계되었습니다. SAVI와 유사하지만, 녹색 밴드를 적색 밴드로 대체합니다.

$$
GSAVI = 1.5 * {(NIR - Green) \over (NIR + Green + 0.5)  }
$$

_참고문헌: Sripada, R. 외. &quot;항공 컬러-적외선 사진을 이용한 옥수수의 생육기 질소 요구량 결정.&quot; 박사 학위 논문, 노스캐롤라이나 주립대학교, 2005._

***

## LAI - 엽면적지수

이 지수는 잎 덮개율을 추정하고 작물 생장 및 수확량을 예측하는 데 사용됩니다. ENVI는 Boegh 등(2002)의 다음 경험적 공식을 사용하여 녹색 LAI를 계산합니다:

$$
LAI = 3.618 * EVI - 0.118
$$

여기서 EVI는:

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

높은 LAI 값은 일반적으로 약 0에서 3.5 사이입니다. 그러나 장면에 구름 및 포화 픽셀을 생성하는 기타 밝은 특징이 포함된 경우 LAI 값이 3.5를 초과할 수 있습니다. LAI 이미지를 생성하기 전에 장면에서 구름과 밝은 특징을 가리는 것이 이상적입니다.

_참고문헌: Boegh, E., H. Soegaard, N. Broge, C. Hasager, N. Jensen, K. Schelde, and A. Thomsen. &quot;농업에서 엽면적지수, 질소 농도 및 광합성 효율 정량화를 위한 항공 다중 스펙트럼 데이터.&quot; Remote Sensing of Environment 81, no. 2-3 (2002): 179-193._

***

## LCI - 엽록소 지수

이 지수는 고등식물의 엽록소 함량을 추정하는 데 사용되며, 엽록소 흡수로 인한 반사율 변화에 민감합니다.

$$
LCI = {NIR2 - RedEdge \over NIR2 + Red}
$$

_참고문헌: Datt, B. &quot;유칼립투스 잎의 수분 함량 원격 감지.&quot; Journal of Plant Physiology 154, 제1호 (1999): 30-36._

***

## MNLI - 수정 비선형 지수

이 지수는 토양 배경을 고려하기 위해 토양 조정 식생 지수(SAVI)를 통합한 비선형 지수(NLI)의 개선판입니다. ENVI는 캐노피 배경 조정 계수(_L_) 값으로 0.5를 사용합니다.

$$
MNLI = {(NIR^{2} - Red) * (1 + L) \over (NIR^{2} + Red + L)  }
$$

_참고문헌: Yang, Z., P. Willis, and R. Mueller. &quot;Impact of Band-Ratio Enhanced AWIFS Image to Crop Classification Accuracy.&quot; Proceedings of the Pecora 17 Remote Sensing Symposium (2008), Denver, CO._

***

## MSAVI2 - 수정 토양 조정 식생 지수 2

이 지수는 Qi 등(1994)이 제안한 MSAVI 지수의 단순화된 버전으로, 토양 조정 식생 지수(SAVI)를 개선한 것입니다. 토양 노이즈를 줄이고 식생 신호의 동적 범위를 증가시킵니다. MSAVI2는 건강한 식생을 강조하기 위해 SAVI와 같이 일정한 _L_ 값을 사용하지 않는 유도적 방법을 기반으로 합니다.

$$
MSAVI2 = {2 * NIR + 1 - \sqrt{(2 * NIR + 1)^{2} - 8(NIR - Red)} \over 2}
$$

_참고문헌: Qi, J., A. Chehbouni, A. Huete, Y. Kerr, and S. Sorooshian. &quot;A Modified Soil Adjusted Vegetation Index.&quot; Remote Sensing of Environment 48 (1994): 119-126._

***

## NDRE- 정규화 차분 RedEdge

이 지수는 NDVI와 유사하지만, Red 대신 NIR와 RedEdge 간의 대비를 비교하여 식생 스트레스를 더 빨리 감지하는 경우가 많습니다.

$$
NDRE = {NIR - RedEdge \over NIR + RedEdge  }
$$

***

## NDVI - 정규화 차분 식생 지수

이 지수는 건강한 녹색 식생을 측정합니다. 정규화된 차분 공식과 엽록소의 최고 흡수 및 반사 영역을 활용하는 조합으로 인해 광범위한 조건에서 견고합니다. 그러나 LAI가 높아질 때 밀집된 식생 조건에서는 포화될 수 있습니다.

$$
NDVI = {NIR - Red \over NIR + Red  }
$$

이 지수의 값 범위는 -1에서 1입니다. 녹색 식생의 일반적인 범위는 0.2에서 0.8입니다.

_참고문헌: Rouse, J., R. Haas, J. Schell, and D. Deering. Monitoring Vegetation Systems in the Great Plains with ERTS. 제3회 ERTS 심포지엄, NASA (1973): 309-317._

***

## NLI - 비선형 지수

이 지수는 많은 식생 지수와 지표 생물물리학적 매개변수 간의 관계가 비선형적이라고 가정합니다. 비선형적인 경향이 있는 지표 매개변수와의 관계를 선형화합니다.

$$
NLI = {NIR^{2} - Red \over NIR^{2} + Red  }
$$

_참고문헌: Goel, N., and W. Qin. &quot;관목 구조가 다양한 식생 지수와 LAI 및 Fpar 간의 관계에 미치는 영향: 컴퓨터 시뮬레이션.&quot; 원격 감지 리뷰 10 (1994): 309-347._

***

## OSAVI - 최적화된 토양 보정 식생 지수

이 지수는 토양 보정 식생 지수(SAVI)를 기반으로 합니다. 캐노피 배경 조정 계수에 표준값 0.16을 사용합니다. Rondeaux(1996)는 이 값이 낮은 식생 피복률에서는 SAVI보다 더 큰 토양 변이를 제공하면서도 50% 이상의 식생 피복률에 대해서는 향상된 감도를 보인다고 결론지었습니다. 이 지수는 상대적으로 식생이 드문 지역에서, 즉 수관 사이로 토양이 보이는 지역에서 가장 효과적으로 사용됩니다.

$$
OSAVI = {(NIR - Red) \over (NIR + Red + 0.16)  }
$$

_참고문헌: Rondeaux, G., M. Steven, and F. Baret. &quot;Optimization of Soil-Adjusted Vegetation Indices.&quot; Remote Sensing of Environment 55 (1996): 95-107._

***

## RDVI - 재정규화 차분 식생 지수

이 지수는 근적외선과 적색 파장 간의 차이와 NDVI를 함께 사용하여 건강한 식생을 강조합니다. 토양과 태양 관측 기하학적 구조의 영향에 민감하지 않습니다.

$$
RDVI = {(NIR- Red) \over \sqrt{(NIR + Red)}  }
$$

_참고문헌: Roujean, J., and F. Breon. &quot;Estimating PAR Absorbed by Vegetation from Bidirectional Reflectance Measurements.&quot; Remote Sensing of Environment 51 (1995): 375-384._

***

## SAVI - 토양 보정 식생 지수

이 지수는 NDVI와 유사하지만 토양 픽셀의 영향을 억제합니다. 이는 식생 밀도의 함수인 캐노피 배경 조정 계수 _L_을 사용하며, 종종 식생량에 대한 사전 지식이 필요합니다. Huete (1988)는 1차 토양 배경 변동을 고려하기 위해 최적 값 _L_=0.5를 제안합니다. 이 지수는 상대적으로 식생이 드문 지역에서, 즉 수관 사이로 토양이 보이는 지역에서 가장 효과적으로 사용됩니다.

$$
SAVI = {1.5 * (NIR- Red) \over (NIR + Red + 0.5)  }
$$

_참고문헌: Huete, A. &quot;토양 보정 식생 지수(SAVI).&quot; Remote Sensing of Environment 25 (1988): 295-309._

***

## TDVI - 변환 차분 식생 지수

이 지수는 도시 환경에서 식생 피복을 모니터링하는 데 유용합니다. NDVI 및 SAVI처럼 포화되지 않습니다.

$$
TDVI = 1.5 * {(NIR- Red) \over \sqrt{NIR^{2} + Red + 0.5}  }
$$

_참고문헌: Bannari, A., H. Asalhi, and P. Teillet. &quot;식생 피복 지도 작성을 위한 변환 차분 식생 지수(TDVI)&quot; In Proceedings of the Geoscience and Remote Sensing Symposium, IGARSS &#x27;02, IEEE International, Volume 5 (2002)._

***

## VARI - 가시광선 대기 저항 지수

이 지수는 ARVI를 기반으로 하며, 대기 영향에 대한 민감도가 낮은 장면에서 식생의 비율을 추정하는 데 사용됩니다.

$$
VARI = {Green - Red \over Green + Red - Blue  }
$$

_참고문헌: Gitelson, A. 외. &quot;가시광선 스펙트럼 영역에서의 식생 및 토양 경계선: 원격 식생 비율 추정 개념 및 기법.&quot; International Journal of Remote Sensing 23 (2002): 2537−2562._

***

## WDRVI - 광역 동적 범위 식생 지수

이 지수는 NDVI와 유사하지만, 가중 계수(_a_)를 사용하여 근적외선 신호와 적색 신호가 NDVI에 기여하는 차이를 줄입니다. WDRVI는 NDVI가 0.6을 초과하는 중~고밀도 식생 장면에서 특히 효과적입니다. NDVI는 식생 분율과 잎 면적 지수(LAI)가 증가할 때 평탄화되는 경향이 있는 반면, WDRVI는 더 넓은 범위의 식생 분율과 LAI의 변화에 더 민감합니다.

$$
WDRVI = {(\alpha * NIR- Red) \over (\alpha * NIR + Red)}
$$

가중 계수(_a_)는 0.1에서 0.2 사이의 값을 가질 수 있습니다. Henebry, Viña, Gitelson(2004)은 0.2 값을 권장합니다.

_참고문헌_

_Gitelson, A. &quot;Wide Dynamic Range Vegetation Index for Remote Quantification of Biophysical Characteristics of Vegetation.&quot; Journal of Plant Physiology 161, No. 2 (2004): 165-173._

_Henebry, G., A. Viña, and A. Gitelson. &quot;광역 동적 범위 식생 지수(WDRI)와 갭 분석(Gap Analysis)에 대한 잠재적 유용성.&quot; 갭 분석 회보 12: 50-56._
