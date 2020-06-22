## 玩转 SpringBoot 2 之整合 JWT 下篇



## 前言

在[《玩转 SpringBoot 2 之整合 JWT 上篇》](https://mp.weixin.qq.com/s?__biz=MzUzMDQyMDM3MQ==&mid=2247483862&idx=1&sn=b9509d5aaec77b1c6380b4601a7a2481&scene=21#wechat_redirect) 中介绍了关于 JWT 相关概念和JWT 基本使用的操作方式。本文为 SpringBoot 整合 JWT 的下篇，通过解决 App 用户登录 Session 问题的实战操作，带你更深入理解 JWT。通过本文你还可以了解到如下内容：

1. SpringBoot 使用拦截器的实际应用

2. SpringBoot 统一异常处理

3. SpringBoot 快速搭建 RESTful Api

   

> 关于生成JWT 操作请参考 [《玩转 SpringBoot 2 之整合 JWT 上篇》](https://mp.weixin.qq.com/s?__biz=MzUzMDQyMDM3MQ==&mid=2247483862&idx=1&sn=b9509d5aaec77b1c6380b4601a7a2481&scene=21#wechat_redirect)

## 实战操作

### 登录操作

**登录操作流程图：**

![img](https://mmbiz.qpic.cn/mmbiz_png/9KGK4tWqBIw5CPVSOcR8EYbKoOd59b9sJhvpNdA6FRIib23SVVdjmSJSia1V5h3wLySFlQZwDJDoVqgB9Nflibqhw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**登录操作流程介绍：**

1. App 根据用户名和密码访问登录接口。
2. 如果用户名和密码错误则提示 App 用户密码输入错误。
3. 如果用户名和密码正确则获取用户信息（表示登录成功）并根据用户信息生成 Token 并将其存入ServletContext 中。
4. 将生成的 Token 返回给 App。

**登录操作具体代码：**

```
@RestController
public class LoginController {
    Logger log = LoggerFactory.getLogger(LoginController.class);
    @Autowired
    private JWTService jwtService;

    @RequestMapping("/login")
    public ReturnMessage<Object> login(String loginName,String password,HttpServletRequest request) {
        if(valid(loginName,password)) {
            ReturnMessageUtil.error(CodeEnum.LOGINNAMEANDPWDERROR);
        }

        Map<String,String> userInfo = createUserInfoMap(loginName,password);
        String token = jwtService.createToken(userInfo, 1);

        ServletContext context = request.getServletContext();
        context.setAttribute(token, token);
        log.info("token:"+token);
        return ReturnMessageUtil.sucess(token);
    }
}

    private Map<String,String> createUserInfoMap(String loginName, String password) {
        Map<String,String> userInfo = new HashMap<String,String>();
        userInfo.put("loginName", loginName);
        userInfo.put("password", password);
        return userInfo;
    }

    private boolean valid(String loginName, String password) {
        if(Objects.equal("ljk", loginName) && Objects.equal("123456", password) ) {
            return true;
        }
        return false;
    }
```

### 拦截操作

**拦截操作流程图：**

![img](https://mmbiz.qpic.cn/mmbiz_png/9KGK4tWqBIw5CPVSOcR8EYbKoOd59b9sjDrDEWMbAjLbUWKT3hKibvIRxO0Fnicjjmpk8nWhicw6hZFa3OnIMkswQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**拦截操作流程介绍：**

1. 服务器获取 （App访问具体的Api 时携带的 Token）Token，如果 Token 为空则提示 App Token不能为空。
2. 如果 Token 不为空则从 ServletContext 中获取 Token，如果不存在则提示 App 该Token为非法 Token ！
3. 如果 Token 不为空并且 ServletContext 中存在该Token，需要判断 Token 是否过期。如果未过期则放开拦截。
4. 如果Token 已经过期则提示 App Token已经过期，需要重新登录。

**拦截操作具体代码：**

```
public class LoginInterceptor implements HandlerInterceptor{

    Logger log = LoggerFactory.getLogger(LoginInterceptor.class);

    private JWTService jwtService;

    public LoginInterceptor(JWTService jwtService) {
        this.jwtService = jwtService;
    }

    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object o) throws Exception {

        log.info("Token Checkout processing");
        String token = request.getParameter("token");

        if (StringUtils.isEmpty(token)) {
            throw new JKException(CodeEnum.TOKENISEMPTY);
        }

        String tokenInServletContext = (String)request.getServletContext().getAttribute(token);
        if(StringUtils.isEmpty(tokenInServletContext)) {
            throw new JKException(CodeEnum.ILLEGALTOKEN);
        }

        try {
             jwtService.verifyToken(token);
        } catch (AlgorithmMismatchException  e) {
            log.error("Token Checkout processing AlgorithmMismatchException 异常！"+e.getLocalizedMessage());
            throw new JKException(CodeEnum.ILLEGALTOKEN);
        }catch (TokenExpiredException  e) {
            log.info("token已经过期");
            throw new JKException(CodeEnum.EXPIRETOKEN);
        }catch (SignatureVerificationException  e) {
            log.error("Token Checkout processing SignatureVerificationException 异常！"+e.getLocalizedMessage());
            throw new JKException(CodeEnum.ILLEGALTOKEN);
         }catch (Exception e) {
            log.error("Token Checkout processing 未知异常！"+e.getLocalizedMessage());
            throw e;
        }

        return true;
    }
}
```

### 退出操作

**退出操作流程介绍：**

访问退出接口并传递登录生成的 Token，然后将 ServletContext中的 Token 删除。

**退出操作具体代码：**

```
    @GetMapping("/logout")
    public ReturnMessage<?> logout(String token,String issuer,HttpServletRequest request) {
        ServletContext context = request.getServletContext();
        context.removeAttribute(token);
        return ReturnMessageUtil.sucess();
    }
```

### 公共代码

IndexController App 访问测试Api，具体代码如下：

```
@RestController
public class IndexController {

    @GetMapping("index")
    public ReturnMessage index() {
        return ReturnMessageUtil.sucess();
    }
}
```

统一异常次处理的 Handle

```
@RestControllerAdvice
public class ExceptionHandle {
    private final static Logger logger = LoggerFactory.getLogger(ExceptionHandle.class);
    @ExceptionHandler(value = Exception.class)
    //@ResponseBody
    public ReturnMessage<Object> handle(HttpServletResponse response, Exception exception) {
         response.setCharacterEncoding("utf-8");
        if(exception instanceof JKException) {
            JKException sbexception = (JKException)exception;
            return ReturnMessageUtil.error(sbexception.getCode(), sbexception.getMessage());
        }else {
            logger.error("系统异常 {}",exception);
            return ReturnMessageUtil.error(-1, "未知异常"+exception.getMessage());
        }
    }
}
```

> JWTService 工具类 代码可以在我的GitHub上进行查看，具体地址请查看下面代码示例章节。

## 测试

这里使用PostMan 进行测试，当然你也可以选用你顺手的工具进行测试哈！

访问 http://localhost:8080/sbe/login?loginName=ljk&password=123456进行登录获取Token，如下图所示date字段的值就是登录成功后生成的 Token。

![img](https://mmbiz.qpic.cn/mmbiz_png/9KGK4tWqBIw5CPVSOcR8EYbKoOd59b9slibvyctFRo1VXNjUWOw4H1nYmuL9Dib86b2oUbQEeI9vE79kIdTQLxCg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

访问 http://localhost:8080/sbe/index?token=具体token值，如下图所示访问成功！![img](https://mmbiz.qpic.cn/mmbiz_png/9KGK4tWqBIw5CPVSOcR8EYbKoOd59b9spLzNoR2ugRaBhliaYU9n4g3VZLaYjU4YTtI9aVMNXlL86E5umPG2j4A/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

如果不携带 Token 会提示Token不能为空，如下图所示：

![img](https://mmbiz.qpic.cn/mmbiz_png/9KGK4tWqBIw5CPVSOcR8EYbKoOd59b9sGgia1wVcm2J9kVnJpeVWco2lfQwXF54ESxCjPf8GsNTO5BsdkYczbibg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

如果输入不存在的 Token 则提示 非法Token！，如下图所示：

![img](https://mmbiz.qpic.cn/mmbiz_png/9KGK4tWqBIw5CPVSOcR8EYbKoOd59b9sdF5oviaWSb8cmUIDia21fqZxgSIboUfRCYnBD0LJveucpOcBNBWuQJrA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

http://localhost:8080/sbe/logout?token=具体token值 进行退出，如下图所示：

![img](https://mmbiz.qpic.cn/mmbiz_png/9KGK4tWqBIw5CPVSOcR8EYbKoOd59b9sic4Z0lp5rv3ksC5RC2Mibj6ibpoyDicezd4NgmYhbicZiajfIReryNWaKbLQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

退出后再次使用已经退出的Token 访问，会提示非法Token 如下图所示：

![img](https://mmbiz.qpic.cn/mmbiz_png/9KGK4tWqBIw5CPVSOcR8EYbKoOd59b9sicvIvuJR6ZV3pzqFAq63Qgw2BbHdmxcvhmmUiaBwf7ibWTezvFE0zh06A/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

## 小结

登录操作通过 JWT 生成Token 返回给App，拦截操作（也可以理解成校验操作）通过拦截器（HandlerInterceptor）来进行实现。最后退出操作是通过将Token 保存ServletContent 中，退出其实就是将 Token 从 ServletContent 中删除。

本文主旨是通过简单实现，带你了解 App 认证过程处理方式，对于拦截部分你也可以通过 Filter 或 Aop 来进行实现。Token 存储也可以考虑使用Redis来实现。

还有一个问题就是：JWT 续期问题本文并没有实现（JWT 过期时间延期问题）。这个部分就当成一个作业，欢迎大家在评论区说说你的解决方案？

## 代码示例

本文并没有对 JWTService 工具类、统一异常处理、拦截器使用搭建进行详细介绍。

如果你想直接查看本文全部源码，请在我的 GitHub仓库SpringbootExamples 中的 spring-boot-2.x-jwt 模块进行查看。

GitHub：https://github.com/zhuoqianmingyue/springbootexamples

同时你也可以通过查看我关于拦截器、统一异常处理、搭建 RESTful Api 详细教程总结自己完成相关的实现：

- [玩转 SpringBoot 2 快速整合拦截器](https://mp.weixin.qq.com/s?__biz=MzUzMDQyMDM3MQ==&mid=2247483869&idx=1&sn=d4c7ee87a7c2409bf2403ae470818e00&scene=21#wechat_redirect)
- [玩转 SpringBoot 2 快速整合 | RESTful Api 篇](https://mp.weixin.qq.com/s?__biz=MzUzMDQyMDM3MQ==&mid=2247483729&idx=1&sn=de160b3323a36b988cba57291a36ff4a&scene=21#wechat_redirect)
- [SpringBoot 2 快速整合 | 统一异常处理](https://mp.weixin.qq.com/s?__biz=MzUzMDQyMDM3MQ==&mid=2247483737&idx=1&sn=dcd66aba0c4029c9a01801e3ce5c8758&scene=21#wechat_redirect)