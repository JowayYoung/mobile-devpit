# JS方向

### 禁止点击穿透

`移动端浏览器`里点击操作会存在`300ms`延迟，往往会造成点击延迟甚至点击无效，这个是众所周知的事情。

`2007年`苹果发布首款`iPhone`搭载的`Safari`为了将桌面端网站能较好地展示在`移动端浏览器`上而使用了双击缩放。该方案就是上述`300ms`延迟的主要原因，当用户执行第一次单击后会预留`300ms`检测用户是否继续执行单击，若是则执行缩放操作，若否则执行点击操作。鉴于该方案的成功，其他`移动端浏览器`也复制了该方案，现在几乎所有`移动端浏览器`都配备该功能。而该方案引发的点击延迟被称为**点击穿透**。

在前端领域里最早解决点击穿透是`jQuery时代`的`zepto`，估计现在大部分同学都未使用过`zepto`，其实它就是移动端版本的`jquery`。`zepto`封装`tap事件`能有效地解决点击穿透，通过监听`document`上的`touch事件`完成`tap事件`的模拟，并将`tap事件`冒泡到`document`上触发。

在`移动端浏览器`上不使用`click事件`而使用`touch事件`是因为`click事件`有着明显的延迟，后续又出现`fastclick`。该解决方案监听用户是否做了双击操作，可正常使用`click事件`，而点击穿透就交给`fastclick`自动判断。更多`fastclick`原理可自行百度，在此不作过多介绍。

