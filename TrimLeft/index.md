## TrimLeft

- 연구
  ### **Trim Left**
  Implement `TrimLeft<T>` which takes an exact string type and returns a new string with the whitespace beginning removed.
  For example
  ```tsx
  type trimed = TrimLeft<"  Hello World  ">; // expected to be 'Hello World  '
  ```
  ```tsx
  //try 1
  type TrimLeft<S extends string> = S extends `${"\n" | "\t" | " "}${infer K}`
    ? TrimLeft<K>
    : S;

  /* _____________ Test Cases _____________ */
  import { Equal, Expect } from "@type-challenges/utils";

  type cases = [
    Expect<Equal<TrimLeft<"str">, "str">>,
    Expect<Equal<TrimLeft<" str">, "str">>,
    Expect<Equal<TrimLeft<"     str">, "str">>,
    Expect<Equal<TrimLeft<"     str     ">, "str     ">>,
    Expect<Equal<TrimLeft<"   \n\t foo bar ">, "foo bar ">>,
    Expect<Equal<TrimLeft<"">, "">>,
    Expect<Equal<TrimLeft<" \n\t">, "">>
  ];
  ```
- 현석
- 화랑
  왼 쪽에 있는 공백만 지워라... string에서 whitespace거나 \n\t 라면 삭제
  string은 순회를 할 수 없으니... 다른 방법이 필요하다
  ```tsx
  type TrimLeft<S extends string> = S extends `${
    | " "
    | "\n"
    | "\t"}${infer Rest}`
    ? TrimLeft<Rest>
    : S;
  ```
- 찬모
  Trim을 먼저 풀어서 Trim을 풀면서 알게 되었습니다.(풀이역시...)
- 지선
  ```tsx
  type trimed = TrimLeft<'  Hello World  '> // expected to be 'Hello World  '

  -----------

  //재귀 사용?

  //try1 ... 대충 이런 느낌?
  type TrimLeft<S extends string> =  S[0] extends ' '| '\n' | '\t' ? TrimLeft<S.slice(1)>: S
  // S[0] 가져오지 못함. slice 사용 못함.

  //answer
  type TrimLeft<S extends string> =  S extends `${' ' | '\n'|'\t'}${infer W}` ? TrimLeft<W> : S
  //`${}` 요거 사용할 줄은 상상도 못함.... ㄴㅇㄱ
  ```
- 재영
  - try1
    ```tsx
    // string의 문자 하나하나에 어떻게 접근할 수 있을까?

    type IndexingString<S extends string> = S[number];
    // 혹시 'w' | 'o' | 'w' 가 나오지 않을까 했지만 그냥 string이 나오네 ...
    type Test = IndexingString<"wow">;
    ```
  - try2
    ```tsx
    // typescript split string literal type으로 구글링하다가 이런걸 발견
    // 비슷하게 따라하면 되겠는데 ?..
    type ExtractSemver<SemverString extends string> =
        SemverString extends `${infer Major}.${infer Minor}.${infer Patch}` ?
       { major: Major, minor: Minor, patch: Patch } : { error: "Cannot parse semver string" }

    // 예전에 재귀적으로 Promise를 벗겨준 것 처럼, 비슷하게 해보자
    type Space = ` `
    type TrimLeft<S extends string> = S extends `${Space}${infer U}` ? TrimLeft<U> : S

    // 여기서 에러 발생
    TrimLeft<'   \n\t foo bar '>
    ```
  - try3
    ```tsx
    // 완성!! 😃
    type RemoveTarget = ` ` | `\n` | `\t`;
    type TrimLeft<S extends string> = S extends `${RemoveTarget}${infer U}`
      ? TrimLeft<U>
      : S;
    ```
