# Linux 개요

Chloros 1.2.0은 **CLI**및**Python SDK** — 헤드리스 다중 스펙트럼 이미지 처리 및 실시간 LATTICE 카메라와 DAQ 광센서 제어 — 에 대한 네이티브 지원을 제공합니다.

{% hint style="info" %}
**Linux에는 데스크톱 GUI가 없습니다.**Chloros 데스크톱 GUI는 Windows에서만 사용할 수 있습니다. Linux 사용자는 [CLI](../CLI.md) 및 [Python SDK](../api-python-sdk.md)를 통해 Chloros와 상호 작용합니다. `.deb`는**Chloros CLI** 항목을 추가하는 것이 아니라, 단순히 `chloros-cli`를 실행하는 터미널 에뮬레이터를 열 뿐입니다.
{% endhint %}

***

## 플랫폼 지원 매트릭스

| 기능 | Windows (GUI) | Windows (CLI/SDK) | Linux amd64 (CLI/SDK) | Linux arm64 / Jetson (CLI/SDK) |
| --- | --- | --- | --- | --- |
| **데스크톱 GUI** | 예 | 해당 없음 | 아니요 | 아니요 |
| **CLI** (`chloros-cli`) | 예 | 예 | 예 | 예 |
| **Python SDK** (`chloros-sdk`) | 예 | 예 | 예 | 예 |
| **이미지 처리 파이프라인** | 예 | 예 | 예 | 예 |
| **LATTICE 카메라 제어 (실시간)** | 예 (카메라 탭) | 예 (`chloros-cli lattice`, SDK) | 예 | 예 |
| **DAQ 광센서 (실시간)** | 예 (광센서 탭) | 예 (`chloros-cli daq pool-*`, SDK) | 예 | 예 |
| **PTP 시간 동기화(호스트가 그랜드마스터)** | 예 | 예 (`chloros-cli time-sync`) | 예 | 예 |
| **GPU 가속(CUDA)** | 예 | 예 | 예 | 예 (JetPack 6) |
| **텍스처 인식 디베이어** | 예 (Chloros+) | 예 (Chloros+) | 예 (Chloros+) | 예 (Chloros+) |
| **동적 컴퓨트 적응** | 예 | 예 | 예 | 예 |
| **시스템 서비스로서의 백엔드** (`chloros-backend.service`) | 아니요 | 아니요 | 예 (선택적 활성화) | 예 (선택적 활성화) |
| **인플레이스 업데이트** (`chloros-cli update`) | 아니요 (설치 프로그램 실행) | 아니요 (설치 프로그램 실행) | 예 | 예 |***

## 지원되는 아키텍처

| 아키텍처 | 설명 | 패키지 |
| --- | --- | --- |
| **amd64 (x86_64)** | 표준 데스크톱/서버 프로세서 (Intel, AMD) | `chloros_<version>_amd64.deb` |
| **arm64 (aarch64)** | ARM 프로세서 — NVIDIA Jetson Orin 제품군 | `chloros_<version>_arm64_jp6.deb` (JetPack 6 빌드) |

## 지원되는 Linux 배포판

* **Ubuntu 22.04 LTS 이상** (amd64)
* **Debian 12 이상** (amd64)
* **NVIDIA JetPack 6** (arm64 — Jetson Orin 플랫폼)***

## Linux 사용자가 얻을 수 있는 기능

