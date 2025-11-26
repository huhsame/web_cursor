# Day 2 - 4교시: 마켓앱 설계 & DB 스키마

## 학습 목표
- 프로젝트 기획의 중요성을 이해한다
- 마켓앱의 기능을 정의한다
- 데이터베이스 스키마를 설계한다
- Supabase에 테이블을 생성한다

---

## 1. 무엇을 만들 것인가?

### 당근마켓 클론!

```
┌─────────────────────────────────────────────────────────┐
│  🥕 미니마켓                    [로그인] [회원가입]      │
├─────────────────────────────────────────────────────────┤
│  [전체] [가전] [의류] [도서] [기타]                      │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐     │
│  │   📱        │  │   👟        │  │   📚        │     │
│  │  아이폰 13   │  │  나이키 운동화 │  │   자바스크립트 │ │
│  │  50만원     │  │   3만원      │  │   1만원      │     │
│  │  서울시     │  │   부산시     │  │   대전시     │     │
│  └─────────────┘  └─────────────┘  └─────────────┘     │
│                                                         │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐     │
│  │   🎸        │  │   🎮        │  │   🪑        │     │
│  │  어쿠스틱기타 │  │  플스5      │  │  사무용의자   │     │
│  │  20만원     │  │  40만원     │  │   5만원      │     │
│  │  인천시     │  │  서울시     │  │   광주시     │     │
│  └─────────────┘  └─────────────┘  └─────────────┘     │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

## 2. 기능 정의하기

### 핵심 기능 목록

```
1. 회원 기능
   ├── 회원가입
   ├── 로그인
   └── 로그아웃

2. 상품 기능
   ├── 상품 등록
   ├── 상품 목록 보기
   ├── 상품 상세 보기
   ├── 상품 수정
   └── 상품 삭제

3. 댓글 기능
   ├── 댓글 작성
   ├── 댓글 목록 보기
   └── 댓글 삭제

4. 추가 기능
   ├── 카테고리 필터
   ├── 검색
   └── 이미지 업로드
```

### 사용자 시나리오

```
[비회원]
홈페이지 접속 → 상품 목록 보기 → 상품 클릭 → 상세 보기
                                    └→ 구매하려면 로그인 필요!

[회원]
로그인 → 상품 등록하기 → 내 상품 관리
     → 다른 상품 보기 → 댓글 남기기
```

---

## 3. 데이터베이스 설계

### DB 설계가 중요한 이유

> 집을 짓기 전에 설계도를 그리는 것처럼,
> 앱을 만들기 전에 데이터 구조를 먼저 정합니다.
>
> 잘못된 설계 = 나중에 대공사!

### 필요한 테이블

```
1. users (사용자)
   - Supabase Auth가 자동 관리

2. products (상품)
   - 상품 정보 저장

3. comments (댓글)
   - 상품에 대한 댓글
```

---

## 4. 테이블 스키마 설계

### products 테이블

```
products (상품)
┌─────────────┬──────────────┬─────────────────────────────┐
│   컬럼명     │    타입      │           설명              │
├─────────────┼──────────────┼─────────────────────────────┤
│ id          │ int8         │ 고유 ID (자동 증가)          │
│ created_at  │ timestamptz  │ 생성 시간                    │
│ title       │ text         │ 상품 제목                    │
│ description │ text         │ 상품 설명                    │
│ price       │ int4         │ 가격                        │
│ category    │ text         │ 카테고리                     │
│ image_url   │ text         │ 이미지 URL                   │
│ user_id     │ uuid         │ 작성자 ID (users 참조)       │
│ user_email  │ text         │ 작성자 이메일                │
└─────────────┴──────────────┴─────────────────────────────┘
```

### comments 테이블

```
comments (댓글)
┌─────────────┬──────────────┬─────────────────────────────┐
│   컬럼명     │    타입      │           설명              │
├─────────────┼──────────────┼─────────────────────────────┤
│ id          │ int8         │ 고유 ID (자동 증가)          │
│ created_at  │ timestamptz  │ 생성 시간                    │
│ content     │ text         │ 댓글 내용                    │
│ product_id  │ int8         │ 상품 ID (products 참조)      │
│ user_id     │ uuid         │ 작성자 ID                    │
│ user_email  │ text         │ 작성자 이메일                │
└─────────────┴──────────────┴─────────────────────────────┘
```

### 테이블 관계도

```
┌──────────────┐
│    users     │  ← Supabase Auth 관리
│   (auth)     │
└──────┬───────┘
       │
       │ 1:N (한 유저가 여러 상품)
       │
       ▼
