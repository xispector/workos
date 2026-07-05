# Data Model Draft

작성일: 2026-07-05

## 1. 목적

이 문서는 Personal Work OS의 핵심 데이터를 저장 가능한 형태로 정리한 초안이다.

아직 ORM이나 프레임워크에 종속되지 않는다. 다만 실제 개인용 MVP의 권장 데이터베이스는 Firebase Firestore다. 이 문서는 Firestore 컬렉션으로 옮기기 쉬운 개념 모델로 작성한다.

## 2. 설계 원칙

### 원칙 1. AI 판단 근거를 저장한다

이 제품은 AI 추천이 핵심이므로, 추천 결과만 저장하면 안 된다. 왜 추천했는지, 어떤 컨텍스트에서 추천했는지 저장해야 한다.

### 원칙 2. 수동 기록과 자동 기록을 구분한다

작업 로그는 신뢰도가 중요하다. 사용자가 직접 남긴 로그와 AI가 자동 생성한 최소 로그는 다르게 취급해야 한다.

### 원칙 3. 프로젝트 맥락을 태스크보다 상위에 둔다

Task는 Project 없이 판단하기 어렵다. 가능한 한 Task는 Project에 연결한다. 단, 아직 연결되지 않은 Task도 임시로 허용한다.

### 원칙 4. 현재 실행 컨텍스트를 추천과 함께 저장한다

같은 작업도 환경과 강도에 따라 추천 여부가 달라진다. Recommendation은 생성 당시 WorkContext를 함께 가진다.

## 3. Entity Overview

핵심 엔티티:

- Project
- Task
- CaptureItem
- WorkContext
- Recommendation
- WorkLog
- Artifact
- WeeklyPlan
- WeeklyProjectDecision
- GoalAdjustment
- AIQuestion
- UserPreference

## 3.1 Firestore Collection Draft

실제 MVP에서 권장하는 Firestore 컬렉션 초안:

- `projects`
- `tasks`
- `captureItems`
- `workContexts`
- `recommendations`
- `workLogs`
- `artifacts`
- `weeklyPlans`
- `weeklyProjectDecisions`
- `goalAdjustments`
- `aiQuestions`
- `userPreferences`
- `integrationSyncStatus`

MVP에서는 단일 사용자 앱이므로 모든 문서에 `userId`를 필드로 둘 수 있다. 초기 개인용만 고려한다면 `userId`는 고정값으로 시작할 수 있지만, Firebase Authentication을 붙이면 실제 사용자 id로 전환한다.

권장:

- 큰 객체는 독립 컬렉션으로 둔다.
- WorkLog처럼 누적되는 데이터는 별도 컬렉션으로 둔다.
- Project 문서 안에는 최근 요약과 현재 상태만 둔다.
- 긴 로그 목록을 Project 문서에 중첩 저장하지 않는다.

## 4. Project

### 설명

사용자의 자기주도적 일을 묶는 맥락 단위.

### 필드

- id
- name
- finalGoal
- statusStage
- currentContextSummary
- recentProgressSummary
- nextSmallOutcome
- nextTenMinuteAction
- blockerSummary
- targetDate
- createdAt
- updatedAt
- archivedAt

### statusStage 값

- waiting
- researching
- designing
- executing
- blocked
- completed

### 관계

- has many Tasks
- has many WorkLogs
- has many Artifacts
- has many GoalAdjustments
- belongs to many WeeklyPlans through WeeklyProjectDecision

## 5. Task

### 설명

사용자가 하거나 AI가 추천할 수 있는 작업 단위.

### 필드

- id
- projectId nullable
- sourceCaptureItemId nullable
- parentTaskId nullable
- title
- description nullable
- taskType
- status
- intensity
- requiredPlace nullable
- requiredTool nullable
- requiredEnvironment nullable
- isTenMinuteAction
- priorityHint nullable
- sizeHint nullable
- dueDate nullable
- createdBy
- createdAt
- updatedAt
- completedAt nullable

### taskType 값

- action
- project_log
- idea
- study_note
- contact_followup
- decision_needed
- admin
- unclear

### status 값

- candidate
- recommended
- in_progress
- completed
- deferred
- cancelled

### intensity 값

- light
- normal
- deep

### createdBy 값

- user
- ai

### 관계

- belongs to Project optional
- belongs to CaptureItem optional
- may belong to parent Task
- has many child Tasks
- has many Recommendations
- has many WorkLogs

## 6. CaptureItem

### 설명

사용자의 자유 입력 원본.

### 필드

- id
- rawText
- inputMode
- processedStatus
- inferredType nullable
- linkedProjectId nullable
- createdAt
- processedAt nullable

### inputMode 값

- text
- voice

### processedStatus 값

- unprocessed
- processing
- processed
- needs_clarification

### 관계

- may create Task
- may create WorkLog
- may create AIQuestion
- may link to Project

## 7. WorkContext

### 설명

현재 작업 추천 기준.

### 필드

- id
- place
- tool
- environment
- intensity
- isActive
- createdAt
- updatedAt

### place 값

- home
- outside
- moving

### tool 값

- laptop
- phone_only

### environment 값

- quiet
- distracting

### intensity 값

- light
- normal
- deep

### 사용 방식

