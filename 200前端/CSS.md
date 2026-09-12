css最好在头部引入，因为html渲染时，会先解析头部。[[html]]

![[Pasted image 20250520161510.png]]

___
## CSS 选择器
CSS 选择器用于“查找”（或选取）要设置样式的HTML 元素。
我们可以将CSS 选择器分为五类： 
◦ 简单选择器（根据名称、id、类来选取元素） 
◦ 组合器选择器 （根据它们之间的特定关系来选取元素） 
◦ 伪类选择器 （根据特定状态选取元素）
◦ 伪元素选择器 （选取元素的一部分并设置其样式） ◦ 属性选择器 （根据属性或属性值来选取元素）
```
h1 { text-align: center; color: red; }
h2 { text-align: center; color: red; } 
p { text-align: center; color: red; } 
h1, h2, p { text-align: center; color: red;
```
![[Pasted image 20250520162733.png]]
页面中的所有样式将按照以下规则“层叠”为新的“虚拟”样式表，其中第一优先级最高： 
![[Pasted image 20250520163716.png|300]]
1. 行内样式（在HTML 元素中） 
2. 外部和内部样式表（在head 部分）   
3. 浏览器默认样式                              


---
## CSS color
指定颜色是通过使用预定义的颜色名称，或RGB、HEX、HSL、RGBA、HSLA 值。
![[Pasted image 20250520164337.png]]
border-color 属性可以设置一到四个值（用于上边框、右边框、下边框和左边框）。 border-color: red green blue yellow; /* 上红、右绿、下蓝、左黄 */

## 外边距合并（margin塌陷）
**外边距合并**指的是，当两个垂直外边距相遇时，它们将形成一个外边距。合并后的 外边距的高度等于两个发生合并的外边距的高度中的较大者.
![[Pasted image 20250520165708.png]]

![[Pasted image 20250520170456.png]]
![[Pasted image 20250520171222.png]]
margin塌陷是有作用的
![[Pasted image 20250520171252.png]]
实在想在margin塌陷留空，可以是由padding。
CSS宽高
![[Pasted image 20250520171439.png]]
上图的设置宽高是**基于父元素**的。
`box-sizing: border-box; /* 指定width height为盒子的高宽 */`
![[Pasted image 20250520171826.png]]
例子：
![[Pasted image 20250901185107.png]]

---
## CSS 文本

| 属性  | color | text-align | text-decoration | letter-spacing | text-indent |
| :-: | :---: | :--------: | :-------------: | :------------: | ----------- |
| 作用  | 文本颜色  |    对齐方式    |  文本装饰（删除线、下划线）  |      字间距       | 定义首行缩进      |
在HTML中无论放多少个空格，只会显示一个。可以用空格占位符：&nbsp

---
## CSS布局

display
>隐藏元素-display:none 还是 visibility:hidden？
>通过将display 属性设置为none 可以隐藏元素。该元素将被隐藏，并且页面将显示为好像该元 素不在其中。
>visibility:hidden; 也可以隐藏元素。 但是，该元素仍将占用与之前相同的空间。

position
>position: static; 的元素不会以任何特殊方式定位；它始终根据页面的正常流进行定位。
>position: relative; 的元素相对于其正常位置进行定位。
>position: fixed; 的元素是相对于视口定位的，这意味着即使滚动页面，它也始终位于同一位置。top、right、bottom 和 left 属性用于定位此元素。
>position: absolute; 的元素相对于最近的定位祖先元素进行定位（而不是相对于视口定位，如 fixed）
>position: sticky; 的元素根据用户的滚动位置进行定位。（消息提示框）


溢出
overflow 属性指定在元素的内容太大而无法放入指定区域时是剪裁内容还是添加滚动条。
>overflow 属性可设置以下值：  
> visible- 默认。溢出没有被剪裁。内容在元素框外渲染  
> hidden- 溢出被剪裁，其余内容将不可见 
> scroll- 溢出被剪裁，同时添加滚动条以查看其余内容 ◦
> auto- 与 scroll 类似，但仅在必要时添加滚动条


浮动和清除。
overflow 属性指定在元素的内容太大而无法放入指定区域时是剪裁内容还是添加滚动条。 
overflow 属性可设置以下值： 
>visible- 默认。溢出没有被剪裁。内容在元素框外渲染 
>hidden- 溢出被剪裁，其余内容将不可见 
>scroll- 溢出被剪裁，同时添加滚动条以查看其余内容 
>auto- 与 scroll 类似，但仅在必要时添加滚动条

向右浮动，下面的元素回向上面补；向左浮动，第一个会把第二个盖住；当一个元素过大时，想要越过它的元素会被卡住。
![[Pasted image 20250520180637.png]]