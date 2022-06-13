### Readonly 2

- 연구
  ### **Readonly 2**
  Implement a generic `MyReadonly2<T, K>` which takes two type argument `T` and `K`.
  `K` specify the set of properties of `T` that should set to Readonly. When `K` is not provided, it should make all properties readonly just like the normal `Readonly<T>`.
  For example
  ```tsx
  interface Todo {
    title: string;
    description: string;
    completed: boolean;
  }

  const todo: MyReadonly2<Todo, "title" | "description"> = {
    title: "Hey",
    description: "foobar",
    completed: false,
  };

  todo.title = "Hello"; // Error: cannot reassign a readonly property
  todo.description = "barFoo"; // Error: cannot reassign a readonly property
  todo.completed = true; // OK
  ```
  ```tsx
  // try 1
  // K를 뺴고 매핑을 해주고 & 로 readonly를 처리 하려고 했는데 왜 1번은 실패지?
  type MyReadonly2<T, K extends keyof T> = {
    [key in keyof Omit<T, K>]: T[key];
  } &
    { readonly [key in keyof Pick<T, K>]: T[key] };

  type a = MyReadonly2<Todo1, "title">;

  /* _____________ Test Cases _____________ */
  import { Alike, Expect } from "@type-challenges/utils";

  type cases = [
    Expect<Alike<MyReadonly2<Todo1>, Readonly<Todo1>>>, // 실패
    Expect<Alike<MyReadonly2<Todo1, "title" | "description">, Expected>>,
    Expect<Alike<MyReadonly2<Todo2, "title" | "description">, Expected>>
  ];

  interface Todo1 {
    title: string;
    description?: string;
    completed: boolean;
  }

  interface Todo2 {
    readonly title: string;
    description?: string;
    completed: boolean;
  }

  interface Expected {
    readonly title: string;
    readonly description?: string;
    completed: boolean;
  }
  ```
  ```tsx
  //try 2
  // 결국 정답을 참고 K의 기본값을 설정해주면 해결이 되는데 이유를 모르겠다..

  type MyReadonly2<T, K extends keyof T = keyof T> = {
    [key in keyof Omit<T, K>]: T[key];
  } &
    { readonly [key in keyof Pick<T, K>]: T[key] };

  type a = MyReadonly2<Todo1, "title">;

  /* _____________ Test Cases _____________ */
  import { Alike, Expect } from "@type-challenges/utils";

  type cases = [
    Expect<Alike<MyReadonly2<Todo1>, Readonly<Todo1>>>, // 실패
    Expect<Alike<MyReadonly2<Todo1, "title" | "description">, Expected>>,
    Expect<Alike<MyReadonly2<Todo2, "title" | "description">, Expected>>
  ];

  interface Todo1 {
    title: string;
    description?: string;
    completed: boolean;
  }

  interface Todo2 {
    readonly title: string;
    description?: string;
    completed: boolean;
  }

  interface Expected {
    readonly title: string;
    readonly description?: string;
    completed: boolean;
  }
  ```
- 현석
  & 로 조합하는 방식으로 해결할 수 있었다.
  default value 설정하는게 이 문제의 key인 것 같다.
  ```tsx
  type MyReadonly2<T, K extends keyof T = keyof T> = {
    [key in Exclude<keyof T, K>]: T[key];
  } &
    { readonly [key in K]: T[key] };
  ```
- 화랑
  첫 번째 시도: 모든 프로퍼티를 readonly로 만들어놓고, 왜 안되지? 했다. keyof T 가 T 의 “모든” UnionType 이 아니기 때문에, 해당하는 keyof T 만 readonly 가 적용되겠지 라고 착각. `Omit<T, K> &` 아이디어를 생각하지 못했다!
  ```powershell
  type MyReadonly2<T, K extends keyof T> = Omit<T, K> & {
    readonly [k in K] : T[k]
  }
  ```
  그렇다면 Type 하나만 넘겨 받는 첫 번째 테스트 코드는 어떻게 통과하지?
  type MyReadonly2<T> = { readonly [k in K] : T[k]} 처럼 좌항을 T 하나로 만드는게 의도가 아닌 것 같아 답을 보았다. :)
  ```powershell
  type MyReadonly2<T, K extends keyof T = keyof T> = Omit<T, K> & {
    readonly [k in K] : T[k]
  }
  ```
  > 테스트 코드에서 Todo1 과 Todo2 를 다르게 구성한 이유가 뭘까? 이미 readonly 프로퍼티인 것과 아닌 것에 대한 차이가 있나?
- **찬모**
  제네릭이 없다는 것을 어떻게 표현하지??
  → optional generic??
  ```tsx
  // Try1 : 틀린 풀이
  type MyReadonly2<T, K = void> = K extends keyof T
    ? {
        readonly [key in K]: T[key];
      }
    : {
        readonly [U in keyof T]: T[U];
      };
  ```
- 지선
  ```tsx
  // readonly1 구현 사항
  type MyReadonly<T> = {
  	readonly [key in keyof T] : T[key]
  }

  // K가 없으면 전체 readonly,  K와 keyof T가 겹치는 경우 readonly. 안겹칠 경우 수정 가능.
  // K optional 하게 처리 어떻게 하지?
  //->  K=void or K={} 하면 됨
  // https://stackoverflow.com/questions/37525094/optional-generic-type

  type MyReadonly2<T, K=void> =
  K extends any ?
  // K만 readonly 처리 해주기....
  :
  {readonly [key in keyof T] : T[key]}

  //K만 뽑아서 readonly 처리를 어떻게 할까...?
  //다른 답 확인 https://github.com/type-challenges/type-challenges/issues/6308
  type MyReadonly2<T, K extends keyof T = keyof T> = {
    readonly [entry in keyof T]: T[entry]
  } & Omit<T,K>

  //readonly와 그냥을 &로 하면 수정 가능한가?
  type B = {
    readonly 'title': string
    readonly 'description': string
  } & {'description': string}

  let bb :B ={
    title:'a',
    description:'a'
  }
  bb.description ='d' //수정 가능
  bb.title='1' // error! 수정 못함

  ```
- 재영
  - try1
    ```tsx
    // key를 순회하면서 특정키에만 readonly를 붙이고 싶다.
    // "특정 key"라는 조건 떄문에 extends를 쓰고 싶지만,
    // mapped type에서 key에 조건을 거는 것 (extends)은 어렵다.
    // => intersection (&) 으로 오버라이딩하자.

    type MyReadonly2<T, K> = {
      readonly [key in keyof T]: T[key];
    } &
      {
        -readonly [key in Exclude<keyof T, K>]: T[key];
      };

    // K를 빈값으로 두지말라며 에러발생
    ```
  - try2
    ```tsx
    // gereric을 optional하게 받는 방법?
    // <T, K=void>로하면 에러 뿜뿜
    // <T, K={}>로 해결.

    type MyReadonly2<T, K = {}> = {
      readonly [key in keyof T]: T[key];
    } &
      {
        -readonly [key in Exclude<keyof T, K>]: T[key];
      };
    ```
- 주운
  ```tsx
  interface Todo {
    title: string;
    description: string;
    completed: boolean;
  }

  const todo: MyReadonly2<Todo, "title" | "description"> = {
    title: "Hey",
    description: "foobar",
    completed: false,
  };

  // T로 정의되어 있는 타입에서 T에 속해있는 K는 readonly(읽기전용)
  // 그래서 todo.completed = true 만 가능
  ```
  `readonly`는 타입스크립트에서 불변성을 지키도록 해주는 키워드
  ```tsx
  // K가 주어지지 않으면 Readonly<T>
  ```
  ...gg
