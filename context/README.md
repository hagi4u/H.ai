# Context 진입점

> 이 폴더는 플랫폼 독립적입니다. Claude Code, Openclaw 등 어떤 AI 에이전트에서도 재사용 가능합니다.
> 이 파일을 진입점으로 설정하고, 필요에 따라 하위 파일을 참조하세요.

## 파일 구조

| 파일 | 설명 |
|------|------|
| `ai_identity.md` | AI(hai)의 정체성, 이름, 역할, 행동 지침 |
| `user_profile.md` | 사용자 프로필, 직업, 기술 스택, 관심사 |
| `schedule.md` | 일정 관리 |
| `growth.md` | Hai의 주간 성장 기록 |
| `error.md` | 실수 및 오류 로그 |

## 로드 순서

1. `ai_identity.md` — AI가 어떻게 행동해야 하는지 파악
2. `user_profile.md` — 사용자가 누구인지 파악
3. 필요시 `schedule.md` 참조

## 외부 참조

| 경로 | 설명 |
|------|------|
| `../works/projects.md` | 프로젝트 히스토리 및 진행 현황 |
| `../works/goals/2026.md` | 2026년 목표 및 Action Items — 업무 대화 시 참조 |
| `../skills/principles.md` | 코딩 원칙 — 기술 질문/코드 제안 시 참조 |
