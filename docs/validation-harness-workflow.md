# Validation Harness Workflow

작성일: 2026-07-05

## 1. 목적

이 문서는 AI Studio 또는 다른 AI 코딩 도구가 Personal Work OS 프로토타입을 만들었을 때, 결과물이 제품 의도대로 동작하는지 검증하기 위한 하네스와 워크플로우를 정의한다.

여기서 하네스는 단순 테스트 코드만 의미하지 않는다.

포함 범위:

- 제품 요구사항 체크리스트
- AI 출력 계약 검증
- 핵심 사용자 시나리오 검증
- 데이터 무결성 검증
- 외부 연동 검증
- 회귀 테스트 시나리오

## 2. 검증 철학

이 제품은 일반 CRUD 앱이 아니라 AI 판단이 들어가는 Work OS다.

따라서 검증은 "버튼이 눌리는가"만으로 부족하다.

검증해야 할 것:

- AI가 사용자의 입력을 적절한 프로젝트에 연결하는가
- 큰 태스크를 현재 환경에서 가능한 작업으로 바꾸는가
- 추천 이유가 납득 가능한가
- 자동 로그를 과장하지 않는가
- 프로젝트 맥락이 시간이 지날수록 쌓이는가
- 목표일 조정 이유가 기록되는가

## 3. 검증 레벨

검증은 5단계로 나눈다.

1. Static Spec Check
2. AI Contract Check
3. User Flow Scenario Check
4. Data Integrity Check
5. Integration Check

## 4. Static Spec Check

### 목적

프로토타입이 문서에서 정의한 핵심 화면과 구조를 갖고 있는지 확인한다.

### 체크리스트

오늘 작업 화면:

- 첫 화면이 Today Work Screen인가
- 상단 실행 컨텍스트 바가 있는가
- 추천 작업 3개가 중심에 있는가
- 각 추천 작업에 이유가 있는가
- 빠른 수집 입력이 있는가
- 이번 주 전진 프로젝트 요약이 있는가

프로젝트 상세 화면:

- 최종 목표가 보이는가
- 상태 단계가 보이는가
- 현재 맥락 한 문장이 보이는가
- 결과물 목록이 보이는가
- 최근 진척 요약이 보이는가
- 다음 작은 결과물이 보이는가
- 다음 10분 행동이 보이는가
- 최근 로그가 보이는가

주간 판단 화면:

- 전진/유지/보류/막힘 그룹이 있는가
- 각 프로젝트의 AI 제안 이유가 보이는가
- 사용자가 제안을 수정할 수 있는가

작업 완료:

- 완료 후 한 줄 로그 입력이 가능한가
- 스킵이 가능한가
- 스킵 시 자동 최소 로그가 생성되는가

## 5. AI Contract Check

### 목적

AI가 `ai-contract-spec.md`에서 정의한 입력/출력 구조를 지키는지 검증한다.

### 검증 대상 함수

- classifyCaptureItem
- linkProject
- generateClarifyingQuestion
- breakDownTask
- generateRecommendations
- summarizeWorkLog
- createAutoMinimalLog
- updateProjectUnderstanding
- generateWeeklyDecisions
- recalculateTargetDate

### 계약 검증 기준

모든 AI 응답은 다음을 만족해야 한다.

- JSON 파싱 가능
- 필수 필드 존재
- enum 값이 정의된 범위 안에 있음
- reason이 필요한 응답에는 reason 존재
- confidence가 필요한 응답에는 confidence 존재
- 질문이 필요한 경우 question이 하나만 존재
- 추천 작업에는 reason이 존재

### 실패 예시

실패:

```json
{
  "recommendations": [
    {
      "title": "수집 시스템 정리하기"
    }
  ]
}
```

이유:

- taskId 없음
- projectId 없음
- reason 없음
- intensity 없음

## 6. Golden Scenario Tests

### 목적

반드시 성공해야 하는 대표 사용자 흐름을 고정 테스트로 만든다.

### Scenario 1. 해야 할 일 수집 후 추천 생성

입력:

```text
수집 시스템 정리해야 함
```

기대 결과:

- CaptureItem 생성
- Personal Work OS 프로젝트에 연결
- Task 생성
- 태스크가 3~5개 이하로 분해
- 현재 WorkContext에 맞는 다음 행동 생성
- Today Work Screen 추천 후보에 표시
- 추천 이유 포함

### Scenario 2. 프로젝트 연결이 불확실한 입력

입력:

```text
인증 흐름 다시 봐야 함
```

기대 결과:

- AI가 바로 후보 5개를 보여주지 않음
- 좁은 질문 하나를 생성
- 질문 예: "이건 지금 만들고 있는 앱 프로젝트를 이어가는 일이야?"

### Scenario 3. 휴대폰/산만함/가볍게 컨텍스트

WorkContext:

```json
{
  "place": "moving",
  "tool": "phone_only",
  "environment": "distracting",
  "intensity": "light"
}
```

기대 결과:

- 긴 문서 작성 추천 금지
- 구현/설계 결정 추천 금지
- 음성 로그, 짧은 메모, 연락 초안 같은 작업 추천
- 추천 이유에 현재 컨텍스트 반영

### Scenario 4. 작업 완료 후 수동 로그

입력 로그:

```text
수집 입력 예시를 정리했고, 다음에는 AI 질문 규칙을 더 구체화하면 될 것 같아.
```

기대 결과:

