#Widget Development Methodology

###Introduction

The Widget methodology is an initiative for defining the optimal way of writing a Backbase powered widget. The methodology is a mix of code samples, guidelines and descriptions, aimed at helping people develop widgets in a way, that will ensure the long term value of them.   


There are several factors that were taken into account when creating this methodology:
+ Usage of Backbase technologies
+ Maintainable and re-usable code
+ Performance
+ Code uniformity
+ Frontend Best Practices

##Conventions

###File names

+ The widget folder should contains the name of the widget, hyphenated and prefixed as widget (eg. widget-transaction-activity)
+ Definition file should always be named index.html
+ Javascript file containing the widget prototype should have the name of the widget, CamelCase (eg. TransactionCtrl.js)
+ CSS file containing the widget styles should have the name of the widget, CamelCase (eg. TransactionActivity.css)
+ The Catalog thumbnail must be named icon.png and be placed in the media folder

###Folder structure

Inside the widget folder, there should be the following folder:
+ scripts/ for all the javascript files inside subfolders (eg. directives, factories)
+ styles/ for all the css files
+ media/ for all images and other static files (eg. pdfs or videos)
+ templates/ for all templates (eg. partials)

An example of the structure for the ‘Todo widget’, would be something like this: 

###Todo

>-index.html
>-bower.json
>-icon.png
>-model.xml
>-README.md
>> styles
>>> -base.less
>>>
>>
>> scripts    
>>> -main.js
>>> -controller.js
>>> -models.js
>>>
>>
>> media
>>> -media.png
>>>
>>
>> templates
>>> -template.mustache
>>> -template.html


###Configuration File

The widget must have a configuration file named model.xml and exist in the root folder of the widget.
 + <name> should be hyphenated (eg. transaction-activity)
 + mandatory properties: TemplateName, Title, thumbnailUrl, widgetChrome, src
 + Semantic tags should be used to characterize the widget


###Widget Model File

This is what the model of the todo widget should look like:

```
<catalog>
    <widget>
        <name>todo</name>
        <contextItemName>[BBHOST]</contextItemName>
        <properties>
            <property label="Title" name="title" viewHint="text-input,user">
                <value type="string">Todo App</value>
            </property>
            <property label="Icon" name="icon" viewHint="text-input,admin,designModeOnly">
                <value type="string"></value>
            </property>
            <property name="src">
                <value type="string">$(itemRoot)/index.html</value>
            </property>
            <property name="thumbnailUrl">
                <value type="string">$(itemRoot)/icon.png</value>
            </property>
        </properties>
        <tags>
            <tag>apps</tag>
            <tag>site-tools</tag>
        </tags>
    </widget>
</catalog>
```
 
##Widgets

There are some general guidelines you should keep in mind when developing for Backbase portal.
+ Widgets are autonomous applications. They should not be dependent on other widgets or have other widgets dependant on them.
+ Widgets should not contain page presentation logic. That should be implemented using containers.
+ Widgets should be rendered correctly (within reason) in any place where you drop it (whether it is a page or a container). Don't only develop them for fitting in the specific place a design is showing them, but try them in other places as well
+ Widgets must not depend on URL state and must not affect URL state (Navigation widgets are the only exception)

###Widget instances

When developing a widget, always keep in mind that it can be placed multiple times in the page. Using ids on elements, doesn't make them unique. As soon as a business user drops the same widget twice in the page, the id is duplicated. This is also why you should use selectors in Javascript starting from the widget body.

To make things even more complicated, keep in mind that you can have multiple widgets in the catalog using the same widget definition (but maybe different preferences). In the end, always develop knowing that markup you write in a widget can appear multiple times on the page.
 
###Markup

####Head Section

#####Guidelines

+ We are using the default HTML5 Doctype for widgets. However, all widgets should be XML-compliant.
+ Only include css files in the head. Javascript files should be loaded through Launchpad's requireJS configuration.
+ Define metadata, as required per project
+ Use the widget namespace


