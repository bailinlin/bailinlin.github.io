---
title: ng1-style-guide1
date: 2015-11-15 20:20:00
tags:  译文,Angular,迁移
---
一个好的编程风格有助于团队的协同开发，所以在做 angular 开发时，我们也有一些约定，本文章主要是针对于使用 angular 和 coffeescript 编程的团队。（这是一个粗糙的翻译版本，原文的链接在文章下，感兴趣的同学可以去看）

>[Angular 编程风格指导原文地址](https://github.com/johnpapa/angular-styleguide)

##1. 单一职责

**原则 1：一个文件只能定义一个组件**

下面的例子定义了app模块和他的依赖，把控制器和服务都定义到一个文件了

    -- 不推荐 --
    SomeController = ()->
    someFactory = ()->
    angular
        .module('app', ['ngRoute'])
        .controller('SomeController' , SomeController)
        .factory('someFactory' , someFactory)

同样的组建，现在我们把它分解到他们自己的文件中

    -- 推荐方式 --
    -- app.module.js --

    angular
        .module('app', ['ngRoute'])


    -- 推荐方式 --
    -- someController.js --
    SomeController = ()->
    angular
        .module('app')
        .controller('SomeController' , SomeController)


    -- 推荐方式 --
    -- someFactory.js --
    someFactory = ()->
    angular
        .module('app')
        .factory('someFactory' , someFactory)

##2. 模块

**原则 2：定义模块的时候不要用变量来定义，用设置的语法来定义**

>因为我们使用的单一职责的原则，每一个组建一个文件，在定义组建的时候你已经用`angular.module`来介绍这个组建是哪个模块的了。


        -- 不推荐 --
        app = angular.module('app', [
            'ngAnimate'
            'ngRoute'
            'app.shared'
            'app.dashboard'
        ])

下面是不用变量的语法

        -- 推荐方式 --
        angular
            .module('app', [
            'ngAnimate'
            'ngRoute'
            'app.shared'
            'app.dashboard'
        ])
**拓展**：当使用模块的时候，不要用变量，要用链式定义的语法

>这样做能够使代码的可读性更高，同时也能避免变量的泄露和碰撞（重名）


        -- 不推荐--
        app = angular.module('app')
        app.controller('SomeController' , SomeController)
        SomeController = ()->

        -- 推荐方式 --
        SomeController = ()->

        angular
          .module('app')
          .controller('SomeController' , SomeController)

**设置 vs 获取**所有的实例只要设置一次

>一个模块只要被创建一次，之后的模块获取只要通过这个切入点来获取就可以了


 - 用 `angular.module('app', [])` 来设置模块
 - 用`angular.module('app')`来获取模块

**命名函数 vs 匿名函数**：使用命名函数来代替向回掉函数传递匿名函数

>这样能够是代码的可读性更高，更加容易debug，减少回调函数的嵌套


        -- 不推荐 --
        angular
          .module('app')
          .controller('Dashboard', ()->)
          .factory('logger', ()-> )

        -- 推荐方式 --
        -- dashboard.js --
        Dashboard = ()->
          # logic goes here -->
          return

        angular
          .module('app')
          .controller('Dashboard', Dashboard)

        -- 推荐方式 --
        -- logger.js --
        logger = ()->
          # logic goes here -->
          return

        angular
          .module('app')
          .factory('logger', logger)
**IIFE（立即调用函数表达式）：**把angular 组建包裹在能够马上调用的函数表达式中

>IIFE 把变量从全局作用域里解放出来，这样做能防止把变量和函数定义在全局作用句中从而造成变量的碰撞（）变量重名造成的莫名其妙的bug）


    (->
      logger = ()->
        # logic goes here -->
        return

      angular
        .module('app')
        .factory('logger', logger);

    )()
**主意**：为了使代码更加简洁，下面的编程先省略 IIFE 语法

##3. 控制器

**controller as view 语法：**用 `controllerAs`语法来代替经典的把controller 绑定到  $scope 作用域的语法

- 控制器是一个构造类，需要通过`newed`来创建一个新的实例，但是`controllerAS`语法更像 javascript 的构造函数
- 这样的用法能够促进我们在视图中绑定对应对象的变量（ 用`customer.name`来取代`name`）等等，这样能够是我们的代码更易读，避免我们没有指定对象的时候的一些参考问题。
- 能够让我们避免在视图中的嵌套控制器中使用`$parent`来调用父级控制器

   		 -- 不推荐 --
   		 <div ng-controller="Customer">
      		{{ name }}
   		 </div>
   		 -- 推荐方式 --
    		<div ng-controller="Customer as customer">
    		  {{ customer.name }}
   		 </div>

**controllerAs Controller Syntax**：用`controllerAs`来代替`传统的将控制器绑定到$scope`的语法

-  `controllerAs`通过 `this`来从控制器的内部把返回的内容绑定到$scope 上
  `controllerAs`在语法上比 `$scope`要友好，使用`controllerAs` 你依旧可以把把数据绑定到视图，依旧可以访问绑定在`$scope`上的方法
- 能够避免可能把方法定义到服务上更好的时候，想要将控制器中的方法绑定到`$scope`上，在服务中要考虑$scope 的用法，在控制器中只有必要的时候才能绑定到`$scope`上，举个例子，当要通过`$emit`,`$broadcast`,`$on`来传递和接收事件的时候，要在服务中定义，再在控制器中调用
**主意**：介于 coffeescript 会自动返回最后一行，我们最好在函数的最后一行加上一个return 声明（即使这个函数没有任何东西返回），大多数上可以没有返回声明，但是在我自己的开发过程中，没有返回声明的时候，会报错，所以建议还是加上这个返回声明

        -- 不推荐 --
        (->
          Customer = ($scope)->
            $scope.name = {}
            $scope.sendMessage = ()->
          angular
            .module('app')
            .controller('Customer', Customer)
        )()
        -- 推荐方法 --
        (->
          Customer = ()->
            @name = {}
            @sendMessage = ()->
            return

          angular
            .module('app')
            .controller('Customer', Customer)
        )()

**controllerAS with vm**当使用`controllerAs`语法时，用一个变量来代替 this ，找一个统一的变量名来代替视图模型例如 `vm`

`this`关键字是联系上下文的，当在控制器中使用函数的时候可能会改变上下文，为了避免这样的情况，最好用一个变量来捕获`this`

        -- 不推荐 --
        (->
          Customer = ()->
            @name = {}
            @sendMessage = ()->
              # here @/this is not the same
              @stuff = "stuff"

            return
          angular
            .module('app')
            .controller('Customer', Customer)
        )()


        -- 推荐方式 --
        (->
          Customer = ()->
            vm = @
            vm.name = {}
            vm.sendMessage = ()->

            return
          angular
            .module('app')
            .controller('Customer', Customer)
        )()

【tip】

        ### OR use the fat arrow in functions => ###
        (->
          Customer = ()->
            @name = {}
            @sendMessage = ()=>
              @stuff

            return
          angular
            .module('app')
            .controller('Customer', Customer)
        )()

【note】：你可以把下面这段代码放到你的代码的最前一行，来避免一些组建的语法检查

        ### jshint validthis: true ###
        vm = @

**把变量放在控制器的最前面**

把数据绑定的变量成员（按照字母顺序）放在控制器的最前面，而不是遍布在整个控制器中。
>把数据绑定的变量成员放在控制器的最前面,使程序更加易读，帮你立即分辨控制器中这个变量成员可以绑定在视图中

        -- 不推荐 --
        function SessionsController() {
            var vm = this;

            vm.gotoSession = function() {
              /* ... */
            };
            vm.refresh = function() {
              /* ... */
            };
            vm.search = function() {
              /* ... */
            };
            vm.sessions = [];
            vm.title = 'Sessions';
        }

        -- 推荐方式 --
        function SessionsController() {
            var vm = this;

            vm.gotoSession = gotoSession;
            vm.refresh = refresh;
            vm.search = search;
            vm.sessions = [];
            vm.title = 'Sessions';

            ////////////

            function gotoSession() {
              /* */
            }

            function refresh() {
              /* */
            }

            function search() {
              /* */
            }
        }
【note】 如果函数是一行的就把函数也放在控制器的前面，这样做对代码的阅读性不会有影响

        -- 不推荐--
        function SessionsController(data) {
            var vm = this;

            vm.gotoSession = gotoSession;
            vm.refresh = function() {
                blabla
            };
            vm.search = search;
            vm.sessions = [];
            vm.title = 'Sessions';
        }



        -- 推荐方法--
        function SessionsController(sessionDataService) {
            var vm = this;

            vm.gotoSession = gotoSession;
            vm.refresh = sessionDataService.refresh; // 1 liner is OK
            vm.search = search;
            vm.sessions = [];
            vm.title = 'Sessions';
        }

### 函数声明，隐藏实现细节

函数声明，隐藏实现的细节。把用于数据绑定的变量成员的声明，放在控制器的前面，实现放在文件的后面。

- 把数据绑定的成员放在前面易于代码的阅读，能让你一眼就分辨出哪个变量用于视图的哪块区域的绑定
- 把函数的实现放在文件的后面，把函数实现这部分比较复杂的部分放在后面。能让你直接看到函数声明这部分重要的信息
- 因为函数的声明被提升了，所以在函数定义前，你是不需要关心这个函数的
- 你不需要担心函数声明的位子移动的问题，你不用担心"因为a函数是依赖于b函数的，a函数移动到b函数前面会让你的代码出错"这样子的问题
- 顺序在函数表达式中很重要

        --不建议使用函数表达式--
        function AvengersController(avengersService, logger) {
            var vm = this;
            vm.avengers = [];
            vm.title = 'Avengers';

            var activate = function() {
                return getAvengers().then(function() {
                    logger.info('Activated Avengers View');
                });
            }

            var getAvengers = function() {
                return avengersService.getAvengers().then(function(data) {
                    vm.avengers = data;
                    return vm.avengers;
                });
            }

            vm.getAvengers = getAvengers;

            activate();
        }

主意上面的例子中把重要的函数声明分散在控制器中，但在下面的例子中重要的函数声明都被提升到了顶部。举个栗子，函数绑定变量如`vm.avengers `和`vm.title`，把函数实现的细节放在后面，这样更加有利于代码的阅读

        --  推荐方法 --
        function AvengersController(avengersService, logger) {
            var vm = this;
            vm.avengers = [];
            vm.getAvengers = getAvengers;
            vm.title = 'Avengers';

            activate();

            function activate() {
                return getAvengers().then(function() {
                    logger.info('Activated Avengers View');
                });
            }

            function getAvengers() {
                return avengersService.getAvengers().then(function(data) {
                    vm.avengers = data;
                    return vm.avengers;
                });
            }
        }

###把控制器的逻辑定义放到服务中

- 逻辑可能会在多个控制器中复用，通过封装在服务中来通过函数暴露给控制器
- 把逻辑封装在服务中，能更容易在单元测试中分离成独立作用域，在控制器中调用函数也更容易模拟
- 从控制器中移除依赖，把实现细节隐藏起来
- 让控制器变得简单，苗条，且专注

        -- 不推荐 --
        function OrderController($http, $q, config, userInfo) {
            var vm = this;
            vm.checkCredit = checkCredit;
            vm.isCreditOk;
            vm.total = 0;

            function checkCredit() {
                var settings = {};
                // Get the credit service base URL from config
                // Set credit service required headers
                // Prepare URL query string or data object with request data
                // Add user-identifying info so service gets the right credit limit for this user.
                // Use JSONP for this browser if it doesn't support CORS
                return $http.get(settings)
                    .then(function(data) {
                     // Unpack JSON data in the response object
                       // to find maxRemainingAmount
                       vm.isCreditOk = vm.total <= maxRemainingAmount
                    })
                    .catch(function(error) {
                       // Interpret error
                       // Cope w/ timeout? retry? try alternate service?
                       // Re-reject with appropriate error for a user to see
                    });
            };
        }
        -- 推荐方法 --
        function OrderController(creditService) {
            var vm = this;
            vm.checkCredit = checkCredit;
            vm.isCreditOk;
            vm.total = 0;

            function checkCredit() {
               return creditService.isOrderTotalOk(vm.total)
                  .then(function(isOk) { vm.isCreditOk = isOk; })
                  .catch(showError);
            };
        }

### 使控制器变得专注
为一个视图定义一个控制器，不要讲一个控制器为多个视图复用，要把复用的逻辑封装到服务中，保证一个控制器只专注于他的视图

给多个视图复用的控制器是很脆的，一个好的 E2E测试覆盖率是用来确定一个应用的稳定性的

### 路由分配

当一个控制器必须和某个视图绑定但也有可能和其他视图或者控制器复用，把控制器的定义和路由条状一起定义

【note】如果一个视图不是通过路由而是通过其他方式加载的，请用`ng-controller="Avengers as vm"`语法

在路由中分配控制器，允许不同的路由去调用不同的控制器和视图，当控制器是在视图中通过`ng-controller` 进行声明，那么这个视图就要和这个控制器一直关联

        -- 不推荐 --

        // route-config.js
        angular
            .module('app')
            .config(config);

        function config($routeProvider) {
            $routeProvider
                .when('/avengers', {
                  templateUrl: 'avengers.html'
                });
        }

        <!-- avengers.html -->
        <div ng-controller="AvengersController as vm">
        </div>



        -- 推荐方式 --

        // route-config.js
        angular
            .module('app')
            .config(config);

        function config($routeProvider) {
            $routeProvider
                .when('/avengers', {
                    templateUrl: 'avengers.html',
                    controller: 'Avengers',
                    controllerAs: 'vm'
                });
        }
        <!-- avengers.html -->
        <div>
        </div>