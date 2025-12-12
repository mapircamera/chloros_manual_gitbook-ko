---
description: Frequently Asked Questions
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/faq
---

# 자주 묻는 질문

<details>

<summary>Chloros로 MAPIR 브랜드가 아닌 카메라의 이미지를 처리할 수 있나요?</summary>

아니요, Chloros는 MAPIR 카메라 이미지만 처리할 수 있습니다. 자세한 내용은 [지원되는 카메라 모델](supported-cameras.md) 목록을 참조하십시오. MAPIR Cloud에서는 다른 카메라의 처리도 제공하며, 전체 목록은 [여기](https://mapir.gitbook.io/mapir-cloud/supported-cameras)에서 확인할 수 있습니다.

</details>

<details>

<summary>교정 대상 없이 반사율에 대한 이미지 교정이 가능한가요?</summary>

아니요. 비표적 이미지 촬영 시점에 함께 촬영된 교정 타겟 이미지가 없으면, 이미지의 픽셀 값을 알려진 반사율 백분율과 연관 지을 수 없습니다. 또한 MAPIR 광 센서의 로그를 포함하지 않으면 주변광 스펙트럼이 측정되지 않아 반사율 결과가 정확하지 않을 수 있습니다.

</details>

<details>

<summary>Chloros에서 처리하기 전에 이미지를 편집할 수 있나요?</summary>

아니요. Chloros는 입력 데이터가 수정되지 않았다고 가정합니다. 파일 이름을 변경하지 마십시오.

</details>

<details>

<summary>MAPIR Survey3 카메라를 자동 노출로 설정하고 Chloros에서 이미지를 처리할 수 있나요?</summary>

아니요. Survey3 이미지 데이터 세트는 노출이 고정/잠겨 있어야 하므로 자동 셔터 속도나 자동 ISO를 사용할 수 없습니다. 동일한 카메라 모델의 모든 이미지는 동일한 셔터 속도와 ISO(노출)를 가져야 합니다.

</details>

<details>

<summary>Chloros는 정사영 모자이크 이미지를 처리하거나 분석할 수 있습니까?</summary>

아니요. 개별 MAPIR 카메라 이미지만 지원되며, 정사모자이크 지도와 같은 스티칭된 이미지는 지원되지 않습니다.

</details>

<details>

<summary>Chloros의 대상 감지 단계를 어떻게 가속화할 수 있나요?</summary>

파일 브라우저 테이블에서 오른쪽 열에 있는 대상 이미지를 미리 선택하면 Chloros가 해당 이미지만에서 보정 대상을 찾도록 지시하여 처리 속도를 크게 높일 수 있습니다.

</details>

<details>

<summary>이미지를 <a href="https://www.mapir.camera/collections/software/products/mapir-cloud-subscription">MAPIR Cloud</a>에 업로드할 예정이라면 업로드 전에 Chloros에서 처리해야 할까요?</summary>

온라인 처리 플랫폼 [MAPIR Cloud](https://www.mapir.camera/collections/software/products/mapir-cloud-subscription)에 업로드할 계획이라면 업로드 전에 이미지를 편집하지 마십시오. 클라우드에서 동일한 처리와 추가 작업을 모두 수행합니다.

</details>

<details>

<summary>MAPIR가 X 기능을 지원할 예정인가요? MAPIR가 X 기능을 제공했으면 좋겠습니다.</summary>

저희는 항상 제품에 대한 피드백을 환영합니다. 제품 관련 문제점을 발견하거나 개선 제안을 하실 경우 [문의하기](https://www.mapir.camera/community/contact)를 통해 의견을 공유해 주십시오. 저희 R&amp;D의 대부분은 고객의 가장 큰 요구사항을 경청하는 데서 방향을 잡습니다.

</details>
