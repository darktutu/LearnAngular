# AngularJs --Forms
原版地址：http://code.angularjs.org/1.0.2/docs/guide/forms

 

　　控件（input、select、textarea）是用户输入数据的一种方式。Form（表单）是这些控件的集合，目的是将相关的控件进行分组。

　　表单和控件提供了验证服务，所以用户可以收到无效输入的提示。这提供了更好的用户体验，因为用户可以立即获取到反馈，知道如何修正错误。请记住，虽然客户端验证在提供良好的用户体验中扮演重要的角色，但是它可以很简单地被绕过，因此，客户端验证是不可信的。服务端验证对于一个安全的应用来说仍然是必要的。

### 一、Simple form

　　理解双向数据绑定的关键directive是ngModel。ngModel提供了从model到view和view到model的双向数据绑定。并且，它还向其他directive提供API，增强它们的行为。

``` html
<!DOCTYPE HTML>
<html lang="zh-cn" ng-app="SimpleForm">
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
<div ng-controller="MyCtrl" class="ng-cloak">
    <form novalidate>
        名字: <input ng-model="user.name" type="text"><br/>
        Email: <input ng-model="user.email" type="email"><br/>
        性别: <input value="男" ng-model="user.gender" type="radio">男
        <input value="女" ng-model="user.gender" type="radio">女
        <br/>
        <button ng-click="reset()">还原上次保存</button>
        <button ng-click="update(user)">保存</button>
    </form>
    <pre>form = {{user | json}}</pre>
    <pre>saved = {{saved | json}}</pre>
</div>
<script src="../angular-1.0.1.js" type="text/javascript"></script>
<script type="text/javascript">
    var app = angular.module("SimpleForm", []);
    app.controller("MyCtrl", function ($scope,$window) {
        $scope.saved = {};

        $scope.update = function(user) {
            $scope.saved = angular.copy(user);
        };

        $scope.reset = function() {
            $scope.user = angular.copy($scope.saved);
        };

        $scope.reset();
        //不合法的值将不会进入user
    });
</script>
</body>
</html>
```
 

### 二、Using CSS classes

　　为了让form以及控件、ngModel富有样式，可以增加以下class：

ng-valid
ng-invalid
ng-pristine（从未输入过）
ng-dirty（输入过）
　　下面的例子，使用CSS去显示每个表单控件的有效性。例子中，user.name和user.email都是必填的，但当它们修改过之后（dirty），背景将呈现红色。这确保用户不会直到与表单交互之后（提交之后？），发现未能满足其有效性，才为一个错误而分心。

``` html
<!DOCTYPE HTML>
<html lang="zh-cn" ng-app="CSSClasses">
<head>
    <meta charset="UTF-8">
    <title>CSSClasses</title>
    <style type="text/css">
        .ng-cloak {
            display: none;
        }
        .css-form input.ng-invalid.ng-dirty {
            background-color: #fa787e;
        }
        .css-form input.ng-valid.ng-dirty {
            background-color: #78fa89;
        }
    </style>
</head>
<body>
<div ng-controller="MyCtrl" class="ng-cloak">
    <form novalidate class="css-form" name="formName">
        名字: <input ng-model="user.name" type="text" required><br/>
        Email: <input ng-model="user.email" type="email" required><br/>
        性别: <input value="男" ng-model="user.gender" type="radio">男
        <input value="女" ng-model="user.gender" type="radio">女
        <br/>
        <button ng-click="reset()">RESET</button>
        <button ng-click="update(user)">SAVE</button>
    </form>
    <pre>form = {{user | json}}</pre>
    <pre>saved = {{saved | json}}</pre>
</div>
<script src="../angular-1.0.1.js" type="text/javascript"></script>
<script type="text/javascript">
    var app = angular.module("CSSClasses", []);
    app.controller("MyCtrl", function ($scope,$window) {
        $scope.saved = {};
        $scope.update = function(user) {
            $scope.saved = angular.copy(user);
        };

        $scope.reset = function() {
            $scope.user = angular.copy($scope.saved);
        };

        $scope.reset();
        //不合法的值将不会进入user
    });
</script>
</body>
</html>
```
 

### 三、Binding to form and control state

　　在angular中，表单是FormController的一个实例。表单实例可以随意地使用name属性暴露到scope中（这里没看懂，scope里面没有一个跟form的name属性一直的property，但由于有“document.表单名”这种方式，所以还是可以获取到的）。相似地，控件是NgModelController的实例。控件实例可以相似地使用name属性暴露在form中。这说明form和控件（control）两者的内部属性对于使用标准绑定实体（standard binding primitives）绑定在视图中的这个做法是可行的。

　　这允许我们通过以下特性来扩展上面的例子：

