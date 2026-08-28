# KOKU Airline Renewal - ERD

## 1. 문서 목적

본 문서는 `05-data-api-design.md`를 기반으로
KOKU Airline Renewal의 MVP Database 구조를 시각화합니다.

본 ERD는 다음 문서를 기준으로 합니다.

- 비즈니스 규칙: `02-domain-policy.md`
- 시스템 구조: `04-system-design.md`
- 데이터 및 API 구조: `05-data-api-design.md`

본 문서는 Data Model을 새롭게 정의하는 문서가 아닙니다.

Entity, Column, Relation 및 Constraint의 최종 기준은
`05-data-api-design.md`를 따릅니다.

현재 확정되지 않은 Data Model은
ERD에서도 Draft 상태로 표현합니다.

---

## 2. MVP ERD

```mermaid
erDiagram

    MEMBER {
    BIGINT id PK
    VARCHAR email UK
    VARCHAR role
    VARCHAR status
    DATETIME(6) created_at
    DATETIME(6) updated_at
    }

    AUTH_ACCOUNT {
    BIGINT id PK
    BIGINT member_id FK
    VARCHAR provider
    VARCHAR provider_subject
    VARCHAR password_hash
    DATETIME(6) created_at
    DATETIME(6) updated_at
    }

    AIRPORT {
    BIGINT id PK
    VARCHAR(3) iata_code UK
    VARCHAR country_code
    VARCHAR(50) timezone
    BOOLEAN active
    DATETIME(6) deactivated_at
    DATETIME(6) created_at
    DATETIME(6) updated_at
    }

    ROUTE {
    BIGINT id PK
    BIGINT departure_airport_id FK
    BIGINT arrival_airport_id FK
    BOOLEAN active
    DATETIME(6) deactivated_at
    DATETIME(6) created_at
    DATETIME(6) updated_at
    }

    AIRCRAFT {
    BIGINT id PK
    VARCHAR aircraft_code
    VARCHAR model_name
    BOOLEAN active
    DATETIME(6) deactivated_at
    DATETIME(6) created_at
    DATETIME(6) updated_at
    }

    AIRCRAFT_SEAT {
    BIGINT id PK
    BIGINT aircraft_id FK
    VARCHAR seat_no
    INT row_no
    VARCHAR seat_column
    VARCHAR seat_class
    BOOLEAN active
    DATETIME(6) deactivated_at
    DATETIME(6) created_at
    DATETIME(6) updated_at
    }

    FLIGHT_SCHEDULE {
    BIGINT id PK
    VARCHAR flight_number
    BIGINT route_id FK
    BIGINT default_aircraft_id FK
    TIME departure_local_time
    TIME arrival_local_time
    INT arrival_day_offset
    BOOLEAN active
    DATETIME(6) deactivated_at
    DATETIME(6) created_at
    DATETIME(6) updated_at
    }

    FLIGHT_SCHEDULE_DAY {
    BIGINT flight_schedule_id FK
    VARCHAR day_of_week
    }

        FLIGHT {
    BIGINT id PK
    BIGINT flight_schedule_id FK
    VARCHAR flight_number
    BIGINT route_id FK
    BIGINT aircraft_id FK
    DATE departure_local_date
    DATETIME(6) departure_at
    DATETIME(6) arrival_at
    VARCHAR status
    VARCHAR cancellation_reason
    DATETIME(6) created_at
    DATETIME(6) updated_at
    }

    SEAT {
    BIGINT id PK
    BIGINT flight_id FK
    VARCHAR seat_no
    INT row_no
    VARCHAR seat_column
    VARCHAR seat_class
    VARCHAR status
    BIGINT held_reservation_id FK
    DATETIME(6) created_at
    DATETIME(6) updated_at
    }

    MEMBER ||--o{ AUTH_ACCOUNT : has
    MEMBER ||--o{ RESERVATION : creates

    AIRPORT ||--o{ ROUTE : departure
    AIRPORT ||--o{ ROUTE : arrival

        ROUTE ||--o{ FLIGHT_SCHEDULE : defines
    ROUTE ||--o{ FLIGHT : contains

    AIRCRAFT ||--o{ AIRCRAFT_SEAT : configures
    AIRCRAFT ||--o{ FLIGHT_SCHEDULE : default_for
    AIRCRAFT ||--o{ FLIGHT : assigned_to

    FLIGHT_SCHEDULE ||--|{ FLIGHT_SCHEDULE_DAY : operates_on
    FLIGHT_SCHEDULE o|--o{ FLIGHT : generates

    FLIGHT ||--o{ SEAT : has

    RESERVATION o|--o{ SEAT : temporarily_holds

    RESERVATION {
    BIGINT id PK
    VARCHAR reservation_no UK
    BIGINT member_id FK
    VARCHAR trip_type
    VARCHAR status
    DECIMAL(15,0) total_amount
    DATETIME(6) hold_expires_at
    VARCHAR cancel_reason
    DATETIME(6) created_at
    DATETIME(6) updated_at
    }

    RESERVATION_FLIGHT {
    BIGINT id PK
    BIGINT reservation_id FK
    BIGINT flight_id FK
    VARCHAR journey_role
    INT sequence
    DATETIME(6) created_at
    }

    PASSENGER {
    BIGINT id PK
    VARCHAR(50) last_name
    VARCHAR(50) first_name
    DATE birth_date
    VARCHAR(10) gender
    CHAR(2) nationality
    VARBINARY(128) test_passport_no_ciphertext
    BINARY(12) test_passport_no_iv
    CHAR(2) test_passport_country
    DATE test_passport_expiry_date
    DATETIME(6) created_at
    DATETIME(6) updated_at
    }

    RESERVATION_PASSENGER {
    BIGINT id PK
    BIGINT reservation_id FK
    BIGINT passenger_id FK
    INT sequence
    DATETIME(6) created_at
    }

    PASSENGER_FLIGHT {
    BIGINT id PK
    BIGINT reservation_id FK
    BIGINT passenger_id FK
    BIGINT flight_id FK
    BIGINT seat_id FK
    BIGINT companion_passenger_id FK
    VARCHAR(10) age_category
    DECIMAL(15,0) fare_amount
    DATETIME(6) created_at
    }

    PAYMENT {
    BIGINT id PK
    BIGINT reservation_id FK
    INT attempt_no
    VARCHAR status
    DECIMAL(15,0) amount
    VARCHAR idempotency_key
    DATETIME(6) created_at
    DATETIME(6) updated_at
    }

    AUDIT_LOG {
    BIGINT id PK
    BIGINT actor_member_id FK
    VARCHAR action_type
    VARCHAR target_type
    BIGINT target_id
    VARCHAR reason
    DATETIME(6) created_at
    }

    RESERVATION ||--|{ RESERVATION_FLIGHT : contains
    FLIGHT ||--o{ RESERVATION_FLIGHT : included_in

    RESERVATION ||--|{ RESERVATION_PASSENGER : contains
    PASSENGER ||--o{ RESERVATION_PASSENGER : included_in

    RESERVATION ||--|{ PASSENGER_FLIGHT : has
    PASSENGER ||--o{ PASSENGER_FLIGHT : boards
    FLIGHT ||--o{ PASSENGER_FLIGHT : boards

    SEAT o|--o{ PASSENGER_FLIGHT : assigned_to

    PASSENGER o|--o{ PASSENGER_FLIGHT : companion_for

    RESERVATION ||--o{ PAYMENT : has

    MEMBER ||--o{ AUDIT_LOG : performs
```

