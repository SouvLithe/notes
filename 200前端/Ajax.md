- 概念：Asynchronous JavaScript And XML，异步的JavaScript和XML。
- 作用：
    - 数据交换：通过Ajax可以给服务器发送请求，并获取服务器响应的数据。
    - 异步交互：可以在不重新加载整个页面的情况下，与服务器交换数据并更新部分网页的技术，如：搜索联想、用户名是否可用的校验等等。
## 同步&异步
![[Pasted image 20250903153351.png]]
## `Axios`
介绍：Axios 对原生的Ajax进行了封装，简化书写，快速开发。
![[Pasted image 20250903154310.png]]
浏览器访问的任何地址都是由get访问的。
图中url就是[[前端工程化#YAPI]]这点中的YAPI网站提供的，可换

请求方式别名
- `axios.get(url [, config])`
- `axios.delete(url [, config])`
- `axios.post(url [, data[, config]])`
- `axios.put(url [, data[, config]])`
![[Pasted image 20250903154731.png]]
项目中推荐用这个。
