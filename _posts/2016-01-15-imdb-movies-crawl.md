---
layout: post
title: Spider for IMDB
categories:
- Programming
tags:
- spider
---

IMDB拥有着全球最大的电影数据库，最近我也在研究如何获取到这些宝贵的电影数据资源

根据时间来划分，可以将所有的电影分为：已上映和未上映。下面我将分这两种情况，分别去寻找方案

### 未上映电影数据

在IMDB官网，可以找到一些爬虫入口页面：[http://www.imdb.com/movies-coming-soon/2016-01](http://www.imdb.com/movies-coming-soon/2016-01)，[http://www.imdb.com/movies-coming-soon/2016-02](http://www.imdb.com/movies-coming-soon/2016-02)等等，通过分析url很容易发现：

url中的'movies-coming-soon'表示该页面的电影即将上映至影院，后面的日期为2016-xx，代表着该页面的电影预计2016年xx月上映

这样的话，我们就可以找到2016一整年的预映电影的入口页面

随意进入任何一个入口页面，里面陈列着该日期下所有预映电影的简短信息，包括指向某具体电影详情页面的link，随意点击某部电影，进入并得到link，比如：[http://www.imdb.com/title/tt3460252/?ref_=cs_ov_tt](http://www.imdb.com/title/tt3460252/?ref_=cs_ov_tt)

其中?ref_=cs_ov_tt是多余的，可以省略，于是link精简为：[http://www.imdb.com/title/tt3460252](http://www.imdb.com/title/tt3460252)，‘?ref_=cs_ov_tt’只是代表某些跳转信息（我猜的）

进入到电影详情页面，我们就可以抓取一切我们想要的数据了，原理很简单：

模拟http请求，然后解析HTML DOM树，主要用到了Python的request库和beautifulSoup4库，我还用到了阻塞队列Queue和多线程，解析电影详情页面的线程代码如下：

```python
def detail_thread():
    while True:
        try:
            m_url = movie_url.get()
            print 'detail_thread fetch: ', m_url
            r = requests.get(m_url)
            soup = BeautifulSoup(r.content)
            nMovie = newMovie()

            # id
            nMovie.id = m_url[m_url.index('/tt')+3:]
            # year
            nMovie.year = year

            # title
            nMovie.title = soup.find('h1', {'class': 'header'}).find(attrs={'class': 'itemprop', 'itemprop': 'name'}).text

            title_details = soup.find('div', {'id': 'titleDetails'})
            txt_block = title_details.findAll('div', {'class': 'txt-block'})
            for t in txt_block:
                try:
                    # countries
                    if t.find('h4').text == 'Country:':
                        c_links = t.findAll('a')
                        if len(c_links) > 0:
                            country = c_links[0].text
                            for c in c_links[1:]:
                                country = country + ':' + c.text
                            nMovie.countries = country
                    # languages
                    if t.find('h4').text == 'Language:':
                        l_links = t.findAll('a')
                        if len(l_links) > 0:
                            lan = l_links[0].text
                            for l in l_links[1:]:
                                lan = lan + ':' + l.text
                            nMovie.languages = lan
                    # production_companies
                    if t.find('h4').text == 'Production Co:':
                        p_c_links = t.findAll('a')
                        if len(p_c_links) > 0:
                            p_companies = p_c_links[0].text
                            for p_c in p_c_links[1:]:
                                p_companies = p_companies + ':' + p_c.text
                            nMovie.production_companies = p_companies
                except:
                    continue
                    #print 'detail_thread: txt_block find no h4 tag'

            # keywords
            keywords = soup.findAll(attrs={'itemprop': 'keywords', 'class': 'itemprop'})
            #print m_url, ' keywords:', len(keywords)
            if len(keywords) > 0:
                key_words = keywords[0].text
                for k in keywords[1:]:
                    key_words = key_words + ':' + k.text
                nMovie.keywords = key_words

            # genres
            genres_div = soup.find(attrs={'class': 'see-more inline canwrap', 'itemprop': 'genre'})
            if genres_div != None:
                g_links = genres_div.findAll('a')
                #print m_url, ' genres:', len(g_links)
                if len(g_links) > 0:
                    genres = g_links[0].text
                    for g in g_links[1:]:
                        genres = genres + ':' + g.text
                    #print genres
                    nMovie.genres = genres

            # cast
            cast_td = soup.findAll(attrs={'itemprop': 'actor'})
            #print m_url, ' cast:', len(cast_td)
            if len(cast_td) > 0:
                casts = cast_td[0].find(attrs={'class': 'itemprop', 'itemprop': 'name'}).text
                for c in cast_td[1:]:
                    casts = casts + ':' + c.find(attrs={'class': 'itemprop', 'itemprop': 'name'}).text
                #print casts
                nMovie.cast = casts

            # ??
            # editor
            # producer

            # writer
            w_div = soup.find(attrs={'class': 'txt-block', 'itemprop': 'creator'})
            if w_div != None:
                w_links = w_div.findAll('a')
                #print m_url, ' writer:', len(w_links)
                if len(w_links) > 0:
                    writer = w_links[0].text
                    for w in w_links[1:]:
                        writer = writer + ':' + w.text
                    nMovie.writer = writer
                #print nMovie.writer

            # director
            d_div = soup.find(attrs={'class': 'txt-block', 'itemprop': 'director'})
            if d_div != None:
                d_links = d_div.findAll('a')
                #print m_url, ' director:', len(d_links)
                if len(d_links) > 0:
                    director = d_links[0].text
                    for d in d_links[1:]:
                        director = director + ':' + d.text
                    nMovie.director = director

            entities.put(nMovie)
            movie_url.task_done()
        except Exception as e:
            print 'detail_thread: An {} exception occured'.format(e)
```

当然，在运行detail_thread这个线程之前，肯定要开启另外一个线程，作用是将所有电影详情页面的url放入movie_url这个Queue中，否则，detail_thread永远阻塞至死。所以，总的来设计，我运用了多线程+多队列搭建起了‘多级生产者和消费者’模型：

![](http://7xl2fd.com1.z0.glb.clouddn.com/imdb_new_movie_crawl_model.png)

index_thread生产所有入口页面，放入index_url阻塞队列；movie_thread从index_url队列取入口页面，生产电影详情页面url，放入movie_url；detail_thread从movie_url获取link，解析生成nMovie的实体，实体中保存着有关电影的详细信息，放入entitie队列；最后交由db_thread做入库持久化操作

运行时，可以视实际运行快慢，给其中的某些线程增加数量，增快其运行速度

总的代码，可以查看我的[https://github.com/su-kaiyao/mrp/blob/master/dataSet/newMovieCrawl.py](https://github.com/su-kaiyao/mrp/blob/master/dataSet/newMovieCrawl.py)

我自己运行了一把，抓了141条新电影，查阅了所有入口页面（2016-01 ~ 2016-12），数了数是145个，少了4个，是因为那4个电影的详细信息太少了，被脚本过滤了

### 已上映电影数据

移步：[https://7color94.github.io/blog/2016/01/imdb-db-structure/](IMDB数据库结构)