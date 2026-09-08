# KOKU Airline Renewal - Frontend Agent Rules

## 1. Purpose

본 문서는 `frontend/` 영역에서 작업하는 AI Agent가 따라야 하는
Frontend 전용 추가 규칙을 정의합니다.

본 문서는 루트 `AGENTS.md`를 대체하지 않습니다.

Frontend 작업에는 다음 규칙이 함께 적용됩니다.

```text
루트 AGENTS.md
→ Repository 공통 정책 및 안전 규칙

frontend/AGENTS.md
→ Frontend 구현 추가 규칙
```

루트 규칙과 함께 적용하며,
충돌 시 루트 `AGENTS.md`에 정의된 Rule Conflict 처리 방식을 따릅니다.

---

## 2. Scope

본 규칙은 원칙적으로 다음 영역에 적용됩니다.

```text
frontend/
```

주요 대상:

- React
- TypeScript
- Component
- Page
- Hook
- State
- API Client
- Form
- Validation
- Routing
- Styling
- Responsive UI
- i18n
- Frontend Test

Frontend 작업 중 다른 영역 수정이 필요해진 경우
현재 Issue의 수정 허용 범위를 확인합니다.

허용 범위 밖 변경이 필요하면
임의로 수정하지 않고 STOP CONDITION으로 처리합니다.

---

## 3. Frontend Source of Truth

Frontend 작업 전
현재 Issue와 관련된 Source of Truth를 확인합니다.

특히 다음 문서를 우선 확인합니다.

```text
docs/02-domain-policy.md
→ Business Rule

docs/03-ui-design.md
→ UI / Screen / User Flow

docs/05-data-api-design.md
→ API Contract / DTO / Error Contract

docs/figma-make-guidelines.md
→ UI 생성 추가 가이드
```

UI / Flow는 `docs/03-ui-design.md`를 기준으로 합니다.

API Request / Response는
`docs/05-data-api-design.md`를 기준으로 합니다.

Frontend 구현 편의를 위해
Business Rule, User Flow 또는 API Contract를 임의 변경하지 않습니다.

---

## 4. Search Before Edit

새로운 Component, Hook, Utility, API Client,
Type, Form Pattern 또는 State 구조를 추가하기 전에
동일하거나 유사한 구현이 존재하는지 먼저 확인합니다.

가능하면 다음을 우선 재사용합니다.

- 기존 Component
- 기존 Hook
- 기존 Layout
- 기존 API Client Pattern
- 기존 Type
- 기존 Form Pattern
- 기존 Error Handling
- 기존 Styling Pattern
- 기존 Responsive Pattern

단순히 구현이 편하다는 이유로
유사 Component나 Utility를 중복 생성하지 않습니다.

재사용을 위해 Issue 범위 밖 Refactoring이 필요한 경우
임의로 수정하지 않습니다.

---

## 5. Component Rules

Component는 기존 Project 구조와 Convention을 따릅니다.

가능한 경우 다음 책임을 분리합니다.

```text
Page
→ 화면 구성 및 Use Case 연결

Component
→ 재사용 가능한 UI

Hook
→ 재사용 가능한 상태 / 동작

API Layer
→ Backend 통신

Type
→ Frontend Type 정의
```

단, 현재 Repository가 다른 구조를 사용한다면
기존 구조를 우선합니다.

AI Agent는 Issue 요구사항 없이
Component Architecture를 재설계하지 않습니다.

다음과 같은 변경은 필요성을 먼저 검토합니다.

- 대규모 Component 분리
- 대규모 Component 통합
- 새로운 상태관리 Library 도입
- 새로운 Design System 도입
- Directory 구조 전면 변경

---

## 6. TypeScript Rules

기존 TypeScript 설정과 Convention을 따릅니다.

가능한 경우 명확한 Type을 사용하고,
타입 오류를 피하기 위해 무분별하게 다음을 사용하지 않습니다.

```text
any
unknown 강제 Casting
as any
불필요한 Non-null Assertion
```

API Response Type을
실제 Backend Contract와 다르게 임의 정의하지 않습니다.

Backend Contract가 불명확하거나
Frontend 요구사항과 충돌하면 STOP CONDITION으로 처리합니다.

---

## 7. API Rules

Frontend API 구현은
`docs/05-data-api-design.md`를 기준으로 합니다.

다음 항목을 Frontend 편의를 위해 임의 변경하거나 가정하지 않습니다.

- Endpoint
- HTTP Method
- Request Field
- Response Field
- Error Code
- Status Code
- Authorization Requirement

Contract에 없는 Field가 필요한 경우
Frontend에서 임의로 Backend Response를 가정하지 않습니다.

예:

```text
API에는 passengerName 없음
↓
Frontend에서 passengerName이 올 것이라고 가정
↓
금지
```

API Contract 변경이 필요하면
STOP CONDITION으로 처리합니다.

---

## 8. State Rules

기존 State 관리 방식을 우선 사용합니다.

Issue 해결만을 위해
새로운 전역 State Library를 임의 도입하지 않습니다.

다음 상황에서는 상태 흐름을 명확히 확인합니다.

- 다단계 예약 Flow
- 이전 Step과 다음 Step 간 데이터 공유
- 비동기 Request
- Loading / Success / Error 상태
- 중복 Submit 방지
- Race Condition 가능성
- 뒤로가기 / 새로고침 처리

State 구조 변경이 여러 화면의 Flow에 영향을 주는 경우
UI / Flow Source of Truth를 확인합니다.

---

## 9. Async / Request Rules

API 요청에서는 필요한 경우 다음 상태를 명확하게 처리합니다.

