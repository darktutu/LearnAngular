# AngularJs --Understanding the Controller Component
原版地址：http://docs.angularjs.org/guide/dev_guide.mvc.understanding_model

　　

　　在angular中，controller是一个javascript 函数（type/class），被用作扩展除了root scope在外的angular scope（http://www.cnblogs.com/lcllao/archive/2012/09/23/2698651.html）的实例。当我们或者angular通过scope.$new API（http://docs.angularjs.org/api/ng.$rootScope.Scope#$new）创建新的child scope时，有一个选项作为方法的参数传入controller（这里没看明白,只知道controller的第一个参数是一个新创建的scope，有绑定parent scope）。这将告诉angular需要联合controller和新的scope，并且扩展它的行为。

　　controller可以用作：

* 设置scope对象的初始状态。
* 增加行为到scope中。
 

### 一、 Setting up the initial state of a scope object（设置scope对象的初始状态） 

　　通常，当我们创建应用的时候，我们需要为angular scope设置初始化状态。

　　angular将一个新的scope对象应用到controller构造函数（估计是作为参数传进去的意思），建立了初始的scope状态。这意味着angular从不创建controller类型实例（即不对controller的构造函数使用new操作符）。构造函数一直都应用于存在的scope对象。

　　我们通过创建model属性，建立了scope的初始状态。例如：

function GreetingCtrl ($scope) {$scope.greeting = “Hola!”;}
　　“GreetingCtrl”这个controller创建了一个叫“greeting”的，可以被应用到模版中的model。

 

### 二、 Adding Behavior to a Scope Object（在scope object中增加行为）

　　在angular scope对象上的行为，是以scope方法属性的形式，供模版、视图使用。这行为（behavior）可以修改应用的model。

　　正如指引的model章节（http://www.cnblogs.com/lcllao/archive/2012/09/24/2699861.html）讨论的那样，任意对象（或者原始的类型）赋值到scope中，成为了model属性。任何附加到scope中的function，对于模版视图来说都是可用的，可以通过angular expression调用，也可以通过ng event handler directive调用（如ngClick）。

 

### 三、 Using Controllers Correctly

　　一般而言，controller不应该尝试做太多的事情。它应该仅仅包含单个视图所需要的业务逻辑（还有点没转过弯了，一直认为Controller就是个做转发的……）。

　　保持Controller的简单性，常见办法是抽出那些不属于controller的工作到service中，在controller通过依赖注入来使用这些service。这些东西会在向导的Dependency Injection Services章节中讨论。

　　不要在Controller中做以下的事情：

任何类型的DOM操作 - controller应该仅仅包含业务逻辑。DOM操作，即应用的表现逻辑，它的测试难度是众所周知的。将任何表现逻辑放到controller中，大大地影响了应用逻辑的可测试性。angular为了自动操作（更新）DOM，提供的数据绑定（http://docs.angularjs.org/guide/dev_guide.templates.databinding）。如果我们希望执行我们自定义的DOM操作，可以把表现逻辑抽取到directive（http://www.cnblogs.com/lcllao/archive/2012/09/09/2677190.html）中。
Input formatting（输入格式化） - 使用angular form controls （http://www.cnblogs.com/lcllao/archive/2012/09/17/2688127.html）代替。
Output filtering （输出格式化过滤） - 使用angular filters 代替。
执行无状态或有状态的、controller共享的代码 - 使用angular services 代替。
实例化或者管理其他组件的生命周期（例如创建一个服务实例）。
 

### 四、 Associating Controllers with Angular Scope Objects

　　我们可以显式地通过scope.$new关联controller和scope对象，或者隐式地通过ngController directive（http://docs.angularjs.org/api/ng.directive:ngController）或者$route service（http://docs.angularjs.org/api/ng.$route）。

　　1. Controller 构造函数和方法的 Example

　　为了说明controller组件是如何在angular中工作的，让我们使用以下组件创建一个小应用：

一个有两个按钮和一个简单消息的template。
一个由名为”spice”的字符串属性组成的model。
一个有两个设置spice属性的方法的controller。
　　在我们的模版里面的消息，包含一个到spice model的绑定，默认设置为”very”。根据被单击按钮，将spice model的值设置为”chili”或者” jalapeño”，消息会被数据绑定自动更新。

``` html
<!DOCTYPE html>
<html ng-app>
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8"/>
    <title>spicy-controller</title>
    <meta content="IE=edge,chrome=1" http-equiv="X-UA-Compatible">
    <style type="text/css">
        .ng-cloak {
            display: none;
        }
    </style>
</head>
<body class="ng-cloak">
    <div ng-controller="SpicyCtrl">
        <button ng-click="chiliSpicy()">Chili</button>
        <button ng-click="jalapenoSpicy('jalapeño')">Jalapeño</button>
        <p>The food is {{spice}} spicy!</p>
    </div>

    <script src="../angular-1.0.1.js" type="text/javascript"></script>
    <script type="text/javascript">
        function SpicyCtrl($scope) {
            $scope.spice = "very";
            $scope.chiliSpicy = function() {
                $scope.spice = "chili";
            };
            $scope.jalapenoSpicy = function(val) {
                this.spice = val;
            };
        }
    </script>
</body>
</html>
```