---

### 2.1 Date / Time 표기 원칙

ERD의 Date / Time Type은
`05-data-api-design.md`의 확정 정책을 따릅니다.

```text
절대 시각
→ Java Instant
→ MySQL DATETIME(6)
→ UTC

날짜
→ Java LocalDate
→ MySQL DATE

Airport Time Zone
→ IANA Zone ID
→ VARCHAR(50)
```

`DATETIME(6)` 자체에는 Time Zone 정보가 포함되지 않으며,
Application / JDBC / Hibernate의 시간 해석 기준을 UTC로 통일합니다.

---

## 3. 핵심 관계

### 3.1 Member - AuthAccount

```text
Member 1 : N AuthAccount
```

하나의 Member는 하나 이상의 인증 수단을 가질 수 있습니다.

MVP Provider:

```text
LOCAL
GOOGLE
```

`Member`는 서비스 사용자와 Role을 관리하고,
`AuthAccount`는 인증 수단을 관리합니다.

서비스 Role:

```text
MEMBER
ADMIN
SUPERADMIN
```

Spring Security 권한 Code:

```text
MEMBER     → USER
ADMIN      → ADMIN
SUPERADMIN → SUPERADMIN
```

---

### 3.2 Member - Reservation

```text
Member 1 : N Reservation
```

