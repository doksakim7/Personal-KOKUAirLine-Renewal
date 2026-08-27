# KOKU Airline Renewal - Data & API Design

## 1. 문서 목적

본 문서는 KOKU Airline Renewal의
Database 구조, Entity 관계, Constraint, Index 및 API Contract의 기본 구조를 정의합니다.

본 문서의 주요 목적은 다음과 같습니다.

- 핵심 Entity와 관계를 정의합니다.
- Domain Policy를 Database 구조로 표현합니다.
- Reservation과 Flight의 관계를 정의합니다.
- Passenger와 Reservation의 관계를 정의합니다.
- Flight별 Seat 상태 관리 구조를 정의합니다.
- Mock Payment 이력 구조를 정의합니다.
- Reservation 번호 생성 및 유일성 보장 방식을 정의합니다.
- Audit Log 데이터 구조를 정의합니다.
- REST API의 기본 규칙을 정의합니다.
- API Request / Response의 기본 Contract를 정의합니다.
- Idempotency가 필요한 API의 기본 방향을 정의합니다.
- 외부 실제 항공편 API와 내부 Domain Model의 데이터 경계를 정의합니다.
- AI 항공편 검색에 필요한 Structured Data Contract의 기본 방향을 정의합니다.

비즈니스 규칙은 `02-domain-policy.md`를 기준으로 합니다.

사용자 화면과 사용자 흐름은
`03-ui-design.md`를 기준으로 합니다.

시스템 Architecture, 인증, Transaction, 동시성 및 Infrastructure 관련 원칙은
`04-system-design.md`를 기준으로 합니다.

본 문서는 상위 문서에서 정의된 정책을
Database 또는 API 구조를 이유로 임의 변경하지 않습니다.

---

### 1.1 설계 원칙

Data 및 API 설계는 다음 원칙을 따릅니다.

1. Domain Policy를 Database 구조보다 우선합니다.
2. 내부 Database Primary Key와 사용자 공개 식별자를 구분합니다.
3. Entity 관계는 실제 Business Lifecycle을 기준으로 정의합니다.
4. Database Constraint로 보호 가능한 정합성은 Application Validation과 함께 보호합니다.
5. API Response에 Database 내부 구조를 그대로 노출하지 않습니다.
6. 외부 Flight API의 Data Model과 내부 KOKU Airline Domain Model을 분리합니다.
7. 사용자에게 필요하지 않은 민감정보는 API Response에 포함하지 않습니다.
8. 상태 값은 Backend의 Canonical Enum을 사용합니다.
9. API Contract 변경은 Frontend와 Backend 양쪽 영향을 검토합니다.
10. 구현 편의를 위해 Domain Policy를 약화하지 않습니다.

---

##  2. 데이터 모델 전체 구조

###  2.1 핵심 Domain Entity

MVP의 핵심 Domain Entity는 다음과 같습니다.

- `Member`
- `AuthAccount`
- `Airport`
- `Route`
- `Aircraft`
- `FlightSchedule`
- `Flight`
- `Seat`
- `Passenger`
- `Reservation`
- `Payment`

Domain Policy에서 정의된 Canonical Term을 유지합니다.

---

###  2.2 지원 관계 모델

핵심 Entity 사이의 관계를 표현하기 위해
다음과 같은 Supporting Model 또는 Mapping Table을 사용할 수 있습니다.

초안:

- `AircraftSeat`
- `FlightScheduleDay`
- `ReservationFlight`
- `ReservationPassenger`
- `PassengerFlight`
- `AuditLog`

이들은 핵심 Domain을 새롭게 추가하기 위한 개념이 아니라
기존 Domain Policy를 관계형 Database에 표현하기 위한 기술적 모델입니다.

실제 JPA Entity로 구현할지,
단순 Join Table 또는 별도 Entity로 구현할지는
각 관계의 추가 속성 필요 여부를 기준으로 결정합니다.

---

### 2.3 전체 관계 초안

```text
Member
  |
  +----< AuthAccount
  |
  +----< Reservation
              |
              +----< ReservationFlight >---- Flight
              |
              +----< ReservationPassenger >---- Passenger
              |
              +----< Payment
              |
              +----< PassengerFlight
                         |
                         +---- Passenger
                         |
                         +---- Flight
                         |
                         +---- Seat
                         |
                         +---- Companion Adult

Airport
  |
  +----< Route >---- Airport
              |
              +----< FlightSchedule
              |        |
              |        +----< FlightScheduleDay
              |        |
              |        +----< Flight
              |
              +----< Flight

Aircraft
  |
  +----< AircraftSeat
  |
  +----< FlightSchedule
  |
  +----< Flight
             |
             +----< Seat
```

위 구조는 초안이며,
구체적인 FK와 Mapping 방향은 본 문서의 각 절에서 정의합니다.

---

## 3. 공통 Database 원칙

### 3.1 Primary Key

내부 Entity의 Primary Key는
Database 내부 식별을 위한 값으로 사용합니다.

기본 방향:

```text
BIGINT
AUTO_INCREMENT
```

또는 JPA에서 이에 대응하는 Identity 전략을 사용합니다.

Database Primary Key는
사용자에게 공개되는 업무 식별자로 사용하지 않습니다.

예:

```text
Reservation PK
12345

사용자 공개 Reservation 번호
KOKU-20260821-A7F3K9
```

---

### 3.2 공통 Date / Time Column

절대 시각을 나타내는 Timestamp는
Backend에서 Java `Instant`를 사용합니다.

MySQL에서는 `DATETIME(6)`을 사용하고
UTC 기준 값을 저장합니다.

기본 Mapping:

```text
Java Instant
→ MySQL DATETIME(6)
→ UTC
```

필요한 Entity는 다음 Audit Timestamp를 가집니다.

```text
created_at
→ Instant
→ DATETIME(6)
→ UTC

updated_at
→ Instant
→ DATETIME(6)
→ UTC
```

삭제 대신 비활성화 정책을 사용하는 Master Data는
다음 값을 사용합니다.

```text
active
→ BOOLEAN

deactivated_at
→ Instant
→ DATETIME(6)
→ UTC
→ 활성 상태에서는 NULL
```

본 프로젝트에서는 범용 `deleted_at` 기반 Soft Delete를 사용하지 않습니다.

Master Data는:

```text
active
+
deactivated_at
```

으로 비활성화 상태와 시각을 관리합니다.

Member, Reservation, Payment 등
명시적인 Lifecycle 상태를 가지는 Domain은
범용 `deleted_at` 대신 각 Domain의 상태 값을 사용합니다.

날짜 자체만 의미하는 값은
절대 시각과 구분합니다.

```text
Java LocalDate
→ MySQL DATE
```

대표 대상:

- Passenger `birth_date`
- Test Passport 만료일

JDBC / Hibernate의 Database Time Zone은
UTC 기준으로 구성합니다.

---

### 3.3 Enum 저장

상태 및 유형 Enum은
Database에서 의미를 확인할 수 있도록 문자열 저장을 기본 방향으로 합니다.

예:

```text
ACTIVE
WITHDRAWN

PENDING
CONFIRMED
CANCELLED

AVAILABLE
HELD
RESERVED
UNAVAILABLE

ECONOMY
PREMIUM_ECONOMY
BUSINESS
```

JPA에서는 `EnumType.STRING` 사용을 기본으로 검토합니다.

Enum 순서 변경으로 Database 의미가 달라질 수 있는
Ordinal 저장은 기본안으로 사용하지 않습니다.

`SeatClass`는 다음 값을 사용하며
Database에는 문자열로 저장합니다.

```text
ECONOMY
PREMIUM_ECONOMY
BUSINESS
```

`FIRST`는 MVP 범위에 포함하지 않습니다.

---

### 3.4 Soft Delete / 비활성화

본 프로젝트에서는 모든 Entity에 공통 `deleted_at`을 두는
범용 Soft Delete 구조를 사용하지 않습니다.

Domain의 Lifecycle에 따라
비활성화 또는 상태 전이를 명시적으로 사용합니다.

Master Data는 다음 구조를 사용합니다.

```text
active
+
deactivated_at
```

대상:

- `Airport`
- `Route`
- `Aircraft`
- `AircraftSeat`
- `FlightSchedule`

Master Data 비활성화 시:

```text
active
true → false

deactivated_at
NULL → 비활성화 시점
```

`deactivated_at`은:

```text
Java Instant
→ MySQL DATETIME(6)
→ UTC
```

기준으로 저장합니다.

다시 활성화 기능을 구현하는 경우
`active = true`로 변경하고
`deactivated_at = NULL`로 복원하는 것을 기본 원칙으로 합니다.

Member는 물리적으로 삭제하지 않고:

```text
Member.status
ACTIVE → WITHDRAWN
```

상태 전이를 사용합니다.

Reservation, Payment, Flight 등
명시적인 상태 Lifecycle을 가지는 Domain 역시
범용 `deleted_at` 대신 각 Domain의 상태 Enum을 사용합니다.

과거 Reservation 또는 Flight와 연결된 Data는
참조 무결성과 이력 보존을 위해 임의로 물리 삭제하지 않습니다.

---

## 4. Member

### 4.1 역할

`Member`는 서비스 사용자를 나타냅니다.

인증 수단 자체는 `Member`가 아니라
`AuthAccount`에서 관리합니다.

하나의 Member는
여러 AuthAccount를 가질 수 있습니다.

```text
Member 1 : N AuthAccount
```

---

### 4.2 주요 Column 초안

```text
member
--------------------------------
id
email
role
status
created_at
updated_at
```

초안 의미:

- `id`: 내부 Primary Key
- `email`: 정규화된 Member Email
- `role`: `MEMBER`, `ADMIN`, `SUPERADMIN`
- `status`: `ACTIVE`, `WITHDRAWN`

`role`은 서비스 Role을 저장하며 다음 값을 사용합니다.

- `MEMBER`
- `ADMIN`
- `SUPERADMIN`

Spring Security에서는 서비스 Role을 다음 권한 Code로 매핑합니다.

```text
MEMBER     → USER
ADMIN      → ADMIN
SUPERADMIN → SUPERADMIN
```

`Member.role`의 `MEMBER`와 Spring Security 권한 Code `USER`를 혼용하지 않습니다.

---

### 4.3 Email

Member Email은
Domain Policy에서 정의한 정규화 규칙을 적용한 결과를 저장합니다.

기준:

```text
trim()
+
lowercase(Locale.ROOT)
```

일반 회원가입, 일반 로그인 및 Google OAuth 계정 연동 모두
동일한 정규화 기준을 사용합니다.

Email 원본 대소문자 형태를 별도로 보존하지 않습니다.

Database에서는 정규화된 Email의 중복을 허용하지 않습니다.

초안 Constraint:

```text
UNIQUE(member.email)
```

---

### 4.4 Member 상태

```text
ACTIVE
WITHDRAWN
```

`WITHDRAWN` Member는 물리적으로 삭제하지 않습니다.

기존 Reservation 및 Payment 이력과의 관계도 유지합니다.

---

## 5. AuthAccount

### 5.1 역할

`AuthAccount`는 Member의 인증 수단을 나타냅니다.

지원 Provider:

```text
LOCAL
GOOGLE
```

하나의 Member는
LOCAL과 GOOGLE AuthAccount를 각각 가질 수 있습니다.

---

### 5.2 주요 Column 초안

```text
auth_account
--------------------------------
id
member_id
provider
provider_subject
password_hash
created_at
updated_at
```

Column 사용 원칙:

#### LOCAL

```text
provider = LOCAL
password_hash = PasswordEncoder 결과
provider_subject = NULL
```

#### GOOGLE

```text
provider = GOOGLE
provider_subject = Google OAuth provider sub
password_hash = NULL
```

---

### 5.3 Google 식별자

Google 계정의 고유 식별에는
Email이 아니라 Provider에서 제공하는 `sub`를 사용합니다.

Google `sub`에는
Member Email 정규화 규칙을 적용하지 않습니다.

초안 Constraint:

```text
UNIQUE(provider, provider_subject)
```

LOCAL에 대해서는
동일 Member에 중복 LOCAL AuthAccount가 생성되지 않도록
Application 및 Database 구조에서 보호합니다.

---

### 5.4 Password

Password 평문은 저장하지 않습니다.

LOCAL AuthAccount에는
PasswordEncoder 결과만 저장합니다.

Password Hash는 API Response에 포함하지 않습니다.

LOCAL AuthAccount의 Password를 변경하려면
현재 Password 재인증을 반드시 수행합니다.

기본 처리:

```text
현재 Password
        ↓
PasswordEncoder 검증
        ↓
새 Password 정책 검증
        ↓
새 Password Hash 저장
```

