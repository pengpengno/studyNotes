---
title: Groovy 学习
date: 2022-02-21 10:37:10.025
updated: 2022-04-27 16:35:41.119
url: /archives/groovy学习
categories: 
tags: 
---



## Groovy 学习
[TOC]
### Groovy 简介
groovy是一种动态的语言  也即是 在编译时可以修改脚本中的属性,相对于 传统 结构性强的Java ,Groovy更适合 完成通用场景下的配置话开发.

最近设计一个数据统计系统,系统中上百种数据统计维度,而且这些数据统计的指标可能随时会调整.如果基于java编码的方式逐个实现数据统计的API设计,工作量大而且维护起来成本较高;最终确定为将"数据统计"的计算部分单独分离成脚本文件(javascript,或者Groovy),非常便捷了实现了"数据统计Task" 与 "数据统计规则(计算)"解耦,且可以动态的加载和运行的能力.顺便对JAVA嵌入运行Groovy脚本做个备忘.

    Java中运行Groovy,有三种比较常用的类支持:GroovyShell,GroovyClassLoader以及Java-Script引擎(JSR-223).

    1) GroovyShell: 通常用来运行"script片段"或者一些零散的表达式(Expression)

    2) GroovyClassLoader: 如果脚本是一个完整的文件,特别是有API类型的时候,比如有类似于JAVA的接口,面向对象设计时,通常使用GroovyClassLoader.

    3) ScriptEngine: JSR-223应该是推荐的一种使用策略.规范化,而且简便.

 

一.GroovyShell代码样例

    1) 简单的表达式执行,方法调用

Java代码   收藏代码
/** 
 * 简答脚本执行 
 * @throws Exception 
 */  
public static void evalScriptText() throws Exception{  
    //groovy.lang.Binding  
    Binding binding = new Binding();  
    GroovyShell shell = new GroovyShell(binding);  
      
    binding.setVariable("name", "zhangsan");  
    shell.evaluate("println 'Hello World! I am ' + name;");  
    //在script中,声明变量,不能使用def,否则scrope不一致.  
    shell.evaluate("date = new Date();");  
    Date date = (Date)binding.getVariable("date");  
    System.out.println("Date:" + date.getTime());  
    //以返回值的方式,获取script内部变量值,或者执行结果  
    //一个shell实例中,所有变量值,将会在此"session"中传递下去."date"可以在此后的script中获取  
    Long time = (Long)shell.evaluate("def time = date.getTime(); return time;");  
    System.out.println("Time:" + time);  
    binding.setVariable("list", new String[]{"A","B","C"});  
    //invoke method  
    String joinString = (String)shell.evaluate("def call(){return list.join(' - ')};call();");  
    System.out.println("Array join:" + joinString);  
    shell = null;  
    binding = null;  
}  
    2)  伪main方法执行.

Java代码   收藏代码
/** 
 * 当groovy脚本,为完整类结构时,可以通过执行main方法并传递参数的方式,启动脚本. 
 */  
public static void evalScriptAsMainMethod(){  
    String[] args = new String[]{"Zhangsan","10"};//main(String[] args)  
    Binding binding = new Binding(args);  
    GroovyShell shell = new GroovyShell(binding);  
    shell.evaluate("static void main(String[] args){ if(args.length != 2) return;println('Hello,I am ' + args[0] + ',age ' + args[1])}");  
    shell = null;  
    binding = null;  
}  
    3)  通过Shell运行具有类结构的Groovy脚本

Java代码   收藏代码
/** 
 * 运行完整脚本 
 * @throws Exception 
 */  
public static void evalScriptTextFull() throws Exception{  
    StringBuffer buffer = new StringBuffer();  
    //define API  
    buffer.append("class User{")  
            .append("String name;Integer age;")  
            //.append("User(String name,Integer age){this.name = name;this.age = age};")  
            .append("String sayHello(){return 'Hello,I am ' + name + ',age ' + age;}}\n");  
    //Usage  
    buffer.append("def user = new User(name:'zhangsan',age:1);")  
            .append("user.sayHello();");  
    //groovy.lang.Binding  
    Binding binding = new Binding();  
    GroovyShell shell = new GroovyShell(binding);  
    String message = (String)shell.evaluate(buffer.toString());  
    System.out.println(message);  
    //重写main方法,默认执行  
    String mainMethod = "static void main(String[] args){def user = new User(name:'lisi',age:12);print(user.sayHello());}";  
    shell.evaluate(mainMethod);  
    shell = null;  
}  
    4)  方法执行和分部调用

