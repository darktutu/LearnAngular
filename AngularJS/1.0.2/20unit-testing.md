# AngularJs --unit-testing
原版地址：http://docs.angularjs.org/guide/dev_guide.unit-testing

 

　　javascript是一门动态类型语言，这给她带来了很强的表现能力，但同时也使编译器几乎不能给开发者提供任何帮助。因为这个原因，我们感受到编写任何javascript代码都必须有一套强大完整的测试。angular拥有许多功能，让我们更加容易地测试我们的应用。我们应该没有借口不去写测试（这个嘛……）。

### 一、 It is all about NOT mixing concerns（全部都关于避免代码关系变得复杂……）

　　单元测试，正如名称那样，是关于测试单个“单元”的代码。单元测试努力解答这些问题：我对逻辑的考虑是否已经正确？排序方法得出的结果是否正确？为了解答这些问题，将这些问题独立出来显得尤其重要。这是因为当我们在测试排序方法的时候，我们不想关心其他相关的片段，例如DOM元素或者发起XHR请求获取数据等。明显地，通常比较难做到在典型的项目中单独调用一个函数。导致这个问题的原因是，开发者通常把关系弄得很复杂，最终让一个代码片段看起来可以做所有事情。它通过XHR获取数据，对数据进行排序，然后操纵DOM。与angular一起，我们可以更加容易地写出较好的代码，所以angular为我们提供XHR（我们可以模拟它）的依赖注入，angular还创建允许我们对model进行排序而不需要操作DOM的抽象。所以，到最后，我们可以简单地写一个排序方法，然后通过测试用例创建数据集合，供排序方法测试时使用，然后判断结果model是否符合预期。测试无须等待XHR、者创建对应的DOM和判断函数是否正确操作DOM。angular的核心思想包含代码的可测试性，但同时也要求我们去做正确的事情。angular致力于简化做正确事情的方法，但angular不是魔法，这意味着我们如果不遵循以下的几点，我们最终可能会得出一个不可测试的应用。

1. Dependency Inject

　　有许多办法可以获得依赖的资源：1）我们可以使用new操作符；2）我们使用一个众所周知的方式，被称为” 全局单例”；3）我们可以向registry service请求（但我们如何取得一个registry？可以查看后面的章节）；4）我们可以期待它会被传递过来。

　　上面列出的方法中，只有最后一个是可测试的，让我们看看为什么：

1) Using the new operator

　　使用new操作符时基本上没有错误，但问题是通过new调用构造函数将会永久地将调用方与type绑定起来。举个例子，我们尝试实例化一个XHR对象，以让我们可以从服务器获得一些数据。

```
function MyClass() {
     this.doWork = function() {
         var xhr = new XRH();
         xhr.open(method,url,true);
         xhr.onreadystatechange = function() {…};
         xhr.send();
　　}
}
```
　　问题来了，在测试时，我们通常需要实例化一个可以返回测试数据或者网络错误的虚拟的XHR。通过调用new XHR()，我们永久地绑定了真实的XHR，并且没有一个很好的方法去替代它。当然，有一个糟糕的补救办法，有很多理由可以证明那是一个糟糕的想法：

var oldXHR = XHR;
XHR = new MockXHR() {};
myClass.doWork();
//判断MockXHR是否通过正常的参数进行调用
XHR = oldXHR;//如果忘了这一步，很容易会发生悲催的事情。
2) Global look-up

　　解决问题的另外一个方法是在一个众所周知的地方获取依赖的资源。

function MyClass() {
     this.doWork = function() {
           global.xhr({…});
    };
}
　　没有创建新依赖对象的实例的情况下，问题基本上与new一致，除了那个悲催的补丁以外，没有一个很好的方法可以再测试时拦截global.xhr的调用。测试的最基本的问题是global变量需要改为调用虚拟的方法而被修改。想进一步了解它的坏处，可以参观这里：http://misko.hevery.com/code-reviewers-guide/flaw-brittle-global-state-singletons/

　　上面的代码比较难去测试，所以我们必须修改global state：

var oldXHR = global.xhr;
global.xhr = function mockXHR(){…};
var myClass = new MyClass();
//判断MockXHR是否通过正常的参数进行调用
global.xhr = oldXHR;//如果忘了这一步，很容易会发生悲催的事情。
3) Service Registry

