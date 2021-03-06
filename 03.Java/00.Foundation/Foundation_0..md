<details>
<summary>点击展开目录</summary>
<!-- TOC -->

- [关键字](#关键字)
    - [static](#static)
    - [final](#final)
    - [abstract](#abstract)
    - [native](#native)

<!-- /TOC -->
</details>

## 关键字

### static

可以用于修饰类,变量,方法,代码块,被修饰内容全局唯一,只加载一次,不依赖实例对象,生命周期为类级别

* 类: 被修饰的类只能作为内部类,且其中只能访问静态变量和静态方法,不依赖于外部类,区别于普通内部类需在其自身构造函数中构造外部类对象
* 方法: 存在于普通类和静态内部类中,被动执行(被调用执行).不存在于普通内部类中
* 变量: 可以存在于普通类和静态内部类
* 代码块:在加载类的时候主动执行,只在第一次创建类对象时调用,可以使用定义在其前面的静态变量,对于其后面的静态变量,只能赋值而不能访问.

> 非静态代码块:在创建对象时执行,每创建一个对象都会执行一次.执行顺序参看`jvm类加载`内容
> 静态代码块的共同点: 1. 构造方法前执行;2. 可以定义多个

### final

可用于声明类, 变量, 方法等, 使其不可被继承, 不可被修改, 不可被重写

对于修饰全局变量, 如果同时被`static`修饰, 须在声明时为其显式赋值或者使用静态代码块对其赋值

只有final修饰的变量, 可以声明时赋值, 也可以在类初始化时赋值, 如在构造函数中或普通代码块中

### abstract

用于修饰方法,使得其只有声明而没有实现, 具体在继承了该类的子类中实现.

不能同时使用的修饰符:

* final:不可重写
* private:不可继承
* static:不可重写

以上修饰符的使用都会导致子类无法重写父类的abstract方法.

### native

本地方法, 这种方法和抽象方法极其类似, 它也只有方法声明, 没有方法实现,
与抽象方法不同的是, 它把具体实现移交给了本地系统的函数库, 而没有通过虚拟机, 可以说是Java与其它语言通讯的一种机制.
使用过程:

1. 编写java代码,声明native方法
2. javac 编译代码生成class文件
3. `javah -jni class文件名`,生成`.h`文件
4. 依据`.h`文件编写c代码
5. 编译c代码为动态链接库文件

> Java语言本身是不能直接操作或访问系统底层的数据,但可以通过JNI(Java Native Interface)调用其他语言间接实现,如C等
> 通常是为了提高性能或隐藏敏感代码而使用

**进程和线程的区别**

进程是分配资源的最小单元
线程是调度的最小单元

在操作系统设计上, 从进程演化出线程, 最主要的目的就是更好的支持SMP以及减小(进程/线程)上下文切换开销。
进程拥有一个独立的虚拟内存地址空间

[深入理解进程和线程](https://blog.csdn.net/zdy0_2004/article/details/44259391)

**Java中有几种类型的流?JDK为每种类型的流提供了一些抽象类以供继承, 请说出他们分别是哪些类?**

* 字节流:`InputStream`,`OutputStream`,可以处理任何类型的对象
* 字符流:`Reader`,`Writer`,只能用于处理字符

> 不能直接处理Unicode字符, 而字符流就可以

**序列化的方式**

* 实现`Serializable`接口
* 实现`Externalizable`接口,该接口继承自Serializable,并增加writeExternal和readExternal方法
* json实现
* protostuff实现
