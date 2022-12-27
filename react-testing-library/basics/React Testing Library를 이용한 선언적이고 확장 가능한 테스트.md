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

이건 웹 브라우저에서 애플리케이션을 실행하는 실제 환경과 유사한 환경에서 테스트 케이스가 실행된다는 것을 의미한다. 테스트 환경이 사용자가 애플리케이션을 사용하는 환경과 비슷할수록 테스트를 더욱 신뢰할 수 있다.

필자가 React Testing Library를 선호하는 또 다른 큰 이유는 테스트가 사용자가 앱과 상호작용하는 방식과 유사해야 한다는 기본적인 라이브러리의 철학때문이다. 사용자는 애플리케이션을 사용할 때 state와 props와 상호작용한다는 사질을 알지 못한다. 함수 컴포넌트에서 훅을 사용하는지, 클래스 컴포넌트와 함께 고차 컴포넌트를 사용하는지 신경쓰지 않는다. 그저 인터페이스(버튼, 입력, 모달 등)을 보고 상호작용할 뿐이다.

그래서 올바른 props나 state가 컴포넌트에서 변경되었는지 대신하는 대신 React Testing Library는 사용자가 보고 수행하는 작업을 테스트하도록 설계 되었다. 따라서 접근 가능한 사용자 인터페이스를 구축하고 HTML을 구성할 때 모범 사례를 준수할 수 있다.

## 철학 적용하기

그렇다면 React Testing Library의 철학이 예제 컴포넌트에서 어떻게 적용되고 무얼 테스트해야하는지 어떻게 알 수 있을까? 사용자가 이 컴포넌트와 상호작용하는 방식을 생각해보자.

### 테스트 케이스 1

앱이 로딩되면 사용자가 가장 먼저 보게 되는 것은 무엇일까? 아마 이름과 성을 입력하는 제목과 21세 이상인지 묻는 체크박스, 그리고 취소, 제출 버튼이 될 것이다. 필자는 항상 사용자가 처음에 보게될 것을 테스트하는 기본 테스트 케이스 작성을 하는 것을 좋아한다.

### 테스트 케이스 2

사용자가 다음에 할 법한 상호작용은 Form을 작성하는 것이다. 따라서 사용자가 Form을 작성하기 시작하면 21세 이상입니까?라는 메시지와 체크박스가 표시된다. 사용자가 체크박스를 클릭하면 좋아하는 음료를 입력할 수 있도록 조건부로 또 다른 입력이 나타난다. 이건 테스트해야하는 별도의 코드의 분기를 나타낸다.

이 테스트가 어떻게 useState의 사용을 직접적으로 테스트하지 않는지 눈여겨보라. 우리는 내부 state가 true나 false로 바뀌는 것이 아닌 사용자가 올바른 정보를 보는지 테스트하려는 것이다. useReducer나 다른 상태관리 솔루션을 사용하도록 리팩터링해도 테스트는 변경할 필요가 없다.

### 테스트 케이스 3, 4

사용자가 이 컴포넌트에서 마지막으로 할 수 있는 것은 취소 또는 제출을 클릭하는 것이다. 이 컴포넌트는 부모 컴포넌트에서 취소 또는 제출 버튼이 클릭될 때의 콜백함수를 넘겨주도록 설계되어있다. 그러므로, 이 테스트 케이스는 이전 테스트 케이스와 약간 다르다. 사용자가 보는 것을 테스트하는 것이 아니라 특정 함수를 호출해 사용자의 작업에 내부적으로 올바르게 반응하는지를 테스트하고자 한다.

## 선언적 프로그래밍을 사용하여 테스트 작성하기

테스트 컴포넌트를 설정하고, React Testing Library가 무엇인지 논의했으며, React Testing Library의 철학을 적용하여 테스트 케이스를 만들었다. 이제 실제 테스트 케이스를 작성해보자.

개발자가 다음과 같은 테스트를 작성하는 것을 자주 볼 수 있다.

```js
it("뭔가 수행한다.", async () => {
  const onSubmit = jest.fn()
  const onCancel = jest.fn()
  const result = render(<ComplexForm onSubmit={onSubmit} onCancel={onCancel} />)

  expect(result.getByLabelText("First Name")).toBeInTheDocument()
  expect(result.getByLabelText("Last Name")).toBeInTheDocument()

  await act(async () => {
    userEvent.click(result.getByLabelText("Over 21?"))
  })

  expect(result.getByLabelText("Favorite Drink?")).toBeInTheDocument()
})
```

이 테스트는 본질적으로는 잘못된 것이 없고 컴포넌트가 정말 간단하고 1~2개의 테스트만 필요하다면 완벽하다. 다만 컴포넌트가 복잡해지고 단일 컴포넌트에 대해 5, 10개 또는 15개 이상의 테스트 케이스를 갖기 시작할 때 문제가 된다.

