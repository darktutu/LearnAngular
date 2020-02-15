# AngularJs --Dependency Injection（DI，依赖注入）
原版地址：http://code.angularjs.org/1.0.2/docs/guide/di

 

### 一、Dependency Injection（依赖注入）

　　依赖注入（DI）是一个软件设计模式，处理代码如何得到它所依赖的资源。

　　关于DI更深层次的讨论，可以参观Dependency Injection（http://en.wikipedia.org/wiki/Dependency_injection），Inversion of Control（http://martinfowler.com/articles/injection.html），也可以参观软件设计模式的书。

　　1. DI in a nutshell（简说DI）

　　object或者function，只能够通过以下三种方式获取他们依赖的资源：

　　　　1) 可以通过new运算符创建依赖的资源。

　　　　2) 可以通过全局变量查找依赖的资源。

　　　　3) 可以通过参数传入依赖的资源。

　　1、2两种方式，并不是最佳的，因为它们对依赖关系进行hard code，这使得修改依赖关系时，不是不可能，但会变得比较复杂。这对于测试来说尤其是个问题，通常在独立测试时，希望能够提供模拟的依赖资源。

　　第3种方法相对来说最可行，因为它去除了从组件（component）中定位依赖的责任。依赖仅仅交给组件就可以了。

``` js
function SomeClass(greeter) {
     this.greeter = greeter
}

SomeClass.prototype.doSomething = function(name) {
     this.greeter.greet(name);
}
```
　　上面的例子，SomeClass不用关心定位greeter这个依赖，它仅仅在运行时传递greeter。
　　这样是比较合适的，但它将获取依赖资源的责任交给了负责构建SomeClass的代码那里。

　　为了管理创建依赖的责任，每一个angular应用都有一个injector（http://code.angularjs.org/1.0.2/docs/api/angular.injector）。injector是一个服务定位器，负责定位并创建依赖的资源。

　　请求依赖，解决了hard code的问题，但它意味着injector需要贯穿整个应用。传递injector，会破坏Law of Demeter（http://baike.baidu.com/view/823220.htm）。为了纠正这个问题，我们将依赖查找的责任转给injector。

　　上面说了那么多，看看下面经我修改过的例子，合并了原文的两个例子，分别在angular内、外使用inject：

``` html
<!DOCTYPE HTML>
<html lang="zh-cn" ng-app="MainApp">
<head>
    <meta charset="UTF-8">
    <title>injector</title>
</head>
<body>
<div ng-controller="MyController">
    <button ng-click="sayHello()">Say Hello</button>
</div>
<script src="../angular-1.0.1.js" type="text/javascript"></script>
<script type="text/javascript">
    //创建OtherModule这个module，相当于外部的module
    var otherModule = angular.module("OtherModule", []);
    //教injector如何创建"greeter"
    //注意，greeter本身需要依赖$window
    otherModule.factory("greeter", function ($window) {
        //这里是一个工厂方法，负责创建greet服务
        return {
            greet:function (text) {
                $window.alert(text);
            }
        };
    });
    //下面展示在非当前module中，通过injector调用greet方法：
    //从module中创建新的injector
    //这个步骤通常由angular启动时自动完成。
    //必须引入'ng'，angular的东东
    //故意颠倒顺序，暂时证实这玩意的顺序是无所谓的。。
    var injector = angular.injector(['OtherModule','ng']);
    //请求greeter这个依赖。
    var g = injector.get("greeter");
    //直接调用它~
    g.greet("Hi~My Little Dada~");

    //这里是当前的主app，需要依赖OtherModule
    var mainApp = angular.module("MainApp", ["OtherModule"]);
    //留意Controller的定义函数的参数，在这里直接注入$scope、greeter。
    // greeter服务是在OtherModule中的
    mainApp.controller("MyController",function MyController($scope,greeter) {
            $scope.sayHello = function() {
                greeter.greet("Hello Kitty~~");
            };
        }
    );
    //ng-controller已经在背后默默地做了这个事情
    //injector.instantiate(MyController);

</script>
</body>
</html>
```
　　注意，因为有ng-controller，初始化了MyController，它可以满足MyController的所有依赖需要，让MyController无须知道injector的存在。这是一个最好的结果。应用代码简单地请求它所需要的依赖而不需要处理injector。这样设置，不会打破Law of Demeter。

 

