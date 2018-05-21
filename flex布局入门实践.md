---
title: flex布局入门实践
date: 2018-05-17 19:26:42
tags: 
- css
- flex 
categories: css
---

最近抽时间阅读了阮一峰大神的《flex布局教程》，初窥门径，小有心得，不过还不是很不熟练，需要多多使用才能熟练掌握。依据实例篇的一些讲解，自己也尝试实现了骰子的布局。这里跟大家分享一下代码和笔记，觉得参照这个代码来初步理解flex布局还是很好的。

``` html
<!DOCTYPE html>
<html lang="en">

<head>
  <title>flex布局</title>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <!-- <link href="css/style.css" rel="stylesheet"> -->
  <style>
    .container {
      background: #000;
      height: 600px;
      width: 800px;
      display: flex;
      flex-wrap: wrap;
      align-content: space-around;
      justify-content: space-around;
      /* padding: 20px; */
    }

    .box {
      display: flex;
      background: #fff;
      width: 200px;
      height: 200px;
      border-radius: 10%;
      padding: 10px;
      /* margin: 20px; */
    }

    .item {
      background: #000;
      color: #fff;
      text-align: center;
      line-height: 60px;
      border-radius: 50%;
      width: 60px;
      height: 60px;
    }

    .column{
      /* 属性定义了在分配多余空间之前，项目占据的主轴空间（main size） */
      /* flex-basis: <length> | auto; */
      flex-basis: 100%;
      display: flex;
    }
    /* 1 */

    .box:nth-child(1) {
      /* 横向：左边|中间|右边 */
      /* justify-content: flex-start|center|flex-end;  */
      /* 纵向：上边|中间|下边 */
      /* align-items: flex-start|center|flex-end; */
      justify-content: center;
      align-items: center;
    }
    /* 2 */

    .box:nth-child(2) {
      /* 排列间隔：两端对齐|每个项目两侧的间隔相等 */
      /* justify-content: space-between|space-around; */
      justify-content: space-between;
    }

    .box:nth-child(2) .item:nth-child(2) {
      /* 属性允许单个项目有与其他项目不一样的对齐方式，可覆盖align-items属性 */
      /* align-self: auto | flex-start | flex-end | center; */
      align-self: flex-end;
    }
    /* 3 */

    .box:nth-child(3) {
      justify-content: space-between;
    }

    .box:nth-child(3) .item:nth-child(2) {
      align-self: center;
    }

    .box:nth-child(3) .item:nth-child(3) {
      align-self: flex-end;
    }
    /* 4 */

    .box:nth-child(4) {
      /* 属性定义，如果一条轴线排不下，如何换行 */
      /* flex-wrap: no-wrap|wrap|wrap-reverse; */
      /* 属性定义了多根轴线的对齐方式。如果项目只有一根轴线，该属性不起作用 */
      /* align-content: flex-start | flex-end | center | space-between | space-around | stretch; */
      flex-wrap: wrap;
      align-content: space-between;
    }

    .box:nth-child(4) .column {
      justify-content: space-between;
    }
    /* 5 */

    .box:nth-child(5) {
      flex-wrap: wrap;
      align-content: space-between;
    }
    .box:nth-child(5) .column{
      justify-content: space-between;
    }
    .box:nth-child(5) .column:nth-child(2){
      justify-content: center;
    }

    /* 6 */
    .box:nth-child(6) {
      /* 排列方向 */
      /* flex-direction: row|row-reverse|column|column-reverse; */
      flex-wrap: wrap;
      flex-direction: column;
      align-content: space-between;
      justify-content: space-between;
    }




  </style>
</head>

<body>
  <div class="container">
    <!-- 1 -->
    <div class="box">
      <span class="item">1</span>
    </div>
    <!-- 2 -->
    <div class="box">
      <span class="item">1</span>
      <span class="item">2</span>
    </div>
    <!-- 3 -->
    <div class="box">
      <span class="item">1</span>
      <span class="item">2</span>
      <span class="item">3</span>
    </div>
    <!-- 4 -->
    <div class="box">
      <div class="column">
        <span class="item">1</span>
        <span class="item">2</span>
      </div>
      <div class="column">
        <span class="item">3</span>
        <span class="item">4</span>
      </div>
    </div>
    <!-- 5 -->
    <div class="box">
      <div class="column">
        <span class="item">1</span>
        <span class="item">2</span>
      </div>
      <div class="column">
        <span class="item">3</span>
      </div>
      <div class="column">
        <span class="item">4</span>
        <span class="item">5</span>
      </div>
    </div>
    <!-- 6 -->
    <div class="box">
      <span class="item">1</span>
      <span class="item">2</span>
      <span class="item">3</span>
      <span class="item">4</span>
      <span class="item">5</span>
      <span class="item">6</span>
    </div>
  </div>

</body>

</html>

```

![image](http://upload-images.jianshu.io/upload_images/4050018-bd3f824fe20fe340.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
