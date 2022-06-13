### Parameters

- 연구
  ```tsx
  // try 1

  type MyParameters<T extends (...args: any[]) => any> = T extends (
    ...args: infer K
  ) => K
    ? K
    : never; // 물음표 앞에 K에 빨간 줄이 가는데 아래는 통과한다 뭐지 ? 에러는 "Cannot find name 'K'"

  /* _____________ Test Cases _____________ */
  import { Equal, Expect, ExpectFalse, NotEqual } from "@type-challenges/utils";

  const foo = (arg1: string, arg2: number): void => {};
  const bar = (arg1: boolean, arg2: { a: "A" }): void => {};
  const baz = (): void => {};

  type cases = [
    Expect<Equal<MyParameters<typeof foo>, [string, number]>>,
    Expect<Equal<MyParameters<typeof bar>, [boolean, { a: "A" }]>>,
    Expect<Equal<MyParameters<typeof baz>, []>>
  ];
  ```
  ```tsx
  //try 2

  type MyParameters<T extends (...args: any[]) => any> = T extends (
    ...args: infer K
  ) => infer R
    ? K
    : never; // 성공한다 하지만 R을 사용안했는데..?

  /* _____________ Test Cases _____________ */
  import { Equal, Expect, ExpectFalse, NotEqual } from "@type-challenges/utils";

  const foo = (arg1: string, arg2: number): void => {};
  const bar = (arg1: boolean, arg2: { a: "A" }): void => {};
  const baz = (): void => {};

  type cases = [
    Expect<Equal<MyParameters<typeof foo>, [string, number]>>,
    Expect<Equal<MyParameters<typeof bar>, [boolean, { a: "A" }]>>,
    Expect<Equal<MyParameters<typeof baz>, []>>
  ];
  ```
- 현석
  infer 몇번 다뤄봤다고 쉽다.
  ```tsx
  type MyParameters<T extends (...args: any[]) => any> = T extends (
    ...args: infer P
  ) => any
    ? P
    : never;
  ```
- 화랑
  - 1차 시도: 파라미터가 없을 때는 빈배열을 리턴해야 하니까...
  - `type MyParameters<T extends (...args: any[]) => any, U> = T extends () => any ? [] : U` → fail.
- 찬모
  ```tsx
  // Try1
  type MyParameters<T extends (...args: any[]) => any> = T extends Function
    ? [...args]
    : never;
  ```
  Try1
  arg에서 빨간줄이 뜬다.
  ![스크린샷 2022-03-02 오후 10.48.33.png](%E1%84%90%E1%85%A1%E1%84%8B%E1%85%B5%E1%86%B8%20%E1%84%8E%E1%85%A2%E1%86%AF%E1%84%85%E1%85%B5%E1%86%AB%E1%84%8C%E1%85%B5%204%E1%84%92%E1%85%AC%E1%84%8E%E1%85%A1%20b937f5a8fb1443ff87d086c1505aace7/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-03-02_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_10.48.33.png)
  → 특정 타입변수를 통해서 args가 무엇인지를 지정해줘야하는데 어떻게 지정하지??
  → `unresolved` ?? 현재는 타입을 잘모르는거니까 `infer` 키워드를 도입해야하는건가??
  → 굳이 타입을 제한할 필요가 있을까?? (앞쪽의 extends)
  ```tsx
  // Try2 ❌
  type MyParameters<T> = T extends (args: infer A) => any ? A : never;

  // Try3 ⭕️
  type MyParameters<T> = T extends (...args: infer A) => any ? A : never;
  ```
- 지선
- 재영
  - 배열로 반환하는 rest ...params
  - Try1
    ```tsx
    type MyParameters<T extends (...args: any[]) => any> = T extends (
      ...args: infer U
    ) => any
      ? U
      : any;
    ```
- 주운
  답안 먼저 확인🎃
  ```jsx
  type MyParameters<T extends (...args: any[]) => any> = (
    T extends (...args: infer Args) => any ? Args : never
  )
  ```
  ***
  으음...? 🧐
  `infer`는 어떤 상황에서 쓰이는건지, `never` 역시 어떤 상황에서 쓰일 수 있는걸까
  - `infer` 나중에 이런 게 들어온다 라고 추론하는 키워드
  - `never` 이런 게 나올리 없어!!! 이게 나오면 안돼!!! 라는 상황에서 쓰인다
