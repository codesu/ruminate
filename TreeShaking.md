---

2019/04/29

---

### 1. 概念

在源代码中，有很多定义了却没有用到的代码，它们并不需要出现在最终打包好的文件中。

将用不到的代码缩减的概念，就是 `tree shaking`。

在之前的 `js` 中，因为模块是动态的，代码是否引入，需要在实际运行的时候才能知道，如：

```js
var myDynamicModule;

if (condition) {
    myDynamicModule = require("foo");
} else {
    myDynamicModule = require("bar");
}
```

所以**打包阶段**，是没法去除没有用到的代码的。

而在 `es6` 的规范中，模块引入是**静态**的，提供了代码分析的基础，可以在打包过程中去掉没有用到的代码。

#### 存在问题：

虽然想法很好，但是实施起来，却困难重重。

对于纯函数来说，执行起来并没有副作用，所以可以放心的把没有用的的代码去除。

但是对于有**副作用**的函数，即指函数运行过程中，影响了本身作用域外的变量，每次运行的结果不一定一样。

比如 `polyfill`，涉及到`windows`，就不能直接断定，这段代码能不能直接去除，更倾向于包含它。

**这时候需要指定，哪些函数是没有副作用的**。

### 2. 探索

综上，要实现 `tree shaking`，有两个前提：

- `es6` 的模块引入方式
- 让打包工具知道**没用到的代码**有没有**副作用**

如此，`webpack` 通过以下步骤来实现 `tree-shaking`

 	1. **标记代码**：它将未使用的，无副作用的函数标记出来
 	2. **移除标记代码**：通过压缩插件，去除打上无用且纯函数标记的代码。

##### 2.1.1 `es6` 模块

我们现在源代码一般都是用 `es6` 的模块写法了，这种写法可以用作静态代码分析，还有动态引用的好处。而打包工具通过 `babel` 来将这部分转码成 `es5` 的写法，这本来没有什么问题。

但是我们要应用 `tree-shaking` 功能时，`webpack` 需要用到 `es6` 风格的模块语法才能静态分析，所以在这一步我们需要去掉模块引入写法的转码。

```js
// .babelrc
{
    "presets": [
        // 关闭转换引入方式模块
        // https://babeljs.io/docs/en/babel-preset-env/
        ["@babel/preset-env", { "modules": false }]
     ]
}
```

这一步之后，我们可以看到转码后的代码会被标记上：

```js
/***/ 21:
/***/ (function(module, __webpack_exports__, __webpack_require__) {

"use strict";
/* unused harmony export Radio */
/* harmony import */ var __WEBPACK_IMPORTED_MODULE_0_react__ = __webpack_require__(1);
```

其中 `unused harmony` 就代表未使用的 `es6` 模块。

##### 2.1.2 副作用

标记好了未使用，我们并不能直接去除它，还要标记它是无副作用的，它存在与否对其他代码运行都没有影响。比如 `defaultProps` 转码后在原类型函数上添加原型方法，就被认为是有副作用的。

```js
Radio.defaultProps = {
  label: 'radio_label_mark'
};
```

有两种方法来标记它是否有副作用：

