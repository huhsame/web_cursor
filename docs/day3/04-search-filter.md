# Day 3 - 4교시: 검색 & 필터 기능

## 학습 목표
- 검색 기능을 구현한다
- 정렬 기능을 추가한다
- URL 쿼리 파라미터를 활용한다
- 디바운싱을 이해한다

---

## 1. 완성 모습

```
┌───────────────────────────────────────────────────────────────┐
│  🥕 미니마켓                               [로그인] [회원가입]  │
├───────────────────────────────────────────────────────────────┤
│  ┌─────────────────────────────────────────────┐              │
│  │ 🔍 검색어를 입력하세요...                    │              │
│  └─────────────────────────────────────────────┘              │
│                                                               │
│  [전체] [가전] [의류] [도서] ...    정렬: [최신순 ▼]           │
├───────────────────────────────────────────────────────────────┤
│                                                               │
│  "아이폰" 검색 결과 3건                                        │
│                                                               │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐                    │
│  │ 아이폰 13 │  │아이폰 케이스│  │ 아이폰 충전기│                │
│  │ 500,000원│  │  10,000원 │  │  15,000원  │                  │
│  └──────────┘  └──────────┘  └──────────┘                    │
│                                                               │
└───────────────────────────────────────────────────────────────┘
```

---

## 2. 검색 기능 구현

### 검색 UI 추가

```tsx
'use client';

import { useState } from 'react';

export default function HomePage() {
  const [searchQuery, setSearchQuery] = useState('');
  // ... 기존 state

  return (
    <div>
      {/* 검색창 */}
      <div className="mb-6">
        <div className="relative">
          <input
            type="text"
            value={searchQuery}
            onChange={(e) => setSearchQuery(e.target.value)}
            placeholder="검색어를 입력하세요..."
            className="w-full border border-gray-300 rounded-lg pl-10 pr-4 py-3 focus:outline-none focus:ring-2 focus:ring-orange-500"
          />
          <span className="absolute left-3 top-1/2 -translate-y-1/2 text-gray-400">
            🔍
          </span>
        </div>
      </div>

      {/* ... 카테고리, 상품 목록 */}
    </div>
  );
}
```

### Supabase 검색 쿼리

```tsx
const fetchProducts = async () => {
  let query = supabase
    .from('products')
    .select('*')
    .order('created_at', { ascending: false });

  // 카테고리 필터
  if (selectedCategory !== 'all') {
    query = query.eq('category', selectedCategory);
  }

  // 검색어 필터 (제목에서 검색)
  if (searchQuery.trim()) {
    query = query.ilike('title', `%${searchQuery}%`);
  }

  const { data, error } = await query;
  // ...
};
```

### ilike 설명

```tsx
// ilike = 대소문자 구분 없이 패턴 매칭
.ilike('title', `%${searchQuery}%`)

// % = 와일드카드 (아무 문자나)
// %아이폰% = "아이폰"을 포함하는 모든 것

// 예시:
// "아이폰 13 프로" ✅
// "중고 아이폰" ✅
// "갤럭시" ❌
```

---

## 3. 디바운싱 적용

### 문제: 타이핑할 때마다 API 호출

```
사용자 입력: "아이폰"
API 호출: 아 → 아이 → 아이폰 (3번 호출!)

→ 서버 부하, 느린 UI
```

### 해결: 디바운싱

```
타이핑 멈추고 300ms 후에 검색
→ 최종 입력값으로 1번만 호출
```

### 구현

```tsx
import { useState, useEffect } from 'react';

export default function HomePage() {
  const [searchQuery, setSearchQuery] = useState('');
  const [debouncedQuery, setDebouncedQuery] = useState('');

  // 디바운싱
  useEffect(() => {
    const timer = setTimeout(() => {
      setDebouncedQuery(searchQuery);
    }, 300);  // 300ms 대기

    // 클린업: 새 입력이 들어오면 이전 타이머 취소
    return () => clearTimeout(timer);
  }, [searchQuery]);

  // 검색 실행 (디바운싱된 값 사용)
  useEffect(() => {
    fetchProducts();
  }, [debouncedQuery, selectedCategory]);

  const fetchProducts = async () => {
    // debouncedQuery 사용
    if (debouncedQuery.trim()) {
      query = query.ilike('title', `%${debouncedQuery}%`);
    }
    // ...
  };

  return (
    <input
      value={searchQuery}
      onChange={(e) => setSearchQuery(e.target.value)}
    />
  );
}
```

---

## 4. 정렬 기능

### 정렬 옵션

```tsx
const SORT_OPTIONS = [
  { id: 'latest', name: '최신순', column: 'created_at', ascending: false },
  { id: 'oldest', name: '오래된순', column: 'created_at', ascending: true },
  { id: 'price_low', name: '낮은가격순', column: 'price', ascending: true },
  { id: 'price_high', name: '높은가격순', column: 'price', ascending: false },
];
```

