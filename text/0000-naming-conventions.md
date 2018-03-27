- Start Date: 2018-03-27
- RFC PR: (leave this empty)
- WordPress Coding Standards Issue: (leave this empty)

# Summary

This guide is intended to maximize consistency when writing code in the WordPress environment.
Naming your language constructs is important and should reflect the nature of what it does.

# Basic example

Naming variables, functions, classes and namespaces should be unified across the WordPress core. Themes and plugins are recommended to use the WordPress naming standards, although it's not required.

Variables should be clear and understandable, written in lowercase using underscore as a separator.

```php
$post_array;
```

Special PHP constants `true`, `false` and `null` should be written in lowercase, other PHP constants should follow the [PHP definition](http://php.net/manual/en/language.constants.php).

```php
define( 'DOING_AJAX', false );
```

The term `class` refers to all classes, interfaces, and traits.

Classes and namespaces should be written in capitalized words separated by underscores.

```php
namespace Theme_Name\Subnamespace;

class Some_Class implements Interface {
  // class implementation.
}
```

File names should be written using lowercase letters separated by hyphens. Class file names (interfaces and traits) must be based on the class name, with the underscores in the class name replaced by hyphens.

```php
content-gallery.php

some-class.php

example-interface.php
```

# Motivation

Motivation behind this proposal is setting a unified standard of naming language constructs in WordPress. This also helps with future proofing when using modern PHP practices in the WordPress core - that will probably happen once support for PHP 5.2 is dropped. Also having a clear and concise naming rules helps new contributors when writing patches, and it's good to have a unified naming when working within a software ecosystem.

The inspiration comes from:

https://www.alainschlesser.com/interface-naming-conventions/

https://make.wordpress.org/core/handbook/best-practices/coding-standards/php/#naming-conventions

https://www.php-fig.org/psr/psr-2/

# Detailed design

#### Keywords

PHP [keywords](http://php.net/manual/en/reserved.keywords.php) have special meaning in PHP. You cannot use their names as constants, class names, function or method names. Using them as variable names is generally OK, but could lead to confusion.

Some examples are


|  |    |  |  | |
|-------|--------|-------|-----|----------|
| die() | echo   | const | do  | continue |
| for   | exit() | else  | new | return   |



#### Constants

Special PHP constants `true`, `false` and `null` should be written in lowercase, other PHP constants should follow the [PHP definition](http://php.net/manual/en/language.constants.php).

```php
<?php
define( 'ENV', 'staging' ); // Works OUTSIDE of a class definition.

const ENV = 'staging'; // Works both OUTSIDE and INSIDE of a class definition since PHP 5.3.
```

#### Variables and function names

When naming variables, actions/filters and function names use `lowercase` letters separated by underscores.
Don't abbreviate or obfuscate the names, code should be unambiguous and self-documenting.

```php
<?php
$post_list = get_posts();

function get_authors_names_from_posts( $posts_array ) {
  foreach ( $posts_array as $post_object => $post ) {
      $user_info      = get_user_by( 'ID', $post->post_author );
      $author_names[] = $user_info->data->user_nicename;
  }

  return $author_names;
}

$authors_list = get_authors_names_from_posts( $post_list );
```

#### Namespace and use declarations

Namespaces should be written the same way the classes are - capitalized words separated by underscores.

```php
<?php
namespace Theme_Name\Subnamespace;

use Foo_Class;
use Bar_Class as Bar;
use Plugin_Name\Admin as Admin;

// ... additional PHP code ...
```

#### Classes

The term `class` refers to all classes, interfaces and traits.

Class names must be capitalized words separated by underscores. Acronyms should be all upper case.

```php
<?php
namespace Some_Name;

class Some_Class {
  // constants, properties, methods.
}
```

```php
<?php
namespace Some_Name;

class Particular_Class implements Thing {
  // constants, properties, methods.
}
```

When naming interfaces use the most meaningful name for the interface. Class name should be more qualified that provide additional information about the specific implementation of the interface it implements. For instance

```php
<?php
namespace Templates;

interface Template {
  // methods which class must implement.
}
```

Now we implement it in different classes

```php
<?php
namespace Templates;

class Post_Template implements Template {
  // Specific implementation for the posts.
}
```

```php
<?php
namespace Templates;

class Page_Template implements Template {
  // Specific implementation for the pages.
}
```

#### Properties

Properties should not be prefixed with a single underscore to indicate protected or private visibility.

```php
<?php
namespace Templates;

class Page_Template implements Template {
  public $template_name = 'page_template';
}
```

#### Methods

Methods should not be prefixed with a single underscore to indicate protected or private visibility.

```php
<?php
namespace Admin;

class Main {
  private $theme_slug = 'my_theme';

  public function __construct() {
    $this->theme_slug = $theme_slug;
  }

  public function get_theme_name() {
    return $this->theme_slug;
  }

  public function foo( $arg1, &$arg2, $arg3 = [] ) {
    // method body.
  }
}
```

#### File naming

Files must be named descriptively, using lowercase letters separated by hyphens. When writing functional code use short descriptive names

```php
content-gallery.php
```

Class file names must be based on the class name, with the underscores in the class name replaced by hyphens. There must be only one class in a class file. For example class `WP_Error` should reside in a

```php
wp-error.php
```

# Drawbacks

If the new file naming rule will only apply to the newly created files in the core, there really isn't any drawback.

# Alternatives

Only good alternative would be to implement parts of PSR-2 standards regarding naming, but they differ quite a bit from WordPress Coding Standards.

# Adoption strategy

For the time being, the core doesn't have to be rewritten, so nothing is broken. But this opens up a clear way for the developers on how these constructs should be implemented in a WordPress environment. That will also help later on if the core decides to implement a rewrite.

# How we teach this

Handbook examples should be made, not only with `foo` and `bar` code, but with real life examples.

PHP manual has a pretty good and elaborate [reference for namespaces](http://php.net/manual/en/language.namespaces.php), which can be provided to developers to get them acquainted with the terminology.

# Unresolved questions

Sniffs should be added in the [WPCS](https://github.com/WordPress-Coding-Standards/WordPress-Coding-Standards/) for PHP_CodeSniffer that will check rules about namespaces, interfaces and traits, but they can be modified from PSR ruleset.
