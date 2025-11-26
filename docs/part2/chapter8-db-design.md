# Chapter 8. 데이터베이스 설계

이번 챕터에서는 고구마마켓에 필요한 **데이터베이스 테이블**을 설계하고 만듭니다.

---

## 8.1 어떤 데이터가 필요할까?

### 고구마마켓에서 저장할 데이터

```
1. 사용자 정보 (users)
   - 누가 회원가입했는지
   - 이메일, 닉네임 등

2. 상품 정보 (products)
   - 어떤 상품이 등록됐는지
   - 제목, 가격, 설명, 이미지 등

3. 댓글 정보 (comments)
   - 상품에 달린 댓글
   - 내용, 작성자, 작성시간 등
```

---

## 8.2 테이블 관계 이해하기

### 1:N 관계란?

> **하나가 여러 개를 가질 수 있는 관계**입니다.

**예시:**
```
한 명의 사용자 → 여러 개의 상품 등록 가능
한 개의 상품 → 여러 개의 댓글 가능

사용자(1) : 상품(N)
상품(1) : 댓글(N)
```

### 그림으로 보면

```
┌─────────┐      ┌─────────┐      ┌─────────┐
│  Users  │──1:N─│Products │──1:N─│Comments │
│ (사용자) │      │ (상품)   │      │ (댓글)   │
└─────────┘      └─────────┘      └─────────┘

- 한 사용자가 여러 상품을 등록
- 한 상품에 여러 댓글이 달림
```

### 외래 키 (Foreign Key)

> **다른 테이블을 참조하는 열**입니다.

```
products 테이블에 user_id가 있으면
→ "이 상품을 누가 등록했는지" 알 수 있음

comments 테이블에 product_id가 있으면
→ "이 댓글이 어느 상품에 달렸는지" 알 수 있음
```

---

## 8.3 테이블 설계하기

### products (상품) 테이블

| 열 이름 | 타입 | 설명 |
|--------|------|------|
| id | bigint | 고유 번호 (자동) |
| created_at | timestamp | 등록 시간 (자동) |
| title | text | 상품 제목 |
| description | text | 상품 설명 |
| price | integer | 가격 |
| image_url | text | 이미지 주소 |
| user_id | uuid | 등록한 사용자 |
| status | text | 판매중/예약중/판매완료 |

### comments (댓글) 테이블

| 열 이름 | 타입 | 설명 |
|--------|------|------|
| id | bigint | 고유 번호 (자동) |
| created_at | timestamp | 작성 시간 (자동) |
| content | text | 댓글 내용 |
| product_id | bigint | 어느 상품의 댓글인지 |
| user_id | uuid | 작성한 사용자 |

### profiles (사용자 프로필) 테이블

> Supabase는 `auth.users` 테이블을 자동으로 만들어줍니다.
> 우리는 추가 정보(닉네임 등)를 위해 `profiles` 테이블을 만듭니다.

| 열 이름 | 타입 | 설명 |
|--------|------|------|
| id | uuid | 사용자 ID (auth.users 참조) |
| created_at | timestamp | 가입 시간 (자동) |
| nickname | text | 닉네임 |
| avatar_url | text | 프로필 이미지 |

---

## 8.4 Supabase에서 테이블 만들기

### Step 1: Supabase 프로젝트 만들기

