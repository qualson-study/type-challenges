- \***\*Length of String\*\***
  - 연구
    ### **Length of String**
    Compute the length of a string literal, which behaves like `String#length`.
    ```tsx
    //try 1 간단히 시도해봤지만 number 라고 뜨는구나..
    type LengthOfString<S extends string> = S["length"];
    ```
    ```tsx
    // try 2 Readonly 로 하면 될까 했지만 실패..!

    type LengthOfString<S extends string> = Readonly<S>["length"];
    ```
    ```tsx
    // try 3 스트링을 배열로 변경하면 length가 잘 잡히는 지 테스트 잘잡히는 것 같다 이제 어떻게 전체를 배열로 만들까?

    type LengthOfString<S extends string> = S extends ""
      ? 0
      : S extends `${infer K}${infer Y}`
      ? [K, Y]["length"]
      : never;

    type a = LengthOfString<"hi">; // 2
    ```
    ```tsx
    // try 4 결국 답확인... 인수를 하나 더받아야 했구나...!

    type LengthOfString<
      S extends string,
      len extends string[] = []
    > = S extends `${infer F}${infer R}`
      ? LengthOfString<R, [F, ...len]>
      : len["length"];

    import type { Equal, Expect } from "@type-challenges/utils";

    type cases = [
      Expect<Equal<LengthOfString<"">, 0>>,
      Expect<Equal<LengthOfString<"kumiko">, 6>>,
      Expect<Equal<LengthOfString<"reina">, 5>>,
      Expect<Equal<LengthOfString<"Sound! Euphonium">, 16>>
    ];
    ```
  - 현석
    Try 1. 아 이런 상수로 뽑는게 하나 있었는데
    ```tsx
    type LengthOfString<S extends string> = ((s: S) => 1) extends (
      s: infer Str
    ) => 1
      ? Str
      : 0;
    type Test = LengthOfString<"kumiko">; // === "kumiko"
    ```
    Try 2. ['length'] 로는 안 될것 같다
    ```tsx
    type T = "asd"["length"]; // "number"
    ```
    Try 3. #template-literal ?
    ```tsx
    type LengthOfString<S extends string, L = 0> = S extends `${infer A}${infer B}` ? (A extends '' ? L : LengthOfString<B, L + 1>) : 0
    // 덧셈이 안된다....
    ```
  - 화랑
  - 찬모
    예전에 **Last of Array**를 풀 때, 배열의 길이를 구하는 방법과 유사하게 접근하면 되지않을까 하는 생각과 함께 시작하였다.
    ```tsx
    // try1 : 당연히 안되겠지만 시도!
    type LengthOfString<S extends string> = S["length"];

    type T1 = LengthOfString<"kumiko">; // number

    // 배열인 경우에는 배열의 길이가 나온다.
    ```
    아이디어1 : 내가 해야할 것은 글자 한글자씩 잘라서 세는 방법?
    → 센다는 것 자체가 값을 이용하는 것이기 때문에 불가능할 것으로 예상
    아이디어2 : 우선 자르자. 문자열을 자르는 split 메소드를 타입으로 만들 수 있을까?
    → 이를 찾아보니... 있다... 이것을 이용하려고 했더니 이미 답이 되었다.
    [참고](https://www.typescriptlang.org/play?ts=4.1.0-dev.20201028&ssl=57&ssc=19&pln=57&pc=38#code/PTAEBUFMFsAcBsCGAXSp4EtUCdHwM6gDGiAdqAEZoCu+kAJqMgPaiQAeyuRyoZj0MhljUkqUPi4ZSAc3RZIueEwCesSPgB0AWABQICAAsNaSdmlzMOPKvX4ANKGlNq2Uo5LkqoWgz6FYbGZ1bGQMDUd+YjJQeg0icyo9A1hmfHwMCng0LjJ8ADNmbEEw5lJCfKDoPgkpWSZWGOYKACtIHidyGIBBAAUASR1dZLAAYlHQAGU6uUmELDD68FZu8gB5VvbkPRGIGAQUNCtFPEJPHzpQWBQccv9QACJ8eeQAWlTpZHwHhs78xSYxl2+GoFDMFkIzioyAA7pBIKRNKAAGJFNjsRBwbKaHE7fRgcCGDCEZBqNDEmrg+rHJSgGFEoiGYhlQrFEmNKYwABqilemAA1qYZkNSeoIJMedgMmVQABeR4AFk0AEZNAAmB54gwAdTQ5wSkEONVFOVYHFyHWQxmZcDKCK+oGY+UBKFq5lkQx1kAA5PBlM8rHwEmkSTDWN7NN7iIZENwcFo9CbQABRThxyYwABuigAPBnoNnsNN3XJzQj6IQqTIAHxyvSgBucguKYsWdGoUgV0AAAwAJABvaT-bCgACyiBaRQAvpoB0OAaPpNPZ4PSMPQL0UIyp93QAB+euN-ugQST7AALjHE6KjmgS4vY-vjmuyEZl83r6ZU9Al+PiiCD4PAAwmQpDMLw1xSqYWYAlWPxTlqYAADLSGgyoSIYzCiIw3j5JiGCYLGTjOioWHetm6DMMwgqMK6VpoIE7QMG2HCYggGjnrsrFYpA56fEErwsIJ+xiJAfIKEohBgNx7HnoIsDqPQglkvgrwwlghjCVihzidYBCIaAKGkGgapOIQiDMp2WAYGUNgmo44BkpMCTCLwmZ4Bg9CHCSMa8PRfwAi+twnluxj4LsiAyIg0iSE2hatvUkGYpAOBDLsRloAAzGZgJoNgGiiLwTq5ZZ9DWbZ8COBgzpcNQOTGOQgTMJmnloE0mw8Ls6lWiVIJgjMASIOkfifKwZX5MO9pXGk5V3M47VtDwSL9DV1pVrs9DMBooBgbwJSMo49HkPlyCuOQ-lJt1TIxP+aL4DG6hpboSbgJMcopmmiA8PmhY5q9krSqQ1YGYSFJhqQ3q8DGnbZE2kqgMqAAMiMAKSOFQJC+E4vAUjdGI8Zxz1kqAABCiD0D9igk9QyBrAA0glcjyqmFrIJT2A5g8SqI5oqKuJoCqCwqDzA8M+KgAAEswcKFo4LPpjBI7qX6oB4QRjrkFWhD0hgjKxGUkOq1gJWsiUSKg2cQ2QF1BHKGU8AqCFn50g1lIzKAMY+Wgpuug8AAamgAJqaAAWg8jg63r-nGZw8jGfr227YTSbs8m2AAe98tfWziuc3zI4fKQvAh4orCF7wyIYNmIsGeMoAAErtK4GQUYzUwvIsMgg9ajEtVhhAybDyt26QDt0kU-IuwioCkdQHuIBRePZ26bYsE7B27KyJ5FGgpDUGQRB+CQdCELPdJkLwYbYJPV0lY56jOeYsC8NzquGqd+WXoPvHueY5O64Jag7EExiwMMsCQLw3Ylk6GvfKrxaCICyHqZgtpjJFwcBAIB2RzL5RqDIKijAYSIEdmvQUkBYC7AtJPYq-l8ogngF8JEEtFA+nMhAwMJpCaJmJnMKweZ2zlkrDMRwAARARnYhEllrLKQ8DYqziK7G9PcK9ZAAG0AC6P5ZFTAUYQb0UZlEaK0boRsOiywSJ7HONcAJwBTgHCIuxq51wAFUdz7lAKo8AjgcSaF4VgHMzjRHVk0ZeVRkx1EAG4DIZQRrEdoSA6FMDDFcWMmIMFwl9MoLGGRZCw0ZKkngigKhogoPlFqpJPRgDevlRidB0G9XdmvAMWBIidlAGIik-k4iYDvKQFARRzZEnCuLTA8cEQgkSfRR2sY0AUHAkyLWT0DAxNMoydo-JITOnkbjeQel0aOzWUQfkbZqp4IRCcf0MxdjnG8OYuIjBKgoJKtIEQvAqzLS2cwRwJ0zpQLbLGXAKgkS6muWQQ2V8b4aR2mUXSFyVEyCWWAZMmgEWAgpMfXi3CxSTHQvKPxyAcxVkcA8TQNdQHIVQqAbKhyNnEQadAik+xSRVU+aAH5bg+DkCZSQrB1sibYtMnil4nNw6PFJaLdKlKFTzzYRkO8CTozrN+FhEcWdvqKw+fSts+0wq7G7Ko3o+V8gYHYPccA6jVEiMgD06Q-TsAWsmNQCaJr7jOPUbuI67Y4wlUYsa01AAKcAABKGBrB6KbwwFKCCqToApQBMVCyp12KHVdnA7A1ByD4o1r1J1frQD+ucSGte4zXB8oMFaV00Ad4unIHadeTIbneywp2SpuwVpavqJtRO4FOhEHgNQOIJVukYF6XalN092VdBcOxR0zp0K7GyLIHqkcG1lGQNFO4-l5HXGGowea5kujYBkNQWNRd82TCDYiqYI6Z0YqxWgSY2UhV8IeKqDUxLxUGW1LbNlTcoIa0Pr8GQKU+Aq38pGSsHcGD3qmNK59-jX3qk0JlUVJKyW7G-T1K0FJ+RgRhNkegwHHBn3ulheAOE2pIN+PlcmnLGAZriFGqIABHagCg+Cb0gDCWdJUiAoOgBmrAjtv6EBoSJI0NJTiOC3t-QmIBdivFAKsdEtTCBBBpmgc0cYSmOxETELk-BFD8kxLsQwyBkCwHwOeEAsIFiKE0Hx6AwAvKkEzPQfkwBJAoFoMAZUmVEbKgAOyI2C2qAAbAAVhC8qAAnAADgVGFhUIwFNKbica0gs1VZok2kQE99pNCscUCoDM2QeA6cfIKBuKgyCmfM5Z6zwBbPmfs454Ad5BTYBq52SAmZPPrtOvgXz-mEuBbVGqQLMWAvIzVBFhUgXAspfFr0SAwRYYxkXgQM07AQgjvtH4W+rH2OOfmCQVAjBt2xjoFKUAuxaCr3E+IeRkmCARwZNdXB+QM2gEUxQGm0LeD5Uc6e+52XsC7CavQagPAbLkD43ECgVsQHyfFmZizVmQAyA0qCBzKDgAyEwqITAihgCIDhPgFBYlUDaVQLCpQyk7C7F2GsEcNHGD+VAs2w+p7eBZGYHIVIkg5NgDRw1kAcRMx85kFoO8wYKf5GQLjpzJp8AuWfqT0gYEM1EAsAz+IT83gKleMqV40JEDAFGNTg4tOXt6-wEAA)
    ```tsx
    type Split<S extends string, D extends string> =
      // string extends S ? string[] : // 이 부분은 없어도 성공, 왜 존재하는 코드이지??
      S extends ""
        ? []
        : S extends `${infer T}${D}${infer U}`
        ? [T, ...Split<U, D>]
        : [S];

    type LengthOfString<S extends string> = Split<S, "">["length"];
    ```
  - 지선
    ```tsx
    //try 1
    type LengthOfString<S extends string> = S["length"];

    //type A = LengthOfString<'kumiko'>
    // type A = number

    //try2 👌
    type LengthOfString<
      S extends string,
      Arr extends string[] = []
    > = S extends `${infer A}${infer B}`
      ? LengthOfString<B, [A, ...Arr]>
      : Arr["length"];
    ```
  - 재영
    - try1
      ```tsx
      // 중요해보이는 성질 ?
      type Test = `` extends `${infer First}${infer Rest}` ? 1 : 2; // 2
      type Test = `` extends `${infer First}` ? 1 : 2; // 1
      ```