#### Code Sample

##### Head section of a widget

```HTML
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml" xmlns:g="http://www.backbase.com/2008/gadget" xml:lang="en">
	<head>
    	<meta http-equiv="Content-Type" content="text/xml; charset=UTF-8" />
    	<meta name="viewport" content="width=device-width, initial-scale=1" />
	    <link rel="stylesheet" href="styles/base.css" type="text/css" />
	</head>
</html>
```

####Body Section

#####Guidelines

+ Always put a class on your body like this: `<widget name>`. eg. todo
+ Use semantic markup. Favor native HTML elements instead of extra classes. eg. use `<h1>` instead of `<div class="main-header">`
+ Prefix all your classes that are widget-specific with your widget class. For example, for a sidebar use: todo-sidebar
+ Don't mix classes for styling and javascript hooks. Use the data attribute named data-js. eg. `<button data-js="submit-form"></button>`
+ Don't write inline javascript
+ Don't write inline CSS

####Code Sample

#####Body section of a widget

```HTML
<body g:onload="requireWidget(__WIDGET__, 'scripts/main')" class="todo">
    <div ng-controller="MainCtrl as mainCtrl">
       	<h1 class="todo-title">todos</h1>
       	<input data-js="new-todo" class="todo-field" placeholder="What needs to be done?" />
    </div>
</body>
```

###Templates

Templates can exist as part of your widget definition, or they can be separate files. In any case, use Mustache as your templating language.
 
###External Templates

They are separate html files and exist in the templates / folder of your widget. Example:
templates/list-example.html
```mustache
<ul>
    {{#links}}
        <li>{{title}}</li>
        <ul>
           {{#children}}
                <li>{{childTitle}}</li>
           {{/children}}
        </ul>
    {{/links}}
</ul>
```

####When to use external templates:

+ You need your template to be server-side rendered
+ You need caching
+ You need data to be transformed in some kind of way

####Internal Templates

These templates are included within the widget definition. 
+ Always put the template in a `<script/>` tag, for performance reasons.
+ Append all templates at the end of the widget's `<body>` element.
+ Use the data-template attribute to name your templates.

Example index.html:
```HTML
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd">
<html xmlns="http://www.w3.org/1999/xhtml" xmlns:g="http://www.backbase.com/2008/gadget" xml:lang="en">
<head>
    <!-- Head Section -->
</head>
<body>
    <!-- Body Section -->
    <script type="text/html" data-template="link-list">
        <ul>
            {{#links}}
                <li>{{title}}</li>
                <ul>
                   {{#children}}
                        <li>{{childTitle}}</li>
                   {{/children}}
                </ul>
            {{/links}}
        </ul>
    </script>
</body>
</html>
```

####When to use internal templates:

+ You need to use the template dynamically (not when the widget is rendered but at some other point in time, eg. after some user interaction)
+ You need to access the template directly in JS
+ You need to get the data with an ajax call

### LP/BB-Templates

LP/BB-Templates are separate files which you can use them to decrease the big amount of code in the index.html file, it's quite easy to work with, basically what you need to do is call the lp-template directive.
 
####How to use lp/bb-templates
```HTML
<body g:onload="requireWidget(__WIDGET__, 'scripts/main')" class="todo">
	<div ng-controller="MainCtrl as mainCtrl">
        <div lp-template="'templates/main.ng.html'"></div>
    </div>
</body>
```

 
##CSS

###Introduction

Writing good CSS code is often underestimated and not taken into account during projects. This results into serious problems in the end, including UI inconsistencies in the portal and performance issues. There are a lot of good books and websites about structuring CSS for large applications, including OOCSS and SMACSS.

###Themeing

Themeing should happen on a project level and not per widget. This means that styles affecting what is called "branding" should happen on your themes and not in your widget. These include things like font styling and colors, backgrounds, borders etc. As a guideline, use the following list for things that should be part of your theme and not your widget

