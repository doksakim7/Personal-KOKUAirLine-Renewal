# KOKU Airline Renewal - Claude Code Rules

## 1. Purpose

본 문서는 Claude Code가
`KOKU Airline Renewal` Repository에서 작업할 때 따라야 하는
Tool-specific 실행 규칙을 정의합니다.

Claude Code는 루트 `AGENTS.md`의 공통 AI Agent 규칙을 반드시 준수합니다.

`CLAUDE.md`는 `AGENTS.md`를 대체하지 않습니다.

```text
AGENTS.md
→ Repository 공통 정책 및 안전 규칙

CLAUDE.md
→ Claude Code 실행 절차 및 작업 방식
```

`AGENTS.md`와 `CLAUDE.md`가 함께 적용되는 경우
`AGENTS.md`의 공통 정책과 안전 규칙을 기준으로 합니다.

`CLAUDE.md`는 Claude Code의 실행 절차를 구체화할 수 있지만
`AGENTS.md`의 규칙을 완화하거나 우회할 수 없습니다.

두 문서 사이의 차이가
`AGENTS.md`를 기준으로 명확하게 해소 가능한 경우에는
해당 규칙을 적용하여 작업을 계속할 수 있습니다.

두 문서를 동시에 충족할 수 없거나
충돌을 안전하게 해석할 수 없는 경우에는
임의로 선택하지 않고 STOP CONDITION으로 처리하여
Human에게 보고합니다.

---

## 2. Rule Priority

Claude Code는 다음 순서로 작업 기준을 확인합니다.

```text
1. Human의 현재 명시적 지시
2. Repository의 AGENTS.md 및 현재 작업 경로의 하위 AGENTS.md
3. CLAUDE.md
4. GitHub Issue 및 관련 Source of Truth 문서
5. 기존 구현 및 Coding Convention
```

Repository의 루트 `AGENTS.md`와
현재 작업 경로의 하위 `AGENTS.md`는 함께 적용합니다.

하위 `AGENTS.md`는 루트 `AGENTS.md`를 대체하지 않습니다.

두 규칙이 모두 적용되는 상황에서 차이가 있는 경우
더 엄격한 규칙을 적용합니다.

어느 규칙이 더 엄격한지 명확하지 않거나
두 규칙을 동시에 충족할 수 없는 경우에는
임의로 선택하지 않고 STOP CONDITION으로 처리합니다.

`AGENTS.md`와 `CLAUDE.md` 사이에 차이가 있는 경우
`AGENTS.md`의 공통 정책 및 안전 규칙을 우선 기준으로 사용합니다.

`CLAUDE.md`는 `AGENTS.md`를 보완하는 실행 규칙이며,
`AGENTS.md`보다 느슨한 규칙을 적용하거나
공통 안전 정책을 우회하는 근거로 사용할 수 없습니다.

두 문서의 규칙을 함께 적용할 수 없거나
어느 규칙을 적용해야 하는지 안전하게 판단할 수 없는 경우에는
STOP CONDITION으로 처리합니다.

위 목록은 작업 기준을 확인하기 위한 순서이며,
GitHub Issue와 Source of Truth 사이의 충돌을
우선순위로 해결하기 위한 규칙이 아닙니다.

GitHub Issue의 요구사항과 Source of Truth가 충돌하는 경우
Claude Code는 어느 한쪽을 임의로 우선하지 않습니다.

해당 상황은 STOP CONDITION으로 처리하고
충돌 내용을 Human에게 보고합니다.

단, Human의 지시가 다음과 충돌하는 경우에는
단순히 실행하지 않고 충돌 사실을 보고합니다.

- Repository 보안 정책
- Protected Branch 정책
- Secret 보호 규칙
- NEVER Rules
- GitHub Ruleset
- 운영 환경 보호 정책

Source of Truth 간 충돌 또는
규칙 간 해석이 불가능한 충돌이 발생하면
STOP CONDITION으로 처리합니다.

---

## 3. Before Starting Work

Claude Code는 파일을 수정하기 전에
다음 순서로 작업 환경을 확인합니다.

