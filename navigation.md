# GUI : 탐색

Chloros 및 Chloros(브라우저)를 처음 실행하면 백엔드가 시작됩니다. 준비가 완료되면 왼쪽 상단의 메인 메뉴 아이콘이 표시됩니다. <img src=".gitbook/assets/image (1) (1) (1).png" alt="" data-size="line"> .

<figure><img src=".gitbook/assets/header.JPG" alt=""><figcaption></figcaption></figure>

상단 헤더에는 왼쪽부터 오른쪽 순서로 다음 항목이 포함됩니다:

### <img src=".gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line"> 메인 메뉴

<figure><img src=".gitbook/assets/image (25).png" alt=""><figcaption></figcaption></figure>

메인 메뉴에서 다음 작업을 수행할 수 있습니다:

* **새 프로젝트** — 새 프로젝트 생성
* **프로젝트 열기** — 기존 프로젝트 열기
* **프로젝트 폴더 열기** — 파일 탐색기에서 프로젝트 폴더 열기
* **파일 추가** — 현재 프로젝트에 개별 이미지 파일 추가 _(프로젝트 열기 후 표시됨)_
* **폴더 추가** — 이미지 폴더를 현재 프로젝트에 추가합니다 _(프로젝트 열기 후 표시됨)_
* **처리 시작 / 처리 중지** — 이미지 처리 파이프라인을 시작하거나 중지합니다 _(파일 추가 후 활성화됨)_

{% hint style="info" %}
**Windows 전용**: Chloros 데스크톱 GUI는 Windows에서 사용할 수 있습니다. Linux 사용자는 [CLI](CLI.md) 및 [Python SDK](api-python-sdk.md) 문서를 참조하십시오.
{% endhint %}

### <img src=".gitbook/assets/image (2) (1).png" alt="" data-size="line"> 재생/시작 버튼

이 기능이 활성화되면 시작 처리 버튼을 눌러 이미지 처리 파이프라인을 시작할 수 있습니다.

### <img src=".gitbook/assets/image (4).png" alt="" data-size="line"> 진행률 표시줄 <img src=".gitbook/assets/image (5).png" alt="" data-size="line">모든 파일을 순차적으로 처리하는 무료 Chloros 모드에서는 진행률 표시줄에 &#x27;타겟 감지(Target Detect)&#x27;와 &#x27;처리(Processing)&#x27;의 두 단계가 표시됩니다.

모든 파일을 동시에 처리하는 유료 Chloros+ 라이선스 모드에서는 진행률 표시줄에 &#x27;감지(Detecting)&#x27;, &#x27;분석(Analyzing)&#x27;, &#x27;보정(Calibrating)&#x27;, &#x27;내보내기(Exporting)&#x27;의 네 단계가 표시됩니다. Chloros+ 진행률 표시줄 위에 마우스 커서를 올리면 4단계의 상세 진행률 패널이 드롭다운되어 진행 상황을 확인할 수 있습니다. 상단 진행률 표시줄을 클릭하면 드롭다운 패널이 고정되고, 다시 클릭하면 고정 해제됩니다.

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

## 사이드 메뉴

왼쪽 사이드바 메뉴에는 상호 작용할 수 있는 다양한 아이콘이 있습니다:

#### <img src=".gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> [프로젝트 설정](project-settings/project-settings.md)

프로젝트 설정 탭에서는 프로젝트의 전체 설정 및 처리 설정을 조정할 수 있습니다. 파일 처리를 시작하기 전에 이 설정들을 조정하십시오.

#### <img src=".gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> 파일 브라우저

프로젝트에 파일/폴더를 추가하거나 제거할 수 있습니다. 중복 파일은 무시됩니다. 대상 이미지 옆의 확인란을 선택하면, 처리 과정에서 대상 이미지로 지정된 이미지만 검사하므로 처리 속도가 크게 향상됩니다. &#x27;이미지/메타데이터&#x27; 토글을 사용하여 선택한 이미지의 썸네일 그리드와 상세 메타데이터 표를 전환하여 볼 수 있습니다.

#### <img src=".gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> [이미지 뷰어](image-viewer-gui/opening-an-image-full-screen.md)

메인 이미지 뷰어에서 이미지를 클릭하면 이미지 뷰어 탭에서 전체 화면으로 열립니다.

#### <img src=".gitbook/assets/image (7).png" alt="" data-size="line"> [지도](image-viewer-gui/map-markers.md)

GPS 좌표를 기반으로 상호작용이 가능한 2D 지도에서 이미지를 확인하세요. Google Maps 및 ESRI 타일 제공자를 지원하며, 사용자의 위치에 가장 적합한 서비스를 자동으로 선택합니다. 마커 위에 마우스를 올리면 이미지 썸네일 미리보기를 볼 수 있습니다.

#### <img src=".gitbook/assets/icon_log.JPG" alt="" data-size="line"> 디버그 로그

문제가 발생하면 로그에서 디버그 출력 내용을 확인하세요. 로그를 복사하거나 다운로드하여 [MAPIR 지원팀](https://www.mapir.camera/community/contact)에 보내 도움을 받으세요.

#### <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> [사용자 로그인](chloros+-login.md)

사용자 로그인 사이드바를 통해 Chloros+ 계정에 로그인하여 고급 기능을 이용할 수 있습니다. 또한 현재 애플리케이션 버전을 확인하고, Chloros GUI 및 CLI에서 표시되는 텍스트의 언어를 설정할 수 있습니다.
