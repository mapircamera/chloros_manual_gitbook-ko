# Linux 설치

Chloros는 Linux용으로, CLI 및 백엔드 서버를 설치하는 `.deb` 패키지로 배포됩니다. Python SDK는 별도의 pip 패키지이며(버전이 일치하는 wheel 파일 형태로 `.deb` 패키지 내에도 포함되어 있습니다).

패키지 파일 이름에는 버전과 아키텍처가 포함됩니다. x86_64용은 `chloros_1.2.0_amd64.deb`이고, JetPack 6 Jetson 빌드용은 `chloros_1.2.0_arm64_jp6.deb`입니다. 아래 명령어에서 실제로 다운로드한 파일 이름으로 대체하십시오.

***

## Linux amd64 (x86_64)

### 시스템 요구 사항

| 요구 사항 | 최소 | 권장 |
| --- | --- | --- |
| **배포판** | Ubuntu 22.04 LTS 이상 / Debian 12 이상 | Ubuntu 24.04 LTS |
| **프로세서** | x86_64 (Intel/AMD) | Intel Core i7 이상 |
| **메모리 (RAM)** | 8GB | 16GB 이상 |
| **그래픽 카드** | 없음 (CPU 처리) | 4GB 이상의 VRAM을 갖춘 NVIDIA GPU (12GB 이상일 경우 `GPU_PARALLEL`가 활성화되며, 7GB 이상일 경우 단일 이미지 경로에서 Texture Aware 기능이 비활성화됨) |
| **저장 공간** | 2GB 여유 공간 | 10GB 이상의 여유 공간이 있는 SSD |
| **Python** | Python 3.7 이상 (SDK용) | Python 3.10 이상 |

> **Ubuntu 20.04 및 Debian 11은 지원되지 않습니다.** `.deb`의 종속성 목록은
> Chloros 백엔드가 실제로 링크하는 내용에서 파생된 것이며, 여기에는
> `libc6 (>= 2.34)`가 포함됩니다. Focal과 bullseye는 모두 glibc 2.31을 포함하고 있으므로, `apt`는
> 나중에 런타임 시 오류가 발생하는 것을 허용하기보다는 설치를
> 아예 거부합니다.

### 설치

```bash
sudo dpkg -i chloros_1.2.0_amd64.deb
sudo apt-get install -f    # pulls the declared dependencies (libibverbs1, libcap2-bin)
```

{% hint style="info" %}
`dpkg -i`는 종속성을 해결하지 못합니다. 누락된 패키지가 있다고 보고되면, `sudo apt-get install -f`(또는 `sudo apt --fix-broken install`)가 설치를 완료합니다. 이는 오류가 아닌 정상적인 절차입니다.
{% endhint %}

설치를 확인하십시오:

```bash
chloros-cli --version    # prints "Chloros CLI 1.2.0"
```



<!-- SCREENSHOT-NEEDED: Terminal on Ubuntu 22.04 immediately after `sudo dpkg -i chloros_1.2.0_amd64.deb`, showing the full postinst output: the "Chloros installed successfully!" banner, the Usage lines, the "Python SDK:" block naming the bundled wheel path under /usr/lib/chloros/sdk/, any "GPU Acceleration:" detection line, and the closing "Systemd Service (optional): sudo systemctl enable --now chloros-backend.service" hint -->***

## Linux arm64 (NVIDIA Jetson)

### 시스템 요구 사항

| 요구 사항 | 최소 | 권장 |
| --- | --- | --- |
| **플랫폼** | JetPack 6이 설치된 NVIDIA Jetson | Jetson Orin NX 16GB 또는 AGX Orin |
| **JetPack** | JetPack 6.x | 최신 JetPack 6 |
| **메모리 (RAM)** | 8GB (GPU/CPU 공유) | 16GB 이상 공유 (병렬 GPU 워커 실행을 위한 최소 요건은 12GB 이상) |
| **저장 공간** | 2GB 여유 공간 | 10GB 이상의 여유 공간이 있는 NVMe SSD |
| **Python** | Python 3.7 이상 (SDK용) | Python 3.10 이상 |

### 설치

```bash
sudo dpkg -i chloros_1.2.0_arm64_jp6.deb
sudo apt-get install -f
chloros-cli --version
```

