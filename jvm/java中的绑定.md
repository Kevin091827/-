# 浅谈java中的绑定

## 1.什么是绑定？
绑定是指一个方法的调用和该方法所属的类（所在的类）相关联，意思就是在执行方法调用的时候，jvm所知道调用了哪个类的方法，类和调用方法相关联

**java绑定分类：**

* 静态绑定（前期绑定）
* 运行时绑定（后期绑定）

## 2.静态绑定

### 什么是静态绑定？

静态绑定就是程序在执行前就知道了该方法所属的类，即在编译前该方法已经绑定，在java中只有private ， static ，final修饰的方法和构造方法是静态绑定

### 浅谈java中只有private ， static ，final修饰的方法和构造方法是静态绑定

**1.private：** 

private所修饰的方法是无法继承的，也就是说子类无法调用该方法，即private修饰的方法还是和定义该方法的类关联在一起

**2.static：**

static这个有点复杂，static定义的方法可以继承，但是子类不能重写父类的静态方法，但是如果子类有同名的静态方法子类可以隐藏父类的该方法，也就是说，如果父类里有一个static方法，它的子类里如果没有对应的方法，那么当子类对象调用这个方法时就会使用父类中的方法。而如果子类中定义了相同的方法，则会调用子类的中定义的方法。唯一的不同就是，当子类对象上转型为父类对象时，不论子类中有没有定义这个静态方法，该对象都会使用父类中的静态方法。

**3.final：**

final所修饰的方法虽然可以继承，但是却无法覆盖（重写），也就是说，虽然是在子类中调用该方法，实际上还是调用原来父类中的方法

**4.构造方法：**

构造方法也是不能被继承的，所以编译时就知道该构造方法所属的类


**结论：如果一个方法不可被继承或者继承后不可被覆盖，那么这个方法就采用的静态绑定。**

在java类的生命周期中，解析阶段可以确定调用方法的版本（即调用哪个类的方法）只有这四种，调用的字节码指令是
    
* invokestatic

* invokespecial  

* invokevirtual

他们在类加载的时候就会把符号引用解析为该方法的直接引用，这四种方法也可以叫做非虚方法

## 3.运行时绑定

### 什么是运行时绑定？
运行时绑定是指在程序运行时根据对象类型进行绑定，也叫动态绑定或者后期绑定


### 运行时绑定支持

运行时绑定是通过一个叫做方法表的结构来处理的

可参考

[jvm字节码执行引擎](https://blog.csdn.net/weixin_41922289/article/details/89623857)

