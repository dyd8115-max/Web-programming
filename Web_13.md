# GitHub 강의 노트

---

## 1. GitHub란?

> Git 저장소를 인터넷에 호스팅해주는 **원격 저장소 플랫폼**

- Git은 로컬에서 버전을 관리하는 도구, GitHub는 그것을 **클라우드에 올려 공유/협업**하는 공간
- 코드 백업, 팀 협업, 오픈소스 기여, 포트폴리오 등 다양한 용도로 사용

```
로컬 (내 PC)                원격 (GitHub)
  Repository   ──push──→   Remote Repository
               ←──pull──
```

---

## 2. 원격 저장소 연결 & 기본 명령어

### 원격 저장소 연결

```bash
git remote add origin <URL>       # 원격 저장소 연결 (origin = 별칭)
git remote -v                     # 연결된 원격 저장소 확인
git remote remove origin          # 연격 저장소 연결 해제
```

### Push / Pull / Fetch

```bash
git push origin main              # 로컬 → 원격 업로드
git push -u origin main           # 업스트림 설정 (이후 git push 만으로 가능)
git push --force                  # 강제 push (히스토리 덮어씀, 주의!)

git pull origin main              # 원격 → 로컬 (fetch + merge 합친 것)
git fetch origin                  # 원격 변경사항 가져오기만 (병합 X)
```

- `pull` = `fetch` + `merge` — 가져오면서 바로 병합
- `fetch` = 가져오기만 — 확인 후 직접 병합 가능 (더 안전)

### 처음 로컬 → GitHub 올리는 흐름

```bash
git init
git add .
git commit -m "first commit"
git remote add origin https://github.com/username/repo.git
git push -u origin main
```

---

## 3. 토큰 (Personal Access Token)

> GitHub에서 HTTPS 방식으로 인증할 때 **비밀번호 대신 사용하는 인증 키**  
> 2021년 8월부터 GitHub는 비밀번호 인증을 폐지 → **토큰 필수**

### 토큰 생성 경로

```
GitHub → Settings → Developer settings
  → Personal access tokens → Tokens (classic) → Generate new token
```

### 토큰 Scope (접근 범위) — 주요 항목

| Scope | 설명 |
|-------|------|
| `repo` | **private/public 저장소 전체 접근** (읽기/쓰기/삭제) |
| `repo:status` | 커밋 상태 접근 |
| `public_repo` | public 저장소만 접근 |
| `repo:invite` | 저장소 초대 관련 접근 |
| `workflow` | GitHub Actions 워크플로우 수정 권한 |
| `write:packages` | 패키지 업로드 권한 |
| `read:org` | 조직 정보 읽기 |
| `admin:repo_hook` | 저장소 훅 관리 |
| `delete_repo` | 저장소 삭제 권한 |
| `user` | 사용자 프로필 정보 접근 |

> 💡 일반적인 코드 push/pull 용도라면 **`repo` 체크만으로 충분**  
> 필요한 최소 권한만 부여하는 것이 보안상 좋음

### 토큰 사용

```bash
# push 시 username/password 입력 창에서
Username: 깃허브아이디
Password: (발급받은 토큰 붙여넣기)
```

```bash
# 토큰을 URL에 포함시키는 방법 (자동화 스크립트 등)
git remote set-url origin https://<TOKEN>@github.com/username/repo.git
```

> ⚠️ 토큰은 생성 시 한 번만 보여줌 — 반드시 복사해서 안전한 곳에 보관  
> 유출 시 즉시 GitHub에서 폐기(Revoke) 후 재발급

### Fine-grained Token (세분화 토큰)

> Classic 토큰보다 더 세밀하게 권한을 제어할 수 있는 신형 토큰

- 특정 저장소에만 접근 범위 제한 가능
- Contents / Pull requests / Issues 등 기능별로 Read/Write 따로 설정
- 만료 기간 설정 필수 (최대 1년)

---

## 4. Clone vs Fork

| | Clone | Fork |
|---|---|---|
| 개념 | 원격 저장소를 **내 로컬**로 복사 | 남의 저장소를 **내 GitHub 계정**으로 복사 |
| 용도 | 내 저장소 또는 협업 프로젝트 작업 | 오픈소스 기여, 남의 프로젝트 수정 |
| 원본과 관계 | 직접 연결됨 | 독립적 (PR로 기여 가능) |

```bash
git clone https://github.com/username/repo.git   # 로컬로 복제
git clone https://github.com/username/repo.git my-folder  # 폴더명 지정
```

---

## 5. Branch 전략

> 협업 시 브랜치를 어떻게 나눠 쓸지 정하는 규칙  
> 팀마다 다르지만 아래가 일반적인 구조

