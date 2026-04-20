# Nginx (엔진엑스)

---

## Nginx 개요

- 고성능 오픈소스 웹 서버 및 리버스 프록시
- **정적 파일 제공** (HTML, CSS, JS, 이미지 등)
- **리버스 프록시** 역할로 요청을 백엔드 서버로 전달
- 가볍고 빠르며 높은 동시성 처리 가능

### Client-Side Rendering과의 조합

```
사용자 (클라이언트)
    ↓
Nginx (포트 80, 443)
├── 정적 파일 제공 (React, Vue, Angular 빌드 파일)
└── 특정 경로 요청 → 백엔드 서버로 프록시
    ↓
백엔드 (Express, Flask 등)
    ↓
데이터 반환 (JSON)
```

---

## Nginx as a Proxy (리버스 프록시)

### 리버스 프록시의 역할

- **클라이언트 → Nginx → 백엔드** 구조
- 클라이언트는 백엔드 서버의 실제 IP를 모름
- Nginx가 요청을 받아 적절한 백엔드로 전달

```
클라이언트
    ↓ (http://3.42.12.34/)
Nginx (포트 80, 443)
    ├── / (정적 파일)
    └── /dweb (백엔드 프록시)
        ↓
백엔드 (포트 3000, 5000 등)
```

---

## 전체 웹 서비스 아키텍처

### 아키텍처 1: 정적 파일 제공

```
┌─────────────────────────────────────┐
│  사용자 브라우저                     │
└─────────────┬───────────────────────┘
              │ http://3.42.12.34/
              ↓
┌─────────────────────────────────────┐
│  EC2 인스턴스                        │
│  ┌─────────────────────────────────┐│
│  │ Nginx (포트 80, 443)            ││
│  │ root: /built React 파일         ││
│  └─────────────────────────────────┘│
└─────────────────────────────────────┘
```

### 아키텍처 2: 정적 파일 + 프록시

```
┌─────────────────────────────────────┐
│  사용자 브라우저                     │
│  ├── GET / (정적 파일)              │
│  └── fetch/axios /dweb (JSON)       │
└─────────────┬───────────────────────┘
              │ http://3.42.12.34/
              ↓
┌─────────────────────────────────────┐
│  EC2 인스턴스                        │
│  ┌─────────────────────────────────┐│
│  │ Nginx (포트 80, 443)            ││
│  │ ├── / → 정적 파일              ││
│  │ └── /dweb → 백엔드 프록시       ││
│  └─────────────────────────────────┘│
│  ┌─────────────────────────────────┐│
│  │ Express (포트 3000)             ││
│  │ my_redis (포트 5000)            ││
│  └─────────────────────────────────┘│
└─────────────────────────────────────┘
```

### 아키텍처 3: 외부 API 연동

```
┌────────────────────────────────┐
│  사용자 브라우저               │
└────────────┬────────────────────┘
             │ fetch/axios
             ↓
┌────────────────────────────────┐
│  Nginx                         │
│  ├── / (정적 파일)            │
│  └── /dweb (백엔드 프록시)     │
└────────┬───────────┬───────────┘
         │           │
         ↓           ↓
    Express      외부 API
                ├── https://www.tvmaze.com/api
                └── https://openweathermap.org/
```

---

## Nginx 설치

### macOS (Homebrew)

```bash
# Nginx 설치
$ brew install nginx

# Nginx 정보 확인
$ brew info nginx

# 설정 파일 위치
/opt/homebrew/etc/nginx/nginx.conf

# Nginx 시작
$ nginx

# 브라우저에서 확인
http://localhost:8080

# 기본 파일 위치
/opt/homebrew/var/www

# 설정 리로드
$ nginx -s reload

# 종료
$ nginx -s stop

# 로그 파일
/opt/homebrew/var/log/nginx/error.log
/opt/homebrew/var/log/nginx/access.log

# 설정 파일 위치
$ cd /opt/homebrew/etc/nginx
```

### Ubuntu

**1. Apache 제거 (충돌 방지)**

```bash
# 실행 중인 서비스 확인
$ service --status-all

# Apache 중지 및 제거
$ service apache2 stop
$ apt-get remove apache2*
$ apt-get --purge remove apache2*
$ apt-get autoremove

# Apache 캐시 클리닝 데몬 제거
$ service apache-htcacheclean stop
$ apt-get remove apache*
$ apt-get --purge remove apache*
$ apt-get autoremove
```

