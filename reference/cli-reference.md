# Chloros CLI 참조

**버전:**

1.3.0**생성일:**2026-07-29 19:19 ·**수정일:** 2026-11-06**대상:** LLM 활용에 최적화되었으며, 사람이 읽기 쉬운 형식으로 작성되었습니다.**범위:** `chloros-cli`의 모든 사용자용 하위 명령어와 해당 옵션, 복사하여 붙여넣을 수 있는 예제.

이 문서는 MAPIR Chloros에 포함된 `chloros-cli` 명령줄 도구에 대한 완전한 참조 자료입니다. LLM(또는 사람)이 소스 코드를 확인하지 않고도 아래 목록을 바탕으로 지원되는 모든 워크플로를 구성할 수 있도록 의도적으로 상세하게 작성되었습니다.

핵심 내용만 확인하시려면 다음으로 이동하세요:
- [5분 퀵스타트](#five-minute-quickstart)
- [LATTICE 카메라 첫 연결 워크플로우](#lattice-camera-first-connect-workflow)
- [DAQ 센서 첫 연결 워크플로우](#daq-sensor-first-connect-workflow)
- [Smart-AE / Smart-Capture](#smart-ae--smart-capture)
- [캡처 모드, 레코더 및 오프라인 재처리](#capture-modes-recorders--offline-reprocess)

> **AI 어시스턴트를 통해 Chloros를 구동하시나요?** Claude Desktop, Claude Code, Cursor 및 기타 MCP 클라이언트는 Python SDK와 함께 설치되는(그리고 Linux .deb 패키지, `chloros-mcp`에 포함된) MCP 서버인 `claude mcp add chloros -- chloros-mcp`를 통해 Processing, LATTICE 카메라 및 어레이, DAQ 광 센서를 실행할 수 있습니다. Python SDK와 함께 설치되는 MCP 서버(Linux .deb 패키지의 경우 `/usr/bin` 포함)인 `chloros-mcp`를 통해 클로로스를 제어할 수 있습니다: `claude mcp add chloros -- chloros-mcp`. 설정, 옵션 및 도구에 대한 내용은 SDK 레퍼런스의 [MCP 서버 (AI 어시스턴트)](sdk-reference.md#mcp-server-ai-assistants)에 나와 있습니다. 셸이 포함된 어시스턴트를 사용하면 이 페이지의 모든 명령을 직접 실행할 수도 있습니다.

> **

1.3.0에서 변경된 사항** (기존 1.2.0 스크립트나 워크플로우에서 감지되는 내용).
>
> **

1.2.0 스크립트에서 처리해야 할 수 있는 변경 사항:**
>
> - **`--preset`가 적용됩니다.** `lattice capture`, `continuous`, `calibrate`, `multi-capture`, `multi-stream`, `align-calibrate`, `align-apply` 및 `align-stream`는 1.2 버전에서 제외되었습니다.0으로 설정되어 카메라의 기본 설정으로 작동했습니다. 이제 프리셋의 포맷, 노출, 게인 및 프레임 속도를 설정합니다: `--preset high_speed`는 5 ms, 6 dB, 60 fps로 8비트 영상을 캡처합니다. `lattice cam-connect --preset` 역시 프리셋의 포맷을 유지합니다(1.2.0 버전에서는 프리셋 적용 후 카메라가 12비트로 설정되었습니다). [공통 옵션](#common-options-shared-by-most-subcommands)을 참조하십시오.
> - **M3M(모노) 카메라에서 베이어 픽셀 형식은 해당 카메라의 비트 심도를 의미합니다.** 프리셋 이름에는 모노 센서가 촬영할 수 없는 베이어 형식이 명시되어 있습니다: 1.2.0 버전에서는 카메라가 해당 형식을 거부하고 기존 형식을 유지했기 때문에, 따라서 `--preset high_speed`는 해당 환경에서 8비트가 아니었습니다. 현재는 동일한 깊이의 모노 형식으로 변경되었습니다: `high_speed`와 `--pixel-format BayerRG8`는 `Mono8`를 생성하고, `high_quality`, `triggered`와 `BayerRG12`는 `Mono12`를 생성합니다. M3C와 M3M이 혼합된 배열에서는 각 카메라가 자체 계열 내에서 공유되는 형식을 사용하며, `network-analysis` / `analyze-array --pixel-format`는 각 카메라를 해당 심도로 매핑합니다. 반대의 경우도 마찬가지입니다. 모노 형식을 요청받은 M3C는 해당 심도의 베이어 형식을 채택하므로, 마스터가 M3M인 어레이는 더 이상 M3C 카메라를 모노 모드로 재개방하지 않습니다. [모노(M3M) 단일 대역 카메라](#mono-m3m-single-band-cameras)를 참조하십시오.
> - **`lattice array-capture` 내보내기 파일은 앱과 마찬가지로 기본적으로 정렬되어 있습니다.** 모든 멤버의 `debayered`, `display`, `radiance`, `reflectance` 및 인덱스 파일은 마스터 카메라의 프레임에 맞춰 변형되고, 멤버들의 중첩 영역 (카메라 오프셋만큼 센서보다 작은 영역, `XMP-mapir:AlignmentJSON` `"applied": true`)로 잘려 나갑니다. 따라서 CLI로 캡처한 파일은 동일한 장면을 Chloros 앱으로 캡처한 파일과 동일하며, `raw`는 원본 크기를 유지합니다. 이전에는 `--aligned`가 없는 캡처의 경우 모든 파일이 왜곡되지 않은 원본 크기로 저장되었습니다. `--no-aligned`는 이 방식을 유지합니다. 동일한 기본값이 `--smart`, `--continuous` 및 `--interval`, 그리고 SDK와 `chloros-mcp`에도 동일하게 적용됩니다. [`array-capture` 모드](#array-capture-modes)를 참조하십시오.
> - **`lattice array-capture --compression`는 기본적으로 `none`로 설정됩니다.** 파일은 가장 빠른 지속 전송 속도를 위해 압축되지 않은 상태로 기록됩니다(전체 해상도 프레임당 약 6.3 MB); 1.2.0 버전은 `deflate`(약 4.1 MB) 형식으로 기록했습니다. 파일 크기를 줄이려면 `--compression deflate`를 전달하십시오. 두 방식 모두 무손실입니다.
> - **PNG 및 JPEG는 `display` / `preview` 및 인덱스 제품만 지원합니다.** 8비트 파일이 수용할 수 없는 레벨을 가진 `--format png` 또는 `jpg` 캡처는 캡처가 시작되기 전에 거부됩니다(HTTP 400, `1` 종료), 또한 `--format png`/`jpg`와 함께 사용되는 기본값인 `--processing all`도 마찬가지입니다. [캡처 내보내기 레벨](#capture-export-levels-the-all-default)을 참조하십시오를 참조하십시오.
> - **`lattice dls`에는 `--mac`가 필요합니다.**더 이상 기본 MAC 주소가 없습니다(이전에는 하나의 장치 주소였습니다). `daq pool-discover --only ble`는 센서의 MAC 주소를 찾습니다.**`lattice dls --camera SERIAL`**는 새로 추가된 항목입니다.
> - **추가 오류 발생 시 `1`가 종료됩니다.** 실행 시 내보낸 데이터를 확인할 수 없거나 이미지를 잃어버린 경우(다른 제품은 기록되었더라도) `process`가 발생합니다; 기록하지 못한 파일이 포함된 캡처 명령어 (다른 파일은 여전히 저장됨); 드라이브 용량이 가득 찼을 때 발생하는 `lattice continuous` 및 `multi-stream`; 레코더가 오류를 보고할 때 발생하는 `array-burst` / `array-record`; 요청된 제품이 건너뛰어지거나 요청대로 생성되지 않았을 때: `array-build-video`; 캡처가 도중에 실패했을 때: `project burst` / `stream`; `project sensor log`: 녹화물에 판독 가능한 데이터가 없는 경우; `lattice exposure --set`, `gain --set` 및 `configure --set`: 카메라가 거부하는 값이 있는 경우. `lattice network-analysis --json`는 요약과 마찬가지로 종료됩니다 (`1`는 `needs_force_slip`에 해당). 반대의 경우: 더 이상 풀에 속하지 않는 센서의 `daq pool-disconnect`는 `0`로 종료됩니다X로 종료됩니다. [종료 코드](#exit-codes)를 참조하십시오.
> - **속도가 느린 `lattice array-connect`는 어레이가 연결된 상태에서는 절대 실패하지 않습니다.** 최대 600초(1.2.0 버전에서는 180초; `project connect` 90초)까지 대기하며, 이 과정에서 백엔드의 단계를 출력하고, 그때까지 완료되지 않은 연결은 백엔드에 의해 — 모든 카메라를 발견된 상태 그대로 복원합니다 — 따라서 `Error:`(종료 `1`)는 어레이가 연결되지 않았음을 의미합니다. 1.2.0 버전에서는 카메라 전원이 켜진 직후에 시작된 연결이 대기 시간을 초과하여, 명령이 실패한 후에도 어레이가 연결될 수 있었습니다. 이제 Ctrl+C를 누르면 연결이 취소됩니다. [지속적 연결](#persistent-connections-smart-prep-gui-equivalent-flow)을 참조하십시오.
> - **`lattice array-connect`는 전체 센서를 유지합니다.** `--width` / `--height` / `--binning`가 설정되지 않은 경우, 모든 카메라가 전체 센서 모드로 설정되며 프레임 속도는 해당 컴퓨터와 네트워크가 제공하는 수준으로 떨어집니다. 연결 출력에는 예상 프레임 속도와 함께 더 빠른 옵션 (비닝 또는 더 작은 ROI)와 예상 프레임 속도를 나열합니다. 1.2.0 버전은 프레임 속도를 유지하기 위해 프레임을 축소하거나 비닝 처리했습니다. [`array-connect` 옵션](#array-connect-options)을 참조하십시오.
> - **느린 네트워크로 인해 더 이상 어레이가 거부되지 않습니다.** 각 카메라는 네트워크 대역폭 할당량에 맞춰 속도가 조절되며, 이에 따라 프레임 속도도 낮아집니다. 카메라의 속도를 그만큼 낮출 수 없는 경우, 어레이는 순차적 출력 (`slip-emit-and-capture`) 방식으로 작동합니다: 모든 프레임이 도착하지만, 카메라들은 동시에 노출되지 않습니다. `--force-tier sim-capture-sim-emit`는 동시 노출을 고수합니다. [어레이 fps 및 버스트 모델](#array-fps--burst-model)을 참조하십시오.
> - **`lattice network-analysis` / `analyze-array`는 connect가 적용되는 방식을 반영합니다**: `--pixel-format`가 없으면, connect가 설정하는 포맷(Mono12 / BayerRG12, 이전에는 BayerRG8)과 [압축된 12비트](#packed-12-bit-transport---packed) 결정이 적용됩니다. 따라서 동일한 명령어도 1.2.0 버전과는 다른 프레임 속도를 표시할 수 있습니다.
> - **상대 경로는 명령어를 실행하는 위치 기준으로 적용됩니다.** `-o` 및 백엔드가 여는 다른 경로들 (`--input`, `--burst-dir`, `--profile`, `--save`, `--mask`), 레시피 단계의 `output`, 그리고 `lattice index --live`의 기본값인 `output/`는 `chloros-cli`를 실행하는 폴더를 기준으로 해결됩니다. 1.2.0 버전에서는 백엔드가 자체 설치 폴더를 기준으로 이를 해결했으나, 설치된 빌드에서는 "액세스가 거부되었습니다"라는 오류가 발생했습니다. [약관](#conventions)을 참조하십시오.
> - **`lattice index`는 다른 `lattice` 명령과 마찬가지로 모든 형태에서 백엔드에서 실행되므로**, 먼저 백엔드를 시작하십시오. 오프라인 인덱스 (`--input`)는 백엔드에서 계산되며, 백엔드는 TIFF 파일을 읽어 그 옆이나 `-o` 위치에 결과물을 기록하고, `-o` 및 `--list-presets` / `--list-gradients`는 백엔드의 테이블을 출력합니다. Windows에서는 이제 다음 기능이 작동합니다: 이전에는 Windows CLI가 `lattice index`(`--live` 포함)의 모든 형태에 대해 `Error: lattice_sdk not available`로 응답했습니다. Linux에서는 오프라인 인덱스는 이전에는 백엔드나 로그인 없이 CLI 내에서 실행되었으나, 이제는 다른 모든 CLI 명령어와 마찬가지로 둘 다 필요합니다. [인덱스 / 식생 수학](#index--vegetation-maths)을 참조하십시오.
> - **결과물 옆에 생성되는 새 파일들.** 밴드 언믹스 과정에서 픽셀이 클램핑된 모든 방사도, 반사도 및 인덱스 산출물에는 해당 산출물 폴더 옆에 있는 `clamp_masks/` 폴더에 마스크가 생성되며, 출력 결과에는 `clamp mask:` 행이 기록됩니다. [밴드 분리 클램프 마스크](#unmix-clamp-mask-which-pixels-a-band-was-clamped-on)를 참조하십시오.
> - **카메라 방향.** 단일 카메라로 캡처할 경우, 센서가 읽은 그대로 `raw` 및 `debayered`를 저장하고 카메라의 뒤집기/회전 정보를 XMP에 기록하며, `process`는 이를 생성된 결과물에 적용합니다. 새로운 `--orient` (`array-capture`, `array-burst`, `project capture` / `burst` / `stream`)를 전달하지 않는 한, 어레이 구성원 파일은 네이티브 형식으로 유지됩니다. [카메라 방향](#what-a-captures-folder-looks-like)을 참조하십시오.
> - **1이 아닌 내보내기 레벨 이름은 허용되지 않습니다.** `lattice capture --processing` 및 모든 `--levels` 목록은 `raw`, `debayered`, `radiance`, `reflectance` 또는 `display`(혹은 `preview`와 같은 별칭; `--processing` 및 `all`). 카메라를 터치하기 전에는 다른 이름은 허용되지 않습니다(HTTP 400 `UNKNOWN_LEVEL`, 종료 `1`); 1.2.0은 이를 위해 `raw`를 저장하고 종료되었습니다(`0`).
> - **`daq pool-set-cap`에는 `--cap-id`가 필요합니다.** 이것이 없으면 명령이 종료됩니다(`2`); 1.2.0에서는 캡 보정을 제거하여 베어 센서를 남겼습니다. `--cap-id none`는 기본 센서를 요청합니다.
> - **`lattice resolution --binning-mode`는 제거되었습니다.** 비닝은 항상 '평균(Average)'으로 설정되며, 해당 플래그는 여전히 받아들여지지만 경고 메시지를 출력하고 아무런 동작도 수행하지 않습니다. [센서 튜닝](#sensor-tuning)을 참조하십시오.
> - **LATTICE 캡처의 경우 `process --debayer texture-aware`가 거부됩니다.** LATTICE 캡처 파일이 포함된 폴더의 경우, `process`는 가져오기 후 `1`를 종료하며, 처리 작업을 시작하기 전에 파일 수를 표시합니다. LATTICE 캡처 파일은 항상 표준 디모자이크 처리를 거치므로, 1.2.0 버전에서는 해당 옵션을 받아들이되 무시했습니다. Chloros 앱에서도 LATTICE 캡처 파일에 대해서는 이 옵션을 제공하지 않습니다. [`chloros-cli process`](#chloros-cli-process)를 참조하십시오.
> - **카메라는 발견된 상태로 되돌아가며 — `--keep-settings`가 아닌 경우.** `lattice array-disconnect`, 모든 일시적 명령(배열이 없는 `multi-capture`, `multi-stream`, `multi-test`, `align-calibrate`, `align-apply`, `align-stream`, `index --live`) 및 `lattice continuous`는 각 카메라를 원래 상태로 복원합니다: 트리거 및 싱크 라인 배선, PTP, 그리고 세션 중에 변경된 모든 카메라 설정(노출 및 게인, AE 목표값, 한계값 및 댐핑, 화이트 밸런스, 블랙 레벨, 픽셀 포맷, 비닝 및 ROI, 플립, 프레임 레이트)을 원상태로 복원합니다. 반면 `--keep-settings`는 각 카메라의설정을 카메라 자체 메모리에 저장하므로, 전원을 켤 때 해당 설정으로 시작됩니다; `--clear-settings`는 해당 메모리를 공장 출하 시 설정으로 초기화합니다. 어느 경우든 카메라는 자유 실행 상태로 남겨집니다. 프로젝트는 카메라를 연결할 때 자체 저장된 설정으로 카메라를 설정합니다. [카메라 메모리](#camera-memory----keep-settings----clear-settings)를 참조하십시오.
> - **모든 모드는 동일한 방식으로 프로젝트를 저장합니다.** 앱, CLI, SDK 또는 MCP 서버를 통해 프로젝트에 연결된 카메라, 어레이 또는 광 센서에 대한 모든 변경 사항은 해당 프로젝트의 `cameras.json` / `sensors.json`에 저장되며, 어떤 모드에서든 해당 프로젝트를 열면 그 설정에 따라 장치가 구성됩니다. 이전에는 앱에서만 프로젝트를 저장했으며, 다음 저장 시 다른 모든 내용을 덮어쓰곤 했습니다. [단일 프로젝트 기록, 모든 모드](#one-project-record-every-mode)를 참조하십시오.
> - **여러 대의 카메라로 생성된 파일에는 각 카메라의 이름이 모두 명시됩니다.** 멀티밴드 스택(`align-apply`, `align-stream`, `index --live --save-multiband`), 여러 대의 카메라 밴드로 생성된 라이브 인덱스, 그리고 배열 캡처의 `composite/` 이미지는 각 소스 카메라의 일련번호, 모델 및 보정 팩을 목록(`XMP-mapir:SourceCameraSerials`, `SourceCameraModels`, `SourceCalBundleSha256s`, …)로 저장하며, EXIF `SerialNumber`에는 일련번호가 쉼표로 구분되어 나열됩니다. 이전에는 카메라 정보를 전혀 포함하지 않거나, 첫 번째 카메라의 일련번호만 포함하곤 했습니다. [여러 대의 카메라로 생성된 파일](#files-made-from-several-cameras)을 참조하십시오.
> - **`raw` 파일만이 카메라의 보정 태그를 포함합니다.** MicaSense 호환 세트(`BlackLevel`, `RadiometricCalibration`, 채널별 `Band{R,G,B}RadiometricCalibration`, `MicaSenseVignetting*` 세트) 및 `CalibrationJSON` 페이로드는 센서의 원시 값을 기술하므로, `raw` 캡처 파일에만 포함됩니다. 그 외의 모든 파일 — 캡처의 `debayered`, `display`, `radiance`, `reflectance` 및 인덱스 파일, `lattice continuous` 및 `multi-stream` 프레임, 그리고 모든 `process` 산출물 — 에는 카메라 식별 정보(`XMP-mapir:CameraSerial`, `CameraModel`, `FilterType`, `LensModel`, `CalBundleSha256`, ...)를 포함하며, 촬영 시에는 프레임 고유의 기록(`XMP-mapir:FrameJSON`: 노출, 게인, 온도, 픽셀 형식, 촬영 시간)을 담고 있습니다. 1.2.0 버전에서는 원시 데이터 세트도 포함되었는데, 이를 적용하는 도구 (Pix4D, Metashape)는 원시 값이 아닌 픽셀에 이를 적용했습니다. `process`는 일련번호와 `CalBundleSha256`를 통해 해당 파일의 보정 정보를 찾아냅니다.
>
> **기타 변경 사항:**
>
> - **어레이의 자동 프레임 속도는 어레이가 제공하는 속도에 맞춰집니다.** 트리거 수의 1% 이상을 잃는 속도는 하향 조정되며, 이는 한 번이 아닌 여러 번의 장시간 관찰을 통해 판단됩니다. 또한 하향 조정된 속도는 수시로 재시도됩니다. 사용자가 직접 설정한 속도는 변경되지 않습니다.
> - **자동 노출이 더 안정적으로 작동합니다.** 일정한 조도에서는 노출을 유지하며, 조도 변화 후 몇 초 이내에 안정화되고, 미세한 하이라이트는 무시하며, 강한 햇빛 아래에서 응답을 멈춘 카메라를 다시 정상 작동 상태로 되돌립니다. [자동 노출 작동 방식](#how-auto-exposure-behaves)을 참조하십시오.
> - **`process`는 사용자의 이미지와 녹화 파일만 집계합니다.** 프로젝트의 `camera_cal/` 파일이 포함된 폴더에서 다시 실행하면 해당 보정 파일은 더 이상 집계되지 않으며(90장의 이미지 세트에서 “96장의 이미지 발견”), 이미 해당 파일을 가져온 프로젝트에서는 해당 파일이 제거됩니다.
> - **`lattice multi-test`는 카메라의 전원 켜짐 상태에서 시작됩니다.** 실행을 위해 카메라의 PTP를 켜고, 첫 번째 트리거가 발생하기 전에 클럭이 동기화되고 안정화될 때까지 기다립니다. 모든 카메라가 전원 켜짐 시점부터 약 2분(120초) 동안 동기화됩니다 (`--ptp-settle SECONDS`가 설정함). 갓 동기화된 클럭은 약 2분 동안 최대 ~200 us의 오차를 보이기 때문입니다. 또한 각 카메라의 PTP를 원래 상태로 되돌립니다. 이전 테스트에서 이미 동기화된 상태인 카메라는 다시 대기하지 않습니다. 이전에는 DAQ-E가 응답하지 않는 단일 스위치 네트워크에서 PTP를 비활성화 상태로 두고, 각 카메라의 자유 주행 클럭을 동기화되지 않는 PTP 클럭으로 인식했습니다(`PTP is not locked`, 매 실행마다). [다중 카메라(일시적 세션)](#multi-camera-transient-sessions)을 참조하십시오.
> - **`.daq` 기록은 캡이 선택되었는지 또는 가정되었는지를 나타냅니다.** DAQ-U, DAQ-M 및 DAQ-E는 캡 없이 출하되며, Chloros는 캡이 장착되었는지 여부를 확인할 수 없으므로, 캡 이름이 지정되지 않은 상태로 연결된 센서는 기본적으로 `sunshine_cosine`를 할당받으며, 이는 *추정*. 이제 모든 Chloros 기록(스키마 1.30)은 `cap_id`와 함께 `cap_id_source`를 저장합니다: 캡에 이름을 지정한 경우(`none` 포함)에는 `operator`가, 기본값인 경우 `auto_default`가 저장됩니다. 캡이 가정된 기록을 처리할 때, 프로젝트의 DAQ 캡을 ‘자동’으로 설정해 두면, 파일 이름을 지정하는 `[daq-dls] WARNING` 메시지가 출력됩니다: 햇빛 보정 처리되지 않은 DAQ-U는 약 30배 높게 측정됩니다. 연결 시 **적용한 캡에 이름을 지정**하거나 프로젝트의 DAQ 캡을 설정하면 경고가 사라집니다.
> - **Windows에서 더 이상 pip 콘솔 창이 표시되지 않습니다.** Python이 설치된 PC에서 백엔드와 각 이미지 작업자는 시작 시 `pip`를 한 번씩 실행했고, 각각 콘솔 창을 열었습니다. 이제는 더 이상 그러지 않습니다.
> - **실시간 `.daq` 기록은 정전 시에도 유지됩니다.**기록은 센서 뒤에서 매 커밋 시 저널과 전체 디스크 동기화를 통해 작성되므로, 따라서 디스크 속도가 느려도 파일 저장만 지연될 뿐 측정값 자체는 절대 지연되지 않습니다. 커밋 도중 중단된(정전, 강제 종료) 기록 파일 옆에는 `<name>.daq-journal` 파일이 남을 수 있습니다:**두 파일을 함께 복사하십시오.**Chloros는 파일을 처음 읽을 때 저널을 롤백하므로, 기록 파일은 중단되기 전까지 커밋된 모든 내용이 포함된 상태로 열립니다(마지막 약 3초 분량이 손실됨). 1.2.0 버전에서는 부적절한 시점에 정전이 발생하면 전체 파일이 읽을 수 없는 상태가 될 수 있었습니다("데이터베이스 디스크 이미지가 손상되었습니다").
> - **사용 중인 디스크에서의 기록 작업이 더 이상 센서를 멈추게 하지 않습니다.** `daq pool-record`는 읽기 작업을 대기열에 넣고 별도의 스레드에서 이를 기록합니다. 기록되지 못한 읽기 작업은 (`daq pool-list`는 마지막 오류가 발생한 `N write error(s)`를 표시함) 센서가 멈추는 대신 처리됩니다(4대의 카메라가 TIFF 파일을 저장하는 동안 한 번의 쓰기 작업에 8.4초가 소요된 사례).
> - **`lattice multi-capture`는 배열이 없어도 트리거가 타임아웃될 때 저장된 프레임을 더 이상 잃지 않습니다.** 타임아웃된 카메라의 트리거(`SC_ERR_TIMEOUT -1011`, 1GbE 스위치 하나에 연결된 4대의 카메라에서 확인됨)는 해당 프레임 그룹에서 실행을 종료합니다: 이미 저장된 그룹은 `Saved:`로 나열된 후, 오류 메시지와 함께 `1` 종료 메시지가 표시됩니다. 1.2.0 버전에서는 파일이 디스크에 존재함에도 불구하고 목록 없이 HTTP 500 오류가 발생했습니다.
> - **소프트웨어 트리거 방식의 카메라가 더 이상 트리거 신호를 번갈아 가며 놓치지 않습니다.** 소프트웨어 트리거 모드의 카메라는 이제 판독 중에 도착하는 트리거를 대기열에 넣으며(또는 Chloros가 준비될 때까지 대기합니다), 이는 케이블 동기화 카메라에서 이미 지원되던 방식과 동일합니다.
> - **`daq pool-discover`는 DAQ-E-S `DAQ-E-S`를 지정합니다.** `MODEL` 열에 표시됩니다(`--json`의 `model`도 마찬가지입니다). 모든 이더넷 센서에 대해 `DAQ-E`라고 표시되었으며, 약 2초 이내에 신원 확인에 응답하지 않는 센서도 여전히 `DAQ-E`로 나열되어 있습니다.
> - 모든 처리 모드에서 `process`의 **LATTICE 지수 산출 결과**(NDVI 및 기타 지수)가 개선되었습니다.
> - **OCN 카메라: 햇빛 조건에서 방사도, 반사도 및 지수 산출 결과가 개선되었습니다.** 밴드 분리 후 플릿 보정이 적용됩니다. RGN 카메라는 변경되지 않았으며, OCN 값은 1.2.0 버전과 다릅니다. 이는 이전 버전으로 촬영된 OCN 캡처 데이터도, 해당 캡처가 촬영된 당시의 보정 정보를 보유한 컴퓨터에서 처리될 경우 적용됩니다 (카메라가 한 번 연결되면 Chloros는 해당 카메라의 보정 정보를 유지합니다).
> - **`lattice continuous`는 원시 프레임을 기록합니다.** `tiff` 실행 시 디모자이징된 3-플레인 프레임을 기록했으며, `process`는 이를 기반으로 시작할 수 없습니다. 이제 해당 프레임은 어레이 캡처가 작성하는 `raw` 파일 형태입니다: 센서 자체 값을 가진 모자이크 플레인 하나, 비압축 상태이며, 보정 패킷이 포함되어 있습니다. 이 파일들에서 세 개의 플레인을 읽는 스크립트는 하나의 플레인만 읽습니다.
>
> - **모든 캡처는 GUI와 마찬가지로 사용된 DAQ 판독값을 저장합니다.** `lattice capture`, `lattice continuous`(프레임당 `.daq` 하나), `array-capture` 및 `multi-capture`는 연결된 어레이(및 SDK)와 함께 `.daq` 및 해당 `.match.json`를 기록합니다. 여기에는 DAQ가 바인딩되거나 풀에 등록될 때마다, 원시 전용 캡처가 포함됩니다. 1.2 버전에서는0에서는 원시 또는 디베이어링된 캡처가 저장될 때 `--force-daq`만 기록됩니다. `--no-daq`는 이 기능을 비활성화하고, `--daq-tolerance-ms MS`는 해당 캡처에 대한 일치 허용 오차를 설정하며, 새로운 `daq pool-bind`는 CLI를 통해 카메라 또는 어레이를 DAQ에 바인딩합니다. `.daq`로 캡처가 저장되면, `process`에서는 명시된 카메라와만 매칭됩니다. 이제 캡처는 노출 전후의 판독값을 혼합할 수 있도록 DAQ의 다음 판독값을 기다립니다. [캡처가 저장하는 DAQ 판독값](#the-daq-reading-a-capture-saves) 및 [노출 시점과 멀리 떨어진 DAQ 판독값](#a-daq-reading-far-from-the-exposure)을 참조하십시오.
> - **`process --indices NAME`**는 각 카메라의 밴드에 대한 인덱스를 프로젝트에 추가하며, 새로운 프로젝트가 포함됩니다. 1.2.0 버전에서는 `Unknown index name(s) ignored`를 통해 프로젝트에 아직 나열되지 않은 이름은 무시되었습니다.
> - **라이브 어레이 옆의**`process`**: 이제 카메라나 DAQ를 포함하는 백엔드를 재시작하지 않으며, 종료 단계에 있는 실행에 대해서는 최대 15초까지 대기합니다.**Ctrl+C**는 CLI뿐만 아니라 백엔드에서도 실행을 중지합니다. [`process` 참고 사항](#notes)을 참조하십시오.
> - **Windows의 260자 경로 제한을 초과하는 지나치게 깊은 프로젝트 폴더**의 경우, 아무것도 기록하지 않는 실행 대신, 제한 내에 들어맞는 가장 긴 폴더를 사용하여 처음부터 거부됩니다. Windows 설치 프로그램에서는 이제 "Windows 긴 경로 사용" 옵션과 백엔드에 대한 Microsoft Defender 제외 설정을 제공하며, 두 항목 모두 기본적으로 선택되어 있습니다. [Windows (.exe)](#windows-exe)를 참조하십시오.
> - **`lattice array-capture --continuous`**는 백엔드 자체의 연속 캡처 기능을 실행합니다. 2초마다 진행률 바가 표시되고, 파일별 목록 대신 요약 정보가 제공되며, 기록되지 않은 파일이 있거나 드라이브 용량이 가득 찬 경우 0이 아닌 종료 코드를 반환합니다. [`array-capture` 모드](#array-capture-modes)를 참조하십시오.
> - **`array-connect` 이후의 첫 번째 `array-capture`**는 HTTP 503 "Array still warming up" 오류로 실패하는 대신, 어레이의 첫 번째 프레임 그룹이 도착할 때까지 최대 20초 동안 대기합니다.
> - **ROI 내 어레이 노출:** 작은 ROI의 경우 더 이상 어레이 구성원의 노출 시간을 센서의 프리런 주기(768 × 304 기준 4.9 ms)로 제한하지 않습니다. [어레이 구성원의 노출](#exposure-on-an-array-member)을 참조하십시오.
> - **자동 노출이 더 이상 어레이의 자동 프레임 속도를 낮추지 않습니다.** 케이블 동기화 어레이는 네트워크와 카메라의 판독 속도가 허용하는 속도로 작동하며, 자동 노출은 해당 속도가 제공하는 주기를 채운 후 게인을 높입니다. 해당 기간보다 긴 수동 노출(`lattice exposure --set`, `configure --set`)은 거부되지 않고, 해당 기간에 맞추기 위해 트리거 속도가 느려지며, 구성 요소가 자동 모드로 돌아오면 속도가 다시 정상으로 회복됩니다. [어레이 구성원의 노출](#exposure-on-an-array-member)을 참조하십시오.
> - **어레이는 불완전한 프레임 그룹 발생 빈도가 줄어듭니다**, 또한 503 오류("새로운 동기화된 프레임 그룹 없음") 발생도 줄어듭니다: 각 슬레이브 카메라의 트리거 입력은 이제 글리치 제거 처리가 적용됩니다.
> - **압축된 12비트 전송** (`--packed`, 기본값 `auto`)은 전체 센서 어레이의 프레임 속도를 약 3분의 1 정도 높여줍니다. 이제 `array-connect --packet-size`가 적용됩니다(1.2.0 버전에서는 이를 수용했으나 무시했습니다). [압축된 12비트 전송](#packed-12-bit-transport---packed)을 참조하십시오.
> - **카메라의 전원 재시작 후 배열이 자동으로 복구됩니다.** 스위치 전원 재시작 후 `array-connect`의 배열은-카메라가 응답하면 다시 열고, PTP가 다시 활성화되면 스트리밍을 재개합니다. 1.2.0에서는 영구적으로 중단되어 새로운 `array-connect`가 필요했습니다. `array-status`는 카메라가 PTP를 읽는 동안 `ptp_enabled: False`(및 `ptp_off:`)를 보고하는 반면, 이전에는 어레이가 연결된 설정을 보고했습니다. [지속적 연결](#persistent-connections-smart-prep-gui-equivalent-flow)을 참조하십시오.
> - **`lattice array-record`는 GUI가 열려 있지 않은 상태에서 기록합니다.** GUI의 통합 보기가 닫혀 있으면 백엔드가 뷰 자체를 렌더링합니다. 즉, 어레이의NDVI 또는 새로운 `--expr`와 함께 전달한 인덱스를 렌더링합니다. 프레임을 기록하지 않는 녹화는 이제 실패합니다(`no-frames: …`, 종료: `1`). 이전에는 GUI의 통합 보기가 열려 있는 동안에만 기록되었으며, 아무것도 기록하지 않는 경우에도 여전히 `0`에서 종료되었습니다. [`array-record`](#array-record--combined-index-videogif-monitoring-grade)를 참조하십시오.
> - **Chloros가 이미 보유하고 있는 카메라는 두 번째로 열리지 않습니다.** 해당 카메라를 직접 열려는 명령어는 대신 HTTP 409 `CAMERA_HELD` 응답을 반환하며, 카메라를 보유하고 있는 대상과 이를 해제하는 명령어의 이름을 명시합니다. 여기에는 현재 다른 명령어가 사용 중인 카메라도 포함됩니다(카메라를 여는 `multi-test` 또는 `multi-capture`, `lattice capture`): 첫 번째 명령어가 완료될 때까지 두 번째 명령어는 거부됩니다. 카메라를 다시 열면 해당 카메라를 보유한 장치의 스트림이 중단되거나 백엔드가 충돌할 수 있습니다. [지속적 연결](#persistent-connections-smart-prep-gui-equivalent-flow)을 참조하십시오.
> - **일시적인 다중 카메라 명령은 모든 카메라를 전달하거나 실패합니다.** 배열이 연결되지 않은 `lattice multi-capture`, `multi-stream`, `multi-test`, `align-calibrate`, `align-apply`, `align-stream` 및 `index --live`는 프레임을 전송하지 않은 카메라 이름을 지정하여 `1`에서 종료됩니다. 스트림은 카메라가 누락된 프레임 그룹을 건너뛰고 3개 연속으로 누락되면 중지됩니다. `multi-test`는 트리거마다 한 행을 출력하며, `(no frame)`는 카메라가 누락된 위치를 표시하고, `align-stream`는 작업 오류 시 `1`가 작업 오류로 종료됩니다. 이후 각 실행에서는 모든 카메라를 발견된 상태 그대로 복원합니다: 트리거 및 동기화 라인 배선, PTP (`PtpEnable`) 및 실행 중에 변경된 모든 카메라 설정(노출 및 게인, AE 목표값, 한계값 및 댐핑, 화이트 밸런스, 블랙 레벨, 픽셀 포맷, 비닝 및 ROI, 플립, 프레임 속도)을 원래 상태로 복원합니다. 단, `--keep-settings` / `--clear-settings`가 실행되지 않는 한(자세한 내용은 [카메라 메모리](#camera-memory----keep-settings----clear-settings) 참조). 이전에는 이러한 명령어가 들어오는 프레임을 모두 유지한 채 `0`에서 종료되었으며(`align-stream`는 `Note:`로 작업 오류를 출력함), 모든 카메라를 `TriggerMode` `On` 상태로 남겨두었습니다. 또한 프레임을 저장하기 전(최대 몇 초)의 장면에서 각 카메라의 노출을 설정하여, 연결된 어레이가 각 카메라를 40 멀티스펙트럼, 60RGB(`--brightness N`가 설정)로 설정하고, 촬영이 진행되는 동안 이 설정을 유지합니다; Chloros가 보유하지 않은 단일 카메라의 `lattice capture`도 마찬가지입니다. 이전에는 각 카메라의 첫 번째 프레임이 조명 조건과 상관없이 카메라 자체의 노출 제한(약 100 ms)에 따라 촬영되었기 때문에, 종종 클리핑 현상이 발생했습니다. [다중 카메라(일시적 세션)](#multi-camera-transient-sessions)를 참조하십시오.
> - **일시적 다중 카메라 명령은 `array-connect`와 동일한 방식으로 네트워크를 공유하며, `--serials`만 엽니다.** 각 카메라는 호스트 링크의 할당된 대역폭에 맞춰 속도가 조절되므로, 1 Gbps 링크에서는 4대의 카메라가 작동합니다. 이전에는 4대가 총 125 MB/s(1 GbE 전체)를 요청했고, 매 프레임마다 패킷 손실이 발생했습니다. 네트워크가 감당할 수 없는 어레이는 어떤 카메라도 스트리밍을 시작하기 전에 거부됩니다. 이제 `--serials`가 카메라를 선택하며, 트리거를 먼저 발동하는 카메라가 우선됩니다. 이전에는 `--serials`의 설정과 상관없이 모든 명령이 네트워크상의 모든 카메라를 열었습니다. GVSP 패킷 손실로 인한 오류는 이제 M8 동기화 케이블을 원인으로 지목하는 대신 해당 오류를 직접 표시합니다. [다중 카메라 (일시적 세션)](#multi-camera-transient-sessions)을 참조하십시오.
> - **`lattice array-disconnect`는 카메라를 발견된 상태 그대로 자유롭게 작동하도록 둡니다.** `TriggerMode`, `Off` 및 연결 시 확인된 나머지 모든 설정 — 트리거 및 싱크 라인 설정(`TriggerSource`, `TriggerActivation`, `TriggerOverlap`, `TriggerDelay`, `LineMode` / `LineSource`), PTP, 그리고 세션 중에 변경된 카메라 설정까지 — 따라서 이전 멤버에게이전 구성원에 대한 단일 카메라 명령만으로도 프레임을 한 번에 가져올 수 있으며, 마스터는 더 이상 `TriggerSource`, `Software`에 머물러 있지 않습니다. `--keep-settings`는 세션 설정을 각 카메라의 메모리에 저장하며, `--clear-settings`는 해당 메모리를 초기화합니다([카메라 메모리](#camera-memory----keep-settings----clear-settings) 참조). `lattice continuous`는 또한 무장 상태로 감지된 카메라를 해당 실행 동안 자유 실행 모드로 설정하며(실행이 끝나면 원래 상태로 되돌립니다), 프레임이 없는 실행은 `1`에서 종료됩니다.
> - **`lattice configure --get` / `lattice exposure`**는 `note:`를 출력하며, 여기에는 카메라의실제 노출값이나 게인을 출력합니다. 거부된 `lattice exposure --set` / `configure --set`는 카메라가 허용하는 범위를 표시하며, 자동 노출을 끄는 대신 자동 모드로 되돌립니다. [센서 튜닝](#sensor-tuning)을 참조하십시오.
> - **새로운 카메라 제어 기능:**`lattice highlights` (RGB 디스플레이 렌더링에서 밝은 흰색 표면의 명도를 낮추는 기능) 및 `lattice exposure --brightness-auto`.**`lattice multi-detect`**는 한 번의 프로브 스캔으로 유선 카메라를 찾아내고, 마스터를 RGB 카메라로 기본 설정합니다.
> - **새로운 DAQ 명령어:** `daq pool-bind`; `pool-enable-ptp`; 그리고 DAQ-E의 자체 설정용으로는 `pool-info`, `pool-set-network`, `pool-set-auto-stream`, `pool-reboot` 및 `pool-update-firmware`가 있습니다. [DAQ-E 장치 설정](#daq-e-device-settings)을 참조하십시오.
> - **데이터 전송이 중단된 DAQ-E는 백엔드에 의해 다시 연결됩니다.** 풀에 포함된 DAQ-E의 스트림이 중단될 경우(전원을 공급하는 스위치의 전원 재시작, 카메라 링크 끊김)을 겪으면, 백엔드는 해당 장치가 가지고 있던 설정으로 다시 연결하며, 연결이 끊어진 동안 간격을 점차 늘려가며 재시도합니다. 해당 장치에서의 녹화는 동일한 `.daq`에서 계속되며, 파일 내에 간격이 표시되고, `pool-record --stop` 파일에는 각 간격이 나열됩니다. 1.2.0 버전에서는 센서가 `pool-disconnect` + `pool-connect` 파일이 생성될 때까지 데이터 전송이 중단된 상태로 유지되었으며, 이때 녹화도 종료되었습니다. [데이터 전송이 중단되는 DAQ-E](#a-daq-e-that-stops-delivering)를 참조하십시오.
> - **`time-sync status`**는 PTP 그랜드마스터의 패킷이 네트워크에 도달하는지 여부를 보고하며(`health`), 도달하지 않을 경우 표준 오류 출력(stderr)에 `WARNING:`가 출력됩니다. [`time-sync`](#chloros-cli-time-sync)를 참조하십시오.
> - **Linux .deb:** 카메라 스트리밍에 필요한 커널 소켓 수신 버퍼를 늘리고(커널 기본값에서는 프레임 손실 및 프레임 속도 저하가 발생함), 사용자 권한으로 실행되는 백엔드 서비스를 추가하며, `lattice network`가 버퍼를 확인합니다. CLI는 서비스가 실행하는 백엔드를 절대 중지하지 않으며, 업그레이드 시 활성화된 백엔드 서비스가 다시 시작됩니다. [백엔드를 서비스로 실행하기](#running-the-backend-as-a-service-headless) 및 [Linux 호스트 — 소켓 수신 버퍼](#linux-hosts--socket-receive-buffers)를 참조하십시오.
> - **`lattice array-capture`는 정렬에 실패한 경우 다음과 같이 표시합니다:**캡처가 정렬되지 않은 상태로 저장된 각 사유(정렬 프로필 없음, 서로 다른 해상도의 구성 요소, 케이블 동기화가 되지 않은 배열, 프로파일에 매핑할 수 없는 프레임)은 `Alignment:` 행으로 출력됩니다.**`chloros-cli daq help`**는 `daq --help`이며(`2`에서 종료됨), Linux의 도움말에는 해당 빌드에 포함된 직접 하드웨어 `daq` 명령어들이 가 포함되어 있습니다.
> - **Jetson:** 이 패키지는 JetPack 6용인 `chloros_1.3.0_arm64_jp6.deb`이며, GPU 처리는 JetPack의 CUDA PyTorch를 사용하여 Orin에서 실행됩니다. [Linux arm64 — Jetson](#linux-arm64--jetson-jetpack-6)을 참조하십시오.

---

## 규칙

- 모든 명령어의 접두사는 `chloros-cli`입니다. Windows에서는 바이너리 이름이 `chloros-cli.exe`이며, Linux/Jetson에서는 `chloros-cli`입니다.
- 선택적 인수는 `--flag` 형식으로 표시됩니다. 필수 위치 인수는 괄호 없이 표시됩니다.
- 기본값이 지정된 경우, 플래그를 생략하면 해당 값이 사용됩니다.
- CLI는 Chloros 백엔드(데스크톱 앱에서도 사용하는 `127.0.0.1:5000`의 서버)를 통해 작동하는 경량 HTTP 클라이언트입니다. **이를 시작하는 명령어는 다음과 같습니다:**
  - 핵심 명령어 — `process`, `login`, `logout`, `status`, `export-status`, `time-sync` 및 `selftest` — 실행 중인 백엔드를 사용하거나, 실행 중인 백엔드가 없을 경우 백엔드를 시작하고, 작업이 완료되면 다시 중지합니다.
  - **`lattice`, `project` 및 `daq` 명령어는 백엔드를 절대 시작하지 않습니다.** 이 명령어들은 이미 실행 중인 백엔드가 필요합니다: Windows 데스크톱 앱의 백엔드, Linux [백엔드 서비스](#running-the-backend-as-a-service-headless), 또는 사용자가 직접 시작한 백엔드([백엔드 시작](#start-the-backend) 참조). 백엔드가 없으면 `Error: Chloros backend not reachable at http://127.0.0.1:5000` 오류 메시지와 함께 백엔드를 시작하는 명령어를 출력한 후, `1` 오류로 종료됩니다.
- `CHLOROS_BACKEND_URL=<url>`는 **`lattice`**,**`project`**및**`daq pool-*`** 명령어 계열을 다른 백엔드로 지정합니다(`lattice network-analysis`는 대신 `--backend-url`를 지정합니다). 핵심 명령어들은 의도적으로 `http://127.0.0.1:<port>`를 고정하고 이를 무시합니다 (IPv4 리터럴을 사용하면 Windows에서 발생하는 `localhost`→`::1` 요청당 약 2초의 성능 저하를 피할 수 있습니다). [환경 변수](#environment-variables)를 참조하십시오.
- **상대 경로는 `chloros-cli`를 실행하는 폴더를 기준으로 해석됩니다.** 백엔드는 출력 및 입력 폴더를 여는 프로세스이므로, 경로를 전송하기 전에 CLI는 이를 절대 경로로 변환합니다: `-o` / `--output` (기본값인 `output` 포함), `--input`, `--burst-dir`, `--profile`, `--save` 및 `--mask`, 그리고 [레시피](#recipe-dsl) 단계의 `output`(기본값: `output`), 그리고 `lattice index --live`의 기본 `output` 폴더입니다. 따라서 `-o captures/`는 `<current folder>/captures/`에 기록하며, 출력된 경로는 절대 경로입니다. `~`는 확장됩니다. 원격 백엔드(다른 머신을 지칭하는 `CHLOROS_BACKEND_URL`)의 경우, 경로는 입력된 대로 전송되며 해당 머신에서 해결됩니다.
- 모든 SDK/CLI 호출 시 Chloros+ 계정 로그인이 필요합니다(머신당 한 번씩 `chloros-cli login`를 실행하면 `~/.chloros/`에 캐시됨).
- 예제에서는 Linux 경로를 사용합니다. Windows에서는 `/home/user/...`를 `C:/Users/.../...`로 대체하십시오. DAQ-U'의 직렬 포트 형식은 Windows에서는 `COM3` 형식이며, Linux에서는 `/dev/ttyUSB0` / `/dev/ttyACM0` 형식입니다.
- 개요에서는 상호 배타적인 플래그를 `{--a | --b}`로, 선택적 플래그는 `[...]`로 작성합니다.

---

## 최상위 개요

```
chloros-cli [global options] COMMAND [command options]
```

### 전역 옵션

| 플래그 | 설명 |
| --- | --- |
| `--backend-exe PATH` | 자동 감지된 백엔드 실행 파일을 재정의합니다. |
| `--port N` | 백엔드 HTTP 포트 (기본값: `5000`). 코어 명령어에서만 사용됨 (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`); `lattice`, `project` 및 `daq`는 `CHLOROS_BACKEND_URL`(기본값인 `http://127.0.0.1:5000`)를 사용합니다. |
| `-v, --verbose` | 상세 출력 활성화. |
| `--restart` | 백엔드를 강제 재시작합니다(실행 중인 모든 `backend_server.py`를 종료합니다). systemd 서비스에 의해 실행되는 백엔드 (Linux .deb)로 실행되는 백엔드는 절대 중지되지 않습니다. 이 경우 명령어는 대신 `sudo systemctl restart <unit>`를 출력하고 종료됩니다 `1`. |
| `--version` | 버전 정보 출력 (`Chloros CLI 1.3.0`). |
| `--help` | 최상위 도움말 표시. |

### 명령어 색인

| 명령어 | 용도 |
| --- | --- |
| [`process`](#chloros-cli-process) | Survey3 또는 LATTICE 캡처 파일이 포함된 폴더를 종단 간으로 처리합니다. |
| [`login`](#chloros-cli-login) | Chloros+ 계정으로 이 컴퓨터를 인증합니다. |
| [`logout`](#chloros-cli-logout) | 캐시된 자격 증명을 지웁니다. |
| [`status`](#chloros-cli-status) | 현재 라이선스/인증 상태를 표시합니다. |
| [`export-status`](#chloros-cli-export-status) | `process` 실행 중 Thread-4 내보내기 진행 상황을 실시간으로 표시합니다. |
| [`language`](#chloros-cli-language) | CLI 표시 언어 설정 또는 목록 확인 (38개 언어 지원). |
| [`set-project-folder`](#project-folder-commands) / [`get-project-folder`](#project-folder-commands) / [`reset-project-folder`](#project-folder-commands) | 기본 프로젝트 폴더 (GUI와 공유). |
| [`update`](#chloros-cli-update) | Chloros 업데이트 확인 및 설치 (Linux/Jetson). |
| [`selftest`](#chloros-cli-selftest) | 시스템 진단 및 스모크 테스트. |
| [`time-sync`](#chloros-cli-time-sync) | PTP 그랜드마스터 상태 / 제어. |
| [`lattice`](#chloros-cli-lattice) | LATTICE 카메라 제어 및 캡처 (45개 이상의 하위 명령어). |
| [`daq`](#chloros-cli-daq) | DAQ 스펙트럼 센서 제어 (DAQ-U / DAQ-M / DAQ-E). |
| [`project`](#chloros-cli-project) | 저장된 Chloros 프로젝트(카메라 + DAQ) 열기 및 실행. |

---

## 설치

`chloros-cli`는 지원되는 모든 플랫폼의 Chloros 데스크톱 설치 프로그램에 포함되어 제공됩니다 — 별도의 CLI 다운로드 파일은 없습니다. 플랫폼 패키지를 설치하면 데스크톱 앱 및 이를 구동하는 백엔드 바이너리와 함께 `chloros-cli`가 `PATH`에 추가됩니다.

최신 다운로드: [`https://mapir.gitbook.io/chloros/download`](https://mapir.gitbook.io/chloros/download)

> 또한 설치 프로그램에는 바로 사용할 수 있는 CLI 셸을 여는 편리한 런처 스크립트(`Chloros_CLI.bat` / `Chloros_CLI.ps1`, `Launch_CLI.*`, `chloros-cli.sh`)를 포함하고 있으며, 이를 통해 바로 사용할 수 있는 CLI 셸을 실행할 수 있습니다.

### Windows (.exe)

1. 다운로드 페이지에서 Windows 설치 프로그램을 다운로드합니다.
2. `Chloros Installer 1.3.0.exe`를 실행하고 마법사의 안내에 따라 진행합니다. 기본 설치 경로는 `C:\Program Files\Chloros\`입니다(CLI는 `C:\Program Files\Chloros\cli\`에 위치하며, 설치 프로그램이 이 경로를 PATH에 추가합니다).
3. 업데이트된 `PATH`가 인식되도록 새 터미널(`cmd.exe`, PowerShell 또는 Windows 터미널)을 엽니다.

```powershell
chloros-cli --version
```

설치 프로그램은 `chloros-cli.exe`를 시스템의 `PATH`에 자동으로 추가하고 LATTICE 카메라 드라이버를 함께 포함합니다. 백엔드는 `C:\Program Files\Chloros\backend\chloros-backend.exe`(설치 폴더 내의 `backend` 폴더)이며, 이는 `PATH`에는 포함되어 있지 않습니다.

**시스템 설정.** 설치 프로그램의 *추가 작업 선택* 페이지에는 **시스템 설정**아래에 두 가지 옵션이 있으며, 둘 다**기본적으로 선택되어** 있습니다:

| 옵션 (설치 프로그램에 표시된 문구) | 기능 | 자동 설치 제외 |
| --- | --- | --- |
| **Windows 긴 경로 활성화** (프로젝트 폴더가 260자보다 더 깊은 위치에 있을 수 있도록 함; 권장) | Windows의 시스템 전체 설정인 `LongPathsEnabled`를 1로 설정합니다. Windows는 이 설정이 비활성화된 상태로 제공되며, 이 경우 경로 길이가 260자를 초과할 수 없습니다. Chloros 제품은 프로젝트 폴더 아래로 최대 약 130자까지 위치하므로, 더 깊은 위치의 프로젝트 폴더는 허용되지 않습니다([문제 해결](#troubleshooting-pointers) 참조). 설치 후 실행되는 프로그램부터 적용됩니다. 다른 프로그램에서 이 설정에 의존할 수 있으므로, 제거 시에도 이 설정은 유지됩니다. | `/MERGETASKS="!longpaths"` |
| **Microsoft Defender 검사에서 Chloros 백엔드 제외**(카메라 캡처 속도 대폭 향상; 권장) | 설치 폴더 내의 `chloros-backend.exe`에 대한 Defender**프로세스** 제외 항목을 추가하여, 백엔드가 파일을 어디에 기록하든 실시간 보호 기능이 해당 파일을 검사하지 않도록 합니다. 다른 프로그램은 제외되지 않으며, 제거 시 이 제외 항목은 삭제됩니다. Defender가 없거나 정책에 의해 잠겨 있는 경우 이 단계는 오류 메시지 없이 실패하고 설치가 계속 진행됩니다. | `/MERGETASKS="!defenderexclusion"` |**Defender 옵션이 중요한 이유:** 캡처 속도가 높을 경우, 캡처 파일에 대한 실시간 스캔은 캡처 속도의 약 30%를 차지합니다(9대의 카메라로 연속 캡처 시 측정 결과: 77.7 캡처/초 제외, 52.7 스캔). `lattice array-capture --continuous`는 캡처 시 Defender가 CPU 코어 1개 이상을 사용하는 것으로 측정되면 알림을 표시합니다. 해당 옵션을 선택한 상태로 설치 프로그램을 다시 실행하십시오.

자동 설치(`/SILENT` 또는 `/VERYSILENT`)의 경우, 명시적으로 제외하지 않는 한(예: `"Chloros Installer 1.3.0.exe" /VERYSILENT /MERGETASKS="!defenderexclusion"`) 두 옵션이 모두 적용됩니다.

**Windows 방화벽.** 설치 프로그램은 모든 네트워크 프로필에 대해 항상 `chloros-backend.exe`에 대한 두 개의 인바운드 규칙을 추가하며, 제거 프로그램은 이를 제거합니다: 카메라 스트림, PTP, mDNS 및 DAQ-E 데이터 스트림용 규칙인 `Chloros Backend (UDP-In)`와, 센서가 이미지를 수집하기 위해 이 컴퓨터에 다시 연결되는 DAQ-E 펌웨어 업데이트용 규칙인 `Chloros Backend (TCP-In)`(로컬 서브넷으로 제한됨)가 있습니다(자세한 내용은 [DAQ-E 장치 설정](#daq-e-device-settings) 참조). 백엔드의 HTTP API는 `127.0.0.1`에 계속 바인딩된 상태로 유지됩니다.

### Linux amd64 (.deb)

Ubuntu 22.04 LTS 이상 또는 Debian 기반 x86_64 워크스테이션용입니다.

> **Ubuntu 20.04는 지원되지 않습니다.** 패키지의 의존성 목록은
> 백엔드가 실제로 링크하는 대상에서 파생되며, 여기에는 `libc6 (>= 2.35)`가 포함됩니다.
> focal은 glibc 2.31을 제공합니다. `apt`는 런타임 시 오류가 발생하도록 내버려 두기보다는
> 설치 자체를 거부합니다. 이 목록에는 `libgl1`도 포함되어 있는데, 이는 번들된 OpenCV가 로드하는 것으로, 최소
> 서버 이미지에는 포함되지 않을 수 있습니다. `apt install`가 이를 제공합니다.

```bash
sudo apt install ./chloros_1.3.0_amd64.deb
chloros-cli --version
```

이 .deb 패키지는 다음을 설치합니다:
- `chloros-cli`부터 `/usr/bin/chloros-cli`까지, 그리고 `chloros-backend` (백엔드를)를 `/usr/bin/chloros-backend`로
- 컴파일된 백엔드와 CLI를 압축 해제하여 `/usr/lib/chloros/backend/` 및 `/usr/lib/chloros/cli/`에 배치
- LATTICE 카메라 드라이버
- 노이즈 제거 모델, 보정 번들 및 업데이트 채널 구성
- 시스템용 Python SDK: `python3` (`import chloros_sdk`). `apt install`에는 필수 구성 요소인 `python3-numpy` 및 `python3-requests`도 포함되어 있습니다(권장 사항이며 필수 사항은 아님: 위 프로그램들은 독립적으로 작동하므로 `dpkg -i`는 이를 건너뛰며, 설치 출력에서 누락된 항목을 명시합니다). virtualenv 환경에서는: `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl`
- `/etc/sysctl.d/60-chloros-gige.conf`를 설치하면, 카메라 스트림용 커널 소켓 수신 버퍼를 32MB로 늘리고 이를 즉시 적용합니다(이미 더 높은 값으로 설정된 경우 변경되지 않습니다). 패키지를 제거하면 해당 파일이 삭제됩니다. [Linux 호스트 — 소켓 수신 버퍼](#linux-hosts--socket-receive-buffers)를 참조하십시오.
- 백엔드용 선택적 systemd 서비스 두 개가 있으나, 둘 다 활성화되어 있지 않습니다(다음 섹션 참조).

#### 백엔드를 서비스로 실행하기 (헤드리스 모드)

서비스가 없으면 백엔드(그리고 이에 따른 PTP 그랜드마스터)는 다음 중 하나를 통해 시작된 동안에만 실행됩니다: 터미널에서 ``chloros-backend``를 실행하거나, SDK 스크립트를 사용하거나, ``chloros-cli`` 명령어를 실행하는 동안에만 백엔드가 작동합니다. 헤드리스 머신에서 백엔드를 상시 가동하려면 **사용자 계정으로** 서비스를 활성화하십시오:

```bash
sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
systemctl status chloros-backend@$USER.service
```

`chloros-backend@<user>.service`는 해당 사용자로 백엔드를 실행합니다. 백엔드가 기록하는 캡처 데이터는 해당 사용자의 것이며, 상세 로그는 해당 사용자의 `~/.cache/chloros/logs` 파일에 저장됩니다. 해당 사용자의 `chloros-cli login`(`~/.chloros`)를 사용하므로, 먼저 해당 사용자로 한 번 로그인하십시오. PTP는 루트 권한이 필요하지 않습니다. 설치된 모든 백엔드와 마찬가지로, 이 백엔드는 `127.0.0.1:5000` 포트에서만 수신 대기합니다.

`chloros-backend.service`는 백엔드를 **root** 권한으로 실행합니다. 이 백엔드는 기능을 활성화한 시스템이 계속 작동하도록 설치되어 있지만, 루트 백엔드가 기록하는 모든 내용은 루트 소유이므로 사용자는 자신의 캡처를 삭제할 수 없습니다. 자신의 사용자로 전환하려면:

```bash
sudo systemctl disable --now chloros-backend.service
sudo systemctl enable --now chloros-backend@$USER.service
```

root 서비스의 상세 로그는 `/var/log/chloros`에 있으며, `adm` 그룹의 구성원이 읽을 수 있습니다(서비스는 `CHLOROS_LOG_DIR`를 설정합니다; [환경 변수](#environment-variables) 참조). 패키지를 설치하면 사용자 이름이 입력된 `enable` 명령어가 출력되며, 루트 서비스가 활성화되어 있을 때는 전환 명령어도 함께 출력됩니다.

**백엔드 유닛 하나를 활성화합니다.** 한 번에 하나의 백엔드만 실행됩니다. 두 번째로 활성화된 유닛(`chloros-backend@<user>.service` 외에 `chloros-backend.service`, 또는 두 사용자의 `@` 유닛)은 시작되지 않고 종료 상태 98인 `failed` 상태로 유지되며, systemd는 이를 재시작하지 않습니다. 해당 유닛의 `systemctl status`에는 그 이유가 명시되어 있습니다: `Another instance is already running. Exiting.`, 그 다음에는 백엔드를 점유하고 있는 대상(포트 5000, 또는 잠금 파일 및 해당 파일의 소유자), 이어서 `systemd will not restart <unit> (exit status 98)`가 표시됩니다. 재부팅 후에는 두 유닛 중 하나가 실행될 수 있습니다. 전환하려면 위와 같이 사용하지 않는 유닛을 비활성화하십시오.**CLI는 서비스의 백엔드를 절대 중지하지 않습니다.** `chloros-cli`는 GUI 및 SDK와 마찬가지로 이를 있는 그대로 사용합니다: `process`는 이를 기반으로 실행되며, 마지막 프로젝트 대신 자체 프로젝트를 불러옵니다. CLI가 일반적으로 백엔드를 재시작해야 하는 경우(`--restart` 또는 임포트 시 아무것도 찾지 못한 `process`), 이 유닛의 이름을 지정하고 `sudo systemctl restart <unit>`를 출력한 뒤 종료됩니다(`1`), 이는 이러한 방식으로 중지된 백엔드는 계속 비활성화된 상태로 남기 때문입니다. `GET /api/status`는 유닛 이름을 `service`로 지정합니다(서비스가 실행되지 않는 백엔드의 경우 `null`).**업그레이드 및 제거.** 패키지를 업그레이드하면 모든 백엔드 서비스가 중지된 후, 활성화된 서비스부터 차례로 다시 시작되며, `@<user>` 유닛이 먼저 시작됩니다(수동으로 시작되었으나 활성화된 적이 없는 유닛은 재부팅 후와 마찬가지로 중지된 상태로 유지됩니다). 활성화된 인스턴스가 두 개 이상인 경우, 하나만 실행되고 나머지는 위와 같이 실패 상태로 유지됩니다. 업그레이드 시 `Chloros: N backend units are enabled: …` 및 `Keep one enabled: sudo systemctl disable --now <unit>`가 출력됩니다. 제거하면 해당 인스턴스가 중지되고 비활성화됩니다. 이 항목을 완전히 삭제하면 `/var/log/chloros`도 함께 삭제됩니다.

### Linux arm64 — Jetson (JetPack 6)

```bash
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
chloros-cli --version
```

amd64 .deb 파일과 동일한 레이아웃이며, **JetPack 6** (L4T R36)을 사용하는 Jetson Orin / Orin NX / Orin Nano용 수신 버퍼 및 [백엔드 서비스](#running-the-backend-as-a-service-headless)가 포함되어 있습니다. 이 패키지의 PyTorch는 Orin용 JetPack 6의 CUDA 빌드이므로, GPU 처리 및 텍스처 인식 디베이어가 포함된 기능이 Jetson의 GPU에서 실행됩니다. 이 패키지에는 CUDA 자체가 포함되어 있지 않습니다. JetPack 6이 설치하는 CUDA 및 cuDNN 라이브러리를 사용합니다(해당 라이브러리가 없는 이미지의 경우 `sudo apt install nvidia-jetpack`). 사용 가능한 CUDA GPU가 없는 경우, 처리는 CPU에서 실행되며 텍스처 인식 기능은 표준 디베이어를 사용합니다([`process`](#chloros-cli-process) 참조).

### 기기당 한 번의 인증

모든 플랫폼에서 SDK/CLI 호출이 작동하려면 Chloros+에 한 번만 로그인해야 합니다:

```bash
chloros-cli login user@example.com 'YourPassword'
```

인증 정보는 `~/.chloros/user_session.json`에 캐시됩니다.

### 백엔드 시작

`lattice`, `project` 및 `daq` 명령어는 이미 실행 중인 백엔드와 통신하며, 백엔드를 직접 시작하지는 않습니다([약관](#conventions) 참조). 이 명령들을 사용하는 동안 백엔드를 계속 실행해 두십시오:

| 플랫폼 | 방법 |
| --- | --- |
| Windows | Chloros 데스크톱 앱을 실행하거나(이 앱의 백엔드는 CLI도 지원합니다), 별도의 터미널(설치 폴더 내의 `backend` 폴더)에서 `"C:\Program Files\Chloros\backend\chloros-backend.exe"`를 실행한 후 창을 열어 둡니다. |
| Linux / Jetson | 별도의 터미널에서 `chloros-backend`를 실행하거나, [백엔드 서비스](#running-the-backend-as-a-service-headless)를 통해 항상 하나를 실행 상태로 유지하십시오. |

한 번에 하나의 백엔드만 실행되며, 모든 클라이언트(GUI, CLI, SDK 스크립트)가 이를 공유합니다. 핵심 명령어(`process`, `login`, `status`, …)도 이를 사용하며, 실행 중인 백엔드가 없으면 하나를 시작합니다.

**백엔드는 명령어 간에도 계속 실행됩니다**. 따라서 다음 명령어가 실행될 때 다시 백엔드를 시작할 필요가 없습니다. 이 프로세스는 로그아웃할 때(Chloros 앱이 열려 있거나 실행 또는 오프라인 빌드가 진행 중이 아닌 경우, `chloros-cli logout`), `chloros-cli backend stop`를 실행할 때, 또는 30분 동안 이를 사용하는 것이 없을 때 자동으로 중지됩니다: 명령어, 스크립트, 앱, 실행 또는 오프라인 빌드, 연결된 카메라나 광 센서가 없는 경우입니다. `backend stop`는 실행 또는 오프라인 빌드가 진행 중일 때 이를 거부하며 해당 작업을 명시합니다; `--force`는 즉시 중지시키며, 해당 실행 또는 빌드는 손실됩니다 (버스트 데이터는 유지되며, `array-build-video`가 이를 다시 빌드합니다). 백엔드가 시작되기 전에 `CHLOROS_BACKEND_IDLE_MIN`를 설정하여 30분 간격을 변경하십시오(`0`: 절대 안 됨); [백엔드 서비스](#running-the-backend-as-a-service-headless)는 절대 스스로 중지되지 않습니다. `chloros-cli backend status`는 실행 중인 백엔드가 있는지, 무엇이 이를 사용하고 있는지, 그리고 실행 중인 오프라인 빌드가 있는지 여부를 보여줍니다. `process`는 Chloros 앱이 프로젝트를 열어둔 상태로 실행 중인 백엔드에서 해당 프로젝트를 엽니다 열어둔 프로젝트가 있는 경우: 이 경우 요청을 거부하며, 앱에서 해당 작업을 처리하거나 먼저 해당 프로젝트를 닫아야 합니다. 이전 `process` 또는 SDK 스크립트가 열어둔 프로젝트는 단순히 대체됩니다.

### 설치 확인

```bash
chloros-cli --version           # prints "Chloros CLI 1.3.0"
chloros-cli selftest            # 7-step diagnostic (backend, GPU, models, CUDA)
chloros-cli status              # shows license tier + logged-in user
```

`selftest`는 7가지 검사 항목이 모두 통과된 경우에만 종료됩니다. 마지막 검사 항목은 CUDA GPU(NVIDIA GPU, 또는 Jetson Orin)과 노이즈 제거 모델이 필요하므로, 해당 장치가 없는 컴퓨터에서는 `Failed: CUDA + Denoiser`가 표시되고 `1`에서 종료됩니다. 이 경우 GPU 전용 텍스처 인식 디베이어를 제외한 모든 기능은 정상적으로 작동합니다.

> **Chloros+ 구독이 필요합니다.**CLI를 사용하려면 유효한 Chloros+ 요금제가 필요합니다.**Copper**는 Chloros+의 기본 요금제입니다. 유료 Chloros+ 요금제는 모두 CLI/SDK에 접근할 수 있으며, 무료**Iron** 요금제만 예외입니다. (요금제-ID 매핑: `0`=Iron/무료, `1`=Copper, `2`=Bronze, `3`=Silver, `4`=Gold.) [`https://cloud.mapir.camera/pricing`](https://cloud.mapir.camera/pricing)에서 업그레이드하십시오.
>
> 이 하한선은 CLI뿐만 아니라 백엔드에서도 적용됩니다. 유료 요금제가 없는 상태에서 SDK/CLI 플래그가 지정된 요청은 `403 PLAN_UPGRADE_REQUIRED` 오류 코드와 함께 거부되며, 이는 `chloros-cli`, Python SDK, 또는 직접 작성한 HTTP 클라이언트에서 전송된 요청 모두에 해당됩니다. 로그아웃된 호출자는 대신 `401 AUTH_REQUIRED` 오류를 받게 됩니다. 요금제의 유예 기간(월간 요금제의 경우 30일, 연간 요금제의 경우 만료일까지) 동안에는 오프라인 상태에서도 액세스가 가능하며 이 기간이 만료되면 중단됩니다. `chloros-cli status`는 계속 작동하므로 오류 사유를 확인할 수 있습니다(이는 티어 제한에서 면제되는 유일한 SDK/CLI 경로인 `GET /api/license-status`입니다).

---

## 5분 퀵스타트

```bash
# 1. Sign in once on this machine
chloros-cli login user@example.com 'YourPassword'

# 2. Survey3 / LATTICE folder → finished radiance + NDVI in one call
chloros-cli process "/home/user/captures/flight_001" \
  --vignette --reflectance --indices NDVI GNDVI OSAVI

# 3. The lattice / daq commands need a running backend: on Windows open the
#    Chloros desktop app; on Linux start one (or see "Start the Backend")
chloros-backend &

# 4. Take a single LATTICE photo with the first camera found
chloros-cli lattice capture -o output/

# 5. Connect a 4-cam LATTICE array with the GUI's smart-prep flow
chloros-cli lattice array-connect \
  --serials 213800234,214000533,214701288,214701292

# 6. Read a spectrum from a connected DAQ-U (Windows: --port COM3)
chloros-cli daq pool-connect --port /dev/ttyUSB0
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F   # id from 'daq pool-list'
```

`process`(2단계)는 백엔드가 실행 중이지 않을 때 자체 실행을 위해 백엔드를 시작하고, 작업이 완료되면 중지하므로, 4단계부터는 백엔드를 직접 시작해야 합니다. [백엔드 시작](#start-the-backend)을 참조하십시오.

---

## `chloros-cli process`

이미지가 포함된 폴더를 전체 Chloros 파이프라인(타겟 감지 → 보정 → 비네트 → 반사율 → 인덱스 내보내기)을 통해 처리합니다.

### 개요

```
chloros-cli process INPUT [OPTIONS]
```

### 위치 매개변수

| 매개변수 | 설명 |
| --- | --- |
| `INPUT` | `.raw + .jpg`(Survey3), `.tif/.tiff`(LATTICE) 또는 `.dng` 파일이 포함된 입력 폴더의 경로. |

### 일반 옵션

| 플래그 | 기본값 | 설명 |
| --- | --- | --- |
| `-o, --output PATH` | 기본 프로젝트 폴더 내에 `-n`라는 이름의 새 폴더 | 프로젝트 폴더. 프로젝트 이름은 이 폴더 이름을 따며, 이미 `project.json` 파일이 있는 폴더는 덮어쓰기되지 않고 `_1` / `_2`라는 동급 폴더가 생성되며, 기존 폴더가 덮어쓰이지 않습니다. 기본 프로젝트 폴더는 `set-project-folder`(또는 GUI)가 선택한 폴더이며, 그렇지 않은 경우 `Documents/MAPIR/Chloros Projects`입니다. |
| `-n, --project-name NAME` | 자동 (타임스탬프) | 프로젝트 이름, `-o`가 **없는** 경우에만 사용됨 (`-o`가 있는 경우 프로젝트는 폴더 이름을 따름). |
| `--debayer {standard,texture-aware}` | `standard` | `texture-aware`는 Chloros+ 신경망 디베이어를 사용하여 Survey3 이미지를 처리합니다. 속도는 느리지만 품질은 더 높습니다. 이는 Survey3 전용입니다: LATTICE 캡처는 항상 표준 디모자이크를 사용하므로, LATTICE 캡처가 포함된 폴더의 경우 `process`는 가져오기 후 `texture-aware`를 거부하며, 파일 수를 명시하는 오류 메시지를 표시한 뒤 `1`를 표시한 후, 어떤 처리도 수행하지 않은 채 종료됩니다(Chloros 앱에서도 해당 이미지에 대해 Texture Aware 기능을 제공하지 않습니다). CUDA GPU가 필요합니다(Windows/Linux x64에서는 NVIDIA GPU, Jetson에서는 Orin의 GPU): 이것이 없으면 실행 시 `standard`가 사용되며, 백엔드에는 그 이유를 설명하는 `[DEBAYER] … needs a CUDA GPU …` 로그 한 줄이 기록됩니다(`--verbose`로 표시됨). |
| `--vignette / --no-vignette` | `--vignette` | 비네트 보정. |
| `--reflectance / --no-reflectance` | `--reflectance` | 반사율 보정 (패널 타겟이 발견되면 이를 사용하며, LATTICE의 경우 NIST 일련번호별 보정값을 사용). LATTICE 다중 스펙트럼의 경우, 이는 반사율 **곱** 역할도 겸합니다 토글 — [제품별 내보내기 토글](#per-product-export-toggles-lattice-multispectral)을 참조하십시오. |
| `--ppk` | off | 사이드카 파일의 PPK GNSS 보정값을 적용합니다. |
| `--exposure-pin-1 CAMERA_MODEL` | none | PPK 시간 동기화를 위해 노출 핀 1에 연결된 카메라 모델입니다. |
| `--exposure-pin-2 CAMERA_MODEL` | 없음 | PPK 시간 동기화를 위해 노출 핀 2에 연결된 카메라 모델. |
| `--recal-interval SECONDS` | `0` | 재보정 최소 간격(초 단위, 프로젝트의 *최소 재보정 간격*). |
| `--timezone-offset HOURS` | `0` | UTC 기준 광센서의 시간대 오프셋(시간 단위)(프로젝트의 *광센서 시간대 오프셋*). |
| `--format FORMAT` | `TIFF (16-bit)` | 다음 중 하나: `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `TIFF (8-bit)`, `PNG (8-bit)`, `JPG (8-bit)` 중 하나입니다. Chloros는 이미지의 비트 심도를 절대 높이지 않습니다: 모든 8비트 이미지의 경우 더 높은 비트 심도의 형식은 `TIFF (8-bit)`로 기록되며, 실행 시 이에 대해 한 번만 언급됩니다. 8비트와 12/16비트 캡처가 혼합된 프로젝트에서는 각 캡처가 자체 비트 심도 또는 그 이하로 내보내집니다. Radiance는 항상 32비트 부동소수점입니다. |
| `--indices NAME [NAME ...]` | 없음 | 사전 설정 이름별 식생 지수 (`NDVI`, `GNDVI`, `OSAVI`, `SAVI`, `EVI`, `GCI`, …), 대소문자 구분 없음. 각 지수는 해당 카메라의 밴드에 따라 프로젝트에 추가되며, 새로운 프로젝트가 포함됩니다. 사전 설정에 없는 이름이나, 카메라의 밴드로 생성할 수 없는 (예: `NDRE`는 RE 및 NIR가 필요하지만, 이를 모두 제공하는 단일 카메라는 없음), 경고 메시지와 함께 표시되며 인덱스 이미지는 생성되지 않습니다. |
| `--input-level {auto,raw,debayered,processed}` | `auto` | LATTICE TIFF에 대한 파이프라인 진입점을 강제 적용합니다(Survey3 .raw는 영향을 받지 않습니다). 또한 **RAW 파일이 없는** 캡처를 처리할 수 있게 해주는 비상 조치도 있습니다 — [캡처 폴더의 구조](#what-a-captures-folder-looks-like)를 참조하십시오. |
| `--debayered / --no-debayered` | on | 선형 디베이어링 처리된 결과물을 출력합니다 (`Debayered_Images`). [제품별 내보내기 토글](#per-product-export-toggles-lattice-multispectral)을 참조하십시오. |
| `--preview / --no-preview` | 켜짐 | 디스플레이 미리보기를 출력합니다(`Preview_Images`): RGB = 화이트 밸런스(사용 가능한 경우 DAQ 광원, 그렇지 않으면 그레이 월드) + 감마; 멀티스펙트럼 = 가색 확장. |
| `--radiance / --no-radiance` | 켜짐 | float32 형식의 복사도 출력 (`Radiance_Images`, W/m²/sr/nm). |
| `--reflectance-source {daq,target,auto}` | `auto` | LATTICE 반사율 산출값에 대한 기준: `auto` = QA를 통과한 인프레임(in-frame) 타겟이 절대 기준이며(QA 플래그가 있는 타겟은 측정 점수에서 DAQ와 경쟁함), DAQ 하향 복사(ρ = π·L/E)가 대체 기준임; `target` = 엄격 모드(DAQ 대체 없음); `daq` = DAQ 우선 모드. [제품별 내보내기 토글](#per-product-export-toggles-lattice-multispectral)을 참조하십시오. |
| `--target-reflectance-dir DIR` | 없음 | 단위별 **측정된** 목표 반사율 스캔 디렉터리 (`<serial>.csv`); 검출 실패 시 명목상 T3/T4P 스펙트럼으로 대체됩니다. |
| `--array-alignment / --no-array-alignment` | on | LATTICE 어레이: 각 캡처의 `Chloros:Alignment*` XMP에 기록된 모듈 간 정렬 정보를 모든 처리된 제품(디베이어링 / 미리보기 / 복사도 / 반사율 / 지수)에 적용합니다. 태그가 없는 이미지의 경우 아무 작업도 수행하지 않습니다. |
| `--array-alignment-crop / --no-array-alignment-crop` | 자르기 | 정렬된 출력물을 어레이의 공통 중첩 영역으로 자르며, 모든 모듈이 하나의 풋프린트를 공유하도록 합니다. `--no-…`는 전체 센서 캔버스를 유지합니다(소스 외부는 검은색으로 채움). |
| `--array-alignment-interp {bilinear,nearest,cubic}` | `bilinear` | 정렬 왜곡에 대한 리샘플링. `nearest`는 원본의 정확한 디지털 노벨(DN) 값을 보존합니다(방사계 값의 픽셀 간 혼합 없음). |

### 타겟 감지 옵션

| 플래그 | 설명 |
| --- | --- |
| `--min-target-size PIXELS` | 검출기에 대한 최소 보정 타겟 크기(픽셀 단위). |
| `--target-clustering 0-100` | 타겟 클러스터링 비율(기본값: `60`). |
| `--target / --targets` | 입력 파일의 `target` 또는 `targets` 하위 폴더에서만 보정 타깃을 검색합니다(처리 속도가 더 빠름). |

### 예시

```bash
# Simplest: defaults are good for most surveys
chloros-cli process "/home/user/images/survey_001"

# Multi-index with explicit format
chloros-cli process "/home/user/images/survey_001" \
  --vignette \
  --reflectance \
  --format "TIFF (32-bit, Percent)" \
  --indices NDVI GNDVI OSAVI

# Texture-aware debayer for highest quality (Chloros+ only)
chloros-cli process "/home/user/images/survey_001" \
  --debayer texture-aware \
  --indices NDVI

# Process LATTICE captures explicitly (auto-detects from EXIF normally)
chloros-cli process "/home/user/captures/lattice_flight" \
  --input-level processed

# LATTICE multispectral → float32 radiance only (no DAQ downwelling needed)
chloros-cli process "/home/user/captures/lattice_flight" \
  --no-debayered --no-preview --no-reflectance

# LATTICE reflectance anchored to an in-frame target (strict, no DAQ fallback),
# with per-unit measured target scans looked up by serial
chloros-cli process "/home/user/captures/lattice_flight" \
  --reflectance-source target --target-reflectance-dir "/home/user/target_scans"

# LATTICE array capture: keep native geometry (ignore stamped alignment)
chloros-cli process "/home/user/captures/array_flight" \
  --no-array-alignment

# Aligned, uncropped, value-preserving resampling
chloros-cli process "/home/user/captures/array_flight" \
  --no-array-alignment-crop --array-alignment-interp nearest

# Save to a project folder of your choice (the project is named after it)
chloros-cli process "C:/input" -o "C:/projects/Field_A_2026-05-26"

# Name the project and let it go to the default project folder
chloros-cli process "C:/input" -n "Field_A_2026-05-26"
```

### 제품별 내보내기 토글 (LATTICE 다중 스펙트럼)

LATTICE 처리는 **한 번의 처리 과정으로 해당되는 모든 제품**으로 분기됩니다. 네 가지 유형별 토글 — `--debayered`, `--preview`, `--radiance`, `--reflectance` —는 모두**기본적으로 켜져** 있습니다. 하나를 비활성화하려면 `--no-<type>` 형식을 사용하십시오. RGB 마스터 카메라는 항상 디베이어링 처리된 + 미리보기 신호만 출력하며(대역별 방사도/반사도 없음), 따라서 `--radiance`/`--reflectance`는 해당 카메라에 대해 아무런 동작도 수행하지 않습니다. 이 토글 설정은 (표준 반사도/타겟 경로를 따르는) Survey3의 `.raw`에서는 무시됩니다. *(기존의 `--radiometric-output {reflectance,radiance,sensor-response}` 플래그는 **제거**되고 이 토글들로 대체되었습니다. 더 이상 `sensor-response` 레벨은 존재하지 않습니다.)*

| 제품 | 출력 | DAQ 다운웰링 필요? |
| --- | --- | --- |
| `--debayered` | 선형 디모자이크 (`Debayered_Images`): 캡처가 `debayered` 레벨에서 저장하는 파일과 동일 — 원본 데이터에서 보정 팩의 다크 오프셋(DSNU)을 제거한 후, OpenCV의 에지 인식 디모자이크를 적용함; 선형성 LUT, 플랫 필드 또는 언믹스 처리 없음. | 아니요. |
| `--preview` | 디스플레이 미리보기 (`Preview_Images`): RGB = 화이트 밸런스(WB) + 감마; 다중 스펙트럼 = 가색(false-colour) 스트레치. | 없음. |
| `--radiance` | 전체 방사계측 체인(`Radiance_Images`)에서 도출된 float32 W/m²/sr/nm. | 번호 |
| `--reflectance` | uint16 반사율 ρ (`32768` = 1.0), Pix4D 호환. | **예**, QA를 통과한 프레임 내 타겟이 이를 고정하지 않는 한 (아래 참조). |

`--reflectance-source`는 반사율 기준값을 선택합니다:**`auto`**(기본값)는 QA를 통과한 프레임 내 타깃을**절대 기준**으로 설정합니다 — 타겟에 고정된 경험적 라인 체인은 제외된 패널에서 교차 평가되며, 측정된 최상값이 밴드별로 적용됩니다: DAQ로 보정할 수 없거나 DAQ 기울기 적합에 실패한 밴드는 해당 타겟의 라인을 (`ReflectanceElBandMethods`가 해당 대상을 지정함) — 플래그(패널 누락, 밴드 순서 오류)가 있는 상태에서 QA를 통과한 대상도 해당 측정 점수에 대해 DAQ 분할값과 경쟁하며, 이 경쟁에서 패배한 프레임에는 `ReflectanceChain=daq`가 표시됩니다 — 타겟이 없거나 QA에 실패할 경우 DAQ 하향 분할(ρ = π·L/E)로 대체되며,**`target`**는 엄격한 방식(DAQ 대체 없음)을 따릅니다;**`daq`**는 DAQ의 결정에 따르는 동작을 선택합니다. 타겟 기하 구조(ArUco / 고정 ROI / 스트립)는 프로젝트 타겟 구성에서 가져옵니다; `--target-reflectance-dir DIR`는 단위별**측정된** 스캔(`<serial>.csv`)을 유지하며, 대상 장치의 일련번호/QR을 통해 조회하고, 명목상 T3/T4P 스펙트럼을 대체값으로 사용합니다.

DAQ 반사율 경로는 기록된 **`.daq`**(DAQ-U/M/E)**또는 이미지와 함께 발견된 DAQ-M 네이티브 `.csv`**에서**타임스탬프가 일치하는 하향파**를 자동으로 추출합니다. 카메라별 또는 DAQ 보정 번들이 로컬에 캐시되어 있지 않은 경우, 파이프라인은 첫 사용 시**AWS에서 이를 자동으로 가져옵니다**(인터넷 연결이 한 번 필요하며, `~/.chloros/`에 캐시됨).**이 측정값은 누구의 것인가.**캡처가 저장하는 `.daq`는 해당 캡처가 설명하는 카메라(들)의 측정값이며, 이는 바로 옆에 있는 `<file>.daq.cameras.json` 사이드카에 명시되어 있습니다 (`lattice capture`, `multi-capture`, `array-capture` 및 GUI에서도 마찬가지입니다. 원시 버스트(raw burst)는 매니페스트에서 각 구성원의 측정값을 명시합니다) — 그리고 `process`는**오직 해당 카메라들의 이미지와만**에 대해서만 일치합니다. 따라서 자체 측정값이 없는 독립형 카메라는 더 이상 다른 카메라의 DAQ를 빌리지 않습니다. DAQ를 해당 카메라에 바인딩하거나, 직접 녹화해야 합니다. DAQ 자체의 녹화 (`daq pool-record`, 또는 GUI에서 시작된 기록)이나, 해당 사이드카가 없는 모든 `.daq`(구버전 캡처 포함)는 여전히 모든 카메라에 적용됩니다.

#### 반사율 픽셀 읽기 (Pix4D / Metashape / 사용자 스크립트)

반사율은 정수형 DN으로 저장되며, **ρ = 1.0을 나타내는 DN 값은 소스 카메라에 따라 다릅니다**:

| 소스 | ρ = 1.0에 해당하는 값 | 확인 방법 |
| --- | --- | --- |
| LATTICE (M3C / M3M) | `32768` (ρ 2.0까지 여유 범위) | 파일에 XMP `Chloros:PixelScale=32768` 태그가 찍혀 있음. |
| Survey3 | `65535` (ρ 1.0에서 잘림) | `Chloros:*` XMP 태그 없음 — 그 부재 자체가 신호입니다. |

**상수를 가정하기보다는 `Chloros:PixelScale` 값을 읽고 이를 기준으로 나누어 계산하십시오**. 이 태그는 uint16 도메인으로 정의되어 있으므로, 재스케일링되는 출력 형식에서도 `32768`로 유지됩니다 — `TIFF (16-bit)`, `TIFF (8-bit)`, `PNG (8-bit)`, `JPG (8-bit)` 및 `TIFF (32-bit, Percent)`는 모두 자체 설명형입니다 (먼저 저장된 데이터 유형을 uint16으로 정규화해야 합니다: 8비트에서 ×257, 부동소수점에서 ×65535).

> **이전 빌드의 파일에는 스케일이 포함될 수 없습니다.** 8비트 반사율 TIFF 파일은 재스케일링되어 다른 파일들과 마찬가지로 `Chloros:PixelScale`로 표시됩니다. 1.3.0 이전 빌드는 대신 0..255로 *클리핑*했기 때문에, ρ≈0.008을 초과하는 모든 값은 255로 평탄화되었으며, 또한 `Chloros:PixelScale`와 `MicaSense:RadiometricCalibration` 튜플을 모두 생략했습니다. **LATTICE 반사율 파일에 해당 태그가 없는 경우, 스케일을 가정하지 말고 — 재처리하십시오**. 나눌 수 없는 픽셀을 무리하게 나누는 것보다 낫습니다.

#### 인덱스 픽셀 읽기 (NDVI 및 기타 식생 지수)

**정수 인덱스 래스터는 이미지별 대비 확장 결과일 뿐, −1…1 범위의 맵이 아닙니다.** `TIFF (16-bit)`, `TIFF (8-bit)`, `PNG (8-bit)` 및 `JPG (8-bit)` 지수 제품(8비트 캡처 데이터가 8비트 TIFF로 기록된 경우 포함)은

```

DN = (value − IndexMin) / (IndexMax − IndexMin) × full_scale
```

여기서 `IndexMin` / `IndexMax`는 **해당 프레임 자체의 가장 작은 유한 인덱스 값과 가장 큰 유한 인덱스 값**입니다. DN 0은 −1이 아니며, 풀 스케일은 +1이 아닙니다.

클로로스(Chloros)는 매핑이 가역성을 유지하도록 양쪽 끝점에 스탬프를 찍습니다:

| 태그 | 의미 |
| --- | --- |
| `Chloros:IndexScaling` | `linear_minmax` — 아래의 규칙 |
| `Chloros:IndexMin` | DN 0으로 저장된 인덱스 값 |
| `Chloros:IndexMax` | 풀 스케일로 저장된 인덱스 값 |

이들을 읽고 반전시킨 후, 저장된 DN을 먼저 저장된 데이터 유형(dtype)의 최대값을 기준으로 0…1 범위로 정규화하여 하나의 표현식으로 모든 정수 형식을 포괄하도록 합니다:

```python
import json, subprocess
import numpy as np, tifffile

tags = json.loads(subprocess.run(
    ["exiftool", "-j", "-XMP:all", path],
    capture_output=True, text=True).stdout)[0]

# float(): exiftool hands back a negative real as a quoted string, so
# IndexMin arrives as str on most real NDVI data while IndexMax is a number.
lo, hi = float(tags["IndexMin"]), float(tags["IndexMax"])

raw = tifffile.imread(path)                    # uint16 for TIFF (16-bit)
ndvi = lo + raw.astype(np.float64) / np.iinfo(raw.dtype).max * (hi - lo)
```

읽기 작업에는 `-config pix4d.config`가 필요하지 않습니다. exiftool은 별도의 도움 없이도 이러한 XMP 속성을 읽을 수 있습니다. 이 설정은 해당 값을 *쓰기* 위해서만 필요합니다.

> **`IndexMin == IndexMax`**는 해당 프레임의 모든 픽셀이 동일한 인덱스 값을 가졌음을 의미합니다. 래스터는 중간 스케일로 채워져 있으므로 나눌 대상이 없으므로 — 끝점을 취합니다.

> **`Chloros:Index*` 태그가 없다는 것은 명시된 매핑이 없다는 것을 의미합니다.** LUT 내보내기는 픽셀이 LUT 임계값 창에 맞춰 그라데이션 색상으로 처리된 컬러 렌더링이며, 유한한 인덱스 픽셀이 없는 프레임은 명시할 범위가 없습니다. 어느 경우든 매핑이 있다고 가정하지 마십시오 — `PixelScale`가 누락된 경우와 동일한 규칙입니다.**스트레칭은 이미지별로 적용되므로, 한 비행의 두 프레임을 DN 기준으로 비교할 수 없습니다**— 각 프레임은 고유한 끝점을 가지므로, 동일한 NDVI 값이 서로 다른 이미지에서는 서로 다른 DN에 해당합니다. 한 비행 전체에 걸친 정량적 작업(구역별 통계, 고정 임계값, 시계열 분석)을 위해**`TIFF (32-bit, Percent)`**를 내보내십시오: 인덱스 래스터는 재스케일링이나 태그 없이**인덱스 값 자체**가 float32 형식으로 기록됩니다. 형식 이름과 달리, 인덱스 제품은 ×100이 아닙니다 — 해당 스케일링은 정수 데이터형 반사율에만 적용됩니다.

#### 언믹스 클램프 마스크(어떤 픽셀에서 밴드가 클램프되었는지)

3×3 스펙트럼 언믹스는 밴드를 음수로 만들 수 있습니다. 이는 빛에 해당 밴드가 거의 포함되지 않았거나, 장면이 카메라의 언믹스 행렬이 설명하는 범위를 벗어난 경우입니다. 이는 실제 값이 아니므로, 내보내기 과정에서 0으로 고정되며 라이브 뷰에도 동일한 0이 표시됩니다. 밴드는 몇 개의 픽셀에서만 클램핑될 수도 있고 거의 모든 픽셀에서 클램핑될 수도 있으며, 마스크는 두 경우 모두를 기록합니다. `Chloros:Band{R,G,B}UnmixClampFraction`는 프레임의 어느 정도에서 그런 현상이 발생했는지를 나타내고, 클램프 마스크는 **어떤 픽셀**에서 발생했는지를 나타냅니다:

- **`clamp_masks/<product folder>/<product name>_clampmask.tif`**— 제품 폴더**옆**에 있는 `clamp_masks/` 폴더에 저장되며, 절대 그 안에는 저장되지 않습니다(`radiance/x_radiance.tif` → `clamp_masks/radiance/x_radiance_clampmask.tif`; `process` 내보내기 파일의 `tiff16/Reflectance_Calibrated_Images/x.tif` → `tiff16/clamp_masks/Reflectance_Calibrated_Images/x_clampmask.tif`) — 모든 방사도, 반사율 및 굴절률 데이터 세트 중 적어도 하나의 픽셀이 클램핑된 모든 데이터에 대해: 캡처 및 `process` 내보내기 파일 모두(그리고 `array-build-video --save-tiffs`의 프레임별 TIFF 파일)에 적용됩니다. 단일 대역 `uint8`, 해당 산출물 고유의 픽셀 그리드 상에서 생성됩니다(방향, ROI, 비닝 및 배열 정렬 포함). 각 마스크는 해당 산출물의 카메라(`Make` / `Model` / 일련번호), 캡처 시간 및 렌즈 태그를 명시하며, 해당 비트가 나타내는 밴드를 나열합니다 (`XMP-Camera:BandName`, 비트 0부터 순서대로); `ImageDescription` 및 `Software`는 이를 클램프 마스크로 표시하며, 보정 정보는 포함되어 있지 않습니다.
- **비트**: 비트 0 = 제품의 R 채널에 해당하는 대역(`Chloros:BandRName`), 비트 1 = G(`BandGName`), 비트 2 = B (`BandBName`). 비트가 설정되어 있으면 해당 밴드의 픽셀 값이 0으로 클램핑되었음을 의미하므로, 이 비트와 이를 기반으로 생성된 모든 인덱스를 신뢰도가 낮은 것으로 간주하십시오.
- **`Chloros:UnmixClampMask`**는 언믹스(unmix) 처리를 거친 모든 제품에 찍혀 있습니다: 마스크의 경로는 제품 자체 폴더를 기준으로 하며(`../clamp_masks/<product folder>/<product name>_clampmask.tif`, 항상 `/`로 구분되어 있습니다 — 이를 이미지 폴더에 결합하여 마스크를 열 수 있습니다), 또는 클램핑된 픽셀이 없을 경우 `none`가 표시됩니다. 태그가 없는 경우, 해당 제품은 언믹스 처리를 거치지 않았음을 의미합니다(RGB 또는 모노 카메라, RAW/디베이어링 레벨).
- `lattice capture` / `array-capture`는 저장된 파일 아래에 마스크 경로를 출력하며 (`clamp mask: …`)를 출력하며, 캡처 결과에서는 이를 프레임의 `clamp_mask`로 나열합니다.
- `lattice index --live --save-multiband`는 멀티밴드 스택에 대해 소스 카메라당 하나의 마스크를 작성합니다 — [`lattice index` — 멀티밴드 스택 마스크](#lattice-index--multiband-stack-masks)를 참조하십시오. 비디오에는 마스크가 없으며, 대신 `array-build-video`는 각 프레임의 클램프 비율을 보고합니다([`array-build-video`](#array-build-video--offline-reprocess-a-saved-burst) 참조).

어떤 깊이에서도 마스크를 포함하는 제품 폴더가 없기 때문에, 제품 폴더(`radiance/`, `reflectance/`, `process` 내보내기 파일의 `*_Images/` 폴더)는 하위 폴더를 포함한 그대로 Pix4D, Metashape 또는 기타 사진측량 소프트웨어에 전달할 수 있으며 — 하위 폴더 포함 — 그대로 전달할 수 있으며, 해당 소프트웨어는 이미지만 처리합니다. 상위 폴더(해당 폴더에는 `clamp_masks/` 및 기타 모든 내보내기 유형이 포함되어 있음)가 아닌 제품 폴더를 전달하십시오. Chloros의 자체 가져오기 기능은 마스크를 건너뜁니다. 마스크 읽기: `(mask & 1) != 0`는 “이 부분에서 R 채널 대역이 클램핑되었습니다”, `& 2`는 G, `& 4`는 B를 의미합니다.

#### 내보내기 시 전달되는 EXIF

`process`는 원본 캡처의 **GPS 블록과 해당 ExifIFD**를 을 모든 결과물에 복사하므로,
내보내기 시 `FocalLength`, `FNumber`, `ExposureTime`, `ISO`, `DateTimeOriginal` 및
`CameraSerialNumber`를 지리 참조 정보와 함께 포함하게 됩니다.

**`FocalLength`는 사진측량에 있어 선택 사항이 아닙니다.** Pix4D는 초점 거리와 고도를 바탕으로
지상 샘플 거리(GSD)를 계산합니다. 이 태그가 누락되면 극도로 잘못된 축척으로 처리됩니다. 한 번의
49장 촬영으로 이루어진 오렌지 과수원 비행에서, 이 태그가 누락되어 411m × 160m 규모의 부지가 재구성된
47.8 km × 13 km 크기로 재구성해 버렸습니다. 이는 대부분 ‘데이터 없음(nodata)’으로 표시된 455 MP 오르토사진이었으며, GSD를 확인하기 전까지는 타일링 문제나
BigTIFF 문제로 오인되었습니다. 만약 생성된 오르토사진의 축척이 비현실적으로
나온다면, 먼저 내보낸 결과물에 `exiftool -FocalLength`를 실행하십시오.

이 사본은 의도적으로 **`-all:all`가 아닙니다**: IFD0'의 구조 태그가 복사 시 LATTICE 출력을
방해하기 때문이며, `ExifImageWidth` / `ExifImageHeight`는
*원본* 캡처를 기술하기 때문에 제외되었습니다. 크기가 조정된 적이 있는 내보내기 파일은 그렇지 않으면
자신의 래스터와 모순되는 치수를 포함하게 될 것입니다. XMP는 복사하는 대신 직접 작성되는데, 이는 ExifTool이
XMP 블록이 복사될 때 동일한 호출의 XMP 태그를 제거하기 때문입니다(이 경우 MAPIR 보정 태그가 삭제될 것입니다).

### 출력 파일의 저장 위치

생성된 파일은 **프로젝트 폴더 아래에, 카메라별로 그룹화된 후 파일 형식별로** 저장됩니다:

```
<project>/
└── LATT-M3M-L41-F550/                  # one folder per camera model+lens+filter
    ├── tiff16/
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── <INDEX>_Index_Images/        # e.g. NDVI_Index_Images
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

LATTICE의 카메라 폴더는 `LATT-<sensor>-<lens>-F<filter>`이며(캡처의 EXIF
`Model`)이고, Survey3의 경우 `<model>_<filter>`입니다. 센서와 필터를 공유하지만 렌즈가 다른 두 대의 카메라는
비네팅, 시야각 및 왜곡이 다르기 때문에 별도의 폴더 구조로 관리됩니다. 포맷
폴더 구조는 `--format`: `tiff16`, `tiff8`, `png8`, `jpg8`, 또는 `tiff32`와 같이
`TIFF (32-bit, Percent)`의 형식을 따릅니다.

> **내보낸 모든 파일은 원본 파일의 이름을 유지합니다.** > `capture_…_raw.tif`의 방사도 내보내기 파일은 여전히 `capture_…_raw.tif`라고 불리지만, 실제로는
> `tiff32/Radiance_Images/`에 저장됩니다. **파일 이름이 아닌 폴더가 제품을 식별합니다**, 따라서
> `*radiance*.tif`를 글로빙(globbing)해도 아무것도 찾지 못합니다. 대신 디렉터리를 기준으로 일치 여부를 확인하십시오.

### 광센서 기록 — 보정된 `.daq` + `.csv`

`process`는 입력 폴더에 있는 `.daq` 기록 파일도 처리하며, 이를 수행하는 데 **이미지 데이터는 필요하지 않습니다**
이미지가 필요하지 않습니다: 단독으로 비행된 DAQ-U / DAQ-M / DAQ-E는 완전한
캡처 데이터이며, `.daq` 파일만 포함된 폴더도 유효한 입력입니다.

DAQ는 보정 **없이**도 기록할 수 있습니다. 이는 공개용
[`chloros_scripts`](https://github.com/mapircamera/chloros_scripts) 레코더
(`record_daq.py`)가 기본적으로 수행하는 작업입니다. 이 레코더들은 원시 센서 카운트 데이터를 기록하고 파일에 타임스탬프를 찍어,
*Chloros가 시리얼을 통해**(먼저 로컬 캐시,
그 다음 MAPIR 클라우드)를 통해 해당 센서의 공장 보정값을 가져와 적용합니다. `process`는 그 결과를 다시 기록합니다:

```
<project>/
└── Light Sensor/
    ├── <name>_calibrated.daq        # reprocessable archive, declares its bundle
    └── <name>_calibrated.csv        # W/m^2/nm per reading + photometric columns
```

`.csv`는 각 측정값마다 한 행을 포함합니다: UTC 타임스탬프, 적분 시간, 총 전력,
광시/암시 럭스, PPFD(및 청색/녹색/적색 분할값), 피크 파장, 그리고
센서 고유의 파장 그리드를 기반으로 한 전체 스펙트럼 순서입니다. `.daq`는
두 번째 보정 과정 없이 다시 가져옵니다.

`<name>`는 해당 기록의 파일 이름입니다. 서로 다른 폴더에 있는 두 개의 기록은
두 대의 카메라가 동일한 밀리초에 저장한
단일 캡처별 측정값(`<capture>/daq/daq_data_<sensor id>_<ts>.daq`)을 공유할 수 있으며, 각 기록은 여전히 고유한 산출값을 얻습니다. 첫 번째 기록은
`<name>`를 유지하고, 다음 녹화본은 이름 **뒤에** 해당 녹화본을 구별하는 폴더명을 추가하여
(`<name>__<capture>_daq_calibrated.daq`), 따라서 녹화본의 날짜와 시간이 여전히 앞에 표시됩니다.

성공 시 실행 결과는 `Light-sensor products written: N (calibrated .daq + .csv)`로 보고됩니다.
괄호 안의 내용은 실제로 기록된 내용을 설명하므로,
번들이 없는 센서의 경우 `(RAW COUNTS — this sensor has no calibration bundle)`로,
둘 다 포함된 폴더의 경우 `(N calibrated, M raw counts)`로 표시됩니다. 백엔드 자체의
`[DAQ-EXPORT]` 및 `[RUN-SUMMARY]` 제목도 동일한 방식으로 표현됩니다. 이
세 가지 모두 원시 내보내기 데이터를 ‘보정된’ 것으로 간주할 수 없습니다.

보정 번들을 가져올 수 없는 DAQ-U / DAQ-M / DAQ-E 기록(오프라인 상태이거나 해당 센서에 파일로 저장된 보정 정보가 없는 경우)은 **사유와 함께 건너뜁니다**.
`[DAQ-EXPORT]` 행에 기록되며, 원시 카운트를 포함하는 “보정된” 파일로 출력되지 않습니다.
인터넷에 연결한 후 다시 실행하십시오. 이유는 리더가 해당 파일에 대해 실제로
확인한 사유(읽을 수 없는 스키마, 번들 없음, 쓰기 오류)이며, 실행
요약에는 **별개의** 사유들이 나열됩니다 — 하나의 원인으로 인해 건너뛴 20개의 파일은 하나의
원인으로 표시되며, 동일한 원인이 20번 반복된 것으로 표시되지 않습니다. CLI는 실행이 끝날 때 건너뛴 각 기록에
이유와 함께 이름을 지정합니다:

```
⚠ Light-sensor recording not exported: <file>.daq: no calibration bundle available for …
```

녹화 파일 중 **단 하나도** 내보내지 못하는 폴더는 실패합니다:
[이미지를 하나도 쓰지 않는 실행은 실패합니다](#a-run-that-writes-no-images-fails)를 참조하십시오.

이름이 `_calibrated` 또는 `_raw`(`field_raw.daq`)로 끝나는 녹화 파일은 **가져오지**가져오지 마십시오**: 이는 `process`가 자체 산출물에 부여하는 이름이며, 산출물을
녹화 파일로 다시 읽어들이면 중복으로 집계됩니다. CLI는 이를 알리며, 빈 폴더라고 보고하는 대신 파일 이름인
`Not imported as a light-sensor recording: field_raw.daq … Rename the file to import it.`
를 표시합니다. 이러한 기록은 처리하기 전에 이름을 변경하십시오.

#### DAQ-A 기록은 원시 카운트 값으로 내보내집니다

**DAQ-A** 제품군은 일련번호별 번들 시스템이 도입되기 이전에 출시된 것으로, 불러올 교정 번들이
없습니다. 대신 현장에서 반사율 타겟을 기준으로 교정되므로
번들이 필요하지 않았습니다. 이러한 녹화 데이터를 거부하면 데이터를
전혀 추출할 방법이 없어지므로, **다른 이름**으로 내보내집니다:

```
<project>/
└── Light Sensor/
    ├── <name>_raw.daq        # NOT _calibrated
    └── <name>_raw.csv        # raw spectral sensor counts, NOT irradiance
```

파일 내부의 플래그가 아닌 별도의 파일 이름을 사용하는 이유는, 이 정보가
단순한 파일 이름 형태로 이메일을 통해 전송될 때도 유효해야 하기 때문입니다. `.csv` 헤더에는
`raw spectral sensor counts (NOT irradiance)`라고 명시되어 있으며, 이 값들은 파일 **내부**에서만
비교 가능하다는 경고가 있습니다 — 이는 바로 타겟 기반 보정이 이를 사용하는 목적이며 —
센서 간에는 비교할 수 없습니다. 전력에 의존하는 광도 측정 열(총 전력, 광시야 및
암시야 럭스, PPFD)은 카운트 값을 통합하여 계산하는 대신 **NULL**로 기록되며, 실행
요약에는 `RAW COUNTS`라고 표시되므로 로그에 “내보낸"라고 표시된 로그를 조도로 해석할 수 없습니다.

구형 **v1.01 / v1.02** 기록 파일(DAQ-A-SD가 이를 기록함)에는 판독별 에포크 정보가 없으며,
오직 파일의 기록 시간만 포함됩니다. 이미지↔하향 복사량 매칭 도구는 여전히 이를 거부합니다 —
프레임을 기록 시간과 매칭하는 것은 눈에 띄지 않게 오류가 발생할 수 있기 때문입니다 — 하지만 내보내기 도구는 이를 읽으며,
CSV 파일에는 `clock=daq_created_on`와 같이 출력되므로, 해당 제품이 어떤 시계를 기준으로 하는지 알 수 있습니다.

### 참고 사항

- `process`는 폴더가 Survey3, LATTICE 또는 혼합형인지 자동으로 감지합니다.
- 진행 상황은 Server-Sent Events를 통해 스트리밍되며, CLI는 스레드별 실시간 진행 상황(탐지, 분석, 처리, 내보내기)을 표시합니다.
- Linux/Jetson의 경우, CLI가 스왑 공간을 확인하며 대용량 폴더를 처리하기 전에 경고 메시지를 표시할 수 있습니다. 또한 텍스처 인식 디베이어(texture-aware debayer)는 저전력 Jetson(Nano, Orin Nano)에서 GPU 주파수 제한을 자동으로 적용합니다.
- 성공 시 실행 보고서에 작성된 이미지 산출물의 개수가 표시됩니다(`Image products written: N`, 픽셀이 클램핑된 경우 `(plus M clamp masks)`)..
- **`process`는 실행 중인 백엔드를 안전하게 공유합니다.** 이 백엔드는 이전 프로젝트가 로드된 상태에서 유휴 상태이며, 카메라, 어레이 또는 DAQ를 보유하지 않은 경우에만 이미 실행 중인 백엔드를 자동으로 재시작합니다. 이 중 하나라도 보유하고 있는 백엔드(어레이를 스트리밍하는 GUI, SDK 스크립트, DAQ 녹화 등)는 있는 그대로 사용되며 절대 재시작되지 않으므로, `process`는 활성 어레이와 동시에 실행될 수 있습니다. 백엔드가 실행 중이라고 보고하는 경우, `process`는 해당 작업이 완료될 때까지 최대 15초 동안 대기합니다(방금 완료된 작업은 지워지는 데 약 1초가 소요됨), 그 후 `Backend is mid-run (processing). Wait for it to finish, or pass --restart to force a fresh backend (this kills the in-flight run).` 오류로 실행을 거부합니다. 전역 플래그 `--restart`는 여전히 새로운 백엔드를 강제하며, 해당 백엔드에서 실행 중인 모든 카메라, 어레이 또는 DAQ 세션을 종료합니다. Linux systemd 서비스로 실행되는 백엔드는 절대 재시작되지 않습니다: `process`가 이를 사용하며, 대신 `--restart`가 `sudo systemctl restart <unit>`를 출력합니다(참조: [서비스로 백엔드 실행하기](#running-the-backend-as-a-service-headless) 참조). Chloros 앱 자체의 백엔드도 마찬가지입니다. 앱이 열려 있는 상태에서 `process`가 이를 사용하며, 앱의 프로젝트 대신 해당 프로젝트를 불러옵니다(그 자리에 백엔드가 시작되면 열려 있는 앱이 해당 백엔드에 접근할 수 없게 됩니다).
- **실행 중인 작업은 해당 프로젝트를 유지합니다.** 앱, `process`, SDK 스크립트 또는 `chloros-mcp`에 의해 시작된 처리 실행이 진행 중인 동안, 백엔드는 프로젝트의 파일을 생성, 열기, 가져오기, 추가 또는 제거, 이동, 삭제하거나 프로젝트의 처리 설정을 변경하는 작업을 거부하며, 이 경우 HTTP 409 `PROCESSING_IN_PROGRESS` 및 `A processing run is in progress on project '<name>', so ... now is refused` 오류가 발생합니다. 이를 수신하는 `process`는 `1`로 종료됩니다. 표시 설정(썸네일 크기)은 여전히 저장됩니다. 처리 모드 및 프로젝트의 목표 반사율 설정도 동일한 방식으로 거부됩니다(실행은 시작된 모드를 유지합니다). 실행 도중 프로젝트에 저장된 캡처(앱, CLI, SDK 또는 `chloros-mcp`에서)는 평소와 같이 저장되지만, 실행이 종료될 때만 프로젝트의 파일 목록에 추가됩니다; 이에 대한 응답(`project_add`)에서도 이 점이 명시되어 있으며, 프로젝트 내에서 중지된 광센서 기록도 마찬가지입니다. 이전에는 두 번째 클라이언트가 실행 중인 프로젝트를 전환할 수 있었고, 실행 중인 작업이 다른 프로젝트로 내보내지기도 했습니다.
- **Ctrl+C는 CLI뿐만 아니라 백엔드에서도 실행을 중지합니다.** CLI는 백엔드에 처리 중지를 요청하고, `Waiting for the backend to stop (Ctrl+C again to stop waiting)...`를 출력한 뒤, `Processing stopped.`가 표시될 때까지 최대 120초 동안 대기하므로, `chloros-cli process … && next`는 실행이 진행 중인 백엔드에서 `next`를 시작하지 않습니다. 두 번째 Ctrl+C를 누르면 대기 상태가 종료됩니다. 그러면 `process`는 중단된 명령어와 마찬가지로 `130`를 종료합니다.

#### 이미지를 기록하지 않는 실행은 실패합니다

결과물을 요청했으나 실행 과정에서 **아무것도** 기록되지 않은 경우 — 오직 `project.json`와
`calibration_data.json`만 있는 경우 — `process`는 이를 실패로 간주합니다:
`Processing finished but wrote no image products.`를 출력하고 **0이 아닌 값으로 종료**하므로, 스크립트에서
이를 감지할 수 있습니다. 이 메시지에는 프로젝트 폴더 이름과 일반적인 원인이 명시되어 있습니다:

- 입력 폴더가 캡처로 인식되지 않았음(레이아웃 및 `--input-level` 확인), 또는
- 요청된 모든 산출물이 해당 카메라에 적용 불가능하여 건너뛴 경우(예:
  RGB 전용 카메라에서 방사도/반사도 요청).

`--verbose`를 사용하여 다시 실행하고 백엔드 로그에서 `[LATTICE-EXPORT]` / `[EXPORT-CHECK]` 행을 확인하십시오.
이 로그에는 CLI 출력에는 나타나지 않는 카메라별 건너뛰기 사유가 설명되어 있습니다.

의도적으로 메타데이터만 처리하는 실행(모든 산출물 토글을 끄고 `--indices`를 사용하지 않는 경우)은 여전히
**성공**으로 간주됩니다. 왜냐하면 이 경우 빈 이미지 출력이 올바른 결과이기 때문입니다.

#### 내보낸 파일을 검증할 수 없는 실행은 실패합니다

실행이 끝나면 `process`는 ExifTool을 사용하여 내보낸 파일의 태그를 확인합니다. ExifTool을 실행할 수 없는 경우,
`내보내기 검증 실패: ExifTool을 실행할 수 없으므로, 내보낸 파일 M개 중 N개의 태그가
확인되지 않았습니다 (백엔드 로그의 [EXIFTOOL] 행 참조)` and ` 결과물은 작성되어
원래 위치에 남아 있습니다. 확인을 위해 다시 실행하십시오. `, does not print ` 처리가 완료되었습니다!`라는 메시지를 출력하고 **0이 아닌 값으로 종료**됩니다.
결과물은 유지됩니다.

#### 실행 시 캡처되는 이미지

`process`가 실행되기 전에 완료된 캡처는 포함됩니다. 실행 중에 저장된 캡처는 포함되지 않습니다. `process`는
누락된 캡처의 수를 경고하며, 다시 실행하면 해당 캡처가 포함됩니다.

#### 이미지가 누락된 실행은 실패합니다

실행 중에 이미지가 실패한 경우, `process`는 가장
흔한 원인을 포함한 `N image(s) failed and were not exported`를 출력하고, `Processing complete!`는 출력하지 않으며, **0이 아닌 값으로 종료**됩니다. 다른 결과물들은
작성되어 제자리에 남아 있으며, 백엔드 로그에는 각 실패 내역이 기록됩니다.

**광센서 전용 실행**도 마찬가지입니다: `.daq` 기록 파일로 구성된 폴더는 정의상 내보낼 이미지가 없으며,
대신 작성된 보정된 `.daq` / `.csv` 파일을 기준으로 실행 결과가 평가됩니다.
만약 **아무것도** 기록하지 않았다면 — 모든 기록이 건너뛴 경우 —
`Processing finished but wrote no light-sensor products.`를 출력하고, 각 기록의 이름을 건너뛴
이유로 지정하며, **0이 아닌 값으로 종료**합니다.

---

## `chloros-cli login`

Chloros+ 클라우드 계정으로 이 기기를 인증합니다. 인증 정보는 `~/.chloros/user_session.json`에 안전하게 캐시됩니다.

```
chloros-cli login EMAIL PASSWORD
```

### 예시

```bash
chloros-cli login user@example.com 'YourPassword'

# Passwords containing $ should use SINGLE quotes
chloros-cli login user@example.com 'my$ecret$pass'
```

> **PowerShell `$$` mangling is auto-corrected.** In double quotes PowerShell expands `$$` (비밀번호의 일부를 제거하거나 중복 입력). 401 오류가 발생하면 CLI는 `$$`를 재-을 추가한 다음, 중복 제거된 비밀번호의 절반을 사용하여 재시도합니다. 재시도가 성공하면 로그인이 완료되고, 다음에 사용할 올바른 작은따옴표 구문이 출력됩니다.

> **헤드리스/스크립트 사용: 캐시된 세션이 없으면 로그인 프롬프트가 표시됩니다.** 코어 명령어(`process`, `status`, `export-status`, `time-sync`, …)는 캐시된 세션 없이 실행될 경우, 진행하기 전에 표준 입력(stdin)을 통해 `Email:` / `Password:`를 입력하도록 요청합니다. 표준 입력(stdin)이 닫혀 있는 경우(cron, systemd 유닛 등)에는 `login cancelled` 오류와 함께 즉시 실패하며, 표준 입력이 열린 터미널이나 파이프인 경우에는 입력을 기다립니다. 헤드리스 작업을 스케줄링하기 전에 각 머신당 `chloros-cli login EMAIL PASSWORD`를 한 번씩 실행하십시오.

---

## `chloros-cli logout`

캐시된 세션을 지우고 다음 호출 시 새로운 로그인을 강제합니다.

처리 실행이 진행 중인 상태에서 로그아웃하더라도 처리가 중단되지는 않습니다. 해당 실행은 시작된 처리 모드 그대로 완료되며, 다음 실행은 표준 처리 모드를 사용합니다. `logout`(`The processing run in progress finishes in <mode> mode; the next run uses standard`)에서 이를 명시하고 있으며, 로그아웃 시 앱에서도 로그아웃할 때 앱에서도 이를 안내합니다.

```bash
chloros-cli logout
```

---

## `chloros-cli status`

현재 라이선스 등급(Iron/Copper/Bronze/Silver/Gold), 인증된 사용자 및 기기 바인딩 횟수를 표시합니다.

```bash
chloros-cli status
```

---

## `chloros-cli export-status`

실시간 Thread-4 내보내기 진행 상황을 확인합니다. 다른 셸에서 `process`가 실행되는 **도중**에도 안전하게 호출할 수 있습니다.

```bash
chloros-cli export-status
```

---

## `chloros-cli language`

CLI의 표시 언어를 설정합니다(CJK, RTL, 인도어 등 38개 언어 지원). 스크립트를 렌더링할 수 없는 구형 콘솔에서는 영어로 원활하게 대체됩니다.

```
chloros-cli language [LANG_CODE] [--list]
```

### 예시

```bash
# List all available languages
chloros-cli language --list

# Switch to Spanish
chloros-cli language es

# Show the currently-active language
chloros-cli language
```

---

## 프로젝트 폴더 명령어

이 명령어들은 기본 프로젝트 폴더 위치를 관리합니다(GUI와 공유됨).

```bash
chloros-cli set-project-folder "/home/user/Chloros Projects"
chloros-cli get-project-folder
chloros-cli reset-project-folder
```

`get-project-folder`는 새 프로젝트가 저장되는 폴더를 출력합니다. 설정된 폴더가 더 이상 존재하지 않는 경우(삭제되었거나, 이름이 변경되었거나, 연결되지 않은 드라이브에 있는 경우), CLI는 GUI와 마찬가지로 기본 폴더로 전환됩니다: 기본 폴더가 stdout에 출력되고, 누락된 폴더 이름을 명시한 경고 메시지가 stderr로 전송되며, 종료 코드는 `0`가 됩니다.

---

## `chloros-cli update`

Linux/Jetson 전용입니다. `version_url`에서 `/etc/chloros/update.conf`를 확인하고, 더 새로운 버전이 존재할 경우, 해당 `.deb`를 다운로드하여 `sudo dpkg -i`와 함께 즉시 설치합니다(확인 메시지는 표시되지 않음; `sudo`에서는 비밀번호 입력을 요청할 수 있음). Jetson에서는 JetPack 6 시스템에만 업데이트가 제공됩니다. Windows에서는 이 명령어가 아무런 동작도 하지 않으며, `up to date`: 새 설치 프로그램을 사용하여 업데이트하라는 메시지를 표시합니다.

```bash
chloros-cli update            # check + install
chloros-cli update --check    # check only
```

Linux/Jetson에서 CLI는 **매번 시작 시 자동 업데이트 확인** (비차단 방식으로, 명령어 실행을 절대 지연시키지 않음): `/etc/chloros/update.conf`를 읽은 후, 결과를 1시간 동안 `~/.chloros/update_cache.json`에 캐시하고, 더 새로운 버전이 존재할 경우 `Update available: vX.Y.Z / Run: chloros-cli update`를 출력합니다. 오류가 발생하거나 Windows에서는 아무런 메시지 없이 건너뛰며, `CHLOROS_DISABLE_UPDATE_CHECK=1`를 설정하면 이 기능이 비활성화됩니다. 1시간 캐시 기능으로 인해, 해당 시간 내에 출시된 릴리스도 여전히 최신 버전으로 표시될 수 있습니다.

---

## `chloros-cli selftest`

7단계 스모크 테스트를 실행합니다: 버전, 포트 가용성, 백엔드 시작, `/api/test`, `/api/system-info` (GPU/CUDA/PyTorch), 노이즈 제거기 모델 존재 여부, CUDA+노이즈 제거기 준비 상태. 이 테스트는 실행 중인 백엔드를 사용하거나, 테스트를 위해 백엔드를 시작한 후 테스트가 끝나면 중지합니다. `N/7 checks passed`와 실패한 검사 항목을 출력하며, 7가지 검사 항목이 모두 통과된 경우에만 `0`를 종료합니다; CUDA GPU가 없는 컴퓨터에서는 CUDA+노이즈 제거기 검사가 실패합니다([설치 확인](#verify-the-install) 참조).

```bash
chloros-cli selftest
```

---

## `chloros-cli time-sync`

PTP 그랜드마스터 상태 및 제어. Chloros 호스트가 PTP 그랜드마스터를 실행하며, LATTICE 카메라와 DAQ-E 장치는 장치 간 타임스탬프를 위해 이에 종속됩니다 (장치 간 약 1 ms)를 위해 이 그랜드마스터에 종속됩니다. PTP는 **타임스탬프**를 동기화합니다: 어레이의 카메라들이 동시에 노출되는 것은 PTP 때문이 아니라, M8 하드웨어 트리거가 카메라를 작동시키기 때문입니다(모듈 간 100 µs 미만). 하위 명령어가 없는 `chloros-cli time-sync`는 `status`를 실행합니다.

| 하위 명령어 | 설명 |
| --- | --- |
| `status` | 그랜드마스터 상태, BMCA 우선순위, 클럭 ID 및 패킷이 네트워크에 도달하고 있는지 여부를 표시합니다: `health`는 `OK` (그랜드마스터가 실행 중이지 않을 때는 `STOPPED`), 또는 `SENDS_FAILING` / `DEGRADED` / `NO_SOCKETS` / `PEERS_SILENT`와 함께 `health_detail`가 표시되고, 그 후 stderr에 `WARNING:` 행이 출력됩니다(stdout은 JSON 형식을 유지함). `sync_sent` / `announce_sent`는 전송된 메시지만 집계하고, `*_send_errors`와 `last_send_error`는 전송되지 않은 메시지를 집계합니다. 카메라 링크가 끊어졌다가 다시 연결되면, 그랜드마스터가 자체적으로 소켓을 재구축합니다(`rebinds`, `last_rebind`). |
| `peers` | Delay_Req를 통해 감지된 슬레이브 목록(카메라 + DAQ-E 센서). |
| `cameras` | 카메라별 PTP 상태(`PtpStatus`, `PtpOffsetFromMaster`, `PtpMeanPathDelay`). |
| `restart` | 그랜드마스터 프로세스 재시작. |
| `set-priority --priority1 N --priority2 N` | BMCA 우선순위 재정의. |

그랜드마스터는 카메라 NIC를 통해 알림을 전송합니다 (`169.254.x.x` 주소가 포함된 모든 어댑터)를 통해 알림을 전송합니다. 활성화된 어댑터가 없을 경우, 기본 경로의 어댑터로만 대체되며, 이는 유선 연결일 때에만 해당됩니다. Wi-Fi나 가상 어댑터(Tailscale, WSL)는 절대 사용되지 않습니다. 카메라나 DAQ-E가 그 뒤에 위치하지 않기 때문입니다. 알릴 대상이 없으면 소켓 없이(`health` `NO_SOCKETS`) 실행되며, 다음 카메라나 DAQ가 연결되면 그때까지 활성화된 NIC에서 소켓을 재구축합니다; `time-sync restart`는 이를 즉시 수행합니다.

그랜드마스터는 백엔드와 별도로 자체 프로세스에서 실행됩니다(`own_process` / `process_pid` in `time-sync status`)와 별도로 자체 프로세스에서 실행되며, Windows에서는 높은 우선순위(`process_priority`)로 실행되므로, 바쁜 백엔드(처리 실행, 내보내기, 동시에 실행되는 여러 명령어)도 타임스탬프 기록 시점을 지연시킬 수 없습니다. 이전에는 이러한 지연이 발생하면 모든 카메라의 시계가 한꺼번에 이동할 수 있었습니다. 이 프로세스는 백엔드에서 종료되며, 백엔드를 시작할 수 없는 경우 그랜드마스터는 이전과 마찬가지로 백엔드 내에서 실행됩니다(`own_process` false, 백엔드 로그에 해당 내용이 기록됨).

### 예시

```bash
chloros-cli time-sync status
chloros-cli time-sync peers
chloros-cli time-sync cameras
chloros-cli time-sync restart
chloros-cli time-sync set-priority --priority1 1 --priority2 1
```

---

## `chloros-cli lattice`

LATTICE 카메라 제어. 모든 하위 명령은 Chloros 백엔드를 통해 전달되며, 백엔드가 카메라 풀을 관리하므로 후속 CLI 호출에서는 동일한 열린 핸들을 재사용합니다.

### 일반 옵션 (대부분의 하위 명령에서 공통적으로 사용)

| 플래그 | 설명 |
| --- | --- |
| `-d, --device N` | 카메라 인덱스(기본값: 0). |
| `-s, --serial SN` | 특정 시리얼 번호; `--device`를 재정의합니다. |
| `--serials SN1,SN2,…` | 다중 카메라 작동을 위한 쉼표로 구분된 일련 번호. [일시적인 다중 카메라 명령어](#multi-camera-transient-sessions)는 이 카메라들만 열고, 첫 번째 카메라가 트리거를 작동시킵니다. |
| `--all` | 탐지된 모든 카메라에 적용합니다. |
| `--exposure US` | 노출 시간(마이크로초 단위). |
| `--gain DB` | 게인(dB 단위). |
| `--pixel-format FMT` | 예: `BayerRG8`, `BayerRG12`; `Mono8`, M3M에서는 `Mono12`. 이 이름은 카메라 자체 제품군 내에서 비트 심도를 설정합니다: M3M에서는 베이어(Bayer) 이름이 해당 깊이 형식의 모노(Mono) 포맷을 설정하며(`BayerRG8`는 `Mono8`를 설정), M3C에서는 모노(Mono) 이름이 베이어(Bayer) 이름을 설정합니다 (`Mono12`는 `BayerRG12`를 설정합니다). |
| `--width N` / `--height N` | 이미지 크기. |
| `--preset {default,high_quality,high_speed,triggered}` | 설정 프리셋 적용: 픽셀 형식, 노출, 게인 및 프레임 속도 (`high_speed`: `BayerRG8`, 5 ms, 6 dB, 60 fps; `high_quality`: `BayerRG12`, 20 ms, 0 dB; `lattice presets`에 나열됨). M3M에서는 동일한 깊이의 모노 형식을 사용합니다: `high_speed`는 `Mono8`, `high_quality` 및 `triggered`, `Mono12`를 캡처합니다. 이와 함께 지정된 플래그 (`--exposure`, `--gain`, `--pixel-format`)는 사전 설정보다 우선하며, 사전 설정에 명시되지 않은 설정은 카메라의 기본값을 유지합니다. 모든 free-런 모드이지만, `triggered`는 예외로, 2번 라인의 하드웨어 에지 신호에 따라 카메라를 작동시킵니다. 해당 라인에 신호가 없으면 촬영 대신 무한정 대기합니다. |
| `-o, --output DIR` | 출력 디렉터리 (기본값: `output`). |
| `--packet-size {auto,jumbo,standard,N}` | GVSP 패킷 크기. `auto`는 ICMP+GVSP 프로브를 실행합니다; `jumbo` = 9000; `standard` = 1500. |

### LATTICE 카메라 첫 연결 워크플로우

```bash
# 0. A backend must be running: the Chloros desktop app (Windows),
#    `chloros-backend`, or the Linux service (see "Start the Backend")

# 1. Discover cameras on the network
chloros-cli lattice info

# 2. Single-cam smoke test: capture one frame.
#    By default this saves EVERY export type applicable to the cam
#    (raw, debayered, radiance, reflectance, preview). Pass e.g.
#    `--processing debayered` to save just one.
chloros-cli lattice capture -o output/

# 3. Connect a synchronized array (RECOMMENDED ENTRY POINT for arrays).
#    This is the same "smart-prep" flow the Chloros GUI uses:
#      - Network capability probe (ICMP DF ping + GVSP probe)
#      - Tier auto-pick (sim-emit / ftd-stagger / slip)
#      - Full sensor (no binning, no ROI); the frame rate falls to
#        what the host and wire deliver, and the faster options are
#        printed
#      - PTP enabled by default
#      - Per-cam pixel format auto-pick
#      - AE seeding from the cam's saved state
#      - GPIO trigger config on Line2
chloros-cli lattice array-connect \
  --serials 213800234,214000533,214701288,214701292

# 4. Capture one synced frame group from the live array.
#    Defaults to --processing all (one file per export type per cam);
#    pass a single level to narrow it, e.g. --processing reflectance.
chloros-cli lattice array-capture --processing reflectance -o output/

# 5. Live-preview one cam in your browser
chloros-cli lattice viewer --serial 213800234

# 6. Tear down when done
chloros-cli lattice array-disconnect
```

### 하위 명령어 참조

#### 탐색 및 정보

| 하위 명령어 | 용도 |
| --- | --- |
| `lattice info` | 연결된 카메라 목록 표시 (제조사, 모델, 일련번호, IP, MAC). |
| `lattice info` | 최적의 카메라 구성을 위해 호스트 시스템을 분석합니다. `--no-discover`는 카메라 탐색 단계를 건너뜁니다(더 빠르며, NIC만 분석). |
| `lattice network [--fix] [--estimate] [--cameras N] [--pixel-format FMT]` | NIC 설정 확인/수정 (Linux의 경우 커널 소켓 수신 버퍼도 포함 — [Linux 호스트](#linux-hosts--socket-receive-buffers) 참조); 대역폭/FPS 추정. |
| `lattice network-analysis --master SN [--slaves SN1,SN2,…] [--width N] [--height N] [--pixel-format FMT] [--binning {1,2,4}] [--force-tier TIER] [--packed {auto,on,off}] [--backend-url URL] [--json]` | Stable-schema 백엔드 네트워크-기능 + 어레이 권장 사항 (`status` ∈ `ok` / `auto_shrunk` / `auto_capped_fps` / `needs_force_slip` / `error`). `--width` / `--height`는 전체 센서로 기본 설정되며, `2048` × `1536`. `0`는 `ok`, `auto_capped_fps` 및 `auto_shrunk`를 거쳐, `1`는 `needs_force_slip` 및 `error`로 이어지며, `--json`의 유무와 관계없이 적용됩니다. 이 설정은 `--backend-url`(기본값 `http://127.0.0.1:5000`)에서 백엔드에 도달하며, `CHLOROS_BACKEND_URL`에서는 도달하지 않습니다. `auto_capped_fps`는 요청된 해상도를 유지하되 대상 fps를 제한합니다. `recommended.recommended_target_fps`를 읽어 연결 대상으로 전달하십시오. 이를 성공으로 간주하고, 오류로 간주하지 마십시오. `auto_shrunk`는 요청된 프레임을 어떤 프레임 속도로도 전달할 수 없는 경우에만 나타납니다. connect는 자체적으로 프레임 크기를 축소하지 않습니다. |
| `lattice analyze-array [--models M1,M2,…] [--binning {1,2,4}] [--n-active N] [--width N] [--height N] [--pixel-format FMT] [--force-tier TIER] [--packed {auto,on,off}] [--json]` | 카메라를 열지 않은 상태에서의 가설 분석 (`--width` / `--height` 기본값 `2048` × `1536`). 이 항목과 `network-analysis` 모두 connect가 스트리밍할 내용을 예측합니다: `--pixel-format`가 없는 경우, connect가 적용하는 카메라별 형식(Mono12 / BayerRG12)을 사용하며, `--packed auto`(기본값)의 경우 connect가 이 컴퓨터에서 내릴 [압축된 12비트](#packed-12-bit-transport---packed) 형식이 적용됩니다. 따라서 명시된 fps는 어레이가 실행되는 속도입니다. 두 경우 모두 패킹 방식을 출력합니다. **`--n-active`는 이 어레이뿐만 아니라 네트워크에 연결된 전체 카메라 수를 나타냅니다**— 독립형 카메라가 동시에 스트리밍되거나, 케이블 대역폭 예산이 카메라 수를 과소 산정한 수요를 기준으로 계산될 때 이 값을 늘리십시오(기본값: `len(--models)`). 항상 집계된 `Wire budget:`(요구된 MB/s 대 충돌 방지 상한) 및 `Max cameras:` 행을 출력하며, 어레이가 와이어를 초과 할당할 경우 `** OVER-SUBSCRIBED**`를 표시합니다 — [어레이 fps 및 버스트 모델](#array-fps--burst-model)을 참조하십시오. |
| `lattice gpu` | GPU 상태 표시. |
| `lattice firmware [--update] [--force] [-y\|--yes]` | 카메라 펌웨어 확인 또는 업데이트. 로컬 `.fwa` 선택 항목은 고정되어 있습니다: 빌드의 `MIN_FIRMWARE_VERSION`와 일치하는 `firmware/<MODEL_PREFIX>/` 내의 파일이 존재할 경우 플래시됩니다 (후비용으로 가장 높은 버전만), 따라서 디스크에 준비된 최신 벤더 이미지는 해당 핀이 변경될 때까지 비활성 상태입니다 — 의도적으로 최신 릴리스는 서명된 AWS 매니페스트를 통해 장치에 전달되며, 최신 버전이 있을 경우 이 방법이 선호됩니다. |
| `lattice presets [--apply NAME]` | 카메라 프리셋을 나열하거나 적용합니다. `--apply`는 프리셋의 구성 요소(픽셀 형식, 노출, 게인, 프레임 속도, 트리거)를 설정하고, 카메라가 이를 읽은 후 해당 설정을 출력합니다: M3M의 경우, 프리셋의 깊이 값에 해당하는 모노(Mono) 형식; 카메라가 실제로 실행하는 프레임 속도(이는 프리셋에서 요청한 값보다 낮을 수 있음. 예를 들어, `high_speed`는 60 fps를 요청하지만, 1 Gbps 링크에 연결된 풀프레임 카메라는 약 35 fps로 실행됨). open이 기록한 그 외 모든 설정은 발견된 그대로 복원됩니다. `--apply default`는 카메라를 공장 기본값으로 재설정합니다: Line2에서 자유 실행 자동 노출이 활성화된 공장 설정, 링크 처리량 제한 및 결함 보정 비활성화, RGB 카메라의 보정된 화이트 밸런스 유지(그 외 모든 경우 기본값). 카메라 메모리에는 아무것도 저장되지 않으므로, 다음에 전원을 켤 때 이전과 마찬가지로 자체 메모리에서 시작합니다. 캡처 또는 연결 시 `--preset`, `default`는 사전 설정으로 사용되지 않습니다. |
| `lattice status` | 실시간 카메라 상태. |

#### 캡처

| 하위 명령 | 용도 |
| --- | --- |
| `lattice capture [--format tiff\|tif\|png\|jpg\|jpeg] [--jpeg-quality N] [--processing LEVEL] [--levels L1,L2,…] [--no-daq] [--daq-tolerance-ms MS]` | 단일 프레임. **기본적으로 모든 내보내기 유형을 저장합니다**(`--processing all`); [캡처 내보내기 수준](#capture-export-levels-the-all-default)을 참조하십시오. `--processing`는 `all`, `raw`, `debayered`, `radiance`, `reflectance` 또는 `display`를 입력받습니다(`preview`는 `display`의 경우 허용됨); 카메라를 터치하기 전에는 다른 이름은 거부됩니다(`1` 종료). `--levels`는 명시적인 하위 집합을 저장합니다(`--processing`를 재정의함). Chloros가 보유하지 않은 카메라가 캡처용으로 열리고, [일시적 명령어](#multi-camera-transient-sessions)에서와 마찬가지로 먼저 장면에서 노출이 설정됩니다(최대 몇 초); 캡처가 종료되면, 종료 방식에 관계없이 해당 카메라는 발견된 상태(포맷, 노출, 색상 파이프라인, 캡처로 인해 변경된 모든 설정 — [다중 카메라(일시적 세션)](#multi-camera-transient-sessions) 아래의 목록 참조)로 복원됩니다. 사용된 캡처를 읽은 DAQ 데이터는 `.daq` 사이드카 파일로 저장되며, 여기에는 원시 데이터만 포함된 캡처 데이터가 포함됩니다. `--no-daq` / `--daq-tolerance-ms`를 사용하여 이를 조정할 수 있습니다 ([캡처를 읽는 DAQ가 저장하는 내용](#the-daq-reading-a-capture-saves)). `--force-daq`도 여전히 허용됩니다. `--jpeg-quality` = JPEG 품질 1–100 (기본값 95).**열려 있는 프로젝트에 저장된 파일은 해당 프로젝트에 병합됩니다.** 앱과 마찬가지로, Chloros에서 열려 있는 프로젝트의 폴더 내에 파일이 저장된 캡처는 해당 프로젝트의 파일 목록에 추가되며, 다른 곳에 저장된 파일은 그대로 유지됩니다. 이 명령어는 이를 알리는 한 줄의 메시지를 출력합니다 (`Project: ...`), 또는 처리 작업이 진행 중인 경우에는 파일이 보류되었다가 작업이 종료되면 프로젝트에 추가된다는 내용을 한 줄로 출력합니다. `multi-capture`, `array-capture` (`--smart`도 마찬가지) 및 `project capture`도 마찬가지입니다. |
| `lattice continuous [--format tiff\|tif\|png\|jpg\|jpeg] [--jpeg-quality N] [--queue-depth N] [--no-daq] [--daq-tolerance-ms MS]` | Ctrl+C를 누를 때까지 디스크로 스트리밍합니다. `tiff` 실행의 프레임은 `raw` 파일로, 해당 카메라의 캡처 데이터 배열입니다: 센서의 모자이크, 각 평면이 고유한 값을 가지며, 압축되지 않은 상태로, 보정 패킷이 포함되어 있습니다 (2048 x 1536 12비트 프레임은 장면에 관계없이 6.0 MiB에 패킷이 추가된 크기)가 포함되어 있으므로, `process`는 이를 기반으로 시작합니다. `png`와 `jpg`는 8비트를 포함하므로, 해당 런은 보기를 위한 디모자이징된 프레임이며, `process`는 이를 기반으로 하지 않습니다. DAQ에 바인딩된 (`daq pool-bind`) 또는 풀링 처리된 경우, 각 프레임은 런의 `daq/` 폴더 내에 고유한 `.daq`와 `.match.json`를 갖게 되며, `lattice capture`는 하나를 저장하고, `--no-daq`는 이를 비활성화하며, 중지 시 저장된 프레임 수를 출력합니다. 각 프레임은 전체 메타데이터와 함께 한 번씩 기록되며; Ctrl+C를 누르면 카메라가 해제되기 전에 이미 캡처된 프레임들이 기록됩니다. 드라이브가 가득 차면 자동으로 중지되며, `Failed:` / `Disk full:`를 출력하고 종료되며, `1`; 기록하지 못한 프레임이 있는 경우에도 `1`로 종료됩니다. 하드웨어 트리거를 위해 준비 상태로 남겨진 카메라 (`TriggerMode` `On`)로 설정된 카메라는 해당 실행 동안 프리런 모드로 설정됩니다. 실행이 종료되면 카메라는 발견된 상태 그대로 복원됩니다 — 트리거 설정 및 실행 중에 변경된 모든 설정 (노출, 게인, 포맷 등) —는 모두 원래 상태로 복원됩니다. 단, 실행 설정을 카메라 자체 메모리에 저장하는 `--keep-settings`(자유 실행) 또는 해당 메모리를 공장 출하 시 설정으로 재설정하는 `--clear-settings`의 경우는 예외입니다. 프레임을 기록하지 못한 실행은 실패로 간주됩니다. 10초 동안 프레임이 없으면 자동으로 중지되고, 이유를 포함한 `No frames:`를 출력한 후 `1`로 종료됩니다. 라이터가 따라잡지 못한 프레임은 누락된 것으로 간주되며, 중단 시 이유(카메라의 프레임 속도, 라이터의 프레임 속도 및 큐)와 함께 `Dropped:`를 출력합니다. 프레임 속도가 더 낮거나 8비트 픽셀 형식인 경우에는 모든 프레임이 유지됩니다. 이 명령은 2초마다 백엔드에 상태를 확인합니다. 녹화를 중지하지 않은 상태에서 종료될 경우(`kill -9`, 메모리 부족), 백엔드는 마지막 상태 확인 후 30초가 지나면 녹화를 중지하고 카메라를 원래 상태로 되돌립니다 (`stop_reason` `client_gone`, 백엔드 로그에 한 줄 기록됨). `multi-stream` 및 `align-stream`도 동일하게 동작합니다. 카메라가 열리는 데 명령어가 허용하는 대기 시간(60초; `multi-stream` / `align-stream`의 경우 180초)보다 더 오래 걸리는 시작(카메라가 여전히 부팅 중인 경우)은 백엔드에 의해 취소됩니다: 카메라는 원래 상태로 되돌려지고 닫히며, 진행 중인 녹화는 남지 않습니다(HTTP 504 `CLIENT_DEADLINE_PASSED`). |
| `lattice viewer [--brightness N] [--ae-damping F] [--frame-rate FPS]` | 브라우저 기반 실시간 MJPEG 미리보기: 카메라에 연결하고, 스트림 URL을 표시한 후 브라우저에서 해당 URL을 엽니다. `--ae-damping`는 자동 노출 감쇠(0.4–100)를 설정합니다. |

##### 실시간 스트림 (MJPEG; RTSP 미지원)

Chloros는 백엔드에서 **HTTP를 통한 MJPEG**(`multipart/x-mixed-replace`, 경계 `frame`)로 전송합니다.**RTSP는 지원되지 않습니다.** 브라우저에서 URL을 열거나, MJPEG 클라이언트(OpenCV의 `cv2.VideoCapture(url)`, ffmpeg, VLC)를 사용하여 스트림을 읽어보세요. 먼저 백엔드에서 카메라를 연결해야 합니다(`lattice cam-connect`, `lattice array-connect`, `lattice viewer` 또는 GUI).

| URL (`http://127.0.0.1:5000` + …) | 스트림 |
| --- | --- |
| `/api/camera/stream/<serial>` | 디스플레이 미리보기, 최대 약 30 fps. 배열 요소의 경우, `?max_h=`(기본값: `720`)와 `?jpeg_q=`(기본값: `80`)는 높이와 JPEG 품질을 설정합니다. 이것은 `lattice viewer`가 여는 URL입니다. |
| `/api/camera/<serial>/stream-annotated` | GUI의 오버레이가 그려진 동일한 미리보기: `zebra=1`, `crosshair=1`, `grid=3x3` (또는 `9x9`), `histogram=1`, `peaking=1`, `spot=1`; 별도의 확인 요청 없이, 해당 카메라에 대해 프로젝트에 저장된 오버레이 설정이 적용됩니다. |
| `/api/camera/<serial>/stream-reflectance` | 대역별 8비트 반사율: ρ = 픽셀 ÷ 해당`X-Chloros-Pixel-Scale` 헤더(`refl_max`가 발생하지 않는 한 255). `fps`(기본값 `10`), `jpeg_q`(기본값: `95`), `max_h`(기본값: `0` = 네이티브), `band=<name>`, `align=1`. JPEG이므로 손실 압축 방식입니다. 카메라에 반사율이 없을 경우(RGB 카메라, 보정 팩 없음, DAQ 판독값 없음) HTTP 409 오류로 거부됩니다. |

- **카메라 간 프레임 매칭.**세 스트림 모두에서**배열 구성원**이 전송하는 모든 파트에는 `X-Chloros-Group-Id`(배열의 동기화 그룹 카운터), `X-Chloros-Frame-Id` 및 `X-Chloros-Timestamp-Ns` 헤더를 포함합니다. 서로 다른 카메라에서 전송된 파트 중 동일한 `X-Chloros-Group-Id`를 가진 것은 동일한 트리거에서 비롯된 것입니다. 독립형 카메라는 그룹이 없으며 그룹 ID를 전송하지 않습니다. 브라우저는 파트별 헤더를 무시하며, 멀티파트 스트림을 직접 파싱하는 클라이언트가 이를 읽습니다.
- **어레이 뷰가 열려 있는 동안 일시 중지됨.** 어레이 구성원의 `/api/camera/stream/<serial>`는 어레이의 통합 라이브 스트림(예: GUI의 어레이 뷰)이 열려 있는 동안 정적 "일시 중지" 프레임을 전송하며, 스트림이 닫히면 다시 재개됩니다.

#### 센서 튜닝

| 하위 명령 | 목적 |
| --- | --- |
| `lattice configure [{--get N1 N2… \| --set N=V N=V… \| --dump}] [--json]` | 모든 GenICam 노드를 읽거나 쓰기. `--set ExposureTime=…` (`Gain=…`)는 `ExposureAuto` (`GainAuto`) 스위치 없이 먼저 자동 모드를 끄고, `lattice exposure --set`가 수행하는 것과 달리; 카메라가 거부하는 값을 입력하면 stderr에 `Failed to set <node>: …`를 출력하고 `1`를 종료합니다. `--get` 및 기본 그룹화된 뷰는 해당 자동 모드가 실행되는 동안 `note:` 아래에 `ExposureTime` / `Gain`를 출력합니다 — [자동 모드에서 노출 및 게인 읽기](#reading-exposure-and-gain-under-auto-mode)를 참조하십시오. |
| `lattice exposure [{--auto \| --auto-once \| --off \| --set US}] [--brightness 0-255] [--brightness-auto] [--damping 0.4-100] [--upper-limit US]` | 노출 및 AE. `--set` 노출값이나 `--brightness` 목표값이 사용자의 설정으로 유지됩니다 (클로로스(Chloros)의 자동 노출 제어 기능은 이를 그대로 둡니다); 게인은 기존 상태를 유지하므로, 자동 모드에서는 사용자가 `lattice gain`를 설정할 때까지 목표값을 향해 계속 조정됩니다. `--auto`는 노출을 다시 제어하고, `--brightness-auto`는 타깃을 Chloros의 자동 타깃으로 되돌립니다. 플래그가 없으면 노출 노드를 읽으며, `note:`와 `configure --get`가 동일하게 적용됩니다. 카메라가 거부하는 `--set` 값의 경우, stderr에 `Failed to set ExposureTime: ExposureTime=<value> refused (<camera error>); the camera accepts <min>..<max> now (ExposureAuto put back to <mode>)`를 출력하고 — 쓰기 기능이 꺼져 있던 자동 모드는 ‘Off’ 상태로 남지 않고 복원되며 — `Exposure: … us (auto OFF)` 줄은 출력되지 않고, `1`에서 종료됩니다. |
| `lattice gain [{--auto \| --off \| --set DB}]` | 게인 및 자동 게인. 카메라가 허용하지 않는 `--set` 값인 경우, stderr에 `Failed to set Gain: …`를 출력하고, `Gain: … dB (auto OFF)` 줄은 출력하지 않은 채 `1`로 종료됩니다. |
| `lattice resolution [--set WxH] [--offset X,Y] [--binning N]` | 센서 ROI 및 비닝. 비닝은 항상 **평균**입니다 — 아래 참조. |
| `lattice format [--set FMT] [--list]` | 픽셀 형식. |
| `lattice trigger [--mode On\|Off] [--source SRC] [--delay-us US] [--activation EDGE] [--list-sources] [--software]` | 하드웨어/소프트웨어 트리거. |
| `lattice white-balance [--auto] [--off] [--red R] [--blue B]` (플래그 없음 = 원샷 화이트 밸런스) | 화이트 밸런스 작업. RGB/Bayer 카메라에만 해당; 모노 M3M에서는 무작동(건너뜀). 원샷은 계산된 화이트 밸런스(`BalanceWhiteAuto` 및 게인)를 유지하며, 오픈 쓰기 작업으로 설정된 나머지 모든 값은 원래 상태로 복원되며, 원샷이 실행될 수 없는 경우에는 모든 값이 복원됩니다. |
| `lattice color-profile [--set raw\|linear\|natural\|enhanced\|custom_temp] [--cct 2000-10000] [--get]` | RGB 디스플레이 색상 파이프라인. `natural`(기본값)는 저사양 라이브 마무리 처리 방식이며; `enhanced`는 디프린지 제거 + 비브런스 + CLAHE 국소 대비를 추가하여 완전한 허브 패리티 룩을 구현하며, 프레임당 처리 비용이 약 2배 증가하므로 **실시간** 프레임 속도가 낮아집니다. 단, 저장된 캡처 영상은 어떤 경우든 항상 완전한 처리가 적용됩니다. RGB/Bayer 카메라 전용; 모노 M3M에서는 건너뜁니다. |
| `lattice color [--saturation 0-200] [--contrast 0-200] [--reset] [--get]` | 라이브 미리보기 채도/명암비 (RGB 필터 카메라); `100`는 중립이며, `--reset`는 두 값을 모두 100으로 설정합니다. 저장된 캡처에는 영향을 미치지 않습니다. 모노 M3M에서는 건너뜁니다. |
| `lattice highlights [{--set -100..0 \| --reset \| --get}]` (플래그 없음 = 읽기) | **하이라이트** 표시 (RGB 필터 카메라): 가장 밝고 흰 픽셀의 명도를 낮춤 — 예: 평평한 흰색 얼룩으로 렌더링되는 밝은 흰색 표면. `0` = 꺼짐, `-100` = 가장 강함. 라이브 미리보기와 디스플레이 수준의 캡처는 이 설정을 따르지만, 방사도/반사도는 절대 따르지 않으며, 클리핑된 센서 데이터는 복구할 수 없습니다(이 경우 노출을 낮추십시오). 모노 M3M에서는 건너뜁니다. |
| `lattice filter [--list]` | Chloros가 인식하는 필터 모델 목록을 표시합니다(`RGN-IMX265`, `OCN`, `NGB`, …). 카메라의 필터는 모델 문자열에서 읽어오며 설정할 수 없습니다: `--set NAME`는 모델만 조회한 후, `1`를 통해 `setting the filter model is not supported`로 종료됩니다. |
| `lattice power [--sleep]` | 전원/열 노드 상태 확인; 저전력 대기 모드 전환. `--sleep`는 카메라를 절전 모드로 전환합니다(링크 처리량 제한이 활성화됨(`DeviceLinkThroughputLimitMode` `On`) 최저값으로 설정되며, 카메라에 절전 모드가 있는 경우 해당 모드가 켜짐) 그 외 모든 설정은 처음 발견된 대로 복원되며; 수면 모드로 전환하는 데 실패한 경우 전체 설정이 복원됩니다. 깨우기 시 링크는 수면 모드에서 발견된 상태(제한값, 모드, 패킷 지연 `GevSCPD`—카메라는 제한값이 활성화된 동안 이를 0으로 유지함; 이 기록은 이 컴퓨터에 저장되므로 따라서 백엔드가 재시작된 후 깨어날 때도 이를 확인할 수 있음), 또는 이 컴퓨터가 절전 모드로 전환하지 않았으나 절전 상태로 보이는 카메라(한도가 최저값으로 설정된 경우)의 경우, 한도를 해제하고 `GevSCPD`를 80 (공장 설정); 하한값보다 높게 설정된 제한은 그대로 유지됩니다. 프로브와 웨이크는 오픈 시 기록한 값을 복원합니다. |

> **비닝은 항상 평균(Average)입니다.** `lattice resolution --binning N`는 평균 비닝을 설정하며
> 모드 전환 기능을 제공하지 않습니다. 합산(Sum) 비닝은 각 DN 값에 N²을 곱하므로, 비닝 단계당 대략 2스톱의
> 하이라이트 헤드룸이 소모됩니다. Chloros는 방사계측 체인 전에
> 해당 계수를 다시 나눕니다만, 합산 비닝으로 인해 센서의 상한값까지 도달한 픽셀은 클리핑되어
> 복구할 수 없습니다. 이는 ‘평균’ 방식이 제공하는 이점 외에 추가적인 이점을 주지 않습니다. 지수는 스케일 불변이며, 복사도
> 스칼라 값은 ‘평균’ 스케일로 보정되어 있습니다. 카메라가 이미 ‘합산’ 모드로 설정된 상태(다른
> 도구로 설정되었거나 공장 초기화되었을 경우)로 도착하면, Chloros는 노드 리드백을 통해 이를 감지하고
> 보정합니다. `lattice resolution --binning N`를 실행하면 다시 Average로 설정됩니다. 1.2.0
> `--binning-mode` 플래그는 여전히 허용되므로 구버전 스크립트도 실행되지만, 경고만 출력됩니다:
> 비닝은 ‘평균’으로 유지됩니다.

> **카메라에서 비닝, 픽셀 형식 및 트리거를 변경해도 아무것도 적용되지 않습니다.** 카메라에서
> 백엔드가 열려 있는 상태로 스트리밍을 수행하는 경우(`cam-connect`, GUI), `lattice resolution --binning`
> 변경 사항이 적용되지 않습니다(여전히 적용 중인 비닝이 출력됨, `Binning: 1x1`), 그리고
> `lattice format --set` 및 `lattice trigger --mode`는 카메라에서 거부됩니다
> (`SC_ERR_ERROR -1001`). 먼저 해제하십시오(`lattice cam-disconnect -s <SN>`), 또는
> 연결 시에 설정하십시오(`cam-connect --pixel-format …`, `array-connect --binning …`).

##### 자동 모드에서 노출 및 게인 읽기

카메라의 `ExposureAuto` (`GainAuto`)가 실행 중일 때, 해당 카메라의 `ExposureTime` (`Gain`) 노드는 카메라의 현재 동작 상태가 아닌 **마지막으로 기록된 값**을 유지합니다. 백엔드가 열어 둔 카메라(`cam-connect`, `array-connect`, GUI), `lattice configure --get`, 기본 뷰인 `lattice configure` 및 `lattice exposure`(플래그 없음)는 노드 값을 출력하고, 그 아래에 카메라의 최신 프레임 정보를 포함한 메모를 표시합니다:

```

ExposureTime = 2000.448
  note: ExposureAuto is Continuous: ExposureTime holds the last value written (2000.448); the camera's newest frame was at 4900 us.
```

메모에서 프레임 값을 읽어오십시오. 노드 값은 마지막에 기록된 값일 뿐입니다. HTTP를 통해 동일한 `POST /api/camera/<serial>/read-nodes` 응답은 `nodes`를 변경하지 않은 채로 유지하고, `frame_values` (`{"ExposureTime": <us>, "Gain": <dB>}`, 최신 프레임)과 `notes`(위의 문장)를 추가합니다. 이 둘은 자동 모드가 `Off`가 아니고 프레임이 확인된 경우에만 존재합니다; 읽기 전용으로 열린 카메라에는 프레임이 없으므로 둘 다 표시되지 않습니다. `lattice gain`(플래그 없음)는 노드만 출력합니다 — 메모를 보려면 `lattice configure --get Gain`를 사용하십시오.

##### 어레이 구성 요소의 노출

어레이가 스트리밍되는 동안, 각 구성 요소는 **`AcquisitionFrameRateEnable`**로 실행되며, 프레임 속도는 배열의 트리거 속도보다 약간 높은 수준입니다. 주기는 트리거 주기에서 15%를 뺀 값(최소 3ms)이거나, 트리거 속도가 이미 최대 속도에 근접한 경우 카메라 자체의 최대 속도입니다. 따라서 구성 요소의**`configure --get`**는 해당 설정에서**`AcquisitionFrameRateEnable`**를**`True`**및**`AcquisitionFrameRate`**로 읽습니다. 어레이가 정지하면 비활성화 상태가 다시 해제됩니다.**어레이 멤버에 이 두 노드를 기록하지 마십시오** — 어레이는 이 노드들을 트리거와 동기화하여 유지하고 재기록합니다.

이 설정은 또한 멤버의 노출 상한선이기도 합니다: 10 fps 트리거(100 ms 주기)에서 **어떤 ROI나 비닝**을 사용하더라도 약 85 ms입니다. 1.3.0 이전 버전에서는 작은 ROI를 설정할 경우, 어레이의 트리거 속도가 아무리 느리더라도 구성원의 노출 시간이 센서의 프리런 주기(768 × 304 기준 4.9 ms)로 제한되었습니다.**상한값보다 긴 수동 `ExposureTime` 값은, `lattice exposure --set` 또는 `configure --set`(혹은 GUI를 통해)로 기록될 때** **상한값에 맞추기 위해 트리거 속도를 늦춥니다**. 어레이는 먼저 해당 멤버의 상한값을 열고 더 느린 속도로 몇 번의 트리거 주기를 실행하므로, 쓰기 작업에 몇 초 더 걸릴 수 있습니다. 그 후 트리거는 노출이 허용하는 범위 내에서만 작동합니다. 자동 속도 어레이의 경우, 수동 노출로 인해 속도가 느려졌던 모든 멤버가 다시 자동 노출로 전환되면 이전 속도로 돌아갑니다. 카메라 자체가 촬영할 수 없는 값은 여전히 거부되며, 오류 메시지에 허용 범위가 표시됩니다. 또한 해당 쓰기 작업만을 위해 자동 노출이 꺼져 있었다면, 해당 멤버의 자동 노출이 다시 활성화됩니다. 레시피 `apply` 노출은 트리거 속도를 늦추지 않으며, 상한값으로 유지됩니다. Chloros는 더 빠른 트리거 속도에 맞추기 위해 수동 노출 시간을 단축하지 않습니다. 노출이 허용하는 속도보다 빠른 고정 속도(`array-connect --target-fps F`)가 설정된 경우, 해당 구성원은 그 속도에 맞는 상한선 길이를 유지하며, 해당 구성원은 노출이 허용하는 속도보다 빠르게 트리거를 수신하지 않습니다.

#### 보정 및 센서

| 하위 명령 | 용도 |
| --- | --- |
| `lattice calibrate [--filter NAME] [--attempts N] [--save PATH]` | 반사율 타겟을 기준으로 보정합니다. 최대 `--attempts` 프레임(기본값 `30`)까지 검색하며; 불완전한 프레임(GVSP 패킷 손실)은 실행 횟수가 아닌 시도 횟수 1회를 소모합니다. `no target detected in N attempts`일 경우 실패하며, 프레임이 온전하게 도착하지 않은 경우에는 `no complete frame in N attempts: <last error>`에서 실패합니다. |
| `lattice dls {--connect \| --spectrum \| --irradiance} --mac MAC [--filter NAME] [--camera SERIAL] [--json]` | DAQ-M의 스펙트럼 및 대역 진폭을 백엔드를 통해 블루투스(BLE)로 읽은 값입니다. `--mac`는 센서 고유의 BLE MAC 주소이며, 다음 세 가지 작업 중 하나를 수행할 때 필수입니다: 각 장치마다 고유한 값을 가지므로 기본값은 없습니다(`daq pool-discover --only ble`에 목록이 나열됨); 이 값이 없으면 명령어가 종료됩니다(`1`). 아무 작업도 지정하지 않으면 도움말이 출력됩니다. `--camera SERIAL`는 필터 모델의 플리트 곡선 대신 해당 카메라가 직접 측정한 스펙트럼 응답을 통해 출력된 대역 강도를 통합합니다 — RGB 카메라의 경우, 해당 카메라의 보정 팩이 로드(연결)되었거나 이 컴퓨터에 캐시되어 있을 때. 제목에는 어떤 곡선이 사용되었는지 표시됩니다: `Band magnitudes (camera <SERIAL>):`, 또는 카메라에 곡선이 없는 경우(RGB 카메라가 아니거나, 패키지가 로드되거나 캐시되지 않은 경우) 대신 플릿 곡선이 사용되었음을 나타냅니다. HTTP를 통해 `POST /api/dls/cli-read`는 `camera_serial`를 받아 `band_magnitudes_source`를 반환합니다. |
| `lattice vignette --input DIR --output DIR [--lens-model KEY]` | 기존 이미지에 비네팅 보정을 적용하며, 각 이미지는 해당 렌즈의 플랫 필드 (EXIF 모델 정보에서 가져옴)을 적용합니다. `--lens-model`는 모델 정보가 카메라를 지정하지 않은 이미지에 적용되며, 모델 정보가 이와 상충하는 이미지는 건너뜁니다. |

#### 다중 카메라 (일시적 세션)

일시적 명령은 한 번의 실행 동안 카메라를 열고, 실행이 끝나면 닫습니다: 지속적 배열이 연결되지 않은 `multi-capture`, 여기서의 `multi-stream` 및 `multi-test`, 아래의 `align-calibrate`, `align-apply`, `align-stream` 및 `index --live`도 마찬가지입니다. 다음 규칙은 이들 모두에 적용됩니다:

- **`--serials SN1,SN2,…`가 카메라를 선택합니다.** 선택된 카메라만 열리며, 첫 번째 카메라가 트리거를 작동시킵니다 (`align-calibrate`, `--reference`가 지정된 경우). 네트워크에 연결되지 않은 시리얼 번호는 어떤 카메라도 열리기 전에 실행이 실패합니다. `--serials`가 없으면 네트워크에 연결된 모든 카메라가 열립니다. Chloros가 이미 보유하고 있는 카메라(GUI, `cam-connect`, `array-connect`)는 `--serials`가 해당 카메라를 명시하든 그렇지 않든 실행을 거부합니다: 해당 카메라가 실행이 설정된 네트워크를 공유하고 있기 때문입니다. (`--serials`가 없는 `index --live`의 경우, 대신 해당 프로필에서 활성화된 카메라를 엽니다. [Index / Vegetation Maths](#index--vegetation-maths)를 참조하십시오.)
- **카메라들은 `array-connect`가 네트워크를 공유하는 방식과 동일하게 네트워크를 공유합니다.** 각 카메라는 호스트 링크가 전송하는 데이터 중 자신에게 할당된 비율에 맞춰 속도가 조절됩니다(패킷 간 지연 시간을 의미하며, 카메라의 처리량 한계를 뜻하는 것은 아닙니다. 1 GbE에서 4대의 카메라가 이미 31.25 MB/s라는 하한선을 초과하고 있습니다). 따라서 4대의 카메라가 1 Gbps 링크에서 작동합니다. 카메라들이 도달한 가장 느린 속도로 동시에 전송할 수 없을 경우, 각 카메라가 링크를 독점하며 차례로 트리거되며, 이때 명령어는 `Serial emit: the wire cannot carry these cameras at once, so each is triggered <ms> ms after the one before; their frames are not exposed together`를 출력합니다. 한 번에 단 한 대의 카메라라도 용량을 초과하면, 어떤 카메라도 스트리밍을 시작하기 전에 실행이 거부됩니다: `array over-subscribes the wire: <n> cameras demand <x> MB/s but the collision-safe ceiling is <y> MB/s. …`, HTTP 409, 종료 `1`.
- **모든 카메라가 데이터를 전송해야 하며, 그렇지 않으면 실행이 실패합니다.** 프레임을 전송하지 않는 카메라는 오류입니다: 명령어는 `Error: … no frame from <n> of <total> cameras: SN:<serial> (<why>), …`를 출력하며, 여기서 `<why>`는 해당 카메라의 데이터 수신 오류, 그 다음은 오류 원인을 나타내며, `1`로 종료됩니다. GVSP 패킷 손실(`IncompleteBufferError: GVSP packet loss …`)과 함께 도착한 프레임은 네트워크가 카메라들의 총 대역폭(`Every missing frame arrived with GVSP packet loss: …`)을 감당하지 못하고 있음을 의미합니다: 호스트와 스위치의 업링크 속도를 확인하고(`lattice network`), 점보 프레임을 사용하거나, 또는 `--serials`를 사용하여 연결된 카메라 수를 줄이십시오. 데이터를 전혀 전송하지 않은 카메라는 대개 마스터의 트리거 신호를 수신하지 못하는 경우입니다: 해당 카메라의 M8 동기화 케이블을 확인한 후, `lattice multi-detect`를 실행하십시오. 카메라가 누락된 프레임 그룹에 대해 각 명령어가 수행하는 작업은 해당 행에 명시되어 있습니다.
- **각 카메라는 발견된 상태 그대로 복원됩니다.** 실행이 종료되면 모든 카메라는 실행 시작 시의 상태로 복원됩니다: `TriggerMode`, `TriggerSource`, `TriggerActivation`, `TriggerOverlap`, `TriggerDelay`, 그리고 `Line2` 및 `Line3` 상의 `LineMode` / `LineSource` (그리고 트리거 입력 디글리치 배열이 슬레이브 라인에 제공하는 신호, `LineFilterSelector` / `LineFilterWidth`); 그것은 `PtpEnable`이며(이와 함께 설정된 `PtpSlaveOnly` / `PtpDomainNumber`); `ExposureAuto` / `GainAuto` (그리고 수동 값으로 설정되어 있던 `ExposureTime` / `Gain`); `TargetBrightness`, `ExposureAutoLimitAuto` / `ExposureAutoUpperLimit` / `ExposureAutoLowerLimit`, `GainAutoUpperLimit`, `ExposureAutoDamping`; `BalanceWhiteAuto` (그리고 화이트 밸런스가 수동으로 설정된 각 `BalanceRatio`); `BlackLevelAuto` / `BlackLevel`; `PixelFormat`, 비닝, `Width` / `Height` / `OffsetX` / `OffsetY`; `GammaEnable` (그리고 감마가 켜져 있던 `Gamma`), `LUTEnable`, `ColorTransformationEnable`; `ReverseX` / `ReverseY`; `UserSetSelector`; 패킷 크기 (`GevSCPSPacketSize` / `DeviceStreamChannelPacketSize`); 페이싱, `DeviceLinkThroughputLimitMode` (그리고 모드가 `On`인 경우 `DeviceLinkThroughputLimit`), 각 카메라에 대한패킷 간 지연 시간 `GevSCPD` 및 시작 시차 `GevSCFTD`는 배열을 통해 각 카메라에 제공되며, 각 스트림이 활성화할 때의 청크 데이터는 (`ChunkModeActive`, 각 `ChunkEnable`는 해당 `ChunkSelector` 아래에 위치함); `AcquisitionFrameRateEnable` (그리고 `AcquisitionFrameRate`는 활성화 상태였음 — 비활성화 상태에서는 속도가 읽기 전용이며 페이싱이 허용하는 최대값을 읽으므로, 이는 설정이 아님). 다른 점만 기록되며, 카메라가 먼저 정지하고, 프레임 주기가 필요한 노출 전의 페이싱 및 프레임 속도(카메라는 `ExposureTime`를 하나의 프레임 주기로 제한함)가 기록됩니다. `--keep-settings`의 경우 트리거 및 싱크 라인 배선이 복원되고, 카메라는 자유 실행 모드로 전환되며, 나머지는 실행 세트에 따라 카메라 자체 메모리에 저장됩니다. `--clear-settings`는 해당 메모리에 공장 초기 설정을 저장합니다.
- **노출은 첫 번째 프레임이 기록되기 전의 장면에서 설정됩니다.** 런(run)은 동기화된 프레임 그룹을 가져와 이를 버리면서, 해당 프레임들을 바탕으로 각 카메라의 노출과 게인을 설정합니다. 목표는 카메라의 대상 영역에서 평균 밝기를 확보하되, 프레임의 1% 이하만 클리핑되도록 하는 것이며, 밝은 하이라이트가 있는 장면에서는 목표 값에 미치지 못하도록 합니다. 이 과정에는 보통 2~6개의 프레임 그룹이 소요되며, 최대 5초가 걸립니다.
  - 각 카메라는 연결된 어레이가 시작 시 지정한 목표물을 필터를 통해 조준합니다: 다중 스펙트럼 카메라(RGN, OCN, NGB)의 경우 TargetBrightness **40**, RGB 카메라의 경우**60**입니다. 이는 노출 목표값 80의 절반(RGB의 경우 4분의 3)에 해당하므로, 밝은 반사율 패널이 클리핑될 가능성이 줄어듭니다. 대신 `--brightness N` (0-255)는 대신 각 카메라의 목표값을 설정합니다(`multi-test`에는 `--brightness`가 없습니다. 자동 노출 기능이 꺼져 있기 때문입니다). 백엔드의 로그에는 각 카메라의 목표값이 이름으로 표시됩니다: `[transient-ae] AE target per camera (filter default): SN:<serial> <target>, …`.
  - 노출이 먼저 적용되며, 카메라 자체의 자동 노출 한계치까지 적용됩니다. 게인은 남은 부분만 처리합니다.
  - 결정된 노출과 게인은 전체 실행 기간 동안 유지되므로, `multi-stream`의 모든 프레임은 동일한 노출로 촬영됩니다. 변화하는 빛에 맞춰 촬영해야 하는 캡처는 연결된 배열(`array-connect`)에 속해야 하며, 해당 어레이의 자동 노출은 지속적으로 조정됩니다.
  - `--exposure`와 `--gain`는 지정된 대로 정확히 유지됩니다. 두 가지가 모두 지정된 경우, 어떤 프레임도 버려지지 않습니다.
  - 실행이 종료되면 각 카메라의 자동 노출 기능이 다시 켜집니다.
  - 백엔드 로그에는 결과가 보고됩니다: `[array-ae] exposure solved in <s> s (<n> warm-up groups): SN:<serial> <ms> ms <dB> dB mean <m>/<target>, …` 또는 `exposure NOT solved after …`(이유 포함), 그리고 해당되지 않은 각 카메라에 대해서는 `(not settled)`가 기록됩니다. 이 경우에도 프레임은 여전히 촬영됩니다.
  - `CHLOROS_TRANSIENT_AE_SETTLE_S`는 5초 제한을 변경하며, `0`는 이 기능을 비활성화합니다([환경 변수](#environment-variables) 참조).

**Chloros가 어떤 카메라**(GUI, `cam-connect` 또는 연결된 어레이)를 보유하고 있는 동안, `multi-stream`, `multi-test`, `multi-detect`, `align-apply`, `align-stream` 및 `index --live`는 실행을 거부합니다. 이는 네트워크상의 모든 카메라를 열게 되며, 이미 보유 중인 카메라를 두 번째로 열려고 할 경우 잠겨 있는 카메라를 다시 열면 백엔드가 충돌할 수 있기 때문입니다. (`index --live`는 열려고 하는 카메라가 잠겨 있을 때 실행을 거부합니다. 해당 카메라는 `--serials`나 해당 프로필의 카메라, 또는 카메라가 지정되지 않은 프로필에 속한 모든 카메라입니다.) 백엔드는 HTTP 409 `CAMERA_HELD` 응답을 반환하며, 각 잠긴 카메라의 이름과 해제 명령어 (`chloros-cli lattice cam-disconnect --all` / `chloros-cli lattice array-disconnect --all`)를 명시하며, 명령은 종료됩니다(`1`). 어레이가 연결된 상태에서, `array-capture --continuous`는 `multi-stream` 대신 이를 기록하고, `array-capture` (기본적으로 정렬됨)는 `align-apply` 대신 정렬된 세트를 저장합니다. `multi-capture`와 `align-calibrate`는 카메라를 새로 열지 않고, 해당 카메라를 포함하는 연결된 배열을 사용합니다.

| 하위 명령 | 용도 |
| --- | --- |
| `lattice multi-info [--serials SN1,SN2,…]` | 동기화 역할을 가진 모든 카메라를 나열합니다. `--serials`를 사용하면 해당 카메라만 나열되며, 첫 번째는 `MASTER`와 같습니다. |
| `lattice multi-capture [--serials SN1,SN2,…] [--format FMT] [--jpeg-quality N] [--processing LEVEL] [--brightness N] [--no-daq] [--daq-tolerance-ms MS]` | 각 카메라에서 동기화된 프레임 하나씩 가져옵니다. 영구 배열이 연결된 경우 **기본적으로 모든 내보내기 유형**을 저장하며, 이를 읽는 DAQ([캡처를 읽는 DAQ가 저장하는 내용](#the-daq-reading-a-capture-saves)); 일시적인 어레이 미연결 시 대체 모드는**디베이어링만 수행**됩니다(나머지를 처리하려면 먼저 `array-connect`를 실행하십시오). `--brightness`는 대체 모드에만 적용됩니다: 연결된 어레이는 자체 자동 노출 설정을 유지합니다. 대체 모드는 `.daq`를 저장하지 않습니다 (`--no-daq` / `--daq-tolerance-ms`는 해당 상황에서 효과가 없음), 또한 `multi-capture`는 `--force-daq`를 적용하지 않습니다. 대체 모드에서, 프레임을 전달하지 않는 카메라는 캡처에 실패하며(`Error: multi-capture failed: no frame from …`, 종료 `1`), 해당 프레임 그룹의 파일은 저장되지 않습니다. |
| `lattice multi-stream [--serials SN1,SN2,…] [--fps F] [--count N] [--format FMT] [--jpeg-quality N] [--brightness N]` | 스트림 동기화 프레임(일시적), `--fps` 기본값 `3`, Ctrl+C 또는 `--count` 그룹이 발생할 때까지. 카메라가 프레임을 전혀 전송하지 않은 프레임 그룹은 기록되지 않으며, 3개 연속으로 발생하면 스트림이 종료됩니다. 어떤 그룹이든 건너뛴 경우, `Incomplete: <n> frame group(s) not written, a camera delivered nothing (SN:<serial> x<count>, …)`를 출력하고, 스트림이 종료된 카메라 이름을 명시한 `Error: the stream stopped: …`를 출력한 후 `1`X로 종료됩니다. 드라이브가 가득 차면 자동으로 중지되며, `Failed:` / `Disk full:`를 출력한 후 `1`로 종료됩니다; 쓰기 실패한 파일이 있는 경우에도 `1`를 출력하고 종료됩니다. |
| `lattice multi-test [--serials SN1,SN2,…] [--count N] [--ptp-settle SECONDS]` | GPIO 동기화 타이밍 테스트(과도 현상), `--count` 기본 `10` 트리거. 네트워크 공유 방식을 출력합니다 (`Wire: each camera paced to <x> MB/s (<n> on a <y> MB/s ceiling, <link> Mbps link)`, 그리고 카메라가 차례로 트리거될 때의 `Serial emit:` 줄—이때 타임스탬프에는 시간 차가 반영됨), 동기화 그룹당 한 줄씩(`Sync group: master SN:<serial> -> SN:<serial>, …`), 동기화 케이블이 연결되지 않아 자체적으로 트리거되는 카메라의 경우(`(no slaves: not synced)`), 그 다음 트리거마다 한 행씩: `[  0] SN:<serial>=<timestamp> \| …`, 해당 트리거에 대해 아무것도 전송하지 않은 카메라의 경우 `SN:<serial>=(no frame)`. 그 다음, 카메라당 한 줄씩 `Offsets from master SN:<serial> (<PTP / device (PTP off)> clock, <n> trigger(s)):` 및 `SN:<serial>: mean <±x> us, jitter <y> us`를 기록하며, 모든 카메라는 한 클럭 내에서 마스터와 동기화됩니다. 카메라의 장치 클럭과 PTP (PTP가 꺼져 있거나 아직 잠금 중)이 혼합된 타임스탬프를 가진 트리거는 제외되며, `Note:`가 이를 집계합니다. 누락된 프레임이 하나라도 있으면 테스트에 실패합니다(`Error: multi-test failed: <k> of <n> frame groups incomplete (SN:<serial> missed <k>); …`, `1` 종료). 불완전한 트리거가 3개 연속으로 발생하면 테스트가 조기에 종료됩니다. 또한, 모든 카메라가 하나의 클럭에 맞춰진 트리거가 두 개 미만일 경우(`1`)나, 클럭이 일치하지 않을 경우(`… the clocks cannot be compared`)에도 테스트는 실패합니다(`… PTP is not locked`). PTP 클럭이 마스터와 1ms 이상(직렬 계층의 스태거 포함) 차이가 나거나(`… PTP is not locked`), 트리거 사이에서 장치 클럭의 오프셋이 그 이상 변동하는 경우에도 실패합니다. PTP 그랜드마스터가 실행 중이 아닐 경우 이를 시작하고, 모든 카메라의 PTP를 켭니다(네트워크 상태와 관계없이 `array-connect`가 수행하는 방식과 동일). 또한 첫 번째 트리거가 발생하기 전에 모든 카메라의 PTP 클럭이 그랜드마스터에 동기화되어 안정화될 때까지 대기합니다: `PtpStatus`: 하나의 그랜드마스터에 슬레이브로 연결되어, 그랜드마스터와 100 us 이내의 오차를 유지하며 3초 동안 안정적으로 작동하고, 각 카메라가 전원. 방금 동기화된 클럭은 ‘Slave’로 판정된 후 약 2분 동안 최대 ~200 us의 오차를 보이지만, 카메라 자체의 `PtpOffsetFromMaster`는 이를 표시하지 않습니다. 120초가 지나면 카메라들의 클럭은 안정화된 클럭과 ~10 us 이내의 오차 범위로 수렴합니다. 이 백엔드가 확인한 바에 따르면, 이 백엔드에서 확인한 바와 같이, PTP를 켠 상태로 진행된 이전 테스트 이후로 여전히 동기화 상태를 유지하고 있는 경우, 해당 동기화 시점부터 카운트를 시작하므로, 연속된 테스트 간에는 다시 대기하지 않습니다. 반면 테스트 중에 전원이 켜진 카메라나, 이 백엔드에서 동기화된 적이 없는 상태로 PTP가 켜진 채 발견된 카메라(다른 곳에서 켜졌거나 백엔드 재시작 전에 켜진 경우)는 120초를 대기하며, 두 카메라가 혼합된 배열에서는 방금 동기화된 카메라들도 여전히 대기 시간을 채웁니다. 슬레이브 상태를 벗어나거나, 대기 중 100 us를 초과하거나, 다른 그랜드마스터를 3회 연속(1.5 s)으로 추적하는 카메라는 카운트를 다시 시작합니다. 이러한 판독값 중 하나 또는 두 개는 (`[array-sync] PTP SN:<serial> off lock at <s> s (<n> of 3): <status> <±x> us gm <id> -- locked <s> s, counting on`)로 기록되며 카운트는 계속됩니다. 또한 읽을 수 없는 그랜드마스터 ID는 변경 사항이 아닌 ‘알 수 없음’으로 간주됩니다. 모든 카메라의 카운트가 동시에(1초 이내) 다시 시작되는 것은 카메라의 시간이 아닌 그랜드마스터의 시간이 이동했기 때문입니다. 백엔드 로그에는 `[array-sync] PTP grandmaster step: all <n> cameras restarted at <s> s …`, `ptp_lock`는 이를 `grandmaster_steps`에 나열하며, 이후 시간이 초과된 대기 작업은 `every camera's count restarted together at <s> s -- the PTP grandmaster's time moved under them …`로 오류가 시작됩니다. 이에 대한 시간 제한은 연장되지 않습니다. `--ptp-settle SECONDS`는 해당 실행에 대한 정산 시간을 설정하며(0~600; 정산 대기 없음의 경우 `0`). 대기 시간은 안정화 시간에 90초를 더한 값으로 제한되며(기본값 210초, 최소 90초), 명령어는 백엔드가 안정화 시간에 240초를 더한 시간 (기본값 360초) 동안 백엔드를 대기합니다. `PTP: locked after <s> s (each camera locked <settle> s, already locked: SN:<serial>, …; from the grandmaster: SN:<serial> <±x> us, …)`가 출력되며, 백엔드 로그에는 5초마다 각 카메라의 상태와 오프셋, 잠금 상태 지속 시간 및 남은 시간이 기록됩니다: `[array-sync] PTP at <s> s: SN:<serial> Slave <±x> us for <s> s (<s> s left), … -- <s> s left, the wait gives up at <s> s`. 모든 카메라는 PTP가 꺼진 상태로 전원이 켜지며, 그 후 약 10초 만에 클럭이 동기화됩니다. 이후 각 카메라는 테스트 시 확인된 상태로 복원됩니다 — 즉, PTP, 테스트 실행 중 비활성화된 자동 노출 및 게인, 그리고 나머지 설정(`--keep-settings`인 경우 제외) — 따라서 PTP가 켜진 상태로 남겨진 카메라(`lattice configure -s <serial> --set PtpEnable=True`, 또는 `--keep-settings`가 적용된 실행)는 동기화된 상태를 유지하며, 이 백엔드가 해당 카메라가 안정화 시간 동안 동기화된 상태를 확인하면 이후 테스트에서는 이를 기다리지 않습니다. 대기 중에 클럭이 안정화되지 않으면, 트리거되기 전에 실패합니다(`PTP: NOT locked …`, 종료 `1`) 그리고 어떻게 처리해야 하는지 안내합니다: 한 번도 잠금 상태가 되지 않은 카메라(`PtpStatus Listening`)는 그랜드마스터가 해당 카메라에 도달하지 못하고 있음을 의미합니다(`time-sync status`); 계속 연결이 끊기던 카메라는 잠금 상태가 유지된 기간을 기준으로 이름이 지정됩니다 (`… had not been locked for the 120 s settle time … (SN:<serial> locked <s> s, <s> s to go)`); 클럭이 여전히 수렴 중인 경우: `time-sync status`를 확인하고, 테스트 전에 카메라를 동기화 상태로 유지하려면 PTP를 켜고 계속 켜두십시오(각 카메라마다 `lattice configure -s <serial> --set PtpEnable=True` 설정하거나, `--keep-settings`로 테스트를 실행하십시오; 테스트는 감지된 설정을 복원하므로 PTP가 계속 켜진 상태로 유지됩니다). 그런 다음 테스트를 실행하십시오. |
| `lattice multi-detect [--line LINE] [--json]` | M8 동기화 라인에 연결된 카메라와 기본 마스터를 확인하십시오: 케이블이 연결된 경우 RGB 카메라(모델명이 `-FRGB`로 끝나는 카메라)가 마스터가 되며, 그렇지 않은 경우 직렬 번호 순서대로 첫 번째로 케이블이 연결된 카메라가 마스터가 됩니다. M8 케이블은 하나의 접속점에서 모이기 때문에 **케이블이 연결된 어떤 카메라라도 마스터가 될 수 있습니다** — 인쇄된 `MASTER`는 기본값일 뿐, 배선상의 사실이 아닙니다. 각 카메라가 차례로 라인을 구동하고 다른 카메라들은 해당 펄스를 감지하는지 확인합니다. 이미 연결된 그룹에서 발견된 카메라는 다시 탐색되지 않으므로, 따라서 완전히 케이블로 연결된 배열은 한 번의 탐색 주기를 거칩니다. `--line`가 없으면 `Line2`를 탐색하고, 그 다음 `Line3`를 탐색합니다. `MASTER: SN:…`를 출력하고, 케이블로 연결된 다른 카메라 하나당 `SLAVE: SN:…`를 하나씩 출력한 뒤 `(probed on <line>)`를 출력합니다. `--json`는 전체 결과(`master`, `slaves`, `no_cable` — 펄스가 도달하지 않은 카메라 — `line`, 그리고 카메라별 `results`)를 더합니다.. 처리가 완료되면 모든 카메라는 원래의 트리거 및 싱크 라인 설정(위와 동일한 노드)으로 복원됩니다. |

#### 정렬

| 하위 명령 | 용도 |
| --- | --- |
| `lattice align-calibrate [--serials SN1,SN2,…] [--method orb\|akaze\|phase\|checkerboard\|manual] [--model translation\|rigid\|affine\|homography] [--frames N] [--checkerboard RxC] [--points PATH] [--reference SN] [--save PATH] [--preview] [--vignette] [--prefilter none\|gradient\|clahe\|blur\|hist_match] [--rms-threshold-px N] [--brightness N]` — 검출기/매칭 노브 추가 `[--max-features N] [--ratio-threshold F] [--matcher bf\|flann] [--knn-k N]`, RANSAC 조정값 `[--ransac-threshold-px F] [--ransac-iters N] [--ransac-confidence F]`, 다중 프레임 결합 `[--averaging mean\|median\|inlier_weighted]`, 기하학적 제약 조건 `[--lock-rotation] [--lock-scale] [--lock-axis x\|y]`, 공간적 제한 `[--roi X0,Y0,X1,Y1] [--mask PATH]` 및-슬레이브 오버라이드 `[--per-cam-override SN:KEY=VALUE]` (반복 가능) | 라이브 카메라로부터 정렬 프로필을 계산합니다. 기본값: `--method orb`, `--model rigid`, `--frames 3`, `--save alignment_profile.json`(실행하는 폴더 내), `--max-features 2000`, `--ratio-threshold 0.75`, `--matcher bf`, `--knn-k 2`, `--ransac-threshold-px 3.0`, `--ransac-iters 2000`, `--ransac-confidence 0.99`, `--averaging mean`. (`project align calibrate`는 별도의 기본값을 갖습니다. [해당 표](#project-align-calibrate-options)를 참조하십시오.) `--prefilter`의 기본값은 `gradient` (에지 맵; GUI/배열 정렬기와 일치하며, 에지는 스펙트럼 대역 간에도 유지됨). `--matcher flann`는 특징 수가 ~5000개 이상일 때 효과가 나타나며; `--averaging median`는 하나의 잘못된 캡처에 대해 견고하며, `inlier_weighted`는 일치 횟수에 따라 가중치를 부여합니다; `--lock-scale`는 가장 가까운 회전(스케일 없음)으로 투영하며, `--lock-axis`는 한 개의 평행 이동 성분을 0으로 설정합니다; `--mask`는 모든 카메라에 적용됩니다(카메라별 설정을 원할 경우 `--per-cam-override`를 사용하십시오. 예: `--per-cam-override 214701292:method=phase`). `--rms-threshold-px`는 재투영 RMS가 허용 범위를 초과하는 보정값의 저장을 거부합니다. `--method manual`는 `--points PATH`에서 점을 가져옵니다: JSON 객체인 `{"<serial>": [[x, y], ...], ...}`는 각 카메라에서 동일한 특징점의 픽셀 좌표를 제공하며, 여기에는 기준 카메라의 좌표도 포함됩니다(카메라당 최소 2개); `--frames`의 각 항목은 이러한 점들을 기반으로 계산됩니다. 연결된 배열에 카메라가 포함되어 있는 경우, 보정은 해당 배열의 라이브 프레임을 사용하며, 프로필은 해당 배열의 캡처에 대한 정렬 정보로도 사용됩니다. 그렇지 않은 경우 [임시 배열](#multi-camera-transient-sessions)을 생성합니다: `--frames` 중 어느 하나에서도 프레임을 전송하지 않는 카메라는 보정 실패로 간주되며(`1`에서 해당 카메라 이름을 지정하여 종료), 프로필은 저장되지 않습니다. |
| `lattice align-apply --profile PATH [--serials SN1,SN2,…] [--format tiff\|png] [--bit-depth 8\|12\|16] [--bands NAMES] [--order NAMES] [--gpu\|--no-gpu] [--no-crop] [--per-camera] [--per-band] [--vignette] [--interpolation nearest\|linear\|cubic\|lanczos] [--border-mode constant\|replicate\|reflect\|wrap] [--border-value N] [--brightness N]` | 정렬된 멀티밴드 프레임 하나를 캡처합니다. `--bit-depth`는 기본적으로 카메라와 일치하도록 설정되며; `--no-crop`는 전체 프레임을 유지합니다(검은색으로 채움); `--interpolation`(기본값 `linear`) 및 `--border-mode`/`--border-value`(기본값 `constant`/0)는 CPU 워프를 제어합니다. GPU 경로는 어떤 경우든 이차선형(bilinear)입니다. [일시적 배열](#multi-camera-transient-sessions): 프레임을 전달하지 않는 카메라는 캡처에 실패하며(`1`로 종료, 이름을 지정), 아무것도 기록되지 않습니다. |
| `lattice align-stream --profile PATH [--serials SN1,SN2,…] [--fps F] [--count N] [--bit-depth 8\|12\|16] [--bands NAMES] [--order NAMES] [--gpu\|--no-gpu] [--no-crop] [--per-band] [--vignette] [--interpolation nearest\|linear\|cubic\|lanczos] [--border-mode MODE] [--border-value N] [--brightness N]` | 스트림 정렬된 멀티밴드 프레임 (`align-apply`와 동일한 워프 조절 기능), `--fps` 기본값, `3`. [일시적 배열](#multi-camera-transient-sessions)을 엽니다: 카메라가 프레임을 전혀 전송하지 않은 프레임 그룹은 저장되지 않으며, 3개 연속으로 발생하면 스트림이 종료됩니다. 어떤 그룹이든 건너뛴 경우, 실행은 `Incomplete: <n> frame group(s) not saved, a camera delivered nothing (SN:<serial> x<count>, …)`와 `Error: …`를 출력한 후 종료되며, `1`를 출력한 후 종료됩니다. 다른 오류로 인해 종료되는 스트림도 `Error: …`를 출력하고 `1`를 출력한 후 종료됩니다. |
| `lattice align-info --profile PATH [--json]` | 프로필 세부 정보 표시. |
| `lattice align-reorder --profile PATH [--order NAMES] [--enable SERIALS] [--disable SERIALS]` | 레이어 순서 변경. |

#### 색인 / 식생 수학

```bash
# Offline: compute NDVI from an aligned multi-band TIFF (an RGN camera's bands)
chloros-cli lattice index --input aligned.tif --preset NDVI \
  --channel red=Red_660_RGN --channel nir=NIR_850_RGN \
  --output ndvi.tif --colorize --gradient RdYlGn

# Live: open the cameras, capture one aligned set with a profile made by
# `lattice align-calibrate`, and compute the index from it
chloros-cli lattice index --live --profile align.json --preset NDVI \
  --channel red=Red_660_RGN --channel nir=NIR_850_RGN \
  --save-multiband -o output/
```

**`--channel`는 사전 설정의 기호를 밴드에 매핑합니다** (NDVI: `red`, `nir`), 각각 하나씩 `--channel SYMBOL=BAND`.

- **`--live`: 스택이 응답할 수 없는 심볼에만 적용됩니다.** `--input`와 마찬가지로, 생략한 프리셋 심볼은 스택(프로파일의 활성화된 카메라 밴드) 중에서 이름이 해당 심볼로 시작하는 하나의 밴드를 차지합니다. 이는 어떤 카메라도 열리기 전에 확인되며, 응답하는 밴드가 없거나 여러 밴드가 응답하지 않는 심볼은 거부되고, 해당 심볼의 이름을 명시한 후 `1`에서 종료됩니다. 사용자 정의 `--formula`는 각 심볼마다 `--channel`가 필요합니다.
- **`--live` 제품은 해당 제품이 누구의 것인지 명시합니다.** `index.tif` 및 `index_LUT.png`는 스택에 포함된 카메라(제조사, 모델, 일련번호), 촬영 시간(참조 카메라의 노출 시간, UTC) 및 렌즈 정보를 포함하며, 두 제품 모두 인덱스를 단일 밴드로 지정합니다(`XMP-Camera:BandName`, `NDVI`; 컬러 PNG 파일은 색상 채널이 아닌 인덱스를 이름으로 사용합니다). 이는 `--input` 제품과 동일한 방식입니다. 반면, 여러 대의 카메라 밴드로 구성된 인덱스는 각 밴드의 이름을 모두 명시하며, 이에 따른 `Model` 파일명은 `LATTICE`가 됩니다 — [여러 대의 카메라로 생성된 파일](#files-made-from-several-cameras)을 참조하십시오.
- **`--input`: 파일이 응답할 수 없는 심볼에만 해당합니다.** 생략한 심볼은 파일 내에서 이름이 해당 심볼로 시작하는 하나의 밴드를 사용합니다: `red`는 `Red_660` 또는 `Red_660_RGN`를, `nir`는 `NIR_850`를 사용합니다. 이름은 파일 자체에서 비롯됩니다. 즉, 스택 자체의 밴드 목록이거나, Chloros 방사도/반사도 캡처 또는 `process` 제품의 경우, 해당 `Chloros:Band{R,G,B}Name` / `XMP-Camera:BandName` 태그에서 유래합니다. 따라서 `lattice index --input <reflectance capture> --preset NDVI`에는 `--channel`가 필요하지 않습니다.
- **하나의 심볼에 대응하는 두 개의 밴드는 추측 대상에서 제외됩니다** (9대 카메라 스택에는 두 개의 NIR 밴드가 있습니다): 해당 밴드를 지정하십시오. 어떤 심볼에도 대응하지 않는 밴드는 `channel_map missing entries`로 거부되며, `1`로 종료됩니다.**스택 내의 밴드 이름**은 카메라의 필터를 나타냅니다: M3C 카메라의 밴드는 `<band>_<filter>` — `NIR_850_RGN`, `Green_550_RGN`, RGN의 경우 `Red_660_RGN`; `NIR_808_OCN`, `Cyan_490_OCN`, OCN에서는 `Orange_615_OCN`; RGB에서는 `Blue_475_RGB`, `Green_550_RGB`, `Red_625_RGB`; `Blue_475_NGB`, `Green_550_NGB`, `NIR_850_NGB`는 NGB에 탑재되며 — M3M 카메라의단일 대역은 `Mono_<filter>`입니다(`Mono_660`, `Mono_850`). `lattice align-info --profile PATH`는 프로파일의 대역을 나열합니다.

전체 플래그 세트: `--input PATH | --live --profile PATH`, `--preset NAME` (NDVI/NDRE/EVI/SAVI/GNDVI/…), `--formula EXPR`, `--channel SYM=BAND`(반복 가능), `--capture-level raw|debayered|radiance|reflectance|unknown`(소스 TIFF에 기록된 캡처 레벨을 재정의함; 기본값: TIFF 메타데이터에서 읽으며, Chloros 파일의 `Chloros:ProcessingLevel`로 대체되므로, 반사율 캡처는 `reflectance`를 보고함), `--output PATH` / `-o` (오프라인: 경로; 이미지 확장자가 있는 경우 파일 이름은 해당 줄기 — `-o ndvi_FRGN.png`는 `ndvi_FRGN.tif` 및 `ndvi_FRGN_LUT.png`로 저장됨 — 확장자가 없는 경우 접두사로 사용되며, `-o ndvi` -> `ndvi.tif`, `ndvi_LUT.png`; 입력 외에 기본값은 `<input name>_<preset>`와 같이; `--live`가 있고 `-o`가 없는 경우, 명령을 실행한 폴더 아래에 `output/` 폴더가 생성됨), `--output-format all|raw|tif|colorized|lut|png`, `--gradient NAME|JSON`, `--vmin/--vmax/--percentile LO,HI`, `--bg-mode clip|transparent|indexColor|backgroundColor`, `--colorize`, `--list-presets`, `--list-gradients`. `--live`부터는 정렬 워프 노브도 적용됩니다: `--save-multiband`, `--gpu/--no-gpu`, `--no-crop`, `--bit-depth 8|12|16`, `--vignette`, `--interpolation nearest|linear|cubic|lanczos`, `--border-mode constant|replicate|reflect|wrap`, `--border-value N`, `--brightness N` (카메라의 자동 노출 대상, 일시적 규칙 참조), `--serials SN1,SN2,…`. `--live`는 [일시적 배열](#multi-camera-transient-sessions): 프레임을 전달하지 않는 카메라는 실행에 실패합니다(`1`로 종료하며 이름을 지정함), 스택이나 인덱스는 기록되지 않습니다. `--serials`를 엽니다(첫 번째 카메라가 트리거를 발동시킵니다). 그렇지 않은 경우, `layer_order` 항목이 활성화된 프로파일의 카메라들(참조 카메라가 먼저 처리됨 — 다른 카메라들이 참조 카메라에 맞춰져 있으므로, 참조 카메라의 밴드가 비활성화된 경우에도 트리거가 발동됩니다 그 참조 카메라에 맞춰져 있고, 이 참조 카메라가 트리거를 발동하기 때문입니다. 밴드가 비활성화된 카메라는 절대 보정되지 않으므로, 프로필에서 비활성화된 RGB 카메라는 더 이상 반사율 인덱스(`no-absolute-radiance-scale`) 테스트에 실패하지 않습니다. 카메라를 명시하지 않은 프로필은 이전과 마찬가지로 네트워크상의 모든 카메라를 엽니다.

모든 양식은 백엔드에서 실행되므로 백엔드가 실행 중이어야 합니다. Chloros GUI에서 시작하거나, CLI가 백엔드에 연결할 수 없을 때 출력하는 명령어를 사용하여 시작하십시오. 오프라인 상태에서 백엔드는 `--input`를 읽고 부동 소수점 TIFF 및 색상 처리된 PNG 파일을 직접 작성하므로, 상대 경로인 `--input` 또는 `-o`는 명령어를 실행한 폴더를 기준으로 해석됩니다([약어 및 표기법](#conventions) 참조). 두 오프라인 산출물 모두 `process` 인덱스 내보내기에서와 마찬가지로 입력 파일의 카메라 기록을 포함합니다: 제조사/모델/일련번호, GPS, 렌즈 태그, 촬영 시간(입력 파일 자체의 시간; EXIF에 정보가 없는 경우, 기록된 노출 순간 또는 파일 이름의 타임스탬프, UTC 기준) 및 MAPIR 보정 식별자 — 인덱스에는 적용되지 않는 raw-DN 보정은 절대 포함되지 않음 — 뿐만 아니라 `Chloros:ProcessingLevel=index` 및 인덱스 표현식을 포함합니다. float TIFF는 하나의 밴드, 즉 프리셋을 나타냅니다. `--list-presets` 및 `--list-gradients`는 백엔드의 프리셋 및 그라디언트 테이블을 출력합니다. 백엔드가 거부하는 수식이나 TIFF에 없는 `--channel` 밴드는 백엔드의 오류 사유와 함께 `1`를 종료합니다.

> **`--channel` 기호는 대소문자를 구분합니다.** 심볼 쪽은 프리셋의 채널 이름과 정확히 일치해야 하며(프리셋은 소문자를 사용함, 예: NDVI = `red`, `nir` — `--list-presets` 확인),, 밴드 쪽은 정렬된 스택의 밴드 이름과 일치해야 하며(또는 오프라인 모드에서는 0을 기준으로 한 밴드 인덱스여야 함). `--channel red=Red_660_RGN --channel nir=NIR_850_RGN`는 작동하지만, `--channel RED=660`는 `channel_map missing entries` 오류가 발생합니다.

#### `lattice index` — 멀티밴드 스택 마스크

`--capture-level radiance` 또는 `reflectance`에서 각 카메라의 3×3 언믹스(unmix)는 밴드를 0으로 클램프할 수 있습니다 로 클램핑할 수 있습니다([언믹스 클램프 마스크](#unmix-clamp-mask-which-pixels-a-band-was-clamped-on) 참조)로 클램프할 수 있습니다. 멀티밴드 스택은 여러 대의 카메라에서 촬영된 밴드를 포함하며, 각 카메라는 자체 밴드를 클램프하므로, `--live --save-multiband`는 스택 옆에 **소스 카메라당 하나의 마스크**를 작성합니다:

```
output/
├── index_multiband.tif
└── clamp_masks/
    ├── index_multiband_SN<serial A>-<filter A>_clampmask.tif
    └── index_multiband_SN<serial B>-<filter B>_clampmask.tif
```

- 각 마스크는 **스택의** 픽셀 그리드 상에서(동일한 정렬 워프 및 크롭을 통해) 단일 밴드인 `uint8`이며, 해당 카메라 고유의 비트를 갖습니다: 비트 0 = R 채널의 밴드, 비트 1 = G, 비트 2 = B입니다. 카메라의 채널 *c*에서 가져온 스택 밴드는 해당 카메라의 비트 *c*(R = 0, G = 1, B = 2)가 설정된 위치에서 클램핑됩니다.
- 각 밴드에 포함된 출처 정보에는 해당 카메라의 마스크 이름 (`unmix_clamp_mask`, 스택 폴더를 기준으로 함, 또는 `none`)와 읽을 비트(`unmix_clamp_mask_bit`)를 나타냅니다.
- 언믹스(unmix) 과정에서 클램핑된 항목이 없는 카메라는 파일이 생성되지 않으며, 모노(mono) 또는 RGB 카메라는 언믹스를 실행하지 않으므로 파일도 생성되지 않습니다. CLI는 스택 아래에 각 마스크(`clamp mask SN…: …`)를 출력하며, 응답에는 이를 `clamp_masks`로 나열합니다.
- 동일한 실행의 인덱스 TIFF/PNG 파일은 동일한 스택과 동일한 그리드를 기반으로 계산되므로, 마스크는 이 파일도 설명합니다. `--capture-level raw`는 언믹스 처리를 수행하지 않으며 마스크도 기록하지 않습니다.
- 스택과 인덱스는 `--output`에 바로 위치하므로, 마스크는 `--output` 내의 `clamp_masks/`에 저장됩니다. 절대 사용자가 지정한 폴더 위에는 위치하지 않습니다. 해당 폴더에는 다양한 유형의 결과물(스택, 인덱스, 컬러 렌더링)이 혼합되어 있습니다. 해당 폴더 전체를 사진측량 소프트웨어에 넘기지 말고, 필요한 파일만 선택하여 전달하십시오.

#### 지속적 연결 (Smart-Prep, GUI와 동등한 흐름)

이 명령어들은 CLI 호출 간에도 백엔드 풀에서 카메라를 계속 열어둡니다.

| 하위 명령 | 용도 |
| --- | --- |
| `lattice cam-connect [--serial SN]` | 풀에 카메라 1개를 추가합니다(단일 카메라, 배열 없음). [공통 옵션](#common-options-shared-by-most-subcommands) `--exposure`, `--gain`, `--preset`, `--pixel-format`, `--width` / `--height` 및 `--packet-size` (`standard` = 1500, `jumbo` = 9000 또는 바이트; `auto` 프로브)는 연결 시 적용됩니다 (`--exposure` 또는 `--gain`는 자동 모드를 끕니다). 카메라의 스트림이 시작되면(이를 위해 약 15초를 대기한 후, 첫 번째 버퍼가 완전히 채워질 때까지 10초를 대기) 반환되며, 스트림에서 아무것도 전송되지 않은 경우 `stream: streaming` 또는 `stream: no_frames -- connected, no frames yet: …`를 출력합니다 — (점보 패킷을 사용하는 카메라의 경우, `--packet-size standard`로 재연결하라는 안내가 표시됨). 한 대의 카메라가 연결되는 동안(첫 연결 시 보정 데이터를 다운로드하는 데 약 70초 소요), 카메라가 필요한 다른 명령은 최대 5초 동안 대기한 후, 그 후 연결 중인 카메라 이름을 명시한 HTTP 503 오류(`CAMERA_POOL_BUSY`)로 실패합니다. 잠시 후 다시 실행해 보세요. `lattice status` 및 카메라 목록은 절대 대기하지 않습니다. 연결 중인 카메라는 연결 상태로 표시되며, 해당 단계가 함께 표시됩니다. |
| `lattice cam-disconnect [--serial SN] [--all] [--keep-settings \| --clear-settings]` | 해제. 카메라는 연결 시 발견된 상태(트리거 배선, PTP, 설정)로 복원됩니다; `--keep-settings`는 대신 설정을 자체 메모리에 저장하고, `--clear-settings`는 해당 메모리를 공장 초기 설정으로 재설정합니다 ([카메라 메모리](#camera-memory----keep-settings----clear-settings) 참조). 어레이 구성원인 카메라는 전체 어레이를 해제합니다. |
| `lattice cam-list` | 풀에 있는 카메라 목록 표시. |
| **`lattice array-connect`**|**지속적인 동기화 어레이 연결 (권장 진입점).** 전체 GUI 스마트 준비(smart-prep) 흐름을 실행합니다. 카메라 9대 기준 약 1분이 소요되며, 카메라 전원이 켜진 직후에는 몇 분이 소요될 수 있습니다. 대기 중에는 백엔드가 도달한 각 단계를 `    <s> s: <step>`(예: `    41 s: open SN:214000535 (4 of 9)`)와 같이 출력하며, 백엔드 로그에는 단계별로 하나의 타이밍 행(`[array-connect <id>] <step>: <s> s …`)이 기록되며, 마지막 행에는 가장 느린 단계들이 표시됩니다. 최대 600초까지 대기합니다. 그 때까지 연결이 완료되지 않으면 백엔드에서 연결을 취소합니다: 열었던 모든 카메라는 발견된 상태로 되돌려진 후 닫히며, 명령어는 도달한 단계를 포함하여 `Error:`를 출력한 후 종료되며(`1`), 어레이는 연결되지 않은 상태가 됩니다(HTTP 504 `CLIENT_DEADLINE_PASSED`). Ctrl+C를 누르면 동일한 방식으로 연결이 취소됩니다(`Interrupted: …`); 어레이가 이미 연결된 상태라면, 해당 줄에 그 사실이 표시되며 이를 해제하는 `array-disconnect`의 이름이 명시됩니다. 대기 중에 강제 종료된 명령어 (`kill -9`)는 백엔드가 30초 동안 해당 명령으로부터 응답을 받지 못하면 연결이 취소됩니다. 가동 중이지만 아직 동기화된 프레임 그룹이 없는 어레이도 여전히 연결된 것으로 보고되며, 해당 스트림은 `no_frames`로 표시됩니다. |
| `lattice array-disconnect [--array-id ID] [--all] [--keep-settings \| --clear-settings]` | 어레이를 해제합니다. 모든 카메라는 자유 실행 상태(`TriggerMode` `Off`)로 남겨지며, 트리거 및 동기화 라인 설정(마스터의의 `TriggerSource`는 원래 상태로, 싱크 라인은 입력 상태로 복원됨), PTP, 그리고 세션에서 변경된 카메라 설정은 그대로 유지됩니다. 따라서 멤버에 대한 단일 카메라 명령이 즉시 적용됩니다. `--keep-settings`는 각 카메라의설정을 세션 종료 시의 상태로 카메라 자체 메모리에 저장하며, `--clear-settings`는 해당 메모리를 공장 출하 시 설정으로 초기화합니다. [카메라 메모리](#camera-memory----keep-settings----clear-settings)를 참조하십시오. |
| `lattice array-list` | 연결된 어레이 목록 표시. |
| `lattice array-status [--array-id ID]` | 라이브 fps, PTP, 마지막 오류. `ptp_enabled`는 모든 구성원이 PTP를 켜진 상태로 읽을 때만 `True`가 되며, `ptp_off:`는 그렇지 않은 구성원의 이름을 지정합니다. 설정 재시작 중에 `state:` 라인은 설정 재시작 중에 `restarting (applying settings)`를, 전원이 끊겼던 멤버가 다시 열릴 때(아래 참고 사항 참조) `recovering (…)`를, `stalled (…)`는 그랩 루프가 실행 중이지만 프레임 그룹을 게시하지 않을 때, `stopped (…)`는 그랩 루프가 중단된 후(어레이 재연결)에 발생합니다. |
| `lattice array-capture [--processing LEVEL\|all] [--levels L1,L2,…] [--aligned\|--no-aligned] [--orient] [--index\|--no-index] [--no-daq] [--daq-tolerance-ms MS] [--smart] [--fastest] [--compression deflate\|none] [--continuous\|--interval S] [--count N] [--duration S]` | 라이브 어레이에서 동기화된 캡처 1회 — 단일 / 연속 / 간격 / 가장 빠른 모드. **기본값은 `all`**입니다(카메라당 해당 내보내기 유형별로 파일 1개), 앱의 캡처와 동일하게**정렬**됩니다(네이티브 파일의 경우 `--no-aligned`). 건너뛴 카메라(예: RGB는 방사도/반사도에서 제외됨)는 `Skipped: SN:<serial> (<reason>)`로 보고됩니다. 사용된 캡처를 읽은 DAQ 기록은 함께 저장되며, 원시 데이터 전용 캡처가 포함되어 `DAQ: <path>`로 보고됩니다 ([캡처를 읽는 DAQ가 저장하는 내용](#the-daq-reading-a-capture-saves)). [캡처 모드, 레코더 및 오프라인 재처리](#capture-modes-recorders--offline-reprocess)를 참조하십시오. |
| `lattice array-record [--fps F] [--duration S] [--gif] [--gif-only] [--expr EXPR]` | 실시간 통합 인덱스 뷰를 비디오/GIF로 기록합니다(모니터링 등급; GUI 뷰가 열려 있지 않은 상태에서 백엔드가 어레이의 NDVI를 렌더링하거나, `--expr`). |
| `lattice array-burst [--duration S] [--max-frames N] [--orient] [--build] [--products …]` | 고프레임률 원시 베이어 버스트 (분석용; 오프라인 재처리). `array-record` 및 `array-capture --continuous`와 마찬가지로, 매 폴링 시 백엔드에 상태를 보고합니다: 중지 없이 종료됨 (`kill -9`), 백엔드는 마지막 체크인 후 30초가 지나면 이를 중지시킵니다(`stop_reason`, `client_gone`; 중지 시와 마찬가지로 버스트의 매니페스트가 확정되고 결합 인덱스 생성이 시작되며), 배열은 연결된 상태를 유지합니다. |
| `lattice array-build-video --burst-dir DIR [--products …] [--fps F] [--save-tiffs] [--gif]` | 저장된 원시 버스트를 보정된 비디오로 재처리합니다. |

> **전원이나 연결이 끊겼던 카메라가 스스로 복구됩니다.** 구성원이 응답을 중단하면(스위치의 전원 재시작 시 전원을 공급받는 모든 카메라가 재부팅됨), 어레이는 해당 카메라를 기다렸다가 다시 응답하면 연결을 재개하고, `array-connect`에서 설정한 값(PTP, 트리거 및 싱크 라인, 네트워크 페이싱, 픽셀 형식 및 프레임 크기, 자동 노출)을 다시 전달합니다. 그런 다음 새로운 `array-connect` 설정 없이 다시 스트리밍을 시작합니다. 한편, `array-status`는 `state: recovering (...)`를 표시하고, `last_error`는 대기 중인 카메라를 알려주며, 그 후 `re-opened:`를 출력합니다. 재부팅된 카메라는 PTP가 꺼진 상태로 복귀하므로, 다시 열릴 때까지 `ptp_enabled`는 `False`가 되며, `ptp_off:`는 해당 카메라의 이름을 표시합니다.

> **보류 중인 카메라는 두 번째로 열리지 않습니다.** 두 번째로 열면 보류 중인 스트림이 중지되고 설정이 재작성되며, 백엔드가 충돌할 수 있습니다. 따라서 풀이 홀드 중인 카메라(GUI 또는 `cam-connect`), 배열 멤버, 또는 백그라운드 작업이 홀드 중인 카메라(실행 중인 `lattice continuous`, `multi-stream` 또는 `align-stream`, 자체 또는 다른 클라이언트의 것), `lattice continuous`, `lattice capture`, `cam-connect`, `power --sleep`, `calibrate`, 단일샷인 `white-balance` 및 `presets --apply`는 HTTP 409 오류로 거부하며, `CAMERA_HELD`는 소유자와 해제 명령을 명시하고 (`chloros-cli lattice cam-disconnect -s <SN>`, `chloros-cli lattice array-disconnect --array-id <ID>`, 또는 작업의 경우 ID와 시작자)를 명시하고 종료합니다 `1`. 다른 명령이 현재 사용 중인 카메라도 마찬가지입니다(카메라를 여는 `multi-test` 또는 `multi-capture`, 또 다른 원샷). 여기 있는 모든 명령에 대해 오류 메시지는 해당 명령이 완료될 때까지 기다리라고 표시합니다. `lattice power`는 `--sleep`가 없는 경우, 홀더 자체의 핸들(풀, 배열, 작업의 열린 카메라)을 통해 읽으며, 다른 명령이 사용 중인 카메라에 대해서는 거부됩니다. 모든 `CAMERA_HELD` 응답에는 `code`와 `error_code` 모두에 해당 코드가 포함됩니다. 모든 카메라를 여는 명령에 대해서는 [다중 카메라(일시적 세션)](#multi-camera-transient-sessions)을 참조하십시오.

##### `array-connect` 옵션

| 플래그 | 기본값 | 설명 |
| --- | --- | --- |
| `--serials SN1,SN2,…` | 모든 LATTICE 카메라 자동 탐색 (2대 이상 필요) | 첫 번째 시리얼이 MASTER가 됩니다. 생략할 경우, 탐색 대상이 LATTICE(M3C / M3M) 모델로 필터링되어 해당 모델의 모든 카메라가 연결됩니다. |
| `--line {Line0,Line2,Line3}` | `Line2` | GPIO 동기화 라인. |
| `--target-fps F` | 자동 | 마스터 트리거 발사 속도. |
| `--force-tier {sim-capture-sim-emit, sim-capture-ftd-stagger, slip-emit-and-capture}` | 자동 | 티어 선택기를 재정의합니다. |
| `--wire-ceiling-mbps MB_PER_S` | 자동 감지 | **호스트의 지속적 와이어 예산(MB/s) — 전체 어레이 할당이 이 수치에 따라 결정됩니다.** 어레이에서 GVSP 손상 프레임을 보고할 경우 이 값을 낮추십시오. 자동 값은 NIC가 알리는 링크 속도에서 파생되는데, 이는 USB 어댑터, 대역폭이 좁은 PCIe 레인 및 부하가 높은 공유 패브릭의 경우 실제보다 높게 표시됩니다. 이 값은 프로젝트의 어레이 캡처 블록에 영구적으로 저장되므로, 재열기/CLI/SDK를 통한 재연결 시 복원됩니다. [어레이 상태](#array-health--which-subsystem-is-losing-frames)를 참조하십시오. |
| `--binning {1,2,4}` | 없음 (전체 센서) | 하드웨어 비닝 (항상 평균). |
| `--width N` / `--height N` / `--pixel-format FMT` | 전체 센서; 카메라별 형식 | 전체 센서 및 카메라별 형식 선택(Mono12 / BayerRG12) 대신, 모든 카메라에 대해 명시적인 프레임 크기 또는 픽셀 형식을 지정합니다. |
| `--no-recommend` | 비활성화 | 네트워크 분석 단계 건너뛰기 (이 경우 각 카메라는 전체 센서로 설정되지 않고 현재 크기를 유지합니다). |
| `--no-ptp` | off | PTP 비활성화 (이 경우 카메라 간 타임스탬프 비교가 **불가능**합니다). |
| `--packed {auto,on,off}` | `auto` | 12비트 프레임을 압축하여 스트리밍하고 이 컴퓨터에서 압축을 해제합니다 — 전체 센서 모드에서 초당 프레임 수가 더 많으며, 데이터는 동일합니다. `auto`는 도움이 되고 이 컴퓨터가 처리할 여력이 있을 때만 압축합니다; `off`는 절대 압축하지 않습니다. [압축된 12비트 전송](#packed-12-bit-transport---packed)을 참조하십시오. |
| `--packet-size {auto,jumbo,standard,N}` | `auto` | 어레이 내 모든 카메라에 대한 GVSP 패킷 크기. `auto`는 각 카메라가 네트워크를 탐색하도록 허용합니다; `jumbo` = 9000, `standard` = 1500 또는 바이트 단위의 크기를 지정하면 고정됩니다. 1.3.0 이전 버전에서는 `array-connect`가 이 플래그를 수락했으나 무시했습니다. |

> **기본적으로 전체 센서 모드입니다.** `--width` / `--height` / `--binning` /
> `--pixel-format`, `array-connect`가 설정되지 않은 경우, 모든 카메라가 전체 센서 모드로 설정됩니다 —
> 비닝 없음, ROI 크롭 없음 — 이 컴퓨터와
> 네트워크가 제공하는 수준까지 프레임 속도를 낮추며, 프레임 크기를 줄여 속도를 유지하는 방식은 사용하지 않습니다.
> 네트워크 분석 줄에는 예상 속도가 표시되며, 다음 줄에는
> 더 빠른 옵션(비닝 또는 더 작은 ROI)과 그에 따른 예상 속도가 나열됩니다.
> `--binning` 또는 `--width`/`--height`를 선택하여 적용하십시오. Connect는 자체적으로 비닝이나 크롭을
> 수행하지 않습니다.

##### 압축된 12비트 전송 (`--packed`)

센서 전체를 사용할 때 어레이의 프레임 속도는 각 카메라가
프레임을 전송할 수 있는 속도에 따라 결정됩니다. 12비트 형식(`BayerRG12`, `Mono12`)은 일반적으로 16비트
워드 단위로 전송되지만, **압축** 전송 방식은 동일한 12비트를 픽셀당 1.5바이트로 전송하고
이 컴퓨터에서 이를 풀어서 처리하므로, 각 프레임을 전송하는 데 걸리는 시간이 25% 단축됩니다 —
**센서 전체 사용 시 초당 약 33% 더 많은 프레임** (측정 결과: 10.7 → 14.0 fps
9대 카메라 어레이 기준). **저장된 데이터는 동일합니다**: 동일한 값, 동일한
16비트 파일, 동일한 메타데이터입니다.

- `auto`(기본값)는 도움이 될 때만 **그리고** 이 컴퓨터가
  압축 해제 처리를 감당할 수 있을 때만 압축합니다. 연결 시 백엔드는 자체 압축 해제 속도, CPU 개수,
  현재 CPU 부하 및 네트워크 대역폭을 측정하므로, 여유 코어가 있는 데스크톱은
  풀 센서 모드에서 압축하여 전송하는 반면, 부하가 높거나 저전력 컴퓨터는 압축 해제된 상태로 스트리밍합니다. 작은 ROI와 비닝된 프레임의 경우 이득이
  이득이 거의 없습니다.
- `off`는 절대 압축하지 않습니다. 진단 시 압축을 배제하거나,
  다른 작업을 위해 CPU 자원을 모두 확보해야 하는 컴퓨터에서 사용하십시오.
- `on`는 항상 12비트 형식으로 압축합니다.

connect 출력에는 결정 사항과 그 이유가 표시됩니다(`12비트 압축: (auto:
예상 11.0 -> 14.4 fps, …)`); `/api/camera/array/status`는
`packed_transport` 및 `packed_decision`를 보고합니다. 매번 플래그를 전달하는 대신 시스템 전체의 기본값을
설정하려면, 백엔드 환경에서 `CHLOROS_ARRAY_PACKED12=off` (또는
`on`)를 백엔드 환경에 설정하십시오. 단, 명시적으로 지정된 `--packed on|off`가 우선 적용됩니다.
8비트 및 16비트 형식은 절대 패킹되지 않습니다.

### 하나의 프로젝트 레코드, 모든 모드 지원

프로젝트 폴더에는 모든 장치의 설정(`cameras.json` — 각 카메라의 설정(`cameras[].settings`, 단일 카메라 및 각 어레이 구성원별) 설정과 각 어레이의 전체 설정(`arrays[].capture`: 형식, ROI, 비닝, 트리거 속도, 와이어 예산) — 그리고 `sensors.json`, 각 광 센서별 설정(적분 시간, 프레임 평균, 캡)을 포함합니다. 앱, CLI, SDK 및 MCP 서버는 모두 동일한 파일을 동일한 방식으로 읽고 씁니다:

- **프로젝트에 연결된 장치는 해당 프로젝트에 저장되며, 모든 변경 사항을 프로젝트에 기록합니다.** "프로젝트 하위"란 백엔드(앱)에서 프로젝트가 열려 있는 상태를 의미하거나, SDK의 경우 자체 폴더 이름을 지정하는 `ChlorosProject`를 통해 연결된 상태를 의미합니다. 프로젝트에 아직 등록되지 않은 카메라, 어레이 또는 광 센서는 연결 시 해당 프로젝트의 파일에 추가됩니다 (`lattice cam-connect`, `array-connect`, `daq pool-connect`, 앱, SDK, `chloros-mcp`), 해당 설정과 함께 추가됩니다. 어떤 모드(`lattice exposure`, `gain`, `configure --set`, `resolution`, 어레이의설정 및 트리거 속도, 광 센서의 캡, 앱의 컨트롤)에 대한 변경 사항은 잠시 후 해당 프로젝트의 파일에 저장됩니다. 카메라에서 설정된 노출, 게인 또는 AE 목표값은 ‘유지’ 상태로 저장되므로, 프로젝트를 다시 열면 해당 설정이 다시 적용됩니다.
- **프로젝트를 열면 모든 모드에서 해당 프로젝트의 장치 설정이 적용됩니다**: 별도의 설정이 없는 카메라, 어레이 또는 광센서가 연결되어 있으면 프로젝트에 저장된 설정이 적용됩니다 (프로젝트가 열려 있는 상태의 `lattice cam-connect` / `array-connect`, `chloros-cli project connect`, `open_project().connect_all()`, 앱 재실행 시). 새로운 ID로 연결된 어레이는 프로젝트에 저장된 것과 동일한 카메라를 가진 어레이를 업데이트하며, 다른 카메라를 가진 어레이는 그 옆에 추가됩니다.
- **앱 자체의 저장 기능은 기기의 실시간 설정을** 이전 버전의 복사본으로 덮어쓰지 않으며, CLI 또는 SDK 호출자가 저장한 구성원별 재정의 사항을 유지합니다.
- **사용자가 설정한 픽셀 형식은 프로젝트에 그대로 유지됩니다.** 배열 설정(Array Settings), `--pixel-format` 또는 `connect_array(pixel_format=)`를 통해 설정된 형식(8비트 포함)은 정확히 저장되고 재연결 시 그대로 적용됩니다. 설정이 없는 경우, 각 카메라는 12비트 기본값(모노 센서의 경우 `BayerRG12`, `Mono12`)으로 연결됩니다. 형식 이름은 해당 비트 심도를 의미하며, 제품군은 센서의 것이므로 컬러 카메라는 절대 모노 형식으로, 모노 카메라는 절대 베이어 형식으로 설정되지 않습니다. 또한 형식이 매핑되었을 때 `[PixelFormat <serial>] ... written` 줄에 해당 내용이 표시됩니다. 8비트 캡처의 출력 데이터는 반사율 포함 8비트이며, 재조정되어 `PixelScale`로 표시됩니다.
- 프로젝트에 연결되지 않은 장치는 아무것도 기록하지 않습니다.
- **녹화는 그 후에 생성된 프로젝트를 따르며, 열기 시에는 기존 프로젝트를 유지합니다.**프로젝트가 생성될 때 — SDK의 `create_project()`, `chloros-mcp`의 `create_project`, 또는 앱의**새 프로젝트**— 또는 복제(앱의**복제**기능으로 복사본을 여는 경우)되는 동안, 녹화가 여전히 이전에 열려 있던 프로젝트에 기록 중이라면 녹화는 계속 진행되며, 기록된 내용은 새 프로젝트 내의 동일한 위치(아래)에 저장됩니다. 기존 프로젝트를 열 경우 — `chloros-cli process`, `project devices --reconnect`, `ChlorosProject.process()`, 앱의**프로젝트 열기** — 를 통해 기존 프로젝트를 열면, 녹화가 진행 중인 프로젝트는 그대로 유지됩니다. 녹화는 해당 프로젝트에서 완료되며, 생성된 파일은 해당 프로젝트의 파일 목록에 추가됩니다(앱은 프로젝트를 열기 전에 모든 녹화를 중지합니다).
  - `array-record` 및 `array-burst`: 녹화가 중지되면 동영상 또는 모든 프레임이 포함된 버스트 폴더가 전체적으로 해당 위치로 이동합니다(이때 `--duration`, Ctrl+C). 출력되는 `Burst dir`는 새로운 파일이며, `--build`와 `array-build-video`는 이를 기반으로 작동합니다. 드라이브가 이동을 거부하면 녹화 파일은 원래 쓰여진 위치에 그대로 남게 되며,끝 부분의 로그와 중지 시의 HTTP 응답(`relocation_error`)에 그 이유가 표시됩니다. 중지된 버스트의 원시 프레임과 `.daq`는 해당 버스트가 포함된 프로젝트의 파일 목록에 추가되지만, 비디오(`array-record`, 버스트로 생성된 비디오)는 프로젝트의 파일 목록에 추가되지 않습니다.
  - `array-capture --continuous`: 변경 후의 모든 캡처는 새 프로젝트에 저장되며, 변경 전에 저장된 캡처는 기존 위치에 그대로 유지됩니다.
  - `daq pool-record` 및 `project sensor log`(그리고 앱 자체의 광센서 녹화): `.daq`는 녹화가 중지되면 새로운 프로젝트의`light_sensor` 폴더로 이동하여 해당 프로젝트의 파일 목록에 추가됩니다.

  열려 있는 프로젝트 외부의 폴더(다른 곳에 위치한 `-o`)에 저장된 녹화 파일은 는 원래 위치에 그대로 남아 있으며, `lattice continuous`와 `multi-stream`는 지정된 폴더를 유지합니다.
- **앱 내에서 새 프로젝트를 생성하면 그 이후의 캡처는 해당 프로젝트에 저장됩니다.** 프로젝트를 생성하거나 복제해도 모든 캡처 실행 및 녹화는 계속 진행되며, 그 이후에 수행된 각 캡처(‘모두 캡처’, ‘캡처 시작’)는 새 프로젝트에 저장되어 해당 파일 목록에 추가됩니다. 앱에서 기존 프로젝트를 열면 먼저 모든 캡처가 중지됩니다.
- **프로젝트는 녹화 중에는 이동되지 않습니다.**열려 있는 프로젝트를 다른 작업 디렉터리로 이동하려는 시도(앱의**설정 → 작업 디렉터리**; CLI 명령어로는 프로젝트를 이동할 수 없으며, `/api/move-project`로 요청을 보내는 HTTP 클라이언트를 통해서만 이동 가능) 시, 어레이 녹화나 버스트, 어레이 연속 캡처, `lattice continuous` / `multi-stream` 작업 또는 광센서 녹화가 해당 디렉터리에 데이터를 기록하는 동안에는: `Project '<name>' is recording (<what>), so moving the project now is refused. Stop the recording, then try again.` 앱 역시 동일한 방식으로 거부하며, 폴더를 요청하기 전에, 앱 자체의 '모두 캡처', '캡처 시작' 또는 광센서 녹화가 실행 중일 때 동일한 방식으로 거부됩니다.

이는 연결 해제 시 카메라의 상태가 어떻게 유지되는지와는 별개의 문제입니다([카메라 메모리](#camera-memory----keep-settings----clear-settings) 참조): 프로젝트는 열릴 때 카메라 설정을 직접 설정합니다.

### 카메라 메모리 — `--keep-settings` / `--clear-settings`

어레이가 연결되면 해당 카메라들은 어레이의 설정을 따르며, 사용자는 이를 변경할 수 있습니다. Chloros가 카메라를 해제할 때 — `array-disconnect`, 일시적 명령의 종료 (배열이 없는 `multi-capture`, `multi-stream`, `multi-test`, `align-calibrate`, `align-apply`, `align-stream`, `index --live`), `lattice continuous`의 정지, 해당 카메라가 보유하지 않은 `lattice capture`의 종료 — 연결 전 상태로 각각을 복원합니다: 트리거 및 싱크 라인 배선, PTP 및 카메라 설정(전체 목록은 [다중 카메라(일시적 세션)](#multi-camera-transient-sessions)에 있음).

프로젝트 외부에서 카메라를 사용할 경우, 해당 명령어 중 두 가지 플래그를 사용하면 카메라의 **자체 메모리**에 설정이 기록됩니다. 이는 어떤 소프트웨어에서든 전원을 켤 때 적용되는 초기 설정입니다:

- **`--keep-settings`**는 세션 종료 시 각 카메라의 설정을 해당 카메라의 메모리(부팅 시 설정으로 사용되는 UserSet2)에 저장합니다. 카메라는 여전히 자유 실행 상태로 유지되며(`TriggerMode` `Off`), 동기화 라인 배선도 기존 상태를 유지합니다. UserSet2가 없는 카메라는 UserSet1에 저장되며, 이 컴퓨터가 이를 기억하므로 클로로스(Chloros) 어레이의 자가 복구 — 이는 Chloros의 기준선에서 벗어난 UserSet1을 재설정하지만 — 해당 설정은 그대로 둡니다.
- **`--clear-settings`**는 각 카메라의 메모리를 공장 출하 시 설정으로 재설정합니다: Chloros의 기준선, 즉 어레이 자가 복구 기능이 반영한 — `TriggerMode`, `Off`, `TriggerSource`, `Line2`를 통해 클로로스의 어레이 자가 복구 기능이 반영한 공장 기본값, 자동 노출, RGB 카메라의 보정된 화이트 밸런스는 유지됩니다 — UserSet1에 저장되어 부팅 세트로 설정됩니다.
- 이 두 가지는 결합할 수 없습니다. 각 카메라마다 한 줄씩 지정됩니다: `SN:…: settings saved to the camera's memory (UserSet2); it powers up with them.` 또는 `SN:…: camera memory reset to the factory settings.`; 응답인 `camera_memory`도 카메라별로 동일합니다.

이 앱은 이러한 플래그를 제공하지 않습니다.

**모든 종료 경로에서 카메라가 원래 상태로 복원됩니다.**위의 명령어뿐만 아니라 각 종료 경로에서 발견된 그대로 복원하는 작업이 수행됩니다: `cam-disconnect` (및 앱의 카메라**X**); 배열 또는 배열 구성원의 행에 있는 앱의**X**(배열은 구성원 없이는 실행될 수 없으므로 전체 배열이 해제됨); 앱 종료(백엔드에 최대 20초의 시간을 주어 모든 카메라, 배열 및 실행 중인 `lattice continuous`를 백엔드가 중지하기 전에 되돌릴 수 있도록 최대 20초의 시간을 부여합니다; 이때 여전히 실행 중인 명령 — 카메라를 열거나 PTP 클럭을 기다리는 일시적인 명령, 일회성 `lattice capture`, `power`, `white-balance` 또는 `presets --apply` — 다음 안전 지점에서 중지되며, 최대 8초 동안 카메라를 우선적으로 복원하고, HTTP 503 `SHUTTING_DOWN` 응답을 반환합니다. 여전히 실행 중인 연결 배열 — `lattice array-connect`, 즉 앱의 연결 —은 다음 카메라에 도달하기 전에 중지되고, 최대 10초 동안 열었던 카메라를 다시 닫은 후, HTTP 503 `SHUTTING_DOWN`를 반환합니다; 종료 중에 도착한 요청은 아무것도 열지 않습니다: HTTP 503 `SHUTTING_DOWN`); 그리고 강제 종료되거나 터미널이 닫힌 CLI 명령(SIGTERM / SIGHUP로 작업이 중지되고 — 그리고 카메라를 원래 상태로 되돌린 후 — 종료됩니다(`143` / `129`, Ctrl+C와 유사). `lattice continuous`, `multi-stream` 또는 `align-stream`가 SIGKILL 신호로 즉시 강제 종료된 경우(이 경우 아무것도 중지할 수 없음), 백엔드는 마지막 체크인 후 30초가 지난 시점에 해당 작업의 실행을 중지하고 카메라를 원래 상태로 되돌립니다 . 충돌하거나 강제 종료된 백엔드는 아무것도 복원할 수 없습니다. 전원을 껐다 켜면 각 카메라가 메모리 상태로 복귀하며, 다음 Chloros 연결 시 무장 상태로 감지된 카메라의 연결을 재설정합니다.**프로젝트는 카메라 설정을 자체적으로 설정합니다.** 프로젝트가 열린 상태에서 어레이가 연결되면, 각 구성원은 프로젝트가 저장해 둔 설정(노출, 게인 또는 AE 목표값 — 이는 사용자의 설정으로 간주되므로 자동 노출 기능은 이를 건드리지 않음 — 및 화이트 밸런스 모드 — Chloros의 보정; 카메라 자체의 화이트 밸런스는 비활성화됨)을 적용받으며, 이는 카메라 메모리에 저장된 내용과 무관합니다.

### 스마트 AE / 스마트 캡처

LATTICE 어레이는 연결되는 즉시 백그라운드에서 연속 AE를 실행하지만, 새로 조준된 장면은 수렴되는 데 잠시 시간이 걸립니다. `array-capture --smart`는 **편의를 위해 패키지화된 기능**입니다. 이 기능은 어레이 내 모든 카메라에서 AE가 안정화될 때까지 기다린 후 캡처를 트리거합니다. 세션 도중 장면을 변경할 때 사용하십시오.

```bash
# Connect once, then take settled captures whenever you re-point the rig
chloros-cli lattice array-connect --serials SN1,SN2,SN3,SN4
chloros-cli lattice array-capture --smart --processing reflectance -o pose_a/
# (move the rig)
chloros-cli lattice array-capture --smart --processing reflectance -o pose_b/
```

안정화 정책은 기본적으로 보수적으로 설정되어 있습니다: 타임아웃 5초, 안정성 창 1.5초, 노출 편차 허용 오차 ±5%. 자동화 설정과 다른 동작이 필요한 경우 SDK(`ArrayHandle.capture_smart(settle_timeout_s=…, stability_window_s=…, exposure_tolerance_pct=…)`)를 통해 조정하십시오.

#### 자동 노출의 작동 방식

- **노출을 먼저, 게인을 나중에 처리합니다.** 게인이 ‘자동’으로 설정된 경우, 노출이 허용되는 최대값에 도달할 때까지 게인은 0 dB로 유지됩니다.
- **수동 노출을 설정해도 게인은 '자동' 상태로 유지됩니다**. 사용자가 게인을 변경하거나 수동으로 설정할 때까지 이 상태가 지속됩니다. 다시 '자동'으로 전환하면 게인의 제한값은 이전과 동일해집니다.
- **작은 하이라이트는 장면의 밝기를 어둡게 만들지 않습니다.** 프레임의 최대 1%에 해당하는 스페큘러 하이라이트는 무시되며, 알려진 보정 타겟 패널의 경우 패널 면적의 최대 0.5%까지 무시됩니다.
- **DAQ 노출 상한은 보정 타겟이 화면에 표시되는 동안에만 적용됩니다.** 이는 흰색 패널이 클리핑되는 것을 방지하며, 패널이 보이지 않을 때는 카메라는 장면의 노출을 조절합니다.
- **일정한 조명은 안정적으로 유지되며, 변화하는 조명은 몇 초 내에 안정화됩니다.** 일정한 조명에서는 노출과 그 상한이 고정되며, 조명이 크게 변한 후에도 화면은 몇 초 내에 안정화됩니다. 클리핑으로 인해 노출이 영구적으로 제한되는 일은 없으며, 관심 영역(ROI)이 변경되어도 노출이 다시 조정됩니다.
- **밝기 목표치는 조절 가능한 범위 내에 유지됩니다.** 카메라가 노출 한계에 도달해 게인을 더 이상 높일 수 없는 상태에서는 밝기 목표치가 상승하지 않으므로, 빛이 다시 들어와도 프레임에 플레어가 발생하지 않습니다.
- **자동 노출이 반응하지 않는 카메라는 현재 조도 수준에 맞춰 유지됩니다.** 강렬한 햇빛 아래에서 카메라는 최단 노출 시간으로 떨어질 수 있으며, 모든 설정이 정상으로 표시된 채 어두운 프레임을 유지한 상태로 머물 수 있습니다. 모든 설정 값은 정상으로 표시될 수 있습니다. Chloros는 프레임을 모니터링하여 해당 조명에 맞춰 유지되던 수준으로 카메라를 고정하고, 조명이 변하면 이를 해제합니다. 빛 센서가 어두워지거나 가려져도 이 과정은 방해받지 않습니다.

### 캡처 내보내기 레벨 (`all` 기본값)

이번 릴리스(`lattice capture`)부터, `lattice multi-capture` 및 `lattice array-capture`는 **기본값이 `--processing all`*로 설정됩니다.* — 각 카메라에 적용되는 내보내기 유형별로 하나의 저장 파일이 생성되며, 이는 GUI의 “모두 캡처(Capture All)” 동작과 일치합니다. 레벨은 다음과 같습니다:

| 레벨 | 출력 | 적용 대상 |
| --- | --- | --- |
| `raw` | 센서에서 직접 출력되는 단일 채널 베이어(단색 카메라의 경우: 단일 대역). | 모든 카메라. |
| `debayered` | 3채널 컬러, **RGB**채널 순서로 저장됨(TIFF는 RGB 광도 해석을 적용하며, PNG/JPEG도 마찬가지로 RGB로 열림): 원본 데이터에서 보정 팩의 다크 오프셋(DSNU)을 제거한 후, OpenCV의 에지인식 디모자이크 적용 — 선형성 LUT, 플랫 필드 또는 언믹스 처리 없음 (모노 카메라: 1채널 그레이스케일). Python SDK가 반환하는 메모리 내 디베이어링 처리된 프레임(`grab()`, `frame_stream()`)은**BGR**, 즉 OpenCV의 순서이며; 저장된 파일만 RGB 형식입니다. | 모든 카메라. |
| `radiance` | 전체 방사계측 체인을 통한 float32 W/m²/sr/nm. | 다중 스펙트럼(M3C/M3M) 전용 — **RGB 필터 카메라의 경우 RGB 필터 카메라의 경우**생략**. |
| `reflectance` | uint16 ρ (`32768` = 1.0), Pix4D 호환. | 다중 스펙트럼 전용이며, **DAQ가 바인딩되어 있고 카메라가 보정된 경우에만** 적용됩니다. 그렇지 않으면 건너뜁니다. |
| `display` | 전체 GUI 미리보기 체인 (카메라 프로필에 따른 CCM + WB + 감마). `preview/` 폴더에 저장됩니다. `lattice capture`는 이 폴더에 대해 `preview`라는 이름도 허용합니다; `array-capture` / `multi-capture`는 `display`만 허용합니다. | 모든 카메라. |

단일 레벨을 지정하여 해당 레벨만 저장합니다(`--processing debayered`). `all`를 요청할 경우, 지정된 캠에 적용되지 않는 레벨은 건너뛰어지며 (보고됨)되며, 오류로 처리되지 않습니다. 연결되지 않았거나 보정되지 않은 카메라도 여전히 `raw` / `debayered` / `preview`를 반환합니다.

#### 캡처를 읽는 DAQ 저장

DAQ가 카메라나 어레이에 바인딩되거나 풀에 등록될 때마다, 캡처에 사용된 하향 측정값은 캡처 폴더의 `daq/` 하위 폴더에 있는 **`.daq`** 파일에 기록되며, 이 파일에는 `daq/` 하위 폴더에 저장됩니다. 또한 `.match.json` (측정값이 노출 시점으로부터 얼마나 지났는지) 및 `.cameras.json` (해당 측정값의 소유자)가 포함되며, `DAQ:` 줄에 보고됩니다. 이는 모든 레벨(RAW 또는 디베이어링된 캡처 포함)에 적용되므로, 어떤 캡처든 나중에 방사도, 반사도 또는 인덱스로 재처리할 수 있습니다. GUI, CLI 또는 SDK에서 캡처한 것이든 상관없이, 파일 이름과 사이드카가 동일한 동일한 파일입니다. DAQ 판독값이 전혀 없는 경우 `.daq`는 없으며, 연결된 어레이가 없는 `multi-capture` (과도적 대체 처리)인 `multi-capture`는 아무것도 저장하지 않습니다.

| 플래그 | 효과 |
| --- | --- |
| `--no-daq` | 해당 레벨에서 이를 사용하지 않는 캡처의 판독값을 저장하지 않음 (`raw`, `debayered`). `radiance`, `reflectance` 또는 `display` 레벨은 여전히 해당 값을 기록합니다. 재가져오기 시 해당 제품이 생성될 때 사용된 측정값이 필요하기 때문입니다. |
| `--daq-tolerance-ms MS` | 허용 오차 범위 내로 간주되기 위해 측정값이 노출값으로부터 얼마나 떨어져 있어도 되는지(기본값 500). 판정 결과만 달라집니다 (`.match.json`의 `in_tolerance` 및 `tolerance_ms`, 반사율의 `DlsInTolerance`), 사용된 측정값 자체는 절대 변경되지 않습니다. [노출값과 크게 다른 DAQ 측정값](#a-daq-reading-far-from-the-exposure)을 참조하십시오. |
| `--force-daq` | RAW 전용 캡처에서 `.daq`를 요청하는 구식 방식입니다. 현재는 이 방식이 기본값이며, `lattice capture`와 `array-capture`는 여전히 이 플래그를 허용합니다(`multi-capture`는 허용하지 않음). `--no-daq`가 이보다 우선합니다. |

어떤 DAQ를 사용할지: 카메라나 어레이에 바인딩된 것(GUI의 광센서 선택, 프로젝트에 저장된 바인딩, 또는 `chloros-cli daq pool-bind`)인지, 아니면 백엔드에 있는 가장 최신의 것 중 하나인지. 원시 버스트 (`array-burst`)는 이전과 마찬가지로 버스트 폴더 내에서 각 개별 판독값마다 고유한 `.daq`를 유지합니다.

**PNG와 JPEG는 채널당 8비트를 저장하므로, `display` / `preview` 및 인덱스 산출물만 사용합니다.** `--format png` 또는 `jpg` 중 해당 형식이 처리할 수 없는 레벨을 가진 파일은 캡처되기 전에 거부되며, 해당 레벨과 이를 처리할 수 있는 형식인 TIFF를 명시합니다:

- `radiance` (float32) 및 `reflectance` (uint16, `32768` = 1.0) — 항상 그렇기 때문에, `--processing all`(기본값)와 함께 `--format png`/`jpg`도 거부됩니다: `--format tiff` 또는 `--levels display`를 사용하십시오.
- 8비트 이상을 스트리밍하는 카메라의 `raw` 및 `debayered` (12비트 M3C / Mono12 M3M 프레임) — CLI는 카메라의 비트 심도를 인식할 수 없으므로 백엔드에서 이를 거부합니다(HTTP 400). 단일-프레임인 `lattice capture --processing debayered`는 PNG/JPEG용으로 8비트 프레임을 캡처하므로 거부되지 않으며, 지속적 배열이 없는 `multi-capture`(8비트 디베이어 처리)도 마찬가지입니다.

모든 캡처 경로는 다음과 같이 동일하게 응답합니다 (`error_code`, `FORMAT_CANNOT_HOLD_LEVEL`): `lattice capture`, `multi-capture`, `array-capture`, `project capture` / `burst`, 그리고 그 뒤에 있는 HTTP 경로들.

### 캡처 폴더의 구조

각 내보내기 유형은 `-o` 아래의 **별도의 하위 폴더**에 저장되므로, 다단계 캡처에서도 유형이 혼합되는 일은 없습니다. 이 스케치는**`all`가 `lattice capture`를 통해 한 대의 카메라를 캡처한 기본 구성**을 보여줍니다. 카메라당 여러 단계가 있으므로, 모든 파일 이름에는 `_<level>` 접미사가 붙습니다:

```
output/
├── raw/           capture_<ts>_SN<serial>-<filter>_raw.tiff
├── debayered/     capture_<ts>_SN<serial>-<filter>_debayered.tiff
├── radiance/      capture_<ts>_SN<serial>-<filter>_radiance.tiff
├── reflectance/   capture_<ts>_SN<serial>-<filter>_reflectance.tiff
├── preview/       capture_<ts>_SN<serial>-<filter>_display.tiff
├── index/         per-camera vegetation-index (LUT) render, when --index is on
├── composite/     array foreground/background live-view composite, when produced
├── clamp_masks/   radiance/…_radiance_clampmask.tif, reflectance/…, index/… — the unmix clamp
│                  masks, one subfolder per product folder, only when a band clamped
└── daq/           daq_data_<sensor id>_<ts>.daq: the downwelling reading matched to the capture
                   (+ .daq.cameras.json: whose)
```

파일 이름의 변형 방식:

- **카메라 필드 — 일련번호, 그 다음 필터:** 모든 캡처 파일은 촬영한 카메라의 이름을 `SN<serial>-<filter>` 형식으로 표기하며, 여기서 `<filter>`는 카메라 모델의 필터 부분입니다: `SN261902142-F450`는 `M3M-L87-F450`에 해당하며, `M3M-L87-F450`의 경우 `SN213800234-FRGN`, `M3C-L87-FRGN`의 경우 `SN214701292-FRGB`, RGB 카메라의 경우 `-`, `_`와는 연결되지 않으므로, 이름은 이전과 마찬가지로 `_`로 구분된 필드(접두사, 날짜, 시간, 밀리초, 카메라, 레벨)를 갖습니다. 모델명을 읽을 수 없는 카메라(DeviceUserID가 삭제된 경우)는 `SN<serial>`로만 이름이 지정됩니다. 이전 릴리스에서 캡처된 영상에는 이름에 필터가 포함되지 않으며, Chloros는 두 가지 모두를 읽습니다.
- **접두사:** `lattice capture`는 파일을 `capture_…`로 명명하며, 연결된 배열이 있는 경우 `array-capture`, `multi-capture`로, 배열 이름의 `project capture` / `burst` / `stream`와 같은 배열 이름의 파일은 `sync_<ts>_SN<serial>-<filter>…`로 명명됩니다.
- **레벨 접미사는 여러 레벨이 있는 경우에만 사용됩니다.**한 번의 캡처에**여러**레벨을 저장하는 카메라(`all` 또는 `--levels a,b`)는 `_<level>`를 추가합니다.**하나** 레벨만 저장하는 카메라는 접미사 없이 기본 이름을 유지합니다(예: `--processing reflectance`는 `reflectance/sync_<ts>_SN<serial>-<filter>.tiff`로 기록됨). 따라서 단일 레벨 캡처를 대상으로 작성된 스크립트는 계속 작동합니다. 레벨은 여전히 폴더 이름으로 표시됩니다.
- **클램프 마스크는 제품 이름 뒤에 붙습니다:** `<product name>_clampmask.tif`. 단일 레벨 반사율 캡처는 `clamp_masks/reflectance/sync_<ts>_SN<serial>-<filter>_clampmask.tif`를 기록하고, `all` 캡처는 `clamp_masks/reflectance/sync_<ts>_SN<serial>-<filter>_reflectance_clampmask.tif`를 기록합니다.
- **확장자**은 `--format` 뒤에 붙습니다. 기본값인 `tiff`는 `.tiff`를, `--format tif`는 `.tif`가 됩니다(클램프 마스크는 항상 `.tif`입니다).
- **단 한 가지 예외:** `lattice capture --processing debayered`(또는 `--levels debayered`)는 빠른 단일프레임 경로를 취하며, `capture_<ts>_SN<serial>-<filter>.tiff`를 하위 폴더 없이 `-o`에 직접 기록합니다.
- **스트림 및 버스트**는 동일한 필드를 전달합니다: `lattice continuous`는 `frame_<seq>_SN<serial>-<filter>.tiff`, `lattice multi-stream`, `stream_<seq>_SN<serial>-<filter>_<frame id>.tiff` 및 원시 버스트 파일인 `<burst>_f<frame>_SN<serial>-<filter>_raw.tiff`를 기록합니다.

`<ts>`는 캡처 타임스탬프이고 `<serial>`는 카메라 일련번호이므로, 하나의 동기화된 그룹은
카메라 간에 타임스탬프를 공유합니다. **한 가지 비대칭적인 점이 있습니다:** `display` 레벨은
`preview/`라는 이름의 폴더에 저장되는 반면, 다단계 파일의 경우 이름에 `_display`가 포함되지만 — 해당 레벨에서만 폴더와 접미사가 다릅니다.
알 수 없는 레벨은 파일 이름과 동일한 폴더로 처리되며, 하위 폴더를
생성할 수 없는 경우 파일이 손실되지 않고 출력 루트에 기록됩니다.

**캡처 폴더 재처리:**`chloros-cli process`를**캡처 루트**
(`output/`)로 지정하십시오. `process`는 일반적으로 지정한 폴더만 가져오지만, 해당 폴더에
이미지가 없고 하위 폴더가 있는 경우 자동으로 하위 폴더로 내려갑니다. 따라서 해당 레벨의 하위 폴더와
`daq/` 하위 폴더가 한 번에 모두 가져옵니다. 캡처의 각 레벨은 레벨당 하나의 이미지로 가져오는 대신,
단일 이미지로 가져오며 다른 레벨은 모드로 사용할 수 있습니다.

**레벨 하위 폴더**를 직접 지정(예: `output/raw/`)하는 방법도 가능합니다. 이렇게 하면 `daq/`가
남게 되므로, `raw/`에서 방사측정
산출물을 도출할 때 DAQ 판독값을 함께 복사하거나 지정해 주십시오`raw/`에서 방사계측
산출물을 도출할 때는 해당 파일을 복사하거나 경로를 지정해야 합니다. 그렇지 않으면 타임스탬프 일치 정보를 참조할 대상이 없습니다.

**처리는 항상 `raw`에서 시작됩니다.** 각 캡처 내에서 원시 프레임이 파이프라인의 소스이며;
`debayered`, `radiance`, `reflectance` 및 `preview`는 표시 가능한 모드로 제공되지만, 파이프라인을 통해
다시 처리되지는 않습니다. 파생된 제품을 재처리하면 이미 픽셀에 베이킹된 비네트, CCM 및
방사도 계산이 다시 적용되므로, Chloros는 이중 처리를 피하기 위해
이를 거부합니다. 알아두어야 할 두 가지 결과:

- `index/` 및 `composite/` 렌더링은 **절대** 처리되지 않습니다. 이들은 캡처가 아닌 출력물이며 —
  NDVI LUT 렌더링에는 의미 있는 방사도 해석이 없습니다.
- `raw`가 **포함되지 않은** 상태로 내보낸 캡처 폴더(예: `array-capture --processing reflectance`)는
  유효한 파이프라인 소스가 없습니다. 이러한 캡처 파일은 정상적으로 가져오고 표시되지만, `process`는
  이를 건너뛰며 다음과 같이 표시합니다:

  ```
  [IMPORT-LEVEL] Skipping 4 already-processed file(s) with no raw source: capture_…_reflectance.tif
  [IMPORT-LEVEL] Processing starts from raw. Re-capture with --processing raw, or force an entry
                 point with --input-level.
  ```

  파생 제품을 반드시 처리해야 하는 경우 —
  `demosaic`가 활성화된 상태에서 캡처된 허브 세션이나 레거시 폴더 등 — `--input-level {raw,debayered,processed}`는 진입점을
  강제로 지정하여 건너뛰기 설정을 무시합니다. 이 플래그는 의도적으로 마련된 비상 탈출구이며, `auto` (기본값)
  는 원시 데이터가 없는 캡처를 절대 처리하지 않습니다.

**보정 태그: `raw`에서만 적용됩니다.** 캡처의 `raw` 파일에는 카메라의 보정 패킷이 포함됩니다: `CalibrationJSON` 페이로드와 MicaSense 호환 태그(`BlackLevel`, `RadiometricCalibration`, `Band{R,G,B}RadiometricCalibration`, `MicaSenseVignetting*`)로, 이는 센서의 원시 값을 설명합니다. 그 외 모든 레벨의 파일(`debayered`, `display`, `radiance`, `reflectance`, `index/` 렌더링) 및 모든 `lattice continuous` / `multi-stream` 프레임은 카메라의 식별 정보(`XMP-mapir:CameraSerial`, `CameraModel`, `FilterType`, `LensModel`, `CalBundleSha256`, ...)와 프레임 자체의 기록인 `XMP-mapir:FrameJSON`(노출, 게인, 온도, 픽셀 형식, 촬영 시간, DAQ 일치)를 포함하며, 모든 `process` 제품에는 고유 식별자가 포함되어 있습니다. `process`는 일련번호를 통해 해당 파일의 보정 정보를 찾아내고, `CalBundleSha256`는 Chloros의 보정 캐시에서 이를 확인하며, 컴퓨터에 해당 정보가 없을 경우 컴퓨터에 해당 파일이 없을 경우 해당 패키지를 한 번만 다운로드합니다.

#### 여러 대의 카메라로 생성된 파일

하나 이상의 카메라 밴드로 생성된 파일 — 멀티밴드 스택(`align-apply`, `align-stream`, `index --live --save-multiband`), 여러 대의 카메라로 생성된 라이브 인덱스(`index.tif`, `index_LUT.png`), 어레이 캡처의 `composite/` 이미지 — 이 파일에는 해당 파일에 사용된 모든 카메라의 이름이 명시됩니다:

- **카메라별 목록, 밴드 순서대로** (합성 이미지: 배경 레이어, 그 다음 전경; 인덱스 레이어, 해당 밴드의 출처가 된 카메라): `XMP-mapir:SourceCameraSerials`, `SourceCameraModels`, `SourceSensorProfiles`, `SourceProductLines`, `SourceFilterTypes`, `SourceLensModels`, `SourceCalDateUTCs`, `SourceCalBundleSha256s` (각 카메라의 밴드가 처리된 보정 팩, 그렇지 않은 경우 — `align-apply` / `align-stream` 스택(이 스택의 밴드는 보정 과정을 거치지 않음) — 카메라의 팩인 Chloros가 캐시한 것, 즉 해당 카메라의 캡처 파일 이름에 사용된 팩), `SourceLightboxCertIds`, `SourceHalogenLampCertIds`, `SourcePackTiers` — 병렬 목록으로, 각 목록의 *n*번째 항목은 동일한 카메라를 설명합니다; `-`는 카메라에 값이 없는 경우(보정 팩 없음, 알 수 없는 렌즈)를 나타냅니다. 각 카메라는 **하나**의 항목으로 구성됩니다. 팩에 필드당 여러 값이 있는 경우(예: 팩의 할로겐 램프 인증서)는 `;`(`F-1479-1;F-1480-1;F-1481-1`)로 연결되며, 쉼표는 절대 사용되지 않습니다. 따라서 목록은 출력 방식에 관계없이 카메라당 하나의 항목으로 표시됩니다.
- **EXIF `SerialNumber`** 에는 쉼표로 연결된 동일한 일련번호가 포함되어 있습니다(`214000535,214000551`).
- 이 항목에는 **단일**카메라 식별 정보가**없습니다**(`XMP-mapir:CameraSerial`, `CalBundleSha256`, …): 이 식별자들은 개별 카메라와 해당 팩을 지칭하며, `process`는 이를 바탕으로 파일의 보정 정보를 도출합니다. 여러 카메라의 밴드(`index.tif`, `index_LUT.png`)로 계산된 인덱스는 EXIF에 `Model` `LATTICE` (리그)를 나타내며, 단일 카메라의 `LATT-…` 모델은 아닙니다.**단일** 카메라의 밴드로 구성된 파일 — 밴드별 TIFF(`--per-band`), `align-apply --per-camera` 프레임, 단일 카메라의 스택 또는 라이브 인덱스 —는 다른 모든 캡처 파일과 마찬가지로 해당 카메라 고유의 식별 태그를 포함합니다. `align-apply`의 스택, 밴드별 및 카메라별 파일, 그리고 각 `align-stream` 스택과 그 밴드별 파일은 해당 그룹의 캡처 시간(참조 카메라의 노출 시간, UTC)을 포함하며, 또한 모노 카메라의 `--per-camera` 프레임은 스택과 마찬가지로 `--bit-depth`에서 하나의 평면을 이룹니다.**카메라 방향(반전/회전).** 카메라의 방향 설정(GUI의 Mirror H /
Mirror V / 회전)은 캡처 시 저장되는 내용을 결정합니다:

- **`raw`와 `debayered`는 설정과 상관없이 센서가 읽은 그대로 정확히 저장됩니다**.
  베이어 모자이크가 회전되거나 뒤집히면 색상 필터 패턴이 달라지며, 보정
  맵(다크 오프셋, 플랫 필드)은 센서 자체의 그리드에 맞춰져 있습니다.
- **`preview`/`display`, `radiance`, `reflectance` 및 `index/` 렌더링 파일은 방향이 조정되어 있습니다** — 모든 보정 계산이 끝난 후
  한 번 회전되어, 카메라가 프레임을 잡은 그대로 열립니다. 이
  `clamp_masks/…_clampmask.tif` 파일들은 동일한 그리드에 위치합니다.
- **모든 캡처 파일은 XMP에 설정 정보를 기록합니다**: `XMP-Camera:ChlorosFlipHorizontal` /
  `ChlorosFlipVertical` (`true`/`false`), `ChlorosRotation` (0 / 90 / 180 / 270, 시계 방향,
  반전 처리 후) 및 `ChlorosOrientationApplied` — `false`는 원본 및 디베이어 처리된(네이티브 픽셀) 이미지에,
  방향 보정된 결과물에서는 `true`. `ChlorosProcessing`는 해당 파일의
  픽셀에 적용된 연산들을 나열합니다(네이티브일 경우 `none`). 여기에는 `lattice continuous`가 스트리밍하는 모든 프레임(
  카메라 자체 설정, 적용되지 않음)과 `lattice multi-stream`의 모든 프레임(정체성: 그
  구성 프레임은 배열 캡처와 마찬가지로 네이티브임)이 포함됩니다. `lattice align-stream` / `align-apply`
  스택은 마스터 카메라의 설정을 적용되지 않은 상태로 기록합니다. 이 스택의 밴드는 마스터의
  프레임으로 워핑되지만 회전되지는 않습니다.
- **`process`는 원시 데이터를 처리할 때 기록된 방향을 적용합니다**: 먼저 센서 좌표계에서 다크 오프셋, 플랫
  필드, 디모자이크 및 언믹스를 수행하고, 다음으로 어레이 정렬 워프를 적용한 뒤,
  방향 조정을 마지막에 수행합니다. 따라서 미리보기, 방사도, 반사도 및 지수 산출값은
  실시간 캡처 결과와 정확히 동일하게 나오며, `ChlorosOrientationApplied=true`로 기록됩니다. 보정되지 않은 대체
  산출물(`Sensor_Response_Images`, `Vignette_Corrected_Images`)은
  플랫 필드 처리 후 동일한 방향으로 정렬됩니다. 디베이어링된 산출물은 원본 상태를 유지합니다.
- **Chloros 이미지 뷰어는 기록 순서에 따라 회전된 원본 또는 디베이어링된 파일을 표시합니다** (표시용일 뿐 —
  파일 자체는 변경되지 않음), 따라서 산출물과 동일한 모양으로 보입니다; 썸네일, 전체 크기 보기,
  커서 판독값 및 인덱스/LUT 샌드박스 모두 동일한 회전 각도를 적용합니다. 레코드가 없는 파일과
  (이미 방향이 정렬된) 결과물은 저장된 그대로 표시됩니다. 샌드박스 내보내기 시 픽셀이 지닌
  방향 정보가 기록됩니다(`ChlorosOrientationApplied=true`; 방향 정보가 없는 레이어의 식별자).
- **레코드가 없는 캡처는 네이티브로 처리됩니다** (변경되지 않은 동작). 1.3.0 이전에는
  플립이나 회전이 설정된 단일 카메라 캡처의 경우, 원본 및 디베이어링된 파일이 자동으로 회전되었으나;
  Chloros는 이미-회전된 원본 파일을 네이티브 파일과 구별할 수 없으며 추측하지도 않으므로,
  저장된 그대로 처리합니다. 보정 맵은 센서 자체의 그리드에 있으므로, 보정이 중요한 경우 이러한
  캡처 대신 1.3.0 버전으로 촬영한 파일을 사용하십시오.
- **어레이는 기본적으로 구성 파일의 방향을 회전시키지 않습니다** (정렬 워프 및 `Alignment*` XMP는
  카메라 고유의, 회전되지 않은 프레임에 있습니다)이며, 동일성을 기록합니다. `array-capture --orient`는 정렬 워프 후 각
  멤버의 산출물을 회전시킵니다: 멤버들이
  마스터 프레임으로 워프되었을 때의 **마스터** 방향에 따라(이렇게 하면 멤버들이 서로 정렬된 상태를 유지함), 그렇지 않은 경우 해당 멤버 카메라의 방향에 따라 회전합니다. 각
  파일은 해당 파일의 픽셀이 속한 프레임의 방향(원본 파일의 경우 해당 카메라의 방향)을 기록하며,
  정렬 변환 정보를 포함하는 원본 파일은 `ChlorosAlignedFlipHorizontal` /
  `ChlorosAlignedFlipVertical` / `ChlorosAlignedRotation`: 마스터의 프레임으로 워프된 결과물의 경우 마스터의 방향
  마스터 프레임으로 왜곡된 결과물의 경우). `process`는 각 결과물을 해당 픽셀에 해당하는 값만큼 회전시킵니다:
  RAW 파일을 왜곡할 때는 정렬된 값(`--array-alignment`, 기본값 — 실시간 정렬
  캡처의 결과물), 정렬되지 않은 경우 카메라 자체의 픽셀에 따라 각 결과물을 회전시킵니다(`--no-array-alignment` — 라이브
  정렬되지 않은 캡처의 결과물). `array-burst`, `project capture` / `burst` /
  `stream`, 레시피 및 SDK에서도 동일한 옵션이 제공됩니다.

### 혼합 필터 어레이에서 건너뛴 캡처

RGB 및 다중 스펙트럼 카메라를 하나의 어레이에 혼합하여 사용할 경우, `array-capture --processing radiance`(또는 `reflectance`)는 다중 스펙트럼 프레임을 저장하고 **RGB 카메라의 캡처를 건너뜁니다** RGB 카메라의 캡처를 건너뜁니다. 광대역 센서의 경우 베이어(Bayer) 단위당 복사도는 의미가 없기 때문입니다. CLI는 저장된 각 파일(내보내기 수준 포함), 작성된 각 `.daq`, 그리고 각 건너뜀을 명시적으로 출력하므로 파일 수가 예상치 못한 것은 아닙니다:

```
  Saved: …/output/reflectance/sync_…_SN213800234-FRGN.tiff [reflectance] (SN:213800234, fid:1)
  Saved: …/output/reflectance/sync_…_SN214000533-FOCN.tiff [reflectance] (SN:214000533, fid:1)
  Saved: …/output/reflectance/sync_…_SN214701288-FNGB.tiff [reflectance] (SN:214701288, fid:1)
  DAQ:   …/output/daq/daq_data_daq-e-54b5e0_….daq
  Skipped: SN:214701292 (reflectance-not-applicable-to-rgb-cam filter=RGB)

  3 synchronized frames captured. (1 skipped)
```

건너뜀 사유 토큰은 `<level>-not-applicable-to-rgb-cam` 형식을 따릅니다. 보정 팩이 로드되지 않은 카메라는 `radiance` / `reflectance`를 건너뛰고, `<level>-needs-calibration (connect the camera so its cal pack loads)`로 건너뛰며, 사용 가능한 DAQ 반사율 측정값이 없을 경우 `reflectance-skipped-no-dls`로 건너뜁니다. 반사율은 또한 `reflectance-skipped-no-fresh-dls` / `reflectance-skipped-bound-daq-unavailable (…)`로 건너뛸 수 있으며, 대역이 DAQ 광 센서의 방사계 보정 범위(~374–974 nm) 밖으로 대부분 벗어날 경우 `dls-uncalibrated-band-<nm>`에서도 반사율 측정값이 누락될 수 있습니다. 출하 중인 SKU 중에서는 F988만이 반사율 패널 워크플로우를 지원합니다.

필터 유형에 관계없이 모든 카메라를 포함하려면 `--processing debayered`(또는 `display`)를 사용하거나, 카메라별로 적용 가능한 모든 레벨을 한 번에 얻으려면 기본값인 `all`를 사용하십시오.

### 노출과 거리가 먼 DAQ 측정값

캡처가 저장하는 각 `.daq`는 해당 노출과 일치하는 DAQ 측정값이며, 그 옆에 있는 `.match.json`는 두 값 간의 차이를 기록합니다 (`gap_ms`)이며, 해당 차이가 허용 오차 범위 내인지(`in_tolerance`, 캡처 세트에서 `--daq-tolerance-ms`로 설정되지 않은 경우 500 ms) 여부도 기록합니다. 허용 오차 범위를 벗어난 측정값도 여전히 사용되므로 프레임이 손실되지 않으며, 이에 대한 라벨이 부여됩니다. 즉, 반사율 파일의 XMP에는 `DlsInTolerance=False` 및 `DlsMatchGapS`가 포함됩니다. `lattice capture`, `multi-capture` 및 `array-capture`도 해당 `.daq` 항목 아래에 이를 출력하며, 설명할 때 DAQ'스트림 오류가 발생합니다. 이를 설명하면 다음과 같습니다:

**측정값이 노출과 어떻게 매칭되는지.** 캡처는 노출 후 DAQ의 첫 번째 측정값을 기다리므로, 그 전후의 측정값을 혼합할 수 있습니다 (`method` `interpolated`). DAQ-E는 일광 하에서는 0.4초마다, 실내에서는 1~2초마다 측정값을 전송하므로, 이 대기 시간은 노출 시간이 아닌 명령 실행 시간에 해당 간격의 약 1.5배가 추가됩니다. 전송이 중단된 DAQ는 대기하지 않습니다. DAQ-E 측정값은 이전 측정 이후의 간격 동안 평균화된 빛의 값이므로, 해당 간격의 중간 지점에 배치됩니다. 즉, `gap_ms`는 노출 시작 시점부터 그 중간 지점까지의 시간을 나타내며, `bracket_stamps_ns`는 측정값 자체의 타임스탬프를 나타냅니다. `array-capture --continuous`는 캡처 속도를 저하시키지 않고 대기합니다. 원본 파일은 즉시 기록되며, 판독값이 도착하면 `.daq`가 기록됩니다. `--interval`의 경우, 각 패스는 단일 캡처와 마찬가지로 대기합니다.

```
  DAQ:   …/output/daq/daq_data_daq-e-54b774_….daq
         DAQ reading 114.9 s from the exposure (tolerance 500 ms): reflectance uses it, labelled DlsInTolerance=False
         DAQ stream fault: no frame accepted from daq-e-54b774.local in 25 s, after 1932 good frame(s). …
```

해당 판독값으로부터 반사율 파일이 기록되지 않은 경우, 첫 번째 줄은 대신 `saved anyway; its .match.json says in_tolerance: false`로 끝납니다.

캡처는 여전히 `0`로 종료됩니다: 모든 파일이 기록되었으며, 파일에 포함된 내용을 나타냅니다. 스트림이 중단된 DAQ는 마지막 판독값을 계속 제공하므로, 해당 캡처의 반사율을 신뢰하기 전에 `chloros-cli daq pool-list`로 확인하십시오. 백엔드는 데이터 전송이 중단된 풀링된 DAQ-E를 다시 연결합니다 ([데이터 전송이 중단된 DAQ-E](#a-daq-e-that-stops-delivering) 참조). 허용 오차 범위를 벗어난 측정값은 (아래의) 유지된 측정값이 아닌 한, 해당 `DAQ:` 행만 출력됩니다.

**DAQ가 중단된 후에도 유지된 측정값.** 특정 노출에 대해 측정값을 제공할 수 있는 DAQ가 없는 경우(바인딩된 DAQ를 사용할 수 없거나, 풀링된 DAQ 중 해당 측정값을 가진 것이 없는 경우, 예: `daq pool-disconnect` 이후), 캡처는 Chloros가 가지고 있던 마지막 측정값을 재사용하며, 해당 측정값을 이 노출에 대해 설명하는 `.match.json`는 다음과 같습니다: `method`, `retained`, `gap_ms`를 해당 노출에서 측정값이 도출된 시료에 매칭하고, `in_tolerance`는 그 간격에 대한 값이며, `retained_from_epoch_ns`는 측정값이 처음 매칭된 노출을 나타냅니다. 따라서 DAQ가 중단된 지 한참 후에 촬영된 캡처에는 위의 줄들이 출력되며, 그 아래에는 그 아래에는 측정값이 여전히 허용 오차 범위 내에 있음에도 불구하고 출력되는 한 줄이 더 있습니다:

```
         This is the DAQ's last reading: the DAQ is no longer reporting, so each capture reuses it until a new one arrives
```

이 `.daq`는 여전히 해당 측정값을 기록한 DAQ의 이름을 나타내며, 해당 측정값이 기록된 보정 및 캡 정보를 포함하고 있으므로, 재가져오기 시 어떤 DAQ, 보정 및 캡이 이를 생성했는지 알 수 있습니다.

### 쓰기 실패한 파일

파일 쓰기가 불가능한 카메라(드라이브 용량 부족, 권한 오류 등)가 있어도 다른 카메라의 파일 저장은 중단되지 않습니다. 모든 캡처 명령(`lattice capture`, `multi-capture`, `array-capture`; `project capture`, `burst`, `stream`; 그리고 레시피의 `capture`, `burst` 및 `stream`)는 저장 실패한 각 파일을 stderr에 출력한 후, 나머지는 정상적으로 저장되었음에도 불구하고 **`1`에서 종료**됩니다:

```
  Failed: SN:214701292 [reflectance] OSError: [Errno 28] No space left on device
  Disk full: the drive the backend captures to has no space left. Free space on it (or capture to another drive) before capturing again.
```

캡처에 포함된 모든 파일이 처리되며, 단일-프레임 디베이어링 처리된 `lattice capture --levels debayered`도 포함됩니다. `.daq` 읽기 작업 중, `.match.json` 출처 사이드카를 기록할 수 없었던 항목은 `[match]`로 표시됩니다(`.daq` 파일 자체는 저장되었습니다).

드라이브가 가득 찬 상태에서 반복 캡처(`array-capture --continuous` / `--interval`, `project burst` / `stream`, 레시피 `burst` / `stream`) **해당 패스에서 중단**되며, 실행의 나머지 기간 동안 동일한 드라이브에 아무것도 기록하지 않습니다. 그 외의 오류는 발생하는 모든 패스에서 보고되며, 실행은 끝까지 진행된 후 `1`에서 종료됩니다. `array-capture --continuous` (백엔드의 연속 캡처)는 실패한 파일을 포함하여 어떤 파일도 나열하지 않습니다. 진행 상황 및 요약 줄에서 해당 파일을 집계하며 (`N failed`, `N file(s) failed to write.`)에서 해당 파일을 집계한 후 `1`에서 종료됩니다 — [연속 모드](#continuous-mode---continuous)를 참조하십시오.

레코더(`array-burst` 및 `array-record`)는 자체적인 오류를 보고합니다. `array-burst`는 드라이브가 가득 차거나 그 직전에 스스로 작동을 중지합니다. 두 레코더 모두 다른 쓰기 오류가 발생하더라도 계속 작동합니다. 둘 중 하나가 오류를 보고하면 카운터 뒤에 오류 메시지가 출력되며, 명령어 **`1`**가 종료됩니다:

```
  Recorder error: disk-full: OSError: [Errno 28] No space left on device
  Disk full: the drive the backend captures to has no space left. Free space on it (or capture to another drive) before capturing again.
```

`Disk low:`가 `Disk full:` 대신 표시된다는 것은 버스트가 드라이브가 가득 차기 전에 중단되었음을 드라이브를 채우지 못한 채 중단되었음을 의미합니다. 중단 전에 기록된 프레임은 유지됩니다. `array-burst --build`는 공간이 부족해지면 빌드를 건너뜁니다. 이는 비디오가 동일한 드라이브로 저장되기 때문입니다. 먼저 여유 공간을 확보한 후, `array-build-video --burst-dir …`를 실행하십시오. 다른 오류가 발생하더라도 빌드는 계속 실행됩니다.

프레임을 전혀 기록하지 못한 `array-record`의 경우도 녹화 실패로 간주됩니다 (`Recorder error: no-frames: the combined view published no frame to record …`, `1` 종료) — [`array-record`](#array-record--combined-index-videogif-monitoring-grade)를 참조하십시오.

---

## 캡처 모드, 레코더 및 오프라인 재처리

이 모든 기능은 **지속적 배열**에서 작동합니다(먼저 `array-connect`를 실행하십시오). 이들은 GUI 캡처 패널을 반영합니다.

### `array-capture` 모드

`array-capture`는 4가지 셔터 모드와 일련의 내보내기 토글을 포함하는 단일 명령어입니다:

| 모드 | 플래그 | 동작 |
| --- | --- | --- |
| **단일** *(기본값)* | (없음) | 동기화된 캡처 그룹 하나를 수행한 후 종료합니다. |
| **연속** | `--continuous` | `Ctrl+C`, `--count N` 캡처가 수행될 때까지 연속 캡처를 수행하거나, `--duration S` 또는 드라이브가 가득 찰 때까지 연속 캡처를 수행합니다. 백엔드 자체의 연속 캡처(GUI의 ‘연속 모드’)를 실행합니다 — 아래 [연속 모드](#continuous-mode---continuous)를 참조하십시오. |
| **간격** | `--interval S` | `S` 초마다 한 번씩(각 패스 시작 시점부터 측정), 동일한 범위. |
| **가장 빠름** | `--fastest` | 원시 전용 + 할당된 DAQ 판독값 + 결합 지수 합성; 광도/반사율/디스플레이 계산을 건너뛰어 프레임을 빠르게 출력합니다. `--processing raw --force-daq`를 의미합니다. 저장된 `.daq`를 나중에 보정된 결과물로 재처리하십시오. |

내보내기 토글(모든 모드와 결합 가능; 모두 GUI/SDK 엔드포인트를 공유):

| 플래그 | 효과 |
| --- | --- |
| `--array-id ID` | 캡처할 배열(하나만 연결된 경우 생략). |
| `-o, --output DIR` | 출력 폴더(기본값: `output`, 명령을 실행한 폴더 아래). |
| `--format {tiff,tif,png,jpg,jpeg}` | 파일 형식(기본값: `tiff`). PNG/JPEG의 경우 `display`와 인덱스 렌더링만 지원합니다. [캡처 내보내기 레벨](#capture-export-levels-the-all-default)을 참조하십시오. |
| `--processing LEVEL` | 단일 내보내기 레벨 (`raw`, `debayered`, `radiance`, `reflectance`, `display`), 또는 `all` (기본값). |
| `--levels L1,L2,…` | 내보내기 유형의 명시적 하위 집합(예: `raw,radiance,reflectance`); **`--processing`를 재정의합니다**. |
| `--aligned` / `--no-aligned` | **기본값: 정렬됨(Chloros 앱이 동일한 배열을 캡처할 때와 동일함)**(배열별**정렬됨** 캡처 설정, 사용자가 해제하지 않는 한 활성화됨). 모든 구성원의 비-원시 내보내기(`debayered`, `display`, `radiance`, `reflectance`, 인덱스 렌더링) 는 마스터 카메라의 프레임으로 변환되고 멤버들의 공통 중첩 영역으로 잘려 나옵니다. 따라서 파일들은 픽셀 정렬이 이루어지며, 앱에서 생성된 파일과 동일한 크기, 자르기 범위 및 `Alignment*` 태그를 갖게 됩니다 — 이는 센서 (또는 ROI)보다 작아집니다; ROI 환경에서는 변환이 창 크기에 맞춰 재설정됩니다. `XMP-mapir:AlignmentJSON`는 `"applied": true`를 나타내며 `crop_roi`를 제공합니다. Raw는 원본 크기로 워프 처리되지 않은 상태로 유지되지만 메타데이터(`"applied": false`)에 변환 정보를 포함합니다. `--no-aligned`는 모든 파일을 원본 크기로, 워프 처리되지 않은 상태로 저장합니다. `--aligned`는 명시된 기본값입니다. 워프가 적용되었는지 여부와 관계없이, 배열에 정렬 프로필이 설정되면 각 구성 파일은 해당 카메라의 `Alignment*` 변환을 기록합니다 — 단, 프로파일과 정렬되지 않은 카메라(생략되었거나, 해결 실패로 인해 신분 변환이 강제된 경우)와 다른 카메라가 정렬되지 않았을 때의 기준 카메라는 예외이며, 이 경우 `Alignment*` 태그가 포함되지 않습니다. 어레이에 프로파일이 없으면 캡처는 정렬되지 않은 상태로 저장되며, CLI는 다른 모든 경우(해워프가 적용될 수 없는 경우: 서로 다른 해상도의 구성원, 케이블 동기화가 되지 않은 어레이, 프로파일에 매핑할 수 없는 프레임 등)와 마찬가지로 `Alignment:` 줄에 그 이유를 출력하며, 이는 워프가 적용될 수 없었던 다른 모든 경우(구성 요소의 해상도 차이, 케이블 동기화가 되지 않은 어레이, 프로파일을 매핑할 수 없는 프레임 등)에서도 마찬가지입니다. 어레이의 자동 정렬 기능은 노출이 안정화되면 프로파일을 생성하며; `align-calibrate`는 필요에 따라 프로파일을 생성합니다. |
| `--orient` | 각 구성 요소의 **결과물**— `display`, `radiance`, `reflectance`, `index/` 렌더링 및 `composite/` — 정렬 워프 후 카메라 방향에 따라:**마스터의**방향으로 워프되어 구성 요소들이 서로 정렬된 상태를 유지합니다; 정렬되지 않은 파일(`--no-aligned` 또는 워프를 적용할 수 없었던 캡처)은 자체 카메라의 방향으로 처리됩니다.**기본값: 꺼짐 — 모든 구성 파일은 원본 상태를 유지**하며, 이는 배열 캡처의 경우 항상 그렇습니다. `raw`와 `debayered`는 어느 경우든 원본 프레임을 유지합니다. 모든 파일은 해당 제품이 가진 방향을 기록합니다([카메라 방향](#what-a-captures-folder-looks-like) 참조); `Alignment*` XMP는 방향이 적용되기 전에 적용된 네이티브 프레임 변환을 유지하므로, `process` (워프, 그다음 방향)은 라이브 결과물을 재현합니다. |
| `--index` / `--no-index` | 카메라별로 구성된 식생 지수 오버레이를 저장하거나 건너뜁니다. 기본값: 렌더링합니다. |
| `--no-daq` | 선택된 레벨 중 DAQ를 사용하는 레벨이 없을 때(원시 데이터만 추출된 경우) `.daq` 사이드카를 건너뜁니다. 기본적으로 측정값이 저장되므로, 프레임을 오프라인에서 반사율/지수 값으로 재처리할 수 있습니다. `--force-daq`도 여전히 허용됩니다(기본값). |
| `--daq-tolerance-ms MS` | 이러한 캡처에 대한 DAQ 일치 허용 오차 (기본값 500 ms). |
| `--smart` | 트리거하기 전에 모든 카메라에서 AE가 안정화될 때까지 대기합니다([Smart-AE / Smart-Capture](#smart-ae--smart-capture) 참조). |
| `--compression {deflate,none}` | TIFF 픽셀 압축. `none` (1.3.0부터 기본값; 1.2.0에서는 `deflate`로 기록됨) = 압축되지 않음 — 센서에서 읽은 픽셀을 정확히 그대로 저장하며, 전체 해상도 프레임당 약 6.3 MB이고, 가장 빠른 지속 전송 속도를 제공합니다; `deflate` = 무손실 zlib L1 + 수평 예측기, 프레임당 약 4.1 MB이며, 속도가 더 느립니다. 두 방식 모두 무손실이며, 가져올 때 동일하게 읽힙니다. |

> **단일 쓰기 TIFF + 지속 전송률 모델.**모든 파일은**한 번**만 기록됩니다: 픽셀, XMP 패킷, IFD0 제조사/모델/카메라 일련번호 및 EXIF `ExposureTime` / `SerialNumber` 태그를 한 번에 처리하며, 이후 재기록되는 내용은 없습니다. `--continuous`X는 연속적으로 동기화된 그룹을 캡처합니다. 각 캡처는 이전 캡처가 완료된 후의 그룹을 가져오며(해당 캡처가 저장 중인 동안 게시된 그룹도 포함), 따라서 어레이 속도로 실행됩니다. DEFLATE는 Python GIL을 점유하므로, 압축된 쓰기 작업은 카메라별 라이터 스레드 간에**병렬화되지**않습니다. 센서 속도로 풀 해상도 캡처를 지속하려면 비압축 쓰기(기본값)**와** NVMe급 디스크(9대 카메라 어레이 기준 ~10 fps에서 지속 속도 ~600 MB/s)가 필요합니다. 동일한 설정 옵션은 `POST /api/camera/array/capture`에서 `compression`로 노출됩니다.

```bash
# Interval timelapse: one reflectance pass every 10 s for 5 minutes
chloros-cli lattice array-capture --interval 10 --duration 300 \
  --processing reflectance -o timelapse/

# Fastest grab for a moving rig — raw + .daq now, calibrate later
chloros-cli lattice array-capture --fastest -o flightline/

# Co-registered multi-band export (aligned is the default; drop the index overlay)
chloros-cli lattice array-capture --processing reflectance --no-index -o out/

# Each member at its native size, unwarped
chloros-cli lattice array-capture --processing reflectance --no-aligned -o native/

# Continuous raw capture for 60 s, as fast as the array publishes
chloros-cli lattice array-capture --continuous --duration 60 --fastest -o flightline/
```

#### 연속 모드 (`--continuous`)

`array-capture --continuous`(`--interval` 또는 `--smart`가 없는 경우)는 작업을 **백엔드 자체의 연속 캡처**로 넘깁니다. 이는 GUI의 연속 모드가 실행되는 세션입니다. 백엔드는 동기화된 모든 그룹을 자체적으로 저장하며, 두 개의 캡처가 동시에 진행되고 그룹 간에 CLI 요청이 발생하지 않으므로, 캡처당 요청이 뒤처지는 고속 어레이의 속도를 따라잡을 수 있습니다(512픽셀 ROI 기준 9대 카메라 어레이에서 73–77 캡처/초로 측정됨). 위의 모든 내보내기 토글은 각 캡처에 적용됩니다.

- **범위(Bounds):** `--count N`는 N개의 캡처 후(1개 캡처 = 1개 동기화된 그룹: 모든 구성원의 파일); `--duration S`는 S초 후에 중지됩니다. 둘 다 설정되지 않은 경우, `Ctrl+C`, 드라이브 용량 소진, 오류 발생 또는 어레이가 중단될 때까지 실행됩니다. `Ctrl+C`는 백엔드 세션을 중지하고, 그 후 요약 정보를 출력합니다.
- **진행 상황**은 2초마다 표시되며, 값이 0이 아닐 경우(건너뛴 그룹은 어레이에서 게시되었으나 캡처되지 않은 그룹을 의미) `, N failed` 및 `, N groups skipped`가 추가됩니다:

  ```
     12.0s  880 captures (73.3/s), 7920 files
  ```

- **파일별 목록은 표시되지 않습니다.** 실행은 하나의 요약 줄(캡처 수, 속도, 파일 수 및 출력 폴더)로 끝나며, 값이 0이 아닐 경우 그 뒤에 `N file(s) failed to write.` / `N group(s) skipped.`가 이어집니다:

  ```
  Done — 880 captures (73.3/s), 7920 files in /home/user/flightline.
  ```

- **드라이브 용량이 가득 차면** `Continuous capture stopped: the drive is full.`로, 오류가 발생하거나 어레이가 제거되면 `Continuous capture stopped: <reason>.`로 중단됩니다(둘 다 stderr에 출력됨).
- **종료 상태:** 캡처 내용을 저장하고 모든 파일이 정상적으로 기록된 경우(`Ctrl+C` 포함)에는 `0`가 반환되며, 파일 기록에 실패한 경우( 드라이브 용량 부족, 오류, 어레이가 사라지는 경우, 또는 캡처가 전혀 없는 경우.
- **Microsoft Defender:** Defender가 캡처가 쓰는 파일을 스캔하는 데 CPU 코어 하나 이상을 사용할 경우, 실행 시 stderr에 한 번 다음과 같이 출력됩니다: `Note: Microsoft Defender is using N.N CPU cores scanning the capture files, CPU the capture needs. Re-run the Chloros installer with "Exclude the Chloros backend from Microsoft Defender scanning" checked.` 설치 프로그램의 [시스템 설정](#windows-exe)을 참조하십시오.
- 어레이당 한 번에 하나의 연속 캡처만 허용되며, 두 번째 캡처 시 `a continuous capture is already running for this array` 오류가 발생합니다. HTTP를 통한 세션은 `POST /api/camera/array/continuous/start`(`/api/camera/array/capture` 본문과 `max_captures` / `duration_s`)이며, `GET /api/camera/array/continuous/status?array_id=ID` 및 `POST /api/camera/array/continuous/stop`로 구성됩니다.

**Interval**(`--interval S`, `--continuous` 유무에 관계없이) 및**`--continuous --smart`**는 요청별 루프를 유지합니다: 패스당 하나의 캡처 요청 —**간격**의 경우 예정된 각 시점에,**`--smart`**의 경우 AE가 안정화된 후 — 각 패스의 파일은 싱글 모드와 동일하게 나열되며, 끝에 `Done — N capture pass(es).` 줄이 있고, `--count N`가 패스 횟수를 세어줍니다.

### `array-record` — 통합 인덱스 비디오/GIF (모니터링 등급)

배열의 **실시간 통합 인덱스 뷰**를 `.avi`(선택적으로 `.gif`)에 기록합니다. GUI의 통합 뷰가 열려 있는 상태에서, 표시된 그대로 해당 뷰를 기록합니다. 열려 있는 창이 없는 경우, 백엔드는 뷰를 직접 렌더링합니다. 즉, `--expr`를 통해 전달된 인덱스나, 배열이 정렬된 후 기본적으로 배열의 NDVI를 사용합니다. 기본 NDVI는 라이브 뷰와 마찬가지로 660 nm 및 850 nm에 가장 가까운 적색 및 NIR 대역을 사용합니다. 녹화 중에 열린 GUI 뷰는 열려 있는 동안 계속 기록됩니다. 이 기능은 2초마다 진행 상황을 확인하고, `--duration`, `Ctrl+C`가 발생하거나 레코더가 자체 종료될 때 중단됩니다.

프레임을 기록하지 못하는 녹화는 실패합니다. 레코더는 `no-frames: …`를 보고하며(배열이 여전히 정렬 중이거나 노출을 안정화 중일 수 있음), 명령은 **`1`에서 종료**됩니다.

```bash
# 30-second combined-index clip at 10 fps, plus a GIF (no GUI needed)
chloros-cli lattice array-record --duration 30 --fps 10 --gif -o monitoring/

# Record a chosen index instead of the default NDVI
chloros-cli lattice array-record --duration 30 \
  --expr "(NIR_850_RGN-Red_660_RGN)/(NIR_850_RGN+Red_660_RGN)" -o monitoring/
```

| 플래그 | 기본값 | 설명 |
| --- | --- | --- |
| `--array-id ID` | 배열만 | 대상 배열(하나만 연결된 경우 생략). |
| `-o, --output DIR` | `output` | 출력 디렉터리. 상대 경로는 명령을 실행하는 폴더를 기준으로 합니다. |
| `--fps F` | `10` | 녹화 프레임 속도. |
| `--duration S` | Ctrl+C를 누를 때까지 | `S` 초 후 자동 중지. |
| `--gif` | off | 애니메이션 GIF도 함께 작성. |
| `--gif-only` | 꺼짐 | GIF만 생성(`.avi` 없음). |
| `--expr EXPR` | 배열의 NDVI | GUI 뷰가 열려 있지 않을 때 기록할 인덱스 표현식. 배열의 밴드 이름으로 지정 (예: `'(NIR_850_RGN-Red_660_RGN)/(NIR_850_RGN+Red_660_RGN)'`; [밴드 이름](#index--vegetation-maths) 참조). 짧은 색상 이름(`NIR`, `Red`)은 배열의 한 밴드에만 해당 이름이 있을 때만 작동합니다. 열려 있는 GUI 뷰는 표시된 대로 기록됩니다. |

### `array-burst` — 원시 베이어 고프레임률 연사 (분석 등급)

그랩 루프의 동기화된 그룹 버퍼를 직접 읽습니다 — **보정 체인, exiftool, 라이브 뷰가 필요하지 않음** — 따라서 카메라의 최대 그랩 속도로 실행됩니다. 원본 프레임 + 프레임별 매니페스트 + `<output>/bursts/<base>/` 하위의 고유한 DLS 판독값마다 하나의 `.daq`를 기록합니다. 오프라인에서 재처리(다음 명령어)하거나, `--build`를 전달하여 중지 시 즉시 재처리합니다.

```bash
# 5-second raw burst, then build the combined index video in one shot
chloros-cli lattice array-burst --duration 5 --build \
  --products combined:index --fps 10 -o capture/
```

| 플래그 | 기본값 | 설명 |
| --- | --- | --- |
| `--array-id ID` | 배열 전용 | 대상 배열. |
| `-o, --output DIR` | `output` | 출력 디렉터리(버스트 데이터는 `<DIR>/bursts/<base>/`에 저장됨). |
| `--duration S` | Ctrl+C까지 | `S` 초 후 자동 중지. |
| `--max-frames N` | 무제한 | `N` 개의 원시 프레임이 처리된 후 자동-중지. |
| `--orient` | 비활성화 | 이 버스트를 기반으로 빌드가 생성한 결과물은 카메라 방향에 따라 회전됩니다. 카메라별 결과물은 해당 카메라의 방향에 따라, 합성 인덱스는 마스터 카메라의 방향에 따라 회전됩니다 — `array-capture --orient`에서와 같이. 원본 프레임은 원본 형식을 유지하며; 버스트의 `manifest.json`는 각 구성원의 방향과 옵션을 기록합니다. |
| `--build` | 꺼짐 | 중지 후 즉시 버스트를 재처리합니다(`array-build-video`와 동일). |
| `--products …` | `combined:index` | `--build`와 함께 사용할 경우: 생성할 동영상(들)을 지정합니다(아래 참조). |
| `--fps F` | `10` | `--build`와 함께 사용할 경우: 출력 동영상의 fps. |
| `--save-tiffs` | 꺼짐 | `--build`와 함께 사용 시: 프레임별 보정된 TIFF 파일도 저장합니다(각 파일에는 픽셀이 클램핑되었을 때의 [언믹스 클램프 마스크](#unmix-clamp-mask-which-pixels-a-band-was-clamped-on)을 포함); 빌드 결과에서는 이를 `clamp_masks`로 나열합니다). |
| `--gif` | off | `--build`와 함께 사용할 경우: 애니메이션 GIF 파일도 작성합니다. |

### `array-build-video` — 저장된 버스트를 오프라인에서 재처리

각 원시 프레임을 가장 가까운 저장된 `.daq` 측정값과 시간적으로 매칭하고, 이를 **가져오기 파이프라인과 동일한 방사도/반사도/지수 체인**을 통해 처리하여 하나 이상의 동영상을 렌더링합니다.

`--products`는 `kind:level` 항목들의 쉼표로 구분된 목록이며, 여기서 `kind` ∈ `per_cam` | `combined`이며, `level` ∈ `radiance` | `reflectance` | `index`입니다. 단독으로 지정된 `level`(`kind:` 없음)의 기본값은 `per_cam`입니다. 기본값은 `combined:index`입니다.

```bash
# Per-cam reflectance video for every member + one combined NDVI video
chloros-cli lattice array-build-video \
  --burst-dir "capture/bursts/2026-06-24_141500" \
  --products per_cam:reflectance,combined:index \
  --fps 10 --save-tiffs
```

| 플래그 | 기본값 | 설명 |
| --- | --- | --- |
| `--burst-dir DIR` | (필수) | 버스트 폴더 경로 (`…/bursts/<base>/`). |
| `--products …` | `combined:index` | `kind:level` 목록(위와 동일). |
| `--fps F` | `10` | 출력 동영상 fps. |
| `--save-tiffs` | off | 동영상과 함께 프레임별 보정된 TIFF 파일도 저장합니다. |
| `--gif` | off | 애니메이션 GIF도 함께 작성합니다. |

**동영상에는 클램프 마스크가 포함되지 않습니다** (단일 이미지 스트림, 8비트). 대신 빌드 결과물에는 `clamp_fractions`가 포함됩니다: 각 동영상 결과물마다, 각 R/G/B 채널이 보유한 대역 내에서 언믹스 클램핑된 픽셀의 비율을 포함합니다. 이는 해당 프레임의 복사도/반사도 곱에 적용될 `Chloros:Band{R,G,B}UnmixClampFraction` 값입니다(통합 인덱스의 경우, 각 구성 카메라별). 이 실행은 제품별로 최악의 프레임을 `Clamp:` 줄에 출력합니다. `--save-tiffs`는 TIFF 파일용 픽셀별 마스크를 추가하며, TIFF 파일의 `<level>/` 폴더 옆에 있는 `clamp_masks/<level>/`에 저장합니다.**빌드가 중단되더라도 손상된 동영상은 남지 않습니다.** 각 동영상은 `<name>.partial.avi` 형식으로 기록되며, 완료된 후에야 최종 이름이 지정되므로, 해당 이름의 이전 동영상은 대체 파일이 완전히 생성될 때까지 남아 있습니다. 남겨진 `.partial.avi` 파일을 대체하려면 빌드를 다시 실행하십시오.

이 실행 과정에서는 기록된 모든 비디오를 출력한 후, `Skipped:` 결과물(비디오가 생성되지 않은 요청된 카메라/레벨 — 또는 결합된 인덱스 — 과 그 이유 — 예: 보정 번들 없음, 반사율에 대한 DLS 측정값 없음, 또는 결합된 인덱스에 대한 적색/NIR 대역 쌍 없음) 및 모든 `Note:`를 출력합니다. 복사도나 반사율을 요청받은 RGB 카메라는 대신 디스플레이 영상을 받게 되며, 카메라의 보정 번들이 있는 경우 화이트 밸런스가 적용됩니다; 교정 번들이 없는 상태에서 렌더링된 카메라는 `Note:`에 명시됩니다. 요청된 산출물이 건너뛰어지거나 요청대로 생성되지 않은 경우(구성된 인덱스 표현식이 오프라인에서 평가되지 않음)에는 **0이 아닌 값으로 종료**되며, 생성된 영상은 여전히 기록되고 나열됩니다.

> **적절한 레코더를 선택하십시오.** `array-record`는 *모니터링 등급*입니다. 이 레코더는 렌더링된 라이브 합성 영상, 즉 GUI의 통합 뷰를 기록하며, GUI가 열려 있지 않은 경우 백엔드 자체의 NDVI(또는 `--expr`) 뷰를 기록합니다. `array-burst` → `array-build-video`는 *분석 등급*입니다 — 이 모델은 원시 센서 데이터를 최대 속도로 저장한 후, 라이브 뷰 없이도 보정된 방사도/반사도/지수 동영상을 재구성합니다.

### 모노(M3M) 단일 대역 카메라

**M3M**라인은 베이어 방식의**M3C**와 대응하는 모노 제품군입니다. 카메라당 하나의 협대역 간섭 필터(`M3M-<lens>-F<wavelength>`, 예: `M3M-L87-F685`)가 장착되어 있어, 따라서 센서는 베이어 모자이크 없이**단일 그레이스케일 대역**을 출력합니다. 디모자이크할 필요가 없고, 채널 간 크로스톡을 분리할 필요도 없으며, 화이트 밸런스를 설정할 필요도 없습니다. 즉, 전체 RGB 디스플레이 컬러 파이프라인이 적용되지 않습니다.

CLI에서 이는 다음과 같은 의미를 가집니다:

- **`lattice white-balance`, `lattice color-profile`, `lattice color`**는 모노 카메라를 감지하면 무의미한 설정을 적용하는 대신**한 줄의 메시지와 함께 건너뜁니다**. 동일한 세션에서 RGB/Bayer M3C 카메라를 대상으로 할 때는 여전히 정상적으로 실행됩니다.
- **Bayer 픽셀 형식은 해당 카메라의 색상 심도를 의미합니다.** M3M은 모노 형식만 지원하므로, `--pixel-format` 또는 `--preset`에서 베이어 형식을 지정하면 동일한 심도의 모노 형식이 설정됩니다: `BayerRG8` 및 `--preset high_speed`는 `Mono8`를 생성하고, `BayerRG12`, `high_quality` 및 `triggered`는 `Mono12`를 생성하며; 패킹된 이름은 패킹된 상태로 유지됩니다(`BayerRG12p`는 `Mono12p`를 생성합니다). 파일과 `lattice format`는 Mono 형식을 나타냅니다. M3C 및 M3M 카메라가 혼합된 배열은 동일한 방식으로 하나의 형식을 취합니다 각 M3C에는 베이어(Bayer) 방식이, 각 M3M에는 모노(Mono) 심도가 적용됩니다. 이름은 비트 심도를 나타내며, 반대의 경우도 마찬가지이므로, 따라서 M3C는 절대 Mono 형식으로 설정되지 않습니다(어느 카메라가 어레이의 마스터이든 상관없이, M3C에서 `--pixel-format Mono12`는 `BayerRG12`가 됩니다). `lattice format --set`도 같은 방식으로 매핑되며 설정된 포맷으로 출력됩니다.
- **`lattice calibrate` / `process --reflectance` / `array-capture --processing radiance`** 는 여전히 작동합니다. 방사도 및 반사도는 *밴드별* 방사계 맵이며, 단일 밴드에 대해서는 완벽하게 정의되어 있습니다. 모노 프레임은 **신원** 센서 응답 행렬(3×3 언믹스 없음)을 포함하므로, 이 평면은 보정 계산을 거치지 않고 그대로 통과합니다.
- **단일 모노 카메라로는 식생 지수를 산출할 수 없습니다.**NDVI/NDRE 등은 최소 두 개의 밴드(예: Red + NIR)가 필요합니다. 모노 하드웨어에서 지수를 얻으려면, 서로 다른 파장의**여러 대**의 M3M 카메라를 사용하여 이미지를 촬영한 후, 이를 하나의 다중 밴드 스택으로 정렬하고 *그* 스택에 대해 지수를 산출해야 합니다:

```bash
# Red (660) + NIR (850) mono pair -> aligned 2-band stack -> NDVI
chloros-cli lattice index --live --profile align.json \
  --preset NDVI --channel red=Mono_660 --channel nir=Mono_850 \
  --save-multiband -o output/
```

`index --live`는 카메라를 자체적으로 실행합니다([일시적 세션](#multi-camera-transient-sessions)), 따라서 먼저 아무것도 연결하지 마십시오: `--serials`, 그렇지 않으면 프로필에서 활성화된 카메라들이 ([인덱스 / 식생 수학](#index--vegetation-maths) 참조). Chloros가 어떤 카메라(GUI, `cam-connect`, `array-connect`)를 보유하고 있는 동안 HTTP 409 오류(`CAMERA_HELD`)가 발생하며 `1`에서 종료됩니다; `chloros-cli lattice cam-disconnect --all` / `chloros-cli lattice array-disconnect --all`를 사용하여 카메라를 해제하십시오.

`--channel` 기호는 프리셋의 채널 이름과 **정확히** 일치해야 합니다(대소문자 구분; NDVI는 소문자입니다 `red`, `nir` — `--list-presets` 참조), 또한 밴드 측면 이름은 정렬된 스택 내의 밴드를 나타내야 합니다(오프라인 모드에서는 0을 기준으로 하는 밴드 인덱스도 허용됨, 예: `--channel red=0 --channel nir=1`).

스택 전체에서 구별 기준이 되는 것은 모델 문자열 내의 `M3M` 토큰이며(이 토큰은 `M3C` 문자열에는 절대 나타나지 않음), GUI/SDK에는 `is_mono`로 표시됩니다.

---

## 호스트 NIC 설정 및 튜닝 (LATTICE 어레이)

LATTICE 카메라는 호스트의 이더넷 어댑터를 통해 GVSP를 스트리밍하므로, 다중 카메라 어레이의 경우 어댑터의 **드라이버**및**수신 링 크기**는 링크 속도만큼 중요합니다. 설정이 잘못되면 어레이 설정 패널에서 `FRAMES WILL DROP` / `Reduce ROI to enable` 경고로 나타납니다(카메라 자체는 정상 상태일지라도).

### USB 10GbE 어댑터 — Realtek RTL8157 (“Realtek USB 10GbE Family Controller”)

아래 설정은 Windows 드라이버 속성입니다. Linux에서도 USB 어댑터에는 전송 속도 제한이 적용됩니다(x86_64에서 400 MB/s, NVIDIA Jetson의 경우 200 MB/s) ([Array fps & burst model](#array-fps--burst-model) 참조)이며, 모든 어댑터에서 커널의 소켓 수신 버퍼가 중요합니다 — [Linux 호스트](#linux-hosts--socket-receive-buffers)를 참조하십시오.

| 항목 | 필수 값 | 중요성 |
| --- | --- | --- |
| **드라이버 버전**|**≥ v10.67 (2026년 1월)**, INF `rtump64x64sta.inf` | 구형**2016**드라이버(v10.65, `rtump64x64.inf`)는 시스템 종료/재시작/절전 모드 시**`DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`)**와 관련된 전원 차단 및 버그 체크를 제대로 처리하지 못합니다. 전환 과정에서 시스템이 멈추고(약 5분 타임아웃), 사용자가 강제 전원 끄기를 수행하면, 반복되는 비정상 종료로 인해**WMI 저장소가 손상**되며(PowerShell/도구가 `Invalid class` 오류로 작동하지 않음),**USB 스택이 멈춰 버립니다** (NIC 활성화 불가; USB 드라이브 열거 중단). 정상 재시작을 시도하기 전에 realtek.com(또는 동글 제조업체)에서 업데이트를 받으십시오. |
| **수신 버퍼**— 키워드 `ReceiveBufferLen` |**256**(드라이버 최대값) | NIC RX 링. 드라이버 기본값인**32**는 사용 가능한 링을 약 0.26 MB만 남겨두는데, 이는 멀티캠 버스트에 비해 너무 작으므로 어레이 패널에서 `Sim-emit burst … exceeds NIC RX ring usable capacity 0.26 MB`를 보고하고 연결을 차단합니다.**256**링은 넉넉한 크기(**실험실 10GbE 호스트에서 측정된 약 13.5 MB**)를 갖춰, RX 파이프라인에 멀티캠 GVSP 버스트를 처리할 수 있는 실질적인 여유 공간을 제공합니다. (주어진 구성이 실제로 *연결*되는지는 두 가지 검사 — **드레인 인식**허용 검사 및**총 과구독** 검사 — 단순한 버스트 대 링 비교가 아닙니다; [어레이 fps 및 버스트 모델](#array-fps--burst-model) 참조.) |
| **수신 URB**— 키워드 `PendingReceives` |**64** (최대) | 전송 중인 USB 요청 블록; 버스트 흡수를 위해 수신 버퍼와 함께 늘려야 합니다. |
| **점보 프레임** — 키워드 `*JumboPacket` | **9014** | 9000바이트 GVSP 패킷에 필요함(1500바이트 패킷 대비 프레임당 패킷 수가 6배 적음). |

> ⚠️ **NIC 드라이버를 업데이트하면 이러한 고급 속성이 기본값으로**재설정**됩니다.**어댑터 드라이버를 업데이트하거나 교체한 후에는**`ReceiveBufferLen=256`**`ReceiveBufferLen=256` 및 `PendingReceives=64`를**다시 적용**해야 합니다. 그렇지 않으면 "하드웨어에 변경 사항이 없음"에도 불구하고 어레이 패널이 다시 차단됩니다. 이는 이전에 정상적으로 작동하던 시스템이 갑자기 연결을 거부하는 가장 흔한 원인입니다.**관리자 권한**으로 실행된 PowerShell에서 다음 명령을 적용하십시오(어댑터 이름을 대체하십시오. 예: `"Ethernet 5"`):

```powershell
Set-NetAdapterAdvancedProperty -Name "Ethernet 5" -RegistryKeyword ReceiveBufferLen -RegistryValue 256
Set-NetAdapterAdvancedProperty -Name "Ethernet 5" -RegistryKeyword PendingReceives  -RegistryValue 64
Get-NetAdapterAdvancedProperty  -Name "Ethernet 5" -RegistryKeyword ReceiveBufferLen,PendingReceives   # verify
```

> **`lattice network --fix`는 USB 10GbE 어댑터를 지원합니다.**이제 어댑터 유형을 감지하여 올바른 수신 링(receive-ring) 키워드를 설정합니다: PCIe NIC(Intel I219, 등), 또는 Realtek**USB** 10GbE 컨트롤러(`*ReceiveBuffers`를 노출하지 않음)의 경우 `ReceiveBufferLen`→256 + `PendingReceives`→64로 조정됩니다.. 대상 값은 각 드라이버가 보고하는 최대값(`NumericParameterMaxValue`)으로 제한되므로, 범위를 벗어난 값이 기록되는 일은 절대 없습니다. **관리자 권한** 터미널에서 실행하십시오. 다른 레지스트리 기반 조정과 마찬가지로, 이 변경 사항은 어댑터를 다시 시작하거나 시스템을 재부팅한 후에 적용됩니다. 위의 수동 `Set-NetAdapterAdvancedProperty` 명령어도 여전히 훌륭한 대안입니다. 이 명령어는 재시작 없이도 즉시 적용되며(어댑터를 다시 바인딩함) 즉시 효과를 볼 수 있습니다.

### Linux 호스트 — 소켓 수신 버퍼

Linux (NVIDIA Jetson 포함)에서는 커널의 소켓 수신 버퍼가 NIC의 수신 링만큼 중요합니다. 커널 기본값(`net.core.rmem_max` / `net.core.rmem_default` = 212,992바이트)에서는 카메라에서 전송되는 GVSP 패킷의 버스트가 클로로스(Chloros)가 이를 읽기 전에 스트림 소켓을 오버플로우시킵니다. NIC 자체는 손실이 없다고 보고하지만, 패킷은 커널에서 드롭되며, 어레이는 프레임 그룹을 잃고 프레임 속도를 낮춥니다. USB 10 GbE 어댑터에 연결된 4대의 카메라로 구성된 어레이에서 측정한 결과, 두 값을 모두 32 MB로 높였을 때 연결 후 어레이의 프레임 속도가 4.96 fps에서 7.52 fps로 향상되었으며, 연속 원시 캡처 속도는 4.7 캡처/s에서 7.6 캡처/s로 증가했고, 커널에서 패킷 손실은 발생하지 않았습니다.

- **.deb 패키지가 이 값들을 설정합니다.** 패키지를 설치하면 `/etc/sysctl.d/60-chloros-gige.conf`(둘 다 `33554432`)가 기록되고 즉시 적용됩니다. 이미 더 높은 값으로 설정된 경우 변경되지 않습니다.
- **`lattice network`는 이를 확인합니다.** 둘 중 하나라도 이 값보다 낮으면 *발견된 문제* 항목에 `socket receive buffers too small for GigE Vision (…)`가 나열되며, 첫 번째 카메라 또는 어레이 연결 시 백엔드 로그에 동일한 경고가 기록됩니다.
- **수동으로 값을 높이려면** (다음 재부팅 전까지; 패키지의 파일은 부팅할 때마다 이 설정을 적용합니다):

  ```bash
  sudo sysctl -w net.core.rmem_max=33554432 net.core.rmem_default=33554432
  ```

  `lattice network --fix`도 이 값들을 높이고 유지하지만, 백엔드가 root 권한으로 실행될 때만 가능합니다. 사용자 권한으로 실행되는 백엔드(`chloros-backend` 또는 [`chloros-backend@<user>`](#running-the-backend-as-a-service-headless))로 시작된 백엔드는 커널 설정을 변경할 수 없으며, 대신 `sudo sysctl` 줄을 출력합니다.

**자동 어댑터 조정 기능은 연결된 어댑터에만 적용됩니다.** 카메라가 처음 연결될 때, 루트 권한으로 실행 중인 백엔드는 수신 링 크기가 작은 유선 어댑터를 조정합니다. 이제 링크가 연결된 어댑터만 조정하므로, 연결이 해제된 온보드 포트는 기존 MTU를 유지하며 수신 버퍼는 그대로 둡니다. `lattice network`는 더 이상 연결이 해제된 어댑터의MTU를 문제로 표시하지 않습니다. `lattice network`, `--fix`, `lattice probe` 및 어댑터 조정 기능은 유선 이더넷 포트만 고려하며, Jetson의 CAN 버스(`can0`)나 호스트 PC에 제공하는 USB 장치 모드 링크 (`usb0`, `usb1`)는 고려 대상에서 제외되지만, 카메라용 USB 이더넷 어댑터는 여전히 포함됩니다.

### 네트워크 기본 사항 (모든 LATTICE 링크)

- **주소 지정:** 링크 로컬 `169.254.0.0/16` (GigE Vision LLA). 호스트는 고정 주소 `169.254.x.x/16`를 사용하며, 카메라와 DAQ-E는 동일한 범위 내에서 자체 할당합니다. DHCP나 게이트웨이는 필요하지 않습니다.
- **패킷 크기:**점보(9000)를 권장하지만, 자동 탐색 기능을 통해 적절한 크기를 찾도록 합니다. GVSP 탐색을 통해 카메라의 1500바이트 ICMP 제한을 우회하므로, 실제 회선이 지원하는 곳이라면 어디에서든 점보 크기로 설정됩니다.**각 카메라는 백엔드 세션당 한 번 측정되며, 링크가 변경될 때(다른 어댑터, 다른 링크 속도, 또는 스위치 교체 시처럼 링크가 끊어진 경우) 다시 측정됩니다. 재연결 시에는 기존 측정 결과를 재사용합니다. 스위치 뒤쪽에서 변경이 발생했으나 스위치가 계속 가동 중이라면, 백엔드를 재시작하여 다시 측정하십시오.**이 검사에서 첫 연결 시 가끔 몇 분이 소요될 수 있으며, 그럴 경우 로그에 해당 내용이 기록됩니다. 프로브보다 더 정확한 정보를 알고 있는 경우에만 `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000`로 고정하고, 영구 설정보다는 명령어별 설정을 우선적으로 사용하십시오. 고정 설정은 프로브를 건너뛰게 하므로, 따라서 경로가 실제로 9000을 전송할 수 없는 경우,**모든** 캡처가 `SC_ERR_TIMEOUT -1011`로 인해 타임아웃됩니다([환경 변수](#environment-variables) 참조).
- **RX 링은 `ReceiveBufferLen`에 따라 확장됩니다:**기본값인 `32`에서는 사용 가능한 링 용량이 ~0.26 MB(어떤 멀티캠 버스트에도 너무 작음)이며, 최대값인 `256`에서는 (~13.5 MB, 실험실 10GbE 호스트에서 측정)로 실제 여유 용량을 제공합니다. 따라서 구성의 연결 여부는 드레인 인식 허용량 검사**및** 아래의 집계 과부하 검사 결과에 따라 결정되며, 단순한 버스트 대 링 용량 비교에 의한 것이 아닙니다.

### 어레이 fps 및 버스트 모델

어레이 설정 패널(및 `lattice analyze-array` / SDK의 `analyze_array_network`)을 읽는 방법:

- **버스트는 각 카메라의 실제 픽셀 형식에 따라 카메라별로 합산됩니다.**모노**M3M**카메라는**Mono12 (2 B/px)**로 스트리밍하거나, `--pixel-format`를 사용할 경우 해당 깊이의 모노 형식으로 스트리밍합니다. (`BayerRG8`는 `Mono8`이며, 1 B/px);**M3C**베이어 카메라는 8비트 또는 12비트로 스트리밍합니다(M3C 펌웨어는 BayerRG8이 요청되더라도 자동으로 BayerRG12를 출력합니다). 따라서 4대의 카메라로 촬영한 풀 해상도 프레임은**모두 8비트일 경우 ~12.6 MB이지만, 12비트 모노 카메라 3대를 사용할 경우 ~25 MB**가 됩니다. 이 프로젝션은 모델(ID 캐시)을 기반으로 각 카메라의 포맷을 파악하므로, 버스트 데이터는 일률적인 BayerRG8 가정이 아닌 실제 전송 라인을 통해 전달되는 데이터와 일치합니다.
- **USB 이더넷 어댑터는 명판에 표기된 사양과 관계없이 200 MB/s로 제한됩니다.** 링크 속도를 지속 전송 속도로 변환하는 효율성 표는 PCIe에서 파생된 것입니다. USB NIC는 *이더넷* 링크 속도를 광고하지만, USB 버스 및 드라이버의 제약에 묶여 있습니다. 한때 ~1063 MB/s "지속" 속도를 기록했었는데 — 이 수치는 실제로 검증된 적이 없습니다 — 그 결과 발생하는 속도 조절 문제로 인해 6~18%의 프레임이 손상되었음에도 불구하고 정상적인 목표 fps는 보고되었습니다. 현재 USB 연결 NIC의 속도는 절대적으로 **200 MB/s**로 제한되며, 이는 Windows 및 NVIDIA Jetson에서 동일합니다. Linux x86_64에서는 **400 MB/s**로 제한됩니다(Linux에서 USB 2.0 포트에 연결된 어댑터의 할당량은 **30 MB/s**이며, 기능 설명서에는 USB 3 포트에 연결하라고 명시되어 있습니다) (이 제한은 호스트의 USB 버스 및 드라이버에 기인하므로 명판 사양에 따라 확장되지 않습니다. USB 1 GbE 어댑터는 약 80 MB/s의 속도를 내며 이 제한의 영향을 받지 않습니다). Linux에서는 sysfs를 통해 어댑터를 식별하며(`ethtool -i`의 `bus-info`로 대체됨). 각 수치는 측정된 결과입니다: Windows에서는 동일한 4대 카메라 어레이가 200 MB/s를 초과할 때 프레임의 6–10 %가 손상되었습니다; Linux x86_64에서는 동일한 어댑터가 480 MB/s(실제 전송 속도 273 MB/s, 카메라 자체의 한계)까지 모든 상한선에서 오류 없이 작동했습니다. NVIDIA Jetson에서는 동일한 어댑터가 200 MB/s에서만 오류 없이 작동했으며, 그 이상 속도에서는 간헐적으로 프레임이 손실되었습니다. 성능 기록의 `wire_ceiling_source` 항목에 이 내용이 문구로 명시되어 있으며, `nic_is_usb` 항목에서 이를 표시합니다. `--wire-ceiling-mbps`를 사용하여 어느 쪽이든 재정의할 수 있습니다.
- **통과 허용은 드레인(drain)을 고려하며, 전체 버스트 대 링(ring) 방식이 아닙니다.** 동시 버스트는 전체 버스트가 아닌 *일시적 백로그* = `max(0, Σ per-cam arrival − host drain) × emit_window`에 맞기만 하면 됩니다. 고속 호스트/저속 CAM 패브릭(**PCIe**10G 호스트 + 4× 1 GbE CAM: 도착 ≈ 320 MB/s, 드레인 ≈ 1063 MB/s) 환경에서는 호스트의 배출 속도가 캠의 채움 속도보다 빠르므로 백로그 ≈ 0이 되고, 25 MB 버스트가 13.5 MB 링을 초과함에도 불구하고 풀 해상도 시뮬레이션 송신(sim-emit)이**허용**됩니다. Windows에서 동일한 4대의 카메라를 **USB**10GbE 어댑터 뒤에 연결하면 드레인 속도는 1063이 아닌 200 MB/s가 됩니다. 도착 속도가 드레인 속도를 앞지르기 때문에, 손실은 프레임 속도 저하가 아닌 손상된 프레임 형태로 나타납니다. 1 GbE 호스트에서는 카메라의 31.25 MB/s DLThr 하한값으로 인해 도착 속도가 배출 속도를 앞지르게 됩니다 → 시스템은 올바르게**차단**합니다(*이러한* 유형의 차단에 대해서는 ROI를 줄이거나 ≥ 2의 비닝을 사용하십시오). 허용 여부는 **두 가지** 연결 게이트 중 하나에 의해 결정되며, 다른 하나는 아래에 설명된 집계 과구독 검사입니다.
- **예상 fps는 가장 느린 카메라의 사이클입니다.**동시 송신(simultaneous emit) 시 모든 카메라가 동시에 프레임을 전송하므로, 어레이는 가장 느린 구성원의 센서 판독 시간과 송신 시간 중 더 긴 쪽에 맞춰 사이클을 돌게 됩니다(프레임 바이트 ÷ 해당 카메라의 송신 속도: 카메라의**액세스 링크**, 1 GbE ≈ 80 MB/s, 또는 호스트의 유선 대역폭 상한이 더 작을 경우 해당 상한에서 카메라가 차지하는 비율). 각 노출은 이전 프레임의 판독과 겹치므로, 노출은 사이클에 포함되지 않습니다: 자동 노출은 전송 속도가 결정하는 주기를 채운 후 게인을 높이며, 어두운 장면에서도 더 이상 전송 속도가 낮아지지 않습니다(1.2.0 버전에서는 속도가 낮아졌습니다). 해당 주기보다 긴 수동 노출만 트리거 속도를 늦춥니다([어레이 구성원의 노출](#exposure-on-an-array-member) 참조). 카메라를 추가하더라도 해당 카메라들의 와이어 공유량이 줄어들기 전까지는 어레이 속도가 느려지지 않습니다. 직렬 방출(`slip-emit-and-capture`)에서는 카메라들의 방출 시간이 합산되며, 각 구성원의 노출 시간은 스태거에 반영됩니다. PCIe 10 GbE 호스트를 사용한 9대 카메라 어레이에서 풀 센서, 12비트 조건으로 측정된 결과: 10.7 fps, 또는 [패킹 전송](#packed-12-bit-transport---packed) 사용 시 14.0 fps.
- **연속 캡처가 끝난 후, 예상 값은 캡처된 내용을 나타냅니다.** 자동 속도로 30초 이상 캡처하면 이 컴퓨터에서 해당 어레이 크기에 대해 안정화된 속도가 기록되며, 속도 제어기가 더 빠른 속도로 시도했던 짧은 기간의 기록은 제외됩니다. 그 이후부터 `array-connect`의 행과 ‘어레이 설정’ 패널은 해당 속도가 모델의 속도보다 낮을 때, 그 제한 요인과 함께 해당 속도를 표시합니다: `clock-bound`(어레이는 안정화된 속도로 실행되었으며, 그랩 루프가 트리거 클럭을 대기 중), `loop-bound` (그랩 루프에 남은 시간이 없음), `capture-bound` (캡처가 뒤처져 그룹을 건너뛴 경우) 또는 `measured` (어느 경우인지는 명시되지 않음). 모델의 수치는 `wire_fps_max`에 나와 있다. 어레이는 여전히 회선이 전달하는 속도까지 상승한다.
- **어레이는 어떤 네트워크에서도 작동합니다 — 프레임 레이트가 결정적인 요소입니다.**각 카메라는**충돌 방지 유선 상한선**(`sustained × sim_emit_factor`)을 GevSCPD에 맞춰 균등하게 분배받으며, 모든 카메라가 여전히 동일한 트리거에 반응합니다. 저속 회선에서는 할당량이 적어지고 프레임 속도도 그에 따라 떨어집니다. 점보 프레임이 없는 1GbE 환경에서 풀 해상도 12비트 카메라 8대를 연결하면 각 카메라당 52 / 8 = 6.5 MB/s를 얻게 되며, 이는**1 fps**(압축 시 약 1.3 fps)를 기록합니다. 반면 10G 호스트에서는 11~14 fps로 작동합니다. 카메라를 이 정도로 느리게 페이싱할 수 없는 경우 — 각 카메라의 GevSCPD 범위를 읽어 확인하십시오 — 어레이는**직렬**전송 모드로 전환됩니다 (`slip-emit-and-capture`) 모드로 전환됩니다: 카메라들은 전체 링크 속도로 선로를 번갈아 사용하며, 차례대로 트리거되므로 모든 프레임은 여전히 도착하지만 구성원들은**동시에 노출되지 않습니다**. `analyze-array` 및 connect 출력 값이 해당 모드를 나타내며, `--force-tier sim-capture-sim-emit`를 핀으로 고정하면 폴백을 거부할 수 있습니다. 프레임 크기는 이 공유 방식에 영향을 미치지 않으므로,**빈닝 및 더 작은 ROI는 프레임 속도를 높이겠지만 어레이가 수용 가능한지 여부는 변경하지 않습니다**. 직렬 방식으로조차 어레이를 전송할 수 없는 와이어만이**과부하 상태**가 됩니다. 예를 들어, 한 와이어가 처리할 수 있는 것보다 훨씬 많은 카메라가 있거나, 해당 와이어에서 스트리밍 중인 다른 카메라가 많은 경우입니다. 이때 나타나는 증상은 전송 속도 저하가 아니라 GVSP 패킷 손실입니다: `analyze-array`는 달성 가능한 fps 수치를 0으로 설정하고 `**OVER-SUBSCRIBED**`를 출력하며, 연결은 큰 경고와 함께 진행되고, `pin_resolution: true`(엄격 모드)를 동반한 HTTP 연결은 **거부**됩니다. 해결책으로는 카메라 수를 줄이거나, 종단 간 점보 프레임을 사용하거나, 더 빠른 NIC를 사용하는 것입니다. `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1`는 벤치마크 작업 시 이 거부를 강력한 경고로 완화합니다 — [환경 변수](#environment-variables)를 참조하십시오.

### 어레이 상태 — 어떤 하위 시스템에서 프레임이 손실되고 있는지

연결된 어레이의 `GET /api/camera/array/<array_id>/capability`는 실시간
`health` 블록을 포함하며, 이는 **10초** 단위의 롤링 창을 기준으로 재평가됩니다. 이 지표는 프레임 손실을
두 가지 원인으로 구분하여 각각 상반된 해결책이 필요한지 표시하며, 두 원인을 모두 명시하지 않은 단일 “불완전”
비율을 보고하는 방식과는 다릅니다:

| 필드 | 의미 | 해당 하위 시스템 |
| --- | --- | --- |
| `gvsp_corrupt_rate_pct` (시리얼별) | 프레임이 **도착했으나 구조적으로 결함이 있음**— GVSP 패킷 손실. |**네트워크**: 와이어 예산, 페이싱, NIC RX 링, MTU |
| `never_arrived_rate_pct` (시리얼별) | 프레임이 **전혀 도착하지 않음**— 카메라가 작동하지 않았거나, 카메라에서 데이터가 전송되지 않았음. |**트리거/싱크**: M8 케이블, `--line`, `TriggerMode` |
| `worst_gvsp_corrupt_pct` / `worst_never_arrived_pct` | 각 항목별 최악의 카메라 전송률. | — |
| `per_cam_rate_pct` | 카메라별 통합 미완료 전송률 (두 원인 합산). | — |
| `stable_for_seconds` | 각 카메라가 0.01% 미만 상태를 유지한 기간. | — |

5%를 초과하면 백엔드는 분할 이름을 명시한 `[array-health <id>] WARN` 행을 기록합니다. —
첫 번째 위반 시, 심각도 대역이 변경될 때, 상태가 지속되는 동안 1분마다 한 번씩, 그리고
상태가 해결될 때 한 번씩 기록합니다. 손상된 절반은 카메라 및 원인별 첫 번째 발생 시 백엔드 로그 파일에
`[gvsp-corrupt <SN>]`를 기록한 다음, 60초마다 집계된 값을 기록합니다. 앱의 로그 패널에는 대신
`[array-health <id>]` 행이 대신 표시됩니다. 모든 평가 결과는 여전히 백엔드 로그 파일에 기록되며,
출력된 내용과 관계없이 버퍼가 처리될 때마다 카운터가 증가합니다.

동일한 레코드에는 전체 할당량이 차지하는 숫자가 보고됩니다:

| 필드 | 의미 |
| --- | --- |
| `wire_ceiling_mbps` | 호스트의 현재 유효한 지속적 전송 대역폭, MB/s. |
| `wire_ceiling_source` | 해당 수치의 출처(문자열) — 예: `USB-capped 200 MB/s (was theoretical 1062; PnPDeviceID=USB\VID_0BDA&PID_815A)` (Linux x86_64의 경우, `USB-capped 400 MB/s (…; sysfs: bus=usb (usb2/2-1/2-1:1.0))`) 또는 `user override 120 MB/s (auto said 200)`. |
| `wire_ceiling_is_user_set` | `true`는 `--wire-ceiling-mbps`(또는 GUI의 **와이어 예산** 필드)에서 해당 값이 설정된 경우입니다. |
| `nic_is_usb` | USB 이더넷 어댑터의 경우 `true` — 위의 USB 제한 사항 참조. |

**해석:** `gvsp_corrupt_rate_pct`가 0이 아닌 상태에서 `never_arrived_rate_pct`가 0인 경우
는 트리거링과 케이블 동기화가 완벽하며 손실의 100%가 네트워크
경로에서 발생함을 의미합니다 — `--wire-ceiling-mbps` 값을 낮추고 다시 연결하십시오. 반대의 패턴은
대신 동기화 케이블이나 트리거 라인을 가리킵니다.

> **`--target-fps`는 프레임 손상의 원인이 아닙니다.** GevSCPD 페이싱은
> 연결 시 한 번만 설정되므로, 트리거 속도를 낮추면 듀티 사이클만 변경될 뿐
> 동시 송신 버스트 속도는 변경되지 않습니다. 측정 결과, 수요를 5배 줄여도 개선 효과가 없었으나;
> 와이어 상한선을 240 MB/s에서 200 MB/s로 낮추자 동일한 장비에서 손상률이 10.4 %
> 에서 0.00 %로 떨어졌습니다.

> **실행 중인 어레이는 LATTICE(M3C / M3M) 펌웨어에서 자체적으로 재계획할 수 없습니다.** 이
> 문제는 자체적으로 해결할 수 없습니다. 연결을 끊었다가 다시 연결하여 연결 시간 선택기가
> 새로운 상한값을 반영해 재계획할 수 있도록 해야 합니다.

### 증상 → 해결 방법

| 증상 (어레이 설정 / 연결 / `analyze_array_network`) | 원인 | 해결 방법 |
| --- | --- | --- |
| `FRAMES WILL DROP … exceeds NIC RX ring usable capacity 0.26 MB`, `Reduce ROI to enable` | `ReceiveBufferLen`가 32로 재설정됨 (일반적으로 드라이버 업데이트 후) | `ReceiveBufferLen`→256, `PendingReceives`→64; 패널을 다시 열기 (백엔드가 이전 링 크기를 캐시한 경우 백엔드를 재시작) |
| 재시작/종료 시 멈춤; 이후 `Invalid class` WMI 오류, NIC 활성화 불가, USB 드라이브 미인식 | 구형 2016 Realtek USB 10GbE 드라이버 → BSOD `0x9F` → 강제 전원 끄기 | 어댑터 드라이버를 v10.67 (2026)로 업데이트한 후, 위의 수신 링 설정을 다시 적용 |
| 어레이는 센서 최대값으로 연결되지만 프레임 속도가 낮음 | ‘자동’ 모드는 센서 최대값을 유지하면서 속도가 회선 용량을 따라 떨어지게 함 | 연결 출력 목록에 있는 더 빠른 옵션 중 하나를 선택하십시오(`--binning 2` / `4`, 또는 더 작은 `--width`/`--height`) 중 하나를 선택하거나, 점보 프레임을 활성화하거나, 링크를 업그레이드하십시오 |
| 어레이가 정상적인 목표 fps를 보고하지만 실제로는 그 일부만 전달하는 경우; `health.gvsp_corrupt_rate_pct`가 0이 아닌 경우, `never_arrived_rate_pct` 0 | 호스트의 추정된 와이어 예산이 실제 유지 가능한 양보다 과대 평가됨(USB 이더넷 어댑터, 얇은 PCIe 레인 또는 공유 패브릭에서 흔히 발생) | 더 낮은 `--wire-ceiling-mbps` 값으로 재연결한 후 상태 블록을 다시 확인하십시오. **아님** `--target-fps` — GevSCPD 페이싱이 연결 시 고정됨 |
| Linux: NIC가 패킷 손실을 보고하지 않음에도 배열이 프레임 그룹을 잃고 프레임 속도를 낮추는 경우; `lattice network`에 `socket receive buffers too small`가 표시됨 | 커널의 소켓 수신 버퍼가 기본값으로 설정되어 있어 카메라 패킷이 커널에서 드롭됨 | `net.core.rmem_max` / `rmem_default`를 32 MB로 늘리십시오 — [Linux 호스트](#linux-hosts--socket-receive-buffers)를 참조하십시오. .deb 패키지는 설치 시 이 작업을 수행합니다 |
| 공개된 그룹에서 카메라가 누락됨; `health.never_arrived_rate_pct`가 0이 아님, `gvsp_corrupt_rate_pct` 0 | 트리거/싱크 경로 — 카메라가 작동하지 않음, 네트워크 문제가 아님 | M8 싱크 케이블 및 `--line`를 확인하고, 모든 구성원이 무장 상태인지 확인하십시오 (`TriggerMode=On`) |
| `**OVER-SUBSCRIBED**` / `Wire budget`가 `analyze-array`에서 초과되었거나, 엄격 모드에서 연결 거부 (`pin_resolution`; `array over-subscribes the wire`) | 카메라가 도달하는 어떤 전송 속도에서도, 동시에 또는 차례대로 배열을 전송할 수 없음 — 케이블이 공유할 수 있는 것보다 훨씬 많은 카메라가 있거나, 다른 카메라가 해당 케이블을 통해 스트리밍 중임 | 카메라 수를 줄이거나, 종단 간 점보 프레임을 사용하거나, 더 빠른 NIC를 사용하십시오. **ROI/비닝은 도움이 되지 않습니다** (공유는 프레임 크기와 무관합니다). `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1`는 벤치 테스트에서 우선 적용됩니다(패킷 손실을 허용함) |
| 느린 네트워크에서 `slip-emit-and-capture`(직렬)를 통해 어레이가 연결됨 | 카메라들의 전송 속도를 동시에 선로를 공유할 수 있을 만큼 충분히 낮출 수 없으므로, 카메라들은 번갈아 가며 전송하며, 프레임은 동시에 노출되지 않음 | 점보 프레임이나 더 빠른 NIC를 사용하면 동시 캡처가 복원됨; `--force-tier sim-capture-sim-emit`는 이를 고집하며(대신 패킷 손실이 발생함) |

---

## `chloros-cli daq`

스펙트럼 센서 명령어. `pool-*` 명령어는 백엔드의 지속적 풀을 통해 센서를 구동하는 경량 HTTP클라이언트이므로, 실행 중인 백엔드가 필요합니다([백엔드 시작](#start-the-backend) 참조). 백엔드가 전송 경로를 관리하므로, CLI와 SDK 스크립트는 시리얼 포트를 놓고 경쟁하지 않고 하나의 활성 핸들을 공유합니다. `pool-discover`는 DAQ-M의 BLE MAC을 파악할 수 있는 유일한 방법입니다.

> **`chloros-cli daq --help`** (또는 `-h`, 또는 `daq help`)는 `pool-*` 하위 명령어 목록을 표시하며(Linux의 경우, 빌드에 포함된 직접 하드웨어 명령어와 함께), 그리고 `chloros-cli daq <subcommand> --help`(또는 `daq help <subcommand>`)는 각 명령의 옵션을 제공합니다. 여기에는 연결, 스트림, 보정된 `.daq` 파일 기록, 캡 프로파일 교환, DAQ를 카메라에 바인딩하는 기능을 포함하며, DAQ-E의 경우 네트워크 설정, PTP, 재시작 및 펌웨어 업데이트 기능을 다룹니다. 또한 `chloros_sdk.connect_daq_sensor()`를 사용하여 Python에서 DAQ를 제어할 수 있으며, 이 명령은 동일한 풀링된 경로를 사용합니다.
>
> **모든 플랫폼에서 `pool-*` 명령을 사용하십시오.** 이 명령은 Windows, Linux 및 Jetson에서 동일하게 작동합니다. Windows에서는, `daq discover`와 `daq list`는 `pool-discover`의 별칭이며, 이 아닌 다른 모든 `daq` 명령어는2567가 아닌 다른 모든 `daq` 명령어는 `1`를 `DAQ direct-hardware commands require the full daq package`로 종료합니다. Linux와 Jetson에서는, `daq discover`, `daq list` 및 `pool-*`가 아닌 다른 `daq` 명령어는 DAQ 도구의 직접하드웨어 명령어입니다. 이 명령어들은 CLI 자체 프로세스 내에서 센서를 열며, 백엔드 풀 외부(즉, 백엔드, GUI 또는 스크립트가 보유한 센서가 아님)에서 센서를 열고, `daq discover`는 `--only` 및 `--timeout`를 수신하지만 `--json`는 지원하지 않습니다. 이 참조 문서는 `pool-*` 명령어에 대해서만 설명합니다.

### DAQ 센서 최초 연결 워크플로우

```bash
# 1. Scan every transport, showing the address to connect with.
#    This is how you find a DAQ-M's BLE MAC — unlike a DAQ-E hostname or a
#    DAQ-U serial port, a MAC isn't printed on the device or listed by the OS.
chloros-cli daq pool-discover
chloros-cli daq pool-discover --only ble           # BLE only
chloros-cli daq pool-discover --json               # machine-readable

# 2. Open a persistent pool session (handle stays alive across CLI calls)
chloros-cli daq pool-connect                                   # smart-detect
chloros-cli daq pool-connect --port /dev/ttyUSB0               # DAQ-U on a serial port (Windows: COM3)
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF           # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-xxx.local        # DAQ-E by hostname

# 3. List what's in the pool, including the sensor_id you'll use next
#    (DAQ-U ids look like 'CB-7C-A8-2E-5F'; DAQ-E ids like 'daq-e-def330')
chloros-cli daq pool-list

# 4. Read the latest spectrum frame
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F

#    Bind it to a camera, so that camera's captures use and save its readings
chloros-cli daq pool-bind --camera 213800234 CB-7C-A8-2E-5F

# 5. Record a calibrated .daq file for 60s
chloros-cli daq pool-record --sensor-id CB-7C-A8-2E-5F --duration 60 \
  -o ~/Documents/spectra --device-name "field-A"

# 6. Release
chloros-cli daq pool-disconnect --sensor-id CB-7C-A8-2E-5F
```

### `pool-*` 참조

| 하위 명령 | 용도 |
| --- | --- |
| `daq pool-connect` (스마트 감지) | 백엔드 풀에서 센서를 엽니다. |
| `daq pool-connect --port PORT` (`-p`) | 특정 직렬 포트의 DAQ-U. |
| `daq pool-connect --ble` | BLE를 통한 DAQ-M, MAC 자동 스캔. |
| `daq pool-connect --mac MAC` (`-m`) | 알려진 MAC 주소를 사용하는 BLE 기반 DAQ-M (`--ble`를 포함함). |
| `daq pool-connect --eth-host HOST` | 알려진 호스트명 또는 IP를 사용하는 이더넷 기반 DAQ-E (`--eth`를 암시함). |
| `daq pool-connect --eth` | 이더넷을 통한 DAQ-E, 호스트 자동 탐색 (mDNS + ARP 폴백; Windows 및 Linux에서 비활성화된 ARP 캐시 상태에서도 작동). |
| `daq pool-connect --integration-time MS (-t) --frame-avg N (-f) --no-ae` | 적분 창/AE 상태 조정. |
| `daq pool-connect --no-stream` | 연결은 하되 스트리밍은 아직 시작하지 않음 (`pool-stream --start`로 재개). |
| `daq pool-connect --cap-id {none, fov_15, fov_30, fov_45, fov_60, fov_90, sunshine_cosine}` | 캡 보정 프로필. 백엔드의 기본값은 `sunshine_cosine`이며; `none`는 베어 센서입니다. [DAQ-E-S](#daq-e-s--fixed-diffuser-and-tilt-sensor)는 이를 무시합니다(고정형 디퓨저). **센서는 베어 상태로 출하되므로, 장착한 캡의 이름을 지정하십시오:** 기본값은 (`cap_id_source=auto_default`)로 간주되어 기록되며, 햇빛 보정 상태의 베어 DAQ-U는 측정값이 약 30배 높게 나오므로 처리 과정에서 이에 대한 경고가 표시됩니다. |
| `daq pool-discover [--only usb,ble,eth] [--timeout SEC] [--json]` | 연결하지 않은 상태에서 연결 가능한 센서가 있는지 모든 전송 경로를 스캔합니다. **이것이 DAQ-M의 BLE MAC을 찾는 방법입니다.** `--timeout`는 전송 방식별로 적용됩니다(기본값 5초, 1~20). 풀에 이미 열려 있는 센서는 목록에 표시되지 않습니다(연결된 DAQ-M은 광고를 중단하기 때문). 따라서 해당 센서를 확인하려면 `pool-list`를 사용하십시오. `usb`는 USB 직렬 장치만 나열합니다: 컴퓨터 자체의 직렬 포트(`COM1`, Linux의 `/dev/ttyS*`)는 센서가 될 수 없으므로 제외됩니다. |
| `daq pool-list` | 백엔드 풀에 있는 모든 센서를 표시합니다. |
| `daq pool-bind (--camera SERIAL \| --array ARRAY_ID) {SENSOR \| --clear}` | GUI의 광센서 선택 기능과 마찬가지로 카메라나 배열을 DAQ에 바인딩합니다: 캡처 시 해당 DAQ의 측정값을 사용하고, 일치시키며, 저장합니다. `SENSOR`는 `pool-list`의 센서 ID이거나 DAQ에 접근하는 주소입니다. `--clear` (`SENSOR` 없음)은 바인딩을 해제하며, 캡처는 가장 최근의 DAQ로 되돌아갑니다. 바인딩은 백엔드가 재시작될 때까지, 또는 해당 카메라, 어레이 또는 DAQ가 연결 해제될 때까지 유지됩니다: 연결 해제(여기서는 SDK에서 또는 앱 내에서)는 연결이 끊긴 하드웨어에 대한 모든 바인딩을 해제하며, 누가 설정했든 상관없이 어레이 구성원의 바인딩도 어레이와 함께 해제됩니다. 카메라는 열려 있는 프로젝트에 연결되어 있거나, 이 컴퓨터에서 이전에 열려 있어야 하며, 어레이도 연결되어 있거나 열려 있는 프로젝트에 포함되어 있어야 합니다. 이름과 다른 시리얼 번호나 ID는 거부되며, 아무것도 바인딩되지 않습니다. |
| `daq pool-disconnect {--sensor-id ID \| --all}` | 해제 — 센서를 연 사용자가 누구든 상관없이, 따라서 크래시된 스크립트가 풀에 남겨둔 센서도 해제합니다(해당 센서의 모든 녹화는 먼저 완료되어야 함). 풀에 없는 센서는 해제할 대상이 없습니다: `ID: already released (not in the pool)`를 출력하고 종료됩니다(`0`); 백엔드가 해제 요청을 거부했거나 확인하지 않은 경우 `1`를 출력하고 종료됩니다. SDK: `DAQSensorSession.force_disconnect()`. |
| `daq pool-latest --sensor-id ID [--recent N] [--json]` | 가장 최근의 N개 스펙트럼 프레임(기본값 1). |
| `daq pool-stream --sensor-id ID [--start \| --stop]` | 스트리밍 재개/일시 중지. |
| `daq pool-record --sensor-id ID [--duration SEC] [-o DIR] [--device-name NAME] [--confirm-after S] [--heartbeat S]` / `--stop` | .daq 녹화 시작/중지. `-o`는 백엔드 컴퓨터에서 `~/Documents/DAQ Live View`로 기본 설정됩니다(상대 경로는 명령을 실행하는 폴더 아래에 있음); `--duration` (`-d`, 기본값 `0` = `--stop`까지). `--confirm-after S`는 시작 후 S초가 지나도 측정값이 도착하지 않으면 경고를 표시합니다(기본값 20; 0으로 설정하면 비활성화됨); `--duration`를 사용하면, `--heartbeat S`는 S초마다 측정값 개수를 출력하고, 증가가 멈출 때 경고를 표시합니다(기본값 30; 0으로 설정하면 비활성화됨). 소리가 나지 않는 DAQ-E에서 데이터 전송이 중단된 기록은 백엔드가 다시 연결되면 동일한 파일에서 계속되며, `--stop`는 데이터 공백 구간을 나열합니다([데이터 전송이 중단된 DAQ-E](#a-daq-e-that-stops-delivering)). Ctrl+C를 누르면 `--duration` 녹화를 조기에 종료하고 이를 보고합니다; `--duration`가 없으면 Ctrl+C(`130` 종료) 후에도 `--stop`까지 녹화가 계속 실행됩니다. |
| `daq pool-set-cap --sensor-id ID --cap-id CAP` | 실행 중에 캡 보정 프로필을 교체합니다. `--cap-id`가 필수이며, `--cap-id none`는 캡 보정을 제거합니다 (베어 센서). DAQ-E-S는 설정된 값을 `as_recorded`로 유지합니다. |
| `daq pool-enable-ptp --sensor-id ID` | PTP가 꺼져 있는 DAQ-E의 PTP를 다시 켭니다. 이 설정은 장치에 저장되므로, 따라서 PTP가 꺼진 상태로 방치된 장치는 이 명령을 실행할 때까지 계속 꺼진 상태로 유지되며(타임스탬프는 PTP와 동기화되지 않음). `pool-list`는 이러한 장치를 표시(`!! PTP is switched OFF`)하고 해당 장치에 대해 이 명령을 출력합니다. Chloros는 장치가 의도적으로 꺼져 있을 수 있으므로, 절대로 장치를 스스로 다시 켜지 않습니다. |
| `daq pool-info --sensor-id ID [--json]` | DAQ-E의 자체 설정: 펌웨어(및 이 Chloros릴리스에 더 새로운 버전이 포함되어 있는지 여부), 주소 및 주소 획득 방식 (DHCP / 링크 로컬 / 고정), 센서가 해당 명령을 실행할 때까지 저장된 네트워크 설정(`--no-reboot`와 함께 저장된 설정 포함), 전원 켜짐 시 스트림, 그리고 PTP. 스트림이 이 컴퓨터에 도달하지 않는 센서의 경우, 새로운 연결 시 센서 정보를 읽어들이고 이를 알립니다. |
| `daq pool-set-network --sensor-id ID {--auto \| --static IP [--mask M] [--gateway G]} [--no-reboot] [--force] [--timeout S]` | DAQ-E를 DHCP (`--auto`) 또는 고정 IP 주소(`--mask`, 기본값 `255.255.255.0`)로 설정합니다. 센서는 설정을 적용하기 위해 재시작되며, 기존 설정을 유지한 채 최대 `--timeout`초(기본값 45) 동안 재연결을 기다립니다; `--no-reboot`는 설정만 저장합니다. [DAQ-E 장치 설정](#daq-e-device-settings)을 참조하십시오를 참조하십시오. |
| `daq pool-set-auto-stream --sensor-id ID {--on \| --off}` | 소프트웨어가 연결되지 않은 상태에서 DAQ-E가 전원을 켤 때 스트리밍을 시작할지 여부(센서에 저장됨). Chloros는 어느 경우든 자체적으로 스트리밍을 시작합니다. |
| `daq pool-reboot --sensor-id ID [--timeout S]` | DAQ-E를 재시작하고 이전 설정으로 다시 연결합니다 (최대 `--timeout`까지 대기, 기본값 45초). |
| `daq pool-update-firmware --sensor-id ID [--timeout S]` | 이전 버전을 실행 중인 센서에 이 Chloros릴리스에 포함된 DAQ-E 펌웨어를 플래시한 후 다시 연결합니다 (약 1분 소요; 전원을 끄지 마십시오; `--timeout`가 다시 작동할 때까지 기본값 45초 대기). 이미 최신 버전을 사용 중인 센서는 그대로 둡니다. |

### 데이터 전송이 중단된 DAQ-E

풀에 있는 DAQ-E가 20초 동안 스펙트럼 데이터를 전송하지 않으면 백엔드에서 해당 센서를 재연결합니다. 센서에 전원을 공급하는 스위치의 전원을 껐다 켜면 이 작업이 수행됩니다. 센서는 스위치와 함께 재시작되며, 명령을 받을 때까지 다시 데이터를 스트리밍하지 않습니다 (`pool-set-auto-stream --on`인 경우는 제외).

- **먼저 동일한 센서인지 확인합니다.** 센서가 가지고 있던 주소를 그대로 사용하여, 아무것도 변경하지 않은 채 센서의 이름, MAC 및 일련번호를 조회합니다. 또한 센서의 `daq-e-<id>.local` 이름으로 쿼리하지만, 해당 응답이 센서 자체의 네트워크 어댑터(MAC)에서 온 경우에만 이를 신뢰합니다. 그 외의 응답은 재연결되거나 데이터가 기록되지 않습니다.
- **세션 설정(통합 시간, 프레임 평균화, 자동 노출 및 캡 프로필)을 그대로 유지한 채 재연결합니다. 센서의 전원 켜짐 시 기본값이 아닌, 이전 설정 그대로 적용됩니다. PTP가 켜져 있던 센서가 꺼진 상태로 돌아온 경우, PTP를 다시 켭니다. 그런 다음 스트림과 이를 수신하는 컴퓨터의 수신기를 다시 시작합니다.
- **점차 간격을 늘려가며 재시도합니다:** 즉시, 그 다음 5, 10, 20, 40초 후, 그리고 센서가 연결되지 않은 동안 60초마다 재시도합니다. 각 시도에서는 센서의 신원만 확인합니다. 응답은 하지만 세 번의 재연결 후에도 데이터를 전송하지 않는 센서는 그대로 두고 (`auto-reconnect GAVE UP`)로 보고됩니다. `pool-stream --start`는 다시 시도합니다.
- **`pool-list`는 현재 상태를 표시합니다.** 센서에서 응답이 없는 동안, `STREAM FAULT` 줄은 재연결 진행 상황을 나타냅니다. 첫 번째 스펙트럼이 표시되면 해당 행이 지워지고, 센서는 `reconnected automatically N time(s)`로 표시됩니다.
- **동일한 `.daq`에서 기록이 계속됩니다.** 파일은 각 간격을 두 행으로 표시합니다: 간격 직전 마지막 스펙트럼에서는 `event_type` 5, 간격 직후 첫 번째 스펙트럼에서는 6을 표시하며, 각각 타임스탬프만 포함됩니다. `pool-record --duration`는 센서가 없는 동안 `!! STREAM LOST …`를 출력하고, 센서가 돌아오면 `stream resumed after …`를 출력합니다. `pool-record --stop`는 각 간격을 나열하고, 빈 파일의 경우와 마찬가지로 각 간격을 나열하고 경고를 표시합니다(측정값이 없는 기록은 `1`로 종료됩니다).
- **기록은 하나의 보정값만 유지합니다.** 기록이 진행 중일 때 센서가 다른 보정값으로 복귀하더라도, 기록의 나머지 부분은 시작 시의 보정값을 유지하며, 중지 경고에 이 사실이 명시됩니다. `pool-disconnect` + `pool-connect`를 실행하면 새로운 보정값이 적용됩니다.
- **사용자의 명령이 우선합니다.** `pool-disconnect`로 중지하거나 `pool-stream --stop`로 일시 정지한 센서는, 재연결이 이미 진행 중일지라도 다시 연결되지 않습니다.
- **응답이 없는 센서에 대한 `pool-info`**는 새로운 연결을 통해 센서를 읽은 후 `!! stream NOT reaching this computer`를 추가합니다. 응답하지 않는 센서는 즉시 보고됩니다. `pool-list`는 더 이상 응답을 중단한 센서에 대해 PTP 상태를 표시하지 않습니다.
- DAQ-U 및 DAQ-M은 해당되지 않습니다. DAQ-M은 블루투스를 통해 자동으로 재연결되며, USB에서 분리된 DAQ-U는 `pool-connect`를 다시 실행해야 합니다.

### DAQ-E-S — 고정형 디퓨저 및 기울기 센서

**DAQ-E-S**는 소프트웨어 측면에서 두 가지 중요한 차이점이 있는 DAQ-E 모델입니다. 그 외의 모든 기능(탐지, 연결, PTP, 기록, 아래의 장치 설정 등)은 DAQ-E와 정확히 동일하게 작동합니다.**이 장치의 디퓨저는 고정형이므로 캡 보정이 필요하지 않습니다.** 선샤인(Sunshine) 스타일의 디퓨저는 내장되어 있으며, 공장 교정 측정 시 이미 장치에 장착된 상태였기 때문에 해당 교정값은 이미 정확한 하향 복사 조도를 나타냅니다. Chloros는 요청 사항에 관계없이 캡을 `as_recorded`(프로필 없음)로 설정합니다: 이 장치 이름을 딴 캡(`sunshine_cosine`, `fov_*`, `none`)은 경고 메시지와 함께 무시되며, 기록 데이터에는 `cap_id_source=model`. 따라서 이 센서에 대해서는 ‘가정된 캡’ 경고가 절대 발생하지 않습니다. 수동으로 데이터에 캡을 적용하지 마십시오: 상단에 `sunshine_cosine`를 적용하면 측정값이 약 11배 높게 나옵니다.

**이 센서에는 3축 가속도계가 탑재되어 있습니다.** 햇빛 확산기의 코사인 응답은 수평을 유지할 때만 유효하므로, 기울어진 센서는 잘못된 조도 값을 보고하며 — 반사율도 이 오류를 그대로 반영합니다 — 다른 증상은 나타나지 않습니다. 따라서 각 DAQ-E-S 측정값에는 해당 센서의 자세 정보가 포함됩니다:

| 필드 | 의미 |
| --- | --- |
| `tilt_deg`, `roll_deg`, `pitch_deg` | 각도(도). 펌웨어가 각도를 정의되지 않음(자유 낙하, 비활성 축)일 때 |
| `x_mg`, `y_mg`, `z_mg`, `mag_mg` | 밀리-g 단위의 가속도. `mag_mg` 값이 ~1000에서 크게 벗어날 경우, 장치가 이동 중이었으며 해당 샘플에 대한 각도 값은 의미가 없음을 의미합니다. |
| `cal_applied` | 설정: 축과 각도는 디퓨저 좌표계에 기반합니다(장치의 장착 보정이 적용됨). 해제: 원시 보드 좌표계로, 실제와 다른 최대 ~2.3°의 기울기가 표시될 수 있습니다. |
| `sample_age_ms` | 스펙트럼이 전송되었을 당시 가속도계 샘플의 시간. |
| `temp_raw` | 보정되지 않은 다이 온도 **추세** — 차이값만 표시되며, 절대 온도계 측정값이 아닙니다. |

`daq pool-latest --json`는 각 프레임의 `imu` 키(DAQ-U, DAQ-M 또는 일반 DAQ-E에서는 `null`). DAQ-E-S에서 기록된 `.daq`는 동일한 값을 `imu_*` 열(`imu_tilt_deg`, `imu_roll_deg`, `imu_pitch_deg`, `imu_mag_mg`, `imu_cal_applied`, …) 열에 동일한 값을 저장하므로, 독자는 구름과 삼각대 충격으로 인한 오류를 구분할 수 있습니다. 다른 모든 모델에서는 해당 열이 비어 있습니다.

자세 데이터 스트림은 별도로 보고됩니다. `daq pool-list`는 시스템이 다운된 동안 `!! ATTITUDE STREAM FAULT (spectra unaffected)`를 출력하며, 이후 복구한 오류 건수도 함께 표시합니다. 이 부분의 오류는 스펙트럼 기록에 공백을 남기지 않습니다. `daq pool-discover`는 해당 장치를 `DAQ-E-S`로 표시합니다(약 2초 이내에 신원 확인에 응답하지 않으면 `DAQ-E`로 표시됨).

### DAQ-E 장치 설정

```bash
chloros-cli daq pool-info --sensor-id daq-e-def330

# A fixed address on the network this computer is on (192.168.1.x)
chloros-cli daq pool-set-network --sensor-id daq-e-def330 \
  --static 192.168.1.50 --gateway 192.168.1.1

# Back to the factory setting: DHCP, falling back to a 169.254.x.x
# link-local address when no DHCP server answers
chloros-cli daq pool-set-network --sensor-id daq-e-def330 --auto

chloros-cli daq pool-update-firmware --sensor-id daq-e-def330
```

- **`--static`에 `--gateway`가 없으면 게이트웨이가 저장되지 않습니다**(이전에 설정된 게이트웨이는 모두 삭제됨). 이는 라우터가 없는 카메라 네트워크에 적합한 설정입니다.
- **이 컴퓨터의 네트워크에 고정 IP 주소가 있어야 합니다.** Chloros는 센서 네트워크에서 이 컴퓨터의 주소가 포함되지 않은 서브넷을 거부합니다. 재시작 후에는 센서에 접근하여 설정을 되돌릴 수 없게 됩니다. 이 컴퓨터가 새 서브넷으로 재할당되거나 라우팅될 때만 `--force`를 전달하십시오.
- **센서가 녹화 중인 동안에는 재시작이 허용되지 않습니다.** 먼저 녹화를 중지하십시오(`pool-record --stop`). 재시작 직전에 시작된 녹화는 재시작으로 인해 종료되며, `pool-set-network`, `pool-reboot` 및 `pool-update-firmware`는 녹화 파일 이름을 명시한 `Warning:` 줄로 끝납니다 (센서가 다시 연결되지 않는 경우에도 마찬가지입니다).
- **재시작 후 Chloros는 센서 자체만 다시 연결합니다.** Chloros는 센서가 있던 위치에서 — 그리고 `--auto` 이후에는 `daq-e-<id>.local` 이름으로도 — 센서의 이름, 호스트명 및 모델이 포함된 응답을 기다린 후, 해당 센서가 가지고 있던 네트워크 어댑터(MAC)를 통해 응답한 주소로 다시 연결합니다. 해당 이름으로 전송된 응답은 오직 해당 MAC 주소에서만 수락되므로, 라우터를 통해 센서에 접속하는 컴퓨터나 기본 NAT 네트워킹 환경의 WSL2에서는 DHCP에 의해 새 주소로 이동된 센서가 재연결되지 않습니다: 명령어가 이를 알리고, 센서가 가지고 있던 캡 및 노출 설정으로 센서를 다시 불러오는 `pool-connect --eth-host <its address> …` 명령어를 출력합니다.
- **`pool-update-firmware`는 펌웨어 버전을 읽을 수 없는 센서를 거부합니다.** 플래시 후 대기 시간보다 오래 걸려 응답하는 센서는 오류가 발생한 것이 아닙니다. 응답이 오면 새로운 펌웨어 설정이 적용된 상태로 재연결되며, 명령어는 응답이 느렸음을 경고합니다.
- **펌웨어 업데이트에는 인바운드 TCP 연결이 필요합니다.** 센서는 이미지를 수집하기 위해 이 컴퓨터에 다시 연결됩니다. Chloros는 센서로 연결되는 어댑터에서 수신 대기하며(어느 어댑터인지 식별할 수 없는 경우 모든 어댑터에서 대기), 해당 센서의 주소에만 서비스를 제공합니다. 그 외의 모든 연결은 서비스되지 않은 채 종료됩니다. Windows에서는 설치 프로그램의 `Chloros Backend (TCP-In)` 방화벽 규칙이 이를 허용합니다([Windows (.exe)](#windows-exe) 참조); 이를 차단하는 방화벽을 사용할 경우 `… device never connected back on TCP <port> …` 오류로 인해 업데이트가 실패합니다.
- **WSL2 환경에서는 미러링 네트워킹을 사용하십시오** (`networkingMode=mirrored` 및 `%UserProfile%\.wslconfig`). 기본 NAT 모드에서는 센서 네트워크 상의 이 컴퓨터 주소가 WSL 자체 주소이므로, 모든 고정 주소가 거부되고(`--force` 오류 제외), 라이브 스트림이 WSL에 도달하지 못하며, 센서가 이미지를 수신하기 위해 컴퓨터에 다시 연결해야 하기 때문에 펌웨어 업데이트가 완료되지 않습니다.
- **센서 ID와 호스트 이름은 변경되지 않습니다.** 주소가 무엇이든 DAQ-E는 여전히 `daq-e-<id>` 및 `daq-e-<id>.local`이므로, `--eth-host daq-e-<id>.local`는 계속해서 이를 찾아냅니다.
- **GUI의 설정은 동일합니다**: '광 센서' 탭, 센서 설정(톱니바퀴 아이콘), '네트워크'. GUI는 센서와의 자체 연결을 유지하므로, GUI에서 열려 있는 센서 설정을 변경할 때는 다음을 유의하십시오: CLI에서 재시작하면 GUI의 해당 센서 세션이 종료됩니다.
- 이에 상응하는 SDK 값은 `DAQSensorSession.device_info()`, `set_network()`, `set_auto_stream()`, `reboot()` 및 `update_firmware()`입니다.

---

## `chloros-cli project`

저장된 Chloros 프로젝트 (`cameras.json` + `sensors.json` + `project.json`가 포함된 폴더)를 엽니다, 연결하고 실행합니다. 모든 작업은 백엔드를 통해 처리되므로 GUI와 CLI는 동일한 하드웨어 상태를 생성합니다.

### 하위 명령어 참조

| 하위 명령어 | 용도 |
| --- | --- |
| `project open PATH [--relink FOLDER]` | 프로젝트의 장치 매니페스트(카메라, 어레이, 센서)를 출력하고, 해당 파일에서 데이터를 읽으며, 소스 파일을 확인합니다([이동된 프로젝트](#a-moved-project---relink) 참조). 이 명령은 백엔드에서 프로젝트를 열지 않습니다. 백엔드에 열려 있는 프로젝트와 그 장치 및 녹화 파일은 그대로 유지됩니다. 프로젝트의 소스 파일을 하나도 찾을 수 없는 경우 `1`로 종료됩니다. |
| `project devices PATH [--reconnect]` | 탐색 결과를 나열하거나 다시 실행합니다. |
| `project connect PATH [--cameras-only] [--sensors-only] [--packed {auto,on,off}]` | 저장된 모든 카메라/어레이/센서를 연결합니다. 각 어레이는 `lattice array-connect`와 동일한 방식으로 연결되며: 최대 600초까지 연결을 시도하며, 그 과정은 stderr에 출력됩니다(`[array <id>]    <s> s: <step>`; stdout은 JSON보고서입니다). 해당 시간 내에 연결되지 않은 어레이는 취소되며 — 해당 카메라들은 발견된 상태로 복원됨 — `errors` 항목 하에 보고되며, 연결된 상태로 남지 않습니다. `--packed`는 모든 어레이의 압축된 12비트 전송 방식을 설정합니다 (기본값 `auto`; 프로젝트는 어레이별 설정을 저장하지 않음; [압축된 12비트 전송](#packed-12-bit-transport---packed) 참조)로 설정합니다. 각 광 센서는 백엔드의 DAQ 풀에서 저장된 주소와 함께 저장된 적분 시간, 프레임 평균화, 자동 노출 및 캡 설정으로 열립니다: 캡이 선택되지 않은 센서는 `sunshine_cosine`를 할당받으며, '베어(bare)' 상태로 저장된 센서(`none`)는 그대로 유지되고, DAQ-E-S는 캡을 적용하지 않습니다. GUI의 DAQ 패널은 풀의 일부가 아니므로, 먼저 해당 패널에서 센서를 닫아야 합니다. |
| `project capture PATH NAME [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--prefix P] [--orient]` | 이름이 지정된 카메라 또는 어레이에서 단일 캡처(`--prefix` 기본값, `capture`). `--orient` (어레이): 어레이는 원시 데이터를 저장하며, 이 데이터는 네이티브 형식을 유지합니다. 이는 `process`가 이를 기반으로 생성하는 산출물이 `array-capture --orient`와 같이 방향이 지정되어야 함을 기록합니다. 기본값: 네이티브. `project burst` / `stream` 및 레시피의 `capture` / `burst` / `stream` (`orient: true`)는 동일한 옵션을 사용합니다. |
| `project burst PATH NAME [-n N] [-i S] [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--prefix P] [--orient]` | 지정된 카메라 또는 어레이에서 N 프레임 버스트 (`-n/--count` 기본값 5; `-i/--interval` 프레임 간 간격(초), 기본값 0). 어레이 버스트는 반복되는 동기화된 그룹을 중복 제거하므로(오래된 데이터 감시 기능), 부분적으로 작동하는 어레이가 하나의 프레임을 N개 복사하여 반환할 수 없으며; 반복마다 결과를 출력합니다. 캡처 도중 실패하면 버스트가 중단되고 `1`에서 종료되며, 그 전에 저장된 프레임 수를 표시합니다. |
| `project stream PATH NAME [-n N] [--fps F] [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--orient]` | 루프 내에서 디스크로 캡처: `--fps F`는 1초마다 (기본값 3), 카메라에서 캡처당 1프레임, 어레이에서 동기화된 프레임 그룹 1개(`project burst`와 동일한 유효성 감시 기능 적용)를 캡처하며, `-n N`가 캡처할 때까지 (기본값: `0` = Ctrl+C를 누를 때까지. 이때 저장된 프레임 수를 출력하고 `0`에서 종료하거나, 저장된 프레임이 없거나 파일 오류가 발생한 경우 `1`에서 종료; `-n N`가 캡처를 완료하기 전에 Ctrl+C를 누르면 `130`가 종료됩니다). 캡처가 도중에 실패하면 스트림이 중지되고 `1`가 종료되며, 그 전에 저장된 프레임 수를 표시합니다. `--poll-interval`는 더 이상 사용되지 않으며 무시됩니다: 여전히 허용되므로 구형 스크립트는 실행되지만, stderr에 경고 메시지가 출력됩니다. |
| `project sensor read PATH NAME [--json]` | 최신 스펙트럼 프레임. |
| `project sensor log PATH NAME [--seconds SEC] [-o DIR] [--device-name NAME]` | `--seconds`에 대한 .daq 파일 기록 (기본값 10). 측정값이 포함되지 않은 기록은 stderr에 보고되며 `1`에서 종료됩니다. |
| `project run PATH RECIPE.yaml` | YAML/JSON 캡처 레시피를 실행합니다. `--dry-run`는 실행하지 않고 유효성 검사만 수행합니다. |
| `project align calibrate PATH NAME [--method M] [--model M] [--frames N] [--reference SN] [--max-features N] [--ratio-threshold F] [--ransac-threshold-px F] [--min-matches N] [--max-reproj-err-px F] [--checkerboard RxC] [--name PROFILE]` | 배열에 대한 정렬을 계산합니다 — [아래 플래그 표](#project-align-calibrate-options)를 참조하십시오. |
| `project align status PATH NAME [--json]` | 현재 정렬 프로필을 출력합니다. |
| `project align clear PATH NAME` | 캐시된 프로필을 삭제합니다. |
| `project align tweak PATH NAME --serial SN [--dx N] [--dy N] [--rotation-deg N] [--scale N]` | 하나의 슬레이브의 변환을 미세 조정합니다 (기본값: `0`, `0`, `0`, `1.0`: 변경 없음). |
| `project align export PATH NAME --to FILE` | 프로필을 JSON으로 저장합니다. |
| `project align import PATH NAME --from FILE [--no-validate]` | 저장된 프로필을 불러옵니다. |

#### `project align calibrate` 옵션

| 플래그 | 기본값 | 설명 |
| --- | --- | --- |
| `--method {feature_orb, feature_akaze, phase_correlation, checkerboard, manual}` | `feature_orb` | 정렬 방법. **이 표기법은 `lattice align-calibrate`**와 다르며, `orb` / `akaze` / `phase`를 사용하며, 이 플래그에서는 두 명령을 서로 바꿔서 사용할 수 없습니다. |
| `--model {translation, rigid, affine, homography}` | `affine` | 모델을 맞추기 위해 변환합니다. |
| `--frames N` | `1` | 동기화된 프레임 스냅샷을 평균화. |
| `--reference SN` | 마스터 | 기준 카메라 일련번호; 나머지 모든 멤버는 이 카메라를 기준으로 변형됩니다. |
| `--max-features N` | `5000` | ORB 특징점 수 제한. |
| `--ratio-threshold F` | `0.75` | Lowe'비율 테스트. |
| `--ransac-threshold-px F` | `3.0` | RANSAC 내점 임계값. |
| `--min-matches N` | `15` | **품질 기준** — 이보다 적은 내점 일치 수일 경우 해를 거부합니다. |
| `--max-reproj-err-px F` | `4.0` | **품질 기준** — 이 RMS 재투영 오차 이상인 경우 해를 거부합니다. |
| `--checkerboard RxC` | — | `--method checkerboard`에 대한 보드 기하 구조, 예: `9x6`. |
| `--name PROFILE` | 비어 있음 | 저장된 JSON에 포함된 프로필 이름. **배열 이름이 아님** — 배열 이름은 위치 기반 `NAME`입니다. |

두 가지 품질 게이트가 바로 보정 작업이 해결에 성공했음에도 불구하고
저장을 거부하는 이유입니다. 두 가지 중 하나라도 실패한 프로필은 이후의 모든
캡처에서 아무런 경고 없이 정렬 오류를 일으키게 되므로, 저장되지 않고 거부됩니다.

### 이동된 프로젝트 (`--relink`)

프로젝트의 `project.json`는 각 소스 파일(모든 이미지와 광센서
기록)을 전체 경로로 명명합니다. 프로젝트 폴더나 전체 프로젝트 폴더를
다른 드라이브로 이동하면, 해당 경로들은 여전히 이전 위치를 가리킵니다.

프로젝트를 열면 앱이 자체적으로 수정 가능한 부분은 자동으로 수정됩니다. 즉, 프로젝트
폴더 내에 있던 파일은 프로젝트가 현재 위치한 폴더에서 검색되며, 그곳에서 발견된 모든 경로는
재설정되어 저장됩니다 (`project.json`는 먼저
`project.json.before-relink-<date>-<time>`로 백업됩니다). 이 작업은 앱을 열 때마다 —
`chloros-cli project open`, `chloros-cli process`를 기존 프로젝트에 열 때, SDK의
파일 및 `chloros-mcp`의 파일을 열 때마다 발생합니다. 경로는 해당 위치에 존재하는 파일에 대해서만 재지정됩니다.

프로젝트 폴더 외부에 있던 파일(허브 세션, ‘폴더 추가’를 통한 가져오기 등)은
이러한 방식으로 찾을 수 없습니다. 해당 파일이 현재 위치한 폴더의 이름을 지정하십시오:

```bash
chloros-cli project open "N:/Chloros Projects/Field_A" --relink "E:/backup/Hub Projects"
```

`--relink`는 파일이 이동된 폴더나 그 상위 폴더를 대상으로 합니다. 누락된
각 파일의 폴더는 파일 이름의 끝부분에 붙은 폴더 이름 순서대로
(`Hub Projects/<session>/raw/<serial>`, 다음으로 `<session>/raw/<serial>`, ...)을 기준으로 그 아래에서 각
폴더의 파일은 가장 많은 파일을 찾을 수 있는 한 곳에서 가져옵니다. `project open`는
재지정된 항목, 발견된 소스 파일 수, 그리고 나머지 파일이 있던
폴더를 출력합니다. 아직 누락된 항목이 있는 경우: 경고 메시지가 표시되고 `0`가 종료됩니다. 나머지 항목은 프로젝트가 정상적으로 열립니다.
찾을 수 없는 항목이 있는 경우: 해당 폴더 이름과 `--relink`를 명시하는 오류 메시지가 표시되고, `1`가 종료됩니다. 백엔드 역시
`process` 또는 `project devices --reconnect`에 대해서도 해당 프로젝트를 열지 못합니다(HTTP 409 `PROJECT_SOURCES_MISSING`).

### 예시

```bash
# Open a project and see what it knows about
chloros-cli project open "/home/user/Chloros Projects/Field_A"

# Its hub captures moved to another drive: point it at them
chloros-cli project open "/home/user/Chloros Projects/Field_A" --relink "/media/backup/Hub Projects"

# Connect everything saved in the project
chloros-cli project connect "/home/user/Chloros Projects/Field_A"

# Capture from a named camera (defined in cameras.json)
chloros-cli project capture "/home/user/Chloros Projects/Field_A" FrontLeft \
  -o output/ --format tiff

# Capture from a named array
chloros-cli project capture "/home/user/Chloros Projects/Field_A" main_rig \
  -o output/ --format tiff

# Capture with overrides
chloros-cli project capture "/home/user/Chloros Projects/Field_A" main_rig \
  --exposure 5000

# Read a spectrum
chloros-cli project sensor read "/home/user/Chloros Projects/Field_A" Sky --json

# Record a DAQ log
chloros-cli project sensor log "/home/user/Chloros Projects/Field_A" Sky \
  --seconds 120 -o ~/Documents/spectra/

# Align an array (live)
chloros-cli project align calibrate "/home/user/Chloros Projects/Field_A" main_rig
chloros-cli project align status "/home/user/Chloros Projects/Field_A" main_rig

# Run a recipe
chloros-cli project run "/home/user/Chloros Projects/Field_A" recipe.yaml
```

### 레시피 DSL

`project run RECIPE.yaml`는 일련의 작업을 설명하는 YAML 또는 JSON 파일을 입력으로 받아들입니다:

```yaml
# recipe.yaml
overrides:
  cameras:
    FrontLeft:
      exposure_us: 5000
      target_brightness: 80

stop_on_error: true
actions:
  - apply:
      name: FrontLeft
      settings:
        auto_exposure: "Off"
        gain_db: 6.0
        auto_gain: "Off"
  - wait: 2s
  - capture:
      name: FrontLeft
      output: pose_a/
      format: tiff
  - stream:
      name: main_rig
      count: 60
      fps: 5
      output: stream/
  - burst:
      name: main_rig
      count: 10
      interval: 0.5
      output: burst_a/
      format: tiff
  - sensor:
      name: Sky
      action: read
```

지원되는 작업: `apply`, `wait`, `capture`, `stream`, `burst`, `sensor`. `apply`는 `cameras.json`, `settings` 키를 가져옵니다 — `exposure_us`, `auto_exposure`, `gain_db`, `auto_gain`, `target_brightness`, … (이전 명칭인 `exposure_time`, `exposure_auto`, `gain`, `gain_auto`는 별칭으로 인정됩니다). 이 방식으로 설정된 노출, 게인 또는 목표 값은 사용자의 설정으로 유지됩니다: Chloros의 자동 노출 제어(DAQ 측정 노출 포함)는 이를 건드리지 않습니다. 수동 노출을 설정하면 게인은 기존 상태를 유지합니다: 자동 모드에서는 `gain_db` 또는 `auto_gain: "Off"`를 설정할 때까지 카메라가 목표 밝기에 맞춰 게인을 (전체 범위 내에서) 계속 조정합니다. 이후에 `auto_exposure: "Continuous"`를 입력하면 노출이 다시 자동 모드로 전환되고, `target_brightness_auto: true`를 입력하면 고정된 목표값이 Chloros의 자동 목표값으로 되돌아갑니다. `burst` 액션은 `name`(필수), `count`(기본값 5), `interval`(초, 기본값 0), `output`, `format`, 및 `settings`(`apply`와 동일한 카메라별 설정 형태)를 사용합니다. 어레이 버스트는 `project burst`와 동일한 새로 동기화된 그룹 워치독을 사용합니다.

`capture`, `burst` 또는 `stream` 단계의 `output`는 기본적으로 `output`로 설정됩니다. 상대 경로인 `output` — 기본값으로 포함됨 — 은 레시피 파일의 폴더나 프로젝트 폴더가 아닌, `chloros-cli project run`를 실행하는 폴더를 기준으로 해결됩니다(1.2.0 버전에서는 백엔드의 설치 폴더에 위치했습니다).

실행 방법:

```bash
chloros-cli project run "/path/to/project" recipe.yaml

# Dry-run to validate without firing hardware
chloros-cli project run "/path/to/project" recipe.yaml --dry-run
```

**매 실행 시마다 먼저 프로젝트에 대해 검증됩니다.**각 액션의 키는 반드시 위의 동사 중 정확히 하나여야 합니다 — `- capture: {name: FrontLeft}`여야 하며, `- action: capture`나 `- type: capture`는 안 됩니다 — `name`의 경우, 프로젝트의 `cameras.json` / `sensors.json`에는 카메라가, `apply` / `capture` / `burst` / `stream`용 어레이와 `sensor`용 광센서를 갖추고 있으며; `stream`는 `count` > 0이 필요하며, `wait`는 지속 시간이 필요합니다. 이러한 오류가 포함된 레시피는**아무런** 작업도 실행하지 않습니다 (`[recipe] error: …`, `[recipe] not run: …`, 종료 `1`), `stop_on_error: false`가 있더라도 마찬가지입니다. 단계에서 사용하지 않는 키는 `[recipe] warning:`이며, 레시피가 제어하지만 현재 연결되어 있지 않은 모든 배열이나 광 센서도 마찬가지입니다. 레시피는 절대 장치를 연결하지 않습니다: 먼저 `chloros-cli project connect PATH`를 실행하십시오. `--dry-run`는 레시피를 출력하고, 동일한 오류 및 경고 메시지를 표시한 뒤, 레시피에 오류가 발생하면 `[recipe] valid: …` / `NOT valid: …`가 표시되고, 레시피에 오류가 있을 경우 아무것도 실행하지 않은 채 종료됩니다.

---

## 환경 변수

| 변수 | 효과 |
| --- | --- |
| `CHLOROS_BACKEND_URL` | 백엔드 URL 재정의 (기본값: `http://127.0.0.1:5000`) — **`lattice`, `project` 및 `daq pool-*` 명령어 계열에서만 적용됩니다** (`lattice network-analysis`는 대신 `--backend-url`를 사용합니다). 핵심 명령어(`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`)는 `http://127.0.0.1:<port>`를 고정하고 이 변수를 무시하므로 (IPv4 리터럴은 Windows의 `localhost`→`::1` 요청당 약 2초의 성능 저하를 우회하므로), 항상 로컬 머신을 대상으로 합니다. |
| `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED` | 백엔드 환경에서 설정됩니다. `1`는 오버서브스크립션 거절 수준을 낮춥니다 — `pin_resolution`가 포함된 배열 연결 또는 [일시적 멀티 카메라 명령](#multi-camera-transient-sessions) (`error_code` `ARRAY_OVERSUBSCRIBED`) — 로 다운그레이드하여, GVSP 패킷 손실을 허용합니다. 테스트 용도로만 사용 — [어레이 fps 및 버스트 모델](#array-fps--burst-model)을 참조하십시오. |
| `CHLOROS_ARRAY_PACKED12` | 백엔드 환경에서 설정: `on` 또는 `off`는 모든 어레이에 대해 [압축된 12비트 전송](#packed-12-bit-transport---packed)을 기본값으로 설정합니다 (기본값: `auto`)의 기본값으로 [패킹된 12비트 전송](#packed-12-bit-transport---packed)을 설정합니다. 명시적으로 지정된 `--packed`가 여전히 우선합니다. |
| `CHLOROS_DISABLE_UPDATE_CHECK` | `1` (또는 `true` / `yes`)는 시작 시 Linux CLI의 [자동 업데이트 확인](#chloros-cli-update) 기능을 비활성화합니다. |
| `CHLOROS_CLI_MODE` | CLI에서 직접 설정하며, 백엔드에 병렬 처리를 활성화하도록 지시합니다. |
| `CHLOROS_GVSP_PROBE_FALLBACK` | `0`는 GVSP 폴백 프로브를 건너뜁니다 (ICMP 결과만 해당). **이 설정은 점보 패킷을 비활성화하며, 단순히 로그 출력을 줄이는 것이 아닙니다** — 카메라는 모든 경로에서 최대 1500까지의 DF 핑에만 응답하므로, 이 프로브만이 점보 패킷을 감지할 수 있습니다. 카메라당 연결 시 약 1초를 절약하지만, 네트워크가 점보 패킷을 전송할 수 있는 경우 선로 상한선의 약 1.45배에 해당하는 비용이 발생합니다. 이 옵션을 설정하면 SDK에서 경고가 표시됩니다. |
| `CHLOROS_GVSP_SESSION_REUSE` | 백엔드 환경에서 설정: `0`는 카메라당 백엔드 세션마다 한 번씩이 아니라, 연결할 때마다 패킷 크기를 측정합니다. |
| `CHLOROS_GVSP_PACKET_SIZE_FORCE` | GVSP 패킷 크기를 N 바이트로 고정합니다. 프로빙 과정을 완전히 건너뜁니다. 영구적으로 설정하는 것보다 명령어별 설정(`CHLOROS_GVSP_PACKET_SIZE_FORCE=9000 chloros-cli …`)을 권장합니다: 고정된 크기는 앞단 네트워크에 대한 적응을 중단시키며, 점보 패킷을 처리할 수 없는 경로에서 9000으로 고정하면 `SC_ERR_TIMEOUT -1011`를 사용할 때 **모든** 캡처가 타임아웃됩니다. |
| `CHLOROS_TRANSIENT_AE_SETTLE_S` | 백엔드 환경에서 설정: [일시적 멀티 카메라 명령](#multi-camera-transient-sessions) 또는 아무것도 유지하지 않는 카메라의 `lattice capture`가 프레임을 유지하기 전까지 각 카메라의 노출을 설정하는 데 소요되는 최대 시간(초)입니다(기본값 `5`). `0`를 설정하면 이 기능이 비활성화됩니다: 이때 각 카메라의 자동 노출 기능은 첫 번째 프레임부터 해당 카메라의 노출 한계값을 기준으로 작동합니다. |
| `CHLOROS_LOG_DIR` | 백엔드 환경에서 설정: 상세 로그가 저장될 폴더입니다. Windows의 기본값은 `%LOCALAPPDATA%\Chloros\logs`, Linux의 기본값은 `~/.cache/chloros/logs`입니다(해당 값이 설정된 경우 `$XDG_CACHE_HOME/chloros/logs`). .deb 파일의 루트 서비스가 `/var/log/chloros`를 설정하며, `adm` 그룹이 이를 읽을 수 있습니다 — [백엔드를 서비스로 실행하기](#running-the-backend-as-a-service-headless)를 참조하십시오. |
| `TMPDIR` (Linux) | 임시 파일이 저장되는 위치. CLI는 해당 경로가 존재할 경우 자동으로 `/mnt/ssd/tmp`를 사용합니다. |

---

## 종료 코드

| 코드 | 의미 |
| --- | --- |
| `0` | 성공. |
| `1` | 일반적인 오류 (대부분의 하위 명령어 오류). 캡처 명령은 다른 파일들은 정상적으로 저장되었더라도 쓰기 요청을 받은 파일 중 하나라도 쓰기에 실패한 경우 `1`로 종료되며, `array-burst` / `array-record`는 레코더가 오류를 보고할 때 종료되며, `1`는 [쓰기 실패한 파일](#files-that-fail-to-write)을 참조하십시오. `process`는 실행의 내보내기 데이터를 확인할 수 없거나 이미지를 잃어버린 경우에도 종료되며, 이 경우 다른 결과물은 정상적으로 기록됩니다. |
| `2` | 인자 오류(알 수 없는 플래그 또는 누락된 플래그; 두 우선순위 모두 없는 경우 `time-sync set-priority`도 해당). |
| `130` | Ctrl+C로 중단됨: 중단된 모든 명령 — `lattice`, `project`, `daq` 및 `process` 모두 해당됨(백엔드가 실행을 중지한 후의 `process`, [`process` 참고 사항](#notes) 참조). Ctrl+C로 정상 종료되는 명령 — `lattice array-capture --continuous` / `--interval`, `lattice viewer`, `lattice continuous`, `multi-stream`, `align-stream`, `array-record`, `array-burst`, `project stream` 및 `-n 0` —는 수행한 작업에 따라 종료됩니다: 모든 데이터가 기록되면 `0`, 오류가 발생하면 `1`. |
| `143` / `129` | 명령어의 정리 작업이 완료된 후 SIGTERM / SIGHUP (128 + 신호 번호)에 의해 강제 종료됨: 실행 중이던 작업이 중지되고 카메라가 초기화되었으며, 백엔드에서 처리 작업이 중지됨. Ctrl+C로 정상 종료되는 `lattice continuous` 또는 기타 녹화 작업도 `143`로 종료되며, `0`로 종료되지는 않습니다. |

`lattice network-analysis`는 `1`에 대해 `needs_force_slip` 및 `error` 판정을 내리며, 또한 `selftest`는 7가지 검사 모두를 통과하지 않는 한 `1`를 종료합니다.

---

## 문제 해결 팁

- **"로그인 필요"** → 이 컴퓨터에서 `chloros-cli login EMAIL PASSWORD`를 한 번 실행하십시오.
- **"이 컴퓨터에서 프로젝트 폴더 경로가 너무 깊습니다"** → 이 컴퓨터에서는 Windows 긴 경로 기능이 비활성화되어 있습니다(Windows 기본 설정). 따라서 경로 길이는 260자를 초과할 수 없으며, 제품 경로는 프로젝트 폴더 아래로 최대 약 130자까지 위치할 수 있습니다. 아무것도 처리되지 않았습니다. 이 메시지는 허용되는 가장 긴 프로젝트 폴더 이름을 표시합니다. 더 짧은 폴더(`-o` 또는 `chloros-cli set-project-folder`)를 사용하거나, Windows 긴 경로 기능을 켜고 Chloros를 다시 시작하십시오. Windows 설치 프로그램에서는 이 설정을 "Windows 긴 경로 사용"으로 제공하며, 기본적으로 선택되어 있습니다. 그렇지 않은 경우 그룹 정책을 사용하십시오: *컴퓨터 구성 > 관리 템플릿 > 시스템 > 파일 시스템 > Win32 긴 경로 사용*.
- **"백엔드에 연결할 수 없음"** → Chloros 데스크톱 앱(Windows)을 시작하거나, 백엔드를 직접 실행하십시오: Windows에서는 `"C:\Program Files\Chloros\backend\chloros-backend.exe"`(설치 폴더 내의 `backend` 폴더; `PATH`에는 없음), Linux에서는 `chloros-backend`. 원격인 경우 `CHLOROS_BACKEND_URL`를 확인하십시오.
- **`lattice` 명령어가 "LATTICE 카메라 드라이버를 찾을 수 없음" 오류로 실패함** → 백엔드가 설치 시 함께 제공된 LATTICE 카메라 런타임을 불러올 수 없습니다. 모든 `lattice` 명령은 백엔드에서 실행되며, CLI 자체에는 카메라 드라이버가 포함되어 있지 않습니다. Chloros를 재설치하십시오. Windows 설치 프로그램과 .deb 패키지는 백엔드 옆에 런타임을 설치합니다.
- **Array connect / Array Settings에 "FRAMES WILL DROP" 또는 "Reduce ROI to enable"이 표시됨** → 호스트 NIC 수신 링 크기가 너무 작습니다(일반적으로 NIC 드라이버 업데이트 후 32로 재설정됨). [호스트 NIC 설정 및 튜닝](#host-nic-setup--tuning-lattice-arrays)을 참조하십시오 — `ReceiveBufferLen=256`, `PendingReceives=64`를 설정하십시오.
- **재시작/종료 시 시스템이 멈추고, WMI `Invalid class` 오류가 발생하거나 NIC가 활성화되지 않거나 USB 드라이브가 인식되지 않는 경우** → 오래된 USB 10GbE 어댑터 드라이버로 인해 `DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`). 어댑터 드라이버를 업데이트하십시오 — [호스트 NIC 설정 및 튜닝](#host-nic-setup--tuning-lattice-arrays)을 참조하십시오.
- **Linux: 캡처 파일은 root 소유이므로 삭제할 수 없으며, 백엔드 로그가 `~/.cache`에 없습니다** → 백엔드는 root 권한으로 실행됩니다(`chloros-backend.service` 참조). 사용자 계정으로 백엔드를 실행하는 `chloros-backend@<user>.service`로 전환하십시오 — [서비스로 백엔드 실행](#running-the-backend-as-a-service-headless)을 참조하십시오. 그 전까지는 루트 서비스의 로그가 `/var/log/chloros`에 있습니다(`adm` 그룹에서 읽을 수 있음).
- **Linux: 백엔드 유닛 `failed`에서 “다른 인스턴스가 이미 실행 중입니다”(종료 상태 98)라는 메시지가 표시됨** → 다른 백엔드가 이미 해당 포트를 점유하고 있습니다. 한 번에 하나만 실행될 수 있습니다. 두 개의 백엔드 유닛이 활성화된 경우, 하나는 비활성화 상태로 유지하십시오: 사용하지 않는 유닛은 `sudo systemctl disable --now <unit>`로 설정하십시오 — [백엔드를 서비스로 실행하기](#running-the-backend-as-a-service-headless)를 참조하십시오. 이 유닛만 활성화된 경우, systemd 외부에서(`chloros-backend`, `chloros-cli` 또는 SDK 스크립트를 통해) 시작된 백엔드가 해당 포트를 점유하고 있는 것입니다: 해당 백엔드를 중지한 다음, `sudo systemctl start <unit>`를 실행하십시오.
- **Jetson 스왑 경고** → 파일 기반 스왑을 추가하십시오. CLI는 정확한 `fallocate` / `swapon` 명령어를 출력합니다.
- **`lattice`, `project` 또는 `daq` 명령에서 발생하는 "백엔드에 연결할 수 없음" 오류** → 실행 중인 백엔드가 없습니다. 해당 명령은 백엔드를 시작하지 않습니다. `chloros-cli backend status`를 실행하고 [백엔드 시작](#start-the-backend)을 참조하십시오.
- **"Chloros에서 `daq net ...`를 사용할 수 없음"** (Linux 및 Jetson) → DAQ-E의 일부 설정은 MAPIR 서비스용으로 예약되어 있습니다. 네트워크 설정, 재시작 및 펌웨어 업데이트는 `pool-set-network`, `pool-reboot` 및 `pool-update-firmware`를 사용하십시오. 그 외의 사항은 MAPIR 지원팀에 문의하십시오.
- **"DAQ 직접 하드웨어 명령을 사용하려면 전체 daq 패키지가 필요합니다"** (Windows) → 백엔드를 통해 동일한 센서를 구동하는 `pool-*`를 사용하거나, Python에서 `chloros_sdk.connect_daq_sensor()`를 사용하십시오.

---

## 참조

- [Python SDK 참조](sdk-reference.md) — 모든 CLI 명령에 상응하는 프로그래밍 방식.
- [DAQ 광 센서](https://mapir.gitbook.io/chloros/daq-light-sensors) (Chloros 매뉴얼) — 센서, 정격 및 보정 범위, 기록, DAQ-E 네트워킹.
- [Chloros 매뉴얼의 LATTICE 카메라](https://mapir.gitbook.io/chloros/lattice-cameras/lattice) — 연결, 카메라 설정, 캡처 모드, 어레이.
- 온라인 문서: `https://mapir.gitbook.io/chloros/cli`
