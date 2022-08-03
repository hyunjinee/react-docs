# Recoil atomFamily를 통해 여러 개의 Atom 관리하기(with Typescript)

상태관리를 위해 Recoil을 도입한 프로젝트에서, 앱 전역에서 공통적으로 사용할 모달을 관리해야하는 상황이 있었다.

각 모달은 모두 공통적인 속성(보임 여부, 열렸을 때 실행할 작업, 제목 등)을 가지고 있으면서도, 앱 내에서는 서로 다른 모달이 여러개 생성되어야 했다. 이를 위해 Recoil의 atomFamily API를 이요해 각각의 모달을 개별적인 Atom으로 관리하기로 결정했다.

본 글은 atomFamily에 대한 소개와 함께 타입스크립트에서 atomFamily와 selectorFamily API를 이용해 같은 성격을 가진 다수의 Atom을 관리하는 과정을 정리했다.

## Recoil의 atomFamily API

상태 관리를 할 때 다수의 객체를 배열의 형태로 관리해야하는 경우가 있다.

예를 들어, 여러개의 게시글을 저장하기 위해 Post[] 형태의 Atom인 postList를 정의하고, 해당 Atom에 여러개의 Post를 저장해 관리할 수 있다.

```ts
const postList = atom<Post[]>({
  key: "postList",
  default: [],
  ...
})
```

하지만 이 패턴은 Redux와 동일하게 한개의 post에 변경이 일어날 때마다 postList Atom에 매번 새로운 배열이 할당되어야하고, postList를 이용해 여러개의 post를 렌더링할 때 결국 별도의 memoization이 필요하다.

또한 Recoil의 사상에 따르면 Atom은 고유한 key를 갖는, 컴포넌트가 구독할 수 있는 가장 작은 단위의 상태값(원자값)이 되도록 구성하는 것이 이상적이다.

