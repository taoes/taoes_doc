[TOC]

### 1. {} 和 # 的区别?

- `${}`是 Properties 文件中的变量占位符，它可以用于标签属性值和 sql 内部，属于原样文本替换，可以替换任意内容，比如${driver}会被原样替换为`com.mysql.jdbc. Driver`。
- `#{}`是 sql 的参数占位符，MyBatis 会将 sql 中的`#{}`替换为? 号，在 sql 执行前会使用 PreparedStatement 的参数设置方法，按序给 sql 的? 号占位符设置参数值，同时也能防止SQL注入.



### 2. Mybatis的方法可以重载吗

可以重载,但是XML的方法ID必须只能有一个,因为XML映射是根据`接口名+方法名`限定的,如

```java
public interface UserMapper{
    
    User getUserById();
    
    User getUserById(@Param("id") Long id);
}

// XML 文件中的SQL必须兼容2个接口定义
```

### 3. MyBatis 的动态SQL是什么?

MyBatis 动态 sql 可以让我们在 xml 映射文件内，以标签的形式编写动态 sql，完成逻辑判断和动态拼接 sql 的功能。其执行原理为，使用 OGNL 从 sql 参数对象中计算表达式的值，根据表达式的值动态拼接 sql，以此来完成动态 sql 的功能。

MyBatis 提供了 9 种动态 sql 标签:

- `<if></if>`
- `<where></where>(trim,set)`
- `<choose></choose>（when, otherwise）`
- `<foreach></foreach>`
- `<bind/>`