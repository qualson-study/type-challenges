- **First of Array**
  **Problem**
  Implement a generic `First<T>` that takes an Array `T` and returns it's first element's type.
  For example
  ```tsx
  type arr1 = ["a", "b", "c"];
  type arr2 = [3, 2, 1];

  type head1 = First<arr1>; // expected to be 'a'
  type head2 = First<arr2>; // expected to be 3
  ```
  - **연구**
    ```tsx
    type arr1 = ["a", "b", "c"];

    type arr2 = [3, 2, 1];

    type First<T extends any[]> = T[0];

    type First2<T extends any[]> = T extends [] ? never : T[0];

    // infer는  extends 절에서만 사용가능하며 특정 변수를 설정해 반환하거나 참조할 수 있게 해준다.
    type First3<T> = T extends [infer U, ...unknown[]] ? U : never;

    type head = First<[]>; // undefined

    type head2 = First<arr2>; // 3

    type head3 = First2<[]>; // never

    type head4 = First2<arr2>; // 3
    ```
  - 현석
    Try 1.
    ```tsx
    type First<T extends any[]> = T[0]

    // 여기에서 에러가 난다.
    Expect<Equal<First<[]>, never>>
    // First<[]> 가 never가 아닌 undefined가 나오는 문제
    ```
    Try 2.
    ```tsx
    type First<T extends any[]> = T[0] extends undefined ? never : T[0]

    // 그러면 여기에서 에러가 난다.
    Expect<Equal<First<[undefined]>, undefined>>
    ```
    Try 3.
    ```tsx
    // 뭔가 특정한 원소가 들어간 타입인지 체크하고 싶었다.
    type First<T extends any[], A extends any> = T extends [A] ? A : never;

    // First 라는 Generic이 두 개의 인자를 받아야 한다며 아무것도 되지 않는다.
    ```
    Try 4.
    ```tsx
    // 0번이 있는지만 체크해보자
    type First<T extends any[]> = T extends { 0: any } ? T[0] : never;

    // PERFECT!
    ```
  - 화랑
    - T extends any []: T 에는 어떤 형태의 배열이든 올 수 있구나!
    - 그럼 T[0]...?
    - 예시를 보니 빈 배열일 때는 never 이네.
    - 예전에 챌린지 코드 짤 때, 타입도 조건문으로 쓸 수 있다고 배웠다
    - `T extends [] ? never : T[0]`
    - never 타입은 언제쓰지? 절대 발생할 수 없는 타입! 빈 배열이면 첫 번째 원소라는게 절대 발생할 수 없기 때문에 never 를 썼구나
  - 찬모
    도전!! 🚀
    ```tsx
    # Try1 😡
    type First<T extends any[]> = {
      T[0] : typeof T[0]
    }

    # Try2 🧐
    type First<T extends any[]> = T[0]

    # Try3 😅
    type First<T extends any[]> = T extends [] ? never : T[0]
    ```
    Try1
    아무런 생각 없이 😡  배열 첫번째의 타입만 주면 되는거잖아 라는 생각에 말도 안되는 코드를 작성하였다. 그리고 나서 경험한 많은 에러로 조금 생각을 해봤다. 해당 문제는 First<T>에 하나의 값으로 떨어지기 때문에 객체형태가 아님을 인지하였다.
    Try2 & Try3
    T[0], 아주 간단하게 생각하였다. 이전 푼 문제(TupleToObject)에서 힌트를 떠올려서 T[number] 혹은 T[0]이 가능하다는 것을 이용해서 첫번째 타입을 주었다. 그런데 여기서도 한가지 에러가 발생하였다. 그 경우는 빈 배열인 경우는 `never 타입`을 줘야한다는 것이였다. 그래서 빈 배열인 경우를 조건을 빼서 타입을 주는 방식으로 구현하는 방법을 찾아보았다. 이 때 사용할 수 있는 방식은 `conditional type` 을 활용하는 것이였다. 그래서 위와 같은 코드를 작성 하였다.
    실제로 답을 확인해보니 아래와 같이 푼 경우도 있었다. 이 부분을 보면서 궁금했던 부분을 찾아보았다.
    ```tsx
    #1
    type First<T extends any[]> = T extends never[] ? never : T[0]
    type First<T extends any[]> = T extends [] ? never : T[0]

    #2
    type First<T extends any[]> = T[number] extends never ? never: T[0]

    #3
    type First<T extends any[]> = T['length'] extends 0 ? never : T[0]

    #4
    type First<T extends any[]> = T[0] extends T[number] ? T[0] : never
    ```
    ### #1
    **Q1. never 에 대해서**
    1. 함수의 리턴 타입으로 쓰일 때, 리턴 타입을 내보낼 수 없음을 의미한다.
       → undefined를 리턴하는 것을 의미하지 않는다.
       (void 타입의 경우 암묵적으로 undefined를 리턴하기 때문에 never 타입이 아니다.)
       → 함수가 정상적인 종료가 아닌 경우를 의미한다. : throws an error | infinite loop
    2. 타입으로서 의미
       → 모든 타입의 슈퍼셋(최상위 타입, top type)은 `unknown` 이고 반대로 모든 타입의 서브셋(최하위 타입, bottom type)은 `never` 이다. 그렇기 때문에 어떤 타입도 never 타입에 할당 할 수 없다.
       (ex. `string extends never` **X**)
       → the empty union type, 공집합으로서 해석된다. 그래서 유니온 타입 안에서 never 타입을 사용하게 되면 이 자체가 의미가 없기 때문에 내부적으로 제거된다. (이 부분이 Exclude 문제에서 활용된다)
    ```tsx
    type Type = number | string | never;

    // type Type = number | string 과 같다.
    ```
    - 참고 Top/Bottom type에 관해서
      [https://medium.com/@KevinBGreene/a-little-theory-with-your-typescript-top-and-bottom-types-61b380f227d](https://medium.com/@KevinBGreene/a-little-theory-with-your-typescript-top-and-bottom-types-61b380f227d)
      [https://www.typescript-training.com/course/fundamentals-v3/11-top-and-bottom-types/](https://www.typescript-training.com/course/fundamentals-v3/11-top-and-bottom-types/)
    **Q2. never[ ] vs [ ]**
    ```tsx
    type D = never[] extends [] ? "true" : "false"; // 'false'

    type E = [] extends never[] ? "true" : "false"; // 'true'
    ```
    > 이 부분은 찾는 중...
    ### #3
    처음에 생각한 풀이 방법인데, 구현하는 방법을 몰랐다. 배열의 길이가 0임을 저런식으로 표현할 수 있다는 점이 새로웠다.
    ### conditional type
    자바스크립트의 삼항 연산자와 같은 방식으로 타입을 설정하는 방식. 단 주의할 점은 삼항 연산자 처럼 다루기때문에 결과값을 값으로 착각할 수 있지만, 이는 오로지 **타입**을 표현하는 것이다.
    ```tsx
    T extends U ? X : Y
    ```
    ## `extends` 에 자세히 좀 더 이해해보기
    ### 일반적으로 알려진 extends의 의미
    - 상속관계
      ![스크린샷 2022-02-16 오후 10.26.59.png](%E1%84%90%E1%85%A1%E1%84%8B%E1%85%B5%E1%86%B8%20%E1%84%8E%E1%85%A2%E1%86%AF%E1%84%85%E1%85%B5%E1%86%AB%E1%84%8C%E1%85%B5%202%E1%84%92%E1%85%AC%E1%84%8E%E1%85%A1%20d0bb54e89b044540a04005a42169906f/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-02-16_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_10.26.59.png)
      상속관계는 위에서 아래로 상속관계를 표현할 수 있다. 이걸 보면 위(조상)에 있는 것이 넓은 개념으로서 아래(자식)의 개념을 포함하는 관계로서 이해될 수 있다.
      그런데 이런 식으로 이해하면 타입스크립트의 타입에서 extends를 위에 처럼 이해하는데 약간 어려움이 있었다. 타입스크립트는 `structural typing(구조적 타이핑)` 을 기반으로 한다. 이 말은 타입의 멤버(속성)의 구조가 어떻게 되어 있냐에 따라서 타입의 관계를 파악하는 것을 말한다. 이러한 관점에서 아래에서 extends를 정의한 표현을 살펴보자.
    ```tsx
    type Animal = {
      name: string;
      age: number;
    };

    type Lion = {
      name: string;
      age: number;
      roar: () => void;
    };

    type A = Lion extends Animal ? "true" : "false";
    type B = Animal extends Lion ? "true" : "false";
    ```
    `A extends B`
    → A is assignable to B
    → A가 B에 할당가능하다
    → 어떤 타입의 구조를 가지고 있을 때 할당 가능할까?
    - A is a superset of B
    → 포함관계 : A(superset) > B(subset)
    ![스크린샷 2022-02-17 오전 1.29.58.png](%E1%84%90%E1%85%A1%E1%84%8B%E1%85%B5%E1%86%B8%20%E1%84%8E%E1%85%A2%E1%86%AF%E1%84%85%E1%85%B5%E1%86%AB%E1%84%8C%E1%85%B5%202%E1%84%92%E1%85%AC%E1%84%8E%E1%85%A1%20d0bb54e89b044540a04005a42169906f/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-02-17_%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB_1.29.58.png)
    ex)
    Typescript is a superset of Javascript
    → Typescript extends Javascript ~ 요런 느낌!
    - A is a possibly-more-specific version of B
    → 구체성 : more specific
    구조적으로 타입의 속성이 더 많은 경우를 말한다. 위 예시에서를 보면 Lion이 더 많은 타입의 속성을 갖고 있기 때문에 Animal에 할당 가능하다.
  - 지선
    ```tsx
    1. type First<T extends any[]> = T[0]
    // -> T에 빈배열 들어갈 경우 : First<[]> : never로 반환 필요
    // 삼항 조건 연산자 사용하여 빈배열 체크?
    // 조건부 타입(conditional Types) 사용

    //ok
    type First<T extends any[]> = T[0] extends T[number] ? T[0] : never
    ```
    - [조건부 타입(conditional Types)](https://typescript-kr.github.io/pages/advanced-types.html)
      `T extends U ? X : Y` : T가 U에 할당될 수 있으면 타입은 X 아니면 Y.
  - 재영
    - 요구사항
      - 제네릭이 배열을 받고 있을 때, 인덱스로 배열의 요소에 접근해서 타입을 만들어보자
      - 단, 제네릭이 빈 배열을 받고 있다면, `never`를 반환해야 한다.
    - 아이디어
      - conditional Type
      - 빈배열이냐?를 어떻게 표현할까 → `T extends []` 혹은 `T[’length’] extends 0`
    - answer
      ```tsx
      type First<T extends any[]> = T extends [] ? never : T[0];

      // 혹은

      type First<T extends any[]> = T["length"] extends 0 ? never : T[0];
      ```
