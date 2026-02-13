# Chapter 16. Supabase MCP로 개발하기

이번 챕터에서는 **Supabase MCP**를 연결합니다.
AI가 직접 데이터베이스를 확인하고 조작할 수 있게 되어요!

---

## 16.1 Supabase MCP란?

### 무엇을 할 수 있나요?

Supabase MCP를 연결하면 AI가:

| 기능 | 설명 |
|------|------|
| **테이블 조회** | DB 스키마 확인 |
| **데이터 조회** | SELECT 쿼리 실행 |
| **데이터 추가** | INSERT 쿼리 실행 |
| **데이터 수정** | UPDATE 쿼리 실행 |
| **데이터 삭제** | DELETE 쿼리 실행 |

### 왜 좋은가요?

**Before (MCP 없이)**
```
1. Supabase 대시보드 접속
2. 테이블 구조 확인
3. Cursor로 돌아와서 AI에게 설명
4. 코드 작성
```

**After (MCP 있으면)**
```
1. AI에게 "products 테이블에 맞는 코드 짜줘"
2. AI가 직접 테이블 확인
3. 바로 코드 작성!
```

---

## 16.2 Supabase MCP 연결하기

### Step 1: Supabase 액세스 토큰 생성

1. **[supabase.com](https://supabase.com)** 로그인
2. 우측 상단 프로필 → **Account Settings**
3. **Access Tokens** 탭 클릭
4. **Generate new token** 클릭
5. 이름 입력 (예: `cursor-mcp`)
6. **Generate** 클릭
7. **토큰 복사** (이 화면을 벗어나면 다시 볼 수 없음!)

<!-- 스크린샷: Access Token 생성 화면 -->

> ⚠️ **토큰은 안전한 곳에 저장하세요!**
> 다시 볼 수 없습니다.

### Step 2: Cursor에 MCP 추가

Cursor 설정 → MCP → **Add new MCP server**

| 항목 | 입력값 |
|------|--------|
| Name | `supabase` |
| Type | `command` |
| Command | `npx -y @supabase/mcp-server-supabase@latest --access-token YOUR_TOKEN` |

> ⚠️ **YOUR_TOKEN을 실제 토큰으로 바꾸세요!**

### Step 3: 연결 확인

MCP 목록에서 **supabase**가 ✅ 상태인지 확인

```
MCP Servers
├── context7 ✅
└── supabase ✅  ← 새로 추가됨!
```

---

## 16.3 mcp.json으로 설정하기

### 프로젝트별 설정 (추천)

`.cursor/mcp.json` 파일:

```json
{
  "mcpServers": {
    "context7": {
      "command": "npx",
      "args": ["-y", "@upstash/context7-mcp"]
    },
    "supabase": {
      "command": "npx",
      "args": [
        "-y",
        "@supabase/mcp-server-supabase@latest",
        "--access-token",
        "sbp_xxxxxxxxxxxxx"
      ]
    }
  }
}
```

> ⚠️ **mcp.json에 토큰이 들어가므로 .gitignore에 추가하세요!**

### .gitignore 추가

```
# MCP 설정 (토큰 포함)
.cursor/mcp.json
```

### 환경변수 사용하기 (더 안전)

토큰을 직접 넣지 않고 **환경변수**를 사용할 수도 있습니다.

```json
{
  "mcpServers": {
    "supabase": {
      "command": "npx",
      "args": [
        "-y",
        "@supabase/mcp-server-supabase@latest",
        "--access-token",
        "${SUPABASE_ACCESS_TOKEN}"
      ]
    }
  }
}
```

`.env.local`에 토큰 저장:
```
SUPABASE_ACCESS_TOKEN=sbp_xxxxxxxxxxxxx
```

---

## 16.4 Supabase MCP 사용해보기

### 테스트 1: 테이블 구조 확인

```
Supabase에 어떤 테이블이 있어?
```

### 예상 결과

```
AI: Supabase MCP를 통해 확인한 결과, 다음 테이블들이 있습니다:

1. products (상품)
   - id: int8, primary key
   - title: text
   - price: int4
   - description: text
   - image_url: text
   - user_id: uuid
   - created_at: timestamptz

2. profiles (사용자 프로필)
   - id: uuid, primary key
   - nickname: text
   - avatar_url: text
   - created_at: timestamptz

3. comments (댓글)
   ...
```

**AI가 직접 DB에서 정보를 가져옵니다!**

### 테스트 2: 데이터 조회

```
products 테이블에서 가격이 10000원 이상인 상품 보여줘
```

### 예상 결과

```
AI: Supabase에서 조회한 결과입니다:

| id | title | price | created_at |
|----|-------|-------|------------|
| 3  | 맥북 프로 | 2500000 | 2024-01-15 |
| 7  | 아이패드 | 850000 | 2024-01-14 |
| 12 | 에어팟 | 180000 | 2024-01-13 |

총 3개의 상품이 검색되었습니다.
```

### 테스트 3: 코드 작성 요청

```
products 테이블 구조에 맞는 상품 등록 폼 컴포넌트 만들어줘
```

### 예상 결과

AI가 **실제 테이블 구조를 확인**하고 그에 맞는 코드를 작성합니다:

```tsx
'use client'

import { useState } from 'react'
import { supabase } from '@/lib/supabase'

export default function ProductForm() {
  const [title, setTitle] = useState('')
  const [price, setPrice] = useState('')
  const [description, setDescription] = useState('')
  const [imageUrl, setImageUrl] = useState('')

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault()

    const { data, error } = await supabase
      .from('products')
      .insert({
        title,
        price: parseInt(price),
        description,
        image_url: imageUrl,
      })
      .select()

    if (error) {
      alert('등록 실패: ' + error.message)
      return
    }

    alert('상품이 등록되었습니다!')
  }

  return (
    <form onSubmit={handleSubmit}>
      {/* 폼 필드들... */}
    </form>
  )
}
```

**테이블 컬럼명과 정확히 일치하는 코드가 생성됩니다!**

---

## 16.5 실전 활용 예시

### 예시 1: 버그 수정

```
products 테이블에서 price가 null인 데이터가 있는지 확인하고,
있으면 0으로 업데이트하는 코드 짜줘
```

AI가:
1. DB에서 null인 데이터 확인
2. 있으면 업데이트 쿼리 제안
3. 코드 작성

### 예시 2: 새 기능 개발

```
products 테이블에 category 컬럼을 추가하고,
카테고리별로 필터링하는 기능 만들어줘
```

AI가:
1. 현재 테이블 구조 확인
2. ALTER TABLE 쿼리 제안
3. 필터링 코드 작성

### 예시 3: 성능 분석

```
어떤 카테고리 상품이 가장 많이 등록되어 있어?
```

AI가:
1. GROUP BY 쿼리 실행
2. 결과 분석
3. 인사이트 제공

---

## 16.6 Supabase MCP가 제공하는 도구

> Supabase MCP는 **20개 이상의 도구**를 제공합니다!
>
> 2025년 Supabase에서 공식 출시한 MCP 서버로,
> AI가 Supabase를 거의 전부 컨트롤할 수 있어요.

### 데이터베이스 관련 도구

| 도구 | 기능 |
|------|------|
| `list_tables` | 테이블 목록 조회 |
| `get_table_schema` | 테이블 스키마 조회 |
| `execute_sql` | SQL 쿼리 실행 |
| `apply_migration` | 마이그레이션 적용 |
| `create_table` | 새 테이블 생성 |
| `alter_table` | 테이블 구조 수정 |

### 프로젝트 관리 도구

| 도구 | 기능 |
|------|------|
| `get_project_info` | 프로젝트 정보 조회 |
| `create_project` | 새 프로젝트 생성 |
| `pause_project` | 프로젝트 일시 중지 |
| `restore_project` | 프로젝트 복원 |
| `get_project_settings` | 설정 정보 가져오기 |

### 개발 및 디버깅 도구

| 도구 | 기능 |
|------|------|
| `create_branch` | 개발용 브랜치 생성 |
| `get_logs` | 로그 확인 |
| `generate_types` | TypeScript 타입 생성 |
| `get_cost_estimate` | 비용 산정 확인 |

### 도구 호출 예시

```
사용자: "users 테이블 구조 알려줘"

AI 내부 동작:
1. list_tables 호출 → 테이블 목록 확인
2. get_table_schema("users") 호출 → 스키마 확인
3. 결과를 사용자에게 설명
```

---

## 16.7 보안 설정

Supabase MCP는 **보안과 사용자 통제**를 위한 기능을 제공합니다.

### 액세스 토큰 관리

| 해야 할 것 | 하면 안 되는 것 |
|-----------|---------------|
| 환경변수로 관리 | 코드에 직접 입력 |
| .gitignore에 추가 | Git에 커밋 |
| 필요시 재발급 | 토큰 공유 |

### 읽기 전용 모드

> **AI가 실수로 데이터를 변경하지 못하도록** 설정할 수 있습니다.

```json
{
  "mcpServers": {
    "supabase": {
      "command": "npx",
      "args": [
        "-y",
        "@supabase/mcp-server-supabase@latest",
        "--access-token", "sbp_xxxxx",
        "--read-only"
      ]
    }
  }
}
```

`--read-only` 옵션을 추가하면:
- SELECT 쿼리만 허용
- INSERT, UPDATE, DELETE 차단
- 테이블 생성/수정 차단

### 프로젝트 범위 제한

> **특정 프로젝트에만 연결**되도록 제한할 수 있습니다.

```json
{
  "mcpServers": {
    "supabase": {
      "command": "npx",
      "args": [
        "-y",
        "@supabase/mcp-server-supabase@latest",
        "--access-token", "sbp_xxxxx",
        "--project-ref", "abcdefghijk"
      ]
    }
  }
}
```

`--project-ref` 옵션을 추가하면:
- 지정한 프로젝트만 접근 가능
- 다른 프로젝트 목록 조회 불가
- 실수로 다른 프로젝트 건드리는 것 방지

### 환경별 추천 설정

| 환경 | 추천 설정 |
|------|----------|
| 개발용 DB | 제한 없이 사용 OK |
| 테스트 DB | 제한 없이 사용 OK |
| 스테이징 DB | `--project-ref` 추가 |
| 프로덕션 DB | `--read-only` + `--project-ref` 모두 추가 |

---

## 16.8 실제 프로젝트 연동하기

MCP로 테이블을 만들었으면, 실제 프로젝트와 연동해야 합니다.

### Step 1: Supabase에서 API 키 확인

1. Supabase 대시보드 접속
2. **Settings** → **API** 클릭
3. 다음 두 값을 복사:
   - **Project URL**
   - **anon public** key

<!-- 스크린샷: API 설정 화면 -->

### Step 2: 환경변수 설정

프로젝트 루트에 `.env.local` 파일 생성:

```bash
NEXT_PUBLIC_SUPABASE_URL=https://abcdefghijk.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6...
```

### Step 3: Supabase 클라이언트 확인

`lib/supabase.ts` 파일이 있는지 확인:

```typescript
import { createClient } from '@supabase/supabase-js'

const supabaseUrl = process.env.NEXT_PUBLIC_SUPABASE_URL!
const supabaseAnonKey = process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!

export const supabase = createClient(supabaseUrl, supabaseAnonKey)
```

### Step 4: 연동 테스트

간단한 데이터를 추가하고 Supabase 대시보드에서 확인:

```typescript
// 테스트 코드
const { data, error } = await supabase
  .from('products')
  .insert({ title: '테스트 상품', price: 10000 })
  .select()

console.log(data, error)
```

Supabase 대시보드 → **Table Editor**에서 데이터가 보이면 성공!

### 연동이 안 될 때

> 💡 **MCP만으로 바로 연동이 안 될 수도 있습니다!**

MCP는 테이블 생성, 스키마 관리에 유용하지만,
실제 프로젝트 연동은 **환경변수 설정**이 별도로 필요합니다.

AI에게 요청:
```
이 프로젝트와 Supabase를 연동하는 방법 알려줘
```

AI가 단계별로 안내해줄 거예요!

---

## 16.9 사용 팁과 주의사항

### MCP 사용이 적합한 경우

| 상황 | 추천도 |
|------|--------|
| 테이블 구조 확인 | ⭐⭐⭐ 매우 적합 |
| 데이터 조회 | ⭐⭐⭐ 매우 적합 |
| 개발 중 스키마 변경 | ⭐⭐⭐ 매우 적합 |
| 개인 프로젝트 | ⭐⭐⭐ 매우 적합 |
| 팀 프로젝트 (개발 DB) | ⭐⭐ 적합 |
| 프로덕션 데이터 수정 | ⭐ 주의 필요 |

### 이런 점이 좋았어요

1. **스키마 확인이 편함**
   - Supabase 대시보드 왔다갔다 안 해도 됨
   - AI가 테이블 구조 보고 바로 코드 작성

2. **마이그레이션 자동화**
   - 컬럼 추가/수정을 AI가 SQL로 생성

3. **타입 생성**
   - TypeScript 타입을 자동으로 생성

### 이런 점은 주의하세요

1. **중요한 데이터는 직접 확인**
   ```
   AI: products 테이블의 모든 데이터를 삭제했습니다.

   (프로덕션이었다면... 😱)
   ```

2. **복잡한 쿼리는 검토 필요**
   - AI가 생성한 SQL을 바로 실행하기 전에 확인

3. **권한 설정은 Supabase에서**
   - RLS 정책 등 보안 설정은 대시보드에서 직접

### 추천 워크플로우

```
1. MCP로 테이블 구조 확인
2. MCP로 필요한 테이블 생성/수정
3. 환경변수로 프로젝트 연동
4. 코드에서 Supabase 클라이언트 사용
5. 중요한 변경은 Supabase 대시보드에서 확인
```

---

## 16.10 문제 해결

### "연결이 안 돼요"

1. **토큰 확인**
   - 토큰이 정확한지 확인
   - 만료되지 않았는지 확인

2. **프로젝트 확인**
   - Supabase 프로젝트가 활성 상태인지 확인

3. **Cursor 재시작**
   - MCP 설정 후 Cursor 재시작

### "권한 오류가 나요"

```
Error: permission denied for table products
```

1. **RLS 정책 확인**
   - Supabase에서 Row Level Security 확인

2. **토큰 권한 확인**
   - Access Token에 충분한 권한이 있는지 확인

### "쿼리가 실행 안 돼요"

1. **SQL 문법 확인**
   - AI에게 SQL 문법 검토 요청

2. **테이블명/컬럼명 확인**
   - 오타가 없는지 확인

---

## 핵심 정리

```
✅ Supabase MCP = AI가 DB에 직접 접근 (20개 이상 도구)
✅ Access Token 필요 (안전하게 관리!)
✅ 테이블 구조, 데이터 조회, 마이그레이션 가능
✅ --read-only 옵션으로 읽기 전용 설정
✅ --project-ref 옵션으로 특정 프로젝트만 접근
✅ 프로젝트 연동은 환경변수 설정 별도 필요
✅ 개인/테스트 프로젝트에 적극 활용, 프로덕션은 주의!
```

---

## 다음 챕터에서는

> MCP를 직접 만들어봅니다!
> 간단한 MCP 서버를 만들고 Cursor에 연결해볼 거예요.
>
> 👉 [Chapter 17. MCP 직접 만들기](chapter17-mcp-create.md)
