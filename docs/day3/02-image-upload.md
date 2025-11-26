# Day 3 - 2교시: 이미지 업로드 기능

## 학습 목표
- Supabase Storage를 설정한다
- 파일 업로드 기능을 구현한다
- 이미지 미리보기를 표시한다
- 업로드된 이미지 URL을 상품에 저장한다

---

## 1. Supabase Storage란?

### 쉬운 설명

> 이미지, 동영상, 문서 같은 **파일**을 저장하는 곳입니다.
>
> 마치 Google Drive나 Dropbox처럼
> 파일을 업로드하고, URL을 통해 접근할 수 있어요.
>
> 데이터베이스에는 파일 자체가 아닌
> **파일의 URL**만 저장합니다.

### 공식적인 설명
```
Supabase Storage는 S3 호환 객체 스토리지 서비스입니다.
버킷(Bucket) 단위로 파일을 관리하며,
RLS 정책으로 접근 권한을 제어할 수 있습니다.
이미지 변환(리사이징 등) 기능도 제공합니다.
```

### 흐름도

```
┌──────────┐      ┌──────────────┐      ┌──────────────┐
│ 사용자    │ ───▶ │  Storage     │ ───▶ │  Database    │
│          │      │  (파일 저장)  │      │  (URL 저장)  │
└──────────┘      └──────────────┘      └──────────────┘
     │                   │
     │ 1. 이미지 업로드    │ 2. URL 반환
     ▼                   ▼
  파일 선택            https://...storage.../image.jpg
```

---

## 2. Storage 버킷 생성

### Step 1: Supabase 대시보드

1. Storage 메뉴 클릭
2. "Create a new bucket" 클릭
3. 정보 입력:

```
Name: product-images
Public bucket: ✅ 체크 (공개 접근)
```

4. "Create bucket" 클릭

### Public vs Private 버킷

```
Public:  누구나 URL로 접근 가능 (이미지 표시용)
Private: 인증된 사용자만 접근 (민감한 파일용)
```

### Step 2: 정책 설정

SQL Editor에서 실행:

```sql
-- 모든 사용자가 이미지를 볼 수 있게
CREATE POLICY "Public Access"
ON storage.objects FOR SELECT
USING (bucket_id = 'product-images');

-- 로그인한 사용자만 업로드 가능
CREATE POLICY "Authenticated users can upload"
ON storage.objects FOR INSERT
WITH CHECK (
  bucket_id = 'product-images'
  AND auth.role() = 'authenticated'
);

-- 본인이 올린 파일만 삭제 가능
CREATE POLICY "Users can delete own files"
ON storage.objects FOR DELETE
USING (
  bucket_id = 'product-images'
  AND auth.uid()::text = (storage.foldername(name))[1]
);
```

---

## 3. 이미지 업로드 컴포넌트

### components/ImageUpload.tsx

