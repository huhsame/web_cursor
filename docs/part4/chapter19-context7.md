# Chapter 19. 문서 참조하며 코딩하기

AI가 최신 문법을 모를 때가 있어요.
**Context7 MCP**를 사용하면 AI가 공식 문서를 직접 참조해서 코딩합니다!

---

## 19.1 왜 문서 참조가 필요한가?

### AI의 한계

```
나: "Next.js 15에서 Server Actions 사용법 알려줘"

AI: "이렇게 하면 됩니다..."
    (2023년 버전 문법으로 알려줌)

나: "에러나는데요?"

AI: "죄송합니다, 제 학습 데이터가..."
```

### 문제 상황들

| 상황 | 원인 |
|------|------|
| 코드가 에러남 | 구버전 문법 사용 |
| 라이브러리 업데이트 후 안 됨 | API 변경사항 모름 |
| 새 기능 사용 불가 | 최신 기능 학습 안 됨 |

### Context7로 해결!

```
나: "@context7 Next.js 15 Server Actions 문서 찾아줘"

AI: (공식 문서를 직접 확인)
    "최신 문서에 따르면 이렇게 합니다..."
```

---

## 19.2 Context7 MCP 연결하기

> Part 3에서 이미 설정했다면 건너뛰세요!

### Step 1: MCP 설정 확인

Cursor 설정 → MCP에서 확인:

```
MCP Servers
├── context7 ✅  ← 이게 있으면 OK!
└── supabase ✅
```

### Step 2: 없다면 추가

`.cursor/mcp.json`:

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

### Step 3: Cursor 재시작

설정 후 Cursor를 완전히 종료했다가 다시 시작하세요.

---

## 19.3 Context7 사용법

### 기본 사용법

AI에게 물어볼 때 **라이브러리 이름**을 명확히 말하세요:

```
Next.js 공식 문서에서 App Router 설명 찾아줘
```

```
Supabase 문서에서 Row Level Security 설정법 알려줘
```

```
Tailwind CSS 문서에서 dark mode 설정 방법 찾아줘
```

### Context7가 지원하는 문서

| 라이브러리 | 문서 |
|-----------|------|
| Next.js | 공식 문서 |
| React | 공식 문서 |
| Supabase | 공식 문서 |
| Tailwind CSS | 공식 문서 |
| TypeScript | 공식 문서 |
| 그 외 | 대부분의 인기 라이브러리 |

---

## 19.4 실습: 최신 문법으로 코드 업데이트

### 시나리오: Server Actions로 변경

고구마마켓의 상품 등록 기능을 최신 방식으로 바꿔봅시다.

**기존 코드 (API Route 방식):**

```typescript
// app/api/products/route.ts
export async function POST(request: Request) {
  const body = await request.json()
  // DB에 저장...
}
```

**AI에게 요청:**

```
Next.js 15 문서에서 Server Actions 사용법 찾아서,
상품 등록 기능을 Server Actions로 바꿔줘
```

**Context7가 문서를 확인한 후:**

```
AI: Next.js 15 공식 문서를 확인했습니다.

Server Actions는 이렇게 사용합니다:

1. 'use server' 지시어 추가
2. 서버 함수에서 직접 DB 작업
3. form의 action으로 연결
```

**변경된 코드:**

```typescript
// app/actions/products.ts
'use server'

import { supabase } from '@/lib/supabase'
import { revalidatePath } from 'next/cache'

export async function createProduct(formData: FormData) {
  const title = formData.get('title') as string
  const price = parseInt(formData.get('price') as string)
  const description = formData.get('description') as string

  const { error } = await supabase
    .from('products')
    .insert({ title, price, description })

  if (error) {
    return { error: error.message }
  }

  revalidatePath('/products')
  return { success: true }
}
```

```tsx
// app/products/new/page.tsx
import { createProduct } from '@/app/actions/products'

export default function NewProductPage() {
  return (
    <form action={createProduct}>
      <input name="title" placeholder="상품명" required />
      <input name="price" type="number" placeholder="가격" required />
      <textarea name="description" placeholder="설명" />
      <button type="submit">등록</button>
    </form>
  )
}
```

---

## 19.5 실습: 라이브러리 업데이트 대응

### 시나리오: Supabase 버전 업그레이드

`supabase-js`가 버전 업그레이드 되면서 문법이 바뀌었을 때:

**AI에게 요청:**

```
Supabase JS v2에서 바뀐 점 알려줘.
특히 인증 관련 API가 어떻게 바뀌었어?
```

**Context7 확인 후:**

```
AI: Supabase JS 문서를 확인했습니다.

v1 → v2 주요 변경사항:

1. auth.user() → auth.getUser()
2. auth.session() → auth.getSession()
3. onAuthStateChange 반환값 변경

기존 코드:
```

