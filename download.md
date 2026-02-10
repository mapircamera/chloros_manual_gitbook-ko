---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/download
---

# 다운로드

다중 스펙트럼 이미지 처리를 시작하려면 최신 버전의 Chloros를 다운로드하십시오.

### 시스템 요구 사항

| 요구 사항          | 최소 사양                                              | 권장 사양                                          |
| -------------------- | ---------------------------------------------------- | ---------------------------------------------------- |
| **운영 체제** | Windows 10 (64비트)                                  | Windows 11 (64비트)                                  |
| **프로세서**        | Intel Core i5 또는 동급 프로세서                          | Intel Core i7 이상 프로세서                              |
| **메모리 (RAM)**     | 8GB                                                  | 16GB 이상                                         |
| **그래픽 카드**    | DirectX 11 호환                                | 4GB 이상 VRAM 탑재 NVIDIA GPU                            |
| **저장 공간**          | 6GB 여유 공간                                       | 10GB 이상 여유 공간 있는 SSD                            |
| **디스플레이**          | 1920x1080                                            | 2560x1440 이상                                  |
| **인터넷**         | [선택 사항] Chloros+ 라이선스 활성화에 필요 | [선택 사항] Chloros+ 라이선스 활성화에 필요 |

{% hint style="info" %}
**GPU 가속**: NVIDIA GPU를 사용하는 Chloros+ 사용자는 CUDA 가속을 통해 훨씬 빠른 처리가 가능합니다. Chloros+ 사용자는 최대 속도를 위한 멀티스레드 처리 기능도 활용할 수 있습니다.
{% endhint %}

***

## Chloros 다운로드

### <a href="https://drive.google.com/file/d/1HjwrUY4M7HGxDbMybO7iPe_6JoHnUGr4/view?usp=drive_link" class="button primary">Chloros 다운로드</a>

### 최신 안정 버전

**Chloros 설치 프로그램 (Windows용)*** **버전**: 1.0.5
* **출시일**: 2026년 2월 10일
* **파일 크기 (다운로드)**: 1.6GB
* **설치 후 파일 크기**: 5.7GB
* **파일 유형**: .exe (Windows 설치 프로그램)

#### **설치 단계:**

