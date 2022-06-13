### \***\*Chainable Options\*\***

- 연구
  ### **Chainable Options**
  Chainable options are commonly used in Javascript. But when we switch to TypeScript, can you properly type it?
  In this challenge, you need to type an object or a class - whatever you like - to provide two function `option(key, value)` and `get()`. In `option`, you can extend the current config type by the given key and value. We should about to access the final result via `get`.
  For example
  ```tsx
  declare const config: Chainable;

  const result = config
    .option("foo", 123)
    .option("name", "type-challenges")
    .option("bar", { value: "Hello World" })
    .get();

  // expect the type of result to be:
  interface Result {
    foo: number;
    name: string;
    bar: {
      value: string;
    };
  }
  ```
  ```tsx
  // try 1 Chaining 을 하려면 기본적으로 같은 타입을 리턴해줘야 하겠지..? R[T] === K 면 있는거니까 아래처럼 해봤는데 실패..
  type Chainable<R = {}> = {
    option<T extends keyof R, K>(
      key: K extends R[T] ? never : T,
      value: K
    ): Chainable<{ [key in T]: K }>;
    get(): any;
  };

  /* _____________ Test Cases _____________ */
  import { Alike, Expect } from "@type-challenges/utils";

  declare const a: Chainable;

  const result1 = a
    .option("foo", 123)
    .option("bar", { value: "Hello World" })
    .option("name", "type-challenges")
    .get();

  const result2 = a
    .option("name", "another name") // @ts-expect-error
    .option("name", "last name")
    .get();

  type cases = [
    Expect<Alike<typeof result1, Expected1>>,
    Expect<Alike<typeof result2, Expected2>>
  ];

  type Expected1 = {
    foo: number;
    bar: {
      value: string;
    };
    name: string;
  };

  type Expected2 = {
    name: string;
  };
  ```
- 현석
  Try 1. Same `key` won't be passed twice... Exclude?
  ```tsx
  type Chainable<Value = Map<never, any>> = {
    option<K extends Exclude<string, keyof Value>, V>(
      key: K,
      value: V
    ): Chainable<Map<K | keyof Value, any>>;
    get(): any;
  };
  ```
  Try 2. 아 거의 다 온거 같은데. Exclude는 소용이 없다...;
  ```tsx
  type Chainable<Value = Map<string, any>> = {
    option<K extends Exclude<string, keyof Value>, V>(
      key: K,
      value: V
    ): Chainable<Map<K, V> & Value>;
    get(): {
      [key in keyof Value]: Value[key];
    };
  };
  ```
  Try 3. 일단 테스트는 패스. 에러케이스 필요
  ```tsx
  type Chainable<Value = {}> = {
    option<K extends Exclude<string, keyof Value>, V>(
      key: K,
      value: V
    ): Chainable<
      { [key in keyof Value | K]: key extends keyof Value ? Value[key] : V }
    >;
    get(): Value;
  };
  ```
  Try 4. COMPLETE
  ```tsx
  type Merge<Prev, K extends string, V> = {
    [key in keyof Prev | K]: key extends keyof Prev ? Prev[key] : V;
  };
  type Chainable<Value = {}> = {
    option<K extends string, V>(
      key: Exclude<K, keyof Value>,
      value: V
    ): Chainable<Merge<Value, K, V>>;
    get(): Value;
  };
  ```
