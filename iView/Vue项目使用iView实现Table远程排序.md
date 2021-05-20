> iView中可以通过给列表中需要实现排序的字段设置 `sortable: true`，但是当列表中的数据量比较多时，列表中会有分页，此时只能对当前页进行排序，针对这个问题，iView中有一个远程排序功能，可以通过远程排序实现多页数据的排序。

步骤：

1. 在Table中监听触发排序的事件（`on-sort-change`）

   ```vue
   <Table border stripe 
          @on-sort-change="sortCustomer"
          :loading="loading" 
          :columns="columns" 
          :data="dispatchData">
   </Table>
   ```

2. 将需要排序的字段的`sortable`属性的值设置为`custom`

   ```json
   {title: '开始时间', key: 'startTime', align: "center", sortable: "custom"}, 
   {title: '结束时间', key: 'lastTime', align: "center", sortable: "custom"}
   ```

3. 在数据查询对象中增加用于字段排序的属性，其中`filed`表示要排序的字段，`sortType`表示排序的类型

   ```json
   searchObj: {
   	filed: '',
     sortType: ''
   }
   ```

4. 每触发一次字段排序，都调用一次获取列表的方法，并将当前排序的字段名和排序方式通过api传递给后台

   ```javascript
   // 对信息排序
   sortCustomer(column) {
     // 排序的字段
     this.searchObj.filed = column.key
     // 排序的方式
     this.searchObj.sortType = column.order
     // 调用获取列表
     // this.getCustomerList()
   }
   ```

   

