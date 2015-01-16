# AngularJS Coffeescript Style Guide

*AngularJS Coffeescript style guide for [@Yoolk Team](//github.com/yoolk)*


## Single Responsibility

  - Define 1 component per file.  

  The following example defines the `app` module and its dependencies, defines a controller, and defines a factory all in the same file.  

  ```coffeescript
  # avoid
  angular
    .module('app', ['ngRoute'])
    .controller('SomeController', SomeController)
    .factory('someFactory', someFactory)
    
  SomeController = ->
    # implementation details

  someFactory = ->
    # implementation details
  
  ```
    
  The same components are now separated into their own files.

  ```coffeescript
  # recommended 
  
  # app.module.js.coffee
  angular
    .module('app', ['ngRoute'])
  ```

  ```coffeescript
  # recommended
  
  # someController.js.coffee
  angular
    .module('app')
    .controller('SomeController', SomeController)

  SomeController = ->
    # implementation details
  ```

  ```coffeescript
  # recommended
  
  # someFactory.js.coffee
  angular
    .module('app')
    .factory('someFactory', someFactory)
    
  someFactory = ->
    # implementation details
  ```

## Modules

  - Declare modules without a variable using the setter syntax. 

  *Why?*: With 1 component per file, there is rarely a need to introduce a variable for the module.
  
  ```coffeescript
  # avoid
  app = angular.module('app', [
    'ngAnimate',
    'ngRoute',
    'app.shared',
    'app.dashboard'
  ])
  ```

  Instead use the simple setter syntax.

  ```coffeescript
  # recommended
  angular
    .module('app', [
      'ngAnimate',
      'ngRoute',
      'app.shared',
      'app.dashboard'
    ])
  ```

### Getters

  - When using a module, avoid using a variable and instead use chaining with the getter syntax.

  *Why?* : This produces more readable code and avoids variable collisions or leaks.

  ```coffeescript
  # avoid
  app = angular.module('app')
  app.controller('SomeController', SomeController)
  
  SomeController = ->
    # implementation details
  ```

  ```coffeescript
  # recommended
  angular
    .module('app')
    .controller('SomeController', SomeController)
  
  SomeController = ->
    # implementation details
  ```

### Setting vs Getting

  - Only set once and get for all other instances.
  
  *Why?*: A module should only be created once, then retrieved from that point and after.
      
    - Use `angular.module('app', [])` to set a module.
    - Use  `angular.module('app')` to get a module. 

### Named vs Anonymous Functions

  - Use named functions instead of passing an anonymous function in as a callback. 

  *Why?*: This produces more readable code, is much easier to debug, and reduces the amount of nested callback code.

  ```coffeescript
  # avoid
  angular
    .module('app')
    .controller('Dashboard', ->
      # implementation details
    )
    .factory('logger', ->
      # implementation details
    )
  ```

  ```coffeescript
  # recommended

  # dashboard.js.coffee
  angular
    .module('app')
    .controller('Dashboard', Dashboard)

  Dashboard = ->
    # implementation details
  ```

  ```coffeescript
  # logger.js.coffee
  angular
    .module('app')
    .factory('logger', logger)

  logger = -> 
    # implementation details
  ```

## Controllers

### controllerAs View Syntax

  - Use the [`controllerAs`](http://www.johnpapa.net/do-you-like-your-angular-controllers-with-or-without-sugar/) syntax over the `classic controller with $scope` syntax. 

  *Why?*: Controllers are constructed, "newed" up, and provide a single new instance, and the `controllerAs` syntax is closer to that of a JavaScript constructor than the `classic $scope syntax`. 

  *Why?*: It promotes the use of binding to a "dotted" object in the View (e.g. `customer.name` instead of `name`), which is more contextual, easier to read, and avoids any reference issues that may occur without "dotting".

  *Why?*: Helps avoid using `$parent` calls in Views with nested controllers.

  ```html
  <!-- avoid -->
  <div ng-controller='Customer'>
    {{ name }}
  </div>
  ```

  ```html
  <!-- recommended -->
  <div ng-controller='Customer as customer'>
    {{ customer.name }}
  </div>
  ```

### controllerAs Controller Syntax

  - Use the `controllerAs` syntax over the `classic controller with $scope` syntax. 

  - The `controllerAs` syntax uses `this` inside controllers which gets bound to `$scope`

  *Why?*: `controllerAs` is syntactic sugar over `$scope`. You can still bind to the View and still access `$scope` methods.  

  *Why?*: Helps avoid the temptation of using `$scope` methods inside a controller when it may otherwise be better to avoid them or move them to a factory. Consider using `$scope` in a factory, or if in a controller just when needed. For example when publishing and subscribing events using [`$emit`](https://docs.angularjs.org/api/ng/type/$rootScope.Scope#$emit), [`$broadcast`](https://docs.angularjs.org/api/ng/type/$rootScope.Scope#$broadcast), or [`$on`](https://docs.angularjs.org/api/ng/type/$rootScope.Scope#$on) consider moving these uses to a factory and invoke from the controller. 

  ```coffeescript
  # avoid
  Customer = ($scope) ->
    $scope.name = {}
    $scope.sendMessage = ->
      # implementation details
  ```

  ```coffeescript
  # recommended - but see next section
  Customer = ->
    @name = {}
    @sendMessage = ->
      # implementation details
  ```

### Bindable Members Up Top

  - Place bindable members at the top of the controller, alphabetized, and not spread through the controller code.
  
    *Why?*: Placing bindable members at the top makes it easy to read and helps you instantly identify which members of the controller can be bound and used in the View. 

    *Why?*: Setting anonymous functions in-line can be easy, but when those functions are more than 1 line of code they can reduce the readability. Defining the functions below the bindable members (the functions will be hoisted) moves the implementation details down, keeps the bindable members up top, and makes it easier to read. 

  ```coffeescript
  # avoid
  Sessions = ->
    @gotoSession = ->
      # implementation details
    @refresh = ->
      # implementation details
    @search = ->
      # implementation details
    @sessions = []
    @title = 'Sessions'
  ```

  ```coffeescript
  # recommended
  Sessions = ->
    @gotoSession = gotoSession
    @refresh = refresh
    @search = search
    @sessions = []
    @title    = 'Sessions'
    
    ################  
    
    @gotoSession = ->
      # implementation details
    @refresh = ->
      # implementation details
    @search = ->
      # implementation details
    @sessions = []
    @title = 'Sessions'
  
  ```

  Note: If the function is a 1 liner consider keeping it right up top, as long as readability is not affected.

  ```coffeescript
  # avoid
  Sessions = (data)->
    @gotoSession = gotoSession
    @refresh = ->
      # lines
      # of
      # code
      # affects
      # readability
    @search = search
    @sessions = []
    @title = 'Sessions'
  ```

  ```coffeescript
  # recommended
  Sessions = (dataservice)->
    @gotoSession = gotoSession
    @refresh = dataservice.refresh # 1 liner is OK
    @search = search
    @sessions = []
    @title = 'Sessions'
  ```

### Keep Controllers Focused

  - Define a controller for a view, and try not to reuse the controller for other views. Instead, move reusable logic to factories and keep the controller simple and focused on its view. 
  
    *Why?*: Reusing controllers with several views is brittle and good end to end (e2e) test coverage is required to ensure stability across large applications.

### Assigning Controllers

  - When a controller must be paired with a view and either component may be re-used by other controllers or views, define controllers along with their routes. 
    
    Note: If a View is loaded via another means besides a route, then use the `ng-controller="Avengers as vm"` syntax. 

    *Why?*: Pairing the controller in the route allows different routes to invoke different pairs of controllers and views. When controllers are assigned in the view using [`ng-controller`](https://docs.angularjs.org/api/ng/directive/ngController), that view is always associated with the same controller.

 ```coffeescript
  # avoid - when using with a route and dynamic pairing is desired

  # route-config.js
  angular
    .module('app')
    .config(config);
  
  config = ($routeProvider)->
    $routeProvider
      .when('/avengers',
        templateUrl: 'avengers.html'
      )
  ```

  ```html
  <!-- avengers.html -->
  <div ng-controller="Avengers as vm">
  </div>
  ```

  ```coffeescript
  # recommended

  # route-config.js
  angular
    .module('app')
    .config(config);

  config = ($routeProvider)->
    $routeProvider
      .when('/avengers',
        templateUrl: 'avengers.html',
        controller: 'Avengers',
        controllerAs: 'avengers'
      )
  ```

  ```html
  <!-- avengers.html -->
  <div>
  </div>
  ```
  
## Services and Factories

### Accessible Members Up Top

  - Expose the callable members of the service (it's interface) at the top, using a technique derived from the [Revealing Module Pattern](http://addyosmani.com/resources/essentialjsdesignpatterns/book/#revealingmodulepatternjavascript). 

    *Why?*: Placing the callable members at the top makes it easy to read and helps you instantly identify which members of the service can be called and must be unit tested (and/or mocked). 

    *Why?*: This is especially helpful when the file gets longer as it helps avoid the need to scroll to see what is exposed.

    *Why?*: Setting functions as you go can be easy, but when those functions are more than 1 line of code they can reduce the readability and cause more scrolling. Defining the callable interface via the returned service moves the implementation details down, keeps the callable interface up top, and makes it easier to read.

  ```coffeescript
  # avoid
  dataService = ->
    someValue = ''

    save = ->
      # implementation details

    validate = ->
      # implementation details
      
    service = 
      save: save
      someValue: someValue
      validate: validate
  
    service
  ```

  ```coffeescript
  # recommended
  dataService = ->
    someValue = ''
    
    service = 
      save: save
      someValue: someValue
      validate: validate
  
    return service
    
    save = ->
      # implementation details

    validate = ->
      # implementation details
  ```

## Directives
### Limit 1 Per File

  - Create one directive per file. Name the file for the directive. 

    *Why?*: It is easy to mash all the directives in one file, but difficult to then break those out so some are shared across apps, some across modules, some just for one module. 

    *Why?*: One directive per file is easy to maintain.

  ```coffeescript
  # avoid
  # directives.js.coffee

  angular
    .module('app.widgets')

    # order directive that is specific to the order module
    .directive('orderCalendarRange', orderCalendarRange)

    # sales directive that can be used anywhere across the sales app
    .directive('salesCustomerInfo', salesCustomerInfo)

    # spinner directive that can be used anywhere across apps
    .directive('sharedSpinner', sharedSpinner)

  orderCalendarRange = ->
    # implementation details
  
  salesCustomerInfo = ->
    # implementation details
    
  sharedSpinner = ->
    # implementation details
  ```

  ```coffeescript
  # recommended
  # calendarRange.directive.js.coffee

   # @desc order directive that is specific to the order module at a company named Acme
   # @example <div acme-order-calendar-range></div>
  
  angular
    .module('sales.order')
    .directive('acmeOrderCalendarRange', orderCalendarRange)
  
  orderCalendarRange = ->
    # implementation details
  ```

  ```coffeescript
  # recommended
  # customerInfo.directive.js

  # @desc spinner directive that can be used anywhere across the sales app at a company named Acme
  # @example <div acme-sales-customer-info></div>
      
  angular
    .module('sales.widgets')
    .directive('acmeSalesCustomerInfo', salesCustomerInfo)

  salesCustomerInfo = ->
    # implementation details
  ```

  ```coffeescript
  # recommended
  # spinner.directive.js

  # @desc spinner directive that can be used anywhere across apps at a company named Acme
  # @example <div acme-shared-spinner></div>
  
  angular
    .module('shared.widgets')
    .directive('acmeSharedSpinner', sharedSpinner)
    
  sharedSpinner = ->
    # implementation details
  ```

    Note: There are many naming options for directives, especially since they can be used in narrow or wide scopes. Choose one that makes the directive and it's file name distinct and clear. Some examples are below, but see the naming section for more recommendations.

### Manipulate DOM in a Directive

  - When manipulating the DOM directly, use a directive. If alternative ways can be used such as using CSS to set styles or the [animation services](https://docs.angularjs.org/api/ngAnimate), Angular templating, [`ngShow`](https://docs.angularjs.org/api/ng/directive/ngShow) or [`ngHide`](https://docs.angularjs.org/api/ng/directive/ngHide), then use those instead. For example, if the directive simply hides and shows, use ngHide/ngShow. 

    *Why?*: DOM manipulation can be difficult to test, debug, and there are often better ways (e.g. CSS, animations, templates)

### Provide a Unique Directive Prefix

  - Provide a short, unique and descriptive directive prefix such as `acmeSalesCustomerInfo` which is declared in HTML as `acme-sales-customer-info`.

    *Why?*: The unique short prefix identifies the directive's context and origin. For example a prefix of `cc-` may indicate that the directive is part of a CodeCamper app while `acme-` may indicate a directive for the Acme company. 

    Note: Avoid `ng-` as these are reserved for AngularJS directives.Research widely used directives to avoid naming conflicts, such as `ion-` for the [Ionic Framework](http://ionicframework.com/). 

### Restrict to Elements and Attributes

  - When creating a directive that makes sense as a stand-alone element, allow restrict `E` (custom element) and optionally restrict `A` (custom attribute). Generally, if it could be its own control, `E` is appropriate. General guideline is allow `EA` but lean towards implementing as an element when its stand-alone and as an attribute when it enhances its existing DOM element.

    *Why?*: It makes sense.

    *Why?*: While we can allow the directive to be used as a class, if the directive is truly acting as an element it makes more sense as an element or at least as an attribute.

    Note: EA is the default for AngularJS 1.3 +

  ```html
  <!-- avoid -->
  <div class="my-calendar-range"></div>
  ```

  ```coffeescript
  # avoid
  angular
      .module('app.widgets')
      .directive('myCalendarRange', myCalendarRange);
  
  myCalendarRange = ->
    directive = 
      link: link
      templateUrl: '/template/is/located/here.html'
      restrict: 'C'
      
    return directive
    
    link = (scope, element, attrs)->
      # implementation details
  ```

  ```html
  <!-- recommended -->
  <my-calendar-range></my-calendar-range>
  <div my-calendar-range></div>
  ```
  
  ```coffeescript
  /* recommended */
  angular
    .module('app.widgets')
    .directive('myCalendarRange', myCalendarRange);

  myCalendarRange = ->
    directive = 
      link: link
      templateUrl: '/template/is/located/here.html'
      restrict: 'EA'
      
    return directive
    
    link = (scope, element, attrs)->
      # implementation details
  ```

### Directives and ControllerAs

  - Use `controller as` syntax with a directive to be consistent with using `controller as` with view and controller pairings.

    *Why?*: It makes sense and it's not difficult.

    Note: The directive below demonstrates some of the ways you can use scope inside of link and directive controllers, using controllerAs. I in-lined the template just to keep it all in one place. 

    Note: Regarding dependency injection, see [Manually Identify Dependencies](#manual-annotating-for-dependency-injection).

    Note: Note that the directive's controller is outside the directive's closure. This style eliminates issues where the injection gets created as unreachable code after a `return`.

  ```html
  <div my-example max="77"></div>
  ```

  ```coffeescript
  angular
    .module('app')
    .directive('myExample', myExample);
  
  myExample = ->
    directive =
      restrict: 'EA'
      templateUrl: 'app/feature/example.directive.html'
      scope:
        max: '='
      link: linkFunc
      controller: ExampleController
      controllerAs: exp
      
    return directive
    
    linkFunc = (scope, el, attr, ctrl)->
      console.log('LINK: scope.max = %i', scope.max)
      console.log('LINK: scope.exp.min = %i', scope.exp.min)
      console.log('LINK: scope.exp.max = %i', scope.exp.max)

  ExampleController.$inject = ['$scope'];

  ExampleController = ($scope)->
    # Injecting $scope just for comparison
    @min = 3
    @max = $scope.max
    console.log('CTRL: $scope.max = %i', $scope.max)
    console.log('CTRL: @min = %i', @min)
    console.log('CTRL: @max = %i', @max)
  ```

  ```html
  /* example.directive.html */
  <div>hello world</div>
  <div>max={{exp.max}}<input ng-model="exp.max"/></div>
  <div>min={{exp.min}}<input ng-model="exp.min"/></div>
  ```

**[Back to top](#table-of-contents)**

## Manual Annotating for Dependency Injection

### Manually Identify Dependencies

  - Use `$inject` to manually identify your dependencies for AngularJS components.
  
    *Why?*: This technique mirrors the technique used by [`ng-annotate`](https://github.com/olov/ng-annotate), which I recommend for automating the creation of minification safe dependencies. If `ng-annotate` detects injection has already been made, it will not duplicate it.

    *Why?*: This safeguards your dependencies from being vulnerable to minification issues when parameters may be mangled. For example, `common` and `dataservice` may become `a` or `b` and not be found by AngularJS.

    *Why?*: Avoid creating in-line dependencies as long lists can be difficult to read in the array. Also it can be confusing that the array is a series of strings while the last item is the component's function. 

    ```coffeescript
    # avoid
    angular
      .module('app')
      .controller('Dashboard', 
          ['$location', '$routeParams', 'common', 'dataservice', ($location, $routeParams, common, dataservice)->
            # implementation details
          ])      
    ```

    ```coffeescript
    # avoid
    angular
      .module('app')
      .controller('Dashboard', 
         ['$location', '$routeParams', 'common', 'dataservice', Dashboard])
      
    Dashboard = ($location, $routeParams, common, dataservice)->
      # implementation details
    ```

    ```coffeescript
    # recommended
    angular
      .module('app')
      .controller('Dashboard', Dashboard);

    Dashboard.$inject = ['$location', '$routeParams', 'common', 'dataservice'];
     
    Dashboard = ($location, $routeParams, common, dataservice)-> 
      # implementation details
    ```

    Note: When your function is below a return statement the $inject may be unreachable (this may happen in a directive). You can solve this by either moving the $inject above the return statement or by using the alternate array injection syntax. 

    Note: [`ng-annotate 0.10.0`](https://github.com/olov/ng-annotate) introduced a feature where it moves the `$inject` to where it is reachable.

    ```coffeescript
    # inside a directive definition
    outer = ->
      return { controller: DashboardPanel }
      
      DashboardPanel.$inject = ['logger'] #Unreachable
      
      DashboardPanel = (logger)->
        # implementation details
    ```

    ```coffeescript
    # inside a directive definition
    outer = ->
      DashboardPanel.$inject = ['logger'] #Unreachable
      
      return { controller: DashboardPanel }
      
      DashboardPanel = (logger)->
        # implementation details
    ```

## Naming

### Naming Guidelines

  - Use consistent names for all components following a pattern that describes the component's feature then (optionally) its type. Our recommended pattern is `feature_type.js.coffee`. There are 2 names for most assets:
    *   the file name (`avengers_controller.js.coffee`)
    *   the registered component name with Angular (`AvengersController`)
 
    *Why?*: Naming conventions help provide a consistent way to find content at a glance. Consistency within the project is vital. Consistency with a team is important. Consistency across a company provides tremendous efficiency.

    *Why?*: The naming conventions should simply help you find your code faster and make it easier to understand. 

### Feature File Names

  - Use consistent names for all components following a pattern that describes the component's feature then (optionally) its type. My recommended pattern is `feature.type.js`.

    *Why?*: Provides a consistent way to quickly identify components.

    *Why?*: Provides pattern matching for any automated tasks.

    ```coffeescript
    # common options 

    # Controllers
    avengers.js
    avengers.controller.js.coffee
    avengersController.js.coffee

    # Services/Factories
    logger.js.coffee
    logger.service.js.coffee
    loggerService.js.coffee
    ```
    
    ```coffeescript
    # recommended
    
    # Controllers
    avengers_controller.js.coffee
    avengers_controller_spec.js.coffee

    # services/factories
    logger_service.js.coffee
    logger_service_spec.js.coffee

    # constants
    constants.js.coffee
    
    # module definition
    avengers_module.js.coffee

    # routes
    avengers_routes.js.coffee
    avengers_routes_spec.js.coffee

    # configuration
    avengers_config.js.coffee
    
    # directives
    avenger_profile_directive.js.coffee
    avenger_profile_directive_spec.js.coffee
    ```
    
### Test File Names

  - Name test specifications similar to the component they test with a suffix of `spec`.  

    *Why?*: Provides a consistent way to quickly identify components.

    *Why?*: Provides pattern matching for [karma](http://karma-runner.github.io/) or other test runners.

    ```coffeescript
    # recommended
    
    avengers_controller_spec.js.coffee
    logger_service_spec.js.coffee
    avengers_routes_spec.js.coffee
    avenger_profile_directive_spec.js.coffee
    ```

## Application Structure

### Overall Guidelines

### Layout

  - Place components that define the overall layout of the application in a folder named `layout`. These may include a shell view and controller may act as the container for the app, navigation, menus, content areas, and other regions. 

    *Why?*: Organizes all layout in a single place re-used throughout the application.

### Folders-by-Feature Structure

  - Create folders named for the feature they represent. Create sub folders of its type such as controllers, templates, directives, factories, services, etc...

    *Why?*: A developer can locate the code, identify what each file represents at a glance, the structure is flat as can be, and there is no repetitive nor redundant names. 

  ```coffeescript
  # recommended
    
  # app/javascripts/
  
  people/
    app.js.coffee # where module is declared
    directives/    
      calendar_directive.js.coffee  
      calendar_directive.html  
      user_profile_directive.js.coffee
      user_profile_directive.html  
    layout/
      nav.html
    controllers/
      people_controller.js.coffee
    templates/
      people.html
    factories/
      people_factories.js.coffee
    config/
      constants.js.coffee
      routes.js.coffee
    ```

## Angular $ Wrapper Services

### $document and $window

  - Use [`$document`](https://docs.angularjs.org/api/ng/service/$document) and [`$window`](https://docs.angularjs.org/api/ng/service/$window) instead of `document` and `window`.

    *Why?*: These services are wrapped by Angular and more easily testable than using document and window in tests. This helps you avoid having to mock document and window yourself.

### $timeout and $interval

  - Use [`$timeout`](https://docs.angularjs.org/api/ng/service/$timeout) and [`$interval`](https://docs.angularjs.org/api/ng/service/$interval) instead of `setTimeout` and `setInterval` .

    *Why?*: These services are wrapped by Angular and more easily testable and handle AngularJS's digest cycle thus keeping data binding in sync.

## Reference links:
  - https://github.com/johnpapa/angularjs-styleguide
  - https://github.com/toddmotto/angularjs-styleguide
  - http://toddmotto.com/digging-into-angulars-controller-as-syntax/
  - https://github.com/mgechev/angularjs-style-guide
  - http://google-styleguide.googlecode.com/svn/trunk/angularjs-google-style.html#controllers
