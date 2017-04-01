# problem
记录前端开发中遇到的各种问题，以及解决办法。

## 1.兼容性问题

(1)chrome浏览器下不支持字体小于12像素

(2)一些移动端设备不支持vedio和audio的自动播放

解决方法是先通过用户 touchstart 触碰，触发播放并暂停（音频开始加载，后面用 JS 再操作就没问题了）以下是代码:

```javascript
document.addEventListener('touchstart',function() {
    document.getElementsByTagName('audio')[0].play();
    document.getElementsByTagName('audio')[0].pause();
});
```

(3)ios系统下单击事件有300ms的延时

出现这个问题，是由于ios系统下有一个默认的双击放大页面(double tap to zoom)的方案，因此在检测到第一个用户tap事件后，会hold一段时间，若在此时间内又检测到新的tap，则判断为双击事件，反之则判断为单击事件，而这个延迟的时间就是300ms左右。

解决方案：FastClick 是 FT Labs 专门为解决移动端浏览器 300 毫秒点击延迟问题所开发的一个轻量级的库。FastClick的实现原理是在检测到touchend事件的时候，会通过DOM自定义事件立即出发模拟一个click事件，并把浏览器在300ms之后的click事件阻止掉。

(4)低版本浏览器不支持getElementByClassName

解决方法是重写一个getByClass()函数：

```javascript
function getByClass(obj,sClass){
    var aResult = [];
    if(obj.getElementsByClassName){
        aResult = obj.getElementsByClassName(sClass);
    }else{
        var aEle = obj.getElementsByTagName('*');
        for(var i=0;i<aEle.length;i++){
            var aClass = aEle[i].className.split(' ');
            if(findInArr(aClass,sClass)){   //调用自定义的findArr方法
                aResult.push(aEle[i]);
            }
        }
    }
    return aResult;
}
function findInArr(arr,sClass){
    for(var i=0;i<arr.length;i++){
        if(arr[i]==sClass){
            return true;
        }
    }
    return false;
}
```

(5)原生ajax中低版本ie不支持xmlhttprequest对象

解决方法是做一个判断，如果有xmlhttprequest方法，则调用，若没有，则改用ie浏览器的ActiveXobject方法：

```javascript
if(window.XMLHttpRequest){
  var oAjax=window.XMLHttpRequest
}else{
  var oAjax=new ActiveXObject('Microsoft.XMLHTTP');
}
```

## 2.H5项目常见问题及注意事项
H5页面窗口自动调整到设备宽度，并禁止用户缩放页面
```html
<!-- 1、HTML页面结构 -->
<meta name="viewport" content="width=device-width,initial-scale=1.0,minimum-scale=1.0,maximum-scale=1.0,user-scalable=no" />
<!-- width    设置viewport宽度，为一个正整数，或字符串‘device-width’
     height   设置viewport高度，一般设置了宽度，会自动解析出高度，可以不用设置
     initial-scale    默认缩放比例，为一个数字，可以带小数
     minimum-scale    允许用户最小缩放比例，为一个数字，可以带小数
     maximum-scale    允许用户最大缩放比例，为一个数字，可以带小数
     user-scalable    是否允许手动缩放 -->
```

```javascript
// 2、JS动态判断
var phoneWidth =  parseInt(window.screen.width);
var phoneScale = phoneWidth/640;
var ua = navigator.userAgent;
if (/Android (\d+\.\d+)/.test(ua)){
    var version = parseFloat(RegExp.$1);
    if(version>2.3){
        document.write('<meta name="viewport" content="width=640, minimum-scale = '+phoneScale+', maximum-scale = '+phoneScale+', target-densitydpi=device-dpi">');
    }else{
        document.write('<meta name="viewport" content="width=640, target-densitydpi=device-dpi">');
    }
} else {
    document.write('<meta name="viewport" content="width=640, user-scalable=no, target-densitydpi=device-dpi">');
}
```

H5空白页基本meta标签

```html
<!-- 设置缩放 -->
<meta name="viewport" content="width=device-width, initial-scale=1, user-scalable=no, minimal-ui" />
<!-- 可隐藏地址栏，仅针对IOS的Safari（注：IOS7.0版本以后，safari上已看不到效果） -->
<meta name="apple-mobile-web-app-capable" content="yes" />
<!-- 仅针对IOS的Safari顶端状态条的样式（可选default/black/black-translucent ） -->
<meta name="apple-mobile-web-app-status-bar-style" content="black" />
<!-- IOS中禁用将数字识别为电话号码/忽略Android平台中对邮箱地址的识别 -->
<meta name="format-detection"content="telephone=no, email=no" />
```

PC端基础meta标签

```html
<!-- 页面关键词-->
<meta name="keywords" content="your tags" />
<!-- 页面描述-->
<meta name="description" content="150 words" />
<!-- 搜索引擎索引方式：robotterms是一组使用逗号(,)分割的值，通常有如下几种取值：none，noindex，nofollow，all，index和follow。确保正确使用nofollow和noindex属性值。-->
<meta name="robots" content="index,follow" />
<!--
    all：文件将被检索，且页面上的链接可以被查询；
    none：文件将不被检索，且页面上的链接不可以被查询；
    index：文件将被检索；
    follow：页面上的链接可以被查询；
    noindex：文件将不被检索；
    nofollow：页面上的链接不可以被查询。
 -->
 
 <!-- 页面重定向和刷新：content内的数字代表时间（秒），既多少时间后刷新。如果加url,则会重定向到指定网页（搜索引擎能够自动检测，也很容易被引擎视作误导而受到惩罚）。-->
 <meta http-equiv="refresh" content="0;url=" />
 
```

页面缓存设置

```html
<!-- 清除缓存 -->
<meta http-equiv="pragma" content="no-cache">
<meta http-equiv="cache-control" content="no-cache">
<meta http-equiv="expires" content="0">  
```

其他meta标签

```html
<!-- 启用360浏览器的极速模式(webkit) -->
<meta name="renderer" content="webkit">
<!-- 避免IE使用兼容模式 -->
<meta http-equiv="X-UA-Compatible" content="IE=edge">
<!-- 针对手持设备优化，主要是针对一些老的不识别viewport的浏览器，比如黑莓 -->
<meta name="HandheldFriendly" content="true">
<!-- 微软的老式浏览器 -->
<meta name="MobileOptimized" content="320">
<!-- uc强制竖屏 -->
<meta name="screen-orientation" content="portrait">
<!-- QQ强制竖屏 -->
<meta name="x5-orientation" content="portrait">
<!-- UC强制全屏 -->
<meta name="full-screen" content="yes">
<!-- QQ强制全屏 -->
<meta name="x5-fullscreen" content="true">
<!-- UC应用模式 -->
<meta name="browsermode" content="application">
<!-- QQ应用模式 -->
<meta name="x5-page-mode" content="app">
<!-- windows phone 点击无高光 -->
<meta name="msapplication-tap-highlight" content="no">

<meta name="author" content="author name" /> <!-- 定义网页作者 -->
<meta name="google" content="index,follow" />
<meta name="googlebot" content="index,follow" />
<meta name="verify" content="index,follow" />
```
