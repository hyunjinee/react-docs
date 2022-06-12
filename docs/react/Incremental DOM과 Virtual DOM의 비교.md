# Incremental DOM과 Virtual DOM의 비교

가까운 미래에 Virtual DOM이 Incremental DOM으로 대체될 것이다.

React에 익숙한 사람이라면 Virtual DOM에 대해서 들어봤을 것이다. Virtual DOM은 UI의 성능을 높여줌으로써 리액트의 인기에 기여를 한 기능히다. 하지만 2019년 Angular가 새로운 렌더러인 Angular Ivy를 발표한 때로 돌아가 보면, 많은 이들이 Angular가 Vritual DOM대신 Incremental(증가형 또는 증분)DOM을 선택했는지 궁금했었다. 그리고 Angualr는 여전히 Incremental DOM을 고수하고 있다. 때문에 앵귤러가 왜 처음에 Incremental DOM을 사용하고 있고 계쏙해서 사용하고 있는지 이유에 대해 궁금할 것이다. 한번 알아보자.

우선 Vritual DOM의 동작 원리를 이해하고 넘어가도록 하자

## Virtual DOM의 동작 원리

Virtual DOM UI는 가상 표현을 메모리상에 두고 Reconciliation과정을 통해 실제 DOM과 동기화하는 것을 말한다. 재조정 과정은 크게 3단계로 구성된다.

1. UI가 변경되면 전체 UI를 Virtual DOM으로 렌더링한다.
2. 현재 Virtual DOM과 이전 Virtual DOM을 비교해 차이를 계산한다.
3. 변경된 부분을 실제 DOM에 반영한다.

