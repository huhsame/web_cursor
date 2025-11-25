# Day 3 - 3교시: 댓글 기능 구현

## 학습 목표
- 댓글 목록을 표시한다
- 댓글 작성 기능을 구현한다
- 댓글 삭제 기능을 구현한다
- 부모-자식 컴포넌트 데이터 흐름을 이해한다

---

## 1. 댓글 기능 구조

```
상품 상세 페이지
├── 상품 정보
└── 댓글 섹션
    ├── 댓글 입력 폼
    └── 댓글 목록
        ├── 댓글 1 (삭제 버튼)
        ├── 댓글 2 (삭제 버튼)
        └── ...
```

---

## 2. 댓글 목록 컴포넌트

### components/CommentList.tsx

```tsx
'use client';

import { useEffect, useState } from 'react';
import { supabase } from '@/lib/supabase';
import type { User } from '@supabase/supabase-js';

interface Comment {
  id: number;
  content: string;
  created_at: string;
  user_id: string;
  user_email: string;
  product_id: number;
}

interface CommentListProps {
  productId: number;
}

// 날짜 포맷팅
const formatDate = (dateString: string) => {
  const date = new Date(dateString);
  const now = new Date();
  const diff = now.getTime() - date.getTime();

  const minutes = Math.floor(diff / (1000 * 60));
  const hours = Math.floor(diff / (1000 * 60 * 60));
  const days = Math.floor(diff / (1000 * 60 * 60 * 24));

  if (minutes < 1) return '방금 전';
  if (minutes < 60) return `${minutes}분 전`;
  if (hours < 24) return `${hours}시간 전`;
  if (days < 7) return `${days}일 전`;

  return date.toLocaleDateString('ko-KR');
};

export default function CommentList({ productId }: CommentListProps) {
  const [comments, setComments] = useState<Comment[]>([]);
  const [newComment, setNewComment] = useState('');
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState(true);
  const [submitting, setSubmitting] = useState(false);

  // 초기 로드
  useEffect(() => {
    fetchComments();
    getUser();
  }, [productId]);

  // 유저 정보
  const getUser = async () => {
    const { data: { user } } = await supabase.auth.getUser();
    setUser(user);
  };

  // 댓글 목록 조회
  const fetchComments = async () => {
    const { data, error } = await supabase
      .from('comments')
      .select('*')
      .eq('product_id', productId)
      .order('created_at', { ascending: true });

    if (error) {
      console.error('댓글 조회 에러:', error);
    } else {
      setComments(data || []);
    }
    setLoading(false);
  };

  // 댓글 작성
  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();

    if (!user) {
      alert('로그인이 필요합니다.');
      return;
    }

    if (!newComment.trim()) {
      alert('댓글 내용을 입력해주세요.');
      return;
    }

    setSubmitting(true);

    const { data, error } = await supabase
      .from('comments')
      .insert({
        content: newComment.trim(),
        product_id: productId,
        user_id: user.id,
        user_email: user.email,
      })
      .select()
      .single();

    if (error) {
      console.error('댓글 작성 에러:', error);
      alert('댓글 작성에 실패했습니다.');
    } else {
      setComments([...comments, data]);
      setNewComment('');
    }

    setSubmitting(false);
  };

  // 댓글 삭제
  const handleDelete = async (commentId: number) => {
    if (!confirm('댓글을 삭제하시겠습니까?')) return;

    const { error } = await supabase
      .from('comments')
      .delete()
      .eq('id', commentId);

    if (error) {
      console.error('댓글 삭제 에러:', error);
      alert('삭제에 실패했습니다.');
    } else {
      setComments(comments.filter(c => c.id !== commentId));
    }
  };

  return (
    <div className="bg-white rounded-lg shadow-md p-6">
      <h2 className="text-lg font-bold mb-4">
        댓글 {comments.length}개
      </h2>

      {/* 댓글 입력 */}
      {user ? (
        <form onSubmit={handleSubmit} className="mb-6">
          <div className="flex gap-2">
            <input
              type="text"
              value={newComment}
              onChange={(e) => setNewComment(e.target.value)}
              placeholder="댓글을 입력하세요..."
              className="flex-1 border border-gray-300 rounded-lg px-4 py-2 focus:outline-none focus:ring-2 focus:ring-orange-500"
              maxLength={500}
            />
            <button
              type="submit"
              disabled={submitting}
              className="bg-orange-500 text-white px-4 py-2 rounded-lg hover:bg-orange-600 disabled:bg-gray-400 transition whitespace-nowrap"
            >
              {submitting ? '등록 중...' : '등록'}
            </button>
          </div>
        </form>
      ) : (
        <p className="text-gray-500 text-sm mb-6">
          댓글을 작성하려면 로그인이 필요합니다.
        </p>
      )}

      {/* 댓글 목록 */}
      {loading ? (
        <p className="text-gray-500">로딩 중...</p>
      ) : comments.length === 0 ? (
        <p className="text-gray-500">아직 댓글이 없습니다.</p>
      ) : (
        <ul className="space-y-4">
          {comments.map((comment) => (
            <li
              key={comment.id}
              className="border-b border-gray-100 pb-4 last:border-0"
            >
              <div className="flex items-start justify-between">
                <div className="flex-1">
                  {/* 작성자 정보 */}
                  <div className="flex items-center gap-2 mb-1">
                    <span className="font-medium text-sm">
                      {comment.user_email}
                    </span>
                    <span className="text-gray-400 text-xs">
                      {formatDate(comment.created_at)}
                    </span>
                  </div>
                  {/* 댓글 내용 */}
                  <p className="text-gray-700">{comment.content}</p>
                </div>

                {/* 삭제 버튼 (본인만) */}
                {user && user.id === comment.user_id && (
                  <button
                    onClick={() => handleDelete(comment.id)}
                    className="text-gray-400 hover:text-red-500 text-sm ml-2"
                  >
                    삭제
                  </button>
                )}
              </div>
            </li>
          ))}
        </ul>
      )}
    </div>
  );
}
```

