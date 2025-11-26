# Day 2 - 1교시: Supabase 소개 & 설정

## 학습 목표
- Supabase가 무엇인지 이해한다
- Supabase 프로젝트를 생성한다
- 데이터베이스 테이블을 만든다
- Next.js 프로젝트와 연결한다

---

## 1. Supabase란?

### 쉬운 설명

> 백엔드 서버를 직접 만들려면 정말 많은 것을 해야 해요.
>
> - 서버 세팅
> - 데이터베이스 설치
> - 로그인 시스템 구축
> - 파일 저장소 설정
> - 보안 설정
> - ...
>
> **Supabase**는 이 모든 걸 **클릭 몇 번**으로 제공합니다!
> 마치 레스토랑에서 요리하는 대신
> 배달 음식을 시키는 것과 같아요.

### 공식적인 설명
```
Supabase는 오픈소스 Firebase 대안으로,
PostgreSQL 데이터베이스, 인증(Auth),
실시간 구독(Realtime), 스토리지(Storage),
Edge Functions 등을 제공하는
Backend-as-a-Service (BaaS) 플랫폼입니다.
```

### Firebase vs Supabase

| 항목 | Firebase | Supabase |
|------|----------|----------|
| 데이터베이스 | NoSQL (Firestore) | SQL (PostgreSQL) |
| 오픈소스 | X | O |
| 가격 | 사용량 기반 | 프리 티어 넉넉 |
| 학습 곡선 | 낮음 | 낮음 |
| SQL 지원 | X | O |

---

## 2. Supabase가 제공하는 것

### 올인원 백엔드 서비스

```
┌─────────────────────────────────────────────────────────────┐
│                        Supabase                             │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │
│   │  Database    │  │     Auth     │  │   Storage    │     │
│   │  (PostgreSQL)│  │   (로그인)    │  │ (파일 저장)   │     │
│   │              │  │              │  │              │     │
│   │ 데이터 저장   │  │ 회원가입     │  │ 이미지       │     │
│   │ 조회/수정    │  │ 소셜 로그인   │  │ 문서         │     │
│   │ 관계 설정    │  │ 세션 관리    │  │ 영상         │     │
│   └──────────────┘  └──────────────┘  └──────────────┘     │
│                                                             │
│   ┌──────────────┐  ┌──────────────┐                       │
│   │   Realtime   │  │  Edge Func   │                       │
│   │  (실시간)     │  │  (서버리스)   │                       │
│   │              │  │              │                       │
│   │ 채팅         │  │ 커스텀 로직   │                       │
│   │ 알림         │  │ API          │                       │
│   └──────────────┘  └──────────────┘                       │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 각 서비스 설명

| 서비스 | 역할 | 우리 프로젝트에서 |
|--------|------|-----------------|
| **Database** | 데이터 저장소 | 할 일, 상품, 댓글 저장 |
| **Auth** | 사용자 인증 | 로그인, 회원가입 |
| **Storage** | 파일 저장 | 상품 이미지 업로드 |
| **Realtime** | 실시간 동기화 | (선택) 실시간 알림 |

---

## 3. Supabase 프로젝트 생성

### Step 1: 회원가입

1. [supabase.com](https://supabase.com) 접속
2. "Start your project" 클릭
3. GitHub 계정으로 로그인 (가장 간편)

### Step 2: 새 프로젝트 생성

1. 대시보드에서 "New Project" 클릭
2. 정보 입력:

```
Organization: Personal (기본값)
Project name: my-todo-app
Database Password: [강력한 비밀번호]
Region: Northeast Asia (Seoul) ← 한국 선택!
Pricing Plan: Free
```

```
⚠️ 중요!
Database Password는 꼭 기억하거나 따로 저장하세요.
나중에 다시 확인할 수 없습니다!
```

3. "Create new project" 클릭
4. 2-3분 기다리기 (프로젝트 설정 중...)

### Step 3: 프로젝트 대시보드 확인

프로젝트가 생성되면 대시보드가 나타납니다:

```
┌─────────────────────────────────────────────────┐
│  my-todo-app                                    │
├─────────────────────────────────────────────────┤
│                                                 │
│  [Table Editor]  [SQL Editor]  [Auth]          │
│  [Storage]       [Functions]   [Settings]      │
│                                                 │
└─────────────────────────────────────────────────┘
```

---

## 4. 테이블 만들기 (투두앱용)

### 테이블이란?

> 엑셀 시트를 생각하세요.
> 행(row)과 열(column)로 이루어진 표입니다.
>
> - 열(Column) = 데이터의 종류 (이름, 나이, 이메일...)
> - 행(Row) = 실제 데이터 한 건

### Step 1: Table Editor 열기

왼쪽 메뉴에서 "Table Editor" 클릭

### Step 2: 새 테이블 생성

1. "Create a new table" 클릭
2. 테이블 정보 입력:

```
Name: todos
Description: 할 일 목록

