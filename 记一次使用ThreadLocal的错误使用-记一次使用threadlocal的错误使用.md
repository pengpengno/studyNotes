---
title: 记一次使用ThreadLocal的错误使用
date: 2022-05-16 20:38:33.158
updated: 2022-05-16 20:40:03.021
url: /archives/记一次使用threadlocal的错误使用
categories: 
tags: 
---

## 结论
在使用 Future 异步模型或 并行流 parallelStream 操作内部千万不要使用threadLocal 会造成信息丢失
如果非要使用 可以 在furture.get中进行处理
如下图
![image-1652704762430](/upload/2022/05/image-1652704762430.png)
