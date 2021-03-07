# Mutation

更改Vuex的store中的状态的唯一方法是提交`mutation`。Vuex中的mutation非常类似于事件：每个mutation都有一个字符串的事件类型和一个回调函数。这个回调函数就是我们实际进行状态更改的地方，并且它会接受state作为第一个参数：

```js
const store = new Vuex.Store({
  state: {
    count: 1
  },
  mutations: {
    increment(state) {
      return state.count++;
    }
  }
})
```

你不能直接调用一个mutation handler。这个选项更像是事件注册：当触发一个类型为 `increment`的mutation时，调用此函数。要唤醒一个mutation handler，你需要以相应的type调用`store.commit`方法:

```js
store.commit('increment');
```

### 提交载荷 payload

你可以向`store.commit`传入额外的参数，即mutation的载荷payload:

```js
mutations: {
  increment (state, n) {
    state.count += n
  }
}


store.commit('increment', 10)
```

在大多数情况下，载荷应该是一个对象，这样可以包含多个字段并且记录的mutation会更易读：

```js
mutations: {
  increment (state, payload) {
    state.count += payload.amount
  }
}


store.commit('increment', {
  amount: 10
})
```

### 对象风格的提交形式

提交mutation的另一个方式是直接使用包含`type`属性的对象：

```js
store.commit({
  type: 'increment',
  amount: 10
})
```

当使用对象风格的提交方式，整个对象都作为载荷传给mutation函数，因此handler保持不变
```js
mutations: {
  increment (state, payload) {
    state.count += payload.amount
  }
}
```

### Mutation需遵守Vue的响应规则

既然Vuex的store中的状态是响应的，那么当我们变更状态时，监视状态的Vue组件也会自动更新。这也意味着Vuex的mutation也需要与使用Vue一样遵守一些注意事项：
- 最好提前在你的store中初始化好所有所需要的属性
- 当需要在对象上添加新属性时，你应该使用`Vue.set(obj, 'newProp', 123)`, 或者以新对象替换老对象。例如，利用扩展运算符：
```js
state.obj = { ...state.obj, newProp: 123 }
```

### 使用常量替代Mutation事件类型

使用常量代替mutation事件类型在各种Flux实现中是很常见的模式。这样可以是linter之类的工具发挥作用，同时把这些常量放在单独的文件中可以让你的代码合作者对整个app包含的mutation一目了然：

```js
// mutation-types.js
export const SOME_MUTATION = 'SOME_MUTATION'


// store.js
import Vuex from 'vuex'
import { SOME_MUTATION } from './mutation-types'

const store = new Vuex.Store({
  state: { ... },
  mutations: {
    // 我们可以使用 ES2015 风格的计算属性命名功能来使用一个常量作为函数名
    [SOME_MUTATION] (state) {
      // mutate state
    }
  }
})
```

用不用常量取决于你——在需要多人协作的大型项目中，这会很有帮助。但如果你不喜欢，你完全可以不这样做。



### Mutation 必须是同步函数

一条重要的原则就是要记住**mutation**必须是同步函数。为什么？请参考下面的例子：

```js
mutations: {
  someMutation (state) {
    api.callAsyncMethod(() => {
      state.count++
    })
  }
}
```

现在想象，我们正在debug一个app并且观察devtool的mutation日志，每一条mutation被记录，devtools都需要捕捉到前一状态和后一状态的快照。然而，在上面的例子中 mutation 中的异步函数中的回调让这不可能完成：因为当 mutation 触发的时候，回调函数还没有被调用，devtools 不知道什么时候回调函数实际上被调用——实质上任何在回调函数中进行的状态的改变都是不可追踪的。

### 在组件中提交Mutation

你可以在组件中使用`this.$store.commit('xx')`提交mutation，或者用`mapMutation`辅助函数将组件中的methods映射为`store.commit`调用

```js
import { mapMutations } from 'vuex'

export default {
  // ...
  methods: {
    ...mapMutations([
      'increment', // 将 `this.increment()` 映射为 `this.$store.commit('increment')`

      // `mapMutations` 也支持载荷：
      'incrementBy' // 将 `this.incrementBy(amount)` 映射为 `this.$store.commit('incrementBy', amount)`
    ]),
    ...mapMutations({
      add: 'increment' // 将 `this.add()` 映射为 `this.$store.commit('increment')`
    })
  }
}
```

### 下一步：Action

在mutations中混合异步调用会导致你的程序很难调试。例如，当你调用了两个包含异步回调的mutation来改变状态，你怎么知道什么时候回调和哪个先回调呢？这就是为什么我们要区分这两个概念。在 Vuex 中，**mutation 都是同步事务**：

```js
store.commit('increment')
// 任何由 "increment" 导致的状态变更都应该在此刻完成。
```