☑ Enable Row Level Security (RLS)
  → 보안을 위해 체크 (나중에 설정)
```

### Step 3: 컬럼 추가

"Add column" 버튼으로 컬럼을 추가합니다:

| Name | Type | Default Value | Options |
|------|------|---------------|---------|
| id | int8 | (자동) | Primary Key |
| created_at | timestamptz | now() | |
| text | text | | |
| completed | bool | false | |

**컬럼 설명:**
```
id         : 각 할 일의 고유 번호 (자동 생성)
created_at : 생성 시간 (자동 기록)
text       : 할 일 내용
completed  : 완료 여부 (true/false)
```

### Step 4: 테이블 저장

"Save" 버튼 클릭

### 결과 확인

```
todos 테이블
┌────┬─────────────────────┬──────────────┬───────────┐
│ id │     created_at      │     text     │ completed │
├────┼─────────────────────┼──────────────┼───────────┤
│    │                     │              │           │
│    │   (아직 데이터 없음) │              │           │
│    │                     │              │           │
└────┴─────────────────────┴──────────────┴───────────┘
```

---

## 5. 테스트 데이터 추가

### Step 1: 데이터 입력

Table Editor에서 "Insert" 버튼 클릭

```
text: 장보기
completed: false
```

"Save" 클릭

### Step 2: 더 추가해보기

같은 방법으로 2-3개 더 추가:

```
text: 운동하기, completed: true
text: 책 읽기, completed: false
```

### 결과

```
todos 테이블
┌────┬─────────────────────┬──────────────┬───────────┐
│ id │     created_at      │     text     │ completed │
├────┼─────────────────────┼──────────────┼───────────┤
│  1 │ 2024-01-15 10:00:00 │    장보기    │   false   │
│  2 │ 2024-01-15 10:01:00 │   운동하기   │   true    │
│  3 │ 2024-01-15 10:02:00 │    책 읽기   │   false   │
└────┴─────────────────────┴──────────────┴───────────┘
```

---

## 6. API 키 확인하기

### API 키가 필요한 이유

> Supabase에 접근하려면 "신분증"이 필요합니다.
> API 키가 바로 그 신분증이에요.

### Step 1: Settings 메뉴

왼쪽 메뉴 맨 아래 "Project Settings" 클릭

### Step 2: API 탭

"API" 탭 선택

### Step 3: 키 확인

두 가지 중요한 정보:

```
Project URL:
https://xxxxxxxxxxxxx.supabase.co

API Keys:
- anon (public): eyJhbGciOiJIUzI1NiIsInR5cCI6...
- service_role (secret): eyJhbGciOiJIUzI1NiIsInR5cCI6...
```

```
⚠️ 중요!
- anon key: 클라이언트(브라우저)에서 사용 가능
- service_role key: 절대 공개하면 안 됨! (서버에서만 사용)
```

---

## 7. Next.js 프로젝트에 연결하기

### Step 1: Supabase 패키지 설치

터미널에서:

```bash
cd my-todo-app
npm install @supabase/supabase-js
```

### Step 2: 환경 변수 파일 생성

프로젝트 루트에 `.env.local` 파일 생성:

```bash
# .env.local
NEXT_PUBLIC_SUPABASE_URL=https://xxxxxxxxxxxxx.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6...
```

```
💡 NEXT_PUBLIC_ 접두사
   Next.js에서 브라우저에서도 사용할 환경 변수는
   반드시 NEXT_PUBLIC_으로 시작해야 합니다.
```

### Step 3: Supabase 클라이언트 생성

`lib/supabase.ts` 파일 생성:

```typescript
// lib/supabase.ts
import { createClient } from '@supabase/supabase-js';

const supabaseUrl = process.env.NEXT_PUBLIC_SUPABASE_URL!;
const supabaseAnonKey = process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!;