　　拥有一个包含所有service的registry的话，似乎可以解决问题，然后，在测试代码中替换所需要的service。

```
function MyClass() {
     var serviceRegistry = ???;
     this.doWork = function() {
         var xhr = serviceRegistry.get(“xhr”);
　　   …
    };
}
```
　　但是，serviceRegistry来自哪里？if it is: * new-ed up, the the test has no chance to reset the services for testing * global look-up, then the service returned is global as well (but resetting is easier, since there is only one global variable to be reset)（这里后面的文字跟乱码一样……没看懂）

　　根据这个方法，将上面的Class修改为如下的方式：

```
var oldServiceLocator = global.serviceLocator;
global.serviceLocator.set('xhr', function mockXHR() {});
var myClass = new MyClass();
myClass.doWork();
//判断MockXHR是否通过正常的参数进行调用
global.serviceLocator = oldServiceLocator; //如果忘了这一步，很容易会发生悲催的事情。
```
4) Passing in Dependencies

　　最后，依赖资源可以被传入。

function MyClass(xhr) {
     this.doWork = function() {
         xhr({…});
    };
}
　　这个是首选的方式，因为代码无须理会xhr是从哪来的，也不关心谁创建了传进来的xhr。因此，类的创建者与类的使用者可以分开编码，这将创建的责任从逻辑中分离出来，这就是依赖注入的概述。

　　这个class很容易测试，在测试中我们可以这样写：

function xhrMock(args) {…}
var myClass = new MyClass(xhrMock);
myClass.doWrok();
//做一些判断……
通过这个测试代码，我们可以意识到没有任何全局变量被破坏。
　　angular附带的dependency-injection（http://www.cnblogs.com/lcllao/archive/2012/09/23/2699401.html），通过这种方式编写的代码，更加容易编写测试代码，如果我们想编写可测试性强的代码，我们最好使用它。

2. Controllers

　　逻辑使每一个应用都是唯一的，这就是我们想去测试的。如果我们的逻辑里面混杂着DOM的操作，这将会跟下面的例子一样难测试：

```
function PasswordController() {
    // 获取DOM对象的引用
    var msg = $('.ex1 span');
    var input = $('.ex1 input');
    var strength;

    this.grade = function() {
         msg.removeClass(strength);
         var pwd = input.val();
         password.text(pwd);
         if (pwd.length > 8) {
               strength = 'strong';
         } else if (pwd.length > 3) {
               strength = 'medium';
         } else {
               strength = 'weak';
         }
        msg.addClass(strength).text(strength);
    }
}
```
　　上面的代码在测试时会遇到问题，因为它需要我们的执行测试时候，需要有正确的DOM。测试代码会如下：

```
var input = $('<input type="text"/>');
var span = $('<span>');
$('body').html('<div class="ex1">').find('div').append(input).append(span);
var pc = new PasswordController();
input.val('abc');
pc.grade();
expect(span.text()).toEqual('weak');
$('body').html('');
```
　　在angular中，controller严格地将DOM操作逻辑分离出来，将大大降低编写测试用例的难度，看看下面的例子：

```
function PasswordCntrl($scope) {
    $scope.password = '';
    $scope.grade = function() {
         var size = $scope.password.length;
         if (size > 8) {
                   $scope.strength = 'strong';
         } else if (size > 3) {
                   $scope.strength = 'medium';
         } else {
                   $scope.strength = 'weak';
         }
    };
}
```
　　测试代码直截了当：

var pc = new PasswordController($scope);
pc.password('abc');
pc.grade();
expect($scope.strength).toEqual('weak');
　　值得注意的是，测试代码不仅仅更加间断，而且更加容易追踪。我们一直说测试用例是在讲故事，而不是判断其他不相关的东西。

3. Filters

　　filter(http://docs.angularjs.org/api/ng.$filter)是用于将数据转换为对用户友好的格式。它们很重要，因为它们将转换格式的责任从应用逻辑中分离出来，进一步简化了应用逻辑。

```
myModule.filter('length', function() {
    return function(text){
         return (''+(text||'')).length;
    }
});

var length = $filter('length');
expect(length(null)).toEqual(0);
expect(length('abc')).toEqual(3);
```
4. Directives

5. Mocks

6. Global State Isolation

7. Preferred way of Testing

8. JavascriptTestDriver

9. Jasmine

10.   Sample project