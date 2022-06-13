## Flatten

- 연구
- 현석
- 화랑
- 찬모
  푸는중... 재귀가 잘 안됨... 😡
  ```tsx
  type Flatten<T extends any[]> = T extends [infer A, ...infer R]
    ? A extends any[]
      ? [...Flatten<A>, ...R]
      : [A, ...Flatten<R>]
    : T;

  // error
  type T0 = Flatten<[[2]]>; // [2, []]
  type T1 = Flatten<[1, [2]]>; // [1, 2, []]
  type T2 = Flatten<[1, 2, [3, 4], [[[5]]]]>; // [1, 2, 3, 4, [[[[5]]]]]
  ```
- 지선
  ```jsx
  type flatten = Flatten<[1, 2, [3, 4], [[[5]]]]>
  // [1, 2, 3, 4, 5]

  /////

  ****Length of String 처럼 재귀와 Arr를 둬서 처리하면 되지 않을까?****

  //try1
  type Flatten<T extends any[], A extends any[] = []> = T extends [infer F, infer R]
    ? F extends any[]
      ? Flatten<[...F, R]>
      : R extends any[]
      ? Flatten<[...R], [...A, F]>
      : [...A, F, R]
    : [...A, ...T]

  // type AA = Flatten<[1, 2, [3, 4], [[[5]]]]>
  // type AA = [1, 2, [3, 4], [[[5]]]]

  //try2👌
  //includes 구현시 확인해보니 array 나눌땐 [infer F, ...infer R]로...!
  type Flatten<T extends any[], A extends any[] = []> =
  T extends [infer F, ...infer R]?
    (F extends any[] ?
      Flatten<[...F, ...R], A>
      : (R extends any[]
        ? Flatten<R, [...A, F]>
        : [...A, F, R]))
    : A

  // 열정 ...
  ```
- 재영
  - 배열안에 배열이 들어가있는걸 재귀적으로 벗겨주면(Escape) 되겠지?
    ```tsx
    type EscapeArray<T> = T extends [...infer U] ? U : T;
    type TEST = EscapeArray<[[1, 2, 3]]>; // [[1, 2, 3]] 으잉??

    // 아 이렇게 해야지
    type EscapeArray<T> = T extends [infer U] ? U : T;
    type TEST = EscapeArray<[[1, 2, 3]]>; // [1, 2, 3]

    // 배열의 요소들을 유니온으로 뽑아내자. 그리고 EscapeArray를 재귀적으로 돌려보자
    type EscapeArray<T> = T extends [infer U] ? EscapeArray<U> : T;
    type Flatten<T extends any[]> = EscapeArray<T[number]>;
    type TEST = Flatten<[1, 2, [3, 4], [[[5]]]]>; // 1 | 2 | [3, 4] | 5

    // 이제 유니온을 튜플로 바꿔주면 되는데 ...
    ```
  - 정답확인
    ```tsx
    // infer로 앞에서 하나씩 꺼내니까 편하네 ...
    type Flatten<T extends unknown[]> = T extends [infer First, ...infer Rest]
      ? [
          ...(First extends unknown[] ? Flatten<First> : [First]),
          ...Flatten<Rest>
        ]
      : [];

    //(First extends unknown[] ? Flatten<First> : [First]) 에서는
    // 배열 안의 첫번째 요소 First를 []안에 담아서 펼쳐준다.
    // 결론적으로 First만 똑 떼어다가 담아놓는 결과가 나온다.
    ```
  - 스스로 복습용 퀴즈
    ```tsx
    //Q1.
    type EscapeArray<T> = T extends [infer U] ? EscapeArray<U>: T

    EscapeArray<[[[1]]]> // ?
    EscapeArray<[[[1, 2, 3]]]> // ?

    //Q2.
    type WOW<T extends unknown[]> = T extends [infer First, ...infer Rest] ? true : false
    type TEST =  WOW<[]> // ?
    type TEST2 = WOW<[1, 2, 3]> // ?
    ```
