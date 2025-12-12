# Chloros+ 로그인

## Chloros 및 Chloros (브라우저) 로그인

사용자 <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> 사이드바 메뉴를 통해 Chloros+ 계정에 로그인하고 추가 기능을 사용할 수 있습니다.

로그인 시 계정 정보가 표시됩니다:

<figure><img src=".gitbook/assets/user_account.JPG" alt="" width="375"><figcaption></figcaption></figure>

## CLI 로그인

CLI 처리를 활성화하려면 Chloros+ 자격 증명으로 로그인하십시오.

**구문:**

```bash
chloros-cli login <email> <password>
```

**예시:**

```powershell
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style=&quot;warning&quot; %}
**특수 문자**: `$`, `!` 또는 공백과 같은 문자가 포함된 비밀번호는 작은따옴표(&#x27;&#x27;)로 묶어 사용하십시오.
{% endhint %}

**출력:**

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>

### 플랜 만료

GUI에 표시되는 플랜 만료일은 라이선스가 무효화되는 시점을 나타냅니다. 월간 정기 구독의 경우 만료일은 해당 월 말일입니다. 연간 구독의 경우 구독 시작일로부터 1년 후입니다. 라이선스 확인은 월간 인터넷 연결을 통해 검증되며, 30일의 유예 기간이 적용됩니다.

### 기기 제한

각 Chloros+ 플랜은 등록 가능한 기기 수가 다릅니다. Chloros+ 계정으로 로그인하는 모든 기기는 등록 기기 수에 포함됩니다. MAPIR 클라우드 계정 페이지에서 기기 이름을 변경하거나 제거할 수 있습니다.

<table><thead><tr><th width="168.5999755859375" align="right">XPROTX 플랜</th><th align="center">구리</th><th align="center">브론즈</th><th align="center">실버</th><th align="center">골드</th></tr></thead><tbody><tr><td align="right">지원 기기</td><td align="center">2</td><td align="center">2</td><td align="center">5</td><td align="center">10</td></tr></tbody></table>
