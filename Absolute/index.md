## Absolute

- 연구
- 현석
- 화랑
- 찬모
  - 모두 값에 대한 이야기인데....깨작깨작거리다가 상단의 태그 힌트 발견 `template-literal`
  - 결과값으로 반환되는 것이 `positive number string` , 양수인 숫자 문자열이 반환된다??
  - 2021년 어떤 인도 개발자분이 해당 문제와 연관된 [제안](https://github.com/microsoft/TypeScript/issues/47141)을 남기셨다.
    → number to string | string to number
  ```tsx
  // try1
  type NumberToString<T extends number> = `${T}`;

  type Absolute<T extends number | string | bigint> = T extends NumberToString<
    infer X
  >
    ? `${X}`
    : `${T}`;

  // 제안을 이용해서 하려고 했는데, 뭔가 안되는 부분이 많았음.
  // 그리고 infer X 이기때문에 X를 반환하면 타입을 반환하게 됨
  // → ex) type T3 = Absolute<"10"> // `${number}`
  ```
  💡아이디어
  - 타입별로(**string | number | bigInt**) 구분해서 접근해보자
  - `template-literal` 이 힌트니까 (앞선 문제들을 생각해보면) **-(마이너스)** 도 문자열로서 접근할 수 있지않을까?
  ```tsx
  // try2
  type Absolute<T extends number | string | bigint> = T extends number | bigint
    ? `${T}`
    : T extends `${infer A}${infer B}`
    ? A extends "-"
      ? B
      : `${T}`
    : never;
  ```
  2개의 test case가 에러!
  - `Absolute<-5>`
    → 여기서 얻은 아이디어 : 모든 숫자는 문자열로 변형시키고 앞에 -를 제외하고 반환하자
    → 앞서 구현한 **type NumberToString** 이용
  - `Absolute<-1_000_000n>`
    → **9_999n**는 작동하는데 위에 테스트는 작동안한네... BigInt에 대해서 알아보자.
        → BigInt

        - 새로운 하나의 타입으로서 끝에 **n**을 적어준다.
        **중간에 _는 큰 수니까 세자리씩 끊어써주는 형식으로서 적어주는 것**이라고 한다.
        - BigInt를 문자열로 변형시키면 말 그대로 타입 변경되어 숫자로된 문자열이 된다.
        → 여기서 얻은 아이디어 :  마찬가지로 모든 BigInt도 문자열로 변형시키고 앞에 -를 제외하고 반환하자
  ```tsx
  // try3 😎
  type NumberToString<T extends number> = `${T}`;

  type Absolute<T extends number | string | bigint> = T extends number | bigint
    ? Absolute<NumberToString<T>>
    : T extends `${infer A}${infer B}`
    ? A extends "-"
      ? B
      : `${T}`
    : never;
  ```
- 지선
  ```tsx
  type Test = -100;
  type Result = Absolute<Test>;
  // expected to be "100"

  /////

  Math.abs()는 사용이....? 안되지....
  T> 0 ? 도? 안되지.... ^_^;;;;;;;;;;

  // answer
  type Absolute<T extends number | string | bigint> = `${T}`extends `-${infer S}` ? `${S}`:`${T}`
  ```
- 재영
  - try1
    ```tsx
    type Absolute<T extends number | string | bigint> = `${T}`;
    type TEST = Absolute<"-5">; //-5

    // 음 ... -(어쩌고) 라는 모양에 끼워진다면(extends) (어쩌고)만 밖으로 뺴주자 (infer)
    ```
  - try2
    ```tsx
    // 완성!
    type Absolute<
      T extends number | string | bigint
    > = `${T}` extends `-${infer U}` ? U : `${T}`;
    ```
