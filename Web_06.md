# Modules (모듈)

---

## 모듈화 이전: 전역 스코프의 문제점

### 함수를 전역 스코프에 노출할 때의 문제

**초기 방식: 스크립트 파일을 분리하고 전역으로 함수 노출**

```html
<!-- index.html -->
<script src="math.js"></script>
<script src="utils.js"></script>
<script src="app.js"></script>
```

```javascript
// math.js
function add(a, b) {
    return a + b;
}

function multiply(a, b) {
    return a * b;
}

// utils.js
function add(a, b) {  // 같은 이름의 함수
    return a + b + 10;
}

// app.js
console.log(add(2, 3));  // 어느 add()가 실행될까?
```

### 발생하는 문제들

- **네임스페이스 충돌**: 같은 이름의 함수가 있으면 마지막에 정의된 것으로 덮어씀
- **의도하지 않은 덮어쓰기**: 다른 파일의 함수를 실수로 수정할 수 있음
- **전역 오염**: 전역 스코프에 너무 많은 함수/변수가 쌓임
- **의존성 파악 어려움**: 어떤 함수가 어디서 정의되었는지 추적 어려움
- **코드 관리 어려움**: 규모가 커질수록 유지보수가 힘들어짐

---

## Module 개념

- 재사용 가능한 코드의 집합
- 각 모듈은 **독립적인 네임스페이스**를 가짐
- 모듈 내 정의된 상수, 변수, 함수, 클래스는 **기본적으로 private** (내부용)
- `export`/`import`를 통해 필요한 것만 **공개**

---

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
        return this.result;
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

---

## Node.js의 모듈 시스템

### Node.js 환경이란

- **Node.js**: 서버에서 JavaScript를 실행하는 환경
- 브라우저와 달리 **서버 컴퓨터에서 실행**
- 파일 시스템 접근, 네트워크 통신 등이 가능
- 터미널에서 `node 파일명.js` 명령으로 실행

### CommonJS: module.exports & require()

- Node.js의 **공식 모듈 시스템**
- 각 파일은 **독립적인 모듈**로 취급
- 모듈 내 정의된 것들은 자동으로 private

### module.exports와 require()

- **module.exports**: 모듈에서 **내보낼 것**을 정의
- **require()**: 다른 모듈의 **내보낸 것**을 가져옴
- 각 파일은 독립적인 스코프를 가지므로 변수/함수 충돌 없음

### require() 상세 설명

**require()의 역할:**
- 다른 모듈 파일의 `module.exports` 객체를 **가져옴**
- CommonJS의 **import 기능**에 해당
- 동기(synchronous)로 모듈을 로드 (파일 읽을 때까지 대기)

**사용 방식:**
```javascript
// 전체 객체 가져오기
const stats = require('./stats.js');
stats.mean([1, 2, 3]);

// 구조 분해로 필요한 것만 가져오기
const { mean, median } = require('./stats.js');
mean([1, 2, 3]);
```

### 모듈 내보내기 (Export)

> **모듈 내보내기**: 함수나 변수를 **다른 파일에서 사용할 수 있도록 공개**하는 것
> `module.exports`에 지정한 것들만 외부에서 접근 가능

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

### 모듈 불러오기 (Import)

> **모듈 불러오기**: 다른 파일에서 **내보낸 함수/변수를 가져와서 사용**하는 것
> `require()`로 가져온 내용은 현재 파일에서 그대로 사용 가능

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

---

---

## Webpack Module Bundler

### 웹 환경에서의 모듈 문제

**Node.js의 CommonJS는 브라우저에서 동작하지 않음:**

```html
<!-- 브라우저 환경에서는 require() 미지원 -->
<script src="stats.js"></script>
<script src="app.js"></script>
<!-- ReferenceError: require is not defined -->
```

**문제:**
- Node.js 환경에서는 파일 시스템으로 모듈을 로드
- 브라우저는 파일 시스템에 접근할 수 없음 (보안)
- `require()` 함수가 없음

### Webpack의 역할

**Webpack의 기능:**
- 모듈을 **번들링** (하나의 파일로 통합)
- CommonJS 모듈을 브라우저에서 사용 가능하도록 변환
- Entry point (입력)을 시작으로 dependency graph 분석 후 output 생성
- 빌드 프로세스: 개발 코드를 프로덕션에 최적화된 형태로 변환

**Webpack이 하는 일:**

- **코드 압축 (Minify)**: 불필요한 공백, 주석 제거해서 파일 크기 줄임
- **번들링**: 여러 `.js` 파일을 분석해서 **하나의 파일로 합침**
- **트리 쉐이킹**: 사용하지 않는 코드 자동 제거로 파일 최적화
- **환경별 설정**: 개발 환경과 프로덕션 환경에 맞게 자동 최적화

### Webpack 설치 및 설정

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

### webpack.config.js의 역할

**webpack.config.js는 Webpack의 설정 파일이다.**

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

**각 항목의 의미:**
- **mode**: 'development' (개발) 또는 'production' (배포) 설정
- **entry**: Webpack이 번들링을 **시작할 진입점 파일**
- **output**: 번들된 파일을 **어디에, 어떤 이름으로 저장**할지 지정
- **path**: 번들 파일이 생성될 디렉토리 경로
- **filename**: 번들 파일의 이름

### package.json의 scripts와 webpack.config.js

**package.json의 scripts 섹션:**

```json
{
  "name": "my-project",
  "version": "1.0.0",
  "scripts": {
    "build": "webpack",
    "dev": "webpack --config webpack.config.js"
  },
  "devDependencies": {
    "webpack": "^5.0.0",
    "webpack-cli": "^4.0.0"
  }
}
```

