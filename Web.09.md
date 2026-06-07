# Serverless — React + OpenAI API 강의 노트

> 과목: 웹 프로그래밍 | 담당: 박기석 교수님

---

## 1. 왜 서버리스가 필요한가? — API Key 보안 문제

React는 브라우저에서 실행되는 **프론트엔드** 코드다.  
빌드된 JS 파일은 누구나 열어볼 수 있기 때문에, API Key를 프론트에 직접 넣으면 **외부에 그대로 노출**된다.

> 💡 프론트엔드에는 민감한 정보를 최대한 적게 담아야 한다.

### 해결 방법 — 중간에 서버를 둔다
```
브라우저(React)  →  서버(Serverless Function)  →  OpenAI API
                         ↑
                   API Key는 여기서만 보관
                   (환경변수 process.env로 관리)
```
- 클라이언트는 서버에 요청만 보냄 → API Key를 알 필요가 없음
- 서버 역할을 하는 선택지: **AWS Lambda**, **EC2**, **Vercel Serverless Functions**

---

## 2. Serverless 개념

> 개발자가 직접 서버를 관리할 필요 없이 **코드만 작성해서 실행**할 수 있는 클라우드 컴퓨팅 모델

### 핵심 특징

| 특징 | 설명 |
|------|------|
| 서버 관리 불필요 | 인프라 설정, 유지보수를 클라우드가 대신 처리 |
| 자동 확장 (Auto Scaling) | 트래픽 증가 시 자동으로 인스턴스 늘어남 |
| 이벤트 기반 실행 | HTTP 요청 등 이벤트가 발생할 때만 함수가 실행됨 |
| 사용한 만큼만 비용 지불 | 24시간 서버를 켜두는 EC2와 달리 호출 횟수 기준 과금 |

### 기존 EC2 방식 vs Serverless 비교

| | EC2 | Serverless |
|---|---|---|
| 서버 관리 | 직접 | 불필요 |
| 비용 | 항상 발생 | 호출 시만 발생 |
| 확장 | 수동 설정 | 자동 |
| 적합한 상황 | 항상 실행되는 서비스 | 간헐적 API 처리 |

---

## 3. Vercel

> 프론트엔드 배포 + Serverless Functions를 동시에 제공하는 클라우드 플랫폼

### 주요 기능
- **프론트엔드 배포 및 호스팅** — GitHub 연동으로 push 시 자동 배포
- **Serverless Functions 제공** — `api/` 폴더에 파일만 만들면 자동으로 API 엔드포인트가 됨
- **간단한 백엔드 기능 제공** — DB 연결, 외부 API 호출 등 가벼운 서버 로직 처리 가능

### Vercel Serverless Function 구조
```
프로젝트/
  ├── api/
  │   └── chat.js     ←  /api/chat 엔드포인트 자동 생성
  ├── src/            ←  React 소스
  └── vercel.json     ←  라우팅/헤더 설정
```

### 기본 함수 형태
```js
// api/hello.js
export default function handler(req, res) {
  res.status(200).json({ message: "Hello!" });
}
```
- `req` — 요청 정보 (method, body 등)
- `res` — 응답 객체 (status, json 등)

### HTTP 메서드 분기
```js
if (req.method === "GET") { ... }
else if (req.method === "POST") { ... }
else { res.status(405).json({ message: "Method Not Allowed" }); }
```

### 배포 명령어
```bash
vercel          # Preview 배포 (테스트용)
vercel --prod   # Production 배포 (실서비스)
vercel dev      # 로컬 테스트
```

---

## 4. Nginx — 정적 파일 서빙

- React를 `npm run build` 하면 `dist/` 폴더에 **정적 파일(HTML/JS/CSS)** 생성
- 이 파일들을 **Nginx**가 EC2에서 서빙
- 외부에서 `http://IP:80` 또는 `https://IP:443` 으로 접근

```
EC2
  └── Nginx (Port 80/443)
        └── /dist (빌드된 React 정적 파일 연결)
```

---

## 5. OpenAI API 연동 흐름

### API Key 환경변수 관리
```bash
# Vercel 환경변수 설정 (대시보드 또는 .env)
OPENAI_API_KEY=sk-...
```
- Vercel Function 내부에서만 `process.env.OPENAI_API_KEY`로 접근
- 클라이언트(React)에는 절대 노출되지 않음

### Serverless Function에서 OpenAI 호출
```js
// api/chat.js (핵심 흐름)
const { role, content } = req.body;  // 클라이언트에서 받은 메시지
const client = new OpenAI({ apiKey: process.env.OPENAI_API_KEY });

const response = await client.chat.completions.create({
  model: "gpt-3.5-turbo",
  messages: [{ role, content }],
});

res.status(200).json({ message: response.choices[0].message.content });
```

### OpenAI 응답 구조 (중요)
```json
{
  "choices": [
    {
      "message": { "role": "assistant", "content": "응답 텍스트" },
      "finish_reason": "stop"
    }
  ],
  "usage": { "prompt_tokens": 10, "completion_tokens": 16, "total_tokens": 26 }
}
```
→ 실제 응답 텍스트: `response.choices[0].message.content`

---

## 6. CORS 처리

> **CORS(Cross-Origin Resource Sharing)** — 서로 다른 출처(도메인) 간 요청을 브라우저가 기본적으로 차단하는 보안 정책

- React 앱과 Vercel API가 도메인이 다를 경우 CORS 오류 발생
- 두 가지 방법으로 해결

### ① vercel.json에 헤더 추가
```json
{
  "routes": [{
    "src": "/api/(.*)",
    "headers": {
      "Access-Control-Allow-Origin": "*",
      "Access-Control-Allow-Methods": "GET, POST, PUT, DELETE, OPTIONS"
    }
  }]
}
```

### ② 핸들러에서 OPTIONS(Preflight) 처리
```js
if (req.method === "OPTIONS") return res.status(200).end();
```
- 브라우저는 실제 요청 전에 OPTIONS 요청(Preflight)을 먼저 보냄 → 이걸 처리해줘야 함

---

## 7. 최종 아키텍처 정리

```
[사용자 브라우저]
     │
     ▼
[React App] — Nginx(EC2)로 정적 파일 서빙
     │  fetch POST /api/chat
     ▼
[Vercel Serverless Function] — API Key 여기서만 보관
     │  OpenAI SDK 호출
     ▼
[OpenAI API Server]
```

---


- **OpenAI 응답 접근**: `response.choices[0].message.content`
- **배포**: `vercel`(preview) vs `vercel --prod`(production)
- **Nginx**: React 빌드 결과물을 EC2에서 정적 서빙
