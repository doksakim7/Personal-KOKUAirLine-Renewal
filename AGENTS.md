# KOKU Airline Renewal - Agent Rules

## 1. Purpose

본 문서는 Codex, Claude Code 등 AI Agent가
`KOKU Airline Renewal` Repository를
안전하고 일관된 방식으로 수정하기 위한 공통 작업 규칙을 정의합니다.

AI Agent의 역할은
사람이 정의한 요구사항과 설계 기준에 따라
Issue 범위 내 작업을 수행하는 것입니다.

AI Agent는 프로젝트의 Business Rule,
Architecture, API Contract 및 핵심 Data Model을
임의로 재설계하거나 확장하지 않습니다.

설계 권한과 최종 승인 권한은 Human에게 있습니다.

---

## 2. Scope

본 규칙은 Repository 전체에 적용됩니다.

적용 대상:

- Codex
- Claude Code
- 기타 Repository를 읽거나 수정하는 AI Agent
- AI Implementer
- AI Reviewer

향후 하위 디렉토리에 별도의 Agent 규칙이 존재하는 경우:

```text
AGENTS.md
→ Repository 전체 공통 규칙

backend/AGENTS.md
→ Backend 추가 규칙

frontend/AGENTS.md
→ Frontend 추가 규칙
```

하위 규칙은 루트 `AGENTS.md`를 대체하지 않습니다.

루트 규칙과 하위 규칙이 함께 적용되며,
두 규칙이 충돌하는 경우 더 엄격한 규칙을 우선합니다.

규칙 간 해석이 불가능한 충돌이 발생하면
작업을 중지하고 Human에게 보고합니다.

---

## 3. Source of Truth

AI Agent는 작업 전
현재 Issue와 관련된 Source of Truth를 확인해야 합니다.

### Project Scope / Milestone

```text
docs/01-project-plan.md
```

프로젝트 목적, MVP 범위, Milestone 및
프로젝트 전체 방향을 정의합니다.

### Business Rule

```text
docs/02-domain-policy.md
```

Domain Policy 및 Business Rule의 최종 기준입니다.

AI Agent는 Business Rule을
구현 편의를 이유로 임의 변경하지 않습니다.

### UI / User Flow

```text
docs/03-ui-design.md
```

화면 구조, 사용자 Flow 및 UI 정책의 최종 기준입니다.

### System / Architecture

```text
docs/04-system-design.md
```

System Architecture,
Authentication,
Transaction,
Concurrency,
Time Handling,
Infrastructure 방향의 기준입니다.

### Data / API Contract

```text
docs/05-data-api-design.md
```

다음 항목의 최종 기준입니다.

- Entity
- Column
- Relation
- Constraint
- DTO
- API Contract
- Error Contract
- Authorization Matrix

### ERD

```text
docs/diagrams/erd.md
```

ERD는 `05-data-api-design.md`의 Data Model을 시각화합니다.

ERD가 독립적으로 새로운 Entity,
Column, Relation 또는 Constraint를 정의하지 않습니다.

Data 설계가 충돌하는 경우
`05-data-api-design.md`를 우선합니다.

### Development Log

```text
docs/06-development-log.md
```

개발 진행 과정과 주요 기술적 의사결정을 기록합니다.

Development Log는 새로운 Business Rule,
Architecture 또는 Data Contract를 단독으로 정의하지 않습니다.

설계 변경이 필요한 경우
관련 Source of Truth를 먼저 또는 함께 수정한 뒤
Development Log에 변경 이력을 기록합니다.

### Figma Make Guidelines

```text
docs/figma-make-guidelines.md
```

Figma Make 및 UI 생성 작업의 추가 가이드입니다.

Business Rule은 `02-domain-policy.md`,
UI / Flow는 `03-ui-design.md`를 우선합니다.

---

## 4. General Working Rules

AI Agent는 작업을 시작하기 전에 다음 순서를 따릅니다.

1. `git status`를 확인하여 기존 Working Tree 상태를 확인합니다.
2. 현재 Issue를 확인합니다.
3. Issue의 목적과 요구사항을 확인합니다.
4. Acceptance Criteria를 확인합니다.
5. 수정 허용 범위와 수정 금지 범위를 확인합니다.
6. 관련 Source of Truth 문서를 확인합니다.
7. 기존 구현과 Coding Convention을 확인합니다.
8. 작업 범위를 최소화한 뒤 구현합니다.
9. 필요한 Test를 작성하거나 실행합니다.
10. 변경 결과를 Completion Report로 보고합니다.

