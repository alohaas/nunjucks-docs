---
layout: subpage
title: API
---
{% raw %}

# API

The API for nunjucks covers rendering templates, adding filters and
extensions, customizing template loading, and more.

## Simple API

If you don't need deep customization of the system, you can use this simple
higher-level API for loading and rendering templates.

{% endraw %}
{% api %}
render
nunjucks.render(name, [context], [callback])

Renders the template named **name** with the **context** hash. If
**callback** is provided, it will be called when done with any
possible error as the first argument and the result as the second.
Otherwise, the result is returned from `render` and errors are thrown.
See [asynchronous support](#asynchronous-support) for more info.

```js
var res = nunjucks.render('foo.html');

var res = nunjucks.render('foo.html', { username: 'James' });

nunjucks.render('async.html', function(err, res) {
});
```
{% endapi %}

{% api %}
renderString
nunjucks.renderString(str, context, [callback])

Same as [`render`](#render), but renders a raw string instead of
loading a template.

{% raw %}
```js
var res = nunjucks.render('Hello {{ username }}', { username: 'James' });
```
{% endraw %}
{% endapi %}

{% api %}
configure
nunjucks.configure([path], [opts]);

Tell nunjucks that your templates live at **path** and flip any
feature on or off with the **opts** hash. You can provide both
arguments or either of them. **path** defaults to the current working
directory, and the following options are available in **opts**:

* **autoescape** *(default: false)* controls if output with dangerous characters are
    escaped automatically. See [autoescaping](#).
* **tags:** *(default: see nunjucks syntax)* defines the syntax for nunjucks tags. Should be a hash with
    the following entries: `blockStart`, `blockEnd`, `variableStart`,
    `variableEnd`, `commentStart`, and `commentEnd`.

`configure` returns an `Environment` instance, which lets you add
filters and extensions while still using the simple API. See below for
more information on `Environment`.

```js
nunjucks.configure('views', { autoescape: true });

nunjucks.configure('views');

nunjucks.configure({ autoescape: true });

var env = nunjucks.configure('views');
env.addFilter('foo', function(val) { return 'foooo-ed: ' + val; });
```

{% endapi %}
{% raw %}

That's it for the simple API! If you want total control over how
templates are loaded, and more customization, you need to manually
set up the system as seen below.

## Environment

The `Environment` class is the central object which handles templates.
It knows how to load your templates, and internally templates depend
on it for inheritance and including templates. The simple API above
dispatches everything to an `Environment` instance that it keeps for
you.

You can manually handle it if you want, which allows you to specify
custom template loaders.

{% endraw %}
{% api %}
constructor
new Environment([loaders], [opts])
{% endapi %}
{% raw %}

The constructor takes a list of **loaders** and a hash of
configuration parameters as **opts**. Both are optional, but you need
to pass `null` for **loaders** if you *only* want to specify **opts**.
You can pass a single loader or an array of loaders. If you pass an
array of loaders, nunjucks will walk through them in order until one
of them finds a template. See [`Loader`](#loader) for more info about loaders.

The available flags in **opts** is exactly the same as defined in
[`configure`](#configure).

In node, the `FileSystemLoader` is available to load templates off the
filesystem, and in the browser the `WebLoader` is available to load
over HTTP (or use precompiled templates). If you use the simple
[`configure`](#configure) API, nunjucks automatically creates the
appropriate loader for you, depending if your in node or the browser.
See [`Loader`](#loader) for more information.

```js
// the FileSystemLoader is available if in node
var env = new Environment(new nunjucks.FileSystemLoader('views'));

var env = new Environment(new nunjucks.FileSystemLoader('views'), { autoescape: false });

var env = new Environment([new nunjucks.FileSystemLoader('views'),
                           new MyCustomLoader()]);

// the WebLoader is available if in the browser
var env = new Environment(new nunjucks.WebLoader('/views'));
```

{% endraw %}
{% api %}
addFilter
env.addFilter(name, func, [async])

Add a custom filter named **name** which calls **func** whenever
invoked. If the filter needs to be async, **async** must be `true`
(see [asynchronous support](#asynchronous-support)). **func** receives
the arguments passed to the filter, and a callback if asynchronous.

See <a href="/api#Custom-Filters">Custom Filters</a>.

{% endapi %}
{% raw %}

### Loading a template

Call the `getTemplate` method to retrieve a template.

```
var tmpl = env.getTemplate('page.html');
tmpl.render({ foo: 'bar' });
```

### Rendering a template

The `render` method is a shortcut for rendering templates.

```js
env.render('page.html', { foo: 'bar' });

// is equal to

env.getTemplate('page.html').render({ foo: 'bar' });
```

### Registering with Express

Nunjucks comes with special [express](http://expressjs.com/) support. Simple call the `express` method and pass the app and everything will be set up for you.

```js
var app = express();
env.express(app);
```

For more details, see [Express support](/home#Express) on the overview page.

## Template

A `Template` is an object that handles the compiling of template
strings and rendering them. The `Environment` method `getTemplate`
returns a `Template` object. You can also create one yourself.

```js
var nunjucks = require('nunjucks');
var tmpl = new nunjucks.Template('Hello {{ username }}');
```

Call the `render` method to render it with a context.

```js
tmpl.render({ username: "James" });

// Displays "Hello James"
```

You cannot use template inheritance if you create the Template by
hand. Only templates retrieved from an `Environment` can use
inheritance.

## Loader

### node

The `FileSystemLoader` is available. This takes a path to load templates from. If you don't pass any loaders, a `FileSystemLoader` is created pointing to the current working directory.

```js
var nunjucks = require('nunjucks');

// Loads templates from the current working directory
var env = new nunjucks.Environment();

// Loads templates from the "views" folder
var env = new nunjucks.Environment(new nunjucks.FileSystemLoader('views'));
```

### browser

The `HttpLoader` is available. This takes a URL to load templates from (must be the same domain, relative URLs are recommended). If you don't pass any loaders, a `HttpLoader` is created with the `/views` URL.

```
var nunjucks = require('nunjucks');

// Loads templates from /views
var env = new nunjucks.Environment();

// Load templates from /templates
var env = new nunjucks.Environment(new nunjucks.HttpLoader('/templates'))
```

### Multiple Loaders

You can also specify multiple loaders and it will look through them in order. This is useful if you want to specify several different template paths, or use a custom loader.

```
// The environment will look in templates first,
// then foo/templates, and then try loading
// from your special MyLoader class
var env = new nunjucks.Environment([new nunjucks.FileSystemLoader('templates'),
                                    new nunjucks.FileSystemLoader('foo/templates'),
                                    new MyLoader()]);
```

## Asynchronous Support

You only need to read this section if you are interested in
asynchronous rendering. There is no performance benefit to this, it is
soley to allow custom filters and extensions to make async calls. If
you don't care about this, you should simply use the normal API like
`var res = env.render('foo.html');`. There's no need to force the
`callback` on you, and it's why it's optional in all the rendering
functions.

As of version 1.0, nunjucks provides a way to render templates
asynchronously. This means that custom filters and extensions can do
stuff like fetch things from the database, and template rendering is
"paused" until the callback is called.

Template loaders can be async as well, allowing you to load templates
from a database or somewhere else. If you are using an async template
loader, you must use the async API. The builtin loaders that load from
the filesystem and over HTTP are synchronous, which is not a
performance problem because they are cached from the filesystem and you
should precompile your templates and never use HTTP in production.

Here's an example of using the async API:

```js
var env = nunjucks.configure('views');

env.addFilter('lookup', function(name, cb) {
    db.getItem(name, cb);
}, true);

env.render('{{ item|lookup }}', function(err, res) {
    // do something with res
});
```

Note that `addFilter` requires `true` as the last argument to indicate
that it's async. Read more async [`filters`](), [`extensions`](), and
[`loaders`]().

## Autoescaping

By default, nunjucks will render all output as it is. If turn on autoescaping, nunjucks will escape all output by default. It's recommended that you do this for security reasons.

Autoescaping is rather simplistic in nunjucks right now. All you have to do is pass the `autoescape` option as `true` to the `Environment` object. In the future, you will have more control over which files this kicks in.

```js
var env = new nunjucks.Environment(loaders, { autoescape: true });
```

## Customizing Variable and Block Tags

If you want different tokens than `{{` and the rest for variables, blocks, and comments, you can specify different tokens as the `tags` option:

```js
var env = new nunjucks.Environment(loaders, {
  tags: {
    blockStart: '<%',
    blockEnd: '%>',
    variableStart: '<$',
    variableEnd: '$>',
    commentStart: '<#',
    commentEnd: '#>'
  }
});
```

Using this environment, templates will look like this:

```
<ul>
<% for item in items %>
  <li><$ item $></li>
<% endfor %>
</ul>
```

## Using Nunjucks in the Browser

Since we're using javascript already, all of your templates are available in the browser! This is extremely helpful because you no longer have to write templates specifically for either the server or the browser.

First, you need to download the js file for the browser. You can get it in the [`browser` folder of nunjucks](https://github.com/jlongster/nunjucks/tree/v0.1.10/browser).

You'll see 3 different files:

* `nunjucks-dev.js`: To be used in development, this includes the full compiler and loads templates individually when they are requested. Templates are not cached, so changes are automatically available.
* `nunjucks.js`: This is a 20K file that only includes the runtime needed for templates (no compiler). You must precompile your templates.
* `nunjucks-min.js`: To be used in production, this is an 8K file that is simply a minified version of `nunjucks.js`. You must precompile your templates.

You should use `nunjucks-dev.js` in development, and **only `nunjucks-min.js` in production**. The production version is *much* faster.

Once one of these files is loaded in the browser, a global object named `nunjucks` will be available with the same API as if you required it with node.

### Precompiling Templates

When you use `nunjucks-min.js` in production, **you have to precompile your templates**. Precompilation simply generates the javascript for your template once, and you save it and only load the generated code into the page. This is much faster than compiling templates every time the page loads.

Nunjucks comes with a script named `nunjucks-precompile` to perform this task (make sure you have `node_modules/.bin` in your `$PATH`). You pass it a folder and it compiles all of them and dumps javascript to standard out, which you should pipe into a single js file.

```
$ nunjucks-precompile
Usage: nunjucks-precompile <folder>

$ nunjucks-precompile views > templates.js

$ ls views
base.html
item.html
foo.html
```

Now just include `templates.js` in your site (or whatever you named the js file). Your script tags should look like this:

```html
<script src="/js/nunjucks-min.js"></script>
<script src="/js/templates.js"></script>
```

Make sure to include nunjucks before your templates. Your base js path might be different too.

The precompiled templates sets up an `Environment` object for you, which you can access as `nunjucks.env`. `nunjucks` is the global object defined by the main js file.

### Browser Integration for Development and Production

You don't want to change your javascript to specialize for the production version, which predefines an `Environment` object. Here's how you should make your code work in both environments:

```js
if(!nunjucks.env) {
    // If not precompiled, create an environment with an HTTP
    // loader
    nunjucks.env = new nunjucks.Environment(new nunjucks.HttpLoader('/views'));
}

// Define a shortcut if you want
var env = nunjucks.env;
```

You should change the `"/views"` argument of `HttpLoader` to whatever base URL your templates are served on. You can also define the `Environment` object wherever you like, just make sure to check for `nunjucks.env` if it's defined for precompiled templates.

Now, use templates like you normally would:

```js
var el = $(env.render('item.html', { name: 'james' }));
$('body').append(el);
```

By default, the development version loads the template every time it's rendered. It's very helpful for changes to be automatically viewable in some cases, but you can turn it off by passing `true` when constructing the `HttpLoader` as a second argument:

```js
nunjucks.env = new nunjucks.Environment(
    // Cache templates and never reload them
    new nunjucks.HttpLoader('/views', true);
);
```

### AMD Support

*Released in 0.1.9*

Nunjucks fully supports AMD. The integration looks a little different, and conditionally including precompiled templates is a little tricky.

First off, if you are using an AMD loader like [require.js](http://requirejs.org/) you can simply load nunjucks like normal:

```js
define(['nunjucks'], function(nunjucks) {
    // ...
})
```

Nunjucks will not create a global object if you load it this way.

When you [precompile your templates](/api#Precompiling-Templates) in production like any good boy or girl, the generated file is also a normal AMD module. So you simply need to load it as well, and then it works the same way as non-AMD code (it automatically creates an Environment for you and assigns it to `nunjucks.env`). Assuming you compiled your templates into a `templates.js` file, your final code to support both development and production would look like this:

```js
define(['nunjucks', 'templates'], function(nunjucks) {
    if(!nunjucks.env) {
        nunjucks.env = new nunjucks.Environment(new nunjucks.HttpLoader('/views'));
    }
})
```

Replacing `/views` with whatever base URL your template live at, of course.

The only problem now is that you need to use the [nunjucks-dev.js](https://github.com/jlongster/nunjucks/tree/master/browser) file on dev and the [nunjucks-min.js](https://github.com/jlongster/nunjucks/tree/master/browser) on production, and require.js will try to load `templates.js` on dev. I recommend that you copy or symlink `nunjucks-dev.js` to `nunjucks.js` on dev and the same for `nunjucks-min.js` on prod, and create a blank `templates.js` file for dev.

A new precompiled API is coming in a future version to make the above easier.

## Custom Filters

TODO: talk about async here

To install a custom filter, use the `Environment` method `addFilter`.
A filter is simply a function that takes the target object as the
first argument and any arguments passed to the filter as the other
arguments, in order.

```js
var nunjucks = require('nunjucks');
var env = new nunjucks.Environment();

env.addFilter('shorten', function(str, count) {
    return str.slice(0, count || 5);
});
```

This adds a filter `shorten` which returns the first `count`
characters in a string, with `count` defaulting to 5. Here is how it
is used:

```jinja
{# Show the first 5 characters #}
A message for you: {{ message|shorten }}

{# Show the first 20 characters #}
A message for you: {{ message|shorten(20) }}
```

### Keyword/Default Arguments

As described in the [templating section](/templating#Keyword-Arguments), nunjucks supports keyword/default arguments. You can write a normal javascript filter that leverages them.

All keyword arguments are passed in as a hash as the last argument. This is a filter `foo` that uses keyword arguments:

```js
env.registerFilter('foo', function(num, x, y, kwargs) {
   return num + (kwargs.bar || 10);
})
```

The template can use it like this:

```jinja
{{ 5 | foo(1, 2) }}          -> 15
{{ 5 | foo(1, 2, bar=3) }}   -> 8
```

You *must* pass all of the positional arguments before keyword arguments (`foo(1)` is valid but `foo(1, bar=10)` is not). Also, you cannot set a positional argument with a keyword argument like you can in Python (such as `foo(1, y=1)`)

## Custom Tags & Extensions

You can create more complicated extensions by creating custom tags. This exposes the parser API and allows you to do anything you want with the template.

An extension is a javascript object with at least two fields: `tags` and `parse`. Extensions basically register new tag names and take control of the parser when they are hit.

`tags` is an array of tag names that the extension should handle. `parse` is the method that actually parses them when the template is compiled. Additionally, there is a special node type `CallExtension` that you can use to call any method on your extension at runtime. This is explained more below.

Because you have to interact directly with the parse API and construct ASTs manually, this is a bit cumbersome. It's necessary if you want to do really complex stuff, however. Here are a few key parser methods you'll want to use:

* `parseSignature([throwErrors], [noParens])` - Parse a list of arguments. By default it requires the parser to be pointing at the left opening paranthesis, and parses up the right one. However, for custom tags you shouldn't use parantheses, so passing `true` to the second argument tells it to parse a list of arguments up until the block end tag. A comma is required between arguments. Example: `{% mytag foo, bar, baz=10 %}`

* `parseUntilBlocks(names)` - Parse content up until it hits a block with a name in the `names` array. This is useful for parsing content between tags.

The parser API needs to be more documented, but for now read the above and check out the example below. You can also look at the [source](https://github.com/jlongster/nunjucks/blob/master/src/parser.js).

The most common usage is to process the content within some tags at runtime. It's like filters, but on steroids because you aren't confined to a single expression. You basically want to lightly parse the template and then get a callback into your extension with the content. This is done with the `CallExtension` node, which takes an extension instance, the method to call, list of arguments parsed from the tag, and a list of content blocks (parsed with `parseUntilBlocks`).

For example, here's how you would implement an extension that fetches content from a URL and injects it into the page:

```js
function RemoteExtension() {
    this.tags = ['remote'];

    this.parse = function(parser, nodes, lexer) {
        // get the tag token
        var tok = parser.nextToken();

        // parse the args and move after the block end. passing true
        // as the second arg is required if there are no parentheses
        var args = parser.parseSignature(null, true);
        parser.advanceAfterBlockEnd(tok.value);

        // parse the body and possibly the error block, which is optional
        var body = parser.parseUntilBlocks('error', 'endtruncate');
        var errorBody = null;

        if(parser.skipSymbol('error')) {
            parser.skip(lexer.TOKEN_BLOCK_END);
            errorBody = parser.parseUntilBlocks('endremote');
        }

        parser.advanceAfterBlockEnd();

        // See above for notes about CallExtension
        return new nodes.CallExtension(this, 'run', args, [body, errorBody]);
    };

    this.run = function(context, url, body, errorBody) {
        var id = 'el' + Math.floor(Math.random() * 10000);
        var ret = new nunjucks.runtime.SafeString('<div id="' + id + '">' + body() + '</div>');
        var ajax = new XMLHttpRequest();

        ajax.onreadystatechange = function() {
            if(ajax.readyState == 4) {
                if(ajax.status == 200) {
                    document.getElementById(id).innerHTML = ajax.responseText;
                }
                else {
                    document.getElementById(id).innerHTML = errorBody();
                }
            }
        };

        ajax.open('GET', url, true);
        ajax.send();

        return ret;
    };
}

env.addExtension('RemoteExtension', new RemoteExtension());
```

Use it like this:

```jinja
{% remote "/stuff" %}
  This content will be replaced with the content from /stuff
{% error %}
  There was an error fetching /stuff
{% endremote %}
```

This is new functionality and will improve over time. If you create anything interesting, make sure to [add it to the wiki!](https://github.com/jlongster/nunjucks/wiki/Custom-Tags)

{% endraw %}