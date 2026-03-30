### JavaScript

---


# 자바스크립트 자료형

---

## 자료형

| 분류 | 자료형 | 설명 | 예시 |
|---|---|---|---|
| 원시 타입 (이뮤터블) | `string` | 문자열 | `"이용균"`, `'hello'` |
| 원시 타입 (이뮤터블) | `number` | 숫자 (정수 + 실수 통합) | `42`, `3.14` |
| 원시 타입 (이뮤터블) | `boolean` | 참/거짓 | `true`, `false` |
| 원시 타입 (이뮤터블) | `null` | 값이 없음 (의도적으로 비움) | `null` |
| 원시 타입 (이뮤터블) | `undefined` | 값이 할당 안 됨 | `let x;` |
| 원시 타입 (이뮤터블) | `bigint` | 매우 큰 정수 | `9007199254740991n` |
| 원시 타입 (이뮤터블) | `symbol` | 유일한 식별자 | `Symbol('id')` |
| 참조 타입 (뮤터블) | `object` | 키-값 쌍 | `{ name: "이용균" }` |
| 참조 타입 (뮤터블) | `array` | 배열 (object의 일종) | `[1, 2, 3]` |
| 참조 타입 (뮤터블) | `function` | 함수 (object의 일종) | `function() {}` |

> 이뮤터블 (Immutable): 값 자체 변경 불가, 재할당만 가능
>
> 뮤터블 (Mutable): 내부 값 직접 수정 가능

---

## 변수 선언 키워드

### let
- 재할당 가능
- 블록 스코프 (`{ }` 안에서만 유효)
- 값이 바뀔 필요 있을 때 사용

### const
- 재할당 불가
- 블록 스코프
- 기본적으로 const 사용 권장
- 단, 참조 타입(배열/객체)은 재할당만 막힐 뿐 내부 값 변경은 가능

```javascript
const arr = [1, 2, 3];
arr.push(4);      // 가능 (내부 값 변경, 뮤터블)
arr = [5, 6, 7];  // 에러 (재할당 불가)
```

### var (구식, 사용 비권장)
- 재할당 가능
- 함수 스코프 (블록 무시, 버그 유발)
- 호이스팅 문제 있음
- 현재는 let/const로 대체

---

## typeof

타입 확인용 연산자 (함수처럼 생겼지만 연산자임)

```javascript
typeof "hello"   // "string"
typeof 42        // "number"
typeof true      // "boolean"
typeof null      // "object" ← 유명한 버그, null인데 object 나옴
typeof undefined // "undefined"
```

- 괄호는 없어도 되나 관례상 붙여 쓰는 경우가 많음
- `null`이 `"object"`로 나오는 건 자바스크립트 초기 설계 버그

---

## 자바스크립트 주요 명령어

### 출력 / 디버깅

| 명령어 | 설명 |
|---|---|
| `console.log()` | 콘솔에 출력 |
| `console.error()` | 에러 메시지 출력 |
| `console.warn()` | 경고 메시지 출력 |
| `console.clear()` | 콘솔 화면 지우기 |

### 배열 메서드

| 명령어 | 설명 |
|---|---|
| `arr.push(x)` | 맨 뒤에 추가 |
| `arr.pop()` | 맨 뒤 제거 |
| `arr.shift()` | 맨 앞 제거 |
| `arr.unshift(x)` | 맨 앞에 추가 |
| `arr.map(fn)` | 각 요소 변환 후 새 배열 반환 |
| `arr.filter(fn)` | 조건 맞는 요소만 새 배열로 반환 |
| `arr.find(fn)` | 조건 맞는 첫 번째 요소 반환 |
| `arr.includes(x)` | 포함 여부 확인 (boolean) |
| `arr.length` | 배열 길이 |
| `arr.forEach(fn)` | 각 요소 순회 (반환값 없음) |
| `arr.reduce(fn)` | 누적 연산 |

### 문자열 메서드

| 명령어 | 설명 |
|---|---|
| `str.length` | 문자열 길이 |
| `str.toUpperCase()` | 대문자 변환 |
| `str.toLowerCase()` | 소문자 변환 |
| `str.includes(x)` | 포함 여부 확인 |
| `str.split(x)` | 구분자로 나눠서 배열 반환 |
| `str.trim()` | 앞뒤 공백 제거 |
| `str.replace(a, b)` | a를 b로 교체 |
| `str.slice(start, end)` | 부분 문자열 추출 |

### 객체 메서드

| 명령어 | 설명 |
|---|---|
| `Object.keys(obj)` | 키 목록 배열로 반환 |
| `Object.values(obj)` | 값 목록 배열로 반환 |
| `Object.entries(obj)` | 키-값 쌍 배열로 반환 |
| `Object.freeze(obj)` | 객체 수정 불가 (완전 이뮤터블) |

### 타입 변환

| 명령어 | 설명 |
|---|---|
| `Number(x)` | 숫자로 변환 |
| `String(x)` | 문자열로 변환 |
| `Boolean(x)` | 불리언으로 변환 |
| `parseInt(x)` | 정수로 변환 |
| `parseFloat(x)` | 실수로 변환 |
| `typeof x` | 타입 확인 |
