# KOKU Airline Renewal - UI Design

## 1. 문서 목적

본 문서는 KOKU Airline Renewal의 Frontend UI 구조와 주요 사용자 흐름을 정의합니다.

본 문서의 목적은 다음과 같습니다.

- MVP에서 필요한 화면 범위를 정의합니다.
- MVP에서는 일반 사용자용 주요 화면을 Desktop Web과 Mobile Web에서 지원하며,
  Responsive Web을 기본으로 설계합니다.
- Admin 및 SuperAdmin 관리 화면은 MVP에서 Desktop Web 사용성을 우선합니다.
- Guest, Member, Admin, SuperAdmin별 접근 가능한 화면을 정의합니다.
- 주요 사용자 시나리오와 화면 이동 흐름을 정의합니다.
- 예약, 좌석, 결제 등 상태에 따라 UI가 어떻게 변경되는지 정의합니다.
- 한국어와 일본어 UI를 동일한 기능 범위로 제공하기 위한 기준을 정의합니다.
- Frontend 구현 과정에서 AI Agent가 화면 구조와 사용자 흐름을 임의로 변경하지 않도록 기준을 제공합니다.

비즈니스 규칙은 `02-domain-policy.md`를 기준으로 하며,
구체적인 Backend API와 데이터 구조는 `05-data-api-design.md`,
시스템 및 기술 구조는 `04-system-design.md`에서 정의합니다.

---

### 1.1 용어 표기 원칙

본 문서는 `01-project-plan.md`와 `02-domain-policy.md`의 용어 표기 원칙을 따릅니다.

설명과 일반적인 기능명은 한국어로 작성하며,
Role, Entity, 상태 및 코드와 직접 연결되는 개념은 영문 Canonical Term을 사용합니다.

주요 Role:

- `Guest`
- `Member`
- `Admin`
- `SuperAdmin`

주요 Entity:

- `Member`
- `AuthAccount`
- `Airport`
- `Route`
- `Aircraft`
- `Flight`
- `Seat`
- `Passenger`
- `Reservation`
- `Payment`

---

## 2. UI 설계 기본 원칙

### 2.1 사용자 흐름 우선

MVP에서는 화려한 시각적 효과보다
사용자가 현재 어떤 단계에 있으며 다음에 무엇을 해야 하는지 명확하게 이해할 수 있도록 설계합니다.

특히 다음 흐름을 명확하게 표현합니다.

- 항공편 검색
- 항공편 선택
- 탑승객 정보 입력
- 좌석 선택
- 예약 내용 확인
- 결제
- 예약 확정

---

### 2.2 내부 항공편과 외부 실제 항공편 구분

KOKU Airline 내부 Flight와 외부 Flight API를 통해 조회한 실제 항공편은
사용자가 혼동하지 않도록 UI에서 명확하게 구분합니다.

#### 내부 KOKU Airline Flight

- KOKU Airline 브랜드 표시
- 좌석 조회 가능
- 예약 가능
- Mock 결제 가능

#### 외부 실제 항공편

- `실제 항공편 조회`임을 명시
- 실제 항공사 정보 표시
- 조회 및 비교만 가능
- KOKU Airline 예약 버튼을 제공하지 않음
- 실제 발권 또는 외부 항공사 예약 기능을 제공하지 않음

---

### 2.3 상태를 명확하게 표시

Reservation, Payment, Seat 등의 상태는 사용자가 이해할 수 있는 자연어로 표시합니다.

예:

| 대상 | 내부 상태 | 한국어 표시 | 일본어 표시 |
|---|---|---|---|
| Reservation | `PENDING` | 예약 진행 중 | 予約手続き中 |
| Reservation | `CONFIRMED` | 예약 확정 | 予約確定 |
| Reservation | `CANCELLED` | 예약 취소 | 予約キャンセル |
| Payment | `PENDING` | 결제 진행 중 | 支払い処理中 |
| Payment | `SUCCESS` | 결제 완료 | 支払い完了 |
| Payment | `FAILED` | 결제 실패 | 支払い失敗 |
| Payment | `CANCELLED` | 결제 취소 | 支払いキャンセル |
| Payment | `REFUNDED` | 환불 완료 | 払い戻し完了 |
| Seat | `AVAILABLE` | 선택 가능 | 選択可能 |
| Seat | `HELD` | 임시 확보 | 一時確保 |
| Seat | `RESERVED` | 예약됨 | 予約済み |
| Seat | `UNAVAILABLE` | 선택 불가 | 選択不可 |

Frontend에서는 Backend 상태 값을 직접 사용자에게 노출하지 않고
Locale에 맞는 사용자용 문자열로 변환하여 표시합니다.

### 2.4 Responsive UI

Guest와 Member가 사용하는 주요 Customer UI는
Desktop과 Mobile에서 동일한 기능 및 사용자 흐름을 제공합니다.

Responsive Layout은 화면 크기에 따라
정보를 삭제하거나 기능을 축소하는 방식이 아니라,
동일한 정보를 적절하게 재배치하는 방식으로 구성합니다.

기본 디자인 기준 Width는 다음과 같습니다.

- Desktop: `1440px`
- Mobile: `390px`

구체적인 CSS Breakpoint는 본 문서에서 확정하지 않으며,
Frontend 구현 단계에서 결정합니다.

Responsive 전환 시에도 다음 요소를 유지합니다.

- 현재 Page 및 Navigation 상태
- 편도 / 왕복 검색 조건
- Flight 선택 상태
- Passenger 정보
- Seat 선택 상태
- Reservation Step
- Hold Countdown
- Payment 상태
- Locale
- 인증 상태

Mobile 화면이라는 이유로
Desktop에서 제공하는 주요 Customer 기능을 제거하지 않습니다.

---

## 3. 다국어 UI 정책

Date Picker 등 외부 UI Component의 월, 요일 및 날짜 관련 표시도
현재 Locale(`ko`, `ja`)에 맞게 제공하는 것을 원칙으로 합니다.

### 3.1 지원 언어

MVP에서는 다음 두 언어를 필수 지원합니다.

- 한국어 (`ko`)
- 일본어 (`ja`)

두 언어 중 하나를 보조 언어로 취급하지 않으며,
주요 사용자 기능은 두 언어에서 동일하게 제공해야 합니다.

---

### 3.2 언어 선택

Global Header에서 사용자가 현재 언어를 확인하고 변경할 수 있도록 합니다.

예:

`한국어 | 日本語`

언어 변경 시 현재 인증 상태와 서비스 상태는 유지합니다.

가능한 경우 현재 Page를 유지한 상태에서 표시 문자열만 변경합니다.

---

### 3.3 초기 Locale 선택

초기 Locale은 다음 우선순위로 결정하는 것을 기본안으로 합니다.

1. 사용자가 이전에 직접 선택한 Locale
2. Browser Locale
3. 기본 Locale `ko`

사용자가 직접 선택한 Locale은 Frontend에서 저장하여
다음 방문에서도 유지할 수 있도록 합니다.

구체적인 저장 방식은 `04-system-design.md`에서 확정합니다.

---

### 3.4 문자열 관리

사용자에게 표시되는 문자열은 Component에 직접 Hard Coding하지 않습니다.

예:

```text
src/
 └─ locales/
     ├─ ko/
     └─ ja/
```

구체적인 i18n Library 및 디렉터리 구조는 Frontend 초기 구성에서 확정합니다.

---

## 4. 전체 Information Architecture

MVP 사용자 화면은 크게 다음 영역으로 구분합니다.

```text
KOKU Airline
│
├─ Home
│
├─ KOKU 항공편 검색
│   ├─ 검색 결과
│   └─ Flight 상세
│
├─ 실제 항공편 검색
│   └─ 실제 항공편 검색 결과
│
├─ AI 항공편 검색
│
├─ 인증
│   ├─ 로그인
│   ├─ 회원가입
│   └─ Google OAuth
│
├─ 예약
│   ├─ Passenger 정보
│   ├─ Seat 선택
│   ├─ 예약 확인
│   ├─ Mock 결제
│   └─ 예약 완료
│
├─ My Page
│   ├─ 회원 정보
│   ├─ 비밀번호 변경
│   ├─ 예약 목록
│   ├─ 예약 상세
│   └─ 회원 탈퇴
│
└─ Admin
    ├─ Dashboard
    ├─ Airport
    ├─ Route
    ├─ Aircraft / Seat 구성
    ├─ Flight / 운항 일정
    └─ Reservation
```

---

## 5. Global Navigation

현재 Page에 해당하는 Navigation은 Primary Color underline 등으로
Active State를 명확하게 표시합니다.

### 5.1 Guest Header

```text
[KOKU Airline]

항공편 검색
실제 항공편
AI 항공편 검색

                    [한국어 / 日本語]
                    [로그인]
                    [회원가입]
```

Guest가 AI 항공편 검색을 선택하면 로그인 필요 안내를 제공합니다.

---

### 5.2 Member Header

```text
[KOKU Airline]

항공편 검색
실제 항공편
AI 항공편 검색

                    [한국어 / 日本語]
                    [내 예약]
                    [My Page]
                    [로그아웃]
```
---

### 5.3 Customer Mobile Navigation

Guest와 Member의 Mobile 화면에서는
Desktop Header의 모든 항목을 한 줄에 유지할 필요가 없습니다.

Mobile Header의 기본 구조는 다음 방향으로 합니다.

```text
[KOKU Airline]                 [KO | JA] [Menu]
```

Menu를 열면 현재 Role에 따라 주요 Navigation과 Account Action을 제공합니다.

#### Guest

```text
항공편 검색
실제 항공편
AI 항공편 검색
로그인
회원가입
```

#### Member

```text
항공편 검색
실제 항공편
AI 항공편 검색
내 예약
My Page
로그아웃
```

현재 Page는 Desktop과 동일하게 Active State를 명확하게 표시합니다.

Mobile Menu를 사용하더라도
Navigation 기능이나 Role별 접근 범위를 변경하지 않습니다.

