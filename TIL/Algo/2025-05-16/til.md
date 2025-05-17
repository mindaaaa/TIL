## 📌 인사이트 요약

- 반복되는 삽입/삭제엔 `Heap`이 압도적으로 유리
- `MinHeap` 구조는 직접 구현<br>→ 시간복잡도 직관

> 디버깅 시에는 **인덱스 기반의 완전이진트리 구조**를 머릿속에 시각화

---

# 소수 판별

## 문제 요약

- 숫자 배열에서 서로 다른 3개의 수를 더했을 때 **소수인 경우의 수**

## 트러블 슈팅

```javascript
function checkPrime(number) {
  for (let i = 2; i <= Math.sqrt(number); i++) {
    if (number % i === 0) return false;
  }
  return true;
}
```

- `checkPrime`을 **호출하는 로직**이 틀렸음
- `return false`일 때 소수라고 착각해서 로직이 맞는데 다른 값이 나오는 상황🤯

### 개선 과정

- 한참 들여다보면서 소수일 때 `count++`하도록 바꿨다...🙇‍♂️
- 조합 로직을 `DFS`로 직접 구현했기 때문에 구조적으로는 합격

## 📌 배운 점

- `reduce`나 `map`처럼 한 줄로 쓰는 구문에서도 **실제 반환값이 뭔지 반드시 의식**
- 조합 문제는 DFS 백트래킹으로 구현하는 게 확장성 좋고 직관적

---

# 날짜 계산

## 문제 요약

- 2016년 a월 b일이 무슨 요일일까?

## 접근

- `month = [31, 29, 31, 30, ...]` 배열을 이용해 a-1월까지 일수를 누적 후 b일 더하기

```javascript
let days = b - 1;
for (let i = 0; i < a - 1; i++) {
  days += month[i];
}
```

### ⚠️ 고민된 부분

- `new Date(2016, a - 1, b)`를 이용하면 자동처리<br>→ **계산 최적화**를 위해 직접 계산하면 어떨까?

### 성능 비교

- **직접 누적합 방식**이 더 빠르고 깔끔
- new Date는 틀리지 않았다.<br>→ 다만 **불필요한 API 호출**이 될 수 있을 뿐...

## 배운 점

- **직접 연산이 유리**
- `Date` 객체는 유연하지만 **정확한 계산을 눈으로 검증하기 어려움**
- 그 날을 포함하지 않기 위해 `b - 1`부터 시작

---

# 명예의 전당

## 문제 요약

- 명예의 전당에 올라간 `상위 점수 K개` 중 가장 낮은 점수를 매일 출력

## 초기 접근

```javascript
if (score > hall[0]) {
  hall[0] = score;
  hall.sort((a, b) => a - b);
}
```

- 매번 `sort()` 호출 → 성능 매우 비효율적
- 명확한 `MinHeap` 구조가 있으면,<br> → 최소값 접근이 `O(1)`<br> → 삽입/제거 `O(log n)`

### 📌 MinHeap 설계 아이디어

- `insert()` 시 `_heapifyUp()` <br>→ 말단에서 부모와 비교해 올라감
- `extract()` 시 `_heapifyDown()` <br>→ 루트 제거 후, 자식과 비교하며 내려감

> 트리 구조를 배열 인덱스로 관리

```javascript
  insert(value) {
    this.heap.push(value);
    this._heapifyUp();
  }

  _heapifyUp() {
    let idx = this.heap.length - 1;
    const inserted = this.heap[idx];

    while (idx > 0) {
      // ...생략
    }
    this.heap[idx] = inserted;
  }
```

```javascript
  extract() {
    if (this.heap.length <= 1) return this.heap.pop();

    const min = this.heap[0];
    this.heap[0] = this.heap.pop();
    this._heapifyDown();
    return min;
  }

  _heapifyDown() {
    let idx = 0;
    const length = this.heap.length;

    while (true) {
      // ...생략
    }
```
