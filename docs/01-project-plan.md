# KOKU Airline Renewal - Project Plan

## 1. 프로젝트 개요

KOKU Airline Renewal은 기존 항공 예약 프로젝트를 최신 기술 스택과 개선된 설계로 재구성하는 개인 풀스택 프로젝트입니다.

본 프로젝트는 한국 ↔ 일본 노선을 중심으로 하는 가상의 항공사 예약 서비스를 구현하며, 단순 CRUD 수준을 넘어 실제 서비스에서 발생할 수 있는 예약, 좌석 경쟁, 결제 흐름, 외부 항공편 조회, AI 기반 검색 기능까지 단계적으로 구현하는 것을 목표로 합니다.

또한 AI 에이전트를 단순 코드 생성 도구로 사용하는 것이 아니라, 구현자와 리뷰어 역할을 분리하고 CI 및 사람의 최종 승인을 포함한 개발 프로세스를 구성합니다.

### 1.1 문서 및 용어 표기 원칙

본 문서와 관련 설계 문서에서는 용어의 일관성을 위해 다음 표기 원칙을 사용합니다.

- 설명, 기능명 및 일반적인 비즈니스 개념은 한국어로 작성합니다.
- Role, Entity, 인증 객체, Enum 등 코드, ERD 및 API Contract와 직접 연결되는 개념은 영문 Canonical Term을 사용합니다.
- `Domain Policy`, `API Contract` 등 문서명 또는 명시적인 기술 개념은 해당 영문 명칭을 유지합니다.
- 일반적인 domain 개념은 `도메인`으로 표기합니다.
- 동일한 개념을 문서마다 서로 다른 용어로 혼용하지 않습니다.
- 주요 Canonical Term의 정의 및 변경은 `02-domain-policy.md`를 기준으로 하며,
  관련 설계 문서에서도 동일한 용어를 사용합니다.

---

## 2. 프로젝트 목적

### 2.1 기술적 목적

- Java 21과 Spring Boot 기반의 Backend 설계 및 구현
- React 기반의 Frontend 구현
- 주요 사용자 화면의 Desktop / Mobile Responsive Web 지원
- REST API 기반 Frontend / Backend 분리
- JPA를 활용한 도메인 및 데이터 모델링
- 예약 및 좌석 처리에서 발생하는 동시성 문제 해결
- 캐싱, 인덱스, 쿼리 개선을 통한 성능 최적화
- 외부 항공편 API 연동 경험 확보
- AI 기반 항공편 조회 및 추천 기능 구현
- Docker 및 AWS 기반 실행/배포 환경 구성
- GitHub Actions 기반 CI/CD 구축
- 단위/통합/동시성/부하 테스트 경험 확보

### 2.2 포트폴리오 목적

- 단순 기능 구현보다 설계 의사결정과 문제 해결 과정을 보여주는 프로젝트 구성
- 예약 시스템의 Transaction 및 동시성 문제 해결 경험 강조
- 성능 개선 전/후 결과를 수치로 비교
- AI Agent 기반 개발 프로세스와 Harness Engineering 경험 정리
- 요구사항 → 설계 → 구현 → 테스트 → 배포까지 전체 개발 Lifecycle 경험 정리
- 한국과 일본의 취업 포트폴리오 활용을 위한 한국어 / 일본어 서비스 지원

---

## 3. 서비스 정의

KOKU Airline은 한국과 일본을 연결하는 가상의 항공사입니다.

사용자는 KOKU Airline이 제공하는 항공편을 검색하고 좌석을 선택한 뒤 예약 및 Mock 결제를 진행할 수 있습니다.

또한 실제 항공 데이터를 제공하는 외부 API를 활용하여 실제 한국 ↔ 일본 항공편을 조회하고, AI를 통해 사용자의 자연어 조건에 적합한 항공편을 검색하거나 추천받을 수 있습니다.

### 3.1 내부 예약 서비스

내부 예약 서비스는 KOKU Airline의 가상 항공편 데이터를 사용합니다.

주요 기능:

- 일반 회원가입 및 로그인
- Google OAuth 2.0 기반 소셜 로그인
- 회원 정보 조회 및 `LOCAL` 비밀번호 변경
- 항공편 검색
- 탑승객 정보 입력
- 좌석 조회 및 선택
- 예약 생성
- Mock 결제
- 예약 조회
- 예약 취소
- Admin 항공편 및 운항 일정 관리
- SuperAdmin 핵심 Master Data 관리

### 3.2 실제 항공편 조회 서비스

실제 항공편 조회 기능은 외부 Flight API 데이터를 사용합니다.

실제 항공편 조회 결과는 KOKU Airline 내부 예약 및 운항 데이터와 명확하게 분리합니다.

외부 항공편 검색 결과는 내부 KOKU Airline 항공편과 동일한 Entity로 저장하지 않으며,
MVP에서는 영구적인 업무 데이터로 저장하지 않는 것을 기본 원칙으로 합니다.

필요한 경우 성능 및 외부 API 호출량을 고려하여 Cache를 적용할 수 있습니다.

외부 항공편 조회 결과에 대해서는 실제 예약이나 발권을 제공하지 않습니다.

주요 기능:

- 실제 한국 ↔ 일본 항공편 검색
- 출발지 / 목적지 / 날짜 기반 조회
- 가격 / 시간 / 경유 여부 등 조건 비교
- AI 기반 자연어 항공편 검색 및 추천

