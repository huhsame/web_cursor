# Day 2 - 3교시: 첫 배포! Vercel로 세상에 공개하기

## 학습 목표
- GitHub에 코드를 업로드한다
- Vercel에서 프로젝트를 배포한다
- 환경 변수를 설정한다
- 실제 URL로 접속해본다

---

## 1. 배포란?

### 쉬운 설명

> 지금까지 우리 앱은 `localhost:3000`에서만 돌아갔어요.
> 이건 "내 컴퓨터에서만 볼 수 있다"는 뜻이에요.
>
> **배포**는 우리 앱을 인터넷에 올려서
> 전 세계 누구나 접속할 수 있게 만드는 것입니다.
>
> 유튜브에 영상 업로드하면 누구나 볼 수 있는 것처럼요!

### 공식적인 설명
```
배포(Deployment)는 개발한 애플리케이션을
실제 사용자가 접근할 수 있는 서버 환경에
설치하고 실행하는 과정입니다.
Vercel은 Next.js 애플리케이션의 배포를
간편하게 해주는 클라우드 플랫폼입니다.
```

---

## 2. 배포 과정 미리보기

```
[내 컴퓨터]          [GitHub]           [Vercel]          [사용자]
    │                  │                   │                 │
    │  1. git push     │                   │                 │
    │─────────────────▶│                   │                 │
    │                  │  2. 자동 감지      │                 │
    │                  │──────────────────▶│                 │
    │                  │                   │  3. 빌드        │
    │                  │                   │  4. 배포        │
    │                  │                   │                 │
    │                  │                   │  5. URL 생성    │
    │                  │                   │◀────────────────│
    │                  │                   │  https://...    │
    │                  │                   │────────────────▶│
    │                  │                   │                 │
                                           실제 서비스 접속!
```

---

## 3. GitHub에 코드 업로드

### Step 1: GitHub 저장소 생성

