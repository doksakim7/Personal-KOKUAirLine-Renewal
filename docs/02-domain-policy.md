# KOKU Airline Renewal - Domain Policy

## 1. 문서 목적

본 문서는 KOKU Airline Renewal의 핵심 Domain Policy와 사용자 시나리오를 정의합니다.

구현 과정에서 Backend, Frontend 및 AI Agent가 비즈니스 규칙을 임의로 결정하지 않도록 공통 기준을 제공합니다.

본 문서에서 정의하는 핵심 영역은 다음과 같습니다.

- 사용자 및 인증
- 공항 및 노선
- 항공기 및 좌석
- 항공편 및 운항 일정
- 탑승객
- 예약
- Mock 결제
- 예약 취소
- 외부 실제 항공편 조회
- AI 항공편 검색
- 관리자

세부 데이터 구조는 `05-data-api-design.md`,
시스템 및 기술 구조는 `04-system-design.md`에서 정의합니다.

### 1.1 용어 표기 원칙

본 문서와 관련 설계 문서에서는 용어의 일관성을 위해 다음 표기 원칙을 사용합니다.

- 설명, 기능명 및 일반적인 비즈니스 개념은 한국어로 작성합니다.
- Role, Entity, 인증 객체, Enum 등 코드, ERD 및 API Contract와 직접 연결되는 개념은 영문 Canonical Term을 사용합니다.
- `Domain Policy`, `API Contract` 등 문서명 또는 명시적인 기술 개념은 해당 영문 명칭을 유지합니다.
- 일반적인 domain 개념은 `도메인`으로 표기합니다.
- 동일한 개념을 문서마다 서로 다른 용어로 혼용하지 않습니다.

주요 Canonical Term 예시는 다음과 같습니다.

- Role: `Guest`, `Member`, `Admin`, `SuperAdmin`
- 인증 객체: `AuthAccount`
- Entity: `Airport`, `Route`, `Aircraft`, `Flight`, `Seat`, `Passenger`, `Reservation`, `Payment`
- 상태 및 Enum: `ACTIVE`, `WITHDRAWN`, `PENDING`, `CONFIRMED`, `CANCELLED`,
  `AVAILABLE`, `HELD`, `RESERVED`, `UNAVAILABLE`,
  `SUCCESS`, `FAILED`, `REFUNDED`,
  `SCHEDULED`, `DEPARTED`,
  `ONE_WAY`, `ROUND_TRIP`

여행 유형은 `TripType`으로 구분하며 다음 값을 사용합니다.

- `ONE_WAY`: 편도
- `ROUND_TRIP`: 왕복

---

## 2. 핵심 도메인 구분 

KOKU Airline Renewal은 크게 두 종류의 항공편 데이터를 다룹니다.

### 2.1 KOKU Airline 내부 항공편

KOKU Airline이 운영한다고 가정한 가상의 항공편입니다.

내부 Database에서 관리하며 다음 기능을 제공합니다.

- 항공편 검색
- 좌석 조회
- 좌석 선택
- 예약
- Mock 결제
- 예약 조회
- 예약 취소

실제 항공사의 예약 시스템과 연결되지 않습니다.

### 2.2 외부 실제 항공편

외부 Flight API를 통해 조회하는 실제 항공편 데이터입니다.

다음 목적으로만 사용합니다.

- 실제 항공편 조회
- 조건 비교
- AI 기반 검색 및 추천

외부 실제 항공편에 대해서는 다음 기능을 제공하지 않습니다.

- 예약
- 좌석 확정
- 실제 결제
- 항공권 발권

### 2.3 데이터 분리 원칙

KOKU Airline 내부 예약 데이터와 외부 실제 항공편 데이터는 명확하게 구분합니다.

외부 API에서 조회한 실제 항공편을 KOKU Airline의 판매 가능한 내부 항공편으로 취급하지 않습니다.

---

## 3. 사용자 역할

### 3.1 Guest

로그인하지 않은 사용자입니다.

가능한 기능:

- KOKU Airline 항공편 검색
- 항공편 상세 조회
- 외부 실제 항공편 검색
- 회원가입
- 일반 로그인
- Google OAuth 로그인

제한:

- AI 항공편 검색 불가
- 예약 생성 불가
- Mock 결제 불가
- 개인 예약 내역 조회 불가

예약을 진행하려는 경우 인증을 요구합니다.

### 3.2 Member

정상적으로 인증된 일반 사용자입니다.

가능한 기능:

- Guest가 사용할 수 있는 모든 조회 기능
- AI 항공편 검색
- 좌석 선택
- 예약 생성
- Mock 결제
- 자신의 예약 조회
- 자신의 예약 취소
- 회원 정보 조회

Member는 다른 사용자의 예약 정보를 조회하거나 수정할 수 없습니다.

### 3.3 Admin

항공사 운영 업무를 담당하는 관리자입니다.

가능한 기능:

- 운영 데이터 조회
- 공항 조회
- 노선 조회
- 항공기 및 좌석 구성 조회
- 항공편 관리
- 운항 일정 관리
- 예약 현황 조회

제한:

- 고객 예약 강제 취소 불가
- 공항 및 노선 등 핵심 기준정보 생성·수정·비활성화 불가
- 항공기 및 좌석 구성 생성·수정·비활성화 불가
- 기타 중요 운영 데이터 변경 불가

핵심 운영 데이터의 변경은 SuperAdmin 권한을 요구합니다.

### 3.4 SuperAdmin

서비스의 핵심 운영 데이터를 관리하는 최고 관리자입니다.

가능한 기능:

- Admin이 사용할 수 있는 모든 기능
- 공항 Master Data 관리
- 노선 Master Data 관리
- 항공기 및 좌석 구성 관리
- 고객 예약 강제 취소
- 중요 운영 데이터 관리

SuperAdmin 권한은 서비스 운영상 중요한 데이터 변경에 사용하며,
일반 관리자보다 높은 권한을 가집니다.

---

## 4. Member 및 인증 정책

### 4.1 인증 방식

MVP에서는 다음 두 인증 방식을 지원합니다.

1. 일반 이메일/비밀번호 인증
2. Google OAuth 2.0 인증

두 인증 방식은 최종적으로 동일한 KOKU Airline Member 및 권한 체계를 사용합니다.

### 4.2 일반 회원가입

일반 회원가입 시 최소한 다음 정보를 사용합니다.

- Email
- Password

세부 회원정보는 데이터 모델 설계 과정에서 확정합니다.

### 4.3 이메일 정책

서비스에서 Email을 저장하거나 비교하기 전에
다음 정규화 규칙을 적용합니다.

1. Email 앞뒤의 공백을 제거합니다.
2. Email 전체를 영문 소문자로 변환합니다.
3. 정규화된 Email을 기준으로 형식을 검증합니다.
4. 정규화된 Email을 기준으로 중복 여부를 검증합니다.
5. Database에는 정규화된 Email을 저장합니다.

예:

```text
 User@Test.com
 user@test.com
 USER@TEST.COM
 ```

 위 Email은 모두 다음과 동일한 Email로 취급합니다.

```text
 user@test.com
  ```

일반 회원가입, 일반 로그인 및 Google OAuth 계정 연동에서
Email을 비교할 때 동일한 정규화 규칙을 적용합니다.

Email의 원본 대소문자 형태는 별도로 보존하지 않습니다.



### 4.4 비밀번호 정책

일반 회원가입 및 비밀번호 변경 시 다음 조건을 모두 만족해야 합니다.

* 비밀번호는 최소 8자 이상이어야 합니다.
* 영문 대문자를 최소 1자 이상 포함해야 합니다.
* 영문 소문자를 최소 1자 이상 포함해야 합니다.
* 숫자를 최소 1자 이상 포함해야 합니다.
* 특수문자를 최소 1자 이상 포함해야 합니다.
* 사용할 수 있는 특수문자는 다음 문자로 제한합니다.
  * `! @ # $ % ^ & *`
* 비밀번호는 평문으로 저장하지 않습니다.
* Spring Security `PasswordEncoder`를 사용하여 단방향 Hash 형태로 저장합니다.

비밀번호 정책은 Backend에서 반드시 검증하며, Frontend에서도 동일한 조건을 사용자에게 안내하고 입력 단계에서 검증합니다.

### 4.5 Google OAuth

MVP 소셜 로그인 Provider는 Google 하나로 제한합니다.

Google OAuth 인증 성공 후:

1. Google로부터 인증된 사용자 정보를 전달받습니다.
2. 기존 GOOGLE AuthAccount가 존재하는지 확인합니다.
3. 기존 GOOGLE AuthAccount가 존재하는 경우 연결된 Member의 상태를 확인합니다.
   - `ACTIVE` 상태이면 해당 Member로 인증합니다.
   - `WITHDRAWN` 상태이면 인증을 거부합니다.
4. 기존 GOOGLE AuthAccount가 없는 경우 동일 Email Member 존재 여부를 확인합니다.
5. 동일 Email Member가 없는 신규 사용자인 경우 Member를 생성하고 GOOGLE AuthAccount를 연결합니다.
6. 동일 Email Member가 존재하는 경우 4.6.1 동일 Email 계정 연동 정책을 따릅니다.
7. 이후 KOKU Airline 내부 인증 체계를 사용합니다.

OAuth Client Secret 등의 민감정보는 Repository에 저장하지 않습니다.

### 4.6 Member와 AuthAccount 관계

Member와 AuthAccount는 역할을 분리하여 관리합니다.

- `Member`: 서비스에서 사용하는 회원 정보와 권한을 관리합니다.
- `AuthAccount`: 일반 로그인 또는 외부 OAuth 등 회원의 인증 수단을 관리합니다.

하나의 Member는 하나 이상의 AuthAccount를 가질 수 있도록 설계합니다.

MVP에서는 다음 인증 Provider를 지원합니다.

- `LOCAL`
- `GOOGLE`

일반 로그인 비밀번호 Hash와 Google OAuth 식별정보 등 인증에 종속적인 정보는
Member가 아니라 AuthAccount에서 관리합니다.

#### 4.6.1 동일 Email 계정 연동

Google OAuth 인증에 성공한 사용자의 검증된 Email이
기존 Member의 Email과 동일한 경우 기존 Member와의 계정 연동 여부를 확인합니다.

기존 Member에 `LOCAL` AuthAccount만 존재하는 경우
Email 일치만으로 `GOOGLE` AuthAccount를 자동 연결하지 않습니다.