1. 현재 Branch를 확인합니다.
2. `git status`를 확인합니다.
3. 기존 Working Tree 변경사항을 확인합니다.
4. 현재 Issue를 확인합니다.
5. Issue의 목적과 요구사항을 확인합니다.
6. Acceptance Criteria를 확인합니다.
7. 수정 허용 범위와 수정 금지 범위를 확인합니다.
8. 관련 Source of Truth 문서를 확인합니다.
9. 기존 구현과 Coding Convention을 확인합니다.
10. 필요한 경우 Work Plan을 작성합니다.

Claude Code는 새로운 Class, Function, Component, DTO, Utility,
Test Helper 또는 설정 구조를 추가하기 전에
Repository 내에 동일하거나 유사한 구현이 이미 존재하는지 먼저 검색합니다.

기존 구현으로 요구사항을 충족할 수 있는 경우
새로운 구조를 중복 생성하지 않고 기존 패턴을 우선 재사용합니다.

유사 구현이 존재하지만 현재 Issue 요구사항과 충돌하거나
재사용을 위해 범위 밖 변경이 필요한 경우
임의로 통합하지 않고 STOP CONDITION 또는 Human Review 대상으로 보고합니다.

Claude Code가 생성하지 않은 기존 변경사항은
임의로 삭제, 복원, 덮어쓰기 또는 재작성하지 않습니다.

기존 변경사항과 현재 작업이 충돌하면
작업을 시작하지 않고 Human에게 보고합니다.

---

## 4. Source of Truth Check

Claude Code는 Issue와 관련된 설계 문서를
작업 전에 확인합니다.

### Project Scope / Milestone

```text
docs/01-project-plan.md
```

### Business Rule

```text
docs/02-domain-policy.md
```

### UI / Flow

```text
docs/03-ui-design.md
```

### System / Architecture

```text
docs/04-system-design.md
```

### Data / API Contract

```text
docs/05-data-api-design.md
```

### ERD

```text
docs/diagrams/erd.md
```

### Development Log

```text
docs/06-development-log.md
```

### Figma Make Guidelines

```text
docs/figma-make-guidelines.md
```

Claude Code는 구현 편의를 이유로
Source of Truth를 임의 변경하지 않습니다.

설계 변경이 필요하면
STOP CONDITION으로 처리합니다.

---

## 5. Work Plan

다음 중 하나에 해당하면
Claude Code는 실제 파일 수정 전에 간단한 Work Plan을 작성합니다.

- 주요 파일을 2개 이상 수정해야 하는 경우
- 여러 Layer 또는 Module에 영향을 주는 경우
- Transaction에 영향을 줄 가능성이 있는 경우
- Security에 영향을 줄 가능성이 있는 경우
- Concurrency에 영향을 줄 가능성이 있는 경우
- Regression 영향 범위가 넓은 경우
- 구현 방향이 둘 이상 존재하는 경우
- 새로운 Dependency 또는 Tool이 필요할 가능성이 있는 경우

Work Plan에는 최소 다음을 포함합니다.

```text
1. 변경 예정 파일
2. 작업 순서
3. 예상 Test
4. 예상 영향 범위
5. STOP CONDITION 발생 가능성
```

다음 작업에는 별도의 Work Plan이 필요하지 않습니다.

- 단순 오타 수정
- 명확한 단일 파일 수정
- 기존 패턴을 그대로 따르는 소규모 변경
- 문서의 단순 문구 수정

Work Plan은 새로운 Architecture를 제안하거나
Issue 범위를 확장하기 위한 수단으로 사용하지 않습니다.

---

## 6. Implementer Mode

Claude Code가 Implementer 역할을 수행하는 경우
다음 원칙을 따릅니다.

### Implement Only the Issue

현재 Issue 해결에 필요한 변경만 수행합니다.

다음 행동을 하지 않습니다.

- Issue에 없는 기능 추가
- 관련 없는 Refactoring
- 관련 없는 파일 Formatting
- 불필요한 Rename
- 향후 필요할 것이라는 추측에 기반한 선행 구현
- "김에 같이 수정"하는 변경

### Minimal Change

