在刚开始学习使用Lambda表达式的时候，你就把Lambda表达式类比成匿名类，把你写的每一个Lambda表达式相对应的匿名内部类都写出来。然后你把其中方法里面的语句和参数列表提取出来，按照Lambda表达式的格式写出来，就是Lambda表达式。

###主要内容

>1. 初步使用及Lambda表达式的格式
>2. 函数式接口
>3. 类型检查以及类型推断
>4. 方法引用
>5. 复合Lambda表达式

### 初步使用

>相信很多人都用过下面这段代码创建一个线程
>
>```java
> new Thread(new Runnable() {
>            @Override
>            public void run() {
>                线程代码
>            }
> }); 
>```
>
>上面那段代码是使用匿名类创建的一个线程，其实Lambda和这个很相似，也很想匿名内部类，但是比这更简化。对于初学者我相信没有什么比学会使用更重要。虽然远离很重要，但是当你学会使用，慢慢的你自然就会在使用过程中知道原理。
>
>下面这段代码是对应上端代码的Lambda表达式
>
>```java
>new Thread(()->{
>  //线程代码
>})
>```
>
>是不是很简洁，确实使用Lambda表达式的一个好处就是可以让你的代码简洁。
>
>下面我们步入正题，Lambda表达式的格式：
>
>可以把Lambda表达式理解为简洁地表示可传递的匿名函数的一种方式:它没有名称，但它有参数列表、函数主体、返回类型，可能还有一个可以抛出的异常列表。这个定义够大的，让我们慢慢道来。
>
>- 匿名——我们说匿名，是因为它不像普通的方法那样有一个明确的名称:写得少而想得多!
>- 函数——我们说它是函数，是因为Lambda函数不像方法那样属于某个特定的类。但和方法一样，Lambda有参数列表、函数主体、返回类型，还可能有可以抛出的异常列表。
>- 传递——Lambda表达式可以作为参数传递给方法或存储在变量中。
>- 简洁——无需像匿名类那样写很多模板代码。
>
>#### Lambda的主要格式
>
>```java
>() -> {}
>```
>
>- 参数列表——这里它采用了Comparator中compare方法的参数，两个Apple。
>- 箭头——箭头->把参数列表与Lambda主体分隔开。
>- Lambda主体——比较两个Apple的重量。表达式就是Lambda的返回值了。
>
>下面都是Lambda表达式的例子：
>
>```		Java
>//第一个Lambda表达式具有一个String类型的参 数并返回一个int。
>//Lambda没有return语句， 因为已经隐含了return
>(String s) -> s.length()
>  
>  
>//第二个Lambda 表达式有一个 Apple类型的参数
>//并返回一 个boolean(苹 果的重量是否 超过150克)
>(Apple a) -> a.getWeight() > 150 
>  
>  
>// 第三个Lambda表达式具有两个int类型的参
>//数而没有返回值(void返回)。注意Lambda } 表达式可以包含多行语句，这里是两行 
>(intx,inty)->{
>    System.out.println("Result:");
>    System.out.println(x+y);
>}
>// 第四个Lambda表达式没有参数，但是返回一个int类型的整数
>() -> 42
>```
>
>不过有一点需要注意的是，如果是下面这个语句就是错误的
>
>```java
>()->{"Iron Man"}
>
>“Iron Man”是一个表达式，不是一个语句。要使此Lambda有效，你可以去除花括号 和分号，
>如下所示:
>(String s) -> "Iron Man"。这句话正确的原因是因为，Lambda表达式默认会有一个return语句。
>或者如果你喜欢，可以使用显式返回语句，如下所示:
>(String s)->{return "IronMan";}。
>```
>
>现在既然你已经学会怎么使用Lambda表达式，下面就是该说在哪使用Lambda表达式	 

### 函数式接口

>一言以蔽之，函数式接口就是只定义一个抽象方法的接口。你已经知道了Java API中的一些
>
>函数式接口，如Comparator和Runnable。
>
>```java
>java.util.Comparator
>public interface Comparator<T> {
>    int compare(T o1, T o2);
>}
>
>
>java.lang.Runnable
>public interface Runnable{
>    void run();
>}
>```
>
>与函数式 接口相对应的就是函数描述符，其定义是：函数式接口的抽象方法的签名基本上就是Lambda表达式的签名，我们将这种抽象方法叫作函数描述符。其实简单的来说，这个函数对应的Lambda的格式是什么。
>
>类如上面的Runnable接口对应的函数描述符是 **`()->{}`** ,Comparator对应的函数描述符是:**(T,T)->{return int；}**
>
>而函数式接口就是你使用Lambda的地点。

### 类型检查及类型推断

