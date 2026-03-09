# hai의 성장 기록

> 매주 리뷰 후 작성. 발견한 내용은 `context/`, `skills/` 등 해당 파일에 반영한다.

---

## 작성 가이드

핵심 질문: "다음에 비슷한 상황에서 hai가 더 잘할 수 있게 됐는가?"

### 기록 대상
- 🛠️ 스킬/워크플로우 생성·개선
- 📐 규칙·원칙·아키텍처 변경
- 🔌 도구(MCP, 훅) 능력 확장
- 🔍 에러에서 배운 구조적 교훈
- 🧠 설계 인사이트/패턴 발견
- 🎯 사용자의 피드백 반영

### 흐름
1. 매주 금요일 대화 시 리뷰 제안
2. 기록된 내용을 바탕으로 관련 파일 업데이트
   - 사용자 성향 → `context/user_profile.md`
   - 행동 지침 → `context/ai_identity.md`
   - 기술 지식 → `skills/` 하위 파일
3. 300줄 초과 시 연도별 아카이빙 (예: `growth_2026_archive.md`)

---

## 2026

### W10 (2026-03-04)

#### 사용자 파악
- Web Front-end 개발자, KREAM 서비스 FE 팀 소속
- TPM 역할도 겸하며 Android/iOS 제약사항을 자주 다룸
- 구조화와 설계를 중시하지만 속도와 단순화도 올해 핵심 과제로 삼고 있음
- Agentic AI, Sub Agent 워크플로우에 관심이 많음
- 집에서는 Openclaw라는 Agentic AI 환경 사용

#### 행동 개선
- hai라는 이름과 역할 정립
- 코딩 원칙 5가지 합의 (SRP, 프레임워크 독립, 리팩토링 용이성, 단방향 의존성, 임의 결정 금지)
- skills에 누적할 만한 내용 발견 시 능동적으로 저장 여부를 묻기로 함

#### 지식/역할 확장
- `skills/` 구조 구축 (javascript, typescript, nuxt3, nextjs, vue3, react, python, platforms)
- `skills/platforms/ios.md` — iOS SVG URL 렌더링 불가 제약사항 추가
- `skills/javascript/patterns.md` — Int32Array 개념 추가
- `works/goals/2026.md` — 연간 목표 및 Action Items 정리
- `~/.claude/CLAUDE.md` — 글로벌 컨텍스트 설정으로 모든 프로젝트에서 hai 로드

### W11 (2026-03-09)

#### 사용자 파악
- 회사 닉네임: Bobby → 캘린더에서 `[1on1] bobby` = 본인 일정
- Yuki = 1on1 상대방 (매주 월요일 11:00)

#### 행동 개선
- 아침 인사 시 확인 묻지 않고 바로 브리핑 시작 (오늘 캘린더 + 안읽은 메일)
- 캘린더는 2개만 사용: 정명학 기본 (`c_1001_ec297333-168e-4de3-b499-290b2fc3a924`) + KREAM Frontend 팀 (`c_1001_0d3fa539-87bc-45a2-bd8d-c371783291b9`)
- 아침 브리핑 포맷: 📅 오늘 일정 → 🚨 확인 필요 → 🔀 PR 알림 → 📦 배포 → 📋 기타

#### 지식/역할 확장
- Naver Works MCP로 캘린더 조회 및 메일 브리핑 가능 확인
- Naver Works MCP 메일 삭제 기능은 미지원 (read/unread, 중요 표시만 가능)

#### 성과
- 크림페이 Phase1 성공적 론칭 (이슈 없이 프로덕션 오픈)
