# Nginx (엔진엑스)

---

## Nginx 개요

- 고성능 오픈소스 웹 서버 및 리버스 프록시
- **정적 파일 제공** (HTML, CSS, JS, 이미지 등)
- **리버스 프록시** 역할로 요청을 백엔드 서버로 전달
- 가볍고 빠르며 높은 동시성 처리 가능

---

## Proxy (프록시) 개념

> **Proxy (프록시)**: "대리인"이라는 뜻으로, **클라이언트와 서버 사이에서 중개 역할**을 하는 것

### Forward Proxy (포워드 프록시)

```
클라이언트 → Forward Proxy → 외부 서버
```

**역할:**
- 클라이언트의 요청을 받음
- 클라이언트를 **대신해서** 외부 서버에 요청
- 외부 서버는 프록시를 거쳐온 것을 모름

**사용 사례:**
- 회사 방화벽 (직원이 인터넷 접속할 때)
- VPN 서비스
- 캐싱 (요청 결과를 저장해두기)

### Reverse Proxy (리버스 프록시)

```
클라이언트 → Reverse Proxy → 백엔드 서버
```

**역할:**
- 클라이언트의 요청을 받음
- 뒤쪽 **백엔드 서버에 전달**
- 클라이언트는 백엔드 실제 주소를 모름

**사용 사례:**
- 로드 밸런싱 (여러 서버에 요청 분산)
- 보안 (백엔드 서버 숨기기)
- 정적 파일 제공

### Forward Proxy vs Reverse Proxy

| 항목 | Forward Proxy | Reverse Proxy |
|---|---|---|
| **방향** | 클라 → 서버 | 클라 → 서버 |
| **숨기는 것** | 클라이언트 정보 | 백엔드 서버 정보 |
| **용도** | 클라이언트 보호 | 백엔드 보호 |
| **예시** | VPN, 회사 방화벽 | Nginx, 로드 밸런서 |

---

## Nginx의 보안 역할

> **Nginx의 보안 이점**: 리버스 프록시로 백엔드 서버를 보호하고,
> 클라이언트의 직접 접근을 차단

### 백엔드 서버 숨기기

**Nginx 없을 때 (위험):**
- 클라이언트가 Express 서버에 직접 접근
- 클라이언트가 Express의 실제 IP와 포트 3000을 알 수 있음
- 해커가 직접 3000번 포트로 공격 시도 가능
- 백엔드 서버가 노출되어 위험

**Nginx 사용 (안전):**
- 클라이언트는 Nginx의 주소(80/443)만 봄
- Express의 실제 주소는 Nginx 뒤에 숨겨짐
- 해커는 내부 포트를 알 수 없음
- 백엔드 서버가 직접 노출되지 않아 훨씬 안전

### Nginx의 주요 보안 기능

**① DDoS 방어**
- Nginx가 먼저 요청을 받음
- 악의적인 요청 필터링 가능
- 악의적인 IP 차단 가능

```nginx
# 특정 IP 차단
server {
    location / {
        deny 192.168.1.100;    # 특정 IP 차단
        allow all;
        index index.html;
    }
}
```

**② SSL/TLS 암호화**
- Nginx에서 HTTPS 처리
- 클라이언트와 Nginx 간 암호화된 통신
- 백엔드는 내부 네트워크에서 HTTP 사용 가능

```nginx
server {
    listen 443 ssl;
    
    ssl_certificate /path/to/cert.pem;
    ssl_certificate_key /path/to/key.pem;
    
    location / {
        proxy_pass http://backend;
    }
}
```

**③ 경로별 접근 제어**
- 특정 경로만 백엔드로 전달
- 나머지는 정적 파일 또는 차단

```nginx
server {
    listen 80;
    root /usr/share/nginx/html;
    
    # 정적 파일만 제공
    location / {
        index index.html;
    }
    
    # 특정 경로만 백엔드로 전달
    location /api {
        proxy_pass http://backend;
    }
    
    # 관리자 경로 (특정 IP만 허용)
    location /admin {
        allow 192.168.1.0/24;   # 내부 네트워크만
        deny all;               # 나머지 차단
        proxy_pass http://backend;
    }
}
```

