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
  - μ°¬λͺ¨
    ```tsx
    # Try1
    type MyExclude<T, U> = T extends keyof U ? never : U π€ͺ
    type MyExclude<T, U> = T extends U ? never : U π€ͺ

    # Try2
    type MyExclude<T, U> = T extends U ? never : T
    ```
    Try1
    λ¬Έμ λ₯Ό μ½κ³  λ  μ²« μκ°μ `never νμ`κ³Ό `conditional type`μ μ¬μ©ν΄μ ννν΄μΌνλ€λ κ²!
    - neverνμμ νΉμ  νμμ μ μΈν΄μΌνκΈ° λλ¬Έμ
    - extendsλ νΉμ  μ‘°κ±΄μμ μ μΈν΄μΌνκΈ°λλ¬Έμ μ‘°κ±΄μ ννν΄μΌνκΈ°λλ¬Έμ
    μ΄λ° μκ°μ κ°μ§κ³  λͺκ°μ§ κ΅¬ννλ€κ° λ³΄λ κ²°κ³Όκ° λ°λλ‘ λμ€λ κ²μ νμΈνμλ€. T νμμμ U νμμ μ μΈνμ¬μΌνλλ° μκΎΈ U νμμ λ¦¬ν΄νλλ‘ κ΅¬ννμμ μκ² λμλ€. keyofμ°μ°μμ λν΄μ λͺνν μ΄ν΄νμ§ λͺ»ν΄μ μ λ° μμν μ½λλ₯Ό μ§κ²λμλ€.
    ## keyof μ°μ°μμ λν΄μ
    keyof μ°μ°μλ κ°μ²΄ νμμμ μμ±μ ν€κ°μ λ¬Έμμ΄ νΉμ μ«μ λ¦¬ν°λ΄ μ λμ¨ νμμΌλ‘ κ°μ Έμ¨λ€.
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
    ## Excludeμ μλ λ°©μ/μμ
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
  - μ§μ 
    ```tsx
    // Tμμ Uμ ν λΉν  μ μλ νμμ μ μΈνλ λ΄μ₯ μ λ€λ¦­ Exclude<T, U>λ₯Ό μ΄λ₯Ό μ¬μ©νμ§ μκ³  κ΅¬ννμΈμ.
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

    type Contents = MyExclude<Todo, Memo | Todo>; //never //Todoλ Memo|Todoμ ν λΉλλ―λ‘
    type Contents = MyExclude<Todo | Memo, Todo>; // Memo
    // = (Todo|Memo) extends Todo ? never : (Todo|Memo) -> Todo|Memo //μ΄κ±Έ μκ°νλλ°
    //κ²°κ³Όλ μ΄λ° λλ? why? [λΆλ°° μ‘°κ±΄λΆ νμ μ°Έμ‘°](https://velog.io/@zeros0623/TypeScript-%EA%B3%A0%EA%B8%89-%ED%83%80%EC%9E%85#%EB%B6%84%EB%B0%B0-%EC%A1%B0%EA%B1%B4%EB%B6%80-%ED%83%80%EC%9E%85distributive-conditional-types)π
    //= (Todo extends Todo ? never : Todo)  | (Memo extends Todo ? never:Memo)
    //= never | Memo
    //= Memo
    ```
    [https://blog.martinwork.co.kr/typescript/2019/05/28/typescript-util-types.html](https://blog.martinwork.co.kr/typescript/2019/05/28/typescript-util-types.html)
    [https://typescript-kr.github.io/pages/utility-types.html#excludetu](https://typescript-kr.github.io/pages/utility-types.html#excludetu)
  - μ¬μ
    - μμ΄λμ΄
      - βν λΉβ β `extends`
      - `Exclude`λ [Predefined conditional types](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-2-8.html#predefined-conditional-types)μ ν΄λΉνλκΉ conditional Typeμ μ¬μ©νκ² μ§...
      - μ λμ¨ νμμΈ Tλ₯Ό μννλ©΄μ μμ νλνλκ° Uμ ν λΉν  μ μλμ§ μ²΄ν¬νλ €λ©΄ μ΄λ»κ² ν΄μΌν κΉ(extends μ΄μ©)
        - ν λΉν  μ μλ€λ©΄ μννκ³  μλ κ·Έ κ°μ μ μ©νκ³  μλλ©΄ neverλ₯Ό μ μ©νλ€.
      - μ λμ¨ νμλ€μ νλνλ μννλ€. β μ΄λ»κ²???
    - μν
      - **[Distributive Conditional Types](https://www.typescriptlang.org/docs/handbook/2/conditional-types.html#distributive-conditional-types)**
        - When conditional types act on a generic type, **they becomeΒ *distributive***
          Β when given a union type.
        β κ·Έλ¬λκΉ Tμ union typeμ΄ μ€κ³  conditional typeμ΄ κ±Έλ¦¬λ©΄ λ§μΉ mapν¨μκ° λ¦¬ν΄κ° λ°ννλ― μννλ©΄μ μλ‘μ΄ union typeμ λ°ννλ€λ λ»
    - answer
      ```tsx
      type MyExclude<T, U> = T extends U ? never : T;
      ```
  - νλ
  - μ°κ΅¬
    ExcludedMembersλ₯Ό μ μΈν Union TypeμΌλ‘ νμμ κ΅¬μ±νλ€
    ```tsx
    type MyExclude<T, U> = T extends U ? never : T;

    type a = MyExclude<1 | "a" | [], number>; // 'a' | []
    ```
  - **νμ**
    Try 1.
    ```tsx
    type MyExclude<T, U, A = any> = A extends T
      ? A extends U
        ? never
        : A
      : never;

    // μ΄λ€ νμ Aκ° Tμλ§ μνκ³  Uμ μνλ©΄ μλλ€.... λ μ΄μΌκΈ°λ₯Ό νμ΄λ³΄μλ€.
    // Aκ° anyκ° λλ€ λ³΄λ, κ²°κ³Ό κ°λ anyκ° λμ΄λ²λ¦°λ€.
    ```
    Try 2.
    ```tsx
    type MyExclude<T, U> = T extends U ? never : T;

    // union typeμ κ°λ€μ κ·Έ μ€ νλκ° λ€μ΄μ¨λ€- λ λ°©μμΌλ‘ νλμ© νμ μΆλ‘ μ΄ μ§νλλ λ― νλ€.
    // μμ£Ό κ°λ¨νκ² ν΄κ²°λμ΄λ²λ Έλ€.
    ```

### λ¦¬μΌν΄λμ€μμ Inferλ₯Ό μ¬μ©μ€μΈ μ½λ

useLimitedTimeValue.ts κ° μλλ°, μ΄μ΄λ³΄μ§ λ§κ³ !

μλ λ¬Έμ λ₯Ό νμ΄λ΄μλ€.

```tsx
interface PreDefined {
  topBanner: LimitedTime<string>;
  loginForPayments: LimitedTime<string>;
  bookBadge: LimitedTime<string>;
  saleTimer: LimitedTime<{ deadline: string; interval: number }>;
  liveOpen: LimitedTime<string>;
  liveDelayMessage: LimitedTime<string>;
  welcomeLive: LimitedTime<number>; // λ¦¬μΌν¨μ€ - νμκ°μμ ν μΈμΏ ν° κΈμ‘
  welcomeBoost: LimitedTime<number>; // λΆμ€νΈ - μλ΄μ μ§κΈλλ ν μΈμΏ ν° κΈμ‘
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

// Question. μλ ValueTypeμ΄ λμνλλ‘ νμμ λ§λ€μ΄λ³΄μ.
function useLimitedTimeValue<K extends keyof PreDefined>(
  key: K,
  fallbackValue?: ValueType<K>
): ValueType<K> {
  return (null as any) as ValueType<K>;
}

const topBanner = useLimitedTimeValue("topBanner");
const saleTimer = useLimitedTimeValue("saleTimer");
type cases = [
  Expect<Equal<ValueType<"topBanner">, string>>, // μ΄λ κ² λ¬Έμ λ₯Ό μ΄ν΄ν΄λ μκ΄μμ
  Expect<Equal<typeof saleTimer, { deadline: string; interval: number }>>
];
```
