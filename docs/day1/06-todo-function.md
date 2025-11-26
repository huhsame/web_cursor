# Day 1 - 6교시: 투두앱 만들기 (2) - 기능 구현

## 학습 목표
- React의 상태(State)를 이해한다
- useState 훅을 사용한다
- 이벤트 핸들링을 배운다
- 추가, 삭제, 완료 토글 기능을 구현한다

---

## 1. 상태(State)란?

### 쉬운 설명

> 게임 캐릭터를 생각해보세요.
>
> - HP: 100
> - 레벨: 5
> - 골드: 500
>
> 이런 값들은 게임하면서 계속 **변합니다**.
> 몬스터한테 맞으면 HP가 줄고,
> 경험치 모으면 레벨이 오르죠.
>
> React에서 **State**는 이렇게
> "변할 수 있는 값"을 의미합니다.

### 공식적인 설명
```
State는 컴포넌트 내에서 관리되는 동적인 데이터입니다.
State가 변경되면 React가 자동으로
컴포넌트를 다시 렌더링하여 화면을 업데이트합니다.
useState Hook을 사용하여 함수형 컴포넌트에서
State를 관리할 수 있습니다.
```

### 일반 변수 vs State

```tsx
// ❌ 일반 변수 - 화면이 안 바뀜!
let count = 0;

function handleClick() {
  count = count + 1;  // 값은 바뀌지만...
  console.log(count);  // 콘솔엔 보임
  // 화면은 그대로!
}
```

```tsx
// ✅ State - 화면이 바뀜!
const [count, setCount] = useState(0);

function handleClick() {
  setCount(count + 1);  // 값이 바뀌면서...
  // 화면도 자동 업데이트!
}
```

---

## 2. useState 사용법

### 기본 문법

```tsx
const [값, 값변경함수] = useState(초기값);
```

### 예시들

```tsx
// 숫자
const [count, setCount] = useState(0);

// 문자열
const [name, setName] = useState('');

// 불린
const [isOpen, setIsOpen] = useState(false);

// 배열
const [items, setItems] = useState([]);

// 객체
const [user, setUser] = useState({ name: '', age: 0 });
```

### 네이밍 규칙

```
[something, setSomething]
      ↑           ↑
    값 이름    set + 값 이름(첫글자 대문자)

예시:
- [todos, setTodos]
- [inputValue, setInputValue]
- [isLoading, setIsLoading]
```

---

## 3. 투두앱에 State 적용하기

### Step 1: todos 배열 State 만들기

```tsx
'use client';

import { useState } from 'react';

export default function Home() {
  // 할 일 목록 State
  const [todos, setTodos] = useState([
    { id: 1, text: '장보기', completed: false },
    { id: 2, text: '운동하기', completed: true },
    { id: 3, text: '책 읽기', completed: false },
  ]);

  return (
    // ... UI 코드
  );
}
```

### Step 2: 입력값 State 만들기

```tsx
// 입력창에 입력한 값
const [inputValue, setInputValue] = useState('');
```

### 전체 State 구조

```tsx
'use client';

import { useState } from 'react';

export default function Home() {
  // State 선언
  const [todos, setTodos] = useState([
    { id: 1, text: '장보기', completed: false },
    { id: 2, text: '운동하기', completed: true },
    { id: 3, text: '책 읽기', completed: false },
  ]);
  const [inputValue, setInputValue] = useState('');

  return (
    <div className="min-h-screen bg-gray-100 py-10">
      <div className="max-w-md mx-auto bg-white rounded-lg shadow-lg p-6">
        <h1 className="text-2xl font-bold text-center text-gray-800 mb-6">
          📝 나의 할 일 목록
        </h1>

        {/* 입력 영역 */}
        <div className="flex gap-2 mb-6">
          <input
            type="text"
            value={inputValue}  {/* State 연결 */}
            onChange={(e) => setInputValue(e.target.value)}  {/* 입력시 State 업데이트 */}
            placeholder="새로운 할 일을 입력하세요..."
            className="flex-1 border border-gray-300 rounded-lg px-4 py-2 focus:outline-none focus:ring-2 focus:ring-blue-500"
          />
          <button className="bg-blue-500 text-white px-4 py-2 rounded-lg hover:bg-blue-600">
            추가
          </button>
        </div>

        {/* 할 일 목록 - State를 map으로 렌더링 */}
        <ul className="space-y-3">
          {todos.map((todo) => (
            <li key={todo.id} className="flex items-center gap-3 p-3 bg-gray-50 rounded-lg">
              <input
                type="checkbox"
                checked={todo.completed}
                className="w-5 h-5"
              />
              <span className={`flex-1 ${todo.completed ? 'line-through text-gray-400' : ''}`}>
                {todo.text}
              </span>
              <button className="text-red-500 hover:text-red-700">삭제</button>
            </li>
          ))}
        </ul>
      </div>
    </div>
  );
}
```

