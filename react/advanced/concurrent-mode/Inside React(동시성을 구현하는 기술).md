# Inside React(동시성을 구현하는 기술)

## 1. 5년의 실험

리액트 팀에서 Async 렌더링이라는 개념을 소개한지 5년이 지났습니다. 그동안 어떤 변화가 있었는지 살펴보겠습니다.

<img width="1477" alt="image" src="https://user-images.githubusercontent.com/63354527/191696936-95e9e6b4-8a06-4c87-bf73-ff5bd6d1a819.png">

2017년에 동시성에 기반이되는 `Fiber` 아키텍처를 구현하였고, 함께 Async 렌더링이라는 개념을 소개했습니다. 그 후 JSConf에서 Time slicing과 Suspense라는 개념을 소개하였습니다.

그리고 이어서 Async 렌더링은 Concurrent 렌더링이라는 이름으로 바뀌었습니다. 이후 React Conf에서 lazy를 이용한 code splitting을 소개하였고, 그해 가을 동시성 모드를 2019년까지 출시하겠다는 로드맵을 발표합니다.

<img width="1466" alt="image" src="https://user-images.githubusercontent.com/63354527/191697704-98c8c430-bb4f-4af3-a97f-9beac784c1ab.png">

2019년 8월에 그 기한이 없어졌습니다. 아키텍처의 큰 변화라서 예상보다 난관이 많았기 때문입니다. 엄청난 노력끝에 21년 6월 리액트 팀은 리액트 18의 알파버전을 출시합니다.

알파버전은 서드파티 라이브러리 개발자의 피드백을 받기 위함이고, 커뮤니티가 점진적으로 적용할 수 있도록 Working Group을 설립하고 열띤 토론을 진행했습니다.

<img width="1445" alt="image" src="https://user-images.githubusercontent.com/63354527/191698400-deffd69c-ee08-4f66-830b-3bfed00eba58.png">

리액트 18에서 추가된 기능은 위와 같습니다. 새로운 기능을 구현하기 위해서 Concurrent Rendering이라는 메커니즘으로 대표되는 다음과 같은 개념을 구현하였습니다.

- 협력적 멀티태스킹
- 우선순쉬 기반 렌더링
- 스케쥴링
- 중단 (Interruptions)

### Concurrent 용어 정리

- ReactDOM.createRoot 실행시 "concurrent mode" 활성화
- 그러나 이 조건에서 반드시 "concurrent rendering"하는 것은 아님
- "concurrent features" 사용할 때만 "concurrrent rendering" 수행

## 2. 동시성으로 해결하려는 문제

concurrent rendering은 무엇이고 왜 리액트팀이 이것을 구현했을까요?

### Concurrency vs Parallelism

- 동시성은 **독립적으로 실행되는 프로세스들**의 조합이다.
- 병렬성은 **연관된 복수의 연산**들을 동시에 실행하는 것이다.
- 동시성은 여러 일을 **한꺼번에 다루는 문제**에 관한 것이다.
- 병렬성은 여러 일을 **한꺼번에 실행하는 방법**에 관한 것이다.
- 동시성이란 마음의 상태이다. 실제로는 여러 개의 스레드가 사용되지는 않지만, 겉으로 보기에는 마치 여러 개의 스레드가 사용되고 있는 것처럼 보이게 만드는 것을 의미한다. 이것이 바로 협동적인 멀티태스킹을 의미한다.

<img width="1433" alt="image" src="https://user-images.githubusercontent.com/63354527/191701239-4b70eb58-9a3a-4c99-9c37-b01698449445.png">

<img width="1469" alt="image" src="https://user-images.githubusercontent.com/63354527/191701374-797e15f6-baa6-4616-b80c-c7b21740e094.png">

동시성은 싱글 코어에서도 작동합니다. 하지만 병렬성은 반드시 멀티 코어가 필요합니다.

<img width="1463" alt="image" src="https://user-images.githubusercontent.com/63354527/191701557-dba837d4-eff3-4921-aad7-a00ae303466c.png">

동시성은 두개 이상의 태스크가 동시에 실행되는 것처럼 보이지만 병렬성에서는 두개 이상의 코어가 존재하기 때문에 실제로 동시에 실행됩니다.

