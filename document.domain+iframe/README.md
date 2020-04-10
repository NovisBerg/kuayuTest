通过document.domain + iframe跨域的方法仅限于主域相同，子域不同的跨域应用场景，如：
`http://parent.domain.com/a.html`
与
`http://child.domain.com/b.html`
实现原理：两个页面都通过js强制设置document.domain为基础主域，就实现了同域。
