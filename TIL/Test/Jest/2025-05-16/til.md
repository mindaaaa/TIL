## 📌 참고 레퍼런스

> [테스트 with Jest: 제로초에게 제대로 배우기](https://www.inflearn.com/course/%ED%85%8C%EC%8A%A4%ED%8A%B8-with-jest-%EC%A0%9C%EB%A1%9C%EC%B4%88) 
> [Jest](https://jestjs.io/docs/expect)

---
# `toThrow()`와 `try/catch` 예외 테스트

## 👀  왜 `() => error()`로 감싸야 `toThrow()`가 작동할까?

>`toThrow()`는 **예외를 던지는 함수 자체**를 전달해야 동작 
함수를 **즉시 실행하면** 테스트 코드 자체가 죽는다☠️

### 📌 Bad Case
```ts
function throwError() {
  throw new Error('🔥');
}

test('에러 발생 테스트', () => {
  expect(throwError()).toThrow('🔥'); // 에러 발생
});
```

### 📌 Good Case
```ts
test('에러 발생 테스트', () => {
  expect(() => throwError()).toThrow('🔥');
});
```

>`() => throwError()`처럼 함수를 감싸서 **지연 호출 형태**로 전달

---

# `try/catch` 안에서는 `toThrow()`를 못 쓴다고?

>`try/catch` 내부의 에러는 이미 **잡힌 상태**이기 때문에  
`toThrow()`처럼 예외 발생 여부를 테스트하는 matcher는 쓸 수 없음

```ts
function fail() {
  throw new Error('oops');
}

test('catch error object', () => {
  try {
    fail();
  } catch (err) {
    expect(err).toStrictEqual(new Error('oops')); // 이렇게 비교
  }
});
```

### 📌 예외가 실제로 발생했는지 확인
- `expect(() => fn().toThrow()`

### 📌 예외 객체의 값을 직접 비교
- `try/catch`와 `toStrictEqual()`

---

# `mockClear`, `mockReset`, `mockRestore`

### 📌 차이점 요약

| 메서드             | 호출 기록 초기화 | 구현 초기화 | 원래 함수 복원 | 주로 쓰는 상황 예시                        |
| --------------- | --------- | ------ | -------- | ---------------------------------- |
| `mockClear()`   | ✅         | ❌      | ❌        | 호출 횟수만 리셋 <br>📌 여러 테스트 간 공유 시     |
| `mockReset()`   | ✅         | ✅      | ❌        | 동작 구현까지 초기화<br>📌 다시 설정            |
| `mockRestore()` | ✅         | ✅      | ✅        | 원래 함수로 완전히 복원 <br>📌  `spyOn()` 전용 |

## 1. `mockClear()`

```ts
const mockFn = jest.fn();
mockFn();

mockFn.mockClear();
expect(mockFn).not.toHaveBeenCalled();
```

- 호출 기록만 초기화
- 이전 구현은 유지됨
>**보통 `beforeEach()`에서 많이 사용**

## 2. `mockReset()`

```ts
const mockFn = jest.fn(() => 'hello');

mockFn.mockReset();
mockFn(); // returns undefined
```

- 호출 기록/구현 모두 초기화
- `mockImplementation`도 제거

## 3. `mockRestore()` 

>spyOn() 전용

```ts
const obj = { greet: () => 'hi' };

jest.spyOn(obj, 'greet').mockImplementation(() => 'bye');

obj.greet.mockRestore(); // 원래 함수로 복원

expect(obj.greet()).toBe('hi');
```
- `spyOn()`으로 바꾼 함수만 사용 가능
- 테스트 후 **원래 함수로 되돌릴 때** 필수

>원본이 계속 mock 상태로 오염되는 것을 방지
    
### 요약
- `mockClear()`는 호출 기록만 초기화
- `mockReset()`은 구현 자체도 초기화
- `mockRestore()`는 `jest.spyOn()`으로 감싼 원래 함수를 **완전히 복원**<br>→ 테스트 간 오염 방지
### 💡TIP
- `jest.fn()`으로 만든 mock은 `mockClear()`와 `mockReset()`만 사용
- `jest.spyOn()`으로 만든 mock은 테스트 후 꼭 `mockRestore()`로 **원복**
- `beforeEach()`에 `mockClear()`를 사용하면 호출 기록이 꼬이지 않음