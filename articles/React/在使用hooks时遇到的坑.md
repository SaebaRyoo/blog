# React Hooks 踩坑记录

### 只在最顶层使用Hook

React是通过Hook的调用顺序来确定state对应的`useState`。

所以，我们需要保证Hook的调用顺序在多次渲染之间保持一致。

**看一下官网的例子**

```jsx
// ------------
// 首次渲染
// ------------
useState('Mary')           // 1. 使用 'Mary' 初始化变量名为 name 的 state
useEffect(persistForm)     // 2. 添加 effect 以保存 form 操作
useState('Poppins')        // 3. 使用 'Poppins' 初始化变量名为 surname 的 state
useEffect(updateTitle)     // 4. 添加 effect 以更新标题

// -------------
// 二次渲染
// -------------
useState('Mary')           // 1. 读取变量名为 name 的 state（参数被忽略）
useEffect(persistForm)     // 2. 替换保存 form 的 effect
useState('Poppins')        // 3. 读取变量名为 surname 的 state（参数被忽略）
useEffect(updateTitle)     // 4. 替换更新标题的 effect

```

但如果我们将一个 Hook (例如 persistForm effect) 调用放到一个条件语句中,在第一次渲染中 name !== '' 这个条件值为 true，所以我们会执行这个 Hook。
但是在下一次渲染时我们可能清空了表单，表达式值变为 false。此时的渲染会跳过hook

```jsx

if (name !== '') {
  useEffect(function persistForm() {
    localStorage.setItem('formData', name);
  });
}

useState('Mary')           // 1. 读取变量名为 name 的 state（参数被忽略）
// useEffect(persistForm)  // 此 Hook 被忽略！
useState('Poppins')        // 2 （之前为 3）。读取变量名为 surname 的 state 失败
useEffect(updateTitle)     // 3 （之前为 4）。替换更新标题的 effect 失败
```

### hook的依赖项

在hook 中，`useEffect`、`useCallback`、`useMemo`等都提供第二个参数，它是一个依赖项数组（当我们不传时，每一次的状态更新都会被调用）。
当在第一个参数的回调函数中使用了props和state，就需要在依赖项中标明。不然，使用的都是初始值。

在useEffect等中，
- 当依赖项指定为 `[]` 数组时，则只会在首次更新时调用回调
- 当不指定依赖项时，则任意state更新都会触发回调
- 当指定依赖项时，则只在依赖项发生改变时执行回调

在useCallback、useMemo中，
- 当依赖项指定为 `[]` 数组时，都会执行，但是回调中的state都是初始值
- 当不指定依赖项时，则任意state更新都会触发回调
- 当指定依赖项时，则只在依赖项发生改变时执行回调

### useEffect和useLayoutEffect的区别
简单的说，就是调用时机不同，`useLayoutEffect`和原来`componentDidMount`、`componentDidUpdate`一致。
在react完成DOM更新后马上同步调用的代码，会阻塞页面渲染。而 `useEffect` 是会在整个页面渲染完才会调用的代码。

### 状态修改是异步的
```jsx
const MyComponent = () => {
  const [value, setValue] = useState(0);

  function handleClick() {
    setValue(1);
    console.log(value); // <- 0
  }

  return (
    <div>
      <span>value: {value} </span>
      <button onClick={handleClick}>点击</button>
    </div>
  );
}
```
useState 返回的修改函数是异步的，调用后并不会直接生效，因此立马读取 value 获取到的是旧值（0）
React 这样设计的目的是为了性能考虑，争取把所有状态改变后只重绘一次就能解决更新问题，而不是改一次重绘一次，也是很容易理解的。

### 在 timeout 中读不到其他状态的新值

```jsx
const MyComponent = () => {
  const [value, setValue] = useState(0);
  const [anotherValue, setAnotherValue] = useState(0);

  useEffect(() => {
    window.setTimeout(() => {
      console.log('setAnotherValue', value) // <- 0
      setAnotherValue(value);
    }, 1000);
    setValue(1);
  }, []);

  return (
    <span>Value：{value}, AnotherValue：{anotherValue}</span>
  );
}
```
因为在生成 timeout 闭包时，value 的值是 0。
虽然之后通过 setValue 修改了状态，但 React 内部已经指向了新的变量，而旧的变量仍被闭包引用，所以闭包拿到的依然是旧的初始值，也就是 0。

我们可以通过使用useRef
```jsx
const MyComponent = () => {
  const [value, setValue] = useState(0);
  const [anotherValue, setAnotherValue] = useState(0);
  const valueRef = useRef(value);
  valueRef.current = value;

  useEffect(() => {
    window.setTimeout(() => {
      console.log('setAnotherValue', valueRef.current) // <- 0
      setAnotherValue(valueRef.current);
    }, 1000);
    setValue(1);
  }, []);

  return (
    <span>Value：{value}, AnotherValue：{anotherValue}</span>
  );
}
```


# 参考
https://zhuanlan.zhihu.com/p/87713171