**④ 요청 헤더 검증**
- 악의적인 헤더 제거 또는 변조
- 클라이언트 정보 숨기기

```nginx
location /api {
    proxy_pass http://backend;
    
    # 원래 IP 정보 전달
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    
    # 불필요한 헤더 제거
    proxy_set_header X-Original-URI $request_uri;
}
```

### Nginx의 보안 이점 정리

| 항목 | 설명 |
|---|---|
| **백엔드 숨기기** | 클라이언트는 Nginx만 봄, 실제 서버 주소 미노출 |
| **DDoS 방어** | 악의적 요청 필터링 및 차단 |
| **SSL/TLS** | 암호화된 통신으로 데이터 보호 |
| **접근 제어** | IP 기반 또는 경로 기반 접근 제한 |
| **요청 검증** | 악의적 헤더 제거, 정상 요청만 전달 |

---

## EC2 (Elastic Compute Cloud)

> **EC2**: AWS의 **가상 서버 서비스**
> 클라우드에서 필요한 성능의 서버를 빌려서 사용

**특징:**
- 물리 서버를 구매할 필요 없음
- 필요에 따라 언제든 추가/삭제 가능
- 전 세계 어디서나 접속 가능
- 사용한 만큼만 비용 지불

**로컬 PC와의 차이:**

| 항목 | 로컬 PC | EC2 |
|---|---|---|
| **위치** | 내 컴퓨터 | AWS 서버 (클라우드) |
| **항상 실행** | ❌ (수동) | ✅ (24/7) |
| **외부 접근** | 어려움 | ✅ (IP로 접근) |
| **확장성** | 제한적 | ✅ (쉬움) |

**Web_07.md 아키텍처에서:**

```
EC2 인스턴스 (AWS 클라우드)
├── Nginx (포트 80/443)
│   ├── 정적 파일 제공
│   └── /dweb → Express로 프록시
└── Express (포트 3000)
    └── API 처리
```

사용자가 EC2의 IP 주소로 접속하면, Nginx가 요청을 받아 처리한다.

---

## 포트(Port)의 개념

> **포트(Port)**: 같은 컴퓨터에서 여러 서비스를 구분하기 위한 번호
> IP 주소가 "집 주소"라면, 포트는 "집 안의 방 번호"

**주요 포트:**

| 포트 | 용도 | 설명 |
|---|---|---|
| **80** | HTTP (웹) | 기본 웹 포트, 생략 가능 |
| **443** | HTTPS (암호화 웹) | 보안 웹, 생략 가능 |
| **3000** | Node.js 개발 서버 | 개발 환경에서 주로 사용 |
| **5432** | PostgreSQL 데이터베이스 | 내부 통신 |
| **6379** | Redis 캐시 | 내부 통신 |

### 80번 포트 vs 3000번 포트

**포트 번호 표기:**

```javascript
// 80번은 생략 가능
http://example.com         // 자동으로 80번
http://example.com:80      // 명시적 표기

// 3000번은 반드시 명시
http://localhost:3000      // 3000번 필수
```

### Nginx와 포트의 관계

**Nginx 없을 때 (위험):**

클라이언트가 Express 서버에 직접 접근하면, 포트 3000이 외부에 노출된다.
포트가 알려지면 해커가 직접 3000번으로 공격할 수 있어 위험하다.

**Nginx 사용 (안전):**

클라이언트는 포트 80(생략 가능)에만 접속하고, Express의 포트 3000은 내부에서만 접근한다.
해커는 내부 포트를 알 수 없으므로 훨씬 안전하다.

### Nginx 설정에서의 포트

```nginx
http {
    upstream backend {
        server 4.23.20.8:3000;  # 내부 Express 서버 (숨겨짐)
    }
    
    events {}
    
    server {
        listen 80;  # 외부에 노출되는 포트
        root /usr/share/nginx/html;
        
        location / {
            index index.html;  # 정적 파일
        }
        
        location /dweb {
            proxy_pass http://backend;  # 내부적으로 3000으로 전달
        }
    }
}
```

**요청 흐름:**

