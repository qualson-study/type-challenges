- \***\*Append Argument\*\***
  - ์ฐ๊ตฌ
    ### **Append Argument**
    For given function type `Fn`, and any type `A` (any in this context means we don't restrict the type, and I don't have in mind any type ๐) create a generic type which will take `Fn` as the first argument, `A` as the second, and will produce function type `G` which will be the same as `Fn` but with appended argument `A` as a last one.
    For example,
    ```tsx
    type Fn = (a: number, b: string) => number;

    type Result = AppendArgument<Fn, boolean>;
    // expected be (a: number, b: string, x: boolean) => number
    ```
    ```tsx
    // try 1 K๋ฅผ ๊ฐ์ ธ์ค๊ณ  ์ถ์๋ฐ ๊ฐ์ ธ์ฌ ์๊ฐ ์๋ค.. ์ด๋ป๊ฒ arg๋ฅผ ๊ฐ์ ธ์ฌ ์ ์์๊น?
    type AppendArgument<Fn extends (...arg: infer K) => any, A extends any> = (
      ...K,
      x: A
    ) => ReturnType<Fn>;
    ```
    ```tsx
    // try 2  ์ K๋ฅผ ์ ๊ธฐ ๊ทธ๋ฅ ํ๊ณ  ์ถ์๋ฐ ํ ์๊ฐ ์์๊น...!

    type AppendArgument<
      Fn extends (...arg: any[]) => any,
      A extends any
    > = Fn extends (...args: infer K) => any
      ? (...K, x: A) => ReturnType<Fn>
      : never;
    ```
    ```tsx
    // try 3 args์ ํ์์ ๋ฃ์ด์ฃผ๊ณ  ...args ํด์ ํด๊ฒฐ !
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
  - ํ์
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
  - ํ๋
  - ์ฐฌ๋ชจ
    ```tsx
    // try1 ๐ค
    type AppendArgument<Fn extends (...args: any[]) => any, A> = ( ...Parameters<Fn>, A )๐ฉ => ReturnType<Fn>

    // try2 ๐
    type AppendArgument<Fn extends (...args: any[]) => any, A> = ( ...args: [...Parameters<Fn>, A ] ) => ReturnType<Fn>
    ```
    ๋จผ์ , ํจ์์ ์ธ์์ ๋ํ ํ์์ ๊ฐ์ ธ์ค๋ ๋ฐฉ๋ฒ์ ๋ํด์ ๊ณ ๋ฏผ์ ํ๋ค. ๊ฒ์์ ํตํด์ `Parameters`๋ผ๋ ๋ด์ฅํ์์ด ์กด์ฌํจ์ ์๊ฒ๋์๋ค. ๊ทธ๋ฆฌ๊ณ  ๊ธฐ์กด์ ReturnType์ ์๊ธฐ๋๋ฌธ์ ์ด ๋์ ์กฐํฉํ๋ฉด ํด๊ฒฐํ  ์ ์์๊ฑฐ๋ผ๊ณ  ์๊ฐํ์ฌ ์ ๊ทผํ์๋ค.
    **try1**์ ํจ์๋ฅผ ํ์์ผ๋ก ํํํ๋ syntax์ ์ต์ํ์ง ๋ชปํด์ ๋์จ ๊ฒฐ๊ณผ์ธ ๊ฒ ๊ฐ๋ค.
    ์๋๋ ์ค์ ๋ก ๊ตฌํ๋ Parameters ๋ด์ฅ ํ์ ์ฝ๋์ด๋ค.
    ```tsx
    type Parameters<T extends (...args: any[]) => any> = T extends (
      ...args: infer P
    ) => any
      ? P
      : never;
    ```
  - ์ง์ 
    ```tsx
    type Fn = (a: number, b: string) => number;

    type Result = AppendArgument<Fn, boolean>;
    // expected be (a: number, b: string, x: boolean) => number

    //----------------------

    // try
    // returnType, argumentType ๊ฐ๊ฐ ๊ตฌํด์ ํจ์๋ก ๋ง๋ค๊ธฐ
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

    // answer๋ฅผ retyrnType, argumentType์ ์ฌ์ฉํ ๋ต
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
  - ์ฌ์
    - ํจ์์๋ค๊ฐ x๋ผ๋ argument ํ๋๋ง ์ถ๊ฐํ๋ฉด ๋๋๊ฑด๊ฐ?..
      ```tsx
      // ํจ์ ์ธ์๋ง ๋จผ์  ๋ฝ์๋ด๋ณด์ -> rest parameter๋ก ๋ฐฐ์ด๋ก ๋ฝ์๋ด๊ธฐ
      type AppendArgument<Fn, A> = Fn extends (...args: infer T) => any
        ? T
        : never;
      ```
    - ์?
      ```tsx
      type AppendArgument<Fn, A> = Fn extends (...args: infer T) => infer R
        ? (args: [...T, A]) => R
        : never;
      ```
    - ๋ฐฐ์ด๋ก ๋ฐ์ ์ธ์๋ฅผ ์ด๋ป๊ฒ ๋ฃ์ด์ค๊น...
      ```tsx
      type AppendArgument<Fn, A> = Fn extends (...args: infer T) => infer R
        ? (...args: [...T, A]) => R
        : never;
      ```