언어 전환은 Mobile에서도 쉽게 접근할 수 있어야 합니다.

---

### 5.4 Admin / SuperAdmin Header

일반 사용자 화면과 관리자 화면은 Navigation을 분리합니다.

```text
[관리자]

Dashboard
Airport
Route
Aircraft
Flight
Reservation

                    [일반 서비스]
                    [한국어 / 日本語]
                    [로그아웃]
```

SuperAdmin에게만 허용된 기능은 권한에 따라 추가 Action을 표시합니다.

### 5.5 Brand Logo

KOKU Airline의 Brand Symbol은 깃발 형태 안에
Wing 요소를 결합한 간결한 항공 Emblem 형태를 기본 방향으로 합니다.

- 깃발을 주요 Motif로 사용
- Wing 요소는 깃발 내부에 통합하여 표현
- 깃발과 Wing이 하나의 Symbol로 인식되도록 구성
- Skull, Bones 등 직접적인 해적 상징은 사용하지 않음
- 작은 Header 영역에서도 식별 가능한 단순한 형태
- KOKU Airline Wordmark와 함께 사용 가능
- Primary Blue 계열과 어울리는 Flat / Minimal 스타일

---

## 6. Role별 접근 화면

| 화면 | Guest | Member | Admin | SuperAdmin |
|---|---:|---:|---:|---:|
| Home | O | O | O | O |
| KOKU Flight 검색 | O | O | O | O |
| Flight 상세 조회 | O | O | O | O |
| 외부 실제 항공편 조회 | O | O | O | O |
| AI 항공편 검색 | X | O | - | - |
| Seat 선택 | X | O | - | - |
| 예약 생성 | X | O | - | - |
| Mock 결제 | X | O | - | - |
| 자신의 예약 조회 | X | O | - | - |
| 관리자 Dashboard | X | X | O | O |
| Master Data 조회 | X | X | O | O |
| Master Data 변경 | X | X | X | O |
| Flight 관리 | X | X | O | O |
| 예약 현황 조회 | X | X | O | O |
| 개별 Reservation 강제 취소 | X | X | X | O |

`Admin`과 `SuperAdmin`의 일반 사용자 서비스 이용 가능 여부는 인증 및 계정 설계에 따라 구현할 수 있으나,
관리자 업무 화면의 권한 기준은 위 표를 따릅니다.

---

## 7. 주요 사용자 흐름

### 7.1 Guest 항공편 조회

```text
Home
 ↓
검색 조건 입력
 ↓
KOKU Flight 검색 결과
 ↓
Flight 상세
```

편도 검색의 기본 흐름은 위와 같습니다.

왕복 검색에서는 출국편과 귀국편을 순서대로 선택합니다.

```text
Home
↓
왕복 검색 조건 입력
↓
출국 Flight 검색 결과
↓
출국 Flight 선택
↓
귀국 Flight 검색 결과
↓
귀국 Flight 선택
↓
왕복 여정 확인
```

Guest는 출국편과 귀국편을 조회하고 선택할 수 있지만
Reservation을 시작할 수 없습니다.

Guest가 왕복 여정에서 예약을 시작하면 로그인 후
선택한 출국 Flight와 귀국 Flight 정보를 모두 유지합니다.

편도 검색에서도 Guest는 Flight를 조회하고 선택할 수 있지만
Reservation을 시작할 수 없습니다.

Guest의 편도 예약 시작 흐름은 다음과 같습니다.

```text
Flight 검색 결과 또는 Flight 상세
 ↓
[항공편 선택]
 ↓
선택한 Flight Summary
 ↓
[로그인 후 예약]
 ↓
로그인
 ↓
인증 성공
 ↓
선택한 Flight 정보 유지
 ↓
Passenger 정보 입력
```

Flight 선택 Action과 예약 절차 시작 Action은 구분합니다.

Guest는 Flight를 선택한 이후에만
`[로그인 후 예약]` Action을 사용할 수 있습니다.

로그인 성공 후에는 선택한 Flight를 다시 검색하거나 선택하지 않고
Passenger 정보 입력 단계로 이동합니다.

편도에서는 로그인 전 사용자가 선택한 Flight 정보를 유지하고,
왕복에서는 선택한 출국 Flight와 귀국 Flight 정보를 모두 유지합니다.

---

### 7.2 Member 예약 흐름

#### 편도 예약

```text
Home
↓
Flight 검색
↓
검색 결과
↓
Flight 검색 결과 또는 Flight 상세
↓
[항공편 선택]
↓
선택한 Flight Summary
↓
[예약하기]
↓
Passenger 정보 입력
↓
Seat 선택
↓
PENDING Reservation 생성
선택한 모든 Seat AVAILABLE → HELD
↓
예약 내용 확인
↓
Mock 결제
↓
결제 성공
↓
Reservation CONFIRMED
선택한 모든 Seat HELD → RESERVED
↓
예약 완료
↓
Reservation 상세
```

Flight 선택과 Reservation 시작은 별도의 Action으로 구분합니다.

Member는 편도 Flight를 선택한 이후
`[예약하기]`를 선택하여 Passenger 정보 입력 단계로 이동합니다.

#### 왕복 예약

```text
Home
↓
왕복 검색
↓
출국 Flight 검색 결과
↓
출국 Flight 선택
↓
귀국 Flight 검색 결과
↓
귀국 Flight 선택
↓
선택한 왕복 여정 Summary
↓
[예약하기]
↓
Passenger 정보 입력
↓
출국 Flight Seat 선택
↓
귀국 Flight Seat 선택
↓
왕복 Reservation 생성
출국 / 귀국 선택 Seat → HELD
↓
예약 내용 확인
↓
Mock 결제
↓
결제 성공
↓
왕복 Reservation CONFIRMED
출국 / 귀국 선택 Seat → RESERVED
↓
예약 완료
↓
Reservation 상세
```

왕복 Reservation에서도 Flight 선택 완료와 Reservation 시작을 구분합니다.

출국 Flight와 귀국 Flight를 모두 선택한 이후
선택한 왕복 여정 Summary를 표시하고,
Member는 `[예약하기]` Action을 통해 Passenger 정보 입력 단계로 이동합니다.

Guest에게는 같은 위치에 `[로그인 후 예약]` Action을 제공합니다.

왕복 예약에서는 동일한 Passenger 구성을
출국 Flight와 귀국 Flight에 공통으로 적용합니다.

Seat는 Flight별로 독립적으로 선택합니다.

따라서 왕복 예약의 Seat 선택 단계에서는
출국 Flight Seat 선택과 귀국 Flight Seat 선택을
명확하게 구분하여 표시합니다.

---

### 7.3 외부 실제 항공편 조회

```text
Home
 ↓
실제 항공편 검색
 ↓
검색 조건 입력
 ↓
외부 Flight API 조회
 ↓
실제 항공편 검색 결과
```

검색 결과에서는 가능한 경우 다음 정보를 표시합니다.

- 실제 항공사
- 편명
- 출발 / 도착 Airport
- 출발 / 도착 시각
- 가격 정보
- 경유 여부
- 외부 실제 항공편 정보라는 안내

외부 실제 항공편에는 KOKU Airline의 `예약하기` 버튼을 제공하지 않습니다.

---

### 7.4 AI 항공편 검색

```text
Member 로그인
 ↓
AI 항공편 검색
 ↓
자연어 조건 입력
 ↓
AI 검색 조건 구조화
 ↓
Application 입력 조건 검증
 ↓
외부 Flight API 조회
 ↓
Application 필터링 / 정렬
 ↓
AI 추천 결과 및 설명
```

AI 화면에는 다음과 같은 안내를 제공합니다.

> AI 검색 결과는 실제 항공편 데이터를 기반으로 제공되며,
> KOKU Airline에서 해당 항공편을 직접 예약할 수 없습니다.

AI가 생성한 추천 설명과 외부 Flight API가 제공한 실제 항공편 데이터는
사용자가 구분할 수 있도록 UI 영역을 분리합니다.

---

## 8. 인증 UI

### 8.1 로그인 화면

#### 입력 항목

- Email
- Password

#### Action

- 로그인
- Google로 로그인
- 회원가입으로 이동

#### Validation

- 필수값 확인
- Email 형식 확인

인증 실패 시 계정 존재 여부 등 불필요한 상세 정보를 노출하지 않습니다.

예:

```text
이메일 또는 비밀번호를 확인해 주세요.
```

---

### 8.2 회원가입 화면

#### 입력 항목

- Email
- Password
- Password 확인

#### Password 조건

다음 조건을 화면에서 안내합니다.

- 8자 이상
- 영문 대문자 최소 1자
- 영문 소문자 최소 1자
- 숫자 최소 1자
- 특수문자 최소 1자
- 허용 특수문자: `! @ # $ % ^ & *`

가능하면 사용자가 Password를 입력하는 동안 각 조건의 충족 여부를 표시합니다.

```text
✓ 8자 이상
✓ 영문 대문자
✓ 영문 소문자
✕ 숫자
✓ 특수문자(!, @, #, $, %, ^, &, * 만 가능)
```

---

### 8.3 Google OAuth 로그인

기본 흐름:

```text
Google 로그인
 ↓
Google OAuth 인증
 ↓
AuthAccount 확인
```

#### 신규 사용자

```text
GOOGLE AuthAccount 없음
+
동일 Email Member 없음
 ↓
Member 생성
 ↓
GOOGLE AuthAccount 연결
 ↓
로그인 완료
```

#### 기존 GOOGLE 사용자

```text
GOOGLE AuthAccount 확인
 ↓
연결된 Member 상태 확인
 ↓
ACTIVE → 로그인 완료
WITHDRAWN → 로그인 거부
```

---

### 8.4 동일 Email의 LOCAL 계정이 존재하는 경우

Google OAuth에서 확인된 Email과 기존 `LOCAL` Member의 Email이 동일하더라도
Email 일치만으로 계정을 자동 연결하지 않습니다.

