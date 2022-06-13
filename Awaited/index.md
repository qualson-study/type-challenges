### Awaited

[문제](https://www.typescriptlang.org/play?#code/PQKgUABBCMAcCcEC0ECCB3AhgSwC4FMATSZJM8kgIwE8IBZTAY23wCsIBlbAawHsAnTBAAUAAQC2TFqwDOPAZgCUEAMT5MM2ioAO-XuOwz8qygFdsAG1xJsAOxIkVTiAEVT+Gbmy97UEgEkAMwh0YwALTAA3YyFcam1jdDDsRjCIQxDBbQTCCDiEiAseYwAFPQMjADoIAAledBDjRkxbCABzfFwIWPjE5NT0mXTbOUJjXDDErJy83oB+CAAxAQh8AA9McW0LY2xg0IgI6IgAAzL9Q3wAHgBRDa2dgBVegD4Tw-q83nbOiDvN7b4Z4JOYOKAvCCPZJDACO7k83laGW0AgIuUC5TykwgAG0BNg2nZMBZuvwvIwdgBdYRhXC4bQyABcwGAY0ilVwvGAkkYcj4gmAmEIkRajCISHyHkY-Gw2ms63w-GYRhkSDhHi8PiQ0CQABYAKzcXXKGi4iRSNh8hTU2n0pkswkTUyUSqMfTci2yeSCRRgiAQgBqLAaPggAHE8DVnYzDnSGczgLgZKlKrJKgI2sA4PAwCBgGAC6AIAB9Utl8tliAATV4pn4EAAwrwxrVFcYKx3SxA8wXJfRqBgcGiIABebq2agFsBFzsdyEaxsaDwl2fl7v57BbVEQADefzhxIANH81glGF0AL4QDH6CAAclEkqQqWJO1sHRkwFMXgsMjvvd6CAAA1Rwgc4KmuTwZXfF4wD7KtQPAy4rj3QIWAsQgY1sUxxEoRUIAvWC+wALUQ8pkKQowriguw2ggAAfCBsNwxUXlguDAOaFVQJxEg7jPXBbgPCwrjoAcsDwIgriAl5jxomDZL4098HPITTGJUTxKHKSq1k3dr3QzCmJwvD60IxSoH4lTBJuYTNMHSTCCuYi9PkujGOY0y2MPMBKULYAIEfVV1gEpBFT0fgOIKcKVjHMSHLRK5PNYqdpxAFdVy7RY6wmfCOAIBkMsy9c0pICEOAifhjGoWt6xkXgLG-REmVjO0EyTFM0wzLMEEFEZQki8EICDfAGnqxrNRGGNbXjFkOrCVMZHTfhM2zYBxqanwZDK+gBGMBsIgsN8P2muN7UTZMFq6lbc3zMAgA)

- 연구
  ### **Awaited**
  If we have a type which is wrapped type like Promise. How we can get a type which is inside the wrapped type? For example if we have `Promise<ExampleType>` how to get ExampleType?
  ```tsx
  // try 1

  type MyAwaited<T> = T extends Promise<infer U> ? U : never

  import { Equal, Expect } from '@type-challenges/utils'

  type X = Promise<string>
  type Y = Promise<{ field: number }>
  type Z = Promise<Promise<string | number>> // 이 부분에서 문제가 생긴다 한꺼풀 더 어떻게 벗겨낼까?

  type cases = [
    Expect<Equal<MyAwaited<X>, string>>,
    Expect<Equal<MyAwaited<Y>, { field: number }>>,
    Expect<Equal<MyAwaited<Z>, string | number>>,
  ```
  ```tsx
  // try 2
  // Promise를 어떻게 하면 한겹 더 벗겨줄 수 있을까 ? 아래와 같이하니 일단은 성공 하지만 아래 ts-expect-error 가 빨개진다..

  type MyAwaited<T> = T extends Promise<infer U> ? U extends Promise<infer K> ? K : U : never

  type X = Promise<string>
  type Y = Promise<{ field: number }>
  type Z = Promise<Promise<string | number>>

  type cases = [
    Expect<Equal<MyAwaited<X>, string>>,
    Expect<Equal<MyAwaited<Y>, { field: number }>>,
    Expect<Equal<MyAwaited<Z>, string | number>>,

  @ts-expect-error //
  type error = MyAwaited<number>

  // try 3
  // T 의 타입을 잡아주니 빨개지는 부분은 사라짐, 하지만 더 깊어 질 경우 어떻게 해야될지...
  type MyAwaited<T extends Promise<any>> = T extends Promise<infer U> ? U extends Promise<infer K> ? K : U : never
  ```
  ```tsx
  // solution
  // type에도 재귀를 사용할 수 있었구나.. 그리고 무슨타입이 올지 모를 때는 any 보단 unknown이 좋아보인다.
  // never(너는 나올리가 없어) vs any(아무나와~~~) vs unknown(보수적)
  type MyAwaited<T extends Promise<unknown>> = T extends Promise<infer U>
    ? U extends Promise<unknown>
      ? MyAwaited<U>
      : U
    : never;
  ```
- 현석
  Try 1.
  ```tsx
  type MyAwaited<T> = T extends Promise<infer V> ? V : never

  type Z = Promise<Promise<string | number>>
  Expect<Equal<MyAwaited<Z>, string | number>> <- 여기에서 에러가 난다.
  // Promise가 중첩이면 중첩을 풀어줘야 한다.... 재귀?
  ```
  Try 2.
  ```tsx
  type MyAwaited<T> = T extends Promise<infer V> ? MyAwaited<V> : T;

  // @ts-expect-error
  type error = MyAwaited<number>;

  // T가 Promise가 아닌 값이면 에러를 내야 한다.
  ```
  Try 3.
  ```tsx
  type MyAwaited<T extends Promise<any>> = T extends Promise<infer V> ? MyAwaited<V> : T
                                                                                  ^
  // V가 Promise라는 보장이 없어서 MyAwaited에 넣을 수 없다고 한다...
  ```
  Try 4.
  ```tsx
  type Resolve<T> = T extends Promise<infer V> ? Resolve<V> : T;
  type MyAwaited<T extends Promise<any>> = T extends Promise<any>
    ? Resolve<T>
    : never;

  // Recursive하게 Promise를 풀어주는 타입 하나와
  // Promise를 받아야만 하는 타입으로 두 가지를 분리했다.
  ```
- 화랑
- 찬모
  도전~🚀
  ```tsx
  // Try1
  type MyAwaited<T extends Promise<T?? any?? unknown??>> = ?? * Infinity 🤯

  ```
  사실 이 문제는 뒤쪽을 구현하는 방법을 생각할 수 없어서 답을 찾아볼 수 밖에 없었다. 흑흑
  답...
  ```tsx
  type MyAwaited<T extends Promise<unknown>> = T extends Promise<infer R>
    ? R extends Promise<unknown>
      ? MyAwaited<R>
      : R
    : never;
  ```
  > 무슨 이상한 나라의 앨리스 같은 코드가 펼쳐지니...
  MyAwaited를 재귀적으로 사용하는 타입 방식이였다. 여기서 힌트를 얻어서 혹시나 하는 마음에 `recursive type in typescript` 라는 검색어로 구글링을 해보았다. 여기에 찾은 예제들을 정리해본다.
  타입스크립트 버전 3.7 이전는 해당 코드(재귀적 타입 별칭)는 오류로 간주되었다. 하지만 3.7 이후에는 정식으로 인정되어서 우리는 자유롭게 쓸 수 있다고 한다. (안좋은듯...😡) [**참고** [공식 스펙 문서](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-7.html#more-recursive-type-aliases)]
  ex1)
  ```tsx
  type Person = {
    name: string;
    age: number;
    friends?: Person[];
  };

  let me: Person = {
    name: "jjanmo1",
    age: 10,
    friends: [
      {
        name: "jjanmo2",
        age: 20,
      },
    ],
  };
  ```
  `friends` 속성이 옵셔널이기 때문에 재귀적인 데이터를 만들수있다. 하지만 옵셔널이 아니면 재귀적 타입으로 만들 수 없게 된다.
  ex2) 공식 스펙 문서에 나온 예시
  ```tsx
  type VirtualNode =
    | string
    | [string, { [key: string]: any }, ...VirtualNode[]];
  const myNode: VirtualNode = [
    "div",
    { id: "parent" },
    ["div", { id: "first-child" }, "I'm the first child"],
    ["div", { id: "second-child" }, "I'm the second child"],
  ];
  ```
  유니온 타입과 스프레드를 이용해서 재귀적 타입을 좀 더 유연하게 사용할 수 있는 것 같다. 사실 이러한 상황을 만나보지 못해서 어떤식으로 사용해야할지 감이 오지 않는다.
  ex3)
  **Step1**
  ```tsx
  type Status = "TODO" | "DOING" | "DONE";

  type Todo = {
    title: string;
    status: Status;
  };

  let todo: Readonly<Todo> = {
    title: "자바스크립트 공부하기",
    status: "TODO",
  };

  todo.status = "DOING"; // error 🤬
  // Cannot assign to 'status' because it is a read-only property.
  ```
  - 참고 Readonly (복습용)
    실제 구현
    ```tsx
    type MyReadonly<T> = { readonly [P in keyof T]: T[P] };
    ```
    여기서 `in` 은 반복의 느낌(?)으로 이해하고 `keyof T` 는 T 타입의 속성을 키값을 문자열 유니온 타입으로 뱉어낸다. 그래서 해당하는 키값의 수 만큼 반복하면서 readonly 속성을 추가한다고 이해할 수 있다.
  `status` 속성을 바꾸려고 하면 아래와 같은 에러를 뿜뿜한다. 왜냐, Readonly 니까!!
  **Step2**
  여기서 type Todo에 `with` 라는 속성을 추가한다.
  ```tsx
  type Todo = {
    title: string;
    status: Status;
    with: {
      friends: string[];
      date: string;
      location: string;
    };
  };

  let todo: Readonly<Todo> = {
    title: "자바스크립트 공부하기",
    status: "TODO",
    with: {
      friends: ["jjanmo"],
      date: "2022.02.30",
      location: "NewYork",
    },
  };

  todo.with.friends.push("michael"); // not error 😎
  todo.with.date = "2022.03.01"; // not error 😎
  ```
  친구를 추가하고 싶다! or 날짜를 바꾸고 싶다!! 라고 한다면 이 때는 다 바꿀수 있다. 위에서 Readonly가 어떻게 구현되어있는지 보았듯이 1depth의 객체 속성만을 바꾸지 못하게 하는 것!
  **Step3**
  그래서 하고 싶은 것은 설정한 todo값의 **레알 Readonly**로 만들고 싶다!
  ```tsx
  // 1)
  type SuperReadonly<T> = {
    readonly [P in keyof T]: T[P] extends Object ? SuperReadonly<T[P]> : T[P];
  };

  // 2)
  type SuperReadonly<T> = {
    readonly [P in keyof T]: SuperReadonly<T[P]>;
  };

  let todo: SuperReadonly<Todo> = {
    title: "자바스크립트 공부하기",
    status: "TODO",
    with: {
      friends: ["jjanmo"],
      date: "2022.02.30",
      location: "NewYork",
    },
  };

  todo.with.friends.push("michael"); // error 🎉
  // Property 'push' does not exist on type 'readonly string[]'

  todo.with.date = "2022.03.01"; // error 🎉
  // Cannot assign to 'date' because it is a read-only property.
  ```
  이런 식으로 **재귀적인 방법**을 활용할 수 있다!! ( 1)은 제 생각, 2)은 someone )
  그런데!! 여기서 약간 이상했던게 2번 같은 경우 속성 타입이 원시타입이여도 재귀타입을 돌게 되는데 그럼에도 아무런 문제가 없었다.
  원시타입일 때, 한 번 과정을 음미해보면,
  ```tsx
  let todo: SuperReadonly<Todo> = {
    title: '자바스크립트 공부하기',
    status: 'TODO',

  	//...
  };

  // 이런 식으로 들어가는거 아닐까 🤔
  title : SuperReadonly<'자바스크립트 공부하기'>
  status : SuperReadonly<'TODO'>
  ```
  그런데 이 과정이 결국엔 재귀적 부분빼고 Readonly하고 같기 때문에 `Readonly<'자바스크립트 공부하기'>`
  이런 식으로 테스트해보면 나오는 타입이 `‘자바스크립트 공부’` 그대로 나오게 된다. 즉 원시타입은 원시타입의 리터럴 타입을 갖게 됨으로서 해당 타입대로의 값을 사용하게 된다.
  ```tsx
  type T1 = Readonly<"자바스크립트 공부하기">;

  let title: T1 = "자바스크립트 공부하기"; // ⭕️

  let title: T1 = "자바 공부하기"; // error ❌
  //Type '"자바 공부하기"' is not assignable to type '"자바스크립트 공부하기"'
  ```
  또 한가지 의문점은 push를 사용하면 결국 같은 객체를 변형시키는 것이기 때문에 `immutable`하지 않다. 즉 push를 사용하면 friends에 재할당하는게 아니기 때문에 제대로 작동해야하는거 아닌가하는 의문을 품을 수 있다. (실제로 friends에는 객체 메모리 주소값을 담고 있기 때문에 push를 하면 값을 변형시킨 것이 아니다.) 그런데 에러메세지를 자세히 보면 `readonly string[]` 즉 튜플이기 때문에 더이상 추가할 수 없다.
  - 참고 튜플이란
    길이와 타입이 고정된 배열.
    - `[string, number]` 문자열과 숫자타입을 가진 길이 2인 배열 → 튜플
    - 아주 많이 마주하는 예로는 `const [state, setState] = useState()` useState()가 반환하는 것이 배열처럼 보이지만 튜플이다.
  ## 레알 참고
  ### **Step4**
  그런데 갑자기 `status` 속성을 제외한 속성을 Readonly로 만들고 싶다면 ?? 왜냐하면 todo에 맞게 뭔가 상태변화를 주고 싶기때문에...
  현재까지의 코드는 아래와 같다. 새로 만들 타입 이름 `SuperReadonlyExceptStatus` 😡
  가능할까??
  ```tsx
  type Status = "TODO" | "DOING" | "DONE";

  type Todo = {
    title: string;
    status: Status;
    with: {
      friends: string[];
      date: string;
      location: string;
    };
  };

  type SuperReadonly<T> = {
    readonly [P in keyof T]: SuperReadonly<T[P]>;
  };

  let todo: SuperReadonly<Todo> = {
    title: "자바스크립트 공부하기",
    status: "TODO",
    with: {
      friends: ["jjanmo"],
      date: "2022.02.30",
      location: "NewYork",
    },
  };
  ```
