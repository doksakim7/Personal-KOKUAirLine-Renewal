# KOKU Airline Renewal - System Design

## 1. 문서 목적

본 문서는 KOKU Airline Renewal의 시스템 구조와 기술적 구현 원칙을 정의합니다.

본 문서의 목적은 다음과 같습니다.

- Backend와 Frontend의 전체 시스템 구조를 정의합니다.
- 인증 및 인가 구조를 정의합니다.
- Transaction 및 동시성 처리의 기본 원칙을 정의합니다.
- 날짜 및 시간 처리 기준을 정의합니다.
- 외부 Flight API와 AI 기능의 연동 구조를 정의합니다.
- Cache, 비동기 처리 및 외부 의존성 도입 원칙을 정의합니다.
- Local 개발환경, CI/CD 및 배포 구조를 정의합니다.
- Logging, Error Handling 및 보안 기본 원칙을 정의합니다.
- AI Agent가 기술 구조를 임의로 변경하지 않도록 구현 기준을 제공합니다.

비즈니스 규칙은 `02-domain-policy.md`를 기준으로 합니다.

Frontend의 화면 구조와 사용자 흐름은 `03-ui-design.md`를 기준으로 합니다.

Entity 관계, Database Column, Index, Constraint 및 API Contract는
`05-data-api-design.md`에서 구체적으로 정의합니다.

---

### 1.1 설계 원칙

본 시스템은 다음 원칙을 따릅니다.

1. Domain Policy와 기술 구현을 분리합니다.
2. Backend를 비즈니스 규칙의 최종 검증 주체로 사용합니다.
3. Frontend Validation은 사용자 편의를 위한 1차 검증으로 사용합니다.
4. Database Constraint를 이용할 수 있는 정합성 규칙은 Application 검증과 함께 보호합니다.
5. 내부 KOKU Airline 데이터와 외부 실제 항공편 데이터를 명확하게 분리합니다.
6. 외부 시스템 장애가 내부 예약 시스템에 전파되지 않도록 합니다.
7. 필요성이 확인되지 않은 Infrastructure 기술을 선제적으로 도입하지 않습니다.
8. 동시성 및 성능 관련 기술은 테스트 결과를 기반으로 선택합니다.
9. Secret과 민감정보를 Source Repository에 저장하지 않습니다.
10. 중요한 Architecture 변경은 Human Review 후 반영합니다.

---

## 2. 전체 시스템 구조

### 2.1 기본 Architecture

KOKU Airline Renewal은 Frontend와 Backend를 분리한 Web Application으로 구성합니다.

```text
User Browser
     |
     v
React Frontend
     |
     | HTTPS / REST API
     v
Spring Boot Backend
     |
     +----------------------+
     |                      |
     v                      v
MySQL                 External Services
                           |
                           +-- Google OAuth
                           |
                           +-- External Flight API
                           |
                           +-- AI Model API
```

Frontend는 사용자 인터페이스와 사용자 입력을 담당합니다.

Backend는 다음 책임을 가집니다.

- 인증 및 인가
- Domain Policy 검증
- KOKU Airline Flight 관리
- 운임 계산
- Passenger Validation
- Seat 상태 관리
- Reservation 처리
- Mock Payment 처리
- Transaction 관리
- 외부 Flight API 호출
- AI 검색 Orchestration
- 데이터 영속화

Database는 KOKU Airline 내부 업무 데이터의 Source of Truth로 사용합니다.

---

### 2.2 Backend Architecture

Backend는 Spring Boot 기반 Layered Architecture를 기본으로 합니다.

```text
Controller
    |
    v
Application / Service
    |
    v
Domain
    |
    v
Repository
    |
    v
Database
```

필요에 따라 외부 시스템 연동을 별도 Adapter 계층으로 분리합니다.

```text
Controller
    |
Application Service
    |
    +-- Domain
    |
    +-- Repository
    |
    +-- External Adapter
           |
           +-- Google OAuth
           +-- Flight API
           +-- AI
```

구체적인 Package 구조는 Backend 초기 구현 전에 확정합니다.

---

### 2.3 Frontend Architecture

Frontend는 React + Vite + TypeScript를 사용합니다.

Frontend의 주요 책임은 다음과 같습니다.

- Routing
- 화면 Rendering
- 사용자 Input 관리
- Client-side Validation
- 인증 상태 표현
- API 호출
- Locale 관리
- Reservation 진행 상태 표현
- Hold Countdown 표시

Backend의 Domain 상태를 Frontend에서 임의로 재정의하지 않습니다.

특히 다음 값은 Backend 응답을 기준으로 합니다.

- Reservation 상태
- Payment 상태
- Seat 상태
- Reservation 취소 가능 여부
- Seat Hold 만료 여부
- Flight 예약 가능 여부
- 운임
- Member 권한

---

## 3. 기술 스택

### 3.1 Backend

- Java 21
- Spring Boot
- Spring Web
- Spring Data JPA
- Spring Security
- Spring OAuth2 Client
- JWT
- Bean Validation
- MySQL
- Gradle

---

### 3.2 Frontend

- React
- Vite
- TypeScript
- REST API
- 한국어 / 일본어 i18n

i18n Library는 Frontend 초기 구성 단계에서 확정합니다.

---

### 3.3 Infrastructure

- Docker
- Docker Compose
- AWS
- GitHub Actions

---

### 3.4 AI / External

- Spring AI
- 외부 Flight API
- Google OAuth 2.0

---

### 3.5 필요 시 추가 검토

다음 기술은 MVP 시작 시점부터 필수로 도입하지 않습니다.

- Redis
- Kafka
- k6 또는 기타 부하 테스트 도구

Redis는 Cache 또는 동시성 문제 해결에서 필요성이 확인된 경우 검토합니다.

Kafka는 비동기 Event 처리 필요성이 실제로 발생한 경우 검토합니다.

동시성 또는 성능 개선 기술은 반드시 도입 전후 결과를 비교할 수 있어야 합니다.

---

## 4. 인증 및 인가 구조

### 4.1 인증 방식

MVP에서는 다음 인증 방식을 제공합니다.

1. Email / Password 기반 LOCAL 인증
2. Google OAuth 2.0 인증

두 인증 방식은 최종적으로 동일한 Member와 권한 체계를 사용합니다.

인증 수단은 `AuthAccount`에서 관리하며,
서비스 사용자와 권한은 `Member`에서 관리합니다.

구체적인 관계는 `05-data-api-design.md`에서 정의합니다.

---

### 4.2 LOCAL 인증

LOCAL 로그인 흐름:

```text
Frontend
   |
   | Email / Password
   v
Authentication API
   |
   v
Email Normalize
   |
   v
AuthAccount 조회
   |
   v
PasswordEncoder 검증
   |
   v
Member 상태 확인
   |
   v
Token 발급
```

Email은 `02-domain-policy.md`의 Email 정규화 정책을 적용한 뒤 비교합니다.

`WITHDRAWN` Member는 인증에 성공할 수 없습니다.

Password는 평문으로 저장하거나 Logging하지 않습니다.

---

### 4.3 Google OAuth

Google OAuth 인증은 Spring Security OAuth2 Client를 사용합니다.

기본 흐름:

```text
Frontend
   |
   v
Backend OAuth Login
   |
   v
Google Authorization
   |
   v
Google Callback
   |
   v
GOOGLE AuthAccount 조회
   |
   +-- 존재 → Member 상태 확인
   |
   +-- 없음 → Email 기준 Member 확인
                   |
                   +-- 없음 → Member + AuthAccount 생성
                   |
                   +-- LOCAL 존재 → 재인증 후 계정 연결
```

