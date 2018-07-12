本篇博客主要讲解流的基本操作和如何构建流

### 主要内容如下

>1. 筛选和切片
>2. 映射
>3. 查找和匹配
>4. 规约
>5. 数值流
>6. 构建流

### 筛选和切片

>筛选包含俩类
>
>1. 谓词筛选 ：`filter(Predicate< ? super T> predicate)`，所谓谓词筛选，就是通过一个返回boolean的函数，来筛选需要的元素
>
>   筛选出偶数的案例：
>
>   ```java
>   Arrays.asList(1,2,3,4,5,6,7,8).stream().filter(i -> i %2 ==0)
>        .collect(toList());  //这个地点是输出List，后面会说
>   ```
>
>2. 筛选各异元素: `distinct()`
>
>   使用案例如下
>
>   ```java
>   List< Integer> numbers = Arrays.asList(1,2,3,4,3,5,3);
>   numbers.stream().distinct()
>   			.forEach(System.out::println);
>   //输出结果 1,2,3,4,5
>   ```
>
>3. 截断流：`limit(int n)`,返回一个不超过给定长度的流
>
>   筛选出数值大于3，的头3个数字
>
>   ```java
>   List< Dish> dishes = Arrays.asList(1,2,3,4,5,6,7,8).
>   							stream()
>                               .filter(d -> d > 3)
>   							.limit(3)
>                               .collect(toList());
>                      
>   ```
>
>4. 跳过元素:`skip(long n)`跳过前n个元素，如果不足n则返回空流
>
>   跳过前3个的数字的案例
>
>   ```
>   List< Dish> dishes = Arrays.asList(1,2,3,4,5,6,7,8).
>   							stream()
>                               .filter(d -> d > 3)
>   							.skip(3)
>                               .collect(toList());
>   ```
>
>   ​
>
>

### 映射

>1. 映射:`map(Function< ? super T,? extends R> mapper)`对流中每一个元素应用一个函数，转换成自己希望的类型
>
>   把数字映射成字符串
>
>   ```
>   List< String> list = Arrays.asList(1,2,3,4,5,6)
>   							.stream()
>   							.map(Ingeger::toString)
>   							.collect(toList());
>   ```
>
>2. 流的扁平化：`flatMap(Function< ? super T,? extends Stream< ? extends R>> mapper)`把产生的多个流映射成一个流
>
>   ```java
>   String[] arrayOfWords = {"Goodbye", "World"}; Stream< String> streamOfwords = Arrays.stream(arrayOfWords);
>   words.stream()                  //这个地点生成的是 stream<String[]>
>        .map(word -> word.split(""))  // 这个地点生成的是 stream<String[]>
>   	 .distinct()
>   	 .collect(toList());   //最后生成的是 List<String[]>
>   //而我们想要的是List<String>，因此上面的不能满足
>
>   //下面显示的是flatMap来做的
>
>   List< String> uniqueCharacters =
>       words.stream()
>            .map(w -> w.split(""))   //这个地点生成的是 stream<String[]>
>            .flatMap(Arrays::stream) //这个地点生成的是 stream<String>
>            .distinct()
>            .collect(Collectors.toList()); //最后生成的是 List<String>
>   ```
>
>   ​
>
>   ​

### 查找和匹配

>1. 检查谓词是否至少匹配一个元素：`anyMatch(Predicate< ? super T> predicate)`，
>
>2. 检查谓词是否匹配所有元素:`allMatch(Predicate< ? super T> predicate)`，
>
>3. 检查谓词没有一个匹配的元素:`allMatch(Predicate< ? super T> predicate)`
>
>   使用案例
>
>   ```
>   boolean t1 = Arrays.asList(1,2,3,4,5,6)   //结果是true
>   							.stream()
>   							.allMatch(d -> d < 1000);
>   boolean t2 = Arrays.asList(1,2,3,4,5,6)   //结果是true
>   							.stream()
>   							.anyMatch(d -> d < 1000);	
>    boolean t3 = Arrays.asList(1,2,3,4,5,6)   //结果是false
>   							.stream()
>   							.noneMatch(d -> d < 1000);	                           
>   ```
>
>4. 查找任何元素：`findAny()`，在后台执行的时候，会进行优化使其直走一遍，如果是并行运行的，无论哪一个线程找到需要的结果，则暂停执行。
>
>5. 查找第一个元素:`findFirst()`，当流时顺序流的时候，这个会找到第一个元素
>
>   ```java
>     
>    List< Integer> someNumbers = Arrays.asList(1, 2, 3, 4, 5);
>    Optional< Integer> firstSquareDivisibleByThree =
>           someNumbers.stream()
>                      .map(x -> x * x)
>                      .filter(x -> x % 3 == 0)
>                      .findAny(); //  结果不确定
>    List< Integer> someNumbers = Arrays.asList(1, 2, 3, 4, 5);
>    Optional< Integer> firstSquareDivisibleByThree =
>           someNumbers.stream()
>                      .map(x -> x * x)
>                      .filter(x -> x % 3 == 0)
>                      .findFirst(); // 9
>                     
>   ```
>
>   ​

### 规约

>reduce 操作：`reduce(U identity, BiFunction< U,? super T,U> accumulator, BinaryOperator< U> combiner)`
>
>reduce接受两个参数:
>
>-  一个初始值，这里是0;
>- 一个BinaryOperator< T>来将两个元素结合起来产生一个新
>
>```
>int total = Arrays.asList(1, 2, 3, 4, 5).stream()
>			.reduce(0,(t1,t2)->t1+t2);
>```

