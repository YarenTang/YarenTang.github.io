title: Web前端模块化发展历程
date: 2017-06-27 20:00:13
---

模块化是一种处理复杂系统分解成为更好的可管理模块的方式，它可以把系统代码划分为一系列职责单一，高度解耦且可替换的模块，系统中某一部分的变化将如何影响其它部分就会变得显而易见，系统的可维护性更加简单...

<!--more-->

<!-- toc -->

## 1. 什么是模块化  
模块化是一种处理复杂系统分解成为更好的可管理模块的方式，它可以把系统代码划分为一系列职责单一，高度解耦且可替换的模块，系统中某一部分的变化将如何影响其它部分就会变得显而易见，系统的可维护性更加简单易得。

## 2. 为什么要模块化  

随着互联网的飞速发展，新的前端框架与技术不断涌现。这些框架在带给开发者便利的同时，也带去了维护困难的隐患———项目逻辑的复杂化与代码库膨胀。我们可以将原因概括为以下三点：  

- Web 网站正在转变为 Web 应用  
- 网站规模越大，代码也越来越复杂   
- 高度解耦的 JS 文件被大家所需求  
前端开发领域(`JavaScript`、`CSS`、`Template`)并没有为开发者们提供以一种简洁、有条理地的方式来管理模块的方法。为了解决以上问题，我们必须进行合理的模块化管理。  

熟悉 Java 语言的同学都知道，在 Java 中有一个重要的概念——`package` (包)，我们会将相同逻辑的代码放到一个 `package` 下，在需要的地方 `import` (引入)即可。    

前端是否有类似 `package` 的概念呢？  

## 3. Web 前端的模块化的历史浪潮  
### 3.1 发展历史  
由于 JavaScript 这门语言设计用时非常短，所以很多方面都没有考虑周全。好在它的生态圈非常强大，广大开发者为了更好的使用它，对它不断优化。
### 3.1.1 函数封装  
  最早期的时候，我们在一个文件里编写相关函数，像下面这样：  
```JavaScript
function fn1(){
    statement
}

function fn2(){
    statement
}
```

需要的时候加载文件并调用相关函数即可。  
**缺点：Global 被污染，代码中很容易出现命名冲突。**  

### 3.1.2 对象  
为了解决上面的问题，我们又将一个个文件划分为一个个对象，这样在加载的时候就不用加载很多文件。代码如下：
```JavaScript
 var MYAPP = {
   foo: function(){},
   bar: function(){}   
 }

 MYAPP.foo(); //需要时执行
```

这样虽然能减少 Global 上的变量数目，避免变量污染。但是**本质是对象，外部对象可以对其他对象进行修改，很不安全**。  

### 3.1.3 立即执行函数(immediately-invoked function expression )   

 > IIFE (立即调用函数表达式) 是一个 JavaScript 函数 ，它会在定义时立即执行.
也有人将 IIFE 称为匿名闭包.

举个例子：  
```JavaScript
var Module = (function(){
    var _private = "safe now";
    var foo = function(){
        console.log(_private)
    }

    return {
        foo: foo
    }
})()

Module.foo();
Module._private; // undefined
```

