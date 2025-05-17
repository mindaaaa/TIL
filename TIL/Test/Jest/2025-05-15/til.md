## 📌 참고 레퍼런스

> [테스트 with Jest: 제로초에게 제대로 배우기](https://www.inflearn.com/course/%ED%85%8C%EC%8A%A4%ED%8A%B8-with-jest-%EC%A0%9C%EB%A1%9C%EC%B4%88) 
> [Jest](https://jestjs.io/docs/expect)

# Promise 기반 코드 테스트

## 1. `resolves` / `rejects` 사용법

```ts
// ✅ 성공 케이스
test('promise resolves', () => {
  return expect(Promise.resolve('hello')).resolves.toBe('hello');
});

// ✅ 실패 케이스
test('promise rejects', () => {
  return expect(Promise.reject(new Error('fail'))).rejects.toThrow('fail');
});
```

- `return` 키워드를 꼭 붙여야 **Jest가 비동기 작업이 끝날 때까지** 기다림
## 2. `async/await` 방식

```ts
// ✅ 성공 케이스
test('promise resolves with await', async () => {
  const result = await someAsyncFn();
  expect(result).toBe('hello');
});

// ✅ 실패 케이스 (try/catch 사용)
test('promise rejects with await', async () => {
  try {
    await someFailingFn();
  } catch (err) {
    expect(err).toBeInstanceOf(Error);
    expect(err.message).toBe('fail');
  }
});
```

## `3. then/catch` 방식

```ts
test('promise resolves with then', () => {
  return someAsyncFn().then(result => {
    expect(result).toBe('hello');
  });
});

test('promise rejects with catch', () => {
  return someFailingFn().catch(err => {
    expect(err).toBeInstanceOf(Error);
  });
});
```

---

# `mockResolvedValue`와 `mockRejectedValue`

### 📌 요약

- `mockResolvedValue(value)` <br>→ `Promise.resolve(value)`처럼 동작
- `mockRejectedValue(error)` <br>→ `Promise.reject(error)`처럼 동작
## 예제

```ts
const fetchData = jest.fn().mockResolvedValue({ data: 'hello' });

test('resolves successfully', async () => {
  const result = await fetchData();
  expect(result.data).toBe('hello');
});

const failData = jest.fn().mockRejectedValue(new Error('fail'));

test('rejects with error', async () => {
  await expect(failData()).rejects.toThrow('fail');
});
```

### 왜 필요할까?

- 실제 API, DB 등을 호출하지 않고도 성공/실패 시나리오를 <br>**가짜로 시뮬레이션**할 수 있음
- 테스트를 빠르고 안정적으로 유지

---
### 👀 모든 콜백 함수를 Promise로 바꿀 수 있을까?

> 모든 콜백을 Promise로 바꿀 수는 없지만, <br>**대부분의 비동기 콜백 함수**는 `Promise`로 래핑해 사용

#### 📌 콜백과 Promise 개념

| 개념      | 설명                                          |
| ------- | ------------------------------------------- |
| 콜백 함수   | 어떤 작업이 끝난 후 실행할 함수<br>보통 인자로 전달됨            |
| Promise | 비동기 작업을 객체로 표현<br>`resolve` 또는 `reject`로 제어 |

##### 전환 가능한 경우, 1회성 비동기 콜백 (Node-style)

```ts
function readFilePromise(path) {
  return new Promise((resolve, reject) => {
    fs.readFile(path, 'utf8', (err, data) => {
      if (err) return reject(err);
      resolve(data);
    });
  });
}
```

- `resolve/reject`로 자연스럽게 변환 가능
- `fs`, `setTimeout`, DB 호출 등에서 자주 사용

##### 전환이 어려운 경우, 반복 호출되는 콜백 (ex. 이벤트 리스너)

```ts
button.addEventListener('click', () => {
  console.log('Clicked!');
});
```

- 클릭처럼 반복적으로 발생하는 이벤트는 `Promise`와 맞지 않음
- 단, 한 번만 발생하면 `once: true` 옵션으로 감싸는 건 가능

```ts
function waitForClick(button) {
  return new Promise(resolve => {
    button.addEventListener('click', resolve, { once: true });
  });
}
```

> Node-style 콜백에서는 **Primise 전환**이 적극 권장됨

#### 📌 요약
- 대부분의 1회성 비동기 콜백 함수는 Promise로 바꾸기
- **이벤트 리스너처럼 반복 발생하는 콜백은 Promise로 감싸지 않는 것이 일반적**
- Node.js 환경에서는 `err-first` 콜백을 `Promise`로 바꿔 테스트 코드 통일성을 확보

---
# spyOn과 jest.fn()
|도구|목적|주로 쓰는 상황|
|---|---|---|
|`jest.spyOn()`|**실제 객체의 메서드 감시 & 교체**|기존 객체/모듈의 메서드에 스파이 심기|
|`jest.fn()`|**스스로 mock 함수 생성**|의존성이 없거나, 가짜 함수가 필요한 경우|
## 📌 jest.spyOn()
```ts
const user = {
  getName: () => 'minda'
};

test('calls getName', () => {
  const spy = jest.spyOn(user, 'getName');
  user.getName();
  expect(spy).toHaveBeenCalled();
});
```
- 원본 `user.getName()` 메서드에 스파이를 *끼워 넣는* 방식
- 기본 동작은 유지되지만, <br>`mockImplementation()` 등으로 덮어쓸 수 있음
## 📌 jest.fn()
```ts
const mockFn = jest.fn(() => 'mocked');

test('custom mock fn', () => {
  expect(mockFn()).toBe('mocked');
  expect(mockFn).toHaveBeenCalled();
});
```

- 완전히 새로운 가짜 함수 생성
	- `props`로 넘길 콜백
	- API 응답 핸들러 등

| 상황                           | 추천 방식          | 이유                        |
| ---------------------------- | -------------- | ------------------------- |
| 기존 객체/모듈의 메서드를 감시하고 싶을 때     | `jest.spyOn()` | 실제 동작 감시/조건부 Mock 가능      |
| 새로운 mock 함수를 테스트용으로 만들고 싶을 때 | `jest.fn()`    | - 콜백 전달<br>- 의존성 주입 테스트 등 |
| 객체 없이 독립적인 함수 로직만 mock할 때    | `jest.fn()`    | 가장 유연함                    |

## 주의 사항

- `spyOn()`은 **기존 객체 메서드가 있어야만 사용 가능**  
- `jest.fn()`은 어디든 쓸 수 있는 **독립된 mock 함수**
    → 인자로 넘기는 콜백 함수 mock에도 아주 유용

## 📌 요약

>Jest에서 mock 함수를 다룰 때는 <br>
`jest.fn()`과 `jest.spyOn()`의 차이를 명확히 이해하는 것이 아주 중요

- `jest.fn()`은 새로운 mock 함수를 생성<br>→ 주로 콜백이나 의존성 주입 함수에 사용
- `jest.spyOn()`은 기존 객체의 메서드에 스파이를 심음<br>→ 호출 여부를 감시하거나 동작을 교체할 때 사용

>특히 `spyOn`은 객체에 존재하는 메서드여야만 사용할 수 있다.
