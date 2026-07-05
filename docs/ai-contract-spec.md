# AI Contract Spec

작성일: 2026-07-05

## 1. 목적

이 문서는 Personal Work OS에서 AI가 어떤 입력을 받고 어떤 구조로 출력해야 하는지 정의한다.

Google AI Studio에서 프로토타입을 만들 때 가장 직접적으로 사용할 수 있는 명세다.

## 2. 공통 원칙

AI 출력은 가능한 한 구조화한다.

필수 원칙:

- 중요한 판단에는 reason을 포함한다.
- 확신이 낮으면 confidence를 낮게 표시한다.
- 사용자에게 질문할 때는 한 번에 하나만 묻는다.
- 선택지는 최대 2개를 기본으로 한다.
- 자동 로그는 실제 진척을 과장하지 않는다.
- 추천 작업에는 항상 한 줄 이유가 있다.

## 3. 공통 타입

### Confidence

```json
"high" | "medium" | "low"
```

### WorkContext

```json
{
  "place": "home | outside | moving",
  "tool": "laptop | phone_only",
  "environment": "quiet | distracting",
  "intensity": "light | normal | deep"
}
```

### ProjectSnapshot

```json
{
  "id": "project-id",
  "name": "Personal Work OS",
  "finalGoal": "개인용 Work OS 웹 앱 MVP 만들기",
  "statusStage": "designing",
  "currentContextSummary": "수집 시스템과 프로젝트 이해 시스템을 설계 중",
  "recentProgressSummary": "수집-프로젝트 연결-태스크 분해 흐름을 문서화함",
  "nextSmallOutcome": "수집 시스템 MVP 명세 완성",
  "nextTenMinuteAction": "수집 입력 예시 3개 작성",
  "blockerSummary": "AI 질문 규칙의 세부 기준이 아직 불명확함",
  "targetDate": "2026-07-12"
}
```

## 4. 자유 입력 처리

### 목적

사용자의 자유 입력을 받아 프로젝트 연결, 유형 분류, 필요한 질문, 태스크 분해 초안을 만든다.

### Input

```json
{
  "rawInput": "수집 시스템 정리해야 함",
  "inputMode": "text",
  "activeWorkContext": {
    "place": "home",
    "tool": "laptop",
    "environment": "quiet",
    "intensity": "normal"
  },
  "recentProjects": [
    {
      "id": "p1",
      "name": "Personal Work OS",
      "finalGoal": "개인용 Work OS 웹 앱 MVP 만들기",
      "statusStage": "designing",
      "currentContextSummary": "제품 문서와 수집 시스템을 구체화 중",
      "recentProgressSummary": "오늘 작업 화면과 AI 행동 규칙을 문서화함",
      "nextSmallOutcome": "수집 시스템 MVP 명세 완성",
      "nextTenMinuteAction": "수집 입력 예시 3개 작성"
    }
  ]
}
```

### Output

```json
{
  "captureType": "task",
  "captureTypeConfidence": "high",
  "linkedProject": {
    "projectId": "p1",
    "confidence": "high",
    "reason": "입력 내용이 최근 설계 중인 수집 시스템과 직접 연결됨"
  },
  "needsQuestion": false,
  "question": null,
  "createdTask": {
    "title": "수집 시스템 정리하기",
    "taskType": "action",
    "intensity": "normal",
    "sizeHint": "medium",
    "projectId": "p1"
  },
  "taskBreakdown": [
    {
      "title": "수집 입력 유형 정리하기",
      "intensity": "normal"
    },
    {
      "title": "프로젝트 연결 규칙 정리하기",
      "intensity": "normal"
    },
    {
      "title": "맥락 질문 규칙 정리하기",
      "intensity": "normal"
    }
  ],
  "contextualNextAction": {
    "title": "수집 입력 예시 3개 작성하기",
    "reason": "현재 노트북과 조용한 환경에서 바로 문서화할 수 있고, 수집 시스템 명세에 직접 연결됨",
    "intensity": "normal"
  }
}
```

