---
description: Lab-measured panels used to calibrate captured data in post processing
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/calibration-targets
---

# 교정 타겟

MAPIR는 다양한 응용 분야를 아우르는 여러 종류의 교정 타겟을 제공합니다. 아래에 보이는 소형 T4-R50 모델은 250~2,500 nm 범위의 광 반사율이 측정된 4개의 패널로 구성되어 있습니다.

<figure><img src=".gitbook/assets/t4-r50_2.jpg" alt=""><figcaption><p>MAPIR T4-R50</p></figcaption></figure>T4 확산 기준 타겟의 반사율 곡선은 다음과 같습니다. [데이터 다운로드: 여기](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (250-2500nm).png" alt=""><figcaption><p>MAPIR T4 반사율 :: 250-2,500nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (400-1000nm).png" alt=""><figcaption><p>MAPIR T4 반사율 :: 400-1,000nm</p></figcaption></figure>T4P 확산 기준 타겟의 반사율 곡선은 다음과 같으며, [데이터는 여기에서 다운로드할 수 있습니다](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4P -- 350-2500nm.jpg" alt=""><figcaption><p>MAPIR T4P 반사율 :: 250-2500nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4P -- 400-1000nm.jpg" alt=""><figcaption><p>MAPIR T4P 반사율 :: 400-1000nm</p></figcaption></figure>반사율 그래프를 보면, 수치가 파장(x축) 대 반사율(y축)의 비율로 표시되어 있음을 확인할 수 있습니다. 교정 타겟의 이미지를 촬영하면, 카메라 센서의 각 밴드가 감지할 수 있는 스펙트럼 범위 내에서 픽셀 값과 반사율 간의 상관관계를 설정하게 됩니다.

즉, 당사 카메라로 촬영한 모든 이미지에 대해 [T4-R50](https://www.mapir.camera/collections/calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t3-r50)나 [T4-R125](https://www.mapir.camera/collections/multispectral-reflectance-reference-calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t4-r125)와 같은 당사의 반사율 타겟 사진을 사용하여 이미지의 반사율을 보정할 수 있습니다. 보정이 완료되면 이미지의 각 픽셀 값은 반사율 백분율과 일치하게 됩니다.

**Survey3** 출력의 경우, 보정된 이미지를 일반적인 JPG 형식(Chloros)이나 TIFF 형식으로 출력할 때, 반사율(%)은 픽셀 값을 이미지 형식의 비트 심도로 나누어 계산됩니다. 따라서 JPG의 경우 255로 나누고, TIFF의 경우 65,535로 나눕니다. 또한 Chloros에서 PERCENT 형식 출력을 선택하면 각 픽셀의 값이 0.0에서 1.0 사이의 백분율 값(반사율 0%에서 100%)으로 표시됩니다. 다만 일부 이미지 응용 프로그램은 백분율(부동 소수점) 형식의 이미지를 지원하지 않을 수 있으며, 이러한 이미지는 저장 공간 측면에서 용량이 크다는 점을 유의하시기 바랍니다.

{% hint style="info" %}
**LATTICE 반사율은 다른 픽셀 스케일을 사용합니다.** LATTICE 반사율은 DN 32768 = 100% 반사율(65535가 아님)로 저장되며, 모든 파일에는 해당 스케일을 명시하는 XMP `Chloros:PixelScale` 태그가 포함되어 있습니다. 상수를 가정하기보다는 태그를 확인한 후 그 값으로 나누십시오 — [출력 이미지 형식](output-image-formats.md)을 참조하십시오.
{% endhint %}

## LATTICE 카메라용 보정 타겟

LATTICE 카메라를 사용할 경우 반사율 측정을 위한 보정 타겟은 **선택 사항**입니다. 대신 Chloros를 사용하여 DAQ 광 센서로 측정된 하향 복사조도(ρ = π·L/E)를 기준으로 반사율을 참조할 수 있습니다. 이 기준은 반사율 소스 설정에서 선택됩니다 (GUI의 ‘프로젝트 설정’; `--reflectance-source`는 CLI에서, `reflectance_source`는 SDK에서 선택):

| 값 | 동작 |
| --- | --- |
| `auto` *(기본값)* | QA를 통과한 프레임 내 타겟이 **절대 기준**이 됩니다. 타겟이 없거나 QA에 실패한 경우, Chloros는 DAQ 하향 분할값으로 대체됩니다. |
| `target` | 엄격한 타겟 전용 — DAQ 대체 없음. |
| `daq` | DAQ 우선 — 다운웰링 측정값이 항상 기준이 됩니다. |

LATTICE용 추가 타겟 동작:

* **타겟 형상** — ArUco 마킹 패널, 고정 ROI 패널 및 스트립 타겟이 모두 지원되며, 형상은 프로젝트의 타겟 구성에서 가져옵니다.
* **단위별 측정된 타겟 데이터** — `--target-reflectance-dir DIR`는 단위별 측정된 타겟 반사율 스캔 데이터가 포함된 디렉터리를 가리킵니다(`<serial>.csv`, 타겟 단위의 일련번호/QR을 통해 조회됨). 타겟을 탐지하지 못한 경우, Chloros는 공칭 T3/T4P 스펙트럼으로 대체됩니다.
* **시간적 고정** — 탐지된 타겟은 주변 프레임을 보정하며, 타겟 탐지 간에도 유지됩니다.

전체 플래그 의미 및 예시는 [CLI 참조 문서](reference/cli-reference.md)에 나와 있습니다(“제품별 내보내기 토글” 참조).

### F988

&quot;F988 반사율은 장면 내 반사율 패널을 사용하여 보정됩니다. 해당 대역은 DAQ 광 센서의 보정 범위를 벗어나므로, Chloros는 가장 최근의 패널 캡처 데이터를 적용하고 패널 관측 사이에도 이를 유지합니다.&quot;

F988을 DAQ 전용 보정 모드로 실행하는 경우, Chloros는 해당 대역에 대한 DAQ 기반 반사율을 거부하고 그 이유를 표시합니다(건너뛰기 사유 `dls-uncalibrated-band-988`); 패널 워크플로가 지원되는 경로입니다.

<div><figure><img src=".gitbook/assets/t3-125.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_2.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_closed.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure></div>
