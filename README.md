javascript-module-flavors
=========================

Taste the javascript modules rainbow.


## Why modules
If you are already sold on the idea of modules, please |skip ahead|.

1. Separation of Concerns
  * Modules perform logically discrete functions. Modules typically follow the single responsibility principal and only expose a functionally similar interface.

2. Explicit Dependencies
  * A module system should allow for explicit dependency declaration. Implicit dependencies are hard to maintain and should be avoided.

3. Maintainability
  * Each module works independently from another module. Keeping functionality contained allows a developer to reason about the program one piece at a time, making updates much simpler. 

4. Portability
  * Keeping functionality contained and listing dependencies explicitly allows modules to be reused from project to project seamlessly.

Of course, you can probably think of many more reasons for why modules are useful. The four listed above are pretty general and apply to most programming languages. Considering javascript in the browser, another important use of modules is global management. Now that you're sold on the advantages of using module, lets proceed.

## Module formats
While there are many module formats a javascript developer can choose from, I'm going to cover the four most popular today. Each format is distinct in usage, benefits, and flaws.

### Revealing module pattern
Of the modules formats covered here, the revealing module pattern is the most basic. Originally design to handle global pollution, this pattern also provides light organization in small projects. To fully appreciate this pattern you should educate yourself on why global pollution is bad.

##### example 1.1
```javascript
var module  = (function () {

    return {};

}());
```

Example 1.1 shows the basic structure of the revealing module pattern. Lets take it bit by bit.

1. `var module =`
  * declare the variable name of the module, the modules entry point.
2. `function () {}()`
  * call the function immediately. (IIFE or immediately invoked function expression)
3. `(function () {} ());`
  * wrapped the function in parenthesis since `function () {}()` is a SyntaxError.
4. `return {};`
  * the return value of the IIFE becomes the value of the module.

##### example 1.2
```javascript
var monthFromNum = (function () {
    
    var months = [
        'Jan', 'Feb', 'Mar', 'Apr',
        'May', 'Jun', 'Jul', 'Aug',
        'Sep', 'Oct', 'Nov', 'Dec'
    ];

    return function (ind) {
        var month = months[ind];

        return (month ? month : false);
    };

}());
```

In the wild you will typically see modules like this defined on a global object, or on an ancestor of a global object. This is called name-spacing.

##### example 1.3
```javascript
// /js/bootup.js
var _CC = { // cool company name-space

    modules: {} // modules name-space

};

// /js/DOM.js
_CC.DOM = (function () {
    
    var _slice = Array.prototype.slice,

        q = function (selector) {
            var collection = document
                .querySelectorAll(selector);

            return _slice.call(collection);
        };

    return {
        q: q
    };

}());

// /js/modules/hide.js
_CC.modules.hide = (function () {
   
    var hide = function (elem) {
            elem.classList.add('is-hidden');
        },

        all = function (selector) {
            var elems = _CC.DOM.q(selector);
            
            elems.forEach(function (elem) {
                hide(elem);
            });
        };

    return {
        all: all
    };

}());

// usage
_CC.modules.hide.all('button');
```
* Pros
  * Very simple
  * Provides the ability to keep certain information private
  * Provides light organization to your javascript

* Cons
  * No conventional way to list dependencies
  * Some global variables are mandatory
  * Portability is difficult due to implicit dependencies
  * Must do dependency management by hand in an external place

### Asynchronous module definition
AMD from Wikipedia:
> Asynchronous module definition (AMD) is a JavaScript API for defining modules such that the module and its dependencies can be asynchronously loaded. It is useful in improving the performance of websites by bypassing synchronous loading of modules along with the rest of the site content. <br />
In addition to loading multiple JavaScript files at runtime, AMD can be used during development to keep JavaScript files encapsulated in many different files. This is similar to other programming languages, like Java, which support keywords such as import, package, and class for this purpose. It is then possible to concatenate and minify all the source JavaScript into one small file used for production deployment.

##### example 2.1
```javascript
// dep.js
define([], function () {
  
  return {};

});

// app.js
require(['dep'], function (dep) {
  // use dep
});
```
Example 2.1 demonstrates the basic, and most common usage of AMD, following require.js syntax. The full api can be found [here](http://requirejs.org/). Lets break it down:

* `define()`
  1. `define([], function () {`
    * use the `define()` function to define a new module called 'dep'. Specify the modules that need to be loaded before this module in the first parameter, which is optional. Define the modules factory function in the second parameter.
  2. `return {};`
    * return an object literal to be the value of 'dep'
  3. `});`
    * close the function
* `require()`
  1. `require(['dep'], function (dep) {`
    * Make sure 'dep' is loaded and ran before this module. Place the return value of 'dep' in as a parameter to this modules factory function.
  2. `// use dep`
    * you are now guaranteed 'dep' is loaded and available.
  3. `});`
    * close the function

##### example 2.2
```javascript
// month.js
define(function () {
    var months = [
            'Jan', 'Feb', 'Mar', 'Apr',
            'May', 'Jun', 'Jul', 'Aug',
            'Sep', 'Oct', 'Nov', 'Dec'
        ],

        fromNum = function (ind) {
            var month = months[ind];

            return (month ? month : false);
        };

    return {
        fromNum: fromNum
    };
});

// tellUserMonth.js
require(['month'], function (month) {
  var date = new Date(Date.now()),
      month = month.fromNum(date.getMonth());

  alert("Hey, it's " + month);
});
```

* Pros
  * Dependency management is handle for you
  * At a glance you can see the dependencies this file requires
  * Global pollution is kept to a minimum

* Cons
  * Although we didn't get into this, setting up require.js can be pretty painful. Just take a look at the [configuration docs](http://requirejs.org/docs/api.html#config)
  * One of the main purposes of require.js is to load files asynchronously. However, once in production it is rightfully recommended to bundle these files into a single package. This adds unnecessary complexity to the project
  * Too much boilerplate code for every single module
  * Doesn't provide a convenient way to download required dependencies.


### CommonJS modules

### ES6 modules
