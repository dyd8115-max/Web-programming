# Iterators and Generators

---

## Iterator (반복자)

### Iterable Objects (반복 가능한 객체)

- `for/of` 루프와 spread operator (`...`)에서 동작 가능한 객체
- **내장 iterable**: Array, Set, Map
- Iterable object는 iteration protocol을 따름

### Iterator Protocol 동작 원리

```
┌─────────────────────┐
│  Iterable Object    │ (Array, Set, Map)
│ (for/of, ...에서)   │
└──────────┬──────────┘
           │ [Symbol.iterator] 메서드 호출
           ↓
┌─────────────────────┐
│   Iterator Object   │
│  (next() 메서드)     │
└──────────┬──────────┘
           │ next() 메서드 반복 호출
           ↓
┌─────────────────────┐
│ Iteration Result    │
│ { value, done }     │
└─────────────────────┘
```

### 단계별 작동 방식

**① Iterable 객체에서 Iterator 생성**
```javascript
const arr = [1, 2, 3];
const iterator = arr[Symbol.iterator]();  // Iterator 객체 반환
```

**② Iterator의 next() 메서드 반복 호출**
```javascript
iterator.next();  // { value: 1, done: false }
iterator.next();  // { value: 2, done: false }
iterator.next();  // { value: 3, done: false }
iterator.next();  // { value: undefined, done: true }
```

**③ done이 true가 될 때까지 반복**
- `value`: 현재 반복 단계의 값
- `done`: 반복 완료 여부 (true면 종료)

### Symbol.iterator

- JavaScript의 **Built-in Symbol** (기호)
- Iterable 객체가 반드시 가져야 할 메서드 키
- Iterator 객체를 반환하는 메서드

```javascript
// Array의 Symbol.iterator
const arr = [10, 20, 30];
const iter = arr[Symbol.iterator]();  // Iterator 객체 반환

// for/of는 내부적으로 이렇게 동작
for (const value of arr) {
    console.log(value);
}

// 위 코드는 다음과 동일
const iterator = arr[Symbol.iterator]();
let result = iterator.next();
while (!result.done) {
    console.log(result.value);
    result = iterator.next();
}
```

### Map과 Set의 Iterator

**Map (키-값 쌍)**
```javascript
const map = new Map([['a', 1], ['b', 2]]);

map.keys();     // 키만 반복
map.values();   // 값만 반복
map.entries();  // [키, 값] 쌍 반복

for (const [key, value] of map) {
    console.log(key, value);
}
```

**Set (중복 없는 값의 집합)**
```javascript
const set = new Set([1, 2, 3]);

for (const value of set) {
    console.log(value);
}
```

### Spread Operator와 Destructuring

```javascript
// Spread operator (...)
const arr = [1, 2, 3];
const expanded = [...arr];  // [1, 2, 3]

// Destructuring assignment
const [first, second, third] = arr;  // first=1, second=2, third=3
```

---

## Generator (생성자)

### Generator Function

- `function*` 키워드로 정의된 특수한 함수
- Generator 객체를 반환 (함수 본체는 즉시 실행되지 않음)
- **주의**: Generator function을 호출해도 함수 본체가 실행되지 않음

```javascript
function* generatorFunc() {
    console.log("이 메시지는 안 나타남");
    yield 1;
    yield 2;
}

const gen = generatorFunc();  // 함수 본체 미실행, Generator 객체만 반환
```

### Generator Object

- Iterator 객체와 유사한 구조
- `[Symbol.iterator]` 메서드와 `next()` 메서드를 가짐
- Iterator이므로 `for/of` 루프에서도 사용 가능

```
┌─────────────────────┐
│  Generator Object   │
│  (Iterator 유사)     │
└──────────┬──────────┘
           │ [Symbol.iterator]
           │ next()
           ↓
┌─────────────────────┐
│ Iteration Result    │
│ { value, done }     │
└─────────────────────┘
```

### yield 키워드

