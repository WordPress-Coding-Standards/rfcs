- Start Date: 2018-03-21
- RFC PR: (leave this empty)
- WordPress Coding Standards Issue: (leave this empty)

# Summary

This guide is intended to maximize consistency when writing code in the WordPress environment.
Naming your language constructs is important and should reflect the nature of what it does.

# Basic example

Naming variables, functions, classes, namespaces should be unified across the core, themes and plugins.

Variables should be clear and understandable, written in lowercase using hyphens as a separator.

```php
$post_array;
```

Classes (namespaces, traits, interfaces, abstract classes and classes) should be written in capitalized words separated by underscores.

```php
namespace Theme_Name\Subnamespace;

class Class_Name implements Interface {
  // class implementation.
}
```

PHP constants should be written in lowercase, while user defined constants should be written in uppercase separated by underscores.

```php
define( 'DOING_AJAX', false );
```

File names should be written using lowercase letters separated by hyphens, and class file names must be based on the class name, with the underscores in the class name replaced by hyphens.

```php
content-gallery.php

class-name.php
```

# Motivation

The motivation behind this is future proofing the WordPress code. Also the current status of WordPress core is a mess. There are mixed OOP constructs with functional programming, all in files that have various namings. At some point in the future PHP 5.2 will be deprecated in the WordPres core. This will enable a gradual, but necessary, rewrite of the WordPress core using the proper OOP principles. With that upgrade come all the modern PHP language constructs (namespaces, interfaces, autoloading etc.). The sooner we have an agreement around the proper naming, the better we'll be prepared to handle the possible rewrite. And it will help with the current [core autoloader proposal](https://core.trac.wordpress.org/ticket/36335).

Once the naming proposal is accepted and unified, a gradual core rewrite can begin that follows the unified rule.

The inspiration comes from:

https://www.alainschlesser.com/interface-naming-conventions/
https://www.php-fig.org/psr/psr-2/

# Detailed design

#### Keywords and constants

PHP [keywords](http://php.net/manual/en/reserved.keywords.php) must be in lower case, as well as PHP constants `true`, `false` and `null`. User defined constants must be in all upper-case with underscores separating words

```php
<?php
define( 'ENV', 'staging' );
```

#### Namespace and use declarations

Namespace must be defined at the top of the file. There must be one blank line after the `namespace` declaration. Namespaces must follow the class names naming - capitalized words separated by underscores.

When present, all `use` declarations must go after the `namespace` declaration. There must be one use keyword per declaration, and there must be one blank line after the `use` block.

```php
<?php
namespace Theme_Name\Subnamespace;

use Foo_Class;
use Bar_Class as Bar;
use Plugin_Name\Admin as Admin;

// ... additional PHP code ...
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

#### Classes

The term 'class' refers to all classes, interfaces and traits.

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

The `extends` and `implements` keywords must be declared on the same line as the class name. The opening brace for the class must go in the same line as the class name.

#### Properties

Visibility must be declared on all properties. Properties should not be prefiexed with a single underscore to indicate protected or private visibility.

```php
<?php
namespace Templates;

class Page_Template implements Template {
  public $template_name = 'page_template';
}
```

#### Methods

Visibility must be declared on all methods. Methods should not be prefiexed with a single underscore to indicate protected or private visibility.

Method names must not be declared with a space after the method name. The opening brace must go on the same line as the method declaration, and the closing brace must go on the next line following the body.

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

The method arguments should follow liberal spacing rules set by the [WordPress Coding Standards](https://make.wordpress.org/core/handbook/best-practices/coding-standards/php/#space-usage).

#### abstract, static and final

When present, `abstract` and `final` declarations must precede the visibility declaration.

When present, `static` declaration must come after the visibility declaration.

```php
<?php
namespace Namespace_Name;

abstract class Class_Name {

  protected static $property;

  abstract protected function foo();

  final public static function bar() {
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

If the core will be rewritten, the time it takes to implement this rule will be significant, plus we must wait for the core to drop the support for the PHP 5.2 which should have happened a long time ago (EOL for PHP 5.2 was 6 Jan 2011 according to [this table](http://php.net/eol.php)). Even PHP 5.6 will have security support until 31 Dec 2018 (according to [this table](http://php.net/supported-versions.php)).

Another thing we should consider is file structure re organization so that it aligns with the OOP principles, but such that it doesn't break userspace. Which is also a pretty big task.

The impact could be big, since we would show developers that they can use modern (?) way of writing PHP.

# Alternatives

Only good alternative would be to implement parts of PSR-2 standards, and PSR-4 standard for autoloading. The **major** issue here would be that we would probably break userspace, since core woould have to rewrite every existing class, and that's not good - imagine updating WordPress to find that `WP_Query` isn't working and that you should change it to `WPQuery`. With probably hundreds of thousands themes and plugins out there, that road is basically closed.

But rewriting core files (filenames as well as adding namespaces) shouldn't have that big impact on the existing code out there if we follow the current naming scheme.

# Adoption strategy

For the time being, the core doesn't have to be rewritten, so nothing is broken. But this opens up a clear way for the developers on how these constructs should be implemented in a WordPress environment. That will also help later on if the core decides to implement a rewrite.

# How we teach this

PHP manual has a pretty good and elaborate [reference for namespaces](http://php.net/manual/en/language.namespaces.php), which can be provided to developers to get them acquainted with the terminology.

Additionaly, a handbook examples should be made, not only with `foo` and `bar` code, but with real life examples.

# Unresolved questions

Sniffs should be added in the [WPCS](https://github.com/WordPress-Coding-Standards/WordPress-Coding-Standards/) for PHP_CodeSniffer that will check rules about namespaces, but they can be modified from PSR ruleset.

Should we cover all the constructs in this proposal (control structures, closures etc.), similar to what [PSR-2](https://www.php-fig.org/psr/psr-2/) is doing, or just stick to the naming of the basic constructs?
