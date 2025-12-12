---
description: 후처리에서 캡처된 데이터를 보정하는 데 사용되는 실험실 측정 패널
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/calibration-targets
---

# 교정 대상

MAPIR은 다양한 응용 분야를 포괄하는 다양한 교정 대상을 제공합니다. 아래에 표시된 컴팩트한 T4-R50에는 250 - 2,500 nm의 빛 반사율이 측정된 4개의 패널이 포함되어 있습니다.

<figure><img src=".gitbook/assets/t4-r50_2.jpg" alt=""><figcaption><p>MAPIR T4-R50</p></figcaption></figure>

T4 확산 참조 타겟에는 다음과 같은 반사율 곡선이 있습니다. [여기서 데이터 다운로드](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (250-2500nm).png" alt=""><figcaption><p>MAPIR T4 반사율:: 250-2500nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (400-1000nm).png" alt=""><figcaption><p>MAPIR T4 반사율:: 400-1000nm</p></figcaption></figure>

반사율 그래프를 보면 값이 파장(x축) 대 반사율 백분율(y축)임을 알 수 있습니다. 교정 대상의 이미지를 캡처하면 카메라의 각 센서 밴드가 민감한 스펙트럼 내에서 픽셀 값과 반사율 비율 사이의 관계를 생성합니다.

즉, 당사 카메라로 캡처한 모든 이미지에 대해 [T4-R50](https://www.mapir.camera/collections/calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t3-r50) 또는 [T4-R125](https://www.mapir.camera/collections/multispectral-reflectance-reference-calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t4-r125)과 같은 반사 대상의 사진을 사용하여 이미지의 반사율을 보정할 수 있습니다. 일단 보정되면 이미지의 각 픽셀은 반사율 백분율과 같습니다.

Chloros에서 보정된 이미지를 일반적인 JPG 또는 TIFF로 출력하는 경우 반사율 백분율은 픽셀 값을 이미지 형식의 비트 심도로 나누어 계산됩니다. 따라서 JPG의 경우 255로 나누고 TIFF의 경우 65,535로 나눕니다. Chloros에서 PERCENT 형식 출력을 선택할 수도 있으며, 그러면 각 픽셀의 범위는 0.0~1.0(0%~100% 반사율)의 백분율 값입니다. 일부 이미지 응용 프로그램은 퍼센트(부동 소수점) 이미지를 허용할 수 없으며 저장 용량이 크다는 점을 명심하세요.

<div><figure><img src=".gitbook/assets/t3-125.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_2.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_closed.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure></div>
