# Asynchronous Programming (비동기 프로그래밍)

---

## 동기 vs 비동기

### 동기 (Synchronous)

> **동기**: 코드가 **순서대로 실행**되며, 각 줄이 완료될 때까지 다음 줄은 실행되지 않음

```javascript
console.log("1. 시작");

function heavyTask() {
    let sum = 0;
    for (let i = 0; i < 1000000000; i++) {
        sum += i;
    }
    return sum;
}

const result = heavyTask();  // 이 작업이 끝날 때까지 대기
console.log("2. 결과:", result);
console.log("3. 끝");

// 출력 순서: 1 → 2 → 3 (항상 순서대로)
```

**동기의 문제:**
- 긴 작업이 있으면 **전체 프로그램이 멈춤** (blocking)
- 사용자 입력에 응답할 수 없음

### 비동기 (Asynchronous)

> **비동기**: 오래 걸리는 작업을 **백그라운드에서 처리**하고, 그동안 다른 코드를 먼저 실행
> 작업 완료 후 **콜백 또는 Promise로 결과를 받음**

```javascript
console.log("1. 시작");

setTimeout(() => {
    console.log("2. 1초 후 실행");
}, 1000);

console.log("3. 끝");

// 출력 순서: 1 → 3 → 2
```

**비동기의 장점:**
- 프로그램이 **멈추지 않음** (non-blocking)
- 사용자 입력에 **바로 응답**
- 여러 작업을 **동시에 처리**

---

## JavaScript의 Event Loop

### 비동기가 작동하는 원리

> **Event Loop**: JavaScript 엔진이 **비동기 작업의 완료를 감시**하고,
> 완료된 작업의 **콜백을 차례대로 실행**하는 메커니즘

### 구성 요소

```
┌─────────────────────────────────┐
│   JavaScript Engine             │
├──────────────┬──────────────────┤
│ Call Stack   │ Global Memory    │
│ (실행 중인   │ (변수, 함수)    │
│  함수)       │                  │
└──────┬───────┴──────────────────┘
       │
       ├─→ Web APIs (setTimeout, fetch 등)
       │
       ├─→ Task Queue (완료된 콜백)
       │
       └─→ Event Loop (실행 감시)
```

### 동작 흐름

```javascript
console.log("A");                // Call Stack 실행

setTimeout(() => {
    console.log("B");            // Task Queue에서 실행
}, 0);

console.log("C");                // Call Stack 실행

// 실행 순서:
// 1. "A" (Call Stack)
// 2. setTimeout 호출 → Web APIs로 이동
// 3. "C" (Call Stack)
// 4. Call Stack 비워짐 → Event Loop이 Task Queue 확인
// 5. "B" (Task Queue의 콜백)

// 최종 출력: A → C → B
```

---

## setTimeout (타이머)

### 개념

> **setTimeout**: **일정 시간 후에 함수를 실행**하는 비동기 함수
> 비동기의 가장 기초적인 예시

```javascript
console.log("요청");

setTimeout(() => {
    console.log("1초 후 실행");
}, 1000);  // 1000밀리초(ms) = 1초

console.log("진행 중");

// 출력: 요청 → 진행 중 → 1초 후 실행
```

### 사용법

**기본 문법:**
```javascript
const timeoutId = setTimeout(callback, delay);
```

**익명함수로 사용 (가장 흔함):**
```javascript
setTimeout(() => {
    console.log("실행");
}, 1000);
```

### clearTimeout (취소)

```javascript
const timeoutId = setTimeout(() => {
    console.log("5초 후");
}, 5000);

// 2초 후 취소
setTimeout(() => {
    clearTimeout(timeoutId);
    console.log("취소됨");
}, 2000);
```

### setInterval (반복)

```javascript
let count = 0;

const intervalId = setInterval(() => {
    count++;
    console.log(`${count}초`);
}, 1000);

// 5초 후 중지
setTimeout(() => {
    clearInterval(intervalId);
}, 5000);
```

