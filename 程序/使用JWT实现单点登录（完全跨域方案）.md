# 使用JWT实现单点登录（完全跨域方案）



## 首先介绍一下什么是JSON Web Token（JWT）？

*官方文档是这样解释的*：JSON Web Token（JWT）是一个开放标准（RFC 7519），它定义了一种紧凑且独立的方式，可以在各方之间作为JSON对象安全地传输信息。此信息可以通过数字签名进行验证和信任。JWT可以使用秘密（使用HMAC算法）或使用RSA或ECDSA的公钥/私钥对进行签名。
虽然JWT可以加密以在各方之间提供保密，但只将专注于签名令牌。签名令牌可以验证其中包含的声明的完整性，而加密令牌则隐藏其他方的声明。当使用公钥/私钥对签署令牌时，签名还证明只有持有私钥的一方是签署私钥的一方。

通俗来讲，JWT是一个含签名并携带用户相关信息的加密串，页面请求校验登录接口时，请求头中携带JWT串到后端服务，后端通过签名加密串匹配校验，保证信息未被篡改。校验通过则认为是可靠的请求，将正常返回数据。

## 什么情况下使用JWT比较适合？

- 授权：这是最常见的使用场景，解决单点登录问题。因为JWT使用起来轻便，开销小，服务端不用记录用户状态信息（无状态），所以使用比较广泛；
- 信息交换：JWT是在各个服务之间安全传输信息的好方法。因为JWT可以签名，例如，使用公钥/私钥对儿 - 可以确定请求方是合法的。此外，由于使用标头和有效负载计算签名，还可以验证内容是否未被篡改。

## JWT的结构体是什么样的？

JWT由三部分组成，分别是头信息、有效载荷、签名，中间以（.）分隔，如下格式：

```
xxx.yyy.zzz
1
```

**header（头信息）**
由两部分组成，令牌类型（即：JWT）、散列算法（HMAC、RSASSA、RSASSA-PSS等），例如：

```java
{
  "alg": "HS256",
  "typ": "JWT"
}
1234
```

然后，这个JSON被编码为Base64Url，形成JWT的第一部分。

