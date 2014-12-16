# PointSource AngularJS Style Guide

*Forked from "Opinionated AngularJS style guide for teams" by [@john_papa](//twitter.com/john_papa)*

NOTE: This style guide was started from [John Papa's](https://github.com/johnpapa/angularjs-styleguide) 'AngularJS Style Guide'. This is a work in progress, and we will add or change things as we collaborate while actively implementing many of these best practices in current projects. Feedback is always welcome.

The purpose of this style guide is to provide guidance on building AngularJS applications by showing the conventions we use and why we choose them. 

## Community Awesomeness and Credit
Per John Papa: "Never work in a vacuum. The AngularJS community is an incredible group who are passionate about sharing experiences. As such, a friend  and  AngularJS expert Todd Motto and I have collaborated on many styles and conventions... I encourage you to check out [Todd's  guidelines](https://github.com/toddmotto/angularjs-styleguide) to get a sense for his approach and how it compares. Many of my styles have been from the many pair programming sessions [Ward Bell](http://twitter.com/wardbell) and I have had. While we don't always agree, my friend Ward has certainly helped influence the ultimate evolution of this guide."

## Sample Files
While this guide explains the *what*, *why* and *how*, it's usually helpful to see them in practice. This guide is accompanied by a samples of code inline below as well as sample configuration files in the [assets](https://github.com/PointSource/angularjs-styleguide-public/tree/master/assets) folder. 

Especially take note of the [.jshintrc](https://github.com/PointSource/angularjs-styleguide-public/tree/master/assets/.jshintrc) and [.editorconfig](https://github.com/PointSource/angularjs-styleguide-public/tree/master/assets/.editorconfig) files.  We highly encourage you to use both of these files and have all developers implement them in their code editor of choice. You should also set up your Gruntfile (or Gulp) to run the JSHint task during the "build" and "serve/watch" tasks (example [Gruntfile.js](https://github.com/PointSource/angularjs-styleguide-public/tree/master/assets/Gruntfile.js)). 

Depending on the type of project, you may need to add or change some of the "globals" to ignore (i.e. "angular", "$", "jQuery", etc).  You should also have and use a .jshintrc file for your test directory, which will include more global variables to ignore like "describe", "before", "expect", etc.  See the test directory [.jshintrc file](https://github.com/PointSource/angularjs-styleguide-public/tree/master/assets/test/.jshintrc).

For an explanation of the .jshintrc options, check out the [JSHint Docs](http://www.jshint.com/docs/options/).  And see the [JSHint and EditorConfig](#jshint-and-editorconfig) section at the bottom which explains the sample project settings and instructions for setting up JSHint in the most common IDEs.

## Table of Contents

1. [Single Responsibility](#single-responsibility)
1. [IIFE](#iife)
1. [Modules](#modules)
1. ~~Controllers~~ (These haven't been decided upon, yet)
1. [Services](#services)
1. [Data Services](#data-services)
1. [Directives](#directives)
1. [Resolving Promises for a Controller](#resolving-promises-for-a-controller) (These recommendations have not been implemented in the Entitlement Site codebase, yet)
1. [Manual Dependency Injection](#manual-dependency-injection)
1. ~~Minification and Annotation~~ (These haven't been decided upon, yet)
1. ~~Exception Handling~~ (These haven't been decided upon, yet)
1. [Naming](#naming)
1. [Application Structure LIFT Principle](#application-structure-lift-principle)
1. [Application Structure](#application-structure)
1. [Modularity](#modularity)
1. [Angular $ Wrapper Services](#angular--wrapper-services)
1. [Testing](#testing)
1. [Animations](#animations) 
1. [Comments](#comments)
1. [JSHint](#jshint-and-editorconfig)
1. [Constants](#constants)
1. [AngularJS Docs](#angularjs-docs)
1. [Contributing](#contributing)
1. [License](#license)

## Single Responsibility

- **Rule of 1**: Define 1 component per file.  

The following example defines the `app` module and its dependencies, defines a controller, and defines a factory all in the same file.  

```javascript
/* avoid */
angular
.module('app', ['ngRoute'])
.controller('SomeController' , SomeController)
.factory('someFactory' , someFactory);

function SomeController() { }

function someFactory() { }
```

The same components are now separated into their own files.

```javascript
/* recommended */

// app.module.js
angular
.module('app', ['ngRoute']);
```

```javascript
/* recommended */

// someController.js
angular
.module('app')
.controller('SomeController' , SomeController);

function SomeController() { }
```

```javascript
/* recommended */

// someFactory.js
angular
.module('app')
.factory('someFactory' , someFactory);

function someFactory() { }
```

**[Back to top](#table-of-contents)**

## IIFE
- **IIFE**: Wrap AngularJS components in an Immediately Invoked Function Expression (IIFE). 

*Why?*: An IIFE removes variables from the global scope. This helps prevent variables and function declarations from living longer than expected in the global scope, which also helps avoid variable collisions.

*Why?*: When your code is minified and bundled into a single file for deployment to a production server, you could have collisions of variables and many global variables. An IIFE protects you against both of these by providing variable scope for each file.

```javascript
/* avoid */
// logger.js
angular
.module('app')
.factory('logger', logger);

// logger function is added as a global variable  
function logger () { }

// storage.js
angular
.module('app')
.factory('storage', storage);

// storage function is added as a global variable  
function storage () { }
```


```javascript
/**
* recommended 
*
* no globals are left behind 
*/

// logger.js
(function () {
angular
.module('app')
.factory('logger', logger);

function logger () { }
})();

// storage.js
(function () {
angular
.module('app')
.factory('storage', storage);

function storage () { }
})();
```

- Note: For brevity only, the rest of the examples in this guide may omit the IIFE syntax. 

**[Back to top](#table-of-contents)**

## Modules

- **Definitions (aka Setters)**: Declare modules without a variable using the setter syntax. 

*Why?*: With 1 component per file, there is rarely a need to introduce a variable for the module.

```javascript
/* avoid */
var app = angular.module('app', [
'ngAnimate',
'ngRoute',
'app.shared',
'app.dashboard'
]);
```

Instead use the simple setter syntax.

```javascript
/* recommended */
angular
.module('app', [
'ngAnimate',
'ngRoute',
'app.shared',
'app.dashboard'
]);
```

- **Getters**: When using a module, avoid using a variables and instead use chaining with the getter syntax.

*Why?* : This produces more readable code and avoids variables collisions or leaks.

```javascript
/* avoid */
var app = angular.module('app');
app.controller('SomeController' , SomeController);

function SomeController() { }
```

```javascript
/* recommended */
angular
.module('app')
.controller('SomeController' , SomeController);

function SomeController() { }
```

- **Setting vs Getting**: Only set once and get for all other instances.

*Why?*: A module should only be created once, then retrieved from that point and after.

- Use `angular.module('app', []);` to set a module.
- Use  `angular.module('app');` to get a module. 

- **Named vs Anonymous Functions**: Use named functions instead of passing an anonymous function in as a callback. 

*Why?*: This produces more readable code, is much easier to debug, and reduces the amount of nested callback code.

```javascript
/* avoid */
angular
.module('app')
.controller('Dashboard', function () { });
.factory('logger', function () { });
```

```javascript
/* recommended */

// dashboard.js
angular
.module('app')
.controller('Dashboard', Dashboard);

function Dashboard () { }
```

```javascript
// logger.js
angular
.module('app')
.factory('logger', logger);

function logger () { }
```


**[Back to top](#table-of-contents)**

## Services

- **Singletons**: Services are instantiated with the `new` keyword, use `this` for public methods and variables. Can also use a factory, which we recommend for consistency. 

- Note: [All AngularJS services are singletons](https://docs.angularjs.org/guide/services). This means that there is only one instance of a given service per injector.

```javascript
// service

angular
.module('app')
.service('logger', logger);

function logger () {
this.logError = function (msg) {
/* */
};
}
```

```javascript
// factory
angular
.module('app')
.factory('logger', logger);

function logger () {
return {
logError: function (msg) {
/* */
}
};
}
```

**[Back to top](#table-of-contents)**

## Data Services

- **Separate Data Calls**: Refactor logic for making data operations and interacting with data to a factory. Make data services responsible for XHR calls, local storage, stashing in memory, or any other data operations.

*Why?*: The controller's responsibility is for the presentation and gathering of information for the view. It should not care how it gets the data, just that it knows who to ask for it. Separating the data services moves the logic on how to get it to the data service, and lets the controller be simpler and more focused on the view.

*Why?*: This makes it easier to test (mock or real) the data calls when testing a controller that uses a data service.

*Why?*: Data service implementation may have very specific code to handle the data repository. This may include headers, how to talk to the data, or other services such as $http. Separating the logic into a data service encapsulates this logic in a single place hiding the implementation from the outside consumers (perhaps a controller), also making it easier to change the implementation.

```javascript
/* recommended */

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
- Note: The data service is called from consumers, such as a controller, hiding the implementation from the consumers, as shown below.

```javascript
/* recommended */

// controller calling the dataservice factory
angular
.module('app.avengers')
.controller('Avengers', Avengers);

Avengers.$inject = ['dataservice', 'logger'];

function Avengers(dataservice, logger) {
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
.then(function (data) {
vm.avengers = data;
return vm.avengers;
});
}
}      
```

- **Return a Promise from Data Calls**: When calling a data service that returns a promise such as $http, return a promise in your calling function too.

*Why?*: You can chain the promises together and take further action after the data call completes and resolves or rejects the promise.

```javascript
/* recommended */

activate();

function activate() {
/**
* Step 1
* Ask the getAvengers function for the
* avenger data and wait for the promise
*/
return getAvengers().then(function() {
/**
* Step 4
* Perform an action on resolve of final promise
*/
logger.info('Activated Avengers View');
});
}

function getAvengers() {
/**
* Step 2
* Ask the data service for the data and wait
* for the promise
*/
return dataservice.getAvengers()
.then(function (data) {
/**
* Step 3
* set the data and resolve the promise
*/
vm.avengers = data;
return vm.avengers;
});
}
```

**[Back to top](#table-of-contents)**

## Directives
- **Limit 1 Per File**: Create one directive per file. Name the file for the directive. 

*Why?*: It is easy to mash all the directives in one file, but difficult to then break those out so some are shared across apps, some across modules, some just for one module. 

*Why?*: One directive per file is easy to maintain.

```javascript
/* avoid */
angular
.module('app.widgets')

/* order directive that is specific to the order module */
.directive('orderCalendarRange', orderCalendarRange)

/* sales directive that can be used anywhere across the sales app */
.directive('salesCustomerInfo', salesCustomerInfo)

/* spinner directive that can be used anywhere across apps */
.directive('sharedSpinner', sharedSpinner);

/* implementation details */
```

```javascript
/* recommended */

/**
* @desc order directive that is specific to the order module at a company named Acme
* @file calendarRange.directive.js
* @example <div acme-order-calendar-range></div>
*/
angular
.module('sales.order')
.directive('acmeOrderCalendarRange', orderCalendarRange);

/**
* @desc spinner directive that can be used anywhere across the sales app at a company named Acme
* @file customerInfo.directive.js
* @example <div acme-sales-customer-info></div>
*/    
angular
.module('sales.widgets')
.directive('acmeSalesCustomerInfo', salesCustomerInfo);

/**
* @desc spinner directive that can be used anywhere across apps at a company named Acme
* @file spinner.directive.js
* @example <div acme-shared-spinner></div>
*/
angular
.module('shared.widgets')
.directive('acmeSharedSpinner', sharedSpinner);

/* implementation details */
```

- Note: There are many naming options for directives, especially since they can be used in narrow or wide scopes. Choose one the makes the directive and it's file name distinct and clear. Some examples are below, but see the naming section for more recommendations.

- **Limit DOM Manipulation**: When manipulating the DOM directly, use a directive. If alternative ways can be used such as using CSS to set styles or the [animation services](https://docs.angularjs.org/api/ngAnimate), Angular templating, [`ngShow`](https://docs.angularjs.org/api/ng/directive/ngShow) or [`ngHide`](https://docs.angularjs.org/api/ng/directive/ngHide), then use those instead. For example, if the directive simply hides and shows, use ngHide/ngShow, but if the directive does more, combining hide and show inside a directive may improve performance as it reduces watchers. 

*Why?*: DOM manipulation can be difficult to test, debug, and there are often better ways (e.g. CSS, animations, templating)

- **Restrict to Elements and Attributes**: When creating a directive that makes sense as a standalone element, allow restrict `E` (custom element) and optionally restrict `A` (custom attribute). Generally, if it could be its own control, `E` is appropriate. General guideline is allow `EA` but lean towards implementing as an element when its standalone and as an attribute when it enhances its existing DOM element.

*Why?*: It makes sense.

*Why?*: While we can allow the directive to be used as a class, if the directive is truly acting as an element it makes more sense as an element or at least as an attribute.

```html
<!-- avoid -->
<div class="my-calendar-range"></div>
```

```javascript
/* avoid */
angular
.module('app.widgets')
.directive('myCalendarRange', myCalendarRange);

function myCalendarRange () {
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

```html
<!-- recommended -->
<my-calendar-range></my-calendar-range>
<div my-calendar-range></div>
```

```javascript
/* recommended */
angular
.module('app.widgets')
.directive('myCalendarRange', myCalendarRange);

function myCalendarRange () {
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
- See the 
**[Back to top](#table-of-contents)**

## Resolving Promises for a Controller 

- **Controller Activation Promises**: Resolve start-up logic for a controller in an `activate` function.

*Why?*: Placing start-up logic in a consistent place in the controller makes it easier to locate, more consistent to test, and helps avoid spreading out the activation logic across the controller.

- Note: If you need to conditionally cancel the route before you start use the controller, use a route resolve instead.

```javascript
/* avoid */
function Avengers(dataservice) {
var vm = this;
vm.avengers = [];
vm.title = 'Avengers';

dataservice.getAvengers().then(function(data) {
vm.avengers = data;
return vm.avengers;
});
}
```

```javascript
/* recommended */
function Avengers(dataservice) {
var vm = this;
vm.avengers = [];
vm.title = 'Avengers';

activate();

////////////

function activate() {
return dataservice.getAvengers().then(function(data) {
vm.avengers = data;
return vm.avengers;
});
}
}
```

- **Route Resolve Promises**: When a controller depends on a promise to be resolved, resolve those dependencies in the `$routeProvider` before the controller logic is executed. If you need to conditionally cancel a route before the controller is activated, use a route resolver.

*Why?*: A controller may require data before it loads. That data may come from a promise via a custom factory or [$http](https://docs.angularjs.org/api/ng/service/$http). Using a [route resolve](https://docs.angularjs.org/api/ngRoute/provider/$routeProvider) allows the promise to resolve before the controller logic executes, so it might take action based on that data from the promise.

```javascript
/* avoid */
angular
.module('app')
.controller('Avengers', Avengers);

function Avengers (movieService) {
var vm = this;
// unresolved
vm.movies;
// resolved asynchronously
movieService.getMovies().then(function (response) {
vm.movies = response.movies;
});
}
```

```javascript
/* better */

// route-config.js
angular
.module('app')
.config(config);

function config ($routeProvider) {
$routeProvider
.when('/avengers', {
templateUrl: 'avengers.html',
controller: 'Avengers',
controllerAs: 'vm',
resolve: {
moviesPrepService: function (movieService) {
return movieService.getMovies();
}
}
});
}

// avengers.js
angular
.module('app')
.controller('Avengers', Avengers);

function Avengers (moviesPrepService) {
var vm = this;
vm.movies = moviesPrepService.movies;
}

```

- Note: The code example's dependency on `movieService` is not minification safe on its own. For details on how to make this code minification safe, see the section below on [dependency injection](#manual-dependency-injection).

**[Back to top](#table-of-contents)**

## Manual Dependency Injection

- **UnSafe from Minification**: Avoid using the shortcut syntax of declaring dependencies without using a minification-safe approach.

*Why?*: The parameters to the component (e.g. controller, factory, etc) will be converted to mangled variables. For example, `common` and `dataservice` may become `a` or `b` and not be found by AngularJS.

```javascript
/* avoid - not minification-safe*/
angular
.module('app')
.controller('Dashboard', Dashboard);

function Dashboard(common, dataservice) {
}
```

- This code may produce mangled variables when minified and thus cause runtime errors.

```javascript
/* avoid - not minification-safe*/
angular.module('app').controller('Dashboard', d);function d(a, b) { }
```

- **Manually Identify Dependencies**: Use $inject to manually identify your dependencies for AngularJS components.

*Why?*: This technique mirrors the technique used by [`ng-annotate`](https://github.com/olov/ng-annotate), which is recommend for automating the creation of minification safe dependencies. If `ng-annotate` detects injection has already been made, it will not duplicate it.

*Why?*: This safeguards your dependencies from being vulnerable to minification issues when parameters may be mangled. For example, `common` and `dataservice` may become `a` or `b` and not be found by AngularJS.

*Why?*: Avoid creating inline dependencies as long lists can be difficult to read in the array. Also it can be confusing that the array is a series of strings while the last item is the component's function. 

```javascript
/* avoid */
angular
.module('app')
.controller('Dashboard', 
['$location', '$routeParams', 'common', 'dataservice', Dashboard]);

function Dashboard($location, $routeParams, common, dataservice) {
}
```

```javascript
/* recommended */
angular
.module('app')
.controller('Dashboard', Dashboard);

Dashboard.$inject = ['$location', '$routeParams', 'common', 'dataservice'];

function Dashboard($location, $routeParams, common, dataservice) {
}
```

- Note: When your function is below a return statement the $inject may be unreachable (this may happen in a directive). You can solve this by either moving the $inject above the return statement or by using the alternate array injection syntax.

```javascript
// inside a directive definition
function outer() {
return {
controller: DashboardPanel,
};

DashboardPanel.$inject = ['logger']; // Unreachable
function DashboardPanel(logger) {
}
}
```

```javascript
// inside a directive definition
function outer() {
DashboardPanel.$inject = ['logger']; // reachable
return {
controller: DashboardPanel,
};

function DashboardPanel(logger) {
}
}
```

- **Manually Identify Route Resolver Dependencies**: Use $inject to manually identify your route resolver dependencies for AngularJS components.

*Why?*: This technique breaks out the anonymous function for the route resolver, making it easier to read.

*Why?*: An `$inject` statement can easily procede the resolver to handle making any dependencies minification safe.

```javascript
/* recommended */
function config ($routeProvider) {
$routeProvider
.when('/avengers', {
templateUrl: 'avengers.html',
controller: 'Avengers',
controllerAs: 'vm',
resolve: {
moviesPrepService: moviePrepService
}
});
}

moviePrepService.$inject =  ['movieService'];
function moviePrepService(movieService) {
return movieService.getMovies();
}
```


**[Back to top](#table-of-contents)**

## Naming

- **Naming Guidelines**: Use consistent names for all components following a pattern that describes the component's feature then (optionally) its type. My recommended pattern is `feature.type.js`. There are 2 names for most assets:
*   the file name 
*   the registered asset name with Angular

*Why?*: Naming conventions help provide a consistent way to find content at a glance. Consistency within the project is vital. Consistency with a team is important. Consistency across a company provides tremendous efficiency.

*Why?*: The naming conventions should simply help the findability and communication of code. 

- **Feature File Names**: Use consistent names for all components following a pattern that describes the component's feature then (optionally) its type. Our recommended pattern is `feature.type.js`.

*Why?*: Provides a consistent way to quickly identify components.

*Why?*: Provides pattern matching for any automated tasks.

```javascript
/**
* common options 
*/

// Controllers
avengers.js
avengers.controller.js
avengersController.js
avengersCtrl.js

// Services/Factories
logger.js
logger.service.js
loggerService.js
loggerSvc.js
```

```javascript
/**
* recommended
*/

// controllers
avengers.controller.js
avengers.controller.spec.js

// services/factories
logger.service.js
logger.service.spec.js

// constants
constants.js

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

- Alternative: Another common convention is naming controller files without the word `controller` in the file name such as `avengers.js` instead of `avengers.controller.js`. All other conventions still hold using a suffix of the type. Controllers are the most common type of component so this just saves typing and is still easily identifiable. We recommend you choose one convention and be consistent for your team.

```javascript
/**
* recommended
*/
// Controllers
avengers.js
avengers.spec.js
```

- **Test File Names**: Name test specifications similar to the component they test with a suffix of `spec`.  

*Why?*: Provides a consistent way to quickly identify components.

*Why?*: Provides pattern matching for [karma](http://karma-runner.github.io/) or other test runners.

```javascript
/**
* recommended
*/
avengers.controller.spec.js
logger.service.spec.js
avengers.routes.spec.js
avenger-profile.directive.spec.js
```

- **Controller Names**: Use consistent names for all controllers named after their feature. Use pascal-casing for controllers, as they are constructors.

*Why?*: Provides a consistent way to quickly identify and reference controllers.

*Why?*: Pascal-casing is conventional for identifying object that can be instantiated using a constructor.

```javascript
/**
* recommended
*/

// avengers.controller.js
angular
.module
.controller('Avengers', Avengers);

function Avengers(){ }
```

- **Factory Names**: Use consistent names for all factories named after their feature. Use pascal-casing for service and factory names but camel-case for the functions within.

*Why?*: Provides a consistent way to quickly identify and reference controllers.

```javascript
/**
* recommended
*/

// logger.service.js
angular
.module
.factory('Logger', Logger);

function logger(){ }
```

- **Directive Component Names**: Use consistent names for all directives using camel-case. Use a short prefix to describe the area that the directives belong (some example are company prefix or project prefix).

*Why?*: Provides a consistent way to quickly identify and reference components.

```javascript
/**
* recommended
*/

// avenger.profile.directive.js    
angular
.module
.directive('xxAvengerProfile', xxAvengerProfile);

// usage is <xx-avenger-profile> </xx-avenger-profile>

function xxAvengerProfile(){ }
```

- **Modules**:  When there are multiple modules, the main module file is named `app.module.js` while other dependent modules are named after what they represent. For example, an admin module is named `admin.module.js`. The respective registered module names would be `app` and `admin`. A single module app might be named `app.js`, omitting the module moniker.

*Why?*: An app with 1 module is named `app.js`. It is the app, so why not be super simple.

*Why?*: Provides consistency for multiple module apps, and for expanding to large applications.

*Why?*: Provides easy way to use task automation to load all module definitions first, then all other angular files (for bundling).

- **Configuration**: Separate configuration for a module into its own file named after the module. A configuration file for the main `app` module is named `app.config.js` (or simply `config.js`). A configuration for a module named `admin.module.js` is named `admin.config.js`.

*Why?*: Separates configuration from module definition, components, and active code.

*Why?*: Provides a identifiable place to set configuration for a module.

- **Routes**: Separate route configuration into its own file. Examples might be `app.route.js` for the main module and `admin.route.js` for the `admin` module. Even in smaller apps this separation from the rest of the configuration is preferred. An alternative is a longer name such as `admin.config.route.js`.


**[Back to top](#table-of-contents)**

## Application Structure LIFT Principle
- **LIFT**: Structure your app such that you can `L`ocate your code quickly, `I`dentify the code at a glance, keep the `F`lattest structure you can, and `T`ry to stay DRY. The structure should follow these 4 basic guidelines. 

*Why LIFT?*: Provides a consistent structure that scales well, is modular, and makes it easier to increase developer efficiency by finding code quickly. Another way to check your app structure is to ask yourself: How quickly can you open and work in all of the related files for a feature?

1. `L`ocating our code is easy
2. `I`dentify code at a glance
3. `F`lat structure as long as we can
4. `T`ry to stay DRY (Don’t Repeat Yourself) or T-DRY

- **Locate**: Make locating your code intuitive, simple and fast.

*Why?*: If the team cannot find the files they need to work on quickly, they will not be able to work as efficiently as possible, and the structure needs to change. You may not know the file name or where its related files are, so putting them in the most intuitive locations and near each other saves a ton of time. A descriptive folder structure can help with this.

```javascript
/bower_components
/client
/app
/avengers
/layout
/blocks
/exception
/logger
/core
/dashboard
/layout
/data
/layout /* shared layout like header and footer */
/shared /* possibly "services" instead */
/widgets
/layout
index.html
.bower.json
```

- **Identify**: When you look at a file you should instantly know what it contains and represents.

*Why?*: You spend less time hunting and pecking for code, and become more efficient. If this means you want longer file names, then so be it. Be descriptive with file names and keeping the contents of the file to exactly 1 component. Avoid files with multiple controllers, multiple services, or a mixture. There are deviations of the 1 per file rule when there is a set of very small features that are all related to each other, they are still easily identifiable.

- **Flat**: Keep a flat folder structure as long as possible. When you get to 7+ files, begin considering separation.

*Why?*: Nobody wants to search 7 levels of folders to find a file. Think about menus on web sites … anything deeper than 2 should take serious consideration. In a folder structure there is no hard and fast number rule, but when a folder has 7-10 files, that may be time to create subfolders. Base it on your comfort level. Use a flatter structure until there is an obvious value (to help the rest of LIFT) in creating a new folder.

- **T-DRY (Try to Stick to DRY)**: Be DRY, but don't go nuts and sacrifice readability.

*Why?*: Being DRY is important, but not crucial if it sacrifices the others in LIFT, which is why it could be called  T-DRY. We don’t want to type session-view.html for a view because, well, it’s obviously a view. If it is not obvious or by convention, then name it. 

- Note: We modified this section as well as the 1 below (Application Structure) from the original John Papa's version, because we all agreed that 'layout' folders should be under each module and also one in the root level for shared layout like headers and footers (wrapper-type layout) if needed.

**[Back to top](#table-of-contents)**

## Application Structure

- **Overall Guidelines**:  Have a near term view of implementation and a long term vision. In other words, start small and but keep in mind on where the app is heading down the road. All of the app's code goes in a root folder named `app`. All content is 1 feature per file. Each controller, service, module, view is in its own file. Small deviations are OK for a set of small, short directives in a `directive.js` file. All 3rd party vendor scripts are stored in another root folder and not in the `app` folder. If you or your team didn't write them, you don't want them cluttering your app (`bower_components`, `scripts`, `lib`).

- Note: Find more details and reasoning behind the structure at [this original post on application structure](http://www.johnpapa.net/angular-app-structuring-guidelines/).

- **Layout**: Place files that define the layout of each main module of the application in a folder named `layout` inside of each module directory (like below example). These may include a shell view and controller may act as the container for the app, navigation, menus, content areas, and other regions. 

*Why?*: Keeps the layout structure within each module/component folder for better reusability.

- **Folders-by-Feature Structure**: Create folders named for the feature they represent. When a folder grows to contain more than 7 files, start to consider creating a folder for them. Your threshold may be different, so adjust as needed. 

*Why?*: A developer can locate the code, identify what each file represents at a glance, the structure is flat as can be, and there is no repetitive nor redundant names. 

*Why?*: The LIFT guidelines are all covered.

*Why?*: Helps reduce the app from becoming cluttered through organizing the content and keeping them aligned with the LIFT guidelines.

*Why?*: When there are a lot of files (10+) locating them is easier with a consistent folder structures and more difficult in flat structures.

```javascript
/**
* recommended
*/

app/
app.module.js
app.config.js
app.routes.js
directives/  /* shared */      
calendar.directive.js  
calendar.directive.html  
user-profile.directive.js  
user-profile.directive.html  
layout/ /* shared */
footer.html
header.html
header.js
layout.less
people/
layout/
layout.controller.js
layout.html
layout.less
attendees.html
attendees.controller.js  
speakers.html
speakers.controller.js
speaker-detail.html
speaker-detail.controller.js
services/  /* shared */ 
data.service.js  
localstorage.service.js
logger.service.js   
spinner.service.js
sessions/
layout/
layout.controller.js
layout.html
sessions.html      
sessions.controller.js
session-detail.html
session-detail.controller.js  
```

- Note: Structuring using folders-by-type is another common option. It requires moving to multiple folders when working on a feature. This could get unwieldy quickly as the app grows to 5, 10 or 25+ views and controllers (and other features), which makes it more difficult than folder-by-feature to locate files.  
**We highly recommend using the 'Folders by Feature Structure' described above for the majority of projects unless the project is *extremely small* and not predicted to grow/expand.**

```javascript
/* 
* Alternative folders-by-type.
* We recommend "folders-by-feature", instead.
*/

app/
app.module.js
app.config.js
app.routes.js
controllers/
attendees.js            
session-detail.js       
sessions.js             
shell.js                
speakers.js             
speaker-detail.js       
topnav.js               
directives/       
calendar.directive.js  
calendar.directive.html  
user-profile.directive.js  
user-profile.directive.html  
services/       
dataservice.js  
localstorage.js
logger.js   
spinner.js
views/
attendees.html     
session-detail.html
sessions.html      
shell.html         
speakers.html      
speaker-detail.html
topnav.html         
``` 

**[Back to top](#table-of-contents)**

## Modularity

- **Many Small, Self Contained Modules**: Create small modules that enapsulate one responsibility.

*Why?*: Modular applications make it easy to plug and go as they allow the development teams to build vertical slices of the applications and roll out incrementally.  This means we can plug in new features as we develop them.

- **Create an App Module**: Create an application root module whose role is pull together all of the modules and features of your application. Name this for your application.

*Why?*: AngularJS encourages modularity and separation patterns. Creating an application root module whose role is to tie your other modules together provides a very straightforward way to add or remove modules from your application.

- **Keep the App Module Thin**: Only put logic for pulling together the app in the application module. Leave features in their own modules.

*Why?*: Adding additional roles to the application root to get remote data, display views, or other logic not related to pulling the app together muddies the app module and make both sets of features harder to reuse or turn off.

- **Feature Areas are Modules**: Create modules that represent feature areas, such as layout, reusable and shared services, dashboards, and app specific features (e.g. customers, admin, sales).

*Why?*: Self contained modules can be added to the application will little or no friction.

*Why?*: Sprints or iterations can focus on feature areas and turn them on at the end of the sprint or iteration.

*Why?*: Separating feature areas into modules makes it easier to test the modules in isolation and reuse code. 

- **Reusable Blocks are Modules**: Create modules that represent reusable application blocks for common services such as exception handling, logging, diagnostics, security, and local data stashing.

*Why?*: These types of features are needed in many applications, so by keeping them separated in their own modules they can be application generic and be reused across applications.

- **Module Dependencies**: The application root module depends on the app specific feature modules, the feature modules have no direct dependencies, the cross-application modules depend on all generic modules.

![Modularity and Dependencies](https://github.com/PointSource/angularjs-styleguide-public/blob/master/assets/modularity-1.png)

*Why?*: The main app module contains a quickly identifiable manifest of the application's features. 

*Why?*: Cross application features become easier to share. The features generally all rely on the same cross application modules, which are consolidated in a single module (`app.core` in the image).

*Why?*: Intra-App features such as shared data services become easy to locate and share from within `app.core` (choose your favorite name for this module).

- Note: This is a strategy for consistency. There are many good options here. Choose one that is consistent, follows AngularJS's dependency rules, and is easy to maintain and scale.

>> Structures may vary slightly between projects but they should all follow these guidelines for structure and modularity. The implementation may vary depending on the features and the team. In other words, don't get hung up on an exact like-for-like structure but do justify your structure using consistency, maintainability, and efficiency in mind. 


**[Back to top](#table-of-contents)**

## Angular $ Wrapper Services

- **$document and $window**: Use [`$document`](https://docs.angularjs.org/api/ng/service/$document) and [`$window`](https://docs.angularjs.org/api/ng/service/$window) instead of `document` and `window`.

*Why?*: These services are wrapped by Angular and more easily testable than using document and window in tests. This helps you avoid having to mock document and window yourself.

- **$timeout and $interval**: Use [`$timeout`](https://docs.angularjs.org/api/ng/service/$timeout) and [`$interval`](https://docs.angularjs.org/api/ng/service/$interval) instead of `setTimeout` and `setInterval` .

*Why?*: These services are wrapped by Angular and more easily testable and handle AngularJS's digest cycle thus keeping data binding in sync.

**[Back to top](#table-of-contents)**

## Testing
Unit testing helps maintain clean code, as such we included some of our recommendations for unit testing foundations with links for more information.

- **Write Tests with Stories**: Write a set of tests for every story. Start with an empty test and fill them in as you write the code for the story.

*Why?*: Writing the test descriptions helps clearly define what your story will do, will not do, and how you can measure success.

```javascript
it('should have Avengers controller', function () {
//TODO
});

it('should find 1 Avenger when filtered by name', function () {
//TODO
});

it('should have 10 Avengers', function () {}
//TODO (mock data?)
});

it('should return Avengers via XHR', function () {}
//TODO ($httpBackend?)
});

// and so on
```

- **Testing Library**: Use [Jasmine](http://jasmine.github.io/) or [Mocha](http://visionmedia.github.io/mocha/) for unit testing.

*Why?*: Both Jasmine and Mocha are widely used in the AngularJS community. Both are stable, well maintained, and provide robust testing features.

Note: When using Mocha, also consider choosing an assert library such as [Chai](http://chaijs.com).

- **Test Runner**: Use [Karma](http://karma-runner.github.io) as a test runner.

*Why?*: Karma is easy to configure to run once or automatically when you change your code.

*Why?*: Karma hooks into your Continuous Integration process easily on its own or through Grunt or Gulp.

*Why?*: Some IDE's are beginning to integrate with Karma, such as [WebStorm](http://www.jetbrains.com/webstorm/) and [Visual Studio](http://visualstudiogallery.msdn.microsoft.com/02f47876-0e7a-4f6c-93f8-1af5d5189225).

*Why?*: Karma works well with task automation leaders such as [Grunt](http://www.gruntjs.com) (with [grunt-karma](https://github.com/karma-runner/grunt-karma)) and [Gulp](http://www.gulpjs.com) (with [gulp-karma](https://github.com/lazd/gulp-karma)).

- **Stubbing and Spying**: Use Sinon for stubbing and spying.

*Why?*: Sinon works well with both Jasmine and Mocha and extends the stubbing and spying features they offer.

*Why?*: Sinon makes it easier to toggle between Jasmine and Mocha, if you want to try both.

- **Headless Browser**: Use [PhantomJS](http://phantomjs.org/) to run your tests on a server.

*Why?*: PhantomJS is a headless browser that helps run your tests without needing a "visual" browser. So you do not have to install Chrome, Safaria, IE, or other browsers on your server. 

Note: You should still test on all browsers in your environment, as appropriate for your target audience.

- **Code Analysis**: Run JSHint on your tests. 

**Why?*: Tests are code. JSHint can help identify code quality issues that may cause the test to work improperly.

- **Alleviate JSHint Rules on Tests**: Relax the rules on your test code.

**Why?*: Your tests won't be run by your end users and do not require as strenuous of code quality rules. Global variables, for example, can be relaxed by including this in your test specs.

```javascript
/*global sinon, describe, it, afterEach, beforeEach, expect, inject */
```

![Testing Tools](https://github.com/PointSource/angularjs-styleguide/blob/master/assets/testing-tools.png)

**[Back to top](#table-of-contents)**

## Animations

- **Usage**: Use subtle [animations with AngularJS](https://docs.angularjs.org/guide/animations) to transition between states for views and primary visual elements. Include the [ngAnimate module](https://docs.angularjs.org/api/ngAnimate). The 3 keys are subtle, smooth, seamless.

*Why?*: Subtle animations can improve User Experience when used appropriately.

*Why?*: Subtle animations can improve perceived performance as views transition.

- **Sub Second**: Use short durations for animations. Generally start with 300ms and adjust until appropriate.  

*Why?*: Long animations can have the reverse affect on User Experience and perceived performance by giving the appearance of a slow application.

**[Back to top](#table-of-contents)**

## Comments

- **jsDoc**: If planning to produce documentation, use [`jsDoc`](http://usejsdoc.org/) syntax to document function names, description, params and returns

*Why?*: You can generate (and regenerate) documentation from your code, instead of writing it from scratch.

*Why?*: Provides consistency using a common industry tool.

```javascript
angular
.module('app')
.factory('logger', logger);

/**
* @name logger
* @desc Application wide logger
*/
function logger ($log) {
var service = {
logError: logError
};
return service;

////////////

/**
* @name logError
* @desc Logs errors
* @param {String} msg Message to log 
* @returns {String}
*/
function logError(msg) {
var loggedMsg = 'Error: ' + msg;
$log.error(loggedMsg);
return loggedMsg;
};
}
```
- Note: We do recommend using jsDoc for when practical, but haven't implemented it in the Entitlement Site codebase, yet. Some of the newer projects have started using it.

**[Back to top](#table-of-contents)**

## JSHint and EditorConfig

- **Use an [Options File](https://github.com/PointSource/angularjs-styleguide/tree/master/assets/.jshintrc)**: Use JSHint for linting your JavaScript and be sure to customize the globals in your [JSHint options file](https://github.com/PointSource/angularjs-styleguide/tree/master/assets/.jshintrc) and include in source control. See the [JS Hint docs](http://www.jshint.com/docs/) for details on the options and see below for the ones we've chosen.
- For IDE plugins and examples of setup:
- SublimeText (2 & 3): [SublimeLinter](https://github.com/SublimeLinter/SublimeLinter-for-ST2)
- Eclipse: [jshint-eclipse](http://github.eclipsesource.com/jshint-eclipse/)
- WebStorm: [built in configuration](http://www.jetbrains.com/webstorm/webhelp/using-javascript-code-quality-tools.html#d142224e63)
- Textmate: [JSHint.tmbundle](https://github.com/bodnaristvan/JSHint.tmbundle)
- Visual Studio: [SharpLinter](https://github.com/jamietre/SharpLinter)
- Brackets / Edge Code: [Brackets JSHint](https://github.com/cfjedimaster/brackets-jshint/) or [Brackets Interactive Linter](https://github.com/MiguelCastillo/Brackets-InteractiveLinter)
- Notepad++: [jslintnpp](http://sourceforge.net/projects/jslintnpp/)
- Komodo 7: [built-in](http://www.activestate.com/blog/2011/05/komodo-7-alpha-2-improved-syntax-checking)
- List of Plugins from JSHint site: [List](http://www.jshint.com/install/)

*Why?*: Provides a first alert prior to committing any code to source control.

*Why?*: Provides consistency across your team.

Example .jshintrc file:

```javascript
{
//Enforcing
"bitwise": true,            // true: Prohibit bitwise operators (&, |, ^, etc.)
"camelcase": true,          // true: Identifiers must be in camelCase
"curly": true,              // true: Require {} for every new block or scope
"eqeqeq": true,             // true: Require triple equals (===) for comparison
"es3": false,               // false: Code does NOT need to adhere to ECMAScript 3 specification
"forin": true,              // true: Require filtering for..in loops with obj.hasOwnProperty()
"freeze": true,             // true: Prohibits overwriting prototypes of native objects such as Array, Date and so on.
"immed": true,              // true: Require immediate invocations to be wrapped in parens e.g. `(function () { } ());`             
"indent": 4,                // {int} Number of spaces to use for indentation
"latedef": "nofunc",        // true: Require variables/functions to be defined before being used; "nofunc" will allow function declarations to be ignored.
"newcap": true,             // true: Require capitalization of all constructor functions e.g. `new F()`
"noarg": true,              // true: Prohibit use of `arguments.caller` and `arguments.callee`
"noempty": true,            // true: Prohibit use of empty blocks
"nonbsp": true,             // true: Warns about "non-breaking whitespace" characters.
"nonew": true,              // true: Prohibit use of constructors for side-effects (without assignment)
"plusplus": false,          // false: Does NOT Prohibit use of `++` & `--`
"quotmark": "single",       // "single" : require single quotes
"undef": true,              // true: Require all non-global variables to be declared (prevents global leaks)
"unused": false,            // true: Require all defined variables be used (false does NOT)
"strict": false,            // true: Requires all functions run in ES5 Strict Mode (false does NOT)
"maxlen": 120,              // {int} Max number of characters per line

//Relaxing
"asi": false,               // false: Warn about missing semicolons (true tolerates missing semicolons)
"boss": false,              // false: Don't suppress warnings about the use of assignments in cases where comparisons are expected (true allows them)
"debug": false,             // false: Warn about debugger statements e.g. browser breakpoints. (true allows them)
"eqnull": true,             // true: Tolerate use of `== null`     
"esnext": false,            // false: Don't allow ES.next (ES6) syntax (ex: `const`)
"evil": false,              // false: Warn about use of `eval` and `new Function()`
"expr": false,              // false: Warn about the use of expressions where normally you would expect to see assignments or function calls.
"funcscope": false,         // false: Warn about defining variables inside control statements
"globalstrict": false,      // false: Don't allow global "use strict" 
"iterator": false,          // false: Warn about using the `__iterator__` property
"lastsemic": false,         // false: Warn when omitting a semicolon for the last statement of a 1-line block
"laxbreak": false,          // false: Warn about possibly unsafe line breakings
"laxcomma": false,          // false: Warn about comma-first style coding      
"loopfunc": true,           // true: Tolerate functions being defined in loops
"maxerr": false,            // false: Doesn't set a maximum number of errors before stopping. Default is 50. {int} Maximum error before stopping
"moz": false,               // false: Don't allow Mozilla specific syntax (extends and overrides esnext features)
"multistr": false,          // false: Don't tolerate multi-line strings
"notypeof": false,          // false: Warn about invalid typeof operator values 
"proto": false,             // false: Warn about using the __proto__ property.
"scripturl": false,         // false: Warn about script-targeted URLs
"shadow": false,            // false: Warn about re-defined variables later in code
"sub": true,                // true: Tolerate using `[]` notation when it can still be expressed in dot notation
"supernew": false,          // false: Warn about `new function () { ... };` and `new Object;`
"validthis": false,         // false: Warn about using 'this' in a non-constructor function
"noyield": false,           // false: Warn about generator functions with no 'yield' statement in them.

//Environments
"browser": true,            // Web Browser (window, document, etc)
"node": true,               // Node.js

//Other Environments you may need
"browserify": false,        // Browserify (node.js code in the browser)
"couch": false,             // CouchDB
"devel": true,              // Development/debugging (alert, confirm, etc)
"dojo": false,              // Dojo Toolkit
"jquery": false,            // jQuery
"nonstandard": false,       // Widely adopted globals (escape, unescape, etc)
"prototypejs": false,       // Prototype and Scriptaculous
"rhino": false,             // Rhino
"worker": false,            // Web Workers

// Custom Globals
"globals": {
"angular": false,
"$": false,
// Others you may need
"WL": false,
"google": false,
"FastClick": false,
"Chart": false,
"gapi": false,
"_": false
}
}
```

- **To relax a JSHint rule**: Use inline configuration to ignore/relax a JSHint rule by adding a comment above the "offending" code.  For example:
```javascript
/*jshint camelcase:false*/

.... code that requires a variable to not use camelcase, i.e. 'access_token'

/*jshint camelcase:true*/
```
- If used inside of a function, you may not need the last line turning the rule back on since it is scoped to just that function.  See the section on "Inline Configuration" in the [JSHint Docs](http://www.jshint.com/docs/).

- **Use an [EditorConfig](https://github.com/PointSource/angularjs-styleguide/tree/master/assets/.editorconfig) file**:  EditorConfig helps developers define and maintain consistent coding styles between different editors and IDEs. JSHint provides warnings in editors when code violates its settings and throws errors during the build process, whereas EditorConfig automatically enforces some of those styles for you within the editor/IDE.  It is mainly used to define spacing, end-of-line, indentation styles, trimming trailing whitespace and can be set for different file types.  All of the options and example file settings can be found on the main [EditorConfig website](http://editorconfig.org), as well as links to all of the plugins available for different IDEs. WebStorm 9 now has support build in without the need of a plugin.  Unfortunately, at this time, there is not a plugin or support available for Eclipse.  *:-(*

**[Back to top](#table-of-contents)**

## Constants

- **Vendor Globals**: Create an AngularJS Constant for vendor libraries' global variables.

*Why?*: Provides a way to inject vendor libraries that otherwise are globals. This improves code testability by allowing you to more easily know what the dependencies of your components are (avoids leaky abstractions). It also allows you to mock these dependencies, where it makes sense.

```javascript
// constants.js

/* global toastr:false, moment:false */
(function () {
'use strict';

angular
.module('app.core')
.constant('toastr', toastr)
.constant('moment', moment);
})();
```

**[Back to top](#table-of-contents)**

## AngularJS docs
For anything else, API reference, check the [Angular documentation](//docs.angularjs.org/api).

## Contributing

Open an issue first to discuss potential changes/additions. If you have questions with the guide, feel free to leave them as issues in the repo. If you find a typo, create a pull request. The idea is to keep the content up to date and use github’s native feature to help tell the story with issues and PR’s, which are all searchable via google. Why? Because odds are if you have a question, someone else does too! You can learn more here at about how to contribute.

*By contributing to this repo you are agreeing to make your content available subject to the license of this repo.*


- **Process**
1. Discuss the changes in an Issue. 
1. Open a Pull Request, reference the issue, and explain the change and why it adds value.
1. The Pull Request will be evaluated and either merged or declined.

## License

- **tldr;** Use this guide. Attributions are appreciated, but not required. 

#### (The MIT License)

Copyright (c) 2014 [John Papa](http://johnpapa.net)

Permission is hereby granted, free of charge, to any person obtaining
a copy of this software and associated documentation files (the
'Software'), to deal in the Software without restriction, including
without limitation the rights to use, copy, modify, merge, publish,
distribute, sublicense, and/or sell copies of the Software, and to
permit persons to whom the Software is furnished to do so, subject to
the following conditions:

The above copyright notice and this permission notice shall be
included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED 'AS IS', WITHOUT WARRANTY OF ANY KIND,
EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY
CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT,
TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE
SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

**[Back to top](#table-of-contents)**
