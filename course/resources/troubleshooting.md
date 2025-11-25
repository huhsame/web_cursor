# 문제 해결 가이드 (Troubleshooting)

자주 발생하는 에러와 해결 방법

---

## 설치 & 환경 문제

### "command not found: node"
```
원인: Node.js가 설치되지 않았거나 PATH에 없음

해결:
1. Node.js 재설치 (nodejs.org)
2. 터미널 재시작
3. 컴퓨터 재부팅
```

### "command not found: npm"
```
원인: Node.js 설치 문제

해결:
1. node --version 확인
2. Node.js 재설치
```

### "command not found: git"
```
원인: Git이 설치되지 않음

해결:
- Mac: xcode-select --install
- Windows: git-scm.com에서 설치
```

---

## npm 관련 에러

### "ENOENT: no such file or directory"
```
원인: package.json이 없는 폴더에서 npm 명령 실행

해결:
1. cd 명령어로 올바른 프로젝트 폴더로 이동
2. ls로 package.json 있는지 확인
```

### "npm ERR! code EACCES"
```
원인: 권한 문제

해결 (Mac/Linux):
sudo npm install

또는 (권장):
sudo chown -R $(whoami) ~/.npm
```

### "Module not found: Can't resolve '...'"
```
원인: 패키지가 설치되지 않음

해결:
npm install 패키지명

예:
npm install @supabase/supabase-js
```

### node_modules 문제
```
증상: 이상한 에러가 계속 발생

해결:
rm -rf node_modules
rm package-lock.json
npm install
```

---

## Next.js 에러

### "Error: Cannot find module"
```
원인: import 경로 오류

확인:
1. 파일이 실제로 존재하는지
2. 대소문자가 정확한지 (Mac/Linux는 구분함)
3. 확장자가 맞는지

예:
import { supabase } from '@/lib/supabase';
// @/는 루트 폴더를 의미
```

### "'use client' must be at the top"
```
원인: 'use client'가 파일 최상단에 없음

해결:
파일 첫 줄에 'use client'; 추가 (주석보다 위에)
```

### "Hydration failed"
```
원인: 서버/클라이언트 HTML 불일치

흔한 원인:
- 날짜/시간 표시
- Math.random() 사용
- localStorage 접근

해결:
useEffect 안에서 처리하거나
'use client' 사용
```

### "Invalid hook call"
```
원인: Hook을 잘못된 위치에서 호출

규칙:
- Hook은 컴포넌트 최상위에서만
- 조건문, 반복문 안에서 X
- 일반 함수 안에서 X

잘못된 예:
if (condition) {
  const [state, setState] = useState();  // ❌
}

올바른 예:
const [state, setState] = useState();  // ✅
if (condition) {
  // state 사용
}
```

### 포트 이미 사용 중
```
Error: Port 3000 is already in use

해결:
npm run dev -- -p 3001

또는 기존 프로세스 종료:
Mac: lsof -i :3000 && kill -9 PID
Windows: netstat -ano | findstr :3000
```

---

## Supabase 에러

### "Invalid API key"
```
원인: API 키가 잘못됨

확인:
1. .env.local 파일 존재 여부
2. 키 값이 정확한지 (복사 실수)
3. NEXT_PUBLIC_ 접두사 있는지
4. 개발 서버 재시작 (환경 변수 변경 후 필수!)
```

### "relation does not exist"
```
원인: 테이블이 없음

확인:
1. Supabase 대시보드에서 테이블 확인
2. 테이블 이름 철자 확인
3. 대소문자 확인
```

### "new row violates row-level security"
```
원인: RLS 정책에 의해 차단됨

해결:
1. Supabase → Table Editor → 해당 테이블
2. RLS 정책 확인
3. 필요한 정책 추가

개발 중 임시 해결:
ALTER TABLE 테이블명 DISABLE ROW LEVEL SECURITY;
(프로덕션에서는 사용 금지!)
```

### "JWT expired"
```
원인: 로그인 세션 만료

해결:
await supabase.auth.signOut();
// 다시 로그인 유도
```

