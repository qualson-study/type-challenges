## Replace

- 연구
  # **Replace**
  Implement `Replace<S, From, To>` which replace the string `From` with `To` once in the given string `S`
  For example
  ```tsx
  type replaced = Replace<"types are fun!", "fun", "awesome">; // expected to be 'types are awesome!'
  ```
  ```tsx
  // try 1 모르겠어서 정답확인.....
  type Replace<
    S extends string,
    From extends string,
    To extends string
  > = From extends ""
    ? S
    : S extends `${infer left}${From}${infer right}`
    ? `${left}${To}${right}`
    : S;

  /* _____________ Test Cases _____________ */
  import { Equal, Expect } from "@type-challenges/utils";

  type cases = [
    Expect<Equal<Replace<"foobar", "bar", "foo">, "foofoo">>,
    Expect<Equal<Replace<"foobarbar", "bar", "foo">, "foofoobar">>,
    Expect<Equal<Replace<"foobarbar", "", "foo">, "foobarbar">>,
    Expect<Equal<Replace<"foobarbar", "bra", "foo">, "foobarbar">>,
    Expect<Equal<Replace<"", "", "">, "">>
  ];
  ```
- 현석
  Try 1.
  ```tsx
  type Replace<S extends string, From extends string, To extends string> = S extends `${infer A}${From}${infer B}` ? `${A}${To}${B}` : S

  Wrong!
  Replace<'foobarbar', '', 'foo'> // === f + foo + oobarbar
  ```
  Try 2. Success: From이 빈 스트링일 때 예외 처리
  ```tsx
  type Replace<
    S extends string,
    From extends string,
    To extends string
  > = From extends ""
    ? S
    : S extends `${infer A}${From}${infer B}`
    ? `${A}${To}${B}`
    : S;
  ```
- 화랑
- 찬모
  아이디어1
  → 한글자씩 비교하면서 같은 글자가 나오면 바꿔준다 라는 생각으로 시작
  ![스크린샷 2022-04-06 오전 7.11.37.png](%E1%84%90%E1%85%A1%E1%84%8B%E1%85%B5%E1%86%B8%20%E1%84%8E%E1%85%A2%E1%86%AF%E1%84%85%E1%85%B5%E1%86%AB%E1%84%8C%E1%85%B5%209%E1%84%92%E1%85%AC%E1%84%8E%E1%85%A1%201fab2b1326be46bfab37f59c297e276d/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-04-06_%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB_7.11.37.png)
  ```jsx
  // try1
  type CharByChar<A extends string, B extends string> = B extends `${infer C}${infer D}` ? `${A}${C}` : `${A}${B}`

  type Replace<S extends string, From extends string, To extends string>
    = S extends `${infer A}${infer B}`
        ? A extends From ? `${To}${B}` : Replace<CharByChar<A, B>, From, To>
        : S

  // example
  type T1 = Replace<'abc', 'a', 'zz'> // zzbc
  type T2 = Replace<'abc', 'ab', 'zz'>
  // error : Type instantiation is excessively deep and possibly infinite
  // → 무한루프

  +
  추가적으로 type Checker를 만들어서 From과 다시 비교해주는 부분을 추가했지만 결국 돌돌~도돌이표!!
  ```
  → 문제점 : 위와 같이하면 처음부터 이어지는 글자만 비교할 수 있게 된다. 문제에서는 그냥 같은 글자가 나오면 바꿔주면 된다는 것!!
  약간 힌트를 얻어서 다시 도전
  ```jsx
  // try2
  type Replace<S extends string, From extends string, To extends string>
    = S extends `${infer A}${From}${infer B}`
        ? `${A}${To}${B}` : S extends `${From}${infer A}${infer B}`
          ? `${To}${A}${B}` : S extends `${infer A}${infer B}${From}`
            ? `${A}${B}${To}` : S

  type T1 = Replace<'foobarbar', '', 'foo'> // ffoooobarbar, expected foobarbar
  ```
  Q1. 왜 type T1은 빈문자열인데 저렇게 바뀌지??
  → 이 부분을 예외처리하여서 푸는 풀이가 있었음
  ```jsx
  type Replace<S extends string, From extends string, To extends string>
  	= From extends '' ? S
  		: S extends `${infer Pre}${From}${infer Post}`
  		? `${Pre}${To}${Post}` : S
  ```
  Q2. 같은 부분을 찾기 위해선 탐색과정이 필요한데, 위 타입에서 어느 부분이 탐색에 대한 부분을 이야기하는 것인지?
  → infer 라는 키워드로 인해 그러한 과정이 암묵적으로, 내부적으로 처리된다고 생각하면 맞을까??
