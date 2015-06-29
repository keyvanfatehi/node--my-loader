# my-loader

Require modules matching glob while respecting that exported arrays may contain desired object

## Description

This script loader initializes with a directory (e.g. `__dirname`)

You must call the `configure()` method on the resulting instance

The configuration object allows you to create named objects with `glob` and `leaf` attributes for use later by the `load` method.

The glob is used with the directory name to require
all the scripts.

The `leaf` function is passed every module.
It is there that you have a chance to return true or false
to indicate if this module should be included in the output.

If the `leaf` function is defined properly, the loader
traverses deeply into objects and arrays to produce a
normalized array containing all objects matching your
criteria. This is what makes this loader useful compared
to other loaders that don't allow inspecting and acting
on the logical subunits of a given module.

The `load` method defined on the loader instance takes the configuration key name and returns a promise that resolves with an array of loaded leaves.

## Example

Using my-loader to define a folder convention and load objects for a Hapi server:

```
var Hapi = require('hapi');
var loader = require('./my-loader')(__dirname)

loader.configure({
  auth: {
    glob: 'auth/**/*.js',
    leaf: function(i) { return i.length === 3; }
  },
  routes: {
    glob: 'routes/**/*.js',
    // Note that in my routes I can have route objects and arrays of route objects. `my-loader` will find them by always searching for leaves that have `handler` defined.
    leaf: function(i) { return i.handler; }
  },
  plugins: {
    glob: 'plugins/**/*.js',
    leaf: function(i) { return i.register; }
  }
});

var server = new Hapi.Server();

server.connection({ port: 3000 });

loader.load('plugins').then(function(plugins) {
  server.register(plugins), function(err) {
    if (err) throw err;
    loader.load('auth').map(function(strategy) {
      server.auth.strategy.apply(this, strategy);
    });
    loader.load('routes').then(function(routes) {
      server.route(routes);
    });
  })
})

server.start()
```
