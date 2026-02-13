# Chapter 21. 찜하기 기능 만들기

이번 챕터에서는 **3개의 MCP를 모두 활용**해서
"찜하기" 기능을 처음부터 끝까지 만들어봅니다!

---

## 21.1 프로젝트 개요

### 만들 기능

```
✅ 로그인한 사용자가 상품을 찜할 수 있음
✅ 상품 카드에 하트 버튼 표시
✅ 찜한 상품은 하트가 채워짐
✅ 마이페이지에서 찜 목록 확인
✅ 찜 취소 가능
```

### 사용할 MCP

| MCP | 용도 |
|-----|------|
| Context7 | 다대다 관계 설계, 최신 문법 참조 |
| Supabase | 테이블 생성, 데이터 확인 |
| Playwright | 기능 테스트 |

### 개발 흐름

```
Step 1: 설계 (Context7)
    ↓
Step 2: DB 테이블 생성 (Supabase MCP)
    ↓
Step 3: 기능 구현
    ↓
Step 4: 데이터 확인 (Supabase MCP)
    ↓
Step 5: 테스트 (Playwright)
```

---

## 21.2 Step 1: 설계하기

### Context7로 설계 참조

**AI에게 요청:**

```
Supabase 문서에서 다대다 관계 테이블 설계 방법 찾아줘.
사용자가 여러 상품을 찜하고,
한 상품이 여러 사용자에게 찜당할 수 있어.
```

**Context7 확인 후:**

```
AI: Supabase 문서를 확인했습니다.

다대다 관계는 "중간 테이블"을 사용합니다.

현재 테이블:
- profiles (사용자)
- products (상품)

추가할 테이블:
- favorites (찜 목록) - 중간 테이블

favorites 테이블 구조:
- user_id (profiles 참조)
- product_id (products 참조)
- created_at

이 조합이 Primary Key가 되어
한 사용자가 같은 상품을 중복 찜하는 것을 방지합니다.
```

### 테이블 관계도

```
profiles ────┐
             │
             ├──→ favorites ←──┤
             │                  │
products ───┘                  └─→ 사용자별 찜 목록
```

---

## 21.3 Step 2: DB 테이블 생성

### Supabase MCP로 테이블 만들기

**AI에게 요청:**

```
Supabase에 favorites 테이블을 만들어줘.

조건:
- user_id: profiles 테이블의 id 참조
- product_id: products 테이블의 id 참조
- created_at: 찜한 시간
- (user_id, product_id)가 복합 Primary Key
- RLS 정책도 설정해줘
```

**AI가 생성하는 SQL:**

```sql
-- favorites 테이블 생성
CREATE TABLE favorites (
  user_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  product_id BIGINT REFERENCES products(id) ON DELETE CASCADE,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  PRIMARY KEY (user_id, product_id)
);

-- RLS 활성화
ALTER TABLE favorites ENABLE ROW LEVEL SECURITY;

-- 정책 1: 누구나 찜 목록 조회 가능 (상품의 찜 개수 표시용)
CREATE POLICY "Anyone can view favorites"
ON favorites FOR SELECT
TO public
USING (true);

-- 정책 2: 로그인한 사용자는 자신의 찜만 추가
CREATE POLICY "Users can add their own favorites"
ON favorites FOR INSERT
TO authenticated
WITH CHECK (auth.uid() = user_id);

-- 정책 3: 자신의 찜만 삭제 가능
CREATE POLICY "Users can delete their own favorites"
ON favorites FOR DELETE
TO authenticated
USING (auth.uid() = user_id);
```

### 테이블 생성 확인

**AI에게 요청:**

```
favorites 테이블이 제대로 만들어졌는지 확인해줘.
테이블 구조랑 RLS 정책 보여줘.
```

**확인 결과:**

```
AI: Supabase를 확인했습니다.

favorites 테이블 ✅

| 컬럼 | 타입 | 설명 |
|------|------|------|
| user_id | uuid | PK, profiles 참조 |
| product_id | int8 | PK, products 참조 |
| created_at | timestamptz | 기본값 now() |

RLS 정책:
- SELECT: 모두 허용 ✅
- INSERT: 본인만 ✅
- DELETE: 본인만 ✅

테이블이 정상적으로 생성되었습니다!
```