### 정렬 UI

```tsx
const [sortOption, setSortOption] = useState('latest');

// JSX
<select
  value={sortOption}
  onChange={(e) => setSortOption(e.target.value)}
  className="border border-gray-300 rounded-lg px-4 py-2"
>
  {SORT_OPTIONS.map((option) => (
    <option key={option.id} value={option.id}>
      {option.name}
    </option>
  ))}
</select>
```

### 쿼리에 적용

```tsx
const fetchProducts = async () => {
  // 현재 정렬 옵션 찾기
  const currentSort = SORT_OPTIONS.find(o => o.id === sortOption)!;

  let query = supabase
    .from('products')
    .select('*')
    .order(currentSort.column, { ascending: currentSort.ascending });

  // ... 필터
};
```

---

## 5. 전체 코드 (홈페이지)

### app/page.tsx

```tsx
'use client';

import { useEffect, useState } from 'react';
import { supabase } from '@/lib/supabase';
import ProductCard from './components/ProductCard';
import { Product } from '@/types';

const CATEGORIES = [
  { id: 'all', name: '전체' },
  { id: 'electronics', name: '가전' },
  { id: 'clothing', name: '의류' },
  { id: 'books', name: '도서' },
  { id: 'sports', name: '스포츠' },
  { id: 'furniture', name: '가구' },
  { id: 'etc', name: '기타' },
];

const SORT_OPTIONS = [
  { id: 'latest', name: '최신순', column: 'created_at', ascending: false },
  { id: 'oldest', name: '오래된순', column: 'created_at', ascending: true },
  { id: 'price_low', name: '낮은가격순', column: 'price', ascending: true },
  { id: 'price_high', name: '높은가격순', column: 'price', ascending: false },
];

export default function HomePage() {
  const [products, setProducts] = useState<Product[]>([]);
  const [loading, setLoading] = useState(true);
  const [selectedCategory, setSelectedCategory] = useState('all');
  const [searchQuery, setSearchQuery] = useState('');
  const [debouncedQuery, setDebouncedQuery] = useState('');
  const [sortOption, setSortOption] = useState('latest');

  // 디바운싱
  useEffect(() => {
    const timer = setTimeout(() => {
      setDebouncedQuery(searchQuery);
    }, 300);
    return () => clearTimeout(timer);
  }, [searchQuery]);

  // 데이터 조회
  useEffect(() => {
    fetchProducts();
  }, [selectedCategory, debouncedQuery, sortOption]);

  const fetchProducts = async () => {
    setLoading(true);

    const currentSort = SORT_OPTIONS.find(o => o.id === sortOption)!;

    let query = supabase
      .from('products')
      .select('*')
      .order(currentSort.column, { ascending: currentSort.ascending });

    // 카테고리 필터
    if (selectedCategory !== 'all') {
      query = query.eq('category', selectedCategory);
    }

    // 검색 필터
    if (debouncedQuery.trim()) {
      query = query.ilike('title', `%${debouncedQuery}%`);
    }

    const { data, error } = await query;

    if (error) {
      console.error('조회 에러:', error);
    } else {
      setProducts(data || []);
    }

    setLoading(false);
  };

  return (
    <div className="min-h-screen bg-gray-100">
      <div className="max-w-6xl mx-auto px-4 py-8">
        {/* 검색창 */}
        <div className="mb-6">
          <div className="relative">
            <input
              type="text"
              value={searchQuery}
              onChange={(e) => setSearchQuery(e.target.value)}
              placeholder="검색어를 입력하세요..."
              className="w-full border border-gray-300 rounded-lg pl-10 pr-4 py-3 focus:outline-none focus:ring-2 focus:ring-orange-500"
            />
            <span className="absolute left-3 top-1/2 -translate-y-1/2 text-gray-400">
              🔍
            </span>
            {searchQuery && (
              <button
                onClick={() => setSearchQuery('')}
                className="absolute right-3 top-1/2 -translate-y-1/2 text-gray-400 hover:text-gray-600"
              >
                ✕
              </button>
            )}
          </div>
        </div>

        {/* 필터 & 정렬 */}
        <div className="flex flex-wrap items-center justify-between gap-4 mb-6">
          {/* 카테고리 */}
          <div className="flex gap-2 overflow-x-auto pb-2">
            {CATEGORIES.map((cat) => (
              <button
                key={cat.id}
                onClick={() => setSelectedCategory(cat.id)}
                className={`px-4 py-2 rounded-full whitespace-nowrap transition ${
                  selectedCategory === cat.id
                    ? 'bg-orange-500 text-white'
                    : 'bg-white text-gray-700 hover:bg-gray-100'
                }`}
              >
                {cat.name}
              </button>
            ))}
          </div>

          {/* 정렬 */}
          <select
            value={sortOption}
            onChange={(e) => setSortOption(e.target.value)}
            className="border border-gray-300 rounded-lg px-4 py-2 bg-white"
          >
            {SORT_OPTIONS.map((option) => (
              <option key={option.id} value={option.id}>
                {option.name}
              </option>
            ))}
          </select>
        </div>

        {/* 검색 결과 표시 */}
        {debouncedQuery && (
          <p className="mb-4 text-gray-600">
            "{debouncedQuery}" 검색 결과 {products.length}건
          </p>
        )}

        {/* 상품 목록 */}
        {loading ? (
          <div className="text-center py-20">
            <p className="text-gray-500">로딩 중...</p>
          </div>
        ) : products.length === 0 ? (
          <div className="text-center py-20">
            <p className="text-gray-500">
              {debouncedQuery
                ? '검색 결과가 없습니다.'
                : '등록된 상품이 없습니다.'}
            </p>
          </div>
        ) : (
          <div className="grid grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-4">
            {products.map((product) => (
              <ProductCard key={product.id} product={product} />
            ))}
          </div>
        )}
      </div>
    </div>
  );
}
```

