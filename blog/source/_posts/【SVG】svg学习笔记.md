---
title: 【SVG】svg学习笔记
tags: [svg]
date: 2017-2-23
---
近期由于要在页面上进行用户自定义绘制操作，需要学习下SVG的相关文档

## 1、在HTML中使用SVG
### HTML中直接加入
```html
<!--画一个 半径为40的圆cx 和 cy 属性定义圆中心的 x 和 y 坐标。
如果忽略这两个属性，那么圆点会被设置为 (0, 0)。-->
<svg xmlns="http://www.w3.org/2000/svg" version="1.1">
   <circle cx="100" cy="50" r="40" stroke="black" stroke-width="2" fill="red" />
</svg>
```
### HTML中连接加入
链接引入.svg文件
```xml
<?xml version="1.0" standalone="no"?>
<!DOCTYPE svg PUBLIC "-//W3C//DTD SVG 1.1//EN"
"http://www.w3.org/Graphics/SVG/1.1/DTD/svg11.dtd">
<svg xmlns="http://www.w3.org/2000/svg" version="1.1">
  <circle cx="100" cy="50" r="40" stroke="black"
  stroke-width="2" fill="red" />
</svg>
```
### SVG预设形状
SVG有一些预定义的形状元素，可被开发者使用和操作：
* 矩形 `<rect>`
* 圆形 `<circle>`
* 椭圆 `<ellipse>`
* 线 `<line>`
* 折线 `<polyline>`
* 多边形 `<polygon>`
* 路径 `<path>`

### (1)矩形-`<rect>`
* rect 元素的 width 和 height 属性可定义矩形的高度和宽度
* style 属性用来定义 CSS 属性
CSS 的 fill 属性定义矩形的填充颜色（rgb 值、颜色名或者十六进制值）
* CSS 的 stroke-width 属性定义矩形边框的宽度
* CSS 的 stroke 属性定义矩形边框的颜色

```html
<svg xmlns="http://www.w3.org/2000/svg" version="1.1">
  <rect width="300" height="100"
  style="fill:rgb(0,0,255);stroke-width:1;stroke:rgb(0,0,0)"/>
</svg>
```
* x 属性定义矩形的左侧位置（例如，x="0" 定义矩形到浏览器窗口左侧的距离是 0px）
* y 属性定义矩形的顶端位置（例如，y="0" 定义矩形到浏览器窗口顶端的距离是 0px）
* CSS 的 fill-opacity 属性定义填充颜色透明度（合法的范围是：0 - 1）
* CSS 的 stroke-opacity 属性定义笔触颜色的透明度（合法的范围是：0 - 1）
* CSS opacity 属性用于定义了元素的透明值 (范围: 0 到 1)。

```html
<svg xmlns="http://www.w3.org/2000/svg" version="1.1">
  <rect x="50" y="20" width="150" height="150"
  style="fill:blue;stroke:pink;stroke-width:5;fill-opacity:0.1;
  stroke-opacity:0.9"/>
</svg>
```

圆形矩阵
* rx 和 ry 属性可使矩形产生圆角。

```html
<svg xmlns="http://www.w3.org/2000/svg" version="1.1">
  <rect x="20" y="10" rx="10" ry="10" width="100" height="100" style="fill:red;stroke:black;stroke-width:5;opacity:0.5" />
</svg>
```

### (2)圆形-`<circle>`
* cx和cy属性定义圆点的x和y坐标。如果省略cx和cy，圆的中心会被设置为(0, 0)
* r属性定义圆的半径

```html
<svg xmlns="http://www.w3.org/2000/svg" version="1.1">
  <circle cx="100" cy="50" r="40" stroke="black"
  stroke-width="2" fill="red"/>
</svg>
```
### (3)椭圆 - `<ellipse>`
* CX属性定义的椭圆中心的x坐标
* CY属性定义的椭圆中心的y坐标
* RX属性定义的水平半径
* RY属性定义的垂直半径

```html
<svg xmlns="http://www.w3.org/2000/svg" version="1.1">
  <ellipse cx="300" cy="80" rx="100" ry="50"
  style="fill:yellow;stroke:purple;stroke-width:2"/>
</svg>
```

三个累叠而上的椭圆
```html
<svg xmlns="http://www.w3.org/2000/svg" version="1.1">
  <ellipse cx="240" cy="100" rx="220" ry="30" style="fill:purple"/>
  <ellipse cx="220" cy="70" rx="190" ry="20" style="fill:lime"/>
  <ellipse cx="210" cy="45" rx="170" ry="15" style="fill:yellow"/>
</svg>
```