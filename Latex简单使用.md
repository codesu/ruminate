# Latex简单使用

> 在markdown中使用，请确定编辑器开启`Latex`支持

## Ⅰ. 使用方式

### 	插入方式

​	行内使用，在公式两侧加上`$`，如`$ 1 + 2 $`

​	独立公式显示，在公示两侧加上`$$`，如`$$ 1 + 2 $$`

### 	表达方式

​	1. 可以自动编号，定义公式名，如：

​		`\begin{}...\label{eq:name}...\end`，后面可以通过`\eqref{eq:name}`引用此公式

​	2. 可以通过`{}`将几个字符合并为一个整体，如：

​		`2^{123}`表示为$2^{123}$

​	3. 可以通过`\`在公式内转码，显示特殊字符，如：

​		`DBL\_EPSILON`显示为$DBL\_EPSILON​$

## Ⅱ. 常用字符写法 

| 输入  | 输出      |
| ----- | --------- |
| `2^2` | $ 2^2 $  |
| `e_{min}` | $ e_{min} $ |
| `1 \sim 2` | $ 1 \sim 2 $ |
| `1 \times 2`，`2 \div 2` | $ 1 \times 2 $， $ 2 \div 2 $ |
| `\leq`， `<` | $ 1 \leq 2 $， $ 1 < 2 $ |
| `\geq`， `>` | $ 2 \geq 1$， $ 2 > 1 $ |
| `\alpha`，`\beta`，`\epsilon`，`\varepsilon` | $ \alpha $，$\beta$，$\epsilon$，$\varepsilon$ |
| `\dots` | $ \dots $ |
| `\to` | $ 1 \to 2 $ |
| `\quad` | 空格 |
| `\not=` | $\not=$ |
| `\infty` | $\infty$ |
| `\approx` | $\approx$ |

## 参考

1. [Cmd Markdown 公式指导手册](https://www.zybuluo.com/codeep/note/163962)
2. [LaTex数学公式语法](http://lixingcong.github.io/2016/04/04/LaTex-intro/)