- Loading
- Success
- Error
- Empty
- Retry 가능 여부

동일 요청이 중복 실행될 가능성이 있는 UI에서는
중복 Submit 또는 Race Condition 가능성을 검토합니다.

단순히 버튼을 비활성화하는 등의 구현이
Business Rule 또는 Backend Idempotency를 대신한다고 가정하지 않습니다.

Backend 동작을 Frontend에서 임의 추측하지 않습니다.

---

## 10. UI / Flow Rules

화면 구조와 User Flow는
`docs/03-ui-design.md`를 기준으로 합니다.

다음 변경을 구현 편의를 이유로 임의 수행하지 않습니다.

- 화면 Step 추가 / 삭제
- 예약 Flow 순서 변경
- 입력 필수 여부 변경
- 주요 Navigation 변경
- Error Flow 변경
- 사용자 권한별 화면 정책 변경

UI Design과 실제 요구사항이 충돌하면
한쪽을 임의 선택하지 않고 STOP CONDITION으로 처리합니다.

---

## 11. Responsive / Accessibility Rules

기존 Responsive 정책과 UI Pattern을 유지합니다.

새로운 UI를 추가하는 경우
현재 Issue 범위에서 필요한 수준의 다음 항목을 확인합니다.

- Keyboard 사용 가능 여부
- Form Label
- Button 의미
- Error Message 인식 가능성
- Mobile Layout
- Overflow
- 주요 Interactive Element의 접근 가능성

단, 현재 Issue와 관련 없는
대규모 Accessibility Refactoring은 수행하지 않습니다.

---

## 12. i18n Rules

기존 다국어 정책과 Pattern을 따릅니다.

사용자에게 노출되는 문구를 추가할 때
현재 Project의 i18n 구조가 존재하면 이를 우선 사용합니다.

Issue 요구사항 없이
기존 다국어 구조를 변경하거나 새로운 i18n Library를 도입하지 않습니다.

번역 문구가 Business Rule 또는 Domain 의미에 영향을 주는 경우
임의로 의미를 변경하지 않습니다.

---

## 13. Styling Rules

기존 Styling 방식과 Convention을 우선합니다.

다음 행동을 하지 않습니다.

- Issue와 무관한 전체 UI Formatting
- 대규모 CSS 재작성
- 새로운 CSS Framework 임의 도입
- 새로운 UI Library 임의 도입
- 기존 Design System 임의 교체

현재 Component의 요구사항을 해결하는 데 필요한
최소 Styling 변경만 수행합니다.

---

## 14. Code Comment Rules

자명한 Frontend 코드에는
불필요한 주석을 추가하지 않습니다.

다음과 같이 구현 이유를 코드만으로 이해하기 어려운 경우
필요한 주석을 작성합니다.

- 복잡한 상태 전이
- 다단계 예약 Flow의 데이터 유지 이유
- 비동기 처리 순서
- 중복 Request 방지
- Race Condition 방지
- Responsive 분기 이유
- i18n 예외 처리 이유
- 비직관적인 Browser 동작 대응
- 중요한 Workaround

주석은 가능한 경우
"무엇을 하는가"보다 "왜 이렇게 구현했는가"를 설명합니다.

다음과 같은 출처 표시용 주석은 작성하지 않습니다.

```text
AI가 작성한 코드
Claude가 작성한 코드
Generated by AI
```

---

## 15. Frontend Test Rules

Frontend 변경에 적절한 검증을 수행합니다.

필요에 따라 다음을 선택합니다.

- Unit Test
- Component Test
- User Interaction Test
- Form Validation Test
- Regression Test
- Lint
- Type Check
- Frontend Build

화면 Flow를 변경한 경우
정상 Flow뿐 아니라 필요한 Error / Boundary Scenario를 확인합니다.

API 연결 변경의 경우
Request / Response Type이 Contract와 일치하는지 확인합니다.

Test를 통과시키기 위한 삭제,
비활성화 또는 Assertion 약화는
루트 NEVER Rules에 따라 금지합니다.

---

## 16. Frontend STOP CONDITIONS

루트 `AGENTS.md`의 STOP CONDITIONS에 더해
다음 상황에서도 관련 변경을 중지합니다.

- API Contract 변경 필요
- 새로운 API Field 필요
- UI / User Flow 정책 변경 필요
- Business Rule 변경 필요
- 새로운 State Management Library 필요
- 새로운 UI Framework / Design System 필요
- Routing Architecture 변경 필요
- Frontend 구현을 위해 Backend 수정이 필요한 경우
- 인증 / 인가 Flow 변경 필요
- 기존 i18n 정책 변경 필요

STOP CONDITION 발생 시
Frontend에서 임시 우회 구현을 만들지 않고
Human에게 보고합니다.

---

## 17. Frontend Completion Check

작업 종료 전 최소 다음을 확인합니다.

- [ ] 현재 Issue 범위 안에서만 수정했는가
- [ ] UI / User Flow Source of Truth를 준수했는가
- [ ] API Contract를 임의로 가정하거나 변경하지 않았는가
- [ ] 기존 Component / Hook / API Pattern을 우선 재사용했는가
- [ ] TypeScript 오류를 강제 Casting으로 숨기지 않았는가
- [ ] 필요한 Loading / Error 상태를 처리했는가
- [ ] 관련 없는 UI / CSS Refactoring을 하지 않았는가
- [ ] 필요한 Frontend Test를 수행했는가
- [ ] Lint / Type Check / Build 결과를 숨기지 않았는가

최종 Completion Report는
루트 `AGENTS.md`의 형식을 따릅니다.