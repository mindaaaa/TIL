# 시저 암호 순환 처리 및 음수 시프트 보정

## 개념 요약

시저 암호(Caesar Cipher)는 알파벳을 일정한 거리만큼 밀어 암호화하는 방식이다.  
문제를 풀면서 아래와 같은 요소들을 고려하게 되었다.

- 알파벳의 순환 구조 (z → a, Z → A)
- 대소문자 구분 처리
- 음수 시프트(복호화) 시의 보정
- 공백 및 특수문자 예외 처리

## 순환 처리 (양수 시프트)

```js
function shiftChar(char, n) {
  const code = char.charCodeAt(0);
  const base = code >= 97 ? 97 : 65; // 소문자 대문자 결정 기준
  return String.fromCharCode(((code - base + n) % 26) + base);
}
```

- 기준점(base)을 빼고 연산 → `a`부터 0, ..., `z`는 25로 처리
- `% 26`으로 z를 넘어가면 자동으로 순환
- 다시 base를 더해 알파벳으로 복원

## 복호화 (음수 시프트 시 보정)

```javascript
function shiftChar(char, n) {
  const code = char.charCodeAt(0);
  const base = code >= 97 ? 97 : 65;
  const offset = (((code - base + n) % 26) + 26) % 26;
  return String.fromCharCode(offset + base);
}
```

- `%` 연산에서 음수 대응 위해 **`+ 26`을 한 번 더 보정**
- `'a'`에 `-1`을 더하면 `'z'`가 되도록 함
- 자바스크립트 `%`는 진짜 나머지가 아니라 부호 유지(mod 연산), 그래서 보정 필요

## 배운 점

- 시저 암호 구현은 **modulo 연산의 동작 방식**을 정확히 이해해야 오류 없이 작동함
- 자바스크립트에서 `%`는 **음수 처리 시 `+ 26` 보정**이 필요
- 이 패턴은 **암호화/복호화**, **원형 큐**, **문자열 인덱스 순환** 등 여러 곳에 응용 가능

## 적용 예시

```javascript
function caesarCipher(s, n) {
  return [...s]
    .map((char) => {
      if (!/[a-zA-Z]/.test(char)) return char;
      return shiftChar(char, n);
    })
    .join('');
}
```

## 📌 예시 결과

```javascript
caesarCipher('abc', 1); // "bcd"
caesarCipher('xyz', 2); // "zab"
caesarCipher('A B C', -1); // "Z A B"
caesarCipher('z', -1); // "y"
```

---

# 숫자 문자열과 영단어 문제 TIL

## 문제 개요

주어진 문자열에서 숫자를 영단어로 표현한 부분을 실제 숫자로 바꾸는 문제.

```bash
"one4seveneight" → "1478"
"23four5six7" → "234567"
"2three45sixseven" → "234567"
```

## 시행착오 및 배운 점

### 1. `Map` 사용 시도 → 구조 이해 부족

```js
map.get(...str); // 예상과 다르게 작동
```

- `str = ['o', 'n', 'e']`인 경우, `...str`은 `'o', 'n', 'e'`로 펼쳐짐
- `Map.prototype.get()`은 한 개의 key만 받기 때문에 `map.get('o')`로 처리됨

#### 📌 배운 점

##### `Map` 초기화 간결화

```javascript
const map = new Map(
  [
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
  ].map((word, idx) => [word, idx])
);
```

### 2. 문자열 누적 로직 문제

```javascript
const str = [];
str.push(v); // 반복적으로 같은 값이 누적됨
```

- 한 글자씩 추가되지만 초기화 타이밍을 놓쳐서 str이 계속 길어짐

#### 📌 배운 점

##### Map/수동파싱

```javascript
function solution(s) {
  const map = new Map(
    [
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
    ].map((word, idx) => [word, idx])
  );

  let result = '';
  let str = '';

  for (const char of s) {
    if (!isNaN(char)) {
      result += char;
      str = '';
      continue;
    }

    str += char;
    if (map.has(str)) {
      result += map.get(str);
      str = '';
    }
  }

  return Number(result);
}
```

### 3. 반복문 흐름 오류 `forEach` vs `for...of`

```js
forEach((v) => {
  if (숫자조건) return;
  // ❌ 전체 루프가 아니라 콜백만 종료됨
});

for (const char of s) {
  if (숫자조건) continue; // 다음 문자로 정상 이동
}
```

- `forEach`에서는 `return`이 루프를 멈추지 않음 → `for...of`로 구조 변경

#### 📌 배운 점

| 루프 구조         | `return`                        | `continue`     | `break`   |
| ----------------- | ------------------------------- | -------------- | --------- |
| `forEach`         | 콜백만 종료 <br>⚠️ 루프 안 끊김 | 사용 불가      | 사용 불가 |
| `for...of`, `for` | 함수 전체 종료                  | 현재 반복 스킵 | 루프 종료 |

## 다른 풀이 훑어보기

### `split/join` 방식

```js
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
    s = s.split(numbers[i]).join(i);
  }

  return Number(s);
}
```

#### 📌 동작 흐름

```js
s = "one4seveneight"
s = s.split("one") → ["", "4seveneight"]
s = ["", "4seveneight"].join("1") → "14seveneight"
s = s.split("seven") → ["14", "eight"]
s = ["14", "eight"].join("7") → "147eight"
s = s.split("eight") → ["147", ""]
s = ["147", ""].join("8") → "1478"
```

> **`split().join()`은 **쪼개고, 그 틈에 뭔가를 끼워 넣는 것\*\*

<br>

```javascript
'apple banana banana apple'.split('banana').join('🍌');
// "apple 🍌 🍌 apple"
```

### replaceAll

```js
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

- 가독성이 가장 좋지만 **ES2021 이상에서만 지원**됨

## 성능 비교

| 방식               | 특징                 | 성능                                  |
| ------------------ | -------------------- | ------------------------------------- |
| `Map` + `for...of` | 루프 1회 + 직접 파싱 | ✅ 빠름                               |
| `split` + `join`   | 최대 10번 전체 순회  | ⚠️ 느려질 수 있음                     |
| `replaceAll`       | 가독성👍             | ⚠️ 브라우저/버전 제한<br>⚠️ 반복 많음 |

## 인사이트

- 자바스크립트 `%`의 음수 처리 방식과 시저 암호의 순환 처리 (보너스 학습)
- `Map` 사용법 & key 조합 방식 (`.get`, `.has`)
- `forEach` vs `for...of` 흐름 제어 차이 (return, continue)
  - 문자열 치환 패턴: `split + join` → 사실상 `replaceAll` 대체