![image](https://user-images.githubusercontent.com/63354527/173212093-cd8803cf-2650-4df2-bb07-f94eea0363a0.png)

## Incremental DOM의 동작 원리

Incremental DOM은 코드 변경점을 찾을 때 실제 DOM을 사용하여 Virtual DOM보다 더 간단한 접근 방식을 제공한다. 따라서 차이를 계산할 때 메모리상에 실제 DOM에 대한 가상 표현이 없으며 새로운 렌더트리와 차이를 비교하는데 실제 DOM이 사용된다. Incremental DOM은 명령(Instruction) 묶음을 통해 모든 컴포넌트를 컴파일한다. 이 명령들은 DOM트리를 생성하고 변경점을 찾아낸다.

![image](https://user-images.githubusercontent.com/63354527/173212169-0359fcdf-7ec5-4d4d-9c85-938c392772d9.png)

## 무엇이 Incremental DOM을 특별하게 만드는가?

위 설명에 따르면, Incremental DOM의 접근 방식이 훨씬 간단하다고 결론 내릴 것이다. 하지만 이게 다가 아니다. Incremental DOM의 진짜 이점은 메모리를 효율적으로 사용한다는 것이다. 이 최적화 방식은 휴대전화와 같이 메모리가 적은 장치에서 사용될 때 굉장히 좋다. 메모리 사용량을 최적화하는 것은 쉬운 일이 아니기 때문이다. 또한 애플리케이션의 메모리 사용은 순수하게 번들 크기와 메모리 점유율(Memory Footprint)에 기반한다.

이 두가지 요인을 감소시키는데 Incremental DOM이 어떻게 도움이 되는지 살펴보자.

1. Tree shaking이 가능한 Incremental DOM
   트리 쉐이킹은 새로운 것이 아니다. 빌드과정에서 필요로 하지 않는 코드를 제거하는 단계를 말한다. Incremental DOM은 명령 기반의 접근 방식을 사용하기 때문에 트리 쉐이킹을 최대한으로 활용한다. 앞서 말했듯이 Incremental DOM은 컴파일 전에 명령 묶음으로 각 컴포넌트를 컴파일하고 사용하지 않는 명령을 식별하는데 도움을 준다. 그래서 컴파일 시점에 불필요한 명령들을 제거할 수 있다.

![image](https://user-images.githubusercontent.com/63354527/173212243-d4113525-0506-4b4c-9c6d-62f1aff4ed78.png)

Virtual DOM은 인터프리터로 사용되기 때문에 트리쉐이킹이 불가능하고, 컴파일 시점에 사용하지 않는 코드를 알아낼 방법이 없다.

2. 메모리 사용량 감소
   Virtual DOM과 Incremental DOM의 차이점을 이해하고 있다면, 메모리 사용량 감소의 비밀도 이미 알 것 이다. Virtual DOM과 다르게 Incremental DOM은 애플리케이션 UI를 다시 렌더링할 때 실제 DOM을 복사해서 생성하지 않는다. 게다가 애플리케이션 UI에 변경이 없다면 메모리를 할당하지도 안흔ㄴ다. 대부분의 경우, 중요한 수정 상항 없이 애플리케이션을 다시 렌더링한다. 그래서 Incremental DOM의 접근 방식은 메모리 사용량을 크게 줄여준다.

![image](https://user-images.githubusercontent.com/63354527/173212295-dbafd254-fffc-4d82-9faf-e2884f0ff2fe.png)

이는 Incremental DOM이 Virtual DOM의 메모리 점유율을 줄여주는방안처럼 보인다. 하지만 왜 다른 프레임워크는 Incremental DOM을 사용하지 않는 것일까?

## Trade off

Incremental DOM이 차이를 계산할 때 더 효율적인 방법을 따르기 때문에 메모리 사용량을 줄여주기는 하지만, 이 방식은 Virtual DOM보다 시간적 비용이 더 든다. 그래서 Incremental DOM과 Virtual DOM중 결정해야할 때 속도와 메모리 사용량 사이에 트레이드오프가 존재한다.

## 최종 정리

Virtual DOM이 제공하는 속도 및 성능 향상이 React가 다른 라이벌 프레임워크와 경쟁하는데 도움이 된것은 분명하다.

Virtual DOM의 장점은 다음과 같다.

- 효율적인 비교 알고리즘(Diffing Algorithm)
- 단순하고 성능 향상에 도움이 된다.
- React 없이 사용할 수 있다.
- 경량
- 상태 변경을 생각하지 않고 애플리케이션을 빌드할 수 있다.

Virtual DOM에서 사용하는 비교 과정은 실제 DOM의 작업량을 실질적으로 줄여준다.그리고 이 과정에서 변경 사항을 식별하기 위해 이전의 Virtual DOM과 현재 Virtual DOM의 상태를 비교하는 일이 필요하다. 잘 이해할 수 있게 짧은 React코드의 예제를 살펴보자.

```js
function WelcomeMessage(props) {
    return <div className"welcome">welcome {props.name}</div>
}
```

name의 초기값을 "Chameera"로 설정한 후 "Reader"로 변경했다고 가정해보자. 전체 코드에서 변경된 부분은 name속성 뿐이고 DOM노드를 변경하거나 div 태그 안에 있는 속성을 비교할 필요가 없다 하지만 diffing 알고리즘은 변경사항을 식별하기 위해 모든 단계를 거친다. 개발 과정에서 사소한 변경 사항이 굉장히 많이 일어나는 것을 볼 수 있고, 이렇게 UI의 각 요소를 비교하는 것은 오버헤드이다. 이것이 Virtual DOM의 단점이라고 할 수 있다. 하지만 Incremental DOM을 사용하면 이러한 높은 메모리 사용량 문제를 해결할 방법이 존재한다.

이전에 말했듯이 Incremental DOM은 변경 사항을 추적하기 위해 실제 DOM을 사용하면서 Virtual DOM에서 발생하는 메모리 소비를 줄이는 방법을 제공한다. 이 접근 방식은 계싼과정의 오버헤드를 크게 줄이고 애플리케이션의 메모리 사용량도 개선한다.

이것이 Incremental DOM이 Virtual DOM보다 나은 주된 이유이며 다른 이점은 다음과 같다.

- 다른 프레임워크와 통합하기가 쉽다.
- 단순한 API는 템플릿 엔진을 구축하는데 강력한 효과를 발휘한다.
- 모바일 기기에 기반한 애플리케이션에 적합하다.

> 대부분의 경우, Incremental DOM이 Virtual DOM 보다 빠르지 않다.

Incremental DOM이 메모리 사용량을 감소시켜주지만, 차이를 계싼하는데 Virtual DOM 접근 방식보다 시간을 더 쓰기 때문에 이방안은 Incremntal DOM의 속도에 영향을 미친다. 이것이 Incremental DOM의 단점이다.

이 두 DOM모두 고유한 장점이 있으며, Virtual DOM과 Incremental DOM 중 어느 것이 낫다고 말할 수 없다. 그러나 확실한건 Virtual DOM과 Incremental DOM 모두 훌륭한 선택지이며, 동적 DOM 변경을 처리하는데 전혀 문제가 없다.