현재 Password 검증에 실패한 경우
Password를 변경하지 않습니다.

`GOOGLE` AuthAccount만 보유하고
`LOCAL` AuthAccount가 존재하지 않는 Member에게는
LOCAL Password 변경 API를 제공하지 않습니다.

---

### 5.5 Refresh Token Server-side 저장

Refresh Token은 MySQL의 영속 Domain Entity로 저장하지 않습니다.

`04-system-design.md`의 Token 정책에 따라
Refresh Token의 Server-side 상태는 Redis에서 관리합니다.

Refresh Token 원문은 Redis에 저장하지 않습니다.

논리적인 저장 정보는 다음과 같습니다.

```text
Refresh Token 식별 정보
Member 식별 정보
Refresh Token Hash
TTL
```

Refresh Token의 원문은
Client의 HttpOnly / Secure Cookie로만 전달합니다.

Redis에는 Refresh Token 검증에 필요한
Hash 및 최소한의 식별 정보만 저장합니다.

TTL은 Refresh Token의 유효기간과 동일한
**7일**을 적용합니다.

```text
Refresh Token TTL
→ 7일
```

Access Token 재발급 시 Refresh Token Rotation을 수행하므로
기존 Refresh Token의 Redis 저장 정보는 폐기하고
새 Refresh Token에 대응하는 Hash 정보를 저장합니다.

Logout 시에도 해당 Refresh Token의
Redis 저장 정보를 삭제합니다.

Refresh Token Hash는 API Response에 포함하지 않습니다.

---

## 6. Airport

### 6.1 역할

`Airport`는 KOKU Airline이 지원하는 공항 Master Data입니다.

MVP 지원 공항은 Domain Policy를 기준으로 합니다.

#### 한국

- ICN
- GMP
- PUS
- CJU

#### 일본

- NRT
- HND
- KIX
- FUK
- CTS
- NGO

---

### 6.2 주요 Column 초안

```text
airport
--------------------------------
id
iata_code
country_code
timezone
active
deactivated_at
created_at
updated_at
```

`timezone`은 IANA Time Zone ID 문자열을 저장합니다.

```text
Java Business Logic
→ ZoneId

Database
→ VARCHAR(50)

예:
Asia/Seoul
Asia/Tokyo
```

Database에 UTC Offset `+09:00` 자체를
Airport Time Zone 식별자로 저장하지 않습니다.

---

### 6.3 IATA Code

Airport의 업무 식별자는 IATA Code입니다.

예:

```text
ICN
NRT
KIX
```

초안 Constraint:

```text
UNIQUE(airport.iata_code)
```

IATA Code는 대문자 영문 3자리 형식을 사용합니다.

Database Column은 다음 형식을 사용합니다.

```text
iata_code
→ VARCHAR(3)
```

IATA Code는 정확히 영문 대문자 3자리이므로
가변적인 길이를 허용하지 않습니다.

---

### 6.4 Time Zone

각 Airport는 IANA Time Zone ID를 저장합니다.

MVP 지원 Airport의 Time Zone:

```text
한국 Airport
→ Asia/Seoul

일본 Airport
→ Asia/Tokyo
```

한국과 일본은 현재 모두 UTC+9이지만,
시스템의 시간 처리를 고정 Offset `+09:00`에 의존하지 않습니다.

Flight의 출발 / 도착 시각을 사용자에게 표시하거나
Local Date / Time 기준 Business Rule을 판단할 때는
해당 Airport의 `timezone`을 사용합니다.

예:

```text
ICN
→ Asia/Seoul

NRT
→ Asia/Tokyo
```

Database의 시간값은 UTC 기준으로 저장하고,
Airport의 `timezone`을 이용하여 Local Date / Time으로 변환합니다.

---

## 7. Route

### 7.1 역할

`Route`는 KOKU Airline이 운항 가능한
출발 Airport와 도착 Airport의 조합을 나타냅니다.

MVP에서는 한국 ↔ 일본 Route만 허용합니다.

---

### 7.2 주요 Column 초안

```text
route
--------------------------------
id
departure_airport_id
arrival_airport_id
active
deactivated_at
created_at
updated_at
```

---

### 7.3 Constraint

다음 조건을 만족해야 합니다.

```text
departure_airport_id != arrival_airport_id
```

동일 방향 Route 중복을 허용하지 않습니다.

초안:

```text
UNIQUE(
  departure_airport_id,
  arrival_airport_id
)
```

예:

```text
ICN → NRT
```

와

```text
NRT → ICN
```

은 서로 다른 Route입니다.

`ROUND_TRIP`에서는
출국 Route와 정확히 반대 방향 Route를 귀국 Flight에 사용합니다.

---

## 8. Aircraft

### 8.1 역할

`Aircraft`는 KOKU Airline의 가상 운항 항공기를 나타냅니다.

하나의 Flight에는 하나의 Aircraft가 배정됩니다.

---

### 8.2 주요 Column 초안

```text
aircraft
--------------------------------
id
aircraft_code
model_name
active
deactivated_at
created_at
updated_at
```

`aircraft_code`의 구체적인 형식은
구현 전 결정합니다.

---

## 9. Aircraft Seat 구성

### 9.1 역할

Aircraft의 좌석 Layout은
Flight 생성 시 Seat 구성의 기준으로 사용합니다.

기존 Domain의 `Seat`와 구분하기 위해
Database 기술 모델로 `AircraftSeat` 사용을 검토합니다.

`AircraftSeat`는 새로운 사용자 Domain이 아니라
Aircraft Seat Configuration을 표현하기 위한 Supporting Model입니다.

---

### 9.2 주요 Column 초안

```text
aircraft_seat
--------------------------------
id
aircraft_id
seat_no
row_no
seat_column
seat_class
active
deactivated_at
created_at
updated_at
```

Column 의미:

- `aircraft_id`: 해당 Seat Configuration이 속한 Aircraft
- `seat_no`: Aircraft 내부 Seat Number
- `row_no`: Seat Row
- `seat_column`: Seat Column
- `seat_class`: KOKU Airline의 좌석 등급

`seat_class`는 다음 Canonical Enum을 사용합니다.

```text
ECONOMY
PREMIUM_ECONOMY
BUSINESS
```

JPA에서는 `SeatClass`를 `EnumType.STRING`으로 저장합니다.

Aircraft마다 서로 다른 Seat 수와 SeatClass 구성을 가질 수 있습니다.

예:

```text
1A  BUSINESS
1B  BUSINESS
5A  PREMIUM_ECONOMY
5B  PREMIUM_ECONOMY
12A ECONOMY
12B ECONOMY
```

Flight 생성 시
활성 상태의 `AircraftSeat`를 기준으로
Flight 전용 Seat Snapshot을 생성합니다.

---

### 9.3 Seat 중복

동일 Aircraft 안에서 Seat Number는 중복되지 않습니다.

초안 Constraint:

```text
UNIQUE(
  aircraft_id,
  seat_no
)
```

---

### 9.4 Seat 인접성

Child Passenger의 인접 Seat Validation을 위해
Seat의 Row와 Column 구조를 판단할 수 있어야 합니다.

MVP에서 인접 Seat는:

- 동일 Row
- 좌우 직접 연결
- 두 Seat 사이에 통로 없음

을 의미합니다.

통로 위치를 어떻게 표현할지는
Aircraft Seat Layout 구현 시 확정합니다.

검토 가능한 방식:

- Seat Column 기반 Rule
- Seat adjacency 정보 명시 저장

MVP에서는 과도한 Layout Model을 만들지 않고
현재 Aircraft 구성에 필요한 최소 구조를 우선합니다.

---

## 10. Flight

### 10.1 역할

`Flight`는 특정 날짜와 시간에 운항하는
KOKU Airline 내부 항공편입니다.

외부 실제 항공편은
이 Entity에 저장하지 않습니다.

---

### 10.2 주요 Column 초안

```text
flight
--------------------------------
id
flight_schedule_id
flight_number
route_id
aircraft_id
departure_local_date
departure_at
arrival_at
status
cancellation_reason
created_at
updated_at
```

`flight_schedule_id`는 운항 일정에서 자동 생성된 Flight가
어떤 `FlightSchedule`로부터 생성되었는지 식별합니다.

```text
자동 생성 Flight
→ flight_schedule_id = 해당 FlightSchedule ID

Admin 수동 생성 Flight
→ flight_schedule_id = NULL
```

`departure_local_date`는
Flight Number 중복 방지와 자동 생성 Idempotency를 위해 저장합니다.

```text
departure_at
+
Route의 departure Airport ZoneId
        ↓
departure_local_date
```

Java에서는 `LocalDate`,
MySQL에서는 `DATE`를 사용합니다.

`departure_local_date`는 Client가 임의로 전달하여 저장하는 값이 아니라
Backend가 `departure_at`과 출발 Airport의 `ZoneId`를 기준으로 계산합니다.

상태:

```text
SCHEDULED
CANCELLED
DEPARTED
```

---

### 10.3 Flight Number

Flight Number 형식:

```text
KO + 3자리 숫자
```

예:

```text
KO101
KO205
```

같은 Flight Number는
서로 다른 운항일에 반복해서 사용할 수 있습니다.

Flight Number 중복 여부는
출발 Airport의 Local Date를 기준으로 판단합니다.

Database에서는 다음 Composite Unique Constraint를 적용합니다.

```text
UNIQUE(
    flight_number,
    departure_local_date
)
```

예:

```text
2026-09-01 KO101
→ 가능

2026-09-02 KO101
→ 가능

2026-09-01 KO101
→ 중복이므로 불가
```

`departure_local_date`는
`departure_at`과 출발 Airport의 IANA `ZoneId`를 이용하여
Backend에서 계산한 값을 저장합니다.

성수기 임시 증편은
같은 Local Date에 기존 Flight와 같은 Flight Number를 사용하지 않고
별도의 Flight Number를 사용합니다.

---

### 10.4 Flight Date / Time

Flight의 절대 시각은 UTC 기준으로 저장합니다.

Backend에서는 시간 계산과 비교를 위해
Java `Instant`를 사용합니다.

Database에서는 MySQL `DATETIME(6)`을 사용합니다.

```text
departure_at
→ Java Instant
→ MySQL DATETIME(6)
→ UTC

arrival_at
→ Java Instant
→ MySQL DATETIME(6)
→ UTC
```

`DATETIME(6)` 자체에는 Time Zone 정보가 저장되지 않으므로
Application / JDBC / Hibernate의 Database Time Zone을
UTC 기준으로 통일합니다.

Airport는 별도로 IANA Time Zone ID를 가집니다.

예:

```text
ICN
→ Asia/Seoul

NRT
→ Asia/Tokyo
```

Flight의 Local Date / Time이 필요한 경우:

```text
Instant
+
Airport ZoneId
→
Local Date / Time
```

방식으로 계산합니다.

사용자에게 전달하는 API Date / Time은
Offset을 포함한 ISO-8601 형식을 사용합니다.

예:

```text
2026-09-10T09:30:00+09:00
```

Frontend에서는 한국 / 일본 Airport의 현지시각을
24시간제 `00:00 ~ 23:59` 형식으로 표시합니다.

시간대에 따른 Business Rule은
해당 Flight의 출발 Airport Local Date / Time을 기준으로 판단합니다.

예:

- 운임 시간대
- 운임 요일
- Passenger 탑승일 기준 연령
- ROUND_TRIP Date Rule
- Flight 출발일

단, 다음과 같은 현재 시각과의 비교는
Backend `Clock`을 기준으로 수행합니다.

- 신규 Reservation 2시간 제한
- Seat Hold 만료
- Reservation 취소 24시간 제한
- Flight `DEPARTED` 여부

항상 다음 조건을 만족해야 합니다.

```text
departure_at < arrival_at
```

---

### 10.5 Aircraft 일정 충돌

동일 Aircraft는
운항 시간이 충돌하는 둘 이상의 Flight에 배정할 수 없습니다.

MVP에서는 모든 Aircraft에
고정 `60분`의 Turnaround Time을 적용합니다.

동일 Aircraft의 연속 Flight는 다음 조건을 만족해야 합니다.

```text
previousFlight.arrival_at
+
60 minutes
<=
nextFlight.departure_at
```

시간 비교는 UTC 기준 Java `Instant`로 수행합니다.

Aircraft 충돌 검증에서는
`CANCELLED` Flight를 제외합니다.

Application에서 후보 Flight와 충돌하는 기존 Flight가 존재하는지
검증할 때의 논리 조건은 다음과 같습니다.

```text
same aircraft
AND status != CANCELLED
AND existing flight != current flight
AND
NOT (
    existing.arrival_at + 60분 <= candidate.departure_at
    OR
    candidate.arrival_at + 60분 <= existing.departure_at
)
```

