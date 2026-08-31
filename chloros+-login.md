# Chloros+ 로그인

## GUI 로그인

<img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> 사이드바 메뉴를 통해 Chloros+ 계정에 로그인하고 추가 기능을 이용할 수 있습니다.

**한 대의 컴퓨터당 한 번만 로그인하면 됩니다.** GUI, CLI 및 Python SDK는 동일한 캐시된 세션을 공유합니다 — 데스크톱 GUI를 통해 로그인하면 해당 기기에서 CLI와 SDK도 활성화됩니다(반대로 `chloros-cli login`를 통해서도 마찬가지입니다).

로그인하면 계정 세부 정보가 표시됩니다:

<figure><img src=".gitbook/assets/user_account.JPG" alt="" width="375"><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: re-shoot the logged-in user account panel in Chloros 1.2.0 — plan name display and the registered-device list UI may have changed; must show plan name, expiration, and device list. -->
## 요금제 등급

| 요금제 | `plan_id` | 유형 |
| --- | --- | --- |
| Iron | `0` | 무료 |
| Copper | `1` | 유료 (Chloros+) |
| 브론즈 | `2` | 유료 (Chloros+) |
| 실버 | `3` | 유료 (Chloros+) |
| 골드 | `4` | 유료 (Chloros+) |

각 유료 등급에 포함된 내용은 [요금제 및 가격](https://cloud.mapir.camera/pricing)을 참조하십시오.

### CLI / SDK에 액세스하려면 유료 요금제가 필요합니다

CLI 및 Python, SDK에 접근하려면 **유료 Chloros+ 요금제(Copper 이상)**가 필요합니다. 이는**서버 측**에서 강제 적용됩니다. 모든 CLI/SDK 요청에는 활성 세션과 유료 요금제가 모두 포함되어야 합니다:

| HTTP 상태 | `error_code` | 의미 | 해결 방법 |
| --- | --- | --- | --- |
| `401` | `AUTH_REQUIRED` | 이 기기에서 로그인되지 않음 | `chloros-cli login <email> <password>` |
| `403` | `PLAN_UPGRADE_REQUIRED` | 로그인은 되었으나 요금제 등급이 너무 낮음(무료 Iron 등급) | 유료 Chloros+ 요금제로 업그레이드 |

`chloros-cli status`는 무료 티어에서도 계속 이용할 수 있으므로, 현재 요금제와 액세스가 거부된 이유를 언제든지 확인할 수 있습니다.

### 요금제별 연결 가능한 하드웨어 제한

각 요금제마다 한 번에 실시간으로 연결할 수 있는 LATTICE 카메라 및 DAQ 광센서의 수에 상한이 있습니다:

| 요금제 | LATTICE 카메라 | DAQ 광 센서 |
| --- | --- | --- |
| Iron (무료 / 로그인 미완료) | 4 | 2 |
| Copper / Bronze | 6 | 3 |
| Silver | 10 | 6 |
| Gold | 20 | 12 |

## CLI 로그인

Chloros+ 자격 증명으로 로그인하여 CLI 처리를 활성화하십시오. Linux(GUI 없음)에서는 이 방법이 라이선스를 활성화하는 유일한 방법입니다.

**구문:**

```bash
chloros-cli login <email> <password>
```

{% hint style="info" %}
**SDK 사용자**: Python SDK는 캐시된 자격 증명을 지우는 프로그래밍 방식의 `logout()` 메서드도 제공합니다. 자세한 내용은 [SDK 참조](reference/sdk-reference.md)를 참조하십시오.
{% endhint %}

**예시:**

```powershell
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style="warning" %}
**특수 문자**: `$`, `!` 또는 공백과 같은 문자가 포함된 비밀번호는 작은 따옴표로 묶어 사용하십시오.
{% endhint %}

**출력:**

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: re-shoot the CLI login output — the banner now prints "Chloros CLI 1.2.0"; capture a successful login with the current output format. -->
### 자격 증명 저장

캐시된 자격 증명과 구성 정보는 **모든 플랫폼**에서 사용자 홈 디렉터리의 `.chloros` 폴더에 저장됩니다:

| 플랫폼 | 자격 증명 캐시 경로 |
| --- | --- |
| **Windows** | `%USERPROFILE%\.chloros\` |
| **Linux** | `~/.chloros/` |

### 플랜 만료 및 오프라인 유예 기간

GUI에 표시되는 플랜 만료일은 라이선스가 무효화되는 시점을 나타냅니다. 월간 정기 구독의 경우 만료일은 해당 월 말일이며, 연간 구독의 경우 구독을 시작한 지 1년 후입니다.

Chloros는 라이선스를 온라인으로 검증하지만, 유예 기간 내에서는 오프라인 작업이 지원됩니다:

* 서버 검증에 성공한 기록은 **5분** 동안 캐시되므로, 정상적인 사용 시 라이선스 호출 횟수는 매우 적습니다.
* 서명되고 특정 기기에 바인딩된 라이선스 캐시는 더 긴 오프라인 기간을 지원합니다: **월간 요금제의 경우 30일**,**연간 요금제의 경우 구독 만료일까지(최대 365일)**입니다.
* 유예 기간이 만료되면, 해당 기기가 라이선스 서버에 한 번 접속할 수 있을 때까지 요금제가 무료 Iron 등급으로 전환되며, 다음 번 확인에 성공하면 액세스가 재개됩니다.

### 기기 제한

각 Chloros+ 요금제는 등록 가능한 기기 수가 다릅니다. Chloros+ 계정으로 로그인하는 각 기기는 등록된 기기 수에 포함됩니다. MAPIR 클라우드 계정 페이지에서 기기 이름을 변경하거나 기기를 삭제할 수 있습니다.

<table><thead><tr><th width="168.5999755859375" align="right">Chloros+ 요금제</th><th align="center">COPPER</th><th align="center">BRONZE</th><th align="center">실버</th><th align="center">GOLD</th></tr></thead><tbody><tr><td align="right">지원되는 기기</td><td align="center">2</td><td align="center">2</td><td align="center">5</td><td align="center">10</td></tr></tbody></table>계정의 정확한 기기 허용 한도는 MAPIR 클라우드 계정 페이지에 표시됩니다. 기기에서 로그아웃하면 해당 슬롯이 확실히 해제되며, 이미 등록된 기기는 계정 기기 한도에 도달한 경우에도 언제든지 다시 로그인할 수 있습니다.