1. **[supabase.com](https://supabase.com)** 접속
2. 로그인 (GitHub 계정 추천)
3. **New Project** 클릭
4. 정보 입력:
   - **Name**: `goguma-market`
   - **Database Password**: 비밀번호 설정
   - **Region**: `Northeast Asia (Seoul)`
5. **Create new project** 클릭

<!-- 스크린샷: 새 프로젝트 생성 -->

> ⏳ 1~2분 기다리세요.

### Step 2: SQL Editor 열기

왼쪽 메뉴에서 **SQL Editor** 클릭

<!-- 스크린샷: SQL Editor 위치 -->

### Step 3: profiles 테이블 만들기

아래 SQL을 **복사해서 붙여넣기** 후 **Run** 클릭:

```sql
-- profiles 테이블 생성
CREATE TABLE profiles (
  id UUID PRIMARY KEY REFERENCES auth.users(id) ON DELETE CASCADE,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  nickname TEXT,
  avatar_url TEXT
);

-- 새 사용자 가입시 자동으로 프로필 생성
CREATE OR REPLACE FUNCTION public.handle_new_user()
RETURNS TRIGGER AS $$
BEGIN
  INSERT INTO public.profiles (id, nickname)
  VALUES (NEW.id, NEW.raw_user_meta_data->>'nickname');
  RETURN NEW;
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;

CREATE TRIGGER on_auth_user_created
  AFTER INSERT ON auth.users
  FOR EACH ROW EXECUTE FUNCTION public.handle_new_user();

-- RLS 활성화
ALTER TABLE profiles ENABLE ROW LEVEL SECURITY;

CREATE POLICY "누구나 프로필 조회 가능"
ON profiles FOR SELECT
USING (true);

CREATE POLICY "본인만 프로필 수정 가능"
ON profiles FOR UPDATE
USING (auth.uid() = id);
```

**Success** 메시지가 나오면 성공!

### Step 4: products 테이블 만들기

새 SQL 쿼리에서 아래를 실행:

```sql
-- products 테이블 생성
CREATE TABLE products (
  id BIGSERIAL PRIMARY KEY,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  title TEXT NOT NULL,
  description TEXT,
  price INTEGER NOT NULL,
  image_url TEXT,
  user_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  status TEXT DEFAULT '판매중'
);

-- RLS 활성화
ALTER TABLE products ENABLE ROW LEVEL SECURITY;

CREATE POLICY "누구나 상품 조회 가능"
ON products FOR SELECT
USING (true);

CREATE POLICY "로그인한 사용자만 상품 등록 가능"
ON products FOR INSERT
WITH CHECK (auth.uid() = user_id);

CREATE POLICY "본인 상품만 수정 가능"
ON products FOR UPDATE
USING (auth.uid() = user_id);

CREATE POLICY "본인 상품만 삭제 가능"
ON products FOR DELETE
USING (auth.uid() = user_id);
```

### Step 5: comments 테이블 만들기

```sql
-- comments 테이블 생성
CREATE TABLE comments (
  id BIGSERIAL PRIMARY KEY,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  content TEXT NOT NULL,
  product_id BIGINT REFERENCES products(id) ON DELETE CASCADE,
  user_id UUID REFERENCES profiles(id) ON DELETE CASCADE
);

-- RLS 활성화
ALTER TABLE comments ENABLE ROW LEVEL SECURITY;

CREATE POLICY "누구나 댓글 조회 가능"
ON comments FOR SELECT
USING (true);

CREATE POLICY "로그인한 사용자만 댓글 작성 가능"
ON comments FOR INSERT
WITH CHECK (auth.uid() = user_id);

CREATE POLICY "본인 댓글만 삭제 가능"
ON comments FOR DELETE
USING (auth.uid() = user_id);
```

### Step 6: 테이블 확인하기

1. 왼쪽 메뉴에서 **Table Editor** 클릭
2. 3개 테이블이 보이면 성공!
   - `profiles`
   - `products`
   - `comments`

<!-- 스크린샷: Table Editor에서 테이블 확인 -->

---

## 8.5 RLS란? (선택)

> 이 섹션은 건너뛰어도 됩니다!

### Row Level Security

> **행 단위로 접근을 제어**하는 보안 기능입니다.

```
예: "본인 상품만 삭제 가능" 정책

사용자 A가 상품 삭제 요청 →
  - 본인 상품이면 → 삭제됨 ✅
  - 남의 상품이면 → 거부됨 ❌
```

### 왜 필요한가요?

> 악의적인 사용자가 남의 데이터를 삭제하거나 수정하는 것을 막습니다.
>
> 코드에서 체크하지 않아도 DB 레벨에서 자동으로 보호!

---

## 8.6 API 키 설정하기

### Step 1: API 키 복사

1. 왼쪽 메뉴에서 **Project Settings** (톱니바퀴)
2. **API** 메뉴 클릭
3. 두 가지 복사:
   - **Project URL**
   - **anon (public) key**

<!-- 스크린샷: API 키 위치 -->

### Step 2: 환경변수 파일 만들기

프로젝트 폴더에 `.env.local` 파일 만들기:

```
NEXT_PUBLIC_SUPABASE_URL=여기에_URL_붙여넣기
NEXT_PUBLIC_SUPABASE_ANON_KEY=여기에_KEY_붙여넣기
```

### Step 3: Supabase 클라이언트 확인

7장에서 만든 `lib/supabase.ts` 파일이 있는지 확인하세요.

없다면 AI에게 요청:

```
lib/supabase.ts 파일 만들어줘.
환경변수 NEXT_PUBLIC_SUPABASE_URL, NEXT_PUBLIC_SUPABASE_ANON_KEY 사용.
```

---

## 8.7 테스트 데이터 넣어보기 (선택)

### Table Editor에서 직접 추가

1. **Table Editor** → `products` 클릭
2. **Insert** 또는 **+ Insert row** 클릭
3. 데이터 입력:
   - title: `테스트 상품`
   - price: `10000`
   - description: `테스트입니다`
4. **Save** 클릭

<!-- 스크린샷: 데이터 추가 -->

> 💡 user_id는 아직 비워두세요. (로그인 기능 만든 후 테스트)

---

## 핵심 정리

```
✅ 테이블 = 엑셀 시트와 비슷 (행과 열)
✅ 1:N 관계 = 하나가 여러 개를 가짐
✅ 외래 키 = 다른 테이블 참조
✅ RLS = 행 단위 보안 (본인 데이터만 수정/삭제)
✅ 3개 테이블: profiles, products, comments
```

---

## 📝 커밋하기

```
데이터베이스 테이블 설계 완료
```

> 💡 아직 코드 변경은 없지만, 진행 상황을 기록해두세요!

---

## 다음 챕터에서는

> 상품 CRUD 기능을 만들어봅니다!
> (등록, 조회, 수정, 삭제)
>
> 👉 [Chapter 9. 상품 기능 만들기](part2/chapter9-product-crud.md)

