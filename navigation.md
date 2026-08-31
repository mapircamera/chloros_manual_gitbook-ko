# GUI : 탐색

Chloros를 처음 실행하면 처리 백엔드가 시작됩니다. 백엔드 준비가 완료되면 왼쪽 상단의 메인 메뉴 아이콘<img src=".gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line">이 표시되고, 왼쪽 사이드바의 ‘카메라’ 및 ‘광 센서’ 탭이 활성화됩니다(그 전까지는 비활성화 상태로 표시됩니다).

<figure><img src=".gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

상단 헤더에는 왼쪽부터 오른쪽 순서로 다음 항목이 표시됩니다.

### <img src=".gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line"> 메인 메뉴

<figure><img src=".gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

메인 메뉴에서 다음 작업을 수행할 수 있습니다:

* **새 프로젝트**— 새 프로젝트를 생성합니다. 저장된 프로젝트 템플릿이 있는 경우,**템플릿 선택** 드롭다운 메뉴가 표시되어 새 프로젝트를 템플릿의 설정으로 시작할 수 있습니다.
* **프로젝트 열기**— 기존 프로젝트를 엽니다. 이 목록에는 파일 탐색기에서 프로젝트 폴더를 여는**프로젝트 폴더 열기** 버튼이 포함되어 있습니다.
* **프로젝트 복제** — 현재 열려 있는 프로젝트를 새 이름으로 복사하고(예: &quot;MyProject (2)&quot;와 같은 자유로운 이름이 제안됨) 복사본을 엽니다. _(프로젝트 열기 후 표시됨)_
* **파일 추가** — 개별 이미지 파일을 현재 프로젝트에 추가합니다 _(프로젝트 열기 후 표시됨)_
* **폴더 추가** — 하나 이상의 이미지 폴더를 현재 프로젝트에 추가합니다 _(프로젝트 열기 후 표시됨)_
* **처리 시작 / 처리 중지** — 이미지 처리 파이프라인을 시작하거나 중지합니다 _(파일 추가 후 활성화됨)_
* **카메라 연결** — [카메라 탭](lattice/)으로 이동하여 LATTICE 카메라 또는 어레이를 연결합니다. 프로젝트가 열려 있지 않아도 작동합니다.
* **광 센서 연결** — [광 센서 탭](daq/)으로 이동하여 DAQ 광 센서를 연결합니다. 프로젝트가 열려 있지 않아도 작동합니다.

{% hint style="info" %}
**Windows 전용**: Chloros 데스크톱 GUI는 Windows에서 사용할 수 있습니다. Linux 사용자는 [CLI](CLI.md) 및 [Python SDK](api-python-sdk.md) 문서를 참조하십시오.
{% endhint %}

###<img src=".gitbook/assets/image (2) (1) (1).png" alt="" data-size="line">

재생/시작 버튼

이 기능이 활성화되면 처리 시작 버튼을 눌러 이미지 처리 파이프라인을 시작할 수 있습니다.

###<img src=".gitbook/assets/image (4).png" alt="" data-size="line">

진행률 표시줄<img src=".gitbook/assets/image (5).png" alt="" data-size="line">

모든 파일을 순차적으로 처리하는 무료 Chloros 모드에서는 진행률 표시줄에 ‘대상 감지’와 ‘처리’라는 두 단계가 표시됩니다.

모든 파일을 동시에 처리하는 유료 Chloros+ 라이선스 모드에서는 진행률 표시줄에 ‘탐지’, ‘분석’, ‘보정’, ‘내보내기’의 4단계가 표시됩니다. Chloros+ 진행률 표시줄 위에 마우스 커서를 올리면 확장된 4단계 진행률 패널이 드롭다운으로 표시되어 진행 상황을 확인할 수 있습니다. 상단의 진행률 표시줄을 클릭하면 드롭다운 패널이 고정되고, 다시 클릭하면 고정 해제됩니다.

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

## 사이드 메뉴

