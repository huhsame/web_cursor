# Chapter 9. 로그인 기능 만들기

> **Chapter 8에서 데이터베이스 이론을 배웠습니다.**
>
> 이제 실제로 Supabase를 사용해봅시다!
>
> 먼저 **로그인/회원가입 기능**을 만들고,
> 다음 챕터에서 데이터베이스를 연결할 겁니다.

이번 챕터에서는 **Supabase 인증 기능**을 사용해서
회원가입과 로그인을 만듭니다!

---

## 9.1 인증이란?

### 쉽게 말하면

> **"너 누구야?"를 확인하는 것**입니다.

```
로그인 = 본인 확인
로그아웃 = 확인 해제
```

### 왜 필요한가요?

| 로그인 없을 때 | 로그인 있을 때 |
|--------------|--------------|
| 누가 글 썼는지 모름 | 작성자 표시 가능 |
| 아무나 수정/삭제 | 본인만 수정/삭제 |
| 개인화 불가능 | 내 상품, 내 댓글 관리 |

---

## 9.2 Supabase 인증 설정하기

> **Chapter 6에서 이미 Supabase 프로젝트를 만들었습니다!**
>
> 이제 **로그인/회원가입** 기능을 활성화하겠습니다.

### Step 1: Authentication 메뉴 열기

1. Supabase 대시보드 접속
2. 왼쪽 메뉴에서 **Authentication** 클릭
3. **Providers** 탭 클릭

<!-- 스크린샷: Authentication 메뉴 -->

### Step 2: 이메일 인증 확인

**Email** Provider가 기본으로 활성화되어 있습니다.

<!-- 스크린샷: Email Provider 설정 -->

### Step 3: 이메일 확인 비활성화 (개발용)

개발할 때는 이메일 확인 과정이 번거로우니 끄겠습니다.

1. **Email** Provider 클릭
2. **Confirm email** 옵션을 **OFF**로 변경
3. **Save** 클릭

<!-- 스크린샷: Confirm email OFF -->

> ⚠️ **실제 서비스에서는 ON으로 해야 합니다!**
> 지금은 개발 편의를 위해 끄는 것입니다.

---

## 9.3 사용자 프로필 테이블 만들기

### 왜 프로필 테이블이 필요한가요?

> **Supabase Auth는 이메일/비밀번호만 저장합니다.**
>
> 닉네임, 프로필 사진 같은 추가 정보는 별도 테이블에 저장해야 해요!

```
Supabase Auth (기본 제공):
- id (사용자 고유 ID)
- email
- password (암호화됨)

profiles 테이블 (우리가 만들 것):
- id (Auth의 id와 연결)
- nickname (닉네임)
- avatar_url (프로필 사진)
- created_at (가입 시간)
```

### Cursor에게 요청하기

```
Supabase에 사용자 프로필 테이블을 만들어줘.

테이블명: profiles

컬럼:
- id: UUID (primary key, Supabase Auth의 user id와 연결)
- nickname: TEXT (닉네임, NOT NULL)
- avatar_url: TEXT (프로필 사진 URL, NULL 가능)
- created_at: TIMESTAMP (생성 시간, 기본값 NOW())

추가로:
- 회원가입할 때 자동으로 profiles 행이 생성되는 트리거도 만들어줘
- RLS(Row Level Security) 정책 추가:
  - 모든 사용자가 profiles를 읽을 수 있음
  - 본인의 profile만 수정/삭제 가능

SQL 코드로 작성해줘.
```

<!-- 스크린샷: Cursor에 요청 -->

### AI가 만든 SQL 실행하기

1. AI가 생성한 SQL 코드 복사
2. Supabase 대시보드 접속
3. 왼쪽 메뉴에서 **SQL Editor** 클릭
4. **New query** 클릭
5. SQL 코드 붙여넣기
6. **Run** 버튼 클릭

<!-- 스크린샷: SQL Editor -->

**예상 SQL 코드 (참고용):**

