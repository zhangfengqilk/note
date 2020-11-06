# 把Object类转换为实体类


### 问题描述

在用SpringBoot写controller的时候，需要接受一个map的Object，之后要把Object转为特定的类，代码如下：

```java
public boolean postArticle(@RequestBody Map<String, Object> map) {
        ArticleInfo articleInfo = (ArticleInfo) map.get("articleInfo");
        ArticleContent articleContent = (ArticleContent) map.get("articleContent");
        System.out.println(articleInfo + " " + articleContent);
        return true;
}
123456
```

之后爆出异常：

```java
java.lang.ClassCastException: class java.util.LinkedHashMap cannot be cast to class 
cn.zi10ng.blog.domain.ArticleInfo (java.util.LinkedHashMap is in module java.base of loader
 'bootstrap'; cn.zi10ng.blog.domain.ArticleInfo is in unnamed module of loader 
 org.springframework.boot.devtools.restart.classloader.RestartClassLoader @19b54dc3)
1234
```

### 问题原因

map中取出的是Object，不能直接把Object转为特定的实体类

### 解决办法有两个：

- ***需要通过json来作为中间介质***：

```java
   public boolean postArticle(@RequestBody Map<String, Object> map) throws IOException {

        ObjectMapper objectMapper = new ObjectMapper();
        String jsonInfo = objectMapper.writeValueAsString(map.get("articleInfo"));
        String jsonContent = objectMapper.writeValueAsString(map.get("articleContent"));
        ArticleInfo articleInfo = objectMapper.readValue(jsonInfo,ArticleInfo.class);
        ArticleContent articleContent = objectMapper.readValue(jsonContent,ArticleContent.class);

        System.out.println(articleContent + " " +articleInfo);
        return articleService.insertArticle(articleInfo,articleContent);
    }
1234567891011
```

- ***通过com.fastxml.jackson的ObjectMapper对象进行转换：***

```java
ObjectMapper objectMapper = new ObjectMapper();
objectMapper.convertValue(Object fromValue, Class<T> toValueType);
```

 