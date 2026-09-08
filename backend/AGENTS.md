# KOKU Airline Renewal - Backend Agent Rules

## 1. Purpose

본 문서는 `backend/` 영역에서 작업하는 AI Agent가 따라야 하는
Backend 전용 추가 규칙을 정의합니다.

본 문서는 루트 `AGENTS.md`를 대체하지 않습니다.

Backend 작업에는 다음 규칙이 함께 적용됩니다.

```text
루트 AGENTS.md
→ Repository 공통 정책 및 안전 규칙

backend/AGENTS.md
→ Backend 구현 추가 규칙
```

루트 규칙과 함께 적용하며,
충돌 시 루트 `AGENTS.md`에 정의된 Rule Conflict 처리 방식을 따릅니다.

---

## 2. Scope

본 규칙은 원칙적으로 다음 영역에 적용됩니다.

```text
backend/
```

주요 대상:

- Java / Spring Boot 코드
- Controller
- Service
- Repository
- Entity
- DTO
- Validation
- Security
- Transaction
- Persistence
- Backend Test
- Backend Configuration

Backend 작업 중 다른 영역의 수정이 필요해진 경우
현재 Issue의 수정 허용 범위를 확인합니다.

허용 범위 밖 변경이 필요하면
임의로 수정하지 않고 STOP CONDITION으로 처리합니다.

---

## 3. Backend Source of Truth

Backend 작업 전
현재 Issue와 관련된 Source of Truth를 확인합니다.

특히 다음 문서를 우선 확인합니다.

```text
docs/02-domain-policy.md
→ Business Rule

docs/04-system-design.md
→ Architecture / Authentication / Transaction / Concurrency

docs/05-data-api-design.md
→ Entity / Relation / Constraint / DTO / API Contract / Error Contract

docs/diagrams/erd.md
→ Data Model 시각화
```

Entity, Column, Relation, Constraint 또는 API Contract는
구현 편의를 이유로 임의 변경하지 않습니다.

설계 변경이 필요한 경우
루트 `AGENTS.md`의 STOP CONDITIONS를 따릅니다.

---

## 4. Search Before Edit

새로운 Class, Interface, DTO, Repository Method,
Exception, Utility 또는 Test Helper를 추가하기 전에
Repository에 동일하거나 유사한 구현이 존재하는지 먼저 확인합니다.

가능하면 다음을 우선 재사용합니다.

- 기존 Package 구조
- 기존 Naming Convention
- 기존 Error Handling 방식
- 기존 DTO Pattern
- 기존 Repository Pattern
- 기존 Validation Pattern
- 기존 Test Fixture / Helper
- 기존 Security Pattern

단순히 구현이 편하다는 이유로
중복 구조를 새로 만들지 않습니다.

기존 패턴을 재사용하려면
Issue 범위 밖 변경이 필요한 경우
임의로 확장하지 않습니다.

---

## 5. Layer Rules

기존 Project Architecture와 Layer 책임을 유지합니다.

일반적으로 다음 책임을 구분합니다.

```text
Controller
→ HTTP Request / Response
→ Validation 진입점
→ Service 호출

Service
→ Use Case
→ Business Logic
→ Transaction Boundary

Repository
→ Persistence 접근

Entity
→ Persistence Model 및 허용된 Domain 상태

DTO
→ API Request / Response Contract
```

AI Agent는 Issue 요구사항 없이
Layer 책임을 재설계하지 않습니다.

다음과 같은 변경이 필요하면
Architecture 영향 여부를 확인하고 필요 시 STOP CONDITION으로 처리합니다.

- Controller에 핵심 Business Logic 추가
- Repository에 Use Case Logic 추가
- Entity 구조의 광범위한 변경
- 기존 Layer 제거 또는 신규 Layer 도입
- 새로운 Architecture Pattern 도입

---

## 6. API Contract Rules

API 구현은 `docs/05-data-api-design.md`의 Contract를 따릅니다.

다음 항목을 임의로 변경하지 않습니다.

- Endpoint
- HTTP Method
- Request Field
- Response Field
- Field Type
- Required / Optional 여부
- HTTP Status
- Error Code
- Authorization Requirement

현재 구현과 API Contract가 충돌하는 경우
구현에 맞추어 Contract를 임의 변경하지 않습니다.

API Contract 변경이 필요하면
STOP CONDITION으로 처리합니다.

---

## 7. Entity / JPA Rules

Entity와 Database Mapping은
`docs/05-data-api-design.md`와 ERD를 기준으로 합니다.

다음 항목을 임의 변경하지 않습니다.

- Entity 추가 / 삭제
- PK 전략
- Relation
- Cardinality
- Column Type
- Nullable
- Unique Constraint
- Index 정책
- 핵심 DB Constraint

JPA 구현에서는 기존 Repository Pattern과
Fetch 전략을 우선 확인합니다.

편의를 이유로 다음을 임의 도입하지 않습니다.

- 불필요한 EAGER Loading
- 광범위한 Cascade
- 광범위한 Orphan Removal
- 무분별한 양방향 Relation
- 전체 Entity 그래프의 불필요한 조회

N+1 또는 Query 문제가 발견되더라도
현재 Issue 범위를 벗어나면 별도 문제로 보고합니다.

---

## 8. Transaction Rules

Transaction Boundary와 정책은
`docs/04-system-design.md`를 따릅니다.

AI Agent는 다음을 임의 변경하지 않습니다.

- Transaction Boundary
- Transaction Propagation
- Isolation 정책
- Lock 전략
- Transaction 내부 처리 순서

