# 데모 발표자료 준비 스킬

SFE 데모 발표를 위한 Confluence 문서 생성부터 코드 변경점 기반 내용 작성까지 자동으로 수행하는 스킬입니다.

## 사용법

```
/demo <branch> <title>
/demo <branch> <confluence_url>
```

**새 문서 생성:**
```
/demo feature/KREAM-40960 "[26.x] 홈 탭 이미지 대응"
```

**기존 문서 업데이트:**
```
/demo feature/KREAM-40960 https://wiki.navercorp.com/pages/viewpage.action?pageId=4888182406
```

## 입력

| 파라미터 | 설명 | 필수 |
|----------|------|------|
| `branch` | 피처 브랜치명 (예: `feature/KREAM-40960`) | ✅ |
| `title` 또는 `confluence_url` | 새 문서 제목(따옴표) 또는 기존 문서 URL | ✅ |

- 두 번째 인자가 `http`로 시작하면 → **기존 문서 업데이트** 모드
- 그 외 → **새 문서 생성** 모드

## 작동 방식

### Step 1: 코드 변경점 분석

```bash
git fetch origin
git log --oneline origin/develop..origin/<branch>       # 커밋 히스토리
git diff origin/develop...origin/<branch> --stat         # 변경 파일 목록
git diff origin/develop...origin/<branch> -- <target>/   # 상세 diff
```

- diff를 기능 단위로 그룹핑:
  - 타입/인터페이스 변경, 컴포넌트 신규/수정, composable/hook, 스타일, 로깅/트래킹, 설정/패키지

### Step 2-A: 새 문서 생성 (title 모드)

`confluence_create_page`로 KREAM 스페이스에 데모 템플릿 문서를 생성한다.

- `space_key`: "KREAM"
- `title`: 사용자 입력 제목
- `content_format`: "storage"
- 생성 시 아래 **표준 템플릿**의 모든 섹션을 포함
- 1~3 섹션의 빈 칸은 Bobby가 직접 채울 수 있도록 placeholder로 둔다
- **4. 시연**과 **로직 설명**은 Step 3~4에서 코드 분석 결과로 채운다

### Step 2-B: 기존 문서 업데이트 (URL 모드)

⚠️ **반드시 섹션 단위 수정** (전체 덮어쓰기 금지)

```
1. confluence_get_page (convert_to_markdown: false) → 원본 storage HTML 확보
2. 원본 HTML에서 수정 대상 섹션만 식별
   - 시연: <h2>4. 시연</h2> ~ 다음 <h2> 사이
   - 로직 설명: <h3>로직 설명</h3> ~ 다음 <h2> 사이
3. 해당 섹션만 새 HTML로 교체, 나머지 원본 유지
4. confluence_update_page (content_format: "storage")
```

### Step 3: 시연 시나리오 작성

그룹핑된 기능 단위로 시연 테이블 작성:
- 기능별로 관련 시나리오를 묶어서 작성 (atom 단위 분리 X, 5~7행 이내)
- 시나리오 셀에는 `<ul>` 로 세부 항목 나열
- 비고에는 기술적 구현 방식 간략 기재

### Step 4: 로직 설명 작성

파일 단위로 코드 변경점을 구조화:
- 파일별 h4 헤더 (`#### 1) 제목 — 파일경로`)
- 변경 의도를 한 줄 설명
- 실제 변경된 코드를 코드 블록 매크로로 삽입 (핵심 로직만 발췌)

### Step 5: Confluence 반영

- 새 문서 → `confluence_create_page` (content_format: "storage")
- 기존 문서 → `confluence_update_page` (content_format: "storage", 섹션 교체)
- 완료 후 문서 URL을 사용자에게 반환

## 표준 템플릿 (storage format)

새 문서 생성 시 아래 HTML을 기본 골격으로 사용한다.

```html
<h2>1. 배경 (1m)</h2>
<ul>
  <li>요청 배경
    <ul>
      <li>(배경 내용을 작성해주세요)</li>
    </ul>
  </li>
</ul>
<h3>참석자</h3>
<p>(참석자를 추가해주세요)</p>

<h2>2. Preview (10m)</h2>
<p>(피그마 시안이나 스크린샷을 첨부해주세요)</p>

<h2>3. 스펙 설계</h2>
<h3>ㄱ) 기획 리뷰 협의 내용</h3>
<ul><li>(협의 내용을 작성해주세요)</li></ul>
<h3>ㄴ) 요구사항 정리</h3>
<table>
  <colgroup><col /><col /><col /></colgroup>
  <thead><tr><th><p>화면</p></th><th><p>요구사항</p></th><th><p>비고</p></th></tr></thead>
  <tbody><tr><td><p /></td><td><p /></td><td><p /></td></tr></tbody>
</table>
<h3>ㄷ) FE 설계 리뷰 문의 내용</h3>
<ul><li>(리뷰 문의 내용을 작성해주세요)</li></ul>
<h3>ㄹ) 테크스펙 사전체크</h3>
<ul><li>(사전체크 항목을 작성해주세요)</li></ul>

<h2>4. 시연</h2>
<!-- ⭐ 코드 분석 결과로 자동 작성 -->

<h2>5. 셀프 회고 (5m)</h2>
<table>
  <colgroup><col /><col /><col /><col /></colgroup>
  <thead><tr><th><p /></th><th><p>Keep</p></th><th><p>Problem</p></th><th><p>Try</p></th></tr></thead>
  <tbody><tr><td><p /></td><td><p /></td><td><p /></td><td><p /></td></tr></tbody>
</table>

<h2>6. 피드백 및 개선사항</h2>
<table>
  <colgroup><col /><col /><col /></colgroup>
  <thead><tr><th><p>작성자</p></th><th><p>피드백 및 개선사항</p></th><th><p>비고</p></th></tr></thead>
  <tbody><tr><td><p /></td><td><p /></td><td><p /></td></tr></tbody>
</table>

<h2>7. Reference</h2>
```

## Confluence storage format 규칙

| 요소 | 사용할 태그 |
|------|-------------|
| 테이블 | `<table><colgroup><col/></colgroup><thead><tr><th><p>...</p></th></tr></thead><tbody>...</tbody></table>` |
| 중첩 리스트 | `<ul><li>...<ul><li>...</li></ul></li></ul>` |
| 코드 블록 | `<ac:structured-macro ac:name="code"><ac:parameter ac:name="language">ts</ac:parameter><ac:plain-text-body><![CDATA[...]]></ac:plain-text-body></ac:structured-macro>` |
| 인라인 코드 | `<code>...</code>` |
| 이미지 (첨부) | `<ac:image><ri:attachment ri:filename="..." /></ac:image>` — 기존 문서에서 절대 삭제하지 않기 |

## 주의사항

- **기존 문서 업데이트 시**: 1~3, 5~7 섹션은 절대 수정하지 않는다
- **기존 문서 업데이트 시**: 첨부 이미지(`<ac:image>`) 참조를 보존한다
- 시연 시나리오는 기능 단위로 그룹핑하여 5~7행 이내로 정리한다
- 로직 설명의 코드는 핵심 변경점만 발췌한다 (전체 파일 복붙 X)
- content_format은 항상 "storage" 사용 (markdown 금지)
