
> 理解：在于间隔多少时间执行一次，而并非循环本身。
>
> 业务描述：
>
> 1. 页面初始化显示第一页数据，随后每隔十秒当前页面数据刷新；
> 2. 更改筛选条件或者更改页面直接刷新数据，随后每隔十秒当前数据刷新

```vue
<script>
export default {
  data () {
    return {
      pollingST: null
    }
  },
  methods: {
    /*下一页*/
    handlePage (current) {
      this.searchObj.page = current
      // 清除现有定时器
      clearTimeout(this.pollingST)
      // 调用轮询
      this.polling(params)
    },
    /*更改每页条数*/
    handleLimit (size) {
      this.searchObj.limit = size
      // 清除现有定时器
      clearTimeout(this.pollingST)
      // 调用轮询
      this.polling(params)
    },
    // 请求接口方法
    progressTask () {
      return new Promise(resolve => resolve({}))
    },
    // 轮询方法
    polling (params) {
      this.progressTask(params).then(res => {
        this.pollingST = setTimeout(() => {
          clearTimeout(this.pollingST)
          this.polling(params)
        }, 10000)
      })
    }
  },
  created () {
    // 调用轮询
    this.polling({ page: 1, pageSize: 5 })
  },
  destroyed () {
    clearTimeout(this.pollingST)
  }
}
</script>
```

