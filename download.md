---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/download
---

# 다운로드

다중 스펙트럼 이미지 처리를 시작하려면 Chloros의 최신 버전을 다운로드하세요.

### 시스템 요구 사항

#### Windows


| 요구 사항          | 최소 사양                                              | 권장 사양                                          |
| -------------------- | ---------------------------------------------------- | ---------------------------------------------------- |
| **운영 체제** | Windows 10 (64비트)                                  | Windows 11 (64비트)                                  |
| **프로세서**        | Intel Core i5 또는 동급                          | Intel Core i7 이상                              |
| **메모리 (RAM)**     | 8GB                                                  | 16GB 이상                                         |
| **그래픽 카드**    | DirectX 11 호환                                | 4GB 이상의 VRAM을 갖춘 NVIDIA GPU                            |
| **저장 공간**          | 6GB의 여유 공간                                       | 10GB 이상의 여유 공간이 있는 SSD                            |
| **디스플레이**          | 1920x1080                                            | 2560x1440 이상                                  |
| **인터넷**         | [선택 사항] Chloros+ 라이선스 활성화에 필요           | [선택 사항] Chloros+ 라이선스 활성화에 필요           |

#### Linux amd64 (x86_64)

| 요구 사항       | 최소                    | 권장               |
| ----------------- | -------------------------- | ------------------------- |
| **배포판**  | Ubuntu 22.04 LTS 이상 / Debian 12 이상 | Ubuntu 24.04 LTS      |
| **프로세서**     | x86_64 (Intel/AMD)        | Intel Core i7 이상   |
| **메모리 (RAM)**  | 8GB                        | 16GB 이상              |
| **그래픽 카드** | 없음 (CPU 처리)      | 4GB 이상의 VRAM을 갖춘 NVIDIA GPU |
| **저장 공간**       | 2GB 여유 공간             | 10GB 이상의 여유 공간이 있는 SSD       |
| **Python**        | Python 3.7 이상 (SDK용)      | Python 3.10 이상              |

#### Linux arm64 (NVIDIA Jetson)

| 요구 사항      | 최소                      | 권장                     |
| ---------------- | ---------------------------- | ------------------------------- |
| **플랫폼**     | JetPack 6이 설치된 NVIDIA Jetson | Jetson Orin NX 16GB 또는 AGX Orin |
| **메모리 (RAM)** | 8GB (GPU/CPU 공유)         | 16GB 이상의 공유 메모리                    |
| **저장 공간**      | 2GB 여유 공간               | 10GB 이상의 여유 공간이 있는 NVMe SSD        |
| **Python**       | Python 3.7 이상 (SDK용)        | Python 3.10 이상                    |

{% hint style="info" %}
**GPU 가속**: NVIDIA GPU를 사용하는 Chloros+ 사용자는 CUDA 가속을 통해 처리 속도를 대폭 높일 수 있습니다. 이 기능은 Windows 및 Linux amd64 환경의 NVIDIA 데스크톱 GPU와 JetPack 6이 설치된 NVIDIA Jetson에서 작동합니다. 또한 Chloros+ 사용자는 최대 속도를 위한 멀티스레드 처리 기능을 이용할 수 있습니다.
{% endhint %}

***

## Chloros
다운로드

### 최신 안정화 버전: 버전 1.3.0 (2026년 11월 11일)

### <a href="https://drive.google.com/uc?export=download&#x26;id=1HjwrUY4M7HGxDbMybO7iPe_6JoHnUGr4" class="button primary">Windows용 Chloros 다운로드 (.exe)</a>


### <a href="https://drive.google.com/uc?export=download&#x26;id=1dB8-ke3wxNXpw_e1qJ4BhwBpCoNd4kLS" class="button primary">Linux amd64용 Chloros 다운로드 (.deb)</a>


### <a href="https://drive.google.com/uc?export=download&#x26;id=1d1OwdcYA4Rf4jkuPi2IBeWT2772_HnyO" class="button primary">Linux arm64 / Jetson용 Chloros 다운로드 (.deb)</a>


