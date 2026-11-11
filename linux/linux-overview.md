# Linux 개요

Chloros 1.3.0은 Linux 워크스테이션, 서버 및 NVIDIA Jetson 엣지 장치에서 **CLI**및**Python SDK**에 대한 네이티브 Linux 지원을 제공합니다. 여기에는 헤드리스 다중 스펙트럼 이미지 처리와 실시간 LATTICE 카메라 및 DAQ 광센서 제어가 포함됩니다.

{% hint style="info" %}
**Linux에서는 데스크톱 GUI를 사용할 수 없습니다.**Chloros 데스크톱 GUI는 Windows 전용입니다. Linux 사용자는 [CLI](../CLI.md) 및 [Python SDK](../api-python-sdk.md)를 통해 Chloros와 상호작용합니다. `.deb`는 애플리케이션 메뉴에**Chloros CLI** 항목을 추가합니다. 이 항목은 단순히 `chloros-cli`를 실행하는 터미널 에뮬레이터를 열 뿐입니다.
{% endhint %}

***

## 플랫폼 지원 매트릭스

| 기능 | Windows (GUI) | Windows (CLI/SDK) | Linux amd64 (CLI/ --- | --- | --- | --- |
| **데스크톱 GUI** | 예 | 해당 없음 | 아니요 | 아니요 |
| **CLI** (`chloros-cli`) | 예 | 예 | 예 | 예 |
| **Python SDK** (`chloros-sdk`) | 예 | 예 | 예 | 예 |
| **이미지 처리 파이프라인** | 예 | 예 | 예 | 예 |
| **LATTICE 카메라 제어 (실시간)** | 예 (카메라 탭) | 예 (`chloros-cli lattice`, SDK) | 예 | 예 |
| **DAQ 광 센서 (실시간)** | 예 (광 센서 탭) | 예 (`chloros-cli daq pool-*`, SDK) | 예 | 예 |
| **PTP 시간 동기화 (호스트가 그랜드마스터)** | 예 | 예 (`chloros-cli time-sync`) | 예 | 예 |
| **GPU 가속 (CUDA)** | 예 | 예 | 예 | 예 (JetPack 6) |
| **텍스처 인식 디베이어** (CUDA GPU 필요) | 예 (Chloros+) | 예 (Chloros+) | 예 (Chloros+) | 예 (Chloros+) |
| **동적 컴퓨트 적응** | 예 | 예 | 예 | 예 |
| **시스템 서비스로서의 백엔드** (`chloros-backend@<user>.service`) | 아니요 | 아니요 | 예 (선택적 적용) | 예 (선택적 적용) |
| **인플레이스 업데이트 기능** (`chloros-cli update`) | 아니요 (설치 프로그램 실행) | 아니요 (설치 프로그램 실행) | 예 | 예 |***

## 지원되는 아키텍처

| 아키텍처 | 설명 | 패키지 |
| --- | --- | --- |
| **amd64 (x86_64)** | 표준 데스크톱/서버 프로세서 (Intel, AMD) | `chloros_<version>_amd64.deb` |
| **arm64 (aarch64)** | ARM 프로세서 — NVIDIA Jetson Orin 제품군 (JetPack 6) | `chloros_<version>_arm64_jp6.deb` |

## 지원되는 Linux 배포판

* **Ubuntu 22.04 LTS 이상** (amd64)
* **Debian 12 이상** (amd64)
* **NVIDIA JetPack 6** (arm64 — Jetson Orin 플랫폼)***

## Linux 사용자가 얻을 수 있는 것

* **Chloros CLI** — 일괄 처리, 자동화 및 스크립팅을 위한 완전한 명령줄 인터페이스
* **Chloros Python SDK** — 연구 파이프라인 및 사용자 지정 도구를 위한 프로그래밍용 Python 인터페이스. 시스템 `python3`용 `.deb`를 통해 설치됩니다 (virtualenv용 버전 호환 휠은 `/usr/lib/chloros/sdk/`에 포함되어 있으며, PyPI는 Chloros 패키지가 없는 시스템용입니다)
* **LATTICE 카메라 제어** — `chloros-cli lattice` 및 SDK를 통해 LATTICE 카메라와 동기화된 다중 카메라 어레이를 탐색, 연결, 구성 및 촬영할 수 있습니다. `.deb`에는 카메라에 필요한 Arena SDK 런타임이 포함되어 있습니다
* **DAQ 광센서 제어** — `chloros-cli daq pool-*` 및 SDK를 통해 DAQ-U/M/E 센서를 연결하고, 보정된 스펙트럼을 스트리밍하며, `.daq` 파일을 기록합니다
* **PTP 시간 동기화** — Chloros 백엔드는 LATTICE 카메라와 DAQ-E 센서가 슬레이브로 연결되는 PTP 그랜드마스터를 실행하며; `chloros-cli time-sync`를 사용하여 이를 점검하고, `chloros-backend@<user>.service` systemd 유닛을 통해 헤드리스 모드로 계속 실행합니다([Linux 설치](linux-installation.md#always-on-ptp-for-headless-hosts) 참조)
* **프로젝트 자동화** — `chloros-cli project` 및 SDK의 `open_project`를 사용하여 저장된 프로젝트를 헤드리스 모드로 실행합니다
* **GPU 가속** — NVIDIA GPU에서 CUDA 가속 처리 지원(amd64의 데스크톱 GPU 및 JetPack 6이 설치된 Jetson의 GPU)
* **동적 컴퓨팅 적응** — 자동 하드웨어 감지 및 처리 전략 선택, 전문가용 비상 탈출구로 `CHLOROS_STRATEGY` 오버라이드 기능 제공
* **모든 처리 기능** — Windows와 동일한 파이프라인: 보정, 비네팅 보정, 식생 지수, 모든 내보내기 형식
* **Chloros+ 기능** — 멀티스레드(파이프라인) 처리, 텍스처 인식 디베이어(CUDA GPU에서), 사용자 정의 지수(유료 Chloros+ 플랜 적용 시)

## Linux 사용자가 이용할 수 없는 기능

* **데스크톱 GUI** — 그래픽 인터페이스가 없으며, 모든 상호 작용은 CLI 또는 Python SDK를 통해 이루어집니다
* **이미지 뷰어** — 대화형 이미지 뷰어, 그리드 뷰 또는 지도 마커 기능이 없습니다
* **시각적 프로젝트 관리** — 프로젝트는 CLI 명령어와 SDK 호출을 통해 생성 및 관리됩니다(카메라, 센서, 캡처 등 하드웨어 자체는 터미널에서 완전히 제어할 수 있습니다)***

## 라이선스 요구 사항

CLI 및 SDK에 접근하려면 **유료 Chloros+ 요금제 — Copper 이상**(Copper, Bronze, Silver, Gold)이 필요합니다. 무료**Iron** 요금제에서는 CLI/SDK에 접근할 수 없습니다. 이 제한은 CLI뿐만 아니라 백엔드에서도 적용됩니다:

| 상황 | 백엔드 응답 |
| --- | --- |
| 로그인하지 않은 상태 | `401` (`error_code: AUTH_REQUIRED` 포함) |
| 무료 Iron 티어에 로그인됨 | `403` 및 `error_code: PLAN_UPGRADE_REQUIRED` |

`chloros-cli status`는 모든 티어에서 작동합니다(게이트에서 제외되는 유일한 경로이기 때문). 따라서 거부 사유가 항상 표시됩니다.

***

## Linux에서 시작하기 Linux

1. **Chloros 설치** — `.deb` 설치 방법은 [Linux 설치 안내](linux-installation.md)를 참조하세요
2. **확인** — `chloros-cli --version`는 `Chloros CLI 1.3.0`를 출력하며, `chloros-cli selftest`는 7단계 진단 절차를 실행합니다
3. **Python SDK 확인** — `.deb`가 이를 설치합니다: `python3 -c "import chloros_sdk; print(chloros_sdk.__version__)"` (virtualenv 내에서는 `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl`)
4. **로그인** — `chloros-cli login your@email.com 'your-password'` (기기당 한 번; 패키지 업그레이드 시 로그인 정보가 유지되며, 패키지 제거 시 로그인 정보가 삭제됩니다)
5. **첫 번째 데이터셋 처리** — `chloros-cli process ~/datasets/flight001`

NVIDIA Jetson의 경우, 플랫폼별 설정, 열 관리 및 현장 배포에 대해서는 전용 [NVIDIA Jetson 가이드](nvidia-jetson-guide.md)를 참조하십시오.

***

## 다음 단계

* [Linux 설치](linux-installation.md) — amd64 및 arm64에 대한 자세한 설치 방법, 파일 위치 및 문제 해결
* [NVIDIA Jetson 가이드](nvidia-jetson-guide.md) — Jetson 전용 설정, 메모리 및 열 동작, 현장 배포
* [CLI : 명령줄](../CLI.md) — CLI 가이드
* [API : Python SDK](../api-python-sdk.md) — SDK 가이드
* [CLI 참조](../reference/cli-reference.md) 및 [SDK 참조](../reference/sdk-reference.md) — 1.3.0 버전의 모든 명령어/API 목록
* [동적 컴퓨팅 적응](../processing-architecture/dynamic-compute-adaptation.md) — Chloros가 사용자의 하드웨어에 어떻게 적응하는지

{% hint style="info" %}
**이 설명서를 프로그래밍 방식으로 읽는 방법.** 모든 페이지는 각 페이지의 고유 URL에 `.md`를 더한 형태(예: `https://mapir.gitbook.io/chloros/linux/linux-installation.md`)로 원시 마크다운(raw Markdown) 형식으로 제공되며, 전체 설명서의 목차는 [`https://mapir.gitbook.io/chloros/llms.txt`](https://mapir.gitbook.io/chloros/llms.txt).
{% endhint %}
