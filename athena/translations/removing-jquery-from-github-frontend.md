# Removing jQuery from GitHub.com frontend

<!-- TOC -->

- [Removing jQuery from GitHub.com frontend](#removing-jquery-from-githubcom-frontend)
    - [为什么一开始选择 jQuery](#为什么一开始选择-jquery)
    - [Web 标准化的来临](#web-标准化的来临)
    - [逐渐解耦](#逐渐解耦)
    - [自定义标签](#自定义标签)
    - [Polyfills](#polyfills)

<!-- /TOC -->

> 原文：[Removing jQuery from GitHub.com frontend][1]

我们最近完成了一个里程碑，那就是 Github.com 在前端代码中移除了对 jQuery 的依赖。这意味着我们
结束了平滑且漫长地对 jQuery 库的解耦。这篇文章，我们会说明为什么一开始选择 jQuery，以及为何我们
意识到不再需要它，并且不同于为了替代它而引入其他框架，为什么我们选择基于标准浏览器 APIs 来实现我们需要的一切。

## 为什么一开始选择 jQuery

Github.com 在 2007 下半年引入了 [jQuery 1.2.1][2] 库。顺便交代一下，这里距 Google 发布第一版
Chrome 浏览器还有一年。当时并没有通过 CSS 选择器查询 DOM 元素的标准方式，也没有元素动画的标准方式，
并且 [XMLHttpRequest 接口][3] 才刚刚被 IE 开拓仅仅是个接口尚未被标准化，同其他许多接口一样在不同的浏览器上形态各异。

jQuery 使得操作 DOM、定义动画、发送 AJAX 请求变得更简单，因此它让 web 开发者能开发更现代化、
动态交互取代死板的应用。最重要的是，你通过 jQuery 编写的 JavaScript 代码在这个浏览器上能运作
那么它同样也能在其他浏览器上生效。在早期 GitHub 开发功能处于水深火热之中 jQuery 让小规模开发团队
能够更快速地开发新功能脱离这种困境，不需要给每个浏览器调整一部分代码。

归功于 jQuery 精巧简洁的接口方式，基于它开发的扩展库 [pjax][4] 和 [facebox][5] 可以良好地
为 GitHub.com 前端提供帮助。

我们会永远感谢 John Resig 和其他 jQuery 贡献者创造并一直维护如此有用且在当时是必不可少的库。

## Web 标准化的来临

经过了几年， GitHub 成长为一个拥有数百位工程师和一支逐渐成型的团队专门负责把关 JavaScript 代码
大小和质量的公司。我们一直担心的就是技术负债，有时候技术负债成为了一种依赖，虽然它曾经提供价值，
随着时间的到来它终究会被抛弃。

此刻 jQuery 成为不得不抛弃的一员，我们重新将它与现代浏览器提供的 web 标准解决方案对比并意识到：

- 可以用 `querySelectorAll()` 替代 `$(selector)`
- CSS 类名切换可以通过 [Element.classList][6]
- CSS 现在支持[在样式中定义可视元素][7]来取代通过 JavaScript 的方式
- 可以用 [标准化 Fetch][8] 来替代 `$.ajax`
- 对于跨平台使用 [addEventListener()][9] 已经足够稳定
- 我们可以轻易地将 [事件委派模式][10] 实现成一个轻量级库
- 有些 jQuery 提供的语法糖和 JavaScript 提供的解决方案重叠了

此外，链式语法不太满足我们接下来想要完成的代码，举个例子：

``` js
$('.js-widget')
  .addClass('is-loading')
  .show()
```

这样的语法虽然写起来很简单，但是按照我们的要求它所展示的意图不太明细。作者是期望当前页面是一个
还是多个 `js-widget` 元素？此外，当我们更新页面的时候不小心留下了一个 `js-widget` 类名，
浏览器会告知我们发生了错误吗？当没有匹配到任何选择器 jQuery 的默认行为是不做任何处理，
但是我们希望它能告知我们这是一个 bug。

最后一个原因，我们希望采用 [Flow][11] 通过注解方式在构建时刻进行静态类型检测，我们认为链式语法
不太利于静态分析因为几乎所有 jQuery 方法调用的结果都是同一种类型。我们选择 Flow 而不是 jQuery 
是因为 Flow 有一个 `@flow weak` 模式(注：带有该代码开头的 js 文件，Flow 会只对有加类型注解的
变量进行类型检测，现已被移除，见 [Weak Mode][12])可以对我们代码库中大量无类型的变量实现高效地检测分析。

总而言之，舍弃 jQuery 意味着我们更贴近 web 标准化，[MDN web 文档][13] 会是我们的前端开发者
必不可少的手册，我们会编写适应性更强的代码，从我们的包依赖中舍弃一个 30 KB 的库，提高加载页面和
执行 JavaScript 的速度。

## 逐渐解耦

虽然我们有了最终目标，要分配所有资源把所有 jQuery 代码重写为 vanilla JS(注：即原生 JavaScript，
有这么一个 JavaScript 库但是里头什么都没有，因为它的一切都在浏览器里。)是不太可行的，急匆匆地大改
会导致网站大量功能衰退导致努力白费。所以，我们是这么做的：

- 我们追踪并监控一行行 jQuery 代码调用的使用比率，并将数据绘制成图，来弄清楚使用率究竟是保持不变
还是升了或者降了。
    ![jQuery Usage](/jquery-usage.png)
- 我们不鼓励任何新编写的代码引入 jQuery。实现了 [eslint-plugin-jquery][14] 这么一个库，
它会在 CI(注：`Continuous Integration`，持续集成) 的时候自动检测来减轻我们的工作量，如果谁
尝试使用 jQuery 的代码例如 `$.ajax` 那么会报错失败。

- 旧的代码我们有大量的 eslint 规则， 所有通过代码注释附带的 `eslint-disable` 规则的代码，都表明这些不适用于我们新的规则。
- 我们创建了一个 pull request 机器人，当任何想要新增一条 `eslint-disable` 规则都会被它侦查到并通知我们。这样我们就能保证在代码审阅(Code Review)时就避免一些问题并给出建议。
- 有一部分老代码使用了 pjax 和 facebox 这两个 jQuery 插件，因此我们保留了同样的接口并将它们替换成原生 JS。通过静态类型检测给我们的重构工作带来了很大的信心。
- 有许多旧代码是基于 [rails-behaviors][15]，我们将 Ruby on Rails 的方法修改为非侵入式 JS(Unobtrusive JavaScript)方式，它会在 AJAX 生命周期时更新到具体的表单中：
    
    ``` js
    // 遗留方式
    $(document).on('ajaxSuccess', 'form.js-widget', function (event, xhr, settings, data) {
      // 添加响应的数据到 DOM 中
    ))
    ```
    
    修改为这种方式后为了不重写之前所有的调用方法，我们选择添加一个伪装的 `ajax*` 生命周期时间，保留了原来的行为；只有在这里调用了 `fetch()` 方法。
    
- 我们保留了一个定制版的 jQuery 直到确保所有代码不再它之后，再将它移除。举例来说，当我们清除了所有 jQuery-specific CSS 伪类选择器如 `:visible` 或者 `:checkbox` 我们就可以将 [Sizzle 模块][16] 移除掉；当所有的 `$.ajax` 调用都被 `fetch()` 取代后，我们就可以将 AJAX 模块移除掉。这里有两个意图：加速 JavaScript 执行时间，同时可以确保没有任何新的代码使用了被移除的功能。
- 根据我们的网站分析我们认为放弃支持旧版 IE 浏览器是可行的。这么做可以减少 JavaScript 兼容性开发同时支持更多现代浏览器。放弃支持 IE 8-9 之前的版本可以减少我们做 polyfill(注：即实现浏览器不支持的原生 API 代码) 的难度。
- 在构建 GitHub.com 的过程中有一部分巧妙的地方，我们尽可能地使用基本的 HTML 元素，只添加一部分 JavaScript 代码来完成剩下的功能。带来的好处就是，即使有些浏览器禁止了 JavaScript，网页表单和界面元素依然可以良好地工作。这样，我们就可以把一些遗留的代码删除掉，不需要将它们也重写为原生 JS。

随着多年来基于上面的这些努力，我们逐渐减少了对 jQuery 的依赖，直到再也没有一行代码使用到它。

## 自定义标签

近些年来我们不断在 [Custom Elements(自定义标签)][17] 上探索，它是浏览器自带的一部分，这意味着我们不需要做任何下载、解析或编译的工作。

自从 2014 年发布了 v0 版本时我们就已经编写了一部分自定义标签，由于标准尚未稳定我们并没有投入太多的工作量。直到 2017 年新一版的 Web Components v1(组件化) 发布之后，我们才[在 Chrome 和 Safari 上采用了自定义标签][18]。

在移除 jQuery 的过程中，我们需要寻找更合适的方式去实现自定义标签。举例来说，我们通过 [\<details-dialog\> 标签][19] 就可以实现 facebox 展示对话框的功能。

我们在实现自定义标签上同样也才用渐进式开发。我们尽可能地采用以标签的形式并只在上面添加想要完成的行为。以 `<local-time>` 标签为例，它默认会展示时间戳并会自动转为当前时区的时间；而嵌套在 `<details>` 标签下的 `<details-dialog>` 标签，不需要添加 JavaScript 代码就可以完成交互。

下面是如何实现 `<local-time>` 标签的例子：

``` js
// The local-time element displays time in the user's current timezone
// and locale.
//
// Example:
//   <local-time datetime="2018-09-06T08:22:49Z">Sep 6, 2018</local-time>
//
class LocalTimeElement extends HTMLElement {
  static get observedAttributes() {
    return ['datetime']
  }

  attributeChangedCallback(attrName, oldValue, newValue) {
    if (attrName === 'datetime') {
      const date = new Date(newValue)
      this.textContent = date.toLocaleString()
    }
  }
}

if (!window.customElements.get('local-time')) {
  window.LocalTimeElement = LocalTimeElement
  window.customElements.define('local-time', LocalTimeElement)
}
```

另一方面我们希望采用 [Shadow DOM][20]，它的一个强大之处在于可以为 web 带来更多的潜能，也意味着更多的适配工作。由于适配导致操作部分与 web 组件无关的 DOM 所带来的性能损耗，目前不太适合将其投入到生产环境。

## Polyfills

下面这些 polyfills 帮助我们能够使用浏览器标准化方式。我们仅仅在必要的时候做这些工作，例如，兼容老版浏览器也能够实现 JavaScript 模块化。

- [github/eventlistener-polyfill][21]
- [github/fetch][22]
- [github/form-data-entries][23]
- [iamdustan/smoothscroll][24]
- [javan/details-element-polyfill][25]
- [jonathantneal/closest][26]
- [kumarharsh/custom-event-polyfill][27]
- [marvinhagemeister/request-idle-polyfill][28]
- [mathiasbynens/Array.from][29]
- [mathiasbynens/String.prototype.codePointAt][30]
- [mathiasbynens/String.prototype.endsWith][31]
- [mathiasbynens/String.prototype.startsWith][32]
- [medikoo/es6-symbol][33]
- [nicjansma/usertiming.js][34]
- [rubennorte/es6-object-assign][35]
- [stefanpenner/es6-promise][36]
- [webcomponents/template][37]
- [webcomponents/URL][38]
- [webcomponents/webcomponentsjs][39]
- [WebReflection/url-search-params][40]
- [yola/classlist-polyfill][41]

---

参考：

- [XMLHttpRequest](https://developer.mozilla.org/zh-CN/docs/Web/API/XMLHttpRequest<Paste>)
- [Technical Debt](https://en.wikipedia.org/wiki/Technical_debt)
- [Custom Elements](https://developer.mozilla.org/zh-CN/docs/Web/Web_Components/Using_custom_elements)

[1]: https://githubengineering.com/removing-jquery-from-github-frontend/
[2]: https://blog.jquery.com/2007/09/16/jquery-1-2-1-released/
[3]: https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest
[4]: https://github.com/defunkt/jquery-pjax
[5]: https://github.com/defunkt/facebox
[6]: https://developer.mozilla.org/en-US/docs/Web/API/Element/classList
[7]: https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Animations/Using_CSS_animations
[8]: https://fetch.spec.whatwg.org/
[9]: https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/addEventListener
[10]: https://github.com/dgraham/delegated-events#readme
[11]: https://flow.org/
[12]: https://github.com/facebook/flow/issues/3316
[13]: https://developer.mozilla.org/en-US/docs/Web/JavaScript
[14]: https://github.com/dgraham/eslint-plugin-jquery#readme
[15]: http://josh.github.io/rails-behaviors/
[16]: https://sizzlejs.com/
[17]: https://developers.google.com/web/fundamentals/web-components/customelements
[18]: https://github.com/search?q=topic%3Aweb-components+org%3Agithub
[19]: https://github.com/github/details-dialog-element#readme
[20]: https://developers.google.com/web/fundamentals/web-components/shadowdom
[21]: https://github.com/github/eventlistener-polyfill#readme
[22]: https://github.com/github/fetch#readme
[23]: https://github.com/github/form-data-entries#readme
[24]: https://github.com/iamdustan/smoothscroll#readme
[25]: https://github.com/javan/details-element-polyfill#readme
[26]: https://github.com/jonathantneal/closest#readme
[27]: https://github.com/kumarharsh/custom-event-polyfill#readme
[28]: https://github.com/marvinhagemeister/request-idle-polyfill#readme
[29]: https://github.com/mathiasbynens/Array.from#readme
[30]: https://github.com/mathiasbynens/String.prototype.codePointAt#readme
[31]: https://github.com/mathiasbynens/String.prototype.endsWith#readme
[32]: https://github.com/mathiasbynens/String.prototype.startsWith#readme
[33]: https://github.com/medikoo/es6-symbol#readme
[34]: https://github.com/nicjansma/usertiming.js#readme
[35]: https://github.com/rubennorte/es6-object-assign#readme
[36]: https://github.com/stefanpenner/es6-promise#readme
[37]: https://github.com/webcomponents/template#template
[38]: https://github.com/webcomponents/URL#readme
[39]: https://github.com/webcomponents/webcomponentsjs#readme
[40]: https://github.com/WebReflection/url-search-params#readme
[41]: https://github.com/yola/classlist-polyfill#readme
