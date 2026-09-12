D: 转到D盘
![[Pasted image 20250525214205.png]]![[Pasted image 20250525221617.png]]

java是一种强类型语言，即每种数据类型都有各自的数据类型，若不是同种数据类型是无法直接计算的。    一般是做拼接。

## 新奇思路
多条件排序时可借用三目运算。
![[Pasted image 20250806163600.png]]
判断空集不加入集合的技巧:
``` Java
ArrayList<String> lines = new ArrayList<>();
String line;  
    // 判断空集不加入集合的技巧  
while ((line = reader.readLine()) != null) {  
    lines.add(line);    
}
```
关于lambda表达式引用要求外部局部变量必须是final的，怎么处理：
```java
for (int i = 0; i < 3; i++) {  
    //lambda表达式要求引用的外部局部变量必须是final,即不可变的
    int id = i;  // 这行可解决这个i一直变的问题  
    new Thread(() -> {  
        mq.put(new Message(id,"值："+id));  
    },"Producer"+i).start();  
}
```



## Adding
字符串能够接收到从  命令行  和  标准输入流（一个抽象字符流） 传来的信息。
标准输入流的一个特点：这些值在你的程序读取它们之后消失。