### 二、Dependency Annotation（依赖注释，说明依赖的方式）

　　injector如何知道什么服务需要被注入呢？

　　应用开发者需要提供被injector用作解决依赖关系的注释信息。所有angular已有的API函数，都引用了injector，每一个文档中提及的API都是这样。下面是用服务名称信息注释我们的代码的三个等同的方法。

　　1. Inferring Dependencies（隐含依赖）

　　这是获取依赖资源的最简单的方式，但需要假定function的参数名称与依赖资源的名称一致。

function MyController($scope, greeter) {
     ...
}
　　函数的injector，可以通过检查函数定义并提取函数名称，猜测需要注入的service的名称（functionName.toString()，RegExp）。在上面的例子中，$scope和greeter是两个需要被注入到函数的服务（名称也一致）。
　　虽然这样做很简单，但这方法在javascript混淆压缩后就行不通了，因为参数名称会被改变。这让这个方式只能对pretotyping（产品可用性原型模拟测试法，http://www.pretotyping.org/，http://tech.qq.com/a/20120217/000320.htm）和demo应用有作用。

　　2. $inject Annotation（$inject注释）

　　为了允许脚本压缩器重命名函数的方法后，仍然能够注入正确的服务，函数必须通过$inject属性来注释依赖。$inject属性是一个需要注入的服务的名称的数组。

``` js
var MyController = function(renamed$scope, renamedGreeter) {

     ...

}
//这里依赖的东东，如果不在当前的module中，它还是不认识的。
//需要在当前module中先依赖对应的module。跟之前的例子差不多。但我不知道这是不是正确的方法。
MyController.$inject = ['$scope', 'greeter'];
```
　　需要小心的是，$inject的顺序需要与函数声明的参数顺序保持一致。
　　这个注释方法，对于controller声明来说是有用的，因为它与函数一起指定注释信息。

　　3. inline Annotation（行内注释）

　　有时候，不方便使用$inject注释的方式，例如注释directive的时候。

　　例如：

someModule.factory('greeter', function($window) {

    ...;

});
　　因为需要临时变量（防止压缩后不能使用），所以代码会膨胀为：
var greeterFactory = function(renamed$window) {
    ...;
};
greeterFactory.$inject = ['$window'];
someModule.factory('greeter', greeterFactory);
　　由于这样（代码膨胀），angular还提供了第三种注释风格：

someModule.factory('greeter', ['$window', function(renamed$window) {
     ...;
}]);
　　记住，所有注释风格都是等价的，可以被用在支持injection的angular中的任何地方。
 
### 三、Where can I user DI？

　　DI遍及整个angular。它通常使用在controller和factory方法中。

　　1. DI in controllers

　　controller是负责（描述）应用行为的类。建议的controller声明方法是：

``` js
var MyController = function(dep1, dep2) {
     ...
}
MyController.$inject = ['dep1', 'dep2'];
MyController.prototype.aMethod = function() {
     ...
}
```
　　2. Factory methods
　　factory方法是负责创建大多数angular对象。例如directive、service、filter。factory方法注册在module中，建议的factory声明方法是：

``` js
angualar.module('myModule', []).
    config(['depProvider', function(depProvider){
    　　...
    }]).
    factory('serviceId', ['depService', function(depService) {
    　　...
    }]).
    directive('directiveName', ['depService', function(depService) {
    　　...
    }]).
    filter('filterName', ['depService', function(depService) {
    　　...
    }]).
    run(['depService', function(depService) {
    　　...
}]);
```