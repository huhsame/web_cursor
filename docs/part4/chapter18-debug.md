# Chapter 18. MCP로 디버깅하기

"내 앱이 안 돌아가요!"
이번 챕터에서는 **Supabase MCP**를 활용해서 실제 문제를 해결하는 방법을 배웁니다.

---

## 18.1 "내 앱이 안 돌아가요!"

### 흔한 상황들

고구마마켓을 개발하다 보면 이런 상황이 생깁니다:

```
😱 "상품 등록 버튼을 눌러도 아무 반응이 없어요"
😱 "로그인은 되는데 프로필 정보가 안 나와요"
😱 "댓글을 달았는데 화면에 안 보여요"
😱 "갑자기 에러가 나기 시작했어요"
```

### 원인 파악이 어려운 이유

```
1. 에러 메시지가 모호함
2. 프론트엔드 vs 백엔드 문제 구분 어려움
3. DB에 뭐가 저장됐는지 모름
4. 테이블 구조가 바뀌었는지 모름
```

### MCP로 해결!

**Supabase MCP**를 사용하면:

```
✅ AI가 직접 DB 상태를 확인
✅ 테이블 구조와 데이터를 즉시 조회
✅ 문제 원인을 정확하게 파악
✅ 해결책까지 제시
```

---

## 18.2 Supabase MCP 연결 확인

> Part 3에서 이미 설정했다면 건너뛰세요!

### MCP 상태 확인

Cursor 설정 → MCP에서 supabase가 **✅ 연결됨** 상태인지 확인:

```
MCP Servers
├── context7 ✅
└── supabase ✅  ← 초록불이면 OK!
```

### 연결 안 됐다면?

Chapter 16을 참고해서 설정하세요:
1. Supabase 액세스 토큰 발급
2. `.cursor/mcp.json`에 추가

```json
{
  "mcpServers": {
    "supabase": {
      "command": "npx",
      "args": [
        "-y",
        "@supabase/mcp-server-supabase@latest",
        "--access-token",
        "sbp_xxxxxxxxxxxxx"
      ]
    }
  }
}
```

---

## 18.3 DB 상태 확인하기

### 기본 명령어들

AI에게 이렇게 물어보세요:

| 질문 | 용도 |
|------|------|
| "Supabase에 어떤 테이블 있어?" | 전체 테이블 목록 |
| "products 테이블 구조 보여줘" | 컬럼명, 타입 확인 |
| "products 테이블 데이터 5개만 보여줘" | 실제 데이터 확인 |
| "profiles 테이블에 데이터가 몇 개야?" | 데이터 개수 확인 |

### 실습: 테이블 구조 확인

```
products 테이블 구조 보여줘
```

**예상 결과:**
```
products 테이블 스키마:

| 컬럼 | 타입 | 기본값 | 설명 |
|------|------|--------|------|
| id | int8 | auto | Primary Key |
| title | text | - | 상품명 |
| price | int4 | - | 가격 |
| description | text | - | 설명 |
| image_url | text | - | 이미지 URL |
| user_id | uuid | - | 판매자 ID |
| category | text | - | 카테고리 |
| created_at | timestamptz | now() | 등록일시 |
```

### 실습: 데이터 확인

```
products 테이블에서 최근 등록된 상품 3개 보여줘
```

**예상 결과:**
```
최근 등록된 상품 3개:

| id | title | price | user_id | created_at |
|----|-------|-------|---------|------------|
| 15 | 맥북 프로 | 2500000 | abc-123 | 2024-01-15 |
| 14 | 아이패드 | 850000 | def-456 | 2024-01-14 |
| 13 | 에어팟 | 180000 | abc-123 | 2024-01-13 |
```

---

## 18.4 흔한 에러 시나리오와 해결법

### 시나리오 1: "상품 등록이 안 돼요"

**증상:** 상품 등록 버튼을 눌러도 아무 반응이 없음

**AI에게 물어보기:**
```
products 테이블에 최근 INSERT가 됐는지 확인해줘.
그리고 RLS 정책도 확인해줘.
```

**원인 찾기:**
```
AI: products 테이블을 확인했습니다.

1. 최근 데이터 확인:
   - 마지막 INSERT: 3일 전
   - 최근 등록 시도 기록 없음

2. RLS 정책 확인:
   ⚠️ INSERT 정책이 없습니다!

   현재 정책:
   - SELECT: 모든 사용자 허용 ✅
   - INSERT: 정책 없음 ❌
   - UPDATE: 본인만 허용 ✅
   - DELETE: 본인만 허용 ✅

해결 방법: INSERT 정책을 추가해야 합니다.
```