Google OAuth에서 전달받은 Email도
`02-domain-policy.md`의 Email 정규화 정책을 적용한 뒤
기존 Member의 Email과 비교합니다.

Google Email과 기존 LOCAL Member의 Email이 동일하더라도
Email 일치만으로 AuthAccount를 자동 연결하지 않습니다.

LOCAL Password 재인증에 성공한 경우에만
기존 Member에 GOOGLE AuthAccount를 연결합니다.

---

## 5. JWT 및 Token 정책

> 본 절은 System Design 검토 과정에서 최종 확정합니다.

### 5.1 기본 방향

Backend API 인증은 JWT 기반 Stateless 인증을 기본 방향으로 합니다.

다음 Token을 구분합니다.

- Access Token
- Refresh Token

Access Token은 인증이 필요한 API 호출에 사용합니다.

Refresh Token은 Access Token 재발급에만 사용합니다.

---

### 5.2 Access Token

초안 정책:

- 짧은 만료시간 사용
- 사용자 식별자와 권한 정보를 Claim에 포함
- Password, Email 등 불필요한 개인정보는 포함하지 않음
- API 요청 시 Backend에서 Signature와 만료시간 검증

Access Token의 구체적인 만료시간은 구현 전에 확정합니다.

---

### 5.3 Refresh Token

초안 정책:

Refresh Token은 Access Token보다 긴 만료시간을 사용합니다.

Refresh Token을 이용해 새로운 Access Token을 발급합니다.

Refresh Token 저장 및 Rotation 여부는 본 문서 검토 과정에서 확정합니다.

검토 대상:

- Database 저장
- Redis 저장
- Refresh Token Rotation
- Token Family / 재사용 탐지

MVP 복잡도를 고려하여 필요 이상의 Token Infrastructure는 도입하지 않습니다.

---

### 5.4 Frontend Token 저장

Token 저장 방식은 보안을 우선하여 결정합니다.

초안 방향:

- Access Token: Frontend Memory 기반 관리 우선 검토
- Refresh Token: HttpOnly / Secure Cookie 우선 검토

`localStorage`에 장기 인증 Token을 저장하는 방식은
XSS 노출 위험 때문에 기본안으로 사용하지 않습니다.

구체적인 Cookie Attribute 및 CSRF 대응 방식은
JWT 정책 확정 시 함께 정의합니다.

---

### 5.5 로그아웃

로그아웃 시 Frontend의 인증 상태를 제거합니다.

Refresh Token을 Server에서 관리하는 구조를 선택하는 경우
로그아웃 시 해당 Refresh Token을 사용할 수 없도록 처리합니다.

Access Token 즉시 폐기 기능이 필요한지는
Access Token 만료시간과 함께 결정합니다.

---

## 6. 권한 구조

서비스의 사용자 Role과 Spring Security 권한 Code를 구분합니다.

서비스 Role:

- `Member`
- `Admin`
- `SuperAdmin`

Spring Security 권한 Code:

- `USER`
- `ADMIN`
- `SUPERADMIN`

권한 의미:

```text
USER
 └─ Member 기능

ADMIN
 └─ 운영 조회 및 Flight 관리

SUPERADMIN
 └─ ADMIN 권한
    + Master Data 변경
    + Reservation 강제 취소
```

`Guest`는 인증되지 않은 사용자이므로 별도 Spring Security 권한 Code를 가지지 않습니다.

Backend API에서 권한을 최종 검증합니다.

Frontend의 메뉴 숨김 또는 Button Disabled만으로
인가를 보장하지 않습니다.

---

## 7. Transaction 설계 원칙

### 7.1 기본 원칙

데이터 정합성이 필요한 Business Operation은
Spring Transaction Boundary 안에서 처리합니다.

Controller에서 Transaction을 직접 관리하지 않고
Application / Service 계층에서 Transaction Boundary를 정의합니다.

---

### 7.2 Reservation 시작 Transaction

Reservation 시작은 다음 작업을 하나의 논리적 단위로 처리합니다.

```text
Reservation 조건 검증
        |
        v
선택한 모든 Seat 확보 가능 여부 검증
        |
        v
PENDING Reservation 생성
+ Reservation 번호 발급
+ 최종 결제 예정 금액 확정
        |
        v
선택한 모든 Seat
AVAILABLE → HELD
        |
        v
전체 Transaction Commit
```

위 작업은 하나의 Transaction Boundary 안에서 처리하며,
구체적인 Entity Persist 및 Flush 순서는
`05-data-api-design.md`의 데이터 모델에 따라 결정합니다.

Seat 확보 중 하나라도 실패하면 전체 Reservation 시작을 실패합니다.

`ROUND_TRIP`에서도 출국 / 귀국 Seat 전체를
하나의 확보 단위로 처리합니다.

Partial Success를 허용하지 않습니다.

---

### 7.3 Mock Payment 성공 Transaction

Mock Payment 성공 시 다음 상태 변경은
정합성을 유지하도록 하나의 Business Operation으로 처리합니다.

```text
Payment
PENDING → SUCCESS

Reservation
PENDING → CONFIRMED

Seat
HELD → RESERVED
```

일부 상태만 성공적으로 반영되는 결과를 허용하지 않습니다.

---

### 7.4 Reservation 취소 Transaction

예약 취소 시 다음 상태 변경의 정합성을 보장합니다.

```text
Reservation
CONFIRMED → CANCELLED

Payment
SUCCESS → REFUNDED

Seat
RESERVED → AVAILABLE
```

`ROUND_TRIP`에서는 출국 / 귀국 Flight를 포함한
Reservation 전체를 하나의 취소 단위로 처리합니다.

---

### 7.5 Hold 만료 Transaction

Seat Hold가 만료된 `PENDING` Reservation은 다음과 같이 처리합니다.

```text
Reservation
PENDING → CANCELLED

Seat
HELD → AVAILABLE

현재 PENDING Payment가 존재하는 경우
PENDING → CANCELLED
```

기존 `FAILED` Payment는 변경하지 않습니다.

---

## 8. Seat 동시성 설계

### 8.1 목표

동일 Flight의 동일 Seat를 여러 사용자가 동시에 확보하려고 하더라도
최종적으로 하나의 Reservation만 성공해야 합니다.

---

### 8.2 MVP 기본 원칙

동시성 제어 방식은 구현 전에 단순히 결정하지 않고
M4에서 동시성 테스트를 통해 검증합니다.

검토 대상:

- Database Unique Constraint
- Pessimistic Lock
- Optimistic Lock
- Conditional Update
- Redis Distributed Lock

Redis Distributed Lock을 기본 전제로 하지 않습니다.

단일 Backend / 단일 Database 환경에서
Database 기반 방식으로 충분한지 먼저 검증합니다.

---

### 8.3 동시성 테스트

최소 다음 Scenario를 검증합니다.

```text
N개의 요청
      |
      v
동일 Flight / 동일 Seat 예약 시도
      |
      v
성공 Reservation = 1
실패 Reservation = N - 1
```

왕복 Reservation에서도 한쪽 Flight의 Seat 확보 실패 시
다른 Flight의 Seat가 `HELD` 상태로 남지 않아야 합니다.

---

## 9. Seat Hold 처리

### 9.1 Hold 시간

Domain Policy에 따라 Seat Hold 기본 시간은 1시간입니다.