사용자가 기존 `LOCAL` AuthAccount의 비밀번호로 재인증에 성공한 경우에만
해당 Member에 `GOOGLE` AuthAccount를 연결합니다.

Email 비교 시 4.3 이메일 정책에서 정의한 정규화 규칙을 적용합니다.

이미 해당 Google AuthAccount가 다른 Member와 연결되어 있는 경우
자동으로 계정을 병합하지 않습니다.

`WITHDRAWN` 상태의 Member는 자동으로 재활성화하거나
새로운 AuthAccount를 연결하지 않습니다.

### 4.7 권한

서비스의 사용자 역할은 `Guest`, `Member`, `Admin`, `SuperAdmin`으로 구분합니다.

인증된 사용자의 Spring Security 권한 코드는 다음과 같이 관리합니다.

- `USER`
- `ADMIN`
- `SUPERADMIN`

권한 관계는 다음과 같습니다.

- `USER`: Member 기능을 사용합니다.
- `ADMIN`: 항공편 운영 및 예약 현황 조회 등 Admin 기능을 사용합니다.
- `SUPERADMIN`: Admin 권한을 포함하며, 고객 예약 강제 취소 및 핵심 운영 데이터 수정 권한을 가집니다.

`Guest`는 인증되지 않은 사용자이므로 별도의 인증 권한 코드를 가지지 않습니다.

구체적인 Spring Security 권한 구조는 시스템 설계 문서에서 정의합니다.

### 4.8 회원 상태 및 탈퇴

MVP에서 회원 상태는 다음과 같이 정의합니다.

- `ACTIVE`
- `WITHDRAWN`

#### ACTIVE

정상적으로 로그인하고 서비스를 이용할 수 있는 Member입니다.

#### WITHDRAWN

회원 탈퇴가 완료된 상태입니다.

회원 탈퇴 시 Member 데이터를 물리적으로 삭제하지 않고
상태를 `WITHDRAWN`으로 변경합니다.

`WITHDRAWN` 상태의 Member는 다음 기능을 사용할 수 없습니다.

- 로그인
- 새로운 예약 생성
- AI 항공편 검색
- 기타 인증이 필요한 서비스 기능

회원 탈퇴 이후에도 기존 예약 및 결제 이력은 데이터 정합성과 과거 이력 보존을 위해 유지합니다.

진행 중인 예약 또는 향후 탑승 예정인 확정 예약을 보유한 Member는 탈퇴할 수 없습니다.

회원 탈퇴를 진행하려면 먼저 다음 예약이 존재하지 않아야 합니다.

- `PENDING` 예약
- 예약에 포함된 Flight 중 아직 출발하지 않은 Flight가 하나 이상 존재하는 `CONFIRMED` 예약

취소 가능한 예약은 Member가 먼저 취소한 후 탈퇴할 수 있습니다.

구체적인 탈퇴 가능 여부 판단 기준은 Reservation 상태와
Reservation에 포함된 Flight 중 아직 출발하지 않은 Flight의 존재 여부를 기준으로 합니다.

---

## 5. 공항 정책

### 5.1 MVP 지원 공항

#### 한국

- ICN - 인천국제공항
- GMP - 김포국제공항
- PUS - 김해국제공항
- CJU - 제주국제공항

#### 일본

- NRT - 나리타국제공항
- HND - 하네다공항
- KIX - 간사이국제공항
- FUK - 후쿠오카공항
- CTS - 신치토세공항
- NGO - 주부국제공항

### 5.2 공항 식별

공항은 IATA Code를 통해 식별할 수 있어야 합니다.

예:

- ICN
- HND
- NRT

### 5.3 공항 관리

공항 정보는 Admin과 SuperAdmin 모두 조회할 수 있습니다.

공항 정보의 생성, 수정 및 비활성화는 SuperAdmin만 수행할 수 있습니다.

공항 데이터는 과거 항공편 및 예약 이력과의 참조 관계를 보존하기 위해
Hard Delete하지 않고 Soft Delete 또는 비활성 상태로 관리합니다.

비활성화된 공항은 신규 노선 및 항공편 생성에 사용할 수 없습니다.

---

## 6. 노선 정책

### 6.1 노선 정의

노선(Route)은 출발 공항과 도착 공항의 조합을 의미합니다.

예:

`ICN → NRT`

### 6.2 MVP 노선 범위

MVP에서는 한국 ↔ 일본 국제 노선만 허용합니다.

허용:

- 한국 → 일본
- 일본 → 한국

허용하지 않음:

- 한국 → 한국
- 일본 → 일본
- 한국/일본 이외 국가와 연결되는 노선

### 6.3 동일 공항

출발 공항과 도착 공항은 동일할 수 없습니다.

### 6.4 노선 중복

동일한 출발 공항과 도착 공항 조합의 중복 Route 생성을 허용하지 않습니다.

### 6.5 노선 관리

노선 정보의 생성, 수정 및 비활성화는 SuperAdmin만 수행할 수 있습니다.

과거 항공편 및 예약에서 사용된 노선은 물리적으로 삭제하지 않습니다.

비활성화된 노선에는 새로운 항공편 또는 운항 일정을 생성할 수 없습니다.

### 6.6 왕복 Route 정책

`ROUND_TRIP`에서는 출국 Route와 귀국 Route를 함께 사용합니다.

귀국 Route는 출국 Route의 정확한 역방향이어야 합니다.

예:

```text
출국: ICN → NRT
귀국: NRT → ICN
```

다음과 같이 출국 Route와 무관한 귀국 Route는
`ROUND_TRIP`으로 허용하지 않습니다.

```text
출국: ICN → NRT
귀국: KIX → ICN
```

이와 같은 형태는 다구간(Multi-city) 여행에 해당하며
MVP 범위에 포함하지 않습니다.

---

## 7. 항공기 정책

### 7.1 항공기

항공기(Aircraft)는 KOKU Airline이 운영하는 가상의 항공기를 의미합니다.

항공기는 최소한 다음 정보를 가집니다.

- 항공기 식별정보
- 기종
- 좌석 구성
- 운영 상태

구체적인 데이터 구조는 ERD에서 정의합니다.

### 7.2 항공기와 좌석

각 항공기는 고유한 좌석 배치를 가집니다.

예:

- 1A
- 1B
- 2A
- 2B

좌석 구성은 항공편 예약 좌석을 생성하는 기준이 됩니다.

### 7.3 항공기 및 좌석 구성 관리

항공기와 좌석 구성의 생성, 수정 및 비활성화는 SuperAdmin만 수행할 수 있습니다.

과거 항공편 또는 예약에서 사용된 항공기와 좌석 구성은 물리적으로 삭제하지 않습니다.

비활성화된 항공기는 새로운 항공편 또는 운항 일정에 배정할 수 없습니다.

기존 항공편 및 예약에서 참조하고 있는 좌석 구성은 과거 이력 보존을 위해 유지합니다.

---

## 8. 좌석 정책

### 8.1 좌석 식별

하나의 항공편 내에서 좌석 번호는 유일해야 합니다.

### 8.2 기본 좌석 상태

예약 관점의 좌석은 다음 상태를 구분합니다.

- `AVAILABLE`: 예약 가능
- `HELD`: 예약 진행 중 임시 점유
- `RESERVED`: 예약 확정
- `UNAVAILABLE`: 사용 불가

사용자가 Seat가 필요한 모든 Passenger의 Seat 선택을 완료하고
예약 시작에 성공하면 선택한 모든 Seat를 `HELD` 상태로 전환합니다.

`HELD` 상태의 좌석은 다른 사용자가 선택하거나 예약할 수 없습니다.

### 8.3 좌석 중복 예약

동일 항공편의 동일 좌석은 동시에 두 개 이상의 확정 예약에 포함될 수 없습니다.

해당 규칙은 Application 수준뿐 아니라 가능한 경우 Database 제약조건 및 동시성 제어를 통해 함께 보호합니다.

### 8.4 동시성

두 명 이상의 사용자가 동일 좌석을 동시에 예약하는 상황이 발생할 수 있습니다.

최종적으로 하나의 예약만 성공해야 합니다.

구체적인 동시성 제어 방식은 M4에서 테스트 및 성능 측정을 통해 결정합니다.

### 8.5 좌석 임시 점유

사용자가 Seat가 필요한 모든 Passenger의 Seat 선택을 완료하고
예약 시작에 성공하면 선택한 모든 Seat를 일정 시간 임시 점유합니다.

좌석 Hold 시간은 MVP 기준 **1시간**으로 설정합니다.

좌석이 `HELD` 상태인 동안에는 다른 사용자가 해당 좌석을 선택하거나 예약할 수 없습니다.

Frontend에서는 사용자가 남은 Hold 시간을 확인할 수 있도록 남은 시간을 표시합니다.

Hold 시간 내에 Mock 결제가 성공하면:

`HELD → RESERVED`

Hold 시간이 만료되면:

`HELD → AVAILABLE`

Hold 시간이 만료될 때까지 결제가 완료되지 않은 예약은 `CANCELLED` 상태로 전환하고 좌석을 반환합니다.

결제 실패가 발생한 경우 결제 시도 횟수가 최대 허용 횟수에 도달하지 않았고
Hold 시간이 남아 있다면 사용자는 결제를 다시 시도할 수 있습니다.

MVP에서 결제 시도는 최초 시도를 포함하여 최대 3회까지 허용합니다.

Hold 만료 및 좌석 반환은 Backend를 기준으로 처리하며,
Frontend의 남은 시간 표시는 사용자 안내 목적으로 사용합니다.

`ROUND_TRIP` Reservation에서는 출국 Flight와 귀국 Flight에서
선택한 모든 Seat를 하나의 Reservation 시작 단위로 처리합니다.

출국 및 귀국 Flight에서 선택한 모든 Seat를 확보할 수 있는 경우에만
Reservation을 `PENDING` 상태로 시작합니다.

두 Flight 중 하나의 Seat라도 확보하지 못한 경우
Reservation 시작 전체를 실패 처리합니다.

이 경우:

- `PENDING` Reservation을 정상 생성된 상태로 남기지 않습니다.
- 출국 또는 귀국 Flight의 일부 Seat만 `HELD` 상태로 남기지 않습니다.
- Flight 단위 또는 Seat 단위 Partial Success를 허용하지 않습니다.

