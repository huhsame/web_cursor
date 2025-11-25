# Day 1 - 5교시: 투두앱 만들기 (1) - UI 구성

## 학습 목표
- React의 JSX 문법을 이해한다
- 투두앱의 기본 UI를 만든다
- 입력창, 버튼, 리스트를 구현한다
- Tailwind CSS로 스타일링한다

---

## 1. 투두앱 완성 모습 미리보기

```
┌──────────────────────────────────────────────┐
│            📝 나의 할 일 목록                  │
├──────────────────────────────────────────────┤
│  ┌──────────────────────────────┐  ┌──────┐ │
│  │ 새로운 할 일을 입력하세요...    │  │ 추가 │ │
│  └──────────────────────────────┘  └──────┘ │
├──────────────────────────────────────────────┤
│                                              │
│  ☐  장보기                           [삭제]  │
│  ─────────────────────────────────────────  │
│  ☑  운동하기                         [삭제]  │
│  ─────────────────────────────────────────  │
│  ☐  책 읽기                          [삭제]  │
│                                              │
└──────────────────────────────────────────────┘
```

---

## 2. JSX란?

### 쉬운 설명

> HTML처럼 생겼는데, JavaScript 안에서 쓸 수 있어요!
>
> React에서 화면을 만들 때 사용하는 문법입니다.
> HTML + JavaScript = JSX

### 공식적인 설명
```
JSX(JavaScript XML)는 JavaScript를 확장한 문법으로,
React에서 UI를 작성할 때 사용합니다.
브라우저가 이해할 수 있도록 Babel이
일반 JavaScript로 변환합니다.
```

### HTML vs JSX 차이점

| HTML | JSX |
|------|-----|
| `class` | `className` |
| `for` | `htmlFor` |
| `onclick` | `onClick` |
| `style="color: red"` | `style={{color: 'red'}}` |

### JSX 예시

```tsx
// JSX로 작성한 코드
function MyComponent() {
  const name = "홍길동";

  return (
    <div className="container">
      <h1>안녕하세요, {name}님!</h1>
      <button onClick={() => alert('클릭!')}>
        클릭하세요
      </button>
    </div>
  );
}
```

```
💡 중괄호 {}의 의미
   JSX 안에서 JavaScript 코드를 쓸 때 사용합니다.
   변수, 함수, 계산식 등을 넣을 수 있어요.
```

---

## 3. 투두앱 UI 만들기 시작

### Step 1: 기존 코드 정리

`app/page.tsx` 파일을 열고 전체 선택 후 삭제합니다.

### Step 2: AI에게 기본 구조 요청

`Cmd/Ctrl + K`를 누르고 다음과 같이 입력:

```
투두앱의 기본 UI를 만들어줘.

포함할 요소:
1. 제목 "나의 할 일 목록"
2. 입력창과 추가 버튼 (가로로 나란히)
3. 할 일 목록 (임시로 3개 정도)
4. 각 항목에 체크박스와 삭제 버튼

TypeScript + Tailwind CSS 사용.
'use client' 지시어 포함.
일단 기능은 빼고 UI만 만들어줘.
```

### 예상 결과 코드

```tsx
'use client';

export default function Home() {
  return (
    <div className="min-h-screen bg-gray-100 py-10">
      <div className="max-w-md mx-auto bg-white rounded-lg shadow-lg p-6">
        {/* 제목 */}
        <h1 className="text-2xl font-bold text-center text-gray-800 mb-6">
          📝 나의 할 일 목록
        </h1>

        {/* 입력 영역 */}
        <div className="flex gap-2 mb-6">
          <input
            type="text"
            placeholder="새로운 할 일을 입력하세요..."
            className="flex-1 border border-gray-300 rounded-lg px-4 py-2 focus:outline-none focus:ring-2 focus:ring-blue-500"
          />
          <button className="bg-blue-500 text-white px-4 py-2 rounded-lg hover:bg-blue-600 transition">
            추가
          </button>
        </div>

        {/* 할 일 목록 */}
        <ul className="space-y-3">
          {/* 할 일 항목 1 */}
          <li className="flex items-center gap-3 p-3 bg-gray-50 rounded-lg">
            <input type="checkbox" className="w-5 h-5" />
            <span className="flex-1">장보기</span>
            <button className="text-red-500 hover:text-red-700">삭제</button>
          </li>

          {/* 할 일 항목 2 (완료) */}
          <li className="flex items-center gap-3 p-3 bg-gray-50 rounded-lg">
            <input type="checkbox" checked className="w-5 h-5" />
            <span className="flex-1 line-through text-gray-400">운동하기</span>
            <button className="text-red-500 hover:text-red-700">삭제</button>
          </li>

          {/* 할 일 항목 3 */}
          <li className="flex items-center gap-3 p-3 bg-gray-50 rounded-lg">
            <input type="checkbox" className="w-5 h-5" />
            <span className="flex-1">책 읽기</span>
            <button className="text-red-500 hover:text-red-700">삭제</button>
          </li>
        </ul>
      </div>
    </div>
  );
}
```