---

## 4. 주요 사용자

### 4.1 Guest

- 항공편 조회
- 실제 항공편 조회
- 회원가입
- 로그인

### 4.2 Member

- 항공편 조회
- 좌석 선택
- 예약
- Mock 결제
- 예약 조회
- 예약 취소
- 회원 정보 조회
- `LOCAL` AuthAccount를 보유한 경우 비밀번호 변경
- AI 항공편 검색 및 추천

### 4.3 Admin

- 운영 데이터 조회
- 공항 조회
- 노선 조회
- 항공기 및 좌석 구성 조회
- 항공편 관리
- 운항 일정 관리
- 예약 현황 조회

Admin은 공항, 노선, 항공기 및 좌석 구성 등의
핵심 Master Data를 생성·수정·비활성화할 수 없습니다.

### 4.4 SuperAdmin

- Admin의 모든 기능
- 공항 Master Data 관리
- 노선 Master Data 관리
- 항공기 및 좌석 구성 관리
- 고객 예약 강제 취소
- 기타 중요 운영 데이터 관리

---

## 5. 서비스 범위

### 5.1 지원 지역

본 프로젝트의 MVP는 한국 ↔ 일본 국제 노선으로 제한합니다.

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

MVP 지원 공항 및 노선 범위는 Domain Policy를 기준으로 합니다.

지원 범위를 변경해야 하는 경우 Domain Policy 변경 절차를 거쳐
관련 요구사항 및 설계 문서를 함께 수정합니다.

### 5.2 지원 언어

KOKU Airline Renewal은 한국과 일본의 취업 포트폴리오 및 서비스 이용 환경을 고려하여
MVP에서 한국어와 일본어를 모두 필수 지원합니다.

지원 언어:

- 한국어 (`ko`)
- 일본어 (`ja`)

한국어와 일본어 중 특정 언어를 보조 언어로 취급하지 않으며,
주요 사용자 기능은 두 언어에서 동일하게 사용할 수 있어야 합니다.

사용자는 서비스에서 표시 언어를 선택하고 전환할 수 있어야 합니다.

Frontend의 사용자 표시 문자열은 Component에 직접 Hard Coding하지 않고,
다국어 처리가 가능한 구조로 분리하여 관리합니다.

Backend의 Domain Model, API Field, Enum, Database 내부 값 및 식별자는
표시 언어와 분리하여 영문 기준으로 관리합니다.

구체적인 Locale 감지, 언어 선택 저장 방식 및 i18n 구현 방식은
UI Design과 System Design에서 정의합니다.

### 5.3 지원 화면 환경

KOKU Airline Renewal의 MVP는 Web 기반 서비스로 구현하며,
일반 사용자용 주요 화면은 Desktop Web과 Mobile Web을 모두 지원합니다.

Frontend는 Responsive Web을 기본으로 하며,
동일한 사용자 기능을 화면 크기에 따라 적절하게 재배치하여 제공합니다.

기본 UI 설계 기준은 다음과 같습니다.

- Desktop: `1440px`
- Mobile: `390px`

Guest와 Member가 사용하는 주요 Customer UI는
Desktop과 Mobile에서 동일한 기능 범위를 제공하는 것을 원칙으로 합니다.

주요 대상 화면:

- Home
- KOKU Flight 검색
- Flight 상세
- 로그인 / 회원가입
- Passenger 정보 입력
- Seat 선택
- 예약 확인
- Mock 결제
- 예약 완료
- My Page
- Reservation 목록 및 상세
- 외부 실제 항공편 조회
- AI 항공편 검색

Admin 및 SuperAdmin 관리 화면은
MVP에서 Desktop Web 사용성을 우선합니다.

Admin / SuperAdmin 화면의 Mobile 전용 최적화는
MVP 필수 범위에 포함하지 않습니다.

구체적인 Responsive Layout, Navigation 및 Component 배치 정책은
`03-ui-design.md`에서 정의합니다.

---

## 6. MVP 핵심 기능

### 회원

- 일반 회원가입
- 일반 로그인 / 로그아웃
- Google OAuth 2.0 기반 소셜 로그인
- Member와 AuthAccount 분리 관리
- 회원 정보 조회
- `LOCAL` AuthAccount 비밀번호 변경
- 회원 탈퇴

### 항공편

- 여행 유형 선택 (`ONE_WAY`, `ROUND_TRIP`)
- 출발 공항 / 도착 공항 선택
- 출발 날짜 선택
- 왕복 예약의 경우 귀국 날짜 선택
- 편도 / 왕복 항공편 검색
- 왕복의 경우 출국 Flight 선택 후 역방향 Route의 귀국 Flight 선택
- 항공편 상세 조회
- 사전에 정의된 고정 운임 규칙 적용
- 출국편 / 귀국편의 여정 역할, 출발 시간대 및 출발 요일에 따른 운임 산정
- Adult / Child / Infant별 Passenger 기본 운임 계산
- KOKU Flight 검색 단계에서는 `ECONOMY` SeatClass 기준 예상 운임 표시
- 실제 선택한 SeatClass를 반영하여 Reservation 시작 시 최종 결제 예정 금액 확정

### 좌석

- 항공편별 좌석 조회
- 좌석 선택
- 예약 가능한 좌석 확인
- SeatClass 지원
  - `ECONOMY`
  - `PREMIUM_ECONOMY`
  - `BUSINESS`
