Sass是成熟、稳定、强大的CSS预处理器，而SCSS是Sass3版本当中引入的新语法特性，完全兼容CSS3的同时继承了Sass强大的动态功能。特性概览变量变量用来存储需要在CSS中复用的信息，例如颜色和字体。SASS通过$符号去声明一个变量。$font-stack: Helvetica, sans-serif;$primary-color: #333;body {  font: 100% $font-stack;  color: $primary-color;}嵌套SASS允许开发人员以嵌套的方式使用CSS，但是过度的使用嵌套会让产生的CSS难以维护，因此是一种不好的实践，下面的例子表达了一个典型的网站导航样式：nav {  ul {    margin: 0;    padding: 0;    list-style: none;  }  li { display: inline-block; }  a {    display: block;    padding: 6px 12px;    text-decoration: none;  }}引入SASS能够将代码分割为多个片段，并以underscore风格的下划线作为其命名前缀（_partial.scss），SASS会通过这些下划线来辨别哪些文件是SASS片段，并且不让片段内容直接生成为CSS文件，从而只是在使用@import指令的位置被导入。CSS原生的@import会通过额外的HTTP请求获取引入的样式片段，而SASS的@import则会直接将这些引入的片段合并至当前CSS文件，并且不会产生新的HTTP请求。下面例子中的代码，将会在base.scss文件当中引入_reset.scss片断。// _reset.scsshtml, body, ul, ol {  margin:  0;  padding: 0;}// base.scss@import 'reset';body {  font: 100% Helvetica, sans-serif;  background-color: #efefef;}混合
混合（Mixin）用来分组那些需要在页面中复用的CSS声明，开发人员可以通过向Mixin传递变量参数来让代码更加灵活，该特性在添加浏览器兼容性前缀的时候非常有用，SASS目前使用@mixin name指令来进行混合操作。

@mixin border-radius($radius) {
          border-radius: $radius;
      -ms-border-radius: $radius;
     -moz-border-radius: $radius;
  -webkit-border-radius: $radius;
}

.box {
  @include border-radius(10px);
}
继承
继承是SASS中非常重要的一个特性，可以通过@extend指令在选择器之间复用CSS属性，并且不会产生冗余的代码，下面例子将会通过SASS提供的继承机制建立一系列样式：

// 这段代码不会被输出到最终生成的CSS文件，因为它没有被任何代码所继承。%other-styles {
  display: flex;
  flex-wrap: wrap;}


// 下面代码会正常输出到生成的CSS文件，因为它被其接下来的代码所继承。%message-common {
  border: 1px solid #ccc;
  padding: 10px;
  color: #333;
}

.message {
  @extend %message-common;
}

.success {
  @extend %message-common;
  border-color: green;
}

.error {
  @extend %message-common;
  border-color: red;
}

.warning {
  @extend %message-common;
  border-color: yellow;
}操作符
SASS提供了标准的算术运算符，例如+、-、*、/、%。在接下来的例子里，我们尝试在aside和article选择器当中对宽度进行简单的计算。


.container { width: 100%; }

article[role="main"] {
  float: left;
  width: 600px / 960px * 100%;
}

aside[role="complementary"] {
  float: right;
  width: 300px / 960px * 100%;
}
CSS扩展引用父级选择器"&"
Scss使用"&"关键字在CSS规则中引用父级选择器，例如在嵌套使用伪类选择器的场景下：

/*===== SCSS =====*/
a {
  font-weight: bold;
  text-decoration: none;
  &:hover { text-decoration: underline; }
  body.firefox & { font-weight: normal; }
}

/*===== CSS =====*/
a {
  font-weight: bold;
  text-decoration: none; }
  a:hover {
    text-decoration: underline; }
  body.firefox a {
    font-weight: normal;
}

无论CSS规则嵌套的深度怎样，关键字"&"都会使用父级选择器级联替换全部其出现的位置：

/*===== SCSS =====*/#main {
  color: black;
  a {
    font-weight: bold;
    &:hover { color: red; }
  }
}

/*===== CSS =====*/
#main {
  color: black; }
  #main a {
    font-weight: bold; }
    #main a:hover {
      color: red; 
}

"&"必须出现在复合选择器开头的位置，后面再连接自定义的后缀，例如：

/*===== SCSS =====*/
#main {
  color: black;
  &-sidebar { border: 1px solid; }
}

/*===== CSS =====*/
#main {
  color: black; }
  #main-sidebar {
    border: 1px solid;
}

如果在父级选择器不存在的场景使用&，Scss预处理器会报出错误信息。嵌套属性
CSS许多属性都位于相同的命名空间（例如font-family、font-size、font-weight都位于font命名空间下），Scss当中只需要编写命名空间一次，后续嵌套的子属性都将会位于该命名空间之下，请看下面的代码：

/*===== SCSS =====*/
.demo {
  // 命令空间后带有冒号:
  font: {
    family: fantasy;
    size: 30em;
    weight: bold;
  }
}

/*===== CSS =====*/
.demo {
  font-family: fantasy;
  font-size: 30em;
  font-weight: bold; 
}
命令空间上可以直接书写CSS简写属性，但是日常开发中建议直接书写CSS简写属性，尽量保持CSS语法的一致性。

.demo {
  font: 20px/24px fantasy {
    weight: bold;
  }
}

.demo {
  font: 20px/24px fantasy;
    font-weight: bold;
}

