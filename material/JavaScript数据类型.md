1. 基本数据类型：string、null、undefined、number、symbol、Boolean

   - 基本数据类型的值是按值访问的
   - 基本数据类型的值是不可变的
   - 基本数据类型的计较是它们值的比较
   - 基本数据类型的变量是存放在栈内存（Stack）里的

   ![图片描述](https://raw.githubusercontent.com/chnjames/cloudImg/main/20210408152537.png)

2. 引用数据类型：（统称Object类型：细分为Object类型、Array类型、Date类型、RegExp类型、Function类型等）

   - 引用类型的值是可变的
   - 引用类型的比较是引用的比较引用类型的值是保存在堆内存（Heap）中的对象

   ![图片描述](https://raw.githubusercontent.com/chnjames/cloudImg/main/20210408152944.png)

   > 栈内存中保存了变量标识符和指向堆内存中该对象的指针
   >
   > 堆内存中保存了对象的内容

3. 检测类型：

   - typeof：检测一个变量的数据类型
   - instanceof：用来判断某个构造函数的prototype属性所指向的对象是否存在于另外一个要检测对象的原型上，简单来说就是判断一个引用数据类型的变量具体是某种类型的对象
