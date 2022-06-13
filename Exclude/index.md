- **Exclude**
  **Problem**
  Implement the built-in Exclude<T, U>
  Constructs a type by excluding from UnionType all union members that are assignable to ExcludedMembers
  > Exclude from T those types that are assignable to U
  ```tsx
  type cases = [
    Expect<
      Equal<MyExclude<"a" | "b" | "c", "a">, Exclude<"a" | "b" | "c", "a">>
    >,
    Expect<
      Equal<
        MyExclude<"a" | "b" | "c", "a" | "b">,
        Exclude<"a" | "b" | "c", "a" | "b">
      >
    >,
    Expect<
      Equal<
        MyExclude<string | number | (() => void), Function>,
        Exclude<string | number | (() => void), Function>
      >
    >
  ];
  ```
  - 찬모
    ```tsx
    # Try1
    type MyExclude<T, U> = T extends keyof U ? never : U 🤪
    type MyExclude<T, U> = T extends U ? never : U 🤪

    # Try2
    type MyExclude<T, U> = T extends U ? never : T
    ```
    Try1
    문제를 읽고 든 첫 생각은 `never 타입`과 `conditional type`을 사용해서 표현해야한다는 것!
    - never타입은 특정 타입을 제외해야하기 때문에
    - extends는 특정 조건에서 제외해야하기때문에 조건을 표현해야하기때문에
    이런 생각을 가지고 몇가지 구현하다가 보니 결과가 반대로 나오는 것을 확인하였다. T 타입에서 U 타입을 제외하여야하는데 자꾸 U 타입을 리턴하도록 구현했음을 알게 되었다. keyof연산자에 대해서 명확히 이해하지 못해서 저런 요상한 코드를 짜게되었다.
    ## keyof 연산자에 대해서
    keyof 연산자는 객체 타입에서 속성의 키값을 문자열 혹은 숫자 리터럴 유니온 타입으로 가져온다.
    ```tsx
    type Person = {
      name: "jjanmo";
      gender: "male";
      age: 100;
      7: "favorite number";
    };

    type Key = keyof Person;
    // 'name' | 'gender' | 'age' | 7
    ```
    ## Exclude의 작동 방식/순서
    ```tsx
    // Step1
    type A = <MyExclude<"a" | "b" | "c", "a">

    // Step2
    type A = <MyExclude<"a", "a"> | <MyExclude<"b", "a"> | <MyExclude<"c", "a">

    // Step3
    type A = never | "b" | "c"

    // Step4
    type A = "b" | "c"
    ```
  - 지선
    ```tsx
    // T에서 U에 할당할 수 있는 타입을 제외하는 내장 제네릭 Exclude<T, U>를 이를 사용하지 않고 구현하세요.
    type MyExclude<T, U> = T extends U ? never : T;

    interface Todo {
      id: string;
      title: string;
      isDone: boolean;
    }

    interface Memo {
      id: string;
      title: string;
      content: string;
    }

    type Contents = MyExclude<Todo, Memo | Todo>; //never //Todo는 Memo|Todo에 할당되므로
    type Contents = MyExclude<Todo | Memo, Todo>; // Memo
    // = (Todo|Memo) extends Todo ? never : (Todo|Memo) -> Todo|Memo //이걸 생각했는데
    //결과는 이런 느낌? why? [분배 조건부 타입 참조](https://velog.io/@zeros0623/TypeScript-%EA%B3%A0%EA%B8%89-%ED%83%80%EC%9E%85#%EB%B6%84%EB%B0%B0-%EC%A1%B0%EA%B1%B4%EB%B6%80-%ED%83%80%EC%9E%85distributive-conditional-types)👉
    //= (Todo extends Todo ? never : Todo)  | (Memo extends Todo ? never:Memo)
    //= never | Memo
    //= Memo
    ```
    [https://blog.martinwork.co.kr/typescript/2019/05/28/typescript-util-types.html](https://blog.martinwork.co.kr/typescript/2019/05/28/typescript-util-types.html)
    [https://typescript-kr.github.io/pages/utility-types.html#excludetu](https://typescript-kr.github.io/pages/utility-types.html#excludetu)
  - 재영
    - 아이디어
      - “할당” → `extends`
      - `Exclude`는 [Predefined conditional types](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-2-8.html#predefined-conditional-types)에 해당하니까 conditional Type을 사용하겠지...
      - 유니온 타입인 T를 순회하면서 요소 하나하나가 U에 할당할 수 있는지 체크하려면 어떻게 해야할까(extends 이용)
        - 할당할 수 있다면 순회하고 있는 그 값을 적용하고 아니면 never를 적용한다.
      - 유니온 타입들을 하나하나 순회한다. → 어떻게???
    - 아하
      - **[Distributive Conditional Types](https://www.typescriptlang.org/docs/handbook/2/conditional-types.html#distributive-conditional-types)**
        - When conditional types act on a generic type, **they become *distributive***
           when given a union type.
        → 그러니까 T에 union type이 오고 conditional type이 걸리면 마치 map함수가 리턴값 반환하듯 순회하면서 새로운 union type을 반환한다는 뜻
    - answer
      ```tsx
      type MyExclude<T, U> = T extends U ? never : T;
      ```
  - 화랑
  - 연구
    ExcludedMembers를 제외한 Union Type으로 타입을 구성한다
    ```tsx
    type MyExclude<T, U> = T extends U ? never : T;

    type a = MyExclude<1 | "a" | [], number>; // 'a' | []
    ```
  - **현석**
    Try 1.
    ```tsx
    type MyExclude<T, U, A = any> = A extends T
      ? A extends U
        ? never
        : A
      : never;

    // 어떤 타입 A가 T에만 속하고 U에 속하면 안된다.... 는 이야기를 풀어보았다.
    // A가 any가 되다 보니, 결과 값도 any가 되어버린다.
    ```
    Try 2.
    ```tsx
    type MyExclude<T, U> = T extends U ? never : T;

    // union type의 값들은 그 중 하나가 들어온다- 는 방식으로 하나씩 타입 추론이 진행되는 듯 하다.
    // 아주 간단하게 해결되어버렸다.
    ```

### 리얼클래스에서 Infer를 사용중인 코드

useLimitedTimeValue.ts 가 있는데, 열어보지 말고!

아래 문제를 풀어봅시다.

```tsx
interface PreDefined {
  topBanner: LimitedTime<string>;
  loginForPayments: LimitedTime<string>;
  bookBadge: LimitedTime<string>;
  saleTimer: LimitedTime<{ deadline: string; interval: number }>;
  liveOpen: LimitedTime<string>;
  liveDelayMessage: LimitedTime<string>;
  welcomeLive: LimitedTime<number>; // 리얼패스 - 회원가입시 할인쿠폰 금액
  welcomeBoost: LimitedTime<number>; // 부스트 - 상담시 지급되는 할인쿠폰 금액
}

interface LimitedTime<T> {
  timers?: {
    value: T;
    startDate: string;
    endDate: string;
  }[];
}

/* _____________ Your Code Here _____________ */

type ValueType<K> = any;

/* _____________ Test Cases _____________ */
import { Equal, Expect } from "@type-challenges/utils";

// Question. 아래 ValueType이 동작하도록 타입을 만들어보자.
function useLimitedTimeValue<K extends keyof PreDefined>(
  key: K,
  fallbackValue?: ValueType<K>
): ValueType<K> {
  return (null as any) as ValueType<K>;
}

const topBanner = useLimitedTimeValue("topBanner");
const saleTimer = useLimitedTimeValue("saleTimer");
type cases = [
  Expect<Equal<ValueType<"topBanner">, string>>, // 이렇게 문제를 이해해도 상관없음
  Expect<Equal<typeof saleTimer, { deadline: string; interval: number }>>
];
```
