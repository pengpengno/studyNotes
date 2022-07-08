---
title: 函数式编程–优雅的空指针处理类 Optional
date: 2021-08-14 09:16:31.171
updated: 2022-04-27 16:35:33.751
url: /archives/函数式编程优雅的空指针处理类optional
categories: 
- Java
tags: 
---



# 函数式编程--优雅的空指针处理类 Optional

######  前言

作为开发，在日常写业务逻辑时会经常碰到需要对对象进行非空判断的情况，在 JDK 1.8 之前个人会经常使用```!=null``` 、```!"".equals()``` ```StringUtil.isBlank```等做空指针判断，这些方法都可以有效解决``` NullPointerException```，不过最近在阅读公司项目时候发现项目中主要使用    ```Optional``` 配合流式写法来处理，较之以前方法可以避免工程中大量```if{...}else{...}``` 的情况（可读性很差，一段上百行，项目不可能是只有自己在维护）

##  一、Optional 使用指北
##### 1.1 什么是 Optional ？
Optional 是JDK8 引入的包装类，是可以存放对象的容器类，在``` java.util```包下
##### 1.2 如何使用 Optional 
Optioinal 类中提供了十二个静态方法 下面会对常用的几个静态方法进行说明

> 获取 Optional 实例

由于 Optional 对构造方法私有化，所以只能通过其提供的类方法获取实例

| 返回类型         | 方法名          | 描述 |
| ---------------- | --------------- | ---- |
| <T> Optional | *of(T value)* | 返回一个指定非null值的Optional。 |
| <T> Optional | *ofNullable(T value)* | 如果为非空，返回 Optional 描述的指定值，否则返回空的 Optional。 |
| <T> Optional | *empty()* | 获取一个 Optional 空实例 |

```
    //一个空的 Optional对象
    private static final Optional<?> EMPTY = new Optional<>();
    // 获取传入的非空对象 new Optional 实例 
    // 如果传递的参数是 null，抛出异常 NullPointerException
    public static <T> Optional<T> of(T value) {
        return new Optional<>(value);
    }

	// 传入对象如果为空则调用empty() 
    public static <T> Optional<T> ofNullable(T value) {
        return value == null ? empty() : of(value);
    }
    // 返回 静态类变量 EMPTY 一个空的 Optional
    public static<T> Optional<T> empty() {
        @SuppressWarnings("unchecked")
        Optional<T> t = (Optional<T>) EMPTY;
        return t;
    }
```

> 空值判断

| 返回类型 | 静态方法                                  | 描述                                                 |
| -------- | ----------------------------------------- | ---------------------------------------------------- |
| boolean  | *isPresent()*                             | value(Optional中的实例对象)== null ? false : true    |
| void     | *ifPresent(Consumer<? super T> consumer)* | 如果值存在则使用该值调用 consumer , 否则不做任何事情 |

```

//ifPresent(Consumer<? super T> consumer)使用实例
Optional<User> user = Optional.ofNullable(getUserById(id));
user.ifPresent(u -> System.out.println("Username is: " + u.getUsername()));
```

> orElse

| 返回类型              | 静态方法                                             | 描述                                                         |
| --------------------- | ---------------------------------------------------- | ------------------------------------------------------------ |
| T                     | orElse(T other)                                      | 如果 Optional 中有值则将其返回，否则返回 orElse 方法传入的参数 |
| T                     | orElseGet(Supplier<? extends T> other)               | 当 Optional 中有值的时候，返回值；当 Optional 中没有值的时候，返回从该 Supplier 获得的值。 |
| <X extends Throwable> | orElseThrow(Supplier<? extends X> exceptionSupplier) | 当 Optional 中有值的时候，返回值；没有值的时候会抛出异常，   |

```
//orElse
User user = Optional
        .ofNullable(getUserById(id))
        .orElse(new User(0, "Unknown"));
        
System.out.println("Username is: " + user.getUsername());

//orElseGet
User user = Optional
        .ofNullable(getUserById(id))
        .orElseGet(() -> new User(0, "Unknown"));
System.out.println("Username is: " + user.getUsername());

//orElseThrow
User user = Optional
        .ofNullable(getUserById(id))
        .orElseThrow(() -> new EntityNotFoundException("id 为 " + id + " 的用户没有找到"));
```

> map

| 返回类型    | 静态方法                                         | 描述                                                         |
| ----------- | ------------------------------------------------ | ------------------------------------------------------------ |
| Optional<U> | map(Function<? super T, ? extends U> mapper)     | 如果当前 Optional 为 Optional.empty，则依旧返回 Optional.empty；否则返回一个新的 Optional，该 Optional 包含的是：函数 mapper 在以 value 作为输入时的输出值 |
| Optional<U> | flatMap(Function<? super T, Optional<U>> mapper) | flatMap 方法与 map 方法的区别在于，map 方法参数中的函数 mapper 输出的是值，然后 map 方法会使用 Optional.ofNullable 将其包装为 Optional；而 flatMap 要求参数中的函数 mapper 输出的就是 Optional。 |
| Optional<T> | filter(Predicate<? super T> predicate)           | filter 方法接受一个 Predicate 来对 Optional 中包含的值进行过滤，如果包含的值满足条件，那么还是返回这个 Optional；否则返回 Optional.empty。 |

```

//MAP
Optional<String> username = Optional
        .ofNullable(getUserById(id))
        .map(user -> user.getUsername())
        .map(name -> name.toLowerCase())
        .map(name -> name.replace('_', ' '));
System.out.println("Username is: " + username.orElse("Unknown"));

//FLATMAP
Optional<String> username = Optional
        .ofNullable(getUserById(id))
        .flatMap(user -> Optional.of(user.getUsername()))
        .flatMap(name -> Optional.of(name.toLowerCase()));
        
System.out.println("Username is: " + username.orElse("Unknown"));

//FILTER
Optional<String> username = Optional
        .ofNullable(getUserById(id))
        .filter(user -> user.getId() < 10)
        .map(user -> user.getUsername());
        
System.out.println("Username is: " + username.orElse("Unknown"));
```

Optional 的方法还有很多，在JDK9时有对其部分方法进行了增强，总得来说 在学习使用了 Optional 优雅的非空处理后，再也不用担心别人说代码写得烂了。