**해결책:**
```
로그인한 사용자가 상품을 등록할 수 있도록 RLS 정책 만들어줘
```

AI가 SQL을 생성해줍니다:
```sql
CREATE POLICY "Users can insert their own products"
ON products
FOR INSERT
TO authenticated
WITH CHECK (auth.uid() = user_id);
```

### 시나리오 2: "프로필 정보가 안 나와요"

**증상:** 로그인은 되는데 닉네임이 안 보임

**AI에게 물어보기:**
```
내 user id가 "abc-123-def-456"인데,
profiles 테이블에 이 사용자 데이터가 있어?
```

**원인 찾기:**
```
AI: profiles 테이블을 확인했습니다.

검색 결과: 데이터 없음 ❌

user_id "abc-123-def-456"에 해당하는
프로필이 profiles 테이블에 존재하지 않습니다.

가능한 원인:
1. 회원가입 시 trigger가 작동하지 않음
2. profiles 테이블에 직접 INSERT가 안 됨
```

**해결책:**
```
profiles 테이블의 트리거가 제대로 설정되어 있는지 확인해줘
```

AI가 확인 후:
```
트리거 확인 결과:

⚠️ handle_new_user 트리거가 없습니다!

회원가입 시 자동으로 프로필을 생성하는 트리거를
만들어야 합니다.
```

트리거 생성 SQL:
```sql
-- 트리거 함수 생성
CREATE OR REPLACE FUNCTION public.handle_new_user()
RETURNS TRIGGER AS $$
BEGIN
  INSERT INTO public.profiles (id, nickname)
  VALUES (NEW.id, NEW.email);
  RETURN NEW;
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;

-- 트리거 연결
CREATE TRIGGER on_auth_user_created
  AFTER INSERT ON auth.users
  FOR EACH ROW EXECUTE FUNCTION public.handle_new_user();
```

### 시나리오 3: "댓글이 안 보여요"

**증상:** 댓글을 달았는데 화면에 표시 안 됨

**AI에게 물어보기:**
```
comments 테이블에서 product_id가 5인 댓글들 보여줘.
그리고 profiles 테이블과 JOIN해서 닉네임도 같이 보여줘.
```

**원인 찾기:**
```
AI: 조회 결과입니다.

comments 테이블:
| id | content | product_id | user_id | created_at |
|----|---------|------------|---------|------------|
| 1  | 좋아요! | 5 | abc-123 | 2024-01-15 |
| 2  | 네고 가능? | 5 | def-456 | 2024-01-15 |

⚠️ 문제 발견:
user_id "def-456"이 profiles 테이블에 없습니다!

JOIN 결과에서 이 댓글이 누락될 수 있습니다.
프론트엔드 코드에서 INNER JOIN을 사용하고 있다면
profiles가 없는 댓글은 표시되지 않습니다.
```

**해결책:**
```
프론트엔드 코드에서 LEFT JOIN으로 바꾸거나,
누락된 profiles 데이터를 추가해야 합니다.

1. LEFT JOIN으로 변경 (추천):
```

```typescript
const { data } = await supabase
  .from('comments')
  .select(`
    *,
    profiles!left(nickname, avatar_url)
  `)
  .eq('product_id', productId)
```

### 시나리오 4: "갑자기 에러가 났어요"

**증상:** 잘 되던 기능이 갑자기 안 됨

**AI에게 물어보기:**
```
products 테이블 구조가 바뀌었어?
원래 어떤 컬럼들이 있었는지 코드랑 비교해줘
```

**원인 찾기:**
```
AI: products 테이블과 코드를 비교했습니다.

현재 테이블 구조:
| 컬럼 | 타입 |
|------|------|
| id | int8 |
| title | text |
| price | int4 |
| description | text |
| image_url | text |
| user_id | uuid |
| category | text |  ← 새로 추가됨!
| created_at | timestamptz |

코드에서 사용하는 구조:
- id, title, price, description, image_url, user_id, created_at

⚠️ 문제:
category 컬럼이 추가되었는데,
코드에서 이를 처리하지 않고 있습니다.

INSERT 시 category가 NOT NULL이면 에러가 발생합니다.
```

**해결책:**
```
category 컬럼이 NOT NULL인지 확인해줘
```

