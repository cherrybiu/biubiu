## vue原理之虚拟DOM的产生

> **虚拟DOM是什么**

- 虚拟`Dom`，即不是真实的`Dom`, 而是使用`js`对象对真实的`Dom`的一个描述，其包含了`tag`、`props`、`children`三个属性。

  真实Dom：

  ```html
  <div id="test">
    <p class="text">hello world</p>
  </div>
  ```

  转为虚拟Dom如下：

  ```javascript
  {
  	tag: 'div',
  	props: {
      id: 'test'
    },
    children: [
      {
        tag: 'p',
        props: {
          className: 'text'
        },
        children: ['hello world']
      }
    ]  
  }
  ```

> **为什么需要虚拟DOM**

- 虚拟`Dom`到底有什么优势， 其中我们熟知的一点是虚拟`Dom`可以通过`diff`算法减少`javascript`操作真实的`Dom`所带来的性能损耗。
- 虚拟dom抽象了原本的渲染过程，实现了跨平台的功能，可以渲染到dom之外的平台，使用`SSR`、`同构渲染`这些高级特性，`Weex`等框架应用的就是这一特性。

> **h函数**

​		对于如上的真实`dom`的示例，我们在实例化Vue的过程中可以在`render`函数内是这样的:

```javascript
new Vue({
	render(h) {
		return h('div', {
			attrs: {
				id: 'test'
			}
		}, [
      h('p', {
        attrs: {
          class: 'test'
        }
      }, 'hello')
    ])
	}
})
```

这个时候还不是对象类型，而是用`render`函数的数据结构来暂时描述，那我们需要将数据结构转化为对象的形式。`render`函数使用的是参数`h`方法并用`VNode`来实例化它们的，`VNode`的类定义如下所示：

```javascript
export default class VNode {
  constructor (
    tag
    data
    children
    text
    elm
    context
    componentOptions
    asyncFactory
  ) {
    this.tag = tag  // 标签名
    this.data = data  // 属性 如id/class
    this.children = children  // 子节点
    this.text = text  // 文本内容
    this.elm = elm  // 该VNode对应的真实节点
    this.ns = undefined  // 节点的namespace
    this.context = context  // 该VNode对应实例
    this.fnContext = undefined  // 函数组件的上下文
    this.fnOptions = undefined  // 函数组件的配置
    this.fnScopeId = undefined  // 函数组件的ScopeId
    this.key = data && data.key  // 节点绑定的key 如v-for
    this.componentOptions = componentOptions  //  组件VNode的options
    this.componentInstance = undefined  // 组件的实例
    this.parent = undefined  // vnode组件的占位符节点
    this.raw = false  // 是否为平台标签或文本
    this.isStatic = false  // 静态节点
    this.isRootInsert = true  // 是否作为根节点插入
    this.isComment = false  // 是否是注释节点
    this.isCloned = false  // 是否是克隆节点
    this.isOnce = false  // 是否是v-noce节点
    this.asyncFactory = asyncFactory  // 异步工厂方法
    this.asyncMeta = undefined  //  异步meta
    this.isAsyncPlaceholder = false  // 是否为异步占位符
  }

  get child () {  // 别名
    return this.componentInstance
  }
}
```