**scripts 사용:**

```bash
# npm run build 실행 시
$ npm run build
# → webpack 명령 실행
# → webpack.config.js 자동으로 찾아서 설정 적용

# 명시적으로 설정 파일 지정
$ npm run dev
# → webpack --config webpack.config.js 실행
```

**webpack.config.js 명시적 지정:**
- `webpack` (기본): webpack.config.js 자동 찾음
- `webpack --config webpack.config.js` (명시): 파일을 직접 지정
- 여러 설정 파일 필요시 (개발/배포 분리) 각각 지정 가능

### package.json으로 개발 환경 동일화

**package.json의 역할:**
- 프로젝트에 필요한 **패키지와 버전**을 기록
- 모든 개발자가 **같은 환경**을 구성할 수 있게 함

**package.json 예시:**

```json
{
  "name": "my-project",
  "version": "1.0.0",
  "devDependencies": {
    "webpack": "^5.0.0",
    "webpack-cli": "^4.0.0",
    "babel-loader": "^8.0.0"
  },
  "dependencies": {
    "react": "^18.0.0",
    "axios": "^1.0.0"
  }
}
```

**환경 동일화 방법:**

```bash
# 개발자 A
$ git clone 프로젝트
$ npm install
# → package.json에 명시된 모든 패키지 자동 설치

# 개발자 B
$ git clone 프로젝트
$ npm install
# → 같은 패키지, 같은 버전 설치
# → 개발 환경 완벽하게 동일!
```

**정확한 버전 보장: package-lock.json**
- `npm install` 실행 시 자동 생성
- 정확한 버전과 의존성을 **모두 기록**
- 모든 개발자가 **정확히 같은 버전** 설치 가능
- 호환성 문제 방지

### 번들링 과정

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

**dist 폴더:**
- **dist** = "distribution" (배포)
- Webpack이 번들링한 **최종 결과물** 저장

**dist를 사용하는 이유:**

- **파일 크기 최소화**: 압축과 최적화로 네트워크 전송 속도 향상
- **성능 향상**: 하나의 파일로 HTTP 요청 횟수 감소
- **브라우저 호환성**: 모듈 시스템을 모든 브라우저에서 사용 가능하게 변환

**dist 폴더의 특징:**

- **압축된 코드**: 공백과 주석 제거로 크기 최소화
- **최적화된 코드**: 불필요한 코드 자동 제거 (트리 쉐이킹)
- **프로덕션 배포용**: 실제 서버에 올릴 때 dist 폴더의 파일 사용
- **개발 코드와 분리**: src는 개발, dist는 배포 - 서로 영향 없음

**빌드 및 실행**

```bash
# npm 스크립트에 build 명령 추가
$ npm run build

# 생성된 dist/main.js 확인
# dist/index.html에서 dist/main.js 로드
<script src="main.js"></script>
```

---

---

## ES6의 모듈 시스템

### ES6 import & export

- **클라이언트 측 모듈화**의 표준
- `export`와 `import` 키워드 사용
- CommonJS와 유사하지만 **브라우저 네이티브 지원**

### Named Export & Import

> **Named Export**: 여러 개의 함수/변수를 **이름을 지정해서 내보내기**
> **Named Import**: 내보낸 것들을 **필요한 것만 선택해서 가져오기**

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

> **Default Export**: **하나의 기본값만 내보내기** (함수, 클래스, 객체 등)
> **Default Import**: 내보낸 기본값을 받아와서 자유롭게 이름 지정 가능

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

> **혼합 사용**: Named Export와 Default Export를 **동시에 사용** 가능
> 하나의 모듈에서 여러 항목과 기본값을 동시에 제공

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
| **Strict Mode** | 모듈은 자동으로 strict mode 실행 |
| **Private by Default** | 모듈 내 정의된 것은 기본적으로 private |
| **Async Loading** | 모듈은 비동기로 로드됨 (렌더링 블로킹 없음) |
| **Top-level await** | 모듈의 최상위에서 `await` 사용 가능 |

### Node.js에서 ES6 모듈 사용

**package.json의 type 속성:**

- `"type": "commonjs"` (기본값): CommonJS 사용 (require/module.exports)
- `"type": "module"`: ES6 모듈 사용 (import/export)

**package.json에서 설정:**

```json
{
  "type": "module"
}
```

이 설정만으로 모든 `.js` 파일이 **자동으로 ES6 모듈**로 인식된다.

**ES6 모듈 사용 방법:**

```javascript
// stats.js (파일 확장자는 그대로)
export function mean(data) {
    return data.reduce((a, b) => a + b) / data.length;
}

// app.js (파일 확장자는 그대로)
import { mean } from './stats.js';
console.log(mean([1, 2, 3]));
```

**또는 명시적으로 파일명으로 지정:**

- `.mjs` 확장자 사용 시 자동으로 ES6 모듈로 인식
- package.json 설정이 없어도 작동

```javascript
// stats.mjs
export function mean(data) {
    return data.reduce((a, b) => a + b) / data.length;
}

// app.mjs
import { mean } from './stats.mjs';
```

**정리:**
- `"type": "module"` 설정 → 모든 `.js` 파일이 ES6 모듈
- 또는 `.mjs` 확장자 사용 → 해당 파일만 ES6 모듈
- 파일명 변경 필요 없음

---

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

---

## 모듈 시스템의 장점

- **캡슐화**: Private 데이터 보호
- **재사용성**: 모듈을 여러 파일에서 공유
- **의존성 관리**: 명시적 import/export로 관계 파악 용이
- **네임스페이스 격리**: 변수명 충돌 방지
- **유지보수성**: 모듈 단위로 코드 관리
