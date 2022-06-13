### Pop

- 연구
  # **Pop**
  Implement a generic `Pop<T>` that takes an Array `T` and returns an Array without it's last element.
  For example
  ```tsx
  type arr1 = ["a", "b", "c", "d"];
  type arr2 = [3, 2, 1];

  type re1 = Pop<arr1>; // expected to be ['a', 'b', 'c']
  type re2 = Pop<arr2>; // expected to be [3, 2]
  ```
  ```tsx
  // try 1
  type Pop<T extends any[]> = T extends [...infer K, any] ? K : never;

  /* _____________ Test Cases _____________ */
  import { Equal, Expect } from "@type-challenges/utils";

  type cases = [
    Expect<Equal<Pop<[3, 2, 1]>, [3, 2]>>,
    Expect<Equal<Pop<["a", "b", "c", "d"]>, ["a", "b", "c"]>>
  ];
  ```
- 현석
  ```tsx
  type Pop<T extends any[]> = T extends [...infer A, any] ? A : never;
  ```
- 화랑
  ```jsx

  // 1차 시도. 이전 문제를 조금만 응용하면 되서 바로 풀었다
  type Pop<T extends any[]> = T extends [...infer S, infer U] ? S : never
  ```
- 찬모
  ```tsx
  // Try1
  type Pop<T extends any[]> = T extends [...infer R, infer F] ? R : never;
  ```
  위에 문제를 풀면서 생각했던 부분을 이용하니 한 번에 통과하였다. 😅
  `infer F` 의 부분에 unkonw, any 다 들어갈 수 있다. 타입이 존재한다라는 의미로 넣으면 되는 것 같다.
- 지선
  ```tsx
  // Pop 구현
  type arr1 = ['a', 'b', 'c', 'd']
  type arr2 = [3, 2, 1]

  type re1 = Pop<arr1> // expected to be ['a', 'b', 'c']
  type re2 = Pop<arr2> // expected to be [3, 2]
  ---------------------

  // last에서 사용했던 방식과 동일하게 하면 될듯?
  //try1 👌
  type Pop<T extends any[]> = T extends [...infer Y, infer Z] ? Y : never;
  ```
- 재영
  - try1
    ```tsx
    type Pop<T extends any[]> = T extends [...infer R, infer E] ? R : any;
    ```