---

## 4. map으로 목록 렌더링

### 쉬운 설명

> 배열의 각 항목을 화면에 보여주고 싶을 때
> **map**을 사용합니다.
>
> 공장에서 같은 틀로 여러 제품을 찍어내는 것처럼,
> 같은 컴포넌트 구조로 여러 항목을 만들어요.

### map 기본 문법

```tsx
배열.map((항목) => 반환할것)
```

### 예시

```tsx
const fruits = ['사과', '바나나', '오렌지'];

// 배열을 리스트로 변환
fruits.map((fruit) => <li>{fruit}</li>)

// 결과:
// <li>사과</li>
// <li>바나나</li>
// <li>오렌지</li>
```

### key가 필요한 이유

```tsx
{todos.map((todo) => (
  <li key={todo.id}>  {/* 👈 key 필수! */}
    {todo.text}
  </li>
))}
```

```
💡 key란?
   React가 각 항목을 구분하기 위한 고유 식별자입니다.
   key가 없으면 React가 어떤 항목이 바뀌었는지 알기 어려워서
   성능 문제가 생기고 경고가 발생합니다.

   보통 데이터의 id를 key로 사용합니다.
```

---

## 5. 이벤트 핸들링

### 쉬운 설명

> 사용자가 뭔가 하면(클릭, 입력 등)
> 그에 반응하는 것을 **이벤트 핸들링**이라고 합니다.

### 주요 이벤트

| 이벤트 | 언제 발생? | 예시 |
|--------|-----------|------|
| onClick | 클릭할 때 | 버튼 클릭 |
| onChange | 값이 바뀔 때 | 입력창 타이핑 |
| onSubmit | 폼 제출할 때 | 엔터 누를 때 |
| onKeyDown | 키보드 누를 때 | 특정 키 입력 |

### 이벤트 핸들러 작성법

```tsx
// 방법 1: 인라인 함수
<button onClick={() => console.log('클릭!')}>
  클릭
</button>

// 방법 2: 별도 함수 정의
function handleClick() {
  console.log('클릭!');
}

<button onClick={handleClick}>
  클릭
</button>
```

### onChange 예시 (입력창)

```tsx
<input
  value={inputValue}
  onChange={(e) => setInputValue(e.target.value)}
/>
```

```
e = 이벤트 객체
e.target = 이벤트가 발생한 요소 (input)
e.target.value = 그 요소의 현재 값
```

---

## 6. 기능 구현하기

### 기능 1: 할 일 추가

```tsx
// 추가 함수
const addTodo = () => {
  // 빈 입력 방지
  if (inputValue.trim() === '') return;

  // 새 할 일 객체 생성
  const newTodo = {
    id: Date.now(),  // 고유 ID로 현재 시간 사용
    text: inputValue,
    completed: false,
  };

  // 기존 목록에 새 항목 추가
  setTodos([...todos, newTodo]);

  // 입력창 비우기
  setInputValue('');
};
```

**버튼에 연결:**
```tsx
<button
  onClick={addTodo}
  className="bg-blue-500 text-white px-4 py-2 rounded-lg hover:bg-blue-600"
>
  추가
</button>
```

### 스프레드 연산자 설명

```tsx
setTodos([...todos, newTodo]);
          ↑
       스프레드 연산자

// 풀어서 설명하면:
// 기존: todos = [할일1, 할일2]
// 결과: [할일1, 할일2, 새할일]
```

