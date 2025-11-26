# Day 2 - 6교시: 상품 등록 기능 만들기

## 학습 목표
- 상품 등록 폼을 만든다
- 인증된 사용자만 접근하게 한다
- 폼 데이터를 Supabase에 저장한다
- 등록 후 상세 페이지로 이동한다

---

## 1. 상품 등록 흐름

```
[로그인 확인] → [폼 입력] → [유효성 검사] → [DB 저장] → [상세 페이지 이동]
     │
     └→ 비로그인 시 로그인 페이지로
```

---

## 2. 상품 등록 페이지 만들기

### 폴더 구조

```
app/
├── products/
│   └── new/
│       └── page.tsx   ← 생성!
└── ...
```

### products/new/page.tsx

```tsx
'use client';

import { useState, useEffect } from 'react';
import { supabase } from '@/lib/supabase';
import { useRouter } from 'next/navigation';
import type { User } from '@supabase/supabase-js';

const CATEGORIES = [
  { id: 'electronics', name: '가전' },
  { id: 'clothing', name: '의류' },
  { id: 'books', name: '도서' },
  { id: 'sports', name: '스포츠' },
  { id: 'furniture', name: '가구' },
  { id: 'etc', name: '기타' },
];

export default function NewProductPage() {
  const router = useRouter();
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState(true);
  const [submitting, setSubmitting] = useState(false);
  const [error, setError] = useState('');

  // 폼 상태
  const [title, setTitle] = useState('');
  const [description, setDescription] = useState('');
  const [price, setPrice] = useState('');
  const [category, setCategory] = useState('etc');

  // 인증 확인
  useEffect(() => {
    const checkUser = async () => {
      const { data: { user } } = await supabase.auth.getUser();

      if (!user) {
        alert('로그인이 필요합니다.');
        router.push('/login');
        return;
      }

      setUser(user);
      setLoading(false);
    };

    checkUser();
  }, [router]);

  // 폼 제출
  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    setError('');

    // 유효성 검사
    if (!title.trim()) {
      setError('제목을 입력해주세요.');
      return;
    }

    if (!price || parseInt(price) < 0) {
      setError('올바른 가격을 입력해주세요.');
      return;
    }

    setSubmitting(true);

    // DB에 저장
    const { data, error: insertError } = await supabase
      .from('products')
      .insert({
        title: title.trim(),
        description: description.trim(),
        price: parseInt(price),
        category,
        user_id: user!.id,
        user_email: user!.email,
      })
      .select()
      .single();

    if (insertError) {
      setError('등록에 실패했습니다. 다시 시도해주세요.');
      console.error(insertError);
      setSubmitting(false);
      return;
    }

    // 성공 → 상세 페이지로 이동
    alert('상품이 등록되었습니다!');
    router.push(`/products/${data.id}`);
  };

  if (loading) {
    return (
      <div className="min-h-screen flex items-center justify-center">
        <p>로딩 중...</p>
      </div>
    );
  }

  return (
    <div className="min-h-screen bg-gray-100 py-10">
      <div className="max-w-2xl mx-auto bg-white rounded-lg shadow-md p-8">
        <h1 className="text-2xl font-bold mb-6">상품 등록</h1>

        <form onSubmit={handleSubmit} className="space-y-6">
          {/* 제목 */}
          <div>
            <label className="block text-sm font-medium text-gray-700 mb-2">
              제목 *
            </label>
            <input
              type="text"
              value={title}
              onChange={(e) => setTitle(e.target.value)}
              placeholder="상품 제목을 입력하세요"
              className="w-full border border-gray-300 rounded-lg px-4 py-2 focus:outline-none focus:ring-2 focus:ring-orange-500"
              maxLength={100}
            />
          </div>

          {/* 카테고리 */}
          <div>
            <label className="block text-sm font-medium text-gray-700 mb-2">
              카테고리
            </label>
            <select
              value={category}
              onChange={(e) => setCategory(e.target.value)}
              className="w-full border border-gray-300 rounded-lg px-4 py-2 focus:outline-none focus:ring-2 focus:ring-orange-500"
            >
              {CATEGORIES.map((cat) => (
                <option key={cat.id} value={cat.id}>
                  {cat.name}
                </option>
              ))}
            </select>
          </div>

          {/* 가격 */}
          <div>
            <label className="block text-sm font-medium text-gray-700 mb-2">
              가격 *
            </label>
            <div className="relative">
              <input
                type="number"
                value={price}
                onChange={(e) => setPrice(e.target.value)}
                placeholder="0"
                min="0"
                className="w-full border border-gray-300 rounded-lg px-4 py-2 pr-12 focus:outline-none focus:ring-2 focus:ring-orange-500"
              />
              <span className="absolute right-4 top-1/2 -translate-y-1/2 text-gray-500">
                원
              </span>
            </div>
          </div>

          {/* 설명 */}
          <div>
            <label className="block text-sm font-medium text-gray-700 mb-2">
              상세 설명
            </label>
            <textarea
              value={description}
              onChange={(e) => setDescription(e.target.value)}
              placeholder="상품에 대한 상세한 설명을 작성해주세요."
              rows={5}
              className="w-full border border-gray-300 rounded-lg px-4 py-2 focus:outline-none focus:ring-2 focus:ring-orange-500 resize-none"
            />
          </div>

          {/* 에러 메시지 */}
          {error && (
            <p className="text-red-500 text-sm">{error}</p>
          )}

          {/* 버튼 */}
          <div className="flex gap-4">
            <button
              type="button"
              onClick={() => router.back()}
              className="flex-1 border border-gray-300 text-gray-700 py-3 rounded-lg hover:bg-gray-50 transition"
            >
              취소
            </button>
            <button
              type="submit"
              disabled={submitting}
              className="flex-1 bg-orange-500 text-white py-3 rounded-lg hover:bg-orange-600 disabled:bg-gray-400 transition"
            >
              {submitting ? '등록 중...' : '등록하기'}
            </button>
          </div>
        </form>
      </div>
    </div>
  );
}
```