* **Chloros CLI** — 일괄 처리, 자동화 및 스크립팅을 위한 완전한 명령줄 인터페이스
* **Chloros Python SDK** — 연구 파이프라인 및 사용자 정의 도구를 위한 프로그래밍 방식의 Python 인터페이스(PyPI에서 설치 가능하며, 버전 호환 휠 형태로 `.deb` 내에 번들로 포함됨)
* **LATTICE 카메라 제어** — `chloros-cli lattice` 및 SDK를 통해 LATTICE 카메라 및 동기화된 멀티 카메라 어레이를 탐색, 연결, 구성하고 영상을 캡처할 수 있습니다; `.deb`에는 카메라에 필요한 Arena SDK 런타임이 번들로 포함되어 있습니다.
* **DAQ 광센서 제어** — `chloros-cli daq pool-*` 및 SDK를 통해 DAQ-U/M/E 센서를 연결하고, 보정된 스펙트럼을 스트리밍하며, `.daq` 파일을 기록할 수 있습니다.
* **PTP 시간 동기화** — Chloros 백엔드는 LATTICE 카메라와 DAQ-E 센서가 슬레이브로 연결되는 PTP 그랜드마스터를 실행합니다. `chloros-cli time-sync`를 사용하여 이를 확인하고, `chloros-backend.service` systemd 유닛을 사용하여 헤드리스 모드로 계속 실행하십시오([Linux 설치](linux-installation.md#always-on-ptp-for-headless-hosts) 참조).
* **프로젝트 자동화** — `chloros-cli project` 및 SDK의 `open_project`를 사용하여 저장된 프로젝트를 헤드리스 모드로 실행합니다.
* **GPU 가속** — NVIDIA GPU(데스크톱 및 Jetson)에서 CUDA 가속 처리
* **동적 컴퓨팅 적응** — 자동 하드웨어 감지 및 처리 전략 선택, 전문가용 비상 탈출구로 `CHLOROS_STRATEGY` 오버라이드 제공
* **모든 처리 기능** — Windows와 동일한 파이프라인: 보정, 비네팅 보정, 식생 지수, 모든 내보내기 형식
* **Chloros+ 기능** — 멀티스레드(파이프라인) 처리, 텍스처 인식 디베이어, 사용자 정의 지수 지원(유료 Chloros+ 플랜 적용 시)

## Linux 사용자가 이용할 수 없는 기능

* **데스크톱 GUI** — 그래픽 인터페이스가 없으며, 모든 상호 작용은 CLI 또는 Python SDK를 통해 이루어집니다
* **이미지 뷰어** — 대화형 이미지 뷰어, 그리드 보기 또는 지도 마커가 없음
* **시각적 프로젝트 관리** — 프로젝트는 CLI 명령어와 SDK 호출을 통해 생성 및 운영됩니다(카메라, 센서, 캡처 등 하드웨어 자체는 터미널에서 완전히 제어할 수 있습니다)***

## 라이선스 요구 사항

CLI 및 SDK에 액세스하려면 **유료 Chloros+ 등급 — Copper 이상**(Copper, Bronze, Silver, Gold)이 필요합니다. 무료**Iron** 등급에서는 CLI/SDK에 접근할 수 없습니다. 이 제한은 CLI뿐만 아니라 백엔드에서도 적용됩니다:

| 상황 | 백엔드 응답 |
| --- | --- |
| 로그인하지 않은 상태 | `401` (`error_code: AUTH_REQUIRED` 포함) |
| 무료 Iron 티어에 로그인됨 | `403` (`error_code: AUTH_REQUIRED` 포함) |

`chloros-cli status`는 모든 티어에서 작동합니다(게이트 적용 대상에서 제외된 유일한 경로이므로) 따라서 거부 사유는 항상 표시됩니다.

***

## Linux 시작하기

1. **Chloros 설치** — `.deb` 설치 방법은 [Linux 설치](linux-installation.md)를 참조하십시오
2. **확인** — `chloros-cli --version`는 `Chloros CLI 1.2.0`를 출력하며, `chloros-cli selftest`는 7단계 진단 절차를 실행합니다
3. **Python 및 SDK 설치** (선택 사항) — `pip install chloros-sdk`
4. **로그인** — `chloros-cli login your@email.com 'your-password'` (기기당 한 번, 그리고 패키지 업그레이드 후 매번 다시 수행)
5. **첫 번째 데이터셋 처리** — `chloros-cli process ~/datasets/flight001`

NVIDIA Jetson의 경우, 플랫폼별 설정, 열적 특성 및 현장 배포에 대해서는 전용 [NVIDIA Jetson 가이드](nvidia-jetson-guide.md)를 참조하십시오.

***

## 다음 단계

* [Linux 설치](linux-installation.md) — amd64 및 arm64에 대한 자세한 설치 방법, 파일 위치 및 문제 해결
* [NVIDIA Jetson 가이드](nvidia-jetson-guide.md) — Jetson 전용 설정, 메모리 및 열 동작, 현장 배포
* [CLI : 명령줄](../CLI.md) — CLI 가이드
* [API : Python SDK](../api-python-sdk.md) — SDK 가이드
* [CLI 참조](../reference/cli-reference.md) 및 [SDK 참조](../reference/sdk-reference.md) — 1.2.0 버전에 대한 상세한 명령어/API 목록
* [동적 컴퓨팅 적응](../processing-architecture/dynamic-compute-adaptation.md) — Chloros가 사용자의 하드웨어에 어떻게 적응하는지

{% hint style="info" %}
**이 설명서를 프로그래밍 방식으로 읽기.** 모든 페이지는 각 페이지의 URL 및 `.md` (예: `https://mapir.gitbook.io/chloros/linux/linux-installation.md`)에서 원시 마크다운 형식으로 제공되며, 전체 설명서의 색인은 [`https://mapir.gitbook.io/chloros/llms.txt`](https://mapir.gitbook.io/chloros/llms.txt)에 게시되어 있습니다.
{% endhint %}
