### JavaScript

---


# 자바스크립트 자료형

---

## 자료형

| 분류 | 자료형 | 설명 | 예시 |
|---|---|---|---|
| 원시 타입 (이뮤터블) | `string` | 문자열 | `"이용균"`, `'hello'` |
| | `number` | 숫자 (정수 + 실수 통합) | `42`, `3.14` |
| | `boolean` | 참/거짓 | `true`, `false` |
| | `null` | 값이 없음 (의도적으로 비움) | `null` |
| | `undefined` | 값이 할당 안 됨 | `let x;` |
| | `bigint` | 매우 큰 정수 | `9007199254740991n` |
| | `symbol` | 유일한 식별자 | `Symbol('id')` |
| 참조 타입 (뮤터블) | `object` | 키-값 쌍 | `{ name: "이용균" }` |
| | `array` | 배열 (object의 일종) | `[1, 2, 3]` |
| | `function` | 함수 (object의 일종) | `function() {}` |

> 이뮤터블 (Immutable): 값 자체 변경 불가, 재할당만 가능
>
> 뮤터블 (Mutable): 내부 값 직접 수정 가능

---

## 변수 선언 키워드

### let
- 재할당 가능
- 값이 바뀔 필요 있을 때 사용

### const
- 재할당 불가(상수)
- 단, 참조 타입(배열/객체)은 재할당만 막힐 뿐 내부 값 변경은 가능

```javascript
const user = { name: "이용균" };
user.name = "이균용";  // 수정 가능 → { name: "이균용" }
user.age = 24;         // 추가 가능 → { name: "이균용", age: 24 }
delete user.name;      // 삭제 가능 → { age: 24 }
user = { name: "이균용" }; // 에러! → 재할당 불가(변수가 가리키는 값 자체를 새로운 값으로 교체하는 것 불가능)
```

> 블록 스코프: { } 안에서 선언된 변수는 { } 밖에서 접근 불가(let, const가 사용하는 스코프 방식)

### var (구식, 사용 비권장)
- 재할당 가능
- 함수 스코프 (블록 무시, 버그 유발)
- 호이스팅 문제 있음
- 현재는 let/const로 대체

> 함수스코프: { } 안에서만 유효한 범위, var가 사용하는 스코프 방식( if, for 같은 블록은 무시)
> ```javascript
> if (true) {
>     var x = 10;
> }
> console.log(x);  // 10 (에러 안남, 블록 밖인데 접근 가능, 블록 스코프 위반)
> ```
> 호이스팅: 변수나 함수 선언이 코드 실행 전에 맨 위로 끌어올려지는 것처럼 동작하는 현상
>
> ```javascript
> console.log(x);  // undefined (에러 안남!)
> var x = 10;
> console.log(x);  // 10
> ```
> ```javascript
> var x;           // 선언이 위로 끌어올려짐
> console.log(x);  // undefined
> x = 10;          // 할당은 그대로
> console.log(x);  // 10
> ```

---

## 객체(Object)

### 객체 생성 방법

**1. with object literals (객체 리터럴) - 가장 많이 씀**

- `{ }` 안에 키:값을 직접 나열해서 객체를 만드는 방식
- 간결하고 직관적이라 실무에서 가장 많이 사용

```javascript
const user = {
    name: "이용균",
    age: 25
};
```

**2. with new keyword (new 키워드)**

2-1. `new Object()` - 빈 객체를 만들고 프로퍼티를 하나씩 추가하는 방식, 거의 안 씀

```javascript
const user = new Object();
user.name = "이용균";
user.age = 25;
```

2-2. 생성자 함수 - 같은 구조의 객체를 여러 개 찍어낼 때 사용, `new` 키워드로 호출

```javascript
function User(name, age) {
    this.name = name;
    this.age = age;
}
const user = new User("이용균", 25);
```

2-3. 클래스 (ES6+) - 생성자 함수와 내부 동작은 같지만 더 명확한 문법, 현재 주로 사용

```javascript
class User {
    constructor(name, age) {
        this.name = name;
        this.age = age;
    }
}
const user = new User("이용균", 25);
```

**생성자 함수 vs 클래스 차이점**

