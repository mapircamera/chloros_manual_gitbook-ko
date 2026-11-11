# Linux 설치

Chloros는 Linux용으로 `.deb` 패키지 형태로 배포되며, 이 패키지는 CLI, 백엔드 서버 및 Python SDK를 설치합니다(시스템용 `python3`의 경우, 가상 환경(virtualenv)용 버전 일치 휠도 포함됨).

패키지 파일 이름에는 버전과 아키텍처가 포함됩니다. x86_64용은 `chloros_1.3.0_amd64.deb`, arm64(NVIDIA Jetson, JetPack 6)용은 `chloros_1.3.0_arm64_jp6.deb`입니다. 아래 명령어에서는 실제로 다운로드한 파일 이름으로 대체하십시오.

{% hint style="info" %}
**

1.3.0의 새로운 기능: 프로그램이 압축 해제된 상태로 제공됩니다.** 백엔드와 CLI는 단일 자동 압축 해제 파일이 아닌 `/usr/lib/chloros/backend/` 및 `/usr/lib/chloros/cli/`라는 폴더 형태로 설치됩니다. 단일 자동 압축 해제 파일이 아닙니다. 이제 실행할 때마다 임시 폴더로 자동으로 압축을 풀지 않으므로 시작 속도가 훨씬 빨라졌습니다(x86_64 환경에서 백엔드는 이전에는 시작할 때마다 압축을 푸는 데 약 100초가 소요되었습니다) 또한 `/tmp` 폴더에 수 GB의 여유 공간이 더 이상 필요하지 않습니다. amd64용 `.deb`의 크기도 이전의 약 절반 수준으로 줄었습니다.
{% endhint %}

***

## Linux amd64 (x86_64)

### 시스템 요구 사항

| 요구 사항 | 최소 | 권장 |
| --- | --- | --- |
| **배포판** | Ubuntu 22.04 LTS 이상 / Debian 12 이상 | Ubuntu 24.04 LTS |
| **프로세서** | x86_64 (Intel/AMD) | Intel Core i7 이상 |
| **메모리 (RAM)** | 8GB | 16GB 이상 |
| **그래픽 카드** | 없음 (CPU 처리) | 4GB 이상의 VRAM을 갖춘 NVIDIA GPU (12GB 이상 시 `GPU_PARALLEL` 잠금 해제, 7GB 이상 시 단일 이미지 경로에서 Texture Aware 비활성화) |
| **저장 공간** | 2GB 여유 공간 | 10GB 이상의 여유 공간이 있는 SSD |
| **Python** | Python 3.7 이상 (SDK용) | Python 3.10 이상 |

> **Ubuntu 20.04 및 Debian 11은 지원되지 않습니다.** `.deb`의 종속성 목록은
> Chloros 백엔드가 실제로 링크하는 항목에서 파생된 것이며, 여기에는
> focal 및 bullseye 배포판에 포함된 glibc 2.31보다 최신 버전의 `libc6`가 포함되어 있으므로, `apt`는
> 나중에 런타임 시 오류가 발생하는 것을 방지하기 위해 설치 자체를 거부합니다.

### 설치

```bash
sudo apt install ./chloros_1.3.0_amd64.deb
```

{% hint style="info" %}
**`dpkg -i`가 아닌 `apt install ./…`를 사용하십시오.** `apt`는 패키지의 종속성을 해결하고, 권장되는 항목인 Python SDK의 필수 구성 요소인 `python3-numpy` 및 `python3-requests`입니다. `dpkg -i`는 이 두 가지를 모두 설치하지 않습니다. CLI와 백엔드는 독립적으로 구성되어 있어 이 패키지들 없이도 실행되지만, `import chloros_sdk`는 이 패키지들이 필요합니다. 설치 출력에는 누락된 항목이 모두 명시됩니다. `./`를 유지하여, `apt`가 해당 이름의 패키지를 검색하는 대신 파일을 직접 설치할 수 있도록 하십시오. 이 패키지는 또한 `libgl1`를 선언하는데, 이는 번들된 OpenCV에서 로드되며 최소 서버 이미지에는 포함되지 않을 수 있습니다.
{% endhint %}

설치를 확인하십시오:

```bash
chloros-cli --version    # prints "Chloros CLI 1.3.0"
```

<!-- SCREENSHOT-NEEDED: Terminal on Ubuntu 22.04 immediately after `sudo apt install ./chloros_1.3.0_amd64.deb`, showing the full postinst output: the "Chloros installed successfully!" banner, the Usage lines, the "Python SDK:" block ("chloros_sdk 1.3.0 for python3" and the "In a virtualenv: pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl" line), any "GPU Acceleration:" detection line, and the closing "Systemd Service (optional): sudo systemctl enable --now chloros-backend@<user>.service" hint -->

***

## Linux arm64 (NVIDIA Jetson)

### 시스템 요구 사항

