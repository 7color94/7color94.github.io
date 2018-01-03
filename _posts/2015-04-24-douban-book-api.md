---
layout: post
title: 豆瓣读书API
categories:
- Programming
tags:
- douban
---

最近自学了jsp，想做个demo验证一下学习效果

脑子里出来这么一个想法：基于web做一个图书查询系统，在查询页面输入你想查询的书籍名称，然后，显示该书籍的详细信息：定价，页数，作者，出版社，出版年等等

业务逻辑十分简单的系统，我把它叫做苏苏查书，所有的图书数据来源是豆瓣Book API

下面，让我们开始这个系统的开发

### 如何获取图书的数据

答案是：豆瓣Book API

基于豆瓣API，我可以获得全面的图书信息，之前开发微信公共平台时，使用过豆瓣Movie API，所以这里写起来还是很快的

根据豆瓣提供的api手册，我的思路是这样的：先根据用户输入的书籍名，调用api找到该书籍的豆瓣id，然后拿这个id去查询该书籍的详细信息

下面，开始编写代码

### 代码架构与编写

看一下后端业务逻辑的包和类结构

![豆瓣API后台包结构](https://github.com/su-kaiyao/record/raw/master/others/imgs/douban_package.png)

先编写一个Book.java，作为图书POJO类，里面的属性可以自行定义，你想需要关于图书的哪些信息，你就定义之；并为属性设置get和set方法

然后，编写BookDAO.java，作为与豆瓣api打交道的类，里面有两个基本方法：

- **public String getBIDByName(String bookName)**：根据书籍名，向豆瓣api请求，返回该书籍的豆瓣唯一id
- **public Book getBookById(String id)**：根据id，向另一个豆瓣api请求，获取关于书籍的详细信息（json格式），然后把我们需要的信息保存到Book类的实例中，并返回

下面，先来看第一个方法：

首先，豆瓣给出，根据书名找id的api url是：`https://api.douban.com/v2/book/search?q=`，后面加上书籍名，就会返回给我一个json格式的数据，里面包含有关于id的值

我采用Httpclient模拟http get请求；关于如何用httpclient模拟http get获得json对象，网上搜一下，一大堆

最终，获得到JSONObject对象json，这个json就是我待会要用的数据，关于id的键值对就在里面

### 解析json数据

因为要解析json数据，从而获得我自己需要的数据，我封了一个工具类JsonUtil.java，里面有两个静态函数，专门用来解析json数据：

- **public static String getBookId(JSONObject json)**：从json数据中，获得豆瓣图书id
- **public staic Book getBookInfo(JSONObject json)**：从json数据中，获得图书信息，将信息存在Book对象中，返回

这样，我的代码就很清晰了，BookDAO.java专门负责从豆瓣获取json数据，不负责解析工作，解析工作交给JsonUtil.java这个工具类，嘿嘿，符合单一职责的设计原则

简单说一下如何解析json数据，我借助的是第三方jar包，里面有两个很重要的类**JSONObject和JSONArray**，当时遇到一个难题，解析时候遇到了问题，先看下面这组json数据(数据不是豆瓣返回给我的，我瞎编的，只是为了说明问题)

```
{
"id": "123456",
"tags": [
  {"count": 1, "name": "test1"},
  {"count": 2, "name": "test2"}
],
"rating": {
  "min": "0",
  "max": "9"
}
}
```

就是这种数组（在方括号中，对应JSONArray）和对象（在花括号中，对应JSONObjcet）相互参杂的解析，那么如何获得"tags"这个key的数据呢？

依据<a href="http://www.json.org.cn/resource/json-in-java.htm" target="_blank">API</a>，JSONObject有一个方法叫**JSONArray getJSONArray(String key)**，该函数的意思是用一个key作为参数，获得该key的value，这个value是一个JSONArray；相应地，JSONArray有一个方法叫**JSONObject getJSONObject(String key)**，自行体会吧

这样，我们不管什么格式的json，肯定都可以解析了

这里，我就不贴出我的代码了，等这个系统弄完，会把源代码放到github上

### 最终结果

我们来看看，后台写的咋样了，测试结果如下：

这是在控制台打印出来的信息，搜了《暗时间》这本书的详细信息（ps：这是本好书啊）

![豆瓣Book API测试](https://github.com/su-kaiyao/record/raw/master/others/imgs/doubanbookapi.png)

足足运行了1m多

下面准备开始用编写前台代码了，并用jsp编写和后台交互的代码

---

推荐阅读：

- <a href="http://sukai.me/doubanapifront/" target="_blank">苏苏查书之前台开发篇</a>