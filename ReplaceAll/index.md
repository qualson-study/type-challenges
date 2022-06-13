## ReplaceAll

- 연구
  # **ReplaceAll**
  Implement `ReplaceAll<S, From, To>` which replace the all the substring `From` with `To` in the given string `S`
  For example
  ```tsx
  type replaced = ReplaceAll<"t y p e s", " ", "">; // expected to be 'types'
  ```
  ```tsx
  // try 1  재귀를 돌렸더니 2가지 케이스가 안된다 한 번 본것은 빼면 되겠지?

  type ReplaceAll<S extends string, From extends string, To extends string> = From extends '' ? S : S extends `${infer left}${From}${infer right}`?  ReplaceAll<`${left}${To}${right}`, From, To>: S;

  import { Equal, Expect } from '@type-challenges/utils'

  type cases = [
    Expect<Equal<ReplaceAll<'foobar', 'bar', 'foo'>, 'foofoo'>>,
    Expect<Equal<ReplaceAll<'foobar', 'bag', 'foo'>, 'foobar'>>,
    Expect<Equal<ReplaceAll<'foobarbar', 'bar', 'foo'>, 'foofoofoo'>>,
    Expect<Equal<ReplaceAll<'t y p e s', ' ', ''>, 'types'>>,
    Expect<Equal<ReplaceAll<'foobarbar', '', 'foo'>, 'foobarbar'>>,
    Expect<Equal<ReplaceAll<'barfoo', 'bar', 'foo'>, 'foofoo'>>,
    Expect<Equal<ReplaceAll<'foobarfoobar', 'ob', 'b'>, 'fobarfobar'>>, // 실패
    Expect<Equal<ReplaceAll<'foboorfoboar', 'bo', 'b'>, 'foborfobar'>>, // 실패
    Expect<Equal<ReplaceAll<'', '', ''>, ''>>,
  ]

  ReplaceAll<'rrrrrrrrrrr', 'rr', 'r'>
  => 'r' | 'rrrrrr'

  ReplaceAll<'foobarfoobar', 'ob', 'b'>
  => Replace<'fobarfoobar', 'ob', 'b'>
  => Replace<'fbarfoobar', 'ob', 'b'>
  => Replace<'fbarfobar', 'ob', 'b'>
  => Replace<'fbarfbar', 'ob', 'b'>

  ```
  ```tsx
  // try 2 From만 To로 변경해주고 right만 재귀를 돌려줬더니 성공 !

  type ReplaceAll<
    S extends string,
    From extends string,
    To extends string
  > = From extends ""
    ? S
    : S extends `${infer left}${From}${infer right}`
    ? `${left}${To}${ReplaceAll<right, From, To>}`
    : S;

  import { Equal, Expect } from "@type-challenges/utils";

  type cases = [
    Expect<Equal<ReplaceAll<"foobar", "bar", "foo">, "foofoo">>,
    Expect<Equal<ReplaceAll<"foobar", "bag", "foo">, "foobar">>,
    Expect<Equal<ReplaceAll<"foobarbar", "bar", "foo">, "foofoofoo">>,
    Expect<Equal<ReplaceAll<"t y p e s", " ", "">, "types">>,
    Expect<Equal<ReplaceAll<"foobarbar", "", "foo">, "foobarbar">>,
    Expect<Equal<ReplaceAll<"barfoo", "bar", "foo">, "foofoo">>,
    Expect<Equal<ReplaceAll<"foobarfoobar", "ob", "b">, "fobarfobar">>,
    Expect<Equal<ReplaceAll<"foboorfoboar", "bo", "b">, "foborfobar">>,
    Expect<Equal<ReplaceAll<"", "", "">, "">>
  ];
  ```
- 현석
  Try 1. Replace를 재귀로!
  ```tsx
  type ReplaceAll<S extends string, From extends string, To extends string> = Equal<Replace<S, From, To>, S> extends true ? S : ReplaceAll<Replace<S, From, To>, From, To>

  Oops
  ReplaceAll<'foobarfoobar', 'ob', 'b'>
  => Replace<'fobarfoobar', 'ob', 'b'>
  => Replace<'fbarfoobar', 'ob', 'b'>
  => Replace<'fbarfobar', 'ob', 'b'>
  => Replace<'fbarfbar', 'ob', 'b'>
  ```
  Try 2. 한번에 처리하려면..... Success
  ```tsx
  type ReplaceAll<
    S extends string,
    From extends string,
    To extends string
  > = From extends ""
    ? S
    : S extends `${infer A}${From}${infer B}`
    ? `${A}${To}${ReplaceAll<B, From, To>}`
    : S;
  ```
- 화랑
- 찬모
  위 문제에서 힌트를 얻음
  ```jsx
  type ReplaceAll<S extends string, From extends string, To extends string>
    = From extends '' ? S
  		: S extends `${infer Pre}${From}${infer Post}`
  			? ReplaceAll<`${Pre}${To}${Post}`, From, To> : S
  ```
  테스트 케이스에서 안되는 부분은 순서가 있어야하는 부분
  → 한번 탐색하고 지나간 부분은 바뀌면 안된다는 것
  → 이 부분은 약간 순억지같은 느낌이 들었음, 위와 같은 작동이 ReplaceAll이라고 정의하는 타입의 작동방식과 맞는거같은데... 음.... 🤔
- 지선
  ```tsx
  type replaced = ReplaceAll<"t y p e s", " ", "">; // expected to be 'types'

  ////////////

  // replace 반복 호출하도록 하면 되겠지?
  // try 1
  type ReplaceAll<
    S extends string,
    From extends string,
    To extends string
  > = From extends ""
    ? S
    : S extends `${infer F}${From}${infer R}`
    ? `${F}${To}${R}`
    : ReplaceAll<R, From, To>;
  //type R = /*unresolved*/ any
  //Cannot find name 'R'.

  // try
  type ReplaceAll<
    S extends string,
    From extends string,
    To extends string
  > = From extends ""
    ? S
    : S extends `${infer F}${From}${infer R}`
    ? From extends R
      ? `${F}${To}${ReplaceAll<R, From, To>}`
      : `${F}${To}${R}`
    : S;

  //Expect<Equal<ReplaceAll<'t y p e s', ' ', ''>, 'types'>>,  // = "ty p e s"
  //Expect<Equal<ReplaceAll<'foobarfoobar', 'ob', 'b'>, 'fobarfobar'>>, // = "fobarfoobar"

  // try 👌
  type ReplaceAll<
    S extends string,
    From extends string,
    To extends string
  > = From extends ""
    ? S
    : S extends `${infer F}${From}${infer R}`
    ? `${F}${To}${ReplaceAll<R, From, To>}`
    : S;
  ```
- 재영
  - try1
    ```tsx
    type Replace<
      S extends string,
      From extends string,
      To extends string
    > = S extends `${infer F}${From}${infer R}` ? `${F}${To}${R}` : S;

    type ReplaceAll<
      S extends string,
      From extends string,
      To extends string
    > = From extends ""
      ? S
      : S extends `${infer F}${From}${infer R}`
      ? ReplaceAll<`${F}${To}${R}`, From, To>
      : S;

    type Test = ReplaceAll<"foobarfoobar", "ob", "b">;

    // 머루겠다...
    ```
