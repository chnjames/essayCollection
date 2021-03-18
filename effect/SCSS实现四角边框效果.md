##### 最终效果：

![image-20210318150221114](https://raw.githubusercontent.com/chnjames/cloudImg/main/20210318150221.png)

##### 实现代码：

```scss
@mixin borderAngle($color, $width, $height) {
  background: linear-gradient(to left, $color, $color) left top no-repeat,
    linear-gradient(to bottom, $color, $color) left top no-repeat,
    linear-gradient(to left, $color, $color) right top no-repeat,
    linear-gradient(to bottom, $color, $color) right top no-repeat,
    linear-gradient(to left, $color, $color) left bottom no-repeat,
    linear-gradient(to bottom, $color, $color) left bottom no-repeat,
    linear-gradient(to left, $color, $color) right bottom no-repeat,
    linear-gradient(to left, $color, $color) right bottom no-repeat;
  background-size: $width $height, $height $width;
}
.backAngle {
  @include borderAngle(#3581e4, 3px, 30px);
  width: 500px;
  height: 300px;
  background-color: #a6ccf8;
}
```
