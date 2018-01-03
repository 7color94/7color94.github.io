---
layout: post
title: 豆瓣读书前端开发
categories:
- Programming
tags:
- douban
---

上一篇讲了豆瓣读书API使用，我们已经可以从豆瓣那边取出关于图书的信息了，下面，我们开始编写前台代码，将这些信息显示在浏览器中

我采用Bootstrap前端开发框架，准备编写两个页面:

- 输入书籍名，开启查询页面：供用户输入书籍名，点击‘Search’按钮，向后台发送请求
- 书籍详细信息显示页面：从后台接受到数据，然后显示在页面中

先来看下，我做的效果图吧，一睹为快

![Search页面](https://github.com/su-kaiyao/record/raw/master/others/imgs/search_page.png)

![book_info](https://github.com/su-kaiyao/record/raw/master/others/imgs/book_info.png)

还是，先看下代码架构吧

![douban_front](https://github.com/su-kaiyao/record/raw/master/others/imgs/douban_front.png)

前段代码的编写就不说了，说下我是如何编写和后端交互那块的

### 从search页面，将书籍名传给书籍详细显示页面

这里，我定义了一个表单，供用户输入书籍名，然后有一个蓝色的**Search**按钮，点击后，将书籍名传给书籍详细显示页面，看一下表单的代码

```html
<form class="text-center" action="bookDetail.jsp" method="post">
  <div class="form-inline">
    <div class="form-group">
      <div class="input-group">
        <div class="input-group-addon"><span class="glyphicon glyphicon-book" aria-hidden="true"></span>
        </div>
      	<input type="text" class="form-control" name="bookname" placeholder="输入您想查询的书籍名">	
      </div>
    </div>
    <button type="submit" class="btn btn-primary">Search</button>
  </div>
</form>
```

通过定义form表单的action属性，点击Search按钮后，将input中数据传给**bookDetail.jsp**页面

### bookDetail.jsp页面接受到bookName后，开始和后台交互

通过jsp内置的request对象，就可以取出前面表单中输入的书籍名：String bookName = request.getParameter("bookname");

然后，调用上一讲编写的BookDAO类中的两个方法，最终得到关于书籍详细信息的Book类实例book

```jsp
<%
request.setCharacterEncoding("utf-8");
String bookName = request.getParameter("bookname");
BookDAO bookDao = new BookDAO();
Book book = bookDao.getBookById(bookDao.getBIDByName(bookName));
if (book == null) {
    out.println("Error");
} else {

}
%>
```

有了这个book实例，想显示什么信息，就能显示什么信息啦，调用book实例的getter方法，就可以取得相应数据了，在通过jsp表达式，显示在浏览器中就行了

好了，做到这里，看到上面的两张截图，我认为，前后端就打通了，小系统基本完成了

下一篇，我想讲一下它的部署问题，想把它部署到SAE上去