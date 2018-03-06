---
title: 你需要知道的 Angular 编程指南(下)
date: 2015-11-10 20:32:58
tags:  [译文,Angular]
---
一个好的编程风格有助于团队的协同开发，所以在做 angular 开发时，我们也有一些约定，本文章主要是针对于使用 angular 和 coffeescript 编程的团队。（这是一个粗糙的翻译版本，原文的链接在文章下，感兴趣的同学可以去看）

[Angular 编程引导](https://github.com/Plateful/plateful-mobile/wiki/AngularJS-CoffeeScript-Style-Guide#factories)

### 服务

#### 单例

>服务是通过`new`关键字进行实例化的，用`this`来定义调用公用的方法和变量，和工厂服务很相似，为了统一，可以使用工厂服务

【note】：所有的 angular 服务都是单例模式，这就意味着服务的每次注入都只有一个实例

    // service
    angular
        .module('app')
        .service('logger', logger);

    function logger() {
      this.logError = function(msg) {
        /* */
      };
    }

    // factory
    angular
        .module('app')
        .factory('logger', logger);

    function logger() {
        return {
            logError: function(msg) {
              /* */
            }
       };
    }

### 工厂服务

#### 单一职责

>工厂服务也应该是`单一职责`，当一个服务要实现的功能超过一个目的，就要重新定义一个工厂服务

#### 单例

>工厂服务是单例服务，返回的对象包括了服务的成员对象

【note】所有的 Angular 服务都是单例

#### 可调用的成员靠前

>将服务器的可调用的成员（暴露的接口）提升到服务器前面，（从《学习 javascript 设计模式》中派生出来）

+ 把可调用的变量成员放在服务的最前面，能提供你的代码可读性，能让你一眼就看出，这个服务哪些成员变量是可调用和可被测试的
+ 在文件变长的时候，这样做就显得很有必要了，你不用滚动到文件的下面去查看，这个服务到底暴露了哪些接口
+ 当你的函数超过一行代码的时候，会降低你的代码可读性，阅读时也会造成多余的滚动操作，所以你要把可调用接口的定义，和服务的return ，提升到文件的顶部定义，把实现的细节放在文件下面，这样来增加代码的可读性


              ### 不推荐方式 ###
        (->
          dataService = ()->

            someValue = ''

            save = ()->
              # ... #

            validate = ()->
              # ... #

            return
              save: save,
              someValue: someValue,
              validate: validate

          angular
            .module('app')
            .service('dataService', dataService)
        )()

        ### 推荐方式 ###
        (->
          dataService = ()->

            someValue = ''

            ##########

            return
              save: ()->
               # . #

              validate: ()->
               # . #
          angular
            .module('app')
            .service('dataService', dataService)
        )()

这种方法绑定的的数据是宿主对象的映射，通过模块模式暴露的单一的原始数据是不能独自进行更新的



        ### 不推荐方式 ###
        angular
          .module('app.widgets')

          # order directive that is specific to the order module
          .directive('orderCalendarRange', orderCalendarRange)

          # sales directive that can be used anywhere across the sales app
          .directive('salesCustomerInfo', salesCustomerInfo)

          # spinner directive that can be used anywhere across apps
          .directive('sharedSpinner', sharedSpinner)

          ### implementation details ###



        ### 推荐方式 ###

         ###
         # @desc order directive that is specific to the order module at a company named Acme
         # @file calendarRange.directive.js
         # @example <div acme-order-calendar-range></div>
         ###
        angular
          .module('sales.order')
          .directive('acmeOrderCalendarRange', orderCalendarRange)

         ###
         # @desc spinner directive that can be used anywhere across the sales app at a company named Acme
         # @file customerInfo.directive.js
         # @example <div acme-sales-customer-info></div>
         ###
        angular
          .module('sales.widgets')
          .directive('acmeSalesCustomerInfo', salesCustomerInfo)

         ###
         # @desc spinner directive that can be used anywhere across apps at a company named Acme
         # @file spinner.directive.js
         # @example <div acme-shared-spinner></div>
         ###
        angular
          .module('shared.widgets')
          .directive('acmeSharedSpinner', sharedSpinner)

          ### implementation details ###

### 指令

> 一个指令一个文件，把所有的指令混到一个文件中容易，但是，后面你要把这些指令从这个文件中分离出来就没那么容易了。所以那些需要在 App 和 模块中被共享的指令，一定要分离出来到一个文件中，这样也有利于代码的维护

        ### 不推荐方式###
        angular
          .module('app.widgets')

          # order directive that is specific to the order module
          .directive('orderCalendarRange', orderCalendarRange)

          # sales directive that can be used anywhere across the sales app
          .directive('salesCustomerInfo', salesCustomerInfo)

          # spinner directive that can be used anywhere across apps
          .directive('sharedSpinner', sharedSpinner)

          ### implementation details ###


        ### 推荐方式 ###

         ###
         # @desc order directive that is specific to the order module at a company named Acme
         # @file calendarRange.directive.js
         # @example <div acme-order-calendar-range></div>
         ###
        angular
          .module('sales.order')
          .directive('acmeOrderCalendarRange', orderCalendarRange)

         ###
         # @desc spinner directive that can be used anywhere across the sales app at a company named Acme
         # @file customerInfo.directive.js
         # @example <div acme-sales-customer-info></div>
         ###
        angular
          .module('sales.widgets')
          .directive('acmeSalesCustomerInfo', salesCustomerInfo)

         ###
         # @desc spinner directive that can be used anywhere across apps at a company named Acme
         # @file spinner.directive.js
         # @example <div acme-shared-spinner></div>
         ###
        angular
          .module('shared.widgets')
          .directive('acmeSharedSpinner', sharedSpinner)

          ### implementation details ###


[note]: 指令有很多命名选项，特别要注意的是，指令在一个大的作用域里，能被使用的范围就会变窄。选择一个能让这个指令看起来一目了然的名字。下面有一些例子，但还是建议去看关于命名的章节

#### 限制DOM的操作

>限制DOM的操作，用指令来直接操作DOM，如果有可以替代的方式，如使用css来设置样式，使用 animation 服务，angular 的模版，ngShow 或者 ngHide ，那么就用这些来代替指令。举个例子，如果指令就是定义一个简单的显示和隐藏，那么久用 nghide 和 ngShow 来代替，但是如果指令除了显示隐藏还需要处理更加复杂的事情，那就把显示隐藏和其他需要实现的操作，一起封装到这个指令里，这样能减少 angular 的监听，来提高应用的性能。

+ 对 DOM 的操作不太容易进行测试和调试，我们有更好的办法前提是对DOM的操作比较简单的话（css，animations，templating）

#### 限制元素和属性


> 限制元素和属性：当创建一个指令，这个指令的如果表现的像一个元素，那么 restrict 设置为 E ，也可以选择设置成 A，，如果这个指令能有他自己的控制器， restrict 设置为 E 是最理想的，不过通常的话，一些引导是将 restrict 设置为 EA，但当指令被封装在独立作用域时，倾向于元素指令表现
，当增强于现有的 DOM 元素，倾向于属性表现

+ 这样做有意义
+ 如果指令倾向于表现得像元素或者属性，这就允许我们定义的指令使用 class 属性

        <!--不推荐方式-->
        <div class="my-calendar-range"></div>
        ### avoid ###
        (->
          myCalendarRange = ()->
              link = (scope, element, attrs)->
                # ... #

              directive =
                link: link,
                templateUrl: '/template/is/located/here.html',
                restrict: 'C'

              return directive

          angular
              .module('app.widgets')
              .directive('myCalendarRange', myCalendarRange)
        )()
        <!-- recommended -->
        <my-calendar-range></my-calendar-range>
        <div my-calendar-range></div>

        ### 推荐方式 ###
        (->

          myCalendarRange = ()->

              link = (scope, element, attrs)->
                # ... #

              directive =
                  link: link,
                  templateUrl: '/template/is/located/here.html',
                  restrict: 'EA'

              return directive

          angular
              .module('app.widgets')
              .directive('myCalendarRange', myCalendarRange)
        )()

