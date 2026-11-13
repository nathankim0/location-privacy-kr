<p align="center">
  <a href="README.md">English</a> · <b>한국어</b>
</p>

# location-privacy-kr

**앱의 위치정보 처리, 약관, 개인정보 처리방침이 위치정보법을 지키는지 파일:줄 근거와 수정안까지 받아 보세요.**

## 무엇인가

Claude Code와 Codex용 에이전트 스킬입니다. 코드와 법정 문서를 함께 읽고 위치정보법·개인정보보호법 기준으로 점검해 보고서를 씁니다. fix 모드에서는 코드를 고치고 빠진 문서 초안을 만듭니다.

## 왜 만들었나

오픈소스 개인정보 스킬 다섯 개를 설치해 실제 앱 두 개(운영 중인 모바일 앱, 오픈소스 위치 공유 앱 OwnTracks)에 돌려 봤습니다. 위치정보법을 다루는 스킬은 하나도 없었습니다. 위치 약관 별도 동의, 이용·제공사실 확인자료 자동 기록, 일시중지 요구권, 즉시 파기, 제3자 제공 사후통보, 14세 미만 규정이 모두 빠졌습니다. 코드를 보지 않고 문서만 평가하기도 했습니다. 한 앱은 요청하지도 않는 "항상 허용" 위치 문구를 선언했습니다. 다른 앱은 백그라운드 정밀 위치를 서버로 보내면서 개인정보 매니페스트에는 수집 데이터를 하나도 적지 않았고, 처리방침은 앱이 아니라 웹사이트 쿠키 이야기뿐이었습니다. 볼 수 없는 서버 쪽 항목을 통과나 위반으로 단정한 스킬도 있었습니다.

이 스킬은 코드부터 봅니다. 위치가 어디서 수집되고 어디로 가고 어디에 남고 언제 지워지는지 지도를 먼저 만들고, 모든 문서를 그 지도와 대조합니다. 볼 수 없는 것은 통과로 쓰지 않고 "확인 불가"로 따로 모읍니다.

## 하는 일

| 모드 | 이렇게 말하면 | 받는 것 |
|---|---|---|
| `audit` (기본) | "위치정보법 준수 점검해줘" | 적용 판정, 위치 데이터 지도, 근거 붙은 발견사항이 담긴 보고서. 대상 코드는 건드리지 않음 |
| `fix` | "고쳐줘", "개선해줘" | 점검 후 저장소 코드 수정, 약관·처리방침 초안, 재점검 |
| `docs` | 약관·처리방침 URL이나 파일 | 문서 체크리스트 결과 |
| `prep` | "실태점검 준비" | 점검에 더해 증빙 목록, 내부 관리지침 목차, 확인자료 테이블 스키마 |

체크리스트는 위치기반서비스 이용약관(T1~T22), 개인정보 처리방침(P1~P20)과 개인위치정보 항목(L1~L7), 동의·권리, 보호조치를 다룹니다. 심각도는 GDPR 등급이 아니라 한국 법의 제재 종류(형사처벌, 과태료, 권고)로 매깁니다. 원본 스킬끼리 부딪힌 17개 지점의 판정은 [`conflicts.md`](skills/location-privacy-kr/references/conflicts.md)에 있습니다.

## 예시

