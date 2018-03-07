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

PHP is a [weakly typed language](https://en.wikipedia.org/wiki/Strong_and_weak_typing). It doesn't require you to declare data types. However, variables still have data types associated with them (e.g., `string`, `int`). In a weakly typed language you can do things like adding a `string` to an `int` with no error. While this can be wonderful in some cases, it can also lead to unanticipated behavior and bugs.

For example, PHP functions and class methods accept parameters, such as `function foo( $bar, $items )`. If Type Declarations aren't being used, the only way a function knows what it's being passed is to run `is_*()` checks, `instanceof`, or use type casting. Otherwise, `$bar` could be anything, and there's a chance that invalid data types would go unnoticed and produce an unexpected or incorrect return value. Imagine type casting an array to an integer. That's forcing a wrong into a right, instead of correcting the underlying issue. A caller should pass the right data type to begin with.

So Type Declarations are advantageous, because ultimately, they produce improved error messages that catch problems right away. In the same way we _discourage_ use of the `@` error control operator and _encourage_ strict comparison `===`, Type Declarations shine a light a bugs. They're a way of being more explicit about the expected data type. An added benefit is more readable code.

# Detailed Design

## Valid Type Declarations in PHP 5.1+

- Class or interface name; e.g., `WP_REST_Request`
- `self`, which references own class or interface name
- `array`, to require an array

```php
protected function foo( Bar $bar, array $items ) {
	// $bar   must be an instance of the Bar class.
	// $items must be passed as an array.
}
```

## Modern Versions of PHP

- The [`callable` Type Declaration](http://php.net/manual/en/functions.arguments.php#functions.arguments.type-declaration) became available in PHP 5.4.
- [Scalar Type Declarations](http://php.net/manual/en/migration70.new-features.php#migration70.new-features.scalar-type-declarations): `string`, `int`, `float`, and `bool` became available in PHP 7.0, along with support for [Return Type Declarations](http://php.net/manual/en/migration70.new-features.php#migration70.new-features.return-type-declarations) and [Strict Typing](http://php.net/manual/en/functions.arguments.php#functions.arguments.type-declaration.strict).
- The [`iterable` Type Declaration](http://php.net/manual/en/migration71.new-features.php#migration71.new-features.iterable-pseudo-type), [Nullable Types](http://php.net/manual/en/migration71.new-features.php#migration71.new-features.nullable-types), and [`void` Return Type Declaration](http://php.net/manual/en/migration71.new-features.php#migration71.new-features.void-functions) became available in PHP 7.1.
- The [`object` Type Declaration](http://php.net/manual/en/migration72.new-features.php#migration72.new-features.object-type) became available in PHP 7.2.

## Formatting

_**Note:** All of these formatting rules follow [PSR-12](https://github.com/php-fig/fig-standards/blob/master/proposed/extended-coding-style-guide.md#45-method-and-function-arguments), with one additional clarification: There MUST be one space before and after a Type Declaration; e.g., `( array $items`, not `(array$items`._

### Whitespace

**The additional PSR-12 clarification:**
There MUST be one space before and after a Type Declaration.

```php
protected function foo( Bar $bar, array $items ) {
	// ...
}
```

**Nullable Types:**
There MUST NOT be a space between the question mark and the Type Declaration.

```php
protected function foo( ?Bar $bar, ?array $items ) {
	// ...
}
```

#### Return Type Declarations

The colon and Type Declaration MUST be on the same line as a function's closing parentheses. There MUST NOT be spaces between the function's closing parentheses and colon. There MUST be one space after the colon, followed by the Type Declaration. There MUST be one space after the Type Declaration, before the function's opening curly bracket.

```php
protected function foo(): array {
	// ...
}
```

**Nullable Types:**
There MUST NOT be a space between the question mark and the Type Declaration.

```php
protected function foo(): ?array {
	// ...
}
```

### Required CaSe

The caSe of Type Declarations MUST be lowercase (e.g., `array`), except for class and interface names, which MUST follow the name as declared by the class or interface; e.g., `WP_REST_Request`

```php
protected function foo( array $data, WP_REST_Request $request ) {
	// ...
}
```

### Required Form

_**Note:** Use `int`, not `integer`. Use `bool`, not `boolean`._

The following are valid Type Declarations:

- Class or interface name; e.g., `WP_REST_Request`
- `self`, which references own class or interface name
- `array`
- `callable`
- `string`
- `int`
- `float`
- `bool`
- `iterable`
- `object`

## WordPress Core Compatibility

At this time, the additional Type Declarations: `callable`, `string`, `int`, `float`, `bool`, `iterable`, and `object` must be avoided in WordPress Core. The same is true for [Return Type Declarations](http://php.net/manual/en/migration70.new-features.php#migration70.new-features.return-type-declarations), [Strict Typing](http://php.net/manual/en/functions.arguments.php#functions.arguments.type-declaration.strict), and [Nullable Types](http://php.net/manual/en/migration71.new-features.php#migration71.new-features.nullable-types). Please see [WordPress requirements](https://wordpress.org/about/requirements/) (PHP 5.2.4+) and [this table](http://php.net/manual/en/functions.arguments.php#functions.arguments.type-declaration) for further details.

The only Type Declarations supported in WordPress Core at this time, are:

- Class or interface name; e.g., `WP_REST_Request`
- `self`, which references own class or interface name
- `array`, to require an array

_These can only be used in function parameters, not as Return Type Declarations._

## When to Use Type Declarations

When you're writing a function or a class method that expects to receive an array or a specific object type, and there is no reason to accept anything other than that specific type.

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

Type Declarations were once known as Type Hints in PHP 5. That name is no longer appropriate, because later versions of PHP added support for [Return Type Declarations](http://php.net/manual/en/migration70.new-features.php#migration70.new-features.return-type-declarations), [Strict Typing](http://php.net/manual/en/functions.arguments.php#functions.arguments.type-declaration.strict), and [Nullable Types](http://php.net/manual/en/migration71.new-features.php#migration71.new-features.nullable-types). For this reason, please use the up-to-date and official terminology:

- [Type Declarations](http://php.net/manual/en/functions.arguments.php#functions.arguments.type-declaration)
- [Scalar Type Declarations](http://php.net/manual/en/migration70.new-features.php#migration70.new-features.scalar-type-declarations)
- [Return Type Declarations](http://php.net/manual/en/migration70.new-features.php#migration70.new-features.return-type-declarations)
- [Strict Typing](http://php.net/manual/en/functions.arguments.php#functions.arguments.type-declaration.strict) (aka: Strict Mode)
- [Nullable Types](http://php.net/manual/en/migration71.new-features.php#migration71.new-features.nullable-types)

## Default Behavior & Strict Mode

[Strict Mode](http://php.net/manual/en/functions.arguments.php#functions.arguments.type-declaration.strict) is possible in PHP 7.0+. Strict Mode only impacts [Scalar Type Declarations](http://php.net/manual/en/migration70.new-features.php#migration70.new-features.scalar-type-declarations): `string`, `int`, `float`, and `bool`. This can be somewhat confusing, so it must be explained in some detail and the [official documentation](http://php.net/manual/en/functions.arguments.php#functions.arguments.type-declaration.strict) should be reviewed carefully. It's important to understand both the default behavior and also the impact Strict Mode has.

### Default Behavior

By default, PHP will, if possible, coerce scalar values of the wrong type into the expected type. For example, a function that is given an `int` for a parameter that expects a `string` will get a variable of type `string`. The `int` is magically converted into the `string` expected by a Scalar Type Declaration.

This is also true in scalar Return Type Declarations. By default, scalar return values will be coerced to the correct type if they are not already of that type.

_**Implication:** Because Strict Mode only impacts Scalar Type Declarations, when an invalid type is passed to a function using a non-scalar Type Declaration, it always produces a recoverable fatal error in PHP 5, or a [TypeError](http://php.net/manual/en/class.typeerror.php) in PHP 7. This is the default behavior and it cannot be altered, because Strict Mode has no impact on non-scalar Type Declarations._

### Enabling Strict Mode

Strict Mode can be enabled on a per-file basis using the `declare` directive:

```php
<?php
declare( strict_types=1 );
```

In Strict Mode, when both the function call and the function declaration are in a strict-typed file, only a variable of the exact type of the Scalar Type Declaration will be accepted. Otherwise, a [TypeError](http://php.net/manual/en/class.typeerror.php) will be thrown. The only exception to this rule is that an `int` may be given to a function expecting a `float`.

This is also true for Return Type Declarations. Whenever a function is defined in a strict-typed file, a scalar return value must be of the correct type, otherwise a [TypeError](http://php.net/manual/en/class.typeerror.php) will be thrown.

For further details, see: [Strict Typing](http://php.net/manual/en/functions.arguments.php#functions.arguments.type-declaration.strict)

# Unresolved Questions

- If sniffs are added for Type Declarations:
  - Are there any circumstances in which Type Declarations **MUST** be used?
  - Are there any circumstances in which Type Declarations **MUST NOT** be used?

- Should steps be taken to suppress runtime errors caused by invalid data types when running in a production environment? If so, what options are available for consideration?

  - In PHP 7.0+ it is possible to catch [TypeError](http://php.net/manual/en/class.typeerror.php) exceptions.
