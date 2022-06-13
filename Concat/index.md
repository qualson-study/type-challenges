### Concat

[문제](https://www.typescriptlang.org/play?#code/PQKgUABBCsDMsQLQQMIHsB2BjAhgF0iUWJMICMBPCAQQwBMAnAUyoGkGcBnNAN04GsqACgACZZrAAM-DgDYAnFk4BKCAGImXKmpwMOFQoTXGIARQCuTTngCWmQ1ACSAWwAOAGybOmGPBDwAFkwQAFI4PDgAylgMNq5+AAbUejgUAHRYmLh4CRAAZubYtpgQNhj+Qf4UrsGcFNZeaTRVNf44-FYVwXgA7mgQugDm5t6+nE0AKpVo5niusxCcATPudBBkwTgQGEw9AylUgfil2O7mdJ1l83icJxDueAwQaAwXDA4QAGIvEEwAHjg3J4PglQTdCHhqsEAEpWcwPCAAXlQWXwAB4ANoARgAugAaCAYgBMOIAfBBgMBfn8alg8Ew1nh+htCViCSTCKCEh9yQA1Gy7Z7lADiNjwAAlzGQAFwQAJ4OacaWUm5YAJpABW4xeg2AcFgYBAwDAJtAEAA+pardarRAAJozJ7oC4QcVMZgWm1e80QI0myGtdDYdETAkAVXJyJwGAMppAnu91ogEysfhQXE6iZtvuNNjcLz8AG8IABRACO5hw7gJJZpTDpEAAvvkGGhnBAAOQiANMRBqqueDCDKzAWY2dycDv+qEQXCcTrIjGEWu0vBo8uV9xooPZTH4wlkgkYsmkvHLut09cVqvb1Fr49H3Gn1kns9QFf1tcbm879HY9n7hisAEgALIerLsgSwEQGBpKnueq5Xput7BveHZYh2kGdrAHaAXkVbzgSZBoGgnjRgSHYgbhz4YuhmEQESFE4QS+ETkwREkWRGAUVRr5gDiJpxgmWY+p85gMIE7oQJE9KuLcIm2n6oCEOSkQBLowQUI6iykWOmBKnKCpycqwCquqWppDqerwMA0acD07oqRA-KCtwZzFBgBnyoqJlmZq2oMLq+rAG5emeU5ACyLzBCg6nuIOw5eUZSoqpwar+ZZgWGsaYBAA)

- 연구
  ### **Concat**
  Implement the JavaScript `Array.concat` function in the type system. A type takes the two arguments. The output should be a new array that includes inputs in ltr order
  For example
  ```tsx
  type Result = Concat<[1], [2]>; // expected to be [1, 2]
  ```
  ```tsx
  // try 1
  // 혹시나해서 스프레드를 써봤는데 ...T 부분이 빨개지면서 안된다, 에러를 확인하니 A rest element type must be an array type. 이런 에러가 떴다. T 와 U 에 배열이 올거란 걸 알려줘봐야겠다.

  type Concat<T, U> = [...T, ...U];

  /* _____________ Test Cases _____________ */
  import { Equal, Expect } from "@type-challenges/utils";

  type cases = [
    Expect<Equal<Concat<[], []>, []>>,
    Expect<Equal<Concat<[], [1]>, [1]>>,
    Expect<Equal<Concat<[1, 2], [3, 4]>, [1, 2, 3, 4]>>,
    Expect<
      Equal<
        Concat<["1", 2, "3"], [false, boolean, "4"]>,
        ["1", 2, "3", false, boolean, "4"]
      >
    >
  ];

  // try 2
  // 배열이라고 알려주니 똑똑한 타입스크립트가 잘 처리해준다. ^^
  type Concat<T extends unknown[], U extends unknown[]> = [...T, ...U];

  /* _____________ Test Cases _____________ */
  import { Equal, Expect } from "@type-challenges/utils";

  type cases = [
    Expect<Equal<Concat<[], []>, []>>,
    Expect<Equal<Concat<[], [1]>, [1]>>,
    Expect<Equal<Concat<[1, 2], [3, 4]>, [1, 2, 3, 4]>>,
    Expect<
      Equal<
        Concat<["1", 2, "3"], [false, boolean, "4"]>,
        ["1", 2, "3", false, boolean, "4"]
      >
    >
  ];
  ```
- 현석
  한참 삽질하다가 얻어걸림
  ```tsx
  type Concat<T extends any[], U extends any[]> = [...T, ...U];
  // 타입에서 Spread 풀어쓰는게 된다고?
  ```
- 화랑
  두 번째로 고민 없이 푼 듯
  T 와 U 를 더하는 걸 어떻게 쓰지 하다가, 손이 가는대로(?) 적어봤더니 성공
  ```powershell
  type Concat<T extends any[], U extends any[]> = [...T, ...U]
  ```
- 찬모
  도전~🚀
  ```tsx
  // Try1
  type Concat<T extends any[], U extends any[]> = Array<T & U>; // wrong

  // Try2
  type Concat<T extends any[], U extends any[]> = [...T, ...U]; // 🎉
  ```
  `Array<T & U>` 는 처음엔 될까 했는데, 배열 안에 배열 타입?? 말이 안됨을 자각 🤪
  다른 풀이
  ```tsx
  type Concat<T extends unknown[], U extends unknown[]> = [...T, ...U];
  ```
  최상위 타입인 `unknown`을 사용한 풀이가 많이 보였다.
- 지선
  ```tsx
  type Result = Concat<[1], [2]> // expected to be [1, 2]

  1. type Concat<T, U> = [...T, ...U]
  //A rest element type must be an array type.

  [🙆‍♀️]
  2. type Concat<T extends any[], U extends any[]> = [...T, ...U]
  ```
- 재영
  - 요구사항

    ```tsx
    type Result = Concat<[1], [2]>; // expected to be [1, 2]
    ```

  - 아이디어

    - 그냥 Spread Syntax 쓰면 되나?
    - 되네?!

  - Try
    ```tsx
    // perfect -> 제네릭 받을 때 "T랑 U는 배열이야" 라는 것만 알려주면 에러 안나고 잘 됨.
    type Concat<T extends any[], U extends any[]> = [...T, ...U];
    ```
