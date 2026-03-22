# Linux 개요

Chloros 1.1.0은 **CLI**및**Python SDK**에 대한 네이티브 지원을 제공하여, Linux 워크스테이션, 서버 및 NVIDIA Jetson 엣지 디바이스에서 헤드리스 다중 스펙트럼 이미지 처리가 가능해졌습니다.

{% hint style="info" %}
**Linux에는 GUI가 없습니다.** Chloros 데스크톱 GUI는 Windows에서만 사용할 수 있습니다. Linux 사용자는 [CLI](../CLI.md) 및 [Python SDK](../api-python-sdk.md)를 통해 상호 작용합니다.
{% endhint %}

***

## 플랫폼 지원 매트릭스

| 기능 | Windows (GUI) | Windows (CLI/SDK) | Linux amd64 (CLI/SDK) | Linux arm64 / Jetson (CLI/SDK) |
| --- | --- | --- | --- | --- |
| **데스크톱 GUI** | 예 | 해당 없음 | 아니요 | 아니요 |
| **CLI** | 예 | 예 | 예 | 예 |
| **Python SDK** | 예 | 예 | 예 | 예 |
| **GPU 가속 (CUDA)** | 예 | 예 | 예 | 예 (JetPack 6) |
| **텍스처 인식 디베이어** | 예 (Chloros+) | 예 (Chloros+) | 예 (Chloros+) | 예 (Chloros+) |
| **동적 컴퓨트 적응** | 예 | 예 | 예 | 예 |***

## 지원 아키텍처

| 아키텍처 | 설명 | 설치 방법 |
| --- | --- | --- |
| **amd64 (x86_64)** | 표준 데스크톱/서버 프로세서 (Intel, AMD) | `.deb` 패키지 |
| **arm64 (aarch64)** | ARM 기반 프로세서, 주로 NVIDIA Jetson | `.deb` 패키지 (JetPack 6) |

## 지원되는 Linux 배포판

* **Ubuntu 20.04 이상** (amd64)
* **Debian 11 이상** (amd64)
* **NVIDIA JetPack 6** (arm64 — Jetson 플랫폼)***

## Linux 사용자가 얻을 수 있는 것

* **Chloros CLI** — 일괄 처리, 자동화 및 스크립팅을 위한 완전한 명령줄 인터페이스
* **Chloros Python SDK** — 연구 파이프라인 및 사용자 지정 도구 통합을 위한 프로그래밍 방식 Python 인터페이스 (`pip install chloros-sdk`)
* **GPU 가속** — NVIDIA GPU(데스크톱 및 Jetson)에서 CUDA 가속 처리
* **동적 컴퓨팅 적응** — 자동 하드웨어 감지 및 처리 전략 최적화
* **모든 처리 기능** — Windows와 동일한 다중 스펙트럼 처리 파이프라인(보정, 비네팅 보정, 식생 지수, 모든 내보내기 형식)
* **Chloros+ 기능** — 멀티스레드 처리, 텍스처 인식 디베이어, 사용자 정의 지수 (Chloros+ 라이선스 필요)

## Linux 사용자가 이용할 수 없는 기능

* **데스크톱 GUI** — 그래픽 인터페이스 없음; 모든 상호작용은 CLI, Python 또는 SDK를 통해 이루어짐
* **이미지 뷰어** — 대화형 이미지 뷰어, 그리드 보기 또는 지도 마커가 없습니다
* **시각적 프로젝트 관리** — 프로젝트는 CLI 명령어와 SDK 호출을 통해 관리됩니다***

## Linux 시작하기

1. **Chloros 설치** — `.deb` 패키지 설치 방법은 [Linux 설치](linux-installation.md)를 참조하십시오
2. **Python 및 SDK 설치** (선택 사항) — `pip install chloros-sdk`
3. **라이선스 활성화** — `chloros-cli login your@email.com 'password'`
4. **첫 번째 데이터셋 처리** — `chloros-cli process ~/datasets/flight001`

NVIDIA Jetson 사용자의 경우, 플랫폼별 설정 및 최적화에 대한 전용 [NVIDIA Jetson 가이드](nvidia-jetson-guide.md)를 참조하십시오.

***

## 다음 단계

* [Linux 설치](linux-installation.md) — amd64 및 arm64용 상세 설치 안내
* [NVIDIA Jetson 가이드](nvidia-jetson-guide.md) — Jetson 전용 설정, 열 관리 및 현장 배포
* [CLI : 명령줄](../CLI.md) — CLI 전체 참조
* [API : Python SDK](../api-python-sdk.md) — 전체 SDK 참조
* [동적 컴퓨팅 적응](../processing-architecture/dynamic-compute-adaptation.md) — Chloros가 사용자의 하드웨어에 어떻게 적응하는지