---

## 21.4 Step 3: 기능 구현

### 3-1. 찜하기/취소 Server Action

**AI에게 요청:**

```
Next.js 15 문서 참고해서
찜하기/취소 Server Action을 만들어줘.

Context7로 최신 문법 확인하고,
Supabase MCP로 favorites 테이블 구조 확인해서 만들어줘.
```

**생성된 코드:**

```typescript
// app/actions/favorites.ts
'use server'

import { createClient } from '@/lib/supabase/server'
import { revalidatePath } from 'next/cache'

export async function toggleFavorite(productId: number) {
  const supabase = await createClient()

  const { data: { user } } = await supabase.auth.getUser()

  if (!user) {
    return { error: '로그인이 필요합니다' }
  }

  // 이미 찜했는지 확인
  const { data: existing } = await supabase
    .from('favorites')
    .select()
    .eq('user_id', user.id)
    .eq('product_id', productId)
    .single()

  if (existing) {
    // 찜 취소
    const { error } = await supabase
      .from('favorites')
      .delete()
      .eq('user_id', user.id)
      .eq('product_id', productId)

    if (error) return { error: error.message }

    revalidatePath('/')
    return { success: true, action: 'removed' }
  } else {
    // 찜 추가
    const { error } = await supabase
      .from('favorites')
      .insert({
        user_id: user.id,
        product_id: productId
      })

    if (error) return { error: error.message }

    revalidatePath('/')
    return { success: true, action: 'added' }
  }
}

// 사용자의 찜 목록 가져오기
export async function getFavorites() {
  const supabase = await createClient()

  const { data: { user } } = await supabase.auth.getUser()

  if (!user) return []

  const { data } = await supabase
    .from('favorites')
    .select(`
      product_id,
      created_at,
      products (
        id,
        title,
        price,
        image_url
      )
    `)
    .eq('user_id', user.id)
    .order('created_at', { ascending: false })

  return data || []
}

// 특정 상품을 찜했는지 확인
export async function isFavorited(productId: number) {
  const supabase = await createClient()

  const { data: { user } } = await supabase.auth.getUser()

  if (!user) return false

  const { data } = await supabase
    .from('favorites')
    .select()
    .eq('user_id', user.id)
    .eq('product_id', productId)
    .single()

  return !!data
}
```

### 3-2. 찜 버튼 컴포넌트

**AI에게 요청:**

```
하트 버튼 컴포넌트 만들어줘.
찜했으면 빨간 하트, 안 했으면 빈 하트.
클릭하면 toggleFavorite 호출해줘.
```

**생성된 코드:**

```tsx
// app/components/FavoriteButton.tsx
'use client'

import { useState, useTransition } from 'react'
import { toggleFavorite } from '@/app/actions/favorites'

interface FavoriteButtonProps {
  productId: number
  initialFavorited: boolean
}

export default function FavoriteButton({
  productId,
  initialFavorited
}: FavoriteButtonProps) {
  const [isFavorited, setIsFavorited] = useState(initialFavorited)
  const [isPending, startTransition] = useTransition()

  const handleClick = () => {
    startTransition(async () => {
      const result = await toggleFavorite(productId)

      if (result.success) {
        setIsFavorited(result.action === 'added')
      } else {
        alert(result.error)
      }
    })
  }

  return (
    <button
      onClick={handleClick}
      disabled={isPending}
      className="p-2 rounded-full hover:bg-gray-100 transition"
      aria-label={isFavorited ? '찜 취소' : '찜하기'}
    >
      {isFavorited ? (
        <svg className="w-6 h-6 text-red-500 fill-current" viewBox="0 0 24 24">
          <path d="M12 21.35l-1.45-1.32C5.4 15.36 2 12.28 2 8.5 2 5.42 4.42 3 7.5 3c1.74 0 3.41.81 4.5 2.09C13.09 3.81 14.76 3 16.5 3 19.58 3 22 5.42 22 8.5c0 3.78-3.4 6.86-8.55 11.54L12 21.35z"/>
        </svg>
      ) : (
        <svg className="w-6 h-6 text-gray-400" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M4.318 6.318a4.5 4.5 0 000 6.364L12 20.364l7.682-7.682a4.5 4.5 0 00-6.364-6.364L12 7.636l-1.318-1.318a4.5 4.5 0 00-6.364 0z"/>
        </svg>
      )}
    </button>
  )
}
```

