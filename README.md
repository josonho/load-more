滚动加载指令，附源码和使用案例，可直接使用。

##### 背景：
>因项目中需实现滚动分页，但是使用v-infinite-scroll渲染时会触发，所以自己实现滚动加载指令来替代

#### 前提条件：

 - jquery.js：使用了jquery计算元素高度，可自行替代为原生JavaScript
 - lodash.js：使用lodash处理滚动节流，可自行取消节流、或替代为原生JavaScript实现
 
 #### 实现思路：
 
通过监听滚动事件，判断**滚动到最底部时，触发加载事件**


**如何判断是否滚动到最底部呢：**

**这里我们通过判断：容器内的滚动距离是否大于或等于，容器的所有子级高度（包括border，padding，margin）减 容器自身的高度的差，如果满足，则认为已经滚动至底部**
`滚动距离 >= 子级高度 - 自身高度`

我们设容器为container，那么：
container的所有子级高度：可以通过遍历其直接子级，并计算子级offsetHeight的总和
container的滚动距离：可以通过jquery的scrollTop()获取
container的高度：可以通过jquery的height()获取

我们有了大概的思路，下面，将使用vue的指令实现滚动加载
我们使用了自定义指令[directives](https://cn.vuejs.org/v2/guide/custom-directive.html)的`inserted`钩子来实现
>inserted：被绑定元素插入父节点时调用 (仅保证父节点存在，但不一定已被插入文档中)。

inserted会接受3个参数，el、binding、vnode，这里我们只使用el，和binding
**el是我们的容器实例，$(el)可以获取到容器的dom。
 binding.value则是我们绑定的事件，例如v-load-more="loadMore"，binding.value就是loadMore**


##### 代码实现：
v-load-more指令的代码实现：
```javascript
directives: {
  /* 通过判断 滚动距离 >= 子级高度 - 自身高度
     如果为true则认为到达底部 */ 
   loadMore: {
     inserted(el, binding) {
       const scrollFn = () => {
         // 子级高度
         let scrollH = 0
         $(el).children().each((idx, item) => {
           scrollH += $(item).get(0).offsetHeight
         })

         let wrapH = $(el).height() // 自身高度
         let scrollT = $(el).scrollTop() // 滚动距离
         let isBottom = scrollT >= scrollH - wrapH
         if (isBottom) {
           binding.value && binding.value() // binding.value获取到绑定的事件
         }
       }
       
       // 使用lodash的throttle函数进行节流处理
       $(el).on('scroll', _.throttle(scrollFn, 100))
     }
   }
 },
```

附使用案例
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/jquery/3.6.0/jquery.min.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/lodash.js/4.17.21/lodash.min.js"></script>
</head>
<body>
  <div id="app">
    <div class="list-container" v-load-more="loadMore">
      <div>
        <div class="list-item" v-for="(item, idx) in list" :key="idx">
          {{item}}
        </div>
      </div>
    </div>
  </div>
</body>
</html>

<script>
var app = new Vue({
  el: '#app',
  data: {
    list: 20,
  },
  directives: {
    /* 通过判断 滚动距离 >= 子级高度 - 自身高度
      如果为true则认为到达底部 */ 
    loadMore: {
      inserted(el, binding) {
        const scrollFn = () => {
          // 子级高度
          let scrollH = 0
          $(el).children().each((idx, item) => {
            scrollH += $(item).get(0).offsetHeight
          })

          let wrapH = $(el).height() // 自身高度
          let scrollT = $(el).scrollTop() // 滚动距离
          let isBottom = scrollT >= scrollH - wrapH
          if (isBottom) {
            binding.value && binding.value()
          }
        }
        $(el).on('scroll', _.throttle(scrollFn, 100))
      }
    }
  },
  methods: {
    loadMore() {
      this.list += 20
    }
  },
})
</script>

<style>
  .list-container {
    margin: 0 auto;
    height: 500px;
    width: 400px;
    overflow-y: auto;
  }
  .list-item {
    line-height: 60px;
    text-align: center;
    background: #ccc;
    border-bottom: 1px solid #fff;
  }
</style>
```
<br>

[在线演示](https://josonho.github.io/load-more/)
