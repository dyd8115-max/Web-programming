# Iterators and Generators

---

## Iterable (반복 가능한 객체)

### 정의

- 어떤 객체에 `[Symbol.iterator]()` 메서드가 있으면, 그 객체를 **Iterable** 이라고 한다.
- Iterable 객체는 `for/of` 루프와 spread operator(`...`)에서 사용할 수 있다.

### Iterable의 종류

| 타입 | 설명 | 예시 |
|---|---|---|
| **Array** | 배열 | `[1, 2, 3]` |
| **String** | 문자열 | `"hello"` |
| **Set** (집합) | 중복 없는 값의 집합 | `new Set([1, 2, 3])` |
| **Map** (맵) | 키-값 쌍 | `new Map([['a', 1]])` |
| **Generator** (생성기) | function*으로 생성 | `function* gen() {}` |

---

## Iterator (반복자)

### 정의

- **Iterator** 는 반복을 제어하는 객체다.
- `next()` 메서드를 가지고 있으며, 호출할 때마다 `{ value, done }` 형태의 객체를 반환한다.
- `value`: 현재 값
- `done`: 반복이 끝났는지 여부 (true = 끝남, false = 계속)

### Iterator의 종류

| 타입 | 설명 | 생성 방법 |
|---|---|---|
| **Array Iterator** | 배열의 Iterator | `arr[Symbol.iterator]()` |
| **String Iterator** | 문자열의 Iterator | `str[Symbol.iterator]()` |
| **Set Iterator** | Set의 Iterator | `set[Symbol.iterator]()` |
| **Map Iterator** | Map의 Iterator | `map[Symbol.iterator]()` |
| **Generator Object** | Generator 함수에서 생성 | `function* gen() {}` 호출 |

### Iterable과 Iterator의 관계

Iterable 객체와 Iterator 객체는 다음과 같은 관계를 가진다.

**① Iterable 객체 생성**
- Array, Set, Map, String 등의 Iterable 객체는 `[Symbol.iterator]()` 메서드를 가지고 있다.
- 이 메서드는 Iterator 객체를 생성하고 반환한다.

**② Iterator 객체 획득**
- Iterable 객체의 `[Symbol.iterator]()` 메서드를 호출하면 Iterator 객체를 얻는다.
- Iterator 객체는 `next()` 메서드를 가지고 있다.

**③ Iteration Result 객체 반환**
- Iterator 객체의 `next()` 메서드를 호출할 때마다 Iteration Result 객체가 반환된다.
- Iteration Result 객체는 `{ value: 값, done: true/false }` 형태이다.
- `value`: 현재 반복 단계의 값
- `done`: 반복이 완료되었는지 여부 (true = 완료, false = 계속)

**④ 반복 제어**
- `done`이 false인 동안 `next()`를 계속 호출하여 반복을 진행한다.
- `done`이 true가 되면 반복을 종료한다.

### 실제 동작 예시

```javascript
const arr = [10, 20, 30];

// Step 1: Iterable에서 Iterator 생성
const iterator = arr[Symbol.iterator]();

// Step 2: next() 반복 호출
iterator.next();  // { value: 10, done: false }
iterator.next();  // { value: 20, done: false }
iterator.next();  // { value: 30, done: false }
iterator.next();  // { value: undefined, done: true }
```

> Iterator 객체는 **반복 상태를 기억**한다.
> 매번 `next()`를 호출할 때마다 이전의 위치를 기억하고 다음 값을 반환한다.

### Iterator의 상태 활용

```javascript
const arr = [1, 2, 3, 4, 5];
const iterator = arr[Symbol.iterator]();

// 처음 2개 값 꺼내기
iterator.next();  // { value: 1, done: false }
iterator.next();  // { value: 2, done: false }

// Iterator를 spread operator로 펼치면 남은 값만 배열 생성
const remaining = [...iterator];
console.log(remaining);  // [3, 4, 5]
```

> Iterator는 이미 반환한 값들을 기억하고 있다.
> 따라서 `next()`로 꺼낸 값들은 제외되고, 남은 값들만 새 배열에 포함된다.

---

## Iterable vs Iterator 정리

**Iterable** 과 **Iterator** 는 서로 다른 역할을 한다.

| 구분 | Iterable | Iterator |
|---|---|---|
| **역할** | 반복할 데이터를 가진 객체 | 반복을 제어하는 객체 |
| **필수 메서드** | `[Symbol.iterator]()` (Prototype) | `next()` (Prototype) |
| **반환값** | Iterator 객체 | `{ value, done }` |
| **예시** | Array, Set, Map | arr[Symbol.iterator]() 결과 |

---

## Symbol.iterator

### 정의

