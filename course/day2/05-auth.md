# Day 2 - 5교시: 회원가입/로그인 구현

## 학습 목표
- Supabase Auth를 이해한다
- 회원가입 기능을 구현한다
- 로그인/로그아웃 기능을 구현한다
- 인증 상태를 관리한다

---

## 1. Supabase Auth란?

### 쉬운 설명

> 로그인 시스템을 직접 만들려면:
> - 비밀번호 암호화
> - 세션 관리
> - 토큰 발급
> - 보안 처리
> - ...정말 많은 것이 필요해요!
>
> **Supabase Auth**는 이 모든 걸 대신 해줍니다.
> 우리는 그냥 "로그인해줘" 함수만 호출하면 돼요!

### 공식적인 설명
```
Supabase Auth는 사용자 인증 및 권한 부여를 위한
완전 관리형 서비스입니다.
이메일/비밀번호, 소셜 로그인(Google, GitHub 등),
매직 링크, 전화번호 인증 등을 지원합니다.
JWT 토큰 기반으로 동작하며,
Row Level Security와 통합됩니다.
```

### 지원하는 로그인 방법

```
┌────────────────────────────────────────┐
│           Supabase Auth                │
├────────────────────────────────────────┤
│                                        │
│  📧 이메일/비밀번호  ← 오늘 사용!       │
│  🔗 매직 링크                          │
│  📱 전화번호                           │
│                                        │
│  ─── 소셜 로그인 ───                   │
│  🔵 Google                             │
│  ⚫ GitHub                             │
│  🟡 Kakao                              │
│  🟢 Naver                              │
│  ...                                   │
│                                        │
└────────────────────────────────────────┘
```

---

## 2. Auth 기본 설정

### Supabase 대시보드에서 설정

1. Authentication → Settings
2. "Email Auth" 활성화 확인
3. (선택) "Confirm email" 비활성화 (개발 편의)

```
💡 개발할 때 팁
   "Confirm email"을 끄면
   회원가입 후 이메일 확인 없이 바로 로그인 가능!
   실제 서비스에서는 켜두는 게 좋아요.
```

### URL 설정

Authentication → URL Configuration:

```
Site URL: http://localhost:3000
Redirect URLs: http://localhost:3000/**
```

---

## 3. 회원가입 페이지 만들기

### 폴더 구조

```
app/
├── signup/
│   └── page.tsx   ← 생성!
└── ...
```

### signup/page.tsx

```tsx
'use client';

import { useState } from 'react';
import { supabase } from '@/lib/supabase';
import Link from 'next/link';
import { useRouter } from 'next/navigation';

export default function SignUpPage() {
  const router = useRouter();
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [confirmPassword, setConfirmPassword] = useState('');
  const [error, setError] = useState('');
  const [loading, setLoading] = useState(false);

  const handleSignUp = async (e: React.FormEvent) => {
    e.preventDefault();
    setError('');

    // 유효성 검사
    if (password !== confirmPassword) {
      setError('비밀번호가 일치하지 않습니다.');
      return;
    }

    if (password.length < 6) {
      setError('비밀번호는 6자 이상이어야 합니다.');
      return;
    }

    setLoading(true);

    const { data, error: signUpError } = await supabase.auth.signUp({
      email,
      password,
    });

    if (signUpError) {
      setError(signUpError.message);
      setLoading(false);
      return;
    }

    // 회원가입 성공
    alert('회원가입 성공! 로그인 페이지로 이동합니다.');
    router.push('/login');
  };

  return (
    <div className="min-h-screen flex items-center justify-center bg-gray-100">
      <div className="bg-white p-8 rounded-lg shadow-md w-full max-w-md">
        <h1 className="text-2xl font-bold text-center mb-6">회원가입</h1>

        <form onSubmit={handleSignUp} className="space-y-4">
          {/* 이메일 */}
          <div>
            <label className="block text-sm font-medium text-gray-700 mb-1">
              이메일
            </label>
            <input
              type="email"
              value={email}
              onChange={(e) => setEmail(e.target.value)}
              required
              className="w-full border border-gray-300 rounded-lg px-4 py-2 focus:outline-none focus:ring-2 focus:ring-blue-500"
              placeholder="example@email.com"
            />
          </div>

          {/* 비밀번호 */}
          <div>
            <label className="block text-sm font-medium text-gray-700 mb-1">
              비밀번호
            </label>
            <input
              type="password"
              value={password}
              onChange={(e) => setPassword(e.target.value)}
              required
              className="w-full border border-gray-300 rounded-lg px-4 py-2 focus:outline-none focus:ring-2 focus:ring-blue-500"
              placeholder="6자 이상"
            />
          </div>

          {/* 비밀번호 확인 */}
          <div>
            <label className="block text-sm font-medium text-gray-700 mb-1">
              비밀번호 확인
            </label>
            <input
              type="password"
              value={confirmPassword}
              onChange={(e) => setConfirmPassword(e.target.value)}
              required
              className="w-full border border-gray-300 rounded-lg px-4 py-2 focus:outline-none focus:ring-2 focus:ring-blue-500"
              placeholder="비밀번호 다시 입력"
            />
          </div>

          {/* 에러 메시지 */}
          {error && (
            <p className="text-red-500 text-sm">{error}</p>
          )}

          {/* 제출 버튼 */}
          <button
            type="submit"
            disabled={loading}
            className="w-full bg-blue-500 text-white py-2 rounded-lg hover:bg-blue-600 disabled:bg-gray-400 transition"
          >
            {loading ? '처리 중...' : '회원가입'}
          </button>
        </form>

        {/* 로그인 링크 */}
        <p className="mt-4 text-center text-sm text-gray-600">
          이미 계정이 있으신가요?{' '}
          <Link href="/login" className="text-blue-500 hover:underline">
            로그인
          </Link>
        </p>
      </div>
    </div>
  );
}
```