**Payload（有效载荷）**
JWT的第二部分是payload，其中包含claims。claims是关于实体（常用的是用户信息）和其他数据的声明，claims有三种类型： registered, public, and private claims。
**Registered claims：** 这些是一组预定义的claims，非强制性的，但是推荐使用， iss（发行人）， exp（到期时间）， sub（主题）， aud（观众）等；
**Public claims:** 自定义claims，注意不要和JWT注册表中属性冲突，[这里可以查看JWT注册表](https://www.iana.org/assignments/jwt/jwt.xhtml)
**Private claims:** 这些是自定义的claims，用于在同意使用这些claims的各方之间共享信息，它们既不是Registered claims，也不是Public claims。
**以下是payload示例：**

```java
{
  "sub": "1234567890",
  "name": "John Doe",
  "admin": true
}
12345
```

然后，再经过Base64Url编码，形成JWT的第二部分；

*注意：对于签名令牌，此信息虽然可以防止篡改，但任何人都可以读取。除非加密，否则不要将敏感信息放入到Payload或Header元素中。*

**Signature**
要创建签名部分，必须采用编码的Header，编码的Payload，秘钥，Header中指定的算法，并对其进行签名。
例如，如果要使用HMAC SHA256算法，将按以下方式创建签名：

```java
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  secret)
1234
```

签名用于验证消息在此过程中未被篡改，并且，在使用私钥签名令牌的情况下，它还可以验证JWT的请求方是否是它所声明的请求方。
输出是三个由点分隔的Base64-URL字符串，可以在HTML和HTTP环境中轻松传递，与SAML等基于XML的标准相比更加紧凑。
例如：

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.
eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.
SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
123
```

## JWT工作机制？

在身份验证中，当用户使用其凭据成功登录时，将返回JSON Web Token（即：JWT）。由于令牌是凭证，因此必须非常小心以防止出现安全问题。一般情况下，不应将令牌保留的时间超过要求。理论上超时时间越短越好。

每当用户想要访问受保护的路由或资源时，用户代理应该使用Bearer模式发送JWT，通常在Authorization header中。标题内容应如下所示：

```
Authorization: Bearer <token>
1
```

在某些情况下，这可以作为无状态授权机制。服务器的受保护路由将检查Authorization header中的有效JWT ，如果有效，则允许用户访问受保护资源。如果JWT包含必要的数据，则可以减少查询数据库或缓存信息。
如果在Authorization header中发送令牌，则跨域资源共享（CORS）将不会成为问题，因为它不使用cookie。

*注意：使用签名令牌，虽然他们无法更改，但是令牌中包含的所有信息都会向用户或其他方公开。这意味着不应该在令牌中放置敏感信息。*

## 使用JWT的好处是什么？

相比**Simple Web Tokens (SWT)（简单Web令牌）** and **Security Assertion Markup Language Tokens (SAML)（安全断言标记语言令牌）**；

- JWT比SAML更简洁，在HTML和HTTP环境中传递更方便；
- 在安全方面，SWT只能使用HMAC算法通过共享密钥对称签名。但是，JWT和SAML令牌可以使用X.509证书形式的公钥/私钥对进行签名。与签名JSON的简单性相比，使用XML数字签名可能会存在安全漏洞；
- JSON解析成对象相比XML更流行、方便。

## 以下是我实际项目中的应用分析

首先看一下大致的架构及流程图：
![架构图](https://img-blog.csdn.net/20180906132049199?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mjg3MzkzNw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
![流程图](https://img-blog.csdn.net/20180906132531764?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mjg3MzkzNw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

**主要有以下三步：**

- 项目一开始我先封装了一个JWTHelper工具包（[GitHub下载](https://github.com/9080/jwt-helper)），主要提供了生成JWT、解析JWT以及校验JWT的方法，其他还有一些加密相关操作，稍后我会以代码的形式介绍下代码。工具包写好后我将打包上传到私服，能够随时依赖下载使用；
- 接下来，我在客户端项目中依赖JWTHelper工具包，并添加Interceptor拦截器，拦截需要校验登录的接口。拦截器中校验JWT有效性，并在response中重新设置JWT的新值；
- 最后在JWT服务端，依赖JWT工具包，在登录方法中，需要在登录校验成功后调用生成JWT方法，生成一个JWT令牌并且设置到response的header中。

**以下是部分代码分享：**

- **JwtHelper工具类：**

```java
/**
 * @Author: Helon
 * @Description: JWT工具类
 * 参考官网：https://jwt.io/
 * JWT的数据结构为：A.B.C三部分数据，由字符点"."分割成三部分数据
 * A-header头信息
 * B-payload 有效负荷 一般包括：已注册信息（registered claims），公开数据(public claims)，私有数据(private claims)
 * C-signature 签名信息 是将header和payload进行加密生成的
 * @Data: Created in 2018/7/19 14:11
 * @Modified By:
 */
public class JwtHelper {

    private static Logger logger = LoggerFactory.getLogger(JwtHelper.class);

    /**
     * @Author: Helon
     * @Description: 生成JWT字符串
     * 格式：A.B.C
     * A-header头信息
     * B-payload 有效负荷
     * C-signature 签名信息 是将header和payload进行加密生成的
     * @param userId - 用户编号
     * @param userName - 用户名
     * @param identities - 客户端信息（变长参数），目前包含浏览器信息，用于客户端拦截器校验，防止跨域非法访问
     * @Data: 2018/7/28 19:26
     * @Modified By:
     */
    public static String generateJWT(String userId, String userName, String ...identities) {
        //签名算法，选择SHA-256
        SignatureAlgorithm signatureAlgorithm = SignatureAlgorithm.HS256;
        //获取当前系统时间
        long nowTimeMillis = System.currentTimeMillis();
        Date now = new Date(nowTimeMillis);
        //将BASE64SECRET常量字符串使用base64解码成字节数组
        byte[] apiKeySecretBytes = DatatypeConverter.parseBase64Binary(SecretConstant.BASE64SECRET);
        //使用HmacSHA256签名算法生成一个HS256的签名秘钥Key
        Key signingKey = new SecretKeySpec(apiKeySecretBytes, signatureAlgorithm.getJcaName());
        //添加构成JWT的参数
        Map<String, Object> headMap = new HashMap<>();
        /*
            Header
            {
              "alg": "HS256",
              "typ": "JWT"
            }
         */
        headMap.put("alg", SignatureAlgorithm.HS256.getValue());
        headMap.put("typ", "JWT");
        JwtBuilder builder = Jwts.builder().setHeader(headMap)
                /*
                    Payload
                    {
                      "userId": "1234567890",
                      "userName": "John Doe",
                    }
                 */
                //加密后的客户编号
                .claim("userId", AESSecretUtil.encryptToStr(userId, SecretConstant.DATAKEY))
                //客户名称
                .claim("userName", userName)
                //客户端浏览器信息
                .claim("userAgent", identities[0])
                //Signature
                .signWith(signatureAlgorithm, signingKey);
        //添加Token过期时间
        if (SecretConstant.EXPIRESSECOND >= 0) {
            long expMillis = nowTimeMillis + SecretConstant.EXPIRESSECOND;
            Date expDate = new Date(expMillis);
            builder.setExpiration(expDate).setNotBefore(now);
        }
        return builder.compact();
    }

    /**
     * @Author: Helon
     * @Description: 解析JWT
     * 返回Claims对象
     * @param jsonWebToken - JWT
     * @Data: 2018/7/28 19:25
     * @Modified By:
     */
    public static Claims parseJWT(String jsonWebToken) {
        Claims claims = null;
        try {
            if (StringUtils.isNotBlank(jsonWebToken)) {
                //解析jwt
                claims = Jwts.parser().setSigningKey(DatatypeConverter.parseBase64Binary(SecretConstant.BASE64SECRET))
                        .parseClaimsJws(jsonWebToken).getBody();
            }else {
                logger.warn("[JWTHelper]-json web token 为空");
            }
        } catch (Exception e) {
            logger.error("[JWTHelper]-JWT解析异常：可能因为token已经超时或非法token");
        }
        return claims;
    }

    /**
     * @Author: Helon
     * @Description: 校验JWT是否有效
     * 返回json字符串的demo:
     * {"freshToken":"A.B.C","userName":"Judy","userId":"123", "userAgent":"xxxx"}
     * freshToken-刷新后的jwt
     * userName-客户名称
     * userId-客户编号
     * userAgent-客户端浏览器信息
     * @param jsonWebToken - JWT
     * @Data: 2018/7/24 15:28
     * @Modified By:
     */
    public static String validateLogin(String jsonWebToken) {
        Map<String, Object> retMap = null;
        Claims claims = parseJWT(jsonWebToken);
        if (claims != null) {
            //解密客户编号
            String decryptUserId = AESSecretUtil.decryptToStr((String)claims.get("userId"), SecretConstant.DATAKEY);
            retMap = new HashMap<>();
            //加密后的客户编号
            retMap.put("userId", decryptUserId);
            //客户名称
            retMap.put("userName", claims.get("userName"));
            //客户端浏览器信息
            retMap.put("userAgent", claims.get("userAgent"));
            //刷新JWT
            retMap.put("freshToken", generateJWT(decryptUserId, (String)claims.get("userName"), (String)claims.get("userAgent"), (String)claims.get("domainName")));
        }else {
            logger.warn("[JWTHelper]-JWT解析出claims为空");
        }
        return retMap!=null?JSONObject.toJSONString(retMap):null;
    }

    public static void main(String[] args) {
       String jsonWebKey = generateJWT("123", "Judy",
               "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/68.0.3440.106 Safari/537.36");
       System.out.println(jsonWebKey);
       Claims claims =  parseJWT(jsonWebKey);
        System.out.println(claims);
       System.out.println(validateLogin(jsonWebKey));
    }
123456789101112131415161718192021222324252627282930313233343536373839404142434445464748495051525354555657585960616263646566676869707172737475767778798081828384858687888990919293949596979899100101102103104105106107108109110111112113114115116117118119120121122123124125126127128129130131132133134135136137138139140
```

- **AES加密工具类：**

```java
/**
 * @Author: Helon
 * @Description: AES加密工具类
 * @Data: Created in 2018/7/28 18:38
 * @Modified By:
 */
public class AESSecretUtil {

    /**秘钥的大小*/
    private static final int KEYSIZE = 128;
    
    /**
     * @Author: Helon
     * @Description: AES加密
     * @param data - 待加密内容
     * @param key - 加密秘钥
     * @Data: 2018/7/28 18:42
     * @Modified By:
     */
    public static byte[] encrypt(String data, String key) {
        if(StringUtils.isNotBlank(data)){
            try {
                KeyGenerator keyGenerator = KeyGenerator.getInstance("AES");
                //选择一种固定算法，为了避免不同java实现的不同算法，生成不同的密钥，而导致解密失败
                SecureRandom random = SecureRandom.getInstance("SHA1PRNG");
                random.setSeed(key.getBytes());
                keyGenerator.init(KEYSIZE, random);
                SecretKey secretKey = keyGenerator.generateKey();
                byte[] enCodeFormat = secretKey.getEncoded();
                SecretKeySpec secretKeySpec = new SecretKeySpec(enCodeFormat, "AES");
                Cipher cipher = Cipher.getInstance("AES");// 创建密码器
                byte[] byteContent = data.getBytes("utf-8");
                cipher.init(Cipher.ENCRYPT_MODE, secretKeySpec);// 初始化
                byte[] result = cipher.doFinal(byteContent);
                return result; // 加密
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
        return null;
    }

    /**
     * @Author: Helon
     * @Description: AES加密，返回String
     * @param data - 待加密内容
     * @param key - 加密秘钥
     * @Data: 2018/7/28 18:59
     * @Modified By:
     */
    public static String encryptToStr(String data, String key){

        return StringUtils.isNotBlank(data)?parseByte2HexStr(encrypt(data, key)):null;
    }


    /**
     * @Author: Helon
     * @Description: AES解密
     * @param data - 待解密字节数组
     * @param key - 秘钥
     * @Data: 2018/7/28 19:01
     * @Modified By:
     */
    public static byte[] decrypt(byte[] data, String key) {
        if (ArrayUtils.isNotEmpty(data)) {
            try {
                KeyGenerator keyGenerator = KeyGenerator.getInstance("AES");
                //选择一种固定算法，为了避免不同java实现的不同算法，生成不同的密钥，而导致解密失败
                SecureRandom random = SecureRandom.getInstance("SHA1PRNG");
                random.setSeed(key.getBytes());
                keyGenerator.init(KEYSIZE, random);
                SecretKey secretKey = keyGenerator.generateKey();
                byte[] enCodeFormat = secretKey.getEncoded();
                SecretKeySpec secretKeySpec = new SecretKeySpec(enCodeFormat, "AES");
                Cipher cipher = Cipher.getInstance("AES");// 创建密码器
                cipher.init(Cipher.DECRYPT_MODE, secretKeySpec);// 初始化
                byte[] result = cipher.doFinal(data);
                return result; // 加密
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
        return null;
    }

    /**
     * @Author: Helon
     * @Description: AES解密，返回String
     * @param enCryptdata - 待解密字节数组
     * @param key - 秘钥
     * @Data: 2018/7/28 19:01
     * @Modified By:
     */
    public static String decryptToStr(String enCryptdata, String key) {
        return StringUtils.isNotBlank(enCryptdata)?new String(decrypt(parseHexStr2Byte(enCryptdata), key)):null;
    }

    /**
     * @Author: Helon
     * @Description: 将二进制转换成16进制
     * @param buf - 二进制数组
     * @Data: 2018/7/28 19:12
     * @Modified By:
     */
    public static String parseByte2HexStr(byte buf[]) {
        StringBuffer sb = new StringBuffer();
        for (int i = 0; i < buf.length; i++) {
            String hex = Integer.toHexString(buf[i] & 0xFF);
            if (hex.length() == 1) {
                hex = '0' + hex;
            }
            sb.append(hex.toUpperCase());
        }
        return sb.toString();
    }

    /**
     * @Author: Helon
     * @Description: 将16进制转换为二进制
     * @param hexStr - 16进制字符串
     * @Data: 2018/7/28 19:13
     * @Modified By:
     */
    public static byte[] parseHexStr2Byte(String hexStr) {
        if (hexStr.length() < 1)
            return null;
        byte[] result = new byte[hexStr.length()/2];
        for (int i = 0;i< hexStr.length()/2; i++) {
            int high = Integer.parseInt(hexStr.substring(i*2, i*2+1), 16);
            int low = Integer.parseInt(hexStr.substring(i*2+1, i*2+2), 16);
            result[i] = (byte) (high * 16 + low);
        }
        return result;
    }

    public static void main(String[] args) {
        String ss = encryptToStr("eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VySWQiOiIxMjMiLCJ1c2VyTmFtZSI6Ikp1ZHkiLCJleHAiOjE1MzI3Nzk2MjIsIm5iZiI6MTUzMjc3NzgyMn0.sIw_leDZwG0pJ8ty85Iecd_VXjObYutILNEwPUyeVSo", SecretConstant.DATAKEY);
        System.out.println(ss);
        System.out.println(decryptToStr(ss, SecretConstant.DATAKEY));
    }
123456789101112131415161718192021222324252627282930313233343536373839404142434445464748495051525354555657585960616263646566676869707172737475767778798081828384858687888990919293949596979899100101102103104105106107108109110111112113114115116117118119120121122123124125126127128129130131132133134135136137138139140141
```

- **所使用的常量类：**

```java
/**
 * @Author: Helon
 * @Description: JWT使用常量值
 * @Data: Created in 2018/7/27 14:37
 * @Modified By:
 */
public class SecretConstant {

    //签名秘钥 自定义
    public static final String BASE64SECRET = "***********";

    //超时毫秒数（默认30分钟）
    public static final int EXPIRESSECOND = 1800000;

    //用于JWT加密的密匙 自定义
    public static final String DATAKEY = "************";

}
123456789101112131415161718
```

- **客户端pom依赖：**

```java
        <!--jwt工具类-->
        <dependency>
            <groupId>com.chtwm.component</groupId>
            <artifactId>jwt-helper</artifactId>
            <version>xxx</version>
        </dependency>
123456
```

- **客户端拦截器：**

```java
/**
 * @Author: Helon
 * @Description: 校验是否登录拦截器
 * @Data: Created in 2018/7/30 14:30
 * @Modified By:
 */
@Slf4j
public class ValidateLoginInterceptor implements HandlerInterceptor {

    @Override
    public boolean preHandle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o) throws Exception {
        //首先从请求头中获取jwt串，与页面约定好存放jwt值的请求头属性名为User-Token
        String jwt = httpServletRequest.getHeader("User-Token");
        log.info("[登录校验拦截器]-从header中获取的jwt为:{}", jwt);
        //判断jwt是否有效
        if(StringUtils.isNotBlank(jwt)){
            //校验jwt是否有效,有效则返回json信息，无效则返回空
            String retJson = JwtHelper.validateLogin(jwt);
            log.info("[登录校验拦截器]-校验JWT有效性返回结果:{}", retJson);
            //retJSON为空则说明jwt超时或非法
            if(StringUtils.isNotBlank(retJson)){
                JSONObject jsonObject = JSONObject.parseObject(retJson);
                //校验客户端信息
                String userAgent = httpServletRequest.getHeader("User-Agent");
                if (userAgent.equals(jsonObject.getString("userAgent"))) {
                    //获取刷新后的jwt值，设置到响应头中
                    httpServletResponse.setHeader("User-Token", jsonObject.getString("freshToken"));
                    //将客户编号设置到session中
                    httpServletRequest.getSession().setAttribute(GlobalConstant.SESSION_CUSTOMER_NO_KEY, jsonObject.getString("userId"));
                    return true;
                }else{
                    log.warn("[登录校验拦截器]-客户端浏览器信息与JWT中存的浏览器信息不一致，重新登录。当前浏览器信息:{}", userAgent);
                }
            }else {
                log.warn("[登录校验拦截器]-JWT非法或已超时，重新登录");
            }
        }
        //输出响应流
        JSONObject jsonObject = new JSONObject();
        jsonObject.put("hmac", "");
        jsonObject.put("status", "");
        jsonObject.put("code", "4007");
        jsonObject.put("msg", "未登录");
        jsonObject.put("data", "");
        httpServletResponse.setCharacterEncoding("UTF-8");
        httpServletResponse.setContentType("application/json; charset=utf-8");
        httpServletResponse.getOutputStream().write(jsonObject.toJSONString().getBytes("UTF-8"));
        return false;
    }

    @Override
    public void postHandle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, ModelAndView modelAndView) throws Exception {

    }

    @Override
    public void afterCompletion(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, Exception e) throws Exception {

    }
}
123456789101112131415161718192021222324252627282930313233343536373839404142434445464748495051525354555657585960
```

- **客户端拦截器在XML文件中配置：**

```java
	<!--拦截器配置-->
    <mvc:interceptors>
        <mvc:interceptor>
	        <!--需拦截url配置-->
            <mvc:exclude-mapping path="/api/aa/bb/**" />
            <mvc:exclude-mapping path="/api/aa/cc/test" />
            <bean id="validateLoginInterceptor" class="com.xxx.ValidateLoginInterceptor" />
        </mvc:interceptor>
    </mvc:interceptors>
123456789
```

到此，后台服务的配置已经完成，下一步就需要前端页面将JWT令牌从response响应头中取出，然后存入localstorage或cookie中。但是遇到跨域场景，处理起来就会比较复杂，因为一旦在浏览器中跨域将获取不到localstorage中的JWT令牌。例如www.a.com域下的JWT，在www.b.com域下是获取不到的，所以我选择了一种页面跨域的方式进行处理，使用iframe+H5的postMessage（[参考博文](https://www.cnblogs.com/tyrion1990/p/8134384.html)），具体我使用代码分享的方式来分析。

- **前端页面js代码（服务端）：**

```js
    /**CURD本地存储信息 start**/
          (function(doc,win){
              var fn=function(){};
              fn.prototype={
                  /*本地数据存储 t:cookie有效时间，单位s; domain:cookie存储所属的domain域*/
                  setLocalCookie: function (k, v, t,domain) {
                      //如果当前浏览器不支持localStorage将存储在cookie中
                      typeof window.localStorage !== "undefined" ? localStorage.setItem(k, v) :
                          (function () {
                              t = t || 365 * 12 * 60 * 60;
                              domain=domain?domain:".jwtserver.com";
                              document.cookie = k + "=" + v + ";max-age=" + t+";domain="+domain+";path=/";
                          })()
                  },
                  /*获取本地存储数据*/
                  getLocalCookie: function (k) {
                      k = k || "localDataTemp";
                      return typeof window.localStorage !== "undefined" ? localStorage.getItem(k) :
                          (function () {
                              var all = document.cookie.split(";");
                              var cookieData = {};
                              for (var i = 0, l = all.length; i < l; i++) {
                                  var p = all[i].indexOf("=");
                                  var dataName = all[i].substring(0, p).replace(/^[\s\uFEFF\xA0]+|[\s\uFEFF\xA0]+$/g,"");
                                  cookieData[dataName] = all[i].substring(p + 1);
                              }
                              return cookieData[k]
                          })();
                  },
                  /*删除本地存储数据*/
                  clearLocalData: function (k) {
                      k = k || "localDataTemp";
                      typeof window.localStorage !== "undefined" ? localStorage.removeItem(k) :
                          (function () {
                              document.cookie = k + "=temp" + ";max-age=0";
                          })()
                  },
                  init:function(){
                      this.bindEvent();
                  },
                  //事件绑定
                  bindEvent:function(){
                      var _this=this;
                      win.addEventListener("message",function(evt){
                          if(win.parent!=evt.source){return}
                          var options=JSON.parse(evt.data);
                          if(options.type=="GET"){
                              var data=tools.getLocalCookie(options.key);
                              win.parent.postMessage(data, "*");
                          }
                          options.type=="SET"&&_this.setLocalCookie(options.key,options.value);
                          options.type=="REM"&&_this.clearLocalData(options.key);
                      },false)
                  }
              };
              var tools=new fn();
              tools.init();
          })(document,window);
          /**CURD本地存储信息 end**/
1234567891011121314151617181920212223242526272829303132333435363738394041424344454647484950515253545556575859
```

- **前端页面js代码（客户端）：**

```js
        //页面初始化向iframe域名发送消息
        window.onload = function() {
            console.log('get key value......................')
            window.frames[0].postMessage(JSON.stringify({type:"GET",key:"User-Token"}),'*');
        }
        //监听message信息，接收从iframe域下获取到的token信息，然后存储到localstorage或cookie中
        window.addEventListener('message', function(e) {
            console.log('listen.....');
            var data = e.data;
            console.log(data);
            if(data != null){
                localStorage.setItem("User-Token", data);
            }
        }, false);
1234567891011121314
```

**总结：**
优点：在非跨域环境下使用JWT机制是一个非常不错的选择，实现方式简单，操作方便，能够快速实现。由于服务端不存储用户状态信息，因此大用户量，对后台服务也不会造成压力；
缺点：跨域实现相对比较麻烦，安全性也有待探讨。因为JWT令牌返回到页面中，可以使用js获取到，如果遇到XSS攻击令牌可能会被盗取，在JWT还没超时的情况下，就会被获取到敏感数据信息。