+ typography (font styles on headings and paragraphs, related colors)
+ form elements (any styles on inputs, dropdowns, checkboxes etc)
+ links and buttons
+ navigation elements, breadcrumbs
+ grid system
+ specific helper classes (eg. clearfix, hidden etc)

Twitter Bootstrap is the way of defining and using styles for your project. It comes automatically included in themes and available in any widget you built. Use the Bootstrap CSS classes along with the custom classes in your widget for a fully styled widget.

###CSS Development

####Guidelines

+ Don't use ids for styling.
+ Avoid long selectors.
+ Avoid styling tags, use classes instead (eg. instead of styling ul.todo-list > li create a new class named todo-list-item and style directly that)
+ Prefer using CSS3 over techniques (eg. gradients or rounded corners)
+ Group relative styles together, avoid re-writing the same rules.
+ Try to be semantic with class names when needed, but not only for the sake of semantics (eg. grid classes like: span12 are fine, even if they are not semantic at all)
+ Don't write all styles in one line.
+ Logically group your styles.
+ Overriding Bootstrap classes in a widget should only happen for exceptions very specific to the widget.
+ Overriding Bootstrap classes in a widget happens by prepending the widget class to the bootstrap class (eg. override .btn by using the rule .todo .btn)
+ Don't use @import in your widget CSS file.
+ Don't use !important.

####Coding conventions

+ Class names should be all lowercase, with dashes as separators (eg. my-article-comments)
+ CSS classes must start with `<project prefix>-<widget name>- (eg. todo-list-item)`

####CSS Preprocessors

Backbase have chosen for LESS when working with twitter bootstrap theme's to give the benefits of general CSS preprocessing. Twitter Bootstrap also have opted for LESS as there go to preprocessors, giving us better all round support in combination. Plus we get a few tools to boot: http://getbootstrap.com/customize/#less

####BAD LESS / CSS

```
/* Example 1
** Long selectors
**/
.todo-sidebar .todo-block .todo-small-text {
    font-size: 12px;
    color: #ddd;
}
/* Selector performance is bad and you cannot re-use this note style elsewhere */
 
/* Example 2
** Grouping selectors
**/
.todo-list-item {
   color: #333;
   font-size: 13px;
   text-decoration: underline;
}
 
.todo-note {
   color: #333;
   font-size: 13px;
}
/* Code is repeated, styles are not re-usable */

 
/* Example 3
** Tidy styles
**/
.todo-main-title{padding:10px;font-size:2em;color:#c00;margin-bottom:1em;font-weight:bold;background:#fff;}
/* Code is unreadable and unmaintainable. Version Control is also impossible */
```

####GOOD LESS / CSS

```
/* Example 1
** Long selectors
**/
.todo {
	.todo-note {
		color: #ddd;
    	font-size: 12px;
    }
}
/* faster, more semantic, and re-usable everywhere */
 
/* Example 2
** Grouping selectors
**/
.todo {
	.todo-list-item, .todo-note {
   		color: #333;
   		font-size: 13px;
   	}
}

.todo {
	.todo-list-item {
   		text-decoration: underline;
   	}
}
/* Code is less bloated and more manageable */
  
/* Example 3
** Tidy styles
**/
.todo {
	.todo-main-title {
   		/* fonts group */
   		font-size: 2em;
   		font-weight: bold;
 
   		/* colors group */
   		color: #c00;
   		background: #fff;
 
   		/* box model group */
   		padding: 10px;
   		margin-bottom: 1em;
 
   		/* position group also possible */
   	}
}
/* rules are logically organized by group and easy to lookup and maintain */
```

##Javascript

###Starting Point

These guidelines will help you jump-start your widget development in a structured way.

