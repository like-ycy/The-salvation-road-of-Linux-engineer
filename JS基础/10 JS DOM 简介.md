# JavaScript DOM 简介

## HTML DOM（文档对象模型）

当网页被加载时，浏览器会创建页面的文档对象模型（**D**ocument **O**bject **M**odel）。

**HTML DOM\* \*模型被结构化为**对象树\*\*：

### 对象的 HTML DOM 树

![DOM HTML 树](10%20JS%20DOM%20%E7%AE%80%E4%BB%8B.assets/ct_htmltree.gif)

通过这个对象模型，JavaScript 获得创建动态 HTML 的所有力量：

- JavaScript 能改变页面中的所有 HTML 元素
- JavaScript 能改变页面中的所有 HTML 属性
- JavaScript 能改变页面中的所有 CSS 样式
- JavaScript 能删除已有的 HTML 元素和属性
- JavaScript 能添加新的 HTML 元素和属性
- JavaScript 能对页面中所有已有的 HTML 事件作出反应
- JavaScript 能在页面中创建新的 HTML 事件

## 什么是 DOM？

DOM 是一项 W3C (World Wide Web Consortium) 标准。

DOM 定义了访问文档的标准：

> “W3C 文档对象模型（DOM）是中立于平台和语言的接口，它允许程序和脚本动态地访问、更新文档的内容、结构和样式。”

W3C DOM 标准被分为 3 个不同的部分：

- Core DOM - 所有文档类型的标准模型
- XML DOM - XML 文档的标准模型
- HTML DOM - HTML 文档的标准模型

## 什么是 HTML DOM？

HTML DOM 是 HTML 的标准*对象*模型和*编程接口*。它定义了：

- 作为**对象**的 HTML 元素
- 所有 HTML 元素的**属性**
- 访问所有 HTML 元素的**方法**
- 所有 HTML 元素的**事件**

**换言之：HTML DOM 是关于如何获取、更改、添加或删除 HTML 元素的标准**。
