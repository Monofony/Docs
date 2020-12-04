# How to design entities with phpspec

Lets configure an Article entity with a title and an author.
Title is a simple string and author implements CustomerInterface.

> By default, phpspec on Monofony is configured with code coverage.
> [Learn how to configure phpspec with code coverage](how-to-configure-phpspec-with-code-coverage.md) or [disable code coverage](how-to-disable-phpspec-code-coverage.md).

Generate phpspec for your entity
--------------------------------

```bash
$ vendor/bin/phpspec describe App/Entity/Article

$ # with phpdbg installed
$ phpdbg -qrr vendor/bin/phpspec describe App/Entity/Article
```

`spec/src/App/Entity/Article.php`
```php
<?php

namespace spec\App\Entity;

use App\Entity\Article;
use PhpSpec\ObjectBehavior;
use Prophecy\Argument;

class ArticleSpec extends ObjectBehavior
{
    function it_is_initializable()
    {
        $this->shouldHaveType(Article::class);
    }
}
```

## Run phpspec and do not fear Red

To run phpspec for our Article entity, run this command:

```bash
$ vendor/bin/phpspec run spec/App/Entity/ArticleSpec.php -n
$
$ # with phpdbg installed
$ phpdbg -qrr vendor/bin/phpspec run spec/App/Entity/ArticleSpec.php -n
```

And be happy with your first error message with red color.

> You can simply run all the phpspec tests by running `vendor/bin/phpspec run -n`

## Create a minimal Article class

`src/App/Entity/Article.php`
```php
<?php

declare(strict_types=1);

namespace App\Entity;

class Article
{
}
```

Rerun phpspec and see a beautiful green color.

Specify it implements sylius resource interface
-----------------------------------------------

```php
function it_implements_sylius_resource_interface(): void
{
    $this->shouldImplement(ResourceInterface::class);
}
```

.. warning::

    And Rerun phpspec, DO NOT FEAR RED COLOR!
    It's important to check that you write code which solves your specifications.

Solve this on your entity
-------------------------

`src/App/Entity/Article.php`
```php
<?php

declare(strict_types=1);

namespace App\Entity;

use Sylius\Component\Resource\Model\ResourceInterface;

class Article implements ResourceInterface
{
    use IdentifiableTrait;
}
```

> Rerun phpspec again and check this specification is solved.

Specify title behaviours
------------------------

```php
function it_has_no_title_by_default(): void
{
    $this->getTitle()->shouldReturn(null);
}

function its_title_is_mutable(): void
{
    $this->setTitle('This documentation is so great');
    $this->getTitle()->shouldReturn('This documentation is so great');
}
```

> Don't forget to rerun phpspec on each step.

Add title on Article entity
---------------------------

`src/App/Entity/Article.php`
```php
/**
 * @var string|null
 */
private $title;

public function getTitle(): ?string
{
    return $this->title;
}

public function setTitle(?string $title): void
{
    $this->title = $title;
}
```

Specify author of the article
-----------------------------

`spec/src/App/Entity/Article.php`
```php
use Sylius\Component\Customer\Model\CustomerInterface;

// [...]

function its_author_is_mutable(CustomerInterface $author): void
{
    $this->setAuthor($author);
    $this->getAuthor()->shouldReturn($author);
}
```

## Add author on your entity

`src/App/Entity/Article.php`
```php
// [...]

/**
 * @var CustomerInterface|null
 */
private $author;

// [...]

public function getAuthor(): ?CustomerInterface
{
    return $this->author;
}

public function setAuthor(?CustomerInterface $author): void
{
    $this->author = $author;
}
```

That's all to design your first entity!