위 조건에 해당하는 기존 Flight가 하나라도 존재하면
Aircraft 배정을 거부합니다.

충돌 검증은 최소 다음 작업에 적용합니다.

- FlightSchedule 기반 Flight 자동 생성
- Admin / SuperAdmin Flight 수동 생성
- Aircraft 변경
- `departure_at` 변경
- `arrival_at` 변경

Database의 단순 Unique Constraint만으로
시간 범위 충돌 전체를 보호하기 어렵기 때문에
Aircraft Schedule Conflict는 Application Validation을 기본으로 합니다.

조회 성능을 위해 다음 Index를 우선 검토합니다.

```text
flight(
    aircraft_id,
    status,
    departure_at,
    arrival_at
)
```

실제 Index 구성은 Query와 `EXPLAIN` 결과를 기준으로 조정합니다.

---

### 10.6 FlightSchedule

`FlightSchedule`은 KOKU Airline의 반복 운항 Template을 나타냅니다.

특정 날짜의 실제 예약 대상은 `FlightSchedule`이 아니라
이를 기준으로 생성된 `Flight`입니다.

주요 Column 초안:

```text
flight_schedule
--------------------------------
id
flight_number
route_id
default_aircraft_id
departure_local_time
arrival_local_time
arrival_day_offset
active
deactivated_at
created_at
updated_at
```

Column 의미:

- `flight_number`: 자동 생성할 Flight Number
- `route_id`: 운항 Route
- `default_aircraft_id`: 자동 생성 시 기본 Aircraft
- `departure_local_time`: 출발 Airport 현지 출발시각
- `arrival_local_time`: 도착 Airport 현지 도착시각
- `arrival_day_offset`: 출발 Local Date 대비 도착 Local Date 차이
- `active`: 자동 생성 사용 여부
- `deactivated_at`: 비활성화 시각

시간 Column은 다음 형식을 사용합니다.

```text
departure_local_time
→ Java LocalTime
→ MySQL TIME

arrival_local_time
→ Java LocalTime
→ MySQL TIME
```

`arrival_day_offset`는 MVP에서 다음 값을 사용합니다.

```text
0
→ 출발 Local Date와 같은 Date에 도착

1
→ 다음 Local Date에 도착
```

FlightSchedule 자체의 Local Time은
Route에 연결된 각 Airport의 `ZoneId`와 결합하여
실제 Flight의 `departure_at`, `arrival_at`을 계산합니다.

자동 생성된 Flight는 다음 관계를 가집니다.

```text
FlightSchedule 1 : N Flight
```

운항 일정 변경은 이미 생성된 Flight를 수정하지 않습니다.

따라서 `Flight`에는 생성 당시 확정된 다음 값을 독립적으로 저장합니다.

- `flight_number`
- `route_id`
- `aircraft_id`
- `departure_local_date`
- `departure_at`
- `arrival_at`

`FlightSchedule` 수정 후에는
앞으로 새로 생성되는 Flight부터 변경된 값을 사용합니다.

---

#### 10.6.1 FlightScheduleDay

하나의 FlightSchedule은
하나 이상의 운항 요일을 가질 수 있습니다.

Supporting Table 초안:

```text
flight_schedule_day
--------------------------------
flight_schedule_id
day_of_week
```

`day_of_week`은 다음 Canonical Enum 값을 사용합니다.

```text
MONDAY
TUESDAY
WEDNESDAY
THURSDAY
FRIDAY
SATURDAY
SUNDAY
```

동일 FlightSchedule에 같은 요일을 중복 등록할 수 없습니다.

```text
UNIQUE(
    flight_schedule_id,
    day_of_week
)
```

---

### 10.7 Flight 자동 생성 Data 규칙

Flight 자동 생성 Scheduler는
`active = true`인 FlightSchedule을 조회합니다.

MVP의 자동 생성 범위는
현재 Month를 기준으로 세 번째 다음 Calendar Month의 마지막 날까지입니다.

예:

```text
2026-08-26 실행
→ 2026-11-30까지 확보

2026-09-01 실행
→ 2026-12-31까지 확보
```

Scheduler는 하루 1회 실행하며,
실행할 때마다 전체 필요한 범위를 다시 확인합니다.

자동 생성 Flight에는
FlightSchedule의 `default_aircraft_id`를 사용합니다.

생성 전 반드시 다음을 검증합니다.

```text
Flight Number / Local Date 중복
Route 활성 상태
Aircraft 활성 상태
Aircraft Schedule Conflict
Turnaround 60분
```

FlightSchedule에서 자동 생성된 Flight는
다음 Constraint로 같은 Schedule / Local Date에
둘 이상의 Flight가 생성되지 않도록 보호합니다.

```text
UNIQUE(
    flight_schedule_id,
    departure_local_date
)
```

`flight_schedule_id`가 `NULL`인 Admin 수동 생성 Flight에는
위 Schedule 기반 Unique Constraint가 적용되지 않습니다.

MySQL의 Unique Constraint는 `NULL` 값을 서로 동일한 값으로 취급하지 않으므로
수동 Flight를 여러 건 저장할 수 있습니다.

모든 Flight에는 별도로 다음 Constraint도 적용합니다.

```text
UNIQUE(
    flight_number,
    departure_local_date
)
```

따라서 다음 두 종류의 중복을 각각 방지합니다.

```text
FlightSchedule + Local Date
→ Scheduler 중복 생성 방지

Flight Number + Local Date
→ 동일 운항일 Flight Number 중복 방지
```

이미 생성된 Flight가 다음 상태여도
해당 FlightSchedule / Local Date는 이미 생성된 것으로 처리합니다.

```text
SCHEDULED
CANCELLED
DEPARTED
```

따라서 `CANCELLED` Flight를
Scheduler가 다시 생성하지 않습니다.

FlightSchedule 수정 시에도
이미 존재하는 Flight를 UPDATE하지 않습니다.

```text
기존 Flight
→ 유지

향후 새로 생성되는 Flight
→ 변경된 FlightSchedule 적용
```

Admin 또는 SuperAdmin이 기존 Flight를 수동 수정한 경우에도
Scheduler가 해당 Flight를 다시 덮어쓰지 않습니다.

---

## 11. Flight Seat

### 11.1 역할

Domain의 `Seat`는
특정 Flight에서 실제 예약 가능한 좌석 상태를 나타냅니다.

Aircraft Seat Configuration과
실제 Flight 예약 상태를 분리합니다.

Flight 생성 시 Aircraft의 Seat Layout을 기준으로
Flight별 Seat를 구성하는 방식을 기본 방향으로 합니다.

---

### 11.2 주요 Column 초안

```text
seat
--------------------------------
id
flight_id
seat_no
row_no
seat_column
seat_class
status
held_reservation_id
created_at
updated_at
```

Column 의미:

- `flight_id`: Seat가 속한 Flight
- `seat_no`: 해당 Flight의 Seat Number
- `row_no`: Seat Row Snapshot
- `seat_column`: Seat Column Snapshot
- `seat_class`: Flight 생성 시점의 SeatClass Snapshot
- `status`: 현재 Seat 상태
- `held_reservation_id`: 현재 Seat를 Hold 중인 `PENDING` Reservation

`seat_class`는 다음 값을 사용합니다.

```text
ECONOMY
PREMIUM_ECONOMY
BUSINESS
```

Flight Seat는
Flight 생성 시 해당 Aircraft의 `AircraftSeat`를 기준으로 생성합니다.

최소 다음 값을 Snapshot으로 복사합니다.

```text
AircraftSeat.seat_no
→ Seat.seat_no

AircraftSeat.row_no
→ Seat.row_no

AircraftSeat.seat_column
→ Seat.seat_column

AircraftSeat.seat_class
→ Seat.seat_class
```

따라서 이후 AircraftSeat 구성이 변경되더라도
이미 생성된 Flight의 Seat Snapshot은 자동 변경하지 않습니다.

`held_reservation_id`는 Nullable FK로 사용합니다.

```text
AVAILABLE
→ held_reservation_id = NULL

HELD
→ held_reservation_id = Hold 중인 Reservation ID

RESERVED
→ held_reservation_id = NULL

UNAVAILABLE
→ held_reservation_id = NULL
```

Seat Hold의 만료시각은 Seat에 중복 저장하지 않습니다.

```text
Seat.held_until
→ 사용하지 않음

Reservation.hold_expires_at
→ Reservation 전체 Hold 만료시각
```

구체적인 FK:

```text
seat.held_reservation_id
→ reservation.id
```

Seat가 `HELD`에서 다른 상태로 전환될 때는
`held_reservation_id`를 `NULL`로 정리합니다.

---

### 11.3 Unique Constraint

하나의 Flight에서
동일 Seat Number는 하나만 존재해야 합니다.

```text
UNIQUE(
  flight_id,
  seat_no
)
```

---

### 11.4 Seat 상태

기본 생성 상태:

```text
AVAILABLE
```

Reservation 시작 성공:

```text
Seat.status
AVAILABLE → HELD

Seat.held_reservation_id
NULL → Reservation ID
```

Mock Payment 성공:

```text
Seat.status
HELD → RESERVED

Seat.held_reservation_id
Reservation ID → NULL
```

Hold 만료 또는 `PENDING` Reservation 진행 취소:

```text
Seat.status
HELD → AVAILABLE

Seat.held_reservation_id
Reservation ID → NULL
```

정상 Reservation 취소:

```text
Seat.status
RESERVED → AVAILABLE
```

운영상 Seat 판매 중지:

```text
AVAILABLE → UNAVAILABLE
```

운영 제한 해제:

```text
UNAVAILABLE → AVAILABLE
```

`HELD` 또는 `RESERVED` Seat를
직접 `UNAVAILABLE`로 변경하지 않습니다.

Flight 취소에서 이미 출발한 Flight의 Seat는
Domain Policy에 따라 변경하지 않습니다.

---

### 11.5 Seat 동시성

Seat 확보의 MVP 동시성 제어는
`04-system-design.md`에 따라
MySQL Pessimistic Row Lock을 사용합니다.

Spring Data JPA에서는
`PESSIMISTIC_WRITE`를 사용합니다.

논리적인 Query는 다음과 같습니다.

```sql
SELECT *
FROM seat
WHERE id IN (...)
ORDER BY id ASC
FOR UPDATE;
```

Repository 구현에서는
선택한 Seat ID 목록을 기준으로
필요한 Seat Row를 하나의 Query에서 Lock하는 방향을 사용합니다.

Lock 순서는 다음과 같이 고정합니다.

```text
seat_id ASC
```

Reservation 시작 기본 흐름:

```text
요청 Seat ID 수집
        |
        v
중복 ID 제거 및 Validation
        |
        v
seat_id ASC 정렬
        |
        v
PESSIMISTIC_WRITE 조회
        |
        v
요청 Seat 수 == 조회 Seat 수 검증
        |
        v
모든 Seat 상태 검증
        |
        +-- 전부 AVAILABLE
        |       → 계속 진행
        |
        +-- 하나라도 확보 불가
                → 전체 실패
```

Lock 대상은 선택한 Seat Row로 제한합니다.

다음 Resource를 Seat 확보 목적으로 Lock하지 않습니다.

- Flight 전체
- Aircraft 전체
- Seat Table 전체

`ROUND_TRIP`에서는
출국 및 귀국 Flight의 Seat ID를 하나의 목록으로 합친 뒤
동일하게 `seat_id ASC` 순서로 Lock합니다.

선택한 Seat 중 하나라도 다음 상태이면
Reservation 시작 전체를 실패합니다.

- `HELD`
- `RESERVED`
- `UNAVAILABLE`

일부 Seat만 `HELD`로 만드는 Partial Success는 허용하지 않습니다.

여러 Backend Instance가 동일 MySQL을 사용하는 경우에도
동일한 Database Row Lock을 사용합니다.

Redis Distributed Lock은
MVP Seat 확보 경로에 사용하지 않습니다.

---

### 11.6 Flight Seat 생성 Transaction

Flight 생성과 해당 Flight의 Seat Snapshot 생성은
하나의 Transaction에서 처리합니다.

```text
Flight 생성
        |
        v
AircraftSeat 조회
        |
        v
Seat Snapshot 생성
        |
        v
Commit
```

Seat Snapshot 생성 중 하나라도 실패하면
Flight 생성도 Rollback합니다.

자동 생성 Flight와
Admin / SuperAdmin 수동 생성 Flight 모두 동일하게 적용합니다.

따라서 정상 생성된 Flight가
Seat Snapshot 없이 존재하는 상태를 허용하지 않습니다.

