### 📌 목표

- Next.js의 `App Router`와 마크다운 파일 활용
- 실제 콘텐츠는 `.md` 파일로 작성<br>→ 정적 빌드 시점에 **HTML**로 변환하여 페이지 구성

# 로직

```plaintext
posts/hello.md       ← 마크다운 원본
↓ (gray-matter)
lib/posts.ts         ← 글 목록 데이터 추출
↓
getStaticProps       ← 빌드 시점에 props로 넘김
↓
pages/index.tsx or app/page.tsx ← 목록 렌더링
↓
PostList + PostCard  ← UI 구성
```

## 1. 마크다운 글 작성

- `posts/` 디렉토리에 `.md` 파일들을 저장
- 각 파일 상단에는 `gray-matter` 형식의 frontmatter를 사용해 메타데이터를 작성

```md
---
title: 'Next.js 시작하기'
date: '2025-05-13'
tags: ['nextjs', 'blog']
slug: 'nextjs-intro'
---

Next.js는 리액트를 위한 프레임워크입니다...
```

---

## 2. 글 목록 추출

- `lib/posts.ts`에서 `fs`와 `path`를 이용해 `posts/` 폴더의 모든 파일을 읽음
- `gray-matter`를 사용하여 **메타데이터**만 추출

```ts
export function getAllPosts() {
  const filenames = fs.readdirSync(postsDirectory);

  return filenames.map((filename) => {
    const filePath = path.join(postsDirectory, filename);
    const fileContents = fs.readFileSync(filePath, 'utf8');
    const { data } = matter(fileContents);

    return {
      ...data,
      slug: filename.replace(/\.md$/, ''),
    };
  });
}
```

→ `title`, `date`, `tags`, `slug` 등의 정보를 갖는 객체 배열 반환

```json
// data 형식
{
  "title": "Next 시작",
  "date": "2025-05-14",
  "tags": ["nextjs", "gray-matter"]
}
```

---

## 3. 라우팅 구조

- App Router를 사용하는 경우,<br>`app/` 디렉토리 내에서 디렉토리/파일 구조가 곧 URL 경로

```plaintext
app/
├── page.tsx               → /
└── posts/
    └── [slug]/
        └── page.tsx       → /posts/:slug
```

> `layout.tsx`는 각 경로에 공통 UI를 적용할 수 있는 컴포넌트<br>
> 루트 디렉토리에 **반드시 존재**해야 함🔥🔥

<br>

### `app/` 디렉토리는 왜 `pages/`를 대체했을까?

#### 📌 `pages/` 라우팅 방식

- 라우트 하나에 파일 하나만 매칭 → `pages/posts/[slug].tsx`
- 공통 레이아웃, 로딩 UI, 에러 처리<br>→ `_app.tsx`, `_document.tsx`에 몰아넣음
- 클라이언트 컴포넌트 중심으로 `SSR`이나 `서버 데이터 패칭`에 제약

> **👀 클라이언트 컴포넌트란?**<br>브라우저에서 실행되며 JS를 통해 동적 UI를 구성하는 React 컴포넌트

#### 📌 `app/` 디렉토리 도입

Next.js 13부터 도입된 _App Router 시스템_

- 폴더 단위로 `라우팅/레이아웃/로딩 처리`까지 분리
- 각 경로에서 **공통 구조와 동작을 명시적으로 정의**
- 기본적으로 **서버 컴포넌트 기반**으로 렌더링 → 성능 최적화에 유리

> **👀 서버 컴포넌트란?**
> 서버에서 실행되어 HTML을 미리 렌더링하고 <br>JS 없이도 초기 화면을 구성할 수 있는 컴포넌트

#### 📌 비교

| 구분             | pages/                      | app/                             |
| ---------------- | --------------------------- | -------------------------------- |
| 라우팅 방식      | 파일 = 경로                 | 디렉토리 + 파일 조합             |
| 레이아웃 구성    | `_app.tsx`에서 처리         | `layout.tsx` 명시적 선언         |
| 페이지 구성 단위 | 단일 컴포넌트               | 폴더 단위의 구조적 구성          |
| 서버 기능 지원   | 클라이언트 중심, fetch 제한 | 서버 컴포넌트 기본<br>fetch 자유 |

