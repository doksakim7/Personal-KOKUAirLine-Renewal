# KOKU Airline Renewal - Development Log

## 1. 문서 목적

본 문서는 KOKU Airline Renewal 프로젝트의
개발 진행 과정과 주요 기술적 의사결정 기록을 관리합니다.

본 문서의 목적은 다음과 같습니다.

- 개발 단계별 진행 상황을 기록합니다.
- 주요 Architecture 및 Data 설계 변경 이유를 기록합니다.
- 구현 중 발생한 문제와 해결 과정을 기록합니다.
- 성능 및 동시성 테스트 결과를 기록합니다.
- 외부 API 및 AI 연동 검증 결과를 기록합니다.
- 문서와 실제 구현의 차이를 추적합니다.
- AI Agent가 수행한 주요 작업과 Human Review 결과를 기록합니다.
- 향후 개선이 필요한 Technical Debt를 기록합니다.

본 문서는 새로운 Business Rule을 정의하지 않습니다.

비즈니스 규칙은 `02-domain-policy.md`,
시스템 구조는 `04-system-design.md`,
Data 및 API Contract는 `05-data-api-design.md`,
ERD는 `diagrams/erd.md`를 기준으로 합니다.

설계 변경이 필요한 경우
본 문서에만 변경 내용을 기록하고 끝내지 않습니다.

반드시 관련 설계 문서를 먼저 또는 함께 수정한 뒤
변경 이력을 본 문서에 기록합니다.

---

### 1.1 기록 원칙

Development Log는 다음 원칙을 따릅니다.

1. 완료된 작업과 예정 작업을 구분합니다.
2. 사실과 추정을 구분합니다.
3. 결정된 사항과 검토 중인 사항을 구분합니다.
4. 기술 도입 이유와 검증 결과를 함께 기록합니다.
5. 실패한 접근도 필요한 경우 기록합니다.
6. 성능 개선은 변경 전 / 후 결과를 함께 기록합니다.
7. Architecture 변경은 관련 문서 변경 여부를 기록합니다.
8. 단순 Commit History를 그대로 복사하지 않습니다.
9. 중요한 문제와 의사결정 중심으로 기록합니다.
10. 날짜 기준으로 개발 흐름을 추적할 수 있어야 합니다.

---

## 2. 프로젝트 진행 상태

### 2.1 전체 단계

현재 프로젝트는 다음 단계로 진행합니다.

```text
기획
  |
  v
Domain Policy
  |
  v
UI Design
  |
  v
System Design
  |
  v
Data & API Design
  |
  v
ERD
  |
  v
구현
  |
  v
Test
  |
  v
Performance / Concurrency
  |
  v
AWS Deployment
```

---

### 2.2 Milestone

Project Plan을 기준으로 다음 Milestone을 사용합니다.

#### M1 - 설계 및 초기 환경

주요 목표:

- Project Plan
- Domain Policy
- UI Design
- System Design
- Data & API Design
- ERD
- Spring Boot 초기 구성
- React 초기 구성
- Docker Compose
- GitHub Actions CI

---

#### M2 - 인증 및 기본 Domain

주요 목표:

- Member
- AuthAccount
- LOCAL 로그인
- Google OAuth
- Spring Security
- JWT
- 권한 관리
- i18n
- Master Data 기본 구조

---

#### M3 - Flight / Reservation / Payment

주요 목표:

- Airport
- Route
- Aircraft
- Flight
- Seat
- Passenger
- Reservation
- Mock Payment
- Reservation Number
- Seat Hold
- Reservation 취소
- External Flight API
- AI Flight Search

---

#### M4 - 동시성 및 성능

주요 목표:

- Seat 동시성 테스트
- Lock 전략 검증
- Query 분석
- Index 검증
- 성능 Bottleneck 분석
- 필요 시 Cache 검토
- 필요 시 Redis 검토
- 부하 테스트

---

#### M5 - 배포 및 마무리

주요 목표:

- AWS 배포
- CD
- 운영 환경 설정
- Secret 관리
- Monitoring
- 통합 테스트
- README 정리
- Architecture 문서 최종 정리

---

## 3. 현재 설계 문서 상태

### 3.1 Project Plan

```text
docs/01-project-plan.md
```

상태:

```text
작성 완료
추후 구현 결과에 따라 일부 수정 가능
```