### 3-3. 상품 카드에 찜 버튼 추가

**AI에게 요청:**

```
ProductCard 컴포넌트에 FavoriteButton을 추가해줘.
우측 상단에 하트 버튼이 표시되도록 해줘.
```

**수정된 ProductCard:**

```tsx
// app/components/ProductCard.tsx
import Link from 'next/link'
import FavoriteButton from './FavoriteButton'

interface ProductCardProps {
  product: {
    id: number
    title: string
    price: number
    image_url: string
  }
  isFavorited?: boolean
  showFavoriteButton?: boolean
}

export default function ProductCard({
  product,
  isFavorited = false,
  showFavoriteButton = true
}: ProductCardProps) {
  return (
    <div className="relative border rounded-lg overflow-hidden hover:shadow-lg transition">
      {/* 찜 버튼 */}
      {showFavoriteButton && (
        <div className="absolute top-2 right-2 z-10">
          <FavoriteButton
            productId={product.id}
            initialFavorited={isFavorited}
          />
        </div>
      )}

      <Link href={`/products/${product.id}`}>
        <img
          src={product.image_url || '/placeholder.png'}
          alt={product.title}
          className="w-full h-48 object-cover"
        />
        <div className="p-4">
          <h3 className="font-semibold truncate">{product.title}</h3>
          <p className="text-lg font-bold text-orange-500">
            {product.price.toLocaleString()}원
          </p>
        </div>
      </Link>
    </div>
  )
}
```

### 3-4. 마이페이지 찜 목록

**AI에게 요청:**

```
마이페이지에 찜한 상품 목록을 보여주는 섹션 만들어줘.
```

**생성된 코드:**

```tsx
// app/mypage/favorites/page.tsx
import { getFavorites } from '@/app/actions/favorites'
import ProductCard from '@/app/components/ProductCard'

export default async function FavoritesPage() {
  const favorites = await getFavorites()

  return (
    <div className="container mx-auto px-4 py-8">
      <h1 className="text-2xl font-bold mb-6">찜한 상품</h1>

      {favorites.length === 0 ? (
        <div className="text-center py-12 text-gray-500">
          <p>찜한 상품이 없습니다.</p>
          <p className="mt-2">마음에 드는 상품에 하트를 눌러보세요!</p>
        </div>
      ) : (
        <div className="grid grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-4">
          {favorites.map((fav: any) => (
            <ProductCard
              key={fav.product_id}
              product={fav.products}
              isFavorited={true}
            />
          ))}
        </div>
      )}
    </div>
  )
}
```

---

## 21.5 Step 4: 데이터 확인

### Supabase MCP로 확인

**AI에게 요청:**

```
찜하기 기능을 테스트했는데,
favorites 테이블에 데이터가 제대로 저장됐는지 확인해줘.
```

**확인 결과:**

```
AI: favorites 테이블을 확인합니다.

현재 데이터:
| user_id | product_id | created_at |
|---------|------------|------------|
| abc-123 | 5 | 2024-01-15 10:30 |
| abc-123 | 12 | 2024-01-15 10:32 |
| def-456 | 5 | 2024-01-15 11:00 |

총 3개의 찜 데이터가 있습니다.
- user abc-123: 2개 상품 찜
- user def-456: 1개 상품 찜
- product 5: 2명이 찜

데이터가 정상적으로 저장되고 있습니다! ✅
```

### 문제 발생 시 디버깅

```
찜하기 버튼을 눌렀는데 저장이 안 돼.
favorites 테이블의 RLS 정책 확인해줘.
```

```
AI: RLS 정책을 확인합니다.

INSERT 정책:
- WITH CHECK (auth.uid() = user_id)

⚠️ 현재 로그인한 사용자의 user_id와
INSERT하려는 user_id가 일치해야 합니다.

코드에서 user_id를 제대로 넣고 있는지 확인해보세요.
```