　　在上面例子中需要注意的东东：

ngController directive被用作为我们的模版（隐式）创建scope，那个scope会称为SpicyCtrl的参数。
SpicyCtrl只是一个普通的javascript function。作为一个（随意）的命名规则，名称以大写字母开头，并以”Ctrl”或者”Controller”结尾。
对属性赋值可以创建或者更新$scope的model。
controller方法可以通过直接分配到$scope实现创建。（chiliSpicy方法）
controller的两个方法在template中都是可用的（在ng-controller属性所在的元素以及其子元素中都有效）。
注意：之前版本的angular(1.0RC之前)允许我们使用this来代替$scope定义$scope的方法，但这里不再适用。在定义在scope上的方法中，this跟$scope是等价的（angular将this至为scope），但不是在我们的controller构造函数中。
注意：之前版本的angular（1.0RC之前），会自动增加controller的prototype方法到scope中，但现在不会了。所有方法都需要人工加入到scope中。（印象中之前有一个guide，有用过这个。还没更新-_-!）
　　

　　controller方法可以带参数的，正如下面例子所示：

``` html
<!DOCTYPE html>
<html ng-app>
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8"/>
    <title>controller-method-aruments</title>
    <meta content="IE=edge,chrome=1" http-equiv="X-UA-Compatible">
    <style type="text/css">
        .ng-cloak {
            display: none;
        }
    </style>
</head>
<body class="ng-cloak">
    <div ng-controller="SpicyCtrl">
        <input ng-model="customSpice" value="wasabi"/>
        <button ng-click="spicy(customSpice)">customSpice</button>
        <br/>
        <button ng-click="spicy('Chili')">Chili</button>
        <p>The food is {{spice}} spicy!</p>
    </div>

    <script src="../angular-1.0.1.js" type="text/javascript"></script>
    <script type="text/javascript">
        function SpicyCtrl($scope) {
            $scope.spice = "very";
            $scope.spicy = function(spice) {
                $scope.spice = spice;
            };
        }
    </script>
</body>
</html>
```
　　注意那个SpicyCtrl controller现在只定义了一个有一个参数”spice”、叫”spicy”的方法。template可以引用controller方法并为它传递常量字符串或model值。

 

　　Controller继承在angular是基于scope继承的。让我们看看下面的例子：

``` html
<!DOCTYPE html>
<html ng-app>
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8"/>
    <title>controller-inheritance</title>
    <meta content="IE=edge,chrome=1" http-equiv="X-UA-Compatible">
    <style type="text/css">
        .ng-cloak {
            display: none;
        }
    </style>
</head>
<body class="ng-cloak">
    <div ng-controller="MainCtrl">
        <p>Good {{timeOfDay}}, {{name}}!</p>
        <div ng-controller="ChildCtrl">
            <p>Good {{timeOfDay}}, {{name}}!</p>
            <p ng-controller="BabyCtrl">Good {{timeOfDay}}, {{name}}!</p>
        </div>
    </div>

    <script src="../angular-1.0.1.js" type="text/javascript"></script>
    <script type="text/javascript">
        function MainCtrl($scope) {
            $scope.timeOfDay = 'Main时间';
            $scope.name = 'Main名称';
        }

        function ChildCtrl($scope) {
            $scope.name = 'Child名称';
        }

        function BabyCtrl($scope) {
            $scope.timeOfDay = 'Baby时间';
            $scope.name = 'Baby名称';
        }
    </script>
</body>
</html>
```
　　注意我们如何嵌套3个ngController directive到模版中的。为了我们的视图，这模版结构将会导致4个scope被创建：

root scope。
MainCtrl scope，包含timeOfDay和name model。
ChildCtrl scope，覆盖了MainCtrl scope的name model，继承了timeOfDay model。
BabyCtrl scope，覆盖了MainCtrl scope 的timeOfDay以及ChildCtrl scope的name。
　　继承的工作，在controller和model中是一样的。所以我们前一个例子中，所有model可以通过controller被重写。

　　注意：在两个Controller之间标准原型继承不是如我们所想地那样工作的，因为正如我们之前提到的，controller不是通过angular直接初始化的，但相反地，apply了那个scope对象。（controllers are not instantiated directly by angular, but rather are applied to the scope object，这里跟之前一样，我还是没理解。）

 

### 五、 Testing Controller

　　虽然有很多方法去测试controller，最好的公约之一，如下面所示，需要注入$rootScope和$controller。（测试需要配合jasmine.js）

