以下所有效果的实现方式均为个人见解，如有不对的地方还请一一指出。

### 目录
* 方块“Z”字形运动
* 线段围绕盒子运动
* 饼图[动图, 固定比例，如20%]
* 移动端录音旋转小按钮效果实现[渐变色][初始原型]

### 方块“Z”字形运动
要求：10s钟循环五次；每次的执行时间为2s；每一次执行到底部后反方向执行；最后停留在最后一帧

考察：animation基本属性值

![demo-1.png](http://static.mxnt.net/img/1500315-582e58f2c40d7c76.png)

[点击查看效果](https://junruchen.github.io/css/demo/css-animation-z-move.html)

重点代码说明：
```
  animation: move 2s linear 0s 5 alternate forwards;
```
| 参数      | 含义                                                                                                                                       |
| --------- | ------------------------------------------------------------------------------------------------------------------------------------------ |
| move      | animation-name 动画名称                                                                                                                    |
| 2s        | animation-duration 动画持续时间, 注意加上单位                                                                                              |
| linear    | animation-timing-function 动画过渡类型，linear：线性过渡                                                                                   |
| 0s        | animation-delay 动画延迟时间，注意加上单位                                                                                                 |
| 5         | animation-iteration-count 动画循环次数, infinite表示无限循环                                                                               |
| alternate | animation-direction 动画是否有反向运动, normal:正向运行; reverse:反向运行; alternate:先正向后反向; alternate-reverse:先反向后正向          |
| forwards  | animation-fill-mode 动画时间之外的状态, none: 不设置; forwards:设置为结束时的状态; backwords:设置为开始时的状态; both:动画结束或开始的状态 |

详细代码示例：
```
  dom结构：
  <div class="box">
    <div class="square"></div>
  </div>
  
  基本样式：
  .box{
    position: relative;
    width: 200px;
    height: 200px;
    background-color: #fb3;
  }
  .box .square{
    position: absolute;
    width: 30px;
    height: 30px;
    background-color: #58a; 
    animation: move 2s linear 0s 5 alternate forwards; //动画，这里要注意，时间必须带上单位
  }
  
  move动画定义
  @keyframes move {
    0% {
      transform: translateX(0);
    }
    33% {
      transform: translateX(170px);
    }
    66% {
      transform: translate(0, 170px);
    }
    100% {
      transform: translate(170px, 170px);
    }
  }
```

### 线段围绕盒子运动
要求：线段围绕盒子运动

考察：clip的用法

这个动画效果使用animation并不能很好的实现，最终实现方法是使用了 ```clip``` 属性，该属性用来剪裁对象。动画实现的原理是用line-box盒子覆盖住box盒子，line-box无背景只设置边框，然后使用clip进行剪裁。

clip语法：rect(number number number number)，依据上-右-下-左的顺序剪裁，如设置为auto则该边不剪裁。

![demo-2.png](http://static.mxnt.net/img/1500315-790af22f20bcb0f7.png)

[点击查看效果](https://junruchen.github.io/css/demo/css-animation-line-move.html)

代码示例：
```
  dom结构：
  <div class="box">
    <div class="line-box"></div>
  </div>
  
  基本样式：
  .box{
    position: relative;
    width: 200px;
    height: 200px;
    background-color: #fb3;
  }
  .box .line-box{
    position: absolute;
    width: 220px; //多出来的20像素为线段到盒子的边距
    height: 220px;
    left: 0;
    top: 0;
    margin-left: -10px;
    margin-top: -10px;
    border: 2px solid #58a;
    box-sizing: border-box;
    animation: move 5s linear infinite;
  }
  
  move动画定义
  @keyframes move {
    0% {
      clip: rect(0 220px 2px 0); //依据上-右-下-左的顺序剪裁
    }
    25% {
      clip: rect(0 0 220px 2px);
    }
    50% {
      clip: rect(2px 220px 0 0);
    }
    75% {
      clip: rect(0 2px 220px 0);
    }
  }
```
### 饼图

**动态饼图**

[点击查看效果](https://junruchen.github.io/css/demo/css-animation-pie-1.html)

实现方法：
* 首先绘制饼图基本形状，使用使用渐变色生成背景
代码：
```
.box {
      border-radius: 50%;
      background: linear-gradient(to right, #58a 50%, #fb3 0);
    }
```
效果：

![3.png](http://static.mxnt.net/img/1500315-fc146f7b8bd96683.png)

* 借助伪元素盖住右侧背景，且需要设置**圆角**( [Border详解](https://github.com/junruchen/junruchen.github.io/wiki/CSS-Border) )。
* 更改旋转中心
* 设置动画，动画在旋转到50%时，需更改伪元素的背景由蓝色变成黄色。

代码示例：
```
    .box:before {
      content: '';
      position: absolute;
      width: 50%;
      height: 100%;
      background-color: #58a;
      right: 0;
      border-radius: 0 100% 100% 0 / 50%;
      transform-origin: left center;
      animation: rotate 6s linear infinite, changeColor 12s step-end infinite;
    }

    @keyframes rotate{
      to{
        transform: rotate(180deg);
      }
    }
    @keyframes changeColor{
      50%{
        background-color: #fb3;
      }
    }
```

**任意大小饼图**

[预览效果](https://junruchen.github.io/css/demo/css-animation-pie-2.html)

如果知道animation-delay一个比较神奇的用法，那这个效果就很容易实现了。

**注意：一个负值的延时值是合法的，与0s的延迟类似，它意味着动画会立即开始播放，但会自动前进到延时值的绝对值处，就好像动画在过去已经播放了指定的时间一样。因此实际效果就是动画跳过指定时间而从中间开始播放了。**

所以我们只需增加伪元素的代码：
```
    .box:before {
      ...

      animation-delay: -3s;
      animation-play-state: paused;
    }
```

最终代码如下：
```
    .box {
      position: relative;
      width: 200px;
      height: 200px;
      margin: 0 auto;
      margin-top: 50px;
      border-radius: 50%;
      background: linear-gradient(to right, #58a 50%, #fb3 0);
      animation-delay: -7.5s;
    }

    .box:before {
      content: '';
      position: absolute;
      width: 50%;
      height: 100%;
      background-color: #58a;
      right: 0;
      border-radius: 0 100% 100% 0 / 50%;
      transform-origin: left center;
      animation: rotate 6s linear infinite, change 12s step-end infinite;
      animation-delay: inherit;
      animation-play-state: paused;
    }

    @keyframes rotate{
      to{
        transform: rotate(180deg);
      }
    }
    @keyframes change{
      50%{
        background-color: #fb3;
      }
    }
```


### 移动端录音旋转小按钮效果实现[渐变色]

类似移动端的点击录音按钮的效果，或者是loading的原始效果，[初始原型，有待优化]。

效果图：

![loading.png](http://static.mxnt.net/img/1500315-3e283ecf764fb9b7.png)

[预览效果](https://junruchen.github.io/css/demo/css-animation-loading.html)

实现方法[个人见解]：左侧与右侧分别控制，分别旋转。

dom结构：
```
<div class="box">
  <div class="item-left"></div>
  <div class="item-right"></div>
  <div class="item-center">60s</div> 
</div>
```
* item-right, 右侧需设置渐变色背景且为半圆形，选择一个灰色作为渐变色的中间点，这里选择的是#666，效果图如下：

![item-right.png](http://static.mxnt.net/img/1500315-cac7cfe2fbdf9126.png)

代码：
```
    .box .item-right {
      position: absolute;
      width: 50%;
      height: 100%;
      top: 0;
      right: 0;
      border-radius: 0 100% 100% 0/50%;  /*设置圆角以实现半圆效果*/
      background: linear-gradient(#fff, #666); /*渐变色背景*/
    }
```

* item-right, 使用伪元素覆盖住原本的颜色，这里需要注意的是，为了使伪元素更好的覆盖住item-right的颜色，可使用等比放大或者加阴影等方法，这里选择的是阴影。
```
    .box .item-right:before{
      content: '';
      position: absolute;
      width: 100%;
      height: 100%;
      top: 0;
      left: 0;
      border-radius: inherit;
      background-color: #fff;
      box-shadow: 3px 0 2px 2px #fff;
    }
```
* 左侧同理，但是要注意圆角和颜色需更改：
```
      border-radius: 100% 0 0 100%/50%;
      background: linear-gradient(#000, #666);
```
* item-center主要是实现中间的小圆，可在这个圆上进行多种操作：
```
    .box .item-center{
      position: absolute;
      width: 190px;
      height: 190px;
      border-radius: 50%;
      background-color: #fff;
      top: 5px;
      left: 5px;
      line-height: 190px;
    }
```
* 下面开始实现旋转。
* 右侧选中中心在左侧中间，而左侧的旋转中心在右侧中间，所以需要分别设置```transform-origin```属性。
* 右侧先开始旋转，第一个动画设置旋转动画，第二个动画则设置当旋转结束时，右侧伪元素的颜色为透明色，以便左侧旋转。
```
animation: rotate 30s linear infinite, changeColor 60s step-end infinite;
```
* 左侧旋转需设置延迟时间，右侧旋转结束方可开始。
```
animation: rotate 30s 30s linear infinite;
```

两个动画的代码：
```
    @keyframes rotate {
      to {
        transform: rotate(180deg);
      }
    }
    @keyframes changeColor{
      50%{
        background-color: transparent;
      }
    }
```
