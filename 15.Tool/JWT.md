# Info

简介 : JWT(JSON Web Token)基于RFC 759的开放数据的标准,定义了一种宽松而紧凑的数据组合方式;

JWT是一种加密后的数据载体,可在各个应用之间进行数据传输;

`JWT Login流程图 :`

![image-20240306101141914](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20240306101141914.png) 

# Compose

简介 : JWT通常是 HEADER PAYLOAD(有效载荷)和SIGNATURE(签名)三部分组成,三者之间使用`.`链接;

`JWT Token :`

![image-20240306101518177](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20240306101518177.png) 

*Header 的组成部分 :*

- 声明类型,默认为 JWT
- 声明加密的算法 常用算法: HMAC RSA HS512

```json
{
    "typ" : "JWT",
    "alg" : "HS512"
}
```

*Payload 的组成部分 :* 存放实际需要传递的有效数据

- 标准载荷(注册声明): 对JWT信息的补充   
  - iss（Issuer）：发布者，即JWT的签发者；
  - sub（Subject）：主题，即JWT的主体，表示Token所面向的用户；
  - aud（Audience）：受众，即Token的接收者；
  - exp（Expiration Time）：过期时间，表示JWT的有效期，Unix时间戳格式；
  - nbf（Not Before）：生效时间，表示在该时间之前Token无效，Unix时间戳格式；
  - iat（Issued At）：签发时间，表示JWT的签发时间，Unix时间戳格式；
  - jti（JWT ID）：JWT的唯一标识符
- 自定义载荷(公共/私有声明): 可以添加任何信息,一般添加用户相关或其他业务需要的非敏感的信息;  

```json
{
    "iat": 1681571257,
    "exp": 1682783999,
    "aud": "Harvey",
    "iss": "Banne",
    "sub": "alluser",
    
    "user_info": [
      {
        "id": "1"
      },
      {
        "name": "Ron"
      },
      {
        "age": "18"
      }
    ],
}
```

*Signature 的组成部分 :* 对于Header 和 Payload部分的签名,防止数据的篡改;

```java
// 通过对Header  Payload 进行有效的Base64编码,使用.进行连接 + 私有秘钥形成字符串
// 再将字符串进行加密生成最终的签名
signature = HMACSHA256(base64UrlEncode(header) + "." + base64UrlEncode(payload), secret)
```

*JWT的优缺点 :*

- 优点:

  - 无状态: JWT本身不需要存储在服务器上,可以实现无状态的身份验证和授权

  - 跨平台: JWT支持多种编译语言和操作系统

  - 扩展性: 有效载荷部分可以存储任意的JSON数据

  - 安全性: JWT使用数字签名或加密算法对数据进行加密,确保了安全性

- 缺点: 

  - 可读性: 令牌本身不进行加密,有可能被破解后泄露信息

  - 令牌传输到客户端,存在被攻击者窃取的分险

  - 载荷具有大小限制不适合大量数据的传输

  - 令牌过期处理: JWT令牌一但发布一直有效果,无法撤销 需要令牌的过期处理和刷新机制

# Apply

`导入JJWT依赖 :`

```xml
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-api</artifactId>
    <version>0.11.2</version>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-impl</artifactId>
    <version>0.11.2</version>
    <scope>runtime</scope>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-jackson</artifactId>
    <version>0.11.2</version>
    <scope>runtime</scope>
</dependency>
```

`JwtUtil.java :` 创建和验证JWT令牌

```java
import io.jsonwebtoken.Jwts;
import io.jsonwebtoken.security.Keys;
import java.security.Key;

public class JwtUtil {
    
   /**
     * 生成jwt
     * 使用Hs256算法, 私匙使用固定秘钥
     *
     * @param secretKey jwt秘钥
     * @param ttlMillis jwt过期时间(毫秒)
     * @param claims    设置的信息
     * @return
     */
    public static String createJWT(String secretKey, long ttlMillis, Map<String, Object> claims) {
        // 1.指定签名的算法   Header部分
        SignatureAlgorithm signatureAlgorithm = SignatureAlgorithm.HS256;

        // 生成JWT的时间
        long expMillis = System.currentTimeMillis() + ttlMillis;
        Date exp = new Date(expMillis);

        // 2.设置jwt的body   Payload部分
        JwtBuilder builder = Jwts.builder()
                // 如果有私有声明，一定要先设置这个自己创建的私有的声明，这个是给builder的claim赋值，一旦写在标准的声明赋值之后，就是覆盖了那些标准的声明的
                .setClaims(claims)
            	// 设置过期时间
                .setExpiration(exp)
                // 设置签名使用的签名算法和签名使用的秘钥  
                .signWith(signatureAlgorithm, secretKey.getBytes(StandardCharsets.UTF_8));

        return builder.compact();
    }
    
   /**
     * Token解密
     *
     * @param secretKey jwt秘钥 此秘钥一定要保留好在服务端, 不能暴露出去, 否则sign就可以被伪造, 如果对接多个客户端建议改造成多个
     * @param token     加密后的token
     * @return
     */
    public static Claims parseJWT(String secretKey, String token) {
        // 得到DefaultJwtParser
        Claims claims = Jwts.parser()
                // 设置签名的秘钥
                .setSigningKey(secretKey.getBytes(StandardCharsets.UTF_8))
                // 设置需要解析的jwt
                .parseClaimsJws(token).getBody();
        return claims;
    }  
}
```

`测试案列 : `

