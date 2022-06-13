## Lookup

- 연구
  ### **Look up**
  Sometimes, you may want to lookup for a type in a union to by their attributes.
  In this challenge, we would like to get the corresponding type by searching for the common `type` field in the union `Cat | Dog`. In other words, we will expect to get `Dog` for `LookUp<Dog | Cat, 'dog'>` and `Cat` for `LookUp<Dog | Cat, 'cat'>` in the following example.
  ```tsx
  interface Cat {
    type: "cat";
    breeds: "Abyssinian" | "Shorthair" | "Curl" | "Bengal";
  }

  interface Dog {
    type: "dog";
    breeds: "Hound" | "Brittany" | "Bulldog" | "Boxer";
    color: "brown" | "white" | "black";
  }

  type MyDogType = LookUp<Cat | Dog, "dog">; // expected to be `Dog`
  ```
  ```tsx
  //try 1

  type LookUp<U, T> = U extends { type: T } ? U : never;

  /* _____________ Test Cases _____________ */
  import { Equal, Expect } from "@type-challenges/utils";

  interface Cat {
    type: "cat";
    breeds: "Abyssinian" | "Shorthair" | "Curl" | "Bengal";
  }

  interface Dog {
    type: "dog";
    breeds: "Hound" | "Brittany" | "Bulldog" | "Boxer";
    color: "brown" | "white" | "black";
  }

  type Animal = Cat | Dog;

  type cases = [
    Expect<Equal<LookUp<Animal, "dog">, Dog>>,
    Expect<Equal<LookUp<Animal, "cat">, Cat>>
  ];
  ```
- 현석
  Try 1. 흠
  ```tsx
  type LookUp<
    U extends { type: string },
    T extends string
  > = T extends U["type"] ? U : never;
  ```
  Try 2. 어?
  ```tsx
  type LookUp<U extends  {type: string}, T> = U & {type: T}

  LookUp<Cat | Dog, 'dog'> == Dog & { type: 'dog' }
  ```
  Try 3.
  ```tsx

  ```
- 화랑
- 찬모
  ```tsx
  type LookUp<U, T> = U extends { type: T } ? U : never;
  ```
  여러 타입 중에서 특정 속성을 통해서 타입을 구별해낼 때 사용하는 방법을 \***\*Discriminated Union\*\*** 이라고 한다. (위의 커스텀 타입을 만들면서 discriminated union 방식을 커스텀 타입을 만드는 것 아닌가 라는 생각이 들어서 개념적인 내용을 첨부해봅니다.) [참고](https://www.typescriptlang.org/docs/handbook/2/narrowing.html#discriminated-unions)
  ```tsx
  // example
  interface Circle {
    kind: "circle";
    radius: number;
  }

  interface Square {
    kind: "square";
    sideLength: number;
  }

  type Shape = Circle | Square;

  function getArea(shape: Shape) {
    switch (shape.kind) {
      case "circle":
        return Math.PI * shape.radius ** 2;
      case "square":
        return shape.sideLength ** 2;
    }
  }
  ```
- 지선
  ```tsx
  interface Cat {
    type: 'cat'
    breeds: 'Abyssinian' | 'Shorthair' | 'Curl' | 'Bengal'
  }

  interface Dog {
    type: 'dog'
    breeds: 'Hound' | 'Brittany' | 'Bulldog' | 'Boxer'
    color: 'brown' | 'white' | 'black'
  }

  type MyDogType = LookUp<Cat | Dog, 'dog'> // expected to be `Dog`

  -----------

  // Union type 분할해서 체크할 수 있는 방법은...?

  //try1
  type LookUp<U, T> = U extends (infer A)|(infer B) ? A['type'] extends T ? A : B :never
  //Type '"type"' cannot be used to index type 'A'

  //try2
  type LookUp<U, T> = U extends (infer A)|(infer B) ? T extends A ? A : B :never
  //테스트 케이스 통과 못함
  type A = LookUp<Cat | Dog, 'dog'>
  //type A = Cat | Dog

  //[answer](https://github.com/type-challenges/type-challenges/issues/8018)
  type LookUp<U, T> =  U extends {type:T} ? U : never

  // [U['type']을 사용하는 방법](https://github.com/type-challenges/type-challenges/issues/7154)
  interface Thing {
    type: string
    [key: string]: any
  }

  type LookUp<U extends Thing, T extends string> =
    U extends U
      ? U['type'] extends T
        ? U
        : never
      : never

  // type외에 breeds나 color가 되는 답 [[링크]](https://github.com/type-challenges/type-challenges/issues/7942)
  type LookUp<U extends object, T extends string, P = U> = U extends U
    ? [U] extends [P]
      ? T extends U[keyof U]
        ? U
        : never
      : never
    : never;
  // type A= LookUp<Animal, 'Hound'>
  // type A = Dog
  ```
- 재영

  - try1
    ```tsx
    // union을 extends와 함께 쓰면 distributive가 되었던 것 같은데 ...
    type LookUp<U extends Animal, T> = U["type"] extends T ? U : never;

    // 그냥 never가 나오네 ...
    type TEST = LookUp<Animal, "dog">;
    ```
  - try2
    ```tsx
    // Type 'U' is not assignable to type 'string | number | symbol'.
    type LookUp<U extends Animal, T> = {
      [P in U]: P extends T ? U : never;
    };
    ```
  - try3
    ```tsx
    // Utility Types 중에 Exclue나 Extract를 쓰면 좋을 것 같은데 ...
    // Extract<T, U> 는 U에 할당 가능한 T만 뽑는다. 즉 U(type이 'dog'인)에 할당 가능한 Animals를 뽑아낸다면 ??

    // 이얏호
    type LookUp<U extends Animal, T> = Extract<U, { type: T }>;
    ```

- 확장한 LookUp
  ```tsx
  type LookUp<U, T extends string> = U extends any
    ? T extends U[keyof U]
      ? U
      : never
    : never;
  ```