- 지선
  ```tsx
  Promise와 같은 타입에 감싸인 타입이 있을 때,
  안에 감싸인 타입이 무엇인지 어떻게 알 수 있을까요?
  예를 들어 Promise<ExampleType>이 있을 때,
  ExampleType을 어떻게 얻을 수 있을까요?

  // ---------------
  1. type MyAwaited<T> = T extends Promise<infer U> ? U : T
  //T가 Prmoise<>일때 에러 발생

  2. type MyAwaited<T> =T extends Promise<infer U>
    ? U extends Promise<infer Y>
      ? Y : U
    : T
  // T가 중첩되는 Promise를 가진다면...? -> 재귀함수처럼?

  3. type MyAwaited<T> =T extends Promise<infer U> ? MyAwaited<U> : T

  //2, 3번 테스트 케이스는 에러 안나나 type error = MyAwaited<number> 는 에러 발생
  // Promise가 아니면 never 반환? -> error를 반환해야됨

  [🙆‍♀️]
  4. type MyAwaited<T> =T extends Promise<infer U>
    ? U extends Promise<infer Y>
      ? Y : U
    : error

  // promise가 여러개 중첩될 경우
  [🙆‍♀️]
  type MyAwaited<T> =T extends Promise<infer U>
    ? U extends Promise<any>
      ? MyAwaited<U> : U
    : error

  ```
