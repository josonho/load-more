滚动加载指令，附源码和使用案例，可直接使用。

##### 背景：
>因项目中需实现滚动分页，但是使用v-infinite-scroll渲染时会触发，所以自己实现滚动加载指令来替代

 #### 实现思路：
通过监听滚动事件，判断**滚动到最底部时，触发加载事件**


**如何判断是否滚动到最底部呢：**

**这里我们通过判断：容器内的滚动距离是否大于或等于，容器的内高度 减 容器自身的高度的差，如果满足，则认为已经滚动至底部**
`滚动距离 >= 子级高度 - 自身高度`

##### 代码实现：
v-load-more指令的代码实现：
```javascript
directives: {
  /* 通过判断 滚动距离 >= 子级高度 - 自身高度
    如果为true则认为到达底部 */ 
  loadMore: {
    inserted(el, binding) {
      el.addEventListener('scroll', function () {
        if (el.scrollTop >= el.scrollHeight - el.clientHeight) {
          binding.value()
        }
      })
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
