# CLI: 명령줄

> **전체 참조:**[CLI 참조](reference/cli-reference.md)는**모든 하위 명령어의 모든 플래그**를 설명하며, AI 어시스턴트에 최적화되어 있습니다. 해당 URL을 어시스턴트에 붙여넣고 실행 가능한 명령어를 요청해 보세요: `https://mapir.gitbook.io/chloros/reference/cli-reference`
>
> **AI 도구 사용 팁:** 이 설명서의 모든 페이지는 해당 URL에 `.md`를 추가하면(예: `https://mapir.gitbook.io/chloros/reference/cli-reference.md`) 원본 마크다운 형식으로 확인할 수 있으며, 또한 `https://mapir.gitbook.io/chloros/llms.txt`는 LLM이 활용할 수 있도록 전체 매뉴얼을 색인화합니다.

> **AI 어시스턴트가 Chloros 자체를 실행하도록 하고 싶으신가요?** Python SDK와 함께 설치된 MCP 서버인 `chloros-mcp`는 Claude Desktop, Claude Code, Cursor 및 기타 MCP 클라이언트와 호환됩니다. 자세한 내용은 [AI 어시스턴트와 Chloros 함께 사용하기](ai-assistants.md)를 참조하세요.

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: banner shows CLI 1.1.0; reshoot the CLI welcome/banner output on the 1.3.0 build so the version line reads "Chloros CLI 1.3.0" -->


## CLI
란?

`chloros-cli`는 Chloros
데스크톱 앱이 사용하는 것과 동일한 처리 엔진의 명령줄 프론트엔드입니다. 이는 Chloros
백엔드(`127.0.0.1:5000`의 로컬 서버)를 통해 작동하는 씬 HTTP
클라이언트입니다. 대부분의 명령은 백엔드를 자동으로 시작하므로, 스크립트에서는 `chloros-cli process …` 호출 한 번만으로도 충분합니다. 그런 다음 백엔드는 다음 명령을 처리하기 위해 계속 실행됩니다: `chloros-cli logout`에서 중지되며(앱이 열려 있거나 실행 또는 빌드가 진행 중인 경우는 제외), `chloros-cli backend stop`에서 중지되거나, 클라이언트, 실행 중인 작업 또는 연결된 장치가 없는 상태에서 30분이 지나면 자동으로 중지됩니다. `chloros-cli backend status`가 이를 보여줍니다.

이 프로그램은 **Windows
10/11 (x64)**및**Linux
(x86_64, JetPack 6의 NVIDIA Jetson arm64)**에서 모든 터미널에서 실행되며, GUI가 필요하지 않습니다. 다음 명령어로 설치 상태를 확인하십시오:

```bash
chloros-cli --version    # prints "Chloros CLI 1.3.0"
```

명령어 그룹 요약:

* **처리 및 계정** — `process`, `login`, `logout`, `status`, `export-status`, `language` (38개 언어 — [지원 언어](supported-languages.md) 참조), `set-project-folder` / `get-project-folder` / `reset-project-folder`, `selftest`, `update` (Linux
/Jetson 전용)
* **실시간 하드웨어** — `lattice` (LATTICE 카메라 제어, 45개 이상의 하위 명령어), `daq pool-*` (DAQ 광 센서), `time-sync` (PTP)
* **자동화** — `project` (YAML 캡처 레시피를 포함하여 저장된 Chloros
프로젝트를 헤드리스 모드로 실행)

알아두면 유용한 전역 옵션: `--port N` (백엔드 포트, 기본값 `5000`), `-v/--verbose`, `--restart` (백엔드 강제 재시작), `--backend-exe PATH`. 전체 목록은 [CLI
참조](reference/cli-reference.md)를 참조하십시오.

***

## 설치

CLI
는 모든 플랫폼에서 **Chloros
설치 프로그램**에 포함되어 제공되며, 별도의 CLI
다운로드 파일은 없습니다. [다운로드](download.md) 페이지에서 설치 프로그램을 다운로드하십시오.

### Windows


설치 프로그램은 CLI
를 다음 위치에 설치합니다:

