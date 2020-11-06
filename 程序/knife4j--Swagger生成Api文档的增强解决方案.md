# knife4j--Swagger生成Api文档的增强解决方案

------

# 简介

knife4j是为Java MVC框架集成Swagger生成Api文档的增强解决方案,前身是swagger-bootstrap-ui,取名kni4j是希望它能像一把匕首一样小巧,轻量,并且功能强悍!

> Knife4j提供导出4种格式的离线文档(Html\Markdown\Word\Pdf)

目前项目主要的模块如下：

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8zNjczODkxLWZkYmZhMGJhNzFiYTAzNmIucG5nP2ltYWdlTW9ncjIvYXV0by1vcmllbnQvc3RyaXB8aW1hZ2VWaWV3Mi8yL3cvODMxL2Zvcm1hdC93ZWJw?x-oss-process=image/format,png)

# 开源仓库

Github

> https://github.com/xiaoymin/swagger-bootstrap-ui

码云

> https://gitee.com/xiaoym/knife4j

# 功能特性

- ## **简洁**

基于左右菜单式的布局方式,是更符合国人的操作习惯吧.文档更清晰...

- ## **个性化配置**

个性化配置项,支持接口地址、接口description属性、UI增强等个性化配置功能...

> 该UI增强包主要包括两大核心功能：文档说明 和 在线调试
>
> 1. 文档说明：根据Swagger的规范说明，详细列出接口文档的说明，包括接口地址、类型、请求示例、请求参数、响应示例、响应参数、响应码等信息，使用swagger-bootstrap-ui能根据该文档说明，对该接口的使用情况一目了然。
> 2. 在线调试：提供在线接口联调的强大功能，自动解析当前接口参数,同时包含表单验证，调用参数可返回接口响应内容、headers、Curl请求命令实例、响应时间、响应状态码等信息，帮助开发者在线调试，而不必通过其他测试工具测试接口是否正确,简介、强大。

- ## **增强**

接口排序、Swagger资源保护、导出Markdown、参数缓存众多强大功能...

# 在线预览

> http://knife4j.xiaominfo.com/doc.html

# 使用简介

- ## 单纯皮肤增强

不使用增强功能,纯粹换一个swagger的前端皮肤，这种情况是最简单的,你项目结构下无需变更

可以直接引用swagger-bootstrap-ui的最后一个版本1.9.6或者使用knife4j-spring-ui

### 老版本引用

```
<dependency>



  <groupId>com.github.xiaoymin</groupId>



  <artifactId>swagger-bootstrap-ui</artifactId>



  <version>1.9.6</version>



</dependency>
```

### 新版本引用

```
<dependency>



  <groupId>com.github.xiaoymin</groupId>



  <artifactId>knife4j-spring-ui</artifactId>



  <version>${lastVersion}</version>



</dependency>
```

- ## Spring Boot项目单体架构使用增强功能

在SpringBoot单体架构下,knife4j提供了starter供开发者快速使用

```
<dependency>



  <groupId>com.github.xiaoymin</groupId>



  <artifactId>knife4j-spring-boot-starter</artifactId>



  <version>${knife4j.version}</version>



</dependency>
```

该包会引用所有的knife4j提供的资源，包括前端Ui的jar包

- ## Spring Cloud微服务架构

在Spring Cloud的微服务架构下,每个微服务其实并不需要引入前端的Ui资源,因此在每个微服务的SpringBoot项目下,引入knife4j提供的微服务starter

```
<dependency>



  <groupId>com.github.xiaoymin</groupId>



  <artifactId>knife4j-micro-spring-boot-starter</artifactId>



  <version>${knife4j.version}</version>



</dependency>
```

在网关聚合文档服务下,可以再把前端的ui资源引入

```
<dependency>



   <groupId>com.github.xiaoymin</groupId>



   <artifactId>knife4j-spring-boot-starter</artifactId>



   <version>${knife4j.version}</version>



</dependency>
```

# 快速开始

相信使用过springboot的人大都知道和用过swagger，knife4j的使用方法和swagger几乎一模一样，没有什么学习成本，不同的是展示的接口UI文档更加友好和人性化。下面开始演示一个集成项目，首先来看pom文件依赖：

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8zNjczODkxLWIyM2UwOWQ4M2Q0NWY0NzkucG5nP2ltYWdlTW9ncjIvYXV0by1vcmllbnQvc3RyaXB8aW1hZ2VWaWV3Mi8yL3cvODgyL2Zvcm1hdC93ZWJw?x-oss-process=image/format,png)

只需要引入一个knife4j的starter即可，不用其它依赖，该包会引用所有的knife4j提供的资源，包括前端Ui的jar包。

springboot的配置文件和启动类不用做任何特殊配置，使用knife4j需要一个swagger的配置类，这个配置类和以前使用swagger几乎是一样的：

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8zNjczODkxLTJkNzQzMzBhYWE2OGFmMGMucG5nP2ltYWdlTW9ncjIvYXV0by1vcmllbnQvc3RyaXB8aW1hZ2VWaWV3Mi8yL3cvMTE2NC9mb3JtYXQvd2VicA?x-oss-process=image/format,png)

可以看到，内容上没什么变化，唯一的变化是类注解需要比原来的swagger多加一个 @EnableSwaggerBootstrapUi。这样knife4j的所有配置都完成了，启动项目可以访问地址：

> 认访问地址是：`http://${host}:${port}/doc.html`

来看一下效果：

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8zNjczODkxLTFjOWY5ODMzMmU5OWRhNTgucG5nP2ltYWdlTW9ncjIvYXV0by1vcmllbnQvc3RyaXB8aW1hZ2VWaWV3Mi8yL3cvMTIwMC9mb3JtYXQvd2VicA?x-oss-process=image/format,png)

