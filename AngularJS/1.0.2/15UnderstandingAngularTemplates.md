# AngularJs --Understanding Angular Templates
原版地址：http://docs.angularjs.org/guide/dev_guide.mvc.understanding_model

 

　　angular template是一个声明规范，与model、controller的信息一起，渲染成用户在浏览器中所看到的视图。它是静态的DOM，包括HTML、CSS、angular特别的元素和angular指定的元素属性。angular元素和属性指示angular去扩展行为以及将template DOM转换为动态视图的DOM。

　　下面是我们可以在template中使用的angular元素已经元素属性的类型：

Directive（http://www.cnblogs.com/lcllao/archive/2012/09/09/2677190.html） - 一个扩展现有DOM元素或者代表一个可复用的DOM组件的属性或者元素，即控件。
Markup（http://code.angularjs.org/1.0.2/docs/api/ng.$interpolate） - 通过双大括号表示法{{}}来绑定表达式到元素中，是内建的angular标记。
Filter（http://code.angularjs.org/1.0.2/docs/guide/dev_guide.templates.filters）- 用于格式化我们给用户看的数据。
Form controls （http://www.cnblogs.com/lcllao/archive/2012/09/17/2688127.html）- 让我们验证用户输入。
　　注意：除了可以在模版中声明上面的元素以外，我们也可以在javascript代码中访问这些元素。

　　下面的代码片段，展示了一个简单的angular template，它由标准的HTML标签以及angular directive、花括号绑定的expression（{{expression}}，http://www.cnblogs.com/lcllao/archive/2012/09/16/2687162.html）组成。

``` html
<!DOCTYPE html>
<!--ng-app，定义应用范围，在这里创建root scop-->
<html ng-app>
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8"/>
    <title>template</title>
    <meta content="IE=edge,chrome=1" http-equiv="X-UA-Compatible">
    <style type="text/css">
        .ng-cloak {
            display: none;
        }
    </style>
</head>
<!--
    ng-cloak，在编译后会去掉的class
    ng-controller，一个directive，用于指定当前的模版对应的Controller为MyController
-->
<body class="ng-cloak" ng-controller="MyController">

<!--
    ng-model，directive，用于指定input的值对应的model为foo。
-->
<input type="text" ng-model="foo" value=""/>
<!--
    ng-click，directive，单击后需要做的事情，可以是expression，如 buttonText = '1'；
    也可以是调用函数，如下面所示。
    {{buttonText}}，用于展示当前scope链中能够或得到的buttonText的值
-->
<button ng-click="changeFoo()">{{buttonText}}</button>

<script src="../angular-1.0.1.js" type="text/javascript"></script>
<script type="text/javascript">
    function MyController($scope) {
        $scope.buttonText = "默认的东东";//初始化model buttonText
        $scope.foo = "修改我吧";//初始化model foo
        $scope.changeFoo = function() {//声明changeFoo
            this.buttonText = this.foo;//将foo的值赋给buttonText
            //这里使用的this，就是指向当前$scope的。
        };
    }
</script>
</body>
</html>
```
 

　　在一个简单的单页应用中，模版由HTML、CSS以及angular directive组成，都包含在一个HTML文件中（通常叫它index.html）。但在一些更加复杂的应用中，我们可以在一个页面中，通过使用“partials”来显示多个视图，即将模版分段存放在独立的HTML文件中。我们可以在主页面中使用$route服务（http://code.angularjs.org/1.0.2/docs/api/ng.$route）与ngView directive（http://code.angularjs.org/1.0.2/docs/api/ng.directive:ngView）来协同“include”那些partials。这个技术的一个例子，展示在angular tutorial（http://code.angularjs.org/1.0.2/docs/tutorial/index）的第七、八步骤中。（这部分我稍后再玩-_-!）