```java
public class Main {
    public static void main(String[] args) {
        
        // secretKey  ttlMillis 通常存放在Jwt.properties文件中,通过配置文件设置
        String secretKey = "Banne";
        long  ttlMillis =  720000
        
        Map<String, Object> claims = new HashMap<>();
        claims.put("userId",100);
        
        String token = JwtUtil.createJWT(secretKey,ttlMillis,claims);
        System.out.println("Token: " + token);
        
	   Clamins content = JwtUtil.parseJWT(secretKey,token);
        Long userId = Long.valueOf(content.get("userId").toString());
    }
}
```

*JWT的应用场景 :*

- 一次性验证  用户注册成功后向邮箱发送激活文件,进行激活操作

- 访问授权

- 身份验证  验证用户身份,确保用户访问合法性;

- 令牌登录

  ![image-20240306111010305](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20240306111010305.png) 

# Scene optimization

##  Redis校验实现令牌泄露保护 

因为JWT是无状态的，当JWT令牌颁发后，在有效时间内，是无法进行销毁，所以就存在很大隐患：**令牌泄露**;

**避坑**：颁发JWT令牌时，在Redis中也缓存一份，当判定某个JWT泄露了，立即移除Redis中的JWT。当接口发起请求时，强制用户重新进行身份验证，直至验证成功。



## 临检JWT限制敏感操作 

基于JWT无状态性，同时泄露可能很大，一些涉及到敏感数据变动，执行临检操作。

**避坑**：在涉及到诸如新增，修改，删除，上传，下载等敏感性操作时，强制检查用户身份，如手机验证码，扫描二维码等手段，确认操作者是用户本人。



## 异常JWT监控：超频识别与限制 

当JWT令牌被盗取，一般会出现高频次的系统访问。针对这种情况，监控用户端在单位时间内的请求次数，当单位时间内的请求次数超出预定阈值值，则判定该用户JWT令牌异常。

**避坑**：当判断JWT令牌异常，直接进行限制(比如：IP限流，JWT黑名单等)。



## 地域检查杜绝JWT泄露可能 

一般用户活动范围是固定，意味着JWT客户端访问IP相对固定，JWT泄露之后，可能会先异地登录的情况。

**避坑**：对JWT进行异地访问检查，有效时间内，IP频繁变动可判断为JWT泄露。



# Interview

**问：什么是JWT？解释一下它的结构。**

JWT是一种开放标准，用于在网络上安全地传输信息。它由三部分组成：头部、载荷和签名。头部包含令牌的元数据，载荷包含实际的信息（例如用户ID、角色等），签名用于验证令牌是否被篡改。

**问：JWT的优点是什么？它与传统的session-based身份验证相比有什么优缺点？**

JWT的优点包括无状态、可扩展、跨语言、易于实现和良好的安全性。相比之下，传统的session-based身份验证需要在服务端维护会话状态，使得服务端的负载更高，并且不适用于分布式系统。

**问：在JWT的结构中，分别有哪些部分？每个部分的作用是什么？**

JWT的结构由三部分组成：头部、载荷和签名。头部包含令牌类型和算法，载荷包含实际的信息，签名由头部、载荷和密钥生成。

**问：JWT如何工作？从开始到验证过程的完整流程是怎样的？**

JWT的工作流程分为三个步骤：生成令牌、发送令牌、验证令牌。在生成令牌时，服务端使用密钥对头部和载荷进行签名。在发送令牌时，将令牌发送给客户端。在验证令牌时，客户端从令牌中解析出头部和载荷，并使用相同的密钥验证签名。

**问：什么是JWT的签名？为什么需要对JWT进行签名？如何验证JWT的签名？**

JWT的签名是由头部、载荷和密钥生成的，用于验证令牌是否被篡改。签名使用HMAC算法或RSA算法生成。在验证JWT的签名时，客户端使用相同的密钥和算法生成签名，并将生成的签名与令牌中的签名进行比较。

**问：什么是JWT的令牌刷新？为什么需要这个功能？**

令牌刷新是一种机制，用于解决JWT过期后需要重新登录的问题。在令牌刷新中，服务端生成新的JWT，并将其发送给客户端。客户端使用新的JWT替换旧的JWT，从而延长令牌的有效期。

**问：JWT是否加密？如果是，加密的部分是哪些？如果不是，那么它如何保证数据安全性？**

JWT本身并不加密，但可以在载荷中包含敏感信息。为了保护这些信息，可以使用JWE（JSON Web Encryption）对载荷进行加密。如果不加密，则需要在生成JWT时确保不在载荷中包含敏感信息。

**问：在JWT中，如何处理Token过期的问题？有哪些方法可以处理？**

JWT过期后，客户端需要重新获取新的JWT。可以通过在JWT中包含过期时间或使用refresh token等机制来解决过期问题。

**问：JWT和OAuth2有什么关系？它们之间有什么区别？**

JWT和OAuth2都是用于身份验证和授权的开放标准。JWT是一种身份验证机制，而OAuth2是一种授权机制。JWT用于在不同的系统中安全地传输信息，OAuth2用于授权第三方应用程序访问受保护的资源。

**问：JWT在什么场景下使用较为合适？它的局限性是什么？**

JWT在单体应用或微服务架构中的使用比较合适。它的局限性包括无法撤销、令牌较大、无法处理并发等问题。在需要针对每次请求进行访问控制或需要撤销令牌的情况下，JWT可能不是最佳选择。