- 재영
  - 아이디어
    - Promise<U> 에 모양맞추기 하고 (extends) 모양이 맞다면 (conditional Type) U를 꺼내오는 형식이네? → infer를 사용해보자
    - 그런데 U 안에서 Promise가 또 들어있을 수도 있다... 재귀적으로 풀어줘야하는데 어떻게 풀지???
  - Try
    ```tsx
    // Try1.
    type MyAwaited<T> = T extends Promise<infer U> ? U : any

    // P1: U 안에 Promise가 또 들어있을 경우를 커버하지 못함
    type Z = Promise<Promise<string | number>>
    Expect<Equal<MyAwaited<Z>, string | number>> // error

    // P2: any를 뱉기 때문에 아래 예시에서 에러가 발생하지 않음
    type error = MyAwaited<number>

    // Try2. -> 여기서 P2는 해결. P1은 어떻게 하지??? -> 나중에 다시 생각해보니 이런식으로 해결하면 안됨
    type MyAwaited<T> = T extends Promise<infer U> ? U : MyAwaited<T>

    // Try3. -> infer를 2번 써서 해결하면 ? ... 깊이가 3단계일경우 커버 안됨. 이걸 어떻게 일반화할 수 없을까 ...
    type MyAwaited<T> = T extends Promise<infer U> ? U extends Promise<infer R> ? R : U: MyAwaited<T>

    // 연결고리를 여기서 끊고 MyAwaited를 다시 대입하자.
    type MyAwaited<T> = T extends Promise<infer U>
                    // ? U extends Promise<infer R> ? R : U: MyAwaited<T>
    ```
  - N단계까지 들어가더라도 답이 되어야함
    ```tsx
    type X = Promise<string>
    type Y = Promise<{ field: number }>
    type Z = Promise<Promise<string | number>>
    type ZZ = Promise<Promise<Promise<string | number>>>
    type ZZZ = Promise<Promise<Promise<Promise<string | number>>>>
    ...

    type cases = [
      Expect<Equal<MyAwaited<X>, string>>,
      Expect<Equal<MyAwaited<Y>, { field: number }>>,
      Expect<Equal<MyAwaited<Z>, string | number>>,
      Expect<Equal<MyAwaited<ZZ>, string | number>>,
      Expect<Equal<MyAwaited<ZZZ>, string | number>>,
    ...
    ]
    ```
  - answer
    ```tsx
    type MyAwaited<T extends Promise<any>> = T extends Promise<infer R>
      ? R extends Promise<any>
        ? MyAwaited<R>
        : R
      : T;
    ```
  - 스터디전날 얘만 다시 풀어봄
    ```tsx
    // 음 그래 가장 쉽게 생각나는건 이건데
    type MyAwaited<T> = T extends Promise<infer U> ? U : any

    // 꼬리를 물고 일단 해결하면 이런식이었고
    type MyAwaited<T> = T extends Promise<infer U>
                    // ? U extends Promise<infer R> ? R : U: MyAwaited<T>

    // RemovePromise같은걸 미리 만들어두는 방식도 본 것 같음. 재귀적으로 Promise<어쩌고>에서 어쩌고를 꺼내줌
    type RemovePromise<T> = T extends Promise<infer U> ? RemovePromise<U> : T

    // 이렇게 쓰면 ts-expected error에 걸림
    type MyAwaited<T> = RemovePromise<T>

    // 제네릭T는 Promise만 받을 수 있다는 제한을 두면 error 해결 (T에 number 같은 애들 할당 방지)
    type MyAwaited<T extends Promise<any>> = RemovePromise<T>
    ```
- etc
  ![Untitled](%E1%84%90%E1%85%A1%E1%84%8B%E1%85%B5%E1%86%B8%20%E1%84%8E%E1%85%A2%E1%86%AF%E1%84%85%E1%85%B5%E1%86%AB%E1%84%8C%E1%85%B5%203%E1%84%92%E1%85%AC%E1%84%8E%E1%85%A1%2010b7c1e723674b06a5345b7eb4b7c127/Untitled.png)
