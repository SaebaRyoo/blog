### [虚拟DOM是啥？以及diff算法原理](https://www.infoq.cn/article/react-dom-diff)

### react事件绑定

在react中使用的是自己实现的合成事件，以达到在不同浏览器和平台上都具有相同属性的一致性。
合成事件通过事件代理，将所有的监听函数都绑定在了document上。每当有事件触发，都会映射到对应的元素组件上，以 **提高性能** 。

**注意** react组件中方法的this会在绑定给事件监听器的时候丢失（这是js本身的特性），所以需要bind

#### 常见的绑定方法

1. 在render中使用bind。
好处：不需要写多余的绑定, 能够矫正方法的this指向
坏处：当组件更新时，每次都会重新bind一个事件处理函数。在大型应用中的性能消耗很明显

```jsx
    class C extends React.Component {
        handleClick() {
            console.log(this);
        }
        render() {
            return (<button onClick={this.handleClick.bind(this)}></button> )
        }
    }
```

2. 在constructor中bind
好处：不会对性能造成潜在威胁
坏处：增加代码的书写量

```jsx
    class C extends React.Component {
        constructor() {
            this.handleClick = this.handleClick.bind(this);
        }
        handleClick() {
            console.log(this)
        }
        render() {
            return (<button onClick={this.handleClick}></button>  )
        }
    }
```

3. 使用箭头函数( () => {} )
好处： 不会对性能造成潜在威胁，简洁的书写方式.
坏处： 需要使用babel才能使用

```jsx
    class C extends React.Component {
        handleClick = () => {
            console.log(this);
        }
        render() {
            return (<button onClick={this.handleClick}></button>  )
        }
    }
```

4. 使用::
好处：不会重复绑定，简洁的书写方式
坏处：需要babel, 不能传参 

```jsx

    class C extends React.Component {
        handleClick() {
            console.log(this);
        }
        render() {
            return (<button onClick={::this.handleClick}></button>  )
        }
    }
```


### 生命周期

#### 挂载阶段

constructor -> static getDerivedStateFromProps(props, state) -> render -> componentDidMount()

#### 更新阶段

static getDerivedStateFromProps(props, state) -> shouldComponentUpdate(nextProps, nextState) -> render -> getSnapshotBeforeUpdate(prevProps, prevState) -> componentDidUpdate(prevProps, prevState, snapshot)

#### 卸载阶段

componentWillUnMount

### 函数式编程，纯函数

### React创建组件的方式

1. 函数组件(无状态组件)

顾名思义，无状态组件的意思就是组件没有自己的state
```jsx
function Person(props) {
    return (
        <div>{props.name}</div> 
        <div>{props.age}</div> 
    )
}

class Child exntends React.Component {
    render () {
        return (
            <Person name='william' age='23' />
        )
    }
}

```

2. 类组件(有状态组件)

```jsx

class Person exntends React.Component {
    state = {
        name: 'william',
        age: 23
    }
    render () {
        return (
            <div>{this.state.name}</div> 
            <div>{this.state.age}</div> 
        )
    }
}

```
### 组件性能优化

1. shouldComponentUpdate(nextProps, nextState)
子组件通过对比props或者state返回true或者false，判断是否更新组件，减少不必要的渲染。
但是由于引用类型是存在相同的内存，比如`this.props.obj === nextProps.obj`，该对比会一直返回true，这样可以使用JSON.stringify、immutable.js。
不过还是不要过早的优化性能。因为每次对比也会损耗性能，不恰当的性能优化反而会更加的损耗性能。

