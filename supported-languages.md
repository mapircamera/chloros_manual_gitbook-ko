# 지원 언어

Chloros는 **전 세계 38개 언어**에 대한 완벽한 인터페이스 지원을 제공하여, 전 세계 사용자가 편리하게 이용할 수 있습니다. 데스크톱 GUI와 CLI 모두에서 즉시 언어를 전환할 수 있습니다.

Chloros는 다음 언어를 지원합니다:

| # | 언어 | 원어명 | CLI 코드 |
|---|----------|-------------|----------|
| 1 | 🇺🇸 영어 | English | `en` |
| 2 | 🇪🇸 스페인어 | Español | `es` |
| 3 | 🇵🇹 포르투갈어 | Português | `pt` |
| 4 | 🇫🇷 프랑스어 | Français | `fr` |
| 5 | 🇩🇪 독일어 | Deutsch | `de` |
| 6 | 🇮🇹 이탈리아어 | Italiano | `it` |
| 7 | 🇯🇵 일본어 | 日本語 | `ja` |
| 8 | 🇰🇷 한국어 | 한국어 | `ko` |
| 9 | 🇨🇳 중국어(간체) | 简体中文 | `zh` |
| 10 | 🇹🇼 중국어(번체) | 繁體中文 | `zh-TW` |
| 11 | 🇷🇺 러시아어 | Русский | `ru` |
| 12 | 🇳🇱 네덜란드어 | Nederlands | `nl` |
| 13 | 🇸🇦 아랍어 | العربية | `ar` |
| 14 | 🇵🇱 폴란드어 | Polski | `pl` |
| 15 | 🇹🇷 터키어 | Türkçe | `tr` |
| 16 | 🇮🇳 힌디어 | हिंदी | `hi` |
| 17 | 🇮🇩 인도네시아어 | Bahasa Indonesia | `id` |
| 18 | 🇻🇳 베트남어 | Tiếng Việt | `vi` |
| 19 | 🇹🇭 태국어 | ไทย | `th` |
| 20 | 🇸🇪 스웨덴어 | Svenska | `sv` |
| 21 | 🇩🇰 덴마크어 | Dansk | `da` |
| 22 | 🇳🇴 노르웨이어 | Norsk | `no` |
| 23 | 🇫🇮 핀란드어 | Suomi | `fi` |
| 24 | 🇬🇷 그리스어 | Ελληνικά | `el` |
| 25 | 🇨🇿 체코어 | Čeština | `cs` |
| 26 | 🇭🇺 헝가리어 | Magyar | `hu` |
| 27 | 🇷🇴 루마니아어 | Română | `ro` |
| 28 | 🇺🇦 우크라이나어 | Українська | `uk` |
| 29 | 🇧🇷 브라질 포르투갈어 | Português Brasileiro | `pt-BR` |
| 30 | 🇭🇰 광둥어 | 粵語 | `zh-HK` |
| 31 | 🇲🇾 말레이어 | Bahasa Melayu | `ms` |
| 32 | 🇸🇰 슬로바키아어 | Slovenčina | `sk` |
| 33 | 🇧🇬 불가리아어 | Български | `bg` |
| 34 | 🇭🇷 크로아티아어 | Hrvatski | `hr` |
| 35 | 🇱🇹 리투아니아어 | Lietuvių | `lt` |
| 36 | 🇱🇻 라트비아어 | Latviešu | `lv` |
| 37 | 🇪🇪 에스토니아어 | Eesti | `et` |
| 38 | 🇸🇮 슬로베니아어 | Slovenščina | `sl` |

## 언어 변경 방법

### Chloros 데스크톱에서

1. 애플리케이션 설정을 엽니다.
2. 언어 선택 메뉴로 이동합니다.
3. 목록에서 원하는 언어를 선택합니다.
4. 인터페이스가 즉시 업데이트됩니다.

### Chloros에서 CLI

`language` 명령어를 사용하여 CLI 인터페이스 언어를 확인하거나 변경하십시오:

```bash
# View current language
chloros-cli language

# Change to Spanish
chloros-cli language es

# Change to Chinese (Simplified)
chloros-cli language zh

# Change to Brazilian Portuguese
chloros-cli language pt-BR

# List all available languages
chloros-cli language --list
```

자세한 내용은 [CLI 설명서](CLI.md)를 참조하십시오.

## 지원 범위

다음 항목 전반에 걸쳐 38개 언어가 모두 완벽하게 지원됩니다:

* **Chloros 데스크톱** - 전체 GUI 번역
* **Chloros CLI** - 명령줄 인터페이스 및 출력 메시지

Python, SDK, API 및 해당 [참조 문서](reference/sdk-reference.md)는 영어로 제공됩니다.

언어 지원 덕분에 전 세계 사용자들이 장벽 없이 모국어로 효율적으로 작업할 수 있습니다.