즉, 왕복 Reservation에서도 전체 Seat 확보 성공 또는 전체 실패
(All-or-Nothing)를 원칙으로 합니다.

---

## 9. 항공편 및 운항 일정 정책

### 9.1 항공편

KOKU Airline의 항공편은 특정 노선과 운항 정보를 가집니다.

항공편은 최소한 다음 정보를 표현할 수 있어야 합니다.

- 편명
- 출발 공항
- 도착 공항
- 출발 시간
- 도착 시간
- 운항 항공기
- 운항 상태

#### 9.1.1 Flight 운항 상태

MVP에서 Flight의 운항 상태는 다음과 같이 정의합니다.

- `SCHEDULED`: 출발 예정 상태
- `CANCELLED`: 출발 전에 운항이 취소된 상태
- `DEPARTED`: 출발 예정 시각이 지나 출발한 것으로 처리되는 상태

##### SCHEDULED

정상적으로 운항 예정인 Flight입니다.

예약 가능 시간 정책을 만족하는 경우 새로운 Reservation을 생성할 수 있습니다.

##### CANCELLED

Admin 또는 SuperAdmin에 의해 출발 전에 운항이 취소된 Flight입니다.

새로운 Reservation을 생성할 수 없습니다.

Flight 취소와 연결된 Reservation 처리는 9.6 Flight 취소 정책을 따릅니다.

##### DEPARTED

출발 예정 시각이 지난 Flight입니다.

새로운 Reservation을 생성하거나 Member가 예약을 취소할 수 없습니다.

MVP에서는 실제 운항 시스템과 연동하지 않으므로
`DEPARTED` 전환의 구체적인 처리 방식은 시스템 설계에서 정의합니다.

### 9.2 편명

KOKU Airline의 편명은 다음 형식을 사용합니다.

`KO + 3자리 숫자`

예:

- `KO101`
- `KO205`
- `KO318`

`KO`는 KOKU Airline을 나타내는 고정 Prefix입니다.

숫자 영역은 MVP 기준 `100 ~ 999` 범위를 사용합니다.

편명은 KOKU Airline의 운항편을 식별하기 위한 번호이며,
같은 편명이 서로 다른 운항일에 반복해서 사용될 수 있습니다.

동일한 날짜 및 운항 기준에서 동일한 편명이 중복되지 않도록 관리합니다.

편명 번호 생성 및 중복 검증의 구체적인 구현 방식은 데이터 및 시스템 설계에서 정의합니다.

### 9.3 운항 시간

- 출발 시간은 도착 시간보다 이전이어야 합니다.
- 날짜 및 시간은 시간대(Time Zone)를 고려할 수 있는 구조로 설계합니다.
- 실제 저장 방식은 시스템 및 데이터 설계에서 확정합니다.

### 9.4 항공기 배정

하나의 운항 일정에는 하나의 항공기가 배정됩니다.

같은 시간대에 동일 항공기를 복수의 운항 일정에 배정하지 않도록 검증이 필요합니다.

세부 충돌 판단 규칙은 추후 정의합니다.

### 9.5 예약 가능 시간

KOKU Airline 항공편의 신규 예약은
항공편 출발 예정 시각 기준 **2시간 전까지** 시작할 수 있습니다.

출발 예정 시각까지 2시간 미만이 남은 항공편에 대해서는
새로운 Reservation을 생성할 수 없습니다.

예약 가능 여부는 Backend 시간을 기준으로 판단합니다.

이미 생성된 `PENDING` Reservation의 Seat Hold는 기존 Hold 정책을 따르며,
Hold 만료 시각이 항공편 출발 예정 시각을 초과하지 않도록 합니다.

`ROUND_TRIP` Reservation을 새로 시작하려면
출국 Flight와 귀국 Flight 모두 예약 가능 조건을 만족해야 합니다.

두 Flight 중 하나라도 다음 조건을 만족하지 않으면
왕복 Reservation을 시작할 수 없습니다.

- `SCHEDULED` 상태가 아님
- 해당 Flight 출발 예정 시각까지 2시간 미만 남음

왕복 Reservation의 Seat Hold 만료 시각은
예약에 포함된 Flight 중 가장 먼저 출발하는 Flight의
출발 예정 시각을 초과할 수 없습니다.

#### 9.5.1 Flight 수정 제한

Reservation이 존재하지 않는 `SCHEDULED` Flight는
Admin 또는 SuperAdmin이 운영 정보를 수정할 수 있습니다.

`PENDING` 또는 `CONFIRMED` Reservation이 하나 이상 존재하는 Flight는
예약 및 Seat 정합성에 영향을 줄 수 있는 다음 핵심 정보를 직접 변경하지 않습니다.

- 출발 Airport
- 도착 Airport
- 출발 예정 시각
- 도착 예정 시각
- Aircraft
- 편명

예약이 존재하는 Flight의 운항을 더 이상 유지할 수 없는 경우에는
기존 Flight 정보를 직접 변경하기보다
9.6 Flight 취소 정책을 적용하는 것을 원칙으로 합니다.

구체적인 수정 가능 Field와 API 제약은 API Contract에서 정의합니다.

### 9.6 Flight 취소

Admin 또는 SuperAdmin은 운영상 필요한 경우
출발 전 Flight를 취소할 수 있습니다.

Flight 취소 시 취소 사유 입력을 필수로 합니다.

`SCHEDULED` 상태이며 아직 출발하지 않은 Flight만 취소할 수 있습니다.

Flight 취소가 정상적으로 처리되면:

- Flight: `SCHEDULED → CANCELLED`

다음 Reservation 처리 규칙은 `ONE_WAY` Reservation의 기본 처리 기준입니다.

`ROUND_TRIP` Reservation에 연결된 Flight 취소는
9.6.1 왕복 Reservation과 Flight 취소 정책을 우선 적용합니다.

취소된 Flight와 연결된 Reservation은 더 이상 유효한 예약 상태로 유지하지 않고
정책에 따라 `CANCELLED` 상태로 전환합니다.

`CONFIRMED` Reservation의 경우:

- Reservation: `CONFIRMED → CANCELLED`
- 성공한 Payment: `SUCCESS → REFUNDED`
- 해당 Reservation의 모든 Seat: `RESERVED → AVAILABLE`

`PENDING` Reservation의 경우:

- Reservation: `PENDING → CANCELLED`
- 진행 중인 Payment가 존재하는 경우: `PENDING → CANCELLED`
- 해당 Reservation의 모든 Seat: `HELD → AVAILABLE`

기존 `FAILED` Payment는 상태를 변경하지 않고 결제 이력으로 유지합니다.

MVP에서는 Flight 취소로 인한 결제 금액을 전액 Mock 환불하며,
대체편 제공 및 자동 재예약은 지원하지 않습니다.

#### 9.6.1 왕복 Reservation과 Flight 취소

`ROUND_TRIP` Reservation에 포함된 출국 Flight 또는 귀국 Flight가 취소되면
해당 Reservation은 더 이상 정상적인 왕복 여정을 제공할 수 없으므로
Reservation 전체를 `CANCELLED` 상태로 전환합니다.

아직 어느 Flight도 출발하지 않은 경우:

- Reservation: `CONFIRMED → CANCELLED`
- 성공한 Payment: `SUCCESS → REFUNDED`
- 출국 및 귀국 Flight의 모든 예약 Seat: `RESERVED → AVAILABLE`

`PENDING` Reservation인 경우:

- Reservation: `PENDING → CANCELLED`
- 진행 중인 Payment가 존재하면 `PENDING → CANCELLED`
- 출국 및 귀국 Flight의 모든 `HELD` Seat → `AVAILABLE`
- 기존 `FAILED` Payment는 결제 이력으로 유지

MVP에서는 왕복 Reservation에 대해서도 부분 환불을 지원하지 않으므로
Mock 결제 금액은 전체 금액을 환불합니다.

출국 Flight가 이미 `DEPARTED` 상태이고
귀국 Flight가 이후 취소된 경우에도
Reservation은 `CANCELLED` 상태로 전환하고
성공한 Mock Payment는 전액 `REFUNDED` 처리합니다.

이미 출발한 Flight의 Seat 상태 및 과거 탑승 이력은 변경하지 않으며,
아직 출발하지 않은 Flight의 예약 Seat만 `AVAILABLE` 상태로 반환합니다.

MVP에서는 Flight 취소로 인한 대체편 제공,
자동 재예약 및 부분 환불을 지원하지 않습니다.

Flight 취소 시 처리한 Admin 또는 SuperAdmin,
대상 Flight, 처리 시각 및 취소 사유를 Audit Log로 기록합니다.

### 9.7 KOKU Airline 운임 정책

KOKU Airline은 포트폴리오용 가상 항공사이므로
MVP에서는 실시간 수요 기반 Dynamic Pricing을 적용하지 않습니다.

각 Flight의 운임은 Reservation 내 여정 역할(출국편 / 귀국편),
출발 시간대 및 출발 요일을 기준으로
사전에 정의된 고정 규칙에 따라 산정합니다.

#### 9.7.1 기준 운임

MVP의 기본 Flight 운임은 200,000원으로 합니다.

#### 9.7.2 출발 시간대 구분

KOKU Airline의 고정 운임 계산에 사용하는 출발 시간대는
Flight의 출발 예정 시각을 기준으로 다음과 같이 구분합니다.

- 새벽: 00:00 ~ 05:59
- 오전: 06:00 ~ 11:59
- 오후: 12:00 ~ 17:59
- 저녁: 18:00 ~ 23:59

시간대는 Flight의 출발 Airport 현지 시간을 기준으로 판단합니다.

한국과 일본은 현재 동일한 UTC+9 시간대를 사용하지만,
운임 정책은 특정 국가의 시간대에 종속되지 않도록
각 Flight 출발지의 Local Time을 기준으로 적용합니다.

#### 9.7.3 출국형 운임

출국형 운임은 여행을 시작하는 Flight에 적용합니다.

`ROUND_TRIP` Reservation에서는 첫 번째 Flight인 출국 Flight에
출국형 운임 정책을 적용합니다.

출발 국가와 관계없이 동일한 규칙을 적용합니다.

예:

- 한국 출발 왕복: 한국 → 일본 Flight
- 일본 출발 왕복: 일본 → 한국 Flight

`ONE_WAY` Reservation은 귀국 Flight가 존재하지 않으므로
해당 Flight에 출국형 운임 정책을 적용합니다.

출발 시간대:

