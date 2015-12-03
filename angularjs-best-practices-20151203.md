![AngularJS Helsinki Logo on Black](//angularjs-helsinki.s3-eu-west-1.amazonaws.com/angularjs_hel_logo_on_black.svg)

## [@angularjs_hel](//twitter.com/angularjs_hel) [meetup](http://www.meetup.com/AngularJS-Helsinki/)
#### [Frantic](https://www.frantic.com/) office, 3rd December, 2015.

---

# AngularJS Best Practices

## AngularJS 1.x + JavaScript

  *This material is heavily based on workings of [@john_papa](//twitter.com/john_papa)*

  The original material can be read from GitHub: https://github.com/johnpapa/angular-styleguide

---

# Single Responsibility

___

# Rule of 1

- Define 1 component per file.

*Why?*: Easier to read, easier to understand and test.

___

# Avoid

The following example defines the `app` module and its dependencies, defines a controller, and defines a factory all in the same file.

```javascript
angular
    .module('app', ['ngRoute'])
    .controller('SomeController', SomeController)
    .factory('someFactory', someFactory);

function SomeController() { }

function someFactory() { }
```

___

# Recommended

The same components are now separated into their own files.

```javascript
// app.module.js
angular
    .module('app', ['ngRoute']);
```

```javascript
// some.controller.js
angular
    .module('app')
    .controller('SomeController', SomeController);

function SomeController() { }
```

```javascript
// someFactory.js
angular
    .module('app')
    .factory('someFactory', someFactory);

function someFactory() { }
```

---

# IIFE

___

# JavaScript Closures

- Wrap Angular components in an Immediately Invoked Function Expression (IIFE)

*Why?*: An IIFE removes variables from the global scope. This helps prevent variables and function declarations from living longer than expected in the global scope.

*Why?*: When your code is minified and bundled into a single file for deployment to a production server, you could have collisions of variables and many global variables. An IIFE protects you against both of these by providing variable scope for each file.

___

# Avoid

```javascript
// logger.js
angular
    .module('app')
    .factory('logger', logger);

// logger function is added as a global variable
function logger() { }
```

```javascript
// storage.js
angular
    .module('app')
    .factory('storage', storage);

// storage function is added as a global variable
function storage() { }
```

___

# Recommended

```javascript
/**
 * no globals are left behind
 */

// logger.js
(function() {
    'use strict';

    angular
        .module('app')
        .factory('logger', logger);

    function logger() { }
})();
```

```javascript
// storage.js
(function() {
    'use strict';

    angular
        .module('app')
        .factory('storage', storage);

    function storage() { }
})();
```

---

# Modules

___


# Avoid naming collitions

- Use unique naming conventions with separators for sub-modules

*Why?*: Unique names help avoid module name collisions. Separators help define modules and their submodule hierarchy. For example `app` may be your root module while `app.dashboard` and `app.users` may be modules that are used as dependencies of `app`.

___

# Definitions (aka Setters)

#### Declare modules without a variable using the setter syntax

*Why?*: With 1 component per file, there is rarely a need to introduce a variable for the module.

___

# Avoid

```javascript
var app = angular.module('app', [
    'ngAnimate',
    'ngRoute',
    'app.shared',
    'app.dashboard'
]);
```

# Recommended

Instead use the simple setter syntax.

```javascript
angular
    .module('app', [
        'ngAnimate',
        'ngRoute',
        'app.shared',
        'app.dashboard'
    ]);
```

___

# Getters

- When using a module, avoid using a variable and instead use chaining with the getter syntax

*Why?*: This produces more readable code and avoids variable collisions or leaks.

___

# Avoid

```javascript
var app = angular.module('app');
app.controller('SomeController', SomeController);

function SomeController() { }
```

# Recommended

```javascript
angular
    .module('app')
    .controller('SomeController', SomeController);

function SomeController() { }
```

___

# Setting vs Getting

- Only set once and get for all other instances.

*Why?*: A module should only be created once, then retrieved from that point and after.

## Recommended

```javascript
// to set a module
angular.module('app', []);

// to get a module
angular.module('app');
```

___

# Named vs Anonymous Functions

- Use named functions instead of passing an anonymous function in as a callback.

*Why?*: This produces more readable code, is much easier to debug, and reduces the amount of nested callback code.

___

# Avoid

```javascript
angular
    .module('app')
    .controller('DashboardController', function() {
        // My cool code here
    })
    .factory('logger', function() {
        // Another cool code
    });
```

___

# Recommended

```javascript
/* recommended */

// dashboard.js
angular
    .module('app')
    .controller('DashboardController', DashboardController);

function DashboardController() {
    // My cool code here
}
```

```javascript
// logger.js
angular
    .module('app')
    .factory('logger', logger);

function logger() {
    // Another cool code
}
```

---

# Controllers

___

# controllerAs View Syntax

- Use the [`controllerAs`](http://www.johnpapa.net/do-you-like-your-angular-controllers-with-or-without-sugar/) syntax over the `classic controller with $scope` syntax.

___

*Why?*: Controllers are constructed, "newed" up, and provide a single new instance, and the `controllerAs` syntax is closer to that of a JavaScript constructor than the `classic $scope syntax`.

*Why?*: It promotes the use of binding to a "dotted" object in the View (e.g. `customer.name` instead of `name`), which is more contextual, easier to read, and avoids any reference issues that may occur without "dotting".

*Why?*: Helps avoid using `$parent` calls in Views with nested controllers.

___

# Avoid

```html
<div ng-controller="CustomerController">
    {{ name }}
</div>
```

# Recommended

```html
<div ng-controller="CustomerController as customer">
    {{ customer.name }}
</div>
```

___

# controllerAs Controller Syntax

- Use the `controllerAs` syntax over the `classic controller with $scope` syntax.

- The `controllerAs` syntax uses `this` inside controllers which gets bound to `$scope`

___

*Why?*: `controllerAs` is syntactic sugar over `$scope`. You can still bind to the View and still access `$scope` methods.

*Why?*: Helps avoid the temptation of using `$scope` methods inside a controller when it may otherwise be better to avoid them or move the method to a factory, and reference them from the controller.

Consider using [`$scope`](https://docs.angularjs.org/guide/scope) in a controller only when needed. For example when publishing and subscribing events using [`$emit`](https://docs.angularjs.org/api/ng/type/$rootScope.Scope#$emit), [`$broadcast`](https://docs.angularjs.org/api/ng/type/$rootScope.Scope#$broadcast), or [`$on`](https://docs.angularjs.org/api/ng/type/$rootScope.Scope#$on).

___

# Avoid

```javascript
function CustomerController($scope) {
    $scope.name = {};
    $scope.sendMessage = function() { };
}
```

# Recommended

```javascript
function CustomerController() {
    this.name = {};
    this.sendMessage = function() { };
}
```

___

# controllerAs with vm

- Use a capture variable for `this` when using the `controllerAs` syntax. Choose a consistent variable name such as `vm`, which stands for ViewModel.

*Why?*: The `this` keyword is contextual and when used within a function inside a controller may change its context. Capturing the context of `this` avoids encountering this problem.

___

# Avoid

```javascript
function CustomerController() {
    this.name = {};
    this.sendMessage = function() { };
}
```

# Recommended

```javascript
function CustomerController() {
    /* jshint validthis: true */
    var vm = this;
    vm.name = {};
    vm.sendMessage = function() { };
}
```

___

# Bindable Members Up Top

- Place bindable members at the top of the controller, alphabetized, and not spread through the controller code.

*Why?*: Placing bindable members at the top makes it easy to read and helps you instantly identify which members of the controller can be bound and used in the View.

*WhY?*: Setting anonymous functions in-line can be easy, but when those functions are more than 1 line of code they can reduce the readability. Defining the functions below the bindable members (the functions will be hoisted) moves the implementation details down, keeps the bindable members up top, and makes it easier to read.

___

# Avoid

```javascript
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
```

___

# Recommended

```javascript
function SessionsController() {
    var vm = this;

    vm.gotoSession = gotoSession;
    vm.refresh = refresh;
    vm.search = search;
    vm.sessions = [];
    vm.title = 'Sessions';

    function gotoSession() {
      /* */
    }

    // ...
}
```

___

# Function Declarations to Hide Implementation Details

- Use function declarations to hide implementation details. Keep your bindable members up top.

___

*Why?*: Placing bindable members at the top makes it easy to read and helps you instantly identify which members of the controller can be bound and used in the View.

*Why?*: Placing the implementation details of a function later in the file moves that complexity out of view so you can see the important stuff up top.

*Why?*: Function declaration are hoisted so there are no concerns over using a function before it is defined (as there would be with function expressions).

*Why?*: You never have to worry with function declarations that moving `var a` before `var b` will break your code because `a` depends on `b`.

*Why?*: Order is critical with function expressions

___

# Avoid

```javascript
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
```

___

# Recommended

```javascript
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
```

___

# Defer Controller Logic to Services

- Defer logic in a controller by delegating to services and factories.

___

*Why?*: Logic may be reused by multiple controllers when placed within a service and exposed via a function.

*Why?*: Logic in a service can more easily be isolated in a unit test, while the calling logic in the controller can be easily mocked.

*Why?*: Removes dependencies and hides implementation details from the controller.

*Why?*: Keeps the controller slim, trim, and focused.

___

# Avoid

```javascript
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
```

___

# Recommended

```javascript
/* recommended */
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
```

___

# Keep Controllers Focused

- Define a controller for a view, and try not to reuse the controller for other views. Instead, move reusable logic to factories and keep the controller simple and focused on its view.

*Why?*: Reusing controllers with several views is brittle and good end-to-end (e2e) test coverage is required to ensure stability across large applications.

___

# Assigning Controllers

- When a controller must be paired with a view and either component may be re-used by other controllers or views, define controllers along with their routes.

Note: If a View is loaded via another means besides a route, then use the `ng-controller="Avengers as vm"` syntax.

*Why?*: Pairing the controller in the route allows different routes to invoke different pairs of controllers and views. When controllers are assigned in the view using [`ng-controller`](https://docs.angularjs.org/api/ng/directive/ngController), that view is always associated with the same controller.

___

# Avoid

```javascript
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
```

```html
<!-- avengers.html -->
<div ng-controller="AvengersController as vm">
</div>
```

___

# Recommended

```javascript
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
```

```html
<!-- avengers.html -->
<div>
</div>
```

---

# Services

___

# Singletons

- Services are instantiated with the `new` keyword, use `this` for public methods and variables.

- Since these are so similar to factories, use a factory instead for consistency.

___

# Service

```javascript
angular
    .module('app')
    .service('logger', logger);

function logger() {
    this.logError = function(msg) {
        /* */
    };
}
```

___

# Factory

```javascript
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
```

---

# Factories

___

- Factories should have a [single responsibility](http://en.wikipedia.org/wiki/Single_responsibility_principle), that is encapsulated by its context. Once a factory begins to exceed that singular purpose, a new factory should be created.

- Factories are singletons and return an object that contains the members of the service.

- Expose the callable members of the service (its interface) at the top, using a technique derived from the [Revealing Module Pattern](http://addyosmani.com/resources/essentialjsdesignpatterns/book/#revealingmodulepatternjavascript).

- Use function declarations to hide implementation details. Keep your accessible members of the factory up top. Point those to function declarations that appears later in the file.

___

# Avoid

```javascript
function dataService() {
  var someValue = '';
  function save() {
    /* */
  };
  function validate() {
    /* */
  };

  return {
      save: save,
      someValue: someValue,
      validate: validate
  };
}
```

___

# Recommended

```javascript
function dataService() {
    var someValue = '';
    var service = {
        save: save,
        someValue: someValue,
        validate: validate
    };
    return service;

    function save() {
        /* */
    };

    function validate() {
        /* */
    };
}
```

---

# Data Services

___

# Separate Data Calls

- Refactor logic for making data operations and interacting with data to a factory. Make data services responsible for XHR calls, local storage, stashing in memory, or any other data operations.

___

*Why?*: The controller's responsibility is for the presentation and gathering of information for the view. It should not care how it gets the data.

*Why?*: This makes it easier to test (mock or real) the data calls when testing a controller that uses a data service.

*Why?*: Data service implementation may have very specific code to handle the data repository. This may include headers, how to talk to the data, or other services such as `$http`.

___

# Recommended

```javascript
// dataservice factory
angular
    .module('app.core')
    .factory('dataservice', dataservice);

dataservice.$inject = ['$http', 'logger'];

function dataservice($http, logger) {
    return {
        getAvengers: getAvengers
    };

    function getAvengers() {
        return $http.get('/api/maa')
            .then(getAvengersComplete)
            .catch(getAvengersFailed);

        function getAvengersComplete(response) {
            return response.data.results;
        }

        function getAvengersFailed(error) {
            logger.error('XHR Failed for getAvengers.' + error.data);
        }
    }
}
```

___

# Recommended

```javascript
// controller calling the dataservice factory
angular
    .module('app.avengers')
    .controller('AvengersController', AvengersController);

AvengersController.$inject = ['dataservice', 'logger'];

function AvengersController(dataservice, logger) {
    var vm = this;
    vm.avengers = [];

    activate();

    function activate() {
        return getAvengers().then(function() {
            logger.info('Activated Avengers View');
        });
    }

    function getAvengers() {
        return dataservice.getAvengers()
            .then(function(data) {
                vm.avengers = data;
                return vm.avengers;
            });
    }
}
```

___

# Return a Promise from Data Calls

- When calling a data service that returns a promise such as `$http`, return a promise in your calling function too.

*Why?*: You can chain the promises together and take further action after the data call completes and resolves or rejects the promise.

___

# Recommended

```javascript
activate();

function activate() {
    // Step 1: Ask getAvengers() for the avenger data and wait for the promise
    return getAvengers().then(function() {
        // Step 4: Perform an action on resolve of final promise
        logger.info('Activated Avengers View');
    });
}

function getAvengers() {
    // Step 2: Ask the data service for the data and wait the promise
    return dataservice.getAvengers()
        .then(function(data) {
            // Step 3: set the data and resolve the promise
            vm.avengers = data;
            return vm.avengers;
        });
}
```

---

# Directives

___

# Limit 1 Per File

- Create one directive per file. Name the file for the directive.

*Why?*: It is easy to mash all the directives in one file, but difficult to then break those out so some are shared across apps, some across modules, some just for one module.

*Why?*: One directive per file is easy to maintain.

___

# Avoid

```javascript
/* directives.js */

angular
    .module('app.widgets')

    /* order directive that is specific to the order module */
    .directive('orderCalendarRange', orderCalendarRange)

    /* sales directive that can be used anywhere across the sales app */
    .directive('salesCustomerInfo', salesCustomerInfo)

    /* spinner directive that can be used anywhere across apps */
    .directive('sharedSpinner', sharedSpinner);

function orderCalendarRange() {
    /* implementation details */
}

function salesCustomerInfo() {
    /* implementation details */
}

function sharedSpinner() {
    /* implementation details */
}
```

___

# Recommended

```javascript
/* calendar-range.directive.js */

/**
 * @desc order directive that is specific to the order module at a company named Acme
 * @example <div acme-order-calendar-range></div>
 */
angular
    .module('sales.order')
    .directive('acmeOrderCalendarRange', orderCalendarRange);

function orderCalendarRange() {
    /* implementation details */
}
```

___

# Recommended

```javascript
/* customer-info.directive.js */

/**
 * @desc sales directive that can be used anywhere across the sales app at a company named Acme
 * @example <div acme-sales-customer-info></div>
 */
angular
    .module('sales.widgets')
    .directive('acmeSalesCustomerInfo', salesCustomerInfo);

function salesCustomerInfo() {
    /* implementation details */
}
```

___

# Recommended

```javascript
/* spinner.directive.js */

/**
 * @desc spinner directive that can be used anywhere across apps at a company named Acme
 * @example <div acme-shared-spinner></div>
 */
angular
    .module('shared.widgets')
    .directive('acmeSharedSpinner', sharedSpinner);

function sharedSpinner() {
    /* implementation details */
}
```

___

# Manipulate DOM in a Directive

- When manipulating the DOM directly, use a directive. If alternative ways can be used such as using CSS to set styles or the [animation services](https://docs.angularjs.org/api/ngAnimate), Angular templating, [`ngShow`](https://docs.angularjs.org/api/ng/directive/ngShow) or [`ngHide`](https://docs.angularjs.org/api/ng/directive/ngHide), then use those instead. For example, if the directive simply hides and shows, use ngHide/ngShow.

*Why?*: DOM manipulation can be difficult to test, debug, and there are often better ways (e.g. CSS, animations, templates)

___

# Provide a Unique Directive Prefix

- Provide a short, unique and descriptive directive prefix such as `acmeSalesCustomerInfo` which would be declared in HTML as `acme-sales-customer-info`.

*Why?*: The unique short prefix identifies the directive's context and origin. For example a prefix of `cc-` may indicate that the directive is part of a CodeCamper app while `acme-` may indicate a directive for the Acme company.

Note: Avoid reserved name spaces such as `ng-` and `ion-`.

___

## Restrict to Elements and Attributes

- When creating a directive that makes sense as a stand-alone element, allow restrict `E` (custom element) and optionally restrict `A` (custom attribute). Generally, if it could be its own control, `E` is appropriate. General guideline is allow `EA` but lean towards implementing as an element when it's stand-alone and as an attribute when it enhances its existing DOM element.

*Why?*: While we can allow the directive to be used as a class, if the directive is truly acting as an element it makes more sense as an element or at least as an attribute.

Note: EA is the default for Angular 1.3+ .

___

# Avoid

```html
<div class="my-calendar-range"></div>
```

```javascript
angular
    .module('app.widgets')
    .directive('myCalendarRange', myCalendarRange);

function myCalendarRange() {
    var directive = {
        link: link,
        templateUrl: '/template/is/located/here.html',
        restrict: 'C'
    };
    return directive;

    function link(scope, element, attrs) {
      /* */
    }
}
```

___

# Recommended

```html
<my-calendar-range></my-calendar-range>
<div my-calendar-range></div>
```

```javascript
angular
    .module('app.widgets')
    .directive('myCalendarRange', myCalendarRange);

function myCalendarRange() {
    var directive = {
        link: link,
        templateUrl: '/template/is/located/here.html',
        restrict: 'EA'
    };
    return directive;

    function link(scope, element, attrs) {
      /* */
    }
}
```

---

# Resolving Promises for a Controller

___

# Controller Activation Promises

- Resolve start-up logic for a controller in an `activate` function.

___

*Why?*: Placing start-up logic in a consistent place in the controller makes it easier to locate, more consistent to test, and helps avoid spreading out the activation logic across the controller.

*Why?*: The controller `activate` makes it convenient to re-use the logic for a refresh for the controller/View, keeps the logic together, gets the user to the View faster, makes animations easy on the `ng-view` or `ui-view`, and feels snappier to the user.

___

# Avoid

```javascript
function AvengersController(dataservice) {
    var vm = this;
    vm.avengers = [];
    vm.title = 'Avengers';

    dataservice.getAvengers().then(function(data) {
        vm.avengers = data;
        return vm.avengers;
    });
}
```

___

# Recommended

```javascript
function AvengersController(dataservice) {
    var vm = this;
    vm.avengers = [];
    vm.title = 'Avengers';

    activate();

    function activate() {
        return dataservice.getAvengers().then(function(data) {
            vm.avengers = data;
            return vm.avengers;
        });
    }
}
```

___

# Route Resolve Promises

- When a controller depends on a promise to be resolved before the controller is activated, resolve those dependencies in the `$routeProvider` before the controller logic is executed.

- If you need to conditionally cancel a route before the controller is activated, use a route resolver.

- Use a route resolve when you want to decide to cancel the route before ever transitioning to the View.

___

*Why?*: A controller may require data before it loads. That data may come from a promise via a custom factory or [$http](https://docs.angularjs.org/api/ng/service/$http). Using a [route resolve](https://docs.angularjs.org/api/ngRoute/provider/$routeProvider) allows the promise to resolve before the controller logic executes, so it might take action based on that data from the promise.

*Why?*: The code executes after the route and in the controller’s activate function. The View starts to load right away. Data binding kicks in when the activate promise resolves. A “busy” animation can be shown during the view transition (via `ng-view` or `ui-view`)

___

# Avoid

```javascript
angular
    .module('app')
    .controller('AvengersController', AvengersController);

function AvengersController(movieService) {
    var vm = this;
    // unresolved
    vm.movies;
    // resolved asynchronously
    movieService.getMovies().then(function(response) {
        vm.movies = response.movies;
    });
}
```

___

# Better

```javascript
// route-config.js
angular
    .module('app')
    .config(config);

function config($routeProvider) {
    $routeProvider
        .when('/avengers', {
            templateUrl: 'avengers.html',
            controller: 'AvengersController',
            controllerAs: 'vm',
            resolve: {
                moviesPrepService: function(movieService) {
                    return movieService.getMovies();
                }
            }
        });
}
```
___

# Better

```javascript
// avengers.js
angular
    .module('app')
    .controller('AvengersController', AvengersController);

AvengersController.$inject = ['moviesPrepService'];
function AvengersController(moviesPrepService) {
    var vm = this;
    vm.movies = moviesPrepService.movies;
}
```

___

# Recommended

```javascript
// route-config.js
angular
    .module('app')
    .config(config);

function config($routeProvider) {
    $routeProvider
        .when('/avengers', {
            templateUrl: 'avengers.html',
            controller: 'AvengersController',
            controllerAs: 'vm',
            resolve: {
                moviesPrepService: moviesPrepService
            }
        });
}

function moviesPrepService(movieService) {
    return movieService.getMovies();
}
```

___

# Recommended

```javascript
// avengers.js
angular
    .module('app')
    .controller('AvengersController', AvengersController);

AvengersController.$inject = ['moviesPrepService'];
function AvengersController(moviesPrepService) {
      var vm = this;
      vm.movies = moviesPrepService.movies;
}
```

---

# Minification and Annotation

___

# ng-annotate

- Use [ng-annotate](//github.com/olov/ng-annotate) for [Gulp](http://gulpjs.com) or [Grunt](http://gruntjs.com) and comment functions that need automated dependency injection using `/* @ngInject */`

*Why?*: This safeguards your code from any dependencies that may not be using minification-safe practices.

*Why?*: [`ng-min`](https://github.com/btford/ngmin) is deprecated

___

Before `ng-annotate`

```javascript
angular
    .module('app')
    .controller('AvengersController', AvengersController);

/* @ngInject */
function AvengersController(storage, avengerService) {
    var vm = this;
    vm.heroSearch = '';
    vm.storeHero = storeHero;

    function storeHero() {
        var hero = avengerService.find(vm.heroSearch);
        storage.save(hero.name, hero);
    }
}
```

___

After `ng-annotate`

```javascript
angular
    .module('app')
    .controller('AvengersController', AvengersController);

/* @ngInject */
function AvengersController(storage, avengerService) {
    var vm = this;
    vm.heroSearch = '';
    vm.storeHero = storeHero;

    function storeHero() {
        var hero = avengerService.find(vm.heroSearch);
        storage.save(hero.name, hero);
    }
}

AvengersController.$inject = ['storage', 'avengerService'];
```

Note: If `ng-annotate` detects injection has already been made (e.g. `@ngInject` was detected), it will not duplicate the `$inject` code.

___

When using a route resolver you can prefix the resolver's function with `/* @ngInject */` and it will produce properly annotated code, keeping any injected dependencies minification safe.

```javascript
// Using @ngInject annotations
function config($routeProvider) {
    $routeProvider
        .when('/avengers', {
            templateUrl: 'avengers.html',
            controller: 'AvengersController',
            controllerAs: 'vm',
            resolve: { /* @ngInject */
                moviesPrepService: function(movieService) {
                    return movieService.getMovies();
                }
            }
        });
}
```

___

# Use Gulp or Grunt for ng-annotate

- Use [gulp-ng-annotate](https://www.npmjs.org/package/gulp-ng-annotate) or [grunt-ng-annotate](https://www.npmjs.org/package/grunt-ng-annotate) in an automated build task. Inject `/* @ngInject */` prior to any function that has dependencies.

*Why?*: ng-annotate will catch most dependencies, but it sometimes requires hints using the `/* @ngInject */` syntax.

___

### gulp + ng-annotate

```javascript
gulp.task('js', ['jshint'], function() {
    var source = pkg.paths.js;

    return gulp.src(source)
        .pipe(sourcemaps.init())
        .pipe(concat('all.min.js', {newLine: ';'}))
        // Annotate before uglify so the code get's min'd properly.
        .pipe(ngAnnotate({
            // true helps add where @ngInject is not used. It infers.
            // Doesn't work with resolve, so we must be explicit there
            add: true
        }))
        .pipe(bytediff.start())
        .pipe(uglify({mangle: true}))
        .pipe(bytediff.stop())
        .pipe(sourcemaps.write('./'))
        .pipe(gulp.dest(pkg.paths.dev));
});
```

___

### grunt + ng-annotate

```javascript
module.exports = function(grunt) {
    grunt.initConfig({
        // ...
        ngAnnotate: {
            compile: {
                files: {
                    "dist/main.js": [ 'src/*.js', 'src/**/*,js' ]
                }
            }
        },
        // ...
    });
    grunt.loadNpmTasks('grunt-ng-annotate');
};
```

___

# Grunt++

Lets take some more Grunt plugins:

```javascript
{
  "name": "foo",
  "version": "0.0.1",
  "description": "FooApp",
  "devDependencies": {
    "grunt": "~0.4",
    "grunt-contrib-copy": "~0.8",
    "grunt-contrib-uglify": "~0.10",
    "grunt-html2js": "~0.3",
    "grunt-ng-annotate": "~1.0"
  }
}
```

```javascript
// Gruntfile.js
module.exports = function(grunt) {
    require('load-grunt-config')(grunt);
};
```

___

```javascript
// grunt/html2js.js
module.exports = {
    app: {
        options: {
            base: "src/html"
        },
        src: [
            "src/**/*.html"
        ],
        dest: "dist/templates-app.js"
    }
}
```

___

```javascript
// grunt/ngAnnotate.js
module.exports = {
    dist: {
        files: {
            "dist/main.js": [
                'src/**/*,js'
            ]
        }
    }
}
```

___

```javascript
// grunt/copy.js
module.exports = {
    dist: {
        files: [
            {
                cwd: 'src'.
                src: [ 'html/*.html' ],
                dest: '../dist',
                expand: true,
                flatten: false
            },
            {
                src: [
                    "bower_modules/angular/angular.min.js",
                    "bower_modules/angular-resource/angular-resource.min.js",
                    "bower_modules/angular-sanitize/angular-sanitize.min.js"
                ],
                dest: 'dist',
                expand: true,
                flatten: true
            }
        ]
    }
}
```

___

```javascript
// grunt/uglify.js
module.exports = {
    dist: {
        files: {
            "dist/main.min.js": [
                "bower_modules/angular/angular.js",
                "bower_modules/angular-resource/angular-resource.js",
                "bower_modules/angular-sanitize/angular-sanitize.js",
                "dist/main.js"
            ]
        }
    }
}
```

---

# Naming

___

# Feature File Names

- Use consistent names for all components following a pattern that describes the component's feature then (optionally) its type.

- Small projects and large projects matter on decision whether to use which type of system.

___

```javascript
// controllers
avengers.controller.js
avengers.controller.spec.js

// services/factories
logger.service.js
logger.service.spec.js

// module definition
avengers.module.js

// routes
avengers.routes.js
avengers.routes.spec.js

// configuration
avengers.config.js

// directives
avenger-profile.directive.js
avenger-profile.directive.spec.js
```

___

# Test Names

- Use `spec` to identify test files:

```javascript
avengers.controller.spec.js
logger.service.spec.js
avenger-profile.directive.spec.js
```

___

# Controller Names

- Use consistent names for all controllers named after their feature. Use UpperCamelCase for controllers, as they are constructors.

*Why?*: Provides a consistent way to quickly identify and reference controllers.

*Why?*: UpperCamelCase is conventional for identifying object that can be instantiated using a constructor.

___

# Recommended

```javascript
// avengers.controller.js
angular
    .module
    .controller('HeroAvengersController', HeroAvengersController);

function HeroAvengersController() { }
```

___

# Factory and Service Names

- Use consistent names for all factories and services named after their feature. Use camelCasing for services and factories.

- Avoid prefixing factories and services with `$`.

- Only suffix service and factories with `Service` when it is not clear what they are (i.e. when they are nouns).

___

*Why?*: Provides a consistent way to quickly identify and reference factories.

*Why?*: Avoids name collisions with built-in factories and services that use the `$` prefix.

*Why?*: Clear service names such as `logger` do not require a suffix.

*Why?*: Service names such as `avengers` are nouns and require a suffix and should be named `avengersService`.

___

# Recommended

```javascript
// logger.service.js
angular
    .module
    .factory('logger', logger);

function logger() { }
```

```javascript
// credit.service.js
angular
    .module
    .factory('creditService', creditService);

function creditService() { }
```

```javascript
// customer.service.js
angular
    .module
    .service('customerService', customerService);

function customerService() { }
```

___

# Directive Component Names

- Use consistent names for all directives using camelCase.

- Use a short prefix to describe the area that the directives belong (some example are company prefix or project prefix).

*Why?*: Provides a consistent way to quickly identify and reference components.

___

# Recommended

```javascript
// avenger-profile.directive.js
angular
    .module
    .directive('xxAvengerProfile', xxAvengerProfile);

function xxAvengerProfile() { }
```

```html
<!-- Usage is: -->
<xx-avenger-profile></xx-avenger-profile>
```

___

# Modules

- When there are multiple modules, the main module file is named `app.module.js` while other dependent modules are named after what they represent.

- For example, an admin module is named `admin.module.js`. The respective registered module names would be `app` and `admin`.

*Why?*: Provides consistency for multiple module apps, and for expanding to large applications.

*Why?*: Provides easy way to use task automation to load all module definitions first, then all other angular files (for bundling).

___

# Configuration

- Separate configuration for a module into its own file named after the module.

- For example:
  * Configuration file for the main `app` module is named `app.config.js` (or simply `config.js`).
  * Configuration for a module named `admin.module.js` is named `admin.config.js`.

*Why?*: Separates configuration from module definition, components, and active code.

*Why?*: Provides an identifiable place to set configuration for a module.

___

### Routes

- Separate route configuration into its own file.

- For example:
  * `app.route.js` for the main module
  * `admin.route.js` for the `admin` module
  
- Even in smaller apps I prefer this separation from the rest of the configuration.

---

# Application Structure

___

# All project

- Separate 3rd party packages from your code - use `node_modules` and `bower_modules` to install packages into.

- Separate distribution directory; you can get your concatinated, uglified and bundled versions to a `dist` directory. Another good name is `target` or `bin`.

- Using multiple package managers (`bower` and `npm`) can ease out separating files to be vendored from the build process.

___

# Small projects

- Keep same type of files in separate directories, which makes it easier to make `grunt` or `gulp` tasks.

- Use as few files and directories as possible - you could stick to a single package manager for simplicity

___

```
// Small project
├── Gruntfile.js
├── bower.json
├── bower_modules
│   └── ...
├── grunt
│   └── ...
├── node_modules
│   └── ...
├── package.json
├── dist
│   ├── main.js
│   └── main.min.js
├── src
    ├── html
    │   ├── bar.directive.tpl.html
    │   └── foo.controller.tpl.html
    └── js
        ├── app.config.js
        ├── app.js
        ├── bar.directive.js
        ├── baz.service.js
        ├── foo.controller.js
        └── templates-app.js
```

___

# Large projects

- Design a directory structure in mind that it's easily adaptable to future changes.

- If can separate commonly used self-made modules, locate them to a separate GitHub repository and source them with `npm` or `bower` (this also keeps you going on to Open Sourcing).

___
```
// Larger project
├── Gruntfile.js
├── bower.json
├── bower_modules
│   └── ...
├── grunt
│   └── ...
├── node_modules
│   └── ...
├── package.json
└── src
    ├── assets
    │   └── ...
    ├── app
    │   ├── about
    │   │   └── ...
    │   ├── app.js
    │   ├── app.config.js
    │   ├── contact
    │   │   └── ...
    │   ├── home
    │   └── services
    ├── fonts
    │   └── ...
    ├── index.html
    ├── less
    │   └── ...
    └── scss
        └── ...
```

---

# Angular $ Wrapper Services

___

# $document and $window

- Use [`$document`](https://docs.angularjs.org/api/ng/service/$document) and [`$window`](https://docs.angularjs.org/api/ng/service/$window) instead of `document` and `window`.

*Why?*: These services are wrapped by Angular and more easily testable than using document and window in tests. This helps you avoid having to mock document and window yourself.

___

# $timeout and $interval

- Use [`$timeout`](https://docs.angularjs.org/api/ng/service/$timeout) and [`$interval`](https://docs.angularjs.org/api/ng/service/$interval) instead of `setTimeout` and `setInterval` .

*Why?*: These services are wrapped by Angular and more easily testable and handle Angular's digest cycle thus keeping data binding in sync.

---

# Wait, there's more...

___

# Yes

___

# You can continue educating yourself:

## These slides:

http://jussikinnula.github.io/angularjs-best-practices-20151203.html

## The "original" version:

https://github.com/johnpapa/angular-styleguide#file-templates-and-snippets

---

# Thank You!