```sql
-- profiles 테이블 생성
CREATE TABLE profiles (
  id UUID PRIMARY KEY REFERENCES auth.users(id) ON DELETE CASCADE,
  nickname TEXT NOT NULL,
  avatar_url TEXT,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- RLS 활성화
ALTER TABLE profiles ENABLE ROW LEVEL SECURITY;

-- 정책: 모든 사용자가 읽을 수 있음
CREATE POLICY "Anyone can view profiles"
  ON profiles FOR SELECT
  USING (true);

-- 정책: 본인만 수정 가능
CREATE POLICY "Users can update own profile"
  ON profiles FOR UPDATE
  USING (auth.uid() = id);

-- 회원가입 시 자동으로 profile 생성하는 트리거 함수
CREATE OR REPLACE FUNCTION public.handle_new_user()
RETURNS TRIGGER AS $$
BEGIN
  INSERT INTO public.profiles (id, nickname)
  VALUES (NEW.id, NEW.email);
  RETURN NEW;
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;

-- 트리거 생성
CREATE TRIGGER on_auth_user_created
  AFTER INSERT ON auth.users
  FOR EACH ROW EXECUTE FUNCTION public.handle_new_user();
```

### 코드 이해하기 (선택)

> 건너뛰어도 됩니다!

**1. profiles 테이블:**
- `id UUID PRIMARY KEY`: 사용자 고유 ID
- `REFERENCES auth.users(id)`: Supabase Auth와 연결
- `ON DELETE CASCADE`: 계정 삭제하면 profile도 삭제

**2. RLS (Row Level Security):**
- 데이터베이스 레벨의 보안 정책
- "누가 어떤 데이터를 읽고 쓸 수 있는지" 제어

**3. 트리거 (Trigger):**
- 특정 이벤트 발생 시 자동 실행
- 회원가입(INSERT on auth.users) → profile 자동 생성

### Supabase에서 확인하기

1. 왼쪽 메뉴에서 **Table Editor** 클릭
2. **profiles** 테이블이 보이면 성공!
3. 아직 데이터는 없습니다 (회원가입해야 생김)

<!-- 스크린샷: profiles 테이블 -->

---

## 9.4 회원가입 페이지 만들기

### Cursor에게 요청하기

```
회원가입 페이지를 만들어줘.
(app/signup/page.tsx)

입력 받을 것:
- 이메일
- 비밀번호 (6자 이상)
- 비밀번호 확인
- 닉네임

기능:
- 비밀번호와 비밀번호 확인이 같은지 검사
- Supabase Auth로 회원가입
- 닉네임도 같이 저장
- 성공하면 메인 페이지로 이동
- 에러 나면 메시지 보여주기 (이미 가입된 이메일 등)
```

<!-- 스크린샷: Cursor에 요청 -->

### AI 응답 따라하기

1. **Apply** 버튼으로 코드 적용
2. 파일 저장

### 헤더에 버튼 추가하기

```
헤더에 로그인/회원가입 버튼을 추가해줘.

- "로그인" 버튼 → /login 페이지로 이동
- "회원가입" 버튼 → /signup 페이지로 이동
```

### 테스트하기

1. 브라우저에서 `localhost:3000/signup` 접속
2. 정보 입력:
   - 이메일: test@example.com
   - 비밀번호: 123456
   - 닉네임: 테스트
3. 가입 버튼 클릭

### Supabase에서 확인

1. Supabase 대시보드 → **Authentication** → **Users**
2. 방금 가입한 사용자가 보이면 성공!

<!-- 스크린샷: Users 목록 -->

---

## 9.5 로그인 페이지 만들기

### Cursor에게 요청하기

```
로그인 페이지를 만들어줘.
(app/login/page.tsx)

입력 받을 것:
- 이메일
- 비밀번호

기능:
- Supabase Auth로 로그인
- 성공하면 메인 페이지로 이동
- 에러 나면 메시지 보여주기 (잘못된 이메일/비밀번호)
- "회원가입" 링크 추가
```

### AI 응답 따라하기

1. **Apply** 버튼으로 코드 적용
2. 파일 저장

### 테스트하기

