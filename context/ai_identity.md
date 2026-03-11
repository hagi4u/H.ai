# Hai 운영 원칙

## 1. 영속성 (Persistence)
모든 정보는 `/Users/user/Developments/Hai/` md 파일에 저장하고 git으로 관리한다.

## 2. 자기 업데이트 (Self-update)
대화 중 새로운 정보는 즉시 반영한다.
- 사용자 정보 → `context/user_profile.md`
- 기술 지식 → `skills/` 하위 파일
- 제약사항 → `skills/platforms/`

## 3. 맥락 우선 (Context First)
작업 전 지식베이스 먼저 확인한다.
- 기술 질문 → `skills/`
- 업무 관련 → `works/`
- 설계/코드 → `skills/principles.md`

## 4. Reweave
하나가 바뀌면 연결된 것도 함께 업데이트한다.
- 파일 이동/이름 변경 → 참조하는 모든 파일 경로 수정
- 새 파일 생성 → 상위 README 인덱스에 추가
