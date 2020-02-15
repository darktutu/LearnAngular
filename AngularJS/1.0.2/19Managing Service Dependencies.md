AngularJs学习笔记--Managing Service Dependencies
原版地址：http://docs.angularjs.org/guide/dev_guide.services.managing_dependencies

　　angular允许service将其他service声明为依赖，使用在自身实例化时使用的构造函数中。

　　为了声明依赖，我们需要在工厂方法声明中指定它们，并且在工厂方法中通过$inject属性（字符串标识数组）或者使用array notation。

　　通常$inject属性声明可以被丢弃（即http://www.cnblogs.com/lcllao/archive/2012/10/16/2726967.html中提到的隐式依赖注入，但这个是实验属性，在而且在压缩混淆后会失效，慎用！）。

使用array notation
function myModuleCfgFn ($provide) {
       $provide.factory(‘myService’,[‘dep1’,’dep2’,function(dep1,dep2){}]);
}
使用$inject属性
    function myModuleCfgFn($provide) {
           var myServiceFactory = function(dep1, dep2) {};
           myServiceFactory.$inject = ['dep1', 'dep2'];
           $provide.factory('myService', myServiceFactory);
    }
使用隐式DI（不兼容压缩混淆的代码）
function myModuleCfgFn($provide) {
　　$provide.factory('myService', function(dep1, dep2) {});
}
       下面有一个例子，里面有两个service，它们之间存在依赖关系，以及其他一些angular提供的service。

 

``` js
    /**
    * batchLog service 允许消息在内存中形成队列，50秒flush一次。
    *
    * @param {*} message Message to be logged.
    */
    function batchLogModule($provide){
    　　$provide.factory('batchLog', ['$timeout', '$log', function($timeout, $log) {
    　　　　var messageQueue = [];
    　　　　function log() {
    　　　　　　if (messageQueue.length) {
    　　　　　　　　$log('batchLog messages: ', messageQueue);
    　　　　　　　　messageQueue = [];
    　　　　　　}
    　　　　　　$timeout(log, 50000);
    　　　　}
    　　　　log(); 
    　　　　return function(message) {
    　　　　　　messageQueue.push(message);
    　　　　}
    　　}]);
    　　/**
    　　* routeTemplateMonitor监控每一个route的变化，每个比阿奴啊都会通过batchLog service记录下来
    　　*/
    　　$provide.factory('routeTemplateMonitor',
    　　　　['$route', 'batchLog', '$rootScope',
    　　　　function($route, batchLog, $rootScope) {
    　　　　　　$rootScope.$on('$routeChangeSuccess', function() {
    　　　　　　　　batchLog($route.current ? $route.current.template : null);
    　　　　　　});
    　　}]);
    }
    // 获得主service，运行应用（监听事件）
　　angular.injector([batchLogModule]).get('routeTemplateMonitor');
```
例子中需要注意的事项：

batchLog service依赖angular内置的$timeout(http://docs.angularjs.org/api/ng.$timeout)与$log services(http://docs.angularjs.org/api/ng.$log)，实现通过console.log批量log消息。
routeTemplateMonitor service依赖内置的$route（http://docs.angularjs.org/api/ng.$route） service与我们自定义的batchLog service。
我们两个service都使用工厂方法签名以及array notation来注释inject，声明它们的依赖。array中的字符串标识的顺序与工厂方法签名（参数）中的顺序必须一致，这十分重要。除非在工厂方法参数中使用隐式依赖声明，否则，injector将根据array中字符串的顺序决定inject哪一个服务。