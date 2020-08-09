## Hooks解决了什么问题

1. 组件之间复用状态逻辑很难
React没有提供将一些将可复用性行为“附加”到组件的途径（例如，把组件连接到store）。常见的解决方案为 ”render prop“和”高阶组件“。但是它们需要**重新组织你的组件结构**，这会使你的代码变得非常难以理解和维护。我们可以在React DevTools中观察到，由 providers，consumers，高阶组件，render props 等其他抽象层组成的组件会形成“嵌套地狱”。
而Hooks则解决了：**为共享状态逻辑提供更好的原生途径**的问题。

举个例子，项目中有一些接口请求需要加上loading并根据这个loading加一些样式，例如tab, 分页，下拉框或者下拉刷新这类的请求。但是这个状态逻辑其实都是一样的。都是变true和false。那么，在使用了hooks之后，我们可以这么写一个hooks。
```jsx
// utils/hooks.js
const useLoading = (fn, deps) => {
    const [loading, setLoading] = useState(false);
    const [data, setData] = useState(nul);

    const request = () => {
        setLoading(true);
        fn()
        .then(setData)
        .finally(() => {
            setLoading(false)
        })
    }

    useEffect(() => {
        request();
    }, deps)

    return { data, loading };
}

```


2. 复杂组件变得难以理解

期初，组件都很简单，但是逐渐被状态逻辑和副作用占据。每个生命周期常常包含一些不相关的逻辑。例如，组件常常componentDidMount和compoentDidUpdate中获取数据。但是，同一个componentDidMout中可能也包含很多其他的逻辑，如事件监听，而之后需要在componentWillUnMount中清除。相互关联且需要对照修改的代码被进行了拆分，而完全不相关的代码却在同一个方法中组合在一起。

且在多数情况下，由于一个状态逻辑在一个组件中无处不在，不可能将组件拆分成更小的粒度。虽然可以通过引入状态管理工具来结合使用。但是，这回增加额外的抽象概念。

而hooks可以将相互关联的部分拆分为更小的函数，而非强制按照生命周期划分。

如下，我们将一个鼠标移动事件封装在一个hooks中, 而useEffect hooks则可以代替`componentDidMount`、`compoentDidUpdate`和`componentWillUnMount`。使得逻辑更加的清晰
```jsx
// useMousePosition.js
const useMousePosition = () => {
  const [position, setPosition] = useState({x: 0, y: 0 })
  useEffect(() => {
    const updateMousePosition = (e) => {
      setPosition({ x: e.clientX, y: e.clientY })
    }
    document.addEventListener('mousemove', updateMousePosition)
    return () => {
      document.removeEventListener('mousemove', updateMousePosition)
    }
  })
  return position
}

// App.js
import React from 'react'
import useMousePosition from './useMousePosition'

const App = () => {
  const {x, y} = useMousePosition()
  return (
    <div>
      <div>x: {x}</div>
      <div>y: {y}</div>
    </div>
  )
}
```

那么，如果我们使用Class的话，需要使用render prop 获取children prop 才能解决以上这种横切问题（封装状态或行为共享给其他需要相同状态的组件）
```jsx
class MousePosition extends React.Component {
  constructor(props) {
    super(props);

    this.state = {
      x: 0,
      y: 0
    }
  }

  updateMousePosition = (e) => {
    this.setState({ x: e.clientX, y: e.clientY })
  }

  componentDidMount() {
    document.addEventListener('mousemove', this.updateMousePosition)
  }

  componentWillUnMount() {
    document.removeEventListener('mousemove', this.updateMousePosition)
  }

  render() {
    return this.props.render(this.state);
  }
}

const App = () => {
  return (
    <div>
      <MousePosition 
        render={ (data) => {
          return [
            <div>x: {data.x}</div>,
            <div>y: {data.y}</div>
          ]
        }}
      />
    </div>
  )
}
```

或者HOC
```jsx
function withMousePosition(Component) {
  class Comp extends React.Component {
    render() {
      return (
        <MousePosition 
          render={ data => (<Component {...this.props} data={data} />)}
        />
      )
    }
  }

  return Comp;
}

class MyApp extends React.Component {
  render () {
    const { data } = this.props;
    return (
      <div>
        <div>x: {data?.x}</div>
        <div>y: {data?.y}</div>
      </div>
    )
  }
}

const App = withMousePosition(MyApp)

ReactDOM.render(
  <App />,
  document.getElementById('root')
);
```

3. 难以理解的Class
除了代码的复用和管理困难外，学习Class你还需要理解this的指向问题。现在，我们可以通过箭头函数来避免this的指向问题。

但是，我们需要知道，在不支持箭头函数之前，事件函数都需要通过bind这类方法来修正this的指向。否则，在调用一个事件处理函数时，this会指向window；


## 总结

1. hooks解决了class组件难以解决的状态或行为复用问题
2. hooks可以有效避免在书写复杂组件时使用render prop或者高阶组件所带来的的让组件难以理解的问题。
3. 减少了多个生命周期的概念，学习和使用成本降低
4. 避免class中this的指向问题