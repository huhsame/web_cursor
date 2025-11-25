# 치트시트 (Cheatsheet)

빠르게 참고할 수 있는 명령어 & 코드 모음

---

## 터미널 명령어

### 기본 명령어
```bash
cd 폴더명        # 폴더 이동
cd ..           # 상위 폴더로
ls              # 파일 목록 (Mac/Linux)
dir             # 파일 목록 (Windows)
mkdir 폴더명     # 폴더 생성
pwd             # 현재 경로
clear           # 화면 지우기
```

### Node.js / npm
```bash
node --version              # Node 버전 확인
npm --version               # npm 버전 확인
npm install                 # 의존성 설치
npm install 패키지명         # 패키지 설치
npm run dev                 # 개발 서버 실행
npm run build               # 프로덕션 빌드
npm run start               # 프로덕션 실행
```

### Git
```bash
git init                    # 저장소 초기화
git status                  # 상태 확인
git add .                   # 모든 파일 스테이징
git commit -m "메시지"       # 커밋
git push                    # 원격에 푸시
git pull                    # 원격에서 가져오기
git log --oneline           # 커밋 히스토리
git branch                  # 브랜치 목록
git checkout -b 브랜치명     # 브랜치 생성 & 이동
```

---

## Next.js

### 프로젝트 생성
```bash
npx create-next-app@latest 프로젝트명
```

### 폴더 구조 → URL
```
app/page.tsx              → /
app/about/page.tsx        → /about
app/products/page.tsx     → /products
app/products/[id]/page.tsx → /products/123
```

### 클라이언트 컴포넌트
```tsx
'use client';  // 파일 최상단에

import { useState, useEffect } from 'react';
```

### 라우팅
```tsx
import { useRouter, useParams, useSearchParams } from 'next/navigation';

const router = useRouter();
router.push('/경로');      // 페이지 이동
router.back();            // 뒤로가기
router.refresh();         // 새로고침

const params = useParams();
params.id                 // 동적 경로 파라미터

const searchParams = useSearchParams();
searchParams.get('query') // ?query=값
```

### Link
```tsx
import Link from 'next/link';

<Link href="/about">소개</Link>
<Link href={`/products/${id}`}>상세보기</Link>
```

---

## React

### useState
```tsx
const [value, setValue] = useState(초기값);
const [count, setCount] = useState(0);
const [text, setText] = useState('');
const [items, setItems] = useState([]);
const [user, setUser] = useState(null);
```

### useEffect
```tsx
// 마운트 시 1번 실행
useEffect(() => {
  fetchData();
}, []);

// 의존성 변경 시 실행
useEffect(() => {
  fetchData();
}, [dependency]);

// 클린업
useEffect(() => {
  const timer = setInterval(() => {}, 1000);
  return () => clearInterval(timer);
}, []);
```

### 이벤트 핸들링
```tsx
<button onClick={() => handleClick()}>클릭</button>
<button onClick={handleClick}>클릭</button>
<input onChange={(e) => setText(e.target.value)} />
<form onSubmit={handleSubmit}>
```

### 조건부 렌더링
```tsx
{condition && <Component />}
{condition ? <A /> : <B />}
{loading ? <p>로딩중</p> : <List />}
```

### 리스트 렌더링
```tsx
{items.map((item) => (
  <li key={item.id}>{item.name}</li>
))}
```

### Props
```tsx
// 부모
<Child name="홍길동" age={20} />

// 자식
function Child({ name, age }: { name: string; age: number }) {
  return <p>{name}은 {age}살</p>;
}
```

---

## Supabase

### 클라이언트 설정
```tsx
// lib/supabase.ts
import { createClient } from '@supabase/supabase-js';

export const supabase = createClient(
  process.env.NEXT_PUBLIC_SUPABASE_URL!,
  process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!
);
```

### CRUD
```tsx
// 조회 (Read)
const { data, error } = await supabase
  .from('테이블')
  .select('*');

// 조건 조회
const { data } = await supabase
  .from('테이블')
  .select('*')
  .eq('컬럼', 값)           // 같음
  .neq('컬럼', 값)          // 다름
  .gt('컬럼', 값)           // 큼
  .lt('컬럼', 값)           // 작음
  .ilike('컬럼', '%검색어%') // 포함 (대소문자 무시)
  .order('컬럼', { ascending: false })
  .limit(10);

// 추가 (Create)
const { data, error } = await supabase
  .from('테이블')
  .insert({ 컬럼: 값 })
  .select()
  .single();

// 수정 (Update)
const { error } = await supabase
  .from('테이블')
  .update({ 컬럼: 값 })
  .eq('id', id);

// 삭제 (Delete)
const { error } = await supabase
  .from('테이블')
  .delete()
  .eq('id', id);
```