---

## 3. 코드 분석

### 인증 확인 로직

```tsx
useEffect(() => {
  const checkUser = async () => {
    const { data: { user } } = await supabase.auth.getUser();

    if (!user) {
      alert('로그인이 필요합니다.');
      router.push('/login');  // 로그인 페이지로 리다이렉트
      return;
    }

    setUser(user);
    setLoading(false);
  };

  checkUser();
}, [router]);
```

```
페이지 로드 → 유저 확인 → 없으면 로그인 페이지로 강제 이동
```

### 폼 제출 로직

```tsx
const { data, error } = await supabase
  .from('products')
  .insert({
    title: title.trim(),        // 앞뒤 공백 제거
    description: description.trim(),
    price: parseInt(price),     // 문자열 → 숫자 변환
    category,
    user_id: user!.id,          // 현재 로그인한 유저 ID
    user_email: user!.email,    // 유저 이메일
  })
  .select()
  .single();
```

```
💡 .select().single()
   insert 후 추가된 데이터를 바로 받아올 수 있음
   단일 객체로 반환 (배열 아님)
```

---

## 4. select 요소 사용하기

### HTML select 기본

```tsx
<select value={category} onChange={(e) => setCategory(e.target.value)}>
  <option value="electronics">가전</option>
  <option value="clothing">의류</option>
  <option value="books">도서</option>
</select>
```

### map으로 동적 생성

```tsx
const CATEGORIES = [
  { id: 'electronics', name: '가전' },
  { id: 'clothing', name: '의류' },
  // ...
];

<select value={category} onChange={(e) => setCategory(e.target.value)}>
  {CATEGORIES.map((cat) => (
    <option key={cat.id} value={cat.id}>
      {cat.name}
    </option>
  ))}
</select>
```

---

## 5. 숫자 입력 처리

### input type="number"

```tsx
<input
  type="number"       // 숫자만 입력 가능
  value={price}
  onChange={(e) => setPrice(e.target.value)}
  min="0"             // 최소값
  placeholder="0"
/>
```

### 문자열 → 숫자 변환

```tsx
// e.target.value는 항상 문자열!
const priceNumber = parseInt(price);  // "10000" → 10000

// 빈 문자열이면 NaN
parseInt("")  // NaN

// 검증
if (isNaN(priceNumber)) {
  setError('숫자를 입력해주세요');
}
```

---

## 6. textarea 사용하기

```tsx
<textarea
  value={description}
  onChange={(e) => setDescription(e.target.value)}
  rows={5}           // 표시할 줄 수
  maxLength={1000}   // 최대 글자 수
  placeholder="설명을 입력하세요..."
  className="resize-none"  // 크기 조절 비활성화
/>
```