```tsx
'use client';

import { useState, useRef } from 'react';
import { supabase } from '@/lib/supabase';

interface ImageUploadProps {
  onUpload: (url: string) => void;
  currentImage?: string;
}

export default function ImageUpload({ onUpload, currentImage }: ImageUploadProps) {
  const [uploading, setUploading] = useState(false);
  const [preview, setPreview] = useState<string | null>(currentImage || null);
  const fileInputRef = useRef<HTMLInputElement>(null);

  const handleFileSelect = async (e: React.ChangeEvent<HTMLInputElement>) => {
    const file = e.target.files?.[0];
    if (!file) return;

    // 파일 유효성 검사
    if (!file.type.startsWith('image/')) {
      alert('이미지 파일만 업로드 가능합니다.');
      return;
    }

    if (file.size > 5 * 1024 * 1024) {  // 5MB
      alert('파일 크기는 5MB 이하여야 합니다.');
      return;
    }

    // 미리보기 표시
    const reader = new FileReader();
    reader.onload = (e) => {
      setPreview(e.target?.result as string);
    };
    reader.readAsDataURL(file);

    // 업로드 시작
    await uploadImage(file);
  };

  const uploadImage = async (file: File) => {
    setUploading(true);

    try {
      // 유저 ID 가져오기
      const { data: { user } } = await supabase.auth.getUser();
      if (!user) throw new Error('로그인이 필요합니다.');

      // 고유한 파일명 생성
      const fileExt = file.name.split('.').pop();
      const fileName = `${user.id}/${Date.now()}.${fileExt}`;

      // Supabase Storage에 업로드
      const { data, error } = await supabase.storage
        .from('product-images')
        .upload(fileName, file);

      if (error) throw error;

      // 공개 URL 가져오기
      const { data: { publicUrl } } = supabase.storage
        .from('product-images')
        .getPublicUrl(data.path);

      // 부모 컴포넌트에 URL 전달
      onUpload(publicUrl);
    } catch (error: any) {
      console.error('업로드 에러:', error);
      alert('업로드에 실패했습니다.');
      setPreview(null);
    } finally {
      setUploading(false);
    }
  };

  const handleClick = () => {
    fileInputRef.current?.click();
  };

  const handleRemove = () => {
    setPreview(null);
    onUpload('');
    if (fileInputRef.current) {
      fileInputRef.current.value = '';
    }
  };

  return (
    <div>
      <input
        type="file"
        ref={fileInputRef}
        onChange={handleFileSelect}
        accept="image/*"
        className="hidden"
      />

      {preview ? (
        // 이미지 미리보기
        <div className="relative">
          <img
            src={preview}
            alt="미리보기"
            className="w-full aspect-video object-contain bg-gray-100 rounded-lg"
          />
          <button
            type="button"
            onClick={handleRemove}
            className="absolute top-2 right-2 bg-red-500 text-white w-8 h-8 rounded-full hover:bg-red-600"
          >
            ✕
          </button>
          {uploading && (
            <div className="absolute inset-0 bg-black/50 flex items-center justify-center rounded-lg">
              <p className="text-white">업로드 중...</p>
            </div>
          )}
        </div>
      ) : (
        // 업로드 버튼
        <button
          type="button"
          onClick={handleClick}
          disabled={uploading}
          className="w-full aspect-video border-2 border-dashed border-gray-300 rounded-lg flex flex-col items-center justify-center hover:border-orange-500 transition"
        >
          <span className="text-4xl mb-2">📷</span>
          <span className="text-gray-500">
            {uploading ? '업로드 중...' : '이미지 업로드'}
          </span>
          <span className="text-sm text-gray-400 mt-1">
            최대 5MB
          </span>
        </button>
      )}
    </div>
  );
}
```

---

## 4. 코드 분석

### 파일 선택 처리

```tsx
const handleFileSelect = async (e: React.ChangeEvent<HTMLInputElement>) => {
  const file = e.target.files?.[0];  // 선택한 파일
  if (!file) return;

  // 이미지인지 확인
  if (!file.type.startsWith('image/')) {
    alert('이미지 파일만 업로드 가능합니다.');
    return;
  }

  // 크기 확인 (5MB)
  if (file.size > 5 * 1024 * 1024) {
    alert('파일 크기는 5MB 이하여야 합니다.');
    return;
  }
};
```

### FileReader로 미리보기

```tsx
const reader = new FileReader();

reader.onload = (e) => {
  // 읽기 완료 시 실행
  const dataUrl = e.target?.result as string;
  setPreview(dataUrl);  // "data:image/jpeg;base64,..."
};

reader.readAsDataURL(file);  // 파일 읽기 시작
```

### Supabase Storage 업로드

```tsx
// 고유한 파일명 생성
const fileName = `${user.id}/${Date.now()}.${fileExt}`;
// 예: "uuid-123/1705312800000.jpg"

// 업로드
const { data, error } = await supabase.storage
  .from('product-images')    // 버킷 이름
  .upload(fileName, file);   // 경로, 파일

// 공개 URL 가져오기
const { data: { publicUrl } } = supabase.storage
  .from('product-images')
  .getPublicUrl(data.path);
// 예: "https://xxx.supabase.co/storage/v1/object/public/product-images/uuid-123/1705312800000.jpg"
```

---

## 5. 상품 등록 페이지에 적용

### products/new/page.tsx 수정

```tsx
'use client';

import { useState, useEffect } from 'react';
import { supabase } from '@/lib/supabase';
import { useRouter } from 'next/navigation';
import ImageUpload from '@/app/components/ImageUpload';
import type { User } from '@supabase/supabase-js';

// ... CATEGORIES 정의

export default function NewProductPage() {
  // ... 기존 state들
  const [imageUrl, setImageUrl] = useState('');  // 이미지 URL 추가!

  // ... useEffect, checkUser

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    // ... 유효성 검사

    const { data, error } = await supabase
      .from('products')
      .insert({
        title: title.trim(),
        description: description.trim(),
        price: parseInt(price),
        category,
        image_url: imageUrl,  // 이미지 URL 추가!
        user_id: user!.id,
        user_email: user!.email,
      })
      .select()
      .single();

    // ...
  };

  return (
    <div className="...">
      <form onSubmit={handleSubmit}>
        {/* 이미지 업로드 추가 */}
        <div className="mb-6">
          <label className="block text-sm font-medium text-gray-700 mb-2">
            상품 이미지
          </label>
          <ImageUpload
            onUpload={(url) => setImageUrl(url)}
            currentImage={imageUrl}
          />
        </div>

        {/* ... 나머지 폼 필드들 */}
      </form>
    </div>
  );
}
```