---

## 4. 로그인 페이지 만들기

### login/page.tsx

```tsx
'use client';

import { useState } from 'react';
import { supabase } from '@/lib/supabase';
import Link from 'next/link';
import { useRouter } from 'next/navigation';

export default function LoginPage() {
  const router = useRouter();
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [error, setError] = useState('');
  const [loading, setLoading] = useState(false);

  const handleLogin = async (e: React.FormEvent) => {
    e.preventDefault();
    setError('');
    setLoading(true);

    const { data, error: loginError } = await supabase.auth.signInWithPassword({
      email,
      password,
    });

    if (loginError) {
      setError('이메일 또는 비밀번호가 올바르지 않습니다.');
      setLoading(false);
      return;
    }

    // 로그인 성공
    router.push('/');  // 홈으로 이동
    router.refresh();  // 페이지 새로고침 (상태 업데이트)
  };

  return (
    <div className="min-h-screen flex items-center justify-center bg-gray-100">
      <div className="bg-white p-8 rounded-lg shadow-md w-full max-w-md">
        <h1 className="text-2xl font-bold text-center mb-6">로그인</h1>

        <form onSubmit={handleLogin} className="space-y-4">
          {/* 이메일 */}
          <div>
            <label className="block text-sm font-medium text-gray-700 mb-1">
              이메일
            </label>
            <input
              type="email"
              value={email}
              onChange={(e) => setEmail(e.target.value)}
              required
              className="w-full border border-gray-300 rounded-lg px-4 py-2 focus:outline-none focus:ring-2 focus:ring-blue-500"
              placeholder="example@email.com"
            />
          </div>

          {/* 비밀번호 */}
          <div>
            <label className="block text-sm font-medium text-gray-700 mb-1">
              비밀번호
            </label>
            <input
              type="password"
              value={password}
              onChange={(e) => setPassword(e.target.value)}
              required
              className="w-full border border-gray-300 rounded-lg px-4 py-2 focus:outline-none focus:ring-2 focus:ring-blue-500"
              placeholder="비밀번호 입력"
            />
          </div>

          {/* 에러 메시지 */}
          {error && (
            <p className="text-red-500 text-sm">{error}</p>
          )}

          {/* 제출 버튼 */}
          <button
            type="submit"
            disabled={loading}
            className="w-full bg-blue-500 text-white py-2 rounded-lg hover:bg-blue-600 disabled:bg-gray-400 transition"
          >
            {loading ? '로그인 중...' : '로그인'}
          </button>
        </form>

        {/* 회원가입 링크 */}
        <p className="mt-4 text-center text-sm text-gray-600">
          계정이 없으신가요?{' '}
          <Link href="/signup" className="text-blue-500 hover:underline">
            회원가입
          </Link>
        </p>
      </div>
    </div>
  );
}
```

---

## 5. 인증 상태 관리

### 현재 로그인한 사용자 가져오기

```tsx
// 현재 세션 가져오기
const { data: { session } } = await supabase.auth.getSession();

// 현재 유저 가져오기
const { data: { user } } = await supabase.auth.getUser();

// user 객체 구조
{
  id: "uuid-...",
  email: "user@email.com",
  created_at: "2024-01-15T...",
  ...
}
```

### 인증 상태 변화 감지

```tsx
useEffect(() => {
  // 인증 상태 변화 리스너
  const { data: { subscription } } = supabase.auth.onAuthStateChange(
    (event, session) => {
      console.log('Auth event:', event);
      console.log('Session:', session);

      if (event === 'SIGNED_IN') {
        // 로그인됨
      }
      if (event === 'SIGNED_OUT') {
        // 로그아웃됨
      }
    }
  );

  // 클린업
  return () => subscription.unsubscribe();
}, []);
```

---

## 6. 헤더 컴포넌트 (로그인 상태 표시)

### components/Header.tsx

