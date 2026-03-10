# Naver Corp Jira (jira.navercorp.com) - Server/DC

## Issue Types (한국어 이름 사용 필수)

- `큰틀` = Epic에 해당 (link_to_epic 도구는 "not an Epic" 에러 → customfield로 직접 설정)
- `작업` = Task (영문 "Task" 사용 시 "issue type is required" 에러 발생)
- `부작업` = Sub-task (API에서는 `Sub-task`로 생성하면 `부작업`으로 생성됨)
- 기타: `버그`, `스토리` 등

## Issue 생성 시 주의사항

- `issue_type` 파라미터에 영문 이름("Task") 사용 불가 → 한국어("작업") 사용
- Sub-task 생성 시: `issue_type="Sub-task"`, `additional_fields={"parent": "PARENT-KEY"}` (parent는 문자열로 키만 전달)
- Epic(큰틀) 하위에 이슈 연결: `jira_update_issue`로 Epic Link 커스텀 필드 직접 설정
  - `additional_fields={"customfield_213534": "EPIC-KEY"}` (customfield_213534 = Epic Link 필드)

## Issue Link Types

- `부모-자식` (is child of / is parent of) - 큰틀 하위 이슈 연결에 사용
- `관련된 이슈` (relates to)
- `블로킹` (blocks / is blocked by)
- `의존된 이슈` (depends / is depended on)
- `중복된 이슈` (duplicates)
- `이슈 분할` (분할)
- `클로너` (clones)
- `Problem/Incident` (causes)
