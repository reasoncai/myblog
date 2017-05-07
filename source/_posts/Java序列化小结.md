---
title: Java序列化小结
date: 2017-05-06 20:07:59
tags: 序列化
categories: Java基础
---

Java序列化就是将一个对象转化成一串二进制表示的字节数组，通过保存或传递这些字节数据来带到持久化或通讯的目的。要序列化，对象必须实现java.io.Serializable接口。反序列化则是将这个字节数组再重新构造成对象，需要原始类作为模板，所以序列化的数据并不像class文件那样保存类的完整的结构信息。

```java
FileOutPutStream fos = new FileOutPutStream("serv.dat");
ObjectOutputStream oos = new ObjectOutputStream(fos);
SerialableObject object = new SerialableObject();
oos.writeObject(object);
oos.flush();
```

- 当父类继承Serializable接口时，所有子类都可以被序列化。
- 子类实现了Serializable接口，父类没有，父类中的属性不能序列化（不报错，数据会丢失），但是在子类中属性仍能正确序列化。
- 如果序列化的属性是对象，则这个对象也必须实现Serializable接口，否则会报错。
- 在反序列化时，如果对象有属性的修改或删减，则修改的部分属性会丢失，但不会报错。
- 在反序列化时，如果serialVersionUID被修改，则反序列化会失败。

在纯java环境下，java序列化可以用。但**个人认为还不如用fastjson序列化和反序列化（效率有人测试过比jdk序列化的高**）。如果是多语言环境，尽量用通用的数据结构传递和保存信息，如json或者xml,也可以考虑其他序列化技术protobuf,thrift,avro等等。