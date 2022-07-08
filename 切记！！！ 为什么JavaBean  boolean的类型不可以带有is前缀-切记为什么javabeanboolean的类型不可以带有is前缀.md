---
title: 切记！！！ 为什么JavaBean  boolean的类型不可以带有is前缀
date: 2022-03-04 17:10:10.918
updated: 2022-04-27 16:35:28.691
url: /archives/切记为什么javabeanboolean的类型不可以带有is前缀
categories: 
- Java
tags: 
---



## 原因
各个JSON序列化工具的实现不尽相同，在对boolean类型的字段处理时，有时辉忽略is的前缀，这一点再RPC的序列化中 尤为明显，
日后开发工作切记 ，对于标识量切记不要带有is



背景：
平时工作中大家经常使用到boolean以及Boolean类型的数据，前者是基本数据类型，后者是包装类，为什么不推荐使用isXXX来命名呢？到底是用基本类型的数据好呢还是用包装类好呢？
例子：

1.其他非boolean类型
	private String isHot;
    public String getIsHot() {
        return isHot;
    }
2.boolean类型
	private boolean isHot;
    public boolean isHot() {
        return isHot;
    }
3.包装类型
	private Boolean isHot;
    public Boolean getHot() {
        return isHot;
    }
4.不以is开头
 	private boolean hot;
    public boolean isHot() {
        return hot;
    }
5.包装类型
	private Boolean hot;
    public Boolean getHot() {
        return hot;
    }    



对于非boolean类型的参数，getter和setter方法命名的规范是以get和set开头
对于boolean类型的参数，setter方法是以set开头，但是getter方法命名的规范是以is开头
包装类自动生成的getter和setter方法的名称都是getXXX()和setXXX()
其实javaBeans规范中对这些均有相应的规定，基本数据类型的属性，其getter和setter方法是getXXX()和setXXX，但是对于基本数据中布尔类型的数据，又有一套规定，其getter和setter方法是isXXX()和setXXX。但是包装类型都是以get开头
这种方式在某些时候是可以正常运行的，但是在一些rpc框架里面，当反向解析读取到isSuccess()方法的时候，rpc框架会“以为”其对应的属性值是success，而实际上其对应的属性值是isSuccess，导致属性值获取不到，从而抛出异常。
总结：
1、boolean类型的属性值不建议设置为is开头，否则会引起rpc框架的序列化异常。

2、如果强行将IDE自动生成的isSuccess()方法修改成getSuccess()，也能获取到Success属性值，若两者并存，则之后通过getSuccess()方法获取Success属性值。

工作中使用基本类型的数据好还是包装类好
咱们举个例子，一个计算盈利的系统，其盈利比例有正有负，若使用了基本类型bouble定义了数据，当RPC调用时，若出现了问题，本来应该返回错误的，但是由于使用了基本类型，返回了0.0，系统会认为没有任何问题，今年收支平衡，而不会发现其实是出现了错误。若使用了包装数据类型Double，当RPC调用失败时，会返回null，这样直接就能看到出现问题了，而不会因为默认值的问题影响判断。

其实阿里java开发手册中对于这个也有强制规定:


因此，这里建议大家POJO中使用包装数据类型，局部变量使用基本数据类型。