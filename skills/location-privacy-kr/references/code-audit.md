# 코드 점검: 위치 데이터 지도 만들기

목표는 "위치가 어디서 생겨서 어디로 가고, 어디에 남고, 언제 지워지는가"를 파일:줄 근거와 함께 한 장으로 그리는 것이다. 문서 점검은 이 지도와 대조해야 의미가 있다.

언어를 가리지 않는다. 확장자를 좁히지 말고 저장소 전체를 검색한다 (Objective-C `.m`, Swift, Kotlin, Java, Dart, TS/JS, ReScript, Python, Go 등). 플랫폼별 저장소가 따로 있으면(iOS 저장소가 별도 등) 최소한 1~2절(권한·매니페스트·수집 지점)은 형제 저장소에서도 돌린다.

## 검색 공통 규칙

- `git grep` 옵션은 패턴 **앞에** 둔다: `git grep -nIiE '<패턴>'`. 패턴 뒤에 `-i`를 붙이면 git grep이 실패하고, 출력을 파이프로 넘기면 "0건"처럼 보인다. 명령이 실패하면 0건으로 해석하지 말고 고쳐서 다시 돌린다.
- 단어 경계는 `\b` 대신 `-w`를 쓴다. macOS git의 `-E`에서 `\b`는 아무것도 맞추지 못한다.
- 두 번째 필터는 `git grep`이 아니라 일반 `grep`으로 파이프한다.
- 테스트·번역·빌드 산출물은 뺀다. 아래 배열을 만들고 모든 명령 끝에 `"${X[@]}"`를 붙인다 (bash, zsh 모두 동작).

```bash
X=(-- . ':!*test*/**' ':!*Test*/**' ':!**/androidTest/**' ':!*.lproj/**' ':!**/values-*/**' ':!**/node_modules/**' ':!**/Pods/**' ':!*.lock' ':!*.md' ':!CHANGELOG*' ':!**/lint*.xml')
```

## 1. 권한 선언

```bash
git grep -nIE 'ACCESS_(FINE|COARSE|BACKGROUND|MEDIA)_LOCATION|ACCESS_WIFI_STATE|NEARBY_WIFI_DEVICES|BLUETOOTH_SCAN' -- '*AndroidManifest.xml'
git grep -nIE 'NSLocation(WhenInUse|Always|AlwaysAndWhenInUse)UsageDescription|NSLocationTemporaryUsageDescriptionDictionary|UIBackgroundModes|NSAllowsArbitraryLoads' -- '*.plist'
git grep -nIiE 'location' -- '*PrivacyInfo.xcprivacy'
git grep -nIE 'cleartextTrafficPermitted' -- '*.xml'
```

판정할 것:
- **선언과 요청의 불일치**: Info.plist에 Always 문구가 있는데 코드는 WhenInUse만 요청하면 과잉 선언. 반대로 코드가 Always를 요청하는데 약관에 백그라운드 수집이 없으면 문서 누락.
- **정밀도**: 기능이 시·군·구 단위면 FINE이 필요한지 묻는다. COARSE나 iOS 대략적 위치로 충분하면 개선 권고.
- `ACCESS_MEDIA_LOCATION`은 사진 EXIF 위치 접근이다. 쓰는 곳이 있으면 별도 위치 사용 지점으로 등록한다.
- 백그라운드 위치(`ACCESS_BACKGROUND_LOCATION`, `UIBackgroundModes`의 `location`, `allowsBackgroundLocationUpdates`)는 약관·처리방침에 명시되어야 한다.
- 평문 전송 허용(`cleartextTrafficPermitted="true"`, `NSAllowsArbitraryLoads`)은 S3(전송 구간 보호) 근거로 기록한다.

## 2. 수집 지점

```bash
git grep -nIE 'getCurrentPosition|watchPosition|requestLocationUpdates|getLastLocation|FusedLocationProvider|CLLocationManager|startUpdatingLocation|startMonitoringSignificantLocationChanges|startMonitoringVisits|requestAlwaysAuthorization|requestWhenInUseAuthorization|GeofencingClient|startMonitoringForRegion|expo-location|react-native-geolocation|geolocation-service|background-geolocation|geolocator|navigator\.geolocation' "${X[@]}"
git grep -nIE 'WifiManager|getScanResults|getConnectionInfo|SSID|BSSID|BluetoothLeScanner|CLBeaconRegion|iBeacon|Eddystone|getAllCellInfo|CellInfo' "${X[@]}"
git grep -nIiE 'ExifInterface|TAG_GPS|kCGImagePropertyGPS|exif.*gps' "${X[@]}"
```

수집 지점마다 기록: 파일:줄, 호출 시점(앱 시작, 화면 진입, 버튼, 백그라운드), 1회/연속, 정밀도 옵션, 동의·권한 확인이 앞에 있는지.

Wi-Fi SSID·BSSID, 기지국 정보는 그 자체로 측위 결과가 아니어도 위치를 추정할 수 있다. 좌표와 함께 서버로 가면 같은 전송 지점에 묶어 기록하고, 단독이면 "판단 필요"로 둔다.

### 2b. 수집을 일으키는 다른 경로

사용자 조작 없이 위치를 수집·전송하게 만드는 경로를 찾는다.

```bash
git grep -nIiE 'reportLocation|requestLocation|remote.?command|push.*location|location.*push|BroadcastReceiver|WorkManager|BGTaskScheduler|setExact|AlarmManager' "${X[@]}" | grep -iE 'locat|report|position'
```

