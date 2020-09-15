## 虚拟DOM
我们在使用开发时会写到如下的代码,这就是虚拟DOM。
```jsx
  <div className="wrap">
    hello world
  </div>
```
它叫jsx。那么为什么使用jsx呢，因为React认为渲染逻辑本质上与其他UI逻辑内在耦合，比如，在UI中绑定事件或者某些时刻状态发生变化时需要通知到UI，以及需要在UI中展示准备好的数据。而jsx和UI放在一起时，会在视觉上有辅助作用。

上面的jsx会通过babel转译为`React.createElement()`方法
```js
const element = React.createElement(
  'div',
  {className: 'wrap'},
  'hello world'
)
```
而createElement方法实际上是创建了一个对象，而这个对象，也就是我们说的虚拟DOM(简化版本)
```js
const element = {
  type: 'div',
  props: {
    className: 'wrap',
    children: 'hello world'
  }
}
```

## react的Diff算法
在react中，会将组建生成一个如上的虚拟DOM树。但是，即使是最前沿的算法中，将一颗树转化为另一颗树的复杂度也是O(n3), n 表示树中元素的数量。

这个算法太过昂贵，如果需要展示1000个元素，那么需要处理的计算量则是1000³，也就是10亿级别的。于是react通过两个假设的基础上，将复杂度降低到了O(n);

1. 两个不同类型的元素会产生不同的树；
2. 通过`key` props来告知react哪些子元素在不同的渲染下保持稳定；

#### 对比不同类型的元素
比如，当一个div元素，变成了一个span元素，那么react会卸载div元素以及它的所有子元素，即使子元素并没有发生更改
```jsx
<div>
  <Son />
</div>

<span>
  <Son />
</span>
```

#### 对比同类型的元素

react保留DOM节点，只对比并更新更改了的属性

```jsx
<div className="a" title="test"/>
<div className="b" title="test"/>
```
对比以上的两个元素，React只会修改DOM元素的class属性

```jsx
<div style={{color: 'red', width: 100}}/>
<div style={{color: 'green', width: 100}}/>
```
对比以上的style样式，只会更新DOM的color,而不会更新width

#### key
假设有如下list
```jsx
<ul>
  <li>a</li>
  <li>b</li>
</ul>


<ul>
  <li>a</li>
  <li>b</li>
  <li>c</li>
</ul>
```
现在需要向list中插入`<li>c</li>`。那么，在list的末尾插入开销是最小的，如果是插入到开头。React则不知道需要保留下面为更改的，那么就需要重新构建每一个元素。

为了解决这种问题，React新增了`key` props，当子元素拥有key时.React使用key来匹配原有树上的子元素以及最新树上的子元素，如下：加了key之后，React则会知道只有c是需要新增加的子元素

```jsx
<ul>
  <li key='a'>a</li>
  <li key='b'>b</li>
</ul>


<ul>
  <li key='a'>a</li>
  <li key='b'>b</li>
  <li key='c'>c</li>
</ul>
```

当子元素的位置不发生改变时，也可以使用数组的索引来作为key。如果有顺序修改，diff 就会变得慢。

当基于下标的组件进行重新排序时，组件 state 可能会遇到一些问题。由于组件实例是基于它们的 key 来决定是否更新以及复用，如果 key 是一个下标，那么修改顺序时会修改当前的 key，导致非受控组件的 state（比如输入框）可能相互篡改导致无法预期的变动。

在 Codepen 有两个例子，分别为 [展示使用下标作为 key 时导致的问题](https://codepen.io/pen?editors=0010)，以及[不使用下标作为 key 的例子的版本，修复了重新排列，排序，以及在列表头插入的问题](https://codepen.io/pen?&editable=true&editors=0010) 。


### vue和react对比
react中，当某个组件的状态发生变化时，它会以该组件为根，重新渲染整个组件子树。如要避免不必要的子组件的重渲染，你需要在所有可能的地方使用 PureComponent，或是手动实现 shouldComponentUpdate 对比条件。同时你可能会需要使用不可变的数据结构来使得你的组件更容易被优化，这也就是react中的**运行时优化**，这样的话，优化的负担会落到开发身上。

而vue对比react有一个很大的优势，就是**编译时优化**。因为在vue应用中，组件的依赖是在渲染过程中自动追踪的，所以系统能精确知晓哪个组件确实需要被重渲染。并不会存在react中的子树问题。这个特点使得开发者不再需要考虑此类优化，从而能够更好地专注于应用本身。
