# Project Dashboard Implementation Guidelines

작성일: 2026-07-05

## 1. 목적

이 문서는 Personal Work OS에 프로젝트 대시보드를 추가하기 위한 구현 지침이다.

프로젝트 대시보드는 AI에게 질문하지 않아도 사용자가 직접 프로젝트들의 현재 상태를 볼 수 있는 화면이다.

목표:

- 전체 프로젝트의 상태를 빠르게 스캔한다.
- 어떤 프로젝트가 전진/유지/보류/막힘인지 확인한다.
- 지연 위험과 막힌 프로젝트를 빠르게 발견한다.
- 각 프로젝트의 다음 작은 결과물과 다음 행동을 바로 볼 수 있다.
- 프로젝트 상세 화면으로 자연스럽게 들어갈 수 있다.

## 2. 리서치 기반 설계 근거

### Dashboard UX

Nielsen Norman Group은 대시보드를 "한 페이지에서 빠르게 이해하고 행동할 수 있는 정보를 보여주는 데이터 시각화 모음"으로 설명한다. 대시보드의 목적은 단순히 많은 정보를 보여주는 것이 아니라, 자주 모니터링해야 하는 정보를 빠르게 파악하고 행동하게 하는 것이다.

참고:

- https://www.nngroup.com/articles/dashboards-preattentive/
- https://www.nngroup.com/videos/data-visualizations-dashboards/

이 제품에 적용:

- 프로젝트 대시보드는 예쁜 차트 모음이 아니라 빠른 상태 판단 화면이어야 한다.
- 사용자가 바로 행동할 수 있는 정보가 우선이다.
- "다음에 뭘 해야 하지?"를 다시 만들면 실패다.

### Linear Projects

Linear는 프로젝트에 상태, 문서, 리소스, 진행 그래프, 프로젝트 업데이트를 함께 둔다. Project updates는 health indicator와 설명을 통해 진행 상태, 도전 과제, 다음 단계를 전달한다.

참고:

- https://linear.app/docs/projects
- https://linear.app/docs/initiative-and-project-updates

이 제품에 적용:

- 각 프로젝트 카드에는 상태 단계와 현재 맥락 문장이 함께 있어야 한다.
- 단순 진행률보다 "현재 상태와 다음 단계"가 중요하다.
- 프로젝트 문서/결과물/로그로 들어가는 연결이 필요하다.

### Asana Portfolio / Status Updates

Asana의 Portfolio와 status update 기능은 여러 프로젝트를 한곳에서 보고, 프로젝트 health와 진행 상황을 모니터링하는 데 초점이 있다.

참고:

- https://help.asana.com/s/article/portfolio-progress-and-reporting
- https://help.asana.com/s/article/project-progress-and-status-updates
- https://help.asana.com/s/article/reporting-with-dashboards

이 제품에 적용:

- 프로젝트 전체를 보는 portfolio형 화면이 필요하다.
- 각 프로젝트에는 현재 상태, 위험, 다음 마일스톤 또는 결과물이 보여야 한다.
- 다만 팀 보고용이 아니라 개인 복귀와 실행을 위한 화면이어야 한다.

### ClickUp Dashboards

ClickUp은 대시보드를 작업 정보, 진행, 시간, 성과를 시각화하는 고수준 화면으로 사용한다.

참고:

- https://help.clickup.com/hc/en-us/articles/6312197753239-Intro-to-Dashboards

이 제품에 적용:

- 위젯형 개념은 유용하지만, MVP에서는 너무 많은 위젯을 만들지 않는다.
- 개인용 Work OS에서는 차트보다 리스트와 상태 신호가 더 실용적이다.

## 3. 제품 내 위치

앱의 주요 화면은 다음으로 확장한다.

1. 오늘 작업 화면
2. 빠른 수집 플로우
3. 프로젝트 대시보드
4. 프로젝트 상세 화면
5. 주간 판단 화면

프로젝트 대시보드는 오늘 작업 화면과 프로젝트 상세 화면 사이에 있다.

역할 구분:

- 오늘 작업 화면: 지금 할 작업을 고르는 실행 화면
- 프로젝트 대시보드: 전체 프로젝트 상태를 스캔하는 화면
- 프로젝트 상세 화면: 프로젝트 하나의 맥락과 로그를 깊게 보는 화면
- 주간 판단 화면: 이번 주 전진/유지/보류/막힘을 확정하는 화면

## 4. 대시보드의 핵심 질문

프로젝트 대시보드는 사용자가 다음 질문에 빠르게 답할 수 있어야 한다.

1. 지금 어떤 프로젝트들이 진행 중인가?
2. 이번 주에 무엇을 전진시키고 있는가?
3. 어떤 프로젝트가 막혀 있는가?
4. 어떤 프로젝트가 너무 오래 결과물 없이 처지고 있는가?
5. 각 프로젝트는 다음에 무엇을 하면 되는가?
6. 최근에 실제 진척이 있었는가?
7. 보류 중인 프로젝트는 복귀 가능한 상태인가?