┌──────────────┐       ┌──────────────┐
│   products   │──────▶│   comments   │
│    (상품)    │  1:N  │    (댓글)    │
└──────────────┘ (한 상품에 여러 댓글)
```

---

## 5. Supabase에 테이블 생성

### 방법 1: Table Editor (GUI)

1. Supabase 대시보드 → Table Editor
2. "Create a new table"

### 방법 2: SQL Editor (추천)

SQL을 직접 실행하는 것이 더 정확합니다.

### products 테이블 생성 SQL

```sql
-- products 테이블 생성
CREATE TABLE products (
  id BIGSERIAL PRIMARY KEY,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  title TEXT NOT NULL,
  description TEXT,
  price INTEGER NOT NULL,
  category TEXT DEFAULT '기타',
  image_url TEXT,
  user_id UUID REFERENCES auth.users(id),
  user_email TEXT
);

-- RLS 활성화
ALTER TABLE products ENABLE ROW LEVEL SECURITY;

-- 정책: 모든 사람이 읽기 가능
CREATE POLICY "Anyone can read products"
ON products FOR SELECT
USING (true);

-- 정책: 로그인한 사용자만 생성 가능
CREATE POLICY "Authenticated users can insert products"
ON products FOR INSERT
WITH CHECK (auth.uid() = user_id);

-- 정책: 작성자만 수정 가능
CREATE POLICY "Users can update own products"
ON products FOR UPDATE
USING (auth.uid() = user_id);

-- 정책: 작성자만 삭제 가능
CREATE POLICY "Users can delete own products"
ON products FOR DELETE
USING (auth.uid() = user_id);
```

### comments 테이블 생성 SQL

```sql
-- comments 테이블 생성
CREATE TABLE comments (
  id BIGSERIAL PRIMARY KEY,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  content TEXT NOT NULL,
  product_id BIGINT REFERENCES products(id) ON DELETE CASCADE,
  user_id UUID REFERENCES auth.users(id),
  user_email TEXT
);

-- RLS 활성화
ALTER TABLE comments ENABLE ROW LEVEL SECURITY;

-- 정책: 모든 사람이 읽기 가능
CREATE POLICY "Anyone can read comments"
ON comments FOR SELECT
USING (true);

-- 정책: 로그인한 사용자만 생성 가능
CREATE POLICY "Authenticated users can insert comments"
ON comments FOR INSERT
WITH CHECK (auth.uid() = user_id);

-- 정책: 작성자만 삭제 가능
CREATE POLICY "Users can delete own comments"
ON comments FOR DELETE
USING (auth.uid() = user_id);
```

### SQL 실행 방법

1. Supabase 대시보드 → SQL Editor
2. "+ New query" 클릭
3. 위 SQL 복사 & 붙여넣기
4. "Run" 버튼 클릭
5. 초록색 체크가 뜨면 성공!

---

## 6. 테이블 확인

### Table Editor에서 확인

```
products 테이블이 생성되었습니다!
┌────┬────────────┬───────┬──────────┬───────┬──────────┬───────────┬─────────┬────────────┐
│ id │ created_at │ title │description│ price │ category │ image_url │ user_id │ user_email │
├────┼────────────┼───────┼──────────┼───────┼──────────┼───────────┼─────────┼────────────┤
│    │            │       │          │       │          │           │         │            │
└────┴────────────┴───────┴──────────┴───────┴──────────┴───────────┴─────────┴────────────┘
```

---

## 7. 새 Next.js 프로젝트 생성

### 마켓앱 프로젝트 생성

```bash
# 바탕화면으로 이동
cd ~/Desktop

