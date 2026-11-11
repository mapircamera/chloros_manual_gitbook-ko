# Chloros Python SDK 참조

**버전:**

1.3.0**생성일:**2026-07-29 19:19 ·**수정일:** 2026-11-06**패키지:** `chloros-sdk` (PyPI)**대상:** LLM 활용에 최적화되었으며, 사람이 읽기 쉬운 형식으로 작성되었습니다.**범위:** `import chloros_sdk`에서 제공하는 모든 공개 클래스, 함수 및 헬퍼를 다루며, 이미지 처리, 단일 카메라 제어, 동기화된 배열, DAQ 센서, 프로젝트 자동화를 다루는 복사-붙여넣기 가능한 예제와 AI 어시스턴트용 `chloros-mcp` 서버를 포함합니다.

주요 내용만 확인하시려면 다음으로 이동하세요:
- [설치 및 빠른 시작](#설치)
- [LATTICE 어레이용 Smart-Connect](#smart-connect-for-lattice-cameras)
- [DAQ 센서 세션](#daq-sensor-sessions)
- [카메라 유틸리티](#camera-utilities)
- [과도 배열, 정렬 프로파일 및 인덱스](#transient-arrays-alignment-profiles--indices)
- [프로젝트 폴더 및 레시피](#project-folder--recipes)
- [프로젝트 자동화](#project-automation--chlorosproject)
- [Smart-AE / Smart-Capture](#smart-ae--smart-capture)
- [자신만의 파이프라인에서 라이브 프레임 활용](#live-frames-in-your-own-pipeline)
- [MCP 서버 (AI 어시스턴트)](#mcp-server-ai-assistants)

> **

1.3.0에서 변경된 사항** (기존 1.2.0 스크립트가 감지하는 내용).
>
> **

1.2.0 스크립트에서 처리해야 할 수 있는 변경 사항:**
>
> - **백엔드에서 확인되지 않은 릴리스가 발생하면 이제 예외가 발생합니다.** 세션이 연 배열에서 발생하는 `CameraSession.disconnect()`, `ArraySession.disconnect()` / `force_disconnect()`, `DAQSensorSession.disconnect()` / `force_disconnect()`는 세션이 연 센서에서 발생하며, `ChlorosProject.disconnect_all()`는 백엔드가 릴리스를 거부하거나 제때 확인하지 않을 때 `ChlorosConnectError`를 발생시킵니다. 이는 장치가 여전히 점유되어 있을 수 있기 때문입니다. 1.2.0 버전에서는 아무런 오류 메시지 없이 정상적으로 반환되었습니다. `with` 블록의 종료 시에도 여전히 예외가 발생하지 않습니다.
> - **차용된 장치는 절대 해제되지 않습니다.** `with attach_array(...)` 및 해당 장치에 대한 명시적인 `disconnect()` 호출은 배열을 계속 실행 상태로 유지합니다(1.2.0 버전에서는 블록을 종료할 때, GUI가 해당 장치를 소유하고 있는 경우에도 해제되었습니다); 종료하려는 경우 `force_disconnect()`를 호출하십시오. 마찬가지로, 센서가 이미 풀에 있는 것을 감지한 `connect_daq_sensor()`(제로 플래그 스마트 감지 포함)는 `disconnect()` 호출 시에도, 그리고 해당 `with` 블록을 떠날 때도 센서를 열어둔 상태로 둡니다.
> - **PNG와 JPEG는 `display` 및 인덱스 제품만 처리합니다.** `capture()`와 `format="png"` / `"jpg"` (`ext=".png"` / `".jpg"`) 및 `radiance` 또는 `reflectance` 레벨, 또는 `processing="all"`인 경우, 어떤 요청을 보내기 전에 `ValueError`를 발생시킵니다; 12비트 `raw` / `debayered`는 백엔드에서 거부됩니다. [PNG / JPEG](#png--jpeg-take-the-display-and-index-products-only)를 참조하십시오.
> - **`connect_daq_sensor(sensor_id=...)`는 해당 센서를 열거나** 발견한 내용을 나열하는 `ChlorosConnectError`를 발생시킵니다. 다른 센서로 대체되지 않습니다. [고정된 전송(Pinned Transport)](#pinned-transport)을 참조하십시오.
> - **`frame_stream()`는 응답을 중단하지 않고 오류를 발생시킵니다:** `FrameFetchError`는 거부 시 다음 틱에서 (DAQ 데이터가 전혀 포함되지 않은 반사율 읽기), 프레임이 60초 동안 없거나 새로운 트리거가 60초 동안 없을 때 발생합니다. [읽기당 하나의 트리거](#one-trigger-per-read--group_id)를 참조하십시오.
> - **읽기 타임아웃이나 연결 끊김 시 모든 스마트 커넥트 호출에서**`ChlorosConnectError`**가 발생하며, 1.2.0 버전에서는 기본 타임아웃이나 연결 오류가 발생했던 위치에서**`analyze_array_network()`**가 발생합니다.
> - **`connect_array()`는 기본적으로 600초 동안 대기하며(`timeout`, 1.2.0 버전에서는 180초), 연결에 실패한 경우 해당 작업은 취소됩니다.** 이 호출은 `timeout`를 백엔드로 전송하며, 백엔드는 모든 카메라를 발견된 상태로 되돌리고, 연결이 제시간에 완료되지 않으면 아무것도 연결하지 않습니다. 이 호출은 `ChlorosConnectError` (HTTP 504 `CLIENT_DEADLINE_PASSED`)를 발생시킵니다. 1.2.0 버전에서는 카메라 전원이 켜진 직후의 연결 시도가 호출이 종료된 후에도 지속되어, 호출이 종료된 후에 어레이가 연결될 수 있었습니다. `connect_all()`는 저장된 각 어레이에 대해 동일한 방식으로 대기합니다 (1.2.0에서는 120초) 대기합니다. [`connect_array()` 서명](#connect_array-signature)을 참조하십시오.
> - **`connect_array()`는 전체 센서를 유지합니다.** `width` / `height` / `binning`가 없는 경우, 모든 카메라는 전체 센서 모드로 설정되며 프레임 속도는 호스트와 네트워크가 제공하는 수준으로 떨어집니다. 1.2.0은 프레임 속도를 유지하기 위해 프레임을 축소하거나 비닝 처리했습니다. [동기화된 어레이](#synchronized-array--arraysession-smart-prep)를 참조하십시오.
> - **M3M(모노) 카메라에서 바이엘 픽셀 형식은 깊이(depth)를 의미합니다.** M3M은 모노 형식만 제공하며, 프리셋 이름은 바이엘 형식을 따릅니다: 1.2.0 버전에서는 카메라가 해당 포맷을 거부하고 자체 포맷을 유지한 반면, `settings.pixel_format`는 베이어 포맷으로 명명되었습니다. `LatticeCamera(preset=...)`, `apply_preset()`, `set_format()`, `CameraSettings(pixel_format=...)`, `connect_camera(preset=...)` 및 배열의공유 형식은 이제 M3M에서 동일한 깊이의 Mono 형식을 설정합니다: `"BayerRG8"`와 `high_speed`는 `Mono8`를 생성하고, `"BayerRG12"`, `high_quality` 및 `triggered`는 `Mono12`를 생성하며, `cam.settings.pixel_format`는 이를 명명합니다. 반대의 경우도 마찬가지입니다. M3C는 Mono 형식으로 저장될 수 없으므로, `"Mono12"` — 이 중 어느 것이든, `connect_array(pixel_format=...)`, `set_settings(pixel_format=...)` 또는 배열의 저장 형식 등 — M3C에서는 배열의 마스터인 카메라에 관계없이 `BayerRG12`가 생성됩니다. 전달된 `CameraSettings`는 이름을 유지하므로, 동일한 객체는 여전히 각 카메라에 해당 깊이 정보를 자체 패밀리 내에서 요청합니다. `connect_camera(preset=...)`는 프리셋의 형식을 유지하며 (1.2.0 버전에서는 프리셋 적용 후 카메라를 12비트로 설정함)을 유지하며, 프리셋으로 열린 카메라는 더 이상 촬영한 내용을 `PRESETS`에 기록하지 않습니다. 카메라가 거부하는 다른 모든 형식 (해당 카메라의 `PixelFormat` 목록에 없는 형식, 예를 들어 M3M의 `RGB8`나 해당 기능이 없는 펌웨어의 `BayerRG16` 등)은 별도의 이름으로 지정되지 않습니다. `LatticeCamera` 연결 후, `apply_settings()` 또는 `set_format()`, `cam.settings.pixel_format` 및 `packed_transport`는 카메라가 스트리밍하는 형식을 나타내며, 한 줄의 출력 내용은 카메라가 거부한 형식을 나타냅니다.
> - **`analyze_array_network()`는 어떤 연결이 적용되는지 나타냅니다:** `pixel_format=None`(이전 명칭: `"BayerRG8"`)는 연결이 설정하는 포맷을 의미하며 (Mono12 / BayerRG12)를 의미하며, `packed="auto"`는 압축 전송 방식을 나타내므로, 표시된 fps는 1.2.0'와 다를 수 있습니다. 전달하는 `pixel_format`는 해당 깊이의 M3M을 나타냅니다. [네트워크 분석 도우미](#network-analysis-helper)를 참조하십시오.
> - **`CameraSession.capture()`는 `CaptureResult`를 반환합니다** (여전히 프레임 딕셔너리 목록)을 반환하며, 여기에는 `.skipped`, `.failed`, `.daq_files` 및 `.daq_matches`가 포함됩니다. `ArraySession.capture()` 결과에는 `.alignment`(정렬 워프가 적용되었는지, 적용되지 않았다면 그 이유)와 `.oriented`도 포함됩니다.
> - **배열 캡처는 앱과 마찬가지로 기본적으로 정렬됩니다.** `ArraySession.capture()`, `continuous()`, `capture_repeated()` 및 `ArrayHandle.capture()` / `capture_smart()`는 각 구성원의 `debayered`, `display`, `radiance`, `reflectance` 및 인덱스 파일을 마스터 카메라의 프레임으로 워프하고, 멤버들의 오버랩 (카메라 오프셋만큼 센서보다 작은 크기, `XMP-mapir:AlignmentJSON` `"applied": true`)로 자르므로, 이는 Chloros 앱이 동일한 장면을 캡처한 파일과 동일한 파일이 됩니다. `raw`는 원본 크기를 유지합니다. 이전에는 `aligned=True`가 없는 캡처의 경우 모든 파일이 원본 크기로, 왜곡 보정 없이 저장되었습니다. `aligned=False`는 이 방식을 유지합니다. `chloros-mcp`의의 `capture_array` 및 `start_capture_run`는 동일한 기본값을 따릅니다. [`ArraySession` 메서드](#arraysession-methods)를 참조하십시오.
> - **`connect_all()`는 SDK를 거부하는 백엔드의 이름을 지정합니다.** 로그아웃되었거나 Copper보다 낮은 요금제를 사용하는 실행 중인 백엔드는 “Chloros 백엔드가 실행 중이 아닙니다”가 아닌 자체적인 사유 (HTTP 401 / 403)로 보고되며, "실행 중인 Chloros 백엔드가 없음"으로 표시되지 않습니다. 또한 각 `connect_all()` 호출은 백엔드를 다시 확인하므로, 로그인 후 호출하면 동일한 프로젝트 객체에서 작동합니다.
> - **상대 경로인 `output_dir`는 로컬 백엔드에서 작동합니다.** 상대 경로는 전송되기 전에 스크립트의 작업 디렉터리를 기준으로 절대 경로로 변환됩니다. 1.2.0 버전에서는 백엔드가 자체 설치 폴더에서 경로를 해석하여 "Access is denied" 오류가 발생했습니다.
> - **`numpy`는 이제 SDK의 필수 요소입니다**(live-frame 호출은 numpy 배열을 반환합니다).
> - **1이 아닌 내보내기 수준 이름은 허용되지 않습니다.** `processing=` 및 `levels=`는 `raw`, `debayered`, `radiance`, `reflectance` 또는 `display` (또는 `preview`와 같은 별칭; `processing=` 및 `"all"`도 포함)을 선택합니다. 카메라를 터치하기 전에는 다른 이름은 허용되지 않습니다(HTTP 400 `UNKNOWN_LEVEL`: `capture()`는 `ChlorosConnectError`, `grab()` / `frame_stream()` `FrameFetchError`); 1.2.0 버전에서는 이에 대해 `raw`를 저장하거나 반환했습니다.
> - **LATTICE 캡처의 경우 Texture Aware가 허용되지 않습니다.** LATTICE 캡처가 포함된 프로젝트에서, `configure(debayer="Texture Aware (Slow, Highest Quality)")`는 `ChlorosConfigurationError`를 발생시키며, 임포트 전에 이전에 이 설정을 요청한 후 발생합니다. 프로젝트가 이전에 저장한 Texture Aware 값은 `UserWarning` 오류와 함께 'Standard'로 설정됩니다. LATTICE 캡처는 항상 표준 디모자이크를 적용받으므로, 1.2.0 버전은 해당 설정을 수락하고 무시했습니다. 또한 Chloros 앱에서도 LATTICE 캡처에 대해서는 이 설정을 제공하지 않습니다. [지원되는 값](#supported-values)을 참조하십시오.
> - **카메라는 발견된 상태로 되돌아가며, `keep_settings=True`인 경우는 예외입니다.** `ArraySession.disconnect()` / `force_disconnect()`, 모든 일시적 함수의 끝 (`capture_cameras_transient()`, `start_multi_camera_stream()`, `test_array_sync()`, `calibrate_alignment_profile()`, `capture_aligned_multiband()`, `start_aligned_stream()`, `capture_live_index()`) 및 `start_camera_continuous()`의 중지 시, 각 카메라는 트리거 및 싱크 라인 배선, PTP, 그리고 세션 중에 변경된 모든 카메라 설정(노출 및 게인, AE 목표값, 한계값 및 댐핑, 화이트 밸런스, 블랙 레벨, 픽셀 포맷, 비닝 및 ROI, 플립, 프레임 레이트)를 모두 원래대로 복원합니다. 반면 `keep_settings=True`는 각 카메라의 설정을 카메라 자체 메모리에 저장하여, 전원을 켤 때 해당 설정으로 시작하도록 하며, `clear_settings=True`는 해당 메모리를 공장 출하 시 설정으로 초기화합니다. 두 경우 모두 카메라는 자유 실행 상태로 유지됩니다. 프로젝트는 카메라를 연결할 때 자체 저장된 설정으로 카메라를 설정합니다. [카메라 메모리](#camera-memory--keep_settings--clear_settings)를 참조하십시오.
> - **모든 모드는 동일한 방식으로 프로젝트를 저장합니다.** 프로젝트에 연결된 카메라, 어레이 또는 광 센서에 대한 변경 사항(SDK, 앱, CLI 또는 MCP 서버를 통해 이루어진 변경 포함)은 해당 프로젝트의 `cameras.json` / `sensors.json`에 저장되며, 어떤 모드에서든 프로젝트를 열면 해당 설정에 따라 장치가 구성됩니다. `ChlorosProject`는 백엔드에 자체 폴더 이름을 지정합니다. 이전에는 앱에서만 프로젝트를 저장했습니다. [하나의 프로젝트 레코드, 모든 모드](#one-project-record-every-mode)를 참조하십시오.
> - **여러 대의 카메라로 생성된 파일은 각 카메라의 이름을 모두 명시합니다.** 멀티밴드 스택, 여러 카메라의 밴드로 생성된 라이브 인덱스, 배열 캡처의 `composite/` 이미지는 각 소스 카메라의 일련번호, 모델 및 보정 팩을 목록(`XMP-mapir:SourceCameraSerials`, `SourceCameraModels`, `SourceCalBundleSha256s`, …)로 저장되며, EXIF `SerialNumber`에는 일련번호가 쉼표로 구분되어 나열됩니다. 이전에는 카메라 정보를 포함하지 않거나 첫 번째 카메라의 일련번호만 포함했습니다. [여러 대의 카메라로 생성된 파일](#files-made-from-several-cameras)을 참조하십시오.
> - **`raw` 파일만 카메라의 보정 태그를 포함합니다.** MicaSense 호환 세트(`BlackLevel`, `RadiometricCalibration`, 각-채널별 `Band{R,G,B}RadiometricCalibration`, `MicaSenseVignetting*` 세트) 및 `CalibrationJSON` 페이로드는 센서의 원시 값을 기술하므로, `raw` 캡처 파일에만 포함됩니다. 그 외의 모든 파일 — 캡처'`debayered`, `display`, `radiance`, `reflectance` 및 인덱스 파일, 연속 녹화 및 멀티 카메라스트림 프레임, 그리고 모든 `process()` 제품 —에는 카메라 식별 정보(`XMP-mapir:CameraSerial`, `CameraModel`, `FilterType`, `LensModel`, `CalBundleSha256`, ...)을 포함하며, 촬영 시에는 프레임 자체의 기록(`XMP-mapir:FrameJSON`: 노출, 게인, 온도, 픽셀 형식, 촬영 시간). 1.2.0 버전에서는 원시 데이터 세트도 포함되어 있었으며, 이를 적용하는 도구(Pix4D, Metashape)는 원시 값이 아닌 픽셀에 이를 적용했습니다. Processing은 일련번호와 `CalBundleSha256`를 통해 해당 파일의 보정 정보를 찾아냅니다.
>
> **기타 변경 사항:**
>
> - **어레이의 자동 프레임 속도는 어레이가 제공하는 속도에 맞춰 조정됩니다.** 트리거 수의 1% 이상을 잃는 속도는 한 번이 아닌 여러 번의 장시간 관찰을 통해 판단하여 단계적으로 낮추며, 단계적으로 낮아진 속도는 수시로 재시도됩니다. 사용자가 직접 설정한 속도는 변경되지 않습니다.
> - **자동 노출이 더 안정적으로 작동합니다.** 일정한 조도에서는 노출을 유지하며, 조도 변화 후 몇 초 이내에 안정화되고, 미세한 하이라이트는 무시하며, 강한 햇빛 아래에서 응답이 중단된 카메라의 작동을 복구합니다. [자동 노출 작동 방식](#how-auto-exposure-behaves)을 참조하십시오.
> - **`import_images(recursive=True)`는 이미지와 녹화 파일만 집계합니다.** 프로젝트의 `camera_cal/` 파일이 포함된 폴더의 경우, 해당 보정 파일은 더 이상 이미지와 녹화 파일로 집계되지 않으며, 이미 이를 포함하고 있던 프로젝트에서는 해당 파일이 제외됩니다.
> - **`configure(radiometric_output=...)`가 적용됩니다.** 이제 프로젝트의 `Export radiance` / `Export reflectance` 산출물을 설정합니다. 이전에는 각 프로젝트의 자체 산출물 설정이 우선 적용되었으므로, 따라서 `radiance`, `sensor-response` 및 `all`는 모두 복사도 및 반사도를 내보냈습니다.
> - **`test_array_sync()`는 카메라의 전원 켜짐 상태에서 시작됩니다.** 이 명령은 실행을 위해 카메라의 PTP를 켜고, 첫 번째 트리거(`ptp_lock`)가 발생하기 전에 카메라 클럭이 동기화되고 안정화될 때까지 대기합니다 — 전원 켜진 시점부터 약 2분(120초) 동안 모든 카메라가 동기화됩니다 (`ptp_settle_s=`가 이를 설정함). 갓 동기화된 클럭은 약 2분 동안 최대 ~200 us의 오차를 보이기 때문이며 — 이후 각 카메라의 PTP를 원래 상태로 되돌립니다. 이전 테스트에서 이미 동기화된 상태인 카메라는 다시 대기하지 않습니다. 과거에는 DAQ-E가 응답하지 않는 단일 스위치 네트워크에서 PTP를 비활성화한 채로 두었고, 각 카메라의 자유 주행 클럭을 PTP 클럭으로 간주하여 동기화되지 않는 것으로 처리했습니다 ("PTP가 잠기지 않음", 매 실행 시).
> - **`.daq` 기록 파일은 캡이 선택되었는지 또는 추정되었는지를 나타냅니다.** DAQ-U, DAQ-M 및 DAQ-E는 캡 없이 출하되며, <sn><id>Chloros</id></sn>는 캡이 장착되었는지 여부를 확인할 수 없으므로, 캡 이름이 지정되지 않은 상태로 연결된 센서는 기본적으로 *추정*된 값인 `sunshine_cosine`를 부여받습니다. 모든 Chloros 기록(스키마 1.30)은 이제 `cap_id`와 함께 `cap_id_source`를 저장합니다: 캡 이름을 지정한 경우(`none` 포함)에는 `operator`, 기본값이었던 경우에는 `auto_default`가 저장됩니다. 캡이 가정된 기록을 처리할 때, 프로젝트의 DAQ 캡을 ‘자동’으로 둔 채로 진행하면 파일 이름을 `[daq-dls] WARNING`로 지정하여 출력합니다: 햇빛 보정 처리되지 않은 순수 DAQ-U의 측정값은 실제보다 약 30배 높게 나타납니다. 연결 시 **장착한 캡에 이름을 지정**하면(`connect_daq_sensor(cap_id=...)` 또는 `DAQSensorSession.set_cap()`, `chloros-cli daq pool-connect --cap-id` / `daq pool-set-cap`, 또는 GUI의 커패시터 드롭다운 메뉴)에 장착한**커패시터 이름을 지정**하거나 프로젝트의 DAQ 커패시터를 설정하면 경고가 사라집니다.
> - **Windows에서 더 이상 pip 콘솔 창이 표시되지 않습니다.** Python가 설치된 PC에서 백엔드와 각 이미지 워커는 시작 시 `pip`를 한 번씩 실행했고, 각각 콘솔 창을 열었습니다. 이제 더 이상 그러지 않습니다.
> - **실시간 `.daq` 녹화 데이터는 정전 시에도 유지됩니다.**녹화 데이터는 센서 뒤에서 매 커밋 시 저널과 전체 디스크 동기화를 통해 기록되므로, 디스크 속도가 느려도 파일 작성만 지연될 뿐 측정값 자체는 절대 지연되지 않습니다. 커밋 도중에 중단된 기록(정전, 강제 종료)은 그 옆에 `<name>.daq-journal` 파일을 남길 수 있습니다:**두 파일을 함께 복사하십시오.**Chloros는 파일을 처음 읽을 때 저널을 롤백하므로, 기록 파일은 중단되기 전까지 커밋된 모든 내용을 포함하여 열립니다(마지막 약 3초 분량이 손실됨). 1.2.0 버전에서는 부적절한 시점에 정전이 발생하면 전체 파일이 읽을 수 없게 될 수 있었습니다("데이터베이스 디스크 이미지가 잘못된 형식임").
> - **사용량이 많은 디스크에서 `DAQSensorSession.record_start()`가 더 이상 센서를 지연시키지 않습니다**: 측정값은 대기열에 쌓여 별도의 스레드에서 기록되며, 기록되지 못한 값은 `status()`의 `record_errors`(`record_last_error` 포함)에 집계됩니다.
> - **Direct `LatticeCamera`:**청크 노출/게인이 없는 프레임은 이제 카메라의**실제** 노출과 게인을 보고합니다(이전 방식은 자동 노출 실행 전체 기간 동안 고정된 상태로 남아 있을 수 있는 캐시된 값을 읽었습니다), 또한 `apply_settings(trigger_source='LineN')` / `set_trigger_source('LineN')`는 Chloros의 케이블 슬레이브가 적용받는 것과 동일한 5 µs 입력 디글리치 처리를 적용합니다(필터링되지 않은 상태로, 트리거 신호 없이 슬레이브가 촬영한 프레임).
> - **소프트웨어 트리거 방식의 카메라는 더 이상 트리거 신호를 번갈아 가며 놓치지 않습니다.** 소프트웨어 트리거 모드의 카메라는 이제 케이블 동기화 카메라와 마찬가지로, 판독 중에 도착한 트리거를 대기열에 넣습니다(또는 Chloros가 카메라가 준비될 때까지 대기합니다).
> - **`discover_daq_sensors()`는 DAQ-E-S를 `DAQ-E-S`로 명명합니다.** 이 장치의 `model`는 모든 이더넷 센서에 대해 `DAQ-E`로 지정되었습니다. 약 2초 이내에 신원 확인에 응답하지 않는 센서는 여전히 `DAQ-E`로 나열됩니다.
> - **신규: `chloros-mcp`, AI 어시스턴트용 MCP 서버.** Claude Desktop, Claude Code, Cursor 및 기타 MCP 클라이언트는 이미지 처리(프로젝트 생성, 캡처 파일 가져오기, 설정 선택, 처리, 실행 결과 내보내기)를 수행하고 LATTICE 카메라, 어레이 및 DAQ 광 센서를 제어할 수 있습니다: 연결, 미리보기, 노출 및 색상 설정, 캡처, 기록. 이 도구들은 이 SDK가 수행하는 기능을 모두 포함합니다. SDK와 함께 설치되며, Linux용 .deb 패키지는 이를 `/usr/bin`에 설치합니다. [MCP 서버 (AI 어시스턴트)](#mcp-server-ai-assistants)를 참조하십시오를 참조하십시오.
> - **신규: 카메라 유틸리티**, SDK 호출과 동일한 `chloros-cli lattice` 명령어: 색상 프로필, 채도/명암, 하이라이트, 화이트 밸런스(원샷, 자동, 고정 비율), 프리셋, 필터 모델, 펌웨어 상태 및 업데이트, 전원(프로브, 절전, 깨우기), 타겟 보정, 네트워크 어댑터(확인, 추정, 수정), 파일의 비네팅 보정, DLS 판독, 호스트 프로브. [카메라 유틸리티](#camera-utilities)를 참조하십시오.
> - **신규: 나머지 CLI가 SDK 호출로 제공됨**, 따라서 CLI, SDK 및 `chloros-mcp`는 동일한 기능을 제공합니다(보안상의 이유로 CLI는 로그인, DAQ-E 프로비저닝 및 직접 DAQ 서버 기능만 유지함): 단일 명령으로 카메라 열기(`start_camera_continuous()`, `start_multi_camera_stream()`, `capture_cameras_transient()`, `test_array_sync()`, `detect_array_master()`, `analyze_array_models()`), 정렬 프로파일 파일(`calibrate_alignment_profile()`, `capture_aligned_multiband()`, `start_aligned_stream()`, `alignment_profile_info()`, `reorder_alignment_profile()`), 인덱스(`index_catalog()`, `compute_index()`, `capture_live_index()`), PTP 그랜드마스터(`time_sync_status()` 및 관련 파일들), 프로젝트 폴더(`project_folder()`, `set_project_folder()`, `reset_project_folder()`) 및 레시피(`run_project_recipe()`, CLI 전용 실행기)가 포함됩니다. 광 센서의 경우: `connect_daq_sensor(cap_id=...)`, `DAQSensorSession.set_cap()`, `daq_bindings()` / `set_daq_bindings()`, `daq_band_presets()` 및 `light_sensor_reflectance()` (두 센서의 주변광 + 물체 반사율 및 지수); 카메라용: `CameraSession.status()` 및 `capture_repeated()`; 어레이의 경우: `ArraySession.capability()` 및 `set_member_settings()` (구성원의 노출, 게인 등). [과도 배열, 정렬 프로파일 및 지수](#transient-arrays-alignment-profiles--indices), [프로젝트 폴더 및 레시피](#project-folder--recipes) 및 [DAQ 센서 세션](#daq-sensor-sessions)을 참조하십시오.
> - **개선된 LATTICE 지수 산출 결과** (NDVI 및 기타 지수)가 `process()`에서 모든 처리 모드에 적용됩니다.
> - **LATTICE OCN 카메라: 햇빛 조건에서 방사도, 반사도 및 지수 산출물이 개선되었습니다.** 밴드 분리 후 플릿 보정이 적용됩니다 — 실시간, 캡처 및 `process()`에서; RGN 카메라는 변경되지 않았습니다. [방사계 출력](#radiometric-output-lattice-multispectral-pipeline)을 참조하십시오.
> - **GUI와 마찬가지로, 모든 캡처는 사용된 DAQ 판독값을 저장합니다.** `CameraSession.capture()` 및 `ArraySession.capture()` / `continuous()`는 `.daq`와 그 `.match.json`를 기록합니다. DAQ가 바인딩되거나 풀에 할당될 때마다, 1.2 버전에서는 원시 전용 캡처가 포함되었습니다.0에서는 `force_daq=True`가 있는 경우에만 원시 또는 디베이어링된 캡처가 저장되었습니다. `daq=False`는 이 기능을 비활성화하고, `daq_tolerance_ms=`는 일치 허용 오차를 설정합니다. 두 결과 모두 `.daq_files`와 `.daq_matches`에 저장된 내용을 나타냅니다. 이제 캡처는 노출 전후의 판독값을 혼합할 수 있도록 DAQ의 다음 판독값을 기다립니다. [캡처가 저장하는 DAQ 판독값](#the-daq-reading-a-capture-saves--daq_files-and-daq_matches)를 참조하십시오.
> - **`ArraySession`에는 라이브 프레임과 서버 측 연속 캡처가 있습니다.** `grab()` 및 `frame_stream()`는 하나의 트리거에서 numpy 프레임을 반환합니다 (프로젝트 API의 `ArrayHandle.grab()` / `frame_stream()`와 동일한 읽기 작업)에서 numpy 프레임을 반환하며, `continuous()`는 백엔드에 모든 동기화된 그룹을 저장하고 [`ContinuousHandle`](#continuoushandle)를 반환합니다. [자신만의 파이프라인에서 라이브 프레임 사용하기](#live-frames-in-your-own-pipeline)를 참조하십시오.
> - **배열 `grab()`는 하나의 트리거에 고정됩니다** (`ArrayHandle.grab()` 및 새로운 `ArraySession.grab()`). 1.2.0 버전에서는 20개의 4-카메라 `ArrayHandle.grab()` 읽기 중 12개는 두 개의 트리거에 걸쳐 있었습니다. 이제 모든 프레임은 `group_id`를 포함하며, 배열 `frame_stream()`는 각 트리거에 대해 한 번씩 결과를 반환합니다. [읽기당 하나의 트리거](#one-trigger-per-read--group_id)를 참조하십시오.
> - **대역별 및 정렬된 프레임:** `grab()` / `frame_stream()`는 `band=`, `align=`, `bit_depth=` 및 `refl_max=`를 카메라 및 어레이 핸들에서 반환합니다. [단일 밴드 내보내기](#exporting-a-single-band--band)를 참조하십시오.
> - **새로운 `ArraySession` 메서드 및 인자:** `set_target_fps()`, `force_disconnect()`, `capture(after_seq=..., orient=...)` (`CaptureResult.group_seq` 및 `.failed` 포함), `continuous()`, `connect_array(packed=...)`; `capture_repeated()`는 연속된 그룹을 연결하며, 드라이브가 가득 차면 중지합니다.
> - **`ArraySession.record()`는 GUI가 열려 있지 않은 상태에서 기록합니다.** GUI의 결합된 보기가 닫혀 있으면, 백엔드가 뷰 자체를 렌더링합니다: 배열의NDVI 또는 `expr=`로 전달한 인덱스를 렌더링합니다. `RecorderHandle.stop()`는 기록에 실패했거나 프레임이 포함되지 않은 기록에 대해 `ChlorosConnectError`를 발생시키고, 중지 결과를 `.result`에 저장합니다(`with` 블록을 남겨두면 예외가 발생하지 않으므로 `rec.result`를 확인하십시오). [`RecorderHandle`](#recorderhandle)를 참조하십시오.
> - **새로운 `DAQSensorSession` 메서드:** `enable_ptp()`, `force_disconnect()`, 그리고 DAQ-E의 자체 설정을 위한 `device_info()`, `set_network()`, `set_auto_stream()`, `reboot()` 및 `update_firmware()`가 있습니다. `record_stop()`는 `warning` 및 `gaps`를 반환합니다. 녹화를 종료한 재시작은 응답에서 해당 녹화 이름을 지정하거나, 재시작이 실패할 경우 `ChlorosConnectError`에 표시됩니다(`recording_closed`; [DAQ-E 장치 설정](#daq-e-device-settings) 참조). [`DAQSensorSession` 메서드](#daqsensorsession-methods)를 참조하십시오.
> - **데이터 전송이 중단된 풀링된 DAQ-E는 백엔드에 의해** 기존 설정을 유지한 채 재연결되며, 해당 DAQ-E에서 진행 중이던 기록은 파일 내에 간격이 표시된 상태로 동일한 `.daq`에서 계속됩니다. `record_stop()`는 간격을 `gaps`로 반환합니다. 1.2.0 버전에서는 센서가 연결이 끊어졌다가 다시 연결될 때까지 응답하지 않았으며, 이로 인해 기록이 종료되었습니다. CLI 참조 문서의 [데이터 전송이 중단되는 DAQ-E](cli-reference.md#a-daq-e-that-stops-delivering)를 참조하십시오.
> - **`open_project()`는 설치된 SDK에서 작동합니다.** 휠(설치 프로그램의및 PyPI)에는 이제 필요한 프로젝트 로더가 포함되어 있습니다. 이전에는 `open_project()`가 `ImportError`를 발생시켜 `project_cameras` / `project_sensors`를 호출하는 오류를 발생시켰습니다. `connect_all()`는 실행 중인 백엔드 (광 센서는 DAQ 풀을 통해) 연결하며, 찾을 수 없는 저장된 장치는 아무런 설명 없이 건너뛰어지지 않고 `errors` 항목에 사유가 함께 표시됩니다. 백엔드(GUI, 또는 `chloros-cli project connect`의)에 이미 등록된 저장된 어레이는 연결된 상태로 계속 실행되며, 이전에는 오류("카메라가 <sn>이미 어레이에 있습니다<id>")</id></sn>로 보고되던 것과 다릅니다<sn><id>. [`ChlorosProject` 메서드](#chlorosproject-methods)를 참조하십시오.
> - **`ChlorosLocal()`, `process_folder()` 및 `process_lattice_capture()`는 Windows에서 Chloros를 찾습니다.** 이 메서드들은 설치 프로그램이 백엔드를 저장한 위치(기본적으로 `<install folder>\backend\`, `C:\Program Files\Chloros\backend\`, 설치 프로그램 자체 기록에 기반한 사용자 지정 폴더)를 확인한 후, 백엔드를 새로 검색하지 않고 이미 실행 중인 백엔드에 연결합니다. 1.2.0 버전에서는 `backend_exe=`를 전달하지 않는 한, 설치된 시스템에서 “Chloros 백엔드를 찾을 수 없음” 오류가 발생했습니다.
> - **시작 시 종료되는 백엔드는 즉시 보고됩니다.** `ChlorosLocal()` 또는 스마트 연결 호출이 응답하기 전에 종료되는 백엔드를 시작할 경우, 해당 호출은 몇 초 내에 종료되며, `ChlorosBackendError`는 `ChlorosLocal`에서, `ChlorosConnectError`는 스마트-connect 호출에서 각각 `ChlorosLocal` 및 `ChlorosConnectError` 오류가 발생하며, 종료 코드, 실행된 명령어 및 백엔드가 출력한 마지막 줄들이 함께 보고됩니다. 1.2.0 버전은 60초의 시작 타임아웃을 기다리기만 했을 뿐, 원인을 알리지 않았습니다. [백엔드 자동 시작](#backend-auto-start)를 참조하십시오.
> - **스크립트 사이에서 백엔드가 계속 실행됩니다.** `with ChlorosLocal(...)` 블록을 벗어나면, `process_folder()` / `process_lattice_capture()`가 반환되고 `chloros-mcp`가 종료되는 것은 더 이상 SDK가 시작한 백엔드를 중지시키지 않습니다. 다음 스크립트가 이를 재사용합니다. 백엔드는 `logout()`에서(Chloros 앱이 열려 있거나 실행 중인 경우가 아닌 한), `shutdown_backend()`에서, 또는 백엔드 환경에서 이를 사용하는 것이 없을 경우 30분이 지나면 자동으로 종료됩니다 (백엔드 환경의 `CHLOROS_BACKEND_IDLE_MIN`가 이를 변경하며, `0`: 절대 발생하지 않음).
> - **SDK가 시작한 백엔드는 강제 종료되지 않고, 종료 요청을 받습니다.** `shutdown_backend()`는 백엔드가 카메라와 광 센서를 해제하고 모든 녹화를 완료할 수 있도록 허용하며, 10초 후에도 여전히 실행 중일 경우에만 종료됩니다. 1.2.0 버전은 즉시 종료시켰습니다 (Windows에서는 강제 종료), 이로 인해 해당 백엔드에 연결된 DAQ 광 센서가 응답하지 않을 수 있었습니다. SDK가 실행 중인 것으로 감지한 백엔드는 여전히 그대로 유지됩니다.
> - **Linux에서는 자동 시작 기능이 현재 폴더에서 `backend_server.py`를 절대 실행하지 않습니다.** 백엔드가 설치되어 있지 않은 상태에서, 백엔드가 포함된 폴더에서 스크립트를 실행하면 해당 백엔드가 실행되었으나, 이제는 설치된 백엔드만 시작됩니다(또는 `ChlorosLocal`의 경우, `backend_exe=`가 지정한 백엔드만 시작됩니다).
> - **`configure(indices=...)`**는 각 카메라의 대역에 대한 프리셋을 프로젝트에 추가하며, 새로운 프로젝트도 포함하고, 추가할 수 없는 이름에 대해서는 경고(`UserWarning`)를 표시합니다.**`import_images()`**는 `.daq` 녹화 파일을 `scan_count`로 보고하며, 더 이상 LATTICE 캡처 폴더에 대해 경고하지 않습니다.
> - **`read_image_audit_tags()`는 pip 설치로 실행됩니다.** 이 버전은 이미지 메타데이터를 직접 읽습니다. 1.2.0 버전에서는 Python 경로에 데스크톱 설치 버전의 내부 구성 요소가 없어도 `{}`를 반환했습니다.
> - **`capture()`는 `connect_array()` 실행 직후**, HTTP 503 "Array still warming up" 오류로 실패하는 대신, 어레이의 첫 번째 프레임 그룹이 나타날 때까지 최대 20초 동안 대기합니다.
> - **`status()`는 중단 상태를 보고합니다.** 새로운 `stalled` 필드는 캡처 루프가 실행 중이지만 아무것도 게시하지 않을 때 True를 나타내며, 이때 `actual_fps`는 0이 됩니다. 1.2.0 버전에서는 `actual_fps`가 마지막 값을 유지했기 때문에, 스톨 상태인 어레이는 `actual_fps > 0`를 표시하는 반면, 캡처는 503 오류("No fresh synced frame group")로 실패했습니다.
> - **불완전한 프레임 세트 및 캡처 503 오류 감소:** 이제 각 슬레이브 카메라의 트리거 입력에서 글리치 현상이 제거되었습니다.
> - **압축된 12비트 전송** (`connect_array(packed="auto")`)은 전체 센서 어레이의 프레임 속도를 약 3분의 1 정도 높여줍니다. [압축된 12비트 전송](#packed-12-bit-transport-packed)을 참조하십시오.
> - **ROI 내 어레이 노출:** 작은 ROI를 지정하더라도 더 이상 구성원의 노출 시간이 센서의 프리런 주기(768 × 304 기준 4.9 ms)로 제한되지 않습니다. [어레이 구성원의 노출](#exposure-on-array-members)을 참조하십시오.(#exposure-on-array-members)를 참조하십시오.
> - **자동 노출이 더 이상 어레이의 자동 프레임 속도를 낮추지 않습니다.** 케이블 동기화 배열은 네트워크와 카메라의 판독 속도가 허용하는 속도로 작동하며, 자동 노출은 해당 속도가 제공하는 주기를 채운 후 게인을 높입니다. 주기보다 긴 수동 노출의 경우, 촬영이 거부되지 않고 해당 주기에 맞춰 트리거 속도가 느려집니다. [배열 구성 요소의 노출](#exposure-on-array-members).
> - **`read_nodes()`**는 자동 노출이 실행되는 동안 `frame_values` 및 `notes`를 추가합니다([자동 노출 시 `read_nodes()`](#read_nodes-under-auto-exposure--frame_values-and-notes) 참조). 카메라가 노출이나 게인을 거부할 경우,**`set_settings()`**는 허용 범위를 표시하고 해당 값에 대해 꺼져 있던 자동 모드를 다시 켭니다. `set_settings(target_brightness_auto=True)`는 새로 추가된 코드입니다.
> - **`orient=`** `ArraySession.capture()` / `capture_fastest()` / `continuous()` / `burst()` 및 `ArrayHandle.capture()`에서 / `capture_smart()`는 카메라 방향에 따라 제품을 회전시킵니다. [`ArraySession` 메서드](#arraysession-methods)를 참조하십시오.
> - **`DLS()`는 센서 고유의 `mac_address`가 필요합니다.** 더 이상 기본 MAC은 없습니다(이전에는 하나의 장치 주소였습니다). MAC이 지정되지 않은 `DLS()`는 `DLSError`를 발생시킵니다. [DAQ-M 대역 크기](#daq-m-band-magnitudes--dlsband_magnitudes)를 참조하십시오.
> - **Windows 설치 프로그램**에는 "Windows 긴 경로 사용" 및 "Microsoft Defender 검사에서 Chloros 백엔드 제외" 옵션이 있으며, 두 옵션 모두 기본적으로 선택되어 있습니다. [1단계](#step-1--install-the-chloros-platform-package)를 참조하십시오.
> - **Linux .deb**는 시스템용 Python SDK를 설치합니다(`python3` (이전에는 디스크에 wheel 파일만 배치했음), 카메라 스트리밍에 필요한 커널의 소켓 수신 버퍼 용량을 늘리고, 사용자 계정으로 실행되는 백엔드 서비스를 추가합니다. JetPack 6용 Jetson 패키지는 `chloros_1.3.0_arm64_jp6.deb`이며, JetPack의 CUDA PyTorch를 포함합니다. [1단계](#step-1--install-the-chloros-platform-package)와 [2단계](#linux-deb)를 참조하십시오.

---

## 60초 만에 알아보는 아키텍처

이 SDK는 Chloros 백엔드(데스크톱 GUI 및 CLI에서 사용하는 것과 동일한 Flask 서버) 위에 구축된 얇은 Python 레이어입니다. 자동화를 위해 `chloros_sdk`를 임포트하고 고수준 메서드를 호출하면, 내부적으로는 모든 호출은 포트 5000의 로컬 백엔드로 전송되는 HTTP 요청으로 변환됩니다 — `http://127.0.0.1:5000/api/...` (의도적으로 `localhost`는 사용하지 않았는데, 이는 Windows에서 먼저 `::1`로 해결되며, IPv4 전용 백엔드에 대한 요청당 약 2초의 비용이 발생하기 때문입니다). 백엔드는 하드웨어 풀(카메라, DAQ 센서, 정렬 프로파일, 프레임 버퍼 등)을 관리하므로, SDK 스크립트는 시리얼 포트나 NIC 대역폭을 놓고 경쟁할 필요 없이 GUI와 공존할 수 있습니다.

사용하게 될 인터페이스는 세 가지입니다:

1. **`ChlorosLocal` + 자유 함수** (`process_folder`, `process_lattice_capture`) — 이미지 처리 파이프라인. 단 한 번의 Python 호출로 전체 폴더에 대해 보정/데베이어/인덱스 내보내기 작업을 수행합니다.
2. **스마트 커넥트 핸들** (`connect_camera`, `connect_array`, `connect_daq_sensor`) — 라이브 하드웨어용 영구 백엔드 세션을 엽니다. GUI와 동일한 “스마트 준비(smart-prep)” 흐름: 네트워크 프로브, 계층 자동 선택, PTP, AE 시딩, GPIO 트리거 구성.
3. **`ChlorosProject` / `open_project`** — 저장된 프로젝트(`cameras.json` + `sensors.json` + `project.json`가 포함된 폴더)를 불러오고, 해당 카메라, 어레이 및 광 센서를 한 번에 연결한 후, 이름이 지정된 핸들을 사용하여 캡처를 실행합니다.

표면 1과 2는 **로컬 백엔드가 아직 수신 대기 중이 아닐 경우 자동으로 백엔드를 시작**합니다(GUI/CLI가 생성하는 것과 동일한 번들 바이너리). 따라서 백엔드를 먼저 시작하지 않아도 새 셸에서 간단한 스크립트만으로도 작동합니다: `ChlorosLocal` (및 `process_folder()` / `process_lattice_capture()`), `connect_camera()`, `connect_array()`, `connect_daq_sensor()`, `discover_lattice_cameras()` 및 `discover_daq_sensors()`. `list_cameras()`, `list_arrays()`, `list_daq_sensors()`, `attach_array()` 및 `analyze_array_network()`는 절대 시작되지 않습니다. 이 프로세스들은 실행 중인 백엔드가 필요합니다. `auto_start_backend=False`를 전달하여 이 기능을 비활성화하십시오 (예: 절대 생성되지 않는 원격 백엔드를 지정하는 경우). [백엔드 자동 시작](#backend-auto-start)를 참조하십시오. Surface 3의 동작은 다릅니다. `open_project()`는 `auto_start_backend` 매개변수를 받지 않으며, `connect_all()`는 백엔드를 절대 생성하지 않습니다 — `http://127.0.0.1:5000`를 한 번 탐색하고, 응답이 없으면 백엔드가 없는 직접적인 `lattice_sdk` 장치 제어 방식으로 전환됩니다. 이 방식은 Python에서 임포트할 수 있는 Arena SDK 런타임과 OpenCV (`cv2`)를 Python에서 임포트할 수 있어야 하는데, SDK 휠은 이를 설치하지 않습니다; 이들이 없으면 저장된 각 카메라와 배열이 오류로 보고되며, 그 이유도 함께 표시됩니다. 프로젝트의 광 센서에는 백엔드 없이 사용할 수 있는 경로가 없습니다. 백엔드가 실행되지 않으면 저장된 각 센서가 오류로 보고되며, 그 이유도 함께 표시됩니다([`SensorHandle`](#sensorhandle) 참조). 오직 `proj.process()`와 `stream(..., overlays=True)`만이 `ChlorosLocal()`를 지연 생성하며(이것은 자동 시작 기능을 수행합니다).

이 세 가지 모두 인증이 필요합니다-gated 방식입니다. 해당 컴퓨터에서 `chloros-cli login`를 한 번 실행하거나, 데스크톱 GUI를 통해 로그인하십시오. 유효한 세션 없이 호출할 경우 발생하는 오류는 인터페이스에 따라 다릅니다([예외](#exceptions) 참조): `ChlorosAuthenticationError`는 `ChlorosLocal`(및 `process_folder()` / `process_lattice_capture()`), 스마트-connect 호출 및 해당 세션에서 발생하는 `ChlorosConnectError`, `status: "error"`와 함께 `error_code`가 `analyze_array_network()`에서 발생하는 경우, 그리고 `connect_all()` 보고서 내의 항목.

요구 사항:
- Python 3.7 이상 (패키지 설명에 따름; 3.10에서 개발 및 테스트됨)
- 로컬에 Chloros Desktop이 설치되어 있어야 함(백엔드 바이너리는 설치 프로그램에 포함되어 제공됨)
- **유료 Chloros+ 요금제(Copper 이상)**중 하나에 대한 유효한 Chloros+ 로그인 계정; 무료**Iron**요금제는 SDK/CLI에 대한 액세스 권한이 없습니다. 이는**서버측**에서 강제 적용됩니다: SDK/CLI 플래그가 지정된 모든 요청은 활성 세션과 유료 요금제를 모두 포함해야 합니다. Copper 미만의 요금제를 사용하는 경우, 스마트 커넥트 호출에서 `403`와 `error_code: PLAN_UPGRADE_REQUIRED`(`ChlorosLicenseError`는 `ChlorosLocal`, 스마트 커넥트 호출의 경우 `ChlorosConnectError`). 로그아웃된 호출자는 대신 `401` / `AUTH_REQUIRED`를 받게 됩니다 (`ChlorosAuthenticationError`는 `ChlorosLocal`에서, `ChlorosConnectError`는 스마트 커넥트 통화에서 발생) — 이 두 코드는 서로 다른데, `chloros-cli login`를 다시 실행하면 실행하면 `401`는 해결되지만, 플랜 업그레이드가 필요한 `403`는 해결할 수 없기 때문입니다.
- 요금제의 유예 기간 내에서는 오프라인 사용이 지원됩니다. 티어는 서버 검증 캐시(5분) 또는 서명된 기기 전용 라이선스 캐시(월간 요금제의 경우 30일, 연간 요금제의 경우 구독 만료일까지)에서 읽혀집니다. 이 유예 기간이 만료되면 요금제는 무료 티어로 전환되며, 기기가 서버에 한 번 연결될 때까지 SDK/CLI 액세스가 중단됩니다. `chloros-cli status`(`GET /api/license-status`)는 무료 티어에서도 계속 접근 가능하므로 오류 원인을 확인할 수 있습니다.
- Windows 10/11 64비트, **Ubuntu 22.04 LTS 이상**또는 Jetson(JetPack 6). Ubuntu 20.04는**지원되지 않습니다**: `.deb`의 종속성은 `libc6 (>= 2.35)`를 포함하여 백엔드가 링크하는 구성 요소에서 파생되며, Focal은 glibc 2.31을 제공합니다.

---

## 설치

Python SDK는 Chloros 백엔드 위에 구축된 얇은 Python 레이어입니다. 다른 컴퓨터에서 백엔드를 구동하는 경우([Remote-Backend Mode](#remote-backend-mode-pip-only-host-via-tunnel) 참조)를 제외하고는 **로컬에 Chloros 데스크톱 패키지가 설치되어 있어야** (Windows 설치 프로그램 또는 Linux `.deb`)를 로컬에 설치해야 하며, 여기에는 DAQ 광 센서가 포함됩니다. 이 패키지는 백엔드 바이너리, LATTICE 카메라용 Arena SDK 런타임, 그리고 보정 번들을 제공합니다.

최신 다운로드: [`https://mapir.gitbook.io/chloros/download`](https://mapir.gitbook.io/chloros/download)

### 1단계 — Chloros 플랫폼 패키지 설치

#### Windows (.exe)

1. 다운로드 페이지에서 `Chloros Installer 1.3.0.exe`를 다운로드합니다.
2. 설치 프로그램을 실행하고 마법사의 안내에 따라 진행하십시오. 기본 설치 경로는 `C:\Program Files\Chloros\`입니다.
3. Chloros를 한 번 이상 실행하고 Chloros+ 계정으로 로그인하십시오.

설치 프로그램은 모든 네트워크 프로필의 Windows 방화벽을 통해 백엔드(`chloros-backend.exe`, 그 외에는 없음)이 모든 네트워크 프로필에서 Windows 방화벽을 통과할 수 있도록 허용합니다: 카메라 스트림, PTP, mDNS 및 DAQ-E 스트림을 위한 UDP 인바운드, 그리고 펌웨어 업데이트 중에 DAQ-E가 컴퓨터로 다시 연결할 때를 위한 로컬 서브넷에서만 허용되는 TCP 인바운드 ([`update_firmware()`](#daq-e-device-settings))를 위해 로컬 서브넷에서만 허용되는 TCP 인바운드 연결을 허용합니다. 제거 프로그램은 이 두 가지 규칙을 모두 제거합니다. 백엔드의 HTTP API는 어느 경우든 `127.0.0.1`에 유지됩니다.

마법사의 **시스템 설정** 그룹에는 두 가지 옵션이 있으며, 둘 다 기본적으로 선택되어 있습니다:

| 옵션 | 기능 | 자동 설치 제외 |
| --- | --- | --- |
| **Windows 긴 경로 허용** | 프로젝트 폴더가 Windows의 260자 경로 제한보다 더 깊은 위치에 있을 수 있도록 허용합니다. 긴 경로 기능이 비활성화된 상태(Windows 기본값)에서는, 제품에 비해 너무 깊은 위치에 있는 프로젝트 폴더는 어떤 처리도 이루어지기 전에 거부됩니다 — [문제 해결](#troubleshooting-pointers) 참조). | `/MERGETASKS="!longpaths"` |
| **Chloros 백엔드를 Microsoft Defender 검사에서 제외** | 카메라 캡처가 기록하는 모든 파일에 대한 Defender의 실시간실시간 스캔을 중지합니다. 높은 캡처 속도에서는 스캔으로 인해 처리 속도의 약 30%가 소모됩니다: 9대 카메라 어레이에서 ROI 512로 측정했을 때, 연속 RAW 캡처 시 스캔을 포함하면 52.7 캡처/초, 백엔드를 제외하면 77.7 캡처/초로 실행되었습니다. | `/MERGETASKS="!defenderexclusion"` |

지속적인 캡처 시 Defender의 부하가 `defender_cores`로 보고됩니다([`ContinuousHandle`](#continuoushandle) 참조).

#### Linux amd64 (.deb)

```bash
sudo apt install ./chloros_1.3.0_amd64.deb
chloros-cli --version
chloros-cli login user@example.com 'YourPassword'
```

`dpkg -i`가 아닌 `apt install ./…`로 설치하십시오. apt는 Python SDK의 필수 구성 요소도 함께 설치하는데, 이는 패키지에서 필수로 지정하기보다는 권장하는 사항입니다([2단계](#linux-deb) 참조).

#### Linux arm64 — Jetson (JetPack 6)

```bash
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
chloros-cli --version
chloros-cli login user@example.com 'YourPassword'
```

JetPack 6(L4T R36)을 실행하는 Jetson Orin / Orin NX / Orin Nano용입니다. 이 버전의 PyTorch는 JetPack 6의 CUDA 빌드이므로 GPU 처리는 Orin의 GPU에서 실행됩니다. 또한 JetPack 6이 설치하는 CUDA 및 cuDNN 라이브러리를 사용합니다(해당 라이브러리가 없는 이미지의 경우 `sudo apt install nvidia-jetpack` 참조).

#### Linux: 수신 버퍼 및 백엔드 서비스

두 .deb 패키지 모두 LATTICE 카메라 스트리밍에 필요한 커널의 소켓 수신 버퍼를 32 MB로 늘립니다(`/etc/sysctl.d/60-chloros-gige.conf`). 커널 기본값에서는 카메라 패킷이 커널에서 드롭되어 어레이에서 프레임과 프레임 속도가 손실됩니다. `chloros-cli lattice network`는 버퍼 용량이 부족할 때 이를 보고합니다. CLI 참조 문서의 [Linux 호스트 — 소켓 수신 버퍼](cli-reference.md#linux-hosts--socket-receive-buffers)를 참조하십시오.

헤드리스 머신에서는 스크립트가 실행 중인 백엔드를 찾을 수 있도록 **사용자 권한으로** 백엔드를 서비스로 실행하십시오(`connect_all()`는 백엔드를 절대 시작하지 않습니다):

```bash
sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
```

그러면 백엔드는 사용자로 실행됩니다. 즉, 백엔드가 기록하는 캡처 데이터는 사용자의 것이며, 사용자의 `chloros-cli login`를 사용합니다. 루트 `chloros-backend.service`는 이를 활성화한 시스템에 여전히 설치되어 있지만, 루트 백엔드가 기록하는 모든 파일은 루트 소유가 됩니다. CLI 참조 문서의 [서비스로 백엔드 실행](cli-reference.md#running-the-backend-as-a-service-headless)을 참조하십시오.

서비스가 실행하는 백엔드는 공유되며, 클라이언트에 의해 중지되지 않습니다: SDK 스크립트와 `chloros-cli`는 발견되는 대로 이를 사용합니다 (`ChlorosLocal.shutdown_backend()`는 `ChlorosLocal`가 직접 시작한 백엔드만 중지하며), `GET /api/status`는 해당 유닛의 이름을 `service`로 지정합니다. `sudo systemctl restart chloros-backend@$USER.service`로 다시 시작하십시오. 패키지를 업그레이드하면 설치 중에 백엔드 서비스가 중지되므로, 따라서 실행 중인 스크립트는 카메라와 센서 연결을 잃게 되며, 이후 활성화된 모든 장치를 다시 시작합니다. 패키지를 제거하면 서비스가 중지되고 활성화된 장치가 비활성화됩니다. 한 번에 하나의 백엔드만 실행되므로, 장치를 하나만 활성화하십시오. 두 번째 장치를 활성화하면 시작을 거부하고 `failed` (종료 상태 98, 재시작되지 않음) 상태가 되며, 해당 유닛의 `systemctl status`에서 그 이유를 확인할 수 있습니다. 유닛을 전환하려면 사용하지 않는 유닛을 비활성화하십시오.</id></sn>

### 2단계 — Python SDK 설치

**Chloros 설치 프로그램에는 호환되는 SDK 휠이 포함되어 있습니다.** 모든 Windows 설치 프로그램과 Linux .deb 패키지는 GUI/CLI/백엔드 버전과 정확히 일치하는 `chloros_sdk-X.Y.Z-py3-none-any.whl` 파일을 디스크에 생성합니다. 버전을 최신 상태로 유지하기 위해 PyPI를 따로 찾아다닐 필요가 없습니다.

#### Windows

설치 프로그램은 시스템 Python을 사용하여 번들된 휠에 대해 `pip install`를 자동으로 실행합니다(`py.exe` 런처가 우선되며, 그렇지 않을 경우 `python -m pip`로 대체됨). 별도의 조치가 필요하지 않습니다. 설치가 성공적으로 완료되면 `import chloros_sdk`가 사용자의 Python 환경에서 작동합니다. 시스템에 Python이 설치되어 있지 않은 경우, 설치 프로그램은 이 단계를 자동으로 건너뛰며 GUI와 CLI은 계속 작동합니다.

#### Linux(.deb)

두 .deb 패키지(x86_64 및 Jetson) 모두 시스템용 SDK인 `python3`를 설치하므로, 설치 후 pip나 네트워크 연결 없이도 `import chloros_sdk`가 작동합니다. 패키지 (`chloros_sdk`, `lattice_sdk`, `lattice_cam`, `arena_api` 및 프로젝트 로더)는 `/usr/lib/python3/dist-packages`에 포함되며, SDK의 `chloros-mcp` 명령어([MCP 서버](#mcp-server-ai-assistants))는 `/usr/bin`에 포함되며, 이 모든 것은 해당 패키지가 소유합니다. 업그레이드 시 이 파일들이 대체되며, 패키지를 제거하면 이 파일들도 함께 제거됩니다.

SDK 자체의 필수 구성 요소인 `python3-numpy`와 `python3-requests`는 **필수 사항이 아닌 권장 사항**입니다(GUI, CLI 및 백엔드는 독립적으로 작동합니다): `sudo apt install ./chloros_<version>_amd64.deb`(Jetson의 경우 `…_arm64_jp6.deb`)는 이를 설치하지만, `dpkg -i`는 설치하지 않습니다. 설치 출력 결과에는 `python3`가 가져오는 SDK 버전, 누락된 요구 사항, 그리고 `python3`의 경로 상에 있는, 해당 패키지의 복사본을 가리는 `chloros_sdk`(예: 이전에 `pip install --user`로 설치된 것)를 표시합니다.

시스템 패키지를 인식하지 못하는 가상 환경(virtualenv)에서는 번들된 휠을 해당 환경에 설치하십시오:

```bash
pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

에어갭(air-gapped) Jetson 배포 환경의 경우, 이 과정은 완전히 오프라인으로 진행됩니다. 즉, SDK와 휠은 이미 디스크에 존재합니다.

#### 공개 PyPI

pip 전용 호스트(Chloros 데스크톱 패키지가 설치되지 않은 경우; 이 경우 SDK는 다른 머신의 백엔드를 구동합니다. [원격 백엔드 모드](#remote-backend-mode-pip-only-host-via-tunnel) 참조)의 경우:

```bash
pip install chloros-sdk
```

PyPI에는 최신 안정 버전이 제공됩니다. SDK는 이를 구동하는 Chloros 백엔드와 동일한 버전을 유지해야 합니다([버전 관리 및 호환성](#versioning--compatibility) 참조). DAQ-M 헬퍼 `DLS`에는 `bleak`: `pip install "chloros-sdk[camera]"`(또는 `pip install bleak`)도 필요합니다.

#### 확인

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)
```

> **Chloros+ 구독이 필요합니다.** 모든 SDK 호출에는 유효한 Chloros+ 로그인이 필요합니다. 각 머신당 `chloros-cli login user@example.com 'YourPassword'`를 한 번 실행하십시오. 자격 증명은 `~/.chloros/`에 캐시됩니다.

### 데스크톱 패키지가 필요한가요?

대부분의 워크플로우에서는 pip 패키지만으로는 **불충분**합니다. 각 SDK 표면에 필요한 사항은 다음과 같습니다:

| SDK 표면 | 데스크톱 패키지 필요? | 이유 |
| --- | --- | --- |
| `ChlorosLocal`, `process_folder`, `process_lattice_capture` | **예**(하나를 시작하기 위해)**/ 아니요** (이미 연결 가능한 백엔드가 있는 경우) | 실행 중인 백엔드(데스크톱 앱의 백엔드 또는 다른 스크립트로 시작된 백엔드)를 사용하거나, 그렇지 않은 경우 설치된 백엔드를 자동으로 시작합니다: `/usr/lib/chloros/chloros-backend` (Linux) 또는 `<install folder>\backend\chloros-backend.exe` (Windows; 기본값은 `C:\Program Files\Chloros\backend\`이며, 사용자 지정 설치 폴더는 설치 프로그램의 기록에서 읽어옵니다). 직접 이름을 지정하려면 `backend_exe=`를 전달하십시오. |
| `chloros-mcp` ([MCP 서버](#mcp-server-ai-assistants)) | **예**(시작할 경우)**/ 아니요** (이미 연결 가능한 백엔드가 있는 경우) | `ChlorosLocal`를 기반으로 하므로 위 행과 동일합니다. |
| `read_image_audit_tags` | **아니요** | 처리된 이미지의 자체 메타데이터를 읽습니다(Python 표준 라이브러리만 해당). |
| `connect_camera`, `connect_array`, `connect_daq_sensor`, `analyze_array_network`, `list_*`, `discover_*` | **예**(로컬)**/ 아니요**(원격) | 백엔드를 통한 순수 HTTP 클라이언트. 로컬 백엔드 → 데스크톱 패키지 필요. 원격 백엔드 →**터널을 통해** `backend_url=` (원격 백엔드 모드 참조 — 제공된 백엔드는 루프백에만 바인딩됨). |
| `ChlorosProject` / `open_project` | **예** | 백엔드를 통해 저장된 프로젝트를 실행합니다. |
| 직접 LATTICE 클래스(`LatticeCamera`, `CameraPool`, `Calibration`, `DLS`, …) | **예** | 데스크톱 패키지 내에 포함된 Arena SDK 런타임과, 데스크톱 패키지나 SDK 모두에서 설치되지 않는 Python용 OpenCV(`cv2`)가 필요합니다 (`pip install opencv-python`). 그렇지 않은 경우, `CAMERA_AVAILABLE`는 임포트 시 `False`로 처리됩니다. `DLS`에는 `bleak`도 필요합니다. |

### 원격 백엔드 모드 (터널을 통한 pip 전용 호스트)

> **설치된 백엔드에 LAN을 통해 접속할 수 없습니다.** 이 백엔드는
> 루프백에서만 수신 대기하므로(`127.0.0.1` / `::1`), `backend_url="http://<lan-ip>:5000"`는
> **설치된 Chloros에 연결할 수 없습니다**. 다른
> 머신에 있는 백엔드를 구동하려면, 해당 머신의 루프백 포트를 직접 포워딩하고 SDK가
> 터널을 가리키도록 설정하십시오:

```bash
# on the pip-only host: forward local 5000 to the Chloros machine's loopback
ssh -N -L 5000:127.0.0.1:5000 user@chloros-host
```

```python
import chloros_sdk

BACKEND = "http://127.0.0.1:5000"     # the tunnel endpoint
serials = ["213800234", "214000533"]   # master first

# auto_start_backend=False: the tunnel URL looks local, so without it a tunnel
# that is down would start this machine's own backend, if one is installed.
arr = chloros_sdk.connect_array(serials, backend_url=BACKEND,
                                auto_start_backend=False)
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-1.local",
                                     backend_url=BACKEND,
                                     auto_start_backend=False)

# The backend opens every path on ITS machine: pass absolute paths valid there.
# (A relative path would be made absolute against this machine's working folder.)
arr.capture("/home/chloros/captures/run1", processing="raw")
```

헤드리스/CI/로봇틱스 호스트의 경우, 전체 데스크톱이 설치된 한 대의 머신을 “Chloros 서버”로 유지하고 나머지 모든 곳에는 `pip install chloros-sdk`를 사용할 수 있습니다. 단, 이들 간의 통신은 위에서 설명한 사용자가 직접 구성한 터널을 통해 이루어지며, 직접적인 LAN URL을 사용하는 것이 아닙니다.

> **pip 전용 호스트의 `ChlorosLocal`.** `ChlorosLocal(backend_url=BACKEND)`는 데스크톱 설치 없이도 접근 가능한 백엔드에 연결됩니다. 이 명령은 백엔드를 시작해야 할 때(응답이 없거나 `auto_start_backend=True` 오류가 발생할 때)에만 로컬 백엔드 바이너리를 검색합니다.. 1.2.0 버전까지는 바이너리를 먼저 검색했으며, 백엔드에 접근할 수 있는 상황에서도 pip 전용 호스트에서 `ChlorosBackendError`("Chloros 백엔드를 찾을 수 없음…") 오류를 발생시켰습니다. 전달한 폴더 (`process_folder`, `import_images`)는 이 머신에서 확인되며 백엔드에서 읽어들입니다. 따라서 원격 백엔드를 사용할 경우, 공유 드라이브와 같이 양쪽에서 동일한 폴더 이름을 지정해야 합니다.

### DAQ 전용 워크플로우 (pip 전용 호스트)

`pip install chloros-sdk`만으로는 DAQ 센서를 구동할 수 없습니다. DAQ 센서는 항상 `connect_daq_sensor()`를 통해 Chloros 백엔드를 경유하여 구동됩니다:

- **이 컴퓨터의 센서:** 이 컴퓨터에 Chloros 데스크톱 패키지를 설치하십시오. `connect_daq_sensor()`는 백엔드가 실행 중이지 않으면 자체 백엔드를 시작합니다.
- **다른 시스템의 센서:** 해당 시스템에는 데스크톱 패키지가 필요합니다. [원격 백엔드 모드](#remote-backend-mode-pip-only-host-via-tunnel)의 터널을 통해 해당 백엔드에 접속하십시오. 백엔드는 자체 시스템에서 센서를 열며, 따라서 DAQ-U는 해당 머신에 연결되어 있어야 하고, DAQ-M은 해당 머신의 블루투스 범위 내에 있어야 하며, DAQ-E는 해당 머신이 접근할 수 있는 네트워크에 있어야 합니다.

```python
import chloros_sdk

# http://127.0.0.1:5000 is the local desktop install's backend, or the tunnel.
with chloros_sdk.connect_daq_sensor() as daq:   # smart-detect USB / BLE / ETH
    print(daq.model, daq.transport, daq.address)
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])
```

어느 쪽이든 백엔드는 ‘Chloros+’ 플랜을 적용합니다: 백엔드가 실행되는 머신(`chloros-cli login` 또는 데스크톱 앱)에 유료 티어(Copper 이상)로 로그인해야 합니다. 로그아웃된 백엔드나 Copper 미만의 요금제를 사용할 경우, 백엔드의 사유(`AUTH_REQUIRED` / `PLAN_UPGRADE_REQUIRED`)를 포함하는 `ChlorosConnectError` 오류가 발생합니다.

---

## 빠른 시작

```python
import chloros_sdk

# === Image processing ===
results = chloros_sdk.process_folder(
    "C:/DroneImages/Flight001",
    indices=["NDVI", "GNDVI", "OSAVI"],
)

# === Live LATTICE single-cam ===
with chloros_sdk.connect_camera("213800234") as cam:
    cam.set_settings(exposure_time=10000, gain=0.0)
    cam.capture("output/")

# === Live LATTICE synchronized array (GUI smart-prep flow) ===
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    arr.capture("output/", processing="reflectance")

# === Live DAQ spectral sensor ===
with chloros_sdk.connect_daq_sensor() as daq:    # smart-detect USB / BLE / ETH
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])

# === Drive a saved project end-to-end ===
proj = chloros_sdk.open_project("/path/to/project")
proj.connect_all()
proj.arrays["main_rig"].capture("output/", processing="reflectance")
proj.disconnect_all()
```

---

## 최상위 API 색인

```python
import chloros_sdk

# === Image processing (full pipeline) ===
chloros_sdk.ChlorosLocal                          # class
chloros_sdk.process_folder(...)                   # one-shot helper
chloros_sdk.process_lattice_capture(...)          # LATTICE-friendly defaults
chloros_sdk.read_image_audit_tags(path)           # post-run audit

# === Live cameras (persistent backend pool) ===
chloros_sdk.connect_camera(serial, ...)           # → CameraSession
chloros_sdk.connect_array(serials, ...)           # → ArraySession (smart-prep)
chloros_sdk.attach_array(serials_or_id, ...)      # → ArraySession (attach without re-connecting)
chloros_sdk.list_cameras()
chloros_sdk.list_arrays()
chloros_sdk.discover_lattice_cameras()
chloros_sdk.analyze_array_network(...)            # network capability + recommendation
chloros_sdk.CaptureResult                         # list subclass returned by CameraSession.capture / ArraySession.capture
chloros_sdk.RecorderHandle                        # handle for an array record()/burst() job
chloros_sdk.ContinuousHandle                      # handle for an array continuous() capture
chloros_sdk.GrabResult                            # dict returned by grab() / yielded by frame_stream()
chloros_sdk.FrameFetchError                       # raised by grab()/frame_stream() on a refusal

# === Live DAQ sensors (persistent backend pool) ===
chloros_sdk.connect_daq_sensor(...)               # → DAQSensorSession
chloros_sdk.discover_daq_sensors()                # scan USB/BLE/ETH (finds a DAQ-M MAC)
chloros_sdk.list_daq_sensors()
chloros_sdk.daq_bindings() / set_daq_bindings(cameras=, arrays=)   # which DAQ feeds which camera
chloros_sdk.daq_band_presets(filter_code=None)    # band sets for Ambient + Object reflectance
chloros_sdk.light_sensor_reflectance(ambient_id, object_id, ...)   # two DAQs -> reflectance + indices
chloros_sdk.time_sync_status() / time_sync_peers() / time_sync_cameras()   # PTP grandmaster
chloros_sdk.restart_time_sync() / set_time_sync_priority(priority1=, priority2=)

# === Camera utilities (the CLI's lattice commands; by serial) ===
chloros_sdk.camera_color_profile(serial) / set_camera_color_profile(serial, ...)
chloros_sdk.camera_color_adjust(serial) / set_camera_color_adjust(serial, ...)
chloros_sdk.camera_highlights(serial) / set_camera_highlights(serial, value)
chloros_sdk.camera_white_balance_once(serial) / set_camera_white_balance(serial, ...)
chloros_sdk.camera_presets() / apply_camera_preset(serial, preset)
chloros_sdk.camera_filter_models() / camera_filter_model_info(name)
chloros_sdk.camera_firmware_status() / update_camera_firmware(serial)
chloros_sdk.camera_power_probe(serial) / sleep_camera(serial) / wake_camera(serial)
chloros_sdk.calibrate_camera_with_target(serial, ...)
chloros_sdk.network_adapters(...) / fix_network_settings()
chloros_sdk.apply_vignette_correction(input_dir, output_dir, ...)
chloros_sdk.read_dls(mac, ...) / probe_system(...) / system_info()

# === Transient cameras, alignment profiles, indices (cameras opened for one command) ===
chloros_sdk.start_camera_continuous(serial, ..., daq=None, daq_tolerance_ms=None)  # → CliJob; a DAQ bound to the camera records one .daq per frame (daq=False: none)
chloros_sdk.start_multi_camera_stream(serials, ...)   # → CliJob
chloros_sdk.capture_cameras_transient(serials, ...)
chloros_sdk.test_array_sync(serials, count, ptp_settle_s=None) / detect_array_master(line) / analyze_array_models(models)
chloros_sdk.calibrate_alignment_profile(serials, save_path, ...)
chloros_sdk.capture_aligned_multiband(profile_path, ...) / start_aligned_stream(profile_path, ...)  # → dict / CliJob
chloros_sdk.alignment_profile_info(path) / reorder_alignment_profile(path, ...)
chloros_sdk.index_catalog() / compute_index(input_path, ...) / capture_live_index(profile_path, ...)
chloros_sdk.CliJob                                # stats() / stop() / wait()

# === Project lifecycle ===
chloros_sdk.open_project(path, relink=None)       # → ChlorosProject
chloros_sdk.project_folder() / set_project_folder(path) / reset_project_folder()
chloros_sdk.run_project_recipe(recipe, project_path)  # `chloros-cli project run`
chloros_sdk.ChlorosProject                        # class
chloros_sdk.AlignmentSpec                         # dataclass
chloros_sdk.ArrayHandle, CameraHandle, SensorHandle

# === Direct-hardware (no-backend) classes (from lattice_sdk) ===
chloros_sdk.LatticeCamera, CameraSettings, PRESETS, CameraPool
chloros_sdk.Calibration, CalibrationCoefficients, FilterModel, list_filters
chloros_sdk.DLS, NetworkDiagnostics               # DLS: DAQ-M over Bluetooth; needs bleak

# === Exceptions ===
chloros_sdk.ChlorosError                          # base
chloros_sdk.ChlorosBackendError
chloros_sdk.ChlorosLicenseError
chloros_sdk.ChlorosConnectionError
chloros_sdk.ChlorosProcessingError
chloros_sdk.ChlorosProjectFilesMissingError      # a ChlorosProcessingError
chloros_sdk.ChlorosAuthenticationError
chloros_sdk.ChlorosConfigurationError
chloros_sdk.ChlorosConnectError                   # raised by smart-connect surface
chloros_sdk.LatticeError, CameraNotFoundError, ...  # from lattice_sdk

# === Availability flags ===
chloros_sdk.CAMERA_AVAILABLE     # True iff lattice_sdk imported cleanly
chloros_sdk.PROJECT_AVAILABLE    # True when chloros_sdk.project imported; open_project()
                                 # still raises ImportError without the project loaders
```

---

## 이미지 처리 — `ChlorosLocal`

주요 파이프라인 클래스입니다. 생성 시 백엔드를 시작하고(응답하는 백엔드가 없거나 `auto_start_backend=True`인 경우), 프로젝트를 생성 및 구성하며, 진행 상황을 모니터링하고, 실행 후 요약 정보를 반환합니다.

### 생성자

```python
ChlorosLocal(
    api_url="http://127.0.0.1:5000",   # backend URL (also: backend_url=)
    auto_start_backend=True,            # spawn backend if not running
    backend_exe=None,                   # override backend binary path
    timeout=30,                         # request timeout seconds
    backend_startup_timeout=60,         # backend boot timeout
    processing_timeout=14400,           # hard cap on process() (4 h)
    processing_stuck_timeout=1800,      # no-progress threshold (30 min)
)
```

### 메서드

| 메서드 | 설명 |
| --- | --- |
| `create_project(project_name, camera=None)` | 새 프로젝트를 생성합니다(선택적으로 `"Survey3N_RGN"`와 같은 카메라 템플릿을 사용하여). |
| `open_project(project_path, relink=None)` | 저장된 프로젝트(해당 폴더 또는 프로젝트 폴더 내의 파일명)를 열고 이를 현재 열려 있는 프로젝트로 설정합니다. 이동된 프로젝트의 경로는 현재 위치한 폴더로 재설정되며, `relink`는 누락된 다른 파일들이 현재 위치한 폴더의 이름을 지정합니다. 백엔드의 응답을 `source_files`로 반환하며, 아무것도 찾을 수 없는 경우 `ChlorosProjectFilesMissingError` 예외를 발생시킵니다. 아래의 **이동된 프로젝트**를 참조하십시오. |
| `import_images(folder_path, recursive=False)` | RAW/TIF/JPG/DNG 이미지 **및 `.daq` 광센서 기록**을 가져옵니다. `count` (이미지) 및 `scan_count` (기록 데이터)를 반환합니다. 폴더에 둘 다 없는 경우에만 경고를 표시합니다. |
| `export_light_sensor(daq=True, csv=True)` | 프로젝트의 모든 광센서 기록에 대해 보정된 `.daq` + `.csv`를 `<project>/Light Sensor/` 파일에 기록합니다. [광센서 기록](#light-sensor-recordings--calibrated-daq--csv)을 참조하십시오. |
| `configure(debayer=..., vignette_correction=..., reflectance_calibration=..., indices=[...], export_format=..., ppk=..., daq_log_path=..., input_level=..., radiometric_output=..., array_alignment=..., array_alignment_crop=..., array_alignment_interpolation=..., custom_settings=None)` | 처리 노브 설정. `indices`는 사전 설정 이름입니다(`NDVI`, `GNDVI`, `OSAVI`, …)이며, 각 프리셋은 해당 카메라의 밴드에 맞춰 프로젝트에 추가됩니다. 프리셋이 아니거나 해당 밴드로 구성할 수 없는 이름(예: `NDRE`는 RE와 NIR가 필요하지만, 이를 모두 제공하는 단일 카메라는 없음), `UserWarning`라는 이름이 부여되며 인덱스 이미지는 생성되지 않습니다. `daq_log_path`는 모든 프레임이 매칭되는 광센서 기록 파일 (`.daq` 또는 `.csv` 파일, 혹은 해당 파일들이 담긴 폴더)를 지칭하며, 이미지와 함께 가져온 기록 파일 대신 LATTICE와 Survey3 모두에서 모든 프레임이 이 기록 파일과 매칭됩니다; 경로가 없거나 해당 경로에 녹화 파일이 없는 경우, 가져온 녹화 파일이 사용됩니다. |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | 파이프라인을 실행합니다. `{"status": "complete", "async": False}`를 반환하며, 백엔드에서 키를 제공하는 경우 `summary` 키를 반환합니다 — [실행 후 요약 및 힌트](#post-run-summary--hints) 참조 — 실행의 출력 결과를 확인할 수 없는 경우 `export_check_failure`를, 이미지가 손실된 경우 `image_failure`를 반환합니다. `wait=False`는 `{"status": "started", "async": True}`를 한 번에 반환합니다. |
| `get_config()` / `get_status()` / `status()` | 백엔드 상태를 확인합니다. |
| `logout()` | 로그아웃: 캐시된 자격 증명을 지우고 이 머신의 장치 슬롯을 해제합니다(결과에서 `device_deregistration`). 진행 중인 처리 실행은 중지되지 않습니다. 시작된 모드 그대로 완료되며(결과에서 `run_in_progress: true` 및 `run_mode`), 다음 실행에서는 표준 처리가 사용됩니다. |
| `shutdown_backend()` | 이 인스턴스가 백엔드를 시작한 경우 백엔드를 중지합니다. 실행 중인 것으로 확인된 백엔드는 그대로 둡니다. `with` 블록이 남아 있거나 인스턴스가 가비지 컬렉션 중인 경우에는 백엔드가 중지되지 않습니다: 백엔드는 다음 스크립트를 위해 계속 실행되며, 30분 동안 이를 사용하는 것이 없으면 스스로 중지됩니다. 백엔드에는 종료 요청이 전송되므로, 먼저 카메라와 광 센서를 해제하고 녹화를 완료합니다; 10초 후에도 여전히 실행 중일 경우에만 종료됩니다. |
| `discover_cameras()` | **이 인스턴스의 백엔드를 통해** LATTICE 카메라를 탐색합니다 (`/api/camera/discover`). 딕셔너리 목록(`serial`, `model`, `ip`, …)의 리스트를 반환합니다 — GUI/CLI에서 확인되는 것과 동일한 형식입니다. 발견되지 않거나 백엔드에 연결할 수 없는 경우 빈 리스트를 반환합니다. |
| `camera_capture(output_dir, format="tiff", **settings)` |**백엔드를 통해**단일 프레임을 캡처합니다(이 핸들에 의해 자동 시작됨). 따라서 GUI/CLI와 동일한 사전 설정(기본 12비트, 풀 재사용, 내장된 보정 메타데이터)이 적용됩니다. `serial=` 또는 `device_index=`를 사용하여 대상을 해결합니다; `exposure`/`gain`/`pixel_format`/`preset`를 `**settings`로 전달하십시오. 레거시 메타데이터 딕셔너리(`filepath`, `width`, `height`, `pixel_format`, `exposure_time`, `gain`, `timestamp`). |
| `camera_stream(serial, *, fps=10.0, overlay=None, decode=True, connect_timeout=10.0, read_timeout=15.0)` | 풀링된 카메라에서 오버레이 합성 미리보기 프레임을 생성합니다 — 백엔드의 `/api/camera/<serial>/stream-annotated` 경로를 통해 얇은 MJPEG 클라이언트 (서버 측에서 그려진 제브라 / 그리드 / 십자선 / 히스토그램 / 피킹 / 스팟). `decode=True`는 BGR 배열을 반환하며 OpenCV(`cv2`)가 필요하지만, SDK에서는 이를 설치하지 않습니다; `False`는 원시 JPEG 바이트를 반환합니다. 또한 프로젝트별로 `ChlorosProject.stream(overlays=True)`로도 호출 가능합니다. |

**실행 중인 프로세스는 해당 프로젝트를 유지합니다.** 처리 실행이 진행 중일 때 — 이 스크립트, 앱, `chloros-cli process` 또는 `chloros-mcp`의 실행이든 — 백엔드는 프로젝트의 생성, 열기, 가져오기 또는 처리 설정 변경을 거부합니다 (HTTP 409 `PROCESSING_IN_PROGRESS`): `create_project()`와 `import_images()`는 `ChlorosProcessingError`를 발생시키고, `configure()`는 `ChlorosConfigurationError`를 발생시키고, 현재 실행 중인 모드와 다른 모드로 실행되는 `process()`는 `ChlorosProcessingError`를 발생시킵니다 (실행은 시작된 모드를 유지합니다), 각각 백엔드의 메시지와 함께 발생합니다. 프로젝트에 저장된 캡처는 실행이 종료되면 해당 프로젝트에 병합됩니다(해당 응답의 `project_add`가 이를 나타냅니다). 실행이 완료될 때까지 기다린 후(`get_status()`), 다음 단계로 진행하십시오.**이동된 프로젝트.** 프로젝트의 `project.json`는 각 소스 파일(모든 이미지 및 광센서 기록)을 전체 경로로 명명하므로, 프로젝트 폴더를 이동하거나 전체 프로젝트 폴더를 다른 드라이브로 옮기면 파일 이름에 이전 위치가 그대로 남아 있게 됩니다. 프로젝트를 열 때마다 가능한 한 자동으로 수정됩니다. 즉, 프로젝트 폴더 내에 있던 파일은 프로젝트가 현재 위치한 폴더에서 검색되며, 그곳에서 발견된 각 경로는 재설정되어 저장됩니다 (`project.json`는 먼저 `project.json.before-relink-<date>-<time>`로 백업됩니다). 경로 재지정은 해당 파일이 실제로 존재하는 경우에만 수행됩니다. 프로젝트 폴더 외부에 있던 파일들(허브 세션, ‘폴더 추가’를 통한 가져오기 등)은 `relink=<the folder they are in now>`(또는 그 상위 폴더)에서만 검색됩니다. `open_project()`는 `source_files`를 반환합니다: `total`, `found`, `missing`, `missing_roots`(누락된 파일이 있던 폴더), `missing_outside_project` 및 `relink`(경로가 재지정된 파일, 어디서 어디로). 일부 파일이 누락된 프로젝트는 나머지 파일과 함께 열리지만, 누락된 파일이 하나도 없는 프로젝트는 열리지 않으며, `open_project()`는 `ChlorosProjectFilesMissingError`(`.missing_roots`, `.audit`; HTTP 409 `PROJECT_SOURCES_MISSING`). CLI의 `project open --relink` 및 `chloros-mcp`'`open_project`(`relink_folder`)도 동일한 동작을 수행합니다.

확실한 정리 처리를 위해 컨텍스트 매니저로 사용하십시오:

```python
with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26", camera="Survey3N_RGN")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "GNDVI", "OSAVI"],
        export_format="TIFF (16-bit)",
    )
    results = cl.process(mode="parallel", wait=True)
print(results["summary"])
```

### 광센서 기록 — 보정된 `.daq` + `.csv`

DAQ-U / DAQ-M / DAQ-E는 보정 번들 **없이**도 기록할 수 있습니다. 이는
공개된 [`chloros_scripts`](https://github.com/mapircamera/chloros_scripts)
레코더(`record_daq.py`)가 기본적으로 수행하는 방식입니다. 이 레코더들은 원시 센서 카운트 값을 기록하고 파일에 타임스탬프를 찍어,
Chloros가 해당 센서의 공장 보정값을 **일련번호 순서대로** — 먼저 로컬 캐시
에서 먼저, 그 다음 MAPIR 클라우드에서 해당 센서의 공장 보정값을 가져와, 가져올 때 이를 적용합니다.

Chloros는 결과를 녹화당 두 개의 제품으로 다시 출력하며,
`<project>/Light Sensor/` 아래에 저장합니다:

| 제품 | 설명 |
| --- | --- |
| `<name>_calibrated.daq` | 재처리 가능한 아카이브 — 실시간 기록과 동일한 스키마이며, 이를 생성한 번들을 명시합니다. 이를 다시 가져와도 **두 번째** 보정은 수행되지 않습니다. |
| `<name>_calibrated.csv` | 센서 고유의 파장 그리드 기준 W/m²/nm 단위의 스펙트럼 조도. 측정값당 한 행씩 표시되며, 광도학적 열(총 전력, 광시/암시 럭스, PPFD 및 그 청색/녹색/적색 분할값, 피크 파장)이 추가됩니다. |
| `<name>_raw.daq` / `<name>_raw.csv` | **번들이 없는 센서 전용 (DAQ-A).** 센서의 원시 스펙트럼 카운트 — *조도*가 아님. 아래 참조. |

`<name>`는 기록 파일 자체의 이름입니다. 서로 다른 폴더에 있는 두 개의 기록이
하나의 — 두 대의 카메라가
동일한 밀리초에 기록한 캡처별 측정값(`<capture>/daq/daq_data_<sensor id>_<ts>.daq`)을 공유할 수 있으며, 각 기록은 여전히 고유한 산출값을 얻습니다: 첫 번째는
`<name>`를 유지하고, 다음 기록은 이름 **뒤에** 폴더명을 추가하여
(`<name>__<capture>_daq_calibrated.daq`) 구별되지만, 기록의 날짜와 시간은 여전히 앞쪽에 표시됩니다.

`process()`는 이 내보내기 작업을 단계 중 하나로 수행합니다. 이 작업에는 **이미지**가 필요하지 않습니다:
단독으로 운용되는 광 센서만으로도 충분한 워크플로우가 가능하며, 이러한 프로젝트는 구조상
이미지가 전혀 없습니다.

**DAQ-A 기록은 원시 카운트 형태로 내보내집니다.** DAQ-A 제품군은 시리얼별
번들 시스템이 도입되기 이전에 출시되었으며 가져올 번들이 없습니다. 대신 현장에서
반사율 타깃을 기준으로 보정되므로 번들이 필요하지 않았습니다. 이러한 기록 데이터는
`_calibrated`가 아닌 `_raw` 접두사로 내보내집니다. 파일 내부의 플래그가 아닌 별도의 파일명을 사용하는 이유는,
이 정보가 파일 이름만 있는 상태로 이메일을 통해 전송될 때도 그 의미가 유지되어야 하기 때문입니다.
`.csv` 헤더에는 `raw spectral sensor counts (NOT irradiance)`라고 명시되어 있으며,
값들이 파일 **내부**에서만 비교 가능하다는 경고가 표시됩니다. 이는 바로 타겟 기반 보정이
사용하는 방식이며, 센서 간 비교를 위한 것이 아닙니다. 전력에 의존하는 광도 측정 열(총 전력,
광시/암시 럭스, PPFD)은 카운트 값을 통합한 값이 아닌 **NULL**로 반환됩니다.

번들을 전혀 가져올 수 없었던 DAQ-U / DAQ-M / DAQ-E는 여전히 **건너뜁니다**.
원시 데이터로 기록되지 않습니다. 이 경우 번들이 존재하므로 “다시 연결하고 재처리”하는 것이 실질적인 해결책입니다.

구형 **v1.01 / v1.02** 기록 데이터(DAQ-A-SD가 이를 기록함)에는 판독별 에포크가 없으며,
오직 파일의 기록 시간만 포함됩니다. 이미지↔하향광 매칭기는 여전히 이를 거부합니다 —
프레임을 기록 시간과 매칭하는 것은 눈에 띄지 않게 오류가 발생하기 때문입니다 — 하지만 익스포터는 이를 읽으며,
생성된 CSV 파일에는 `clock=daq_created_on`가 출력되므로, 해당 제품이 어떤 시계를 기준으로 하는지 명시합니다.

```python
import chloros_sdk

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("DAQ-U_2026-08-26")
    cl.import_images("C:/Flights/raw_daq")     # .daq only — no camera involved
    result = cl.export_light_sensor()          # or just cl.process()

for rec in result["exported"]:
    print(rec["csv"])
for rec in result["skipped"]:
    print("skipped", rec["source"], "--", rec["reason"])
```

보정 번들을 가져올 수 없는 기록(오프라인이거나 파일에 보정 정보가 없는
센서)은 **사유와 함께** `skipped`로 보고됩니다. 이 기록은 원시 카운트 값을 포함하는 “보정된” 파일로
절대 출력되지 않습니다. 인터넷에 연결한 후
다시 실행하면 내보내기가 완료됩니다.

### 진행 상황 콜백

```python
def show_progress(percent, message):
    print(f"[{percent:3d}%] {message}")

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(indices=["NDVI"])
    cl.process(progress_callback=show_progress, poll_interval=1.0)
```

### 실행 후 요약 및 힌트

완료 시, `process()`는 `GET /api/processing-summary`를 가져와 본문을 `result["summary"]`로 첨부합니다. 이 가져오기 작업은 최선을 다해 수행되며, 성공적인 반환을 절대 차단하지 않습니다 — 요약 정보를 사용할 수 없는 경우, `process()`는 일반 `{"status": "complete", "async": False}` 형식으로 대체됩니다. `summary["hints"]`의 각 항목 — 제안된 수정 사항이 포함된 완전한 문장, 예: 실행 결과가 0인 이유 — 역시 Python `UserWarning`로 재출력되므로, 딕셔너리를 확인하지 않더라도 0 출력 실행은 자체 진단이 가능합니다:

```python
result = cl.process()
for hint in result.get("summary", {}).get("hints", []):
    print("HINT:", hint)
# hints also arrive on the warnings channel:
#   python -W always::UserWarning your_script.py
```

`summary["totals"]`는 기계가 읽을 수 있는 부분입니다:

| 키 | 집계 대상 |
| --- | --- |
| `models` | 실행 내 카메라 그룹 수. |
| `images_in_groups` | 해당 그룹에 포함된 원본 이미지. |
| `targets_found` | 감지된 반사율 타겟. |
| `images_calibrated` | 해당 실행에서 반사율 산출물을 생성한 이미지. |
| `images_failed` | 오류가 발생하여 내보내지지 않은 이미지. |
| `exported_files` | **해당 실행에서 생성된 이미지 산출물 파일**이 런에서 생성된**: 모든 형식(디베이어링, 미리보기, 방사도, 반사도, 인덱스, 타겟 렌더링)의 모든 산출물. |
| `reflectance_files` | 그중 반사도 산출물 (`Reflectance_Calibrated_Images`)는 타겟 또는 광센서(DAQ)의 조도 보정을 통해 생성되었는지 여부와 관계없이 포함됩니다. 타겟이 없음에도 반사도 데이터를 기록한 실행의 경우 힌트에 해당 내용이 표시되며, 반사도 데이터를 전혀 기록하지 않은 실행에 대해서만 반사도 출력이 누락되었다는 메시지가 표시됩니다. |
| `clamp_masks_written` | [언믹스 클램프 마스크](#unmix-clamp-mask--which-pixels-a-band-was-clamped-on)는 해당 실행 옆에 기록되어 있습니다. 이는 결과물이 아닌 보조 정보이므로 `exported_files`에는 포함되지 않습니다(CLI에서도 동일한 방식으로 별도로 집계합니다). |
| `captures_left_out` | 이 실행에서 촬영하지 않은 프로젝트의 `captures` 폴더에 있는 원본 프레임. 실행은 시작 시 이전 캡처 결과를 기다리기 때문에, 실행 중에 캡처가 저장되거나 대기 시간이 만료되지 않는 한 이 값은 0입니다. `summary["captures_left_out_note"]`에는 이에 대한 처리 방법이 설명되어 있습니다. |
| `daq_recordings_exported` / `daq_recordings_skipped` | 광센서 기록 데이터로, 의도적으로 별도로 집계됨 — 이들은 다른 단계에서 비롯되며 이미지가 전혀 없는 실행에서도 존재하므로, 이를 합치면 DAQ 전용 실행이 마치 이미지를 내보낸 것처럼 보일 수 있습니다. |

이와 함께: `summary["output_dirs"]` (기록된 모든 디렉터리),
`summary["light_sensor_export"]`, `summary["stopped"]` (사용자가 실행을 중단했을 때
참으로 표시되며, 따라서 부분 카운트가 생산량이 부족한 완료된 실행으로 인식되지 않도록), 그리고
`summary["groups"]` (그룹별 내역)가 있습니다.

`exported_files`는 파이프라인이 **쓰기 작업 중**에 기록하는 값이며, 나중에
프로젝트의 이미지 객체에서 스캔하여 추출하는 것이 아닙니다. 병렬 및 GPU 전략은 자체 이미지
객체를 생성하므로(GPU 경로의 경우 워커 하위 프로세스에서), 기존 스캔 방식은
`0 file(s) written`를 보고한 뒤, 모든 작업이 정상적으로 수행된 실행에서도
'zero-exports' 힌트를 출력했습니다. 이 번호를 기준으로 스크립트를 작성할 경우, 정상적인 병렬 실행에서는 이제
0이 아닌 카운트가 보고됩니다.

스킵된 파일의 경우, 리더가 각 파일에 대해 실제로 확인한 이유(읽을 수 없는 스키마, 누락된 번들, 쓰기 오류 등)가 **중복 제거**되어 보고되므로, 한 가지 원인으로 인해 스킵된 20개의 파일이 해당 원인이 20번 반복된 것으로 표시되지 않고 하나의 원인으로 처리됩니다.

> **실행 결과 이미지가 생성되지 않은 경우 `process()` 오류가 발생하지 않습니다.** 이는 SDK와
> CLI가 의도적으로 차이를 보이는 두 가지 경우 중 하나입니다. `chloros-cli process`는 "제품이 요청되었으나 아무것도
> 기록되지 않음"을 오류로 간주하고 0이 아닌 값으로 종료하는 반면, SDK는 정상적으로 반환하고
> 해당 상태를 보고합니다. 파이프라인이 빈 실행 시 중단되어야 한다면, 직접 확인하십시오
> — `summary` (또는 프로젝트 폴더 내 파일 수를 세어보세요)를 직접 확인하십시오.
> 예외가 발생하지 않았다는 사실에만 의존하지 마십시오. 일반적인 원인은 입력 폴더가
> 캡처로 인식되지 않았거나, 현재 설치된 카메라에 적용할 수 없어 생략된 산출물(예: RGB 전용
> 카메라에서 나온 복사도) 때문입니다.
>
> 실행의 내보내기 결과를 확인할 수 없는 경우에도 `process()`는 발생하지 않습니다. 대신 경고가 표시되고
> `result["export_check_failure"]`가 설정됩니다. 산출물은 작성됩니다. 이 경우 실행을 중지하려면 해당 키를 확인하십시오. 이미지를
> 잃어버린 실행도 마찬가지입니다: 경고가 표시되고 `result["image_failure"]`가 설정되며, `chloros-cli process`는
> 이 키가 설정된 경우 0이 아닌 값으로 종료됩니다.

### 편의 함수

```python
# One-call process: project + import + configure + process
results = chloros_sdk.process_folder(
    folder_path="C:/DroneImages/Flight001",
    project_name="FieldA_2026-05-26",
    camera="Survey3N_RGN",
    indices=["NDVI", "GNDVI", "OSAVI"],
    vignette_correction=True,
    reflectance_calibration=True,
    export_format="TIFF (16-bit)",
    mode="parallel",
    debayer="High Quality (Faster)",      # or "Texture Aware (Slow, Highest Quality)"
    ppk=False,
    recursive=False,
    processing_timeout=14400,
)

# LATTICE-friendly defaults (no panel-target detection, standard debayer)
results = chloros_sdk.process_lattice_capture(
    folder_path="C:/Captures/2026-05-13_Field",
    indices=["NDVI"],
)

# process_folder() / process_lattice_capture() also take input_level=,
# array_alignment=, array_alignment_crop=, array_alignment_interpolation=,
# progress_callback=, poll_interval=, timeout=, processing_stuck_timeout=,
# backend_exe= and api_url= (NOT backend_url=). indices defaults to ["NDVI"].

# Audit which calibration sources were applied to a processed image
tags = chloros_sdk.read_image_audit_tags("output/Reflectance_Calibrated/x.tif")
print(tags["CalibrationSource"])   # 'per_serial' / 'legacy_lookup' / 'none'
print(tags["VignetteSource"])      # 'per_serial' / 'legacy_polynomial' / 'none'
```

각 실행은 자체 `with ChlorosLocal(...)` 내에서 수행됩니다. 실행이 반환될 때, 실행이 시작해야 했던 백엔드는 계속 실행되며, 다음 호출을 위해 계속 실행되며, 30분 동안 아무도 사용하지 않으면 스스로 중지됩니다. 두 함수 모두 `auto_start_backend`, `radiometric_output` 또는 `daq_log_path`를 받지 않으므로, 해당 함수에는 `ChlorosLocal`와 `configure()`를 사용하십시오. `process_lattice_capture()`는 `camera`, `debayer` 또는 `ppk`를 지원하지 않습니다. 이 옵션은 패널 감지를 건너뛰고 표준 디베이어를 사용합니다.

### 지원되는 값

```python
# export_format
"TIFF (16-bit)"           # default, recommended
"TIFF (32-bit, Percent)"  # reflectance percentage as float32
"TIFF (8-bit)"            # all-8-bit images are written as this; never raised
"PNG (8-bit)"
"JPG (8-bit)"

# debayer
"High Quality (Faster)"               # standard, default
"Texture Aware (Slow, Highest Quality)"  # neural debayer for Survey3 imagery, Chloros+ only.
                                         # Survey3 only: on a project holding LATTICE captures
                                         # (which always use the standard demosaic) configure()
                                         # or process() raises ChlorosConfigurationError; a
                                         # Texture Aware value the project saved earlier is set
                                         # to Standard with a UserWarning, as the app does. Needs a
                                         # CUDA GPU the backend's PyTorch can use (an NVIDIA
                                         # GPU, or a Jetson Orin's). Without one, process()
                                         # uses the standard debayer, and the backend prints
                                         # one "[DEBAYER] ... needs a CUDA GPU" line saying why.
"Standard (Fast, Medium Quality)"      # alias used internally for LATTICE

# input_level (LATTICE only; Survey3 .raw ignores)
"auto"        # default — infers from each file's XMP ProcessingLevel tag
"raw"         # force-treat as raw Bayer
"debayered"   # force-treat as Chloros's saved debayered (3-channel) product
"processed"   # force-treat as already-calibrated radiance

# array_alignment / array_alignment_crop (LATTICE arrays; None = keep saved setting)
True          # backend default — apply the module-to-module transform stamped
              # in each capture's Chloros:Alignment* XMP to every product
False         # export in native sensor geometry / skip the common-overlap crop

# array_alignment_interpolation (alignment warp resampling)
"bilinear"    # backend default
"nearest"     # preserves exact source DNs (no inter-pixel value mixing)
"cubic"
```

#### 방사계 출력 (LATTICE 다중 스펙트럼 파이프라인)

`process` 파이프라인의 LATTICE 다중 스펙트럼(M3C/M3M) 내보내기 수준 — `reflectance`(기본값), `radiance`, `sensor-response` 또는 `all`(이미지당 적용 가능한 모든 모드))는 프로젝트의 **"방사도 내보내기"**및**"반사도 내보내기"**** 산출물을 설정합니다: `radiance`는 복사도만 내보내고, `reflectance`는 반사도만 내보내며, `sensor-response`는 둘 다 내보내지 않습니다(데베이어링 처리된 산출물과 미리보기 산출물만 제공), `all`는 둘 다 내보냅니다. 디베이어링 및 미리보기 산출물은 이 레벨에서 변경할 수 없습니다. `configure()`에는 이를 위한 전용 키워드가 있습니다:

```python
with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("Field_A")
    cl.import_images("C:/Captures/lattice_flight")
    cl.configure(
        radiometric_output="radiance",   # reflectance (default) / radiance / sensor-response / all
        export_format="TIFF (32-bit, Percent)",
    )
    cl.process()
```

고급 우회 방법 — `custom_settings`를 통해 산출값을 작성하는 것 — 은 여전히 작동하지만, 이 방법은 전체 설정 블록을 대체한다는 점을 기억하십시오 (아래 경고 참조). `"Export <product>"` 키를 작성하십시오: 프로젝트 고유의 `"Export radiance"` / `"Export reflectance"` 설정은 이전의 `"Radiometric output"` 키보다 우선하므로, 해당 키만으로는 아무것도 변경되지 않습니다.

```python
cl.configure(custom_settings={
    "Project Settings": {
        "Processing": {"Export radiance": True, "Export reflectance": False},
        "Export": {"Calibrated image format": "TIFF (32-bit, Percent)"},
    }
})
```

`reflectance` (기본값)은 카메라의 복사도를 **타임스탬프가 일치하는 DAQ 하향 복사량**으로 나눈 값으로, 기록된 `.daq` (DAQ-U/M/E)**이미지와 함께 발견된 DAQ-M 네이티브 `.csv`**중 하나로 자동 해결됩니다. 로컬에 없는 카메라별 또는 DAQ 보정 번들은 첫 사용 시**AWS에서 자동으로 가져옵니다**. CLI는 이를`chloros-cli process`: `--radiance`/`--no-radiance`, `--reflectance`/`--no-reflectance`, `--debayered`, `--preview`에서 이 기능을 제공합니다.**이 측정값은 누구의 것인가.**캡처가 저장하는 `.daq`는 해당 캡처가 기술하는 카메라(들)의 측정값이며 — 바로 옆에 있는 `<file>.daq.cameras.json` 사이드카에 이름이 명시되어 있습니다 (단일, 동기화 및 배열 캡처; 원시 버스트의 경우 매니페스트에 각 구성원의 측정값이 명시됨) —이며,**해당 카메라에서 나온 이미지와만** 매칭됩니다. 따라서 자체 측정값이 없는 독립형 카메라는 더 이상 다른 카메라의 DAQ를 차용하지 않으므로, DAQ를 해당 카메라에 바인딩하거나 직접 기록해야 합니다. DAQ 자체의 기록 (`chloros-cli daq pool-record` 또는 GUI에서 시작된 녹화)나, 해당 사이드카가 없는 모든 `.daq`(구형 캡처 포함)는 여전히 모든 카메라에 적용됩니다.**햇빛 아래의 OCN 카메라.** 모든 OCN 보정 팩은 협대역 조명(대역당 LED 1개) 하에서 구축되지만, 햇빛은 여러 대역을 혼합합니다. Chloros 1.3.0은 대역 분리 후 일괄적인 햇빛 기반 보정을 적용하여, 햇빛 아래에서 OCN의 방사도, 반사도 및 지수 산출값을 개선합니다. 이 보정은 밴드 분리 처리가 이루어지는 모든 곳(`process()`, 캡처 파일, `grab()` / `frame_stream()` 및 라이브 뷰)에서, 이를 지원하는 OCN 팩에 대해 적용됩니다. RGN 카메라는 변경 사항이 없습니다. 이전 버전으로 촬영된 OCN 이미지도 촬영 당시 사용된 보정값이 저장된 컴퓨터에서 처리될 경우 이 보정을 적용받습니다(Chloros는 카메라가 한 번 연결되면 해당 카메라의 보정값을 유지합니다). 그렇지 않은 경우 이전과 동일하게 처리됩니다. 따라서 OCN의 복사도, 반사율 및 지수 값은 1.2.0 버전과 다릅니다.

> `custom_settings`는 계산된 설정 블록 전체를 **대체**합니다 (설계상 `configure()`의 다른 키워드와 유효성 검증을 우회합니다). 이를 사용할 때는 위 예시와 같이 필요한 모든 `Project Settings` 키를 포함시켜야 합니다.

#### 지수 픽셀 읽기 (NDVI 및 기타 식생 지수)

**정수 지수 래스터는 −1…1 맵이 아니라, 이미지별 대비 확장 결과입니다.** `TIFF (16-bit)`, `TIFF (8-bit)`, `PNG (8-bit)` 및 `JPG (8-bit)` 지수 제품(8비트 캡처 데이터가 8비트 TIFF로 저장된 경우 포함)은

```

DN = (value − IndexMin) / (IndexMax − IndexMin) × full_scale
```

여기서 `IndexMin` / `IndexMax`는 **해당 프레임 자체의 가장 낮은 유한 인덱스 값과 가장 높은 유한 인덱스 값**입니다. DN 0은 −1이 아니며, 풀 스케일은 +1이 아닙니다.

클로로스(Chloros)는 매핑이 가역성을 유지하도록 양쪽 끝점에 스탬프를 찍습니다:

| 태그 | 의미 |
| --- | --- |
| `Chloros:IndexScaling` | `linear_minmax` — 아래의 규칙 |
| `Chloros:IndexMin` | DN 0으로 저장된 인덱스 값 |
| `Chloros:IndexMax` | 전체 스케일로 저장된 인덱스 값 |

이들을 읽어온 후 반전시키고, 저장된 DN을 먼저 저장된 데이터 유형(dtype)의 최대값을 기준으로 0…1 범위로 정규화하여 하나의 표현식으로 모든 정수 형식을 포괄하도록 합니다:

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

읽기 작업에는 `-config pix4d.config`가 필요하지 않습니다. exiftool은 별도의 도움 없이 이러한 XMP 속성을 읽습니다. 이 설정은 해당 값을 *쓰기* 위해서만 필요합니다.

> **`IndexMin == IndexMax`**는 해당 프레임의 모든 픽셀이 동일한 인덱스 값을 가졌음을 의미합니다. 래스터는 중간 스케일로 채워져 있으며 나눌 대상이 없으므로 — 끝점을 사용하십시오.

> **`Chloros:Index*` 태그가 없다면 명시된 매핑이 없다는 뜻입니다.** LUT 내보내기는 픽셀이 LUT 임계값 창에 맞춰 그라데이션 색상으로 렌더링된 색상 처리된 결과물이며, 유한한 인덱스 픽셀이 없는 프레임은 명시할 범위가 없습니다. 두 경우 모두 매핑이 있다고 가정하지 마십시오 — `PixelScale`가 누락된 경우와 동일한 규칙이 적용됩니다.**스트레칭은 이미지별로 적용되므로, 한 비행의 두 프레임을 DN 기준으로 비교할 수 없습니다**— 각 프레임은 고유한 끝점을 가지므로, 동일한 NDVI 값이 서로 다른 이미지에서는 서로 다른 DN에 해당합니다. 한 비행 전체에 걸친 정량적 작업(구역별 통계, 고정 임계값, 시계열 분석)을 위해**`TIFF (32-bit, Percent)`**를 내보내십시오: 인덱스 래스터는 재스케일링이나 태그 없이**인덱스 값 자체**가 float32 형식으로 기록됩니다. 형식 이름과 달리, 인덱스 산출물은 ×100이 아닙니다 — 해당 스케일링은 정수 데이터형 반사율에만 적용됩니다.

#### 언믹스 클램프 마스크 — 특정 밴드가 클램프된 픽셀

3×3 스펙트럼 언믹스 처리로 인해 밴드가 음수 값을 가질 수 있습니다. 이는 빛에 해당 밴드가 거의 포함되지 않았거나, 장면이 카메라의 언믹스 행렬이 설명하는 범위를 벗어난 경우입니다. 이는 실제 값이 아니므로, 내보내기 과정에서 0으로 고정됩니다 — 라이브 뷰에도 동일한 0이 표시됩니다. 밴드는 몇 개의 픽셀에서만 클램핑될 수도 있고 거의 모든 픽셀에서 클램핑될 수도 있으며, 마스크는 두 경우 모두를 기록합니다. `Chloros:Band{R,G,B}UnmixClampFraction`는 프레임의 어느 정도에서 그런 현상이 발생했는지를 나타내고, 클램프 마스크는 **어떤 픽셀**에서 발생했는지를 나타냅니다:

- **`clamp_masks/<product folder>/<product name>_clampmask.tif`**— 제품 폴더**옆에** 있는 `clamp_masks/` 폴더 내에, 절대 그 내부에는 위치하지 않습니다(`radiance/x_radiance.tif` → `clamp_masks/radiance/x_radiance_clampmask.tif`) — 적어도 하나의 픽셀이 클램핑된 모든 방사도, 반사도 및 지수 제품에 대해 생성됩니다. 단일 대역 `uint8`의 경우, 해당 제품의 픽셀 그리드(방향, ROI, 비닝 및 배열 정렬 워프 포함). 각 마스크는 해당 제품의 카메라(`Make` / `Model` / 일련번호), 촬영 시간 및 렌즈 태그를 명시하며, 비트가 나타내는 밴드를 나열합니다 (`XMP-Camera:BandName`, 비트 0부터 순서대로); `ImageDescription` 및 `Software`는 이를 클램프 마스크로 표시하며, 보정 정보는 포함되어 있지 않습니다.
- **비트**: 비트 0 = 제품의 R 채널에 해당하는 대역(`Chloros:BandRName`), 비트 1 = G(`BandGName`), 비트 2 = B (`BandBName`). 비트가 설정되어 있으면 해당 밴드의 픽셀 값이 0으로 클램핑되었음을 의미하므로, 이 비트와 이를 기반으로 생성된 모든 인덱스를 신뢰도가 낮은 것으로 간주하십시오.
- **`Chloros:UnmixClampMask`**는 언믹스(unmix) 처리를 수행한 모든 제품에 찍혀 있습니다: 제품 자체 폴더를 기준으로 한 마스크의 상대 경로(`../clamp_masks/<product folder>/<product name>_clampmask.tif`, 항상 `/`-로 구분됨 — 이를 이미지 폴더에 결합하면 마스크가 열립니다), 또는 픽셀이 클램핑되지 않은 경우 `none`로 표시됩니다. 태그가 없는 경우 해당 제품은 언믹스 처리를 거치지 않았음을 의미합니다(RGB 또는 모노 카메라, 원본/데베이어링된 레벨).
- 캡처 결과에는 프레임의 **`clamp_mask`**(마스크의 경로, 클램핑된 항목이 없을 경우 `None`)로 포함됩니다 — `CameraSession.capture()` 결과에서는 `ArraySession.capture()`'`CaptureResult` 프레임 및 HTTP 캡처 응답에 포함됩니다. 오프라인 빌드에서는 해당 마스크를 `clamp_masks`로 나열합니다.
- 라이브 인덱스의 멀티밴드 스택(`/api/camera/cli-array-index`와 `save_multiband`)은 소스 카메라당 하나의 마스크를 가지며, 이는 출력 폴더 내의 `clamp_masks/`에 저장됩니다 — CLI 참조'`lattice index`를 참조하십시오. 동영상에는 마스크가 없으며, 대신 오프라인 빌드 결과에서 각 프레임의 클램프 비율을 보고합니다.

어떤 깊이에서도 마스크를 포함하는 제품 폴더가 없기 때문에, 제품 폴더는 하위 폴더를 포함하여 있는 그대로 사진측량 소프트웨어에 전달할 수 있으며, 소프트웨어는 이미지만 처리합니다. 제품 폴더를 전달할 때는 그 위의 폴더가 아닌 해당 제품 폴더를 전달하십시오. Chloros의 자체 가져오기 기능은 마스크를 건너뜁니다.

```python
import os, numpy as np, tifffile

mask = tifffile.imread(
    "clamp_masks/radiance/capture_<ts>_SN<serial>-<filter>_radiance_clampmask.tif")
# or, from the image: os.path.join(os.path.dirname(image_path), tag), with
# tag = the image's Chloros:UnmixClampMask
r_clamped = (mask & 1) != 0      # the band Chloros:BandRName names
g_clamped = (mask & 2) != 0
b_clamped = (mask & 4) != 0
```

---

## LATTICE 카메라용 Smart-Connect

라이브 하드웨어를 위한 지속적 백엔드 세션입니다. GUI에서 사용하는 것과 동일한 엔드포인트를 사용하므로, SDK / CLI / GUI 전반에 걸쳐 동작이 동일합니다.

### 단일 카메라 — `CameraSession`

```python
import chloros_sdk

# Open by serial; reuses existing pool entry if one exists
with chloros_sdk.connect_camera("213800234") as cam:
    # cam is a CameraSession; supports context manager + manual disconnect
    cam.set_settings(
        exposure_time=10000,    # microseconds
        gain=0.0,               # dB
        pixel_format="BayerRG12",
        target_brightness=80,
        ae_damping=8.0,
    )
    cam.capture("output/", ext=".tiff")
```

여기서 설정된 노출, 게인 및 타겟은 사용자의 설정으로 유지됩니다. Chloros의 자동 노출 제어(DAQ 측정 노출 포함)는 이 설정들을 변경하지 않습니다. `cam.set_settings(exposure_auto="Continuous")`(또는 `gain_auto=`)는 노출(게인)을 다시 넘겨주고, `cam.set_settings(target_brightness_auto=True)`는 타깃을 Chloros의 자동 타깃 설정으로 되돌립니다.

#### `connect_camera()` 시그니처

```python
connect_camera(
    serial,
    *,
    preset=None,                       # "default" | "high_quality" | "high_speed" | "triggered"
    settings=None,                     # dict overlaid on the preset
    backend_url="http://127.0.0.1:5000",  # deliberately not 'localhost' (::1-first on Windows ≈ 2 s/request)
    timeout=60.0,
    auto_start_backend=True,           # spawn a local backend if none is running
) -> CameraSession
```

- **스트림이 시작된 상태로 반환됩니다.** 백엔드는 대기하며, 대기 시간은 제한되어 있습니다(스트림이 시작될 때까지 약 15초, 그 후 첫 번째 버퍼가 완전히 채워질 때까지 10초 — 약 1.5초 후에 이어지는 첫 번째 미리보기 프레임까지는 포함되지 않음), 따라서 바로 뒤에 `status()`가 나타나고, 그 다음에는 `streaming`가 나타납니다. 세션의 `connect_stream`는 응답의 `stream` 블록입니다: `state` `streaming`, `no_frames` (스트림이 시작되었으나 아무것도 전송되지 않았음 — `note`가 이를 나타내며, 점보 패킷을 사용하는 카메라의 경우 1500바이트 패킷 크기로 재연결해야 함), `starting` 또는 `start_failed`, 여기서 `waited_s`. 이미 연결된 카메라의 경우 `None`. 미리보기 (`get_camera_preview`, `/api/camera/jpeg/<serial>`)는 요청 직후, “프레임이 아직 없습니다(No frame available yet)”라고 응답하는 대신, 카메라의 첫 번째 미리보기 프레임을 기다리며(4초 제한) 대기합니다. "아직 사용할 수 있는 프레임이 없습니다"라고 응답하는 대신, 카메라의 첫 번째 미리보기 프레임을 기다립니다(4초 제한); 스트림에서 아무것도 전송하지 않은 카메라(`no_frames`, 또는 하드웨어 트리거를 위해 활성화된 경우)는 여전히 즉시 응답합니다.
- **패킷 크기는 사용자가 직접 설정할 수 있습니다.** `settings={"gev_packet_size": 1500, "gev_packet_size_auto": False}`는 1500바이트 패킷으로 카메라를 열며, 프로브는 수행되지 않습니다(`chloros-cli lattice cam-connect --packet-size standard`도 동일하게 전송함).
- **한 번에 하나의 연결만 카메라 풀을 점유합니다**: 연결 열기, 패킷 크기 탐색, 첫 번째 보정 데이터 다운로드(이 컴퓨터에서 처음 사용하는 카메라의 경우 약 70초 소요), 스트림 시작. 그동안 카메라가 필요한 요청(`read_nodes`, `set_settings`, 캡처, 두 번째 연결)은 최대 5초 동안 대기한 후, 연결 중인 카메라 이름을 명시하며 HTTP 503 오류(`CAMERA_POOL_BUSY`, `ChlorosConnectError`)로 실패합니다; 잠시 후 잠시 후 다시 요청하십시오. 목록 조회 요청은 절대 대기하지 않습니다: `list_cameras()` (`GET /api/camera/pool`), `list_arrays()` 및 `CameraSession.status()`는 즉시 응답합니다. 연결 중인 카메라는 풀 응답의 `connecting` 항목 아래에 나열됩니다 (`serial`, `phase` — `opening the camera`, `rebalancing the pool bandwidth`, `loading its calibration`, `starting its stream`, `waiting for its stream` — 및 `elapsed_s`), 그리고 `status()`는 `connecting: True`를, `phase`는 `connecting: True`를 가리킵니다; 나머지 카메라들은 풀이 비어 있을 때 실시간으로 읽히며, 그렇지 않은 경우 마지막으로 읽은 값(`live: False`)을 반환합니다.

#### `CameraSession` 메서드

| 메서드 | 설명 |
| --- | --- |
| `status(timeout=15.0)` | `chloros-cli lattice status`가 읽어온 카메라의 실시간 상태: `streaming`, `stream_state` (`starting`, `no_frames`, `streaming`, `start_failed` 또는 `stopped`), `device` (모델, 일련번호, 펌웨어, 온도, `is_mono`) 및 현재 설정을 반환합니다. |
| `read_nodes(names, enum_names=(), timeout=30.0)` | GenICam 노드를 읽습니다. 자동 노출이 실행되는 동안 `{nodes, errors, enums, device, was_in_pool, in_array_id}`, `frame_values` 및 `notes`를 반환합니다(아래 참조). |
| `set_settings(**kwargs)` | 친숙한 이름으로 노드를 기록합니다(`exposure_time`, `gain`, `pixel_format`, `width`, `height`, `target_brightness`, `target_brightness_auto`, `ae_damping`, `ae_upper_limit`, `trigger_mode`, `trigger_source`, …). `exposure_time` (`gain`)는 수동 노출(게인)입니다. `exposure_auto` (`gain_auto`)를 생략하여 기록되며, 이 경우 먼저 자동 모드가 꺼지므로 실행 중인 자동 모드에 의해 값이 거부되지 않습니다. 결과인 `applied`에는 카메라가 촬영한 값이 저장되며; 카메라가 거부한 기록 값은 `errors`에 저장되며 유지되지 않습니다. 오류 메시지는 카메라가 현재 허용하는 범위를 명시하며(`ExposureTime`의 최대값은 카메라의 프레임 속도에 따름), 그리고 거부된 값에 대해서만 비활성화되었던 자동 모드는 다시 활성화됩니다; `restored`에 이를 나열합니다(예: `{"ExposureAuto": "Continuous"}`). |
| `capture(output_dir="output", ext=".tiff", jpeg_quality=95, processing=None, levels=None, force_daq=None, daq=None, daq_tolerance_ms=None, settings=None, timeout=None)` | **단일** 프레임을 캡처합니다. `CaptureResult`를 반환합니다: 프레임 메타데이터 딕셔너리 목록(저장된 레벨당 하나씩)으로, `.skipped`, `.failed`, `.daq_files` 및 `.daq_matches`가 포함됩니다. (연속/다중 프레임 캡처 기능은 제거되었습니다. 일련의 프레임이 필요한 경우 루프 내에서 `capture()`를 호출하십시오.) |
| `capture_repeated(output_dir="output", *, count=None, duration_s=None, interval_s=0.0, on_capture=None, **capture_kwargs)` | 이 카메라에서 연속(`interval_s=0`) 또는 간격 캡처를 수행하려면, `count` 캡처 및/또는 `duration_s`(최소 하나 이상이어야 종료됨)에 대해, 배열에 대한 `ArraySession.capture_repeated()`와 동일한 방식으로 수행됩니다. `CaptureResult` 목록을 반환하며, 드라이브가 가득 찼음을 보고하는 캡처가 발생하면 중단됩니다. `on_capture(i, result)`는 각 캡처 후 실행되며, 다른 키워드는 `capture()`로 전달됩니다. |
| `disconnect(keep_settings=False, clear_settings=False)` | 풀에서 해제하며, 카메라를 연결 시 발견된 상태(트리거 배선, PTP, 설정)로 복원합니다; `keep_settings=True`는 대신 설정을 자체 메모리에 저장하며, `clear_settings=True`는 해당 메모리를 초기화합니다( [카메라 메모리](#camera-memory--keep_settings--clear_settings) 참조); 둘 중 하나를 요청하면 `camera_memory`를 반환합니다. 배열의 구성원인 카메라는 전체 배열을 해제합니다. 이미 열린 세션에 연결된 경우 아무 작업도 수행하지 않습니다. 백엔드가 해제를 확인하거나, 해제할 대상이 없을 때(카메라가 이미 사라졌거나 백엔드가 실행 중이지 않을 때) 반환됩니다; 타임아웃이 발생했거나 백엔드가 거부한 해제 요청의 경우 `ChlorosConnectError`를 발생시킵니다. 이는 카메라가 여전히 점유되어 있을 수 있으며, 다음 연결 시 충돌이 발생할 수 있기 때문입니다. `with` 블록의 종료 시에는 절대 예외가 발생하지 않습니다. |

속성: `serial`, `device`(카메라의 검색 기록) 및 `backend_url`.

`capture()` 내보내기 제어(배열 및 GUI와 동일한 모델):

- `processing` / `levels` — `processing="all"`는 적용 가능한 모든 내보내기 유형을 저장하고, `levels=["raw","radiance"]`는 해당 유형만 저장합니다(`processing`를 재정의함). 백엔드 기본값을 사용하려면 두 항목 모두 생략하십시오.
- `daq` — 사용된 캡처를 읽은 DAQ는 기본적으로 `.daq` 사이드카로 저장되며, 여기에는 원시 데이터만 포함된 그랩이 포함되어 있으므로 나중에 프레임을 반사율/굴절률로 재처리할 수 있습니다. `daq=False`는 어떤 레벨에서도 이를 사용하지 않을 경우 이를 건너뜁니다. `force_daq=True`는 기본값의 구식 표기법입니다. DAQ가 없으면, `.daq`도 없습니다.
- `daq_tolerance_ms` — 노출 시점으로부터 얼마나 멀리 떨어져 있어도 허용 오차 범위 내에 포함되는지(기본값 500). 판정 결과만 변경됩니다.

#### 자동 노출 시의 `read_nodes()` — `frame_values` 및 `notes`

`ExposureAuto` (`GainAuto`)가 실행되는 동안, `ExposureTime` (`Gain`) 노드는 카메라의 현재 동작 상태가 아닌 **마지막으로 기록된 값**을 읽어옵니다. `nodes`는 여전히 해당 노드의 값을 유지하고 있습니다. 자동 모드가 `Off`가 아닌 연결된 카메라에서 `ExposureTime` 또는 `Gain`를 요청하면, 응답에 다음 내용이 추가됩니다:

- `frame_values` — 카메라의 최신 프레임이 전달한 값을 해당 노드 이름 아래에 표시합니다(`ExposureTime`는 µs 단위, `Gain`는 dB 단위);
- `notes` — 해당 노드마다 두 값을 모두 포함하는 한 문장.

```python
r = cam.read_nodes(["ExposureTime", "Gain"])
print(r["nodes"]["ExposureTime"])                  # last value written, e.g. 2000.448
print(r.get("frame_values", {}).get("ExposureTime"))  # what the newest frame used, e.g. 19600.0
for name, note in r.get("notes", {}).items():
    print(note)
# ExposureAuto is Continuous: ExposureTime holds the last value written (2000.448);
# the camera's newest frame was at 19600 us.
```

자동 모드가 `Off`일 경우(이때 노드는 카메라의실제 값이 됩니다)이거나 백엔드에 연결되지 않은 카메라의 경우 두 키 모두 존재하지 않습니다. 배열의 경우, `ArraySession.status()["member_ae"]`는 모든 구성원의 프레임 값을 한 번에 제공합니다.

#### DAQ 측정 노출 상한값 (HTTP)

DAQ 광 센서가 연결된 경우, Chloros는 측정된 일광을 기반으로 카메라의 노출 × 게인을 제한하여 흰색 기준 패널이 클리핑되지 않도록 합니다. 이 상한값은 보정 타겟이 해당 카메라의 시야 내에 있는 동안(지난 10초 이내에 프레임에 포착된 경우)에만 적용됩니다. 시야 내에 타겟이 없을 경우, 카메라의 일반적인 자동 노출(자체 하이라이트 보호 기능 포함)이 대신 작동하므로, 장면의 나머지 부분이 필요 이상으로 어둡게 처리되지 않습니다. SDK 래퍼는 제공되지 않으며, `GET` / `POST /api/camera/<serial>/daq-exposure-cap`를 사용하여 카메라별로 이 기능을 확인하거나 전환할 수 있습니다(POST 본문에 `{"enabled": false}`를 포함하면 해당 카메라에서 이 기능이 비활성화됩니다). 응답에는 `enabled`, `cap_us`(실시간 상한값, 비활성 상태일 때는 `None`이며, 보정 대상이 화면에 없는 경우도 포함), `k_exp_g`(상한값 계산에 사용되는 상수) 및:

| 필드 | 의미 |
| --- | --- |
| `k_exp_g_source` | 사용 중인 상수: `camera` — 이 카메라 자체의 상수(이 카메라의 보정 팩 내 랩 포인트로부터 계산됨 (RGB 카메라); `fleet` — 팩에 랩 포인트가 없는 경우 사용되는 플릿 상수; `override` — 백엔드에서 설정된 필드 오버라이드. |
| `k_exp_g_fleet` | 비교를 위한 플릿 상수. |

### 동기화된 어레이 — `ArraySession` (Smart-Prep)

`connect_array`는 다중 카메라 설정을 위한 **권장 진입점**입니다. 이 제품은 내부적으로 전체 GUI 스마트 프리프(Smart-Prep) 워크플로를 실행합니다:

1. **네트워크 분석**(`/api/camera/array/recommend`) —**전체 센서** (비닝 없음, ROI 자르기 없음)을 설정하고, Connect가 사용할 픽셀 형식과 패킹 방식을 고려하여 이 호스트와 네트워크가 제공하는 프레임 속도로 떨어지도록 합니다. 프레임 속도를 유지하기 위해 프레임을 축소하는 일은 절대 없습니다: 더 작은 `width` / `height` 또는 `binning=2` / `4`를 전달하여 해상도를 희생하고 프레임 속도를 확보할 수 있습니다(엔드포인트의의 `alternatives` 목록에는 예상 fps와 함께 해당 옵션들이 나열되어 있습니다).
2. **티어 자동 선택** — `sim-capture-sim-emit` (모든 카메라가 동일한 트리거에 노출되며, 와이어를 공유하도록 페이스가 조정되고, 공유 비율이 줄어들수록 프레임 속도가 떨어짐) 카메라들이 페이스를 조정하여 와이어를 공유할 수 있는 모든 네트워크에서; `slip-emit-and-capture` (카메라들이 통신선을 번갈아 사용하며, 동시에 노출되지 않음)는 카메라들이 통신선을 공유할 수 없는 경우에만 적용됩니다.
3. 기본적으로 **PTP 활성화**— 카메라 간 타임스탬프가 하나의 공유 클럭에**~1 ms**오차 범위 내에서 정렬되며, 백엔드의 [PTP 그랜드마스터](#ptp-grandmaster--time_sync_status)와 동기화됩니다. 동시 노출은 PTP가 아닌 M8 하드웨어 트리거(모듈 간**< 100 µs**)를 통해 이루어집니다. PTP는 *타임스탬프*를 동기화할 뿐, 노출을 동기화하지는 않습니다.
4. **카메라별 픽셀 형식 자동 선택** — 흑백 → `Mono12`, 컬러 → `BayerRG12`.
5. **AE 시딩** — 각 카메라의 현재 AE 상태를 스냅샷으로 저장하여, 연결 시 촬영 도중 노출 설정이 재설정되지 않도록 합니다.
6. **GPIO 트리거 구성** — `connect_array`가 모든 카메라(`TriggerMode=On`, `TriggerSource=Line2`)를 준비 상태로 설정하므로, 마스터의펄스가 M8 케이블을 통해 슬레이브를 구동합니다. 이는 어레이 전용 단계입니다: `LatticeCamera`로 단일 카메라를 열면 대신 자유 실행 모드로 전환됩니다. 어레이를 해제하면(`disconnect()` / `force_disconnect()`, `chloros-cli lattice array-disconnect`)을 해제하면 모든 카메라가 자유 작동(`TriggerMode=Off`) 상태로 되돌아가며, 그 외 모든 설정은 연결 시 확인된 상태 그대로 유지됩니다: 트리거 및 싱크 라인 설정 (`TriggerSource`, `TriggerActivation`, `TriggerOverlap`, `TriggerDelay`, `LineMode` / `LineSource`; 마스터의 싱크 라인은 입력으로 재설정), PTP, 그리고 세션 중에 변경된 카메라 설정 — 단, `keep_settings=True` / `clear_settings=True`인 경우를 제외하고 — [카메라 메모리](#camera-memory--keep_settings--clear_settings) 참조).

```python
import chloros_sdk

# First serial is the MASTER (fires the trigger pulse); rest are slaves.
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    print(arr.array_id, arr.sync_mode, arr.ptp_enabled)
    arr.capture("output/", processing="reflectance")
```

속성: `array_id`, `master_serial`, `slave_serials`, `ptp_enabled`, `sync_mode`, `packed_transport` 및 `packed_reason` (마지막 네 개는 `attach_array`에서 파생된 핸들의 `None`), `connect_stream` 및 `backend_url`입니다. `connect_stream`는 연결 응답의 `stream` 블록이며, 단일 연결의 형태에 `members` (`{serial: {state, why?}}`): 배열의 첫 번째 동기화된 그룹이 도착하면 (백엔드는 이를 약 10초 동안 유한한 시간 내에 대기함), 또는 각 구성원의 마지막 페치 실패를 `why`로 포함하는 `no_frames`이며, 점보 패킷의 경우 `note` 내의 1500바이트 폴백을 사용합니다. `None`는 `attach_array` 핸들에 적용됩니다.

#### `connect_array()` 시그니처

```python
connect_array(
    serials,                              # list[str]; serials[0] = master
    *,
    line="Line2",                         # GPIO sync line: Line0 | Line2 | Line3
    target_fps=None,                      # pin the master trigger rate (auto if None); see set_target_fps
    force_tier=None,                      # override tier picker; see below
    wire_ceiling_mbps=None,               # host sustained wire budget, MB/s (auto if None)
    width=None,                           # explicit frame size; skips network analysis
    height=None,
    pixel_format=None,
    binning=None,
    recommend=True,                       # set False to skip the recommend step
    ptp_enable=True,                      # set False to disable PTP
    packed="auto",                        # packed 12-bit transport: "auto" | "on" | "off"
    backend_url="http://127.0.0.1:5000",  # same IPv6-avoidance default as connect_camera
    timeout=600.0,                        # s; also the backend's deadline (see below)
    auto_start_backend=True,              # spawn a local backend if none is running
) -> ArraySession
```

`timeout`는 호출이 연결을 대기하는 시간이며, 이는
데드라인으로 백엔드에 전송됩니다. 그때까지 연결이 완료되지 않으면 작업이 취소됩니다. 즉, 호출이
열었던 모든 카메라는 발견된 상태로 되돌려지고 닫히며, 어떤 배열도 연결되지 않은 상태가 되고,
호출은 `ChlorosConnectError` (HTTP 504 `CLIENT_DEADLINE_PASSED`, 연결이 도달한
단계를 명시함)를 발생시킵니다. 따라서 예외가 발생한 호출은 그 뒤에 연결된 어레이를
남기지 않습니다. 9대의 카메라로 구성된 어레이는 약 1분 만에 연결되며,
카메라 전원이 켜진 직후에는 몇 분이 걸릴 수도 있습니다. 가동 중이지만 아직
동기화된 프레임 그룹이 없는 어레이는 `connect_stream["state"]`
`no_frames` 코드와 함께 반환됩니다. 백엔드 로그에는 연결 단계마다 하나의 타이밍 행
(`[array-connect <id>] <step>: <s> s …`)이 기록되며, 가장 느린 단계를 명시하는 마지막 행이 있습니다.

`force_tier` 값:
- `"sim-capture-sim-emit"` — 진정한 동시성(모든 카메라가 동일한 클럭 에지에서 촬영).
- `"sim-capture-ftd-stagger"` — 유연한 시간 영역 스태거(캠들이 약간 오프셋된 시간에 신호를 방출하여 패킷이 배선상에서 직렬화됨).
- `"slip-emit-and-capture"` — 캠별 순차 캡처 (시간적 동기화 없음). 카메라의 속도를 충분히 늦춰 배선을 동시에 공유할 수 없는 경우에만 자동으로 선택됩니다.

`wire_ceiling_mbps`는 **호스트의 지속적 배선 할당량**(MB/s)을 재정의합니다. 이는 전체
어레이 할당이 의존하는 단일 수치입니다. 자동 감지된
값을 사용하려면 `None`로 두십시오. 어레이가 GVSP 손상 프레임을 보고할 때는 이 값을 낮추십시오: 자동 값은
NIC가 알리는 링크 속도에서 파생되는데, 이는 USB 어댑터, 대역폭이 좁은 PCIe 레인 및
부하가 높은 공유 패브릭의 속도를 과대평가하는 경향이 있습니다. 이러한 과대평가는
눈에 띄게 느린 링크 속도보다는 손상된 프레임 형태로 나타납니다. 이 값은 프로젝트의 어레이 캡처 블록에 저장되므로,
프로젝트를 다시 열거나 나중에 `connect_array`를 설정하면 다른 어레이 설정과 마찬가지로 이 값이 복원됩니다.
[어레이 상태](#array-health--which-subsystem-is-losing-frames)를 참조하십시오.

#### 압축된 12비트 전송(`packed=`)

센서가 최대 용량으로 작동할 때 어레이의 프레임 속도는 각 카메라가
프레임을 전송할 수 있는 속도에 따라 결정됩니다. 12비트 형식(`BayerRG12`, `Mono12`)는 일반적으로 16비트
워드 단위로 전송되지만, 압축 전송 방식은 동일한 12비트를 픽셀당 1.5바이트로 전송한 후
호스트에서 압축을 해제하므로, 각 프레임의 전송 시간이 25% 단축됩니다 — 약
**센서 최대 성능 시 초당 약**33% 더 많은 프레임**을 처리합니다(측정 결과: 9대 카메라 어레이에서 10.7 → 14.0 fps).**저장된 데이터는 동일합니다**: 동일한 값, 동일한 16비트
파일, 동일한 메타데이터입니다.

- `packed="auto"`(기본값)는 압축이 도움이 되고 **또한** 호스트가
  압축 해제 처리를 감당할 수 있을 때만 압축을 수행합니다. 연결 시 백엔드는 자체 압축 해제 속도,
  CPU 개수, 현재 부하 및 네트워크 대역폭을 측정합니다. 여유 코어가 있는 데스크톱은
  센서 전체 데이터를 압축하여 전송하고, 부하가 높거나 저전력 호스트(싱글 보드 컴퓨터,
  임베디드 컨트롤러)는 압축을 풀지 않은 상태로 스트리밍합니다. 작은 ROI와 비닝된 프레임의 경우 이점이 거의 없습니다.
- `packed="off"`(또는 `False`)는 절대 압축하지 않습니다. 진단 시 압축 기능을 배제해야 하거나,
  다른 작업을 위해 CPU 자원을 확보해야 하는 호스트에서 사용합니다.
- `packed="on"`(또는 `True`)는 12비트 형식을 항상 패킹합니다.
- 그 외의 모든 값은 `ValueError`를 발생시킵니다. 8비트 및 16비트 형식은 절대
  패킹되지 않습니다.

`analyze_array_network(..., packed="auto")`도 동일한 결정에 따라 처리되므로,
해당 fps는 배열이 실행될 속도를 나타냅니다(응답에는
`packed_transport` 및 `packed_reason`가 포함됨); 해당 `pixel_format`는 8비트가 아닌
connect가 적용하는 값(Mono12 / BayerRG12)을 기본값으로 사용합니다.

결정 사항과 그 이유는 세션에 기록됩니다:

```python
arr = chloros_sdk.connect_array(serials)          # packed="auto"
print(arr.packed_transport, arr.packed_reason)
# True  auto: projected 11.0 -> 14.4 fps, unpack 0.16 of 1.50 budgeted cores

arr = chloros_sdk.connect_array(serials, packed="off")
```

`ChlorosProject.connect_all(packed=...)`는 프로젝트 내의 모든
배열에 동일한 재정의 설정을 적용합니다(기본값은 `None`이며, 이는 `"auto"`를 의미합니다: 즉, 프로젝트는 배열별로
압축 설정을 저장하지 않습니다).
시스템 전체에 적용되는 기본값을 설정하려면 백엔드 환경에서 `CHLOROS_ARRAY_PACKED12=off`(또는 `on`)를 설정하십시오.
단, 명시적으로 설정된 `packed="on"` / `"off"`가 여전히 우선 적용됩니다.
`/api/camera/array/status`는 `packed_transport` 및 `packed_decision`를 보고합니다.

#### 예상 프레임 속도

측정된 어레이 프레임 속도(하드웨어 동기화, 12비트; 전체 센서 = 2048 × 1536). 이 값들을 사용하여 `status()["actual_fps"]`의 타당성을 확인하십시오. `analyze_array_network()`는 사용자의 호스트 및 네트워크에 대한 프레임 속도를 예측합니다.

| 설정 | 어레이 프레임 속도 |
| --- | --- |
| 전체 센서, PCIe 10 GbE 호스트 NIC, 압축 전송 (데스크톱의 `packed="auto"`) | ~14 fps |
| 전체 센서, PCIe 10 GbE 호스트 NIC, 언팩 (`packed="off"`) | ~10.5 fps |
| 전체 센서, USB 10 GbE 어댑터에 연결된 카메라 8대 | ~3.8 fps |
| 전체 센서, 1 GbE 호스트 링크 | 1.2–1.7 fps |
| 2 × 2 비닝 (`binning=2`), 10 GbE 호스트 NIC | ~30–33 fps |

- PCIe 10 GbE 링크에서 풀 센서 모드 시 성능의 한계는 네트워크가 아닌 **카메라** 수에 의해 결정됩니다. 4대와 8대의 카메라 모두 동일한 속도로 작동합니다.
- **`target_fps`는 트리거 속도이며, 어레이는 이보다 빠르게 작동하지 않습니다.** `target_fps=1`는 어레이를 1 fps로 구동하며, `None`(기본값)는 호스트와 네트워크가 처리할 수 있는 최고 속도로 구동합니다. 해당 속도를 초과하는 핀은 해당 속도로 낮아집니다 (자세한 내용은 [`set_target_fps`](#interval-capture-for-drone-surveys) 참조).
- **`link_speed_mbps`의 `4294`는 링크 속도가 아닙니다.** 이는 Windows가 어댑터의 속도를 읽을 수 없음을 의미합니다(드라이버가 최대 32비트 값을 보고함). 이는 USB 이더넷 어댑터에서 흔히 발생하는 현상입니다. Windows에서 USB 어댑터의 사용 가능한 속도는 어쨌든 200 MB/s로 제한됩니다([Array Health](#array-health--which-subsystem-is-losing-frames) 참조); 조정 방법은 [호스트 NIC 설정 및 튜닝](cli-reference.md#host-nic-setup--tuning-lattice-arrays)의 CLI 참조 문서에 있는 “USB 10GbE 어댑터” 섹션을 참조하십시오.
- **리눅스에서도 USB 속도 제한이 적용됩니다: x86_64에서는 400 MB/s, NVIDIA Jetson에서는 200 MB/s입니다.** 어댑터는 sysfs를 통해 식별되며(`ethtool -i`로 대체됨), 따라서 USB 5 GbE 또는 10 GbE 어댑터는 명판에 표기된 500 / ~1060 MB/s 속도가 아닌 상한선 속도로 처리됩니다. x86_64의 수치는 리눅스에서 측정된 것입니다. 윈도우에서 200 MB/s를 초과하면 프레임이 손상되던 동일한 어댑터가, 리눅스에서는 480 MB/s에 이르는 모든 상한선에서 4대의 카메라로 구성된 어레이를 오류 없이 처리했습니다. NVIDIA Jetson에서는 동일한 어댑터가 200 MB/s에서만 오류 없이 작동했으며, 그 이상의 속도에서는 간헐적으로 프레임이 손실되었습니다. 호스트가 `gvsp_corrupt` 프레임 없이 더 많은 데이터를 처리할 수 있다면, `wire_ceiling_mbps`를 사용하여 상한을 높여주세요. 리눅스에서 USB 2.0 포트에 연결된 어댑터의 대역폭은 30 MB/s로 제한되므로, USB 3.0 포트에 연결하세요.

#### 오버서브스크립션(카메라별 하한선)

어레이는 어떤 네트워크에서도 작동하지만, 프레임 속도는 네트워크 성능에 따라 달라집니다. Sim-emit 페이싱은 각 카메라에 충돌 방지 회선 대역폭을 균등하게 할당하며, 모든 카메라가 동일한 트리거에 노출되므로, 느린 회선에서는 할당량과 프레임 속도가 낮아집니다: 8개의 풀해상도 12비트 카메라 8대가 점보 프레임이 없는 1 GbE 환경에서 각각 6.5 MB/s, 약 **1 fps**(압축 시 약 1.3 fps)를 얻습니다. 할당량은 카메라가 페이싱될 수 있는 가장 느린 속도로 하한이 설정됩니다 (`per_cam_floor_bps`: 연결 시 각 카메라의 GevSCPD 범위에서 읽으며, 1 MB/s 이하로 떨어지지 않음). 네트워크 회선 상의 제약으로 인해 이보다 더 느린 속도가 필요한 경우, 연결은**직렬**전송(`slip-emit-and-capture`)으로 대체됩니다: 카메라들은 최대 링크 속도로 번갈아 가며 데이터를 전송하므로 모든 프레임이 도착하지만, 구성원들은**동시에 노출되지 않습니다**— 응답의 티어 및 차단기 (`SERIAL: ...`)에서 이를 명시합니다; `force_tier="sim-capture-sim-emit"`를 전달하여 대체 방식을 거부할 수 있습니다. 프레임 크기는 공유 계산에 포함되지 않으므로,**빈닝 및 ROI는 프레임 속도를 높이기는 하지만 어레이가 수용 가능한지 여부는 변경하지 않습니다**. 어떤 페이싱(동시 또는 교대)으로도 어레이를 전송할 수 없는 회선만이**과부하** 상태가 됩니다 — 오류 모드는 프레임 속도 저하가 아닌 GVSP 패킷 손실입니다. 해결책: 카메라 수 줄이기, 종단 간 점보 프레임 사용, 또는 더 빠른 NIC 사용.

- `analyze_array_network()` 및 `/api/camera/array/connect` 응답에는 `oversubscribed`, `aggregate_demand_bps`, `collision_safe_ceiling_bps`, `max_cams_collision_safe` 및 `per_cam_floor_bps`를 포함합니다. `oversubscribed`가 true인 경우, 프로젝션은 **fps 필드를 0으로 설정**합니다(`achievable_fps_max` / `fps_bright` / `fps_dark`)를**0으로 설정**하며, 오해의 소지가 있는 ‘느리지만 작동하는’ 속도를 보고하지 않습니다.
- `POST /api/camera/array/connect`는 `pin_resolution` 본문 매개변수를 허용합니다(**HTTP 전용 — SDK 키워드 인수가 아님**; `connect_array`는 이를 노출하지 않음). 이는 엄격한 모드입니다: `pin_resolution`가 설정된 상태에서 과부하된 연결 시, 모든 해결책을 명시한 오류 메시지와 함께**강제 거부**됩니다. 이 매개변수가 없으면, 배열에서 패킷 손실이 발생할 수 있다는 경고와 함께 연결이 진행됩니다. 어느 경우든 연결은 요청된 프레임 크기를 유지하며, 자체적으로 프레임을 병합하거나 잘라내지 않습니다.
- 테스트용으로만: 백엔드 환경에서 `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1`를 설정하여 거부 처리를 강력한 경고로 완화할 수 있습니다. 이 경우 연결은 진행되지만 패킷 손실을 감수해야 합니다.

#### 어레이 상태 — 어떤 하위 시스템에서 프레임이 손실되고 있는지

`ArraySession.capability()`(`GET /api/camera/array/<array_id>/capability`)는 연결된 어레이에서 활성 `health` 블록을 전달하며,
**10초** 간격으로 재평가됩니다. 기간 동안 재평가됩니다. 이 지표는 프레임 손실을
단 하나의 “불완전” 비율로 묶어 두 가지 원인을 모두 명시하지 않는 대신,
서로 반대되는 해결책이 필요한 두 가지 원인으로 구분합니다:

| 필드 | 의미 | 해당 하위 시스템 |
| --- | --- | --- |
| `gvsp_corrupt_rate_pct` (시리얼별) | 프레임이 **도착했으나 구조적으로 결함이 있음**— GVSP 패킷 손실. |**네트워크**: 와이어 예산, 페이싱, NIC RX 링, MTU |
| `never_arrived_rate_pct` (시리얼별) | 프레임이 **전혀 도착하지 않음**— 카메라가 작동하지 않았거나, 카메라에서 데이터가 전송되지 않았음. |**트리거 / 동기화**: M8 케이블, `line=`, `TriggerMode` |
| `worst_gvsp_corrupt_pct` / `worst_never_arrived_pct` | 카메라별 최악의 전송률 각 카메라별. | — |
| `per_cam_rate_pct` | 카메라당 통합 불완전률 (두 원인 합산). | — |
| `stable_for_seconds` | 각 카메라가 0.01% 미만 상태를 유지한 기간. | — |

`health`와 함께, 동일한 레코드에는 전체 할당량이 얼마나 남아 있는지도 보고됩니다:

| 필드 | 의미 |
| --- | --- |
| `wire_ceiling_mbps` | 호스트의 현재 유효한 지속적 유선 대역폭, MB/s. |
| `wire_ceiling_source` | 해당 수치의 출처(문구로 표기) — 예: `USB-capped 200 MB/s (was theoretical 1062; …)` 또는 `user override 120 MB/s (auto said 200)`. |
| `wire_ceiling_is_user_set` | `true`가 `wire_ceiling_mbps=`를 설정했을 때의 값. |
| `nic_is_usb` | USB 이더넷 어댑터의 경우 `true`. |

```python
import chloros_sdk

serials = ["213800234", "214000533"]           # master first
arr = chloros_sdk.attach_array(serials)
h = arr.capability()

health = h.get("health", {})
print("wire ceiling:", h["wire_ceiling_mbps"], "MB/s", h["wire_ceiling_source"])
print("corrupt (network) :", health.get("worst_gvsp_corrupt_pct"), "%")
print("absent  (trigger) :", health.get("worst_never_arrived_pct"), "%")

if (health.get("worst_gvsp_corrupt_pct") or 0) > 1.0:
    # Network path. Reconnect with a lower budget -- NOT a lower target_fps.
    # force_disconnect(), not disconnect(): this handle came from attach_array,
    # and disconnect() never tears down an array the session only attached to.
    # It returns only once the backend confirms the release; a teardown that
    # timed out or was refused raises ChlorosConnectError instead, so the
    # reconnect below never races a half-released array.
    arr.force_disconnect()
    arr = chloros_sdk.connect_array(serials, wire_ceiling_mbps=120)
```

**해석:** 0이 아닌 `gvsp_corrupt_rate_pct` 값과 0인 `never_arrived_rate_pct` 값은
트리거링과 케이블 동기화가 완벽하며, 손실의 100%가 네트워크 경로에서 발생함을 의미합니다 —
`wire_ceiling_mbps`를 낮추고 다시 연결하십시오. 반대의 패턴은 동기화 케이블이나
트리거 라인을 가리킵니다.

> **`target_fps`는 프레임 손상의 원인이 아닙니다.** GevSCPD 페이싱은
> 연결 시 한 번만 설정되므로, 트리거 속도를 낮추면 듀티 사이클만 변경될 뿐
> 동시 송신 버스트 속도는 변경되지 않습니다. 측정 결과, 수요를 5배 줄여도 개선 효과가 없었으나
> 와이어 상한선을 240 MB/s에서 200 MB/s로 낮추자 동일한 장비에서 오류율이 10.4%에서
> 0.00%로 떨어졌습니다.

> **실행 중인 어레이는 LATTICE(M3C / M3M) 펌웨어에서 자체적으로 재계획할 수 없습니다.** 이 문제를
> 자체적으로 해결할 수 없습니다. 연결을 끊었다가 다시 연결하여, 연결 시간 선택기가 새로운 상한값을 기준으로
> 재계획을 수행하도록 해야 합니다.

**USB 이더넷 어댑터는 명판 사양과 관계없이** 프로브에 의해 200 MB/s로 제한됩니다.
명판에 적힌 명칭과 관계없이 **USB 이더넷 어댑터의 전송 속도는 200 MB/s로 제한됩니다**. 링크 속도를 지속 전송 속도로 변환하는 효율성 표는
PCIe에서 파생된 것이며, USB NIC는 이더넷 링크 속도를 광고하지만
USB 버스 및 해당 드라이버에 의해 제한을 받습니다. 이 제한은 비율(fraction)이 아닌 절대값입니다. USB 1 GbE 어댑터는
약 80 MB/s를 도출하므로 영향을 받지 않습니다. 이 제한은 Windows와 Linux(Jetson 포함)에 적용되며,
측정 위치에 대해서는 [예상 프레임 속도](#expected-frame-rates)를 참조하십시오.

#### `ArraySession` 메서드

| 메서드 | 설명 |
| --- | --- |
| `status(timeout=10.0)` | 실행 중인 어레이의 실시간 상태: `target_fps` / `target_fps_auto` (트리거 속도 및 속도 컨트롤러가 이를 소유하고 있는지 여부), `target_fps_clamped_reason` (고정된 속도가 더 낮게 유지되는 이유, 그렇지 않은 경우 `None`), `actual_fps` (측정값; 그랩 루프가 포기하거나 정지하면 0), `recommended_fps`, `streaming` (그랩 루프가 중단되면 False가 됨 — `last_error`뿐만 아니라 이 값도 확인해야 함; 설정 재시작 시에도 True 상태를 유지하며, `restarting`가 이를 표시함), `stalled` (그랩 루프가 실행 중이지만 약 10 프레임 주기(2–30초) 동안 프레임 그룹을 게시하지 않은 경우 True; `last_error`는 60초가 지난 후에만 이를 표시함), `last_frame_age_s`, `last_error`, `serials`, `ptp_enabled` (`True`는 어레이가 PTP로 연결되었을 때만 **이며** 모든 구성원이 마지막으로 읽은 시점에 `PtpEnable`가 켜져 있는 상태; `ptp_epoch`가 측정하는 ‘잠김’ 상태와는 다름), `ptp_configured` (연결 요청 시 지정된 값), `ptp_off_members` / `ptp_unread_members` (마지막으로 `PtpEnable`를 'off'로 읽었거나 응답하지 않은 구성원), `member_recovery` (`None`, 또는 전원이 끊기거나 연결이 끊긴 구성원: 응답하지 않는 동안 `state`, `waiting`, `reopening`, 그 다음 `recovered`, `members` 및 `why` — 어레이는 해당 노드가 다시 응답하면 이를 재개하고 연결 시점의 PTP, 트리거 및 동기화 라인 배선, 페이싱 및 카메라 설정을 적용한 뒤, 새로운 `connect_array` 없이 다시 스트리밍을 시작하며, `ptp_epoch`(측정값), `sync_mode` / `sync_tier`, `packed_transport` / `packed_decision`, `per_cam_emit_mbps`, `link_speed_mbps` (`4294`는 Windows가 어댑터의 속도를 읽을 수 없음을 의미합니다 — [예상 프레임 속도](#expected-frame-rates) 참조), `trigger_pipeline` / `pipeline_tune`, `radiometric_chain` (대역별 복사도 체인이 사용 가능한지, 그리고 어떤 구성 요소가 이를 손상시켰는지), 그리고 `band_clamp` — 라이브 뷰가 어레이를 렌더링하는 동안: 언믹스(unmix)가 0으로 클램핑한 조명 픽셀의 비율(구성 요소별) (`members`: 카메라의 R, G, B 밴드 순서대로 `{serial: [band0, band1, band2]}`) 및 밴드별로 합성 이미지가 보여주는 (`composite`: `{band name: share}`), 여기서 `warn`는 GUI에서 “신뢰도 낮은 밴드 데이터”로 표시되는 비율 (약 1초에 한 번씩 측정됨; 라이브 뷰가 실행되지 않을 때는 `None`), 그리고 `member_ae` — 각 구성원별로 노출(`exposure_us`) 및 게인 (`gain_db`)를 해당 프레임 자체(`read_nodes` 또는 `ExposureTime` / `Gain`는 자동 노출이 실행 중일 때 기록된 마지막 값을 반환하며, 현재 수행 중인 값은 아닙니다). |
| `set_member_settings(serial, **kwargs)` | `CameraSession.set_settings()`가 설정값(노출, 게인, 목표 밝기 등)을 가져오면, 하나의 멤버 카메라 설정을 기록합니다 (노출, 게인, 목표 밝기 등); 해당 구성원은 배열에 남아 있습니다. 구성원이 아닌 시리얼에 대해서는 `ChlorosConnectError` 예외를 발생시킵니다. [배열 구성원에 대한 노출](#exposure-on-array-members)을 참조하십시오. |
| `capability(timeout=10.0)` | 스마트 프리프(smart-prep)가 결정한 사항(캡처 계층, 프레임 크기, `pixel_format`), 라이브 `health` 블록 및 호스트의 와이어 예산. [어레이 상태](#array-health--which-subsystem-is-losing-frames)를 참조하십시오. |
| `set_target_fps(fps, timeout=10.0)` | 재연결 없이 어레이가 실행 중인 동안 트리거 속도를 변경합니다. 숫자를 입력하면 해당 값이 고정됩니다(속도 컨트롤러는 이를 변경하지 않음); `None`는 이를 자동 모드로 되돌립니다. `{requested, target_fps, target_fps_auto, clamped}`를 반환하며, 속도가 호스트와 네트워크가 처리할 수 있는 수준으로 낮아졌을 때 `clamped`는 True입니다. [드론 측량을 위한 간격 캡처](#interval-capture-for-drone-surveys)를 참조하십시오. |
| `grab(processing="debayered", with_metadata=False, apply_calibration=True, apply_white_balance=True, apply_index=False, index_expression=None, bit_depth=None, band=None, align=False, refl_max=None)` | 모든 카메라에서 한 프레임씩, **모두 하나의 트리거로**, numpy 배열 형태로 반환합니다(파일 기록 없음). `GrabResult`(또는 `dict`): `{serial: array}`, 또는 `{serial: (array, metadata)}`와 `with_metadata=True`를 반환합니다. `.group_id`는 프레임들이 공유하는 트리거이며, `.errors`는 제외된 각 카메라를 그 사유에 매핑합니다(해당 트리거에서 처리할 수 없는 카메라는 생략되며, 다른 카메라의 데이터로 채워지지 않습니다). `processing`는 `raw` / `debayered` / `radiance` / `reflectance` / `display`입니다. 이는 프로젝트 API의 [`ArrayHandle.grab()`](#one-trigger-per-read--group_id) — 하나의 공유 구현체 — 와 동일하게 작동하므로, 해당 키워드(`band`, `align`, `bit_depth`, `refl_max`)는 여기서도 작동합니다. `numpy`가 필요합니다. |
| `frame_stream(*, processing="debayered", fps=5.0, duration=None, count=None, apply_calibration=True, apply_white_balance=True, apply_index=False, index_expression=None, with_metadata=False, bit_depth=None, band=None, align=False, refl_max=None)` | 틱당 하나의 `grab()` 결과를 반환하는 이터레이터(키워드 인자 전용), `fps`에 맞춰 속도 조절되며, `duration`(초) 및/또는 `count`(`None` = 제한 없음)로 제한됩니다. **반환된 각 집합은 새로운 트리거가 됩니다:** 가장 최근 트리거가 이미 반환된 집합인 틱은 해당 트리거를 반복하지 않고 다음 트리거를 기다리므로, 스트림은 `fps`와 배열 자체의 프레임 레이트 중 더 느린 속도로 실행됩니다. 침묵 상태로 전환되지 않고 `FrameFetchError`를 발생시킵니다: 모든 카메라가 다음 틱이 반복될 만한 이유로 거부할 때, 프레임이 없는 틱이 60초 동안 지속될 때, 그리고 새로운 트리거가 60초 동안 없을 때("60초 동안 새로운 트리거 없음") 즉시 발생합니다. `ArrayHandle.frame_stream()`와 동일한 구현 방식입니다. [자신만의 파이프라인에서 라이브 프레임 사용하기](#live-frames-in-your-own-pipeline)를 참조하십시오. |
| `continuous(output_dir="output", *, format="tiff", processing="debayered", levels=None, aligned=None, render_index=None, force_daq=None, orient=None, max_captures=None, duration_s=None, daq=None, daq_tolerance_ms=None, timeout=30.0)` | 백엔드 자체의 연속 캡처(GUI의 연속 모드) 시작 → [`ContinuousHandle`](#continuoushandle). 배열이 발행하는 **모든**동기화된 그룹을 `capture()`가 저장하는 방식(동일한 내보내기 제어)으로 저장하며, 중지되거나 `max_captures` / `duration_s` (`None` = 중지될 때까지)가 호출될 때까지**모든** 동기화된 캡처 그룹을 저장합니다. 해당 어레이에 대해 이미 실행 중인 프로세스가 있는 경우 `ChlorosConnectError` 예외를 발생시킵니다. |
| `capture(output_dir="output", format="tiff", processing="debayered", levels=None, aligned=None, render_index=None, force_daq=None, orient=None, smart=False, after_seq=None, daq=None, daq_tolerance_ms=None, timeout=300.0)` | 동기화된 캡처 그룹 하나. `CaptureResult` (프레임 딕셔너리 목록 + `.skipped` + `.failed` + `.group_seq` + `.daq_files` + `.daq_matches` + `.alignment` + `.oriented`). 이전 결과인`group_seq`를 `after_seq`로 전달하여, 새로운 그룹을 기다리는 대신 그 다음에 오는 동기화된 그룹을 얻습니다. 아래의 내보내기 제어 항목. |
| `capture(..., smart=True)` | **스마트 캡처** — 모든 카메라에서 AE가 안정화될 때까지 기다린 후 트리거합니다. |
| `capture_fastest(output_dir="output", force_daq=True, render_index=True, orient=None, timeout=120.0)` | 가장 빠른 캡처: 원시 데이터 전용 + 할당된 DAQ 판독값 (+ 자유 결합 인덱스). GUI의 "가장 빠른 캡처" 버튼과 동일합니다. |
| `capture_repeated(output_dir="output", count=None, duration_s=None, interval_s=0.0, on_capture=None, **capture_kwargs)` | 단일 / 연속 / 간격 모드를 하나의 제한된 루프에서 실행합니다. `list[CaptureResult]`를 반환합니다.**종료하려면 `count` 및/또는 `duration_s`가 필요합니다**(SDK에는 Ctrl+C 기능이 없음). 또한, 패스가 완료된 후, 패스가 드라이브가 가득 찼음을 보고할 때(`disk_full`가 포함된 `.failed` 항목) 조기 종료되며, 해당 패스가 마지막 결과입니다. |
| `record(output_dir="output", fps=10.0, duration_s=None, video=True, gif=False, expr=None, timeout=30.0)` | 라이브 결합 인덱스 뷰를 video/GIF로 녹화 시작 → `RecorderHandle`. 동영상은 Motion-JPEG 형식이며 `.avi` (ffmpeg 불필요) 형식입니다. GUI 뷰가 열려 있지 않은 경우 백엔드에서 이를 렌더링합니다: `expr` (배열의 밴드 이름으로 구성된 인덱스 표현식, 예: `"(NIR_850_RGN-Red_660_RGN)/(NIR_850_RGN+Red_660_RGN)"`) 또는 기본적으로 배열의 NDVI. 배열당 하나의 합성 레코더. |
| `burst(output_dir="output", duration_s=None, max_frames=None, index_config=None, serial_index_config=None, orient=None, timeout=30.0)` | 고프레임률 원시 베이어(raw-Bayer) 연사 시작 → `RecorderHandle`. `build_video()`를 사용하여 오프라인에서 재처리합니다. `orient=True`: `build_video()`가 생성하는 결과물은 카메라 방향에 따라 회전됩니다(각 카메라별 결과물은 해당 카메라의 방향에 따라, 통합 인덱스는 마스터 카메라의 방향에 따라). 원본 프레임은 원본 상태를 유지하며, 버스트 매니페스트에는 각 구성원의 방향이 기록됩니다. |
| `build_video(burst_dir, products=None, fps=10.0, video=True, gif=False, save_tiffs=False, wait=True, poll_s=2.0, timeout=1800.0)` | 저장된 원본 버스트를 보정된 비디오로 오프라인 재처리합니다. 작업이 완료될 때까지 대기(`wait=True`)한 후 빌드 결과를 반환합니다: `{success, outputs, errors, skipped, substitutions, notes, degraded, combined}` — 아래 [빌드 결과](#the-build_video-result) 참조. |
| `build_video_status(job_id, timeout=15.0)` | 오프라인 빌드 작업(`{running, result, error, burst_dir}`)을 폴링합니다. |
| `disconnect(keep_settings=False, clear_settings=False)` | 전체 어레이를 해제합니다 — **이 세션에서 어레이를 열었다면** (`connect_array`): 이 경우 `force_disconnect()`가 되며, 릴리스가 확인되지 않을 때 `ChlorosConnectError`를 발생시킵니다 (`with` 블록의 종료 시에는 예외가 발생하지 않음). `attach_array`에서 반환된 핸들을 사용할 경우, 어레이는 계속 실행되며 이를 알리는 한 줄의 메시지가 stderr에 출력됩니다: 해당 어레이는 GUI나 다른 스크립트에 속할 수 있습니다. 각 카메라는 자유롭게 작동하도록 남겨지며, 연결 시 발견된 상태 그대로 유지됩니다. `keep_settings=True`는 설정을 카메라 자체 메모리에 저장하고, `clear_settings=True`는 해당 메모리를 공장 출하 시 설정으로 재설정합니다([카메라 메모리](#camera-memory--keep_settings--clear_settings) 참조). |
| `force_disconnect(timeout=60.0, keep_settings=False, clear_settings=False)` | 누가 열었는지와 관계없이 전체 어레이를 해제합니다. 어레이를 종료하려는 경우(예: 다른 설정으로 재연결하기 위해) 연결된 핸들에 이 명령을 사용하십시오. 백엔드가 해제를 확인하거나 해제할 대상이 없을 때(배열이 이미 사라졌거나 실행 중인 백엔드가 없는 경우) 반환됩니다. 타임아웃이 발생했거나 백엔드가 거부한 해체 요청의 경우 `ChlorosConnectError`를 발생시킵니다. 두 플래그 중 하나를 지정하면 각 카메라의 메모리가 (`camera_memory`), 그렇지 않으면 `None`를 반환합니다. |

`capture()` 내보내기 제어(GUI/CLI에서 사용하는 것과 동일한 엔드포인트):

- `processing` / `levels` — `processing="all"`(또는 `levels=["raw","radiance",…]`)는 카메라별로 적용 가능한 모든 내보내기 유형을 저장하며; 단일 `processing` 값은 해당 레벨만 저장합니다.
- `aligned` — **기본적으로 활성화됨(`None`), 이는 Chloros 앱이 동일한 어레이를 캡처할 때**(배열별**정렬** 캡처 설정)로 설정되어 있기 때문입니다. 모든 멤버의 비-원시(non-raw) 내보내기 데이터는 배열의 [정렬 프로필](#array-alignment)에 따라 마스터 카메라의 프레임으로 변환되고, 구성원들의 공통 중첩 영역으로 잘려 나옵니다. 따라서 파일들은 픽셀 단위로 정렬되며, 앱에서 생성하는 것과 동일한 크기, 자르기 범위 및 `Alignment*` 태그를 갖게 됩니다(ROI 내에서는 변환이 창을 기준으로 재조정됨); `XMP-mapir:AlignmentJSON`는 `"applied": true`에 대해 `crop_roi`를 지정합니다. Raw 파일은 원본 크기로 왜곡되지 않은 상태로 유지되지만 메타데이터에 변환 정보를 포함합니다. `aligned=False`는 모든 파일을 원본 크기로 왜곡 없이 저장하며, `aligned=True`는 이를 명시한 기본 설정입니다. `continuous()`, `capture(smart=True)`, `capture_repeated()` 및 프로젝트 수준의 `ArrayHandle.capture()` / `capture_smart()`는 동일한 기본값을 따릅니다. 워프가 적용되었는지 여부와 관계없이, 배열에 정렬 프로필이 있으면 각 구성 파일은 해당 카메라의 `Alignment*` 변환을 기록합니다. 단, 프로필이 정렬되지 않은 카메라 (제외되었거나, 해결 실패로 인해 신분 변환이 강제된 경우) 및 다른 카메라가 정렬되지 않은 경우의 기준 카메라는 예외입니다. 이러한 카메라에는 `Alignment*` 태그가 포함되지 않습니다. 어레이에 프로필이 없는 경우 캡처는 정렬되지 않은 상태로 저장되며, 결과에도 그 사실이 표시됩니다: `result.alignment`는 `{"requested": True, "applied": False, "warnings": [...]}`이며, 워프가 적용되지 못한 이유(프로필 없음, 구성 요소의 해상도 차이, 어레이가 동기화되지 않음동기화되지 않음, 프로파일을 매핑할 수 없는 프레임 등)에 대해 각각 하나의 경고가 표시됩니다. `applied`는 `True`가 되며, 이는 워프를 요청한 모든 내보내기 작업이 워프를 적용받은 경우에만 해당합니다. 어레이의 자동 정렬 기능은 노출이 안정되면 프로필을 생성하며, `ArrayHandle.calibrate_alignment()` / CLI의 `align-calibrate`는 요청 시 프로필을 생성합니다. `result.oriented`는 `orient=True`가 산출물을 반환했는지 여부를 나타냅니다.
- `render_index=False` — 카메라별 식생 지수 오버레이를 건너뜁니다. 기본값은 구성된 위치에 렌더링합니다.
- `daq` / `daq_tolerance_ms` — DAQ 측정값은 선택된 레벨에서 필요하지 않은 경우에도 기본적으로 `.daq` 사이드카로 저장됩니다. `daq=False`는 이 경우 이를 건너뛰며, `daq_tolerance_ms`는 일치 허용 오차를 설정합니다(기본값 500). `force_daq=True`는 기본값의 이전 표기법입니다. [캡처가 저장하는 DAQ 측정값](#the-daq-reading-a-capture-saves--daq_files-and-daq_matches)를 참조하십시오.
- `orient=True` — 각 구성원의 **곱**(`display`, `radiance`, `reflectance`, 카메라별 인덱스 및 합성 이미지)를 정렬 워프 후 카메라 방향에 따라 변환합니다: 해당 멤버들이**마스터**의 좌표계로 워프되었을 때의 마스터 카메라 방향(`aligned`의 기본값 — 멤버들은 정렬 상태를 유지함)을 기준으로 하거나, 그렇지 않은 경우 각 카메라의 방향을 기준으로 합니다.**기본값: 모든 구성 파일은 원본 형식**이며, 이는 배열 캡처의 경우 항상 그렇습니다. `raw`와 `debayered`는 어느 경우든 원본 형식을 유지합니다. 각 파일은 해당 픽셀이 속한 프레임의 방향을 기록합니다 (`XMP-Camera:ChlorosRotation` / `ChlorosFlip*` / `ChlorosOrientationApplied` — 원본 파일은 해당 카메라의 프레임), 그리고 `Alignment*` XMP 파일은 방향 정보 적용 전에 적용된 네이티브 프레임 변환을 유지합니다; 해당 변환을 포함하는 RAW 파일은 `ChlorosAligned*`(마스터 프레임의 방향)도 기록하며, 이는 마스터 프레임으로 워핑된 결과물의 방향을 나타냅니다. 처리 과정에서는 각 결과물을 해당 픽셀에 속하는 방향만큼 회전시킵니다. 즉, RAW 파일을 워핑할 때는 정렬된 방향(배열 정렬 활성화)을, 정렬되지 않은 경우 카메라 자체의 회전 각도를 적용합니다. `capture_repeated()`와 `capture_fastest()`는 이를 그대로 통과시키며, 프로젝트 수준의 `ArrayHandle.capture()` / `capture_smart()`도 이를 그대로 적용합니다.

**TIFF 압축 (HTTP 전용 설정):** `ArraySession.capture()`는 `compression` 키를 전송하지 않으므로, 따라서 백엔드 기본값이 적용됩니다 — `POST /api/camera/array/capture`는 `compression` 본문 매개변수를 읽으며, `"none"` 기본적으로: 압축되지 않은 상태로, 센서에서 읽은 픽셀을 그대로 반영합니다(풀 해상도 프레임당 약 6.3 MB). `"deflate"`(무손실 zlib L1 + 수평 예측기, 프레임당 약 4.1 MB)는 파일 크기는 더 작지만 쓰기 속도는 더 느립니다. 두 방식 모두 무손실이며, 가져올 때 동일하게 읽힙니다. SDK는 이에 대한 kwarg를 제공하지 않으며, 우회 방법은 `chloros-cli lattice array-capture --compression deflate` 또는 원시 HTTP입니다. DEFLATE는 Python GIL을 점유하므로, 압축된 쓰기 작업은 카메라별 라이터 스레드 간에 병렬화되지 않습니다. 센서 속도로 풀 해상도 캡처를 지속적으로 수행하려면 압축되지 않은 기본 설정을 사용해야 합니다. 세부 정보: [CLI 참조 → array-capture](cli-reference.md).**연속 그룹(HTTP):** 엔드포인트는 `group_seq`를 반환합니다. 다음 요청 시 이를 `after_seq`로 다시 전송하는 루프를 사용하면, 이전 캡처가 저장되는 동안 게시된 그룹을 포함하여 다음 동기화된 그룹을 받을 수 있습니다 — 요청 이후에 게시된 그룹을 기다리는 대신, `skipped_groups`는 루프가 놓친 그룹을 보고합니다. `capture_repeated`, CLI의 `--continuous`와 GUI의 연속 모드 모두 이 방식을 사용하며, 간격 기반 캡처 결과는 예정된 시점마다 최신 상태로 유지됩니다.**멤버별 내보내기 재정의 (HTTP 전용):**동일한 엔드포인트는 `exclude_serials`(목록 — 저장된 세트에서 멤버 제거; 배열은 여전히 하나의 동기화된 그룹으로 트리거되며, 제외된 멤버는 `excluded`에 반환됨), `serial_levels` (`{serial: [level tokens]}` 카메라별 수준 재정의), `serial_index` (`{serial: bool}` 카메라별 인덱스 오버레이 재정의)를 수락합니다. 이들은 GUI와 동등한 본문 매개변수이며**아직 SDK kwargs는 아닙니다**. 맵에 없는 멤버는 배열 전체에 적용되는 `levels` / `render_index`로 대체됩니다.

##### 건너뛴 캠 검사 — `CaptureResult.skipped`

`ArraySession.capture()`는 `CaptureResult`를 반환하며, 이는 `list`의 서브클래스입니다: 이를 반복 처리하고, 인덱싱하고, `len()` 처리 — 기존의 모든 패턴은 계속 정상 작동합니다. 새로운 코드를 통해 `.skipped` 속성을 검사하여 어떤 캠이 제외되었는지 및 그 이유를 확인할 수 있습니다. 가장 흔한 경우는 혼합 필터 어레이 내의 RGB 캠으로, `processing="radiance"` 또는 `"reflectance"`를 요청할 때입니다. — 광대역 센서에서는 베이어(Bayer) 단위 복사도가 의미가 없으므로, 백엔드는 무의미한 결과를 생성하기보다는 해당 카메라를 건너뜁니다.

```python
with chloros_sdk.connect_array(serials) as arr:
    result = arr.capture("output/", processing="reflectance")

    # Back-compat: iterate as a plain list
    for frame in result:
        print(frame["filepath"], frame["serial"])

    # New: see why N-1 cams were saved
    for skip in result.skipped:
        print(f"skipped SN:{skip['serial']} reason={skip['reason']}")
        # e.g. {'serial': '214701292', 'level': 'reflectance',
        #       'reason': 'reflectance-not-applicable-to-rgb-cam',
        #       'filter': 'RGB'}
```

이유 토큰은 `<level>-not-applicable-to-rgb-cam` 패턴을 따릅니다(건너뛴 레벨마다 하나의 항목이 있으며, 각 항목은 `level`를 포함합니다). 보정 팩이 로드되지 않은 카메라는 `radiance` / `reflectance`를 `<level>-needs-calibration (connect the camera so its cal pack loads)`로 건너뛰며, 사용 가능한 DAQ 반사율 측정값이 없는 경우 `reflectance-skipped-no-dls`로 건너뜁니다. 그 외 반사율 관련 건너뛰기 항목으로는 `reflectance-skipped-no-fresh-dls`(새로운 하향 복사 측정값 없음), `reflectance-skipped-bound-daq-unavailable (…)` (바인딩된 DAQ에 접근할 수 없음), 그리고 `dls-uncalibrated-band-<nm>` — 해당 대역이 대부분 DAQ 광센서의 방사계 보정 범위(~374–974 nm)에 속하므로, DAQ 기반의 절대 반사율 분할이 거부되고 프레임은 센서 응답 방식으로 강제로 전환됩니다. 출하 중인 SKU 중에서는 F988만이 이 현상을 유발하며, 해당 카메라의 지원 경로는 반사율 패널 워크플로우입니다.

##### 쓰기 실패한 파일 — `CaptureResult.failed`

일부 파일만 쓸 수 있었던 캡처에서도 다음과 같은 결과가 반환됩니다: 이 목록에는 **작성된** 모든 파일이 포함되어 있으며, `.failed`는 나머지 파일을 나열합니다. 각 파일당 하나의 딕셔너리 형태로, `serial` (합성 이미지와 배열의 사이드카를 결합한 `None`의 경우 `.daq`), `level` (`raw`, `radiance`, …, `index`, `composite`, `daq`, 또는 `match` — `.daq`의 `.match.json` 출처(`.daq`는 기록되었으나 해당 사이드카는 기록되지 않은 경우) 및 `error`. 드라이브가 가득 찬 경우 해당 항목에는 `disk_full: True`도 포함됩니다. 해당 드라이브에 대한 이후의 모든 캡처 시도는 동일한 방식으로 실패하게 되므로, `capture_repeated()`는 그 지점에서 중단됩니다. 부분 캡처의 경우 예외가 발생하지 않으며, `.failed`에 있는 파일은 디스크에 반쯤 쓰인 상태로 남아 있지 않습니다. 정상 캡처 시 `.failed`는 비어 있습니다.

```python
result = arr.capture("output/", processing="all")
for f in result.failed:
    print(f"SN:{f['serial']} {f['level']} not written: {f['error']}")
if any(f.get("disk_full") for f in result.failed):
    raise SystemExit("drive full -- free space and capture again")
```

##### 캡처를 읽는 DAQ가 저장하는 파일 — `.daq_files` 및 `.daq_matches`

DAQ가 카메라나 어레이에 바인딩되거나 풀에 등록될 때마다, 모든 캡처는 사용된 하향 측정값을 다음과 같이 저장합니다: 캡처 폴더의 `daq/` 하위 폴더에 `.daq` 파일이 저장되며, 여기에는 해당 `.match.json` 및 `.cameras.json` (해당 측정값의 출처) 파일도 함께 저장됩니다. 이는 원시(raw) 또는 디베이어링(debayered) 캡처에도 적용되며, 캡처가 GUI, CLI 또는 SDK를 통해 수행되었는지와 관계없이 동일한 파일이 생성됩니다. `daq=False`는 레벨에 DAQ (`radiance`, `reflectance` 또는 `display` 레벨은 여전히 이를 기록합니다); `daq_tolerance_ms`는 해당 캡처에 대한 허용 오차선을 이동시키며, 사용된 측정값은 절대 이동시키지 않습니다. 어떤 DAQ를 사용할지: 카메라나 어레이에 바인딩된 DAQ(GUI의 광센서 선택, 프로젝트의 바인딩 또는 `chloros-cli daq pool-bind`)인지, 아니면 백엔드가 보유한 가장 최신 DAQ인지입니다. CLI에서는 제어 명령을 `--no-daq` 및 `--daq-tolerance-ms`로 표기합니다([CLI 참조](cli-reference.md#the-daq-reading-a-capture-saves)).

```python
r = cam.capture("out", levels=["raw"])      # a CameraSession; ArraySession results match
for m in r.daq_matches:                      # one per .daq in r.daq_files
    print(m["daq_file"], m["gap_ms"], m["in_tolerance"])
```

캡처는 사용된 광센서 측정값을 `.daq`로 저장하며, 그 옆에는 측정값이 노출과 어떻게 연결되었는지를 나타내는 `<file>.daq.match.json`가 함께 기록됩니다: `method`, `gap_ms`(노출에서 가장 가까운 DAQ 판독값까지) 및 `in_tolerance`(500ms 이내, 또는 캡처의 `daq_tolerance_ms`). 반사율 파일의 XMP에는 `Chloros:DlsMatchMethod`, `Chloros:DlsMatchGapS` 및 `Chloros:DlsInTolerance`와 동일한 결과가 포함되어 있습니다.

캡처는 노출 후 DAQ의 첫 번째 판독값을 기다리므로, 그 전후의 판독값을 혼합할 수 있습니다(`method` `interpolated`). DAQ-E는 일광 하에서는 0.4초마다, 실내에서는 1~2초마다 측정값을 전송하므로, `capture()`는 해당 간격의 최대 약 1.5배가 지난 후에 반환됩니다. 노출 자체는 지연되지 않습니다. 전송을 중단한 DAQ는 대기하지 않습니다. DAQ-E 측정값은 이전 측정값 이후의 간격 동안 평균화된 광량이므로, 해당 간격의 중간 지점에 배치됩니다: `gap_ms`는 그 중간 지점을 기준으로 측정되며, `.match.json`'의 `bracket_stamps_ns`는 측정값 자체의 타임스탬프를 제공합니다. `ArraySession.continuous()`는 캡처 속도를 늦추지 않고 대기합니다. `capture_repeated()`의-뒤로 통과 시 대기하지 않으므로 속도를 유지하며, 노출 전 가장 가까운 측정값과 매칭됩니다.

측정값은 만료되지 않으므로, 늦은 측정값으로 인해 프레임이 손실되는 일은 없습니다. 대신 다음과 같이 표시됩니다:

- **DAQ 스트림이 정지됨** — 가장 최신 측정값이 사용되며, `gap_ms`는 해당 측정값의 경과 시간을 표시합니다(`in_tolerance: false`: 500 ms 초과).
- **읽을 DAQ 없음**— 연결이 끊어졌거나(`disconnect()`, `chloros-cli daq pool-disconnect`), 바인딩된 DAQ에 접근할 수 없는 경우 — 카메라의 마지막 측정값이 재사용되며**이** 노출에 대해 기술됩니다: 이 노출에서 가져온 `method`, `retained`, `gap_ms`, `in_tolerance`는 재계산되며, `retained_from_epoch_ns`는 처음 매칭된 노출의 이름을 나타냅니다. 측정값이 한 번도 없었던 카메라의 경우 여전히 반사율이 없으며, 결과에는 그 이유가 명시됩니다 (`reflectance-skipped-no-fresh-dls` / `reflectance-skipped-bound-daq-unavailable (…)`, 또는 캡처 거부).

Chloros에서 열려 있는 프로젝트 폴더 내에 캡처가 저장하는 파일은, 앱 내 캡처와 마찬가지로 해당 프로젝트의 파일 목록에 추가됩니다(다른 곳에 저장된 파일은 그대로 유지됨): `CameraSession.capture()`, `ArraySession.capture()` (`smart=True`도 마찬가지) 그리고 `capture_cameras_transient()`. 결과는 `.project_add`에 명시되어 있습니다(`capture_cameras_transient()`: 사전의 `project_add`) -- `held`, `files` 및 백엔드의 `message`; `held`는 `True`이며, 처리 실행이 진행 중일 때 생성되고, 처리가 종료되면 파일들이 병합됩니다. 프로젝트에 파일이 도착하지 않은 경우 `None`가 생성됩니다.

`CameraSession.capture()` 및 `ArraySession.capture()`는 결과인 `.daq_files` 및 `.daq_matches`로 반환되며, HTTP 캡처 응답(`/api/camera/<serial>/cli-capture`, `/api/camera/capture`, `/api/camera/capture-all-synced`, `/api/camera/array/capture`) 에는 `daq_files`와 `daq_matches`가 포함되어 있으며, `.daq`당 하나씩: `daq_file`, `cameras`, `method`, `gap_ms`, `in_tolerance`, `tolerance_ms` 및 `stream_fault` — DAQ 자체의 스트림 오류로, 허용 오차를 벗어난 판독값에 대해서만 조회됩니다. 캡처에서 얻은 반사율을 신뢰하기 전에, 해당 `.match.json` 또는 XMP에서 `in_tolerance`를 확인하거나, DAQ의 [`status()`](#daqsensorsession-methods)를 확인하십시오.

`processing` 레벨:

| 레벨 | 출력 |
| --- | --- |
| `"raw"` | 센서에서 직접 출력되는 단일 채널 베이어(모노 카메라: 단일 대역). |
| `"debayered"` *(SDK 기본값)* | 3채널 컬러: 원본 데이터에서 보정 팩의 다크 오프셋(DSNU)을 제거한 후, OpenCV의 에지 인식 디모자이크 처리 — 선형성 LUT, 플랫 필드 또는 언믹스 없음 (모노 카메라: 1채널 그레이스케일). 저장된 TIFF 파일은 **RGB**채널 순서(광도계 RGB이므로 모든 이미지 뷰어에서 실제 색상을 표시함)로 저장되며, `grab()` / `frame_stream()`가 반환하는 메모리 내 배열은 OpenCV의**BGR** 순서로 반환됩니다 — 저장된 파일과 비교하려면 채널 순서를 반대로 바꿔야 합니다(`frame[..., ::-1]`). `chloros-cli process`는 `Debayered_Images`와 동일한 결과를 출력합니다. |
| `"radiance"` | 전체 방사측정 체인을 통해 산출된 float32 W/m²/sr/nm. 다중 스펙트럼 전용 — RGB 카메라는 건너뜁니다. |
| `"reflectance"` | uint16 0..32768 (Pix4D 호환); 절대 기준값을 얻으려면 실시간 DAQ 페어링이 필요합니다. 다중 스펙트럼 전용. |
| `"display"` | GUI 미리보기와 일치하는 전체 체인(카메라 프로필에 따른 CCM + WB + 감마). |
| `"all"` | **각 카메라에 대해 적용 가능한 레벨**당 하나의 파일(GUI의 “모두 캡처(Capture All)” / CLI 기본값과 일치). 반환된 `CaptureResult`는 `(cam, level)`당 하나의 프레임 딕셔너리를 포함하며, 각 딕셔너리에는 레벨이 포함됩니다. 적용되지 않는 레벨은 `.skipped`에 나타납니다. 캡처를 읽은 DAQ는 `.daq` 사이드카로 저장됩니다. |

> **참고 — 기본값은 CLI와 다릅니다.** `ArraySession.capture()`의 기본값은 `processing="debayered"`이며, `chloros-cli lattice array-capture` 명령어의 기본값은 `processing="all"`입니다. SDK에서 `processing="all"`를 명시적으로 전달하여 CLI/GUI의 다단계 저장 방식을 반영하십시오.

##### 배열 캡처 파일 이름

각 레벨은 `output_dir`의 개별 하위 폴더(`raw/`, `debayered/`, `radiance/`, `reflectance/`, `preview/`는 `display`, `index/`, `composite/`를 가리킵니다). 각 파일명은 카메라를 `SN<serial>-<filter>` 형식으로 지정됩니다. 즉, 일련번호 뒤에 카메라 모델의 필터 부분이 이어집니다. 예를 들어, `M3C-L87-FOCN`의 경우 `SN214000533-FOCN`, `SN261902142-F450`는 `M3M-L87-F450`를, `-`는 따라서 파일 이름은 이전과 마찬가지로 `_`로 구분된 필드를 갖습니다. 모델명을 읽을 수 없는 카메라는 `SN<serial>`로만 표시되며, 이전 릴리스에서 캡처된 파일의 이름에는 필터가 포함되지 않습니다. 파일 이름은 회원이 요청한 수준 수에 따라 달라집니다:

| 요청된 멤버 수 | 파일 이름 | 예시 (`reflectance`, 기본값 `format="tiff"`) |
| --- | --- | --- |
| **한 단계** (`processing="reflectance"`, 또는 항목이 하나인 `levels`) | `sync_<timestamp>_SN<serial>-<filter>.<format>` — 이전 릴리스와 마찬가지로 레벨 접미사가 없으므로 기존 스크립트는 계속 작동합니다 | `reflectance/sync_<ts>_SN214000533-FOCN.tiff` |
| **여러 레벨** (`processing="all"`, 또는 항목이 두 개 이상인 `levels`) | `sync_<timestamp>_SN<serial>-<filter>_<level>.<format>` | `reflectance/sync_<ts>_SN214000533-FOCN_reflectance.tiff` |

[클램프 마스크](#unmix-clamp-mask--which-pixels-a-band-was-clamped-on)는 제품명 뒤에 이어지며, `clamp_masks/<product folder>/<product name>_clampmask.tif`: 단일레벨 반사율 캡처는 `clamp_masks/reflectance/sync_<ts>_SN<serial>-<filter>_clampmask.tif`를 기록하고, `processing="all"`는 `clamp_masks/reflectance/sync_<ts>_SN<serial>-<filter>_reflectance_clampmask.tif`를 기록합니다. 접미사 규칙은 요청된 레벨에 따라 각 구성원별로 결정되므로, 건너뛴 레벨(예: RGB 카메라의 반사율)은 이를 변경하지 않습니다. 경로를 직접 생성하기보다는 반환된 프레임에서 경로를 읽어오십시오. 단일 카메라 캡처 (`CameraSession.capture()`)의 경우 항상 `capture_<ts>_SN<serial>-<filter>_<level>.tiff` 레벨이 추가됩니다.

##### PNG/JPEG는 디스플레이 및 인덱스 제품만 지원합니다

PNG 또는 JPEG는 채널당 8비트를 저장합니다. 해당 형식이 지원할 수 없는 레벨을 요청할 경우, 캡처가 시작되기 전에 요청이 거부되며, 해당 레벨의 이름과 이를 지원하는 형식인 TIFF를 명시하는 메시지가 표시됩니다:

- **`radiance`**(float32) 및**`reflectance`**(uint16, `32768` = 1.0) — 그리고 이 두 값을 모두 포함하는 `processing="all"`. SDK는 인수를 통해 이를 확인합니다: `capture()`는 요청 없이**`ValueError`**를 요청 없이 호출하면 (`CameraSession`, `ArraySession`, `CameraHandle`, `ArrayHandle`, `capture_smart`)가 없는 경우**`ValueError`**를 발생시킵니다.
- **8비트 이상으로 스트리밍하는 카메라에서 발생하는 `raw` / `debayered`** (12비트 M3C, Mono12 M3M) — 백엔드는 각 카메라의 비트 심도를 파악하고 있으므로 이를 거부합니다: `error_code` 및 `FORMAT_CANNOT_HOLD_LEVEL`에 대해 HTTP 400 오류가 발생하며, 이는 `ChlorosConnectError` (`CameraSession` / `ArraySession`) 또는 `RuntimeError` (`CameraHandle` / `ArrayHandle`)로 발생합니다. 단일 레벨인 `debayered`로 한 대의 카메라를 캡처할 경우 PNG/JPEG용 8비트 프레임을 가져오며, 이 경우 오류가 발생하지 않습니다.

방사계 데이터를 `format="tiff"` (`ext=".tiff"`) 형식으로 저장하고, `display` 및 인덱스 렌더링에는 PNG/JPEG 형식을 사용합니다.

#### 자동 노출의 작동 방식

- **노출을 먼저, 게인을 나중에 설정합니다.** 게인을 '자동'으로 설정하면, 노출이 허용되는 최대값에 도달할 때까지 게인은 0 dB로 유지됩니다.
- **수동 노출을 설정해도 게인은 '자동' 상태로 유지됩니다**. 사용자가 게인을 변경하거나 수동으로 설정할 때까지 이 상태가 지속됩니다. 다시 '자동'으로 전환하면 게인의 제한 값은 이전과 동일합니다.
- **작은 하이라이트는 장면을 어둡게 만들지 않습니다.** 프레임의 최대 1%에 해당하는 스페큘러는 무시되며, 알려진 보정 타겟의 패널에서는 패널 면적의 최대 0.5%까지 무시됩니다.
- **DAQ 노출 상한은 보정 타겟이 화면에 표시되는 동안에만 적용됩니다.** 이는 흰색 패널이 클리핑되는 것을 방지하며, 패널이 보이지 않을 때는 카메라가 장면 전체에 맞춰 노출을 조정합니다.
- **일정한 조명은 안정적으로 유지되며, 변화한 조명은 몇 초 내에 안정화됩니다.** 일정한 조명 상태에서는 노출과 그 한계가 고정되며, 조명이 크게 변한 후에는 화면이 몇 초 내에 안정화됩니다. 클리핑이 발생해도 노출이 영구적으로 제한되지 않으며, 관심 영역(ROI)이 변경되면 노출 값이 다시 조정됩니다.
- **밝기 목표치는 관리 가능한 범위 내에 유지됩니다.** 카메라가 노출 한계에 도달해 게인을 더 이상 높일 수 없는 상태에서는 밝기 목표치가 상승하지 않으므로, 빛이 다시 들어와도 프레임에 플레어가 발생하지 않습니다.
- **자동 노출이 반응하지 않는 카메라는 현재 조도 수준에 맞춰 고정됩니다.** 강렬한 햇빛 아래에서 카메라는 최단 노출로 떨어질 수 있으며, 모든 설정이 정상으로 표시되는 상태에서도 어두운 프레임을 유지한 채 그 상태로 머물 수 있습니다. Chloros는 프레임을 모니터링하여 해당 조도에 맞춰 카메라를 고정하고, 조도가 변하면 다시 정상으로 되돌립니다. 조명 센서가 어두워지거나 가려져도 이 과정에는 영향을 미치지 않습니다.

#### 어레이 구성원의 노출

LATTICE 카메라는 `ExposureTime`를 프레임 속도의 한 주기만큼 제한합니다. 어레이가 스트리밍되는 동안, Chloros는 모든 구성원을 `AcquisitionFrameRateEnable`를 켜진 상태로, 트리거 속도보다 약간 높은 속도로 구동하므로, 구성원의 노출 상한은 **모든 ROI에서** 트리거 주기보다 약간 낮은 수준에 위치합니다:

- 구성 요소의 프레임 주기는 트리거 주기에서 주기의 15%에 해당하는 여유 시간(최소 3 ms, 최대 주기의 절반)을 뺀 값이며, 노출 상한은 해당 프레임 주기에서 약 0.1 ms를 뺀 값입니다. 예시: 10 fps 트리거(100 ms)의 경우, 멤버 속도는 약 11.8 fps가 되고 노출 상한은 약 85 ms가 됩니다.
- **해당 상한을 초과하는 수동 `ExposureTime`는 이를 맞추기 위해 트리거 속도를 늦춥니다.** Chloros는 먼저 해당 멤버의 상한을 해당 값으로 열고, 속도가 느려진 상태로 몇 개의 트리거 주기를 실행한 후(쓰기 작업에 몇 초 더 걸릴 수 있음) 해당 값을 기록하며, 트리거는 노출이 허용하는 한도 내에서만 작동합니다. 자동 속도 배열의 경우, 수동 노출로 인해 속도가 느려졌던 모든 멤버가 다시 자동 노출로 돌아오면 이전에 실행되던 속도가 복원됩니다. 카메라 자체가 수용할 수 없는 값은 여전히 거부되며, 현재 허용 범위가 오류로 표시되고, 해당 쓰기 작업만을 위해 자동 노출이 꺼졌던 멤버의 경우 자동 노출이 다시 활성화됩니다. 1.3.0 이전 버전에서는 트리거 속도와 관계없이 작은 ROI가 각 멤버를 센서 자체의 자유 실행 주기로 제한했으므로, 트리거 속도와 무관하게: 768 × 304 해상도에서 4.9 ms였으므로, 어떤 멤버도 더 밝게 조정될 수 없었습니다.
- Chloros는 더 빠른 트리거에 맞추기 위해 수동 노출 시간을 절대 단축하지 않습니다. 자동 속도는 수동 노출이 허용하는 범위 내에서 유지되며, 노출이 허용하는 속도보다 빠른 고정 속도가 설정된 경우 해당 멤버는 그 속도에 맞는 충분한 상한 시간을 유지하며, 노출이 허용하는 속도보다 빠른 트리거는 처리하지 않습니다. 모든 멤버가 모든 트리거를 처리해야 할 때는 노출 시간이나 고정 속도를 낮추십시오.
- **리드백:** 멤버의 노드를 읽으면 `AcquisitionFrameRateEnable` True와 `AcquisitionFrameRate`가 멤버의 속도로 표시됩니다. 어레이가 정지하면 인에이블이 다시 꺼집니다.
- **이 두 노드는 어레이가 소유하고 있습니다. 어레이 구성원에 `AcquisitionFrameRateEnable` 또는 `AcquisitionFrameRate`를 기록하지 마십시오** — 대신 `target_fps` / `set_target_fps()`를 사용하여 어레이의대신 `target_fps` / `set_target_fps()`를 사용하여 어레이의 속도를 설정하십시오.

`arr.set_member_settings(serial, exposure_time=20000)`를 사용하여 멤버의 노출을 설정하십시오(`CameraSession.set_settings()`의 키워드; `chloros-cli lattice exposure --serial`도 동일한 기능을 수행합니다). 해당 멤버는 어레이에 남아 있습니다. `connect_camera()`를 사용하여 어레이 멤버를 열지 마십시오: connect는 이미 배열에 포함된 카메라를 거부하며, `CameraSession`는 연결 해제 시 해당 카메라를 배열에서 제거합니다. `ArraySession.status()["member_ae"]`는 모든 구성원의노출 및 게인 값을 보고합니다.

### 하나의 프로젝트 레코드, 모든 모드 지원

프로젝트 폴더에는 모든 장치의 설정이 포함됩니다: `cameras.json` — 각 카메라의 설정(`cameras[].settings`: 단일 카메라 및 각 어레이 구성원별)과 각 어레이의 전체 설정(`arrays[].capture`: 포맷, ROI, 비닝, 트리거 속도, 와이어 예산) — 그리고 `sensors.json`(각 광 센서의 통합 시간, 프레임 평균, 캡)까지 포함합니다. 앱, CLI, SDK 및 MCP 서버는 모두 동일한 파일을 동일한 방식으로 읽고 씁니다:

- **프로젝트에 연결된 장치는 해당 프로젝트에 저장되며, 모든 변경 사항이 프로젝트에 기록됩니다.** "프로젝트 하위"란 백엔드(앱)에서 프로젝트가 열려 있는 상태를 의미하거나, `ChlorosProject`(`open_project()`)를 통해 백엔드에 자체 폴더 이름을 지정하는 경우를 말합니다 (`project_path`). 프로젝트에 아직 등록되지 않은 카메라, 어레이 또는 광 센서는 연결 시 해당 설정과 함께 프로젝트 파일에 추가됩니다(`connect_camera()`, `connect_array()`, 광 센서 연결, CLI, 앱, `chloros-mcp`)에 해당 센서의 설정과 함께 파일에 추가됩니다. 어떤 모드에서든 변경이 발생하면 (`set_settings()`, `ArraySession.set_member_settings()`, `set_target_fps()`, 어레이 설정, `set_cap()`, CLI, 앱의 제어 기능)에서 이루어진 변경 사항은 잠시 후 해당 프로젝트의 파일에 저장됩니다. 카메라에서 설정된 노출, 게인 또는 AE 목표값은 ‘유지’ 상태로 저장되므로, 프로젝트를 다시 열면 해당 설정이 다시 적용됩니다.
- **프로젝트를 열면 모든 모드에서 해당 프로젝트의 설정을 기반으로 장치가 설정됩니다**: 별도의 설정이 없는 카메라, 어레이 또는 광 센서는 프로젝트의저장된 설정을 적용받습니다(프로젝트 열림 시: `lattice cam-connect` / `array-connect`, 프로젝트 재개 시: `chloros-cli project connect`, `open_project().connect_all()`). 새로운 ID로 연결된 어레이는 프로젝트에 저장된 것과 동일한 카메라를 가진 어레이를 업데이트하며, 다른 카메라를 가진 어레이는 그 옆에 추가됩니다.
- **앱 자체의 저장 기능은 기기의 실시간 설정을** 이전 버전의 복사본으로 덮어쓰지 않으며, CLI나 SDK 호출자가 저장한 구성원별 재정의 설정을 유지합니다.
- **사용자가 설정한 픽셀 형식이 프로젝트에 유지됩니다.** 배열 설정, `--pixel-format` 또는 `connect_array(pixel_format=)`를 통해 설정된 형식(8비트 포함)은 정확히 저장되고 재연결 시 그대로 적용됩니다. 설정이 없는 경우, 각 카메라는 12비트 기본값(모노 센서의 경우 `BayerRG12`, `Mono12`)으로 연결됩니다. 형식 이름은 해당 비트 심도를 의미하며, 제품군은 센서의 것이므로, 컬러 카메라는 절대 모노 포맷으로 설정되지 않으며, 모노 카메라는 베이어 포맷으로 설정되지 않습니다. 그리고 `[PixelFormat <serial>] ... written` 줄은 포맷이 매핑되었음을 나타냅니다. 8비트 캡처의 출력 파일은 반사율을 포함하여 8비트이며, 재조정되어 `PixelScale`로 표시됩니다.
- 프로젝트 없이 연결된 장치는 아무것도 기록되지 않습니다.
- **녹화는 그 이후에 생성된 프로젝트를 따르며, 열기 작업 시에는 기존 프로젝트를 유지합니다.**프로젝트가 생성되면 — `create_project()`, `chloros-mcp`의 `create_project`, 또는 앱의**새 프로젝트**— 또는 복제(앱의**복제**기능으로, 복사본을 여는 경우)되는 동안 녹화가 여전히 이전에 열려 있던 프로젝트에 데이터를 기록하고 있다면, 녹화는 계속 진행되며 기록된 내용은 새 프로젝트 내의 동일한 위치에 저장됩니다 (아래)에 저장됩니다. 기존 프로젝트를 여는 경우 — `ChlorosProject.process()` (백엔드에서 해당 프로젝트를 엽니다), `chloros-cli process`, 앱의**프로젝트 열기** — 녹화가 진행 중인 프로젝트가 그대로 유지됩니다: 녹화는 해당 프로젝트에서 완료되며, 그 파일들은 해당 프로젝트의 파일 목록에 추가됩니다(앱은 프로젝트를 열기 전에 모든 녹화를 중지합니다).
  - `ArraySession.record()` 및 `ArraySession.burst()`: 녹화가 중지되면 동영상 또는 모든 프레임이 포함된 버스트 폴더가 전체적으로 해당 위치로 이동합니다. 중지 결과에는 새로운 경로 (`out_dir`, `video_path`)를 표시하며, 해당 경로를 기반으로 버스트 파일 합성 작업이 수행됩니다. 드라이브에서 이동이 거부되면 녹화 파일은 원래 위치에 그대로 남아 있으며, 결과에도 그 사실이 표시됩니다(`relocation_error`). 중지된 버스트의 원본 프레임과 `.daq`는 해당 버스트가 포함된 프로젝트의 파일 목록에 추가되지만, 동영상은 프로젝트의 파일 목록에 추가되지 않습니다.
  - `ArraySession.continuous()`: 변경 후의 모든 캡처는 새 프로젝트에 저장되며, 변경 전에 저장된 캡처는 원래 위치에 그대로 남습니다.
  - `DAQSensorSession.record_start()` (및 앱 자체의 광센서 녹화): 녹화가 중지되면 `.daq`는 새 프로젝트의 `light_sensor` 폴더로 이동하며, `record_stop()`는 새로운 경로를 반환하고 해당 파일은 해당 프로젝트의 파일 목록에 추가됩니다.

  열려 있는 프로젝트 외부의 폴더로 기록된 내용은 원래 위치에 그대로 남아 있으며, `start_camera_continuous()` / 멀티 스트림 작업은 지정된 폴더를 유지합니다.
- **앱 내에서 새 프로젝트를 생성하면 그 이후에 수행되는 캡처 작업은 해당 프로젝트로 이동합니다.** 프로젝트를 생성하거나 복제해도 모든 캡처 실행 및 녹화는 계속 진행되며, 그 이후에 수행되는 각 캡처 작업(‘모두 캡처’, ‘캡처 시작’)은 새 프로젝트에 저장되어 해당 프로젝트의 파일 목록에 추가됩니다. 앱에서 기존 프로젝트를 열면 먼저 모든 캡처가 중지됩니다.
- **프로젝트는 녹화 중에는 이동되지 않습니다.**열려 있는 프로젝트를 다른 작업 디렉터리로 이동하는 경우(앱의**설정 → 작업 디렉터리**; SDK 호출로는 프로젝트를 이동할 수 없으며, HTTP 클라이언트가 `/api/move-project`로 요청을 보내는 경우는 예외)는 배열 녹화나 버스트, 배열 연속 캡처, 카메라 연속/다중 스트림 작업 또는 광센서 녹화가 해당 디렉터리에 데이터를 기록 중인 동안 HTTP 409 `PROJECT_RECORDING` 오류로 거부됩니다: `Project '<name>' is recording (<what>), so moving the project now is refused. Stop the recording, then try again.`

이는 연결 해제 시 카메라의 상태가 어떻게 유지되는지와는 별개의 문제입니다([카메라 메모리](#camera-memory--keep_settings--clear_settings) 참조): 프로젝트는 열릴 때 자체적으로 카메라 설정을 지정합니다.

### 카메라 메모리 — `keep_settings` / `clear_settings`

어레이가 연결되면 해당 카메라들은 어레이의 설정을 따르며, 이후 사용자는 이 설정을 변경할 수 있습니다. Chloros가 카메라를 해제할 때 — `ArraySession.disconnect()` / `force_disconnect()`, 일시적 함수의 종료 시, `start_camera_continuous()`의 중지 시 — 각 카메라를 연결 전의 상태로 되돌립니다. 즉, 트리거 및 동기화 라인 배선, PTP 및 카메라 설정 — `ExposureAuto` / `GainAuto` (그리고 수동 `ExposureTime` / `Gain`), `TargetBrightness` 및 AE 한계와 댐핑, 화이트 밸런스, 블랙 레벨, `PixelFormat`, 비닝 및 ROI, 컬러 파이프라인(`GammaEnable` / `Gamma`, `LUTEnable`, `ColorTransformationEnable`), `ReverseX` / `ReverseY`, `UserSetSelector`, PTP의 `PtpSlaveOnly` / `PtpDomainNumber`, 패킷 크기 (`GevSCPSPacketSize` / `DeviceStreamChannelPacketSize`), 페이싱 (`DeviceLinkThroughputLimitMode` / `DeviceLinkThroughputLimit`, `GevSCPD`, `GevSCFTD`), 청크 데이터 (`ChunkModeActive`, 각 `ChunkEnable`는 해당 `ChunkSelector` 아래에 위치), 트리거 입력 글리치 제거 및 배열이 슬레이브의 동기화 라인을 제공합니다 (`LineFilterSelector` / `LineFilterWidth`) 및 획득 프레임 속도 (`AcquisitionFrameRate`는 활성화 상태일 때만 해당: 비활성화 상태에서는 설정값이 아닌 페이싱이 허용하는 최대 속도가 읽힙니다). 차이점만 기록되며, 카메라가 먼저 정지했고, 프레임 주기가 필요한 노출 전의 페이싱 및 프레임 속도입니다.

프로젝트 외부에서 카메라를 사용할 경우, 두 개의 인수가 카메라의 **자체 메모리**에 기록됩니다. 이는 어떤 소프트웨어에서든(`disconnect()` / `force_disconnect()`, 모든 일시적 함수, `start_camera_continuous()` 및 `ChlorosProject.disconnect_all()`; MCP: `disconnect_device` 및 일시적 도구의 동일한 인자; CLI: `--keep-settings` / `--clear-settings`):

- **`keep_settings=True`**는 각 카메라의 설정을 세션 종료 시점에 메모리에 저장된 상태 그대로(UserSet2, 부팅 설정으로 지정됨) 저장합니다. 카메라는 여전히 자유 실행 상태로 유지되며(`TriggerMode` `Off`), 동기화 라인 배선도 발견된 그대로 유지됩니다. UserSet2가 없는 카메라는 UserSet1에 저장되며, 이 컴퓨터는 이를 기억하므로 Chloros의 어레이 자가 복구 기능 — Chloros의 기준선에서 벗어난 UserSet1을 재설정하는 기능 — 은 해당 카메라를 그대로 둡니다.
- **`clear_settings=True`**는 각 카메라의 메모리를 공장 출하 시 설정으로 재설정합니다: Chloros의 기준값, 즉 Chloros의 어레이 자가 복구 기능이 반영하는 값 — `TriggerMode`, `Off`, `TriggerSource`, `Line2`를 통해 적용된 공장 기본값, 자동 노출, RGB 카메라의 보정된 화이트 밸런스 유지 — UserSet1에 포함되어 부팅 세트로 설정됨.
- 둘 다 아님 (`ValueError`; 백엔드가 400을 반환함). 응답에는 `camera_memory`가 포함되며, 카메라별로 `{"saved_to": "UserSet2" | "UserSet1" | None, "errors": [...]}` 또는 `{"cleared": bool, "errors": [...]}` — `force_disconnect()`가 이를 반환하며, 작업의 경우 `stop()`와 함께 제공됩니다.

앱은 이를 제공하지 않습니다.

**모든 종료 경로에서 카메라가 원래 상태로 복원됩니다.**발견된 상태 그대로의 복원 처리는 각 종료 경로에서 실행됩니다: `CameraSession.disconnect()`(앱의 카메라**X**), 앱의**X**가 배열이나 배열의 구성원 중 하나일 경우(배열 전체가 해제됨), 앱 종료 시(백엔드는 모든 카메라, 배열 및 실행 중인 연속 녹화를 복원하는 데 최대 20초가 주어집니다; 여전히 실행 중인 일시적 함수나 일회성 호출은 다음 안전 지점에서 중지되어 먼저 카메라를 복원하고, 최대 8초 동안, 그리고 HTTP 503 `SHUTTING_DOWN`를 반환합니다; 여전히 실행 중인 `connect_array()`는 다음 카메라에 도달하기 전에 중지하고, 열었던 카메라를 복원하며, 최대 10초 동안, 그리고 `ChlorosConnectError`(HTTP 503 `SHUTTING_DOWN`)를 발생시킵니다; 그리고 종료 중에 도착한 함수는 아무것도 열지 않습니다: HTTP 503 `SHUTTING_DOWN`), 그리고 강제 종료되거나 터미널이 닫힌 `chloros-cli` 명령(해당 작업이 중지되고 카메라가 먼저 원래 상태로 복원됨). 충돌하거나 강제 종료된 백엔드는 아무것도 원래 상태로 되돌릴 수 없으며, 전원을 껐다 켜면 각 카메라가 메모리 상태로 복원됩니다.**프로젝트는 카메라를 직접 설정합니다.** 어레이가 프로젝트를 통해 연결될 때 — 앱의 열린 프로젝트나 `ChlorosProject`, 백엔드(`project_path`)에 자체 이름을 알리는 경우 — 각 구성원은 프로젝트가 저장해 둔 설정을 적용받습니다: 해당 카메라에 적용된 노출, 게인 또는 AE 목표값(사용자 설정으로 유지되므로 자동 노출은 이를 건드리지 않음)과 화이트 밸런스 모드(Chloros 보정; 카메라 자체의 화이트 밸런스는 비활성화된 상태로 유지됨), 카메라 메모리에 저장된 모든 내용을 반영합니다.

### 캡처 모드 및 레코더

배열 표면은 GUI 캡처 패널을 그대로 반영합니다: 단일 / 연속 / 간격 / 최단 셔터 모드, 그리고 두 가지 레코더(라이브 합성 비디오 및 RAW 버스트 → 오프라인 재처리).

```python
import time, chloros_sdk

with chloros_sdk.connect_array(serials) as arr:
    # Single (default) — one synced group
    arr.capture("out/", processing="reflectance")

    # Fastest — raw + .daq + combined index now, calibrate later
    arr.capture_fastest("flightline/")

    # Interval — one reflectance pass every 2 s, 5 passes (bounded so it ends)
    arr.capture_repeated("timelapse/", count=5, interval_s=2.0,
                         processing="reflectance",
                         on_capture=lambda i, r: print(f"pass {i}: {len(r)} frames"))

    # Continuous — every synced group for 60 s, saved by the backend itself
    with arr.continuous("run1/", processing="raw", duration_s=60) as run:
        run.wait()
    print(run.result["captures"], run.result["captures_per_s"],
          run.result["stop_reason"])

    # Combined-index video/GIF recorder: the GUI's combined view, or with no
    # GUI open the array's NDVI (pass expr="..." for another index)
    with arr.record("monitoring/", fps=10, gif=True) as rec:
        time.sleep(30)
    if rec.result and rec.result.get("error"):   # leaving the block does not raise
        print("recording failed:", rec.result["error"])
    else:
        print(rec.result["video_path"])

    # Raw-Bayer burst → offline reprocess into calibrated video(s)
    with arr.burst("capture/", duration_s=5) as b:
        pass
    out = arr.build_video(b.result["out_dir"], products=[
        {"kind": "per_cam", "level": "reflectance"},
        {"kind": "combined", "level": "index"}])
    print(out["outputs"])
```

- **`capture_repeated`**는 SDK의 연속/간격 루프입니다. 스크립트에서 이 루프를 중단시킬 수 있는 `Ctrl+C`가 없기 때문에,**반드시** `count` 및/또는 `duration_s`를 전달해야 합니다(둘 중 하나에 도달하면 중지됨). `interval_s`는 각 패스의 시작 시점부터 측정되며 (GUI와 동일). 연속 패스(`interval_s=0`)는 `group_seq` → `after_seq`로 연결되므로, 이들은 배열의속도로 동기화된 연속 그룹입니다. 나머지 kwargs는 `capture()`로 바로 전달됩니다.
- **`continuous`**는 백엔드 자체의 연속 캡처 기능으로, GUI의 ‘연속 모드’에 해당합니다.**높은 프레임 속도로 모든 트리거를 디스크에 기록하려면 `capture_repeated()` 대신 이 옵션을 사용하십시오:** `capture_repeated()`는 스크립트에서 그룹당 하나의 HTTP 요청을 보내는 반면, `continuous()`는 그룹 간 왕복 통신 없이 백엔드 내에서 두 개의 캡처를 동시에 처리합니다. 9대 카메라 어레이에서 ROI 512로 측정된 결과: 73–77 캡처/초, 약 78을 출력하는 어레이와 비교. 간격 캡처(`interval_s`)에는 `capture_repeated()`를 유지하고 및 하나씩 검토하는 패스용으로 `capture_repeated()`를 유지하십시오. Windows에서는 높은 처리 속도를 위해 백엔드를 Microsoft Defender에서 제외하십시오([1단계](#step-1--install-the-chloros-platform-package) 참조).
- **`record`**는 *모니터링 등급*입니다: 렌더링된 실시간 통합 인덱스 합성 화면을 기록합니다. GUI의 결합 보기가 열려 있으면 표시된 그대로 해당 보기를 기록합니다. 열려 있는 것이 없으면 백엔드가 뷰를 직접 렌더링합니다: 배열을 전달하면 `expr`를, 그렇지 않으면 배열의 NDVI(라이브 뷰에서 선택되는 660 nm 및 850 nm에 가장 가까운 적색 및 NIR 밴드, 라이브 뷰에서 이를 선택하므로)를 기록합니다. 배열이 정렬되면 `expr`에서 배열이 지정한 대로 밴드 이름을 지정하십시오: M3C 카메라의 경우 `<band>_<filter>`(`NIR_850_RGN`, `Red_660_RGN`, `Orange_615_OCN`), M3M용 `Mono_<filter>`(`Mono_850`); 짧은 색상 이름(`NIR`, `Red`) 는 배열의 한 밴드에만 해당 이름이 지정된 경우에만 작동합니다. 녹화 중에 열린 GUI 뷰는 열려 있는 동안 녹화됩니다. 프레임을 기록하지 않는 녹화는 `error` 및 `"no-frames: …"` 오류로 실패하며(배열이 여전히 정렬 중이거나 노출을 안정화 중일 수 있음), `stop()` 예외가 발생합니다([`RecorderHandle`](#recorderhandle) 참조). 어레이당 하나의 복합 레코더만 허용되며(이미 실행 중인 경우 예외 발생).
- **`burst` → `build_video`**는 *분석 등급*입니다: `burst`는 원시 프레임 + 프레임별 매니페스트 + `<output>/bursts/<base>/` 하에서 고유한 DLS 판독값마다 하나의 `.daq`를 그랩 루프의 최대 속도로 기록합니다(체인 없음, exiftool 없음, 라이브 뷰 없음). `build_video`는 각 프레임을 가장 가까운 `.daq`와 시간적으로 매칭하고, 임포트 파이프라인의 방사도/반사율/지수 체인을 다시 실행합니다. `products`는 `{"kind": "per_cam"|"combined", "level": "radiance"|"reflectance"|"index"}`(기본값: 결합된 지수)의 목록입니다. `burst().stop()`는 또한 최선의 노력을 다한 결합된-지수 빌드를 자동으로 트리거하며, 이는 중지 결과에서 `build_job`로 반환됩니다.

##### `build_video` 결과

| 키 | 의미 |
| --- | --- |
| `success` | `True`는 요청된 모든 제품이 요청대로 구축된 경우에만 표시됩니다: 출력물이 하나 이상 있고, `skipped`에는 아무것도 없으며, `degraded`에 해당하지 않는 경우에만 표시됩니다. |
| `outputs` | 작성된 동영상(및 GIF)의 경로. |
| `errors` | 문제점(텍스트 형식). |
| `skipped` | 동영상이 **생성되지 않은** 요청된 제품마다 하나의 `{"serial", "level", "reason"}` — 카메라의 (해당 카메라에 대한 보정 번들이 없거나, 반사율에 대한 DLS 측정값이 없거나, 처리 체인이 실패한 경우) 또는 `"kind": "combined"`, `serial`, `None`를 포함하는 결합 인덱스의 경우 (카메라 중 적색/NIR 대역 쌍이 없거나, 두 대역 모두에 대한 복사도를 가진 그룹이 없음). 항목이 하나라도 있으면 `success` `False`가 생성되며, 나머지 영상은 여전히 기록되고 나열됩니다. |
| `substitutions` | 요청된 수준과 다르게 렌더링된 카메라 및 그 이유 — RGB 카메라가 복사도/반사도를 요청한 경우, 카메라의 보정 번들이 있을 때 화이트 밸런스가 조정된 디스플레이 비디오를 반환합니다. 오류가 아닙니다. |
| `notes` | 문제점이 아닌 정보성 텍스트 — 예: 카메라별 인덱스 표현식이 없는 카메라는 인덱스 영상이 없거나, RGB 카메라의 디스플레이 영상에 화이트 밸런스가 적용되지 않은 경우(보정 번들이 없거나 공장 출하 시 화이트 밸런스가 설정되지 않은 경우), 카메라 이름을 명시합니다. 오류가 아닙니다. |
| `degraded` | `True`: 요청된 제품과 다른 제품이 작성된 경우: 오프라인 결합 인덱스는 내장된 적색/NIR 정규화 차분(ND)이므로, 따라서 구성된 인덱스 표현식은 평가되지 않습니다. 이로 인해 `success` 및 `False`가 발생합니다. |
| `combined` | 결합 인덱스 요약: 사용된 밴드, LUT 범위, 생성된 프레임. |
| `clamp_masks` | 프레임별 [언믹스 클램프 마스크](#unmix-clamp-mask--which-pixels-a-band-was-clamped-on)의 경로프레임별 보정된 TIFF 파일(`save_tiffs=True`)의 [언믹스 클램프 마스크](#unmix-clamp-mask--which-pixels-a-band-was-clamped-on) 경로. TIFF 파일의 `<level>/` 폴더 옆에 있는 `clamp_masks/<level>/`에 저장되며, 픽셀이 클램프된 각 TIFF 파일당 하나씩 포함됩니다. 동영상에는 포함되지 않습니다. |
| `clamp_fractions` | 마스크 대신 비디오가 제공하는 정보: 비디오 제품당 하나의 `{"kind", "serial", "level", "frames"}` (`kind` `"combined"`는 통합 인덱스에 포함된 카메라들에 대한 것), `frames`는 `[frame, R, G, B]` 목록입니다. 이는 해당 프레임에서 언믹스(unmix)가 R/G/B 채널이 차지하는 대역 내에서 클램핑한 픽셀의 비율을 나타냅니다 (`Chloros:Band{R,G,B}UnmixClampFraction`). |

#### `RecorderHandle`

`ArraySession.record()` 및 `ArraySession.burst()`에 의해 반환됩니다. 스코프 종료 시 자동으로 중지되도록 컨텍스트 매니저로 사용하거나, 수동으로 제어할 수 있습니다.

| 구성 요소 | 설명 |
| --- | --- |
| `job_id` | 백엔드 작업 ID (문자열). |
| `kind` | `"composite"` (`record`에서 파생) 또는 `"raw"` (`burst`에서 파생). |
| `start_stats` | `start` 호출에 의해 반환된 딕셔너리. |
| `result` | 실행 중일 때는 `None`; 중지된 후의 최종 중지 결과 딕셔너리. |
| `stats(timeout=10.0)` | 실시간 작업 통계 (기록된 프레임 수, 실제 fps, 경과). |
| `stop(timeout=60.0)` | 레코더를 중지합니다. 최종 결과를 반환하고 캐시에 저장합니다(중지 요청 자체가 실패한 경우 `.result`는 `None` 상태를 유지합니다). 실패한 `record()` 녹화(프레임이 하나도 기록되지 않은 경우 — `error` `"no-frames: …"`)의 경우 —의 경우, 결과를 `.result`에 캐시한 다음 `ChlorosConnectError`를 발생시킵니다. 이멱포텐트(두 번째 호출 시 예외를 발생시키지 않고 캐시된 결과를 반환합니다). `with` 블록을 벗어나면 예외를 발생시키지 않고 레코더가 중지되므로, 그 후 `rec.result.get("error")`를 확인하십시오. |

```python
rec = arr.burst("capture/")
# ... drive manually ...
print(rec.stats()["frames"])
result = rec.stop()
print(result["out_dir"], result.get("build_job"))
```

#### `ContinuousHandle`

`ArraySession.continuous()`에 의해 반환됩니다. 컨텍스트 매니저로 사용하십시오 — `with` 블록을 벗어나면 캡처가 아직 실행 중일 경우 캡처가 중지됩니다 — 또는 `status()` / `wait()` / `stop()`로 제어하십시오로 제어하십시오. 배열당 한 번에 하나의 연속 캡처만 실행됩니다.

| 멤버 | 설명 |
| --- | --- |
| `start_status` | 시작 호출에서 반환된 상태 딕셔너리. |
| `result` | 실행 중에는 `None`; 캡처가 종료된 후의 최종 상태 딕셔너리(`stop()`에 의해 설정되거나, 또는 종료된 것을 감지한 `status()` / `wait()`에 의해 설정됨). |
| `status(timeout=10.0)` | 활성 상태 딕셔너리 (아래 필드 참조). |
| `wait(timeout=None, poll_s=0.5)` | 캡처가 자체적으로 종료될 때까지(`max_captures` 또는 `duration_s`에 도달, 배열 소진, 드라이브 용량 초과, 오류 발생 등) 대기한 후 최종 상태를 반환합니다. `timeout`의 경우, 캡처가 여전히 실행 중이라 하더라도 지정된 시간이 지난 후의 최신 상태를 반환합니다: `running`를 확인하십시오. |
| `stop(timeout=30.0)` | 캡처를 중지하고 최종 상태를 반환합니다. 이멧포텐트(두 번째 호출 시 캐시된 결과를 반환함). |

상태 필드:

| 필드 | 의미 |
| --- | --- |
| `running` | 캡처가 실행 중일 때 True입니다. |
| `captures` | 동기화된 그룹이 저장되었습니다. |
| `files` | 파일이 기록되었습니다. |
| `failed_files` | 쓰기 실패한 파일 ([`CaptureResult.failed`](#files-that-could-not-be-written--captureresultfailed) 참조). |
| `failed_captures` | 캡처 재시도 건을 포함하여 전체적으로 실패한 캡처 시도(어레이에 제때 새로운 그룹이 없었을 때 발생한 HTTP 503 오류). |
| `skipped_groups` | 어레이가 게시했으나 저장되지 않은 그룹(캡처가 뒤처짐). |
| `captures_per_s` | 시작 이후의 평균 저장률. |
| `elapsed_s` / `started_at` | 현재까지의 실행 시간(초); 시작 시간(유닉스 에포크 초). |
| `last_group_seq` | 가장 최근에 동기화되어 저장된 그룹. |
| `last_error` | 가장 최근 오류, 또는 `None`. |
| `stop_reason` | 종료된 경우: `count` (`max_captures` 도달), `duration` (`duration_s`에 도달), `stopped` (`stop()` 호출), `array_gone` (어레이 연결이 끊어짐), `error` (연속 20회 실패 또는 백엔드에서 거부한 요청), `disk_full` (드라이브에 저장할 공간이 없는 파일). `None` 실행 중. |
| `max_captures` / `duration_s` | 시작 시 설정된 경계(`0` = 없음). |
| `defender_cores` | 지난 10초 동안 Microsoft Defender가 사용한 CPU 코어 수; `None`는 측정값이 나올 때까지, Defender가 설치되지 않은 호스트에서는 항상 `None`로 표시됩니다. |
| `defender_warned` | 캡처가 Defender로 인해 속도가 저하되고 있음을(1개 이상의 코어에서) 기록한 경우 true가 됩니다. 백엔드를 Defender 스캔에서 제외하십시오 — [1단계](#step-1--install-the-chloros-platform-package) 참조를 참조하십시오. |

```python
run = arr.continuous("captures/run1", processing="raw", max_captures=5000)
try:
    while True:
        st = run.wait(timeout=5)              # returns every 5 s while running
        print(f"{st['captures']} captures, {st['captures_per_s']:.1f}/s, "
              f"{st['skipped_groups']} skipped")
        if not st["running"]:
            break
finally:
    final = run.stop()                        # no-op once it has ended
if final["failed_files"] or final["stop_reason"] in ("disk_full", "error"):
    raise SystemExit(f"continuous capture: {final['stop_reason']}: "
                     f"{final['last_error']}")
```

### 자체 파이프라인에서 라이브 프레임 처리

동기화된 프레임을 사용자 정의 코드(머신 러닝, 사용자 정의 뷰어, 사용자 정의 레코더 등)에 공급하려면 `ArraySession.frame_stream()`를 사용하십시오. 각 틱마다 `GrabResult`가 생성됩니다: 모든 카메라에 대해 `{serial: numpy array}`가 **단일 트리거로** 생성되며, 해당 트리거의 ID는 `.group_id`에 포함됩니다. 이는 프로젝트 API의 `ArrayHandle.frame_stream()`와 동일한 구현 방식이므로, 따라서 [읽기당 하나의 트리거](#one-trigger-per-read--group_id), [`band`](#exporting-a-single-band--band), [`align`](#aligning-the-streams--align) 및 [`bit_depth`](#bit_depth--trading-precision-for-bandwidth)에 설명된 내용은 두 가지 모두에 적용됩니다.

```python
import chloros_sdk
from chloros_sdk import FrameFetchError

serials = ["213800234", "214000533", "214701288", "214701292"]   # master first

with chloros_sdk.connect_array(serials) as arr:
    try:
        for frames in arr.frame_stream(processing="raw", fps=10, duration=300,
                                       with_metadata=True):
            if not frames:              # no camera answered this tick
                continue
            if frames.errors:           # cameras missing from this trigger
                print("left out:", {s: str(e) for s, e in frames.errors.items()})
            for serial, (img, hdr) in frames.items():
                # img: numpy array (raw: one Bayer or mono plane)
                # hdr: group_id, frame_id, timestamp_ns, exposure_us, gain_db, ...
                my_pipeline(frames.group_id, serial, img, hdr["timestamp_ns"])
    except FrameFetchError as exc:      # the stream cannot continue
        print("stream ended:", exc.status, exc)
```

- **반환된 각 집합은 새로운 트리거입니다.** `frames.group_id`는 스트림 내에서 절대 반복되지 않습니다. 배열이 실행되는 속도보다 더 많은 `fps`를 요청하면, 스트림은 다음 트리거를 기다리므로 두 속도 중 더 느린 속도로 실행됩니다.
- **누락된 카메라는 이름이 지정되며, 빈 자리는 채워지지 않습니다.** 해당 트리거에서 처리되지 못한 카메라는 딕셔너리에서 제외되며 `frames.errors`에 나열됩니다.
- **종료 시 명확한 신호를 보냅니다.** 거부 시, 다음 틱에서 반복되거나, 60초 동안 프레임이 없거나, 60초 동안 새로운 트리거가 없을 때 `FrameFetchError`가 발생합니다.**제공하는 성능.**`frame_stream()`는 전체 numpy 프레임을 로컬 HTTP 연결을 통해 전송하며, 최대**170 MB/s**에 가까운 속도를 기록합니다. 이는 원시 프레임이나 빈 처리된 프레임, ROI 배열의 실시간 처리에 충분하지만, 모든 트리거를 풀 해상도로 처리하기에는 부족합니다.**모든** 트리거를 유지하려면 다음 명령을 사용하여 디스크에 저장해야 합니다. [`continuous()`](#continuoushandle)를 사용해야 합니다. 패킹 전송 방식을 적용한 4대 카메라 어레이(FOCN, FRGN, FRGB, F450)에서 측정된 결과이며, 각 틱은 하나의 트리거 세트를 완전히 포함했습니다:

| | 전체 센서 (어레이 14.2–14.4 fps) | 2 × 2 비닝 (어레이 ~33 fps) |
| --- | --- | --- |
| `frame_stream(processing="raw")` | 6.7 세트/초 | 22.2 세트/초 |
| `frame_stream(processing="debayered")` | 2.3 세트/초 | 9.3 세트/초 |
| `frame_stream(processing="display")` | 0.8 세트/초 | 3.0 세트/초 |
| 카메라별 MJPEG, 720p 해상도 4개 스트림 | 각 스트림당 13.5 fps; 모든 프레임이 트리거와 일치 | 각 스트림당 ~25 fps; 4개 스트림 모두에서 트리거의 약 45%가 존재 |

사용자 측에서 처리량 및 디모자이크를 수행하려면 `processing="raw"`를 사용하십시오. `debayered` 레벨은 또한 보정 팩의 다크 오프셋을 제거하지만, 반면 `raw`의 디모자이크는 이를 제거하지 않습니다.

#### 카메라별 MJPEG 스트림 (RTSP 미지원)

**RTSP는 지원되지 않습니다.** 스트리밍 인터페이스는 HTTP를 통한 MJPEG(`multipart/x-mixed-replace`, 파트당 1개의 JPEG, 각 파트에 `Content-Length` 적용)이며, 카메라당 하나의 URL가 제공됩니다:

| URL | 제공하는 내용 |
| --- | --- |
| `GET /api/camera/stream/<serial>` | 8비트 미리보기, 최대 ~30 fps. 어레이 구성원의 경우, `max_h` (기본값 720, 64–2160) 및 `jpeg_q` (기본값 80)가 크기와 화질을 설정합니다. |
| `GET /api/camera/<serial>/stream-annotated` | GUI 오버레이(제브라, 그리드, 십자선, 히스토그램, 피킹, 스팟 미터)가 그려진 동일한 미리보기입니다. `ChlorosLocal.camera_stream()`가 이를 읽어줍니다. |
| `GET /api/camera/<serial>/stream-reflectance` | 대역별 8비트 반사율 — [실시간 반사율 스트림](#live-reflectance-stream-mjpeg)을 참조하십시오. |

- **트리거 헤더.** 배열 구성원의 경우, 세 스트림 모두의 모든 부분에 `X-Chloros-Group-Id`, `X-Chloros-Frame-Id` 및 `X-Chloros-Timestamp-Ns`를 포함합니다. 동일한 `X-Chloros-Group-Id`를 가진 서로 다른 카메라의 파트는 동일한 트리거에 해당합니다: 파츠를 수집한 후 해당 헤더를 기준으로 그룹화하여 세트를 재구성합니다. 독립형 카메라에는 그룹이 없습니다. 이 카메라의 미리보기 파츠에는 이러한 헤더가 전혀 포함되지 않으며 (반사율 부품에는 빈 `X-Chloros-Group-Id`가 포함됩니다).
- **미리보기 스트림은 부하가 걸리면 프레임을 생략합니다.** 가장 최신 프레임을 제공하고 제시간에 전송하지 못한 프레임은 건너뛰므로, 어레이 속도가 높을 경우 모든 트리거가 모든 카메라의 스트림에 도달하지는 않습니다(위의 표 참조). 모든 트리거에 대해 `frame_stream()` 또는 `continuous()`를 사용하십시오.
- **`/api/camera/stream/<serial>`는 어레이의 통합 라이브 스트림이 열려 있는 동안 일시 중지됩니다** (`/api/camera/array/stream/<array_id>` — GUI의 어레이 보기가 이를 엽니다). 이후 각 구성원의 스트림은 `X-Chloros-*` 헤더가 없는 "일시 중지됨(어레이 스트림 활성)" 플레이스홀더 프레임 하나를 전송하며, 통합 스트림이 닫힐 때까지는 더 이상 아무것도 전송하지 않습니다. 이후 자동으로 재개됩니다. `stream-annotated` 및 `stream-reflectance`는 일시 정지되지 않습니다. 스크립트가 카메라별 미리보기 스트림을 읽을 때는 GUI의 어레이 보기를 닫으십시오.
- **이미지가 아닌 숫자의 경우 `frame_stream()`를 사용하십시오.**JPEG는 손실 압축 방식이며 8비트입니다. `frame_stream()`는 경로에 코덱이 없어도 동일한 결과를 반환합니다.

### 드론 측량을 위한 간격별 캡처

측량 비행에서는 어레이가 생성할 수 있는 모든 프레임이 아니라, 이동 거리 몇 미터마다 하나의 동기화된 이미지 그룹이 필요합니다. 이를 제어하는 두 가지 설정은 다음과 같습니다:**`target_fps`**는 카메라가 프레임을 촬영하는 빈도를 설정하고,**`capture_repeated(interval_s=…)`**는 프레임이 저장되는 빈도를 설정합니다.**카메라는 저장 사이에도 계속 스트리밍을 유지합니다.** 각 간격마다 가장 최근의 동기화된 그룹이 저장되며, 그 사이의 프레임은 기록되지 않습니다. 자동 노출은 매 프레임마다 조정되므로, 저장이 이루어질 때쯤이면 노출은 이미 안정화된 상태입니다. 촬영 사이에 카메라가 정지했다가 매 촬영 시마다 다시 활성화되는 모드는 없습니다. 그런 모드가 있다면 매 촬영마다 스트리밍이 시작되고 노출이 안정화될 때까지 기다려야 하기 때문입니다.**간격에 필요한 프레임 속도를 고정하십시오.**자동 모드로 두면 어레이는 호스트와 네트워크가 감당할 수 있는 최고 속도로 실행됩니다. 속도를 더 낮게 고정하면 저장되는 내용에 변화를 주지 않으면서도 카메라 발열, 네트워크 부하 및 호스트 CPU 사용량을 줄일 수 있습니다.**간격당 2~4 프레임**이 나오도록 속도를 선택하십시오:

- 각 저장 작업은 요청 후 다음 프레임이 올 때까지 대기하므로, 저장된 그룹은 예정된 시점보다 최대 한 프레임 주기만큼 늦게 도착합니다. 프레임 속도가 높을수록 이 간격을 줄일 수 있습니다.
- 자동 노출은 프레임당 한 번씩 조정되므로, 프레임 속도가 매우 낮으면 지나가는 구름을 따라가는 노출 속도가 느려집니다.

**비행 계획에서 간격을 계산해 보세요:**

```
spacing    = along-track footprint × (1 − forward overlap)
interval_s = spacing / ground speed
```

예를 들어, 80% 전방 중첩률에서 60m의 촬영 범위는 12m 간격을 의미하며, 10m/s 속도에서는 `interval_s=1.2`가 되고, 고정된 `target_fps=3`는 간격당 3~4프레임을 제공합니다. 촬영은 시간 기반입니다. SDK에는 거리나 GPS로 트리거되는 모드가 없으므로, 측량 경로에서는 일정한 지상 속도를 유지하십시오.

**원본 데이터를 항공기에 저장하고 나중에 보정하십시오.** `interval_s`는 각 저장 시작 시점부터 측정되므로, 저장 시간이 간격보다 길면 다음 저장이 늦게 시작되어 간격이 어긋나게 됩니다. 원시 데이터와 광센서 측정값(`processing="raw"`; 측정값은 기본적으로 저장됨)을 함께 저장하면 각 저장 시간을 짧게 유지할 수 있으며, 나중에 모든 프레임을 반사율로 변환할 수 있게 됩니다.

```python
import chloros_sdk

INTERVAL_S = 1.2      # from the flight plan
FLIGHT_S = 20 * 60    # bounds the loop; it also stops at count= if given

with chloros_sdk.connect_array(serials, target_fps=3.0) as arr:
    groups = arr.capture_repeated(
        "survey/", interval_s=INTERVAL_S, duration_s=FLIGHT_S,
        processing="raw", render_index=False,
        on_capture=lambda i, r: print(f"group {i}: {len(r)} frames"))
```

- **`connect_array(target_fps=…)`**는 시작 시점부터 전송 속도를 고정하고,**`arr.set_target_fps(fps)`**는 재연결 없이 비행 도중 전송 속도를 변경하며,**`arr.set_target_fps(None)`**는 전송 속도를 자동 모드로 되돌립니다. 호스트와 네트워크가 처리할 수 있는 한도를 초과하는 전송 속도는 해당 상한선으로 낮아지며 — 결과에서 `clamped`를 확인하십시오 — 최저 전송 속도는 0.1 fps입니다.
- **프레임을 얇게 만들기 위해 `burst()`를 사용하지 마십시오.** 이 명령은 배열이 생성하는 모든 프레임을 기록합니다. 고정된 `target_fps`를 따르기는 하지만, 모든 프레임이 필요한 경우에만 적합한 도구입니다.
- **`target_fps`는 손상된 프레임을 수정하지 않습니다.** 이 값을 낮추면 생성되는 프레임 수가 줄어들 뿐, 각 프레임이 전송되는 속도는 줄어들지 않습니다. GVSP로 인해 손상된 프레임의 경우, 대신 `wire_ceiling_mbps` 값을 낮추십시오([어레이 상태](#array-health--which-subsystem-is-losing-frames) 참조).

```python
r = arr.set_target_fps(5.0)
if r["clamped"]:
    print(f"asked for {r['requested']} fps, running at {r['target_fps']}")
```

### 이미 연결된 어레이에 연결하기 — `attach_array`

어레이가 이미 가동 중인 경우 (GUI에서 열었거나, 이전 SDK 세션에서 `connect_array`를 호출한 경우), 재연결하는 대신 `attach_array`를 사용하여 해당 어레이의 핸들을 확보하십시오. `connect_array`는 해당 상황에서 항상 "Camera  <sn>is already in array<id>"라는 오류를 반환합니다. 이는 풀에 속한 멤버에 대해 `/array/connect`를 POST하는 것이 항등적이지 않기 때문입니다. `attach_array`는 `/api/camera/array/list`를 읽고 array_id 또는 일련번호를 기준으로 일치 여부를 확인합니다.

```python
import chloros_sdk

# By serials (matches if every serial is a member of one existing array)
arr = chloros_sdk.attach_array(
    ["213800234", "214000533", "214701288", "214701292"])

# By array_id (when you've already noted it down)
arr = chloros_sdk.attach_array("array-1779862544497")

# attach_array returns the same ArraySession as connect_array
arr.capture("output/", processing="reflectance")
```

패턴: 데스크톱 GUI와 공동 테넌트인 SDK 스크립트는 먼저 `attach_array`를 시도하고, 풀에 아직 배열이 없는 경우 `connect_array`로 대체해야 합니다.

```python
import chloros_sdk

try:
    arr = chloros_sdk.attach_array(serials)
except chloros_sdk.ChlorosConnectError:
    arr = chloros_sdk.connect_array(serials)
```

> **연결된 어레이는 소유된 것이 아니라 차용된 것입니다.** `attach_array`에서 반환된 핸들은 어레이를 절대 해제하지 않습니다. `with` 블록이 종료되더라도, 명시적인 `disconnect()` 호출을 통해 해제하더라도 어레이는 해제되지 않습니다(명시적인 호출 시 stderr에 “ 배열이 계속 실행 중이라고) 한 줄을 출력합니다). `CameraSession` / `DAQSensorSession`와 동일한 규칙입니다. 따라서 `with`의 유무와 상관없이 GUI와 함께 사용하는 것은 안전합니다:
>
> ```python
> with chloros_sdk.attach_array(serials) as arr:
>     arr.capture("output/", processing="reflectance")
> # … the array stays up for the GUI
> ```
>
> 정말로 종료하려는 경우 — 예를 들어 다른 `wire_ceiling_mbps`와 재연결하려는 경우 — `arr.force_disconnect()`를 호출하십시오. 연결된 핸들에 대해 단순한 `disconnect()`를 호출한 후 재연결을 시도하면, 아무것도 해제되지 않았기 때문에 "카메라가 <sn>이미 배열에 포함되어 있습니다<id>"라는</id> 오류가 <id>발생합니다. 이는 아무것도 해제되지 않았기 때문입니다.

### 네트워크 분석 도우미

배열을 열기 전에 유용하며, 제안된 설정이 적합한지 여부를 예측해 줍니다:

```python
result = chloros_sdk.analyze_array_network(
    master_serial="214701288",
    slave_serials=["213800234", "214000533", "214701162"],
    width=2048, height=1536,
    pixel_format="BayerRG12",
    binning=1,
)

if result["status"] == "ok":
    print("Use the requested settings.")
elif result["status"] == "auto_capped_fps":
    r = result["recommended"]
    print(f"Keep the resolution; cap the trigger rate at {r['recommended_target_fps']} fps")
elif result["status"] == "auto_shrunk":
    r = result["recommended"]
    # width / height are sensor pixels BEFORE binning (what connect_array
    # takes with binning=); out_width / out_height is what is streamed.
    print(f"Connect with width={r['width']} height={r['height']} "
          f"binning={r['binning']} ({r['out_width']}x{r['out_height']} streamed)")
elif result["status"] == "needs_force_slip":
    print("Sim-sync impossible on this wire; force_tier='slip-emit-and-capture' required")
```

```python
analyze_array_network(
    master_serial,
    slave_serials=None,
    width=2048, height=1536,              # sensor pixels before binning
    pixel_format=None,                    # None = what connect applies (Mono12 / BayerRG12)
    binning=1,                            # 1 | 2 | 4
    force_tier=None,
    packed="auto",                        # "auto" | "on" | "off"
    backend_url="http://127.0.0.1:5000",
    timeout=30.0,
) -> dict
```

백엔드를 절대 시작하지 않습니다. 응답하지 않는 백엔드는 `ChlorosConnectError` 오류를 발생시키는 반면, 요청이 거부된 경우에는 결과가 반환됩니다. 잘못된 `packed`는 `ValueError` 오류를 발생시킵니다.

`status`는 `ok` / `auto_capped_fps` / `auto_shrunk` / `needs_force_slip` 중 하나입니다(그 외의 경우 `error` — 거부된 요청 포함: 로그인되지 않은 상태, `error_code: AUTH_REQUIRED`, 또는 SDK 최저 한도 미만의 요금제, `PLAN_UPGRADE_REQUIRED`). `auto_capped_fps`는 요청된 해상도가 상한 트리거 속도에서만 RX 링에 적합함을 의미합니다 — 해상도를 유지하고 `target_fps=result["recommended"]["recommended_target_fps"]`에서 `connect_array`로 전달하십시오([예제 6](#6-capability-probe-before-connecting-a-4-cam-array) 참조). `auto_shrunk`는 요청된 프레임을 어떤 프레임 레이트로도 전달할 수 없는 경우에만 나타납니다. connect는 자체적으로 프레임을 축소하지 않으므로, `recommended`를 직접 적용해야 합니다.

**투영값 읽는 방법** (GUI의 ‘어레이 설정’ 패널과 동일한 모델):

- **버스트(`frame_bytes_total`)는 각 카메라의 실제 픽셀 형식에 따라 카메라별로 합산됩니다.**모노**M3M**카메라는 Mono12(2 B/px)로 스트리밍하거나, `pixel_format`를 적용하여 해당 카메라의 깊이 정보에 대한 모노 형식으로 스트리밍합니다 (`"BayerRG8"`는 `Mono8`이며, 1 B/px)의 모노 포맷으로 스트리밍합니다. 따라서 `pixel_format`가 없는 경우, 4대의 카메라로 구성된 풀 해상도 프레임은 3대의 모노 카메라를 사용할 때**~25 MB**가 되며, 모든 카메라가 8비트라고 가정했을 때의 ~12.6 MB와는 다릅니다. 백엔드는 각 카메라의 모델 정보를 바탕으로 해당 카메라의 포맷을 파악합니다.
- **입력 허용도(`burst_fits_nic_ring`)는 드레인 상태를 고려합니다**, 전체 버스트 대 링 방식이 아닙니다: 호스트가 RX 링을 캠이 채우는 속도보다 빠르게 비울 때 시뮬레이션-방출(sim-emit)이 적용됩니다. 10G 호스트 + 1 GbE 캠의 경우, 버스트가 링을 초과하더라도**전체 해상도를 허용**하며; 1GbE 호스트는 차단합니다(`needs_force_slip` / `auto_shrunk`).
- **`achievable_fps_max`는 보수적인 직렬 수신 상한치**입니다 — `max(readout+emit, N×emit)`는 카메라당 송신량을 1 GbE 카메라 링크에 맞춰 제한하며, 노출과 무관합니다. 예: 4대의 카메라로 구성된 풀 해상도 12비트 어레이의 경우 약 2.8 fps (런타임에서 측정된 약 2.7–3.0과 일치). 전체 모델: [CLI 참조 → 어레이 fps 및 버스트 모델](cli-reference.md#array-fps--burst-model).
- **연속 캡처가 끝난 후, 투영 결과는 캡처된 내용을 나타냅니다.** 자동 속도로 30초 이상 캡처하면 이 컴퓨터에서 해당 어레이 크기에 대해 안정화된 속도가 기록되며, 속도 제어기가 더 빠른 속도로 수행한 짧은 시험은 제외됩니다. 그 이후부터는 `array-connect`'줄부터, `connect_array()`의 분석 및 ‘배열 설정’ 패널은 해당 속도가 모델의 속도보다 낮을 경우 이를 인용하며, `cycle_limiter`에서 이를 제한한 요인을 다음과 같이 명시합니다: `"clock"` (어레이는 최종적으로 결정된 속도로 실행되었고, 그랩 루프는 트리거 클럭을 대기 중이었음), `"loop"` (그랩 루프에 남은 시간이 없었음), `"loop"`(그랩 루프에 남은 시간이 없음), `"capture"`(캡처가 뒤처져 그룹을 건너뛴 경우) 또는 `"measured"`(어떤 경우인지 명시되지 않음)에서 나타납니다. 모델의 수치는 `wire_fps_max`에 나와 있습니다. 배열은 여전히 전송선이 처리할 수 있는 수준까지 증가합니다.
- **과부하(`oversubscribed: true`)는 카메라가 도달하는 어떤 프레임 속도에서도 전송선이 배열을 처리할 수 없음을 의미합니다** — fps 필드(`achievable_fps_max` / `fps_bright` / `fps_dark`)는 0으로 표시되며, 비닝으로도 이 문제를 해결할 수 없습니다 (비닝은 프레임당 바이트 수를 줄일 뿐, 초당 페이싱 바이트 수는 줄이지 않기 때문입니다). 단순히 네트워크 회선 속도가 느리다고 해서 이런 현상이 발생하는 것은 아닙니다. 대신 프레임 속도가 떨어지게 되며, 카메라의 페이싱 속도를 충분히 낮출 수 없는 경우 티어(tier)가 `slip-emit-and-capture` (시리얼) 티어로 변경됩니다. 해결책으로는 카메라 수를 줄이거나, 점보 프레임을 사용하거나, 더 빠른 NIC를 사용하는 것이며, `max_cams_collision_safe`는 해당 회선 및 카메라의 상한값을 보고합니다. 응답에는 `aggregate_demand_bps`, `collision_safe_ceiling_bps` 및 `per_cam_floor_bps`도 포함됩니다. [과다 할당](#over-subscription-the-per-cam-floor)을 참조하십시오.

### 검색 및 나열

```python
chloros_sdk.discover_lattice_cameras()   # list all cams visible to the backend
chloros_sdk.list_cameras()               # cams currently in the pool
chloros_sdk.list_arrays()                # active arrays in the pool
```

각 함수는 키워드 전용 매개변수인 `backend_url=` 및 `timeout=30.0`를 받아 딕셔너리 목록을 반환합니다. `discover_lattice_cameras()` (`serial`, `ip`, `mac`, `model`, `lattice_model`, `vendor`, `connected`, `array_id`, …)도 `auto_start_backend=True`를 호출하지만, `list_*` 호출은 백엔드를 절대 시작하지 않습니다.

---

## 스마트 AE / 스마트 캡처

LATTICE 어레이는 연결되는 즉시 백그라운드에서 지속적인 AE를 실행하지만, 새로 조준된 장면은 수렴되는 데 잠시 시간이 걸립니다. **Smart-capture**는 이를 한 번에 처리해 주는 편리한 기능입니다. 각 카메라의 노출 값을 폴링하고, 어레이가 특정 창 전체에서 안정화될 때까지 기다린 후 캡처를 실행합니다. 이는 GUI와 동등한 기능으로, 데스크톱 앱의 “스마트” 캡처 버튼은 동일한 백엔드 엔드포인트를 호출합니다.

```python
import chloros_sdk

with chloros_sdk.connect_array([
        "213800234", "214000533", "214701288", "214701292"]) as arr:
    # Initial pose
    arr.capture("pose_a/", processing="reflectance", smart=True)
    input("Move the rig, then press Enter...")
    # New pose — smart-capture waits for AE to re-settle automatically
    arr.capture("pose_b/", processing="reflectance", smart=True)
```

`ChlorosProject`(다음 섹션)를 통해 구동할 경우 더 많은 설정 옵션을 사용할 수 있습니다:

```python
proj.arrays["main_rig"].capture_smart(
    output_dir="out/",
    processing="reflectance",
    settle_timeout_s=5.0,           # max wait
    stability_window_s=1.5,         # exposure must hold steady this long
    exposure_tolerance_pct=5.0,     # %-spread allowed within the window
)
```

스마트 AE 정책은 기본적으로 보수적으로 설정되어 있습니다. 정밀한 방사계 측정이 필요한 작업에서는 `exposure_tolerance_pct` 값을 더 좁게 설정하고, "대략적으로만" 측정하면 되는 빠르게 변화하는 장면에서는 더 넓게 설정하십시오.

---

## DAQ 센서 세션

스펙트럼 센서(USB 기반 DAQ-U, BLE 기반 DAQ-M, 이더넷 기반 DAQ-E)를 위한 영구 백엔드 풀입니다. 카메라 표면을 반영합니다: 스마트 감지, 풀 재사용, 항등적 연결.

### 스마트 감지(제로 구성)

```python
import chloros_sdk

with chloros_sdk.connect_daq_sensor() as daq:
    print(daq.model, daq.transport, daq.address)
    for frame in daq.latest(n=10):
        spectrum = frame["spectrum"]   # list[float] (W/m²/nm if calibrated)
        is_sat = frame["is_saturated"]
        x, y, z = frame["x"], frame["y"], frame["z"]
        print(len(spectrum), is_sat)
```

우선순위: 이더넷 → BLE → USB. 전송 방식을 고정하려면 명시적인 힌트를 하나 전달하십시오.

### 고정된 전송 방식

```python
# DAQ-U on a specific serial port
daq = chloros_sdk.connect_daq_sensor(transport="usb", port="COM3")

# DAQ-M over BLE by MAC (implies transport="ble")
daq = chloros_sdk.connect_daq_sensor(mac="AA:BB:CC:DD:EE:FF")

# DAQ-E over Ethernet by hostname (implies transport="eth")
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-xxx.local")

# A specific sensor by id (as list_daq_sensors() shows it; ':' or '-' alike)
daq = chloros_sdk.connect_daq_sensor(sensor_id="CB-7C-A7-5A-73")

# Tuning knobs
daq = chloros_sdk.connect_daq_sensor(
    port="COM3",
    integration_time=64,      # ms
    frame_avg=20,
    enable_ae=True,
    start_streaming=True,
)
```

`sensor_id=`는 하나의 센서를 지정하며, 해당 센서만 열립니다. 이미 열려 있는 센서들과 대조한 후, DAQ-E의 호스트명 또는 IP(네트워크 스캔에서 IP로만 식별되는 DAQ-E의 경우, 이름, MAC 및 일련번호를 읽기 전용으로 요청하며, 따라서 이 중 어느 것으로도 해당 센서를 찾을 수 있음), DAQ-M의 BLE MAC, 그리고 DAQ-U 또는 DAQ-M이 열렸을 때 보고하는 ID(검사를 위해 열린 다른 센서는 다시 닫힙니다)와 대조됩니다. 해당 센서가 발견되지 않으면, `connect_daq_sensor`는 발견된 항목을 나열하는 `ChlorosConnectError`를 발생시키며, 주소도 함께 포함합니다(다른 센서를 보유한 `port=` / `mac=` / `eth_host=`)가 다른 센서를 포함하고 있는 경우에도 예외가 발생합니다. 이 코드는 표시된 센서 중 어느 것으로도 대체되지 않습니다. 바로 이것이 제로 플래그 호출의 목적입니다.</id></sn></id></sn>

### `connect_daq_sensor()` 시그니처

```python
connect_daq_sensor(
    *,                                    # every argument is keyword-only
    sensor_id=None,                       # one named sensor (see above)
    transport=None,                       # "usb" | "ble" | "eth"; None = smart-detect
    port=None,                            # DAQ-U serial port ("COM3", "/dev/ttyUSB0")
    mac=None,                             # DAQ-M BLE MAC (implies transport="ble")
    eth_host=None,                        # DAQ-E hostname or IP (implies transport="eth")
    integration_time=None,                # ms; None = the sensor's own
    frame_avg=None,
    enable_ae=None,
    cap_id=None,                          # cap correction: "sunshine_cosine", "fov_30", ..., "none" (bare)
    start_streaming=True,
    backend_url="http://127.0.0.1:5000",
    timeout=60.0,
    auto_start_backend=True,              # spawn a local backend if none is running
) -> DAQSensorSession
```

이미 풀에 있는 센서는 있는 그대로 반환됩니다(차용됨): `integration_time`, `frame_avg` 및 `enable_ae`는 해당 센서에 적용되지 않습니다. 사용자 설정으로 열려면 먼저 연결을 해제하십시오. `cap_id`는 예외입니다: 차용된 센서에서는 `set_cap()`로 설정됩니다. 생략할 경우 백엔드의 기본값이 적용됩니다(메서드 표 아래의 참고 사항 참조).

### `DAQSensorSession` 메서드

| 메서드 | 설명 |
| --- | --- |
| `status(timeout=10.0)` | 풀 항목 요약 (스트리밍/기록 상태, 파장 범위, 보정 SHA, 적분 시간, frame_avg, AE 상태). 스트림이 중단된 DAQ-E의 경우: `reconnect` (백엔드의 자동 재연결이 작동 중인 경우: `state`, `attempts`, `next_attempt_in_s`, `last_error`; 스트리밍 중에는 `None`), `stream_lost_s`, `reconnects` 및 `last_reconnect`, 그리고 진행 중인 녹화에 대해서는 `record_gaps` 진행 중인 녹화용; `rx_stream_fault`는 첫 번째 스펙트럼이 도착할 때까지 재연결 상태를 나타냅니다. |
| `latest(*, n=1, timeout=10.0)` | 가장 최근의 스펙트럼 프레임 중 최대 N개까지 반환합니다(각각 `ts_ns`, `spectrum`, `is_saturated`, `integration_time`, `x`, `y`, `z`). |
| `stream_start()` / `stream_stop()` | 스트리밍 재개 / 일시 중지 (핸들은 열린 상태로 유지됨). |
| `record_start(*, output_dir=None, device_name=None, timeout=15.0)` | .daq 파일 녹화 시작. 파일 경로를 반환합니다. AWS 교정 번들이 없는 DAQ-U/M의 경우 거부됩니다(DAQ-E는 예외). |
| `record_stop()` | 녹화 중지. `{path, rows, warning, gaps}`를 반환합니다. `warning`는 정상적인 녹화 시 `None`와 동일하며, 그렇지 않은 경우 백엔드의 진단 메시지(예: 빈 파일 또는 스트림의 공백)를 반환하며, 이 메시지는 stderr에도 출력됩니다. `gaps`는 스트림에서 데이터가 전달되지 않은 각 구간을 나열합니다: `start_utc`, `end_utc` (`None`: 중지 시점에 여전히 손실됨), `duration_s`, `reconnected`, `marked_in_file`. 연속 기록의 경우 이 항목은 비어 있습니다. |
| `set_cap(cap_id, timeout=15.0)` | 다음 측정값부터 적용될 캡 보정 프로필을 설정합니다: `"sunshine_cosine"`, `"fov_15"` ... `"fov_90"`, 또는 베어 센서의 경우 `"none"` (혹은 `None`). 센서의요약 정보를 반환하며, `cap_id`가 이를 다시 읽어옵니다. DAQ-E-S는 요청된 내용에 관계없이 `as_recorded`를 유지합니다(디퓨저가 내장되어 있음). 프로파일이 CLI: `chloros-cli daq pool-set-cap`. |
| `enable_ptp(timeout=15.0)` | DAQ-E 전용: PTP가 꺼져 있는 장치의 경우 PTP를 다시 켭니다 (`status()["rx_ptp_enabled"]`는 `False`와 동일하며, 설정은 장치에 저장되므로 다시 켜기 전까지 비활성화 상태로 유지됨). 갱신된 상태를 반환합니다. DAQ-U / DAQ-M의 경우, 또는 장치가 확인하지 않을 때 `ChlorosConnectError`를 발생시킵니다. Chloros는 꺼짐 상태가 의도적인 것일 수 있으므로, 절대로 장치를 스스로 다시 켜지 않습니다. CLI: `chloros-cli daq pool-enable-ptp`. |
| `device_info(timeout=15.0)` | DAQ-E 전용: 장치 자체 설정. `firmware` (`running`, `bundled`, `update_available`), `ip` 및 `ip_mode` (`dhcp` / `linklocal` / `static`: 현재 실행 중인 설정), `network` (저장된 설정: `mode`, 그리고 `ip` / `mask` / 정지 상태일 때 `gateway`), `network_change_pending` (장치가 저장된 설정(`set_network(..., reboot=False)`가 포함된 설정)을 실행할 때까지 유효), `auto_stream`, `ptp`. `firmware["update_available"]`는 장치의버전을 읽을 수 없을 때 `firmware["update_available"]`가 됩니다. 장치의 스트림이 이 컴퓨터에 도달하지 않는 동안 새로운 연결에서 읽히면, 사전에는 `stream_silent` (`True`), `stream_fault` 및 `reconnect`를 추가합니다. 응답하지 않는 장치의 경우 CLI: `pool-info`가 발생합니다. |
| `set_network(mode="auto", *, ip=None, mask=None, gateway=None, reboot=True, force=False, comeback_timeout=45.0)` | DAQ-E 전용: DHCP (`"auto"`) 또는 고정 IP 주소. 설정을 적용하고 장치를 다시 연결하려면 장치를 재시작해야 합니다. 이 세션은 계속 작동하며, `address`는 장치를 따라갑니다. `reboot=False`는 설정만 저장합니다. [DAQ-E 장치 설정](#daq-e-device-settings)을 참조하십시오. CLI: `pool-set-network`. |
| `set_auto_stream(enabled, timeout=15.0)` | DAQ-E 전용: 소프트웨어가 연결되지 않은 상태에서 전원을 켤 때 스트리밍을 시작합니다(장치에 저장됨). 확인된 설정을 반환합니다. CLI: `pool-set-auto-stream`. |
| `reboot(*, comeback_timeout=45.0)` | DAQ-E 전용: 장치를 재시작하고 이전 설정 상태로 다시 연결합니다. CLI: `pool-reboot`. |
| `update_firmware(*, comeback_timeout=45.0)` | DAQ-E 전용: 이전 버전을 실행 중인 장치에 이 Chloros 릴리스에 포함된 펌웨어를 플래시한 후 다시 연결합니다(약 1분 소요; 전원을 끄지 마십시오). `updated`, `fw_before`, `fw_after`, `bundled` 및 `warning`가 반환됩니다. 단, 장치가 응답이 느리거나 다른 버전으로 복귀한 경우; 이미 최신 버전을 사용 중인 장치의 경우 `updated`는 `False`로 표시됩니다. 버전을 읽을 수 없는 장치의 경우 `ChlorosConnectError`를 발생시킵니다. CLI: `pool-update-firmware`. |
| `disconnect()` | 풀에서 해제합니다.소유하지 않은 핸들의 경우 해당 연산자가 없습니다. 이 세션에서 연 센서의 경우 `force_disconnect()`이며, 해제 확인이 되지 않을 때 `ChlorosConnectError`를 발생시킵니다(`with` 블록의 종료 시에는 는 절대 예외를 발생시키지 않음). |
| `force_disconnect(timeout=30.0)` | 누가 열었는지와 관계없이 센서를 해제합니다 — 예를 들어, 스크립트 충돌로 인해 풀에 남아 있는 센서의 경우, 새로운 스크립트는 `disconnect()`가 아무 작업도 수행하지 않는 연결된 핸들로서만 해당 센서에 접근합니다. 다른 클라이언트의 센서를 종료하면 해당 센서가 수행 중인 모든 기록이 중단됩니다. 백엔드가 해제를 확인하거나 해제할 대상이 없을 때 (센서가 풀에 없거나 백엔드가 실행 중이지 않은 경우); 타임아웃이 발생했거나 백엔드가 해제를 거부한 경우 `ChlorosConnectError`를 발생시킵니다. CLI: `chloros-cli daq pool-disconnect --sensor-id ID`. |

속성: `sensor_id`, `model`, `transport`, `address`(`set_network()`가 주소를 재지정한 DAQ-E에 뒤따름), `info` 및 `backend_url`.

> **캡의 이름을 지정하십시오.** DAQ-U, DAQ-M 및 DAQ-E는 커패시터 없이 출하되며, Chloros는 커패시터가 장착되었는지 여부를 확인할 수 없습니다. `cap_id` 없이 연결된 센서는 `sunshine_cosine`를 기록하는데, 기록 내용에 따르면 이는 (`cap_id_source=auto_default`). 이 보정값을 적용했을 때 캡이 없는 DAQ-U는 약 30배 높은 값을 읽으므로, 캡이 없는 센서의 경우 `cap_id="none"`를 전달하고 (또는 `set_cap("none")`)를 통과시키고, 그렇지 않은 경우에는 장착한 캡의 프로필을 확인하십시오.

### DAQ-E-S — 고정형 디퓨저 및 틸트 센서

**DAQ-E-S**는 소프트웨어 측면에서 두 가지 중요한 차이점이 있는 DAQ-E 모델입니다. 그 외 모든 기능 — 검색, 연결, PTP, 기록, 아래의 장치 설정 — 은 DAQ-E와 정확히 동일하게 작동합니다.**이 모델의 디퓨저는 고정형이므로 캡 보정이 필요하지 않습니다.** 선샤인(Sunshine) 스타일의 디퓨저는 내장되어 있으며, 공장 교정 측정 시 이미 장치에 장착된 상태였으므로, 해당 교정값은 이미 정확한 하향 복사 조도를 나타냅니다. Chloros는 요청된 내용이 무엇이든 상관없이 캡을 `as_recorded`(프로파일 없음)로 설정합니다. 이 장치 이름을 딴 캡(`sunshine_cosine`, `fov_*`, `none`)는 메시지와 함께 무시되며, 기록 데이터에는 `cap_id_source=model`가 저장되므로 이 기기에 대해서는 가정된 캡 경고가 절대 발생하지 않습니다. 데이터에 수동으로 상한값을 적용하지 마십시오: 상단에 `sunshine_cosine`를 적용하면 측정값이 약 11배 높게 나옵니다.

**이 센서에는 3축 가속도계가 탑재되어 있습니다.** 선샤인 디퓨저의 코사인 응답은 수평을 유지할 때만 유효하므로, 센서가 기울어져 있으면 잘못된 조도 값을 보고하게 되며 — 반사율도 이 오류를 그대로 반영하게 됩니다 — 다른 증상은 나타나지 않습니다. 따라서 각 DAQ-E-S 측정값에는 해당 센서의 자세 정보가 포함됩니다:

| 필드 | 의미 |
| --- | --- |
| `tilt_deg`, `roll_deg`, `pitch_deg` | 각도(도 단위). 펌웨어가 각도를 정의되지 않음(자유 낙하, 축 고장)으로 표시할 때 `None`. |
| `x_mg`, `y_mg`, `z_mg`, `mag_mg` | 밀리-g 단위의 가속도. `mag_mg` 값이 ~1000에서 크게 벗어난 경우, 장치가 이동 중이었으며 해당 샘플에 대한 각도 값은 의미가 없음을 의미합니다. |
| `cal_applied` | 설정됨: 축과 각도가 디퓨저 좌표계에 기반합니다(장치의 장착 보정이 적용됨). 해제됨: 원시 보드 좌표계로, 실제와 다른 최대 ~2.3°의 기울기가 표시될 수 있습니다. |
| `sample_age_ms` | 스펙트럼이 전송되었을 당시 가속도계 샘플의 경과 시간. |
| `temp_raw` | 보정되지 않은 다이 온도 **추세** — 차이값만 표시되며, 절대 온도계 측정값이 아닙니다. |

`DAQSensorSession.latest()`는 각 프레임의 `imu` 키(DAQ-U, DAQ-M 또는 일반 DAQ-E의 경우 `None`) 아래에 해당 값을 반환합니다. DAQ-E-S에서 기록된 `.daq`는 동일한 값을 `imu_*` 열(`imu_tilt_deg`, `imu_roll_deg`, `imu_pitch_deg`, `imu_mag_mg`, `imu_cal_applied`, …)에 동일한 값을 저장하므로, 독자는 구름과 삼각대 충격으로 인한 오류를 구분할 수 있습니다. 다른 모든 모델에서는 해당 열이 비어 있습니다.

자세 스트림은 `status()` (`rx_imu_stream_fault`, `rx_imu_stream_faults`, `rx_last_imu_stream_fault`)의 자체 필드를 보고하며, 이는 스펙트럼 필드와 분리되어 있어 가속도계 오작동이 기록의 공백으로 오인되는 일이 없도록 합니다. `discover_daq_sensors()` 및 세션의 `model`는 `DAQ-E-S`를 나타냅니다.

### DAQ-E 장치 설정

```python
with chloros_sdk.connect_daq_sensor(eth_host="daq-e-def330.local") as daq:
    info = daq.device_info()
    print(info["ip"], info["ip_mode"], info["firmware"])

    # A fixed address on the network this computer is on
    daq.set_network("static", ip="192.168.1.50", gateway="192.168.1.1")
    print(daq.address)              # 192.168.1.50: the session follows it

    if info["firmware"]["update_available"]:
        daq.update_firmware()
```

- **이 컴퓨터의 네트워크에 고정 IP 주소가 있어야 합니다.** `set_network`는 서브넷에 이 컴퓨터의: 재시작 후에는 센서에 접근하여 설정을 원래대로 되돌릴 수 없게 됩니다. 이 컴퓨터가 새 서브넷으로 재할당되거나 라우팅될 때만 `force=True`를 통과시키십시오.
- **`set_network`, `reboot` 및 `update_firmware`는 센서를 재시작하며**, 기록 중에 `ChlorosConnectError`를 발생시키고, `comeback_timeout` 초 이내에 복귀하지 않을 경우 센서를 재시작합니다. 센서 ID와 `daq-e-<id>.local` 호스트명은 절대 변경되지 않습니다.
- **재시작 직전에 시작된 기록은 재시작 과정에서 종료되며, 파일이 생성됩니다.** 센서가 다시 연결되면, 반환된 딕셔너리에는 `recording_closed`(파일 경로) 및 이를 알리는 `warning`를 포함합니다. 센서가 다시 연결되지 않거나 펌웨어 전송에 실패하면, 대신 `ChlorosConnectError`가 이를 알립니다: 해당 메시지는 경고로 끝나며, `e.recording_closed` / `e.warning`가 설정됩니다(녹화가 종료되지 않은 경우 두 개 모두 `None`). 센서가 복구되면 녹화를 다시 시작하십시오.
- **센서 자체만 재연결됩니다.** 재시작 후 장치는 원래 위치에 있었음을 응답해야 하며 -- 그리고 `set_network("auto")` 이후에는 `.local` 이름으로도 -- 이전에 사용하던 네트워크 어댑터(MAC)를 통해 이름, 호스트명 및 모델을 응답해야 합니다; `address`는 응답한 IP가 됩니다. 이름으로 이루어진 응답은 해당 MAC 주소로만 유효하므로, 라우터를 통해 센서에 접속하는 컴퓨터나 WSL2의 기본 NAT 네트워킹 환경에서 DHCP로 주소가 변경된 센서는 재연결되지 않습니다 (`ChlorosConnectError`; 새 주소로 다시 연결해야 함).
- **펌웨어 업데이트를 수행하려면 센서가 다시 연결되어야 합니다.** `update_firmware()`가 장치를 호출하면, 장치는 OS가 선택한 포트를 통해 이미지를 받기 위해 이 컴퓨터로 TCP 연결을 다시 엽니다. 백엔드는 장치에 연결된 어댑터에서 수신 대기하며(어느 어댑터인지 식별할 수 없는 경우 모든 어댑터에서 대기), 해당 장치에만 서비스를 제공합니다: 대기 중인 동안 다른 모든 연결은 거부됩니다(백엔드 출력에서 `[DAQ-FW] … refused a firmware connection from <ip>`). 연결을 차단하는 방화벽은 `device never connected back on TCP <port>` 오류와 함께 업데이트를 실패시킵니다. Windows 설치 프로그램은 이에 대한 규칙을 추가합니다([1단계](#windows-exe) 참조). 다른 방화벽의 경우, 센서의 서브넷에서 백엔드로 들어오는 TCP 연결을 허용해야 합니다.
- **WSL2 환경에서는 미러링 네트워킹을 사용하십시오** (`networkingMode=mirrored` 및 `.wslconfig`): 기본 NAT 모드에서는 모든 고정 주소가 거부되며(WSL 자체 주소가 센서 네트워크에 속하지 않음), 스트림이 WSL에 도달하지 않아 펌웨어 업데이트가 완료되지 않습니다.
- GUI는 센서와 별도의 연결을 유지합니다: 스크립트를 통해 재시작하면 GUI의 센서 세션이 종료됩니다. GUI에는 동일한 설정(‘Light Sensors’ 탭, 센서 설정, ‘Network’)이 적용됩니다.

### 검색 — 연결할 주소 찾기

`discover_daq_sensors()`는 USB / BLE / ETH를 스캔하여 *열 수 있는* 센서를 찾습니다. 이는 `discover_lattice_cameras()`의 DAQ 대응 기능이며, **DAQ-M의 BLE MAC**을 얻는 유일한 방법입니다 — DAQ-E는 호스트명이 있고 DAQ-U는 COM 포트가 있지만, MAC 주소는 기기에 표시되지도 않고 OS에도 나열되지 않습니다.

```python
for s in chloros_sdk.discover_daq_sensors():
    print(s["transport"], s["address"], s["model"], s["extra"])
# ble  C3:D8:85:E0:0A:19  DAQ-M  {'name': 'NSP32_SPECTRUM'}
# usb  COM3               DAQ-U  {'vid': '0403', 'pid': '6001', 'manufacturer': 'FTDI'}

# `address` is exactly what connect_daq_sensor wants:
for s in chloros_sdk.discover_daq_sensors(transports=["ble"]):
    if s["model"] == "DAQ-M":
        daq = chloros_sdk.connect_daq_sensor(mac=s["address"])
```

| 필드 | 설명 |
| --- | --- |
| `transport` | `usb` \| `ble` \| `eth`. |
| `address` | COM 포트 / BLE MAC / 호스트 이름 — `connect_daq_sensor`로 전달됨 (`port=` / `mac=` / `eth_host=`. |
| `display` | 사람이 읽을 수 있는 레이블. |
| `model` | `DAQ-U` \| `DAQ-M` \| `DAQ-E` \| `DAQ-E-S`, 또는 `None`. 이더넷 센서는 `DAQ-E-S` 또는 `DAQ-E`이며, 스캔 중에 요청 시 자체를 보고하며, 읽기 전용이며 (제시간에 응답하지 않을 경우 `DAQ-E`). USB 시리얼 포트의 브리지 칩이 DAQ-U에서 사용하는 칩(FTDI, CH340, CP210x, Prolific)인 경우, 해당 포트는 `DAQ-U`로 인식됩니다. 이는 포트 설명에서 알 수 있는 힌트이며, 프로브가 아님 — 이 외의 모든 USB 장치의 경우 `None`가 표시되며, 이 값은 여전히 목록에 포함되어 있으므로 `port=`로 전달할 수 있습니다. USB 벤더 ID가 없고 해당 브리지 칩이 없는 시리얼 포트(Linux의 `/dev/ttyS*`와 같은 마더보드 UART, 블루투스 시리얼 링크, Intel AMT 등)는 DAQ-U가 될 수 없으므로 목록에 포함되지 않습니다. |
| `extra` | 전송 방식별 세부 정보 (BLE 광고 이름; USB `vid`, `pid`, `manufacturer`, `serial_number`; DAQ-E ip/fw/…). 빈 값은 생략됩니다. |

| 매개변수 | 기본값 | 설명 |
| --- | --- | --- |
| `transports` | 세 가지 모두 | 스캔을 제한하는 시퀀스(또는 CSV 문자열). 원하는 값을 알고 있을 때 전달하는 것이 좋습니다 — BLE는 속도가 느린 편입니다. |
| `scan_timeout` | 5 | 전송별 스캔 창(초 단위); 백엔드에서 1~20 사이로 제한합니다. |
| `timeout` | 60.0 | 전체 호출에 대한 HTTP 상한값(SDK의 다른 부분과 동일). |
| `auto_start_backend` | `True` | 실행 중인 로컬 백엔드가 없으면 로컬 백엔드를 생성합니다. 원격 `backend_url`의 경우 절대 생성하지 않습니다. |

> **풀에 이미 열려 있는 센서는 표시되지 않습니다.** 연결된 BLE 주변기기는 광고를 중단하며, 열린 COM 포트는 탐색할 수 없으므로, 디스커버리에서는 *연결 가능한* 항목만 나열합니다. 무언가를 연결한 직후에는 결과가 비어 있는 것이 정상입니다. 이미 보유한 장치에 대해서는 `list_daq_sensors()`를 사용하십시오. 스캔을 실행할 수 없는 전송 프로토콜(bleak/zeroconf 미설치)은 오류를 발생시키지 않고 건너뛰므로, 블루투스가 없는 컴퓨터에서도 USB 및 ETH 응답을 받을 수 있습니다.

### 리스팅

```python
for s in chloros_sdk.list_daq_sensors():
    print(s["sensor_id"], s["model"], s["transport"], s["wavelength_range"])
```

### 어떤 DAQ가 어떤 카메라에 신호를 전송하는지 — `daq_bindings()`

캡처의 반사율과 이를 저장하는 `.daq`는 해당 카메라나 어레이에 바인딩된 DAQ를 사용하며, 그렇지 않은 경우 백엔드가 보유한 가장 최신 DAQ를 사용합니다. GUI의 광센서 선택 및 `chloros-cli daq pool-bind`가 이를 설정하며; 다음도 마찬가지입니다:

```python
chloros_sdk.set_daq_bindings(cameras={"214701281": "COM7"},          # a sensor id or an address
                             arrays={"array-1779862544497": "daq-e-def330"})
chloros_sdk.set_daq_bindings(cameras={"214701281": None})            # None clears it
for serial, b in chloros_sdk.daq_bindings()["cameras"].items():
    print(serial, b["binding"], b["source"], b["resolved"]["in_pool"])
```

`daq_bindings()`는 `{"cameras": {serial: b}, "arrays": {array_id: b}}`를 반환합니다: 각 `b`는 `binding`(설정된 값), `source`(`project`, 프로젝트의 `sensors.json` 또는 `runtime`에서 가져온), `known` / `known_as` (카메라나 배열 키가 Chloros가 알고 있는 이름을 지칭하는지, 그리고 그 방식: `connected`, `project` — 열려 있는 프로젝트의 `cameras.json` — 또는 카메라의 경우, `seen`: 이 컴퓨터에서 이전에 열렸던 것; `known: False`는 아무것도 지정하지 않은 키) 및 `resolved` (현재 매핑된 풀 센서: `in_pool`, `sensor_id`, `address`, `model`, `streaming`). 런타임 바인딩은 프로젝트; 프로젝트 파일은 변경되지 않습니다. 카메라, 어레이 또는 DAQ의 연결을 해제하면, 누가 설정했든(어레이의 경우 어레이 구성원들의 설정 포함) 해당 하드웨어에 대한 모든 바인딩이 해제됩니다.

`set_daq_bindings()`는 연결된 카메라, 열려 있는 프로젝트에 포함된 카메라, 또는 이전에 이 컴퓨터에서 열렸던 카메라와, 연결된 어레이 또는 열려 있는 프로젝트에 포함된 어레이(혹은 이미 바인딩된 키)에만 바인딩됩니다. 그 외의 키는 `ChlorosConnectError`를 발생시키며, (`400`, `code`, `UNKNOWN_BINDING_TARGET`, `unknown` 키와 함께), 그리고 해당 호출에서는 **아무것도** 바인딩되지 않습니다. `None`는 모든 키를 지웁니다.

### 주변광 + 대상 반사율 — `light_sensor_reflectance()`

하나는 하늘을 향하고(주변광), 다른 하나는 대상물을 향하는(대상) 두 개의 통합된 DAQ는 최신 측정값을 바탕으로 대상물의 반사율과 식생 지수를 제공합니다: GUI의 ‘주변광 + 대상’ 쌍과, 센서를 두 번 열지 않고도 `chloros-cli daq reflectance`를 통해 확인할 수 있습니다. 두 DAQ 모두 연결되어 있고 데이터를 스트리밍 중이어야 합니다.

```python
r = chloros_sdk.light_sensor_reflectance("daq-e-def330", "daq-e-54b774")
print(r["reflectance_type"])                       # "Apparent Reflectance" when both are calibrated
for name, band in r["bands"].items():
    print(name, band["rho"] if band["valid"] else band["reason"])
for idx in r["indices"]:
    print(idx["name"], idx["value"])
```

```python
light_sensor_reflectance(
    ambient_sensor_id, object_sensor_id, *,
    bands=None,          # [{"name", "center_nm", "fwhm_nm", "role"}]; role blue / green / red / nir
    preset=None,         # a preset id from daq_band_presets(); default: the suggested one, else "GENERIC"
    expressions=None,    # custom indices: [{"name": "MyNDVI", "expr": "(NIR_850 - Red_660) / (NIR_850 + Red_660)"}]
    builtins=True,       # include the built-in indices
    backend_url="http://127.0.0.1:5000", timeout=30.0,
) -> dict
```

- **밴드.** 각 밴드의 반사율은 대상의 밴드-에 대한 대상의 밴드 적분 조도입니다. `daq_band_presets(filter_code=None)`에는 밴드 세트가 나열되어 있습니다: LATTICE M3C 필터(`RGN`, `OCN`, `NGB`, 카메라 자체의 밴드 모델이므로 DAQ NDVI와 카메라 NDVI는 동일한 값을 가집니다) 및 `GENERIC` 광대역입니다. 이 중 `suggested`는 `filter_code`와 일치하는 사전 설정값이며, 그렇지 않은 경우 가장 먼저 연결된 카메라의 필터 (`suggested_from`가 어느 것인지 나타냄).
- **반환값** `bands` (대역별 `rho`, `valid`, `reason`, `e_ambient`, `e_object`), `indices` (내장-인), `custom` (각각 `value` 또는 `valid: False` 및 `error`와 함께), `invalid_bands`, `reflectance_type`, `preset` 및 두 측정값인 `timestamps` (`ts_ns`)를 포함합니다. 주변 센서가 측정할 수 없는 범위(측정 범위 외 또는 노이즈 플로어 수준)의 데이터는 무효로 처리되며, 그 사유가 명시되고, 해당 데이터가 필요한 모든 지수는 제외되며, 절대 0을 기준으로 계산되지 않습니다.
- **겉보기 또는 상대적.** 스케일에 의존하는 지수(SAVI, OSAVI, EVI, MSAVI2, RDVI, TDVI, LAI, NLI, MNLI, FCI, GEMI)는 절대 비율이 필요하므로, 두 센서 모두 전력 보정된 경우에만 산출됩니다 (각각 `calibration_bundle_sha`를 가짐): `"Apparent Reflectance"`. 그렇지 않은 경우 결과는 `"Relative Reflectance"`이며, 이때는 스케일 불변 지수만 포함됩니다.
- **두 개의 서로 다른 파장 그리드**는 중첩 영역에서 비교됩니다: 주변 센서의 측정값은 물체의 측정 범위 내에 위치하며, 물체의 스펙트럼은 해당 위치로 보간됩니다.
- **센서는 반드시 두 개여야 하며, 절대 하나가 되어서는 안 됩니다.** 하나의 센서를 두 개로 지정할 경우(풀이 허용하는 어떤 표기법이든 동일한 ID를 사용하는 경우) `ChlorosConnectError` 오류가 발생합니다. 센서가 스스로를 측정할 때 모든 대역에서 1.0이 나오며, 이는 아무것도 측정하지 않은 것과 같습니다. 백엔드에서도 이를 거부합니다(`POST /api/daq/indices`는 `400`에 응답하고, `code`는 `SAME_SENSOR`에 응답하지만, SDK가 전송하는 `ambient_sensor_id`와 `object_sensor_id`는 하나의 풀링된 센서, 즉 ID와 포트)를 나타냅니다.

### CLI 및 기타 스크립트와의 공동 사용

백엔드의 DAQ 센서 풀은 해당 머신상의 모든 CLI 및 SDK 클라이언트가 공유합니다. 센서가 이미 풀에 존재할 경우(`chloros-cli daq pool-connect` 또는 다른 스크립트에 의해 열려 있는 경우), Python에서 `connect_daq_sensor(port="COM3")`를 호출하면 차용된 핸들이 반환됩니다: 해당 센서의 `disconnect()`는 아무 작업도 수행하지 않으므로, 스크립트가 스코프 종료 시 해당 센서를 열었던 클라이언트로부터 센서를 강제로 빼앗지 않습니다. `sensor_id=`(`:` 또는 `-`로 입력, 둘 중 하나와 일치)의 경우에도 마찬가지이며, 이는 호스트명으로 열렸을 때 IP 주소로 명명된 DAQ-E의 경우, 호스트명으로 열렸을 때 IP로 명명된 경우(또는 그 반대의 경우)와, 백엔드가 이미 보유하고 있던 센서로 응답할 때의 제로 플래그 스마트 감지 기능에도 동일하게 적용됩니다. `with` 블록은 오직 스스로 연 센서만 해제합니다.

Chloros GUI는 이 풀의 일부가 **아닙니다**: 해당 DAQ 패널은 센서에 대한 자체 연결을 실행하므로, `connect_daq_sensor`는 GUI가 열어둔 센서를 절대 빌리지 않습니다. GUI와 스크립트에서 동시에 동일한 센서를 사용하지 마십시오. GUI가 보유하고 있는 DAQ-U 또는 DAQ-M은 일반적으로 열리지 않지만, 반면 DAQ-E는 두 번째 세션을 허용합니다. 스크립트의 연결 명령은 GUI 하에서 장치의 수집 설정을 변경하며, 해당 `disconnect()`는 장치의 스트림을 중지시켜 GUI의 라이브 뷰와 진행 중인 모든 녹화를 종료시킵니다. 먼저 GUI에서 센서를 닫으십시오.

### PTP 그랜드마스터 — `time_sync_status()`

백엔드는 LATTICE 카메라와 DAQ-E 장치가 타임스탬프를 가져오는 PTP 그랜드마스터를 실행합니다. 이 기능은 요청 시 시작되며, `connect_camera()`, `connect_array()`, `connect_daq_sensor()` 및 `connect_all()`는 PTP 그랜드마스터가 실행 중이지 않을 경우 이를 시작합니다 — 예를 들어, 존재하지 않는 DAQ-E에 연결을 시도하는 `connect_all()`와 같이 연결이 실패하는 경우를 포함하여 — 또한 `test_array_sync()` 및 하나의 명령을 위해 카메라를 여는 모든 호출 (`capture_cameras_transient()`, `start_multi_camera_stream()`, 정렬 호출, `capture_live_index()`).

- **알림 위치.** 카메라와 연결된 유선 네트워크 어댑터(`169.254.x.x` 링크 로컬 주소 포함)에서 발생하며, 해당 어댑터가 유선 연결된 경우 기본 경로 어댑터에서도 발생합니다. Wi-Fi나 가상 어댑터(Tailscale, WSL)에서는 절대 발생하지 않습니다. 해당 어댑터 뒤에는 카메라나 DAQ-E가 없기 때문입니다. 해당 어댑터가 활성화되어 있지 않으면, 아무것도 알리지 않은 상태로 실행되며(`health` `NO_SOCKETS`), 다음 카메라나 DAQ가 연결되어 어댑터가 활성화되면 해당 어댑터에 바인딩됩니다.
- **링크가 끊긴 후.** 카메라 어댑터의 링크가 끊었다가 다시 연결되면(케이블 뽑기, 스위치 교체, USB 어댑터 재인식 등), 그랜드마스터는 재시작 없이 스스로 소켓을 재구축하고, 연결된 모든 DAQ-E를 재확인하여, 연결이 끊긴 상태로 남아 있던 장치를 다시 활성화합니다. 또한, 서비스 중이던 모든 카메라와 DAQ-E가 15초 동안 응답이 없을 경우에도 소켓을 재구축합니다.
- **케이블 연결 상태는 정상인가요?** `time_sync_status()`(`GET /api/time-sync/status`)를 참조하십시오. `state`는 그랜드마스터 스레드가 실행 중인지 여부만 표시하며, `health`는 패킷이 전송되는지 여부를 나타냅니다: `OK`(오류 없음), `STOPPED`, `NO_SOCKETS`, `SENDS_FAILING`, `DEGRADED`(일부 어댑터 오류) 또는 `PEERS_SILENT`(서비스 대상인 카메라 및 DAQ-E가 응답을 중단함), 그리고 `health_detail`에 문장이 표시됩니다. `announce_sent` / `sync_sent` / `delay_resps_sent`는 전송된 메시지만 집계하며; `announce_send_errors`, `sync_send_errors`, `follow_up_send_errors`, `delay_resp_send_errors`, `last_send_error`, `failing_interfaces` 및 `last_delay_req_age_s`는 전송에 실패한 메시지를, `rebinds` / `last_rebind`는 재전송된 메시지를 나타냅니다. `sync_sends_slow`는 전송 시간이 200 us를 초과한 동기화(Sync)의 횟수를 집계하며, `sync_send_max_us`는 가장 긴 전송 시간을 기록한 것입니다. 각 Follow_Up의 발생 시간은 해당 전송 시간 중 최대 200 us까지만 집계됩니다. 그랜드마스터는 별도의 프로세스(`own_process`, `process_pid`)에서 실행되며, Windows에서는 높은 우선순위(`process_priority`)로 실행됩니다. 따라서 부하가 많은 백엔드에서도 타임스탬프 기록을 지연시키거나 모든 카메라의 시계를 한꺼번에 조정할 수 없습니다. 해당 프로세스를 시작할 수 없는 경우, 이전과 마찬가지로 백엔드 내에서 실행됩니다(`own_process` false).

```python
st = chloros_sdk.time_sync_status()
if st.get("health") not in (None, "OK", "STOPPED"):
    print("PTP:", st["health"], "-", st.get("health_detail"))
    chloros_sdk.restart_time_sync()
```

| 기능 | 역할 |
| --- | --- |
| `time_sync_status()` | 그랜드마스터의 상태, 정상 여부, 시계 식별자, 우선순위 및 메시지 카운터(위 참조). `chloros-cli time-sync status`. |
| `time_sync_peers()` | 수신하는 PTP 슬레이브 (카메라, DAQ-E): `peers`, 각 `port_identity`, `last_seen_ip`, `last_delay_req_age_ms`, `delay_req_count`. `time-sync peers`. |
| `time_sync_cameras()` | 연결된 각 어레이 카메라의 `PtpStatus`, `PtpOffsetFromMaster` 및 `PtpMeanPathDelay` (펌웨어에 PTP 노드가 없는 카메라의 경우 시리얼 번호만 표시됨). `time-sync cameras`. |
| `restart_time_sync()` | 그랜드마스터를 중지하고 다시 시작합니다. 이후 모든 풀링된 DAQ-E의 PTP가 재확인됩니다 (`daq_ptp_rechecked`). 다음이 발생하면 `ChlorosConnectError`를 발생시킵니다 복구되지 않을 때 `ChlorosConnectError`를 발생시킵니다. `time-sync restart`. |
| `set_time_sync_priority(priority1=None, priority2=None)` | 다른 PTP 마스터가 있는 네트워크에서 이 프로세스의 PTP 우선순위(0-255, 수치가 낮을수록 베스트 마스터 선출에서 우위를 차지함). 해당 마스터가 실행 중이어야 합니다. `time-sync set-priority`. |

각각 `backend_url=`와 `timeout=`를 받아 실행 중이지 않은 로컬 백엔드를 시작합니다.

---

## 카메라 유틸리티

`chloros-cli lattice`는 `chloros_sdk`(모듈 `chloros_sdk.camera_utils`)의 일반 함수에서 제공하는 연결 및 캡처 기능 외에도 다음과 같은 역할을 수행합니다. 각 함수는 CLI 명령어가 사용하는 백엔드 경로를 CLI의 규칙에 따라 호출하므로, 따라서 두 함수는 동일하게 동작합니다. 이 함수들은 세션이 아닌 카메라 **시리얼** 번호를 인수로 받으며, 스마트 연결 호출과 동일한 `backend_url=` / `timeout=` 키워드를 사용합니다. `connect_camera()`와 마찬가지로, 실행 중이지 않은 백엔드가 시작됩니다. 요청이 거부되면 백엔드의 거부 사유가 포함된 `ChlorosConnectError`가 발생합니다.

```python
import chloros_sdk as c

c.set_camera_color_profile("214701281", profile="natural")
c.set_camera_highlights("214701281", -40)
print(c.camera_firmware_status()["cameras"])
st = c.network_adapters(estimate=True, cameras=4, pixel_format="BayerRG12")
```

| 함수 | 기능 |
| --- | --- |
| `camera_color_profile(serial)` / `set_camera_color_profile(serial, profile=, custom_cct_k=)` | 디스플레이 색상 프로필: `raw`, `linear`, `natural`, `enhanced` 또는 `custom_temp`, 그리고 해당 색온도(2000~10000 K로 제한됨; 프로파일이 `custom_temp`일 때 읽힘). |
| `camera_color_adjust(serial)` / `set_camera_color_adjust(serial, saturation=, contrast=, reset=False)` | 화면 채도 및 명암비, 각각 0~200 % (100 = 변경 없음); `reset=True`는 두 값을 모두 100으로 되돌립니다. |
| `camera_highlights(serial)` / `set_camera_highlights(serial, value)` | 디스플레이 렌더링에서 가장 밝고 흰 픽셀의 값을 낮추기: -100..0 (0 쪽으로 잘린 후 클램핑됨; 0 = 꺼짐). |
| `camera_white_balance_once(serial, settle_s=None)` | 원샷 화이트 밸런스: 카메라가 잠시 스트리밍하여 수치를 수렴시킨 후, `red_ratio` / `blue_ratio` (보고할 수 없는 경우 `ratios_read` False). 해결된 화이트 밸런스를 유지하며, open이 쓴 다른 모든 설정은 발견된 그대로 반환됩니다. |
| `set_camera_white_balance(serial, mode=None, red=None, blue=None)` | `mode="auto"` (연속) 또는 `"off"`; 또는 고정된 `red` / `blue` 밸런스 비율로, 이 경우 자동 기능이 먼저 꺼집니다. 카메라가 거부하는 비율은 `errors` 아래에 명명됩니다. |
| `camera_presets()` / `apply_camera_preset(serial, preset)` | 카메라의 사전 설정이며, 하나를 적용하면: 카메라는 사전 설정 이름에 해당하는 설정을 유지합니다 (예: 카메라가 이후 읽는 프레임 속도 등, 응답의 `settings`; 프리셋에 명시되지 않은 필드의 경우 `None`), 그리고 open이 기록한 다른 모든 설정은 발견된 그대로 복원됩니다. `"default"`는 카메라를 공장 초기값으로 재설정합니다(자유 실행 자동 노출, 링크 처리량 제한 및 결함 보정 기능이 꺼진 공장 설정이며, RGB 카메라의 보정된 화이트 밸런스는 유지됨). 이 설정은 카메라 메모리에 저장되지 않습니다: 응답에는 `factory_defaults: True`와 `settings`의 모든 필드가 포함됩니다. `camera_presets()`에서, `default`는 `{factory_defaults: True, description}`입니다. |
| `camera_filter_models()` / `camera_filter_model_info(name)` | 필터 모델 및 한 모델의 세부 정보(대역, 분광 응답 조건수). 카메라의 필터 모델은 모델 문자열에서 읽혀지며, 수동으로 설정되지 않습니다. |
| `camera_firmware_status(serial=None)` | 각 카메라의펌웨어: `current_version`, `available_version`, `update_offered`, `needs_minimum_update` 및 `minimum_firmware`. |
| `update_camera_firmware(serial, force=False)` | 이 Chloros 빌드의 펌웨어를 카메라 한 대에 플래시합니다: 공장 초기화, 플래시, 재부팅, Chloros 기본 설정이 다시 기록되며; 카메라의 보정 값은 유지됩니다. 몇 분 소요되며, 이 동안 카메라의 전원은 계속 켜져 있어야 합니다. `force=True`는 현재 카메라에 펌웨어를 다시 플래시합니다. |
| `camera_power_probe(serial)` | 카메라의 전원 및 열 노드와 그 온도. |
| `sleep_camera(serial)` / `wake_camera(serial)` | 카메라의 저전력 대기 모드(깨워질 때까지 절전 상태로 유지됨: 링크 처리량 제한이 활성화된 상태(`DeviceLinkThroughputLimitMode` `On`)에서 최저값으로 설정되어 있으며, 카메라에 절전 모드가 있는 경우 해당 모드가 활성화된 상태), 그 전후의 온도; 깨어날 때 링크는 절전 모드 진입 시의 상태(제한값, 모드, 패킷 지연 `GevSCPD`—제한값이 활성화된 동안 카메라가 0으로 유지하는 값)로 복원되며, 이 기록은 이 컴퓨터에 저장되므로 이후 백엔드에서 깨울 때도 이를 확인할 수 있습니다; 또는, 이 컴퓨터가 절전 모드로 전환하지 않았으나 절전 상태로 보이는 카메라(한도 설정이 최저값으로 활성화됨)의 경우, 한도 설정을 비활성화하고 `GevSCPD` 값을 80(공장 출하 시 설정)에서 0으로 유지하며; 최저값 이상으로 설정된 제한 값은 그대로 유지됩니다. 전원 상태만 저장되며, open 명령어가 기록한 다른 모든 설정은 발견된 그대로 복원됩니다. |
| `calibrate_camera_with_target(serial, filter=None, attempts=30, settings=None)` | MAPIR 보정 대상이 화면에 나타날 때까지 프레임을 캡처하고, 해당 패널의 채널별 계수를 적용합니다: `target_id`, `panel_dns`, `coefficients`. |
| `network_adapters(estimate=False, cameras=1, pixel_format="BayerRG8")` | 이 컴퓨터의 GigE Vision용 이더넷 어댑터(MTU, 링크 속도, 점보 프레임) 및 모든 `issues`; `estimate=True`는 `cameras` 카메라가 얻을 프레임 속도와 대역폭을 추가합니다. |
| `fix_network_settings()` | 이 컴퓨터의카메라 어댑터에 적용합니다. 이 작업은 어댑터 설정을 변경하며, Chloros가 관리자 권한으로 실행될 때만 작동합니다. 그렇지 않은 경우, 발견된 내용에 대한 로그를 출력하며 종료됩니다. |
| `apply_vignette_correction(input_dir, output_dir, lens_model=None)` | 폴더 내 이미지에 비네트 보정을 적용하며, 각 이미지는 해당 렌즈의 플랫 필드 맵(`processed`, `lens_models`(각 파일이 보정된 렌즈), `skipped`(이유 포함). `lens_model`가 없으면 각 파일의 렌즈는 EXIF 모델 정보에서 가져옵니다; `lens_model`는 모델명이 명시되지 않은 파일에 적용되며, 모델명이 이와 상충하는 파일은 건너뜁니다. 파일만 해당됩니다. |
| `read_dls(mac, filter=None, camera_serial=None)` | DLS(블루투스 하향광 센서)에서 읽은 한 번의 측정값: 스펙트럼, 대역별 조도, 그리고 `camera_serial`가 지정되었을 때 카메라 자체의 응답을 통한 대역별 광도 값. `discover_daq_sensors(transports=["ble"])`는 MAC 주소를 찾습니다. |
| `probe_system(pixel_format="BayerRG12", discover=True)` / `system_info()` | 호스트의 카메라용 하드웨어 및 권장 구성(`report_text`), 그리고 시스템의 CPU / GPU / PyTorch 런타임. |

- **일부 호스트는 카메라를 단독으로 사용해야 합니다.** `camera_white_balance_once()`, `apply_camera_preset()`, `sleep_camera()` / `wake_camera()`, `calibrate_camera_with_target()` 및 `update_camera_firmware()`는 카메라를 엽니다. 반면 Chloros가 카메라를 점유하고 있는 동안(앱에 연결되어 있거나, `connect_camera()`와 같이 어레이 구성원으로 포함되어 있거나, 현재 다른 명령에 의해 사용 중이거나, 예: 배열 테스트나 카메라를 여는 캡처 명령 등) 백엔드는 HTTP 409 `CAMERA_HELD` 오류를 반환하며(`code` 및 `error_code` 모두에서)오류 코드와 함께 해당 리소스를 해제하는 방법을 명시합니다. 색상 설정, 하이라이트, `camera_power_probe()` 및 읽기 기능은 연결된 카메라에서 작동합니다.
- **색상, 하이라이트 및 화이트 밸런스는 RGB 카메라용입니다.** 단일 대역 모노(M3M) 카메라에서는 설정 함수가 아무것도 변경하지 않고, CLI와 마찬가지로 `{"skipped": True, "reason": ...}`를 반환합니다. 이는 디스플레이 설정으로, 라이브 뷰 및 디스플레이 수준의 캡처에 적용되며, 절대 복사도나 반사도는 아닙니다.

---

## 일시적 배열, 정렬 프로파일 및 인덱스

`chloros-cli lattice`는 지속적인 연결이 없는 상태에서, `chloros_sdk`(모듈 `chloros_sdk.array_utils`)의 함수들과 마찬가지로: `continuous`, `multi-capture`, `multi-stream`, `multi-test`, `multi-detect`, `analyze-array`, `align-calibrate`, `align-apply`, `align-stream`, `align-info`, `align-reorder` 및 `index`입니다. 각 명령은 CLI 명령이 보내는 요청을 CLI의 기본값을 사용하여 전송합니다.

```python
import chloros_sdk as c

prof = c.calibrate_alignment_profile(["214000535", "214000551", "214000556"],
                                     "D:/rig/alignment_profile.json", method="orb", model="rigid")
print({sn: t["reprojection_error"] for sn, t in prof["transforms"].items()})

c.capture_aligned_multiband("D:/rig/alignment_profile.json", "D:/rig/out")
idx = c.capture_live_index("D:/rig/alignment_profile.json", preset="NDVI",
                           output_dir="D:/rig/ndvi", capture_level="reflectance")

with c.start_multi_camera_stream(["214000535", "214000551"], "D:/rig/stream", fps=5, count=50) as job:
    print(job.wait()["final_stats"])          # waits for the 50 groups, then closes the cameras
```

- **이 명령어들은 카메라를 직접 엽니다.** 각 명령어는 카메라를 열고, 작업을 수행한 후 닫으므로, Chloros가 보유하지 않은(앱에 연결되어 있거나, `connect_camera()`로 연결되었거나 배열 요소로 지정된) 카메라가 필요합니다. 백엔드는 HTTP 409 오류(`CAMERA_HELD`)를 반환하며 카메라 해제 방법을 안내합니다. 일시적 열기 작업에는 시간이 다소 소요됩니다(카메라 몇 대 기준 약 40초). 반복적인 캡처의 경우, `connect_array()`를 한 번 호출하는 것이 가장 빠른 방법이며, 일시적 캡처가 디베이어링된 이미지를 저장하는 모든 내보내기 유형에서 이 방법을 사용할 수 있습니다. `calibrate_alignment_profile()`는 예외입니다. 해당 카메라들이 연결된 어레이에서는 어레이 자체의 프레임에서 문제를 해결합니다 (`used_live_array`)를 해결합니다.
- **`serials`**는 열어야 할 카메라 목록이며, 첫 번째 카메라가 트리거를 작동시킵니다. `None`는 네트워크상의 모든 카메라를 엽니다.
- **`settings`**는 카메라 설정 재정의 값을 담은 딕셔너리로, CLI 플래그가 설정하는 필드입니다: `exposure_time` (µs), `gain` (dB), `pixel_format`, `width`, `height`, `target_brightness`, `ae_damping`, `ae_upper_limit`, `frame_rate`, `gev_packet_size`, 그리고 `preset`(이름으로 지정된 카메라 프리셋)입니다. 노출값이나 게인 값을 지정하면 플래그와 마찬가지로 자동 제어 기능이 비활성화됩니다.
- **`gpu`**: `True`는 GPU 워프를 강제하고, `False`는 CPU (`--gpu` / `--no-gpu`)를 강제하며, `None`는 백엔드가 선택하도록 합니다.
- **`keep_settings`**/**`clear_settings`** (카메라를 여는 모든 함수, 기본값은 `False`): 처리 종료 시 각 카메라를 발견된 상태 그대로 복원합니다; `keep_settings=True`는 대신 실행 설정을 카메라 자체 메모리에 저장합니다(자유 실행 모드), `clear_settings=True`는 해당 메모리를 공장 초기 설정으로 재설정합니다 — [카메라 메모리](#camera-memory--keep_settings--clear_settings)를 참조하십시오. 응답(작업에 대한, `stop()`의)에는 `camera_memory`가 포함됩니다.
- **경로는 백엔드의 경로입니다**: 백엔드가 이 컴퓨터에 있을 때 상대 경로는 작업 디렉터리를 기준으로 절대 경로로 변환됩니다. 정렬 프로필 파일 기능(`alignment_profile_info()`, `reorder_alignment_profile()`)은 CLI와 마찬가지로 이 컴퓨터에서 파일을 읽습니다.

| 함수 | 기능 |
| --- | --- |
| `start_camera_continuous(serial, output_dir="output", *, format="tiff", jpeg_quality=95, queue_depth=64, settings=None)` | 중지될 때까지 하나의 카메라를 지속적으로 녹화합니다. `CliJob`를 반환합니다. `tiff` 실행의 프레임은 해당 카메라의 캡처 배열이 작성한 `raw` 파일입니다(센서의 모자이크 평면 하나당값을 기반으로 한 모자이크 평면 하나)로 구성된 배열로 기록됩니다(비압축); `png` 및 `jpg`는 8비트를 저장하므로, 해당 실행의 프레임은 보기를 위해 디모자이징된 프레임입니다. 바운드 또는 풀링된 DAQ는 실행의`daq/` 폴더(통계 정보상 `daq_files`)에 기록합니다. 각 프레임은 전체 메타데이터(모델, 캡처 시간, 렌즈, 밴드, 원시 프레임에 대한 보정)와 함께 한 번씩 기록됩니다; `stop()`는 캡처된 모든 프레임이 기록될 때마다(최대 25초) 한 번씩 반환되며, 그 후에야 카메라가 종료됩니다. 하드웨어 트리거를 위해 준비 상태로 남겨진 카메라 (`TriggerMode`, `On`)는 해당 실행 기간 동안 프리런(free-run) 모드로 설정됩니다(시작 응답의 `trigger`에서 확인된 내용을 확인할 수 있습니다). 중지 시, `keep_settings=True`(런 설정을 카메라 자체 메모리에 저장함, 프리런 모드) 또는 또는 해당 메모리를 초기화하는 `clear_settings=True`가 사용되지 않는 한, 카메라가 발견된 상태 그대로 복원됩니다. 프레임을 기록하지 않는 실행은 10초 후 `stop_reason` 및 `no_frames`를 통해 종료되며, 그 이유는 `last_error`에 기록되며, `stop()`는 해당 사유를 `error`로 반환합니다. 라이터가 처리 속도를 따라가지 못해 누락된 프레임은 집계되며, `drop_reason`가 그 이유를 설명합니다. Chloros가 보유한 카메라의 경우, `CameraSession.capture_repeated()`를 사용합니다. `lattice continuous`. |
| `start_multi_camera_stream(serials=None, output_dir="output", *, fps=3.0, count=None, format="tiff", jpeg_quality=95, settings=None)` | `fps`에서 `count` 그룹 수만큼 또는 중지될 때까지 동기화된 프레임 그룹을 스트리밍합니다. `CliJob`를 반환합니다 (이 `start_reply`에는 `n_cams`와 와이어 `pacing`가 포함됨); 최종 통계에는 캡처된 그룹, 기록된 그룹, 삭제된 그룹, 라이터가 기록하지 못한 파일, 그리고 카메라가 아무것도 전달하지 않은 그룹의 수가 포함됩니다. `lattice multi-stream`. |
| `capture_cameras_transient(serials=None, output_dir="output", *, format="tiff", jpeg_quality=95, settings=None)` | 동기화된 캡처 1회, 카메라당 디베이어링된 이미지 1장: `results` (`filepath`, `serial`, `frame_id`) 및 `pacing`. 카메라가 누락된 그룹이 발생하면, 먼저 저장된 파일 수를 명시합니다. 어레이가 연결되지 않은 `lattice multi-capture`. |
| `test_array_sync(serials=None, count=10, ptp_settle_s=None, timeout=None)` | 고정 노출에서 동기화된 캡처를 수행하는 `count`: `sync_groups`, `master`, 트리거당 한 행의 `group_timestamps`(한 장을 놓친 카메라는 해당 행에서 누락됨) 및 `sync`: 트리거마다 마스터 카메라 대비 각 카메라의 오프셋 (`offsets_us`: 카메라당 `mean_us`, `min_us`, `max_us`, `jitter_us`, `groups`), 하나의 클럭 단위(`clock_domain`: `ptp`, 또는 `device` — PTP가 꺼진 상태에서 각 카메라의 자체 가동 시간 클럭을 사용하므로, 오프셋은 부팅 시점으로 간주되며 지터만 동기화를 측정함). 타임스탬프에 장치 클럭과 PTP가 혼합된 트리거(PTP 비활성화 또는 여전히 동기화 중)는 제외되며, PTP 클럭이 마스터 클럭과 여전히 `agree_within_us` 이상 차이가 나는 트리거도 마찬가지로 집계됩니다 (`groups_clocks_disagree`); `notes`가 이를 나타냅니다. `clock_offset_mean_us` / `clock_offset_jitter_us`는 카메라별 최대 평균값과 지터입니다. `success`는 거짓이며, `error`는 트리거를 놓쳤을 때, 하나의 클럭에서 모든 카메라가 두 번 미만의 트리거를 기록했을 때, 또는 클럭이 일치하지 않았을 때 발생합니다 (`sync.problems`)인 경우 false이며, 이 경우 해당 행은 어떤 경우든 반환됩니다. PTP 그랜드마스터가 실행 중이 아닌 경우 이를 시작합니다(`time_sync`는 `grandmaster_running` 및 `grandmaster_started_by_this_test`를 나타냄), 모든 카메라의 PTP를 켭니다(`connect_array()`가 수행하는 대로), 그리고 첫 번째 트리거가 발생하기 전에 모든 카메라의 PTP 클럭이 그랜드마스터에 동기화되어 안정화될 때까지 대기합니다: `PtpStatus`는 하나의 그랜드마스터에 슬레이브로 연결되어, 그랜드마스터와 100 us 이내의 오차를 보이며 3초 동안 안정적으로 유지되고, 각 카메라가 이와 같이 동기화된 상태에서, 중단 없이 120초 동안 — 전원을 켠 지 약 2분 후까지 — 유지됩니다. 방금 동기화된 클럭은 ‘Slave’로 판정된 후 약 2분 동안 최대 ~200 us의 오차를 보이지만, 카메라 자체의 `PtpOffsetFromMaster`는 이를 표시하지 않습니다; 120초가 지나면 카메라들의 클럭은 안정화된 클럭과 ~10 us 이내의 오차 범위로 수렴합니다. 이 백엔드에서 확인한 바와 같이, 이전 테스트에서 PTP를 켠 상태로 종료된 후 여전히 동기화된 상태인 카메라는 해당 동기화 시점부터 카운트를 시작하므로, 따라서 연속된 테스트 간에는 다시 대기하지 않습니다. 테스트 중에 전원이 켜진 카메라나, PTP가 켜져 있으나 이 백엔드에서 아직 동기화된 적이 없는 카메라(다른 곳에서 켜졌거나 백엔드 재시작 전에 켜진 경우)는 120초를 기다리며, 두 카메라가 모두 방금 동기화된 상태인 경우 해당 카메라들은 여전히 각자의 대기 시간을 기다립니다. 슬레이브 상태를 벗어나거나, 100 us를 초과하거나, 대기 시간 동안 3회 연속으로 다른 그랜드마스터를 따라가는 (1.5초) 동안 그 카메라는 카운트를 다시 시작합니다; 이러한 판독값 중 하나 또는 두 개가 로그에 기록되며(`[array-sync] PTP SN:<serial> off lock at <s> s (<n> of 3): <status> <±x> us gm <id> -- locked <s> s, counting on`), 카운트는 계속됩니다. 또한 읽을 수 없는 그랜드마스터 ID는 변경 사항이 아닌 '알 수 없음'으로 간주됩니다. 모든 카메라의 카운트가 동시에(1초 이내) 다시 시작되는 것은 카메라의 시간이 아닌 그랜드마스터의 시간이 이동했기 때문입니다. 백엔드로그에는 `[array-sync] PTP grandmaster step: all <n> cameras restarted at <s> s …`라고 기록되어 있고, `ptp_lock`는 이를 `grandmaster_steps`에 나열하며, 이후 대기 시간이 만료되면 오류 기록은 `every camera's count restarted together at <s> s -- the PTP grandmaster's time moved under them …`로 시작됩니다로 시작됩니다. 이에 대한 시간 제한은 연장되지 않습니다. `ptp_settle_s`(0-600; 정산 대기 시간이 없는 경우 `0`)는 실행에 대한 정산 시간을 설정하며, 다른 값은 거부됩니다(HTTP 400). 대기 시간은 정산 시간에 90초를 더한 값으로 제한되며(기본값 210초, 최소 90초), 호출은 백엔드에서 `timeout` 초 동안 대기합니다. 기본값은 240초에 정산 시간을 더한 값(360초)이며, `chloros-mcp`의 `test_array_sync`와 마찬가지로. `ptp_lock`에는 `converged`, `waited_s`, `timeout_s`, `settle_s`, `locked_s` (각 카메라가 얼마나 오랫동안 고정되어 안정적으로 유지되었는지), `found_settled` (발견 시점에 이미 안정화 시간 동안 고정되어 있던 카메라) 및, 카메라별로, `status`, `offsets_ns`, `slave_s` (동기화된 지 경과한 시간; None: 동기화되지 않음) 및 `remaining_s` (어떤 카메라의 PTP도 켜지지 않은 경우 None)가 있습니다. 백엔드 로그에는 5초마다 각 카메라의 상태, 동기화된 지 경과한 시간 및 남은 시간이 기록된 행이 하나씩 포함됩니다. 모든 카메라는 PTP가 꺼진 상태로 전원이 켜지며, 이후 약 10초 만에 클럭이 동기화됩니다. 그 후 각 카메라는 테스트 시 확인된 상태(PTP, 자동 노출 및 게인)로 복원되며, 테스트 실행 기간 동안 해당 설정은 ‘Off’로 유지됩니다 (`ExposureAuto` / `GainAuto`, 그리고 수동 모드인 `ExposureTime` / `Gain`), 나머지는 `keep_settings=True`인 경우를 제외하고 — 즉, PTP가 켜진 상태로 남겨진 카메라(`PtpEnable=True`, 또는 `keep_settings=True`가 설정된 실행) 는 잠금 상태를 유지하며, 이 백엔드가 해당 카메라가 정착 시간 동안 잠금 상태임을 확인하면 이후 테스트에서는 이를 기다리지 않습니다. 클럭이 정착되지 않으면, 트리거 발생 전 `success`는 false이며, `error`는 어떻게 처리해야 하는지 명시합니다: 한 번도 잠금 상태에 도달하지 못한 카메라(`not_slave`)는 그랜드마스터가 해당 카메라에 도달하지 못하고 있음을 의미합니다(`time_sync_status()`); 계속해서 연결이 끊기던 카메라는 잠금 상태를 유지한 기간을 기준으로 이름이 지정됩니다 (`remaining_s`); 클럭이 여전히 수렴 중인 경우(`far`): `time_sync_status()`를 확인하고, 테스트 전에 카메라를 잠그려면 PTP를 켜고 계속 켜두십시오 (각 카메라에서 `PtpEnable=True` 또는 `keep_settings=True`; 테스트는 감지된 상태를 복원하므로 PTP는 켜진 상태로 유지됨), 그런 다음 테스트를 실행하십시오. `lattice multi-test`. |
| `detect_array_master(line=None)` | 동기화 라인에 연결된 카메라와 기본 마스터: `master_serial`, `slave_serials`, `line`가 탐지되었습니다. `lattice multi-detect`. |
| `analyze_array_models(camera_models=None, *, width=2048, height=1536, n_active=None, force_tier=None, binning=1, packed="auto", pixel_format=None)` | 이 카메라 모델들(제품명, `"LATT-M3C-L87-FOCN"`)가 이 컴퓨터의 네트워크에 연결될지: `tier`, `achievable_fps_max`, 케이블 예산 (`max_cams_collision_safe`, `oversubscribed`), `cam_caps` 및 `fix_suggestions`. 기본적으로 탐지된 카메라의 모델명이며; 열린 카메라는 없습니다. 연결에 사용되는 설정을 적용한 기존 카메라에 대해서는 [`analyze_array_network()`](#network-analysis-helper)를 참조하십시오. `lattice analyze-array`. |
| `calibrate_alignment_profile(serials=None, save_path="alignment_profile.json", *, frames=3, method="orb", model="rigid", reference_serial=None, checkerboard=None, control_points=None, preview=False, vignette=False, roi=None, mask_path=None, rms_threshold_px=None, per_cam_overrides=None, settings=None, ...)` | 실시간 프레임에서 정렬 프로필을 도출하고 저장합니다. `method`: `orb`, `akaze`, `phase`, `checkerboard` (`checkerboard="9x6"`와 함께, 내부 모서리) 또는 `manual` (`control_points={serial: [[x, y], ...]}`와 함께, 기준 카메라의 프로파일도 포함). `model`: `translation`, `rigid`, `affine` 또는 `homography`. 더 세부적인 옵션은 CLI의 플래그입니다: `max_features`, `ratio_threshold`, `matcher` (`bf` / `flann`), `knn_k`, `prefilter` (`none` / `gradient` / `clahe` / `blur` / `hist_match`), `ransac_threshold_px`, `ransac_max_iters`, `ransac_confidence`, `averaging`, `lock_rotation`, `lock_scale`, `lock_axis`; `per_cam_overrides`는 `{serial: {option: value}}`입니다. `rms_threshold_px`는 더 나쁜 해를 저장하는 것을 거부합니다. `save_path`, `output_width` / `output_height`, `crop_roi`, `reference_serial`, `transforms` (카메라별 `reprojection_error`, `num_matches`) 및 `preview_path`. `lattice align-calibrate`. |
| `capture_aligned_multiband(profile_path, output_dir="output", *, serials=None, format="tiff", bit_depth=None, gpu=None, no_crop=False, bands=None, order=None, vignette=False, per_band=False, per_camera=False, interpolation="linear", border_mode="constant", border_value=0, settings=None)` | 프로파일을 통해 정렬된 단일 다중 대역 이미지: `files` (해당 `bands` 및 크기를 가진 다중 대역 TIFF 파일, 대역당 하나씩(`per_band`) 및 카메라당 하나씩(`per_camera`) 추가된 파일) 및 `vignette_corrected`. `bands`는 밴드를 선택하고, `order`는 이름(목록 또는 쉼표로 구분된 문자열) 순서대로 밴드를 정렬합니다. `lattice align-apply`. |
| `start_aligned_stream(profile_path, output_dir="output", *, serials=None, fps=3.0, count=None, ...)` | 스트림 정렬된 다중 대역 이미지(`capture_aligned_multiband()`와 동일한 워프 옵션)를 `count`까지 또는 중지될 때까지 처리합니다. `CliJob`를 반환하며, 최종 통계에는 `frames_written` 및 모든 `frame_groups_incomplete`가 포함됩니다. `lattice align-stream`. |
| `alignment_profile_info(path)` | `align-info`로 요약된 프로필 파일은 이를 출력합니다: `name`, `created`, `method`, `reference_serial`, `sensor_resolution`, `output_width` / `output_height`, `crop_roi`, `camera_serials`, `transforms` (카메라당 `model`, `reprojection_error`, `num_matches`, `reference`), `camera_models`, `layer_order` (카메라별 `band_names`, `wavelengths_nm`, `enabled`); `profile`는 전체 파일입니다. |
| `reorder_alignment_profile(path, order=None, enable=None, disable=None)` | 프로필 파일을 제자리에서 변경: `order`는 밴드 이름입니다(각각 해당 순서대로 해당 카메라의 레이어를 위로 이동시킵니다), `enable` / `disable`는 일련번호입니다. 새로운 `layer_order`를 반환합니다. `lattice align-reorder`. |
| `index_catalog()` | 인덱스 프리셋(`presets`: 각각 `name`, `channels`, `formula`) 및 색상 `gradients`. `lattice index --list-presets` / `--list-gradients`. |
| `compute_index(input_path, *, preset=None, formula=None, channel_map=None, output=None, capture_level="unknown", output_format="all", colorize=False, gradient="RdYlGn", vmin=None, vmax=None, percentile=None, bg_mode="clip")` | 정렬된 다중 대역 TIFF에 대한 인덱스: `preset` (`NDVI`, `NDRE`, ...) 또는 `formula`는 `channel_map` 기호에 대해 (`{"NIR": "NIR_850", "Red": 0}`, 이름 또는 인덱스별로) 밴드로 매핑됩니다. '사전 설정'심볼 `channel_map`는 해당 심볼로 시작하는 이름을 가진 파일의 단일 밴드만 포함합니다(`red` -> `Red_660`, `nir` → `NIR_850_RGN`), 파일 자체(해당 파일의 밴드 목록 또는 Chloros 캡처/제품의 `Chloros:Band{R,G,B}Name`)에 의해 명명된, 따라서 Chloros 반사도 캡처의 NDVI에는 `channel_map`가 필요하지 않습니다; 하나의 심볼에 대응하는 두 개의 밴드는 추측되지 않고 거부됩니다. `capture_level`는 밴드가 포함하는 내용을 나타내며, `reflectance` 또는 `radiance`는 값을 정량화합니다; 보정되지 않은 스택(`align-apply`)은 정성적 값을 제공합니다; `"unknown"`는 파일에서 이를 읽으며, 여기에는 Chloros 파일의 `Chloros:ProcessingLevel`가 포함됩니다. `stats`를 반환합니다 (모양, 1~99 백분위수 범위, LUT 범위, 사용된 `capture_level`) 및 입력 옆에 기록된 `files` (`<input stem>_<preset>.tif`, `..._LUT.png`) 옆이나 `output` 위치에 기록된 경로: 이미지 확장자가 있는 파일은 그 줄기(stem)를 기준으로 이름이 지정됩니다 (`output="D:/out/ndvi_FRGN.png"`는 `ndvi_FRGN.tif` 및 `ndvi_FRGN_LUT.png`로 작성됨), 확장자가 없는 경우 접두사로 처리됩니다 (`D:/out/ndvi` → `ndvi.tif`, `ndvi_LUT.png`); `files`에서 경로를 읽어옵니다. 이 파일들은 입력 데이터의 카메라 기록(제조사/모델/일련번호, GPS, 렌즈, UTC 기준 촬영 시간, 보정 식별자(단, 원시-DN 보정 값은 절대 포함되지 않음), `Chloros:ProcessingLevel=index`, 그리고 인덱스를 단일 밴드로 포함합니다(`XMP-Camera:BandName`, TIFF 및 LUT PNG 모두). 파일만 포함되며, 카메라 정보는 없습니다. `lattice index --input`. |
| `capture_live_index(profile_path, *, preset=None, formula=None, channel_map=None, serials=None, output_dir="output", output_prefix="index", capture_level="reflectance", save_multiband=False, ...)` | 촬영, 보정 (`reflectance` 또는 `radiance`; `raw`는 이 단계를 건너뜁니다), 정렬 및 인덱스 계산을 한 번에 수행합니다; `save_multiband=True`는 보정된 스택을 유지하며, `compute_index()`는 나중에 이를 가져올 수 있습니다. `stats`(사용한 대역이 포함된 `serials`와 함께)와 `files`를 반환합니다. `serials`를 엽니다(첫 번째가 트리거를 발사함). 그렇지 않으면 `layer_order` 항목이 활성화된 프로파일의 카메라들을 엽니다(참조 카메라를 먼저 (밴드가 비활성화된 상태에서도 열림); 밴드가 비활성화된 카메라는 절대 보정되지 않습니다. `channel_map`는 스택이 응답할 수 없는 심볼의 경우에만 필요하며, 예를 들어 `compute_index()`의 경우: 프리셋의 심볼은 이름이 해당 심볼로 시작하는 활성화된 단일 대역을 선택하며, 이는 어떤 카메라도 열리기 전에 확인됩니다(응답하는 카메라가 없거나 여러 대인 경우 거부되며, 해당 카메라들의 이름이 명시됨); 사용자 정의 `formula`는 모든 심볼에 필요합니다. `index.tif`와 LUT는 스택의 카메라, 촬영 시간(기준 카메라의 노출 시간, UTC) 및 렌즈 정보를 포함하며, 둘 다 인덱스를 단일 밴드로 지정합니다(`XMP-Camera:BandName`, 예: `NDVI`: 컬러 PNG는 색상 채널이 아닌 인덱스를 지정합니다). `lattice index --live`. |

### `CliJob`

세 개의 스트림이 반환하는 핸들입니다. 작업은 백엔드에서 실행되며, 중지될 때까지 해당 카메라를 소유합니다.

| 멤버 | 설명 |
| --- | --- |
| `stats(timeout=10.0)` | `running`, `state`, `stats` (저장된 프레임 / 기록됨, 삭제됨, `fps`, `elapsed`, 기록 실패, `stop_reason`; 연속 작업의 경우 `frames_grabbed` 및 `frames_queued` — 캡처된 모든 프레임은 기록, 대기열에 추가, 삭제 또는 실패 처리됨 — 및 `drop_reason`, `writers` 및 `write_ms`(한 프레임을 쓰는 데 걸린 평균 시간) 및 `output_dir`. `state`는 `running`, `ended`(자체적으로 중지되었으며 아직 완전히 중지되지 않은 상태), `stopping`(중지 중인 프로세스가 대기열을 출력 중: 실시간 통계), `stopped_by`) 또는 `finished`(최종 통계, 중지 작업의 `error` 및 `stopped_by`). 중지된 작업의 기록은 모든 클라이언트에 대해 10분 동안(최대 최근 32개까지) 보관되며, 다른 스크립트나 `chloros-mcp`가 동일한 상태 및 최종 통계를 읽습니다. `chloros-cli`로 시작된 작업 (`lattice continuous`, `multi-stream`, `align-stream`) CLI가 30초 동안 체크인을 중단하면(종료되었거나, 해당 머신이 절전 모드인 경우) 백엔드에 의해 중지되며, 이때 백엔드에는 `stop_reason`, `client_gone` 및 `stopped_by`가 기록됩니다; 여기서 해당 프로세스의 상태를 읽어오는 것만으로는 프로세스를 유지하지 못합니다. SDK에서 시작된 작업은 중지될 때까지 실행됩니다. 각 시작 시에는 `timeout`를 백엔드의 마감 시간(`client_deadline_s`)으로 전송합니다: 그 이후에 열리는 카메라(부팅 중인 카메라)는 다시 복원되었다가 닫히며, 작업이 시작되지 않으면 호출 시 `ChlorosConnectError`(`CLIENT_DEADLINE_PASSED`)가 발생합니다. 작업이 점유 중인 카메라는 다른 모든 열기 요청에서 거부됩니다 (`connect_camera()`, 다른 시작, 일시적 캡처) 시, 해당 작업과 시작자를 명시하며 HTTP 409 오류(`CAMERA_HELD`)를 반환합니다. |
| `stop(timeout=30.0)` | 중지하고 카메라를 닫습니다: `final_stats`, `output_dir`, 그리고 중지가 정상적으로 완료되지 않았을 때 `error`. 두 번째 호출은 동일한 응답을 반환하며, 다른 클라이언트에서 중지 요청을 보내도 마찬가지입니다: 첫 번째 요청이 대기열을 출력하는 동안 전송된 요청은 이를 기다립니다(최대 25초; 그 시간이 지나면 예외가 발생하며, 최종 통계는 `stats()`에 이어집니다). |
| `wait(timeout=None, poll_s=0.5)` | 작업이 자체적으로 종료될 때까지 대기한 후(`count` 도달, 드라이브 용량 초과, 오류 발생 시), `stop()`를 호출하여 최종 카운트를 수집합니다. `timeout`는 작업이 여전히 실행 중인 상태에서 `TimeoutError`를 발생시킵니다. `count`가 없는 작업은 중지될 때만 종료됩니다. |
| `job_id`, `kind`, `output_dir`, `start_reply`, `stopped` | 해당 작업의 ID 및 종류(`continuous`, `multi-stream`, `align-stream`), 기록 위치, 시작 경로의 응답, 그리고 `stop()`가 실행되었는지 여부. |

컨텍스트 매니저로 사용하여 범위 종료 시 이를 중지하십시오.

---

## 프로젝트 폴더 및 레시피

**프로젝트 폴더**는 새 프로젝트를 생성하는 폴더로, Chloros 앱, CLI 및 SDK(모듈 `chloros_sdk.system_utils`; `chloros-cli project-folder`)에서 공유됩니다:

```python
print(chloros_sdk.project_folder())                # the configured folder, else the default
chloros_sdk.set_project_folder("D:/Flights")       # created if missing
chloros_sdk.reset_project_folder()                 # -> {"path": <default>, "was_set": True}
```

설정된 폴더가 더 이상 존재하지 않으면 앱과 마찬가지로 기본값(`Documents/MAPIR/Chloros Projects`)으로 돌아갑니다. 백엔드가 이 컴퓨터에 있는 경우, `set_project_folder()`에 지정된 상대 경로는 작업 디렉터리에서 참조됩니다.

**레시피**는 `chloros-cli project run PATH RECIPE`가 저장된 프로젝트에 대해 실행하는 YAML/JSON 작업 목록입니다. `run_project_recipe()`는 CLI의자체 러너(SDK와 함께 설치됨)를 사용하여 실행하므로, 하나의 레시피는 다음 두 경우 모두에서 동일하게 동작합니다:

```python
r = chloros_sdk.run_project_recipe("D:/campaign/morning.yaml", "D:/Projects/Field_A")
print(r["ok"], r["actions"])
print(r["log"])                                    # what the CLI would have printed

chloros_sdk.run_project_recipe({"actions": [{"apply": {"name": "FrontLeft", "settings": {"exposure_us": 5000}}},
                                            {"wait": "2s"},
                                            {"capture": {"name": "FrontLeft", "output": "pose_a/", "format": "tiff"}}]},
                               "D:/Projects/Field_A")
```

```python
run_project_recipe(recipe, project_path, *, dry_run=False, echo=False,
                   backend_url="http://127.0.0.1:5000") -> dict
```

- **`recipe`**는 `.yaml` / `.yml` / `.json` 파일이나 레시피 자체(`actions`(`apply`, `capture`, `burst`, `stream`, `sensor`, `wait`)를 포함하는 딕셔너리이며, 선택적으로 `overrides` 및 `stop_on_error`를 포함할 수 있습니다(기본값 true: 첫 번째 실패한 액션에서 실행이 중지됨). 액션은 CLI 참조 문서의 [Recipe DSL](cli-reference.md#recipe-dsl)에 정의된 바와 같으며, 상대 경로인 `output`는 스크립트의 작업 디렉터리를 기준으로 해석됩니다.
- **반환값** `ok` (모든 액션 성공), `actions` (성공한 액션 수), `log` (실행기의 출력; `echo=True`도 실행 중에 이를 출력함) 및 `validation`를 반환합니다. 레시피 파일이 없거나 구문 분석에 실패하면 `ChlorosConnectError`가 발생합니다.
- **실행 전에 확인됨.**`validation`는 `{"ok", "errors", "warnings", "steps"}`와 같습니다: 모든 액션은 키로 정확히 하나의 알려진 동사(`{"capture": {"name": ...}}`, `{"action": "capture"}`나 `{"type": "capture"}`가 아님)로, 프로젝트에 포함된 올바른 유형의 카메라, 배열 또는 센서를 지정해야 합니다. 오류가 있는 레시피는**아무** 작업도 실행하지 않습니다(`ok` false, `log`의 오류 참조), `stop_on_error`에 무엇이 명시되어 있든 상관없이 실행되지 않습니다. 사용되지 않은 키, 현재 연결되어 있지 않은 레시피가 제어하는 모든 어레이 또는 광 센서는 `warnings`에 해당합니다. 레시피는 장치를 연결하지 않습니다: 프로젝트의 첫 번째 장치를 연결하려면 (`ChlorosProject.connect_all()`, `chloros-cli project connect`, 또는 MCP 도구 `connect_project_devices`)를 통해 직접 연결해야 합니다. `dry_run=True`는 아무것도 실행하지 않고 `{"recipe": ..., "validation": ...}`를 반환합니다.
- 프로세스에서는 한 번에 하나의 레시피만 실행됩니다. YAML 레시피를 사용하려면 PyYAML이 필요합니다.

---

## 프로젝트 자동화 — `ChlorosProject`

저장된 Chloros 프로젝트는 `cameras.json` + `sensors.json` + `project.json`를 포함하는 폴더입니다. `open_project(path, relink=None)`는 매니페스트(`relink`: 이동된 프로젝트의 누락된 소스 파일이 현재 저장된 폴더로, 백엔드가 이를 불러올 때 사용됩니다 — `process()` — **이동된 프로젝트** 참조), 그리고 `connect_all`는 저장된 모든 카메라, 어레이 및 광 센서를 저장된 설정과 함께 온라인 상태로 전환합니다 — 이는 GUI에서 생성되는 하드웨어 상태와 동일합니다. 광 센서는 백엔드의 [DAQ 센서 풀](#daq-sensor-sessions)을 통해 연결되므로, 백엔드가 실행 중이어야 합니다([`SensorHandle`](#sensorhandle) 참조).

### 최소 예제

```python
import chloros_sdk

proj = chloros_sdk.open_project("/home/user/Chloros Projects/Field_A")
report = proj.connect_all(verbose=True)
print(report)  # {'cameras': {...}, 'arrays': {...}, 'sensors': {...}}

# Cameras and arrays are addressable by name OR serial / array_id
cam = proj.cameras["FrontLeft"]
cam.capture("./out", format="tiff", processing="reflectance")

arr = proj.arrays["main_rig"]
arr.capture("./out", format="tiff", processing="reflectance")

# Light sensors are addressable by name OR slot_id
reading = proj.sensors["Sky"].read()   # (spectrum, is_saturated, ...) or None

# One frame from every camera and array, one spectrum from every sensor
proj.capture_all("./out")

proj.disconnect_all()
```

또는 컨텍스트 매니저로 사용:

```python
with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    proj.arrays["main_rig"].capture("./out", processing="reflectance")
```

### `ChlorosProject` 메서드

| 메서드 | 설명 |
| --- | --- |
| `connect_all(*, cameras=True, arrays=True, sensors=True, verbose=False, align=None, packed=None)` | 저장된 모든 카메라, 어레이 및 조명 센서를 검색하고 연결합니다. 각 클래스별 연결 보고서를 반환합니다 (`{"cameras": {...}, "arrays": {...}, "sensors": {...}}`, 각각 `connected`, `missing` 및 `errors`; `arrays`는 `alignment`도 포함합니다). `127.0.0.1:5000`에서 수신 대기 중인 백엔드가 있으면, 저장된 카메라와 어레이를 찾아 이를 통해 연결합니다. 백엔드가 없는 경우, 이 프로세스는 `lattice_sdk`를 통해 직접 카메라와 어레이를 찾아 제어합니다. 이 경우 Arena SDK 런타임과 OpenCV (`cv2`)가 필요합니다. SDK 휠은 이 둘을 설치하지 않습니다. 설치에 실패할 경우, 저장된 모든 카메라와 어레이는 `errors` 아래에 사유가 함께 나열되며, `missing`는 비어 있게 됩니다. 실행 중이지만 SDK를 거부하는 백엔드(로그아웃: HTTP 401; Copper 이하 플랜: 403)는 “백엔드 없음” 상태가 아닙니다. 저장된 모든 카메라, 배열 및 광 센서는 백엔드의 사유가 함께 `errors` 아래에 나열되며, 이 프로세스에서는 아무것도 열려 있지 않습니다. 각 호출 시 백엔드를 다시 확인하므로, 로그인 후 또는 Chloros 시작 후의 `connect_all()`는 동일한 프로젝트 객체에 연결됩니다. 동일한 카메라(GUI의 카메라 또는 `chloros-cli project connect`의 카메라)를 포함하고 백엔드에 이미 활성화된 저장된 어레이는 저장된 설정을 다시 적용하지 않고 있는 그대로 연결되며, `disconnect_all()`는 이를 계속 실행 상태로 둡니다. 광 센서는 항상 백엔드를 통해 처리됩니다([`SensorHandle`](#sensorhandle) 참조). 백엔드를 생성하지 않습니다. |
| `disconnect_all()` | 모든 것을 해제합니다. 먼저 모든 장치를 해제하고, 백엔드가 카메라, 어레이 또는 광 센서의 해제를 확인하지 못한 경우(해제 요청이 시간 초과되었거나 거부된 경우), 각각에 대해 하나의 `ChlorosConnectError`를 할당합니다. 이미 해제된 장치이거나, 또는 백엔드가 실행 중이지 않은 경우, 아무런 응답도 하지 않습니다. 다른 클라이언트가 열어둔 어레이나 광 센서는 계속 실행 상태로 남습니다. 프로젝트의 `with` 종료는 절대 발생하지 않습니다. |
| `capture_all(output_dir=".")` | 연결된 모든 카메라 및 어레이에서 한 프레임씩, 연결된 모든 조명 센서에서 스펙트럼(`SensorHandle.read()`) 하나씩을, 한 번에 모두 수집하는 것이 아니라 순차적으로(카메라, 어레이, 센서 순) 수집합니다. |
| `stream(camera, overlays=False, fps=10.0)` | 지정된 카메라(또는 어레이)에서 BGR `numpy` 프레임을 생성하는 생성기입니다. `overlays=False`는 `grab()` 레벨에서 해당 핸들을 사용하여 프레임을 읽습니다.`grab()`를 사용하여 프레임을 읽습니다. `debayered` 수준에서: 백엔드를 통해 프레임을 읽습니다(프로젝트가 백엔드를 통해 연결된 경우). 그렇지 않은 경우 직접 `lattice_sdk` 캡처 루프를 사용합니다(배열은 `{serial: frame}` 딕셔너리를 반환합니다). `overlays=True`는 `ChlorosLocal.camera_stream()` → 백엔드의 `/api/camera/<serial>/stream-annotated` MJPEG 피드로 라우팅되며, 카메라에 저장된 `ui.overlay` 블록은 쿼리 매개변수로 전달됩니다. 백엔드 모드, OpenCV (`cv2`) 및 **독립형 카메라**가 필요합니다: 다이렉트 모드 카메라는 `RuntimeError`를 발생시키고(백엔드는 이 프로세스가 소유한 카메라를 가져올 수 없음), 배열은 `NotImplementedError`를 발생시킵니다(카메라별로 합성 오버레이 — 이름으로 멤버 스트리밍). 일회성 처리의 경우: `CameraHandle.capture(annotated=True)`. |
| `align_arrays(align=True, verbose=False)` | 현재 연결된 모든 배열에 대해 정렬을 실행합니다. |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | 프로젝트의 이미지에 대해 보정/인덱스 파이프라인을 실행합니다(`ChlorosLocal.process`를 감싸는 함수; 이 네 가지가 **유일하게** 허용되는 키워드 인자입니다 — `indices=` 등은 `TypeError` 예외 발생; `ChlorosLocal.configure()`를 통해 인덱스 설정). `ChlorosLocal()`를 지연 생성하며, 이는 백엔드를 자동으로 시작합니다. |

속성:
- `proj.cameras` — 이름과 일련번호를 키로 하는 `Dict[str, CameraHandle]`.
- `proj.arrays` — 이름과 array_id를 키로 하는 `Dict[str, ArrayHandle]`.
- `proj.sensors` — 이름과 slot_id를 키로 하는 `Dict[str, SensorHandle]`.
- `proj.config` — `project.json["config"]` 사전.

### `CameraHandle`

```python
cam = proj.cameras["FrontLeft"]

# Save a frame to disk (processing-aware)
filepath = cam.capture(
    output_dir="./out",
    format="tiff",
    processing="radiance",           # see the level table below
    apply_calibration=True,          # DSNU + flat + 3x3 unmix + NIST
    apply_white_balance=True,        # DLS-aware WB
    apply_index=False,
    index_expression=None,
)

# In-memory grab (numpy array)
frame = cam.grab(processing="debayered")
frame, header = cam.grab(processing="radiance", with_metadata=True)

# Frame iterator (generator)
for arr in cam.frame_stream(processing="debayered", fps=5, count=100):
    my_analysis(arr)
```

**처리 레벨.** `capture()`, `grab()` 및 `frame_stream()`는 모두 동일한 `processing`
토큰을 사용하며, 체인은 누적적입니다. 즉, 각 레벨은 그 위의 모든 작업을 실행합니다:

| 레벨 | 출력 | 참고 사항 |
| --- | --- | --- |
| `raw` | 1채널 베이어, 센서 네이티브 | 디모자이크 없음. 이 레벨에서는 오버레이를 사용할 수 없습니다. |
| `debayered` | 3채널 컬러 (`CameraHandle` 및 모든 `grab()` / `frame_stream()`; `ArrayHandle.capture()`, `capture_smart()` 및 `stream()`는 `raw`로 기본 설정됨): 메모리 내 BGR (`grab()` / `frame_stream()`), `capture()`에 의해 RGB로 저장됨 | 백엔드 모드에서는 캡처 시: RAW 데이터에서 보정 팩의 다크 오프셋(DSNU)을 제거한 후, OpenCV의 에지 인식 디모자이크를 적용합니다 — 선형성 LUT, 플랫 필드 또는 언믹스 처리는 없습니다. 백엔드 모드 없이 작동하는 유일한 레벨은 카메라 라이브러리의 일반 이선형 디모자이크 (다크 오프셋 없음). 저장된 TIFF 파일은 광도계 RGB이므로, 메모리 내 프레임의 채널을 반전시켜 (`frame[..., ::-1]`)하여 파일과 비교합니다. |
| `radiance` | float32, W/m²/sr/nm, **대역당 하나의 평면** | 전체 방사계측 체인: 디모자이크 + 3×3 언믹스(다중 스펙트럼) + DSNU + 플랫 필드 + NIST 스케일. 노출 × 게인이 제거되어 값이 절대값으로 표시됩니다. |
| `reflectance` | uint16, 32768 = 1.0 (또는 uint8, 255 = 1.0 — `bit_depth` 참조), **밴드당 하나의 플레인** | 방사도를 하향 복사조도(ρ = π·L/E)로 나눈 값. DLS/DAQ 측정값이 필요함 — 아래 참고 사항 참조. |
| `display` | 8비트 sRGB 유사 | GUI와 동등한 렌더링: 카메라의 활성 색상 프로필을 통한 CCM + 화이트 밸런스 + 감마. |

직접(백엔드 미사용) 모드에서는 `debayered`만 사용할 수 있습니다: 다른 레벨의 `grab()`는
`NotImplementedError`를 발생시키고, `capture()`는 `processing`를 무시하며, `frame_stream()`는
`RuntimeError`를 발생시킵니다. `reflectance`에는 사용 가능한 하향 측정값이 필요합니다. 프레임 종점은
풀링된 DAQ를 카메라의 DLS 슬롯으로 자동으로 끌어오지만, DAQ가 바인딩되지 않은 경우 체인은
반사율 출력을 거부하고, 조용히
품질이 낮은 산출물을 반환하는 대신, 반환된 메타데이터에 강등 사실을 명확히 기록합니다.

> **반사율 DN 스케일 — 하드코딩하지 마십시오.** LATTICE 반사율은 `32768` = ρ 1.0을 사용하며
> XMP `Chloros:PixelScale=32768`로 표시합니다. Survey3 반사율은 `65535` = ρ 1.0을 사용하며
> `Chloros:*` 태그를 포함하지 않습니다. 태그 값을 읽어 해당 값으로 나누십시오. 이 값은 uint16 도메인으로 정의되어 있으므로, 재스케일링되는 모든 형식(16비트 TIFF, 8비트 TIFF,
> 8비트 PNG/JPG, 32비트 백분율)에서 `32768`로 그대로 유지됩니다
> — 먼저 저장된 데이터 유형을 uint16으로 정규화하십시오(8비트에서 ×257, 부동 소수점(float)에서 ×65535). 단,
> 1.3.0 이전 빌드에서 생성된 파일은 예외입니다. 당시 8비트 TIFF는 크기 조정이 아닌 *클리핑* 처리되었으므로
> 이를 설명하는 스케일이 없으며, `PixelScale`나 MicaSense 튜플도 포함하지 않습니다.
> LATTICE 반사율 파일에서 태그가 누락된 경우, 이를 기본값으로 간주하지 말고 “유효한 스케일 없음”으로 처리하십시오.

**보정 태그: `raw` 파일에만 해당.** 캡처의 `raw` 파일에는 카메라의 보정 패킷이 포함되어 있습니다: `CalibrationJSON` 페이로드와 MicaSense 호환 태그(`BlackLevel`, `RadiometricCalibration`, `Band{R,G,B}RadiometricCalibration`, `MicaSenseVignetting*`)가 포함되어 있으며, 이는 센서의 원시 값을 설명합니다. 그 외 모든 레벨의 파일(`debayered`, `display`, `radiance`, `reflectance`, `index/` 렌더링) 및 모든 연속 녹화/다중 카메라 스트림 프레임에는 카메라 식별 정보가 포함됩니다 (`XMP-mapir:CameraSerial`, `CameraModel`, `FilterType`, `LensModel`, `CalBundleSha256`, ...) 및 프레임 자체의 기록 정보인 `XMP-mapir:FrameJSON`(노출, 게인, 온도, 픽셀 형식, 캡처 시간, DAQ 매칭)을 포함하며, 모든 `process()` 제품에는 고유 식별자가 포함되어 있습니다. 처리 과정에서는 Chloros의 보정 캐시에서 해당 파일의 일련번호와 `CalBundleSha256`를 기반으로 보정 정보를 찾아내며, 해당 컴퓨터에 파일이 없는 경우 패키지를 한 번 다운로드합니다.

#### 여러 대의 카메라로 생성된 파일

하나 이상의 카메라 밴드로 생성된 파일 — 멀티밴드 스택 (`capture_aligned_multiband()`, `start_aligned_stream()`, `capture_live_index(save_multiband=True)`, `Registration.save_multiband()`), 여러 대의 카메라로 생성된 라이브 인덱스(`index.tif`, `index_LUT.png`), 어레이 캡처의 `composite/` 이미지 — 이 파일에는 해당 파일에 포함된 모든 카메라의 이름이 명시되어 있습니다:

- **카메라별-카메라별 목록, 밴드 순서대로** (합성 이미지: 배경 레이어, 그 다음 전경 레이어; 인덱스 레이어, 해당 밴드가 유래한 카메라): `XMP-mapir:SourceCameraSerials`, `SourceCameraModels`, `SourceSensorProfiles`, `SourceProductLines`, `SourceFilterTypes`, `SourceLensModels`, `SourceCalDateUTCs`, `SourceCalBundleSha256s` (각 카메라의 밴드가 거친 보정 팩, 그렇지 않은 경우 — `capture_aligned_multiband()` / `start_aligned_stream()` 스택으로, 이 스택의 밴드는 어떤 보정 팩도 거치지 않음 — 카메라의 팩인 Chloros가 캐시한 것, 해당 카메라의 캡처 파일 이름과 일치하는 팩), `SourceLightboxCertIds`, `SourceHalogenLampCertIds`, `SourcePackTiers` — 병렬 목록으로, 각 목록의 *n*번째 항목은 동일한 카메라를 설명합니다; `-`는 카메라에 값이 없는 경우(보정 팩 없음, 알 수 없는 렌즈)를 나타냅니다. 각 카메라는 **하나**의 항목으로 구성됩니다: 팩의 특정 필드에 여러 값이 있는 경우 — 팩의 할로겐 램프 인증서 — 이들은 `;` (`F-1479-1;F-1480-1;F-1481-1`)로 연결되며, 쉼표는 절대 사용되지 않으므로, 목록은 출력 방식에 관계없이 카메라당 하나의 항목으로 읽힙니다.
- **EXIF `SerialNumber`**는 동일한 일련번호를 쉼표로 연결하여 저장합니다(`214000535,214000551`).
- 이 파일에는 **단일 카메라 식별 정보**(`XMP-mapir:CameraSerial`, `CalBundleSha256`, …)가 포함되어 있지 않습니다. 해당 식별 정보는 특정 카메라와 그 팩을 지칭하며, 처리 과정에서 이를 바탕으로 파일의 보정 정보를 도출합니다. 여러 대의 카메라 밴드 (`index.tif`, `index_LUT.png`)로 계산된 인덱스는 EXIF `Model` `LATTICE`(리그)를 가지며, 단일 카메라의 `LATT-…` 모델이 아닙니다.**단일** 카메라의 밴드로 구성된 파일 — 밴드별 TIFF, `capture_aligned_multiband(per_camera=True)` 프레임, 단일 카메라의 스택 또는 라이브 인덱스 — 은 다른 모든 캡처 파일과 마찬가지로 해당 카메라 고유의 식별 태그를 포함합니다. 이는 다른 모든 캡처 파일과 마찬가지로 적용됩니다. `capture_aligned_multiband()`의 스택, 밴드별 및 카메라별 파일, 그리고 각 `start_aligned_stream()` 스택과 그 밴드별 파일은 해당 그룹의 캡처 시간 (참조 카메라의 노출 시간, UTC)를 포함하며, 모노 카메라의 `per_camera` 프레임은 스택과 마찬가지로 `bit_depth`에서 하나의 평면을 이룹니다.**카메라 방향(뒤집기/회전).** `grab()` / `frame_stream()`의 프레임은 모든 레벨에서 항상
카메라의 기본 방향을 따릅니다. 저장된 캡처는 카메라의 방향
설정(GUI의 Mirror H / Mirror V / 회전)을 따릅니다:

- **`raw` 및 `debayered` 파일은 설정과 관계없이 센서가 읽은 그대로의 데이터입니다**. 베이어
  모자이크가 회전되거나 뒤집히면 색상 필터 패턴이 달라지며, 보정 맵은
  센서 자체의 그리드에 기반합니다.
- **`display`, `radiance`, `reflectance` 및 인덱스 파일은 방향이 조정된** 상태입니다 — 모든
  보정 계산이 모두 끝난 후 한 번 회전됩니다. 이들의 클램프 마스크는 동일한 그리드에 있습니다.
- **모든 캡처 파일은 XMP에 설정값을 기록합니다**: `XMP-Camera:ChlorosFlipHorizontal` /
  `ChlorosFlipVertical` (`true`/`false`), `ChlorosRotation` (0 / 90 / 180 / 270, 시계 방향,
  반전 후) 및 `ChlorosOrientationApplied` — `false`는 원본 및 디베이어 처리된 이미지에, `true`는 방향이 조정된
  결과물에 적용됩니다. `ChlorosProcessing`는 해당 파일의 픽셀에 적용된 연산들을 나열합니다(네이티브인 경우 `none`).
   여기에는 연속 녹화(카메라 자체 설정,
  처리 미적용) 및 멀티 카메라 스트림(신원: 구성 프레임이
  어레이 캡처와 마찬가지로 네이티브임)의 모든 프레임이 포함됩니다.
- **프로젝트 처리는 기록된 방향을** 원본 데이터에서 생성된 결과물에 적용합니다
  (먼저 센서 좌표계에서 보정, 다음으로 어레이 정렬 워프, 마지막으로 방향 보정),
  따라서 라이브 캡처의 산출물과 일치하게 됩니다 — 보정되지 않은 센서 응답 / 비네팅 보정된
  폴백 산출물도 포함되며, 디베이어 처리된 산출물은 네이티브 상태로 유지됩니다.
- **Chloros 이미지 뷰어는 기록 정보에 따라 회전된 원본 또는 디베이어링된 파일을 표시합니다** — 이는 표시용일 뿐,
  파일 자체는 변경되지 않습니다. 기록 정보가 없는 파일은 저장된 그대로 표시됩니다.
- **기록 정보가 없는 캡처는 네이티브로 처리됩니다.**

1.3.0 이전 버전에서는 단일 카메라 캡처 시
  RAW 파일과 디베이어링된 파일이 자동으로 변환되었습니다. Chloros는 이러한 RAW 파일을 네이티브 파일과 구별할 수 없으며
  추측하지도 않습니다.
- **어레이 캡처는 기본적으로 모든 구성 파일을 네이티브 상태로 유지**하며 식별 정보를 기록합니다;
  `ArraySession.capture(orient=True)`는 해당 결과물을 처리합니다(어레이 캡처 제어 항목 참조).

#### 대역별 출력 및 혼합 어레이

`radiance` 및 `reflectance` 레벨에서 카메라는 **해당 카메라의 대역당 하나의 플레인**을 반환합니다 — M3C의 경우
세 개 (혼합되지 않은 3개 밴드, BGR 채널 순서), M3M의 경우 1개(단일
협대역 필터)를 반환합니다. 모노 카메라의 복제된 평면은 병합되므로, **혼합
M3C/M3M 어레이는 시리얼마다 서로 다른 어레이 형태를 갖습니다**. 각 플레인이 무엇인지 확인하려면 프레임
메타데이터에서 `bands` 키를 참조하십시오. 채널 수만 보고 추측하지 마십시오:

```python
for serial, (arr, hdr) in arr_handle.grab(processing="reflectance",
                                          with_metadata=True).items():
    print(serial, arr.shape, hdr["bands"])   # ('…', (1536, 2048, 3), ['nir','green','red'])
                                             # ('…', (1536, 2048, 1), ['F685'])
    rho = arr.astype("float32") / hdr["pixel_scale"]
```

값을 사용하기 전에 항상 `hdr["processing_level"] == "reflectance"`를 확인하십시오. 등급 하향 조정
(DAQ 판독값 없음, 캘리브레이션 팩 없음, 대역 레이아웃 미확인)은 `processing_level` 및
`radiometric_refused`에 보고되며, 등급 상향 조정이 아닌 경우 — 반환되는 어레이는 여전히 유효한 프레임이지만, 단지
요청한 결과물은 아닙니다. 등급이 하향 조정된 저장된 캡처는 XMP에
`Chloros:RadiometricRefused` (예: `no-band-layout`, `no-dls-snapshot`)와 동일한 코드를 포함합니다.

#### 읽기당 하나의 트리거 — `group_id`

**어레이 구성원은 각 카메라의 개별
그랩 버퍼가 아닌, 어레이의 동기화된 프레임 그룹에서 제공됩니다.** 하드웨어 동기화된 어레이의 카메라들은 함께 노출되지만, 카메라당 N회의 읽기 작업은
함께 *샘플링*되지 않습니다 — 각 카메라의 최신 버퍼링된 프레임을 읽는 작업은 트리거
경계를 넘나들며, 단 하나의 프레임이 누락되어도 아무도 눈치채지 못한 채 전체 세트의 동기화가 깨집니다.

그룹에서 데이터를 제공하는 것은 필요하지만, 그것만으로는 충분하지 않습니다. 각 요청은 여전히
자신의 순간에 *가장 최신* 그룹을 스냅샷으로 캡처하며, 그 사이에 그랩 루프가 프레임을 발행할 수 있습니다.
실제 4대 카메라 어레이에서 측정한 결과, **병렬 읽기 20건 중 12건이 두 그룹에 걸쳐 있었으며** — 항상
정확히 인접한 그룹이었습니다. 따라서 읽기는 **고정**됩니다:

1. 모든 프레임에는 어레이의 단조 증가 퍼블리시 카운터인 `group_id`가 포함됩니다.
2. `grab()`는 마스터를 읽은 후(마스터를 처리할 수 없는 경우, 처리 가능한 다음 카메라를 읽은 후),
   나머지 카메라들을 해당 그룹 ID(HTTP 엔드포인트의 `?group_id=N`)에
   고정시킵니다.
3. 백엔드는 최근 그룹을 **최소 0.5초** 동안 유지하므로, 후속 퍼블리시에 의해 이미
   추월당한 형제 그룹도 해당 트리거에 대해 계속 서비스됩니다. 이는 9대의 카메라로 구성된 풀 센서 어레이에서도
   반사율 내보내기의 왕복 시간을 감당할 수 있을 만큼 충분한 시간입니다. 백엔드가 더 이상
   유지하지 않는 할당은, 다른 트리거로 묵묵히 응답하는 대신 `409` 및 `group_aged_out: true`로 거부됩니다
   다른 트리거로 응답하는 대신 — `grab()`는 세트를 다시 읽습니다.

핀 설정을 적용하여 재측정한 결과: **20회 읽기 중 20회 모두 하나의 트리거를 반환했습니다.** `grab()`가 이를 자동으로 처리해 주므로,
HTTP 엔드포인트를 직접 호출할 때만 `group_id=`가 필요합니다.

- 고정된 그룹에서 서비스를 제공할 수 없는 카메라(해당 그룹에 없거나 요청을 거부하는 경우
  (가지고 있지 않은 `band`, 생성할 수 없는 레벨))는 **다른 트리거로 채워지는 대신 생략되며**
  다른 트리거에서 생략되며, 나머지 카메라들은 여전히 반환됩니다.
- `grab()`가 반환하는 딕셔너리는 `GrabResult` (일반 `dict` 하위 클래스;
  `from chloros_sdk import GrabResult`)입니다: `.errors`는 생략된 각
  시리얼을 해당 읽기 작업에서 발생한 예외에 매핑하며, `.group_id`는 공유 그룹입니다. 빈
  `.errors`는 모든 카메라가 이 그룹을 구성했음을 의미합니다.
- 독립형 카메라는 그룹이 없으며, `group_id: None`를 보고하고, `group_id=`
  핀을 무시하지 않고 **거부**합니다.

```python
frames = arr_handle.grab(processing="reflectance")
for serial, err in frames.errors.items():
    print(f"SN:{serial} omitted: {err}")       # e.g. status=400 ... band 'F685' not in [...]

frames = arr_handle.grab(processing="reflectance", with_metadata=True)
ids = {hdr["group_id"] for _a, hdr in frames.values()}
assert len(ids) == 1, f"straddled triggers: {ids}"     # belt and braces
```

배열 구성원의 **모든 카메라별 MJPEG 스트림**(미리보기(`/api/camera/stream/<serial>`), 주석이 달린 미리보기(`/stream-annotated`) 및
반사율 스트림(`/stream-reflectance`))에 동일한 카운터가 게시되며,
반사율 스트림(`/stream-reflectance`) — 각 부분에서 `X-Chloros-Group-Id`로, `X-Chloros-Frame-Id` 및 `X-Chloros-Timestamp-Ns`와
병행하여 제공됩니다. 스트림은 특정 시점의 데이터가 아닌
연속적인 데이터이므로, 파트를 수집하고 해당 헤더를 기준으로 그룹화하여 매칭됩니다. 별도로
설정해야 할 핀은 없습니다. 독립형 카메라에는 그룹이 없습니다. 자세한 내용은
[카메라별 MJPEG 스트림](#per-camera-mjpeg-streams-no-rtsp)을 참조하십시오.

**`frame_stream()`는 절대 침묵하지 않습니다.** 스스로 해결될 수 있는 이유로 실패한 틱은
건너뜁니다 — 카메라 스트림은 해당 틱에 대해 아무것도 반환하지 않고, 어레이 스트림은 응답한
카메라들을 반환합니다 (아무것도 없을 수도 있음). 이러한 이유는 다음과 같습니다: 아직 프레임이 없음; 카메라의 그랩 스레드가
1~2초 동안 일시 중지됨(`404 not streaming`, 이는 다른 카메라가 연결되거나
연결이 끊어질 때, 또는 설정 변경으로 인해 풀의 균형이 재조정될 때마다 백엔드가 응답함); 백엔드 재시작 또는 `5xx`; 정렬이
아직 안정화 중 (`409 … retry shortly`); DAQ에 아직 측정값이 없어 반사율 요청이 거부된 경우
(`no-dls-snapshot`, 오래된 측정값 또는 빈 측정값). 다음 틱에서 거부 오류가 발생하면 동일한 오류가 반복될 뿐입니다 —
알 수 없는 `band`, `bit_depth=8`(방사도 관련), 방사도 척도가 없는 보정 팩, **DAQ가 전혀 없는*
반사율 거부* (카메라나 어레이에 바인딩된 DAQ가 없거나, 백엔드에 연결된
DAQ가 없는 경우: 거부된 데이터의 `dls_source`는 `"none"`임), 그 외의 모든 `4xx` —
`chloros_sdk.FrameFetchError`를 발생시킴 (`RuntimeError`, `.status` 및 `.body`를 포함)을
한 번에 발생시킵니다. 또한, 그 사이에 프레임이 없는 **60초** 동안 틱이 건너뛴 경우에도 마찬가지이며, `fps`가 무엇이든 간에 — 정렬 안정화와 백엔드 재시작을 견딜 수 있을 만큼
충분히 길면서도, 실제로
스트림을 종료할 수 있을 만큼 충분히 짧은 시간입니다. 이 포기 오류 역시 `FrameFetchError`이며, 카메라든
어레이든 상관없이 적용되며, 마지막 읽기 작업 자체의 예외는 `.errors` (카메라의 경우 `__cause__`).
어레이의 경우, “거부됨”은 모든 카메라가 거부되었음을 의미하며, 예외인 `.errors`에는 각 카메라의
거부 사유가 포함됩니다.

**배열 `frame_stream()`는 각 트리거를 한 번씩 반환합니다** (`ArrayHandle` 및 `ArraySession`도 마찬가지입니다).
가장 최근 세트가 이미 반환된 트리거인 틱은 동일한 세트를 다시 반환하는 대신
다음 트리거를 기다리므로, 스트림은 `fps`와 배열 자체의 프레임
중 더 느린 속도로 실행되며, 연속된 세트는 절대 `group_id`를 공유하지 않습니다. 퍼블리싱을 중단하는 배열은
`FrameFetchError`("60초 동안 새로운 트리거 없음")를 발생시키며, 마지막 세트를 반복하지 않습니다. 이 기능이 없다면,
배열이 게시한 양보다 더 많은 데이터를 요청받은 스트림 — 또는 연결 직후 시작된 스트림 — 은
반복 데이터를 제공하게 됩니다: 한 번의 측정 실행에서, 20개의 세트가 11개의 서로 다른 그룹 ID를 포함했습니다.

```python
from chloros_sdk import FrameFetchError

try:
    for arr in cam.frame_stream(processing="radiance", bit_depth=8):
        my_analysis(arr)
except FrameFetchError as exc:
    print(exc.status, exc.body)        # the HTTP status and the backend's reason
```

#### 단일 대역 내보내기 — `band`

`grab()` / `frame_stream()`는 `band="<name>"`를 사용하여 카메라의 밴드 중 하나만을
단일 채널 `(H, W, 1)` 배열 — 모노 카메라가 생성하는 것과 동일한 형상이므로, 단일 밴드 출력은
M3M에서 나온 것이든 M3C를 분할한 것이든 동일하게 처리됩니다. FRGN의 적색 밴드:

```python
red = cam.grab(processing="reflectance", band="red")        # (1536, 2048, 1)
```

허용되는 표기법은 전체 레이아웃 이름(`Red_660`, `NIR_850`, `Green_550`)이나 그 약칭인
헤드 (`red`, `nir`, `green`)이며, 대소문자를 구분하지 않습니다. 알 수 없는 이름은 사용 가능한
이름 목록과 함께 거부됩니다.

> **선택은 채널 번호가 아닌 이름으로 이루어지며, 이는 의도된 것입니다.** "채널 1"은 어레이의 BGR 채널 순서에서는 `Green_550`이지만,
> 필터 코드의(`RGN`)에서는 *빨간색*이며, 이 둘은
> 일치하지 않습니다. 잘못된 것을 선택하면 잘못된 대역의 완벽하게 그럴듯한 프레임이 생성되는데, 이는
> 하류에서 감지할 수 없습니다. 이러한 이유로 숫자 인덱스는 존재하지 않습니다.

#### 스트림 정렬 — `align`

`align=True`는 프레임을 배열의 공통 기준 프레임으로 변환합니다. 각 카메라는
**동일한** 프로필 해상도와 크롭 ROI를 통해 변환되므로, 따라서 독립적으로 열린 카메라별 스트림은**서로 정렬된** 상태로 출력됩니다 — 함께 요청할 필요가 없습니다:

```python
frames = arr_handle.grab(processing="reflectance", align=True)   # {serial: array}
# or one MJPEG stream per camera, each co-registered on its own:
#   GET /api/camera/<serial>/stream-reflectance?align=1
```

요구 사항 및 동작:

- 카메라는 정렬
  프로파일이 있는 **하드웨어 동기화** 어레이(M8 동기화 케이블)에 속해 있어야 합니다. — 공동 정렬은 구성 요소들이 동시에 노출된다고 가정하며, 이는 소프트웨어나
  자유 실행 어레이는 이를 제공하지 않습니다. 그렇지 않은 경우 `409` 오류로 거부됩니다.
- **거부되며, 절대 조용히 정렬되지 않은 상태로 처리되지 않습니다.** 캡처 경로는 경고와 함께 정렬되지 않은 상태로 저장될 수 있습니다.
  이는 파일이 출처 태그를 유지하여 나중에 재정렬할 수 있기 때문입니다. 라이브 스트림에는
  이러한 두 번째 기회가 없으므로, 프로파일이 누락된 경우 대체 처리 대신 오류로 간주됩니다.
- **정렬을 요청하면 정렬이 시작됩니다.** 프로파일이 없는 어레이에서 `align`를 요청하면
  보정이 시작되고, 대기 중인 내용(예: *"노출이
  214000551 (1/3)에서 안정화되기를 대기 중"*) — 재시도하면 일반적으로 수십 초 이내에 성공합니다.
  이는 헤드리스 환경에서도 중요합니다: 연결 시 자동 정렬은 30초 후에 중단되며,
  시도들은 건너뛰어집니다. 따라서 노출이
  천천히 수렴하는 어레이의 경우, 전체 세션에 대한 프로필을 얻지 못했고, 데스크톱 통합 보기를 열지 않고서는
  프로필을 얻을 방법이 없었습니다. `alignment_status`는 또한
  `ae_align_holdouts`를 보고하며, 아직 안정화 중인 카메라와 그 진행 상황을 명시합니다.
- 하나의 어레이에 속한 모든 스트림에 **동일한 `max_h`**를 부여해야 합니다. 그렇지 않으면 각 스트림의 크롭이 서로 다른 픽셀
  그리드에 위치하게 됩니다.
- 정렬은 비트 심도 인코딩 *이전*에 적용되므로, 워프는 전체 정밀도
  값을 보간합니다.

#### `bit_depth` — 정밀도와 대역폭의 절충

`grab()` 및 `frame_stream()`는 `bit_depth=16`(기본값) 또는 `bit_depth=8`를 지원합니다:

| `bit_depth` | 반사율 인코딩 | 비용 |
| --- | --- | --- |
| `16` | uint16, 32768 = ρ 1.0, ρ 2.0까지의 여유 | 전체 정밀도. |
| `8` | uint8 (`0..refl_max` (기본값 ρ 1.0) | 바이트 수 절반; 기본 상한값에서 ~0.004 ρ 양자화. 상한값을 초과하는 값은 클리핑됨 — 아래 참조. |

##### 8비트 상한 — `refl_max`

`refl_max`는 8비트 범위가 포괄하는 ρ를 설정하므로, 따라서 `pixel_scale = 255 / refl_max`와 ρ는 여전히
`pixel / pixel_scale`입니다. `grab()`와 `frame_stream()`는 카메라와
배열 핸들(HTTP 엔드포인트의 `?refl_max=`)에서 모두 키워드로 처리합니다. 이를 높여 스페큘러
하이라이트를 위한 여유 공간을 확보하고, 단계는 비례적으로 더 거칠게 조정합니다:

```python
frame = cam.grab(processing="reflectance", bit_depth=8, refl_max=1.5)   # 0..1.5 in 0..255
```

| `refl_max` | `pixel_scale` | 단계 |
| --- | --- | --- |
| `1.0` (기본값) | 255 | 0.0039 ρ |
| `1.5` | 170 | 0.0059 ρ |
| `2.0` | 127.5 | 0.0078 ρ |

> **자동 노출은 8비트 클리핑을 수정할 수 없으며, 이를 시도하면 데이터 품질이 오히려 악화됩니다.**
> ρ = π·L/E이며, 여기서는 노출과 게인이 이미 고려된 값이므로 ρ는 카메라 설정이 아닌
> 장면의 특성입니다. 라이브 어레이에서 측정한 결과, 노출을 절반으로 줄였을 때 평균 ρ가 1.3% 변동했습니다.
> 노출을 높여 범위에 “맞추려고” 하면 오히려 픽셀이 **센서** 포화 상태에 이르게 되며, 이는
> L을 과소평가하고 오류 메시지 없이 ρ를 **낮게** 편향시킵니다(측정 결과: 최대 ρ 1.203 → 0.985,
> 노출 2배 시, 평균 −23%). `refl_max`로 범위를 선택하십시오. 센서가
> 포화 상태에 이르지 않도록 하는 것은 이미 AE의 역할이며, 이는 ρ의 범위보다는 *타당성*을 보호합니다.

**`reflectance_clipped_fraction`를 읽어보십시오, 플래그가 아닙니다.** `reflectance_clipped_at_1_0`는
단일 픽셀로 인해 오류가 발생합니다 — 실제 장면에서 비율은 `0.000001`였음에도 `True`를 보고했으며, 이는 백만 개 중 하나의 픽셀
발생할 확률이 백만 분의 1에 불과한 경우입니다. 상한값을 높여야 할지 여부는 이 비율을 통해 판단합니다.

**`bit_depth=8`는 반사율 전용입니다.** `radiance`에 대해 이를 요청하면 처리되지 않고
거부됩니다: 반사율은 8비트 인코딩을 견딜 수 있는 0~1의 고정된 범위를 갖는 반면, 복사도는
고정된 범위가 없기 때문에 8비트에 맞추려면 프레임마다 백분위수 스트레칭을 적용해야 하며, 이 매핑은
매 프레임마다 변경됩니다. 이는 측정값이 아닌 단순한 이미지일 뿐입니다.

#### 실시간 반사율 스트림 (MJPEG)

`GET /api/camera/<serial>/stream-reflectance`는 밴드당 8비트 반사율을
`multipart/x-mixed-replace`로 제공합니다. 즉, 3채널 JPEG 형식의 M3C 또는 그레이스케일 형식의 M3M입니다. 쿼리 매개변수:
`fps`(기본값 10), `jpeg_q`(기본값 95), `max_h` (0 = 네이티브), `band` (이름 지정된 밴드 1개,
위와 동일), `align` (`1`를 공동 등록 — SDK의 `align`와 동일한 규칙), `refl_max` (8비트
인코딩 상한, 위와 동일). 각 파트는
`X-Chloros-Group-Id`, `X-Chloros-Frame-Id`, `X-Chloros-Timestamp-Ns`, `X-Chloros-Bands`,
`X-Chloros-Pixel-Scale`, `X-Chloros-Aligned` 및 `X-Chloros-Reflectance-Clipped`를 포함합니다.
**서로 다른 카메라에서 동일한 `X-Chloros-Group-Id`를 포함하는 파트는 동일한 트리거입니다** —
이것이 하나의 어레이에 대한 카메라별 여러 스트림이 재조립되는 방식이며, 어레이 구성 요소가
동기화된 프레임 그룹에서 제공되므로 서버 측에서 보장됩니다(참조:
[읽기당 하나의 트리거](#one-trigger-per-read--group_id) 참조). 연결 시점에 반사율 데이터를
획득할 수 없을 경우 `409`를 반환하며, 실행 도중 반사율 데이터가
사용 불가능해지면 스트림을 종료하므로, 중단된
스트림은 “여전히 정상이지만 정적 상태”가 아니라 “더 이상 반사율이 없음”을 의미합니다.

> **JPEG는 손실 압축 방식이므로, 이 픽셀들은 측정값에 불과합니다.** 압축 오류는
> 반사도 값에 직접 반영되며, 재료 경계가 있는 가장자리에서 가장 심각하게 나타납니다. 이러한 이유로 품질
> 기본값은 95로 설정되어 있습니다. 수치를 그대로 유지해야 하는 경우,
> `grab(processing="reflectance")` / `frame_stream(...)`를 사용하십시오. 이 명령어들은 경로에 코덱이 없는 상태에서
> 동일한 결과를 반환합니다(코덱이 아닌 전송 비용이 제약 조건인 경우에는 `bit_depth=8`를 사용하십시오).

> **EXIF 정보가 내보내기 파일에 그대로 반영됩니다.** `process()`는 소스 캡처의 GPS 블록
> **및 해당 ExifIFD**를 모든 산출물에 복사하므로, 내보낸 파일에는 `FocalLength`, `FNumber`,
> `ExposureTime`, `ISO`, `DateTimeOriginal` 및 `CameraSerialNumber`와 더불어
> 지리 참조 정보도 포함됩니다. `FocalLength`는 Pix4D가 지상 샘플 거리(GSD)를 계산하는 데 사용하는 데이터로, 이 데이터가 없으면
> 재구성 결과의 축척이 극도로 부정확해집니다(실제 측정 사례에서 411m 규모의 현장이
> 47.8km 규모로 변한 경우). 이 파일은 의도적으로 `-all:all`가 아닙니다. IFD0의 구조 태그가
> LATTICE 출력을 깨뜨리기 때문이며, `ExifImageWidth`/`Height`는 내보낸 래스터가 아닌 원본
> 캡처 정보를 기술하므로 제외되었습니다.

캡처 단계 하위 플래그(방사계 수준에 적용됨 — `radiance`, `reflectance`, `display`):

| 플래그 | 기본값 | 의미 |
| --- | --- | --- |
| `apply_calibration` | `True` | DSNU + 플랫 필드 + 3x3 언믹스 + NIST 방사계 척도. |
| `apply_white_balance` | `True` | WB LUT. DAQ가 카메라에 바인딩된 경우 DLS를 고려함. |
| `apply_index` | `False` | 식생 지수 평가. |
| `index_expression` | `None` | 공식 재정의. 비비어 있지 않을 경우 → 지수 자동 활성화. |
| `annotated` | `False` | GUI 장식(제브라/그리드/피킹) 오버레이. `raw`에서는 사용할 수 없음. |

### `ArrayHandle`

```python
arr = proj.arrays["main_rig"]

# Single synced capture group
files = arr.capture("./out", format="tiff", processing="reflectance")
# → {"213800234": "/path/to/x.tif", "214000533": "/path/to/y.tif", ...}

# Multi-level: each serial's value becomes an ordered LIST, not a str
files = arr.capture("./out", processing="all")
# → {"213800234": ["/raw.tif", "/debayered.tif", ...], "combined": "/idx.tif"}

# Smart capture (wait for AE to settle)
result = arr.capture_smart(
    "./out", processing="reflectance",
    settle_timeout_s=5.0,
    stability_window_s=1.5,
    exposure_tolerance_pct=5.0,
)
print(result["frames"], result["settle"])

# In-memory grab: {serial: numpy array}
frames = arr.grab(processing="debayered")
frames = arr.grab(processing="radiance", with_metadata=True)

# Stream-to-disk loop
arr.stream(count=60, output_dir="./stream", fps=5, processing="raw")

# Frame-iterator (tolerates per-cam drops; great for downstream analysis pipelines)
for frames in arr.frame_stream(processing="radiance", fps=5, count=100):
    if "213800234" in frames:
        my_analysis_pipeline(frames["213800234"])

# Preview iterator (live MJPEG-equivalent; tolerates partial cycles)
counts = arr.preview_stream("./preview", fps=3.0, duration=30.0)
print(counts)  # frames written per serial
```

> **반환 값은 `chloros_sdk.CapturePaths`이며, 타입은 `CapturePathMap`이며, `Dict[str, str]`가 아닙니다.**
> `chloros_sdk.CapturePathMap`는 `Dict[str, Union[str, List[str]]]`입니다: 단일 레벨
> `processing`는 각 시리얼에 하나의 경로를 제공하는 반면, `processing="all"`는 해당 카메라에 대해 저장된 모든 제품의
> **정렬된 목록**을 제공합니다. `ArrayHandle.capture()`는
> `levels=`를 받지 않으며, 알 수 없는 키워드 인자(`levels`, `daq`,
> `render_index`, `force_daq`)는 인식하지 못하므로, 여기에 표시된 것들만 전달하십시오 (`aligned=False`
>는 `ArraySession.capture()`와 마찬가지로 네이티브 파일을 저장합니다); 해당 컨트롤에는
> `ArraySession.capture()`를 사용하십시오. 스트리밍 중이던 라이브 결합 컴포지트는 시리얼 아래가 아닌 별도의
> `"combined"` 키 아래로 전달되며, 시리얼 키 아래로는 전달되지 않습니다. `str`를 가정하는 코드는
> 타입 검사기가 아무런 오류도 표시하지 않는 목록 형식에서 오류가 발생합니다 — 주석에 따르면 `Dict[str, str]`
> 이라는 주석이 있었기 때문에 이 별칭이 존재합니다. 플랫 형식을 원할 때는
> 정규화하십시오:
>
> ```python
> paths = arr.capture(processing="all")
> flat = [p for v in paths.values()
>         for p in (v if isinstance(v, list) else [v])]
> ```

> **다음과 같이 작성할 수 없는 경우: `.failed`.** `capture()`가 반환하는 맵은
> `chloros_sdk.CapturePaths` — 단순한 `dict` 하위 클래스이므로 위의 모든 내용은
> 변경 없이 작동합니다 — 이 클래스는 또한 `.failed`를 포함합니다: 백엔드가 쓰지 못한 파일들,
> 다음 항목과 동일한 항목들입니다 [`CaptureResult.failed`](#files-that-could-not-be-written--captureresultfailed)
> (`serial`, `level`, `error`, `disk_full: True`)와 동일한 항목입니다. 부분
> 캡처에서는 여전히 기록된 모든 경로가 반환되지만, 드라이브가 가득 찬 경우 맵은
> 비어 있으며 `.failed` 파일에 그 이유가 명시되어 있습니다. `.skipped`는 카메라별 건너뜀 항목을 나열하고
> `.group_seq`는 동기화된 그룹을, `.alignment` 및 `.oriented`는
> 백엔드 관련 정보이며, `CaptureResult`에서도 마찬가지입니다. `capture_smart()`는 이를
> `result["frames"].failed` 및 `result["failed"]`로 반환합니다.
>
> ```python
> paths = arr.capture("./out", processing="all")
> if any(f.get("disk_full") for f in paths.failed):
>     raise SystemExit("drive full -- free space and capture again")
> ```

### 배열 정렬

`ArrayHandle`는 전체 정렬 영역을 노출합니다. 프로파일은 기본적으로 세션 단위로만 유지되므로, 영구적으로 저장하려면 `export_alignment()`를 명시적으로 호출해야 합니다.

```python
from chloros_sdk import AlignmentSpec

arr = proj.arrays["main_rig"]

# Defaults: ORB / affine / one synced snapshot — same as the GUI's auto-cal
result = arr.calibrate_alignment()
print(result["profile"]["rms_residual_px"])

# Custom spec for tough scenes (low-contrast canopy)
spec = AlignmentSpec(
    method="feature_orb",         # feature_orb / feature_akaze / phase_correlation / checkerboard / manual
    model="rigid",                # translation / rigid / affine / homography
    num_frames=5,
    max_features=8000,
    ratio_threshold=0.7,
    ransac_threshold_px=2.0,
    min_matches=30,
    max_reproj_err_px=2.0,
)
arr.calibrate_alignment(spec)

# Or tweak one knob at a time
arr.calibrate_alignment(num_frames=3, model="affine")

# Manual control points: each camera's pixel coordinates of the same
# features, the reference camera's included (>= 2 per camera). Every one
# of num_frames is solved from these points.
arr.calibrate_alignment(method="manual", model="rigid", control_points={
    "214701173": [[412, 300], [1630, 310], [1020, 1180]],   # reference (master)
    "214701292": [[418, 297], [1637, 306], [1027, 1177]],
})

# Inspect / manipulate
status = arr.alignment_status()
arr.tweak_alignment("214701292", dx=2.5, dy=-1.0, rotation_deg=0.0, scale=1.0)
arr.export_alignment("/tmp/main_rig_alignment.json")
arr.import_alignment("/tmp/main_rig_alignment.json", validate=True)
arr.clear_alignment()
```

#### 연결 시 정렬

`connect_all(align=...)`는 연결 시 모든 어레이를 자동으로 정렬할 수 있습니다:

```python
# Align every array with defaults
proj.connect_all(align=True)

# Per-array control
proj.connect_all(align={
    "main_rig": AlignmentSpec(num_frames=5, model="affine"),
    "side_rig": True,             # use defaults
    "verify_rig": False,          # skip
})
```

지정되지 않은 경우 `project.json["config"]["auto_align_on_connect"]`로 대체됩니다.

### `SensorHandle`

백엔드의 [DAQ 센서 풀](#daq-sensor-sessions)에서 `connect_all()`에 의해 열린 저장된 광 센서입니다. 이 풀은 `connect_daq_sensor()` 및 `chloros-cli daq pool-connect`가 사용하는 것과 동일합니다.

```python
reading = proj.sensors["Sky"].read()
if reading is not None:
    spectrum, is_saturated, integration_time, x, y, z = reading
```

- `read()`는 센서의 **다음** 스펙트럼을 기다립니다. 호출 시점에 이미 버퍼링된 스펙트럼은 포함되지 않습니다 —를 반환하며, 5초 이내에 도착하지 않으면 `(spectrum, is_saturated, integration_time, x, y, z)` 또는 `None`를 반환합니다. 일시 중지된 스트림이 재개됩니다. 센서가 보정된 경우 `spectrum`는 W/m²/nm 단위입니다.
- `.sensor`는 센서의 [`DAQSensorSession`](#DAQ 센서 세션)이며, 그 외의 모든 경우에는 `latest(n=...)`, `status()`, `record_start()` / `record_stop()`입니다.
- `.slot_id`, `.model`, `.serial`, `.name` 및 `.settings`는 `sensors.json`에서 저장된 항목입니다.

**`connect_all`가 복원하는 내용.** 각 센서는 저장된 주소(COM 포트, 블루투스 주소 또는 DAQ-E 호스트)에서 열리며, 프로젝트에 센서 ID가 기록된 경우 해당 센서여야 합니다. 해당 주소에 다른 장치가 있으면 연결이 거부됩니다. 저장된 적분 시간, 프레임 평균화 및 자동 노출 설정이 적용되며, 캡 보정값도 적용됩니다:

| 저장된 캡 | 연결 대상 |
| --- | --- |
| 캡 프로필 (예: `fov_30`) | 해당 프로필. |
| 선택되지 않음 | `sunshine_cosine`(연결 시 기본값) — 가정된 값(`cap_id_source=auto_default`)으로 기록되어 처리 과정에서 경고가 발생합니다. 센서는 캡이 장착되지 않은 상태로 출하되므로, 실제로 장착된 캡을 선택하십시오. |
| `none` — GUI에서 **없음(베어 센서)**으로 선택됨 | 캡 보정 없음. |

DAQ-E-S는 저장된 내용과 관계없이 캡 프로필을 적용하지 않습니다. 이 센서의 디퓨저는 보정 과정의 일부입니다([DAQ-E-S](#daq-e-s--fixed-diffuser-and-tilt-sensor) 참조). 이전 버전의 Chloros로 저장된 프로젝트는 선택 사항이 없을 때 노출된 센서를 기록했기 때문에, `sunshine_cosine`와 다시 연결됩니다. GUI에서 다시 **없음(노출된 센서)**을 선택하고 프로젝트를 저장하십시오.

다른 클라이언트(CLI, 다른 스크립트 등)가 풀에서 이미 열어 놓은 센서는 설정을 다시 적용하지 않고 있는 그대로 연결되며, `disconnect_all()`는 이를 계속 실행 상태로 유지하며, `connect_all()`가 연 센서는 해제됩니다. `connect_daq_sensor()`의 경우와 마찬가지로, Chloros GUI는 풀의 일부가 **아닙니다**: 스크립트를 통해 프로젝트를 연결하기 전에 GUI에서 센서를 닫으십시오.**요구 사항 및 보고서.** 조도 센서를 사용하려면 유료 Chloros+ 티어(Copper 이상)에 로그인된 Chloros 백엔드가 실행 중이어야 하며, `connect_all()`는 이를 절대 시작하지 않습니다. 실행 중인 백엔드가 없으면, 저장된 모든 센서가 `report["sensors"]["errors"]` 아래에 해당 내용이 명시된 상태로 나열됩니다. 백엔드가 열 수 없는 센서도 백엔드의 오류 사유와 함께 해당 목록에 표시되며, `report["sensors"]["missing"]`는 광 센서의 경우 비어 있습니다.

---

## 직접 하드웨어(백엔드 미사용)

LATTICE 카메라(CI, 헤드리스 로봇, 임베디드)에 대해 백엔드에 전혀 의존하지 않으려면 `lattice_sdk`를 직접 임포트하십시오. 이 모듈은 `chloros_sdk`에 의해 재수출됩니다. `CAMERA_AVAILABLE`에 대한 주의 사항: `lattice_sdk`는 SDK 휠에 포함되어 있지만, Arena SDK 런타임과 OpenCV(`cv2`)가 설치되어 있어야 하며, 휠은 이 둘을 설치하지 않습니다. 광 센서는 백엔드를 통해 작동합니다 — [`connect_daq_sensor()`](#daq-sensor-sessions) 또는 프로젝트의 `proj.sensors`를 통해 처리되지만, DAQ-M 대역-진폭 헬퍼 [`DLS`](#daq-m-band-magnitudes--dlsband_magnitudes)는 예외이며, 이 헬퍼는 `bleak`도 필요합니다.

```python
from chloros_sdk import (
    # cameras
    LatticeCamera, CameraSettings, PRESETS, CameraPool,
    Calibration, CalibrationCoefficients, FilterModel, list_filters,
    DLS, NetworkDiagnostics, gpu_info, gpu_available,
    # discovery
    discover_cameras, discover_cameras_via_backend,
    # exceptions
    LatticeError, CameraNotFoundError, StreamError, CaptureError,
    CalibrationError, NetworkError, DLSError,
)

# Find a camera and capture in one go
cams = discover_cameras(timeout_ms=3000)
print(cams)

settings = PRESETS["high_quality"]
with LatticeCamera(serial="213800234", settings=settings) as cam:
    result = cam.capture(output_dir="./out", format="tiff")
    print(result.filepath, result.width, result.height)
```

##### 프리셋과 트리거

네 가지 프리셋 중 세 가지는 **자유 실행(free-run)** 모드입니다. 이 경우 카메라는 지속적으로 노출을 진행하며,
`capture()`가 다음 프레임을 반환합니다. `triggered`는 예외로 — 이 프리셋은
2번 라인의 하드웨어 에지가 감지될 때까지 카메라를 대기 상태로 유지하므로, 에지가 감지될 때까지는 아무것도 촬영하지 않습니다.

| 프리셋 | 트리거 | 사용 시점 |
| --- | --- | --- |
| `default` | 프리런 | 일반 용도 |
| `high_speed` | 프리런 | 8비트, 60fps 제한, 짧은 노출 |
| `high_quality` | 프리런 | 12비트, fps 제한 없음 — 스틸 사진 촬영 시 일반적으로 선택 |
| `triggered` | **준비 상태, 라인 2** | 카메라가 M8 싱크 케이블에 연결되어 있으며 다른 장치가 셔터를 작동시킴 |

`triggered`를 선택하거나(또는 직접 `trigger_mode="On"`를 설정할 경우) 라인 2를
구동하는 것이 아무것도 없다면, 모든 `capture()`는 타임아웃이 발생합니다 — 이는 카메라에 대기하도록 지시했기 때문에
정당한 현상입니다. SDK에서는 이러한 상황이 발생할 때 이에 대해 설명하고 있습니다.
[캡처 중 SC_ERR_TIMEOUT](#direct-hardware-backend-free)를 참조하십시오.

> **참고 — 연결 시 나타나는 "GVSP probe" / `SC_ERR_TIMEOUT -1011` 메시지는 오류가 아닙니다.**
> 연결 시 SDK는 더 높은 처리량을 위해**점보 프레임**(9000바이트 GVSP 패킷) 협상을 시도합니다. 직접적인 지점 간(point-to-point) NIC 링크(예: 링크 로컬 `169.254.x.x` 주소)에서는 네트워크가 일반적으로 점보 프레임을 전송할 수 없으므로, 이 프로브가 타임아웃되고 다음과 같은 로그가 기록됩니다:
>
> ```
> [Network] GVSP probe: unexpected error (TimeoutError: ... SC_ERR_TIMEOUT -1011)
> [Network] GVSP probe at 9000 did not deliver a complete buffer; reverting to ICMP-chosen size
> [Network] GVSP packet size: 1500 bytes (standard)
> ```
>
> 이것이 **설계된 대체 조치**입니다. SDK는 자동으로 표준 1500바이트 패킷으로 되돌아가며, 카메라는 정상적으로 연결을 유지합니다(그 뒤에 나오는 `[chunk-enable …]` 줄들은 정상적인 연결 순서의 일부입니다). 캡처 기능은 여전히 작동합니다.
>
> 이 프로브를 건너뛸 수는 있지만, **이 프로브는 단순히 로그 출력을 억제하는 것이 아니라 점보 프레임을 비활성화합니다.** 네트워크 상태가 아무리 양호하더라도 카메라는 최대 1500바이트까지만 ‘Don't-Fragment’ 핑에 응답하므로, 핑 테스트만으로는 점보 프레임을 절대 감지할 수 없습니다. 이를 감지할 수 있는 유일한 방법은 이 프로브뿐입니다. 이 프로브를 비활성화하면 카메라는 어떤 네트워크에서든 영구적으로 표준 1500바이트 패킷을 전송하게 됩니다: 어떤 네트워크에서든:
>
> ```bash
> CHLOROS_GVSP_PROBE_FALLBACK=0   # gives up jumbo — see the warning it prints
> ```
>
> 이 기능은 *확실히* 점보 프레임을 처리할 수 없는 네트워크에서만 유용하며, 이 경우 카메라당 연결 시간을 약 1초 정도 단축해 줍니다. 이는 단순한 외관상의 차이가 아닌 실질적인 타협이므로, 이제 이 기능을 사용할 때 SDK에서 다음과 같이 안내합니다:
>
> ```
> [Network] ⚠️ GVSP probe disabled (CHLOROS_GVSP_PROBE_FALLBACK=0) — staying at
> 1500 bytes, jumbo NOT tested. … if this network does carry it, you are giving
> up ~1.45x wire ceiling. Unset the variable to test for jumbo.
> ```
>
> **특별한 이유가 없는 한 이 설정을 변경하지 마십시오.**이 기능을 활성화한 상태로 두면, 각 카메라는 백엔드 세션당 한 번 측정되며, 링크가 변경될 때(다른 어댑터, 다른 링크 속도, 또는 스위치 교체 시처럼 링크가 끊어진 경우) 다시 측정됩니다. 재연결 시에는 측정 결과를 재사용합니다.**스위치 뒤에서 변경이 발생했으나 스위치가 계속 작동한 경우, 백엔드를 재시작하여 다시 측정하십시오.** 이 검사에서 첫 연결 시 가끔 몇 분이 소요될 수 있으며, 그럴 경우 로그에 해당 내용이 기록됩니다.
>
> 점보 처리량을 *원하는* 경우, 점보를 종단 간으로 활성화하십시오 (NIC MTU 9000 + 이를 통과시키는 스위치)를 활성화하거나, 링크가 이를 지원한다는 것을 알고 있다면 `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000`로 고정하십시오. 단, 영구적으로 설정하는 것보다 명령어별 `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000 python …`를 사용하는 것이 더 바람직합니다. 고정된 크기는 프로브를 건너뛰고 앞쪽 네트워크에 대한 적응을 중단하기 때문입니다. 경로상의 **모든** 장치가 점보 패킷을 통과시켜야 합니다 — PoE 스플리터나 인젝터도 포함되며, 이는 점보 기능을 지원하는 설정임에도 불구하고 점보 패킷을 전송하지 못하는 일반적인 원인입니다.

> **`SC_ERR_TIMEOUT -1011`가 `capture()` / `grab*()` 중에 발생하는 것은 별개의 문제입니다 — 해당 오류는 실제 오류입니다.**
> 위의 참고 사항은**connect-time probe**에 의해 기록된 `-1011`에 대해서만 해당됩니다.**캡처** 과정에서 발생한 동일한 오류는 카메라가 정상적으로 연결되었으나 이미지를 전송하지 않고 있음을 의미합니다:
>
> ```
> File ".../lattice_sdk/camera.py", line ..., in grab_frame_with_metadata
>   buffer = self._get_buffer(timeout)
> lattice_sdk.exceptions.CaptureError: Capture failed: ... SC_ERR_TIMEOUT -1011
> ```
>
> 결정적인 단서는 *제어* 채널 상태가 정상인 카메라(탐지 작동, 설정 및 `[chunk-enable …]` 쓰기 모두 성공)임에도 불구하고 *모든* 프레임에서 타임아웃이 발생하는 경우입니다.
>
> **일반적인 원인은 카메라가 하드웨어 트리거 모드로 설정되어 있기 때문입니다.** `trigger_mode="On"` 및 `trigger_source="Line2"` 오류가 발생하면, M8 동기화 케이블에 전기적 에지가 도달할 때까지 카메라는 아무것도 전송하지 않습니다. 해당 라인을 구동하는 케이블이 없다면, 모든 데이터 캡처가 무한정 대기하게 됩니다. 카메라는 고장 나지 않았고 네트워크도 정상이며 — 지시받은 대로 정확히 작동하고 있을 뿐입니다.
>
> `CameraSettings()` 및 `default` / `high_speed` / `high_quality` 프리셋은 프리런(free-run) 모드를 설정하며, 준비 상태에서 타임아웃된 캡처는 단순한 `-1011` 오류 메시지 대신 구체적인 원인을 표시합니다. `PRESETS["triggered"]`는 설계상 Line2를 준비 상태로 만듭니다.
>
> 카메라를 강제 프리런 모드로 전환하려면:
>
> ```python
> import copy
> settings = copy.deepcopy(PRESETS["high_quality"])   # PRESETS entries are shared
> settings.trigger_mode = "Off"        # free-run; don't wait for an M8 edge
> ```
>
> `trigger_mode="Off"`를 사용해도 여전히 타임아웃이 발생한다면, 해당 카메라는 실제로 데이터를 전송하지 못하고 있는 것입니다. 로그 파일과 `ip link show`를 저희에게 보내주십시오.

#### 색상 프로필 (RGB 실시간 미리보기) — `set_color_profile`

`LatticeCamera.set_color_profile(profile, custom_cct_k=None)`는 RGB 카메라의 **실시간 미리보기**에 사용할 디스플레이 색상 프로필을 선택합니다(다중 스펙트럼 카메라는 이 설정을 무시합니다):

| 프로파일 | 의미 |
| --- | --- |
| `raw` | 방사계 체인을 완전히 우회합니다. |
| `linear` | DSNU + 플랫 + WB, CCM 및 감마 적용 없음. |
| `natural` | 선형 + 측정된 CCM + sRGB 감마, 저렴한 마무리 처리만 적용 (채도 평활화 + 하이라이트 채도 감소) — 사실적인 기본 설정. |
| `enhanced` | `natural`에 풀 허브 패리티 마무리(프린지 제거, 비브런스, CLAHE 국소 대비) 추가. 프레임당 마무리 비용이 대략 **두 배**, 따라서 라이브 프레임 속도가 낮아집니다. |
| `custom_temp` | `natural`이지만, 화이트 밸런스가 `custom_cct_k` 켈빈 (DLS 무시; 백엔드 측에서 2000–10000 K로 제한됨). |

이 프로필은 **실시간 미리보기 전용** 속도/화질 조절 기능입니다: 저장된 캡처는 선택한 프로필과 관계없이 항상 완전하고 풍부한 마무리를 적용받으므로, 프레임 시간을 확보하기 위해 `natural`를 선택하더라도 디스크에 저장되는 영상의 품질은 저하되지 않습니다. 알 수 없는 프로필은 `ValueError`를 증가시킵니다. 클로로스(chloros) 백엔드에 연결 가능한 경우, 변경 사항이 해당 백엔드로 POST되므로 다음 미리보기 프레임에 반영됩니다(백엔드가 없는 직접 SDK 사용자도 설정 변경 사항을 적용받습니다).

```python
with LatticeCamera(serial="214701292") as cam:   # RGB cam
    cam.set_color_profile("enhanced")            # richer look, lower LIVE fps
    cam.set_color_profile("custom_temp", custom_cct_k=5600)
```

#### 하이라이트 (RGB 디스플레이) — `set_highlights`

`LatticeCamera.set_highlights(value)`는 **디스플레이** RGB 카메라에서 렌더링된 가장 밝고 하얀 픽셀을 압축하여 렌더링합니다. 이는 밝은 흰색 표면이 평평한 흰색 덩어리로 렌더링되는 문제를 해결합니다. `value`는 `-100`..`0`(클램핑됨); `0`는 비활성화 상태이며, 이 경우 렌더링 결과는 해당 설정이 적용되지 않았을 때와 정확히 동일합니다. 설정을 더 강하게 적용하면 디스플레이의 흰색 수준이 낮아지고(`-100`에서 약 60% 수준) 톤 커브가 넓어지므로, 따라서 미리보기의 자동 밝기 조절로 순백색으로 변했던 표면의 디테일이 다시 살아납니다.

이는 디스플레이 제어 기능이지만, 채도나 대비(라이브 미리보기만 변경함)와 달리 저장되는 결과물도 변경합니다. 라이브 미리보기와 **디스플레이 수준** 캡처가 이 설정을 따르기 때문에 해당 파일들도 더 어둡게 나옵니다(이 파일들은 `Chloros:DisplayHighlights` 값을 반영함); 방사도 및 반사도 내보내기 파일은 절대 영향을 받지 않으며, 클리핑된 센서 데이터는 복구할 수 없습니다 — 이 경우 노출을 낮추십시오. 멀티스펙 및 모노 카메라에서는 이 설정을 무시합니다. 이 값은 `settings.highlights`에 저장되며, 연결 가능한 클로로스 백엔드로 POST되며, 이는 `set_color_profile`와 정확히 동일하며, CLI에서 이에 해당하는 명령은 `lattice highlights --set N`입니다.

```python
with LatticeCamera(serial="214701292") as cam:   # RGB cam
    cam.set_highlights(-60)                      # 0 = off, -100 = strongest
```

#### 모노(M3M) 카메라와 `Calibration`

모노 **M3M** 카메라(`M3M-<lens>-F<wavelength>`)는 단일 대역입니다: 단일 그레이스케일 평면이며, 베이어 모자이크나 3×3 스펙트럼 크로스톡 행렬이 없습니다. `Calibration`는 이를 인식하고 `is_mono` 플래그를 노출합니다. 반사율은 여전히 대역별 방사계 지도로서 적용되지만(분리 연산은 단위 행렬입니다), 단일 카메라에 대한 다중 대역 연산은 무의미한 결과를 반환하기보다는 오히려 유의미한 결과를 산출합니다:

```python
from chloros_sdk import Calibration, CalibrationError

calib = Calibration("M3M-L87-F685")
print(calib.is_mono)        # True  (False for any M3C / RGN Bayer cam)
print(calib.filter_type)    # 'mono'  (sentinel; not a real crosstalk key)

# NDVI needs two bands (Red + NIR); one mono band can't supply both.
try:
    calib.compute_ndvi(reflectance_frame)
except CalibrationError as e:
    print(e)   # "...single-band mono (M3M) camera. Combine multiple..."
```

모노크롬 하드웨어로 식생 지수를 구축하려면, 서로 다른 파장의 여러 M3M 카메라를 정렬된 다중 대역 스택으로 결합하고( [어레이 정렬](#array-alignment) 참조), 단일 카메라가 아닌 해당 스택 전체에 대해 지수를 계산하십시오.

#### DAQ-M 밴드 광도 — `DLS.band_magnitudes`

`DLS(mac_address, filter_model="RGN-IMX265")`는 자체 BLE MAC을 통해 블루투스로 DAQ-M에 연결됩니다. 이 클래스는 백엔드를 거치지 않고 센서와 직접 통신하는 유일한 광센서 클래스이므로, 직접 카메라 클래스에 필요한 요소(`CAMERA_AVAILABLE`: Arena SDK 런타임 및 OpenCV)와 더불어 블루투스용 `bleak`가 필요합니다 (`pip install "chloros-sdk[camera]"` 또는 `pip install bleak`; 이 구성 요소가 없으면 `DLS`에서 `DLSError` 오류가 발생합니다). 다른 곳에 연결된 DAQ-M (백엔드의 풀, 프로젝트, GUI)는 광고를 중지하므로, 먼저 해당 위치에서 릴리스해야 합니다. **`mac_address`가 필수입니다**: 각 장치마다 고유한 값을 가지므로 기본값이 없으며, 이 값이 없는 `DLS()`는 연결을 시도하기 전에 `DLSError`를 발생시킵니다. XPR을 사용하여 MAC 주소를 찾으십시오OTX003351 ([Discovery](#discovery--finding-an-address-to-connect-with) 참조) 또는 `chloros-cli daq pool-discover --only ble`를 사용하여 MAC 주소를 찾으십시오.

`DLS.band_magnitudes(spectrum=None, *, spectral_response=None)`는 DAQ-M 스펙트럼을 통합합니다(`spectrum`가 `None`일 때의 최신 측정값; 아직 없는 경우 `DLSError`)를 카메라의 세 채널 응답을 통해 통합하고, `{"red": …, "green": …, "nir": …}` — 키 이름은 해당 순서대로 채널을 나타냅니다. RGB 카메라의 곡선에서 세 번째(`nir`)는 파란색 채널입니다.

- `spectral_response`가 없으면 필터 모델의 플릿 커브(`filter_model`가 `DLS`로 구축된)를 사용하며, 이는 단일 장치에서 측정된 것입니다.
- `spectral_response`는 대신 카메라의 **자체** 측정 곡선을 사용합니다: 해당 카메라의 보정 팩(`color_correction.spectral_response_smoothed` — RGB 카메라; 다중 스펙트럼 팩에는 이러한 곡선이 포함되지 않음)에 저장된 dict `{"wavelengths": [...], "R": [...], "G": [...], "B": [...]}`입니다. 사용할 수 없는 dict인 경우 플릿 곡선으로 대체됩니다.

```python
from chloros_sdk import DLS

# camera_curves: {"wavelengths": [...], "R": [...], "G": [...], "B": [...]}
with DLS(mac_address="AA:BB:CC:DD:EE:FF") as dls:
    reading = dls.acquire()
    fleet = dls.band_magnitudes(reading)                         # fleet curve
    own = dls.band_magnitudes(reading,
                              spectral_response=camera_curves)   # the camera's own
```

CLI의 `chloros-cli lattice dls --camera SERIAL`는 백엔드를 통해 동일한 작업을 수행하며, 로드되거나 캐시된 보정 팩에서 카메라의 곡선을 찾아내고, 어떤 곡선을 사용했는지 알려줍니다. HTTP를 통해서는 `POST /api/dls/cli-read`와 `mac` (필수: 이 정보가 없으면 응답은 HTTP 400 오류가 발생하며, `code` `MAC_REQUIRED`) 본문에는 `camera_serial`가 포함됩니다; 해당 카메라의 자체 곡선이 사용된 경우 응답의 `band_magnitudes_source`는 `camera <serial>`가 되며, 플릿 곡선(곡선이 없는 카메라: RGB가 아니거나, 팩이 로드되지 않았거나 캐시되지 않은 경우).

---

## 예외

"Chloros에서 발생한 모든 오류"를 처리하기 위해 기본 클래스를 포착합니다:

```python
import chloros_sdk

try:
    chloros_sdk.process_folder("/path/to/folder")
except chloros_sdk.ChlorosAuthenticationError:
    print("Run `chloros-cli login` first.")
except chloros_sdk.ChlorosLicenseError:
    print("Chloros+ subscription required.")
except chloros_sdk.ChlorosError as e:
    print(f"Chloros error: {e}")
```

> `ChlorosAuthenticationError` 및 `ChlorosConfigurationError`는 `chloros_sdk` (및 `chloros_sdk.exceptions`)의 속성이지만 `__all__`에는 없으므로, `from chloros_sdk import *`는 이를 포함하지 않습니다: 이름으로 직접 임포트하십시오. `ChlorosError` 계열은 `ChlorosLocal`와 이를 기반으로 구축된 함수들(`process_folder()`, `process_lattice_capture()`)에 의해 상속되며; 스마트 커넥트 표면은 `ChlorosConnectError`를 상속받습니다.

계층 구조:

```

ChlorosError
├── ChlorosBackendError           (backend binary not found / failed to start)
├── ChlorosConnectionError        (HTTP transport failure; no backend running
│                                  with auto_start_backend=False)
├── ChlorosLicenseError           (403 PLAN_UPGRADE_REQUIRED: plan below Copper)
├── ChlorosAuthenticationError    (401 AUTH_REQUIRED: not logged in)
├── ChlorosConfigurationError     (bad configure() inputs)
└── ChlorosProcessingError        (pipeline failed; create/import refused)
    └── ChlorosProjectFilesMissingError  (open_project(): none of the
                                  project's source files found, even after
                                  the relink; .missing_roots, .audit)

ChlorosConnectError                (the smart-connect surface: connect_*, attach_array,
                                    list_*, discover_*, every CameraSession /
                                    ArraySession / DAQSensorSession method,
                                    RecorderHandle, ContinuousHandle,
                                    disconnect_all(), and analyze_array_network()
                                    when no backend answers; 401 and 403 included.
                                    Derives from plain Exception, NOT from
                                    ChlorosError, so `except ChlorosError` will
                                    not catch it. .warning: a warning the
                                    backend sent beside its error (also at the
                                    end of the message); .recording_closed: the
                                    recording a failed DAQ-E restart closed;
                                    both None otherwise)

FrameFetchError (RuntimeError)     (grab() / frame_stream() refusals; .status,
                                    .body, .errors)
ValueError                         (capture() arguments: PNG/JPEG with a level they
                                    cannot hold; a bad packed=)
FileNotFoundError / ImportError    (open_project(): no such folder / no project
                                    loaders)
RuntimeError                       (CameraHandle / ArrayHandle: a backend refusal)

lattice_sdk exceptions:
LatticeError
├── CameraNotFoundError
├── CameraConnectionError
├── StreamError
├── CaptureError
├── CalibrationError
├── NetworkError
└── DLSError
```

---

## 종단 간 예제

### 1. 사용자 정의 진행률 표시줄을 사용하여 폴더 처리하기

```python
from chloros_sdk import ChlorosLocal

def progress(percent, message):
    bar = "#" * (percent // 5)
    print(f"\r[{bar:<20s}] {percent:3d}% {message}", end="", flush=True)

with ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26")
    cl.import_images("C:/DroneImages/Flight001", recursive=True)
    cl.configure(
        debayer="High Quality (Faster)",
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "GNDVI", "OSAVI", "SAVI"],
        export_format="TIFF (16-bit)",
    )
    cl.process(progress_callback=progress)
print()
```

### 2. 실시간 LATTICE 어레이 → 반사율 + DAQ 참조

```python
import chloros_sdk

# Open a paired sensor first so the array's reflectance step has an
# absolute reference. Smart-detect picks USB / BLE / ETH automatically.
with chloros_sdk.connect_daq_sensor() as daq:
    with chloros_sdk.connect_array([
            "213800234", "214000533", "214701288", "214701292"
    ]) as arr:
        # Smart capture: wait for AE to settle, then snap
        arr.capture("./out", processing="reflectance", smart=True)

        # Record the corresponding DAQ frames as ground truth
        daq.record_start(output_dir="./out", device_name="sky-reference")
        # ... do whatever capture campaign ...
        info = daq.record_stop()
        print(info["path"], info["rows"])
```

### 3. 프로젝트 기반 캡처 캠페인

```python
import time, chloros_sdk

with chloros_sdk.open_project("/home/user/Chloros Projects/Field_A") as proj:
    report = proj.connect_all(verbose=True, align=True)
    if report["arrays"]["errors"]:
        raise SystemExit(f"Array(s) failed to connect: {report['arrays']['errors']}")

    rig = proj.arrays["main_rig"]

    # Re-align right before the campaign
    rig.calibrate_alignment(num_frames=5)
    rig.export_alignment("./alignments/main_rig.json")

    # 50 sequential single-frame captures at 2 fps
    for i in range(50):
        frames = rig.capture(
            output_dir=f"./out/frame_{i:04d}",
            processing="reflectance",
            apply_calibration=True,
            apply_white_balance=True,
        )
        time.sleep(0.5)

    # End-of-day: process the captured folder. process() accepts only
    # mode/wait/progress_callback/poll_interval — indices come from the
    # project's saved config (or set them via ChlorosLocal.configure()).
    proj.process()
```

### 4. 멀티 카메라 프레임 스트림 → NumPy 파이프라인

```python
import chloros_sdk
import numpy as np

with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    rig = proj.arrays["main_rig"]

    for frames in rig.frame_stream(
            processing="radiance",
            fps=5.0, count=300,
            apply_calibration=True,
            apply_white_balance=True):
        # frames is {serial: numpy_array}; cams not delivering this tick are omitted
        for serial, frame in frames.items():
            print(serial, frame.shape, frame.dtype, frame.mean())
```

### 5. 헤드리스 직접 하드웨어(백엔드 없음) 캡처 스크립트

```python
from chloros_sdk import LatticeCamera, PRESETS, discover_cameras

cams = discover_cameras(timeout_ms=3000)
print(f"Found {len(cams)} cams")

settings = PRESETS["high_quality"]
for c in cams:
    with LatticeCamera(serial=c.serial, settings=settings) as cam:
        result = cam.capture(output_dir="./out", format="tiff")
        print(c.serial, result.filepath)
```

### 6. 4-Cam 어레이 연결 전 기능 확인

```python
import chloros_sdk

serials = ["214701288", "213800234", "214000533", "214701162"]

probe = chloros_sdk.analyze_array_network(
    master_serial=serials[0],
    slave_serials=serials[1:],
    width=2048, height=1536,
    pixel_format="BayerRG12",
)

if probe["status"] == "ok":
    arr = chloros_sdk.connect_array(
        serials, width=2048, height=1536, pixel_format="BayerRG12")
elif probe["status"] == "auto_capped_fps":
    r = probe["recommended"]
    print(f"Keeping resolution; capping trigger rate at "
          f"{r['recommended_target_fps']} fps")
    arr = chloros_sdk.connect_array(
        serials, width=2048, height=1536, pixel_format="BayerRG12",
        target_fps=r["recommended_target_fps"])
elif probe["status"] == "auto_shrunk":
    r = probe["recommended"]
    print(f"Auto-shrinking to {r['out_width']}x{r['out_height']} "
          f"binning={r['binning']} for sim-sync")
    arr = chloros_sdk.connect_array(
        serials,
        width=r["width"], height=r["height"],     # before binning
        pixel_format=r["pixel_format"], binning=r["binning"])
elif probe["status"] == "needs_force_slip":
    print("Wire can't sustain sim-sync; falling back to slip mode")
    arr = chloros_sdk.connect_array(
        serials, force_tier="slip-emit-and-capture")
else:
    raise RuntimeError(f"Probe error: {probe.get('error')}")
```

### 7. 캡처 레시피와 동등한 구현 (순수 Python)

`run_project_recipe()`는 CLI 레시피를 있는 그대로 실행합니다([프로젝트 폴더 및 레시피](#project-folder--recipes) 참조). 이를 Python로 작성했을 때의 동일한 단계는 다음과 같습니다:

```python
import time, chloros_sdk

with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    cam = proj.cameras["FrontLeft"]
    rig = proj.arrays["main_rig"]
    sky = proj.sensors["Sky"]

    # apply: connect_all() applied each camera's saved settings. CameraHandle
    # has no settings method; a CameraSession on the same pooled camera does
    # (it is borrowed, so it never releases the camera)
    chloros_sdk.connect_camera(cam.serial).set_settings(
        exposure_time=5000, gain=6.0)

    # wait
    time.sleep(2)

    # capture
    cam.capture("pose_a/", format="tiff", processing="radiance")

    # stream
    rig.stream(count=60, fps=5, output_dir="stream/", processing="raw")

    # sensor read
    print(sky.read())
```

---

## MCP 서버 (AI 어시스턴트)

`chloros-mcp`를 사용하면 Claude Desktop, Claude Code 또는 Cursor와 같이 [모델 컨텍스트 프로토콜](https://modelcontextprotocol.io)(MCP)을 지원하는 AI 어시스턴트가 Chloros를 제어할 수 있습니다. 즉, 이미지 처리(프로젝트 생성, 캡처 파일 폴더 가져오기, 설정 선택, 처리, 실행 결과 내보내기)를 수행하고, LATTICE 카메라, 어레이 및 DAQ 광 센서를 제어(연결, 미리보기, 노출 및 색상 설정, 캡처, 기록, 펌웨어 업데이트)할 수 있게 해줍니다. 이 서버의 도구는 이 SDK가 수행하는 기능을 모두 포함합니다. 사용자가 자연어로 요청하면 어시스턴트가 서버의 도구를 호출합니다:

```text
You:  Process the flight in D:\Flights\2026-10-17 to reflectance and NDVI.

The assistant calls:
  create_project          {"project_name": "Flight_2026-10-17"}
  import_images           {"folder_path": "D:\\Flights\\2026-10-17"}
  configure_processing    {"indices": ["NDVI"]}
  start_processing        {}
  get_processing_status   {}      (every 30-60 s until its state is "finished")

You:  Show me what the array sees, then capture reflectance every 10 seconds
      for 5 minutes.

The assistant calls:
  list_devices            {}
  get_camera_preview      {"array_id": "array-1779862544497"}
  start_capture_run       {"array_id": "array-1779862544497", "mode": "interval",
                           "interval_s": 10, "duration_s": 300,
                           "processing": "reflectance"}
  get_job_status          {"job_id": "..."}   (until its state is "finished")
```

서버는 이 SDK를 기반으로 구축되어 있습니다. 처리 도구는 [`ChlorosLocal`](#image-processing--chloroslocal)을 호출하고, 하드웨어 도구는 [스마트 커넥트 호출](#smart-connect-for-lattice-cameras), [DAQ 세션](#daq-sensor-sessions), `ChlorosProject` 및 [카메라 유틸리티](#camera-utilities)를 호출하며, 이 모든 것은 Chloros 앱이 사용하는 동일한 백엔드와 통신합니다. 요청에는 `X-Chloros-Client: sdk`가 포함되므로, 따라서 백엔드는 SDK의 로그인 및 요금제 확인을 적용하며, SDK의 규칙도 함께 적용됩니다(예를 들어, LATTICE 캡처의 경우 Texture Aware는 거부됩니다).

### 설치

서버는 `chloros-sdk` 패키지에 포함되어 제공되며, 별도로 설치할 사항은 없습니다. SDK는 `chloros-mcp` 명령어를 설치하며, 이는 `python -m chloros_sdk.mcp_server`와 동일합니다:

```bash
pip install chloros-sdk        # not needed where the Chloros installer set up the SDK (Step 2)
chloros-mcp --help
```

- **Windows:** Chloros 설치 프로그램이 SDK에 SDK를 pip를 통해 설치하며, 이때 `chloros-mcp.exe`가 해당 Python의 `Scripts` 폴더에 배치됩니다.
- **Linux(.deb):** 이 패키지는 시스템 `python3`용 SDK를 설치하고, `chloros-mcp` 명령어를 `/usr/bin`에 추가하므로, 모든 사용자의 `PATH`에 추가됩니다. 가상 환경(virtualenv)에서는 `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl`가 가상 환경의 `bin`에 `chloros-mcp`를 추가합니다.

서버는 SDK(Python 3.7 이상) 위에 표준 라이브러리만 사용합니다. MCP 클라이언트가 이를 시작하고 stdin/stdout을 통해 통신합니다. `--help`를 사용하는 경우를 제외하고는 직접 실행할 필요가 없습니다.

### 클라이언트 설정

**Claude Desktop:**서버를 `claude_desktop_config.json`에 추가한 후(Windows에서는 `%APPDATA%\Claude\claude_desktop_config.json`;**설정 > 개발자 > 구성 편집**을 통해 열 수 있음), Claude Desktop을 다시 시작합니다:

```json
{
  "mcpServers": {
    "chloros": {"command": "chloros-mcp"}
  }
}
```

**Claude 코드:**

```bash
claude mcp add chloros -- chloros-mcp
```

**커서:** Claude Desktop과 동일한 `mcpServers` 항목을 `~/.cursor/mcp.json`(모든 프로젝트; Windows에서는 `%USERPROFILE%\.cursor\mcp.json`) 또는 `.cursor/mcp.json`에 있습니다.

옵션은 `args`에 입력하거나, Claude Code(`claude mcp add chloros -- chloros-mcp --no-auto-start`) 명령어 뒤에 입력합니다. `chloros-mcp`에 포함된 `PATH` 대신 특정 Python의 SDK를 사용하려면, 해당 Python으로 모듈을 실행하십시오(해당 Python이 `PATH`에서 첫 번째가 아닌 경우, 전체 경로를 `command`로 지정하십시오):

```json
{
  "mcpServers": {
    "chloros": {"command": "python3", "args": ["-m", "chloros_sdk.mcp_server"]}
  }
}
```

클라이언트는 셸 없이 서버를 시작하므로, 서버 시작에 실패했다는 오류가 보고된다면, 클라이언트가 인식하는 `PATH`에 `chloros-mcp`가 설치되어 있지 않은 것일 가능성이 높습니다. 대신 전체 경로를 지정하십시오(Windows의 경우 `where chloros-mcp`, Linux의 경우 `which chloros-mcp`, 여기서 .deb 파일의 위치는 `/usr/bin/chloros-mcp`입니다). 또는 SDK가 포함된 Python으로 모듈을 실행하십시오.

### 옵션

| 옵션 | 기본값 | 기능 |
| --- | --- | --- |
| `--backend-url URL` | `http://127.0.0.1:5000` | 사용할 Chloros 백엔드. |
| `--no-auto-start` | off | 백엔드를 절대 시작하지 않음: Chloros 앱, Linux 백엔드 서비스, 또는 다른 머신의 (참조: [원격 백엔드 모드](#remote-backend-mode-pip-only-host-via-tunnel); 이 경우 폴더 경로는 두 시스템 모두에서 동일한 폴더를 가리켜야 함). |
| `--backend-exe PATH` | Chloros가 설치하는 위치 | SDK가 검색하지 않는 위치에 Chloros가 설치된 경우, 백엔드가 실행 중이지 않을 때 시작할 백엔드. |
| `--log-level LEVEL` | `WARNING` | `DEBUG`, `INFO`, `WARNING` 또는 `ERROR`. 로그는 stderr로 출력되며, 대부분의 클라이언트는 이를 MCP 로그에 표시합니다. stdout에는 프로토콜 메시지만 전송됩니다. |

### 도구

각 도구의 인수는 해당 도구가 래핑하는 SDK 호출의 인수와 동일하며, 클라이언트는 각 인수를 전체 스키마와 함께 나열합니다. 경로는 절대 경로입니다.

**처리**

| 도구 | SDK 호출 | 기능 |
| --- | --- | --- |
| `create_project` | `ChlorosLocal.create_project` | 프로젝트를 생성하고 이를 열려 있는 프로젝트로 설정합니다. 처리 도구는 이 프로젝트를 대상으로 작동합니다. 일반 이름을 지정하면 기본 프로젝트 폴더에 폴더가 생성되며, 절대 경로를 지정하면 해당 위치에 폴더가 생성됩니다. |
| `open_project` | `ChlorosLocal.open_project` | 저장된 프로젝트를 열고 이를 열려 있는 프로젝트로 설정합니다. 이동된 프로젝트의 경로는 현재 위치한 폴더를 가리키도록 재설정되며, `relink_folder`(절대 경로)는 누락된 다른 파일들이 현재 위치한 폴더의 이름을 지정합니다. `source_files` 및 `image_count`를 반환합니다. 찾을 수 없음: 해당 파일들의 위치를 나열하는 데 오류가 발생했으며 `relink_folder`로 명명됨; 열려 있는 프로젝트가 없습니다. |
| `import_images` | `import_images` | Survey3 `.RAW`+`.JPG` 쌍을 가져오고, ‘LATTICE’ `.tif` 캡처 및 DAQ `.daq` 녹화 데이터를 가져옵니다. `count`(이미지) 및 `scan_count`(녹화 데이터)를 반환합니다. |
| `configure_processing` | `configure` | 전달된 설정만 변경하고 그 외 설정은 변경하지 않습니다(값: [지원되는 값](#supported-values)); `custom_settings`는 주어진 전체 구성을 전송합니다. |
| `get_project_settings` | `get_config` | 열려 있는 프로젝트의 저장된 처리, 내보내기 및 인덱스 설정을 읽습니다. |
| `start_processing` | `process(wait=False)` | 열려 있는 프로젝트의 처리를 시작하고 즉시 반환합니다. |
| `get_processing_status` | `get_status` | `state` (`idle`, `starting`, `processing` 또는 `finished`) 및 진행 상황을 보고합니다; 완료되면, 해당 실행의 `summary`, 내보내기 파일을 확인할 수 없었을 때의 `export_check_failure`, 이미지가 손실되었을 때의 `image_failure`를 기록하고, 나머지 결과물은 정상적으로 기록됩니다; 해당 실행을 실패로 보고하십시오. |
| `process_folder` | `process_folder` / `process_lattice_capture` | 폴더에 대한 프로젝트를 생성하고, 이를 가져온 후, 한 번의 호출로 구성을 완료하고 실행을 시작합니다 (LATTICE 캡처의 경우 `lattice: true`). 실행이 진행 중인 동안에는 아무것도 생성하기 전에 요청을 거부하며, 이때 백엔드에서도 어떤 도구에서든 프로젝트 변경을 거부합니다. |
| `export_light_sensor` | `export_light_sensor` | 프로젝트의 광센서 기록에 대해 보정된 `.daq` 및 `.csv` 파일을 작성합니다 기록을 위해 보정된 `.daq` 및 `.csv` 파일을 작성하며, 이미지가 처리되지 않은 상태에서 캐시에 저장되어 있지 않은 경우 MAPIR Cloud에서 공장 보정 데이터를 가져옵니다([광센서 기록](#light-sensor-recordings--calibrated-daq--csv) 참조). |
| `read_image_audit_tags` | `read_image_audit_tags` | 처리된 이미지의 Chloros 감사 메타데이터(출처, 보정, 적용된 단계)를 읽습니다. |

**카메라 및 어레이**

| 도구 | SDK 호출 | 기능 |
| --- | --- | --- |
| `list_devices` | `list_cameras`, `list_arrays`, `list_daq_sensors` | 현재 어떤 클라이언트에 무엇이 연결되어 있는지(각 클라이언트마다 `connected_here` 포함) 및 서버에서 실행 중인 작업. |
| `discover_devices` | `discover_lattice_cameras`, `discover_daq_sensors` | 연결될 수 있는 카메라 및 광 센서. |
| `get_device_status` | `CameraSession.status`, `ArraySession.status` / `capability`, `DAQSensorSession.status` | 하나의 카메라, 어레이(해당 기능 및 상태 포함) 또는 광 센서의 실시간 상태. |
| `analyze_array_network` | `analyze_array_network` | 제안된 어레이에 대해 이 호스트와 네트워크가 처리할 수 있는 용량. |
| `connect_camera` / `connect_array` | `connect_camera` / `connect_array` | 카메라 또는 동기화된 어레이(마스터 우선) 연결. |
| `connect_project_devices` | `open_project(...).connect_all` | 저장된 프로젝트에 기록된 모든 장치를 해당 설정과 함께 연결합니다. |
| `disconnect_device` | `disconnect` / `force_disconnect` | 카메라, 어레이 또는 광 센서 해제 (‘동작’ 참조). 어레이의 `keep_settings: true`는 각 카메라의 설정을 해당 카메라의 메모리에 저장하고, `clear_settings: true`는 해당 메모리를 초기화합니다 (그 결과는 `camera_memory`를 반환합니다). |
| `disconnect_project_devices`, `align_project_arrays`, `capture_project_devices` | `disconnect_all`, `align_arrays`, `capture_all` | 프로젝트의 모든 장치. |
| `read_camera_settings` / `set_camera_settings` | `CameraSession.read_nodes` / `set_settings` | 친숙한 이름이나 GenICam 노드 이름을 사용하여 카메라 설정을 읽거나 씁니다. |
| `set_array_frame_rate` | `ArraySession.set_target_fps` | 배열의 속도를 고정하거나 자동 속도로 되돌립니다. |
| `capture_camera` / `capture_array` | `CameraSession.capture` / `ArraySession.capture` | 단일 캡처 (`fastest` 및 `wait_for_exposure`는 `capture_fastest`이며, 노출이 확정된 캡처입니다). `output_dir`가 없는 경우, 열려 있는 프로젝트의`captures` 폴더에 저장되며, 파일은 프로젝트에 추가됩니다(처리 작업이 끝날 때까지 보류됨); 응답의 `project_add`에 그 내용이 명시되어 있습니다. |
| `start_capture_run` | `ArraySession.continuous`, `capture_repeated` | 작업 형태로 연속 또는 간격 촬영. |
| `start_array_recording` | `ArraySession.record` / `burst` | 결합 인덱스 비디오 또는 원시 버스트를 작업으로 처리합니다. |
| `build_burst_video` | `ArraySession.build_video` | 버스트를 보정된 비디오로 재처리하는 작업. |
| `get_job_status` / `stop_job` | | 작업을 추적하거나 중지합니다. |
| `get_camera_preview` | | 각 카메라의 실시간 미리보기를 이미지로 반환합니다. |
| `grab_frames` | `ArraySession.grab`, `CameraHandle.grab` | 저장하지 않고 처리 단계별 1프레임(또는 1개의 동기화된 세트): 플레인별 통계 및 각 플레인의 PNG 파일. |
| `calibrate_array_alignment`, `get_array_alignment`, `clear_array_alignment`, `nudge_array_alignment`, `export_array_alignment`, `import_array_alignment` | `ArrayHandle.calibrate_alignment`, `alignment_status`, `clear_alignment`, `tweak_alignment`, `export_alignment`, `import_alignment` | 배열의 정렬. |

**광 센서**

| 도구 | SDK 호출 | 기능 |
| --- | --- | --- |
| `connect_light_sensor` | `connect_daq_sensor` | DAQ-U, DAQ-M 또는 DAQ-E를 연결합니다(`cap_id`는 캡 보정을 설정합니다). |
| `get_light_sensor_reading` | `DAQSensorSession.latest` | 최신 측정값: 타임스탬프, 포화도, 스펙트럼 범위 및 피크, 그리고 파장 구간별로 평균화된 스펙트럼(각 점에 대한 값은 `full_spectrum`). |
| `set_light_sensor_streaming` | `stream_start` / `stream_stop` | 스트림 시작 또는 중지. |
| `start_light_sensor_recording` / `stop_light_sensor_recording` | `record_start` / `record_stop` | `.daq` 파일을 작업으로 기록합니다. `duration_s`는 자동으로 중지됩니다. |
| `set_light_sensor_cap` | `DAQSensorSession.set_cap` | 캡 보정 (베어 센서의 경우 `none`). |
| `get_light_sensor_bindings` / `set_light_sensor_bindings` | `daq_bindings` / `set_daq_bindings` | 각 카메라와 어레이의 반사율을 어떤 센서가 제공하는지. |
| `get_light_sensor_band_presets` / `get_light_sensor_reflectance` | `daq_band_presets` / `light_sensor_reflectance` | 대역 세트 및 두 개의 스트리밍 센서의 주변광 + 물체 반사율 및 지수. |
| `get_light_sensor_device_info`, `set_light_sensor_network`, `set_light_sensor_auto_stream`, `enable_light_sensor_ptp`, `reboot_light_sensor`, `update_light_sensor_firmware` | `device_info`, `set_network`, `set_auto_stream`, `enable_ptp`, `reboot`, `update_firmware` | DAQ-E 본체에 저장된 자체 설정. |

**카메라 유틸리티** ([카메라 유틸리티](#camera-utilities))

| 도구 | SDK 호출 | 기능 |
| --- | --- | --- |
| `get_camera_color_settings` / `set_camera_color_settings` | `camera_color_profile`, `camera_color_adjust`, `camera_highlights` 및 해당 설정 함수 | RGB 카메라의 디스플레이 색상: 프로파일 및 색온도, 채도/명암비, 하이라이트. |
| `run_camera_white_balance` / `set_camera_white_balance` | `camera_white_balance_once` / `set_camera_white_balance` | 원샷 화이트 밸런스, 또는 자동 / 꺼짐 / 고정 비율. |
| `list_camera_presets` / `apply_camera_preset` | `camera_presets` / `apply_camera_preset` | 카메라 사전 설정. |
| `list_filter_models` | `camera_filter_models`, `camera_filter_model_info` | 필터 모델 또는 이름으로 하나 선택. |
| `get_camera_firmware_status` / `update_camera_firmware` | `camera_firmware_status` / `update_camera_firmware` | 펌웨어 버전 및 플래시(기본적으로 작업). |
| `get_camera_power_info` / `set_camera_power` | `camera_power_probe`, `sleep_camera`, `wake_camera` | 전원 및 열 상태; 절전 모드 또는 활성 모드. |
| `calibrate_camera_with_target` | `calibrate_camera_with_target` | 화면에 표시된 보정 대상의 계수. |
| `get_network_adapters` / `fix_network_settings` | `network_adapters` / `fix_network_settings` | 카메라 네트워크 어댑터, 추정값 및 수정 사항 (관리자 전용). |
| `apply_vignette_correction` | `apply_vignette_correction` | 이미지 폴더의 비네팅 보정. |
| `read_dls` | `read_dls` | DLS 판독 결과 1건. |
| `probe_system` / `get_system_info` | `probe_system` / `system_info` | 호스트의 하드웨어 및 런타임. |

**일시적 카메라, 정렬 프로파일 및 인덱스** ([일시적 배열, 정렬 프로파일 및 인덱스](#transient-arrays-alignment-profiles--indices)). 여기에서 카메라를 여는 모든 도구는 `keep_settings` / `clear_settings`(기본값 false)를 사용합니다: [카메라 메모리](#camera-memory--keep_settings--clear_settings)를 참조하십시오.

| 도구 | SDK 호출 | 기능 |
| --- | --- | --- |
| `start_camera_continuous` / `start_multi_camera_stream` | `start_camera_continuous` / `start_multi_camera_stream` | 하나의 카메라를 녹화하거나, 여러 대를 동기화하여 스트리밍하며, 이를 하나의 작업으로 처리합니다 (`duration_s`가 자동으로 중지함). |
| `capture_cameras_transient` | `capture_cameras_transient` | 어레이를 연결하지 않고 하나의 동기화된 캡처 수행. |
| `test_array_sync` / `detect_array_master` | `test_array_sync` / `detect_array_master` | 트리거 동기화 확인 및 동기화 라인에 연결된 카메라 목록. |
| `analyze_array_models` | `analyze_array_models` | 이 모델들로 구성된 어레이가 이 네트워크에서 얻을 수 있는 결과. |
| `calibrate_alignment_profile` | `calibrate_alignment_profile` | 정렬 프로필 파일을 계산하고 저장합니다(`options`에는 더 세부적인 계산 옵션이 포함되어 있습니다). |
| `capture_aligned_multiband` / `start_aligned_stream` | `capture_aligned_multiband` / `start_aligned_stream` | 정렬된 다중 대역 이미지 또는 작업으로 처리되는 일련의 이미지. |
| `read_alignment_profile` / `reorder_alignment_profile` | `alignment_profile_info` / `reorder_alignment_profile` | 프로파일 파일의 내용; 밴드 순서 및 활성화된 카메라. |
| `list_index_presets`, `compute_index`, `capture_live_index` | `index_catalog`, `compute_index`, `capture_live_index` | 인덱스 프리셋 및 그라데이션; 멀티밴드 TIFF에 대한 인덱스; 한 번의 단계로 캡처-투-인덱스. |

**시간 동기화, 프로젝트 폴더 및 레시피** ([PTP 그랜드마스터](#ptp-grandmaster--time_sync_status), [프로젝트 폴더 및 레시피](#project-folder--recipes))

| 도구 | SDK 호출 | 기능 |
| --- | --- | --- |
| `get_time_sync_status` | `time_sync_status`, `time_sync_peers`, `time_sync_cameras` | PTP 그랜드마스터의 상태, 피어 및 각 어레이 카메라의 PTP 상태. |
| `restart_time_sync` / `set_time_sync_priority` | `restart_time_sync` / `set_time_sync_priority` | 재시작하고 우선순위를 설정합니다. |
| `get_project_folder` / `set_project_folder` / `reset_project_folder` | `project_folder` / `set_project_folder` / `reset_project_folder` | 새 프로젝트가 생성되는 폴더. |
| `run_project_recipe` | `run_project_recipe` | 저장된 프로젝트의 레시피를 작업으로 실행하고 로그를 기록합니다. 먼저 확인: 오류가 있는 레시피는 거부되어 아무것도 실행되지 않으며, 연결되지 않은 어레이/센서에 대해서는 경고가 표시됩니다(이때는 연결하지 않음: 먼저 `connect_project_devices`를 실행). `dry_run`는 확인을 반환하고 아무것도 실행하지 않습니다. |

저장된 프로젝트를 열어 처리할 수 있는 도구는 없습니다. 각 처리 작업은 `create_project`(또는 `process_folder`)로 시작됩니다.

### 요구 사항

- **동일한 컴퓨터에 Chloros가 설치되어 있어야 합니다.** Chloros 백엔드가 처리를 수행합니다. 대신 서버는 터널을 통해 연결된 백엔드를 사용할 수도 있습니다(`--backend-url`와 `--no-auto-start`, [옵션](#options) 참조).
- **해당 컴퓨터에서 로그인되어 있어야 합니다**. Chloros 앱이나 `chloros-cli login`를 통해 로그인해야 합니다. 서버가 사용자를 대신해 로그인할 수는 없습니다. 로그아웃된 상태에서 도구를 호출하면 백엔드의 사유를 포함한 "Chloros에 로그인되어 있지 않습니다"라는 응답이 돌아오며, 어시스턴트가 이를 알려줍니다.
- **유료 Chloros+ 요금제(Copper 이상)**. 서버는 SDK를 통해 액세스되며, 무료 Iron 요금제에는 해당 기능이 없습니다. 무료 요금제에서 도구를 호출하면 백엔드의 사유를 포함하여 "이 Chloros 요금제에서는 사용할 수 없습니다"라는 응답이 반환됩니다.
- SDK가 포함된 Python 3.7 이상 버전과, 로컬(stdio) 서버를 시작하는 MCP 클라이언트.

### 동작 방식

- **백엔드.** 서버를 시작하거나 도구 목록을 표시해도 아무것도 실행되지 않으며, 첫 번째 도구 호출 시 연결이 이루어집니다. Chloros 앱(또는 Linux 백엔드 서비스)이 실행 중이면, 서버는 해당 백엔드를 사용하고 계속 실행 상태로 둡니다. 백엔드가 실행 중이지 않으면, 서버는 설치된 백엔드를 창 없이 시작합니다. 서버가 종료되면(클라이언트를 종료하거나 서버를 재시작할 때), 서버는 자신이 시작한 캡처 작업과 광센서 기록을 중지하고 연결된 장치를 해제합니다. 백엔드는 계속 실행되므로, 따라서 진행 중인 처리 작업은 끝까지 계속되며, 30분 동안 아무도 사용하지 않으면 백엔드가 자동으로 종료됩니다. `--no-auto-start`의 경우 서버는 백엔드를 절대 시작하지 않습니다. 백엔드가 실행 중이지 않은 상태에서 도구를 호출하면 Chloros 앱을 열라는 메시지가 표시되며, 앱을 실행한 후 다음 호출 시 연결이 이루어집니다.
- **열린 프로젝트는 하나뿐입니다.** 백엔드에는 열린 프로젝트가 하나 있으며, 이는 앱이 열려 있을 때 Chloros 앱과 공유되므로, 어시스턴트가 작동하는 동안 앱에서 다른 프로젝트를 작업하지 마십시오.
- **전달된 설정만 변경됩니다.** `configure_processing`는 전달되지 않은 모든 설정을 저장된 값 그대로 유지합니다(SDK의 `configure()`는 데베이어, 비네트 보정, 반사율 보정, PPK 및 내보내기 형식이 전달되지 않았을 때 이를 기본값으로 재설정하며, 서버는 프로젝트에서 해당 값을 가져와 채웁니다). 프로젝트의 저장된 설정을 읽을 수 없는 경우, 다섯 가지 설정 모두 전달되지 않는 한 아무것도 변경하지 않고 그 사실을 알립니다. `indices`는 프로젝트의 인덱스 목록을 대체합니다. 결과에는 `settings_sent`로 표시하며, `warnings` 아래에는 카메라의 밴드로 생성할 수 없는 인덱스 이름을 나열합니다.
- **처리는 '시작 후 폴링' 방식입니다.** `start_processing`는 실행이 시작되는 즉시 반환되며, 이후 처리는 몇 분에서 몇 시간까지 소요됩니다. 그 후 어시스턴트가 `get_processing_status`를 이어서 실행합니다. `state`는 `idle`(처리할 항목 없음), `starting`(백엔드가 아직 실행 결과를 보고하지 않은 상태, 최대 20초까지), `processing` 또는 `finished`이며, 여기에는 실행의 [요약](#post-run-summary--hints): 내보낸 데이터 및 예상보다 적은 양이 내보내진 경우의 힌트(이미지 없음, 보정 대상 미검출)가 포함됩니다. 산출물은 앱과 마찬가지로 프로젝트 폴더 아래에 저장됩니다. 실행이 진행 중인 동안에는 `start_processing`가 거부되며, 응답을 중단한 백엔드는 완료된 실행이 아닌 오류로 보고됩니다.
- **어시스턴트가 확인하는 내용.** SDK의 경고는 `warnings` 항목 하위의 도구 결과에 표시되며, 실패 사항(로그아웃, 계획, 잘못된 경로)은 어시스턴트가 읽고 사용자에게 전달할 수 있는 오류 텍스트로 나타납니다. 20개 항목을 초과하는 목록은 처음 20개 항목만 표시되며, 전체 길이는 `<key>_total`로 표시됩니다. 경로는 반드시 절대 경로여야 합니다.
- **장치는 공유됩니다.** 카메라, 어레이 및 광 센서는 백엔드에 상주하므로, 어시스턴트가 연결된 Chloros 앱 중 하나를 제어할 수 있으며, 변경된 내용은 앱에 반영됩니다. `disconnect_device`는 서버가 연결한 내용을 반환하며, 다른 클라이언트(앱, `chloros-cli`, 스크립트, 다른 `chloros-mcp`)에 연결된 기기는 SDK의 `force_disconnect`인 `force: true`를 통해서만 해제되며, 백엔드가 연결 주체를 파악할 경우 해당 주체의 이름을 반환합니다; 백엔드가 전혀 보유하지 않은 기기는 연결되지 않은 것으로 보고됩니다(`released: false`, `connected: false`, 해제할 항목 없음). 백엔드가 실행하는 녹화 또는 스트림은 `get_job_status` 뷰와 함께 `counts` — `frames_written`, `frames_dropped`, `frames_failed`, `drop_reason`(다중 카메라 스트림용 프레임 그룹) — 및 한 줄짜리 `summary`("220 프레임 기록, 55 프레임 손실, 0 실패: ...")를 표시하며, 실행 중일 때와 완료 시 최종 수치를 보여줍니다. 다른 클라이언트의 해당 항목 목록 (`other_clients_jobs`)에는 동일한 내용이 포함되어 있으며 (`list_cameras()`, `list_arrays()` 및 `list_daq_sensors()`에는 각 장치의`connected_by`: `client`, `label`, `name`, `at`). 각 `chloros-mcp`는 백엔드에 자신을 `chloros-mcp (pid N)`로 식별합니다(`chloros_sdk.connect.CLIENT_LABEL`는 `X-Chloros-Client-Label`로 전송되며, 모든 SDK 스크립트에서 이를 설정할 수 있습니다). 다른 클라이언트의 광센서 기록을 중지하는 데에도 `force`가 사용됩니다.
- **장시간 작업은 하나의 작업입니다.** `start_capture_run`, `start_array_recording`, `build_burst_video`, `start_light_sensor_recording`, `start_camera_continuous`, `start_multi_camera_stream`, `start_aligned_stream`, `run_project_recipe`, 그리고 `background`에 대한 캡처, 해결 또는 인덱스 요청이 있을 경우, 즉시 `job_id`를 반환합니다. 이어서 `get_job_status`가 실행되고, `stop_job`가 이를 종료합니다 이 과정을 마무리합니다. `max_captures`, `duration_s`(또는 버스트의 경우 `max_frames`)를 지정하지 않는 한, 실행은 중지될 때까지 계속됩니다; `min_free_gb`는 드라이브의 여유 공간이 부족할 경우 시작을 거부합니다. 직접 제한을 설정하지 않는 한 제한은 적용되지 않습니다. 백엔드가 실행하는 녹화 또는 스트림(`start_camera_continuous`, `start_multi_camera_stream`, `start_aligned_stream`)는 백엔드의 작업 ID를 유지하므로, 나중에 시작된 다른 `chloros-mcp`는 해당 ID를 통해 이를 찾아냅니다: `get_job_status`는 이를 이어받으며(`other_clients_jobs`, `GET /api/camera/cli-jobs`, 각각 `started_by`를 가진 작업들을 나열하고), 그리고 `stop_job`는 `force: true`를 통해서만 이를 중지시킵니다. 그 외의 모든 작업 (캡처 실행, 배열 기록 또는 버스트, 백그라운드 캡처)는 이를 시작한 `chloros-mcp`에서 실행되며, 오직 해당 서버만이 이를 인식합니다: 다른 서버가 해당 작업을 조회하면 그렇게 표시됩니다. 중지된 작업은 큐를 출력하는 동안 `stopping`를 읽고, 최종 카운트가 포함된 `finished`를 10분 동안, 그리고 어떤 `chloros-mcp`에서든: 시작한 서버는 다른 서버의 강제 중지 상황도 동일한 방식으로 감지합니다. `get_job_status`는 중지를 절대 기다리지 않습니다: 중지 중일 때(자신의 `stop_job`, 다른 클라이언트의중지, 또는 자체적으로 종료된 작업의 카메라를 해제하는 중지) 동안 즉시 `stopping`와 실시간 카운트 값으로 응답하며, 이미 중지 중인 작업의 `stop_job`는 이를 알립니다. 종료된 작업은 `stop_reason`에 종료 사유를 명시합니다: 자체 사유(`no_frames`, `disk_full`, `member_missing`, 또는 배열 연속 실행의 경우 `count`, `duration`, `array_gone`), `limit reached`는 카운트 값이 `count`(또는 버스트의 `max_frames`, 녹화의`duration_s`)에 도달했을 때만, `error`, `stopped` (`stopped_by` 포함) 클라이언트가 중지한 경우, 그 외에는 `finished`.
- **캡처 파일 저장 위치.** `output_dir`가 지정되지 않은 경우, 앱이 캡처 내용을 저장하는 열려 있는 프로젝트의 `captures` 폴더로 이동합니다.
- **이미지.** `get_camera_preview` 및 `grab_frames`는 JSON 데이터와 함께 어시스턴트가 확인할 수 있는 이미지를 반환합니다. `grab_frames` 이미지는 보기를 위해 명암비가 조정된 것이며, 해당 이미지의 통계값은 실제 값입니다.
- **서버가 종료되면**, 서버가 시작한 작업은 중지되고 서버가 시작한 광센서 기록은 종료됩니다. 서버가 백엔드를 시작한 경우, 백엔드를 중지하기 전에 연결했던 장치들도 해제합니다. 앱의 백엔드에서는 해당 장치들이 계속 연결된 상태로 유지됩니다.
- **프로토콜.** stdio 전용(HTTP 엔드포인트 없음), 도구 전용(리소스나 프롬프트 없음), 텍스트 및 이미지 결과, MCP 프로토콜 버전 2024-11-05부터 2025-11-25까지.**제외된 기능.** 로그인 또는 로그아웃(계정은 사용자의 것이며, 서버는 로그인할 수 없음), 그리고 서버가 시작하지 않은 백엔드 중지. 이 SDK가 처리 및 하드웨어와 관련하여 수행하는 그 외 모든 작업에는 전용 도구가 있습니다. (DAQ-E의 프로비저닝 — 이름, 인증 토큰, OTA 비밀번호, 서명 요구 사항, 공장 초기화) 및 직접 DAQ 서버는 `chloros-cli` 전용이며, SDK에도 포함되어 있습니다: 잘못된 이름이나 토큰을 입력하면 모든 Chloros 설치본이 장치에서 잠기게 됩니다.)

---

## 백엔드 자동 시작

스마트 커넥트 진입점 — `connect_camera`, `connect_array`, `connect_daq_sensor`, `discover_lattice_cameras` 및 `discover_daq_sensors` —는 백엔드가 `127.0.0.1:5000` (스마트 커넥트 인터페이스의 기본 URL)에서 수신 대기하고 있다고 가정합니다. GUI나 CLI가 이미 실행 중일 때는 백엔드가 하나 있습니다. 스크립트만 단독으로 실행할 경우, 해당 백엔드가 실행 중이지 않을 수 있으므로, 이 함수들은 첫 호출 전에 **번들로 제공된 백엔드 바이너리를 자동으로 시작**(`ChlorosLocal`와 마찬가지로 창 없이)한 다음, 백엔드가 준비될 때까지 최대 60초 동안 대기합니다. `list_cameras()`, `list_arrays()`, `list_daq_sensors()`, `attach_array()` 및 `analyze_array_network()`는 백엔드를 절대 시작하지 않습니다.

규칙:

- **로컬 URL만 생성됩니다.** `backend_url`가 `localhost` / `127.0.0.1` / `[::1]`를 가리키는 `backend_url`만 허용됩니다. 그 외의 호스트는 타인의 컴퓨터로 간주되어 절대 생성되지 않습니다.
- **응답하기 전에 종료되는 백엔드는 대기 상태를 종료합니다.** 이 경우 호출은 60초를 기다리지 않고 즉시 종료 코드를 반환하며, 실행된 명령어와 백엔드가 출력한 마지막 줄들을 함께 반환합니다. 종료 코드 98은 예외입니다: 다른 백엔드가 이미 해당 포트나 잠금 파일을 점유하고 있는 경우(Chloros 앱, 백엔드 서비스, 또는 아직 시작 중인 다른 스크립트 등)이므로, 호출은 해당 백엔드가 응답할 때까지 계속 대기합니다. `ChlorosLocal`도 동일하게 동작합니다.
- **백엔드는 재사용을 위해 실행 상태로 유지됩니다** (CLI와 동일) — 스크립트가 종료되어도 암묵적인 종료는 발생하지 않습니다. 스크립트를 다시 실행하면 실행 중인 백엔드가 재사용됩니다.
- **해당 호출 중 어느 것이든 `auto_start_backend=False`**를 사용하여 이 기능을 해제할 수 있습니다(예: 원격 백엔드를 지정했거나 백엔드 수명 주기를 직접 관리하는 경우).

```python
import chloros_sdk

# Fresh shell, no backend running, no GUI open — this still works:
with chloros_sdk.connect_camera("213800234") as cam:   # spawns the backend
    cam.capture("output/")

# Remote backend (via tunnel — see Remote-Backend Mode): don't spawn one locally
arr = chloros_sdk.connect_array(serials,
                                backend_url="http://127.0.0.1:5000",
                                auto_start_backend=False)
```

번들된 바이너리를 찾을 수 없거나 시작할 수 없는 경우, 후속 HTTP 호출은 단순한 연결 거부 트레이스 대신 조치 가능한, **플랫폼 인식** `ChlorosConnectError`를 발생시킵니다 — Windows에서는 데스크톱 앱이나 `chloros-cli` 명령어를 안내하며, Linux(GUI 없음)에서는 `chloros-cli` 명령어나 `.deb`를 안내합니다.

---

## 환경 및 헤더

SDK는 모든 백엔드 HTTP 호출에 `X-Chloros-Client: sdk`를 표시합니다. 백엔드는 GUI 무료 티어 경로 대신 SDK/CLI 라이선스 규칙(로그인 **및** 유료 Chloros+ 요금제 필수)을 적용합니다. 이는 가져오기 시점에 자동으로 설정되므로 별도로 조치를 취할 필요가 없습니다.

이 헤더는 URL에 `localhost` 또는 `127.0.0.1`가 포함된 모든 `requests` / `urllib` 호출에 추가되며, URL에 `localhost` 또는 `127.0.0.1`가 포함된 모든 포트(사용자 자신의 로컬 서비스 포함)에 적용됩니다 —에 관계없이, URL에 `localhost` 또는 `127.0.0.1`가 포함된 모든 `requests` /`urllib` 호출에 헤더가 추가되며, 사용자가 직접 설정한 값을 대체합니다. 다른 호스트에 대한 호출은 변경되지 않습니다.

`backend_url=`를 전달하여 백엔드 URL을 재정의할 수 있습니다(`ChlorosLocal`는 `backend_url=` 또는 `api_url=`를 받으며, `process_folder()`와 `process_lattice_capture()`는 오직 `api_url=`만 받음):

```python
chloros_sdk.connect_camera("213800234", backend_url="http://127.0.0.1:5000")
chloros_sdk.connect_array(serials, backend_url="http://127.0.0.1:5000")
chloros_sdk.connect_daq_sensor(eth_host="daq-e-1.local",
                                backend_url="http://127.0.0.1:5000")
chloros_sdk.ChlorosLocal(backend_url="http://127.0.0.1:5000")
```

(설치된 백엔드는 루프백에서만 수신 대기하므로, LAN상의 `backend_url`는 해당 백엔드에 접속할 수 없습니다. 터널 패턴에 대해서는 [원격 백엔드 모드](#remote-backend-mode-pip-only-host-via-tunnel)을 참조하십시오.)

---

## 버전 관리 및 호환성

- SDK 버전은 `chloros_sdk.__version__`로 노출됩니다.
- SDK는 동작을 번들된 백엔드 버전에 고정합니다. 구버전 SDK와 신버전 백엔드를 함께 사용하는 것은 대개 정상적으로 작동하지만(앞으로 호환되는 엔드포인트), 신버전 SDK와 구버전 백엔드를 함께 사용할 경우 새로운 엔드포인트에서 `404` 오류가 발생할 수 있습니다. 이 경우 데스크톱 앱을 해당 버전에 맞게 업그레이드하십시오.
- 스마트 커넥트 인터페이스(`connect_camera` / `connect_array` / `connect_daq_sensor`)와 네트워크 분석 엔드포인트는 안정적인 JSON 스키마를 반환하며, 새로운 필드는 추가되는 방식입니다.

---

## 문제 해결 지침

- **`ChlorosAuthenticationError: Login required`** (`ChlorosLocal`에서 발생; 동일한 원인으로 스마트 커넥트 호출 시 `ChlorosConnectError: Login required. …` 오류가 발생합니다) → 해당 컴퓨터에서 `chloros-cli login EMAIL PASSWORD`를 한 번 실행하거나, Chloros 데스크톱 앱을 통해 로그인하십시오.
- **`ChlorosProcessingError: Failed to start processing: … The project folder is too deep for this computer …`** (또는 `create_project()`에서 파생된 `Failed to create project: …`) → 이 컴퓨터에서는 Windows 긴 경로 기능이 비활성화되어 있습니다(Windows 기본 설정). 따라서 경로 길이가 260자를 초과할 수 없으며, 제품들은 프로젝트 폴더 아래에서 최대 약 130자 위치까지 배치됩니다. 아무 것도 처리되지 않았습니다. 이 메시지는 허용되는 가장 긴 프로젝트 폴더 이름을 표시합니다. 더 짧은 폴더를 사용하거나, Windows 긴 경로 기능을 켜고 Chloros를 다시 시작하십시오. Windows 설치 프로그램에서는 이 설정을 "Windows 긴 경로 사용"으로 제공하며, 기본적으로 선택되어 있습니다. 그렇지 않은 경우 그룹 정책을 사용하십시오: *컴퓨터 구성 > 관리 템플릿 > 시스템 > 파일 시스템 > Win32 긴 경로 사용*.
- **`ChlorosConnectError: No Chloros backend is running …`** → 스마트 커넥트는 로컬 백엔드를 자동으로 시작하도록 호출하므로, 이 오류는 번들된 바이너리를 찾을 수 없거나 시작할 수 없는 경우(예: 데스크톱 패키지가 없는 pip만 있는 호스트 등). 이 메시지는 플랫폼에 따라 다르게 처리됩니다. Windows에서는 데스크톱 앱을 열거나 `chloros-cli` 명령을 실행하십시오. Linux에서는 `chloros-cli` 명령을 실행하거나(GUI가 없음) `.deb`를 설치하십시오. 원격 백엔드의 경우, `backend_url=` (및 `auto_start_backend=False`)를 전달하십시오.
- **`CAMERA_AVAILABLE == False`** 임포트 시 → `lattice_sdk` 로딩 실패: Arena SDK 런타임 또는 OpenCV (`cv2`)는 이 Python에서 임포트할 수 없습니다. 백엔드 기반 서피스(`connect_*`, 백엔드를 통해 렌더링됨, `ChlorosLocal`)는 여전히 작동합니다.
- **센서 전체를 연결하지만 프레임 속도가 낮은 경우**→ 설계상: 자동 모드는 센서 전체를 유지하며, 전송선로에서 처리할 수 있는 수준까지 프레임 속도를 낮춥니다. 더 많은 프레임을 얻으려면 `binning=2` / `4` 또는 더 작은 크기의 `width` / `height`를 전달하여 초당 프레임 수를 높이거나, 점보 프레임을 활성화하거나 링크를 업그레이드하십시오. 비닝 및 ROI는 집계 과부하(`oversubscribed: true`, fps 필드 0)를**해결하지** 못합니다: 네트워크 대역폭 대비 카메라 수가 너무 많은 문제는 비닝/ROI로는 해결할 수 없습니다. 카메라 수를 줄이거나, 점보 프레임을 활성화하거나, 더 빠른 NIC로 교체하십시오([오버서브스크립션](#over-subscription-the-per-cam-floor) 참조).
- **`analyze_array_network()`는 NIC 수신 링 크기가 매우 작음(~0.26 MB)을 보고하며, "FRAMES WILL DROP"** 메시지가 표시된 게이트가 연결됨 → 호스트 NIC의 수신 링이 기본값으로 설정되어 있음(NIC 드라이버 업데이트 후 종종 32로 재설정됨). Realtek USB 10GbE 어댑터의 경우 `ReceiveBufferLen=256` 및 `PendingReceives=64`(권한 상승)를 설정한 후, 백엔드를 재시작하여 링을 다시 읽도록 하십시오. 전체 절차: [CLI 참조 → 호스트 NIC 설정 및 튜닝](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **재시작/종료 시 호스트가 응답하지 않음, 이후 WMI `Invalid class` 오류 발생 / NIC 활성화 불가** → 오래된 USB 10GbE 드라이버로 인해 `DRIVER_POWER_STATE_FAILURE`(BSOD `0x9F`) 오류가 발생합니다. 어댑터 드라이버를 최신 버전(2026 이상)으로 업데이트하고 수신 링 설정을 다시 적용하십시오. [CLI 참조 → 호스트 NIC 설정 및 튜닝](cli-reference.md#host-nic-setup--tuning-lattice-arrays)을 참조하십시오.
- **HTTP 503 "No fresh synced frame group" 오류로 어레이 캡처 실패** → 대기 시간 내에 어레이가 새로운 프레임 그룹을 게시하지 않았습니다. `arr.status()`: `stalled: True` (또는 `streaming: False`)를 참조하십시오. `actual_fps` 값이 0인 경우, 캡처 루프가 배포를 중지했음을 의미합니다 — 어레이의 연결을 끊었다가 다시 연결하십시오. `stalled`가 False이고 `actual_fps`가 0보다 크면 다시 시도하십시오. `connect_array()` 바로 뒤에서 캡처가 첫 번째 그룹을 최대 20초 동안 대기 중입니다.
- **`status()`는 `link_speed_mbps: 4294`를 나타냅니다** → Windows가 어댑터의 속도를 읽을 수 없습니다. 이는 USB 이더넷 어댑터에서 흔히 발생하는 현상이며 오류가 아닙니다. [예상 프레임 속도](#expected-frame-rates)를 참조하십시오.
- **`update_firmware()`가 `device never connected back on TCP <port>`로 인해 실패합니다** → 방화벽이 DAQ-E가 이 컴퓨터로 다시 연결되는 것을 차단했습니다. Windows 설치 프로그램이 해당 규칙을 추가합니다([DAQ-E 장치 설정](#daq-e-device-settings) 참조). 다른 방화벽을 사용하는 경우, 센서 서브넷에서 백엔드로 들어오는 TCP 연결을 허용하십시오.
- **반사율 측정 거부** → 절대 스케일 반사율을 측정하려면 활성 DAQ를 카메라(또는 어레이)에 바인딩해야 합니다. GUI를 통해 바인딩하거나, 페어링된 센서가 필요 없는 `processing="radiance"`(W/m²/sr/nm)를 사용하십시오.
- **`smart=True` 캡처 시간이 예상보다 오래 걸림** → AE 수렴은 장면의 동적 특성에 따라 달라집니다. `ArrayHandle.capture_smart()`를 사용할 경우, `exposure_tolerance_pct`의 폭을 넓히거나, `stability_window_s`의 시간을 단축하거나 `settle_timeout_s`의 값을 낮추면 더 빠르고 안정화되지 않은 트리거를 얻을 수 있습니다.

---

## 참조

- [CLI 참조](cli-reference.md) — 모든 CLI 하위 명령어는 SDK 호출과 일치합니다.
- [DAQ 광 센서](https://mapir.gitbook.io/chloros/daq-light-sensors) (Chloros 매뉴얼) — 센서, 센서 캡 및 보정된 측정 범위, 기록, DAQ-E 네트워킹.
- [Chloros 매뉴얼의 LATTICE 카메라](https://mapir.gitbook.io/chloros/lattice-cameras/lattice) — 연결, 카메라 설정, 캡처 모드, 어레이.
- 온라인 문서: `https://mapir.gitbook.io/chloros/api-python-sdk`
