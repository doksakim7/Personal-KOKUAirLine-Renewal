# Figma Make Guidelines - KOKU Airline Renewal

## 1. 목적

본 문서는 Figma Make가 `KOKU Airline Renewal`의 UI를 생성할 때 따라야 하는 고정 가이드라인입니다.

이 문서의 목적은 다음과 같습니다.

- Figma Make가 프로젝트의 범위와 설계 의도를 정확히 이해하도록 합니다.
- `03-ui-design.md`를 기반으로 화면을 생성하되, 임의의 비즈니스 규칙을 추가하지 않도록 합니다.
- 생성되는 결과물이 MVP 범위, 사용자 흐름, 다국어 정책, Role 정책과 일관되도록 합니다.
- Wireframe / Prototype 생성 시 화면 구조와 UX 방향을 명확히 제한합니다.

---

## 2. Source of Truth

Figma Make는 다음 우선순위에 따라 문서를 해석합니다.

1. `02-domain-policy.md`
2. `03-ui-design.md`
3. `01-project-plan.md`
4. 본 문서 `figma-make-guidelines.md`

단, UI 생성과 직접 관련된 구조·화면 흐름·표시 정책은 `03-ui-design.md`를 우선적으로 따릅니다.

비즈니스 규칙, 상태 전이, Role 권한, 예약 정책 등은 반드시 `02-domain-policy.md`를 기준으로 해석합니다.

문서 간 해석이 애매하거나 충돌하는 경우, Figma Make는 임의로 결정하지 말고 **보수적으로 UI를 생성**해야 합니다.

---

## 3. 프로젝트 개요

프로젝트명: `KOKU Airline Renewal`

프로젝트 성격:

- 포트폴리오용 가상 항공 예약 시스템
- KOKU Airline의 내부 가상 항공편 예약 기능 제공
- 외부 실제 항공편 조회 기능 제공
- AI 기반 실제 항공편 검색/추천 기능 제공

핵심 특징:

- 내부 `Flight`는 예약 가능
- 외부 실제 항공편은 조회/비교만 가능
- 실제 금융 결제는 없으며 `Mock Payment`만 제공
- 한국어/일본어를 모두 지원하는 Desktop / Mobile Responsive Web MVP
- Customer UI는 Desktop과 Mobile에서 동일한 주요 기능 범위를 제공
- Admin / SuperAdmin 관리 UI는 Desktop Web 사용성을 우선

---

## 4. MVP 범위

### 4.1 지원 플랫폼

MVP는 Web 기반 Responsive UI를 지원합니다.

Customer UI는 Desktop Web과 Mobile Web을 모두 지원하며,
화면 크기에 따라 Layout과 Component 배치를 적절하게 재구성합니다.

기본 디자인 기준 Width:

- Desktop: `1440px`
- Mobile: `390px`

Guest와 Member가 사용하는 주요 Customer UI는
Desktop과 Mobile에서 동일한 주요 기능과 사용자 흐름을 제공합니다.

Mobile이라는 이유로 주요 기능, 상태 또는 사용자 흐름을
임의로 제거하거나 단순화하지 않습니다.

Mobile에서는 필요에 따라 다음 Responsive Pattern을 사용할 수 있습니다.

- Hamburger / Menu Navigation
- 1 Column Layout
- Vertical Form Stack
- Full-width CTA
- Mobile에 적합한 Dialog / Sheet 표현
- Seat Map 탐색을 위한 Horizontal Scroll

단, 위 Pattern은 기존 기능을 변경하기 위한 것이 아니라
좁은 화면에 적합하게 동일 기능을 재배치하기 위한 용도로만 사용합니다.

Admin 및 SuperAdmin 관리 화면은
MVP에서 Desktop Web 사용성을 우선합니다.

Admin / SuperAdmin의 Mobile 전용 최적화와
Mobile Wireframe 생성은 MVP 필수 범위에 포함하지 않습니다.

Tablet 전용 Wireframe 또한 MVP 필수 범위에 포함하지 않습니다.

Responsive 구현 과정에서 중간 Width가 심각하게 깨지지 않도록 고려하되,
구체적인 CSS Breakpoint는 Frontend 구현 단계에서 확정합니다.

### 4.2 MVP에서 우선 생성할 수 있는 주요 화면

- Home
- KOKU Flight 검색 결과
- 왕복 Flight 검색 결과
  - 출국 Flight 선택
  - 귀국 Flight 선택
- Flight 상세
- 로그인
- 회원가입
- Passenger 정보 입력
- Seat 선택
- 예약 확인
- Mock 결제
- 예약 완료
- Reservation 상세
- 외부 실제 항공편 검색 결과
- AI 항공편 검색
- My Page
- Admin Dashboard
- Admin / SuperAdmin 관리 화면

위 화면 중 Guest와 Member가 사용하는 주요 Customer UI는
Desktop 및 Mobile Wireframe을 모두 생성합니다.

Mobile Wireframe은 Desktop 화면을 단순 축소하지 않고,
`03-ui-design.md`의 Responsive 정책에 따라
동일한 기능과 정보를 Mobile Layout으로 재배치합니다.

우선적인 Mobile Wireframe 대상은 다음과 같습니다.

- Home
- KOKU Flight 검색 결과
- 왕복 Flight 검색
- Passenger 정보 입력
- Seat 선택
- 예약 확인
- Mock 결제
- Reservation 상세
- My Page

Admin / SuperAdmin 관리 화면의 Mobile Wireframe은
MVP 필수 생성 범위가 아닙니다.

---

## 5. 언어 정책

MVP는 다음 두 언어를 모두 지원해야 합니다.

- 한국어 (`ko`)
- 일본어 (`ja`)

생성 원칙:

- 한 화면 안에서 언어 전환 UI를 고려합니다.
- 기본 시안은 **한국어 UI 기준**으로 생성합니다.
- 단, Header에 `한국어 | 日本語` 또는 동등한 언어 전환 UI를 반드시 포함합니다.
- 일본어로 전환되어도 레이아웃이 무너지지 않도록 충분한 여백을 둡니다.
- 한/일 두 언어 모두 동일한 기능 범위를 제공해야 합니다.
- Date Picker 등 날짜 관련 UI Component의
  월, 요일 및 날짜 표기도 현재 Locale(`ko`, `ja`)에 맞게 표현합니다.
- 일본어 전환 시 Date Picker의 날짜 관련 문자열을
  한국어 상태로 고정해서 표현하지 않습니다.

---

## 6. 용어 사용 원칙

Figma Make는 화면 라벨과 설명을 생성할 때 다음 원칙을 따릅니다.

- 일반적인 설명과 기능명은 한국어를 기본으로 사용합니다.
- 코드/ERD/상태값과 직접 연결되는 개념은 영문 Canonical Term을 유지할 수 있습니다.

예:

- 일반 UI 문구: `예약 확인`, `결제 진행`, `회원가입`
- Canonical Term 유지 가능: `Member`, `Passenger`, `Reservation`, `Payment`, `Flight`, `Seat`, `Admin`, `SuperAdmin`

상태값은 내부적으로는 영문 Enum을 따르지만, 사용자에게는 자연어로 표시합니다.

예:

- `PENDING` → `예약 진행 중`
- `CONFIRMED` → `예약 확정`
- `CANCELLED` → `예약 취소`
- `SUCCESS` → `결제 완료`
- `FAILED` → `결제 실패`
- `REFUNDED` → `환불 완료`

---

## 7. 디자인 방향

### 7.1 전체 톤

다음 방향의 UI를 생성합니다.

