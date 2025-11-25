# Day 3 - 1교시: 상품 목록 & 상세 페이지

## 학습 목표
- 홈페이지에 상품 목록을 표시한다
- 상품 카드 컴포넌트를 만든다
- 동적 라우팅으로 상세 페이지를 구현한다
- 수정/삭제 기능을 구현한다

---

## 1. 상품 목록 페이지

### 완성 모습

```
┌─────────────────────────────────────────────────────────────┐
│  🥕 미니마켓                            [로그인] [회원가입]  │
├─────────────────────────────────────────────────────────────┤
│  [전체] [가전] [의류] [도서] [스포츠] [가구] [기타]          │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │  [이미지]     │  │  [이미지]     │  │  [이미지]     │      │
│  │  아이폰 13    │  │  나이키 운동화 │  │  자바스크립트  │      │
│  │  500,000원   │  │   30,000원    │  │   10,000원   │      │
│  │  가전        │  │   의류        │  │   도서        │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
│                                                             │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │  ...         │  │  ...         │  │  ...         │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 2. 상품 카드 컴포넌트

### components/ProductCard.tsx

```tsx
import Link from 'next/link';
import { Product } from '@/types';

interface ProductCardProps {
  product: Product;
}

// 가격 포맷팅 함수
const formatPrice = (price: number) => {
  return price.toLocaleString('ko-KR');
};

// 카테고리 한글화
const getCategoryName = (category: string) => {
  const categories: Record<string, string> = {
    electronics: '가전',
    clothing: '의류',
    books: '도서',
    sports: '스포츠',
    furniture: '가구',
    etc: '기타',
  };
  return categories[category] || '기타';
};

export default function ProductCard({ product }: ProductCardProps) {
  return (
    <Link href={`/products/${product.id}`}>
      <div className="bg-white rounded-lg shadow-md overflow-hidden hover:shadow-lg transition cursor-pointer">
        {/* 이미지 영역 */}
        <div className="aspect-square bg-gray-200 relative">
          {product.image_url ? (
            <img
              src={product.image_url}
              alt={product.title}
              className="w-full h-full object-cover"
            />
          ) : (
            <div className="w-full h-full flex items-center justify-center text-gray-400">
              <span className="text-4xl">📦</span>
            </div>
          )}
        </div>

        {/* 정보 영역 */}
        <div className="p-4">
          <h3 className="font-medium text-gray-900 truncate">
            {product.title}
          </h3>
          <p className="text-lg font-bold text-orange-500 mt-1">
            {formatPrice(product.price)}원
          </p>
          <p className="text-sm text-gray-500 mt-1">
            {getCategoryName(product.category)}
          </p>
        </div>
      </div>
    </Link>
  );
}
```

---

## 3. 홈페이지 (상품 목록)

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

export default function HomePage() {
  const [products, setProducts] = useState<Product[]>([]);
  const [loading, setLoading] = useState(true);
  const [selectedCategory, setSelectedCategory] = useState('all');

  // 상품 목록 조회
  useEffect(() => {
    fetchProducts();
  }, [selectedCategory]);

  const fetchProducts = async () => {
    setLoading(true);

    let query = supabase
      .from('products')
      .select('*')
      .order('created_at', { ascending: false });

    // 카테고리 필터
    if (selectedCategory !== 'all') {
      query = query.eq('category', selectedCategory);
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
        {/* 카테고리 필터 */}
        <div className="flex gap-2 mb-6 overflow-x-auto pb-2">
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

        {/* 상품 목록 */}
        {loading ? (
          <div className="text-center py-20">
            <p className="text-gray-500">로딩 중...</p>
          </div>
        ) : products.length === 0 ? (
          <div className="text-center py-20">
            <p className="text-gray-500">등록된 상품이 없습니다.</p>
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

## 4. 그리드 레이아웃 이해하기

### Tailwind 그리드

```css
grid grid-cols-2       /* 기본: 2열 */
md:grid-cols-3         /* 중간 화면: 3열 */
lg:grid-cols-4         /* 큰 화면: 4열 */
gap-4                  /* 간격 */
```

### 반응형 설명

```
모바일 (< 768px):     2열
┌─────┐ ┌─────┐
│     │ │     │
└─────┘ └─────┘

