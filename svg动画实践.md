# svg动画实践

`svg` 动画，全称为 `SMIL` ([Synchronized Multimedia Integration Language](http://www.w3.org/TR/REC-smil))

兼容性相比 `svg` 还是比较差的，基本只适用于 `chrome` 和 `safari`，[caniuse](https://caniuse.com/#search=SMIL)，对于大多数场景够用了，毕竟（IE也要弃暗投 `webkit` 了）

## 一、基本实现

#### 1. 标签

因为是 `xml` 格式，动画也是通过标签形式定义。

共有以下几种标签：

- *set*：并没有中间态，直接过渡到终态，如几秒后移动位置

- *animate*：控制元素属性变化

- *animateMotion*: 控制元素按指定路径运动

- *animateTransform*: 控制元素的 `transform` 属性，如 `rotate`，`translate`等。

  _干嘛不把这个集成在 *animate*，反正要写一个 `attributeName="transfrom"`？_

#### 2. 使用方式有两种

1. 嵌入元素内部使用

   ```sv
   <rect width="50" height="50" x="10" y="10" fill="#c00">
     <set attributeName="x" to="40" begin="2s" />
   </rect>
   ```

2. 元素外引用

   ```svg
   <rect width="60" height="60" x="50" y="50" fill="#0c0" id="qq"></rect>
   <animate dur="2s" attributeName="x" repeatCount="indefinite" to="100" xlink:href="#qq"/>
   ```

#### 3. 属性

- *attributeName*: 动画变化的属性

- *attributeType*: 确定是 `css` 属性还是 `xml` 属性，如 `font-size`，默认 `auto`

- *dur*: 动画播放时间

- *begin*: 动画开始时间

  有几种值：

  1. 时间，例如：`begin = "1s"` ，`1s` 后开始动画

  2. *id.end; id.begin*，表示在`#id` 此动画开始时或结束时播放本动画，例如：

     ```svg
     <rect width="60" height="60" x="50" y="50" fill="#0cc">
        <animate id="a1" dur="2s" attributeName="x" to="100"/> 
        <animate dur="2s" attributeName="y" to="100" begin="a1.end"/>
         <!--a1结束的时候播放-->
     </rect>
     ```

  3. *id.event;event*，表示在某个事件后开始播放动画，例如：

     ```svg
     <rect id="qq" width="120" height="30" x="50" y="10" fill="#aaa"/>
     <rect width="60" height="60" x="50" y="50" fill="#0c0">
       <!--qq点击后才开始动画-->
       <animate dur="2s" attributeName="x" to="100" begin="qq.click"/>  
     </rect>
     ```

   4. *id.repeat*，表示 `#id` 动画重复几次后播放，例如：

      ```svg
      <rect width="60" height="30" x="50" y="50" fill="#c00">
        <animate id="r1"   dur="1s" attributeName="width" to="120" repeatCount="indefinite" />
        <animate dur="2s" attributeName="y" to="100" begin="r1.repeat(3)"/> 
      </rect>
      ```

  5. *indefinite*：一直等待，直到超链接控制，或者触发元素的`beginElement`事件，例如：

     ```svg
     <rect width="60" height="30" x="50" y="50" fill="#c00">  
       <animate id="a1" dur="2s" attributeName="y" to="100" begin="indefinite" repeatCount="indefinite"/> 
     </rect>
     <a xlink:href="#a1"><text x="50" y="30">開始動畫</text></a>
     ```

  6. *accessKey*: 按下哪个键盘的按键，例如：

     ```svg
     <text font-family="microsoft yahei" font-size="120" y="160" x="160">马
        <animate attributeName="x" to="60" begin="accessKey(s)" dur="3s" repeatCount="indefinite" />
     </text>
     ```

  7. *wallclock-sync-value*，确切时间值

- *end*: 同上，结束时间

- *from, to, by, values*：动画的变化范围

  - from = “**<value>**“

      动画的起始值。

  - to = “**<value>**“

      指定动画的结束值。

  - by = “**<value>**“

      动画的相对变化值。

  - values = “**<list>**“

      用分号分隔的一个或多个值，可以看出是动画的多个关键值点。

- *repeatCount, repeatDur*，重复次数或者时间

- *fill*，设置动画间隙怎么表达，可以设置为 `freeze`，这样动画结束后也会一直保持下去

- *calcMode, keyTimes, keySplines*，动画快慢变化，可以在参考文中了解详情

### 4. 暂停和继续

```
// 暂停
svg.pauseAnimations();

// 重启动
svg.unpauseAnimations()
```

## 二、stroke-dashoffset

这个属性一般配合 `stroke-dasharray` 使用，类似 `css` 中的边框表现：`border: 1px dashed #333`，用来表示线段和空白的比例。

一般可以用来做线段的长短变化动画，无所谓图形，只要是通过 `stroke` 来展示形状的元素。

例如：

```html
<!-- 
    半径为20, 周长为2πr, 约为125.6, 即stroke的长度为125.6
    分三个阶段来变化
    1. 实线长1，空白长200，此时offset为0，展示出长度1的线段在x轴之下
    2. 实线长89，空白长200，此时offset从0变道-35，一直在实线边长
    3. 实线空白分割同上，offset继续变长，实线变长，一直到-125，只余下长度1的线段在x轴之上
    4. 循环到1
-->
<svg viewBox="25 25 50 50">
    <circle
        class="loader-path"
        cx="50"
        cy="50"
        r="20"
        fill="none"
        stroke="#70c542"
        stroke-width="2"
        stroke-linecap="round">
        <animate
            attributeName="stroke-dasharray"
            values="1,200;89,200;89,200"
            dur="2s"
            calcMode="paced"
            repeatCount="indefinite">
        </animate>
        <animate
            attributeName="stroke-dashoffset"
            values="0;-35;-125"
            dur="2s"
            calcMode="paced"
            repeatCount="indefinite">
        </animate>
        <animateTransform
            attributeName="transform"
            type="rotate"
            dur="2s"
            from="0 50 50"
            to="360 50 50"
            repeatCount="indefinite">
        </animateTransform>
    </circle>
</svg>
```

仅以参考！

## 参考

1. [超级强大的SVG SMIL animation动画详解](https://www.zhangxinxu.com/wordpress/2014/08/so-powerful-svg-smil-animation/)
2. [纯CSS实现帅气的SVG路径描边动画效果](https://www.zhangxinxu.com/wordpress/2014/04/animateion-line-drawing-svg-path-%E5%8A%A8%E7%94%BB-%E8%B7%AF%E5%BE%84/)
3. [SVG 研究之路 (21) - 初探 SMIL Animation](https://www.oxxostudio.tw/articles/201409/svg-21-smil-animation.html)
4. [SVG Loading Circle Animation](https://codepen.io/aleksander351/pen/KzgKPo)