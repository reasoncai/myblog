---
title: npm install安装时忘记--save解决方法
date: 2017-05-07 20:17:54
tags: npm
categories:
---
![image](http://ooxz0ztfx.bkt.clouddn.com/npminit.png)
网上还有一个解决方案就是：
```
npm install `ls node_modules` --save
```
或
```
npm install --save $(ls node_modules)
```

**假如忘记保存的包太多，上面的方法都太麻烦了，直接```npm init``` 重新生成package.json搞定。**