---

## 6. URL 쿼리 파라미터 (선택)

### 왜 필요한가?

```
현재: 새로고침하면 검색/필터 초기화

URL 파라미터 사용:
/products?search=아이폰&category=electronics&sort=price_low

→ 새로고침해도 유지
→ 링크 공유 가능
```

### useSearchParams 사용

```tsx
'use client';

import { useSearchParams, useRouter } from 'next/navigation';

export default function HomePage() {
  const searchParams = useSearchParams();
  const router = useRouter();

  // URL에서 초기값 읽기
  const initialCategory = searchParams.get('category') || 'all';
  const initialSearch = searchParams.get('search') || '';
  const initialSort = searchParams.get('sort') || 'latest';

  const [selectedCategory, setSelectedCategory] = useState(initialCategory);
  const [searchQuery, setSearchQuery] = useState(initialSearch);
  const [sortOption, setSortOption] = useState(initialSort);

  // URL 업데이트 함수
  const updateURL = (params: Record<string, string>) => {
    const current = new URLSearchParams(searchParams.toString());

    Object.entries(params).forEach(([key, value]) => {
      if (value && value !== 'all' && value !== 'latest') {
        current.set(key, value);
      } else {
        current.delete(key);
      }
    });

    router.push(`/?${current.toString()}`);
  };

  // 카테고리 변경
  const handleCategoryChange = (category: string) => {
    setSelectedCategory(category);
    updateURL({ category, search: debouncedQuery, sort: sortOption });
  };

  // ...
}
```

---

## 7. 검색 초기화 버튼

```tsx
const clearSearch = () => {
  setSearchQuery('');
  setSelectedCategory('all');
  setSortOption('latest');
};

// 검색 결과 영역
{(debouncedQuery || selectedCategory !== 'all') && (
  <div className="flex items-center justify-between mb-4">
    <p className="text-gray-600">
      {debouncedQuery && `"${debouncedQuery}" `}
      검색 결과 {products.length}건
    </p>
    <button
      onClick={clearSearch}
      className="text-sm text-orange-500 hover:underline"
    >
      초기화
    </button>
  </div>
)}
```

---

## 8. 검색 제목 + 설명 동시 검색

### or 조건 사용

```tsx
// 제목 또는 설명에서 검색
if (debouncedQuery.trim()) {
  query = query.or(
    `title.ilike.%${debouncedQuery}%,description.ilike.%${debouncedQuery}%`
  );
}
```

---

## 9. 검색어 하이라이트 (선택)

### 검색어 강조 함수

```tsx
const highlightText = (text: string, query: string) => {
  if (!query.trim()) return text;

  const regex = new RegExp(`(${query})`, 'gi');
  const parts = text.split(regex);

  return parts.map((part, i) =>
    regex.test(part) ? (
      <mark key={i} className="bg-yellow-200">{part}</mark>
    ) : (
      part
    )
  );
};

// 사용
<h3>{highlightText(product.title, searchQuery)}</h3>
```

---

## 10. 테스트하기

### 테스트 체크리스트

```
□ 검색어 입력 → 결과 필터링
□ 디바운싱 동작 (빠른 타이핑 시 마지막만 검색)
□ 검색 초기화 (X 버튼)
□ 카테고리 + 검색 조합
□ 정렬 변경 → 순서 변경
□ 검색 결과 0건 시 메시지
□ 검색어 있을 때 결과 건수 표시
```

---

## 핵심 정리

```
✅ ilike = 대소문자 무시 패턴 검색
✅ %검색어% = 포함 검색
✅ 디바운싱 = 타이핑 후 잠시 기다렸다가 검색
✅ order(column, {ascending}) = 정렬
✅ 여러 필터 조합 가능
```

---

## 다음 시간 예고

> 마지막 시간!
> 마켓앱을 Vercel에 최종 배포하고,
> 지금까지 배운 내용을 정리합니다.
> 다음 학습 방향도 안내해드립니다!
