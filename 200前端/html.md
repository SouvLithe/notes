
[HTML 系列教程](https://www.w3school.com.cn/h.asp)
## 标签

> `<br>` 自闭合标签，就是没有关闭标签的空元素（标签定义换行）。
>
> `<hr>` `<hr/>` 分割线，但有时hr/可以不用，它是容器分割的一种。
>
> `<a>` 只有这个标签有href属性（用于添加超链接 ）
>
> `<span>` 标签可以用来对文本进行分组，以便对特定部分的文本应用样式，而不会影响其他内容。(没有语意，一行可放多个)
>
> `<video>`标签可以包住`<source>`标签播放视频

### 表单
- 场景：在网页中主要负责数据采集功能，如 注册、登录等数据采集。
- 标签：`<form`>
- 表单项：不同类型的 input 元素、下拉列表、文本域等。
    - `<input>`：定义表单项，通过type属性控制输入形式
    - `<select>`：定义下拉列表
    - `<textarea>`：定义文本域
- 属性：
    - action：规定当提交表单时向何处发送表单数据，URL
    - method：规定用于发送表单数据的方式。GET、POST
action属性若不指定，则默认提交到当前页面。
![[Pasted image 20250902223721.png]]
#### 表单项标签
![[Pasted image 20250902224035.png]]

### 页面布局 
![[Pasted image 20250901184728.png]]
## 属性
### class
> title  鼠标悬停时显示的提示信息

### input

> **type=text**
>
> disabled 禁用表单控件，使其不可用(和input配用)
>
> readonly 设为只读，用户无法修改输入框内容
>
> value  表单控件的默认值（自动填充，可改）
>
> maxlength 输入框允许的最大字符数
>
> name 表单控件的名称，用于表单数据提交(类似于提示账户的作用)
>
> **type=checkbox**
>
> checked 设为选中状态(默认是不选中状态)

#### a

> accesskey  规定激活元素的快捷键(使用Alt+定义的键，就可以使用了)

#### video

```html
width 和 height：设置视频的显示尺寸。
autoplay：视频在页面加载完成后自动播放。
controls：显示播放组件
muted：视频默认静音。
loop：视频播放完成后自动重新播放。
poster：视频加载前显示的封面图片。
<video width="640" height="360" controls autoplay muted loop poster="thumbnail.jpg">
  <source src="example.mp4" type="video/mp4">
  <source src="example.webm" type="video/webm">
  <source src="example.ogg" type="video/ogg">
  您的浏览器不支持 HTML5 video 标签。
</video>
```
#### audio
```html
src:音频源的url
controls：显示播放组件
```

## 冷门小注释

HTML 标签对大小写不敏感.

属性对大小写不敏感，建议使用小写属性.

始终为属性值加引号.

对于超链接，使用  Target   属性，你可以定义被链接的文档在何处显示。
![[Pasted image 20251128183244.png]]

