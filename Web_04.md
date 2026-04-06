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
│  콜스택 (Call Stack)      │ ← 함수 실행 순서 관리
│  메모리 힙 (Memory Heap)  │ ← 객체/변수 저장 공간
└──────────────────────────┘
```

### 컴파일러 & 머신코드

- JS는 인터프리터 언어지만 V8은 **JIT(Just-In-Time) 컴파일** 방식 사용
- JS 코드 → 머신코드(CPU가 직접 읽는 이진코드)로 변환 → 실행 속도 향상

> 인터프리터 (Interpreter): 코드를 한 줄씩 읽으면서 바로 실행하는 방식, 변환 없이 즉시 실행되지만 느림
>
> V8: 구글이 만든 JS 엔진, 처음엔 인터프리터처럼 실행하다가 자주 반복되는 코드를 감지하면 머신코드로 컴파일해서 캐싱 → 반복 실행할수록 빨라짐
>
> JIT (Just-In-Time): 실행하면서 자주 쓰는 코드만 골라 머신코드로 변환하는 방식, 인터프리터(즉시 실행)와 컴파일러(빠른 실행) 둘의 장점만 취함

### 콜스택 (Call Stack)

- 함수 호출을 관리하는 자료구조 (LIFO - 나중에 들어온 게 먼저 나감)
- 함수 호출 시 실행 컨텍스트를 스택에 **push**, 함수 종료 시 **pop**

```javascript
function a() { b(); }
function b() { console.log("b 실행"); }
a();

// [main] → [a] → [b]  push
// [main] → [a]         pop (b 종료)
// [main]               pop (a 종료)
```

> 실행 컨텍스트 (Execution Context): 함수가 실행될 때 필요한 정보(변수, this, 스코프 등)를 담은 객체, 함수 호출마다 생성되어 콜스택에 push, 종료되면 pop

### 메모리 힙 (Memory Heap)

- 객체, 배열, 함수 등 참조 타입 데이터가 저장되는 공간
- 콜스택은 순서가 있지만 힙은 구조 없이 자유롭게 저장

```
콜스택                  메모리 힙
┌──────────┐           ┌──────────────────┐
│ b()      │ ────────▶ │ { name: "이용균" }│
│ a()      │                │ [1, 2, 3]         │
│ main()   │                └──────────────────┘
└──────────┘
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

## JS 실행 관련

### 소스코드 실행 순서

- 브라우저가 HTML 파싱 → CSS 파싱 → JS 실행 순서로 동작
- `<script>` 태그를 만나면 파싱 멈추고 JS 먼저 실행 (블로킹)
- 이를 방지하기 위해 `defer`, `async` 속성 사용

```html
<script defer src="app.js"></script>  <!-- HTML 파싱 후 실행 -->
<script async src="app.js"></script>  <!-- 다운로드 완료되면 바로 실행 -->
```

### 콜백 (Callback)

- 함수를 인자로 넘겨서 특정 시점에 실행되게 하는 것
- 비동기 처리의 기본 패턴, 완료된 콜백은 큐에 들어가 이벤트 루프가 콜스택으로 올려줌

```javascript
setTimeout(() => console.log("1초 후 실행"), 1000); // 비동기 콜백
arr.map(item => item * 2);                          // 동기 콜백
```

---

## 비동기 동작

- JS는 싱글스레드 → 한 번에 하나만 실행 가능
- 오래 걸리는 작업(네트워크, 타이머 등)은 Web API에 위임하고 다음 코드 먼저 실행

### Web API (브라우저 제공 API)

- 브라우저가 JS 엔진 외부에서 제공하는 기능들
- `setTimeout`, `fetch`, `DOM 조작`, `localStorage` 등
- Node.js 환경에서는 브라우저 Web API 대신 Node.js API 사용

### 큐 (Queue)

- 비동기 작업 완료 시 콜백 함수가 대기하는 곳 (대기실 역할)
- FIFO 자료구조 (먼저 들어온 게 먼저 나감)
- 동기 코드는 큐를 거치지 않고 콜스택에 바로 올라감

| 구분 | 예시 | 우선순위 |
|---|---|---|
| 마이크로태스크 큐 | Promise `.then()`, `async/await` | 높음 (먼저 실행) |
| 태스크 큐 | `setTimeout`, `setInterval`, DOM 이벤트 | 낮음 (나중에 실행) |

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
                                    (콜스택 감시 → 비면 큐에서 꺼냄)
      Web API (비동기 작업 처리)               │
┌─────────────────┐                            │
│ setTimeout      │──▶ 태스크 큐 ──────────────┘
│ fetch           │──▶ 마이크로태스크 큐 ───────┘
│ DOM 이벤트      │──▶ 태스크 큐
└─────────────────┘
         완료된 콜백 함수가 큐(대기실)에 들어감
```

> 공통 원리 (소켓 통신과 비교): 소켓 1이 블로킹 없이 새 클라이언트를 받고 소켓 3이 실제 통신을 담당하는 것처럼, 콜스택은 블로킹 없이 계속 실행되고 비동기 작업은 Web API가 따로 처리함

### 멀티스레드 & 웹 워커 (Web Worker)

- JS는 기본적으로 싱글스레드
- 웹 워커: 별도 스레드에서 JS를 실행할 수 있게 해주는 Web API
- 무거운 연산을 메인 스레드 방해 없이 백그라운드에서 처리 가능

```javascript
// main.js
const worker = new Worker("worker.js");
worker.postMessage("시작해");        // 워커에 메시지 전송
worker.onmessage = (e) => {
    console.log(e.data);            // 워커 결과 받음
};

// worker.js
onmessage = (e) => {
    // 무거운 연산 처리
    postMessage("완료");            // 메인에 결과 전송
};
```

> 웹 워커는 DOM에 접근 불가 → 순수 연산만 가능, UI 조작은 메인 스레드에서만 가능

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
