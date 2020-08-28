首先看一下官方的demo

```js
import { createStore } from 'redux'

// reducer
function todos(state = [], action) {
  switch (action.type) {
    // case 对应action
    case 'ADD_TODO':
      return state.concat([action.text])
    default:
      return state
  }
}

let store = createStore(todos, ['Use Redux'])

store.dispatch({
  type: 'ADD_TODO',
  text: 'Read the docs'
})

console.log(store.getState())
```