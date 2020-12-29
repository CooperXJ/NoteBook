#### Vue的核心优势

​	可以动态的监听数据的变化，不需要刷新页面即可对数据进行局部刷新

​	数据双向绑定

​	组件（Component）

#### 引入Vue的cdn才能够使用Vue的语法

```html
<link rel="stylesheet" href="https://cdn.bootcss.com/element-ui/2.6.1/theme-chalk/index.css">

<script src="https://cdn.bootcss.com/vue/2.5.2/vue.min.js"></script>
<script src="https://cdn.bootcss.com/vue-router/3.0.1/vue-router.min.js"></script>
<script src="https://cdn.bootcss.com/element-ui/2.4.0/index.js"></script>
```



#### Vue基本语法

- Hello,Vue

  ```html
  <script>
      var vm = new Vue({
          el: "#app",
          data: {
              message: "HelloWorld"  //主体
          }
      });
  </script>
  ```

  两种方式对数据进行绑定：

   1. {{message}}

   2. v-bind

      例如：

      ```html
      <span v-bind:title="message">  <!---span是html的属性-->
         鼠标悬浮在此处查看绑定信息
      </span>
      ```

      `<span title="我这个属性只要鼠标停留在上面就会显示出来">文字文字文字</span>`

- If else 

  ```html
  <div id="app">
      <h1 v-if="message">true</h1>
      <h1 v-else>false</h1>
      
      <h1 v-if="type==='A'">A</h1>
      <h1 v-else-if="type==='B'">B</h1>
      <h1 v-else>C</h1>
  </div>
  
  <script>
      var vm = new Vue({
          el: "#app",
          data: {
              type: 'A',
              message: true
          }
      });
  </script>
  ```

- for 循环

  ```html
  <div id="app">
      <li v-for="(item,index) in items">
          {{item.message}}--{{index}}
      </li>
  </div>
  <script>
          var vm = new Vue({
              el: "#app",
              data: {
                  items: [
                      {message: 'A'},
                      {message: 'B'},
                      {message: 'C'}
                  ]
              }
          });
      </script>
  ```

- 绑定事件

  ```html
  <div id="app">
      <button v-on:click="sayhi">click me</button>
  </div>
  
  <script>
      var vm = new Vue({
          el: "#app",
          data: {
              message: "HelloWorld"
          },
          methods: {
              sayhi: function (){
                  alert(this.message)//this指当前对象
              }
          }
      });
  </script>
  ```

- 双向绑定

  <img src="https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20200821210052.png" alt="image-20200821210037181" style="zoom:50%;" />

  ```html
   <div id="app">
  <!--       <input type="text" v-model="message">-->
  <!--        <span>{{message}}</span>-->
          <select v-model="message">
  <!--            如果v-model表达式的初始值未能匹配任何选项，select元素将被渲染为未选择状态，在ios中这会使得用户无法选择第一个选项，因此需要添加一个空的禁用选项-->
              <option value="" disabled>--请选择--</option>
              <option>A</option>
              <option>B</option>
              <option>C</option>
          </select>
  
          <span>
              {{message}}
          </span>
      </div>
  
  
      <script>
          var vm = new Vue({
              el: "#app",
              data: {
                  message: ""
              },
              methods: {
                  sayhi: function (){
                      alert(this.message)//this指当前对象
                  }
              }
          });
      </script>
  ```

- Vue组件 类似于模板

  ```html
  <div id="app">
      <cooper v-for="item in items" v-bind:qin="item"></cooper>
  </div>
  
  <script>
      // 定义组件
     Vue.component("Cooper",{
         props: ['qin'], //此处必须要添加props，因为这是一个桥梁传递参数
         template: '<li>{{qin}}</li>'
     })
  
     var vm = new Vue({
         el: "#app",
         data: {
             items:["Java","C","C#","Python"]
         }
     })
  </script>
  ```

