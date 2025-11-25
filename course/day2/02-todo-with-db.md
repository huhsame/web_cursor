# Day 2 - 2교시: 투두앱에 DB 연결하기

## 학습 목표
- Supabase에서 데이터를 CRUD한다
- 투두앱을 데이터베이스와 연동한다
- 비동기 처리(async/await)를 이해한다
- 에러 처리 방법을 배운다

---

## 1. CRUD 복습

### CRUD란?

| 영어 | 한글 | Supabase 메서드 |
|------|------|----------------|
| **C**reate | 생성 | `.insert()` |
| **R**ead | 조회 | `.select()` |
| **U**pdate | 수정 | `.update()` |
| **D**elete | 삭제 | `.delete()` |

### 기본 문법 미리보기

```typescript
// 조회
const { data } = await supabase.from('todos').select('*');

// 추가
await supabase.from('todos').insert({ text: '새 할일', completed: false });

// 수정
await supabase.from('todos').update({ completed: true }).eq('id', 1);

// 삭제
await supabase.from('todos').delete().eq('id', 1);
```

---

## 2. 비동기 처리 이해하기

### 쉬운 설명

> 식당에서 음식을 주문하면 바로 안 나오죠?
> 요리하는 동안 기다려야 합니다.
>
> 데이터베이스도 마찬가지예요.
> "데이터 주세요"라고 요청하면
> 서버가 찾아서 보내주는 데 시간이 걸립니다.
>
> **async/await**는 "기다리는 방법"입니다.

### 공식적인 설명
```
JavaScript에서 비동기 처리란
시간이 걸리는 작업(API 호출, DB 접근 등)을
기다리는 동안 다른 작업을 수행할 수 있게 하는 것입니다.
async/await는 Promise 기반 비동기 코드를
동기 코드처럼 읽기 쉽게 작성하는 문법입니다.
```

### 동기 vs 비동기

```javascript
// 동기 (순서대로, 기다림)
console.log('1');
console.log('2');  // 1이 끝나야 실행
console.log('3');  // 2가 끝나야 실행

// 비동기 (기다리지 않음)
console.log('1');
setTimeout(() => console.log('2'), 1000);  // 1초 후 실행
console.log('3');  // 기다리지 않고 바로 실행
// 결과: 1 → 3 → 2
```

### async/await 사용법

```typescript
// async 함수 선언
async function fetchData() {
  // await = 기다려!
  const result = await supabase.from('todos').select('*');
  console.log(result);  // 데이터 도착 후 실행
}
```

```
💡 규칙
1. await는 async 함수 안에서만 사용 가능
2. await 뒤에는 Promise가 와야 함
3. await를 만나면 결과가 올 때까지 기다림
```

---

## 3. 데이터 조회하기 (Read)

### 기본 조회

```typescript
// 모든 데이터 조회
const { data, error } = await supabase
  .from('todos')      // todos 테이블에서
  .select('*');       // 모든 컬럼 선택

// data: 결과 데이터 배열
// error: 에러 객체 (없으면 null)
```

### 조건부 조회

```typescript
// 완료된 것만
const { data } = await supabase
  .from('todos')
  .select('*')
  .eq('completed', true);  // completed가 true인 것만

// 특정 ID
const { data } = await supabase
  .from('todos')
  .select('*')
  .eq('id', 1);  // id가 1인 것
```

### 정렬

```typescript
// 최신순 정렬
const { data } = await supabase
  .from('todos')
  .select('*')
  .order('created_at', { ascending: false });  // 내림차순
```

### 투두앱에 적용

```tsx
'use client';

import { useEffect, useState } from 'react';
import { supabase } from '@/lib/supabase';

interface Todo {
  id: number;
  text: string;
  completed: boolean;
  created_at: string;
}

export default function Home() {
  const [todos, setTodos] = useState<Todo[]>([]);
  const [loading, setLoading] = useState(true);

  // 컴포넌트가 처음 렌더링될 때 데이터 조회
  useEffect(() => {
    fetchTodos();
  }, []);

  // 데이터 조회 함수
  const fetchTodos = async () => {
    setLoading(true);

    const { data, error } = await supabase
      .from('todos')
      .select('*')
      .order('created_at', { ascending: false });

    if (error) {
      console.error('조회 에러:', error.message);
    } else {
      setTodos(data || []);
    }

    setLoading(false);
  };

  if (loading) {
    return <div className="p-8">로딩 중...</div>;
  }

  return (
    <div className="p-8">
      <h1 className="text-2xl font-bold mb-4">할 일 목록</h1>
      <ul>
        {todos.map((todo) => (
          <li key={todo.id}>{todo.text}</li>
        ))}
      </ul>
    </div>
  );
}
```

