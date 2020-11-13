# openapi

保障接口请求合法性的常见办法：

* API Key + API Secret
* JWT
* OAuth
* cookie-session认证

## API Key + API Secret

Resource + API Key + API Secret 匹配正确后，才可以访问Resource，通常还会配合时间戳来进行时效控制。

### 特点：

1）API Key / API Secret这种模式本质不是RBAC（基于决算单的权限管理），而是做的ACL访问权限控制用的。

2）服务器负责为每个客户端生成一对 key/secret （ key/secret 没有任何关系，不能相互推算），保存，并告知客户端。

3）一般是把所有的请求参数（API Key也放在请求参数内）排序后和API Secret做hash生成一个签名sign参数，服务器后台只需要按照规则做一次签名计算，然后和请求的签名做比较，如果相等验证通过，不相等就不通过。

4）为避免重放攻击，可加上 timestamp 参数，指明客户端调用的时间。服务端在验证请求时若 timestamp 超过允许误差则直接返回错误。

5）一般来说每一个api用户都需要分配一对API Key / API Secret的，比如你有几百万的用户，那么需要几百万个密钥对的，数据量不大时一般存在xml中，数据量大时可以存在mysql表中。

### 签名规则
* 线下分配appid和appsecret，针对不同的调用方分配不同的appid和appsecret。
* 加入timestamp（时间戳），10分钟内数据有效。
* 加入流水号nonce（防止重复提交），至少为10位。针对查询接口，流水号只用于日志落地，便于后期日志核查。 针对办理类接口需校验流水号在有效期内的唯一性，以避免重复请求。
  *  （1）nonce为客户端随机生成的验证码，当服务器接收到请求后，会把nonce存储到数据库中，一般使用redis，并设置一个有效期，一般和时间戳timestamp的失效时间保持一致，设为10分钟有效期。
  * （2）当服务器接收到请求后，用请求中nonce（比如7878）和redis中的nonce集合做比较，如果已经存在，则拒绝访问接口，只有当10分钟之内第一次使用7878这个动态码，才判定访问有效。
* 加入signature，所有数据的签名信息。 其中appid、timestamp、nonce、signature这四个字段放入请求头中。

### 优点：

1）占用计算资源和网络资源都很少。

2）安全性较好。请求体内没有明文secret和其他敏感信息。

 
### 缺点：

1）当API Key / API Secret足够多时，服务端有一定的存储成本

2）鉴权本身不能承载其它的信息，服务端只能通过API Key来区别调用者。

3）API Secret一旦泄密，将是致命的。

4）客户端sdk逻辑较多


## Json Web Token（JWT）
JWT的组成

JWT含有三个部分：

* 头部（header）
* 载荷（payload）
* 签证（signature）

### 头部

一般有两部分信息：类型、加密的算法（通常使用HMAC SHA256）
```
{
  "alg": "HS256",
  "typ": "JWT"
}
```

### 载荷（payload）

该部分一般存放一些有效的信息。

iss: jwt签发者
sub: jwt所面向的用户
aud: 接收jwt的一方
exp: jwt的过期时间，这个过期时间必须要大于签发时间
nbf: 定义在什么时间之前，该jwt都是不可用的
iat: jwt的签发时间
jti: jwt的唯一身份标识，主要用来作为一次性token,从而回避重放攻击


```
{
    "iss": "xx JWT",
    "iat": 1441593502,
    "exp": 1441594722,
    "aud": "www.example.com",
    "sub": "xx@163.com"
}
```
### 签证（signature）

JWT最后一个部分。该部分是使用了HS256加密后的数据；包含三个部分：

