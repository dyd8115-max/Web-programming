### JavaScript

---

# 자바스크립트 실행 환경

---

## 런타임 (Runtime)

- JS 코드가 실행되는 환경 전체
- 실행 환경에 따라 사용할 수 있는 API가 달라짐

| 런타임 | 설명 | 사용 API |
|---|---|---|
| 브라우저 | 화면을 그리고 사용자와 상호작용 | Web API (DOM, fetch, setTimeout 등) |
| Node.js | 서버에서 JS 실행 | Node.js API (파일시스템, 네트워크 등) |

> 웹 어셈블리 (WebAssembly): C, C++, Rust 등 다른 언어로 작성한 코드를 브라우저에서 실행 가능하게 해주는 기술 → JS 대신 다른 언어로 프론트 개발 가능 (성능이 중요한 게임, 영상편집 등에 활용)

---

## JS 엔진

- JS 코드를 읽고 실행하는 핵심 프로그램
- 크롬 / Node.js → **V8** 엔진 사용

```
[ JS 엔진 ]
┌──────────────────────────┐
│  컴파일러                 │ ← JS 코드를 머신코드로 변환
│  메모리 힙 (Memory Heap)  │ ← 객체/변수 저장 공간
│  콜스택 (Call Stack)      │ ← 함수 실행 순서 관리
└──────────────────────────┘
```

### 컴파일러 & 머신코드

- JS는 인터프리터 언어지만 V8은 **JIT(Just-In-Time) 컴파일** 방식 사용
- JS 코드 → 머신코드(CPU가 직접 읽는 이진코드)로 변환 → 실행 속도 향상

> 인터프리터 (Interpreter): 코드를 한 줄씩 읽으면서 바로 실행하는 방식, 변환 없이 즉시 실행되지만 느림
>
> V8: 구글이 만든 JS 엔진, JVM과 유사하게 런타임 컴파일 방식 사용. 처음엔 인터프리터처럼 실행하다가 자주 반복되는 코드를 감지하면 머신코드로 컴파일해서 캐싱 → 반복 실행할수록 빨라짐
>
> JIT (Just-In-Time): 실행하면서 자주 쓰는 코드만 골라 머신코드로 변환하는 방식, 인터프리터(즉시 실행)와 컴파일러(빠른 실행) 둘의 장점만 취함
>
> ```
> 이그니션 (Ignition): V8의 인터프리터, 모든 코드를 바이트코드로 변환해 즉시 실행
> 터보팬 (TurboFan): V8의 JIT 컴파일러, Ignition이 실행 중 자주 반복되는 핫한 코드 감지 시에만 작동 → 머신코드로 최적화 후 캐싱
> ```

변환 흐름:
```
JS 소스코드
    ↓ (파싱)
AST (추상 구문 트리)
    ↓ (이그니션, 인터프리터)
바이트코드
    ↓ (자주 실행되는 부분 감지)
머신코드 (터보팬, JIT 컴파일러)
```

> 파싱 (Parsing): 소스코드를 읽어서 문법 검사 및 구조 분석 후 AST로 변환하는 과정
>
> AST (Abstract Syntax Tree): 파싱된 코드를 계층적 트리 구조로 표현한 것
>
> 인터프리팅 (Interpreting): 이그니션이 AST를 읽어서 한 줄씩 바이트코드로 변환하고 즉시 실행하는 과정
>
> 바이트코드 (Bytecode): 중간 단계 코드, 가상머신이 읽음 / 머신코드 (Machine Code): CPU가 직접 읽는 이진코드, 가장 빠름

**바이트코드를 거치는 이유**

| 이유 | 설명 |
|---|---|
| 빠른 시작 | 바이트코드 생성이 빠름 → 즉시 실행 가능 (머신코드는 생성 시간 오래 걸림) |
| 동적 타입 대응 | 컴파일 시점에 타입을 모르므로 미리 최적화 불가 → 실행하면서 실제 타입 패턴 관찰 → 관찰된 패턴으로 효율적인 머신코드 생성 |

### JIT vs AOT 컴파일

