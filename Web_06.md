# Modules (모듈)

---

## Module 개념

- 재사용 가능한 코드의 집합
- 각 모듈은 **독립적인 네임스페이스**를 가짐
- 모듈 내 정의된 상수, 변수, 함수, 클래스는 **기본적으로 private** (내부용)
- `export`/`import`를 통해 필요한 것만 **공개**

---

## Classes, Objects, Closures를 이용한 모듈화

### 클로저를 통한 모듈 패턴

```javascript
// math.js
const mathModule = (function() {
    // Private 변수
    const PI = 3.14159;
    
    // Private 함수
    function privateHelper() {
        return "숨겨진 함수";
    }
    
    // Public 인터페이스
    return {
        add: function(a, b) {
            return a + b;
        },
        multiply: function(a, b) {
            return a * b;
        },
        getPI: function() {
            return PI;
        }
    };
})();

// app.js
console.log(mathModule.add(2, 3));        // 5
console.log(mathModule.getPI());          // 3.14159
console.log(mathModule.PI);               // undefined (private)
```

### 클래스를 이용한 모듈

```javascript
// calculator.js
class Calculator {
    constructor() {
        this.result = 0;
    }
    
    add(a, b) {
        this.result = a + b;
        return this;result;
    }
    
    multiply(a, b) {
        this.result = a * b;
        return this.result;
    }
}

// app.js
const calc = new Calculator();
calc.add(5, 3);      // 8
calc.multiply(2, 4); // 8
```

---

## Node.js의 모듈 시스템

### CommonJS: module.exports & require()

- Node.js의 **공식 모듈 시스템**
- 각 파일은 **독립적인 모듈**로 취급
- 모듈 내 정의된 것들은 자동으로 private

**모듈 내보내기 (Export)**

```javascript
// stats.js
function mean(data) {
    return data.reduce((a, b) => a + b) / data.length;
}

function median(data) {
    const sorted = data.sort((a, b) => a - b);
    const mid = Math.floor(sorted.length / 2);
    return sorted[mid];
}

// Public 인터페이스 정의
module.exports = {
    mean,
    median
};
```

**모듈 불러오기 (Import)**

```javascript
// app.js
const { mean, median } = require('./stats.js');

const numbers = [1, 2, 3, 4, 5];
console.log(mean(numbers));    // 3
console.log(median(numbers));  // 3
```

**실행**

```bash
$ node app.js
```

### Node.js의 한계: 브라우저에서 사용 불가

```html
<!-- 브라우저 환경에서는 require() 미지원 -->
<script src="stats.js"></script>
<script src="app.js"></script>
<!-- ❌ ReferenceError: require is not defined -->
```

### 해결책: Webpack Module Bundler

**Webpack의 역할:**
- 모듈을 **번들링** (하나의 파일로 통합)
- CommonJS 모듈을 브라우저에서 사용 가능하도록 변환
- Entry point (입력)을 시작으로 dependency graph 분석 후 output 생성

**Webpack 설치 및 설정**

```bash
# 1. 프로젝트 디렉토리 생성
$ mkdir WebIncheon
$ cd WebIncheon
$ code .

# 2. package.json 생성
$ npm init

# 3. Webpack 설치
$ npm install --save-dev webpack
$ npm install --save-dev webpack-cli
```

**webpack.config.js 설정**

```javascript
module.exports = {
    mode: 'development',           // 또는 'production'
    entry: './src/index.js',       // Entry point (시작점)
    output: {
        path: __dirname + '/dist',  // 출력 디렉토리
        filename: 'main.js'         // 번들된 파일명
    }
};
```

**빌드 및 실행**

```bash
# npm 스크립트에 build 명령 추가
$ npm run build

# 생성된 dist/main.js 확인
# dist/index.html에서 dist/main.js 로드
<script src="main.js"></script>
```

**번들링 과정:**

```
src/
├── index.js (entry point)
├── stats.js (모듈 1)
└── utils.js (모듈 2)
     ↓
  Webpack 분석
     ↓
dist/
└── main.js (단일 번들 파일)
```

---

## ES6의 모듈 시스템

### ES6 import & export

- **클라이언트 측 모듈화**의 표준
- `export`와 `import` 키워드 사용
- CommonJS와 유사하지만 **브라우저 네이티브 지원**

### Named Export & Import

**모듈 내보내기 (Export)**

```javascript
// math.js
export const add = (a, b) => a + b;

export const subtract = (a, b) => a - b;

export const PI = 3.14159;

export function multiply(a, b) {
    return a * b;
}
```

**모듈 불러오기 (Import)**

```javascript
// app.js
import { add, subtract, PI, multiply } from './math.js';

console.log(add(5, 3));        // 8
console.log(PI);               // 3.14159
console.log(multiply(4, 2));   // 8
```

### Default Export & Import

**하나의 기본 내보내기**

```javascript
// calculator.js
class Calculator {
    add(a, b) { return a + b; }
    multiply(a, b) { return a * b; }
}

export default Calculator;
```

**기본 import**

```javascript
// app.js
import Calculator from './calculator.js';  // 중괄호 없음

const calc = new Calculator();
console.log(calc.add(2, 3));  // 5
```

### Named와 Default 혼합

```javascript
// utils.js
export const PI = 3.14159;
export const E = 2.71828;

export default function greeting(name) {
    return `Hello, ${name}!`;
}
```

```javascript
// app.js
import greeting, { PI, E } from './utils.js';

console.log(greeting("이용균"));  // Hello, 이용균!
console.log(PI);                  // 3.14159
```

### HTML에서 ES6 모듈 사용

```html
<!-- index.html -->
<!DOCTYPE html>
<html>
<head>
    <title>ES6 Modules</title>
</head>
<body>
    <!-- type="module" 속성 필수 -->
    <script type="module" src="app.js"></script>
</body>
</html>
```

### ES6 모듈의 특징

| 특징 | 설명 |
|---|---|
| **Strict Mode** | 모듈은 자동으로 strict mode 실행 (`"use strict"` 불필요) |
| **Private by Default** | 모듈 내 정의된 것은 기본적으로 private |
| **Async Loading** | 모듈은 비동기로 로드됨 (렌더링 블로킹 없음) |
| **Top-level await** | 모듈의 최상위에서 `await` 사용 가능 |

### Node.js에서 ES6 모듈 사용

**package.json에서 설정:**

```json
{
  "type": "module"
}
```

또는 파일명을 `.mjs`로 변경

```bash
# stats.mjs
export function mean(data) {
    return data.reduce((a, b) => a + b) / data.length;
}

# app.mjs
import { mean } from './stats.mjs';
```

---

## Node.js vs ES6 모듈 비교

| 항목 | Node.js (CommonJS) | ES6 |
|---|---|---|
| 문법 | `module.exports` / `require()` | `export` / `import` |
| 실행 환경 | Node.js (서버) | 브라우저 (클라이언트) |
| 로딩 방식 | 동기 (synchronous) | 비동기 (asynchronous) |
| Strict Mode | 명시 필요 | 자동 적용 |
| 브라우저 지원 | ❌ (Webpack 필요) | ✅ (직접 사용 가능) |

---

## 모듈 시스템의 장점

1. **캡슐화** — Private 데이터 보호
2. **재사용성** — 모듈을 여러 파일에서 공유
3. **의존성 관리** — 명시적 import/export로 관계 파악 용이
4. **네임스페이스 격리** — 변수명 충돌 방지
5. **유지보수성** — 모듈 단위로 코드 관리