```typescript
// v1 (기존)
const user = supabase.auth.user()
const session = supabase.auth.session()
```

```typescript
// v2 (새로운 방식)
const { data: { user } } = await supabase.auth.getUser()
const { data: { session } } = await supabase.auth.getSession()
```

**전체 코드 업데이트 요청:**

```
내 AuthContext 코드를 supabase-js v2에 맞게 업데이트해줘
```

---

## 19.6 실습: 새 라이브러리 적용하기

### 시나리오: 무한 스크롤 추가

상품 목록에 무한 스크롤을 추가하고 싶을 때:

**AI에게 요청:**

```
react-intersection-observer 문서 찾아서
무한 스크롤 구현하는 방법 알려줘
```

**Context7 확인 후:**

```
AI: react-intersection-observer 문서를 확인했습니다.

useInView 훅을 사용하면 됩니다:
```

```tsx
'use client'

import { useInView } from 'react-intersection-observer'
import { useEffect, useState } from 'react'
import { supabase } from '@/lib/supabase'

export default function ProductList() {
  const [products, setProducts] = useState<Product[]>([])
  const [page, setPage] = useState(0)
  const [hasMore, setHasMore] = useState(true)
  const { ref, inView } = useInView()

  const loadMore = async () => {
    const { data } = await supabase
      .from('products')
      .select('*')
      .range(page * 10, (page + 1) * 10 - 1)
      .order('created_at', { ascending: false })

    if (data) {
      if (data.length < 10) setHasMore(false)
      setProducts(prev => [...prev, ...data])
      setPage(prev => prev + 1)
    }
  }

  useEffect(() => {
    if (inView && hasMore) {
      loadMore()
    }
  }, [inView])

  return (
    <div>
      {products.map(product => (
        <ProductCard key={product.id} product={product} />
      ))}

      {hasMore && (
        <div ref={ref} className="h-10">
          로딩 중...
        </div>
      )}
    </div>
  )
}
```

---

## 19.7 Context7 활용 팁

### 효과적인 질문 방법

| 좋은 질문 | 나쁜 질문 |
|----------|----------|
| "Next.js 15 문서에서 Server Actions" | "서버 액션 어떻게 해?" |
| "Supabase 문서에서 RLS 설정" | "보안 설정 알려줘" |
| "Tailwind v3 문서에서 dark mode" | "다크모드 CSS" |

### 라이브러리 이름을 정확하게!

```
✅ "Next.js" (O)
❌ "넥스트" (X)

✅ "Tailwind CSS" (O)
❌ "테일윈드" (X)

✅ "supabase-js" (O)
❌ "수파베이스 자바스크립트" (X)
```

### 버전을 명시하면 더 정확!

```
"Next.js 15 App Router에서..."
"Supabase JS v2에서..."
"React 18의 새로운..."
```

---

## 19.8 Context7 + 다른 MCP 조합

### Context7 + Supabase MCP

```
1. Context7로 최신 Supabase 문법 확인
2. Supabase MCP로 현재 DB 구조 확인
3. 최신 문법 + 실제 구조에 맞는 코드 생성!
```

**예시:**

```
Supabase 문서에서 RLS 정책 설정법 찾아줘.
그리고 내 products 테이블에 맞는 RLS 정책 만들어줘.
```

AI가:
1. Context7로 RLS 문서 확인
2. Supabase MCP로 products 테이블 구조 확인
3. 딱 맞는 RLS 정책 SQL 생성!

---

## 19.9 문제 해결

### "문서를 못 찾겠어요"

1. **라이브러리 이름 확인**
   - 정확한 영문 이름 사용

2. **인기 있는 라이브러리인지 확인**
   - 마이너한 라이브러리는 지원 안 될 수 있음

3. **직접 문서 URL 제공**
   ```
   이 문서 참고해서 코드 짜줘:
   https://nextjs.org/docs/app/...
   ```

### "구버전 정보가 나와요"

```
"Next.js 15" 처럼 버전을 명시해줘
```

```
"2024년 기준 최신 문법으로 알려줘"
```

---

## 핵심 정리

```
✅ AI가 최신 문법 모를 때 → Context7!
✅ 라이브러리 이름을 정확하게 (영문)
✅ 버전을 명시하면 더 정확
✅ Context7 + Supabase MCP = 최강 조합
✅ 마이너 라이브러리는 직접 URL 제공
```

---

## 다음 챕터에서는

> **Playwright MCP**로 E2E 테스트를 자동화합니다!
> AI가 직접 브라우저를 조작하며 테스트하는 걸 볼 수 있어요.
>
> 👉 [Chapter 20. E2E 테스트 자동화](chapter20-playwright.md)