- 현대적이고 깔끔한 항공 예약 사이트
- 과한 장식보다 정보 전달 중심
- 실무 포트폴리오에 어울리는 정돈된 기업형 UI
- 사용자가 현재 단계와 다음 행동을 쉽게 이해할 수 있는 구조
- White / Light Gray 기반의 깔끔한 배경
- Primary CTA가 명확한 구성
- 과도한 애니메이션이나 실험적 레이아웃 지양

### 7.2 시각적 우선순위

다음 요소를 명확하게 구분합니다.

1. 현재 Page의 목적
2. 주요 입력/선택 영역
3. 현재 진행 단계
4. 주요 CTA 버튼
5. 상태 정보
6. 주의/제한/에러 메시지

### 7.3 레이아웃 방향

Desktop에서는 가능하면 다음 패턴을 우선 사용합니다.

- 상단 Global Header
- 중앙 정렬된 메인 콘텐츠
- 검색/입력/요약 Card 기반 UI
- 필요 시 2 Column Layout
- 예약 과정에서는 요약 정보 Side Panel 허용
- 적절한 간격과 섹션 구분 사용

Mobile에서는 동일한 정보와 기능을 다음 방향으로 재배치합니다.

- 주요 콘텐츠는 1 Column Layout 우선
- Desktop Horizontal Form은 Vertical Stack으로 전환 가능
- 2 Column Layout은 1 Column으로 전환 가능
- Side Panel 정보는 본문 Section 또는 Summary Card로 이동 가능
- 주요 CTA는 충분한 Touch Area를 확보
- 긴 Form은 Field 단위로 세로 배치
- 주요 Action이 좁은 화면 밖으로 밀리지 않도록 구성

Desktop과 Mobile 간 전환으로
기능 또는 주요 정보를 삭제하지 않습니다.

### 7.4 Brand Logo

KOKU Airline의 Brand Symbol은 깃발 형태 안에
Wing 요소를 결합한 간결한 항공 Emblem 형태를 기본 방향으로 합니다.

Figma Make는 다음 기준을 따릅니다.

- 깃발을 주요 Motif로 사용
- Wing 요소는 깃발 내부에 통합하여 표현
- 깃발과 Wing이 하나의 Symbol로 인식되도록 구성
- Skull, Bones 등 직접적인 해적 상징은 사용하지 않음
- 작은 Header 영역에서도 식별 가능한 단순한 형태
- KOKU Airline Wordmark와 함께 사용 가능
- Primary Blue 계열과 어울리는 Flat / Minimal 스타일

Logo를 지나치게 복잡한 Illustration 형태로 만들지 않습니다.

첨부된 확정 Logo 시안을 이후 UI 생성의 기본 Brand Symbol 기준으로 사용합니다.
Figma Make는 Symbol의 핵심 구조를 임의로 재해석하거나 다른 형태로 변경하지 않습니다.

### 7.5 Responsive / Accessibility 기본 원칙

- Mobile의 Button, Link, Form Control은 Touch로 조작하기 충분한 크기와 간격을 확보합니다.
- 상태는 색상만으로 구분하지 않습니다.
- 한국어 / 일본어 전환 시 Text가 잘리거나 주요 CTA가 가려지지 않도록 합니다.
- Desktop과 Mobile 모두에서 주요 Form Label과 Error Message의 관계가 명확하게 보이도록 합니다.
- Mobile Header와 상단 고정 UI는 Device Safe Area를 고려하여
  Logo, Locale Toggle, Menu 등 주요 요소가 시스템 UI 영역과 겹치지 않도록 배치합니다.
- Header Background는 화면 최상단까지 이어질 수 있지만,
  Header Content는 Safe Area 아래에서 시작하도록 구성합니다.

---

## 8. 반드시 지켜야 할 핵심 UI 구분

### 8.1 내부 KOKU Flight vs 외부 실제 항공편

Figma Make는 내부 `KOKU Flight`와 외부 실제 항공편을 **절대로 같은 유형의 카드나 같은 CTA 의미로 혼동해서는 안 됩니다.**

#### 내부 `KOKU Flight`

- KOKU Airline 브랜드 표시
- Domain Policy에 따라 계산된 현재 고정 운임 표시
- 예약 가능
- Seat 선택 가능
- Mock Payment 가능
- Flight 선택 완료 후 예약 절차 시작 CTA 제공 가능

KOKU Flight 검색 결과와 Flight 선택 Summary에서는
Domain Policy에 따라 계산된 현재 고정 운임을 표시합니다.

검색 화면의 운임은 고정 운임 정책에 따라 계산된 금액이며,
최종 결제 예정 금액은 `PENDING` Reservation 생성 시 확정됩니다.

Figma Make는 잔여 Seat 수, Seat 점유율 또는 수요에 따라
운임이 실시간으로 변경되는 UI를 생성하지 않습니다.

Flight 선택 Action과 예약 절차 시작 Action은 구분합니다.

- `ONE_WAY`: `[항공편 선택]`
- `ROUND_TRIP` 출국: `[출국편 선택]`
- `ROUND_TRIP` 귀국: `[귀국편 선택]`

필요한 Flight 선택이 모두 완료된 이후:

- Guest: `[로그인 후 예약]`
- Member: `[예약하기]`

#### 외부 실제 항공편

- `실제 항공편 조회`임을 명확히 표시
- 실제 항공사명 / 실제 편명 표시 가능
- 조회 및 비교만 가능
- KOKU Airline 예약 CTA 제공 금지
- Seat 선택 UI 제공 금지
- Mock Payment UI 제공 금지
- 실제 발권 UI 제공 금지

### 8.2 AI 항공편 검색

AI 검색은 **실제 항공편 데이터를 기반으로 조건을 해석하고 추천하는 기능**입니다.

반드시 구분해야 할 것:

- 실제 항공편 데이터 영역
- AI 추천 설명 영역

금지 사항:

- AI가 만든 설명을 실제 Flight 데이터처럼 보이게 만들지 말 것
- AI 추천 결과에 KOKU Airline 예약 CTA를 붙이지 말 것

---

## 9. Role별 UI 정책

### 9.1 Guest

가능:

- Home
- KOKU Flight 검색
- Flight 상세 조회
- 외부 실제 항공편 조회
- 로그인
- 회원가입

제한:

- 예약 생성 불가
- Seat 선택 불가
- Mock 결제 불가
- 개인 예약 조회 불가
- AI 항공편 검색은 진입 가능하더라도 실제 사용 전 로그인 요구

Guest는 KOKU Flight를 먼저 선택한 후 Reservation을 시작할 수 있습니다.

필요한 Flight 선택이 완료되면 다음 Action을 제공합니다.

```text
[로그인 후 예약]
```

Guest가 해당 Action을 선택하면 로그인 UI로 이동합니다.

로그인 성공 후에는 로그인 이전의 검색 및 Flight 선택 상태를 유지합니다.

- `ONE_WAY`: 선택한 Flight 유지
- `ROUND_TRIP`: 선택한 출국 Flight와 귀국 Flight를 모두 유지

인증 성공 후 Passenger 정보 입력 단계로 이어집니다.

Figma Make는 로그인 이후 사용자가
Flight를 처음부터 다시 검색해야 하는 흐름을 만들지 않습니다.

### 9.2 Member

가능:

- Guest 가능 기능 전체
- Passenger 정보 입력
- Seat 선택
- 예약 생성
- Mock 결제
- 자신의 Reservation 조회/취소
- AI 항공편 검색 사용

### 9.3 Admin

가능:

- 운영 데이터 조회
- Flight 관리
- Reservation 현황 조회

