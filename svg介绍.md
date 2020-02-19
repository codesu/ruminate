# svg介绍

`svg `全称可缩放矢量图(`Scalable Vector Graphics`)。基于`xml`格式，标准制定于1999年。

因为是**矢量图**，通过 `xml`元素声明元素为之大小方向，缩放图片时，坐标系也跟着缩放，而声明好的元素不变，所以图片不会失真。

相对的，**位图**对每一个像素点都有描述，缩放的时候像素点扩大展示，多出来像素点没有对应描述，所以图片会缩放失真。

优点：

- 标签方式书写，即使不用了也不会忘记，上手也很快。`canvas` 中的各种绘图 `api`，用起来好用，但是忘得也快。
- 兼容性好[caniuse](https://caniuse.com/#feat=svg)，基本现有浏览器都支持
- 方便的响应鼠标事件，直接 `addEventListen` 即可，不需要像 `canvas` 那样获取坐标值，判断当前图形。

缺点：

- 元素过多时，`svg` 图片大小会超过位图，而且 `dom `元素超多，影响渲染


## 一、`namespace`与 `dtd`

既然是 `xml` 格式，那少不了 `dtd` 的声明了，当然这个节点可以省略，而且[相关回答](https://stackoverflow.com/questions/38169475/svg-in-html5-when-is-xml-declaration-xml-version-1-0-encoding-utf-8)也建议省略。

- `dtd`: 用来约束 `xml` 的格式与父子关系等，规定了这个 `xml` 文件只能用哪些标签
- `namespace`: 因为 `xml` 文件之间，可能有相同的标签名，属性，但是要求的解析方式又不同。例如：`svg` 中图片的 `href` 与 `html` 的 `a `链接的 `href` 属性，需要不同的解析方式，所以加上 `namespace`。

#### 使用方式

##### 1. `namespace`

在 `svg` 标签属性中添加`xmlns`属性，`xmlns`代表默认命名空间，`xmlns:xxx`代表其他命名空间，这两个属性值都为 `url`。使用命名空间的时候，在属性前加上此命名空间即可，没有命名空间的为默认命名空间。例如：

使用 `xlink` 命名空间：

```xml
<svg xmlns:xlink="http://www.w3.org/1991/xlink">
    <image xlink:href="./image.png"></image>
</svg>
```

使用 `URI` 来作为命名空间的值，并不会请求它，只是因为 `URI` 是唯一字符串，才这么使用（试了瞎填也可以用>_<）。

##### 2. `dtd`

这个约束方式是放在 `xml` 声明节点上的，可以省略这个约束声明。使用方法有两种：

1. 内部声明，直接把 `dtd` 文件嵌入 `svg` 中，一般不使用这种办法
2. 外部声明，引用的是外部 `dtd` 文件

##### 3. `scheme`

`dtd` 语法格式并不是 `xml` 语法格式，所以现在一般推荐 `scheme` 写法，更简单易读，灵活，自身也可以使用命名空间，不赘述。

## 二、标签简介

形状可以通过标签元素直接生成，然后在标签中定义属性，也可以通过 `css` 取控制，毕竟也是一个 `html` 元素，可以响应点击事件等。

还可以在 `<def>` (不用于展示)标签中，定义一些渐变，蒙层，通过 `id`  在后面的形状中展示。

#### 形状

- 矩形 `<rect>`
- 圆形 `<circle>`
- 椭圆 `<ellipse>`
- 线 `<line>`
- 折线` <polyline>`
- 多边形 `<polygon>`
- 路径` <path>`

#### 属性

- 宽高：width 和 height
- 定义 CSS 属性： style
- 填充颜色：fill，可以为色值，也可以为蒙层，渐变等标签的id
- 边框的宽度： stroke-width
- 边框的颜色: stroke

## 三、坐标系

左上角为(0, 0)的坐标，`y` 轴向下为正，`x` 轴向右为正，就是要这种 `html` 腔。

`svg` 中涉及坐标的有几方面：

1. `viewbox`
2. `transform`
3. `gradientUnits`, `patternUnits`...
4.  等...

**`viewbox`** 代表视区大小，在完整的 `svg` 中，只展示这么大的一个区域，并且配合 `preserveAspectRatio` 来定义宽高的对齐方式。例如：

```svg
<svg viewbox="0 0 250 200" width="250px">
    <rect fill="#0c0" width="300" height="300"></rect>
</svg>
```

上面的 `viewbox` 就代表，从(0,0) 开始，展示宽高为(250, 200)的区域。`width`为了控制下宽度，可以任意填，内部比例还是相对的。可以鼠标悬浮上去，看看 `rect` 原本大小。

**`transform`**属性，可以对标签进行变形。类似 `css` 的属性写法，如 `rotate(30)` 以左上角为圆心，旋转30度；如 `translate(0, 150)` 代表向下移动150的距离。

进行了变换后，后续的变换都是基于已经变换的自身坐标系进行的。例如：

```svg
<svg width="700" height="700">
    <rect width="200" height="100" fill="none" stroke="#333" stroke-width="1" transform="rotate(45)"></rect>
    <rect width="200" height="100" fill="none" stroke="#333" stroke-width="1"></rect>
    <g transform="translate(0, 150)">
        <rect width="200" height="100" fill="#0c0"></rect>
    </g>
</svg>
```

上例中，前两个 `rect` 向下旋转，第三个 `rect` 是放在向下位移后的 `group` 中，所以它在位移后的坐标系里。

**`gradientUnits`** 表示使用的坐标系为应用元素的坐标系，例如：

```svg
<defs>
   <linearGradient id="gr_oExzxkpmkJ" spreadMethod="pad" gradientUnits="userSpaceOnUse" x1="23" y1="0" x2="-91" y2="117">
       <stop offset="0%" stop-color="rgb(10,219,255)"></stop>
       <stop offset="50%" stop-color="rgb(9,165,255)"></stop>
       <stop offset="100%" stop-color="rgb(8,111,255)"></stop>
    </linearGradient>
</defs>
<g transform="matrix(1.20564,0,0,1.20564, 277.11500000000001,550)">
  <circle cx="6" cy="-4" r="151" fill="none" stroke="url(#gr_oExzxkpmkJ)" stroke-width="3" stroke-dasharray="949"></circle>
</g>
```

上面就表示，线性渐变的起始结束坐标，是对于使用它的元素坐标而言的。

## 四、[MARTIX变化][matrix]

这个部分就是上面提到的 `transform` 属性的一个属性值。`martix` 矩阵变换，其实就是上面的 `rotate`, `translate` 等变化方式的详细写法。那既然已经有简略写法了，可以实现了，为什么还要这种写法呢？

因为这种写法更统一，也便于多种变化的计算。

如位移的变化是：`translate(50, 50)`，可以写作 `matrix(1, 0, 1, 0, 50 ,50)`

旋转30度的写法是：`rotate(30)`，可以写作 `matrix(0.866, 0.5, -0.5, 0.866, 0, 0)`

`matrix(a, b, c, d, e, f)` 对应的矩阵为：
$$
\begin{bmatrix} a & c & e \\ b & d & f \\ 0 & 0 & 1 \end{bmatrix}
$$
多个变化的话，是通过矩阵乘法来计算的。

下面为各种变化等效规则：

```
translate(x, y) = matrix(1 0 0 1 x y)

scale(a) = matrix(a 0 0 a 0 0)

rotate(a, cx, cy) = translate(cx,cy) Matrix(cos(a) sin(a) -sin(a) cos(a) 0 0) translate(-cx, -cy)

skewX(a) = matrix(1 0 tan(a) 1 0 0)

skewY(a) = matrix(1 tan(a) 0 1 0 0)
```

例如：`transform="translate(50, 50) rotate(30)"`

变化为：`matrix(1, 0, 1, 0, 50 ,50) * matrix(0.866, 0.5, -0.5, 0.866, 0, 0) = matrix(0.866, 0.5, -0.5, 0.866, 50, 50)` **注意不要调换两个因子，矩阵乘法法则不满足交换律，所以变换也是有顺序的**

```svg
<svg>
    <rect fill="#030" width="100" height="50" transform="translate(50, 50) rotate(30)"></rect>
    <g transform="translate(100)">
        <!--可以把次分组去掉，会发现两个图形重合了，两种变换等效-->
        <rect fill="#080" width="100" height="50" transform="matrix(0.866, 0.5, -0.5, 0.866, 50, 50)"></rect>
    </g>
</svg>
```

## 五、`<base href="/">` 

`<base href="/">` 在页面中是用来重定向资源链接的，规定页面中所有相对链接的基准 URL。

对于 `svg` 来说，会影响到它的寻址方法：`url()`。这个方法默认寻址为当前页面，而单页面应用添加了上述 `base`，它就会去寻找根目录页面，然而根目录页面又被路由定位到其他组件上，找不到 `svg` 中定义的渐变(                                                                                                                                                                                                                                                                                                                                                          		     		     		     		     		     		     		     		     		     		     	     `linearGradient`)，`mask` 等元素，所以会找不到渐变方式，显示不出来。

解决办法有：

1. `url(this_page_name.html#xxx)`
2. `url(window.location.href#xxx)`

这样它就可以找到定义好的渐变元素了

## 六、嵌入react中

1. 直接嵌入页面中

   这个适用于有外部依赖图片的，而且图片路径被替换了的，如 `webpack` 的 `file-loader`，这样可以获取到正确地址插入 `svg`。

   例如：

   ```html
   <!DOCTYPE html>
   <html>
   <head>
       <meta charset="UTF-8">
   </head>
   <body>
       <svg>
           <rect fill="#030" width="100" height="50"></rect>
       </svg>
   </body>
   </html>
   ```

   注意点：

   1. 不要引入 `svg` 的  `<xml>` 声明，只用 `namespace` 即可， `html` 有自己的 `dtd(或!DOCTYPE)`
   2. 标签属性转换，如 `xlink:href` 转换成 `xlinkHref`，破折号同样，`react` 认为前面的属性非法，查看[react支持的SVG属性](https://reactjs.org/docs/dom-elements.html#all-supported-html-attributes)

2. 第二种与第一种类似，只是把 `svg` 图片单独拎出来，然后通过 `use` 标签使用

   如：

   ```svg
   <svg>
     <use xlinkHref="#svgFile"></use>
   </svg>
   ```

3. 转换成字体，不细说

4. 通过插件加载

   例如 `svg-sprite-loader`

**最后，`svg` 图片不是手画的，是工具生成的，不要尝试手动计算坐标，这种行为很二，会哭的。**

## 参考

1. [SVG 图像入门教程](http://www.ruanyifeng.com/blog/2018/08/svg.html)

2. [SVG简单入门](https://www.yuque.com/ada87/svg/acyvog)

3. [xml约束、DTD、命名空间、scheme](https://blog.csdn.net/qq_34983808/article/details/80398804)

4. [svg命名空间](https://www.jianshu.com/p/c590983dbc87)

5. [matrix]: https://www.oxxostudio.tw/articles/201409/svg-20-transform-matrix.html "SVG 研究之路 (20) - transform Matrix"

6. [矩阵乘法](https://zh.m.wikipedia.org/zh-hans/%E7%9F%A9%E9%98%B5)

7. [在react中使用svg的各种骚姿势](https://segmentfault.com/a/1190000015746485)