---

## 4. 데이터 추가하기 (Create)

### 기본 추가

```typescript
const { data, error } = await supabase
  .from('todos')
  .insert({ text: '새 할일', completed: false })
  .select();  // 추가된 데이터 반환

// insert 후 select()를 붙이면 추가된 데이터를 받을 수 있음
```

### 여러 개 추가

```typescript
const { data, error } = await supabase
  .from('todos')
  .insert([
    { text: '할일 1', completed: false },
    { text: '할일 2', completed: false },
  ])
  .select();
```

### 투두앱에 적용

```tsx
const [inputValue, setInputValue] = useState('');

// 추가 함수
const addTodo = async () => {
  if (inputValue.trim() === '') return;

  const { data, error } = await supabase
    .from('todos')
    .insert({ text: inputValue, completed: false })
    .select()
    .single();  // 단일 객체로 반환

  if (error) {
    console.error('추가 에러:', error.message);
    return;
  }

  // 목록 맨 앞에 추가
  setTodos([data, ...todos]);
  setInputValue('');
};
```

### UI 연결

```tsx
<div className="flex gap-2 mb-6">
  <input
    type="text"
    value={inputValue}
    onChange={(e) => setInputValue(e.target.value)}
    onKeyPress={(e) => e.key === 'Enter' && addTodo()}
    placeholder="새로운 할 일..."
    className="flex-1 border rounded-lg px-4 py-2"
  />
  <button
    onClick={addTodo}
    className="bg-blue-500 text-white px-4 py-2 rounded-lg"
  >
    추가
  </button>
</div>
```

---

## 5. 데이터 수정하기 (Update)

### 기본 수정

```typescript
const { error } = await supabase
  .from('todos')
  .update({ completed: true })  // 변경할 값
  .eq('id', 1);                 // 조건: id가 1인 것
```

### 여러 필드 수정

```typescript
const { error } = await supabase
  .from('todos')
  .update({
    text: '수정된 내용',
    completed: true
  })
  .eq('id', 1);
```

### 투두앱에 적용 (완료 토글)

```tsx
const toggleTodo = async (id: number, currentCompleted: boolean) => {
  const { error } = await supabase
    .from('todos')
    .update({ completed: !currentCompleted })
    .eq('id', id);

  if (error) {
    console.error('수정 에러:', error.message);
    return;
  }

  // 로컬 State 업데이트
  setTodos(
    todos.map((todo) =>
      todo.id === id ? { ...todo, completed: !currentCompleted } : todo
    )
  );
};
```

### UI 연결

```tsx
<input
  type="checkbox"
  checked={todo.completed}
  onChange={() => toggleTodo(todo.id, todo.completed)}
  className="w-5 h-5 cursor-pointer"
/>
```

---

## 6. 데이터 삭제하기 (Delete)

### 기본 삭제

```typescript
const { error } = await supabase
  .from('todos')
  .delete()
  .eq('id', 1);  // id가 1인 것 삭제
```

### 조건부 삭제

```typescript
// 완료된 것 모두 삭제
const { error } = await supabase
  .from('todos')
  .delete()
  .eq('completed', true);
```

### 투두앱에 적용

```tsx
const deleteTodo = async (id: number) => {
  const { error } = await supabase
    .from('todos')
    .delete()
    .eq('id', id);

  if (error) {
    console.error('삭제 에러:', error.message);
    return;
  }

  // 로컬 State에서도 제거
  setTodos(todos.filter((todo) => todo.id !== id));
};
```

### UI 연결

```tsx
<button
  onClick={() => deleteTodo(todo.id)}
  className="text-red-500 hover:text-red-700"
>
  삭제
</button>
```

---

## 7. 전체 완성 코드

