1. 效果图

   ![表格嵌套](https://raw.githubusercontent.com/chnjames/cloudImg/main/20210413114226.png)

2. 新建一个expandRow组件，在需要的页面引入

   `import expandRow from "@/components/expandRow";`

3. 在根据render函数的方式使用：

   ```javascript
   columnsTh: [
             {
               type: "expand",
               width: 50,
               render: (h, params) => {
                 let list = params.row.trainData;
                 console.log(params)
                 return h(expandRow, {
                   props: {
                     row: list
                   }
                 });
               }
             }, 
             ......
           ],
   ```

4. 在新建的expandRow组件中编辑好嵌套表格的内容

   ```javascript
   <template>
     <div class="page-content">
       <Table border ref="table" size="small" :columns="columnsRow" :data="row">
         <template slot-scope="{row}" slot="memberInfo">
           <div class="table-info">
             <div>{{row.memberInfo}}</div>
           </div>
         </template>
         <template slot-scope="{row}" slot="groupGoods">
           <div class="table-info">
             <div>{{row.groupGoods}}</div>
           </div>
         </template>
       </Table>
     </div>
   </template>
   <script>
     const columnsRow = [
       {
         title: 'title',
         key: 'groupIdentity',
         align: 'center'
       },
       ......
     ]
     export default {
       name: "expandRow",
       data() {
         return {
           columnsRow: columnsRow, // 表格表头
         }
       },
       props: {
         row: Array
       }
     }
   </script>
   ```

   



