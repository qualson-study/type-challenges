- \***\*Permutation\*\***
  - 연구
    ### **Permutation**
    Implement permutation type that transforms union types into the array that includes permutations of unions.
    ```tsx
    type perm = Permutation<"A" | "B" | "C">; // ['A', 'B', 'C'] | ['A', 'C', 'B'] | ['B', 'A', 'C'] | ['B', 'C', 'A'] | ['C', 'A', 'B'] | ['C', 'B', 'A']
    ```
    ```tsx
    //try 1 간단하게 해봤는데 한개씩 배열에 담기고 있다 [A] | [B] | [C] 이렇게... 어떻게 다담을 수 있을까
    type Permutation<T> = T extends any ? [T] : never;
    ```
    ```tsx
    // try 2 결국 정답확인...

    type Permutation<T, U = T> = [T] extends [never]
      ? []
      : T extends T
      ? [T, ...Permutation<Exclude<U, T>>]
      : never;

    /* _____________ Test Cases _____________ */
    import type { Equal, Expect } from "@type-challenges/utils";

    type cases = [
      Expect<Equal<Permutation<"A">, ["A"]>>,
      Expect<
        Equal<
          Permutation<"A" | "B" | "C">,
          | ["A", "B", "C"]
          | ["A", "C", "B"]
          | ["B", "A", "C"]
          | ["B", "C", "A"]
          | ["C", "A", "B"]
          | ["C", "B", "A"]
        >
      >,
      Expect<
        Equal<
          Permutation<"B" | "A" | "C">,
          | ["A", "B", "C"]
          | ["A", "C", "B"]
          | ["B", "A", "C"]
          | ["B", "C", "A"]
          | ["C", "A", "B"]
          | ["C", "B", "A"]
        >
      >,
      Expect<Equal<Permutation<never>, []>>
    ];
    ```
  - 현석
    Try 1. Union 의 extends 는 루프를 돈다 그랬는데
    ```tsx
    type Permutation<T> = T extends infer I
      ? [I, ...Permutation<Exclude<T, I>>]
      : [];
    ```
  - 화랑
  - 찬모
  - 지선
    ```tsx
    type perm = Permutation<'A' | 'B' | 'C'>;
    // ['A', 'B', 'C'] | ['A', 'C', 'B'] | ['B', 'A', 'C'] | ['B', 'C', 'A'] | ['C', 'A', 'B'] | ['C', 'B', 'A']

    //----------------------

    // union type을 배열로 만들기?
    // union type extends union type 해서 하나씩 풀면 될까?

    //try 1
    type Permutation<T> = T extends T ?
      T extends [key in T] ? [key]:never
      : []
    type a = 'A' | 'B'
    type A = Permutation<a>

    //answer
    type Permutation<T, U = T> = [T] extends [never]
      ? []
      : (T extends U
        ? [T, ...Permutation<Exclude<U, T>>]
        : [])

    type Permutation<T, U = T> = [T] extends [never]
      ? [] //-> T=never일때 빈배열 반환
      : (T extends U // union type 하나씩 풀기
        ? [T, ...Permutation<Exclude<U, T>>]  // Exclude<'A'|'B'|'C', 'A'> = 'B'|'C'
        : [])

    ```
  - 재영
    - 순열
      ```tsx
      // 모르게따. 그냥 js로 순열 조합 구하는 것도 쉽지 않다.
      ```
