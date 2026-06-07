# React Basic Syntax 강의 노트

---

## 1. 초기 세팅

```bash
npm create vite@latest
npm i
```

---

## 2. JSX

> JavaScript 안에서 HTML을 작성할 수 있게 해주는 문법

### 주요 규칙

| 규칙 | 설명 |
|------|------|
| 단일 루트 요소 | 여러 요소 반환 시 부모 태그 또는 `<> </>` (Fragment)로 감싸야 함 |
| 태그 닫기 | `<img />` 처럼 self-closing 태그도 반드시 닫아야 함 |
| camelCase | 속성명은 camelCase 사용 (`class` → `className`) |
| 예약어 대체 | JS 예약어는 대체 속성명 사용 (`class` → `className`) |

---

## 3. Component

> UI를 독립적으로 재사용할 수 있는 단위. JavaScript 함수 안에 markup을 embedding해서 사용

- 컴포넌트 이름은 **대문자**로 시작
- 컴포넌트는 다른 컴포넌트를 포함(렌더링)할 수 있음

```
App (부모)
 ├── Avatar (자식)
 ├── Avatar (자식)
 └── Avatar (자식)
```

---

## 4. Props

> 부모 컴포넌트가 자식 컴포넌트에 데이터를 전달하는 방법

- Props는 **읽기 전용(immutable)** — 자식에서 직접 수정 불가
- 수정이 필요한 데이터는 **State**로 관리해야 함

```jsx
// 부모
<Avatar picture={picture[0]} />

// 자식
export default function Avatar({ picture }) {
  const { src, height, name } = picture;
  return <img src={src} height={height} />;
}
```

### 리스트 렌더링
```jsx
data.map((item) => (
  <Incheon key={item.id} name={item.name} />
))
```
- 리스트 렌더링 시 반드시 **`key`** 속성 필요

---

## 5. Conditional Rendering

> 조건에 따라 다른 UI를 렌더링

```jsx
{ 조건 ? ( <> true일 때 </> ) : ( <> false일 때 </> ) }
```
- `null` 반환 시 아무것도 렌더링되지 않음
- JSX 내부 이벤트 속성도 camelCase (`onClick`, `onChange` 등)

```jsx
// 예시 — 다크모드 토글
const [isDark, setIsDark] = useState(false);
<h1>{isDark ? "🌙 Dark Mode" : "☀ Light Mode"}</h1>
<button onClick={() => setIsDark(!isDark)}>전환</button>
```

---

## 6. State

> 컴포넌트의 메모리 — 값이 바뀌면 리렌더링 발생

```jsx
const [count, setCount] = useState(0);
```

- State는 직접 수정 금지 → 반드시 `setState` 함수 사용
- State를 자식에게 내려줄 수도 있음 (Props로 전달)

### Prop Drilling 문제
- State를 여러 단계의 자식에게 계속 Props로 전달해야 하는 구조
- 컴포넌트가 깊어질수록 관리가 복잡해짐
- 해결책 → **전역 상태 관리 라이브러리 (Zustand)**

```
App (state 보유)
 └── Child1 (props로 전달)
       └── Child2 (props로 전달)
             └── Child3 (여기서 쓰고 싶은데...)
```

---

## 7. Hooks — useEffect

> 컴포넌트의 생명주기에 맞춰 특정 코드를 실행하는 Hook

```jsx
useEffect(() => {
  // 실행할 코드
}, [의존성 배열]);
```

| 의존성 배열 | 실행 시점 |
|-------------|-----------|
| 없음 | 렌더링마다 실행 |
| `[]` (빈 배열) | 컴포넌트 마운트 시 1회만 실행 |
| `[값]` | 해당 값이 바뀔 때마다 실행 |

---

## 8. Networking — Axios

> 외부 API 데이터를 가져올 때 사용하는 HTTP 클라이언트 라이브러리

```bash
npm install axios
```

```jsx
// .env
VITE_API_KEY=...
VITE_API_URL=https://api.themoviedb.org

// 사용
axios.get(`${API_URL}/3/discover/movie`, {
  headers: { Authorization: `Bearer ${API_KEY}` }
})
.then((response) => setMovies(response.data.results))
.catch((error) => console.error(error));
```

### 개발 환경 CORS 해결 — Vite Proxy
```js
// vite.config.js
server: {
  proxy: {
    "/endpoint": "http://localhost:3000",
  }
}
```
- `localhost:5173/endpoint` 요청 → Vite가 `localhost:3000/endpoint` 로 프록시

---

## 9. 전역 상태 관리 — Zustand

> Prop Drilling 없이 어느 컴포넌트에서나 상태에 접근할 수 있게 해주는 라이브러리

```bash
npm install zustand
```

### 핵심 개념
- `create()` — 전역 스토어 생성
- `devtools` — Redux DevTools로 상태 디버깅 가능 (Chrome 확장)
- `persist` — 상태를 **localStorage**에 저장해 새로고침 후에도 유지

```js
// stores/ctnStore.js
const useCtnStore = create(
  devtools(
    persist(
      (set) => ({
        ctn: 0,
        increase: () => set((state) => ({ ctn: state.ctn + 1 })),
        decrease: () => set((state) => ({ ctn: state.ctn - 1 })),
      }),
      { name: "ctn-storage" }  // localStorage 키 이름
    )
  )
);
```

```jsx
// 어느 컴포넌트에서든 동일하게 사용
const { ctn, increase, decrease } = useCtnStore();
```

### 객체 State 업데이트 시 — 스프레드 연산자 필수
```js
// 객체 일부만 수정할 때
set((state) => ({
  ctn: { ...state.ctn, age: state.ctn.age + 1 }
}))
```

---

## 10. 스타일링

### ① Inline Style
```jsx
<h1 style={{ color: "red", fontSize: "20px" }}>텍스트</h1>
```
- JS 객체로 작성 → curly brace 두 겹 `{{ }}`
- 속성명 camelCase (`font-size` → `fontSize`)

### ② CSS Module
```jsx
import styles from "./incheon.module.css";
<h1 className={styles.headings}>텍스트</h1>
```
- 파일명: `*.module.css`
- 클래스명이 자동으로 고유하게 변환됨 → **클래스명 충돌 방지**

### ③ Tailwind CSS
```bash
npm install --save-dev tailwindcss@3
npx tailwindcss init -p
```
- 유틸리티 클래스 기반 CSS 프레임워크
- 클래스명으로 바로 스타일 적용: `flex`, `pt-4`, `text-center`, `bg-gray-100` 등
- 참고: https://flowbite.com/tools/tailwind-cheat-sheet/

### ④ Material UI (MUI)
```bash
npm install @mui/material @emotion/react @emotion/styled
```
- Google Material Design 기반 컴포넌트 라이브러리
- `Button`, `Stack` 등 완성된 UI 컴포넌트 바로 사용 가능
- `sx` prop으로 인라인 커스텀 스타일 적용

---