하나의 Reservation은 반드시 하나의 Member에 소속됩니다.

Member는 여러 Reservation을 생성할 수 있습니다.

Reservation의 사용자 공개 식별자는
Database PK가 아니라 `reservation_no`를 사용합니다.

---

### 3.3 Airport - Route

Route는 두 Airport를 참조합니다.

```text
departure_airport_id
arrival_airport_id
```

동일 Airport를 출발지와 도착지로 사용할 수 없습니다.

동일 방향 Route 중복을 허용하지 않습니다.

```text
UNIQUE(
  departure_airport_id,
  arrival_airport_id
)
```

---

### 3.4 Route - FlightSchedule

```text
Route 1 : N FlightSchedule
```

하나의 FlightSchedule은
하나의 Route를 기준으로 반복 운항합니다.

FlightSchedule은 실제 예약 대상 Flight가 아니라
향후 Flight를 자동 생성하기 위한 운항 Template입니다.

비활성화된 Route는
새로운 FlightSchedule에 사용할 수 없습니다.

---

### 3.5 Route - Flight

```text
Route 1 : N Flight
```

하나의 Flight는 하나의 Route에 속합니다.

Flight는 특정 날짜와 시간에 운항하는
KOKU Airline 내부 항공편입니다.

외부 실제 항공편은 `Flight` Entity에 저장하지 않습니다.

---

### 3.6 Aircraft - AircraftSeat

```text
Aircraft 1 : N AircraftSeat
```

`AircraftSeat`는 Aircraft의 기본 Seat Layout을 나타냅니다.

`AircraftSeat`는 Seat Number와 Layout 정보뿐 아니라
해당 좌석의 SeatClass를 정의합니다.

MVP SeatClass:

```text
ECONOMY
PREMIUM_ECONOMY
BUSINESS
```

`FIRST`는 MVP에서 사용하지 않습니다.

Flight 생성 시 활성 `AircraftSeat`의 다음 정보를
Flight별 `Seat`로 Snapshot합니다.

```text
seat_no
row_no
seat_column
seat_class
```

동일 Aircraft 안에서
Seat Number는 중복될 수 없습니다.

```text
UNIQUE(
  aircraft_id,
  seat_no
)
```

---

### 3.7 Aircraft - FlightSchedule

```text
Aircraft 1 : N FlightSchedule
```

FlightSchedule은 자동 생성 Flight에 사용할
기본 Aircraft를 참조합니다.

`default_aircraft_id`는
Flight 생성 시 초기 Aircraft 배정값으로 사용되며,
이미 생성된 Flight의 Aircraft를 자동 변경하지 않습니다.

---

### 3.8 FlightSchedule - FlightScheduleDay / Flight

FlightSchedule은 반복 운항 규칙을 나타냅니다.

```text
Route
   |
   v
FlightSchedule
   |
   +----< FlightScheduleDay
   |
   +----< Flight
```

하나의 FlightSchedule은
하나 이상의 운항 요일을 가집니다.

```text
FlightSchedule 1 : N FlightScheduleDay
```

운항 요일 값:

```text
MONDAY
TUESDAY
WEDNESDAY
THURSDAY
FRIDAY
SATURDAY
SUNDAY
```

동일 FlightSchedule에
같은 요일을 중복 등록할 수 없습니다.

```text
UNIQUE(
    flight_schedule_id,
    day_of_week
)
```

FlightSchedule에서 자동 생성된 Flight는
해당 Schedule을 참조합니다.

```text
FlightSchedule 1 : N Flight
```

단, Admin 또는 SuperAdmin이 직접 생성한 Flight는
FlightSchedule을 참조하지 않을 수 있습니다.

```text
자동 생성 Flight
→ flight_schedule_id = FlightSchedule ID

수동 생성 Flight
→ flight_schedule_id = NULL
```

