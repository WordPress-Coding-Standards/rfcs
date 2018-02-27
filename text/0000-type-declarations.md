- Start Date: 2018-02-21
- RFC PR:
- WordPress Coding Standards Issue:

# Summary

[Type Declarations](http://php.net/manual/en/functions.arguments.php#functions.arguments.type-declaration) make code more readable. They also protect code, because Type Declarations are a way of enforcing specific data types in parameters passed to a function.

# Basic Example

You can force callers to pass an `array`, or a specific type of object, such as a `WP_REST_Request`. If a caller doesn't pass the data type expected, PHP 5 triggers a recoverable fatal error and PHP 7 throws a [TypeError](http://php.net/manual/en/class.typeerror.php) exception. This behavior catches problems right away.

```php
protected function foo( Bar $bar, array $items ) {
  // $bar   must be an instance of the Bar class.
  // $items must be passed as an array.
}
```

# Motivation

PHP is a [weakly typed language](https://en.wikipedia.org/wiki/Strong_and_weak_typing). It doesn't require you to declare data types. However, variables still have data types associated with them (e.g., `string`, `integer`). In a weakly typed language you can do things like adding a `string` to an `integer` with no error. While this can be wonderful in some cases, it can also lead to unanticipated behavior and bugs.

For example, PHP functions and class methods accept parameters, such as `function foo( $bar, $items )`. If Type Declarations aren't being used, the only way a function knows what it's being passed is to run `is_*()` checks, `instanceof`, or use type casting. Otherwise, `$bar` could be anything, and there's a chance that invalid data types would go unnoticed (no error) and produce an unexpected or incorrect return value. Imagine type casting an array to an integer. That's forcing a wrong into a right, instead of correcting the underlying issue. A caller should pass the right data type to begin with.

So Type Declarations are advantageous, because ultimately, they produce improved error messages that catch problems right away. In the same way we _discourage_ use of the `@` error control operator and _encourage_ strict comparison `===`, Type Declarations shine a light a bugs. They're a way of being more explicit about the expected data type. An added benefit is more readable code.

# Detailed Design

## Valid Type Declarations in PHP 5.1+

- Class or interface name; e.g., `WP_REST_Request`
- `self`, which references own class or interface name
- `array`, to require an array

**Formatting:** There should be one space before and after a Type Declaration.

```php
protected function foo( Bar $bar, array $items ) {
  // $bar   must be an instance of the Bar class.
  // $items must be passed as an array.
}
```

## Modern Versions of PHP

- The [`callable` Type Declaration](http://php.net/manual/en/functions.arguments.php#functions.arguments.type-declaration) became available in PHP 5.4.
- [Scalar Type Declarations](http://php.net/manual/en/migration70.new-features.php#migration70.new-features.scalar-type-declarations): `string`, `int`, `float`, and `bool` became available in PHP 7.0, along with support for [Return Type Declarations](http://php.net/manual/en/migration70.new-features.php#migration70.new-features.return-type-declarations) and [Strict Typing](http://php.net/manual/en/migration70.new-features.php#migration70.new-features.scalar-type-declarations).
- The [`iterable` Type Declaration](http://php.net/manual/en/migration71.new-features.php#migration71.new-features.iterable-pseudo-type), [Nullable Types](http://php.net/manual/en/migration71.new-features.php#migration71.new-features.nullable-types), and [Void Functions](http://php.net/manual/en/migration71.new-features.php#migration71.new-features.void-functions) became available in PHP 7.1.
- The [`object` Type Declaration](http://php.net/manual/en/migration72.new-features.php#migration72.new-features.object-type) became available in PHP 7.2.

**Formatting:** A Return Type Declaration should immediately follow a function's round closing bracket `)`, with no space before `:`, and with one space before and after the data type.

```php
protected function foo( string $str, int $num, bool $flag, callable $callback ): bool {
  // $str      must be passed as a string.
  // $num      must be passed as an integer.
  // $flag     must be passed as true or false.
  // $callback must be a callable function, method, closure.

  return true; // must return true or false.
}
```

## WordPress Core Compatibility

At this time, the additional Type Declarations: `callable`, `string`, `int`, `float`, `bool`, `iterable`, and `object` must be avoided in WordPress Core. The same is true for [Return Type Declarations](http://php.net/manual/en/migration70.new-features.php#migration70.new-features.return-type-declarations), [Strict Typing](http://php.net/manual/en/migration70.new-features.php#migration70.new-features.scalar-type-declarations), and [Void Functions](http://php.net/manual/en/migration71.new-features.php#migration71.new-features.void-functions). Please see [WordPress requirements](https://wordpress.org/about/requirements/) (PHP 5.2.4+) and [this table](http://php.net/manual/en/functions.arguments.php#functions.arguments.type-declaration) for further details.

## When to Use Type Declarations

When you're writing a function or a class method that expects to receive an array or a specific object type, and there is no reason to accept anything other than that specific data type.

```php
protected function foo( Bar $bar, array $items ) {
  // $bar   must be an instance of the Bar class.
  // $items must be passed as an array.
}
```

## When Not to Use Type Declarations

If the function you're writing is part of a public API and enforcing a specific data type would make the function less flexible in the eyes of a caller. For example, the [`get_post()`](https://developer.wordpress.org/reference/functions/get_post/) function in WordPress core accepts `int|WP_Post|null` as the first parameter. This maximizes flexibility for callers, making the function more convenient in a variety of circumstances.

Likewise, if you're writing a public function for an API and it needs a `WP_Post` instance, it's better not to enforce `WP_Post` with a type declaration. Instead, use the `get_post()` function to resolve the `$post` reference, making it possible for a caller to pass `int|WP_Post|null` to your function as well.

```php
function foo( $post ) {
  $post = get_post( $post );
  ...
}
```

# Drawbacks

## Runtime Errors

Using Type Declarations can lead to recoverable fatal errors in PHP 5, and [TypeError](http://php.net/manual/en/class.typeerror.php) exceptions in PHP 7. This is both a blessing and a curse. Runtime errors are mostly advantageous for reasons already stated elsewhere in this RFC; i.e., they help catch bugs right away.

However, unlike [`_doing_it_wrong()`](https://developer.wordpress.org/reference/functions/_doing_it_wrong/), errors associated with invalid data types are more difficult to suppress. Imagine a plugin author writing code that calls upon a core function. If the core function uses Type Declarations to enforce specific data types, and the plugin passes an invalid type, an HTTP error response or WSOD could occur.

# Adoption Strategy

Retrofitting existing functions with Type Declarations is generally discouraged. Doing so would suddenly enforce a rule that did not exist prior, causing runtime errors. For example, imagine themes and plugins calling core functions that previously accepted multiple data types, but now require a specific type. Not good! This scenario should be avoided.

However, new functions (particularly protected and private methods of a class) are encouraged to use Type Declarations supported by the minimum version of PHP they are targeting. If targeting a modern version of PHP (7.0+), the use of [Return Type Declarations](http://php.net/manual/en/functions.returning-values.php#functions.returning-values.type-declaration) is encouraged also.

# Teaching Strategy

Type Declarations were once known as Type Hints in PHP 5. That name is no longer appropriate, because later versions of PHP added support for [Return Type Declarations](http://php.net/manual/en/migration70.new-features.php#migration70.new-features.return-type-declarations), [Strict Typing](http://php.net/manual/en/migration70.new-features.php#migration70.new-features.scalar-type-declarations), and [Void Functions](http://php.net/manual/en/migration71.new-features.php#migration71.new-features.void-functions). For this reason, please use the up-to-date and official terminology:

- [Type Declarations](http://php.net/manual/en/functions.arguments.php#functions.arguments.type-declaration)
- [Return Type Declarations](http://php.net/manual/en/functions.returning-values.php#functions.returning-values.type-declaration)
- [Strict Typing](http://php.net/manual/en/functions.arguments.php#functions.arguments.type-declaration.strict) (aka: Strict Mode)
- [Void Functions](http://php.net/manual/en/migration71.new-features.php#migration71.new-features.void-functions)

## Limitations in WordPress Core

Given current minimum requirements in WordPress Core (PHP 5.2.4+), the additional Type Declarations: `callable`, `string`, `int`, `float`, `bool`, `iterable`, and `object` must be avoided altogether, and only mentioned for the purpose of explaining why they cannot be used at this time. The same is true for [Return Type Declarations](http://php.net/manual/en/migration70.new-features.php#migration70.new-features.return-type-declarations), [Strict Typing](http://php.net/manual/en/migration70.new-features.php#migration70.new-features.scalar-type-declarations), and [Void Functions](http://php.net/manual/en/migration71.new-features.php#migration71.new-features.void-functions) — these cannot be used in WordPress Core.

The only Type Declarations supported in WordPress Core at this time, are:

- Class or interface name; e.g., `WP_REST_Request`
- `self`, which references own class or interface name
- `array`, to require an array

# Unresolved Questions

- If sniffs are added for Type Declarations:
  - Are there any circumstances in which Type Declarations **MUST** be used?
  - Are there any circumstances in which Type Declarations **MUST NOT** be used?

- Should steps be taken to suppress runtime errors caused by invalid data types when running in a production environment? If so, what options are available for consideration?
