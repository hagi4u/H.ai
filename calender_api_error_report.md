# 캘린더 API 오류 레포트

## 발생 일시
2026년 3월 9일 09:20

## 시스템 정보
- API: 네이버 웍스 캘린더 API (mcp__naver-works__calendar_list_events)
- 조회 대상일: 2026-03-09 (월요일)
- 캘린더 ID: c_1001_0d3fa539-87bc-45a2-bd8d-c371783291b9 (KREAM Frontend)

## 오류 현상

### 1. 반복 일정 반환 오류
특정 날짜(오늘)의 일정을 조회했음에도 불구하고, 서로 다른 날짜의 두 개 이벤트가 반환됨:
- 첫 번째 이벤트: 2026-01-12 생성된 기본 반복 일정
- 두 번째 이벤트: 2026-02-09의 예외 처리된 인스턴스

### 2. 날짜 불일치
요청한 날짜(2026-03-09)의 일정이 아닌, 과거 날짜의 일정 정보를 반환

### 3. Recurrence Rule 분석
```json
"recurrence": [
  "RRULE:FREQ=WEEKLY;INTERVAL=1;BYDAY=MO",
  "EXDATE;TZID=Asia/Seoul:20260209T150000"
]
```
- 매주 월요일 반복
- 2026-02-09 월요일은 예외 처리됨

## 원인 분석

### 1. API의 반복 일정 처리 방식
- 네이버 웍스 캘린더 API가 반복 일정을 조회 시,
  - 기본 이벤트 정보와
  - 예외 처리된 인스턴스 정보를 함께 반환하는 것으로 보임
- 이로 인해 요청한 날짜와 관계없는 과거 인스턴스가 포함됨

### 2. 시스템 설계상의 한계
- 특정 날짜 범위 Filter 로직이 반복 일정의 예외 인스턴스를 제대로 처리하지 못함
- `recurringEventId` 필드가 있으나, 시스템이 이를 활용해 적절히 필터링하지 못함

## 영향

### 1. 사용자 경험
- 오늘의 일정이 아닌 과거 일정이 표시되어 혼동 발생
- "공휴일로 취소"와 같은 잘못된 정보 제공 가능성

### 2. 데이터 정확성
- 실제 일정과 표시되는 정보 간의 불일치
- 중복된 이벤트 정보로 인한 혼란

## 보완 방안

### 1. 단기 해결책 (Application Layer)
```javascript
// 클라이언트 측에서 날짜 필터링 강화
function filterEventsByDate(events, targetDate) {
  return events.filter(event => {
    const eventStart = new Date(event.start.dateTime);
    const eventEnd = new Date(event.end.dateTime);
    const target = new Date(targetDate);

    // 해당 날짜의 00:00:00부터 23:59:59까지 범위 필터링
    return eventStart.toDateString() === target.toDateString();
  });
}
```

### 2. 중기 해결책 (API Wrapper)
```javascript
// API Wrapper 레이어 구현
class CalendarAPI {
  async getEvents(calendarId, fromDate, toDate) {
    const rawEvents = await this.client.listEvents(calendarId, fromDate, toDate);

    // 반복 일정 처리 로직
    const processedEvents = this.processRecurringEvents(rawEvents, fromDate, toDate);

    return processedEvents;
  }

  processRecurringEvents(events, fromDate, toDate) {
    // RRULE 파싱하여 실제 발생 일정 계산
    // EXDATE 처리하여 예외 일정 제외
    // 날짜 범위 내의 인스턴스만 반환
  }
}
```

### 3. 장기 해결책 (Backend Enhancement)

#### 가. API 개선 요청
- 네이버 웍스 팀에 API 개선 요청
  - 특정 날짜 범위의 실제 인스턴스만 반환하도록 수정
  - `includePastInstances`, `includeFutureInstances`, `includeExceptions`
    파라미터 추가 제안

#### 나. 자체 캐싱 시스템 구축
```javascript
// 캘린더 이벤트 캐싱 시스템
class CalendarCacheService {
  async syncEvents(calendarId, from, to) {
    // 1. API에서 전체 반복 일정 획득
    // 2. RRULE/EXDATE 파싱하여 확장
    // 3. 특정 기간의 인스턴스 생성 및 캐싱
    // 4. 실시간 업데이트를 위한 Webhook 연동
  }
}
```

### 4. 데이터 모델 개선
```javascript
// 정규화된 이벤트 데이터 모델
interface CalendarEvent {
  id: string;
  title: string;
  isRecurring: boolean;
  recurrenceRule?: string;
  exceptionDates?: Date[];
  instances: EventInstance[];
}

interface EventInstance {
  id: string;
  baseEventId: string;
  start: Date;
  end: Date;
  isModified: boolean;
  isCancelled: boolean;
}
```

## 권장 조치 순서

1. **즉시 (긴급)**: 클라이언트 측 날짜 필터링 로직 적용 (NaverWorks MCP 레벨)
2. **단기 (1-2주)**: API Wrapper 구현으로 일관성 있는 처리 로직 마련
3. **중기 (1개월)**: 캐싱 시스템 도입으로 성능 및 안정성 개선
4. **장기 (3개월)**: API 개선 요청 및 자체 스케줄링 시스템 고려

## 모니터링 방안

1. 오류 탐지: 날짜 불일치 이벤트에 대한 알림 시스템
2. 성능 모니터링: 반복 일정 처리 시간 측정
3. 정확도 검증: API 결과와 실제 캘린더 비교 자동화

## 첨부
- API 응답 Raw 데이터 (보안을 위해 다 부분 마스킹 필요)
- 오류 재현 테스트 케이스