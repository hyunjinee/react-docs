# Thinking in React

React는 JavaScript로 규모가 크고 빠른 웹 애플리케이션을 만드는 가장 좋은 방법입니다. React는 Facebook과 Instagram을 통해 확장성을 입증했습니다.

React의 가장 멋진 점 중 하나는 앱을 설계하는 방식입니다. 이 문서를 통해 React로 상품들을 검색할 수 있는 데이터 테이블을 만드는 과정을 함께 생각해 봅시다.

## 목업으로 시작하기

JSON API와 목업을 디자이너로부터 받았다고 가정해 봅시다. 목업은 다음과 같을 것 입니다.

![image](https://user-images.githubusercontent.com/63354527/190615243-3e8dea7f-8d9c-4347-9813-42065559a19a.png)

JSON API는 아래와 같은 데이터를 반환합니다.

```json
[
  {category: "Sporting Goods", price: "$49.99", stocked: true, name: "Football"},
  {category: "Sporting Goods", price: "$9.99", stocked: true, name: "Baseball"},
  {category: "Sporting Goods", price: "$29.99", stocked: false, name: "Basketball"},
  {category: "Electronics", price: "$99.99", stocked: true, name: "iPod Touch"},
  {category: "Electronics", price: "$399.99", stocked: false, name: "iPhone 5"},
  {category: "Electronics", price: "$199.99", stocked: true, name: "Nexus 7"}
];
```

## 1단계: UI를 컴포넌트 계층 구조로 나누기

우리가 할 첫 번째 일은 모든 컴포넌트(와 하위 컴포넌트)의 주변에 박스를 그리고 그 각각에 이름을 붙이는 것입니다. 디자이너와 함께 일한다면 이것들을 이미 정해두었을 수 있으니 한번 대화해보세요! 디자이너의 Photoshop 레이어 이름이 React 컴포넌트 이름이 될 수 있습니다.

하지만 어떤 것이 컴포넌트가 되어야 할지 어떻게 알 수 있을까요? 우리가 새로운 함수나 객체를 만들 때처럼 만드시면 됩니다. 단 한기지 테크닉은 단일 책임 원칙입니다. 이는 하나의 컴포넌트는 한 가지 일을 하는게 이상적이라는 원칙입니다. 하나의 컴포넌트가 커지게 된다면 이는 보다 작은 하위 컴포넌트로 분리되어야 합니다.

주로 JSON 데이터를 유저에게 보여주기 때문에, 데이터 모델이 적절하게 만들어졌다면, UI(컴포넌트 구조)가 잘 연결될 것 입니다. 이는 UI와 데이터 모델이 같은 인포메이션 아키텍처(information architecture)를 가지는 경향이 있기 때문입니다. 각 컴포넌트가 데이터 모델의 한 조각을 나타내도록 분리해주세요.

![image](https://user-images.githubusercontent.com/63354527/190618103-8bf89a50-249c-4e3e-9248-c2e604f2236f.png)

다섯개의 컴포넌트로 이루어진 앱을 한번 봅시다. 각각의 컴포넌트에 들어간 데이터는 이탤릭체로 표기했습니다. 이미지의 숫자는 아래 숫자에 해당됩니다.

1. FilterableProductTable (orange): 모든 것이 들어있습니다.
2. SearchBar (blue): 모든 유저의 입력(user input)을 받습니다.
3. ProductTable (green): 유저의 입력을 기반으로 데이터 콜렉션을 필터링해서 보여줍니다.
4. ProductCategoryRow (하늘색): 각 카테고리(category)의 헤더를 보여줍니다.
5. ProductRow (빨강): 각각의 제품(product)에 해당하는 행을 보여줍니다.

ProductTable을 보면 "Name"과 "Price"레이블을 포함한 테이블 헤더만을 가진 컴포넌트는 없습니다. 이 같은 경우, 데이터를 위한 독립도니 컴포넌트를 생성할지 생성하지 않을지는 선택입니다. 이 예시에서는 ProductTable의 책임인 데이터 컬렉션이 렌덜이의 일부이기 때문에 ProductTable을 남겨두었습니다. 그러나 이 헤더가 복잡해지면 (즉, 정렬을 위한 기능을 추가하는 등) ProductTableHeader 컴포넌트를 만드는 것이 더 합리적일 것 입니다.

이제 목업에서 컴포넌트를 확인하였으므로 이를 계층 구조로 나열해봅시다. 모형의 다른 컴포넌트 내부에 나타나는 컴포넌트의 계층 구조의 자식으로 나타냅니다.

- FilterableProductTable
  - SearchBar
  - ProductTable
    - ProductCategoryRow
    - ProductRow

## 2단계: React로 정적인 버전 만들기

```tsx
class ProductCategoryRow extends React.Component {
  render() {
    const category = this.props.category
    return (
      <tr>
        <th colSpan="2">{category}</th>
      </tr>
    )
  }
}

class ProductRow extends React.Component {
  render() {
    const product = this.props.product
    const name = product.stocked ? (
      product.name
    ) : (
      <span style={{ color: "red" }}>{product.name}</span>
    )

    return (
      <tr>
        <td>{name}</td>
        <td>{product.price}</td>
      </tr>
    )
  }
}

class ProductTable extends React.Component {
  render() {
    const rows = []
    let lastCategory = null

    this.props.products.forEach((product) => {
      if (product.category !== lastCategory) {
        rows.push(
          <ProductCategoryRow
            category={product.category}
            key={product.category}
          />
        )
      }
      rows.push(<ProductRow product={product} key={product.name} />)
      lastCategory = product.category
    })

    return (
      <table>
        <thead>
          <tr>
            <th>Name</th>
            <th>Price</th>
          </tr>
        </thead>
        <tbody>{rows}</tbody>
      </table>
    )
  }
}

class SearchBar extends React.Component {
  render() {
    return (
      <form>
        <input type="text" placeholder="Search..." />
        <p>
          <input type="checkbox" /> Only show products in stock
        </p>
      </form>
    )
  }
}

class FilterableProductTable extends React.Component {
  render() {
    return (
      <div>
        <SearchBar />
        <ProductTable products={this.props.products} />
      </div>
    )
  }
}

const PRODUCTS = [
  {
    category: "Sporting Goods",
    price: "$49.99",
    stocked: true,
    name: "Football",
  },
  {
    category: "Sporting Goods",
    price: "$9.99",
    stocked: true,
    name: "Baseball",
  },
  {
    category: "Sporting Goods",
    price: "$29.99",
    stocked: false,
    name: "Basketball",
  },
  {
    category: "Electronics",
    price: "$99.99",
    stocked: true,
    name: "iPod Touch",
  },
  {
    category: "Electronics",
    price: "$399.99",
    stocked: false,
    name: "iPhone 5",
  },
  { category: "Electronics", price: "$199.99", stocked: true, name: "Nexus 7" },
]

const root = ReactDOM.createRoot(document.getElementById("container"))
root.render(<FilterableProductTable products={PRODUCTS} />)
```

이제 컴포넌트 계층 구조가 만들어졌으니 앱을 실제로 구현해봅시다. 가장 쉬운 방법은 데이터 모델을 가지고 UI를 렌더링은 되지만 아무 동작도 없는 버전을 만들어보는 것입니다. 이처럼 과정을 나누는 것이 좋은데 정적 버전을 만드는 것은 생각은 적게 필요하지만 타이핑은 많이 필요로 하고, 상호작용을 만드는 것은 생각은 많이 해야하지만 타이핑은 적게 필요로 하기 때문입니다. 나중에 보다 자세히 살펴봅시다.

데이터 모델을 렌더링하는 앱의 정적 버전을 만들기 위해 다른 컴포넌트를 재사용하는 컴포넌트를 만들고 props를 이용해 데이터를 전달해줍니다. props는 부모가 자식에게 데이터를 넘겨줄 때 사용할 수 있는 방법입니다. 정적 버전을 만들기 위해 state를 사용하지 마세요. state는 오직 상호작용을 위해, 즉 시간이 지남에 따라 바뀌는 것에 사용합니다. 우리는 앱의 정적 버전을 만들고 있기 때문에 지금은 필요하지 않습니다.

앱을 만들 때 하향식(top-down)이나 상향식(bottom-up)으로 만들 수 있습니다. 다시 말해 계층 구조의 상층부에 있는 컴포넌트(즉, FilterableProductTable부터 시작하는 것)부터 만들거나 하층부에 있는 컴포넌트(ProductTable) 부터 만들 수도 있습니다. 간단한 예시에서는 보통 하향식으로 만드는게 쉽지만 프로젝트가 커지면 상향식으로 만들고 테스트를 작성하면서 개발하기가 더 쉽습니다.

이 단계가 끝나면 데이터 렌더링을 위해 만들어진 재사용 가능한 컴포넌트들의 라이브러리를 가지게 됩니다. 현재는 앱의 정적 버전이기 때문에 컴포넌트는 render() 메서드만 가지고 있을 것 입니다. 계층 구조의 최상단 컴포넌트 (FilterableProductTable)는 prop으로 데이터 모델을 받습니다. 데이터 모델이 변경되면 root.render()를 다시 호출해서 UI가 업데이트됩니다. UI가 어떻게 업데이트되고 어디에 변경해야하는지 알 수 있습니다. React의 단방향 데이터 흐름(one-way data flow)(또는 단방향 바인딩)는 모든 것을 모듈화하고 빠르게 만들어줍니다.

## 3단계: UI state에 대한 최소한의 (하지만 완전한) 표현 찾아내기

UI를 상호작용하게 만들려면 기반 데이터 모델을 변경할 수 있는 방법이 있어야합니다. 이를 React는 state를 통해 변경합니다.

애플리케이션을 올바르게 만들기 위해서는 애플리케이션에서 필요로 하는 변경 가능한 state의 최소 집합을 생각해보아야 합니다. 여기서 핵심은 중복 배제 원칙입니다. 애플리케이션이 필요로하는 가장 최소한의 state를 찾고 이를 통해 나머지 모든 것들이 필요에 따라 그때 그때 계산되도록 만들어야합니다. 예를 들어 Todo 리스트를 만든다고 하면, Todo 아이템을 저장하는 배열만 유지하고 Todo아이템의 개수를 표현하는 state는 별도로 만들지 마세요. Todo 갯수를 렌더링해야한다면 Todo 아이템 배열의 길이를 가져오면 됩니다.

예시 애플리케이션 내 데이터들을 생각해봅시다. 애플리케이션은 다음과 같은 데이터를 가지고 있습니다.

- 제품의 원본 목록
- 유저가 입력한 검색어
- 체크 박스의 값
- 필터링 된 제품들의 목록

각각 살펴보고 어떤게 state가 되어야 하는 지 살펴봅시다. 이는 각 데이터에 대해 아래 세 가지 질문을 통해 결정할 수 있습니다.

1. 부모로부터 props를 통해 전달됩니까? 그러면 확실히 state가 아닙니다.
2. 시간이 지나도 변하지 않나요? 그러면 확실히 state가 아닙니다.
3. 컴포넌트 안의 다른 state나 props를 가지고 계산 가능한가요? 그러면 state가 아닙니다.

제품의 원본 목록은 props를 통해 전달되므로 state가 아닙니다. 검색어와 체크박스는 state로 볼 수 있는데 시간이 지남에 따라 변하기도 하면서 다른 것들로부터 계산될 수 없기 때문입니다. 그리고 마지막으로 필터링된 목록은 state가 아닙니다.

결과적으로 애플리케이션은 다음과 같은 state를 가집니다.

- 유저가 입력한 검색어
- 체크박스의 값

## 4단계: State가 어디에 있어야 할 지 찾기

```ts
class ProductCategoryRow extends React.Component {
  render() {
    const category = this.props.category
    return (
      <tr>
        <th colSpan="2">{category}</th>
      </tr>
    )
  }
}

class ProductRow extends React.Component {
  render() {
    const product = this.props.product
    const name = product.stocked ? (
      product.name
    ) : (
      <span style={{ color: "red" }}>{product.name}</span>
    )

    return (
      <tr>
        <td>{name}</td>
        <td>{product.price}</td>
      </tr>
    )
  }
}

class ProductTable extends React.Component {
  render() {
    const filterText = this.props.filterText
    const inStockOnly = this.props.inStockOnly

    const rows = []
    let lastCategory = null

    this.props.products.forEach((product) => {
      if (product.name.indexOf(filterText) === -1) {
        return
      }
      if (inStockOnly && !product.stocked) {
        return
      }
      if (product.category !== lastCategory) {
        rows.push(
          <ProductCategoryRow
            category={product.category}
            key={product.category}
          />
        )
      }
      rows.push(<ProductRow product={product} key={product.name} />)
      lastCategory = product.category
    })

    return (
      <table>
        <thead>
          <tr>
            <th>Name</th>
            <th>Price</th>
          </tr>
        </thead>
        <tbody>{rows}</tbody>
      </table>
    )
  }
}

class SearchBar extends React.Component {
  render() {
    const filterText = this.props.filterText
    const inStockOnly = this.props.inStockOnly

    return (
      <form>
        <input type="text" placeholder="Search..." value={filterText} />
        <p>
          <input type="checkbox" checked={inStockOnly} /> Only show products in
          stock
        </p>
      </form>
    )
  }
}

class FilterableProductTable extends React.Component {
  constructor(props) {
    super(props)
    this.state = {
      filterText: "",
      inStockOnly: false,
    }
  }

  render() {
    return (
      <div>
        <SearchBar
          filterText={this.state.filterText}
          inStockOnly={this.state.inStockOnly}
        />
        <ProductTable
          products={this.props.products}
          filterText={this.state.filterText}
          inStockOnly={this.state.inStockOnly}
        />
      </div>
    )
  }
}

const PRODUCTS = [
  {
    category: "Sporting Goods",
    price: "$49.99",
    stocked: true,
    name: "Football",
  },
  {
    category: "Sporting Goods",
    price: "$9.99",
    stocked: true,
    name: "Baseball",
  },
  {
    category: "Sporting Goods",
    price: "$29.99",
    stocked: false,
    name: "Basketball",
  },
  {
    category: "Electronics",
    price: "$99.99",
    stocked: true,
    name: "iPod Touch",
  },
  {
    category: "Electronics",
    price: "$399.99",
    stocked: false,
    name: "iPhone 5",
  },
  { category: "Electronics", price: "$199.99", stocked: true, name: "Nexus 7" },
]

const root = ReactDOM.createRoot(document.getElementById("container"))
root.render(<FilterableProductTable products={PRODUCTS} />)
```

좋습니다. 이제 앱에서 최소한의 필요한 state가 뭔지 찾아냈습니다. 다음으로는 어떤 컴포넌트가 state를 변경하거나 소유할지 찾아야합니다.

기억하세요: React는 항상 컴포넌트 계층 구조를 따라 아래로 내려가는 단방향 데이터 흐름을 따릅니다. 어떤 컴포넌트가 어떤 state를 가져야 하는지 바로 결정하기 어려울 수 있습니다. 많은 초보자가 이 부분을 가장 어려워합니다. 아래 과정을 따라 결정해보세요.

애플리케이션이 가지는 각각의 state에 대해서

- state 기반으로 렌더링하는 모든 컴포넌트를 찾으세요.
- 공통 소유 컴포넌트 (common owner component)를 찾으세요.(계층 구조 내에서 특정 state가 있어야하는 모든 컴포넌트들의 상위에 있는 하나의 컴포넌트)
- 공통 혹은 더 상위에 있는 컴포넌트가 state를 가져야 합니다.
- state를 소유할 적절한 컴포넌트를 찾지 못하였다면, state를 소유하는 컴포넌트를 하나 만들어서 공통 소유 컴포넌트의 상위 계층에 추가하세요.

이 전략을 애플리케이션에 적용해봅시다.

- ProductTable은 state에 의존한 상품 리스트를 필터링해야하고 SearchBar는 검색어와 체크박스의 상태를 표시해주어야 합니다.
- 공통 소유 컴포넌트는 FilterableProductTable 입니다.
- 의미 상으로도 FilterableProductTable이 검색어와 체크박스의 여부를 가지는 것이 타당합니다.

좋습니다. state를 FilterableProductTable에 두기로 했습니다. 먼저 인스턴스 속성인 this.state = {filterText: '', inStockOnly: false}를 FilterableProductTable의 constructor에 추가하여 애플리케이션의 초기 상태를 반영합니다. 이제 애플리케이션의 동작을 볼 수 있습니다. filterText를 "ball"로 설정하고 앱을 새로고침 해보세요. 데이터 테이블이 올바르게 업데이트 된 것을 볼 수 있습니다.

## 5단계 역방향 데이터 흐름 추가하기

```ts
class ProductCategoryRow extends React.Component {
  render() {
    const category = this.props.category
    return (
      <tr>
        <th colSpan="2">{category}</th>
      </tr>
    )
  }
}

class ProductRow extends React.Component {
  render() {
    const product = this.props.product
    const name = product.stocked ? (
      product.name
    ) : (
      <span style={{ color: "red" }}>{product.name}</span>
    )

    return (
      <tr>
        <td>{name}</td>
        <td>{product.price}</td>
      </tr>
    )
  }
}

class ProductTable extends React.Component {
  render() {
    const filterText = this.props.filterText
    const inStockOnly = this.props.inStockOnly

    const rows = []
    let lastCategory = null

    this.props.products.forEach((product) => {
      if (product.name.indexOf(filterText) === -1) {
        return
      }
      if (inStockOnly && !product.stocked) {
        return
      }
      if (product.category !== lastCategory) {
        rows.push(
          <ProductCategoryRow
            category={product.category}
            key={product.category}
          />
        )
      }
      rows.push(<ProductRow product={product} key={product.name} />)
      lastCategory = product.category
    })

    return (
      <table>
        <thead>
          <tr>
            <th>Name</th>
            <th>Price</th>
          </tr>
        </thead>
        <tbody>{rows}</tbody>
      </table>
    )
  }
}

class SearchBar extends React.Component {
  constructor(props) {
    super(props)
    this.handleFilterTextChange = this.handleFilterTextChange.bind(this)
    this.handleInStockChange = this.handleInStockChange.bind(this)
  }

  handleFilterTextChange(e) {
    this.props.onFilterTextChange(e.target.value)
  }

  handleInStockChange(e) {
    this.props.onInStockChange(e.target.checked)
  }

  render() {
    return (
      <form>
        <input
          type="text"
          placeholder="Search..."
          value={this.props.filterText}
          onChange={this.handleFilterTextChange}
        />
        <p>
          <input
            type="checkbox"
            checked={this.props.inStockOnly}
            onChange={this.handleInStockChange}
          />{" "}
          Only show products in stock
        </p>
      </form>
    )
  }
}

class FilterableProductTable extends React.Component {
  constructor(props) {
    super(props)
    this.state = {
      filterText: "",
      inStockOnly: false,
    }

    this.handleFilterTextChange = this.handleFilterTextChange.bind(this)
    this.handleInStockChange = this.handleInStockChange.bind(this)
  }

  handleFilterTextChange(filterText) {
    this.setState({
      filterText: filterText,
    })
  }

  handleInStockChange(inStockOnly) {
    this.setState({
      inStockOnly: inStockOnly,
    })
  }

  render() {
    return (
      <div>
        <SearchBar
          filterText={this.state.filterText}
          inStockOnly={this.state.inStockOnly}
          onFilterTextChange={this.handleFilterTextChange}
          onInStockChange={this.handleInStockChange}
        />
        <ProductTable
          products={this.props.products}
          filterText={this.state.filterText}
          inStockOnly={this.state.inStockOnly}
        />
      </div>
    )
  }
}

const PRODUCTS = [
  {
    category: "Sporting Goods",
    price: "$49.99",
    stocked: true,
    name: "Football",
  },
  {
    category: "Sporting Goods",
    price: "$9.99",
    stocked: true,
    name: "Baseball",
  },
  {
    category: "Sporting Goods",
    price: "$29.99",
    stocked: false,
    name: "Basketball",
  },
  {
    category: "Electronics",
    price: "$99.99",
    stocked: true,
    name: "iPod Touch",
  },
  {
    category: "Electronics",
    price: "$399.99",
    stocked: false,
    name: "iPhone 5",
  },
  { category: "Electronics", price: "$199.99", stocked: true, name: "Nexus 7" },
]

const root = ReactDOM.createRoot(document.getElementById("container"))
root.render(<FilterableProductTable products={PRODUCTS} />)
```

지금까지 우리는 계층 구조 아래로 흐르는 props와 state의 함수로써 앱을 만들었습니다. 이제 다른 방향의 데이터 흐름을 만들어볼 시간입니다. 계층 구조의 하단에 있는 폼 컴포넌트에서 FilterableProductTable의 state를 업데이트 할 수 있어야 합니다.

React는 전통적인 양방향 데이터 바인딩과 비교하면 더 많은 타이핑을 필요로 하지만 데이터 흐름을 명시적으로 보이게 만들어서 프로그램이 어떻게 동작하는지 파악할 수 있게 도와줍니다.

4단계의 예시에서 체크하거나 키보드의 타이핑을 할 경우 React가 입력을 무시하는 것을 확인할 수 있습니다. 이는 input 태그의 value 속성이 항상 FilteralbeProductTable에서 전달된 state와 동일하도록 설정했기 때문입니다.

우리가 원하는 것이 무엇인지를 한번 생각해봅시다. 우리는 사용자가 폼을 변경할 때마다 사용자의 입력을 반영할 수 있도록 state를 업데이트 하기를 원합니다. 컴포넌트는 그 자신의 state만 변경할 수 있기 때문에 FilterableProductTable는 SearchBar에 콜백을 넘겨서 state가 업데이트되어야 할 때마다 호출되도록 할 것 입니다. 우리는 input에 onChange 이벤트를 사용해서 알림을 받을 수 있습니다. FilterableProductTable에서 전달된 콜백은 setState()를 호출하고 앱이 업데이트될 것 입니다.

## 이게 전부입니다.

이 글을 통해 React를 가지고 애플리케이션과 컴포넌트를 만드는 데에 대한 사고방식을 얻어갈 수 있기를 바랍니다. 이전보다 더 많은 타이핑을 해야할 수 있지만, 코들르 쓸 일 보다 읽을 일이 더 많다는 사실을 기억하세요. 모듈화되고 명시적인 코드는 읽을 때 조금 덜 어렵습니다. 큰 컴포넌트 라이브러리를 만들게 되면 이 명시성과 모듈성에 감사할 것이며 코드 재사용성을 통해 코드 라인이 줄어들기 시작할 것 입니다.

## 읽으면서

이 글을 기반으로 함수형 컴포넌트로 위 글을 리팩토링했습니다.

[함수형 컴포넌트로 리팩토링한 코드](https://github.com/hyunjinee/ui-engineering/tree/master/thinking-in-react)
