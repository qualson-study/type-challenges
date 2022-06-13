### Omit

- 연구
  ### **Omit**
  Implement the built-in `Omit<T, K>` generic without using it.
  Constructs a type by picking all properties from `T` and then removing `K`
  For example
  ```tsx
  interface Todo {
    title: string;
    description: string;
    completed: boolean;
  }

  type TodoPreview = MyOmit<Todo, "description" | "title">;

  const todo: TodoPreview = {
    completed: false,
  };
  ```
  ```tsx
  //try 1
  // never일게 아니라 아예 존재하지 않아야 되는데 어떻게 해야할까..

  type MyOmit<T, K> = {
    [key in keyof T]: key extends K ? never : T[key];
  };

  /* _____________ Test Cases _____________ */
  import { Equal, Expect } from "@type-challenges/utils";

  type cases = [
    Expect<Equal<Expected1, MyOmit<Todo, "description">>>,
    Expect<Equal<Expected2, MyOmit<Todo, "description" | "completed">>>
  ];

  interface Todo {
    title: string;
    description: string;
    completed: boolean;
  }

  interface Expected1 {
    title: string;
    completed: boolean;
  }

  interface Expected2 {
    title: string;
  }
  ```
  ```tsx
  //try 2
  // 결국 정답확인..
  type MyExclude<T, K> = T extends K ? never : T;

  type MyOmit<T, K> = {
    [key in keyof T as MyExclude<key, K>]: T[key];
  };

  type a = MyOmit<Todo, "title">;

  /* _____________ Test Cases _____________ */
  import { Equal, Expect } from "@type-challenges/utils";

  type cases = [
    Expect<Equal<Expected1, MyOmit<Todo, "description">>>,
    Expect<Equal<Expected2, MyOmit<Todo, "description" | "completed">>>
  ];

  interface Todo {
    title: string;
    description: string;
    completed: boolean;
  }

  interface Expected1 {
    title: string;
    completed: boolean;
  }

  interface Expected2 {
    title: string;
  }
  ```
- **현석**
  Try 1. 실패
  ```tsx
  type MyOmit<T, K extends string> = { [key in keyof T]: T[key] } & { [key in K]: never }

  // never도 타입이라. omit이라기보단 덮어쓰기로 이해해야 한다.
  { description: string } & { description: never }  => { description: never }
  ```
  Try 2.
  ```tsx
  // 아, Exclude
  type MyOmit<T, K> = { [key in Exclude<keyof T, K>]: T[key] };
  ```
- 화랑
  이전에 MyPick 을 구현 했던 걸 떠올렸다. in 다음만 바꾸면 될 것 같은데...!
  ```powershell
  type MyPick<T, K extends keyof T> = {
    [k in T] : T[k]
  }
  ```
  ```powershell
  type MyOmit<T, K extends keyof T> =  {
    [k in Exclude<keyof T, K>]: T[k]
  }
  ```
