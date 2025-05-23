# 목차
[프로젝트 목표](## 프로젝트 목표)
[기능 구현 목록](## 기능 구현 목록)
[구현 기술 스택](## 구현 기술 스택)
[주요 디렉토리 구조](## 🧱 주요 디렉토리 구조)
[학습 포인트 정리](## 학습 포인트 정리)
[주차별 진행 계획](## 주차별 진행 계획)
[참고 도서](## 참고 도서)
[기능 범위 선정에 대하여](## 기능 범위 선정에 관하여)

---
## 🐙 mini-git

 - `핵심 구조를 직접 구현하며, Git의 내부 동작을 몸으로 이해한다.`

> `풀스펙 Git`이 아니라,
> `Git의 핵심 원리를 직접 구현해서 구조를 이해`하기 위한 프로젝트

> 핵심 구조 (`add`, `commit`, `log`, `branch`, `checkout`, `HEAD`) 위주로 진행

## 프로젝트 목표

> Git의 내부 구조와 동작 원리를 직접 구현하며  
> 자료구조, 파일 시스템, 소프트웨어 설계 개념을 체득하는 CS 중심 실습 프로젝트입니다.

> `add`, `commit`, `log`, `HEAD` 등 핵심 기능을 직접 구현하여  
> Git의 구조와 원리를 이해하고, 이를 코드로 구조화해보는 것이 목적입니다.

- 단순히 완성도 높은 Git 클론을 만드는 것이 아니라,  
  Git이 왜 이렇게 설계되었는지를 **직접 구현하며 체득**합니다.
- 많은 기능보다 **학습의 밀도**, **구조적 이해**, **과정 중심 사고**에 집중합니다.

## 기능 구현 목록

> ✅ 필수
> ⭕ 선택
> ❌ 제외

| 분류  | 기능                                                             | 설명                                                 |
| --- | -------------------------------------------------------------- | -------------------------------------------------- |
| ✅   | `init` <br>   – `.mygit` 폴더 생성, 초기 구조                          | - `.mygit/` 디렉토리와 기본 구조 생성                         |
| ✅   | `add` <br>  – 스테이징 영역에 파일 추가                                   | - 파일을 읽고 blob 형태로 해시화하여 objects에 저장<br>- index에 기록 |
| ✅   | `commit` <br>  – snapshot 저장, 메시지 기록                           | - staging된 파일들을 snapshot으로 커밋<br>- 메시지와 함께 저장      |
| ✅   | `log` <br>  – 커밋 내역 조회 <br>  – `해시`, `메시지`, `부모 해시`            | HEAD에서 시작하여 커밋 히스토리를 따라가며 로그 출력                    |
| ✅   | `HEAD` <br>  – 현재 브랜치 포인터 추적                                   | 현재 가리키는 커밋 혹은 브랜치 위치 관리                            |
| ✅   | `.mygit/objects` 구조 만들기 <br>  - `blob` / `tree` / `commit` 저장소 | 실제 Git이 사용하는 저장소 구조를 재현                            |
| ⭕   | `branch` <br>  – 브랜치 생성, 브랜치별 HEAD 추적                          | - 새로운 브랜치 포인터 생성<br>- `refs/heads/`에 저장            |
| ⭕   | `checkout` <br>  – 브랜치 이동 (HEAD 전환)                            | - HEAD를 특정 브랜치로 전환<br>- 간단한 워킹 디렉토리 변경 포함          |
| ❌   | `merge`, `stash`, `reset` 등                                    | 복잡한 `conflict` 처리 포함                               |

## 구현 기술 스택

| 요소 | 선택 이유 |
|------|-----------|
| Node.js | CLI 구현에 적합하고 파일 입출력 구조 익히기 쉬움 |
| `fs` 모듈 | `.mygit/objects`, `HEAD`, `index` 파일 처리 |
| CLI | `process.argv` 또는 `readline` 기반 입력 처리 |

## 🧱 주요 디렉토리 구조

```sql
.mygit/
├── HEAD
├── index
├── objects/
│   ├── <해시> (blob/tree/commit)
├── refs/
│   └── heads/
│       ├── main
│       └── other-branch
```

## 학습 포인트 정리

> 핵심 구조는 `tree`, `blob`, `commit`, `HEAD`, `refs`🏄

- Git은 snapshot 기반 버전관리 시스템<br>→ 구조를 직접 다뤄서 감을 잡기
- 커밋 객체의 구성을 통해 *트리, blob, 커밋 해시 등* 구조 학습
- `HEAD`, `refs` 등 포인터와 상태 전이 흐름 설계
- SHA 해시 <br>→ 데이터 무결성과 참조 관리 방법 습득
- CLI 기반 프로젝트 설계, 명령어 파싱 및 입출력 처리

| 기능 | 학습되는 CS 개념 |
|------|------------------|
| `add`, `commit` | 파일 시스템, 해시, 데이터 저장 방식 |
| `log`, `HEAD` | 포인터, 연결 구조 감각 |
| `branch`, `checkout` | 상태 전이, 브랜치 구조 |

## 주차별 진행 계획
### 1. *Week 1*
- 구조 이해 & 최소 동작 만들기

| 목표           | 상세 내용                                                                       |
| ------------ | --------------------------------------------------------------------------- |
| `git init`   | `.mygit/` 폴더, `objects/`, `refs/`, `HEAD` 초기화                               |
| `git add`    | 파일을 읽어 해시(blob) 생성 <br>→ `objects/`에 저장, `index` 역할 JSON에 기록                |
| `git commit` | staging 영역 <br>→ tree <br>→ commit 객체 생성 <br>→ `objects/` 저장, `HEAD` 포인터 갱신 |
| 학습           | Git object 구조 이해 <br>→ blob, tree, commit, HEAD 관계도 그리기                     |

> *Git이 파일을 어떻게 저장할까?* #snapshot
> **자료구조와 파일 시스템 감각**을 키우는 데 집중합니다🔥

### 2. *Week 2* 
- 커밋 히스토리 & 브랜치 흐름

| 목표                  | 상세 내용                                   |
| ------------------- | --------------------------------------- |
| `git log`           | HEAD에서 부모 커밋 따라가며 로그 출력                 |
| `HEAD`              | 현재 커밋 또는 브랜치 정보 기록 및 이동 처리              |
| `branch` (optional) | `refs/heads/브랜치명` 파일 생성 <br>  → HEAD 연결 |
| 구조 학습               | Git에서 `포인터` 개념이 왜 중요한지 알아보자🏄           |

> `Git은 결국 포인터의 집합`이다.
> 구조적 흐름 감각을 훈련하는 데 초점 두기💪🏻

### 3. *Week 3*
- `checkout` & 확장

| 목표         | 상세 내용                                    |
| ---------- | ---------------------------------------- |
| `checkout` | HEAD를 브랜치로 전환하고, 작업 디렉토리 갱신              |
| 리팩토링       | - 클래스 분리<br>- 상태 패턴 적용<br>- 구조화된 커밋 시나리오 |
| 테스트        | 커밋 시나리오 테스트                              |

> 📌 커밋 시나리오
> init → add → commit → branch → commit → checkout → log  

### ✨ *Week 3.5* #회고 #자료정리

- 삽질 정리, 배운 개념 블로그 포스팅 
- `.mygit/` 폴더 전체 설명 문서 작성
- Git 개념 다이어그램(`tree`, `commit` `HEAD`) 시각화 

## 참고 도서

| 목적                     | 추천 도서              |
| ---------------------- | ------------------ |
| 내부 구조 이해 및 mini-git 설계 | 깃: 분산 버전 관리 시스템    |
| 구조 및 실습 대비             | Pro Git            |
| 입문 / 정리용 스낵 도서         | 모두의 Git            |
| 핵심만 간단하게               | Git Internals #PDF |

> 구조 이해를 위해 책을 읽고
> 실습은 `Git object`와 `JSON`구조로 *mini-git* 구현

> `tig`, `git log --graph`, `.git/objects/` 등을 직접 열어보면서 진행하기🪡
> `.git/objects`, `cat-file`, `hash-object` 실습


## 기능 범위 선정에 관하여

> Git의 **90% 이상은 snapshot 구조 + 포인터 구조**로 설명 가능
#### Git의 핵심 개념을 직접 다뤄볼 수 있는 기능

- `add`, `commit`, `log`, `HEAD`, `branch`은 <br>Git의 **핵심 자료구조**인 `blob`, `tree`, `commit`, 그리고 `HEAD` 포인터를 모두 사용합니다.
- 특히 `objects`에 저장되는 구조는 SHA-1 해시 기반으로 설계되어 있어<br>이걸 직접 구현하면 **해시의 역할, 스냅샷 저장 방식, 포인터 추적 구조**를 모두 경험할 수 있습니다

> 이 기능들만으로도 Git이 **왜 스냅샷 기반인지**,  
> **어떻게 파일을 추적하고 브랜치를 관리하는지**를 직접 체감할 수 있다.

더불어, 다음과 같은 CS 영역을 추가 학습할 수 있습니다.

- 자료구조: 트리`커밋 히스토리`, 연결 리스트`커밋 간 참조`, 해시 맵`SHA 객체`
- 운영체제: 파일 입출력, 디렉토리 생성, 버퍼링 개념
- 컴퓨터 구조: 포인터`HEAD`, 참조에 따른 상태 변화

#### 2. 고급 기능은 학습 효율 대비 복잡도 상승

예를 들어 `merge`, `reset`, `stash` 같은 기능은<br>
Git을 실무에서 써 본 사람도 헷갈릴 정도로 구조가 복잡합니다🙇‍♂️

- `merge`: 두 브랜치를 병합하면서 공통 조상을 기준으로 `diff` 계산<br>→ conflict 알고리즘 필요
- `reset`: working dir, index, HEAD를 따로따로 움직이기 때문에<br>내부 상태 관리가 어렵다🤯
- `stash`: 커밋하지 않은 변경 사항을 임시 저장소에 따로 쌓는 로직 필요 <br>→ 복잡한 스냅샷 큐 구조 필요

📌 따라서 이 기능들은 지금은 일단 넘어가고 *나중에 확장 과제로 구현해보면 좋을 부분*으로 남겨둘 예정입니다.

> Git의 구조나 설계를 학습하는 관점에서는 우선순위가 상대적으로 낮다.

#### 3. mini-git 프로젝트를 통해 무엇을 배울 것인가?

> ⚠️ mini-git은 단순한 기능 구현 프로젝트가 아니다.

 이 프로젝트는 기능의 **개수**보다 `구조적 사고와 문제 해결력`에 초점을 둡니다.
 
 - 스냅샷을 어떤 구조로 저장할 것인가?
- HEAD 포인터를 어떻게 움직일 것인가?
- 브랜치는 어떻게 분기하고 참조할 것인가?

이런 것들은 직접 설계해봐야 알 수 있는 영역입니다.

> 핵심 동작을 스스로 모델링하고 구현할 수 있는가가 핵심!
> Git을 단순히 `쓴다`를 넘어서 `이해하고 설명할 수 있다`에 초점을 두자

> Git은 단순히 `add`와 `commit`을 넘어 <br>*포인터(HEAD), 저장소 구조, 해시 객체, 트리 구조*가 유기적으로 연결된 시스템임을 깨닫기