Aircraft를 변경할 수 있는 Flight의 경우에도
Aircraft 변경과 Seat Snapshot 재생성을
하나의 Transaction으로 처리합니다.

```text
기존 Seat Snapshot 제거
+
Aircraft 변경
+
새 AircraftSeat 기준 Seat Snapshot 생성
        |
        v
전체 성공 → Commit

일부 실패 → Rollback
```

Aircraft 변경 자체의 허용 조건은
10장 Flight 정책과 Admin Flight API 정책을 따릅니다.

---

## 12. Reservation

### 12.1 역할

`Reservation`은 한 Member가 생성한
하나의 예약 단위를 나타냅니다.

상태:

```text
PENDING
CONFIRMED
CANCELLED
```

여행 유형:

```text
ONE_WAY
ROUND_TRIP
```

---

### 12.2 주요 Column 초안

```text
reservation
--------------------------------
id
reservation_no
member_id
trip_type
status
total_amount
hold_expires_at
cancel_reason
created_at
updated_at
```

금액 Column은 Java에서 `BigDecimal`을 사용합니다.

MVP는 KRW 기준으로 최종 금액을 1원 단위로 확정하므로
Database의 최종 금액 Column은 정수 원화 금액을 정확하게 저장할 수 있는
`DECIMAL` Type을 사용합니다.

기본 Mapping:

```text
Reservation.total_amount

Java
→ BigDecimal

MySQL
→ DECIMAL(15, 0)
```

운임 계산 중 소수 원 단위가 발생하는 경우
최종 Passenger / Flight별 운임은
1원 단위에서 `RoundingMode.HALF_UP`으로 반올림합니다.

```text
중간 계산
→ BigDecimal

Passenger / Flight별 최종 운임
→ scale(0, RoundingMode.HALF_UP)

Reservation.total_amount
→ 반올림이 완료된 각 운임의 합계
```

반올림 시점과 방식은
Application 전체에서 동일하게 적용합니다.

운임 계산 과정에서도
`double` 또는 `float`을 사용하지 않습니다.

SeatClass 고정 배율은 Application의
`SeatClassFarePolicy`에서 `BigDecimal` 값으로 제공합니다.

```text
ECONOMY
→ 1.0

PREMIUM_ECONOMY
→ 1.3

BUSINESS
→ 2.0
```

최종 Reservation 금액은
모든 Passenger / Flight별 운임을 계산한 뒤 합산하여
`PENDING` Reservation 생성 시 Snapshot으로 저장합니다.

---

### 12.3 Reservation 번호

Database PK와 별도로
사용자 공개 Reservation 번호를 저장합니다.

형식:

```text
KOKU-YYYYMMDD-XXXXXX
```

예:

```text
KOKU-20260821-A7F3K9
```

Random 영역에서는
사용자 혼동을 줄이기 위해 다음 문자를 제외합니다.

```text
I
O
0
1
```

Reservation 번호는:

- `PENDING` Reservation 정상 생성 시 발급
- 생성 후 변경하지 않음
- 재사용하지 않음
- 시스템 전체에서 중복되지 않음

을 원칙으로 합니다.

Database Constraint:

```text
UNIQUE(reservation_no)
```

---

### 12.4 Reservation 번호 Collision

Application에서 Reservation 번호를 생성한 뒤
Database Unique Constraint를 최종 보호 장치로 사용합니다.

Collision 발생 시
제한된 횟수 내에서 새로운 번호를 생성하여 재시도합니다.

구체적인 재시도 횟수는 구현 단계에서 결정합니다.

Reservation 번호 생성 실패 때문에
중복 Reservation 번호를 허용해서는 안 됩니다.

---

### 12.5 Reservation 금액

`total_amount`는
해당 Reservation의 최종 결제 예정 금액입니다.

Domain Policy에 따라
모든 Seat 확보에 성공하여 `PENDING` Reservation이 생성되는 시점에 확정합니다.

확정 이후 다음 상황에서도 변경하지 않습니다.

- Mock Payment 재시도
- Payment 실패 후 재시도

실제 환불 처리 시에도
원 Reservation 금액 자체를 변경하지 않습니다.

Payment 상태를 통해 환불 이력을 표현합니다.

---

### 12.6 Hold 만료

`PENDING` Reservation은
Seat Hold 만료시각을 가져야 합니다.

```text
hold_expires_at
→ Java Instant
→ MySQL DATETIME(6)
→ UTC
```

Hold 만료시각은
Domain Policy에 따라 최대 1시간이며,
Reservation에 포함된 가장 빠른 Flight 출발 예정시각을 초과할 수 없습니다.

Hold 만료 여부는
Backend `Clock`의 현재 `Instant`와 비교하여 판단합니다.

---

## 13. Reservation과 Flight 관계

### 13.1 필요성

`ONE_WAY` Reservation은 Flight 1개를 가집니다.

`ROUND_TRIP` Reservation은 Flight 2개를 가집니다.

따라서 Reservation에 단순히 하나의 `flight_id`만 저장하지 않습니다.

---

### 13.2 ReservationFlight

Reservation과 Flight의 관계를 표현하기 위해
`ReservationFlight` Supporting Model을 사용합니다.

초안:

```text
reservation_flight
--------------------------------
id
reservation_id
flight_id
journey_role
sequence
created_at
```

`journey_role` 초안:

```text
OUTBOUND
RETURN
```

---

### 13.3 ONE_WAY

편도:

```text
Reservation
  |
  └─ ReservationFlight
       journey_role = OUTBOUND
       sequence = 1
```

방향이 일본 → 한국이라고 하더라도
ONE_WAY의 단일 Flight는 Reservation Journey 기준
여정 시작 Flight이므로 `OUTBOUND` 역할로 표현할 수 있습니다.

---

### 13.4 ROUND_TRIP

왕복:

```text
Reservation
  |
  +─ ReservationFlight
  |    journey_role = OUTBOUND
  |    sequence = 1
  |
  └─ ReservationFlight
       journey_role = RETURN
       sequence = 2
```

출국과 귀국 Flight는
하나의 Reservation에 포함됩니다.

Flight마다 별도의 Reservation 번호를 만들지 않습니다.

---

### 13.5 Constraint

하나의 Reservation 안에서
동일 Flight가 중복 연결되지 않도록 보호합니다.

초안:

```text
UNIQUE(
  reservation_id,
  flight_id
)
```

`ROUND_TRIP`의 정확한 Flight 개수와 Journey Role Validation은
Application에서 Domain Policy에 따라 검증합니다.

---

## 14. Passenger

### 14.1 역할

`Passenger`는 실제 탑승 대상에 해당하는
테스트용 탑승객 정보를 나타냅니다.

`Member`와 `Passenger`는 별개의 Entity입니다.

Member가 자신이 아닌 다른 Passenger를 포함한 Reservation을 생성할 수 있습니다.

---

### 14.2 주요 Column 초안

```text
passenger
--------------------------------
id
last_name
first_name
birth_date
gender
nationality
test_passport_no
test_passport_country
test_passport_expiry_date
created_at
updated_at
```

날짜 Column Type:

```text
birth_date
→ Java LocalDate
→ MySQL DATE

test_passport_expiry_date
→ Java LocalDate
→ MySQL DATE
```

Passenger의 생년월일과 Test Passport 만료일은
특정 순간의 절대 시각이 아니라 날짜 자체를 의미하므로
`Instant` 또는 `DATETIME(6)`을 사용하지 않습니다.

---

### 14.3 영문 이름

Passenger 이름은
테스트용 여권 영문명 형식을 기준으로 저장합니다.

구체적인 허용 문자, 길이 및 Validation은
구현 전에 확정합니다.

---

### 14.4 생년월일과 연령 구분

Passenger의 Adult / Child / Infant 구분을
Passenger Table에 고정 Enum으로 저장하지 않는 것을 기본 방향으로 합니다.

이유:

`ROUND_TRIP`에서는 같은 Passenger라도
각 Flight 탑승일에 따라 연령 구분이 달라질 수 있습니다.

따라서 연령 Category는
각 Flight의 탑승일과 Passenger `birth_date`를 기준으로 계산합니다.

---

## 15. 테스트용 여권정보

### 15.1 기본 구조

사용자는 실제 Passport 정보를 입력하지 않습니다.

Passenger 기본 정보 입력 후
시스템이 테스트용 여권정보를 생성합니다.

생성 항목:

- 테스트용 Passport Number
- 발급국
- 만료일

---

### 15.2 발급국

초안:

```text
test_passport_country = Passenger nationality
```

---

### 15.3 만료일

Domain Policy 기준:

```text
생성일 + 5년
```

생성된 만료일은
Reservation에 포함된 모든 Flight의 탑승일 이후여야 합니다.

유효하지 않으면
Reservation을 `CONFIRMED`로 변경할 수 없습니다.

---

### 15.4 Passport Number 생성

테스트용 Passport Number는
실제 여권번호가 아님을 명확하게 구분할 수 있는
시스템 생성 값을 사용합니다.

구체적인 형식은 구현 전에 확정합니다.

사용자가 직접 입력하거나 수정할 수 없습니다.

---

### 15.5 저장 및 보호

테스트용 Passport Number는
실제 개인정보는 아니지만 민감정보와 유사하게 취급합니다.

다음 원칙을 적용합니다.

- Log 출력 금지
- 불필요한 API Response 제외
- 불필요한 관리자 화면 노출 금지

Database Encryption 적용 여부와
API Masking 방식은 아직 확정하지 않습니다.

`04-system-design.md`의 보안 미확정 항목과 함께 결정합니다.

---

## 16. ReservationPassenger

### 16.1 역할

Reservation과 Passenger의 관계를 표현합니다.

하나의 Reservation은
한 명 이상의 Passenger를 가질 수 있습니다.

초안:

```text
Reservation 1 : N ReservationPassenger
Passenger   1 : N ReservationPassenger
```

---

### 16.2 주요 Column 초안

```text
reservation_passenger
--------------------------------
id
reservation_id
passenger_id
sequence
created_at
```

Reservation 안에서 동일 Passenger의
중복 연결을 허용하지 않는 것을 기본 방향으로 합니다.

초안:

```text
UNIQUE(
  reservation_id,
  passenger_id
)
```

---

## 17. PassengerFlight

### 17.1 필요성

ROUND_TRIP에서는 Passenger의 상태가
Flight마다 달라질 수 있습니다.

예:

- 출국 시 Infant
- 귀국 시 Child

또한 Flight별로 Seat와 Infant Companion 관계를 표현해야 합니다.

따라서 Passenger와 Flight 사이의
Flight별 예약 속성을 별도 관계로 표현하는 방안을 사용합니다.

---

### 17.2 주요 Column 초안

```text
passenger_flight
--------------------------------
id
reservation_id
passenger_id
flight_id
seat_id
companion_passenger_id
created_at
```

---

### 17.3 Seat

해당 Flight에서 Seat가 필요한 Passenger는
`seat_id`를 가집니다.

해당 Flight에서 Infant인 Passenger는
별도 Seat를 사용하지 않으므로:

```text
seat_id = NULL
```

을 허용합니다.

Seat 필요 여부는
Passenger의 생년월일과 Flight 탑승일을 기준으로
Backend에서 판단합니다.

---

### 17.4 Infant Companion

해당 Flight에서 Infant인 경우
Adult Passenger와 연결되어야 합니다.

초안:

```text
companion_passenger_id
```

이 값은 동일 Reservation에 포함된 Passenger를 참조합니다.

지정된 Companion은
해당 Flight에서 Adult 조건을 만족해야 합니다.

Adult 1명당
해당 Flight에서 최대 Infant 1명만 연결할 수 있습니다.

---

### 17.5 ROUND_TRIP Companion

Domain Policy에 따라
Passenger가 하나 이상의 Flight에서 Infant인 경우
동일 Companion Adult를 기본으로 사용합니다.

그러나 Validation은
각 Flight별로 수행합니다.

한 Flight에서 Companion이 Adult 조건을 만족하지 못하면
Reservation을 시작할 수 없습니다.

---

### 17.6 Child Seat Validation

Passenger가 해당 Flight에서 Child인 경우
동일 Flight에 Adult Passenger가 최소 1명 있어야 합니다.

Child의 Seat는
동일 Flight의 Adult Seat 중 최소 하나와
Domain Policy상 인접 Seat 조건을 만족해야 합니다.

이 Validation은 Application / Domain Logic에서 수행합니다.

---

## 18. Payment

### 18.1 역할

`Payment`는 Mock Payment의
개별 결제 시도를 나타냅니다.

하나의 Reservation은
여러 Payment 이력을 가질 수 있습니다.

```text
Reservation 1 : N Payment
```

별도의 `PaymentAttempt` Entity는 사용하지 않습니다.