기존 구조를 유지하면서
Issue를 해결할 수 있는 최소 변경을 우선합니다.

더 좋은 구현 방법이 존재한다는 이유만으로
기존 Architecture나 구조를 재설계하지 않습니다.

### Scope Expansion Check

작업 중 실제 변경 범위가 Work Plan 또는 Issue에서 예상한 범위보다
유의미하게 커지는 경우 Claude Code는 계속 확장하지 않습니다.

다음과 같은 상황은 Scope Expansion으로 간주합니다.

- 예상보다 많은 주요 파일을 수정해야 하는 경우
- 새로운 Layer 또는 Module까지 변경이 확장되는 경우
- 관련 없는 Package / Directory까지 수정이 필요한 경우
- 구현 과정에서 추가 기능이나 별도 Refactoring이 필요해지는 경우
- Diff가 커져 현재 Issue의 최소 변경 원칙을 벗어났다고 판단되는 경우

이 경우 Claude Code는:

```text
1. 추가 변경을 중지
2. 현재까지 변경한 범위 확인
3. 확장이 필요한 이유 정리
4. 예상 추가 변경 파일 및 영향 범위 보고
5. Human의 추가 지시 대기
```

단순히 파일 수가 많다는 이유만으로 중단하지 않습니다.

현재 Issue 특성상 여러 파일 변경이 원래 예상된 작업이라면
Work Plan 범위 내에서 계속 진행할 수 있습니다.

### Existing Patterns

가능한 경우 기존 Repository의 다음 패턴을 따릅니다.

- Package / Directory 구조
- Naming Convention
- Error Handling
- Test 구조
- DTO 구조
- Coding Style
- Commit Convention

기존 패턴 자체에 문제가 있다고 판단되더라도
현재 Issue 범위 밖이면 임의 수정하지 않습니다.

---

## 7. Reviewer Mode

Claude Code가 Reviewer 역할을 수행하는 경우
구현 작업과 리뷰 작업을 구분합니다.

Reviewer는 다음을 확인합니다.

- Issue 요구사항 충족 여부
- Acceptance Criteria 충족 여부
- Source of Truth 준수 여부
- 수정 허용 범위 준수 여부
- 수정 금지 범위 침범 여부
- API Contract 위반 여부
- Architecture 위반 여부
- Test 누락 여부
- Regression 가능성
- Security Risk
- 관련 없는 변경 존재 여부

Reviewer는 검토 요청 없이
직접 구현을 수정하지 않습니다.

문제가 발견되면 다음 중 하나로 판정합니다.

```text
PASS
NEEDS_FIX
```

### PASS

PASS인 경우에도 최소 다음 근거를 보고합니다.

- 검토한 Acceptance Criteria
- 확인한 주요 변경 파일
- 실행하거나 확인한 Test
- Source of Truth 위반 여부
- 남아 있는 Risk
- Human Review 필요 여부

단순히 다음과 같이 끝내지 않습니다.

```text
문제 없음
PASS
```

### NEEDS_FIX

NEEDS_FIX인 경우 다음을 명확하게 보고합니다.

- 파일 또는 문서
- 문제 위치
- 문제 원인
- 수정이 필요한 이유
- 관련 Acceptance Criteria 또는 Source of Truth

Reviewer는 PR을 직접 Merge하지 않습니다.

가능하면 동일한 Claude Code Session이
Implementer와 최종 Reviewer 역할을 동시에 수행하지 않습니다.

---

## 8. File Modification Rules

Claude Code는 Issue의
`수정 허용 범위`에 명시된 파일 또는 영역을 우선합니다.

허용된 파일이라도
Issue 해결에 필요하지 않은 변경은 하지 않습니다.

수정 금지 범위 변경이 필요하면
파일을 수정하지 않고 STOP CONDITION으로 처리합니다.

파일 수정 시 다음을 지킵니다.

- 필요한 최소 범위만 수정
- 불필요한 전체 파일 재작성 금지
- 관련 없는 Formatting 금지
- 기존 주석 임의 삭제 금지
- 기존 사용자 변경사항 덮어쓰기 금지
- 자동 생성 파일을 임의 수정하지 않음