→ `app/` 디렉토리는 **확장성과 유지보수를 고려한 구조적 진화**

---

## 4.메타데이터 설정

- App Router에서는 각 페이지 또는 layout 파일 안에<br>`metadata` 객체를 export 하여 SEO 관련 정보를 설정 가능

```ts
export const metadata = {
  title: 'Mindapresso 블로그',
  description: '프론트엔드 개발자의 기술 기록',
};
```

> 👀 SEO와 메타데이터<br>`metadata`는 SEO(Search Engine Optimization)를 위한 정보로, <br>검색 시 제목이나 설명, OG 이미지 등에 활용됨

- `layout.tsx`에 설정하면 하위 모든 페이지에 적용
- `page.tsx`에서 별도로 선언하면 그 페이지에만 적용

### 📌 메타데이터는 왜 필요할까?

> 마크다운 기반 블로그에서는 단순히 `.md` 파일만 존재할 경우,<br>  
> 제목, 날짜, 태그 같은 정보가 없으면 **글을 제대로 목록에 나열하거나 정렬할 수 없다.**

→ 목록에서는 단순히 `hello.md` 같은 **파일명**만 보임🤯

#### 예시

- 글의 **제목**을 목록에 출력하려면 → `title` 필요
- 최신순으로 정렬하려면 → `date` 필요
- 태그 필터링을 하려면 → `tags` 필요
- URL 경로를 커스터마이징하려면 → `slug` 필요
- SEO 최적화를 하려면 → `description`, `keywords`, `og:title` 등이 필요

```md
---
title: 'Next.js 시작하기'
date: '2025-05-16'
tags: ['nextjs', 'blog']
slug: 'nextjs-intro'
---
```

→ 메타데이터가 없다면, **블로그는 단순 텍스트 뷰어**에 불과하다🤦‍♀️

## 5. UI 구성

- `getAllPosts()`로 추출한 글 목록<br> → `<PostList />` 컴포넌트로 전달<br> → 각 글은 `<PostCard />`로 렌더링

```tsx
// PostList
{
  posts.map((post) => <PostCard key={post.slug} post={post} />);
}
```

```tsx
// PostCard
<Link href={`/posts/${post.slug}`}>{post.title}</Link>
```

## 6. 정적 페이지 생성

> 📌 2주차에서 진행되는 부분

- 상세 페이지(`/posts/:slug`)는 정적으로 생성
- `generateStaticParams()`에서 가능한 slug 목록을 미리 반환<br>→`getPostBySlug()` 함수로 해당 마크다운 본문을 읽음

## 7. 레이아웃 구조

- `app/layout.tsx` 사이트 전체 공통 레이아웃 <br>→ header, footer 등 포함
- `app/posts/layout.tsx` `/posts/*` 전용 UI 구조 추가

```tsx
export default function RootLayout({ children }) {
  return (
    <html lang='ko'>
      <body>
        <Header />
        {children}
        <Footer />
      </body>
    </html>
  );
}
```

→ 모든 페이지는 결국 layout의 `children`으로 들어감

> React 관점에서 보면, <br>**모든 페이지가 `layout`이라는 부모 컴포넌트 아래에서 렌더링되는 구조**.

> App Router에서는 이 `children` 흐름이 레이아웃 계층을 명확하게 만듦

### 📌 요약

- `posts/*.md` 파일을 기반으로 글 데이터를 추출
- `App Router` 기반으로 페이지를 구성
- `layout`과 `metadata`를 통해 공통 UI 및 SEO를 설정<br>→ 정적 블로그 구조 구현

#### 다음 목표

- 글 상세 페이지 제작 → 2주차
- 동적 라우팅을 이용해 `Obsidian` 렌더링 출력 → 3주차