- 새벽: 기준 운임
- 오전: 기준 운임 + 40,000원
- 오후: 기준 운임 + 20,000원
- 저녁: 기준 운임

출발 요일:

- 금요일, 토요일: +30,000원
- 일요일 ~ 목요일: 추가 요금 없음

시간대와 요일 할증은 함께 적용할 수 있습니다.

출국형 운임은 여행을 시작하기 좋은 시간대와
주말 여행 수요를 단순화하여 표현한 Mock 운임 정책입니다.

#### 9.7.4 귀국형 운임

귀국형 운임은 `ROUND_TRIP` Reservation의 두 번째 Flight인
귀국 Flight에 적용합니다.

귀국 Flight는 출국 Flight Route의 정확한 역방향이며,
출발 국가와 관계없이 귀국형 운임 정책을 적용합니다.

예:

- 한국 출발 왕복
  - 출국: 한국 → 일본
  - 귀국: 일본 → 한국

- 일본 출발 왕복
  - 출국: 일본 → 한국
  - 귀국: 한국 → 일본

출발 시간대:

- 새벽: 기준 운임
- 오전: 기준 운임
- 오후: 기준 운임 + 20,000원
- 저녁: 기준 운임 + 40,000원

출발 요일:

- 일요일: +30,000원
- 월요일 ~ 토요일: 추가 요금 없음

시간대와 요일 할증은 함께 적용할 수 있습니다.

귀국형 운임은 여행 마지막 날 늦은 시간까지 체류하려는 수요와
일요일 귀국 수요를 단순화하여 표현한 Mock 운임 정책입니다.

#### 9.7.5 Passenger별 운임

Passenger별 운임은 각 Flight에서 계산된 Passenger의 연령 구분을 기준으로 합니다.

- Adult: Flight 운임의 100%
- Child: Flight 운임의 60%
- Infant: 0원

ROUND_TRIP Reservation에서는 출국 Flight와 귀국 Flight의
Passenger별 운임을 각각 계산한 후 합산합니다.

#### 9.7.6 Reservation 운임 확정

항공편 검색 화면에서는 해당 Flight의 현재 고정 운임을 표시합니다.

Reservation에 포함된 모든 Seat 확보에 성공하여
PENDING Reservation이 생성되는 시점에
Reservation의 최종 결제 예정 금액을 확정합니다.

확정된 금액은 Reservation에 보존하며
이후 결제 재시도 과정에서는 변경하지 않습니다.

#### 9.7.7 Dynamic Pricing

MVP에서는 다음 요소에 따른 실시간 운임 변경을 적용하지 않습니다.

- 잔여 Seat 수
- Seat 점유율
- 예약 수요
- 검색량
- 예약 시점에 따른 실시간 가격 변경

잔여 Seat 비율 및 수요를 기반으로 운임을 조정하는
Dynamic Pricing은 향후 개선사항으로 남깁니다.

---

## 10. 탑승객 정책

### 10.1 Member와 Passenger

예약자와 실제 탑승객은 동일하지 않을 수 있습니다.

따라서 Member와 Passenger는 별개의 개념으로 취급합니다.

예:

한 명의 Member가 가족 3명의 항공편을 예약할 수 있습니다.

### 10.2 탑승객 정보

예약 시 Passenger별로 다음 테스트용 기본 정보를 입력합니다.

- 테스트용 영문 성 (여권 영문명 형식)
- 테스트용 영문 이름 (여권 영문명 형식)
- 생년월일
- 성별
- 국적

구체적인 Database Column 및 Validation 규칙은 데이터 모델 설계에서 정의합니다.

### 10.3 테스트용 여권 정보

KOKU Airline은 실제 항공권을 발권하지 않는 포트폴리오용 가상 서비스이므로
실제 여권 정보를 입력받지 않습니다.

Passenger의 기본 정보 입력이 완료되면
예약 검증에 사용할 테스트용 여권 정보를 시스템에서 자동으로 생성합니다.

자동 생성 항목은 다음과 같습니다.

- 여권번호: 테스트용 여권번호 자동 생성
- 여권 발급국: Passenger가 입력한 국적과 동일한 국가로 자동 설정
- 여권 만료일: 테스트용 여권정보 생성 시점 기준 5년 뒤 날짜로 자동 설정

자동 생성된 테스트용 여권 정보는 사용자가 직접 입력하거나 수정하지 않습니다.

생성된 테스트용 여권 만료일은
해당 Passenger가 Reservation을 통해 탑승할 모든 Flight의 출발일 이후여야 합니다.

`ONE_WAY`에서는 출국 Flight의 탑승일을 기준으로 합니다.

`ROUND_TRIP`에서는 출국 Flight와 귀국 Flight 모두의
탑승일 조건을 만족해야 합니다.

하나의 Flight라도 테스트용 여권 유효기간 조건을 만족하지 못하면
Reservation을 `CONFIRMED` 상태로 확정할 수 없습니다.

테스트용 여권 정보는 Passenger에 귀속됩니다.

Reservation을 `CONFIRMED` 상태로 확정하기 전에
필요한 테스트용 여권 정보가 모두 생성되어 있어야 합니다.

테스트용 여권번호는 실제 개인정보가 아니더라도 Log에 출력하지 않는 것을 기본으로 합니다.

구체적인 생성 규칙, 저장 위치 및 보호 방식은
데이터 및 시스템 설계에서 정의합니다.

### 10.4 테스트용 개인정보 정책

본 프로젝트는 실제 항공권 발권 서비스를 제공하지 않는 포트폴리오용 가상 서비스입니다.

사용자는 실제 탑승객의 개인정보 및 실제 여권 정보를 입력하지 않는 것을 원칙으로 합니다.

예약 UI에서는 테스트용 Passenger 정보 사용을 명확하게 안내합니다.

여권번호, 여권 발급국, 여권 만료일은
10.3 정책에 따라 시스템에서 테스트용 데이터로 자동 생성합니다.

프로젝트의 기능 검증 목적상 여권 형식과 유효성 검증 로직은 구현하되,
실제 개인정보 수집을 목적으로 하지 않습니다.

### 10.5 연령 구분

탑승객의 연령은 항공편 **탑승일을 기준**으로 계산합니다.

탑승객은 다음과 같이 구분합니다.

* **유아(Infant)**: 생후 7일 이상 ~ 만 2세 미만
* **소아(Child)**: 만 2세 이상 ~ 만 12세 미만
* **성인(Adult)**: 만 12세 이상

예약 시 탑승객의 생년월일과 항공편 탑승일을 기준으로 연령 구분을 자동으로 판단합니다.

생후 7일 미만의 탑승객은 예약할 수 없습니다.

연령 구분은 탑승객이 직접 선택하는 값이 아니라, 입력된 생년월일을 기준으로 시스템에서 계산하는 것을 원칙으로 합니다.

`ROUND_TRIP`에서는 Passenger의 연령 구분을
출국 Flight와 귀국 Flight의 탑승일을 기준으로 각각 계산합니다.

따라서 동일 Passenger라도 출국 Flight와 귀국 Flight에서
연령 구분이 달라질 수 있습니다.

예를 들어 출국 시 `Infant`였던 Passenger가
귀국일에는 `Child`가 될 수 있습니다.

Seat 필요 여부와 Adult 동반 Validation은
각 Flight에서 계산된 Passenger의 연령 구분을 기준으로 적용합니다.

### 10.6 Child 및 Infant 동반 정책

Child 및 Infant 관련 정책은 각 Flight의 탑승일을 기준으로
계산된 Passenger 연령 구분에 따라 적용합니다.

해당 Flight에서 Child인 Passenger가 포함된 경우
동일 Flight에 최소 1명 이상의 Adult가 함께 탑승해야 합니다.

Child는 동일 Flight에서 함께 탑승하는 Adult 중 최소 1명과
인접한 Seat를 배정받아야 합니다.

MVP에서 인접한 Seat는 동일 Row에서 좌우로 직접 연결되어 있으며,
두 Seat 사이에 통로가 존재하지 않는 Seat를 의미합니다.

해당 Flight에서 Infant인 Passenger는 독립된 Seat를 사용하지 않습니다.

Infant는 해당 Flight에서 Adult인 Passenger와 연결되어야 하며,
Adult 1명당 최대 1명의 Infant를 동반할 수 있습니다.

해당 Flight에서 Infant만으로 탑승 구성을 만들 수 없습니다.

`ROUND_TRIP`에서는 위 Validation을
출국 Flight와 귀국 Flight 각각에 독립적으로 적용합니다.

Infant의 동반 Adult는 Reservation의 Passenger 중에서 지정합니다.

`ROUND_TRIP`에서 동일 Passenger가 하나 이상의 Flight에서
`Infant`로 분류되는 경우 동일한 동반 Adult를 기본으로 사용합니다.

지정된 동반 Adult는 해당 Infant가 `Infant`로 분류되는
모든 Flight에서 `Adult` 조건을 만족해야 합니다.

어느 Flight에서든 지정된 동반 Passenger가 `Adult` 조건을
만족하지 못하면 Reservation을 시작할 수 없습니다.

구체적인 Passenger 간 동반 관계의 저장 구조는
`05-data-api-design.md`에서 정의합니다.

---

## 11. 예약 정책

### 11.1 여행 유형 및 Reservation의 Flight 구성

MVP에서 Reservation의 여행 유형은 `TripType`으로 구분합니다.

- `ONE_WAY`
- `ROUND_TRIP`

#### ONE_WAY

`ONE_WAY` Reservation은 하나의 KOKU Airline Flight를 포함합니다.

#### ROUND_TRIP

`ROUND_TRIP` Reservation은 다음 두 KOKU Airline Flight를 하나의 예약 단위로 포함합니다.

1. 출국 Flight
2. 귀국 Flight

귀국 Flight의 Route는 출국 Flight Route의 역방향이어야 합니다.

귀국 Flight의 출발 예정 시각은
출국 Flight의 도착 예정 시각보다 이후여야 합니다.

MVP에서는 귀국 Flight의 출발 Date가
출국 Flight의 출발 Date보다 이후여야 합니다.

따라서 출국보다 먼저 출발하는 귀국 Flight나
동일 Date의 왕복 일정은 허용하지 않습니다.

`ROUND_TRIP`에서도 하나의 Reservation으로 처리하며,
출국 Flight와 귀국 Flight를 별개의 독립 Reservation으로 생성하지 않습니다.

