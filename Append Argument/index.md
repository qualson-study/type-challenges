- \***\*Append Argument\*\***
  - 연구
    ### **Append Argument**
    For given function type `Fn`, and any type `A` (any in this context means we don't restrict the type, and I don't have in mind any type 😉) create a generic type which will take `Fn` as the first argument, `A` as the second, and will produce function type `G` which will be the same as `Fn` but with appended argument `A` as a last one.
    For example,
    ```tsx
    type Fn = (a: number, b: string) => number;

    type Result = AppendArgument<Fn, boolean>;
    // expected be (a: number, b: string, x: boolean) => number
    ```
    ```tsx
    // try 1 K를 가져오고 싶은데 가져올 수가 없다.. 어떻게 arg를 가져올 수 있을까?
    type AppendArgument<Fn extends (...arg: infer K) => any, A extends any> = (
      ...K,
      x: A
    ) => ReturnType<Fn>;
    ```
    ```tsx
    // try 2  왜 K를 저기 그냥 풀고 싶은데 풀 수가 없을까...!

    type AppendArgument<
      Fn extends (...arg: any[]) => any,
      A extends any
    > = Fn extends (...args: infer K) => any
      ? (...K, x: A) => ReturnType<Fn>
      : never;
    ```
    ```tsx
    // try 3 args에 타입을 넣어주고 ...args 해서 해결 !
    type AppendArgument<
      Fn extends (...arg: any[]) => any,
      A extends any
    > = Fn extends (...args: infer K) => any
      ? (...arg: [...K, A]) => ReturnType<Fn>
      : never;

    /* _____________ Test Cases _____________ */
    import type { Equal, Expect } from "@type-challenges/utils";

    type Case1 = AppendArgument<(a: number, b: string) => number, boolean>;
    type Result1 = (a: number, b: string, x: boolean) => number;

    type Case2 = AppendArgument<() => void, undefined>;
    type Result2 = (x: undefined) => void;

    type cases = [Expect<Equal<Case1, Result1>>, Expect<Equal<Case2, Result2>>];
    ```
  - 현석
    Try 1.
    ```tsx
    type AppendArgument<Fn extends (...args: any[]) => any, A> = Fn extends (...args: infer Args) => infer Return ? (args: ...Args, another: A) => Return: never
    ```
    Try 2. Success
    ```tsx
    type AppendArgument<Fn extends (...args: any[]) => any, A> = Fn extends (
      ...args: infer Args
    ) => infer Return
      ? (...args: [...Args, A]) => Return
      : never;
    ```
  - 화랑
  - 찬모
    ```tsx
    // try1 🤔
    type AppendArgument<Fn extends (...args: any[]) => any, A> = ( ...Parameters<Fn>, A )😩 => ReturnType<Fn>

    // try2 😇
    type AppendArgument<Fn extends (...args: any[]) => any, A> = ( ...args: [...Parameters<Fn>, A ] ) => ReturnType<Fn>
    ```
    먼저, 함수의 인자에 대한 타입을 가져오는 방법에 대해서 고민을 했다. 검색을 통해서 `Parameters`라는 내장타입이 존재함을 알게되었다. 그리고 기존의 ReturnType을 알기때문에 이 둘을 조합하면 해결할 수 있을거라고 생각하여 접근하였다.
    **try1**은 함수를 타입으로 표현하는 syntax에 익숙하지 못해서 나온 결과인 것 같다.
    아래는 실제로 구현된 Parameters 내장 타입 코드이다.
    ```tsx
    type Parameters<T extends (...args: any[]) => any> = T extends (
      ...args: infer P
    ) => any
      ? P
      : never;
    ```
  - 지선
    ```tsx
    type Fn = (a: number, b: string) => number;

    type Result = AppendArgument<Fn, boolean>;
    // expected be (a: number, b: string, x: boolean) => number

    //----------------------

    // try
    // returnType, argumentType 각각 구해서 함수로 만들기
    type ReturnType<F extends Function> = F extends (...args: any) => infer R
      ? R
      : never;
    type ArgumentType<F extends Function> = F extends (...args: infer A) => any
      ? A
      : never;

    type AppendArgument<
      Fn extends Function,
      A,
      Args = ArgumentType<Fn>,
      R = ReturnType<Fn>
    > = (...Args, A) => R;
    ///Parameter has a name but no type. Did you mean 'arg0: Args[]'?

    // answer
    type AppendArgument<Fn, A> = Fn extends (...args: infer Arg) => any
      ? (...args: [...Arg, A]) => ReturnType<Fn>
      : never;

    // answer를 retyrnType, argumentType을 사용한 답
    type ReturnType<F extends Function> = F extends (...args: any) => infer R
      ? R
      : never;
    type ArgumentType<F extends Function> = F extends (...args: infer A) => any
      ? A
      : never;

    type AppendArgument<Fn extends Function, A> = (
      ...args: [...ArgumentType<Fn>, A]
    ) => ReturnType<Fn>;
    ```
  - 재영
    - 함수에다가 x라는 argument 하나만 추가하면 되는건가?..
      ```tsx
      // 함수 인자만 먼저 뽑아내보자 -> rest parameter로 배열로 뽑아내기
      type AppendArgument<Fn, A> = Fn extends (...args: infer T) => any
        ? T
        : never;
      ```
    - 아?
      ```tsx
      type AppendArgument<Fn, A> = Fn extends (...args: infer T) => infer R
        ? (args: [...T, A]) => R
        : never;
      ```
    - 배열로 받은 인자를 어떻게 넣어줄까...
      ```tsx
      type AppendArgument<Fn, A> = Fn extends (...args: infer T) => infer R
        ? (...args: [...T, A]) => R
        : never;
      ```
