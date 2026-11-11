# AI 어시스턴트와 함께 Chloros 사용하기

이 매뉴얼은 두 가지 대상, 즉 인간과 인간이 점점 더 많이 활용하고 있는 AI 어시스턴트를 위해 작성되었습니다. 모든 페이지에는 정확한 값, 기본값, 복사하여 붙여넣을 수 있는 명령어가 수록되어 있어, 어시스턴트(Claude, ChatGPT, Copilot, 코딩 에이전트 등)가 첫 시도부터 정상 작동하는 Chloros 자동화 스크립트를 작성할 수 있습니다.

Chloros 버전: **

1.3.0**. CLI/SDK 플랫폼: Windows 10/11 x64 및 Linux (x86\_64 / Jetson aarch64).

## 어시스턴트에게 전달해야 할 내용

| 리소스 | URL | 용도 |
| --- | --- | --- |
| **llms.txt** | `https://mapir.gitbook.io/chloros/llms.txt` | 이 설명서의 모든 페이지를 기계가 읽을 수 있는 형식으로 정리한 색인. |
| **CLI 참조** | `https://mapir.gitbook.io/chloros/reference/cli-reference` | `chloros-cli` 명령어의 전체 목록: 모든 명령어, 플래그, 기본값, 종료 코드 및 출력 폴더 규칙. LLM이 활용할 수 있도록 작성되었습니다. |
| **SDK 참조** | `https://mapir.gitbook.io/chloros/reference/sdk-reference` | `chloros_sdk` Python API의 전체 내용: 클래스, 시그니처, 예외 및 예제. LLM이 활용할 수 있도록 작성됨. |
| **모든 페이지를 원시 마크다운 형식으로** | 페이지 URL에 `.md`를 추가 | 예: `https://mapir.gitbook.io/chloros/reference/sdk-reference.md`는 페이지를 원시 마크다운 형식으로 반환합니다. 컨텍스트 창에 붙여넣거나 에이전트에서 가져오기에 이상적입니다. |

매뉴얼 내 링크: [CLI 참조](reference/cli-reference.md) · [SDK 참조](reference/sdk-reference.md).

{% hint style="info" %}
이 두 참조 페이지는 독립적으로 구성되어 있습니다. 이 중 하나를 읽은 어시스턴트는 올바른 스크립트를 작성하기 위해 매뉴얼의 나머지 부분을 참조할 필요가 없습니다.
{% endhint %}

## 어시스턴트가 Chloros를 직접 실행하도록 하세요: `chloros-mcp`