| 구분 | JIT | AOT |
|---|---|---|
| 컴파일 시점 | 실행 중 (동적) | 실행 전 (정적) |
| 초기 실행 속도 | 빠름 ✅ | 느림 ❌ |
| 최종 실행 속도 | 빠름 ✅ | 매우 빠름 ✅✅ |
| 사용 예 | JavaScript, Java | C, C++, Rust |

**JavaScript는 JIT 사용 — 왜?**

JavaScript는 동적 타입 언어라 타입이 런타임에 결정됨 → AOT로 미리 컴파일 불가능 → JIT로 실행 중에 타입 감지해서 컴파일해야 함

```javascript
let x = 10;        // 숫자
x = "hello";       // 문자열로 변경 가능
// AOT는 이런 변수 타입 미리 알 수 없음
```

**JavaScript는 인터프리터와 컴파일러의 중간 위치**

| 구분 | 기존 인터프리터 | JavaScript (JIT) | 컴파일러 |
|---|---|---|---|
| 실행 방식 | 한 줄씩 읽고 즉시 실행 | 인터프리터 + 컴파일러 결합 | 전부 미리 컴파일 후 실행 |
| 초기 시작 | 빠름 ✅ | 빠름 ✅ | 느림 ❌ |
| 최종 속도 | 느림 ❌ | 빠름 ✅ | 매우 빠름 ✅✅ |
| 최적화 | 없음 (매번 해석) | 있음 (자주 쓰는 부분만 컴파일) | 완전함 (모든 코드 최적화) |
| 예시 | Python, Ruby | JavaScript (V8) | C, C++ |

**실행 흐름 비교**

```
기존 인터프리터 (Python)
코드 → 한 줄씩 읽음 → 해석 → 실행
      (매번 반복, 최적화 없음)
      ↓ 반복 실행: 계속 느림

JavaScript (JIT, V8)
코드 → 이그니션(인터프리터) 한 줄씩 해석 → 바이트코드 → 즉시 실행
      ↓ 자주 실행되는 부분 감지
      터보팬(컴파일러) → 머신코드로 컴파일 → 캐싱
      ↓ 반복 실행: 점점 빨라짐 ✅

컴파일러 (C++)
코드 → 전체 컴파일 → 머신코드 → 배포 → 사용자가 빠르게 실행
      (시간 걸리지만 한 번만, 이후 계속 빠름)
```

**결론: JavaScript = 인터프리터의 빠른 시작 + JIT 컴파일러의 점진적 최적화**

반복되는 코드가 없으면 JIT가 컴파일할 게 없으니까 그냥 이그니션(인터프리터)만 작동해서 기존 인터프리터처럼 동작. 반복되는 부분을 감지하면 터보팬이 머신코드로 컴파일 → 반복할수록 빨라짐.

### 메모리 힙 (Memory Heap)

- 객체, 배열, 함수 등 참조 타입 데이터가 저장되는 공간
- 콜스택은 순서가 있지만 힙은 구조 없이 자유롭게 저장

```
메모리 힙                콜스택
┌──────────────────┐    ┌──────────┐
│ { name: "이용균" }│    │ b()      │
│ [1, 2, 3]        │◀───┤ a()      │
│ function() {}    │    │ main()   │
└──────────────────┘    └──────────┘
```

### 콜스택 (Call Stack)

- 함수 호출을 관리하는 자료구조 (LIFO - 나중에 들어온 게 먼저 나감)
- 함수 호출 시 실행 컨텍스트를 스택에 **push**, 함수 종료 시 **pop**
- 메모리 힙의 참조를 저장하고 동기 코드 실행 순서를 관리

```javascript
function a() { b(); }
function b() { console.log("b 실행"); }
a();

// [main] → [a] → [b]  push
// [main] → [a]         pop (b 종료)
// [main]               pop (a 종료)
```

> 실행 컨텍스트 (Execution Context): 함수가 실행될 때 필요한 정보(변수, this, 스코프 등)를 담은 객체, 함수 호출마다 생성되어 콜스택에 push, 종료되면 pop

---

## 소스코드 실행 순서