### 기능 2: 할 일 삭제

```tsx
// 삭제 함수
const deleteTodo = (id: number) => {
  // 해당 id가 아닌 것들만 필터링 (= 해당 id 제외)
  setTodos(todos.filter((todo) => todo.id !== id));
};
```

**버튼에 연결:**
```tsx
<button
  onClick={() => deleteTodo(todo.id)}
  className="text-red-500 hover:text-red-700"
>
  삭제
</button>
```

### filter 설명

```tsx
todos.filter((todo) => todo.id !== id)
                            ↑
                      조건이 true인 것만 남김

// 예시: id가 2인 항목 삭제
// 기존: [{id:1}, {id:2}, {id:3}]
// filter: 1 !== 2? true (남김), 2 !== 2? false (제거), 3 !== 2? true (남김)
// 결과: [{id:1}, {id:3}]
```

### 기능 3: 완료 토글

```tsx
// 완료 상태 토글 함수
const toggleTodo = (id: number) => {
  setTodos(
    todos.map((todo) =>
      todo.id === id
        ? { ...todo, completed: !todo.completed }  // 해당 항목의 completed 반전
        : todo  // 다른 항목은 그대로
    )
  );
};
```

**체크박스에 연결:**
```tsx
<input
  type="checkbox"
  checked={todo.completed}
  onChange={() => toggleTodo(todo.id)}
  className="w-5 h-5 cursor-pointer"
/>
```

---

## 7. 전체 완성 코드

```tsx
'use client';

import { useState } from 'react';

interface Todo {
  id: number;
  text: string;
  completed: boolean;
}

export default function Home() {
  const [todos, setTodos] = useState<Todo[]>([
    { id: 1, text: '장보기', completed: false },
    { id: 2, text: '운동하기', completed: true },
    { id: 3, text: '책 읽기', completed: false },
  ]);
  const [inputValue, setInputValue] = useState('');

  // 추가
  const addTodo = () => {
    if (inputValue.trim() === '') return;

    const newTodo: Todo = {
      id: Date.now(),
      text: inputValue,
      completed: false,
    };

    setTodos([...todos, newTodo]);
    setInputValue('');
  };

  // 삭제
  const deleteTodo = (id: number) => {
    setTodos(todos.filter((todo) => todo.id !== id));
  };

  // 완료 토글
  const toggleTodo = (id: number) => {
    setTodos(
      todos.map((todo) =>
        todo.id === id
          ? { ...todo, completed: !todo.completed }
          : todo
      )
    );
  };

  // 엔터 키로 추가
  const handleKeyPress = (e: React.KeyboardEvent) => {
    if (e.key === 'Enter') {
      addTodo();
    }
  };

  return (
    <div className="min-h-screen bg-gray-100 py-10">
      <div className="max-w-md mx-auto bg-white rounded-lg shadow-lg p-6">
        <h1 className="text-2xl font-bold text-center text-gray-800 mb-6">
          📝 나의 할 일 목록
        </h1>

        {/* 입력 영역 */}
        <div className="flex gap-2 mb-6">
          <input
            type="text"
            value={inputValue}
            onChange={(e) => setInputValue(e.target.value)}
            onKeyPress={handleKeyPress}
            placeholder="새로운 할 일을 입력하세요..."
            className="flex-1 border border-gray-300 rounded-lg px-4 py-2 focus:outline-none focus:ring-2 focus:ring-blue-500"
          />
          <button
            onClick={addTodo}
            className="bg-blue-500 text-white px-4 py-2 rounded-lg hover:bg-blue-600 transition"
          >
            추가
          </button>
        </div>

        {/* 할 일 목록 */}
        {todos.length === 0 ? (
          <p className="text-center text-gray-500 py-8">
            할 일이 없습니다. 새로운 할 일을 추가해보세요!
          </p>
        ) : (
          <ul className="space-y-3">
            {todos.map((todo) => (
              <li
                key={todo.id}
                className="flex items-center gap-3 p-3 bg-gray-50 rounded-lg"
              >
                <input
                  type="checkbox"
                  checked={todo.completed}
                  onChange={() => toggleTodo(todo.id)}
                  className="w-5 h-5 cursor-pointer"
                />
                <span
                  className={`flex-1 ${
                    todo.completed ? 'line-through text-gray-400' : ''
                  }`}
                >
                  {todo.text}
                </span>
                <button
                  onClick={() => deleteTodo(todo.id)}
                  className="text-red-500 hover:text-red-700 text-sm"
                >
                  삭제
                </button>
              </li>
            ))}
          </ul>
        )}

        {/* 통계 */}
        {todos.length > 0 && (
          <div className="mt-4 text-sm text-gray-500 text-center">
            총 {todos.length}개 | 완료 {todos.filter(t => t.completed).length}개
          </div>
        )}
      </div>
    </div>
  );
}
```

