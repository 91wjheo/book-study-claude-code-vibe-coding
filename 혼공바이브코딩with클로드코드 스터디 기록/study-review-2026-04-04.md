# 클로드코드 공부 복습 — 2026년 4월 4일

## 오늘 만든 것
**AI 공감 다이어리** (`index.html`)
- 오늘 하루를 한 줄로 쓰면, AI가 감정을 분석하고 공감·위로 메시지를 돌려주는 웹앱
- 서버 없이 브라우저에서 직접 열 수 있는 단일 HTML 파일

---

## 1. 멀티 에이전트 협업 패턴

클로드코드에서는 `backend-agent`, `frontend-agent`, `qa-agent` 같은 **전문 에이전트**를 지정해 역할을 분리할 수 있다.

```
프롬프트 예시:
"backend-agent가 OpenRouter API를 연동하고,
 frontend-agent가 UI를 만들고,
 qa-agent가 테스트 후 문제를 완전히 해결할 때까지 수정해줘."
```

| 에이전트 | 담당한 일 |
|---|---|
| `backend-agent` | OpenRouter API 연동, `analyzeEmotion()` 함수 구현 |
| `frontend-agent` | 따뜻한 다이어리 UI (CSS, 레이아웃, 폰트) |
| `qa-agent` | 7가지 버그 사전 수정 (XSS, 중복 제출, JSON 파싱 오류 등) |

---

## 2. OpenRouter API 연동

### 기본 구조

```javascript
const response = await fetch('https://openrouter.ai/api/v1/chat/completions', {
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${API_KEY}`,
    'Content-Type': 'application/json',
    'HTTP-Referer': 'http://localhost',
    'X-Title': 'AI Empathy Diary'   // ← 영문만 가능 (아래 오류 참고)
  },
  body: JSON.stringify({
    model: 'qwen/qwen3.6-plus:free',
    messages: [
      { role: 'system', content: SYSTEM_PROMPT },
      { role: 'user', content: diaryText }
    ],
    response_format: { type: 'json_object' }  // ← JSON 구조화 응답 요청
  })
});
```

### 모델 선택
- `response_format: { type: 'json_object' }` 로 JSON 형식 응답을 강제할 수 있다.
- 무료 모델 예: `qwen/qwen3.6-plus:free`
- 사용 가능한 모델 목록은 `https://openrouter.ai/api/v1/models` 에서 확인

---

## 3. 발생한 오류와 해결 과정

### 오류 1: ISO-8859-1 헤더 오류

```
Failed to execute 'fetch' on 'Window':
Failed to read the 'headers' property from 'RequestInit':
String contains non ISO-8859-1 code point.
```

**원인:** HTTP 헤더 값에 한글(Korean) 문자를 넣었기 때문

```javascript
// 잘못된 코드
'X-Title': 'AI공감다이어리'   // ← 한글 포함 → 오류

// 수정된 코드
'X-Title': 'AI Empathy Diary'  // ← ASCII만 사용
```

**핵심 개념:** HTTP 헤더는 ISO-8859-1 (ASCII 범위) 문자만 허용한다. 한글, 일본어, 중국어 등은 사용 불가.

---

### 오류 2: 모델을 찾을 수 없음

```
No endpoints found for deepseek/deepseek-chat-v3-0324:free.
```

**원인:** 존재하지 않는 모델 ID를 사용

**해결:** OpenRouter API로 실제 모델 목록을 조회해 유효한 ID 확인 후 변경
- 최종 선택 모델: `qwen/qwen3.6-plus:free`

**배운 점:** 모델 ID는 정확해야 하고, 무료/유료 여부와 실제 존재 여부를 API로 직접 확인해야 한다.

---

## 4. QA 에이전트가 사전 수정한 버그들

`qa-agent`는 실행 전에 다음 7가지 문제를 미리 잡아냈다:

| # | 문제 | 해결 방법 |
|---|---|---|
| 1 | AI가 JSON 대신 마크다운 코드블록으로 응답할 수 있음 | 정규식으로 `` ```json ``` `` 제거 후 파싱 |
| 2 | JSON 파싱 실패 시 앱이 죽음 | `try/catch`로 사용자 친화적 에러 메시지 |
| 3 | XSS 취약점 (innerHTML에 사용자 입력 노출) | `escapeHtml()` 함수로 특수문자 이스케이프 |
| 4 | 로딩 중 textarea 활성화 상태 | 로딩 시 `disabled` 처리 |
| 5 | 버튼 중복 클릭으로 API 중복 호출 | `isSubmitting` 플래그로 가드 |
| 6 | 히스토리 항목에 `result`가 없는 경우 처리 안 됨 | null 체크 추가 |
| 7 | 히스토리 localStorage 파싱 실패 | `try/catch`로 빈 배열 반환 |

---

## 5. 주요 기술 패턴

### 마크다운 코드블록 제거 후 JSON 파싱

```javascript
const cleaned = raw
  .replace(/^```(?:json)?\s*/i, '')
  .replace(/\s*```$/, '')
  .trim();

try {
  return JSON.parse(cleaned);
} catch {
  throw new Error('AI 응답을 파싱할 수 없습니다. 다시 시도해 주세요.');
}
```

### XSS 방어 (`escapeHtml`)

```javascript
function escapeHtml(str) {
  if (typeof str !== 'string') return '';
  return str
    .replace(/&/g, '&amp;')
    .replace(/</g, '&lt;')
    .replace(/>/g, '&gt;')
    .replace(/"/g, '&quot;')
    .replace(/'/g, '&#x27;');
}
```

### 중복 제출 방지

```javascript
let isSubmitting = false;

async function handleSubmit() {
  if (isSubmitting) return;  // 이미 처리 중이면 무시
  isSubmitting = true;
  try {
    // ... API 호출
  } finally {
    isSubmitting = false;
  }
}
```

### localStorage로 히스토리 관리

```javascript
function saveHistory(entry) {
  const history = loadHistory();
  history.unshift(entry);                          // 최신 항목을 앞에 추가
  if (history.length > MAX_HISTORY) history.splice(MAX_HISTORY);  // 최대 5개 유지
  localStorage.setItem(HISTORY_KEY, JSON.stringify(history));
}
```

---

## 6. 프롬프트 엔지니어링 — JSON 응답 유도

시스템 프롬프트에 응답 형식을 직접 명시해 구조화된 데이터를 받았다:

```
응답 형식 (JSON):
{
  "emotion": "감지된 주요 감정 (이모지 포함, 예: 😊 기쁨)",
  "empathy": "공감 메시지 (2-3문장)",
  "comfort": "위로/응원 메시지 (2-3문장)",
  "quote": "오늘 하루와 어울리는 짧은 명언이나 격려 한마디"
}
```

`response_format: { type: 'json_object' }` + 시스템 프롬프트에 JSON 형식 명시 = 안정적인 구조화 응답

---

## 오늘의 핵심 요약

1. **멀티 에이전트** 지시로 역할을 분리하면 복잡한 앱을 체계적으로 만들 수 있다
2. **HTTP 헤더**에는 ASCII 문자만 써야 한다 (한글 불가)
3. **OpenRouter 모델 ID**는 정확해야 하고, API로 직접 확인하는 것이 안전하다
4. **QA 에이전트**는 실행 전에 미리 버그를 잡는 데 효과적이다
5. AI 응답은 항상 마크다운 래핑, JSON 파싱 실패, XSS 등을 방어적으로 처리해야 한다