Java代码   收藏代码
/** 
 * 以面向"过程"的方式运行脚本 
 * @throws Exception 
 */  
public static void evalScript() throws Exception{  
    Binding binding = new Binding();  
    GroovyShell shell = new GroovyShell(binding);  
    //直接方法调用  
    //shell.parse(new File(//))  
    Script script = shell.parse("def join(String[] list) {return list.join('--');}");  
    String joinString = (String)script.invokeMethod("join", new String[]{"A1","B2","C3"});  
    System.out.println(joinString);  
    脚本可以为任何格式,可以为main方法,也可以为普通方法  
    //1) def call(){...};call();  
    //2) call(){...};  
    script = shell.parse("static void main(String[] args){i = i * 2;}");  
    script.setProperty("i", new Integer(10));  
    script.run();//运行,  
    System.out.println(script.getProperty("i"));  
    //the same as  
    System.out.println(script.getBinding().getVariable("i"));  
    script = null;  
    shell = null;  
}  
二. GroovyClassLoader代码示例

    1) 解析groovy文件

Java代码   收藏代码
/** 
 * from source file of *.groovy 
 */  
public static void parse() throws Exception{  
    GroovyClassLoader classLoader = new GroovyClassLoader(Thread.currentThread().getContextClassLoader());  
    File sourceFile = new File("D:\\TestGroovy.groovy");  
    Class testGroovyClass = classLoader.parseClass(new GroovyCodeSource(sourceFile));  
    GroovyObject instance = (GroovyObject)testGroovyClass.newInstance();//proxy  
    Long time = (Long)instance.invokeMethod("getTime", new Date());  
    System.out.println(time);  
    Date date = (Date)instance.invokeMethod("getDate", time);  
    System.out.println(date.getTime());  
    //here  
    instance = null;  
    testGroovyClass = null;  
}  
    2) 如何加载已经编译的groovy文件(.class)

Java代码   收藏代码
public static void load() throws Exception {  
    GroovyClassLoader classLoader = new GroovyClassLoader(Thread.currentThread().getContextClassLoader());  
    BufferedInputStream bis = new BufferedInputStream(new FileInputStream("D:\\TestGroovy.class"));  
    ByteArrayOutputStream bos = new ByteArrayOutputStream();  
    for(;;){  
        int i = bis.read();  
        if( i == -1){  
            break;  
        }  
        bos.write(i);  
    }  
    Class testGroovyClass = classLoader.defineClass(null, bos.toByteArray());  
    //instance of proxy-class  
    //if interface API is in the classpath,you can do such as:  
    //MyObject instance = (MyObject)testGroovyClass.newInstance()  
    GroovyObject instance = (GroovyObject)testGroovyClass.newInstance();  
    Long time = (Long)instance.invokeMethod("getTime", new Date());  
    System.out.println(time);  
    Date date = (Date)instance.invokeMethod("getDate", time);  
    System.out.println(date.getTime());  
      
    //here  
bis.close();  
    bos.close();  
    instance = null;  
    testGroovyClass = null;  
}  
三. ScriptEngine

    1) pom.xml依赖

Xml代码   收藏代码
<dependency>  
    <groupId>org.codehaus.groovy</groupId>  
    <artifactId>groovy</artifactId>  
    <version>2.1.6</version>  
</dependency>  
<dependency>  
    <groupId>org.codehaus.groovy</groupId>  
    <artifactId>groovy-jsr223</artifactId>  
    <version>2.1.6</version>  
</dependency>  
    2) 代码样例

Java代码   收藏代码
public static void evalScript() throws Exception{  
    ScriptEngineManager factory = new ScriptEngineManager();  
    //每次生成一个engine实例  
    ScriptEngine engine = factory.getEngineByName("groovy");  
    System.out.println(engine.toString());  
    assert engine != null;  
    //javax.script.Bindings  
    Bindings binding = engine.createBindings();  
    binding.put("date", new Date());  
    //如果script文本来自文件,请首先获取文件内容  
    engine.eval("def getTime(){return date.getTime();}",binding);  
    engine.eval("def sayHello(name,age){return 'Hello,I am ' + name + ',age' + age;}");  
    Long time = (Long)((Invocable)engine).invokeFunction("getTime", null);  
    System.out.println(time);  
    String message = (String)((Invocable)engine).invokeFunction("sayHello", "zhangsan",new Integer(12));  
    System.out.println(message);  
}  
    需要提醒的是,在groovy中,${expression} 将会被认为一个变量,如果需要输出"$"符号,需要转义为"\$".   

关于ScriptEngine更多介绍,请参考.