- 브라우저가 HTML 파싱 → CSS 파싱 → JS 실행 순서로 동작
- `<script>` 태그를 만나면 파싱 멈추고 JS 먼저 실행 (블로킹)
- 이를 방지하기 위해 `defer`, `async` 속성 사용

```html
<script defer src="app.js"></script>  <!-- HTML 파싱 후 실행 -->
<script async src="app.js"></script>  <!-- 다운로드 완료되면 바로 실행 -->
```

---

## 동기 vs 비동기 실행

### 콜백 (Callback)

- 함수를 인자로 넘겨서 특정 시점에 실행되게 하는 것
- 비동기 처리의 기본 패턴

```javascript
setTimeout(() => console.log("1초 후 실행"), 1000); // 비동기 콜백
setTimeout(function() { console.log("1초 후 실행"); }, 1000); // function 키워드로도 사용 가능
arr.map(item => item * 2);                          // 동기 콜백
```

> 개발 과정 중 개발자가 직접 선언

**동기 vs 비동기**

동기: 코드가 순서대로 한 줄씩 실행 (이전 작업 완료 대기)
비동기: 작업을 위임하고 기다리지 않음 (완료 후 콜백으로 처리)

**동기 콜백 vs 비동기 콜백 실행 흐름**

```javascript
console.log("1");

setTimeout(() => {
    console.log("2");  // 비동기 콜백
}, 1000);

console.log("3");

// 결과:
// 1
// 3
// (1초 후)
// 2
```

**실행 단계별:**

| 단계 | 코드 | 실행 위치 | 설명 |
|---|---|---|---|
| 1 | `console.log("1")` | 콜스택 | 동기 코드 즉시 실행 |
| 2 | `setTimeout(..., 1000)` | 콜스택 → Web API | 콜백을 Web API에 위임 (1초 타이머 시작) |
| 3 | `console.log("3")` | 콜스택 | 동기 코드 즉시 실행 (콜스택 비움) |
| 4 | (1초 경과) | Web API | 타이머 완료 → 콜백을 **태스크 큐**에 넣음 |
| 5 | 콜백 실행 | 이벤트 루프 → 콜스택 | 콜스택 비어있으니 큐에서 꺼내 실행 |

**콜백이 Web API로 가는 과정:**

```
동기 콜백
arr.forEach(item => console.log(item))
    ↓
바로 콜스택에 push
    ↓
즉시 실행

비동기 콜백
setTimeout(() => console.log("2"), 1000)
    ↓
콜백을 Web API에 전달 (콜스택은 비움)
    ↓
Web API가 1초 대기
    ↓
1초 후 콜백을 태스크 큐에 넣음
    ↓
이벤트 루프가 콜스택 감시
    ↓
콜스택 비어있음 → 큐에서 꺼내 콜스택으로 올림
    ↓
콜백 실행
```

### Web API (브라우저 제공 API)

- 브라우저가 JS 엔진 외부에서 제공하는 기능들
- `setTimeout`, `fetch`, `DOM 조작`, `localStorage` 등
- Node.js 환경에서는 브라우저 Web API 대신 Node.js API 사용

### 큐 (Queue)

- 비동기 작업 완료 시 콜백 함수가 대기하는 곳 (대기실 역할)
- FIFO 자료구조 (먼저 들어온 게 먼저 나감)
- 동기 코드는 큐를 거치지 않고 콜스택에 바로 올라감

| 구분 | 특징 | 예시 | 우선순위 |
|---|---|---|---|
| 태스크 큐 | Web API(타이머, 이벤트 등)의 콜백이 대기 | `setTimeout`, `setInterval`, DOM 이벤트, `fetch` | 낮음 (나중에 실행) |
| 마이크로태스크 큐 | Promise 기반의 고우선순위 콜백이 대기 | Promise `.then()`, `async/await`, `MutationObserver` | 높음 (먼저 실행) |

> `setInterval`: 일정 시간 간격으로 반복 실행 (setTimeout은 1회만)
>
> `fetch`: 서버에 HTTP 요청하는 Web API (Promise 반환)
>
> Promise/async/await: 비동기 작업의 결과를 더 깔끔하게 처리하는 방식 (콜백 대체)
>
> `MutationObserver`: DOM 변경을 감지하는 Web API

