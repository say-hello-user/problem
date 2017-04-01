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

