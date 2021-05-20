1. 双重循环：

   ```javascript
   Array.prototype.unique = function () {
     const newArray = [];
     let isRepeat;
     for (let i = 0; i < this.length; i++) {
       isRepeat = false;
       for (let j = 0; j < newArray.length; j++) {
         if (this[i] === newArray[j]) {
           isRepeat = true;
           break;
         }
       }
       if (!isRepeat) {
         newArray.push(this[i]);
       }
     }
     return newArray;
   }
   ```

2. Array.prototype.indexOf()：

   ```javascript
   let arr = [1, 2, 3, 22, 233, 22, 2, 233, 'a', 3, 'b', 'a'];
   Array.prototype.unique = function () {
     const newArray = [];
     this.forEach(item => {
       if (newArray.indexOf(item) === -1) {
         newArray.push(item);
       }
     });
     return newArray;
   }
   ```

3. Array.prototype.sort()：

   ```javascript
   Array.prototype.unique = function () {
     const newArray = [];
     this.sort();
     for (let i = 0; i < this.length; i++) {
       if (this[i] !== this[i + 1]) {
         newArray.push(this[i]);
       }
     }
     return newArray;
   }
   ```

4. Array.prototype.includes()：

   ```javascript
   Array.prototype.unique = function () {
     const newArray = [];
     this.forEach(item => {
       if (!newArray.includes(item)) {
         newArray.push(item);
       }
     });
     return newArray;
   }
   ```

5. Array.prototype.reduce()：

   ```javascript
   Array.prototype.unique = function () {
     return this.sort().reduce((init, current) => {
       if(init.length === 0 || init[init.length - 1] !== current){
         init.push(current);
       }
       return init;
     }, []);
   }
   ```

6. 对象键值对：

   ```javascript
   Array.prototype.unique = function () {
     const newArray = [];
     const tmp = {};
     for (let i = 0; i < this.length; i++) {
       if (!tmp[typeof this[i] + this[i]]) {
         tmp[typeof this[i] + this[i]] = 1;
         newArray.push(this[i]);
       }
     }
     return newArray;
   }
   ```

7. Map：（最优的数组去重算法）

   ```javascript
   Array.prototype.unique = function () {
     const tmp = new Map();
     return this.filter(item => {
       return !tmp.has(item) && tmp.set(item, 1);
     })
   }
   ```

8. Set：

   ```javascript
   Array.prototype.unique = function () {
     return [...new Set(this)];
   }
   ```

   