동일한 Passenger 구성을 출국 Flight와 귀국 Flight에 공통으로 적용합니다.

단, Passenger의 연령 구분과 Seat 배정은
각 Flight의 탑승일 및 Seat 구성에 따라 Flight별로 판단합니다.

구체적인 Reservation과 Flight의 Database 관계 및 Mapping 방식은
`05-data-api-design.md`에서 정의합니다.

### 11.2 예약 생성 및 확정 조건

#### PENDING 예약 생성

예약 진행을 시작하려면 최소한 다음 조건을 만족해야 합니다.

- `ACTIVE` 상태의 인증된 Member
- Reservation에 포함되는 모든 KOKU Airline Flight가 유효한 `SCHEDULED` 상태
- Reservation에 포함되는 모든 Flight가 예약 가능 시간 조건을 만족
- Passenger 정보 입력 완료
- Reservation에 포함된 각 Flight를 기준으로
  Passenger의 연령 및 Child / Infant 동반 조건 충족
- Reservation에 포함된 각 Flight에서 Seat가 필요한 모든 Passenger의 Seat 선택 완료
- 출국 / 귀국을 포함하여 선택한 모든 Seat가 `AVAILABLE` 상태

모든 조건을 만족하고 Reservation에 포함된 모든 Flight의
Seat 확보에 성공한 경우에만 `PENDING` Reservation을 생성합니다.

선택한 모든 Seat는 `AVAILABLE → HELD` 상태로 전환합니다.

`ROUND_TRIP`에서는 출국 Flight와 귀국 Flight의 Seat 확보를
하나의 Reservation 시작 과정으로 처리하며 Partial Success를 허용하지 않습니다.

#### 예약 확정

예약을 최종 확정하려면 추가로 다음 조건을 만족해야 합니다.

- 필요한 탑승객 정보 입력 완료
- 예약에 필요한 형식과 유효기간 조건을 충족하는 테스트용 여권 정보
- 좌석 Hold 시간이 만료되지 않음
- Mock 결제 성공

모든 조건을 만족하면 예약을 `CONFIRMED` 상태로 전환합니다.

### 11.3 예약 소유자

하나의 Reservation은 하나의 Member에 소속됩니다.

Member는 자신의 예약만 조회하거나 취소할 수 있습니다.

Admin과 SuperAdmin은 운영 목적의 예약 현황을 조회할 수 있습니다.

### 11.4 Reservation과 Passenger

하나의 Reservation은 한 명 이상의 Passenger를 포함할 수 있습니다.

Reservation 생성 전에 예약에 포함할 Passenger 정보를 입력합니다.

Passenger 정보 입력은 Seat 선택 및 `PENDING` Reservation 생성 전에 완료되어야 합니다.

다만 Passenger 정보의 실제 저장 시점과
Reservation과 Passenger의 구체적인 데이터 관계는
`05-data-api-design.md`에서 정의합니다.

`CONFIRMED` Reservation은 반드시 한 명 이상의 Passenger를 포함해야 합니다.

한 명의 예약자가 자신을 포함한 여러 Passenger의 항공편을 함께 예약할 수 있습니다.

`ROUND_TRIP` Reservation에서는 동일한 Passenger 구성을
출국 Flight와 귀국 Flight에 공통으로 적용합니다.

Passenger 자체를 출국편과 귀국편별로 별도로 다시 입력하지 않습니다.

단, Passenger의 연령 구분, Seat 필요 여부 및 Seat 배정은
각 Flight별로 독립적으로 판단합니다.

### 11.5 예약 상태

MVP에서 예약 상태는 다음과 같이 정의합니다.

- `PENDING`
- `CONFIRMED`
- `CANCELLED`

#### PENDING

좌석이 임시 점유되어 있고 결제가 아직 완료되지 않은 예약입니다.

#### CONFIRMED

Mock 결제가 성공하여 최종 확정된 예약입니다.

#### CANCELLED

다음 사유 등으로 더 이상 예약을 진행하거나 유지할 수 없는 상태입니다.

- 사용자가 예약 또는 결제 진행을 취소한 경우
- 좌석 Hold 시간이 만료된 경우
- Mock 결제 최대 시도 횟수인 3회를 모두 실패한 경우
- 확정된 예약이 정상적으로 취소된 경우
- SuperAdmin에 의해 강제 취소된 경우
- Flight 자체가 취소된 경우

MVP에서는 취소 사유별로 별도의 예약 상태를 추가하지 않고
`CANCELLED` 상태로 통합하여 관리합니다.

필요한 경우 취소 원인은 별도의 정보로 구분합니다.

### 11.6 예약 생성과 Seat 확보

Reservation에 포함된 모든 Flight에서
Seat가 필요한 모든 Passenger의 Seat 선택이 완료되면
선택한 모든 Seat가 `AVAILABLE` 상태인지 검증합니다.

선택한 모든 Seat를 확보할 수 있는 경우에만
`PENDING` Reservation을 생성하고,
해당 Reservation의 모든 Seat를 `AVAILABLE → HELD` 상태로 전환합니다.

`PENDING` Reservation 생성과 선택한 모든 Seat의 `HELD` 전환은
하나의 예약 시작 과정으로 처리합니다.

편도 또는 왕복 여부와 관계없이,
Reservation에 포함된 Flight의 선택 Seat 중 하나라도 확보에 실패하면
예약 시작 전체를 실패 처리합니다.

이 경우:

- `PENDING` Reservation을 정상 생성된 상태로 남기지 않습니다.
- 일부 Seat만 `HELD` 상태로 남기지 않습니다.
- 선택한 Seat 중 일부만 확보하는 Partial Success를 허용하지 않습니다.

즉, 모든 Seat 확보 성공 또는 전체 실패(All-or-Nothing)를 원칙으로 합니다.

구체적인 Transaction 및 동시성 제어 구현 방식은
시스템 및 데이터 설계에서 정의합니다.

### 11.7 Reservation 번호

모든 Reservation은 사용자 및 운영자가 예약을 식별할 수 있는
고유한 Reservation 번호를 가집니다.

Reservation 번호는 Database 내부 Primary Key와 별도로 관리합니다.

MVP에서는 다음 형식을 사용합니다.

`KOKU-YYYYMMDD-XXXXXX`

예:

- `KOKU-20260821-A7F3K9`

구성은 다음과 같습니다.

- `KOKU`: KOKU Airline 고정 Prefix
- `YYYYMMDD`: Reservation이 생성된 날짜
- `XXXXXX`: 6자리 무작위 영문 대문자 및 숫자 조합

랜덤 문자열의 영문자는 반드시 대문자를 사용합니다.

사용자 입력 및 확인 과정에서 혼동을 줄이기 위해
다음 문자는 랜덤 문자열에서 사용하지 않습니다.

- `I`
- `O`
- `0`
- `1`

Reservation 번호는 `PENDING` Reservation이 정상적으로 생성되는 시점에 발급합니다.

Seat 확보에 실패하여 Reservation 시작 전체가 실패한 경우에는
정상적인 Reservation 번호를 발급하지 않습니다.

한 번 발급된 Reservation 번호는 Reservation의 상태가 변경되더라도
변경하지 않습니다.

취소된 Reservation의 번호를 새로운 Reservation에서 재사용하지 않습니다.

`ROUND_TRIP` Reservation도 하나의 Reservation이므로
출국 Flight와 귀국 Flight에 대해 하나의 Reservation 번호만 사용합니다.

Reservation 번호는 전체 시스템에서 유일해야 합니다.

Application에서 중복 여부를 검증하며,
Database에서도 Unique Constraint를 적용하여 중복 생성을 방지합니다.

랜덤 문자열 충돌이 발생한 경우 새로운 Reservation 번호를 생성하여 다시 시도합니다.

구체적인 Primary Key 및 Reservation 번호 생성 구현 방식은
`05-data-api-design.md`에서 정의합니다.

---

## 12. Mock 결제 정책

### 12.1 목적

본 프로젝트에서는 실제 PG를 사용하지 않고 Mock 결제 시스템을 구현합니다.

목적은 금융 결제 자체가 아니라 다음 Backend 문제를 구현하고 검증하는 것입니다.

- 예약과 결제 상태 연결
- Transaction 처리
- 결제 성공/실패 처리
- 중복 요청 처리
- 예약 상태 전이

### 12.2 실제 금융 거래

실제 카드 결제 또는 금융 거래는 발생하지 않습니다.

실제 카드 번호 등의 결제 개인정보를 저장하지 않습니다.

### 12.3 Payment 모델

MVP에서는 하나의 `Payment`를 하나의 Mock 결제 시도로 정의합니다.

하나의 Reservation은 여러 Payment를 가질 수 있습니다.

`Reservation 1 : N Payment`

`ROUND_TRIP` Reservation에서도 Payment는 출국 Flight와 귀국 Flight별로
분리하지 않습니다.

하나의 Payment는 하나의 Reservation 전체에 대한 Mock 결제 시도를 의미합니다.

따라서 왕복 Reservation의 Mock 결제 금액은
출국 및 귀국 여정 전체 금액을 기준으로 하며,
결제 시도 횟수 최대 3회 정책도 Reservation 전체를 기준으로 적용합니다.

사용자가 Mock 결제를 요청할 때마다 새로운 `Payment`를 생성합니다.

결제 재시도 시 기존 Payment를 재사용하지 않고
새로운 Payment를 생성합니다.

하나의 Reservation에서는 최초 시도를 포함하여
최대 3개의 Payment를 생성할 수 있습니다.

실패한 Payment는 `FAILED` 상태로 유지하여 결제 시도 이력을 보존합니다.

하나의 Reservation에서는 최대 하나의 Payment만
`SUCCESS` 상태가 될 수 있습니다.

하나의 Payment가 `SUCCESS` 상태가 되면
해당 Reservation에서는 새로운 Payment를 생성할 수 없습니다.

성공한 Payment는 Reservation 취소, SuperAdmin 강제 취소 또는 Flight 취소로
환불 정책이 적용되는 경우 `SUCCESS → REFUNDED` 상태로 전환합니다.

동일한 결제 요청이 중복 전달되더라도
Payment가 중복 생성되거나 결제 시도 횟수가 중복 증가하지 않아야 합니다.

구체적인 Database 관계와 중복 요청 방지 방식은
`05-data-api-design.md`와 API Contract에서 정의합니다.

### 12.4 결제 상태

