## Diff - 연구

- 연구
  ## **Type Challenge**
  [타입챌린지](https://github.com/type-challenges/type-challenges)
  ### **Diff**
  ```
  Get an Object that is the difference between O & O1
  ```
  ```tsx
  type Diff<O, O1> = {
    [K in keyof Omit<O, keyof O1> | keyof Omit<O1, keyof O>]: K extends keyof O
      ? O[K]
      : K extends keyof O1
      ? O1[K]
      : never;
  };

  import type { Equal, Expect } from "@type-challenges/utils";

  type Foo = {
    name: string;
    age: string;
  };
  type Bar = {
    name: string;
    age: string;
    gender: number;
  };
  type Coo = {
    name: string;
    gender: number;
  };

  type cases = [
    Expect<Equal<Diff<Foo, Bar>, { gender: number }>>,
    Expect<Equal<Diff<Bar, Foo>, { gender: number }>>,
    Expect<Equal<Diff<Foo, Coo>, { age: string; gender: number }>>,
    Expect<Equal<Diff<Coo, Foo>, { age: string; gender: number }>>
  ];
  ```
- 지선
  ```tsx
  // exclude해서 안겹치는거 뽑아내면 되지 않을까?\

  // try 1
  type Keys<A, B> = keyof A | keyof B
  type Merge<F extends Record<any ,any> , S> = {
    [key in Keys<F,S>] : key extends keyof S ? S[key]: F[key]
  }
  type Diff<O, O1> = {
    [key in Keys<O, O1> ] : key extends keyof Exclude<O, O1> ? never : Merge<O, O1>[key]
  }

  type test = Diff<Foo, Bar>
  //type test = {
  //    name: never;
  //    age: never;
  //    gender: number;
  //}
  -> 값이 never인 경우를 제외하면...?

  //try2 키만 Exclude해서 처리하기 👌
  type getDiffKey<O, O1> = Exclude<keyof O, keyof O1> |  Exclude<keyof O1, keyof O>

  type Diff<O, O1 extends Record<any ,any> > = {
    [key in getDiffKey<O, O1> ] : key extends keyof O ? O[key] : O1[key]
  }
  ```
- 재영
  - try1
    ```tsx
    // exclude 복습
    type T0 = Exclude<"a" | "b" | "c", "a">;
    // 추론결과 type T0 = "b" | "c"

    // 키값만 가지고 테스트
    type Foo = {
      name: string;
      age: string;
    };
    type Bar = {
      name: string;
      age: string;
      gender: number;
    };

    type KFoo = keyof Foo;
    type KBar = keyof Bar;
    type DiffKey<F, S> = Exclude<F, S> | Exclude<S, F>; // 차집합을 생각
    type Test = DiffKey<KFoo, KBar>; // "gender"

    // 결론
    type Foo = {
      name: string;
      age: string;
    };
    type Bar = {
      name: string;
      age: string;
      gender: number;
    };

    type KFoo = keyof Foo;
    type KBar = keyof Bar;
    type DiffKey<F, S> = Exclude<F, S> | Exclude<S, F>;
    type Test = DiffKey<KFoo, KBar>; // "gender"

    type BasicObject = Record<any, any>;
    type Diff<O extends BasicObject, O1 extends BasicObject> = {
      [P in DiffKey<keyof O, keyof O1>]: P extends keyof O ? O[P] : O1[P];
    };
    ```
