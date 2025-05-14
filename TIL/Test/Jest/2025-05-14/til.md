# TypeScript에 Jest 설정하기

## 목표

타입스크립트 기반 프로젝트에 Jest를 도입하여 테스트 환경을 구축한다.  
Babel 없이 `ts-jest`만으로 구성하며, `import/export` 문법(ESM)을 유지한다.

## 설치

### 1. 기본 설치

```bash
npm i -D jest ts-jest @types/jest
```

| 패키지        | 설명                                        |
| ------------- | ------------------------------------------- |
| `jest`        | 테스트 러너                                 |
| `ts-jest`     | 타입스크립트를 Jest에서 실행할 수 있게 함   |
| `@types/jest` | Jest 함수의 타입 정의를 제공 (ex. `expect`) |

> `-D`는 `--save-dev`의 축약어이며, `devDependencies`에 패키지를 설치한다.  
> 실행 시 필요한 의존성과 개발용 도구를 분리하여 **배포 최적화 및 관리 용이성** 확보한다.

### 2. `ts-jest` 설정 초기화

```bash
npx ts-jest config:init
```

- `jest.config.js` 또는 `jest.config.ts` 파일이 생성됨

### 3. 타입스크립트 설정 `tsconfig.json`

```bash
npx tsc --init
```

이후 다음 설정을 포함한다.

```json
{
  "compilerOptions": {
    "target": "ES6",
    "module": "CommonJS",
    "esModuleInterop": true,
    "strict": true,
    "skipLibCheck": true
  }
}
```

- `module: "CommonJS"` → Jest가 요구하는 모듈 시스템
- `esModuleInterop: true` → `import fs from 'fs'` 같은 문법 허용

### 4. Jest 설정 `jest.config.mjs`

`ESM` 방식을 유지하기 위해 `package.json`에 `"type": "module"`을 설정하였다면<br>`jest.config.js`에서 `export default` 문법이 동작하지 않으므로, **확장자를 `.mjs`로 변경**해야 한다.

```bash
mv jest.config.js jest.config.mjs
```

```javascript
// jest.config.mjs
export default {
  preset: 'ts-jest',
  testEnvironment: 'node',
};
```

---

## 시행착오

### ⚠️ SyntaxError: Unexpected token 'export'

```bash
export default {
^^^^^^
```

- `export default` 문법이 `jest.config.js`에서 **SyntaxError**를 유발했다.

#### 원인 분석

- `package.json`에 `"type": "module"`을 넣으면 **ESM 방식**으로 해석된다.
- **Jest는 기본적으로 CommonJS**환경으로 `jest.config.js`를 실행한다.<br> → 충돌 발생🧨🧨

#### 해결 방법

- `jest.config.js` → `jest.config.mjs` 확장자 변경
- `.mjs` 확장자는 Node와 Jest 모두에게 **ESM 파일**임을 명확하게 알려준다.

```json
export default {
  preset: 'ts-jest',
  testEnvironment: 'node',
};
```

##### 👀 왜 가능할까?

- Jest는 `.mjs` 확장자에 ESM을 허용해주기 때문에 `export default`를 문제 없이 해석함
- ESM 환경과 Jest의 기본 CJS 환경 충돌을 **파일 확장자**로 해결한 방법

---

### ⚠️ `tsconfig.json`이 없는 문제

- `ts-jest`는 내부적으로 `tsconfig.json`을 사용해 타입 컴파일을 진행
- `tsconfig.json`이 없거나 설정이 누락되면 `import` 에러 발생

#### 해결 방법

```bash
npx tsc --init
```

```json
{
  "compilerOptions": {
    "module": "CommonJS",
    "esModuleInterop": true
  }
}
```

##### 👀 왜 가능할까?

- `module: "CommonJS"` <br>→ Jest가 요구하는 모듈 시스템과 맞춤
- `esModuleInterop: true` <br>→ CommonJS 모듈을 ESM 스타일로 import할 수 있게 함 `import fs from 'fs'` 등

---

### `jest.config.js` 파일의 `preset` 설정

> 문제가 발생하지 않았다면 생략 가능

```json
transform: {
  "^.+\\.tsx?$": ["ts-jest", {}]
}
```

#### 문제점

- transform 설정으로 `ts-jest`의 모든 기능을 온전히 활용하지 못할 수 있음<br>→ `tsconfig` 반영 등
- 수동 설정은 실수 가능성이 있음

#### 방안

```json
export default {
  preset: 'ts-jest',
  testEnvironment: 'node',
};
```

- `preset: 'ts-jest` 추가
- transform 생략 → 자동 적용

##### 👀 왜 가능할까?

- `preset: 'ts-jest`는 `ts-jest`가 추천하는 표준 구성
- 내부적으로 `transform`, `diagnostics`, `tsconfig` 경로까지 모두 자동 설정

## 📁 디렉토리 구조

```shell
jest-lecture/
├── src/
│   └── sum.ts
├── test/
│   └── sum.spec.ts
├── tsconfig.json
├── jest.config.mjs
├── package.json

```

## 배우고 느낀 것

#### 📌 배운 점

- `"type": "module"` 사용 시에는 `jest.config.mjs`로 확장자를 명시한다.
- `preset: "ts-jest"`는 단순한 줄 하나지만, **transform 설정을 자동화**해주므로 실수를 줄이고 Jest를 안정적으로 돌리는 핵심 키였다.

#### 📌 느낀 점

- `TypeScript/Jest` 환경은 사소한 충돌이 많지만, <br>**모듈 시스템(ESM vs CJS)과 설정 파일 우선순위**만 이해하면 해결 가능하다.
- 특히 `"type": "module"`과 `jest.config.js`의 충돌은 아주 흔한 문제였다.

- `tsconfig.json` 설정은 **Jest와 타입스크립트 컴파일러 모두에 영향**을 미치므로, 프로젝트 초기부터 정확히 잡아두는 것이 중요하다.
