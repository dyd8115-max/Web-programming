# Git 강의 노트

---

## 1. Git이란?

> **버전 컨트롤 시스템 (VCS, Version Control System)**  
> Snapshot(커밋) 기술을 통해 모든 소스코드 변경 이력을 관리

- 중요한 변경사항이 있을 때마다 코드를 **커밋(commit)** 으로 저장
- 커밋된 시점으로 언제든지 **되돌아갈 수 있음**
- 여러 명이 동시에 작업해도 코드 충돌을 관리할 수 있음

---

## 2. Git 처리 과정 — 3단계 구조

```
Working Directory  →  Staging Area  →  Repository
   (작업 공간)          (대기 공간)        (저장소)

  파일 수정/생성       git add           git commit
                   (스테이징)          (스냅샷 저장)
```

| 단계 | 설명 |
|------|------|
| **Working Directory** | 실제 파일을 수정하는 공간. 아직 Git이 추적하지 않음 |
| **Staging Area** | 커밋할 변경사항을 선별해서 올려두는 대기 공간 |
| **Repository** | 커밋된 스냅샷들이 영구적으로 저장되는 공간 (`.git` 폴더) |

---

## 3. Markdown 기초 문법

> Git 프로젝트의 README.md 등에서 사용하는 마크업 언어

```markdown
# 제목 1
## 제목 2
### 제목 3

**굵게**  *기울임*  ~~취소선~~

- 목록 항목
- 목록 항목
  - 중첩 목록

1. 순서 있는 목록
2. 두 번째

`인라인 코드`

```코드 블록```

[링크 텍스트](URL)
![이미지 설명](이미지URL)

> 인용문

---  (구분선)

| 열1 | 열2 |
|-----|-----|
| 값1 | 값2 |
```

---

## 4. Git 기초 명령어

### 초기 설정

```bash
git config --global user.name "이름"
git config --global user.email "이메일"
```

### 저장소 초기화 / 복제

```bash
git init          # 현재 폴더를 Git 저장소로 초기화
git clone <URL>   # 원격 저장소 복제
```

### 기본 워크플로우

```bash
git status                  # 현재 상태 확인
git add <파일>              # 특정 파일 스테이징
git add .                   # 모든 변경사항 스테이징
git commit -m "메시지"      # 스냅샷 저장
git log                     # 커밋 이력 확인
git log --oneline           # 한 줄로 간략히 보기
```

### 되돌리기

```bash
git restore <파일>          # Working Directory 변경사항 취소
git restore --staged <파일> # Staging Area에서 내리기
```

---

## 5. .env 와 .gitignore

### .env — 환경변수 파일

> API Key, DB 비밀번호 등 **민감한 정보**를 코드에서 분리해 관리하는 파일

```bash
# .env
VITE_API_KEY=sk-abcd1234
DB_PASSWORD=secret
```

- 코드에서 `process.env.VITE_API_KEY` 로 접근
- **절대 Git에 올리면 안 됨** → `.gitignore`에 반드시 추가

### .gitignore — Git 추적 제외 목록

> Git이 추적하지 않을 파일/폴더를 지정하는 파일

```bash
# .gitignore 예시
.env                # 환경변수 파일
node_modules/       # 패키지 폴더 (용량 큼)
dist/               # 빌드 결과물
.DS_Store           # macOS 시스템 파일
*.log               # 로그 파일
```

- 프로젝트 루트에 `.gitignore` 파일 생성
- 이미 추적 중인 파일은 `git rm --cached <파일>` 로 추적 해제 후 적용

---

## 6. Reset — 커밋 되돌리기

> 커밋 히스토리를 특정 시점으로 되돌리는 명령어

```bash
git reset <옵션> <커밋해시>
```

| 옵션 | Working Directory | Staging Area | 커밋 이력 |
|------|-------------------|--------------|-----------|
| `--soft` | 유지 | 유지 | 되돌림 |
| `--mixed` (기본) | 유지 | 초기화 | 되돌림 |
| `--hard` | 초기화 | 초기화 | 되돌림 |