header(base64后的）
payload（base64后的）
secret 私钥

```
HMACSHA256(base64UrlEncode(header) + "." + base64UrlEncode(payload), secret)
```

示例
```
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJhdWQiOiJhZG1pbiIsInN1YiI6ImFjY2VzcyIsImlzcyI6Imluc2lnaHQiLCJleHAiOjE2MDUxNzM3ODksImlhdCI6MTYwNTE3MzcyOX0.7sm1Ej4crJD56S7i2bRMkp3JdVFF6EsHtap5UPsnxL8
```

### 使用方式

![jwt_desc](images/jwt_desc.webp)

1. 用户使用账号和密码发出POST登录请求；
2. 服务器使用私钥创建一个JWT；
3. 服务器返回这个JWT给浏览器；
4. 浏览器将该JWT串放在请求头中向服务器发送请求；
5. 服务器验证该JWT；
6. 返回响应的资源给浏览器。

注：考虑到JWT刷新续期的场景，第三步一般返回access_token（访问接口使用的token）、refresh_token（access_token过期后用于刷续期的token），refresh_token的过期时间比access_token的过期时间长

### JWT特点

紧凑：意味着这个字符串很小，甚至可以放在URL参数，POST Parameter中以Http Header的方式传输。
自包含：传输的字符串包含很多信息，别人拿到以后就不需要多次访问数据库获取信息，而且通过其中的信息就可以知道加密类型和方式（当然解密需要公钥和密钥）。

### 优点：

1）易于水平扩展（当访问量足够在时，相对于cookie-session方案而言的。如果把session中的认证信息都保存在JWT中，在服务端就没有session存在的必要了。当服务端水平扩展的时候，就不用处理session复制（session replication）/ session黏连（sticky session）或是引入外部session存储了）

2）无状态。

3）支持移动设备。

4）跨程序调用。

5）安全。

6）token的承载的信息很丰富。

7）客户端逻辑简单


事实上，所有使用token的模式，都是无状态、跨程序调用的。

### 缺点：

1）如果JWT中的payload的信息过多，网络资源占用较多。

2）JWT 的过期和刷新处理起来较麻烦。

3）JWT本身不加密信息，需要依赖https或者SSL加密信息


## openapi JWT技术方案 1
1. 控制台配置用户名密码
2. 客户端使用用户名密码请求接口，得到access_token
3. 客户端使用access_token请求数据
4. 只要在时间X秒内有正常请求，access_token自动续期，不会过期
5. X秒内没有正常请求，access_token过期失效，需要重新使用用户名密码请求新的access_token
6. 控制台修改密码或者删除了用户名，access_token立刻失效

缺点：access_token长期不会变化，需要引入refresh_token，刷新access_token

## openapi JWT技术方案 2
1. 控制台配置用户名密码
2. 客户端使用用户名密码请求接口，得到有效时间较短的 access_token（例如 10 分钟），和有效时间较长的 refresh_token（例如 7 天）
3. 客户端使用access_token请求数据，access_token不会自动续期
4. access_token过期后，使用refresh_token请求新的access_token和refresh_token，旧的access_token、refresh_token失效
5. 如果refresh_token已过期，需要重新使用用户名密码请求access_token、refresh_token
6. 控制台修改密码或者删除了用户名，access_token、refresh_token立刻失效

注：为使服务重启后原token继续保持有效，需要将token持久化，一般用redis+TTL

缺点：客户端逻辑更复杂一些

## JWT token失效方案
1. 数据库里存有效的refresh_token,设置字段：过期时间，是否失效
2. 每次响应refresh_token的请求时，检测redis里是否存在对应的refresh_token，若存在，则认为有效，响应请求，若不存在，则认为已过期，拒绝请求
3. 当需要使token失效时，删掉对应的refresh_token

优点：正常请求时不用去数据库校验ccess_token有效性，仅需低频校验refresh_token，平衡了性能和安全性

[stackoverflow](https://stackoverflow.com/questions/3487991/why-does-oauth-v2-have-both-access-and-refresh-tokens)

## JWT 并发更新access_token
客户端逻辑：
当请求返回access_token过期时，中断所有请求，调用一个不能并发执行的方法刷新access_token，再触发业务请求