---

## 7. 폼 유효성 검사

### 클라이언트 검증

```tsx
const handleSubmit = async (e: React.FormEvent) => {
  e.preventDefault();

  // 1. 제목 검사
  if (!title.trim()) {
    setError('제목을 입력해주세요.');
    return;
  }

  // 2. 가격 검사
  if (!price || parseInt(price) < 0) {
    setError('올바른 가격을 입력해주세요.');
    return;
  }

  // 3. 제목 길이 검사 (선택)
  if (title.length > 100) {
    setError('제목은 100자 이내로 입력해주세요.');
    return;
  }

  // 통과하면 DB 저장...
};
```

### HTML 속성으로 검증

```tsx
<input
  required         // 필수 입력
  minLength={2}    // 최소 글자
  maxLength={100}  // 최대 글자
  min={0}          // 최소 숫자
  max={999999999}  // 최대 숫자
  pattern="[0-9]*" // 정규식 패턴
/>
```

---

## 8. 가격 표시 포맷팅

### 천 단위 콤마

```tsx
// 숫자 → 콤마 포맷
const formatPrice = (price: number) => {
  return price.toLocaleString('ko-KR');
};

// 사용
formatPrice(10000)  // "10,000"
formatPrice(1500000)  // "1,500,000"

// JSX에서
<p>{formatPrice(product.price)}원</p>
```

### 입력할 때 콤마 표시 (고급)

```tsx
const [displayPrice, setDisplayPrice] = useState('');
const [actualPrice, setActualPrice] = useState(0);

const handlePriceChange = (e: React.ChangeEvent<HTMLInputElement>) => {
  const value = e.target.value.replace(/,/g, '');  // 콤마 제거
  const number = parseInt(value) || 0;

  setActualPrice(number);
  setDisplayPrice(number.toLocaleString());
};
```

---

## 9. 에러 처리 패턴

### try-catch 사용

```tsx
const handleSubmit = async (e: React.FormEvent) => {
  e.preventDefault();

  try {
    setSubmitting(true);
    setError('');

    const { data, error } = await supabase
      .from('products')
      .insert({ ... })
      .select()
      .single();

    if (error) throw error;

    router.push(`/products/${data.id}`);
  } catch (err: any) {
    console.error(err);
    setError(err.message || '오류가 발생했습니다.');
  } finally {
    setSubmitting(false);  // 항상 실행
  }
};
```

---

## 10. 테스트하기

### 테스트 순서

```
1. 비로그인 상태에서 /products/new 접속
   → 로그인 페이지로 리다이렉트 되어야 함

2. 로그인 후 /products/new 접속
   → 상품 등록 폼이 보여야 함

3. 빈 폼 제출
   → 에러 메시지 표시

4. 정상적으로 입력 후 제출
   → 성공 알림 → 상세 페이지로 이동

5. Supabase Table Editor에서 데이터 확인
```

### Supabase에서 확인

Table Editor → products 테이블에서 새 데이터 확인!

---

## Day 2 마무리

### 오늘 배운 것

```
✅ Supabase 소개 및 설정
✅ 테이블 생성 및 RLS 설정
✅ 투두앱 DB 연동 (CRUD)
✅ Vercel 배포
✅ 마켓앱 설계
✅ 회원가입/로그인 구현
✅ 상품 등록 기능
```

### 완성된 기능

```
□ 투두앱 (DB 연동, 배포 완료!)
□ 마켓앱
  ├── ✅ DB 스키마 설계
  ├── ✅ 회원가입/로그인
  ├── ✅ 상품 등록
  ├── ⬜ 상품 목록
  ├── ⬜ 상품 상세
  ├── ⬜ 이미지 업로드
  ├── ⬜ 댓글 기능
  └── ⬜ 검색/필터
```

---

## 핵심 정리

```
✅ 인증된 사용자만 접근하게 하는 패턴
✅ 폼 상태 관리 (useState)
✅ 유효성 검사 (클라이언트)
✅ insert().select().single()로 추가 후 데이터 받기
✅ 로딩, 에러 상태 UI
```

---

## Day 3 예고

```
내일은 마켓앱을 완성합니다!

📚 상품 목록 & 상세 페이지
📚 이미지 업로드
📚 댓글 기능
📚 검색 & 필터
📚 최종 배포

마지막 날, 화이팅! 💪
```
