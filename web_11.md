# Express Framework 강의 노트

---

## 1. Express란?

> Node.js를 위한 유연하고 간결한 웹 프레임워크  
> HTTP 요청 처리를 단순화하고 Routing / Middleware / View Engine을 제공

### 기본 동작 흐름

```
클라이언트 (브라우저)
  │  ① index.html 수신
  │  ② HTTP 요청 (Request)
  ▼
웹 서버 (Express)
  │  ③ HTTP 응답 (Response)
  ▼
클라이언트
```

### Express의 3가지 핵심 역할

| 역할 | 설명 |
|------|------|
| **Routing** | HTTP verb + URI 기반으로 요청을 핸들러에 연결 |
| **Middleware** | 요청-응답 사이 공통 로직 분리 (로그, 인증, 파싱 등) |
| **View Engine** | 서버에서 동적 HTML 생성 후 응답 (SSR) |

---

## 2. 초기 세팅

```bash
mkdir <디렉토리> && cd <디렉토리>
npm init -y
npm install express
npm install --save-dev nodemon
touch server.js
npm run dev
```

```json
"scripts": { "dev": "nodemon server.js" }
```

---

## 3. 배경 지식

### HTTP 프로토콜 구조

> 네트워크 레이어에서 패킷은 IP header + TCP header + HTTP header + Data 로 구성  
> Express에서는 HTTP header를 직접 다루지 않아도 됨 — req 객체로 추상화되어 있음

```
요청 (Request)
  ├── Request Line : GET /olivia HTTP/1.1   ← verb | URI | HTTP 버전
  ├── Headers      : HOST, Accept, Content-Type, User-Agent 등
  └── Body         : POST/PUT 시 데이터 포함 (GET은 비어 있음)

응답 (Response)
  ├── Status Line  : HTTP/1.1 200 OK
  ├── Headers      : Date, Content-Type, Content-Length 등
  └── Body         : HTML, JSON, CSS 등 응답 데이터
```

URL 구조: `http://www.example.com/index.html`
- `http` — 프로토콜
- `www.example.com` — 호스트명
- `/index.html` — 경로(pathname)

### HTTP 메소드

| 메소드 | 의미 | 용도 |
|--------|------|------|
| `GET` | 데이터 조회 | 페이지 요청, 데이터 불러오기 |
| `POST` | 데이터 생성 | 로그인, 폼 제출 |
| `PUT` | 데이터 수정 (전체) | 리소스 전체 업데이트 |
| `PATCH` | 데이터 수정 (일부) | 리소스 일부 업데이트 |
| `DELETE` | 데이터 삭제 | 리소스 삭제 |

### HTTP 상태 코드

| 범위 | 분류 | 대표 예시 |
|------|------|-----------|
| 1xx | Informational | 처리 중 |
| 2xx | Success | 200 OK, 201 Created |
| 3xx | Redirection | 301 영구, 302 임시, 304 캐시 사용 |
| 4xx | Client Error | 400 잘못된 요청, 401 인증, 403 권한, 404 없음 |
| 5xx | Server Error | 500 서버 내부 오류 |

- **4xx**: 클라이언트가 서버와 약속되지 않은 잘못된 요청을 보낼 때 발생
- **304 Not Modified**: 자원이 변경되지 않아 클라이언트 캐시 사용을 권장하는 코드

```js
res.status(200).send("OK");
res.status(404).send("Not found");
res.status(201).json({ message: "생성 완료" });
```

### JSON

> 서버-클라이언트 간 데이터 교환에 사용하는 경량 데이터 형식

```
CLIENT  →  HTTP (GET/POST/PUT/DELETE) + URL  →  SERVER
CLIENT  ←──────────────── JSON 응답 ←──────────
```

- SPA에서 서버는 HTML 대신 **JSON만 응답** → 프론트가 화면 구성
- `res.json({ key: "value" })` 로 응답
- 클라이언트에서 수신: `.then(res => res.json())`

---

## 4. 개발 환경 & CORS

```
내 PC
  ├── Live Server → localhost:5500  ← index.html 서빙
  └── Express 서버 → localhost:3000  ← API 처리
```

- 포트가 달라 다른 출처(Origin)로 인식 → **CORS 오류 발생**
- CORS: 브라우저가 다른 출처의 응답을 기본적으로 차단하는 보안 정책

```bash
npm install cors
```

```js
app.use(cors()); // 모든 출처 허용
```

---

## 5. MPA vs SPA

| | MPA (Multi-Page App) | SPA (Single-Page App) |
|---|---|---|
| 서버 응답 | HTML 전체 반환 | JSON 데이터만 반환 |
| 화면 전환 | 페이지 새로고침 (DOM 전체 재생성) | 클라이언트에서 부분 렌더링 |
| 사용자 경험 | 전환마다 깜빡임 → 경험 저하 | 자연스러운 전환 |
| 렌더링 위치 | 서버 (SSR) | 클라이언트 (CSR) |

> MPA는 DOM이 통째로 새로 만들어지기 때문에 UX가 저하됨 → 요즘은 **SPA 주로 사용**

