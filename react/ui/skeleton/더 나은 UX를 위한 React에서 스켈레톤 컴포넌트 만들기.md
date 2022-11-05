# 더 나은 UX를 위한 React에서 스켈레톤 컴포넌트 만들기

![image](https://user-images.githubusercontent.com/63354527/200098434-5403a441-55a7-4a69-9ba4-95bace07fddd.png)

스켈레톤 컴포넌트는 왜 필요할까? 스켈레톤 컴포넌트는 데이터를 가져오는 동안 콘텐츠를 표시하는 컴포넌트다. 사용자는 콘텐츠를 기다리다가 쉽게 지치고 지루함을 느끼므로 단순희 흰색 화면만 보여준다면 많은 사람들은 사이트를 떠나게 될 것이다.

이 글에서는 스켈레톤 컴포넌트를 만들고 사용하는 방법을 설명할 것이다.

## React 프로젝트 설정

CRA를 사용하여 간단한 기본 React 프로젝트를 만들었다. 프로젝트 구조는 다음과 같다.

```tsx
import axios from "axios"
import React, { useEffect, useState } from "react"

function App() {
  const [data, setData] = useState([])

  useEffect(() => {
    axios
      .get("https://reqres.in/api/users?page=2")
      .then((res) => setData(res.data.data))
  }, [])

  return (
    <div className="App">
      <ul className="contentWrapper">
        {data.length > 0 &&
          data.map((item) => {
            return (
              <li key={item.id} className="item">
                <div>
                  <img className="img" src={item.avatar} />
                </div>
                <div className="info">
                  <p>
                    <strong>
                      {item.first_name} {item.last_name}
                    </strong>
                  </p>
                  <p>{item.email}</p>
                </div>
              </li>
            )
          })}
      </ul>
    </div>
  )
}

export default App
```

npm start로 실행하면 다음과 같은 화면이 보일 것이다.

![image](https://user-images.githubusercontent.com/63354527/200098598-352860fb-6513-48d6-a0eb-4cae1ae0277a.png)

## 스켈레톤 UI는 무엇인가?

스켈레톤 UI는 실제 콘텐츠가 들어가게될 자리를 잠시 대신할 빈 껍데기이다.

![](https://miro.medium.com/max/605/0*ICXfiqEGoeZ_O33e.gif)

웹 사이트에 접속했을 때 로딩 중임을 알려주는 컴포넌트를 본적이 있을 것이다. 데이터를 가져오고 있다는 것을 알려주는 사용자 친화적인 UI이다. 일반적으로 로딩 바나 스피너만 있는건 아니고 배경도 흐리게 표시한다. 데이터가 로딩되고 있음을 보여주는 스피너(spinner)나 바(bar)는 좋은 전략이며 사용자가 페이지또는 컨테이너가 로딩되고 있음을 이해하는데 도움이 된다.

요즘 많은 웹 사이트에서 지루한 대기 시간을 줄이기 위해 사용자에게 대체 UI 컴포넌트를 보여준다. 스켈레톤 UI의 가장 일반적인 패턴은 흰색 배경과 반짝이는 CSS 애니메이션이 있는 작은 박스이다.

![](https://miro.medium.com/max/700/0*qRwLOnqBHnYRAmKJ.gif)

빈 공간을 채울 뿐만 아니라 사용자는 페이지의 다른 컴포넌트와 상호작용할 수 있다.

이제 예제를 통해 만드는 방법을 살펴보자.

## 1. 스켈레톤 UI 구조화

이 단계에서는 비어있는 스켈레톤 컴포넌트를 만든다.

```tsx
import './Skeleton.css';

import React from 'react';

const Skeleton = () => {
  return (
    <li className="skeleton-item">
      <div>
        <div className="skeleton-img" />
      </div>
      <div className="skeleton-info">
        <p className="skeleton-name" />
        <p className="skeleton-email" />
      </div>
    </li>
  );
);

export default Skeleton;
```

![](https://miro.medium.com/max/700/1*JMeJaXez2DvpSfV1l6d89A.png)

> 스켈레톤은 가능한 콘텐츠와 같아야 한다.

스켈레톤 컴포넌트를 만들때 주의해야할 점이 있다. 스켈레톤 UI의 목적은 데이터에 대해 지루한 대기시간을 줄이는 것으로 실제 UI 컴포넌트와 너무 다르지 않아야한다는 점이다. 너무 다르면 사용자는 스켈레톤 컴포넌트가 또 다른 독립적인 UI 컴포넌트라고 느낄 수 있기 때문이다.

## 2. CSS 애니메이션

두 번째 단계는 스켈레톤을 통과할 애니메이션을 선택하는 것이다. 어떤 사람들은 순수한 CSS 애니메이션을 사용하고 어떤 사람들은 이미지를 사용한다. 필자는 개인적으로 이미지를 사용하는 것을 선호한다.

![](https://miro.medium.com/max/700/0*nS0d--Obi0NRNtu6.gif)

왼쪽에서 오른쪽으로 스켈레톤을 통과하는 흰색 그래디언트가 보이는가? 이 그래디언트를 위해 16.6ms 마다 각 지점의 모든 그래디언트 색상 값을 계산(CPU)하고, 화면에 그리는 (GPU) 불필요한 작업을 해야한다. 그래서 이미지를 선호한다.

그러나 이 예제에서는 순수하게 CSS만 사용하겠다. 항상 이미지를 사용할 순 없기 때문이다.

```css
@keyframes loading {
  0% {
    transform: translateX(0);
  }
  50%,
  100% {
    transform: translateX(460px);
  }
}

.skeleton-item::before {
  content: "";
  position: absolute;
  top: 0;
  left: 0;
  width: 30px;
  height: 100%;
  background: linear-gradient(to right, #f2f2f2, #ddd, #f2f2f2);
  animation: loading 2s infinite linear;
}
```

![](https://miro.medium.com/max/700/1*3Cbco2eycVKNwQLLbJJCYg.gif)

전체 콘텐츠를 감싸는 요소에 그래디언트 배경을 만들었는데 이상해 보인다. 그라데이션 기둥이 통째로 보이는데 회색 영역에만 표시되었으면 좋겠다.

이를 위해 모든 회색 요소에 의사 클래스를 추가한다.

![](https://miro.medium.com/max/700/1*yLZRqnFNtsN_3QuQleyaNA.gif)

그라데이션 배경이 속한 요소는 아래 스타일을 가져야 한다.

## 비교

이제 스켈레톤이 존재하는 경우와 존재하지 않는 경우의 두 가지 사례를 비교해보자.

![](https://miro.medium.com/max/700/1*cliigGKN7ktpBaMJiSvJYQ.gif)

데이터를 가져오는 동안 흰색 배경을 보여주는 것보다는 스켈레톤 컴포넌트가 훨씬 더 좋아보이는 것은 당연하다. 흰색 배경대신 로딩 바를 사용할 수도 있을 것이다.

## 소스코드 sandbox

이 예제의 소스코드는 [sandbox]()에서 볼 수 있다.

## 결론

물론 스켈레톤 UI를 채택하는 것이 항상 최고의 해결책은 아니다. 중요한 것은 사이트에서 어떤 UI 컴포넌트가 좋게 보이는지 아는 것이다. 때로는 로딩바로도 충분하지만 더 많은 것이 필요할 수도 있다.