- 화랑
- 찬모
- 지선
  ```tsx
  // 완전 모르겠다...

  //answer 1
  type Chainable<T = {}> = {
    option<K extends string, V>(key: K, value: V): Chainable<T & {[key in K]: V}>
    get(): { [K in keyof T]: T[K] }
  	// or get(): T로 한 답도 봤으나 이 경우 typeof result1이 &로 연결됨
  }
  -> result2의 키 중복 에러 조건 만족하지 않음

  //answer2
  type Chainable = {
    option<T, K extends string, V>(this: T, key: Exclude<K, keyof T>, value: V): T & { [k in K]: V }
    get<T>(this: T): Omit<T, 'option' | 'get'>;
  }

  ```
  - [https://ronnieschaniel.com/typescript/type-chainable-options/](https://ronnieschaniel.com/typescript/type-chainable-options/)
  - [https://ghaiklor.github.io/type-challenges-solutions/en/medium-chainable-options.html](https://ghaiklor.github.io/type-challenges-solutions/en/medium-chainable-options.html)
- 재영
  - try1
    ```tsx
    // 음? 못 알아듣게 했구나 ...
    type Chainable = {
      option(key: K, value: V): { [P in K]: V };
      get(): any;
    };
    ```
  - 제네릭이 읽히는 방향을 다시 보자 - 타입인수추론 (_type argument inference)_
    [https://www.typescriptlang.org/docs/handbook/2/generics.html#hello-world-of-generics](https://www.typescriptlang.org/docs/handbook/2/generics.html#hello-world-of-generics)
    ```tsx
    function identity<T>(arg: T): T {
      return arg;
    }

    let output = identity<string>("myString"); // output: string

    // 타입 인수를 꺾쇠괄호(<>)에 담아 명시적으로 전달해 주지 않은 것을 주목하세요 😊
    // 컴파일러는 값인 "myString"를 보고 그것의 타입으로 Type를 정합니다.
    // 인수 추론은 코드를 간결하고 가독성 있게 하는 데 있어 유용하지만
    // 더 복잡한 예제에서 컴파일러가 타입을 유추할 수 없는 경우엔
    // 명시적인 타입 인수 전달이 필요할 수도 있습니다.
    let output = identity("myString");
    ```
  - try2
    ```tsx
    type Chainable = {
      // 반환값에서 다시 option 메소드를 사용하려면, Chainable 타입을 반환하는걸로 수정하자.
      // 그러면서 이전 메소드 체이닝의 결과인 {[P in K]: V}를 어딘가에 축적해야 한다.
      // => Chainable<R={}>
      // get()으로 축적된 결과인 R을 반환하도록 수정하자.

      option<K extends string, V>(key: K, value: V): { [P in K]: V };
      get(): any;
    };
    ```
  - try3
    ```tsx
    type Chainable<R = {}> = {
      option<K extends string, V>(
        key: K,
        value: V
      ): Chainable<R & { [P in K]: V }>;
      get(): R;
    };
    ```
  - try4
    ```tsx
    const result2 = a
      .option("name", "another name")
      // @ts-expect-error << 오버라이딩을 허용하지 않고 싶나보다.
      .option("name", "last name")
      .get();

    // 이미 option의 key로 썼던 string 값은 다시 option의 key로 사용하지 못하게 제한하자
    ```
  - try5
    ```tsx
    // 어 왜 계속 @ts-expect-error가 발생하지 않지?
    type Chainable<R={}> = {
      option<K extends **Exclude<string,keyof R>**, V>(key: K, value: V) : Chainable<R & {[P in K]: V}>
      get(): R
    }
    ```
  - try6
    ```tsx
    // 이렇게 하니까 되네?? 무슨차이지 ???
    type Chainable<R = {}> = {
      option<K extends string, V>(
        key: Exclude<K, keyof R>,
        value: V
      ): Chainable<R & { [P in K]: V }>;
      get(): R;
    };

    // 이렇게 key에 never를 할당해서 오버라이딩을 무효화해도 됨 👍👍
    type Chainable<R = {}> = {
      option<K extends string, V>(
        key: K extends keyof R ? never : K,
        value: V
      ): Chainable<R & { [P in K]: V }>;
      get(): R;
    };
    ```
  - 정답참고
    [https://github.com/ghaiklor/type-challenges-solutions/blob/main/ko/medium-chainable-options.md](https://github.com/ghaiklor/type-challenges-solutions/blob/main/ko/medium-chainable-options.md)

```tsx
실제로 비슷한 타입을 구현할 때에는 보통 이런 식으로 구현한다
다만 this.value의 타입이 추적되지 않는다.

class Chainable {
  value = {}
  constructor(value: {[key: string]: any} = {}) {
    this.value = value
  }

  option(key: string, value: any) {
    if (key in this.value) {
      throw Error()
    }
    return new Chainable({...this.value, [key]: value})
  }

  get() {
    return this.value
  }
}
```
