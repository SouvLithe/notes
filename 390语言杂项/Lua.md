# Lua语法入门
Lua 是一种轻量小巧的脚本语言，用标准C语言编写并以源代码形式开放， 其设计目的是为了嵌入应用程序中，从而为应用程序提供灵活的扩展和定制功能。官网：[Lua官网](https://www.lua.org/)
## 学习目的
![[Pasted image 20251002210544.png]]
在Nginx里面去做编程，可实现业务编写

| 数据类型     | 描述                                                                                                                    |
| -------- | --------------------------------------------------------------------------------------------------------------------- |
| nil      | 这个最简单，只有值nil属于该类，表示一个无效值（在条件表达式中相当于false）。                                                                            |
| boolean  | 包含两个值：false和true                                                                                                      |
| number   | 表示双精度类型的实浮点数                                                                                                          |
| string   | 字符串由一对双引号或单引号来表示                                                                                                      |
| function | 由 C 或 Lua 编写的函数                                                                                                       |
| table    | Lua 中的表（table）其实是一个"关联数组"（associative arrays），数组的索引可以是数字、字符串或表类型。在 Lua 里，table 的创建是通过"构造表达式"来完成，最简单构造表达式是{}，用来创建一个空表。 |
# 声明变量
Lua声明变量的时候，并不需要指定数据类型：
```lua
-- 声明字符串
local str = 'hello'
-- 字符串拼接可以使用 ..
local str2 = 'hello' .. 'world'
-- 声明数字
local num = 21
-- 声明布尔类型
local flag = true
-- 声明数组 key为索引的 table
local arr = {'java', 'python', 'lua'}
-- 声明table，类似java的map
local map =  {name='Jack', age=21}
```
访问table：
```lua
-- 访问数组，lua数组的角标从1开始
print(arr[1])
-- 访问table
print(map['name'])
print(map.name)
```
可以利用type函数测试给定变量或者值的类型：
```lua
>print(type("Hello world"))
string
>print(type(10.4*3))
number
```
lua字符串拼接用 .. 拼接。
# 循环：
```lua
-- 声明数组 key为索引的 table
local arr = {'java', 'python', 'lua'}

-- 遍历数组
for index,value in ipairs(arr) do
    print(index, value) 
end
```

```lua
-- 声明map，也就是table
local map = {name='Jack', age=21}

-- 遍历table
for key,value in pairs(map) do
   print(key, value) 
end
```
# 函数
定义函数的语法：
```lua
function 函数名( argument1, argument2..., argumentn)
    -- 函数体
    return 返回值
end


例如，定义一个函数，用来打印数组：
function printArr(arr)
    for index, value in ipairs(arr) do
        print(value)
    end
end
```
# 条件控制
```lua
if(布尔表达式) 
then
   --[ 布尔表达式为 true 时执行该语句块 --]
else
   --[ 布尔表达式为 false 时执行该语句块 --]
end
```
与java不同，布尔表达式中的逻辑运算是基于英文单词：
![[Pasted image 20251002210222.png]]