```text
Google OAuth 성공
 ↓
동일 Email LOCAL 계정 발견
 ↓
기존 계정 확인 안내
 ↓
LOCAL Password 재입력
 ↓
재인증 성공
 ↓
GOOGLE AuthAccount 연결
 ↓
로그인 완료
```

안내 예:

> 동일한 이메일로 가입된 KOKU Airline 계정이 있습니다.
> 계정 보호를 위해 기존 비밀번호를 확인해 주세요.

재인증에 실패하면 계정을 연결하지 않습니다.

`WITHDRAWN` 상태의 Member는 자동으로 재활성화하거나
새로운 AuthAccount를 연결하지 않습니다.

---

## 9. Home

Home은 서비스의 주요 Entry Point로 사용합니다.

Home Hero의 기본 보조 문구는 다음과 같이 합니다.

`가상 항공편 예약과 실제 항공편 조회를 한 번에`

### 9.1 KOKU Flight 검색

주요 입력 항목:

- 여행 유형
  - 편도
  - 왕복
- 출발지
- 도착지
- 출발일
- 도착일 (왕복 선택 시)

[항공편 검색]

기본 여행 유형은 `편도`로 합니다.

`편도`를 선택한 경우 도착일을 입력하지 않습니다.

`왕복`을 선택한 경우 출발일과 도착일을 모두 입력하며,
도착일은 출발일보다 이후여야 합니다.

MVP에서는 한국 ↔ 일본 노선만 입력할 수 있습니다.

왕복 검색에서는 출국 Flight와 귀국 Flight를 각각 선택합니다.

귀국 Flight의 출발 Airport와 도착 Airport는
출국 Flight의 출발 Airport와 도착 Airport의 역방향이어야 합니다.

예:

```text
출국: ICN → NRT
귀국: NRT → ICN
```

---

### 9.2 주요 서비스 안내

Home에서 내부 KOKU Airline 서비스와 외부 조회 서비스를
사용자가 쉽게 구분할 수 있도록 합니다.

```text
[KOKU Airline 항공편]

KOKU Airline의 가상 항공편입니다.
좌석을 선택하고 Mock 결제를 통해 예약할 수 있습니다.


[실제 항공편 조회]

외부 데이터를 이용하여 실제 한국 ↔ 일본 항공편을 검색합니다.
조회만 가능하며 KOKU Airline에서 예약할 수 없습니다.


[AI 항공편 추천 받기]

자연어로 원하는 조건을 입력하여
실제 항공편을 검색하고 추천받을 수 있습니다.
```

AI 기능은 Guest에게 노출할 수 있지만,
실제 사용 시 로그인을 요구합니다.

---

## 10. KOKU Flight 검색

### 10.1 검색 조건

- 여행 유형 (`ONE_WAY`, `ROUND_TRIP`)
- 출발 Airport
- 도착 Airport
- 출발 Date
- 도착 Date (`ROUND_TRIP`인 경우)

출발 Airport와 도착 Airport는 동일할 수 없습니다.

다음 조합만 허용합니다.

- 한국 → 일본
- 일본 → 한국

`ONE_WAY`에서는 도착 Date를 사용하지 않습니다.

`ROUND_TRIP`에서는 도착 Date가 필수이며,
도착 Date는 출발 Date보다 이후여야 합니다.

왕복 검색에서는 귀국 Route가 출국 Route의 역방향이어야 합니다.

예:

- 출국: ICN → NRT
- 귀국: NRT → ICN

왕복 검색 결과에서는 먼저 출국 Flight를 선택한 후
해당 Route의 역방향 귀국 Flight를 선택합니다.

---

### 10.2 Flight 검색 결과

각 Flight는 Card 형태로 표시할 수 있습니다.

```text
--------------------------------
KOKU Airline

KO101

ICN                 NRT
09:30      →        11:50

2026.09.10

예약 가능

                [상세보기]
--------------------------------
```

출발 예정 시각까지 2시간 미만이 남은 Flight는
새로운 Reservation을 생성할 수 없습니다.

검색 결과에서는 다음과 같이 표시합니다.

```text
예약 마감
```

상세 조회는 가능하지만 예약 Action은 비활성화합니다.

#### 편도 Flight 선택

`ONE_WAY` 검색 결과에서는 예약 가능한 Flight에 대해
다음 Action을 제공할 수 있습니다.

```text
[상세보기]
[항공편 선택]
```

`[항공편 선택]`은 Reservation을 즉시 시작하는 Action이 아닙니다.

Flight를 선택하면 선택한 Flight Summary를 표시합니다.

예:

```text
선택한 항공편

KO103
ICN → NGO
2026-08-26
09:30 → 11:50
```

Flight 선택 완료 후 인증 상태에 따라
예약 절차 시작 Action을 제공합니다.

Guest:

```text
[로그인 후 예약]
```

Member:

```text
[예약하기]
```

예약 불가 Flight에는 [항공편 선택] Action을 제공하지 않거나
Disabled 처리합니다.

#### 왕복 검색 결과

왕복 검색에서는 출국 Flight와 귀국 Flight 선택 단계를 구분합니다.

```text
1. 출국편 선택
ICN → NRT
2026-09-10

[Flight 목록]

↓ 출국편 선택

2. 귀국편 선택
NRT → ICN
2026-09-15

[Flight 목록]
```

출국 Flight를 선택하기 전에는
귀국 Flight 선택 단계로 진행하지 않습니다.

귀국 Flight 선택 화면에서는
현재 선택한 출국 Flight의 요약 정보를 함께 표시합니다.

귀국 Flight까지 선택하면
출국 Flight와 귀국 Flight를 함께 포함한 왕복 여정 Summary를 표시합니다.

Flight 선택 완료 후 인증 상태에 따라
예약 절차 시작 Action을 제공합니다.

Guest:

```text
[로그인 후 예약]
```

Member:

```text
[예약하기]
```

출국 Flight 또는 귀국 Flight 중 하나라도 선택되지 않은 상태에서는
예약 절차 시작 Action을 활성화하지 않습니다.

---

## 11. Flight 상세

표시 정보:

- 편명
- 출발 Airport
- 도착 Airport
- 출발 Date / Time
- 도착 Date / Time
- Aircraft
- 운항 상태 (`SCHEDULED`, `CANCELLED`, `DEPARTED`) 
- 예약 가능 여부

#### ONE_WAY Flight 선택

예약 가능한 편도 Flight 상세에서는 인증 상태와 관계없이
먼저 Flight 선택 Action을 제공합니다.

```text
[항공편 선택]
```

Flight 선택 이후 예약 절차 시작 Action은
선택된 Flight Summary 영역에서 인증 상태에 따라 구분합니다.

Guest:

```text
[로그인 후 예약]
```

Member:

```text
[예약하기]
```

Flight 상세에서 Flight 선택 이전에
`[예약하기]` 또는 `[로그인 후 예약]`을 직접 예약 절차 시작 Action으로 제공하지 않습니다.

#### 예약 마감 Flight

```text
[예약 마감]
```

Flight 선택 Action을 Disabled 처리합니다.

#### 예약 불가 Flight 상태

다음 Flight 상태에서는 새로운 Reservation을 생성할 수 없습니다.

- `CANCELLED`
- `DEPARTED`

해당 상태에서는 `[항공편 선택]`, `[출국편 선택]`, `[귀국편 선택]` 등
Flight 선택 Action을 제공하지 않거나 Disabled 처리합니다.

`SCHEDULED` 상태인 경우에도
Flight 출발 예정 시각까지 2시간 미만이면
Flight 선택 Action을 Disabled 처리합니다.

#### 왕복 검색 중 Flight 선택

왕복 검색에서 Flight 상세에 진입한 경우
현재 선택 단계에 따라 Action Label을 구분할 수 있습니다.

출국 Flight:

```text
[출국편 선택]
```

귀국 Flight:

```text
[귀국편 선택]
```

출국 Flight를 이미 선택한 상태에서 귀국 Flight를 조회하는 경우
선택된 출국 Flight의 요약 정보를 함께 표시합니다.

---

## 12. 예약 Step UI

예약 과정에서는 사용자가 현재 어느 단계인지 명확하게 확인할 수 있도록 합니다.

```text
1. 탑승객 정보
 ↓
2. 좌석 선택
 ↓
3. 예약 확인
 ↓
4. 결제
 ↓
5. 완료
```

Desktop 예:

```text
[1 탑승객] ─ [2 좌석] ─ [3 확인] ─ [4 결제] ─ [5 완료]
```

Mobile에서는 좁은 화면에서 Step Label이 지나치게 압축되지 않도록
간결한 Step Indicator를 사용할 수 있습니다.

예:

```text
2 / 5  좌석 선택

● ─ ● ─ ○ ─ ○ ─ ○
```

또는 공간이 충분한 경우:

```text
탑승객 > 좌석 > 확인 > 결제 > 완료
```

Mobile에서도 전체 예약 단계 수와
현재 진행 단계를 사용자가 확인할 수 있어야 합니다.

왕복 Reservation의 Seat 단계에서는
추가로 현재 선택 중인 Flight를 표시합니다.

```text
2 / 5 좌석 선택

출국편
ICN → NRT

[좌석 선택 UI]
```

귀국 Seat 선택 시에는 같은 위치에서
`귀국편`으로 변경하여 표시합니다.

왕복 Reservation에서도 기본 Step은 동일하게 유지합니다.

```text
[1 탑승객] ─ [2 좌석] ─ [3 확인] ─ [4 결제] ─ [5 완료]
```

왕복의 Seat 선택 단계에서는 내부적으로 다음 순서를 제공합니다.

1. 출국 Flight Seat 선택
2. 귀국 Flight Seat 선택

사용자가 현재 어느 Flight의 Seat를 선택하고 있는지
명확하게 표시합니다.

---

## 13. Seat 선택

### 13.1 Seat 상태

최소 다음 상태를 시각적으로 구분합니다.

