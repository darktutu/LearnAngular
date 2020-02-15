# AngularJs --Creating Services
原版地址：http://docs.angularjs.org/guide/dev_guide.services.creating_services

　　虽然angular提供许多有用的service，在一些特别的应用中，我们会发现编写自定义service是很有用的。如果我们想做这件事，我们首先要在module中注册一个service工厂方法，可以通过Module.factory api（http://docs.angularjs.org/api/angular.module）或者在module配置方法中直接通过$provide api（http://docs.angularjs.org/api/AUTO.$provide）。

　　所有angular service都参与到DI（http://www.cnblogs.com/lcllao/archive/2012/09/23/2699401.html）中，既可以通过angular DI系统（injector）中使用名称（id）注册自己，也可以通过在其他工厂方法中声明对已存在的service的依赖。

### 一、Registering Services

　　为了注册一个service，我们必须拥有一个module，并且使这个server成为这个module的一部分。然后，我们可以通过Module api或者在module配置函数中注册service。下面的伪代码将展示这两种注册方式。

　　使用angular.module api:

``` js
var myModule = angular.module(‘myModule’,[]);
myModule.factory(‘serviceId’,function() {
       var someService;
       //工厂方法体，构建someService
       return someService;

});
```
　　使用$provide service：

``` js
angular.module(‘myModule’,[],function($provide) {
       $provide.factory(‘serviceId’,function() {
              var someService;
              //工厂方法体，构建someService
              return someService;
        });
});    
```
　　注意，我们无须注册一个服务实例，相反地，工厂方法会在它被调用的时候被实例化。

### 二、Dependencies

　　service不仅仅可以被依赖，更可以拥有自己的依赖。可以在工厂方法的参数中指定依赖。阅读（http://www.cnblogs.com/lcllao/archive/2012/09/23/2699401.html）更多关于angular中的DI、数组标记的用途和$inject属性，让DI声明更加简洁。(Read more about the DI in Angular and the use of array notation and $inject property to make DI annotation minification-proof……)

　　下面是一个非常简单的service例子。这个服务依赖$window service（通过工厂方法参数传递），而且只有一个方法。这个service简单地储存所有通知，在第三个之后，这个service会通过window.alert显示所有通知。
 
``` html
<!DOCTYPE HTML>
<html lang="zh-cn" ng-app="MainApp">
<head>
    <meta charset="UTF-8">
    <title>services</title>
</head>
<body>
<div ng-controller="MyController">
    <input type="text" ng-model="msg"/>
    <button ng-click="saveMsg()">save msg</button>
    <ul>
        <li ng-repeat="msg in msgs">{{msg}}</li>
    </ul>
</div>
<script src="../angular-1.0.1.js" type="text/javascript"></script>
<script type="text/javascript">
    var app = angular.module("MainApp",[],function($provide) {
        $provide.factory("notify",["$window","$timeout",function(win,timeout) {
            var msgs = [];
            return function(msg) {
                msgs.push(msg);
                if(msgs.length==3) {
                    timeout(function() {
                        win.alert(msgs.join("\n"));
                        msgs = [];
                    },10);
                }
            }
        }])
    });
    app.controller("MyController",function($scope,notify) {
        $scope.msgs = [];
        $scope.saveMsg  = function() {
            this.msgs.push(this.msg);
            notify(this.msg);
            this.msg = "";
        };
    });
</script>
</body>
</html>
```
### 三、Instantiating Angular Services

　　所有在angular中的service都是延迟实例化的（lazily）。这意味着service仅仅在其他依赖它的已实例化的service或者应用组件中被依赖时，才会创建。换句话说，angular直到服务被直接或者间接请求时候，才会实例化service。

### 四、Services as singletons

　　最后，我们必须意识到所有angular service都是一个单例应用。这意味着每一个injector中有且只有一个给定service的实例。由于angular是极其讨厌破坏global state的，所以创建多个injector，使每一个都有指定service的实例是可行的，除了在测试中有强烈的需求外，一般很少有这样的需要。