### Step 3: 저장 및 확인

1. `Cmd/Ctrl + S`로 저장
2. 브라우저에서 `localhost:3000` 확인

---

## 4. 'use client' 이해하기

### 쉬운 설명

> Next.js에서는 기본적으로 서버에서 페이지를 만들어요.
> 하지만 버튼 클릭, 입력 같은 **사용자 상호작용**은
> 브라우저(클라이언트)에서 처리해야 해요.
>
> `'use client'`는 "이 파일은 브라우저에서 실행해줘"라는 표시입니다.

### 공식적인 설명
```
Next.js 13의 App Router는 기본적으로
서버 컴포넌트(Server Components)를 사용합니다.
상태 관리(useState), 이벤트 핸들러(onClick) 등
클라이언트 기능이 필요한 경우
'use client' 지시어를 파일 맨 위에 추가합니다.
```

### 언제 'use client'가 필요한가?

| 필요 O | 필요 X |
|--------|--------|
| useState 사용 | 데이터만 표시 |
| useEffect 사용 | 정적 페이지 |
| onClick 등 이벤트 | API 호출 (서버) |
| 브라우저 API 사용 | 데이터베이스 연결 |

---

## 5. UI 코드 분석하기

### 구조 분해

```tsx
<div className="min-h-screen bg-gray-100 py-10">
  {/* 전체 배경: 최소 화면 높이, 회색 배경, 상하 패딩 */}

  <div className="max-w-md mx-auto bg-white rounded-lg shadow-lg p-6">
    {/* 카드 컨테이너: 최대 너비, 가운데 정렬, 흰 배경, 둥근 모서리, 그림자 */}

    {/* 제목 */}
    {/* 입력 영역 */}
    {/* 할 일 목록 */}
  </div>
</div>
```

### Tailwind 클래스 설명

**컨테이너:**
```
min-h-screen : 최소 높이 = 화면 전체
bg-gray-100  : 배경색 회색 (100 = 밝은)
py-10        : padding-y (상하) = 2.5rem
max-w-md     : 최대 너비 = 28rem (448px)
mx-auto      : margin-x auto = 가운데 정렬
rounded-lg   : 둥근 모서리 (large)
shadow-lg    : 그림자 (large)
```

**입력 영역:**
```
flex         : 플렉스박스 (가로 배치)
gap-2        : 요소 간 간격 = 0.5rem
flex-1       : 남은 공간 모두 차지
focus:ring-2 : 포커스시 링 표시
```

**목록:**
```
space-y-3    : 자식 요소들 사이 세로 간격
items-center : 세로 방향 가운데 정렬
line-through : 취소선
```

---

## 6. 컴포넌트 분리하기

### 왜 분리할까?

> 코드가 길어지면 읽기 어려워져요.
> 관련 있는 코드끼리 묶어서 **컴포넌트**로 분리하면
> 관리하기 쉬워집니다.

### TodoItem 컴포넌트 만들기

**Step 1: 컴포넌트 폴더 생성**

`app` 폴더 안에 `components` 폴더 생성

**Step 2: TodoItem.tsx 파일 생성**

AI에게 요청 (`Cmd/Ctrl + K`):

```
할 일 항목 하나를 표시하는 TodoItem 컴포넌트를 만들어줘.

Props:
- text: 할 일 내용
- completed: 완료 여부

체크박스, 텍스트, 삭제 버튼 포함.
완료되면 텍스트에 취소선.
TypeScript + Tailwind 사용.
```

### 예상 결과

```tsx
// app/components/TodoItem.tsx

interface TodoItemProps {
  text: string;
  completed: boolean;
}

export default function TodoItem({ text, completed }: TodoItemProps) {
  return (
    <li className="flex items-center gap-3 p-3 bg-gray-50 rounded-lg">
      <input
        type="checkbox"
        checked={completed}
        className="w-5 h-5 cursor-pointer"
      />
      <span className={`flex-1 ${completed ? 'line-through text-gray-400' : ''}`}>
        {text}
      </span>
      <button className="text-red-500 hover:text-red-700 text-sm">
        삭제
      </button>
    </li>
  );
}
```

---

## 7. 컴포넌트 사용하기

### page.tsx 수정