```
AI: category 컬럼 확인 결과:

category: text, NOT NULL ❌

NOT NULL 제약이 있어서
category 값 없이 INSERT하면 에러가 납니다.

해결 방법:
1. 코드에서 category 값을 추가하거나
2. 테이블에서 NOT NULL 제약을 제거

추천: 기본값 설정
```

```sql
ALTER TABLE products
ALTER COLUMN category SET DEFAULT '기타';
```

---

## 18.5 MCP로 안 될 때

### 현실적인 이야기

> MCP가 만능은 아닙니다!

실제로 사용해보면:
- 스키마 확인, 데이터 조회는 잘 됨 ✅
- 하지만 복잡한 설정은 직접 해야 함 ⚠️

### MCP로 잘 되는 것

| 작업 | MCP 활용도 |
|------|-----------|
| 테이블 구조 확인 | ⭐⭐⭐ |
| 데이터 조회 | ⭐⭐⭐ |
| 간단한 INSERT/UPDATE | ⭐⭐⭐ |
| 문제 원인 파악 | ⭐⭐⭐ |
| SQL 쿼리 생성 | ⭐⭐⭐ |

### 직접 해야 하는 것

| 작업 | 방법 |
|------|------|
| RLS 정책 설정 | Supabase 대시보드 |
| 트리거 생성 | SQL Editor |
| Storage 버킷 설정 | Supabase 대시보드 |
| Auth 설정 | Supabase 대시보드 |

### 추천 워크플로우

```
1. 에러 발생!
   ↓
2. AI에게 증상 설명
   ↓
3. MCP로 DB 상태 확인
   ↓
4. 원인 파악
   ↓
5. 해결책 받기 (SQL 코드)
   ↓
6. Supabase 대시보드에서 실행
   ↓
7. 확인!
```

---

## 18.6 디버깅 체크리스트

문제가 생겼을 때 순서대로 확인하세요:

### Step 1: 에러 메시지 확인

```
브라우저 콘솔(F12)에서 에러 메시지 확인
→ AI에게 에러 메시지 보여주기
```

### Step 2: DB 연결 확인

```
Supabase에 어떤 프로젝트들이 있어?
내 프로젝트가 정상 상태야?
```

### Step 3: 테이블 구조 확인

```
[테이블명] 테이블 구조 보여줘
코드에서 사용하는 컬럼이랑 일치해?
```

### Step 4: 데이터 확인

```
[테이블명]에 데이터가 몇 개야?
최근 데이터 보여줘
```

### Step 5: RLS/권한 확인

```
[테이블명]의 RLS 정책 보여줘
INSERT/UPDATE/DELETE 권한이 있어?
```

### Step 6: 관계 확인

```
[테이블A]와 [테이블B] JOIN이 제대로 돼?
외래 키 관계가 맞아?
```

---

## 18.7 실전 연습

### 연습 1: 직접 해보기

자신의 고구마마켓에서 다음을 확인해보세요:

```
1. "내 Supabase에 어떤 테이블들이 있어?"
2. "products 테이블 구조 보여줘"
3. "products 테이블에 데이터 몇 개 있어?"
4. "profiles 테이블과 products 테이블의 관계 알려줘"
```

### 연습 2: 의도적으로 문제 만들기

테스트를 위해 일부러 문제를 만들어보세요:

1. products 테이블에서 user_id를 NULL로 변경
2. 상품 등록 시도
3. 에러 발생!
4. MCP로 원인 파악

```
products 테이블에서 user_id가 NULL인 데이터 있어?
```

### 연습 3: 해결까지

문제를 찾았으면 해결해보세요:

```
user_id가 NULL인 products 데이터를 찾아서
내 user_id로 업데이트하는 SQL 만들어줘
```

---

## 핵심 정리

```
✅ 에러 발생 → MCP로 DB 상태 확인
✅ "테이블 구조 보여줘"로 스키마 확인
✅ "데이터 보여줘"로 실제 값 확인
✅ "RLS 정책 보여줘"로 권한 확인
✅ 복잡한 설정은 Supabase 대시보드에서
✅ MCP = 문제 파악 도구, 만능 해결사 아님!
```

---

## 다음 챕터에서는

> **Context7 MCP**를 활용해서 최신 문서를 참조하며 코딩합니다!
> AI가 모르는 최신 문법도 Context7로 해결할 수 있어요.
>
> 👉 [Chapter 19. 문서 참조하며 코딩하기](chapter19-context7.md)
