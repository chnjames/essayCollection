步骤：

1. 安装mockjs

   `npm install mockjs --save-dev`

2. 启动项目

   `npm run serve`

3. 创建mock.js 文件

   - 在src路径下创建mock.js文件
   - 在main.js引入mock.js 文件

   ![mock.js](https://raw.githubusercontent.com/chnjames/cloudImg/main/20210521164004.png)

4. mock.js 使用

   - 在mock.js文件中写入测试代码（此处箭头函数中的代码可以根据mockjs官网来进行修改随机数据及属性名称）

     ```javascript
     //引入mockjs
     const Mock = require('mockjs')
     // 获取 mock.Random 对象
     const Random = Mock.Random;
     //使用mockjs模拟数据
     Mock.mock('/api/data', (req, res) => {//当post或get请求到/api/data路由时Mock会拦截请求并返回上面的数据
         let list = [];
         for(let i = 0; i < 30; i++) {
             let listObject = {
                 title: Random.csentence(5, 10),//随机生成一段中文文本。
                 company: Random.csentence(5, 10),
                 attention_degree: Random.integer(100, 9999),//返回一个随机的整数。
                 photo: Random.image('114x83', '#00405d', '#FFF', 'Mock.js')
             }
             list.push(listObject);
         }
         return {
             data: list
         }
     })
     ```

5. 在需要的xxx.vue文件中使用axios获取mock.js中的随机数据

   ```javascript
   import axios from 'axios'
   
   export default {
         data() {
           return {
             data:[]
           }
         },
         mounted:function() {
           axios.get('/api/data').then(res => {//get()中的参数要与mock.js文件中的Mock.mock()配置的路由保持一致
             this.data = res.data.data;
             console.log(res.data);//在console中看到数据
           }).catch(res => {
             alert('wrong');
           })
         },
         methods:{
             
         }
   }
   ```

   
