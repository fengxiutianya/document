# Java 中的instanceof和getClass()的作用

### 主要内容

>1. instaceof
>2. getClass()

### 首先预定义一些类，方便后面讲解

>父类A:
>
>```
>class A{}
>```
>
>子类B
>
>```
>class B extends A{}
>```
>
>构造对象
>
>Object o1 = new A();
>
>Object o2 = new B();

### 1. instaceof

>用法：
>
>​	object instaceof class 
>
>注意的是：instaceof关键字前面必须是对象，后面可以是class和interface
>
>演示一：
>
>```
>1、o1 instanceof A => true  
>2、o1 instanceof B => false  
>3、o2 instanceof A => true  // <================ HERE  
>4、o2 instanceof B => true 
>```
>
>从上面可以看出，java中的instaceof运算符是用来在运行时指出对象是否是特定类的一个实例。instaceof通过返回一个布尔值来指出，如果为true就表明这个对象是某个类或者其子类的一个实例。
>
>需要注意的一点是：
>
>String 是

### 2. getClass()

>这个方法在java中每个对象都会拥有，因为这个方法是Object的一个方法。
>
>演示二：
>
>```
>1、o1.getClass().equals(A.class) => true 
>2、o1.getClass().equals(B.class) => false 
>3、o2.getClass().equals(A.class) => false // <=======HERE 
>4、o2.getClass().equals(B.class) => true  
>```
>
>分析：
>
>getClass 方法在JDK1.8中的定义如下
>
>```
>/**
>*    Returns the runtime class of this Object
>*/
>public final native Class<?>  getClass();
>```
>
>功能：返回运行时期对象的类。
>
>如果你想确定某个对象是特定类的实例而不是特定类子类的实例,则使用getClass方法更实用一些。
>
>