1. `localhost:3000/login` 접속
2. 방금 가입한 계정으로 로그인
3. 메인 페이지로 이동하면 성공!

---

## 9.6 로그인 상태 관리하기

### 현재 문제점

로그인해도 헤더가 안 바뀝니다.
앱이 "누가 로그인했는지" 모르기 때문입니다.

### Cursor에게 요청하기

```
로그인 상태를 관리하는 기능을 만들어줘.

필요한 것:
1. AuthContext 만들기 (app/contexts/AuthContext.tsx)
   - 현재 로그인한 사용자 정보 저장
   - 로그인 상태 변경 감지
   - 앱 전체에서 사용 가능하게

2. app/layout.tsx 수정
   - AuthProvider로 앱 전체 감싸기

3. 로그아웃 기능 추가
   - AuthContext에 logout 함수 만들기
```

### AI 응답 따라하기

1. **Apply** 버튼으로 코드 적용
2. 파일 저장

### 동작 확인

브라우저 콘솔에서 로그인 상태가 감지되는지 확인

---

## 9.7 헤더 수정하기

### Cursor에게 요청하기

```
헤더를 로그인 상태에 따라 다르게 보여줘.

로그인 안 한 상태:
- "로그인" 버튼
- "회원가입" 버튼

로그인한 상태:
- "상품 등록" 버튼
- 닉네임 표시 (예: "안녕하세요, 테스트님")
- "로그아웃" 버튼

AuthContext의 user 정보 사용해서 구현.
```

### AI 응답 따라하기

1. **Apply** 버튼으로 코드 적용
2. 파일 저장

### 테스트하기

1. **로그아웃 상태**: 로그인/회원가입 버튼이 보이는지 확인
2. **로그인**: 로그인 버튼 클릭 → 로그인
3. **로그인 상태**: 닉네임과 로그아웃 버튼이 보이는지 확인
4. **로그아웃**: 로그아웃 버튼 클릭 → 다시 로그인/회원가입 버튼으로 바뀌는지 확인

<!-- 스크린샷: 로그인 전 헤더 -->
<!-- 스크린샷: 로그인 후 헤더 -->

---

## 9.8 문제가 생겼다면?

### "회원가입이 안 돼요"

1. Supabase Authentication → Providers 확인
   - Email Provider가 활성화되어 있나요?

2. 비밀번호가 6자 이상인가요?

3. 에러 메시지를 확인하세요
   - 이미 가입된 이메일일 수 있습니다

### "로그인해도 헤더가 안 바뀌어요"

1. AuthContext가 제대로 적용되었나요?
   - layout.tsx에서 AuthProvider로 감싸졌는지 확인

2. 브라우저 콘솔 확인
   - 에러 메시지가 있나요?

3. 새로고침 해보세요

### "user_id가 비어있어요"

1. 로그인 상태에서 상품을 등록했나요?

2. 상품 등록 코드에서 user.id를 제대로 사용했나요?

---

## 핵심 정리

```
✅ Supabase Auth = 인증 기능 제공 (회원가입, 로그인)
✅ AuthContext = 앱 전체에서 로그인 상태 공유
✅ 보호된 페이지 = 로그인 필요, 리다이렉트

흐름:
1. 회원가입 → Supabase Auth에 저장
2. 로그인 → 세션 생성
3. AuthContext → 로그인 상태 감지
4. 헤더가 로그인 상태에 따라 바뀜
```

**이제 로그인 기능이 완성되었습니다!**

다음 챕터에서 이 로그인 기능을 활용해서:
- 상품 등록 (로그인한 사람만)
- 본인 확인 (user_id 비교)
- 댓글 작성 (로그인 필요)

등을 만들 겁니다!

---

## 다음 챕터에서는

> 로그인 기능이 완성되었습니다!
>
> 이제 **데이터베이스를 연결**하고
> 고구마마켓의 핵심 기능(상품, 댓글 등)을 만들어봅니다!
>
> 👉 [Chapter 10. 데이터베이스로 고구마마켓 만들기](part2/chapter10-db-connect.md)