## 5. 기본 화면 구조

프로젝트 대시보드는 4개 영역으로 구성한다.

1. 상단 요약 스트립
2. 주의 필요 섹션
3. 프로젝트 리스트
4. 빠른 필터/정렬

## 6. 상단 요약 스트립

### 목적

전체 프로젝트 상태를 5초 안에 파악하게 한다.

### 표시 항목

4개 요약 카드만 둔다.

1. 전진 프로젝트
2. 막힘
3. 지연 위험
4. 최근 7일 내 결과물

예:

- 전진 2
- 막힘 1
- 지연 위험 2
- 최근 결과물 3

### 설계 원칙

- 숫자를 너무 많이 만들지 않는다.
- "해야 할 일 총 개수"는 상단 핵심 지표로 두지 않는다.
- 사용자가 행동해야 하는 상태 신호를 우선한다.

## 7. 주의 필요 섹션

### 목적

사용자가 가장 먼저 봐야 하는 프로젝트를 자동으로 끌어올린다.

### 포함 조건

다음 중 하나라도 해당하면 표시한다.

- 상태가 `막힘`
- 지연 위험이 `risk`
- 목표일이 임박했는데 최근 높은 신뢰도 로그가 없음
- 2주 이상 결과물이 없음
- 다음 10분 행동이 없음
- AI가 프로젝트 연결/상태를 확신하지 못함

### 카드 내용

- 프로젝트 이름
- 위험/주의 이유
- 현재 상태 한 문장
- 필요한 조치

예:

```text
Personal Work OS
주의: 다음 작은 결과물 목표일이 가까운데 최근 높은 신뢰도 로그가 없음
현재: 수집 시스템 명세를 작성 중
필요한 조치: 목표일 재계산 또는 10분 복귀 작업 실행
```

## 8. 프로젝트 리스트

### 기본 형태

MVP에서는 카드 그리드보다 밀도 있는 리스트를 기본으로 한다.

이유:

- 프로젝트를 여러 개 빠르게 비교해야 한다.
- 카드가 커지면 한 화면에서 볼 수 있는 프로젝트 수가 줄어든다.
- 개인 운영 도구는 장식보다 스캔성이 중요하다.

### 각 프로젝트 행에 표시할 정보

필수:

- 프로젝트 이름
- 상태 단계
- 이번 주 역할
- 현재 맥락 한 문장
- 다음 작은 결과물
- 다음 10분 행동
- 마지막 높은 신뢰도 로그 날짜
- 목표일 또는 지연 위험

선택:

- 최근 결과물 개수
- 막힌 점
- 프로젝트 상세 링크

### 행 예시

```text
Personal Work OS
설계 중 · 전진 · 위험 낮음
현재: 수집/AI/검증 문서를 정리했고 대시보드 지침을 추가하는 중
다음 결과물: 프로젝트 대시보드 구현 지침 완성
다음 행동: 대시보드 행 정보 필드 정리
최근 로그: 오늘
목표일: 2026-07-12
```

## 9. 정렬 기본값

기본 정렬은 사용자가 직접 고르지 않아도 앱이 제공한다.

정렬 순서:

1. 막힘 프로젝트
2. 지연 위험 프로젝트
3. 이번 주 전진 프로젝트
4. 최근 작업한 프로젝트
5. 유지 프로젝트
6. 보류 프로젝트

이유:

- 대시보드는 먼저 주의가 필요한 것을 보여줘야 한다.
- 오늘 작업 화면은 "지금 할 일" 중심이지만, 프로젝트 대시보드는 "상태 감지"가 중심이다.

## 10. 필터

MVP 필터는 최소로 둔다.

필터:

- 전체
- 전진
- 유지
- 보류
- 막힘
- 지연 위험

검색:

- 프로젝트 이름 검색

나중으로 미룰 것:

- 복잡한 태그 필터
- 여러 조건 조합 필터
- 커스텀 대시보드 위젯

## 11. 프로젝트 상태 신호

프로젝트 행에는 다음 신호가 있어야 한다.

### 상태 단계

- 대기
- 조사 중
- 설계 중
- 실행 중
- 막힘
- 완료

### 이번 주 역할

- 전진
- 유지
- 보류
- 막힘

### 지연 위험

- 안전
- 주의
- 위험

판단 기준:

- 안전: 최근 결과물 또는 높은 신뢰도 로그가 있고 다음 행동이 있음
- 주의: 7일 이상 높은 신뢰도 로그가 없거나 목표일이 가까움
- 위험: 2주 이상 결과물이 없거나 목표일이 지났고 다음 행동도 불명확함

### 로그 신뢰도 표시

최근 활동이 자동 최소 로그뿐이면 "최근 작업 있음"은 표시하되 "진척 있음"으로 과장하지 않는다.

표시 예:

