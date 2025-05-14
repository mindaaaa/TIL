# Mindapresso 마크다운 블로그

## 이번 주 목표

| 주차  | 목표          | 키워드                             |
| --- | ----------- | ------------------------------- |
| 1주차 | 글 목록 페이지 구현 | `getStaticProps`, `gray-matter` |

### 1. 글 목록 페이지란?

- 블로그 첫 화면에서 **여러 글들의 제목/날짜/태그 등을 나열**하는 페이지.
- 상세 페이지(`/posts/[slug]`)와는 다르게, 한눈에 글들을 탐색할 수 있는 UI.
- 블로그의 핵심 UI 중 하나이며, 글 데이터를 **리스트 형태**로 보여줘야 한다.

### 2. `gray-matter`가 필요한 이유

- `.md` 파일 내부에는 글 내용(content)과 함께 **메타데이터(frontmatter)**가 있음.
  ```md
  ---
  title: "Next.js 시작하기"
  date: "2025-05-13"
  tags: ["nextjs", "blog"]
  ---
  본문 내용...
- `gray-matter`는 이 `.md` 파일을 읽어서

    - `data` title, date, tags 등 메타데이터
    - `content` 실제 본문 내용  
으로 깔끔하게 나눠주는 역할을 함.

직접 구현할 수도 있지만,
줄바꿈·들여쓰기·YAML 파싱 등 복잡한 예외 처리가 많음
→  **실무에서도 거의 다 `gray-matter`를 표준처럼 씀**

> npm 주간 다운로드수 200만 이상

### 3. 정적 사이트란?

> “페이지가 요청될 때 서버가 HTML을 만드는 게 아니라,  
> **미리 만들어진 HTML을 보여주는 방식.**”

- 속도 빠르고, SEO에 매우 유리.
- 블로그처럼 내용이 자주 안 바뀌는 페이지에 특히 적합.

### 4. Next.js에서 정적 사이트 구현하기
#### 📌 Pages Router 기준

- `getStaticProps()` 사용  
    → 빌드 시점에 데이터를 불러와서 정적 HTML 생성
#### 📌 App Router 기준 (우리가 쓰는 구조)

- `generateStaticParams()`로 URL 경로 목록 정의
- 서버 컴포넌트 내에서 파일 불러오기 → 정적 HTML 생성

### 5. 전체 흐름 요약

```txt
posts/mockPost.md      ← 마크다운 원본
↓ (gray-matter)
lib/posts.ts        ← 메타데이터 + 본문 파싱
↓
getStaticProps      ← 빌드 시점에 호출
↓
page.tsx            ← 글 목록 렌더링 (PostList 컴포넌트 사용)
```

## 옵시디언 연동 문제

### 📌 추후 작업 예정 

> 현재 구조는 `글 렌더링 구조의 뼈대`를 만드는 과정이다.

1. 이후 Obsidian에서 작성한 `.md`를 GitHub repo에 연동한다.
2. 블로그 프로젝트에 그 `repo`를 **git submodule**로 연결한다.  
	→ **글 데이터가 블로그에 들어오게 된다.**
3. GitHub Actions를 설정한다.
>Obsidian 저장소에 커밋이 발생할 때  블로그 프로젝트의 `submodule`을 자동으로 업데이트하고 Vercel 배포까지 트리거하여 자동 연동이 완성된다.