![image](https://user-images.githubusercontent.com/63354527/182623518-89c4a85b-b718-43d8-89ad-cd63f2d94a20.png)

따라서 한 배열 안에 여러개의 post를 관리하는 패턴은 작동엔 문제가 없으나 Recoil의 설계에는 정확하게 들어맞지 않는 패턴이다.

참고로 Recoil의 공식문서에서도,

> 서로 다른 요소(element)를 개별 Atom으로 관리하고, 각 컴포넌트에서는 필요한 개별 Atom만을 구독함으로써 렌더링에서의 이점이 생긴다.

라고 소개한다.

그렇다고 할지라도 아래와 같이 Post 타입을 갖는 Atom 여러개를 정의하는 것은 그다지 바람직하지 않다.

```ts
const post1Atom = atom<Post>({
  key: "post1",
  ...
})

const post2Atom = atom<Post>({
  key: "post2",
  ...
})

...
```

이 경우에 atomFamily API를 사용할 수 있다. atomFamily는 동일한 형태의 atom을 생성해주는 팩토리 함수를 제공한다.(정확히 말하면 팩토리 함수를 리턴하는 함수를 리턴) 이름에서 혼동이 올 수 있지만, atomFamily는 그 자체로 Atom을 관리할 수 있는 배열을 제공해주는 API는 아니다. 즉, atomFamily를 호출할 때마다 지정한 형식의 Atom을 생성해내게 된다.

## AtomFamily 정의

atomFamily를 활용해 모달을 정의해보자. 다음과 같이 modalsAtomFamily를 정의한다.

```js
const modalsAtomFamily = atomFamily<ModalInfo, ModalId>({
  key: "modalsAtomFamily",
  default: (id) => ({
    id,
    isOpen: false,
    title: "",
  }),
})
```

atomFamily는 일반적인 Atom과 동일하게, 파라미터 객체의 default 영역에 이 atomFamily로 만들어질 Atom의 초기값을 지정할 수 있다. 다만 atom API를 이용한 Atom 생성과의 차이점은 default 값이 특정한 파라미터를 받는 함수가 될 수 있다는 점이다.

위 코드에서는 모들을 구분하기 위해 id라는 파라미터를 받게 했고, 이 id 값을 이용해 만들어질 Atom의 id값을 정해준다.

또한 타입을 정의하기 위해서 설정한 제네릭의 각 값의 역할은 다음과 같다.

1. ModalInfo: 생성될 Atom의 타입
2. ModalId: default 값이 파라미터를 받는 함수인 경우, 그 파라미터의 타입, 이 프로젝트에서는 모달의 id로 지정할 수 있는 값이 정해져 있다.

이제 이 atomFamily를 이용해 다음과 같이 Atom을 계속 생성해낼 수 있다.

```js
const [myModal, setMyModal] = useRecoilState(modalsAtomFamily("myModal"))
const [yourModal, setYourModal] = useRecoilState(modalsAtomFamily("yourModal"))
```

## key관리의 필요성

이와 같이 atomFamily를 이용하면 동일한 형식의 Atom을 쉽게 여러번 만들 수 있지만, 또 다른 문제점이 존재한다. atomFamily는 Atom을 생성해주는 팩토리 함수일 뿐이기에, 지금까지 이 atomFamily를 이용해서 어떤 Atom을 만들었는지 전혀 알 수 없다. 예를 들어 현재 열려있는 모달들을 한번에 닫는 등의 작업등은 atomFamily만으로는 수행할 수 없을 것이다. 따라서 atomFamily와 동시에 atomFamily를 통해 생성된 Atom의 key를 별도로 관리해주는 작업이 필요하다.(위 예시에서는 ModalId)

이를 위해 modalIdsAtom이라는 Atom을 만들어 atomFamily를 이용해 모달 Atom을 생성할 때마다 해당 모달의 ModalId를 추가한다.

```ts
export const modalIdsAtom = atom<ModalId[]>({
  key: "modalIdsAtom",
  default: [],
})
```

결국 새 모달을 생성하고자 할 때에는 아래와 같이 두 단계의 작업이 필요하다는 의미입니다.

```ts
const setModalIdsAtom = useSetRecoilState(modalIdsAtom)

/* 1. atomFamily로 모달 Atom 생성 */
const myModal = modalsAtomFamily("myModal")

/* 2. 생성한 Atom의 key를 별도의 배열에 넣기 */
setModalIdsAtom((prev) => [...prev, modalIdsAtom])
```

이 작업은 별도의 hook(이를 테면 useModal)등으로 분리할 수도 있지만, recoil의 selector로 해결했다.

## Recoil의 selectorFamily API

Recoil에서 selector는 redux와 동일하게 특정한 Atom을 기반으로 해서 파생된 상태 (derived state)를 만들어 낸다. 하지만 차이점이 있다면 Recoil의 selector는 set값을 이용해 쓰기 가능한 상태 (writable state)를 정의할 수 있다. set은 특정한 타입의 파라미터를 받는데, 이 타입의 파라미터를 이용해 다른 Recoil Atom을 업데이트하는 용도로 사용할 수 있다.

여기서 selector와 selectorFamily의 관계는 atom과 atomFamily와의 관계와 동일하다. 즉, selectorFamily는 한 파라미터를 받아 이 파라미터를 이용해 작업을 수행하는 selector를 리턴하는 팩토리 함수를 리턴한다.

selectorFamily가 set을 이용해 다른 Atom을 업데이트할 수 있다는 점에 착안하여, 모달을 생성하고 모달 ID를 관리하는 작업을 아래와 같이 정리할 수 있다.

```ts
export const modalsSelectorFamily = selectorFamily<ModalInfo, ModalId>({
  key: "modalsSelectorFamily",

  get:
    (modalId) =>
    ({ get }) =>
      get(modalsAtomFamily(modalId)),

  set:
    (modalId) =>
    ({ get, set, reset }, modalInfo) => {
      if (modalInfo instanceof DefaultValue) {
        reset(modalsAtomFamily(modalId))
        set(modalIdsAtom, (prevValue) =>
          prevValue.filter((item) => item !== modalId)
        )

        return
      }

      set(modalsAtomFamily(modalId), modalInfo)
      set(modalIdsAtom, (prev) => Array.from(new Set([...prev, modalInfo.id])))
    },
})
```

1. get
   useRecoilState나 useRecoilValue를 통해 modalsSelectorFamily("myModal")의 값을 얻어오면 get(modalsAtomFamily(modalId))에 의해 modalsAtomFamily("myModal")의 값을 가져온다. 따라서 myModal이라는 키를 가진 atomFamily가 기존에 생성된 적이 있었다면 그 값을 읽어올 것이다.
   하지만 get 내부의 syntax를 살펴보면 생기는 의문이 있다. atomFamily는 Atom을 생성하는 팩토리 함수를 리턴하는데, get(modalsAtomFamily(modalId))와 같이 작성한다면 ID가 동일한 Atom을 매번 새롭게 생성하는 것이 아닐까요? 사실은 그렇지 않다. Recoil은 동일한 atomFamily로 생성된 Atom들을 구분하기 위해 atomFamily에 넘겨준 파라미터를 내부적인 ID로 이용한다.  
    가령, 아래와 같이 작성했을 때

   ```ts
   const myModal = useRecoilValue(modalsAtomFamily("myModal"))
   const notMyModal = useRecoilValue(modalsAtomFamily("myModal"))
   ```

   myModal과 notMyModal은 같은 Atom 값을 가리키게 된다.

2. set
   set값에서는 첫번째 파라미터로 set 내부에서 다른 Recoil 상태를 읽거나 업데이트할 때 사용하는 get,set,reset함수를 가져온다. 두번째 파라미터는 특정한 값을 받는데, 이 값을 이용해 다른 상태를 읽거나 업데이트할 수 있다.
   먼저 set으로 받은 파라미터의 타입이 DefaultValue인지 체크한다. useResetRecoilState를 이용해 Atom을 기본값으로 초기화하려는 경우이. 이 경우 전달받은 reset함수를 이용해 modalsAtomFamily(modalId)를 초기화하고, 동시에 ModalId 키를 관리하고 있는 modalIdsAtom에서 해당 modalId를 제거한다.

   ```ts
   ...
   if (modalInfo instanceof DefaultValue) {
   /* 키가 modalId인 Atom 리셋 */
   reset(modalsAtomFamily(modalId))
   /_ 해당 modalId 제거 _/
   set(modalIdsAtom, (prevValue) => prevValue.filter((item) => item !== id))
   return
   }
   ...
   ```

   set으로 받은 파라미터 타입이 DefaultValue가 아니라면 useSetRecoilState훅 등을 통해 값을 할당하려는 경우이다. 전달받은 set함수를 이용해 modalsAtomFamilly(modalId)를 원하는 값(set의 두번째 파라미터로 받은 값)으로 설정하고, 마찬가지로 modalIdsAtom에서 해당 modalId를 추가해준다. 다만, 같은 modalId에 여러 번 값을 할당하더라도 같은 키는 한번만 추가되어야 하므로 Set 객체를 이용했다.

   ```ts
   ...
   set(modalsAtomFamily(modalId), modalInfo)
   set(modalIdsAtom, (prev) => Array.from(new Set([...prev, modalInfo.id])))
   ...

   ```

## 사용하기

최종적으로 modalsSelectorFamily를 이용해 앱 내 모달을 제어하는 useModal 훅을 아래와 같이 작성할 수 있다.

```ts
const useModal = (modalId: ModalId) => {
  const [modal, setModal] = useRecoilState(modalsSelectorFamily(modalId))
  const resetModal = useResetRecoilState(modalsSelectorFamily(modalId))

  ...

  const openModal = () => {
    setModal((current) => ({ ...current, isOpen: true }))
  }

  const hideModal = () => {
    setModal((current) => ({ ...current, isOpen: false }))
  }

  const closeModal = () => {
    resetModal()
  }

  ...

  return { modal, setModal, openModal, hideModal, destroyModal, ... }
}
...

```

앞서 작성한 modalsSelectorFamily를 이용하여 모달을 생성하고, 열림/닫힘 상태를 설정하거나, 혹은 모달의 데이터를 초기화 시키는 로직을 모듈화할 수 있다.

실제 컴포넌트에서는

```ts
...
const loginModal = useModal("loginModal")

loginModal.openModal()
...

```

처럼 사용해 모달을 제어할 수 있을 것이다. 물론 서로 다른 페이지에서 loginModal 키를 이용해 useModal을 여러 번 호출 하더라도 atomFamily의 특성에 의해 모두 같은 모달을 가리키게 될 것이다.
