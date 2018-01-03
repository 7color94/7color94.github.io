---
layout: post
title: IMDB数据库结构
categories:
- Programming
---

想要获取IMDB已上映电影的相关信息数据，我立马想到的两点就是：Restful API和爬虫，但就在我Google方案的过程中，我意外收获到了其他的东西：

其中一个，是使用Python语言开发的

[http://imdbpy.sourceforge.net/index.html](http://imdbpy.sourceforge.net/index.html)

，它的作用就是帮助开发者取回和管理IMDb的电影数据库

另外一个就是，IMDB网站其实提供了所有电影的数据库镜像：

[http://www.imdb.com/interfaces](http://www.imdb.com/interfaces)

顺着IMDbPy的文档，会发现一个非常实用的脚本：

[http://imdbpy.sourceforge.net/docs/README.sqldb.txt](http://imdbpy.sourceforge.net/docs/README.sqldb.txt)

，它的作用就是将镜像文件转换为本地的数据库

第一次使用，就顺着文档

[http://imdbpy.sourceforge.net/docs/README.sqldb.txt](http://imdbpy.sourceforge.net/docs/README.sqldb.txt)

一步一步来，运行完脚本后，本地数据库（我用的Mysql）会得到一些IMDB的表和数据

PS：下载所需的镜像文件，并运行imdbpy2sql.py脚本，我全部写成了自动化程序：

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

import subprocess
import os
import imdb

location = './dbfiles'
imdb_script = './code/bin/imdbpy2sql.py'
base_download_url = 'ftp://ftp.fu-berlin.de/pub/misc/movies/database/'
to_download_files = [
    'movie-links.list.gz', 'keywords.list.gz', 'directors.list.gz',
    'editors.list.gz', 'genres.list.gz', 'language.list.gz',
    'movies.list.gz', 'producers.list.gz', 'production-companies.list.gz', 'ratings.list.gz',
    'writers.list.gz', 'countries.list.gz', 'complete-cast.list.gz']

mysql_ip = 'localhost'
mysql_user = 'root'
mysql_passwd = '1227401054'
mysql_db = 'mrp'

def download_db_files():
    for file in to_download_files:
        url = base_download_url + file
        print 'Downloading ', url
        args = ['wget', '-P', location, url]
        t_pro = subprocess.Popen(args)
        # block model too slow
        #t_pro.wait()


def trans_db_to_local():
    while True:
        allDone = True
        for file in to_download_files:
            if not os.path.isfile(location + '/' + file):
                #print 'need file: ', location+file
                allDone = False
                break
        if allDone == True:
            break

    print 'Running imdbpy2sql.py begin'
    # mysql://user:password@host/database
    mysql_list = ['mysql://', mysql_user, ':', mysql_passwd, '@', mysql_ip, '/', mysql_db]
    subprocess.call(imdb_script + ' -d ' + location + ' -u ' + ''.join(mysql_list) + ' --mysql-force-myisam', shell=True)
    print 'Running imdbpy2sql.py. over'


def run():
    download_db_files()
    trans_db_to_local()

if __name__ == '__main__':
    run()

```

下面将分析IMDB的数据库结构，从中我们可以看出IMDB数据库设计的很好，数据高度结构化，数据表无冗余（PS：说来惭愧，平时我做开发，很喜欢一张表，这样写sql语句很方便，但是缺点真的很多），

### title

首先，最想找的就是电影名称了吧，它存储title表里的title字段中，除此之外，title表还有些重要的信息：id和kind_id，production_year，id代表该电影的全局唯一标识，production_year表示电影上映的年份，kind_id暂时不知道，那么可以找到kind_type表，

kind_type表的信息如下：

![](http://7xl2fd.com1.z0.glb.clouddn.com/imdb_db_kind_type.png)

我需要的是电影信息，那么就对应着kind_type为1的数据

以id为2459950的电影为例子吧，在title中的数据如下：

![](http://7xl2fd.com1.z0.glb.clouddn.com/imdb_title_table_ex.png)

从图中可以看到select出来的结果为：id为249950的电影名为After Words，上映时间为2015年

在imdb网站搜索下该电影，链接为：[http://www.imdb.com/title/tt2226630/](http://www.imdb.com/title/tt2226630/)

### movie_info

之后，我需要电影相关的info，比如：countries, languages, genres, votes, rating

首先涉及到的表式：movie_info，通过‘SELECT * FROM mrp.movie_info where movie_id=2459950;’，得到上面‘After Words’电影的相关信息如下：

![](http://7xl2fd.com1.z0.glb.clouddn.com/imdb_movie_info_ex.png)

其中info_type_id不是很明白，找到info_type表，里面解释着各个id代表的含义：

- id为3代表genres
- id为8代表countries
- id为4代表languages

结合截图select的结果，得知'After Words'的countries为USA，genres为Drama，languages为English

### movie_info_id

还有一些相关的info在movie_info_idx中，通过‘SELECT * FROM mrp.movie_info_idx where movie_id=2459950;’，得到如下结果：

![](http://7xl2fd.com1.z0.glb.clouddn.com/imdb_movie_info_idx_ex.png)

相应的：

- info_type_id为99代表votes distribution（这个数据值我暂时没搞懂）
- info_type_id为100代表votes
- info_type_id为101代表rating

### movie_keywords

需要查找电影的keywords信息，就需要用到movie_keywords表了。‘SELECT * FROM mrp.movie_keyword where movie_id=2459950;’：

![](http://7xl2fd.com1.z0.glb.clouddn.com/movie_keywords_ex.png)

这样就可以找到电影所有的keywords_id，然后使用keywords_id去keywords表中寻找具体的值，即可：

比如‘SELECT * FROM mrp.keyword where id=37348’得到的具体值为：costa-rica，这个和imdb官网是符合的，是正确的！

### cast_info

下面我们来找一下cast_info：cast, editor, writer, director, producer等，这些信息当然是在cast_info表中了，‘SELECT * FROM mrp.cast_info where movie_id=2459950;’:

![](http://7xl2fd.com1.z0.glb.clouddn.com/cast_info_ex.png)

里面同样有个role_id，查阅role_type表：

- role_id为1和2，分别代表actor和actress
- role_id为3，代表producer
- role_id为4，代表writer
- role_id为8，代表director
- rile_id为9，代表editor

那么如何找到具体的人名呢？通过person_id和name表，不多说，你可以试一下，再对照官网，可以发现信息是完全吻合的

### movie_companies

下面我们来找一下电影的production_companies，涉及到表为movie_companies，‘SELECT * FROM mrp.movie_companies where movie_id=2459950;’，之后会得到company_id

拿着company_id去company_name里面即可找到具体公司名

这里不再累述

### 额外信息

还有一些额外的信息我需要知道：有关评分的信息，包括1~10评分的分布，女性评分占多少，男性多少，年龄段评分多少等等

如[http://www.imdb.com/title/tt2226630/ratings?ref_=tt_ov_rt](http://www.imdb.com/title/tt2226630/ratings?ref_=tt_ov_rt)所示

方法是通过Imdbpy这个开源库里面的一个核心的函数：update，来完成（感谢东南大学杨远溢同学），他在这方面已经研究出方案了，我只需要站在他的基础上，编写程序即可

实例程序如下：（搜索"A Father's Journey"电影，并获取评分信息）

```python
import imdb

def init_db():
    title = "A Father's Journey"
    ia = imdb.IMDb()
    s_result = ia.search_movie(title)
    '''
    for item in s_result:
        print item['title']
    '''
    the_unt = s_result[0]
    ia.update(the_unt, info=('vote details'))
    if the_unt.has_key('rating'):
        print the_unt['rating']
    if the_unt.has_key('number of votes'):
        number_votes = the_unt['number of votes']
        for r in xrange(1, 11):
            print 'R'+str(r), number_votes[r]
    if the_unt.has_key('demographic'):
        infos = [
            'males', 'females',
            'aged under 18', 'males under 18', 'females under 18',
            'aged 18-29', 'males aged 18-29', 'females aged 18-29',
            'aged 30-44', 'males aged 30-44', 'females aged 30-44',
            'aged 45+', 'males aged 45+', 'females aged 45+',
            'imdb staff', 'top 1000 voters',
            'us users','non-us users'
        ]
        demo_value = the_unt['demographic']
        print 'get'
        for in_fo in infos:
            if demo_value.get(in_fo) != None:
                print in_fo, demo_value.get(in_fo)[0], demo_value.get(in_fo)[1]
```

### 总结

IMDB提供数据的方式确实很独特，通过镜像的方式。通过分析所有的表结构，不难发现imdb数据库设计之好

好了，截止目前，imdb未上映和已上映电影数据就都可以拿到了