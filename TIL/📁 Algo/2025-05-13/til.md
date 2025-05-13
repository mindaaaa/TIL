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