Hold 만료 판단은 Frontend Countdown이 아니라 Backend 시간을 기준으로 합니다.

Frontend Countdown은 사용자 안내 목적입니다.

---

### 9.2 Hold 만료시각

각 `PENDING` Reservation은 Hold 만료를 판단할 수 있는
기준 시각을 가져야 합니다.

구체적인 Column은 `05-data-api-design.md`에서 정의합니다.

Hold 종료 시각은 Reservation에 포함된
가장 빠른 Flight 출발 예정 시각을 초과할 수 없습니다.

---

### 9.3 Hold 만료 처리 방식

MVP에서는 다음 두 방식을 함께 사용합니다.

1. Business API 진입 시 방어적인 Hold 만료 검증
2. 주기적인 Scheduled Job

Seat Hold 만료 Scheduled Job은
기본적으로 1분 주기로 실행합니다.

```text
1분 주기 Scheduled Job
        |
        v
만료된 PENDING Reservation 조회
        |
        v
Reservation
PENDING → CANCELLED

Seat
HELD → AVAILABLE

현재 PENDING Payment가 존재하는 경우
PENDING → CANCELLED
```

Scheduled Job 실행이 일시적으로 지연되더라도
Business API 진입 시 Backend `Clock`을 기준으로
Hold 만료 여부를 다시 검증합니다.

따라서 Scheduler 실행 여부만으로
Reservation 또는 Payment 가능 여부를 판단하지 않습니다.

기존 `FAILED` Payment는 변경하지 않습니다.

---

## 10. 날짜 및 시간 처리

### 10.1 기본 원칙

시간 관련 Business Rule은 Backend 시간을 기준으로 판단합니다.

Frontend Device 시간은
Business Rule 판정 기준으로 사용하지 않습니다.

현재 시각과 비교하는 대표 Business Rule:

- 신규 Reservation 2시간 제한
- Seat Hold 만료
- Member 예약 취소 24시간 제한
- Flight `DEPARTED` 여부

Flight의 날짜 또는 시간대 의미가 필요한 Business Rule은
해당 Flight의 출발 Airport Local Date / Time을 기준으로 판단합니다.

대표 대상:

- 운임 시간대
- 운임 요일
- Passenger 탑승일 기준 연령
- ROUND_TRIP Date Rule
- Flight 출발일

---

### 10.2 Time Zone

각 Airport는 IANA Time Zone ID를 가집니다.

MVP 기준:

```text
한국 Airport
→ Asia/Seoul

일본 Airport
→ Asia/Tokyo
```

한국과 일본은 현재 모두 UTC+9이지만,
시스템의 시간 처리를 고정 Offset `+09:00`에 종속시키지 않습니다.

Flight의 절대 시각은 UTC 기준으로 저장합니다.

Backend에서는 Flight의 절대 시각 계산과 비교에
`Instant`를 기본으로 사용합니다.

```text
Database UTC Time
        |
        v
Instant
        |
        + Airport ZoneId
        |
        v
Airport Local Date / Time
```

Flight의 출발 / 도착 시각을 사용자에게 표시할 때는
각 Airport의 `ZoneId`를 사용하여 현지시각으로 변환합니다.

Frontend에서는 현지시각을
24시간제 `00:00 ~ 23:59` 형식으로 표시합니다.

API Date / Time은
Offset을 포함한 ISO-8601 형식을 사용합니다.

예:

```text
2026-09-10T09:30:00+09:00
```

Database의 절대 시각 Column은
MySQL `DATETIME(6)`을 사용하고 UTC 기준 값을 저장합니다.

```text
Java
Instant

↓

MySQL
DATETIME(6)

↓

저장 기준
UTC
```

`DATETIME(6)` 자체에는 Time Zone 정보가 포함되지 않으므로
Application과 Database 사이의 시간 해석 기준을 UTC로 통일합니다.

JDBC / Hibernate의 Database Time Zone도
UTC 기준으로 구성합니다.

날짜 자체만 의미하는 값은
절대 시각과 구분하여 `LocalDate` / MySQL `DATE`를 사용합니다.

구체적인 Entity별 Column Type은
`05-data-api-design.md`를 기준으로 합니다.

---

### 10.3 Backend Clock

Backend의 현재 시각 Source는
주입 가능한 `Clock`을 사용합니다.

Business Logic에서 직접 다음 호출을 사용하는 방식은
기본 구현으로 사용하지 않습니다.

```text
LocalDateTime.now()
Instant.now()
```

대신 Application에서 관리하는 `Clock`을 통해
현재 시각을 획득합니다.

예:

```java
Instant now = clock.instant();
```

이를 통해 다음 Boundary를
고정된 시간 기준으로 테스트할 수 있습니다.

- 신규 Reservation 2시간 전
- Reservation 취소 24시간 전
- Seat Hold 만료
- Passenger 연령 Boundary
- Flight `DEPARTED` 전환

운영 환경에서는 System Clock을 사용하고,
Test에서는 Fixed Clock을 주입할 수 있도록 구성합니다.

---

### 10.4 시간 변환 원칙

절대 시각 비교와 Airport Local Time 기반 판단을 구분합니다.

절대 시각 비교:

```text
Instant ↔ Instant
```

대표 대상:

- 현재 시각과 Flight 출발시각 비교
- Hold 만료 여부
- 취소 가능 시간
- Flight DEPARTED 여부

Airport Local Time 판단:

```text
Instant
+
Airport ZoneId
→
Local Date / Time
```

대표 대상:

- 운임 시간대
- 운임 요일
- Passenger 연령 기준 탑승일
- ROUND_TRIP 날짜 비교

Local Date / Time을
시스템 전체의 절대 시각 저장값으로 사용하지 않습니다.

---

## 11. Flight 상태 처리

### 11.1 `DEPARTED` 전환

KOKU Airline은 실제 운항 시스템과 연결되지 않으므로
실제 항공기의 출발 Event를 전달받지 않습니다.

MVP에서는 출발 예정 시각이 지난 `SCHEDULED` Flight를
`DEPARTED` 상태로 처리합니다.

Flight `DEPARTED` 전환 Scheduled Job은
기본적으로 1분 주기로 실행합니다.

```text
1분 주기 Scheduled Job
        |
        v
SCHEDULED Flight 조회
        |
        v
Backend Clock과 departure_at 비교
        |
        v
departure_at <= 현재 시각
        |
        v
SCHEDULED → DEPARTED
```

시간 비교는 UTC 기준 `Instant`로 수행합니다.

Scheduled Job 실행이 지연되더라도
Business API에서도 Backend `Clock`과
Flight 출발 예정시각을 직접 비교합니다.

따라서 다음 Business Rule은
Flight의 저장된 상태 값만으로 판단하지 않습니다.

- 신규 Reservation 가능 여부
- Member Reservation 취소 가능 여부
- SuperAdmin 강제 취소 가능 여부

Scheduler는 상태를 주기적으로 동기화하기 위한 수단이며,
시간 기반 Business Rule의 최종 판단은
Backend 현재 시각을 함께 사용합니다.

---

## 12. 외부 실제 항공편 연동

### 12.1 데이터 경계

외부 Flight API에서 조회한 실제 항공편 데이터와
KOKU Airline 내부 Flight 데이터는 명확하게 분리합니다.

외부 실제 항공편은 다음 기능에만 사용합니다.

- 실제 항공편 조회
- 조건 비교
- AI 항공편 검색 및 추천

외부 Flight API에서 조회한 항공편을
KOKU Airline의 판매 가능한 내부 `Flight`로 취급하지 않습니다.

