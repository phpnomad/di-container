# phpnomad/di-container

[![Latest Version](https://img.shields.io/packagist/v/phpnomad/di-container.svg)](https://packagist.org/packages/phpnomad/di-container) [![Total Downloads](https://img.shields.io/packagist/dt/phpnomad/di-container.svg)](https://packagist.org/packages/phpnomad/di-container) [![PHP Version](https://img.shields.io/packagist/php-v/phpnomad/di-container.svg)](https://packagist.org/packages/phpnomad/di-container) [![License](https://img.shields.io/packagist/l/phpnomad/di-container.svg)](https://packagist.org/packages/phpnomad/di-container)

`phpnomad/di-container` is the concrete dependency injection container that ships with [PHPNomad](https://phpnomad.com). It's a single class, `PHPNomad\Di\Container\Container`, that does reflection-based autowiring and needs no configuration files to work.

The split between interfaces and implementation is deliberate. `phpnomad/di` defines the contracts (`InstanceProvider`, `HasBindings`, `DiException`). `phpnomad/di-container` provides the class that implements them. Most applications install both, and installing `phpnomad/core` pulls them in transitively. PHPNomad has been running in production for years powering [Siren](https://sirenaffiliates.com) and several MCP servers, so this container has been resolving real dependency graphs the whole time.

## Installation

```bash
composer require phpnomad/di-container
```

Composer will pull in `phpnomad/di` automatically as a required dependency, so there's nothing else to install for a minimal setup.

## Quick Start

Construct the container, bind your abstracts to their concretes, and ask for what you need. Constructor dependencies are resolved recursively by reflection.

```php
<?php

use PHPNomad\Di\Container\Container;
use YourApp\Database;
use YourApp\PostRepository;
use YourApp\PostRepositoryInterface;

$container = new Container();

// Map an interface to a concrete class.
$container->bind(PostRepository::class, PostRepositoryInterface::class);

// Bind something that needs a runtime argument via a factory.
$container->bindFactory(
    Database::class,
    fn() => new Database($dsn)
);

// Resolve. PostRepository's constructor dependencies are autowired.
$repository = $container->get(PostRepositoryInterface::class);
```

If `PostRepository`'s constructor takes a `Database` parameter, the container hands it the `Database` instance produced by the factory. Any other typed, non-built-in constructor parameters get resolved the same way, all the way down. A second `get(PostRepositoryInterface::class)` call returns the same repository instance the first call returned.

## Key Concepts

- Reflection-based autowiring. Typed, non-built-in constructor parameters are resolved recursively through `get()`.
- `bind($concrete, $abstract, ...$moreAbstracts)` associates one concrete with one or more abstracts. All abstracts bound this way share the same resolved instance.
- `bindFactory($abstract, $callable)` binds an abstract to a callable. Use it when construction needs a runtime value like an existing database handle, a config array, or an environment variable.
- `get($abstract)` is memoized. The first call instantiates, every call after that returns the cached instance. There is no separate singleton API because sharing is the default.
- Asking `get()` for an interface that hasn't been bound to a concrete throws `DiException`. The same exception wraps any `ReflectionException` raised while resolving a graph.

## Documentation

Full documentation lives at [phpnomad.com](https://phpnomad.com). The bootstrapping guide shows how the container composes with `Bootstrapper` and initializers in a complete application, which is how most PHPNomad projects actually wire their dependency graph.

## License

MIT, see [LICENSE.txt](LICENSE.txt) for the full text.
