# Persistence and Database Strategy

작성일: 2026-07-05

## 1. 목적

이 문서는 Personal Work OS의 데이터 저장 전략을 정의한다.

핵심 질문:

- MVP에서 데이터를 어디에 저장할 것인가?
- Google Sheets를 데이터 저장소로 쓸 수 있는가?
- 실제 프로덕션으로 갈 때 어떤 DB 구조가 필요한가?
- AI가 생성한 판단과 로그를 어떻게 안전하게 저장할 것인가?

## 2. 저장 전략 결론

MVP는 2단계 저장 전략을 권장한다.

### 1단계: 로컬/앱 DB 우선

앱 내부 데이터 모델을 기준으로 저장한다.

저장 대상:

- Project
- Task
- CaptureItem
- WorkContext
- Recommendation
- WorkLog

목적:

- 제품의 핵심 루프를 안정적으로 검증한다.
- Google Sheets/Calendar 연동 실패가 핵심 데이터에 영향을 주지 않게 한다.

### 2단계: Google Sheets/Calendar는 연동 대상으로 사용

Google Sheets는 백업, 리뷰, 외부 분석용 export 대상으로 사용한다.

Google Calendar는 목표일, 집중 블록, 일정 참조용으로 사용한다.

초기에는 Google Sheets를 주 데이터베이스로 쓰지 않는 것이 안전하다.

## 3. 왜 Google Sheets를 주 DB로 쓰지 않는가

Google Sheets API는 값 읽기/쓰기와 append를 지원하므로 MVP 실험에는 유용하다.

하지만 Personal Work OS의 핵심 데이터는 관계가 많다.

- Project와 Task
- Task와 Recommendation
- Recommendation과 WorkLog
- WorkLog와 Project 상태 업데이트
- WeeklyPlan과 Project 판단
- GoalAdjustment와 목표일 변경 이력

이 관계를 Sheets만으로 관리하면 다음 문제가 생긴다.

- 관계 무결성 관리가 어렵다.
- row 수정/삭제가 복잡하다.
- AI가 동시에 여러 객체를 업데이트할 때 충돌 위험이 있다.
- JSON 구조와 중첩 데이터를 다루기 어렵다.
- 나중에 앱 DB로 옮길 때 마이그레이션 비용이 생긴다.

따라서 Sheets는 "보조 저장소"로 보는 것이 좋다.

## 4. 저장소 옵션

### Option A. Browser Local Storage / IndexedDB

용도:

- 매우 초기 프로토타입
- 로그인 없는 개인용 데모

장점:

- 구현이 빠르다.
- 서버가 필요 없다.
- AI Studio 프로토타입에 적합할 수 있다.

단점:

- 기기 간 동기화가 어렵다.
- 데이터 유실 위험이 있다.
- 장기 운영에는 부족하다.

적합도:

- 첫 데모: 적합
- 실제 개인 운영: 제한적

### Option B. App Database

예:

- Firebase Firestore
- Supabase Postgres
- SQLite 기반 로컬 앱
- Postgres 계열 DB

장점:

- 관계 데이터 관리 가능
- 장기 운영 가능
- 외부 연동과 동기화에 유리
- 백업/마이그레이션 가능

단점:

- 초기 설정이 더 필요하다.
- 인증/보안 설계를 고려해야 한다.

적합도:

- 실제 MVP: 적합

### Option C. Google Sheets as Primary Store

장점:

- 사람이 직접 볼 수 있다.
- export/import가 쉽다.
- 빠른 실험에 좋다.

단점:

- 관계형 데이터 관리에 약하다.
- 동시성/무결성/권한 관리가 어렵다.
- 앱이 커질수록 한계가 뚜렷하다.

적합도:

- 로그 export: 적합
- 주 DB: 비추천

## 5. 권장 MVP 저장 구조

MVP에서는 다음 순서로 저장한다.

1. 앱 내부 DB에 원본 저장
2. AI 출력 결과 저장
3. 사용자 확인 여부 저장
4. 필요 시 Google Sheets로 export
5. 필요 시 Google Calendar에 event 생성

