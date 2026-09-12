## shell command of MySQL

这个文件简要记录一下mysql使用期间所用到的shell命令，方便以后查询

---
本人Mysql系统服务名称：MySQL80
以下命令需要以管理员身份运行cmd：

```mysql
 net start mysql80   //开启MySQL系统服务  
 net stop mysql80    //关闭MySQL系统服务  
 mysql -u root -p    //建立mysql的root账户
```

mysql用户名是：root
mysql密码是：123456
## 记得启动链接MySQL开网络端口！！！！


---
## 修改MySQL密码：

### 第一步：管理员命令行
![[Pasted image 20250529224453.png]]
打开后找到 MySQL80（相应版本服务），如果是开启的要给它关了。
![[Pasted image 20250529224658.png]]
接着双击打开，  
--defaults-file="D:\MySQL\MySQL Server 8.0\my.ini"
从  --defaults-file  复制到  my.ini。
![[Pasted image 20250529224813.png]]
接着在，管理员身份的cmd窗口下输入：
```bash
mysqld <刚刚复制的代码> --shared-memory --skip-grant-tables
```
然后会卡住不用管，开个一般的cmd。
### 一般命令行窗口
依次输出：
```shell
flush privileges;  //更新用户列表
aliter user '<新用户名>'@'localhost' identified by '<新密码>';
exit;
```
依次是：刷新用户列表、更改  用户名  和  密码、退出。
### 返回那个卡住的管理员身份窗口
中断进程。
```
输入：mysql -uroot -p'<新密码>'    测试是否更改成功.
```
