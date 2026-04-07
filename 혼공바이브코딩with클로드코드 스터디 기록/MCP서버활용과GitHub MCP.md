# 클로드코드 공부 복습 — 2026년 4월 8일

## 오늘 한 것
- 클로드코드에서 MCP(Model Context Protocol) 서버 목록 확인 및 관리
- GitHub MCP 서버 연결 확인
- **GitHub MCP를 사용해서 쇼핑 리스트 앱을 GitHub 저장소에 업로드**

---

## 1. MCP(Model Context Protocol)란?

MCP는 Claude Code가 외부 도구·서비스와 통신하기 위한 프로토콜이다.  
클로드코드에 MCP 서버를 연결하면 웹 브라우저 자동화, GitHub API 호출, 문서 검색 등을 Claude가 직접 수행할 수 있다.

### 오늘 확인한 MCP 서버

| 서버명 | 타입 | 용도 | 상태 |
|---|---|---|---|
| `context7` | HTTP | 라이브러리·프레임워크 최신 문서 조회 | 연결됨 |
| `playwright` | npx | 브라우저 자동화 (클릭, 스크린샷 등) | 연결됨 |
| `github` | HTTP | GitHub API (저장소, 이슈, PR 관리) | 연결됨 |

---

## 2. MCP 서버 관리 명령어

### MCP 서버 목록 확인

```bash
claude mcp list
```

출력 예시:
```
Checking MCP server health...

context7: https://mcp.context7.com/mcp (HTTP) - ✓ Connected
playwright: npx @playwright/mcp@latest - ✓ Connected
github: https://api.githubcopilot.com/mcp (HTTP) - ✓ Connected
```

### 특정 MCP 서버 상세 정보 확인

```bash
claude mcp get github
```

출력 예시:
```
github:
  Scope: Local config (private to you in this project)
  Status: ✓ Connected
  Type: http
  URL: https://api.githubcopilot.com/mcp
  Headers:
    Authorization: Bearer $GITHUB_API_KEY
```

**핵심:** `$GITHUB_API_KEY` 환경변수에 GitHub Personal Access Token을 설정하면 GitHub MCP가 인증된다.

### MCP 서버 제거

```bash
claude mcp remove "github" -s local
```

---

## 3. GitHub MCP 주요 도구 목록

| 도구 | 설명 |
|---|---|
| `mcp__github__get_me` | 인증된 GitHub 사용자 정보 조회 |
| `mcp__github__create_repository` | 새 저장소 생성 |
| `mcp__github__push_files` | 여러 파일을 한 번에 커밋·업로드 |
| `mcp__github__get_file_contents` | 파일/디렉토리 내용 조회 |
| `mcp__github__list_issues` | 이슈 목록 조회 |
| `mcp__github__create_pull_request` | PR 생성 |
| `mcp__github__search_repositories` | 저장소 검색 |
| `mcp__github__list_branches` | 브랜치 목록 조회 |
| `mcp__github__merge_pull_request` | PR 병합 |

---

## 4. 실습: GitHub MCP로 저장소 생성 및 파일 업로드

### Step 1. 사용자 정보 확인

```
mcp__github__get_me
```

응답 예시:
```json
{
  "login": "91wjheo",
  "id": 37071926,
  "profile_url": "https://github.com/91wjheo"
}
```

### Step 2. 저장소 생성

```
mcp__github__create_repository
  name: "shopping-list-app"
  description: "쇼핑 리스트 웹 앱"
  private: false
  autoInit: false
```

응답으로 저장소 URL이 반환된다:
```json
{
  "url": "https://github.com/91wjheo/shopping-list-app"
}
```

### Step 3. 파일 업로드 (push_files)

```
mcp__github__push_files
  owner: "91wjheo"
  repo: "shopping-list-app"
  branch: "main"
  message: "Initial commit: 쇼핑 리스트 앱 추가"
  files:
    - path: "index.html"
      content: "(파일 내용)"
    - path: "README.md"
      content: "(파일 내용)"
```

**핵심:** `push_files`는 여러 파일을 **한 번의 커밋**으로 올릴 수 있다. 초기화(`autoInit: false`)한 빈 저장소에도 바로 사용 가능하다.

---

## 5. 기존 저장소에 파일 추가하기

### 디렉토리 구조 확인

```
mcp__github__get_file_contents
  owner: "91wjheo"
  repo: "book-study-claude-code-vibe-coding"
  path: "혼공바이브코딩with클로드코드 스터디 기록"
```

폴더 내 파일 목록이 배열로 반환된다. 각 항목에는 `name`, `path`, `sha`, `html_url` 등이 포함된다.

### 특정 파일 내용 읽기

```
mcp__github__get_file_contents
  owner: "91wjheo"
  repo: "book-study-claude-code-vibe-coding"
  path: "혼공바이브코딩with클로드코드 스터디 기록/파일명.md"
```

### 기존 저장소에 파일 업로드

```
mcp__github__push_files
  owner: "91wjheo"
  repo: "book-study-claude-code-vibe-coding"
  branch: "main"
  message: "스터디 기록 추가"
  files:
    - path: "혼공바이브코딩with클로드코드 스터디 기록/새파일.md"
      content: "(내용)"
```

**주의:** 한글 경로도 그대로 `path`에 입력하면 된다. URL 인코딩은 자동으로 처리된다.

---

## 6. GitHub MCP vs git CLI 비교

| 항목 | git CLI | GitHub MCP |
|---|---|---|
| 로컬 git 저장소 필요 | 필요 | 불필요 |
| 인증 방식 | SSH키 또는 HTTPS | Personal Access Token |
| 여러 파일 한 번에 커밋 | `git add` + `git commit` | `push_files` 한 번으로 가능 |
| 저장소 생성 | `gh repo create` (별도 CLI) | `create_repository` |
| API 기반 작업 | 불가 (git CLI 범위 초과) | 이슈·PR·검색 등 모두 가능 |

---

## 오늘의 핵심 요약

1. **`claude mcp list`** 로 연결된 MCP 서버 목록과 상태를 확인할 수 있다.
2. GitHub MCP는 **`$GITHUB_API_KEY` 환경변수**로 Personal Access Token을 인증한다.
3. **`push_files`** 도구 하나로 여러 파일을 한 커밋에 업로드할 수 있다 — 로컬 git 없이도 가능.
4. **`get_file_contents`** 로 저장소 내 디렉토리 구조와 파일 내용을 읽어볼 수 있다.
5. MCP를 활용하면 Claude가 GitHub 저장소 생성, 파일 업로드, 이슈 관리 등을 대화만으로 수행한다.
