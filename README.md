# Angular Tips and Tricks
https://www.lynda.com/Angular-tutorials/One-time-binding/597030/611899-4.html

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