### Promise.All

- 연구
  # **Promise All**
  Type the function `PromiseAll` that accepts an array of PromiseLike objects, the returning value should be `Promise<T>` where `T` is the resolved result array.
  ```tsx
  const promise1 = Promise.resolve(3);
  const promise2 = 42;
  const promise3 = new Promise<string>((resolve, reject) => {
    setTimeout(resolve, 100, "foo");
  });

  // expected to be `Promise<[number, number, string]>`
  const p = Promise.all([promise1, promise2, promise3] as const);
  ```
  ```tsx
  //try 1 이렇게하면 내부의 promise를 풀어줄수가 없다.. 하나씩 돌려야 할 것 같다
  declare function PromiseAll<T>(values: T): Promise<T>;

  /* _____________ Test Cases _____________ */
  import { Equal, Expect } from "@type-challenges/utils";

  const promiseAllTest1 = PromiseAll([1, 2, 3] as const);
  const promiseAllTest2 = PromiseAll([1, 2, Promise.resolve(3)] as const);
  const promiseAllTest3 = PromiseAll([1, 2, Promise.resolve(3)]);

  type cases = [
    Expect<Equal<typeof promiseAllTest1, Promise<[1, 2, 3]>>>,
    Expect<Equal<typeof promiseAllTest2, Promise<[1, 2, number]>>>,
    Expect<Equal<typeof promiseAllTest3, Promise<[number, number, number]>>>
  ];
  ```
  ```tsx
  // try 2 복잡하긴하지만 여러번 이거니? 저거니? 물어봐서 벗기기 성공 하지만 3번째껀 어떻게 해줘야 할까?

  declare function PromiseAll<T>(
    values: T
  ): T extends readonly any[]
    ? Promise<
        {
          -readonly [key in keyof T]: T[key] extends Promise<infer R>
            ? R
            : T[key];
        }
      >
    : never;

  /* _____________ Test Cases _____________ */
  import { Equal, Expect } from "@type-challenges/utils";

  const promiseAllTest1 = PromiseAll([1, 2, 3] as const);
  const promiseAllTest2 = PromiseAll([1, 2, Promise.resolve(3)] as const);
  const promiseAllTest3 = PromiseAll([1, 2, Promise.resolve(3)]); // 실패

  type cases = [
    Expect<Equal<typeof promiseAllTest1, Promise<[1, 2, 3]>>>,
    Expect<Equal<typeof promiseAllTest2, Promise<[1, 2, number]>>>,
    Expect<Equal<typeof promiseAllTest3, Promise<[number, number, number]>>>
  ];
  ```
  ```tsx
  // try 3 이렇게 바꿔보니 false 배열이 된다 왜 Promise<infer U> 여기에 걸리지 않았을까? readonly가 없어 순서보장의 문제인 것 같다

  declare function PromiseAll<T>(
    values: T
  ): T extends any[]
    ? Promise<
        { [key in keyof T]: T[key] extends Promise<infer U> ? true : false }
      >
    : Promise<
        {
          -readonly [key in keyof T]: T[key] extends Promise<infer R>
            ? R
            : T[key];
        }
      >;

  /* _____________ Test Cases _____________ */
  import { Equal, Expect } from "@type-challenges/utils";

  const promiseAllTest1 = PromiseAll([1, 2, 3] as const);
  const promiseAllTest2 = PromiseAll([1, 2, Promise.resolve(3)] as const);
  const promiseAllTest3 = PromiseAll([1, 2, Promise.resolve(3)]); // Promise<false[]>

  type cases = [
    Expect<Equal<typeof promiseAllTest1, Promise<[1, 2, 3]>>>,
    Expect<Equal<typeof promiseAllTest2, Promise<[1, 2, number]>>>,
    Expect<Equal<typeof promiseAllTest3, Promise<[number, number, number]>>>
  ];
  ```
  ```tsx
  // try 4 T를 Readonly로 하고 하면 될 것 같았는데 되지 않는다. 이전에 Readonly로 바꿔줘야 하는걸까?
  declare function PromiseAll<T>(
    values: T
  ): T extends any[]
    ? Promise<
        {
          [key in keyof Readonly<T>]: T[key] extends Promise<infer U>
            ? U
            : T[key];
        }
      >
    : Promise<
        {
          -readonly [key in keyof T]: T[key] extends Promise<infer R>
            ? R
            : T[key];
        }
      >;

  /* _____________ Test Cases _____________ */
  import { Equal, Expect } from "@type-challenges/utils";

  const promiseAllTest1 = PromiseAll([1, 2, 3] as const);
  const promiseAllTest2 = PromiseAll([1, 2, Promise.resolve(3)] as const);
  const promiseAllTest3 = PromiseAll([1, 2, Promise.resolve(3)]);

  type cases = [
    Expect<Equal<typeof promiseAllTest1, Promise<[1, 2, 3]>>>,
    Expect<Equal<typeof promiseAllTest2, Promise<[1, 2, number]>>>,
    Expect<Equal<typeof promiseAllTest3, Promise<[number, number, number]>>>
  ];
  ```
  ```tsx
  // try 5 답 참고, T를 받아서 values에 readonly로 변경해주면 되는 거 였구나...

  declare function PromiseAll<T extends any[]>(
    values: readonly [...T]
  ): Promise<
    { -readonly [key in keyof T]: T[key] extends Promise<infer R> ? R : T[key] }
  >;

  /* _____________ Test Cases _____________ */
  import { Equal, Expect } from "@type-challenges/utils";

  const promiseAllTest1 = PromiseAll([1, 2, 3] as const);
  const promiseAllTest2 = PromiseAll([1, 2, Promise.resolve(3)] as const);
  const promiseAllTest3 = PromiseAll([1, 2, Promise.resolve(3)]);

  type cases = [
    Expect<Equal<typeof promiseAllTest1, Promise<[1, 2, 3]>>>,
    Expect<Equal<typeof promiseAllTest2, Promise<[1, 2, number]>>>,
    Expect<Equal<typeof promiseAllTest3, Promise<[number, number, number]>>>
  ];
  ```
