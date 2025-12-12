# 클로로스+ 로그인

## 클로로스와 클로로스(브라우저) 로그인

사용자 <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> 사이드바 메뉴를 사용하면 Chloros+ 계정에 로그인하고 추가 기능을 잠금 해제할 수 있습니다.

로그인하면 귀하의 계정 세부정보가 표시됩니다:

<figure><img src=".gitbook/assets/user_account.JPG" alt="" width="375"><figcaption></figcaption></figure>

## CLI 로그인

CLI 처리를 활성화하려면 Chloros+ 자격 증명으로 로그인하세요.

**통사론:**

```bash
chloros-cli login <email> <password>
```

**예:**

```powershell
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% 힌트 스타일="경고" %}
**특수 문자**: `$`, `!` 또는 공백과 같은 문자가 포함된 비밀번호 주위에는 작은따옴표를 사용하세요.
{% 끝힌트 %}

**산출:**

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>

### 계획 만료

GUI의 계획 만료는 라이센스가 무효화되는 시기를 보여줍니다. 반복되는 월간 구독의 경우 만료일은 해당 월말입니다. 연간 구독의 경우 구독을 시작한 지 1년이 됩니다. 라이센스 확인을 위해서는 30일의 유예 기간을 포함하여 월별 인터넷 연결이 필요합니다.

### 기기 한도

각 Chloros+ 플랜은 서로 다른 수의 등록된 장치를 제공합니다. Chloros+ 계정으로 로그인하는 각 장치는 등록된 장치 수에 포함됩니다. MAPIR Cloud 계정 페이지에서 장치 이름을 바꾸고 장치를 제거할 수 있습니다.

<table><thead><tr><th width="168.5999755859375" align="right">Chloros+ 요금제</th><th align="center">COPPER</th><th align="center">BRONZE</th><th align="center">SILVER</th><th align="center">GOLD</th></tr></thead><tbody><tr><td align="right">기기 지원됨</td><td align="center">2</td><td align="center">2</td><td align="center">5</td><td align="center">10</td></tr></tbody></table>
