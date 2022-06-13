### Includes

- 연구
  ```tsx
  // try 1
  type Includes<T extends readonly any[], U> = U extends T[number]
    ? true
    : false;
  // 타입이 정확히 as const 같은 느낌으로 추론이 안되는 것 같다 어떻게 정확하게 명시할 수 있을까?

  /* _____________ Test Cases _____________ */
  import { Equal, Expect } from "@type-challenges/utils";

  type cases = [
    Expect<
      Equal<Includes<["Kars", "Esidisi", "Wamuu", "Santana"], "Kars">, true>
    >,
    Expect<
      Equal<Includes<["Kars", "Esidisi", "Wamuu", "Santana"], "Dio">, false>
    >,
    Expect<Equal<Includes<[1, 2, 3, 5, 6, 7], 7>, true>>,
    Expect<Equal<Includes<[1, 2, 3, 5, 6, 7], 4>, false>>,
    Expect<Equal<Includes<[1, 2, 3], 2>, true>>,
    Expect<Equal<Includes<[1, 2, 3], 1>, true>>,
    Expect<Equal<Includes<[{}], { a: "A" }>, false>>, // 실패  뭔가 ObjectType 이면 다 ok
    Expect<Equal<Includes<[boolean, 2, 3, 5, 6, 7], false>, false>>, // 실패  boolean type 이면 다 ok 이런느낌?
    Expect<Equal<Includes<[true, 2, 3, 5, 6, 7], boolean>, false>>, // 실패
    Expect<Equal<Includes<[false, 2, 3, 5, 6, 7], false>, true>>,
    Expect<Equal<Includes<[{ a: "A" }], { readonly a: "A" }>, false>>, // 실패
    Expect<Equal<Includes<[{ readonly a: "A" }], { a: "A" }>, false>> // 실패
  ];
  ```
  ```tsx
  // solution
  // 한개씩 재귀돌면서 타입 확인을 하는 방법인듯..?
  type Includes<T extends readonly unknown[], U> = T extends [
    infer First,
    ...infer Rest
  ]
    ? Equal<First, U> extends true
      ? true
      : Includes<Rest, U>
    : false;

  /* _____________ Test Cases _____________ */
  import { Equal, Expect } from "@type-challenges/utils";

  type cases = [
    Expect<
      Equal<Includes<["Kars", "Esidisi", "Wamuu", "Santana"], "Kars">, true>
    >,
    Expect<
      Equal<Includes<["Kars", "Esidisi", "Wamuu", "Santana"], "Dio">, false>
    >,
    Expect<Equal<Includes<[1, 2, 3, 5, 6, 7], 7>, true>>,
    Expect<Equal<Includes<[1, 2, 3, 5, 6, 7], 4>, false>>,
    Expect<Equal<Includes<[1, 2, 3], 2>, true>>,
    Expect<Equal<Includes<[1, 2, 3], 1>, true>>,
    Expect<Equal<Includes<[{}], { a: "A" }>, false>>,
    Expect<Equal<Includes<[boolean, 2, 3, 5, 6, 7], false>, false>>,
    Expect<Equal<Includes<[true, 2, 3, 5, 6, 7], boolean>, false>>,
    Expect<Equal<Includes<[false, 2, 3, 5, 6, 7], false>, true>>,
    Expect<Equal<Includes<[{ a: "A" }], { readonly a: "A" }>, false>>,
    Expect<Equal<Includes<[{ readonly a: "A" }], { a: "A" }>, false>>
  ];
  ```