- Passenger별 선택 Seat의 SeatClass를 운임에 반영
- 동일 Reservation에서도 Passenger마다 서로 다른 SeatClass 선택 가능
- `ROUND_TRIP`에서는 동일 Passenger가 출국 / 귀국 Flight에서 서로 다른 SeatClass 선택 가능
- 예약 시작 성공 시 선택한 모든 좌석을 1시간 임시 점유(Hold)
- Hold 만료 시 Reservation 취소 및 HELD Seat 자동 반환

### 예약

- 편도 / 왕복 Reservation 지원
- 탑승객 정보 입력
- 왕복 Reservation에서는 동일 Passenger 구성을 출국 / 귀국 Flight에 공통 적용
- Flight별 Seat 선택
- 왕복의 경우 출국 / 귀국 Seat 전체를 All-or-Nothing으로 확보
- 예약 생성
- 예약 조회
- 정책에 따른 확정 예약 취소
- 왕복 Reservation의 부분 취소는 지원하지 않음
- 정상 취소 시 Mock 전액 환불
- 사용자 및 운영자용 고유 Reservation 번호 발급

### 결제

- Mock 결제
- 결제 성공 / 실패 / 취소 처리
- 하나의 예약당 최초 시도 포함 최대 3회 결제 시도
- 실패한 결제 시도 이력 유지
- 최대 결제 시도 실패 시 예약 취소 및 좌석 반환
- 확정 예약 취소 시 Mock 전액 환불 처리
- 결제 상태 관리

### 관리자

#### Admin

- 공항 조회
- 노선 조회
- 항공기 및 좌석 구성 조회
- 항공편 관리
- Flight별 Seat 운영 상태 관리
- 운항 일정 관리
- 예약 현황 조회

#### SuperAdmin

- Admin의 모든 기능
- 공항 Master Data 관리
- 노선 Master Data 관리
- 항공기 및 좌석 구성 관리
- 고객 예약 강제 취소

### 외부 항공편 조회

- 외부 Flight API 연동
- 한국 ↔ 일본 실제 항공편 검색

### AI

AI 항공편 검색은 인증된 Member에게만 제공합니다.

- 자연어 검색 조건 분석
- 실제 항공편 조회 Tool 호출
- 조건 기반 항공편 필터링
- 추천 항공편 설명

### 다국어 지원

- 한국어 UI 지원
- 일본어 UI 지원
- 한국어 / 일본어 언어 전환
- 사용자 표시 문자열의 다국어 분리 관리
- 두 언어에서 주요 사용자 흐름 동일 지원

---

## 7. MVP 제외 범위

다음 기능은 초기 MVP에서 구현하지 않습니다.

- 실제 항공권 발권
- 실제 항공사 예약 시스템 연동
- 실제 PG 결제
- 실제 PG를 이용한 금융 환불 처리
- 마일리지 시스템
- 쿠폰 시스템
- 잔여 Seat 수, Seat 점유율 및 수요 기반 Dynamic Pricing
- 다구간 항공권
- 코드셰어 처리
- 실제 체크인
- 탑승권 발급
- 수하물 관리
- 전 세계 노선 지원
- AI를 통한 직접 예약 또는 결제 실행

필요한 경우 프로젝트 완료 후 확장 기능으로 검토합니다.

---

## 8. 기능 요구사항

### FR-01 회원가입 및 인증

사용자는 이메일과 비밀번호를 이용하여 일반 회원가입 및 로그인을 할 수 있어야 합니다.

사용자는 Google OAuth 2.0을 이용하여 소셜 로그인을 할 수 있어야 합니다.

Google OAuth 로그인을 처음 이용하는 사용자의 경우 필요한 회원 정보를 기반으로
Member를 생성할 수 있어야 합니다.

일반 로그인과 Google OAuth 로그인 모두 동일한 Member 및 권한 체계에서 처리되어야 합니다.

`LOCAL` AuthAccount를 보유한 인증된 Member는
정책에 따라 비밀번호를 변경할 수 있어야 합니다.

비밀번호 변경 시 현재 비밀번호 재인증을 요구하며,
`GOOGLE` AuthAccount만 보유하고 `LOCAL` AuthAccount가 없는 Member에게는
LOCAL 비밀번호 변경 기능을 제공하지 않습니다.

Member는 정책에 따라 회원 탈퇴를 수행할 수 있어야 합니다.

탈퇴된 Member는 `WITHDRAWN` 상태로 관리하며 로그인할 수 없어야 합니다.

진행 중인 `PENDING` 예약 또는 향후 탑승 예정인 `CONFIRMED` 예약을 보유한 Member는
탈퇴할 수 없어야 합니다.

### FR-02 항공편 검색

사용자는 `ONE_WAY` 또는 `ROUND_TRIP` 여행 유형을 선택하고
출발지, 목적지 및 날짜를 기준으로 항공편을 검색할 수 있어야 합니다.

`ROUND_TRIP`에서는 출국 Flight를 먼저 선택한 후
출국 Route의 역방향 Route에 해당하는 귀국 Flight를 선택할 수 있어야 합니다.

### FR-03 좌석 조회

사용자는 선택한 항공편의 좌석 상태와 SeatClass를 확인할 수 있어야 합니다.

MVP에서 지원하는 SeatClass는 다음과 같습니다.

- `ECONOMY`
- `PREMIUM_ECONOMY`
- `BUSINESS`