중요 원칙:

외부 연동은 원본 데이터의 source of truth가 아니다.

source of truth는 앱 DB다.

## 6. Source of Truth 정의

### 앱 DB가 원본인 데이터

- Project
- Task
- CaptureItem
- WorkLog
- Recommendation
- WeeklyPlan
- GoalAdjustment
- UserPreference

### Google Sheets가 보조 사본인 데이터

- WorkLog export
- WeeklyPlan snapshot
- Project summary export
- Recommendation history export

### Google Calendar가 표현하는 데이터

- 목표일
- 작업 블록
- 주간 계획 블록
- 리뷰 일정

Calendar event는 앱 데이터의 표현일 뿐, 앱 내부 목표일 기록을 대체하지 않는다.

## 7. 저장 이벤트별 처리

### CaptureItem 생성

저장:

- rawInput
- inputMode
- createdAt
- processedStatus

AI 처리 후 업데이트:

- inferredType
- linkedProjectId
- processedAt

### Task 생성

저장:

- projectId
- title
- taskType
- status
- intensity
- required context hints
- sourceCaptureItemId

### Recommendation 생성

저장:

- taskId
- projectId
- workContext snapshot
- reason
- rank
- generatedAt

Recommendation은 생성 당시 WorkContext를 보존해야 한다. 나중에 사용자가 환경을 바꿔도 과거 추천 이유가 깨지지 않게 하기 위해서다.

### WorkLog 생성

저장:

- trustLevel
- source
- rawInput
- summary
- workDone
- stateChange
- nextAction
- blockers

스킵 로그:

- trustLevel = low
- stateChange = null
- artifactCandidate = null

### Project 업데이트

Project 상태는 AI가 바로 덮어쓰지 않는다.

흐름:

1. WorkLog 생성
2. AI가 Project update candidate 생성
3. 중요한 변경이면 사용자 확인
4. 확인 후 Project 업데이트

중요한 변경:

- statusStage 변경
- finalGoal 변경
- targetDate 변경
- nextSmallOutcome 변경
- Artifact 추가

## 8. 동기화와 백업

### MVP 백업 전략

최소 백업:

- JSON export
- Google Sheets export

권장 export 단위:

- Projects
- Tasks
- WorkLogs
- WeeklyPlans
- GoalAdjustments

### Google Sheets export 목적

- 사람이 읽기 쉬운 백업
- 주간 리뷰
- 나중에 분석
- AI가 긴 맥락을 다시 읽기 위한 외부 자료

## 9. 데이터 무결성 규칙

필수 규칙:

- Project.name은 비어 있으면 안 된다.
- Task.title은 비어 있으면 안 된다.
- Recommendation.reason은 비어 있으면 안 된다.
- WorkLog.summary는 비어 있으면 안 된다.
- low trust WorkLog는 stateChange를 만들면 안 된다.
- GoalAdjustment.reason은 필수다.
- Calendar event 생성 여부와 앱 내부 목표일은 분리해서 저장한다.

## 10. 삭제와 보관

### 삭제보다 archive 우선

프로젝트와 로그는 AI 개인화에 쓰이므로 바로 삭제보다 archive를 기본으로 한다.

archive 대상:

- Project
- Task
- Recommendation

삭제 가능:

- 잘못 입력한 CaptureItem
- 민감한 로그
- 테스트 데이터

### 민감 정보

민감한 로그나 연락 정보는 사용자가 직접 삭제할 수 있어야 한다.

## 11. Google Sheets API 관련 참고

Google Sheets API는 spreadsheets.values를 통해 값 읽기/쓰기를 제공하고, append로 행을 추가할 수 있다.

공식 문서:

- https://developers.google.com/workspace/sheets/api/guides/values
- https://developers.google.com/workspace/sheets/api/samples/writing
- https://developers.google.com/workspace/sheets/api/reference/rest

## 12. 다음 결정

다음 문서에서는 Google Sheets와 Google Calendar를 구체적으로 어떤 용도로 연동할지 정의한다.

파일:

`google-integrations-strategy.md`

