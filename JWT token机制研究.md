# JWT token机制研究

#### 什么是JWT?

JWT(json web token)， 是基于[RFC7519](https://www.jianshu.com/p/3adfc2f8e15f#)定义的实现双方传输可信数据的安全机制。打个比方：JWT是有领导签字的文书，因为有领导签字，所以我确定对方是合法的，不是别人冒充的。那如何确认这个签字是合法的呢？这个涉及到JWT的签名机制，下面会详细讨论。

#### 如何使用JWT?

![img](https://upload-images.jianshu.io/upload_images/2230743-e2db44af8e810d38.png?imageMogr2/auto-orient/strip|imageView2/2/w/800/format/webp)

jwt.png

1 用户使用用户名/密码，向认证服务器请求。

2 认证服务器认证通过后，生成JWT token返回给用户

3 用户向应用服务器调用API请求资源时都需添加JWT token

4 应用服务器接收请求后先取出JWT token进行校验，通过后返回有效资源，否则返回错误。

JWT由三部分组成：Header，Payload和Signature。

格式是以字符串信息，由逗号分隔



```css
header.payload.signature
```

#### 1 创建header

Header json格式如下



```json
{    "type" : "JWT",    "alg" : "HS256"}
```

type: 指明这个object是JWT.

alg: 指明使用什么签名算法，上面例子中使用的是HMAC-SHA256 摘要算法。

#### 2 创建payload

Payload, 是存储在JWT的数据，在[JWT协议](https://www.jianshu.com/p/3adfc2f8e15f#)中定义为claims。

json格式如下：



```json
{    "iss" : "gd.com",    "sub" : "userId",    "exp" : "3600",    "iat" : "2012553355235"}
```

协议定义了几个标准claims,

"iss" issuer 请求JWT一方

"sub" "subject"

"exp" : 过期时间

“iat” : "issued at", 请求JWT的时间，可用来判断JWT是否过期。

以上claim是可选的，根据业务添加，也可以根据实际需求自定义claim。但需要注意的是payload的大小会影响生成JWT token后的大小。过大的JWT token会导致延迟和性能下降等负面影响。

#### 3创建签名：

签名过程参考伪代码如下:



```kotlin
data = base64urlEncode( header ) + “.” + base64urlEncode( payload )signature = Hash( data, secret );
```

使用base64对header和payload编码生成字符串，然后用逗号(.)拼接成data字串。



```cpp
// headereyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9// payloadeyJ1c2VySWQiOiJiMDhmODZhZi0zNWRhLTQ4ZjItOGZhYi1jZWYzOTA0NjYwYmQifQ
```

对data字串使用摘要算法(比如的HS256)生成签名.



```cpp
// signature-xN_h82PHVTCMA9vdoHrcZxH-x5mb11y1537t3rGzcM
```

然后将三部分用逗号拼接逗号(.)生成JWT token



```cpp
// JWT TokeneyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VySWQiOiJiMDhmODZhZi0zNWRhLTQ4ZjItOGZhYi1jZWYzOTA0NjYwYmQifQ.-xN_h82PHVTCMA9vdoHrcZxH-x5mb11y1537t3rGzcM
```

#### JWT如何保护我们的数据？

从JWT token的生成过程了解，JWT对数据进行的是签名而不是加密，所以JWT的payload部分不适合存放敏感数据，实际使用中需使用HTTPS，防止JWT token被非法拦截。理解加密和签名的区别，请[参考此文](https://www.jianshu.com/p/3adfc2f8e15f#)

#### 验证JWT：

在如何使用JWT的第4步中，应用服务器获取用户请求带的JWT token后，取出header和payload重新签名然后和原签名比较，比较一致则验证通过，不一致则说明该用户是非法请求拒绝。

#### refresh token 机制：

可参考：[https://vimsky.com/article/3601.html](https://www.jianshu.com/p/3adfc2f8e15f#)，采用Auth0的处理思路。

#### JWT和OAuth的关系：

两者没有任何关系，JWT是认证机制，OAuth是一套授权框架。OAuth的Access Token，Refresh Token也可以使用JWT形式。

#### 好用的JWT 三方框架：

[https://github.com/jwtk/jjwt](https://www.jianshu.com/p/3adfc2f8e15f#)

[https://github.com/pac4j/pac4j](https://www.jianshu.com/p/3adfc2f8e15f#)

#### 好用的demo：

[https://github.com/szerhusenBC/jwt-spring-security-demo](https://link.jianshu.com/?t=https%3A%2F%2Fgithub.com%2FszerhusenBC%2Fjwt-spring-security-demo)

#### 引用：

[5 Easy Steps to Understanding JSON Web Tokens (JWT)](https://www.jianshu.com/p/3adfc2f8e15f#)