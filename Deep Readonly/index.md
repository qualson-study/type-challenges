### \***\*Deep Readonly\*\***

- 연구
  ### **Deep Readonly**
  Implement a generic `DeepReadonly<T>` which make every parameter of an object - and its sub-objects recursively - readonly.
  You can assume that we are only dealing with Objects in this challenge. Arrays, Functions, Classes and so on are no need to take into consideration. However, you can still challenge your self by covering different cases as many as possible.
  For example
  ```tsx
  type X = {
    x: {
      a: 1;
      b: "hi";
    };
    y: "hey";
  };

  type Expected = {
    readonly x: {
      readonly a: 1;
      readonly b: "hi";
    };
    readonly y: "hey";
  };

  type Todo = DeepReadonly<X>; // should be same as `Expected`
  ```
  ```tsx
  // try 1 객체면 재귀를 돌리려했더니 함수도 객체인데..?! 그렇다면 함수와 객체를 어떻게 구분할까?
  type DeepReadonly<T> = {
    readonly [key in keyof T]: T[key] extends Object
      ? DeepReadonly<T[key]>
      : T[key];
  };

  /* _____________ Test Cases _____________ */
  import { Equal, Expect } from "@type-challenges/utils";

  type cases = [Expect<Equal<DeepReadonly<X>, Expected>>];

  type X = {
    a: () => 22;
    b: string;
    c: {
      d: boolean;
      e: {
        g: {
          h: {
            i: true;
            j: "string";
          };
          k: "hello";
        };
      };
    };
  };

  type Expected = {
    readonly a: () => 22;
    readonly b: string;
    readonly c: {
      readonly d: boolean;
      readonly e: {
        readonly g: {
          readonly h: {
            readonly i: true;
            readonly j: "string";
          };
          readonly k: "hello";
        };
      };
    };
  };
  ```
  ```tsx
  // try2 객체를 확인하려 했으나 실패 ...

  type DeepReadonly<T> = {
    readonly [key in keyof T]: T[key] extends { [key: string]: any }
      ? DeepReadonly<T[key]>
      : T[key];
  };

  /* _____________ Test Cases _____________ */
  import { Equal, Expect } from "@type-challenges/utils";

  type cases = [Expect<Equal<DeepReadonly<X>, Expected>>];

  type X = {
    a: () => 22;
    b: string;
    c: {
      d: boolean;
      e: {
        g: {
          h: {
            i: true;
            j: "string";
          };
          k: "hello";
        };
      };
    };
  };

  type Expected = {
    readonly a: () => 22;
    readonly b: string;
    readonly c: {
      readonly d: boolean;
      readonly e: {
        readonly g: {
          readonly h: {
            readonly i: true;
            readonly j: "string";
          };
          readonly k: "hello";
        };
      };
    };
  };
  ```
  ```tsx
  //try 3 결국 함수를 한번 더 걸러줘서 성공

  type DeepReadonly<T> = {
    readonly [key in keyof T]: T[key] extends Object
      ? T[key] extends Function
        ? T[key]
        : DeepReadonly<T[key]>
      : T[key];
  };

  /* _____________ Test Cases _____________ */
  import { Equal, Expect } from "@type-challenges/utils";

  type cases = [Expect<Equal<DeepReadonly<X>, Expected>>];

  type X = {
    a: () => 22;
    b: string;
    c: {
      d: boolean;
      e: {
        g: {
          h: {
            i: true;
            j: "string";
          };
          k: "hello";
        };
      };
    };
  };

  type Expected = {
    readonly a: () => 22;
    readonly b: string;
    readonly c: {
      readonly d: boolean;
      readonly e: {
        readonly g: {
          readonly h: {
            readonly i: true;
            readonly j: "string";
          };
          readonly k: "hello";
        };
      };
    };
  };
  ```
  ```tsx
  // 의문점 발견 왜 any는 안되고 unknown은 되지?

  type DeepReadonly<T> = {
    readonly [key in keyof T]: T[key] extends { [key: string]: unknown }
      ? DeepReadonly<T[key]>
      : T[key];
  };

  /* _____________ Test Cases _____________ */
  import { Equal, Expect } from "@type-challenges/utils";

  type cases = [Expect<Equal<DeepReadonly<X>, Expected>>];

  type X = {
    a: () => 22;
    b: string;
    c: {
      d: boolean;
      e: {
        g: {
          h: {
            i: true;
            j: "string";
          };
          k: "hello";
        };
      };
    };
  };

  type Expected = {
    readonly a: () => 22;
    readonly b: string;
    readonly c: {
      readonly d: boolean;
      readonly e: {
        readonly g: {
          readonly h: {
            readonly i: true;
            readonly j: "string";
          };
          readonly k: "hello";
        };
      };
    };
  };
  ```