- Generator 함수 내에서만 사용 가능
- `return`과 유사하지만 함수를 완전히 종료하지 않고 **일시 중지** (suspend)
- `next()` 호출 시마다 다음 `yield`까지 실행

```javascript
function* countUp() {
    yield 1;
    yield 2;
    yield 3;
    // 마지막 next() 호출 시 done: true
}

const gen = countUp();
gen.next();  // { value: 1, done: false }
gen.next();  // { value: 2, done: false }
gen.next();  // { value: 3, done: false }
gen.next();  // { value: undefined, done: true }
```

### Generator 상태 관리

- **Closed** (닫힘): Generator가 생성되었거나 완료됨
- **Suspended** (일시 중지): `yield`에서 대기 중

```javascript
function* example() {
    console.log("시작");      // next() 호출 시 실행
    yield 1;                  // 여기서 일시 중지 (suspended)
    console.log("재개");      // 다음 next() 호출 시 실행
    yield 2;
    console.log("완료");
}

const gen = example();
gen.next();  // "시작" 출력 → { value: 1, done: false } 반환
gen.next();  // "재개" 출력 → { value: 2, done: false } 반환
gen.next();  // "완료" 출력 → { value: undefined, done: true } 반환
```

### Generator와 for/of

```javascript
function* range(n) {
    for (let i = 0; i < n; i++) {
        yield i;
    }
}

// for/of에서 직접 사용
for (const num of range(3)) {
    console.log(num);  // 0, 1, 2
}

// Spread operator에서 사용
const arr = [...range(5)];  // [0, 1, 2, 3, 4]
```

### 무한 Generator

```javascript
function* infiniteCounter() {
    let count = 0;
    while (true) {
        yield count++;
    }
}

const counter = infiniteCounter();
counter.next();  // { value: 0, done: false }
counter.next();  // { value: 1, done: false }
counter.next();  // { value: 2, done: false }
// 계속 진행 가능 (done이 true가 되지 않음)
```

### Shorthand Notation

```javascript
// 객체의 메서드로 generator 정의
const obj = {
    // Shorthand generator syntax
    *generator() {
        yield 1;
        yield 2;
    }
};

for (const value of obj.generator()) {
    console.log(value);  // 1, 2
}
```

---

## yield* Keyword

### yield*와 Recursive Generators

- `yield*` 키워드를 사용해 다른 iterable 객체를 순회하고 값을 yield
- Recursive generator 구현에 유용

```javascript
function* innerGen() {
    yield 1;
    yield 2;
}

function* outerGen() {
    yield* innerGen();  // innerGen의 모든 값을 yield
    yield 3;
}

for (const value of outerGen()) {
    console.log(value);  // 1, 2, 3
}
```

### 배열과의 조합

```javascript
function* flatten(arr) {
    for (const item of arr) {
        if (Array.isArray(item)) {
            yield* flatten(item);  // 재귀적 처리
        } else {
            yield item;
        }
    }
}

const nested = [1, [2, [3, 4]], 5];
const flat = [...flatten(nested)];  // [1, 2, 3, 4, 5]
```

### Spread Operator와의 차이

```javascript
function* gen() {
    yield 1;
    yield 2;
}

// yield* (Iterator 순회)
function* useYieldStar() {
    yield* gen();  // gen()의 모든 값을 yield
}

// spread operator (배열로 확장)
const arr = [...gen()];  // [1, 2]
```

---

## 요약

| 개념 | 설명 |
|---|---|
| **Iterable** | `[Symbol.iterator]` 메서드를 가진 객체 (Array, Set, Map) |
| **Iterator** | `next()` 메서드를 가진 객체, 반복 제어 |
| **Iteration Result** | `{ value, done }` 형태의 객체 |
| **Generator Function** | `function*` 문법, Generator 객체 반환 |
| **Generator Object** | Iterator와 유사, `yield`로 값 반환 |
| **yield** | Generator 함수를 일시 중지하고 값 반환 |
| **yield*** | 다른 iterable 객체를 순회하며 값 yield |