이렇게 생긴 모든 테스트의 경우 테스트 파일이 클 뿐 아니라 다른 개발자와 미래의 자신이 테스트에서 무슨 일이 일어나고 있는지 빠르게 이해하기 어려울 것이다. 테스트 진행상황을 이해하기 위해서는 각 코드 라인을 주의 깊게 읽어야 하기 때문이다.

대신 테스트가 선언적이라면 어떨까? 하는 일을 보여주는 테스트를 작성하는 대신 사용자의 의도를 설명하는 테스트를 작성하면 어떨까? 무슨 뜻인지 예시를 들어보겠다. 위의 테스트 케이스 예시는 선언적 프로그래밍을 이용해 다시 작성할 수 있다.

```js
it("뭔가 수행한다.", async () => {
  const { FirstNameInput, LastNameInput, clickIsOver21, FavoriteDrinkInput } =
    renderComplexForm()

  expect(FirstNameInput()).toBeInTheDocument()
  expect(LastNameInput()).toBeInTheDocument()

  await clickIsOver21()

  expect(FavoriteDrinkInput()).toBeInTheDocument()
})
```

테스트가 더 읽기 쉽고 이해하기 쉽지 않은가? 함수를 읽기만해도 현재 상황을 즉시 파악할 수 있다. 이름과 성 입력이 document에 있는지 확인한다. "그런 다음 21세 이상입니까?" 체크박스를 클릭한 다음 즐겨 찾는 음료 입력이 document에 있는지 확인한다.

이 테스트는 훨씬 읽기 쉬울 뿐 아니라 renderComplexForm 함수에서 내보낸 테스트 헬퍼를 다른 테스트 케이스에서 재사용할 수 있다. 따라서 총 테스트 케이스가 10개 또는 20개라면 훨씬 적은 코드를 작성하고 반복해야 하며 가독성을 크게 높여야한다. 다른 개발자가 반년 후 이 컴포넌트에 기능을 추가해야하고 테스트를 업데이트해야하는 경우 테스트를 훨씬 쉽게 업데이트할 수 있다.

이런 식으로 테스트를 작성하면 대규모 프로젝트에서 매우 원활하게 확장되고 복잡한 컴포넌트를 더 쉽게 테스트할 수 있다.

## ComplexForm 컴포넌트에 대한 테스트 작성

마지막으로 이 테스트 방법을 ComplexForm 컴포넌트에 적용하고 위에 작성한 4가지 테스트 케이스에 대한 실제 테스트를 작성해보자.