---

## 3. 상세 페이지에 댓글 추가

### products/[id]/page.tsx 수정

```tsx
import CommentList from '@/app/components/CommentList';

export default function ProductDetailPage() {
  // ... 기존 코드

  return (
    <div className="min-h-screen bg-gray-100 py-8">
      <div className="max-w-4xl mx-auto px-4">
        {/* 상품 정보 카드 */}
        <div className="bg-white rounded-lg shadow-md">
          {/* ... 상품 정보 */}
        </div>

        {/* 댓글 섹션 */}
        <div className="mt-8">
          <CommentList productId={parseInt(productId)} />
        </div>
      </div>
    </div>
  );
}
```

---

## 4. 코드 분석

### 상대 시간 표시

```tsx
const formatDate = (dateString: string) => {
  const date = new Date(dateString);
  const now = new Date();
  const diff = now.getTime() - date.getTime();  // 밀리초 차이

  const minutes = Math.floor(diff / (1000 * 60));
  const hours = Math.floor(diff / (1000 * 60 * 60));
  const days = Math.floor(diff / (1000 * 60 * 60 * 24));

  if (minutes < 1) return '방금 전';
  if (minutes < 60) return `${minutes}분 전`;
  if (hours < 24) return `${hours}시간 전`;
  if (days < 7) return `${days}일 전`;

  return date.toLocaleDateString('ko-KR');
};

// 결과 예시:
// "방금 전"
// "5분 전"
// "3시간 전"
// "2일 전"
// "2024. 1. 15."
```

### 댓글 추가 후 목록 업데이트

```tsx
const { data, error } = await supabase
  .from('comments')
  .insert({ ... })
  .select()
  .single();

if (!error) {
  // 기존 목록에 새 댓글 추가
  setComments([...comments, data]);
  // 입력창 초기화
  setNewComment('');
}
```

### 댓글 삭제 후 목록 업데이트

```tsx
const handleDelete = async (commentId: number) => {
  // ... 삭제 API 호출

  // 해당 댓글만 제외한 새 배열
  setComments(comments.filter(c => c.id !== commentId));
};
```

---

## 5. 조건부 렌더링 패턴들

### 로그인 여부에 따른 UI

```tsx
{user ? (
  // 로그인 O: 입력 폼 표시
  <form>...</form>
) : (
  // 로그인 X: 안내 메시지
  <p>로그인이 필요합니다.</p>
)}
```

### 본인 댓글만 삭제 버튼

```tsx
{user && user.id === comment.user_id && (
  <button onClick={() => handleDelete(comment.id)}>
    삭제
  </button>
)}
```

### 로딩/빈 상태/목록

```tsx
{loading ? (
  <p>로딩 중...</p>
) : comments.length === 0 ? (
  <p>댓글이 없습니다.</p>
) : (
  <ul>
    {comments.map(c => <li key={c.id}>...</li>)}
  </ul>
)}
```

---

## 6. 최적화: 낙관적 업데이트

