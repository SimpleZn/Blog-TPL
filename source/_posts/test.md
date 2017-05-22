---
title: 用css实现三角箭头
date: 2016-12-05
tags:
  - css
header-img: "css-arrow.jpg"
---


> 在开发中有时候会用到用CSS快速实现三角箭头，在此记录一下。本文将介绍如何用CSS写出三角箭头，以及会使用到的一些场景。原理比较简单，是利用border来进行一些设置。

### 三角形

当元素宽高为0，设置border宽度，其他边框为透明颜色时，可以形成一个三角形。

```html
<div class="demo1"></div>

<style type="text/css">
  .demo1 {
    width: 0;
    height: 0;
    border: 10px solid transparent;
    border-left-color: red;
  }
</style>
```
[CSS三角形demo](http://codepen.io/znsimple/pen/XNzLXv)

### 三角箭头

利用伪元素制造两个三角形并进行重叠，形成一个三角形箭头。

```html
<div class="demo2"></div>

<style type="text/css">
.demo2 {
  porition: relative;
}

.demo2:before, .demo2:after {
  border: 10px solid transparent;
  border-left: 10px solid #fff;
  width: 0;
  height: 0;
  position: absolute;
  top: 0;
  left: -2px;
  content: ' ';
}

.demo2:before {
  border-left-color: red;
  left: 0px;
}
</style>
```
[CSS三角箭头demo](http://codepen.io/znsimple/pen/woPVRa)

### 与矩形组合的提示框

```html
<div class="demo2"></div>

<style type="text/css">
.demo2 {
  position: relative;
  width: 100px;
  height: 100px;
  border: 2px solid red
}

.demo2:before, .demo2:after {
  border: 10px solid transparent;
  border-left: 10px solid #fff;
  width: 0;
  height: 0;
  position: absolute;
  top: 50%;
  right: -20px;
  content: ' ';
  transform: translateY(-50%);
}

.demo2:before {
  border-left-color: red;
  right: -22px;
}
</style>
```
[提示框](http://codepen.io/znsimple/pen/BQYXrv)