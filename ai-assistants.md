# AI 어시스턴트와 함께 Chloros 사용하기

이 설명서는 두 가지 대상, 즉 인간과 인간이 점점 더 많이 활용하고 있는 AI 어시스턴트를 위해 작성되었습니다. 각 페이지에는 정확한 값, 기본값 및 복사하여 붙여넣을 수 있는 명령어가 수록되어 있어, 어시스턴트(Claude, ChatGPT, Copilot, 코딩 에이전트 등)가 첫 시도에서 바로 작동하는 Chloros 자동화 코드를 작성할 수 있습니다.

Chloros 버전: **

1.2.0**. CLI/SDK 플랫폼: Windows 10/11 x64 및 Linux (x86_64 / Jetson aarch64).

## 어시스턴트에게 전달할 내용

| 리소스 | URL | 용도 |
| --- | --- | --- |
| **llms.txt** | `https://mapir.gitbook.io/chloros/llms.txt` | 이 설명서의 모든 페이지를 기계가 읽을 수 있는 형식으로 정리한 색인. |
| **CLI 참조** | `https://mapir.gitbook.io/chloros/reference/cli-reference` | `chloros-cli` 명령어의 전체 목록: 모든 명령어, 플래그, 기본값, 종료 코드 및 출력 폴더 규칙. LLM 활용을 위해 작성됨. |
| **SDK 참조** | `https://mapir.gitbook.io/chloros/reference/sdk-reference` | `chloros_sdk`, Python, API에 대한 전체 설명: 클래스, 시그니처, 예외 및 예제. LLM이 활용할 수 있도록 작성됨. |
| **모든 페이지를 원시 마크다운으로** | 페이지 URL 끝에 `.md`를 추가 | 예: `https://mapir.gitbook.io/chloros/reference/sdk-reference.md`는 해당 페이지를 원시 마크다운 형식으로 반환합니다. 컨텍스트 창에 붙여넣거나 에이전트에서 불러오기에 이상적입니다. |

매뉴얼 내 링크: [CLI 참조](reference/cli-reference.md) · [SDK 참조](reference/sdk-reference.md).

{% hint style="info" %}
이 두 참조 페이지는 독립적으로 구성되어 있습니다. 이 중 하나를 읽은 어시스턴트는 올바른 스크립트를 작성하기 위해 매뉴얼의 나머지 부분을 참고할 필요가 없습니다.
{% endhint %}

## 프롬프트 예시

`<placeholders>`를 복사하여 내용을 채운 뒤, 어시스턴트에 붙여넣으세요.

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

### 4. DAQ 광센서 스펙트럼 기록

```

Read https://mapir.gitbook.io/chloros/reference/cli-reference.md (section
"chloros-cli daq" — use only the pool-* commands). Write a script that:
1. connects my DAQ-E sensor with `chloros-cli daq pool-connect --eth-host <daq-e-xxxxxx.local>`,
2. lists the pool with `pool-list` to get the sensor id,
3. records a 10-minute calibrated .daq file named "<field-A>" with `pool-record`,
4. disconnects with `pool-disconnect`.
```

{% hint style="warning" %}
명령줄에서 수행하는 DAQ 스크립팅은 항상 `daq pool-*` 계열(`pool-connect`, `pool-list`, `pool-latest`, `pool-stream`, `pool-record`, `pool-set-cap`, `pool-disconnect`)를 통해 수행됩니다. 어시스턴트가 만들어 낼 수 있는 그 외의 `daq` 하위 명령어들은 출시된 빌드에서는 사용할 수 없으며 오류가 발생하여 종료됩니다.
{% endhint %}

## AI가 작성한 스크립트가 Chloros와 잘 작동하는 이유

이들 각각은 Chloros 1.2.0 버전의 실제 검증된 동작이며, 기계가 작성한 자동화 스크립트에서 흔히 발생하는 고전적인 오류 유형을 제거합니다:

