<template>
  <div>
    <h1>文本</h1>
    <span>Message: {{ msg }}</span>
    <h2>v-once指令</h2>
    <span v-once> 这个将不会改变：{{ msg2 }} </span>
    <button v-on:click="changeMsg2">尝试改变msg2</button>
    <h1>原始HTML</h1>
    <p>Using mustaches: {{ rawHtml }}</p>
    <p>Using v-html directive: <span v-html="rawHtml"></span></p>
    <h1>修饰符</h1>
    <div>表单submit绑定.prevent修饰符阻止页面刷新</div>
    <form v-on:submit.prevent="onSubmit">
      <input type="text" name="test" placeholder="会车触发onSubmit" />
    </form>
    <h1>计算属性</h1>
    <div>
      <p>Original message: {{ message }}</p>
      <p>Computed reversed message: {{ reversedMesssage }}</p>
    </div>
    <h1>Class与Style绑定</h1>
    <h2># 对象语法</h2>
    <div :class="{ active: isActive }">isActive</div>
  </div>
</template>
<script>
export default {
  name: "Base",
  data() {
    return {
      msg: "信息",
      msg2: "信息2",
      rawHtml: '<span style="color:red">This should be red.</span>',
      message: "Hello",
      isActive: false
    };
  },
  computed: {
    reversedMesssage: function() {
      return this.message
        .split("")
        .reverse()
        .join("");
    }
  },
  methods: {
    changeMsg2() {
      this.msg2 = "changeMsg2";
    },
    onSubmit() {
      alert("提交表单");
    }
  }
};
</script>
<style lang="scss" scoped>
.active {
  text-decoration: underline;
}
</style>