MVP에서 Mock 결제 상태는 다음과 같이 정의합니다.

- `PENDING`
- `SUCCESS`
- `FAILED`
- `CANCELLED`
- `REFUNDED`

#### PENDING

사용자가 Mock 결제를 요청하여 Payment가 생성되었고,
아직 결제 성공 또는 실패 결과가 확정되지 않은 상태입니다.

#### SUCCESS

사용자가 결제하기를 선택하여 Mock 결제가 정상적으로 완료된 상태입니다.

#### FAILED

예상하지 못한 오류 등으로 Mock 결제를 정상적으로 처리하지 못한 상태입니다.

#### CANCELLED

결제가 완료되기 전에 더 이상 해당 Payment를 진행할 수 없어
취소된 상태입니다.

다음 경우 발생할 수 있습니다.

- 사용자가 예약 또는 결제 진행을 취소한 경우
- Seat Hold 시간이 만료된 경우
- 연결된 Flight가 취소된 경우

#### REFUNDED

결제가 성공한 이후 환불 정책이 적용되어
Mock 결제 금액이 전액 환불된 상태입니다.

다음 경우 발생할 수 있습니다.

- Member가 취소 정책에 따라 Reservation을 정상 취소한 경우
- SuperAdmin이 Reservation을 강제 취소한 경우
- 연결된 Flight가 취소된 경우

### 12.5 결제 성공

사용자가 Mock 결제 화면에서 `Mock 결제하기`를 선택하면
새로운 `PENDING` Payment를 생성하고 Mock 결제를 처리합니다.

결제가 정상적으로 처리되면:

- Payment: `PENDING → SUCCESS`
- Reservation: `PENDING → CONFIRMED`
- 해당 Reservation의 모든 Seat: `HELD → RESERVED`

Payment가 `SUCCESS` 상태가 되면 해당 Reservation에서는 추가 Payment를 생성할 수 없습니다.

예약, 결제 및 좌석 상태 변경은 데이터 정합성을 유지해야 합니다.

### 12.6 결제 실패

Mock 결제 처리 과정에서 예상하지 못한 오류가 발생하면
현재 Payment를 `PENDING → FAILED` 상태로 변경합니다.

MVP에서는 하나의 Reservation에 대해
최초 시도를 포함하여 최대 3개의 Payment를 생성할 수 있습니다.

Payment가 `FAILED` 상태가 되더라도
해당 Reservation에서 생성된 Payment 수가 3개 미만이고
Seat Hold 시간이 남아 있다면 Reservation은 `PENDING` 상태를 유지합니다.

사용자는 새로운 Payment를 생성하여 Mock 결제를 다시 시도할 수 있습니다.

예:

`Payment #1 → FAILED`  
`Payment #2 → FAILED`  
`Payment #3 → SUCCESS`

기존 `FAILED` Payment는 상태를 변경하지 않고 결제 이력으로 유지합니다.

세 번째 Payment까지 모두 `FAILED` 상태가 된 경우:

- Reservation: `PENDING → CANCELLED`
- 해당 Reservation의 모든 Seat: `HELD → AVAILABLE`

더 이상 해당 Reservation에서는 새로운 Payment를 생성할 수 없습니다.

Seat Hold 시간이 세 번째 결제 시도 전에 만료된 경우에는
Seat Hold 만료 정책을 우선 적용합니다.

### 12.7 결제 및 예약 진행 취소

사용자가 결제 완료 전에 예약 진행을 취소하면:

- Reservation: `PENDING → CANCELLED`
- 해당 Reservation의 모든 Seat: `HELD → AVAILABLE`

현재 처리 중인 `PENDING` Payment가 존재하는 경우:

- Payment: `PENDING → CANCELLED`

기존 `FAILED` Payment는 상태를 변경하지 않고 결제 이력으로 유지합니다.

Payment가 생성되기 전에 예약 진행을 취소한 경우에는
새로운 Payment를 생성하지 않습니다.

예약, Payment 및 Seat 상태 변경 과정에서 데이터 정합성을 유지해야 합니다.

### 12.8 Seat Hold 만료

Seat Hold 시간이 만료되었는데 Reservation이 아직 `PENDING` 상태라면:

- Reservation: `PENDING → CANCELLED`
- 해당 Reservation의 모든 Seat: `HELD → AVAILABLE`

현재 처리 중인 `PENDING` Payment가 존재하는 경우:

- Payment: `PENDING → CANCELLED`

기존 `FAILED` Payment는 상태를 변경하지 않고 결제 이력으로 유지합니다.

Seat Hold 만료 이후에는 해당 Reservation에서
새로운 Payment를 생성할 수 없습니다.

Hold 만료 처리는 Backend 시간을 기준으로 판단합니다.

---

## 13. 예약 취소 정책

### 13.1 취소 권한

본 절의 예약 취소 정책은 Mock 결제가 완료되어
`CONFIRMED` 상태가 된 예약의 취소를 대상으로 합니다.

`PENDING` 상태에서 예약 진행을 중단하는 경우는
12.7 결제 및 예약 진행 취소 정책을 따릅니다.

Member는 자신의 예약만 취소할 수 있습니다.

Admin 및 SuperAdmin의 취소 권한은 별도 관리자 정책에서 정의합니다.

### 13.2 취소 가능 조건

다음 기본 취소 가능 조건은 `ONE_WAY` Reservation에 적용합니다.

Member는 항공편 출발 예정 시각 기준 24시간 전까지 확정된 예약을 취소할 수 있습니다.

출발 예정 시각까지 24시간 미만이 남은 예약은 Member가 취소할 수 없습니다.

이미 취소된 예약은 다시 취소할 수 없습니다.

MVP에서는 정상 취소 시 Mock 결제 금액을 전액 환불합니다.

### 13.2.1 왕복 Reservation 취소

MVP에서는 `ROUND_TRIP` Reservation의 부분 취소를 지원하지 않습니다.

따라서 다음 기능을 제공하지 않습니다.

- 출국 Flight만 취소
- 귀국 Flight만 취소
- 출국 후 남은 귀국 Flight만 Member가 취소
- 일부 Flight에 대한 부분 Mock 환불

Member가 `ROUND_TRIP` Reservation을 정상 취소하려면
출국 Flight 출발 예정 시각까지 24시간 이상 남아 있어야 합니다.

조건을 만족하면 왕복 Reservation 전체를 취소합니다.

- Reservation: `CONFIRMED → CANCELLED`
- 성공한 Payment: `SUCCESS → REFUNDED`
- 출국 Flight의 예약 Seat: `RESERVED → AVAILABLE`
- 귀국 Flight의 예약 Seat: `RESERVED → AVAILABLE`

Mock 결제 금액은 전체 왕복 Reservation에 대해 전액 환불합니다.

출국 Flight가 이미 출발했거나
출국 Flight 출발까지 24시간 미만인 경우
Member는 해당 왕복 Reservation을 취소할 수 없습니다.

### 13.3 좌석 반환

예약이 정상 취소되면 예약되어 있던 좌석을 다시 예약 가능한 상태로 반환해야 합니다.

예약 취소와 좌석 반환은 데이터 정합성을 보장해야 합니다.

### 13.4 환불

결제가 완료된 `CONFIRMED` 예약이 정상적으로 취소되면:

- Reservation: `CONFIRMED → CANCELLED`
- 성공한 Payment: `SUCCESS → REFUNDED`
- 해당 Reservation의 모든 Seat: `RESERVED → AVAILABLE`

MVP에서는 취소 수수료 및 부분 환불을 구현하지 않고 전액 환불만 지원합니다.

환불은 해당 Reservation에서 `SUCCESS` 상태인 Payment에 적용합니다.

기존 `FAILED` Payment는 상태를 변경하지 않고 결제 이력으로 유지합니다.

### 13.5 취소 수수료

실제 금전 결제를 하지 않으므로 MVP에서는 실제 취소 수수료 정산 기능을 구현하지 않습니다.

필요하다면 정책 표현 및 Mock 계산 수준으로 확장할 수 있습니다.

---

## 14. 외부 실제 항공편 조회 정책

### 14.1 데이터 출처

실제 항공편 정보는 외부 Flight API에서 가져옵니다.

AI가 실제 항공편 정보를 직접 생성하지 않습니다.

### 14.2 지원 범위

외부 항공편 조회 또한 한국 ↔ 일본 노선으로 제한합니다.

지원하지 않는 출발지 또는 목적지가 입력되면 외부 API 호출 전에 Application에서 검증합니다.

### 14.3 예약 기능 분리

외부 API에서 조회된 항공편은 조회 및 추천만 가능합니다.

외부 항공편 결과에 대해서는 다음 기능을 제공하지 않습니다.

- KOKU Airline 예약 생성
- 좌석 예약
- Mock 결제
- 실제 발권

### 14.4 데이터 저장

외부 Flight API의 검색 결과는 KOKU Airline 내부 운항 데이터와 동일한 Entity로 저장하지 않습니다.

MVP에서는 외부 항공편 검색 결과를 영구적인 업무 데이터로 저장하지 않는 것을 기본 원칙으로 합니다.

성능 또는 외부 API 호출량 최적화가 필요한 경우 Cache를 적용할 수 있으며,
Cache 데이터에는 TTL을 적용하여 일정 시간이 지나면 자동으로 만료되도록 합니다.

구체적인 Cache 적용 여부 및 TTL은 성능 측정 후 결정합니다.

---

## 15. AI 항공편 검색 정책

### 15.1 목적

인증된 Member가 자연어로 원하는 항공편 조건을 입력하여 실제 항공편을 검색하고 추천받을 수 있도록 합니다.

AI 항공편 검색 기능은 인증된 Member에게만 제공합니다.

Guest는 일반 항공편 검색 및 외부 실제 항공편 검색은 사용할 수 있지만,
AI 항공편 검색 기능은 사용할 수 없습니다.

예:

> *9월 초에 인천에서 도쿄 가는데 오전 출발이고 너무 비싸지 않은 항공편 찾아줘.*

### 15.2 기본 처리 흐름

1. 사용자가 자연어 검색 조건을 입력합니다.
2. AI가 검색 조건을 구조화합니다.
3. Application이 지원 지역 및 입력값을 검증합니다.
4. 외부 Flight API를 호출합니다.
5. 실제 API 데이터를 기준으로 항공편을 필터링하고 정렬합니다.
6. AI가 결과를 설명하거나 추천합니다.

