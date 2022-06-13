- **Length of Tuple**
  **Problem**
  For given a tuple, you need create a generic `Length`, pick the length of the tuple
  For example
  ```tsx
  type tesla = ['tesla', 'model 3', 'model X', 'model Y']
  type spaceX = ['FALCON 9', 'FALCON HEAVY', 'DRAGON', 'STARSHIP', 'HUMAN SPACEFLIGHT'] as const

  type teslaLength = Length<tesla>  // expected 4
  type spaceXLength = Length<spaceX> // expected 5
  ```
  - **재영**
    - 요구사항
      - 타입으로 배열의 크기를 체크하기
    - 아이디어
      - `['length']` 로 배열을 상속하는 (length 프로퍼티가 있는 객체의) 제네릭의 길이를 알아낼 수 있다.
    - answer
      ```jsx
      type Length<T extends readonly any[]> = T['length']
      ```
    - infer 키워드를 사용한 answer
      ```tsx
      // T가 length 프로퍼티를 갖는 객체를 상속한다면 그 프로터피를 U라고 추론하고 U를 반환한다.
      // T에 length 프로퍼티가 없으면 never를 반환한다.
      type Length<T extends readonly any[]> = T extends { length: infer U }
        ? U
        : never;
      ```
  - 지선
    ```tsx
    // 배열(튜플)을 받아 길이를 반환하는 제네릭 Length<T>를 구현하세요.
    type tesla = ['tesla', 'model 3', 'model X', 'model Y']
    type spaceX = ['FALCON 9', 'FALCON HEAVY', 'DRAGON', 'STARSHIP', 'HUMAN SPACEFLIGHT']

    type teslaLength = Length<tesla>  // expected 4
    type spaceXLength = Length<spaceX> // expected 5

    // ----------

    // Cannot access 'T.length' because 'T' is a type, but not a namespace. Did you mean to retrieve the type of the property 'length' in 'T' with 'T["length"]'?
    // 'T'가 타입이지만 네임스페이스가 아니므로 'T.length'에 액세스할 수 없습니다. 'T'의 속성 'length' 유형을 'T["length"]'와 함께 검색하시겠습니까?
    1. type Length<T extends any[]> = T.length

    //테스트 케이스에서 에러 발생 -> tesla, spaceX에 as const 붙어있음
    2. type Length<T extends any[]> = T['length']

    //ok
    type Length<T extends readonly any[]> = T['length']
    ```
  - 찬모
    도전!! 🚀
    ```tsx
    # Try1
    type Length<T extends any> = T extends any[] ? T['length'] : never

    # Try2
    type Length<T extends any> = T extends readonly any[] ? T['length'] : never

    ```
    Try1 & Try2
    - 튜플
      튜플이란 배열과 유사하지만 생성될 때 길이가 정해진 배열을 말한다. readonly 배열이라고 할 수 있다.
    이전 문제에서 `[’length’]` 방식으로 적을 수 있다는 것을 알게 되어서 이를 활용하여 구현할 수 있었다. `any[ ]` 일반적인 배열 타입을 말하는 것이고 이 문제는 튜플 타입이라는 것을 알려줘야했다. 이를 나타내는 방법이 앞에 `readonly`를 붙여주는 것임을 알게 되었다.
    그런데 자꾸 아래와 같은 오류가 나왔다.
    `Unused '@ts-expect-error' directive.`
    @ts-expect-error 지시문이 적혀있는 경우 타입스크립트가 해당 코드에서 발생하는 에러를 무시한다. 반대로 해당 코드에서 에러가 발생하지 않는다면 위와 같은 에러 메세지를 메세지를 띄운다. 즉, 필요없는 지시문이다라는 의미!
    위 코드를 살펴보면, 조건에 맞지않는 경우 에러가 나야하는데 에러가 아닌 never라는 정상적인 타입이 들어가기 때문에 위와 같은 오류가 났던 것이다.
    풀이 확인 ~⭐️
    ```tsx
    type Length<T extends readonly any[]> = T["length"];

    type Length<T extends any> = T extends { length: infer U } ? U : never;
    ```
  - 화랑
    - 처음 생각: T.length...?
    - 답: `type Length<T extends readonly any[]> = T[’length’]`
      - index signature 에는 일반 프로퍼티와 공존해서 사용할 수 있다! ([링크](https://lakelouise.tistory.com/196))
        - 여기서는 T 가 배열이니까 T[’concat’], T[’indexOf’] 처럼 타입 선언이 가능하다.
      - Tuple 은 원래 불변 구조니까 readonly 를 써주자
  - 현석
    생각보다 금방 끝났다.
    ```tsx
    type Length<T extends readonly any[]> = T["length"];
    ```
  - 연구
    ```tsx
    type tesla = ["tesla", "model 3", "model X", "model Y"]; // string[] // number
    type spaceX = [
      "FALCON 9",
      "FALCON HEAVY",
      "DRAGON",
      "STARSHIP",
      "HUMAN SPACEFLIGHT"
    ];

    type Length<T extends any[]> = T["length"];
    type Length<T extends readonly any[]> = T["length"];

    type teslaLength = Length<[tesla]>; // expected 4
    type spaceXLength = Length<spaceX>; // expected 5
    ```
