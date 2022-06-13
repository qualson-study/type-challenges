## Capitalize

- 연구
  ### **Capitalize**
  Implement `Capitalize<T>` which converts the first letter of a string to uppercase and leave the rest as-is.
  For example
  ```tsx
  type capitalized = Capitalize<"hello world">; // expected to be 'Hello world'
  ```
  ```tsx
  // try 1 Trim 했을 때 처럼 앞자리만 가져와서 하면 되겠지? 생각하고 하니 성공

  type MyCapitalize<S extends string> = S extends `${infer U}${infer K}`
    ? `${Uppercase<U>}${K}`
    : S;

  type a = MyCapitalize<"">;

  /* _____________ Test Cases _____________ */
  import { Equal, Expect } from "@type-challenges/utils";

  type cases = [
    Expect<Equal<MyCapitalize<"foobar">, "Foobar">>,
    Expect<Equal<MyCapitalize<"FOOBAR">, "FOOBAR">>,
    Expect<Equal<MyCapitalize<"foo bar">, "Foo bar">>,
    Expect<Equal<MyCapitalize<"">, "">>,
    Expect<Equal<MyCapitalize<"a">, "A">>,
    Expect<Equal<MyCapitalize<"b">, "B">>,
    Expect<Equal<MyCapitalize<"c">, "C">>,
    Expect<Equal<MyCapitalize<"d">, "D">>,
    Expect<Equal<MyCapitalize<"e">, "E">>,
    Expect<Equal<MyCapitalize<"f">, "F">>,
    Expect<Equal<MyCapitalize<"g">, "G">>,
    Expect<Equal<MyCapitalize<"h">, "H">>,
    Expect<Equal<MyCapitalize<"i">, "I">>,
    Expect<Equal<MyCapitalize<"j">, "J">>,
    Expect<Equal<MyCapitalize<"k">, "K">>,
    Expect<Equal<MyCapitalize<"l">, "L">>,
    Expect<Equal<MyCapitalize<"m">, "M">>,
    Expect<Equal<MyCapitalize<"n">, "N">>,
    Expect<Equal<MyCapitalize<"o">, "O">>,
    Expect<Equal<MyCapitalize<"p">, "P">>,
    Expect<Equal<MyCapitalize<"q">, "Q">>,
    Expect<Equal<MyCapitalize<"r">, "R">>,
    Expect<Equal<MyCapitalize<"s">, "S">>,
    Expect<Equal<MyCapitalize<"t">, "T">>,
    Expect<Equal<MyCapitalize<"u">, "U">>,
    Expect<Equal<MyCapitalize<"v">, "V">>,
    Expect<Equal<MyCapitalize<"w">, "W">>,
    Expect<Equal<MyCapitalize<"x">, "X">>,
    Expect<Equal<MyCapitalize<"y">, "Y">>,
    Expect<Equal<MyCapitalize<"z">, "Z">>
  ];
  ```
- 현석
  Think
  ```tsx
  type MyCapitalize<S extends string> = S extends `${infer A}${infer B}`
    ? A
    : never;
  type Test = MyCapitalize<"foobar">; // 'f'
  // 첫 글자를 뽑아낼 수 있다.

  // 그렇다면 이렇게 하면 될 것 같은데
  type MyCapitalize<S extends string> = S extends `${infer A}${infer B}`
    ? `${UpperCase<A>}${B}`
    : never;
  ```
  Answer. 이게 맞나?
  ```tsx
  type UpperCase<C extends string> = C extends "a"
    ? "A"
    : C extends "b"
    ? "B"
    : C extends "c"
    ? "C"
    : C extends "d"
    ? "D"
    : C extends "e"
    ? "E"
    : C extends "f"
    ? "F"
    : C extends "g"
    ? "G"
    : C extends "h"
    ? "H"
    : C extends "i"
    ? "I"
    : C extends "j"
    ? "J"
    : C extends "k"
    ? "K"
    : C extends "l"
    ? "L"
    : C extends "m"
    ? "M"
    : C extends "n"
    ? "N"
    : C extends "o"
    ? "O"
    : C extends "p"
    ? "P"
    : C extends "q"
    ? "Q"
    : C extends "r"
    ? "R"
    : C extends "s"
    ? "S"
    : C extends "t"
    ? "T"
    : C extends "u"
    ? "U"
    : C extends "v"
    ? "V"
    : C extends "w"
    ? "W"
    : C extends "x"
    ? "X"
    : C extends "y"
    ? "Y"
    : C extends "z"
    ? "Z"
    : C;
  type MyCapitalize<S extends string> = S extends `${infer A}${infer B}`
    ? `${UpperCase<A>}${B}`
    : S;
  ```
- 화랑
  ```jsx
  type MyCapitalize<S extends string> = Capitalize<S>

  너무 꼼수인가...?

  type MyCapitalize<S extends string> = S extends `${infer F}${infer Rest}` ? `${Uppercase<F>}${Rest}` : ''
  ```