### 15.3 AI의 역할

AI가 담당할 수 있는 영역:

- 자연어 검색 조건 해석
- 검색 조건 구조화
- 조회 결과 설명
- 추천 이유 생성

가능한 경우 가격, 시간 등의 명확한 정렬 및 필터링은 Application Code에서 결정론적으로 처리합니다.

### 15.4 금지 사항

AI는 다음 정보를 임의로 생성하지 않습니다.

- 편명
- 실제 가격
- 실제 출발 시간
- 실제 도착 시간
- 실제 운항 여부

실제 데이터는 외부 Flight API 결과를 기준으로 합니다.

### 15.5 Transaction 제한

AI는 직접 다음 작업을 수행하지 않습니다.

- 예약 생성
- 예약 취소
- 결제
- 좌석 확정

AI 기능은 조회 및 추천 영역으로 제한합니다.

---

## 16. 관리자 정책

### 16.1 Admin 권한

Admin은 일반적인 항공사 운영 업무를 수행합니다.

가능한 기능:

- 운영 데이터 조회
- 공항 조회
- 노선 조회
- 항공기 및 좌석 구성 조회
- 항공편 생성·수정 및 운영 관리
- 운항 일정 생성·수정 및 운영 관리
- 예약 현황 조회

Admin은 다음 작업을 수행할 수 없습니다.

- 고객 예약 강제 취소
- 공항 Master Data 생성·수정·비활성화
- 노선 Master Data 생성·수정·비활성화
- 항공기 및 좌석 구성 생성·수정·비활성화
- 기타 중요 운영 데이터 변경

### 16.2 SuperAdmin 권한

SuperAdmin은 Admin의 모든 권한을 포함합니다.

추가로 다음 작업을 수행할 수 있습니다.

- 공항 Master Data 생성·수정·비활성화
- 노선 Master Data 생성·수정·비활성화
- 항공기 및 좌석 구성 생성·수정·비활성화
- 고객 예약 강제 취소
- 기타 중요 운영 데이터 관리

핵심 기준정보와 중요 운영 데이터의 변경은 SuperAdmin에게만 허용하는 것을 원칙으로 합니다.

### 16.3 예약 데이터

Admin은 운영 목적으로 예약 현황을 조회할 수 있습니다.

Admin은 개별 사용자의 Reservation을 직접 강제로 취소할 수 없습니다.

SuperAdmin은 운영상 필요한 경우 사용자의 예약을 강제로 취소할 수 있습니다.

강제 취소 시 예약 상태와 좌석 상태의 데이터 정합성을 유지해야 합니다.

### 16.4 SuperAdmin 강제 예약 취소

SuperAdmin은 운영상 필요한 경우 출발 전의
`CONFIRMED` Reservation을 강제로 취소할 수 있습니다.

SuperAdmin 강제 취소에는 Member에게 적용되는
출발 24시간 전 취소 제한을 적용하지 않습니다.

강제 취소 시 취소 사유 입력을 필수로 합니다.

강제 취소가 완료되면 다음 상태 전이를 수행합니다.

- Reservation: `CONFIRMED → CANCELLED`
- 성공한 Payment: `SUCCESS → REFUNDED`
- 해당 Reservation의 모든 Seat: `RESERVED → AVAILABLE`

기존 `FAILED` Payment는 상태를 변경하지 않고 결제 이력으로 유지합니다.

MVP에서는 강제 취소 시 Mock 결제 금액을 전액 환불합니다.

이미 `CANCELLED` 상태인 Reservation은 중복 상태 전이를 수행하지 않습니다.

이미 출발한 Flight의 Reservation은 강제 취소할 수 없습니다.

강제 취소 시 SuperAdmin, 대상 Reservation, 처리 시각 및 취소 사유를
Audit Log로 기록합니다.

`ROUND_TRIP` Reservation의 경우
출국 Flight와 귀국 Flight가 모두 아직 출발하지 않은 경우에만
SuperAdmin이 Reservation 전체를 강제 취소할 수 있습니다.

왕복 Reservation의 일부 Flight가 이미 `DEPARTED` 상태인 경우
SuperAdmin의 Reservation 강제 취소 기능을 제공하지 않습니다.

MVP에서는 왕복 Reservation의 일부 Flight만 강제 취소하거나
부분 Mock 환불하는 기능을 제공하지 않습니다.

강제 취소가 가능한 왕복 Reservation은 전체를 취소하며:

- Reservation: `CONFIRMED → CANCELLED`
- 성공한 Payment: `SUCCESS → REFUNDED`
- 출국 및 귀국 Flight의 모든 예약 Seat: `RESERVED → AVAILABLE`

처리합니다.

---

## 17. 데이터 삭제 및 비활성화 정책

### 17.1 기본 원칙

서비스의 주요 도메인 데이터는 과거 예약 및 운영 이력과의 참조 관계를 보존하기 위해
물리 삭제(Hard Delete)를 최소화합니다.

삭제가 필요한 주요 운영 데이터에는 Soft Delete 또는 비활성화 정책을 적용합니다.

### 17.2 Soft Delete 적용 대상

다음 Master Data에는 기본적으로 Soft Delete 또는 비활성화 정책을 적용합니다.

- 공항
- 노선
- 항공기
- 좌석 기본 정보

회원 탈퇴는 Soft Delete 대신 회원 상태를 `WITHDRAWN`으로 변경하여 관리합니다.

### 17.3 예약 및 결제 데이터

예약과 결제는 삭제 대상이 아니라 상태 전이를 통해 관리합니다.

예약 취소 시 예약 데이터를 삭제하지 않고 `CANCELLED` 상태로 변경합니다.

결제 데이터 또한 처리 결과 및 이력 보존을 위해 물리적으로 삭제하지 않습니다.

### 17.4 항공편

이미 운항 기록 또는 Reservation 데이터가 존재하는 Flight는 삭제하지 않습니다.

출발 전에 운항이 취소된 Flight는 삭제하지 않고
`CANCELLED` 상태로 유지하여 과거 운영 및 예약 이력을 보존합니다.

이미 출발한 Flight 역시 과거 운항 이력 보존을 위해 삭제하지 않습니다.

### 17.5 Hard Delete

Hard Delete는 테스트 데이터 정리 등 명확한 관리 목적이 있는 경우에만 제한적으로 사용합니다.

운영 데이터에 대한 Hard Delete는 일반적인 서비스 기능으로 제공하지 않습니다.

구체적인 Soft Delete 구현 방식과 조회 Filter는 데이터 및 시스템 설계에서 정의합니다.

---

## 18. 주요 상태 전이

다음 Reservation 상태 전이는 `ONE_WAY`와 `ROUND_TRIP` 모두 동일하게 적용합니다.
`ROUND_TRIP`에서도 출국 / 귀국 Flight별 Reservation 상태를 별도로 관리하지 않습니다.

### 18.1 예약

기본 상태 전이:

`PENDING → CONFIRMED`

Mock 결제 성공 시 발생합니다.

`PENDING → CANCELLED`

다음 경우 발생할 수 있습니다.

- 사용자가 `PENDING` 상태의 예약 진행을 취소한 경우
- 좌석 Hold 시간이 만료된 경우
- Mock 결제 최대 시도 횟수인 3회를 모두 실패한 경우
- 연결된 Flight가 취소된 경우

`CONFIRMED → CANCELLED`

확정된 예약을 사용자가 정상 취소하거나,
SuperAdmin이 정책에 따라 강제 취소하거나,
연결된 Flight가 취소된 경우 발생합니다.

### 18.2 결제

다음 상태 전이는 개별 Payment에 적용합니다.

기본 상태 전이:

`PENDING → SUCCESS`

사용자가 결제하기를 선택하고 Mock 결제가 정상 처리된 경우 발생합니다.

`PENDING → FAILED`

Mock 결제 처리 과정에서 예상하지 못한 오류가 발생한 경우 발생합니다.

`PENDING → CANCELLED`

다음 경우 발생할 수 있습니다.

- 결제가 완료되기 전에 사용자가 예약 또는 결제 진행을 취소한 경우
- 좌석 Hold 시간이 만료되어 더 이상 결제를 진행할 수 없는 경우
- 연결된 Flight가 취소된 경우

`SUCCESS → REFUNDED`

결제가 완료된 Reservation이 취소 정책에 따라 정상 취소되거나,
SuperAdmin에 의해 강제 취소되거나,
연결된 Flight가 취소되어 전액 환불된 경우 발생합니다.

`ROUND_TRIP`에서는 다음 Seat 상태 전이를
출국 Flight와 귀국 Flight의 각 Seat에 적용합니다.

### 18.3 좌석

예약 진행 시작:

`AVAILABLE → HELD`

Mock 결제 성공:

`HELD → RESERVED`

결제 취소 또는 Hold 만료:

`HELD → AVAILABLE`

예약 취소:

`RESERVED → AVAILABLE`

단, `ROUND_TRIP` Reservation에서 일부 Flight가 이미 `DEPARTED`한 이후
나머지 Flight 취소로 Reservation 전체가 `CANCELLED`되는 경우에는
이미 출발한 Flight의 Seat 상태를 변경하지 않습니다.

아직 출발하지 않은 Flight의 `RESERVED` Seat만
`AVAILABLE` 상태로 반환합니다.

구체적인 예외는 9.6.1 왕복 Reservation과 Flight 취소 정책을 따릅니다.

### 18.4 Flight

기본 상태 전이:

`SCHEDULED → CANCELLED`

출발 전 Admin 또는 SuperAdmin이 Flight 취소 정책에 따라
운항을 취소한 경우 발생합니다.

`SCHEDULED → DEPARTED`

출발 예정 시각이 지난 Flight를 출발한 것으로 처리하는 경우 발생합니다.

MVP에서는 실제 운항 시스템과 연동하지 않으므로
`DEPARTED` 전환의 구체적인 처리 방식은 시스템 설계에서 정의합니다.

---

## 19. 주요 정상 시나리오

### 19.1 일반 회원가입 및 편도 예약