- 현석
  Try 1. 음 이런 식일 것 같긴 한데, 항상 false가 나온다.
  ```tsx
  type Includes<T extends readonly any[], U> = [U] extends T ? true : false;
  ```
  Try 2. any 때문에 항상 true가 나와버린다
  ```tsx
  type Includes<T extends readonly any[], U> = T extends [...any, U, ...any]
    ? true
    : false;
  ```
  Try 3. number index가 기억났다.
  ```tsx
  type Includes<T extends readonly any[], U> = U extends T[number]
    ? true
    : false;

  // false 가 boolean에 대입 가능하므로 true가 나와버린다.
  // Equal<Includes<[boolean, 2, 3, 5, 6, 7], false>, false>
  ```
  Try 4. 첫번째 요소부터 하나씩 검사하는 재귀?
  ```tsx
  type IsSame<T, U> = T extends U ? (U extends T ? true : false) : false;
  type Includes<T extends readonly any[], U> = T extends [infer F, ...infer K]
    ? IsSame<F, U> extends true
      ? true
      : Includes<K, U>
    : false;

  // readonly에 대한 비교에 실패한다.
  // Equal<Includes<[{ a: 'A' }], { readonly a: 'A' }>, false>
  ```
  Try 5. 꼼수
  ```tsx
  // 내가 만든 IsSame 말고 만들어진 Equal을 사용하면 통과는 한다...
  type Includes<T extends readonly any[], U> = T extends [infer F, ...infer K]
    ? Equal<F, U> extends true
      ? true
      : Includes<K, U>
    : false;
  ```
- 화랑
  - 1차 시도: `T["includes"] extends U ? true : false`
    - 흠.. includes 로는 뭘 못하겠구나.
  - T를 순회하면서, 그 친구가 U라면 true, 아니라면 false를 리턴하면 될 듯한데.. T를 순회할 수 있나? → 모르겠다
  - 답: `type Includes<T extends readonly any[], U> = T extends [infer A, ...infer B] ? (Equal<A, U> extends true ? true : Includes<B, U>) : false;`
    - 이해는 가는데.. 내부에서 Includes 를 쓰면 치트키 아닌가 🤪
  - infer에 대해 아주 조금 더 이해한 것 같다. type을 정의하는 시점에는 어떤 type인지 모르니, 특정한 이름으로 붙여놓고(ex. infer A), 런타임 시에 만약 A 가 이런 type 이면 ? 땡땡 : 똥똥.
- 찬모
  ```tsx
  // Try1
  type Includes<T extends readonly any[], U> = U extends T[keyof T]
    ? true
    : false;
  ```
  Try1의 결과로는 U가 원시타입이 아닌 경우에는 제대로 작동을 안함. 그리고 생각해보면 `T[keyof T]` 제대로 작동하지 않을거라고 생각했는데, 어느 정도 작동해서 약간 당황?! 생각해보니 아래처럼 작동하는 듯했다.
  ```tsx
  T[keyof T]

  keyof T // 배열이니까  0 | 1 | 2 ... 이런 식의 유니온 타입으로 나옴

  T[ 0 | 1 | 2]

  T[0] | T[1] | T[2] // 결국은 이것과 같은듯...
  ```
  Try1의 시도로 재귀적인 풀이를 어렴풋이 생각하게되었고, 한 번 더 조건이 필요함을 해결할 수 있을거라고 생각하게 되었다.
  ***
  ```tsx
  // Try2
  type Includes<T extends readonly any[], U> = T extends [infer A, ...infer B]
    ? U extends A
      ? true
      : Includes<B, U>
    : false;
  ```
  U extends A ?? A extends U??
  → extends 정확히 같음을 비교하는것은 아니니까 내가 원하는 연산의 결과를 보여줄 수 없는건가?
  → 위에처럼 풀면 아래의 4가지 경우를 해결할 수 없었다.
  → 실제 다른 풀이를 살펴보면 `Equal<>` 유틸타입을 이용해서 이를 풀었는데, 이 방법말고는 없는건가??
  ```tsx
  Expect<Equal<Includes<[boolean, 2, 3, 5, 6, 7], false>, false>>,
  Expect<Equal<Includes<[true, 2, 3, 5, 6, 7], boolean>, false>>,

  Expect<Equal<Includes<[{ a: 'A' }], { readonly a: 'A' }>, false>>,
  Expect<Equal<Includes<[{ readonly a: 'A' }], { a: 'A' }>, false>>,
  ```
