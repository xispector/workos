# Google AI Studio Prototype Prompt

작성일: 2026-07-05

## 1. 사용 목적

이 문서는 Google AI Studio에서 Personal Work OS의 초기 웹 앱 프로토타입을 만들기 위한 통합 프롬프트 초안이다.

실제 구현 전에 제품 방향, 화면, AI 동작, 데이터 구조를 한 번에 전달하기 위한 문서다.

## 2. Prototype Prompt

```text
Build a personal web app prototype called "Personal Work OS".

The app is for one user, not a team or SaaS product. The user manages self-directed work such as personal software projects, startup ideas, studying, writing, networking, and operational tasks.

The product should not be a generic todo list. Its core purpose is to help the user capture messy tasks and thoughts, connect them to project context, break them down into executable actions, recommend what to do now, and update project understanding through lightweight work logs.

Language requirement:

- Build the user-facing app in Korean.
- All UI copy, navigation labels, buttons, empty states, helper text, AI assistant messages, recommendation reasons, clarification questions, log summaries, sample projects, and sample tasks should be written in Korean.
- Technical identifiers, JSON field names, enum values, function names, and external API names may remain in English.
- Do not default to English product copy.

Core product loop:

Capture -> Project linking -> Task breakdown -> Context-aware recommendation -> Execution -> Work log -> Project understanding update.

Main screens:

1. Today Work Screen
2. Quick Capture Flow
3. Project Detail Screen
4. Weekly Planning Screen

The first screen should be the Today Work Screen.

Today Work Screen requirements:

- Show an execution context bar at the top.
- Context bar format: "Home · Laptop · Quiet · Normal".
- Context fields:
  - Place: Home / Outside / Moving
  - Tool: Laptop / Phone only
  - Environment: Quiet / Distracting
  - Work intensity: Light / Normal / Deep
- The user can change these values manually.
- Show 3 recommended tasks.
- Each recommendation card should show:
  - Task title
  - Linked project
  - Work intensity
  - One-line recommendation reason
  - Start button
- Include a quick capture input on the same screen.
- Include a compact summary of this week's advance projects.

Quick Capture requirements:

- The user can enter messy natural language text.
- The user should not need to manually choose tags, priority, estimated time, or project.
- The AI should infer:
  - capture type
  - related project
  - whether clarification is needed
  - task breakdown
  - next action for the current context
- If the AI is uncertain, it should ask only one narrow question at a time.
- Avoid long forms and many-choice questions.

Project Detail Screen requirements:

Each project should have a project understanding card with:

- Project name
- Final goal
- Status stage
- Current context summary
- Result / artifact list
- Recent progress summary
- Next small outcome
- Next 10-minute action
- Blockers
- Target date
- Recent logs

Status stages:

- Waiting
- Researching
- Designing
- Executing
- Blocked
- Completed

Weekly Planning Screen requirements:

- Show projects grouped by weekly role:
  - Advance
  - Maintain
  - Defer
  - Blocked
- The AI should suggest a role for each project.
- Each suggestion must show a short reason.
- The user can accept or change the role.
- The weekly plan should focus on 1-2 advance projects.

Work completion flow:

- When a user completes a task, ask:
  "Leave a one-line log?"
- The user can type, speak, or skip.
- If the user leaves a log, AI summarizes it into:
  - work done
  - state change
  - next action
  - blockers
- If the user skips, create a low-trust automatic log:
  "Auto record - marked {task title} as complete. No detailed log."
- Low-trust logs should count as recent activity but should not strongly update project progress.

Persistence and integration requirements:

- The app should treat its internal app database as the source of truth.
- Google Sheets and Google Calendar are optional integrations, not the primary database.
- Google Sheets should be used for export and review, especially WorkLogs, Projects, WeeklyPlans, and GoalAdjustments.
- Google Calendar should be used for target dates, focus blocks, and weekly review events only after user confirmation.
- Do not automatically create calendar events for every recommended task.
- External API failures must not delete or corrupt internal app data.
- Store sync status for external exports or calendar event creation:
  - not_synced
  - synced
  - failed
  - pending_retry

Validation harness requirements:

- Include enough mock data and sample flows to verify the core loop.
- The prototype should make it easy to test:
  - quick capture
  - project linking
  - AI clarification question
  - context-aware task breakdown
  - recommendation reason
  - work completion log
  - low-trust automatic log
  - project recent progress update
- The prototype should not overstate progress from skipped logs.
- The prototype should show a clear difference between high-trust and low-trust logs.

AI behavior principles:

- Always show a one-line reason for recommendations.
- Do not overwhelm the user with choices.
- Do not ask for priority, due date, tags, and estimated time all at once.
- Prefer one narrow question if clarification is needed.
- Do not overstate progress from automatic logs.
- AI is a reasoning assistant, not an unquestionable manager.
- Important changes, such as project state or target date changes, should be shown for user confirmation.

Recommendation ranking should prioritize:

1. Tasks connected to this week's advance projects
2. Tasks possible in the current execution context
3. Tasks connected to the next small outcome
4. Tasks matching the selected intensity
5. Tasks with target date or delay risk
6. Tasks that help resume a project with stale logs

Design tone:

- Calm, utilitarian, work-focused
- Korean-first user experience
- Avoid marketing-style landing pages
- Avoid decorative hero sections
- Prioritize clarity, scanability, and fast action
- The first screen should be the actual working interface

Use mock data for the prototype.

Include sample projects:

1. Personal Work OS
   - Goal: Build a personal web app MVP for AI-assisted work management
   - Status: Designing
   - Next small outcome: Finish capture system MVP spec

2. Meetup Networking System
   - Goal: Turn meetup activity into structured follow-up and startup opportunities
   - Status: Researching
   - Next small outcome: Create a follow-up note template

3. Study Workflow
   - Goal: Build a repeatable workflow for learning and reviewing concepts
   - Status: Waiting
   - Next small outcome: Define a concept note template

Prototype should demonstrate:

- Quick capture creates a task
- AI links it to a project
- AI generates a context-aware next action
- Today screen updates recommendations
- Completing a task creates a work log
- Project recent progress updates from high-trust logs
```

## 3. 추가 개발 시 참조 문서

이 프롬프트는 다음 문서들을 기반으로 한다.

- `product-brief.md`
- `mvp-functional-spec.md`
- `ai-behavior-spec.md`
- `user-flows.md`
- `information-architecture.md`
- `data-model-draft.md`
- `ai-contract-spec.md`
- `validation-harness-workflow.md`
- `persistence-and-database-strategy.md`
- `google-integrations-strategy.md`

## 4. 프로토타입 검증 기준

Google AI Studio에서 만든 초안은 다음 기준으로 검토한다.

- 첫 화면이 Today Work Screen인가?
- 추천 작업 3개가 중심에 있는가?
- 상단 실행 컨텍스트 바가 있는가?
- 빠른 수집이 분류 폼처럼 무겁지 않은가?
- 추천 이유가 각 작업에 표시되는가?
- 프로젝트 상세 화면에 목표, 결과물, 최근 진척, 다음 행동이 있는가?
- 작업 완료 후 로그 입력 또는 스킵이 가능한가?
- AI가 자동 로그를 과장하지 않는가?
- 앱 DB가 source of truth로 유지되는가?
- Google Sheets/Calendar 연동은 선택적이고 실패해도 앱 데이터가 보존되는가?
