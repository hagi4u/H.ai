# 설계문서 작성 스킬

PRD(기획문서)를 기반으로 SFE 설계문서를 Confluence에 생성하는 스킬입니다.
개발 착수 전, 기획 리뷰 및 테크스펙 사전체크를 위한 문서를 준비합니다.

## 사용법

```
/설계
```

입력하면 아래 정보를 물어본다:

```
🤖 설계문서를 준비할게요! 아래 정보를 알려주세요.

1. 📄 문서 타이틀:
2. 📋 기획문서(PRD) URL:
```

### 입력 예시

```
1. [26.x] 홈 탭 이미지 대응
2. https://wiki.navercorp.com/pages/viewpage.action?pageId=1234
```

## 워크플로우 위치

```
👉 /설계  ← 지금 여기
   ↓
   테크스펙 리뷰 (팀 미팅)
   ↓
   브랜치 생성 & 개발
   ↓
   /데모
```

## 작동 방식

### Step 1: 기획문서 분석

- `confluence_get_page` (convert_to_markdown: true) → 기획문서 내용 파싱
- 추출 대상:
  - 프로젝트 배경 및 목적
  - 요구사항 / 기능 정의
  - 피그마 시안 URL 또는 이미지
  - 기획자와 합의된 스펙 사항 (포함/제외 범위)
  - 일정, 관련 이해관계자

### Step 2: 설계문서 생성

`confluence_create_page` (space_key: "KREAM", content_format: "storage")

기획문서 분석 결과로 1~3 섹션을 채우고, 4~7은 placeholder로 둔다.

| 섹션 | 소스 | 작성 내용 |
|------|------|-----------|
| 1. 배경 | PRD | 요청 배경, 히스토리 요약. 참석자는 고정 (ri:user) |
| 2. Preview | PRD | 피그마 URL 있으면 widget 매크로, 없으면 PRD 링크 |
| 3. 스펙 설계 | PRD | ㄱ) 기획 협의 내용 요약, ㄴ) 요구사항 테이블, ㄷ) 빈 상태 (리뷰 미팅용), ㄹ) FE 관점 테크스펙 체크 |
| 4. 시연 | - | 빈 테이블 (데모 시 작성) |
| 5. 셀프 회고 | - | 빈 KPT 테이블 |
| 6. 피드백 | - | 빈 테이블 |
| 7. Reference | PRD | 기획문서 링크 |

### Step 3: 3. 스펙 설계 상세

**ㄱ) 기획 리뷰 협의 내용**
- PRD에서 결정된 스펙 사항 요약
- 포함 범위 / 제외 범위 구분

**ㄴ) 요구사항 정리**
- PRD의 요구사항을 화면/요구사항/비고 테이블로 구조화

**ㄷ) FE 설계 리뷰 문의 내용**
- 설계 리뷰 미팅에서 채우는 섹션 → **기본적으로 빈 상태로 둔다**
- 단, PRD를 읽고 웹 FE 관점에서 명확히 문의가 필요한 사항이 있으면 참고용으로 기재 가능
- 각 항목은 팀 리뷰에서 논의 후 ✅ 체크 처리할 수 있도록 구성

**ㄹ) 테크스펙 사전체크**
- **웹 FE 관점**에서 기술적 의사결정이 필요한 포인트를 체크리스트로 정리
- API 연동 관련 사항 (엔드포인트, 요청/응답 스펙, 파라미터 등)
- PRD에 iOS/Android 연동이 언급된 경우, 플랫폼 간 연동 포인트도 포함

### Step 4: 완료

- 생성된 문서 URL을 사용자에게 반환
- Bobby가 참석자 추가, Preview 시안 보강, 리뷰 포인트 조정 후 팀 리뷰에 사용

## 표준 템플릿 (storage format)