| 요구 사항 | 최소 | 권장 |
| --- | --- | --- |
| **플랫폼** | JetPack 6이 설치된 NVIDIA Jetson | Jetson Orin NX 16GB 또는 AGX Orin |
| **JetPack** | JetPack 6.x | 최신 JetPack 6 |
| **메모리 (RAM)** | 8GB (GPU/CPU 공유) | 16GB 이상 공유 (병렬 처리를 위한 최소 요건은 12GB 이상) |
| **저장 공간** | 2GB 여유 공간 | 10GB 이상의 여유 공간이 있는 NVMe SSD |
| **Python** | Python 3.7 이상 (SDK용) | Python 3.10 이상 |

### 설치

```bash
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
chloros-cli --version
```

amd64 `.deb`(Python SDK, 수신 버퍼 및 백엔드 서비스 포함)와 동일한 레이아웃으로, **JetPack 6**(L4T R36)을 실행하는 Jetson Orin / Orin NX / Orin Nano용입니다. 이 패키지의 PyTorch는 Orin용 JetPack 6의 CUDA 빌드이므로, GPU 처리(Texture Aware 디베이어 포함)는 Jetson의 GPU에서 실행됩니다. 이 패키지는 CUDA 자체를 포함하지 않으며, JetPack 6이 설치하는 CUDA 및 cuDNN 라이브러리를 사용합니다(해당 라이브러리가 없는 이미지의 경우 `sudo apt install nvidia-jetpack`). CUDA GPU를 사용할 수 없는 경우, 처리는 CPU에서 실행되며 `--debayer texture-aware`는 표준 디베이어를 사용합니다(백엔드에는 그 이유를 설명하는 한 줄의 `[DEBAYER] … needs a CUDA GPU …` 로그가 기록됩니다). Jetson의 메모리, 발열 및 현장 배포 관련 동작에 대해서는 [NVIDIA Jetson 가이드](nvidia-jetson-guide.md)를 참조하십시오.***

## Python SDK 설치 (모든 Linux)**`.deb`가 SDK를 자동으로 설치합니다.** 두 패키지(amd64 및 Jetson) 모두 시스템용 `python3`로 설치되므로, `import chloros_sdk`는 설치 직후 pip나 네트워크 연결 없이도 바로 작동합니다. 이 패키지의 구성 요소(`chloros_sdk`, `lattice_sdk`, `lattice_cam`, `arena_api` 및 프로젝트 로더)는 해당 패키지가 소유하는 `/usr/lib/python3/dist-packages`에 포함됩니다. 업그레이드 시 이 패키지들이 대체되며, 패키지를 제거하면 이 패키지들도 함께 제거됩니다. 버전은 항상 설치한 CLI 및 백엔드와 일치합니다.

SDK 자체의 필수 구성 요소인 `python3-numpy` 및 `python3-requests`는 **권장 사항일 뿐, 필수 사항은 아닙니다**: `sudo apt install ./chloros_<version>_<arch>.deb`는 이를 설치하지만, `dpkg -i`는 설치하지 않습니다. 설치 출력 결과에는 SDK 버전 `python3`가 가져오는 항목, 누락된 필수 구성 요소, 그리고 `python3`의 경로 상에 있는 다른 위치의 `chloros_sdk` 중 패키지의 사본을 가리는 항목(예: 이전에 `pip install --user`로 설치된 것)을 보고합니다.

시스템의 패키지를 인식하지 못하는 **virtualenv** 환경에서는 번들된 휠을 해당 환경 내에 (완전히 오프라인 상태로) 설치하십시오:

```bash
pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

**PyPI에서** — Chloros 패키지가 설치되어 있지 않은 컴퓨터에서만 사용하며, 해당 SDK가 다른 컴퓨터의 백엔드를 구동하는 경우( [원격 백엔드 모드](../reference/sdk-reference.md#remote-backend-mode-pip-only-host-via-tunnel) 참조):

```bash
pip install chloros-sdk
```

pip 설치(virtualenv 또는 PyPI)를 위한 선택적 추가 항목:

| 추가 항목 | 명령어 | 추가 내용 |
| --- | --- | --- |
| `progress` | `pip install chloros-sdk[progress]` | 실시간 진행 상황 스트리밍용 `sseclient-py` |
| `camera` | BLE(DAQ-M) 전송용 `pip install chloros-sdk[camera]` | BLE(DAQ-M) 전송용 `bleak` |

SDK 확인:

```bash
python3 -c "import chloros_sdk; print(chloros_sdk.__version__)"
```

{% hint style="info" %}
Python SDK는 `.deb`가 설치한 백엔드와 로컬 HTTP API(`http://127.0.0.1:5000`)를 통해 통신합니다. `ChlorosLocal`와 연결 헬퍼는 필요할 때 이를 자동으로 시작하지만, 저장된 프로젝트의 `connect_all()`는 절대 시작하지 않으므로, 따라서 헤드리스 머신에서는 백엔드를 [서비스](#always-on-ptp-for-headless-hosts)로 실행하십시오로 실행하십시오. 항상 `localhost` 대신 리터럴 IPv4 주소를 사용하십시오. `localhost`는 `::1`로 해결될 수 있으며, 요청당 약 2초의 시간이 소요될 수 있습니다.
{% endhint %}

