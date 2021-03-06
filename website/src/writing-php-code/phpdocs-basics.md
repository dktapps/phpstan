---
title: PHPDocs Basics
---

PHPDocs are a big part of what makes PHPStan work. PHP in its most recent versions can express a lot of things in the native typehints, but it still leaves a lot of room for PHPDocs to augment the information.

Valid PHPDocs start with `/**`. Variants starting only with `/*` or line comments `//` are not considered PHPDocs. [^phpdocs]

[^phpdocs]: Only the `/**` style comments are supported because they're represented [with different tokens](https://www.php.net/manual/en/tokens.php) (`T_DOC_COMMENT`) by the PHP parser and only this token type is supposed to represent a PHPDoc.

Methods and functions
----------

This is how a valid PHPDoc above a function or a method can look like:

```php
/**
 * @param Foo $param
 * @return Bar
 */
function foo ($param) { ... }
```

Properties
----------

PHPDocs can be written above class properties to denote their type:

```php
/**
 * @var Foo
 */
private $bar;
```

Inline @var
------------

Casting a type using an inline `@var` PHPDocs should be used only as a last resort. It can be used in a variable assignment like this:

```php
/** @var Foo $foo */
$foo = createFoo();
```

This is usually done if the symbol on the right side of the `=` operator has a wrong type, or isn't specific enough. Usage of inline `@var` is problematic for a couple of reasons:

1) Because it might be used to correct wrong type information of the called symbol, PHPStan always trusts it. But if there's a mistake in this annotation, the analysis of the code below might be wrong.
2) The inline `@var` needs to be repeated above all usages of the symbol which leads to repetition in the codebase.

Instead, the type should be fixed at its source. If the called symbol comes from 3rd party code, you can correct it using a [stub file](/user-guide/stub-files). If the return type differs in each call based on the passed arguments, you can use [generics](/blog/generics-in-php-using-phpdocs), or write a [dynamic return type extension](/developing-extensions/dynamic-return-type-extensions) instead.

If you really need to use an inline `@var`, consider an alternative - an `assert()` call, which can throw an exception at runtime, so the code execution doesn't continue if the requirements aren't met.

```php
$foo = createFoo();
assert($foo instanceof Foo);
```

Magic properties
-------------

For custom `__get`/`__set` methods logic, a `@property` PHPDoc tag can be placed above a class. If the property is only supposed to be read or written to, `@property-read`/`@property-write` variants can be used.

```php
/**
 * @property int $foo
 * @property-read string $bar
 * @property-write \stdClass $baz
 */
class Foo { ... }
```

The `@property` tag can also be used to override wrong property type from a parent class.

<!-- link Solving undefined properties -->

Magic methods
-------------

For custom `__call` methods logic, a `@method` PHPDoc tag can be placed above a class:

```php
/**
 * @method int computeSum(int $a, int $b)
 * @method void doSomething()
 * @method static int staticMethod()
 */
class Foo { ... }
```

Combining PHPDoc types with native typehints
-------------

PHPDocs can also complement native typehints with additional information. The most common use-case is telling PHPStan what's in an array:

```php
/**
 * @param User[] $users
 */
function foo(array $users) { ... }
```

You can also use an alternative `array<User>` syntax, or even specify the key type:

```php
/**
 * @param array<int, User> $users
 */
function foo(array $users) { ... }
```

More about this in [PHPDoc Types > Iterables](/writing-php-code/phpdoc-types#iterables).

Using `@return static` along with the `self` native typehint means that the method returns a child class ([see example](/r/5f856517-5303-4237-95de-2bfa5bc4b9de)):

```php
/**
 * @return static
 */
public function returnStatic(): self
{
    return $this;
}
```

A narrower `@return $this` instead of `@return static` can also be used, and PHPStan will check if you're really returning the same object instance and not just the child class.

Generics
---------------

PHPDoc tags `@template`, `@template-covariant`, `@extends`, and `@implements` are reserved for generics. [Learn more about generics »](/blog/generics-in-php-using-phpdocs)

Prefixed tags
---------------

Supported tags (`@var`, `@param`, `@return`, and all [generics-related ones](/blog/generics-in-php-using-phpdocs)) can be prefixed with `@phpstan-`:

```php
/**
 * @phpstan-param Foo $param
 * @phpstan-return Bar
 */
function foo ($param) { ... }
```

This is useful in the context of advanced types and [generics](/blog/generics-in-php-using-phpdocs). IDEs and other PHP tools might not understand the advanced types that PHPStan takes advantage of. So you can leave the ordinary `@param` in the PHPDoc and add a `@phpstan-param` with an advanced type syntax.
