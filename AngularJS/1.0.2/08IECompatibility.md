# AngularJs --IE Compatibility  
原版地址：http://docs.angularjs.org/guide/ie

 

Internet Explorer Compatibility

 

### 一、总括

　　这文章描述Internet Explorer（IE）处理自定义HTML属性、标签的特性（可以理解为“特别糟糕的性质”）。如果我们打算让应用兼容IE8以及以下的版本，那么可以继续往下看。

 

### 二、Short Version（简述）

　　为了让我们的angular应用在IE上工作，请确保：

　　1. 按需引入JSON.stringify（IE7或以下的都需要这玩意）。我们可以使用JSON2（https://github.com/douglascrockford/JSON-js）或者JSON3（http://bestiejs.github.com/json3/）。

　　2. 不要使用自定义标签，诸如<ng:view>（用属性版代替，如<div ng-view>）。如果还是想使用，则请看第3点。

　　3. 如果你确实想使用自定义标签，那么你必须做以下步骤，让老IE认识你的自定义标签。

   

``` html
<html xmlns:ng="http://angularjs.org">
<head>
<!--[if lte IE 8]>
<script>
    document.createElement('ng-include');
    document.createElement('ng-pluralize');
    document.createElement('ng-view');
  

    // Optionally these for CSS
    document.createElement('ng:include');
    document.createElement('ng:pluralize');
    document.createElement('ng:view');
</script>
<![endif]-->
</head>
<body>
    ...
</body>
</html>
```
需要关注的是：

xmls:ng - 命名空间 - 对于每一个我们计划使用的自定义标签，都需要有一个命名空间。
document.createElement(“自定义标签名称”) - 自定义标签名称的创建 - 因为这是旧版IE一个问题，我们需要通过IE判断注释（<!--[if lte IE 8]>…<![endif]-->）来特殊处理。对于每一个没有命名空间或者非HTML默认标签，都需要进行这种预定义，以让IE不会犯傻（例如没样式…）。
 

### 三、Long Version（详情）

　　IE对于非标准HTML标签的处理会有问题。这大致可以氛围两类（有、无命名空间），每一类都有他自己的一个解决方式。

如果标签名称以”my:”开头的话，将被当作命名空间，必须要一个想对应的命名空间定义（<html xmlns:my=”ignored”>）。
如果标签没有命名空间（xx:开头），但并非一个标准的HTML的话，需要通过document.createElement(“标签名称”)进行声明。
如果我们打算对自定义标签定义样式的话，我们必须使用document.createElement(“标签名称”)来进行自定义，regardless of XML命名空间（实验证明，regardless of XML namespace意思很有可能是：不用管有命名空间的自定义标签）。
 

### 四、The Good News（好消息）

　　好消息是，这个限制仅仅是对于元素名称的，对属性名称没影响。所以不需要对自定义属性（<div> my-tag your:tag></div>）做特殊处理。

 

### 五、What happens if I fail to do this？（没做这些处理的话，会发生什么事呢？！）

　　假设我们有一个非标准的HTML标签（对于my:tag或者my-tag都有一样的结果。但测试过后，发现命名空间方式不会有这种烦恼）。

<html>
    <body>
        <mytag>some text</mytag>
    </body>
</html>
　　
　　一般来说，将会转换为一下的DOM结构：
#document
    +- HTML
        +- BODY
            +- mytag
                +- #text: some text
 　　我们期望的，是BODY元素有一个mytag子元素，mytag又有一个文本子元素”some text”。

 

　　但IE不是这么干的（如果做了纠正措施，则不包括在内）！

#document
    +- HTML
        +- BODY
             +- mytag
             +- #text: some text
             +- /mytag
 　　在IE里面，BODY将会有3个孩子元素：

　　1. 一个自闭合的” mytag”，与<br/>类似。末尾的“/”是可选的，但<br>标签不允许有任何子元素，所以浏览器将其分为<br>、some text、</br>三个兄弟元素，而不是单独的<br>元素中含有some text元素。

　　2. 一个文本节点“some text”。这本来应该是mytag元素的子节点，不是它的兄弟节点。

　　3. 一个错误的自闭合标签” /mytag”，说它错误，是因为元素名称不允许含有”/”字符（在最后应该是允许的<br/>）。此外，闭合元素不应该是DOM的一部分（不应该以元素形式出现），因为它只用作勾画DOM结构的边界。

 

### 六、CSS Styling of Custom Tag Names（对自定义标签进行CSS样式定义）

　　如果想让CSS选择器对自定义元素有效，那么自定义元素必须通过document.createElement(“元素名称”)进行预定义，regardless of XML namespace（实验证明，这里是不用管有XML命名空间的？！）

　　这里是自定义标签样式定义的例子：

 

``` html
<!DOCTYPE html>
<html xmlns:ng="needed for ng: namespace">
<head>
    <title>IE Compatbility</title>
    <!--[if lte IE 8]>
    <script>
        // needed to make ng-include parse properly
        document.createElement('ng-include');

        // needed to enable CSS reference
        document.createElement('ng:view');//注释掉也可以？！
    </script>
    <![endif]-->
    <style>
        ng\:view {
            display: block;
            border: 1px solid red;
            width:100px;
            height:100px;
        }

        ng-include {
            display: block;
            border: 1px solid blue;
            width:100px;
            height:100px;
        }
    </style>
</head>
<body>
    <ng:view></ng:view>
    <ng-include></ng-include>
</body>
</html>
```
 