기본 원칙:

- 최소 변경 원칙을 따릅니다.
- Issue 해결에 필요한 변경만 수행합니다.
- 기존 Architecture와 Convention을 우선합니다.
- 기존 코드 스타일을 불필요하게 재작성하지 않습니다.
- 관련 없는 Refactoring을 수행하지 않습니다.
- 요구되지 않은 기능을 추가하지 않습니다.
- 단순히 더 좋은 방식이 있다는 이유만으로 구조를 변경하지 않습니다.
- 불확실한 요구사항을 임의로 해석하여 구현하지 않습니다.
- 기존 Source of Truth와 충돌하는 구현을 만들지 않습니다.
- 작업 시작 전에 존재하던 변경사항은 AI Agent의 변경으로 간주하지 않습니다.
- AI Agent가 생성하지 않은 기존 변경사항을 임의로 삭제, 복원, 덮어쓰기 또는 재작성하지 않습니다.
- 기존 Working Tree 변경과 현재 Issue 변경이 충돌할 경우 작업을 중지하고 Human에게 보고합니다.

### Work Plan

다음 중 하나에 해당하는 작업은
파일 수정 전에 간단한 Work Plan을 작성합니다.

- 주요 파일을 2개 이상 수정해야 하는 경우
- 여러 Layer 또는 Module에 영향을 주는 경우
- Transaction / Security / Concurrency에 영향을 줄 가능성이 있는 경우
- Regression 영향 범위가 넓다고 판단되는 경우
- 구현 방향이 둘 이상 가능하여 선택이 필요한 경우

Work Plan에는 최소 다음을 포함합니다.

```text
변경 예정 파일
작업 순서
예상 Test
예상 영향 범위
STOP CONDITION 발생 가능성
```

단순 오타 수정,
명확한 단일 파일 변경,
기존 패턴을 그대로 따르는 소규모 작업에는
별도의 Work Plan을 요구하지 않습니다.

설계 문서와 Issue 요구사항이 충돌하는 경우
AI Agent가 어느 한쪽을 임의로 선택하지 않습니다.

해당 상황은 STOP CONDITION으로 처리합니다.

---

## 5. Issue Scope Rules

AI Agent는 현재 할당된 Issue 범위 안에서만 작업합니다.

Issue에 명시된 다음 항목을 작업 경계로 사용합니다.

- 목적
- 요구사항
- Acceptance Criteria
- 수정 허용 범위
- 수정 금지 범위
- 테스트 요구사항
- Stop Conditions
- 완료 조건

다음 행동은 금지합니다.

- Issue에 없는 기능 추가
- 관련 없는 기능 수정
- 관련 없는 코드 정리
- 관련 없는 파일 Formatting
- 대규모 Rename
- 불필요한 Dependency 변경
- 향후 필요할 것이라는 추측에 기반한 선행 구현
- "김에 같이 수정"하는 변경

Issue 범위 밖에서 문제를 발견한 경우:

```text
현재 Issue에서 수정하지 않음
        ↓
Completion Report에 별도 기록
        ↓
필요 시 Human이 새로운 Issue 생성
```

심각한 보안 문제 또는
현재 작업을 안전하게 진행할 수 없게 하는 문제가 아니라면
범위 밖 문제를 임의로 수정하지 않습니다.

---

## 6. Allowed / Forbidden Changes

AI Agent는 Issue에 정의된
`수정 허용 범위`를 우선합니다.

허용된 파일 또는 영역이라도
Issue 해결에 필요하지 않은 변경은 하지 않습니다.

Issue의 `수정 금지 범위`는
명시적인 작업 경계로 취급합니다.

수정 금지 범위의 변경이 필요해진 경우:

```text
작업 중지
→ 변경하지 않음
→ 필요한 변경 내용과 이유 보고
→ Human Approval 대기
```

수정 허용 범위와 실제 필요한 변경이 충돌하면
STOP CONDITION으로 처리합니다.

---

## 7. Git / Branch / PR Rules

