---
title: ApplicationEventPublisher的事件处理
date: 2021-12-27 19:42:15.505
updated: 2022-04-27 16:35:36.024
url: /archives/applicationeventpublisher的事件处理
categories: 
tags: 
---



## Spring中的事件处理
### 100 使用场景
在业务设计中 常常会有 不同业务类相互间钉同步或异步处理（即BeanA   BeanB之间钉信息交互）
### 200 设计原理
![ApplicationPushlier的设计原理](http://180.76.240.8:8090/upload/2021/12/ApplicationPushlier%E7%9A%84%E8%AE%BE%E8%AE%A1%E5%8E%9F%E7%90%86-e2138b54f7dd455b830c0e67e8570c99.png)
### 300 代码实现
#### 310 自定义事件（实现applicationEvent）
```
public class OrderEvent extends ApplicationEvent {
    private Object object;
    public OrderEvent(Object source,Object t) {
        super(source);
        this.object=t;
    }

    public Object getObject() {
        return object;
    }

    public void setObject(Object object) {
        this.object = object;
    }
}

```
#### 320 定义事件的监听器（实现ApplicationListener）
```
@Component
public class OrderEventListener implements ApplicationListener<OrderEvent> {

   @Async
    @Override
    public void onApplicationEvent(OrderEvent event) {
        //真正做业务的地方
       try {
           System.out.println("开始做事"+ Thread.currentThread().getName());
           Thread.sleep(2000);
       } catch (InterruptedException e) {
           e.printStackTrace();
       }

       String s = event.getObject().toString();
       System.out.println("结束做事"+s);
    }
}

```
#### 330 使用applicationContext 对事件进行发布
```
@Component
public class DemoEventPublisher {

    @Autowired
    private ApplicationContext applicationContext;

    public void pushlish(Object o){
        applicationContext.publishEvent(new OrderEvent(this,o));
    }
}
```
#### 340 推送事件
```


public class TestEvent {
   @Autowired
    private ApplicationContext applicationContext;
    public static void main(String[] args) {

        applicationContext.publishEvent(new OrderEvent(applicationContext,"ddddd"));
        applicationContext.publishEvent(new OrderEvent(applicationContext,"2222"));
        applicationContext.publishEvent(new 
    }
}


```