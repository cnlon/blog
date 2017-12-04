<script type="application/ld+json">
{
  "@type": "Article",
  "headline": "用 Vue 观察属性变化",
  "name": "watching-property-with-vue",
  "url": "https://lon.im/post/watching-property-with-vue.html",
  "dateCreated": "2016-11-16",
  "dateModified": "2017-02-06",
  "datePublished": "2016-11-16"
}
</script>


**响应系统是** Vue 一个显著功能，修改属性，可以更新视图，这让状态管理变得非常简单且直观。
创建 Vue 实例时，Vue 将遍历 `data` 的属性，通过 ES5 的 `Object.defineProperty` 将它们转为 getter/setter，在其内部 Vue 可以追踪依赖、通知变化。

```javascript
const vm = new Vue({
  data: {foo: 1} // 'vm.foo' (在内部，同 'this.foo') 是响应的
})
```

<h2 id="watching">观察属性变化</h2>

Vue 的实例提供了 `$watch` 方法，用于观察属性变化。

```javascript
const vm = new Vue({
  data: {foo: 1}
})

vm.$watch('foo', function (newValue, oldValue) {
  console.log(newValue, oldValue) // 输出 2 1
  console.log(this.foo) // 输出 2
})

vm.foo = 2
```

当属性变化后，响应函数将会被调用，在其内部，`this` 自动绑定到 Vue 的实例 `vm` 上。
需要注意的是，响应是异步的。如下：

```javascript
const vm = new Vue({
  data: {foo: 1}
})

vm.$watch('foo', function (newValue, oldValue) {
  console.log('inner:', newValue) // 后输出 "inner" 2
})

vm.foo = 2
console.log('outer:', vm.foo) // 先输出 "outer" 2
```

通过 `$watch` Vue 实现了数据和视图的绑定。观察到数据变化，Vue 便**异步**更新 DOM ，在同一事件循环内，多次数据变化将会被缓存起来，在下次事件循环中，Vue 刷新队列并仅执行必要的更新。如下：

```javascript
const vm = new Vue({
  data: {foo: 1}
})

vm.$watch('foo', function (newValue, oldValue) {
  console.log('inner:', newValue) // 后只输出一次 "inner" 5
})

vm.foo = 2
vm.foo = 3
vm.foo = 4
console.log('outer:', vm.foo) // 先输出 "outer" 4
vm.foo = 5
```

<h2 id="computed-property">计算属性</h2>

MV* 中，将 Model 层数据展现到 View，经常有复杂的数据处理逻辑，这种情况下，使用计算属性 (computed property) 更加明智。

```javascript
const vm = new Vue({
  data: {
    width: 0,
    height: 0,
  },
  computed: {
    area () {
      let output = ''
      if (this.width > 0 && this.height > 0) {
        const area = this.width * this.height
        output = area.toFixed(2) + 'm²'
      }
      return output
    }
  }
})

vm.width = 2.34
vm.height = 5.67
console.log(vm.area) // 输出 "13.27m²"
```

在计算属性内部，`this` 自动绑定 `vm`，因此**声明计算属性时需要避免使用箭头函数**。
上例中，`vm.width` 和 `vm.height` 是响应的，`vm.area` 内部首次读取 `this.width` 和 `this.height` 时，Vue 收集其做为 `vm.area` 的依赖，此后 `vm.width` 或 `vm.height` 变化时，`vm.area` 重新求值。
**计算属性是基于它的依赖缓存**，如果 `vm.width` 和 `vm.height` 没有变化，多次读取 `vm.area`，会立即返回之前的计算结果，而不必再次求值。
同样由于 `vm.width` 和 `vm.height` 是响应的，在 `vm.area` 中可以**将依赖的属性赋值给一个变量，通过读取变量来减少读取属性次数，同时解决在条件分支中，Vue 有时会无法收集到依赖的问题**。
新的实现如下：

```javascript
const vm = new Vue({
  data: {
    width: 0,
    height: 0,
  },
  computed: {
    area () {
      let output = ''
      const {width, height} = this
      if (width > 0 && height > 0) {
        const area = width * height
        output = area.toFixed(2) + 'm²'
      }
      return output
    }
  }
})

vm.width = 2.34
vm.height = 5.67
console.log(vm.area) // 输出 "13.27m²"
```

<h2 id="ob-js">通过 smart-observe 单独使用 Vue 的属性观察模块</h2>

为方便学习和使用，smart-observe 将 Vue 中属性观察模块提取并封装了一下。

smart-observe GitHub 地址：<https://github.com/cnlon/smart-observe>

安装

```bash
npm install --save smart-observe
```

观察属性变化
```javascript
const target = {a: 1}
ob(target, 'a', function (newValue, oldValue) {
  console.log(newValue, oldValue) // 3 1
})
target.a = 3
```

添加计算属性
```javascript
const target = {a: 1}
ob.compute(target, 'b', function () {
  return this.a * 2
})
target.a = 10
console.log(target.b) // 20
```

像声明 Vue 实例一样传入参数集合

```javascript
const options = {
  data: {
    PI: Math.PI,
    radius: 1,
  },
  computed: {
    'area': function () {
      return this.PI * this.square(this.radius)
    },
  },
  watchers: {
    'area': function (newValue, oldValue) {
      console.log(newValue) // 28.274333882308138
    },
  },
  methods: {
    square (num) {
      return num * num
    },
  },
}
const target = ob.react(options)
target.radius = 3
```

更详细的使用介绍请 [点击这里](https://github.com/cnlon/smart-observe/blob/master/README.zh.md)

同时推荐其它两款同类库

 - Watch.JS <https://github.com/melanke/Watch.JS>
 - observe.js <https://github.com/kmdjs/observejs>
