###Java基础

#### 数据类型

1. 基本数据类型
   * 数值型
     * 整数类型 byte/short/int/long
     * 浮点类型 float/double
   * 字符型 char
   * 布尔型 boolean
2. 引用数据类型
   * 类 class
   * 接口 interface
   * 数组

#### String、StringBuilder、StringBuffer的区别

- String：字符串常量，不可修改，不可继承
- StringBuilder：不可继承，非线程安全，*toString*方法不会对结果进行缓存
- StringBuffer：不可继承，线程安全，*toString*方法会对结果进行缓存

####  Java序列化

- 实现方式：Serializable， Externalizable（可自定义序列化字段）
- 不序列化：transient关键字修饰
- **serialVersionUID的作用** 其目的是序列化对象版本控制，有关各版本反序列化时是否兼容
- XML、JSON、Hession、Protobuf

####类初始化顺序

- 加载
- 连接
  - 验证 确保被加载的类的准确性
  - 准备 为类的静态变量分配内存，并将其初始化为默认值
  - 解析 把类中的符号引用转换为直接引用（指针）
- 初始化
  - 初始化一个类或者接口时，并不会先初始化它的父接口
  - 初始化顺序
  - - 父类静态变量
    - 父类静态代码块
    - 子类静态变量
    - 子类静态代码块
    - 父类成员变量
    - 父类构造函数
    - 子类成员变量
    - 子类构造函数