**2. OS 업데이트**

```bash
$ apt update
$ apt upgrade
```

**3. Nginx 설치**

```bash
$ sudo apt-get update
$ sudo apt-get install nginx -y

# 방화벽 설정
$ sudo ufw app list
$ sudo ufw allow 'Nginx HTTP'

# Nginx 상태 확인
$ systemctl status nginx  # active (running)

# Nginx 제어 명령어
$ systemctl stop nginx          # 중지
$ systemctl start nginx         # 시작
$ sudo systemctl restart nginx  # 재시작
$ systemctl reload nginx        # 설정 리로드
$ systemctl disable nginx       # 자동 시작 비활성화
$ systemctl enable nginx        # 자동 시작 활성화
```

---

## Nginx 설정 (nginx.conf)

### 기본 구조

```
http block
    ├── upstream block (백엔드 서버 정의)
    └── server block
        └── location block (경로별 처리)
```

### 정적 파일 제공 설정

```nginx
http {
    include mime.types;  # 파일 타입 정의
    
    events {}
    
    server {
        listen 8080;                     # 포트
        root /Users/dweb/nginx;          # 공개 디렉토리
        index index.html;                # 기본 파일
    }
}
```

**설정 확인 및 적용:**

```bash
# 설정 문법 검사
$ nginx -t

# 설정 리로드
$ nginx -s reload
```

### location 블록으로 경로별 처리

```nginx
http {
    include mime.types;
    events {}
    
    server {
        listen 8080;
        root /Users/dweb/nginx;
        
        # / 요청 → /Users/dweb/nginx/index.html
        location / {
            index index.html;
        }
        
        # /incheon 요청 → /Users/dweb/nginx/incheon/index.html
        location /incheon {
            root /Users/dweb/nginx;
        }
        
        # /songdo 요청 → /Users/dweb/nginx/songdo/incheon.html
        location /songdo {
            alias /Users/dweb/nginx;
            index incheon.html;
        }
    }
}
```

**root vs alias:**

| 구분 | root | alias |
|---|---|---|
| 경로 합치기 | location 경로 + root | alias 경로만 사용 |
| 예시 | `/incheon` + `/root` → `/incheon/` | `/songdo` → `/alias/` |

### Upstream으로 백엔드 프록시

```nginx
http {
    include mime.types;
    
    # 백엔드 서버 정의
    upstream backend {
        server 4.23.20.8:3000;
    }
    
    events {}
    
    server {
        listen 80;
        root /Users/dweb/nginx;
        
        # 정적 파일
        location / {
            index index.html;
        }
        
        # /dweb 경로 → 백엔드로 프록시
        location /dweb {
            proxy_pass http://backend;
        }
    }
}
```

**요청 흐름:**

```
클라이언트
    ↓
http://localhost:8080/dweb/software
    ↓
Nginx (/dweb 감지)
    ↓
proxy_pass http://backend (4.23.20.8:3000)
    ↓
Express 서버
    ↓
응답 (JSON)
```

### Express 백엔드 예시

```javascript
// server.js
const express = require("express");
const app = express();

app.get("/dweb/software", function (req, res) {
    res.send("<h1>Software Design</h1>");
});

app.get("/dweb/web", function (req, res) {
    res.send("<h1>Web Programming</h1>");
});

app.listen(3000);
```

**테스트:**

```bash
# Nginx를 통한 요청
http://localhost:8080/dweb/software
http://localhost:8080/dweb/web
```

---

## Docker를 이용한 Nginx 설정

### Dockerfile 구성

```dockerfile
FROM nginx:alpine

# 설정 파일 복사
COPY nginx.conf /etc/nginx/nginx.conf

# 정적 파일 복사
COPY static/ /usr/share/nginx/html/

EXPOSE 80

# Nginx 포그라운드 실행
CMD ["nginx", "-g", "daemon off;"]
```

### nginx.conf (Docker 환경)

```nginx
http {
    include mime.types;
    
    events {}
    
    server {
        listen 80;  # Docker 컨테이너 내부 포트
        root /usr/share/nginx/html;
        
        location / {
            index index.html;
        }
        
        location /incheon {
            root /usr/share/nginx/html;
        }
    }
}
```

### Docker 명령어

