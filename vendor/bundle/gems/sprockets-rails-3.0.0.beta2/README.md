# Sprockets Rails

Provides Sprockets implementation for Rails 4.x (and beyond) Asset Pipeline.


## Installation

``` ruby
gem 'sprockets-rails', :require => 'sprockets/railtie'
```

Or alternatively `require 'sprockets/railtie'` in your `config/application.rb` if you have Bundler auto-require disabled.


## Usage


### Rake task

**`rake assets:precompile`**

Deployment task that compiles any assets listed in `config.assets.precompile` to `public/assets`.

**`rake assets:clean`**

Only removes old assets (keeps the most recent 3 copies) from `public/assets`. Useful when doing rolling deploys that may still be serving old assets while the new ones are being compiled.

**`rake assets:clobber`**

Nuke `public/assets`.

#### Customize

If the basic tasks don't do all that you need, it's straight forward to redefine them and replace them with something more specific to your app.

You can also redefine the task with the built in task generator.

``` ruby
require 'sprockets/rails/task'
Sprockets::Rails::Task.new(Rails.application) do |t|
  t.environment = lambda { Rails.application.assets }
  t.assets = %w( application.js application.css )
  t.keep = 5
end
```

Each asset task will invoke `assets:environment` first. By default this loads the Rails environment. You can override this task to add or remove dependencies for your specific compilation environment.

Also see [Sprockets::Rails::Task](https://github.com/rails/sprockets-rails/blob/master/lib/sprockets/rails/task.rb) and [Rake::SprocketsTask](https://github.com/rails/sprockets/blob/master/lib/rake/sprocketstask.rb).


### Initializer options

**`config.assets.precompile`**

Add additional assets to compile on deploy. Defaults to `application.js`, `application.css` and any other non-js/css file under `app/assets`.

**`config.assets.paths`**

Add additional load paths to this Array. Rails includes `app/assets`, `lib/assets` and `vendor/assets` for you already. Plugins might want to add their custom paths to this.


**`config.assets.version`**

Set a custom cache buster string. Changing it will cause all assets to recompile on the next build.

``` ruby
config.assets.version = 'v1'
# after installing a new plugin, change loads paths
config.assets.version = 'v2'
```

**`config.assets.prefix`**

Defaults to `/assets`. Changes the directory to compile assets to.

**`config.assets.manifest`**

Defines the full path to be used for the asset precompiler's manifest file. Defaults to a randomly-generated filename in the `config.assets.prefix` directory within the public folder.

**`config.assets.digest`**

When enabled, fingerprints will be added to asset filenames.

**`config.assets.debug`**

Enable expanded asset debugging mode. Individual files will be served to make referencing filenames in the web console easier. This feature will eventually be deprecated and replaced by Source Maps in Sprockets 3.x.

**`config.assets.compile`**

Enables Sprockets compile environment. If disabled, `Rails.application.assets` will be `nil` to prevent inadvertent compilation calls. View helpers will depend on assets being precompiled to `public/assets` in order to link to them. Initializers expecting `Rails.application.assets` during boot should be accessing the environment in a `config.assets.configure` block. See below.

**`config.assets.configure`**

Invokes block with environment when the environment is initialized. Allows direct access to the environment instance and lets you lazily load libraries only needed for asset compiling.

``` ruby
config.assets.configure do |env|
  env.js_compressor  = :uglifier # or :closure, :yui
  env.css_compressor = :sass   # or :yui

  require 'my_processor'
  env.register_preprocessor 'application/javascript', MyProcessor

  env.logger = Rails.logger

  env.cache = ActiveSupport::Cache::FileStore.new("tmp/cache/assets")
end
```


## Complementary plugins

The following plugins provide some extras for the Sprockets Asset Pipeline.

* [coffee-rails](https://github.com/rails/coffee-rails)
* [sass-rails](https://github.com/rails/sass-rails)

**NOTE** That these plugins are optional. The core coffee-script, sass, less, uglify, (any many more) features are built into Sprockets itself. Many of these plugins only provide generators and extra helpers. You can probably get by without them.


## Changes from Rails 3.x

* Only compiles digest filenames. Static non-digest assets should simply live in public/.
* Unmanaged asset paths and urls fallback to linking to public/. This should make it easier to work with both compiled assets and simple static assets. As a side effect, there will never be any "asset not precompiled errors" when linking to missing assets. They will just link to a public file which may or may not exist.
* JS and CSS compressors must be explicitly set. Magic detection has been removed to avoid loading compressors in environments where you want to avoid loading any of the asset libraries. Assign `config.assets.js_compressor = :uglifier` or `config.assets.css_compressor = :sass` for the standard compressors.
* The manifest file is now in a JSON format. Since it lives in public/ by default, the initial filename is also randomized to obfuscate public access to the resource.
* `config.assets.manifest` (if used) must now include the manifest filename, e.g. `Rails.root.join('config/manifest.json')`. It cannot be a directory.
* Two cleanup tasks. `rake assets:clean` is now a safe cleanup that only removes older assets that are no longer used. While `rake assets:clobber` nukes the entire `public/assets` directory and clears your filesystem cache. The clean task allows for rolling deploys that may still be linking to an old asset while the new assets are being built.

## Experimental

### [SRI](http://www.w3.org/TR/SRI/) support

Sprockets 3.x adds experimental support for subresource integrity checks. The spec is still evolving and the API may change in backwards incompatible ways.

``` ruby
javascript_include_tag :application, integrity: true
# => "<script src="/assets/application.js" integrity="sha-256-TvVUHzSfftWg1rcfL6TIJ0XKEGrgLyEq6lEpcmrG9qs="></script>"
```


## Contributing

Usual bundler workflow.

``` shell
$ git clone https://github.com/rails/sprockets-rails.git
$ cd sprockets-rails/
$ bundle install
$ bundle exec rake test
```

[![Build Status](https://travis-ci.org/rails/sprockets-rails.svg?branch=master)](https://travis-ci.org/rails/sprockets-rails)


## Releases

sprockets-rails 3.x will primarily target sprockets 3.x. And future versions will target the corresponding sprockets release line.

The minor and patch version will be updated according to [semver](http://semver.org/).

* Any new APIs or config options that don't break compatibility will be in a minor release
* Any time the sprockets dependency is bumped, there will be a new minor release
* Simple bug fixes will be patch releases


## License

Copyright &copy; 2014 Joshua Peek.

Released under the MIT license. See `LICENSE` for details.
