1. 效果图

   ![iVIew单选](https://raw.githubusercontent.com/chnjames/cloudImg/main/20210413150815.png)

   ```vue
   <template>
     <div class="page-content">
       <Table border :columns="columnsPr" :data="productArr" height="400"></Table>
     </div>
   </template>
   <script>
     export default {
       name: "productTree",
       data() {
         return {
           // 添加商品表头
           columnsPr: [
             {
               align: 'center',
               width: 80,
               key: 'checkBox',
               title: '选择',
               render: (h, params) => {
                 return h('div', [
                   h('Checkbox', {
                     props: {
                       value: params.row.checkBox
                     },
                     on: {
                       'on-change': (e) => {
                         this.productArr.forEach((items) => {      //先取消所有对象的勾选，checkBox设置为false
                           this.$set(items, 'checkBox', false)
                         });
                         this.productArr[params.index].checkBox = e;  //再将勾选的对象的checkBox设置为true
                       }
                     }
                   })
                 ])
               }
             }, 
             ......
           ]
       },
    	 // 在需要的事件监听获取到的单选
      handleSubmit() {
           this.selectGoods = this.productArr.filter(item => item.checkBox)
         }
     }
   </script>
   ```

   