---

## 6. 숨겨진 input 패턴

### 왜 숨기나요?

```
기본 파일 input은 디자인이 못생겼어요.
숨겨두고, 예쁜 버튼으로 대체합니다!
```

### 구현 방법

```tsx
const fileInputRef = useRef<HTMLInputElement>(null);

// 숨겨진 input
<input
  type="file"
  ref={fileInputRef}
  className="hidden"  // 숨기기
  onChange={handleFileSelect}
/>

// 보이는 버튼
<button onClick={() => fileInputRef.current?.click()}>
  이미지 업로드
</button>
```

---

## 7. 이미지 최적화 팁

### Next.js Image 컴포넌트

```tsx
import Image from 'next/image';

// 외부 이미지 사용시 next.config.js 설정 필요
<Image
  src={product.image_url}
  alt={product.title}
  width={300}
  height={300}
  className="object-cover"
/>
```

### next.config.js 설정

```js
// next.config.js
module.exports = {
  images: {
    remotePatterns: [
      {
        protocol: 'https',
        hostname: '*.supabase.co',
      },
    ],
  },
};
```

### 일반 img 태그로 시작

```tsx
// 개발 단계에서는 일반 img로 시작해도 OK
<img
  src={product.image_url}
  alt={product.title}
  className="w-full h-full object-cover"
/>
```

---

## 8. aspect-ratio 유틸리티

### Tailwind aspect 클래스

```
aspect-square   : 1:1 (정사각형)
aspect-video    : 16:9 (영상)
aspect-[4/3]    : 4:3 (커스텀)
```

### 사용 예시

```tsx
// 정사각형 썸네일
<div className="aspect-square bg-gray-200">
  <img src="..." className="w-full h-full object-cover" />
</div>

// 영상 비율 이미지
<div className="aspect-video bg-gray-200">
  <img src="..." className="w-full h-full object-contain" />
</div>
```

### object-fit 차이

```
object-cover   : 비율 유지, 영역 채움 (잘릴 수 있음)
object-contain : 비율 유지, 영역 안에 맞춤 (여백 생김)
object-fill    : 비율 무시, 영역 채움 (찌그러짐)
```

---

## 9. 에러 처리 개선

### 상세한 에러 메시지

```tsx
const uploadImage = async (file: File) => {
  try {
    // 파일 타입 검사
    const allowedTypes = ['image/jpeg', 'image/png', 'image/gif', 'image/webp'];
    if (!allowedTypes.includes(file.type)) {
      throw new Error('JPG, PNG, GIF, WebP 형식만 지원합니다.');
    }

    // 파일 크기 검사
    const maxSize = 5 * 1024 * 1024;  // 5MB
    if (file.size > maxSize) {
      throw new Error(`파일 크기는 ${maxSize / 1024 / 1024}MB 이하여야 합니다.`);
    }

    // ... 업로드 로직

  } catch (error: any) {
    alert(error.message || '업로드에 실패했습니다.');
  }
};
```

---

## 10. 테스트하기

### 테스트 체크리스트

```
□ 이미지 선택 시 미리보기 표시
□ 5MB 초과 파일 업로드 시 에러
□ 이미지 아닌 파일 업로드 시 에러
□ 업로드 중 로딩 표시
□ X 버튼으로 이미지 제거
□ 상품 등록 시 이미지 URL 저장
□ 상품 목록/상세에서 이미지 표시
□ Supabase Storage에서 파일 확인
```

### Storage에서 확인

Supabase → Storage → product-images 버킷에서 업로드된 파일 확인!

---

## 핵심 정리

```
✅ Supabase Storage = 파일 저장소
✅ Public 버킷 = 누구나 URL로 접근 가능
✅ FileReader로 미리보기 생성
✅ upload()로 업로드, getPublicUrl()로 URL 획득
✅ 파일 타입/크기 검증 필수
✅ 숨겨진 input + 예쁜 버튼 패턴
```

---

## 다음 시간 예고

> 상품에 댓글 기능을 추가합니다!
> 댓글 작성, 목록 표시, 삭제를 구현하고
> 실시간 업데이트도 경험해봅니다.