>Lambda的类型是从Lambda的上下文推断出来的。上下文中Lambda表达式需要 的类型称为 目标类型。
>
>下面这个例子可以很好的解释了Lambda表达式背后发生了什么。
>
>![类型检查](./img1.png)
>
>类型检查过程可以分解为如下表示：
>
>- 首先，你要找出filter方法的声明。
>- 第二，要求同时Predicate < Apple >(目标类型)对象的第二个正式参数
>- 第三，Predicate< Apple>是一个函数式接口，定义了一个叫做test的抽象方法。
>- 第四，test方法描述了一个函数描述符，他可以接收一个Apple，并返回一个boolean。
>- 最后，filter的任何实际参数都必须匹配这个要求。
>
>#### 类型推断
>
>>其实你只要用过java 7 以上的版本，你一定见过类型推断,下面这段代码，就是使用了类型推断：
>>
>>```java
>>List < Apple >  list = new ArrayList< >();// 使用了类型推断，
>>
>>List< Apple > list   = new ArrayList< Apple>(); //没有使用类型推断
>>```
>>
>>其实这就是类型推断，只不过Lambda表达式中的类型推断是来推断函数式接口中的参数类型。下面的代码就是个例子,上面一个没有使用类型推断，下面一个例子省去了参数的类型，使用了类型推断。
>>
>>```java
>>Comparator<Apple> c =  //没有类 型推断
>>(Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight()); 
>>
>>Comparator<Apple> c =  有类型推断
>>   (a1, a2) -> a1.getWeight().compareTo(a2.getWeight()); 
>>```
>
>#### 提示
>
>>有一点需要注意的就是使用局部变量，必须是final类型的变量，原因如下：
>>
>>1. 实例变量和局部变量背后的实现有一个关键不同。实例变量都存储在堆中，而局部变量则保存在栈上。如果Lambda可以直接访问局部变量，而且Lambda是在一个线程中使用的，则使用Lambda的线程，可能会在分配该变量的线程将这个变量收回之后，去访问该变量。因此，Java在访问自由局部变量时，实际上是在访问它的副本，而不是访问原始变量。如果局部变量仅仅赋值一次那就没有什么区别了——因此就有了这个限制。
>>2. 这一限制不鼓励你使用改变外部变量的典型命令式编程模式。

### 方法引用

>可以简单的理解成，我们把方法传递给需要的地点，然后JVM在编译的时帮我们自动转换成要使用的样子。
>
>方法引用主要由三类，
>
>1. 指向静态方法的方法引用(例如Integer的parseInt方法，写作Integer::parseInt)。
>2.  指向任意类型实例方法的方法引用(例如String的length方法，写作String::length)。
>3.  指向现有对象的实例方法的方法引用(假设你有一个局部变量expensiveTransaction用于存放Transaction类型的对象，它支持实例方法getValue，那么你就可以写expensive-Transaction::getValue)。
>
>第二种和第三种方法引用可能乍看起来有点儿晕。类似于String::length的第二种方法引用的思想就是你在引用一个对象的方法，而这个对象本身是Lambda的一个参数。例如，Lambda表达式(String s) -> s.toUppeCase()可以写作String::toUpperCase。但第三种方法引用指的是，你在Lambda中调用一个已经存在的外部对象中的方法。例如，Lambda表达式()->expensiveTransaction.getValue()可以写作expensiveTransaction::getValue。
>
>下面用一个图来解释上面三种引用
>
>![](./img2.png)			
>
>​		
>
>下面是一个使用的demo
>
>```java
>List<String> str = Arrays.asList("a","b","A","B");
>str.sort((s1, s2) -> s1.compareToIgnoreCase(s2));
>
>Lambda表达式的签名与Comparator的函数描述符兼容。
>利用前面所述的方法，这个例子可 以用方法引用改写成下面的样子:
>List<String> str = Arrays.asList("a","b","A","B");
>str.sort(String::compareToIgnoreCase);
>```
>
>#### 构造函数引用
>
>> 我们直接用例子来说明
>>
>> ```java
>> public interface Supplier<T>{  //Supplier 接口
>>   T get();
>> }
>> //默认构造函数
>> Supplier<Apple> c1 = Apple::new;
>> Apple a1 = c1.get();
>> // 上面的等价于:  
>> Supplier<Apple> c1 = () -> new Apple();
>> Apple a1 = c1.get();
>>
>>
>> // 带参数的构造函数
>>
>> Function<Integer, Apple> c2 = Apple::new;
>> Apple a2 = c2.apply(110);
>>
>> 这就等价于:指向Apple(Integer weight) 的构造函数引用
>> 调用该Function函数的apply方法，并给出要求的重量，将产
>> 生一个Apple用要求的重量创建一个Apple的Lambda表达式 
>>
>> Function<Integer, Apple> c2 = (weight) -> new Apple(weight); 
>> Apple a2 = c2.apply(110);
>>
>> ```
>
>
>​			

### 复合Lambda表达式

>#### 谓词复合
>
>>Predicate 接口有三个默认方法，（至于什么是默认方法看我后面的博客）：negate，and和or 对应我们使用的 非，与 ，或三个谓词。
>>
>>下面的例子就是使用这三个谓词
>>
>>```java
>>//假设Apple对象有getColor 和 getWeight方法
>>Predicate< Apple > redApple = (a)->"red".equals(a.getColor);
>>
>>//产生非红果的谓词
>> Predicate<Apple> notRedApple = redApple.negate();
>>
>>//使用and 谓词，来表示既是红苹果重量也大于150
>>Predicate<Apple> redAndHeavyApple =
>>    redApple.and(a -> a.getWeight() > 150);
>>//来生成，是红苹果 或者 重量大于 150的谓词
>>Predicate<Apple> redAppleOrGreen =
>>   redApple.or(a -> "green".equals(a.getColor()));
>>```
>
>### 函数复合
>
>>最后，你还可以把Function接口所代表的Lambda表达式复合起来。Function接口为此配了andThen和compose两个默认方法，它们都会返回Function的一个实例。
>>
>>下面是一个使用例子:
>>
>>```java
>>//addThen
>>Function<Integer, Integer> f = x -> x + 1;
>>Function<Integer, Integer> g = x -> x * 2;
>>Function<Integer, Integer> h = f.andThen(g);
>>int result = h.apply(1);   // 数学上会写作g(f(x))
>>
>>//compose 
>>Function<Integer, Integer> f = x -> x + 1;
>>Function<Integer, Integer> g = x -> x * 2;
>>Function<Integer, Integer> h = f.compose(g);
>>int result = h.apply(1);  //数学上会写作 f(g(x))	
>>```

 



