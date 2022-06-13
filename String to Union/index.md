## String to Union

- 연구
- 현석
- 화랑
- 찬모
  ```tsx
  // try1 😎
  type StringToUnion<T extends string> = T extends `${infer A}${infer B}`
    ? A | StringToUnion<B>
    : never;
  ```
  상상을 했더니 풀리는 기적??
  이거슨 **타입스크립트 메타 월드** 인가 🤯
- 지선
  ```tsx
  type Test = "123";
  type Result = StringToUnion<Test>;
  // expected to be "1" | "2" | "3"

  ///////

  // length of string 비슷하게 사용하면 되지 않을까?
  type StringToUnion<
    S extends string,
    Arr extends string[] = []
  > = S extends `${infer A}${infer B}`
    ? StringToUnion<B, [...Arr, A]>
    : Arr[number];
  ```
- 재영
  - try1: 재귀는 안쓰고 싶은데 떠오르는게 재귀밖에 없다 ...
    ```tsx
    // C 제네릭은 Cache 비스무리하게 추론된 타입을 저장하는 용도로 사용. []로 초기화해둔다
    // C에다가 'hello'의 'h', 'e' ... 를 순서대로 넣어준다.
    // ["h", "e", "l", "l", "o"] 라고 추론된걸 Union으로 바꿔주는건 간단하다. C[number]로 접근하면된다.

    type StringToUnion<
      T extends string,
      C extends string[] = []
    > = T extends `${infer F}${infer R}`
      ? StringToUnion<R, [...C, F]>
      : C[number];
    ```
