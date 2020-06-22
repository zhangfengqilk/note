# 使用auth0构建JWT

# JWT

全称 `Json Web Token`

用于用户认证

用于前后端分离项目（App/微信小程序 无法产生cookie的项目）

文中所提到的 Token泛指身份验证时使用的令牌，而JWT，是json 格式的 web token，两者稍作区别

## JWT的构成

JWT 官网 [点击前往](https://jwt.io/) ,下列数据解释官网内容：

> 由三段字符串组成，两端中间用`.`分隔

```javascript
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
1
```

------

- 第一段字符串：`eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9`

> **HEADER：**`ALGORITHM` & `TOKEN TYPE`
>
> 包含生成token使用的算法与token类型

```javascript
{
  "alg": "HS256", //ALGORITHM ,默认算法 哈希256
  "typ": "JWT"  //TOKEN TYPE ,token类型
}
1234
```

> 将该JSON字符串做 base64Url 编码 得到`eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9`

------

- 第二段字符串：`eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ`

> **PAYLOAD：**`DATA`
>
> 数据载体，可以有自定义数据

```javascript
{
  "sub": "1234567890", // 自定义数据
  "name": "John Doe", // 自定义数据
  "iat": 1516239022	// token起作用时间 、生产日期
}
12345
```

> 将该JSON字符串做 base64Url 编码得到 `eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ`
>
> base64Url 可被解码，所以不宜将敏感信息写在token中

------

- 第三段字符串：`SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c`

> **VERIFY SIGNATURE**
>
> 签名验证

```java
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  secret

) secret base64 encoded
123456
```

> 1. 将第一段 + 第二段 字符串拼接起来（中间用`.`）
> 2. 将拼接完成的字符串进行加密， 算法 + 盐 + 密钥
> 3. 对算法 加密后的密文再做base64Url编码

## JWT实现认证的大致过程

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200129151217226.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2OTEzMjA4,size_16,color_FFFFFF,t_70)

假设使用HS256算法

1. 用户提交用户名+密码发送请求给服务端，服务端接受参数使用JWT 创建token返回 （先登录）

2. 用户第二次发送请求，带上token （登录后的操作 ）

3. 服务端接受token ，将token 分割开 （切割成三部分）

   ```javascript
   eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
   1
   ```

4. 对第二段字符串进行 base64Url 解密并获取 `PAYLOAD`数据

   ```javascript
   {
     "sub": "1234567890", // 自定义数据
     "name": "John Doe", // 自定义数据
     "iat": 1516239022	// token起作用时间 、生产日期
   }
   12345
   ```

5. 检测 `PAYLOAD`中的信息是否过期（比较 `iat` 和 `exp` 时间）

   为了保证前面两段数据没有被恶意篡改，来校验第三段字符串：

6. 因为 HS256 不能被反解密（RS256、MD5 亦是如此），所以将第一、二段字符串拼接， 进行 HS256

   ```javascript
   `eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9` + eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ
   
   //toHS256
   123
   ```

7. 拿着生成的 HS256 密文与第三段字符串进行比较

   - 一致，则校验通过
   - 不同，则不通过

## JWT 在JAVA中的应用