``` html
<!DOCTYPE html>
<html ng-app>
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8"/>
    <title>controller-test</title>
    <meta content="IE=edge,chrome=1" http-equiv="X-UA-Compatible">
    <link rel="stylesheet" href="../jasmine.css">
    <style type="text/css">
        .ng-cloak {
            display: none;
        }
    </style>
</head>
<body class="ng-cloak">

<script src="../angular-1.0.1.js" type="text/javascript"></script>
<script src="../angular-scenario-1.0.1.js" type="text/javascript"></script>
<script src="../jasmine.js" type="text/javascript"></script>
<script src="../jasmine-html.js" type="text/javascript"></script>
<script src="../angular-mocks-1.0.1.js" type="text/javascript"></script>
<script type="text/javascript">
    function MyController($scope) {
        $scope.spices = [
            {"name":"pasilla", "spiciness":"mild"},
            {"name":"jalapeno", "spiceiness":"hot hot hot!"},
            {"name":"habanero", "spiceness":"LAVA HOT!!"}
        ];

        $scope.spice = "habanero";
    }
    describe("MyController function", function () {
        describe("MyController", function () {
            var scope;
            beforeEach(inject(function ($rootScope, $controller) {
                scope = $rootScope.$new();
                var ctrl = $controller(MyController, {$scope:scope});
            }));

            it('should create "cpices" model with 3 spices', function () {
                expect(scope.spices.length).toBe(3);
            });

            it('should set the default value of spice', function () {
                expect(scope.spice).toBe("habanero");
            });
        });
    });

    (function () {
        var jasmineEnv = jasmine.getEnv();
        jasmineEnv.updateInterval = 1000;

        var trivialReporter = new jasmine.TrivialReporter();

        jasmineEnv.addReporter(trivialReporter);

        jasmineEnv.specFilter = function (spec) {
            return trivialReporter.specFilter(spec);
        };

        var currentWindowOnload = window.onload;

        window.onload = function () {
            if (currentWindowOnload) {
                currentWindowOnload();
            }
            execJasmine();
        };

        function execJasmine() {
            jasmineEnv.execute();
        }

    })();

</script>
</body>
</html>
```
 

　　如果我们需要测试嵌套的controller，我们需要在test中创建与DOM里面相同的scope继承关系。

``` html
<!DOCTYPE html>
<html ng-app>
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8"/>
    <title>controller-test</title>
    <meta content="IE=edge,chrome=1" http-equiv="X-UA-Compatible">
    <link rel="stylesheet" href="../jasmine.css">
    <style type="text/css">
        .ng-cloak {
            display: none;
        }
    </style>
</head>
<body class="ng-cloak">

<script src="../angular-1.0.1.js" type="text/javascript"></script>
<script src="../angular-scenario-1.0.1.js" type="text/javascript"></script>
<script src="../jasmine.js" type="text/javascript"></script>
<script src="../jasmine-html.js" type="text/javascript"></script>
<script src="../angular-mocks-1.0.1.js" type="text/javascript"></script>
<script type="text/javascript">
    function MainCtrl($scope) {
        $scope.timeOfDay = 'Main时间';
        $scope.name = 'Main名称';
    }

    function ChildCtrl($scope) {
        $scope.name = 'Child名称';
    }

    function BabyCtrl($scope) {
        $scope.timeOfDay = 'Baby时间';
        $scope.name = 'Baby名称';
    }

    describe("MyController", function () {
        var mainScope,childScope,babyScope;
        beforeEach(inject(function ($rootScope, $controller) {
            mainScope = $rootScope.$new();
            var mainCtrl = $controller(MainCtrl, {$scope:mainScope});
            childScope = mainScope.$new();
            var childCtrl  = $controller(ChildCtrl, {$scope:childScope});
            babyScope = childScope.$new();
            var babyCtrl  = $controller(BabyCtrl, {$scope:babyScope});
        }));

        it('should have over and selected', function () {
            expect(mainScope.timeOfDay).toBe("Main时间");
            expect(mainScope.name).toBe("Main名称");
            expect(childScope.timeOfDay).toBe("Main时间");
            expect(childScope.name).toBe("Child名称");
            expect(babyScope.timeOfDay).toBe("Baby时间");
            expect(babyScope.name).toBe("Baby名称");
        });
    });

    (function () {
        var jasmineEnv = jasmine.getEnv();
        jasmineEnv.updateInterval = 1000;

        var trivialReporter = new jasmine.TrivialReporter();

        jasmineEnv.addReporter(trivialReporter);

        jasmineEnv.specFilter = function (spec) {
            return trivialReporter.specFilter(spec);
        };

        var currentWindowOnload = window.onload;

        window.onload = function () {
            if (currentWindowOnload) {
                currentWindowOnload();
            }
            execJasmine();
        };

        function execJasmine() {
            jasmineEnv.execute();
        }

    })();

</script>
</body>
</html>
```