각 Payment 자체가 하나의 Mock 결제 시도를 나타냅니다.

---

### 18.2 주요 Column 초안

```text
payment
--------------------------------
id
reservation_id
attempt_no
status
amount
idempotency_key
created_at
updated_at
```

상태:

```text
PENDING
SUCCESS
FAILED
CANCELLED
REFUNDED
```

---

### 18.3 Payment 횟수

하나의 Reservation당
최대 3회의 Payment를 생성할 수 있습니다.

예:

```text
attempt_no = 1
attempt_no = 2
attempt_no = 3
```

세 번째 Payment까지 실패하면
추가 Payment를 생성하지 않습니다.

---

### 18.4 Payment 금액

각 Payment의 `amount`는
Reservation에 확정된 `total_amount`를 그대로 Snapshot으로 저장합니다.

```text
Reservation.total_amount
        |
        v
Payment.amount
```

Payment 생성 시점에
운임을 다시 계산하지 않습니다.

금액 Mapping:

```text
Java
→ BigDecimal

MySQL
→ DECIMAL(15, 0)
```

Payment 재시도 과정에서도
Reservation 금액을 다시 계산하지 않습니다.

따라서 동일 Reservation의 정상적인 Payment 시도는
모두 동일한 확정 금액을 사용합니다.

---

### 18.5 Payment 성공

하나의 Reservation에는
최대 하나의 `SUCCESS` Payment만 존재할 수 있어야 합니다.

Application에서 먼저 검증하고,
가능한 Database Constraint 또는 Transaction 구조를 통해 보호합니다.

구체적인 Database 보장 방식은
MySQL 특성과 실제 구현 구조를 검토한 후 확정합니다.

---

### 18.6 실패 이력

`FAILED` Payment는 삭제하거나 덮어쓰지 않습니다.

각 실패 Payment는
결제 시도 이력으로 유지합니다.

---

### 18.7 환불

Reservation 취소로 Mock 환불이 발생하면
성공한 Payment를:

```text
SUCCESS → REFUNDED
```

상태로 변경합니다.

실제 금융 환불은 발생하지 않습니다.

---

## 19. Payment Idempotency

### 19.1 목적

동일한 Mock Payment 요청이
Network Retry 또는 중복 Click 등에 의해 반복되어도

- 새로운 Payment가 중복 생성되지 않고
- Payment 시도 횟수가 중복 증가하지 않으며
- Reservation이 중복 확정되지 않아야 합니다.

---

### 19.2 Idempotency Key

Mock Payment 요청에는
Idempotency Key를 사용하는 방안을 기본안으로 합니다.

예:

```text
Idempotency-Key: 550e8400-e29b-41d4-a716-446655440000
```

구체적인 Header 이름은 API Contract 확정 시 결정합니다.

---

### 19.3 Database 보호

초안에서는 Payment에
`idempotency_key`를 저장합니다.

동일 Reservation에서
동일 Idempotency Key를 중복 사용할 수 없도록 합니다.

초안 Constraint:

```text
UNIQUE(
  reservation_id,
  idempotency_key
)
```

---

### 19.4 동일 요청 재처리

동일 Idempotency Key 요청이 다시 들어오면
새로운 Payment를 생성하지 않고
기존 처리 결과를 반환하는 방향을 사용합니다.

동일 Key에 서로 다른 Request Body가 전달된 경우의 처리 방식은
API 구현 전에 추가로 확정합니다.

---

## 20. AuditLog

### 20.1 역할

운영상 중요한 관리자 Action을
Audit Log로 영구 기록합니다.

최소 대상:

- Flight 취소
- SuperAdmin Reservation 강제 취소
- 중요 Master Data 생성
- 중요 Master Data 수정
- 중요 Master Data 비활성화

---

### 20.2 주요 Column 초안

```text
audit_log
--------------------------------
id
actor_member_id
action_type
target_type
target_id
reason
created_at
```

---

### 20.3 Actor

`actor_member_id`는
Action을 수행한 Admin 또는 SuperAdmin을 참조합니다.

Audit 이력 보존을 위해
Member가 이후 `WITHDRAWN` 또는 다른 상태가 되어도
과거 기록은 유지합니다.

---

### 20.4 Target

다양한 관리 대상을 기록할 수 있도록
초안에서는 다음 구조를 검토합니다.

```text
target_type
target_id
```

예:

```text
FLIGHT
123

RESERVATION
456
```

실제 FK를 직접 사용하는 여러 Audit Table보다
단일 AuditLog 구조를 우선 검토합니다.

---

### 20.5 Reason

Domain Policy에서 사유 입력을 요구하는 Action은
`reason`이 필수입니다.

예:

- Flight 취소
- SuperAdmin Reservation 강제 취소

---

## 21. 주요 Unique Constraint 초안

MVP에서 최소 다음 데이터 정합성을
Database 수준에서도 보호하는 방향을 사용합니다.

```text
member.email
    UNIQUE

auth_account(provider, provider_subject)
    UNIQUE

airport.iata_code
    UNIQUE

route(departure_airport_id, arrival_airport_id)
    UNIQUE

aircraft_seat(aircraft_id, seat_no)
    UNIQUE

flight_schedule_day(flight_schedule_id, day_of_week)
    UNIQUE

flight(flight_schedule_id, departure_local_date)
    UNIQUE

flight(flight_number, departure_local_date)
    UNIQUE

seat(flight_id, seat_no)
    UNIQUE

reservation.reservation_no
    UNIQUE

reservation_flight(reservation_id, flight_id)
    UNIQUE

reservation_passenger(reservation_id, passenger_id)
    UNIQUE

payment(reservation_id, idempotency_key)
    UNIQUE
```

정확한 Constraint 이름과
추가 Composite Unique Key는
ERD 확정 시 결정합니다.

---

## 22. Index 설계 원칙

### 22.1 기본 원칙

Index는 모든 Column에 임의로 추가하지 않습니다.

다음 기준을 우선 검토합니다.

- 실제 조회 조건
- Join FK
- 정렬 조건
- Unique Constraint
- 자주 사용하는 상태 Filter

---

### 22.2 초기 검토 대상

예:

```text
member(email)

auth_account(member_id)
auth_account(provider, provider_subject)

route(departure_airport_id, arrival_airport_id)

flight(route_id, departure_at)
flight(aircraft_id, departure_at)
flight(status, departure_at)

flight_schedule(active)
flight_schedule(route_id)
flight_schedule(default_aircraft_id)
flight(flight_schedule_id, departure_local_date)
flight(flight_number, departure_local_date)
flight(aircraft_id, status, departure_at, arrival_at)

seat(flight_id, status)

reservation(member_id, created_at)
reservation(member_id, status)
reservation(reservation_no)
reservation(status, hold_expires_at)

payment(reservation_id, created_at)
payment(reservation_id, status)
```

실제 Index는
Query 및 `EXPLAIN` 결과를 검토해 확정합니다.

---

## 23. API 기본 원칙

### 23.1 Base Path

REST API는 공통 Prefix를 사용합니다.

초안:

```text
/api/v1
```

MVP에서 Versioning을 사용하지 않을 경우
단순 `/api`도 가능하나,
초안에서는 `/api/v1`을 기준으로 합니다.

최종 Prefix는 구현 시작 전에 확정합니다.

---

### 23.2 Content Type

기본 Request / Response:

```text
application/json
```

---

### 23.3 Naming

JSON Field는
Frontend / Backend Contract에서 일관된 Naming을 사용합니다.

초안:

```text
camelCase
```

예:

```json
{
  "reservationNo": "KOKU-20260821-A7F3K9",
  "tripType": "ROUND_TRIP",
  "totalAmount": 540000
}
```

Database Column Naming은:

```text
snake_case
```

를 기본 방향으로 합니다.

---

### 23.4 내부 PK 노출

API URL 또는 Response에서
내부 PK를 기술적 식별자로 사용할 수 있지만,
사용자에게 공개되어야 하는 업무 식별자는 별도로 사용합니다.

Reservation 사용자 조회는
가능하면 공개 `reservationNo`를 기준으로 제공하는 방안을 우선합니다.

내부 관리자 API에서 PK 사용이 필요한 경우에도
Frontend에 업무 식별자와 혼동되게 표시하지 않습니다.

---

## 24. 공통 API Response

### 24.1 성공 Response

모든 API를 반드시 하나의 Wrapper로 감싸야 하는지는
구현 전에 최종 결정합니다.

초안에서는 Resource 중심의 Response를 우선합니다.

예:

```json
{
  "reservationNo": "KOKU-20260821-A7F3K9",
  "status": "CONFIRMED"
}
```

불필요하게 다음과 같은 중복 Wrapper를 강제하지 않습니다.

```json
{
  "success": true,
  "data": {
  }
}
```

필요성이 확인되면 공통 Response 구조를 추가합니다.

---

### 24.2 Error Response

Error Response는
일관된 구조를 사용합니다.

초안:

```json
{
  "code": "SEAT_NOT_AVAILABLE",
  "message": "선택한 좌석을 사용할 수 없습니다.",
  "details": []
}
```

필수 기본 Field:

- `code`
- `message`

`details`는 Validation Error 등 필요한 경우 사용합니다.

---

### 24.3 Error Message와 i18n

Backend Error `code`는
Locale과 독립적인 영문 Canonical Value를 사용합니다.

Frontend는 Error Code를 기준으로
`ko` / `ja` 사용자 메시지를 표시할 수 있습니다.

Backend가 사용자 UI 문구를
Business Logic 안에 직접 Hard Coding하지 않습니다.

---

## 25. 인증 API 초안

### 25.1 회원가입

```text
POST /api/v1/auth/signup
```

Request 초안:

```json
{
  "email": "user@test.com",
  "password": "Password1!"
}
```

처리:

1. Email 정규화
2. 형식 Validation
3. 중복 Member 확인
4. Password 정책 검증
5. Member 생성
6. LOCAL AuthAccount 생성

Response에 Password Hash를 포함하지 않습니다.

---

### 25.2 LOCAL 로그인

```text
POST /api/v1/auth/login
```

Request:

```json
{
  "email": "user@test.com",
  "password": "Password1!"
}
```

처리:

1. Email 정규화
2. LOCAL AuthAccount 조회
3. Password 검증
4. Member 상태 검증
5. Access Token 발급
6. Refresh Token 발급
7. Refresh Token Hash를 Redis에 저장
8. Refresh Token Cookie 설정
9. CSRF Token Cookie 설정

Access Token의 유효기간은 **30분**입니다.

Access Token은 Response Body를 통해 전달하고,
Frontend Memory에서 관리합니다.

Response 예:

```json
{
  "accessToken": "<Access Token>",
  "tokenType": "Bearer",
  "expiresIn": 1800
}
```

Refresh Token은 Response Body에 포함하지 않습니다.

Refresh Token은 다음 속성을 가진 Cookie로 전달합니다.

```text
HttpOnly = true
Secure = true
SameSite = 적용
Path = 인증 관련 Endpoint 범위
TTL = 7일
```

CSRF Token은 Refresh Token과 별도의 Cookie로 전달하며,
Frontend에서 CSRF 보호 요청 Header를 구성할 수 있도록
JavaScript에서 읽을 수 있어야 합니다.

Password, Password Hash, Refresh Token Hash는
Response Body에 포함하지 않습니다.

---

### 25.3 Token 재발급

```text
POST /api/v1/auth/refresh
```

Refresh Token은 Request Body가 아니라
HttpOnly / Secure Cookie를 통해 전달합니다.

CSRF 보호를 위해
CSRF Token을 Request Header에 함께 전달합니다.

예:

```text
Cookie
refresh_token=<Refresh Token>
csrf_token=<CSRF Token>

Header
X-CSRF-TOKEN: <CSRF Token>
```

Backend는 다음 순서로 요청을 검증합니다.

```text
CSRF 검증
        ↓
Refresh Token Signature / 만료시간 검증
        ↓
Refresh Token Hash 계산
        ↓
Redis 저장 정보와 비교
        ↓
기존 Refresh Token 폐기
        ↓
새 Access Token 발급
        ↓
새 Refresh Token 발급
        ↓
새 Refresh Token Hash Redis 저장
```

Access Token 재발급 성공 시
Refresh Token Rotation을 적용합니다.

기존 Refresh Token은 재사용할 수 없습니다.

새 Access Token은 Response Body로 반환합니다.

Response 예:

```json
{
  "accessToken": "<New Access Token>",
  "tokenType": "Bearer",
  "expiresIn": 1800
}
```

새 Refresh Token은
새로운 HttpOnly / Secure Cookie로 전달합니다.

Refresh Token 원문 또는 Hash를
Response Body에 포함하지 않습니다.

