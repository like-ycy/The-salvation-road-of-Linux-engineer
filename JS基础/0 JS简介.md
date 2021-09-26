# JavaScript简介

JavaScript（简称“JS”） 是一种具有函数优先的轻量级，解释型或即时编译型的[编程语言](https://baike.baidu.com/item/编程语言/9845131)。虽然它是作为开发[Web](https://baike.baidu.com/item/Web/150564)页面的[脚本语言](https://baike.baidu.com/item/脚本语言/1379708)而出名，但是它也被用到了很多非[浏览器](https://baike.baidu.com/item/浏览器/213911)环境中，JavaScript 基于原型编程、多范式的动态脚本语言，并且支持[面向对象](https://baike.baidu.com/item/面向对象/2262089)、命令式、声明式、[函数](https://baike.baidu.com/item/函数/301912)式编程范式。 

JavaScript在1995年由[Netscape](https://baike.baidu.com/item/Netscape/2778944)公司的Brendan Eich，在网景导航者浏览器上首次设计实现而成。因为Netscape与[Sun](https://baike.baidu.com/item/Sun/69463)合作，Netscape管理层希望它外观看起来像Java，因此取名为JavaScript。但实际上它的语法风格与[Self](https://baike.baidu.com/item/Self/4959923)及[Scheme](https://baike.baidu.com/item/Scheme/8379129)较为接近。 

JavaScript的标准是[ECMAScript ](https://baike.baidu.com/item/ECMAScript /1889420)。截至 2012 年，所有浏览器都完整的支持ECMAScript 5.1，旧版本的浏览器至少支持ECMAScript 3 标准。2015年6月17日，ECMA国际组织发布了ECMAScript的第六版，该版本正式名称为 ECMAScript 2015，但通常被称为ECMAScript 6 或者ES2015。

## 产生背景

JavaScript最初由[Netscape](https://baike.baidu.com/item/Netscape)的[Brendan Eich](https://baike.baidu.com/item/Brendan Eich)设计，最初将其脚本语言命名为[LiveScript](https://baike.baidu.com/item/LiveScript)，后来Netscape在与Sun合作之后将其改名为JavaScript。JavaScript最初受[Java](https://baike.baidu.com/item/Java/85979)启发而开始设计的，目的之一就是“看上去像Java”，因此语法上有类似之处，一些名称和命名规范也借自Java，但JavaScript的主要设计原则源自[Self](https://baike.baidu.com/item/Self/4959923)和[Scheme](https://baike.baidu.com/item/Scheme)。JavaScript与Java名称上的近似，是当时[Netscape](https://baike.baidu.com/item/Netscape)为了营销考虑与Sun微系统达成协议的结果。微软同时期也推出了[JScript](https://baike.baidu.com/item/JScript)来迎战JavaScript的脚本语言。

发展初期，JavaScript的标准并未确定，同期有[Netscape](https://baike.baidu.com/item/Netscape)的JavaScript，微软的JScript和CEnvi的ScriptEase三足鼎立。为了互用性，[Ecma国际](https://baike.baidu.com/item/Ecma国际)（前身为欧洲计算机制造商协会）创建了ECMA-262标准（[ECMAScript](https://baike.baidu.com/item/ECMAScript)），两者都属于ECMAScript的实现，尽管JavaScript作为给非程序人员的脚本语言，而非作为给程序人员的脚本语言来推广和宣传，但是JavaScript具有非常丰富的特性。 1997年，在ECMA（[欧洲计算机制造商协会](https://baike.baidu.com/item/欧洲计算机制造商协会/2052072)）的协调下，由Netscape、Sun、微软、Borland组成的工作组确定统一标准：ECMA-262。完整的JavaScript实现包含三个部分：[ECMAScript](https://baike.baidu.com/item/ECMAScript)，[文档对象模型](https://baike.baidu.com/item/文档对象模型)，浏览器对象模型。

JavaScript是[甲骨文](https://baike.baidu.com/item/甲骨文/471435)公司的注册商标。[Ecma国际](https://baike.baidu.com/item/Ecma国际)以JavaScript为基础制定了[ECMAScript](https://baike.baidu.com/item/ECMAScript)标准。JavaScript也可以用于其他场合，如服务器端编程（[Node.js](https://baike.baidu.com/item/Node.js/7567977)）。

## 主要功能

1.嵌入[动态文本](https://baike.baidu.com/item/动态文本)于HTML页面。 

2.对浏览器事件做出响应。 

3.读写[HTML元素](https://baike.baidu.com/item/HTML元素)。 

4.在数据被提交到服务器之前验证数据。 

5.检测访客的浏览器信息。 [7] 控制[cookies](https://baike.baidu.com/item/cookies/187064)，包括创建和修改等。 

6.基于[Node.js](https://baike.baidu.com/item/Node.js/7567977)技术进行服务器端编程。 

## 语言组成

[ECMAScript](https://baike.baidu.com/item/ECMAScript)，描述了该语言的语法和基本[对象](https://baike.baidu.com/item/对象/2331271)。

[文档对象模型](https://baike.baidu.com/item/文档对象模型)（DOM），描述处理网页内容的方法和接口。 

浏览器对象模型（[BOM](https://baike.baidu.com/item/BOM/1801)），描述与浏览器进行交互的[方法](https://baike.baidu.com/item/方法/3009367)和[接口](https://baike.baidu.com/item/接口)。 

## 语言特点

(1)[脚本语言](https://baike.baidu.com/item/脚本语言)。JavaScript是一种解释型的脚本语言，C、[C++](https://baike.baidu.com/item/C%2B%2B)等语言先[编译](https://baike.baidu.com/item/编译)后执行，而JavaScript是在程序的运行过程中逐行进行解释。

(2)基于对象。JavaScript是一种基于对象的脚本语言，它不仅可以创建对象，也能使用现有的对象。

(3)简单。JavaScript语言中采用的是弱类型的变量类型，对使用的数据类型未做出严格的要求，是基于Java基本语句和控制的脚本语言，其设计简单紧凑。

(4)动态性。JavaScript是一种采用事件驱动的脚本语言，它不需要经过Web服务器就可以对用户的输入做出响应。在访问一个网页时，鼠标在网页中进行鼠标点击或上下移、窗口移动等操作JavaScript都可直接对这些事件给出相应的响应。

(5)跨平台性。JavaScript脚本语言不依赖于操作系统，仅需要浏览器的支持。因此一个JavaScript脚本在编写后可以带到任意机器上使用，前提是机器上的浏览器支 持JavaScript脚本语言，JavaScript已被大多数的浏览器所支持。 不同于服务器端脚本语言，例如[PHP](https://baike.baidu.com/item/PHP/9337)与[ASP](https://baike.baidu.com/item/ASP/128906)，JavaScript主要被作为客户端脚本语言在用户的浏览器上运行，不需要服务器的支持。所以在早期程序员比较青睐于JavaScript以减少对服务器的负担，而与此同时也带来另一个问题，安全性。

而随着服务器的强壮，虽然程序员更喜欢运行于服务端的脚本以保证安全，但JavaScript仍然以其跨平台、容易上手等优势大行其道。同时，有些特殊功能（如[AJAX](https://baike.baidu.com/item/AJAX/8425)）必须依赖JavaScript在客户端进行支持。