---

### 3.2 Domain Policy

```text
docs/02-domain-policy.md
```

상태:

```text
핵심 Business Rule 작성 완료
일부 세부 기술 정책은 System / Data Design에 위임
```

---

### 3.3 UI Design

```text
docs/03-ui-design.md
```

상태:

```text
MVP 사용자 흐름 및 주요 화면 구조 작성 완료
```

---

### 3.4 System Design

```text
docs/04-system-design.md
```

상태:

```text
초안 작성 완료
일부 기술 세부사항 미확정
```

주요 미확정 항목:

- Access Token 만료시간
- Refresh Token 만료시간
- Refresh Token 저장 방식
- CSRF 대응
- 시간 저장 기준
- Seat 동시성 최종 방식
- Scheduler 주기
- External Flight API Provider
- Cache TTL
- AWS 세부 Architecture

---

### 3.5 Data & API Design

```text
docs/05-data-api-design.md
```

상태:

```text
초안 작성 완료
Entity 관계 및 API 기본 구조 정의
일부 Column / Constraint / DTO 미확정
```

주요 미확정 항목:

- Date / Time 실제 Database Type
- Flight Number Composite Unique
- Aircraft Schedule Conflict 기준
- Seat Lock 방식
- Reservation Mapping Entity 최종 형태
- Passport Encryption / Masking
- Payment SUCCESS Database 보호 방식
- API Error 상세 Mapping
- External Flight DTO
- AI Structured Output

---

### 3.6 ERD

```text
docs/diagrams/erd.md
```

상태:

```text
Draft 작성 완료
05-data-api-design.md 기준 시각화
```

ERD는 Data Design을 선행하지 않습니다.

`05-data-api-design.md` 변경 후
ERD를 함께 수정합니다.

---

## 4. 설계 단계 주요 결정 기록

### 4.1 내부 Flight와 외부 Flight 분리

#### 결정

KOKU Airline 내부 Flight와
외부 실제 항공편 데이터를 완전히 분리합니다.

```text
KOKU Flight
→ 내부 Database
→ Reservation 가능

External Flight
→ 외부 API
→ 조회 / AI 추천만 가능
```

#### 이유

외부 Flight API는 실제 항공편 검색용이며
KOKU Airline Reservation Transaction과 연결하지 않기 위해서입니다.

#### 영향 문서

- `02-domain-policy.md`
- `04-system-design.md`
- `05-data-api-design.md`

---

### 4.2 Reservation과 Flight 관계

#### 결정

Reservation에 단일 `flight_id`를 저장하지 않고
`ReservationFlight` Mapping 구조를 사용합니다.

```text
ONE_WAY
Reservation → Flight 1개

ROUND_TRIP
Reservation → Flight 2개
```

#### 이유

왕복 Reservation을 하나의 Reservation Number와
하나의 Payment 단위로 관리하기 위해서입니다.

#### 상태

```text
Mapping 구조는 채택
최종 JPA Entity 구현 방식은 미확정
```

---

### 4.3 PassengerFlight 도입

#### 결정

Passenger의 Flight별 속성을
`PassengerFlight` 구조로 분리합니다.

#### 이유

ROUND_TRIP에서 같은 Passenger라도
Flight 탑승일에 따라 다음 정보가 달라질 수 있기 때문입니다.

- Adult / Child / Infant 구분
- Seat 필요 여부
- Seat 배정
- Infant Companion

#### 상태

```text
Data Model 초안 채택
최종 JPA Mapping은 구현 전 재검토
```

---

### 4.4 PaymentAttempt Entity 제거

#### 결정

별도의 `PaymentAttempt` Entity를 사용하지 않습니다.

각 `Payment` Row 자체를
한 번의 Mock Payment 시도로 사용합니다.

```text
Reservation 1 : N Payment
```

#### 이유

MVP에서는 Payment 자체가 결제 시도 이력을 충분히 표현할 수 있기 때문입니다.

---

### 4.5 Seat 동시성 기술 미확정

#### 결정

초기 설계에서 특정 Lock 방식으로 고정하지 않습니다.

검토 대상:

- Database Unique Constraint
- Conditional Update
- Pessimistic Lock
- Optimistic Lock
- Redis Distributed Lock

#### 이유

