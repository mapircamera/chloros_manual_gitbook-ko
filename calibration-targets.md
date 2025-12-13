---
description: Lab-measured panels used to calibrate captured data in post-processing
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/calibration-targets
---
# 교정 타겟

MAPIR는 다양한 응용 분야를 커버하기 위해 여러 교정 타겟을 제공합니다. 아래에 보이는 소형 T4-R50에는 250~2,500 nm 범위의 빛 반사율을 측정된 4개의 패널이 포함되어 있습니다.

<figure><img src=".gitbook/assets/t4-r50_2.jpg" alt=""><figcaption><p>MAPIR T4-R50</p></figcaption></figure>

T4 확산 기준 타겟은 다음과 같은 반사율 곡선을 가집니다. [데이터 다운로드](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (250-2500nm).png" alt=""><figcaption><p>MAPIR T4 반사율 :: 250-2500nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (400-1000nm).png" alt=""><figcaption><p>MAPIR T4 반사율 :: 400-1000nm</p></figcaption></figure>반사율 그래프를 보면 값이 파장(x축) 대 반사율 백분율(y축)로 표시되어 있음을 확인할 수 있습니다. 교정 타겟의 이미지를 캡처하면 카메라 센서 밴드 각각이 감지하는 스펙트럼 범위 내에서 픽셀 값과 반사율 백분율 간의 관계를 생성합니다.

즉, 당사 카메라로 촬영한 모든 이미지에서 [T4-R50](https://www.mapir.camera/collections/calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t3-r50) 또는 [T4-R125](https://www.mapir.camera/collections/multispectral-reflectance-reference-calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t4-r125)와 같은 당사의 반사율 타깃 사진을 사용하여 이미지의 반사율을 보정할 수 있습니다. 보정된 이미지의 각 픽셀은 반사율 백분율에 해당합니다.

보정된 이미지를 일반적인 JPG(Chloros) 또는 XPROTX(TIFF)로 출력할 경우, 반사율 백분율은 픽셀 값을 이미지 포맷의 비트 심도로 나눈 값으로 계산됩니다. 따라서 JPG의 경우 255로, XPROTX(TIFF)의 경우 65,535로 나누어야 합니다. Chloros에서 PERCENT 형식 출력을 선택할 수도 있으며, 이 경우 각 픽셀은 0.0~1.0(0%~100% 반사율) 범위의 백분율 값을 가집니다. 다만 일부 이미지 애플리케이션은 백분율(부동 소수점) 이미지를 지원하지 않으며, 저장 공간 측면에서 파일 크기가 크다는 점을 유의하십시오.

<div><figure><img src=".gitbook/assets/t3-125.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_2.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_closed.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure></div>