- WorkLog trustLevel = high
- workDone 추출
- nextAction 추출
- Project recentProgressSummary 업데이트 후보 생성

### Scenario 5. 작업 완료 후 스킵

조건:

- 사용자가 완료 후 로그 입력을 스킵

기대 결과:

- low trust 자동 로그 생성
- 문구가 과장되지 않음
- 프로젝트 상태를 크게 바꾸지 않음

예:

```text
자동 기록 - 수집 입력 예시 3개 작성 작업을 완료로 표시함. 세부 로그 없음.
```

### Scenario 6. 목표일 재계산

조건:

- 목표일 3일 전
- 최근 높은 신뢰도 로그 없음

기대 결과:

- 새 목표일 제안
- 변경 이유 포함
- 조정하지 않을 때 위험 포함
- 사용자 확정 전까지 적용하지 않음

## 7. Data Integrity Check

### 목적

데이터가 연결성을 잃지 않고 축적되는지 확인한다.

### 검증 규칙

Project:

- name은 필수
- finalGoal은 프로젝트 등록 시 필수
- statusStage는 정의된 enum 안에 있어야 함

Task:

- title은 필수
- status는 정의된 enum 안에 있어야 함
- projectId가 없으면 unlinked 상태로 처리해야 함

Recommendation:

- taskId 필수
- workContextId 필수
- reason 필수
- rank 필수

WorkLog:

- trustLevel 필수
- summary 필수
- low trust 로그는 stateChange를 비워야 함
- low trust 로그는 Artifact를 자동 생성하면 안 됨

GoalAdjustment:

- reason 필수
- previousTargetDate 또는 previousOutcome 중 하나 이상 존재해야 함

## 8. AI Safety / Trust Harness

### 목적

AI가 사용자의 프로젝트 맥락을 오염시키거나 과장하지 않도록 방지한다.

### 검증 규칙

AI는 다음을 하면 안 된다.

- 자동 로그만으로 결과물 완료를 선언
- 낮은 신뢰도 로그만으로 프로젝트 상태를 `완료`로 변경
- 사용자가 확인하지 않은 목표일 변경을 확정
- 추천 이유 없이 작업을 추천
- 불확실한 프로젝트 연결을 확정처럼 표시
- 선택지를 4개 이상 나열하는 맥락 질문 생성

### 신뢰도 기반 반영 규칙

높은 신뢰도 로그:

- 최근 진척 요약에 강하게 반영 가능
- 다음 행동 업데이트 가능
- 상태 변화 후보 생성 가능

낮은 신뢰도 로그:

- 최근 작업 여부에만 반영
- 프로젝트 상태를 크게 바꾸지 않음
- 목표일 조정의 약한 보조 신호로만 사용

## 9. Integration Check

외부 연동은 MVP에서 선택적이다. 다만 설계상 검증 기준은 미리 둔다.

### Google Sheets

검증:

- WorkLog를 한 행으로 append할 수 있는가
- Project 목록을 export할 수 있는가
- Recommendation 히스토리를 export할 수 있는가
- API 실패 시 로컬 데이터가 유실되지 않는가

### Google Calendar

검증:

- 목표일을 Calendar event로 생성할 수 있는가
- 전진 프로젝트 집중 블록을 event로 생성할 수 있는가
- Calendar event 생성 전 사용자 확인이 있는가
- Calendar API 실패 시 앱 상태가 깨지지 않는가

## 10. Manual Review Checklist

AI Studio가 만든 프로토타입을 사람이 검토할 때 사용할 체크리스트:

- 첫 화면에서 해야 할 작업이 가장 먼저 보이는가
- 사용자가 태그/우선순위/예상 시간을 직접 많이 입력하지 않아도 되는가
- AI 질문이 한 번에 하나인가
- 추천 이유가 짧고 납득 가능한가
- 현재 실행 컨텍스트를 바꾸면 추천이 달라지는가
- 프로젝트 상세에서 목표와 최근 진척을 이해할 수 있는가
- 작업 완료 후 로그가 부담스럽지 않은가
- 스킵해도 자동 최소 로그가 안전하게 남는가
- 주간 판단에서 보류 이유가 신뢰 가능한가

## 11. Regression Test Set

제품을 수정할 때마다 다시 확인해야 하는 회귀 테스트:

1. 빠른 수집 입력이 깨지지 않는가
2. 프로젝트 연결이 동작하는가
3. 맥락 질문이 과도하게 늘어나지 않았는가
4. 추천 작업에 이유가 유지되는가
5. low trust 로그가 과장되지 않는가
6. WorkContext 변경이 추천에 반영되는가
7. 주간 판단 근거가 사라지지 않았는가
8. 목표일 조정 이유가 저장되는가

## 12. 공식 문서 근거

Google AI Studio는 웹 앱 빌드를 지원하며, Gemini API는 function calling으로 외부 API와 도구 호출을 연결할 수 있다. Google Calendar API는 이벤트 생성/조회가 가능하고, Google Sheets API는 값 읽기/쓰기 및 append를 지원한다.

참고:

- https://ai.google.dev/gemini-api/docs/aistudio-build-mode
- https://ai.google.dev/gemini-api/docs/function-calling
- https://developers.google.com/workspace/calendar/api/guides/create-events
- https://developers.google.com/workspace/calendar/api/v3/reference/events/list
- https://developers.google.com/workspace/sheets/api/guides/values
- https://developers.google.com/workspace/sheets/api/samples/writing