| 구분 | 생성자 함수 | 클래스 (ES6+) |
|---|---|---|
| `new` 없이 호출 | 에러 안남 (버그 유발) | 에러 발생 (안전) |
| 호이스팅 | 됨 (선언 전에 사용 가능) | 안됨 (선언 전 사용 불가) |
| 상속 | `prototype` 직접 조작 | `extends` 키워드로 간결하게 |
| 가독성 | Java 등 다른 언어와 문법 다름 | Java 클래스 문법과 유사 |
| 현재 사용 | 레거시 코드에서 볼 수 있음 | 현재 표준, 실무 주로 사용 |

```javascript
// new 없이 호출했을 때 차이
function User(name) { this.name = name; }
User("이용균");   // 에러 안남 → this가 전역객체(window)가 되어 버그 유발

class User2 {
    constructor(name) { this.name = name; }
}
User2("이용균");  // 에러! → TypeError: Class constructor must be called with 'new'
```

**3. with Object.create() function**

- 특정 객체를 프로토타입으로 지정해서 새 객체를 만드는 방식
- 프로토타입을 직접 제어할 때 사용

```javascript
const proto = {
    greet() {
        return `안녕, 나는 ${this.name}`;
    }
};
const user = Object.create(proto);
user.name = "이용균";
user.greet(); // "안녕, 나는 이용균" → proto에서 찾아옴
```

> `Object.create(null)`: 프로토타입 없는 완전히 순수한 객체 생성 (toString 같은 내장 메서드도 없음)

---

### 프로토타입 (Prototype)

- 객체를 생성하면 메모리에 두 영역이 생김

```
[ 객체 메모리 영역 ]
┌─────────────────────────────┐
│  개발자가 넣은 속성값        │  ← name, age 등 직접 정의한 프로퍼티
├─────────────────────────────┤
│  [[Prototype]] 링크          │  ← 상속받은 프로토타입 영역 (Object.prototype)
└─────────────────────────────┘
```

- 프로퍼티/메서드를 찾을 때 첫 번째 영역(직접 정의한 것)에 없으면 → 두 번째 영역(프로토타입)에서 찾음, 이걸 **프로토타입 체인**이라고 함

**프로토타입 체인 예시**

- `toString()`은 직접 만든 적 없지만, JS가 프로토타입 체인을 타고 올라가 `Object.prototype`에서 찾아와서 실행됨

```javascript
const user = { name: "이용균" };
user.toString(); // "[object Object]" → Object.prototype.toString 실행됨
```

```
user (name만 있음)
  ↓ toString 없음 → 위로 올라감
Object.prototype (toString, hasOwnProperty 등 내장 메서드 여기 있음)
  ↓
null (체인 끝, 여기까지 없으면 undefined)
```

- 배열도 동일한 방식 → `push`, `map` 등을 직접 만든 적 없어도 쓸 수 있는 이유

```
arr
  ↓
Array.prototype (push, pop, map, filter 등)
  ↓
Object.prototype
  ↓
null
```

**프로토타입 확인 방법**

- `Object.getPrototypeOf(obj)`: 표준 방식, 현재 권장
- `obj.__proto__`: 구식 방식, 비권장 (브라우저 호환용으로 남아있음)

```javascript
const user = { name: "이용균" };
Object.getPrototypeOf(user) === Object.prototype; // true
user.__proto__ === Object.prototype;              // true (결과 같지만 비권장)
```

> Java의 모든 클래스가 암묵적으로 Object를 상속하는 것처럼, JS도 모든 객체가 Object.prototype을 프로토타입으로 가짐

---

### 객체 프로퍼티

- 프로퍼티: 객체 안에 저장된 데이터로, 키: 값 형태로 구성된 것

```javascript
const user = {
    name: "이용균",  // 키: name, 값: "이용균"
    age: 25,         // 키: age, 값: 25
    role: "backend"  // 키: role, 값: "backend"
}
user.name   // "이용균" (점 표기법으로 접근)
user["age"] // 25 (괄호 표기법으로 접근)
```

### 프로퍼티 조작

```javascript
const user = { name: "이용균" };
user.age = 25;        // 추가
user.name = "이균용"; // 수정
delete user.age;      // 삭제
```

### 프로퍼티 열거

```javascript
const user = { name: "이용균", age: 25 };

Object.keys(user);    // ["name", "age"]
Object.values(user);  // ["이용균", 25]
Object.entries(user); // [["name", "이용균"], ["age", 25]]

for (let key in user) {
    console.log(key, user[key]); // name 이용균 / age 25
}
```

---

### 프로퍼티 디스크립터 (Property Descriptor)