###Guidelines
+ Begin your module with "use strict"
+ Use the prototype pattern for structuring your widget js code.
+ Use Launchpad dependency management mechanism (RequireJS).
+ Use constants for your string selectors at the top of the file. Separate into SELECTORS (for traversing the DOM, eg. to find all list items) and CLASSES (for behavior, eg. a class to make an element hidden).
+ Add 2 properties in your prototype: widget and $widgetBody, for easy access of the other methods.
+ Cache jQuery objects in the constructor. Name those objects with names starting with $ (to immediately understand which objects are jQuery ones).
+ Avoid running the same jQuery selectors again and again either cache (previous point) or use chaining
+ Avoid heavy DOM manipulation with javascript, for performance reasons
+ Define an init() method that starts the widget.
+ At the end of your module, return a function that returns an initiated instance of the widget.
+ Write JSDoc comments
+ Variable definitions within a function should be done in the beginning of the function
+ Use classes to hold state (eg. $element.addClass('hidden') instead of $element.hide() )

###Coding conventions

+ Widget prototype should be a TitleCase version of the widget name (eg. TransactionActivity)
+ Function names and variables should be camelCase.
+ All capital words mean constant
+ Group private functions under the _private object

###Available JS Frameworks

+ RequireJS is always be used when defining the widget resources and introducing JS behaviour, as in the code sample
+ AngularJS is our recommended toolset when widget functionalities meet our definition of what requires an MVC pattern. See Advanced Concepts


###Code Sample main.js

```
 /**
 *  ----------------------------------------------------------------
 *  Copyright © Backbase B.V.
 *  ----------------------------------------------------------------
 *  Filename : main.js
 *  Description: ${widget.description}
 *  ----------------------------------------------------------------
 */

define( function (require, exports, module) {
    'use strict';

    module.name = 'ginclude';

    var deps = [];

    /**
     * @ngInject
     */
    function run() {
        // Module is Bootstrapped
    }

    module.exports = base.createModule(module.name, deps)
        .constant('WIDGET_NAME', module.name)
        .controller( require('./controllers') )
        .service( require('./models') )
        .run( run );
});
```

###Code Sample controller.js

```
/**
 * Controllers
 * @module controllers
 */
define(function (require, exports) {
    'use strict';

    /**
     * Main controller
     * @ngInject
     * @constructor
     */
    function MainCtrl(WidgetModel) {
        this.model = WidgetModel;
    }

    /**
     * Export Controllers
     */
    exports.MainCtrl = MainCtrl;
});
```
 
##Advanced Concepts

###Event Binding
+ Bind events only within the context of the widget. Either delegate them from the $widgetBody, or bind directly on an element which is specific for the widget (ie. an element that is selected within the $widgetBody).
+ Do all the event binding in the widget init() function.
+ Create the handling of the event in a separate function.
+ Put a short comment to explain what the event does.

```
Todo.prototype = {
    init : function () {
        /* Click on an item to toggle its completed status */
        this.$widgetBody.on('click', SELECTORS.LIST_ITEM, selectListItemHandler);
    }

    var selectListItemHandler= function(evt) {

    };
};
```
 
###Pubsub Pattern

Use the pubsub pattern for communication with components outside the widget. Pubsub pattern is good for avoiding hard coupling the widget with the rest of the portal, but it still is some sort of coupling and should be used with caution.
+ Don't use pubsub for UI logic. For example if you need to press a button somewhere else in the page and make the widget appear, this should not be solved with pubsub. (chromes, containers and perspectives are options for solving this problem).
+ When using subscribe, make sure your widget works without it. Lets assume you have 2 widgets: Filters widget and List widget. The List widget "listens" to a channel that the Filter widget publishes the filter changed event to. The List widget should in any case have some default settings for getting data, even if the Filter widget is not present in the page.
+ When you publish an event, pass along all the data. Don't depend on DOM or any other structure for pulling the data.
+ Pubsub is stateless and without any history information and should be treated as such (pass all the data needed with each publish, don't rely on previous states)


##Tips

###BB-CLI

You can easily generate your widgets using the Backbase CLI tool, also you can contribute with improvements for it.

To see more click [here](https://github.com/Backbase/bb-cli).
