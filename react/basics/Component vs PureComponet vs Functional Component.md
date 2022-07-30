# Component vs PureComponent

이 글은 2017년에 작성된 글 입니다.

React에서 컴포넌트를 만드는 방법에는 크게 클래스 기반 (React.Component), React.PureComponent를 확장(extends)해서 사용, 함수 기반 컴포넌트 (Function Component)으로 나눌 수 있다.

먼저 클래스 기반 컴포넌트 (React.Component, React.PureCompoent)에 대해서 알아보자. 사실 두 개는 shouldComponentUpdate 라이프 사이클 메소드를 다루는 방식을 제외하곤 같다. 즉 PureComponent는 shouldComponentUpdate 라이프 사이클 메소드가 이미 적용된 버전의 React.Component클래스라고 보면 된다.

React.Component를 확장해서 컴포넌트를 만들 때, shouldComponentUpdate메서드를 별도로 선언하지 않았다면, 컴포넌트는 props, state값이 변경되면 항상 리렌더링(re-render)하도록 되어있다.

하지만, React.PureComponent를 확장해서 컴포넌트를 만들면, shouldComponentUpdate 메서드를 선언하지 않았다고하더라도, PureComponet 내부에서 props와 state를 shallow level 안에서 비교하여,변경된 값이 있을시에만 리렌더링 하도록 되어있다.

이를 제외하고 React.Component와 React.PureComponent의 다른 점은 없다. 그렇다면 함수형 컴포넌트가 클래스기반의 컴포넌트와 다른점은 무엇일까? 함수 컴포넌트는 클래스 기반의 컴포넌트와 달리 state, 라이프 사이클 메소드(componentDidMount, shouldComponentUpdate)와 ref콜백을 사용할 수 없다는데 있다.(context는 사용할 수 있다.). (React 16.8 버전 이후에는 hooks API가 추가되어 함수 컴포넌트에서도 state관리를 할 수 있다.)언뜻보면, 함수 컴포넌트가 state도 없고, 라이프사이클도 신경쓰지 않기 때문에,클래스 기반의 컴포넌트 보다 퍼포먼스가 뛰어날 것이라고 예상 하지만 실제로는 그렇지 않다.

아래에 글에서도 확인할 수 있듯이, 함수 컴포넌트가 클래스 컴포넌트의 퍼포먼스보다 우위에 있다고 하기엔 어렵다. 오히려 React.PureComponent가 React.Component와 함수 컴포넌트보다 더 빠른걸 알 수 있다. React.PureComponet는 shouldComponentUpdate를 통해 리렌더링을 최소화하는 로직이 들어가 있으니 그렇다 치더라도 함수 컴포넌트와 클래스 컴포넌트의 속도 차이가 없는 것은 의외라고 할 수 있다. 사실 그 이유는 의외로 간단한데, 함수 컴포넌트도 결국엔 클래스 기반 컴포넌트로 래핑(wrapping)되기 때문이다. 아래 Dan의 트윗을 보자.

> I'm not sure if we explained this elsewhere but they are not faster becaure they _are_ class internally - Dan Abramov (July 19, 2016)

> There is no 'optimized' support for them yet because stateless component is wrapped in a class internally. It's same code path. - Dan Abramov (July 19, 2016)

그렇다면 항상 React.PureComponent를 사용하는게 좋다고 할 수 있을까? 대답은 당연히 아니다. 위에서도 언급 했지만, PureComponent는 shallow level로만 데이터를 비교하기 때문에, nested object등의 변경된 데이터는 감지하지 못하기 때문에 React.Component의 shouldComponentUpdate 직접 다뤄야한다. 또한 퍼포먼스 적으로도 모든 컴포넌트에 PureComponent를 사용하는 것은 오히려 앱을 더 느리게 할 수 있다.

> PSA: React.PureComponent can make your app slower if you use it everywhere. - Dan Abramov (January 15, 2017)

> Think about it. If component's props are shallowly unequal more than not, it re-renders anyway, but it also had to run the checks. - Dan Abramov (January 15, 2017)

클래스 컴포넌트 (state, lifecycle, ref)가 필요한 상황이 아니라면 항상 컴포넌트는 함수로 만드는게 좋다.

그렇다면, 함수 컴포넌트를 그대로 유지하면서 리렌더링을 최소화할 수 있는 방법은 없을까? 리액트 유틸리티(higher order components)라이브러리인 recompose등을 사용하면 함수 컴포넌트를 그대로 유지하면서 PureComponent(pure)의 효과를 누리거나 그보다 더 나은 퍼포먼스 상에 이점(onlyUpdateForKeys, onlyUpdateForPropTypes)을 가질 수 있는 함수들이 있기 때문에 이런 라이브러리를 적극 활용하면 함수 컴포넌트를 그대로 유지하면서 리렌더링을 최소화할 수 있다.

pure HOC(Higher Order Component)의 경우 React.PureComponent와 로직상 같고, onlyUpdateForKeys, onlyUpdateForPropTypes HOC는 특정 props값이 변경되었을 때만 리렌더링을 하도록 하기 때문에, pure, React.PureComponent 보다 퍼포먼스 상에 우위를 가져갈 수 있다.