`@Transactional`은
단순히 오류를 해결하기 위한 임시 수단으로 추가하지 않습니다.

Transaction 변경이 필요한 경우
기존 설계와 영향 범위를 확인합니다.

정책 변경이 필요하면 STOP CONDITION으로 처리합니다.

---

## 9. Concurrency / Lock Rules

Concurrency와 Lock은
`docs/04-system-design.md`의 전략을 따릅니다.

다음 변경을 임의 수행하지 않습니다.

- Pessimistic Lock → Optimistic Lock 변경
- Lock 제거
- Lock 획득 순서 변경
- Redis Lock 도입
- 새로운 분산 Lock 도입
- Retry 정책 도입
- Idempotency 정책 변경

Concurrency 관련 코드는
단일 요청에서 정상 동작하는 것만으로 완료로 판단하지 않습니다.

Race Condition, Deadlock, Duplicate Processing 가능성을 검토합니다.

기존 Lock 순서 또는 동시성 정책을 변경해야 하는 경우
STOP CONDITION으로 처리합니다.

---

## 10. Security Rules

Authentication / Authorization 구현은
루트 `AGENTS.md`와 `docs/04-system-design.md`,
`docs/05-data-api-design.md`를 따릅니다.

다음 행동을 하지 않습니다.

- 인증 로직 임시 우회
- Authorization Check 제거
- Security Filter 비활성화
- 실제 Secret 값 사용
- Test를 위해 Production Security 정책 완화
- CORS / CSRF 정책 임의 완화
- Token 정책 임의 변경

Security 정책 변경이 필요하면
STOP CONDITION으로 처리합니다.

---

## 11. Validation / Error Handling

Request Validation과 Error Handling은
기존 Project Pattern과 API Error Contract를 따릅니다.

기존 공통 Exception 또는 Error Code가 존재하는 경우
동일 의미의 새로운 Error 구조를 중복 생성하지 않습니다.

다음 변경은 API Contract 영향 여부를 확인합니다.

- Error Code 추가 / 변경
- HTTP Status 변경
- Validation Rule 변경
- Error Response 구조 변경

Contract 변경이 필요한 경우
STOP CONDITION으로 처리합니다.

---

## 12. Code Comment Rules

자명한 Backend 코드에는
불필요한 주석을 추가하지 않습니다.

다음과 같이 코드만으로 구현 이유를 이해하기 어려운 경우
필요한 주석을 작성합니다.

- 복잡한 Business Rule
- Transaction Boundary의 이유
- Lock 획득 순서
- Deadlock 방지 전략
- Idempotency 처리
- Authorization 처리 이유
- Retry / Timeout 정책
- 비직관적인 상태 전이
- 중요한 Workaround

주석은 가능한 경우
"무엇을 하는가"보다 "왜 이렇게 구현했는가"를 설명합니다.

예:

```java
// 여러 Seat를 항상 seatId 오름차순으로 Lock하여
// 동시 예약 시 Deadlock 가능성을 줄인다.
```

다음과 같은 출처 표시용 주석은 작성하지 않습니다.

```text
AI가 작성한 코드
Claude가 작성한 코드
Generated by AI
```

---

## 13. Backend Test Rules

변경한 Backend 기능에 적절한 Test를 수행합니다.

필요에 따라 다음을 선택합니다.

- Unit Test
- Service Test
- Repository Test
- Integration Test
- Security Test
- Concurrency Test
- Regression Test
- Backend Build

Business Rule 변경에는
가능한 경우 정상 Case뿐 아니라 실패 Case와 Boundary Case도 확인합니다.

Transaction / Concurrency 변경에는
관련 경쟁 상태와 실패 Scenario를 검토합니다.

Test 실패를 숨기기 위한 삭제,
비활성화 또는 Assertion 약화는
루트 NEVER Rules에 따라 금지합니다.

---

## 14. Backend STOP CONDITIONS

루트 `AGENTS.md`의 STOP CONDITIONS에 더해
다음 상황에서도 관련 변경을 중지합니다.

- API Contract 변경 필요
- Entity / Relation / Constraint 변경 필요
- Transaction Boundary 변경 필요
- Lock / Concurrency 전략 변경 필요
- Authentication / Authorization 정책 변경 필요
- 새로운 Persistence 기술 도입 필요
- 새로운 Cache / Message Broker / External Service 도입 필요
- 기존 Layer Architecture 변경 필요
- Database Migration 또는 Schema 변경 필요
- Backend 구현을 위해 Frontend Contract 변경이 필요한 경우

STOP CONDITION 발생 시
임의로 설계를 수정하지 않고 Human에게 보고합니다.

---

## 15. Backend Completion Check

작업 종료 전 최소 다음을 확인합니다.

- [ ] 현재 Issue 범위 안에서만 수정했는가
- [ ] Backend Source of Truth를 준수했는가
- [ ] API Contract를 임의 변경하지 않았는가
- [ ] Entity / DB Contract를 임의 변경하지 않았는가
- [ ] Transaction 정책을 임의 변경하지 않았는가
- [ ] Concurrency 전략을 임의 변경하지 않았는가
- [ ] Security 정책을 임의 완화하지 않았는가
- [ ] 기존 Backend Pattern을 우선 재사용했는가
- [ ] 필요한 Backend Test를 수행했는가
- [ ] Test 실패를 숨기지 않았는가

최종 Completion Report는
루트 `AGENTS.md`의 형식을 따릅니다.