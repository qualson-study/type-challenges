### If

[문제](https://www.typescriptlang.org/play?#code/PQKgUABBBMBsAcEC0ECSAzSyk91gRgJ4QAKAhgG4CmANhAOI0CuAzgBYDWA9hRABQABAA5l2TAC4cAlBADEVUcVkSAljRZYss7RACKTKi3EquAO01RUAWyE0qVqqfEQyEVeogADDJ4gB3NhUAYzYXIKCqIXEWCCCzABMVYzMvAGFPABoXCHEAJwk2YlyqcSZc0xzCISovABVMl1N47PQydSKSsorxKprPADFPADo03xUYqgAPaqDxKmbxLgh8GqoktipcrzyDXy4tz1b1Kl8AtT76xuaB3yCyCpXG4h7qoYsIfv2IKbIbOwAud6eYHRLAvGoAQQgAF40OgADw7KhZADkZBRqPwKIAfFBgMBvtMqLN5jklo80Siwb0IAAhGFw+FHFjIiCUzE4iD4wkzOYLck1FFYrDAzzvXEANRUVD8EBS9CSAAkmPh-hA2OJxEIWP98dEQkMAFYsIb7ADmwDg8DAIGAYHtoAgAH0Xa63a6IABNLhlCCpLjxGqKzY1d1hl0QW328GM1JZWpZfq42H3Qj2sCO8NhiC1QzOVKiQzOrNuyN2lQ2fbOADeEAAogBHJhtLJ1omzCAAXwg6FyXCsbIE4KQITadlMZsMwHcLCpYBjdxZMVhAG0sG3efDG82aPCMIj8qz2WysdjUejsWf1+3xFum209wjmUf0VloGeYJeMmAALoOglDiwSBTLywG5H2uTzjSmwQQy+6mEwNA0OeGInji6YZiAxYlhG-RlOIGxbAAynM2rYThZaYVguJEWwZDFBAhA+lsLBcMwySmDq6qatqurAPqbBGia5qWggwD3CwfibNREBSjKECsexJicWqGpajqeosAaxqmrkFpWsAimqGYGhQLiACy+w1KkdFIY4k5cWpvGadpwl6TadpgEAA)

- 연구
  ### **If**
  Implement a utils `If` which accepts condition `C`, a truthy return type `T`, and a falsy return type `F`. `C` is expected to be either `true` or `false` while `T` and `F` can be any type.
  For example:
  ```tsx
  type A = If<true, "a", "b">; // expected to be 'a'
  type B = If<false, "a", "b">; // expected to be 'b'
  ```
  ```tsx
  // try 1
  // C 를 불리언 타입으로 정해주면 true or false 밖에 오지 못하기 때문에 가능
  type If<C extends boolean, T, F> = C extends true ? T : F;

  /* _____________ Test Cases _____________ */
  import { Equal, Expect } from "@type-challenges/utils";

  type cases = [
    Expect<Equal<If<true, "a", "b">, "a">>,
    Expect<Equal<If<false, "a", 2>, 2>>
  ];

  // @ts-expect-error
  type error = If<null, "a", "b">;
  ```
- 현석
  한번에 끝냈다!
  ```tsx
  type If<C extends boolean, T, F> = C extends true ? T : F;
  ```
- 화랑
  처음으로 고민 없이 푼 듯.
  ```powershell
  type If<C extends Boolean, T, F> = C extends true ? T : F
  ```
- 찬모
  도전~🚀
  ```tsx
  // Try1
  type If<C, T, F> = C extends boolean ? T : F; // wrong

  // Try2
  type If<C, T, F> = C extends true ? T : F; // Unused '@ts-expect-error' directive.

  // Try3
  type If<C extends boolean, T, F> = C extends true ? T : F; // 🎉
  ```
  처음에 고민을 많이 하였다. 문제가 값에 대한 이야기 인데 값과 타입에 대해서 적절하게 표현할 수 있는 방법이 무엇이 있을까에 대해서 고민하였다.
  이 문제를 통해서 지난번에 이야기한 것처럼 제네릭 안에서 extends는 타입을 제한하는 역할을 하고 타입을 정의할 때의 extends는 할당한다는 느낌으로 사용한다 라는 의미를 조금은 이해한듯하다.
- 지선
  ```tsx
  조건 C, 참일 때 반환하는 타입 T,
  거짓일 때 반환하는 타입 F를 받는 타입 If를 구현하세요.
  C는 true 또는 false이고, T와 F는 아무 타입입니다.

  type A = If<true, 'a', 'b'>  // expected to be 'a'
  type B = If<false, 'a', 'b'> // expected to be 'b'

  // ---------------

  1. type If<C, T, F> = C ? T : F
  // 조건 C 연산을 어떻게 할것인가?
  // 조건에 대한 연산 결과는 boolean이니 true로 extends?
  2. type If<C, T, F> = C extends true ? T : F
  // -> null 처리 안됨.

  [🙆‍♀️]
  3. type If<C extends boolean, T, F> = C extends true ? T : F;
  C true, false
  'a' === 2
  ```
- 재영
  - 요구사항

    - type If<C, T, F>에서 C가 참이면 T를 반환하고 거짓이면 F를 반환하게하는 타입을 만들어보기
    - C는 true | false만 올 수 있음.
    - T, F can be any type

  - 아이디어

    - 그냥 extends랑 conditional type 쓰면될 것 같다?

  - Try
  ```tsx
  // perfect -> true | false는 그냥 boolean으로 쓰면되는디...
  type If<C extends true | false, T, F> = C extends true ? T : F;
  ```