---

### 25.4 로그아웃

```text
POST /api/v1/auth/logout
```

Logout은 Refresh Token Cookie를 사용하는 Endpoint이므로
CSRF 보호를 적용합니다.

Request에는 다음 정보가 사용됩니다.

```text
Cookie
refresh_token=<Refresh Token>
csrf_token=<CSRF Token>

Header
X-CSRF-TOKEN: <CSRF Token>
```

기본 처리:

```text
CSRF 검증
        ↓
Refresh Token 확인
        ↓
Redis Refresh Token 정보 삭제
        ↓
Refresh Token Cookie 제거
        ↓
CSRF Cookie 정리
```

Access Token은 Server-side Blacklist에 등록하지 않습니다.

Frontend는 Logout 성공 시
Memory에서 Access Token을 제거합니다.

Refresh Token이 이미 만료되었거나
Redis에 존재하지 않는 경우에도
Client의 인증 Cookie 정리는 수행할 수 있어야 합니다.

정상 처리 Response는 Body가 필요하지 않으므로
`204 No Content` 사용을 기본 방향으로 합니다.

---

### 25.5 Google OAuth

Google OAuth 시작 및 Callback URL은
Spring Security OAuth2 Client 구조를 사용합니다.

구체적인 URI는
Spring Security Configuration 확정 시 문서화합니다.

---

## 26. Member API 초안

### 26.1 내 정보 조회

```text
GET /api/v1/members/me
```

---

### 26.2 비밀번호 변경

LOCAL AuthAccount가 존재하는 Member만 사용할 수 있습니다.

```text
PATCH /api/v1/members/me/password
```

인증된 Member만 요청할 수 있으며,
Access Token은 `Authorization` Header를 통해 전달합니다.

Request:

```json
{
  "currentPassword": "CurrentPassword1!",
  "newPassword": "NewPassword1!"
}
```

처리:

1. 현재 Member의 LOCAL AuthAccount 조회
2. 현재 Password 재인증
3. 새 Password 정책 검증
4. 새 Password Hash 생성
5. Password Hash 변경

현재 Password 검증에 실패하면
Password 변경을 거부합니다.

`GOOGLE` AuthAccount만 존재하고
`LOCAL` AuthAccount가 없는 Member에게는
해당 API를 허용하지 않습니다.

Password 및 Password Hash는
Response에 포함하지 않습니다.

---

### 26.3 회원 탈퇴

```text
DELETE /api/v1/members/me
```

실제 DELETE Query를 의미하지 않습니다.

Domain Policy에 따라:

```text
Member.status
ACTIVE → WITHDRAWN
```

으로 변경합니다.

탈퇴 가능 여부는 Backend에서
Reservation 상태와 Flight 출발 여부를 기준으로 검증합니다.

---

## 27. KOKU Flight 검색 API

### 27.1 편도 검색

초안:

```text
GET /api/v1/flights
```

Query 예:

```text
departure=ICN
arrival=NRT
departureDate=2026-09-10
tripType=ONE_WAY
```

---

### 27.2 왕복 검색

하나의 Search Endpoint에서 처리하거나
Frontend에서 출국 / 귀국 조건을 나누어 호출할 수 있습니다.

초안 Request Query:

```text
departure=ICN
arrival=NRT
departureDate=2026-09-10
returnDate=2026-09-15
tripType=ROUND_TRIP
```

구체적인 Search Contract는
Frontend 구현 방식과 함께 최종 결정합니다.

---

### 27.3 검색 Response

KOKU Flight 검색 결과에는
UI Design에서 필요한 최소 정보를 제공합니다.

예:

```json
{
  "flightId": 101,
  "flightNumber": "KO101",
  "departureAirport": "ICN",
  "arrivalAirport": "NRT",
  "departureAt": "2026-09-10T09:30:00+09:00",
  "arrivalAt": "2026-09-10T11:50:00+09:00",
  "status": "SCHEDULED",
  "bookable": true,
  "economyFare": 270000
}
```

`economyFare`는
해당 Flight의 `ECONOMY` SeatClass를 기준으로 계산한
검색 단계의 예상 운임입니다.

Passenger별 실제 최종 운임은
Reservation 시작 시 선택한 Seat의 SeatClass를 반영하여 확정합니다.

---

## 28. Flight 상세 API

초안:

```text
GET /api/v1/flights/{flightId}
```

Response에는 다음 정보를 포함할 수 있습니다.

- Flight Number
- Route
- Airport
- Date / Time
- Aircraft 기본 정보
- 예약 가능 여부
- ECONOMY 기준 예상 운임

외부 실제 Flight Data를
이 API Response와 혼합하지 않습니다.

---

## 29. Seat 조회 API

초안:

```text
GET /api/v1/flights/{flightId}/seats
```

Response 예:

```json
{
  "flightId": 101,
  "seats": [
    {
      "seatId": 1001,
      "seatNo": "1A",
      "seatClass": "BUSINESS",
      "status": "AVAILABLE"
    },
    {
      "seatId": 1002,
      "seatNo": "1B",
      "seatClass": "BUSINESS",
      "status": "RESERVED"
    },
    {
      "seatId": 1010,
      "seatNo": "5A",
      "seatClass": "PREMIUM_ECONOMY",
      "status": "AVAILABLE"
    }
  ]
}
```

Frontend에서는 `seatClass`를 이용하여
좌석 등급과 예상 추가 운임을 표현할 수 있습니다.

`HELD`, `RESERVED`, `UNAVAILABLE`은 모두
일반 Member에게 선택 불가 상태로 표현합니다.

일반 사용자 Seat 조회 Response에는
내부 Hold 소유자를 식별하는
`heldReservationId`를 노출하지 않습니다.

---

## 30. Reservation 시작 API

### 30.1 목적

Passenger와 모든 Flight별 Seat 선택이 완료된 후
실제 Reservation을 `PENDING` 상태로 시작합니다.

---

### 30.2 Endpoint 초안

```text
POST /api/v1/reservations
```

인증된 Member만 사용할 수 있습니다.

---

### 30.3 Request 초안

예시 구조:

```json
{
  "tripType": "ROUND_TRIP",
  "flights": [
    {
      "flightId": 101,
      "journeyRole": "OUTBOUND"
    },
    {
      "flightId": 202,
      "journeyRole": "RETURN"
    }
  ],
  "passengers": [
    {
      "lastName": "KIM",
      "firstName": "JIHUN",
      "birthDate": "1991-02-04",
      "gender": "MALE",
      "nationality": "KR",
      "flightSelections": [
        {
          "flightId": 101,
          "seatId": 1001
        },
        {
          "flightId": 202,
          "seatId": 2001
        }
      ]
    }
  ]
}
```

Infant가 있는 경우 Companion 식별 방식은
Request DTO 최종 설계 시 별도로 정의합니다.

Client가 Passenger Age Type을
신뢰 가능한 값으로 직접 전달하지 않습니다.

Backend가 생년월일과 각 Flight 탑승일을 기준으로 계산합니다.

---

### 30.4 처리

Reservation 생성 API는 최소 다음을 검증합니다.

1. 인증 Member
2. Flight 존재
3. Flight 상태
4. 신규 Reservation 2시간 제한
5. `ONE_WAY` / `ROUND_TRIP` 구조
6. 왕복 Reverse Route
7. 왕복 Date Rule
8. Passenger 정보
9. Adult / Child / Infant 판단
10. Infant Companion
11. Child Seat 인접성
12. Seat가 요청 Flight에 실제로 속하는지
13. Passenger별 Seat 필요 여부
14. 모든 Flight의 Seat 확보 가능 여부

Seat 확보 단계에서는
요청된 모든 Seat ID를 정렬한 뒤
하나의 Pessimistic Lock Query로 조회합니다.

```text
모든 Seat ID
        |
        v
중복 제거 / 검증
        |
        v
seat_id ASC
        |
        v
PESSIMISTIC_WRITE
        |
        v
전체 Seat 상태 검증
```

모든 Validation과 Seat 확보에 성공한 경우
하나의 Transaction Boundary 안에서 다음 작업을 처리합니다.

```text
PENDING Reservation 생성
+ Reservation 번호 생성
+ hold_expires_at 저장
        |
        v
Reservation / Flight / Passenger Mapping 생성
        |
        v
Passenger / Flight별 Seat 연결
        |
        v
SeatClass 반영 운임 계산
        |
        v
Reservation.total_amount 확정
        |
        v
선택 Seat
AVAILABLE → HELD
        |
        v
Seat.held_reservation_id
→ 생성한 Reservation ID
        |
        v
Commit
```

`ROUND_TRIP`에서도
출국 / 귀국 Seat 전체를 하나의 Transaction에서 처리합니다.

일부 Seat만 확보한 상태의 Commit은 허용하지 않습니다.

---

### 30.5 실패

Seat 중 하나라도 확보하지 못하면
Reservation 시작 전체를 실패합니다.

ROUND_TRIP에서도:

- 출국만 성공
- 귀국만 성공
- Passenger 일부만 성공

같은 Partial Success를 허용하지 않습니다.

Seat Pessimistic Lock 획득 후
하나 이상의 Seat가 더 이상 확보 가능한 상태가 아니면:

```text
HTTP 409 Conflict

code
→ SEAT_NOT_AVAILABLE
```

을 반환합니다.

동시 요청 경쟁으로 인해
사용자가 선택한 Seat를 다른 Reservation이 먼저 확보한 경우도
동일한 Business Error로 처리합니다.

Frontend는 해당 Error를 받으면
최신 Seat 상태를 다시 조회하도록 유도합니다.

Database Lock Timeout 또는 Deadlock과 같이
Business Seat 상태 충돌과 구분되는 Database 동시성 예외는
무한 재시도하지 않습니다.

구체적인 Retry 적용 여부는
동시성 테스트 결과를 기준으로 제한적으로 검토합니다.

---

## 31. Reservation 조회 API

### 31.1 내 Reservation 목록

```text
GET /api/v1/reservations
```

인증된 Member 자신의 Reservation만 조회합니다.

Query 예:

```text
status=CONFIRMED
page=0
size=20
```

Pagination의 구체적인 Response 형식은
구현 전에 확정합니다.

---

### 31.2 Reservation 상세

사용자 공개 Reservation 번호를 기준으로 조회하는 방안을 우선합니다.

```text
GET /api/v1/reservations/{reservationNo}
```

Backend는 현재 인증된 Member의 Reservation인지 검증합니다.

다른 Member의 Reservation이면 조회를 거부합니다.

---

### 31.3 상세 Response

예시:

```json
{
  "reservationNo": "KOKU-20260821-A7F3K9",
  "tripType": "ROUND_TRIP",
  "status": "CONFIRMED",
  "totalAmount": 540000,
  "flights": [],
  "passengers": [],
  "payments": [],
  "createdAt": "2026-08-21T20:30:00+09:00"
}
```

실제 Passport Number 전체를
기본 Reservation 상세 Response에 포함하지 않는 방향을 우선합니다.

---

## 32. Reservation 진행 취소 API

`PENDING` Reservation의 예약 진행을
사용자가 중단하는 경우 사용합니다.

초안:

```text
POST /api/v1/reservations/{reservationNo}/cancel-pending
```

처리:

```text
Reservation
PENDING → CANCELLED

Seat
HELD → AVAILABLE

현재 PENDING Payment
→ CANCELLED
```

기존 `FAILED` Payment는 변경하지 않습니다.

Endpoint Naming은 API 전체 설계 검토 후 최종 결정합니다.

---

## 33. Confirmed Reservation 취소 API

초안:

```text
POST /api/v1/reservations/{reservationNo}/cancel
```

Backend에서 다음을 검증합니다.

- 자신의 Reservation인지
- `CONFIRMED`인지
- Domain Policy의 24시간 제한
- ROUND_TRIP이면 부분 취소가 아닌 전체 취소인지
- 관련 Flight 출발 여부

성공 시:

```text
Reservation → CANCELLED
Payment SUCCESS → REFUNDED
아직 출발하지 않은 RESERVED Seat → AVAILABLE
```

ROUND_TRIP의 일부 Flight가 이미 출발한 Flight 취소 특수 케이스는
일반 Member 취소와 구분하여 Domain Policy를 적용합니다.

---

## 34. Mock Payment API

### 34.1 Endpoint 초안

```text
POST /api/v1/reservations/{reservationNo}/payments
```

---

### 34.2 요청 조건

다음 조건을 만족해야 합니다.

- 현재 Member의 Reservation
- Reservation `PENDING`
- Seat Hold 유효
- 기존 SUCCESS Payment 없음
- Payment 시도 횟수 3회 미만

---

