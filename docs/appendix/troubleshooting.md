# 문제 해결 가이드

에러가 발생했을 때 이 가이드를 참고하세요.

---

## 1. 에러 읽는 법

### 에러 메시지 구조

```
Error: Cannot find module '@supabase/supabase-js'
       └───────────────────┬───────────────────┘
                      에러 내용
```

**핵심:** 에러 메시지를 **그대로** 읽으면 해결책이 보입니다!

### 흔한 에러 메시지 해석

| 메시지 | 의미 | 해결책 |
|--------|------|--------|
| Cannot find module | 패키지가 없음 | `npm install` |
| is not defined | 변수/함수가 정의 안 됨 | import 확인 |
| Unexpected token | 문법 오류 | 괄호, 세미콜론 확인 |
| 404 Not Found | 페이지가 없음 | URL, 파일명 확인 |
| 500 Internal Server Error | 서버 오류 | 서버 로그 확인 |

---

## 2. 설치 문제

### "npm: command not found"

**원인:** Node.js가 설치 안 됨

**해결:**
1. [nodejs.org](https://nodejs.org) 에서 Node.js 설치
2. LTS 버전 선택
3. 터미널 새로 열기

### "npm install 에러"

**해결 방법들:**

```bash
# 1. 캐시 삭제
npm cache clean --force

# 2. node_modules 삭제 후 재설치
rm -rf node_modules
npm install

# 3. package-lock.json도 삭제
rm package-lock.json
rm -rf node_modules
npm install
```

### "npx create-next-app 실패"

```bash
# npm 업데이트
npm install -g npm

# 다시 시도
npx create-next-app@latest my-app
```

---

## 3. 개발 서버 문제

### "Port 3000 is already in use"

**원인:** 다른 프로그램이 3000 포트 사용 중

**해결:**

```bash
# 다른 포트 사용
npm run dev -- -p 3001

# 또는 해당 프로세스 종료 (Mac/Linux)
lsof -i :3000
kill -9 [PID]

# Windows
netstat -ano | findstr :3000
taskkill /PID [PID] /F
```

### "Module not found"

**원인:** 패키지가 설치 안 됨

**해결:**
```bash
npm install [패키지명]

# 예시
npm install @supabase/supabase-js
```

### 페이지가 하얗게만 나와요

1. 브라우저 개발자 도구(F12) → Console 탭 확인
2. 터미널에서 에러 확인
3. 에러 메시지를 AI에게 보여주기

---

## 4. Supabase 문제

### "Invalid API key"

**원인:** API 키가 잘못됨

**해결:**
1. Supabase 대시보드 → Settings → API
2. 키 다시 복사
3. `.env.local` 파일 수정
4. 서버 재시작

### "relation does not exist"

**원인:** 테이블이 없음

**해결:**
1. Supabase Table Editor에서 테이블 확인
2. SQL Editor에서 테이블 생성 쿼리 실행

### "new row violates row-level security policy"

**원인:** RLS 정책 위반

**해결:**
1. 로그인 상태 확인
2. user_id가 제대로 들어가는지 확인
3. RLS 정책 확인

```sql
-- 현재 정책 확인
SELECT * FROM pg_policies WHERE tablename = 'products';
```

### 데이터가 저장 안 돼요

1. `.env.local` 확인
2. 서버 재시작 (`Ctrl + C` → `npm run dev`)
3. Supabase 대시보드에서 직접 확인
4. 브라우저 콘솔에서 에러 확인

---

## 5. 인증 문제

### 로그인이 안 돼요

1. Supabase → Authentication → Users 확인
2. 이메일/비밀번호 확인
3. 비밀번호 6자 이상인지 확인

### 회원가입 후 자동 로그인이 안 돼요

**Supabase 설정 확인:**
1. Authentication → Providers → Email
2. "Confirm email" 옵션이 OFF인지 확인 (개발 중)

### 로그인 상태가 유지 안 돼요

**AuthContext 확인:**
- `onAuthStateChange`가 제대로 설정됐는지
- 페이지 새로고침 시 세션 복원이 되는지

---

## 6. GitHub 문제

### "fatal: not a git repository"

**원인:** Git 저장소가 아님

**해결:**
```bash
git init
```

### "error: failed to push"

**원인:** 원격 저장소 설정 안 됨 또는 권한 없음

**해결:**
```bash
# 원격 저장소 확인
git remote -v

# 원격 저장소 추가
git remote add origin https://github.com/유저명/저장소명.git

# 다시 push
git push -u origin main
```

### "Please commit your changes or stash them"

**원인:** 커밋 안 한 변경사항이 있음

**해결:**
```bash
# 변경사항 커밋
git add .
git commit -m "메시지"

# 또는 변경사항 버리기
git checkout -- .
```

---

## 7. 배포 문제

### Vercel 빌드 실패

1. **Logs** 탭에서 에러 확인
2. 로컬에서 빌드 테스트:
   ```bash
   npm run build
   ```
3. 에러 수정 후 다시 push

### "Error: Missing env variables"

**해결:**
1. Vercel 프로젝트 → Settings → Environment Variables
2. 필요한 환경변수 추가
3. **Redeploy** 클릭

### 배포는 됐는데 화면이 다르게 보여요

1. 브라우저 캐시 삭제
2. 강력 새로고침 (Ctrl + Shift + R)
3. 환경변수 확인

---

## 8. TypeScript 에러

### "Property does not exist on type"

**원인:** 타입에 해당 속성이 없음

**해결:**
```typescript
// 타입 정의 추가
interface Product {
  id: number;
  title: string;
  price: number;
  // 필요한 속성 추가
}
```

### "Type 'null' is not assignable"

**원인:** null 가능성 처리 안 됨

**해결:**
```typescript
// 옵셔널 체이닝 사용
user?.name

// 또는 기본값 설정
const name = user?.name ?? '익명'
```

---

## 9. 일반적인 해결 순서

문제가 생겼을 때 이 순서로 시도하세요:

### Step 1: 에러 메시지 읽기
```
에러 메시지가 뭐라고 하는지 확인
```

### Step 2: 검색하기
```
에러 메시지를 구글에 검색
```

### Step 3: AI에게 질문
```
이 에러가 나는데 해결해줘:
[에러 메시지]
```

### Step 4: 재시작
```
- 터미널 껐다 켜기
- 브라우저 새로고침
- 컴퓨터 재시작
```

### Step 5: 처음부터 다시
```
- 폴더 삭제하고 새로 시작
- 강사에게 도움 요청
```

---

## 10. 디버깅 팁

### console.log 활용

```typescript
console.log('여기까지 왔음');
console.log('데이터:', data);
console.log('에러:', error);
```

### 브라우저 개발자 도구 (F12)

- **Console**: 에러 메시지 확인
- **Network**: API 호출 확인
- **Application**: 로컬 스토리지, 쿠키 확인

### 단계별로 확인

```
1. 데이터를 가져오는 게 문제인가?
2. 데이터를 저장하는 게 문제인가?
3. 화면에 표시하는 게 문제인가?
```

---

문제가 계속되면 강사에게 질문하세요!

에러 메시지, 현재 상황, 시도해본 것을 함께 알려주시면
더 빨리 해결할 수 있습니다.