---

## 8. 추가 기능들

### 기능 4: 엔터 키로 추가

```tsx
const handleKeyPress = (e: React.KeyboardEvent) => {
  if (e.key === 'Enter') {
    addTodo();
  }
};

<input
  onKeyPress={handleKeyPress}
  // ...
/>
```

### 기능 5: 빈 상태 UI

```tsx
{todos.length === 0 ? (
  <p className="text-center text-gray-500 py-8">
    할 일이 없습니다. 새로운 할 일을 추가해보세요!
  </p>
) : (
  <ul>{/* 목록 */}</ul>
)}
```

### 기능 6: 통계 표시

```tsx
<div className="mt-4 text-sm text-gray-500 text-center">
  총 {todos.length}개 |
  완료 {todos.filter(t => t.completed).length}개
</div>
```

---

## 9. 문제점 발견!

### 새로고침하면 데이터가 사라진다!

```
현재 상황:
1. 할 일 추가 ✅
2. 새로고침 (F5)
3. 데이터 초기화됨... 😢

왜?
→ State는 메모리에만 저장됨
→ 새로고침하면 메모리가 초기화됨
→ 영구 저장하려면 데이터베이스 필요!
```

### 내일 배울 것: Supabase

```
Day 2에서 Supabase를 연결해서
데이터를 영구적으로 저장할 거예요!

새로고침해도, 컴퓨터를 꺼도
데이터가 유지됩니다.
```

---

## 10. Day 1 마무리

### 오늘 배운 것

```
✅ 커서 설치 및 사용법
✅ 웹 동작 원리 (클라이언트-서버)
✅ 개발 환경 설정 (Node.js, Git)
✅ Next.js 프로젝트 생성
✅ React 컴포넌트와 JSX
✅ Tailwind CSS 스타일링
✅ State와 useState
✅ 이벤트 핸들링
✅ 투두앱 기능 구현 (추가, 삭제, 토글)
```

### 오늘의 성과

```
🎉 여러분은 이제 동작하는 웹 앱을 만들 수 있습니다!
🎉 AI를 활용해 코드를 작성할 수 있습니다!
🎉 React의 핵심 개념을 이해했습니다!
```

---

## 핵심 정리

```
✅ State = 변할 수 있는 데이터 (useState)
✅ State가 바뀌면 화면이 자동 업데이트
✅ map = 배열을 화면에 렌더링
✅ filter = 조건에 맞는 것만 남기기
✅ 스프레드 연산자 [...] = 배열 복사 + 추가
✅ 이벤트: onClick, onChange, onKeyPress
```

---

## 실습 과제 (선택)

### 과제 1: 수정 기능 추가
할 일 텍스트를 더블클릭하면 수정할 수 있게 만들어보세요.

### 과제 2: 필터 기능
"전체", "완료", "미완료" 버튼으로 목록을 필터링해보세요.

### 과제 3: 로컬 스토리지
`localStorage`를 사용해 새로고침해도 데이터가 유지되게 해보세요.
(힌트: AI에게 "localStorage로 데이터 저장하는 법" 물어보기)

---

## Day 2 예고

```
내일은 진짜 데이터베이스를 연결합니다!

📚 Supabase 소개 및 설정
📚 투두앱에 DB 연결
📚 첫 배포 (Vercel)
📚 마켓앱 설계 시작
📚 회원가입/로그인 구현

내일 만나요! 🙌
```