---

## 콜백 (Callback)

### 개념

> **콜백**: **나중에 호출될 함수**를 인자로 전달
> 비동기 작업 완료 후 실행할 함수

```javascript
// setTimeout의 콜백
setTimeout(() => {
    console.log("콜백 실행");
}, 1000);

// 배열 메서드의 콜백
[1, 2, 3].forEach((item) => {
    console.log(item);
});

// 이벤트의 콜백
button.addEventListener('click', () => {
    console.log("클릭됨");
});
```

### 에러 퍼스트 콜백 (Error-First Callback)

> **에러 퍼스트 콜백**: **첫 번째 인자를 에러 객체로 받는 패턴**
> Node.js 표준 관례

```javascript
function fetchData(callback) {
    setTimeout(() => {
        if (Math.random() > 0.5) {
            callback(null, { data: "성공" });  // error는 null
        } else {
            callback(new Error("실패"), null);  // error 전달
        }
    }, 1000);
}

fetchData((error, data) => {
    if (error) {
        console.log("에러:", error.message);
    } else {
        console.log("데이터:", data);
    }
});
```

### Callback Hell (콜백 지옥)

> **콜백 헬**: 콜백이 계속 중첩되면서 **코드가 복잡해지는 현상**

**레벨 1 (문제 없음):**
```javascript
getUser(1, (user) => {
    console.log(user);
});
```

**레벨 2 (읽기 어려워짐):**
```javascript
getUser(1, (user) => {
    getPosts(user.id, (posts) => {
        console.log(posts);
    });
});
```

**레벨 3 (콜백 지옥):**
```javascript
// ❌ 너무 깊게 중첩됨
getUser(1, (user) => {
    getPosts(user.id, (posts) => {
        getComments(posts[0].id, (comments) => {
            getLikes(comments[0].id, (likes) => {
                console.log(likes);
            });
        });
    });
});
```

**해결책: Promise 또는 async/await 사용**

---

## Promise (약속)

### 개념

> **Promise**: 비동기 작업의 **상태와 결과를 객체로 표현**
> 콜백 헬을 해결하는 현대적 방식

```javascript
const promise = new Promise((resolve, reject) => {
    setTimeout(() => {
        resolve("성공!");
    }, 1000);
});

console.log(promise);  // Promise { <pending> }
```

### 3가지 상태

**1. Pending (대기)**
```javascript
const promise = new Promise((resolve, reject) => {
    // resolve/reject 호출 전 = pending
});
console.log(promise);  // Promise { <pending> }
```

**2. Fulfilled (성공)**
```javascript
const promise = new Promise((resolve) => {
    resolve("완료!");  // fulfilled 상태로 전환
});

promise.then(result => {
    console.log(result);  // "완료!"
});
```

**3. Rejected (실패)**
```javascript
const promise = new Promise((resolve, reject) => {
    reject(new Error("실패!"));  // rejected 상태로 전환
});

promise.catch(error => {
    console.log(error.message);  // "실패!"
});
```

### 상태 전이

```
Pending (대기)
   ↙      ↘
resolve  reject
   ↓        ↓
Fulfilled  Rejected
(최종 상태, 변경 불가)
```

**한 번 변경되면 다시 변할 수 없음:**
```javascript
const promise = new Promise((resolve, reject) => {
    resolve("첫 번째");
    reject("두 번째");  // 무시됨
});

promise.then(result => console.log(result));  // "첫 번째"
```

### Promise의 3가지 메서드

**then() - 성공 처리 (Fulfilled)**
```javascript
promise.then((result) => {
    console.log("성공:", result);
});
```

**catch() - 실패 처리 (Rejected)**
```javascript
promise.catch((error) => {
    console.log("실패:", error.message);
});
```

**finally() - 항상 실행**
```javascript
promise
    .then(result => console.log("성공:", result))
    .catch(error => console.log("실패:", error))
    .finally(() => console.log("완료"));
```

