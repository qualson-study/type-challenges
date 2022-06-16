# IsUnion

[문제 링크](https://github.com/type-challenges/type-challenges/blob/main/questions/01097-medium-isunion/README.md)

## 목표

```tsx
type case1 = IsUnion<string>; // false

type case2 = IsUnion<string | number>; // true

type case3 = IsUnion<[string | number]>; // false
```

## Solution

```tsx
// Equal은 써먹을데가 종종 있으니 계속 봐두자
type MyEqual<X, Y> = (<T>() => T extends X ? 1 : 2) extends <T>() => T extends Y ? 1 : 2 ? true : false>

// 부정 접두사
type NotPrefix<T extends boolean> = T extends true ? false : true

// T를 C에 할당할 수 있는데, T와 C가 서로 동치가 아니라면 T는 유니온 타입이다.
type IsUnion<T, C = T> = T extends C ? NotPrefix<MyEqual<T, C>> : false
```