- 선택 가능
- 현재 사용자가 선택
- 다른 Reservation에서 `HELD`
- `RESERVED`
- `UNAVAILABLE`

상태는 색상만으로 구분하지 않고
Text, Icon 또는 Pattern 등을 함께 사용합니다.

---

### 13.2 Reservation 시작

MVP에서 Seat Hold 시간은 1시간입니다.

사용자가 Seat가 필요한 모든 Passenger의 좌석 선택을 완료한 후 예약을 시작하면
Backend에서 다음 처리를 수행합니다.

```text
Reservation → PENDING
선택한 모든 Seat → HELD
```

Backend 처리가 성공한 이후 Hold Countdown을 표시합니다.

```text
좌석이 임시 확보되었습니다.

남은 시간
00:59:42
```

Countdown은 사용자 안내 목적이며
실제 Hold 만료 여부는 Backend 시간을 기준으로 판단합니다.

---

### 13.3 Hold 만료

Hold가 만료된 경우 사용자에게 명확하게 안내합니다.

```text
좌석 임시 확보 시간이 만료되었습니다.

선택한 모든 좌석이 다시 예약 가능한 상태로 반환되었습니다.

[좌석 다시 선택]
```

만료된 Reservation에서는 기존 상태로 결제를 계속 진행할 수 없습니다.

Hold 만료 시 Backend에서는 다음 상태 전이를 처리합니다.

```text
Reservation
PENDING → CANCELLED

해당 Reservation의 모든 Seat
HELD → AVAILABLE

현재 처리 중인 PENDING Payment가 존재하는 경우
PENDING → CANCELLED
```

기존 `FAILED` Payment는 결제 이력으로 유지합니다.

Hold 만료 이후 해당 Reservation에서는
새로운 Payment를 생성할 수 없습니다.

### 13.4 왕복 Seat 선택

왕복 Reservation에서는 출국 Flight와 귀국 Flight의 Seat를
각각 선택해야 합니다.

기본 흐름:

```text
출국 Flight Seat 선택
↓
귀국 Flight Seat 선택
↓
전체 Seat 선택 확인
↓
Reservation 시작
```

화면에는 현재 선택 중인 Flight를 명확하게 표시합니다.

예:

```text
출국편 ICN → NRT

또는

귀국편 NRT → ICN
```

Passenger별 Seat 배정은 각 Flight마다 별도로 관리합니다.

예:

```text
출국편
KIM JIHUN → 12A

귀국편
KIM JIHUN → 8C
```

Passenger의 Seat 필요 여부는 각 Flight의 탑승일을 기준으로 계산된
연령 구분에 따라 판단합니다.

해당 Flight에서 `Infant`인 Passenger는 별도의 Seat를 사용하지 않으며,
해당 Flight에서 `Child` 또는 `Adult`인 Passenger는 Seat를 선택해야 합니다.

따라서 왕복 일정 중 Passenger의 연령 구분이 변경되는 경우
출국 Flight와 귀국 Flight의 Seat 필요 여부가 달라질 수 있습니다.

왕복 Reservation 시작 시에는 출국 Flight와 귀국 Flight에서
선택한 모든 Seat를 하나의 예약 시작 과정으로 처리합니다.

선택한 모든 Seat 확보에 성공한 경우에만
왕복 Reservation을 `PENDING` 상태로 시작합니다.

출국 또는 귀국 Flight의 Seat 중 하나라도 확보하지 못한 경우
왕복 Reservation 시작 전체를 실패 처리하며,
일부 Seat만 `HELD` 상태로 남기는 UI를 제공하지 않습니다.

---

## 14. Passenger 정보 입력

Reservation은 한 명 이상의 Passenger를 포함할 수 있습니다.

Passenger별로 다음 정보를 입력합니다.

### 14.1 ROUND_TRIP Flight Summary

`ROUND_TRIP` Passenger 정보 입력 화면에서는
현재 Reservation에 포함된 출국 Flight와 귀국 Flight의 요약 정보를 모두 표시합니다.

두 Flight는 하나의 `ROUND_TRIP` Reservation에 포함된 여정으로 표현하며,
귀국 Flight만 단독으로 표시하지 않습니다.

최소 다음 정보를 구분하여 표시합니다.

- 출국 Flight
  - Flight Number
  - 출발 / 도착 Airport
  - 출발 Date
  - 출발 / 도착 Time
- 귀국 Flight
  - Flight Number
  - 출발 / 도착 Airport
  - 출발 Date
  - 출발 / 도착 Time

Desktop에서는 두 Flight Summary를 나란히 또는 명확하게 구분된 영역으로 표현할 수 있으며,
Mobile에서는 출국 Flight → 귀국 Flight 순서의 1 Column Layout으로 배치할 수 있습니다.

Passenger 구성은 출국 Flight와 귀국 Flight에 공통으로 적용하며,
Flight별로 별도의 Passenger 입력 Form을 제공하지 않습니다.

#### 기본 정보

- 테스트용 영문 성 (여권 영문명 형식)
- 테스트용 영문 이름 (여권 영문명 형식)
- 생년월일
- 성별
- 국적

#### 테스트용 여권 정보

Passenger의 기본 정보를 입력하면 테스트용 여권 정보는 시스템에서 자동으로 생성합니다.

- 여권번호: 테스트용 번호 자동 생성
- 여권 발급국: 입력한 국적과 동일한 국가로 자동 설정
- 여권 만료일: 생성 시점 기준 5년 뒤 날짜로 자동 설정

자동 생성된 테스트용 여권 정보는 사용자가 직접 입력하거나 수정하지 않는 것을 기본으로 합니다.

화면에는 다음 안내를 표시합니다.

> 본 서비스는 포트폴리오용 가상 항공사 서비스입니다.
> 실제 탑승객의 개인정보나 실제 여권 정보를 입력하지 마세요.
> 테스트용 여권번호, 발급국, 만료일은 시스템에서 자동 생성됩니다.

테스트용 여권 만료일은 예약에 포함된 모든 Flight의 탑승일 이후인지 검증합니다.

편도에서는 출국 Flight의 탑승일을 기준으로 하며,
왕복에서는 출국 Flight와 귀국 Flight 모두의 탑승일 조건을 만족해야 합니다.

조건을 만족하지 않는 경우 예약을 계속 진행할 수 없으며
다음과 같이 안내합니다.

```text
테스트용 여권 유효기간이 Flight 출발일 조건을 만족하지 않습니다.

예약을 계속 진행할 수 없습니다.
```

---

### 14.2 Passenger 추가 및 삭제

하나의 Reservation에는 여러 Passenger가 포함될 수 있으므로
사용자는 Passenger 입력 화면에서 탑승객을 추가하거나 삭제할 수 있습니다.

예:

```text
Passenger 1
[탑승객 정보 입력]

Passenger 2
[탑승객 정보 입력]

[+ Passenger 추가]
```

Reservation 진행을 위해 최소 1명의 Passenger가 필요하므로
마지막 남은 Passenger는 삭제할 수 없습니다.

Infant와 연결된 Adult를 삭제하려는 경우
해당 Infant의 동반 Adult를 먼저 변경하거나 Infant를 삭제해야 합니다.

Passenger 정보 입력이 완료되면
입력된 생년월일과 Reservation에 포함된 각 Flight의 탑승일을 기준으로
Adult, Child, Infant를 판단합니다.

`ONE_WAY`에서는 출국 Flight를 기준으로 계산하고,
`ROUND_TRIP`에서는 출국 Flight와 귀국 Flight의 연령 구분을
각각 계산한 후 Seat 선택 단계로 이동합니다.

---

## 15. Adult / Child / Infant UI

Passenger의 연령 구분은 사용자가 직접 선택하지 않습니다.

왕복 Reservation에서는 각 Flight의 탑승일을 기준으로
Passenger의 연령 구분을 Flight별로 계산합니다.

따라서 출국 Flight와 귀국 Flight에서 연령 구분이 달라질 수 있으며,
Seat 및 동반 Adult 관련 Validation은 각 Flight의 연령 구분을 기준으로 적용합니다.

MVP의 연령 구분 기준은 다음과 같습니다.

- `Infant`: 생후 7일 이상 ~ 만 2세 미만
- `Child`: 만 2세 이상 ~ 만 12세 미만
- `Adult`: 만 12세 이상

생후 7일 미만의 Passenger는 예약할 수 없습니다.

해당 조건에 해당하는 경우 다음 단계로 진행하지 못하도록 하고
사용자에게 예약 불가 사유를 안내합니다.

표시 예:

```text
KIM JIHUN
Adult

KIM MINSU
Child
```

---

### 15.1 Child

Passenger의 Child 여부는 각 Flight의 탑승일을 기준으로 판단합니다.

해당 Flight에서 Child인 Passenger가 포함된 경우
동일 Flight에 최소 1명 이상의 Adult가 함께 탑승해야 합니다.

Child는 동일 Flight에서 함께 탑승하는 Adult 중 최소 1명과
인접한 Seat를 배정받아야 합니다.

MVP에서 인접한 Seat는 동일 Row에서 좌우로 직접 연결되어 있으며,
두 Seat 사이에 통로가 존재하지 않는 Seat를 의미합니다.

안내 예:

```text
소아 탑승객은 해당 Flight에서 함께 탑승하는
성인과 인접한 좌석을 선택해야 합니다.
```

조건을 만족하지 않는 경우 다음 단계로 진행하지 못하도록 합니다.

ROUND_TRIP에서는 위 Validation을
출국 Flight와 귀국 Flight 각각에 적용합니다.

---

### 15.2 Infant

Passenger의 Infant 여부는 각 Flight의 탑승일을 기준으로 판단합니다.

해당 Flight에서 `Infant`인 Passenger는 별도의 Seat를 사용하지 않습니다.

Passenger 입력 화면에서는 Reservation에 포함된 Passenger 중
Infant를 동반할 Adult를 지정합니다.

```text
Infant
KIM BABY

동반 Adult
[ KIM JIHUN ▼ ]
```