1. [github.com](https://github.com) 접속 (로그인)
2. 오른쪽 위 "+" → "New repository"
3. 정보 입력:

```
Repository name: my-todo-app
Description: 투두앱 프로젝트
Public 선택 (무료 배포를 위해)
☐ Add a README file (체크 안 함)
```

4. "Create repository" 클릭

### Step 2: 로컬 프로젝트와 연결

터미널에서:

```bash
# 프로젝트 폴더로 이동
cd my-todo-app

# Git 초기화 (이미 했다면 생략)
git init

# 모든 파일 스테이징
git add .

# 첫 커밋
git commit -m "Initial commit: Todo app with Supabase"

# GitHub 저장소 연결
git remote add origin https://github.com/사용자명/my-todo-app.git

# 메인 브랜치 설정 및 푸시
git branch -M main
git push -u origin main
```

### Step 3: 업로드 확인

GitHub 저장소 페이지를 새로고침하면 코드가 보입니다!

```
⚠️ .env.local은 업로드되면 안 됩니다!
   .gitignore에 자동으로 포함되어 있어서
   GitHub에 올라가지 않습니다.
   (API 키 노출 방지)
```

---

## 4. Vercel 계정 만들기

### Step 1: 회원가입

1. [vercel.com](https://vercel.com) 접속
2. "Sign Up" 클릭
3. **"Continue with GitHub"** 선택 (가장 간편!)
4. GitHub 권한 승인

### Vercel이 좋은 이유

```
✅ Next.js를 만든 회사 (완벽 호환)
✅ 무료 티어가 넉넉함
✅ GitHub 연동으로 자동 배포
✅ HTTPS 자동 적용
✅ 전 세계 CDN
✅ 커스텀 도메인 지원
```

---

## 5. Vercel에서 배포하기

### Step 1: 새 프로젝트 생성

1. Vercel 대시보드에서 "Add New..." → "Project"
2. "Import Git Repository" 섹션에서
3. **my-todo-app** 저장소 찾아서 "Import" 클릭

### Step 2: 프로젝트 설정

```
Project Name: my-todo-app (자동)
Framework Preset: Next.js (자동 감지)
Root Directory: ./ (기본값)
Build Command: npm run build (기본값)
Output Directory: (비워두기)
Install Command: npm install (기본값)
```

### Step 3: 환경 변수 설정 (중요!)

"Environment Variables" 섹션 펼치기

**추가할 환경 변수:**

| Name | Value |
|------|-------|
| NEXT_PUBLIC_SUPABASE_URL | https://xxx.supabase.co |
| NEXT_PUBLIC_SUPABASE_ANON_KEY | eyJhbGciOiJIUzI1NiIsInR5cCI6... |

```
⚠️ 중요!
환경 변수를 설정하지 않으면
배포된 앱에서 Supabase 연결이 안 됩니다!
```

### Step 4: 배포 시작

"Deploy" 버튼 클릭!

### Step 5: 빌드 과정 지켜보기

```
Building...
├── Installing dependencies
├── Building application
├── Collecting page data
├── Generating static pages
└── Finalizing page optimization

✓ Build completed!
```

**1-2분 정도 소요됩니다.**

---

## 6. 배포 완료!

### 성공 화면

```
🎉 Congratulations!
Your project has been successfully deployed.

https://my-todo-app-xxxxx.vercel.app
         ↑
     여러분의 URL!
```

### 접속해보기

1. 생성된 URL 클릭
2. 투두앱이 보이면 성공!
3. 할 일 추가해보기
4. 새로고침해도 데이터 유지되는지 확인

```
🎉 축하합니다!
   여러분의 앱이 전 세계에 공개되었습니다!
   이 URL을 친구에게 공유해보세요!
```

---

## 7. 자동 배포 이해하기

### Git Push = 자동 배포

```
Vercel의 마법:
코드를 GitHub에 푸시하면 → 자동으로 재배포!

1. 코드 수정
2. git add .
3. git commit -m "메시지"
4. git push
5. ✨ Vercel이 자동 감지하고 배포 시작!
```

### 실습: 수정 후 재배포

**Step 1:** `app/page.tsx`에서 제목 수정

```tsx
<h1>📝 나의 할 일 목록</h1>
// ↓ 변경
<h1>📝 OOO의 할 일 목록</h1>  // 본인 이름으로!
```

**Step 2:** 커밋 & 푸시

```bash
git add .
git commit -m "Update: 제목에 이름 추가"
git push
```

**Step 3:** Vercel 대시보드 확인

- "Deployments" 탭에서 새 배포 진행 중 확인
- 완료 후 URL 접속해서 변경 확인!

---

## 8. 배포 문제 해결

### 문제 1: 빌드 실패

```
Error: Build failed

해결:
1. 로컬에서 npm run build 실행해보기
2. 에러 메시지 확인
3. 코드 수정 후 다시 푸시
```

### 문제 2: 환경 변수 누락

```
Error: supabaseUrl is required

해결:
1. Vercel 대시보드 → Settings → Environment Variables
2. 환경 변수 추가/확인
3. "Redeploy" 버튼 클릭
```

### 문제 3: 데이터가 안 보임

```
원인: Supabase 연결 문제

확인사항:
1. 환경 변수가 정확한지
2. Supabase RLS 설정이 되어있는지
3. 브라우저 콘솔에서 에러 확인
```

### 재배포 방법

```
Vercel 대시보드 → Deployments → ... → Redeploy
```

---

## 9. Vercel 대시보드 둘러보기

### 주요 탭

```
Project Dashboard
├── Overview: 프로젝트 개요, 최근 배포
├── Deployments: 배포 기록
├── Analytics: 방문자 통계 (Pro)
├── Speed Insights: 성능 분석
├── Logs: 실시간 로그
├── Storage: (KV, Blob 등)
└── Settings: 프로젝트 설정
```

### Settings에서 할 수 있는 것

```
- 도메인 설정 (커스텀 도메인)
- 환경 변수 관리
- 빌드 설정
- Git 연동 설정
```

---

## 10. 도메인 설정 (선택)

### 기본 도메인

```
https://my-todo-app-xxxxx.vercel.app
         └──────────┬──────────┘
           Vercel이 자동 생성
```

### 커스텀 도메인 연결 (유료 도메인 필요)

1. Settings → Domains
2. 도메인 입력: `mytodo.com`
3. DNS 설정 안내에 따라 설정

### 무료 서브도메인 변경

```
Settings → Domains → Edit
my-todo-app-xxxxx.vercel.app
→ my-todo.vercel.app (가능하면)
```

---

## 핵심 정리

```
✅ 배포 = 앱을 인터넷에 공개하는 것
✅ GitHub에 코드 업로드 → Vercel 자동 배포
✅ 환경 변수는 Vercel에도 설정 필요!
✅ git push 하면 자동 재배포
✅ 실제 URL로 전 세계 어디서나 접속 가능
```

---

## 배포 체크리스트

```
✅ GitHub 저장소 생성
✅ 로컬 코드 푸시
✅ Vercel 계정 생성
✅ 프로젝트 import
✅ 환경 변수 설정
✅ 배포 성공
✅ URL 접속 확인
✅ 데이터 정상 동작 확인
```

---

## 축하합니다! 🎉

```
여러분은 이제:
✅ 웹 앱을 만들 수 있고
✅ 데이터베이스를 연결할 수 있고
✅ 전 세계에 배포할 수 있습니다!

이것이 바로 "풀스택 개발"의 시작입니다!
```

---

## 다음 시간 예고

> 이제 더 큰 프로젝트를 시작합니다!
> 중고거래 마켓앱을 설계하고
> 데이터베이스 구조를 잡아봅니다.
