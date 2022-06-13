## Append to object

- 연구
- 현석
- 화랑
- 찬모
  ```tsx
  // try1
  type AppendToObject<T, U extends string, V> = {
    [K in keyof T] : T[K]
    [U] : V
  }
  // error : A mapped type may not declare properties or methods.

  // try2
  type AppendToObject<T, U extends string, V> = {
    [K in keyof T] : T[K]
   } & {
  	[U] : V // U : V
  }
  // error는 아닌데 안됨... 😩
  ```
  💡아이디어
  try1에서 힌트를 얻어서 keyof T를 하면 리터럴 유니온 타입을 만드니까 여기에 U추가해서 map을 돌리면 되겠다 싶었음.
  ```tsx
  // try3
  type AppendToObject<T, U extends string, V> = {
    [K in keyof T | U]: K extends U ? V : T[K];
  };

  // 마지막 T[K]에서 error : Type 'K' cannot be used to index type 'T'
  // → T[K] 대신 false를 넣으면 의도대로 작동함, map도 잘돈다!!
  // → 이 에러만 해결하면 되는데...

  /*
  💡 
  위에서 K extends U 라는 말은 결국 K extends string 이기 때문에 
  이 부분이 false이면 T[K]의 K는 string이 아니라는 말이 된다. 
  즉 index로 사용할 수 있는 타입임을 보장받지 못하게 되어 위와 같은 오류가 나타나는 것
  */

  // try4 😎
  type AppendToObject<T, U extends string, V> = {
    [K in keyof T | U]: K extends keyof T ? T[K] : V;
  };
  ```
- 지선
  ```tsx
  type Test = { id: '1' }
  type Result = AppendToObject<Test, 'value', 4>
  // expected to be { id: '1', value: 4 }

  //////
  type AppendToObject<T, U, V> = any
  -> object T를 펼쳐서 U:V 값을 넣어주면 될거 같은데에

  //try1
  type AppendToObject<T extends object, U extends string, V> = {
    [key in keyof T]: T[key];
    U:V
  }
  //-> A mapped type may not declare properties or methods

  /////
  // 1. T의 key와 U를 union type으로 반환하는 타입 생성
  // 2. in 사용해서 key가 U일경우 V, 아니면 T[key]

  //try 2 🤔
  type newKeys<T, U> = (keyof T) | U
  type AppendToObject<T extends object, U extends string, V> = {
    [key in newKeys<T,U>]: key extends U ? V: T[key];
  }
  //Type 'key' cannot be used to index type 'T'.
  // 테스트 케이스 통과하나 밑줄 생김쓰....

  // key를 U랑 비교하는게 아니라 keyof T로 비교하자
  //answer
  type newKeys<T, U> = (keyof T) | U
  type AppendToObject<T extends object, U extends string, V> = {
    [key in newKeys<T,U>]: key extends keyof T ? T[key] : V;
  }
  ```
- 재영
  - try1: intersecting
    ```tsx
    type AppendToObject<T, U extends string, V> = T & { U: V };
    type TEST = AppendToObject<test2, "home", 1>;

    // 추론결과
    //type TEST = test2 & {
    //    U: 1;
    //}

    type AppendToObject<T, U extends string, V> = T & { [P in U]: V };
    type TEST = AppendToObject<test2, "home", 1>;

    // 추론결과
    //type TEST = test2 & {
    //    home: 1;
    //}
    // -> intersecting은 합집합으로 표현은 되지만, Equal 테스트를 통과하지 못함
    ```
  - try2: keyof로 다시 순회하면서 merge하자
    ```tsx
    // Object의 key를 순회하는 P가 U에 할당할 수 있다면, U의 value는 T[P]가 아니라 V라고 써준다면..
    // 완성
    type AppendToObject<T extends Record<string, any>, U extends string, V> = {
      [P in keyof T | U]: P extends U ? V : T[P];
    };
    ```
