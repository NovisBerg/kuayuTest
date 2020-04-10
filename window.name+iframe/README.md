window.name的独特之处在于：name的值在不同的页面（甚至不同域名）加载后依然存在，并且可以支持非常长的name值（2MB）。
1. a.html(http://www.domain1.com/a.html)
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>

</body>
<script>
    var proxy = function (url, callback) {
        var state = 0;
        var iframe = document.createElement('iframe');

        // 加载跨域页面
        iframe.src = url;

        // onload事件会触发2次，第1次加载跨域页，并留存数据与window.name
        iframe.onload = function () {
            if (state === 1) {
                // 第2次onload（同域proxy页）成功后，读取同域window.name中数据
                callback(iframe.contentWindow.name);
                destroyFrame();
            } else if (state === 0) {
                // 第1次onload（跨域页）成功后，切换到同域代理页面
                iframe.contentWindow.location = 'http://www.domain1.com/proxy.html';
                state = 1;
            }
        };

        document.body.appendChild(iframe);

        // 获取数据以后销毁这个iframe，释放内存，这也保证了安全（不被其他域frame js访问）
        function destroyFrame() {
            iframe.contentWindow.document.write('');
            iframe.contentWindow.close();
            document.body.removeChild(iframe);
        }
    };

    // 请求跨域b页面数据
    proxy('http://www.domain2.com/b.html', function (data) {
        alert(data);
    })
</script>
</html>
```
2. proxy.html(http://www.domain1.com/proxy.html)
中间代理页，与a.html同域，内容为空即可。
3. b.html(http://www.domain2.com/b.html)
```html
<script>
    window.name = 'This is domain2 data!';
</script>
```

# 总结
通过iframe的src属性由外域转向本地域，跨域数据即由iframe的window.name从外域传递到本地域。这样就巧妙地绕过了浏览器的跨域访问限制，但同时它又是安全操作。
