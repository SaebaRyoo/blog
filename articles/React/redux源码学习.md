> 首先，我们需要知道redux的相关概念

# 概念

### redux

1. Action：一个JavaScript对象，描述动作相关信息，必须要包含的是一个type属性，用于描述当前Action，其他属性为值。

2. Reducer：定义应用状态如何响应不同动作（action），如何更新状态；
3. Store：管理action和reducer及其关系的对象，主要提供以下功能：

    - 维护应用状态并支持访问状态（getState()）；
    - 支持监听action的分发，更新状态（dispatch(action)）；
    - 支持订阅store的变更（subscribe(listener)）；