Adult 1명당 최대 1명의 Infant를 연결할 수 있습니다.

지정된 동반 Adult는 해당 Infant가 Infant로 분류되는 Flight에서
Adult 조건을 만족해야 합니다.

ROUND_TRIP에서 동일 Passenger가 하나 이상의 Flight에서
Infant로 분류되는 경우 동일한 동반 Adult를 기본으로 사용합니다.

어느 Flight에서든 지정된 동반 Passenger가
Adult 조건을 만족하지 못하면 Reservation을 시작할 수 없습니다.

왕복 일정 중 Passenger의 연령 구분이 변경되는 경우
각 Flight의 연령 구분에 따라 Seat 필요 여부와
Infant 동반 Validation을 각각 적용합니다.

Infant만으로 Reservation을 생성할 수 없습니다.

---

## 16. 예약 확인

Mock 결제로 이동하기 전에 Reservation 전체 내용을 확인합니다.

표시 정보:

- 여행 유형
- 출국 Flight
- 귀국 Flight (왕복인 경우)
- 출발 / 도착 Airport
- Date / Time
- Passenger
- Seat
- Infant 동반 정보
- Mock 결제 금액
- Hold 남은 시간

예:

```text
예약 내용 확인

KO101
ICN → NRT

Passenger 1
KIM JIHUN
Seat 12A

Passenger 2
KIM MINSU (Child)
Seat 12B

Hold 남은 시간
00:42:31

[예약 진행 취소]
[결제로 이동]
```

Reservation이 `PENDING` 상태로 생성된 이후 Passenger 또는 Seat를 변경하려면
현재 예약 진행을 취소하고 다시 예약을 시작합니다.

왕복 Reservation에서는 출국 Flight와 귀국 Flight 정보를
별도 Section으로 구분하여 표시합니다.

각 Flight별로 다음 정보를 확인할 수 있어야 합니다.

- Route
- Date / Time
- Passenger
- Seat

Mock 결제 금액은 왕복 전체 예약의 금액을 기준으로 표시합니다.

---

## 17. Mock 결제

### 17.1 결제 안내

실제 금융 거래가 발생하지 않는다는 것을 명확하게 표시합니다.

```text
Mock Payment

본 프로젝트에서는 실제 금융 거래가 발생하지 않습니다.
```

실제 카드번호 등 금융 개인정보를 입력받는 UI는 구현하지 않습니다.

왕복 Reservation에서도 Mock Payment는 전체 왕복 여정을 대상으로
한 번 진행합니다.

결제 화면에는 다음 정보를 요약하여 표시합니다.

- 출국 Flight
- 귀국 Flight
- Passenger
- 출국 Seat
- 귀국 Seat
- 전체 Mock 결제 금액

---

### 17.2 결제 Action

```text
[예약 진행 취소]
[Mock 결제하기]
```

`예약 진행 취소`를 선택하면 Confirmation UI를 표시합니다.

취소가 완료되면:

```text
Reservation
PENDING → CANCELLED

해당 Reservation의 모든 Seat
HELD → AVAILABLE

현재 처리 중인 PENDING Payment가 존재하는 경우
PENDING → CANCELLED
```

기존 `FAILED` Payment는 결제 이력으로 유지합니다.

Payment가 아직 생성되지 않은 상태에서 예약 진행을 취소한 경우
새로운 Payment를 생성하지 않습니다.

---

### 17.3 결제 성공

결제 성공 후:

```text
Payment
PENDING → SUCCESS

Reservation
PENDING → CONFIRMED

선택한 모든 Seat
HELD → RESERVED
```

아래 예시는 편도 Reservation 기준입니다.

사용자 화면:

```text
예약이 완료되었습니다.

KO101
ICN → NRT
2026.09.10

[예약 상세 보기]
[홈으로]
```

왕복 Reservation의 예약 완료 화면에서는
출국 Flight와 귀국 Flight를 함께 요약하여 표시합니다.

예:

```text
출국: KO101 / ICN → NRT / 2026-09-10
귀국: KO102 / NRT → ICN / 2026-09-15
```

---

### 17.4 결제 실패 및 재시도

결제가 실패했지만 재시도가 가능한 경우:

```text
결제를 완료하지 못했습니다.

결제 시도 1 / 3
좌석 Hold 00:25:14

[다시 결제]
[예약 진행 취소]
```

결제 재시도 시 새로운 `Payment`를 생성합니다.

하나의 `Payment`는 하나의 Mock 결제 시도를 의미하며,
기존 `FAILED` Payment는 결제 이력으로 유지합니다.

하나의 Reservation에서는 최초 시도를 포함하여
최대 3개의 Payment를 생성할 수 있습니다.

---

### 17.5 결제 3회 실패

```text
결제 가능 횟수를 모두 사용했습니다.

Reservation이 취소되었으며
선택한 모든 좌석이 다시 예약 가능한 상태로 반환되었습니다.

[항공편 다시 검색]
```

상태 변화:

```text
Reservation → CANCELLED
선택한 모든 Seat → AVAILABLE
```

마지막 및 이전의 실패한 결제 시도는 결제 이력으로 유지합니다.

---

## 18. My Page

### 18.1 기본 구조

```text
My Page

├─ 내 정보
├─ 비밀번호 변경
├─ 내 예약
└─ 회원 탈퇴
```

---

### 18.2 비밀번호 변경

LOCAL AuthAccount를 가진 Member는 비밀번호를 변경할 수 있습니다.

새 Password는 회원가입과 동일한 비밀번호 정책을 적용합니다.

- 8자 이상
- 영문 대문자 최소 1자
- 영문 소문자 최소 1자
- 숫자 최소 1자
- 특수문자 최소 1자
- 허용 특수문자: `! @ # $ % ^ & *`

Frontend에서도 각 조건의 충족 여부를 안내하고 검증합니다.

구체적인 현재 Password 재인증 여부 및 API 흐름은
`04-system-design.md`와 `05-data-api-design.md`에서 정의합니다.

---

### 18.3 Reservation 목록

상태별 Filter를 제공할 수 있습니다.

```text
[전체]
[예약 진행 중]
[예약 확정]
[예약 취소]
```

편도 Reservation Card 예:

```text
KO101

ICN → NRT
2026.09.10

예약 확정

[상세보기]
```

왕복 Reservation Card에서는 출국 / 귀국 Route와 Date를 함께 표시합니다.

예:

```text
왕복

ICN → NRT
2026.09.10

NRT → ICN
2026.09.15

예약 확정

[상세보기]
```

---

## 19. Reservation 상세

표시 정보:

- Reservation 식별정보
- Reservation 상태
- Flight
- Passenger
- Seat
- Payment 상태
- 예약 생성 시각
- 취소 가능 여부

`PENDING`인 경우 Hold 남은 시간을 표시합니다.

`CANCELLED`인 경우 가능하면 취소 사유를 사용자에게 표시합니다.

결제가 완료된 `CONFIRMED` Reservation이 Flight 취소로 인해 취소된 경우 예:

```text
Flight 취소로 인해 예약이 취소되었습니다.

Mock 결제 금액은 전액 환불 처리되었습니다.
```

`PENDING` Reservation이며 Seat Hold 시간이 남아 있는 경우
Hold 남은 시간을 표시하고 예약 진행을 계속하거나 취소할 수 있습니다.

```text
좌석 임시 확보 남은 시간
00:32:15

[예약 진행 취소]
[예약 계속하기]
```

`예약 계속하기`를 선택하면 해당 Reservation의 예약 확인 단계로 이동합니다.

사용자는 예약 내용을 다시 확인한 후 Mock 결제를 진행할 수 있습니다.

Seat Hold 시간이 만료된 Reservation은 Backend 정책에 따라
`CANCELLED` 상태로 처리되며 기존 예약 진행을 계속할 수 없습니다.

왕복 Reservation인 경우 다음 정보를 구분하여 표시합니다.

- 여행 유형: 왕복
- 출국 Flight
- 출국 Passenger / Seat
- 귀국 Flight
- 귀국 Passenger / Seat
- 전체 Payment 상태

출국편과 귀국편은 각각 별도 Section으로 표시합니다.

---

## 20. Member 예약 취소

`ONE_WAY` Reservation의 경우
`CONFIRMED` 상태이며 Flight 출발 예정 시각까지 24시간 이상 남아 있으면
예약 취소 Action을 제공합니다.

```text
[예약 취소]
```

선택 시 Confirmation Modal:

```text
예약을 취소하시겠습니까?

예약 취소 시 Mock 결제 금액이 전액 환불되며
해당 Reservation에 포함된 모든 좌석은 다시 예약 가능한 상태로 변경됩니다.

[돌아가기]
[예약 취소]
```

상태 변화:

```text
Reservation
CONFIRMED → CANCELLED

Payment
SUCCESS → REFUNDED

해당 Reservation의 모든 Seat
RESERVED → AVAILABLE
```

출발 예정 시각까지 24시간 미만인 경우:

```text
출발 24시간 이내에는 예약을 취소할 수 없습니다.
```

예약 취소 Action을 제공하지 않거나 Disabled 처리합니다.

### 왕복 Reservation 취소

MVP에서는 `ROUND_TRIP` Reservation의 부분 취소를 지원하지 않습니다.

Member는 출국 Flight 출발 예정 시각까지 24시간 이상 남아 있는 경우에만
왕복 Reservation 전체를 취소할 수 있습니다.

취소가 가능한 경우:

```text
Reservation
CONFIRMED → CANCELLED

성공한 Payment
SUCCESS → REFUNDED

출국 Flight의 예약 Seat
RESERVED → AVAILABLE

귀국 Flight의 예약 Seat
RESERVED → AVAILABLE
```

왕복 전체 Mock 결제 금액을 전액 환불합니다.

다음 경우 Member에게 예약 취소 Action을 제공하지 않거나
Disabled 처리합니다.

- 출국 Flight 출발 예정 시각까지 24시간 미만인 경우
- 출국 Flight가 이미 출발한 경우