단일 Backend / 단일 Database 환경에서
Redis Distributed Lock이 실제로 필요한지 먼저 검증하기 위해서입니다.

#### 결정 시점

```text
M4 동시성 테스트 이후
```

---

### 4.6 Cache 선제 도입 금지

#### 결정

Redis Cache를 초기 Architecture에 포함하지 않습니다.

#### 도입 검토 조건

- External API Rate Limit
- 반복 호출 비용
- 높은 Latency
- 반복 조회 Bottleneck
- 실제 성능 테스트에서 필요성 확인

---

### 4.7 Kafka 선제 도입 금지

#### 결정

Kafka를 MVP 기본 Architecture에 포함하지 않습니다.

#### 이유

현재 Domain에서 Message Broker가 반드시 필요한
Business Requirement가 존재하지 않기 때문입니다.

실제 비동기 Event 분리가 필요해진 경우에만 검토합니다.

---

## 5. 인증 설계 기록

### 5.1 인증 방식

MVP 인증:

```text
LOCAL
GOOGLE
```

서비스 사용자는 `Member`,
인증 수단은 `AuthAccount`에서 관리합니다.

---

### 5.2 Service Role과 Security Authority

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

두 개념을 혼용하지 않습니다.

---

### 5.3 Email 정규화

다음 인증 흐름에서 동일한 Email 정규화 규칙을 사용합니다.

- 회원가입
- LOCAL 로그인
- Google OAuth 계정 연결

기준:

```text
trim()
+
lowercase(Locale.ROOT)
```

---

### 5.4 Google OAuth 계정 연결

동일 Email의 LOCAL Member가 존재하더라도
Google Email 일치만으로 자동 연결하지 않습니다.

```text
Google OAuth
    |
    v
동일 Email LOCAL Member
    |
    v
LOCAL Password 재인증
    |
    +-- 성공 → GOOGLE AuthAccount 연결
    |
    +-- 실패 → 연결 거부
```

---

### 5.5 JWT

현재 기본 방향:

```text
Access Token
+
Refresh Token
```

미확정:

- Access Token TTL
- Refresh Token TTL
- Refresh Token 저장 위치
- Rotation
- Reuse Detection
- Cookie Attribute
- CSRF
- Logout 무효화 방식

해당 결정은 구현 전에
`04-system-design.md`를 먼저 수정한 뒤 반영합니다.

---

## 6. Reservation 설계 기록

### 6.1 Reservation Number

사용자 공개 Reservation Number:

```text
KOKU-YYYYMMDD-XXXXXX
```

Random 문자에서 제외:

```text
I
O
0
1
```

Database PK와 사용자 공개 Reservation Number를 분리합니다.

Database Constraint:

```text
UNIQUE(reservation_no)
```

---

### 6.2 PENDING 생성

PENDING Reservation 생성은
Seat 확보와 함께 하나의 Transaction으로 처리합니다.

```text
조건 검증
    |
    v
Seat 확보 가능 여부 검증
    |
    v
PENDING Reservation 생성
    |
    v
Reservation Number 생성
    |
    v
최종 금액 확정
    |
    v
Seat AVAILABLE → HELD
    |
    v
Commit
```

구체적인 Persist / Flush 순서는
실제 JPA Mapping 이후 확정합니다.

---

### 6.3 Seat Hold

기본 Hold 시간:

```text
1시간
```

Backend 시간이 최종 기준입니다.

Frontend Countdown은
사용자 안내 목적으로만 사용합니다.

---

### 6.4 Payment 성공

Mock Payment 성공 시:

```text
Payment
PENDING → SUCCESS

Reservation
PENDING → CONFIRMED

Seat
HELD → RESERVED
```

하나의 Business Transaction으로 처리합니다.

---

### 6.5 Payment 실패

하나의 Reservation당
최대 3회의 Payment를 허용합니다.

```text
1차 실패
2차 실패
3차 실패
    |
    v
Reservation CANCELLED
Seat RELEASE
```

기존 FAILED Payment는 삭제하지 않습니다.

---

## 7. External Flight API 기록

### 7.1 목적

External Flight API는 다음 기능에만 사용합니다.

- 실제 항공편 조회
- AI 항공편 검색
- AI 추천 근거 Data

---

### 7.2 내부 Transaction과 분리

