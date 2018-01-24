---
title: Java8-Optional的使用
date: 2017-09-28 23:45:26
tags: java8
categories: java基础
---

### 目的
去掉烦人的NPE异常和非NULL判断代码
### 1.三种构造函数
- Optional.of(obj)  要求传入的对象不能为null,否则立刻报NPE异常
- Optional.ofNullable(obj)  传 null 进到就得到 Optional.empty(), 非 null 就调用 Optional.of(obj)
```java
源码
    public static <T> Optional<T> ofNullable(T value) {
        return value == null ? empty() : of(value);
    }
```
- Optional.empty() 顾名思义value为null

### 2.常用方法
#### 2.1.存在即返回，否则返回默认值oreElse(T other)
```java
源码
    /**
     * Return the value if present, otherwise return {@code other}.
     *
     * @param other the value to be returned if there is no value present, may
     * be null
     * @return the value, if present, otherwise {@code other}
     */
    public T orElse(T other) {
        return value != null ? value : other;
    }
```
#### 2.2存在即返回, 无则由函数（函数式编程）来产生orElseGet(Supplier<? extends T> other)
```
    /**
     * Return the value if present, otherwise invoke {@code other} and return
     * the result of that invocation.
     *
     * @param other a {@code Supplier} whose result is returned if no value
     * is present
     * @return the value if present otherwise the result of {@code other.get()}
     * @throws NullPointerException if value is not present and {@code other} is
     * null
     */
    public T orElseGet(Supplier<? extends T> other) {
        return value != null ? value : other.get();
    }
```
#### 2.3存在则返回，无则抛出异常orElseThrow(Supplier<? extends X> exceptionSupplier) 
```java
   /**
     * Return the contained value, if present, otherwise throw an exception
     * to be created by the provided supplier.
     *
     * @apiNote A method reference to the exception constructor with an empty
     * argument list can be used as the supplier. For example,
     * {@code IllegalStateException::new}
     *
     * @param <X> Type of the exception to be thrown
     * @param exceptionSupplier The supplier which will return the exception to
     * be thrown
     * @return the present value
     * @throws X if there is no value present
     * @throws NullPointerException if no value is present and
     * {@code exceptionSupplier} is null
     */
    public <X extends Throwable> T orElseThrow(Supplier<? extends X> exceptionSupplier) throws X {
        if (value != null) {
            return value;
        } else {
            throw exceptionSupplier.get();
        }
    }
```
#### 2.4若存在则进行一些操作
```java
   /**
     * If a value is present, invoke the specified consumer with the value,
     * otherwise do nothing.
     *
     * @param consumer block to be executed if a value is present
     * @throws NullPointerException if value is present and {@code consumer} is
     * null
     */
    public void ifPresent(Consumer<? super T> consumer) {
        if (value != null)
            consumer.accept(value);
    }
```
#### 2.5map函数
```java
    public<U> Optional<U> map(Function<? super T, ? extends U> mapper) {
        Objects.requireNonNull(mapper);
        if (!isPresent())
            return empty();
        else {
            return Optional.ofNullable(mapper.apply(value));
        }
    }
```
实例：
```java
package com.cai.javademo.optional;

import java.util.Optional;

/**
 * Created by reason on 17/9/27.
 */
public class OptionalTest {
    public static void main(String[] args) throws Exception {
        Student student = getNull();
        Student defaultStudent = getStudent();
        Optional<Student> studentOptional = Optional.ofNullable(defaultStudent);
        Student student1 = studentOptional.orElse(getDefaultStudent());
        System.out.println(student1);
        Student student2 = studentOptional.orElseGet(() -> getFuntionStudent());
        System.out.println(student2);
        Student student3 = studentOptional.orElseThrow(() -> new Exception("空抛出自定义异常"));
        System.out.println(student3.toString());

        //若存在则进行某种操作（这里将名字改为modify）
        studentOptional.ifPresent((x) -> {
            x.setName("modify");
        });
        System.out.println(studentOptional.get());

        //map函数(这里将获取名字并转大写)无限级联
        String s = studentOptional.map(a -> a.getName()).map(b -> b.toUpperCase()).orElse(null);
        System.out.println(s);
    }

    private static Student getStudent() {
        Student student = new Student();
        student.setName("normal");
        student.setAge(10);
        return student;
    }

    private static Student getNull() {
        return null;
    }

    private static Student getDefaultStudent() {
        Student student = new Student();
        student.setName("default");
        student.setAge(1);
        return student;
    }

    private static Student getFuntionStudent() {
        Student student = new Student();
        student.setName("Funtion");
        student.setAge(129000);
        return student;
    }
}

```
当student不为null时输出
```
Student{name='normal', age=10}
Student{name='normal', age=10}
Student{name='normal', age=10}
Student{name='modify', age=10}
MODIFY

```
当student为null时输出
```
Student{name='default', age=1}
Student{name='Funtion', age=129000}
Exception in thread "main" java.lang.Exception: 空抛出自定义异常
	at com.cai.javademo.optional.OptionalTest.lambda$main$1(OptionalTest.java:17)
	at java.util.Optional.orElseThrow(Optional.java:290)
	at com.cai.javademo.optional.OptionalTest.main(OptionalTest.java:17)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at com.intellij.rt.execution.application.AppMain.main(AppMain.java:147)
```
### 3.总结
- 常用方法：
- oreElse(T other)
- orElseGet(Supplier<? extends T> other) 
- orElseThrow(Supplier<? extends X> exceptionSupplier) 
- map()
- 尽量不用isPresent()和get(),因为这样做与以前直接判断==null?没有什么区别。且用get()方法前要用isPresent()判断是否为null，过于啰嗦。
- 不要将Optional 类型用作属性或是方法参数，Optional 类型不可被序列化, 用作字段类型会出问题的。


其实说了那么多，还不如看看jdk源码java.util.Optional来的简单。。。。。。