- 지선
  ```tsx
  // idea? 재귀해서 배열의 0번째부터 n번째까지 동일한 값 있는지 비교?

  type Includes<T extends readonly any[], U> = U extends T[number]
    ? true
    : false;
  // 문자열 배열, 숫자 배열은 테스트 케이스 통과하나 다른 예제는 통과 못함(true 반환)

  //expect : false
  type a = { a: "A" } extends {} ? true : false; //true
  type b = false extends boolean ? true : false; //true 이므로

  // -> 같은지를 체크하면 될거 같은데....

  //Equal 구현사항
  // type Equal<X, Y> =
  //    (<T>() => T extends X ? 1 : 2) extends
  //        (<T>() => T extends Y ? 1 : 2) ? true : false

  //🙆‍♀️
  type Includes<T extends readonly any[], U> = T extends [
    infer First,
    ...infer Rest
  ]
    ? Equal<First, U> extends true
      ? true
      : Includes<Rest, U>
    : false;
  ```
- 재영
  - 요구사항
    - U가 T의 일부원소인지를 확인해줘
    - Equal 타입 구현이 궁금하다!
      [https://stackoverflow.com/questions/68961864/how-does-the-equals-work-in-typescript](https://stackoverflow.com/questions/68961864/how-does-the-equals-work-in-typescript)
  - Try1.
    ```tsx
    // U를 T를 유니온화 한 것에 할당할 수 있으면 true, 그렇지 않으면 false를 뱉어줘!
    type Includes<T extends readonly any[], U> = U extends T[number]
      ? true
      : false;

    // 예외케이스 발생
    type TEST1 = Includes<[{}], { a: "A" }>; // true
    type TEST2 = Includes<[boolean, 2, 3, 5, 6, 7], false>; // true
    type TEST3 = Includes<[true, 2, 3, 5, 6, 7], boolean>; // boolean 🙄🙄
    type TEST4 = Includes<[{ a: "A" }], { readonly a: "A" }>; // true
    type TEST5 = Includes<[{ readonly a: "A" }], { a: "A" }>; // true

    // 아... "할당할 수 있으면"으로 체크하는게 아니라 "동치라면"으로 체크해야한다.
    ```
  - Try2.
    ```tsx
    // key로 T를 순회해서 **[Distributive Conditional Types](https://www.typescriptlang.org/docs/handbook/2/conditional-types.html#distributive-conditional-types) 성질을 이용하려고** 뻘짓을 하다가 ...
    type Includes<T extends readonly any[], U> = {
      [key in T[number]]: key;
    };

    type ArrUnion<T extends any[]> = T[number];
    type StringArrUnion<T extends any[]> = `${T[number]}`;
    type TEST1 = ArrUnion<[boolean, 2, 3, 5, 6, 7]>; // **"boolean"** | "6" | "2" | "3" | "5" | "7"
    type TEST2 = StringArrUnion<[boolean, 2, 3, 5, 6, 7]>; // **"false" | "true"** | "6" | "2" | "3" | "5" | "7"
    ```
  - answer
    ```tsx
    // 정답봄.
    // Equal을 만들 줄 알아야겠는데 ...
    type Includes<T extends readonly any[], U> = T extends [infer F, ...infer R]
      ? Equal<F, U> extends true
        ? true
        : Includes<R, U>
      : Equal<T[0], U>;
    ```
- 주운
    <aside>
    💡 답을 어떻게 적어야할지 모르겠어서 일단 다른 사람들의 문제풀이방법을 보고 이런 식으로 적어야겠다는 흐름을 파악함
    
    </aside>
    
    ---
    
    - **다른 분들이 쓰신 답안**
        
        ```jsx
        type Includes<T extends any[], K> = T extends [infer R, ...infer U] 
        	? Equal<R, K> extends true 
        	? true 
        	: Includes<U, K> 
        	: false
        ```
        
    
    ```jsx
    type Includes<T extends readonly any[], U> = any;
    ```
    
    - T의 []에 어느 타입이든 들어갈 수 있고, U가 T의 Array에 있는 값이면 `true`, 없으면 `false`를 반환.
    
    ```jsx
    type Includes<T extends readonly any[], U> = U extends T[] ? true : false;
    ```
    
    - T의 배열에 U의 일부분이라고 하면 `true`를 반환하고, `false`를 반환하고 싶은데 위와 같은 코드를 적었을 때는 모두 `false`를 반환해서 `true`의 값인 곳에 에러를 뱉어낸다.
    
    ---
    
    ```jsx
    T extends [infer F, ...infer R]
    ```
    
    `infer`은 어떻게 쓰는걸까