### Storage 업로드 실패
```
확인:
1. 버킷이 존재하는지
2. 버킷이 Public인지
3. RLS 정책 설정
4. 파일 크기 제한 (기본 50MB)
5. 파일 타입 제한
```

---

## TypeScript 에러

### "Type 'X' is not assignable to type 'Y'"
```
원인: 타입이 맞지 않음

예:
const [user, setUser] = useState(null);
// user는 null 타입으로 추론됨

해결:
const [user, setUser] = useState<User | null>(null);
```

### "Property 'X' does not exist"
```
원인: 객체에 해당 속성이 없음

확인:
1. 속성 이름 철자
2. 타입 정의에 속성 있는지
3. optional(?)로 정의된 속성인지

해결:
user?.name  // optional chaining
user?.name || '기본값'  // 기본값
```

### "Parameter implicitly has 'any' type"
```
원인: 타입 명시 필요

해결:
// 이전
const handleClick = (e) => {}

// 수정
const handleClick = (e: React.MouseEvent) => {}
```

---

## Git 에러

### "fatal: not a git repository"
```
원인: git init 안 함

해결:
git init
```

### "error: failed to push"
```
원인: 원격 저장소에 변경사항이 있음

해결:
git pull origin main
# 충돌 해결 후
git push
```

### "Please tell me who you are"
```
원인: Git 사용자 설정 안 함

해결:
git config --global user.name "이름"
git config --global user.email "이메일"
```

### 커밋 메시지 수정
```
# 마지막 커밋 메시지 수정
git commit --amend -m "새 메시지"

# 주의: 이미 push한 커밋은 수정하지 않는 게 좋음
```

---

## Vercel 배포 에러

### 빌드 실패
```
확인:
1. 로컬에서 npm run build 성공하는지
2. 에러 메시지 확인
3. 환경 변수 설정 확인

흔한 원인:
- TypeScript 타입 에러
- ESLint 에러
- 환경 변수 누락
```

### 환경 변수 안 됨
```
확인:
1. Vercel → Settings → Environment Variables
2. 변수명이 정확한지
3. NEXT_PUBLIC_ 접두사 (클라이언트용)
4. 설정 후 재배포 필요!
```

### 404 에러
```
확인:
1. 페이지 파일이 존재하는지
2. 파일명이 page.tsx인지
3. 폴더 구조가 올바른지
```

---

## 브라우저 문제

### 변경사항 안 보임
```
해결:
1. 파일 저장 확인 (Cmd/Ctrl + S)
2. 강력 새로고침 (Cmd/Ctrl + Shift + R)
3. 개발 서버 재시작
4. 브라우저 캐시 삭제
```

### 콘솔 에러 확인
```
F12 또는 Cmd + Option + I
→ Console 탭

Network 탭에서 API 요청 확인
```

### CORS 에러
```
원인: 다른 도메인 API 호출 제한

Supabase는 CORS 설정됨 (보통 문제없음)

다른 API면:
1. 해당 API의 CORS 설정 확인
2. 서버 사이드에서 호출 (API Route)
```

---

## 일반적인 디버깅 팁

### console.log 활용
```tsx
console.log('데이터:', data);
console.log('에러:', error);
console.log('user:', user);
```

### 단계별 확인
```
1. API 응답이 오는지
2. 데이터가 올바른지
3. State에 저장되는지
4. 화면에 렌더링되는지
```

### 최소 재현 코드
```
문제가 복잡하면:
1. 새 파일에서 해당 기능만 테스트
2. 하나씩 추가하며 문제 지점 찾기
```

---

## 도움 요청하기

### 좋은 질문 방법
```
1. 무엇을 하려고 했는지
2. 어떤 코드를 작성했는지
3. 어떤 에러가 발생했는지 (전체 메시지)
4. 어떤 해결을 시도했는지
```

### 검색 팁
```
에러 메시지 그대로 검색

예:
"Error: Cannot find module '@/lib/supabase'"
→ Google에 복붙

Stack Overflow, GitHub Issues 확인
```

### AI에게 질문
```
에러 메시지 + 코드를 함께 제공

"이 에러가 발생했어요:
[에러 메시지]

코드:
[관련 코드]

어떻게 해결할 수 있을까요?"
```