### Promise 체이닝

메서드들이 새로운 Promise를 반환하므로 **체이닝이 가능**하다.

```javascript
Promise.resolve(10)
    .then(num => {
        console.log("첫 번째:", num);
        return num * 2;  // 20
    })
    .then(num => {
        console.log("두 번째:", num);
        return num + 5;  // 25
    })
    .then(num => {
        console.log("세 번째:", num);  // 25
    });

// 출력: 첫 번째: 10 → 두 번째: 20 → 세 번째: 25
```

### 콜백 반환값에 따른 동작

**반환값이 있는 경우 (일반 값):**
```javascript
Promise.resolve(10)
    .then(num => {
        return num * 2;  // 반환값 20
    })
    .then(num => {
        console.log(num);  // 20 (즉시 fulfilled)
    });
```

**Promise를 반환하는 경우:**
```javascript
Promise.resolve(10)
    .then(num => {
        // Promise 반환 = "resolved but not yet fulfilled"
        return new Promise((resolve) => {
            setTimeout(() => {
                resolve(num * 2);
            }, 1000);
        });
    })
    .then(num => {
        console.log(num);  // 1초 후 20 (promise settle 대기)
    });
```

**반환값이 없는 경우:**
```javascript
Promise.resolve(10)
    .then(num => {
        console.log(num);
        // 반환값 없음 = undefined로 fulfilled
    })
    .then(value => {
        console.log(value);  // undefined
    });
```

---

## Fetch API (네트워크 요청)

### 개념

> **Fetch**: **Promise를 반환하는 비동기 API**
> HTTP 요청을 보내고 응답을 받음

```javascript
// fetch는 Promise 반환
fetch('https://api.example.com/users')
    .then(response => response.json())
    .then(data => console.log(data))
    .catch(error => console.log(error));
```

### Response 객체와 바디 접근

```javascript
fetch('https://api.example.com/users')
    .then(response => {
        // response: HTTP 응답 객체
        console.log(response.status);  // 200
        console.log(response.ok);      // true
        
        // 바디를 JSON으로 파싱 (Promise 반환)
        return response.json();
    })
    .then(data => {
        console.log("데이터:", data);
    });
```

**text() 메서드 (텍스트 읽기):**
```javascript
fetch('file.txt')
    .then(response => response.text())  // Promise 반환
    .then(text => console.log(text));
```

**json() 메서드도 Promise 반환:**
```javascript
// response.json()도 Promise를 반환
// 왜? 응답 바디가 크면 읽는데 시간이 걸리기 때문
fetch('https://api.example.com/users')
    .then(response => response.json())  // Promise 반환
    .then(data => console.log(data));   // json() 완료 후 실행
```

### 데이터 전달 (POST)

```javascript
fetch('https://api.example.com/users', {
    method: 'POST',
    headers: {
        'Content-Type': 'application/json'
    },
    body: JSON.stringify({
        name: '이용균',
        email: 'user@example.com'
    })
})
.then(response => response.json())
.then(newUser => console.log("생성됨:", newUser));
```

### 반환값을 변수에 저장

```javascript
// Promise를 변수에 저장
const fetchPromise = fetch('https://api.example.com/users');

// 나중에 처리
fetchPromise
    .then(response => response.json())
    .then(data => console.log(data));
```

---

## $.getJSON (jQuery)

> **$.getJSON**: jQuery의 **JSON 비동기 로딩 함수**
> 내부적으로 Promise 반환

```javascript
$.getJSON('https://api.example.com/users')
    .then(data => console.log("데이터:", data))
    .catch(error => console.log("에러:", error));
```

**then()의 두 번째 인자로 에러 처리:**
```javascript
$.getJSON('https://api.example.com/users',
    (data) => {
        console.log("성공:", data);
    },
    (error) => {
        console.log("에러:", error);
    }
);
```

