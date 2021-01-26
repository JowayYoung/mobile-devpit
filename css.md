# CSS方向

### 自动适应布局

针对移动端，笔者通常会结合JS依据`屏幕宽度`与`设计图宽度`的比例动态声明`<html>`的`font-size`，以`rem`为长度单位声明所有节点的几何属性，这样就能做到大部分移动设备的页面兼容，兼容出入较大的地方再通过`媒体查询`做特别处理。

笔者通常将`rem布局比例`设置成`1rem=100px`，即在设计图上`100px`长度在CSS代码上使用`1rem`表示。

```js
function AutoResponse(width = 750) {
    const target = document.documentElement;
    if (target.clientWidth >= 600) {
        target.style.fontSize = "80px";
    } else {
        target.style.fontSize = target.clientWidth / width * 100 + "px";
    }
}
AutoResponse();
window.addEventListener("resize", () => AutoResponse());
```

当然还可依据`屏幕宽度`与`设计图宽度`的比例使用`calc()`动态声明`<html>`的`font-size`，这样就能节省上述代码。不对，是完全代替上述代码。

```css
html {
    font-size: calc(100vw / 7.5);
}
```

若以`iPad Pro`分辨率`1024px`为移动端和桌面端的断点，还可结合`媒体查询`做断点处理。`1024px`以下使用`rem布局`，否则不使用`rem布局`。

```css
@media screen and (max-width: 1024px) {
    html {
        font-size: calc(100vw / 7.5);
    }
}
```

### 自动适应背景

使用`rem布局`声明一个元素背景，多数情况会将`background-size`声明为`cover`。可能在设计图对应分辨率的移动设备下，背景会完美贴合显示，但换到其他分辨率的移动设备下就会出现左右空出`1px`到`npx`的空隙。

此时将`background-size`声明为`100% 100%`，跟随`width`和`height`的变化而变化。反正`width`和`height`都是量好的实际尺寸。

```css
.elem {
    width: 1rem;
    height: 1rem;
    background: url("pig.jpg") no-repeat center/100% 100%;
}
```

### 监听屏幕旋转

你还在使用JS判断横屏竖屏调整样式吗？那就真的`Out`了。

```css
/* 竖屏 */
@media all and (orientation: portrait) {
    /* 自定义样式 */
}
/* 横屏 */
@media all and (orientation: landscape) {
    /* 自定义样式 */
}
```

### 支持弹性滚动

在苹果系统上非`<body>`元素的滚动操作可能会存在卡顿，但安卓系统不会出现该情况。通过声明`overflow-scrolling:touch`调用系统原生滚动事件优化`弹性滚动`，增加页面滚动的流畅度。

```css
body {
    -webkit-overflow-scrolling: touch;
}
.elem {
    overflow: auto;
}
```

### 禁止滚动传播

与`桌面端浏览器`不一样，`移动端浏览器`有一个奇怪行为。当页面包含多个滚动区域时，滚完一个区域后若还存在滚动动量则会将这些剩余动量传播到下一个滚动区域，造成该区域也滚动起来。这种行为称为**滚动传播**。

若不想产生这种奇怪行为可直接禁止。

```css
.elem {
    overscroll-behavior: contain;
}
```

### 禁止屏幕抖动

对于一些突然出现滚动条的页面，可能会产生左右抖动的不良影响。在一个滚动容器里，打开弹窗就隐藏滚动条，关闭弹窗就显示滚动条，来回操作会让屏幕抖动起来。提前声明滚动容器的`padding-right`为滚动条宽度，就能有效消除这个不良影响。

每个`移动端浏览器`的滚动条宽度都有可能不一致，甚至不一定占位置，通过以下方式能间接计算出滚动条的宽度。`100vw`为视窗宽度，`100%`为滚动容器内容宽度，相减就是滚动条宽度，妥妥的动态计算。

```css
body {
    padding-right: calc(100vw - 100%);
}
```

### 禁止长按操作

有时不想用户长按元素呼出菜单进行`点链接`、`打电话`、`发邮件`、`保存图片`或`扫描二维码`等操作，声明`touch-callout:none`禁止用户长按操作。

有时不想用户`复制粘贴`盗文案，声明`user-select:none`禁止用户长按操作和选择复制。

