---
description: 자주 묻는 질문
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/faq
---

# FAQ

<details>

<summary>MAPIR 브랜드가 아닌 카메라의 이미지를 Chloros로 처리할 수 있나요?</summary>

아니요, Chloros는 MAPIR 카메라 이미지 처리만 지원합니다. 자세한 내용은 [지원되는 카메라 모델](supported-cameras.md) 목록을 참조하세요. MAPIR Cloud에서 다른 카메라에 대한 처리 기능을 제공합니다. 전체 목록을 확인하세요([여기](https://mapir.gitbook.io/mapir-cloud/supported-cameras)).

</details>

<details>

<summary>보정 타겟 없이 이미지의 반사율을 보정할 수 있나요?</summary>

아니요. 비대상 이미지를 캡처할 때 캡처된 교정 대상의 이미지가 없으면 이미지의 픽셀 값을 알려진 반사율 백분율과 연결할 수 없습니다. MAPIR 광 센서의 로그도 포함하지 않으면 주변 광 스펙트럼이 측정되지 않으며 반사율 결과가 정확하지 않게 됩니다.

</details>

<details>

<summary>Chloros에서 처리하기 전에 이미지를 편집할 수 있나요?</summary>

아니요. Chloros는 입력 데이터가 수정되지 않았다고 가정합니다. 파일 이름을 변경하지 마십시오.

</details>

<details>

<summary>MAPIR Survey3 카메라를 자동 노출로 설정하고 Chloros에서 이미지를 처리할 수 있나요?</summary>

아니요. Survey3 이미지 데이터세트에는 노출이 고정/고정되어 있어야 하므로 자동 셔터 속도나 자동 ISO가 없습니다. 동일한 카메라 모델의 모든 이미지는 셔터 속도와 ISO(노출)가 동일해야 합니다.

</details>

<details>

<summary>Chloros는 정사모자이크 이미지를 처리하거나 분석할 수 있나요?</summary>

아니요. 개별 MAPIR 카메라 이미지만 지원되며 정사모자이크 지도와 같은 연결된 이미지는 지원되지 않습니다.

</details>

<details>

<summary>Chloros의 타겟 감지 단계를 어떻게 가속화할 수 있나요?</summary>

파일 브라우저 표에서 오른쪽 열의 대상 이미지를 미리 선택하면 Chloros가 해당 이미지에서 보정 대상만 찾도록 지시하여 처리 속도를 크게 높일 수 있습니다.

</details>

<details>

<summary>내 이미지를 <a href="https://www.mapir.camera/collections/software/products/mapir-cloud-subscription">MAPIR Cloud</a>에 업로드하려면 업로드하기 전에 Chloros에서 처리해야 합니까?</summary>

온라인 처리 플랫폼 [마피르 클라우드](https://www.mapir.camera/collections/software/products/mapir-cloud-subscription)에 업로드할 계획이라면 업로드하기 전에 이미지를 편집하지 마세요. 클라우드는 동일한 처리 등을 모두 수행합니다.

</details>

<details>

<summary>MAPIR에서는 X 기능을 지원할 예정인가요? MAPIR에서 X를 제공했으면 좋겠습니다.</summary>

우리는 항상 제품에 대한 피드백을 받는 데 관심이 있습니다. 당사 제품에 문제가 있거나 당사 제품을 개선할 수 있는 방법에 대한 제안 사항이 있는 경우 [문의하기](https://www.mapir.camera/community/contact) 귀하의 생각을 공유해 주십시오. 당사의 R\&D 대부분은 고객의 가장 큰 요구 사항을 경청하는 방식으로 진행됩니다.

</details>