Issue 범위 밖 문제를 발견하면
현재 작업에서 수정하지 않고 Completion Report에 기록합니다.

### Generated Files

Claude Code는 자동 생성 파일이나 Build Artifact를
Source File처럼 직접 수정하지 않습니다.

예:

```text
build/
dist/
target/
coverage/
generated/
자동 생성 API Client
자동 생성 Schema
컴파일 결과물
```

자동 생성 결과를 변경해야 하는 경우
원본 Source 또는 Generator 설정을 수정한 뒤
정상적인 생성 절차를 통해 다시 생성합니다.

Lock File은 자동 생성 파일로 취급하되,
Dependency 변경이 Human Approval을 통해 명시적으로 허용된 경우에만
Package Manager의 정상적인 명령을 통해 갱신합니다.

Generated File 또는 Lock File을
수동 편집하여 원하는 결과를 만들지 않습니다.

---

## 9. Code Comment Rules

Claude Code는 모든 코드에 주석을 추가하지 않습니다.

자명한 코드를 그대로 설명하는 주석은 피합니다.

다음과 같이 구현 의도를 코드만으로 이해하기 어려운 경우에는
필요한 주석을 작성합니다.

- 복잡한 Business Rule
- Transaction 처리 이유
- Concurrency / Lock 처리 이유
- Security 처리 이유
- Idempotency 처리
- 비직관적인 상태 전이
- 일반적인 구현과 다른 선택을 한 이유
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
Generated by Claude
```

Backend / Frontend의 세부 주석 규칙은
각 하위 `AGENTS.md`가 존재하는 경우 해당 규칙을 추가로 따릅니다.

---

## 10. Test Execution Rules

Claude Code는 Test를 구현의 일부로 취급합니다.

변경 범위에 따라 적절한 검증을 선택합니다.

예:

```text
Backend
→ Unit Test
→ Integration Test
→ Build

