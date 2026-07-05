# Documentation Roadmap

작성일: 2026-07-05

## 1. 문서화 원칙

이 프로젝트의 문서는 실제 프로덕트를 만들 수 있을 때까지 단계적으로 구체화한다.

사용자가 제품을 아직 사용해보지 않았으므로, 세부 UI 구성이나 버튼 개수 같은 것은 매번 묻지 않는다. 그런 결정은 관련 앱 레퍼런스, 생산성 이론, 인지부하 관점, AI UX 원칙을 바탕으로 기본안을 제안한다.

사용자에게 물어볼 질문은 방향성 중심으로 제한한다.

물어볼 질문 예:

- 이 제품은 개인용인가, SaaS인가?
- AI가 얼마나 자동화해도 되는가?
- 사용자가 더 신뢰하는 판단 근거는 무엇인가?
- MVP에서 가장 먼저 검증하고 싶은 가치는 무엇인가?

묻지 않을 질문 예:

- 버튼은 왼쪽인가 오른쪽인가?
- 카드 여백은 몇 px인가?
- 추천 작업을 정확히 몇 줄로 보여줄 것인가?
- 상세 필터 이름을 무엇으로 할 것인가?

## 2. 현재 문서 목록

### Discovery / Direction

- `work-os-architecture-notes.md`
  - 인터뷰 기반 전체 구조 노트

- `research-backed-product-workflow.md`
  - 생산성 앱 조사와 기본 UX 방향 결정

- `product-brief.md`
  - 제품 목적, 대상 사용자, 문제, 원칙, MVP 성공 기준

### Product Spec

- `mvp-functional-spec.md`
  - MVP 화면과 기능 범위

- `user-flows.md`
  - 주요 사용자 흐름

- `information-architecture.md`
  - 핵심 객체, 객체 관계, 화면별 정보 구조

- `data-model-draft.md`
  - 저장 가능한 개념 모델 초안

- `project-dashboard-implementation-guidelines.md`
  - 프로젝트 대시보드 구성과 구현 지침

### AI Spec

- `ai-behavior-spec.md`
  - AI의 질문, 분류, 추천, 로그 처리 규칙

- `ai-contract-spec.md`
  - AI 입력/출력 JSON 구조와 함수 계약

- `google-ai-studio-prototype-prompt.md`
  - Google AI Studio에 넣을 수 있는 앱 생성용 프롬프트

### Validation / Integration

- `validation-harness-workflow.md`
  - AI가 만든 프로토타입이 제품 의도대로 동작하는지 검증하는 하네스와 시나리오

- `persistence-and-database-strategy.md`
  - Firebase Firestore, Google Sheets, Google Calendar의 역할과 source of truth 전략

- `google-integrations-strategy.md`
  - Google Sheets/Calendar/Gemini function calling 연동 전략

## 3. 다음에 작성할 문서

### 1. Acceptance Criteria

파일명:

`acceptance-criteria.md`

내용:

- MVP가 제대로 동작한다고 볼 수 있는 테스트 가능한 기준
- 화면별 통과 기준
- AI 동작별 통과 기준
- Google AI Studio 프로토타입 리뷰 체크리스트

### 2. Implementation Plan

파일명:

`implementation-plan.md`

내용:

- MVP를 어떤 순서로 구현할지
- 첫 프로토타입 범위
- 데이터 저장 단계
- AI function calling 적용 단계
- Google Sheets/Calendar 연동 단계

### 3. Prototype Review Report Template

파일명:

`prototype-review-report-template.md`

내용:

- AI Studio 결과물을 검토할 때 사용할 리뷰 양식
- 통과/수정 필요/보류 항목
- 다음 프롬프트 개선 사항

### 4. Production Hardening Checklist

파일명:

`production-hardening-checklist.md`

내용:

- 인증
- 백업
- 데이터 삭제
- Google API 실패 처리
- 로그/모니터링
- 보안과 민감 정보

## 4. 결정 방식

앞으로 세부 구성은 다음 기준으로 결정한다.

1. 사용자의 기존 문제를 직접 해결하는가
2. 입력 부담을 줄이는가
3. 프로젝트 맥락을 더 잘 쌓는가
4. 지금 실행 가능한 작업으로 이어지는가
5. AI의 판단 근거가 사용자에게 보이는가
6. MVP에서 구현 가능한가
7. 검증 가능한가
8. 데이터 손실과 AI 과장을 방지하는가

## 5. 다음 작업

다음 작업은 `acceptance-criteria.md`를 작성하는 것이다.

이 문서가 있어야 AI Studio에서 만든 결과물이 실제로 쓸 만한지 일관되게 평가할 수 있다.