Seat가 필요한 Passenger는
각 Flight에서 예약 가능한 Seat를 선택할 수 있어야 합니다.

동일 Reservation의 Passenger들은
서로 다른 SeatClass를 선택할 수 있어야 합니다.

`ROUND_TRIP`에서는 동일 Passenger도
출국 Flight와 귀국 Flight에서 서로 다른 SeatClass를 선택할 수 있어야 합니다.

### FR-04 예약

`ACTIVE` 상태의 인증된 Member는 유효한 항공편을 선택한 후
Passenger 정보를 입력할 수 있어야 합니다.

Passenger 정보 입력 및 예약 조건 검증이 완료되면
Seat가 필요한 모든 Passenger의 예약 가능한 좌석을 선택할 수 있어야 합니다.

선택한 모든 Seat를 확보할 수 있는 경우에만
`PENDING` 상태의 Reservation을 생성하고,
선택한 모든 Seat를 `HELD` 상태로 전환할 수 있어야 합니다.

Reservation 시작 시
각 Passenger와 Flight에서 실제로 선택한 Seat의 SeatClass를 반영하여
최종 결제 예정 금액을 확정할 수 있어야 합니다.

확정된 Reservation 금액은 이후 Payment 재시도 과정에서
다시 계산하거나 변경하지 않아야 합니다.

Seat 확보는 All-or-Nothing을 원칙으로 하며,
선택한 Seat 중 하나라도 확보하지 못한 경우 예약 시작 전체가 실패해야 합니다.

`ROUND_TRIP` Reservation은 하나의 Reservation으로 관리하며,
출국 Flight와 귀국 Flight를 포함합니다.

왕복 Reservation에서는 동일한 Passenger 구성을 두 Flight에 공통으로 적용하고,
Seat는 각 Flight별로 선택합니다.

출국 Flight와 귀국 Flight에서 선택한 모든 Seat를
확보할 수 있는 경우에만 Reservation을 시작할 수 있어야 합니다.

필요한 테스트용 여권 정보는 시스템에서 자동 생성하며,
Mock 결제가 정상적으로 완료된 경우 Reservation을
`CONFIRMED` 상태로 확정할 수 있어야 합니다.

### FR-05 결제

사용자는 예약에 대해 Mock 결제를 수행할 수 있어야 합니다.

### FR-06 예약 조회

사용자는 자신의 예약 내역을 조회할 수 있어야 합니다.

### FR-07 예약 취소

사용자는 정책에 따라 예약을 취소할 수 있어야 합니다.

### FR-08 관리자

Admin은 운영 목적의 공항, 노선, 항공기 및 좌석 구성 정보를 조회하고,
항공편 및 운항 일정을 관리할 수 있어야 합니다.

Admin과 SuperAdmin은
특정 Flight의 운영상 판매 가능한 Seat를 판매 불가 상태로 전환하거나
운영 제한이 해제된 Seat를 다시 판매 가능 상태로 복구할 수 있어야 합니다.

이미 Reservation에 사용 중인 Seat의 상태를
이 기능으로 임의 변경해서는 안 됩니다.

SuperAdmin은 Admin의 모든 권한을 포함하며,
공항, 노선, 항공기 및 좌석 구성 등의 핵심 Master Data를 관리할 수 있어야 합니다.

고객 예약 강제 취소는 SuperAdmin에게만 허용합니다.

### FR-09 실제 항공편 검색

사용자는 외부 API를 통해 실제 한국 ↔ 일본 항공편을 조회할 수 있어야 합니다.

### FR-10 AI 항공편 검색

인증된 Member는 자연어를 통해 항공편 검색 조건을 입력하고
외부 실제 항공편 데이터를 기반으로 적합한 항공편을 추천받을 수 있어야 합니다.

Guest는 AI 항공편 검색을 사용할 수 없습니다.

### FR-11 다국어 지원

사용자는 서비스의 표시 언어를 한국어 또는 일본어로 선택할 수 있어야 합니다.

MVP의 주요 사용자 기능은 한국어와 일본어 환경에서 동일하게 사용할 수 있어야 합니다.

언어를 변경하더라도 현재 사용자 상태 및 서비스 기능의 동작이 달라지지 않아야 합니다.

사용자에게 표시되는 UI Text, 안내 메시지 및 주요 오류 메시지는
한국어와 일본어를 모두 지원해야 합니다.

---

## 9. 비기능 요구사항

### NFR-01 데이터 무결성

동일한 좌석이 동시에 여러 예약에 확정되지 않도록 해야 합니다.

### NFR-02 Transaction

예약, 좌석, 결제와 같이 데이터 정합성이 중요한 기능은 적절한 Transaction 경계를 가져야 합니다.

### NFR-03 성능

항공편 검색 API의 주요 병목을 측정하고 필요한 경우 인덱스 및 캐싱을 적용합니다.

### NFR-04 테스트

핵심 비즈니스 로직에는 테스트를 작성합니다.

특히 예약 및 좌석 기능에는 동시성 테스트를 포함합니다.

### NFR-05 보안