위에서 설명한 모든 내용은 어시스턴트가 스크립트나 명령어를 *작성*하도록 하는 것입니다. `chloros-mcp`를 사용하면, Claude Desktop, Claude Code 또는 Cursor와 같이 [모델 컨텍스트 프로토콜](https://modelcontextprotocol.io)(MCP)을 지원하는 어시스턴트가 Chloros를 직접 제어합니다. 사용자가 자연어로 요청하면, 어시스턴트가 서버의 도구를 호출합니다:

* **이미지 처리:** 프로젝트 생성, 캡처된 이미지가 담긴 폴더 가져오기, 설정 선택, 처리 수행, 그리고 실행 결과 내보내기.
* **하드웨어 실행:** LATTICE 카메라, 어레이 및 DAQ 광 센서를 연결하고, 미리 보기, 노출 및 색상 설정, 캡처, 기록, 펌웨어 업데이트를 수행합니다.

```text
You:  Process the flight in D:\Flights\2026-10-17 to reflectance and NDVI.

You:  Show me what the array sees, then capture reflectance every 10 seconds
      for 5 minutes.
```

서버의 도구는 [Python SDK](api-python-sdk.md)가 수행하는 기능을 모두 포함합니다. 이 도구는 SDK와 함께 설치되므로 일반적으로 별도로 설치할 필요가 없습니다. Windows에서는 Chloros 설치 프로그램이 `chloros-mcp.exe`를 Python의 `Scripts` 폴더에 배치하며, Linux의 경우 `.deb`가 `chloros-mcp`를 `/usr/bin`에 배치합니다.

**Claude 코드:**

```bash
claude mcp add chloros -- chloros-mcp
```

**Claude Desktop:**이 항목을 `claude_desktop_config.json`에 추가한 후(**설정 &gt; 개발자 &gt; 구성 편집**을 통해 열 수 있음), Claude Desktop을 다시 시작하세요.**Cursor**는 `~/.cursor/mcp.json`에 동일한 항목을 추가합니다.

```json
{
  "mcpServers": {
    "chloros": {"command": "chloros-mcp"}
  }
}
```

클라이언트에서 서버 시작에 실패했다는 메시지가 표시된다면, `chloros-mcp`가 클라이언트가 인식하는 `PATH`에 포함되어 있지 않은 것일 수 있습니다: 대신 전체 경로를 지정하십시오(Windows의 경우 `where chloros-mcp`, Linux의 경우 `which chloros-mcp`).

동일한 컴퓨터에 Chloros가 설치되어 있어야 하며, 해당 컴퓨터에서 앱이나 `chloros-cli login`를 통해 유료 Chloros+ 요금제(Copper 이상)로 로그인되어 있어야 합니다. 서버가 사용자를 대신해 로그인할 수는 없습니다.

{% hint style="info" %}
각 앱, 옵션, 모든 도구에 대한 클라이언트 설정 및 실행 방식: SDK 참조 문서의 [MCP 서버 (AI 어시스턴트)](reference/sdk-reference.md#mcp-server-ai-assistants).
{% endhint %}

## 프롬프트 레시피

`<placeholders>`를 복사하여 내용을 입력한 후, 어시스턴트에 붙여넣으세요.

### 1. 비행 폴더를 NDVI로 처리하기

```

Read https://mapir.gitbook.io/chloros/reference/cli-reference.md.
Then write a script for <Windows PowerShell | bash> that:
1. logs in with `chloros-cli login <email> '<password>'` (only needed once per machine),
2. processes the folder <path/to/flight_001> with reflectance and the NDVI index,
3. prints where each output product landed, using the reference's
   "Where the outputs land" folder rules.
```

### 2. 캡처 디렉터리를 일괄 모니터링

```

Read https://mapir.gitbook.io/chloros/reference/sdk-reference.md (sections
"Quickstart" and "Post-Run Summary & Hints"). Write a Python script that
watches <path/to/captures> for new flight subfolders and runs
chloros_sdk.process_folder() with indices=["NDVI"] on each new one.
After each run, print every hint from result["summary"]["hints"] and treat
a run with zero image products as a failure for that folder.
```

### 3. LATTICE 어레이 연결 및 캡처

```

Read https://mapir.gitbook.io/chloros/reference/sdk-reference.md (section
"connect_array"). Write a Python script that connects my LATTICE cameras
with serials <213800234, 214000533, ...> as one synchronized array, captures
a reflectance image set into <output/> every 10 seconds for one hour, and
disconnects cleanly when done (use the context-manager form).
```

### 4. DAQ 광 센서 스펙트럼 기록

```

Read https://mapir.gitbook.io/chloros/reference/cli-reference.md (section
"chloros-cli daq" — use only the pool-* commands). Write a script that:
1. connects my DAQ-E sensor with `chloros-cli daq pool-connect --eth-host <daq-e-xxxxxx.local>`,
2. lists the pool with `pool-list` to get the sensor id,
3. records a 10-minute calibrated .daq file named "<field-A>" with `pool-record`,
4. disconnects with `pool-disconnect`.
```

{% hint style="warning" %}
명령줄에서 수행하는 DAQ 스크립팅은 항상 `daq pool-*` 계열(`pool-connect`, `pool-list`, `pool-latest`, `pool-stream`, `pool-record`, `pool-set-cap`, `pool-disconnect`)를 통해 수행됩니다. 어시스턴트가 생성할 수 있는 그 외의 `daq` 하위 명령어들은 출시된 빌드에서는 사용할 수 없으며 오류가 발생하여 종료됩니다.
{% endhint %}

## AI가 작성한 스크립트가 Chloros에서 잘 작동하는 이유

이 모든 것은 Chloros 1.3.0에서 실제로 확인된 동작입니다. 이들은 기계가 작성한 자동화 스크립트에서 흔히 발생하는 고전적인 오류 유형을 제거합니다:

* **복잡한 설정 과정이 필요 없습니다.**SDK의 스마트 커넥트 헬퍼(`connect_camera`, `connect_array`, `connect_daq_sensor`)와 처리 진입점 (`ChlorosLocal`, `process_folder`)는**로컬 백엔드를 자동으로 시작**합니다. 생성된 스크립트는 GUI가 열려 있거나 서버를 수동으로 시작할 필요가 없으며, 데스크톱/CLI 패키지만 설치되어 있으면 됩니다.
* **전체 파이프라인은 하나의 호출로 이루어집니다.** `chloros_sdk.process_folder("path", indices=["NDVI"])`는 가져오기 → 보정 → 반사율 → 지수 내보내기를 처음부터 끝까지 실행합니다. 처리 단계가 줄어들어 생성된 스크립트에서 오류가 발생할 가능성이 줄어듭니다.
* **출력이 없는 실행은 자체 진단됩니다.** `process()` 실행 후, 실행 요약이 결과에 첨부되며, 모든 처리 관련 힌트(예: *왜* 출력이 생성되지 않았는지)도 Python `UserWarning`로 재출력되므로, 결과 딕셔너리를 검사하지 않는 스크립트라도 진단 정보를 확인할 수 있습니다.
* **CLI는 명확하게 오류를 표시합니다.**결과물을 요청했으나 아무것도 작성하지 않은 `chloros-cli process` 실행은 `Processing finished but wrote no image products.`를 출력하고**0이 아닌 종료 코드를 반환**하므로, 쉘 스크립트와 CI는 간단한 종료 코드 확인만으로 이를 감지할 수 있습니다. 성공적인 실행은 `Image products written: N`를 보고하며, 마스크가 기록된 경우에는 `(plus M clamp masks)`도 함께 보고합니다. 캡처 명령어도 마찬가지로 기록하지 못한 모든 파일의 이름을 나열하고 `1`로 종료됩니다.
* **상대 경로는 스크립트가 예상하는 대로 해석됩니다.** 로컬 백엔드를 사용할 경우, 상대 출력 경로(CLI에서는 `-o out/`, SDK에서는 `output_dir="out/"`)는 명령어나 스크립트가 실행되는 폴더를 기준으로 해석됩니다.

어시스턴트가 알아야 할 한 가지 차이점: SDK의 `process()`는 결과물이 없는 실행 시 의도적으로 **예외를 발생시키지** 않으며, 대신 요약/힌트를 통해 보고합니다. Python 파이프라인이 빈 실행 시 중단되어야 하는 경우, 요약 정보를 확인하십시오(레시피 2에서 확인합니다).

## 주의 사항

* **Chloros+ 로그인이 필요합니다.**CLI와 SDK를 사용하려면**유료** Chloros+ 등급이 필요하며, 이는 서버 측에서 강제 적용됩니다. 로그인하지 않은 상태에서는 `401 AUTH_REQUIRED` 오류가 발생하고, 무료 등급에서는 `403 PLAN_UPGRADE_REQUIRED` 오류가 발생하여 요청이 실패합니다. 생성된 스크립트를 실행하기 전에 각 머신당 한 번씩 `chloros-cli login`를 실행하십시오. [Chloros+ 로그인](chloros+-login.md)을 참조하십시오.
* **캡처 명령어는 실제 하드웨어를 제어합니다.** `lattice` / `daq` / `project` 명령어, SDK 세션 객체 및 `chloros-mcp`의 도구는 물리적 카메라와 센서에 연결하고, 데이터를 스트리밍하며, 이를 작동시킵니다. 생성된 스크립트를 처음 실행하기 전에 검토하고, 하드웨어가 작동 중인 상태에서 실행하십시오.
* **출력 결과를 무작위로 확인하십시오.** 결과를 게시하기 전에 제품 폴더와 몇 가지 픽셀 값을 확인하십시오. 특히, 반사율 TIFF 파일은 소스별로 스케일링됩니다. `Chloros:PixelScale` XMP 태그를 확인하십시오(LATTICE 파일에는 이 정보가 포함되어 있습니다: 32768 = 1.0 반사율; Survey3 파일은 이 태그가 없으며 65535 = 1.0을 사용합니다)를 참조하여 조정해야 하며, 나눗수를 가정해서는 안 됩니다. 두 참조 페이지 모두 “반사율 픽셀 읽기” 항목에서 이를 설명하고 있습니다.
* **생성된 코드를 방해하는 사소한 함정:**`pool-record`는**백엔드 호스트의** 파일 시스템에 기록합니다(기본값: `~/Documents/DAQ Live View/`); 네트워크 인터페이스가 여러 개인 시스템에서는 자동 탐색 대신 `daq pool-connect --eth-host <ip-or-hostname>`를 사용하는 것이 좋습니다; 백엔드 URL이 나타나는 곳에서는 어디서나 `http://127.0.0.1:5000`를 사용하십시오(절대 `localhost`를 사용하지 마십시오); 프로젝트의 출력 트리를 탐색하는 스크립트는 `clamp_masks/` 폴더를 건너뛰어야 합니다(이 폴더 내의 `*_clampmask.tif` 파일은 이미지가 아닌 마스크입니다 — [출력 이미지 형식](output-image-formats.md#clamp-masks) 참조); 또한 긴 경로 기능이 비활성화된 Windows 환경에서는 프로젝트 폴더 이름을 짧게 유지해야 합니다. 약 130자보다 깊은 프로젝트 폴더는 처리 시작 전에 거부됩니다.
