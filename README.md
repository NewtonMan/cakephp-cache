# Cache plugin for CakePHP
[![Build Status](https://api.travis-ci.org/dereuromark/cakephp-cache.svg)](https://travis-ci.org/dereuromark/cakephp-cache)
[![Coverage Status](https://coveralls.io/repos/dereuromark/cakephp-cache/badge.png)](https://coveralls.io/r/dereuromark/cakephp-cache)
[![License](https://poser.pugx.org/dereuromark/cakephp-cache/license.svg)](https://packagist.org/packages/dereuromark/cakephp-cache)
[![Minimum PHP Version](http://img.shields.io/badge/php-%3E%3D%205.4-8892BF.svg)](https://php.net/)
[![Coding Standards](https://img.shields.io/badge/cs-PSR--2--R-yellow.svg)](https://github.com/php-fig-rectified/fig-rectified-standards)

## What is it for?
It is the successor of the 2.x CacheHelper and allows you to cache your complete views as HTML.
No dynamic parts anymore, just complete static content ready to be delivered.
If you don't want to set up ESI and other third party caching software, this CakePHP only approach
does the job.

It uses a dispatcher and a component.
Why not a helper anymore? Mainly because a helper is too limited and would
not be able to cache serialized views, e.g. JSON, CSV, RSS content which have been build view-less.

## Installation

You can install this plugin into your CakePHP application using [composer](http://getcomposer.org).

The recommended way to install composer packages is:
```
composer require dereuromark/cakephp-cache
```

Also don't forget to load the plugin in your bootstrap:
```php
Plugin::load('Cache');
// or
Plugin::loadAll();
```

## Usage
You need to add the component to the controllers you want to make cache-able:
```php
public $components = ['Cache.Cache'];
```

And your bootstrap needs to enable the dispatcher filter:
```php
DispatcherFactory::add('Cache.Cache');
```

The component creates the cache file, the dispatcher on the next request will discover it and deliver this static file instead as long
as the file modification date is within the allowed range. Once the file got too old it will be cleaned out.
The next complete dispatching process will make the component create a fresh cache file then.

### Global Configuration
The `CACHE` constant can be adjusted in your bootstrap and defaults to `tmp/cache/views/`.

If you need a prefix in order to allow multiple (sub)domains to deliver content in multiple languages for example, use
 `Configure::write('Cache.prefix', 'myprefix')` to separate them.

### Component Configuration
If you only want certain actions to be cached, provide them as `actions` array.
The default `cacheTime` can be set as global value, but if you want certain actions to be cached differently, use the `actions` array:
```php
'actions' => ['report', 'view' => DAY, 'index' => HOUR]
```

In case you want to further compress the output, you can either use the basic built in compressor:
```php
'compress' => true
```
or you can use any custom compressor using a callable:
```php
'compress' => function ($content, $ext) { ... }
```
The latter is useful if you want to control the compression per extension.


### Filter Configuration
In case you need to run this before other high priority filters to avoid those to be invoked, you can raise the `priority` config.
You can also adjust the `cacheTime` value for how long the browser should cache the unlimited cache files, defaults to `+1 day`.

### Clear the Cache
The Cache shell shipped with this plugin should make it easy to clear the cache manually:
```
cake cache clear [optional/url]
```

### Further Cache Shell Goodies
Using
```
cake cache info [optional/url/]
```
you get the amount of currently cached files.

Using
```
cake cache info /some-controller/some-action/?maybe=querystrings
```
You can get information on the cache of this particular URL, e.g. how long it is still cached.


### Debugging
In debug mode or with config `debug` enabled, you will see a timestamp added as comment to the beginning of the cache file.

## TODOS
- Limit filename length to around 200 (as it includes query strings) and add md5 hashsum instead as suffix
- Allow other caching approaches than just file cache
- Allow usage of subfolders for File cache to avoid the folder to have millions of files as a flat list
- What happens with custom headers set in the original request? How can we pass those to the final cached response?
- Re-implement the removed CacheHelper with its nocache parts?
- Backport to 2.x?
- Extract the common file name part into a trait for both component and filter to use.
- Evaluate combining the filter with https://github.com/WyriHaximus/MinifyHtml to further compress the output.