# 새 프로젝트 생성
npx create-next-app@latest my-market-app

# 옵션 선택
✔ TypeScript? Yes
✔ ESLint? Yes
✔ Tailwind CSS? Yes
✔ src/ directory? No
✔ App Router? Yes
✔ customize import alias? No
```

### 프로젝트 설정

```bash
cd my-market-app

# Supabase 패키지 설치
npm install @supabase/supabase-js

# 개발 서버 실행
npm run dev
```

### 환경 변수 설정

`.env.local` 파일 생성:

```bash
NEXT_PUBLIC_SUPABASE_URL=https://xxx.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=eyJhbGci...
```

### Supabase 클라이언트 설정

`lib/supabase.ts` 생성:

```typescript
import { createClient } from '@supabase/supabase-js';

const supabaseUrl = process.env.NEXT_PUBLIC_SUPABASE_URL!;
const supabaseAnonKey = process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!;

export const supabase = createClient(supabaseUrl, supabaseAnonKey);
```

---

## 8. 폴더 구조 계획

```
my-market-app/
├── app/
│   ├── page.tsx              # 홈 (상품 목록)
│   ├── layout.tsx            # 공통 레이아웃
│   ├── login/
│   │   └── page.tsx          # 로그인 페이지
│   ├── signup/
│   │   └── page.tsx          # 회원가입 페이지
│   ├── products/
│   │   ├── page.tsx          # 상품 목록
│   │   ├── new/
│   │   │   └── page.tsx      # 상품 등록
│   │   └── [id]/
│   │       └── page.tsx      # 상품 상세
│   └── components/
│       ├── Header.tsx        # 헤더
│       ├── ProductCard.tsx   # 상품 카드
│       └── CommentList.tsx   # 댓글 목록
├── lib/
│   └── supabase.ts           # Supabase 클라이언트
├── types/
│   └── index.ts              # 타입 정의
└── ...
```

---

## 9. 타입 정의

### types/index.ts

```typescript
// types/index.ts

export interface Product {
  id: number;
  created_at: string;
  title: string;
  description: string | null;
  price: number;
  category: string;
  image_url: string | null;
  user_id: string;
  user_email: string;
}

export interface Comment {
  id: number;
  created_at: string;
  content: string;
  product_id: number;
  user_id: string;
  user_email: string;
}

export interface User {
  id: string;
  email: string;
}
```

```
💡 TypeScript 타입의 장점
   - 자동 완성 지원
   - 오타 방지
   - 코드 문서화
   - 에러 미리 감지
```

---

## 10. 카테고리 정의

### constants/categories.ts

```typescript
// constants/categories.ts

export const CATEGORIES = [
  { id: 'all', name: '전체', emoji: '📦' },
  { id: 'electronics', name: '가전', emoji: '📱' },
  { id: 'clothing', name: '의류', emoji: '👕' },
  { id: 'books', name: '도서', emoji: '📚' },
  { id: 'sports', name: '스포츠', emoji: '⚽' },
  { id: 'furniture', name: '가구', emoji: '🪑' },
  { id: 'etc', name: '기타', emoji: '📦' },
] as const;

export type CategoryId = typeof CATEGORIES[number]['id'];
```

---

## 핵심 정리

```
✅ 기능 정의 → DB 설계 → 구현 순서가 중요
✅ products, comments 테이블 생성
✅ RLS로 데이터 접근 권한 설정
✅ 작성자만 수정/삭제 가능하게 설정
✅ TypeScript 타입으로 안전한 개발
```

---

## 데이터베이스 체크리스트

```
✅ products 테이블 생성
✅ comments 테이블 생성
✅ RLS 정책 설정
✅ 새 Next.js 프로젝트 생성
✅ Supabase 연결 설정
✅ 타입 정의
```

---

## 다음 시간 예고

> 회원가입과 로그인 기능을 구현합니다!
> Supabase Auth를 사용해서
> 이메일/비밀번호 인증을 만들어봅니다.