제한:

- Master Data 변경 불가
- 강제 Reservation 취소 불가

### 9.4 SuperAdmin

가능:

- Admin의 모든 권한
- Airport / Route / Aircraft / Seat 구성 등 Master Data 관리
- 개별 Reservation 강제 취소
- 중요 운영 데이터 관리

---

## 10. 예약 핵심 흐름

Figma Make는 편도와 왕복 Reservation의 Happy Path를 구분하여 이해해야 합니다.

#### Flight 선택 완료 UI 통일

Figma Make는 `ONE_WAY`와 `ROUND_TRIP`에서
Flight 선택 완료 후 동일한 UX 패턴을 사용합니다.

```text
선택 완료
→ 선택한 여정 Summary
→ 예약 절차 시작 Action
```

- `ONE_WAY`: 1개 Flight Summary
- `ROUND_TRIP`: 출국 / 귀국 Flight를 포함한 하나의 왕복 여정 Summary

`ROUND_TRIP` Summary를 별도의 독립 Page로 강제하지 않습니다.

Desktop과 Mobile 모두
선택 완료 상태 안에서 Summary와 예약 절차 시작 CTA를 제공하는 의미를 유지합니다.

### 10.1 편도 예약

```text
Home
→ Flight 검색
→ 검색 결과
→ Flight 검색 결과 또는 Flight 상세
→ [항공편 선택]
→ 선택한 Flight Summary
→ 예약 절차 시작
→ Passenger 정보 입력
→ Seat 선택
→ PENDING Reservation 생성 + 선택한 모든 Seat HELD
→ 예약 확인
→ Mock 결제
→ 결제 성공
→ Reservation CONFIRMED + Seat RESERVED
→ 예약 완료
→ Reservation 상세
```

예약 절차 시작 Action은 인증 상태에 따라 구분합니다.

Guest:

```text
[로그인 후 예약]
```

Member:

```text
[예약하기]
```

Figma Make는 ONE_WAY Flight 선택과 예약 절차 시작을
하나의 CTA로 합치지 않습니다.

### 10.2 왕복 예약

```text
Home
→ 왕복 검색
→ 출국 Flight 검색 결과
→ 출국 Flight 선택
→ 귀국 Flight 검색 결과
→ 귀국 Flight 선택
→ 선택한 왕복 여정 Summary
→ 예약 절차 시작
→ Passenger 정보 입력
→ 출국 Flight Seat 선택
→ 귀국 Flight Seat 선택
→ PENDING 왕복 Reservation 생성
→ 출국 / 귀국 선택 Seat HELD
→ 예약 확인
→ 왕복 전체 Mock 결제
→ 결제 성공
→ Reservation CONFIRMED
→ 출국 / 귀국 선택 Seat RESERVED
→ 예약 완료
→ Reservation 상세
```

왕복 예약에서는 다음 원칙을 반드시 유지합니다.

- 하나의 ROUND_TRIP Reservation으로 표현
- 출국 Flight와 귀국 Flight를 별도의 Reservation처럼 표현하지 않음
- 출국 Flight를 먼저 선택
- 귀국 Flight는 출국 Route의 정확한 역방향
- 동일한 Passenger 구성을 두 Flight에 공통으로 적용
- Seat는 각 Flight별로 선택
- 출국 / 귀국 Seat 전체 확보에 성공한 경우에만 Reservation 시작
- 왕복 전체 여정을 대상으로 하나의 Mock Payment 흐름 제공
- 출국 Flight와 귀국 Flight를 모두 선택한 이후 예약 절차 시작 Action 제공
- Flight 선택 Action과 예약 절차 시작 Action을 별도로 표현

왕복 검색 중 Flight 상세 화면의 CTA는 현재 선택 단계에 따라 구분합니다.

출국 Flight 선택 단계:

```text
[출국편 선택]
```

귀국 Flight 선택 단계:

```text
[귀국편 선택]
```

귀국 Flight를 조회하는 동안에는
현재 선택된 출국 Flight의 요약 정보를 함께 표시합니다.

Figma Make는 왕복 Flight 선택 단계에서
일반적인 `[예약하기]` CTA로 임의 변경하지 않습니다.

출국 Flight와 귀국 Flight 선택이 모두 완료된 이후에는
왕복 여정 Summary와 함께 인증 상태에 따른 예약 절차 시작 Action을 제공합니다.

Guest:

```text
[로그인 후 예약]
```

Member:

```text
[예약하기]
```

#### 왕복 Flight 변경

왕복 Flight 선택 완료 후
출국 Flight와 귀국 Flight는 각각 변경할 수 있습니다.

Flight 변경은 Home에서 확정한 Airport와 Date를 변경하는 기능이 아닙니다.

- `출국편 변경`: 기존 출국 Route와 출국 Date를 유지하고 다른 출국 Flight를 선택
- `귀국편 변경`: 기존 귀국 Route와 귀국 Date를 유지하고 다른 귀국 Flight를 선택

Flight 변경 시 반대 구간의 선택은 유지합니다.

Airport, Date, Trip Type 등의 검색 조건 변경은
별도의 `[검색 수정]` Action을 사용합니다.

Figma Make는 `[출국편 변경]` 또는 `[귀국편 변경]`을
Airport / Date 변경 기능으로 해석하지 않습니다.

편도와 왕복 모두 Passenger 정보 입력이 Seat 선택보다 먼저입니다.

결제 성공 후에만 Reservation은 CONFIRMED,
선택한 Seat는 RESERVED 상태가 됩니다.

#### Flight 상세의 선택 CTA

Flight 상세에서도 Flight 선택과 예약 절차 시작을 구분합니다.

`ONE_WAY`:

```text
[항공편 선택]
```

`ROUND_TRIP` 출국 Flight:

```text
[출국편 선택]
```

`ROUND_TRIP` 귀국 Flight:

```text
[귀국편 선택]
```

Flight 선택 이전에는
`[예약하기]` 또는 `[로그인 후 예약]`을
예약 절차 시작 CTA로 직접 제공하지 않습니다.

필요한 Flight 선택이 완료된 이후
선택된 여정 Summary에서 다음 Action을 제공합니다.

- Guest: `[로그인 후 예약]`
- Member: `[예약하기]`

### 10.3 Responsive Reservation Step

Desktop에서는 다음과 같이 전체 Step을 가로로 표시할 수 있습니다.

```text
[1 탑승객] ─ [2 좌석] ─ [3 확인] ─ [4 결제] ─ [5 완료]
```

Mobile에서는 좁은 화면에서 다음과 같이 간결한 Step Indicator를 사용할 수 있습니다.

```text
2 / 5 좌석 선택

● ─ ● ─ ○ ─ ○ ─ ○
```

Mobile에서도 전체 단계 수와 현재 진행 단계를
사용자가 확인할 수 있어야 합니다.

ROUND_TRIP의 Seat 단계에서는
현재 선택 중인 Flight가 출국편인지 귀국편인지 함께 표시합니다.

### 10.4 Reservation 번호 표시

`PENDING` Reservation이 정상적으로 생성된 이후에는
Domain Policy에서 정의한 공개 Reservation 번호를 사용자에게 표시합니다.

Reservation 번호는 다음 화면에서 확인할 수 있어야 합니다.

- 예약 확인
- 예약 완료
- Reservation 목록
- Reservation 상세

표시 예:

```text
예약번호
KOKU-20260821-A7F3K9
```

`ROUND_TRIP`도 하나의 Reservation이므로
출국 Flight와 귀국 Flight에 별도의 Reservation 번호를 생성하거나 표시하지 않습니다.

