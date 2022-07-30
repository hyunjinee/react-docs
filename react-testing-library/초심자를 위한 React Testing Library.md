# 초심자를 위한 React Testing Library

React Testing Library (이하 RTL)은 구현 기반의 테스트 도구인 Enzyme의 대안으로 자리 잡은 테스트 도구이다. 따라서 RTL은 세부적인 구현사항보다는 실제 사용자 경험과 유사한 방식의 테스트를 작성할 것을 권고한다.

예를 들어 <div>Hello World</div>라는 코드가 있다면, RTL은 div 태그를 사용하는지보다 Hello World 메시지가 브라우저에 노출이 되는지 파악하는 것을 더 중요하다고 본다. 이와 같이 구현보다 기능에 초점을 맞춘 테스트 방식은 신뢰도를 높임과 동시에 코드 리팩토링 시 테스트 코드 수정 빈도를 줄일 수 있다.

> The more your tests resemble the way your software is used, the more confidence they can give you. - RTL 공식문서

> RTL은 이름 그대로 React 컴포넌트를 테스트 하기 위해 만들어진 도구이기 때문에, 기본적으로 CRA에 내장되어 있다.. 만약 CRA를 사용하지 않고 개발 환경을 직접 설정한다면 npm install --save-dev @testing-library/react 명령어를 이용해 설치하면 된다.

RTL을 처음 접할 떄 RTL을 Jest의 대안으로 혼동하는 경우가 있다.

두 도구는 React내에서 테스트를 진행할 때 같이 사용되기에 상호 보완 관계라고 할 수 있다. 엄밀히 말아면 RTL이 Jest를 포함하는 구조이다. 전반적으로 Jest를 통해 기능 테스트를 진행할 수는 있지만, React 컴포넌트를 렌더링하고 테스트하기 위해서는 몇가지 기능이 더 필요하기 때문이다.

- Jest: 자체적인 test runner와 test util 제공
- RTL: Jest + React 컴폰너트의 test util 제공

그렇기에 리액트 내부의 테스트 코드 작성 방식은 Jest의 방식과 상당 부분 유사하다.

## 테스트 코드 작성해보기

Counter 컴포넌트를 테스트 하며 테스트 코드를 작성할 때 가장 기초가 되는 부분부터 알아보자.

TDD는 테스트 코드부터 짜는 것이 정석이지만 이해를 돕기위해 앱 코드와 같이 진행해보자. Counter 컴포넌트의 테스트 항목을 정의해보자.

- 컴포넌트가 렌더링 되는지
- - 버튼을 눌렀을 때 count가 증가하는지
- - 버튼을 눌렀을 때 count가 감소하는지

요구사항을 도출했으니 해당 기능을 단계별로 테스트해보자.

```js
// Counter.js
import React, { useState } from "react"

const Counter = () => {
  const [count, setCount] = useState(0)

  const decrement = () => setCount(count - 1)

  const increment = () => setCount(count + 1)

  return (
    <>
      <div>{count}</div>
      <button onClick={decrement}>-</button>
      <button onClick={increment}>+</button>
    </>
  )
}

export default Counter
```

```js
// Counter.test.js
import { render } from "@testing-library/react"

import Counter from "./Counter"

describe("Counter test", () => {
  it("should render Counter", () => {
    render(<Counter />)
  })
})
```

- test runner는 테스트 실행시 \*.test.js suffix를 가진 파일을 자동으로 탐색하며 테스트를 진행
- describe: 같은 맥락의 테스트들을 그룹화
- it: 개별 테스트를 수행
- render: 테스트를 위해 특정 컴포넌트를 jsdom에 렌더링.

테스트를 단위별로 끊어보면 대개 컴포넌트를 띄우고 -> 특정액션을 발생시킨 후 -> 결과를 확인하는 패턴으로 이어진다.

이 패턴을 충족하기 위해 어떤 API들이 필요한가 알아보자.

### Query

렌더링된 DOM 노드에 접근하여 엘리먼트를 가져오는데 사용한다. 다양한 메서드가 존재하지만 사용방법은 어렵지 않다. 예컨대 getAllByRole 메서드를 섹션별로 나누어 보자

- get(쿼리 타입)/All(타겟의 개수)/ByRole(타겟 유형)

각 섹션에 들어갈 수 있는 타입은 다음과 같다.

- get: 동기적으로 처리되며 타겟을 찾지 못할시 에러를 던진다.
- find: 비동기적으로 처리되며 타겟을 찾지 못할시 에러를 던진다.
- query: 동기적으로 처리되며 타겟을 찾지 못할시 null을 반환한다.

코드를 작성하다보면 이벤트, 데이터 fetching등 비동기적인 처리가 필요하거나 해당 타겟을 찾지 못하더라도 에러를 던지지 않게끔 처리해야하는 경우가 있다. 때문에 각 경우에 맞게 쿼리 타입을 지정해줘야한다.

### 타겟의 개수

만약 다수의 엘리먼트가 탐색되는 상황이라면 뒤에 All을 붙인다.

타겟 유형 (우선 순위별 배치)

- ByRole
- ByLabelText
- ByPlaceholderText
- ByText
- ByAltText
- ByTitle
- ByAltText
- ByTestId