---

## 21.6 Step 5: 테스트

### Playwright로 찜하기 테스트

**AI에게 요청:**

```
찜하기 기능을 Playwright로 테스트해줘:

1. test@test.com으로 로그인
2. 메인 페이지에서 첫 번째 상품의 찜 버튼 클릭
3. 하트가 빨간색으로 변하는지 확인
4. /mypage/favorites 페이지로 이동
5. 찜한 상품이 목록에 있는지 확인
6. 찜 취소 버튼 클릭
7. 목록에서 사라지는지 확인
```

**테스트 결과:**

```
AI: 찜하기 기능을 테스트합니다.

1. 로그인 완료 ✅
2. 메인 페이지 접속 ✅
3. 첫 번째 상품 찜 버튼 클릭 ✅
4. 하트 색상 변경 확인:
   - Before: 회색 (빈 하트)
   - After: 빨간색 (채워진 하트) ✅
5. /mypage/favorites 이동 ✅
6. 찜 목록 확인:
   - "맥북 프로" 상품 발견 ✅
7. 찜 취소 버튼 클릭 ✅
8. 목록 갱신 확인:
   - "맥북 프로" 사라짐 ✅

테스트 결과: 모두 성공! ✅
```

### 전체 흐름 테스트

```
찜하기 전체 시나리오 테스트해줘:
1. 비로그인 상태에서 찜 버튼 클릭 → 로그인 안내
2. 로그인 후 찜하기
3. 새로고침 후에도 찜 상태 유지
4. 다른 브라우저에서도 찜 목록 동기화
```

---

## 21.7 완성 코드 정리

### 파일 구조

```
app/
├── actions/
│   └── favorites.ts          # 찜 관련 Server Actions
├── components/
│   ├── FavoriteButton.tsx    # 찜 버튼 컴포넌트
│   └── ProductCard.tsx       # 수정됨 (찜 버튼 추가)
├── mypage/
│   └── favorites/
│       └── page.tsx          # 찜 목록 페이지
└── ...
```

### DB 테이블

```
favorites
├── user_id (PK, FK → profiles)
├── product_id (PK, FK → products)
└── created_at
```

### RLS 정책

```
- SELECT: 모두 허용
- INSERT: 본인만 (auth.uid() = user_id)
- DELETE: 본인만 (auth.uid() = user_id)
```

---

## 21.8 더 해보기

### 찜 개수 표시

```
상품 카드에 "❤️ 5" 이렇게 찜 개수도 표시해줘
```

### 찜 많은 순 정렬

```
메인 페이지에서 "인기순" 버튼 누르면
찜 많은 상품이 먼저 나오게 해줘
```

### 알림 기능

```
내가 찜한 상품 가격이 내려가면
알림이 오도록 해줘
```

---

## 핵심 정리

```
✅ 기능 개발 = Context7 + Supabase MCP + Playwright 조합

개발 흐름:
1. Context7로 설계 참조 (다대다 관계)
2. Supabase MCP로 테이블 생성
3. 코드 구현 (최신 문법 참조)
4. Supabase MCP로 데이터 확인
5. Playwright로 전체 테스트

이 패턴을 익히면 어떤 기능이든 만들 수 있어요!
```

---

## Part 4 마무리

축하합니다! 🎉

Part 4에서 배운 것:

```
✅ Chapter 18: MCP로 디버깅하기
✅ Chapter 19: 문서 참조하며 코딩하기
✅ Chapter 20: E2E 테스트 자동화
✅ Chapter 21: MCP 조합해서 새 기능 개발
```

이제 MCP를 활용해서 **더 스마트하게** 개발할 수 있어요!

---

## 다음 단계

1. **직접 만들어보기**
   - 찜하기 외에 다른 기능도 MCP로 개발해보세요
   - 예: 팔로우, 알림, 결제 기능

2. **MCP 더 탐험하기**
   - Smithery에서 다양한 MCP 찾아보기
   - 내 프로젝트에 맞는 MCP 연결해보기

3. **나만의 MCP 만들기**
   - Part 3의 Chapter 17 참고
   - 우리 팀/회사 전용 MCP 개발

**화이팅! 🚀**