AI Agent는 Repository의 기존 Branch 전략과
GitHub Ruleset을 준수합니다.

### Protected Branch

다음 Branch에는 직접 Push하지 않습니다.

```text
main
develop
```

AI Agent는 보호 Branch에서 직접 작업하거나
Commit을 생성하지 않습니다.

작업은 Issue 전용 Branch에서 수행합니다.

예:

```text
feature/*
fix/*
refactor/*
test/*
docs/*
infra/*
ai/*
chore/*
```

Branch 이름은
Repository의 기존 Convention과 Issue 목적을 따릅니다.

### Commit

Commit은 현재 Issue와 관련된 변경만 포함해야 합니다.

권장 Commit Prefix:

```text
feat
fix
refactor
test
docs
infra
ai
chore
```

Commit Message는 변경 목적을 명확하게 표현합니다.

관련 없는 변경을 하나의 Commit에 포함하지 않습니다.

### Push

AI Agent는 현재 작업 Branch에는
허용된 범위에서 Push할 수 있습니다.

다음 Branch에는 직접 Push하지 않습니다.

```text
main
develop
```

### Pull Request

구현 완료 후 변경사항은 Pull Request를 통해 반영합니다.

PR에는 최소 다음 내용을 포함합니다.

- 관련 Issue
- 작업 내용
- 주요 변경 사항
- Test 결과
- 영향 범위
- Known Risks
- Human Review가 필요한 항목

### Merge

AI Agent는 Pull Request를 직접 Merge하지 않습니다.

다음 작업은 Human의 책임입니다.

```text
PR 최종 승인
Merge 결정
Protected Branch 반영
```

AI Reviewer가 PASS를 판단하더라도
자동 Merge의 근거로 사용하지 않습니다.

---

## 8. Agent Roles

AI Agent는 작업 시
`Implementer` 또는 `Reviewer` 역할을 명확하게 구분합니다.

---

### 8.1 Implementer

Implementer의 역할:

- Issue 요구사항 확인
- 관련 Source of Truth 확인
- Issue 범위 내 구현
- 필요한 Test 작성
- 기존 Test 실행
- Build 또는 정적 검증 수행
- 변경사항 보고
- 필요한 경우 PR 생성

Implementer는 구현 과정에서
새로운 Business Rule 또는 Architecture를 임의 결정하지 않습니다.

Issue 요구사항을 구현하기 위해
설계 변경이 필요하다고 판단되면
STOP CONDITION으로 처리합니다.

Implementer는 Test 실패를 숨기거나
무력화하지 않습니다.

---

### 8.2 Reviewer

Reviewer의 역할:

- Issue 요구사항 충족 여부 검토
- Acceptance Criteria 검증
- Source of Truth 준수 여부 검토
- Issue 범위 밖 변경 여부 확인
- Architecture 위반 여부 확인
- API Contract 위반 여부 확인
- Test 누락 여부 확인
- 보안 위험 검토
- Regression 가능성 검토

Reviewer는
검토 요청이 없는 구현 변경을 임의 수행하지 않습니다.

문제가 발견되면:

```text
PASS
또는
NEEDS_FIX
```

형태로 명확하게 판단하고
문제의 위치와 이유를 보고합니다.

Reviewer는 `PASS`를 판단한 경우에도
검토 근거를 최소한으로 남깁니다.

PASS Report에는 가능한 범위에서 다음을 포함합니다.

- 검토한 Acceptance Criteria
- 확인한 주요 변경 파일
- 실행하거나 확인한 Test
- Source of Truth 위반 여부
- 남아 있는 Risk 또는 Human Review 필요 여부

단순히 "문제 없음" 또는 "PASS"만 출력하고
검토 근거를 생략하지 않습니다.

Reviewer도 PR을 직접 Merge하지 않습니다.

`NEEDS_FIX`인 경우에는
문서 또는 파일 위치, 문제 원인, 수정이 필요한 이유를 명확하게 기록합니다.

가능하면 동일한 AI 작업 세션이
Implementer와 최종 Reviewer 역할을 동시에 수행하지 않도록 합니다.

---

## 9. NEVER Rules

다음 행동은 Human Approval 여부와 관계없이
AI Agent가 수행해서는 안 됩니다.

### Git / Repository

