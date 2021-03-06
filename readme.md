
# Laravel Factory Enhanced 🔥

[![Latest Version on Packagist](https://img.shields.io/packagist/v/makeabledk/laravel-factory-enhanced.svg?style=flat-square)](https://packagist.org/packages/makeabledk/laravel-factory-enhanced)
[![Build Status](https://img.shields.io/travis/makeabledk/laravel-factory-enhanced/master.svg?style=flat-square)](https://travis-ci.org/makeabledk/laravel-factory-enhanced)
[![StyleCI](https://styleci.io/repos/117680722/shield?branch=master)](https://styleci.io/repos/117680722)

Bring the magic of eloquent relationships into the Laravel Factory. 

Traditionally if you wanted to factory a team with some users, you'd have to manually create the individual team and users and then tie them together afterwards. This can easily lead to very verbose tests.

**Before**

```php
$team = factory(Team::class)->create();
$user = factory(User::class)->create();
$member = factory(TeamMember::class)->create([
    'team_id' => $team,
    'user_id' => $user,
]);
```

**After**

```php
$team = factory(Team::class)->with(1, 'users')->create();
```

How about 10 teams each with 3 users?

```php
$team = factory(Team::class)->with(3, 'users')->times(10)->create();
```

## Installation

You may install the package via composer

```bash
composer require makeabledk/laravel-factory-enhanced
```

Installing on Laravel 5.5+ will automatically load the package service provider.

## Usage

Once the package is installed, you may use the `factory()` helper exactly as you normally would. Only now you have additional functionality at your disposal.

If you're not familiar with Laravel factories, please refer to the official documentation: https://laravel.com/docs/master/database-testing

### Simple relationships

You may use the enhanced factory to create any additional relationships defined on the model.

Example: A `Server` model with a `sites()` relationship (has-many)

```php
factory(Server::class)->with(3, 'sites')->create();
```

Note that you may still use any native functionality that you are used to, such as states and attributes:

```php
factory(Server::class)->states('online')->with(3, 'sites')->create([
    'name' => 'production-1'
]);
```

### Multiple relationships

You are free to specify multiple relationships on the same factory. 

Given our previous `Server` model has another relationship called `team` (belongs-to), you may do:

```php
factory(Server::class)
    ->with('team')
    ->with(3, 'sites')
    ->create();
```

Now you would have `1 team` that has `1 server` which has `3 sites`.


### Nested relationships

Moving on to the more advanced use-cases, you may also do nested relationships. 

For instance we could rewrite our example from before using nesting:

```php
factory(Team::class)
    ->with(3, 'servers.sites')
    ->create();
```
Please note that the count '3' applies to *the final relationship*, in this case `sites`.

If you are wanting 2 servers each of which has 3 sites, you may write it as following:

```php
factory(Team::class)    
    ->with(2, 'servers')
    ->with(3, 'servers.sites')
    ->create();
```

### States in relationships

Just as you may specify pre-defined states on the factoring model ([see official documentation](https://laravel.com/docs/master/database-testing#factory-states)), you may also apply the very same states on the relation.

```php
factory(Team::class)    
    ->with(2, 'online', 'servers')
    ->create();
```

You may finding yourself wanting a relationship in multiple states. In this case you may use the `andWith()` method.

```php
factory(Team::class)    
    ->with(2, 'online', 'servers')
    ->andWith(1, 'offline', 'servers')
    ->create();
```

By using the `andWith` we will create a 'clean cut', so that no further calls to `with` can interfere with relations specified prior to the `andWith`. 

In the above example any further nesting of relations will apply to the 'offline' server.

```php
factory(Team::class)    
    ->with(2, 'online', 'servers')
    ->andWith(1, 'offline', 'servers')
    ->with(3, 'servers.sites')
    ->create();
```

The above example will create 1x team that has

- 2x online servers
- 1x offline servers with 3 sites

### Using closures for customization

In addition to passing *count* and *state* directly into the `with` function, you may also pass a closure that will receive the `FactoryBuilder` instance directly.

```php
factory(Team::class)    
    ->with('servers', function (FactoryBuilder $servers) {
        $servers
            ->times(2)
            ->states('active')
    })
    ->create();
```

You may also nest further relationships in the builder should you wish to.

One advantage by using closures, is that it will be evaluated each time the factory-model (in this case a `server`) is created.
This is especially handy if you'd like some random relations in your `DevelopmentSeeder`.

```php
factory(Team::class)    
    ->with('servers', function (FactoryBuilder $servers) {
        $servers
            ->times(random_int(1, 5))
            ->states(array_random(['online', 'offline']))
    })
    ->create();
```

## Examples

### Creating random translations

For seeding an entire development environment it can be useful with random relations. 

Consider the following example from a real-life project:

```php
$courses = Collection::times(5)->map(function () {
    return factory(Course::class)
        ->with(random_int(1, 5), 'chapters')
        ->with(random_int(1, 5), 'chapters.videos')
        ->when(random_int(0, 2), function ($course) { // 66% of courses will be translated  
            $course
                ->with(1, 'danish', 'translations')
                ->with(random_int(1, 5), 'translations.chapters')
                ->with(random_int(1, 5), 'danish', 'translations.chapters.videos')
        })
    });
});
```

With a few lines of code we have seeded several models in several random states. Pretty neat!

### Factoring models with no definitions

Traditionally trying to use `factory()` on a model with no defined model-factory would throw an exception. Not anymore!

After installing this package, you are completely free to use `factory()` on any Eloquent model, whether or not you have defined a model factory.

If need be, you may pass attributes inline through the `with` method or use the `fill` helper.

**Passing attributes inline**

```php
factory(Server::class)->with(1, 'sites', ['name' => 'example.org'])->create();
```

**Using the `fill` method**

```php
factory(Server::class)->with(1, 'sites', function (FactoryBuilder $sites) {
    $sites->fill(function (Faker $faker) { 
        return [
            'name' => $faker->name,
        ];
    });
})->create();
```

## Available methods

These are the provided methods on the `FactoryBuilder` instance in addition to the core methods.

- fill
->fillPivot (only applicable on BelongsToMany
- when
- with
- andWith

## Word of caution

This package swaps the native factory implementation with it's own implementation. 

While the public API is aimed to be 100% compatible with the native one, some differences may occur. In that case we're happy to hear from you.

## Change log

Please see [CHANGELOG](CHANGELOG.md) for more information what has changed recently.

## Testing

You can run the tests with:

```bash
composer test
```

## Contributing

We are happy to receive pull requests for additional functionality. Please see [CONTRIBUTING](CONTRIBUTING.md) for details.

## Credits

- [Rasmus Christoffer Nielsen](https://github.com/rasmuscnielsen)
- [All Contributors](../../contributors)

## License

Attribution-ShareAlike 4.0 International. Please see [License File](LICENSE.md) for more information.