我们可以调用定义的方法，但是不能调用定义的属性。这样的模式使得变量冲突降到最低，并且能够控制暴露的方法(类似于接口）。这便是现代模块实现的基石！  

### 3.1.4 Script Loader  

最原始的 JS 加载方法就是在 Html 头文件中通过 Script 标签直接引入，类似下面这样：

```JavaScript
<script src="http://remote.tld/jquery.js"></script>
<script src="local/plugin1.jquery.js"></script>
<script src="local/plugin2.jquery.js"></script>
<script src="local/init.js"></script>
<script>
	initMyPage();
</script>
```

初学者可能会有疑问，全都写在一个文件中看着不是挺方便的吗？  
首先以上列举的可能只是实际项目中的一小部分，其次不要忽视 `<script>` 的一个特点：`并行加载，顺序执行`，这要求开发者需要按照严格的读取顺序，并且很有可能会阻塞阻止文档渲染(`async` 新特性在 HTML5 中才提出)。文件少的时候写起来会很方便，一旦项目变复杂缺点就会暴露无疑：**难以维护，依赖模糊，请求过多** 。  

#### LABjs  
为了解决依赖模糊，我们迫切需要一个加载器（`Loader`）最先出现在开发者视野中的 `Loader` 应该是 [`LABjs`](https://github.com/getify/LABjs])(2009)（ HTML5 发布才有了 `async` 特性）

> LABjs 的核心是 LAB（Loading and Blocking）：Loading 指异步并行加载，Blocking 是指同步等待执行。  

通过使用 LABjs ,上方代码就可以变成这样：  

```JavaScript
<script src="LAB.js"></script>
<script>
  $LAB
  .script("http://remote.tld/jquery.js").wait()//`.wait` 表示加载后立即运行
  .script("/local/plugin1.jquery.js")
  .script("/local/plugin2.jquery.js").wait()
  .script("/local/init.js").wait(function(){
      initMyPage();
  });
</script>
```

当然同一时期风靡的还有类库模块化框架 [YUI](https://yuilibrary.com/)，大家可自行查阅。

#### CommonJS  

在开发者们拼命为浏览器兼容等问题绞尽脑汁的时候，CommonJS 横空出世！
服务器端的 `Node.js` 遵循 [CommonJS规范](http://wiki.commonjs.org/wiki/CommonJS)，该规范的核心思想是允许模块通过 `require` 方法来同步加载所要依赖的其他模块，然后通过 `exports` 或 `module.exports` 来导出需要暴露的接口。

```JavaScript
require("module");
require("../file.js");
exports.doStuff = function() {};
module.exports = someValue;
```

这是 JavaScript 第一次真正意义上跳出浏览器，被人们尊称为模块化的第一座里程碑（`MODULES/1.0`）。给开发者们带来了以下便捷之处：

优点:  

- 服务器端模块便于重用  
- [NPM](https://www.npmjs.com/) 中已经有将近20万个可以使用模块包  
- 简单并容易使用  

缺点：  

- 同步的模块加载方式不适合在浏览器环境中，同步意味着阻塞加载，浏览器资源是异步加载的  
- 不能非阻塞的并行加载多个模块  

实现:  

- NodeJS



#### 双塔奇兵 AMD/CMD  



|       -        | AMD          | CMD  |  
| ------------- |:-------------|:-----|  
| 概念      | [Asynchronous Module Definition](https://github.com/amdjs/amdjs-api) 规范其实只有一个主要接口 `define(id?, dependencies?, factory)`，它要在声明模块的时候指定所有的依赖 `dependencies`，并且还要当做形参传到 factory 中，对于依赖的模块提前执行，依赖前置。| [Common Module Definition](https://github.com/cmdjs/specification/blob/master/draft/module.md) 规范和 AMD 很相似，尽量保持简单，并与 `CommonJS` 和 `Node.js` 的 `Modules 规范` 保持了很大的兼容性。 |
| 优点      | 1. 适合在浏览器环境中异步加载模块</br> 2. 可以并行加载多个模块      |   1. 依赖就近，延迟执行 </br> 2. 可以很容易在 Node.js 中运行 |  
| 缺点      | 1. 提高了开发成本，代码的阅读和书写比较困难，模块定义方式的语义不顺畅</br> 2. 不符合通用的模块化思维方式，是一种妥协的实现      |   依赖 SPM 打包，模块的加载逻辑偏重 |  
| 实现      |     [RequireJS](http://requirejs.org/)          |  [SeaJS](http://seajs.org/)     |  

两者最明显的区别可以从以下代码中查看：  

```JavaScript
//AMD 推崇依赖前置，在定义模块的时候就要声明其依赖的模块
define(['a', 'b'], function(a, b){
    a.doSomething();    // 依赖前置，提前执行
    b.doSomething();
})

//CMD 推崇依赖就近，只有在用到某个模块的时候再去require  
define(function(require, exports, module){
    var a = require("a");
    a.doSomething();
    var b = require("b");
    b.doSomething();    // 依赖就近，延迟执行
})
```

概括而言 Commonjs 用在服务器端，加载是同步的。    
AMD, CMD 用在浏览器端，加载是异步的。  

`CommonJS` (规划并标准化、致力于设计 JavaScript API)的诞生开启了 "JavaScript 模块化" 的时代。`CommonJS` 的模块提案为在服务器端的 JavaScript 模块化做出了很大的贡献，但是在浏览器下的 JavaScript 模块应用很有限。随之而来又诞生了其它前端领域的模块化方案，像 requireJS、SeaJS 等，然而这些模块化方案并不是十分适用 ，并没有从根本上解决模块化的问题。

### 3.2 解决方案  

前面已经为大家回顾了 JavaScript 的模块化发展历史，然而，在前端开发过程中还涉及到样式、图片、字体、HTML 模板等等众多的资源。这些资源还会以各种方言的形式存在，比如 coffeescript、 less、 sass、众多的模板库、多语言系统（i18n）等等。如果他们都可以视作模块，并且都可以通过 `require` 的方式来加载，将带来优雅的开发体验，比如：  
```JavaScript
require("./style.css");
require("./style.less");
require("./template.jade");
require("./image.png");
```

那么如何做到让 `require` 能加载各种资源呢？  

#### Webpack

> [Webpack](http://webpack.github.io/docs/) 是一个模块打包器。它将根据模块的依赖关系进行静态分析，然后将这些模块按照指定的规则生成对应的静态资源。

![webpack](http://om6vvgjw7.bkt.clouddn.com/what-is-webpack.png)  

Webpack 的优点可以概括为以下几点：  

- **代码拆分**  
Webpack 有两种组织模块依赖的方式，同步和异步。异步依赖作为分割点，形成一个新的块。在优化了依赖树后，每一个异步区块都作为一个文件被打包。  

- **Loader**    
Webpack 本身只能处理原生的 JavaScript 模块，但是 loader 转换器可以将各种类型的资源转换成 JavaScript 模块。这样，任何资源都可以成为 Webpack 可以处理的模块。  

- **智能解析**    
Webpack 有一个智能解析器，几乎可以处理任何第三方库，无论它们的模块形式是 CommonJS、 AMD 还是普通的 JS 文件。甚至在加载依赖的时候，允许使用动态表达式 `require("./templates/" + name + ".jade")`。  

- **插件系统**    
Webpack 还有一个功能丰富的插件系统。大多数内容功能都是基于这个插件系统运行的，还可以开发和使用开源的 Webpack 插件，来满足各式各样的需求。  

- **快速运行**    
Webpack 使用异步 I/O 和多级缓存提高运行效率，这使得 Webpack 能够以令人难以置信的速度快速增量编译。    

当然，光有 `Webpack` 是不够的，目前大家比较推崇的最佳拍档为：

|     包管理工具       | 模块化构建工具          | 模块化规范  |  
|:--------:|:----------:|:-----:|
|   npm        |Webpack       | ES6模块|

当然，目前比较有许多 “自带模块化buff” 的前端框架（组件化）：`Vue` 、 `React`等，它们都有以下特点:  

- 使用 Virtual DOM  
- 提供了响应式（ Reactive ）和组件化（ Composable ）的视图组件。  
- 保持注意力集中在核心库，伴随于此，有配套的路由和负责处理全局状态管理的库。    

在 `Vue` 中，一个组件文件(`.Vue`)中，拥有`<template>`,`<script>`,`<style>`三个部分。每个组件中管理自己的样式、逻辑，这样在大型项目开发与维护时都起到了非常便捷的作用。  

## 4. 未来的模块化  

说到模块化未来的发展，值得一提的便是 [WebAssembly](http://webassembly.org/)。大家都知道 `JavaScript` 是一种解释性脚本语言，导致其运行时消耗大量的性能做为代价，这也是 `JavaScript` 的瓶颈之一。`WebAssembly` 旨在解决这个问题。  

### WebAssembly 是什么

>  Define a portable, size- and load-time-efficient binary format to serve as a compilation target which can be compiled to execute at native speed by taking advantage of common hardware capabilities available on a wide range of platforms, including mobile and IoT.
定义一个可移植，体积紧凑，加速迅捷的二进制格式为编译目标，而此二进制格式文件将可以在各种平台（包括移动设备和物联网设备）上被编译，然后发挥通用的性能以原生应用的速度运行。

### WebAssembly 的目标

某乎上广传 “谷歌、苹果、谋智、微软一起再一次发明了 Silverlight ”。可以看出，大家对 `WebAssembly` 充满了信心与期待。  

作为 [W3C WebAssembly Community Group](https://www.w3.org/community/webassembly) 中的一项开放标准，WebAssembly 是为下列目标而生的：

- 快速、高效、可移植——通过利用常见的硬件能力，WebAssembly 代码在不同平台上能够以接近本地速度运行。  
- 可读、可调试—— WebAssembly 是一门低阶语言，但是它有确实有一种人类可读的文本格式（其标准即将得到最终版本），这允许通过手工来写代码，看代码以及调试代码。   
- 保持安全—— WebAssembly 被限制运行在一个安全的沙箱执行环境中。像其他网络代码一样，它遵循浏览器的同源策略和授权策略。  
- 不破坏网络—— WebAssembly 的设计原则是与其他网络技术和谐共处并保持向后兼容。  

### WebAssembly 的核心  

那么 WebAssembly 是如何在浏览器中运行的呢？我们可以看一下它的几个关键的概念：  

- **模块** : 表示一个已经被浏览器编译为可执行机器码的 WebAssembly 二进制代码。一个模块是无状态的并且像一个二进制大对象（Blob）一样能够被缓存到 IndexedDB 中或者在 windows 和 workers 之间进行共享（通过 `postMessage()` 函数）。一个模块能够像一个 ES2015的模块一样声明导入和导出。   
- **内存** : 一个可变大小的 ArrayBuffer 。它包含了一个连续的字节数组并且 WebAssembly 的低级内存存取指令可对其进行读写操作。  
- **表格** : 一个可变大小的包含引用类型（比如，函数）的带类型数组。它包含了不能作为原始字节存储在内存中的引用（为了安全和可移植性的原因）。  
- **实例** : 一个模块及其在运行时使用的所有状态，包括内存、表格和一系列导入值。一个实例就像一个已经被加载到一个拥有一组特定导入的特定的全局变量的 ES2015模块。  
