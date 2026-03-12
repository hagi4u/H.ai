# 데모 발표자료 준비 스킬

SFE 데모 발표를 위한 Confluence 문서를 코드 변경점 기반으로 자동 작성하는 스킬입니다.

## 사용법

```
/demo <branch> <confluence_url>
```

**예시:**
```
/demo feature/KREAM-40960 https://wiki.navercorp.com/pages/viewpage.action?pageId=4888182406
```

## 입력

| 파라미터 | 설명 | 필수 |
|----------|------|------|
| `branch` | 피처 브랜치명 (예: `feature/KREAM-40960`) | ✅ |
| `confluence_url` | 데모 문서 Confluence URL (pageId 포함) | ✅ |

## 작동 방식

### Step 1: 컨텍스트 수집 (병렬)

**A) Confluence 문서 읽기**
- `confluence_get_page` (convert_to_markdown: false) → 원본 storage format HTML 확보
- 문서 구조 파악: 어떤 섹션이 비어있는지 식별
- 기존 콘텐츠(참석자, Preview 시안, 스펙 등) 위치 기록

**B) 코드 변경점 분석**
- `git fetch origin`
- `git log --oneline origin/develop..origin/<branch>` → 커밋 히스토리
- `git diff origin/develop...origin/<branch> --stat` → 변경 파일 목록
- `git diff origin/develop...origin/<branch> -- <target_dir>/` → 상세 diff

### Step 2: 변경점 분석 & 그룹핑

diff를 분석하여 기능 단위로 그룹핑:
- **타입/인터페이스 변경** — 서버 응답 스펙 변경 사항
- **컴포넌트 신규/수정** — UI 렌더링 로직
- **composable/hook 추가** — 재사용 로직
- **스타일 변경** — CSS/SCSS 변경
- **로깅/트래킹** — Amplitude 등 이벤트 추가
- **설정/패키지** — 의존성 추가, 설정 변경

### Step 3: 시연 시나리오 작성

그룹핑된 기능 단위로 시연 테이블 작성:
- 기능별로 관련 시나리오를 묶어서 작성 (atom 단위 분리 X)
- 시나리오는 사용자 관점의 동작 설명
- 비고에는 기술적 구현 방식 간략 기재

### Step 4: 로직 설명 작성

파일 단위로 코드 변경점을 구조화:
- 파일별 h4 헤더 (`#### 1) 제목 — 파일경로`)
- 변경 의도를 한 줄 설명
- 실제 변경된 코드를 `ac:structured-macro` 코드 블록으로 삽입
- 코드는 핵심 로직만 발췌 (전체 파일 복붙 X)

### Step 5: Confluence 업데이트

⚠️ **반드시 섹션 단위 수정** (전체 덮어쓰기 금지)

```
1. 원본 HTML에서 수정 대상 섹션 식별
   - 시연: <h2>4. 시연</h2> ~ 다음 <h2> 사이
   - 로직 설명: <h3>로직 설명</h3> ~ 다음 <h2> 사이
2. 해당 섹션만 새 HTML로 교체
3. 나머지(배경, Preview, 스펙, 참석자, 회고, 피드백) 원본 유지
4. content_format: "storage" 로 업데이트
```

## Confluence storage format 규칙

| 요소 | 사용할 태그 |
|------|-------------|
| 테이블 | `<table><colgroup><col/></colgroup><thead><tr><th><p>...</p></th></tr></thead><tbody>...</tbody></table>` |
| 중첩 리스트 | `<ul><li>...<ul><li>...</li></ul></li></ul>` |
| 코드 블록 | `<ac:structured-macro ac:name="code"><ac:parameter ac:name="language">ts</ac:parameter><ac:plain-text-body><![CDATA[...]]></ac:plain-text-body></ac:structured-macro>` |
| 인라인 코드 | `<code>...</code>` |
| 이미지 (첨부) | `<ac:image><ri:attachment ri:filename="..." /></ac:image>` — 절대 삭제하지 않기 |

## 데모 문서 표준 구조

```
1. 배경 — 요청 배경, 히스토리, 참석자
2. Preview — 피그마 시안, 스크린샷
3. 스펙 설계 — 기획 협의, 요구사항, FE 리뷰, 테크스펙
4. 시연 — ⭐ 자동 작성 대상
   └ 로직 설명 — ⭐ 자동 작성 대상
5. 셀프 회고 — KPT (발표 후 직접 작성)
6. 피드백 및 개선사항 — (발표 후 팀원 작성)
7. Reference
```

## 주의사항

- 문서의 1~3, 5~7 섹션은 절대 수정하지 않는다
- 기존 첨부 이미지(`<ac:image>`) 참조를 보존한다
- 시연 시나리오는 기능 단위로 그룹핑하여 5~7행 이내로 정리한다
- 로직 설명의 코드는 핵심 변경점만 발췌한다 (전체 파일 X)
