# JavaScript 패턴

## Typed Array

### Int32Array
32비트 정수값만 저장하는 고정 길이 배열. `ArrayBuffer` 기반으로 연속 메모리를 사용해 일반 Array보다 빠르다.

```js
const arr = new Int32Array(3)          // [0, 0, 0]
const arr2 = new Int32Array([1, 2, 3]) // 초기값 지정
```

**값 변환 규칙 (에러 없이 조용히 변환)**
```js
arr[0] = 1.9         // → 1        (소수점 버림)
arr[0] = 2147483648  // → -2147483648  (overflow, 범위: -2,147,483,648 ~ 2,147,483,647)
arr[0] = 'hello'     // → 0        (문자 → 0)
```

**일반 Array와 차이**
| | `Array` | `Int32Array` |
|--|---------|-------------|
| 타입 | 무엇이든 | 32비트 정수만 |
| 메모리 | 동적 | 고정 (요소당 4byte) |
| 크기 변경 | 가능 | 불가능 |

**사용 가능한 메서드**: `forEach`, `map`, `filter`, `includes` 등
**사용 불가**: `push`, `pop`, `splice` (크기 고정이라 TypeError)

**주요 사용처**: 바이너리 데이터 처리, WebAssembly 연동, SharedArrayBuffer(멀티스레드), 대량 수치 연산
