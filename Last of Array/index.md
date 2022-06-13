### Last of Array

- 연구
  # **Last of Array**
  Implement a generic `Last<T>` that takes an Array `T` and returns its last element.
  For example
  ```tsx
  type arr1 = ["a", "b", "c"];
  type arr2 = [3, 2, 1];

  type tail1 = Last<arr1>; // expected to be 'c'
  type tail2 = Last<arr2>; // expected to be 1
  ```
  ```tsx
  // try 1
  type Last<T extends any[]> = T extends [...any, infer K] ? K : never;

  type a = Last<[3, 2, 1]>;

  /* _____________ Test Cases _____________ */
  import { Equal, Expect } from "@type-challenges/utils";

  type cases = [
    Expect<Equal<Last<[3, 2, 1]>, 1>>,
    Expect<Equal<Last<[() => 123, { a: string }]>, { a: string }>>
  ];
  ```
- 현석
  Try 1.
  ```tsx
  type Last<T extends any[]> = T[T["length"] - 1]
  전혀 아니야
  ```
  Try 2.
  ```tsx
  type Last<T extends any[]> = T extends [...any, infer B] ? B : never;
  ```
- 화랑
  ```jsx
  // 1차 시도
  type Last<T extends any[]> = T[T["length"] - 1]

  // 2차 시도
  type Last<T extends any[]> = T extends [...any, infer U] ? U : never

  ```
- 찬모
  - _TypeScript 4.0 릴리즈 노트_ [영문](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-4-0.html) | [한글](https://han41858.tistory.com/52) ( _TypeScript 4.0 is recommended in this challenge 써있길래..)_
  **첫번째 든 생각**
  배열의 마지막이라는 것을 어떻게 표현할 수 있을까??
  → index `n - 1` 을 어떻게 표현하지??
  1. 길이를 알아야한다

  2. 길이에서 1을 빼는 것을 어떻게 타입으로 표현할 것인가?
     → `length - 1` 은 값이기 때문에 이런식으로 타입을 표현할 수 없다.
  ```tsx
  // 1) getLength
  type GetLength<T extends any[]> = T["length"];
  ```
  1번은 위와 같이 구현 가능하지만 2번은 어떻게 만들수 있을까?? 고민중...
  → 구글링 성공 ~ 😆
  ```tsx
  // 2) length - 1
  type LengthMinusOne<T extends any[]> = T extends [any, ...infer R]
    ? R["length"]
    : never;
  ```
  원래 의도는 GetLength를 이용하는 것이였는데, spread operator를 사용하다보니 GetLength가 필요없어진다.
  ```tsx
  // 완성코드
  type LengthMinusOne<T extends any[]> = T extends [any, ...infer R]
    ? R["length"]
    : never;

  type Last<T extends any[]> = T[LengthMinusOne<T>];
  ```
  ***
  아래는 원래 풀이였는데, length - 1을 표현한 것과 똑같은데, 무엇을 반환하게 하는냐의 차이!
  → `LengthMinusOne` 는 사실 돌아가는 풀이였다는....
  ```tsx
  // ⭕️
  type Last<T extends any[]> = T extends [...infer A, infer B] ? B : never;
  // ⭕️
  type Last<T extends any[]> = T extends [...any, infer B] ? B : never;

  // ❌
  type Last<T extends any[]> = T extends [...unknown, infer B] ? B : never;
  ```
  unknown인 경우는 error!!
  → `A rest element type must be an array type.`
  무엇이든 타입이 있는 것이 들어와야하는데 unknown은 타입을 모르는 것이기때문에 에러!
- 지선
  ```tsx
  // Last 구현하기
  type arr1 = ['a', 'b', 'c']
  type arr2 = [3, 2, 1]

  type tail1 = Last<arr1> // expected to be 'c'
  type tail2 = Last<arr2> // expected to be 1
  ---------------------

  // 배열 길이(T['length']) -1의 값을 반환해주면 되지 않을까?
  // try1
  type Last<T extends any[]> = T[T['length']-1]
  // type에서 -1안됨. T['length']는 타입인걸...

  //[answer 1](https://dev.to/miracleblue/how-2-typescript-get-the-last-item-type-from-a-tuple-of-types-3fh3)
  type Length<T extends any[]> =  T['length']
  type Prev<T extends number> = [-1, 0, 1, 2, 3, 4, 5][T];
  type Last<T extends any[]> = T[Prev<Length<T>>]

  //[answer 2](https://github.com/type-challenges/type-challenges/issues/7640)
  type Last<T extends any[]> = T extends [...infer _, infer Z] ? Z : never;

  // 배열 infer로 분리하는거 어디서 봤더라....
  // -> 4주차 - [includes](https://github.com/type-challenges/type-challenges/blob/master/questions/898-easy-includes/README.ko.md)에서 사용했었음...!
  type Includes<T extends readonly any[], U> =
    T extends [infer First, ...infer Rest]
      ? Equal<First, U> extends true
        ? true
        : Includes<Rest, U>
      : false;
  ```
- 재영
  - try1
    ```tsx
    // 어거지다! 어거지로 안됐던듯
    type Last<T extends any[]> = T extends {length: infer L } ? T[`${L-1}`] : any
    // 안되네 ?.. 꿈이었나 ?..
    ```
  - try2
    ```tsx
    type Last<T extends any[]> = T extends [...infer F, infer R] ? R : any;
    ```