### 이벤트 루프 (Event Loop)

- 콜스택이 비어있는지 계속 감시하다가, 비어있으면 큐에서 콜백을 꺼내 콜스택에 올려주는 역할
- JS가 싱글스레드임에도 비동기 동작이 가능한 이유

```
[ JS 런타임 전체 구조 ]

        JS 엔진
┌─────────────────┐
│  콜스택          │◀──────────────────────────┐
│  메모리 힙       │                            │
└─────────────────┘                     이벤트 루프
                                    (콜스택 감시 → 비면 큐에서 꺼냠)
      Web API (비동기 작업 처리)               │
┌─────────────────┐                            │
│ setTimeout      │──▶ 태스크 큐 ──────────────┘
│ fetch           │──▶ 마이크로태스크 큐 ───────┘
│ DOM 이벤트      │──▶ 태스크 큐
└─────────────────┘
         완료된 콜백 함수가 큐(대기실)에 들어감
```

---

## 싱글스레드와 웹 워커

- JS는 기본적으로 싱글스레드 → 한 번에 하나씩만 실행 가능
- 논블로킹 (Non-blocking): 작업 완료를 기다리지 않고 바로 다음 코드 실행

> JS가 싱글스레드인 이유: 브라우저 보안 (DOM 동시 수정 방지) + 개발 복잡도 감소. 현재는 웹 워커로 멀티스레드 보완.

### 싱글스레드의 문제점

> 콜스택 블로킹: 초반에 메인 스레드의 무거운 작업이 발생하면 완료될 때까지 다른 모든 작업(이벤트, 렌더링, 콜백)이 대기

메인 스레드에서 무거운 연산 → 다른 작업 블로킹 (UI 멈춤)

```javascript
function heavyCalculation() {
    let sum = 0;
    for (let i = 0; i < 1000000000; i++) {
        sum += i;
    }
    return sum;
}

heavyCalculation();  // 이 동안 UI 반응 없음 (버튼 클릭 안 됨, 화면 스크롤 안 됨)
```

### 웹 워커 (Web Worker)

- 별도 스레드에서 JS를 실행할 수 있게 해주는 Web API
- 무거운 연산을 메인 스레드 방해 없이 백그라운드에서 처리 가능 (논블로킹)

**웹 워커로 해결:**

```javascript
// 무거운 연산을 별도 스레드에서 처리 → UI는 계속 반응 (논블로킹)
const worker = new Worker("heavy.js");
worker.postMessage("계산 시작");
// UI는 여전히 반응 가능 ✅
```

**사용 예시:**

```javascript
// main.js (메인 스레드)
const worker = new Worker("worker.js");

// 워커에 데이터 전송
worker.postMessage({
    type: "calculate",
    data: [1, 2, 3, 4, 5]
});

// 워커로부터 결과 받음
worker.onmessage = (e) => {
    console.log("계산 결과:", e.data);  // 완료된 결과 받음
};
```

```javascript
// worker.js (별도 스레드)
onmessage = (e) => {
    if (e.data.type === "calculate") {
        // 무거운 연산 수행
        let result = 0;
        for (let i = 0; i < 1000000000; i++) {
            result += i;
        }
        
        // 메인 스레드에 결과 전송
        postMessage(result);
    }
};
```

**실제 사용 사례:**

| 사용처 | 예시 |
|---|---|
| 이미지/영상 처리 | 이미지 필터 적용, 영상 인코딩 |
| 데이터 분석 | 대용량 데이터 정렬, 계산 |
| 암호화 | 복잡한 암호화 알고리즘 |
| 게임 | 물리 연산, AI 계산 |
| 맵 렌더링 | 타일 생성, 경로 탐색 |

**웹 워커의 제한사항:**

```javascript
// DOM에 접근 불가 ❌
document.getElementById("btn");  // 에러!

// window 객체 접근 불가 ❌
console.log(window);  // 에러!

// 부모 페이지의 변수 접근 불가 ❌
// 오직 메시지 전달(postMessage)로만 통신

// 같은 출처 규칙 적용 (CORS)
new Worker("http://another-domain.com/worker.js");  // 에러!
```