- 찬모
  ```tsx
  // Try1
  type MyOmit<T, K extends keyof T> = {
    [key in keyof T]: key extends K ? never : T[key];
  };

  interface Todo {
    title: string;
    description: string;
    completed: boolean;
  }

  type T1 = MyOmit<Todo, "description">;

  /*
  type T1 = {
      title: string;
      description: never;
      completed: boolean;
  }
  */
  ```
  1. 속성을 제거하려면, 특정 키값을 제거해야만 한다. 그렇다면, 유니온 타입에서 특정 요소만 예외 처리는 방법은 없나? 아래 이미지에서 A만 속한 것만 고를 수 있는 방법은?? → Exclude 내장 타입을 이용해야하나??
  ![스크린샷 2022-03-09 오후 8.28.15.png](%E1%84%90%E1%85%A1%E1%84%8B%E1%85%B5%E1%86%B8%20%E1%84%8E%E1%85%A2%E1%86%AF%E1%84%85%E1%85%B5%E1%86%AB%E1%84%8C%E1%85%B5%205%E1%84%92%E1%85%AC%E1%84%8E%E1%85%A1%2069d814bae1c94b068ff310a12f360c2b/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-03-09_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_8.28.15.png)
  ```tsx
  // Try2
  type MyExclude<T, U> = T extends U ? never : T;

  type MyOmit<T, K> = {
    [key in MyExclude<keyof T, K>]: T[key];
  };
  ```
  이걸 한번에 쓸 수 있는 방법은 없을까??
  ```tsx
  // sol1)
  type MyOmit<T, K> = {
    [key in keyof T as key extends K ? never : key]: T[key];
  };
  ```
  - `~~as` 라는 타입 단언을 할 때 사용하는 키워드를 통해서 key의 타입을 단언해주었다.~~
  **[MORE]** 위의 방법을 사용해서 또 다른 커스텀 타입을 만들어보자
  → `MySpecificPartial<T, U>` : T의 특정 프로퍼티(U)를 선택적으로 만드는 타입을 구성합니다.
  ```tsx
  type MySpecificPartial<T, U extends keyof T> = {
    [key in keyof T as key extends U ? key : never]?: T[key];
  } &
    {
      [key in keyof T as key extends U ? never : key]: T[key];
    };

  type T1 = MySpecificPartial<Person, "age">;
  ```
  [참고] `as` 라는 키워드가 타입 단언과 관련된거라고 생각했는데, 이게 아니라 mapped type에서 remapping에 관련한 것인거 같네요. 스터디때 말은 나왔었는데, 정확히 집고 가지 못했던거 같아서 [링크](https://www.typescriptlang.org/docs/handbook/2/mapped-types.html#key-remapping-via-as) 첨부해용 :)
- 지선
  ```tsx
  interface Todo {
    title: string
    description: string
    completed: boolean
  }

  type TodoPreview = MyOmit<Todo, 'description' | 'title'>

  const todo: TodoPreview = {
    completed: false,
  }

  ---------

  Pick 구현 활용
  K자리는 T의 key들만 들어가야 한다.
  K extends keyof T

  // T의 key를 하나씩 K에 extends 되는지 비교해서 extends되는 key는 반환?
  type MyOmit<T, K extends keyof T> = {
    [key in keyof T] : key extends K ? never: T[key]
  }

  type A =  MyOmit<Todo, 'description'>
  //expect
  type A = {
    title: string
    completed: boolean
  }
  // result
  type A = {
    title: string
  	description: never
    completed: boolean
  }

  //keyof T와 K에서 겹치는 부분 제외 해서 [key in (제외한 값)] : T[key] 하면?
  // exclude 이용?

  //🙆‍♀️
  type MyExclude<T, U> = T extends U ? never : T

  type MyOmit<T, K extends keyof T> = {
    [key in MyExclude<keyof T, K>]: T[key]
  }

  ```
- 재영
  - try1
    ```tsx
    // TS가 key를 정하는 과정을 못알아 듣고 에러 뿜뿜
    type MyOmit<T, K extends keyof T> = {
      [(key extends any ? key : key) in keyof T] : key extends K ? never : T[key]
    }

    // [key~ ] 안에서 extends하기는 어려운 것 같다.

    ```
  - 공식문서
    ```tsx
    // 공식문서 🙏
    // You can filter out keys by producing never via a conditional type:
    // Remove the 'kind' property
    type RemoveKindField<Type> = {
      [Property in keyof Type as Exclude<Property, "kind">]: Type[Property];
    };

    interface Circle {
      kind: "circle";
      radius: number;
    }

    type KindlessCircle = RemoveKindField<Circle>;
    ```
  - try2
    ```tsx
    type ExcludeTest = Exclude<1 | 2, 1>; // 2... 즉 Exclude<A, B>는 A-B

    // 👍
    type MyOmit<T, K> = {
      [key in keyof T as Exclude<key, K>]: T[key];
    };

    // 👍👍 as 안쓰는게 읽기 더 편한 것 같다.
    type MyOmit<T, K> = {
      [key in Exclude<keyof T, K>]: T[key];
    };

    // 👍👍👍 Pick을 써주면 key in~ 이런거 안써도된다.
    type MyOmit<T, K> = Pick<T, Exclude<keyof T, K>>;
    ```
- 주운
  ```tsx
  // 해당 키 값을 제외한 interface 구성
  // K는 T의 키로 한정

  type MyOmit<T, K extends keyof T> = {};

  type cases = [
    Expect<Equal<Expected1, MyOmit<Todo, "description">>>,
    Expect<Equal<Expected2, MyOmit<Todo, "description" | "completed">>>
  ];

  // K 값을 제외하고 어떻게 할 수 있을까...
  ```
  [https://blog.martinwork.co.kr/typescript/2019/05/28/typescript-util-types.html](https://blog.martinwork.co.kr/typescript/2019/05/28/typescript-util-types.html)
  **구글검색:** 해당 키 값을 제외 타입스크립트
  ```tsx
  type MyOmit<T, K extends keyof T> = Pick<T, Exclude<keyof T, K>>;
  ```
  `Exclude`를 통해 해당 하는 키 값을 제거