Frontend
→ Unit Test
→ Lint
→ Type Check
→ Build
```

모든 작업에서 전체 Test Suite를 실행할 필요는 없습니다.

현재 변경과 직접 관련된 Test를 우선 실행합니다.

Test를 실행하지 못하면
Completion Report에 이유를 기록합니다.

Test 실패 시:

```text
1. 실패 원인 확인
2. 현재 변경과 관련 있는지 확인
3. 관련 있는 경우 Issue 범위 내에서 수정
4. 관련 없는 경우 실패 사실 보고
```

Test를 통과시키기 위해 다음 행동을 하지 않습니다.

- Test 실패를 숨기거나 CI를 통과시키기 위한 기존 Test 삭제
- Test 비활성화
- `@Disabled` 추가
- `skip` 추가
- `xit` 추가
- Assertion 약화
- 실패 결과 은폐

기존 Test 자체가 잘못되었다고 판단되더라도
현재 Issue 범위를 벗어나면 STOP CONDITION으로 처리합니다.

---

## 11. Dependency Rules

Claude Code는 편의를 위해
새로운 Dependency를 임의 추가하지 않습니다.

다음 상황은 STOP CONDITION입니다.

- 새로운 외부 Dependency 추가 필요
- 기존 Dependency Major Version Upgrade 필요
- 새로운 Plugin 필요
- 새로운 외부 Service 필요
- Package Manager 변경 필요
- Build Tool 변경 필요

Claude Code는 가능한 경우
현재 프로젝트에 이미 존재하는 Dependency와 기능을 우선 사용합니다.

---

## 12. Git Rules

Claude Code는 Repository의
Branch 전략과 GitHub Ruleset을 준수합니다.

### Protected Branch

다음 Branch에는 직접 Push하지 않습니다.

```text
main
develop
```

보호 Branch에서 직접 Commit하지 않습니다.

Issue 전용 Branch에서 작업합니다.

### Destructive Commands

다음 명령 또는 동등한 효과를 가지는 작업을 수행하지 않습니다.

```text
git reset --hard
git clean -fd
git checkout -- .
git restore .
git push --force
git push --force-with-lease
```

Claude Code가 생성하지 않은 변경사항을
삭제하기 위한 Git 명령을 사용하지 않습니다.

### Commit

Commit에는 현재 Issue 관련 변경만 포함합니다.

기존 Commit Convention을 따릅니다.

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

### Pull Request

Claude Code가 PR을 작성하는 경우
루트 `AGENTS.md`의 Pull Request Rules를 기준으로 합니다.

PR에는 최소 다음 내용을 포함합니다.

- Related Issue
- 작업 내용
- 주요 변경 사항
- Test 결과
- 영향 범위
- Known Risks
- Human Review Required
- Agent 정보

Claude Code는 실제 실행한 Test만 기록하며,
실행하지 않은 Test를 수행한 것처럼 표시하지 않습니다.

Issue 전체를 완료하는 PR인 경우
Repository의 Issue Closing Convention을 따릅니다.

예:

```text
Closes #9
```

Issue 전체를 완료하지 않는 중간 PR인 경우
Issue를 자동 종료시키지 않습니다.

예:

```text
Refs #9
```

PR 작성과 Issue 연결 방식에 대한 세부 기준은
루트 `AGENTS.md`의 Pull Request Rules를 따릅니다.

Claude Code는 PR을 직접 Merge하지 않습니다.
Merge는 Human의 책임입니다.

---

## 13. Security / Secret Rules

Claude Code는 실제 Secret 값을 읽거나 출력하지 않습니다.

다음 값을 직접 확인하려고 시도하지 않습니다.

```text
AWS Access Key
AWS Secret Access Key
Database Password
JWT Secret
OAuth Client Secret
GitHub Token
Production Credential
Private Key
API Secret
```

`.env` 등 실제 Secret이 포함될 가능성이 높은 파일은
직접 열어 값을 확인하지 않습니다.

설정 구조 확인이 필요한 경우 다음을 우선 사용합니다.

```text
.env.example
Example Configuration
README
docs
```

실제 값 대신 Placeholder를 사용합니다.

예:

```text
GOOGLE_CLIENT_ID=<configured-by-human>
GOOGLE_CLIENT_SECRET=<configured-by-human>
```

실제 Secret 값이 필요한 작업은
STOP CONDITION으로 처리합니다.

---

## 14. STOP CONDITIONS

다음 상황에서는
Claude Code가 임의로 해결하지 않고 작업을 중지합니다.

- API Contract 변경 필요
- DB 핵심 모델 변경 필요
- Architecture 변경 필요
- Domain Policy 변경 필요
- 인증 / 인가 정책 변경 필요
- Transaction Boundary 변경 필요
- Concurrency 전략 변경 필요
- 새로운 외부 Dependency 필요
- Dependency Major Version Upgrade 필요
- Package Manager / Build Tool 변경 필요
- 수정 금지 범위 변경 필요
- Issue 범위 확장 필요
- Secret / Credential 접근 필요
- GitHub Secrets 변경 필요
- 운영 Infrastructure 접근 필요
- Protected Branch 권한 필요
- Merge 권한 필요
- Source of Truth 충돌
- Agent Rule 충돌
- 요구사항 부족으로 안전한 구현이 불가능한 경우

STOP CONDITION 발생 시:

```text
1. 관련 변경 중지
2. 임의 해결 금지
3. 안전하게 수행된 기존 작업 유지
4. 문제 위치 확인
5. 필요한 변경 정리
6. 영향 범위 정리
7. Human에게 보고
8. 추가 지시 대기
```

---

## 15. Human Approval

Claude Code는 다음 사항을
독자적으로 최종 결정하지 않습니다.

- Domain Policy 변경
- MVP Scope 변경
- API Contract 변경
- 핵심 Entity / Relation / Constraint 변경
- Architecture 변경
- 인증 / 인가 정책 변경
- Transaction 정책 변경
- Concurrency 전략 변경
- 새로운 Dependency 추가
- Dependency Major Version Upgrade
- Package Manager / Build Tool 변경
- 새로운 외부 Service 도입
- Secret 환경 변수 구조 변경
- Infrastructure 설계 변경
- Production Database Schema / 운영 정책 변경
- Source of Truth 충돌 해결
- Commit History Rewrite / Rebase

Claude Code는 필요한 경우
권장안과 예상 영향 범위를 Human에게 제시할 수 있습니다.

Human Approval이 필요한 변경은
승인 전에 실제 적용하지 않습니다.

단, Human Approval은
`AGENTS.md`의 NEVER Rules를 해제하거나 완화하지 않습니다.

NEVER Rules에 해당하는 행동은
Human Approval 여부와 관계없이
Claude Code가 직접 수행하지 않습니다.

따라서 Human Approval을 받았더라도
승인된 설계 또는 정책 변경과
Claude Code가 직접 수행할 수 있는 실행 작업의 범위를 구분합니다.

예를 들어 다음과 같은 NEVER 행동은
Human Approval을 받아도 Claude Code가 직접 수행하지 않습니다.

- `main` 또는 `develop` Branch 직접 Push
- Pull Request 직접 Merge
- `git push --force` 또는 `git push --force-with-lease`
- GitHub Ruleset 또는 Branch Protection 임의 변경
- GitHub Secrets 직접 생성, 수정 또는 삭제
- Production Database Data 또는 Schema 직접 변경
- Production 환경 직접 배포
- 운영 Resource 직접 생성, 수정, 삭제 또는 중지
- 실제 Secret 또는 Credential 값에 대한 접근, 출력 또는 노출

Human Approval 이후에도
실제 수행하려는 행동이 NEVER Rules에 해당하는지 먼저 확인하며,
해당하는 경우 직접 실행하지 않고 Human에게 필요한 후속 조치를 보고합니다.

---

## 16. Completion Report

Claude Code의 Completion Report 형식은
루트 `AGENTS.md`의 `## 13. Completion Report`를 최종 기준으로 사용합니다.

