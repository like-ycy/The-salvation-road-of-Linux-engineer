# CSS语法

## CSS 语法

CSS 规则集（rule-set）由选择器和声明块组成：

![CSS 选择器](CSS%E8%AF%AD%E6%B3%95.assets/selector.gif)

选择器指向您需要设置样式的 HTML 元素。

声明块包含一条或多条用分号分隔的声明。

每条声明都包含一个 CSS 属性名称和一个值，以冒号分隔。

多条 CSS 声明用分号分隔，声明块用花括号括起来。

### 实例

在此例中，所有 `<p>` 元素都将居中对齐，并带有红色文本颜色：

```CSS
p {
  color: red;
  text-align: center;
}
```



### 例子解释

- p 是 CSS 中的**选择器**（它指向要设置样式的 HTML 元素：<p>）。
- color 是属性，red 是属性值
- text-align 是属性，center 是属性值

