　Directive是教HTML玩一些新把戏的途径。在DOM编译期间，directives匹配HTML并执行。这允许directive注册行为或者转换DOM结构。

　　Angular自带一组内置的directive，对于建立Web App有很大帮助。继续扩展的话，可以在HTML定义领域特定语言（domain specific language ,DSL)。

一、在HTML中引用directives

　　Directive有驼峰式（camel cased）的风格的命名，如ngBind（放在属性里貌似用不了~）。但directive也可以支蛇底式的命名（snake case），需要通过:（冒号）、-（减号）或_（下划线）连接。作为一个可选项，directive可以用“x-”或者“data-”作为前缀，以满足HTML验证需要。这里列出directive的合法命名：

ng:bind
ng-bind
ng_bind
x-ng-bind
data-ng-bind
　　Directive可以放置于元素名、属性、class、注释中。下面是引用myDir这个directive的等价方式。（但很多directive都限制为“属性”的使用方式）

复制代码
<span my-dir="exp"></span>

<span class="my-dir: exp;"></span>

<my-dir></my-dir>

<!-- directive: my-dir exp -->
复制代码
　　Directive可以通过多种方式引用，下面列出N种等价的方式：

复制代码
<!DOCTYPE HTML>
<html lang="zh-cn" ng-app>
<head>
    <meta charset="UTF-8">
    <title>invoke-directive</title>
    <style type="text/css">
        .ng-cloak {
            display: none;
        }
    </style>
</head>
<body>
<div ng-controller="MyCtrl">
    Hello <input ng-model="name"/><hr/>
    ngBind="name" 这个用不了~~ <span ngBind="name"></span><br/>
    ng:bind="name"<span ng:bind="name"></span><br/>
    ng_bind="name"<span ng_bind="name"></span><br/>
    ng-bind="name"<span ng-bind="name"></span><br/>
    data-ng-bind="name"<span data-ng-bind="name"></span><br/>
    x-ng-bind="name"<span x-ng-bind="name"></span><br/>
</div>
<script src="../angular-1.0.1.js" type="text/javascript"></script>
<script type="text/javascript">
    function MyCtrl($scope) {
        $scope.name = "beauty~~";
    }
</script>
</body>
</html>
复制代码
 

二、String interpolation

　　在编译过程中，compiler通过$interpolate服务匹配文本与属性中的嵌入表达式（如{{something}}）。这些表达式将会注册为watches，并且作为digest cycle(之前不是digest-loop吗？！)的一部分，一同更新。下面是一个简单的interpolation：

<img src="img/{{username}}.jpg"/>Hello {{username}}!
三、Compilation process, and directive matching

　　HTML“编译”的三个步骤：

　　1. 首先，通过浏览器的标准API，将HTML转换为DOM对象。这是很重要的一步。因为模版必须是可解析（符合规范）的HTML。这里可以跟大多数的模版系统做对比，它们一般是基于字符串的，而不是基于DOM元素的。

　　2. 对DOM的编译（compilation）是通过调用$comple()方法完成的。这个方法遍历DOM，对directive进行匹配。如果匹配成功，那么它将与对应的DOM一起，加入到directive列表中。只要所有与指定DOM关联的directive被识别出来，他们将按照优先级排序，并按照这个顺序执行他们的compile() 函数。directive的编译函数（compile function），拥有一个修改DOM结构的机会，并负责产生link() function的解析。$compile()方法返回一个组合的linking function，是所有directive自身的compile function返回的linking function的集合。

　　3. 通过上一步返回的linking function，将模版与scope连接起来。这反过来会调用directive自身的linking function，允许它们在元素上注册一些监听器（listener），以及与scope一起建立一些watches。这样得出的结果，是在scope与DOM之间的一个双向、即时的绑定。scope发生改变时，DOM会得到对应的响应。

 

复制代码
    var $compile = ...; // injected into your code

    var scope = ...;

    var html = '<div ng-bind='exp'></div>';

    // Step 1: parse HTML into DOM element
    var template = angular.element(html);

    // Step 2: compile the template
    var linkFn = $compile(template);

    // Step 3: link the compiled template with the scope.
    linkFn(scope);
复制代码
 

四、Reasons behind the compile/link separation

　　在这个时候，你可能会想知道为什么编译进程会划分为compile和linke两个步骤。为了明白这一点，让我们看看一个真实的例子（repeater）

    Hello {{user}}, you have these actions:
    <ul>
        <li ng-repeat="action in user.actions">{{action.description}}</li>
    </ul>
