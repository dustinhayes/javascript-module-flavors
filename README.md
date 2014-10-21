javascript-module-flavors
=========================

Taste the javascript modules rainbow.

* [Why modules](#why-modules)
* [Module formats](#module-formats)
 * [Revealing module pattern](#revealing-module-pattern)
 * [Asynchronous module definition](#asynchronous-module-definition)
 * [CommonJS modules](#commonjs-modules)

## Why modules
If you are already sold on the idea of modules, please [skip ahead](#module-formats).

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
While there are many module formats a javascript developer can choose from, I'm going to cover the three most popular today. Each format is distinct in usage, benefits, and flaws.

### Revealing module pattern
Of the modules formats covered here, the revealing module pattern is the most basic. Originally designed to handle global pollution, this pattern also provides light organization in small projects. To fully appreciate this pattern you should educate yourself on why global pollution is bad. (looking at you third-party ads)

##### example 1.1
```javascript
var myUniqueModuleName = (function () {

    // private logic

    // public API
    return {};

}());
```

Example 1.1 shows the basic structure of the revealing module pattern. Lets take it bit by bit.

1. Declare a variable with a unique name.
2. Assign that variable to an immediately invoked function expression ([iife](http://benalman.com/news/2010/11/immediately-invoked-function-expression/)).
3. Perform some private logic
4. Return a public API to the unique variable.

Let's look at a slightly more complex example:

##### example 1.2
```javascript
var monthsUtil = (function () {
    
    var months = [
            'Jan', 'Feb', 'Mar', 'Apr',
            'May', 'Jun', 'Jul', 'Aug',
            'Sep', 'Oct', 'Nov', 'Dec'
        ],

        fromNum = function fromNum(ind) {
            var month = months[ind];

            return (month ?
                month : false
            );
        },

        getCurrent = function getCurrent() {
            var date = new Date();

            return fromNum(
                date.getMonth()
            );
        };

    return {
        fromNum: fromNum,
        getCurrent: getCurrent
    };

}());

// usage
(function () {
    var currentMonth = monthsUtil.getCurrent();

    alert("Hey {{user}}, it's " + currentMonth);
}());
```

In the wild you will typically see modules like this defined as a property of a global object or an ancestor of a global object. This is called [name-spacing](http://www.2ality.com/2011/04/modules-and-namespaces-in-javascript.html).

##### example 1.3
```javascript
// /js/bootup.js
var _CC = { // cool company name-space

    modules: {} // modules name-space

};

// /js/DOM.js
_CC.DOM = (function () {
    
    var _slice = Array.prototype.slice,

        q = function query(selector) {
            var collection = document
                .querySelectorAll(selector);

            return _slice.call(collection);
        };

    return {
        q: q
    };

}());

// /js/DOM/hide.js
_CC.modules.hide = (function () {
   
    var hide = function hide(elem) {
            elem.classList.add('is-hidden');
        },

        all = function hideAll(selector) {
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
  * Provides light organization to your javascript modules

* Cons
  * No conventional way to list dependencies
  * Some global variables are mandatory
  * Portability is difficult due to implicit dependencies
  * Must do dependency management by hand in an external place

#### Additional Resources
* http://www.adequatelygood.com/JavaScript-Module-Pattern-In-Depth.html
* http://addyosmani.com/resources/essentialjsdesignpatterns/book/#modulepatternjavascript
* http://addyosmani.com/resources/essentialjsdesignpatterns/book/#revealingmodulepatternjavascript

### Asynchronous module definition
AMD from Wikipedia:
> Asynchronous module definition (AMD) is a JavaScript API for defining modules such that the module and its dependencies can be asynchronously loaded. It is useful in improving the performance of websites by bypassing synchronous loading of modules along with the rest of the site content. <br />
In addition to loading multiple JavaScript files at runtime, AMD can be used during development to keep JavaScript files encapsulated in many different files. This is similar to other programming languages, like Java, which support keywords such as import, package, and class for this purpose. It is then possible to concatenate and minify all the source JavaScript into one small file used for production deployment.

There is a lot that goes into making AMD work for production sites. I'll simply be focusing on the module syntax.

##### example 2.1
```javascript
// myUniqueModule.js
define(function () {
  
    return {};

});

// requireIt.js
require(['myUniqueModule'], function (myUniqueModule) {

    // use myUniqueModule

});
```
Example 2.1 demonstrates the basic and most common usage of AMD, following the require.js syntax. The full api can be found [here](http://requirejs.org/).

In `myUniqueModule.js` we define an anonymous module, which allows it's name to default to the name of the file (more portable). The AMD spec provides a `define()` function globally which takes at most three parameters, two of which are optional. In this case we passed a function. We could have also passed any valid javascript expression, including an object, array, or a string. When we pass a function `define()` uses the return value as modules public value. For example, we returned an object literal, so as it stands `myUniqueModule` is equal to an empty object literal. As I said `define()` can take at most three parameters. One of them is not important as its use is discouraged (module ids). The other is an array dependencies. This allow you to declaratively state, within the same file, which modules should run before this module. Very powerful. Finally, to use a module with out declaring a new one the AMD spec provides the `require()` function. This function expects an array of dependencies to run then maps their return values as parameters to the function that is passed as the second parameter.

Here is that same months module AMD style:

##### example 2.2
```javascript
// monthsUtil.js
define(function () {
    var months = [
            'Jan', 'Feb', 'Mar', 'Apr',
            'May', 'Jun', 'Jul', 'Aug',
            'Sep', 'Oct', 'Nov', 'Dec'
        ],

        fromNum = function fromNum(ind) {
            var month = months[ind];

            return (month ?
                month : false
            );
        },

        getCurrent = function getCurrent() {
            var date = new Date();

            return fromNum(
                date.getMonth()
            );
        };

    return {
        fromNum: fromNum,
        getCurrent: getCurrent
    };
});

// tellUserMonth.js
require(['monthsUtil'], function (monthsUtil) {
    var currentMonth = monthsUtil.getCurrent();

    alert("Hey {{user}}, it's " + currentMonth);
});
```

* Pros
  * Dependency management is handled for you (kind of)
  * At a glance you can see the dependencies this file requires
  * Global pollution is kept to a minimum

* Cons
  * Although we didn't get into this, setting up require.js can be pretty painful. Just take a look at the [configuration docs](http://requirejs.org/docs/api.html#config)
  * One of the main purposes of require.js is to load files asynchronously. However, once in production it is rightfully recommended to bundle these files into a single package. This adds unnecessary complexity to the project
  * Too much boilerplate code for every single module (naming modules twice. once for dep again for param)
  * Doesn't provide a convenient way to download required dependencies.

#### Additional Resources
* http://www.sitepoint.com/understanding-requirejs-for-effective-javascript-module-loading/
* http://tomdale.net/2012/01/amd-is-not-the-answer/
* http://code.tutsplus.com/tutorials/the-essentials-of-amd-and-requirejs--net-24790

### CommonJS modules
Okay, so technically AMD originated from the CommonJS group. I only name them this way because of the industry conventions. Anyway, back to the important stuff. 

CommonJS modules have two simple concepts: `require()` and `module.exports`. The require function accepts a module identifier, which is a string that describes how to find the module. In most implementations this is either a relative path (no extension), the name for a global module stored on the machine, or the name of a module provided by the host environment.

##### example 3.1
```javascript
// myUniqueModule.js
var myUniqueModule = {};

module.exports = myUniqueModule;

// requireIt.js
var myUniqueModule = require('./myUniqueModule');
```

In example 3.1 we see the basic usage of the CommonJS module structure. Export a module interface then require for use in another file. Lets see how we might write the name-space example with CommonJS:

##### example 3.2
```javascript
// js/DOM/q.js
var _slice = Array.prototype.slice,

    q = function query(selector) {
        var collection = document
            .querySelectorAll(selector);

        return _slice.call(collection);
    };

module.exports = q;

// js/DOM/hide.js
var q = require('./q'),
    
    hide = function hide(elem) {
        elem.classList.add('is-hidden');
    },

    all = function hideAll(selector) {
        var elems = q(selector);
        
        elems.forEach(function (elem) {
            hide(elem);
        });
    };

module.exports.all = all;

// js/modules/hideButtons.js
var hide = require('../DOM/hide');

hide.all('button');
```

In an environment like node, `require()` and `module.exports` is provided for you. It's how the module system works. The example above is written to run in a browser environment. How is this possible? Well, there a few tools. 

* http://browserify.org/
* http://webpack.github.io/

I'm going to briefly cover browserify. Although webpack seems very interesting. Specifically in terms of it's [loaders](http://webpack.github.io/docs/using-loaders.html) and it's concept of [multiple entry points](http://webpack.github.io/docs/multiple-entry-points.html).

So why browserify? What does it do for us? From the website:

> Browserify lets you require('modules') in the browser by bundling up all of your dependencies.

More specifically, browserify is a node module that allows you to write CommonJS style modules in the browser. It does this by crawling each JS file in your project, building a dependency graph, and spitting out a single file with everything in the order it is supposed to be in. (simple description)

Let's start with getting node and npm set up.

* Download [node](http://nodejs.org/)
* Type npm in the console to ensure it built with node

At this point you could install browserify globally, however let's go the more common route and set up browserify in a new project.

* Create a new project called test
* From the project root in the command-line run `npm init`
* Hit enter until you finish the package.json walk through
* From the project root in the command-line run `npm install browserify --save-dev`

With in package.json, npm allows you to run certain scripts via the scripts property. That's how we'll run browserify. Here is how the package.json file should look:

```json
{
    "name": "test",
    "version": "0.0.0",
    "description": "",
    "main": "index.js",
    "scripts": {
        "test": "echo \"Error: no test specified\" && exit 1"
    },
    "author": "",
    "license": "ISC"
}
```

Add a property to the scripts object called start:

```json
"scripts": {
    "start": "browserify index.js -o bundle.js",
    "test": "echo \"Error: no test specified\" && exit 1"
}
```

Create a file called index.js and add the following:

```javascript
var monthsUtil = require('./monthsUtil'),
    currentMonth = monthsUtil.getCurrent();

alert("Hey {{user}}, it's " + currentMonth);
```

Create a file called monthsUtil.js and add the following:

```javascript
var months = [
        'Jan', 'Feb', 'Mar', 'Apr',
        'May', 'Jun', 'Jul', 'Aug',
        'Sep', 'Oct', 'Nov', 'Dec'
    ];

    fromNum = function fromNum(ind) {
        var month = months[ind];

        return (month ?
            month : false
        );
    },

    getCurrent = function getCurrent() {
        var date = new Date();

        return fromNum(
            date.getMonth()
        );
    };

module.exports = {
    fromNum: fromNum,
    getCurrent: getCurrent
};
```

Create a index.html file with the following:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>test</title>
</head>
<body>
    
    <script src="bundle.js"></script>
</body>
</html>
```

Load it up in a browser and watch magic happen.

It can get pretty annoying to have to build the source each time you make a change. Thankfully we have the watchify module for that, which we can install from npm.

`npm install watchify --save-dev`

Now change the scripts start property to the following:

```json
"scripts": {
    "start": "watchify index.js -o bundle.js",
    "test": "echo \"Error: no test specified\" && exit 1"
}
```

Now we can change the file and it will build automatically. Nice.

* Pros
  * Explicit dependency declaration
  * Hands free dependency management
  * Portability, due to package management
  * Access to all of npm (huge win)

* Cons
  * Have to build in development (but you just saw how easy that can be)
  * Can be tough to convert an existing project

#### Additional Resources
* http://dailyjs.com/2010/10/18/modules/
* https://egghead.io/lessons/nodejs-what-are-commonjs-modules
* http://javascriptplayground.com/blog/2013/11/backbone-browserify/
