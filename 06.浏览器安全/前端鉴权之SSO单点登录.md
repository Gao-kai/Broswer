## 维护HTTP无状态的方案
由于HTTP协议在通信过程中的无状态特性，所以无法保持通信双方的状态，在前后端交互的过程中不可能用户每调用一次业务接口或者访问一次域名就需要用户进行一次登录鉴权，所有就出现了两种主流的维护HTTP登录状态在一段时间内不丢失的解决方案：
1. cookie + session
前端登录之后后端校验账户名和密码，校验通过生成session_id,然后将其种在当前请求域名下的cookie中，之后该域名下的每次请求都有携带cookie，服务端就可以取出cookie中的sid，拿去session库中进行校验，就知道了当前请求的人是谁。

优点是浏览器自动携带cookie，无需维护。缺点有很多，比如：
+ 由于是凭cookie中携带的sid鉴权，容易发生CSRF攻击
+ 配置了负载均衡之后session存储的问题，需要搞分布式存储

2. JWT Token
前端在登录之后服务端给前端返回一个JWT Token,这个token中既包含了过期时间、用户信息等数据实体，也包含了加密算法。服务端在校验登录之后将token返回给前端，前端每次在请求头中携带token即可，不会发生CSRF攻击，并且解决了session在服务端多台机器上同步存储等问题。

但是以上的维护状态的方案都只是单系统的方案，也就是一个客户端对应一个服务端。但如果一家公司有不仅一个web产品，比如有a.com，b.com这两个网站域名不同但都是同一家公司的，要想用户在登录a网站之后，然后登录b网站的时候无感知自动登录，也就是无需二次输入账户名和密码，这就涉及到了多系统单点登录的场景了。

## 什么是单点登录？
单点登录就是Single Sign On，简称为SSO。关于单点登录需要明确以下几点：
1. 单点登录也是一个单独的web产品，也就是xxx.sso.com。
2. sso是一个独立的身份认证中心，不管一家公司有多少个子系统，但是只有sso这个身份认证中心可以接受用户的登录名和密码等安全信息，其他系统是不提供登录入口的，他们只需要认证中心间接的告诉他们认证的结果即可。

### 单点登录核心原理
单点登录的最最核心的原理就是sso在接受到用户的登录名和密码之后，会做两个事情：
1. 从sso/login接口中获取到是从那个子系统跳转而来的，并且生成一个ticket票据，以url callback的形式重定向到原来的子系统，这样以来子系统就得到了sso的间接授权的令牌

2. 种下一个sso.com域名下的cookie,可以是一个凭证，记录该用户uid在sso系统的登录状态。

### 单点登录流程
假设现在用户没有登录任何系统，公司有：
+ a.com系统
+ b.com系统
+ sso统一身份认证中心

下面是用户先登录a系统然后访问b系统单点登录的过程：

1. 用户访问a.com业务接口，a.com服务端进行校验，发现没有登录凭证ticket,a系统将其重定向到sso中心，跳转过去的url是：
```js
https://www.sso.com/signin?redirectUrl=https://www.a.com/list/myquestion
```
2. sso系统读取本次请求的cookie中是否包含sso登录凭证，结果没有。接下来引导用户输入账号密码进行登录。
3. sso系统去user库中校验用户账号密码，校验通过之后生成一个临时的ticket，然后种一个sso域下的cookie，作用是下一次用户在访问sso.com域的时候就会自动携带上这个cookie。
4. sso系统将生成的ticket通过callback的形式发送给a系统，注意这里是服务端和服务端的通信，不会出现跨域的限制：
```js
https://www.a.com/list/myquestion?ticket=xxx
```
5. a系统获取到ticket然后去sso中心校验ticket是否有效并且读取用户信息，这一步必须得有，为了防止前端伪造。
6. sso校验该ticket有效并返回用户信息，a系统利用该ticket成功建立起和用户(浏览器端)的会话，浏览器端会在本地存储该ticket。
7. 之后用户和a系统之间的每次交互都会带上ticket来维持身份
8. 用户访问b.com业务接口，b系统进行校验，发现没有该系统的登录凭证ticket，b系统将其重定向到sso中心
9. sso读取cookie中是否包含sso登录凭证，由于之前登录a系统的时候种下了sso域下的该用户的登录信息，所以sso下发一个b系统的ticket
10. sso系统将b系统的ticket通过url的形式重定向到浏览器中的b系统页面
11. b系统将获取到的ticket去sso进行验证，验证通过之后返回业务接口请求