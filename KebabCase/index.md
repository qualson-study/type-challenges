## KebabCase - 찬모, 연구

- 연구
  ## **Type Challenge**
  [타입챌린지](https://github.com/type-challenges/type-challenges)
  ### **KebabCase**
  ```
  FooBarBaz -> foo-bar-baz
  ```
  ```tsx
  // try 1 이런식으로 접근해보기로 함..

  type KebabCase<S> = S extends `${infer R}${infer U}`
    ? R extends Uppercase<R>
      ? `-${Lowercase<R>}${U}`
      : `-${R}${U}`
    : `-${R}${U}`;

  type b = KebabCase<"FooBar">;
  ```
  ```tsx
  // try 2 맨앞에만 뺴면 될 것 같은데...!!!
  type KebabCase<S> = S extends `${infer F}${infer R}`
    ? F extends Uppercase<F>
      ? `-${Lowercase<F>}${KebabCase<R>}`
      : `${F}${KebabCase<R>}`
    : S;

  type b = KebabCase<"FooBar">; // -foo-bar
  ```
  ```tsx
  // try 3 앞에를 날렸더니..

  type First<S> = S extends `${infer F}${infer R}`
    ? F extends Capitalize<F>
      ? `-${Uncapitalize<F>}${First<R>}`
      : `${F}${First<R>}`
    : S;

  type KebabCase<S> = First<S> extends `${infer F}${infer B}` ? B : never;

  type b = KebabCase<"foo-bar">; // oo--bar
  ```
  ```tsx
  // try 4 결국 답을 봄... 첫번째를 저렇게 할 수 있구나..!

  type KebabCase<
    S extends string,
    IsFirst extends boolean = true
  > = S extends `${infer First}${infer Rest}`
    ? First extends Lowercase<First>
      ? `${First}${KebabCase<Rest, false>}`
      : `${IsFirst extends true ? "" : "-"}${Lowercase<First>}${KebabCase<
          Rest,
          false
        >}`
    : S;
  ```
- 현석
- 화랑
- 찬모
  ```tsx
  // try1
  type KebabCase<S> = S extends `${infer A}${infer B}`
    ? A extends Uppercase<A>
      ? `${Lowercase<A>}${B}`
      : B extends Uppercase<B>
      ? `${A}-${Lowercase<B>}`
      : `${A}${B}`
    : S;

  // 어디선가 제대로 값이 안나온다...
  // 여기에 추가로 재귀를 돌려야할 것 같다.
  ```
- 지선
  ```tsx
  FooBarBaz -> foo-bar-baz

  // 대문자로 잘라서 대문자는 소문자로 만들고, 사이에 - 넣으면 되겠지?
  // -> 대문자 찾는 방법은...? 정규 표현식 체크 가능한가?
  //try1
  var newmsg = str.replace(/[a-z]/g, '');
  ~~var old = str.replace(/[A-Z]/g, '');~~

  // 답확인 -> 대소문자 체크를 F extends Uppercase<F>로 할 수 있다는걸 캐치
  //try 1
  type KebabCase<S> =
    S extends `${infer F}${infer R}`
    ? F extends Uppercase<F> ? `-${Lowercase<F>}${KebabCase<R>}`:`${F}${KebabCase<R>}`
    : S

  -> 첫번째 문자 앞에는 - 붙으면 안됨

  //try 2
  type KebabCase<S, isFirst extends boolean =true> =
    S extends `${infer F}${infer R}`
    ? isFirst extends true
      ? `${Lowercase<F>}${KebabCase<R, false>}`
      : F extends Uppercase<F>
        ? `-${Lowercase<F>}${KebabCase<R, false>}`
        : `${F}${KebabCase<R, false>}`
    : S

  ->  특수문자가 있는 케이스 통과 못함.
  //type Test = KebabCase<'foo_bar'> // -> "foo-_bar"
  //type Test = KebabCase<'foo_bar'> // -> "foo-_bar"
  //type Test = KebabCase<'Foo-Bar'> // -> "foo---bar"
  type Test = '-' extends Uppercase<'-'>? true: false // -> true

  // 특수문자에서 Uppercase 동일하게 나오므로 Uppercase -> Lowercase로 변경해서 -안붙도록 처리하면?

  // ok 👌
  type KebabCase<S, isFirst extends boolean =true> =
    S extends `${infer F}${infer R}`
    ? isFirst extends true
      ? `${Lowercase<F>}${KebabCase<R, false>}`
      : F extends Lowercase<F>
        ? `${F}${KebabCase<R, false>}`
        : `-${Lowercase<F>}${KebabCase<R, false>}`
    : S

  // 보기 어려움.... infer를 3개로 나누면?
  type KebabCase<S> =
    S extends `${infer A}${infer B}${infer R}`
    ? B extends Lowercase<B> ? `${Lowercase<A>}${KebabCase<`${B}${R}`>}`
      :`${Lowercase<A>}-${KebabCase<`${B}${R}`>}`
    :S
  //Equal<KebabCase<'ABC'>, 'a-b-c'> 통과 못함. 결과 : a-b-C

  // ok 👌
  type KebabCase<S extends string> =
    S extends `${infer A}${infer B}${infer R}`
    ? B extends Lowercase<B> ? `${Lowercase<A>}${KebabCase<`${B}${R}`>}`
      :`${Lowercase<A>}-${KebabCase<`${B}${R}`>}`
    :Lowercase<S>
  ```
- 재영
  - 정답보기
    ```tsx
    type KebabCase<S extends string> = S extends `${infer L}${infer R}`
      ? R extends Uncapitalize<R>
        ? `${Uncapitalize<L>}${KebabCase<R>}`
        : `${Uncapitalize<L>}-${KebabCase<R>}`
      : S;
    ```
  - 복습
    ```tsx
    // string을 infer하기
    type Test = "" extends `${infer F}${infer R}` ? true : false; // false
    type Test = "a" extends `${infer F}${infer R}` ? true : false; // true. 최소 string이 1음절 이상이어야 infer를 2번 쓰는게 의미있다.

    type Test = "a" extends `${infer F}${infer R}` ? R : false; // ''로 추론.
    ```