외부 API 호출은 다음 Transaction에 포함하지 않습니다.

- Reservation 생성
- Seat 확보
- Payment
- Reservation 취소
- Seat 반환

---

### 7.3 Provider

현재 상태:

```text
미확정
```

Provider 결정 시 기록할 항목:

- Provider 이름
- 무료 / 유료 정책
- Rate Limit
- 응답 Data
- 한국 / 일본 지원 범위
- 가격 Data 제공 여부
- Timeout
- Retry
- Cache 필요 여부

---

## 8. AI 기능 기록

### 8.1 AI 역할

AI는 다음 역할만 수행합니다.

- 자연어 해석
- 검색 조건 구조화
- 추천 설명
- 사용자 친화적 요약

---

### 8.2 AI가 수행하지 않는 작업

AI는 다음 작업을 수행하지 않습니다.

- Reservation 생성
- Seat 확보
- Payment
- Reservation 취소
- 외부 실제 항공편 예약
- 실제 결제
- 실제 발권

---

### 8.3 AI Search Flow

```text
Natural Language
      |
      v
Structured Output
      |
      v
Application Validation
      |
      v
Route Validation
      |
      v
External Flight API
      |
      v
Java Filtering
      |
      v
Java Ranking
      |
      v
AI Explanation
```

---

### 8.4 Hallucination 방지

다음 정보는 External Flight API를 Source of Truth로 사용합니다.

- Airline
- Flight Number
- Price
- Departure Time
- Arrival Time
- Stops

AI가 값을 생성하거나 변경하지 않습니다.

---

## 9. 구현 진행 기록 Template

실제 개발 시작 이후
각 주요 작업은 다음 형식으로 기록합니다.

---

### YYYY-MM-DD - 작업 제목

#### 작업 목적

```text
이번 작업에서 해결하려는 문제 또는 구현 목표
```

#### 관련 Issue

```text
#IssueNumber
```

#### 관련 문서

- `02-domain-policy.md`
- `04-system-design.md`
- `05-data-api-design.md`

필요한 문서만 기록합니다.

#### 구현 내용

- 구현 항목 1
- 구현 항목 2
- 구현 항목 3

#### 주요 결정

```text
결정 내용
```

#### 결정 이유

```text
왜 이 방식을 선택했는지
```

#### 검토한 대안

```text
Alternative A
Alternative B
Alternative C
```

#### Test

```text
수행한 Test
결과
```

#### 결과

```text
SUCCESS
PARTIAL
FAILED
```

#### 후속 작업

- [ ] 후속 작업 1
- [ ] 후속 작업 2

#### 관련 Commit / PR

```text
Commit:
PR:
```

---

## 10. Bug / 문제 해결 기록 Template

### YYYY-MM-DD - 문제 제목

#### 증상

```text
발생한 문제
```

#### 재현 조건

```text
문제 재현 방법
```

#### 원인

```text
Root Cause
```

#### 해결

```text
적용한 해결 방법
```

#### 검토한 대안

```text
대안 및 제외 이유
```

#### Test

```text
회귀 Test 포함
```

#### 영향 범위

```text
Domain
API
Database
Frontend
Infrastructure
```

#### 관련 문서 수정

```text
없음
또는
수정한 문서
```

---

## 11. Architecture Decision 기록 Template

중요한 Architecture 결정은
다음 형식으로 기록합니다.

### ADR-XXX - 결정 제목

#### 상태

```text
PROPOSED
ACCEPTED
SUPERSEDED
REJECTED
```

#### Context

```text
해결해야 하는 Architecture 문제
```

#### Decision

```text
최종 결정
```

#### Alternatives

```text
대안
```

#### Reason

```text
선택 이유
```

#### Consequences

##### 장점

- 장점

##### 단점

- 단점

#### 관련 문서

- `04-system-design.md`
- `05-data-api-design.md`

필요한 문서만 기록합니다.

---

## 12. 동시성 Test 기록 Template

### YYYY-MM-DD - Seat Concurrency Test

#### 목적

동일 Flight / 동일 Seat에 대한
동시 Reservation 요청을 검증합니다.

#### Test 조건

```text
Concurrent Requests:
Flight:
Seat:
Backend Instance:
Database:
```

#### Test Scenario

