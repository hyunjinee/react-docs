# React Testing Library를 이용한 선언적이고 확장 가능한 테스트

> 무죄가 입증될 때까지 모든 코드는 유죄다. - 누군가

솔직히 React 컴포넌트 테스트를 작성하는 건 아마도 당신이 제일 좋아하는 일은 아닐 것이다. 번거롭고 어렵고 성가신 것으로 느껴질 수 있다. 때로는 무엇을 테스트해야할지, 심지어는 컴포넌트를 테스트하는 방법도 모른다. 하지만 실제로 테스트는 애플리케이션의 무결성에서 매우 중요하며 올바로 실행되면 당신이 의도한 대로 애플리케이션이 동작한다는 확신을 줄 수 있다.

소규모로 React Testing Library를 사용하는 방법에 대한 예제를 제공하는 글들은 많지만 클린 테스트를 작성하는 방법과 여러 개발자들과 함께 대규모 프로젝트를 쉽게 테스트하는 방법에 대해 논의하는 글은 많지 않다.

복잡한 컴포넌트와 유동적인 부분이 많은 대규모 프로젝트에 테스트를 작성하면서 배운 점을 공유하고 싶다. 필자의 목표는 테스트 작성이 쉽게 느껴지고 코드 커버리지가 부족했던 시절 아주 오래전일인 것 마냥 React 컴포넌트를 테스트하는 방법을 보여주는 것이다.

한 번 해보자.

사용자가 조작하는 방식으로 컴포넌트와 쉽게 상호작용할 수 있는 테스트 라이브러리의 이름은 `@testing-library/user-event`이며 이 라이브러리는 테스트 케이스에서 클릭이나 입력같은 실제 사용자 이벤트를 매우 쉽게 시뮬레이션할 수 있다.

IDE에서 프로젝트를 열고 `/components/ComplexForm/ComplexForm.ts` 경로에 파일을 만들고 아래 코드를 붙여넣자.

```ts
import React, { useState, VoidFunctionComponent } from "react"

type FormJSON = Record<string, any>

export type ComplexFormProps = {
  onSubmit: (data: FormJSON) => void
  onCancel: VoidFunction
}

const ComplexForm: VoidFunction<ComplexProps> = ({ onSubmit, onCancel }) => {
  const [isOver21, setIsOver21] = useState<boolean>(false)

  const handleFormSubmit = (event: React.FormEvent<HTMLFormElement>) => {
    event.preventDefault()

    const form = event.target as HTMLFormElement
    const data = formToJSON(form.elements)

    onSubmit(data)
  }

  const handleIsOver21Change = ({
    target,
  }: React.ChangeEvent<HTMLInputElement>) => {
    const checkbox = target as HTMLInputElement
    setIsOver21(checkbox.checked)
  }

  return (
    <form id="myForm" name="myForm" onSubmit={handleFormSubmit}>
      <h1>Welcome, Zerry</h1>
      <div>
        <label htmlFor="first_name">First Name</label>
        <input type="text" id="first_name" name="first_name" />
      </div>
      <div>
        <label htmlFor="last_name">Last Name</label>
        <input type="text" id="last_name" name="last_name" />
      </div>
      <div>
        <label htmlFor="is_over_21">Are you at least 21 years old?</label>
        <input
          type="checkbox"
          id="is_over_21"
          name="is_over_21"
          checked={isOver21}
          onChange={handleIsOver21Change}
        />
      </div>
      {isOver21 && (
        <div>
          <label htmlFor="favorite_drink">What's your favorite drink?</label>
          <input type="text" id="favorite_drink" name="favorite_drink" />
        </div>
      )}
      <button type="button" onClick={onCancel}>
        Cancel
      </button>
      <button type="submit">Apply</button>
    </form>
  )
}

/**
 * form에 입력한 데이터를 검색해서 JSON 객체로 반환
 */
const formToJSON = (elements: HTMLFormControlsCollection): FormJSON => {
  return Array.from(elements).reduce((data, element: any) => {
    if (element.name) {
      const value =
        element.type === "checkbox" ? element.checked : element.value
      return {
        ...data,
        [element.name]: value,
      }
    }
    return { ...data }
  }, {} as FormJSON)
}

export default ComplexForm
```

그 다음 src 폴더의 루트에 있는 App.tsx 파일에 다음 코드를 붙여넣자.

```ts
import React from "react"
import ComplexForm from "./components/ComplexForm"

function App() {
  return <ComplexForm onSubmit={() => {}} onCancel={() => {}} />
}

export default App
```

1. ComplexForm 컴포넌트는 사용자가 상호작용할 수 있는 간단한 HTML 요소를 렌더링한다.
2. 이 Form은 사용자가 이름과 성을 입력하고 21세 이상인지 확인하도록 요청한다.
3. 사용자가 21세 이상이면 사용자에게 가장 좋아하는 음료가 무엇인지 묻는 또 다른 인풋이 렌더링된다.
4. 사용자는 Form을 작성한 후 적용 또는 취소를 클릭할 수 있다.App.tsx에서 볼 수 있든 두 버튼 모두 ComplexForm으로 전달되는 콜백함수를 실행한다.

![image](https://user-images.githubusercontent.com/63354527/189102356-31959c79-6c82-445b-9ad9-547986cf4287.png)

## 왜 React Testing Libraray인가?

React Testing Library는 컴포넌트를 테스트하기 위해 설계된 라이브러리이다. 과거에는 React 컴포넌트를 테스트하기위해 Enzyme을 사용했을 수 있다. React Testing Library가 Enzyme과 다른 점은 테스트를 렌더링할 때 React 컴포넌트의 인스턴스가 아닌 실제 DOM 노드를 사용한다는 점이다.

이건 웹 브라우저에서 애플리케이션을 실행하는 실제 환ㅕ