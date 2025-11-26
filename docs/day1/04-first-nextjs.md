# Day 1 - 4교시: 첫 Next.js 프로젝트 만들기

## 학습 목표
- Next.js 프로젝트를 생성한다
- 프로젝트 구조를 이해한다
- 개발 서버를 실행하고 결과를 확인한다
- 첫 번째 페이지를 수정해본다

---

## 1. Next.js란?

### 쉬운 설명

> React로 웹사이트 만들려면 이것저것 설정할 게 많아요.
> 라우팅, 서버, 최적화, 빌드...
>
> **Next.js**는 이 모든 걸 미리 세팅해놓은
> "올인원 패키지"입니다.
>
> 레고로 치면, 그냥 레고 블록이 아니라
> 설명서까지 포함된 "레고 세트"인 거죠!

### 공식적인 설명
```
Next.js는 Vercel에서 만든 React 프레임워크입니다.
서버 사이드 렌더링(SSR), 정적 사이트 생성(SSG),
파일 기반 라우팅, API 라우트 등을 제공합니다.
풀스택 웹 애플리케이션을 쉽게 만들 수 있습니다.
```

### Next.js를 쓰는 이유

| 기능 | 설명 |
|------|------|
| **파일 기반 라우팅** | 폴더 만들면 자동으로 URL 생성 |
| **풀스택** | 프론트엔드 + 백엔드 한 번에 |
| **빠른 성능** | 자동 최적화 |
| **쉬운 배포** | Vercel과 완벽 호환 |

---

## 2. 프로젝트 생성하기

### Step 1: 커서에서 폴더 열기

1. 커서(Cursor) 실행
2. `File` → `Open Folder`
3. 바탕화면(Desktop) 선택

### Step 2: 터미널 열기

커서에서 터미널 열기:
- 단축키: `` Ctrl + ` `` (백틱)
- 또는: `View` → `Terminal`

### Step 3: 프로젝트 생성 명령어

```bash
npx create-next-app@latest my-todo-app
```

### Step 4: 옵션 선택

질문이 나오면 아래처럼 선택하세요:

```
✔ Would you like to use TypeScript? … Yes
✔ Would you like to use ESLint? … Yes
✔ Would you like to use Tailwind CSS? … Yes
✔ Would you like to use `src/` directory? … No
✔ Would you like to use App Router? … Yes
✔ Would you like to customize the default import alias? … No
```

**각 옵션 설명:**

| 옵션 | 선택 | 이유 |
|------|------|------|
| TypeScript | Yes | 타입으로 에러 방지 |
| ESLint | Yes | 코드 품질 검사 |
| Tailwind CSS | Yes | 스타일링 쉽게 |
| src/ directory | No | 구조 단순하게 |
| App Router | Yes | 최신 방식 |

### Step 5: 프로젝트 폴더로 이동

```bash
cd my-todo-app
```

### Step 6: 커서에서 프로젝트 열기

```
File → Open Folder → my-todo-app 선택
```

---

## 3. 프로젝트 구조 이해하기

### 폴더 구조

```
my-todo-app/
├── app/                    # 👈 메인 코드가 들어가는 곳
│   ├── favicon.ico         # 탭에 표시되는 아이콘
│   ├── globals.css         # 전체 스타일
│   ├── layout.tsx          # 공통 레이아웃
│   └── page.tsx            # 👈 메인 페이지 (중요!)
├── public/                 # 이미지 등 정적 파일
├── node_modules/           # 설치된 패키지들 (건드리지 마세요)
├── package.json            # 프로젝트 설정 파일
├── tailwind.config.ts      # Tailwind 설정
└── tsconfig.json           # TypeScript 설정
```

### 중요한 파일들

**1. `app/page.tsx` - 메인 페이지**
```
웹사이트의 첫 화면입니다.
https://내사이트.com/ 에 접속하면 보이는 페이지
```

**2. `app/layout.tsx` - 레이아웃**
```
모든 페이지에 공통으로 적용되는 틀입니다.
헤더, 푸터 같은 것을 여기에 넣습니다.
```

**3. `package.json` - 프로젝트 정보**
```
프로젝트 이름, 사용하는 패키지 목록,
실행 명령어 등이 적혀있습니다.
```

---

## 4. 개발 서버 실행하기

### 명령어 실행

```bash
npm run dev
```

### 예상 결과

```
  ▲ Next.js 14.x.x
  - Local:        http://localhost:3000

 ✓ Ready in 2.3s