```text
N개의 요청
      |
      v
동일 Flight / 동일 Seat
      |
      v
Reservation 생성 시도
```

#### 기대 결과

```text
성공 = 1
실패 = N - 1
```

#### 적용 방식

```text
No Lock
Pessimistic Lock
Optimistic Lock
Conditional Update
Redis Lock
```

#### 결과

```text
Success Count:
Failure Count:
Average Response Time:
Max Response Time:
```

#### 정합성 확인

- [ ] 중복 Reservation 없음
- [ ] 중복 RESERVED Seat 없음
- [ ] 실패 Transaction Rollback
- [ ] ROUND_TRIP Partial Hold 없음

#### 결론

```text
해당 방식 유지 / 변경 여부
```

---

## 13. 성능 Test 기록 Template

### YYYY-MM-DD - Performance Test

#### 대상 API

```text
Endpoint
```

#### Test 환경

```text
Backend:
Database:
Instance:
Dataset Size:
Tool:
```

#### 변경 전

```text
Average Response:
P95:
P99:
Throughput:
Error Rate:
Query Count:
```

#### 문제

```text
Bottleneck
```

#### 개선

```text
Query 변경
Index 추가
Fetch 전략 수정
Cache
기타
```

#### 변경 후

```text
Average Response:
P95:
P99:
Throughput:
Error Rate:
Query Count:
```

#### 결과

```text
Improvement:
```

#### 결론

```text
적용 / 롤백 / 추가 검토
```

---

## 14. External API 검증 기록 Template

### YYYY-MM-DD - External Flight API 검증

#### Provider

```text
Provider Name
```

#### 확인 항목

- [ ] 한국 Airport 지원
- [ ] 일본 Airport 지원
- [ ] KR → JP 조회
- [ ] JP → KR 조회
- [ ] Flight Number 제공
- [ ] Airline 제공
- [ ] 출발 / 도착 시각 제공
- [ ] 가격 제공
- [ ] 직항 / 경유 정보 제공
- [ ] Rate Limit 확인
- [ ] Timeout 확인

#### Response 예시

```json
{
}
```

#### 문제

```text
Provider Data의 제한
```

#### Adapter Mapping

```text
Provider Field
→
KOKU ExternalFlight DTO
```

#### 결론

```text
채택 / 보류 / 제외
```

---

## 15. AI 기능 검증 기록 Template

### YYYY-MM-DD - AI Flight Search Test

#### 사용자 입력

```text
자연어 Query
```

#### Structured Output

```json
{
}
```

#### Validation 결과

```text
PASS
FAIL
```

#### External Flight API 결과

```text
Candidate Count:
```

#### Java Filtering 결과

```text
Filtered Count:
```

#### Java Ranking 결과

```text
Top Candidate:
```

#### AI 설명 결과

```text
추천 설명
```

#### Hallucination 검증

- [ ] Flight Number 변경 없음
- [ ] Airline 변경 없음
- [ ] Price 변경 없음
- [ ] Departure Time 변경 없음
- [ ] Arrival Time 변경 없음

#### 결론

```text
PASS / FAIL
```

---

## 16. Security 검토 기록 Template

### YYYY-MM-DD - Security Review

#### 대상

```text
Authentication
Authorization
JWT
OAuth
Secret
Passenger Data
```

#### 확인 항목

- [ ] Password 평문 저장 없음
- [ ] Password Hash API 노출 없음
- [ ] Token Log 출력 없음
- [ ] Secret Repository 포함 없음
- [ ] Test Passport Log 출력 없음
- [ ] 권한 Backend 검증
- [ ] 다른 Member Reservation 접근 차단

#### 발견 문제

```text
문제
```

#### 수정

```text
수정 내용
```

---

## 17. AI Agent 작업 기록

AI Agent를 사용하여 구현 또는 Review를 수행한 경우
중요한 변경은 필요에 따라 기록합니다.

### YYYY-MM-DD - AI Agent 작업

#### Agent 역할

```text
Implementer
Reviewer
```

#### 작업 범위

```text
Issue
```

#### 변경 파일

```text
파일 목록
```

#### 설계 변경 여부

```text
NO
YES
```

`YES`인 경우
Human 승인과 관련 문서 수정 여부를 반드시 기록합니다.

#### Review 결과

```text
PASS
CHANGES_REQUESTED
```