---

## 이벤트 (Event)

### 개념

> **이벤트**: **사용자 또는 브라우저에서 발생하는 동작**
> (클릭, 입력, 페이지 로드 등)

> **이벤트 드리븐 프로그래밍**: **이벤트 발생 시 실행할 콜백을 미리 등록**

### DOM 요소 선택

```javascript
// getElementById
const button = document.getElementById('myButton');

// querySelector (권장)
const button = document.querySelector('#myButton');      // ID
const buttons = document.querySelectorAll('.btn-class');  // Class
```

### 주요 이벤트

| 이벤트 | 설명 |
|---|---|
| **click** | 클릭 |
| **change** | 입력값 변경 |
| **focus** | 포커스 획득 |
| **blur** | 포커스 상실 |
| **keydown** | 키 누름 |
| **keyup** | 키 뗌 |
| **load** | 페이지 로드 완료 |
| **mouseover** | 마우스 진입 |
| **mouseout** | 마우스 이탈 |
| **submit** | 폼 제출 |

### 이벤트 리스너 등록

```javascript
const button = document.querySelector('#submitBtn');

button.addEventListener('click', () => {
    console.log("클릭됨");
});
```

**실제 예시:**
```javascript
// 입력값 변경 감지
document.querySelector('#input').addEventListener('change', (event) => {
    console.log("입력값:", event.target.value);
});

// Enter 키 입력
document.querySelector('#search').addEventListener('keydown', (event) => {
    if (event.key === 'Enter') {
        console.log("검색");
    }
});

// 페이지 로드 완료
window.addEventListener('load', () => {
    console.log("페이지 로드 완료");
});
```

---

## 네트워크와 비동기

### 비동기가 필수인 이유

> **데이터 요청과 수신에 시간이 걸리기 때문**
> 동기로 처리하면 프로그램이 멈춤

```javascript
// ❌ 만약 동기였다면 (불가능)
const data = fetchDataSync('https://api.example.com/users');
// 1초 이상 대기... (프로그램 멈춤)

// ✅ 비동기 (현실)
console.log("요청 시작");

fetch('https://api.example.com/users')
    .then(r => r.json())
    .then(data => console.log("데이터:", data));

console.log("요청 진행 중...");

// 출력: 요청 시작 → 요청 진행 중... → (1초 후) 데이터: {...}
```

---

## async/await (최현대 방식)

### async 함수

> **async**: 함수를 **비동기 함수로 만들고 항상 Promise 반환**

```javascript
async function getUser() {
    return { id: 1, name: "이용균" };  // 자동으로 Promise 반환
}

getUser().then(user => {
    console.log("사용자:", user);
});
```

### await 키워드

> **await**: Promise가 **settle될 때까지 기다렸다가 결과값 반환**
> **async 함수 내에서만 사용 가능**

```javascript
async function getUsers() {
    // Promise 방식 (여러 줄)
    const response = await fetch('https://api.example.com/users');
    const data = await response.json();
    console.log("데이터:", data);
}

getUsers();
```

### async/await vs Promise 비교

**Promise 방식:**
```javascript
fetch('https://api.example.com/users/1')
    .then(r => r.json())
    .then(user => fetch(`https://api.example.com/posts/${user.id}`))
    .then(r => r.json())
    .then(posts => console.log(posts));
```

**async/await 방식 (훨씬 깔끔):**
```javascript
async function fetchAllData() {
    const userRes = await fetch('https://api.example.com/users/1');
    const user = await userRes.json();
    
    const postsRes = await fetch(`https://api.example.com/posts/${user.id}`);
    const posts = await postsRes.json();
    
    console.log(posts);
}

fetchAllData();
```

### 에러 처리 (try/catch)

```javascript
async function getUser() {
    try {
        const response = await fetch('https://api.example.com/users/1');
        
        if (!response.ok) {
            throw new Error(`HTTP ${response.status}`);
        }
        
        const data = await response.json();
        console.log("사용자:", data);
        
    } catch (error) {
        console.log("에러:", error.message);
        
    } finally {
        console.log("요청 완료");
    }
}