***

## 초기 설정

### 1. 로그인

CLI 및 SDK에 액세스하려면 유료 Chloros+ 티어(**Copper** 이상)가 필요하며, 이는 서버 측에서 강제 적용됩니다. 로그아웃된 호출자에게는 `401 AUTH_REQUIRED`가, 무료 티어(Iron) 호출자에게는 `403 PLAN_UPGRADE_REQUIRED`가 할당됩니다.

```bash
chloros-cli login your@email.com 'your-password'
```

인증 정보는 `~/.chloros/user_session.json`에 캐시됩니다.

{% hint style="info" %}
**업그레이드 시 로그인 정보는 유지되며, 제거 시 삭제됩니다.** 기존 설치본(`chloros-cli update`를 통한 설치 포함) 위에 최신 버전의 `.deb`를 설치하면 `~/.chloros/user_session.json`와 캐시된 라이선스가 유지되므로, 헤드리스 CLI/SDK 작업이 계속 실행됩니다. 패키지(`sudo apt remove chloros`)를 제거하면 해당 컴퓨터의 모든 사용자에 대해 저장된 로그인 정보와 라이선스 캐시가 삭제되므로, 새로 설치할 때는 `chloros-cli login`가 다시 필요합니다.
{% endhint %}

### 2. 라이선스 상태 확인

```bash
chloros-cli status
```

`chloros-cli status`는 모든 티어(무료 포함)에서 작동하므로, 액세스가 가능한지 또는 불가능한지 그 이유를 항상 확인할 수 있습니다.

### 3. 시스템 진단 실행

```bash
chloros-cli selftest
```

7가지 검사가 순서대로 실행되며, 그중 하나라도 실패하면 명령어가 0이 아닌 값을 반환합니다:

| # | 검사 항목 | 확인 사항 |
| --- | --- | --- |
| 1 | **버전** | CLI가 자체 버전(`v1.3.0`)을 보고합니다. |
| 2 | **포트 사용 가능** | 포트 5000이 비어 있거나, *또는* 정상적인 Chloros 백엔드에서 이미 응답한 경우(이 경우 통과로 간주됨). |
| 3 | **백엔드 시작** | 백엔드 바이너리가 실행됩니다. |
| 4 | **API 테스트 (`/api/test`)** | 백엔드가 `status: ok`로 응답합니다. |
| 5 | **시스템 정보** | `/api/system-info`에서 `GPU: <name>, CUDA: <bool>, PyTorch: <version>`를 출력합니다. |
| 6 | **노이즈 제거기 모델** | `*.pth.enc` 모델을 찾습니다 (Linux의 경우: `/usr/lib/chloros/models`). |
| 7 | **CUDA + 노이즈 제거기**| Texture Aware를 실제로 사용할 수 있습니다 — CUDA**및** 최소 하나의 모델 파일이 필요합니다. |

실행은 `N/7 checks passed`로 종료되며, 실패한 항목은 이름으로 나열됩니다.

### 4. 첫 번째 데이터셋 처리하기

```bash
chloros-cli process ~/datasets/flight001
```

***

## 파일 및 디렉터리

### 사용자별