Maven库搜索 [JWT](https://mvnrepository.com/search?q=JWT), jjwt使用率排行第一（优点的话，我粗糙的看了下实现的代码，简明易懂，易使用，但网上说说封装的时候获取信息的能力有限），我这里使用的是 auth0，加入pom.xml 依赖

对于auth0的缺点，应该就是在验证RSA256 加密后的 token的时候，需要给算法实例传递两个key（公钥 + 私钥），这明显不太符合常规情况（常规情况是只需提供公钥即可）

```xml
<dependency>
    <groupId>com.auth0</groupId>
    <artifactId>java-jwt</artifactId>
    <version>3.9.0</version>
</dependency>
12345
```

观察GitHub中的教程：

https://github.com/auth0/java-jwt

JWT 规定了7个官方字段，提供使用。

- iss (issuer)：发布者
- sub (subject)：主题
- iat (Issued At)：生成签名的时间
- exp (expiration time)：签名过期时间
- aud (audience)：观众，相当于接受者
- nbf (Not Before)：生效时间
- jti (JWT ID)：编号

### Using HS256

HS256,签名/验证的时候用的都是同一个密钥，称为对称算法

#### 创建JWT

首先拿到算法实例，用于创建后的签入 `sign`（传入算法实例）

```JAVA
Algorithm algorithm = Algorithm.HMAC256("secret"); //secret 密钥，只有服务器知道
1
```

观察源码：

```java
public static Algorithm HMAC256(String secret) throws IllegalArgumentException {
 return new HMACAlgorithm("HS256", "HmacSHA256", secret);
}
123
```

> 实际使用的时候，将 secret 字符串弄得长点，复杂点

使用`JWT.create()`创建一个 `JWTCreator` 实例

```java
String token = JWT.create()
1
```

使用`sign()`签入`algorithm` 在签入之前：

使用`withIssuer()`给PAYLOAD添加一跳数据 => token发布者

使用`withClaim()`给PAYLOAD添加一跳数据 => 自定义声明 （key，value）

使用`withIssuedAt()` 给PAYLOAD添加一条数据 => 生成时间

使用`withExpiresAt()`给PAYLOAD添加一条数据 => 保质期

```java
@Test
public void creatToken(){
    try{
        Algorithm algorithm = Algorithm.HMAC256("secret");
        String token = JWT.create()
            .withIssuer("auth0")    // 发布者
            .withIssuedAt(new Date())   // 生成签名的时间
            .withExpiresAt(DateUtils.addHours(new Date(),2))   // 生成签名的有效期,小时
            .withClaim("name","wuyuwei") // 插入数据
            .sign(algorithm);

        System.out.println(token);
    }catch(JWTCreationException e){
        e.printStackTrace();
        //如果Claim不能转换为JSON，或者在签名过程中使用的密钥无效，那么将会抛出JWTCreationException异常。
    }


}
12345678910111213141516171819
```

> `withIssuer()`用于对参数 添加声明,观察源码：
>
> ```java
> public JWTCreator.Builder withIssuer(String issuer) {
>     this.addClaim("iss", issuer);
>     return this;
> }
> 
> private void addClaim(String name, Object value) {
>     if (value == null) {
>         this.payloadClaims.remove(name);
>     } else {
>         this.payloadClaims.put(name, value);
>     }
> }
> 
> 12345678910111213
> ```
>
> 其他`with`方法差不多

> 在`sign()`方法之前，存在几个`with()`，最后生成的第二段密文时（解密后）就有几条数据

执行单元测试得出以下结果

```java
2020-01-11 16:34:38 //输出时间
    
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJhdXRoMCIsIm5hbWUiOiJ3dXl1d2VpIiwiZXhwIjoxNTc4NzM4ODc4LCJpYXQiOjE1Nzg3MzE2Nzh9.B1TBjznMRsnIVsKEQDrkpLIA5AwLhoot3wE3e1KeM3Y


Process finished with exit code 0
123456
```

取出第二段字符串

```javascript
`eyJpc3MiOiJhdXRoMCIsIm5hbWUiOiJ3dXl1d2VpIiwiZXhwIjoxNTc4NzM4ODc4LCJpYXQiOjE1Nzg3MzE2Nzh9`
1
```

拿到[在线base64解密](http://www.ofmonkey.com/encrypt/base64) 中解密，得到结果：

```json
{
    "iss":"auth0",
    "name":"wuyuwei",
    "exp":1578738878,
    "iat":1578731678
}
123456
```

> 你也可以 复制三段生成的密文到粘贴到 [jwt.io](https://jwt.io/) 中提供给你的在线验证

#### 验证JWT

首先通过调用`JWT.require()`并传递`Algorithm`实例来创建 `JWTVerifier`实例,如果您要求令牌具有特定的Claim值，`use the builder to define them`(使用builder 来定义它们)。方法`build()`返回的实例是可复用的，因此您可以定义一次，且用它来验证不同的标记。最后调用`verifier.verify()`来验证token

```java
@Test
public void verifierToken(){
    String token = "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJhdXRoMCIsIm5hbWUiOiJ3dXl1d2VpIiwiZXhwIjoxNTc4NzM4ODc4LCJpYXQiOjE1Nzg3MzE2Nzh9.B1TBjznMRsnIVsKEQDrkpLIA5AwLhoot3wE3e1KeM3Y";
    try {
        Algorithm algorithm = Algorithm.HMAC256("secret");
        JWTVerifier verifier = JWT.require(algorithm)
            .withIssuer("auth0") //匹配指定的token发布者 auth0
            .build();
        DecodedJWT jwt = verifier.verify(token); //解码JWT ，verifier 可复用

        System.out.println(jwt);
    }catch (JWTVerificationException e){
        //无效的签名/声明
        System.out.println("666");
        e.printStackTrace();
    }
}
1234567891011121314151617
```

> 将上一个例子中得到的三段密文作为要验证的 token，

控制台抛出了错误,因为我创建的时间是`2020-01-11 16:34:38`,而我在写这条记录是时候已经 `20:18`了

```java
666
com.auth0.jwt.exceptions.TokenExpiredException: 
The Token has expired on Sat Jan 11 18:34:38 CST 2020.
123
```

验证令牌时，将自动进行时间验证，从而导致`JWTVerificationException`值无效时引发抛出异常。输出了666

如果将`.withIssuer("auth0")`中的参数修改为其它内容，则会提示：

```java
 The Claim 'iss' value doesn't match the required issuer.
 
 // “iss”值与所指定的 token发者 匹配不上。
123
```

------

接下来看一个成功的例子，我们将`creatToken`方法重新运行一遍单元测试，拿到三段密文

```java
2020-01-11 20:25:13 // 生成的时间
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJhdXRoMCIsIm5hbWUiOiJ3dXl1d2VpIiwiZXhwIjoxNTc4NzUyNzEzLCJpYXQiOjE1Nzg3NDU1MTN9.LlC_jiUJI1pe7uEDdmQz4JoL4Qyee3kSY_RWN2ibZmo
12
```

将三段密文放入第二个测试方法中 运行`verifierToken`单元测试

```java
//直接输出是 类名 + 哈希码 (默认执行了 `toString`方法)
System.out.println(jwt);// => com.auth0.jwt.JWTDecoder@212b5695 

// 获取withIssuer 设置的值
System.out.println(jwt.getIssuer()); // => auth0

// 获取开始生效时间/创建时时间
System.out.println(jwt.getIssuedAt()); // =>Sat Jan 11 20:25:13 CST 2020

// 获取过期时间，
System.out.println(jwt.getExpiresAt()); //=>Sat Jan 11 22:25:13 CST 2020

// 获取Claim中的值
Map<String, Claim> claims = jwt.getClaims();
Claim claim = claims.get("name");
System.out.println(claim.asString()); // => wuyuwei

//或者
Claim claim = jwt.getClaim("name");
System.out.println(claim.asString()); // => wuyuwei
1234567891011121314151617181920
```

重点代码：

Claim类是Claim值的包装器。它允许您将Claim作为不同的类类型。以下列出可能对你有帮助的方法：

- **`asBoolean()`**：返回布尔值；如果无法转换，则返回null。
- **`asInt()`**：返回Integer值；如果无法转换，则返回null。
- **`asDouble()`**：返回Double值；如果无法转换，则返回null。
- **`asLong()`**：返回Long值；如果无法转换，则返回null。
- **`asString()`**：返回String值；如果无法转换，则返回null。
- **`asDate()`**：返回日期值；如果无法转换，则返回null。这必须是一个NumericDate（Unix Epoch / Timestamp）。请注意，[JWT标准](https://tools.ietf.org/html/rfc7519#section-2)指定所有*NumericDate*值必须以秒为单位。

在上述测试方法中 直接输出 `claim`得到的是类名 + @ + 哈希值，所以使用辅助方法 `asString`，

如果你在`getClaim("exp")`时，还使用`asString()`将得到一个`null`值，这个时候请使用`asDate()`来转换接收对应的参数

#### 解码JWT

解码也就是将密文进行 base64 解密，请看源码：

```java
JWTDecoder(JWTParser converter, String jwt) throws JWTDecodeException {
    this.parts = TokenUtils.splitToken(jwt);

    String headerJson;
    String payloadJson;
    try {
        headerJson = StringUtils.newStringUtf8(Base64.decodeBase64(this.parts[0]));
        payloadJson = StringUtils.newStringUtf8(Base64.decodeBase64(this.parts[1]));
    } catch (NullPointerException var6) {
        throw new JWTDecodeException("The UTF-8 Charset isn't initialized.", var6);
    }

    this.header = converter.parseHeader(headerJson);
    this.payload = converter.parsePayload(payloadJson);
}
123456789101112131415
@Test
public void decodeToken(){
    String token = "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJhdXRoMCIsIm5hbWUiOiJ3dXl1d2VpIiwiZXhwIjoxNTc4NzUyNzEzLCJpYXQiOjE1Nzg3NDU1MTN9.LlC_jiUJI1pe7uEDdmQz4JoL4Qyee3kSY_RWN2ibZmo";
    try {
        DecodedJWT jwt = JWT.decode(token);

        String algorithm = jwt.getAlgorithm(); //获取算法类型
        String type = jwt.getType();	//获取token类型
        String issuer = jwt.getIssuer();	//获取token发布者
        Date expiresAt = jwt.getExpiresAt(); //获取token过期时间
        Date issuedAt = jwt.getIssuedAt();	// 获取token生产日期
        


        System.out.println(algorithm); 	//=> 	HS256
        System.out.println(type);		//=>	JWT
        System.out.println(issuer);		//=> 	auth0
        System.out.println(expiresAt);	//=>	Sat Jan 11 22:25:13 CST 2020
        System.out.println(issuedAt);	//=>	Sat Jan 11 20:25:13 CST 2020
        

    } catch (JWTDecodeException exception){
        //无效的 token
    }
}
12345678910111213141516171819202122232425
```

至此，你可以使用这些方法来创建、解码验证 前后端一致的 Key了， 使用HS256，双方之间仅共享一个密钥。由于使用相同的密钥生成签名和验证签名, 因此必须注意确保密钥不被泄密。

接下来相信你看 RS256的加密方式生成token也比较容易了：

### Using RS256

#### 定义

是一种非对称加密算法, 它使用公共/私钥 进行 签发/验证

Token提供方采用`privateKey` （私钥）签发token，也只能用`privateKey`解密

Token使用方获取`publicKey`（公钥）使用公钥验证token

#### 公钥 / 私钥

举两个例子：

**我给别人发信息时：**

> 私钥就是一个有锁的箱子，只有我才有这种特制的箱子，我把要发送的重要信息锁在箱子中，发给接收人
>
> 接收人收到箱子后会用我提供的公钥来开箱子，如果箱子能打开，就说明这是真实的我发来的重要信息
>
> **使用私钥加密，使用公钥解密**（使用私钥签发token，使用公钥验证token）
>
> 为了防止前端来的token可能会被伪造，非法操作会破坏数据安全，所以我需要用公钥来验证是不是我所授权签发出去的信息，如果是就放行，不是就拦截请求（从前端到后端，这里看似是别人给我发信息我来验证，验证token时，实则是验证是不是我发出去的信息，是不是我授权过的信息）

**别人给我发信息时：**

> 公钥就是一个有锁的箱子，人人都能拿到我提供的这个箱子，把源数据锁起来（加密），但是只有我有这箱子的钥匙，那就是私钥，把私钥插入到箱子的锁孔中，开锁（解密）就能得到源数据，如果我拿私钥解不开，拿这信息肯定不是发给我的，直接无视就好
>
> **使用公钥加密，使用私钥解密**

#### 创建的业务流程

前端携带参数给服务器，服务端首先产生 公钥 / 私钥对

使用 privateKey + payload + alg(rs256) 生成token

#### 如何保存JWT（token）

客户端接收服务器返回的JWT，将其存储在Cookie或localStorage中。

此后，客户端将在与服务器交互中都会带JWT。如果将它存储在Cookie中，就可以自动发送，但是不会跨域，

因此一般是将它放入HTTP请求的Header Authorization字段中。

Authorization: [token]

当跨域时，也可以将JWT被放置于POST请求的数据主体中。

#### 描述一个登陆的前后端业务流程

1. 前端输入账号和密码，提交登录请求

2. 后端接受参数，使用and语句查询数据库，看是否有此用户信息

   ```mysql
   where accountNumber = 'xxx' and password = 'xxx'
   1
   ```

3. 有结果则使用 `privateKey`+ `payload`+ `alg(rs256)` 生成token

4. 反馈给前端，前端判断 `response.status` 如果没问题，就说明登录成功

5. 提示登录成功，JavaScript中存储token 到 localStorage，跳转主页

### 代码实现

需要公钥和私钥，此处我们建立一个 实体类`RSA256Key`，公私钥的类型采用`java.security`包中的类型

```java
package cn.wuyuwei.tiny_shop.entity;

import lombok.Data;

import java.security.interfaces.RSAPrivateKey;
import java.security.interfaces.RSAPublicKey;

@Data
public class RSA256Key {
    private RSAPublicKey publicKey;
    private RSAPrivateKey privateKey;
}
123456789101112
```

#### 公钥 / 私钥的创建

并且需要 公钥 / 私钥的构造类 `SecretKeyUtils`

```java
package cn.wuyuwei.tiny_shop.utils;

import sun.misc.BASE64Decoder;
import sun.misc.BASE64Encoder;

import cn.wuyuwei.tiny_shop.entity.RSA256Key;

import java.security.Key;
import java.security.KeyPair;
import java.security.KeyPairGenerator;
import java.security.interfaces.RSAPrivateKey;
import java.security.interfaces.RSAPublicKey;
import java.util.HashMap;
import java.util.Map;

/**
 * KeyPairGenerator https://www.jianshu.com/p/4de1ee0e7206  key的生成使用方法
 *
 */
public class SecretKeyUtils {

    public static final String KEY_ALGORITHM = "RSA";
    private static final String PUBLIC_KEY = "RSAPublicKey";
    private static final String PRIVATE_KEY = "RSAPrivateKey";

    private static RSA256Key rsa256Key;

    //获得公钥
    public static String getPublicKey(Map<String, Object> keyMap) throws Exception {
        //获得map中的公钥对象 转为key对象
        Key key = (Key) keyMap.get(PUBLIC_KEY);
        //byte[] publicKey = key.getEncoded();
        //编码返回字符串
        return encryptBASE64(key.getEncoded());
    }
    public static String getPublicKey(RSA256Key rsa256Key) throws Exception {
        //获得map中的公钥对象 转为key对象
        Key key = rsa256Key.getPublicKey();
        //byte[] publicKey = key.getEncoded();
        //编码返回字符串
        return encryptBASE64(key.getEncoded());
    }

    //获得私钥
    public static String getPrivateKey(Map<String, Object> keyMap) throws Exception {
        //获得map中的私钥对象 转为key对象
        Key key = (Key) keyMap.get(PRIVATE_KEY);
        //byte[] privateKey = key.getEncoded();
        //编码返回字符串
        return encryptBASE64(key.getEncoded());
    }
    //获得私钥
    public static String getPrivateKey(RSA256Key rsa256Key) throws Exception {
        //获得map中的私钥对象 转为key对象
        Key key = rsa256Key.getPrivateKey();
        //byte[] privateKey = key.getEncoded();
        //编码返回字符串
        return encryptBASE64(key.getEncoded());
    }

    //解码返回byte
    public static byte[] decryptBASE64(String key) throws Exception {
        return (new BASE64Decoder()).decodeBuffer(key);
    }

    //编码返回字符串
    public static String encryptBASE64(byte[] key) throws Exception {
        return (new BASE64Encoder()).encodeBuffer(key);
    }

    //使用KeyPairGenerator 生成公私钥，存放于map对象中
    public static Map<String, Object> initKey() throws Exception {
        /* RSA算法要求有一个可信任的随机数源 */
        //获得对象 KeyPairGenerator 参数 RSA 1024个字节
        KeyPairGenerator keyPairGen = KeyPairGenerator.getInstance(KEY_ALGORITHM);
        keyPairGen.initialize(1024);

        //通过对象 KeyPairGenerator 生成密匙对 KeyPair
        KeyPair keyPair = keyPairGen.generateKeyPair();

        //通过对象 KeyPair 获取RSA公私钥对象RSAPublicKey RSAPrivateKey
        RSAPublicKey publicKey = (RSAPublicKey) keyPair.getPublic();
        RSAPrivateKey privateKey = (RSAPrivateKey) keyPair.getPrivate();
        //公私钥对象存入map中
        Map<String, Object> keyMap = new HashMap<String, Object>(2);
        keyMap.put(PUBLIC_KEY, publicKey);
        keyMap.put(PRIVATE_KEY, privateKey);
        return keyMap;
    }

    /**
     * 获取公私钥
     * @return
     * @throws Exception
     */
    public static synchronized RSA256Key getRSA256Key() throws Exception {
        if(rsa256Key == null){
            synchronized (RSA256Key.class){
                if(rsa256Key == null) {
                    rsa256Key = new RSA256Key();
                    Map<String, Object> map = initKey();
                    rsa256Key.setPrivateKey((RSAPrivateKey) map.get(SecretKeyUtils.PRIVATE_KEY));
                    rsa256Key.setPublicKey((RSAPublicKey) map.get(SecretKeyUtils.PUBLIC_KEY));
                }
            }
        }
        return rsa256Key;
    }

    public static void main(String[] args) {
        Map<String, Object> keyMap;
        try {
            keyMap = initKey();  // 使用 java.security.KeyPairGenerator 生成 公/私钥
            String publicKey = getPublicKey(keyMap);
            System.out.println("公钥：\n"+publicKey);
            String privateKey = getPrivateKey(keyMap);
            System.out.println("私钥：\n"+privateKey);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
123456789101112131415161718192021222324252627282930313233343536373839404142434445464748495051525354555657585960616263646566676869707172737475767778798081828384858687888990919293949596979899100101102103104105106107108109110111112113114115116117118119120121122
```

#### JWT创建 / 验证

构建JWT工具类 `JwtUtils`，其中的`DateUtils` 等非关键代码请自己编写

```java
package cn.wuyuwei.tiny_shop.utils;

import cn.wuyuwei.tiny_shop.entity.RSA256Key;
import cn.wuyuwei.tiny_shop.entity.UserInfo;
import com.alibaba.fastjson.JSON;
import com.auth0.jwt.JWT;
import com.auth0.jwt.algorithms.Algorithm;

import java.util.*;


public class JwtUtils {
    private static final String ISSUER = "WUYUWEI_BACK_API";

    /*------------------------------Using RS256---------------------------------*/
    /*获取签发的token，返回给前端*/
    public static String generTokenByRS256(UserInfo user) throws Exception{

        RSA256Key rsa256Key = SecretKeyUtils.getRSA256Key(); // 获取公钥/私钥
        Algorithm algorithm = Algorithm.RSA256(
            rsa256Key.getPublicKey(),rsa256Key.getPrivateKey());
        
        return createToken(algorithm, user);

    }

    /*签发token*/
    public static String createToken(Algorithm algorithm,Object data) throws Exception {

        String[] audience  = {"app","web"};
        return JWT.create()
                .withIssuer(ISSUER)   		//发布者
                .withAudience(audience)     //观众，相当于接受者
                .withIssuedAt(new Date())   // 生成签名的时间
                .withExpiresAt(DateUtils.offset(new Date(),2, Calendar.HOUR))    // 生成签名的有效期
                .withClaim("data", JSON.toJSONString(data)) //存数据
                .withNotBefore(new Date())  //生效时间
                .withJWTId(UUID.randomUUID().toString())    //编号
                .sign(algorithm);							//签入
    }

    /*验证token*/
     public static DecodedJWT verifierToken(String token)throws Exception{

                RSA256Key rsa256Key = SecretKeyUtils.getRSA256Key(); // 获取公钥/私钥

                //其实按照规定只需要传递 publicKey 来校验即可，这可能是auth0 的缺点
                Algorithm algorithm = Algorithm.RSA256(rsa256Key.getPublicKey(), rsa256Key.getPrivateKey());
                JWTVerifier verifier = JWT.require(algorithm)
                        .withIssuer(ISSUER)
                        .build(); //Reusable verifier instance 可复用的验证实例
                DecodedJWT jwt = verifier.verify(token);


                return jwt;

        }


}
123456789101112131415161718192021222324252627282930313233343536373839404142434445464748495051525354555657585960
```

> 使用 Alibaba 的 fastjson 的时候 请注意 data中的元素不能为 null ，否则栈溢出

之后外部就能使用该工具类来创建token了

```java
JwtUtils.generTokenByRS256(user)
1
```

user 是用户实体类实例，包含从数据库中查询到的数据，作为 `withClaim` 的参数，添加到token 中，属于`payload`的一部分

## 闲言碎语还说Token

在简书上看到一篇关于 web认证 时使用 session 与 token注意事项的精品文章

https://www.jianshu.com/p/805dc2a0f49e