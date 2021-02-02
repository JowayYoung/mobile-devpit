# HTML方向

### 调用系统功能

使用`<a>`能快速调用移动设备的`电话/短信/邮件`三大通讯功能，使用`<input>`能快速调用移动设备的的`图库/文件`。

这些功能方便了页面与系统的交互，关键在于调用格式一定要准确，否则会被`移动端浏览器`忽略。

```html
<!-- 拨打电话 -->
<a href="tel:10086">拨打电话给10086小姐姐</a>

<!-- 发送短信 -->
<a href="sms:10086">发送短信给10086小姐姐</a>

<!-- 发送邮件 -->
<a href="mailto:young.joway@aliyun.com">发送邮件给JowayYoung</a>

<!-- 选择照片或拍摄照片 -->
<input type="file" accept="image/*">

<!-- 选择视频或拍摄视频 -->
<input type="file" accept="video/*">

<!-- 多选文件 -->
<input type="file" multiple>
```

### 忽略自动识别

有些`移动端浏览器`会自动将数字字母符号识别为`电话/邮箱`并将其渲染成上述**调用系统功能**里的`<a>`。虽然很方便却有可能违背需求。

```html
<!-- 忽略自动识别电话 -->
<meta name="format-detection" content="telephone=no">

<!-- 忽略自动识别邮箱 -->
<meta name="format-detection" content="email=no">

<!-- 忽略自动识别电话和邮箱 -->
<meta name="format-detection" content="telephone=no, email=no">
```

### 弹出数字键盘

使用`<input type="tel">`弹起数字键盘会带上`#`和`*`，适合输入电话。推荐使用`<input type="number" pattern="\d*">`弹起数字键盘，适合输入验证码等纯数字格式。

```html
<!-- 纯数字带#和* -->
<input type="tel">

<!-- 纯数字 -->
<input type="number" pattern="\d*">
```

### 唤醒原生应用

通过`location.href`与原生应用建立通讯渠道，这种页面与客户端的通讯方式称为**URL Scheme**，其基本格式为`scheme://[path][?query]`，笔者曾经发表过[《H5与App的通讯方式》](https://juejin.cn/post/6844904020201455624)讲述`URL Scheme`的使用。

- **scheme**：应用标识，表示应用在系统里的唯一标识
- **path**：应用行为，表示应用某个页面或功能
- **query**：应用参数，表示应用页面或应用功能所需的条件参数

`URL Scheme`一般由前端与客户端共同协商。唤醒原生应用的前提是必须在移动设备里安装了该应用，有些`移动端浏览器`即使安装了该应用也无法唤醒原生应用，因为它认为`URL Scheme`是一种潜在的危险行为而禁用它，像`Safari`和`微信浏览器`。还好`微信浏览器`可开启白名单让`URL Scheme`有效。

若在页面引用第三方原生应用的`URL Schema`，可通过抓包第三方原生应用获取其`URL`。

```html
<!-- 打开微信 -->
<a href="weixin://">打开微信</a>

<!-- 打开支付宝 -->
<a href="alipays://">打开支付宝</a>

<!-- 打开支付宝的扫一扫 -->
<a href="alipays://platformapi/startapp?saId=10000007">打开支付宝的扫一扫</a>

<!-- 打开支付宝的蚂蚁森林 -->
<a href="alipays://platformapi/startapp?appId=60000002">打开支付宝的蚂蚁森林</a>
```

### 禁止页面缩放

在智能手机的普及下，很多网站都具备`桌面端`和`移动端`两种浏览版本，因此无需双击缩放查看页面。禁止页面缩放可保障`移动端浏览器`能无遗漏地展现页面所有布局。

```html
<meta name="viewport" content="width=device-width, user-scalable=no, initial-scale=1, minimum-scale=1, maximum-scale=1">
```

### 禁止页面缓存

**Cache-Control**指定请求和响应遵循的缓存机制，不想使用浏览器缓存就禁止呗！

```html
<meta http-equiv="Cache-Control" content="no-cache">
```

### 禁止字母大写

有时在输入框里输入文本会默认开启首字母大写纠正，就是输入首字母小写会被自动纠正成大写，特么的烦。直接声明`autocapitalize=off`关闭首字母大写功能和`autocorrect=off`关闭纠正功能。

```html
<input autocapitalize="off" autocorrect="off">
```

### 针对Safari配置

贴一些`Safari`较零散且少用的配置。

```html
<!-- 设置Safari全屏，在iOS7+无效 -->
<meta name="apple-mobile-web-app-capable" content="yes">

<!-- 改变Safari状态栏样式，可选default/black/black-translucent，需在上述全屏模式下才有效 -->
<meta name="apple-mobile-web-app-status-bar-style" content="black">

<!-- 添加页面启动占位图 -->
<link rel="apple-touch-startup-image" href="pig.jpg" media="(device-width: 375px)">

<!-- 保存网站到桌面时添加图标 -->
<link rel="apple-touch-icon" sizes="76x76" href="pig.jpg">

<!-- 保存网站到桌面时添加图标且清除默认光泽 -->
<link rel="apple-touch-icon-precomposed" href="pig.jpg">
```

### 针对其他浏览器配置

贴一些其他浏览器较零散且少用的配置，主要是常用的`QQ浏览器`、`UC浏览器`和`360浏览器`。从网易MTL的测试数据得知，新版的`QQ浏览器`和`UC浏览器`已不支持以下`<meta>`声明了。

```html
<!-- 强制QQ浏览器竖屏 -->
<meta name="x5-orientation" content="portrait">

<!-- 强制QQ浏览器全屏 -->
<meta name="x5-fullscreen" content="true">

<!-- 开启QQ浏览器应用模式 -->
<meta name="x5-page-mode" content="app">

<!-- 强制UC浏览器竖屏 -->
<meta name="screen-orientation" content="portrait">

<!-- 强制UC浏览器全屏 -->
<meta name="full-screen" content="yes">

<!-- 开启UC浏览器应用模式 -->
<meta name="browsermode" content="application">

<!-- 开启360浏览器极速模式 -->
<meta name="renderer" content="webkit">
```

### 让:active有效，让:hover无效

有些元素的`:active`可能会无效，而元素的`:hover`在点击后会一直处于点击状态，需点击其他位置才能解除点击状态。给`<body>`注册一个空的`touchstart事件`可将两种状态反转。

```html
<body ontouchstart></body>
```