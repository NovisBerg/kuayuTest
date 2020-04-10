通过location.hash与iframe进行跨域。

实现的原理是：A域：a.html -> B域：b.html -> A域：c.html

a与b属于不同域，只能通过hash值单向通信，b与c也是不同域，所以也只能单向通信。但c与a同域，所以c可以通过parent.parent访问a页面所有对象。

