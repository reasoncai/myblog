---
title: Java常量定义在interface中好还是定义在class好
date: 2017-09-17 00:30:58
tags: java
categories: 编码习惯
---

项目中经常需要定义一些常量，定义方法有以下几种：

##### 1.定义在interface中
```java
public interface Constants {
    int A = 90;
    int B = 80;
    int C = 60;
}
```
- 优点：接口中定义的"变量", 其实就是常量, 接口中的"变量"默认都是 "public static final"类型，即为常量，因此接口可以省略"public static final"而直接写成 "type variable"，在interface和class中定义相同的常量, interface生成的class文件比class生成的class文件会更小, 而且更简洁, 效率更高。

- 缺点：接口定义常量, 虽不能实例化, 但能被实现。因此有这么一种设计模式"The constant interface pattern". 所谓的"常量接口模式",  就是其他类要使用接口中定义的常量, 就实现该接口. 我认为这是对接口的烂用。 接口中定义的常量应为所有类频繁使用的常量, 但并不是每个类都使用了接口中定义的所有常量, 因而导入了不必要的常量到这个类中, 并且导入后这个类的子类也会基础导入的常量, 这样会导致混乱, 应当避免此种用法。

##### 2.定义在普通的class中
```java
public final class Constants {
    private Constants() {}
    public static final int A = 90;
    public static final int B = 80;
    public static final int C = 60;
}
```
- 优点：final类修饰类防止被继承，私用构造方法防止实例化。

- 缺点：代码量比用接口的多。

##### 3.定义在抽象类中
```java
public abstract class Constants {
    public static final int A = 90;
    public static final int B = 80;
    public static final int C = 60;
}
```
- 优点：抽象类防止实例化。
- 缺点："abstract class"的目的是为了让其他类继承, 因此语意别扭, 同样具有interface定义常量的缺点--即可以被继承。

#### 总结
- 个人推荐用方法1-interface定义常量，代码更加简洁，生成的class文件更小, jvm不用考虑类的附加特性, 如方法重载等, 因而更为高效。当然方法2更加严谨。
- 建议使用常量的地方以 "接口.常量名" 的方式使用。不要使用静态导入import static XXX，因为代码的可读性会下降。
- 不要使用"常量接口模式", 此模式会导致类中的常量混乱, 增加维护难度。