2. [PureComponent](https://juejin.im/entry/5934c9bc570c35005b556e1a)
该类是15.3版本引入的。只需要将`Component`换成`PureComponent`即可。可以减少不必要的render渲染，少些`shouldComponentUpdate`提高性能。

当组件更新时，如果组件的 props 和 state 都没发生改变，render 方法就不会触发，省去 Virtual DOM 的生成和比对过程，达到提升性能的目的。具体就是 React 自动帮我们做了一层浅比较：

```
if (this._compositeType === CompositeTypes.PureClass) {
  shouldUpdate = !shallowEqual(prevProps, nextProps)
  || !shallowEqual(inst.state, nextState);
}
```
而 shallowEqual 又做了什么呢？会比较 Object.keys(state | props) 的长度是否一致，每一个 key是否两者都有，并且是否是一个引用，也就是只比较了第一层的值，确实很浅，所以深层的嵌套数据是对比不出来的。

3. [不可变数据](https://juejin.im/entry/5934c9bc570c35005b556e1a)

4. 当有列表数据需要渲染时，给每一项元素都加上一个`唯一的key`,方便react diff对比

5. React.memo

该方法和`PureComponent`类作用相似，但是它适用于函数组价。且返回值和`shouldComponentUpdate`方法相反, `true`为不需要更新, `false`表示需要更新。

默认情况下不传第二个参数，和PureComponent一样，使用浅对比
```
const MyComponent = React.memo(function MyComponent(props) {})
```

如果需要自行控制对比过程，需要如下写法

```javascript
function MyComponent(props) {
  /* 使用 props 渲染 */
}
function areEqual(prevProps, nextProps) {
  /*
  如果把 nextProps 传入 render 方法的返回结果与
  将 prevProps 传入 render 方法的返回结果一致则返回 true，
  否则返回 false
  */
}
export default React.memo(MyComponent, areEqual);
```

### [如何设计一个号组件](https://juejin.im/post/5c49cff56fb9a049bd42a90f)

### 在哪里进行网络请求？为什么？
可以在constructor、componentDidMount、componentDidUpdate中发起网络请求。
最常使用的是componentDidMount。

1. constructor
constructor能够更快的发起请求，组件正处于初始化的过程，不需要等待虚拟DOM渲染完毕就可以响应。且在constructor中调用setState会被合并，并不会造成多次更新.

2. componentDidMount
一般都会在这个生命周期中发起网络请求。因为在这个生命周期中，该组件已经渲染完成。如果需要获取到数据后就操作真实dom的话，就必须在这个生命周期中。因为在constructor中调用可能会出现网络请求响应很快，但是虚拟DOM并没有渲染完成。导致无法获取到真实的DOM节点

3. componentDidUpdate
当组件作为子组件，且props发生变化，需要更新组件数据时，可以在这个生命周期中发起网络请求

### [调用setState之后发生了什么](https://zhuanlan.zhihu.com/p/65627731)

### [refs](https://juejin.im/post/5c9d783cf265da60d0005390#heading-8)

refs是react提供的一种访问真实DOM的方法

1. const instance = React.createRef();
通过 instance.current访问DOM
```javascript
class TestComp extends React.Component {
  constructor(props) {
    super(props);
    // 创建一个 ref 来存储 DOM元素 input
    this.textInput = React.createRef();
  }
  focusEvent = () => {
    // 直接通过原生API访问输入框获取焦点事件
    this.textInput.current.focus();
  }
  render() {
    return (
      <div>
        <input type="text" ref={this.textInput} />
        <input type="button" value="获取文本框焦点事件" onClick={this.focusEvent}/>
      </div>
    );
  }
}
```
2. ref回调
```javascript

class TestComp extends React.Component {
  constructor(props) {
    super(props);
    // 创建一个 ref 来存储 DOM元素 input
    this.textInput = null;
  }
  focusEvent = () => {
    // 直接通过原生API访问输入框获取焦点事件
    this.textInput.current.focus();
  }
  render() {
    return (
      <div>
        <input type="text" ref={(textInput) => this.textInput = textInput} />
        <input type="button" value="获取文本框焦点事件" onClick={this.focusEvent}/>
      </div>
    );
  }
}
```

3. 如何将DOM通过refs暴露给父组件
在一些情况下，我们希望父组件能够引用子组件中的DOM节点，用户触发焦点或者测量子DOM 节点的大小或者位置。虽然我们可以通过向子组件添加 ref的方式来解决，但这并不是一个理想的解决方案，因为我们只能获取组件实例而不是 DOM节点。并且它还在函数组件上无效。

在react 16.3 或者更高版本中，我们推荐使用 ref 转发的方式来实现以上操作。

ref 转发使得组件可以像暴露自己的 ref一样暴露子组件的 ref。

Ref forwarding 是一种自动将ref 通过组件传递给其子节点的技术。下面我们通过具体的案例来演示一下效果。
```js
const ref = React.createRef();
const BtnComp = React.forwardRef((props, ref) => {
  return (
    <div>
      <button ref={ref} className='btn'>
        { props.children }
      </button>
    </div>
  )
});

class TestComp extends React.Component {
  clickEvent() {
    if (ref && ref.current) {
      ref.current.addEventListener('click', () => {
        console.log('hello click!')
      });
    }
  }
  componentDidMount() {
    console.log('当前按钮的class为：', ref.current.className); // btn
    this.clickEvent(); // hello click!
  }
  render() {
    return (
      <div>
        <BtnComp ref={ref}>点击我</BtnComp>
      </div>
    );
  }
}


```

上述案例，使用的组件BtnComp 可以获取对底层 button DOM 节点的引用并在必要时对其进行操作，就像正常的HTML元素 button直接使用DOM一样。

**注意事项**
第二个ref参数仅在使用React.forwardRef 回调 定义组件时存在。常规函数或类组件不接收ref参数，并且在props中也不提供ref。

Ref转发不仅限于DOM组件。您也可以将refs转发给类组件实例。


**高阶组件中的refs**
高阶组件（HOC）是React中用于重用组件逻辑的高级技术，高阶组件是一个获取组件并返回新组件的函数。下面我们通过具体的案例来看一下refs如何在高阶组件钟正常使用。

```jsx

// 记录状态值变更操作
function logProps(Comp) {
  class LogProps extends React.Component {
    componentDidUpdate(prevProps) {
      console.log('old props:', prevProps);
      console.log('new props:', this.props);
    }
    render() {
      const { forwardedRef, ...rest } = this.props;
      return <Comp ref={ forwardedRef } {...rest} />;
    }
  }
  return React.forwardRef((props, ref) => {
    return <LogProps { ...props } forwardedRef={ ref } />;
  });
}

// 子组件
const BtnComp = React.forwardRef((props, ref) => {
  return (
    <div>
      <button ref={ref} className='btn'>
        { props.children }
      </button>
    </div>
  )
});

// 被logProps包装后返回的新子组件
const NewBtnComp = logProps(BtnComp);


class TestComp extends React.Component {
  constructor(props) {
    super(props);
    this.btnRef = React.createRef();

    this.state = {
      value: '初始化'
    }
  }

  componentDidMount() {
    console.log('ref', this.btnRef);
    console.log('ref', this.btnRef.current.className);
    this.btnRef.current.classList.add('cancel'); // 给BtnComp中的button添加一个class
    this.btnRef.current.focus(); // focus到button元素上
    setTimeout(() => {
      this.setState({
        value: '更新'
      });
    }, 10000);
  }

  render() {
    return (
      <NewBtnComp ref={this.btnRef}>{this.state.value}</NewBtnComp>
    );
  }
}


```

### react 16 新特性

#### [time slicing、suspens](https://www.zhihu.com/question/268028123)

### [在 React 当中 Element 和 Component 有何区别](https://reactjs.org/blog/2015/12/18/react-components-elements-and-instances.html)

### [容器组件和展示组件](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0)

### [路由实现原理](https://github.com/youngwind/blog/issues/109)

### [react的setState同步还是异步？](https://github.com/Advanced-Frontend/Daily-Interview-Question/issues/17)

### Redux，react-redux等原理

[redux原理](https://github.com/lxnxbnq/blog/issues/1)

[react-redux](https://blog.suisuijiang.com/react-redux-source-code-analyse/)

### 组件间通信

子组件获取父组件的数据，可以通过props
父组件获取子组件的数据，可以通过回调函数

### [高阶组件是什么和常见的高阶组件](https://zh-hans.reactjs.org/docs/higher-order-components.html)

具体而言，高阶组件是参数为组件，返回值为新组件的函数。

```
const EnhancedComponent = higherOrderComponent(WrappedComponent);

```
组件是将 props 转换为 UI，而高阶组件是将组件转换为另一个组件。

HOC 在 React 的第三方库中很常见，例如 Redux 的 connect 和 Relay 的 createFragmentContaine， 以及React.lazy。

现在我们手动实现一个和lazy一样的动态引入组件的高阶函数

```jsx
// 
function AsyncFunc(Comp) {
    class TempComp extends React.Component {
        state = {
            Com: null
        }
        async componentDidMount() {
            const { default: Component } = await Comp;
            this.setState({ Com: <Component {...this.props} />})
        }

        render () {
            return this.state.Com;
        }
    }
    return TempComp;
}

// 然后可以通过这样导入
const Home = AsyncFunc(() => import('./pages/Home'));

```

### [React key是干嘛的？](https://zh-hans.reactjs.org/docs/lists-and-keys.html#___gatsby)

key给与React元素一个唯一的标识，帮助React识别哪些元素改变了。以提升组件更新的性能
[为什么key是必须的](https://zh-hans.reactjs.org/docs/reconciliation.html#recursing-on-children)