외부 실제 항공편에 대해서는 다음 작업을 수행하지 않습니다.

- KOKU Airline Reservation 생성
- Seat 확보
- Mock Payment
- 실제 결제
- 실제 발권

---

### 12.2 호출 구조

Frontend에서 외부 Flight API를 직접 호출하지 않습니다.

기본 호출 구조는 다음과 같습니다.

```text
Frontend
   |
   v
Spring Boot Backend
   |
   v
External Flight Service
   |
   v
Flight API Adapter
   |
   v
External Flight API
```

외부 Flight API Key 및 Credential은 Backend에서 관리합니다.

Frontend에는 외부 API Secret을 전달하지 않습니다.

---

### 12.3 지원 노선 검증

외부 Flight API 호출 전에 Backend에서
Domain Policy의 지원 노선 범위를 검증합니다.

MVP에서는 다음 범위만 허용합니다.

```text
한국 → 일본
일본 → 한국
```

지원하지 않는 Route가 입력되면
외부 Flight API를 호출하지 않고 요청을 거부합니다.

AI 항공편 검색에서도 동일한 검증을 적용합니다.

---

### 12.4 외부 API 장애

다음 상황을 고려합니다.

- Connection Timeout
- Read Timeout
- HTTP 4xx
- HTTP 5xx
- Rate Limit
- 잘못되거나 예상하지 못한 Response
- 일시적인 Network 장애

외부 Flight API 장애는
KOKU Airline 내부 Reservation 시스템에 영향을 주지 않아야 합니다.

예:

```text
External Flight API 장애

→ 실제 항공편 검색 실패
→ AI 항공편 검색 일부 또는 전체 실패

하지만

→ KOKU Flight 검색 정상
→ Seat 조회 정상
→ Reservation 정상
→ Mock Payment 정상
```

---

### 12.5 Retry

외부 API Retry는 모든 실패에 무조건 적용하지 않습니다.

일시적인 장애로 판단할 수 있는 경우에만
제한적인 Retry 적용을 검토합니다.

예:

- Network 일시 장애
- 일부 5xx Response

다음과 같은 요청은 기본적으로 Retry 대상으로 사용하지 않습니다.

- 잘못된 Request
- 인증 실패
- 지원하지 않는 Route
- 명확한 Client Error

구체적인 Retry 횟수와 Backoff 정책은
실제 사용할 Flight API가 확정된 이후 결정합니다.

---

### 12.6 Cache

외부 Flight API 결과는
KOKU Airline의 영구 업무 데이터로 저장하지 않습니다.

MVP 초기에는 Cache를 필수 Infrastructure로 도입하지 않습니다.

다음 문제가 실제로 확인되면 Cache 도입을 검토합니다.

- External API Rate Limit
- 반복 호출에 따른 비용
- 높은 Response Latency
- 동일 조건의 반복 요청

Cache 도입 시 다음 사항을 함께 결정합니다.

- Cache Key
- TTL
- Cache 대상 Response
- Invalid / Expiration 정책

Redis는 Cache 필요성이 확인된 이후 도입 여부를 결정합니다.

---

## 13. AI 항공편 검색

### 13.1 기본 역할

AI 항공편 검색은
외부 실제 항공편 데이터를 기반으로 동작합니다.

AI의 역할은 다음과 같습니다.

- 자연어 검색 조건 해석
- 검색 조건 구조화
- 사용자 의도 설명
- 추천 결과 설명
- 추천 이유 생성

AI는 다음 작업을 수행하지 않습니다.

- KOKU Airline Reservation 생성
- Seat 확보
- Mock Payment 실행
- 외부 실제 항공편 예약
- 실제 결제
- 실제 발권

AI는 Transaction을 직접 실행하는 주체로 사용하지 않습니다.

---

### 13.2 처리 흐름

기본 처리 흐름은 다음과 같습니다.

```text
Member 자연어 입력
        |
        v
Spring AI
        |
        v
Structured Search Criteria
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
실제 Flight Data
        |
        v
Application Filter / Rank
        |
        v
AI 추천 설명
        |
        v
Frontend
```

AI가 외부 Flight API를 임의의 조건으로
무제한 호출하도록 구성하지 않습니다.

Application이 외부 Tool 호출의 입력과 범위를 통제합니다.

---

### 13.3 Structured Output

사용자의 자연어 입력은
가능한 경우 Structured Output으로 변환합니다.

예:

```text
출발 Airport
도착 Airport
출발 Date
귀국 Date
출발 시간 선호
가격 조건
직항 / 경유 조건
기타 검색 조건
```

AI가 생성한 Structured Output은
외부 API 호출 전에 Application에서 다시 검증합니다.

AI Output 자체를 신뢰하여
바로 외부 API 또는 Business Logic에 사용하지 않습니다.

구체적인 Request / Response DTO는
`05-data-api-design.md`에서 정의합니다.

---

### 13.4 Deterministic Filtering

객관적으로 비교하거나 판단할 수 있는 조건은
Java Application Logic에서 처리하는 것을 원칙으로 합니다.

예:

- Airport
- Date
- Departure Time
- 가격
- 직항 / 경유 여부
- 정렬
- 지원 Route Validation

AI가 이러한 객관적 Filtering 결과를
임의로 변경하지 않습니다.

AI는 필터링된 결과를 기반으로
추천 설명과 사용자 친화적인 요약을 생성합니다.

---

### 13.5 AI 추천 결과

MVP에서는 조건에 적합한 후보를
Application에서 필터링 및 정렬한 뒤
AI가 추천 이유를 설명하는 구조를 사용합니다.

필요한 경우 상위 후보를 제한하여 AI에 전달할 수 있습니다.

예:

```text
External Flight API 결과

        ↓

Application Filtering

        ↓

Application Ranking

        ↓

상위 후보

        ↓

AI 추천 설명
```

구체적인 추천 개수는
AI 기능 구현 시 확정합니다.

---

### 13.6 Hallucination 방지

AI는 실제 Flight Data를 임의로 생성하거나 변경해서는 안 됩니다.

AI가 생성해서는 안 되는 정보:

- 존재하지 않는 Flight Number
- 외부 Flight API에 없는 Airline
- 실제 데이터에 없는 가격
- 실제 데이터에 없는 출발 시각
- 실제 데이터에 없는 도착 시각
- 확인되지 않은 예약 가능 여부

사용자에게 표시하는 실제 Flight 정보는
외부 Flight API 결과를 Source of Truth로 사용합니다.

AI가 생성한 설명과 실제 Flight Data는
Frontend에서도 명확하게 구분합니다.

---

### 13.7 AI 장애 격리

AI 응답 생성이 실패하더라도
외부 Flight API의 실제 데이터와
내부 KOKU Airline 데이터가 변경되어서는 안 됩니다.

AI 장애 시 다음과 같은 처리가 가능합니다.

```text
AI 추천 생성 실패

→ 오류 안내
→ 다시 시도
→ 일반 실제 항공편 검색으로 이동
```

AI 장애가 내부 Reservation 기능에 영향을 주지 않아야 합니다.

---

## 14. Error Handling

### 14.1 기본 원칙

Backend Error Response는
일관된 구조로 제공하는 것을 원칙으로 합니다.

구체적인 Error Response Contract는
`05-data-api-design.md`에서 정의합니다.

최소 다음 Error Category를 구분할 수 있어야 합니다.

- Validation Error
- Authentication Error
- Authorization Error
- Resource Not Found
- Business Rule Violation
- Conflict
- External API Error
- Internal Server Error

