---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/download
---

# 다운로드

다중 스펙트럼 이미지 처리를 시작하려면 Chloros의 최신 버전을 다운로드하세요.

### 시스템 요구 사항

#### Windows

| 요구 사항          | 최소                                              | 권장                                          |
| -------------------- | ---------------------------------------------------- | ---------------------------------------------------- |
| **운영 체제** | Windows 10 (64비트)                                  | Windows 11 (64비트)                                  |
| **프로세서**        | Intel Core i5 또는 동급                          | Intel Core i7 이상                              |
| **메모리 (RAM)**     | 8GB                                                  | 16GB 이상                                         |
| **그래픽 카드**    | DirectX 11 호환                                | 4GB 이상의 VRAM을 갖춘 NVIDIA GPU                            |
| **저장 공간**          | 6GB의 여유 공간                                       | 10GB 이상의 여유 공간이 있는 SSD                            |
| **디스플레이**          | 1920x1080                                            | 2560x1440 이상                                  |
| **인터넷**         | [선택 사항] Chloros+ 라이선스 활성화에 필요 | [선택 사항] Chloros+ 라이선스 활성화에 필요 |

#### Linux amd64 (x86_64)

| 요구 사항       | 최소                    | 권장               |
| ----------------- | -------------------------- | ------------------------- |
| **배포판**  | Ubuntu 20.04 이상 / Debian 11 이상 | Ubuntu 22.04 이상             |
| **프로세서**     | x86_64 (Intel/AMD)        | Intel Core i7 이상   |
| **메모리 (RAM)**  | 8GB                        | 16GB 이상              |
| **그래픽 카드** | 없음 (CPU 처리)      | 4GB 이상 VRAM 탑재 NVIDIA GPU |
| **저장 공간**       | 2GB 여유 공간             | 10GB 이상 여유 공간 있는 SSD       |
| **Python**        | Python 3.7 이상 (SDK용)      | Python 3.10 이상              |

#### Linux arm64 (NVIDIA Jetson)

| 요구 사항      | 최소                      | 권장                     |
| ---------------- | ---------------------------- | ------------------------------- |
| **플랫폼**     | JetPack 6이 설치된 NVIDIA Jetson | Jetson Orin NX 16GB 또는 AGX Orin |
| **메모리 (RAM)** | 8GB (GPU/CPU 공유)         | 16GB+ 공유                    |
| **저장 공간**      | 2GB 여유 공간               | 10GB+ 여유 공간이 있는 NVMe SSD        |
| **Python**       | Python 3.7+ (SDK용)        | Python 3.10+                    |

{% hint style="info" %}
**GPU 가속**: NVIDIA GPU를 사용하는 Chloros+ 사용자는 CUDA 가속을 통해 처리 속도를 대폭 높일 수 있습니다. 이 기능은 Windows(데스크톱 GPU)와 Linux(데스크톱 GPU 및 NVIDIA Jetson) 모두에서 작동합니다. Chloros+ 사용자는 최대 속도를 위한 멀티스레드 처리 기능도 이용할 수 있습니다.
{% endhint %}

***

## Chloros 다운로드

### 최신 안정화 버전 (2026년 3월 23일): 버전 1.1.0

### <a href="https://drive.google.com/uc?export=download&#x26;id=1HjwrUY4M7HGxDbMybO7iPe_6JoHnUGr4" class="button primary">Windows용 Chloros 다운로드 (.exe)</a>



### <a href="https://drive.google.com/uc?export=download&#x26;id=1dB8-ke3wxNXpw_e1qJ4BhwBpCoNd4kLS" class="button primary">Linux amd64용 Chloros 다운로드 (.deb)</a>



### <a href="https://drive.google.com/uc?export=download&#x26;id=1d1OwdcYA4Rf4jkuPi2IBeWT2772_HnyO" class="button primary">Linux arm64 / Jetson용 Chloros 다운로드 (.deb)</a>

#### Windows 설치 프로그램 (GUI + CLI + 백엔드)

* **파일 형식**: .exe (Windows 설치 프로그램)**설치 단계:**