#### Windows 설치 프로그램 (GUI + CLI + 백엔드)

* **파일 유형**: .exe (Windows 설치 프로그램)

**설치 단계:**

1. 위의 .exe 파일을 다운로드합니다.
2. 설치 프로그램을 두 번 클릭하여 설치를 시작합니다.
3. 설치 마법사의 안내에 따라 진행합니다.
4. 설치 디렉터리를 선택합니다(기본값: `C:\Program Files\Chloros\`).
5. **추가 작업 선택**페이지에서 두 가지**시스템 설정** 옵션 모두에 체크된 상태를 유지합니다(권장 — 아래 참조).
6. 설치를 완료하고 Chloros 또는 Chloros CLI를 실행합니다. 7. [MAPIR Cloud Chloros+ 계정](https://cloud.mapir.camera/pricing)으로 로그인합니다. (또는 무료 버전으로 계속 진행)

**시스템 설정 옵션** (기본적으로 둘 다 선택됨):

| 설치 프로그램에 표시된 옵션명 | 기능 | 자동 설치 비활성화 |
| --- | --- | --- |
| **Windows 긴 경로 활성화** (프로젝트 폴더 경로가 260자보다 길어도 허용; 권장) | Windows의 시스템 전체 설정인 `LongPathsEnabled`를 1로 설정합니다. Windows는 이 설정이 비활성화된 상태로 제공되며, 이 경우 파일 경로는 260자 이상일 수 없습니다. Chloros 제품은 프로젝트 폴더 아래로 최대 약 130자까지 위치하므로, 긴 경로 기능이 비활성화되면 더 깊은 위치의 프로젝트 폴더는 허용되지 않습니다. 설치 후 실행되는 프로그램에 대해 적용됩니다. 다른 프로그램에서 이 설정에 의존할 수 있으므로, 제거 후에도 설정은 유지됩니다. | `/MERGETASKS="!longpaths"` |
| **Microsoft Defender 스캔에서 Chloros 백엔드 제외** (카메라 캡처 속도 대폭 향상; 권장) | 설치 폴더 내의 `chloros-backend.exe`에 대해 Microsoft Defender 프로세스 제외 항목을 추가하여, 백엔드가 어디에 파일을 기록하든 실시간 보호 기능이 해당 파일을 스캔하지 않도록 합니다. 높은 캡처 속도에서 스캔에 소요되는 시간은 캡처 속도의 약 30% 수준이었습니다(9대의 카메라로 연속 캡처 시 초당 77.7회 대 52.7회). 다른 프로그램은 제외되지 않으며, 제거 시 이 제외 항목은 삭제됩니다. Defender가 없거나 정책에 의해 잠겨 있는 경우, 이 단계는 오류 메시지 없이 실패하고 설치가 계속 진행됩니다. | `/MERGETASKS="!defenderexclusion"` |

무음 설치(`/SILENT` 또는 `/VERYSILENT`)는 표시된 `/MERGETASKS` 값으로 제외를 선택하지 않는 한 두 옵션을 모두 적용합니다.

{% hint style="success" %}
설치 프로그램은 CLI 폴더(`C:\Program Files\Chloros\cli\`)를 시스템에 한 번만 추가합니다(`PATH`). 재설치나 업그레이드 시 중복 항목이 추가되지 않으며, 이전 설치 프로그램이 추가한 중복 항목은 제거됩니다. 설치 후 새로운 터미널을 열어 `chloros-cli`를 사용하십시오. 또한 설치 프로그램은 모든 네트워크 프로필에서 백엔드를 위한 두 가지 Windows방화벽 규칙을 추가합니다: 카메라 스트림, PTP, mDNS 및 DAQ-E 데이터 스트림을 위한 **Chloros 백엔드(UDP-In)**와, 센서가 이미지를 수집하기 위해 이 컴퓨터에 다시 연결되는 DAQ-E 펌웨어 업데이트(로컬 서브넷으로 제한됨)를 위한**Chloros 백엔드(TCP-In)**입니다. 두 규칙 모두 프로그램 제거 시 삭제됩니다.
{% endhint %}

#### Linux amd64 (.deb 패키지 — CLI + 백엔드)

* **파일 형식**: .deb (Debian/Ubuntu 패키지)
* **아키텍처**: x86_64 (amd64)

```bash
sudo apt install ./chloros_1.3.0_amd64.deb
chloros-cli --version     # Verify installation: "Chloros CLI 1.3.0"
```

#### Linux arm64 — NVIDIA Jetson (.deb 패키지 — CLI + 백엔드)

* **파일 형식**: .deb (JetPack 6)
* **아키텍처**: aarch64 (arm64)

```bash
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
chloros-cli --version     # Verify installation: "Chloros CLI 1.3.0"
```

실제로 다운로드한 파일 이름을 사용하고, `./`를 그대로 유지하여 `apt`가 이름으로 패키지를 찾는 대신 해당 파일을 설치하도록 하십시오. `apt install`로 설치하십시오. `dpkg -i`가 아닌 `apt install`로 설치하십시오. apt는 패키지가 필수 조건이 아닌 권장 사항으로 지정하는 Python SDK의 종속성도 함께 설치하지만, `dpkg -i`는 이를 건너뜁니다. 1.3.0 버전부터 백엔드와 CLI는 `/usr/lib/chloros/` 아래에 압축 해제된 상태로 설치되므로, 별도로 압축을 풀지 않고도 바로 실행됩니다.

Jetson 패키지는 JetPack 6의 CUDA 빌드 PyTorch를 사용하므로, 포함된 Texture Aware 디베이어인 processing이 Jetson의 GPU에서 실행됩니다. 이 패키지는 JetPack 6이 설치하는 CUDA 및 cuDNN 라이브러리를 사용합니다(해당 라이브러리가 없는 이미지의 경우 `sudo apt install nvidia-jetpack`).

자세한 설정 지침은 [Linux 설치](linux/linux-installation.md)를, Jetson 관련 지침은 [NVIDIA Jetson 가이드](linux/nvidia-jetson-guide.md)를 참조하십시오.

#### Python SDK (모든 플랫폼)

모든 설치 프로그램에는 해당 `chloros_sdk` 휠이 포함되어 있으므로, SDK버전은 항상 설치된 GUI/CLI/백엔드와 일치합니다. Windows에서는 설치 프로그램이 이를 시스템 Python에 자동으로 설치합니다. Linux에서는 `.deb`가 시스템 `python3` 자체용 SDK (`/usr/lib/python3/dist-packages` 내)를 설치하므로, 따라서 `import chloros_sdk`는 설치 직후, pip나 네트워크 연결 없이도 바로 작동합니다. 시스템 패키지를 인식하지 못하는 가상 환경(virtualenv)의 경우에만 번들된 휠을 해당 환경에 설치해야 합니다:

```bash
pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

pip만 지원하는 호스트(Chloros 패키지가 설치되지 않은 경우; 이 경우 SDK는 다른 머신의 백엔드를 구동합니다)의 경우, SDK는 PyPI에서도 제공됩니다:

```bash
pip install chloros-sdk
```

자세한 내용은 [API: Python SDK](api-python-sdk.md) 및 [SDK 참조](reference/sdk-reference.md) 문서를 참조하십시오.

{% hint style="info" %}
**Linux 사용자**: `.deb` 패키지는 CLI, 백엔드 및 Python SDK를 설치합니다. Linux용 GUI는 제공되지 않으며, 모든 상호 작용은 CLI 또는 SDK를 통해 이루어집니다.
{% endhint %}

***

## 추가 리소스

### Python SDK


개발자 및 자동화 워크플로우를 위해 모든 설치 프로그램에 Chloros Python SDK가 포함되어 있습니다. Windows 설치 프로그램은 이를 사용자의 Python 환경에 설치하며, Linux용 `.deb`는 시스템용 `python3`에 설치합니다. Chloros 패키지가 설치되지 않은 시스템(다른 시스템의 백엔드를 구동하는 경우)에서는 PyPI에서 설치하십시오:

```bash
pip install chloros-sdk
```

**문서**: [API: Python SDK](api-python-sdk.md)

**필수 조건**: 구동할 Chloros 백엔드(Windows 설치 프로그램 또는 Linux용 `.deb` 패키지가 이를 설치합니다)와 유료 등급의 Chloros+ 로그인 정보

***

## 포함 사항

### Windows 설치 프로그램

* ✅ **Chloros GUI** - 모든 기능을 갖춘 그래픽 사용자 인터페이스
* ✅ **Chloros CLI** - 명령줄 인터페이스 (Chloros+ 라이선스 필요)
* ✅ **Chloros 백엔드** - 처리 엔진
* ✅ **Python SDK** - Python이 설치되어 있는 경우 해당 환경에 설치됨 (Chloros+ 라이선스 필요)
* ✅ **카메라 프로필** - 사전 구성된 MAPIR 카메라 템플릿

### Linux.deb 패키지

* ✅ **Chloros CLI** - 명령줄 인터페이스 (Chloros+ 라이선스 필요)
* ✅ **Chloros 백엔드** - 처리 엔진
* ✅ **Python SDK** - `python3` 시스템에 설치됨 (Chloros+ 라이선스 필요)
* ✅ **카메라 프로필** - 사전 구성된 MAPIR 카메라 템플릿
* ❌ GUI 없음 — Linux는 헤드리스 모드로, CLI/SDK 전용입니다

### Python SDK (PyPI, Chloros 패키지가 없는 시스템용)

* ✅ **Chloros SDK** - Python API (Chloros+ 라이선스 필요)

***

## Chloros+로 업그레이드

Chloros+ 구독을 통해 고급 기능을 활용하세요:

* 🚀 **멀티스레드 처리** - 이미지를 병렬로 처리
* ⚡ **GPU (CUDA) 가속** - NVIDIA GPU 성능 활용
* 💻 **CLI액세스** - 명령줄 도구를 통한 자동화
* 🐍 **Python SDK** - 프로그래밍 방식의 API액세스
* 📱 **다중 기기** - 2~10대 이상의 기기에서 사용 가능(요금제별 상이)
* **🐻 고급 텍스처 인식 디베이어링 방식** - AI/ML 노이즈 제거 모델과 결합된 고품질 에지 인식 디베이어링 방식으로, 디베이어링 과정에서 발생하는 노이즈를 거의 모두 제거합니다.
* 🧮 **사용자 정의 공식** - 사용자 정의 다중 스펙트럼 지수 생성

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary">Chloros+ 요금제 및 가격 보기</a></p>

***

## 설치 도움말

### 문제 해결

**다음 오류 메시지가 표시되며 설치가 실패하는 경우:**

* 관리자 권한이 있는지 확인하십시오.
* 바이러스 백신 소프트웨어를 일시적으로 비활성화하십시오.
* 시스템 최소 요구 사항을 충족하는지 확인하십시오.

**응용 프로그램이 실행되지 않는 경우 (Windows):**

* Windows 10/11(64비트)가 설치되어 있는지 확인하십시오
* 그래픽 드라이버를 업데이트하십시오
* Windows 이벤트 뷰어에서 오류 세부 정보를 확인하십시오
* 오류 로그를 첨부하여 지원팀에 문의하십시오

**CLI가 실행되지 않는 경우 (Linux):**

* `.deb` 패키지가 올바르게 설치되었는지 확인하십시오: `dpkg -l | grep chloros`
* 권한을 확인하십시오: `sudo chmod +x /usr/bin/chloros-cli`
* 진단 실행: `chloros-cli selftest`
* 누락된 라이브러리 확인: `ldd /usr/lib/chloros/backend/chloros-backend | grep "not found"` (1.3.0 버전부터 `/usr/lib/chloros/chloros-backend`는 작은 실행 스크립트이며, 프로그램 자체는 `/usr/lib/chloros/backend/`에 있습니다)

**라이선스 활성화 문제:**

* 인터넷 연결이 정상적으로 되어 있는지 확인하십시오
* [https://cloud.mapir.camera](https://cloud.mapir.camera)에서 인증 정보를 확인하십시오
* 방화벽이 Chloros를 차단하고 있지 않은지 확인하세요* 자세한 지침은 [Chloros+ 로그인](chloros+-login.md)을 참조하세요

### 지원 받기

설치나 설정에 도움이 필요하신가요?

* 📧 **이메일**: info@mapir.camera
* 🌐 **웹사이트**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **문서**: [시작하기](./)
* ❓ **FAQ**: [자주 묻는 질문](faq.md)

***

## 소프트웨어 업데이트

Chloros는 업데이트를 확인하고, 새 버전이 출시되면 알림을 보내며, 이 다운로드 페이지로 연결해 줍니다. 사용자는 새로 서명된 설치 프로그램을 실행하여 업데이트할 수 있습니다. 설정, 프로젝트 및 로그인 정보는 업데이트 후에도 유지됩니다. Linux 및 Jetson에서 `chloros-cli update`는 최신 버전을 확인하고, 해당 `.deb`를 다운로드하여 설치할 것을 제안합니다(이 명령어는 Linux에서만 사용 가능합니다).

***

## 변경 내역**버전 1.3.0 (최신 버전, 2026년 11월 11일)**— 전체 목록은 [시작하기](./) 페이지의**Chloros

1.3.0의 새로운 기능**을 참조하십시오.**버전 1.2.0**— [시작하기](./) 페이지의**Chloros 1.2.0의 새로운 기능**을 참조하십시오.

<details>

<summary>버전 1.0.5</summary>

**출시일: 2026년 2월 10일**

**새로운 기능**

* **텍스처 인식 디베이어 방식 \[Chloros+ 전용] -** 텍스처 인식 방식은 고품질의 에지 인식 디베이어와 AI/ML 노이즈 제거 모델을 결합하여 디베이어링 과정에서 발생하는 노이즈를 거의 모두 제거합니다.
* **T4P 보정 타겟 지원**
* **Chloros+의 GPU 처리 속도 향상 및 메모리 관리 개선**

**버그 수정**

* 완전히 새로워진 프론트엔드(GUI)로, 이제 모든 Windows 컴퓨터에서 정상 작동합니다.

</details>

<details>

<summary>버전 1.0.4</summary>

**출시일: 2026년 1월 5일**

**새로운 기능**

* **이미지/메타데이터 전환**: 파일 브라우저에 토글 기능을 추가하여, 선택한 이미지의 메타데이터를 이미지 그리드 대신 표 형식으로 볼 수 있습니다.
* **이미지 그리드 확대/축소 슬라이더**: 썸네일 크기를 조절할 수 있는 새로운 UI 슬라이더 (CTRL + 마우스 휠도 지원)
* **이미지 그리드 내보내기 버튼**: 상단 행에 있는 버튼을 통해 썸네일을 JPG에서 처리된 내보내기 파일(타겟, 반사율, 인덱스, LUT)로 전환할 수 있습니다.
* **지도 탭**: 이미지의 GPS 위치 마커를 표시하는 새로운 대화형 2D 지도
  * Google Maps 및 ESRI 지도 타일 지원 (확대/축소 수준에 따라 최적의 타일 서비스를 자동으로 선택)
  * 지도 마커 위에 마우스를 올리면 썸네일 미리보기 표시

**버그 수정**

* 비영어권 컴퓨터에서 Chloros 설치 지원 개선

</details>

<details>

<summary>버전 1.0.3</summary>

**출시일: 2025년 12월 20일**

**새로운 기능**

* 첫 출시

**개선 사항**

* 첫 출시

**버그 수정**

* 첫 출시

**알려진 문제**

* 첫 출시

</details>

***

## 라이선스 계약**독점 소프트웨어** - Copyright (c) 2026 MAPIR Inc.

무단 사용, 배포 또는 수정은 금지됩니다.

**무료 버전**: 기능 제한이 있으나 개인 및 상업적 용도로 사용 가능**Chloros+**: 고급 기능 및 상업적 배포를 위한 구독 기반 라이선스