# 配置简单接口

下面来配置一个简单的接口，查看文档的展示效果。首先来看接口的通用返回结果模型定义：

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8zNjczODkxLTc0MzIxZTEyZjc2Zjg0MDkucG5nP2ltYWdlTW9ncjIvYXV0by1vcmllbnQvc3RyaXB8aW1hZ2VWaWV3Mi8yL3cvMTEzNS9mb3JtYXQvd2VicA?x-oss-process=image/format,png)

注意要在文档上面展示，需要使用图中的注解。这个通用结果的具体数据是一个泛型类型。下面我们定义一个具体的业务数据模型：

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8zNjczODkxLTliN2U5YjA0NjQ1YmU2ZWIucG5nP2ltYWdlTW9ncjIvYXV0by1vcmllbnQvc3RyaXB8aW1hZ2VWaWV3Mi8yL3cvOTc2L2Zvcm1hdC93ZWJw?x-oss-process=image/format,png)

通过上面两个的定义，接口的返回类型就搞定了，下面来看接口：

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8zNjczODkxLTI5OWMxODljNDZkOTdjMTAucG5nP2ltYWdlTW9ncjIvYXV0by1vcmllbnQvc3RyaXB8aW1hZ2VWaWV3Mi8yL3cvMTIwMC9mb3JtYXQvd2VicA?x-oss-process=image/format,png)

这个接口类中分为几个部分需要注意，第一是类上面的@Api注解，描述了整个类的接口分类含义。还有一个是每个接口上面的 @ApiImplicitParams 注解，定义了接口的所有参数。还有@ApiResponses注解，定义了返回时，所有状态码所代表的的含义，最后是@ApiOperation注解，描述了单个接口本身的功能。

定义接口完成后，我们来重启项目，查看文档的效果：

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8zNjczODkxLTgxOTg0YzM5MWQ1NWI0NzQucG5nP2ltYWdlTW9ncjIvYXV0by1vcmllbnQvc3RyaXB8aW1hZ2VWaWV3Mi8yL3cvMTIwMC9mb3JtYXQvd2VicA?x-oss-process=image/format,png)

首页上面有一些变化，左侧列表多了HelloController类的整体描述栏目，我们点开这个栏目，可以看到类中定义的所有接口：

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8zNjczODkxLTQ0YjJlNTkxZTBhMTM2NjkucG5nP2ltYWdlTW9ncjIvYXV0by1vcmllbnQvc3RyaXB8aW1hZ2VWaWV3Mi8yL3cvMzI2L2Zvcm1hdC93ZWJw?x-oss-process=image/format,png)

点击这个接口，看到右侧非常详细的接口文档：

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8zNjczODkxLWYxZmFkMTgyMzRiZmQ0ZDAucG5nP2ltYWdlTW9ncjIvYXV0by1vcmllbnQvc3RyaXB8aW1hZ2VWaWV3Mi8yL3cvMTIwMC9mb3JtYXQvd2VicA?x-oss-process=image/format,png)

上图中展示的是接口地址，接口类型，接口描述和详细的入参描述，下面的相应状态展示了我们定义的两种状态类型，还有接口的回参也非常详细的列了出来：

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8zNjczODkxLTIyMGJiZDUxNjBiYmM0Y2MucG5nP2ltYWdlTW9ncjIvYXV0by1vcmllbnQvc3RyaXB8aW1hZ2VWaWV3Mi8yL3cvMTIwMC9mb3JtYXQvd2VicA?x-oss-process=image/format,png)

文字描述类型，数据结构，类型都有，还有响应示例，可以说非常清晰了。个人认为这种展示效果比原来的swagger要友好很多。

右侧还有调试功能，可以直接使用来测试接口：

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8zNjczODkxLTE0OGZlZjNiMmNhMThjOWUucG5nP2ltYWdlTW9ncjIvYXV0by1vcmllbnQvc3RyaXB8aW1hZ2VWaWV3Mi8yL3cvMTIwMC9mb3JtYXQvd2VicA?x-oss-process=image/format,png)

在左侧的文档管理中，还可以设置全局参数，支持类似jwt的带权限的测试：

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8zNjczODkxLTRiNjg0MGI4NmFmMzUzNzcucG5nP2ltYWdlTW9ncjIvYXV0by1vcmllbnQvc3RyaXB8aW1hZ2VWaWV3Mi8yL3cvMTIwMC9mb3JtYXQvd2VicA?x-oss-process=image/format,png)

对文档还可以进行个性化设置：

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8zNjczODkxLWU3NDRmNDYyNjQyZGFiYjEucG5nP2ltYWdlTW9ncjIvYXV0by1vcmllbnQvc3RyaXB8aW1hZ2VWaWV3Mi8yL3cvMTIwMC9mb3JtYXQvd2VicA?x-oss-process=image/format,png)

# 说明

上面简单介绍和演示了knife4j，这个starter不仅支持swagger-bootstrap-ui，原始的swagger-ui还是可以使用的:

> - 访问地址 http://`${host}:${port}`/swagger-ui.html

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8zNjczODkxLWU1Y2U5ZDdlYzQ3NTA0OGEucG5nP2ltYWdlTW9ncjIvYXV0by1vcmllbnQvc3RyaXB8aW1hZ2VWaWV3Mi8yL3cvMTIwMC9mb3JtYXQvd2VicA?x-oss-process=image/format,png)

有些更加喜欢原始风格的同学可以看这个页面。另外，swagger有很多注解，可以使文档展示的信息更加完善和友好，大家可以自行尝试和学习。