RESET按钮仅仅在form发生改变之后才可用。
SAVE按钮仅仅在form发生改变而且输入有效的情况下可用。
为user.email和user.agree定制错误信息。
``` html
<!DOCTYPE HTML>
<html lang="zh-cn" ng-app="ControlState">
<head>
    <meta charset="UTF-8">
    <title>ControlState</title>
    <style type="text/css">
        .ng-cloak {
            display: none;
        }
        .css-form input.ng-invalid.ng-dirty {
            background-color: #fa787e;
        }
        .css-form input.ng-valid.ng-dirty {
            background-color: #78fa89;
        }
    </style>
</head>
<body>
<div ng-controller="MyCtrl" class="ng-cloak">
    <form novalidate class="css-form" name="formName">
        名字: <input ng-model="user.name" name="userName" type="text" required><br/>
        <div ng-show="formName.userName.$dirty&&formName.userName.$invalid">
            <span>请填写名字</span>
        </div>
        Email: <input ng-model="user.email" name="userEmail" type="email" required><br/>
        <div ng-show="formName.userEmail.$dirty && formName.userEmail.$invalid">提示:
            <span ng-show="formName.userEmail.$error.required">请填写Email</span>
            <span ng-show="formName.userEmail.$error.email">这不是一个有效的Email</span>
        </div>
        性别: <input value="男" ng-model="user.gender" type="radio">男
        <input value="女" ng-model="user.gender" type="radio">女
        <br/>
        <input type="checkbox" ng-model="user.agree" name="userAgree" required/>我同意：
        <input type="text" ng-show="user.agree" ng-model="user.agreeSign" required/>
        <br/>
        <div ng-show="!formName.userAgree || !user.agreeSign">请同意并签名~</div>
        <button ng-click="reset()" ng-disabled="isUnchanged(user)">RESET</button>
        <button ng-click="update(user)" ng-disabled="formName.$invalid || isUnchanged(user)">SAVE</button>
    </form>
    <pre>form = {{user | json}}</pre>
    <pre>saved = {{saved | json}}</pre>
</div>
<script src="../angular-1.0.1.js" type="text/javascript"></script>
<script type="text/javascript">
    var app = angular.module("ControlState", []);
    app.controller("MyCtrl", function ($scope,$window) {
        $scope.saved = {};
        $scope.update = function(user) {
            $scope.saved = angular.copy(user);
        };

        $scope.reset = function() {
            $scope.user = angular.copy($scope.saved);
        };

        $scope.isUnchanged = function(user) {
            return angular.equals(user, $scope.saved);
        };

        $scope.reset();
        //不合法的值将不会进入user
    });
</script>
</body>
</html>
```
 

### 四、Custom Validation

　　angular为大多数公共的HTML表单域（input，text,number,url,email,radio,checkbox）类型提供了实现，也有一些为了表单验证的directive（required，pattern,，inlength，maxlength，min，max）。

　　可以通过定义增加定制验证函数到ngModel controller（这里是连在一起的ngModelController吗？）中的directive来定义我们自己的验证插件。为了得到一个controller，directive如下面的例子那样指定了依赖（directive定义对象中的require属性）。

Model到View更新 - 无论什么时候Model发生改变，所有在ngModelController.$formatters（当model发生改变时触发数据有效性验证和格式化转换）数组中的function将排队执行，所以在这里的每一个function都有机会去格式化model的值，并且通过NgModelController.$setValidity（http://code.angularjs.org/1.0.2/docs/api/ng.directive:ngModel.NgModelController#$setValidity）修改控件的验证状态。
View到Model更新 - 一个相似的方式，无论任何时候，用户与一个控件发生交互，将会触发NgModelController.$setViewValue。这时候轮到执行NgModelController$parsers（当控件从DOM中取得值之后，将会执行这个数组中所有的方法，对值进行审查过滤或转换，也进行验证）数组中的所有方法。
　　在下面的例子中我们将创建两个directive。

