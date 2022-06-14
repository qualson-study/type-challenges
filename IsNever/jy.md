[참고글](https://ui.toast.com/weekly-pick/ko_20220323#never-%ED%83%80%EC%9E%85%EC%9D%80-%EC%96%B4%EB%96%BB%EA%B2%8C-%EA%B2%80%EC%82%AC%ED%95%A0%EA%B9%8C)

1. 타입스크립트는 조건부 타입에 대해 자동적으로 유니언 타입을 할당한다.
2. never은 빈 유니언 타입이다.
3. 그러므로 할당이 발생하면 할당할 것이 없으므로 조건부 타입은 never로 평가된다.

```tsx
type IsNever<T> =  T extends never ? true : false
// never로 추롣됨
왜 true도 아니로 false로 아니고 never로 추론되는걸까?
```

## A extends B ? C : D

1. 여기서 A가 유니온타입이면 유니온의 요소 하나하나가 분배하면서 할당되는 성질이 있다. (Distributive Conditional Types)
2. never는 비어있는 유니온 타입이다.
3. never extends ~~~ 를 하면 요소 하나하나를 분배하려고 하는데, 요소가 없다.
4. 즉 never extends ~~~ 자체가 모순이므로 never로 추론된다.
5. 해결하려면 배열 [ ] 로 감싸면된다. - [공식문서](https://www.typescriptlang.org/docs/handbook/2/conditional-types.html) - Typically, distributivity is the desired behavior. **To avoid that behavior**, you can surround each side of the extends keyword with square brackets.