　　简单地讲，之所以分开compile和linke两步，是因为有时候需要在model改变后，对应的DOM结构也需要改变的情况，如repeaters。
　　当上面的例子被编译时，编译器会遍历所有节点以寻找directive。{{user}}是一个interpolation directive的例子。ngRepeat又是另外一个directive。但ngRepeat有一个难点。它需要能够很快地为每一个在users.actions中的action制造出新的li的能力。这意味着它为了满足克隆li并且嵌入特定的action（这里是指user的actions的其中一个值）的目的，需要保持一个干净li元素的拷贝，li元素需要被克隆和插入ul元素。但仅仅克隆li元素是不够的。还需要编译li，以便它的directive（{{action.descriptions}}）能够在正确的scope中被解析。原始的方法，一般会简单地插入一个li元素的拷贝，然后编译它。但编译每一个li元素的拷贝会比较缓慢，因为编译过程需要我们遍历DOM节点树，查找directive并运行它们。如果我们有一个编译，需要通过repeater对100个item进行处理，那么我们将陷入性能问题。

　　问题的解决方案，是将编译过程分解为两个步骤。compile阶段识别出所有directive，并且将它们按照优先级进行排序，在linking阶段将特定的scope与特定的li绑定在一起。

　　ngRepeat将各个li分开编译以防止编译过程落入li元素中。li元素的编译结果是一个包含所有包含在li元素中的directive的linking function，准备与特定li元素的拷贝进行连接。在运行时，ngRepeat监测表达式，并作为一个item，被加入到一个li元素拷贝的数组，为克隆好的li元素创建新的scope，并调用该拷贝对应的link function。

　　总结：

编译函数(compile function) - 编译函数在directive中是比较少见的，因为大多数directive只关心与指定的DOM元素工作，而不是改变DOM元素的模版（DOM自身以及内部的结构）。为了优化性能，一些可以被directive实例共享的操作，可以移动到compile函数中。
连接函数(link function) - 极少directive是没有link function的。link function允许directive在指定的拷贝后的DOM元素实例上注册监听器，也可以将scope中特定内容复制到DOM中。
 

五、写一个directive（简易版）

　　在这个例子里面，我们将建立一个根据输入格式，显示当前时间的directive。

 

复制代码
<!DOCTYPE HTML>
<html lang="zh-cn" ng-app="TimeFormat">
<head>
    <meta charset="UTF-8">
    <title>time-format</title>
</head>
<body>
<div ng-controller="MyCtrl" id="main">
    Date format: <input ng-model="format" type="text"/><hr/>
    <!--下面使用属性x-current-time，是为了试试上面说的合法命名~~current:time、current-time、current_time、data-current-time -_-!!! -->
    Current time is : <span x-current-time="format" id="myFormat"></span><br/>
    <button ng-click="remove()">remove the span</button>
</div>
<script src="../angular-1.0.1.js" type="text/javascript"></script>
<script type="text/javascript">
    angular.module("TimeFormat", [])
        //在TimeFormat应用中注册“currentTime”这个directive的工厂方法
        //前文提到过，依赖注入，可以直接在function的参数中写入，这里注入了$timeout、dataFilter
            .directive("currentTime", function (dateFilter) {
                //这个是上面提到的linking function。(不需要添加compile function，为啥？。。。)
                return function (scope, element, attr) {
                    var intervalId;
                    //更新对应element的text值，即更新时间
                    function updateTime() {
                        element.text(dateFilter(new Date(), scope.format));
                    }
                    //通过watch，监控span对象的currentTime的值(是format这个model值，即input的值！！)
                    //这个方法仅仅在format发生改变的时候执行
                    scope.$watch(attr.currentTime, function (value) {
                        scope.format = value;
                        updateTime();
                    });
                    //当span被去掉的时候，取消更新
                    element.bind("$destroy", function () {
                        clearInterval(intervalId);
                    });

                    intervalId = setInterval(updateTime, 1000);
                };
            }).controller("MyCtrl",function($scope,$rootScope) {
                $scope.format = "M/d/yy h:mm:ss a";
                $scope.remove = function() {
                    var oFormat = document.getElementById("myFormat");
                    if(oFormat) {
                        angular.element(oFormat).remove();//通过这种方式调用remove，可以触发$destroy事件啊！！！试了我N久。。。
                    }
                };
            });
