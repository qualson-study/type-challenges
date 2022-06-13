### \***\*Tuple to Union\*\***

- 연구
  ### **Tuple To Union**
  Implement a generic `TupleToUnion<T>` which covers the values of a tuple to its values union.
  For example
  ```tsx
  type Arr = ["1", "2", "3"];

  type Test = TupleToUnion<Arr>; // expected to be '1' | '2' | '3'
  ```
  ```tsx
  // try 1

  type TupleToUnion<T extends unknown[]> = T[number];

  /* _____________ Test Cases _____________ */
  import { Equal, Expect } from "@type-challenges/utils";

  type cases = [
    Expect<Equal<TupleToUnion<[123, "456", true]>, 123 | "456" | true>>,
    Expect<Equal<TupleToUnion<[123]>, 123>>
  ];

  /* _____________ Further Steps _____________ */
  /*
    > Share your solutions: https://tsch.js.org/10/answer
    > View solutions: https://tsch.js.org/10/solutions
    > More Challenges: https://tsch.js.org
  */
  ```
- 현석
  저번에 했던 것에서 아이디어를 얻어
  ```tsx
  type TupleToUnion<T extends any[]> = T[number];
  ```
- 화랑
- 찬모
  ```tsx
  // Try1 😡
  type TupleToUnion<T> = T[number];
  ```
  아래와 같은 하나의 에러가 생긴다.
  ![스크린샷 2022-03-13 오후 2.46.08.png](%E1%84%90%E1%85%A1%E1%84%8B%E1%85%B5%E1%86%B8%20%E1%84%8E%E1%85%A2%E1%86%AF%E1%84%85%E1%85%B5%E1%86%AB%E1%84%8C%E1%85%B5%206%E1%84%92%E1%85%AC%E1%84%8E%E1%85%A1%20f79080f9ee764d97ab5a161db7a074f5/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-03-13_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_2.46.08.png)
  > 타입 number는 타입 T의 인덱스 타입으로 사용될 수 없다. 라는 의미
  곰곰히 생각해보니, T가 항상 배열이라고 어디에서도 말해주지 않았다. 그렇기 때문에 T타입이 배열임을 제한해줘야했다. (이제 좀 **생각**이라는건 하는듯...)
  ```tsx
  // Try2 😎
  type TupleToUnion<T extends any[]> = T[number];
  ```
- 지선
  ```tsx
  // object를 tuple로 변환
  type TupleToObject<T extends readonly string[]> = {
  	[ key in T[number] ] : key
  }

  //try 1 👌
  type TupleToUnion<T extends any[]> = T[number]
  // 테스트 케이스 성공. 하지만 union 순서는 변경된다.

  type A = TupleToUnion<[123, '456', true, false]>
  // -> boolean | 123 | '456'

  type A = TupleToUnion<[123, '456', true, '12',222]>
  // -> true | 123 | '456' | '12' | 222

  TupleToUnion<[[2,3,4],123, '456', true, '12', [1,2,3]]>
  // -> true | [2,3,4] | 123 | '456' | '12' | [1,2,3]
  // boolean만 앞으로 이동되나?

  ```
- 재영
  - 이게 왜 medium이지?
    ```tsx
    type TupleToUnion<T extends unknown[]> = T[number];
    ```

??? 여기에서는 any / unknown 동작 차이가 없다?