동시성에서는 다른 작업으로 전환할 때 현재 작업을 저장하거나 이전 작업을 불러오는 과정이 필요한데 이를 컨텍스트 스위칭이라고 합니다.

<img width="1464" alt="image" src="https://user-images.githubusercontent.com/63354527/191702031-46cc39ec-4f44-4a66-bd6e-f802e0c18846.png">

동시성에서는 최소 두개의 논리적 통제흐름이 존재합니다. 즉 서로 독립된 두개 이상의 작업이 존재합니다. 반면에 병렬성에서는 최소 한개의 논리적 통제 흐름이 존재합니다.

<img width="1470" alt="image" src="https://user-images.githubusercontent.com/63354527/191702548-33f7b20d-b882-4687-b0fd-55faae4d57d9.png">

리액트는 이 동시성을 어떤 문제를 해결하는데 사용했을까?

<img width="1240" alt="image" src="https://user-images.githubusercontent.com/63354527/191703146-1249941c-ec2d-4388-ab57-87027dcaca4f.png">

브라우저의 메인 스레드는 싱글 스레드로서 한번에 하나의 작업만 처리할 수 있습니다. 즉, HTML을 파싱하거나 자바스크립트를 실행하거나 화면에 보이는 내용을 렌더링하는데 사용됩니다.

<img width="1251" alt="image" src="https://user-images.githubusercontent.com/63354527/191703367-98e71814-2297-4670-9c7a-08dcbe629fba.png">

그런데 브라우저의 메인스레드가 한번 작업을 시작하면 그 작업이 완료될 때 까지 멈출 수 없습니다.

React를 비롯한 대다수의 UI라이브러리 작동 방식도 이 한계에 종속될 수 밖에 없습니다. 리액트도 화면에 그리기 위한 내부 연산, 즉 렌더링을 시작해서 화면을 완성할 때까지 실행을 멈출수 없습니다. 정확히는 리액트 18이전까지는 그랬습니다. 이는 리액트 팀에서는 블로킹 렌더링이라고 부릅니다.

<img width="1248" alt="image" src="https://user-images.githubusercontent.com/63354527/191703917-abbb7079-c3d1-4246-960e-fc7a9fed7cee.png">

보통은 렌더링을 멈출 수 없더라도 큰 문제는 없습니다. 리액트의 변경-비교 알고리즘이 매우 빠르기 때문이죠. 그런데 렌더링 연산이 매우 길어진다면 어떻게 될까요?

화면이 오래 블로킹되면 사용자 경험이 악화됩니다. 이를 블로킹 렌더링이라고 합니다.

<img width="1246" alt="image" src="https://user-images.githubusercontent.com/63354527/191704454-b1ea03db-2281-4ad0-b203-8d0e8a0c07ef.png">

자바스크립트의 연산이 길어지면서 프레임이 드롭되고 입력은 한참 후에 예약됩니다.

리액트팀은 이를 동시성을 활용하여 해결하기로 했습니다.