FlightSchedule의 변경은
이미 생성된 Flight의 값을 자동 변경하지 않습니다.

---

### 3.9 Aircraft - Flight

```text
Aircraft 1 : N Flight
```

하나의 Flight에는 하나의 Aircraft가 배정됩니다.

자동 생성 Flight는
FlightSchedule의 `default_aircraft_id`를 기본값으로 사용합니다.

Admin 또는 SuperAdmin의 Aircraft 변경은
다음 조건을 모두 만족하는 경우에만 허용합니다.

```text
Flight.status = SCHEDULED

아직 출발하지 않음

Reservation 연결 이력 = 0건
```

현재 활성 Reservation이 없더라도
과거에 Reservation이 한 번이라도 연결된 Flight는
Aircraft를 변경할 수 없습니다.

다음 Reservation 이력도 Aircraft 변경을 차단합니다.

```text
PENDING
CONFIRMED
CANCELLED
```

Aircraft 변경이 허용되는 경우:

```text
기존 Flight Seat Snapshot 제거
→ Flight.aircraft 변경
→ 새 AircraftSeat 기준 Seat Snapshot 재생성
```

위 작업은 하나의 Database Transaction으로 처리하며,
Seat Snapshot 재생성에 실패하면
Aircraft 변경 전체를 Rollback합니다.

동일 Aircraft의 Flight는
MVP 기준 고정 `60분`의 Turnaround Time을 포함하여
운항 일정이 충돌하지 않아야 합니다.

```text
previous Flight arrival_at
+
60 minutes
<=
next Flight departure_at
```

`CANCELLED` Flight는
Aircraft Schedule Conflict 계산에서 제외합니다.

이 충돌 규칙은 단순 Database Unique Constraint가 아니라
Application Validation으로 보호합니다.

---

### 3.10 Flight - Seat

```text
Flight 1 : N Seat
```

`AircraftSeat`는 Aircraft의 기본 Seat Configuration이고,
`Seat`는 특정 Flight에서 실제 예약 상태를 가지는
Flight별 Snapshot 좌석입니다.

Flight 생성 시
배정된 Aircraft의 활성 `AircraftSeat`를 기준으로
다음 값을 복사하여 Seat를 생성합니다.

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

따라서 이후 AircraftSeat Master Data가 변경되더라도
이미 생성된 Flight의 Seat Snapshot을
자동으로 변경하지 않습니다.

Flight와 해당 Flight의 Seat Snapshot 생성은
하나의 Database Transaction으로 처리합니다.

```text
Flight 생성
→ AircraftSeat 조회
→ Flight별 Seat Snapshot 생성
→ Commit
```

Seat Snapshot 생성에 실패하면
Flight 생성도 함께 Rollback합니다.

이 정책은 다음 Flight 모두에 적용합니다.

- FlightSchedule 기반 자동 생성 Flight
- Admin / SuperAdmin 수동 생성 Flight

동일 Flight에서 동일 Seat Number는 중복될 수 없습니다.

```text
UNIQUE(
  flight_id,
  seat_no
)
```

Seat 상태:

```text
AVAILABLE
HELD
RESERVED
UNAVAILABLE
```

Seat Hold Owner는 Nullable FK인
`held_reservation_id`로 표현합니다.

상태별 관계:

```text
AVAILABLE
→ held_reservation_id = NULL

HELD
→ held_reservation_id = Hold를 소유한 Reservation ID

RESERVED
→ held_reservation_id = NULL

UNAVAILABLE
→ held_reservation_id = NULL
```

Seat 자체에는 별도의 Hold 만료 시각을 저장하지 않습니다.

```text
Seat.held_until
→ 사용하지 않음

Reservation.hold_expires_at
→ Reservation 전체 Seat Hold 만료 시각
```

하나의 Reservation에서 확보한 Seat들은
동일한 `hold_expires_at`을 공유합니다.

`HELD` 상태가 종료되어
다른 Seat 상태로 전환될 때에는
`held_reservation_id`를 반드시 `NULL`로 정리합니다.

---

## 4. Reservation 관계

### 4.1 Reservation - ReservationFlight

Reservation과 Flight의 관계는
명시적인 `ReservationFlight` Entity로 관리합니다.

