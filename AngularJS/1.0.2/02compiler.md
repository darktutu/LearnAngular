# AngularJs html compiler

### 一、总括

　　Angular的HTML compiler允许开发者自定义新的HTML语法。compiler允许我们对任意HTML元素或属性，甚至是新的HTML标签、属性（如 `<beautiful girl=”cf”></beautiful >` ）附加行为。Angular将这些附加行为称为directives。

　　HTML有很多专门格式化静态文档的预定义HTML样式结构（可以告诉浏览器如何显示标记的内容）。假设某东东需要被居中，而我们不需要教浏览器如何去做（此处省略N字）。我们只需要简单地对需要居中的标签加入align=”center”即可。这就是声明式语言（declarative language）的牛X之处。

　　但是声明式语言也有它的局限性，即你不能告诉浏览器如何处理在预定义范围外的语法。例如，我们不能很简单地告诉浏览器如何让文本在浏览器的1/3处对齐。所以，我们正需要一个让浏览器与时俱进，学学新语法的途径。

　　Angular预先绑定了一些对构建应用有帮助的directives。我们也可以自己创建属于自己应用的独特的directives。这些directive扩展将成为我们自己的应用的“特定领域语言”（Domain Specific Language）。

　　这些编译将仅仅发生在浏览器端，无须服务端或者预编译步骤。

### 二、Compiler

　　Compiler作为Angular的一个服务（Service），负责遍历DOM结构，寻找属性。编译过程分成两个阶段：
1. 编译（Compile）：遍历DOM节点树，收集所有directives。返回结果是一个链接函数（linking function）。
2. 链接（Link）：将directives绑定到一个作用域（scope）中，创建一个实况视图（live view）。在scope中的任何改变，将会在视图中得到体现（更新视图）；任何用户对模版的活动（改变），将会体现在scope model中（双向绑定）。这使得scope model能够反映正确的值。

　　一些directives，诸如ng-repeat，会为每一个在集合（collection）中的元素复制一次特定的元素（组合）。编译和链接两个阶段，使性能得以提升。因为克隆出来的模版(template)只需要编译一次，然后为每一个集合中的元素进行一次链接（类似模版缓存）。

三、Directive

　　Directive是一个行为，在编译过程中遇到特定的HTML结构时，它会被触发。Directives可以放置在元素的name、attribute、class甚至注释中。以下是几种引用ng-bind（一个内置directive）的方法：

``` html
<span ng-bind="exp"></span>

<span class="ng-bind: exp;"></span>

<ng-bind></ng-bind>

<!-- directive: ng-bind exp -->

```
　　Directive只是一个当编译器在DOM中遇到时会执行的一个函数（function）。directive API文档中有详细讲解如何创建一个directive。

下面是一个样例，可以让一个元素跟你的鼠标玩躲猫猫……

``` html
<!DOCTYPE html>
<html lang="zh-cn" ng-app="HideAnkSeek">
<head>
    <meta charset="UTF-8">
    <title>躲猫猫</title>
    <style type="text/css">
        .ng-cloak {
            display: none;
        }
    </style>
</head>
<body>
<span class="ng-cloak" wildcat>一碰我就跑~~来点我啊~~</span>
<script src="../angular-1.0.1.js" type="text/javascript"></script>
<script type="text/javascript">
    angular.module("HideAnkSeek", []).directive("wildcat", function ($document) {
        var maxLeft = 400,maxTop = 300;
        var msg = ["我闪~~", "抓我呀~~~", "雅蠛蝶~~", "噢耶~~", "你真逊~！","就差那么一点点了！","继续吧~~总有一天我会累的"];
        return function (scope, element, attr) {
            element.css({
                "position":"absolute",
                "border":"1px solid green"
            });
            element.bind("mouseenter", function (event) {
                element.css({
                    "left":parseInt(Math.random() * 10000 % maxLeft) + "px",
                    "top":parseInt(Math.random() * 10000 % maxTop) + "px"
                }).text(msg[parseInt(Math.random() * 10000 % msg.length)]);
            }).bind("click",function (event) {
                        element.text("噢My Lady Gaga。。。被你逮到了。。。");
                        element.unbind("mouseenter");
                    });
        };
    });
</script>
</body>
</html>
```
　　在任意元素中添加“wildcat”这个属性，将会使该元素拥有新的行为。就这样，我们教会了浏览器如何处理会躲猫猫的元素（放心，你不是在某个房间，你不会挂的-_-!）。我们通过这一途径扩展了浏览器的“词汇量”。对于任意一个熟悉HTML规则的人，这算是一个比较自然的方式。

 

 

现在已经夜深了，明天继续。。。广告之后见

===================华丽的分割线=======================

四、理解视图（View）

　　外面有许多模版系统，它们通常都通过模版字符串与数据进行连接，生成最终的HTML字符串，并将结果通过innerHTML属性写入某元素里。



 　　这意味着任何数据发生改变时，都需要重新将数据、模版合并成字符串，然后当作innerHTML写回对应元素中。这里存在一些问题：(这里直译实在没法懂..唯有YY)假设有这么一个场景，模版里包含输入框。用户对在输入框进行输入，模版同步更新。普通模版通过innerHTML、字符串与数据连接的方式更新视图，这样会打断用户的输入，体验不好。

　　Angular是与众不同的。Angular编译器（compiler）通过directives处理DOM，而不是通过处理字符串模版。处理结果是一个与scope model组合并生成实时模版的链接函数（linking function）。视图与scope model的绑定对我们来说是透明的。开发者无须为更新视图、model做任何动作。而且，因为没有使用innerHTML更新视图模版，所以用户输入不会被打断。此外，angular directives不仅可以绑定文本值，而且还可以是拥有行为的结构（behavioral constructs）。



　　Angular的这个处理方式，产生了一个稳定的DOM。这意味着在DOM元素的生命周期里，一直与某model的实例绑定着，这个关系不会发生改变。这也意味着代码可以保持对某DOM对象的引用，对其注册事件函数，并且这个引用不会被模版数据合并所销毁。