# CSRF
Cross-Site-Request-Forgery 跨站请求伪造

## CSRF攻击原理

a网站：

```html
<a href="www.b.com/account/transfer">hello</a>
```

由于用户登录了b网站，导致转账被执行。

即利用了浏览器的登录“状态”。

## CSRF解决方案

*不要依赖cookie保存登录的状态！！！*

比如利用 Authentication Header，每次请求带上Header，那么CSRF攻击就无从进行了！！！