```text
Reservation 1 : N ReservationFlight
Flight      1 : N ReservationFlight
```

각 `ReservationFlight`는
별도의 `BIGINT id`를 단일 Primary Key로 사용합니다.

Composite Primary Key는 사용하지 않습니다.

Reservation은 하나 이상의 Flight를 포함합니다.

```text
ONE_WAY
Reservation
  |
  └─ ReservationFlight
       journey_role = OUTBOUND
       sequence = 1
```

```text
ROUND_TRIP
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

`ReservationFlight`에는 다음 Unique Constraint를 적용합니다.

```text
UNIQUE(
  reservation_id,
  flight_id
)

UNIQUE(
  reservation_id,
  journey_role
)

UNIQUE(
  reservation_id,
  sequence
)
```

따라서 하나의 Reservation 안에서:

- 동일 Flight 중복
- 동일 `journey_role` 중복
- 동일 `sequence` 중복

을 허용하지 않습니다.

`ONE_WAY` / `ROUND_TRIP`에 따른 정확한 Flight 개수와
Journey Role / Sequence 조합은
Application Validation으로 추가 보호합니다.

---

### 4.2 Reservation - ReservationPassenger

Reservation과 Passenger의 관계는
명시적인 `ReservationPassenger` Entity로 관리합니다.

```text
Reservation 1 : N ReservationPassenger
Passenger   1 : N ReservationPassenger
```

각 `ReservationPassenger`는
별도의 `BIGINT id`를 단일 Primary Key로 사용합니다.

Composite Primary Key는 사용하지 않습니다.

하나의 Reservation에는
한 명 이상의 Passenger가 포함됩니다.

다음 Unique Constraint를 적용합니다.

```text
UNIQUE(
  reservation_id,
  passenger_id
)

UNIQUE(
  reservation_id,
  sequence
)
```

따라서 동일 Reservation 안에서
동일 Passenger 또는 동일 Passenger 순서를
중복 사용할 수 없습니다.

`ROUND_TRIP`에서도 Passenger 자체를
출국 / 귀국별로 별도로 생성하지 않습니다.

하나의 Reservation 안에서는
동일 Passenger를 출국 / 귀국 Flight에 공통으로 사용하고,
Flight별 속성은 `PassengerFlight`에서 관리합니다.

다만 MVP의 Passenger는
Reservation-scoped Snapshot입니다.

```text
Reservation A
→ Passenger A

Reservation B
→ 새로운 Passenger
```

동일한 실제 사람이 다른 Reservation에 다시 탑승하더라도
기존 Passenger Row를 재사용하지 않습니다.

이 정책은 Application / Service Layer에서 보장합니다.

---

### 4.3 PassengerFlight

`PassengerFlight`는
명시적인 JPA Entity로 구현하며,
Passenger의 Flight별 예약 속성을 표현합니다.

각 `PassengerFlight`는
별도의 `BIGINT id`를 단일 Primary Key로 사용합니다.

Composite Primary Key는 사용하지 않습니다.

기본 관계:

```text
PassengerFlight
├─ Reservation
├─ Passenger
├─ Flight
├─ Seat (nullable)
├─ Companion Passenger (nullable)
├─ AgeCategory Snapshot
└─ fare_amount
```

`PassengerFlight`는
`reservation_id`를 통해 Reservation을 직접 참조합니다.

필요한 이유:

```text
Passenger
  |
  +-- Flight A
  |    → Seat 1A
  |    → ADULT
  |    → 확정 운임 A
  |
  +-- Flight B
       → Seat 3C
       → ADULT
       → 확정 운임 B
```

연령 Boundary가 있는 경우:

```text
Passenger
  |
  +-- 출국 Flight
  |    → INFANT
  |
  +-- 귀국 Flight
       → CHILD
```

처럼 동일 Passenger라도
Flight별 예약 속성이 달라질 수 있습니다.

따라서 다음 정보는 Flight별로 관리합니다.

- Seat
- `AgeCategory`
- Passenger / Flight별 확정 운임
- Infant Companion

`AgeCategory`는 다음 값을 사용합니다.

```text
ADULT
CHILD
INFANT
```

AgeCategory는 Passenger Table에 저장하지 않습니다.

```text
Passenger.birth_date
+
Flight 출발 Airport Local Date
        ↓
