# 치트시트

강의에서 배운 내용을 한눈에 볼 수 있는 요약입니다.

---

## 터미널 명령어

### Node.js / npm

| 명령어 | 설명 |
|--------|------|
| `node --version` | Node.js 버전 확인 |
| `npm --version` | npm 버전 확인 |
| `npm install` | package.json의 패키지 설치 |
| `npm install [패키지명]` | 특정 패키지 설치 |
| `npm run dev` | 개발 서버 실행 |
| `npm run build` | 프로덕션 빌드 |

### Next.js 프로젝트 생성

```bash
npx create-next-app@latest [프로젝트명]
```

---

## Git 명령어

| 명령어 | 설명 |
|--------|------|
| `git status` | 현재 상태 확인 |
| `git add .` | 모든 변경사항 스테이징 |
| `git add [파일명]` | 특정 파일 스테이징 |
| `git commit -m "메시지"` | 커밋 |
| `git push` | 원격 저장소에 업로드 |
| `git pull` | 원격 저장소에서 다운로드 |
| `git log --oneline` | 커밋 히스토리 (간단히) |
| `git diff` | 변경사항 확인 |
| `git checkout -- .` | 모든 변경사항 되돌리기 |

---

## 단축키

### Cursor / VS Code

| 동작 | Windows | Mac |
|------|---------|-----|
| 저장 | `Ctrl + S` | `Cmd + S` |
| 실행 취소 | `Ctrl + Z` | `Cmd + Z` |
| 다시 실행 | `Ctrl + Y` | `Cmd + Shift + Z` |
| 터미널 열기 | `` Ctrl + ` `` | `` Cmd + ` `` |
| 명령 팔레트 | `Ctrl + Shift + P` | `Cmd + Shift + P` |
| 파일 검색 | `Ctrl + P` | `Cmd + P` |
| 전체 검색 | `Ctrl + Shift + F` | `Cmd + Shift + F` |
| 주석 처리 | `Ctrl + /` | `Cmd + /` |

### 브라우저

| 동작 | Windows | Mac |
|------|---------|-----|
| 새로고침 | `F5` | `Cmd + R` |
| 강력 새로고침 | `Ctrl + Shift + R` | `Cmd + Shift + R` |
| 개발자 도구 | `F12` | `Cmd + Option + I` |

---

## Next.js App Router

### 파일 구조

```
app/
├── layout.tsx      # 공통 레이아웃
├── page.tsx        # / (홈)
├── login/
│   └── page.tsx    # /login
├── products/
│   ├── page.tsx    # /products
│   ├── new/
│   │   └── page.tsx    # /products/new
│   └── [id]/
│       ├── page.tsx    # /products/1, /products/2, ...
│       └── edit/
│           └── page.tsx    # /products/1/edit, ...
```

### 기본 페이지 구조

```tsx
// 서버 컴포넌트 (기본)
export default function Page() {
  return <div>내용</div>
}

// 클라이언트 컴포넌트
'use client'

export default function Page() {
  return <div>내용</div>
}
```

### 동적 라우팅 파라미터

```tsx
// app/products/[id]/page.tsx
export default function Page({ params }: { params: { id: string } }) {
  const productId = params.id
  return <div>상품 {productId}</div>
}
```

---

## React 기초

### useState

```tsx
'use client'
import { useState } from 'react'

export default function Counter() {
  const [count, setCount] = useState(0)

  return (
    <button onClick={() => setCount(count + 1)}>
      클릭: {count}
    </button>
  )
}
```

### useEffect

```tsx
'use client'
import { useEffect, useState } from 'react'

export default function Page() {
  const [data, setData] = useState(null)

  useEffect(() => {
    // 컴포넌트 마운트 시 실행
    fetchData()
  }, []) // 빈 배열 = 한 번만 실행

  return <div>{data}</div>
}
```

### 조건부 렌더링

```tsx
// if문
{isLoggedIn ? <LogoutButton /> : <LoginButton />}

// &&
{isLoggedIn && <UserProfile />}
```

### 리스트 렌더링

```tsx
{items.map((item) => (
  <div key={item.id}>{item.name}</div>
))}
```

---

## Tailwind CSS

### 자주 쓰는 클래스