### Question Output

프로젝트 연결이 불확실하면 다음 형식으로 출력한다.

```json
{
  "captureType": "task",
  "captureTypeConfidence": "medium",
  "linkedProject": null,
  "needsQuestion": true,
  "question": {
    "questionText": "이건 기존 프로젝트를 이어가는 일이야?",
    "questionType": "project_linking",
    "answerMode": "yes_no",
    "options": ["yes", "no"]
  }
}
```

## 5. 태스크 분해

### 목적

큰 태스크를 프로젝트 맥락과 현재 실행 컨텍스트에 맞게 실행 가능한 작업으로 바꾼다.

### Input

```json
{
  "task": {
    "id": "t1",
    "title": "프로젝트 이해 시스템 설계하기",
    "description": null
  },
  "project": {
    "id": "p1",
    "name": "Personal Work OS",
    "finalGoal": "개인용 Work OS 웹 앱 MVP 만들기",
    "currentContextSummary": "프로젝트 목표와 현재 상태 카드를 설계 중",
    "nextSmallOutcome": "프로젝트 이해 카드 구조 완성"
  },
  "workContext": {
    "place": "outside",
    "tool": "phone_only",
    "environment": "distracting",
    "intensity": "light"
  }
}
```

### Output

```json
{
  "breakdown": [
    {
      "title": "프로젝트 이해 카드에 들어갈 정보 후보 적기",
      "intensity": "light",
      "environmentFit": "good"
    },
    {
      "title": "현재 프로젝트 예시 하나를 음성으로 설명하기",
      "intensity": "light",
      "environmentFit": "good"
    }
  ],
  "recommendedNextAction": {
    "title": "프로젝트 이해 카드에 들어갈 정보 후보 3개 적기",
    "reason": "현재 휴대폰만 가능하고 산만한 환경이라 긴 문서 작성 대신 짧은 후보 적기가 적합함",
    "intensity": "light"
  }
}
```

## 6. 추천 작업 생성

### 목적

오늘 작업 화면에 보여줄 추천 작업 3개를 생성한다.

### Input

```json
{
  "workContext": {
    "place": "home",
    "tool": "laptop",
    "environment": "quiet",
    "intensity": "normal"
  },
  "weeklyPlan": {
    "advanceProjectIds": ["p1"],
    "maintainProjectIds": ["p2"],
    "deferredProjectIds": ["p3"]
  },
  "projects": [
    {
      "id": "p1",
      "name": "Personal Work OS",
      "nextSmallOutcome": "수집 시스템 MVP 명세 완성",
      "recentProgressSummary": "제품 문서와 AI 행동 규칙을 작성함",
      "nextTenMinuteAction": "수집 입력 예시 3개 작성"
    }
  ],
  "candidateTasks": [
    {
      "id": "t1",
      "projectId": "p1",
      "title": "수집 입력 예시 3개 작성",
      "intensity": "normal",
      "isTenMinuteAction": true
    }
  ]
}
```

### Output

```json
{
  "recommendations": [
    {
      "taskId": "t1",
      "projectId": "p1",
      "title": "수집 입력 예시 3개 작성",
      "reason": "이번 주 전진 프로젝트이고, 현재 집/노트북/조용함/보통 컨텍스트에서 바로 실행 가능함",
      "rank": 1,
      "intensity": "normal"
    }
  ]
}
```

## 7. 작업 완료 로그 정리

### 목적

사용자의 자유 로그를 프로젝트 상태 업데이트에 쓸 수 있는 구조로 정리한다.

### Input

```json
{
  "task": {
    "id": "t1",
    "title": "수집 입력 예시 3개 작성"
  },
  "project": {
    "id": "p1",
    "name": "Personal Work OS",
    "currentContextSummary": "수집 시스템 MVP 명세를 작성 중"
  },
  "rawLog": "해야 할 일, 프로젝트 로그, 아이디어 입력 예시를 정리했고 다음엔 AI 질문 규칙을 더 구체화하면 될 것 같아.",
  "inputMode": "text"
}
```

