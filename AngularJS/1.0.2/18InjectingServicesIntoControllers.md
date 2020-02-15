# AngularJs --Injecting Services Into Controllers
原版地址：http://docs.angularjs.org/guide/dev_guide.services.injecting_controllers

　　把service当作被依赖的资源加载到controller中的方法，与加载到其他服务中的方法很相似。

　　由于javascript是一个动态语言，DI不能弄明白应该通过static types（like in static typed languages）注入哪一个service。因此，我们需要通过$inject属性指定service名称， 它是一个包含需要注入的service名称的字符串数组。service ID顺序的重要性：工厂方法中的参数顺序，与service在数组中的顺序一致。工厂方法的参数名称并不重要，但是按照惯常的做法，他们与service ID一一匹配，下面将讨论这样做的好处。

1.显式依赖注入

``` js
function myController($scope,$loc,$log) {
　　$scope.firstMethod = function() {
    　　//使用$location service
      　$loc.setHash();
   };
　　$scope.secondMethod = function() {
       //使用$log service
       $log.info(‘…’)
　　};
}
myController.$inject = [‘$location’,’$log’];    
```
例子：

``` html
<!DOCTYPE HTML>
<html lang="zh-cn" ng-app="MainApp">
<head>
    <meta charset="UTF-8">
    <title>explicit-inject-service</title>
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
            //这里是服务依赖服务，通过这种显式的方式，参数名可以乱填，但顺序要对应
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
        }]);
    });

    function MyController($s,$noti) {
        //这里是controller依赖服务，通过这种显式的方式，参数名可以乱填，但顺序要对应
        $s.msgs = [];
        $s.saveMsg  = function() {
            this.msgs.push(this.msg);
            $noti(this.msg);
            this.msg = "";
        };
    }
    //指定注入的东东
    //也可以参考http://www.cnblogs.com/lcllao/archive/2012/10/16/2725317.html里面的例子
    MyController.$inject = ['$scope','notify'];

</script>
</body>
</html>
```
2. 隐式依赖注入

　　angular DI的一个新特性，允许通过参数名称决定依赖。让我们重写上面的例子，展示如何隐式注入$window、$scope与notify service。

例子：

``` html
<!DOCTYPE HTML>
<html lang="zh-cn" ng-app="MainApp">
<head>
    <meta charset="UTF-8">
    <title>implicit-inject-service</title>
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
        $provide.factory("notify",function($window,$timeout) {
            //服务依赖服务，隐式依赖，名称一致即可
            var msgs = [];
            return function(msg) {
                msgs.push(msg);
                if(msgs.length==3) {
                    $timeout(function() {
                        $window.alert(msgs.join("\n"));
                        msgs = [];
                    },10);
                }
            }
        });
    });

    function MyController($scope,notify) {
        //服务依赖服务，隐式依赖，名称一致即可
        $scope.msgs = [];
        $scope.saveMsg  = function() {
            this.msgs.push(this.msg);
            notify(this.msg);
            this.msg = "";
        };
    }
</script>
</body>
</html>
```
 

　　虽然这样很方便，但是假如我们需要压缩、混淆我们的代码，这可能会导致参数名称被更改，遇到这种情况的时候，我们还是需要使用显式声明依赖的方式。