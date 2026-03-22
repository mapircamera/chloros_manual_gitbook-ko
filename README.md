---
metaLinks: {}
---

# 시작하기

<div data-full-width="false"><figure><img src=".gitbook/assets/chloros_logo_transparent.png" alt=""><figcaption></figcaption></figure></div>Chloros는 [MAPIR](https://www.mapir.camera)에서 제공하는 이미지 및 기타 센서 데이터를 처리하는 소프트웨어 애플리케이션입니다.

***{% hint style="success" %}**Chloros 1.1.0의 새로운 기능**: 네이티브 Linux 지원(amd64 및 arm64), NVIDIA Jetson 엣지 컴퓨팅, 동적 컴퓨트 적응(Dynamic Compute Adaptation), 4스레드 처리 파이프라인, 새로운 CLI 명령어 및 옵션. 전체 변경 내역은 [다운로드](download.md)를 참조하십시오.
{% endhint %}

Chloros는 다음 3가지 애플리케이션 모드로 제공됩니다:

## Chloros: 데스크톱 GUI 애플리케이션

모든 기능을 갖춘 독립형 별도 창. _Windows 전용._

## [Chloros CLI: 명령줄 인터페이스](CLI.md)

명령줄 일괄 처리. 자동화, 스크립팅 및 헤드리스(headless) 작업에 적합합니다. **Windows, Linux amd64 및 Linux arm64 (NVIDIA Jetson)**에서 사용 가능합니다. _CLI를 사용하려면 Chloros 이상 라이선스가 필요합니다._

## [Chloros API: Python SDK](api-python-sdk.md)

자동화 및 사용자 지정 워크플로우를 위한 프로그래밍 방식의 Python 인터페이스입니다. 연구 파이프라인, 기존 Python 애플리케이션과의 통합, 사용자 지정 도구 구축에 이상적입니다. `pip install chloros-sdk`를 통해 **모든 플랫폼**에서 사용할 수 있습니다. _API를 사용하려면 Chloros+ 라이선스가 필요합니다._***

## 지원 플랫폼

| 플랫폼 | GUI | CLI | Python SDK |
| --- | --- | --- | --- |
| **Windows 10/11** | 예 | 예 | 예 |
| **Linux amd64 (x86_64)** | 아니요 | 예 | 예 |
| **Linux arm64 (NVIDIA Jetson)** | 아니요 | 예 | 예 |

Linux 설치 지침은 [Linux 및 엣지 컴퓨팅](linux/linux-overview.md) 섹션을 참조하십시오.

***

## Chloros+

Chloros는 대부분의 작업에 무료로 사용할 수 있지만, 더 많은 기능이 필요할 수도 있습니다. 이때 Chloros+ 유료 라이선스가 유용할 수 있습니다. Chloros+ 라이선스를 사용하면 다음과 같은 새로운 기능을 이용할 수 있습니다:

* **멀티스레드 처리**: 파이프라인을 통해 이미지를 동시에 처리하여 대규모 프로젝트의 이미지 처리 속도를 대폭 향상시킵니다.
* **GPU (CUDA) 가속**: 최신 고사양 GPU 메모리를 활용하여 이미지 처리 파이프라인의 속도를 한층 더 높일 수 있습니다. 최상의 결과를 얻으려면 4GB 이상의 VRAM을 권장합니다.
* **Chloros+**[**CLI**](CLI.md)**액세스**: 명령줄에서 Chloros+를 실행하여 자동화하고 사용자만의 소프트웨어에 통합할 수 있습니다.
* **Chloros+**[**API**](api-python-sdk.md)**접근:** Python에서 Chloros+를 실행하여 프로그래밍 방식으로 제어함으로써, 연구 파이프라인, 데이터 분석 워크플로우 및 사용자 지정 애플리케이션과 원활하게 통합할 수 있습니다.
* **다중 기기 사용**: 각 Chloros+ 라이선스로 2대 이상의 기기를 등록할 수 있습니다. MAPIR 클라우드 계정을 사용하여 등록된 기기를 관리하십시오. Chloros+ 라이선스를 업그레이드하여 더 많은 기기를 지원하십시오.
* **고급 텍스처 인식 디베이어링 방식:** 고품질의 에지 인식 디베이어링 기술과 AI/ML 노이즈 제거 모델을 결합하여 디베이어링 과정에서 발생하는 노이즈를 거의 완벽하게 제거합니다. 
* **사용자 정의 다중 스펙트럼 지수 공식:** Chloros 래스터 계산기에 사용자 정의 다중 스펙트럼 지수를 입력하여, 이미지 처리 및 이미지 뷰어 샌드박스에서 활용할 수 있습니다.
* **Linux 및 엣지 컴퓨팅:** 현장 및 엣지 처리를 위해 NVIDIA Jetson을 포함한 Linux x86_64 및 ARM64 플랫폼에서 Chloros를 실행할 수 있습니다. [Linux 개요](linux/linux-overview.md)를 참조하십시오.

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary" data-icon="envira">Chloros+ 가격 및 가입</a></p>

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_zoom.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_mode.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_meta.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
