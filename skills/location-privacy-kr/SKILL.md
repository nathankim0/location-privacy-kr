---
name: location-privacy-kr
description: Audit and fix a project's compliance with Korea's Location Information Act (위치정보법) and Personal Information Protection Act (개인정보보호법). Maps where location data is collected, sent, stored and deleted in the code, checks the location-based service terms (위치기반서비스 이용약관), privacy policy (개인정보 처리방침) and consent screens against the statutory items, and reports evidence-backed findings, or patches code and drafts documents in fix mode. Works with any stack (Android, iOS, React Native, Flutter, web, backend). Use when the user says "위치정보법 준수 점검", "개인정보처리방침 검수", "이용약관 점검", "위치기반서비스 약관", "위치정보 실태점검 준비", "location privacy audit Korea", or asks whether an app that uses GPS or location is compliant in Korea.
argument-hint: "[audit|fix|docs|prep] [repo path] [terms/privacy URL or file]"
---

# location-privacy-kr

코드와 문서를 함께 보고, 한국 위치정보법·개인정보보호법 기준으로 빠진 것을 찾아 고친다.

트라이얼에서 얻은 핵심 교훈 두 가지:
1. **문서만 보면 틀린다.** 백그라운드로 정밀 위치를 서버에 보내는 앱이 개인정보 매니페스트에는 수집 데이터를 하나도 적지 않았고, 처리방침은 앱이 아니라 웹사이트 쿠키 얘기만 했다. 그래서 코드에서 위치 데이터 지도를 먼저 만들고, 문서는 그 지도와 대조한다.
2. **볼 수 없는 것은 "확인 불가"다.** 모바일 저장소만으로는 서버 저장, 확인자료 기록을 알 수 없다. 추측해서 통과나 위반으로 쓰지 않는다.

법률 자문이 아니다. 점검 결과와 수정안을 낼 뿐이고, 신고 여부 같은 법적 판단은 근거를 붙여 "판단 필요"로 남긴다.

## 모드

| 요청 | 모드 | 결과 |
|---|---|---|
| "점검해줘", "준수하고 있어?" (기본) | `audit` | 보고서. 대상 코드와 문서는 수정하지 않음 |
| "고쳐줘", "개선해줘" | `fix` | audit + 저장소 안의 코드 수정, 약관·처리방침 초안 파일 |
| "약관/처리방침만 봐줘", URL이나 문서만 줌 | `docs` | 문서 체크리스트 결과 (코드 없으면 대조 항목은 확인 불가) |
| "실태점검 준비", "점검표 준비" | `prep` | audit + 증빙 목록, 내부 관리지침 목차, 확인자료 스키마 |

질문하지 않고 시작한다. 대상 저장소는 현재 디렉터리, 문서는 저장소 안의 약관·처리방침 파일과 코드에 박힌 약관 URL에서 찾는다. 고른 기본값은 보고서 맨 위에 적는다.

## 1단계: 범위 잡기

- 저장소 종류(모바일, 웹, 서버, 모노레포)와 커밋 해시를 적는다.
- 문서 찾기: `git grep -nIiE 'terms|privacy|policy|약관|처리방침|위치기반' -- . ':!*.lproj/**' ':!**/values-*/**' ':!LICENSE*'`로 파일과 URL을 모은다. 앱에 URL이 없으면 프로젝트 웹사이트·문서 폴더, 스토어 메타데이터(`fastlane/metadata` 등)를 본다. 사용자가 준 URL은 모드와 상관없이 받는다.
- 웹 문서는 `curl -sL`로 원문을 받아 텍스트로 저장해 둔다(스크래치 폴더). 요약 도구 결과만으로 판정하지 않는다.
- 플랫폼별 형제 저장소(iOS·Android 분리)가 있으면 최소한 권한·매니페스트·수집 지점 검색은 같이 돌린다.
- 서버 저장소가 따로 있으면 경로를 묻지 말고, 있는 것만 보고 서버 항목은 확인 불가로 둔다. 사용자가 경로를 주면 같이 본다.

## 2단계: 적용 판정

`references/applicability.md`를 따른다.
- 위치정보인가, 개인위치정보인가, 사업자 유형은 무엇으로 보이는가, 제3자 제공이 있는가, 14세 미만이 쓸 수 있는가.
- 위치정보를 전혀 처리하지 않으면 위치정보법 항목은 "해당없음"으로 닫고 개인정보 처리방침·약관 점검만 한다.
- 결과를 5절 형식으로 기록한다.

## 3단계: 위치 데이터 지도

