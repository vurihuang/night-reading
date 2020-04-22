# Why is AJAX called Asynchronous JavaScript and XML

> 原文: [Why is ajax called asynchronous javascript and xml](https://hashnode.com/post/why-is-ajax-called-asynchronous-javascript-and-xml-civastzn310dm1j53jequzvpd#civaudfhu10k51j53oof0mizn)

早期的时候，那时候还没有 Javascript 而 HTML 还只是类似一种 XML 文档(虽然两者确实不同但都是基于 SGML)。
后来微软为了在 HTML 中加载 HTML 而引入了 `<iframe>` 标签，并增加了 `XMLHTTP` 组件(即我们熟知的
[XMLHttpRequest](https://developer.mozilla.org/zh-CN/docs/Web/API/XMLHttpRequest))。
通过这种技术，就可以获取 XML 并在页面中显示出来。最后，这种技术被 JavaScript 采用并成为所有浏览器的标准。
它的目的依旧是获取 XML 数据并在页面中展示，不过是通过 JavaScript 异步加载 XML 并展示。

想了解更多详情，可以查看 [Wikipedia][1]。

这就是为什么，虽然我们现在广泛使用 JSON 或其他的数据格式，我们依然称这项技术为
AJAX(Asynchronous JavaScript and XML)。不过现在有更多的人在使用JSON，这项技术也演化成了
AJAJ(Asynchronous JavaScript and JSON)。

[1]: https://en.wikipedia.org/wiki/Ajax_(programming)#History