[fastclick](https://github.com/ftlabs/fastclick)有现成的`NPM包`，可直接安装到项目里。引入`fastclick`可使用`click事件`代替`tap事件`，接入方式极其简单。

```js
import Fastclick from "fastclick";

FastClick.attach(document.body);
```

### 禁止滑动穿透

`移动端浏览器`里出现弹窗时，若在屏幕上滑动能触发弹窗底下的内容跟着滚动，这个是众所周知的事情。

首先明确解决滑动穿透需保持哪些交互行为，那就是`除了弹窗内容能点击或滚动，其他内容都不能点击或滚动`。目前很多解决方案都无法做到这一点，全部解决方案都能禁止`<body>`的滚动行为却引发其他问题。

- 弹窗打开后内部内容无法滚动
- 弹窗关闭后页面滚动位置丢失
- `Webview`能上下滑动露出底色

当打开弹窗时给`<body>`声明`position:fixed;left:0;width:100%`并动态声明`top`。声明`position:fixed`会导致`<body>`滚动条消失，此时会发现虽然无滑动穿透，但页面滚动位置早已丢失。通过`scrollingElement`获取页面当前滚动条偏移量并将其取负值且赋值给`top`，那么在视觉上就无任何变化。当关闭弹窗时移除`position:fixed;left:0;width:100%`和动态`top`。

`scrollingElement`可兼容地获取`scrollTop`和`scrollHeight`等属性，在`移动端浏览器`里屡试不爽。`document.scrollingElement.scrollHeight`可完美代替曾经的`document.documentElement.scrollHeight || document.body.scrollHeight`，一眼看上去就是代码减少了。

该解决方案在视觉上无任何变化，完爆其他解决方案，其实就是一种反向思维和障眼法。该解决方案完美解决`固定弹窗`和`滚动弹窗`对`<body>`全局滚动的影响，当然也可用于局部滚动容器里，因此很值得推广。

```css
body.static {
    position: fixed;
    left: 0;
    width: 100%;
}
```

```js
const body = document.body;
const openBtn = document.getElementById("open-btn");
const closeBtn = document.getElementById("close-btn");
openBtn.addEventListener("click", e => {
    e.stopPropagation();
    const scrollTop = document.scrollingElement.scrollTop;
    body.classList.add("static");
    body.style.top = `-${scrollTop}px`;
});
closeBtn.addEventListener("click", e => {
    e.stopPropagation();
    body.classList.remove("static");
    body.style.top = "";
});
```

### 支持往返刷新

点击`移动端浏览器`的`前进按钮`或`后退按钮`，有时不会自动执行旧页面的JS代码，这与`往返缓存`有关。这种情况在`Safari`上特别明显，简单概括就是往返页面无法刷新。

**往返缓存**指浏览器为了在页面间执行前进后退操作时能拥有更流畅体验的一种策略，以下简称`BFCache`。该策略具体表现为：当用户前往新页面前将旧页面的DOM状态保存在`BFCache`里，当用户返回旧页面前将旧页面的DOM状态从`BFCache`里取出并加载。大部分`移动端浏览器`都会部署`BFCache`，可大大节省接口请求的时间和带宽。

了解什么是`BFCache`再对症下药，解决方案就在`window.onunload`上做文章。

```js
// 在新页面监听页面销毁事件
window.addEventListener("onunload", () => {
    // 执行旧页面代码
});
```

若在`Vue SPA`上使用`keep-alive`也不能让页面刷新，可将接口请求放到`beforeRouteEnter()`里。

当然还有另一种解决方案。`pageshow事件`在每次页面加载时都会触发，无论是首次加载还是再次加载都会触发，这就是它与`load事件`的区别。`pageshow事件`暴露的`persisted`可判断页面是否从`BFCache`里取出。

```js
window.addEventListener("pageshow", e => e.persisted && location.reload());
```

若浏览器不使用`<meta http-equiv="Cache-Control" content="no-cache">`禁用缓存，该解决方案还是很值得一用。

### 解析有效日期

在苹果系统上解析`YYYY-MM-DD HH:mm:ss`这种日期格式会报错`Invalid Date`，但在安卓系统上解析这种日期格式完全无问题。

```js
new Date("2019-03-31 21:30:00"); // Invalid Date
```

查看`Safari`相关开发手册发现可用`YYYY/MM/DD HH:mm:ss`这种日期格式，简单概括就是年月日必须使用`/`衔接而不能使用`-`衔接。当然安卓系统也支持该格式，然而接口返回字段的日期格式通常是`YYYY-MM-DD HH:mm:ss`，那么需替换其中的`-`为`/`。

```js
const date = "2019-03-31 21:30:00";
new Date(date.replace(/\-/g, "/"));
```

### 修复高度坍塌

当页面同时出现以下三个条件时，键盘占位会把页面高度压缩一部分。当输入完成键盘占位消失后，页面高度有可能回不到原来高度，产生坍塌导致`Webview`底色露脸，简单概括就是输入框失焦后页面未回弹。

- 页面高度过小
- 输入框在页面底部或视窗中下方
- 输入框聚焦输入文本

只要保持前后滚动条偏移量一致就不会出现上述问题。在输入框聚焦时获取页面当前滚动条偏移量，在输入框失焦时赋值页面之前获取的滚动条偏移量，这样就能间接还原页面滚动条偏移量解决页面高度坍塌。

```js
const input = document.getElementById("input");
let scrollTop = 0;
input.addEventListener("focus", () => {
    scrollTop = document.scrollingElement.scrollTop;
});
input.addEventListener("blur", () => {
    document.scrollingElement.scrollTo(0, this.scrollTop);
});
```

### 修复输入监听

在苹果系统上的输入框输入文本，`keyup/keydown/keypress事件`可能会无效。当输入框监听`keyup事件`时，逐个输入英文和数字会有效，但逐个输入中文不会有效，需按回车键才会有效。

此时可用`input事件`代替输入框的`keyup/keydown/keypress事件`。

### 简化回到顶部

曾几何时编写一个`返回顶部`函数麻烦得要死，需`scrollTop`、`定时器`和`条件判断`三者配合才能完成。其实DOM对象里隐藏了一个很好用的函数可完成上述功能，一行核心代码就能搞定。

该函数就是[scrollIntoView](https://developer.mozilla.org/zh-cn/docs/web/api/element/scrollintoview)，它会滚动目标元素的父容器使之对用户可见，简单概括就是相对视窗让容器滚动到目标元素位置。它有三个可选参数能让`scrollIntoView`滚动起来更优雅。

- **behavior**：动画过渡效果，默认`auto无`，可选`smooth平滑`
- **inline**：水平方向对齐方式，默认`nearest就近对齐`，可选`start顶部对齐`、`center中间对齐`和`end底部对齐`
- **block**：垂直方向对齐方式，默认`start顶部对齐`，可选`center中间对齐`、`end底部对齐`和`nearest就近对齐`

```js
const gotopBtn = document.getElementById("gotop-btn");
openBtn.addEventListener("click", () => document.body.scrollIntoView({ behavior: "smooth" }));
```

当然还可滚动到目标元素位置，只需将`document.body`修改成目标元素的DOM对象。一行核心代码就能搞掂的事情为何还编写那么多代码去完成，不累吗？

### 简化懒性加载

与上述**简化回到顶部**一样，编写一个`懒性加载`函数也同样需`scrollTop`、`定时器`和`条件判断`三者配合才能完成。其实DOM对象里隐藏了一个很好用的函数可完成上述功能，该函数无需监听容器的`scroll事件`，通过浏览器自身机制完成滚动监听。

该函数就是[IntersectionObserver](https://developer.mozilla.org/zh-CN/docs/Web/API/IntersectionObserver)，它提供一种异步观察目标元素及其祖先元素或顶级文档视窗交叉状态的方法。详情可参照[MDN文档](https://developer.mozilla.org/zh-CN/docs/Web/API/IntersectionObserver)，在此不作过多介绍。

懒性加载的第一种使用场景：**图片懒加载**。只需确认图片进入可视区域就赋值加载图片，赋值完成还需对图片停止监听。

```html
<img data-src="pig.jpg">
<!-- 很多<img> -->
```

```js
const imgs = document.querySelectorAll("img.lazyload");
const observer = new IntersectionObserver(nodes => {
    nodes.forEach(v => {
        if (v.isIntersecting) { // 判断是否进入可视区域
            v.target.src = v.target.dataset.src; // 赋值加载图片
            observer.unobserve(v.target); // 停止监听已加载的图片
        }
    });
});
imgs.forEach(v => observer.observe(v));
```

懒性加载的第二种使用场景：**下拉加载**。在列表最底部部署一个占位元素且该元素无任何高度或实体外观，只需确认占位元素进入可视区域就请求接口加载数据。

```html
<ul>
    <li></li>
    <!-- 很多<li> -->
</ul>
<!-- 也可将#bottom以<li>的形式插入到<ul>内部的最后位置 -->
<div id="bottom"></div>
```

```js
const bottom = document.getElementById("bottom");
const observer = new IntersectionObserver(nodes => {
    const tgt = nodes[0]; // 反正只有一个
    if (tgt.isIntersecting) {
        console.log("已到底部，请求接口");
        // 执行接口请求代码
    }
})
bottom.observe(bottom);
```

### 优化扫码识别

通常`移动端浏览器`都会配备`长按二维码图片识别链接`的功能，但长按二维码可能无法识别或错误识别。二维码表面看上去是一张图片，可二维码生成方式却五花八门，二维码生成方式有以下三种。

- [x] 使用`<img>`渲染
- [x] 使用`<svg>`渲染
- [x] 使用`<canvas>`渲染

从网易MTL的测试数据得知，大部分`移动端浏览器`只能识别`<img>`渲染的二维码，为了让全部`移动端浏览器`都能识别二维码，那只能使用`<img>`渲染二维码了。若使用`SVG`和`Canvas`的方式生成二维码，那就想方设法把二维码数据转换成`Base64`再赋值到`<img>`的`src`上。

一个页面可能存在多个二维码，若长按二维码只能识别最后一个，那只能控制每个页面只存在一个二维码。

### 自动播放媒体

常见媒体元素包括音频`<audio>`和视频`<video>`，为了让用户得到更好的媒体播放体验与不盲目浪费用户流量，大部分`移动端浏览器`都明确规定不能自动播放媒体或默认屏蔽`autoplay`。为了能让媒体在页面加载完成后自动播放，只能显式声明播放。

```js
const audio = document.getElementById("audio");
const video = document.getElementById("video");
audio.play();
video.play();
```

对于像微信浏览器这样的内置浏览器，还需监听其应用SDK加载完成才能触发上述代码，以保障`WebView`正常渲染。其他内置浏览器同理，在此不作过多介绍。

```js
document.addEventListener("WeixinJSBridgeReady", () => {
    // 执行上述媒体自动播放代码
});
```

在苹果系统上明确规定用户交互操作开始后才能播放媒体，未得到用户响应会被`Safari`自动拦截，因此需监听用户首次触摸操作并触发媒体自动播放，而该监听仅此一次。

```js
document.body.addEventListener("touchstart", () => {
    // 执行上述媒体自动播放代码
}, { once: true });
```