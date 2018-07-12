### 命令

>```
>command key flags exptime bytes [noreply] value
>commmand 命令
>key    存储的key值
>flags 可以包括键值对的整型参数，客户机使用它存储关于键值对的额外信息 
>exptime  在缓存中保存键值对的时间长度（以秒为单位，0 表示永远）
>bytes：在缓存中存储的字节数 
>noreply（可选）： 该参数告知服务器不需要返回数据
>value：存储的值（始终位于第二行）
>       （可直接理解为key-value结构中的value）
>       
>存储命令
>set  如果不存在则添加，存在则更新
>
>add 如果 add 的 key 已经存在，则不会更新数据(过期的 key 会更新)，   
>     之前的值将仍然保持相同，并且您将获得响应 
>replace
>    命令用于替换已存在的 key(键) 的 value(数据值)。
>    如果 key 不存在，则替换失败，并且您将获得响应 NOT_STORED。
>append
>   命令用于向已存在 key(键) 的 value(数据值) 后面追加数据 。
>prepend
>   命令用于向已存在 key(键) 的 value(数据值) 前面追加数据 
>cas 
>```
>
>