### 34.3 Idempotency

Mock Payment 요청은
Idempotency Key를 사용하는 방향을 기본으로 합니다.

예:

```text
Idempotency-Key: <UUID>
```

동일 Key 요청을 다시 받으면
새 Payment를 생성하지 않습니다.

---

### 34.4 성공

성공 시 동일 Transaction 안에서:

```text
Payment
PENDING → SUCCESS

Reservation
PENDING → CONFIRMED

Seat
HELD → RESERVED
```

를 처리합니다.

---

### 34.5 실패

실패 시:

```text
Payment
PENDING → FAILED
```

처리합니다.

3번째 Payment까지 실패하면:

```text
Reservation
PENDING → CANCELLED

Seat
HELD → AVAILABLE
```

추가 Payment를 허용하지 않습니다.

---

## 35. Admin Flight API

### 35.1 Flight 조회

Admin과 SuperAdmin이 사용할 수 있습니다.

예:

```text
GET /api/v1/admin/flights
GET /api/v1/admin/flights/{flightId}
```

---

### 35.2 Flight 생성 / 수정

Admin과 SuperAdmin의 Flight 관리 권한은
Domain Policy를 기준으로 합니다.

초안:

```text
POST  /api/v1/admin/flights
PATCH /api/v1/admin/flights/{flightId}
```

Flight 생성 또는 핵심 운항정보 수정 시 Backend는 다음을 검증합니다.

- Flight Number + Departure Local Date 중복
- Route 활성 상태
- Aircraft 활성 상태
- Aircraft Schedule Conflict
- 60분 Turnaround Time
- `departure_at < arrival_at`

`departure_local_date`는 Request에서 직접 신뢰하지 않고
Backend가 Route의 출발 Airport `ZoneId`를 기준으로 계산합니다.

Admin이 수동 생성한 Flight는:

```text
flight_schedule_id = NULL
```

로 저장합니다.

기존 PENDING 또는 CONFIRMED Reservation이 연결된 Flight의
핵심 운항정보 변경 제한을 적용합니다.

단, Aircraft 변경은 더 엄격한 정책을 적용합니다.

Aircraft 변경은 다음 조건을 모두 만족해야 합니다.

- Flight가 `SCHEDULED`
- 아직 출발하지 않음
- 해당 Flight와 연결된 Reservation 이력이 한 번도 없음

현재 Reservation이 존재하지 않더라도
과거 `CANCELLED` Reservation 이력이 있으면
Aircraft 변경을 허용하지 않습니다.

Aircraft 변경이 허용된 경우:

```text
Aircraft Schedule Conflict 검증
        |
        v
Reservation 이력 없음 검증
        |
        v
기존 Seat Snapshot 제거
        |
        v
Aircraft 변경
        |
        v
새 AircraftSeat 기준 Seat Snapshot 생성
        |
        v
Commit
```

전체 작업은 하나의 Transaction으로 처리하며
Seat Snapshot 재생성에 실패하면
Aircraft 변경도 Rollback합니다.

---

### 35.3 Flight 취소

```text
POST /api/v1/admin/flights/{flightId}/cancel
```

Request:

```json
{
  "reason": "운항 일정 변경"
}
```

취소 사유는 필수입니다.

처리한 Admin / SuperAdmin 정보와
대상 Flight, 시각 및 사유를 Audit Log로 기록합니다.

---

### 35.4 Flight 물리 삭제

일반적인 운영 변경에서는
Flight를 물리 삭제하지 않고 `CANCELLED` 처리를 우선합니다.

잘못 생성된 Flight에 한하여
다음 API를 제한적으로 제공합니다.

```text
DELETE /api/v1/admin/flights/{flightId}
```

다음 조건을 모두 만족해야 합니다.

- Flight 상태가 `SCHEDULED`
- 아직 출발하지 않음
- `flight_schedule_id = NULL`
- 해당 Flight와 연결된 Reservation이 한 번도 존재하지 않음
- 운영 이력 보존이 필요한 Flight가 아님

즉, MVP에서 물리 삭제는
Admin 또는 SuperAdmin이 직접 생성한 잘못된 Flight에만 허용합니다.

FlightSchedule을 기준으로 자동 생성된 Flight는
물리 삭제하지 않고 `CANCELLED` 상태로 관리합니다.

이는 자동 생성 Flight를 물리 삭제한 뒤
Daily Scheduler가 동일 운항일의 Flight를 다시 생성하는 상황을 방지하기 위함입니다.

다음과 같은 Reservation 관계가 하나라도 존재하면
현재 Reservation 상태와 관계없이 삭제할 수 없습니다.

```text
ReservationFlight
→ 해당 flight_id 존재
```

즉 과거에 `PENDING`, `CONFIRMED`, `CANCELLED` 상태의
Reservation과 연결된 이력이 있는 Flight도 물리 삭제하지 않습니다.

조건을 만족하지 않으면
삭제 요청을 거부하고 필요한 경우 Flight 취소 기능을 사용합니다.

---

### 35.5 FlightSchedule 관리 API

Admin과 SuperAdmin은
정규 Flight 자동 생성에 사용하는 FlightSchedule을 관리할 수 있습니다.

초안:

```text
GET   /api/v1/admin/flight-schedules
GET   /api/v1/admin/flight-schedules/{scheduleId}
POST  /api/v1/admin/flight-schedules
PATCH /api/v1/admin/flight-schedules/{scheduleId}
```

FlightSchedule 생성 / 수정 Request에는
최소 다음 정보를 사용할 수 있습니다.

```text
flightNumber
routeId
defaultAircraftId
operatingDays
departureLocalTime
arrivalLocalTime
arrivalDayOffset
active
```

FlightSchedule 변경은
이미 생성된 Flight를 일괄 UPDATE하지 않습니다.

```text
기존 Flight
→ 변경 없음

향후 생성 Flight
→ 변경된 Schedule 적용
```

FlightSchedule을 비활성화하면
향후 자동 Flight 생성을 중단합니다.

이미 생성되어 있는 Flight는
자동 취소하거나 삭제하지 않습니다.

기존 Flight의 운항을 중단해야 하는 경우
각 Flight에 대해 별도의 Flight 취소 정책을 적용합니다.

---

### 35.6 Flight Seat 운영 상태 관리 API

Admin과 SuperAdmin은
특정 Flight의 `AVAILABLE` Seat를 운영상 판매 불가 상태로 변경하거나
운영 제한이 해제된 Seat를 다시 판매 가능 상태로 복구할 수 있습니다.

초안:

```text
PATCH /api/v1/admin/flights/{flightId}/seats/{seatId}/availability
```

Request 예:

```json
{
  "available": false
}
```

처리 규칙:

```text
available = false

AVAILABLE → UNAVAILABLE
```

```text
available = true

UNAVAILABLE → AVAILABLE
```

다음 상태의 Seat는
이 API를 통해 직접 `UNAVAILABLE`로 변경하지 않습니다.

- `HELD`
- `RESERVED`

Backend는 다음을 검증합니다.

- 대상 Flight 존재
- 대상 Seat 존재
- Seat가 해당 Flight에 속하는지
- 현재 Seat 상태가 허용된 전이인지

허용되지 않는 상태 전이는
현재 Resource 상태와의 충돌로 처리합니다.

---

## 36. SuperAdmin Master Data API

SuperAdmin만 다음 변경 API를 사용할 수 있습니다.

#### Airport

```text
POST  /api/v1/admin/airports
PATCH /api/v1/admin/airports/{airportId}
```

#### Route

```text
POST  /api/v1/admin/routes
PATCH /api/v1/admin/routes/{routeId}
```

#### Aircraft

```text
POST  /api/v1/admin/aircraft
PATCH /api/v1/admin/aircraft/{aircraftId}
```

Master Data는
물리 DELETE보다 비활성화를 사용합니다.

구체적인 Endpoint에서 `/admin`과 `/superadmin`을 구분할지는
Spring Security 권한 구조 및 API Naming 검토 후 결정합니다.

---

## 37. Admin Reservation 조회 API

Admin과 SuperAdmin은
운영상 Reservation 상태를 조회할 수 있습니다.

초안:

```text
GET /api/v1/admin/reservations
GET /api/v1/admin/reservations/{reservationNo}
```

조회 API에서는
Passenger Test Passport Number 등
불필요한 민감정보 전체를 기본적으로 노출하지 않습니다.

---

## 38. SuperAdmin Reservation 강제 취소 API

초안:

```text
POST /api/v1/admin/reservations/{reservationNo}/force-cancel
```

SuperAdmin만 사용할 수 있습니다.

Request:

```json
{
  "reason": "운영자 강제 취소 사유"
}
```

Backend에서 Domain Policy의
강제 취소 가능 조건을 검증합니다.

사유는 필수입니다.

Audit Log를 기록합니다.

---

## 39. 외부 실제 항공편 조회 API

### 39.1 데이터 경계

외부 실제 항공편 조회 결과는
KOKU 내부 `Flight`, `Seat`, `Reservation`으로 저장하지 않습니다.

---

### 39.2 Endpoint 초안

```text
GET /api/v1/external-flights
```

Query 예:

```text
departure=ICN
arrival=NRT
departureDate=2026-09-10
```

Backend에서 한국 ↔ 일본 지원 범위를 먼저 검증한 후
외부 API를 호출합니다.

---

### 39.3 External DTO

외부 API Provider Response를
Frontend에 그대로 전달하지 않습니다.

Provider별 Adapter에서
서비스 내부 External Flight DTO로 변환합니다.

초안:

```json
{
  "airline": "Example Airline",
  "flightNumber": "EX101",
  "departureAirport": "ICN",
  "arrivalAirport": "NRT",
  "departureAt": "2026-09-10T09:30:00+09:00",
  "arrivalAt": "2026-09-10T11:50:00+09:00",
  "price": 250000,
  "currency": "KRW",
  "stops": 0
}
```

실제 Field는
선택한 External Flight API의 제공 Data에 맞춰 확정합니다.

Provider가 제공하지 않는 값을
임의로 생성하지 않습니다.

---

## 40. AI 항공편 검색 API

### 40.1 권한

AI 항공편 검색 실행은
인증된 Member만 사용할 수 있습니다.

---

### 40.2 Endpoint 초안

```text
POST /api/v1/ai/flight-search
```

Request:

```json
{
  "query": "9월 10일 인천에서 도쿄로 오전에 출발하는 직항편 중 저렴한 항공편 찾아줘"
}
```

---

### 40.3 처리 단계

```text
자연어 Query
    |
    v
AI Structured Output
    |
    v
Application Validation
    |
    v
한국 ↔ 일본 범위 검증
    |
    v
External Flight API
    |
    v
Application Filter
    |
    v
Application Rank
    |
    v
AI 추천 설명
```

---

### 40.4 Structured Search Criteria 초안

```json
{
  "departureAirport": "ICN",
  "arrivalAirport": "NRT",
  "departureDate": "2026-09-10",
  "departureTimePreference": "MORNING",
  "directOnly": true,
  "maxPrice": null
}
```

AI가 생성한 Criteria는
외부 API 호출 전 Backend에서 다시 Validation합니다.

---

### 40.5 AI Response 초안

```json
{
  "criteria": {
    "departureAirport": "ICN",
    "arrivalAirport": "NRT",
    "departureDate": "2026-09-10"
  },
  "recommendations": [
    {
      "flight": {
      },
      "reason": "오전 출발 조건을 만족하는 직항편입니다."
    }
  ]
}
```

실제 Flight의 사실 정보는
External Flight API 데이터를 기준으로 합니다.

AI는 Flight Number, 가격, Airline, 출도착 시각 등을
임의로 생성하지 않습니다.

---

## 41. Pagination

목록 API에는 필요한 경우 Pagination을 적용합니다.

초기 대상:

- Reservation 목록
- Admin Reservation 목록
- Admin Flight 목록

초안 Query:

```text
page=0
size=20
```

Page 기반 Pagination을 우선 검토합니다.

실제 데이터 규모와 Query 특성상
Cursor Pagination이 필요하다고 판단되면 이후 변경할 수 있습니다.

---

## 42. API Authorization Matrix 초안

| API 영역 | Guest | Member | Admin | SuperAdmin |
| --- | --- | --- | --- | --- |
| KOKU Flight 검색 | O | O | O | O |
| Flight 상세 | O | O | O | O |
| 외부 실제 Flight 조회 | O | O | O | O |
| AI Flight 검색 실행 | X | O | 정책상 필요 시 가능 | 정책상 필요 시 가능 |
| Reservation 생성 | X | O | 자신의 Member 기능 사용 시 | 자신의 Member 기능 사용 시 |
| 내 Reservation 조회 | X | O | 자신의 예약에 한함 | 자신의 예약에 한함 |
| Admin Reservation 조회 | X | X | O | O |
| Flight 관리 | X | X | O | O |
| Master Data 변경 | X | X | X | O |
| Reservation 강제 취소 | X | X | X | O |