#### Human Decision

```text
MERGED
REJECTED
REWORK
```

---

## 18. Technical Debt

현재 또는 구현 과정에서 발견된
Technical Debt를 관리합니다.

| ID | 영역 | 내용 | 우선순위 | 상태 |
| --- | --- | --- | --- | --- |
| TD-001 | Auth | JWT 세부 정책 확정 필요 | High | Open |
| TD-002 | Time | Database Time 저장 기준 확정 필요 | High | Open |
| TD-003 | Seat | 동시성 제어 방식 검증 필요 | High | Open |
| TD-004 | Passenger | Test Passport 보호 방식 확정 필요 | Medium | Open |
| TD-005 | External | Flight API Provider 확정 필요 | High | Open |
| TD-006 | AI | Structured Output Contract 확정 필요 | Medium | Open |
| TD-007 | Infra | AWS 세부 Architecture 확정 필요 | Medium | Open |

Technical Debt가 해결되면
관련 설계 문서를 수정한 뒤 상태를 변경합니다.

예:

```text
Open
→
Resolved
```

---

## 19. 미확정 항목 추적

본 절은
`04-system-design.md`와 `05-data-api-design.md`의
미확정 항목을 추적하기 위한 요약입니다.

본 절에서 정책을 직접 확정하지 않습니다.

### Authentication

- [ ] Access Token TTL
- [ ] Refresh Token TTL
- [ ] Refresh Token 저장
- [ ] Rotation
- [ ] CSRF
- [ ] Logout 무효화

### Time

- [ ] Database Time Type
- [ ] UTC / Zone 저장 기준
- [ ] Backend Clock 적용
- [ ] DEPARTED Scheduler
- [ ] Hold Scheduler

### Flight

- [ ] Flight Number Unique 기준
- [ ] Aircraft Conflict 기준
- [ ] Turnaround Time

### Seat

- [ ] Flight Seat 생성 시점
- [ ] Lock 방식
- [ ] held_reservation_id 필요 여부

### Passenger

- [ ] Gender Enum
- [ ] Nationality 저장
- [ ] Passport 형식
- [ ] Encryption
- [ ] Masking

### Payment

- [ ] SUCCESS Unique 보호
- [ ] attempt_no Constraint
- [ ] Idempotency 상세 정책

### External

- [ ] Flight API Provider
- [ ] Timeout
- [ ] Retry
- [ ] Cache
- [ ] TTL

### AI

- [ ] Provider / Model
- [ ] Structured Output
- [ ] Tool Contract
- [ ] Candidate Limit
- [ ] Timeout

### Infrastructure

- [ ] AWS Architecture
- [ ] CD
- [ ] Secret Management
- [ ] Monitoring

---

## 20. 개발 완료 시 기록 항목

MVP 개발 완료 시
최종적으로 다음 내용을 정리합니다.

### Architecture

```text
최종 Architecture
초기 설계와 달라진 부분
변경 이유
```

### Database

```text
최종 ERD
주요 Constraint
Index
```

### Concurrency

```text
최종 Seat Lock 방식
Test 결과
선택 이유
```

### Performance

```text
주요 성능 개선
변경 전 / 후 결과
```

### External API

```text
최종 Provider
제약사항
Cache 여부
```

### AI

```text
최종 Model
Structured Output
추천 방식
Hallucination 방지 방식
```

### Deployment

```text
AWS Architecture
CI/CD
Monitoring
```

### Technical Debt

```text
남아 있는 개선사항
```

---

## 21. Development Log 변경 원칙

Development Log는
프로젝트 진행의 기록 문서입니다.

설계 기준 문서보다 우선하지 않습니다.

문서 우선순위:

```text
Domain Policy
    |
    v
System Design
    |
    v
Data & API Design
    |
    v
ERD
    |
    v
Development Log
```

Development Log에 기록된 내용과
설계 문서가 충돌하는 경우
설계 문서를 기준으로 합니다.

구현 과정에서 기존 설계가 변경된 경우:

```text
문제 발견
    |
    v
Human Review
    |
    v
관련 설계 문서 수정
    |
    v
구현 수정
    |
    v
Development Log 기록
```

순서를 유지합니다.

Development Log만 수정하여
Architecture 또는 Business Rule을 변경하지 않습니다.