```css
* {
    /* pointer-events: none; */ /* 微信浏览器还需附加该属性才有效 */
    user-select: none; /* 禁止长按选择文字 */
    -webkit-touch-callout: none;
}
```

但声明`user-select:none`会让`<input>`和`<textarea>`无法输入文本，可对其声明`user-select:auto`排除在外。

```css
input,
textarea {
    user-select: auto;
}
```

### 禁止字体调整

旋转屏幕可能会改变字体大小，声明`text-size-adjust:100%`让字体大小保持不变。

```css
* {
    text-size-adjust: 100%;
}
```

### 禁止高亮显示

触摸元素会出现半透明灰色遮罩，不想要！

```css
* {
    -webkit-tap-highlight-color: transparent;
}
```

### 禁止动画闪屏

在移动设备上添加动画，多数情况会出现闪屏，给动画元素的父元素构造一个`3D环境`就能让动画稳定运行了。

```css
.elem {
    perspective: 1000;
    backface-visibility: hidden;
    transform-style: preserve-3d;
}
```

### 美化表单外观

表单元素样式太丑希望自定义，`appearance:none`来帮你。

```css
button,
input,
select,
textarea {
    appearance: none;
    /* 自定义样式 */
}
```

### 美化滚动占位

滚动条样式太丑希望自定义，`::-webkit-scrollbar-*`来帮你。记住以下三个关键词就能随机应变了。

- [x] **::-webkit-scrollbar**：滚动条整体部分
- [x] **::-webkit-scrollbar-track**：滚动条轨道部分
- [x] **::-webkit-scrollbar-thumb**：滚动条滑块部分

```css
::-webkit-scrollbar {
    width: 6px;
    height: 6px;
    background-color: transparent;
}
::-webkit-scrollbar-track {
    background-color: transparent;
}
::-webkit-scrollbar-thumb {
    border-radius: 3px;
    background-image: linear-gradient(135deg, #09f, #3c9);
}
```

### 美化输入占位

输入框占位文本太丑，`::-webkit-input-placeholder`来帮你。

```css
input::-webkit-input-placeholder {
    color: #66f;
}
```

### 对齐输入占位

有强迫症的同学总会觉得输入框文本位置整体偏上，感觉未居中心里就痒痒的。`桌面端浏览器`里声明`line-height`等于`height`就能解决，但`移动端浏览器`里还是未能解决，需将`line-height`声明为`normal`才行。

```css
input {
    line-height: normal;
}
```

### 对齐下拉选项

下拉框选项默认向左对齐，是时候改改向右对齐了。

```css
select option {
    direction: rtl;
}
```

### 修复点击无效

在苹果系统上有些情况下非可点击元素监听`click事件`可能会无效，针对该情况只需对不触发`click事件`的元素声明`cursor:pointer`就能解决。

```css
.elem {
    cursor: pointer;
}
```

### 识别文本换行

多数情况会使用JS换行文本，那就真的`Out`了。若接口返回字段包含`\n`或`<br>`，千万别替换掉，可声明`white-space:pre-line`交由浏览器做断行处理。

```css
* {
    white-space: pre-line;
}
```

### 开启硬件加速

想动画更流畅吗，开启`GPU硬件加速`呗！

```css
.elem {
    transform: translate3d(0, 0, 0);
    /* transform: translateZ(0); */
}
```

### 描绘像素边框

万年话题，如何描绘`一像素边框`？

```scss
.elem {
    position: relative;
    width: 200px;
    height: 80px;
    &::after {
        position: absolute;
        left: 0;
        top: 0;
        border: 1px solid #f66;
        width: 200%;
        height: 200%;
        content: "";
        transform: scale(.5);
        transform-origin: left top;
    }
}
```

### 控制溢出文本

万年话题，如何控制文本做`单行溢出`和`多行溢出`？

```scss
.elem {
    width: 400px;
    line-height: 30px;
    font-size: 20px;
    &.sl-ellipsis {
        overflow: hidden;
        text-overflow: ellipsis;
        white-space: nowrap;
    }
    &.ml-ellipsis {
        display: -webkit-box;
        overflow: hidden;
        text-overflow: ellipsis;
        -webkit-line-clamp: 3;
        -webkit-box-orient: vertical;
    }
}
```