- ① 클라이언트가 http://4.23.20.8에 접속 (포트 80은 생략됨)
- ② Nginx가 포트 80에서 요청 수신
- ③ 요청 경로 분석 (/ → 정적 파일, /dweb → 백엔드)
- ④ /dweb 요청인 경우 Nginx가 내부적으로 Express (포트 3000)에 전달
- ⑤ Express에서 처리 후 응답
- ⑥ Nginx를 통해 클라이언트에게 응답

### HTTPS 443번 포트

```nginx
server {
    listen 443 ssl;  # HTTPS 포트
    
    ssl_certificate /path/to/cert.pem;
    ssl_certificate_key /path/to/key.pem;
    
    location /dweb {
        proxy_pass http://backend;  # 내부적으로 3000으로 전달
    }
}
```

- 클라이언트: `https://4.23.20.8` (포트 443, 생략)
- Nginx: 포트 443에서 HTTPS 수신
- 내부: Express 포트 3000으로 전달 (HTTP 가능)

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

### Windows

**1. Nginx 다운로드**

- 공식 사이트: http://nginx.org/en/download.html
- "Stable version" Windows zip 다운로드

**2. 설치**

```bash
# 다운로드한 zip 파일 해제
# 예: C:\nginx\

# 폴더 구조
C:\nginx\
├── conf\
│   └── nginx.conf  (설정 파일)
├── html\           (정적 파일 디렉토리)
├── logs\
└── nginx.exe       (실행 파일)
```

**3. Nginx 실행**

```bash
# nginx.exe 파일이 있는 디렉토리로 이동
cd C:\nginx\

# Nginx 시작
nginx.exe

# 또는 cmd에서
start nginx.exe
```

**4. 브라우저에서 확인**

```
http://localhost:80
또는
http://localhost
```

**5. 기본 파일 위치**

```
정적 파일: C:\nginx\html\
설정 파일: C:\nginx\conf\nginx.conf
```

**6. 설정 변경 후 리로드**

```bash
# nginx.exe가 있는 디렉토리에서
nginx.exe -s reload

# 또는 명령 프롬프트 (cmd)
cd C:\nginx
nginx -s reload
```

**7. Nginx 종료**

```bash
# nginx.exe가 있는 디렉토리에서
nginx.exe -s stop

# 또는
nginx -s quit
```

**8. 로그 파일 위치**

```
에러 로그: C:\nginx\logs\error.log
접근 로그: C:\nginx\logs\access.log
```

**9. 방화벽 설정 (필요 시)**

- Windows Defender 방화벽에서 nginx.exe 허용
- 또는 포트 80, 443 개방

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

### 설정 파일의 계층 구조

**기본 구조:**

```
http block (전체 웹 서버 설정)
    ├── upstream block (백엔드 서버 정의)
    └── server block (가상 호스트 설정)
        └── location block (경로별 처리)
```

### http block

> **http block**: Nginx의 **웹 서버 전체 설정**을 담는 최상위 블록
> 모든 웹 관련 설정이 이 블록 안에 포함됨

**역할:**
- MIME 타입 정의 (파일 타입 인식)
- 압축 설정
- 캐시 설정
- 전체 서버의 기본 설정

```nginx
http {
    include mime.types;  # 파일 타입 정의
    
    # 여기에 upstream, server 블록이 들어감
    
    upstream backend {
        server 4.23.20.8:3000;
    }
    
    server {
        # 서버 설정
    }
}
```

### server block

> **server block**: **가상 호스트(Virtual Host) 설정**
> 하나의 Nginx에서 여러 도메인/IP를 관리할 수 있음

**역할:**
- 포트 설정 (listen)
- 루트 디렉토리 (root)
- 기본 파일 (index)
- location 블록으로 경로 분기

```nginx
server {
    listen 80;                  # 포트
    server_name example.com;    # 도메인
    root /usr/share/nginx/html; # 공개 디렉토리
    
    location / {
        # 경로별 처리
    }
}
```

**여러 도메인을 한 Nginx에서 관리:**

