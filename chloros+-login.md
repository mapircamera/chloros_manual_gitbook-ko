# Chloros+ 로그인

## Chloros 및 Chloros (브라우저) 로그인

사용자 <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> 사이드바 메뉴를 통해 Chloros+ 계정에 로그인하고 추가 기능을 이용할 수 있습니다.

로그인하면 계정 정보가 표시됩니다:

<figure><img src=".gitbook/assets/user_account.JPG" alt="" width="375"><figcaption></figcaption></figure>## CLI 로그인

Chloros+ 자격 증명으로 로그인하여 CLI 처리를 활성화하십시오. Linux(GUI 없음)에서는 이 방법이 라이선스를 활성화하는 유일한 방법입니다.

**구문:**

```bash
chloros-cli login <email> <password>
```

{% hint style="info" %}
**SDK 사용자**: Python SDK는 캐시된 자격 증명을 지우는 프로그래밍 방식의 `logout()` 메서드도 제공합니다. 자세한 내용은 [Python SDK 문서](api-python-sdk.md#logout)를 참조하십시오.
{% endhint %}

**예시:**

```powershell
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style="warning" %}
**특수 문자**: `$`, `!` 또는 공백과 같은 문자가 포함된 비밀번호는 작은 따옴표로 묶어 주십시오.
{% endhint %}

**출력:**

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>### 자격 증명 저장

캐시된 자격 증명은 플랫폼별 위치에 저장됩니다:

| 플랫폼 | 자격 증명 캐시 경로 |
| --- | --- |
| **Windows** | `%APPDATA%\Chloros\cache\` |
| **Linux** | `~/.cache/chloros/` |

### 플랜 만료

GUI의 플랜 만료 날짜는 라이선스가 무효화되는 시점을 나타냅니다. 월간 정기 구독의 경우 만료일은 해당 월의 말일입니다. 연간 구독의 경우 구독 시작일로부터 1년 후입니다. 라이선스 확인을 위해서는 매월 인터넷 연결이 필요하며, 30일의 유예 기간이 제공됩니다.

### 기기 제한

각 Chloros+ 요금제는 등록 가능한 기기 수가 다릅니다. Chloros+ 계정으로 로그인하는 각 기기는 등록된 기기 수에 포함됩니다. MAPIR 클라우드 계정 페이지에서 기기 이름을 변경하거나 기기를 제거할 수 있습니다.

<table><thead><tr><th width="168.5999755859375" align="right">Chloros+ 요금제</th><th align="center">COPPER</th><th align="center">BRONZE</th><th align="center">실버</th><th align="center">골드</th></tr></thead><tbody><tr><td align="right">지원되는 기기</td><td align="center">2</td><td align="center">2</td><td align="center">5</td><td align="center">10</td></tr></tbody></table>