- `Symbol.iterator`는 JavaScript의 **내장 기호(Built-in Symbol)**이다.
- Iterable 객체가 Iterator를 반환하는 메서드를 정의할 때 사용되는 키다.
- 모든 Iterable 객체는 반드시 `[Symbol.iterator]()` 메서드를 구현해야 한다.

### 사용 예시

```javascript
const arr = [1, 2, 3];

// Symbol.iterator 메서드는 배열에 내장되어 있음
const iterator = arr[Symbol.iterator]();

// 이를 통해 Iterator 객체를 얻을 수 있음
iterator.next();  // { value: 1, done: false }
```

---

## Spread Operator 활용

### 정의

- Spread operator는 Iterable 객체의 모든 값을 펼쳐서 개별 요소로 만드는 문법이다.

### 배열 복사

```javascript
const arr = [1, 2, 3];
const copy = [...arr];  // 새로운 배열 생성

copy[0] = 99;
console.log(arr);   // [1, 2, 3] (원본 유지)
console.log(copy);  // [99, 2, 3] (copy만 변경)
```

> spread operator(`...`)는 완전히 새로운 배열 객체를 생성한다.
> 따라서 `copy`와 `arr`은 서로 다른 배열이고, 한쪽을 수정해도 다른 쪽에 영향이 없다.

### 배열 합치기

```javascript
const arr1 = [1, 2];
const arr2 = [3, 4];

// spread 사용
const merged = [...arr1, ...arr2];
console.log(merged);  // [1, 2, 3, 4]

// spread 미사용
const notMerged = [arr1, arr2];
console.log(notMerged);  // [[1, 2], [3, 4]] - 2차원 배열
```

> spread operator 없이 배열을 감싸면 중첩된 배열이 생성된다.
> spread operator는 배열을 펼쳐서 평탄화한다.

### 배열에서의 추가 활용

| 용도 | 코드 | 결과 |
|---|---|---|
| **새 요소 추가** | `[...arr, 4, 5]` | 끝에 요소 추가 |
| **앞에 추가** | `[0, ...arr]` | 앞에 요소 추가 |
| **정렬 (원본 보존)** | `[...arr].sort()` | 원본은 유지, 정렬본만 생성 |

### 객체에서의 사용

```javascript
const obj = {a: 1, b: 2};

// 객체 복사
const copy = {...obj};

// 객체 합치기
const merged = {...obj, c: 3};  // {a: 1, b: 2, c: 3}

// 속성 덮어쓰기
const updated = {...obj, b: 99};  // {a: 1, b: 99}
```

### 함수 인자로의 사용

```javascript
function sum(a, b, c) {
    return a + b + c;
}

const nums = [1, 2, 3];
sum(...nums);  // sum(1, 2, 3)과 동일 → 6
```

### 문자열에서의 사용

```javascript
const str = "hello";
const chars = [...str];  // ['h', 'e', 'l', 'l', 'o']
```

### 주의: 얕은 복사 (Shallow Copy)

```javascript
const nested = {a: {b: 1}};
const copy = {...nested};

copy.a.b = 2;
console.log(nested.a.b);  // 2 - 원본도 변경됨
```

> 얕은 복사는 최상위 객체만 복사하고, 중첩된 객체/배열은 참조만 복사된다.

---

## Spread Operator 함수 활용

### Math 함수 사용

| 함수 | 설명 | 예시 |
|---|---|---|
| `Math.max(...arr)` | 배열에서 최댓값 찾기 | `Math.max(...[5, 2, 8])` → 8 |
| `Math.min(...arr)` | 배열에서 최솟값 찾기 | `Math.min(...[5, 2, 8])` → 2 |
| `Array.push(...arr)` | 배열에 여러 요소 추가 | `arr.push(...[4, 5])` |
| `Array.concat(...arr)` | 배열 합치기 | `[...arr1, ...arr2]` |

```javascript
const numbers = [5, 2, 8, 1, 9];
const max = Math.max(...numbers);  // 9
const min = Math.min(...numbers);  // 1
```

### 배열 메서드와 함께 사용

| 메서드 | 설명 | 예시 |
|---|---|---|
| `Array.from()` | Iterable을 배열로 변환 | `Array.from(set)` |
| `array.slice()` | Spread와 함께 배열 복사 | `[...array]` |
| `array.concat()` | Spread로 배열 합치기 | `[...arr1, ...arr2]` |
| `array.join()` | 배열 요소를 문자열로 | `[...str].join('-')` |

```javascript
// Set을 배열로 변환
const set = new Set([1, 2, 3]);
const arr = [...set];  // [1, 2, 3]

// 문자열을 배열로 변환 후 역순
const str = "hello";
const reversed = [...str].reverse().join('');  // "olleh"
```

