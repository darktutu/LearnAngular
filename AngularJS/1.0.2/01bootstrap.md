# AngularJs -- bootstrap

AngularJs学习笔记系列第一篇，希望我可以坚持写下去。本文内容主要来自 http://docs.angularjs.org/guide/ 文档的内容，但也加入些许自己的理解与尝试结果。

## 一、总括

本文用于解释Angular初始化的过程，以及如何在你有需要的时候对Angular进行手工初始化。

## 二、Angular `<script>` 标签

本例用于展示如何通过推荐的路径整合Angular，实现自动初始化。

``` HTML
<!doctype html>
<html xmlns:ng="http://angularjs.org" ng-app>
    
<body>
...
<script src="angular.js">
</body>
</html>

 ```

将sciprt标签放置于页面底部。这样做能避免因为加载angular.js而阻挡HTML的加载，从而降低应用的加载时间。我们可以在http://code.angularjs.org中获取到最新版本的angularJs。出于安全考虑，切勿在产品中直接引用这个地址来加载脚本。但如果仅仅是研究学习使用的话，直接连接也无妨。
选择：angular-[version].js 是方便阅读的一个版本，适合日常开发、调试使用。
选择：angular-[version].min.js 是压缩、混淆后的版本，适合最终产品使用。
放置”ng-app”到应用的根节点中，如果你想让angular自动启动你的应用，通常可以放置于<html>标签中。
```
<html ng-app>
```
如果我们需要使用老派风格的directive语法”ng:”，那么我们需要加入一个xml-namespace到html标签中以“取悦”IE。（这个是一个历史原因，我们也不推荐使用ng:）

``` 
    <html xmlns:ng="http://angularjs.org"> 
```

## 三、自动初始化

Angular会在DOMContentLoaded事件中自动初始化，Angular会找出由你通过ng-app这个directive指定的应用根节点。如果找到，Angular会做以下事情：

* 加载与module相关的directive。
* 创建应用相关的injector（依赖管理器）。
* 以ng-app指定根节点，开始对DOM进行相关“编译”工作。换言之，可以将页面的其中一部分（非 `<html>` ）作为根节点，从而限制angular的作用范围。

``` html
<!DOCTYPE HTML>
<html lang="zh-cn">
<head>
    <meta charset="UTF-8">
    <title>Bootstrap-auto</title>
    <style type="text/css">
        .ng-cloak {
            display: none;
        }
    </style>
</head>
<body>
    这里是ng-app外面的~~{{1+2}}
    <div ng-app class="ng-cloak">这里是ng-app里面~~~{{1+3*2}}</div>
    <script src="../angular-1.0.1.js" type="text/javascript"></script>
</body>
</html>
```

注：里面的”ng-cloak”，这个是用于在angular.js编译完成之前（对！没错！是编译完成之前，不是angularjs加载完成之前。所以，如果想很好地避免这个情况，最好的办法是优化应用的加载流程，或者结合css对未编译的模版进行处理。而由于那万恶的ie6、7不支持属性选择器，所以最好使用class=”ng-cloak”的方式。编译完成后，这个class或属性会被删除。）隐藏模版，避免在页面显示原模版。
四、手工初始化

如果我们想进一步控制初始化进程(例如你需要通过script loader加载angular.js或者在angular编译页面前做一些操作)，那么我们可以用一个手工调用的启动方法去代替。

以下例子等同于使用ng-app这个directive：
``` html
<!DOCTYPE HTML>
<html lang="zh-cn">
<head>
    <meta charset="UTF-8">
    <title>Bootstrap-manual</title>
    <style type="text/css">
        .ng-cloak {
            display: none;
        }
    </style>
</head>
<body>
    这里是ng-app外面的~~{{1+2}}
    <div id="rootOfApp" class="ng-cloak">这里是ng-app里面~~~{{1+3*2}}</div>
    <script src="../angular-1.0.1.js" type="text/javascript"></script>
    <script type="text/javascript">
        angular.element(document).ready(function() {
            angular.bootstrap(angular.element(document.getElementById("rootOfApp")));
        });
    </script>
</body>
</html>
```
就是说，我们的代码可以按照以下步骤编写：
1. 在页面和其他代码加载完成后，找到应用模版的根节点；
2. 调用angular.bootstrap，让angular去将模版编译为一个可执行的，双向绑定的应用！