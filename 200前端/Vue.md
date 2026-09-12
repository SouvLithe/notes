基于MVVM(`Model-View-ViewModel`)思想，实现数据的双向绑定，将编程的关注点放在数据上。
![[Pasted image 20250903143710.png]]
插值表达
- 形式：{{ 表达式 }}。
- 内容可以是：变量、三元运算符、函数调用、算术运算
## 常用指令：
指令：HTML 标签上带有 v- 前缀 的特殊属性，不同指令具有不同含义。例如：v-if，v-for...
![[Pasted image 20250903144744.png]]
注意：
	通过v-bind或者v-model绑定的变量，必须在数据模型（data：{}）中声明。
	v-show不论是否为true都会渲染<加载到页面上>，通过CSS样式的display属性控制是否隐藏。
	v-if：为true会加载到页面上。
![[Pasted image 20250903145325.png]]
![[Pasted image 20250903145858.png]]
## 声明周期
生命周期的八个阶段：每触发一个生命周期事件，会自动执行一个生命周期方法(钩子)。
```markdown
| 状态           | 阶段周期   |
|----------------|------------|
| beforeCreate   | 创建前     |
| created        | 创建后     |
| beforeMount    | 挂载前     |
| mounted        | 挂载完成   |
| beforeUpdate   | 更新前     |
| updated        | 更新后     |
| beforeDestroy  | 销毁前     |
| destroyed      | 销毁后     |
```
![[Pasted image 20250903150904.png]]
mounted：挂载完成，Vue初始化成功，HTML页面渲染成功。（发送请求到服务端，加载数据）
也是在mounted发送[[Ajax]]异步请求，加载数据。
## 项目

```bash
# 创建项目
vue create vue-project01  # 命令行
vue ui  # 图形化界面

# 启动项目，cd到当前目录
npm run server
```
结构：
![[Pasted image 20250903162934.png]]
流程：
![[Pasted image 20250903163518.png]]
在index.html默认引入文件main.js.
vue文件:
![[Pasted image 20250903163833.png]]
script块中的data可定义数据模型,模型需封装成函数  而非  对象(之前的是对象)并返回.
data同级定义其它方法.
## Element
Element：是饿了么团队研发的，一套为开发者、设计师和产品经理准备的基于 Vue 2.0 的桌面端组件库。
组件：组成网页的部件，例如超链接、按钮、图片、表格、表单、分页条等等。
### 快速入门
![[Pasted image 20250903165131.png]]
有点类似于bootstrap。
常用的组件：Table表格、Pagination分页、Dialog对话框、Form表单

页面组装步骤：
- 创建页面，完成页面的整体布局规划
- 布局中各个部分的组件实现
- 列表数据的异步加载，并渲染展示

## Vue路由
介绍：Vue Router 是 Vue 的官方路由。
组成：
- VueRouter：路由器类，根据路由请求在路由视图中动态渲染选中的组件
- `<router-link>`：请求链接组件，浏览器会解析成 `<a>`
- `<router-view>`：动态视图组件，用来渲染展示与路由路径对应的组件
![[Pasted image 20250903235718.png]]
路由定义在  index.html文件（当前目录）  中定义。
router-view在App.vue文件中添加， router-link在自己创建对应vue文件添加。
记得配置根目录要不然访问端口会访问一个   空白页面   。
## Nginx部署
部署只是Nginx的作用之一。
Nginx是一款轻量级的Web服务器/反向代理服务器及电子邮件（IMAP/POP3）代理服务器。其特点是占有内存少，并发能力强，在各大型互联网公司都有非常广泛的使用。
![[Pasted image 20250904001556.png]]
部署：将打包好的 `dist` 目录下的文件，复制到nginx安装目录的html目录（默认有两个文件，直接删了）下。
启动：双击 nginx.exe 文件即可，Nginx服务器默认占用80端口号
```shell
# 查找当前进程中，那个进程占据了80端口
netstat -ano | findStr 80
```
找到 `conf/nginx.conf `文件中更改默认端口。