* **별도의 설정 과정이 필요 없습니다.**SDK의 스마트 커넥트 헬퍼(`connect_camera`, `connect_array`, `connect_daq_sensor`)와 처리 진입점 (`ChlorosLocal`, `process_folder`)는**로컬 백엔드를 자동으로 시작**합니다. 생성된 스크립트는 GUI가 열려 있거나 서버가 수동으로 시작되어 있을 필요가 없으며, desktop/CLI 패키지만 설치되어 있으면 됩니다.
* **전체 파이프라인은 하나의 호출로 이루어집니다.** `chloros_sdk.process_folder("path", indices=["NDVI"])`는 가져오기 → 보정 → 반사율 → 지수 내보내기를 처음부터 끝까지 실행합니다. 처리 단계가 적을수록 생성된 스크립트에서 오류가 발생할 가능성이 줄어듭니다.
* **출력이 없는 실행은 자체 진단됩니다.** `process()` 실행 후, 실행 요약이 결과에 첨부되며, 모든 처리 관련 힌트(예: *왜* 실행 결과가 출력되지 않았는지)도 Python `UserWarning` 형태로 재출력되므로, 결과 딕셔너리를 검사하지 않는 스크립트라도 진단 정보를 확인할 수 있습니다.
* **CLI는 명확하게 실패합니다.**출력을 요청했으나 아무것도 기록하지 않은 `chloros-cli process` 실행은 `Processing finished but wrote no image products.`를 출력하고**0이 아닌 종료 코드를 반환**하므로, 쉘 스크립트와 CI는 간단한 종료 코드 확인만으로 이를 감지할 수 있습니다. 성공한 실행은 `Image products written: N`를 반환합니다.

조교가 알아야 할 한 가지 비대칭적인 점은, SDK의 `process()`는 제품이 없는 실행 시 의도적으로 **예외를 발생시키지** 않으며, 대신 요약/힌트를 통해 결과를 보고한다는 것입니다. Python 파이프라인이 빈 실행 시 중단되어야 한다면, 요약 정보를 확인하십시오(레시피 2에서 확인함).

## 주의 사항

* **Chloros+ 로그인이 필요합니다.**CLI 및 SDK는**유료** Chloros+ 티어가 필요하며, 이는 서버 측에서 강제 적용됩니다. 로그인하지 않은 상태에서는 `401 AUTH_REQUIRED` 오류가 발생하고, 무료 티어에서는 `403 PLAN_UPGRADE_REQUIRED` 오류가 발생합니다. 생성된 스크립트를 실행하기 전에 각 머신당 한 번씩 `chloros-cli login`를 실행하십시오. [Chloros+ 로그인](chloros+-login.md)을 참조하십시오.
* **캡처 명령은 실제 하드웨어를 제어합니다.** `lattice` / `daq` / `project` 명령어와 SDK 세션 객체는 물리적 카메라 및 센서에 연결하고, 스트리밍하며, 작동시킵니다. 생성된 스크립트를 처음 실행하기 전에 검토하고, 하드웨어가 작동 중인 상태에서 실행하십시오.
* **출력 결과를 무작위로 확인하십시오.** 결과를 게시하기 전에 제품 폴더와 몇 가지 픽셀 값을 확인하십시오. 특히, 반사율 TIFF 파일은 소스별로 스케일링되므로, 나눗수를 가정하지 말고 `Chloros:PixelScale` XMP 태그 (LATTICE: 32768 = 1.0 반사율; Survey3: 65535)를 참조하십시오. 두 참조 문서 모두 “반사율 픽셀 읽기” 항목에서 이를 설명하고 있습니다.
* **생성된 코드를 방해하는 사소한 함정:**`pool-record`는**백엔드 호스트의** 파일 시스템에 기록합니다(기본값 `~/Documents/DAQ Live View/`); 네트워크 인터페이스가 여러 개인 시스템에서는 자동 탐지 대신 `daq pool-connect --eth-host <ip-or-hostname>`를 선호하십시오; 그리고 백엔드 URL가 나타나는 곳에서는 어디서나 `http://127.0.0.1:5000`(절대 `localhost`는 사용하지 마십시오)를 사용하십시오.