```

### 브라우저에서 확인

1. 브라우저 열기
2. 주소창에 `localhost:3000` 입력
3. Next.js 기본 페이지가 보이면 성공!

```
💡 localhost란?
   "내 컴퓨터"를 의미하는 특별한 주소입니다.
   localhost:3000은 "내 컴퓨터의 3000번 포트"라는 뜻이에요.
```

### 개발 서버 종료

터미널에서 `Ctrl + C` 누르면 종료됩니다.

---

## 5. 첫 페이지 수정하기

### Step 1: page.tsx 열기

커서에서 `app/page.tsx` 파일을 열어보세요.

복잡한 코드가 보일 거예요. 걱정 마세요!

### Step 2: AI에게 수정 요청

1. `app/page.tsx` 파일 전체 선택 (`Cmd/Ctrl + A`)
2. `Cmd/Ctrl + K` 누르기
3. 다음과 같이 입력:

```
이 파일의 내용을 전부 지우고,
"안녕하세요! 나의 첫 Next.js 앱입니다"라는
텍스트만 화면 중앙에 크게 보여주는
심플한 페이지로 바꿔줘.
Tailwind CSS 사용해서 스타일링 해줘.
```

### 예상 결과 코드

```tsx
export default function Home() {
  return (
    <main className="flex min-h-screen items-center justify-center">
      <h1 className="text-4xl font-bold">
        안녕하세요! 나의 첫 Next.js 앱입니다
      </h1>
    </main>
  );
}
```

### Step 3: 결과 확인

브라우저를 새로고침하면 변경된 내용이 보입니다!

```
💡 Hot Reload (핫 리로드)
   Next.js는 파일을 저장하면 자동으로
   브라우저가 새로고침됩니다.
   따로 새로고침 안 해도 돼요!
```

---

## 6. 컴포넌트 이해하기

### 쉬운 설명

> 레고 블록을 생각해보세요.
>
> 작은 블록들을 조합해서 큰 구조물을 만들죠?
>
> **컴포넌트**도 마찬가지입니다.
> 작은 UI 조각들을 조합해서
> 전체 페이지를 만듭니다.

### 컴포넌트 예시

```tsx
// 버튼 컴포넌트
function MyButton() {
  return <button>클릭하세요</button>;
}

// 카드 컴포넌트
function MyCard() {
  return (
    <div>
      <h2>제목</h2>
      <p>내용</p>
      <MyButton />  {/* 버튼 컴포넌트 사용 */}
    </div>
  );
}

