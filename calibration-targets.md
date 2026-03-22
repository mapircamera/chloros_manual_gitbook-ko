---
description: Lab-measured panels used to calibrate captured data in post processing
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/calibration-targets
---

# 교정 타겟

MAPIR는 다양한 응용 분야를 아우르는 여러 종류의 교정 타겟을 제공합니다. 아래에 보이는 소형 T4-R50 모델은 250~2,500 nm 파장 범위에서 광 반사율이 측정된 4개의 패널로 구성되어 있습니다.

<figure><img src=".gitbook/assets/t4-r50_2.jpg" alt=""><figcaption><p>MAPIR T4-R50</p></figcaption></figure>T4 확산 기준 타겟의 반사율 곡선은 다음과 같습니다. [데이터 다운로드](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (250-2500nm).png" alt=""><figcaption><p>MAPIR T4 반사율 :: 250-2500nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (400-1000nm).png" alt=""><figcaption><p>MAPIR T4 반사율 :: 400-1000nm</p></figcaption></figure>T4P 확산 기준 타겟의 반사율 곡선은 다음과 같습니다. [데이터 다운로드](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4P -- 350-2500nm.jpg" alt=""><figcaption><p>MAPIR T4P 반사율 :: 250-2500nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4P -- 400-1000nm.jpg" alt=""><figcaption><p>MAPIR T4P 반사율 :: 400-1000nm</p></figcaption></figure>반사율 그래프를 보면 값이 파장(x축) 대 반사율 백분율(y축)로 표시되어 있음을 알 수 있습니다. 보정 타겟의 이미지를 캡처하면, 카메라 센서의 각 대역이 감지하는 스펙트럼 범위 내에서 픽셀 값과 반사율 백분율 간의 관계를 설정하게 됩니다.

즉, 당사 카메라로 촬영한 모든 이미지에 대해 [T4-R50](https://www.mapir.camera/collections/calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t3-r50)이나 [T4-R125](https://www.mapir.camera/collections/multispectral-reflectance-reference-calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t4-r125)와 같은 반사율 타겟 사진을 사용하여 이미지의 반사율을 보정할 수 있습니다. 보정이 완료되면 이미지의 각 픽셀은 반사율 백분율과 일치합니다.

Chloros에서 보정된 이미지를 일반적인 JPG 또는 TIFF 형식으로 출력하면, 픽셀 값을 이미지 형식의 비트 심도로 나누어 반사율 백분율을 계산합니다. 따라서 JPG의 경우 255로 나누고, TIFF의 경우 65,535로 나눕니다. 또한 Chloros에서 PERCENT 형식 출력을 선택하면 각 픽셀의 값이 0.0에서 1.0 사이의 백분율 값(0%에서 100% 반사율) 범위를 갖게 됩니다. 다만 일부 이미지 응용 프로그램은 백분율(부동 소수점) 형식의 이미지를 지원하지 않을 수 있으며, 저장 공간 측면에서 파일 크기가 크다는 점을 유의하시기 바랍니다.

<div><figure><img src=".gitbook/assets/t3-125.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_2.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_closed.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure></div>