- 최근 진척: 오늘
- 최근 활동: 오늘, 자동 기록만 있음

## 12. 행동 버튼

각 프로젝트 행의 기본 행동은 3개로 제한한다.

1. 상세 보기
2. 다음 10분 행동 시작
3. 목표/상태 재조정

상황별 보조 행동:

- 막힘 프로젝트: 막힌 점 정리
- 다음 행동 없음: 다음 행동 만들기
- 지연 위험: 목표일 재계산
- 보류 프로젝트: 복귀 가능 상태 확인

버튼이 많아지면 스캔성이 떨어지므로 행 안에는 최대 3개만 노출한다.

## 13. AI가 대시보드에서 해야 할 일

AI는 대시보드에서 직접 대화형 답변만 하는 것이 아니라, 화면에 표시될 정보를 미리 정리해야 한다.

AI 역할:

- 최근 진척 요약 생성
- 지연 위험 판단
- 주의 필요 프로젝트 선별
- 다음 10분 행동 누락 감지
- 보류해도 되는 근거 생성
- 목표일 재계산 후보 생성

AI가 하면 안 되는 것:

- 사용자 확인 없이 프로젝트 상태를 크게 변경
- 자동 로그만 보고 결과물이 생겼다고 판단
- 모든 프로젝트를 같은 우선순위로 보여주기
- 대시보드를 설명 텍스트로 채우기

## 14. 데이터 요구사항

프로젝트 대시보드에 필요한 데이터:

- Project.name
- Project.statusStage
- Project.currentContextSummary
- Project.recentProgressSummary
- Project.nextSmallOutcome
- Project.nextTenMinuteAction
- Project.targetDate
- WeeklyProjectDecision.role
- WorkLog.createdAt
- WorkLog.trustLevel
- Artifact.createdAt
- GoalAdjustment.reason

추가로 계산되는 값:

- lastHighTrustLogAt
- lastActivityAt
- artifactCountRecent
- delayRisk
- needsAttention
- missingNextAction

## 15. Google AI Studio 프롬프트 반영 지침

Google AI Studio 프로토타입 프롬프트에는 다음을 명시한다.

```text
Add a Project Dashboard screen.

The Project Dashboard is not a decorative analytics dashboard. It is a status-scanning screen for personal project control.

It should show:

- Summary strip with Advance, Blocked, Delay Risk, Recent Artifacts
- Needs Attention section
- Dense project list
- Filters: All, Advance, Maintain, Defer, Blocked, Delay Risk

Each project row should show:

- Project name
- Status stage
- Weekly role
- Current context summary
- Next small outcome
- Next 10-minute action
- Last high-trust log date
- Target date or delay risk

Default sorting:

1. Blocked projects
2. Delay risk projects
3. This week's advance projects
4. Recently active projects
5. Maintain projects
6. Deferred projects

All user-facing copy must be Korean.
```

## 16. MVP 구현 우선순위

### MVP에 포함

- 프로젝트 대시보드 화면
- 상단 요약 스트립
- 주의 필요 섹션
- 프로젝트 리스트
- 기본 필터
- 상세 보기 연결
- 다음 10분 행동 시작

### MVP 이후

- 커스텀 위젯
- 차트 기반 분석
- 드래그 앤 드롭 정렬
- 여러 대시보드 뷰 저장
- 프로젝트별 상세 그래프

## 17. 수용 기준

프로젝트 대시보드는 다음을 만족해야 한다.

- 사용자가 AI에게 묻지 않아도 전체 프로젝트 상태를 볼 수 있다.
- 막힌 프로젝트와 지연 위험 프로젝트가 눈에 띈다.
- 각 프로젝트의 다음 작은 결과물과 다음 10분 행동이 보인다.
- 자동 로그만 있는 프로젝트가 실제 진척으로 과장되지 않는다.
- 오늘 작업 화면과 프로젝트 상세 화면으로 자연스럽게 이동할 수 있다.
- UI와 문구는 한국어다.

## 18. 공식/참고 출처

- Nielsen Norman Group, Dashboards: Making Charts and Graphs Easier to Understand  
  https://www.nngroup.com/articles/dashboards-preattentive/

- Nielsen Norman Group, Data Visualizations for Dashboards  
  https://www.nngroup.com/videos/data-visualizations-dashboards/

- Linear Docs, Projects  
  https://linear.app/docs/projects

- Linear Docs, Initiative and Project Updates  
  https://linear.app/docs/initiative-and-project-updates

- Asana Help, Portfolio progress and reporting  
  https://help.asana.com/s/article/portfolio-progress-and-reporting

- Asana Help, Project progress and status updates  
  https://help.asana.com/s/article/project-progress-and-status-updates

- Asana Help, Reporting with dashboards  
  https://help.asana.com/s/article/reporting-with-dashboards

- ClickUp Help, Intro to Dashboards  
  https://help.clickup.com/hc/en-us/articles/6312197753239-Intro-to-Dashboards

