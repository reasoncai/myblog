---
title: Java方法参数传递
date: 2017-05-28 23:06:25
tags:
categories: JAVA基础
---

### 1.基础概念
- 值传递表示方法接收的是调用者提供的值。
- 引用传递表示方法接收的是调用者提供的变量地址

Java总是采用值传递的方式。不像C++有值传递和引用传递，引用参数用&符号表示。
### 2.情况分析
#### 2.1.参数为基本类型，方法内改变参数的值
```java
public void static tripleValue(int x){
    x = x * 3; //percent值不变
}
int percent = 10;
tripleValue(percent);
System.out.println(percent);
```
调用这个方法后percent的值还是10，执行过程如下：
1. x被初始化为percent值得一个拷贝（也就是10）。
2. x被乘以3后等于30,但是percent仍然是10。
3. 这个方法结束后，参数变量x不再使用。如图:
   ![](http://ooxz0ztfx.bkt.clouddn.com/cs1.png)
#### 2.2.参数为对象引用，修改对象的状态
```java
public static void tripleSalary(Employee x){
    x.salary = x.salary * 3;  //harry的salary属性变了
}
harry = new Emploee(...);
tripleSalary(harry);
```
调用这个方法后harry的salary属性变成原来的3倍，执行过程：
1. x被初始化为harry值的拷贝，这里是一个对象的引用。
2. tripleSalary()方法应用于这个对象应用。x和harry同时引用的那个Employee对象的薪金乘以3。
3. 方法结束后，参数变量x不再使用。但对象变量harry继续用用那个薪金增至3倍的雇员对象。如图:
   ![](http://ooxz0ztfx.bkt.clouddn.com/cs2.png)
#### 2.3.参数为对象引用，交换这2个对象引用
```java
pulic static void swap(Employee x, Employee y){
    Employee tmp = x;
    x = y;
    y = tmep;  //crs和cai没有交换
}
Employee crs = new Employee(...);
Employee cai = new Employee(...);
swap(crs,cai);
```
调用这个方法后crs和cai这两个对象没有交换成功。执行过程：
1.x被初始化为crs对象引用的拷贝，y被初始化为cai对象引用的拷贝。
2.交换的是这两个拷贝。
3.在方法结束后，x,y被丢弃了。原来的变量crs和cai仍然引用这个方法调用之前所引用的对象。如图:
![](http://ooxz0ztfx.bkt.clouddn.com/cs3.png)
### 3.总结
Java中方法参数的使用情况：
- 一个方法不能修改一个基本数据类型的参数（即数值型或布尔值）。
- 一个方法可以改变一个对象参数的状态。
- 一个方法不能让对象参数引用一个新的对象（包括String）。