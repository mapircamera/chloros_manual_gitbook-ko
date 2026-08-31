---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/supported-cameras
---

# 지원되는 카메라

Chloros는 **모든 플랫폼**에서 두 가지 MAPIR 카메라 제품군의 영상을 처리합니다 (Windows, Linux amd64, Linux arm64/Jetson)에서 다음 두 가지 MAPIR 카메라 제품군의 영상을 처리합니다:

* **Survey3** — Survey3W(와이드) 및 Survey3N(나로우) 카메라. 입력: `RAW+JPG`.
* **LATTICE**— M3C 및 M3M 다중 스펙트럼 카메라 모듈. 입력: `.tif`/`.tiff` 캡처 데이터. LATTICE 카메라는 Chloros에서**실시간으로 제어**할 수도 있습니다. — GUI의 ‘카메라’ 탭(Windows) 또는 `chloros-cli lattice` / Python 및 SDK (Windows 및 Linux)를 통해**실시간 제어**가 가능합니다. 여기에는 동기화된 멀티 카메라 어레이도 포함됩니다. [LATTICE 가이드](lattice/)를 참조하십시오.

이 처리 파이프라인은 `.dng` 입력 파일도 지원합니다.

## Survey3

<table data-header-hidden><thead><tr><th width="156">제조사</th><th width="250">카메라 모델</th><th width="138">필터 모델</th><th width="187">이미지 유형</th></tr></thead><tbody><tr><td><strong>제조사</strong></td><td><strong>카메라 모델</strong></td><td><strong>필터 모델</strong></td><td><strong>이미지 유형</strong></td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>RGB</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>RGN</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>OCN</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>NGB</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>RE</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>NIR</td><td>RAW+JPG, JPG</td></tr></tbody></table>## LATTICE

LATTICE 제품군은 Sony IMX265 글로벌 셔터 센서(3.1 MP, 3.45 µm 픽셀)를 기반으로 한 모듈형 다중 스펙트럼 카메라 시스템입니다. 모든 카메라는 모델 문자열 형태로 고유 식별자를 저장합니다:

```
<sensor>-<lens>-F<filter>        e.g.  M3C-L41-FRGN,  M3M-L87-F550
```

Chloros는 이를 `LATT-` 접두사와 함께 표시하며(예: `LATT-M3M-L41-F550`), 이 모델 문자열은 센서 프로파일, 대역 레이아웃, 보정 등 후속 모든 과정을 자동으로 처리하므로 카메라별로 별도로 설정할 사항은 없습니다. 렌즈 번호는 **도(°) 단위의 수평 시야각**을 나타냅니다: `L41` = 협각 41°, `L87` = 광각 87°.

두 가지 센서 구성이 존재합니다:

| 구성 | 센서      | 필터 유형                           | 카메라당 대역 수                                                        |
| ------------- | ----------- | ------------------------------------- | ----------------------------------------------------------------------- |
| **M3C**       | 베이어 컬러 | 트리플 대역 통과                       | 단일 노출로 3개의 스펙트럼 대역                                 |
| **M3M**       | 단색    | 단일 협대역 간섭 필터 | 보정된 1개 대역 — 식생 지수 산출을 위해 여러 대의 M3M 카메라를 결합 |

### M3C (Bayer) 필터 옵션

| 필터 | 대역 (중심 파장 @ nm / FWHM nm)       |
| ------ | ---------------------------------------- |
| `FRGB` | Blue 475/30 · Green 550/30 · Red 625/30  |
| `FRGN` | Red 660/21 · Green 550/30 · NIR 850/30   |
| `FOCN` | Orange 615/21 · Cyan 490/38 · NIR 808/14 |
| `FNGB` | Blue 475/30 · Green 550/30 · NIR 850/30  |

### M3M (모노) 필터 카탈로그 — 23개 SKU

F-번호는 SKU 라벨이며, 측정된 대역폭(교정된 모든 수출품에 각인됨)은 로트별 필터 스캔 결과입니다:

| SKU    | 중심 (nm, 측정값) | FWHM 에지 (nm) | 폭 (nm) |
| ------ | --------------------- | --------------- | ---------- |
| F385   | 379.4                 | 367–392         | 25         |
| F405   | 403.9                 | 390–417         | 27         |
| F450   | 443.7                 | 430–458         | 28         |
| F485   | 489.7                 | 478–502         | 24         |
| F520   | 519.9                 | 504–536         | 32         |
| F550   | 548.4                 | 531–566         | 35         |
| F590   | 589.0                 | 570–608         | 38         |
| F615   | 623.8                 | 614–634         | 20         |
| F632   | 633.4                 | 616–651         | 35         |
| F650   | 651.1                 | 636–666         | 30         |
| F685   | 686.2                 | 675–698         | 23         |
| F715   | — (명목상)           | 706–724         | 18         |
| F725   | 725.2                 | 712–738         | 26         |
| F750   | 746.0                 | 729–763         | 34         |
| F780   | 775.1                 | 754–796         | 42         |
| F808   | 810.3                 | 789–832         | 43         |
| F832   | 826.1                 | 810–843         | 33         |
| F850   | 846.5                 | 828–865         | 37         |
| F880   | — (명목상)           | 867–893         | 26         |
| F905   | — (명목상)           | 892–920         | 28         |
| F940   | 940.6                 | 923–958         | 35         |
| F950   | 945.1                 | 929–961         | 32         |
| F988 † | 985.3                 | 968–1003        | 35         |

_&quot;밴드 에지는 MAPIR의 로트별 필터 스캔 결과에서 측정된 반폭 최대값(FWHM)이며, 이는 Chloros가 모든 보정된 내보내기 파일에 기록하는 값과 동일합니다.&quot;_ &quot;— (공칭)&quot; = 아직 로트별 스캔이 수행되지 않음; 해당 SKU의 경우 명시된 중심은 SKU 번호이며, 폭은 제조업체가 제공한 수치입니다.

† &quot;F988 반사율은 현장 반사율 패널을 사용하여 보정됩니다. 해당 밴드는 DAQ 광 센서의 보정 범위를 벗어남에 따라, Chloros는 사용자의 가장 최근 패널 캡처 값을 적용하며 패널 관측 사이에는 해당 값을 유지합니다.&quot; [보정 타겟](calibration-targets.md)을 참조하십시오.

실시간 카메라 제어, 어레이, 네트워크 설정 및 방사측정 처리 체인에 대해서는 [LATTICE 가이드](lattice/)를 참조하십시오.
