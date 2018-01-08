# Magento Performance

There are multiple quick and simple methods of improving base magento performance that do not require any code modifications.
* [Enabling production mode](http://devdocs.magento.com/guides/v2.0/config-guide/cli/config-cli-subcommands-mode.html) - this will disable a lot of file system lookups by assuming all dynamically generated code (so make sure You did run `setup:di:compile` and `setup:static-content:deploy` commands or your store will brake).
* [Enabling and tuning opcache settings for PHP interpreter](http://php.net/manual/en/opcache.configuration.php) will allow to reduce number of file system lookups by caching preparsed class definitions between requests. 
* [Using redis for cache and session storage](http://devdocs.magento.com/guides/v2.0/config-guide/redis/config-redis.html) - will move filesystem heavy operation to in-memory key-value storage service.
* [Using Varnish as full page cache provider](http://devdocs.magento.com/guides/v2.0/config-guide/varnish/config-varnish.html) - will completly prevent running PHP code for cached requests.
* [Running optimized composer autoload dump](https://getcomposer.org/doc/articles/autoloader-optimization.md) - will decrease number of filesystem operations needed to perform class autoloading (do remember to do this after auto-generated classes are generated - so after `setup:di:compile` there are a lot of points loading those classes, improving their autoloading is crucial)

As You can see most of those quick methods are cache based.
This means that is mostly hiding other issues under cache layer.
In order to address other issues we have to take a look at code.
At this point major performance problems in core Magento code are resolved, but You still have to be carefull about code you create or third party modules You install.

* [Profiling](performance/profiler.md) will allow to spot performance bottlenecks.
* [EAV](performance/eav.md) understanding how some more complex objects are stored in database will allow for more optimalized database related code.
