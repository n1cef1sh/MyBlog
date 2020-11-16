---
layout: post
title: "Java中的static关键字"
categories: [Java]

---

### 起因
在做课程大作业的时候要求使用java编程，菜鸡对于java的基础语法都掌握不好。过程中在使用某些变量或者方法时，IDE会提示将该变量/方法修改为static类型，虽然点一下就改好了，但是并不知道其中的原因，还是太辣鸡了，所以学习一下java中的static关键字。
### 学习
#### 类的加载机制
首先复习一下两年前学习过的[Java类的加载机制](https://www.nicefish.xyz/posts/2018/11/09/%E7%B1%BB%E7%9A%84%E5%8A%A0%E8%BD%BD%E6%9C%BA%E5%88%B6.html)  
类的生命周期包括这几个阶段
> 加载——验证——准备——解析——初始化——使用——卸载

其中两个关键阶段
- 准备 ：为类的静态变量分配内存，并将起初始化为默认值。进行内存分配的只包括类变量(static修饰的)，而不包括实例变量。
初始值是数据类型的零值，例如public static int age = 20; 经过准备阶段后，age的初始值是0，而非20。把age赋值为20的操作在初始化阶段。

- 初始化 ： 初始化是使用前的最后一个阶段。在准备阶段有一个给变量赋初始值的操作，而初始化阶段则根据用户的自定义计划对变量进行赋值或者初始化其他资源。例如上面提到的public static int age = 20;在这里就对age赋值了20.

#### static关键字
> “static方法就是没有this的方法。在static方法内部不能调用非静态方法，反过来是可以的。而且可以在没有创建任何对象的前提下，仅仅通过类本身来调用static方法。这实际上正是static方法的主要用途。”

也就是说：方便在没有创建对象的情况下来进行调用（方法/变量）。
1. static方法  
对于静态方法来说，因为它不依附于任何对象，既然都没有对象，就谈不上this了。由于这个特性，在静态方法中不能访问类的非静态成员变量和非静态成员方法，因为非静态成员方法/变量都是必须依赖具体的对象才能够被调用。但是在非静态成员方法中是可以访问静态成员方法/变量的。

![image.png](https://i.loli.net/2020/11/16/Bnx59f38iJpsTtu.png)  

2. static变量  
静态变量被所有的对象所共享，在内存中只有一个副本，它当且仅当在类初次加载时会被初始化。而非静态变量是对象所拥有的，在创建对象的时候被初始化，存在多个副本，各个对象拥有的副本互不影响。static成员变量的初始化顺序按照定义的顺序进行初始化。  
在C/C++中static是可以作用域局部变量的，但是在Java中切记：**static是不允许用来修饰局部变量。**

3. static代码块  
tatic块可以置于类中的任何地方，类中可以有多个static块。在类初次被加载的时候，会按照static块的顺序来执行每个static块，并且只会执行一次。  
static块可以用来优化程序性能，是因为它的特性:只会在类加载的时候执行一次。所以可以将一些只需要进行一次的初始化操作都放在static代码块中进行。

### 练习1

```
package test;

public class Demo1 {
    Person person = new Person("demo1");
    static{
        System.out.println("demo1 static");
    }
     
    public Demo1() {
        System.out.println("demo1 constructor");
    }
     
    public static void main(String[] args) {
        new MyClass();
    }
}
 
class Person{
    static{
        System.out.println("person static");
    }
    public Person(String str) {
        System.out.println("person "+str);
    }
}
 
 
class MyClass extends Demo1 {
    Person person = new Person("MyClass");
    static{
        System.out.println("myclass static");
    }
     
    public MyClass() {
        System.out.println("myclass constructor");
    }
}

```  
这段代码的输出会是什么？  
- 首先找到程序入口main函数，它在Demo1类，因此首先加载Demo1类，其中有static代码块，于是第一个输出的就是"demo1 static"。
- 然后执行主函数里的MyClass()，就会去加载MyClass类，发现它继承于Demo1，应当先加载父类，不过刚刚已经首先加载了Demo1类，于是就直接加载MyClass类。  
- 因为MyClass类里也有static代码块，于是接着输出了"myclass static"。
- 然后就执行构造器生成Person对象，但是在生成对象的时候，必须先初始化父类的成员变量，因此会执行Test中的Person person = new Person()。
- 但是Person类还没有加载，于是加载Person类，执行了其中的static代码段，输出"person static"
- 然后执行new Person("demo1")，输出"person demo1"
- 然后执行父类自己的构造器Demo1()，输出"demo1 constructor"
- 然后执行子类中的new Person("MyClass")，输出"person MyClass"
- 最后执行子类自己的构造器MyClass(),输出"myclass constructor"

![image.png](https://i.loli.net/2020/11/16/SPcvBrJUCs5f2K3.png)  

### 练习2

```
package test;

public class Demo1 {
    
    static{
        System.out.println("test static 1");
    }
    public static void main(String[] args) {
         
    }
     
    static{
        System.out.println("test static 2");
    }
}
```
虽然主函数里并没有输出语句，但是加载Demo1类时会先执行static代码块，可以出现类中的任何地方（除了方法内部），并且执行是按照static块的顺序执行的。
  
![image.png](https://i.loli.net/2020/11/16/kjiuCLrFQ9qDPRE.png)  

### 小结
积累经验，搞清原理。不过有时候要抓紧主线，毕竟各种知识太庞杂了，不可能全都仔细研究学习，所以有的知识点只需要简单了解即可，先完成主线任务，不然就是浪费时间。

### 参考
http://www.cnblogs.com/dolphin0520/p/3799052.html  
https://blog.csdn.net/weixin_44545800/article/details/109248633