---

## 브라우저 내장 기능

### DOM (Document Object Model)

- 브라우저가 HTML을 읽어서 만든 트리 구조의 객체 모델
- JS가 HTML 요소에 접근/수정할 수 있게 해주는 인터페이스

```javascript
document.getElementById("btn");   // 요소 접근
document.querySelector(".class"); // CSS 선택자로 접근
document.createElement("div");    // 요소 생성
element.innerHTML = "안녕";       // 내용 수정
```

### 빌트인 라이브러리 (Built-in Library)

- JS 엔진에 기본으로 내장된 기능들, 따로 설치 없이 바로 사용 가능
- `Math`, `Date`, `JSON`, `Promise`, `Array`, `Object` 등

```javascript
Math.random();          // 랜덤 숫자
Date.now();             // 현재 시간
JSON.stringify(obj);    // 객체 → JSON 문자열
JSON.parse(str);        // JSON 문자열 → 객체
```

### 스토리지 (Storage)

- 브라우저에 데이터를 저장하는 Web API
- 서버 없이 클라이언트 측에 데이터 보관 가능

| 종류 | 유지 기간 | 용량 |
|---|---|---|
| `localStorage` | 브라우저 닫아도 유지 | 약 5MB |
| `sessionStorage` | 탭 닫으면 삭제 | 약 5MB |
| `cookie` | 만료일 설정 가능 | 약 4KB |

```javascript
localStorage.setItem("token", "abc123"); // 저장
localStorage.getItem("token");           // 읽기
localStorage.removeItem("token");        // 삭제
```

### 리소스 (Resource)

- 브라우저가 페이지를 그리기 위해 서버에서 받아오는 것들
- HTML, CSS, JS, 이미지, 폰트 등
- 개발자 도구 Network 탭에서 확인 가능

### 인터랙션 (Interaction)

- 사용자가 화면과 상호작용하는 것 (클릭, 스크롤, 키보드 입력 등)
- JS의 이벤트 리스너로 처리, 완료된 콜백은 태스크 큐에 들어감

```javascript
element.addEventListener("click", () => { ... });
element.addEventListener("keydown", (e) => { ... });
element.addEventListener("scroll", () => { ... });
```

---

## 라이브러리 & 컴포넌트

### 라이브러리 (Library)

- 자주 쓰는 기능을 미리 만들어 놓은 코드 모음
- 필요할 때 가져다 씀 (흐름 제어는 개발자가 담당)
- ex) React, Lodash, Axios

### 컴포넌트 (Component)

- UI를 독립적인 단위로 쪼갠 것
- React 등 프론트 프레임워크에서 화면을 구성하는 기본 단위
- 재사용 가능하고 각자 독립적으로 동작

---

## 전체 흐름 정리

```
[ 브라우저에서 JS가 실행되는 전체 흐름 ]

1. 브라우저가 서버에서 리소스(HTML, CSS, JS) 받아옴
        ↓
2. HTML 파싱 → DOM 트리 생성
   CSS 파싱 → 스타일 적용
   JS 실행 (script 태그 만나면 파싱 멈추고 실행)
        ↓
3. JS 엔진(V8)이 코드를 JIT 컴파일 → 머신코드로 변환
        ↓
4. 동기 코드 → 콜스택에 push하여 순서대로 실행
   비동기 코드(setTimeout, fetch 등) → Web API에 위임
        ↓
5. Web API가 비동기 작업 처리
   완료되면 콜백을 큐에 넣음
   - Promise / async → 마이크로태스크 큐
   - setTimeout / DOM 이벤트 → 태스크 큐
        ↓
6. 이벤트 루프가 콜스택 감시
   콜스택 비면 → 마이크로태스크 큐 먼저, 그 다음 태스크 큐에서 꺼내 실행
        ↓
7. 사용자 인터랙션(클릭, 스크롤 등) 발생
   → 이벤트 리스너 콜백이 태스크 큐에 들어가 동일하게 처리
        ↓
8. 무거운 연산 필요 시 웹 워커로 분리
   → 메인 스레드 블로킹 없이 백그라운드 처리
```
