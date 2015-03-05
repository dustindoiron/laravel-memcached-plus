# laravel-memcached-plus

Integrating with cloud memcached services such as [MemCachier](https://www.memcachier.com/) and
[memcached cloud](https://redislabs.com/memcached-cloud) can require memcached features not available
with the built-in [Laravel 5 Cache](http://laravel.com/docs/5.0/cache) memcached driver.

These include:

* persistent connections
* SASL authentication 
* custom options

Adding new configuration items, this package _enhances_ the built-in Laravel 5 Cache memcached driver.
If you don't use the extra configuration items, the built-in memcached driver will not be overridden.

Optionally, this package also allows these extra configuration items to be used for memcached
Sessions.

## Requirements

* >= PHP 5.4 with [ext-memcached](http://php.net/manual/en/book.memcached.php)
* To use [SASL](http://docs.php.net/manual/en/memcached.setsaslauthdata.php) it must be compiled with
SASL support. This is the default on [Heroku](https://devcenter.heroku.com/articles/php-support)

## Installation

Available to install as a Composer package on
[Packagist](https://packagist.org/packages/b3it/laravel-memcached-plus), all you need to do is:

`composer require b3it/laravel-memcached-plus`

If your local environment does not meet the requirements you may need to append the
`ignore-platform-reqs` option:

`composer require b3it/laravel-memcached-plus --ignore-platform-reqs`

## Configuration

### Providers

Once installed you can use this package to enhance the Laravel Cache and Session services.

In your laravel application configuration `app/config.php` you need to append the Service Providers
from this package to the `providers` array:

```
'providers' => [
    ...
    /*
     * Application Service Providers...
     */
     ...

    'B3IT\MemcachedPlus\CacheServiceProvider',
    'B3IT\MemcachedPlus\SessionServiceProvider',
],
```

As of Laravel 5.0.14 this is on line 148.

The `B3IT\MemcachedPlus\SessionServiceProvider` is optional. You only need to add this if:

* You want to specify the memcached store from the session config (see below), or
* You want MemcachedPlus sessions!

### Cache

This section discusses the Laravel cache configuration file `config/cache.php`.

This package makes the following extra configuration items are available for use with a memcached store:

* `persistent_id` - [`Memcached::__construct`] (http://php.net/manual/en/memcached.construct.php)
explains how this is used
* `sasl` - used by [`Memcached::setSaslAuthData`](http://php.net/manual/en/memcached.setsaslauthdata.php)
* `options` - see [`Memcached::setOptions`](http://php.net/manual/en/memcached.setoptions.php)

These may be used in a store config like so:

```
'stores' => [
    'memcachedstorefoo' => [
        'driver'  => 'memcached',
        'persistent_id' => 'laravel',
        'sasl'       => [
            env('MEMCACHIER_USERNAME'),
            env('MEMCACHIER_PASSWORD')
        ],
        'options'    => [
            'OPT_NO_BLOCK'         => true,
            'OPT_AUTO_EJECT_HOSTS' => true,
            'OPT_CONNECT_TIMEOUT'  => 2000,
            'OPT_POLL_TIMEOUT'     => 2000,
            'OPT_RETRY_TIMEOUT'    => 2,
        ],
        'servers' => [
            [
                'host' => '127.0.0.1', 'port' => 11211, 'weight' => 100
            ],
        ],
    ],
],
```

When defining `options` you should set the config key to the `Memcached` constant name as a string.
This avoids any issues with local environments missing ext-memcached and throwing a warning about
undefined constants. The config keys are automatically resolved into `Memcached` constants by the
`MemcachedPlus\Connector` which throws a `RuntimeException` if the constant is invalid.

Note: as this package _extends_ the built-in Laravel 5 memcached Cache driver the driver string
remains `memcached`.

### Session

This section discusses the Laravel session configuration file `config/session.php`.

If you are using memcached sessions you will have set the `driver` configuration item to 'memcached'.

If you have added the `B3IT\MemcachedPlus\SessionServiceProvider` as discussed above, the
`memcached_store` configuration item is available. This is explained in the following new snippet
you can paste into your session configuration file:

```
    /*
    |--------------------------------------------------------------------------
    | Session Cache Store
    |--------------------------------------------------------------------------
    |
    | When using the "memcached" session driver, you may specify a cache store
    | that should be used for these sessions. This should correspond to a
    | store in your cache configuration options which uses the memcached
    | driver.
    |
    */

    'memcached_store' => 'memcachier',
```

## Support

Please do let me know if you have any comments or queries.

Thanks!