### 인증 (Auth)
```tsx
// 회원가입
await supabase.auth.signUp({ email, password });

// 로그인
await supabase.auth.signInWithPassword({ email, password });

// 로그아웃
await supabase.auth.signOut();

// 현재 유저
const { data: { user } } = await supabase.auth.getUser();

// 세션
const { data: { session } } = await supabase.auth.getSession();

// 상태 변화 감지
supabase.auth.onAuthStateChange((event, session) => {
  console.log(event, session);
});
```

### Storage
```tsx
// 업로드
const { data, error } = await supabase.storage
  .from('버킷명')
  .upload('경로/파일명', file);

// 공개 URL
const { data: { publicUrl } } = supabase.storage
  .from('버킷명')
  .getPublicUrl('경로/파일명');

// 삭제
await supabase.storage
  .from('버킷명')
  .remove(['경로/파일명']);
```

---

## Tailwind CSS

### 레이아웃
```
flex              플렉스박스
flex-col          세로 방향
items-center      세로 중앙
justify-center    가로 중앙
justify-between   양쪽 정렬
gap-4             간격

grid              그리드
grid-cols-2       2열
grid-cols-3       3열
md:grid-cols-4    중간화면 4열
```

### 크기
```
w-full            너비 100%
w-1/2             너비 50%
h-screen          높이 = 화면 전체
min-h-screen      최소 높이 = 화면
max-w-md          최대 너비 (md)
aspect-square     1:1 비율
aspect-video      16:9 비율
```

### 여백
```
p-4               padding 전체
px-4              padding 좌우
py-4              padding 상하
m-4               margin 전체
mx-auto           가로 가운데
mt-4              margin-top
mb-4              margin-bottom
```

### 텍스트
```
text-sm           작은 글씨
text-lg           큰 글씨
text-2xl          더 큰 글씨
font-bold         굵은 글씨
text-center       가운데 정렬
text-gray-500     회색 글씨
truncate          말줄임표
```

### 배경 & 테두리
```
bg-white          흰 배경
bg-gray-100       밝은 회색 배경
bg-blue-500       파란 배경
border            테두리
border-gray-300   회색 테두리
rounded-lg        둥근 모서리
shadow-md         그림자
```

### 상호작용
```
hover:bg-blue-600 호버 시 색상
cursor-pointer    포인터 커서
disabled:opacity-50 비활성화 시
transition        애니메이션
```

### 반응형
```
sm:               640px 이상
md:               768px 이상
lg:               1024px 이상
xl:               1280px 이상

예: md:grid-cols-3 (768px 이상에서 3열)
```

---

## TypeScript

### 기본 타입
```tsx
const name: string = '홍길동';
const age: number = 20;
const isActive: boolean = true;
const items: string[] = ['a', 'b'];
const user: { name: string; age: number } = { name: '홍길동', age: 20 };
```

### Interface
```tsx
interface User {
  id: number;
  name: string;
  email: string;
  age?: number;  // 선택적
}

interface Product {
  id: number;
  title: string;
  price: number;
  description: string | null;  // null 가능
}
```

### 함수 타입
```tsx
const add = (a: number, b: number): number => {
  return a + b;
};

const handleClick = (e: React.MouseEvent) => {};
const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {};
const handleSubmit = (e: React.FormEvent) => {};
```

---

## 유용한 패턴

### 로딩 상태
```tsx
const [loading, setLoading] = useState(true);

if (loading) return <p>로딩 중...</p>;
```

### 에러 처리
```tsx
try {
  const { data, error } = await supabase.from('...').select();
  if (error) throw error;
  // 성공 처리
} catch (error) {
  console.error(error);
  alert('오류가 발생했습니다.');
}
```

### 폼 제출
```tsx
const handleSubmit = async (e: React.FormEvent) => {
  e.preventDefault();

  if (!title.trim()) {
    alert('제목을 입력해주세요.');
    return;
  }

  setLoading(true);
  // API 호출
  setLoading(false);
};
```

### 디바운싱
```tsx
const [query, setQuery] = useState('');
const [debouncedQuery, setDebouncedQuery] = useState('');

useEffect(() => {
  const timer = setTimeout(() => {
    setDebouncedQuery(query);
  }, 300);
  return () => clearTimeout(timer);
}, [query]);
```

### 가격 포맷
```tsx
const formatPrice = (price: number) => {
  return price.toLocaleString('ko-KR');
};
// 1000000 → "1,000,000"
```
