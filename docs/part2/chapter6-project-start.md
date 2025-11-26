# Chapter 6. 프로젝트 시작

Part 2에 오신 것을 환영합니다! 🎉

이제 본격적으로 **고구마마켓**을 만들어봅니다.
가계부보다 더 복잡하지만, 단계별로 차근차근 진행할 거예요.

---

## 6.1 고구마마켓 소개

### 무엇을 만들까요?

> **중고거래 플랫폼**입니다.
>
> 당근마켓처럼 물건을 사고파는 서비스예요.
> (우리는 고구마마켓이라고 부를게요! 🍠)

<!-- 스크린샷: 완성된 고구마마켓 미리보기 -->

### 만들 기능들

| 기능 | 설명 |
|------|------|
| **회원가입/로그인** | 사용자 인증 |
| **상품 등록** | 판매할 물건 올리기 |
| **상품 목록** | 등록된 상품들 보기 |
| **상품 상세** | 상품 정보 자세히 보기 |
| **상품 삭제** | 내가 올린 상품 삭제 |
| **댓글** | 상품에 댓글 달기 |
| **이미지 업로드** | 상품 사진 올리기 |
| **검색** | 상품 검색하기 |
| **카테고리 필터** | 카테고리별 보기 |

### 가계부와 다른 점

| 가계부 | 고구마마켓 |
|--------|----------|
| 페이지 1개 | 여러 페이지 |
| 로그인 없음 | 로그인 필요 |
| 단순 CRUD | 데이터 간의 관계 |
| 텍스트만 | 이미지도 |

---

## 6.2 웹 개발 기본 개념

고구마마켓을 만들기 전에,
알아두면 좋은 개념들을 간단히 설명합니다.

### URL과 페이지

URL은 **웹페이지의 주소**입니다.

```
https://goguma-market.com/           → 메인 페이지
https://goguma-market.com/login      → 로그인 페이지
https://goguma-market.com/products   → 상품 목록
https://goguma-market.com/products/1 → 1번 상품 상세
```

### 라우팅이란?

> **URL에 따라 다른 페이지를 보여주는 것**입니다.

Next.js에서는 **폴더 구조 = URL 구조**입니다!

```
app/
├── page.tsx              → /
├── login/
│   └── page.tsx          → /login
├── signup/
│   └── page.tsx          → /signup
└── products/
    ├── page.tsx          → /products
    ├── new/
    │   └── page.tsx      → /products/new
    └── [id]/
        └── page.tsx      → /products/123 (동적)
```

### 동적 라우팅

`[id]` 폴더는 **어떤 값이든 받을 수 있습니다.**

```
/products/1   → id = 1
/products/2   → id = 2
/products/99  → id = 99
```

이걸로 상품 상세 페이지를 만들 수 있어요!

### CRUD란?

대부분의 웹 서비스는 CRUD로 이루어져 있습니다.

| | 의미 | 예시 |
|---|------|------|
| **C**reate | 생성 | 상품 등록 |
| **R**ead | 조회 | 상품 목록, 상세 |
| **U**pdate | 수정 | 상품 수정 |
| **D**elete | 삭제 | 상품 삭제 |

인스타그램, 유튜브, 블로그... 다 CRUD입니다!

---

## 6.3 프로젝트 만들기

### Step 1: 새 프로젝트 생성

터미널에서 **직접 타이핑**:

```bash
npx create-next-app@latest goguma-market
```

### Step 2: 옵션 선택

```
✔ Would you like to use TypeScript? … Yes
✔ Would you like to use ESLint? … Yes
✔ Would you like to use Tailwind CSS? … Yes
✔ Would you like to use `src/` directory? … No
✔ Would you like to use App Router? (recommended) … Yes
✔ Would you like to customize the default import alias? … No
```

### Step 3: 프로젝트 열기

```bash
cd goguma-market
```

커서에서 `File` → `Open Folder` → `goguma-market` 선택

### Step 4: 개발 서버 실행

터미널에서 **직접 타이핑**:

```bash
npm run dev
```

브라우저에서 `localhost:3000` 확인!

---

## 6.4 Supabase 설정

### Step 1: 새 프로젝트 만들기

1. [supabase.com](https://supabase.com) 접속
2. **New Project** 클릭
3. 정보 입력:
   - Name: `goguma-market`
   - Region: `Northeast Asia (Seoul)`
4. **Create new project**

### Step 2: 환경변수 설정

프로젝트 루트에 `.env.local` 파일 생성:

```
NEXT_PUBLIC_SUPABASE_URL=여기에_URL
NEXT_PUBLIC_SUPABASE_ANON_KEY=여기에_KEY
```

### Step 3: Supabase 패키지 설치

터미널에서 **직접 타이핑**:

```bash
npm install @supabase/supabase-js
```

---

## 6.5 첫 커밋하기 📝

### 왜 커밋하나요?

> **"여기까지 완성!"** 을 기록해두는 것입니다.
>
> 나중에 문제가 생기면 이 시점으로 돌아올 수 있어요.

### Cursor에서 커밋하기

1. **Source Control** 패널 열기 (왼쪽 사이드바)
2. 변경된 파일들 확인
3. **+** 버튼 클릭 (스테이징)
4. 메시지 입력: `프로젝트 초기 세팅`
5. **✓** 버튼 클릭 (커밋)

<!-- 스크린샷: 커밋 과정 -->

> 💡 **앞으로 각 기능을 완성할 때마다 커밋합니다!**

---

## 6.6 GitHub에 올리기

### Step 1: GitHub에서 새 저장소 만들기

1. github.com 접속
2. **+** → **New repository**
3. Name: `goguma-market`
4. **Create repository**

### Step 2: 커서에서 Push

<!-- 스크린샷: Push 과정 (사용자가 추가) -->

---

## 핵심 정리

```
✅ 고구마마켓 = 중고거래 플랫폼
✅ 여러 페이지, 로그인, CRUD
✅ Next.js 폴더 구조 = URL 구조
✅ CRUD = Create, Read, Update, Delete
✅ 프로젝트 생성 완료
✅ 첫 커밋 완료
```

---

## 다음 챕터에서는

> 메인 페이지와 레이아웃을 만들어봅니다!
>
> 👉 [Chapter 7. 메인 페이지 만들기](part2/chapter7-main-page.md)