```nginx
http {
    # 첫 번째 도메인
    server {
        listen 80;
        server_name example.com;
        root /var/www/example;
    }
    
    # 두 번째 도메인
    server {
        listen 80;
        server_name blog.example.com;
        root /var/www/blog;
    }
}
```

### location block

> **location block**: **요청 경로에 따른 처리 방식 정의**
> 같은 서버 내에서 경로마다 다르게 처리

**역할:**
- 정적 파일 제공
- 백엔드 프록시
- 리다이렉트
- 접근 제어

```nginx
server {
    location / {
        # / 경로 → 정적 파일
        index index.html;
    }
    
    location /api {
        # /api 경로 → 백엔드 프록시
        proxy_pass http://backend;
    }
    
    location /admin {
        # /admin 경로 → 특정 IP만 허용
        allow 192.168.1.0/24;
        deny all;
    }
}
```

**경로 매칭 종류:**

| 표기 | 의미 | 예시 |
|---|---|---|
| `location /` | 모든 경로 (기본값) | /index.html, /api/users |
| `location /api` | /api로 시작하는 경로 | /api, /api/users |
| `location = /` | 정확히 / 만 | / (다른 경로는 X) |
| `location ~ \.php$` | 정규표현식 (PHP 파일) | index.php, test.php |
| `location ^~ /files` | /files로 시작 (우선순위 높음) | /files/image.jpg |

### location의 순서와 우선순위

> **Nginx는 location을 순서대로 평가하지 않음**
> **대신 정해진 우선순위에 따라 가장 먼저 매칭되는 것을 선택**

**우선순위 (높은 순서부터):**

1. **정확한 매칭** (`location = /path`)
   - 정확히 일치하는 경로만 매칭
   - 우선순위가 가장 높음

2. **우선순위 높은 정규식** (`location ^~ /path`)
   - /path로 시작하는 경로 매칭
   - 정규식보다 먼저 평가

3. **정규식 매칭** (`location ~ 또는 location ~*`)
   - 정규표현식으로 매칭
   - 순서대로 평가 (처음 매칭되는 것 선택)

4. **접두사 매칭** (`location /path`)
   - /path로 시작하는 경로 매칭
   - 가장 긴 접두사 선택

5. **기본값** (`location /`)
   - 어떤 것도 매칭되지 않으면 마지막 선택

**우선순위 예시:**

```nginx
server {
    # 우선순위 1: 정확한 매칭
    location = / {
        return 200 "정확히 / 만 처리";
    }
    
    # 우선순위 2: 우선순위 높은 정규식
    location ^~ /api {
        proxy_pass http://backend;
    }
    
    # 우선순위 3: 정규식
    location ~ \.json$ {
        return 200 "JSON 파일 처리";
    }
    
    # 우선순위 4: 접두사 매칭
    location /static {
        root /usr/share/nginx/html;
    }
    
    # 우선순위 5: 기본값
    location / {
        index index.html;
    }
}
```

**요청별 매칭 결과:**

```
요청: GET /
→ location = / 매칭 (우선순위 1)
→ "정확히 / 만 처리" 반환

요청: GET /api/users
→ location ^~ /api 매칭 (우선순위 2)
→ 백엔드 프록시

요청: GET /data.json
→ location ~ \.json$ 매칭 (우선순위 3)
→ "JSON 파일 처리" 반환

요청: GET /static/style.css
→ location /static 매칭 (우선순위 4)
→ 정적 파일 제공

요청: GET /about
→ 어떤 것도 매칭 안 됨
→ location / 매칭 (우선순위 5)
→ index.html 제공
```

### location 실제 사용 예시

**예시 1: 일반적인 웹 사이트**

```nginx
server {
    listen 80;
    root /var/www/html;
    
    # 정적 파일 (CSS, JS, 이미지)
    location ~* \.(css|js|jpg|jpeg|png|gif)$ {
        expires 30d;  # 30일 캐시
    }
    
    # 정적 폴더
    location /assets {
        root /var/www;
    }
    
    # API 요청
    location /api {
        proxy_pass http://backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
    
    # 기본값
    location / {
        index index.html index.htm;
        try_files $uri $uri/ /index.html;  # SPA용
    }
}
```