---

## 6. Routing

> 애플리케이션 엔드포인트(URI)와 동작을 정의하여  
> 클라이언트 Request에 적절한 Response를 만드는 과정

- 사용자 요청을 수신할 때 **HTTP verb + URI** 를 기반으로 request handler에 전달해 적절한 동작 수행

```js
app.get("/", (req, res) => { res.send("홈페이지"); });
app.post("/submit", (req, res) => { ... });
app.put("/update", (req, res) => { ... });
app.delete("/delete", (req, res) => { ... });
```

### req 객체 주요 속성

```js
req.headers  // HTTP 헤더
req.body     // 요청 바디 (POST 등, body-parser 필요)
req.query    // 쿼리스트링 (?key=value)
req.params   // Path Variable (:id)
```

### Path Variable

```js
app.get("/users/:id", (req, res) => {
  console.log(req.params.id); // URL의 :id 값 접근
});
// http://localhost:3000/users/1  →  req.params.id = "1"
```

### app.param — Path Variable 전처리

```js
// :id 가 포함된 라우트가 실행되기 전에 먼저 동작
app.param("id", (req, res, next, id) => {
  req.xxx = students[id]; // id로 데이터 미리 찾아서 req에 붙여놓기
  next();
});
```

### Router — 라우트 파일 분리

> Router = 미들웨어와 라우팅만 수행하는 독립된 **"미니 앱"**  
> 라우터를 활용하면 앱을 구조화할 수 있음

```
App
  ├── /
  ├── /foo    →  Foo Router ( /, /bar )
  ├── /users
  ├── /api
  └── /details
```

```js
// routes/api_router.js
const router = require("express").Router();
router.get("/users", (req, res) => { ... });
router.post("/user", (req, res) => { ... });
module.exports = router;

// server.js
const apiRouter = require("./routes/api_router");
app.use("/api", apiRouter); // /api/users, /api/user 로 접근 가능
```

---

## 7. Middleware

> Request와 Response 사이에 동작하는 기능(함수)들  
> req, res 두 객체가 미들웨어 스택을 순차적으로 통과

```
HTTP Request
  → [Logging]       req | res  → next()
  → [User Auth]     req | res  → next()
  → [JSON Parsing]  req | res  → next()
  → [Static Files]  req | res  → next()
  → [App Routing]   req | res
HTTP Response
```

```js
app.use((req, res, next) => {
  console.log("요청 URL: " + req.url);
  next(); // 다음 미들웨어로 넘김 — 필수!
});
```

- **순서 중요** — 위에서 아래로 순차 실행
- `next()` 없으면 요청이 그 미들웨어에서 멈춤
- 특정 라우트에만 적용: `app.get("/home", logger, handler)`

### 404 핸들러 — 반드시 마지막에 배치

```js
app.use((req, res) => {
  res.status(404).send("File not found!");
});
```

---

## 8. 유용한 미들웨어 모듈

### express.static — 정적 파일 서빙

```js
app.use(express.static("public"));
// http://localhost:3000/style.css → public/style.css 서빙
```

### body-parser — 요청 바디 파싱

```bash
npm install body-parser
```

```js
app.use(bodyParser.json());                         // JSON 파싱
app.use(bodyParser.urlencoded({ extended: true })); // Form 데이터 파싱
// → POST 요청 바디를 req.body 로 접근 가능
```

### cookie-parser — 쿠키 관리

```bash
npm install cookie-parser
```

```js
app.use(cookieParser());
req.cookies.username;                                              // 읽기
res.cookie("username", "값", { maxAge: 900000, httpOnly: true }); // 설정
res.clearCookie("username");                                       // 삭제
```

### helmet — 보안 강화

```bash
npm install helmet
```

```js
app.use(helmet());
// 적용 전 헤더 7개 → 적용 후 18개
// Content-Security-Policy, X-XSS-Protection, Strict-Transport-Security 등 자동 추가
```

---

## 9. View Engine — EJS

> HTML에 동적으로 JS 코드 및 데이터를 삽입할 수 있는 템플릿 엔진  
> **SSR**: 서버에서 Template + Data를 결합해 완성된 HTML을 만들어 응답

```
Data ──┐
       ├──→  Template Engine  ──→  HTML Document  ──→  클라이언트
Template (.ejs) ──┘
```

```bash
npm install ejs
```

```js
app.set("view engine", "ejs");
app.set("views", "./views");

app.get("/", (req, res) => {
  res.render("index", { username: "Incheon", items: ["A", "B"] });
});
```

### EJS 문법

| 태그 | 용도 |
|------|------|
| `<% %>` | JS 코드 실행 (출력 없음, 조건문/반복문 등) |
| `<%= %>` | 값 출력 (XSS 방지 이스케이프 처리) |
| `<%- %>` | HTML 태그 포함 원본 출력 (XSS 주의) |

```html
<h1>Welcome, <%= username %>!</h1>
<ul>
  <% items.forEach((item) => { %>
    <li><%= item %></li>
  <% }); %>
</ul>
```

---

