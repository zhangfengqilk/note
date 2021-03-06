# 符合规范的url



# 描述

进公司没有多久遇到一个问题，定义的url会被大神吐槽说是很渣。之前从来没有注意这块，今天把我们团队的url规范分享给大家。

# 为什么需要URL规范化

1、网站URL和结构已经成为网站搜索引擎友好的最大基础性问题，网站URL 和结构问题，早发现早优化，越是往后放，最后就成了制约网站运营和产品开发的决定性因素。
2、无论是网站的可用性还是网站对搜索引擎的吸引力，清晰明了的浏览路径都是相当重要的，URL是统一资源定位，即每个网页的网址、路径。
3、浏览路径让网站的导航结构更清晰，可以更加平衡的分布网站权重。

# 反例（不规范的URL）

### URL中多余的字符

1、子域名的URL中包含"www": "http://www.shuchao.cnblogs.com/"

2、含有默认端口: "[http://www.cnblogs.com:80/shuchao/](http://www.cnblogs.com/shuchao/)"

3、松散的URL: "http://www.chapters.indigo.ca/books/amazon-sucks-donkey-balls/9780470170779-item.html"

4、多余默认文件名index.html,default.aspx等："http://www.cnblogs.com/shuchao/index.html"

5、文件路径中包含多余的"/":"http://www.cnblogs.com/shuchao//"，多余的点修饰串:"x/y/z/http://www.cnblogs.com/a/b/http://www.cnblogs.com/../page.html"

6、查询串中多余的 ? (空查询串):http://www.cnblogs.com/shuchao?

7、多余的& 无用的查询变量:http://www.example.com/display?id=123&fake=fake

### URL缺少字符串

缺少"/":"http://www.cnblogs.com/shuchao"
查询串缺少名称或者值:"http://www.example.com/display?id=" 或者 "http://www.example.com/display?=123"

### 其他不规范的URL

1、"http://shuchao.cnblogs.com/" 与 "http://www.cnblogs.com/shuchao/"其实是相同的内容即同一个资源，最好不要有两个urL

2、使用IP代替域名

3、大小写敏感("http://www.google.cn/Intl/zh-CN/about.html" 和"http://www.google.cn/intl/zh-CN/about.html")

4、查询变量顺序混乱:"http://www.example.com/test.aspx?bar=1&a=test"

5、含临时的状态变量:http://www.example.com/test?back=/prevpage.aspx

# 设计URL应该遵循的原则

### 一、简单，好记

简单好记的域名会给人以深刻的印象。

### 二、URL中的字母全部用小写

全部用小写，用户比较容易输入，不用因为大小写混合而出现错误，这是人们的输入习惯
有些服务器是区分大小写的，例如Lunix服务器，这样在站长做链接或者是用户输入时，会因为大小写的问题而出现404错误，
而且robots也是区分大小写的，如果大小写搞错了，可能会造成不能收录的严重问题。所以建议所有的URL都使用小写

### 三、连词符的使用

目录或者文件名中如果有两个单词组成时，一般建议中间使用中划线(-)隔开，
切记不要使用下划线或者其他字符，在搜索引擎中，它是把中划线当作一个空格来处理的，而下划线则是被忽略的，
例如seo-caipiao会被读成seo与caipiao。这是比较友好的写法

### 四、URL中避免太多参数

设计的则是URL中的参数应该尽量减少，不要超过三个，一般的情况下URL中的参数2-3个就可以了。

### 五、目录层次尽量少

这里所指的目录层次是指物理目录结构，而不是指逻辑结构，我们在进行URL的设计时，
网站的结构要尽量的去减少目录层次，层次不能太深了，一般建议不要超过三层，特别对于一些新站来说，
权重低，搜索引擎蜘蛛爬行得很浅，深一点的页面，蜘蛛都很可能不会去爬行的，所以要尽量的做到使目录层次减少，
URL缩短。根据观察，百度尤其比较喜欢目录层次比较少的页面。

### 六、文件名及目录名要具描述性

文件名及目录名要具有可描述性，不但让用户一眼就能看出来这个页面是关于什么的，
对用户体验比较友好，而且搜索引擎也比较喜欢这样的URL。
例如一个关于新闻的目录，我们可以把它命名为news，用户看到这个目录名称，大概就知道这个目录是关于什么内容的了。

### 七、URL应该呈现一个降级的次序

例如：域名/类型/分类/标题
例如：域名/年/月/日
http://domain.com/news/tech/2007/11/05/google-announces-android

### 其他

1.URL能反应站点的结构
2.URL是可以被用户猜测和hack的（也鼓励用户如此）
3.永久链接，Cool URLdon't change
4.动态的也要做成伪静态

# url规范诞生

### 一、基本规范

1、不能使用中文单词，最好使用有意义的英文单词，少用拼音。

2、层级不能超过三级。
例如：http://domain.com/xx/xx/xx/xx.html不被允许。

3、URL的参数不允许超过3个

4、URL全部小写

5、网站内部在链接到其他网页，尤其是主页时，只使用一种URL，即不允许同一个资源有多个URL。

6、不允许出现没有意义的下URL
例如：http://www.uxuexi.cn/123.html。谁也看不明白是什么意思。

7、如果是内容资源URL，不允许以参数的方法显示
例如：http://www.uxuexi.cn/user.html?userId=123 需要改成http://www.uxuexi.cn/user/123.html

### 二、URL类型设置

1、目录
一般用在频道页或是文章栏目（这种方式能获得更多的权重），最后面必须加上“/”
例如：http://www.uxuexi.cn/search需要改成http://www.uxuexi.cn/search/

2、网页
一般用来表现网页内容，需要直接显示在页面的必须以.html结尾
例如:http://www.uxuexi.cn/123 需要改成http://www.uxuexi.cn/user/123.html

3.特定功能或交互式
统一以.json 或者.html结尾
例如：
添加评论 http://www.uxuexi.cn/addcomment.json

### 三、静态化

1、不经常更新的内容采用静态化。例如：http://course.uxuexi.cn/detail/111.html。URL中不允许使用?带参数。
2、实时更新的内容采用伪静态。例如：http://www.uxuexi.cn/user/111.html。URL中不允许使用?带参数。
特定功能或交互式用动态URL。

# 约定

所有需要跳转页面的url必须进行统一的管理，统一使用cms:url自定义标签来实现，方便维护和优化。
例如
每次添加url，必须写上注释。
注释url功能，注释每一个参数是什么意思