---

### 14.2 Validation Error

다음과 같은 입력 오류를 처리합니다.

예:

- 필수값 누락
- Email 형식 오류
- Password 정책 위반
- 잘못된 Airport
- 잘못된 Date
- Passenger 입력 오류

Bean Validation과 Business Validation의 역할을 구분합니다.

단순한 입력 형식 검증은 Bean Validation을 사용할 수 있습니다.

Domain Policy가 필요한 검증은
Application / Domain Logic에서 처리합니다.

---

### 14.3 Business Exception

Domain Policy 위반은
명확한 Business Exception으로 처리합니다.

예:

- 예약할 수 없는 Flight
- 이미 다른 사용자가 확보한 Seat
- Seat Hold 만료
- Mock Payment 최대 횟수 초과
- Reservation 취소 제한시간 미충족
- 다른 Member의 Reservation 접근
- Child / Infant 정책 위반
- 지원하지 않는 Route

Business Exception이 발생해도
일부 데이터만 변경된 상태로 남지 않도록
Transaction Boundary를 유지합니다.

---

### 14.4 인증 / 인가 오류

인증되지 않은 사용자가
인증이 필요한 API에 접근하면 요청을 거부합니다.

권한이 부족한 사용자가
허용되지 않은 API에 접근하면 요청을 거부합니다.

Frontend의 화면 또는 Button 노출 여부와 관계없이
Backend에서 최종적으로 권한을 검증합니다.

---

### 14.5 외부 시스템 오류

외부 시스템 오류는 내부 시스템 오류와 구분합니다.

대상:

- Google OAuth
- External Flight API
- AI Model API

외부 시스템 오류의 세부 내부 정보나
Credential을 Client에 노출하지 않습니다.

---

## 15. Logging 및 Audit

### 15.1 Logging 기본 원칙

Application 운영 및 문제 분석에 필요한 정보를
적절한 Log Level로 기록합니다.

주요 대상:

- Application Error
- External API 실패
- AI 요청 실패
- Scheduled Job 실행 결과
- 주요 Reservation 처리 실패
- Mock Payment 처리 실패
- 인증 관련 주요 오류

---

### 15.2 Logging 금지 정보

다음 정보는 Log에 출력하지 않습니다.

- Password
- Password Hash
- Access Token 전체 값
- Refresh Token
- OAuth Client Secret
- External Flight API Key
- AI API Key
- 테스트용 여권번호

민감한 Header 또는 Cookie 값을
그대로 Logging하지 않습니다.

---

### 15.3 Business Logging

모든 정상 Business Operation을
무조건 상세 Logging하지 않습니다.

운영 및 장애 분석에 필요한 핵심 Event를 중심으로 기록합니다.

예:

- Reservation 생성 성공 / 실패
- Mock Payment 처리 결과
- Reservation 취소
- Flight 취소
- Hold 만료 처리
- External API 장애

사용자 개인정보가 불필요하게 Log에 포함되지 않도록 합니다.

---

### 15.4 Audit Log

운영상 중요한 관리자 Action은
일반 Application Log와 별도로
Audit Log로 기록해야 합니다.

최소 대상:

- Flight 취소
- SuperAdmin Reservation 강제 취소
- 중요 Master Data 생성
- 중요 Master Data 수정
- 중요 Master Data 비활성화

Audit 정보는 최소 다음 내용을 식별할 수 있어야 합니다.

- 수행한 관리자
- Action
- 대상
- 처리 시각
- 변경 또는 취소 사유

구체적인 Audit Entity와 Column은
`05-data-api-design.md`에서 정의합니다.

---

## 16. 테스트용 Passenger 정보 보호

### 16.1 기본 원칙

본 프로젝트는 실제 항공권 발권 서비스가 아니므로
실제 여권정보를 입력받지 않습니다.

Passenger의 여권정보는
Domain Policy에 따라 시스템에서 테스트용으로 생성합니다.

---

### 16.2 보호 대상

테스트용 데이터이더라도
여권번호는 민감정보와 유사한 방식으로 취급합니다.

다음 원칙을 적용합니다.

- Log 출력 금지
- 필요한 API에서만 전달
- 불필요한 화면에서 노출 금지
- 실제 개인정보 사용 금지
- 실제 Passport Number 입력 금지

---

### 16.3 API 노출

Passenger 정보를 사용하는 모든 API에서
여권번호 전체가 항상 필요한 것은 아닙니다.

API 목적에 따라 필요한 정보만 반환합니다.

예:

```text
Reservation 목록
→ 여권번호 불필요

Reservation 상세
→ 필요 여부 검토

내부 Validation
→ 전체 값 사용 가능
```

구체적인 Response Field는
`05-data-api-design.md`에서 정의합니다.

---

### 16.4 저장 방식

테스트용 여권번호의 Database 저장 방식은
Data Design 단계에서 확정합니다.

검토 대상:

- 평문 저장
- Encryption
- Masking
- API Response Masking

테스트 데이터라는 특성과
포트폴리오 프로젝트의 보안 설계 목적을 함께 고려하여 결정합니다.

---

## 17. Internationalization

### 17.1 지원 Locale

MVP에서 다음 Locale을 지원합니다.

- `ko`
- `ja`

두 Locale에서 주요 기능과 Domain Policy는 동일하게 적용합니다.

---

### 17.2 Backend 데이터

다음 값은 사용자 Locale과 분리하여
영문 Canonical Term으로 관리합니다.

- Domain Model
- Entity
- Enum
- API Field
- Database 내부 값
- Identifier

사용자 표시 문자열을
Backend Business Logic에 직접 Hard Coding하지 않습니다.

---

### 17.3 Frontend 문자열

사용자에게 표시되는 문자열은
Frontend i18n 구조를 통해 관리합니다.

예:

```text
src/
 └─ locales/
      ├─ ko/
      └─ ja/
```

구체적인 i18n Library와 Directory 구조는
Frontend 초기 구성 시 확정합니다.

---

### 17.4 초기 Locale 선택

UI Design에서 정의한 다음 우선순위를 사용합니다.

```text
1. 사용자가 이전에 직접 선택한 Locale
2. Browser Locale
3. 기본 Locale ko
```

---

### 17.5 Locale 저장

사용자가 직접 선택한 Locale은
Frontend에서 유지하는 것을 기본 방향으로 합니다.

Guest는 Browser Storage 등을 이용한
Client-side 저장 방식을 사용할 수 있습니다.

Member도 MVP에서는
Frontend 저장 방식을 우선 사용할 수 있습니다.

향후 필요하면 Member Preference로
Server 저장 방식으로 확장할 수 있습니다.

Locale 정보는 인증 Token과 분리합니다.

---

## 18. Secret 및 Configuration 관리

### 18.1 환경설정 분리

환경별 설정은 Application Code와 분리합니다.

예:

- Database URL
- Database Username
- Database Password
- JWT Secret 또는 Signing Key
- Google Client ID
- Google Client Secret
- External Flight API Key
- AI API Key

---

### 18.2 Repository 금지

Secret 값은 Git Repository에 Commit하지 않습니다.

다음 파일 또는 값이 Repository에 포함되지 않도록 합니다.

- 실제 Password
- API Key
- Client Secret
- Private Key
- 운영 환경 Credential

필요한 환경변수 이름과 설정 예시는
실제 Secret을 제외한 형태로 문서화할 수 있습니다.

---

### 18.3 Local 환경

Local 개발환경에서는 다음 방식을 사용할 수 있습니다.