태블릿 (768px ~):     3열
┌─────┐ ┌─────┐ ┌─────┐
│     │ │     │ │     │
└─────┘ └─────┘ └─────┘

데스크탑 (1024px ~):  4열
┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐
│     │ │     │ │     │ │     │
└─────┘ └─────┘ └─────┘ └─────┘
```

---

## 5. 동적 라우팅 이해하기

### 쉬운 설명

> 상품이 100개면 페이지를 100개 만들어야 할까요?
> 아니요!
>
> `/products/1`, `/products/2`, `/products/3`...
> 이런 URL을 **하나의 페이지**로 처리할 수 있어요.
>
> `[id]`라는 특별한 폴더명을 사용합니다.

### 폴더 구조

```
app/products/
├── page.tsx          → /products (목록)
├── new/
│   └── page.tsx      → /products/new (등록)
└── [id]/             ← 대괄호! 동적 라우트
    └── page.tsx      → /products/1, /products/2, ...
```

### 공식적인 설명
```
Next.js App Router에서 동적 세그먼트는
폴더명을 대괄호로 감싸서 정의합니다.
[id]는 URL의 해당 부분을 params로 전달받습니다.
예: /products/123 → params.id = "123"
```

---

## 6. 상품 상세 페이지

### app/products/[id]/page.tsx

```tsx
'use client';

import { useEffect, useState } from 'react';
import { supabase } from '@/lib/supabase';
import { useParams, useRouter } from 'next/navigation';
import { Product } from '@/types';
import type { User } from '@supabase/supabase-js';

// 가격 포맷팅
const formatPrice = (price: number) => {
  return price.toLocaleString('ko-KR');
};

// 날짜 포맷팅
const formatDate = (dateString: string) => {
  const date = new Date(dateString);
  return date.toLocaleDateString('ko-KR', {
    year: 'numeric',
    month: 'long',
    day: 'numeric',
  });
};

export default function ProductDetailPage() {
  const params = useParams();
  const router = useRouter();
  const productId = params.id as string;

  const [product, setProduct] = useState<Product | null>(null);
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetchProduct();
    getUser();
  }, [productId]);

  const fetchProduct = async () => {
    const { data, error } = await supabase
      .from('products')
      .select('*')
      .eq('id', productId)
      .single();

    if (error) {
      console.error('조회 에러:', error);
      setProduct(null);
    } else {
      setProduct(data);
    }
    setLoading(false);
  };

  const getUser = async () => {
    const { data: { user } } = await supabase.auth.getUser();
    setUser(user);
  };

  // 삭제 처리
  const handleDelete = async () => {
    if (!confirm('정말 삭제하시겠습니까?')) return;

    const { error } = await supabase
      .from('products')
      .delete()
      .eq('id', productId);

    if (error) {
      alert('삭제에 실패했습니다.');
      return;
    }

    alert('삭제되었습니다.');
    router.push('/');
  };

  // 작성자인지 확인
  const isOwner = user && product && user.id === product.user_id;

  if (loading) {
    return (
      <div className="min-h-screen flex items-center justify-center">
        <p>로딩 중...</p>
      </div>
    );
  }

  if (!product) {
    return (
      <div className="min-h-screen flex items-center justify-center">
        <div className="text-center">
          <p className="text-gray-500 mb-4">상품을 찾을 수 없습니다.</p>
          <button
            onClick={() => router.push('/')}
            className="text-orange-500 hover:underline"
          >
            홈으로 돌아가기
          </button>
        </div>
      </div>
    );
  }

  return (
    <div className="min-h-screen bg-gray-100 py-8">
      <div className="max-w-4xl mx-auto px-4">
        <div className="bg-white rounded-lg shadow-md overflow-hidden">
          {/* 이미지 */}
          <div className="aspect-video bg-gray-200 relative">
            {product.image_url ? (
              <img
                src={product.image_url}
                alt={product.title}
                className="w-full h-full object-contain"
              />
            ) : (
              <div className="w-full h-full flex items-center justify-center text-gray-400">
                <span className="text-8xl">📦</span>
              </div>
            )}
          </div>

          {/* 정보 */}
          <div className="p-6">
            {/* 카테고리 */}
            <span className="inline-block bg-gray-100 text-gray-600 text-sm px-3 py-1 rounded-full">
              {product.category}
            </span>

            {/* 제목 */}
            <h1 className="text-2xl font-bold mt-3">{product.title}</h1>

            {/* 가격 */}
            <p className="text-3xl font-bold text-orange-500 mt-2">
              {formatPrice(product.price)}원
            </p>

            {/* 메타 정보 */}
            <div className="flex items-center gap-4 mt-4 text-sm text-gray-500">
              <span>{product.user_email}</span>
              <span>•</span>
              <span>{formatDate(product.created_at)}</span>
            </div>

            {/* 구분선 */}
            <hr className="my-6" />

            {/* 설명 */}
            <div className="prose max-w-none">
              <h3 className="text-lg font-medium mb-2">상품 설명</h3>
              <p className="text-gray-700 whitespace-pre-wrap">
                {product.description || '설명이 없습니다.'}
              </p>
            </div>

            {/* 작성자 버튼 */}
            {isOwner && (
              <div className="flex gap-3 mt-8">
                <button
                  onClick={() => router.push(`/products/${productId}/edit`)}
                  className="flex-1 border border-orange-500 text-orange-500 py-3 rounded-lg hover:bg-orange-50 transition"
                >
                  수정
                </button>
                <button
                  onClick={handleDelete}
                  className="flex-1 border border-red-500 text-red-500 py-3 rounded-lg hover:bg-red-50 transition"
                >
                  삭제
                </button>
              </div>
            )}
          </div>
        </div>

        {/* 댓글 영역 (나중에 추가) */}
        <div className="mt-8">
          {/* 여기에 댓글 컴포넌트가 들어갈 예정 */}
        </div>
      </div>
    </div>
  );
}
```

---

## 7. useParams 이해하기

### 사용법

```tsx
import { useParams } from 'next/navigation';

