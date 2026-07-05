# Google Integrations Strategy

작성일: 2026-07-05

## 1. 목적

이 문서는 Personal Work OS가 Google Sheets, Google Calendar, Google AI Studio/Gemini function calling과 어떻게 연동할지 정의한다.

핵심 원칙:

앱 DB가 원본이고, Google 서비스는 보조 연동이다.

## 2. 공식 기능 근거

현재 공식 문서 기준:

- Google AI Studio는 웹 앱 빌드 모드를 지원한다.
- Gemini API는 function calling을 통해 외부 API와 도구 호출을 연결할 수 있다.
- Google Sheets API는 값 읽기/쓰기와 append를 지원한다.
- Google Calendar API는 이벤트 생성, 조회, 동기화 등을 지원한다.

참고:

- https://ai.google.dev/gemini-api/docs/aistudio-build-mode
- https://ai.google.dev/gemini-api/docs/function-calling
- https://ai.google.dev/gemini-api/docs/tools
- https://developers.google.com/workspace/sheets/api/guides/values
- https://developers.google.com/workspace/sheets/api/samples/writing
- https://developers.google.com/workspace/calendar/api/guides/create-events
- https://developers.google.com/workspace/calendar/api/v3/reference/events/list

## 3. 연동 우선순위

### MVP 필수

없음.

MVP 핵심 루프는 Google Sheets/Calendar 없이도 동작해야 한다.

### MVP 이후 1순위

- Google Sheets export
- Google Calendar 목표일/작업 블록 생성

### MVP 이후 2순위

- Google Calendar 일정 조회를 통한 주간 계획 보조
- Google Sheets 기반 주간 리뷰 리포트

### 후순위

- 양방향 동기화
- 자동 캘린더 스케줄링
- Sheets를 통한 직접 편집 후 앱 반영

## 4. Google Sheets 연동

### 역할

Google Sheets는 주 DB가 아니다.

용도:

- 백업
- 로그 export
- 주간 리뷰
- 수동 분석
- 외부에서 읽기 쉬운 데이터 사본

### Export 대상

#### WorkLogs 시트

컬럼:

- logId
- createdAt
- projectName
- taskTitle
- trustLevel
- source
- summary
- workDone
- nextAction
- blockers

#### Projects 시트

컬럼:

- projectId
- name
- finalGoal
- statusStage
- currentContextSummary
- recentProgressSummary
- nextSmallOutcome
- nextTenMinuteAction
- targetDate

#### WeeklyPlans 시트

컬럼:

- weekStartDate
- projectName
- role
- reason
- riskLevel
- nextSmallOutcome
- targetDate

#### GoalAdjustments 시트

컬럼:

- projectName
- previousTargetDate
- newTargetDate
- reason
- trigger
- createdAt

### Sheets 쓰기 방식

초기 연동은 append-only를 기본으로 한다.

이유:

- 구현이 단순하다.
- 변경 이력을 보존한다.
- 동기화 충돌이 적다.
- AI가 잘못 덮어쓰는 위험이 낮다.

Google Sheets API는 values append를 지원한다.

### Sheets 연동 실패 처리

원칙:

Sheets export 실패는 앱 핵심 데이터 저장 실패가 아니다.

처리:

1. 앱 DB에 먼저 저장
2. Sheets export 시도
3. 실패하면 syncStatus = failed
4. 나중에 재시도 가능

사용자에게 보여줄 문구:

```text
앱에는 저장됐지만 Google Sheets 내보내기는 실패했어. 나중에 다시 시도할 수 있어.
```

## 5. Google Calendar 연동

### 역할

Calendar는 시간 약속과 목표일을 표현하는 도구다.

앱 내부 목표일과 계획이 원본이고, Calendar event는 외부 표현이다.

### 연동 대상

#### 1. 작은 결과물 목표일

Project.nextSmallOutcome과 targetDate를 Calendar event로 만들 수 있다.

이벤트 예:

```text
[Work OS] 수집 시스템 MVP 명세 목표일
```

#### 2. 집중 작업 블록

사용자가 원할 때 추천 작업 또는 전진 프로젝트를 Calendar block으로 만들 수 있다.

이벤트 예:

```text
Deep Work: Personal Work OS - 수집 시스템 명세
```

#### 3. 주간 리뷰 일정

매주 프로젝트 상태를 보는 리뷰 이벤트를 만들 수 있다.

이벤트 예:

```text
Weekly Work OS Review
```

### Calendar에 바로 넣지 말아야 할 것

- 모든 추천 작업
- 모든 10분 행동
- 가벼운 작업
- AI가 확신 낮게 만든 작업

이유:

Calendar가 너무 복잡해지면 오히려 부담이 된다.

### Calendar event 생성 전 확인

Calendar event 생성은 항상 사용자 확인이 필요하다.

AI가 자동으로 Calendar를 채우면 신뢰 문제가 생길 수 있다.

확인 문구 예:

```text
이 목표일을 Google Calendar에 추가할까?
```

또는:

```text
이번 주 전진 프로젝트를 위한 60분 집중 블록을 캘린더에 만들까?
```

### Calendar 조회 사용

MVP 이후에는 Calendar events를 읽어서 주간 판단에 반영할 수 있다.

사용 예:

- 밋업 일정이 있는 날 감지
- 외부 일정이 많은 주에는 목표일 제안 완화
- 이동/외부 일정 후 복귀 작업 추천