왼쪽 사이드바 메뉴에는 상호작용할 수 있는 다양한 아이콘이 포함되어 있으며, 상단부터 하단까지 순서대로 다음과 같습니다:

#### <img src=".gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> [프로젝트 설정](project-settings/project-settings.md)

프로젝트 설정 탭에서는 프로젝트의 전체 설정 및 처리 설정을 조정할 수 있습니다. 파일 처리를 시작하기 전에 이 설정들을 조정하십시오.

#### <img src=".gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> 파일 브라우저

프로젝트에 파일/폴더를 추가하거나 제거할 수 있습니다. 중복 파일은 무시됩니다. 대상 이미지 옆의 확인란을 선택하면, 처리 과정에서 대상 이미지로 지정된 이미지만 처리되므로 처리 속도가 크게 향상됩니다. ‘이미지/메타데이터(Image/Metadata)’ 토글을 사용하여 선택한 이미지의 썸네일 그리드 보기와 상세 메타데이터 표 보기 사이를 전환할 수 있습니다.

#### <img src=".gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> [이미지 뷰어](image-viewer-gui/opening-an-image-full-screen.md)

메인 이미지 뷰어에서 이미지를 클릭하면 ‘이미지 뷰어’ 탭에서 전체 화면으로 열립니다.

#### <img src=".gitbook/assets/image (3) (1).png" alt="" data-size="line"> [지도 뷰어](image-viewer-gui/map-markers.md)

GPS 좌표를 기반으로 대화형 2D 지도에서 이미지를 확인하세요. Google Maps 및 ESRI 타일 제공자를 지원하며, 사용자의 위치에 가장 적합한 서비스를 자동으로 선택합니다. 마커 위에 마우스를 올리면 이미지 썸네일 미리보기를 확인할 수 있습니다.

#### <img src=".gitbook/assets/image (17).png" alt="" data-size="line"> [카메라](lattice/)

LATTICE 카메라를 실시간으로 연결하고 제어할 수 있습니다. 카메라를 하나씩 개별적으로 사용하거나, 동기화된 다중 카메라 어레이로 사용할 수 있습니다. 이 탭에서는 오버레이 및 히스토그램이 포함된 실시간 미리보기 타일, 카메라별 및 어레이별 설정, 그리고 “모두 캡처(Capture All)” 기능이 어떤 카메라와 내보내기 유형을 생성할지 선택하는 캡처 설정을 확인할 수 있습니다. 백엔드 준비가 완료되면 사용 가능하며, 전체 사용 방법은 [LATTICE 섹션](lattice/)을 참조하십시오.

#### <img src=".gitbook/assets/image (23).png" alt="" data-size="line"> [광 센서](daq/)

DAQ 광 센서(DAQ-U(USB), DAQ-M(블루투스), DAQ-E(이더넷))를 연결하고, W/m²/nm 단위로 보정된 실시간 스펙트럼 차트를 확인하세요. 여기서 열린 프로젝트에 `.daq` 파일을 기록하고, 센서 이름을 변경하며, 캡 보정 프로필을 선택하고, DAQ-E 펌웨어를 업데이트할 수 있습니다. 백엔드가 준비되면 사용할 수 있으며, 전체 사용 방법은 [DAQ 섹션](daq/)을 참조하십시오.

#### <img src=".gitbook/assets/icon_log.JPG" alt="" data-size="line"> 디버그 로그

문제가 발생할 경우 로그에서 디버그 출력 내용을 확인하십시오. 로그를 복사하거나 다운로드하여 [MAPIR 지원팀](https://www.mapir.camera/community/contact)에 보내면 도움을 받으실 수 있습니다.

#### <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> [사용자 로그인](chloros+-login.md)

사용자 로그인 사이드바를 통해 Chloros+ 계정에 로그인하여 고급 기능을 이용할 수 있습니다. 또한 현재 애플리케이션 버전을 확인하거나, Chloros GUI 및 CLI에서 표시되는 텍스트의 언어를 조정할 수도 있습니다.