MVP에서는 다음 기능을 제공하지 않습니다.

- 출국 Flight만 취소
- 귀국 Flight만 취소
- 출국 후 남은 귀국 Flight만 취소
- 일부 Flight에 대한 부분 Mock 환불

---

## 21. 회원 탈퇴

회원 탈퇴 화면에서는 탈퇴 조건과 결과를 명확히 안내합니다.

다음 Reservation이 존재하는 Member는 탈퇴할 수 없습니다.

- `PENDING`
- 예약에 포함된 Flight 중 아직 출발하지 않은 Flight가 하나 이상 존재하는 `CONFIRMED`

탈퇴할 수 없는 경우:

```text
회원 탈퇴를 진행할 수 없습니다.

현재 진행 중이거나 향후 탑승 예정인 예약이 있습니다.
예약을 먼저 확인해 주세요.

[내 예약 보기]
```

탈퇴 가능한 경우:

```text
회원 탈퇴 후 로그인할 수 없습니다.

기존 예약 및 결제 이력은
서비스 데이터 정합성을 위해 유지됩니다.

[취소]
[회원 탈퇴]
```

---

## 22. 실제 항공편 조회 UI

내부 KOKU Flight와 외부 실제 항공편은 사용자가 혼동하지 않도록 명확하게 구분합니다.

예:

```text
[KOKU 항공편] [실제 항공편 조회]
```

실제 항공편 Card 예:

```text
--------------------------------

실제 항공편

Airline
Flight Number

ICN → NRT
09:20 → 11:40

가격
경유 여부

외부 항공편 정보

[상세 정보]

--------------------------------
```

KOKU Airline의 예약 CTA는 표시하지 않습니다.

---

## 23. AI 항공편 검색 UI

### 23.1 검색 입력

```text
어떤 항공편을 찾고 계신가요?

┌──────────────────────────────┐
│ 9월 초 인천에서 도쿄 가는    │
│ 오전 항공편 찾아줘            │
└──────────────────────────────┘

[AI로 검색]
```

---

### 23.2 Guest 접근

Guest가 AI 항공편 검색에 접근하면:

```text
AI 항공편 검색은
로그인한 Member에게 제공됩니다.

[로그인]
[회원가입]
```

---

### 23.3 AI 검색 결과

```text
AI 추천

검색 조건
- ICN → Tokyo
- 오전 출발
- 가격 우선

추천 1

실제 Flight 정보
...

추천 이유
오전 출발 조건을 만족하고
조회된 항공편 중 가격이 낮은 편입니다.
```

다음 두 영역은 UI에서 명확하게 구분합니다.

- 외부 Flight API 실제 데이터
- AI가 생성한 추천 설명

AI 설명을 실제 항공편 데이터처럼 표시하지 않습니다.

---

## 24. Admin UI

### 24.1 Admin Dashboard

MVP에서는 복잡한 통계 Dashboard보다
주요 운영 화면으로 이동하기 위한 Navigation 중심으로 구성합니다.

```text
Admin Dashboard

[Airport]
[Route]
[Aircraft]
[Flight]
[Reservation]
```

---

### 24.2 Airport 관리

#### Admin

- 목록 조회
- 상세 조회

#### SuperAdmin

- 목록 조회
- 상세 조회
- 생성
- 수정
- 비활성화

권한이 없는 Action은 가능하면 UI에서 숨기는 것을 기본으로 합니다.

다만 실제 권한 검증은 반드시 Backend에서도 수행합니다.

---

### 24.3 Route 관리

#### Admin

- 목록 조회
- 상세 조회

#### SuperAdmin

- 목록 조회
- 상세 조회
- 생성
- 수정
- 비활성화

Route 생성 또는 수정 시 출발 Airport와 도착 Airport는
현재 사용 가능한 Airport만 선택할 수 있도록 합니다.

비활성화된 Airport는 신규 Route 생성에 사용할 수 없습니다.

---

### 24.4 Aircraft / Seat 구성

#### Admin

- Aircraft 목록 조회
- Aircraft 상세 조회
- Seat 구성 조회

#### SuperAdmin

- Aircraft 생성
- Aircraft 수정
- Aircraft 비활성화
- Seat 구성 관리

---

## 25. Flight 관리

Admin과 SuperAdmin은 Flight 및 운항 일정을 관리할 수 있습니다.

주요 흐름:

```text
Flight 목록
 ↓
Flight 상세
 ↓
Flight 생성 / 수정
```

Flight 상세에는 최소 다음 정보를 표시합니다.

- Flight Number
- Route
- Aircraft
- 출발 시각
- 도착 시각
- 운항 상태 (`SCHEDULED`, `CANCELLED`, `DEPARTED`)
- 연결된 Reservation 현황

Flight 생성 또는 수정 시 Route와 Aircraft 선택 항목에는
현재 사용 가능한 데이터만 제공합니다.

- 비활성화된 Route는 새로운 Flight에 사용할 수 없습니다.
- 비활성화된 Aircraft는 새로운 Flight에 배정할 수 없습니다.

실제 유효성 검증은 Backend에서도 반드시 수행합니다.

---

### 25.1 Flight 수정 제한

Reservation이 존재하지 않는 `SCHEDULED` Flight는
Admin 또는 SuperAdmin이 수정할 수 있습니다.

`PENDING` 또는 `CONFIRMED` Reservation이 하나 이상 존재하는 Flight는
예약 및 Seat 정합성에 영향을 주는 핵심 정보의 수정 Action을 제공하지 않습니다.

수정 제한 대상:

- 출발 Airport
- 도착 Airport
- 출발 예정 시각
- 도착 예정 시각
- Aircraft
- Flight Number

수정이 제한된 경우 다음과 같이 안내합니다.

```text
연결된 예약이 존재하여
이 Flight의 핵심 운항 정보는 수정할 수 없습니다.

운항을 중단해야 하는 경우 Flight 취소를 사용해 주세요.

[확인]
[Flight 취소]
```

수정 Action 표시 기준:

```text
Reservation 없음 → [수정] 표시
PENDING/CONFIRMED Reservation 존재 → [수정] 숨김 또는 비활성화
```

---
### 25.2 Flight 취소

아래의 기본 Flight 취소 흐름은 편도 Reservation을 기준으로 합니다.

`SCHEDULED` 상태이며 아직 출발하지 않은 Flight에만 취소 Action을 제공합니다.

```text
[Flight 취소]
```

선택 시 위험 Action Confirmation UI를 표시합니다.

```text
Flight를 취소하시겠습니까?

연결된 PENDING 및 CONFIRMED Reservation이 모두 취소됩니다.

CONFIRMED Reservation
→ Mock 전액 환불

PENDING Reservation
→ 예약 진행 취소 및 모든 좌석 반환

취소 사유
[                              ]

[돌아가기]
[Flight 취소]
```

취소 사유는 필수 입력입니다.

Flight 취소 후:

#### Flight

```text
Flight
SCHEDULED → CANCELLED
```

#### CONFIRMED Reservation

```text
Reservation
CONFIRMED → CANCELLED

성공한 Payment
SUCCESS → REFUNDED

해당 Reservation의 모든 Seat
RESERVED → AVAILABLE
```

#### PENDING Reservation

```text
Reservation
PENDING → CANCELLED

현재 처리 중인 PENDING Payment가 존재하는 경우
PENDING → CANCELLED

해당 Reservation의 모든 Seat
HELD → AVAILABLE
```

기존 `FAILED` Payment는 상태를 변경하지 않고 결제 이력으로 유지합니다.

MVP에서는 대체편 제공 또는 자동 재예약을 제공하지 않습니다.

#### 왕복 Reservation과 Flight 취소

`ROUND_TRIP` Reservation에 포함된 출국 Flight 또는 귀국 Flight 중
하나가 취소되면 왕복 Reservation 전체를 `CANCELLED` 상태로 처리합니다.

아직 출국 Flight와 귀국 Flight가 모두 출발하지 않은
`CONFIRMED` Reservation의 경우:

```text
Reservation
CONFIRMED → CANCELLED

성공한 Payment
SUCCESS → REFUNDED

출국 / 귀국 Flight의 모든 예약 Seat
RESERVED → AVAILABLE
```

PENDING Reservation의 경우:

```text
Reservation
PENDING → CANCELLED


현재 처리 중인 PENDING Payment가 존재하는 경우
PENDING → CANCELLED


출국 / 귀국 Flight의 모든 HELD Seat
HELD → AVAILABLE
```

기존 FAILED Payment는 결제 이력으로 유지합니다.

출국 Flight가 이미 DEPARTED 상태인 이후
귀국 Flight가 취소된 경우:

```text
Reservation
CONFIRMED → CANCELLED

성공한 Payment
SUCCESS → REFUNDED
```

이 경우:
- 이미 출발한 Flight의 Seat 상태 및 과거 탑승 이력은 변경하지 않음
- 아직 출발하지 않은 귀국 Flight의 RESERVED Seat만 AVAILABLE로 반환
- Mock 결제 금액은 전액 환불

MVP에서는 왕복 Reservation에서도
대체편 제공, 자동 재예약 및 부분 환불을 제공하지 않습니다.

UI에서는 Flight 취소가 왕복 여정 전체에 미치는 영향을
Confirmation UI에서 명확하게 안내합니다.

---

## 26. Reservation 관리자 화면

Admin과 SuperAdmin은 Reservation 현황을 조회할 수 있습니다.

검색 및 Filter 예:

- Reservation 상태
- Flight Number
- 출발 Date
- Member
- Reservation 식별정보

목록 예:

```text
Reservation   Member     Flight     State
------------------------------------------------
R00001        user1      KO101      CONFIRMED
R00002        user2      KO101      CANCELLED
```

왕복 Reservation은 출국 Flight Number 또는 귀국 Flight Number로도
검색할 수 있어야 합니다.

왕복 Reservation 목록에서는 필요 시
출국 / 귀국 Flight를 함께 식별할 수 있도록 표시합니다.

