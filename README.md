# Angular Tips and Tricks
https://www.lynda.com/Angular-tutorials/One-time-binding/597030/611899-4.html

### Notes are on the following:
* ng-show vs ng-if
* ng-repeat and adding "track by"
* watchers and ``{{ ::item.name }}`` < one-time bindings
* ng-include vs component
* $scope.watch vs ng-change
* Ampersand ( ``` '&' ``` ) Binding
    * Invoking Binding with Argument
* Understanding @ binding
* Services vs Factories
* Angular: Special Functions
    * angular.equals() and _.isEqual() ?
    * angular.toJson() and JSON.stringify() ?
    * angular.fromJson() and JSON.parse()
* Create promises without $q.defer()
    * Promise Chaining
* Circular Dependencies
    * Interceptor -> AuthService -> $http -> Intercepter
    * $injector service to the rescue!
* Transclusion
    * Multiple Transclusion
* Turn URLs in paragraphs to safe links
    * linky: The Filter

### ng-show & ng-if 
**(test to make sure which one is faster)**
* ``ng-show`` loads to the DOM first but then attaches watcher to the element (which can slow down the application)
* ``ng-if`` removes the DOM element and regenerates it if true (this can get expensive if it takes too much time to load)

## ng-repeat
* In Angular 1.x you write 
    * ``ng-repeat=“task in $ctrl.tasks”``
    * might be expensive (loading longer times if the list or long or each element template is complex
    * attached ``$$haskKey`` property
* In Angular 2+ you can now write
    * ``<div ng-repeat=“task in $ctrl.tasks track by task.id”>``
    * Used instead of ``$$haskKey``
    * This only loads specific items and not the entire list of items

### Too many watchers slow down apps
* Biggest source is template bindings ```{{ $ctrl.value }}```
* To minimize the constant watchers on the app you can use
    * one-time bindings (Angular 1.3)
    * Allow indicating which watchers should only be evaluated once
    * If the items details aren’t important to be updated all the time this item is consider static. 
* CHANGE
    * ``{{ item.name }}`` < not static (costly)
    * ``{{ ::item.name }}`` < one-time bindings

## ng-include vs component
* ``ng-include`` 50-60% slower than component (just loading time) 
    * has a significant performance hit on apps
    * performs slower than completed components, scopes, etc
    * because ng-include becomes part of the Angular Digest Loop - it can be a 100% increase 
* **component** 50-60% faster than component
    * Even though components create new scopes, and introduce more watchers, they are significantly faster

## Perform Action on User Typing

### $scope.watch vs ng-change 
**(try to use ``ng-change`` in most cases)**
```
<input type=“text” ng-model=“$ctrl.show.name”>

	$scope.watch ( 
		function () { return self.show.name; }, 
		nameChanged);
		
```

```
<input type=“text” ng-model=“$ctrl.show.name”
	 ng-change=“$ctrl.nameChanged()”>
```

* simple directive that operates a lot like jquery
* only triggers for changes made by the user
* expresses intent more clearly
* easier to trace
* more performant; saves on watches since it uses one less watch expression. Since Angular knows it should only call the expression in change events, it doesn’t need to keep evaluating on every digest cycle, as opposed to a watch. 




### Ampersand ( ``` '&' ``` ) Binding
* typically used to bind functions to a component
* Not simple two-way or one-way bindings
* Can be an expression, not just function call
* Angular Saves expression for later
* (directive)

```
angular.module('app').component('toggle', {
    templateUrl: '...',
    bindings: {
        callback: '&'
    },
    controller: function() {
        // ...
        this.callback();
    }
});
```

```
<toggle callback="$ctrl.myCallback()"></toggle>

```

* often time people get confused when they have to pass a parameter to the function. Simply trying to invoke it as a regular function doesn’t work, since Angular reads everything as strings / objects first. Remember you are not really calling a function; you are evaluating an expression and supplying it with values

```
<toggle callback="$ctrl.myCallback( value )"></toggle>
```
#### Invoking Binding with Argument

```
this.callback({value: 'toggle changed'});
```


### Understanding @ binding
* ```@``` bindings can be used when the input is a string, especially when the value of the binding doesn't change.

```
angular.module('app').component('passwordBox', {
    template: 
        '<input type=password ng-model="$ctrl.ngModel"
          placeholder="{{ $ctrl.placeholder }}">',
    bindings: { 
        ngModel: '=',
        placeholder: '@'
    }
}
```

```
<password-box ng-model="$ctrl.password"
    placeholder="'Enter password'"> //no extra quotes
</password-box>
```

**Example: Dynamic Usage** 
```
<password-box ng-model="$ctrl.password"
    placeholder="{{ $ctrl.user }}'s password"> //interpolation
</password-box>

```

## Services vs Factories
* Similar use cases
* Both create an injectable
* Both are invoked once to create a **singleton**
* It doesn’t matter whether SomeService is actually a service or a factory. The MyCtrl that’s using it is unaware of it. 

#### **Factories** Essentially, factories are functions that return an object

below is a matching factory to implement this… The definition is passing a function whose return value is used as the instance that will be injected. This will result in an injectable object called SomeService with a single public function called someFunction

```
angular.module(‘app’).factory(‘SomeService’, function() {
	return {
		someFunction: function() {}
	};
});
```

#### **Services** Are constructor functions of the object
* And here is a matching service below. 
* The difference, as you can see, is that the service function will be invoked once by Angular using the new keyword. This will result in the exact same injectable as below. 
* As you can see, these are truly equivalent, and the decision of which should be used comes down to a matter of taste. 
```
angular.module(‘app’).service(‘SomeService’, function() {
	this.someFunction=function() {};
});
```

### What to use? Factory vs Services
* Matter of taste
* Choose one and Stick to that choice
* ES5 should use factories
* ES6 should use services




## Angular: Special Functions

### angular.equals() and _.isEqual() ?
* Angular implemented angular.equalsso that it ignores properties that start with the $ dollar sign.

```
angular.equals( { a: 1 }, { a: 1 } === true
angular.equals( { a: 1 }, { a: 1, $a: 2 } === true

angular.equals( { a: 1 }, { a: 1, b: function() {} }) === true
```


### angular.toJson() and JSON.stringify() ?
### angular.fromJson() and JSON.parse()

* Angular prefixes its own APIs with “$”
    * Examples
        * `` ng-resource: `$save` ``
        * `` $scope: `$apply` ``
* ``angular.toJson()``
    * Ignores properties that start with “$$”
    * “$$” means private and internal Angular API (such as $$hashKey in ng-repeat elements)
    * toJson() is safer in Angular apps
    * Used automatically by $http
    * ``angular.fromJson()`` equivalent to ``JSON.parse()``
    
    

## Create promises without $q.defer()
**Almost never needed**

**BAD**
```
var defer;
defer = $q.defer();
defer.resolve( [ ‘detail’, ‘simple’ ] );
return defer.promise;
```
**GOOD**
```
return $q.when( [ ‘detail’, ‘simple’ ] );

return $q.reject( { error: ‘something bad’ } );
```
**BAD**
```
var defer = $q.defer();
$http.get( ‘options.json ’).then( function (response) {
	defer.resolve(response.data);
});
return defer.promise;
```
**GOOD**
```
return $http.get( ‘options.json’ ).then( function(response) {
	return response.data;
});
```
### Promise Chaining
```
return promiseA().then( function(a) {
	return promiseB(a);
}).then( function(b) {
	return promiseC(b);
}).then( function(c) {
	return promiseD(c);
});

```
**Useless Wrapping: BAD**
```
var defer = $q.defer();
$timeout(function() {
	defer.resolve();
}, 5000);
return defer,promise
```
### So, When is $q.defer() GOOD?
* When integrating with non-angular libraries that use callbacks
* So, when should you use defer? Basically the main reason is: You’re converting a callback API to be a promise-based. This usually means you’re wrapping some front-party library, like a jQuery plugin, to work with Angular. Callbacks are not as fun as promises, and break the API Angular expects to work with. To kickstart the promise chain and covert callbacks you have the create the first promise yourself.
* This is exactly what defer should be used for. In most other scenarios, leave it be. 

**GOOD**
```
var deferred = $q.defer();
jQueryPlugin.initialize( function callback( result ) {
	defered.resolve( result );
});
return deferred.promise;
```


## Circular Dependencies
A -> B -> C -> A
* Causes Angular’s bootstrap to error
* Usually are a code smell
    * some examples of code smells: http://www.codelord.net/2015/09/30/angular-2-preparation-controller-code-smells/
* May be solved by splitting services smaller

### HTTP Interceptors
* Angular’s way for handling cross-app HTTP concerns
* Most commonly used for authentication and error handling
* Example: Interceptor
    * Listens for HTTP failures
    * In case of 401 failures, calls AuthService
    * AuthService performs logic request

**Example: Interceptor**
```
$httpProvider.interceptors.push(function(AuthService) {
	return {
		response: function( response ) {
			//Detect and handle 401 errors
			AuthService.handleExpiredSession();
		}
	};
});
```
**Example: AuthService**
```
function AuthService( $http ) {
	return {
		login: function( user, password ) {
			// This uses $http to login
		},
		handleExpiredSession: function() {
			// Redirect to login page
		}
	}
});
```
**Initialization Failure because The Cycle (Circular Dependancies)**

**Interceptor -> AuthService -> $http -> Intercepter**
```
Error: [ $injector:cep ] Circular dependency found:
$http <- AuthService <- $http
```
### $injector service to the rescue! ###
* The injector is the programmatic way to access Angular’s dependency injection at runtime
* Using it, it is possible to manually inject AuthService inside the interceptor and break the circular dependency. 
* Calling $injector.get() with the name of a service will return the exact same singleton instance of AuthService as you’d get when injecting it to a controller’s constructor. The main difference is that now we are performing this at a later point, after Angular has finished bootstrapping the project. At this point in time, where everything is up and running, it’s safe to inject AuthService
```
$httpProvider.interceptors.push( function ($injector) {
	return {
		response: function ( response ) {
			// Detect and handle 401 errors
			$injector.get( ‘AuthService’ ).hadleExpiredSession();
		}
	};
});
```


## Transclusion
* Create components with customizable contents
* The magic behind ``ng-repeat``
* Examples of Transclusion Usages:
    * News Feed
    * Tabs
    * Modal

### Multiple Transclusion
*Prior to **Angular 1.5**, a component could only transcluyde a single entry. Whatever you gave it had to be used as a whole, but in **1.5** Angular introduced **multiple slot transclusion**.*
* Introduced in Angular 1.5
* Allows specifies multiple slots in the same component

Example: Generic Modal Component
* Allows customizing both header and body
* Usage:
```
<modal>
	<modal-title>Reset {{ $ctrl.name }}?</modal-title> // <- <modal-title> transclusion
	<modal-body>  // <- <modal-body> transclusion
		You can only do this {{ $ctrl.times }} times
	</modal-body>
</modal>
```
* The modal title and modal body elements are the transclusion slots
* The modal title and modal body elements supply the transcultion slots
* Their content will be injected
```
app.component( ‘modal’, {
	template: [
		‘<dv ng-transclude=“title”></div>,
		‘<div ng-transclude=“body”></div>
	].join(‘’),
	transclude: {
		title: ‘modalTitle’,
		body: ‘modalBody’
	},
	controller: function() {
		// Stuff to make this component render as a modal
	}
});
```
* Fortunately enough, transclusion automatically takes care of things like passing the scopes correctly so that even though the templates pass reference the original component scope, for example the control property, it’s still properly visible inside.


## Turn URLs in paragraphs to safe links
**Automatically Detecting Links**

###linky: The Filter
* Built-in solution by Angular
* Safe
* Introduced way back in 1.0
**linky: Setting Up**
* Add angular-sanitize.js to the project
* Require the `ngSanitize` module
```
angular.module( ‘myApp’, [‘ngSanitize’] );
```
```
<div ng-bind-htlm=“blog.post | linky”></div>
```
* Basic usage: make sure to pass the text to the links filter, and then use that with a special ng-bind-html direct instead of the regular ng-bind directive. This one allows the input to actually contain some HTML. This will take **blog.post** and display it regularly except that URLs inside it would now become real clickable links.
* What happens is that Angular safely sanitizes all the text. The filter also supports some more advanced features. It’s possible to set the target of the links, for example to make sure they open in a new tab. 
```
<div ng-bind-html=“blog.post | link:’_blank’”></div>
```
* And even more, one can add specific attributes to links such as adding the ``{ real: ‘nofollow’ }`` attribute which is usually recommended when putting up links to user-generated content in order to let search engine bots know that the link pages should not be considered.
    * To do that, simply pass another parameter to the filter with the attributes object. And that’s it! Link away.
```
<div ng-bind-html=“blog.post | link:’_blank’:{ rel: ‘nofollow’ }”></div>
```
