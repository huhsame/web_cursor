# Chapter 15. Cursor에 MCP 연결하기

이제 직접 Cursor에 MCP를 연결해봅니다!
**Context7 MCP**를 연결해서 최신 개발 문서를 참조해볼 거예요.

---

## 15.1 MCP 연결 방식 이해하기

### 두 가지 연결 방식

MCP를 연결하는 방법은 크게 두 가지입니다.

| 방식 | 설명 | 특징 |
|------|------|------|
| **STDIO** | 내 컴퓨터에서 실행 | 로컬 전용, 명령어 기반 |
| **HTTP** | 원격 서버에서 실행 | 어디서든 접속 가능 |

### STDIO 방식

```
내 컴퓨터에서 MCP 서버를 직접 실행
↓
Cursor가 이 서버와 통신
```

- 장점: 빠름, 인터넷 없이도 사용 가능
- 단점: 내 컴퓨터에서만 사용 가능

### HTTP 방식

```
원격 서버에서 MCP 서버 실행
↓
Cursor가 인터넷을 통해 접속
```

- 장점: 어디서든 사용 가능, 팀 공유 가능
- 단점: 인터넷 필요

> 💡 **우리는 STDIO 방식을 사용합니다.**
> 대부분의 MCP가 이 방식을 지원해요.

---

## 15.2 Smithery에서 MCP 찾기

### Smithery란?

> **MCP를 모아놓은 마켓플레이스**입니다.
>
> 6,000개 이상의 MCP가 있어요!

### Step 1: Smithery 접속