getUser();
```

### 병렬 처리 (성능 최적화)

**순차 처리 (느림):**
```javascript
async function sequential() {
    const user = await fetch('...').then(r => r.json());      // 1초
    const posts = await fetch('...').then(r => r.json());     // 1초
    const comments = await fetch('...').then(r => r.json());  // 1초
    // 총 3초
}
```

**병렬 처리 (빠름):**
```javascript
async function parallel() {
    // Promise.all로 동시 실행
    const [user, posts, comments] = await Promise.all([
        fetch('...').then(r => r.json()),
        fetch('...').then(r => r.json()),
        fetch('...').then(r => r.json())
    ]);
    // 총 1초
}
```

---

## HTTPS (보안)

### HTTP vs HTTPS

| 항목 | HTTP | HTTPS |
|---|---|---|
| **포트** | 80 | 443 |
| **암호화** | ❌ | ✅ |
| **보안** | 낮음 | 높음 |

### TLS/SSL 암호화

HTTPS는 **TLS(Transport Layer Security)**로 데이터를 암호화하여 전송한다.

```javascript
// HTTP (평문 - 위험)
fetch('http://api.example.com/login', {
    method: 'POST',
    body: JSON.stringify({ password: 'myPassword' })
});

// HTTPS (암호화 - 안전)
fetch('https://api.example.com/login', {
    method: 'POST',
    body: JSON.stringify({ password: 'myPassword' })
});
```

---

## 비동기 패턴 비교

| 패턴 | 가독성 | 에러 처리 | 사용 시기 |
|---|---|---|---|
| **Callback** | ⚠️ 나쁨 | 복잡 | 기초 학습 |
| **Promise** | ✅ 좋음 | 간단 | 중간 |
| **async/await** | ✅ 매우 좋음 | try/catch | 실무 (권장) |

---

## 실전 예제

### 사용자 정보와 포스트 가져오기

```javascript
async function getUserWithPosts(userId) {
    try {
        // 사용자 정보
        const userRes = await fetch(`https://api.example.com/users/${userId}`);
        const user = await userRes.json();
        
        // 포스트 목록
        const postsRes = await fetch(`https://api.example.com/posts?userId=${userId}`);
        const posts = await postsRes.json();
        
        return { user, posts };
        
    } catch (error) {
        console.log("데이터 로드 실패:", error.message);
        return null;
    }
}

// 사용
getUserWithPosts(1).then(data => {
    if (data) {
        console.log("사용자:", data.user);
        console.log("포스트:", data.posts);
    }
});
```

### 폼 제출

```javascript
const form = document.querySelector('#userForm');

form.addEventListener('submit', async (event) => {
    event.preventDefault();
    
    const formData = new FormData(form);
    const userData = Object.fromEntries(formData);
    
    try {
        const response = await fetch('https://api.example.com/users', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify(userData)
        });
        
        const result = await response.json();
        console.log("생성됨:", result);
        
    } catch (error) {
        console.log("제출 실패:", error.message);
    }
});
```

---

## 정리

| 개념 | 설명 |
|---|---|
| **동기** | 순서대로 실행, 끝날 때까지 대기 |
| **비동기** | 백그라운드 실행, 다른 코드 먼저 실행 |
| **Event Loop** | 비동기 작업 감시, 콜백 실행 |
| **setTimeout** | 지연 실행 |
| **콜백** | 나중에 호출될 함수 |
| **Promise** | 비동기 상태 객체 (pending → fulfilled/rejected) |
| **then/catch** | Promise 성공/실패 처리 |
| **async/await** | Promise를 동기처럼 작성 |
| **fetch** | Promise 기반 네트워크 API |
| **HTTPS** | TLS로 암호화된 통신 |