Claude Code는 별도의 축약된 Completion Report 형식을 사용하지 않습니다.

작업 종료 시
현재 상태에 따라 다음 Status 중 하나를 정확하게 사용합니다.

```text
SUCCESS
PARTIAL
BLOCKED
FAILED
```

Test Result는 루트 `AGENTS.md`에 정의된 다음 값을 사용합니다.

```text
PASS
FAILED
NOT_RUN
NOT_APPLICABLE
```

Claude Code는 다음 원칙을 지킵니다.

- 실행하지 않은 Test, Build, Lint 또는 Type Check를 실행한 것처럼 보고하지 않음
- 실패한 Test 또는 Build 결과를 숨기거나 PASS로 보고하지 않음
- 미충족 Acceptance Criteria가 있다면 명시
- Issue 범위 밖에서 발견했지만 수정하지 않은 문제를 명시
- STOP CONDITION 발생 여부와 중단 지점을 명시
- Human의 판단 또는 Review가 필요한 항목을 명시

Completion Report의 항목, Status 정의 및 Test Result 기준이
`CLAUDE.md`와 루트 `AGENTS.md` 사이에서 다르게 해석될 가능성이 있는 경우
루트 `AGENTS.md`의 `## 13. Completion Report`를 따릅니다.

---

## 17. Final Self-Check

작업 종료 전에 Claude Code는 다음을 확인합니다.

- [ ] 현재 Issue 범위 안에서만 수정했는가
- [ ] 수정 허용 범위를 준수했는가
- [ ] 수정 금지 범위를 침범하지 않았는가
- [ ] 기존 Working Tree 변경사항을 보존했는가
- [ ] Source of Truth를 준수했는가
- [ ] API Contract를 임의 변경하지 않았는가
- [ ] Architecture를 임의 변경하지 않았는가
- [ ] 필요한 Test를 실행했는가
- [ ] Test 실패를 숨기지 않았는가
- [ ] Secret을 읽거나 노출하지 않았는가
- [ ] 관련 없는 파일을 수정하지 않았는가
- [ ] STOP CONDITION 발생 여부를 확인했는가
- [ ] Completion Report를 작성했는가
- [ ] PR Merge를 수행하지 않았는가

하나라도 충족하지 못한 항목이 있다면
작업을 완료 상태로 보고하지 않습니다.