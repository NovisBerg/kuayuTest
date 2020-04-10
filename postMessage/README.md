postMessage是HTML5 XMLHttpRequest Level 2中的API，且是为数不多可以跨域操作的window属性之一，它可用于解决以下方面的问题：
1. 页面和其打开的新窗口的数据传递
2. 多窗口之间的消息传递
3. 页面与嵌套的iframe消息传递
4. 上面三个场景的跨域数据传递

用法：postMessage(data, origin)方法接收两个参数
data: html5规范支持任意基本类型或可复制的对象，但部分浏览器只支持字符串，所以传参时最好有JSON.stringy()序列化。
origin: 协议+主机+端口号，也可以设置为"*"，表示可以传递给任意窗口，如果要指定和当前窗口同源的话设置为"/"。

1. a.html(http://www.domain1.com/a.html)
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<iframe id="iframe" src="http://www.domain2.com/b.html" frameborder="0"></iframe>
</body>
<script>
    var iframe = document.getElementById('iframe');
    iframe.onload = function() {
        var data = {
            name: 'aym'
        };
        // 向domain2传递跨域数据
        iframe.contentWindow.postMessage(JSON.stringify(data), 'http://www.domain2.com');
    };

    // 接收domain2的返回数据
    window.addEventListener('message', function (e) {
        alert('data from domain2 ---> ' + e.data);
    }, false);
</script>
</html>
```
2. b.html(http://www.domain2.com/b.html)
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
    // 接收domain1的数据
    window.addEventListener('message', function (e) {
        alert('data from domain1 ---> ' + e.data);

        var data = JSON.parse(e.data);
        if (data) {
            data.number = 16;

            // 处理后再发回domain1
            window.parent.postMessage(JSON.stringify(data), 'http://www.domain1.com');
        }
    }, false);
</script>
</html>
```