- 지선
  ```tsx
  type replaced = Replace<"types are fun!", "fun", "awesome">; // expected to be 'types are awesome!'

  ////////////

  ("t / ypes are fun!");
  ("ty / pes are fun!");
  ("typ / es are fun!");

  type Replace<
    S extends string,
    From extends string,
    To extends string
  > = S extends `${infer F}${infer R}`
    ? F extends From
      ? `${To}${R}`
      : R extends From
      ? `${F}${To}`
      : S
    : never;

  // F가 한글자만 되지 않나...? 단어로 찾는 방법은?

  //try
  type Replace<
    S extends string,
    From extends string,
    To extends string
  > = S extends `${infer F}${From}${infer R}` ? `${F}${To}${R}` : S;
  // Expect<Equal<Replace<'foobarbar', '', 'foo'>, 'foobarbar'>>, 해당 케이스 통과 못함
  //type A = Replace<'foobarbar', '', 'foo'>
  // type A = "ffoooobarbar"

  // 일단 모든 테스트 케이스 통과
  //try 👌
  type Replace<
    S extends string,
    From extends string,
    To extends string
  > = From extends ""
    ? S
    : S extends `${infer F}${From}${infer R}`
    ? `${F}${To}${R}`
    : S;
  ```
- 재영
  - try1
    ```tsx
    type SliceString<T extends string> = T extends `${infer F}${infer R}`
      ? R
      : "";
    type IgnoreEmptyString<From extends string, S, R> = From extends "" ? S : R;
    type Replace<
      S extends string,
      From extends string,
      To extends string
    > = IgnoreEmptyString<From, S, ReplaceMiddle<S, From, To>>;
    type ReplaceMiddle<
      S extends string,
      From extends string,
      To extends string
    > = S extends `${infer F}${From}${infer R}`
      ? `${F}${To}${R}`
      : ReplaceMiddle<SliceString<S>, From, To>;

    type Test2 = Replace<"", "", "">; // Type instantiation is excessively deep and possibly infinite.
    ```
  - try2
    ```tsx
    // 1
    type IgnoreEmptyString<
      From extends string,
      S extends string,
      R extends string
    > = From extends "" ? S : R;
    type Replace<
      S extends string,
      From extends string,
      To extends string
    > = IgnoreEmptyString<From, S, ReplaceMiddle<S, From, To>>;

    // 2
    type Replace<
      S extends string,
      From extends string,
      To extends string
    > = From extends "" ? S : ReplaceMiddle<S, From, To>;

    // 2는 그냥 1을 풀어쓴건데 왜 결과가 다르지 ??
    ```
  - try3
    ```tsx
    type SliceString<T extends string> = T extends `${infer F}${infer R}`
      ? R
      : "";
    type Replace<
      S extends string,
      From extends string,
      To extends string
    > = From extends "" ? S : ReplaceMiddle<S, From, To>;
    type ReplaceMiddle<
      S extends string,
      From extends string,
      To extends string
    > = S extends `${infer F}${From}${infer R}`
      ? `${F}${To}${R}`
      : ReplaceMiddle<SliceString<S>, From, To>;

    // from에 해당하는 문자열이 없을 때 "" 상태가 되고 이게 계속 빙빙 도나보다
    type Test = Replace<"foobarbar", "bra", "foo">; // Type instantiation is excessively deep and possibly infinite.

    // 캐시(?)로 해결! 😊
    type SliceString<T extends string> = T extends `${infer F}${infer R}`
      ? R
      : "";
    type Replace<
      S extends string,
      From extends string,
      To extends string
    > = From extends "" ? S : ReplaceMiddle<S, From, To, S>;
    type ReplaceMiddle<
      S extends string,
      From extends string,
      To extends string,
      C
    > = S extends ""
      ? C
      : S extends `${infer F}${From}${infer R}`
      ? `${F}${To}${R}`
      : ReplaceMiddle<SliceString<S>, From, To, C>;
    ```
  - 답을 보고나니 한참을 빙빙 돌아온 기분
    ```tsx
    type Replace<
      S extends string,
      From extends string,
      To extends string
    > = From extends ""
      ? S
      : S extends `${infer Left}${From}${infer Right}` // 여기서 infer Left, infer Right는 빈 문자열도 포함이다!! 🔥🔥
      ? `${Left}${To}${Right}`
      : S;
    ```