Reservation을 선택하면 상세 화면으로 이동합니다.

---

## 27. SuperAdmin 강제 예약 취소

SuperAdmin의 Reservation 상세 화면에서만 강제 취소 Action을 제공합니다.

Admin에게는 해당 Action을 제공하지 않습니다.

강제 취소 가능 조건:

- Reservation이 `CONFIRMED`
- `ONE_WAY`에서는 해당 Flight가 아직 출발하지 않음
- `ROUND_TRIP`에서는 출국 Flight와 귀국 Flight가 모두 아직 출발하지 않음

```text
[강제 취소]
```

선택 시:

```text
이 Reservation을 강제로 취소하시겠습니까?

취소 후:

Reservation: CONFIRMED → CANCELLED
성공한 Payment: SUCCESS → REFUNDED
해당 Reservation의 모든 Seat: RESERVED → AVAILABLE

취소 사유
[                              ]

[돌아가기]
[강제 취소]
```

취소 사유는 필수입니다.

Member에게 적용되는 출발 24시간 전 취소 제한은
SuperAdmin 강제 취소에는 적용하지 않습니다.

강제 취소 완료 후 처리 결과를 화면에 표시합니다.

왕복 Reservation의 일부 Flight가 이미 `DEPARTED` 상태인 경우
강제 취소 Action을 제공하지 않거나 Disabled 처리합니다.

강제 취소가 가능한 왕복 Reservation은 전체를 취소합니다.

```text
Reservation
CONFIRMED → CANCELLED

성공한 Payment
SUCCESS → REFUNDED

출국 / 귀국 Flight의 모든 예약 Seat
RESERVED → AVAILABLE
```

Mock 결제 금액은 전액 환불합니다.

MVP에서는 왕복 Reservation의 일부 Flight만 강제 취소하거나
부분 Mock 환불하는 기능을 제공하지 않습니다.

---

## 28. Loading / Empty / Error State

모든 주요 조회 화면은 최소 다음 상태를 고려합니다.

### 28.1 Loading

예:

```text
항공편을 조회하고 있습니다...
```

Loading Spinner 또는 Skeleton UI 등을 사용할 수 있습니다.

---

### 28.2 Empty

예:

```text
조건에 맞는 항공편이 없습니다.

검색 조건을 변경해 주세요.
```

---

### 28.3 Error

예:

```text
항공편 정보를 불러오지 못했습니다.

잠시 후 다시 시도해 주세요.

[다시 시도]
```

---

## 29. 주요 예외 UI

### 29.1 Seat 경쟁 실패

선택한 Seat 중 하나 이상을 다른 사용자가 먼저 확보한 경우
Reservation 시작이 완료되지 않았음을 사용자에게 안내합니다.

```text
선택한 좌석 중 일부를 다른 사용자가 먼저 선택했습니다.

좌석 정보를 다시 확인해 주세요.

[좌석 다시 선택]
```

Seat 경쟁 실패 시 나머지 Seat의 처리 방식은
`02-domain-policy.md`에서 정의한 정책을 따릅니다.

---

### 29.2 Seat Hold 만료

```text
좌석 임시 확보 시간이 만료되었습니다.

[좌석 다시 선택]
```

---

### 29.3 외부 Flight API 장애

```text
현재 실제 항공편 정보를 불러올 수 없습니다.

KOKU Airline 내부 항공편 예약 서비스는
정상적으로 이용할 수 있습니다.

[다시 시도]
```

---

### 29.4 AI 응답 실패

```text
AI 추천을 생성하지 못했습니다.

일반 실제 항공편 검색을 이용하거나
다시 시도해 주세요.

[다시 시도]
[일반 검색으로 이동]
```

---

### 29.5 인증 필요

```text
로그인이 필요한 서비스입니다.

[로그인]
```

---

### 29.6 권한 없음

```text
이 기능에 접근할 권한이 없습니다.
```

---

### 29.7 지원하지 않는 노선

KOKU Airline 내부 검색,
외부 실제 항공편 검색 및 AI 항공편 검색은
한국 ↔ 일본 노선만 지원합니다.

지원하지 않는 노선이 입력된 경우:

```text
현재 KOKU Airline은
한국과 일본 사이의 항공편만 지원합니다.

출발지와 목적지를 다시 확인해 주세요.

[검색 조건 수정]
```

외부 실제 항공편 검색 및 AI 항공편 검색에서는
해당 Validation이 실패한 경우 외부 Flight API를 호출하지 않습니다.

---

## 30. 화면 지원 범위

KOKU Airline Renewal의 MVP는 Web 기반 서비스로 구현하며,
일반 사용자용 Customer UI는 Desktop Web과 Mobile Web을 지원합니다.

Frontend는 Responsive Web을 기본으로 합니다.

기본 디자인 기준 Width:

- Desktop: `1440px`
- Mobile: `390px`

Desktop과 Mobile은 동일한 주요 사용자 기능과
비즈니스 규칙을 제공합니다.

화면 크기에 따라 Layout과 Component 배치는 변경할 수 있지만
기능, 상태 또는 사용자 흐름을 임의로 제거하지 않습니다.

### 30.1 Desktop Customer UI

Desktop에서는 다음 Layout을 우선적으로 사용할 수 있습니다.

- Global Header Navigation
- 넓은 검색 Form
- 검색 결과 Card Layout
- 필요한 경우 2 Column Layout
- 예약 과정에서 요약 정보 Side Panel
- 충분한 Horizontal Space를 활용한 Reservation Step
- Seat Map과 Passenger 정보를 함께 확인할 수 있는 Layout

Figma Wireframe의 Desktop 기준 Width는 `1440px`로 합니다.

### 30.2 Mobile Customer UI

Mobile에서는 동일한 정보와 기능을
좁은 화면에 맞게 재배치합니다.

Figma Wireframe의 Mobile 기준 Width는 `390px`로 합니다.

기본 방향:

- 주요 콘텐츠는 1 Column Layout 우선
- Desktop Horizontal Form은 Vertical Stack으로 전환 가능
- Flight Card는 화면 Width에 맞춰 세로형으로 재배치
- 2 Column Layout은 필요 시 1 Column으로 전환
- Desktop Side Panel 정보는 본문 Section 또는 Summary Card로 이동
- 주요 CTA는 충분한 Touch Area 확보
- 긴 Form은 Field 단위로 세로 배치
- Modal은 Mobile에서 화면 폭에 맞는 Dialog 또는 Full-width 표현 가능
- 주요 Action이 화면 밖으로 과도하게 밀리지 않도록 구성
- Mobile Header와 상단 고정 UI는 Device Safe Area를 고려하여
  Logo, Navigation, Locale 및 주요 Action이 화면 상단 시스템 영역과 겹치지 않도록 배치합니다.
- Header Background는 화면 최상단까지 이어질 수 있으나,
  Header Content는 Safe Area 아래에서 시작하도록 구성합니다.

Desktop에서 존재하는 주요 Customer 기능을
Mobile에서 임의로 제거하지 않습니다.

### 30.3 Mobile Seat 선택

Mobile Seat Map에서도 Seat의 실제 Row / Column 관계와
인접 Seat 의미를 유지해야 합니다.

좁은 화면에서 전체 Seat Map을 한 번에 표시하기 어려운 경우
가로 Scroll 등 Mobile에 적합한 탐색 방식을 사용할 수 있습니다.

단, 화면 크기에 맞추기 위해 Seat의 실제 배치 관계를
임의로 변경해서는 안 됩니다.

Mobile에서도 다음 내용을 확인할 수 있어야 합니다.

- 현재 Flight
- Passenger
- Seat 상태 범례
- 현재 Passenger의 선택 Seat
- 선택 완료 Action
- 왕복의 경우 출국 / 귀국 Flight 구분

### 30.4 Admin / SuperAdmin

Admin 및 SuperAdmin 관리 화면은
MVP에서 Desktop Web 사용성을 우선합니다.

관리 화면은 Table, 상세 정보, 운영 Action이 많으므로
Mobile 전용 최적화를 MVP 필수 범위로 두지 않습니다.

Admin / SuperAdmin의 Mobile Wireframe은
MVP에서 필수로 생성하지 않습니다.

### 30.5 Tablet

Tablet 전용 Wireframe은 MVP 필수 범위에 포함하지 않습니다.

Responsive Web 구현 과정에서 일반 Customer UI가
중간 Width에서도 심각하게 깨지지 않도록 고려하되,
Tablet 전용 Navigation이나 별도 Layout을
독립적으로 설계하는 것은 MVP 필수사항이 아닙니다.

구체적인 CSS Breakpoint와 구현 방식은
Frontend 구현 단계에서 확정합니다.

---

## 31. 접근성 기본 원칙

MVP에서도 최소한 다음 사항을 고려합니다.

- Form Input에는 연결된 Label을 제공합니다.
- Error Message를 해당 Input과 연결합니다.
- Keyboard를 통한 주요 기능 접근을 고려합니다.
- 상태를 색상만으로 표현하지 않습니다.
- Button과 Link의 역할을 구분합니다.
- Disabled 상태를 명확하게 표현합니다.
- Focus 상태를 확인할 수 있도록 합니다.
- 한국어와 일본어 환경에서 Text가 잘리지 않도록 합니다.
- Mobile에서는 Button, Link, Form Control 등 주요 Action이
  Touch로 조작하기 충분한 크기와 간격을 가지도록 합니다.
- Desktop과 Mobile 모두에서 한국어 / 일본어 전환 시
  Text Overflow로 인해 주요 Action이나 정보가 가려지지 않도록 합니다.

---

## 32. Low-Fidelity Wireframe

### 32.1 Home