- 서버가 보낸 명령(푸시, MQTT 메시지)으로 위치를 보고하는 기능
- 다른 앱의 인텐트·URL 스킴으로 위치 보고를 시작하는 기능
- 예약 작업·백그라운드 태스크

## 3. 전송 지점

좌표가 담긴 변수를 따라 네트워크 호출까지 간다.

```bash
git grep -nIiwE '(lat|lng|lon|latitude|longitude|coords?|coordinates?|geohash|geopoint)' "${X[@]}"
```

- GraphQL: 스키마와 쿼리에서 `latitude`, `longitude`, `coordinates`를 인자로 받는 필드를 찾는다. 화면 포커스마다 다시 보내는지 본다.
- REST·MQTT·WebSocket: 요청 바디·쿼리스트링·메시지 페이로드에 좌표가 들어가는지. 페이로드 모델(직렬화 클래스)을 열어 같이 실리는 필드(기기 ID, 사용자명, Wi-Fi 정보, 배터리)를 기록한다.
- 외부 API: 지도·지오코딩·날씨·주소 변환 API로 좌표를 직접 보내는지. 직접 보내면 처리방침의 위탁·제3자·국외이전 항목과 대조한다.
- 정밀도: 보내기 전에 반올림·격자화를 하는지. 원래 정밀도 그대로면 기록한다.

### 3b. 설정으로 켜지는 전송

설정 가져오기(설정 파일, 원격 설정, QR, 딥링크)로 외부 전송 옵션이 켜질 때 앱 안의 동의 화면을 건너뛰는지 확인한다.

```bash
git grep -nIiE 'import.*(config|setting|preference)|remote.?config|loadConfiguration|deeplink|otrc' "${X[@]}"
```

## 4. 저장과 로그

클라이언트 저장소:
```bash
git grep -nIE 'AsyncStorage|MMKV|SharedPreferences|DataStore|UserDefaults|CoreData|Realm|SQLite|Room|localStorage|IndexedDB' "${X[@]}"
```
좌표나 위치 파생값(주소, 지역 코드)이 디스크에 남는지 본다. 메모리 상태만이면 그렇게 적는다.

분석·크래시·로깅:
```bash
git grep -nIE 'logEvent|track\(|amplitude|mixpanel|FirebaseAnalytics|Sentry|Crashlytics|datadog|console\.log|Log\.(d|i|w|e)|Timber|NSLog|os_log' "${X[@]}" | grep -iE 'lat|lng|coord|locat'
```
위치 변수를 쓰는 파일 안의 로그·분석 호출은 직접 읽는다. 분석 SDK의 자동 위치 수집 옵션도 설정 파일에서 확인한다. 화면 이동 이벤트의 route params에 좌표가 실리는지도 본다. `disableAnalytics` 같은 테스트 러너 인자는 SDK가 아니다.

서버(접근 가능할 때):
- 좌표 컬럼이 있는 테이블, 보존 기간, 삭제 배치
- 요청 로그·액세스 로그에 좌표가 평문으로 남는지
- 확인자료 테이블이 있는지 (`safeguards.md` S1)

## 5. 권리 기능

```bash
git grep -nIiE 'withdraw|revoke|unagree|disagree|opt.?out|delete.?account|deleteAccount|탈퇴|철회|일시.?중지' "${X[@]}"
git grep -nIiwE '(pause|paused|resume)' "${X[@]}" | grep -iE 'locat|track|report|share'
```
- 위치 동의 철회 API·화면이 있는지, 동의 API만 있는지
- 일시중지 토글이 있는지, 서버가 그 상태를 존중하는지
- 회원 탈퇴가 서버의 위치 데이터와 확인자료 처리까지 이어지는지 (클라이언트에서 로컬 저장소만 지우는 것은 부족)

## 6. 결과물: 위치 데이터 지도

```text
위치 데이터 지도
수집   src/location/useLocation.ts:48  getCurrentPosition, 1회, 기본 정밀도, 화면 진입 시
       → 앞단 확인: 약관 동의(클라이언트) → OS 권한 WhenInUse
트리거 없음 / 서버 명령으로 위치 보고 (src/push/handler.ts:77)
전송   src/nearby/NearbyList.tsx:120  GET /nearby?lat=&lng= 원 정밀도, 화면 포커스마다
외부   없음 (서버 경유) / 지오코딩 API 직접 호출 (src/geo/geocoder.ts:55)
저장   클라이언트: 메모리 상태만 / 서버: 확인 불가
로그   분석 이벤트에 좌표 없음 (src/analytics.ts 검토), 크래시 리포트 PII 비활성
권리   철회: 없음 / 일시중지: 없음 / 탈퇴: 클라이언트 저장소만 삭제, 서버 확인 불가
선언   Info.plist Always 문구 있음, 코드 요청 없음 → 과잉 선언
```

예시 줄은 형식 설명용 자리표시자다. 실제 보고서에는 실제 파일:줄만 쓴다.

## 확인 불가 처리

모바일 저장소만 받으면 서버 저장·보존·확인자료는 볼 수 없다. 이것을 "문제 없음"이나 "위반"으로 판정하지 말고 "확인 불가(서버 저장소 필요)"로 따로 모아 보고서에 적는다. 한 번 더 확인할 사람이 무엇을 보면 되는지(테이블 이름 후보, API 이름)를 같이 적는다.