- `main` 직접 Push 금지
- `develop` 직접 Push 금지
- PR 직접 Merge 금지
- 보호 Branch 우회 금지
- GitHub Ruleset 임의 변경 금지
- Branch Protection 임의 변경 금지
- 다른 작업자의 변경사항 임의 삭제 금지
- 파괴적 Git 명령 사용 금지
- 다음 명령 또는 동등한 효과를 가지는 작업을 임의 수행하지 않음:
  - `git reset --hard`
  - `git clean -fd`
  - `git checkout -- .`
  - `git restore .`
  - `git push --force`
  - `git push --force-with-lease`

### Scope

- Issue 범위 밖 기능 추가 금지
- 관련 없는 Refactoring 금지
- 관련 없는 파일 수정 금지
- 요구되지 않은 대규모 Rename 금지

### Test

- Test 실패를 숨기거나 CI를 통과시키기 위한 기존 Test 삭제 금지
- Test 실패를 숨기기 위한 수정 금지
- Test를 무력화하여 CI를 통과시키는 행동 금지
- `@Disabled`, `skip`, `xit` 등의 임의 추가 금지
- Assertion 약화로 Test를 억지로 통과시키는 행동 금지
- 실패한 Test 결과를 성공으로 보고하는 행동 금지

### Security

- Secret 값 출력 금지
- Secret 값 커밋 금지
- Credential 노출 금지
- 운영 Credential 사용 금지
- 보안 정책 우회 금지

### Infrastructure

- AWS 운영 환경 임의 변경 금지
- GitHub Secrets 임의 변경 금지
- Production Database 임의 변경 금지
- 운영 Resource 삭제 금지

---

## 10. STOP CONDITIONS

다음 상황이 발생하면
AI Agent는 작업을 즉시 중단하고 Human에게 보고합니다.

- API Contract 변경이 필요한 경우
- DB 핵심 모델 변경이 필요한 경우
- 핵심 Architecture 변경이 필요한 경우
- 새로운 외부 Dependency가 필요한 경우
- 기존 Dependency의 Major Version Upgrade가 필요한 경우
- Package Manager 또는 Build Tool 변경이 필요한 경우
- 기존 Architecture와 요구사항이 충돌하는 경우
- Domain Policy 변경이 필요한 경우
- 인증 / 인가 정책 변경이 필요한 경우
- Transaction Boundary 변경이 필요한 경우
- 기존 동시성 전략 변경이 필요한 경우
- 수정 금지 범위의 변경이 필요한 경우
- Issue 범위를 벗어난 변경 없이는 구현할 수 없는 경우
- Secret 또는 Credential 접근이 필요한 경우
- GitHub Secrets 변경이 필요한 경우
- 운영 Infrastructure 접근이 필요한 경우
- 보호 Branch 권한이 필요한 경우
- Merge 권한이 필요한 경우
- Agent 규칙 사이에 충돌이 발생한 경우
- Source of Truth 문서들이 서로 충돌하는 경우
- 요구사항이 불충분하여 안전하게 구현할 수 없는 경우

STOP CONDITION 발생 시 다음 순서를 따릅니다.

```text
1. 관련 변경 중지
2. 임의 해결 금지
3. 현재까지 안전하게 수행된 작업 보존
4. 문제 위치 확인
5. 필요한 변경과 영향 범위 정리
6. Human에게 보고
7. 승인 또는 추가 지시 대기
```

STOP CONDITION은
"작업 실패"를 의미하지 않습니다.

사람의 설계 또는 권한 판단이 필요한 상황에서
안전하게 작업을 중단하는 정상적인 절차입니다.

---

## 11. Test Rules

AI Agent는 Test를
구현의 일부로 취급합니다.

변경된 기능에 적절한 Test가 필요한 경우
Issue 범위 안에서 작성합니다.

작업 완료 전 가능한 범위에서 다음을 수행합니다.

- 관련 Unit Test
- 관련 Integration Test
- Regression Test
- Backend Build
- Frontend Build
- Lint / Type Check
- 필요한 정적 검증

모든 작업에서 모든 Test를 실행해야 하는 것은 아닙니다.

현재 Issue와 변경 범위에 적절한 검증을 선택합니다.

Test를 실행하지 못한 경우
실행하지 않은 이유를 Completion Report에 기록합니다.