다시 Counter 예시로 돌아가 이벤트 수행을 위한 타겟 엘리먼트를 얻어오겠습니다.

```js
import { render, screen } from "@testing-library/react"

import Counter from "./Counter"

describe("Counter test", () => {
  it("should render Counter", () => {
    render(<Counter />)

    // 두 쿼리 모두 같은 element 탐색(문자열 대신 정규식 탐색도 가능)
    screen.getByRole("button", { name: "+" })
    screen.getByText("+")
  })
})
```

여기서 유의할 점은, 공식문서에서 getByRole을 권장한다고 해서 테스트를 위해 role을 억지로 선언할 필요는 없다. 기본적으로 몇몇 HTML 시맨틱 태그는 이미 implicit role을 가지고 있기 때문에 해당 role과 두번째 인자로 들어가는 옵션 객체를 통해 찾고자 하는 타입을 좁힐 수 있다. 만약 implicit role 파악이 어렵다면 RTL 에러 로그를 통해 현재 DOM 노드 내에서 사용 가능한 role을 제안받을 수 있다. 또 위에서 나열한 다른 메서드를 통해 타겟을 얻는 방법을 적절히 취사선택 하면 된다.

### Action

사용자가 브라우저에서 이벤트를 발생시키는 것처럼, RTL은 얻어온 타겟을 이용해 이벤트를 발동시킬 수 있다.

```js
import { fireEvent, render, screen } from "@testing-library/react"

import userEvent from "@testing-library/user-event"

it("should render Counter", () => {
  render(<Counter />)

  const target = screen.getByRole("button", { name: "+" })
  // 두 api 모두 이벤트 호출 용도로 사용됩니다.
  fireEvent.click(target)
  userEvent.click(target)
})
```

이벤트를 호출하는 두 api중 되도록 userEvent를 사용할 것을 권장한다. 이 API는 내부적으로 fireEvent를 사용하며 실제 유저의 행동과 흡사한 기능을 추가로 제공하기 때문이다.

### Asynchronous Test

컴포넌트 내부의 동작은 항상 동기적으로만 이루어지는 것은 아니다. axios, fetch 그 외의 Web API를 통해 다양한 비동기 상황을 접할 수 있는데 이번엔 가상으로 데이터를 불러오는 Fetch 컴포넌트와 그 테스트 코드를 작성해보자.

```js
// Fetch.js
import React, { useEffect, useState } from "react"

const mockFetch = () =>
  new Promise((resolve) => {
    setTimeout(
      resolve([
        { id: "1", name: "Person1" },
        { id: "2", name: "Person2" },
      ]),
      100
    )
  })

const Fetch = () => {
  const [loading, setLoading] = useState(true)
  const [result, setResult] = useState(null)

  useEffect(() => {
    const loadResult = async () => {
      const fetchedResult = await mockFetch()

      setResult(fetchedResult)
      setLoading(false)
    }

    loadResult()
  }, [])

  return (
    <div>
      {loading && <div>Loading</div>}
      {result && (
        <ul>
          {result.map(({ id, name }) => (
            <li key={id}>{name}</li>
          ))}
        </ul>
      )}
    </div>
  )
}

export default Fetch
```

```js
// Fetch.test.js
import "@testing-library/jest-dom"

import { render, screen } from "@testing-library/react"

import Fetch from "./Fetch"

describe("Fetch", () => {
  it("should load result", async () => {
    render(<Fetch />)

    expect(screen.getByText("Loading")).toBeInTheDocument()

    // 1)
    expect(await screen.findAllByRole("listitem")).toHaveLength(2)
    // 2)
    await waitFor(() => {
      expect(screen.getAllByRole("listitem")).toHaveLength(2)
    })
  })
})
```

Fetch 컴포넌트는 <div>Loading</div>를 먼저 렌더링하고 데이터를 불러오면 해당 데이터를 렌더링한다. 따라서 각 시점에 맞게 assertion을 작성해주어야 한다.

데이터를 확인하는 방법은 위에서 작성한 것처럼 두가지의 방법을 택할 수 있지만, 가독성을 위해 find 메서드를 사용할 것을 권장하는 추세이다. 사실 이외에도 axios,fetch를 통한 api 요청 테스트를 위해 다양한 방법론은 존재한다.

프론트엔드 쪽에서 테스트를 할 때는 백엔드 api 통신이 성공적으로 이루어지는지가 아닌 api 응답결과(대기, 성공, 실패)에 따라 컴포넌트가 어떻게 반응하는지에 초점을 두고 테스트 전략을 짜는 것이 바람직하다. 즉 테스트의 성공 여부가 환경에 의존성을 두지 않게끔 그 범위를 명확하게 가져가는 것이 좋다.

## Summary

테스트 코드를 작성하다보면 테스트를 어디까지 진행해야할지 의문이 들 때가 있다. 수시로 바뀌는 기획으로 인해 테스트 자체가 무의미 해질 수도 있고, 떄로는 테스트를 위한 테스트를 진행한다는 느낌을 받기 떄문이다. 그럼에도 불구하고 테스트의 중요성을 무시할 수는 없기에 스스로 점검하며 적절한 중간점을 찾으려는 자세가 필요하다.