- 프로퍼티는 키:값 외에 내부적으로 속성 3개를 가짐

**우리가 만든 객체의 기본값**

| 속성 | 의미 | 기본값 |
|---|---|---|
| `writable` | 값 수정 가능 여부 | `true` ✅ |
| `enumerable` | `for...in` 루프에 나타나는지 (출력/열거 가능 여부) | `true` ✅ |
| `configurable` | 프로퍼티 삭제/속성 변경 가능 여부 | `true` ✅ |

```javascript
const user = { name: "이용균" };
Object.getOwnPropertyDescriptor(user, "name");
// { value: "이용균", writable: true, enumerable: true, configurable: true }
```

**JS Built-in 객체(내장 객체)는 기본값이 반대 → 외부에서 건드릴 수 없게 잠겨있음**

- 내장 객체(`Array`, `Object` 등)는 `.prototype` 안에 메서드들을 갖고 있음
- 이 메서드들이 프로토타입 체인을 통해 우리가 만든 객체에서도 사용 가능한 것
- 단, 실수로 덮어쓰거나 삭제하면 안 되기 때문에 모두 잠겨있음

| 속성 | 우리가 만든 객체 | JS Built-in 객체 |
|---|---|---|
| `writable` | `true` ✅ 수정 가능 | `false` ❌ read-only → 값 덮어쓰기 불가 |
| `enumerable` | `true` ✅ 열거 가능 | `false` ❌ non-enumerable → `for...in` 출력 안됨 |
| `configurable` | `true` ✅ 삭제/변경 가능 | `false` ❌ non-configurable → 삭제/변경 불가 |

```javascript
// enumerable: false 예시
// push, pop 등은 Array.prototype에 있지만 for...in에 출력 안됨
const arr = [1, 2, 3];
for (let key in arr) {
    console.log(key); // "0", "1", "2" 만 출력, push/pop 등은 안 나옴
}
```

> `Object.freeze(obj)`: writable, configurable을 false로 바꿔 객체를 완전히 잠금

---

### use strict (엄격 모드)

- JS는 기본적으로 느슨한 모드(sloppy mode)로 동작 → 암묵적으로 허용되는 것들이 많음
- `"use strict"` 선언 시 엄격 모드 활성화 → 실수를 에러로 잡아줌

```javascript
"use strict";

x = 10;         // 에러! → 선언 없이 변수 사용 불가 (느슨한 모드에서는 허용됨)
delete Object;  // 에러! → built-in 객체 삭제 불가
```

| 구분 | 느슨한 모드 (기본) | 엄격 모드 (`use strict`) |
|---|---|---|
| 선언 없이 변수 사용 | 허용 (전역변수로 생성) | 에러 |
| `this` (일반 함수) | `window` (전역 객체) | `undefined` |
| 중복 매개변수 | 허용 | 에러 |

> 파일 최상단 또는 함수 최상단에 `"use strict";` 문자열 선언으로 활성화
>
> ES6 모듈(`import`/`export`) 사용 시 자동으로 엄격 모드 적용

---

## 스프레드 연산자 (Spread Operator)

- 스프레드 연산자(`...`): 배열이나 객체를 펼쳐서 요소를 꺼내는 연산자

```javascript
// 배열
let arr = [1, 2, 3];
let arr2 = [...arr, 4, 5];        // [1, 2, 3, 4, 5]

// 객체
let user = { name: "이용균" };
let user2 = { ...user, age: 25 }; // { name: "이용균", age: 25 }

// 객체 복사
let original = { name: "이용균" };
let copy = { ...original };       // 새 객체로 복사
```

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

### 타입 관련 etc

| 명령어 | 설명 |
|---|---|
| `Number(x)` | 숫자로 변환 |
| `String(x)` | 문자열로 변환 |
| `Boolean(x)` | 불리언으로 변환 |
| `parseInt(x)` | 정수로 변환 |
| `parseFloat(x)` | 실수로 변환 |
| `typeof x` | 타입 확인 |

> `typeof`는 괄호 없이 써도 되나 관례상 붙여 쓰는 경우가 많음
>
> `null`은 원시 타입이지만 `typeof null`이 `"object"`로 나오는 자바스크립트 설계 버그가 있음
>
> ```javascript
> typeof "hello"   // "string"
> typeof 42        // "number"
> typeof true      // "boolean"
> typeof null      // "object" ← 버그
> typeof undefined // "undefined"
> ```