기존 Test가 실패하는 경우:

```text
원인 확인
        ↓
현재 변경과 관련 있는지 판단
        ↓
실패 사실 보고
```

Test를 삭제하거나 약화하여
실패를 숨기지 않습니다.

기존 Test가 잘못되었다고 판단되더라도
Test 변경이 현재 Issue 범위를 벗어나거나
Business Rule 변경을 의미한다면
STOP CONDITION으로 처리합니다.

---

## 12. Security / Secret Rules

AI Agent는 Secret과 Credential을
최소 권한 원칙에 따라 취급합니다.

다음 값의 원문을
읽거나 출력하거나 기록하거나 커밋하려고 시도하지 않습니다.

예:

```text
AWS Access Key
AWS Secret Access Key
Database Password
JWT Secret
OAuth Client Secret
GitHub Token
GitHub Secrets
Production Credential
Private Key
API Secret
```

`.env` 등 Secret을 포함할 가능성이 높은 파일은
AI Agent가 직접 열어 실제 값을 확인하지 않습니다.

환경 변수 이름이나 설정 구조 확인이 필요한 경우에는
`.env.example`, Example Configuration 또는 문서를 우선 사용합니다.

실제 Secret 값 확인이 필요한 상황은
STOP CONDITION으로 처리하고 Human에게 요청합니다.

Secret의 이름이나
환경 변수 구조만 필요한 경우에는
실제 값 대신 Placeholder를 사용합니다.

예:

```text
GOOGLE_CLIENT_ID=<configured-by-human>
GOOGLE_CLIENT_SECRET=<configured-by-human>
```

Secret 값이 구현에 필요하면
AI Agent가 값을 생성하거나 추측하지 않습니다.

다음과 같이 처리합니다.

```text
환경 변수 이름 정의
        ↓
Placeholder 또는 Example 작성
        ↓
실제 Secret 설정은 Human에게 요청
```

다음 파일은 실제 Secret을 포함하지 않아야 합니다.

```text
.env.example
application-example.yml
README
docs
Test Fixture
```

Secret 노출 가능성을 발견한 경우
값을 반복해서 출력하지 않고
Human에게 보안 문제를 보고합니다.

---

## 13. Completion Report

AI Implementer는 작업 완료 시
다음 형식으로 결과를 보고합니다.

```text
## Completion Report

### Changed Files

- 변경한 파일 목록

### Implemented

- 구현 또는 수정한 내용
- 충족한 Acceptance Criteria

### Tests

- 실행한 Test
- 실행한 Build / Lint / Type Check

### Test Result

- PASS / FAILED
- 실패한 경우 원인

### Unchanged Scope

- 의도적으로 수정하지 않은 관련 영역

### Risks

- Regression 가능성
- Technical Risk
- 알려진 제한사항

### Stop Conditions

- 발생 여부
- 발생했다면 내용

### Human Review Required

- 사람이 확인하거나 결정해야 할 항목
```

작업이 완료되지 않은 경우에도
현재 상태를 숨기지 않습니다.

예:

```text
SUCCESS
PARTIAL
BLOCKED
FAILED
```

중 적절한 상태를 명확하게 보고합니다.

---

## 14. Human Approval Required

다음 사항은
AI Agent가 독자적으로 최종 결정하지 않습니다.

Human Approval이 필요합니다.

- Domain Policy 변경
- MVP Scope 변경
- API Contract 변경
- DB 핵심 Entity / Relation / Constraint 변경
- 핵심 Architecture 변경
- 인증 / 인가 정책 변경
- Transaction 정책 변경
- 동시성 전략 변경
- 새로운 외부 Dependency 추가
- 기존 Dependency의 Major Version Upgrade
- Package Manager 또는 Build Tool 변경
- 새로운 외부 Service 도입
- Secret / Credential의 환경 변수 구조 또는 연동 방식 변경
- AWS 또는 운영 Infrastructure 설계 / 구성 변경
- Production Database Schema 또는 운영 정책 변경
- 기존 Source of Truth 간 충돌 해결
- 기존 Commit History를 재작성하는 Rebase / History Rewrite

AI Agent는 위 상황에서
권장안을 제시할 수 있습니다.

그러나 Human Approval 전에
해당 변경을 실제 적용하지 않습니다.