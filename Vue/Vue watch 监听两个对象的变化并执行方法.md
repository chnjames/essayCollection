使用`computed` 定义一个新的对象：`newObj`，然后再去 `watch newObj`

```javascript
export default {
  data() {
    return {
      objOne: '',
      ObjTwo: ''
    }
  },
  computed: {
    newObj() {
      const {objOne, objTwo} = this
      return {
        objOne,
        objTwo
      }
    }
  },
  watch: {
    newObj(val) {
      console.log(val)
    }
  }
}
```

