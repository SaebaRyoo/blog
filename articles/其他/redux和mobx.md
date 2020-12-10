# 概念

### redux

1. Action：一个JavaScript对象，描述动作相关信息，必须要包含的是一个type属性，用于描述当前Action，其他属性为值。

2. Reducer：定义应用状态如何响应不同动作（action），如何更新状态；
3. Store：管理action和reducer及其关系的对象，主要提供以下功能：

    - 维护应用状态并支持访问状态（getState()）；
    - 支持监听action的分发，更新状态（dispatch(action)）；
    - 支持订阅store的变更（subscribe(listener)）；


4. 异步流：由于Redux所有对store状态的变更，都应该通过action触发，异步任务（通常都是业务或获取数据任务）也不例外，而为了不将业务或数据相关的任务混入React组件中，就需要使用其他框架配合管理异步任务流程，如redux-thunk，redux-saga等；