```bash
git reset --soft HEAD~1   # 마지막 커밋만 취소, 파일은 그대로
git reset --mixed HEAD~1  # 마지막 커밋 취소 + 스테이징 해제
git reset --hard HEAD~1   # 마지막 커밋 취소 + 파일도 되돌림 (주의!)
```

- `HEAD~1` = 현재 커밋에서 1단계 이전
- `--hard` 는 작업 내용이 날아가므로 **신중하게 사용**

---

## 7. Branch — 분기

> 메인 코드에서 독립적인 작업 공간을 분리하는 개념  
> 기능 개발, 버그 수정 등을 **메인 브랜치에 영향 없이** 진행 가능

```
main   ──●──●──────────────●── (merge)
              \            /
feature        ●──●──●────
```

### 브랜치 명령어

```bash
git branch                    # 브랜치 목록 확인
git branch <브랜치명>         # 브랜치 생성
git switch <브랜치명>         # 브랜치 이동
git switch -c <브랜치명>      # 생성 + 이동 동시에
git branch -d <브랜치명>      # 브랜치 삭제 (병합 후)
git branch -D <브랜치명>      # 브랜치 강제 삭제
```

### Merge — 브랜치 합치기

```bash
git switch main
git merge <브랜치명>
```

| 종류 | 설명 |
|------|------|
| **Fast-forward** | main에 추가 커밋이 없을 때 — 포인터만 이동 |
| **3-way merge** | 양쪽에 커밋이 있을 때 — 새 병합 커밋 생성 |

---

## 8. Conflict — 충돌

> 같은 파일의 같은 줄을 서로 다른 브랜치에서 수정했을 때 발생

```
<<<<<<< HEAD
현재 브랜치의 내용
=======
병합하려는 브랜치의 내용
>>>>>>> feature
```

### 해결 절차

```
1. 충돌 파일 열기
2. <<<<<<< / ======= / >>>>>>> 마커 확인
3. 남길 내용만 선택하고 마커 제거
4. git add <파일>
5. git commit
```

---

## 9. Rebase

> 브랜치의 **base(시작점)를 재설정**해 커밋 히스토리를 일직선으로 만드는 명령어

```
# Merge
main   ──●──●──────────●  (merge commit)
              \        /
feature        ●──●───

# Rebase
main   ──●──●──●──●──●   (일직선 히스토리)
```

```bash
git switch feature
git rebase main       # feature 브랜치를 main 최신 커밋 위로 재배치
```

| | Merge | Rebase |
|---|---|---|
| 히스토리 | 분기 흔적 남음 | 깔끔한 일직선 |
| 안전성 | 안전 | 공유된 브랜치에선 주의 |
| 사용 시점 | 팀 협업, 기록 보존 | 로컬 정리, PR 전 커밋 정돈 |

> ⚠️ 이미 원격에 push된 브랜치를 rebase하면 다른 사람의 히스토리와 충돌할 수 있음

---

## 10. Git Hook

> 특정 Git 이벤트가 발생할 때 **자동으로 스크립트를 실행**하는 기능  
> `.git/hooks/` 폴더에 스크립트 파일로 관리

### 주요 Hook 종류

| Hook | 실행 시점 | 활용 예시 |
|------|-----------|-----------|
| `pre-commit` | `git commit` 실행 직전 | 코드 린트, 테스트 자동 실행 |
| `commit-msg` | 커밋 메시지 작성 후 | 커밋 메시지 형식 검증 |
| `pre-push` | `git push` 실행 직전 | 테스트 통과 여부 확인 |
| `post-merge` | 병합 완료 후 | 패키지 자동 설치 등 |

```bash
# .git/hooks/pre-commit 예시 (실행 권한 필요)
#!/bin/sh
npm run lint   # 린트 통과 못하면 커밋 차단
```

```bash
chmod +x .git/hooks/pre-commit   # 실행 권한 부여
```

- Hook 파일은 `.git/` 안에 있어 **기본적으로 Git에 추적되지 않음**
- 팀 공유가 필요하면 `husky` 같은 라이브러리 활용

---

