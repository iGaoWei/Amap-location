# 基于高德地图JsAPI进行浏览器精确定位，实现手机端考勤打卡功能

版权声明：本文为博主原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接和本声明。
本文链接：https://blog.csdn.net/qq_31766533/article/details/102639681

# 前言：

由于项目需求需要在项目中实现手机端（基于网页）考勤打卡功能，最初考虑使用H5自身定位功能，但尝试过后，效果很不稳定。然后尝试使用百度地图JsAPI，百度家的稳定倒是很稳定，没想到的是定位位置和实际位置居然相差几十公里，一开始是以为自己配置有问题，浪费了我大半天时间去找原因，最后发现他本身提供的API就是偏差很大距离的，他自己家的倒是定位很准，对外开放的API简直惨不忍睹。

[百度API浏览器定位](https://www.eiun.net/tools/baidumap.html)

[高德API浏览器定位](https://www.eiun.net/tools/amap.html)

然后换用高德去测试，高德开放的API精确度和百度地图是一样的，小伙伴可以亲自去体验下，难怪百度如今沦落到这样。。。

所以就决定使用高德API来进行定位了；

主要思路：利用高德API获取当前位置经纬度、设置考勤点经纬度、计算两点距离判断是否在考勤范围内。

高德JS API提供的浏览器定位接口，融合了HTML5 Geolocation定位接口、精确IP定位服务，以及安卓定位sdk定位。所以在定位上大大提高了精准度以及成功率。

效果如下：

![](https://img-blog.csdnimg.cn/20191021111139825.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzMxNzY2NTMz,size_16,color_FFFFFF,t_70)

# 正文：

首先注册账号并申请Key

1. 首先，注册开发者账号，成为高德开放平台开发者

2. 登陆之后，在进入「应用管理」 页面「创建新应用」

3. 为应用添加 Key，「服务平台」一项请选择「 Web 端 ( JSAPI ) 」

准备页面
1. 在页面添加 JS API 的入口脚本标签，并将其中「您申请的key值」替换为您刚刚申请的 key；

HTML

 `<script type="text/javascript" src="https://webapi.amap.com/maps?v=1.4.15&key=您申请的key值"></script>`

2. 添加div标签作为地图容器，同时为该div指定id属性；

HTML

`<div id="container"></div> `

3. 为地图容器指定高度、宽度；

CSS

#container {width:300px; height: 180px; }  

4. 进行移动端开发时，请在head内添加viewport设置，以达到最佳的绘制性能；

HTML

`<meta name="viewport" content="initial-scale=1.0, user-scalable=no"> `


5. 在完成如上准备工作之后便可以开始进行开发工作了。

### 显示定位地图以及获取当前经纬度地址

`<!doctype html>
<html>
<head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="initial-scale=1.0, user-scalable=no, width=device-width">
    <title>浏览器精确定位</title>
      <link rel="stylesheet" href="https://a.amap.com/jsapi_demos/static/demo-center/css/demo-center.css" />
    <style>
        html,body,#container{
            height:100%;
        }
        .info{
            width:26rem;
        }
    </style>
<body>
<div id='container'></div>
<div class="info">
    <h4 id='status'></h4><hr>
    <p id='result'></p><hr>
    <p >由于众多浏览器已不再支持非安全域的定位请求，为保位成功率和精度，请升级您的站点到HTTPS。</p>
</div>
<script type="text/javascript" src="https://webapi.amap.com/maps?v=1.4.15&key=您申请的key值"></script>
<script type="text/javascript">
    var map = new AMap.Map('container', {
        resizeEnable: true
    });
    AMap.plugin('AMap.Geolocation', function() {
        var geolocation = new AMap.Geolocation({
            enableHighAccuracy: true,//是否使用高精度定位，默认:true
            timeout: 10000,          //超过10秒后停止定位，默认：5s
            buttonPosition:'RB',    //定位按钮的停靠位置
            buttonOffset: new AMap.Pixel(10, 20),//定位按钮与设置的停靠位置的偏移量，默认：Pixel(10, 20)
            zoomToAccuracy: true,   //定位成功后是否自动调整地图视野到定位点

```html
    });
    map.addControl(geolocation);
    geolocation.getCurrentPosition(function(status,result){
        if(status=='complete'){
            onComplete(result)
        }else{
            onError(result)
        }
    });
});
//解析定位结果
function onComplete(data) {
    document.getElementById('status').innerHTML='定位成功'
    var str = [];
    str.push('定位结果：' + data.position);
    str.push('定位类别：' + data.location_type);
    if(data.accuracy){
         str.push('精度：' + data.accuracy + ' 米');
    }//如为IP精确定位结果则没有精度信息
    str.push('是否经过偏移：' + (data.isConverted ? '是' : '否'));
    document.getElementById('result').innerHTML = str.join('<br>');
}
//解析定位错误信息
function onError(data) {
    document.getElementById('status').innerHTML='定位失败'
    document.getElementById('result').innerHTML = '失败原因排查信息:'+data.message;
}
```
</script>
</body>
</html>`