하나의 왕복 Reservation에는 하나의 Reservation 번호만 표시합니다.

Figma Make는 Database Primary Key를
사용자용 예약번호처럼 표시하지 않습니다.

---

## 11. Seat / Hold UX 규칙

### 11.1 Seat 상태

Backend / Domain의 Seat 상태는 다음과 같이 유지합니다.

* `AVAILABLE`
* `HELD`
* `RESERVED`
* `UNAVAILABLE`

다만 최종 사용자용 Seat 선택 UI에서는 다음 세 가지 상태만 표현합니다.

* 선택 가능
* 내가 선택
* 선택 불가

UI 표현 기준:

* `AVAILABLE` → 선택 가능
* 현재 사용자가 화면에서 선택한 Seat → 내가 선택
* 다른 Reservation에서 `HELD` → 선택 불가
* `RESERVED` → 선택 불가
* `UNAVAILABLE` → 선택 불가

Figma Make는 `HELD`, `RESERVED`, `UNAVAILABLE`을
Seat 선택 화면에서 각각 별도의 사용자용 상태명으로 노출하지 않습니다.

`다른 사용자 선택 중`, `예약 완료` 등의 별도 범례도 생성하지 않습니다.

다른 Reservation에서 `HELD`된 Seat는
내부적으로 Hold 상태를 유지하면서 현재 사용자에게는 선택 불가로 표현합니다.

상태 구분은 색상만으로 하지 말고,
텍스트, 아이콘, 패턴 또는 Disabled 표현 등을 함께 사용합니다.

### 11.2 Hold 규칙

* Seat Hold 시간은 `1시간`입니다.
* Hold Countdown UI를 표시합니다.
* Countdown은 사용자 안내용입니다.
* 실제 Hold 만료 여부는 Backend 시간을 기준으로 판단합니다.

### 11.3 Seat 확보 원자성

편도와 왕복 모두 선택한 모든 Seat를 하나의 확보 대상 집합으로 처리합니다.

- 선택한 모든 Seat 확보 성공 → Reservation 생성
- 선택한 Seat 중 하나라도 확보 실패 → Reservation 시작 실패
- 일부 Seat만 `HELD`되고 일부는 실패하는 UI를 만들지 않음

`ROUND_TRIP`에서는 출국 Flight와 귀국 Flight에서
선택한 모든 Seat를 하나의 Reservation 시작 단위로 처리합니다.

따라서:

- 출국 Flight Seat 전체 확보
- 귀국 Flight Seat 전체 확보

두 조건이 모두 성공한 경우에만
왕복 Reservation을 `PENDING` 상태로 표현합니다.

어느 Flight에서든 Seat 확보에 실패하면
왕복 Reservation 시작 전체 실패로 표현합니다.

Flight 단위 Partial Success UI를 생성하지 않습니다.

### 11.4 Mobile Seat UX

Mobile Seat Map에서도 실제 Row / Column 구조와
Seat 간 인접 관계를 유지합니다.

좁은 화면에서 Seat Map 전체가 들어가지 않는 경우
Horizontal Scroll 등 적절한 탐색 방식을 사용할 수 있습니다.

다음 정보는 Mobile에서도 확인할 수 있어야 합니다.

- 현재 Flight
- Passenger
- Seat 상태 범례
- 현재 선택 Seat
- 선택 완료 Action
- ROUND_TRIP인 경우 출국 / 귀국 Flight 구분

화면 크기에 맞추기 위해 Seat 배치 관계를
임의로 변경해서는 안 됩니다.

---

## 12. Passenger / 테스트용 여권 정책

### 12.1 Passenger 입력

Passenger별 입력 정보는 다음과 같습니다.

* 영문 성
* 영문 이름
* 생년월일
* 성별
* 국적

`ROUND_TRIP` Passenger 입력 화면에서는
출국 Flight와 귀국 Flight의 Summary를 모두 함께 표시합니다.

최소 다음 정보를 구분하여 표현합니다.

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

출국 Flight와 귀국 Flight는 하나의 `ROUND_TRIP` Reservation에 포함된 여정으로 표현합니다.

다음 표현은 사용하지 않습니다.

- 귀국 Flight 정보만 단독으로 표시
- 출국 Flight와 귀국 Flight를 별도의 Reservation처럼 표현
- Flight별로 Passenger 입력 Form을 별도로 생성

Desktop에서는 두 Flight Summary를 나란히 표현할 수 있으며,
Mobile에서는 출국 Flight → 귀국 Flight 순서의 1 Column Layout을 사용할 수 있습니다.

### 12.2 테스트용 여권 정보

테스트용 여권 정보는 사용자가 직접 입력하지 않습니다.

시스템이 자동으로 생성합니다.

반드시 UI에 반영할 내용은 다음과 같습니다.

* 여권번호: 테스트용 번호 자동 생성
* 여권 발급국: 입력한 국적과 동일하게 자동 설정
* 여권 만료일: 생성 시점 기준 5년 뒤
* 사용자가 직접 입력하거나 수정할 수 없음

UI에는 다음 의미의 안내를 포함합니다.

* 실제 개인정보나 실제 여권정보를 입력하지 않도록 안내
* 테스트용 여권정보가 시스템에서 자동 생성됨을 안내

테스트용 여권 만료일은 Reservation에 포함된
모든 Flight의 탑승일 이후여야 합니다.

- `ONE_WAY`: 출국 Flight 기준
- `ROUND_TRIP`: 출국 Flight와 귀국 Flight 모두 기준

조건을 만족하지 않으면 예약을 계속 진행할 수 없는 UI를 제공합니다.

예:

```text
테스트용 여권 유효기간이 Flight 출발일 조건을 만족하지 않습니다.
예약을 계속 진행할 수 없습니다.
```



---

## 13. 연령 규칙 UI

Passenger의 연령 구분은 사용자가 직접 선택하지 않고
각 Flight의 탑승일을 기준으로 시스템이 계산합니다.

- `Infant`: 생후 7일 이상 ~ 만 2세 미만
- `Child`: 만 2세 이상 ~ 만 12세 미만
- `Adult`: 만 12세 이상

생후 7일 미만의 Passenger는 예약할 수 없습니다.

`ROUND_TRIP`에서는 출국 Flight와 귀국 Flight 각각의 탑승일을 기준으로
Passenger의 연령 구분을 독립적으로 계산합니다.

따라서 동일 Passenger라도 출국 Flight와 귀국 Flight에서
연령 구분이 달라질 수 있습니다.

Seat 필요 여부와 동반 Adult Validation도
각 Flight의 연령 구분을 기준으로 표현합니다.

### 13.1 Child 규칙

해당 Flight에서 `Child`인 Passenger가 포함된 경우:

- 동일 Flight에 최소 1명 이상의 `Adult`가 함께 탑승해야 함
- 해당 Flight에서 함께 탑승하는 Adult 중 최소 1명과 인접한 Seat 필요
- 인접 Seat 조건 미충족 시 다음 단계 진행 불가

`ROUND_TRIP`에서는 출국 Flight와 귀국 Flight 각각에
위 Validation을 적용합니다.

### 13.2 Infant 규칙

해당 Flight에서 `Infant`인 Passenger는 별도의 Seat를 사용하지 않습니다.

Passenger 입력 화면에서 Reservation에 포함된 Passenger 중
Infant를 동반할 Adult를 지정합니다.

