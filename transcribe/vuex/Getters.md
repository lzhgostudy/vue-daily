# Getter

有时候我们需要从store中派生出一些状态，例如对列表进行过滤并计数：

```js
computed: {
  doneTodosCount () {
    return this.$store.state.todos.filter(todo => todo.done).length
  }
}
```

如果多个组件需要此属性，我们要么赋值这个函数，或者抽取到一个共享函数然后在多处导入它————无论哪种方法都不是很理想。

Vuex允许我们在store中定义`getter`，就像计算属性一样，getter的返回值会根据它的依赖被缓存起来，且只有当它的依赖值发生了改变才会被重新计算。

Getter接受state作为其第一个参数：

```js
const store = new Vuex.Store({
  state: {
    todos: [
      { id: 1, text: '...', done: true },
      { id: 2, text: '...', done: false }
    ]
  },
  getters: {
    doneTodos: state => {
      return state.todos.filter(todo => todo.done)
    }
  }
})
```

### 通过属性访问

Getter会暴露为`store.getters`对象，你可以以属性的形式访问这些值：

```js
store.getters.doneTodos
```

Getter也可以接受其他getter作为第二个参数：

```js
getters: {
  // ...
  doneTodosCount: (state, getters) => {
    return getters.doneTodos.length
  }
}

store.getters.doneTodosCount // -> 1
```

我们可以很容易地在任何组件中使用它：

```js
computed: {
  doneTodosCount () {
    return this.$store.getters.doneTodosCount
  }
}
```

**注意，getter在通过属性访问时是作为Vue的响应式系统的一部分缓存其中的。**


### 通过方法访问

你也可以通过让getter返回一个函数，来实现给getter传参。在你对store里的数组进行查询时非常有用。
```js
getters: {
  // ...
  getTodoById: (state) => (id) => {
    return state.todos.find(todo => todo.id === id)
  }
}


store.getters.getTodoById(2) // -> { id: 2, text: '...', done: false }
```

**注意，getter在通过方法访问时，每次都会去进行调用，而不会缓存结果。**



### mapGetters 辅助函数

`mapGetters`辅助函数仅仅是将store中的getters映射到局部计算属性：

```js
import { mapGetters } from 'vuex';

export default {
  // ...

  computed: {
    // 使用对象展开运算符将 getter 混入到 computed 对象中
    ...mapGetters([
      'doneTodoCount',
      'anotherGetter'
    ])
  }
}
```

如果你想将一个getter属性另外取名，使用对象形式：

```js
...mapGetters({
  // 把 this.doneCount 映射为 this.$store.getters.doneTodosCount
  doneCount: 'doneTodosCount'
})
```