```tsx
'use client';

import TodoItem from './components/TodoItem';

export default function Home() {
  return (
    <div className="min-h-screen bg-gray-100 py-10">
      <div className="max-w-md mx-auto bg-white rounded-lg shadow-lg p-6">
        <h1 className="text-2xl font-bold text-center text-gray-800 mb-6">
          📝 나의 할 일 목록
        </h1>

        <div className="flex gap-2 mb-6">
          <input
            type="text"
            placeholder="새로운 할 일을 입력하세요..."
            className="flex-1 border border-gray-300 rounded-lg px-4 py-2 focus:outline-none focus:ring-2 focus:ring-blue-500"
          />
          <button className="bg-blue-500 text-white px-4 py-2 rounded-lg hover:bg-blue-600 transition">
            추가
          </button>
        </div>

        <ul className="space-y-3">
          <TodoItem text="장보기" completed={false} />
          <TodoItem text="운동하기" completed={true} />
          <TodoItem text="책 읽기" completed={false} />
        </ul>
      </div>
    </div>
  );
}
```

### import 이해하기

```tsx
import TodoItem from './components/TodoItem';
```

- `import`: 다른 파일에서 가져오기
- `TodoItem`: 가져올 것의 이름
- `from './components/TodoItem'`: 파일 경로
  - `./` = 현재 폴더
  - `.tsx` 확장자는 생략 가능

---

## 8. Props 이해하기

### 쉬운 설명

> 택배를 보낼 때 상자에 물건을 담잖아요?
>
> **Props**는 부모 컴포넌트가 자식 컴포넌트에게
> 보내는 "데이터 택배"입니다.

### 공식적인 설명
```
Props(Properties)는 부모 컴포넌트에서
자식 컴포넌트로 데이터를 전달하는 방법입니다.
읽기 전용이며, 자식 컴포넌트에서 수정할 수 없습니다.
```

### Props 사용 예시

**부모 (page.tsx):**
```tsx
<TodoItem text="장보기" completed={false} />
          ↑           ↑
        props 전달    props 전달
```

**자식 (TodoItem.tsx):**
```tsx
function TodoItem({ text, completed }) {
                    ↑      ↑
                  props 받기
  return <span>{text}</span>;
                  ↑
                props 사용
}
```

### TypeScript로 Props 타입 정의

```tsx
// Props의 타입을 미리 정의
interface TodoItemProps {
  text: string;      // 문자열
  completed: boolean; // true/false
}

// 타입 적용
function TodoItem({ text, completed }: TodoItemProps) {
  // ...
}
```

```
💡 TypeScript의 장점
   잘못된 타입을 전달하면 에러로 알려줍니다.
   예: text={123} → 에러! (숫자가 아닌 문자열이어야 함)
```

---

## 9. 조건부 스타일링

### 문제: 완료된 항목에만 취소선

```tsx
// 방법 1: 삼항 연산자
<span className={completed ? 'line-through text-gray-400' : ''}>
  {text}
</span>

// 방법 2: 템플릿 리터럴
<span className={`flex-1 ${completed ? 'line-through text-gray-400' : ''}`}>
  {text}
</span>
```

### 삼항 연산자 설명

```
조건 ? 참일때 : 거짓일때

completed ? 'line-through' : ''
    ↓            ↓           ↓
  조건확인     true면     false면
```

### 더 복잡한 조건

```tsx
// 여러 조건
<span className={`
  flex-1
  ${completed ? 'line-through' : ''}
  ${completed ? 'text-gray-400' : 'text-gray-800'}
`}>
```

---

## 10. 실습 과제

### 과제 1: 스타일 커스터마이징

투두앱의 색상을 바꿔보세요:
- 배경색 변경
- 버튼 색상 변경
- 그림자 크기 변경

### 과제 2: 빈 상태 UI 추가

할 일이 없을 때 보여줄 메시지를 추가해보세요:
```
"아직 할 일이 없습니다. 새로운 할 일을 추가해보세요!"
```

### 과제 3: 헤더 컴포넌트 분리

제목 부분을 `Header.tsx` 컴포넌트로 분리해보세요.

---

## 핵심 정리

```
✅ JSX = JavaScript 안에서 HTML처럼 작성하는 문법
✅ 'use client' = 클라이언트에서 실행할 컴포넌트 표시
✅ 컴포넌트 = 재사용 가능한 UI 조각
✅ Props = 부모 → 자식 데이터 전달
✅ 조건부 스타일링 = 삼항 연산자 활용
```

---

## 현재 진행 상황

```
투두앱 개발
├── ✅ UI 구조 완성
├── ✅ 컴포넌트 분리
├── ⬜ 상태(State) 관리 ← 다음 시간!
├── ⬜ 추가 기능
├── ⬜ 삭제 기능
├── ⬜ 완료 토글 기능
└── ⬜ 데이터베이스 연동 (Day 2)
```

---

## 다음 시간 예고

> 지금은 화면만 있고, 클릭해도 아무 반응이 없죠?
> 다음 시간에는 **상태(State)**를 배워서
> 진짜 동작하는 투두앱을 만들어봅니다!
>
> - 할 일 추가
> - 할 일 삭제
> - 완료 체크