- Adult 1명당 최대 1명의 Infant
- 지정된 동반 Passenger는 Infant가 Infant로 분류되는 Flight에서 `Adult`여야 함
- `ROUND_TRIP`에서 동일 Passenger가 여러 Flight에서 Infant이면 동일 동반 Adult를 기본 사용
- 어느 Flight에서든 동반 Passenger가 Adult 조건을 만족하지 못하면 Reservation 시작 불가
- Infant만으로 Reservation 생성 불가

왕복 일정 중 연령 구분이 변경되면
Flight별로 Seat 필요 여부와 Infant Validation이 달라질 수 있습니다.

---

## 14. Reservation / Payment 상태 정책

### 14.1 Reservation 상태

MVP 기준 Reservation 상태는 다음과 같습니다.

* `PENDING`
* `CONFIRMED`
* `CANCELLED`

### 14.2 Payment 상태

MVP 기준 Payment 상태는 다음과 같습니다.

* `PENDING`
* `SUCCESS`
* `FAILED`
* `CANCELLED`
* `REFUNDED`

### 14.3 결제 성공

결제 성공 시 다음 상태 전이를 전제로 합니다.

* Payment: `PENDING → SUCCESS`
* Reservation: `PENDING → CONFIRMED`
* 선택한 모든 Seat: `HELD → RESERVED`

### 14.4 결제 실패 및 재시도

* 하나의 Payment는 하나의 Mock 결제 시도를 의미합니다.
* 결제 실패 시 재시도할 수 있습니다.
* 재시도 시 새로운 `Payment`를 생성합니다.
* 기존 `FAILED` Payment는 결제 이력으로 유지합니다.
* 하나의 Reservation에서는 최초 시도를 포함하여 최대 3개의 Payment를 생성할 수 있습니다.

UI는 최소 다음 내용을 보여줄 수 있어야 합니다.

* 현재 결제 시도 횟수
* 남은 Seat Hold 시간
* 다시 결제
* 예약 진행 취소

### 14.5 결제 3회 실패

결제 가능 횟수를 모두 사용하면 다음 상태 전이를 전제로 합니다.

* Reservation: `PENDING → CANCELLED`
* 해당 Reservation의 모든 Seat: `HELD → AVAILABLE`

기존 `FAILED` Payment는 결제 이력으로 유지합니다.

### 14.6 왕복 Mock Payment

`ROUND_TRIP`에서도 Payment는 출국 Flight와 귀국 Flight별로 분리하지 않습니다.

왕복 전체 Reservation을 대상으로 하나의 Mock Payment 흐름을 제공합니다.

결제 화면에는 최소 다음 내용을 함께 표시합니다.

- 출국 Flight
- 귀국 Flight
- Passenger
- 출국 Seat
- 귀국 Seat
- 전체 Mock 결제 금액
- 현재 결제 시도 횟수
- Hold 남은 시간

하나의 `Payment`는 하나의 결제 시도를 의미하며,
왕복 Reservation에서도 최초 시도를 포함하여 최대 3회까지 시도합니다.

Figma Make는 출국편 결제와 귀국편 결제를
별도의 결제 단계로 분리해서는 안 됩니다.

---

## 15. 취소 정책 UI

### 15.1 예약 진행 중 취소

`PENDING` Reservation에서 사용자가 예약 진행을 취소하면 다음과 같이 처리합니다.

* Reservation: `PENDING → CANCELLED`
* 해당 Reservation의 모든 Seat: `HELD → AVAILABLE`
* 현재 처리 중인 Payment가 `PENDING` 상태라면 `PENDING → CANCELLED`

Payment가 아직 생성되지 않은 경우 새로운 Payment를 생성하지 않습니다.

### 15.2 Member 예약 취소

#### ONE_WAY

Member가 직접 취소하려면:

- Reservation이 `CONFIRMED`
- Flight 출발 예정 시각까지 24시간 이상 남음

취소 시:

- Reservation: `CONFIRMED → CANCELLED`
- 성공한 Payment: `SUCCESS → REFUNDED`
- 모든 예약 Seat: `RESERVED → AVAILABLE`

#### ROUND_TRIP

MVP에서는 왕복 Reservation의 부분 취소를 지원하지 않습니다.

Member는 출국 Flight 출발 예정 시각까지
24시간 이상 남은 경우에만 왕복 Reservation 전체를 취소할 수 있습니다.

취소 시:

- Reservation: `CONFIRMED → CANCELLED`
- 성공한 Payment: `SUCCESS → REFUNDED`
- 출국 / 귀국 Flight의 모든 예약 Seat: `RESERVED → AVAILABLE`
- 왕복 전체 Mock 결제 금액 전액 환불

다음 UI는 생성하지 않습니다.

- 출국 Flight만 취소
- 귀국 Flight만 취소
- 출국 후 귀국 Flight만 취소
- 부분 Mock 환불

### 15.3 Flight 취소

아래의 기본 Flight 취소 흐름은 `ONE_WAY` Reservation을 기준으로 합니다.

`ROUND_TRIP` Reservation에 포함된 Flight 취소는
아래의 `ROUND_TRIP Reservation의 Flight 취소` 정책을 우선 적용합니다.

Flight 취소 가능 조건은 다음과 같습니다.

* Flight 상태가 `SCHEDULED`
* 아직 출발하지 않은 Flight

Flight 취소 시:

* Flight: `SCHEDULED → CANCELLED`

연결된 `CONFIRMED` Reservation은 다음과 같이 처리합니다.

* Reservation: `CONFIRMED → CANCELLED`
* 성공한 Payment: `SUCCESS → REFUNDED`
* 해당 Reservation의 모든 Seat: `RESERVED → AVAILABLE`

연결된 `PENDING` Reservation은 다음과 같이 처리합니다.

* Reservation: `PENDING → CANCELLED`
* 현재 처리 중인 Payment가 `PENDING`이면 `PENDING → CANCELLED`
* 해당 Reservation의 모든 Seat: `HELD → AVAILABLE`

기존 `FAILED` Payment는 상태를 변경하지 않고 결제 이력으로 유지합니다.

#### ROUND_TRIP Reservation의 Flight 취소

출국 또는 귀국 Flight 중 하나가 취소되면
왕복 Reservation 전체를 `CANCELLED`로 처리합니다.

두 Flight 모두 아직 출발 전인 경우:

- Reservation 전체 취소
- 성공한 Payment 전액 `REFUNDED`
- 출국 / 귀국 예약 Seat 전체 `AVAILABLE`

출국 Flight가 이미 `DEPARTED`한 후 귀국 Flight가 취소된 경우:

- Reservation 전체 `CANCELLED`
- 성공한 Payment 전체 `REFUNDED`
- 이미 출발한 Flight의 Seat 상태 및 과거 이력은 유지
- 아직 출발하지 않은 귀국 Flight Seat만 `AVAILABLE`

MVP에서는 대체편, 자동 재예약, 부분 환불 UI를 만들지 않습니다.

### 15.4 SuperAdmin 강제 취소

- `SuperAdmin`만 사용 가능
- `Admin`은 사용 불가
- Reservation은 `CONFIRMED`
- 취소 사유 필수
- Member의 24시간 제한은 적용하지 않음

`ONE_WAY`:
- 해당 Flight가 아직 출발하지 않은 경우에만 강제 취소 가능

`ROUND_TRIP`:
- 출국 Flight와 귀국 Flight가 모두 아직 출발하지 않은 경우에만 강제 취소 가능
- 일부 Flight가 이미 `DEPARTED`이면 강제 취소 Action을 제공하지 않거나 Disabled 처리
- 왕복 전체 Reservation 취소
- 성공한 Payment 전체 `REFUNDED`
- 출국 / 귀국 예약 Seat 전체 `AVAILABLE`