### 함수 인자 전달

```javascript
function printNumbers(a, b, c) {
    console.log(`a=${a}, b=${b}, c=${c}`);
}

const arr = [10, 20, 30];
printNumbers(...arr);  // printNumbers(10, 20, 30)
// 출력: a=10, b=20, c=30
```

---

## Generator (생성기)

### Generator Function 정의

- Generator function은 `function*` 키워드로 정의되는 특수한 함수다.
- 일반 함수와 다르게 호출 시 함수 본체를 **실행하지 않음**
- 대신 Generator 객체를 **반환**
- Generator 객체는 Iterator다

### Generator Function 호출의 특징

```javascript
function* gen() {
    console.log("1. 실행됨");
    yield 1;
    
    console.log("2. 실행됨");
    yield 2;
    
    console.log("3. 실행됨");
    yield 3;
}

const g = gen();  // 함수 본체가 실행되지 않음 (아무 출력 없음)
```

> Generator Function을 호출해도 함수의 본체(Body)는 실행되지 않는다.
> 대신 Generator 객체만 반환된다.
> 함수의 코드는 `next()` 메서드를 호출할 때마다 **일부씩 실행**된다.

### 기본 예시

```javascript
function* gen() {
    yield 1;
    yield 2;
    yield 3;
}

const g = gen();  // 함수 본체는 아직 실행 안 됨
g.next();  // { value: 1, done: false } - 첫 yield까지 실행
g.next();  // { value: 2, done: false } - 두 번째 yield까지 실행
g.next();  // { value: 3, done: false } - 세 번째 yield까지 실행
g.next();  // { value: undefined, done: true } - 완료
```

---

## yield 키워드

### 정의

- `yield`는 Generator 함수에서만 사용할 수 있으며, 함수를 **일시 중지** 하고 값을 반환한다.
- `return`과 다르게, Generator는 `yield` 이후에도 상태를 유지했다가 다시 `next()`가 호출되면 그 지점부터 계속 실행된다.

### 실행 흐름 예시

```javascript
function* example() {
    console.log("1. 시작");
    yield 10;                  // 여기서 일시 중지
    
    console.log("2. 재개");    // 다음 next()에서 실행
    yield 20;                  // 또 일시 중지
    
    console.log("3. 마지막");  // 그 다음 next()에서 실행
    return 30;                 // 완료
}

const g = example();
g.next();  // "1. 시작" 출력 → { value: 10, done: false }
g.next();  // "2. 재개" 출력 → { value: 20, done: false }
g.next();  // "3. 마지막" 출력 → { value: 30, done: true }
```

### for/of와 함께 사용

Generator를 Iterable처럼 사용할 수 있다.

```javascript
function* range(n) {
    for (let i = 0; i < n; i++) {
        yield i;
    }
}

// for/of에서 자동으로 next() 호출
for (const num of range(3)) {
    console.log(num);  // 0, 1, 2
}

// spread operator에서도 사용
const arr = [...range(5)];  // [0, 1, 2, 3, 4]
```

### 무한 Generator

Generator는 언제든 끝낼 수 있으므로, 무한 루프도 안전하다.

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
// 필요한 만큼만 호출 가능
```

---

## yield* (위임 연산자)

### 정의

- `yield*`는 다른 Generator나 Iterable을 순회하면서 그 값들을 자신의 값으로 반환한다.

### 기본 예시

```javascript
function* inner() {
    yield 1;
    yield 2;
}

function* outer() {
    yield* inner();  // inner의 모든 값을 yield
    yield 3;
}

for (const val of outer()) {
    console.log(val);  // 1, 2, 3
}
```

### 재귀적 사용 (배열 평탄화)

```javascript
function* flatten(arr) {
    for (const item of arr) {
        if (Array.isArray(item)) {
            yield* flatten(item);  // 중첩된 배열 재귀 처리
        } else {
            yield item;
        }
    }
}

const nested = [1, [2, [3, 4]], 5];
const flat = [...flatten(nested)];  // [1, 2, 3, 4, 5]
```

---

## 정리

| 개념 | 설명 |
|---|---|
| **Iterable** (반복 가능한 객체) | `[Symbol.iterator]()` 메서드를 가진 객체 |
| **Iterator** (반복자) | `next()` 메서드를 가진 객체 |
| **Spread Operator** (전개 연산자) | Iterable을 펼쳐서 개별 요소로 변환 |
| **Generator** (생성기) | `function*`로 정의, Iterator 객체 반환 |
| **yield** | Generator 함수에서 값을 반환하고 일시 중지 |
| **yield*** (위임 연산자) | 다른 Iterable의 값을 위임받아 yield |
