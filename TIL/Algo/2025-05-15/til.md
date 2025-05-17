## 📌 인사이트 요약

- `콜라 교환 문제` 수식화에 대한 깨달음
- `비밀지도` 비트 연산
- `문자열 숫자 매핑` Map과 흐름 제어
  - `forEach`와 `for...of`의 흐름 제어 가능 여부
  - `.get()`과 `.set()`의 기본 사용법 기본 사용법
- 치환`split/join, replace`이 더 깔끔하게 풀리는 문제도 많다👀

---

# 콜라 문제

## 문제 요약

- 병이 `n`개 있을 때, 빈 병 `a`개를 내면 `b`개를 받는 구조
- 받은 병을 마셔 다시 빈 병이 생김<br>→ **반복**

> 최종적으로 **받은 콜라의 총합**을 구하기

## 초기 접근

- `while`문을 통해 **반복 교환**

```javascript
while (n >= a) {
  const service = Math.floor(n / a) * b;
  sum += service;

  n = service + (n % a);
}
```

### 👀 수식화를 이용해 한 줄로 풀 수 있다고?

```javascript
solution = (a, b, n) => Math.floor(Math.max(n - b, 0) / (a - b)) * b;
```

> `시뮬레이션` 문제처럼 보이지만, _몇 번의 교환이 가능한가?_ <br>→ 등차 수열 공식화 가능

#### 📌 풀이

- `n - b`를 한 이유는 **최초 교환이 불가능한 경우 제외**
- `a - b`는 **1회 교환 후 순환에 필요한 병 수 변화량**을 의미
- **보너스만 구하면 되기 때문에, <br>누적 병 자체를 더하지 않아도 된다는 점을 이용**

---

# 숫자 문자열과 영단어

## 문제 요약

- 숫자의 일부 자릿수가 영단어로 바뀐 문자열 `s`가 매개변수
- `s`가 의미하는 원래 숫자를 return

### 📌 입출력 예시

- 1478 → "one4seveneight"
- 234567 → "23four5six7"
- 10203 → "1zerotwozero3"

## 문제 접근

> `Map`은 빠른 O(1) 조회 덕분에 `반복 문자열 매핑`에 적합

- 문자열을 숫자로 빠르게 매핑하기 위해 `Map` 사용
- 객체보다 **key의 자료형 제약**이 없으며 **입력 순서 보장**

## 실수 모음🤯

`Map` 자료 구조를 제대로 써보는 게 처음이라 많이 헤맴

### 실수 1. `Map` 사용 시 스프레드 연산 오용

```js
map.get(...str); // map.get('o', 'n', 'e') → undefined
```

- `['o','n','e']` 배열을 펼쳐서 여러 인자로 넘기는 코드
- `map.get(str.join(''))`을 통해 조회

### 실수 2. 문자열 누적 처리 꼬임

```javascript
let str = '';
forEach((v) => {
  str += v;
  // str 초기화를 적절히 안해서 중복 누적됨
});
```

### 실수 3. `forEach` 흐름 제어 실패

- `return`으로 루프 탈출하려 했지만 콜백만 종료됨
- `continue` 사용 불가

#### 📌 해결법

##### `for...of` 구조로 변환

- `continue`, `break` 사용 가능
- 코드 흐름이 훨씬 명확해지고 디버깅 쉬워짐

## 배운 점

#### 📌 `Map` 초기화 방식

##### 이전 방식

```javascript
map.set('zero', 0);
map.set('one', 1); // ...
```

##### 배운 방식

```javascript
const nums = ['zero', 'one', ..., 'nine'];
const map = new Map(nums.map((word, idx) => [word, idx]));
```

### 👀 replaceAll을 이용하면 한 번에 풀린다고?

```javascript
function solution(s) {
  const numbers = [
    'zero',
    'one',
    'two',
    'three',
    'four',
    'five',
    'six',
    'seven',
    'eight',
    'nine',
  ];

  for (let i = 0; i < numbers.length; i++) {
    s = s.replaceAll(numbers[i], i);
  }

  return Number(s);
}
```

- 성능상 `Map`을 이용한 경우가 조금 더 좋지만 약간 부러웠다...😹

---

## 비밀지도

### 핵심 흐름

- 각 줄을 `toString(2)`으로 이진수 변환
- 두 배열의 같은 인덱스를 OR 연산
- padStart로 앞자리를 0으로 채워 자릿수 맞춤
- `replace(/1/g, "#").replace(/0/g, " ")`로 시각화

### 👀 replace는 `split`과 `join`의 합동이라고?

```javascript
for (let i = 0; i < words.length; i++) {
  s = s.split(words[i]).join(i);
}
```
