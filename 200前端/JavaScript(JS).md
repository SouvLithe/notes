[[CSS]]
详情请看[JavaScript 数据类型 | 菜鸟教程](https://www.runoob.com/js/js-datatypes.html)

如今**JS的兼容性**还是一大问题。
![[Pasted image 20250520224329.png]]
## 引入方式
 >脚本可位于 HTML 的  body 或  head  部分中，或者同时存在于两个部分中。
 >JavaScript 中，常见的是驼峰法的命名规则.
 >在定义后可以通过 typeOf() 来获取JavaScript中变量的数据类型. **typeof 不能用来判断是 Array 还是Object**无法区分数组和普通对象，因为它们都返回 `"object"`
 >可以在文本字符串中使用反斜杠(在文本内插入)对代码行进行换行。
 >**字面量就是没有用标识符封装起来的量，是“值”的原始状态。**

两个引用方式：
![[Pasted image 20250902225346.png]]
## 基础语法
大概和java是一样的，注意输出语句和Java的不同
### 书写规则：
![[Pasted image 20250902225918.png]]
### 输出语句：
![[Pasted image 20250902230421.png]]
`document.write` 直接在页面中写入数据。
### 变量
JS是一门弱类型的语言，变量可以存放不同类型的值（命名建议用 驼峰命名法）。
- **`var`**：ES5 引入，具有函数（全局）作用域（块级域声明的值可被全局域访问）。
- **`let`**：ES6 引入，具有块级作用域。
- **`const`**：ES6 引入，具有块级作用域，且值不可变。
**var 声明特点：**
- 变量可以重复声明（覆盖原变量）。
- 变量未赋值时，默认值为 undefined。
- var 声明的变量会提升（Hoisting），但不会初始化。
**add:**
- var f = function () {} 和 function f () {} 的区别（有var就有内存）
- JavaScript 允许重复声明变量，后声明的覆盖之前的
- JavaScript 没有重载这个概念，它仅依据函数名来区分函数。后定义的同名函数覆盖之前的，与参数无关。(**JavaScript 允许重复定义函数**)
![[Pasted image 20250520225543.png]]

### 数据类型&算数符
![[Pasted image 20250902232105.png]]
**基本类型的变量是存放在栈内存（Stack）里的**
**引用类型的值是保存在堆内存（Heap）中的对象（Object）**

**null and undefined**
- 在 JavaScript 中, null 用于对象, undefined 用于变量，属性和方法。
- 对象只有被定义才有可能为 null，否则为 undefined。
- 如果我们想测试对象是否存在，在对象还没定义时将会抛出一个错误。
算数符：
![[Pasted image 20250902232955.png]]
强制转换：
- 字符串类型转为数字：
    - `<parseInt`>将字符串字面值转为数字。如果字面值不是数字，则转为NaN。
- 其他类型转为boolean：
    - Number：0 和 NaN为false，其他均转为true。
    - String：空字符串为false，其他均转为true。
    - Null 和 undefined：均转为false。
## 函数
![[Pasted image 20250902233802.png]]
第二种定义方式：
![[Pasted image 20250902234235.png]]
JS中函数可以传递任意个数的参数，只是多出的参数不会参与函数计算。
## 对象
只写JS较为特有的，其余查文档。
### 自定义对象：
![[Pasted image 20250903000230.png]]
### JSON
- 概念：JavaScript Object Notation，JavaScript对象标记法。
- JSON 是通过 JavaScript 对象标记法书写的  文本  。
- 由于其语法简单，层次结构鲜明，现多用于作为  数据载体  ，在网络中进行数据传输。
![[Pasted image 20250903000747.png]]
和JS自定义对象的区别是：它的键必须加双引号（单引号不行）
### BOM
概念：Browser Object Model 浏览器对象模型，允许JS与浏览器对话，JS 将浏览器的各个组成部分封装为对象。
- 组成：
    - Window：浏览器窗口对象
    - Navigator：浏览器对象
    - Screen：屏幕对象
    - History：历史记录对象
    - Location：地址栏对象
目前重点是Window和Location这两个。
#### Window
![[Pasted image 20250903001410.png]]
confirm方法的返回值是boolean。
调用Window时，其本身可省略。
#### Location
![[Pasted image 20250903001656.png]]
### DOM
- 概念：Document Object Model，文档对象模型。
- 将标记语言的各个组成部分封装为对应的对象：
    - Document：整个文档对象
    - Element：元素对象
    - Attribute：属性对象
    - Text：文本对象
    - Comment：注释对象
对比着下图理解这五个对象：
![[Pasted image 20250903001943.png]]
DOM树：
转为DOM对象时内存会形成以下结构，称之为DOM树。
![[Pasted image 20250903002037.png]]
- JavaScript 通过DOM，就能够对HTML进行操作：
    - 改变 HTML 元素的内容
    - 改变 HTML 元素的样式（CSS）
    - 对 HTML DOM 事件作出反应
    - 添加和删除 HTML 元素

DOM分类：
1. Core DOM - 所有文档类型的标准模型（父）
2. XML DOM - XML 文档的标准模型（子）
3. HTML DOM - HTML 文档的标准模型（子）
在Core DOM中是把任意元素即标签都封装成Element对象，而扩充的HTML DOM是把每个元素标签都单独封装成不同的对象，如img----Image对象
#### 使用
![[Pasted image 20250903002916.png]]
## 事件监听
- 事件：HTML事件是发生在HTML元素上的“事情”。比如：
    - 按钮被点击
    - 鼠标移动到元素上
    - 按下键盘按键
- 事件监听：JavaScript可以在事件被侦测到时执行代码。
### 事件绑定
![[Pasted image 20250903003920.png]]
1. 通过将HTML和JS耦合在一起实现事件绑定
2. 通过document获取指定的DOM元素，从而获取到该元素的事件属性来完成绑定。
### 常见事件
| 事件名           | 说明           |
| ------------- | ------------ |
| `onclick`     | 鼠标单击事件       |
| `onblur`      | 元素失去焦点       |
| `onfocus`     | 元素获得焦点       |
| `onload`      | 某个页面或图像被完成加载 |
| `onsubmit`    | 当表单提交时触发该事件  |
| `onkeydown`   | 某个键盘的键被按下    |
| `onmouseover` | 鼠标被移到某元素之上   |
| `onmouseout`  | 鼠标从某元素移开     |
