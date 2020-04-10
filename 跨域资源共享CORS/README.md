普通跨域请求：只需服务端设置Access-Control-Allow-Origin即可，前端无须设置；若要带cookie请求，则前后端都需要设置。
需要注意的是：由于同源策略的限制，所读取的cookie为跨域请求接口所在域的cookie，而非当前页。如果想实现当前页的cookie的写入，需要nginx反向代理中设置proxy_cookie_domain和Node.js中间件代理中cookieDomainRewrite参数的设置。
目前，所有浏览器都支持该功能（IE8+、IE8/9需要使用XDomainRequest对象来支持CORS），CORS也已经成为主流的跨域解决方案。

# 1. 前端设置
1. 原生Ajax
```javascript
    var xhr = new XMLHttpRequest(); // IE8/9需用window.XDomainRequest兼容

    // 前端设置是否带cookie
    xhr.withCredentials = true;

    xhr.open('post', 'http://www.domain2.com:8080', true);
    xhr.setRequestHeader('Content-Type', 'application/x-www-form-urlencoded');
    xhr.send('user=admin');

    xhr.onreadystatechange = function () {
        if (xhr.readyState === 4 && xhr.status === 200) {
            alert(xhr.responseText);
        }
    }
```
2. jQuery ajax
```javascript
$.ajax({
    ...
   xhrFields: {
       withCredentials: true    // 前端设置是否带cookie
   },
   crossDomain: true,   // 会让请求头中包含跨域的额外信息，但不会含cookie
    ...
});
```
3. vue框架
a. axios设置：
`axios.defaults.withCredentials = true`
b. vue-resource设置
`Vue.http.options.credentials = true`
# 2. 服务端设置
若后端设置成功，前端浏览器控制台则不会出现跨域报错信息，反之说明没成功。
1. Java后台
```java
/*
 * 导入包：import javax.servlet.http.HttpServletResponse;
 * 接口参数中定义：HttpServletResponse response
 */

// 允许跨域访问的域名：若有端口需写全（协议+域名+端口），若没有端口末尾不用加'/'
response.setHeader("Access-Control-Allow-Origin", "http://www.domain1.com"); 

// 允许前端带认证cookie：启用此项后，上面的域名不能为'*'，必须指定具体的域名，否则浏览器会提示
response.setHeader("Access-Control-Allow-Credentials", "true"); 

// 提示OPTIONS预检时，后端需要设置的两个常用自定义头
response.setHeader("Access-Control-Allow-Headers", "Content-Type,X-Requested-With");
```
2. Nodejs后台
```javascript
var http = require('http')
var server = http.createServer()
var qs = require('querystring')

server.on('request', function (req, res) {
    var postData = ''

    // 数据块接收中
    req.addListener('data', function (chunk) {
        postData += chunk
    })

    // 数据接收完毕
    req.addListener('end', function () {
        postData = qs.parse(postData)

        // 跨域后台设置
        res.writeHead(200, {
            'Access-Control-Allow-Credentials': 'true', // 后端允许发送cookie
            'Access-Control-Allow-Origin': 'http://www.domain1.com', // 允许访问的域（协议+域名+端口）
            /*
                此处设置的cookie还是domain2的而非domain1，因为后端也不能跨域写cookie（nginx反向代理可以实现）
                但只要domain2中写入一次cookie认证，后面的跨域接口都能从domain2中获取cookie，从而实现所有的接口都能跨域访问。
             */
            'Set-Cookie': 'l=a123456;Path=/;Domain=www.domain2.com;HttpOnly' // HttpOnly的作用是让js无法读取cookie
        })

        res.write(JSON.stringify(postData))
        res.end()
    })
})

server.listen('8080')
console.log('Server is running at port 8080...')

```