```tsx
'use client';

import { useEffect, useState } from 'react';
import { supabase } from '@/lib/supabase';

interface Todo {
  id: number;
  text: string;
  completed: boolean;
  created_at: string;
}

export default function Home() {
  const [todos, setTodos] = useState<Todo[]>([]);
  const [inputValue, setInputValue] = useState('');
  const [loading, setLoading] = useState(true);

  // 초기 데이터 로드
  useEffect(() => {
    fetchTodos();
  }, []);

  // 조회
  const fetchTodos = async () => {
    setLoading(true);
    const { data, error } = await supabase
      .from('todos')
      .select('*')
      .order('created_at', { ascending: false });

    if (error) {
      console.error('조회 에러:', error.message);
    } else {
      setTodos(data || []);
    }
    setLoading(false);
  };

  // 추가
  const addTodo = async () => {
    if (inputValue.trim() === '') return;

    const { data, error } = await supabase
      .from('todos')
      .insert({ text: inputValue, completed: false })
      .select()
      .single();

    if (error) {
      console.error('추가 에러:', error.message);
      return;
    }

    setTodos([data, ...todos]);
    setInputValue('');
  };

  // 토글
  const toggleTodo = async (id: number, completed: boolean) => {
    const { error } = await supabase
      .from('todos')
      .update({ completed: !completed })
      .eq('id', id);

    if (error) {
      console.error('수정 에러:', error.message);
      return;
    }

    setTodos(
      todos.map((todo) =>
        todo.id === id ? { ...todo, completed: !completed } : todo
      )
    );
  };

  // 삭제
  const deleteTodo = async (id: number) => {
    const { error } = await supabase
      .from('todos')
      .delete()
      .eq('id', id);

    if (error) {
      console.error('삭제 에러:', error.message);
      return;
    }

    setTodos(todos.filter((todo) => todo.id !== id));
  };

  if (loading) {
    return (
      <div className="min-h-screen flex items-center justify-center">
        <p className="text-gray-500">로딩 중...</p>
      </div>
    );
  }

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
            onKeyPress={(e) => e.key === 'Enter' && addTodo()}
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
                  onChange={() => toggleTodo(todo.id, todo.completed)}
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
            총 {todos.length}개 | 완료 {todos.filter((t) => t.completed).length}개
          </div>
        )}
      </div>
    </div>
  );
}
```

---

## 8. 에러 처리 개선

### 사용자 친화적 에러 처리

```tsx
const [error, setError] = useState<string | null>(null);

const addTodo = async () => {
  if (inputValue.trim() === '') {
    setError('할 일을 입력해주세요');
    return;
  }

  setError(null);  // 에러 초기화

  const { data, error: supabaseError } = await supabase
    .from('todos')
    .insert({ text: inputValue, completed: false })
    .select()
    .single();

  if (supabaseError) {
    setError('추가에 실패했습니다. 다시 시도해주세요.');
    return;
  }

  setTodos([data, ...todos]);
  setInputValue('');
};

// UI에 에러 표시
{error && (
  <p className="text-red-500 text-sm mb-4">{error}</p>
)}
```

### 로딩 상태 개선

```tsx
const [isAdding, setIsAdding] = useState(false);

const addTodo = async () => {
  setIsAdding(true);  // 로딩 시작

  // ... 추가 로직

  setIsAdding(false);  // 로딩 끝
};

<button
  onClick={addTodo}
  disabled={isAdding}
  className={`... ${isAdding ? 'opacity-50 cursor-not-allowed' : ''}`}
>
  {isAdding ? '추가 중...' : '추가'}
</button>
```

---

## 9. 테스트해보기

### 체크리스트

```
□ 페이지 로드 시 기존 데이터 표시되는지
□ 새 할 일 추가되는지
□ 새로고침해도 데이터 유지되는지 ← 중요!
□ 체크박스 토글 동작하는지
□ 삭제 버튼 동작하는지
□ Supabase 대시보드에서 데이터 확인
```

### Supabase 대시보드에서 확인

1. Supabase 대시보드 → Table Editor → todos
2. 앱에서 추가/수정/삭제한 내용이 반영되는지 확인

---

## 핵심 정리

```
✅ CRUD = Create, Read, Update, Delete
✅ async/await = 비동기 작업 기다리기
✅ supabase.from('테이블').select/insert/update/delete
✅ .eq('컬럼', 값) = 조건 지정
✅ 에러 처리로 사용자 경험 개선
✅ 로딩 상태 표시
```

---

## 다음 시간 예고

> 드디어 첫 배포!
> Vercel을 통해 전 세계에 우리 앱을 공개합니다.
> 실제 URL로 접속할 수 있게 됩니다!
