1. setState 只在合成事件和钩子函数中是“异步”的，在原生事件和 setTimeout 中都是同步的。
2. setState的“异步”并不是说内部由异步代码实现，其实本身执行的过程和代码都是同步的，只是合成事件和钩子函数的调用顺序在更新之前，导致在合成事件和钩子函数中没法立马拿到更新后的值，形式了所谓的“异步”，当然可以通过第二个参数 setState(partialState, callback) 中的callback拿到更新后的结果。
3. setState 的批量更新优化也是建立在“异步”（合成事件、钩子函数）之上的，在原生事件和setTimeout 中不会批量更新，在“异步”中如果对同一个值进行多次 setState ， setState 的批量更新策略会对其进行覆盖，取最后一次的执行，如果是同时 setState 多个不同的值，在更新时会对其进行合并批量更新。


### [为什么setState是异步的？](https://github.com/facebook/react/issues/11527)

为了批量更新而延迟reconciliation, `setState()`同步渲染在多数情况下效率低下，如果我们知道可能会得到多个更新，最好进行批处理更新
例如，如果我们在浏览器单击处理程序中，并且`Child`和`Parent`都调用`setState`，则我们不想将其重新渲染两次，而是希望将它们标记为 `dirty`，然后再将它们一起渲染 退出浏览器事件。

你会问： 为什么我们不能做同样的事情(批处理)，而是立即将`setState`更新写入`this.state`而无需等待`recelication`结束. 我认为没有一个明显的答案（两种解决方案都需要权衡），但是我可以想到一些原因。

1. 保持内部一致性

即使state被同步更新，props也不会被更新。 （除非您重新渲染父组件，否则您将不知道props，而如果您同步进行渲染，则批处理将无法进行。）

比如以下这种场景:

当您只使用`state`时，如果它以同步方式刷新，则以下模式将工作
```js
console.log(this.state.value) // 0
this.setState({ value: this.state.value + 1 });
console.log(this.state.value) // 1
this.setState({ value: this.state.value + 1 });
console.log(this.state.value) // 2
```

但是，假设这个state需要被提升，以便在几个组件之间共享，所以您将它移动到父组件
```js
- this.setState({ value: this.state.value + 1 });
+ this.props.onIncrement(); // Does the same thing in a parent
```

我想强调的是，在依赖于setState()的典型React应用程序中，这是您每天都会进行的最常见的特定于react的重构。

但是，这破坏了我们的代码！

```js
console.log(this.props.value) // 0
this.props.onIncrement();
console.log(this.props.value) // 0
this.props.onIncrement();
console.log(this.props.value) // 0
```

这是因为在您建议的模型中，会立即刷新this.state，但不会刷新this.props。 而且，我们无法在不重新渲染父级的情况下立即刷新this.props，这意味着我们将不得不放弃批处理（根据情况，批处理会大大降低性能）。

那么React今天如何解决这个问题呢？ 在React中，this.state和this.props都仅在reconciliation 和 flushing,之后更新，因此您将看到0在重构前后都被打印。 这使提升状态安全。

综上所述，React模型并不总是导致最简洁的代码，但是它在内部是一致的，并且可以确保提升状态是安全的

2. 启用Concurrent Updates(并发更新)

从概念上讲，React的行为就像每个组件只有一个更新队列一样。 这就是为什么讨论完全有意义的原因：我们讨论是否立即对this.state进行更新，因为我们毫无疑问更新将按照确切的顺序应用。 但是，事实并非如此 。

我们一直在解释“异步渲染”的一种方式是，React可以根据来自何处的setState（）调用分配不同的优先级：事件处理程序，网络响应，动画等。

例如，如果要键入消息，则需要立即刷新TextBox组件中的setState（）调用。 但是，如果您在键入时收到新消息，则最好将新MessageBubble的呈现延迟到某个阈值（例如，一秒钟），而不是由于阻塞线程而使键入变得混乱。

如果我们让某些更新具有较低的优先级，我们可以将它们的渲染分成几毫秒的小块，这样它们就不会被用户注意到。

我知道这样的性能优化可能听起来并不令人兴奋或令人信服。 您可能会说：“ MobX不需要此功能，我们的更新跟踪速度足够快，可以避免重新渲染。” 我认为并非在所有情况下都是如此（例如，无论MobX有多快，您仍然必须创建DOM节点并为新安装的视图进行渲染）。 不过，如果确实如此，并且您有意识地决定始终将对象包装到跟踪读写操作的特定JavaScript库中，那么您可能不会从这些优化中受益匪浅。

**但是异步渲染不仅仅涉及性能优化。 我们认为这是React组件模型可以做什么的根本转变。**

例如，考虑从一个屏幕导航到另一个屏幕的情况。 通常，您会在新屏幕呈现时显示 spinner 。

但是，如果导航足够快（在一秒钟左右），则闪烁并立即隐藏 spinner 会降低用户体验。 更糟糕的是，如果您拥有具有不同异步依赖关系（数据，代码，图像）的多个级别的组件，则最终会看到一堆 spinners ，它们会一一短暂地闪烁。 由于所有DOM的重排，这在视觉上都是不愉快的，并且使您的应用在实践中变慢。 它也是许多样板代码的来源。