`references/code-audit.md`를 따른다. 확장자를 좁히지 않는다.
- 권한 선언 → 수집 지점 → 전송(내부 API, 외부 지도·지오코딩 API) → 저장·로그·분석 → 권리 기능(철회, 일시중지, 탈퇴).
- 각 줄에 파일:줄. 선언과 실제 요청의 차이, 정밀도, 반복 전송을 기록한다.
- 이 지도가 이후 모든 대조의 기준이다. 문서 점검보다 먼저 끝낸다.

## 4단계: 문서 점검

- 위치기반서비스 이용약관: `references/terms-checklist.md` (T1~T22)
- 개인정보 처리방침과 개인위치정보 항목: `references/privacy-policy-checklist.md` (P1~P20, L1~L7)
- 각 항목: 통과 / 미흡 / 누락 / 해당없음 / 확인 불가 + 근거(원문 인용, URL, 파일:줄).
- 문서끼리의 충돌(가입 화면과 동의서의 필수·선택 표기 등)도 발견사항이다.

## 5단계: 동의·권리·파기

`references/consent-and-rights.md` (C1~C3, R1~R4, N1, K1).
- UI와 서버를 따로 판정한다.
- OS 권한 허용은 위치정보 동의가 아니고, OS 권한 해제는 동의 철회가 아니다.
- 공유 기능이 있으면 사후통보(N1), 가입 연령 제한이 없으면 14세 미만(K1)을 반드시 본다.

## 6단계: 보호조치와 기록

`references/safeguards.md` (S1~S5).
- 확인자료 자동 기록·보존(S1)은 형사처벌 조항이다. 서버 코드에서 기록 지점을 찾고, 없거나 볼 수 없으면 그대로 적는다.
- 관리 체계 문서(S4)는 코드로 확인할 수 없다. prep 모드에서만 증빙 목록으로 낸다.

## 7단계: 보고서

`references/report-template.md` 형식. 심각도는 `references/penalties.md`.
- 요약(적용 판정, 심각도별 건수, 먼저 할 일 3가지) → 위치 데이터 지도 → 발견사항 표 → 확인 불가 목록 → 부록.
- 저장 위치는 한 가지 규칙: 대상이 사용자의 작업 저장소(현재 작업 트리)면 `docs/compliance/location-privacy-YYYY-MM-DD.md`, 남의 저장소나 읽기 전용으로 받은 대상이면 저장소 밖(임시 폴더나 현재 디렉터리)에 쓴다. 사용자가 원치 않으면 대화로만.
- 운영 주체에 따라 의무가 갈리는 발견사항(셀프호스팅, "출시한다고 가정")은 심각도 뒤에 `조건부(전제: …)`를 붙이고 점수는 매긴다. 전제가 확정되지 않으면 해당없음으로 닫지 않는다.

## 8단계: 수정 (fix 모드)

`references/fix-patterns.md`.
1. 치명 → 높음 → 보통 순서. 같은 심각도에서는 코드 먼저, 문서 나중.
2. 코드: 동의 집행(F1), 철회·일시중지(F2), 확인자료 기록(F3), 최소 수집(F4), 파기(F5), 사후통보(F6), 연령 분기(F7). 프로젝트의 기존 패턴과 언어를 따른다.
3. 문서: 약관 골격(D1), 처리방침 위치 절(D2). 값은 위치 데이터 지도에서만 가져오고, 모르는 값은 `{{자리표시자}}`.
4. 프로젝트의 타입체크·린트·테스트를 돌린다. 실패하면 고치고, 고칠 수 없으면 보고한다.
5. 3단계~6단계를 수정된 코드로 다시 돌려 전후 비교표를 보고서에 붙인다.

## 출력

사용자에게 보여주는 것:
- 적용 판정 한 단락
- 심각도별 건수와 먼저 할 일 3가지
- 보고서 파일 경로, fix 모드면 바뀐 파일 목록과 검증 결과
- 확인 불가 항목과 누가 무엇을 보면 되는지
- 고른 기본값

절대 하지 않는 것:
- 확인하지 않은 항목을 "통과", "준수"로 쓰기
- 회사명, 책임자 이름, 연락처를 추측해 채우기
- 웹에 게시된 약관·처리방침 직접 수정, 배포, 신고서 제출, 운영 DB 변경
- 법령 금액·시행일을 출처 없이 단정하기
- 홍보 문구나 무관한 외부 링크 출력

규칙이 서로 부딪힐 때의 판정: `references/conflicts.md`.