export const supabase = createClient(supabaseUrl, supabaseAnonKey);
```

### 폴더 구조

```
my-todo-app/
├── app/
│   └── page.tsx
├── lib/                 ← 새로 생성
│   └── supabase.ts      ← 새로 생성
├── .env.local           ← 새로 생성
├── package.json
└── ...
```

---

## 8. 연결 테스트

### Step 1: 테스트 코드 작성

`app/page.tsx`를 수정해서 Supabase 연결을 테스트:

```tsx
'use client';

import { useEffect, useState } from 'react';
import { supabase } from '@/lib/supabase';

export default function Home() {
  const [todos, setTodos] = useState<any[]>([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetchTodos();
  }, []);

  const fetchTodos = async () => {
    const { data, error } = await supabase
      .from('todos')
      .select('*');

    if (error) {
      console.error('Error:', error);
    } else {
      console.log('Data:', data);
      setTodos(data || []);
    }
    setLoading(false);
  };

  if (loading) return <p>로딩 중...</p>;

  return (
    <div className="p-8">
      <h1 className="text-2xl font-bold mb-4">Supabase 연결 테스트</h1>
      <pre className="bg-gray-100 p-4 rounded">
        {JSON.stringify(todos, null, 2)}
      </pre>
    </div>
  );
}
```

### Step 2: 개발 서버 실행

```bash
npm run dev
```

### Step 3: 결과 확인

브라우저에서 `localhost:3000` 접속

**성공하면:**
```json
[
  {
    "id": 1,
    "created_at": "2024-01-15T10:00:00.000Z",
    "text": "장보기",
    "completed": false
  },
  ...
]
```

**에러가 나면:**
```
1. .env.local 파일 확인
2. API 키가 맞는지 확인
3. 개발 서버 재시작 (환경 변수 변경 후 필수!)
```

---

## 9. RLS (Row Level Security) 설정

### RLS란?

> 데이터베이스의 "보안 경비원"입니다.
>
> "이 사람은 자기 데이터만 볼 수 있어"
> "로그인 안 한 사람은 읽기만 가능해"
>
> 같은 규칙을 설정할 수 있어요.

### 공식적인 설명
```
Row Level Security는 PostgreSQL의 기능으로,
행(row) 단위로 접근 권한을 설정할 수 있습니다.
누가 어떤 데이터를 읽고/쓰고/수정/삭제할 수 있는지
세밀하게 제어합니다.
```

### 임시로 모든 접근 허용 (개발용)

1. Supabase 대시보드 → Table Editor → todos 테이블
2. 오른쪽 상단 "RLS disabled" 확인
3. SQL Editor에서 다음 실행:

```sql
-- 모든 사용자가 todos 테이블을 읽을 수 있게
CREATE POLICY "Allow public read" ON todos
FOR SELECT USING (true);

-- 모든 사용자가 todos를 추가할 수 있게
CREATE POLICY "Allow public insert" ON todos
FOR INSERT WITH CHECK (true);

-- 모든 사용자가 todos를 수정할 수 있게
CREATE POLICY "Allow public update" ON todos
FOR UPDATE USING (true);

-- 모든 사용자가 todos를 삭제할 수 있게
CREATE POLICY "Allow public delete" ON todos
FOR DELETE USING (true);
```

```
⚠️ 주의!
이 설정은 개발/학습용입니다.
실제 서비스에서는 사용자별 권한을 설정해야 합니다.
```

---

## 10. 정리

### Supabase 설정 체크리스트

```
✅ Supabase 계정 생성
✅ 새 프로젝트 생성 (Seoul 리전)
✅ todos 테이블 생성
✅ 테스트 데이터 추가
✅ API 키 확인
✅ Next.js에 패키지 설치
✅ .env.local에 키 저장
✅ supabase.ts 클라이언트 생성
✅ 연결 테스트 성공
✅ RLS 정책 설정
```

### 핵심 정리

```
✅ Supabase = 백엔드 올인원 서비스
✅ PostgreSQL 데이터베이스 제공
✅ 테이블 = 데이터를 저장하는 표
✅ API 키로 접근 인증
✅ RLS로 보안 설정
```

---

## 다음 시간 예고

> Supabase 연결이 완료됐으니,
> 이제 투두앱의 CRUD를 데이터베이스와 연동합니다!
>
> - 데이터 조회 (Read)
> - 데이터 추가 (Create)
> - 데이터 수정 (Update)
> - 데이터 삭제 (Delete)