```

C:\Program Files\Chloros\cli\chloros-cli.exe
```

에 설치하며, 해당 폴더를 시스템 `PATH`에 한 번 추가합니다(업그레이드 시 이전 설치 프로그램이 남긴 중복 항목도 제거됩니다). 설치 후 **새로운 터미널을 열면**업데이트된 `PATH`가 인식됩니다. 또한 설치 프로그램은 설치 루트 디렉터리에 런처 스크립트(`Chloros_CLI.bat` / `Chloros_CLI.ps1`)와**Chloros
CLI
** 시작 메뉴 바로가기를 생성하며, 각각을 실행하면 `chloros-cli`가 실행 준비된 상태로 터미널이 열립니다.

### Linux


사용 중인 아키텍처에 맞는 `.deb`를 `apt`와 함께 설치하십시오(Python
SDK
의 권장 요구 사항을 생략하는 `dpkg -i`는 사용하지 마십시오):

```bash
# Linux x86_64
sudo apt install ./chloros_1.3.0_amd64.deb

# NVIDIA Jetson (arm64, JetPack 6)
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
```

이렇게 하면 `chloros-cli`에서 `/usr/bin/chloros-cli`까지 설치되며 (이미 `PATH`에 포함됨), `/usr/lib/chloros/backend/` 및 `/usr/lib/chloros/cli/`에 압축 해제된 백엔드 및 CLI
프로그램 (`/usr/lib/chloros/chloros-backend`는 백엔드를 시작하는 런처입니다), 백엔드가 LATTICE 카메라를 위해 필요로 하는 Arena SDK
런타임, 그리고 시스템용 Python
SDK
인 `python3`를 설치합니다. 자세한 내용은 [Linux
설치](linux/linux-installation.md)를 참조하십시오.

### 확인

```bash
chloros-cli --version    # "Chloros CLI 1.3.0"
chloros-cli selftest     # 7-step diagnostic: backend, API, GPU/CUDA, denoiser models
chloros-cli status       # license tier + logged-in user
```

***

## 로그인 및 라이선스

CLI
(및 Python
SDK
)에 액세스하려면 **유료 Chloros
+ 요금제**가 필요합니다. 모든 유료 요금제에는 이 기능이 포함되어 있지만, 무료 요금제에는 포함되어 있지 않습니다. 이 제한은 CLI
바이너리가 아닌 백엔드에 의해 **서버 측**에서 적용됩니다. 로그아웃된 상태에서 호출하면 `401 AUTH_REQUIRED` 오류가 발생하며, 무료 요금제에서 로그인한 상태의 호출은 `403 PLAN_UPGRADE_REQUIRED` 오류로 반환되며, 이는 호출이 `chloros-cli`, SDK
또는 직접 작성한 HTTP
클라이언트에서 이루어졌는지와 무관합니다. [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)에서 업그레이드하십시오.

**기기당 한 번**만 로그인하십시오:

```bash
chloros-cli login user@example.com 'YourPassword'
chloros-cli status
```

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: login success output predates 1.2.0; reshoot `chloros-cli login` followed by `chloros-cli status` on the 1.3.0 build showing the license tier line -->


{% hint style="warning" %}
**특수 문자가 포함된 비밀번호**(`$`, `!`, spaces): wrap the password in**single quotes**, as shown above. In PowerShell double quotes, `$$`는 셸에 의해 왜곡됩니다. (CLI
는 401 오류 시 이를 감지하여 자동으로 재시도하지만, 작은따옴표를 사용하면 이 문제를 완전히 피할 수 있습니다).
{% endhint %}

세션은 `~/.chloros/user_session.json`에 캐시되며, 요금제의 유예 기간 동안(월간 요금제의 경우 30일, 연간 요금제의 경우 만료일까지) 오프라인 상태에서도 계속 작동합니다. `chloros-cli status`는 유료 요금제가 없어도 작동하므로, 거부 사유를 항상 확인할 수 있습니다.