```bash
# 이미지 빌드
$ docker build -t nginx-proxy .

# 컨테이너 실행 (8080 → 80 포트 매핑)
$ docker run -d -p 8080:80 nginx-proxy

# 브라우저에서 확인
http://localhost:8080
```

---

## Docker Compose를 이용한 전체 구성

### docker-compose.yml

```yaml
services:
  express:
    build:
      context: ./server
      dockerfile: Dockerfile
    container_name: express-server
    ports:
      - "3000:3000"
    networks:
      - dweb-network
  
  nginx:
    build:
      context: ./proxy
      dockerfile: Dockerfile
    container_name: nginx-proxy
    ports:
      - "8080:80"
    depends_on:
      - express
    networks:
      - dweb-network

networks:
  dweb-network:
    driver: bridge
```

### server/Dockerfile (Express)

```dockerfile
FROM node:16-alpine

WORKDIR /usr/src/app

COPY package*.json ./

RUN npm install

COPY . .

EXPOSE 3000

CMD ["npm", "run", "start"]
```

### proxy/Dockerfile (Nginx)

```dockerfile
FROM nginx:alpine

COPY nginx.conf /etc/nginx/nginx.conf

COPY static/dist /usr/share/nginx/html/

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

### proxy/nginx.conf (Docker Compose 환경)

```nginx
http {
    include mime.types;
    
    # 컨테이너 이름으로 접근
    upstream backend {
        server express-server:3000;
    }
    
    events {}
    
    server {
        listen 80;
        root /usr/share/nginx/html;
        
        # 정적 파일
        location / {
            root /usr/share/nginx/html;
        }
        
        # 백엔드 프록시
        location /dweb {
            proxy_pass http://backend;
        }
    }
}
```

### server/server.js (Express)

```javascript
const express = require("express");
const app = express();

let cnt = 0;

app.get("/dweb", function (req, res) {
    console.log("...");
    cnt += 1;
    res.json({ num: cnt });
});

app.listen(3000, () => {
    console.log("Server is running on port 3000");
});
```

### frontend/app.jsx (React)

```javascript
import { useState, useEffect } from "react";
import "./App.css";

function App() {
    const [data, setData] = useState(null);
    const [count, setCount] = useState(0);
    
    useEffect(() => {
        if (count === 0) return;
        
        const fetchData = async () => {
            try {
                // Nginx를 통한 요청 (프록시)
                const response = await fetch("/dweb/");
                const result = await response.json();
                console.log(result);
                setData(result.num);
            } catch (error) {
                console.error("Error fetching data:", error);
            }
        };
        
        fetchData();
    }, [count]);
    
    return (
        <div className="p-4">
            <button
                onClick={() => setCount((prev) => prev + 1)}
                className="bg-blue-500 text-white px-4 py-2 rounded-md"
            >
                입장하시겠습니까?
            </button>
            
            {data && (
                <div className="mt-4 p-4 border rounded-md">
                    <h3 className="font-bold">{data}</h3>
                </div>
            )}
        </div>
    );
}

export default App;
```

**주의: 직접 백엔드 접근 불가**

```javascript
// ❌ 잘못된 방법
const response = await fetch("http://express-server:3000/dweb");

// ✅ 올바른 방법 (Nginx 프록시)
const response = await fetch("/dweb");
```

### Docker Compose 실행

```bash
# 컨테이너 시작
$ docker-compose up -d

# 브라우저에서 확인
http://localhost:8080

# 컨테이너 중지 및 정리
$ docker-compose down --rmi all -v
```

---

## 실제 환경에서의 고려사항

### IP 주소 변경 문제

```nginx
# ❌ 하드코딩된 IP (변경 시 재설정 필요)
upstream backend {
    server 4.23.20.8:3000;
}
```

**해결책:**
- Docker Compose 사용 (컨테이너 이름으로 자동 해결)
- 환경 변수 사용
- DNS 설정

### Docker Compose의 장점

- 컨테이너 간 **자동 네트워킹** (dweb-network)
- 컨테이너 이름으로 접근 가능 (`express-server:3000`)
- IP 주소 변경 불필요

---

## 요청 흐름 정리

```
사용자 브라우저
    ├── GET http://localhost:8080/
    │   ├── Nginx 정적 파일 제공
    │   └── React 빌드 파일 로드
    │
    └── fetch("/dweb/")
        ├── Nginx가 /dweb 감지
        ├── upstream backend로 프록시
        ├── Express 서버에서 처리
        └── JSON 응답 반환
```