AgeCategory 계산
        ↓
PassengerFlight.age_category
```

Reservation이 `PENDING` 상태로 생성될 때
계산 결과를 `PassengerFlight.age_category`에
Snapshot으로 저장합니다.

따라서 이미 생성된 Reservation의 AgeCategory를
나중에 다시 계산하여 변경하지 않습니다.

Passenger / Flight별 최종 확정 운임은:

```text
PassengerFlight.fare_amount
→ DECIMAL(15,0)
```

으로 Snapshot 저장합니다.

`Reservation.total_amount`는
해당 Reservation에 속한 모든
`PassengerFlight.fare_amount`의 합계입니다.

SeatClass는 `PassengerFlight`에
별도 Column으로 중복 저장하지 않습니다.

```text
Seat.seat_class
```

를 해당 Passenger / Flight의 SeatClass 기준으로 사용합니다.

동일 Reservation에서
동일 Passenger와 동일 Flight를 중복 연결할 수 없습니다.

```text
UNIQUE(
  reservation_id,
  passenger_id,
  flight_id
)
```

또한 `PassengerFlight`의 Passenger와 Flight는
반드시 동일 Reservation에 포함되어 있어야 합니다.

```text
Passenger
→ 동일 Reservation의 ReservationPassenger에 존재

Flight
→ 동일 Reservation의 ReservationFlight에 존재
```

Membership 정합성은
Application / Service Layer에서 검증합니다.

Database에서는 복잡한 Composite Foreign Key 대신
일반 FK와 핵심 Unique Constraint를 사용합니다.

---

### 4.4 PassengerFlight - Seat

Seat 필요 여부는
`PassengerFlight.age_category` Snapshot을 기준으로 합니다.

```text
ADULT
→ seat_id 필수

CHILD
→ seat_id 필수

INFANT
→ seat_id = NULL
```

따라서 다음 조합은 허용하지 않습니다.

```text
ADULT / CHILD + seat_id = NULL
→ 불가

INFANT + seat_id 존재
→ 불가
```

이 정합성은 Application / Service Layer에서 검증합니다.

취소된 과거 Reservation의 PassengerFlight 기록은 유지될 수 있으므로
하나의 Seat가 시간 전체에 걸쳐
여러 PassengerFlight 이력과 연결될 수 있습니다.

동일 시점의 Seat 중복 확보는
Seat 상태와 Transaction / 동시성 제어를 통해 방지합니다.

`passenger_flight.seat_id`가 존재하는 경우
해당 Seat는 반드시 같은 `PassengerFlight.flight_id`가 가리키는
Flight에 속해야 합니다.

```text
PassengerFlight.flight_id
=
Seat.flight_id
```

이 정합성은 Application / Service Layer에서 검증합니다.

취소된 과거 Reservation의 Mapping 이력을 유지하므로
`passenger_flight.seat_id` 자체에는
전역 Unique Constraint를 적용하지 않습니다.

---

### 4.5 Infant Companion

Passenger가 해당 Flight에서 `INFANT`인 경우:

```text
passenger_flight.companion_passenger_id
```

를 사용하여 동반 Adult Passenger를 표현합니다.

AgeCategory별 Companion 규칙은 다음과 같습니다.

```text
INFANT
→ companion_passenger_id 필수

ADULT / CHILD
→ companion_passenger_id = NULL
```

Companion Passenger는 반드시 다음 조건을 만족해야 합니다.

```text
동일 Reservation Passenger
+
동일 Flight에 포함
+
해당 Flight의 PassengerFlight.age_category = ADULT
```

같은 Flight에서 Adult 1명은
최대 1명의 Infant만 동반할 수 있습니다.

이 정책은 Application / Service Layer에서 검증합니다.

복잡한 Composite Foreign Key 또는
별도의 Companion Mapping Entity는 사용하지 않습니다.

`ROUND_TRIP`에서도
Infant Companion은 Flight별로 독립적으로 관리합니다.

따라서 동일 Infant가
출국과 귀국 모두 `INFANT`인 경우에도
각 Flight에서 서로 다른 Adult를 지정할 수 있습니다.

예:

```text
OUTBOUND
Infant → Passenger A