```
main          ── 배포용 (항상 안정적인 상태 유지, 직접 push 금지)
develop       ── 개발 통합 브랜치 (기능 브랜치들이 여기로 합쳐짐)
feature/login ── 기능 개발 브랜치 (기능 하나 = 브랜치 하나)
bugfix/navbar ── 버그 수정 브랜치
hotfix/crash  ── 긴급 수정 (main에서 바로 분기)
```

### Git Flow 전략 (대표적인 협업 브랜치 전략)

```
main ──────────────────────────────────────── (릴리즈)
  └── develop ──────────────────────────────── (통합)
        ├── feature/login ──── (기능 개발 후 develop으로 PR)
        ├── feature/signup ─── (기능 개발 후 develop으로 PR)
        └── bugfix/header ──── (버그 수정 후 develop으로 PR)
```

### 브랜치 네이밍 컨벤션

| 종류 | 예시 |
|------|------|
| 기능 개발 | `feature/login`, `feature/user-profile` |
| 버그 수정 | `bugfix/login-error`, `fix/navbar-crash` |
| 긴급 수정 | `hotfix/payment-bug` |
| 릴리즈 | `release/v1.0.0` |

---

## 6. Pull Request (PR)

> 내 브랜치의 변경사항을 다른 브랜치에 **병합 요청**하는 GitHub 기능  
> 단순 병합이 아니라 **코드 리뷰 + 토론 + 승인** 과정을 포함

### PR 전체 흐름

```
① 브랜치 생성
   git switch -c feature/login

② 작업 후 커밋
   git add . && git commit -m "feat: 로그인 기능 추가"

③ 원격에 push
   git push origin feature/login

④ GitHub에서 PR 생성
   - base: develop (병합 대상)
   - compare: feature/login (내 브랜치)
   - 제목 / 설명 / 리뷰어 / 라벨 설정

⑤ 팀원 코드 리뷰
   - 댓글, 변경 요청, 승인(Approve)

⑥ Merge
   - 승인 후 병합 방식 선택 → Merge

⑦ 브랜치 삭제
   git branch -d feature/login
   git push origin --delete feature/login
```

### PR 생성 시 작성 요소

| 항목 | 설명 |
|------|------|
| **Title** | 변경사항을 한 줄로 요약 |
| **Description** | 무엇을, 왜 변경했는지 설명 |
| **Reviewers** | 코드 리뷰 요청할 팀원 지정 |
| **Assignees** | 이 PR의 담당자 |
| **Labels** | `feature`, `bug`, `hotfix` 등 분류 |
| **Linked Issues** | 연관된 이슈 번호 (`Closes #12`) |

### PR Description 예시 템플릿

```markdown
## 변경 사항
- 로그인 API 연동
- JWT 토큰 localStorage 저장 로직 추가

## 테스트 방법
1. `npm run dev` 실행
2. /login 페이지 접속
3. 계정 입력 후 로그인 확인

## 관련 이슈
Closes #12
```

### PR 병합 방식 3가지

| 방식 | 히스토리 | 설명 |
|------|----------|------|
| **Merge commit** | 분기 흔적 유지 | 병합 커밋 생성, 모든 커밋 보존 |
| **Squash and merge** | 커밋 1개로 압축 | 여러 커밋을 하나로 합쳐서 병합 — 지저분한 커밋 정리에 유용 |
| **Rebase and merge** | 일직선 | 커밋을 그대로 이어 붙임, 병합 커밋 없음 |

### 코드 리뷰 에티켓

- **리뷰어**: 비판이 아닌 제안 형태로 작성 (`이렇게 하면 어떨까요?`)
- **작성자**: 리뷰 댓글에 답변하고 반영 여부 명시
- `Approve` — 병합 승인
- `Request changes` — 수정 요청 (수정 전까지 병합 불가)
- `Comment` — 단순 의견 (승인/거절 아님)

---

## 6. 원격 브랜치 관리

```bash
git push origin <브랜치명>          # 로컬 브랜치를 원격에 push
git push origin --delete <브랜치명> # 원격 브랜치 삭제

git branch -r                       # 원격 브랜치 목록 확인
git branch -a                       # 로컬 + 원격 브랜치 모두 확인

git switch -c <브랜치명> origin/<브랜치명>  # 원격 브랜치를 로컬로 가져와서 전환
```

---

## 7. 팀 협업 워크플로우

### 협업 시작 세팅

```bash
# 팀원이 저장소 clone
git clone https://github.com/팀/repo.git
cd repo

# 브랜치 전략 확인 후 develop 기준으로 작업 브랜치 생성
git switch develop
git pull origin develop          # 항상 최신 상태로 맞추고 시작
git switch -c feature/내기능
```