- 현석
  Try 1. 이런 방식인건 알겠는데, object가 아닐 때도 돌아가야 한다....
  ```tsx
  type DeepReadonly<T> = {
    readonly [key in keyof T]: DeepReadonly<T[key]>;
  };
  ```
  Try 2. 이게 성공하긴 했는데, 영 찜찜하다
  ```tsx
  type DeepReadonly<T> = T extends string | boolean | number | Function
    ? T
    : {
        readonly [key in keyof T]: DeepReadonly<T[key]>;
      };
  ```
- 화랑
- 찬모
- 지선
  ```tsx
  readonly에서 T가 object면 재귀로 DeepReadonly를 부르면 되지 않을까?

  // readonly
  type Readonly<T> = {
    readonly [key in keyof T] : T[key]
  }

  //try 1 ❌
  type DeepReadonly<T> = {
    readonly [key in keyof T] : T[key] extends object? DeepReadonly<T[key]> : T[key]
  }
  // -> DeepReadonly로 표시됨 & 함수도 object
  // 함수인 a 주석처리하면 테스트 케이스 통과하긴 함

  //다른 답 확인
  type DeepReadonly<T> = T extends Record<string, unknown> ? {readonly[Key in keyof T]: DeepReadonly<T[Key]>} : T;
  type DeepReadonly<T> = T extends {[k: string]: {}} ?
  {readonly [k in keyof T]: DeepReadonly<T[k]> }
  : T;

  //Record
  type T = Record<string, unknown>
  // => { [x: string]: unknown }
  ```
  [**Record< K, T>**](https://typescript-kr.github.io/pages/utility-types.html#recordkt) : 타입 T의 프로프터의 집합 K로 타입을 구성한다. 타입의 프로퍼티들을 다른 타입에 매핑시킬 때 사용
- 재영
  - try1
    ```tsx
    // 왜 안되는겨~~!
    type DeepReadonly<T> = {
      readonly [key in keyof T]: DeepReadonly<T[key]>;
    };
    ```
  - try2
    ```tsx
    type DeepReadonly<T> = T extends {}
      ? {
          readonly [key in keyof T]: DeepReadonly<T[key]>;
        }
      : T;
    ```
  - try3
    ```tsx
    type DeepReadonly<T> = T extends {}
      ? {
          readonly [key in keyof T]: DeepReadonly<T[key]>;
        }
      : T;

    // extends { } 를 수정하니까 해결됨.
    // 우리가 포현하고자 하는, key와 value로 이루어진 object에 할당가능함을 표현하기 위해서는,
    // extends {} 는 사용하지 않는게 좋겠다.

    // 👍👍
    type DeepReadonly<T> = T extends Record<string, unknown>
      ? {
          readonly [key in keyof T]: DeepReadonly<T[key]>;
        }
      : T;

    // 아니 근데 Record<string, any>로 하면 왜 안되는거지 ??~~
    type DeepReadonly<T> = T extends Record<string, any>
      ? {
          readonly [key in keyof T]: DeepReadonly<T[key]>;
        }
      : T;
    ```

????? any vs unknown in **extends** blabla