오픈소스 [owntracks/android](https://github.com/owntracks/android), [owntracks/ios](https://github.com/owntracks/ios) 저장소에 돌린 입력:

```text
/location-privacy-kr audit — 이 앱을 한국에 출시한다고 가정하고 위치정보법·개인정보보호법 준수 점검해줘
```

출력 (2026-11-13 실제 실행 요약 발췌):

```text
적용 판정
- 위치정보 처리: 있음 (FusedLocationProvider 연속 측위 GMSLocationProviderClient.kt:91,
  지오펜스 GMSGeofencingClient.kt:29, iOS startUpdatingLocation/startMonitoringVisits LocationManager.m:428,446)
- 개인위치정보: 해당 (좌표 + tid/deviceId 토픽과 함께 서버 전송, LocationProcessor.kt:246-264).
  기본 extendedData=true로 Wi-Fi SSID/BSSID도 함께 전송 (DefaultsProvider.kt:63)
- 사업자 유형(추정): 판단 필요. 앱 기본 서버 없음 (DefaultsProvider.kt:36,80) → 셀프호스팅 구조
- 제3자 제공: 있음(조건부) — 친구 위치 구독 기본 켜짐 (DefaultsProvider.kt:72)

| 치명 1 (조건부) | 높음 5 (조건부) | 보통 5 | 낮음 2 | 확인 불가 8 |

LP-05 높음(조건부) 친구 위치 구독으로 위치가 다른 사용자에게 제공되나
      제3자 제공 고지·별도 동의·사후통보 장치 없음 | 위치정보법 제19조 제2항~제4항 | F6
```

## 필요한 것

- Claude Code, Codex 또는 Agent Skills를 읽는 에이전트
- `git`, `curl` (저장소 검색, 게시된 문서 받기)

## 설치

### Claude Code (플러그인)

```text
/plugin marketplace add nathankim0/location-privacy-kr
/plugin install location-privacy-kr@location-privacy-kr
```

### Claude Code 또는 Codex (직접 복사)

```bash
git clone https://github.com/nathankim0/location-privacy-kr.git
cp -R location-privacy-kr/skills/location-privacy-kr ~/.claude/skills/location-privacy-kr   # Claude Code
cp -R location-privacy-kr/skills/location-privacy-kr ~/.codex/skills/location-privacy-kr    # Codex
```

### Agent Skills를 지원하는 다른 에이전트

```bash
npx skills add nathankim0/location-privacy-kr
```

## 사용법

```text
/location-privacy-kr                       현재 저장소 점검
/location-privacy-kr fix                   점검 후 코드 수정과 문서 초안
/location-privacy-kr docs https://example.com/terms/location
/location-privacy-kr prep                  실태점검 준비
```

Codex에서는 "위치정보법 기준으로 이 저장소 점검해줘"처럼 말하면 설명을 보고 스킬이 켜집니다.

이 스킬은 문서를 게시하거나 배포하지 않고, 신고서를 내지 않으며, 운영 데이터를 건드리지 않습니다. 법률 자문이 아닙니다. 위치기반서비스사업 신고 대상인지 같은 판단은 근거와 함께 "판단 필요"로 남깁니다.

## 저장소 구조

```text
skills/location-privacy-kr/SKILL.md            워크플로
skills/location-privacy-kr/references/
  applicability.md                             위치정보 해당 여부, 사업자 유형
  code-audit.md                                코드에서 위치 데이터 지도 만들기
  terms-checklist.md                           위치기반서비스 이용약관 T1~T22
  privacy-policy-checklist.md                  처리방침 P1~P20, 개인위치정보 L1~L7
  consent-and-rights.md                        동의, 철회, 일시중지, 파기, 사후통보, 14세 미만
  safeguards.md                                확인자료, 접근기록, 통제, 실태점검 준비
  penalties.md                                 심각도와 제재
  fix-patterns.md                              코드 수정 패턴과 문서 골격
  report-template.md                           보고서 형식
  conflicts.md                                 원본 스킬 간 충돌 판정
.claude-plugin/                                Claude Code 플러그인·마켓플레이스 매니페스트
```

## 크레딧

오픈소스 개인정보 스킬 다섯 개를 실제로 돌려 비교해 만들었습니다. [ATTRIBUTIONS.md](ATTRIBUTIONS.md)를 보세요. 법령 체크리스트는 조문을 풀어 쓴 것이며, 방송미디어통신위원회·KISA의 2026년 사업자 대상 위치정보 보호 교육자료(공개)를 참고했습니다.

## 라이선스

[MIT](LICENSE)