주의:

Calendar 일정은 컨디션을 완전히 설명하지 않는다. AI는 일정만 보고 확정 판단하지 않고, 근거 중 하나로만 사용한다.

## 6. Gemini Function Calling 설계

Gemini function calling은 AI가 외부 API 호출을 직접 결정할 수 있게 해준다.

이 제품에서는 AI가 직접 API를 호출한다기보다, 앱이 안전한 함수 인터페이스를 제공하고 AI가 필요한 함수 호출을 제안하는 방식이 좋다.

### 함수 설계 원칙

- 함수 이름은 명확해야 한다.
- 함수는 한 가지 일만 해야 한다.
- destructive action은 사용자 확인 후 실행한다.
- 외부 연동 함수는 실패 가능성을 반환해야 한다.

### 후보 함수

#### exportWorkLogToSheet

입력:

```json
{
  "workLogId": "log-id"
}
```

출력:

```json
{
  "success": true,
  "spreadsheetId": "sheet-id",
  "rowNumber": 42
}
```

#### exportWeeklyPlanToSheet

입력:

```json
{
  "weeklyPlanId": "weekly-plan-id"
}
```

#### createCalendarEventForTargetDate

입력:

```json
{
  "projectId": "project-id",
  "targetDate": "2026-07-12",
  "title": "[Work OS] 수집 시스템 MVP 명세 목표일"
}
```

#### createCalendarFocusBlock

입력:

```json
{
  "projectId": "project-id",
  "taskId": "task-id",
  "start": "2026-07-08T10:00:00+09:00",
  "end": "2026-07-08T11:00:00+09:00"
}
```

#### listCalendarEventsForWeek

입력:

```json
{
  "weekStart": "2026-07-06",
  "weekEnd": "2026-07-12"
}
```

출력:

```json
{
  "events": [
    {
      "title": "Meetup",
      "start": "2026-07-09T19:00:00+09:00",
      "end": "2026-07-09T21:00:00+09:00"
    }
  ]
}
```

## 7. 연동 권한과 보안

### 최소 권한 원칙

필요한 범위만 요청한다.

Sheets:

- 읽기/쓰기 범위는 Work OS 전용 스프레드시트에 한정하는 것이 이상적이다.

Calendar:

- 목표일/작업 블록 생성과 일정 조회에 필요한 범위만 사용한다.

### 사용자 확인이 필요한 작업

- Calendar event 생성
- Calendar event 수정/삭제
- Sheets export 대상 변경
- 외부 데이터 import

## 8. 동기화 상태 모델

외부 연동 데이터에는 syncStatus가 필요하다.

값:

- not_synced
- synced
- failed
- pending_retry

저장 대상:

- WorkLog export status
- WeeklyPlan export status
- Calendar event creation status

예:

```json
{
  "entityType": "WorkLog",
  "entityId": "log-123",
  "target": "google_sheets",
  "syncStatus": "failed",
  "lastError": "quota_exceeded",
  "lastAttemptAt": "2026-07-05T10:00:00+09:00"
}
```

## 9. 실패 처리 원칙

### Sheets 실패

- 앱 내부 저장은 유지
- 실패 상태 기록
- 재시도 가능
- 사용자에게 과도하게 방해하지 않음

### Calendar 실패

- 앱 내부 목표일은 유지
- Calendar event id를 비워둠
- 실패 메시지를 보여줌
- 다시 시도 가능

### AI function call 실패

- 원본 사용자 요청은 보존
- 실패한 tool call과 에러를 기록
- AI가 사실을 꾸며서 성공했다고 말하지 못하게 함

## 10. MVP 연동 구현 순서

추천 순서:

1. 앱 내부 DB만으로 핵심 루프 구현
2. WorkLogs Google Sheets export
3. WeeklyPlan Google Sheets export
4. Project targetDate -> Calendar event 생성
5. Weekly review Calendar event 생성
6. Calendar events 조회 후 주간 판단 보조

## 11. 검증 시나리오

### Sheets export

1. 작업 완료 로그 생성
2. Sheets export 버튼 클릭
3. WorkLogs 시트에 행 추가
4. 앱 DB에는 syncStatus = synced 저장

### Sheets export 실패

1. API 실패 발생
2. 앱 데이터는 유지
3. syncStatus = failed 저장
4. 재시도 버튼 표시

### Calendar target event 생성

1. 프로젝트 nextSmallOutcome과 targetDate 존재
2. 사용자가 Calendar 추가 확인
3. events.insert 호출
4. eventId 저장

### Calendar 조회

1. 주간 판단 화면 진입
2. 해당 주 Calendar events 조회
3. 외부 일정이 많은 날 표시
4. AI 목표일 제안 근거에 반영

## 12. 연동에 대한 제품적 결론

Google Sheets와 Calendar는 이 제품의 중심이 아니다.

중심은 다음이다.

수집 -> 프로젝트 이해 -> 실행 추천 -> 로그 -> 프로젝트 이해 업데이트

Google 연동은 이 루프를 보조해야 한다.

Sheets는 기록을 밖으로 꺼내 읽고 분석하는 장치다.

Calendar는 목표일과 집중 시간을 현실 시간에 연결하는 장치다.

AI function calling은 이 둘을 안전하게 연결하는 방법이다.

