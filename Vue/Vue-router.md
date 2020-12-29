1. 获取vue-router包

   npm install vue-router --save-dev

2. 编写路由配置

   ```js
   import Vue from 'vue'
   import VueRouter from "vue-router";
   import Content from "../src/components/Content";
   import Main from "../src/components/Main";
   
   //安装路由
   Vue.use(VueRouter);//显式声明使用vue-router，必须声明
   
   //配置导出路由
   export default new VueRouter({
     routes:[
       {
         //路由路径
         path: '/Content',
         //命名(可以不加)
         name: 'Content',
         //跳转的组件
         component: Content
       },
       {
         //路由路径
         path: '/main',
         //命名(可以不加)
         name: 'main',
         //跳转的组件
         component: Main
       }
     ]
   })
   ```

3. 编写component

   Content.vue

   ```vue
   <template>
     <h1>内容页</h1>
   </template>
   
   <script>
   export default {
     name: "Content"
   }
   </script>
   
   <style scoped>
   
   </style>
   ```

   Main.vue

   ```vue
   <template>
     <h1>Main</h1>
   </template>
   
   <script>
   export default {
     name: "Main"
   }
   </script>
   
   <style scoped>
   
   </style>
   ```

4. 在App.vue中配置路由

   ```vue
   <template>
     <div id="app">
       <h1>HelloWorld</h1>
       <router-view></router-view><!--页面中放置路由跳转的组件的地方-->
       <router-link to="/Content">内容页</router-link>
       <router-link to="/main">首页</router-link>
     </div>
   </template>
   
   <script>
   
   export default {
     name: 'App',
   }
   </script>
   
   <style>
   #app {
     font-family: 'Avenir', Helvetica, Arial, sans-serif;
     -webkit-font-smoothing: antialiased;
     -moz-osx-font-smoothing: grayscale;
     text-align: center;
     color: #2c3e50;
     margin-top: 60px;
   }
   </style>
   ```

5. 在main.js中配置路由

   ```js
   import Vue from 'vue'
   import App from './App'
   import router from '../router'//自动扫描内部路由的配置
   
   Vue.config.productionTip = false
   
   /* eslint-disable no-new */
   new Vue({
     el: '#app',
     //配置路由
     router,
     components: { App },
     template: '<App/>'
   })
   ```

6. 完成

   <img src="https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20200822214116.png" alt="image-20200822213950982" style="zoom:50%;" />

7. 传递参数 （此处以参数为id进行考虑）

   - 方式一：不开启props：true

     1. 对路由进行配置

     ```js
     routes:[
       {
         //路由路径
         path: '/Content/:id',
         //命名(可以不加)
         name: 'Content',
         //跳转的组件
         component: Content,
         //props: true//开启接受参数
       },
     ```

     2. 视图中对route-link进行绑定参数，记住是绑定

        ```html
        <router-link v-bind:to="{name:'Content',params:{id:1}}">内容页</router-link>
        ```

     3. 在对应的组件中进行显示  （注意组件中的template要想显示内容一定要有根结点，否则不能生效，即要有`<div></div>`）

        ```vue
        {{$route.params.id}}
        ```

   - 方拾二：开始props：true

     ​	1.对路由进行配置，要对props进行开启

     ```js
     {
       //路由路径
       path: '/Content/:id',
       //命名(可以不加)
       name: 'Content',
       //跳转的组件
       component: Content,
       props: true//开启接受参数
     },
     ```

      2. 在相应的组件中进行配置

         ```vue
         export default {
           name: "Content",
           props: ['id']//接受参数
         }
         ```

         ```vue
         {{id}}//直接在template中调用参数
         ```

8. 创建404页面

   创建一个404组件，然后再route中进行注册即可，需要写到最后，因为地址是从上往下进行寻找路由

   ```js
   {
     path: '*',
     component: NotFound
   }
   ```

   