### 计算当前位置与考勤点距离

`var signzone = [121.52625, 31.66925];//设置的签到点
console.log(signzone);
//计算当前位置与考勤点距离
var distance = AMap.GeometryUtil.distance(getposition,signzone).toFixed(2);

//document.getElementById('distance').innerHTML = distancestr;
console.log(distance);
			
if (distance <= 1000) {
//在范围内
    //数据操作
} else {
//不在范围内
    //数据操作
}`

### 绘制签到点范围 

//绘制签到范围

var circle = new AMap.Circle({
	center: signzone,
	radius: 100, //签到范围半径
	borderWeight: 1,
	strokeOpacity: 1,

```javascript
strokeOpacity: 0.2,
fillOpacity: 0.4,
```
})

circle.setMap(map)
// 缩放地图到合适的视野级别
map.setFitView([ circle ])

var circleEditor = new AMap.CircleEditor(map, circle)


到这里定位打卡的基本功能就完成了，然后再加上一些判断，比如用户是否进入考勤范围这些等等，配合上后端数据操作就可以实现该需求了。

![](https://img-blog.csdnimg.cn/20191021143323149.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzMxNzY2NTMz,size_16,color_FFFFFF,t_70)

最终页面效果如下：

![](https://img-blog.csdnimg.cn/20191021145834484.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzMxNzY2NTMz,size_16,color_FFFFFF,t_70)

# 一些注意事项

定位一般分为两种场景：移动端和PC，下面分别讲下这两个场景在使用定位过程中的一些注意事项。

移动端

移动端包括手机，pad和其它带有GPS定位芯片的智能设备（如手表、音箱等），移动端的系统包括iOS和Android。成功完成定位需要达成以下前提条件：

系统GPS打开

所使用的App或浏览器已获取定位权限

对打开的页面允许使用定位

对于iOS10以上系统和Android的一些版本已禁止在非HTTPS协议的域名下定位，请尽快将站点升级到HTTPS

注意，以上只是定位成功的前提条件，满足这些并不一定等于可以成功定位，定位还与当前位置（室内会影响GPS信息）、手机信号和定位权限等因素影响。如果您在使用过程中定位失败，可以参考FAQ：Geolocation的定位流程以及定位失败的原因 ，将失败信息通过工单发送给我们，高德的工程师将协助您解决问题。

PC

因为pc设备上大都缺少GPS芯片，所以在PC上的定位主要通过IP精准定位服务，该服务的失败率在5%左右。

定位失败

如果定位失败或者遇到其它问题，请参考FAQ：Geolocation的定位流程以及定位失败的原因 

原文链接：

## 声明

本资料仅供参考，水平有限，难免存在纰漏错误之处

欢迎转载，转载请标明来源出处，谢谢~~

作者：折腾不止的追梦人：Dream@eiun.net

链接：https://blog.csdn.net/qq_31766533/article/details/102639681