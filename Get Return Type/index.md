### Get Return Type

- 연구
  ### **Get Return Type**
  Implement the built-in `ReturnType<T>` generic without using it.
  For example
  ```tsx
  const fn = (v: boolean) => {
    if (v) return 1;
    else return 2;
  };

  type a = MyReturnType<typeof fn>; // should be "1 | 2"
  ```
  ```tsx
  //try 1
  // 파라미터가 불리언일 경우 처리가 안되는 건가..?
  type MyReturnType<T> = T extends (...args: unknown[]) => infer K ? K : never;

  /* _____________ Test Cases _____________ */
  import { Equal, Expect } from "@type-challenges/utils";

  type cases = [
    Expect<Equal<string, MyReturnType<() => string>>>,
    Expect<Equal<123, MyReturnType<() => 123>>>,
    Expect<Equal<ComplexObject, MyReturnType<() => ComplexObject>>>,
    Expect<Equal<Promise<boolean>, MyReturnType<() => Promise<boolean>>>>,
    Expect<Equal<() => "foo", MyReturnType<() => () => "foo">>>,
    Expect<Equal<1 | 2, MyReturnType<typeof fn>>>, // 실패
    Expect<Equal<1 | 2, MyReturnType<typeof fn1>>> // 실패
  ];

  type ComplexObject = {
    a: [12, "foo"];
    bar: "hello";
    prev(): number;
  };

  const fn = (v: boolean) => (v ? 1 : 2);
  const fn1 = (v: boolean, w: any) => (v ? 1 : 2);
  ```
  ```tsx
  //try 2
  // unknown이 아니라 any[] 로 해주면 통과한다 뭐지...?
  type MyReturnType<T> = T extends (...args: any[]) => infer K ? K : never;

  /* _____________ Test Cases _____________ */
  import { Equal, Expect } from "@type-challenges/utils";

  type cases = [
    Expect<Equal<string, MyReturnType<() => string>>>,
    Expect<Equal<123, MyReturnType<() => 123>>>,
    Expect<Equal<ComplexObject, MyReturnType<() => ComplexObject>>>,
    Expect<Equal<Promise<boolean>, MyReturnType<() => Promise<boolean>>>>,
    Expect<Equal<() => "foo", MyReturnType<() => () => "foo">>>,
    Expect<Equal<1 | 2, MyReturnType<typeof fn>>>, // 실패
    Expect<Equal<1 | 2, MyReturnType<typeof fn1>>> // 실패
  ];

  type ComplexObject = {
    a: [12, "foo"];
    bar: "hello";
    prev(): number;
  };

  const fn = (v: boolean) => (v ? 1 : 2);
  const fn1 = (v: boolean, w: any) => (v ? 1 : 2);
  ```
- 현석
  ```tsx
  type MyReturnType<T extends (...args: any) => any> = T extends (
    ...args: any
  ) => infer R
    ? R
    : never;
  ```
  흠 그런데 똑같은걸 두 번 쓰는게 싫다.
  ```tsx
  type MyReturnType<T> = T extends (...args: any) => infer R ? R : never;
  ```
  T 자리에 뭘 넣었을때 에러를 뱉어야 한다는 케이스가 없으니, 이렇게 써도 된다.
  이런 코드를 넣으면 에러가 날까?
  ```tsx
  type MyReturnType<T> = T extends (...args: any) => infer R ? R : never;

  // @ts-expect-error <- 그렇다. 에러가 나야 하는데 안 난다고 빨간 줄이 생긴다.
  type Test = MyReturnType<number>; // never
  ```
  Let’s talk about
    <aside>
    👉 ReturnType은 어떻게 구현되어있을까? 
    ReturnType<number>는 에러가 날까 never를 반환할까?
    **extends를 사용해 Generic을 제한하는 것과 never를 통해 타입 에러를 내는 것, 어떤 게 더 좋은 방법일까?**
    
    </aside>

- **화랑**
  1차 시도: `type MyReturnType<T> = T extends Function ? ReturnType<T> : never`
  테스트는 통과하지만 아래와 같은 에러 메세지가 든다
  - Type 'Function & T' does not satisfy the constraint '(...args: any) => any'.
    Type 'Function' provides no match for the signature '(...args: any): any'.
  공식 문서에서 ReturnType을 확인해봤더니 <Function> 이라고 바로 쓰면 안되는 구나. 제시된 것 처럼 (...args: any): any 를 넘기자
  `type MyReturnType<T> = T extends (...args: any[]) => infer U ? U : never`
  ![Untitled](%E1%84%90%E1%85%A1%E1%84%8B%E1%85%B5%E1%86%B8%20%E1%84%8E%E1%85%A2%E1%86%AF%E1%84%85%E1%85%B5%E1%86%AB%E1%84%8C%E1%85%B5%205%E1%84%92%E1%85%AC%E1%84%8E%E1%85%A1%2069d814bae1c94b068ff310a12f360c2b/Untitled.png)
  > 궁금한 것
  ReturnType<string> 은 사용 할 수 없는데, ReturnType<any> 와 ReturnType<never> 은 사용할 수 있는 이유가 뭘까? `(...args : any) ⇒ any` 제한을 지키지 않아도 괜찮나?
- 찬모
  ```tsx
  // Try1
  type MyReturnType<T> = T extends () => infer R ? R : any

  // 이거 2개가 왜 에러지?? 🤔
  Expect<Equal<1 | 2, MyReturnType<typeof fn>>>,
  Expect<Equal<1 | 2, MyReturnType<typeof fn1>>>,
  ```
- 지선
  ```tsx
  const fn = (v: boolean) => {
    if (v)
      return 1
    else
      return 2
  }

  type a = MyReturnType<typeof fn> // should be "1 | 2"

  --------
  // T가 함수인지 체크해서 infer로 반환 함수 넘겨주기.
  //🙆‍♀️
  type MyReturnType<T> = T extends (...args:any[]) => infer R ? R : never

  ```
- 재영
  - idea
    - return type을 추론해야하니까 infer
    - infer를 사용하기 위해서 모양 맞춰보기를 해야하니까 extends
  - try 1
    ```tsx
    type MyReturnType<T> = T extends () => infer R ? R : any;

    // 여기서 const fn = (v: boolean) => v ? 1 : 2 이렇게 conditional 하게 반환되는
    // 애들은 infer로 추론이 안되나보다.

    // 정답봄
    // 아!! 그냥 모양이 안맞아서 추론이 안된거였다. arguments 모양도 맞춰줘야한다.
    type MyReturnType<T> = T extends (...args: any) => infer R ? R : any;
    ```
- 주운
  ```tsx
  // 같은 타입을 반환하도록 타입을 지정한다.

  type MyReturnType<T> = T extends (...args: never[]) => infer R ? R : never;
  ```
  [https://code-masterjung.tistory.com/123](https://code-masterjung.tistory.com/123)
  **infer** : `infer` 키워드는 조건부 타입에서 사용하면 실제 분기의 비교할 유형에서 추론하는 방법을 제공해줄 수 있다.
  `MyReturnType<T>`를 쓰고 `=` 표시로 어떤 타입이 될 것인지 보여주는데, `MyReturnType` 함수는 왼쪽에 있는 값과 동일한 값을 반환한다. () 매개변수를 비워놓으면 v, w를 받는 함수의 타입이 통과하지 못한다.
  R이면 R을 반환한다로 추론...🧐