### 매일 작업 루틴

```bash
# 작업 시작 전 — 항상 최신 코드 받기
git switch develop
git pull origin develop
git switch feature/내기능
git merge develop                # develop 변경사항을 내 브랜치에 반영

# 작업 후 커밋
git add .
git commit -m "feat: 기능 설명"

# 원격에 push
git push origin feature/내기능

# PR 생성 → 리뷰 → 병합
```

### 커밋 메시지 컨벤션

> 팀에서 커밋 메시지 형식을 통일하면 히스토리 추적이 쉬워짐

```
<타입>: <설명>

예시:
feat: 로그인 기능 추가
fix: 회원가입 유효성 검사 오류 수정
docs: README 업데이트
style: 코드 포맷 정리
refactor: 인증 로직 리팩토링
test: 유닛 테스트 추가
chore: 패키지 의존성 업데이트
```

| 타입 | 설명 |
|------|------|
| `feat` | 새로운 기능 추가 |
| `fix` | 버그 수정 |
| `docs` | 문서 변경 |
| `style` | 코드 포맷, 세미콜론 등 (기능 변경 없음) |
| `refactor` | 코드 리팩토링 |
| `test` | 테스트 코드 추가/수정 |
| `chore` | 빌드, 패키지 관련 작업 |

### 협업 충돌 해결 흐름

```bash
# push 전에 항상 pull 먼저
git pull origin develop

# 충돌 발생 시
# 1. 충돌 파일 열기 → <<<<<<< / ======= / >>>>>>> 마커 확인
# 2. 남길 내용 선택 후 마커 제거
# 3. 저장 후
git add <충돌파일>
git commit
git push origin feature/내기능
```

### 실수했을 때 대처법

```bash
# 커밋 메시지 수정 (push 전)
git commit --amend -m "새 메시지"

# 마지막 커밋 취소 (파일은 유지)
git reset --soft HEAD~1

# 원격에 잘못 올린 파일 제거 (이미 push한 경우)
git rm --cached <파일>
git commit -m "chore: 민감 파일 제거"
git push origin <브랜치>

# 원격 브랜치에 강제 push (혼자 쓰는 브랜치에서만!)
git push --force-with-lease origin feature/내기능
```

---

## 8. GitHub 팀 프로젝트 설정

### Repository 권한 관리

| 역할 | 권한 |
|------|------|
| **Owner** | 저장소 모든 권한 (설정, 삭제 포함) |
| **Admin** | 설정 변경, 팀원 관리 |
| **Maintainer** | PR 병합, 이슈 관리 |
| **Write** | 코드 push, PR 생성 |
| **Read** | 코드 열람만 가능 |

### Branch Protection Rules (브랜치 보호 규칙)

> main / develop 브랜치를 실수로 덮어쓰지 않도록 보호

```
GitHub → Settings → Branches → Add rule

주요 옵션:
  ✅ Require a pull request before merging  (직접 push 금지, PR 필수)
  ✅ Require approvals (최소 승인 인원 설정)
  ✅ Require status checks to pass (CI 통과 필수)
  ✅ Restrict who can push (push 가능한 사람 제한)
```

### Issue 템플릿

> 팀원이 이슈를 일정한 형식으로 작성하도록 유도

```markdown
<!-- .github/ISSUE_TEMPLATE/feature_request.md -->
## 기능 설명
어떤 기능인지 설명

## 구현 방법
어떻게 구현할 예정인지

## 완료 조건
- [ ] 체크리스트 1
- [ ] 체크리스트 2
```

### 프로젝트 보드 (GitHub Projects)

- Kanban 보드 형식으로 이슈/PR 관리
- `Todo → In Progress → Done` 컬럼으로 진행 상황 시각화
- 이슈와 연동해서 자동으로 상태 이동 설정 가능

---

## 8. 고급 Push / Pull 명령어

### Push 심화

```bash
git push origin main                   # 기본 push
git push -u origin main                # 업스트림 설정 (이후 git push 만으로 가능)
git push origin feature/login          # 특정 브랜치 push
git push origin --all                  # 모든 브랜치 push
git push origin --tags                 # 태그 push
git push origin --delete <브랜치명>    # 원격 브랜치 삭제
git push --force                       # 강제 push (히스토리 덮어씀, 위험!)
git push --force-with-lease            # 안전한 강제 push (원격이 변경됐으면 거부)
```

> `--force` 대신 `--force-with-lease` 사용 권장 — 다른 사람 커밋을 덮어쓰는 사고 방지

### Pull 심화

```bash
git pull origin main                   # fetch + merge
git pull --rebase origin main          # fetch + rebase (히스토리 일직선 유지)
git pull --no-ff origin main           # 항상 병합 커밋 생성
git pull --ff-only origin main         # fast-forward 가능할 때만 pull (불가능하면 중단)
```

