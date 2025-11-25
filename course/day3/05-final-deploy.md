# Day 3 - 5교시: 최종 배포

## 학습 목표
- GitHub에 마켓앱 코드를 업로드한다
- Vercel에 프로덕션 배포한다
- 환경 변수를 설정한다
- 배포 후 테스트를 진행한다

---

## 1. 배포 전 체크리스트

### 기능 확인

```
□ 홈페이지 - 상품 목록 표시
□ 로그인 - 이메일/비밀번호 로그인
□ 회원가입 - 새 계정 생성
□ 상품 등록 - 이미지 포함 등록
□ 상품 상세 - 정보 표시
□ 상품 수정/삭제 - 본인 글만
□ 댓글 - 작성/삭제
□ 검색 - 키워드 검색
□ 필터 - 카테고리 필터
□ 정렬 - 가격순/최신순
```

### 코드 정리

```
□ console.log 제거 (또는 주석)
□ 테스트용 코드 제거
□ 에러 처리 확인
□ 불필요한 파일 삭제
□ .env.local이 .gitignore에 있는지 확인
```

---

## 2. 로컬 빌드 테스트

### 왜 필요한가?

```
개발 서버(npm run dev)와 프로덕션 빌드는 다릅니다!
배포 전에 빌드 에러가 없는지 확인해야 합니다.
```

### 빌드 실행

```bash
npm run build
```

### 성공 시

```
✓ Creating an optimized production build
✓ Compiled successfully
✓ Collecting page data
✓ Generating static pages
✓ Finalizing page optimization

Route (app)                              Size     First Load JS
┌ ○ /                                    5.2 kB        92 kB
├ ○ /login                               2.1 kB        89 kB
├ ○ /products/[id]                       3.8 kB        91 kB
└ ...
```

### 에러 시

```
에러 메시지를 읽고 수정!

흔한 에러:
- TypeScript 타입 에러
- import 경로 오류
- 사용하지 않는 변수 (ESLint)
```

### 프로덕션 모드로 테스트

```bash
npm run start
```

`localhost:3000`에서 프로덕션 버전 확인!

---

## 3. GitHub 업로드

### 저장소 생성

1. [github.com](https://github.com) → New repository
2. 정보 입력:
   - Repository name: `my-market-app`
   - Public 선택
3. Create repository

### 코드 푸시

```bash
# Git 초기화 (처음이면)
git init

# 모든 파일 추가
git add .

# 커밋
git commit -m "Complete market app with all features"

# 원격 저장소 연결
git remote add origin https://github.com/사용자명/my-market-app.git

# 푸시
git branch -M main
git push -u origin main
```

### 확인

GitHub 저장소 페이지에서 코드 확인!

---

## 4. Vercel 배포

### Step 1: 프로젝트 import

1. [vercel.com](https://vercel.com) 로그인
2. "Add New..." → "Project"
3. GitHub 저장소에서 `my-market-app` 선택
4. "Import" 클릭

### Step 2: 환경 변수 설정

**Environment Variables** 섹션에서 추가:

| Name | Value |
|------|-------|
| `NEXT_PUBLIC_SUPABASE_URL` | `https://xxx.supabase.co` |
| `NEXT_PUBLIC_SUPABASE_ANON_KEY` | `eyJhbGciOiJI...` |

```
⚠️ 중요!
환경 변수 없으면 Supabase 연결 안 됨!
```

### Step 3: 배포

"Deploy" 버튼 클릭!

### Step 4: 배포 완료

```
🎉 Congratulations!

https://my-market-app-xxx.vercel.app
```

---

## 5. Supabase URL 설정 업데이트

### 왜 필요한가?

```
개발: http://localhost:3000
배포: https://my-market-app-xxx.vercel.app

Supabase Auth 리다이렉트 URL을 업데이트해야 함!
```

### Supabase 대시보드에서

1. Authentication → URL Configuration
2. **Site URL** 수정:
   ```
   https://my-market-app-xxx.vercel.app
   ```
3. **Redirect URLs** 추가:
   ```
   https://my-market-app-xxx.vercel.app/**
   ```

---

## 6. 배포 후 테스트

### 전체 기능 테스트

```
1. 사이트 접속
   □ 홈페이지 정상 로드
   □ 상품 목록 표시

2. 회원 기능
   □ 회원가입 (새 계정)
   □ 로그인
   □ 로그아웃

3. 상품 기능
   □ 상품 등록 (이미지 포함)
   □ 상품 상세 보기
   □ 상품 수정
   □ 상품 삭제

4. 댓글 기능
   □ 댓글 작성
   □ 댓글 삭제

5. 검색/필터
   □ 키워드 검색
   □ 카테고리 필터
   □ 정렬 변경

6. 반응형
   □ 모바일에서 확인 (개발자 도구)
```

### 에러 확인 방법

```
브라우저 개발자 도구 (F12)
→ Console 탭에서 에러 확인
→ Network 탭에서 API 요청 확인
```

---

## 7. 도메인 연결 (선택)

### 커스텀 도메인

```
기본: my-market-app-xxx.vercel.app
커스텀: mymarket.com (별도 구매 필요)
```

### Vercel에서 설정

1. Project Settings → Domains
2. 도메인 입력
3. DNS 설정 안내 따르기

### 무료 대안

```
Vercel 서브도메인 커스터마이징:
my-market-app-xxx.vercel.app
→ mymarket.vercel.app (가능하면)
```

---

## 8. 자동 배포 확인

### Git Push = 자동 배포

```bash
# 코드 수정 후
git add .
git commit -m "Fix: 버그 수정"
git push

# → Vercel이 자동으로 감지하고 재배포!
```

### Vercel 대시보드에서 확인

Deployments 탭에서 배포 히스토리 확인 가능

---

## 9. 배포 문제 해결

### 문제 1: 빌드 실패

```
해결:
1. 로컬에서 npm run build 실행
2. 에러 메시지 확인 및 수정
3. 다시 푸시
```

### 문제 2: 환경 변수 누락

```
증상: 페이지는 뜨는데 데이터가 안 나옴
해결: Vercel → Settings → Environment Variables 확인
```

### 문제 3: 이미지 업로드 안 됨

```
확인:
1. Supabase Storage 버킷 설정
2. RLS 정책 설정
3. 환경 변수 확인
```

### 문제 4: 로그인 안 됨

```
확인:
1. Supabase Auth URL 설정
2. Redirect URLs에 배포 도메인 추가
```

---

## 10. 배포 완료!

### 축하합니다! 🎉

```
여러분은 이제:
✅ AI로 코드를 작성하고
✅ 데이터베이스를 설계하고
✅ 인증 시스템을 구현하고
✅ 파일 업로드를 처리하고
✅ 실제 서비스를 배포할 수 있습니다!

이것이 바로 풀스택 개발입니다!
```

### 완성된 것들

```
📱 투두앱
   - CRUD 기능
   - Supabase DB 연동
   - Vercel 배포

🛒 마켓앱
   - 회원 인증 (가입/로그인)
   - 상품 CRUD
   - 이미지 업로드
   - 댓글 기능
   - 검색 & 필터
   - 반응형 디자인
   - Vercel 배포
```

---

## 핵심 정리

```
✅ npm run build로 배포 전 테스트
✅ GitHub에 코드 업로드
✅ Vercel에서 환경 변수 설정 필수
✅ Supabase URL 설정 업데이트 필요
✅ Git push → 자동 재배포
```

---

## 다음 시간 예고

> 마지막 시간!
> 3일간 배운 내용을 정리하고,
> 앞으로의 학습 방향을 안내합니다.
> Q&A 시간도 있습니다!