// 페이지에서 카드 컴포넌트 사용
export default function Home() {
  return (
    <main>
      <MyCard />
      <MyCard />
      <MyCard />
    </main>
  );
}
```

### 컴포넌트의 장점

```
1. 재사용: 한 번 만들면 여러 곳에서 사용
2. 관리 용이: 수정하면 사용한 모든 곳에 반영
3. 가독성: 코드가 깔끔해짐
```

---

## 7. Tailwind CSS 기초

### 쉬운 설명

> 예전에는 CSS 파일에 스타일을 따로 작성했어요.
>
> ```css
> .title {
>   font-size: 24px;
>   font-weight: bold;
>   color: blue;
> }
> ```
>
> **Tailwind**는 미리 만들어진 클래스를
> HTML에 바로 적용하는 방식이에요.
>
> ```html
> <h1 class="text-2xl font-bold text-blue-500">제목</h1>
> ```

### 자주 쓰는 Tailwind 클래스

**텍스트:**
| 클래스 | 효과 |
|--------|------|
| `text-sm` | 작은 글씨 |
| `text-lg` | 큰 글씨 |
| `text-2xl` | 더 큰 글씨 |
| `font-bold` | 굵은 글씨 |
| `text-center` | 가운데 정렬 |

**색상:**
| 클래스 | 효과 |
|--------|------|
| `text-red-500` | 빨간 글씨 |
| `bg-blue-500` | 파란 배경 |
| `text-gray-700` | 회색 글씨 |

**여백:**
| 클래스 | 효과 |
|--------|------|
| `p-4` | 안쪽 여백 (padding) |
| `m-4` | 바깥 여백 (margin) |
| `px-4` | 좌우 padding |
| `py-4` | 상하 padding |

**레이아웃:**
| 클래스 | 효과 |
|--------|------|
| `flex` | 플렉스박스 |
| `justify-center` | 가로 중앙 |
| `items-center` | 세로 중앙 |
| `gap-4` | 요소 간 간격 |

### 실습: 스타일 적용해보기

```tsx
export default function Home() {
  return (
    <main className="flex min-h-screen flex-col items-center justify-center bg-gray-100">
      <h1 className="text-4xl font-bold text-blue-600 mb-4">
        환영합니다!
      </h1>
      <p className="text-lg text-gray-600">
        나의 첫 Next.js 웹사이트입니다.
      </p>
      <button className="mt-4 bg-blue-500 text-white px-6 py-2 rounded-lg hover:bg-blue-600">
        시작하기
      </button>
    </main>
  );
}
```

---

## 8. 새 페이지 만들기

### Next.js App Router의 규칙

```
app/page.tsx         → https://사이트.com/
app/about/page.tsx   → https://사이트.com/about
app/contact/page.tsx → https://사이트.com/contact
```

**폴더 이름 = URL 경로**

### 실습: About 페이지 만들기

**Step 1: 폴더 생성**

커서에서 `app` 폴더 우클릭 → `New Folder` → `about`

**Step 2: AI로 페이지 생성**

1. `app/about` 폴더 우클릭 → `New File` → `page.tsx`
2. 파일을 열고 `Cmd/Ctrl + K` 누르기
3. 다음과 같이 입력:

```
자기소개 페이지를 만들어줘.
내 이름, 취미, 좋아하는 것을 보여주는 페이지.
예쁘게 Tailwind로 스타일링 해줘.
```

**Step 3: 확인**

브라우저에서 `localhost:3000/about` 접속

---

## 9. 네비게이션 추가하기

### Link 컴포넌트

```tsx
import Link from 'next/link';

export default function Home() {
  return (
    <main className="p-8">
      <nav className="flex gap-4 mb-8">
        <Link href="/" className="text-blue-500 hover:underline">
          홈
        </Link>
        <Link href="/about" className="text-blue-500 hover:underline">
          소개
        </Link>
      </nav>

      <h1 className="text-4xl font-bold">홈 페이지</h1>
    </main>
  );
}
```

### a 태그 vs Link 컴포넌트

| `<a>` 태그 | `<Link>` 컴포넌트 |
|-----------|------------------|
| 페이지 전체 새로고침 | 필요한 부분만 업데이트 |
| 느림 | 빠름 |
| 외부 링크에 사용 | 내부 페이지 이동에 사용 |

---

## 10. 실습 과제

### 과제 1: 페이지 꾸미기

메인 페이지(`app/page.tsx`)를 다음 요소들로 꾸며보세요:
- 제목
- 간단한 설명
- 버튼 2개 (서로 다른 색상)

### 과제 2: 새 페이지 만들기

`app/contact/page.tsx`를 만들어서:
- 이메일 주소
- 전화번호
- SNS 링크

를 보여주는 연락처 페이지를 만들어보세요.

### 과제 3: 네비게이션 연결

모든 페이지에 네비게이션을 추가해서
페이지 간 이동이 가능하게 만들어보세요.

---

## 핵심 정리

```
✅ npx create-next-app@latest 로 프로젝트 생성
✅ npm run dev 로 개발 서버 실행
✅ app/page.tsx가 메인 페이지
✅ 폴더 이름 = URL 경로
✅ Tailwind로 클래스 기반 스타일링
✅ Link 컴포넌트로 페이지 이동
```

---

## 자주 발생하는 문제

### "npm run dev" 안 됨
```bash
# node_modules 삭제 후 재설치
rm -rf node_modules
npm install
npm run dev
```

### 포트 이미 사용 중
```bash
# 다른 포트로 실행
npm run dev -- -p 3001
```

### 변경사항이 안 보임
```
1. 파일 저장했는지 확인 (Cmd/Ctrl + S)
2. 브라우저 강력 새로고침 (Cmd/Ctrl + Shift + R)
3. 개발 서버 재시작
```

---

## 다음 시간 예고

> 드디어 투두앱 만들기 시작!
> 할 일을 추가하고, 완료 체크하고, 삭제하는
> 기능을 구현해봅니다.
> React의 상태(State) 개념을 배웁니다!