1. 위의 .exe 파일을 다운로드합니다.
2. 설치 프로그램을 더블 클릭하여 설치를 시작합니다.
3. 설치 마법사의 안내에 따라 진행합니다.
4. 설치 디렉터리를 선택합니다(기본값: `C:\Program Files\[USER]\Chloros\`)
5. 설치를 완료하고 Chloros 또는 Chloros CLI를 실행합니다
6. [MAPIR Cloud Chloros+ 계정](https://cloud.mapir.camera/pricing)으로 로그인하십시오(또는 무료 버전으로 계속 진행하십시오)

{% hint style="success" %}
설치 프로그램이 명령줄에서 액세스할 수 있도록 시스템 PATH에 `chloros-cli`를 자동으로 추가합니다.
{% endhint %}

#### Linux amd64 (.deb 패키지 — CLI + 백엔드)

* **파일 형식**: .deb (데비안/우분투 패키지)
* **아키텍처**: x86_64 (amd64)

```bash
sudo dpkg -i chloros-amd64.deb
chloros-cli --version  # Verify installation
```

#### Linux arm64 — NVIDIA Jetson (.deb 패키지 — CLI + 백엔드)

* **파일 형식**: .deb (JetPack 6)
* **아키텍처**: aarch64 (arm64)

```bash
sudo dpkg -i chloros-arm64-jp6.deb
chloros-cli --version  # Verify installation
```

자세한 설치 지침은 [Linux 설치](linux/linux-installation.md)를, Jetson 관련 지침은 [NVIDIA Jetson 가이드](linux/nvidia-jetson-guide.md)를 참조하십시오.

#### Python SDK (모든 플랫폼)

```bash
pip install chloros-sdk
```

문서는 [API : Python SDK](api-python-sdk.md)를 참조하십시오.

{% hint style="info" %}
**Linux 사용자**: `.deb` 패키지는 CLI 및 백엔드를 설치합니다. Python SDK는 pip를 통해 별도로 설치됩니다. Linux용 GUI는 없으며, 모든 상호 작용은 CLI 또는 SDK를 통해 이루어집니다.
{% endhint %}

***

## 추가 리소스

### Python SDK

개발자 및 자동화 워크플로우의 경우, 다음을 설치하십시오: Chloros Python SDK:

```bash
pip install chloros-sdk
```

**문서**: [API: Python SDK](api-python-sdk.md)**필수 사항**: Chloros가 설치되어 있어야 함 (Windows 설치 프로그램 또는 Linux `.deb` 패키지), Chloros+ 라이선스 로그인 필요***

## 포함 내용

### Windows 설치 프로그램

* ✅ **Chloros GUI** - 모든 기능을 갖춘 그래픽 사용자 인터페이스
* ✅ **Chloros CLI** - 명령줄 인터페이스 (Chloros+ 라이선스 필요)
* ✅ **Chloros 백엔드** - 처리 엔진
* ✅ **카메라 프로필** - 사전 구성된 MAPIR 카메라 템플릿

### Linux .deb 패키지

* ✅ **Chloros CLI** - 명령줄 인터페이스 (Chloros+ 라이선스 필요)
* ✅ **Chloros 백엔드** - 처리 엔진
* ✅ **카메라 프로필** - 사전 구성된 MAPIR 카메라 템플릿
* ❌ GUI 없음 — Linux는 헤드리스 CLI/SDK 전용

### Python SDK (PIP, 모든 플랫폼)

* ✅ **Chloros SDK** - Python API (Chloros+ 라이선스 필요)***

## Chloros+로 업그레이드

Chloros+ 구독으로 고급 기능을 이용하세요:

* 🚀 **멀티스레드 처리** - 이미지를 병렬로 처리
* ⚡ **GPU (CUDA) 가속** - NVIDIA GPU 성능 활용
* 💻 **CLI 액세스** - 명령줄 도구를 통한 자동화
* 🐍 **Python SDK** - 프로그래밍 방식의 API 액세스
* 📱 **다중 기기** - 2~10대 이상의 기기에서 사용 가능(요금제별 상이)
* **🐻 고급 텍스처 인식 디베이어링 방식** - AI/ML 노이즈 제거 모델과 결합된 고품질 에지 인식 디베이어링으로, 디베이어링 노이즈를 거의 모두 제거합니다.
* 🧮 **사용자 정의 공식** - 맞춤형 다중 스펙트럼 지수 생성

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary">Chloros+ 요금제 및 가격 보기</a></p>***

## 설치 도움말

### 문제 해결

**다음 오류 메시지와 함께 설치가 실패하는 경우:**

* 관리자 권한이 있는지 확인하십시오
* 바이러스 백신 소프트웨어를 일시적으로 비활성화하십시오
* 시스템 최소 요구 사항을 충족하는지 확인하십시오

**응용 프로그램이 시작되지 않음 (Windows):**

* Windows 10/11 (64비트)가 설치되어 있는지 확인하십시오
* 그래픽 드라이버를 업데이트하십시오
* Windows 이벤트 뷰어에서 오류 세부 정보를 확인하십시오
* 오류 로그를 첨부하여 지원팀에 문의하십시오

**CLI가 시작되지 않음 (Linux):**

* `.deb` 패키지가 올바르게 설치되었는지 확인하십시오: `dpkg -l | grep chloros`
* 권한을 확인하십시오: `sudo chmod +x /usr/bin/chloros-cli`
* 진단 실행: `chloros-cli selftest`
* 누락된 라이브러리 확인: `ldd /usr/lib/chloros/chloros-backend | grep "not found"`

**라이선스 활성화 문제:**

* 인터넷 연결 상태 확인
* [https://cloud.mapir.camera](https://cloud.mapir.camera)에서 자격 증명 확인
* 방화벽이 Chloros를 차단하고 있지 않은지 확인
* 자세한 지침은 [Chloros+ 로그인](chloros+-login.md)을 참조하십시오

### 지원 받기

설치 또는 설정에 도움이 필요하십니까?

* 📧 **이메일**: info@mapir.camera
* 🌐 **웹사이트**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **문서**: [시작하기](./)
* ❓ **FAQ**: [자주 묻는 질문](faq.md)***

## 변경 내역

<details>

<summary>버전 1.1.0 (최신)</summary>

**출시일: 2026년 3월**

**새로운 기능*** **Linux 지원** — Linux amd64(x86_64) 및 arm64(NVIDIA Jetson JetPack 6)용 네이티브 CLI 및 SDK. `.deb` 패키지를 통해 설치하십시오.
* **NVIDIA Jetson 지원** — Jetson Nano, Orin Nano, Orin NX 및 AGX Orin 엣지 디바이스에 대한 최적화된 처리.
* **동적 컴퓨팅 적응** — 자동 하드웨어 감지 및 처리 전략 최적화. Chloros는 Jetson Nano부터 멀티 GPU 워크스테이션에 이르기까지 사용자의 하드웨어에 적응합니다.
* **4스레드 처리 파이프라인** — 동적 GPU 메모리 할당을 통한 동시 감지, 보정, 처리 및 내보내기 스레드.
* **새로운 CLI 명령어** — `selftest`(시스템 진단) 및 `update`(Linux 업데이트 관리).
* **새로운 CLI 프로세스 플래그** — `--debayer`(표준/텍스처 인식), `--indices`(인덱스 지정), `--target`(더 빠른 탐지를 위해 대상 하위 폴더 우선 검색).
* **새로운 GUI 메뉴 항목** — 메인 메뉴 드롭다운에서 &#x27;파일 추가&#x27;, &#x27;폴더 추가&#x27;, &#x27;처리 시작/중지&#x27;에 액세스할 수 있습니다.**개선 사항**

* 크로스 플랫폼 백엔드 자동 감지 (Windows 및 Linux 경로)
* 스레드별 진행 상황 추적 기능을 강화한 SDK 및 `get_status()`
* 새로운 SDK 예외: `ChlorosConfigurationError`, `ChlorosAuthenticationError`
* NVIDIA Jetson용 열 관리 및 적응형 스로틀링
* OOM 발생 시 타일형 GPU 처리로 전환되는 자동 메모리 관리

</details>

<details>

<summary>버전 1.0.5</summary>

**출시일: 2026년 2월 10일**

**새로운 기능*** **텍스처 인식 디베이어링 방식 \[Chloros+ 전용] -** 텍스처 인식 방식은 고품질의 에지 인식 디베이어링과 AI/ML 노이즈 제거 모델을 결합하여 디베이어링 노이즈를 거의 모두 제거합니다.
* **T4P 보정 타겟 지원*** **더 빨라진 Chloros+ GPU 처리, 개선된 메모리 관리**

**버그 수정*** 완전히 새로운 프론트엔드(GUI)로, 이제 모든 Windows 컴퓨터에서 작동해야 합니다.

</details>

<details>

<summary>버전 1.0.4</summary>

**출시일: 2026년 1월 5일**

**새로운 기능*** **이미지/메타데이터 전환**: 파일 브라우저에 토글 기능을 추가하여, 이미지 그리드 대신 테이블 형식으로 선택한 이미지의 메타데이터를 볼 수 있습니다.
* **이미지 그리드 확대/축소 슬라이더**: 썸네일 크기를 조절할 수 있는 새로운 UI 슬라이더 (CTRL + 마우스 휠도 지원)
* **이미지 그리드 내보내기 버튼**: 상단 행에 있는 버튼으로 썸네일을 JPG에서 처리된 내보내기 파일(타겟, 반사율, 인덱스, LUT)로 전환
* **지도 탭**: 이미지의 GPS 위치 마커를 표시하는 새로운 대화형 2D 지도
  * Google 지도 및 ESRI 지도 타일 지원 (확대/축소 수준에 따라 최적의 타일 서비스를 자동 선택)
  * 지도 마커 위에 마우스를 올리면 썸네일 미리보기 표시

**버그 수정*** 비영어권 컴퓨터에서 Chloros 설치 지원 개선

</details>

<details>

<summary>버전 1.0.3</summary>

**출시일: 2025년 12월 20일**

**새로운 기능*** 초기 출시

**개선 사항*** 초기 출시

**버그 수정*** 초기 출시

**알려진 문제*** 초기 출시

</details>***

## 라이선스 계약**독점 소프트웨어** - Copyright (c) 2026 MAPIR Inc.

무단 사용, 배포 또는 수정은 금지됩니다.

**무료 버전**: 기능 제한이 있는 개인 및 상업적 용도로 사용 가능**Chloros+**: 고급 기능 및 상업적 배포를 위한 구독 기반 라이선스