amd64 `.deb`와 동일한 레이아웃이며, Jetson Orin / Orin NX / Orin Nano에 최적화된 CUDA 빌드가 포함되어 있습니다. Jetson의 메모리, 열 관리 및 현장 배포 관련 정보는 [NVIDIA Jetson 가이드](nvidia-jetson-guide.md)를 참조하십시오.

***

## Python SDK 설치 (모든 Linux)

SDK는 백엔드용 순수 Python HTTP 클라이언트이므로, 동일한 패키지가 amd64 및 arm64에서 모두 작동합니다. 두 가지 소스:**PyPI에서** — 공개된 안정 버전:

```bash
pip install chloros-sdk
```

**번들된 wheel 파일에서** — 방금 설치한 CLI /backend와 호환이 보장됩니다(사용 중인 `.deb` 버전이 PyPI에 있는 버전보다 최신일 때 이 방법을 사용하세요):

```bash
pip install --user /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

{% hint style="warning" %}
**PEP 668 배포판**(Ubuntu 23.10 이상, Debian 12 이상)은 시스템 전체에 대한 pip 설치를 허용하지 않습니다. `pip install --user …`, 가상 환경 또는 `sudo pip install --break-system-packages …`를 사용하십시오. 패키지 설치 프로그램은 SDK를 시스템Python에 자동으로 설치하지 않으며, 이는 사용자의 선택에 달려 있습니다.
{% endhint %}

선택적 추가 기능:

| 추가 기능 | 명령어 | 추가 내용 |
| --- | --- | --- |
| `progress` | `pip install chloros-sdk[progress]` | 실시간 진행 상황 스트리밍용 `sseclient-py` |
| `camera` | `pip install chloros-sdk[camera]` | BLE(DAQ-M) 전송용 `bleak` |

SDK를 확인하십시오:

```bash
python -c "import chloros_sdk; print(chloros_sdk.__version__)"
```

{% hint style="info" %}
`.deb`는 Chloros CLI 및 백엔드를 설치합니다. Python SDK는 로컬 HTTP API (`http://127.0.0.1:5000`)를 통해 해당 백엔드와 통신하며, 필요할 때 이를 자동으로 시작합니다. `localhost` 대신 항상 리터럴 IPv4 주소를 사용하십시오. `localhost`는 `::1`로 해석될 수 있으며, 요청당 약 2초의 처리 시간이 소요됩니다.
{% endhint %}

***

## 초기 설정

### 1. 로그인

CLI 및 SDK에 액세스하려면 유료 Chloros+ 티어(**Copper** 이상)가 필요하며, 이는 서버 측에서 강제 적용됩니다. 로그아웃된 호출자에게는 `401 AUTH_REQUIRED`가, 무료 티어(Iron) 호출자에게는 `403 PLAN_UPGRADE_REQUIRED`가 할당됩니다.

```bash
chloros-cli login your@email.com 'your-password'
```

인증 정보는 `~/.chloros/user_session.json`에 캐시됩니다.

{% hint style="warning" %}
**설치 또는 업그레이드 후에는 매번 다시 로그인해야 합니다.** 패키지의 `prerm` 스크립트는 해당 컴퓨터의 모든 사용자에 대해 `~/.chloros/user_session.json` 및 캐시된 라이선스를 의도적으로 지우므로, 새로운 빌드는 오래된 캐시를 신뢰하지 않고 항상 라이선스를 재인증합니다.
{% endhint %}

### 2. 라이선스 상태 확인

```bash
chloros-cli status
```

`chloros-cli status`는 모든 티어(무료 버전 포함)에서 작동하므로, 액세스가 가능한지 또는 불가능한지 그 이유를 항상 확인할 수 있습니다.

### 3. 시스템 진단 실행

```bash
chloros-cli selftest
```

7가지 점검 항목이 순서대로 실행되며, 그중 하나라도 실패하면 명령어가 0이 아닌 값으로 종료됩니다:

| # | 점검 항목 | 확인 사항 |
| --- | --- | --- |
| 1 | **버전** | CLI가 자신의 버전(`v1.2.0`)을 보고합니다. |
| 2 | **포트 사용 가능** | 포트 5000이 비어 있거나, *또는* 정상적인 Chloros 백엔드에서 이미 응답한 상태입니다(이 경우 통과로 간주됨). |
| 3 | **백엔드 시작** | 백엔드 바이너리가 실행됩니다. |
| 4 | **API 테스트 (`/api/test`)** | 백엔드가 `status: ok`로 응답합니다. |
| 5 | **시스템 정보** | `/api/system-info`에서 `GPU: <name>, CUDA: <bool>, PyTorch: <version>`를 출력합니다. |
| 6 | **노이즈 제거 모델** | `*.pth.enc` 모델을 찾음 (Linux 기준: `/usr/lib/chloros/models`). |
| 7 | **CUDA + 노이즈 제거기**| Texture Aware 기능을 실제로 사용할 수 있음 — CUDA**및** 모델 파일 최소 1개가 필요합니다. |

실행은 `N/7 checks passed`로 종료되며, 오류가 발생한 항목은 이름으로 나열됩니다.

### 4. 첫 번째 데이터셋 처리하기

```bash
chloros-cli process ~/datasets/flight001
```

***

## 파일 및 디렉터리

### 사용자별

