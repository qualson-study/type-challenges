## Trim

- 연구
  ### **Trim**
  Implement `Trim<T>` which takes an exact string type and returns a new string with the whitespace from both ends removed.
  For example
  ```tsx
  type trimed = Trim<"  Hello World  ">; // expected to be 'Hello World'
  ```
  ```tsx
  //try 1  mapping 해서 공백이면 빼주려고 했지만 실패... 공백을 어떻게 지워야 할까?

  type Trim<S extends string> = {
    [key in keyof S]: S[key] extends " " ? never : S[key];
  };

  type a = Trim<" str">;

  /* _____________ Test Cases _____________ */
  import { Equal, Expect } from "@type-challenges/utils";

  type cases = [
    Expect<Equal<Trim<"str">, "str">>,
    Expect<Equal<Trim<" str">, "str">>,
    Expect<Equal<Trim<"     str">, "str">>,
    Expect<Equal<Trim<"str   ">, "str">>,
    Expect<Equal<Trim<"     str     ">, "str">>,
    Expect<Equal<Trim<"   \n\t foo bar \t">, "foo bar">>
  ];
  ```
  ```tsx
  // try 2 이런식으로 지울 수가 있구나... !!!

  type Trim<S extends string> = S extends
    | `${" " | "\n" | "\t"}${infer R}`
    | `${infer R}${" " | "\n" | "\t"}`
    ? Trim<R>
    : S;

  /* _____________ Test Cases _____________ */
  import { Equal, Expect } from "@type-challenges/utils";

  type cases = [
    Expect<Equal<Trim<"str">, "str">>,
    Expect<Equal<Trim<" str">, "str">>,
    Expect<Equal<Trim<"     str">, "str">>,
    Expect<Equal<Trim<"str   ">, "str">>,
    Expect<Equal<Trim<"     str     ">, "str">>,
    Expect<Equal<Trim<"   \n\t foo bar \t">, "foo bar">>
  ];
  ```
- 현석
- 화랑
  얘만 풀 수 있었다고 한다...
  ```jsx
  type Trim<S extends string> = S extends `${' ' | '\n' | '\t'}${infer Rest}`
      ? Trim<Rest> :
  S extends `${infer Rest}${' ' | '\n' | '\t'}` ? Trim<Rest> : S
  ```
- 찬모
  ✅  아이디어
  문자열도 인덱스로 접근가능하니까 인덱스로 map을 돌리면서 빈칸인 경우 never를 주면 되지 않을까 🤔
  1. 첫번째 아이디어
     문자열도 index가 존재하니까 mapped type을 이용해서 돌리면서 공백이면 never, 아니면 글자를 넣어주는 방식으로 구현해보고자 했다.
  ```tsx
  // test1
  type Blank = " ";

  type Char<T extends string> = T extends Blank ? true : false;

  type T1 = Char<"str">; // false
  type T2 = Char<" ">; // true

  // → 빈칸을 타입으로 인지할 수 있나보다.
  ```
  그리고 삽질.....
  ```tsx
  // wrong!!
  type Blank = ' '

  // step1
  type Trim<S extends string> = { [K in keyof S] : S[K] extends Blank ? never : S[K] }

  //step2 : step1이 작동을 안해서 못함...😫
  아이디어 : 글자만을 모아서 붙인다 join() 의 느낌
  ```
  생각해보니 step1의 과정에서 너무 많은 중간 과정이 생략되어있는듯한 느낌을 받았다.
  - 문자열이 글자단위로 쪼개져서 타입을 체크할 수 있나?
  - `type T1 = keyof “ str”` 이렇게 하면 이게 0,1,2 number 뿐만아니라 string 객체로 인식되어 그 안의 내장메소드의 이름(key)까지고 다 나오게 됨.
    ![스크린샷 2022-03-31 오후 2.00.21.png](%E1%84%90%E1%85%A1%E1%84%8B%E1%85%B5%E1%86%B8%20%E1%84%8E%E1%85%A2%E1%86%AF%E1%84%85%E1%85%B5%E1%86%AB%E1%84%8C%E1%85%B5%208%E1%84%92%E1%85%AC%E1%84%8E%E1%85%A1%207f382f7820bf44dcbb8b2dfe72a4e6d1/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-03-31_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_2.00.21.png)
  🤔 이런식의 접근방법으로는 절대 해결할 수 없나?? 개념적으로 안되는건가?? 라는 생각과 함께 살며시 답을 보았다고 한다.
  ```tsx
  type Blank = " ";
  type TrimRight<S extends string> = S extends `${Blank}${infer R}`
    ? TrimRight<R>
    : S;
  type TrimLeft<S extends string> = S extends `${infer R}${Blank}`
    ? TrimLeft<R>
    : S;

  // test에서 \n, \t가 같이 있어서 Blank 타입에 포함시켜주면 test는 통과할 수 있다.
  ```
  타입에서도 ``를 사용할 수 있다는 것을 처음 알게 되었다.(얼핏 재영님께서 사용하시는걸 보긴했지만...)
- 지선
  ```tsx
  type trimed = TrimLeft<'  Hello World  '> // expected to be 'Hello World'

  -----------

  /////TrimLeft에서 오른쪽도 체크하도록
  //try 1 👌
  type Trim<S extends string> =
    S extends `${' ' | '\n' | '\t'}${infer W}`
    ? Trim<W>
    : S extends `${infer W}${' ' | '\n' | '\t'}`
      ? Trim<W>
      : S
  ```
- 재영
  - try1
    ```tsx
    // trimLeft에서 했던걸 써먹자
    type RemoveTarget = ` ` | `\n` | `\t`;
    type Trim<S extends string> = S extends
      | `${RemoveTarget}${infer U}`
      | `${infer U}${RemoveTarget}`
      ? Trim<U>
      : S;
    ```
