# AngularJs --E2E Testing
原版地址：http://docs.angularjs.org/guide/dev_guide.e2e-testing

 

　　当一个应用的复杂度、大小在增加时，使得依靠人工去测试新特性的可靠性、抓Bug和回归测试是不切实际的。

　　为了解决这个问题，我们建立了Angular Scenario Runner，模仿用户的操作，帮助我们去验证angular应用的健壮性。

 

### 一、   总括

　　我们可以在javascript中写情景测试（scenario test），描述我们的应用发生的行为，在某个状态下给与某些互动。一个情景包含一个或者多个”it”块（我们可以将这些当作对我们应用的要求），依次由命令（command）和期望（expectation）组成。command告诉Runner在应用中做某些事情（例如转到某个页面或者单击某个按钮），expectation告诉runner去判断一些关于状态的东西（例如某个域的值或者当前的URL）。如果任何expectation失败了，那么runner标记这个”it”为”false”，然后继续下一个”it”。Scenario也可以拥有” beforeEach”和” afterEach”block，这些block会在每一个”it”block之前或者之后运行，不管它是否通过。



　　除了上述元素外，scenario也可以包含helper function，避免在”it”block中有重复的代码。

　　这里是一个简单的scenario例子：  

``` js
describe('Buzz Client', function() {
    it('should filter results', function() {
      input('user').enter('jacksparrow');
      element(':button').click();
      expect(repeater('ul li').count()).toEqual(10);
      Input('filterText').enter('Bees');
      expect(repeater('ul li').count()).toEqual(1);
    });
});
```
　　这个scenario描述了网络客户端的要求，明确地，它应该有过滤user的能力。它开始的时候，输入了一个值到”user”输入框中，单击页面上唯一的按钮，然后它验证是否有10个项目列表。然后，它输入”Bees”到”filterText”的输入框中，然后验证那个列表是不是会减少到只有一个项。

下面的API章节，列出了在Runner中可用的command和expectation。

### 二、   API

　　源代码：https://github.com/angular/angular.js/blob/master/src/ngScenario/dsl.js

　　pause()

　　暂停执行测试，直到我们在console中调用resume()（也可以在Runner界面中点击resume链接）

　　

　　sleep(seconds)

　　暂停测试执行N秒。

 

　　browser().navigateTo(url)

　　在tset frame中加载指定url。

 

　　browser().navigateTo(url,fn)

　　在test frame中加载fn返回的url地址。这里的url参数只是用作测试输出。当目的url是动态的时候可以使用这个API（写测试的时候，地址还是未知的）。

 

　　browser().reload()

　　在test frame中刷新当前加载的页面。

 

　　browser().window().href()

　　返回test frame当前页面的window.location.href。

 

　　browser().window().path()

　　返回test frame当前页面的window.location.pathname。

 

　　browser().window().search()

　　返回test frame当前页面的window.location.search。

　　

　　browser().window().hash()

　　返回test frame当前页面的window.location.hash（不包含#）。

 

　　browser().location().url()

　　返回test frame 当前页面的$location.url()的返回结果（http://docs.angularjs.org/api/ng.$location）

 

　　browser().location().path()

　　返回test frame 当前页面的$location. path ()的返回结果（http://docs.angularjs.org/api/ng.$location）

 

　　browser().location().search()

　　返回test frame 当前页面的$location. search ()的返回结果（http://docs.angularjs.org/api/ng.$location）

 

　　browser().location().hash()

　　返回test frame 当前页面的$location. hash ()的返回结果（http://docs.angularjs.org/api/ng.$location）

 

　　expect(future).{matcher}

　　判断给定的期望(future)值是否满足matcher。所有API的声明都返回一个在它们执行完毕之后获取到的一个指定值的future对象。matcher是使用angular.scenario.matcher定义的，他们使用futures的值去执行expectation。例如：

　　

expect(browser().location().href()).toEqual(‘http://www.google.com’);
 

　　expect(future).not().{matcher}

　　判断给定future的值是否与指定的matcher的预期相反。

 

　　using(selector,label)

　　Scopes the next DSL element selection.（大概是限定选择器的作用域，label估计是用于测试输出）

　　例子：

using('#foo', "'Foo' text field").input('bar')
 

　　binding(name)

　　返回第一个与指定的name匹配的绑定(也许是跟ng-bind相关)。

 

　　input(name).enter(value)

　　输入指定的value到name指定的表单域。

　　

　　input(name).check()

　　选中或者解除选中指定name的checkbox。

 

　　input(name).select(value)

　　选中指定name的radio中值为value的input[type=” radio”]。

 

　　input(name).val()

　　返回指定name的input的当前值。

 

　　repeater(selector,label).count()

　　返回与指定selector（jQuery selector）匹配的repeater的行数。label只用作测试输出。

　　

repeater('#products table', 'Product List').count() //number of rows
 

　　repeater(selector,label).row(index)

　　返回一个数组，绑定指定selector（jQuery selector）匹配的repeater中指定index的行。label仅仅用于测试输出。

repeater('#products table', 'Product List').row(1) //all bindings in row as an array
 

　　repeater(selector,label).column(binding)

　　返回一个数组，值为指定selector（jQuery selector）匹配的repeater中符合指定binding的列。label仅仅用于测试输出。

repeater('#products table', 'Product List').column('product.name') //all values across all rows in an array
　　

　　select(name).option(value)

　　选择指定name的select中指定value的option。

 

　　select(name).option(value1,value2)

　　选择指定name的select中指定value的option（多选）。

 

　　element(selector,label).count()

　　返回与指定selector匹配的元素的个数。label仅仅用作测试输出。

 

　　element(selector,label).click()

　　单击与指定selector匹配的元素。label仅仅用作测试输出。

 

　　element(selector,label).query(fn)

　　执行指定的fn（selectedElements,done），selectedElement就是与指定selector匹配的元素集合；而done是一个function，会在fn执行完毕后执行。label仅仅用作测试输出。

 

　　element(selector,label).{method}()

　　返回在指定selector匹配的元素上执行method的返回值。method可以是以下的jQuery方法：val、text、html、height、innerHeight、outerHeight、width、innerWidth、outerWidth、position、scrollLelft、scrollTop、offset。label仅仅用作测试输出。

 

　　element(selector,label).{method}(value)

　　在指定selector匹配的元素上执行指定method，并以key、value作为参数。method可以是以下的jQuery方法：val、text、html、height、innerHeight、outerHeight、width、innerWidth、outerWidth、position、scrollLelft、scrollTop、offset。label仅仅用作测试输出。

 

　　element(selector,label).{method}(key)

　　返回在指定selector匹配的元素上执行指定method的结果，这些方法可以是以下的jQuery方法：attr，prop，css。label仅仅用作测试输出。

　

　　element(selector,label).{method}(key,value)

在指定的selector匹配的元素上执行method并以key、value作为参数，这些方法可以是以下的jQuery方法：attr，prop，css。label仅仅用作测试输出。

　　

　　javascript是动态类型的语言，带来了强大力量的表达式，但它同时让我们从编译器中几乎得不到任何帮助。因此，我们很强烈地感受到，任何用javascript写的代码都需要进行大量、全面的测试。angular有很多特性，可以让我们更加容易地测试我们的应用。所以我们没有借口不去写测试。(-_-!!)