1. 代码标记（`webpack@2，3`）

   这种办法，是在 `babel` 转码的时候，就在代码中添加一个注释，`/*#__PURE__*/`，然后 `uglifyjs` 读到这里，又知道它是纯函数，这样，它就可以放心[移除它](https://github.com/mishoo/UglifyJS2/pull/1448)。

   ```js
   // webpack@3
   // UglifyJsPlugin需要为1.2.7版本，手动安装
   // new webpack.optimize.UglifyJsPlugin自带版本为0.4.6，不支持根据注释删除代码
   // 生成以下代码时，不要开启UglifyJsPlugin
   var Radio =
   /*#__PURE__*/
   function (_Component) {
       
   
   // 开启UglifyJsPlugin后，这段代码被删除
   ```

   但是如果给组件添加静态属性，还是不能够 `tree-shaking`，即使是已经有标记 `#pure` 的时候。[参考](https://github.com/bvaughn/react-virtualized/issues/889)

2. 手动标记（`webpack@4`）

   有两种方式来标记副作用：

   1. 通过 `sideEffects` 来标记是否有副作用。在 `package.json` 中，加上 [`sideEffects`](<https://webpack.js.org/guides/tree-shaking/#mark-the-file-as-side-effect-free>)，可以表明这个包是否有副作用。

   ```js
   {
     "name": "your-project",
     "sideEffects": false // 无副作用，放心去除用不到的代码
      
      // 以下为有副作用写法，需要打到包里去
      "sideEffects": [
       "./src/some-side-effectful-file.js",
       "*.css"
     ]
   }
   ```

   2. 在 [`module.rule`](https://webpack.js.org/configuration/module/#rulesideeffects) 中来定义，`bool` 类型

   最后，设置好了副作用文件，还需要打开识别副作用的开关：

   ````js
   module.exports = {
     //...
     optimization: {
       sideEffects: true
     }
   };
   ````

##### 2.1.3 另一种方法

在上面的步骤中，我们不由的会想，既然它知道 `harmony`和 `PURE` ，为什么不在标记的时候就去掉它呢？然后再进行代码压缩不就没有 `uglify` 什么事了嘛。`Rollup` 就是通过这种方式实现。

`webpack` 也有这方面的插件，就是[`babel-minify-webpack-plugin`](https://webpack.js.org/plugins/babel-minify-webpack-plugin/)，它使用 [`babel-minify`](https://github.com/babel/minify) 做代码分析和缩减，但是最好还是别在生产中用，一直处于 `beta` 版本，很多 `bug`。

##### 2.1.4 `rollup` 可以么

对于 `rollup`，实验了下 `tree-shaking` 的[体验](https://www.rollupjs.com/guide/en#tree-shaking)非常好，打包后就已经删除了未使用代码，如果要混淆压缩的话，还是要 `uglify` 插件配合。

但是最好不要将它用作业务项目的打包工具，只用作库的打包工具。因为它的目的并不是为了业务项目而生，比如概念中没有 `loader` 的想法，对于 `file-loader` 这些，都是通过插件去实现的，在这方面，明显 `webpack` 更擅长。


### 3. 结论

 `webpack@4` 相比之前版本，更加的开箱即用，[`tree-shaking`](https://webpack.js.org/guides/tree-shaking#conclusion) 也换成了 `TerserPlugin` 来分析代码，在**副作用**这块我们要做的只是指明 `sideEffects` 的范围，设置 [`mode=production`](https://webpack.js.org/configuration/mode/#mode-production) 即可开启一大堆优化插件和配置。

```js
// webpack.production.config.js
module.exports = {
  mode: 'production'
}
```

而 `rollup` 方面则更加方便，不过它最好用来打包 `js` 库，这才是它最擅长的地方。

### 参考：

1. [what-is-tree-shaking](https://bitsofco.de/what-is-tree-shaking/)
2. [reduce javascript payloads with tree shaking](https://developers/google.com/web/fundamentals/performance/optimizing-javascript/tree-shaking/)
3. [【译】如何在 Webpack 2 中使用 tree-shaking](https://juejin.im/post/599bc13b6fb9a024a370f4ec)
4. [你的Tree-Shaking并没什么卵用](https://segmentfault.com/a/1190000012794598)
5. [Webpack 4 教程 - 7. 通过“tree shaking”减少打包的尺寸](https://segmentfault.com/a/1190000016767989)
6. [Webpack系列——探索babel7结合webpack3 tree-shaking对打包后代码的影响](https://www.colabug.com/2163246.html)
7. [antd webpack.config](https://github.com/ant-design/antd-tools/blob/master/lib/getWebpackConfig.js)
8. [3.4.2 "sideEffects: false" option causes site to render improperly](https://github.com/ant-design/ant-design/issues/10190#issuecomment-383443469)
9. [ant-design 代码缩减](https://ant.design/docs/react/faq-cn#%E6%88%91%E5%8F%AA%E6%83%B3%E4%BD%BF%E7%94%A8-Menu/Button-%E7%AD%89%EF%BC%8C%E4%BD%86%E4%BC%BC%E4%B9%8E%E6%88%91%E5%BF%85%E9%A1%BB-import-%E6%95%B4%E4%B8%AA-antd-%E5%92%8C%E5%AE%83%E7%9A%84%E6%A0%B7%E5%BC%8F%E6%96%87%E4%BB%B6%E3%80%82)
10. [material-ui 代码缩减](https://material-ui.com/guides/minimizing-bundle-size/)

