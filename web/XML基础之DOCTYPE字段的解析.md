# XML基础之DOCTYPE字段的解析

### 概述

>1. DTD 
>2. DOCTYPE

### 1. DTD

>DTD — 文档类型定义(Document type Definition)
>
>由于XML可以自定义标签,那么自然各人编写的标签不一样,这样同步数据便成了问题,因为其它人不知道某个标签应该怎么用,表示什么意思.DTD就是为了解决此问题的! 
>
>DTD是一种保证XML文档格式正确的有效方法，可以比较XML文档和DTD文件来看文档是否符合规范，元素和标签使用是否正确。一个DTD文档包含：元素的定义规则，元素间关系的定义规则，元素可使用的属性，可使用的实体或符号规则。
>
>DTD分类
>
>1. 内部DTD：内部DTD包含在XML文档中,
>2. 外部DTD：则通过URL引用.一个DTD文件是以.dtd结尾的文本文件 
>
>在XML中引入DTD DOCTYPE 文档类型声明 内部DTD,可以将standalone设置成yes. 
>
>```
><?xml version="1.0" standalone="yes"?>    
><!DOCTYPE root [     
><!ELEMENT root EMPTY>    
>]>   
>```
>
>外部DTD,需要将standalone设成no 
>
>```
><?xml version="1.0" standalone="no"?>    
> <!DOCTYPE root SYSTEM "http://www.test.org/test.dtd">   
>
>```
>
>
>
>

### DOCTYPE

>DTD声明始终以**!DOCTYPE**开头,空一格后跟着文档根元素的名称,如果是内部DTD,则再空一格出现[],在中括号中是文档类型定义的内容. 而对于外部DTD,则又分为私有DTD与公共DTD,私有DTD使用SYSTEM表示,接着是外部DTD的URL. 而公共DTD则使用PUBLIC,接着是DTD公共名称,接着是DTD的URL.下面是一些示例 
>
>公共DTD,DTD名称格式为"注册//组织//类型 标签//语言","注册"指示组织是否由国际标准化组织(ISO)注册,+表示是,-表示不是."组织"即组织名称,如:W3C; "类型"一般是DTD,"标签"是指定公开文本描述，即对所引用的公开文本的唯一描述性名称,后面可附带版本号。最后"语言"是DTD语言的ISO 639语言标识符,如:EN表示英文,ZH表示中文,在下面的地址有完整的ISO 639语言标识符列表[url]http://ftp.ics.uci.edu/pub/ietf/http/related/iso639.txt [/url] 
>
>```
><!DOCTYPE root SYSTEM "http://www.test.org/test.dtd">  
>```
>
>下面是XHTML 1.0 Transitional的DTD.以!DOCTYPE开始,html是文档根元素名称,PUBLIC表示是公共DTD,后面是DTD名称,以-开头表示是非ISO组织 组织名称是W3C,EN表示DTD语言是英语,最后是DTD的URL 
>
>```
><!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN"      
>    "http://www.w3.org/TR/xhtml/DTD/xhtml1-transitional.dtd">  
>```
>
>

注意:虽然DTD的文件URL可以使用相对URL也可以使用绝对URL,但推荐标准是使用绝对URL.另一方面,对于公共DTD,如果解释器能够识别其名称,则不去查看URL上的DTD文件 