- 찬모
  ```tsx
  // try1
  interface Base {
    [key: string]: string;
  }

  interface Alphabet extends Base {
    a: "A";
    b: "B";
    c: "C";
    d: "D";
    e: "E";
    f: "F";
    g: "G";
    h: "H";
    i: "I";
    j: "J";
    k: "K";
    l: "L";
    m: "M";
    n: "N";
    o: "O";
    p: "P";
    q: "Q";
    r: "R";
    s: "S";
    t: "T";
    u: "U";
    v: "V";
    w: "W";
    x: "X";
    y: "Y";
    z: "Z";
  }

  type MyCapitalize<S extends string> = S extends `${infer A}${infer B}`
    ? `${Alphabet[A]}${B}`
    : S;

  // case 1개가 에러!!
  type T1 = MyCapitalize<"FOOBAR">; // error : `${string}OOBAR`
  ```
  - `interface Base`가 구현한 이유
    interface Base 없이 처음에 구현한 풀이대로 하면, `Alphabet[A]` 에서 아래와 같은 에러가 발생한다.
    `Type 'Alphabet[A]' is not assignable to type 'string | number | bigint | boolean | null | undefined'.`
    interface Alphabet 속성에 대한 타입 정의가 안되어 있어서 그런듯. 그래서 어쩔수 없이 interface Base를 구현하여서 상속으로 해결하였다.
    → 다른 방법이 있을까??
  - 에러가 발생하는 이유 : 모든 경우가 `${infer A}${infer B}` 에 속하기 때문에 S로 가는 경우가 없다.
  ```tsx
  // try2
  type MyCapitalize<S extends string> = S extends `${infer A}${infer B}`
    ? A extends keyof Alphabet
      ? `${Alphabet[A]}${B}`
      : S
    : S;

  // 같은 에러
  type R = "F" extends keyof Alphabet ? true : false; // true 🤔 ??
  ```
  - 이렇게 해도 같은 에러가 발생
    → `keyof Alphabet` 으로 구분될 줄 알았는데, 구분이 안됨.
    → interface Base로 인해서 크게보면 ‘F’ 역시 [key:string] : string에 속하기 때문에 구분하지 못하는듯하다.
  - **그런데**,  
    → `extends keyof Alphabet`에 의해서 조건이 제한
    → `Alphabet[A]` 의 타입이 자동 추론 가능
    → interface Base 가 필요없게 된 상태로 진화(?)하였다.
    → 결국, 위의 풀이에서 interface Base를 삭제하면 풀이 완료!! 🙏
- 지선
  ```tsx
  type capitalized = Capitalize<'hello world'> // expected to be 'Hello world'

  /////////

  // 첫번째 글자만 대문자로 바꾸기...
  //.toUpperCase()가 type에서 쓸 수 있는가?
  //try 1
  type MyCapitalize<S extends string> = S extends `${infer F}${infer R}` ? `${F.toUpperCase()}${R}` : never
  //Cannot access 'F.toUpperCase' because 'F' is a type, but not a namespace. Did you mean to retrieve the type of the property 'toUpperCase' in 'F' with 'F["toUpperCase"]'?

  //try 2
  type MyCapitalize<S extends string> = S extends `${infer F}${infer R}` ? `${F["toUpperCase"]}${R}` : never
  //Type 'F["toUpperCase"]' is not assignable to type 'string | number | bigint | boolean | null | undefined'. Type '() => string' is not assignable to type 'string | number | bigint | boolean | null | undefined

  //[Uppercase<StringType>](https://www.typescriptlang.org/docs/handbook/2/template-literal-types.html#uppercasestringtype) 사용
  // try 3
  type MyCapitalize<S extends string> = S extends `${infer First}${infer Rest}` ? `${Uppercase<First>}${Rest}` : never
  //Expect<Equal<MyCapitalize<''>, ''>>, 통과 못함

  //try 👌
  type MyCapitalize<S extends string> = S extends `${infer F}${infer R}` ? `${Uppercase<F>}${R}` : S

  ```
- 재영
  - try1
    ```tsx
    // 이렇게 길게 쓸 수 밖에 없는 것인가 ...

    type LowerCaseUnions =
      | "a"
      | "b"
      | "c"
      | "d"
      | "e"
      | "f"
      | "g"
      | "h"
      | "i"
      | "j"
      | "k"
      | "l"
      | "m"
      | "n"
      | "o"
      | "p"
      | "q"
      | "r"
      | "s"
      | "t"
      | "u"
      | "v"
      | "w"
      | "x"
      | "y"
      | "z";

    type mapping = {
      a: "A";
      b: "B";
      c: "C";
      d: "D";
      e: "E";
      f: "F";
      g: "G";
      h: "H";
      i: "I";
      j: "J";
      k: "K";
      l: "L";
      m: "M";
      n: "N";
      o: "O";
      p: "P";
      q: "Q";
      r: "R";
      s: "S";
      t: "T";
      u: "U";
      v: "V";
      w: "W";
      x: "X";
      y: "Y";
      z: "Z";
    };

    type UpperCase<F extends string> = F extends LowerCaseUnions
      ? `${mapping[F]}`
      : `${F}`;
    type MyCapitalize<S extends string> = S extends `${infer F}${infer R}`
      ? `${UpperCase<F>}${R}`
      : "";

    // 솔루션 찾아보니 Uppercase라는 내장 타입이 있었네 ... 만들필요 없었다 ...
    // Intrinsic String Manipulation Types 4가지가 정의되어있음.
    // => https://www.typescriptlang.org/docs/handbook/2/template-literal-types.html#intrinsic-string-manipulation-types
    ```