Chloros는 인증 정보와 CLI 구성을 단일 크로스 플랫폼 디렉터리인 **`~/.chloros/`**(Windows의 경우 `%USERPROFILE%\.chloros\`)에 저장합니다. 반면, 두 개의 Linux 전용 캐시는 XDG 규약을 따르며, 설정 시 `XDG_CONFIG_HOME` / `XDG_CACHE_HOME` 경로를 사용합니다.

| 경로 | 용도 |
| --- | --- |
| `~/.chloros/user_session.json` | `chloros-cli login`에 의해 기록된 로그인 세션 캐시 (패키지 업그레이드 시에도 유지되며, 패키지가 제거되면 삭제됨) |
| `~/.chloros/working_directory.txt` | 기본 프로젝트 폴더 재정의 (`chloros-cli set-project-folder` / `get-project-folder` / `reset-project-folder`) |
| `~/.chloros/cli_language.json` | CLI 언어 기본 설정 (`chloros-cli language <code>`) |
| `~/.chloros/user.json` | Windows GUI와 공유되는 언어 설정 — 이 항목(`language`)은 `cli_language.json`보다 우선합니다 |
| `~/.chloros/update_cache.json` | Linux/Jetson 시작 시 업데이트 확인을 위한 1시간 캐시 |
| `~/.chloros/backend.log` | CLI를 통해 백엔드가 실행되었을 때의 백엔드 로그 |
| `~/.chloros/camera_cal/<serial>/<bundle_sha>/` | 일련번호 및 번들 해시를 키로 하는 카메라별 LATTICE 보정 팩 캐시 |
| `~/.chloros/daq_cap_profiles/<u\|m\|e>/<cap_id>.json` | DAQ 캡 보정 프로필에 대한 선택적 사용자 재정의 |
| `~/.config/chloros/system_config.json` | 동적 컴퓨팅 적응(Dynamic Compute Adaptation)에서 가져온 캐시된 하드웨어 프로필 — 이를 삭제하면 새로운 하드웨어 감지를 강제 실행합니다 |
| `~/.cache/chloros/logs/backend_<YYYYMMDD_HHMMSS>.log` | 백엔드 서버 로그, 실행 횟수당 하나의 파일 |
| `~/Documents/MAPIR/Chloros Projects/` | 재정의가 설정되지 않았을 때의 기본 프로젝트 폴더 |

### 시스템 전체

| 경로 | 용도 |
| --- | --- |
| `/usr/bin/chloros-cli` | 래퍼 스크립트 — 번들된 네이티브 라이브러리에 대해 `LD_LIBRARY_PATH`를 설정하고 (해당 폴더가 존재할 경우 `TMPDIR`를 `/mnt/ssd/tmp`로 설정), 이후 실제 바이너리를 실행 |
| `/usr/bin/chloros-backend` | 래퍼 스크립트 — 백엔드에 대해서도 동일하게 처리하며, 이를 설치된 프로덕션 빌드로 표시함 |
| `/usr/lib/chloros/cli/`, `/usr/lib/chloros/backend/` | 컴파일된 CLI와 백엔드로, 압축을 풀어서 설치됩니다(1.3.0부터 추가됨); 프로그램명은 `cli/chloros-cli` 및 `backend/chloros-backend`입니다 |
| `/usr/lib/chloros/chloros-cli`, `/usr/lib/chloros/chloros-backend` | 위의 실제 프로그램을 실행하는 2줄짜리 런처 스크립트 — SDK, CLI 및 구버전 스크립트가 이곳에서 해당 파일을 찾기 때문에 이 경로에 보관됨 |
| `/usr/lib/chloros/arena_runtime/` | LATTICE 카메라에 필요한 Arena SDK 런타임 |
| `/usr/lib/chloros/models/*.pth.enc` | Texture Aware 디베이어에서 사용하는 암호화된 노이즈 제거 모델 |
| `/usr/lib/python3/dist-packages/chloros_sdk/` (및 `lattice_sdk`, `lattice_cam`, `arena_api`, 프로젝트 로더들) | 시스템 `python3`에 설치되고 해당 패키지가 소유한 Python SDK |
| `/usr/lib/chloros/sdk/chloros_sdk-*.whl` | 가상 환경(virtualenv)용으로, 이 빌드와 정확히 일치하는 Python SDK 휠 |
| `/usr/lib/chloros/exiftool` | 번들된 ExifTool 런처 (시스템 exiftool이 존재하지 않는 경우에만 `/usr/local/bin/exiftool`에 심볼릭 링크됨). 1.3.0 버전부터 ExifTool 스크립트는 `/usr/lib/chloros/exiftool-dist/`에 자체 Perl 라이브러리를 포함하여 제공되므로, 시스템 `libimage-exiftool-perl` 패키지 없이도 실행됩니다 |
| `/etc/chloros/update.conf` | `chloros-cli update`가 읽는 업데이트 채널 구성 |
| `/etc/sysctl.d/60-chloros-ptp.conf` | 백엔드가 루트 권한 없이 PTP 포트에 바인딩할 수 있도록 `net.ipv4.ip_unprivileged_port_start = 319`를 설정합니다 |
| `/etc/sysctl.d/60-chloros-gige.conf` | LATTICE 카메라 스트림의 경우 커널 소켓 수신 버퍼 (`net.core.rmem_max`, `net.core.rmem_default`)를 32MB로 늘리고, 설치 시 이를 적용합니다 (이미 더 높은 값으로 설정된 경우 변경되지 않음). 커널 기본값에서는 카메라 패킷이 커널에서 드롭되어 어레이의 프레임 및 프레임 속도가 저하됩니다. `chloros-cli lattice network`는 버퍼 용량이 부족할 때 이를 보고합니다. [Linux 호스트 — 소켓 수신 버퍼](../reference/cli-reference.md#linux-hosts--socket-receive-buffers) 참조 |
| `/etc/ld.so.conf.d/Arena_SDK.conf` | 동적 로더를 `/usr/lib/chloros/arena_runtime`로 지정합니다 |
| `/lib/udev/rules.d/70-chloros-daq.rules` | 로그인한 사용자에게 DAQ-U USB 직렬 브리지 (CP2102N, `10c4:ea60`) |
| `/lib/systemd/system/chloros-backend@.service` | 지정한 사용자로 백엔드를 실행하는 상시 가동 백엔드 서비스(설치됨, **비활성화됨**) — 활성화해야 할 서비스는 [상시 가동 PTP](#always-on-ptp-for-headless-hosts) |
| `/lib/systemd/system/chloros-backend.service` | 백엔드를 **root**권한으로 실행하는 동일한 서비스(설치됨,**활성화되지 않음**). 이 서비스를 활성화한 시스템이 계속 실행되도록 유지하기 위해 남겨둠 |
| `/var/log/chloros/` | root 서비스의 상세 로그로, `adm` 그룹이 읽을 수 있음 (패키지가 완전히 제거되면 삭제됨) |
| `/usr/share/applications/chloros-cli.desktop` | 터미널을 여는 &quot;Chloros CLI&quot; 애플리케이션 메뉴 항목 |

**방화벽 (1.3.0 버전부터 추가됨).** 설치 시점에 `ufw` 또는 `firewalld`가 활성화된 경우, 이 패키지는 UDP 포트 `5002` (DAQ-E 데이터), `5353` (mDNS 검색), `319` 및 `320` (PTP)를 열고, 패키지가 제거되면 해당 규칙을 삭제합니다. 방화벽이 설치되어 있지만 비활성화된 경우, 아무것도 추가하지 않으며 나중에 활성화할 경우 허용해야 할 포트를 표시합니다. TCP 포트는 열지 않습니다. DAQ-E 펌웨어 업데이트 시, 센서가 이미지를 수집하기 위해 TCP를 통해 이 컴퓨터에 다시 연결하는 과정에서 활성화된 방화벽이 해당 연결을 차단하면 `… device never connected back on TCP <port> …` 오류가 발생합니다.

## 백엔드 실행 파일 위치

CLI와 SDK는 백엔드를 자동으로 감지합니다:

| 구성 요소 | 경로 |
| --- | --- |
| CLI | `/usr/bin/chloros-cli` |
| 백엔드 | `/usr/lib/chloros/chloros-backend` (`/usr/lib/chloros/backend/chloros-backend`용 런처) |

`--backend-exe` CLI 플래그나 `backend_exe` SDK 생성자 매개변수를 사용하여 백엔드 경로를 재정의하고, 포트는 `--port`(기본값 `5000`)로 재정의할 수 있습니다.

{% hint style="info" %}
`CHLOROS_BACKEND_URL`는 원격 백엔드에서 **`lattice`**,**`project`**및**`daq pool-*`** 명령어 계열을 원격 백엔드로 가리킵니다. 핵심 명령어(`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`)는 이를 의도적으로 무시하고 항상 `http://127.0.0.1:<port>`를 대상으로 합니다.
{% endhint %}

***

## Linux에서 LATTICE 카메라 및 DAQ 광 센서

live-hardware 명령어 계열은 모두 Linux(amd64 및 Jetson)에서 작동합니다:

* **`chloros-cli lattice`** — LATTICE 카메라 및 동기화된 어레이를 탐색, 연결, 구성하고 데이터를 캡처합니다. `.deb`는 이에 필요한 Arena SDK 런타임을 묶어 동적 로더에 등록합니다.
* **`chloros-cli daq pool-*`** — 백엔드 풀을 통해 DAQ-U/M/E 광 센서에 연결하고, 보정된 스펙트럼을 스트리밍하며, `.daq` 파일을 기록하고 캡 프로필을 교체할 수 있습니다. 또한 DAQ-E의 경우 네트워크 설정을 변경하고, 재시작하며, 펌웨어를 업데이트할 수 있습니다. `chloros-cli daq discover`는 연결 가능한 센서를 검색하고, `chloros-cli daq --help`는 `pool-*` 하위 명령어를 나열합니다.
* **`chloros-cli project`** — 저장된 프로젝트(카메라, 센서 및 처리 설정)를 헤드리스 모드로 실행합니다.
* **`chloros-cli time-sync`** — LATTICE 카메라 및 DAQ-E 센서를 위해 Chloros 백엔드가 실행하는 PTP 그랜드마스터를 확인합니다.

```bash
# DAQ-E at a known address — the reliable path on multi-homed hosts
chloros-cli daq pool-connect --eth-host 192.168.2.50

# DAQ-U over USB serial
chloros-cli daq pool-connect --port /dev/ttyUSB0

# What is connected, then the latest calibrated spectrum as JSON
chloros-cli daq pool-list
chloros-cli daq pool-latest --sensor-id daq-e-a1b2c3 --json
```

`--sensor-id`는 `pool-latest`, `pool-stream`, `pool-record` 및 `pool-set-cap`에 의해 요구되며, `pool-list`는 현재 풀에 있는 ID를 표시합니다.

{% hint style="info" %}
**`--eth-host`는 검색 단계를 건너뜁니다.** `pool-connect --eth`는 mDNS를 통해 DAQ-E를 찾으며, 비활성 ARP 캐시에서도 작동하는 ARP 폴백 기능을 사용합니다. `--eth-host`에 센서의 IP 또는 호스트 이름을 전달하면 해당 센서에 직접 연결됩니다.
{% endhint %}

**DAQ-U 시리얼 권한**은 설치된 udev 규칙(`uaccess` + 그룹 `dialout`)에 의해 처리됩니다. 이미 연결된 센서에 계속 접근할 수 없는 경우, 규칙을 재로드하거나 센서를 다시 연결하십시오:

```bash
sudo udevadm control --reload-rules
sudo udevadm trigger --subsystem-match=tty
```

전체 명령어 세트는 [CLI 참조](../CLI.md)를 참조하십시오.

### 헤드리스 호스트용 상시 작동 PTP

이 패키지는 백엔드용으로 두 개의 systemd 서비스를 설치하지만, 둘 다 활성화되어 있지 않습니다. DAQ-E 센서 및 LATTICE 카메라를 위해 PTP 시간 동기화를 지속적으로 실행해야 하는 헤드리스 Jetson 또는 서버에서, 혹은 SDK 스크립트가 저장된 프로젝트를 연결하는 환경(`connect_all()`는 백엔드를 절대 시작하지 않음)에서는 **사용자 권한으로** 서비스를 활성화하십시오:

```bash
sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
systemctl status chloros-backend@$USER.service
```

`chloros-backend@<user>.service`는 해당 사용자로 백엔드를 실행합니다. 즉, 이 서비스가 기록하는 캡처 데이터는 해당 사용자의 것이며, 상세 로그는 해당 사용자의 `~/.cache/chloros/logs`에 저장되며, 해당 사용자의 `chloros-cli login` (`~/.chloros`)를 사용하므로, 먼저 해당 사용자로 한 번 로그인하십시오. 패키지를 설치하면 사용자 이름이 채워진 이 명령이 출력됩니다.

서비스가 없는 경우, 백엔드(그리고 이에 따른 PTP 그랜드마스터)는 `chloros-cli` 명령어나 SDK 스크립트와 같이 이를 시작한 것이 있을 때만 실행됩니다.

이 유닛은 백엔드를 유닛 내부의 `127.0.0.1:5000` (유닛 내부의 `CHLOROS_HOST` / `CHLOROS_PORT` 환경 설정; `sudo systemctl edit chloros-backend@.service`로 재정의 가능)에 바인딩하고, 오류 발생 시 5초 후에 다시 시작합니다.

**루트 서비스.**`chloros-backend.service`는 백엔드를**루트** 권한으로 실행합니다. 이 기능을 활성화한 시스템이 계속 실행되도록 여전히 설치되어 있지만, 하지만 루트 백엔드가 작성하는 모든 내용은 루트 소유이므로, 사용자는 자신의 캡처를 삭제할 수 없습니다. 해당 로그 파일은 `/var/log/chloros`에 있으며, `adm` 그룹에서 읽을 수 있습니다. 사용자 계정으로 전환하려면:

```bash
sudo systemctl disable --now chloros-backend.service
sudo systemctl enable --now chloros-backend@$USER.service
```

**서비스의 백엔드는 공유되며, 클라이언트에 의해 중지되지 않습니다.** `chloros-cli` 및 SDK 스크립트는 백엔드를 발견하는 대로 사용합니다: `process`는 백엔드에서 실행되며, 마지막 프로젝트 대신 자체 프로젝트를 불러옵니다. CLI가 다른 경우라면 백엔드를 재시작했을 상황(`--restart`, 또는 임포트 시 아무것도 찾지 못한 `process`)에서, 대신 유닛 이름을 지정하고 `sudo systemctl restart <unit>`를 출력한 뒤 종료됩니다(`1`). 이는 이러한 방식으로 중지된 백엔드는 다시 시작되지 않기 때문입니다. `sudo systemctl restart chloros-backend@$USER.service`를 사용하여 직접 다시 시작하십시오. 서비스의 백엔드는 유휴 상태에서도 스스로 중지되지 않으며, `chloros-cli backend stop`는 이를 거부하고 `systemctl stop` 명령을 표시합니다.**PTP가 포트를 할당받는 방식.** PTP는 UDP 319/320 포트를 사용하며, 두 포트 모두 일반적인 1024 특권 포트 하한선보다 낮습니다. 이 패키지의 `postinst`는 `/etc/sysctl.d/60-chloros-ptp.conf`에 `net.ipv4.ip_unprivileged_port_start = 319`를 기록하며, 이를 통해 백엔드는 사용자의 계정으로 실행되는 동안 해당 포트에 바인딩할 수 있습니다. 1.3.0 버전부터 이 패키지는 백엔드에 파일 권한(`setcap`)을 설정하지 않으며, `libcap2-bin`는 더 이상 의존성이 아닙니다. 압축을 푼 백엔드에서 파일 권한이 설정되어 있으면 백엔드가 시작되지 않으므로, 따라서 직접 추가하지 마십시오.***

## Bash 스크립팅 예제

{% hint style="info" %}
**스크립팅에 적합한 종료 코드.**`chloros-cli process`는 성공 시 `0`를 종료하고**실패 시 0이 아닌 값을 반환합니다 — 이미지 제품을 요청했으나 아무것도 생성하지 못한 실행도 포함됩니다** (이 경우 `Processing finished but wrote no image products.`를 출력하고 프로젝트 폴더 이름 및 일반적인 원인을 표시합니다). 실행에 성공하면 작성된 이미지 제품의 개수를 보고합니다(`Image products written: N`, 마스크가 작성된 경우에는 `(plus M clamp masks)` 추가). 종료 코드: `0`(성공), `1`(실패), `2`(인수 오류), `130`(중단됨).
{% endhint %}

### 여러 데이터셋 처리

```bash
#!/bin/bash
for dataset in ~/datasets/2026/*/; do
    echo "Processing $(basename "$dataset")..."
    if chloros-cli process "$dataset" --format "TIFF (32-bit, Percent)"; then
        echo "Done: $(basename "$dataset")"
    else
        echo "FAILED: $(basename "$dataset")" >&2
    fi
done
```

### 사용자 지정 설정으로 처리

```bash
#!/bin/bash
chloros-cli process ~/datasets/field_a \
    --output ~/output/field_a \
    --format "TIFF (32-bit, Percent)" \
    --indices NDVI GNDVI OSAVI \
    --debayer texture-aware \
    --no-vignette
```

유효한 `--format` 값은 정확히 네 개이며, 공백이 포함되어 있으므로 항상 따옴표로 묶어야 합니다:

| `--format` 값 | 출력 폴더 |
| --- | --- |
| `TIFF (16-bit)` *(기본값)* | `tiff16` |
| `TIFF (8-bit)` | `tiff8` |
| `TIFF (32-bit, Percent)` | `tiff32` |
| `PNG (8-bit)` | `png8` |
| `JPG (8-bit)` | `jpg8` |

`--debayer`는 `standard`(기본값) 또는 `texture-aware`(Chloros+)를 허용합니다.

### Cron을 이용한 자동 처리

```cron
# Process any new datasets at 2 AM daily
0 2 * ** /usr/bin/chloros-cli process /data/incoming --output /data/processed >> /var/log/chloros.log 2>&1
```

### Python SDK 예제

```python
from chloros_sdk import process_folder

# One-line processing
result = process_folder(
    "/home/user/datasets/flight001",
    indices=["NDVI", "OSAVI"],
    export_format="TIFF (32-bit, Percent)"
)
```

***

## 문제 해결

### 설치 후 CLI를 찾을 수 없음

```bash
# Check if the binary exists
which chloros-cli
ls -la /usr/bin/chloros-cli

# List everything the package installed
dpkg -L chloros

# Reload your shell
source ~/.bashrc
```

### 권한 거부

```bash
sudo chmod +x /usr/bin/chloros-cli
sudo chmod +x /usr/lib/chloros/chloros-backend /usr/lib/chloros/backend/chloros-backend
```

### 파일 권한 추가 후 백엔드가 시작되지 않음

백엔드에서 `setcap`를 실행하지 마십시오. 1.3.0 버전부터 이 패키지는 파일 기능(file capabilities)을 설정하지 않으며, PTP의 319/320 포트는 [PTP가 포트를 확보하는 방법](#always-on-ptp-for-headless-hosts)에 설명된 `sysctl` 설정에서 비롯됩니다. 파일 기능(file capability)을 설정하면 압축이 해제된 백엔드가 보안 실행 모드로 전환되어 자체 라이브러리를 찾을 수 없게 되므로, 백엔드가 시작되지 않습니다. 파일 기능을 추가한 경우, 이를 제거하십시오:

```bash
sudo setcap -r /usr/lib/chloros/backend/chloros-backend
```

### PTP가 시작되지 않음 / 포트 319에 바인딩할 수 없음

비특권 포트 하한값이 낮아졌는지 확인하고, 그렇지 않은 경우 현재 부팅에 대해 다시 적용하십시오:

```bash
sysctl net.ipv4.ip_unprivileged_port_start     # expect 319
sudo sysctl -w net.ipv4.ip_unprivileged_port_start=319
```

그런 다음 그랜드마스터를 확인하십시오:

```bash
chloros-cli time-sync status
chloros-cli time-sync peers
```

### `lattice` 명령이 &quot;LATTICE 카메라 드라이버를 찾을 수 없음&quot; 오류로 실패함

백엔드가 함께 설치된 LATTICE 카메라 런타임을 로드할 수 없습니다. 모든 `lattice` 명령은 백엔드에서 실행되며, CLI 자체에는 카메라 드라이버가 포함되어 있지 않습니다. Chloros(`sudo apt install --reinstall ./chloros_1.3.0_<arch>.deb`)를 재설치하십시오: `.deb`는 백엔드 옆(`/usr/lib/chloros/arena_runtime`)에 런타임을 설치하고, 동적 로더에 이를 등록합니다. 등록 여부를 확인하려면:

```bash
cat /etc/ld.so.conf.d/Arena_SDK.conf     # expect /usr/lib/chloros/arena_runtime
ls /usr/lib/chloros/arena_runtime | head
```

### 백엔드 시작 실패

```bash
# Check if port 5000 is already in use
lsof -i :5000

# Kill any existing process on port 5000
kill $(lsof -t -i :5000)

# Try starting with a different port
chloros-cli --port 5001 process ~/datasets/flight001
```

시작에 실패한 백엔드의 로그 정보는 `~/.cache/chloros/logs/`에 있습니다. 포트 5000이 백엔드 서비스(`systemctl status chloros-backend@$USER.service`)에 속해 있다면, 해당 프로세스를 강제 종료하기보다는 서비스를 다시 시작하십시오.

### CUDA 감지되지 않음

```bash
# Check NVIDIA driver installation (x86_64)
nvidia-smi

# Check CUDA availability
nvcc --version

# On Jetson, check the L4T release (JetPack 6 is R36) and that JetPack's CUDA libraries are installed
cat /etc/nv_tegra_release
sudo apt install nvidia-jetpack
```

`chloros-cli selftest` 파일의 한 줄(`GPU: <name>, CUDA: <bool>, PyTorch: <version>`)에 동일한 내용이 기록되어 있습니다.

### 누락된 공유 라이브러리

```bash
sudo apt-get update
sudo apt-get install -f

# Check for missing libraries (the program itself, not the launcher script)
ldd /usr/lib/chloros/backend/chloros-backend | grep "not found"
```

### SD 카드 시스템에서의 느린 시작

1.3.0 버전부터 백엔드와 CLI는 압축 해제된 상태로 설치되므로, 더 이상 실행할 때마다 임시 디렉터리로 파일을 추출하지 않습니다. 해당 폴더가 존재하면 임시 파일은 자동으로 `/mnt/ssd/tmp`로 저장되며, 그렇지 않은 경우 빠른 파일 시스템으로 경로를 지정할 수 있습니다:

```bash
export TMPDIR=/mnt/nvme/tmp
```

***

## Linux에서 Chloros 업데이트 Linux에서 Chloros 업데이트

`update` 명령어는 Linux/Jetson 전용입니다. 이 명령어는 `/etc/chloros/update.conf`에서 구성된 업데이트 채널에 게시된 버전을 확인하고, 해당 버전에 맞는 `.deb`를 다운로드하여 설치할 수 있도록 제안합니다:

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

Linux/Jetson 환경에서 CLI는 매 시작 시 비차단 방식의 업데이트 확인을 수행하며 (결과는 `~/.chloros/update_cache.json`에 1시간 동안 캐시됨)하며, 더 새로운 버전이 존재할 경우 `Update available: vX.Y.Z`를 출력합니다. 업데이트 후에도 설정, 프로젝트 및 로그인 정보는 유지됩니다.

패키지 업그레이드 시 설치 과정 동안 모든 백엔드 서비스가 중지되므로, 실행 중인 스크립트는 카메라 및 센서 연결을 잃게 되며, 이후 활성화된 각 서비스가 다시 시작됩니다(수동으로 시작되었으나 활성화된 적이 없는 서비스는 재부팅 후와 마찬가지로 중지된 상태로 유지됩니다).

## 제거

```bash
sudo apt remove chloros
```

제거 시 모든 백엔드 서비스가 중지 및 비활성화되며, 비특권 포트 하한값(1024)이 기본값으로 복원되고, 수신 버퍼 설정 파일(실행 중인 값은 재부팅 전까지 유지됨), 번들된 exiftool 심볼릭 링크, Arena 로더 구성, 패키지가 추가한 방화벽 규칙 및 설치된 Python SDK를 제거하고, 캐시된 자격 증명을 지웁니다. 프로젝트와 `~/.chloros/` 데이터 파일은 그대로 유지됩니다. `sudo apt purge chloros`는 또한 `/var/log/chloros`를 삭제합니다.

***

## 다음 단계

* [NVIDIA Jetson 가이드](nvidia-jetson-guide.md) — Jetson 전용 최적화 및 배포
* [CLI : 명령줄](../CLI.md) — CLI 가이드
* [API : Python SDK](../api-python-sdk.md) — SDK 가이드
* [CLI 참조](../reference/cli-reference.md) 및 [SDK 참조](../reference/sdk-reference.md) — 1.3.0 버전의 전체 명령어/API 목록
* [동적 컴퓨팅 적응](../processing-architecture/dynamic-compute-adaptation.md) — Chloros가 사용자의 하드웨어에 어떻게 적응하는지
