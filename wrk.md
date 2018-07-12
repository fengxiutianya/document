### wrk 使用

### 概述

>1. ​





> 暂存内容
>
> ```
> wrk -t12 -c100 -d30s http://www.baidu.com  
>
> 输出结果
> Running 30s test @ http://www.baidu.com  
>   12 threads and 100 connections  
>   Thread Stats   Avg      Stdev     Max   +/- Stdev  
>     Latency   538.64ms  368.66ms   1.99s    77.33%  
>     Req/Sec    15.62     10.28    80.00     75.35%  
>   5073 requests in 30.09s, 75.28MB read  
>   Socket errors: connect 0, read 5, write 0, timeout 64  
> Requests/sec:    168.59  
> Transfer/sec:      2.50MB  
> ```
>
> 
>
> 
>
> 先解释一下输出: 
> 12 threads and 100 connections 
> 这个能看懂英文的都知道啥意思: 用12个线程模拟100个连接. 
> 对应的参数 -t 和 -c 可以控制这两个参数. 
>
> 一般线程数不宜过多. 核数的2到4倍足够了. 多了反而因为线程切换过多造成效率降低. 因为 wrk 不是使用每个连接一个线程的模型, 而是通过异步网络 io 提升并发量. 所以网络通信不会阻塞线程执行. 这也是 wrk 可以用很少的线程模拟大量网路连接的原因. 而现在很多性能工具并没有采用这种方式, 而是采用提高线程数来实现高并发. 所以并发量一旦设的很高, 测试机自身压力就很大. 测试效果反而下降. 
>
> Latency: 可以理解为响应时间, 有平均值, 标准偏差, 最大值, 正负一个标准差占比. 
>
> Req/Sec: 每个线程每秒钟的完成的请求数, 同样有平均值, 标准偏差, 最大值, 正负一个标准差占比. 
>
> 一般我们来说我们主要关注平均值和最大值. 标准差如果太大说明样本本身离散程度比较高. 有可能系统性能波动很大. 
>
> ```
>   5073 requests in 30.09s, 75.28MB read  
>   Socket errors: connect 0, read 5, write 0, timeout 64  
> Requests/sec:    168.59  
> Transfer/sec:      2.50MB  
> ```
>
> 30秒钟总共完成请求数和读取数据量. 
> 然后是错误统计, 上面的统计可以看到, 5个读错误, 64个超时. 
> 然后是所以线程总共平均每秒钟完成168个请求. 每秒钟读取2.5兆数据量. 
>
> 可以看到, 相对于专业性能测试工具. wrk 的统计信息是非常简单的. 但是这些信息基本上足够我们判断系统是否有问题了. 

