# AngularJs --Modules
原版地址：http://code.angularjs.org/1.0.2/docs/guide/module

 

### 一、什么是Module？

　　很多应用都有一个用于初始化、加载（wires是这个意思吗？）和启动应用的main方法。angular应用不需要main方法，作为替代，module提供有指定目的的声明式，描述应用如何启动。这样做有几项优点：

* 这过程是声明描述的，更加容易读懂。
* 在单元测试中，不需要加载所有module，这对写单元测试很有帮助。
* 额外的module可以被加载到情景测试中，可以覆盖一些设置，帮助进行应用的端对端测试（end-to-end test）。
* 第三方代码可以作为可复用的module打包到angular中。
* module可以通过任意顺序或并行加载（取决于模块执行的延迟性，due to delayed nature of module execution）。
 

### 二、The Basics（基础）

　　我们很迫切想知道如何让Hello World module能够工作。下面有几个关键的事情要注意：

module API（http://code.angularjs.org/1.0.2/docs/api/angular.Module）
注意的提及的在<html ng-app=”myApp”>中的myApp module，它让启动器启动我们定义的myApp module。
``` html
<!DOCTYPE HTML>
<html lang="zh-cn" ng-app="myApp">
<head>
    <meta charset="UTF-8">
    <title>basics</title>
    <style type="text/css">
        .ng-cloak {
            display: none;
        }
    </style>
</head>
<body>
<div class="ng-cloak">
    {{'Kitty' | greet}}
</div>
<script src="../angular-1.0.1.js" type="text/javascript"></script>
<script type="text/javascript">
    var simpleModule = angular.module("myApp", []);
    simpleModule.filter("greet", function () {
        return function(name) {
            return "Hello " + name + " !";
        }
    });
</script>
</body>
</html>
```
 

### 三、（Recommended Setup）推荐设置

　　虽然上面的例子很简单，它不属于大规模的应用。我们建议将自己的应用按照如下建议，拆分为多个module：

- service module，用于声明service。
- directive module，用于声明directive。
- filter module，用于声明filter。
应用级别的module，依赖上述的module，并且包含初始化的代码。
　　这样划分的理由是，当我们在测试的时候，往往需要忽略那些让测试变得困难的初始化代码。通过将代码分成独立的module，在测试中就可以很容易地忽略那些代码。这样，我们就可以更加专注在加载相应的module进行测试。

　　上面的只是一个建议，可以随意地按照自己的需求制定。

 

### 四、Module Loading & Dependencies（模块加载和依赖）

　　module是配置（configuration）的集合，执行在启动应用的进程中应用的块（blocks）。在它的最简单的形式中，由两类block组成：

　　1.配置块（configuration blocks）：在provider注册和配置的过程中执行的。只有provider和constant（常量？）可以被注入（injected）到configuration blocks中。这是为了避免出现在service配置完毕之前service就被执行的意外。

　　2.运行块（run blocks）：在injector创建完成后执行，用于启动应用。只有实例（instances）和常量（constants）可以被注入到run block中。这是为了避免进一步的系统配置在程序运行的过程中执行。

``` js
angular.module('myModule', []).

    config(function(injectables) { // provider-injector

    // 这里是config block的一个例子

    // 我们可以根据需要，弄N个这样的东东

    // 我们可以在这里注入Providers (不是实例，not instances)到config block里面

    }).

    run(function(injectables) { // instance-injector

    // 这里是一个run block的例子

    // 我们可以根据需要，弄N个这样的东东

    // 我们只能注入实例（instances ）（不是Providers）到run block里面

});
```
 

　　a) Configuration Blocks（配置块）

　　有一个方便的方法在module中，它相当于config block。例如：

``` js
angular.module('myModule', []).
  value('a', 123).
    factory('a', function() { return 123; }).
    directive('directiveName', ...).
    filter('filterName', ...);

// 等同于

angular.module('myModule', []).
  config(function($provide, $compileProvider, $filterProvider) {
    $provide.value('a', 123)
    $provide.factory('a', function() { return 123; })
    $compileProvider.directive('directiveName', ...).
    $filterProvider.register('filterName', ...);
});
```
　　configuration blocks被应用的顺序，与它们的注册的顺序一致。对于常量定义来说，是一种额外的情况，即放置在configuration blocks开头的常量定义。

 

　　b) Run Blocks（应用块）

　　run block是在angular中最接近main方法的东东。run block是必须执行，用于启动应用的代码。它将会在所有service配置、injector创建完毕后执行。run block通常包含那些比较难以进行单元测试的代码，就是因为这个原因，这些代码应该定义在一个独立的module中，让这些代码可以在单元测试中被忽略。

 

　　c) Dependencies（依赖）

　　一个module可以列出它所依赖的其他module。依赖一个module，意味着被请求（required）的module（被依赖的）必须在进行请求（requiring）module（需要依赖其他module的module，请求方）加载之前加载完成。换一种说法，被请求的module的configuration block会在请求的module的configuration block执行前执行（before the configuration blocks or the requiring module，这里的or怎么解释呢？）。对于run block也是这个道理。每一个module只能够被加载一次，即使有多个其他module需要（require）它。

 

　　d) Asynchronous Loading（异步加载）

　　module是管理$injector配置的方法之一，不用对加载脚本到VM做任何事情。现在已经有现成的项目专门用于处理脚本加载，也可以用到angular中。因为module在加载的过程中不做任何事情，它们可以按照任意的顺序被加载到VM中。脚本加载器可以利用这个特性，进行并行加载。

 

### 五、Unit Testing（单元测试）

　　在单元测试的最简单的形式中，其中一个是在测试中实例化一个应用的子集，然后运行它们。重要的是，我们需要意识到对于每一个injector，每一个module只会被加载一次。通常一个应用只会有一个injector。但在测试中，每一个测试用例都有它的injector，这意味着在每一个VM中，module会被多次加载。正确地构建module，将对单元测试有帮助，正如下面的例子：

　　在这个例子中，我们准备假设定义如下的module：

``` js
angular.module('greetMod', []).
factory('alert', function($window) {
     return function(text) {
     　　$window.alert(text);
     };

})
.value('salutation', 'Hello')
.factory('greet', function(alert, salutation) {
     return function(name) {
　　     alert(salutation + ' ' + name + '!');
     };
});
```
 
　　让我们写一些测试用例：

``` js
describe('myApp', function() {
    // 加载应用响应的module，然后加载指定的将$window重写为mock版本的测试module，
    // 这样做，当进行window.alert()时，测试器就不会因被真正的alert窗口阻挡而停止
    //这里是一个在测试中覆盖配置信息的例子
    beforeEach(module('greetMod', function($provide) {//这里看来是要将真正的$window替换为以下的东东
    　　$provide.value('$window', {
    　　　　alert: jasmine.createSpy('alert')
    　　});
    }));
     
    // inject()会创建一个injector，并且注入greet和$window到测试中。
    // 测试不需要关心如何写应用，只需要关注如何测试应用。
    it('should alert on $window', inject(function(greet, $window) {
    　　greet('World');
    　　expect($window.alert).toHaveBeenCalledWith('Hello World!');
    }));
    
    // 这里是在测试中通过行内module和inject方法来覆盖配置的方法
    it('should alert using the alert service', function() {
    　　var alertSpy = jasmine.createSpy('alert');
    　　module(function($provide) {
    　　　　$provide.value('alert', alertSpy);
    　　});
    　　inject(function(greet) {
    　　　　greet('World');
    　　　　expect(alertSpy).toHaveBeenCalledWith('Hello World!');
    　　});
    });
});
```