부분 강제 취소 또는 부분 Mock 환불 UI는 생성하지 않습니다.

---

## 16. Flight 상태 정책

Flight UI는 최소 다음 상태를 고려합니다.

* `SCHEDULED`
* `CANCELLED`
* `DEPARTED`

다음 조건에서는 새로운 Reservation을 생성할 수 없습니다.

* Flight가 `CANCELLED`
* Flight가 `DEPARTED`
* Flight가 `SCHEDULED` 상태이더라도 출발 예정 시각까지 2시간 미만

UI에서는 상황에 따라 다음과 같이 표현합니다.

* `예약 가능`
* `예약 마감`
* 예약 Action 비활성화
* 예약 Action 미노출

---

## 17. Home 화면 생성 규칙

Figma Make가 `Home`을 생성할 때 반드시 다음 요소를 포함합니다.

### 17.1 Global Header

Guest 기준 Header에는 다음 요소를 포함합니다.

* `KOKU Airline`
* `항공편 검색`
* `실제 항공편`
* `AI 항공편 검색`
* 언어 전환 `한국어 | 日本語`
* `로그인`
* `회원가입`

현재 Page에 해당하는 Navigation은
Primary Color underline 등의 Active State를 명확하게 표시합니다.

Home의 항공편 검색 영역을 기준으로 생성하는 경우
`항공편 검색` Navigation을 Active 상태로 표현합니다.

KOKU Airline Logo는 §7.4 Brand Logo 정책을 따릅니다.

Header에서는 깃발 내부에 Wing을 통합한 단순한 Emblem과
KOKU Airline Wordmark 조합을 기본 방향으로 합니다.

#### Mobile Header

Mobile Customer UI에서는 다음 형태를 기본 방향으로 사용할 수 있습니다.

```text
[KOKU Airline]                 [KO | JA] [Menu]
```

Guest Menu:

```text
항공편 검색
실제 항공편
AI 항공편 검색
로그인
회원가입
```

Member Menu:

```text
항공편 검색
실제 항공편
AI 항공편 검색
내 예약
My Page
로그아웃
```

Mobile Menu에서도 현재 Page의 Active State와
Role별 접근 정책은 Desktop과 동일하게 유지합니다.

언어 전환 UI는 Mobile에서도 쉽게 접근 가능해야 합니다.

### 17.2 Hero / 메인 검색 영역

다음 요소를 포함합니다.

- Hero 보조 문구:
  `가상 항공편 예약과 실제 항공편 조회를 한 번에`
- KOKU Flight 검색용 메인 검색 Form
- 여행 유형 선택
  - 편도
  - 왕복
- 출발지
- 도착지
- 출발일
- 도착일 (왕복 선택 시)
- `항공편 검색` 버튼

기본 여행 유형은 `편도`입니다.

왕복 선택 시에만 도착일 Input을 표시합니다.

MVP에서는 한국 ↔ 일본 노선만 입력할 수 있습니다.

왕복에서는:

- 도착일이 출발일보다 이후여야 함
- 출국 Flight를 먼저 선택
- 귀국 Route는 출국 Route의 정확한 역방향
- 출국 선택 후 귀국 Flight 선택 단계로 이동

### 17.3 주요 서비스 안내

다음 3개 서비스를 명확하게 구분합니다.

#### KOKU Airline 항공편

* KOKU Airline의 내부 가상 Flight
* Seat 선택 가능
* 예약 가능
* Mock Payment 가능

#### 실제 항공편 조회

* 외부 Flight API 기반의 실제 항공편 조회
* 조회 및 비교만 가능
* KOKU Airline 예약 불가
* KOKU Airline 예약 CTA 제공 금지

#### AI 항공편 검색

* 자연어 조건 입력
* 실제 항공편 데이터를 기반으로 검색 및 추천
* 실제 Flight 데이터와 AI 추천 설명을 명확하게 구분
* Guest가 실제 기능을 사용하려면 로그인 필요

Home의 AI 서비스 CTA Label은 다음 문구를 사용합니다.

`AI 항공편 추천 받기`

### 17.4 Responsive 표현

#### Desktop

- 기준 Width는 `1440px`입니다.
- 폭이 넓은 Hero 영역을 사용할 수 있습니다.
- 검색 Form은 가로형 또는 Card형 Layout을 사용할 수 있습니다.
- 주요 서비스 영역은 Card 또는 Section 단위로 구분합니다.
- 필요하면 2 Column Layout 및 Summary Side Panel을 사용할 수 있습니다.

#### Mobile

- 기준 Width는 `390px`입니다.
- Desktop 화면을 단순히 축소하지 않습니다.
- Header는 Logo, Locale, Menu 중심으로 간결하게 구성할 수 있습니다.
- 검색 Form은 Vertical Stack을 기본으로 합니다.
- 주요 서비스 Card는 1 Column으로 배치합니다.
- 주요 CTA는 Mobile Width에 맞게 충분한 조작 영역을 확보합니다.
- Desktop의 Side Panel 정보는 본문 Summary 영역으로 이동할 수 있습니다.

Desktop과 Mobile은 동일한 기능 및 사용자 흐름을 유지합니다.

Tablet 전용 화면은 MVP 필수 생성 범위가 아닙니다.

---

## 18. 인증 UI 생성 규칙

### 18.1 로그인

로그인 화면의 주요 입력 항목:

* Email
* Password

주요 Action:

* 로그인
* Google로 로그인
* 회원가입으로 이동

인증 실패 시 계정 존재 여부 등의 불필요한 상세 정보를 노출하지 않습니다.

권장 안내 문구:

`이메일 또는 비밀번호를 확인해 주세요.`

### 18.2 회원가입

주요 입력 항목:

* Email
* Password
* Password 확인

Password 조건:

* 8자 이상
* 영문 대문자 최소 1자
* 영문 소문자 최소 1자
* 숫자 최소 1자
* 특수문자 최소 1자
* 허용 특수문자: `! @ # $ % ^ & *`

가능하면 Password 입력 중 각 조건의 충족 여부를 표시합니다.

### 18.3 Google OAuth

Google OAuth 관련 UI에서는 다음 정책을 따릅니다.

* 신규 사용자는 Google 인증 후 Member와 GOOGLE AuthAccount를 생성할 수 있습니다.
* 기존 GOOGLE 사용자는 연결된 Member 상태를 확인합니다.
* `WITHDRAWN` Member는 로그인할 수 없습니다.
* Google Email과 기존 LOCAL 계정 Email이 같아도 자동으로 계정을 연결하지 않습니다.
* 기존 LOCAL Password 재인증 성공 후 GOOGLE AuthAccount를 연결합니다.

---

## 19. 실제 항공편 조회 UI 규칙

외부 실제 항공편은 내부 KOKU Flight와 시각적으로 명확하게 구분합니다.

가능하면 다음 정보를 표시합니다.

* 실제 항공사
* Flight Number
* 출발 Airport
* 도착 Airport
* 출발 시각
* 도착 시각
* 가격 정보
* 경유 여부
* 외부 실제 항공편이라는 안내

다음 기능은 제공하지 않습니다.

* KOKU Airline 예약 CTA
* Seat 선택
* Mock Payment
* 실제 발권

---

## 20. AI 항공편 검색 UI 규칙

### 20.1 접근 정책

AI 항공편 검색은 로그인한 `Member`에게 제공됩니다.

Guest가 접근하면 로그인 필요 안내를 제공합니다.

### 20.2 검색 입력

자연어 검색 Input을 제공합니다.

입력 예시:

`9월 초 인천에서 도쿄 가는 오전 항공편 찾아줘`

### 20.3 검색 결과

다음 두 영역을 명확하게 구분합니다.

* 외부 Flight API가 제공한 실제 항공편 데이터
* AI가 생성한 추천 설명

AI 설명을 실제 Flight 데이터처럼 표현하지 않습니다.

AI 추천 항공편에는 KOKU Airline 예약 CTA를 제공하지 않습니다.

---

## 21. Admin UI 규칙

### 21.1 Admin Dashboard

MVP에서는 복잡한 통계 Dashboard보다 주요 운영 화면으로 이동하기 위한 Navigation 중심으로 구성합니다.

주요 메뉴:

* Airport
* Route
* Aircraft
* Flight
* Reservation

### 21.2 Airport 관리

`Admin`:

* 목록 조회
* 상세 조회

`SuperAdmin`:

* 목록 조회
* 상세 조회
* 생성
* 수정
* 비활성화

### 21.3 Route 관리

`Admin`:

* 목록 조회
* 상세 조회

`SuperAdmin`:

* 목록 조회
* 상세 조회
* 생성
* 수정
* 비활성화

비활성화된 Airport는 신규 Route 생성에 사용할 수 없습니다.

### 21.4 Aircraft / Seat 구성

`Admin`:

* Aircraft 목록 조회
* Aircraft 상세 조회
* Seat 구성 조회

`SuperAdmin`:

* Aircraft 생성
* Aircraft 수정
* Aircraft 비활성화
* Seat 구성 관리

---

## 22. Flight 관리 UI 규칙

Admin과 SuperAdmin은 Flight 및 운항 일정을 관리할 수 있습니다.

Flight 상세에는 최소 다음 정보를 표시합니다.

* Flight Number
* Route
* Aircraft
* 출발 시각
* 도착 시각
* 운항 상태
* 연결된 Reservation 현황

Flight 생성 또는 수정 시 다음 조건을 따릅니다.

* 활성 Route만 선택 가능
* 활성 Aircraft만 선택 가능
* 비활성화된 Route는 새로운 Flight에 사용할 수 없음
* 비활성화된 Aircraft는 새로운 Flight에 배정할 수 없음

### 22.1 Flight 수정 제한

Reservation이 존재하지 않는 `SCHEDULED` Flight는 수정할 수 있습니다.

`PENDING` 또는 `CONFIRMED` Reservation이 하나 이상 존재하면 다음 핵심 정보의 수정 Action을 제공하지 않습니다.

* 출발 Airport
* 도착 Airport
* 출발 예정 시각
* 도착 예정 시각
* Aircraft
* Flight Number

운항을 중단해야 하는 경우 Flight 취소 Action을 사용합니다.

---

## 23. Reservation 관리자 UI 규칙

Admin과 SuperAdmin은 Reservation 현황을 조회할 수 있습니다.

검색 및 Filter 예:

* Reservation 상태
* Flight Number
* 출발 Date
* Member
* Reservation 번호

Reservation 목록에서 Reservation 상세로 이동할 수 있어야 합니다.

`SuperAdmin`의 Reservation 상세에서는 조건을 충족할 경우 강제 취소 Action을 제공합니다.

`Admin`에게는 강제 취소 Action을 제공하지 않습니다.

`ROUND_TRIP` Reservation은
출국 Flight Number 또는 귀국 Flight Number 중 어느 쪽으로도
검색할 수 있어야 합니다.

왕복 Reservation 목록 및 상세에서는
출국 Flight와 귀국 Flight를 함께 식별할 수 있도록 표현합니다.

Figma Make는 왕복 Reservation을
두 개의 독립 Reservation처럼 표시하지 않습니다.

---

## 24. My Page UI 규칙

My Page에는 다음 영역을 포함합니다.

* 내 정보
* 비밀번호 변경
* 내 예약
* 회원 탈퇴

### 24.1 비밀번호 변경

LOCAL AuthAccount를 가진 Member는 비밀번호를 변경할 수 있습니다.

새 Password에는 회원가입과 동일한 Password 정책을 적용합니다.

### 24.2 Reservation 목록

Reservation 상태별 Filter를 제공할 수 있습니다.

예:

* 전체
* 예약 진행 중
* 예약 확정
* 예약 취소

왕복 Reservation Card에서는 다음 내용을 함께 표시합니다.

- 여행 유형: 왕복
- 출국 Route / Date
- 귀국 Route / Date
- Reservation 상태
- 상세보기 Action

### 24.3 Reservation 상세

가능하면 다음 정보를 표시합니다.

* Reservation 번호
* Reservation 상태
* Flight
* Passenger
* Seat
* Payment 상태
* 예약 생성 시각
* 취소 가능 여부

`ROUND_TRIP` Reservation 상세에서는 다음 정보를 구분하여 표시합니다.

- 여행 유형: 왕복
- 출국 Flight
- 출국 Passenger / Seat
- 귀국 Flight
- 귀국 Passenger / Seat
- 전체 Payment 상태

출국편과 귀국편은 각각 별도 Section으로 표현합니다.

`PENDING` Reservation에서는 Hold 남은 시간을 표시합니다.

Hold 시간이 남아 있으면 예약 진행을 계속하거나 취소할 수 있습니다.

---

## 25. 회원 탈퇴 UI 규칙

다음 Reservation이 존재하는 Member는 회원 탈퇴를 진행할 수 없습니다.

- `PENDING`
- Reservation에 포함된 Flight 중
  아직 출발하지 않은 Flight가 하나 이상 존재하는 `CONFIRMED`

탈퇴 가능한 경우 다음 의미를 명확하게 안내합니다.

* 탈퇴 후 로그인할 수 없음
* 기존 예약 및 결제 이력은 서비스 데이터 정합성을 위해 유지됨

---

## 26. Loading / Empty / Error State 규칙

모든 주요 조회 화면은 최소 다음 상태를 고려합니다.

### 26.1 Loading

예시 문구:

`항공편을 조회하고 있습니다...`

### 26.2 Empty

예시 문구:

`조건에 맞는 항공편이 없습니다.`

`검색 조건을 변경해 주세요.`

### 26.3 Error

예시 문구:

`항공편 정보를 불러오지 못했습니다.`

`잠시 후 다시 시도해 주세요.`

가능한 Action:

* 다시 시도

---

## 27. 주요 예외 UI 규칙

### 27.1 Seat 경쟁 실패

선택한 Seat 중 하나 이상을 다른 사용자가 먼저 확보한 경우 Reservation 시작 실패를 안내합니다.

예시:

* `선택한 좌석 중 일부를 다른 사용자가 먼저 선택했습니다.`
* `좌석 정보를 다시 확인해 주세요.`
* `좌석 다시 선택`

### 27.2 Seat Hold 만료

Seat Hold가 만료되면 다음 내용을 안내합니다.

* 좌석 임시 확보 시간이 만료됨
* 기존 예약 진행을 계속할 수 없음
* 좌석을 다시 선택해야 함

### 27.3 외부 Flight API 장애

다음 내용을 안내합니다.

* 현재 실제 항공편 정보를 불러올 수 없음
* KOKU Airline 내부 항공편 예약 서비스는 정상적으로 이용 가능
* 다시 시도 Action 제공 가능

### 27.4 AI 응답 실패

다음 내용을 안내합니다.

* AI 추천 생성 실패
* 다시 시도 가능
* 일반 실제 항공편 검색으로 이동 가능

### 27.5 인증 필요

Guest가 인증이 필요한 기능을 사용하려는 경우 로그인 필요 안내를 표시합니다.