**예시 2: 마이크로서비스 아키텍처**

```nginx
server {
    listen 80;
    
    # 사용자 서비스
    location /users {
        proxy_pass http://user-service:3001;
    }
    
    # 상품 서비스
    location /products {
        proxy_pass http://product-service:3002;
    }
    
    # 주문 서비스
    location /orders {
        proxy_pass http://order-service:3003;
    }
    
    # 관리자 (특정 IP만)
    location /admin {
        allow 192.168.1.0/24;
        deny all;
        proxy_pass http://admin-service:3000;
    }
    
    # 정적 파일
    location / {
        root /usr/share/nginx/html;
        index index.html;
    }
}
```

**예시 3: React SPA + Express 백엔드**

```nginx
server {
    listen 80;
    root /usr/share/nginx/html;  # React 빌드 폴더
    
    # API 요청 (Express)
    location /api {
        proxy_pass http://express-backend:3000;
        proxy_set_header X-Real-IP $remote_addr;
    }
    
    # React 정적 파일 (캐시)
    location ~* \.(js|css|jpg|png|gif)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }
    
    # 모든 요청을 index.html로 (SPA 라우팅)
    location / {
        try_files $uri $uri/ /index.html;
    }
}
```

**예시 4: 여러 도메인 처리**

```nginx
# example.com
server {
    server_name example.com;
    root /var/www/example;
    location / {
        index index.html;
    }
}

# blog.example.com
server {
    server_name blog.example.com;
    root /var/www/blog;
    location / {
        index index.html;
    }
    location /api {
        proxy_pass http://blog-backend:3001;
    }
}

# api.example.com
server {
    server_name api.example.com;
    location / {
        proxy_pass http://api-backend:3000;
    }
}
```

### 계층 구조 동작 예시

```nginx
http {
    include mime.types;
    
    upstream backend {
        server 4.23.20.8:3000;
    }
    
    server {
        listen 80;
        root /usr/share/nginx/html;
        
        # 요청 1: GET /index.html
        location / {
            index index.html;
            # → /usr/share/nginx/html/index.html 제공
        }
        
        # 요청 2: GET /api/users
        location /api {
            proxy_pass http://backend;
            # → 4.23.20.8:3000/api/users로 프록시
        }
        
        # 요청 3: GET /admin
        location /admin {
            allow 192.168.1.0/24;
            deny all;
            proxy_pass http://backend;
            # → IP 확인 후 프록시 또는 거부
        }
    }
}
```

---

## Nginx 기본 설정

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

### index 파일 설정

> **index**: 경로만 요청했을 때 **자동으로 제공할 파일**
> 보통 index.html이 기본값이지만, 다른 파일도 지정 가능

**단일 index 파일:**

```nginx
server {
    root /usr/share/nginx/html;
    
    location / {
        index index.html;  # / 요청 시 index.html 제공
    }
}
```

**요청 흐름:**

- 클라이언트: GET / (또는 http://localhost:80)
- Nginx: root (/usr/share/nginx/html) + index (index.html)
- 결과: /usr/share/nginx/html/index.html 제공

**여러 index 파일 지정 (우선순위):**

```nginx
location / {
    index index.html index.htm default.html;  # 순서대로 찾음
}
```

- 첫 번째: `index.html` 찾기
- 두 번째: `index.html` 없으면 `index.htm` 찾기
- 세 번째: 둘 다 없으면 `default.html` 찾기

**index 파일이 없을 때:**

```nginx
server {
    autoindex on;  # 폴더 내 파일 목록 표시
}
```

설정이 있으면 디렉토리의 파일 목록을 HTML로 표시.

**경로별로 다른 index 파일:**

```nginx
server {
    root /usr/share/nginx/html;
    
    location / {
        index index.html;  # / → index.html
    }
    
    location /blog {
        index blog.html;   # /blog → blog.html
    }
    
    location /docs {
        index README.md;   # /docs → README.md
    }
}
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
| **경로 합치기** | location 경로 + root | alias 경로만 사용 |
| **예시** | `/incheon` + `/root` → `/incheon/` | `/songdo` → `/alias/` |

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