</script>
</body>
</html>
复制代码
 

六、写一个directive（详细版）

　　下面是一个创建directive样例（directive对象定义模版）。想看详细列表，请继续往下看。

复制代码
var myModule = angular.module(...);

     

myModule.directive('directiveName', function factory(injectables) {

    var directiveDefinitionObject = {

    　　priority: 0,

    　　template: '<div></div>',

    　　templateUrl: 'directive.html',

    　　replace: false,

    　　transclude: false,

    　　restrict: 'A',

    　　scope: false,

    　　compile: function compile(tElement, tAttrs, transclude) {

    　　　　return {

    　　　　　　pre: function preLink(scope, iElement, iAttrs, controller) { ... },

    　　　　　　post: function postLink(scope, iElement, iAttrs, controller) { ... }

    　　　　}

    　　},

    　　link: function postLink(scope, iElement, iAttrs) { ... }

    };

    return directiveDefinitionObject;

 });
复制代码
　　在大多数场景下，我们并不需要精确控制，所以上面的定义是可以化简的。定义模版中的每一部分，将在下面章节讲解。在这个章节，我们仅仅关注定义模版的异构体（isomers of this skeleton，没看懂。。。期待大家补充）。

　　简化代码的第一步是依赖默认值。因此，上面的代码可以简化为：

复制代码
    var myModule = angular.module(...);
     
    myModule.directive('directiveName', function factory(injectables) {
    　　var directiveDefinitionObject = {
    　　　　compile: function compile(tElement, tAttrs) {
    　　　　　　return function postLink(scope, iElement, iAttrs) { ... }
    　　　　}
    　　};
    　　return directiveDefinitionObject;
    });
复制代码
　　大多数directive只关心实例，而不是模版转换，所以可以进一步化简（翻译得很勉强。。。期待大家补充）：

  

    var myModule = angular.module(...);
     
    myModule.directive('directiveName', function factory(injectables) {
    　　return function postLink(scope, iElement, iAttrs) { ... }
    });
 

七、工厂方法

　　工厂方法负责创建directive。它仅仅使用一次，就在compiler第一次匹配到directive的时候。你可以在这里执行一些初始化操作。工厂方法通过$injector.invoke执行，让它遵守所有注入声明规则（rules of injection annotation），让其变为可注入的。

 

八、directive定义对象说明

 

　　directive定义对象提供了compiler的结构。属性如下：

 

name - 当前scope的名称，注册时可以使用默认值（不填）。
 

priority（优先级）- 当有多个directive定义在同一个DOM元素时，有时需要明确它们的执行顺序。这属性用于在directive的compile function调用之前进行排序。如果优先级相同，则执行顺序是不确定的（经初步试验，优先级高的先执行，同级时按照类似栈的“后绑定先执行”。另外，测试时有点不小心，在定义directive的时候，两次定义了一个相同名称的directive，但执行结果发现，两个compile或者link function都会执行）。
 

terminal（最后一组）- 如果设置为”true”，则表示当前的priority将会成为最后一组执行的directive。任何directive与当前的优先级相同的话，他们依然会执行，但顺序是不确定的（虽然顺序不确定，但基本上与priority的顺序一致。当前优先级执行完毕后，更低优先级的将不会再执行）。
 

scope - 如果设置为：
 

true - 将为这个directive创建一个新的scope。如果在同一个元素中有多个directive需要新的scope的话，它还是只会创建一个scope。新的作用域规则不适用于根模版（root of the template），因此根模版往往会获得一个新的scope。
 