Admin / SuperAdmin이 일반 Member 기능을 함께 사용할지
계정 Role 설계에서 별도 Member 계정으로 분리할지는
구현 전에 최종 확인합니다.

따라서 해당 부분은 현재 Domain Policy를 확장하는 방식으로
임의 확정하지 않습니다.

---

## 43. Transaction과 API Boundary

다음 API는
Database 정합성을 위해 명확한 Transaction Boundary를 가져야 합니다.

#### Reservation 시작

```text
POST /reservations
```

#### Mock Payment

```text
POST /reservations/{reservationNo}/payments
```

#### Reservation 취소

```text
POST /reservations/{reservationNo}/cancel
```

#### Flight 취소

```text
POST /admin/flights/{flightId}/cancel
```

#### SuperAdmin 강제 취소

```text
POST /admin/reservations/{reservationNo}/force-cancel
```

Controller가 직접 Transaction을 관리하지 않고
Application / Service 계층에서 처리합니다.

---

## 44. External API와 Transaction Boundary

External Flight API 또는 AI API 호출을
KOKU 내부 Reservation Transaction과 결합하지 않습니다.

예:

```text
External Flight API 실패

≠

Reservation Transaction 실패
```

외부 API 장애가
내부 Reservation Database 상태에 영향을 주지 않도록 합니다.

---

## 45. 데이터 노출 정책

### 45.1 API에 노출하지 않는 값

기본적으로 다음 값은 Client API Response Body에 전달하지 않습니다.

- Password Hash
- Refresh Token Hash 등 Server-side 저장 정보
- OAuth Provider Secret
- External API Key
- AI API Key
- 내부 Security 정보

Refresh Token 원문은
`04-system-design.md`의 인증 정책에 따라
HttpOnly / Secure Cookie로만 전달하며
API Response Body에는 포함하지 않습니다.

---

### 45.2 제한적으로 노출하는 값

테스트용 Passport Number 등은
업무상 필요한 Endpoint에만 제한적으로 포함합니다.

Reservation 목록과 같이
필요하지 않은 API에는 포함하지 않습니다.

---

## 46. Data Migration 및 Seed

### 46.1 초기 Master Data

MVP 개발환경에서는
지원 Airport 등 필요한 Master Data를
Seed Data로 구성할 수 있습니다.

예:

```text
ICN
GMP
PUS
CJU

NRT
HND
KIX
FUK
CTS
NGO
```

---

### 46.2 테스트 Flight

Local / Test 환경에서는
KOKU Airline Flight와 Aircraft를
테스트용 Seed Data로 구성할 수 있습니다.

운영 환경 Seed와
테스트 전용 Data를 구분합니다.

---

### 46.3 Migration Tool

Database Schema Migration Tool 도입 여부는
Backend 초기 구성 단계에서 결정합니다.

검토 대상:

- Flyway
- Liquibase

Migration Tool을 사용하는 경우
Schema 변경 이력을 Repository에서 관리합니다.

---

## 47. 아직 확정하지 않는 Data 설계

다음 사항은
`04-system-design.md` 또는 실제 구현 검토 후 확정합니다.

### 47.1 Seat

- [ ] Aircraft Seat 통로 표현 방식

---

### 47.2 Passenger

- [ ] Gender Enum 상세 값
- [ ] Nationality 저장 규칙
- [ ] Test Passport Number 형식
- [ ] Passport Number Encryption 적용 여부
- [ ] Passport API Masking 규칙

---

### 47.3 Reservation

- [ ] Reservation Number Collision 재시도 횟수
- [ ] `ReservationFlight` 실제 Entity 여부
- [ ] `ReservationPassenger` 실제 Entity 여부
- [ ] `PassengerFlight` 실제 Entity 여부
- [ ] Cancellation Reason 상세 코드 구조

---

### 47.4 Payment

- [ ] Idempotency Header 최종 이름
- [ ] 동일 Key + 다른 Body 요청 처리
- [ ] SUCCESS Payment Database 추가 보호 방식

---

### 47.5 API

- [ ] `/api/v1` Version Prefix 최종 적용 여부
- [ ] 공통 Success Response Wrapper 여부
- [ ] Pagination Response 형식
- [ ] Validation Error `details` 구조
- [ ] HTTP Status Code 상세 Mapping

---

### 47.6 External / AI

- [ ] 실제 External Flight API Provider
- [ ] External Flight DTO 최종 Field
- [ ] AI Structured Output DTO
- [ ] AI Recommendation 최대 후보 수
- [ ] Tool Contract

---

## 48. 권장 HTTP Status 기본 방향

구체적인 Error Code Mapping은
API 구현 전에 최종 확정합니다.

기본 방향:

```text
200 OK
→ 정상 조회 / 처리

201 Created
→ Resource 생성

204 No Content
→ Response Body가 필요 없는 정상 처리

400 Bad Request
→ 입력 Validation 실패

401 Unauthorized
→ 인증 필요 / 인증 실패

403 Forbidden
→ 권한 부족

404 Not Found
→ Resource 없음

409 Conflict
→ Seat 경쟁 등 현재 Resource 상태와 충돌

500 Internal Server Error
→ 예상하지 못한 서버 오류

502 / 503 계열
→ External API 장애를 표현해야 하는 경우 검토
```

Business Error와 HTTP Status는
일관되게 Mapping합니다.

---

## 49. Error Code 초안

Error Code는
Locale과 독립적인 영문 식별자를 사용합니다.

예:

```text
AUTHENTICATION_REQUIRED
ACCESS_DENIED

MEMBER_NOT_FOUND
MEMBER_WITHDRAWN
EMAIL_ALREADY_EXISTS

FLIGHT_NOT_FOUND
FLIGHT_NOT_BOOKABLE
FLIGHT_ALREADY_DEPARTED

SEAT_NOT_AVAILABLE
SEAT_HOLD_EXPIRED

INVALID_PASSENGER
CHILD_ADULT_REQUIRED
CHILD_ADJACENT_SEAT_REQUIRED
INFANT_COMPANION_REQUIRED

RESERVATION_NOT_FOUND
RESERVATION_NOT_CANCELLABLE

PAYMENT_ATTEMPT_EXCEEDED
PAYMENT_ALREADY_SUCCEEDED

EXTERNAL_FLIGHT_API_ERROR
AI_RESPONSE_ERROR
```

최종 Error Code 목록은
API 구현 과정에서 Domain Exception과 함께 확정합니다.

---

## 50. Data & API Design 완료 기준

다음 조건을 모두 만족하면
MVP Data & API Design이 완료된 것으로 판단합니다.

### Data Model

- [ ] 핵심 Entity가 정의되어 있습니다.
- [ ] Member와 AuthAccount 관계가 정의되어 있습니다.
- [ ] Airport와 Route 관계가 정의되어 있습니다.
- [ ] Aircraft와 Seat Configuration 관계가 정의되어 있습니다.
- [ ] Flight와 Seat 관계가 정의되어 있습니다.
- [ ] Reservation과 Flight 관계가 정의되어 있습니다.
- [ ] Reservation과 Passenger 관계가 정의되어 있습니다.
- [ ] Passenger의 Flight별 Seat 구조가 정의되어 있습니다.
- [ ] Infant Companion 표현 방식이 정의되어 있습니다.
- [ ] Reservation과 Payment 관계가 정의되어 있습니다.
- [ ] Audit Log 구조가 정의되어 있습니다.
- [ ] FlightSchedule과 Flight 관계가 정의되어 있습니다.
- [ ] FlightSchedule의 운항 요일 구조가 정의되어 있습니다.

### Constraint

- [ ] Member Email Unique가 정의되어 있습니다.
- [ ] Airport IATA Unique가 정의되어 있습니다.
- [ ] Route 중복 방지가 정의되어 있습니다.
- [ ] Flight Seat 중복 방지가 정의되어 있습니다.
- [ ] Reservation 번호 Unique가 정의되어 있습니다.
- [ ] Payment Idempotency Unique가 정의되어 있습니다.
- [ ] 필요한 Foreign Key가 정의되어 있습니다.
- [ ] Flight Number + Departure Local Date Unique가 정의되어 있습니다.
- [ ] FlightSchedule + Departure Local Date 중복 생성 방지가 정의되어 있습니다.
- [ ] FlightSchedule 운항 요일 중복 방지가 정의되어 있습니다.
- [ ] Aircraft Schedule Conflict Validation 구조가 정의되어 있습니다.

### Reservation

- [ ] ONE_WAY / ROUND_TRIP Mapping이 정의되어 있습니다.
- [ ] Journey Role 표현 방식이 정의되어 있습니다.
- [ ] PENDING 생성 시 운임 저장 구조가 정의되어 있습니다.
- [ ] Hold 만료시각 저장 구조가 정의되어 있습니다.
- [ ] Reservation 번호 생성 구조가 정의되어 있습니다.

### Passenger

- [ ] Passenger 기본정보 Column이 정의되어 있습니다.
- [ ] 연령을 Flight별로 계산하는 구조가 정의되어 있습니다.
- [ ] Test Passport 저장 구조가 정의되어 있습니다.
- [ ] Test Passport 보호 정책이 확정되어 있습니다.
- [ ] Child Seat Validation에 필요한 데이터가 정의되어 있습니다.
- [ ] Infant Companion 관계가 정의되어 있습니다.

### Payment

- [ ] Payment가 개별 결제 시도를 나타내도록 정의되어 있습니다.
- [ ] Payment 최대 3회 구조가 정의되어 있습니다.
- [ ] Payment Idempotency 구조가 정의되어 있습니다.
- [ ] SUCCESS / FAILED / CANCELLED / REFUNDED 상태 처리가 정의되어 있습니다.

### API

- [ ] 인증 API Contract가 정의되어 있습니다.
- [ ] Member API Contract가 정의되어 있습니다.
- [ ] Flight 검색 / 상세 API가 정의되어 있습니다.
- [ ] Seat 조회 API가 정의되어 있습니다.
- [ ] Reservation 생성 API가 정의되어 있습니다.
- [ ] Reservation 조회 / 취소 API가 정의되어 있습니다.
- [ ] Mock Payment API가 정의되어 있습니다.
- [ ] Admin / SuperAdmin API가 정의되어 있습니다.
- [ ] External Flight API Contract가 정의되어 있습니다.
- [ ] AI Flight Search API Contract가 정의되어 있습니다.
- [ ] Error Response Contract가 정의되어 있습니다.
- [ ] Admin FlightSchedule 관리 API가 정의되어 있습니다.
- [ ] 제한적인 Flight 물리 삭제 API 조건이 정의되어 있습니다.

### Security / Boundary

- [ ] 내부 Primary Key와 공개 식별자가 구분되어 있습니다.
- [ ] Password Hash가 API에 노출되지 않습니다.
- [ ] Passport 정보 노출 정책이 정의되어 있습니다.
- [ ] 외부 Flight Data와 내부 Flight Entity가 분리되어 있습니다.
- [ ] AI가 Transaction API를 직접 실행하지 않습니다.

---

## 51. Data & API Design 변경 원칙

본 문서는 Backend와 Frontend 사이의
Database 및 API Contract 기준으로 사용합니다.

구현 과정에서 Data Model 또는 API Contract 변경이 필요한 경우
코드를 먼저 변경하지 않습니다.

기본 절차:

```text
설계 문제 발견
        |
        v
영향 범위 확인
        |
        v
Domain Policy 영향 검토
        |
        v
System Design 영향 검토
        |
        v
Data & API Design 수정
        |
        v
Frontend / Backend Contract 수정
        |
        v
구현
```

다음 변경은 반드시 관련 문서 정합성을 검토합니다.

- Core Entity 추가 / 삭제
- Entity 관계 변경
- Reservation / Flight Mapping 변경
- Passenger / Flight Mapping 변경
- Seat 상태 저장 구조 변경
- Payment 상태 또는 시도 구조 변경
- Reservation 번호 정책 변경
- API Endpoint 변경
- Request / Response Field 변경
- Error Contract 변경
- Idempotency 방식 변경
- External Flight Data 경계 변경
- AI Tool Contract 변경

비즈니스 규칙 자체를 변경해야 하는 경우
`02-domain-policy.md`를 먼저 수정합니다.

사용자 흐름에 영향을 주는 경우
`03-ui-design.md`를 함께 검토합니다.

Architecture 또는 Transaction 구조에 영향을 주는 경우
`04-system-design.md`를 함께 수정합니다.