- Environment Variable
- Git에서 제외된 Local 설정파일
- IDE Run Configuration

`.gitignore`를 이용하여
실제 Secret이 포함된 Local 설정파일을 보호합니다.

---

### 18.4 운영 환경

운영 환경의 Secret은
AWS 배포 Architecture가 확정된 이후
적절한 Secret 관리 방식을 선택합니다.

Application Image 또는 Source Code 내부에
운영 Secret을 포함하지 않습니다.

---

## 19. Local 개발환경

### 19.1 기본 구조

Docker Compose를 이용하여
Local Infrastructure를 구성합니다.

MVP 초기 필수 Infrastructure는 다음과 같습니다.

```text
MySQL
```

Spring Boot Backend와 React Frontend는
개발 편의를 위해 Local Process로 실행할 수 있습니다.

---

### 19.2 기본 개발환경

예:

```text
Developer Mac

├─ React / Vite
│
├─ Spring Boot
│
└─ Docker Compose
     |
     └─ MySQL
```

Application 전체를 반드시 Container로 실행해야 하는 것은 아닙니다.

개발 생산성을 고려하여
Backend와 Frontend는 Local Process 실행을 허용합니다.

---

### 19.3 Redis

Redis는 초기 Local 개발환경의
필수 Container에 포함하지 않습니다.

다음과 같은 기술적 필요성이 확인된 경우에만 추가합니다.

- Cache
- Distributed Lock
- 기타 명확한 Redis 사용 사례

---

### 19.4 Kafka

Kafka 역시 초기 Local 개발환경에 포함하지 않습니다.

비동기 Event 처리 필요성이 명확하게 발생한 경우
도입 여부를 별도로 검토합니다.

---

## 20. CI

### 20.1 기본 방향

GitHub Actions를 이용하여
Pull Request 기반 CI를 구성합니다.

CI는 코드가 Merge되기 전에
Build와 Test 문제를 자동으로 검증합니다.

---

### 20.2 Backend CI

Backend에서는 최소 다음 항목을 검증합니다.

- Java Compile
- Unit Test
- Integration Test
- Gradle Build

예상 흐름:

```text
Pull Request

    ↓

Backend CI

    ↓

Compile

    ↓

Test

    ↓

Build
```

---

### 20.3 Frontend CI

Frontend에서는 최소 다음 항목을 검증합니다.

- Dependency Install
- Type Check
- Build
- Test가 구성된 경우 Test 실행

예상 흐름:

```text
Pull Request

    ↓

Frontend CI

    ↓

Install

    ↓

Type Check

    ↓

Test

    ↓

Build
```

---

### 20.4 CI 실패

CI가 실패한 Pull Request는
정상 Merge 대상으로 판단하지 않습니다.

실패 원인을 수정한 뒤
CI를 다시 통과해야 합니다.

Test를 삭제하거나 비활성화하여
CI를 강제로 통과시키지 않습니다.

---

### 20.5 Human Gate

CI 통과만으로 Pull Request를 자동 Merge하지 않습니다.

프로젝트의 개발 Workflow는 다음 구조를 유지합니다.

```text
Issue
  ↓
Implementation
  ↓
Pull Request
  ↓
CI
  ↓
AI Review
  ↓
수정
  ↓
Human Review
  ↓
Merge
```

최종 Merge 결정은 Human이 수행합니다.

---

## 21. CD 및 AWS 배포

### 21.1 기본 방향

MVP 최종 단계에서 AWS 기반 배포 환경을 구성합니다.

초기 배포 구조는 다음과 같은 형태를 기본 방향으로 합니다.

```text
User Browser
     |
     v
Frontend Hosting
     |
     v
Spring Boot Backend
     |
     v
MySQL
```

구체적인 AWS 서비스 조합은
구현 상태, 비용 및 운영 복잡도를 검토한 뒤 확정합니다.

검토 가능한 서비스 예:

- EC2
- RDS
- S3
- CloudFront

초기 System Design 단계에서
불필요하게 복잡한 Infrastructure를 선제적으로 확정하지 않습니다.

---

### 21.2 Backend 배포

Spring Boot Backend는
AWS Compute 환경에 배포합니다.

초기 검토 대상:

- EC2
- Docker 기반 Application 실행

Backend 배포 시 다음 사항을 고려합니다.

- 환경변수 주입
- Secret 분리
- Database 연결
- HTTPS 적용
- Application Log
- Health Check

구체적인 배포 방식은 M5에서 확정합니다.

---

### 21.3 Frontend 배포

React Frontend는
정적 Build 결과물을 기반으로 배포합니다.

검토 대상:

- S3
- CloudFront

Frontend Build에는
Backend API Endpoint 등 환경별 설정을 분리하여 적용합니다.

Secret 값은 Frontend Build 결과물에 포함하지 않습니다.

---

### 21.4 Database

운영 Database는 MySQL을 사용합니다.

AWS 배포 환경에서는 RDS 사용을 우선 검토합니다.

Database는 외부에 불필요하게 공개하지 않습니다.

Application에서 필요한 Network 경로만 허용하는 방향으로 구성합니다.

구체적인 Network 및 Security Group 구성은
AWS Architecture 확정 시 정의합니다.

---

### 21.5 CD

CD는 배포 환경이 확정된 이후
GitHub Actions에 추가합니다.

기본 흐름 예:

```text
Merge
  |
  v
GitHub Actions
  |
  v
Build
  |
  v
Deployment
  |
  v
AWS
```

초기 M1에서는 CI 구축을 우선하며,
CD는 실제 AWS 배포 단계에서 추가합니다.

배포 실패 시 기존 정상 서비스 상태를
가능한 한 유지할 수 있도록 구성합니다.

구체적인 배포 전략은 M5에서 확정합니다.

---

## 22. 성능 설계

### 22.1 기본 원칙

초기 구현에서는
성능 최적화보다 정확성과 데이터 정합성을 우선합니다.

성능 문제를 추측하여
Infrastructure나 복잡한 Architecture를 먼저 추가하지 않습니다.

성능 개선은 다음 순서로 진행합니다.

```text
측정
  |
  v
Bottleneck 확인
  |
  v
원인 분석
  |
  v
Query / Index 개선
  |
  v
Cache 검토
  |
  v
Architecture 변경 검토
```

---

### 22.2 Database 성능

Database 관련 성능 문제는
우선 다음 항목을 확인합니다.

- 불필요한 Query
- N+1 문제
- 과도한 Join
- 잘못된 Fetch 전략
- Index 부족
- 불필요한 Full Scan
- Pagination 문제

Query와 Index 설계의 구체적인 내용은
`05-data-api-design.md`에서 정의합니다.

---

### 22.3 Cache

Cache는 성능 문제가 실제로 확인된 이후 검토합니다.

검토 대상:

- 반복적인 조회
- 변경 빈도가 낮은 데이터
- External Flight API 반복 호출
- API Rate Limit 완화

Cache 도입 전 다음 사항을 확인합니다.

- Cache가 실제 Bottleneck을 해결하는지
- 데이터 최신성 문제가 발생하지 않는지
- Invalid / Expiration 복잡도가 과도하지 않은지

Redis는 Cache 필요성이 확인된 경우에만 도입합니다.

---

### 22.4 Redis Distributed Lock

Seat 동시성 문제 해결을 위해
Redis Distributed Lock을 기본 전제로 사용하지 않습니다.

우선 Database 기반 동시성 제어 방식을 검증합니다.