- 비밀번호를 평문으로 저장하지 않습니다.
- Secret 및 API Key를 Repository에 저장하지 않습니다.
- 인증이 필요한 API는 적절한 접근 제어를 적용합니다.
- OAuth Client Secret은 환경변수 또는 Secret 관리 기능을 통해 관리합니다.
- OAuth 인증 결과를 신뢰할 수 있는 Provider를 통해 검증합니다.
- 일반 로그인과 OAuth 로그인의 인증 결과는 서비스 내부의 일관된 인증/인가 체계로 처리합니다.
- 인증 Token 및 사용자 개인정보가 Log에 노출되지 않도록 합니다.

### NFR-06 유지보수성

Backend와 Frontend의 책임을 명확하게 분리하고, 도메인 및 API Contract를 문서화합니다.

### NFR-07 AI 신뢰성

AI는 편명, 가격, 출발 시간 등의 실제 데이터를 임의 생성하지 않습니다.

실제 항공편 정보는 외부 API의 결과를 기준으로 사용합니다.

### NFR-08 Internationalization

서비스는 한국어와 일본어를 모두 필수 지원합니다.

Frontend의 사용자 표시 문자열은 다국어 확장이 가능한 구조로 관리하고,
특정 언어의 문자열을 Business Logic에 직접 의존시키지 않습니다.

Backend API, Domain Model, Enum 및 Database 내부 값은
표시 언어와 독립적으로 영문 기준으로 관리합니다.

한국어와 일본어 환경에서 동일한 기능 및 비즈니스 규칙이 적용되어야 합니다.

### NFR-09 Responsive Web

Guest와 Member가 사용하는 주요 사용자 화면은
Desktop 및 Mobile Web 환경에서 정상적으로 사용할 수 있어야 합니다.

Responsive Layout은 화면 크기에 따라 정보 구조를 임의로 변경하지 않고,
동일한 사용자 흐름과 기능을 유지하는 것을 원칙으로 합니다.

Desktop과 Mobile에서 다음 요소가 정상적으로 사용 가능해야 합니다.

- Navigation
- 검색 Form
- Flight 검색 결과
- Passenger 입력
- Seat 선택
- 예약 Step
- Mock Payment
- Reservation 조회 및 취소

Admin 및 SuperAdmin 관리 화면은
MVP에서 Desktop Web 사용성을 우선합니다.

---

## 10. 기술 스택

### Backend

- Java 21
- Spring Boot
- Spring Web
- Spring Data JPA
- Spring Security
- OAuth 2.0 Client
- JWT
- Bean Validation
- MySQL
- Gradle

### Frontend

- React
- Vite
- TypeScript
- REST API 기반 Backend 연동
- 한국어 / 일본어 Internationalization 지원

### Infrastructure

- Docker
- Docker Compose
- Redis
- AWS
- GitHub Actions

### AI / External API

- Spring AI
- 외부 Flight API

### 향후 검토

- Kafka
- k6 또는 기타 부하 테스트 도구

Redis는 Refresh Token의 Server-side 상태 및 TTL 관리를 위해
MVP 인증 Infrastructure에 포함합니다.

Cache 또는 Distributed Lock 등 인증 이외의 Redis 활용은
실제 기술적 필요성과 측정 결과가 확인된 경우에만 추가 적용합니다.

Kafka는 비동기 Event 처리가 실제로 필요한 경우 도입을 검토합니다.

불필요한 기술 사용을 피하고
기술 도입 전후의 문제와 개선 효과를 측정합니다.

---

## 11. 개발 방식

본 프로젝트는 GitHub Issue와 Pull Request 기반으로 개발합니다.

### 11.1 브랜치 전략

기본 브랜치 흐름은 다음과 같습니다.

- `main`: 배포 가능한 안정 브랜치
- `develop`: 개발 결과를 통합하는 기본 개발 브랜치
- `feature/*`: 기능 개발 및 기능 확장
- `fix/*`: 버그 및 오류 수정
- `refactor/*`: 기능 변경 없는 코드 구조 개선
- `docs/*`: 문서 작성 및 수정
- `chore/*`: 프로젝트 설정, 의존성, 빌드 등 기타 유지보수 작업

`ai`, `perf`, `infra`, `test` 등의 작업 성격은 GitHub Label로 구분하며,
브랜치 이름은 작업의 주된 변경 성격에 따라 위 Prefix 중 하나를 사용합니다.

기본 흐름:

`작업 브랜치 → Pull Request → develop → Pull Request → main`

### 11.2 Git 운영 원칙

- `main`과 `develop`에 직접 Push하지 않습니다.
- 모든 변경사항은 작업 브랜치에서 진행합니다.
- 작업은 GitHub Issue를 기준으로 진행합니다.
- 작업 완료 후 `develop`을 대상으로 Pull Request를 생성합니다.
- Pull Request는 기본적으로 Squash Merge를 사용합니다.
- 미해결 Review Conversation이 있는 경우 Merge하지 않습니다.
- Force Push를 사용하지 않습니다.
- 보호 브랜치를 삭제하지 않습니다.
- 최종 Merge 여부는 사람이 판단합니다.
- CI 구축 이후에는 필수 Status Check를 통과한 PR만 Merge합니다.

---

## 12. AI Agent 개발 방식

본 프로젝트에서는 AI Agent를 단순 코드 생성 도구가 아니라 구현 및 리뷰를 수행하는 개발 Agent로 활용합니다.

AI Agent의 작업 범위와 권한을 명확하게 제한하고, 요구사항과 주요 기술적 의사결정은 사람이 담당합니다.

### 12.1 역할 분담

#### Backend

- Implementer: Claude Code
- Reviewer: Codex