1. `CHLOROS INSTALLER - CURRENT VERSION.exe` 파일을 다운로드합니다
2. 설치 프로그램을 더블클릭하여 설치를 시작하세요
3. 설치 마법사의 안내를 따르세요
4. 설치 디렉터리를 선택하세요 (기본값: `C:\Program Files\[USER]\Chloros\`)
5. 설치를 완료하고 Chloros 또는 Chloros CLI를 실행하세요
6. [MAPIR Cloud Chloros+ 계정](https://cloud.mapir.camera/pricing)으로 로그인하세요 (또는 무료 버전으로 계속 사용)

{% hint style="success" %}
설치 프로그램은 명령줄 접근을 위해 시스템 PATH에 `chloros-cli`를 자동으로 추가합니다.
{% endhint %}

***

## 추가 리소스

### Python SDK

개발자 및 자동화 워크플로를 위해 다음을 설치하십시오:
Chloros Python SDK:

```bash
pip install chloros-sdk
```

**문서**: [API: Python SDK](api-python-sdk.md)**필수 조건**: Chloros 데스크톱 설치 완료, Chloros+ 라이선스 로그인 필요***

## 포함 항목

Chloros 설치에는 다음이 포함됩니다:

* ✅ **Chloros** - 모든 기능을 갖춘 그래픽 사용자 인터페이스(GUI)
* ✅ **Chloros CLI** - 명령줄 인터페이스(Chloros+ 라이선스 필요)
* ✅ **Chloros SDK** - Python API (Chloros+ 라이선스 필요)
* ✅ **카메라 프로필** - 사전 구성된 카메라 템플릿***

## Chloros+로 업그레이드

Chloros+ 구독으로 고급 기능 활용:

* 🚀 **멀티스레드 처리** - 병렬 이미지 처리
* ⚡ **GPU(CUDA) 가속** - NVIDIA GPU 성능 활용
* 💻 **CLI 액세스** - 명령줄 도구로 자동화
* 🐍 **Python SDK** - 프로그래매틱 API 접근
* 📱 **다중 기기** - 2~10개 이상의 기기에서 사용 가능 (플랜에 따라 다름)
* **🐻 고급 텍스처 인식 디베이링 방식** - 고품질의 에지 인식 디베이링과 AI/ML 노이즈 제거 모델을 결합하여 디베이링 노이즈를 거의 완벽하게 제거합니다.
* 🧮 **사용자 정의 공식** - 맞춤형 다중 스펙트럼 지수 생성

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary">Chloros+ 플랜 및 가격 보기</a></p>***

## 설치 도움말

### 문제 해결

**설치 실패 시 오류 메시지:**

* 관리자 권한을 확인하세요
* 바이러스 백신 소프트웨어를 일시적으로 비활성화하세요
* 최소 시스템 요구 사항을 충족하는지 확인하세요

**응용 프로그램이 실행되지 않음:**

* Windows 10/11 (64비트) 설치 여부 확인
* 그래픽 드라이버 업데이트
* Windows 이벤트 뷰어에서 오류 세부 정보 확인
* 오류 로그와 함께 지원팀에 문의

**라이선스 활성화 문제:**

* 인터넷 연결이 활성화되었는지 확인하세요
* [https://cloud.mapir.camera](https://cloud.mapir.camera)에서 인증 정보를 확인하세요
* 방화벽이 Chloros를 차단하지 않는지 확인하세요
* 자세한 안내는 [Chloros+ 로그인](chloros+-login.md) 참조

### 지원 받기

설치 또는 설정 관련 도움이 필요하신가요?

* 📧 **이메일**: info@mapir.camera
* 🌐 **웹사이트**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **문서**: [시작하기](./)
* ❓ **FAQ**: [자주 묻는 질문](faq.md)***

## 변경 내역

<details>

<summary>버전 1.0.5</summary>

#### **출시일**: 2026년 2월 10일**새로운 기능*** **텍스처 인식 디베이링 방법 \[Chloros+ 전용] -** 텍스처 인식은 고품질의 에지 인식 디베이링과 AI/ML 노이즈 제거 모델을 결합하여 디베이링 노이즈를 거의 완전히 제거합니다.
* **T4P 보정 타겟 지원*** **Chloros+ GPU 처리 속도 향상, 메모리 관리 개선**

**버그 수정*** 완전히 새로워진 프론트엔드(GUI), 이제 모든 Windows 컴퓨터에서 작동합니다.

</details>

<details>

<summary>버전 1.0.4</summary>

#### **출시일**: 2026년 1월 5일**새로운 기능*** **이미지/메타데이터 전환**: 파일 브라우저에 선택한 이미지의 메타데이터를 이미지 그리드 대신 테이블로 볼 수 있는 전환 기능 추가
* **이미지 그리드 확대/축소 슬라이더**: 새 UI 슬라이더로 썸네일 크기 조절 가능 (CTRL + 마우스 휠도 지원)
* **이미지 그리드 내보내기 버튼**: 상단 행에 JPG에서 처리된 내보내기(타겟, 반사율, 지수, LUT)로 썸네일 형식을 전환하는 버튼 추가
* **지도 탭**: 이미지 GPS 위치 마커를 표시하는 새로운 인터랙티브 2D 지도
  * Google Maps 및 ESRI 지도 타일 지원 (확대/축소 수준 가용성에 따라 최적 타일 서비스 자동 선택)
  * 지도 마커 위 마우스 오버 시 썸네일 미리보기

**버그 수정*** 비영어권 컴퓨터에서 Chloros 설치 지원 개선

</details>

<details>

<summary>버전 1.0.3</summary>

#### **출시일**: 2025년 12월 20일**신규 기능*** 초기 출시

**개선 사항*** 초기 출시

**버그 수정*** 초기 출시

**알려진 문제*** 초기 출시

</details>***

## 라이선스 계약**독점 소프트웨어** - 저작권 (c) 2026 MAPIR Inc.

무단 사용, 배포 또는 수정은 금지됩니다.

**무료 버전**: 기능 제한이 있는 개인 및 상업적 사용 가능**Chloros+**: 고급 기능 및 상업적 배포를 위한 구독 기반 라이선스