Redis Lock은 다음 조건이 실제로 필요한 경우 검토합니다.

- 다중 Application Instance
- Database Lock만으로 요구사항 충족이 어려움
- 동시성 테스트에서 명확한 필요성이 확인됨

기술 도입 전후 결과를 비교합니다.

---

### 22.5 Kafka

Kafka는 MVP의 기본 Architecture에 포함하지 않습니다.

다음과 같은 요구가 실제로 발생한 경우 검토합니다.

- 비동기 Event 처리
- 느슨한 서비스 결합
- 높은 처리량의 Event 전달
- Transaction 이후 후속 작업 분리

단순한 기술 사용 경험을 위해
Kafka를 추가하지 않습니다.

---

### 22.6 성능 측정

성능 개선이 필요한 기능은
변경 전과 변경 후 결과를 비교합니다.

측정 대상 예:

- Response Time
- Throughput
- Database Query 횟수
- Query 실행 시간
- 동시 요청 성공률
- External API 호출 횟수

필요한 경우 k6 또는 유사한 부하 테스트 도구를 사용합니다.

---

## 23. Observability

### 23.1 기본 방향

MVP에서는 복잡한 Observability Platform보다
문제 원인을 추적할 수 있는 최소한의 가시성을 우선합니다.

최소 다음 항목을 확인할 수 있어야 합니다.

- Application Error
- External Flight API 장애
- AI 요청 실패
- Scheduler 실패
- Reservation 처리 실패
- Mock Payment 처리 실패
- 인증 관련 오류

---

### 23.2 Application Log

Application Log는
장애 분석에 필요한 Context를 제공해야 합니다.

가능한 경우 다음 정보를 활용할 수 있습니다.

- Request 식별자
- 요청 시각
- 처리 결과
- Error Code
- 주요 Business Operation 종류

개인정보와 Secret은 Logging하지 않습니다.

---

### 23.3 Request 식별

하나의 Request를 Log에서 추적할 수 있도록
Request ID 또는 Correlation ID 사용을 검토합니다.

예:

```text
Request
   |
   v
Request ID 생성
   |
   v
Controller
   |
   v
Service
   |
   v
External API
```

동일 Request의 Log를 연결하여 확인할 수 있도록 하는 것이 목적입니다.

구체적인 구현 여부는
운영 및 Logging 구조 확정 시 결정합니다.

---

### 23.4 Scheduler Monitoring

다음 Scheduled Job의 실패 여부를
Log에서 확인할 수 있어야 합니다.

예:

- Seat Hold 만료 처리
- Flight `DEPARTED` 상태 전환

Scheduler 실패가 발생해도
Business API에서 방어적 검증을 수행하여
잘못된 상태 변경을 최소화합니다.

---

### 23.5 Monitoring Tool

추가 Monitoring Tool은
AWS 배포 및 성능 테스트 단계에서 검토합니다.

초기 MVP에서는
도입 자체보다 실제 필요성과 학습 비용을 고려합니다.

---

## 24. 외부 시스템 장애 격리

### 24.1 기본 원칙

외부 시스템 장애는
가능한 한 해당 기능에만 영향을 주도록 설계합니다.

외부 의존성:

- Google OAuth
- External Flight API
- AI Model API

KOKU Airline 내부 Reservation 기능은
외부 시스템과 불필요하게 결합하지 않습니다.

---

### 24.2 Google OAuth 장애

Google OAuth 장애 시:

```text
Google OAuth 로그인
→ 사용 불가 또는 실패

LOCAL 로그인
→ 영향 없음

KOKU Airline 내부 기능
→ 이미 인증된 Member는 정상 사용 가능
```

Google OAuth 장애가
LOCAL 인증이나 내부 Reservation 데이터에 영향을 주지 않아야 합니다.

---

### 24.3 External Flight API 장애

External Flight API 장애 시:

```text
실제 항공편 검색
→ 실패 가능

AI 항공편 검색
→ 실제 Flight Data 조회 실패로 제한 가능

KOKU Flight 검색
→ 정상

Reservation
→ 정상

Seat
→ 정상

Mock Payment
→ 정상
```

외부 실제 항공편 조회와
내부 KOKU Airline Reservation을
별도의 시스템 경계로 유지합니다.

---

### 24.4 AI Model API 장애

AI Model API 장애 시:

```text
AI 추천 설명
→ 실패 가능

일반 실제 항공편 검색
→ 정상 사용 가능

KOKU Airline 내부 Reservation
→ 영향 없음
```

AI API 장애로 인해
내부 Database Transaction이 Rollback되거나
업무 데이터가 변경되는 구조를 만들지 않습니다.

---

### 24.5 Transaction과 외부 호출

외부 API 호출을
내부 Reservation Database Transaction 안에서
불필요하게 수행하지 않습니다.

특히 다음 Transaction은
외부 Flight API 또는 AI API에 의존하지 않습니다.

- Seat 확보
- Reservation 생성
- Mock Payment
- Reservation 취소
- Seat 반환

내부 Transaction과 외부 시스템 호출의 경계를 명확하게 유지합니다.

---

## 25. AI Agent 구현 제한

### 25.1 기본 원칙

AI Agent는
본 문서와 상위 설계 문서에 정의된 Architecture를
임의로 변경하지 않습니다.

AI Agent는 Issue와 Acceptance Criteria에 정의된 범위 안에서만 작업합니다.

---

### 25.2 임의 변경 금지 영역

AI Agent는 다음 항목을
Human 승인 없이 변경하지 않습니다.

- System Architecture
- Domain Policy
- 사용자 Role
- 인증 및 인가 구조
- JWT / Token 정책
- Transaction Boundary
- Seat 동시성 전략
- Seat Hold 정책
- External API 경계
- 내부 Flight와 외부 Flight의 데이터 경계
- AI 역할 범위
- Cache 도입
- Redis 도입
- Kafka 도입
- 새로운 Infrastructure 추가
- Secret 관리 방식
- Database 구조
- API Contract
- AWS Architecture

---

### 25.3 변경 필요 시 처리

구현 과정에서 설계 변경이 필요하다고 판단되면
AI Agent는 임의로 변경하지 않습니다.

기본 흐름:

```text
구현 중 설계 문제 발견
        |
        v
작업 중단
        |
        v
문제 및 변경 필요성 보고
        |
        v
Human 검토
        |
        +-- 승인 → 문서 수정 후 구현
        |
        +-- 거부 → 기존 설계 유지
```

관련 문서 수정 없이
코드만 새로운 Architecture로 변경하지 않습니다.

---

### 25.4 Implementer / Reviewer 분리

동일한 AI Agent가
구현과 최종 Review를 모두 수행하지 않는 것을 기본으로 합니다.

Reviewer는 다음 사항을 확인합니다.

- Issue 범위 준수
- Domain Policy 준수
- System Design 준수
- API Contract 준수
- Test 존재 여부
- 불필요한 기술 도입 여부
- Secret 포함 여부
- 관련 없는 파일 변경 여부

최종 Merge는 Human이 결정합니다.

---

## 26. System Design에서 아직 확정할 항목

본 문서의 기본 Architecture는 정의되었지만
다음 세부 항목은 구현 전에 추가로 확정합니다.

### 26.1 인증 및 Token

- [ ] Access Token 만료시간
- [ ] Refresh Token 만료시간
- [ ] Refresh Token 저장 위치
- [ ] Refresh Token Rotation 여부
- [ ] Token 재사용 탐지 여부
- [ ] Access Token Frontend 저장 방식
- [ ] Refresh Token Cookie 정책
- [ ] CSRF 대응 방식
- [ ] 로그아웃 Token 무효화 방식