### 27.6 권한 없음

현재 Role에 허용되지 않은 기능인 경우 권한 없음 안내를 표시합니다.

### 27.7 지원하지 않는 노선

다음 기능은 한국 ↔ 일본 노선만 지원합니다.

* KOKU Airline 내부 Flight 검색
* 외부 실제 항공편 검색
* AI 항공편 검색

지원하지 않는 노선이 입력되면 외부 Flight API를 호출하지 않고 검색 조건 수정을 안내합니다.

---

## 28. 접근성 기본 원칙

MVP에서도 최소 다음 사항을 고려합니다.

* Form Input에 연결된 Label 제공
* Error Message를 해당 Input과 연결
* Keyboard를 통한 주요 기능 접근 고려
* 상태를 색상만으로 표현하지 않음
* Button과 Link의 역할 구분
* Disabled 상태 명확히 표현
* Focus 상태 확인 가능
* 한국어와 일본어 환경에서 Text가 잘리지 않도록 구성

---

## 29. 생성 방식 규칙

### 29.1 한 번에 전체 시스템을 만들지 않음

Figma Make는 요청받은 화면 단위로 생성합니다.

권장 생성 순서:

1. Home
2. Flight 검색 결과
3. Flight 상세
4. 로그인 / 회원가입
5. Passenger 정보 입력
6. Seat 선택
7. 예약 확인
8. Mock 결제
9. 예약 완료
10. Reservation 상세
11. My Page
12. 외부 실제 항공편
13. AI 항공편 검색
14. Admin 화면

KOKU Flight 검색 결과와 예약 화면을 생성할 때는
`ONE_WAY`와 `ROUND_TRIP`의 화면 상태를 모두 고려합니다.

특히 왕복은 다음 상태가 시각적으로 구분되어야 합니다.

```text
출국 Flight 선택
→ 귀국 Flight 선택
→ Passenger 입력
→ 출국 Seat 선택
→ 귀국 Seat 선택
→ 왕복 예약 확인
→ 왕복 전체 Mock Payment
```

편도 화면만 생성한 뒤 이를 그대로 왕복 화면으로 복제하지 않습니다.

### 29.2 기존 스타일 유지

새 화면을 추가할 때 기존 화면과 다음 기준을 일관되게 유지합니다.

* Global Header
* Typography
* Spacing
* Button / CTA
* Card
* Form
* 상태 표현
* 전체 색상 체계

각 화면을 서로 다른 웹사이트처럼 디자인하지 않습니다.

### 29.3 기본 흐름 우선

첫 시안에서는 정상적인 Happy Path를 우선 생성합니다.

모든 Error / Empty / Loading / Exception 상태를 한꺼번에 한 화면에 표현하지 않습니다.

필요한 상태는 이후 별도로 추가합니다.

---

## 30. Figma Make 금지 사항

Figma Make는 다음을 해서는 안 됩니다.

* 프로젝트 문서에 없는 비즈니스 정책 추가
* 내부 KOKU Flight와 외부 실제 항공편의 경계를 흐리기
* 외부 실제 항공편에 KOKU Airline 예약 기능 추가
* 실제 카드 결제 Form 추가
* 실제 발권 기능 추가
* AI 자동 예약 기능 추가
* AI 추천 설명을 실제 Flight 데이터처럼 표현
* Reservation 상태 임의 추가 또는 변경
* Payment 상태 임의 추가 또는 변경
* Seat 상태 임의 추가 또는 변경
* Passenger와 Seat 선택 순서 변경
* Seat Hold 시간을 임의 변경
* Desktop에서 제공하는 주요 Customer 기능을 Mobile에서 임의로 제거
* Mobile을 별도의 기능 축소 버전으로 생성
* Desktop Layout을 재배치 없이 단순 축소하여 Mobile UI로 사용
* Mobile Seat Map에서 실제 Row / Column 또는 Seat 인접 관계를 임의 변경
* Admin / SuperAdmin Mobile UI를 MVP 필수 범위로 임의 확대
* Tablet 전용 UI를 MVP 필수 범위로 임의 확대
* 실제 여권정보 입력 UI 생성
* Guest에게 인증 없이 AI 항공편 검색 기능 제공
* `Admin`에게 `SuperAdmin` 전용 기능 제공
* `ROUND_TRIP`을 출국 / 귀국 두 개의 독립 Reservation으로 임의 분리
* 왕복 Passenger 구성을 Flight별로 별도 입력하도록 변경
* 왕복 출국 / 귀국 Seat 확보에 Partial Success UI 추가
* 출국 Flight와 귀국 Flight를 별도로 결제하는 UI 생성
* 왕복 부분 취소 또는 부분 Mock 환불 기능 추가
* 귀국 Route를 출국 Route와 무관하게 선택하는 Multi-city UI 추가
* Flight별 Passenger 연령 Validation을 Reservation 전체 기준으로 임의 단순화

---

## 31. Figma Make 프롬프트 운용 원칙

처음부터 전체 서비스를 한 번에 생성하도록 요청하지 않습니다.

첫 요청은 Home 화면만 생성하는 것을 권장합니다.

Home 생성 프롬프트에는 최소 다음 내용을 포함합니다.

* `03-ui-design.md`와 `figma-make-guidelines.md`를 기준으로 생성
* Customer UI는 Desktop / Mobile Responsive Web 기준
* Desktop 기준 Width `1440px`
* Mobile 기준 Width `390px`
* Desktop과 Mobile에서 동일한 주요 기능 및 사용자 흐름 유지
* Guest Header 기준
* 한국어 기본 UI
* `한국어 | 日本語` 언어 전환 제공
* KOKU Airline 내부 Flight와 외부 실제 항공편 기능 명확히 구분
* AI 항공편 검색 영역 별도 구분
* 문서에 없는 정책 추가 금지
* Home 이외의 화면은 아직 생성하지 않음
* Hero 보조 문구는 `가상 항공편 예약과 실제 항공편 조회를 한 번에`
* KOKU 검색 Form에 `편도 / 왕복` 선택 제공
* 기본값은 `편도`
* 왕복 선택 시에만 도착일 표시
* `항공편 검색` Navigation에 Active underline 표현
* AI CTA는 `AI 항공편 추천 받기`
* KOKU Airline Logo는 깃발 내부에 Wing을 통합한 Minimal Emblem 방향
* Mobile Home은 Desktop Home을 단순 축소하지 않고
  1 Column Layout과 Mobile Navigation을 기준으로 재배치

Home 확정 이후 다음 화면을 생성할 때는 현재 디자인 스타일을 유지하도록 요청합니다.

---

## 32. 최종 원칙

Figma Make는 UI와 Prototype을 생성하는 도구이며 프로젝트의 비즈니스 정책을 결정하지 않습니다.

다음 원칙을 항상 유지합니다.

* UI는 프로젝트 문서를 반영합니다.
* 비즈니스 정책을 임의로 만들지 않습니다.
* 애매한 경우 단순하고 보수적으로 표현합니다.
* 기존 화면의 디자인 스타일을 유지합니다.
* 사용자 흐름의 순서를 변경하지 않습니다.
* 내부 KOKU Flight와 외부 실제 항공편을 명확히 구분합니다.
* AI 추천 설명과 실제 Flight 데이터를 명확히 구분합니다.
* Customer UI는 Desktop / Mobile Responsive Web 기준으로 완성합니다.
* Admin / SuperAdmin 관리 UI는 Desktop Web 사용성을 우선합니다.
* Desktop과 Mobile에서 동일한 주요 Customer 기능과 사용자 흐름을 유지합니다.