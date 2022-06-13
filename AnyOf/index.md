## AnyOf - 지선

- 연구
  ## **Type Challenge**
  [타입챌린지](https://github.com/type-challenges/type-challenges)
  ### **AnyOf**
  Implement Python liked `any` function in the type system. A type takes the Array and returns `true` if any element of the Array is true. If the Array is empty, return `false`.
  For example:
  ```tsx
  type Sample1 = AnyOf<[1, "", false, [], {}]>; // expected to be true.
  type Sample2 = AnyOf<[0, "", false, [], {}]>; // expected to be false.
  ```
  ```tsx
  // try 1 간단하게 해봤지만 객체, 배열이 문제 인 것 같다

  type AnyOf<T extends readonly any[]> = T[number] extends
    | []
    | 0
    | ""
    | false
    | {}
    ? false
    : true;
  ```
  ```tsx
  // try 2

  type AnyOf<T extends readonly any[]> = T[number] extends 0 | "" | false | []
    ? false
    : keyof T[number] extends string
    ? true
    : false;

  /* _____________ Test Cases _____________ */
  import type { Equal, Expect } from "@type-challenges/utils";

  type cases = [
    Expect<
      Equal<
        AnyOf<[1, "test", true, [1], { name: "test" }, { 1: "test" }]>,
        true
      >
    >,
    Expect<Equal<AnyOf<[1, "", false, [], {}]>, true>>,
    Expect<Equal<AnyOf<[0, "test", false, [], {}]>, true>>,
    Expect<Equal<AnyOf<[0, "", true, [], {}]>, true>>,
    Expect<Equal<AnyOf<[0, "", false, [1], {}]>, true>>,
    Expect<Equal<AnyOf<[0, "", false, [], { name: "test" }]>, true>>,
    Expect<Equal<AnyOf<[0, "", false, [], { 1: "test" }]>, true>>,
    Expect<
      Equal<AnyOf<[0, "", false, [], { name: "test" }, { 1: "test" }]>, true>
    >,
    Expect<Equal<AnyOf<[0, "", false, [], {}]>, false>>, /// {} 빼면 성공인데... 실패   keyof {} never 인데 왜...
    Expect<Equal<AnyOf<[]>, false>>
  ];
  ```
  ```tsx
  // try 3 결국 답확인 { [key: string]: never } 를 사용하면 됐구나..

  type AnyOf<T extends readonly any[]> = T[number] extends
    | 0
    | ""
    | false
    | []
    | { [key: string]: never }
    ? false
    : true;
  ```
- 지선
  ```tsx
  type Sample1 = AnyOf<[1, '', false, [], {}]> // expected to be true.
  type Sample2 = AnyOf<[0, '', false, [], {}]> // expected to be false.
  /////////////

  // try1
  type IsEmpty<T> = T extends 0 | '' | false | [] | {} ? true : false
  type AnyOf<T extends readonly any[]> =
    T extends [infer F, ...infer R] ?
     IsEmpty<F> extends true ? AnyOf<R>: true
    :IsEmpty<T>

  // 테스트 케이스 통과 못함
  //Expect<Equal<AnyOf<[0, '', false, [], {}]>, false>>,
  //Expect<Equal<AnyOf<[]>, false>>,

  //try2
  type AnyOf<T extends readonly any[]> =
  T extends [infer F, ...infer R]
    ? IsEmpty<R> extends true? IsEmpty<F> : AnyOf<R>
    : IsEmpty<T> extends true ? false: true
  // 테스트 케이스 통과 못함
  //Expect<Equal<AnyOf<[0, '', false, [], {}]>, false>>,
  -> R이 [{}]일 경우 AnyOf<[{}]>를 하게되면 F는 {}, R은 []가 나옴
  -> 따라서 IsEmpty<R>은 true이므로 IsEmpty<{}>는 true가 나와버림.

  //type test = IsEmpty<[0, '', false, [], {}]> -> true
  //type a = [0, '', false, [], {}] extends {} ? 1: 2 -> 2
  //type a = [0, '', false, [], {}] extends [] ? 1: 2 -> 1
  //큰틀에서 object 타입이니까.....

  //try3 👌
  type IsEmpty<T> = T extends 0 | '' | false | [] ? true : Equal<T, {}>

  type AnyOf<T extends readonly any[]> =
  T extends [infer F, ...infer R]
    ? IsEmpty<F> extends true? AnyOf<R> : true
    : IsEmpty<T> extends true ? false: true
  ```
- 재영
  - try1
    ```tsx
    // 배열의 유니온화 복습
    type Foo = [1, 2, 3];
    type Test = Foo[number]; //  순서가 이상하긴 하지만 어쨌든 3 | 1 | 2

    type Falsy = false | 0 | "" | null | undefined;
    type AnyOf<T extends readonly any[]> = T[number] extends Falsy
      ? false
      : true;

    type Test = AnyOf<[0, "", false, [], {}]>; // 얘가 true라서 실패
    ```
  - try2
    ```tsx
    // false | {} | "" | 0 | [1] << 요놈이
    // false | 0 | '' | null | undefined | [] | {} << 요놈을 extends하는게 true네...

    // fail
    type Falsy = false | 0 | "" | null | undefined | [] | {};
    type AnyOf<T extends readonly any[]> = T[number] extends Falsy
      ? false
      : true;

    // 정답봄
    type Falsy = false | 0 | "" | null | undefined | [] | Record<string, never>; // <--- {} 대신에 Record<string, never>로 표현하니까 풀림!!

    // 아하 ... {}랑 Record<string, never>랑 서로 동치다.
    type Test1 = Record<string, never> extends {} ? true : false; // true
    type Test2 = {} extends Record<string, never> ? true : false; // true
    ```