### 현재 방식 (서버 응답 후 업데이트)

```
1. 등록 버튼 클릭
2. 서버에 요청
3. 응답 받음 (0.5~1초)
4. 화면 업데이트
```

### 낙관적 업데이트 (즉시 업데이트)

```
1. 등록 버튼 클릭
2. 화면 즉시 업데이트 ← 빠른 피드백!
3. 서버에 요청 (백그라운드)
4. 실패 시 롤백
```

### 구현 예시

```tsx
const handleSubmit = async (e: React.FormEvent) => {
  e.preventDefault();

  // 임시 댓글 (ID는 임시값)
  const tempComment = {
    id: Date.now(),  // 임시 ID
    content: newComment.trim(),
    product_id: productId,
    user_id: user!.id,
    user_email: user!.email || '',
    created_at: new Date().toISOString(),
  };

  // 즉시 화면에 추가 (낙관적)
  setComments([...comments, tempComment]);
  setNewComment('');

  // 서버에 요청
  const { data, error } = await supabase
    .from('comments')
    .insert({
      content: tempComment.content,
      product_id: productId,
      user_id: user!.id,
      user_email: user!.email,
    })
    .select()
    .single();

  if (error) {
    // 실패 시 롤백
    setComments(comments);
    alert('댓글 작성에 실패했습니다.');
  } else {
    // 성공 시 임시 ID를 실제 ID로 교체
    setComments(prev =>
      prev.map(c => c.id === tempComment.id ? data : c)
    );
  }
};
```

---

## 7. textarea로 변경 (선택)

### 긴 댓글 지원

```tsx
<textarea
  value={newComment}
  onChange={(e) => setNewComment(e.target.value)}
  placeholder="댓글을 입력하세요..."
  rows={3}
  maxLength={500}
  className="w-full border rounded-lg px-4 py-2 resize-none"
/>

<div className="flex justify-between mt-2">
  <span className="text-sm text-gray-400">
    {newComment.length}/500
  </span>
  <button type="submit">등록</button>
</div>
```

---

## 8. 실시간 댓글 (선택 - 고급)

### Supabase Realtime 사용

```tsx
useEffect(() => {
  // 실시간 구독
  const channel = supabase
    .channel('comments')
    .on(
      'postgres_changes',
      {
        event: 'INSERT',
        schema: 'public',
        table: 'comments',
        filter: `product_id=eq.${productId}`,
      },
      (payload) => {
        // 새 댓글이 추가되면
        setComments(prev => [...prev, payload.new as Comment]);
      }
    )
    .subscribe();

  // 클린업
  return () => {
    supabase.removeChannel(channel);
  };
}, [productId]);
```

```
💡 Realtime은 선택 사항입니다.
   기본 CRUD만으로도 충분히 동작합니다.
   실시간 채팅 같은 기능에 적합합니다.
```

---

## 9. 댓글 수 표시

### 상품 카드에 댓글 수

방법 1: 별도 쿼리

```tsx
// 댓글 수 조회
const { count } = await supabase
  .from('comments')
  .select('*', { count: 'exact', head: true })
  .eq('product_id', productId);
```

방법 2: 조인 쿼리 (고급)

```tsx
const { data } = await supabase
  .from('products')
  .select(`
    *,
    comments(count)
  `);

// data[0].comments[0].count
```

---

## 10. 테스트하기

### 테스트 체크리스트

```
□ 비로그인 상태: 댓글 입력 폼 대신 안내 메시지
□ 로그인 상태: 댓글 입력 폼 표시
□ 댓글 작성 → 목록에 즉시 추가
□ 빈 댓글 제출 → 에러 메시지
□ 본인 댓글에 삭제 버튼 표시
□ 타인 댓글에 삭제 버튼 없음
□ 댓글 삭제 → 목록에서 제거
□ 새로고침 → 댓글 유지 (DB 저장 확인)
```

---

## 핵심 정리

```
✅ CommentList 컴포넌트로 댓글 기능 분리
✅ productId를 props로 받아 특정 상품 댓글만 조회
✅ 로그인 상태에 따른 조건부 렌더링
✅ 본인 댓글만 삭제 가능
✅ 상대 시간 포맷팅 (몇 분 전, 몇 시간 전)
✅ 낙관적 업데이트로 UX 개선 (선택)
```

---

## 다음 시간 예고

> 검색과 필터 기능을 추가합니다!
> 키워드로 상품을 찾고,
> 가격순/최신순 정렬도 구현해봅니다.