```tsx
'use client';

import { useEffect, useState } from 'react';
import { supabase } from '@/lib/supabase';
import Link from 'next/link';
import { useRouter } from 'next/navigation';
import type { User } from '@supabase/supabase-js';

export default function Header() {
  const router = useRouter();
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    // 초기 유저 정보 가져오기
    const getUser = async () => {
      const { data: { user } } = await supabase.auth.getUser();
      setUser(user);
      setLoading(false);
    };
    getUser();

    // 인증 상태 변화 감지
    const { data: { subscription } } = supabase.auth.onAuthStateChange(
      (event, session) => {
        setUser(session?.user ?? null);
      }
    );

    return () => subscription.unsubscribe();
  }, []);

  const handleLogout = async () => {
    await supabase.auth.signOut();
    router.push('/');
    router.refresh();
  };

  return (
    <header className="bg-white shadow-sm">
      <div className="max-w-6xl mx-auto px-4 py-4 flex items-center justify-between">
        {/* 로고 */}
        <Link href="/" className="text-xl font-bold text-orange-500">
          🥕 미니마켓
        </Link>

        {/* 네비게이션 */}
        <nav className="flex items-center gap-4">
          {loading ? (
            <span className="text-gray-400">로딩중...</span>
          ) : user ? (
            <>
              <span className="text-sm text-gray-600">{user.email}</span>
              <Link
                href="/products/new"
                className="bg-orange-500 text-white px-4 py-2 rounded-lg hover:bg-orange-600 transition"
              >
                글쓰기
              </Link>
              <button
                onClick={handleLogout}
                className="text-gray-600 hover:text-gray-800"
              >
                로그아웃
              </button>
            </>
          ) : (
            <>
              <Link
                href="/login"
                className="text-gray-600 hover:text-gray-800"
              >
                로그인
              </Link>
              <Link
                href="/signup"
                className="bg-orange-500 text-white px-4 py-2 rounded-lg hover:bg-orange-600 transition"
              >
                회원가입
              </Link>
            </>
          )}
        </nav>
      </div>
    </header>
  );
}
```

### layout.tsx에 헤더 추가

```tsx
// app/layout.tsx
import Header from './components/Header';

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="ko">
      <body>
        <Header />
        <main>{children}</main>
      </body>
    </html>
  );
}
```

---

## 7. 로그아웃 구현

### 로그아웃 함수

```tsx
const handleLogout = async () => {
  const { error } = await supabase.auth.signOut();

  if (error) {
    console.error('로그아웃 에러:', error.message);
    return;
  }

  // 홈으로 이동
  router.push('/');
  router.refresh();
};
```

---

## 8. Auth 흐름 정리

### 회원가입 흐름

```
1. 사용자: 이메일, 비밀번호 입력
2. supabase.auth.signUp() 호출
3. Supabase: 유저 생성, (이메일 확인 발송)
4. 성공 → 로그인 페이지로 이동
```

### 로그인 흐름

```
1. 사용자: 이메일, 비밀번호 입력
2. supabase.auth.signInWithPassword() 호출
3. Supabase: 인증 확인, JWT 토큰 발급
4. 토큰이 브라우저에 저장됨 (자동)
5. 성공 → 홈으로 이동
```

### 로그아웃 흐름

```
1. supabase.auth.signOut() 호출
2. 브라우저의 토큰 삭제
3. 홈으로 이동
```

---

## 9. 테스트하기

### 테스트 체크리스트

```
□ 회원가입 페이지 접속 (/signup)
□ 새 계정으로 회원가입
□ 로그인 페이지 접속 (/login)
□ 방금 만든 계정으로 로그인
□ 헤더에 이메일 표시되는지 확인
□ 로그아웃 버튼 동작 확인
□ 새로고침해도 로그인 유지되는지 확인
```

### Supabase에서 확인

Authentication → Users에서 생성된 계정 확인 가능!

---

## 핵심 정리

```
✅ Supabase Auth = 인증 시스템 올인원
✅ signUp() = 회원가입
✅ signInWithPassword() = 로그인
✅ signOut() = 로그아웃
✅ getUser() = 현재 유저 정보
✅ onAuthStateChange = 인증 상태 변화 감지
```

---

## Auth 메서드 요약

```typescript
// 회원가입
await supabase.auth.signUp({ email, password });

// 로그인
await supabase.auth.signInWithPassword({ email, password });

// 로그아웃
await supabase.auth.signOut();

// 현재 유저
await supabase.auth.getUser();

// 현재 세션
await supabase.auth.getSession();

// 상태 변화 감지
supabase.auth.onAuthStateChange((event, session) => {});
```

---

## 다음 시간 예고

> 회원 기능이 완성됐으니,
> 이제 상품 등록 기능을 만들어봅니다!
> 로그인한 사용자만 글을 쓸 수 있게 하고,
> 폼으로 상품 정보를 입력받아 DB에 저장합니다.