```text
[KOKU Airline]    항공편  실제항공편  AI    KO | JA

          한국과 일본을 연결하는 KOKU Airline
        가상 항공편 예약과 실제 항공편 조회를 한 번에

[편도] [왕복 - 선택]

출발지       도착지       출발일       도착일
ICN          NRT          2026-09-10   2026-09-15

※ 편도 선택 시 도착일 Input은 표시하지 않습니다.

                         [항공편 검색]

[KOKU Airline 항공편]
좌석을 선택하고 직접 예약할 수 있습니다.

[실제 항공편 조회]
실제 한국 ↔ 일본 항공편을 검색합니다.

[AI 항공편 추천]
자연어로 원하는 항공편을 검색합니다.

[AI 항공편 추천 받기]
```

---

### 32.2 Flight 검색 결과

```text
+------------------------------------------------------+
| ICN → NRT                            2026-09-10      |
+------------------------------------------------------+

KO101

09:30 ICN ---------------------- 11:50 NRT

예약 가능

                                      [상세보기]

--------------------------------------------------------

KO205

14:00 ICN ---------------------- 16:20 NRT

예약 마감

                                      [상세보기]
```

---

### 32.3 왕복 Flight 검색 결과

```text
[1 출국편] ─ [2 귀국편]

출국편
ICN → NRT
2026-09-10

KO101
09:30 → 11:50
[선택]

--------------------------------

선택한 출국편
KO101 / ICN → NRT / 09:30

귀국편
NRT → ICN
2026-09-15

KO102
17:00 → 19:30
[선택]
```

---

### 32.4 Seat 선택 (편도 예시)

```text
KO101
ICN → NRT


                FRONT

          A   B       C   D

1         □   □       ■   □
2         □   X       □   □
3         □   □       □   □


□ 선택 가능
■ 현재 선택
X 선택 불가


Passenger

Adult  KIM JIHUN  → 1A
Child  KIM MINSU  → 1B


                              [좌석 선택 완료]
```

---

### 32.5 Mock 결제 (편도 예시)

```text
+---------------------------------------+
| Mock Payment                          |
+---------------------------------------+

KO101
ICN → NRT

Passenger 2명
Seat 1A / 1B

남은 Hold 시간
00:31:52

본 화면에서는 실제 금융 거래가 발생하지 않습니다.

결제 시도
1 / 3


[예약 진행 취소]       [Mock 결제하기]
```

---

### 32.6 Reservation 상세 (편도 예시)

```text
Reservation

예약 확정

KO101
ICN → NRT
2026-09-10 09:30


Passenger

KIM JIHUN
Seat 1A

KIM MINSU
Seat 1B


Payment
결제 완료

출발까지 24시간 이상 남았습니다.


                              [예약 취소]
```

---

### 32.7 Admin Flight 상세

```text
Flight Detail

KO101
ICN → NRT

Departure
2026-09-10 09:30

Arrival
2026-09-10 11:50

Aircraft
KOKU-A01

운항 상태
[상태 표시]

Reservations
24


[수정]

[Flight 취소]
```
---

### 32.8 Mobile Home

기준 Width: `390px`

```text
+------------------------------+
| KOKU Airline       KO|JA ☰  |
+------------------------------+

한국과 일본을 연결하는
KOKU Airline

가상 항공편 예약과
실제 항공편 조회를 한 번에

[ 편도 ] [ 왕복 ]

출발지
[ ICN                  ]

도착지
[ NRT                  ]

출발일
[ 2026-09-10           ]

도착일
[ 2026-09-15           ]

[        항공편 검색        ]

-------------------------------

KOKU Airline 항공편
좌석을 선택하고 예약할 수 있습니다.

[항공편 검색]

-------------------------------

실제 항공편 조회
실제 한국 ↔ 일본 항공편을 검색합니다.

[실제 항공편 조회]

-------------------------------

AI 항공편 검색
자연어로 항공편을 추천받습니다.

[AI 항공편 추천 받기]
```

편도 선택 시 도착일 Input은 표시하지 않습니다.

---

### 32.9 Mobile Flight 검색 결과

```text
+------------------------------+
| ←  ICN → NRT                |
|    2026-09-10               |
+------------------------------+

KOKU Airline
KO101

09:30 ICN
   ↓
11:50 NRT

예약 가능

[          상세보기          ]

-------------------------------

KOKU Airline
KO205

14:00 ICN
   ↓
16:20 NRT

예약 마감

[          상세보기          ]
```

---

### 32.10 Mobile 왕복 Flight 검색

```text
1 / 2 출국편 선택

ICN → NRT
2026-09-10

KO101
09:30 → 11:50

[출국편 선택]

------------------------------

선택한 출국편
KO101
ICN → NRT
09:30 → 11:50

2 / 2 귀국편 선택

NRT → ICN
2026-09-15

KO102
17:00 → 19:30

[귀국편 선택]
```

출국 Flight 선택 전에는 귀국 Flight 선택 단계로 진행하지 않습니다.

---

### 32.11 Mobile Seat 선택

```text
2 / 5 좌석 선택

출국편
KO101
ICN → NRT

Passenger
KIM JIHUN

□ 선택 가능
■ 현재 선택
X 선택 불가

        FRONT

     A  B    C  D
1    ■  □    □  □
2    □  X    □  □
3    □  □    □  □

선택 Seat
1A

[      좌석 선택 완료      ]
```

Seat Map의 실제 Row / Column 및 인접 Seat 관계는
Desktop과 동일하게 유지합니다.

---

### 32.12 Mobile Mock 결제

```text
4 / 5 결제

Mock Payment

KO101
ICN → NRT

Passenger 2명
Seat 1A / 1B

Hold 남은 시간
00:31:52

결제 시도
1 / 3

실제 금융 거래가
발생하지 않습니다.

[예약 진행 취소]

[      Mock 결제하기      ]
```

---

## 33. UI에서 직접 결정하지 않는 사항

다음 사항은 UI Design에서 임의로 결정하지 않습니다.

- API Request / Response 구조
- Database Column
- Entity 관계
- JWT 저장 방식
- Access Token / Refresh Token 정책
- 동시성 제어 구현 방식
- Transaction Boundary
- 개인정보 저장 및 보호 방식
- Cache 적용 여부 및 TTL
- 구체적인 Spring Security 내부 구조

해당 사항은 `04-system-design.md` 또는
`05-data-api-design.md`에서 정의합니다.

---

## 34. UI Design 완료 기준

다음 조건을 충족하면 MVP UI 설계가 완료된 것으로 판단합니다.

- [ ] Guest 주요 화면이 정의되어 있습니다.
- [ ] Member 주요 화면이 정의되어 있습니다.
- [ ] Admin 및 SuperAdmin 주요 화면이 정의되어 있습니다.
- [ ] KOKU Flight 검색 흐름이 정의되어 있습니다.
- [ ] 편도 / 왕복 검색 흐름이 정의되어 있습니다.
- [ ] 왕복 출국 / 귀국 Flight 선택 흐름이 정의되어 있습니다.
- [ ] 왕복 Seat 선택 및 예약 확인 UI가 정의되어 있습니다.
- [ ] 왕복 결제 / 취소 / Flight 취소 정책이 Domain Policy와 동기화되어 있습니다.
- [ ] 로그인 및 회원가입 흐름이 정의되어 있습니다.
- [ ] Google OAuth 및 동일 Email 계정 연동 흐름이 정의되어 있습니다.
- [ ] Seat 선택 및 Hold 흐름이 정의되어 있습니다.
- [ ] Passenger 입력 흐름이 정의되어 있습니다.
- [ ] Child 및 Infant UI 규칙이 정의되어 있습니다.
- [ ] Mock 결제 성공 / 실패 / 재시도 흐름이 정의되어 있습니다.
- [ ] Reservation 조회 및 취소 흐름이 정의되어 있습니다.
- [ ] Flight 취소 UI가 정의되어 있습니다.
- [ ] SuperAdmin 강제 취소 UI가 정의되어 있습니다.
- [ ] 내부 KOKU Flight와 외부 실제 항공편 UI가 명확하게 구분되어 있습니다.
- [ ] AI 항공편 검색 흐름이 정의되어 있습니다.
- [ ] Loading / Empty / Error 상태가 정의되어 있습니다.
- [ ] 한국어 / 일본어 UI 정책이 정의되어 있습니다.
- [ ] 주요 Low-Fidelity Wireframe이 정의되어 있습니다.
- [ ] 주요 Customer UI의 Desktop / Mobile Layout 정책이 정의되어 있습니다.
- [ ] Mobile Navigation 정책이 정의되어 있습니다.
- [ ] Mobile에서 검색 / Flight 선택 / Passenger / Seat / Payment 흐름이 정의되어 있습니다.
- [ ] Desktop과 Mobile에서 동일한 주요 Customer 기능을 제공하도록 정의되어 있습니다.
- [ ] 주요 Mobile Low-Fidelity Wireframe이 정의되어 있습니다.
- [ ] Admin / SuperAdmin의 Mobile 최적화가 MVP 필수 범위가 아님을 명확히 정의했습니다.

---

## 35. UI Design 변경 원칙

본 문서는 `01-project-plan.md`와 `02-domain-policy.md`의 범위와 정책을 기반으로 합니다.

UI 설계 또는 Frontend 구현 과정에서 새로운 비즈니스 규칙이 필요한 경우
Frontend 또는 AI Agent가 임의로 정책을 추가하지 않습니다.

다음과 같은 변경이 필요한 경우 `02-domain-policy.md`를 먼저 검토합니다.

- 새로운 Reservation 상태가 필요한 경우
- Seat 상태 또는 Hold 정책을 변경해야 하는 경우
- 새로운 Role 또는 권한이 필요한 경우
- 결제 흐름을 변경해야 하는 경우
- Passenger 및 연령 정책을 변경해야 하는 경우
- Flight 취소 정책을 변경해야 하는 경우
- 내부 KOKU Flight와 외부 실제 항공편의 경계를 변경해야 하는 경우
- 편도 / 왕복 여행 유형 또는 왕복 예약 정책을 변경해야 하는 경우

Domain Policy 변경이 필요한 경우
Human의 검토 및 승인 후 관련 설계 문서의 일관성을 함께 수정합니다.