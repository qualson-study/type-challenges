### Push

- 연구
  ```tsx
  // try 1

  type Push<T extends unknown[], U> = [...T, U];
  ```
- 현석
  concat과 비슷했다
  ```tsx
  type Push<T extends any[], U> = [...T, U];
  ```
- 화랑
  ```powershell
  type Push<T extends any[], U> = [...T, U]
  ```
- 찬모
  ```tsx
  // Try1
  type Push<T extends unknown[], U> = [...T, U];
  ```
- 지선
  ```tsx
  type Result = Push<[1, 2], "3">; // [1, 2, '3']

  // concat이랑 흡사. 고민 없이 푼듯
  type Push<T extends any[], U> = [...T, U];
  ```
- 재영
  - Try1
  ```tsx
  type Push<T extends any[], U> = [...T, U];
  ```
- 주운
  - 이번에도 역시 답을 먼저 확인😇
  ```jsx
  type Push<T extends any[], U> = [...T, U]
  ```
  ***
  T의 타입을 `any[]`로 범위를 좁히는건데, 다른 사람들의 답변을 보니 `any` 대신 `unknown`으로 쓴 분들도 있는데 뭐가 다른건지...
