# gcoord

[![npm version](https://img.shields.io/npm/v/gcoord.svg)](https://www.npmjs.com/package/gcoord)
[![Build Status](https://travis-ci.org/hujiulong/gcoord.svg?branch=master)](https://travis-ci.org/hujiulong/gcoord)
[![codecov](https://codecov.io/gh/hujiulong/gcoord/branch/master/graph/badge.svg)](https://codecov.io/gh/hujiulong/gcoord)
[![gzip size](http://img.badgesize.io/https://unpkg.com/gcoord/dist/gcoord.js?compression=gzip)](https://unpkg.com/gcoord/dist/gcoord.js)
[![LICENSE](https://img.shields.io/npm/l/vue.svg)](https://www.npmjs.com/package/gcoord)

gcoord( **g**eographic **coord**inates)是一个处理地理坐标的js库

它能够处理GeoJSON，能够在不同坐标系之间做转换

* 轻量，零依赖，gzip后大小仅3kb
* 兼容性强，能在node环境以及所有现代浏览器（IE8+）中运行
* 稳定高效，100%测试覆盖
* 支持转换GeoJSON

## 关于坐标系
我们通常用经纬度来表示一个地理位置，但是由于一些原因，我们从不同渠道得到的经纬度信息可能并不是在同一个坐标系下。

* 高德地图、腾讯地图以及谷歌中国区地图使用的是**GCJ-02**坐标系
* 百度地图使用的是**BD-09**坐标系
* 底层接口(HTML5 Geolocation或ios、安卓API)通过GPS设备获取的坐标使用的是**WGS-84**坐标系

不同的坐标系之间可能有几十到几百米的偏移，所以在开发基于地图的产品，或者做地理数据可视化时，我们需要修正不同坐标系之间的偏差。



### WGS-84 - 世界大地测量系统
WGS-84（World Geodetic System, WGS）是使用最广泛的坐标系，也是世界通用的坐标系，GPS设备得到的经纬度就是在WGS84坐标系下的经纬度。通常通过底层接口得到的定位信息都是WGS84坐标系。

### GCJ-02 - 国测局坐标
GCJ-02（G-Guojia国家，C-Cehui测绘，J-Ju局），又被称为火星坐标系，是一种基于WGS-84制定的大地测量系统，由中国国测局制定。此坐标系所采用的混淆算法会在经纬度中加入随机的偏移。

国家规定，**中国大陆所有公开地理数据都需要至少用GCJ-02进行加密**，也就是说我们从国内公司的产品中得到的数据，一定是经过了加密的。绝大部分国内互联网地图提供商都是使用GCJ-02坐标系，包括高德地图，谷歌地图中国区等。

> 导航电子地图在公开出版、销售、传播、展示和使用前，必须进行空间位置技术处理。<br> — GB 20263―2006《导航电子地图安全处理技术基本要求》，4.1

### BD-09 - 百度坐标系
BD-09（Baidu, BD）是百度地图使用的地理坐标系，其在GCJ-02上多增加了一次变换，用来保护用户隐私。从百度产品中得到的坐标都是BD-09坐标系。

<p align="center">
  <img src="./crs.jpg">
  <p align="center">不同坐标系下的点在百度地图上会有偏移</p>
</p>

### 相互转换
GCJ-02和BD-09都是用来对地理数据进行加密的，所以也不会公开逆向转换的方法。理论上，GCJ-02的加密过程是不可逆的，但是可以通过一些方法来逼近接原始坐标，并且这种方式的精度很高。gcoord使用的纠偏方式达到了厘米级的精度，能满足绝大多数情况。

## 安装
通过npm安装
```bash
npm install gcoord --save
```
或者直接下载[gcoord.js](https://unpkg.com/gcoord/dist/gcoord.js)

## 使用
例如从手机的GPS得到一个经纬度坐标，需要将其展示在百度地图上，则应该将当前坐标从WGS-84坐标系转换为BD-09坐标系
```js
var result = gcoord.transform(
    [ 116.403988, 39.914266 ],    // 经纬度坐标
    gcoord.WGS84,                 // 当前坐标系
    gcoord.BD09                   // 目标坐标系
);

console.log( result );  // [ 116.41661560068297, 39.92196580126834 ]
```
同时gcoord还可以生成GeoJSON以及转换GeoJSON的坐标系，详细使用方式可以参考[API](#api)

## API

* **坐标转换**
    * [transform](#transform-input-from-to-)
    * [CRS](#crs)
* **GeoJSON**
    * [feature](#feature-geometry-properties-options---)
    * [geometry](#geometry-type-coordinates-options--)
    * [point](#point-coordinates-properties-options---)
    * [points](#points-coordinates-properties-options---)
    * [polygon](#polygon-coordinates-properties-options---)
    * [polygons](#polygons-coordinates-properties-options---)
    * [lineString](#linestring-coordinates-properties-options---)
    * [lineStrings](#linestrings-coordinates-properties-options---)
    * [featureCollection](#featurecollection-features-options--)
    * [multiLineString](#multilinestring-coordinates-properties-options---)
    * [multiPoint](#multipoint-coordinates-properties-options---)
    * [multiPolygon](#multipolygon-coordinates-properties-options---)
    * [geometryCollection](#geometrycollection-geometries-properties-options---)

-------

### transform( input, from, to )
进行坐标转换

**参数**
-   `input` **[geojson][geojson] | [ string][string] | [ Array][Array]&lt;[number][number]>** geoJSON对象，或geoJSON字符串，或经纬度数组
-   `from` **[CRS][CRS]** 当前坐标系
-   `to` **[CRS][CRS]** 目标坐标系

**返回值**

**[geojson][geojson] | [ Array][Array]&lt;[number][number]>**

**示例**
```js
// 将GCJ02坐标转换为WGS84坐标
var result = gcoord.transform( [ 123, 45 ], gcoord.GCJ02, gcoord.WGS84 );
console.log( result );  // [ 122.99395597, 44.99804071 ]
```

```js
// 转换GeoJSON坐标
var geojson = {
    "type": "Point",
    "coordinates": [ 123, 45 ]
}
gcoord.transform( geojson, gcoord.GCJ02, gcoord.WGS84 );
console.log( geojson.coordinates ); // [ 122.99395597, 44.99804071 ]
```

```js
// 生成GeoJSON并转换坐标
var geojson = gcoord.point( [ 123, 45 ] );
gcoord.transform( geojson, gcoord.GCJ02, gcoord.WGS84 );
console.log( geojson.coordinates ); // [ 122.99395597, 44.99804071 ]
```
返回数组或GeoJSON对象（由输入决定），**注意：当输入为geojson时，transform会改变输入对象**

### CRS
CRS为坐标系，目标支持以下几种坐标系

| CRS              | 说明    |
| --------         | ----- |
| gcoord.WGS84     | WGS-84坐标系，GPS设备获取的经纬度坐标   |
| gcoord.GCJ02     | GCJ-02坐标系，google中国地图、soso地图、aliyun地图、mapabc地图和高德地图所用的经纬度坐标   |
| gcoord.BD09      | BD-O9坐标系，百度地图采用的经纬度坐标    |
| gcoord.WGS84     | WGS-84坐标系别名，同WGS-84  |
| gcoord.EPSG4326  | WGS-84坐标系别名，同WGS-84  |


### feature( geometry[, properties[, options ] ] )
生成一个 GeoJSON [Feature][Feature]

**参数**
-   `geometry` **[Geometry][Geometry]** 输入geometry
-   `properties` **[Object][Object]** 属性 (可选, 默认值 `{}`)
-   `options` **[Object][Object]** 选项 (可选, 默认值 `{}`)
    -   `options.bbox` **[Array][Array]&lt;[number][number]>** 外包围框 [west, south, east, north]
    -   `options.id` **([string][string] \| [number][number])** Feature的id

**返回值**

**[Feature][Feature]**

**示例**
```javascript
var geometry = {
    "type": "Point",
    "coordinates": [ 110, 50 ]
};

var feature = gcoord.feature( geometry );
```

### geometry( type, coordinates[, options ] )
生成一个GeoJSON [Geometry][Geometry]
如果需要创建GeometryCollection，可以使用`gcoord.geometryCollection`

**参数**
-   `type` **[string][string]** Geometry 类型
-   `coordinates` **[Array][Array]&lt;[number][number]>** 坐标
-   `options` **[Object][Object]** 选项 (可选, 默认值 `{}`)
    -   `options.bbox` **[Array][Array]&lt;[number][number]>** 外包围框 [west, south, east, north]

**返回值**

**[Geometry][Geometry]**

**示例**
```javascript
var type = 'Point';
var coordinates = [ 110, 50 ];

var geometry = gcoord.geometry( type, coordinates );
```

### point( coordinates[, properties[, options ] ] )
生成一个 [Point][Point][Feature][Feature]

**参数**
-   `coordinates` **[Array][Array]&lt;[number][number]>** 坐标
-   `properties` **[Object][Object]** 属性 (可选, 默认值 `{}`)
-   `options` **[Object][Object]** 选项 (可选, 默认值 `{}`)
    -   `options.bbox` **[Array][Array]&lt;[number][number]>** 外包围框 [west, south, east, north]
    -   `options.id` **([string][string] \| [number][number])** Feature的id

**返回值**

**[Feature][Feature]&lt;[Point][Point]>**

**示例**
```javascript
var point = gcoord.point([-75.343, 39.984]);

//=point
```

### points( coordinates[, properties[, options ] ] )

生成一个 [Point][Point][FeatureCollection][FeatureCollection]

**参数**
-   `coordinates` **[Array][Array]&lt;[Array][Array]&lt;[number][number]>>** 坐标
-   `properties` **[Object][Object]** 每个feature的属性 (可选, 默认值 `{}`)
-   `options` **[Object][Object]** 选项 (可选, 默认值 `{}`)
    -   `options.bbox` **[Array][Array]&lt;[number][number]>** 外包围框 [west, south, east, north]
    -   `options.id` **([string][string] \| [number][number])** FeatureCollection的id

**返回值**

**[FeatureCollection][FeatureCollection]&lt;[Point][Point]>**

**示例**
```javascript
var points = gcoord.points([
  [-75, 39],
  [-80, 45],
  [-78, 50]
]);

//=points
```

### polygon( coordinates[, properties[, options ] ] )

生成一个 [Polygon][Polygon] [Feature][Feature]

**参数**
-   `coordinates` **[Array][Array]&lt;[Array][Array]&lt;[Array][Array]&lt;[number][number]>>>** 坐标
-   `properties` **[Object][Object]** 属性 (可选, 默认值 `{}`)
-   `options` **[Object][Object]** 选项 (可选, 默认值 `{}`)
    -   `options.bbox` **[Array][Array]&lt;[number][number]>** 外包围框 [west, south, east, north]
    -   `options.id` **([string][string] \| [number][number])** Feature的id

**返回值**

**[Feature][Feature]&lt;[Polygon][Polygon]>**

**示例**
```javascript
var polygon = gcoord.polygon([[[-5, 52], [-4, 56], [-2, 51], [-7, 54], [-5, 52]]], { name: 'poly1' });

//=polygon
```

### polygons( coordinates[, properties[, options ] ] )

生成一个 [Polygon][Polygon] [FeatureCollection][FeatureCollection]

**参数**
-   `coordinates` **[Array][Array]&lt;[Array][Array]&lt;[Array][Array]&lt;[Array][Array]&lt;[number][number]>>>>** 坐标
-   `properties` **[Object][Object]** 属性 (可选, 默认值 `{}`)
-   `options` **[Object][Object]** 选项 (可选, 默认值 `{}`)
    -   `options.bbox` **[Array][Array]&lt;[number][number]>** 外包围框 [west, south, east, north]
    -   `options.id` **([string][string] \| [number][number])** FeatureCollection的id

**返回值**

**[FeatureCollection][FeatureCollection]&lt;[Polygon][Polygon]>**

**示例**

```javascript
var polygons = gcoord.polygons([
  [[[-5, 52], [-4, 56], [-2, 51], [-7, 54], [-5, 52]]],
  [[[-15, 42], [-14, 46], [-12, 41], [-17, 44], [-15, 42]]],
]);

//=polygons
```

### lineString( coordinates[, properties[, options ] ] )

生成一个 [LineString][LineString] [Feature][Feature]

**参数**
-   `coordinates` **[Array][Array]&lt;[Array][Array]&lt;[number][number]>>** 坐标
-   `properties` **[Object][Object]** 属性 (可选, 默认值 `{}`)
-   `options` **[Object][Object]** 选项 (可选, 默认值 `{}`)
    -   `options.bbox` **[Array][Array]&lt;[number][number]>** 外包围框 [west, south, east, north]
    -   `options.id` **([string][string] \| [number][number])** Feature的id

**返回值**

**[Feature][Feature]&lt;[LineString][LineString]>**

**示例**
```javascript
var linestring1 = gcoord.lineString([[-24, 63], [-23, 60], [-25, 65], [-20, 69]], {name: 'line 1'});
var linestring2 = gcoord.lineString([[-14, 43], [-13, 40], [-15, 45], [-10, 49]], {name: 'line 2'});

//=linestring1
//=linestring2
```

### lineStrings( coordinates[, properties[, options ] ] )

生成一个 [LineString][LineString] [FeatureCollection][FeatureCollection]

**参数**
-   `coordinates` **[Array][Array]&lt;[Array][Array]&lt;[number][number]>>** 坐标
-   `properties` **[Object][Object]** 属性 (可选, 默认值 `{}`)
-   `options` **[Object][Object]** 选项 (可选, 默认值 `{}`)
    -   `options.bbox` **[Array][Array]&lt;[number][number]>** 外包围框 [west, south, east, north]
    -   `options.id` **([string][string] \| [number][number])** FeatureCollection的id

**返回值**

**[FeatureCollection][FeatureCollection]&lt;[LineString][LineString]>**

**示例**
```javascript
var linestrings = gcoord.lineStrings([
  [[-24, 63], [-23, 60], [-25, 65], [-20, 69]],
  [[-14, 43], [-13, 40], [-15, 45], [-10, 49]]
]);

//=linestrings
```

### featureCollection( features[, options ] )

生成一个 [FeatureCollection][FeatureCollection].

**参数**
-   `features` **[Array][Array]&lt;[Feature][Feature]>** input features
-   `options` **[Object][Object]** 选项 (可选, 默认值 `{}`)
    -   `options.bbox` **[Array][Array]&lt;[number][number]>** 外包围框 [west, south, east, north]
    -   `options.id` **([string][string] \| [number][number])** Feature的id

**返回值**

**[FeatureCollection][FeatureCollection]**

**示例**
```javascript
var locationA = gcoord.point([-75.343, 39.984], {name: 'Location A'});
var locationB = gcoord.point([-75.833, 39.284], {name: 'Location B'});
var locationC = gcoord.point([-75.534, 39.123], {name: 'Location C'});

var collection = gcoord.featureCollection([
  locationA,
  locationB,
  locationC
]);

//=collection
```

### multiLineString( coordinates[, properties[, options ] ] )

生成一个 [Feature&lt;MultiLineString>](Feature<MultiLineString>)

**参数**
-   `coordinates` **[Array][Array]&lt;[Array][Array]&lt;[Array][Array]&lt;[number][number]>>>** 坐标
-   `properties` **[Object][Object]** 属性 (可选, 默认值 `{}`)
-   `options` **[Object][Object]** 选项 (可选, 默认值 `{}`)
    -   `options.bbox` **[Array][Array]&lt;[number][number]>** 外包围框 [west, south, east, north]
    -   `options.id` **([string][string] \| [number][number])** Feature的id

**返回值**

**[Feature][Feature]&lt;[MultiLineString][MultiLineString]>**

**示例**
```javascript
var multiLine = gcoord.multiLineString([[[0,0],[10,10]]]);

//=multiLine
```

### multiPoint( coordinates[, properties[, options ] ] )

生成一个 [Feature&lt;MultiPoint>](Feature<MultiPoint>)

**参数**
-   `coordinates` **[Array][Array]&lt;[Array][Array]&lt;[number][number]>>** 坐标
-   `properties` **[Object][Object]** 属性 (可选, 默认值 `{}`)
-   `options` **[Object][Object]** 选项 (可选, 默认值 `{}`)
    -   `options.bbox` **[Array][Array]&lt;[number][number]>** 外包围框 [west, south, east, north]
    -   `options.id` **([string][string] \| [number][number])** Feature的id

**返回值**

**[Feature][Feature]&lt;[MultiPoint][MultiPoint]>**

**示例**

```javascript
var multiPt = gcoord.multiPoint([[0,0],[10,10]]);

//=multiPt
```

### multiPolygon( coordinates[, properties[, options ] ] )

生成一个 [Feature&lt;MultiPolygon>](Feature<MultiPolygon>)

**参数**
-   `coordinates` **[Array][Array]&lt;[Array][Array]&lt;[Array][Array]&lt;[Array][Array]&lt;[number][number]>>>>** 坐标
-   `properties` **[Object][Object]** 属性 (可选, 默认值 `{}`)
-   `options` **[Object][Object]** 选项 (可选, 默认值 `{}`)
    -   `options.bbox` **[Array][Array]&lt;[number][number]>** 外包围框 [west, south, east, north]
    -   `options.id` **([string][string] \| [number][number])** Feature的id

**返回值**

**[Feature][Feature]&lt;[MultiPolygon][MultiPolygon]>**

**示例**

```javascript
var multiPoly = gcoord.multiPolygon([[[[0,0],[0,10],[10,10],[10,0],[0,0]]]]);

//=multiPoly
```

### geometryCollection( geometries[, properties[, options ] ] )

生成一个 [Feature&lt;GeometryCollection>](Feature<GeometryCollection>)

**参数**
-   `geometries` **[Array][Array]&lt;[Geometry][Geometry]>** 一个 GeoJSON Geometries数组
-   `properties` **[Object][Object]** 属性 (可选, 默认值 `{}`)
-   `options` **[Object][Object]** 选项 (可选, 默认值 `{}`)
    -   `options.bbox` **[Array][Array]&lt;[number][number]>** 外包围框 [west, south, east, north]
    -   `options.id` **([string][string] \| [number][number])** Feature的id

**返回值**

**[Feature][Feature]&lt;[GeometryCollection][GeometryCollection]>**

**示例**

```javascript
var pt = {
    "type": "Point",
      "coordinates": [100, 0]
    };
var line = {
    "type": "LineString",
    "coordinates": [ [101, 0], [102, 1] ]
  };
var collection = gcoord.geometryCollection([pt, line]);

//=collection
```

[CRS]: #CRS

[number]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Number
[string]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String
[Array]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array
[Object]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object
[Error]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Error

[geojson]: https://tools.ietf.org/html/rfc7946#page-6
[Feature]: https://tools.ietf.org/html/rfc7946#section-3.2
[FeatureCollection]: https://tools.ietf.org/html/rfc7946#section-3.3
[Geometry]: https://tools.ietf.org/html/rfc7946#section-3.1
[GeometryCollection]: https://tools.ietf.org/html/rfc7946#section-3.1.8
[Point]: https://tools.ietf.org/html/rfc7946#section-3.1.2
[Polygon]: https://tools.ietf.org/html/rfc7946#section-3.1.6
[LineString]: https://tools.ietf.org/html/rfc7946#section-3.1.4
[MultiPoint]: https://tools.ietf.org/html/rfc7946#section-3.1.3
[MultiPolygon]: https://tools.ietf.org/html/rfc7946#section-3.1.7
[MultiLineString]: https://tools.ietf.org/html/rfc7946#section-3.1.5

## LICENSE
MIT