### Output

```json
{
  "trustLevel": "high",
  "summary": "수집 입력 유형 예시를 정리했고, 다음 작업은 AI 질문 규칙 구체화로 이어짐.",
  "workDone": "해야 할 일, 프로젝트 로그, 아이디어 입력 예시를 정리함",
  "stateChange": "수집 시스템 명세가 입력 사례 수준까지 구체화됨",
  "nextAction": "AI 질문 규칙을 구체화하기",
  "blockers": null,
  "projectUpdateCandidate": {
    "currentContextSummary": "수집 시스템의 입력 예시를 정리했고, AI 질문 규칙 구체화가 다음 작업으로 남음",
    "nextTenMinuteAction": "AI 질문 규칙의 좋은 예/나쁜 예를 3개씩 작성하기"
  },
  "artifactCandidate": null,
  "needsConfirmation": true,
  "confirmationTarget": "projectUpdateCandidate"
}
```

## 8. 자동 최소 로그

### Input

```json
{
  "task": {
    "id": "t1",
    "title": "수집 입력 예시 3개 작성"
  },
  "userSkippedLog": true
}
```

### Output

```json
{
  "trustLevel": "low",
  "summary": "자동 기록 - 수집 입력 예시 3개 작성 작업을 완료로 표시함. 세부 로그 없음.",
  "workDone": null,
  "stateChange": null,
  "nextAction": null,
  "blockers": null,
  "projectUpdateCandidate": null,
  "artifactCandidate": null
}
```

## 9. 주간 판단 생성

### 목적

프로젝트별 이번 주 역할을 전진/유지/보류/막힘으로 제안한다.

### Output

```json
{
  "weeklyDecisions": [
    {
      "projectId": "p1",
      "suggestedRole": "advance",
      "reason": "최근 문서화 진척이 있고, 다음 작은 결과물이 MVP 명세 완성과 직접 연결됨",
      "riskLevel": "safe",
      "nextSmallOutcome": "수집 시스템 MVP 명세 완성",
      "targetDate": "2026-07-12"
    },
    {
      "projectId": "p2",
      "suggestedRole": "defer",
      "reason": "최근 결과물이 있고 다음 행동이 남아 있어 이번 주에는 보류해도 복귀 가능함",
      "riskLevel": "safe"
    }
  ]
}
```

## 10. 목표일 재계산

### Input

```json
{
  "project": {
    "id": "p1",
    "name": "Personal Work OS",
    "nextSmallOutcome": "수집 시스템 MVP 명세 완성",
    "targetDate": "2026-07-08",
    "recentProgressSummary": "최근 3일간 작업 로그가 없음"
  },
  "trigger": "near_due_no_progress"
}
```

### Output

```json
{
  "previousTargetDate": "2026-07-08",
  "newTargetDate": "2026-07-12",
  "reason": "목표일이 가까우나 최근 높은 신뢰도 로그가 없어 기존 목표일은 빡빡해 보임. 결과물 범위는 유지하되 목표일을 조정하는 것이 안전함.",
  "riskIfNotAdjusted": "지난 목표일이 계속 노출되어 회피감이 커질 수 있음",
  "confidence": "medium"
}
```

## 11. MVP에서 필요한 AI 함수 목록

Google AI Studio 프로토타입 기준으로 필요한 AI 기능:

1. classifyCaptureItem
2. linkProject
3. generateClarifyingQuestion
4. breakDownTask
5. generateRecommendations
6. summarizeWorkLog
7. createAutoMinimalLog
8. updateProjectUnderstanding
9. generateWeeklyDecisions
10. recalculateTargetDate

## 12. 다음 문서로 이어지는 결정

다음 문서는 `google-ai-studio-prototype-prompt.md`다.

이 문서에서는 지금까지 작성한 제품 명세를 바탕으로 Google AI Studio에서 앱 초안을 만들기 위한 통합 프롬프트를 작성한다.