- axios通信

  ```html
  <div id="app">
      <div>{{info.name}}</div>
      <div>{{info.age}}</div>
      <div>{{info.address.city}}</div>
      <a v-bind:href="info.url">click me</a>
  </div>
  
  <script>
      var vm = new Vue({
          el: "#app",
          data(){
              return{
                  info:{
                      name:null,
                      age:null,
                      url:null,
                      address: {
                          street:null,
                          city:null
                      }
                  }
              }
          },
          mounted(){
              axios.get("data.json").then(response=>(this.info=response.data));
          }
      });
  </script>
  ```

  ```json
  {
    "name": "cooper",
    "age": "18",
    "url": "www.baidu.com",
    "address": {
      "street": "清华街道",
      "city": "江城"
    }
  }
  ```
  
- 计算属性

  就是将一些不经常变的结果放入到缓存中，节省系统的开销

  <img src="https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20200822142251.png" alt="image-20200822142243421" style="zoom:50%;" />

  

   可以看到vm.function1()不断变化，但是vm.function2只要页面不刷新就不会变化，也就是存放到了内存中。

  ```html
  <div id="app">
     <p>{{function1()}}</p>
      <p>{{function2}}</p>
  </div>
  
  <script>
      var vm = new Vue({
          el: "#app",
          data: {
              message: "HelloWorld"
          },
          methods: {
              function1: function (){
                  return Date.now();
              }
          },
          computed: {
              function2: function (){
                  return Date.now();
              }
          }
      });
  </script>
  ```

- slot

  将模板插入到指定的地方

  ```html
  <div id="app">
      <todo>
          <todo-title slot="todo-title" v-bind:title="title"></todo-title>
          <todo-items slot="todo-items" v-for="item in items" v-bind:item="item"></todo-items>
      </todo>
  </div>
  
  <script>
      Vue.component("todo",{
          template: '<div>\
                      <slot name="todo-title"></slot>\
                      <ul>\
                      <slot name="todo-items"></slot>\
                      </ul>\
              </div>'
      });
  
      Vue.component("todo-title",{
          props: ['title'],
          template: '<div>{{title}}</div>'
      });
  
      Vue.component("todo-items",{
          props: ['item'],
          template: '<li>{{item}}</li>'
      });
  
      var vm = new Vue({
          el: "#app",
          data: {
              title: "编程语言",
              items: ['C','C++','C#','Java','Python']
          }
      });
  </script>
  ```

- 自定义事件

  通过双向绑定实现组件中的方法来调用vue实例中的方法

  <font color=red>组件中的方法不能调用直接调用vue实例中的方法</font>

  组件内绑定事件需要使用this.$emit('事件名'，参数)

  ```html
  <div id="app">
      <todo>
          <todo-title slot="todo-title" v-bind:title="title"></todo-title>
          <todo-items slot="todo-items" v-for="(item,index) in items" v-bind:item="item" v-bind:index="index" v-on:remove="removeSingle(index)"></todo-items>
      </todo>
  
  </div>
  
  
  <script>
      Vue.component("todo",{
          template: '<div>\
                      <slot name="todo-title"></slot>\
                      <ul>\
                      <slot name="todo-items"></slot>\
                      </ul>\
              </div>'
      });
  
      Vue.component("todo-title",{
          props: ['title'],
          template: '<div>{{title}}</div>'
      });
  
      Vue.component("todo-items",{
          props: ['item','index'],
          template: '<li>{{item}} <button @click="removeOne">删除</button></li>',
          //只能绑定当前组件的方法，不能绑定vue实例中的方法
          methods:{
              removeOne: function (index){
                  // alert("123");
                  this.$emit('remove',index);
              }
          }
      });
  
      var vm = new Vue({
          el: "#app",
          data: {
              title: "编程语言",
              items: ['C','C++','C#','Java','Python']
          },
          methods: {
              removeSingle: function (index){
                  console.log("删除了"+this.items[index])
                  this.items.splice(index,1);
              }
          }
      });
  </script>
  ```

  

