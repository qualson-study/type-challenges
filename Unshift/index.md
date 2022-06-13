### Unshift

- 연구
  ```tsx
  // try 1
  type Unshift<T extends unknown[], U> = [U, ...T];
  ```
- 현석
  push나 unshift나
  ```tsx
  type Unshift<T extends any[], U> = [U, ...T];
  ```
- 화랑
  ```powershell
  type Unshift<T extends any[], U> = [U, ...T]
  ```
- 찬모
  ```tsx
  // Try1
  type Unshift<T extends unknown[], U> = [U, ...T];
  ```
- 지선
  ```tsx
  type Result = Unshift<[1, 2], 0>; // [0, 1, 2,]

  type Unshift<T extends any[], U> = [U, ...T];
  ```
- 재영
  - Try1
    ```tsx
    type Unshift<T extends any[], U> = [U, ...T];
    ```
- 주운
  앞에 `Push`와 같은 느낌
  ```jsx
  type Unshift<T extends unknown[], U> = [U, ...T]
  ```
