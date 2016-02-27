# RiotJS Style Guide

Opinionated *RiotJS Style Guide* for teams by [De Voorhoede](https://twitter.com/devoorhoede).


## Purpose

This guide provides a uniform way to structure your [RiotJS](http://riotjs.com/) code. Making it

* easier for developers / team members to understand and find things.
* easier for IDEs to interpret the code and provide assistance.
* easier to (re)use build tools you already use.
* easier to cache and serve bundles of code separately.

This guide is inspired by the [AngularJS Style Guide](https://github.com/johnpapa/angular-styleguide) by John Papa.


## Table of Contents

* [Module based development](#module-based-development)
* [Tag module names](#tag-module-names)
* [1 module = 1 directory](#1-module--1-directory)
* [Use `*.tag.html` extension](#use-taghtml-extension)
* [Use `<script>` inside tag](#use-script-inside-tag)
* [Keep tag expressions simple](#keep-tag-expressions-simple)
* [Keep tag options primitive](#keep-tag-options-primitive)
* [Tag name as style scope](#tag-name-as-style-scope)


## Module based development

Always construct your app out of small modules which do one thing and do it well. 

A module is a small self-contained part of an application. The RiotJS micro-framework is specifically designed to help you create *view-logic modules*, which Riot calls *tags*.

### Why?

Small modules are easier to learn, understand, maintain, reuse and debug. Both by you and other developers.

### How?

Each riot tag (like any module) must be [FIRST](https://addyosmani.com/first/): *Focused* ([single responsibility](http://en.wikipedia.org/wiki/Single_responsibility_principle)), *Independent*, *Reusable*, *Small* and *Testable*.

If your module does too much or gets too big, split it up into smaller modules which each do just on thing.
Also ensure your tag module works in isolation. For instance by adding a stand-alone demo.


## Tag module names

A tag module is a specific type of module, containing a Riot tag. 

Each module name must be:

* **Meaningful**: not overspecific, not overly abstract.
* **Short**: 2 or 3 words.
* **Pronouncable**: we want to be able talk about them.

Tag module names must also be:

* **Custom element spec compliant**: [include a hyphen](https://www.w3.org/TR/custom-elements/#concepts), don't use reserved names.
* **`app-` namespaced**: if very generic and otherwise 1 word, so that it can easily be reused in other projects.

### Why?

* The name is used to communicate about the module. So it must be short, meaningful and pronouncable.
* The `tag` element is inserted into the DOM. As such, they need to adhere to the spec.

### How?

```html
<!-- recommended -->
<app-header />
<user-list />
<range-slider />

<!-- avoid -->
<btn-group /> <!-- short, but unpronouncable. use `button-group` instead -->
<ui-slider /> <!-- all tags are ui elements, so is meaningless -->
<slider /> <!-- not custom element spec compliant -->
```


## 1 module = 1 directory

Bundle all files which construct a module into a single place.

### Why?

Bundling module files (Riot tags, tests, assets, docs, etc.) makes them easy to find, move and reuse.

### How?

Use the module name as directory name and file basename.
The file extension depends on the purpose of the file.

	modules/
		my-example/
			my-example.tag.html
			my-example.less
			...
			README.md

If your project uses nested structures, you can nest a module within a module.
For example a generic `radio-group` module may be placed directly inside "modules/". While `search-filters` may only make sense inside a `search-form` and may therefore be nested:

    modules/
        radio-group/
            radio-group.tag.html
        search-form/
            search-form.tag.html
            ...
            search-filters/
               search-filters.tag.html


## Use `*.tag.html` extension

Riot introduces a new concept called *tags*, and suggests to use a `*.tag` extension.
However in essence these tags are simply custom elements containing markup. Therefore you should **use the `*.tag.html` extension**.

### Why?

* Tells developers it's not just HTML, but a Riot tag element.
* Improves IDE support (signals how to interpret).

### How?

In case of [in-browser compilation](http://riotjs.com/guide/compiler/#in-browser-compilation):
```html
<script src="path/to/modules/my-example/my-example.tag.html" type="riot/tag"></script>
```

In case of [pre-compilation](http://riotjs.com/guide/compiler/#pre-compilation), set the [custom extension](http://riotjs.com/guide/compiler/#custom-extension):
```bash
riot --ext tag.html modules/ dist/tags.js
```


## Use `<script>` inside tag

While Riot supports writing JavaScript inside a tag element [without a `<script>`](http://riotjs.com/guide/#no-script-tag),
you should **always use `<script>`** around scripting. This is closer to web standards and prevents confusing developers and IDEs. 

### Why?

* Prevents markup being interpreted as script.
* Improves IDE support (signals how to interpret).
* Tells developers where markup stops and scripting starts.

### How?

```html
<!-- recommended -->
<my-example>
	<h1>The year is { this.year }</h1>
	
	<script>
		this.year = (new Date()).getUTCFullYear();
	</script>
</my-example>

<!-- avoid -->
<my-example>
	<h1>The year is { this.year }</h1>
	
	this.year = (new Date()).getUTCFullYear();
</my-example>
```


## Keep tag expressions simple

Riot's inline [expressions](http://riotjs.com/guide/#expressions) are 100% Javascript. This makes them extemely powerful, but potentially also very complex. Therefore you should **keep tag expressions simple**.

### Why?

* Complex inline expressions are hard to read.
* Inline expressions can't be reused elsewehere. This can lead to code duplication and code rot.
* IDEs typically don't have support for expression syntax, so your IDE can't autocomplete or validate.

### How?

Move complex expressions to tag methods or tag properties. 

```html
<!-- recommended -->
<my-example>
	{ year() + '-' + month() }
	
	<script>
		const twoDigits = (num) => ('0' + num).slice(-2);
		this.month = () => twoDigits((new Date()).getUTCMonth() +1);
		this.year  = () => (new Date()).getUTCFullYear();
	</script>
</my-example>

<!-- avoid -->
<my-example>
	{ (new Date()).getUTCFullYear() + '-' + ('0' + ((new Date()).getUTCMonth()+1)).slice(-2) }
</my-example>
```


## Keep tag options primitive

Riot supports passing options to tag instances using attributes on tag elements. Inside the tag instance these options are available through `opts`. For example the value of `my-attr` on `<my-tag my-attr="{ value }" />` will be available inside `my-tag` via `opts.myAttr`. 

While Riot supports passing complex JavaScript objects via these attributes, you should try to **keep the tag options as primitive as possible**. Try to only use [JavaScript primitives](https://developer.mozilla.org/en-US/docs/Glossary/Primitive) (strings, numbers, booleans) and functions. Avoid complex objects.

Exceptions to this rule are situations which can only be solved using objects (eg. collections or recursive tags) or well-known objects inside your app (eg. a product in a web shop).

### Why?

* By using an attribute for each option separately the tag has a clear and expressive API.
* By using only primitives and functions as option values our tag APIs are similar to the APIs of native HTML(5) elements. Which makes our custom elements directly familiar.
* By using an attribute for each option, other developers can easily understand what is passed to the tag instance.
* When passing complex objects it's not apparent which properties and methods of the objects are actually being used by the custom tags. This makes it hard to refactor code and can lead to code rot.

### How?

Use a tag attribute per option, with a primitive or function as value:

```html
<!-- recommended -->
<range-slider
	values="[10, 20]"
	min="0"
	max="100"
	step="5"
	on-slide="{ updateInputs }"
	on-end="{ updateResults }"
	/>
	
<!-- avoid -->
<range-slider config="{ complexConfigObject }">
```
```html
<!-- exception: recursive tag, like menu item -->
<menu-item>
	<a href="{ opts.url }">{ opts.text }</a>
	<ul if="{ opts.items }">
		<li each="{ item in opts.items }">
			<menu-item 
				text="{ item.text }" 
				url="{ item.url }" 
				items="{ item.items }" />
		</li>
	</ul>
</menu-item>
```


## Tag name as style scope

Riot tag elements are custom elements which can very well be used as style scope root.
Alternatively the module name can be used as CSS class namespace.

### Why?

* Scoping styles to a tag element improves predictability as its prevents styles leaking outside the tag element.
* Using the same name for the module directory, the Riot tag and the style root makes it easy for developers to understand they belong together.

### How?

Use the tag name as selector, as parent selector or as namespace prefix (depending on your CSS naming strategy):

```css
/* recommended */
my-example { }
my-example li { }
.my-example__item { }

/* avoid */
.my-alternative { } /* not scoped to tag or module name */
.my-parent .my-example { } /* .my-parent is outside scope, so should not be used in this file */
```
