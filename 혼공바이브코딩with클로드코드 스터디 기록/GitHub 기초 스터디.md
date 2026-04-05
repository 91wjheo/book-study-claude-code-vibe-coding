# GitHub 기초 완전 정복 📚

> 로컬에서 작업하고 원격 저장소(GitHub)와 연동하는 전체 흐름을 정리한 복습 노트

---

## 목차

1. [핵심 개념 — 3가지 영역](#1-핵심-개념--3가지-영역)
2. [전체 흐름 한눈에 보기](#2-전체-흐름-한눈에-보기)
3. [기본 명령어 흐름](#3-기본-명령어-흐름)
4. [PR / MR 이란?](#4-pr--mr-이란)
5. [브랜치(Branch) 개념](#5-브랜치branch-개념)
6. [브랜치 명령어](#6-브랜치-명령어)
7. [전체 명령어 요약표](#7-전체-명령어-요약표)

---

## 1. 핵심 개념 — 3가지 영역

Git은 코드를 3개의 공간에서 관리한다.

| 영역 | 이름 | 설명 |
|---|---|---|
| 내 컴퓨터 | **작업 디렉토리** (Working Directory) | 실제 파일을 수정하는 공간 |
| 내 컴퓨터 | **스테이징 영역** (Staging Area / Index) | 커밋할 파일을 준비해두는 대기실 |
| 내 컴퓨터 | **로컬 저장소** (Local Repository) | 커밋이 쌓이는 내 로컬 Git 저장소 |
| 인터넷 | **원격 저장소** (Remote Repository) | GitHub에 올라간 저장소 |

---

## 2. 전체 흐름 한눈에 보기

```
[GitHub 원격 저장소]
       ↑  git push          git pull / git clone  ↓
[로컬 저장소] ← git commit ← [스테이징 영역] ← git add ← [작업 디렉토리]
```

```
내 컴퓨터 (로컬 환경)
┌─────────────────────────────────────────────────────────┐
│                                                         │
│  [작업 디렉토리]  →(git add)→  [스테이징]  →(git commit)→  [로컬 저장소]  │
│                                                         │
└─────────────────────────────────────────────────────────┘
         ↑ git pull / git clone                ↓ git push
                    [GitHub 원격 저장소]
```

---

## 3. 기본 명령어 흐름

### Step 1 — 로컬 폴더를 Git 저장소로 초기화

```bash
cd my-project    # 작업 폴더로 이동
git init         # 이 폴더를 Git 저장소로 초기화 (딱 한 번만)
```

> `git init`은 `.git` 숨김 폴더를 만들어 모든 버전 정보를 관리한다.

---

### Step 2 — 파일을 스테이징 영역에 추가

```bash
git add index.html    # 특정 파일만 추가
git add .             # 현재 폴더의 모든 변경 파일 추가
```

> 아직 저장된 게 아니다. "다음 커밋에 이 파일 포함할게요" 라고 예약하는 단계.

---

### Step 3 — 로컬 저장소에 커밋 (스냅샷 저장)

```bash
git commit -m "첫 번째 커밋: 기본 파일 추가"
```

> `-m` 뒤의 메시지는 무엇을 변경했는지 기록하는 메모. 나중에 히스토리 확인 시 중요.

---

### Step 4 — GitHub 원격 저장소 연결 & 업로드 (push)

```bash
# GitHub 저장소 주소를 "origin"이라는 이름으로 등록
git remote add origin https://github.com/내아이디/my-project.git

# 로컬의 main 브랜치를 GitHub에 업로드 (처음 한 번)
git push -u origin main
```

> `-u origin main`은 앞으로 push/pull의 기본 대상을 설정하는 것. 처음 1회만 붙이면 된다.

---

### Step 5 — GitHub에서 로컬로 받아오기 (pull)

```bash
git pull origin main    # GitHub 최신 내용을 로컬에 반영
```

> `git pull` = `git fetch`(가져오기) + `git merge`(합치기)를 한 번에 처리.

---

### Step 6 — 수정 후 다시 push (반복 사이클)

```bash
git add .
git commit -m "로그인 기능 추가"
git push
```

> 이 세 줄이 매일 반복하는 기본 사이클: **수정 → add → commit → push**

---

## 4. PR / MR 이란?

### PR과 MR의 차이

| 용어 | 플랫폼 | 풀네임 |
|---|---|---|
| **PR** | GitHub | Pull Request |
| **MR** | GitLab | Merge Request |

둘 다 **같은 개념**이다. 플랫폼만 다를 뿐.

---

### 한 줄 정의

> "내 브랜치의 코드를 main에 합쳐달라고 팀원에게 요청하는 것"

---

### 왜 헷갈리나?

`git pull`과 `git merge`는 **내 로컬**에서 내가 실행하는 명령어인데,  
PR/MR에서의 pull과 merge는 **상대방(팀원, 관리자)이 하는 행동**이다.

```
나    → "내 코드를 pull해서 보고, merge해 주세요!" → 팀원
                                                     (Request = 요청)
```

주어가 반대이기 때문에 처음에 헷갈리는 것이 자연스럽다.

---

### PR의 실제 흐름

```bash
# 1. 새 기능용 브랜치 생성
git checkout -b feature/login

# 2. 코드 작성 후 커밋
git add .
git commit -m "로그인 기능 구현"

# 3. 내 브랜치를 GitHub에 push
git push origin feature/login

# 4. GitHub 웹사이트에서 PR 버튼 클릭
#    "feature/login 브랜치를 main에 합쳐주세요!"
```

> push까지는 터미널, PR 생성은 GitHub 웹사이트에서 버튼으로 한다.

---

### PR 생성 후 진행 순서

```
[나]       feature/login 브랜치 → PR 생성
[팀원 A]   코드 리뷰: "여기 버그 있어요" 댓글
[나]       수정 후 다시 commit & push
[팀원 A]   "Approve (승인)" 클릭
[팀장]     "Merge" 클릭 → main 브랜치에 합쳐짐 ✅
```

---

### PR 관련 용어 정리

| 용어 | 의미 |
|---|---|
| **PR / MR** | 내 브랜치를 main에 합쳐달라는 요청 |
| **코드 리뷰** | PR을 보고 팀원들이 코드를 검토하는 것 |
| **Approve** | 팀원이 "이 코드 괜찮아요" 승인하는 것 |
| **Merge** | 검토 완료 후 실제로 코드를 합치는 것 |
| **Request Changes** | "수정 필요해요" 변경 요청 |

---

## 5. 브랜치(Branch) 개념

### 한 줄 정의

> 브랜치 = 원본(main)을 건드리지 않고 안전하게 작업할 수 있는 **나만의 평행 세계**

---

### 왜 필요한가?

`main` 브랜치는 실제 서비스에 배포된 안정적인 코드다.  
여기서 바로 개발하다가 코드가 망가지면 서비스 전체에 문제가 생긴다.  
그래서 별도 브랜치를 만들어 작업하고, 완성되면 main에 합친다.

---

### 브랜치 타임라인 구조

```
main    ●────────────●────────────────────────────●  ← C6 (merge 완료)
        C1           C2                           ↑
                      \                           │
feature/login          ●──────────●──────────────┘
                       C3(로그인UI) C4(API연결)  (PR → Merge)

                      \
bugfix/header          ●──────────────────────────┘
                       C5(헤더 버그 수정)         (PR → Merge)
```

---

### 브랜치 이름 짓는 관례

| 접두어 | 용도 | 예시 |
|---|---|---|
| `feature/` | 새 기능 개발 | `feature/login` |
| `bugfix/` | 버그 수정 | `bugfix/header-error` |
| `hotfix/` | 긴급 수정 (배포 중 발생) | `hotfix/payment-crash` |
| `release/` | 배포 준비 | `release/v1.2.0` |

---

## 6. 브랜치 명령어

```bash
git branch                      # 현재 브랜치 목록 확인
git branch feature/login        # 새 브랜치 만들기
git checkout feature/login      # 그 브랜치로 이동
git checkout -b feature/login   # 만들고 바로 이동 (위 두 개를 한 번에)
```

```bash
# 작업 완료 후 main으로 돌아와서 합치기
git checkout main
git merge feature/login         # feature/login 브랜치를 main에 합치기
```

```bash
git branch -d feature/login     # merge 완료한 브랜치 삭제
```

---

### 브랜치 + PR 세트로 기억하기

```
브랜치 생성 → 작업 → commit → push → PR 생성 → 코드 리뷰 → Merge → 브랜치 삭제
```

브랜치와 PR은 항상 세트로 움직인다.

---

## 7. 전체 명령어 요약표

### 저장소 초기화 & 연결

| 명령어 | 설명 |
|---|---|
| `git init` | 현재 폴더를 Git 저장소로 초기화 |
| `git clone <URL>` | GitHub 저장소를 내 컴퓨터로 복제 |
| `git remote add origin <URL>` | 원격 저장소 주소 등록 |
| `git remote -v` | 등록된 원격 저장소 확인 |

### 변경사항 저장

| 명령어 | 설명 |
|---|---|
| `git status` | 현재 변경사항 상태 확인 |
| `git add <파일>` | 특정 파일을 스테이징 영역에 추가 |
| `git add .` | 모든 변경 파일을 스테이징 영역에 추가 |
| `git commit -m "메시지"` | 스테이징 내용을 로컬 저장소에 저장 |
| `git log` | 커밋 히스토리 확인 |

### 원격 저장소 연동

| 명령어 | 설명 |
|---|---|
| `git push` | 로컬 커밋을 GitHub에 업로드 |
| `git push -u origin main` | 처음 push 시 기본 원격 브랜치 설정 |
| `git pull` | GitHub 최신 내용을 로컬에 반영 |
| `git fetch` | GitHub 내용을 가져오기만 함 (merge 안 함) |

### 브랜치

| 명령어 | 설명 |
|---|---|
| `git branch` | 브랜치 목록 확인 |
| `git branch <이름>` | 새 브랜치 생성 |
| `git checkout <이름>` | 브랜치 이동 |
| `git checkout -b <이름>` | 브랜치 생성 + 이동 (한 번에) |
| `git merge <브랜치명>` | 현재 브랜치에 다른 브랜치 합치기 |
| `git branch -d <이름>` | 브랜치 삭제 |

---

## 마무리 — 실무 워크플로우 한 장 요약

```
1. git clone <URL>              # 프로젝트 처음 받아오기

2. git checkout -b feature/xxx  # 새 기능 브랜치 생성

3. (코드 작성)

4. git add .
   git commit -m "기능 설명"    # 로컬에 저장

5. git push origin feature/xxx  # GitHub에 내 브랜치 올리기

6. GitHub에서 PR 생성           # 팀원에게 검토 요청

7. 코드 리뷰 후 Merge           # main에 합쳐짐

8. git checkout main
   git pull                     # 최신 main 받아오기

9. git branch -d feature/xxx    # 완료된 브랜치 정리

→ 2번부터 반복
```

---

*이 노트는 GitHub 기초 학습 내용을 정리한 복습용 문서입니다.*