#### Frontend

- Implementer: Codex
- Reviewer: Claude Code

#### Human

사람은 다음 사항에 대한 최종 결정권을 가집니다.

- 프로젝트 요구사항
- MVP 범위
- 도메인 정책
- 사용자 시나리오
- 시스템 아키텍처
- ERD 및 데이터 모델
- API Contract
- 인증/인가 정책
- Transaction 전략
- 동시성 제어 전략
- 주요 외부 의존성 도입
- Infrastructure 및 배포 정책
- AI Agent 작업 결과 검토
- Pull Request 최종 Merge

### 12.2 AI Agent 기본 원칙

AI Agent는 다음 원칙을 준수합니다.

- Issue에 정의된 범위 안에서만 작업합니다.
- `main` 또는 `develop`에 직접 Push하지 않습니다.
- Pull Request를 직접 Merge하지 않습니다.
- 관련 없는 파일을 임의로 수정하지 않습니다.
- 테스트를 삭제하거나 비활성화하여 Build를 통과시키지 않습니다.
- Secret, API Key, Password 등의 민감정보를 Repository에 추가하지 않습니다.
- 요구사항을 임의로 확대하지 않습니다.
- Architecture 및 API Contract를 임의로 변경하지 않습니다.
- 중요한 설계 변경이 필요한 경우 구현을 중단하고 사람에게 보고합니다.

### 12.3 Implementer / Reviewer 분리

동일한 AI Agent가 구현과 최종 리뷰를 모두 담당하지 않는 것을 기본 원칙으로 합니다.

예시:

- Backend 구현: Claude Code
- Backend 리뷰: Codex
- Frontend 구현: Codex
- Frontend 리뷰: Claude Code

Reviewer는 구현자의 Branch를 직접 수정하지 않고, 문제점 및 개선사항을 Review 형태로 전달합니다.

수정이 필요한 경우 Implementer가 Review 내용을 반영합니다.

### 12.4 Human Gate

CI 통과 또는 AI Review 완료만으로 자동 Merge하지 않습니다.

최종 흐름은 다음과 같습니다.

1. Issue 작성
2. 작업 범위 및 Acceptance Criteria 확정
3. AI Implementer 작업
4. 테스트 및 Build
5. Pull Request 생성
6. CI 검증
7. 다른 AI Agent의 Review
8. Review 수정사항 반영
9. 사람의 최종 검토
10. Squash Merge

---

## 13. 개발 Milestone

### M1 - 설계 및 AI 개발환경 구축

목표:

구현 전에 프로젝트의 주요 설계를 확정하고 AI Agent가 안전하게 개발할 수 있는 환경을 구축합니다.

주요 작업:

- 프로젝트 기획 및 요구사항 정의
- 도메인 정책 정의
- 사용자 시나리오 설계
- ERD 및 데이터 모델 설계
- API Contract 설계
- 시스템 아키텍처 설계
- Desktop / Mobile 주요 UI 흐름 및 Wireframe 설계
- AI Harness 규칙 작성
- Spring Boot Backend 초기 구성
- React Frontend 초기 구성
- Docker 기반 Local 개발환경 구성
- GitHub Actions 기본 CI 구축
- AI Agent 개발 Workflow 검증

완료 기준:

AI Agent가 정의된 설계와 규칙을 기반으로 독립적인 작업을 수행하고, Issue → 구현 → PR → CI → AI Review → Human Review → Merge의 전체 개발 Cycle을 최소 한 번 성공적으로 수행합니다.

### M2 - 기본 기능 구현

주요 작업:

- 일반 회원가입
- 일반 로그인 / 로그아웃
- Google OAuth 2.0 소셜 로그인
- Spring Security 및 JWT 기반 인증/인가
- Access Token / Refresh Token 인증 흐름 구현
- Redis 기반 Refresh Token Server-side 관리
- `LOCAL` 비밀번호 변경 및 현재 비밀번호 재인증
- Member 및 AuthAccount 관리
- Admin 공항·노선·항공기/좌석 구성 조회
- Admin 항공편 및 운항 일정 관리
- SuperAdmin 공항·노선·항공기/좌석 구성 Master Data 관리
- 관리자 권한별 접근 제어
- 핵심 기능 단위 테스트
- Frontend 다국어(i18n) 기본 구조 구성
- 한국어 / 일본어 Locale 및 언어 전환 기능 구성

### M3 - 예약 시스템 구현

주요 작업:

- 한국 ↔ 일본 항공편 검색
- 항공편 상세 조회
- Domain Policy 기반 KOKU Airline 고정 운임 산정
- KOKU Flight 검색 시 `ECONOMY` 기준 예상 운임 제공
- SeatClass별 고정 운임 정책 적용
- Passenger / Flight별 SeatClass 선택
- PENDING Reservation 생성 시 실제 선택 SeatClass를 반영한 최종 운임 확정 및 보존
- 좌석 조회
- 좌석 선택
- 예약 생성
- 탑승객 정보 관리
- 예약 조회
- 예약 취소
- Mock 결제
- 실제 항공편 API 연동
- 예약 및 결제 통합 테스트
- 좌석 1시간 Hold 및 만료 처리
- Mock 결제 최대 3회 시도 및 실패 처리
- 예약 취소 및 Mock 전액 환불
- Reservation 번호 생성 및 중복 방지

### M4 - AI·성능·동시성 고도화

