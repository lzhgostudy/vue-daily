# Action

Action类似于mutation, 不同于：
- Action提交的是mutation, 而不是直接变更状态。
- Action可以包含任意异步操作

让我们来注册一个简单的action:

```js
const store = new Vuex.Store({
  state: {
    count: 0
  },
  mutations: {
    increment(state) {
      state.count++;
    }
  },
  actions: {
    increment(context) {
      context.commit('increment');
    }
  }
})
```

Action函数接受一个与store实例具有相同方法和属性的context对象，因此你可以调用`context.commit`提交一个mutation, 或者通过`context.state`和`context.getters`来获取state和getters。当我们在之后介绍到`Modules`时，你就知道context对象为什么不是store本身了。

实践中，我们会经常会用到ES2015的参数解构来简化代码：

```js
actions: {
  increment({commit}) {
    commit('increment')
  }
}
```

### 分发Action

Action通过`store.dispatch`方法触发：

```js
store.dispatch('increment')
```

乍一看上去感觉多此一举，我们直接分发mutation岂不是更方便？实际上并非如此，还记得mutation必须同步执行这个限制么？Action就不受约束！我们可以在action内部执行异步操作：

```js
actions: {
  incrementAsync({ commit }) {
    setTimeout(() => {
      commit('increment', 1000);
    })
  }
}
```

Actions支持同样的载荷方式和对象方式进行分发：

```js
// 以载荷形式分发
store.dispatch('incrementAsync', {
  amount: 10
});

// 以对象形式分发
store.dispatch({
  type: 'incrementAsync',
  amount: 10
})
```

来看一个更加实际的购物车示例，涉及到调用异步API和分发多重mutation:

```js
actions: {
  checkout({commit, state}, products) {
    // 把当前购物车的物品备份起来
    const saveCartItems = [...state.cart.added]
    // 发出结账请求，然后乐观地清空购物车
    commit(types.CHECKOUT_REQUEST)
    // 购物API接受一个成功回调和一个失败回调
    shop.buyProducts(
      products,
      // 成功操作
      () => commit(types.CHECKOUT_SUCCESS),
      // 失败操作
      () => commit(types.CHECKOUT_FAILURE, savedCartItems)
    )
  }
}
```

注意我们正在进行一系列异步操作，并且通过提交mutation来记录action来记录action产生的副作用（即状态变更）。

### 在组件中分发Action

你在组件中使用`this.$store.dispatch('xxx')`分发action，或者使用`mapActions`辅助函数将组件的methods映射为`store.dispatch`调用

```js
import { mapActions } from 'vuex'

export default {
  // ...
  methods: {
    ...mapActions([
      'increment', // 将 `this.increment()` 映射为 `this.$store.dispatch('increment')`

      // `mapActions` 也支持载荷：
      'incrementBy' // 将 `this.incrementBy(amount)` 映射为 `this.$store.dispatch('incrementBy', amount)`
    ]),
    ...mapActions({
      add: 'increment' // 将 `this.add()` 映射为 `this.$store.dispatch('increment')`
    })
  }
}
```

### 组合Actions

Action通常是异步的，那么如何知道action什么时候结束呢？更重要的是，我们如何才能组合多个action，以处理更加复杂的异步流程？

首先，你需要明白`store.dispatch`可以处理被触发action的处理函数返回的Promise，并且`store.dispatch`仍旧返回Promise:

```js
actions: {
  actionA({commit}) {
    return new Promise((resolve, reject) => {
      setTimeout(() => {
        commit('someMutation')
        resolve()
      }, 1000)
    })
  }
}


// 调用
store.dispatch('actionA').then(() => {
  // ...
}) 

// 在另一个action中也可以：
actions: {
  actionB({ dispatch, commit }) {
    return dispatch('actionA').then(() => {
      commit('someOtherMutation')
    })
  }
}

// 最后，如果我们利用 async/await ，我们可以如下组合action
actions: {
  async actionA({commit}) {
    commit('gotData', await getData())
  },
  async actionB({ dispatch, commit }) {
    await dispatch('actionA')
    commit('gotOtherData', await getOtherData())
  }
}
```

一个`store.dispatch`在不同模块中可以触发多个action函数。在这种情况下，只有当所有触发函数完成后，返回的Promise才会执行。