{}(object hash) - 将创建一个新的、独立(isolate)的scope。”isolate” scope与一般的scope的区别在于它不是通过原型继承于父scope的。这对于创建可复用的组件是很有帮助的，可以有效防止读取或者修改父级scope的数据。这个独立的scope会创建一个拥有一组来源于父scope的本地scope属性(local scope properties)的object hash。这些local properties对于为模版创建值的别名很有帮助（useful for aliasing values for templates -_-!）。本地的定义是对其来源的一组本地scope property的hash映射(Locals definition is a hash of local scope property to its source #&)$&@#)($&@#_)：
 

@或@attr - 建立一个local scope property到DOM属性的绑定。因为属性值总是String类型，所以这个值总是返回一个字符串。如果没有通过@attr指定属性名称，那么本地名称将与DOM属性的名称一直。例如<widget my-attr=”hello {{name}}”>，widget的scope定义为：{localName:’@myAttr’}。那么，widget scope property的localName会映射出”hello {{name}}"转换后的真实值。name属性值改变后，widget scope的localName属性也会相应地改变（仅仅单向，与下面的”=”不同）。name属性是在父scope读取的（不是组件scope）
 

=或=expression（这里也许是attr） - 在本地scope属性与parent scope属性之间设置双向的绑定。如果没有指定attr名称，那么本地名称将与属性名称一致。例如<widget my-attr=”parentModel”>，widget定义的scope为：{localModel:’=myAttr’}，那么widget scope property “localName”将会映射父scope的“parentModel”。如果parentModel发生任何改变，localModel也会发生改变，反之亦然。（双向绑定）
 

&或&attr - 提供一个在父scope上下文中执行一个表达式的途径。如果没有指定attr的名称，那么local name将与属性名称一致。例如<widget my-attr=”count = count + value”>，widget的scope定义为：{localFn:’increment()’}，那么isolate scope property “localFn”会指向一个包裹着increment()表达式的function。一般来说，我们希望通过一个表达式，将数据从isolate scope传到parent scope中。这可以通过传送一个本地变量键值的映射到表达式的wrapper函数中来完成。例如，如果表达式是increment(amount)，那么我们可以通过localFn({amount:22})的方式调用localFn以指定amount的值（上面的例子真的没看懂，&跑哪去了？）。
 

controller - controller 构造函数。controller会在pre-linking步骤之前进行初始化，并允许其他directive通过指定名称的require进行共享（看下面的require属性）。这将允许directive之间相互沟通，增强相互之间的行为。controller默认注入了以下本地对象：
 

$scope - 与当前元素结合的scope
 

$element - 当前的元素
 

$attrs - 当前元素的属性对象
 

$transclude - 一个预先绑定到当前转置scope的转置linking function :function(cloneLinkingFn)。(A transclude linking function pre-bound to the correct transclusion scope)
 

require - 请求另外的controller，传入当前directive的linking function中。require需要传入一个directive controller的名称。如果找不到这个名称对应的controller，那么将会抛出一个error。名称可以加入以下前缀：
 

? - 不要抛出异常。这使这个依赖变为一个可选项。
 

^ - 允许查找父元素的controller
 

restrict - EACM的子集的字符串，它限制directive为指定的声明方式。如果省略的话，directive将仅仅允许通过属性声明：
 

E - 元素名称： <my-directive></my-directive>
 

A - 属性名： <div my-directive=”exp”></div>
 

C - class名： <div class=”my-directive:exp;”></div>
 

M - 注释 ： <!-- directive: my-directive exp -->
 

template - 如果replace 为true，则将模版内容替换当前的HTML元素，并将原来元素的属性、class一并迁移；如果为false，则将模版元素当作当前元素的子元素处理。想了解更多的话，请查看“Creating Widgets”章节（在哪啊。。。Creating Components就有。。。）
 

templateUrl - 与template基本一致，但模版通过指定的url进行加载。因为模版加载是异步的，所以compilation、linking都会暂停，等待加载完毕后再执行。
 

replace - 如果设置为true，那么模版将会替换当前元素，而不是作为子元素添加到当前元素中。（注：为true时，模版必须有一个根节点）
 

transclude - 编译元素的内容，使它能够被directive所用。需要(在模版中)配合ngTransclude使用(引用)。transclusion的优点是linking function能够得到一个预先与当前scope绑定的transclusion function。一般地，建立一个widget，创建isolate scope，transclusion不是子级的，而是isolate scope的兄弟。这将使得widget拥有私有的状态，transclusion会被绑定到父级（pre-isolate）scope中。（上面那段话没看懂。但实际实验中，如果通过<any my-directive>{{name}}</any my-directive>调用myDirective，而transclude设置为true或者字符串且template中包含<sometag ng-transclude>的时候，将会将{{name}}的编译结果插入到sometag的内容中。如果any的内容没有被标签包裹，那么结果sometag中将会多了一个span。如果本来有其他东西包裹的话，将维持原状。但如果transclude设置为’element’的话，any的整体内容会出现在sometag中，且被p包裹）
 

true - 转换这个directive的内容。（这个感觉上，是直接将内容编译后搬入指定地方）
 

‘element’ - 转换整个元素，包括其他优先级较低的directive。（像将整体内容编译后，当作一个整体（外面再包裹p），插入到指定地方）
 

compile - 这里是compile function，将在下面章节详细讲解
 

link - 这里是link function ，将在下面章节详细讲解。这个属性仅仅是在compile属性没有定义的情况下使用。
 

 

九、Compile function

 

function compile ( tElement, tAttrs, transclude) { … }
　　compile function用于处理DOM模版的转换。由于大多数directive都不需要转换模版，所以compile不会经常被使用到。需要compile function的directive，一般是那种需要转换DOM模版的（如ngRepeat），或者是需要异步加载内容的（如ngView）。compile function拥有一下的参数：

tElement - 模版元素 – 使用当前directive的元素。仅仅在当前元素或者当前元素子元素下做模版转换是安全的。
tAttrs - 模版属性 - 标准化的属性，在当前元素声明的，可以在各个directive之间共享。详情请看Attributes章节
transclude – 一个转换用的linking function:  function(scope,cloneLinking).
　　注意：如果template被克隆过，那么template实例和link实例不能是同一个object。为此，在compile function中做任何除了DOM转换以外的事情，是不安全的，这将应用到所有克隆中。特别是，DOM事件监听器的注册的操作，应该在linking function中进行，而不是compile function。

　　compile function 可以有一个返回值，类型可以是function或者object。

返回function – 通常在不需要compile function（为空）时使用，等同于通过link(directive定义模版的属性)注册linking function。
返回包含pre、post属性的object - 允许我们控制在linking phase期间何时调用linking function。详情请看下面关于pre-linking、post-linking function的章节。
 

十、Linking function

function link( scope, iElement, iAttrs, controller) { … }
　　link function负责注册DOM事件监听器，也可以进行DOM的更新操作。link function会在模版克隆操作完毕之后执行。这里存放着directive大多数的逻辑。

scope - Scope - 被directive用于注册watches(http://docs.angularjs.org/api/ng.$rootScope.Scope#$watch)。
iElement - 元素实例 - directive使用的元素。只有在postLink function里面对子元素进行操作，才是安全的。因为子元素已经被link（与model连接吗？！）。
iAttrs - 属性实例 - 标准的当前元素的属性列表。在所有directive的linking function之间分享的。
controller - controller实例 - 如果在当前元素的directive中，有其中一个定义了controller，那么可以在这里获取到该controller的实例。这个controller是在所有directive之间共享的，允许各个directive将controller当作一个它们之间沟通频道。
 

Pre-link function

　　在子元素linked之前执行。在这里做DOM转换是不安全的，因为compiler的linking function在link时可能会定位不到正确的元素。

Post-link function

　　在子元素linked之后执行。在这里执行DOM转换是安全的。

 

十一、Attributes

　　attribute object - 在link()或compile()中作为参数 - 是一个访问以下东东的途径：

标准化的属性名称：由于directive，例如ngBind，可以表现为多种形式，如”ng:bind”、”x-ng-bind”……这个attribute对象允许我们通过标准命名（驼峰式）访问属性。
directive 之间通讯：所有directive共享一个attribute对象实例，使得directive可以通过attribute 对象进行directive之间的内部通讯。
支持interpolation：interpolation属性是分配到attribute对象中，允许其他directive读取该interpolated value。
观察interpolated属性：通过attr.$observe去观察属性值的变化，包括interpolation（例如src=”{{bar}}”）。不单单很有效，而且是简单地获取真实值唯一的办法。因为在linking阶段，interpolation还没被赋值（替换真实值），所以在这时候访问它，结果是undefined。
 

复制代码
<!DOCTYPE HTML>
<html lang="zh-cn" ng-app="DirectiveProperty">
<head>
    <meta charset="UTF-8">
    <title>directive-attribute-test</title>
    <style type="text/css">
        .ng-cloak {
            display: none;
        }
    </style>
</head>
<body ng-controller="MyCtrl">
<input type="text" ng-model="name" value="myName"/>
<p my-attr="123" directive-p2 attr-dd="{{name}}"></p>
<script src="../angular-1.0.1.js" type="text/javascript"></script>
<script type="text/javascript">
    var app = angular.module("DirectiveProperty", []);
    app.controller("MyCtrl", function ($scope) {
        $scope.name = "my little dada";
    });
    var directiveP2 = app.directive("directiveP2", function () {
        return {
            link:function postLink(scope,lEle,lAttr) {                console.log("myAttr:" + lAttr.myAttr);//123
                console.log("myAttr:" + lAttr.attrDd);//undefinded
                lAttr.$observe('attrDd', function(value) {
                    console.log('attrDd has changed value to ' + value); //my little dada
                    //除此以外，还可啥办法可以拿到这个值啊。。。￥&（）@#&￥（@#
                });
            }
        };
    });
</script>
</body>
</html>
复制代码
 

十二、理解transclusion和scope

　　我们经常需要一些可重用的组件。下面是一段伪代码，展示一个简单的dialog组件如何能够工作。

　　
复制代码
<div>

    <button ng-click="show=true">show</button>

    <dialog title="Hello {{username}}."

        visible="show"

        on-cancel="show = false"

        on-ok="show = false; doSomething()">

        Body goes here: {{username}} is {{title}}.

</dialog>
复制代码
　　点击“show”按钮将会打开dialog。dialog有一个标题，与数据“username”绑定，还有一段我们想放置在dialog内部的内容。
　　下面是一段为了dialog而编写的模版定义：

复制代码
<div ng-show="show()">

    <h3>{{title}}</h3>

    <div class="body" ng-transclude></div>

    <div class="footer">

        <button ng-click="onOk()">Save changes</button>

        <button ng-click="onCancel()">Close</button>

    </div>

</div>
复制代码
　　这将无法正确渲染，除非我们对scope做一些特殊处理。

　　第一个我们需要解决的问题是，dialog模版期望title会被定义，而在初始化时会与username绑定。此外，按钮需要onOk、onCancel两个function出现在scope里面。这限制了插件的效能（this limits the usefulness of the widget。。。）。为了解决映射问题，通过如下的本地方法（locals，估计在说directive定义模版中的scope）去创建模版期待的本地变量：

复制代码
scope :{

    title: 'bind', // set up title to accept data-binding

    onOk: 'expression', // create a delegate onOk function

    onCancel: 'expression', // create a delegate onCancel function

    show: 'accessor' // create a getter/setter function for visibility.

}
复制代码
　　在控件scope中创建本地属性，带来了两个问题：
　　1. isolation（属性隔离？） - 如果用户忘记了在控件模版设置title这个元素属性，那么title将会与祖先scope的属性”title”（如有）绑定。这是不可预知、不可取的。

　　2. transclusion - transcluded DOM可以查看控件的locals（isolate scope?），locals会覆盖transclusion中真正需要绑定的属性。在我们的例子中，插件中的title属性破坏了transclusion的title属性。

　　为了解决这个缺乏属性隔离的问题，我们需要为这个directive定义一个isolated scope。isoloted scope不是通过从child scope（这里为啥是child scope？不是parent scope吗？）原型继承的，所以我们无需担心属性冲突问题（作为当前scope的兄弟）。

　　然而，isolated scope带来了一个新问题：如果transcluded DOM 是控件的isolated scope的一个子元素，那么他将不能与任何东西绑定（if a transcluded DOM is a child of the widget isolated scope then it will not be able to bind to anything）。因此，transcluded scope是原始scope的一个子scope，在控件为本地属性而创建isolated scope之前创建的。transcluded与控件isolated scope属于兄弟节点（scope树中）。

　　这也许看起来会有点意想不到的复杂，但这样做让控件用户与控件开发者至少带来了惊喜。（解决了问题）

　　因此，最终的directive定义大致如下：

复制代码
transclude:true,

scope :{

    title: 'bind', // set up title to accept data-binding

    onOk: 'expression', // create a delegate onOk function

    onCancel: 'expression', // create a delegate onCancel function
    
    show: 'accessor' // create a getter/setter function for visibility.

    //我试过这么整，失败……请继续往下看

}
复制代码
　　我尝试把上面的代码拼凑成一个完整的例子。直接复制的话，无法达到预期效果。但稍作修改后，插件可以运行了。
 

复制代码
<!DOCTYPE html>
<html ng-app="Dialog">
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8"/>
    <title>directive-dialog</title>
    <meta content="IE=edge,chrome=1" http-equiv="X-UA-Compatible">
    <script src="../angular.js" type="text/javascript"></script>
</head>
<body>
    <div ng-controller="MyCtrl">
        <button ng-click="show=true">show</button>
        <dialog title="Hello {{username}}"
                visible="{{show}}"
                on-cancel="show=false;"
                on-ok="show=false;methodInParentScope();">
                <!--上面的on-cancel、on-ok，是在directive的isoloate scope中通过&引用的。如果表达式中包含函数，那么需要将函数绑定在parent scope（当前是MyCtrl的scope）中-->
            Body goes here: username:{{username}} , title:{{title}}.
            <ul>
                <!--这里还可以这么玩~names是parent scope的-->
                <li ng-repeat="name in names">{{name}}</li>
            </ul>
        </dialog>
    </div>
    <script type="text/javascript">
        var myModule = angular.module("Dialog", []);
        myModule.controller("MyCtrl", function ($scope) {
            $scope.names = ["name1", "name2", "name3"];
            $scope.show = false;
            $scope.username = "Lcllao";
            $scope.title = "parent title";
            $scope.methodInParentScope = function() {
                alert("记住。。scope里面通过&定义的东东，是在父scope中玩的！！。。。");
            };
        });
        myModule.directive('dialog', function factory() {
            return {
                priority:100,
                template:['<div ng-show="visible">',
                    '    <h3>{{title}}</h3>',
                    '    <div class="body" ng-transclude></div>',
                    '    <div class="footer">',
                    '        <button ng-click="onOk()">OK</button>',
                    '        <button ng-click="onCancel()">Close</button>',
                    '    </div>',
                    '</div>'].join(""),
                replace:false,
                transclude: true,
                restrict:'E',
                scope:{
                    title:"@",//引用dialog标签title属性的值
                    onOk:"&",//以wrapper function形式引用dialog标签的on-ok属性的内容
                    onCancel:"&",//以wrapper function形式引用dialog标签的on-cancel属性的内容
                    visible:"@"//引用dialog标签visible属性的值
                }
            };
        });
    </script>
</body>
</html>
复制代码
 

 

十三、Creating Components

　　我们通常会期望通过复杂的DOM结构替换掉directive（所在的元素？目的大概是使directive内部复杂点，看起来牛点@_@）。这让directive成为使用可复用组件的建立应用的一个快捷方式。

　　下面是一个可复用组件的例子：

复制代码
<!DOCTYPE html>
<html ng-app="ZippyModule">
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8"/>
    <title>ZippyModule</title>
    <meta content="IE=edge,chrome=1" http-equiv="X-UA-Compatible">
    <style type="text/css">
        .zippy {
            border: 1px solid black;
            display: inline-block;
            width: 250px;
        }
        .zippy.opened > .title:before { content: '▼ '; }
        .zippy.opened > .body { display: block; }
        .zippy.closed > .title:before { content: '► '; }
        .zippy.closed > .body { display: none; }
        .zippy > .title {
            background-color: black;
            color: white;
            padding: .1em .3em;
            cursor: pointer;
        }
        .zippy > .body {
            padding: .1em .3em;
        }
    </style>
    <script src="../angular.js" type="text/javascript"></script>
</head>
<body>
    <div ng-controller="MyCtrl">
        Title: <input ng-model="title" type="text"><br/>
        Text: <textarea ng-model="text" ></textarea>
        <hr/>
        <div class="zippy" zippy-title="Details: {{title}}...">{{text}}</div>
    </div>
    <script type="text/javascript">
        var myModule = angular.module("ZippyModule", []);
        myModule.controller("MyCtrl", function ($scope) {
            $scope.title = "这里是标题";
            $scope.text = "这里是内容哇。。。";
        });
        myModule.directive('zippy', function () {
            return {
                template: '<div>' +
                        '   <div class="title">{{title}}</div>' +//这个title属于当前directive isolate scope的property
                        '   <div class="body" ng-transclude></div>' + //这里的东西，获取的是父scope的property咯
                        '</div>',
                replace:true,
                transclude: true,
                restrict:'C',
                scope:{
                    title:"@zippyTitle"//绑定directive元素身上的zippy-title属性
                },
                link:function(scope,element,attrs) {
                    var title = angular.element(element.children()[0]),
                            opened = false;

                    title.bind("click", toogle);
                    element.addClass("closed");

                    function toogle() {
                        opened = !opened;
                        element.removeClass(opened ? "closed" : "opened");
                        element.addClass(opened ? "opened" : "closed");
                    }
                }
            };
        });
    </script>
</body>
</html>
复制代码
 