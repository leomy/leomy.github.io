---
title: ClassFile - JSR202
toc: true
date: 2018-03-31 17:55:23
categories:
- jsr
tags: [Java, JSR]
---
字节码文件作为JVM语言编译后的产物，在<a href="https://jcp.org/en/jsr/detail?id=202">JSR202</a>中详细描述了字节码文件结构等。字节码生命周期包括加载、连接（验证/准备/解析）、初始化、使用和卸载。
## 结构
一个字节码文件只包含单个的`class`或`interface`的定义信息，不是所有的字节码文件都要求在存储在文件中（如通过`ClassLoader`生成的`class`）。一个字节码文件由8位的字节流组成，16、32、64位分别通过连续读2、4、8次8位字节构成，字节码采用大端（`big-endian`，高位在前、低位在后）存储。u1-无符号的1字节，u2-无符号的2字节，u4-无符号的四字节。字节码文件采用C风格的`struct`来书写伪代码，为避免混乱，描述字节码结构的的被称为`items`，相邻的`items`按照顺序中间无填充任何空白存储在字节码文件中，零或多个变长的`items`组成了`table`（即不能通过索引获得某个`items`在`table`的字节偏移量）

一个字节码文件由以下结构组成
```
ClassFile {
    u4                  magic;
    u2                  minor_version;
    u2                  major_version;
    u2                  constant_pool_count;
    cp_info             constant_pool[constant_pool_count-1];
    u2                  access_flags;
    u2                  this_class;
    u2                  super_class;
    u2                  interfaces_count;
    u2                  interfaces[interfaces_count];
    u2                  fields_count;
    field_info          fields[fields_count];
    u2                  methods_count;
    method_info         methods[methods_count];
    u2                  attributes_count;
    attribute_info      attributes[attributes_count];
}
```

- `magic`：该项提供魔数来识别文件格式是否是字节码文件格式，其值必须是`OxCAFEBABE`
- `minor_version`,`major_version`：分别代表字节码文件的主次版本号，主次版本号一起确定字节码文件格式的版本号
- `constant_pool_count`： `constant_pool_count`是`constant_pool[]`table的条目数加1。`constant_pool`的索引有效值是大于0且小于`constant_pool_count`
- `constant_pool[]`：`constant_pool[]`用一个`table`结构代表所有的字符串常量（类名、接口名、字段名和在`ClassFile`结构及其子结构中出现的常量）。每个`items`入口由第一个`tag`标示出。`constant_pool[]`的索引从1到`constant_pool_count - 1`
- `access_flags`：标志的掩码（mask，每一个access_flag可以相加），用来表示访问权限、`class`或`interface`的属性。对于每一个可以设置标志，下表都有解释：
    - `ACC_SYNTHETIC` 位被设置，表明它是由`compiler`生成，并且在源码中不会出现
    - `ACC_ENUM` 位被设置，则表明该类或父类为枚举类型
    - `ACC_INTERFACE` 位被设置（`ACC_ABSTRACT`位必须设置，但`ACC_FINAL`、`ACC_SUPER`或`ACC_ENUM`位不能被设置），表明是一个接口而不是类。反之是一个类而不是接口
    - `ACC_ANNOTATION` 位被设置（`ACC_INTERFACE`位必须设置），表明是一个枚举类型
    如果`ACC_INTERFACE`标志位没有被设置，那么在表中除了`ACC_ANNOTATION`标志位外，其他都可以设置。但一个类不能同时有`ACC_FINAL`和`ACC_ABSTRACT`标志位被置位
    
| Flag Name        | Value           | Interpretation  |
| :--------------: | :-------------: | :-------------: |
| ACC_PUBLIC       | 0x0001 |`public`声明：可以在其他`package`下访问  |
| ACC_FINAL        | 0x0010          | `final`声明：不允许有子类      |
| ACC_SUPER        | 0x0020 |处理指定的调用`invokespecial`指令父类方法 |
| ACC_INTERFACE    | 0x0200          | 只是一个接口，不是类           |
| ACC_ABSTRACT     | 0x0400          | 抽象声明：绝对不能被实例化      |
| ACC_SYNTHETIC    | 0x1000          | 合成声明：不在源码中           | 
| ACC_ANNOTATION   | 0x2000          | 注解类型声明                  |
| ACC_ENUM         | 0x4000          | 枚举类型声明                  |
     
