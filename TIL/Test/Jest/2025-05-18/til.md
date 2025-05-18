## 📌 참고 레퍼런스

> [테스트 with Jest: 제로초에게 제대로 배우기](https://www.inflearn.com/course/%ED%85%8C%EC%8A%A4%ED%8A%B8-with-jest-%EC%A0%9C%EB%A1%9C%EC%B4%88) <br> [Jest](https://jestjs.io/docs/expect)

## 📌 배운 것

- 테스트 초기화
- spy 관리

# `beforeAll` vs `beforeEach`

### 공통점

- **테스트 전에 실행되는 설정용 훅**
- **공통 상태 세팅**/**초기화 작업** 때 사용

### 차이점

| 구분             | `beforeAll()`                            | `beforeEach()`                               |
| ---------------- | ---------------------------------------- | -------------------------------------------- |
| 실행 시점        | 테스트 전체 중 1번                       | 각 `test`, `it`마다 실행                     |
| 상태 재사용 여부 | 모든 테스트가 **동일 인스턴스 사용**     | **각 테스트마다 새로 세팅**                  |
| 주로 하는 일     | DB 연결, 서버 부팅 등<br>→ 무거운 설정용 | - mock 함수 초기화<br>- fresh한 배열 선언 등 |
| 속도             | 빠름 `1번만 실행`                        | 느림 `매번 실행`                             |

## 예시

```ts
// beforeAll: 상태 공유
let db;

beforeAll(() => {
  db = connectDB(); // 딱 1번 일어남
});

test('db 연결 확인', () => {
  expect(db.connected).toBe(true);
});
```

```ts
// beforeEach: 상태 초기화
let users;

beforeEach(() => {
  users = ['minda', 'gu'];
});

test('유저 추가', () => {
  users.push('hana');
  expect(users).toHaveLength(3);
});
```

### 👀 언제 문제가 생길까?

| 잘못된 선택                            | 문제 예시                                               |
| -------------------------------------- | ------------------------------------------------------- |
| mock 초기화에 `beforeAll` 사용         | 이전 테스트 상태가 다음 테스트에 누적됨 <br>→ 상태 오염 |
| 전역 상태가 필요한데 `beforeEach` 사용 | - 매번 초기화돼서 성능 낭비<br>- 기능 불안정            |

---

# `jest.clearAllMocks()` vs `jest.resetAllMocks()`

### 공통점

- **모든 mock 함수에 적용되는 전역 함수**
- `mockClear()`와 `mockReset()`의 **전체 버전**

| 함수              | 호출 기록 초기화 | 구현 초기화                                                                         | 사용 예시                                   |
| ----------------- | ---------------- | ----------------------------------------------------------------------------------- | ------------------------------------------- |
| `clearAllMocks()` | ✅               | ❌                                                                                  | mock 함수 사용 이력만 초기화<br>→ 호출 횟수 |
| `resetAllMocks()` | ✅               | ✅ mock 구현 자체까지 초기화<br>→ `mockReturnValue`<br>→ `mockImplementation` 등 >→ |

## 예시

```ts
// clearAllMocks 예시
const fn1 = jest.fn();
fn1('hello');

jest.clearAllMocks();
expect(fn1).not.toHaveBeenCalled();
```

- 호출 기록은 사라지나 구현은 유지

```ts
// resetAllMocks 예시
const mockFn = jest.fn(() => 'hi');
mockFn();

jest.resetAllMocks();
mockFn(); // undefined 반환됨
```

- 호출 기록과 `mockImplementation`도 초기화

### 👀 왜 필요할까?

- `beforeEach()` 훅에 넣어 테스트 간 상태 오염 방지 → mock 함수 테스트 환경에 유용
- `mcokReturnValueOnce()` 같은 한정 동작 재사용시 `resetAllMocks()` 사용 적합

### 자동화 설정

> `jest.config.js`에서 모든 테스트 전 자동 초기화 되도록 설정 가능

```ts
module.exports = {
  clearMocks: true,
  resetMocks: true,
};
```

---

# ⚠️ spy는 항상 한 번만 생성하자

## 잘못된 패턴

### spy를 `beforeEach()`에서 생성

```ts
let spy;

beforeEach(() => {
  spy = jest.spyOn(api, 'fetchData');
});
```

- 매 테스트마다 새로운 spy 인스턴스가 생김
- 이전 spy와 **연결이 끊겨** Jest가 추적을 못함
- `toHaveBeenCalled()` 같은 matcher가 **오작동**할 가능성 있음🧨🧨

#### 👀 Q. 스코프/라이프사이클 무조건 작게 가져가야하지 않나요?

> *무조건 작게 가져가야 한다*는 **지나친 일반화**

##### 테스트의 핵심

- 테스트의 불변성이나 스코프 분리는 테스트의 예측 가능성과 독립성을 높이기 위함<br> → `신뢰 가능한 테스트`를 향한 수단일 뿐 **목적이 아님**

> 결국 테스트는,<br>테스트가 얼마나 **안정적**이고 **신뢰 가능**한지,<br>테스트의 의도가 **명확히** 드러나는지가 핵심📌

#### Q. 👀 spy를 매번 만들면 어떤 문제가 발생할까?

1. **spy 인스턴스가 매번 새로 생성**  
   → Jest는 이전에 만든 spy와 지금 spy가 **전혀 다른 객체**라고 인식
   <br>→ 이후 `mockClear()`나 `mockRestore()` 호출 시 **원래 spy를 못 찾는 경우가 생김**
2. **spy 중복 생성**  
   → `jest.spyOn()`은 내부적으로 원본 메서드를 감싸는 **래퍼(wrapper)** 를 새로 만듦
   <br>→ 메모리 낭비 + Jest 내부 상태 혼란 발생 ⚠️
3. **spy 추적이 꼬이고, 테스트 안정성 저하**  
   → 예전 spy에는 호출 기록이 있지만 새로운 spy에는 없음  
   → `toHaveBeenCalled()`가 false 나올 수 있음 `잘못된 실패`

##### 📌 핵심은 **인스턴스의 신뢰성**

```ts
const spy = jest.spyOn(obj, 'method');
```

- `obj.method`에 **감시기(래퍼 함수)** 를 붙인다.<br> → `spy`는 이 래퍼 객체를 가리키는 변수

- 이걸 `beforeEach()`에서 반복하면? <br>→ 매번 새로운 래퍼 함수가 생성
  > `기존에 추적하던 spy`와 `지금 만든 spy`를 Jest가 **구분 못함**

##### 예시

```ts
let spy;

describe('bad pattern', () => {
  beforeEach(() => {
    spy = jest.spyOn(api, 'fetchData'); // ⚠️ 매번 spy 생성
  });

  test('A', () => {
    api.fetchData();
    expect(spy).toHaveBeenCalled(); // 통과
  });

  test('B', () => {
    expect(spy).not.toHaveBeenCalled(); // 실패 가능성 ↑
  });
});
```

- A에서 fetchData 호출 → A에서 만든 spy에만 기록됨
- B에서 새로 spy 생성 `깨끗한 spy` <br>→ 하지만 `not.toHaveBeenCalled()`의 **실패 가능성 ↑**

## 올바른 패턴

### spy는 `describe` 상단에서 딱 한 번만 생성

```ts
const spy = jest.spyOn(api, 'fetchData');

beforeEach(() => {
  spy.mockClear(); // 호출 기록만 초기화
});

test('호출 여부 확인', () => {
  api.fetchData();
  expect(spy).toHaveBeenCalled(); // ✅ 안정적
});
```

#### 🤔 `beforeEach()` 안에서 spy 호출하면 되지 않나요?

```ts
let spy;

beforeEach(() => {
  spy = jest.spyOn(api, 'fetchData');
  api.fetchData(); // 여기에 호출 포함
});
```

**이 코드는...**

1. **spy 인스턴스가 매번 새로 생성**
   → 이후 `mockClear()`나 **추적이 꼬일 가능성 ↑**
2. **spy가 중복 생성됨**
   → 성능 문제 및 Jest가 내부 상태 추적 혼란
3. **테스트 의도 불명확**  
   → 호출이 테스트 본문에 보이지 않아서, **왜 호출됐는지 맥락 파악이 어려움**

```ts
test('fetch 호출됨', () => {
  expect(spy).toHaveBeenCalled(); // 🤔 호출이 어디서 된 거지?
});
```

## 추천 구조

```ts
const spy = jest.spyOn(api, 'fetchData');

beforeEach(() => {
  spy.mockClear();
});

test('명확한 호출 위치', () => {
  api.fetchData();
  expect(spy).toHaveBeenCalled();
});
```

> spy는 같은 인스턴스를 계속 유지해야  
> 호출 기록 추적과 상태 관리가 안정적으로 된다.
>
> 테스트마다 spy를 새로 만들면 기록이 꼬이고, 관리도 어려워진다.

## 📌 핵심 요약

| 상황                               | 권장 방식                                  |
| ---------------------------------- | ------------------------------------------ |
| 상태 공유가 필요한 DB/리소스       | `beforeAll()`                              |
| 매번 fresh한 mock/배열/객체가 필요 | `beforeEach()`                             |
| mock 전체 초기화                   | `jest.clearAllMocks()` / `resetAllMocks()` |
| spy 인스턴스 안정적으로 관리       | 상단 1회 선언 및 `mockClear()`로 초기화    |
| 호출 위치를 명확히 하고 싶을 때    | 테스트 안에서 명시적으로 호출              |

# 인사이트

> 테스트는 **동작의 정확성**과 **의도의 명확함**의 미학
> <br>→ spy/mocks도 생명주기를 깔끔히 정리하면  
> → **디버깅·협업·신뢰성** 전부 올라감 💪
