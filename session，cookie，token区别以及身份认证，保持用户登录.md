---
title: session，cookie，token区别以及身份认证，保持用户登录
---

## session，cookie，token

HTTP是无状态的，关闭网页之后，服务器是无法确定两个请求是同一个用户发出的，对此问题也慢慢的延申发展出了session，cookie，token，核心就是存储，三者都要满足同源

### COOKIE: 最早被提出来的本地储存方式，用于记录用户状态的一种方式，通过服务器设置，保存再客户端浏览器，再次发起去请求之后就会携带数据发送给服务器，服务器会设置相应的一些参数

#### cookie参数

- name：名称

- value：值，包括一些web服务器提供的访问令牌

- path：路径，用于读取对应路径

- Secure：字面含义，安全协议，保护服务器与浏览器传输过程

- Domain：遵从同源策略，无法跨域名，但是并不是特别严格的同源测了，子级可以访问

- HTTP：包含HTTP的一些属性，可以控制是否开源通过cookie来进行访问

- Max-size/Expires：设置cookie有效事件，超时就会失效

  cookie储存大小比较小，不可以跨域调用

### Session会话：因为COOKIE并不安全，当服务器和浏览器交互时也就是会话，关闭浏览器及结束，用户登陆成功之后，服务器会给每一个用户设置一个sessionId(无规律字符串)，当服务器设置完cookie之后会将sessionId添加入cookie之中，会设置sessionId相对应的有效期，也就是cookie的有效期，浏览器保存cookie也自然将sessionId保存，用sessionID替代用户名密码，相当与给用户名密码加密

### token是在session完善了用户数量庞大可能导致的一些问题，比如服务器崩溃导致数据库数据流失，因此诞生了JWT(JSON  web   Token)，用户登录之后，会生成一个JWT密文并发送给浏览器，和sessionId类似，保存在浏览器，再次登录时发送JWT(加密算法)与服务器的密文进行算法解密，从而确认用户登陆状态，只不过服务器只需要保存密文，JWT是保存再客户端的，减少服务器压力，jwt一般保存在本地Storage中

## session，cookie，token区别

- 首先，cookie是存储在本地浏览器中的一小字段，通过请求一起发送给服务器(提供交互)，和storage一样是一种存储手段，只不过有一些区别
- session呢是存储在服务器中的一组数据，将sessionid存储在cookie中用与验证身份
- token只是一段密文，jwt(头部，负载，签名三部分)就是token的一种，可以存储在许多地方，一般存储在本地localstorage发送服务器之后验证用户信息(解决跨域用户问题)

## token为什么不是存储在cookie，而存储在请求头authorization，因为authorization本就有认证的含义，专门用作认证的请求头，并且由于前后端分离，cookie，session可能存在一些跨域上的问题，更不规范

authorization：Bearer token(一段密文)，这里的Bearer指的是Token遵守的一种规范

然会会将密文提交给后端进行验证登录