- 현석
  Try 1. PromiseAll 이 function, 즉 value라 안 되네
  ```tsx
  declare function PromiseAll<T extends []>(
    values: T
  ): T extends [infer A, ...infer B]
    ? [Awaited<A>, ...ReturnType<PromiseAll<B>>]
    : never;
  ```
  Try 2. 음 순환해야 할텐데
  ```tsx
  declare function PromiseAll<T extends []>(values: T):
    T extends [infer A, ...infer B] ? [Awaited<A>, ...ReturnType<typeof PromiseAll<B>>] : never
  ```
  Answer
  ```tsx
  type AwaitAll<T extends readonly any[]> = T extends [infer T1, ...infer T2]
    ? [Awaited<T1>, ...AwaitAll<T2>]
    : T;
  declare function PromiseAll<T extends any[]>(
    values: readonly [...T]
  ): Promise<AwaitAll<T>>;
  ```
- 화랑
  ```jsx

  // 1차 시도.
  // 일단 인자에 대한 타입을 먼저 선언하고, 리턴되는 것도 Promise<저쩌고>이다.
  //근데 저쩌고도.. 인자로 전달 받은 배열의 개수와 동일하니까 map을 돌리자
  declare function PromiseAll<T extends unknown[]>(values: [...T]): Promise<{[k in keyof T] : T[k]}>

  // 2차 시도. readonly 추가
  declare function PromiseAll<T extends unknown[]>(values: readonly [...T]): Promise<{[k in keyof T] : T[k]}>

  // 3차 시도. [k in keyof T] : T[k] 이게 Promise일 때의 대응을 추가
  declare function PromiseAll<T extends unknown[]>(values: readonly [...T]): Promise<{[k in keyof T] : T[k] extends Promise<infer R> ? R : T[k] }>
  ```
- 찬모
  어떻게 접근해야할지 잘 몰라서, 여러가지 참조 자료 + 솔루션을 바탕으로 테스트해보면서 정리해보았다.
  처음에 함수(메소드)에 대한 타입 선언을 하는데 어떻게 접근해야지 라는 생각이 많이 들었다. 지금 타입을 만드는 것이기때문에 함수의 바디에 어떤 로직이 들어가는가보다, 타입이 선언되어야만 하는 부분에 초점을 맞춰서 생각해야할 듯했다. 함수에서는 인자 타입과 리턴 타입 두가지가 필요하다.
  - 어떤 인자 타입을 받아들이지?
  - 어떤 타입을 반환하지 ?
  - 제네릭 변수를 선언해서 어떻게 사용하지? <공통>
  이렇게 3가지를 생각하는 관점에서 바라보았다. 이런 말이 당연한 말이긴한데 실제로 커스텀타입을 구현하려고 보면 접근방향이나 생각해야할 부분이 무엇인지 헷갈렸다.
  1. 제네릭 변수 선언(제한) + 인자 타입 선언
  values의 값이 `array of Promiselike objects` 인데, 이를 어떻게 표현하지?
  → 우선 배열인 것만 표현, 나머지(promiselike object)는 제네릭 타입으로 정의?
  ```tsx
  declare function PromiseAll<T>(values: T): Promise<T>;

  declare function PromiseAll<T>(values: [...T]): Promise<T>;
  // error : A rest element type must be an array type.
  // → 우선 T의 타입을 제한하지 않아서. T를 사용하기 위해선 제네릭 선언(타입변수 선언)시에 먼저 제네릭 타입을 제한해줘야한다.

  declare function PromiseAll<T extends []>(values: [...T]): Promise<T>;
  // error : test case에서 들어오는 value값 들을 할당할 수 없다고 모두 error
  // → 이럴 때 가장 큰 unknown??

  declare function PromiseAll<T extends unknown[]>(values: [...T]): Promise<T>;
  // unknown 대신 any도 정상작동함
  // error : testcase 에서 error 한개 없어졌지만 그래도 error
  // → readonly 라서 안된다는데...

  declare function PromiseAll<T extends unknown[]>(
    values: readonly [...T]
  ): Promise<T>;
  // 앞에 readonly 붙여보니 오! 해결됨
  // 우선 들어가는 값들에 대한 에러는 해결됨
  ```
  1. 리턴 타입 선언
  resolved T를 반환해야한다.
  ```tsx
  // 1)
  declare function PromiseAll<T extends unknown[]>(
    values: readonly [...T]
  ): Promise<T extends Promise<infer R> ? R : T>;
  // error : T가 배열/튜플인데 그대로 반환, 배열/튜플 안의 값을 하나 하나 확인해서 반환해줘야한다.

  // 2)
  // → in 통해서 반복하면서 조건을 검사해줘야한다.
  declare function PromiseAll<T extends unknown[]>(
    values: readonly [...T]
  ): Promise<{ [P in keyof T]: T[P] extends Promise<infer R> ? R : T[P] }>;
  ```