![image](https://user-images.githubusercontent.com/63354527/192077123-0740c249-fa7e-456d-b1ff-51306fd2a3be.png)

입력으로 인한 텍스트 처리는 빠르게 처리되기를 기대하고 그것으로 인한 목록 갱신은 느리게 처리해도 괜찮습니다.

![image](https://user-images.githubusercontent.com/63354527/192077186-383b65d9-1346-40ee-89cb-2523721b7cf5.png)

리액트는 두개의 작업을 병렬로 배치하여 두개의 작업의 스택을 분리시킵니다. 동시성 렌더링에서는 하나의 스레드에서 일정 작업을 마친 후 메인스레드에게 일정 시간을 양보합니다. 이 때 브라우저는 이벤트를 처리할 수 있고 응답성을 향상 시킬 수 있습니다.

동시성 렌더링에서는 하나의 컴포넌트 렌더링을 잘게 분해하여 처리합니다.

![image](https://user-images.githubusercontent.com/63354527/192077233-b725e63c-2b64-4274-84f5-adab0130d16e.png)

![image](https://user-images.githubusercontent.com/63354527/192077343-8ae7d632-bf87-41e6-af09-50b8d855f806.png)

![image](https://user-images.githubusercontent.com/63354527/192077362-c98f4f6f-571e-4471-8621-b67c34cd5357.png)

![image](https://user-images.githubusercontent.com/63354527/192077389-65c3b281-24c6-4906-9e8f-564b05b9ac93.png)

![image](https://user-images.githubusercontent.com/63354527/192077400-7a84df66-2d0c-4a2a-a154-4a771b41172c.png)

"A", "B"를 입력한 후 "C"를 입력한 상황에서 "C"렌더링 도중에 "D"입력이 발생한다면 "C"로 인한 렌더링 작업을 일시 중단하고(낮은 순위 렌더링) 그리고 더 급한 작업인 "D"에대한 입력 렌더링을 먼저 수행합니다.

이러면 텍스트가 화면에 먼저 갱신되고, 그 다음에 Pending 상태의 목록 낮은 우선 순위 렌더링을 리베이스합니다.

하나의 차선은 고속차선, 다른 하나의 차선은 저속 차선이라고 할 수 있는데, 리액트에서는 내부적으로 이를 레인이라고 부르고 우선순위를 제어하기 위해서 사용합니다.

## 3. React 18의 동시성 기능

![image](https://user-images.githubusercontent.com/63354527/192077537-7dc8fba4-1b61-4859-ac49-7ae48fb40bdb.png)

startTransition이라는 API를 사용해 느린 차선을 만듭니다. 상태업데이트를 콜백함수로 넘기면 콜백안쪽의 상태 업데이트는 느린 차선, 즉 낮은 우선 순위를 갖게됩니다.

startTransition은 콜백함수가 바로 실행됩니다. 대신 상태변경의 우선순위가 낮아지게 됩니다. 이렇게 낮은 업데이트 우선 순위를 부여받으면 보다 중요한 업데이트에 CPU 사용을 양보하게 됩니다.

![image](https://user-images.githubusercontent.com/63354527/192077724-d88c3a04-6238-46f0-ba67-64a089c1c83e.png)

이를 통해 대규모 화면 업데이트 중에도 응답성을 유지할 수 있습니다. 상태 전환중인지, 아닌지를 알려주는 API를 통해서 전환중에 사용자에게 시각적 피드백을 줄 수 있습니다. 리액트팀에 따르면 궁극적인 방향은 네이트브 앱과같은 높은 수준의 애니메이션, 높은 수준의 응답성을 구현하는 것 이라고 합니다.

![image](https://user-images.githubusercontent.com/63354527/192077822-cd7f0958-91b2-4d1d-8cb4-d1622e55c6c3.png)

블로킹 렌더링과 동시성 렌더링의 프레임 차트는 다릅니다. 블로킹 렌더링이 하나의 긴 스택을 갖은것과 달리, 동시성 렌더링에서는 짧은 여러개의 스택이 반복되는 것을 알 수 있습니다. 각각의 스택이 비워질 때 브라우저는 입력과 같은 더 중요한 요소들을 처리할 수 있습니다.

![image](https://user-images.githubusercontent.com/63354527/192077882-a696709f-e917-4172-8cd0-38c85108d6e5.png)

startTransition은 3가지 기능을 통해 작동됩니다. 우선 중요한 일을 양보하는 yield, 우선순위가 높은 작업이 생기면 중단하기 interrupting, 그리고 최종 상태만 업데이트되도록 이전 결과를 건너뛰는 것.

setTimeout으로도 구현할 수 있지만 setTimeout과 크게 다른점은 건너뛰는 것 입니다. setTimeout으로 태스크 큐에 등록된 작업은 들어간 순서대로 처리되고 건너뛸 수 없습니다. 언젠가는 반드시 처리되어야합니다.

그러나 startTransition에서 처리된 내용은 이전결과를 건너뛰고 필요한 최종 상태를 반영합니다.

![image](https://user-images.githubusercontent.com/63354527/192078009-2d9a05b9-46b8-48ff-9e35-bee15b6be2f3.png)

![image](https://user-images.githubusercontent.com/63354527/192078079-f7447336-02c2-494a-82f8-09f959348b24.png)

startTransition이 CPU바운드의 개선이라고 하면 I/O 바운드의 개선인 Suspense 기반의 Streaming SSR을 알아보도록 하겠습니다. React 18에는 Streaming SSR과 선택적 하이드레이션이라는 기능이 들어가 있습니다.

## 4. React 동시성의 기반 기술

## 5. References