| 카테고리 | 클래스 | 설명 |
|----------|--------|------|
| **레이아웃** | `flex` | Flexbox |
| | `grid` | Grid |
| | `grid-cols-3` | 3열 그리드 |
| | `gap-4` | 간격 1rem |
| **정렬** | `justify-center` | 가로 중앙 |
| | `items-center` | 세로 중앙 |
| | `text-center` | 텍스트 중앙 |
| **크기** | `w-full` | 너비 100% |
| | `h-screen` | 높이 화면 전체 |
| | `max-w-md` | 최대 너비 28rem |
| **여백** | `p-4` | 패딩 1rem |
| | `m-4` | 마진 1rem |
| | `px-4` | 좌우 패딩 |
| | `py-4` | 상하 패딩 |
| **색상** | `bg-blue-500` | 배경색 |
| | `text-white` | 글자색 |
| | `border-gray-300` | 테두리색 |
| **텍스트** | `text-lg` | 크기 |
| | `font-bold` | 굵기 |
| **테두리** | `rounded` | 둥근 모서리 |
| | `border` | 테두리 |
| | `shadow` | 그림자 |

### 반응형

| 접두사 | 화면 크기 |
|--------|----------|
| (없음) | 기본 (모바일) |
| `sm:` | 640px 이상 |
| `md:` | 768px 이상 |
| `lg:` | 1024px 이상 |
| `xl:` | 1280px 이상 |

```html
<div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3">
```

---

## Supabase

### 클라이언트 설정

```typescript
// lib/supabase.ts
import { createClient } from '@supabase/supabase-js'

const supabaseUrl = process.env.NEXT_PUBLIC_SUPABASE_URL!
const supabaseKey = process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!

export const supabase = createClient(supabaseUrl, supabaseKey)
```

### 데이터 조회 (SELECT)

```typescript
// 전체 조회
const { data, error } = await supabase
  .from('products')
  .select('*')

// 조건 조회
const { data, error } = await supabase
  .from('products')
  .select('*')
  .eq('id', 1)
  .single()

// 정렬
const { data, error } = await supabase
  .from('products')
  .select('*')
  .order('created_at', { ascending: false })

// 조인
const { data, error } = await supabase
  .from('products')
  .select('*, profiles(nickname)')
```

### 데이터 추가 (INSERT)

```typescript
const { data, error } = await supabase
  .from('products')
  .insert({ title: '상품명', price: 10000 })
  .select()
```

### 데이터 수정 (UPDATE)

```typescript
const { data, error } = await supabase
  .from('products')
  .update({ title: '새 제목' })
  .eq('id', 1)
```

### 데이터 삭제 (DELETE)

```typescript
const { error } = await supabase
  .from('products')
  .delete()
  .eq('id', 1)
```

### 인증

```typescript
// 회원가입
const { data, error } = await supabase.auth.signUp({
  email: 'email@example.com',
  password: 'password',
  options: {
    data: { nickname: '닉네임' }
  }
})

// 로그인
const { data, error } = await supabase.auth.signInWithPassword({
  email: 'email@example.com',
  password: 'password'
})

// 로그아웃
await supabase.auth.signOut()

// 현재 사용자
const { data: { user } } = await supabase.auth.getUser()

// 상태 변경 감지
supabase.auth.onAuthStateChange((event, session) => {
  console.log(event, session)
})
```

### 스토리지 (파일 업로드)

```typescript
// 업로드
const { data, error } = await supabase.storage
  .from('product-images')
  .upload(`${Date.now()}.jpg`, file)

// URL 가져오기
const { data: { publicUrl } } = supabase.storage
  .from('product-images')
  .getPublicUrl(filePath)
```

---

## SQL 기초

### 테이블 생성

```sql
CREATE TABLE products (
  id BIGSERIAL PRIMARY KEY,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  title TEXT NOT NULL,
  price INTEGER NOT NULL,
  user_id UUID REFERENCES profiles(id)
);
```

### RLS 활성화

```sql
ALTER TABLE products ENABLE ROW LEVEL SECURITY;

CREATE POLICY "누구나 조회 가능"
ON products FOR SELECT
USING (true);

CREATE POLICY "본인만 수정 가능"
ON products FOR UPDATE
USING (auth.uid() = user_id);
```

---

## 환경변수

### .env.local

```
NEXT_PUBLIC_SUPABASE_URL=https://xxx.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=eyJhbG...
```

- `NEXT_PUBLIC_` 접두사 = 브라우저에서 사용 가능
- `.gitignore`에 추가되어 있음 (GitHub에 안 올라감)
- Vercel 배포 시 Environment Variables에 추가 필요

---

## 체크리스트

### 개발 시작 전

- [ ] Node.js 18+ 설치됨
- [ ] Cursor 설치됨
- [ ] GitHub 계정 있음
- [ ] Supabase 계정 있음

### 배포 전

- [ ] 모든 기능 테스트 완료
- [ ] 에러 없음 (콘솔 확인)
- [ ] .env.local 값 Vercel에 설정
- [ ] GitHub에 Push 완료

---

이 치트시트를 프린트해서 옆에 두고 사용하세요! 📝