{% hint style="danger" %}
**헤드리스 작업을 예약하시나요? 먼저 로그인하세요.**백엔드 생성 명령어(`process`, `status`, `export-status`, …)을**캐시된 세션 없이**실행하면 즉시 실패하지 않고, 표준 입력(stdin)을 통해 대화형 `Email:` / `Password:` 프롬프트로 전환됩니다. 따라서 무인 cron 작업이나 CI 단계는**입력을 기다리다가 응답이 멈출** 것입니다. 작업을 예약하기 전에 해당 시스템에서 `chloros-cli login EMAIL 'PASSWORD'`를 한 번 실행하십시오.
{% endhint %}

***

## 첫 번째 처리 실행

`process`를 캡처 파일이 있는 폴더로 지정하면, Survey3
(`.raw` + `.jpg`), LATTICE(`.tif`/`.tiff`), `.dng` 또는 이들의 혼합:

```bash
chloros-cli process "C:\Images\flight_001"          # Windows
chloros-cli process ~/images/flight_001              # Linux
```

진행 상황은 파이프라인 스레드(탐지, 분석, 처리, 내보내기)별로 실시간으로 표시되며, 실행이 성공적으로 완료되면 작성된 이미지 산출물의 개수가 보고됩니다: `Image products written: N`, 그리고 실행 과정에서 [클램프 마스크](output-image-formats.md#clamp-masks)도 작성된 경우(이들은 산출물로 집계되지 않음)에는 `(plus M clamp masks)`가 이어집니다.



Chloros<!-- SCREENSHOT-NEEDED: terminal capture of a `chloros-cli process` run on a LATTICE captures folder completing successfully — per-thread progress lines visible and the final "Image products written: N (plus M clamp masks)" summary line -->
백엔드가 이미 실행 중일 경우(예: 데스크톱 앱의 백엔드), `process`는 이를 사용하여 해당 백엔드에서 프로젝트를 엽니다. 앱이 프로젝트를 열어둔 상태에서는 실행을 거부합니다: 앱에서 프로세스를 실행하거나 해당 프로젝트를 닫으십시오. 이전 실행이 아직 완료되지 않은 경우, 최대 15초 동안 기다린 후 거절합니다. `--restart`는 새로운 백엔드를 강제적으로 시작합니다. Linux
에서 systemd 서비스로 실행되는 백엔드는 절대 재시작되지 않습니다: `process`는 이를 있는 그대로 사용하며, 일반적으로 재시작해야 할 상황(`--restart` 또는 아무것도 찾지 못한 임포트)에서는 `sudo systemctl restart <unit>`를 출력하고 종료됩니다(`1`).

### 출력 결과 저장 위치

`process`는 입력 폴더가 아닌 **프로젝트 폴더**에 결과를 기록합니다:

* `-o`가 없는 경우: 프로젝트는 기본 프로젝트 폴더(GUI와 공유됨; `get-project-folder` / `set-project-folder`로 관리; 설정된 폴더가 없을 경우, 사용자 폴더 내의 `Documents/MAPIR/Chloros Projects`)에 생성되며, 이름이 `-n/--project-name`로 지정되거나 생략 시 타임스탬프(`YYYYMMDD_HHMMSS`)로 명명됩니다.
* `-o PATH`를 사용할 경우: 해당 폴더가 **바로** 프로젝트 폴더가 됩니다. 해당 폴더에 이미 `project.json` 파일이 존재하는 경우, 기존 파일을 덮어쓰지 않고 `_1`/`_2`…와 같은 접미사가 붙은 하위 폴더가 생성되며, 기존 파일이 덮어쓰이지 않습니다. 상대 경로인 `PATH`는 `chloros-cli`를 실행하는 폴더를 기준으로 합니다.

{% hint style="info" %}
**상대 경로 (1.3.0에서 변경됨).** 백엔드가 여는 모든 경로 — 어떤 명령어에서든 (캡처 명령어의 기본값인 `output` 포함) `-o`/`--output`, `--input`, `--burst-dir`, `--profile`, `--save` 및 `--mask` —는 `chloros-cli`를 실행한 폴더를 기준으로 해석되므로, `-o captures/`는 `<current folder>/captures/`에 데이터를 기록합니다. 1.2.0 버전에서는 백엔드가 자체 설치 폴더에서 경로를 해결했으나, 이로 인해 &quot;액세스가 거부되었습니다&quot; 오류가 발생했습니다. 원격 백엔드(`CHLOROS_BACKEND_URL`)의 경우, 경로는 입력된 대로 전송되며 해당 머신에서 해결됩니다.
{% endhint %}

{% hint style="warning" %}
**Windows
: 긴 경로 기능이 켜져 있지 않은 한 프로젝트 폴더 이름을 짧게 유지하십시오.** Windows
에서 긴 경로 기능이 꺼져 있는 경우(Windows
기본 설정), 파일 경로는 260자 이상일 수 없으며, 제품 폴더는 프로젝트 폴더 아래로 최대 약 130자까지 위치할 수 있습니다. `process`는 시작하기 전에 다음 사항을 확인합니다. 프로젝트 폴더가 너무 깊으면 **&quot;이 컴퓨터에 대해 프로젝트 폴더가 너무 깊습니다&quot;**라는 오류 메시지와 함께 거부되며, 허용되는 가장 긴 프로젝트 폴더 이름과 긴 경로 활성화 방법이 표시됩니다. 이 경우 아무 작업도 처리되지 않습니다. 더 짧은 `-o`(또는 `set-project-folder`)를 사용하거나, Windows
긴 경로 기능을 활성화(설치 프로그램의 **Windows
긴 경로 활성화** 옵션, 기본적으로 선택됨)한 후 Chloros
를 다시 시작하십시오.
{% endhint %}

프로젝트 내에서는 제품이 **카메라별, 그 다음 파일 형식별**로 그룹화됩니다:

```
<project>/
├── project.json
├── calibration_data.json
└── LATT-M3M-L41-F550/                  # one folder per camera model+lens+filter
    ├── tiff16/
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── NDVI_Index_Images/           # one folder per requested index
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

LATTICE의 경우 카메라 폴더는 `LATT-<sensor>-<lens>-F<filter>`(캡처된 이미지의 EXIF 정보인 `Model`와 일치)이며, Survey3
의 경우 `<model>_<filter>` (예: `Survey3N_RGN`)로 구성됩니다. 형식 폴더는 `--format`에 이어 `tiff16`, `tiff8`, `png8`, `jpg8` 또는 `tiff32`(`TIFF (32-bit, Percent)`의 경우).

다중 스펙트럼 M3C 카메라의 경우, 제품 폴더(예: `tiff16/clamp_masks/Reflectance_Calibrated_Images/` 및 `tiff32/clamp_masks/Radiance_Images/`) 옆에 `clamp_masks/` 폴더가 형식 폴더에 나타날 수도 있습니다. 이 폴더에는 밴드 언믹스(unmix) 과정에서 음수 값이 0으로 클램핑된 복사도 및 반사도 제품의 픽셀별 마스크가 포함되어 있습니다. [클램프 마스크](output-image-formats.md#clamp-masks)를 참조하십시오. 제품 폴더에는 마스크가 포함되지 않습니다.

{% hint style="info" %}
**내보낸 모든 산출물은 원본 파일의 이름을 그대로 유지합니다.**`capture_..._raw.tif`의 복사도 여전히 `capture_..._raw.tif`라는 이름을 가지며, 단지 `tiff32/Radiance_Images/`에 위치할 뿐입니다.**폴더가 제품을 식별하는 것이지, 파일 이름이 아닙니다**. 따라서 `*radiance*`와 같은 접미사가 아닌 디렉터리를 대상으로 글로브(glob)를 사용하십시오.
{% endhint %}

### 실제로 사용할 옵션

| 플래그 | 기본값 | 기능 |
| --- | --- | --- |
| `-o, --output PATH` | 기본 프로젝트 폴더 | 프로젝트 폴더 위치 (위 참조). |
| `-n, --project-name NAME` | 타임스탬프 | 프로젝트 이름. |
| `--format FMT` | `TIFF (16-bit)` | `TIFF (16-bit)`, `TIFF (8-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)` 중 하나. Chloros
는 해상도를 높이지 않습니다: 모든 8비트 이미지에 대해 더 깊은 형식은 `TIFF (8-bit)`로 기록되며, 이에 대한 설명은 한 번만 표시됩니다. |
| `--indices NAME [NAME ...]` | 없음 | 내보낼 식생 지수 ([식생 지수](#vegetation-indices) 참조). |
| `--debayer {standard,texture-aware}` | `standard` | `texture-aware` = 신경망 디베이어, 처리 속도가 느리며, 최고 품질 (Chloros
+, CUDA GPU 필요: GPU가 없는 경우 `standard`가 사용됨; LATTICE 캡처의 경우 항상 `standard`가 적용되므로 이 옵션은 적용되지 않음). |
| `--vignette / --no-vignette` | 켜기 | 비네팅 보정. |
| `--reflectance / --no-reflectance` | 켜기 | 반사율 보정; LATTICE의 경우 이는 반사율 곱셈 기능의 토글이기도 합니다. |
| `--input-level {auto,raw,debayered,processed}` | `auto` | LATTICE TIFF에 대한 파이프라인 진입점을 강제 적용합니다. |

그 외 모든 항목(표적 감지 조정, PPK, 노출 핀, 배열 정렬 플래그 등)에 대해서는 [CLI
참조 문서의 `process` 섹션](reference/cli-reference.md)을 참조하십시오.

***

## 내보낼 항목 선택 (LATTICE 제품)

LATTICE 처리는 **한 번의 패스로 해당되는 모든 제품**으로 분기됩니다. 제품별 4개의 토글은 모두**기본적으로 ON**으로 설정되어 있습니다. `--no-` 양식을 사용하여 하나를 비활성화할 수 있습니다:

| 토글 | 제품 |
| --- | --- |
| `--debayered` | 선형 디모자이크 → `Debayered_Images/`: 캘리브레이션 팩의 다크 오프셋을 제거한 후, 선형성 보정, 플랫 필드 또는 언믹스 없이 에지 인식 데모자이킹을 수행합니다. 이는 캡처가 `debayered` 레벨에서 저장하는 파일과 동일합니다. 센서의 기본 방향(XMP에 기록됨)을 유지합니다. |
| `--preview` | 디스플레이 미리보기(화이트 밸런스 + 감마; 다중 스펙트럼용 가색 확장) → `Preview_Images/` |
| `--radiance` | float32 방사도, W/m²/sr/nm → `Radiance_Images/` (항상 `tiff32/`) |
| `--reflectance` | uint16 반사율, Pix4D 호환 → `Reflectance_Calibrated_Images/` |

RGB
마스터 카메라는 항상 디베이어링 처리된 데이터와 미리보기 데이터만 출력합니다. 광대역 센서의 경우 대역별 복사도/반사율은 의미가 없으므로, 해당 토글은 이 카메라들에 대해 아무런 동작도 수행하지 않습니다. Survey3
의 `.raw`는 토글을 무시하고 표준 반사도/타겟 경로를 따릅니다.

```bash
# Radiance only — no DAQ downwelling needed
chloros-cli process ~/captures/lattice_flight --no-debayered --no-preview --no-reflectance
```

**`--reflectance-source {auto,target,daq}`** (기본값 `auto`)는 반사율 기준을 선택합니다: `auto`는 QA를 통과한 프레임 내 [보정 타겟](calibration-targets.md)을 절대 기준점으로 삼으며, 타겟이 없을 경우 DAQ 광센서의 하향 분할값 (ρ = π·L/E)로 대체합니다(플래그가 설정된 QA 통과 타겟은 측정 점수에서 DAQ와 경쟁합니다); `target`는 엄격한 모드(DAQ 대체 없음)입니다; `daq`는 DAQ가 우선합니다. 단위별 측정된 타겟 스캔은 `--target-reflectance-dir`로 제공될 수 있습니다.

{% hint style="info" %}
**반사율 픽셀 읽기:**ρ = 1.0을 의미하는 DN은**소스별**입니다 — LATTICE 파일은 XMP에 `Chloros:PixelScale=32768` 태그를 삽입합니다; Survey3
파일은 65535를 사용하며 `Chloros:PixelScale` 태그를 포함하지 않습니다(다른 `Chloros:*` 처리 태그는 포함합니다). 상수를 가정하기보다는 태그를 읽고 그 값으로 나누십시오. 자세한 내용과 의도적으로 스케일이 적용되지 않은 하나의 예외 사례는 [CLI
참조](reference/cli-reference.md)에 나와 있습니다.
{% endhint %}

**처리는 항상 `raw`에서 시작됩니다.** 파생 제품(디베이어링/방사도/반사도 내보내기)은 파이프라인을 통해 다시 입력되지 않습니다. 이를 재수입하여 처리하면 보정 계산이 이중으로 적용되므로, Chloros
는 이를 건너뛰며 해당 사실을 명시합니다. `--input-level`는 정말로 진입점을 강제로 지정해야 할 때를 대비해 의도적으로 마련된 비상 탈출구입니다.

***

## 실행이 실패할 때

1.2.0 버전부터, `process`는 표시할 결과가 없어도 “성공”으로 처리되지 않고 명확하게 실패를 알립니다:

* **산출물을 요청했으나 아무것도 기록하지 않은**실행(단, `project.json` 및 `calibration_data.json`만 해당)은 `Processing finished but wrote no image products.`를 출력하고**0이 아닌 값으로 종료**되므로, 따라서 스크립트에서 이를 감지할 수 있습니다. 일반적인 원인: 입력 폴더가 캡처로 인식되지 않았거나(레이아웃 및 `--input-level` 확인), 요청된 모든 산출물이 해당 카메라에 적용 불가능한 경우입니다(예: RGB
전용 카메라에서 방사도/반사도 요청).
* **의도적으로 메타데이터만 처리하는 실행**(모든 산출물 토글 해제, `--indices` 없음)은 여전히 성공으로 간주됩니다. 이 경우 빈 이미지 출력이 올바른 결과입니다.
* `--verbose`를 사용하여 다시 실행하고, 백엔드 로그에서 `[LATTICE-EXPORT]` / `[EXPORT-CHECK]` 행을 확인하십시오. 이 행들은 카메라별 건너뛰기 사유를 설명합니다.
* 긴 경로 기능이 비활성화된 Windows
환경에서, 260자 경로 제한을 초과하는 깊이의 프로젝트 폴더는 처리 시작 **전**에 거부되며, 제한 내에 맞는 가장 긴 프로젝트 폴더가 사용됩니다([출력 파일 저장 위치](#where-the-outputs-land) 참조).
* 캡처 명령어(`lattice capture`, `lattice array-capture` 및 기타 캡처 계열)는 쓰기 실패한 모든 파일의 이름을 나열한 후 `1`로 종료되며, 드라이브가 가득 차면 반복 캡처가 중지됩니다.

종료 코드: `0` 성공 · `1` 일반 오류 · `2` 인자 오류 · `130` Ctrl+C로 중단됨.

***

## 식생 지수

하나 이상의 사전 설정 이름을 지정하여 `--indices`를 실행하면, 각 지수는 별도의 `<INDEX>_Index_Images/` 폴더에 저장됩니다:

```bash
chloros-cli process ~/images/flight_001 --indices NDVI GNDVI OSAVI
```

`process --indices`가 허용하는 22개의 사전 설정 이름:

`NDVI` `GNDVI` `NDRE` `OSAVI` `SAVI` `MSAVI2` `EVI` `MSR` `TDVI` `LAI` `GCI` `GRVI` `GSAVI` `GOSAVI` `NLI` `MNLI` `RDVI` `WDRVI` `CVI` `ENDVI` `GLI` `VARI`

프로젝트에 아직 나열되지 않은 이름이 프로젝트에 추가됩니다. 사전 설정된 이름이 아니거나 카메라의 대역으로 생성할 수 없는 이름은 해당 이름을 명시하는 경고와 함께 건너뜁니다. CLI
가 프로젝트의 현재 인덱스 목록을 읽을 수 없는 경우, 목록을 대체하지 않고 그대로 두며 경고 메시지를 표시합니다.

{% hint style="warning" %}
**세 개의 인덱스 목록이 존재하므로 혼동하지 마십시오.**GUI의 ‘프로젝트 설정’ 드롭다운 메뉴에는 27개의 공식이 있습니다(`FCI1`, `FCI2`, `GARI`, `GEMI`, `LCI` — 이 다섯 개는 GUI 전용이며 `--indices`에는**적용되지** 않습니다). 라이브/오프라인 `lattice index --preset` 명령어는 별도의 22개 프리셋 목록을 사용합니다. 공식 및 대역 계산에 대한 내용은 [다중 스펙트럼 지수 공식](project-settings/multispectral-index-formulas.md)에 설명되어 있습니다.
{% endhint %}

***

## DAQ 광 센서: 간략한 소개

`daq pool-*` 제품군은 백엔드의 영구 풀을 통해 MAPIR
DAQ 스펙트럼 센서(USB 기반 DAQ-U, BLE 기반 DAQ-M, 이더넷 기반 DAQ-E)를 구동합니다. GUI, CLI
및 SDK
는 모두 하나의 라이브 핸들을 공유합니다. **`pool-*`는 제공되는 CLI
에서 지원되는 DAQ 경로입니다**; 참고할 수 있는 다른 `daq` 하위 명령어들은 MAPIR
내부 전용 소스 표면이며, `pool-*`를 가리키는 명시적인 오류 메시지와 함께 종료됩니다.

```bash
# 1. Open a pooled session (pick the line matching your sensor)
chloros-cli daq pool-connect                              # smart-detect
chloros-cli daq pool-connect --port COM3                  # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF      # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-xxx.local   # DAQ-E by hostname (reliable)

# 2. List pooled sensors and their ids
#    (DAQ-U ids look like 'CB-7C-A8-2E-5F'; DAQ-E ids like 'daq-e-def330')
chloros-cli daq pool-list

# 3. Read the latest calibrated spectrum (W/m²/nm)
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F

# 4. Record a calibrated .daq file for 60 s
chloros-cli daq pool-record --sensor-id CB-7C-A8-2E-5F --duration 60 \
  -o ~/Documents/spectra --device-name "field-A"

# 5. Release
chloros-cli daq pool-disconnect --sensor-id CB-7C-A8-2E-5F
```

`pool-record`는 `--duration`가 없으면 `pool-record --stop`까지 실행되며; 기본 출력 디렉터리는 **백엔드 시스템**의 `~/Documents/DAQ Live View/`입니다. 캡 보정 프로파일은 연결 시점에 선택되며(`--cap-id`, 백엔드 기본값 `sunshine_cosine`), `pool-set-cap` — 캡 프로파일 및 센서의 보정 범위는 본 매뉴얼의 DAQ 장에서 다루고 있습니다.

{% hint style="warning" %}
**다중 NIC 호스트에서의 DAQ-E:** 부팅 후 첫 번째 `pool-connect --eth` 자동 검색은 센서가 정상 상태라 하더라도 실패할 수 있습니다. `--eth-host <ip-or-hostname>`가 안정적인 방식이므로, 검색 결과가 없을 때는 항상 이 방식을 사용하십시오.
{% endhint %}

***

## LATTICE 카메라, PTP 및 프로젝트 자동화

`lattice` 계열(45개 이상의 하위 명령어)은 LATTICE 카메라 작업을 종단 간(end-to-end)으로 포괄합니다: 탐색, 단일 캡처, GUI의 스마트 준비(smart-prep) 연결 흐름을 통한 지속적인 동기화 배열, 라이브 브라우저 미리보기, 정렬, 인덱스 계산 및 호스트 NIC 진단. 일부 예시:

```bash
chloros-cli lattice info                                          # discover cameras
chloros-cli lattice capture -o output/                            # one frame, all export types
chloros-cli lattice array-connect --serials SN1,SN2,SN3,SN4       # persistent synced array
chloros-cli lattice array-capture --processing reflectance -o out/
```

이와 함께: `chloros-cli time-sync`는 Chloros
호스트가 실행하는 PTP 그랜드마스터에 대한 정보를 보고하며(LATTICE 카메라와 DAQ-E 센서는 기기 간 타임스탬프를 위해 이 그랜드마스터에 슬레이브로 연결됨), 또한 `chloros-cli project`는 저장된 Chloros
프로젝트를 열고, 스크립트로 작성된 YAML 캡처 레시피를 포함하여 카메라, 어레이 및 센서를 헤드리스 모드로 제어합니다.

이 세 가지 계열(`lattice`, `project`, `daq pool-*`)는 **원격** 백엔드를 제어하기 위한 `CHLOROS_BACKEND_URL`를 지원하는 유일한 제품군이기도 합니다. 핵심 명령어는 항상 로컬 머신을 대상으로 합니다.

자세한 사용 방법은 이 설명서의 LATTICE 장에 수록되어 있으며, 모든 플래그는 [CLI
참조](reference/cli-reference.md)에 나와 있습니다.

***

## 문제 해결: 상위 5가지

| 증상 | 해결 방법 |
| --- | --- |
| `Login required` 또는 예약된 작업이 `Email:` 프롬프트에서 멈춤 | 이 컴퓨터에서 `chloros-cli login EMAIL 'PASSWORD'`를 한 번 실행하십시오. 캐시된 세션이 없는 명령은 즉시 실패하지 않고 대화형 모드로 실행됩니다. |
| `backend unreachable` | Chloros
데스크톱 앱을 시작하거나 백엔드를 직접 시작하십시오: Windows
에서는 `"C:\Program Files\Chloros\backend\chloros-backend.exe"`(설치 폴더 내의 `backend` 폴더; `PATH`에는 없음), Linux
에서는 `chloros-backend`. `chloros-cli backend status`는 실행 중인 것이 있는지 표시하며; `lattice`/`project`/`daq pool-*`는 절대로 백엔드를 시작하지 않습니다. 원격 백엔드를 지정하는 경우, `CHLOROS_BACKEND_URL`를 확인하십시오. |
| 배열 연결 차단: `FRAMES WILL DROP` / `Reduce ROI to enable` | 호스트 NIC 수신 링이 기본값으로 재설정됨 — 일반적으로 NIC 드라이버 업데이트 후, 이전에 정상 작동하던 시스템이 연결을 거부하는 가장 흔한 원인입니다. **관리자 권한** 터미널에서 `chloros-cli lattice network --fix`를 실행하거나(또는 `ReceiveBufferLen=256`, `PendingReceives=64`를 설정하십시오); 참조 문서의 *호스트 NIC 설정 및 튜닝*을 참조하십시오. |
| `daq` 하위 명령이 종료됨: &quot;전체 DAQ 패키지가 필요합니다…&quot; | 백엔드를 통해 동일한 센서를 제어하는 `daq pool-*` 명령어를 사용하십시오: 연결, 스트리밍, 기록, 캡처 선택, 그리고 DAQ-E의 경우 네트워크 설정, 재시작 및 펌웨어 업데이트(또는 Python
에서 `chloros_sdk.connect_daq_sensor()`). |
| Jetson은 대용량 폴더 처리 전에 스왑 경고 메시지를 출력합니다 | 파일 기반 스왑을 추가하십시오 — CLI
는 실행해야 할 정확한 `fallocate`/`swapon` 명령어를 표시합니다. |

***

## 도움말 보기

```bash
chloros-cli --help              # top-level help
chloros-cli process --help      # per-command help
chloros-cli lattice --help
chloros-cli daq --help          # lists the pool-* subcommands
```

* **모든 플래그 및 하위 명령어:** [CLI
참조](reference/cli-reference.md)
* **Python
대응 명령어:** [Python
SDK
](api-python-sdk.md) 및 [SDK Reference](reference/sdk-reference.md)
* **지원:** info@mapir.camera · [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