```html
<p><ac:structured-macro ac:name="toc" ac:schema-version="1"></ac:structured-macro></p>

<h2>1. 배경 (1m)</h2>
<ul>
  <li>요청 배경
    <ul>
      <li>(PRD에서 추출한 배경)</li>
    </ul>
  </li>
</ul>
<h3>참석자</h3>
<p><ac:link><ri:user ri:userkey="8a298b9290e994ce0190fac01a830007"></ri:user></ac:link>  <ac:link><ri:user ri:userkey="8a298a51873a354c01883f8c8d0e00f3"></ri:user></ac:link> <ac:link><ri:user ri:userkey="8a298a518ab74c4c018ac86dd46408c8"></ri:user></ac:link> <ac:link><ri:user ri:userkey="8a298b92923e458b0192633d425a0059"></ri:user></ac:link> <ac:link><ri:user ri:userkey="8a298b92893f77ed018983fd2fdb003b"></ri:user></ac:link> <ac:link><ri:user ri:userkey="8af0661276ae12e80176baf367d40076"></ri:user></ac:link> <ac:link><ri:user ri:userkey="8aeba715978cd1930197998261dc000a"></ri:user></ac:link> <ac:link><ri:user ri:userkey="8a298a5191317c860191aefeaf870366"></ri:user></ac:link> </p>
<p><br/></p>

<h2>2. Preview (10m)</h2>
<p>(PRD에서 추출한 피그마 링크 또는 widget 매크로)</p>
<p><br/></p>

<h2>3. 스펙 설계</h2>
<h3>ㄱ) 기획 리뷰 협의 내용</h3>
<ul><li>(PRD 기반 협의 내용)</li></ul>
<h3>ㄴ) 요구사항 정리</h3>
<table class="wrapped"><colgroup><col/><col/><col/></colgroup><thead><tr><th><p>화면</p></th><th><p>요구사항</p></th><th><p>비고</p></th></tr></thead><tbody><tr><td><p><br/></p></td><td><p><br/></p></td><td><p><br/></p></td></tr></tbody></table>
<h3>ㄷ) FE 설계 리뷰 문의 내용</h3>
<ul><li>(FE 관점 리뷰 포인트)</li></ul>
<h3>ㄹ) 테크스펙 사전체크</h3>
<ul><li>(사전체크 항목)</li></ul>

<h2>4. 시연</h2>
<table class="wrapped"><colgroup><col/><col/><col/></colgroup><thead><tr><th><p>기능</p></th><th><p>시나리오</p></th><th><p>비고</p></th></tr></thead><tbody><tr><td><p><br/></p></td><td><p><br/></p></td><td><p><br/></p></td></tr></tbody></table>
<h3>로직 설명</h3>
<p>(데모 시 작성)</p>

<h2>5. 셀프 회고 (5m)</h2>
<table class="wrapped"><colgroup><col/><col/><col/><col/></colgroup><thead><tr><th><p><br/></p></th><th><p>Keep</p></th><th><p>Problem</p></th><th><p>Try</p></th></tr></thead><tbody><tr><td><div class="content-wrapper"><p><ac:link><ri:user ri:userkey="8a298b9290e994ce0190fac01a830007"></ri:user></ac:link> </p></div></td><td><p><br/></p></td><td><p><br/></p></td><td><p><br/></p></td></tr></tbody></table>

<h2>6. 피드백 및 개선사항</h2>
<table class="wrapped relative-table" style="width: 56.0887%;"><colgroup><col style="width: 18.5612%;"/><col style="width: 46.9784%;"/><col style="width: 34.5324%;"/></colgroup><thead><tr><th><p>작성자</p></th><th><p>피드백 및 개선사항</p></th><th><p>비고</p></th></tr></thead><tbody><tr><td><div class="content-wrapper"><p><ac:link><ri:user ri:userkey="8a298a51873a354c01883f8c8d0e00f3"></ri:user></ac:link></p></div></td><td><p><br/></p></td><td><p><br/></p></td></tr><tr><td><div class="content-wrapper"><p><ac:link><ri:user ri:userkey="8aeba715978cd1930197998261dc000a"></ri:user></ac:link></p></div></td><td><div class="content-wrapper"><br/></div></td><td><p><br/></p></td></tr><tr><td><div class="content-wrapper"><p><ac:link><ri:user ri:userkey="8af0661276ae12e80176baf367d40076"></ri:user></ac:link>  </p></div></td><td><p><br/></p></td><td><p><br/></p></td></tr><tr><td><div class="content-wrapper"><p><ac:link><ri:user ri:userkey="8a298a518ab74c4c018ac86dd46408c8"></ri:user></ac:link></p></div></td><td><p><br/></p></td><td><p><br/></p></td></tr><tr><td><div class="content-wrapper"><p><ac:link><ri:user ri:userkey="8a298a5191317c860191aefeaf870366"></ri:user></ac:link> </p></div></td><td><p><br/></p></td><td><p><br/></p></td></tr><tr><td><div class="content-wrapper"><p><ac:link><ri:user ri:userkey="8a298b92923e458b0192633d425a0059"></ri:user></ac:link></p></div></td><td><p><br/></p></td><td><p><br/></p></td></tr><tr><td><div class="content-wrapper"><p><ac:link><ri:user ri:userkey="8a298b92893f77ed018983fd2fdb003b"></ri:user></ac:link></p></div></td><td><p><br/></p></td><td><p><br/></p></td></tr></tbody></table>

<h2>7. Reference</h2>
<ul><li><a href="(PRD URL)">(PRD 문서 제목)</a></li></ul>
```

## Confluence storage format 규칙

| 요소 | 사용할 태그 |
|------|-------------|
| 테이블 | `<table><colgroup><col/></colgroup><thead><tr><th><p>...</p></th></tr></thead><tbody>...</tbody></table>` |
| 중첩 리스트 | `<ul><li>...<ul><li>...</li></ul></li></ul>` |
| 코드 블록 | `<ac:structured-macro ac:name="code"><ac:parameter ac:name="language">ts</ac:parameter><ac:plain-text-body><![CDATA[...]]></ac:plain-text-body></ac:structured-macro>` |
| 인라인 코드 | `<code>...</code>` |
| 외부 링크 | `<a href="URL">텍스트</a>` |

## 주의사항

- content_format은 항상 "storage" 사용 (markdown 금지)
- 기획문서의 이미지/피그마 링크는 가능한 한 그대로 참조한다
- FE 리뷰 포인트는 Bobby가 팀 리뷰 전에 수정할 수 있도록 구성한다
- 4~7 섹션은 빈 placeholder로만 둔다 (데모 스킬에서 채움)