MVP에서는 사용자가 직접 변경한다. 자동 추정은 하지 않는다.

## 8. Recommendation

### 설명

AI가 특정 WorkContext에서 제안한 작업.

### 필드

- id
- taskId
- projectId nullable
- workContextId
- title
- reason
- rank
- status
- generatedAt
- startedAt nullable
- dismissedAt nullable
- completedAt nullable

### status 값

- shown
- started
- dismissed
- completed

### 관계

- belongs to Task
- belongs to Project optional
- belongs to WorkContext
- may create WorkLog

## 9. WorkLog

### 설명

작업 완료 후 남는 로그.

### 필드

- id
- projectId nullable
- taskId nullable
- recommendationId nullable
- source
- trustLevel
- rawInput nullable
- summary
- workDone nullable
- stateChange nullable
- nextAction nullable
- blockers nullable
- createdAt

### source 값

- user_text
- user_voice
- auto_minimal
- ai_generated

### trustLevel 값

- high
- low

### 자동 최소 로그 예시

summary:

`자동 기록 - 수집 시스템 정리하기 작업을 완료로 표시함. 세부 로그 없음.`

### 관계

- belongs to Project optional
- belongs to Task optional
- belongs to Recommendation optional

## 10. Artifact

### 설명

프로젝트에서 실제로 남은 결과물.

### 필드

- id
- projectId
- name
- artifactType
- description nullable
- url nullable
- fileRef nullable
- sourceWorkLogId nullable
- createdAt
- updatedAt

### artifactType 값

- document
- screen
- code
- demo
- draft
- hypothesis
- note
- decision
- other

### 관계

- belongs to Project
- may belong to WorkLog

## 11. WeeklyPlan

### 설명

한 주의 프로젝트 판단 단위.

### 필드

- id
- weekStartDate
- weekEndDate
- summary nullable
- createdAt
- updatedAt

### 관계

- has many WeeklyProjectDecisions

## 12. WeeklyProjectDecision

### 설명

특정 주에 특정 프로젝트를 전진/유지/보류/막힘 중 무엇으로 둘지에 대한 판단.

### 필드

- id
- weeklyPlanId
- projectId
- role
- aiSuggestedRole nullable
- reason
- userAdjusted
- riskLevel nullable
- nextSmallOutcome nullable
- targetDate nullable
- createdAt
- updatedAt

### role 값

- advance
- maintain
- defer
- blocked

### riskLevel 값

- safe
- caution
- risk

## 13. GoalAdjustment

### 설명

목표일이나 다음 작은 결과물의 조정 기록.

### 필드

- id
- projectId
- previousTargetDate nullable
- newTargetDate nullable
- previousOutcome nullable
- newOutcome nullable
- reason
- trigger
- createdAt

### trigger 값

- overdue
- near_due_no_progress
- user_requested
- scope_increased
- blocked
- context_changed

### 관계

- belongs to Project

## 14. AIQuestion

### 설명

AI가 사용자에게 던진 좁은 맥락 질문.

### 필드

- id
- captureItemId nullable
- projectId nullable
- taskId nullable
- questionText
- questionType
- answerText nullable
- status
- createdAt
- answeredAt nullable

### questionType 값

- project_linking
- task_clarification
- goal_clarification
- next_action
- priority_context

### status 값

- pending
- answered
- skipped

## 15. UserPreference

### 설명

사용자의 장기 작업 패턴과 선호.

MVP에서는 설정값보다는 AI가 요약한 패턴 기록에 가깝다.

### 필드

- id
- preferenceType
- summary
- evidence
- confidence
- createdAt
- updatedAt

### preferenceType 값

- productive_pattern
- underestimation_pattern
- overestimation_pattern
- context_success_pattern
- post_meetup_disruption
- avoidance_pattern

### confidence 값

- low
- medium
- high

## 16. 추천 생성에 필요한 최소 데이터

Recommendation을 만들려면 최소 다음 정보가 필요하다.

- 활성 WorkContext
- 이번 주 WeeklyPlan
- Project의 nextSmallOutcome
- Project의 recentProgressSummary
- Task 후보

없어도 되는 정보:

- 정교한 예상 시간
- 캘린더 전체
- 자동 위치
- 고급 태그

## 17. 프로젝트 이해 업데이트 규칙

WorkLog가 생성되면 AI는 다음 업데이트 후보를 만든다.

- Project.currentContextSummary
- Project.recentProgressSummary
- Project.nextTenMinuteAction
- Project.blockerSummary
- Artifact 후보

높은 신뢰도 로그는 업데이트에 강하게 반영한다.

낮은 신뢰도 로그는 recent activity로만 약하게 반영한다.

## 18. MVP 저장 우선순위

반드시 저장:

- Project
- Task
- CaptureItem
- WorkContext
- Recommendation
- WorkLog

가능하면 저장:

- Artifact
- WeeklyPlan
- WeeklyProjectDecision

후순위:

- GoalAdjustment
- AIQuestion
- UserPreference

## 19. 다음 문서로 이어지는 결정

다음 문서는 `ai-contract-spec.md`다.

이 문서에서는 AI가 위 데이터를 입력받아 어떤 JSON을 반환해야 하는지 정의한다.