주요 작업:

- 좌석 예약 동시성 문제 분석
- 좌석 예약 동시성 제어
- 동시성 테스트
- 항공편 조회 성능 분석
- DB Index 최적화
- 필요한 영역에 Cache 적용
- 성능 개선 전/후 비교
- 부하 테스트
- AI 기반 항공편 조회 및 추천
- 핵심 도메인 내부 구조 리팩터링 (Domain Policy 변경 제외)

### M5 - 배포 및 마무리

주요 작업:

- AWS Infrastructure 구축
- 운영 환경 배포
- CD Pipeline 구축
- 전체 통합 테스트
- E2E 테스트
- Monitoring 및 Logging 환경 구성
- 최종 버그 수정
- README 작성
- Architecture 문서 정리
- 성능 개선 결과 정리
- 동시성 문제 해결 과정 정리
- Troubleshooting 문서화
- AI Agent 개발 방식 및 Harness Engineering 경험 정리
- 한국어 / 일본어 전체 사용자 흐름 및 UI 문구 검증

---

## 14. 프로젝트 완료 기준

다음 조건을 충족하면 프로젝트의 주요 개발이 완료된 것으로 판단합니다.

### 기능

- [ ] 일반 회원가입 및 로그인이 정상 동작합니다.
- [ ] Google OAuth 2.0 소셜 로그인이 정상 동작합니다.
- [ ] `LOCAL` AuthAccount를 보유한 Member가 현재 비밀번호 재인증 후 비밀번호를 변경할 수 있습니다.
- [ ] 일반 로그인과 Google OAuth 로그인이 동일한 Member 및 권한 체계로 처리됩니다.
- [ ] 한국 ↔ 일본 KOKU Airline 항공편을 검색할 수 있습니다.
- [ ] 편도 / 왕복 항공편을 검색할 수 있습니다.
- [ ] KOKU Airline Flight의 운임이 Domain Policy의 고정 운임 규칙에 따라 계산됩니다.
- [ ] Flight 검색 단계에서 `ECONOMY` SeatClass 기준 예상 운임이 제공됩니다.
- [ ] `ECONOMY`, `PREMIUM_ECONOMY`, `BUSINESS` SeatClass를 선택할 수 있습니다.
- [ ] Passenger / Flight별로 선택한 SeatClass가 최종 운임에 정상적으로 반영됩니다.
- [ ] PENDING Reservation 생성 시 최종 결제 예정 금액이 확정되고 이후 결제 재시도 과정에서 변경되지 않습니다.
- [ ] 왕복 Reservation에서 출국 / 귀국 Flight와 Seat가 하나의 Reservation으로 정상 처리됩니다.
- [ ] 왕복 Reservation의 전체 Mock 결제 및 전체 취소 정책이 정상 동작합니다.
- [ ] 항공편의 예약 가능한 좌석을 조회할 수 있습니다.
- [ ] 좌석을 선택하고 예약할 수 있습니다.
- [ ] 탑승객 정보를 등록할 수 있습니다.
- [ ] Mock 결제를 수행할 수 있습니다.
- [ ] 자신의 예약을 조회할 수 있습니다.
- [ ] 정책에 따라 예약을 취소할 수 있습니다.
- [ ] Admin과 SuperAdmin의 권한이 Domain Policy에 따라 구분되어 동작합니다.
- [ ] Admin은 항공편 및 운항 일정을 관리할 수 있습니다.
- [ ] Admin과 SuperAdmin은 정책에 따라 Flight별 Seat의 운영상 판매 가능 / 불가 상태를 관리할 수 있습니다.
- [ ] SuperAdmin은 핵심 Master Data를 관리할 수 있습니다.
- [ ] 선택된 좌석은 1시간 동안 Hold되고 만료 시 정상 반환됩니다.
- [ ] 하나의 예약에 대해 Mock 결제는 최대 3회까지만 시도할 수 있습니다.
- [ ] 결제 3회 실패 시 예약 취소 및 좌석 반환이 정상 처리됩니다.
- [ ] 편도 및 왕복 Reservation의 취소 가능 조건이 Domain Policy에 따라 정상 적용되며,
      정상 취소 시 Mock 전액 환불이 처리됩니다.
- [ ] 회원 탈퇴 시 Member가 `WITHDRAWN` 상태로 변경되고 더 이상 로그인할 수 없습니다.
- [ ] 진행 중이거나 향후 탑승 예정인 예약이 존재하는 Member의 탈퇴가 제한됩니다.
- [ ] PENDING Reservation 생성 시 고유한 Reservation 번호가 발급되고 중복되지 않습니다.

### 데이터 및 동시성

- [ ] 동일 좌석의 중복 확정 예약을 방지합니다.
- [ ] 좌석 예약 동시성 테스트가 작성되어 있습니다.
- [ ] 예약 및 결제 Transaction의 데이터 정합성이 검증되어 있습니다.

### 외부 데이터 및 AI

- [ ] 실제 항공편 API를 통해 한국 ↔ 일본 항공편을 조회할 수 있습니다.
- [ ] 내부 KOKU Airline 예약 데이터와 외부 실제 항공 데이터가 명확하게 구분되어 있습니다.
- [ ] 자연어 조건을 이용한 AI 항공편 검색 기능이 동작합니다.
- [ ] AI가 항공편 가격, 시간, 편명 등을 임의 생성하지 않도록 구성되어 있습니다.