```js
import React from "react";
import userEvent from "@testing-library/user-event";
import { act, render } from "@testing-library/react";
import ComplexForm, { ComplexFormProps } from "./ComplexForm";

/**
 * 이건 모든 테스트에서 호출되는 테스트 설정이다.
 *
 * 테스트 설정 함수를 만들면 테스트 케이스에 대해 작성해야 하는 반복 코드의 양이
 * 줄어들고 테스트중인 컴포넌트와 상호작용하기 위한 선언적 테스트 헬퍼를 설정할 수 있다.
 */
function renderComplexForm(props?: Partial<ComplexFormProps>) {
  /* 제출과 취소 버튼을 위한 mock 콜백 함수를 설정한다. */
  const onSubmit = jest.fn();
  const onCancel = jest.fn();

  /* React Testing Library를 사용해 컴포넌트를 렌더링한다. */
  const result = render(<ComplexForm onSubmit={onSubmit} onCancel={onCancel} {...props} />);

  /* 다음 7개의 함수는 컴포넌트에서 공통 DOM 요소를 가져오기 위한 헬퍼 함수이다. */

  const Heading = () => result.getByText("Welcome, Zerry");

  const FirstNameInput = () => result.getByLabelText("First Name");

  const LastNameInput = () => result.getByLabelText("Last Name");

  const IsOver21Input = () =>
    result.getByLabelText("Are you at least 21 years old?");

  const FavoriteDrinkInput = () => result.queryByLabelText("What's your favorite drink?");

  const CancelButton = () => result.getByText("Cancel");

  const SubmitButton = () => result.getByText("Apply");

  /* 다음 6개의 함수는 DOM 요소와 상호작용하기 위한 헬퍼 함수이다. */

  function changeFirstName(name: string) {
    userEvent.type(FirstNameInput(), name);
  }

  function changeLastName(name: string) {
    userEvent.type(LastNameInput(), name);
  }

  function changeFavoriteDrinkInput(name: string) {
    userEvent.type(FavoriteDrinkInput() as HTMLElement, name);
  }

  async function clickIsOver21() {
    await act(async () => {
      userEvent.click(IsOver21Input());
    });
  }

  function clickSubmit() {
    userEvent.click(SubmitButton());
  }

  function clickCancel() {
    userEvent.click(CancelButton());
  }

  /*
    마지막으로 이 유틸리티 렌더 함수에서 모든 함수와 상수를 내보낸다. 이를 통해 모든
    테스트 케이스에서 필요한 것을 얻을 수 있다.
  */
  return {
    result,
    onSubmit,
    changeFirstName,
    changeLastName,
    clickIsOver21,
    clickSubmit,
    clickCancel,
    FirstNameInput,
    LastNameInput,
    IsOver21Input,
    SubmitButton,
    CancelButton,
    Heading,
    FavoriteDrinkInput,
    changeFavoriteDrinkInput,
    onCancel,
  };
}

describe("<ComplexForm />", () => {
  it("기본 필드를 렌더링해야 한다.", async () => {
    const {
      FirstNameInput,
      LastNameInput,
      IsOver21Input,
      SubmitButton,
      Heading,
      FavoriteDrinkInput,
      CancelButton
    } = renderComplexForm();

    // 헤더
    expect(Heading()).toBeInTheDocument();
    // 입력
    expect(FirstNameInput()).toBeInTheDocument();
    expect(LastNameInput()).toBeInTheDocument();
    expect(IsOver21Input()).toBeInTheDocument();
    expect(FavoriteDrinkInput()).not.toBeInTheDocument();
    // 버튼들
    expect(CancelButton()).toBeInTheDocument();
    expect(SubmitButton()).toBeInTheDocument();
  });

  it("21세 이상 체크 여부에 따라 좋아하는 음료 입력을 토글해야한다.", async () => {
    const { clickIsOver21, FavoriteDrinkInput } = renderComplexForm();

    expect(FavoriteDrinkInput()).not.toBeInTheDocument();

    await clickIsOver21();

    expect(FavoriteDrinkInput()).toBeInTheDocument();

  });

  it("취소 버튼이 클릭되면 onCancel 함수가 호출되야 한다.", async () => {
    const { clickCancel, onCancel } = renderComplexForm();

    clickCancel();

    expect(onCancel).toHaveBeenCalled();
  });

  it("form 값으로 onSubmit을 호출해야 한다.", async () => {
    const {
      changeFirstName,
      changeLastName,
      clickIsOver21,
      changeFavoriteDrinkInput,
      clickSubmit,
      onSubmit
    } = renderComplexForm();

    changeFirstName('Zerry');
    changeLastName('Hogan');
    await clickIsOver21();
    changeFavoriteDrinkInput('Bourbon');
    clickSubmit();

    expect(onSubmit).toHaveBeenCalledWith({
      first_name: 'Zerry',
      last_name: 'Hogan',
      is_over_21: true,
      favorite_drink: 'Bourbon',
    });
  });
});
```

나눠서 설명하겠다.

1. 먼저 컴포넌트에 대한 렌더링 함수를 만든다. render 함수는 React Testing Library를 사용해 컴포넌트를 렌더링하고 테스트 케이스를 위한 헬퍼함수를 내보내는 역할을 한다. 렌더링 기능을 위한 별도의 파일을 만들어 테스트 케이스로 가져올 수도 있다.
2. 각 테스트 케이스는 renderComplexForm을 호출해 특정 테스트 케이스에 필요한 유틸리티 함수를 가져온다.
3. 입력 값을 변경하기 위해 changeFirstname 테스트 헬퍼 함수를 만들었다. 사용자가 상호작용하는 방식을 시뮬레이션하고 테스트에서 어떤 일이 일어나는지 명백히 보여준다.
4. renderComplexForm 함수는 props 인수를 받는다. 컴포넌트가 컴포넌트의 UI 또는 사용자가 보는 것을 변경하는 props를 받는 경우가 많다. 각 테스트 케이스가 props를 넘기도록 허용해 서로 다른 상호작용을 테스트할 수도 있다.
5. onSubmit과 onCancel props를 위해 jest mock 함수를 사용하고 있다. jest mock 함수는 함수가 호출되었는지, 몇번이나 호출되었는지, 그리고 어떤 인수로 호출되었는지 테스트하는데 유용하다. 마지막 두 테스트 케이스에서 버튼 클릭시 적절한 콜백함수가 호출되었는지 테스트하기 위해 jest mock 함수를 사용했다.

결과적으로 더욱 읽기 쉽고 확장 가능하며 오래 지속될 수 있는 테스트를 만들었다고 믿는다. 이 테스트로 돌아와서 입력을 추가하고 테스트 코드를 구문 분석하여 새 테스트 적용 범위를 추가할 위치를 알 필요없이 몇 분 안에 테스트를 업데이트할 수 있다.