- `git pull --rebase` 는 불필요한 병합 커밋 없이 히스토리를 깔끔하게 유지할 때 사용

### Fetch 심화

```bash
git fetch origin                       # 원격 변경사항 가져오기 (병합 X)
git fetch origin main                  # 특정 브랜치만 fetch
git fetch --all                        # 모든 원격 저장소 fetch
git fetch --prune                      # 삭제된 원격 브랜치 로컬에서도 정리
```

- fetch 후 `git diff origin/main` 으로 변경사항 확인 가능
- 확인 후 `git merge origin/main` 으로 수동 병합

---

## 9. 태그 (Tag)

> 특정 커밋에 이름을 붙이는 기능 — 주로 **릴리즈 버전 관리**에 사용

```bash
git tag                                # 태그 목록 확인
git tag v1.0.0                         # 경량 태그 생성 (현재 커밋)
git tag -a v1.0.0 -m "첫 번째 릴리즈"  # 주석 태그 생성 (권장)
git tag -a v1.0.0 <커밋해시>           # 특정 커밋에 태그 생성
git tag -d v1.0.0                      # 태그 삭제

git push origin v1.0.0                 # 특정 태그 push
git push origin --tags                 # 모든 태그 push
git push origin --delete v1.0.0        # 원격 태그 삭제
```

| 종류 | 설명 |
|------|------|
| **경량 태그 (lightweight)** | 커밋에 이름만 붙임 |
| **주석 태그 (annotated)** | 태그 메시지, 작성자, 날짜 포함 — 릴리즈에 권장 |

---

## 10. Stash — 임시 저장

> 커밋하지 않은 변경사항을 **임시로 보관**해두는 기능  
> 브랜치를 급하게 전환해야 할 때 유용

```bash
git stash                              # 현재 변경사항 임시 저장
git stash save "작업 중인 기능"         # 메시지와 함께 저장
git stash list                         # 저장된 stash 목록
git stash pop                          # 가장 최근 stash 복원 + 목록에서 제거
git stash apply stash@{0}              # 특정 stash 복원 (목록 유지)
git stash drop stash@{0}               # 특정 stash 삭제
git stash clear                        # 모든 stash 삭제
```

### 사용 시나리오

```
feature 브랜치 작업 중
  → 갑자기 main 브랜치에서 긴급 버그 수정 필요
  → git stash         (작업 임시 저장)
  → git switch main   (브랜치 전환)
  → 버그 수정 + 커밋
  → git switch feature
  → git stash pop     (작업 복원)
```

---

## 11. Cherry-pick

> 다른 브랜치의 **특정 커밋만 골라서** 현재 브랜치에 적용

```bash
git cherry-pick <커밋해시>             # 특정 커밋 가져오기
git cherry-pick <해시1> <해시2>        # 여러 커밋 가져오기
git cherry-pick <시작해시>..<끝해시>   # 범위로 가져오기
git cherry-pick --no-commit <해시>     # 커밋 없이 변경사항만 적용
```

### 사용 시나리오

```
develop 브랜치에서 버그 수정 커밋이 생겼는데
main 에도 동일하게 적용해야 할 때

→ git switch main
→ git cherry-pick <버그수정 커밋해시>
```

---

## 12. 원격 저장소 관리

```bash
git remote add origin <URL>            # 원격 저장소 추가
git remote add upstream <URL>          # 원본 저장소 추가 (fork 시)
git remote -v                          # 원격 저장소 목록 확인
git remote rename origin new-name      # 원격 저장소 별칭 변경
git remote remove origin               # 원격 저장소 제거
git remote set-url origin <새 URL>     # 원격 저장소 URL 변경
```

### Fork 후 원본 저장소 동기화

```bash
git remote add upstream <원본 저장소 URL>
git fetch upstream
git merge upstream/main
git push origin main
```

---

## 13. 유용한 GitHub 기능

### Issues

- 버그 리포트, 기능 요청, 할 일 관리 등에 활용
- 브랜치명이나 커밋 메시지에 `#이슈번호` 를 넣으면 자동 연결
- `Closes #12` 를 PR에 작성하면 PR 병합 시 이슈 자동 닫힘

### README.md

- 저장소 첫 화면에 표시되는 프로젝트 설명 파일
- Markdown으로 작성 — 프로젝트 소개, 설치 방법, 사용법 등 기재

### GitHub Pages

- 정적 파일(HTML/CSS/JS)을 GitHub에서 무료로 호스팅
- `gh-pages` 브랜치 또는 `docs/` 폴더를 소스로 설정
- 주소: `https://username.github.io/repo-name`

---