RETURN
Infant → Passenger B
```

Frontend에서는
출국 Flight에서 선택한 Adult를
귀국 Flight의 기본값으로 제안할 수 있지만,
Backend 또는 Database에서 동일 Adult를 강제하지 않습니다.

---

## 5. Payment 관계

### 5.1 Reservation - Payment

```text
Reservation 1 : N Payment
```

하나의 Payment는
하나의 Mock Payment 시도를 나타냅니다.

별도의 `PaymentAttempt` Entity는 사용하지 않습니다.

Payment 상태:

```text
PENDING
SUCCESS
FAILED
CANCELLED
REFUNDED
```

하나의 Reservation당 최대 3회의 Payment를 허용합니다.

```text
attempt_no = 1
attempt_no = 2
attempt_no = 3
```

---

### 5.2 Payment Idempotency

동일 Mock Payment 요청의 중복 처리를 방지하기 위해
Payment에 `idempotency_key`를 저장합니다.

초안 Constraint:

```text
UNIQUE(
  reservation_id,
  idempotency_key
)
```

하나의 Reservation에는
최대 하나의 `SUCCESS` Payment만 존재해야 합니다.

구체적인 Database 보호 방식은
MySQL 및 Transaction 구조를 검토한 뒤 확정합니다.

---

## 6. AuditLog 관계

### 6.1 Member - AuditLog

```text
Member 1 : N AuditLog
```

`actor_member_id`는
관리 작업을 수행한 Admin 또는 SuperAdmin을 참조합니다.

최소 Audit 대상:

- Flight 취소
- SuperAdmin Reservation 강제 취소
- 중요 Master Data 생성
- 중요 Master Data 수정
- 중요 Master Data 비활성화

---

### 6.2 Audit Target

Audit 대상은 초안에서 다음 구조로 표현합니다.

```text
target_type
target_id
```

예:

```text
target_type = FLIGHT
target_id   = 101
```

또는:

```text
target_type = RESERVATION
target_id   = 5001
```

`target_id`는 여러 종류의 Entity를 가리킬 수 있으므로
현재 ERD에서는 특정 Entity와 FK Relation으로 연결하지 않습니다.

Audit Target의 세부 구조는
Data Design 확정 과정에서 최종 결정합니다.

---

## 7. 주요 Unique Constraint

현재 Data Design 초안 기준
주요 Unique Constraint는 다음과 같습니다.

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

reservation_flight(reservation_id, journey_role)
    UNIQUE

reservation_flight(reservation_id, sequence)
    UNIQUE

reservation_passenger(reservation_id, passenger_id)
    UNIQUE

reservation_passenger(reservation_id, sequence)
    UNIQUE

passenger_flight(reservation_id, passenger_id, flight_id)
    UNIQUE

payment(reservation_id, idempotency_key)
    UNIQUE
```

추가 Constraint가 필요한 미확정 Domain은
8장의 항목을 기준으로 추후 반영합니다.

---

## 8. ERD에서 아직 확정하지 않는 항목

다음 사항은 현재 Draft ERD에서
최종 확정하지 않습니다.

### Seat

- [ ] `AircraftSeat`의 통로 표현 방식

### Payment

- [ ] 하나의 Reservation당 최대 하나의 `SUCCESS` Payment를 보장하는 Database 구조
- [ ] `attempt_no` 추가 Unique Constraint 여부
- [ ] Idempotency 동일 Key + 다른 Request 처리 방식

### Audit

- [ ] Audit Target 구조 최종 확정
- [ ] Audit 변경 전 / 후 값 저장 여부

---

## 9. ERD 변경 원칙

ERD는 `05-data-api-design.md`의 시각적 표현입니다.

따라서 ERD 자체에서
새로운 Business Rule 또는 Data Model을 먼저 확정하지 않습니다.

변경 순서:

```text
Domain Policy 변경 필요
        |
        v
02-domain-policy.md
        |
        v
04-system-design.md 검토
        |
        v
05-data-api-design.md 수정
        |
        v
ERD 수정
```

단순한 Data Model 변경인 경우:

```text
05-data-api-design.md 수정
        |
        v
ERD 수정
```

ERD와 `05-data-api-design.md`의 내용이 충돌하는 경우
`05-data-api-design.md`를 우선 기준으로 합니다.