第一个是integer，它负责验证输入到底是不是一个有效的整数。例如1.23是一个非法的值，因为它包含小数部分。注意，我们通过在数组头部插入（unshift）来代替在尾部插入（push），这因为我们想它首先执行并使用这个控件的值（估计这个Array是当作队列来使用的），我们需要在转换发生之前执行验证函数。
第二个directive是smart-float。他将”1.2”和”1,2”转换为一个合法的浮点数”1.2”。注意，我们在这不可以使用HTML5的input类型”number”，因为浏览器不允许用户输入我们预期的非法字符，如”1,2”（它只认识”1.2”）。
``` html
<!DOCTYPE HTML>
<html lang="zh-cn" ng-app="CustomValidation">
<head>
    <meta charset="UTF-8">
    <title>CustomValidation</title>
    <style type="text/css">
        .ng-cloak {
            display: none;
        }
        .css-form input.ng-invalid.ng-dirty {
            background-color: #fa787e;
        }
        .css-form input.ng-valid.ng-dirty {
            background-color: #78fa89;
        }
    </style>
</head>
<body>
<div class="ng-cloak">
    <form novalidate class="css-form" name="formName">
        <div>
            大小(整数 0 - 10):<input integer type="number" ng-model="size" name="size" min="0" max="10"/>{{size}}<br/>
            <span ng-show="formName.size.$error.integer">这不是一个有效的整数</span>
            <span ng-show="formName.size.$error.min || formName.size.$error.max">
                数值必须在0到10之间
            </span>
        </div>
        <div>
            长度（浮点数）：
            <input type="text" ng-model="length" name="length" smart-float/>
            {{length}}<br/>
            <span ng-show="formName.length.0error.float">这不是一个有效的浮点数</span>
        </div>
    </form>
</div>
<script src="../angular-1.0.1.js" type="text/javascript"></script>
<script type="text/javascript">
    var app = angular.module("CustomValidation", []);
    var INTEGER_REGEXP = /^\-?\d*$/;
    app.directive("integer", function () {
        return {
            require:"ngModel",//NgModelController
            link:function(scope,ele,attrs,ctrl) {
                ctrl.$parsers.unshift(function (viewValue) {//$parsers，View到Model的更新
                    if(INTEGER_REGEXP.test(viewValue)) {
                        //合格放心肉
                        ctrl.$setValidity("integer", true);
                        return viewValue;
                    }else {
                        //私宰肉……
                        ctrl.$setValidity("integer", false);
                        return undefined;
                    }
                });
            }
        };
    });
    var FLOAT_REGEXP = /^\-?\d+(?:[.,]\d+)?$/;
    app.directive("smartFloat", function () {
        return {
            require:"ngModel",
            link:function(scope,ele,attrs,ctrl) {
                ctrl.$parsers.unshift(function(viewValue) {
                    if(FLOAT_REGEXP.test(viewValue)) {
                        ctrl.$setValidity("float", true);
                        return parseFloat(viewValue.replace(",", "."));
                    }else {
                        ctrl.$setValidity("float", false);
                        return undefined;
                    }
                });
            }
        }
    });
</script>
</body>
</html>
```
 

### 五、Implementing custom form control (using ngModel)

　　angular实现了所有HTML的基础控件（input，select，textarea），能胜任大多数场景。然而，如果我们需要更加灵活，我们可以通过编写一个directive来实现自定义表单控件的目的。

　　为了制定控件和ngModel一起工作，并且实现双向数据绑定，它需要：

实现render方法，是负责在执行完并通过所有NgModelController.$formatters方法后，呈现数据的方法。
调用$setViewValue方法，无论任何时候用户与控件发生交互，model需要进行响应的更新。这通常在DOM事件监听器里实现。
　　可以查看$compileProvider.directive（http://www.cnblogs.com/lcllao/archive/2012/09/09/2677190.html）获取更多信息。

　　下面的例子展示如何添加双向数据绑定特性到可以编辑的元素中。

 

``` html
 <!DOCTYPE HTML>
<html lang="zh-cn" ng-app="CustomControl">
<head>
    <meta charset="UTF-8">
    <title>CustomControl</title>
    <style type="text/css">
        .ng-cloak {
            display: none;
        }
        div[contenteditable] {
            cursor: pointer;
            background-color: #D0D0D0;
        }
    </style>
</head>
<body ng-controller="MyCtrl">
<div class="ng-cloak">
    <div contenteditable="true" ng-model="content" title="点击后编辑">My Little Dada</div>
    <pre>model = {{content}}</pre>
    <button ng-click="reset()">reset model tirgger model to view($render)</button>
</div>
<script src="../angular-1.0.1.js" type="text/javascript"></script>
<script type="text/javascript">
    var app = angular.module("CustomControl", []);
    app.controller("MyCtrl", function ($scope) {
        $scope.reset = function() {
            $scope.content = "My Little Dada";
            //在这里如何获取NgModelController呢？如果你们知道，希望可以指点指点！谢谢
        };
    });
    app.directive("contenteditable", function () {
        return {
            require:"ngModel",
            link:function (scope, ele, attrs, ctrl) {
                //view -> model
                ele.bind("blur keyup",function() {
                    scope.$apply(function() {
                        console.log("setViewValue");
                        ctrl.$setViewValue(ele.text());
                    });
                });

                //model -> view
                ctrl.$render = function(value) {
                    console.log("render");
                    ele.html(value);
                };
                //读取初始值
                ctrl.$setViewValue(ele.text());
            }
        };
    });
</script>
</body>
</html>
```