- 지선
  ```tsx
  // promiseAll 구현
  const promise1 = Promise.resolve(3);
  const promise2 = 42;
  const promise3 = new Promise<string>((resolve, reject) => {
    setTimeout(resolve, 100, 'foo');
  });

  // expected to be `Promise<[number, number, string]>`
  const p = Promise.all([promise1, promise2, promise3] as const)
  ---------------------

  [answer 1](https://github.com/type-challenges/type-challenges/issues/7672)
  declare function PromiseAll<T extends unknown[]>(
  	values: readonly [...T]
  ): Promise<{ [P in keyof T]: T[P] extends Promise<infer R> ? R : T[P] }>;

  //참고: unknown[]와 any[] 둘다 성공
  ```
- 재영
  - `declare`는 언제쓰는건가? - 이미 존재하는 변수, 함수를 타이핑할 때. namespace와는 다르게 js로 컴파일 되지 않음.
    - `declare`is used to tell the compiler "this thing (usually a variable) exists already, and therefore can be referenced by other code, also there is no need to compile this statement into any JavaScript".
      - [https://stackoverflow.com/questions/43335962/purpose-of-declare-keyword-in-typescript](https://stackoverflow.com/questions/43335962/purpose-of-declare-keyword-in-typescript)
    - **Ambient declarations** are used to provide **static typing over existing JavaScript code**. Ambient declarations differ from regular declarations in that **no JavaScript code is emitted for them**. Instead of introducing new variables, functions, classes, enums, or namespaces, ambient declarations provide type information for entities that exist "ambiently" and are included in a program by external means, for example by referencing a JavaScript library in a <script/> tag.
      - [https://github.com/microsoft/TypeScript/blob/main/doc/spec-ARCHIVED.md#12-ambients](https://github.com/microsoft/TypeScript/blob/main/doc/spec-ARCHIVED.md#12-ambients)
    - 구현이 없는 선언을 **ambient**라고 말하며, 이 앰비언트들은 **.d.ts** 파일에 서술한다.
      - [https://kjwsx23.tistory.com/522](https://kjwsx23.tistory.com/522)
  - `typeof` 제한사항: 변수 이름같은 identifier 혹은 properties앞에서만 쓸 수 있다.
    [https://www.typescriptlang.org/docs/handbook/2/typeof-types.html#limitations](https://www.typescriptlang.org/docs/handbook/2/typeof-types.html#limitations)
  - try1.
    ```tsx
    // 이걸 잘 녹여내면 될 것 같은데 ...
    const A1 = Promise.all([1, 2, Promise.resolve(3)] as const);
    type TA1 = typeof A1; //  Promise<[1, 2, number]>
    ```
  - try2.
    ```tsx
    // 1. Promise.all 타입정의를 보다가 일단 그대로 옮겨왔더니 ... 성공했으나 ... 꼼수 ...

    declare function PromiseAll<T extends readonly unknown[] | []>(
      values: T
    ): Promise<{ -readonly [P in keyof T]: Awaited<T[P]> }>;

    // 2. 그러니까 배열로 받은 인자 [...args]를 아래와 같은 식으로 순회 가능한 것.
    declare function Test<T extends unknown[]>(
      values: T
    ): { [P in keyof T]: T[P] };

    // 3. | []의 차이...
    declare function Test<T extends readonly unknown[]>(
      values: T
    ): { [P in keyof T]: T[P] };
    // (string | number | Promise<number>)[] 🙄🙄
    const test1 = Test([1, "스트링", Promise.resolve(2)]);

    // [number, string, Promise<number>] 🙄🙄
    declare function Test<T extends readonly unknown[]>(
      values: T
    ): { [P in keyof T]: T[P] };
    const test1 = Test([1, "스트링", Promise.resolve(2)]);
    ```
