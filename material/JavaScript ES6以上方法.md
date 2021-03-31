##### ES6：（ECMAScript2015）

- 类

  - ```javascript
    class Animal {
      // 构造函数，实例化的时候调用，如果不指定，会有一个不带参数的默认构造函数
      constructor(name, age) {
        this.name = name;
        this.age = age;
      }
      // toString 是原型对象上的属性
      toString() {
        console.log('name:' + this.name, 'age:' + this.age)
      }
    }
    var animal = new Animal('cat', 15); // 实例化Animal
    
    class Cat extends Animal {
      constructor(action) {
        // 子类必须要在constructor中指定 super 函数，否则在新建实例的时候会报错
        // 如果没有置顶constructor，默认带super函数的constructor将会被添加
        super('cat', 15);
        this.action = action;
      }
    }
    ```

    

- 模块化

  - 模块的功能主要由 `export` 和 `import`组成
  - 每个模块都有自己单独的作用域，模块之间的相互调用关系是通过 `export`来规定模块对外暴露的接口，通过`import`来引用其他模块提供的接口，同时还为模块创造了命名空间，防止函数的命名冲突
  - 导出（`export`）
    - 导出变量，也支持常量的导出
    - 导出函数
  - 导入（`import`）
    - 一条`import `语句可以同时导入默认函数和其他变量
      - `import defaultMethod, {otherMethod} from 'x.js'`

- 箭头函数

- 函数参数默认值

- 模板字符串

  - `${}`完成字符串的拼接，将变量放在大括号中 

- 解构赋值

  - 获取数组中的值
  - 获取对象中的值

- 延展操作符

  - 函数调用
  - 数组构造或字符串
  - 构造对象时，进行克隆或属性拷贝

- 对象属性简写

- Promise

- Let 和 Const

##### ES7：

- includes()

  - `Array.prototype.includes()`函数用来判断一个数组是否包含一个指定的值，如果包含则返回 true ，否则返回 false

  - `includes` 函数 和 `indexOf` 函数类似，等价表达式：

    ```javascript
    arr.includes(x)
    arr.indexOf(x) >= 0
    let arr = ['react', 'angular', 'vue']
    if(arr.includes('react')) {
      console.log('存在')
    }
    ```

- 指数操作符

  - 指数操作符：`**`

  - 与`Math.pow(..)`等效

    ```javascript
    console.log(2**10); // 1024
    ```

##### ES8：

- async/await

  - async 声明一个function是异步的，await(async wait) 用于等待一个异步方法执行完成
  - await只能出现在async函数中

  > `async`函数返回的是一个`Promise`对象
  >
  > `await`等待一个表达式，这个表达式的计算结果是`Promise`对象或其他值；
  >
  > `await`可以用于等待一个`async`函数的返回值；
  >
  > `await`后面可以接普通函数调用或直接量。
  >
  > `await`是个运算符，用于组成表达式，`await`表达式的运算结果取决于它等待的东西：
  >
  > 1. 若结果不是`Promise`对象，则`await`表达式的运算结果就是最终结果；
  > 2. 若结果是`Promise`对象，await会阻塞后面的代码，等着`Promise`对象`resolve`，然后得到`resolve`的值，作为`await`表达式的运算结果。`async`函数调用不会造成阻塞，它内部所有的阻塞都被封装在一个`Promise`对象中异步执行。

- Object.values()

  - `Object.values()`是一个与`Object.keys()`类似的函数，返回的是`Object`自身属性的所有值，不包括继承的值

- Object.entries()

  - 返回一个给定对象自身可枚举属性的键值对的数组

- String padding

  - 将空字符串或其他字符串添加到原始字符串的开头`String.prototype.padStart`或结尾`String.prototype.padEnd`
  - `String.padStart(targetLength, [padString])`

- 函数参数列表结尾允许逗号

- Object.getOwnPropertyDescriptors()

  - 用来获取一个对象的所有自身属性的描述符，若没有任何自身属性，则返回空对象。
  - `Object.getOwnPropertySescriptors(obj)`

##### ES10:

- Array方法：`flat()`和`flatMap()`

  - `flat()`和`flatMap()`本质上就是归纳（reduce）与合并（concat）的操作

  - `Array.prototype.flat()`：

    - `flat()`方法会按照一个可指定的深度递归遍历数组，并将所有元素与遍历到的子数组中的元素合并为一个新数组返回

    - `flat()`方法最基本的作用就是数组降维

    - 使用 `Infinity` 作为深度，展开任意深度的嵌套数组：`flat.(Infinity )`

    - `flat()`方法可以去除数组的空项

    - ```javascript
      var arr1 = [1, 3, 4, , 45]
      console.log(arr1.flat())
      ```

  - `Array.prototype.flatMap()`：

    - 首先使用映射函数映射每个元素，然后将结果压缩成一个新数组。
