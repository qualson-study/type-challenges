## Merge - 재영

- 연구
  ## **Type Challenge**
  [타입챌린지](https://github.com/type-challenges/type-challenges)
  ### **Merge**
  Merge two types into a new type. Keys of the second type overrides keys of the first type.
  For example
  ```tsx
  type foo = {
    name: string;
    age: string;
  };
  type coo = {
    age: number;
    sex: string;
  };

  type Result = Merge<foo, coo>; // expected to be {name: string, age: number, sex: string}
  ```
  ```tsx
  // 오랜만에해서 감 잡을 겸 답을 봐버림...
  type Merge<F, S> = {
    [key in keyof F | keyof S]: key extends keyof S
      ? S[key]
      : key extends keyof F
      ? F[key]
      : never;
  };

  /* _____________ Test Cases _____________ */
  import type { Equal, Expect } from "@type-challenges/utils";

  type Foo = {
    a: number;
    b: string;
  };
  type Bar = {
    b: number;
    c: boolean;
  };

  type cases = [
    Expect<
      Equal<
        Merge<Foo, Bar>,
        {
          a: number;
          b: number;
          c: boolean;
        }
      >
    >
  ];
  ```
- 현석
- 화랑
- 찬모
  ```tsx
  // try1
  type Merge<F, S> = {
    [K in keyof F]: F[K];
  } &
    {
      [T in keyof S]: S[T];
    };

  //역시 지금까지의 경험으로 비추어보면 이렇게 하면 안된다. Equal 통과 못함
  ```
  ```tsx
  // try2
  // → F key와 S key를 유니온으로 모아서 map을 돌리는 방법
  type Merge<F, S> = {
    [K in keyof F | keyof S]: S[K] extends never ? F[K] : S[K];
  };

  // → 이 때 중복되는 값은 S로 오버라이딩된다고 했으니 S부터 검사해줘야한다.
  // → error : Type 'K' cannot be used to index type 'S'
  ```
  ```tsx
  // try3 😎
  // → 그래서 타입체크를 다 해줬더니 통과
  type Merge<F, S> = {
    [K in keyof F | keyof S]: K extends keyof S
      ? S[K]
      : K extends keyof F
      ? F[K]
      : never;
  };
  ```
- 지선
  ```tsx
  type foo = {
    name: string;
    age: string;
  }
  type coo = {
    age: number;
    sex: string
  }

  type Result = Merge<foo,coo>; // expected to be {name: string, age: number, sex: string}

  ///////

  // append of object처럼 두개 obejct키를 union해서 사용하면?

  // try 1
  type Keys<A, B> = keyof A | keyof B
  type Merge<F , S> = {
    [key in Keys<F,S>] : key extends keyof S ? S[key]: F[key]
  }
  // 테스트 케이스는 통과
  Type 'key' cannot be used to index type 'F'

  //try 2 👌
  type Keys<A, B> = keyof A | keyof B
  type Merge<F extends Record<any ,any> , S> = {
    [key in Keys<F,S>] : key extends keyof S ? S[key]: F[key]
  }
  ```
- 재영
  - try1
    ```tsx
    type Merge<F extends Record<string, any>, S extends Record<string, any>> = {
      [P in keyof (F | S)]: P extends keyof F ? F[P] : S[P];
    };

    type TEST = Merge<Foo, Bar>;
    // 추론결과
    // type TEST = {
    //     b: string;
    // }
    ```
  - try2
    ```tsx
    type Merge<F extends Record<string, any>, S extends Record<string, any>> = {
      [P in keyof F | keyof S]: P extends keyof S
        ? S[P]
        : P extends keyof F
        ? F[P]
        : never;
    };

    // try1, 2의 차이 => keyof F | keyof S 와 keyof (F | S)의 차이

    type Bird = {
      id: string;
      wings: number;
    };
    type Dog = {
      id: number;
      legs: number;
    };

    type Keys1<F, S> = keyof F | keyof S;
    type Keys2<F, S> = keyof (F | S);

    type Test1 = Keys1<Bird, Dog>; // "id" | "wings" | "legs"
    type Test2 = Keys2<Bird, Dog>; // "id"

    // keyof를 사용할 때 | 와 &의 차이

    type Wow1 = keyof (
      | {
          // "id"
          id: string;
          wings: number;
        }
      | {
          id: number;
          legs: number;
        }
    );

    type Wow2 = keyof ({
      // "id" | "wings" | "legs"
      id: string;
      wings: number;
    } & {
      id: number;
      legs: number;
    });
    ```
  - Union vs Intersection 비교 - 이펙티브 타입스크립트 p.41
    - Union ( 연산자 | )은 값 집합들의 합집합이다.
    - Intersection ( 연산자 & )은 값 집합들의 교집합이다.
    ```tsx

    type Person = {
    	name: string
    }

    type Lifespan = {
    	birth: Date
    	death?: Date
    }

    type PersonSpan = Person & Lifespan  // never???

    // PersonSpan은 공집합 아닌가? -> 아니다 ~~~~
    // 타입 연산자는 인터페이스의 속성이 아닌, 값의 집합 (타입의 범위)에 적용된다.

    const ps: PersonSpan = {
      name: 'jae young',
      birth: new Date('1994/01/17'),
      death: new Date('2094/01/17')
    } // 타입 에러 없이 정상.

    type K = keyof (Person | Lifespan) // 타입이 never !!!

    // 아래와 같이 정리 가능... 드모르간 법칙이랑 비스무리하다.
    keyof (A & B) = (keyof A) | (keyof B)
    keyof (A | B) = (keyof A) & (keyof B)

    ```
