# AngularJs --expression
原版地址：http://code.angularjs.org/1.0.2/docs/guide/expression

 

　　表达式（Expressions）是类Javascript的代码片段，通常放置在绑定区域中（如｛｛expression｝｝）。表达式通过$parse服务（http://code.angularjs.org/1.0.2/docs/api/ng.$parse）解析执行。

　　例如，以下是angular中有效的表达式：

* 1+2
* 3*10 | currency
* user.name
 

### 一、Angular表达式 vs. Js 表达式

　　这很容易让人将angular视图表达式联想为javascript表达式，但这并不完全正确，因为angular不是通过javascript的eval()对表达式进行求值。你可以将angular表达式想象为带有以下差异的javascript表达式：

属性求值：所有属性的求值是对于scope的，而javascript是对于window对象的。
宽容（forgiving）：表达式求值，对于undefined和null，angular是宽容的，但Javascript会产生NullPointerExceptions（-_-!!!!怎么我没见过）。
没有流程控制语句：在angular表达式里，我们不能做以下任何的事：条件分支、循环、抛出异常。
过滤器（filters）：我们可以就将表达式的结果传入过滤器链（filter chains）。例如将日期对象转换为本地指定的人类可读的格式。
　　另一方面，如果我们想（在angular表达式中）执行任意的Javascript代码，我们可以将那些代码写到Controller的一个方法中并调用它。如果我们想在javascript中eval()一个angular表达式，可以使用$eval()方法。

 

``` html
<!DOCTYPE HTML>
<html lang="zh-cn" ng-app="ExpressionTest">
<head>
    <meta charset="UTF-8">
    <title>expression-e1</title>
    <style type="text/css">
        .ng-cloak {
            display: none;
        }
    </style>
</head>
<body ng-controller="MyCtrl">
1 + 2 = {{1+2}}
<br/>
Expression:
<input type="text" ng-model="expr"/>
<button ng-click="addExp(expr)">Evaluate</button>
<ul>
    <li ng-repeat="expr in exprs">
        [<a ng-click="removeExp($index)" href="">X</a>]
        <tt>{{expr}}</tt>=><span ng-bind="$parent.$eval(expr)"></span>
    </li>
</ul>
<script src="../angular-1.0.1.js" type="text/javascript"></script>
<script type="text/javascript">
    var app = angular.module("ExpressionTest", []);
    app.controller("MyCtrl", function ($scope) {
        var exprs = $scope.exprs = [];
        $scope.expr = "3*10|currency";
        $scope.addExp = function(expr) {
            exprs.push(expr);
        };
        $scope.removeExp = function (index) {
            exprs.splice(index, 1);
        };
    });
</script>
</body>
</html>
```
 

 

### 二、属性求值（Property Evaluation）

　　angular的表达式解析环境的上下文是scope，而javascript则是window（应该是指严格模式evel的时候），angular需要通过$window访问global window对象。例如，如果我们需要在表达式中调用定义在window对象上的alert()，我们需要使用$window.alert()。这样做的用意是避免意外访问了公共属性（global state）（一个同源的小BUG？a common source of subtle bugs）。

``` html
<!DOCTYPE HTML>
<html lang="zh-cn" ng-app="PropertyEvaluation">
<head>
    <meta charset="UTF-8">
    <title>PropertyEvaluation</title>
    <style type="text/css">
        .ng-cloak {
            display: none;
        }
    </style>
</head>
<body>
<div ng-controller="MyCtrl">
    Name: <input ng-model="name" type="text"/>
    <button ng-click="greet()">Greet</button>
</div>
<script src="../angular-1.0.1.js" type="text/javascript"></script>
<script type="text/javascript">
    var app = angular.module("PropertyEvaluation", []);
    app.controller("MyCtrl", function ($scope,$window) {
        $scope.name = "Kitty";
        $scope.greet = function() {
            $window.alert("Hello " + $scope.name);
        };
    });
</script>
</body>
</html>
```
 

### 三、Forgiving（宽容，容错？）

　　表达式求值对undefined和null是宽容的。在javascript中，当a不是object的时候，对a.b.c求值，那么将会抛出一个异常。有时候这对于通用语言来说是合理的，而表达式求值主要用于数据绑定，一般形式如下：

{{a.b.c}}
 　　如果a不存在，没有任何显示似乎比抛出异常更加合理（除非我们等待服务端响应，不一会儿就会被定义）。如果表达式求值时不够宽容，那么我们如此混乱地写绑定代码：

{{((a||{}).b||{}).c}}    //这……
 　　相似地，引用一个函数a.b.c()时，如果它是undefined或者null，那么简单地返回undefined。

 

### 四、没有控制流程语句（No Control Flow Statements）

　　我们不可以在表达式中写流程控制语句。背后的原因是，angular的核心体系是应用的逻辑应当在controller（的scope）里面，而不是在view里面。如果我们需要在视图表达式中加入条件分支、循环或者抛出异常的话，可以委托javascript方法去代替（可以调用scope中的方法）。

 

### 五、过滤器（Filters）

　　当我们向用户呈现数据时，我们可能需要将数据从原始格式转换为友好（可读性强）的格式。例如，我们有一个数据对象需要在显示给用户之前根据地域进行格式化。我们可以将表达式传递给一连串的过滤器，如：

name | uppercase
 　　这表达式求值器可简单地传递name的值到uppercase过滤器中。

　　链式过滤器使用这种语法：

value | filter1 | filter2
 　　我们也可以传送用冒号分割的参数到filter中，例如，以两位小数的格式显示123：

123 | number:2
 

### 六、前缀”$”

　　我们可能会感到奇怪，前缀”$”的意义是什么？它是angular为了使本身的API名称能够区别于其他的API而使用的一个简单的前缀（防止冲突）。如果angular不使用$，那么对a.length()求值将返回undefined。因为a和angular本身都没有定义这个属性。

　　考虑到angular将来的版本可能会选择增加length这个方法，这将令这个表达式的行为发生改变。更糟糕的是，我们开发者可能会创建一个length属性，那么将与angular发生冲突。这个问题存在因为angular通过增加方法扩展了当前存在的对象。通过加入前缀”$”，angular保留了特定的namespace，所以angular的开发者与使用angular的开发者都可以和谐共处。