Chloros는 인증 정보와 CLI 구성을 단일 크로스 플랫폼 디렉터리인 **`~/.chloros/`**(Windows에서는 `%USERPROFILE%\.chloros\`)에 저장합니다. 반면, 두 개의 Linux 전용 캐시는 XDG 규약을 따르며, 설정된 경우 `XDG_CONFIG_HOME` / `XDG_CACHE_HOME`를 따릅니다.

| 경로 | 용도 |
| --- | --- |
| `~/.chloros/user_session.json` | `chloros-cli login`에 의해 기록된 로그인 세션 캐시 (패키지 설치/업그레이드 시마다 지워짐) |
| `~/.chloros/working_directory.txt` | 기본 프로젝트 폴더 재정의 (`chloros-cli set-project-folder` / `get-project-folder` / `reset-project-folder`) |
| `~/.chloros/cli_language.json` | CLI 언어 기본 설정 (`chloros-cli language <code>`) |
| `~/.chloros/user.json` | Windows GUI와 공유되는 언어 설정 — 여기서 `language`는 `cli_language.json`보다 우선순위가 높음 |
| `~/.chloros/update_cache.json` | Linux/Jetson 시작 시 업데이트 확인을 위한 1시간 캐시 |
| `~/.chloros/backend.log` | CLI에 의해 백엔드가 실행되었을 때의 백엔드 로그 |
| `~/.chloros/camera_cal/<serial>/<bundle_sha>/` | 일련번호 및 번들 해시를 키로 하는 카메라별 LATTICE 보정 팩 캐시 |
| `~/.chloros/daq_cap_profiles/<u\|m\|e>/<cap_id>.json` | DAQ 캡 보정 프로필에 대한 선택적 사용자 재정의 |
| `~/.config/chloros/system_config.json` | 동적 컴퓨팅 적응(Dynamic Compute Adaptation)에서 가져온 캐시된 하드웨어 프로필 — 이를 삭제하면 새로운 하드웨어 감지를 강제합니다 |
| `~/.cache/chloros/logs/backend_<YYYYMMDD_HHMMSS>.log` | 백엔드 서버 로그, 실행 횟수당 하나의 파일 |
| `~/Chloros Projects/` | 재정의가 설정되지 않았을 때의 기본 프로젝트 폴더 |

### 시스템 전체

| 경로 | 용도 |
| --- | --- |
| `/usr/bin/chloros-cli` | 래퍼 스크립트 — 번들된 네이티브 라이브러리에 대해 `LD_LIBRARY_PATH`를 설정한 후, 실제 바이너리를 실행합니다 |
| `/usr/bin/chloros-backend` | 래퍼 스크립트 — 위와 동일하며, 추가로 `CHLOROS_PRODUCTION=1`를 설정하여 백엔드의 인증 게이트가 자동으로 비활성화되지 않도록 함 |
| `/usr/lib/chloros/chloros-cli`, `/usr/lib/chloros/chloros-backend` | 컴파일된 바이너리 |
| `/usr/lib/chloros/arena_runtime/` | LATTICE 카메라에 필요한 Arena SDK 런타임 |
| `/usr/lib/chloros/models/*.pth.enc` | Texture Aware 디베이어에서 사용하는 암호화된 노이즈 제거 모델 |
| `/usr/lib/chloros/sdk/chloros_sdk-*.whl` | Python 이 빌드와 정확히 일치하는 SDK 휠 |
| `/usr/lib/chloros/exiftool` | 번들된 exiftool (시스템에 exiftool이 존재하지 않을 경우에만 `/usr/local/bin/exiftool`로 심볼릭 링크됨) |
| `/etc/chloros/update.conf` | `chloros-cli update`가 읽는 업데이트 채널 구성 |
| `/etc/sysctl.d/60-chloros-ptp.conf` | 백엔드가 루트 권한 없이 PTP 포트를 바인딩할 수 있도록 `net.ipv4.ip_unprivileged_port_start = 319`를 설정합니다 |
| `/etc/ld.so.conf.d/Arena_SDK.conf` | 동적 로더가 `/usr/lib/chloros/arena_runtime`를 가리키도록 설정 |
| `/lib/udev/rules.d/70-chloros-daq.rules` | 로그인한 사용자에게 DAQ-U USB 직렬 브리지(CP2102N, `10c4:ea60`)에 대한 액세스 권한 부여 |
| `/lib/systemd/system/chloros-backend.service` | 상시 실행 백엔드 서비스 선택 활성화 (설치됨, **비활성화됨**) |
| `/usr/share/applications/chloros-cli.desktop` | 터미널을 여는 &quot;Chloros CLI&quot; 애플리케이션 메뉴 항목 |

## 백엔드 실행 파일 위치

CLI와 SDK는 백엔드를 자동으로 감지합니다:

| 구성 요소 | 경로 |
| --- | --- |
| CLI | `/usr/bin/chloros-cli` |
| 백엔드 | `/usr/lib/chloros/chloros-backend` |

CLI 플래그인 `--backend-exe` 또는 SDK 생성자 매개변수인 `backend_exe`를 사용하여 백엔드 경로를 재정의하고, `--port` (기본값 `5000`)로 재정의할 수 있습니다.

{% hint style="info" %}
`CHLOROS_BACKEND_URL`는 원격 백엔드에서 **`lattice`**,**`project`**,**`daq pool-*`** 명령어 계열을 원격 백엔드로 가리킵니다. 핵심 명령어(`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`)는 이를 의도적으로 무시하고 항상 `http://127.0.0.1:<port>`를 대상으로 합니다.
{% endhint %}

***

## Linux에서 LATTICE 카메라 및 DAQ 광 센서

live-hardware 명령어 계열은 모두 Linux(amd64 및 Jetson)에서 작동합니다:

* **`chloros-cli lattice`** — LATTICE 카메라 및 동기화된 어레이를 탐색, 연결, 구성하고 데이터를 캡처합니다. `.deb`는 이에 필요한 Arena SDK 런타임을 번들로 제공하며, 이를 동적 로더에 등록합니다.
* **`chloros-cli daq pool-*`** — 백엔드 풀을 통해 DAQ-U/M/E 광 센서에 연결하고, 보정된 스펙트럼을 스트리밍하며, `.daq` 파일을 기록합니다. 컴파일된 CLI에는 `pool-*` 계열만 포함되어 있습니다: `pool-connect`, `pool-disconnect`, `pool-list`, `pool-latest`, `pool-stream`, `pool-record`, `pool-set-cap`.
* **`chloros-cli project`** — 저장된 프로젝트(해당 카메라, 센서 및 처리 설정)를 헤드리스 모드로 실행합니다.
* **`chloros-cli time-sync`** — Chloros 백엔드가 LATTICE 카메라 및 DAQ-E 센서를 위해 실행하는 PTP 그랜드마스터를 확인합니다.

```bash
# DAQ-E at a known address — the reliable path on multi-homed hosts
chloros-cli daq pool-connect --eth-host 192.168.2.50

# DAQ-U over USB serial
chloros-cli daq pool-connect --port /dev/ttyUSB0

# What is connected, then the latest calibrated spectrum as JSON
chloros-cli daq pool-list
chloros-cli daq pool-latest --sensor-id daq-e-a1b2c3 --json
```

`--sensor-id`는 `pool-latest`, `pool-stream`, `pool-record` 및 `pool-set-cap`에 의해 필수로 지정됩니다. `pool-list`는 현재 풀에 있는 ID를 표시합니다.

{% hint style="info" %}
**다중 호스팅 시스템에서 첫 번째 DAQ-E 연결 시에는 `--eth-host`를 우선적으로 사용하십시오.** 자동 검색은 mDNS를 탐색하므로, 비활성 ARP 캐시로 인해 센서의 인터페이스를 놓칠 수 있습니다. 따라서 부팅 후 첫 번째 `pool-connect --eth` 연결은 센서가 완벽하게 정상 상태라 하더라도 실패할 수 있습니다. 센서의 IP 또는 호스트명을 직접 지정하면 검색 과정을 완전히 건너뛸 수 있습니다.
{% endhint %}

**DAQ-U 시리얼 권한**은 설치된 udev 규칙(`uaccess` + 그룹 `dialout`)에 의해 처리됩니다. 이미 연결되어 있던 센서에 계속 접근할 수 없는 경우, 규칙을 다시 로드하거나 센서를 다시 연결하십시오:

```bash
sudo udevadm control --reload-rules
sudo udevadm trigger --subsystem-match=tty
```

전체 명령어 세트는 [CLI 참조 문서](../CLI.md)를 참조하십시오.

### 헤드리스 호스트용 상시 PTP

최초 설치 시 systemd 유닛 `chloros-backend.service`가 생성되지만 **활성화되지는 않습니다**. DAQ-E 센서 및 LATTICE 카메라를 위해 PTP 시간 동기화를 지속적으로 실행해야 하는 헤드리스 Jetson 또는 서버에서는 다음을 활성화하십시오:

```bash
sudo systemctl enable --now chloros-backend.service
sudo systemctl status chloros-backend.service
```

이 설정이 없으면 PTP는 Chloros 백엔드가 실행되는 동안, 즉 CLI / SDK 세션이 활성화된 동안에만 실행됩니다.

이 유닛은 백엔드를 `127.0.0.1:5000`에 바인딩하며(유닛 내부의 `CHLOROS_HOST` / `CHLOROS_PORT` 환경 설정; `sudo systemctl edit chloros-backend.service`로 재정의)에 바인딩하고, 오류 발생 시 5초 후에 다시 시작합니다.

**PTP가 포트를 할당받는 방식.** PTP는 UDP 319/320을 사용하며, 두 포트 모두 일반적인 1024 특권 포트 하한선보다 낮습니다. 이 패키지의 `postinst`는 `/etc/sysctl.d/60-chloros-ptp.conf`에 `net.ipv4.ip_unprivileged_port_start = 319`를 기록하여, 이를 통해 백엔드는 사용자의 권한으로 실행되는 동안 해당 포트에 바인딩할 수 있습니다. 또한 만약을 대비하여 백엔드 바이너리에 `setcap cap_net_bind_service,cap_net_raw=+ep`를 적용하는데, 이것이 바로 `libcap2-bin`가 이 패키지의 선언된 종속성으로 지정된 이유입니다.***

## Bash 스크립트 예제

{% hint style="info" %}
**스크립트 작성에 적합한 종료 코드.**`chloros-cli process`는 성공 시 `0`를 반환하고,**실패 시 0이 아닌 값을 반환합니다 — 이미지 제품을 요청했으나 아무것도 생성하지 못한 실행도 포함됩니다** (이 경우 `Processing finished but wrote no image products.`를 출력하고 프로젝트 폴더 이름 및 일반적인 원인을 표시합니다). 실행에 성공하면 기록된 이미지 제품의 수(`Image products written: N`)를 보고합니다. 종료 코드: `0`(성공), `1`(실패), `2`(인수 오류), `130`(중단).
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
    --indices NDVI NDRE GNDVI \
    --debayer texture-aware \
    --no-vignette
```

유효한 `--format` 값은 정확히 네 개이며, 공백이 포함되어 있으므로 항상 따옴표로 묶어야 합니다:

| `--format` 값 | 출력 폴더 |
| --- | --- |
| `TIFF (16-bit)` *(기본값)* | `tiff16` |
| `TIFF (32-bit, Percent)` | `tiff32` |
| `PNG (8-bit)` | `png8` |
| `JPG (8-bit)` | `jpg8` |

`--debayer`는 `standard`(기본값) 또는 `texture-aware`(Chloros+)를 허용합니다.

### Cron을 이용한 자동 처리

```cron
# Process any new datasets at 2 AM daily
0 2 * ** /usr/bin/chloros-cli process /data/incoming --output /data/processed >> /var/log/chloros.log 2>&1
```

### Python SDK 예시

```python
from chloros_sdk import process_folder

# One-line processing
result = process_folder(
    "/home/user/datasets/flight001",
    indices=["NDVI", "NDRE"],
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
sudo chmod +x /usr/lib/chloros/chloros-backend
```

### 설치 중 &quot;setcap failed&quot; 오류

`.deb`는 `cap_net_bind_service`를 `/usr/lib/chloros/chloros-backend`에 적용하여 루트 권한 없이 PTP 포트 319/320에 바인딩할 수 있도록 합니다. 설치 시점에 `libcap2-bin`가 설치 시점에 누락된 경우 해당 호출은 건너뜁니다. 이를 설치한 후 패키지를 다시 설치하십시오:

```bash
sudo apt install libcap2-bin
sudo apt reinstall chloros
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

### &quot;LATTICE 카메라 드라이버를 찾을 수 없음&quot;

Arena SDK 런타임이 해결되지 않고 있습니다. 패키지가 작성하는 로더 구성이 존재하고 최신 상태인지 확인하십시오:

```bash
cat /etc/ld.so.conf.d/Arena_SDK.conf     # expect /usr/lib/chloros/arena_runtime
sudo ldconfig
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

시작에 실패한 백엔드의 로그 파일은 `~/.cache/chloros/logs/`에 있습니다.

### CUDA 감지되지 않음

```bash
# Check NVIDIA driver installation
nvidia-smi

# Check CUDA availability
nvcc --version

# On Jetson, check JetPack version
cat /etc/nv_tegra_release
```

`chloros-cli selftest` 파일은 `GPU: <name>, CUDA: <bool>, PyTorch: <version>`라는 한 줄로 동일한 내용을 보고합니다.

### 공유 라이브러리 누락

```bash
sudo apt-get update
sudo apt-get install -f

# Check for missing libraries
ldd /usr/lib/chloros/chloros-backend | grep "not found"
```

### SD 카드 시스템에서의 느린 시작

컴파일된 바이너리는 실행될 때마다 임시 디렉터리로 자동으로 압축을 풉니다. `/mnt/ssd/tmp` 파일이 존재하면 Chloros가 이를 자동으로 사용하며, 그렇지 않은 경우 `TMPDIR`를 빠른 파일 시스템으로 설정하십시오:

```bash
export TMPDIR=/mnt/nvme/tmp
```

***

## Linux에서 Chloros 업데이트하기

`update` 명령어는 Linux /Jetson에서만 사용할 수 있습니다. 이 명령어는 `/etc/chloros/update.conf`에서 구성된 업데이트 채널에 게시된 버전을 확인하고, 해당 버전에 맞는 `.deb`를 다운로드하여 설치할 수 있도록 제안합니다:

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

Linux/Jetson에서는 CLI도 매 시작 시 비차단 업데이트 확인을 수행하며(결과는 `~/.chloros/update_cache.json`에 1시간 동안 캐시됨), 더 새로운 버전이 존재할 경우 `Update available: vX.Y.Z`를 출력합니다. 설정과 프로젝트는 업데이트 후에도 유지되며, 업데이트 후에는 다시 로그인해야 합니다.

## 제거

```bash
sudo apt remove chloros
```

제거하면 `chloros-backend.service`가 중지되고, 기본 비특권 포트 하한값(1024)으로 복원되며, 번들된 exiftool 심볼릭 링크와 Arena 로더 구성이 제거되고, 캐시된 인증 정보가 지워집니다. 프로젝트와 `~/.chloros/` 데이터 파일은 그대로 유지됩니다.

***

## 다음 단계

* [NVIDIA Jetson 가이드](nvidia-jetson-guide.md) — Jetson 전용 최적화 및 배포
* [CLI : 명령줄](../CLI.md) — CLI 가이드
* [API : Python SDK](../api-python-sdk.md) — SDK 가이드
* [CLI 참조](../reference/cli-reference.md) 및 [SDK 참조](../reference/sdk-reference.md) — 1.2.0 버전에 대한 상세한 명령어/API 목록
* [동적 컴퓨팅 적응(Dynamic Compute Adaptation)](../processing-architecture/dynamic-compute-adaptation.md) — Chloros가 사용자의 하드웨어에 어떻게 적응하는지
