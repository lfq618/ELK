## ELK环境使用说明
- **目的**：方便大家在开发过程中收集查看日志  
- **存储策略**： 只会存储一天的日志，每天会将之前收集的日志清空。
- **日志源服务器**

> 目前只收集 **172.28.20.12/15/16** 服务器的日志， 若有开发环境基于其他服务器，需要收集日志的，请联系我进行配置。

### Kibana 服务环境
- 域名： http://172.28.50.56:5601
- 索引： 基于stdout收集的日志已创建索引：logstash-log-YYYY.MM.DD


### Kibana使用lucene查询语法
> kibana在ELK中用来查询展示数据    
> elasticsearch构建在Lucene只上，过滤器语法和Lucene相同

![image](http://oetghrmbm.bkt.clouddn.com/ELK%E7%95%8C%E9%9D%A2.png)

#### 全文搜索
在搜索栏输入login， 会返回所有字典值中包含login的文档
![image](http://oetghrmbm.bkt.clouddn.com/%E6%90%9C%E7%B4%A2.png)

使用双引号包起来作为一个短语搜索    
**"CPU iPhone"**

#### 字段
Kibana也可以按照页面左侧显示的字段进行搜索    
限定字段全文搜索：**field:value**  
精确搜索： 关键字加上双引号 **field:"value"**


字段本身是否存在   
```code
_exists_:http   //返回的结果中需要有http字段
_missting_:http  //不能包含http字段
```

#### 通配符
匹配单个字符： **?**  
匹配0到多个字符： __*__

#### 正则
es支持部分正则功能，性能比较差   
```code
name: /joh?n(ath[oa]n)/
```

#### 模糊搜索
**~**: 在一个单词后边加上**~** 启用模糊搜索， 可以搜索到一些拼写错误的单词     
还可以设定距离（整数）， 指定需要多少相似度   
**cromn~1** 会匹配到from和chrome   

![image](http://oetghrmbm.bkt.clouddn.com/%E6%A8%A1%E7%B3%8A%E6%90%9C%E7%B4%A2.png)

#### 近似搜索
在短语后面加上 **~**，可以搜到被隔开或顺序不同的单词    
**"where select"~5** 表示 select 和 where 中间可以隔着5个单词，可以搜到 select password from users where id=1

#### 范围搜索
数值/时间/IP/字符串 类型的字段可以对某一范围进行查询

```code
length:[100 TO 200]
sip:["172.24.20.110" TO "172.24.20.140"]
date:{"now-6h" TO "now"}
tag:{b TO e}       //搜索b到e中间的字符
count:[10 TO *]    //* 表示一端不限制范围
count:[1 TO 5}     //[ ] 表示端点数值包含在范围内，{ } 表示端点数值不包含在范围内，可以混合使用，此语句为1到5，包括1，不包括5

//可以简化成以下写法：
age:>10
age:<=10
age:(>=10 AND <20)
```

#### 优先级
使用 **^** 使一个词语比另一个搜索优先级更高，默认为1，可以为0~1之间的浮点数，来降低优先级   
例： quick^2 fox

#### 逻辑操作
**AND**    
**OR**  
**+** ： 搜索结果中必须包含此项    
**-** :  搜索结果不能含有此项

例：
```code
+apache -jakarta test aaa bbb：结果中必须存在apache，不能有jakarta，剩余部分尽量都匹配到
```

#### 分组
例：
```code
(jakarta OR apache) AND jakarta
```

#### 字段分组
例：
```code
title:(+return +"pink panther")
host:(baidu OR qq OR google) AND host:(com OR cn)
```

#### 转义特殊字符
**+ - = && || > < ! ( ) { } [ ] ^ " ~ * ? : \ /**   
以上字符当作值搜索的时候需要用\转义   
**\\(1\\+1\\)\\=2用来查询(1+1)=2**