---

### 26.2 날짜 및 시간

확정:

- [x] Flight 절대 시각은 UTC 기준으로 저장
- [x] Backend 절대 시각 계산은 `Instant` 사용
- [x] MySQL 절대 시각 Column은 `DATETIME(6)` 사용
- [x] `DATETIME(6)`에 저장되는 절대 시각은 UTC 기준
- [x] 날짜만 의미하는 값은 Java `LocalDate` / MySQL `DATE` 사용
- [x] Airport Time Zone은 IANA `ZoneId` 기준으로 관리
- [x] Airport Time Zone DB Column은 IANA ID 문자열로 저장
- [x] JDBC / Hibernate Database Time Zone은 UTC 기준으로 구성
- [x] 사용자 표시 시각은 Airport 현지시간 기준 24시간제 사용
- [x] API Date / Time은 ISO-8601 Offset 포함 형식 사용
- [x] Backend 현재 시각 Source는 주입 가능한 `Clock` 사용
- [x] Flight `DEPARTED` Scheduled Job은 1분 주기
- [x] Seat Hold 만료 Scheduled Job은 1분 주기
- [x] Scheduler와 함께 Business API에서 방어적 시간 검증 수행

---

### 26.3 Reservation / Seat

- [ ] Seat 동시성 제어 최종 구현 방식
- [x] Hold 만료 처리: 1분 Scheduled Job + Business API 방어적 만료 검증
- [ ] Reservation Transaction 내부 Persist / Flush 순서
- [ ] 동시성 실패 시 Error 처리 방식

---

### 26.4 Passenger 정보 보호

- [ ] 테스트용 여권번호 Database 저장 방식
- [ ] Encryption 적용 여부
- [ ] API Response Masking 여부
- [ ] 관리자 화면 Masking 여부

---

### 26.5 External Flight API

- [ ] 실제 사용할 External Flight API
- [ ] Connection Timeout
- [ ] Read Timeout
- [ ] Retry 정책
- [ ] Retry 횟수
- [ ] Backoff 방식
- [ ] Cache 적용 여부
- [ ] Cache TTL

---

### 26.6 AI

- [ ] 사용할 AI Model / Provider
- [ ] Structured Output 구체 구조
- [ ] Spring AI Tool Contract
- [ ] AI에 전달할 최대 Flight 후보 수
- [ ] AI Timeout
- [ ] AI Retry 여부

---

### 26.7 Infrastructure

- [ ] AWS 세부 Architecture
- [ ] Backend 배포 방식
- [ ] Frontend 배포 방식
- [ ] RDS 구성
- [ ] 운영 환경 Secret 관리 방식
- [ ] Monitoring 방식
- [ ] CD Workflow

---

## 27. System Design 완료 기준

다음 조건을 모두 만족하면
MVP System Design이 완료된 것으로 판단합니다.

### Architecture

- [ ] Frontend / Backend 기본 Architecture가 정의되어 있습니다.
- [ ] Backend Layer 역할이 정의되어 있습니다.
- [ ] 내부 시스템과 외부 시스템의 경계가 정의되어 있습니다.

### Authentication / Authorization

- [ ] LOCAL 인증 흐름이 정의되어 있습니다.
- [ ] Google OAuth 흐름이 정의되어 있습니다.
- [ ] Member / AuthAccount 책임이 구분되어 있습니다.
- [ ] Spring Security 권한 구조가 정의되어 있습니다.
- [ ] JWT 및 Token 정책이 최종 확정되어 있습니다.

### Transaction / Concurrency

- [ ] Reservation 시작 Transaction이 정의되어 있습니다.
- [ ] Mock Payment Transaction이 정의되어 있습니다.
- [ ] Reservation 취소 Transaction이 정의되어 있습니다.
- [ ] Seat Hold 만료 Transaction이 정의되어 있습니다.
- [ ] Seat 동시성 처리 방향이 정의되어 있습니다.
- [ ] 동시성 테스트 방향이 정의되어 있습니다.

### Time

- [ ] Backend 시간 기준이 정의되어 있습니다.
- [ ] Time Zone 처리 원칙이 정의되어 있습니다.
- [ ] Database 시간 저장 기준이 확정되어 있습니다.
- [ ] Flight `DEPARTED` 처리 방식이 정의되어 있습니다.
- [ ] Seat Hold 만료 처리 방식이 정의되어 있습니다.

### External / AI

- [ ] External Flight API 호출 구조가 정의되어 있습니다.
- [ ] 지원 Route 사전 검증 방식이 정의되어 있습니다.
- [ ] External API 장애 격리 원칙이 정의되어 있습니다.
- [ ] Cache 도입 원칙이 정의되어 있습니다.
- [ ] AI 검색 Architecture가 정의되어 있습니다.
- [ ] AI와 Application의 역할이 명확하게 분리되어 있습니다.
- [ ] AI Hallucination 방지 원칙이 정의되어 있습니다.

### Security / Logging

- [ ] Secret 관리 원칙이 정의되어 있습니다.
- [ ] Logging 금지 정보가 정의되어 있습니다.
- [ ] 테스트용 Passenger 정보 보호 원칙이 정의되어 있습니다.
- [ ] 관리자 Audit 대상이 정의되어 있습니다.

### Frontend / i18n

- [ ] Backend와 Frontend의 책임이 구분되어 있습니다.
- [ ] `ko` / `ja` Locale 관리 원칙이 정의되어 있습니다.
- [ ] Locale 저장 방향이 정의되어 있습니다.

### Infrastructure / Delivery

- [ ] Local Docker 환경이 정의되어 있습니다.
- [ ] CI 기본 구조가 정의되어 있습니다.
- [ ] Human Gate가 유지됩니다.
- [ ] AWS 배포 기본 방향이 정의되어 있습니다.
- [ ] CD 도입 시점이 정의되어 있습니다.

### AI Development Workflow

- [ ] AI Agent가 임의로 변경할 수 없는 기술 영역이 정의되어 있습니다.
- [ ] Architecture 변경 시 Human 승인 절차가 정의되어 있습니다.
- [ ] Implementer / Reviewer 역할이 분리되어 있습니다.

---

## 28. System Design 변경 원칙

본 문서는 Backend와 Frontend 구현의
기술적 기준으로 사용합니다.

구현 중 본 문서와 다른 Architecture가 필요하다고 판단되면
코드를 먼저 변경하지 않습니다.

다음 순서로 처리합니다.

```text
문제 발견
  |
  v
변경 필요성 분석
  |
  v
Human Review
  |
  v
관련 설계 문서 수정
  |
  v
Issue / Acceptance Criteria 수정
  |
  v
구현
```

다음 변경은 반드시 관련 문서의 정합성을 함께 검토합니다.

- 인증 및 인가 방식 변경
- JWT / Token 정책 변경
- Transaction Boundary 변경
- 동시성 전략 변경
- Database 구조 변경
- External API 경계 변경
- AI 역할 변경
- 새로운 Infrastructure 도입
- Cache / Message Broker 도입
- AWS Architecture 변경

System Design 변경으로 인해
Domain Policy가 변경되어야 하는 경우
`02-domain-policy.md`를 먼저 검토합니다.

데이터 구조 또는 API가 변경되는 경우
`05-data-api-design.md`도 함께 수정합니다.

UI 흐름에 영향을 주는 경우
`03-ui-design.md`의 정합성도 함께 검토합니다.