export default function ProductDetailPage() {
  const params = useParams();

  // /products/123 접속 시
  console.log(params);  // { id: '123' }
  console.log(params.id);  // '123'
}
```

### 주의: 문자열로 전달됨!

```tsx
const productId = params.id as string;  // '123' (문자열)

// 숫자가 필요하면 변환
const numericId = parseInt(productId);  // 123 (숫자)
```

---

## 8. 조건부 렌더링 패턴

### 로딩/에러/정상 3단계

```tsx
// 1. 로딩 중
if (loading) {
  return <p>로딩 중...</p>;
}

// 2. 데이터 없음 (에러)
if (!product) {
  return <p>상품을 찾을 수 없습니다.</p>;
}

// 3. 정상
return <div>{product.title}</div>;
```

### 작성자 확인

```tsx
const isOwner = user && product && user.id === product.user_id;

{isOwner && (
  <div>
    <button>수정</button>
    <button>삭제</button>
  </div>
)}
```

---

## 9. 삭제 전 확인

### window.confirm 사용

```tsx
const handleDelete = async () => {
  // 확인 대화상자
  if (!confirm('정말 삭제하시겠습니까?')) {
    return;  // 취소 클릭 시 함수 종료
  }

  // 확인 클릭 시 삭제 실행
  await supabase.from('products').delete().eq('id', productId);
};
```

---

## 10. 테스트하기

### 테스트 체크리스트

```
□ 홈페이지에서 상품 목록 표시
□ 카테고리 버튼 클릭 시 필터링
□ 상품 카드 클릭 → 상세 페이지 이동
□ 상세 페이지에서 정보 정상 표시
□ 비로그인 시 수정/삭제 버튼 안 보임
□ 다른 사람 글에 수정/삭제 버튼 안 보임
□ 내 글에서 삭제 기능 동작
□ 없는 상품 ID 접속 시 에러 처리
```

---

## 핵심 정리

```
✅ ProductCard 컴포넌트로 재사용
✅ grid grid-cols-N 으로 반응형 레이아웃
✅ [id] 폴더 = 동적 라우팅
✅ useParams()로 URL 파라미터 가져오기
✅ 로딩/에러/정상 3단계 조건부 렌더링
✅ isOwner 패턴으로 작성자만 수정/삭제
```

---

## 다음 시간 예고

> 상품에 이미지를 추가해봅니다!
> Supabase Storage를 사용해서
> 파일 업로드 기능을 구현합니다.