### Internationalization

- [ ] 한국어 UI가 주요 사용자 흐름 전체에서 정상적으로 제공됩니다.
- [ ] 일본어 UI가 주요 사용자 흐름 전체에서 정상적으로 제공됩니다.
- [ ] 사용자가 한국어와 일본어 사이에서 표시 언어를 전환할 수 있습니다.
- [ ] 주요 안내 및 오류 메시지가 한국어와 일본어를 모두 지원합니다.
- [ ] 언어 변경에 따라 API 및 Business Logic의 동작이 달라지지 않습니다.

### Responsive Web

- [ ] 주요 Customer UI가 Desktop Web에서 정상적으로 동작합니다.
- [ ] 주요 Customer UI가 Mobile Web에서 정상적으로 동작합니다.
- [ ] Desktop과 Mobile에서 동일한 주요 사용자 기능을 사용할 수 있습니다.
- [ ] Mobile 환경에서 Navigation, 검색, 예약, Seat 선택 및 Mock 결제 흐름이 정상적으로 동작합니다.
- [ ] 한국어와 일본어 모두 Desktop / Mobile Layout에서 Text가 잘리거나 주요 UI가 깨지지 않습니다.
- [ ] Admin / SuperAdmin 관리 화면은 Desktop Web에서 정상적으로 사용할 수 있습니다.

### 성능

- [ ] 핵심 조회 API에 대해 성능을 측정합니다.
- [ ] 주요 병목 원인을 분석합니다.
- [ ] 필요한 Index 또는 Cache를 적용합니다.
- [ ] 성능 개선 전/후 결과를 기록합니다.
- [ ] 주요 기능에 대해 부하 테스트를 수행합니다.

### 테스트 및 품질

- [ ] 핵심 비즈니스 로직에 테스트가 작성되어 있습니다.
- [ ] 주요 통합 테스트가 통과합니다.
- [ ] CI가 정상 동작합니다.
- [ ] 배포된 서비스의 주요 사용자 흐름이 정상 동작합니다.

### 배포 및 문서

- [ ] AWS에 서비스를 배포합니다.
- [ ] CD Pipeline을 구성합니다.
- [ ] README를 최종 작성합니다.
- [ ] ERD 및 Architecture를 문서화합니다.
- [ ] 주요 Troubleshooting을 기록합니다.
- [ ] 동시성 및 성능 개선 과정을 문서화합니다.
- [ ] AI Agent 활용 방식과 사람의 역할을 문서화합니다.

---

## 15. 프로젝트 개발 원칙

본 프로젝트는 기능의 수를 늘리는 것보다 완성도와 기술적 근거를 우선합니다.

### 15.1 명확한 요구사항

구현 전에 기능의 목적, 범위 및 완료 조건을 정의합니다.

AI Agent가 요구사항을 임의로 해석하여 기능을 확대하지 않도록 합니다.

### 15.2 명확한 도메인 정책

예약, 좌석, 결제 등 핵심 도메인의 규칙은 구현 코드보다 먼저 정의합니다.

### 15.3 데이터 정합성

예약 시스템에서 데이터 정합성을 최우선으로 고려합니다.

특히 좌석과 예약 데이터의 일관성을 보장합니다.

### 15.4 Transaction 설계

단순히 `@Transactional`을 사용하는 것에 그치지 않고 Transaction Boundary와 실패 시 데이터 상태를 고려합니다.

### 15.5 동시성 문제 해결

동시에 동일 좌석을 예약하는 상황을 재현하고, 테스트를 통해 해결 방법을 검증합니다.

### 15.6 측정 기반 성능 개선

추측만으로 최적화하지 않습니다.

성능을 측정하고 병목을 확인한 후 Index, Query 개선, Cache 등의 기술을 적용합니다.

적용 전후의 결과를 비교하여 기록합니다.

### 15.7 테스트 가능한 구조

핵심 비즈니스 로직이 자동화된 테스트를 통해 검증될 수 있도록 설계합니다.

### 15.8 최소한의 기술 도입

포트폴리오를 위해 기술을 무조건 추가하지 않습니다.

Redis는 Refresh Token의 Server-side 상태 및 TTL 관리라는
명확한 인증 요구사항을 위해 사용합니다.

이미 도입된 Redis를 Cache 또는 Distributed Lock 등
다른 목적으로 확대 적용하는 것은
실제 문제 또는 측정 가능한 필요성이 존재할 때만 결정합니다.

Kafka 등 추가 Infrastructure 기술 역시
실제 문제 또는 요구사항이 존재할 때 도입합니다.

### 15.9 AI 결과에 대한 Human Ownership

AI Agent가 작성한 코드라도 최종 책임은 사람에게 있습니다.

사람이 이해하지 못하는 코드나 설계는 그대로 Merge하지 않습니다.

### 15.10 안전한 AI 개발 환경

AI의 자율성을 높이는 것보다 잘못된 작업을 안전하게 차단할 수 있는 개발 환경을 우선합니다.

Ruleset, CI, Branch Policy, Agent Rule 및 Human Review를 통해 AI Agent의 작업 범위를 통제합니다.

### 15.11 MVP 우선

기능을 계속 추가하기보다 정해진 MVP를 완성하는 것을 우선합니다.

새로운 기능은 기존 핵심 기능의 완성도, 테스트 및 배포가 충분히 확보된 이후 검토합니다.