브라우저에서 **[smithery.ai](https://smithery.ai)** 접속

<!-- 스크린샷: Smithery 메인 페이지 -->

### Step 2: Context7 검색

1. 검색창에 `context7` 입력
2. **Context7** 클릭

<!-- 스크린샷: Context7 검색 결과 -->

### Context7이 뭔가요?

> **개발 문서를 AI에게 제공해주는 MCP**입니다.

| 기능 | 설명 |
|------|------|
| 문서 검색 | Next.js, React 등 최신 문서 검색 |
| 코드 예제 | 공식 문서의 코드 예제 가져오기 |
| 최신 정보 | 학습되지 않은 최신 버전 정보 |

**AI가 모르는 최신 문법도 Context7이 알려줍니다!**

---

## 15.3 Context7 연결 정보 확인

### Step 1: 설치 방법 확인

Context7 페이지에서 **설치 방법**을 확인합니다.

<!-- 스크린샷: Context7 설치 방법 -->

### Step 2: JSON 복사

Cursor용 JSON 설정을 복사합니다.

```json
{
  "mcpServers": {
    "context7": {
      "command": "npx",
      "args": ["-y", "@upstash/context7-mcp"]
    }
  }
}
```

> ⚠️ **실제 Smithery에서 최신 JSON을 복사하세요!**
> 버전이 바뀔 수 있습니다.

---

## 15.4 Cursor에 MCP 설정하기

### Step 1: MCP 설정 열기

Cursor에서 **설정**을 엽니다.

**방법 1:** `Cmd + Shift + P` (Mac) 또는 `Ctrl + Shift + P` (Windows)
→ `Preferences: Open User Settings (JSON)` 검색

**방법 2:** Cursor 설정 → Features → MCP

<!-- 스크린샷: Cursor MCP 설정 화면 -->

### Step 2: MCP 서버 추가

**Add new MCP server** 클릭

<!-- 스크린샷: Add MCP Server 버튼 -->

### Step 3: 정보 입력

| 항목 | 입력값 |
|------|--------|
| Name | `context7` |
| Type | `command` |
| Command | `npx -y @upstash/context7-mcp` |

<!-- 스크린샷: MCP 서버 정보 입력 -->

### Step 4: 저장 및 확인

저장 후 MCP 목록에 **context7**이 추가되었는지 확인!

```
MCP Servers
├── context7 ✅ (연결됨)
```

<!-- 스크린샷: MCP 연결 완료 -->

---

## 15.5 mcp.json 파일로 설정하기 (대안)

### 프로젝트별 MCP 설정

프로젝트 루트에 `.cursor/mcp.json` 파일을 만들어서 설정할 수도 있습니다.

### Step 1: 폴더 및 파일 생성

```
프로젝트 폴더/
├── .cursor/
│   └── mcp.json    ← 이 파일 생성
├── app/
├── package.json
└── ...
```

### Step 2: mcp.json 작성

```json
{
  "mcpServers": {
    "context7": {
      "command": "npx",
      "args": ["-y", "@upstash/context7-mcp"]
    }
  }
}
```

### 언제 이 방법을 쓰나요?

| 상황 | 추천 방법 |
|------|----------|
| 모든 프로젝트에서 사용 | Cursor 설정 (전역) |
| 특정 프로젝트에서만 사용 | mcp.json (프로젝트별) |
| 팀원과 공유 | mcp.json (Git에 포함) |

---

## 15.6 Context7 MCP 사용해보기

### 테스트 1: Next.js 최신 문법 질문

Cursor Chat에서 질문해봅니다:

```
@context7 Next.js 15에서 새로 바뀐 점이 뭐야?
```

<!-- 스크린샷: Context7 사용 예시 -->

### 예상 결과

```
AI: Context7 MCP를 통해 Next.js 15 문서를 확인했습니다.

주요 변경사항:
1. React 19 지원
2. 새로운 캐싱 방식
3. Turbopack 안정화
...
```

### 테스트 2: 코드 작성 요청

```
@context7 Next.js 15 문법으로 서버 컴포넌트에서
데이터 fetching하는 코드 짜줘
```

### 예상 결과

```typescript
// app/posts/page.tsx
async function getPosts() {
  const res = await fetch('https://api.example.com/posts', {
    next: { revalidate: 3600 }
  })
  return res.json()
}

export default async function PostsPage() {
  const posts = await getPosts()

  return (
    <ul>
      {posts.map((post) => (
        <li key={post.id}>{post.title}</li>
      ))}
    </ul>
  )
}
```

**최신 문서를 참조해서 정확한 코드를 작성합니다!**

---

## 15.7 MCP 도구 확인하기

### 연결된 MCP 도구 보기

Cursor Chat에서 MCP가 제공하는 **도구(Tools)**를 확인할 수 있습니다.

<!-- 스크린샷: MCP Tools 목록 -->

### Context7이 제공하는 도구

| 도구 | 기능 |
|------|------|
| `resolve-library-id` | 라이브러리 ID 검색 |
| `get-library-docs` | 문서 내용 가져오기 |

### 도구 호출 과정

```
1. 사용자: "Next.js 라우팅 방법 알려줘"
2. AI: (resolve-library-id 호출) → Next.js 라이브러리 ID 찾기
3. AI: (get-library-docs 호출) → 라우팅 관련 문서 가져오기
4. AI: 문서 기반으로 답변 생성
```

> 💡 **AI가 알아서 적절한 도구를 선택해서 사용합니다!**

---

## 15.8 MCP 사용 시 주의사항

### 권한 확인 팝업

MCP 도구를 처음 사용할 때 **권한 확인 팝업**이 뜹니다.

```
Context7 MCP wants to use: get-library-docs
[Allow] [Deny] [Always Allow]
```

| 버튼 | 의미 |
|------|------|
| Allow | 이번만 허용 |
| Deny | 거부 |
| Always Allow | 항상 허용 |

> 💡 **신뢰할 수 있는 MCP는 "Always Allow"를 선택하세요.**

### 민감한 정보 주의

일부 MCP는 **민감한 정보**에 접근할 수 있습니다.

| MCP 종류 | 접근 가능 정보 |
|---------|---------------|
| 파일 시스템 MCP | 내 컴퓨터 파일 |
| DB MCP | 데이터베이스 내용 |
| Git MCP | 코드 저장소 |

> ⚠️ **알 수 없는 MCP는 신중하게 사용하세요!**

---

## 15.9 문제 해결

### "MCP 연결이 안 돼요"

1. **Node.js 설치 확인**
   ```bash
   node --version
   ```
   - 버전이 안 나오면 Node.js 설치 필요

2. **npx 확인**
   ```bash
   npx --version
   ```

3. **Cursor 재시작**
   - MCP 설정 후에는 Cursor를 완전히 종료했다가 다시 시작

### "도구 호출이 안 돼요"

1. **MCP 상태 확인**
   - 설정에서 MCP가 ✅ 상태인지 확인

2. **권한 허용 확인**
   - "Deny"를 눌렀다면 다시 허용 필요

3. **인터넷 연결 확인**
   - Context7은 인터넷이 필요함

### "@context7이 안 먹혀요"

Cursor 버전에 따라 `@context7` 대신 그냥 질문해도 됩니다.
AI가 알아서 필요한 MCP를 사용합니다.

```
Next.js 15 공식 문서 참고해서 라우팅 코드 짜줘
```

---

## 핵심 정리

```
✅ MCP 연결 방식: STDIO(로컬), HTTP(원격)
✅ Smithery에서 MCP 검색 가능
✅ Context7 = 개발 문서 참조 MCP
✅ Cursor 설정 또는 mcp.json으로 연결
✅ 권한 확인 팝업에서 허용 필요
✅ 최신 문법도 MCP로 참조 가능!
```

---

## 다음 챕터에서는

> Supabase MCP를 연결해서
> AI가 직접 데이터베이스를 조회하고 조작하게 해봅니다!
>
> 👉 [Chapter 16. Supabase MCP로 개발하기](chapter16-supabase-mcp.md)