1. 사용자가 일반 회원가입을 진행합니다.
2. 로그인합니다.
3. 출발지, 목적지, 날짜를 입력합니다.
4. KOKU Airline 항공편을 검색합니다.
5. 항공편을 선택합니다.
6. Passenger 정보를 입력합니다.
7. 입력된 생년월일과 Flight 탑승일을 기준으로 Adult, Child, Infant를 판단합니다.
8. 시스템에서 Passenger별 테스트용 여권 정보를 생성합니다.
9. Seat가 필요한 Passenger의 예약 가능한 좌석을 확인합니다.
10. Seat가 필요한 모든 Passenger의 좌석을 선택합니다.
11. 선택한 모든 Seat를 확보할 수 있는지 검증합니다.
12. 모든 Seat 확보에 성공하면 `PENDING` 상태의 Reservation을 생성합니다.
13. 선택한 모든 Seat를 `AVAILABLE → HELD` 상태로 전환하고 최대 1시간 동안 임시 점유합니다.
14. 사용자가 예약 내용을 확인합니다.
15. Mock 결제 화면으로 이동합니다.
16. 사용자가 `Mock 결제하기`를 선택합니다.
17. 새로운 `PENDING` Payment를 생성합니다.
18. Mock 결제를 처리합니다.
19. 정상 처리되면 해당 Payment를 `SUCCESS` 상태로 변경합니다.
20. Reservation을 `CONFIRMED` 상태로 변경합니다.
21. 해당 Reservation의 모든 Seat를 `HELD → RESERVED`로 변경합니다.
22. 사용자가 확정된 예약 내역을 조회합니다.

### 19.2 Google OAuth 로그인 및 예약

1. 사용자가 Google 로그인을 선택합니다.
2. Google OAuth 인증을 수행합니다.
3. 기존 GOOGLE AuthAccount가 존재하는 경우 연결된 Member의 상태를 확인하고,
   `ACTIVE` 상태인 경우에만 인증합니다.
4. 기존 GOOGLE AuthAccount가 없으면 동일 Email Member 존재 여부를 확인합니다.
5. 동일 Email Member가 없는 경우 Member를 생성하고 GOOGLE AuthAccount를 연결합니다.
6. 동일 Email Member가 존재하는 경우 4.6.1 동일 Email 계정 연동 정책을 따릅니다.
7. 서비스 인증이 완료됩니다.
8. 이후 Member와 동일한 예약 흐름을 이용합니다.

### 19.3 편도 Reservation 취소

1. Member가 자신의 예약을 조회합니다.
2. 취소 가능한 `CONFIRMED` 예약을 선택합니다.
3. 시스템이 항공편 출발 예정 시각까지 24시간 이상 남아 있는지 확인합니다.
4. Member가 예약 취소를 요청합니다.
5. 예약 상태를 `CANCELLED`로 변경합니다.
6. 해당 Reservation의 성공한 Payment를 `SUCCESS → REFUNDED`로 변경합니다.
7. 해당 Reservation의 모든 Seat를 `RESERVED → AVAILABLE`로 반환합니다.
8. 변경된 예약 및 환불 상태를 사용자에게 제공합니다.

### 19.4 AI 실제 항공편 검색

1. 사용자가 자연어로 항공편 조건을 입력합니다.
2. AI가 검색 조건을 구조화합니다.
3. Application이 입력 조건을 검증합니다.
4. 외부 Flight API를 호출합니다.
5. 실제 항공편 결과를 수집합니다.
6. 조건에 맞게 결과를 필터링 및 정렬합니다.
7. AI가 추천 항공편과 추천 이유를 제공합니다.

### 19.5 왕복 예약

1. Member가 `ROUND_TRIP`을 선택합니다.
2. 출발 Airport, 도착 Airport, 출발 Date, 귀국 Date를 입력합니다.
3. 출국 Flight를 검색하고 선택합니다.
4. 출국 Route의 역방향 Route에서 귀국 Flight를 검색하고 선택합니다.
5. 동일한 Passenger 구성을 입력합니다.
6. 각 Flight 탑승일 기준으로 Passenger 연령을 계산합니다.
7. 출국 Flight의 Seat를 선택합니다.
8. 귀국 Flight의 Seat를 선택합니다.
9. 출국 및 귀국 Flight의 모든 선택 Seat가 `AVAILABLE`인지 검증합니다.
10. 모든 Seat 확보에 성공한 경우 하나의 `PENDING` Reservation을 생성합니다.
11. 출국 및 귀국 Flight의 선택 Seat를 모두 `HELD` 상태로 전환합니다.
12. 사용자가 왕복 전체 Reservation 내용을 확인합니다.
13. 왕복 전체 금액을 대상으로 Mock 결제를 한 번 진행합니다.
14. Payment가 `SUCCESS`가 되면 Reservation을 `CONFIRMED`로 변경합니다.
15. 출국 및 귀국 Flight의 모든 `HELD` Seat를 `RESERVED`로 변경합니다.
16. 사용자는 하나의 왕복 Reservation으로 출국 및 귀국 여정을 조회합니다.

### 19.6 왕복 Reservation 취소

1. Member가 자신의 `ROUND_TRIP` Reservation을 조회합니다.
2. Reservation이 `CONFIRMED` 상태인지 확인합니다.
3. 출국 Flight 출발 예정 시각까지 24시간 이상 남아 있는지 확인합니다.
4. 조건을 만족하면 Member가 왕복 Reservation 전체 취소를 요청합니다.
5. Reservation을 `CONFIRMED → CANCELLED` 상태로 변경합니다.
6. 성공한 Payment를 `SUCCESS → REFUNDED` 상태로 변경합니다.
7. 출국 및 귀국 Flight의 모든 `RESERVED` Seat를 `AVAILABLE`로 반환합니다.
8. 왕복 전체 Mock 결제 금액을 전액 환불합니다.
9. 변경된 Reservation 및 환불 상태를 Member에게 제공합니다.

출국 Flight 출발 예정 시각까지 24시간 미만이거나
이미 출국한 경우 Member 취소를 허용하지 않습니다.

MVP에서는 출국 또는 귀국 Flight만 별도로 취소하는
부분 취소를 지원하지 않습니다.

---

## 20. 주요 예외 시나리오

### 20.1 지원하지 않는 노선

한국 ↔ 일본 이외 노선을 요청하면 검색을 거부합니다.

### 20.2 동일 좌석 동시 예약

두 사용자가 동일한 좌석을 동시에 예약하면 하나의 예약만 최종 성공해야 합니다.

실패한 사용자에게는 좌석을 다시 선택하도록 안내합니다.

### 20.3 예약 시작 중 Seat 상태 변경

`PENDING` Reservation 생성과 선택한 모든 Seat의
`AVAILABLE → HELD` 전환 과정에서
선택한 Seat 중 하나 이상이 다른 Transaction에 의해
더 이상 `AVAILABLE` 상태가 아닌 경우 예약 시작 전체를 실패 처리합니다.

일부 Seat만 `HELD` 상태로 확보하는 Partial Success는 허용하지 않습니다.

실패한 사용자에게는 선택한 Seat 중 일부를 확보하지 못했음을 안내하고
최신 Seat 상태를 다시 조회한 후 좌석을 다시 선택하도록 합니다.

`ROUND_TRIP`에서는 출국 Flight와 귀국 Flight의 모든 선택 Seat를
하나의 확보 대상 집합으로 취급하며,
어느 Flight의 Seat에서든 확보 실패가 발생하면 왕복 Reservation 시작 전체를 실패 처리합니다.

### 20.4 Mock 결제 실패

Mock 결제 처리 과정에서 예상하지 못한 오류가 발생하면
현재 Payment를 `PENDING → FAILED` 상태로 변경합니다.

해당 Reservation에 생성된 Payment가 3개 미만이고
Seat Hold 시간이 남아 있다면:

- Reservation은 `PENDING` 상태를 유지합니다.
- 해당 Reservation의 모든 Seat는 `HELD` 상태를 유지합니다.
- 사용자는 새로운 Payment를 생성하여 결제를 다시 시도할 수 있습니다.

세 번째 Payment까지 모두 실패하면:

- 모든 Payment는 `FAILED` 상태로 결제 이력을 유지합니다.
- Reservation을 `CANCELLED` 상태로 변경합니다.
- 해당 Reservation의 모든 Seat를 `AVAILABLE` 상태로 반환합니다.
- 추가 Payment 생성을 허용하지 않습니다.

Payment 수와 관계없이 Seat Hold 시간이 먼저 만료되면
Seat Hold 만료 정책을 적용합니다.

### 20.5 이미 취소된 예약

이미 취소된 예약에 다시 취소 요청이 들어오면 요청을 거부하거나 멱등하게 처리합니다.

구체적인 API 정책은 API Contract에서 결정합니다.

### 20.6 권한 없는 접근

다른 Member의 예약을 조회하거나 수정하려는 요청을 거부합니다.

Admin 또는 SuperAdmin 권한이 필요한 API에 권한 없는 사용자가 접근하는 요청을 거부합니다.

### 20.7 외부 Flight API 장애

외부 Flight API의 Timeout, 오류 또는 Rate Limit 발생 시 내부 KOKU Airline 예약 시스템에 영향을 주지 않아야 합니다.

사용자에게 외부 항공편 조회 실패를 명확하게 전달합니다.

### 20.8 AI 응답 실패

AI 응답 생성이 실패하더라도 실제 항공편 API 원본 데이터나 내부 예약 시스템에 잘못된 변경이 발생하지 않아야 합니다.

---

## 21. 미확정 정책

다음 항목은 M1 설계 과정에서 추가로 확정합니다.

- [ ] Access Token / Refresh Token 정책
- [ ] JWT 저장 및 재발급 방식
- [ ] 여권번호 등 민감정보의 저장 및 보호 방식
- [ ] 외부 Flight API Cache 적용 여부 및 TTL

---

## 22. Domain Policy 변경 원칙

본 문서의 정책은 Backend와 Frontend 구현의 기준으로 사용합니다.

AI Agent는 구현 과정에서 본 문서의 Domain Policy를 임의로 변경하지 않습니다.

다음 변경이 필요한 경우 구현을 중단하고 Human에게 보고합니다.

- 예약 정책 변경
- 좌석 상태 및 동시성 정책 변경
- 결제 상태 전이 변경
- 인증/인가 정책 변경
- 사용자 Role 변경
- Member와 AuthAccount 관계 변경
- 외부 실제 항공편과 내부 항공편의 경계 변경
- 새로운 핵심 도메인 추가
- `TripType` 또는 편도 / 왕복 예약 정책 변경
- Reservation과 Flight 구성 관계 변경

정책 변경이 승인된 경우 관련 문서, ERD 및 API Contract의 일관성을 함께 검토합니다.