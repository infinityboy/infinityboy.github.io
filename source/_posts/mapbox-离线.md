---
title: mapbox 离线
date: 2019-08-03 10:19:35
tags: mapbox
categories: 
- web前端
---

>本篇博客仅以个人学习记录使用
>参考链接：https://zhuanlan.zhihu.com/p/30967394
>参考链接：https://jacelyn.fish/2018/10/19/mapbox-localization/


### 介绍

mapbox是一款webgis的地图产品，起初作为leaflet的延伸，在2d地图上，有着优雅的Api，相较于arcgis等老牌产品更加轻量（当然arcgis继承的功能也更多）。那么Mapbox GL则是作为基于WebGL技术在3D地图领域的又一产品，虽然此时也还有Cesium做的也已经比较完善，但是Mapbox GL仍然有着她自己的优势，比如她的风格高度可定制化，并且她也是开源的，同时也可以用于移动端，还和Uber的几款开源产品可以相继承，更好的展现出地理信息可视化的魅力。相较于传统的瓦片地图，Mapbox GL是支持矢量数据进行渲染的并且可以实时交互，不需要与服务端通信，便可更改地图的风格、同时也可以按图层配置样式。因为Mapbox GL使用的是pbf编码的格式数据，相比较于图片资源更小，如果是在我们没有合法的底图数据，并且对于地理地形或者行政区要求不严格的情景下，还可以使用OpenStreetMap(OSM)开源地图数据用于Mapbox GL的地图底图原始数据，用于地理信息可视化。这些地图底图的原始数据可以使用sqlite数据库存储，甚至是用于移动端的离线地图数据包，可能是基于这些因素最后Map box GL采用的MBTiles数据，也是sqlite存储导出导出的数据。

### 基本使用

虽然Mapbox 是开源产品， 但是如果想完整体验所有cool的功能需要配合使用mapbox的云平台，在云平台上可以直接处理地图风格及标记等，并且可以发布成在线服务，当然使用者也必须要注册一个accessToken。产品很好，体验也很好，也很方便，但是总是会因为一些不可描述的原因，并不能让我们这些劳苦大众爽歪歪的去使用她。。在某些场景下我们是只能在内网访问的，所以说一切的云云都将变成过眼云烟，但是这个东西真的很好啊，又可以解决我们现有的问题，这时就需要我们把她搬到我们的内网上。

### 愚公移山

要问愚公移山拢(总)共分几步？？ 我想应该是需要很多步。

首先，既然是要做地理信息可视化，我们就应该先有一份底图的原始数据也就是我们说pbf格式的数据，开篇已经说到了，在我们没有紧张的合法的地图数据时我们可以使用OSM的开源地图数据来实现我们野心的第一步。

#### 获取数据

想要获取OSM数据最直接的办法是去官网下载，当然更多的情况下我们手里是shp格式的数据，这时我们可以使用QGIS等工具将shp导出成geojson格式的文件，我们也可以使用ogr2ogr进行转换，并支持属性选择和坐标系统转换等功能。也可以使用GDAL命令行转换

```
//Homebrew安装方式
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)”;; 

//安装tippecanoe
brew install tippecanoe

```

```
/由于工具使用的GeoJson数据，所以先将相关的矢量文件转换为GeoJson文件
ogr2ogr -f GeoJson hechi4326.json hechi4326.shp

//将GeoJson文件转换为mbtiles文件
// -o 输出文件
// -z 裁剪层级
// -f 
// -n 图层说明
// -l 输出的图层名称
// xxx.json 原文件
tippecanoe -o hechi4326.mbtiles -z 16 -f -n 河池市各个镇 -l hechi hechi4326.json

```

#### 数据处理

当我们有了转换成功的geojson数据后，我们可以使用 Mapbox 开源瓦片数据处理工具 tippecanoe 转换 json 数据为 .mbtiles 格式。在这个过程中可进行独立图层合并、设置缩放范围和过滤属性等操作

```
tippecanoe -o geodata.mbtiles -z 18 -Z 13 -f -n geodata ~/road.geojson ~/water.geojson ~/sea.geojson;
```

#### 搭建服务器

根据官方的介绍我们可以使用nodejs来搭建我们的地图服务器，事实上是官方有支持更好的模块用来解析数据去渲染数据。这个包就是tilelive。

例如：
```
var express = require('express');
var http = require('http');
var app = express();
var tilelive = require('tilelive');
require('mbtiles').registerProtocols(tilelive);


// 放置的矢量瓦片数据位置
tilelive.load('mbtiles:///Users/lsw/Desktop/mapbox-server/hechi4326.mbtiles', function(err, source) {
   if (err) {
       throw err;
   }

   // 设置端口为7777
   app.set('port', 7777);

   app.use(function(req, res, next) {
       // 服务端要支持跨域，否则会出现跨域问题
       res.header("Access-Control-Allow-Origin", "*");
       res.header("Access-Control-Allow-Headers", "Origin, X-Requested-With, Content-Type, Accept");
       next();
   });

   //访问的url是：http://localhost:7777/tiles/{z}/{x}/{y}.pbf
   app.get(/^\/tiles\/(\d+)\/(\d+)\/(\d+).pbf$/, function(req, res) {
       var z = req.params[0];
       var x = req.params[1];
       var y = req.params[2];
       console.log('get tile %d, %d, %d', z, x, y);


       source.getTile(z, x, y, function(err, tile, headers) {
           if (err) {
               res.status(404)
               res.send(err.message);
               console.log(err.message);
           } else {
               res.set(headers);
               res.send(tile);
           }
       });
   });

   http.createServer(app).listen(app.get('port'), function() {
       console.log('Express server listening on port ' + app.get('port'));
   });
});
```

```
node server.js
```

这时一个地图服务器就搭建成功了。

PS: 若练此功亦无需自宫 -------------- 也有很多开源的支持Mbtiles的服务可以直接使用



