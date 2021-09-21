# How to design entities with phpspec

Let's configure a Book entity with a name and an author.
Name is a simple string and author is an Author resource.

## Generate phpspec for your entity {#generate-your-entity}

```bash
$ vendor/bin/phpspec describe App/Entity/Book/Book
```

It will generate this new file `spec/App/Entity/Book/BookSpec.php`.

```php
<?php

namespace spec\App\Entity\Book;

use App\Entity\Book\Book;
use PhpSpec\ObjectBehavior;

class BookSpec extends ObjectBehavior
{
    function it_is_initializable()
    {
        $this->shouldHaveType(Book::class);
    }
}
```

## Run phpspec and do not fear Red {#do-not-fear-red}

To run phpspec for our Book entity, run this command:

```bash
$ vendor/bin/phpspec run spec/App/Entity/Book/BookSpec.php -n
```

And be happy with your first error message:

```shell
App/Entity/Book/Book                                                            
  10  - it is initializable
      class App\Entity\Book\Book does not exist.

                                      100%                                       1
1 specs
1 example (1 broken)
7ms
```

<div class="block-note">
    You can simply run all the phpspec tests by running `vendor/bin/phpspec run -n`
</div>

## Create a minimal Book class {#minimal-book-class}

`src/App/Entity/Book/Book.php`
```php
<?php

declare(strict_types=1);

namespace App\Entity\Book;

class Book
{
}
```

Rerun phpspec and see a beautiful green color:

```shell
                                      100%                                       1
1 specs
1 example (1 passed)
7ms
```

## Specify it implements Sylius resource interface {#implements-resource-interface}

```php
<?php

# Import the ResourceInterface from Sylius Resource component.
use Sylius\Component\Resource\Model\ResourceInterface;

class BookSpec extends ObjectBehavior
{
    ## Add this new method
    function it_is_a_resource(): void
    {
        $this->shouldImplement(ResourceInterface::class);
    }
}
```

<div class="block-warning">
    And Rerun phpspec, DO NOT FEAR RED COLOR!
    It's important to check that your new code solves your specifications.
</div>

```shell
App/Entity/Book/Book                                                              
  16  - it implements sylius resource interface
      expected an instance of Sylius\Component\Resource\Model\ResourceInterface, but got
      [obj:App\Entity\Book\Book].

                  50%                                     50%                    2
1 specs
2 examples (1 passed, 1 failed)
9ms
```

## Solve this on your entity {#solve-it}

`src/App/Entity/Book/Book.php`
```php
<?php

declare(strict_types=1);

namespace App\Entity\Book;

use App\Entity\IdentifiableTrait;
use Sylius\Component\Resource\Model\ResourceInterface;

class Book implements ResourceInterface
{
    use IdentifiableTrait;
}
```

<div class="block-note">
    Rerun phpspec again and check this specification is solved.
</div>

The Identifiable Trait is yours. This trait has been copied with Symfony Flex during the installation.
By default, this use an auto-incremented identifier, But you can customize it as you want. 
ResourceInterface does not need an integer as identifier, thus you can use another system to generate ids.

## Specify title behaviours {#specify-title}

```php
function it_has_no_name_by_default(): void
{
    $this->getName()->shouldReturn(null);
}

function its_name_is_mutable(): void
{
    $this->setName('Shining');
    
    $this->getName()->shouldReturn('Shining');
}
```

<div class="block-note">
Don't forget to rerun phpspec on each step.
</div>

```shell
App/Entity/Book/Book                                                              
  21  - it has no name by default
      method App\Entity\Book\Book::getName not found.

App/Entity/Book/Book                                                            
  26  - its name is mutable
      method App\Entity\Book\Book::setName not found.

                  50%                                     50%                    4
1 specs
4 examples (2 passed, 2 broken)
9ms
```

## Add name on Book entity {#add-name}

`src/App/Entity/Book/Book.php`
```php
private ?string $name = null;

public function getName(): ?string
{
    return $this->name;
}

public function setName(?string $name): void
{
    $this->name = $name;
}
```

## Create the Author resource {#create-author-resource}

Create a first name and a last name on your Author resource. 
Do not forget to implements Sylius Resource interface.

Do not copy and paste the whole `spec/Entity/Book/BookSpec.php` file.
Repeat the previous steps to describe your Author resource and running Phpspec each time with a baby step.
You can copy and adapt one method for each baby step from the previous file. 
And that's why you need to rerun to ensure the test is not implemented yet (error during copy and paste step).

Do it yourself as an entertainment!

At the end the result should be:
```php
<?php

namespace spec\App\Entity\Author;

use App\Entity\Author\Author;
use PhpSpec\ObjectBehavior;
use Sylius\Component\Resource\Model\ResourceInterface;

class AuthorSpec extends ObjectBehavior
{
    function it_is_initializable(): void
    {
        $this->shouldHaveType(Author::class);
    }

    function it_is_a_resource(): void
    {
        $this->shouldImplement(ResourceInterface::class);
    }

    function it_has_no_first_name_by_default(): void
    {
        $this->getFirstName()->shouldReturn(null);
    }

    function its_first_name_is_mutable(): void
    {
        $this->setFirstName('Stephen');

        $this->getFirstName()->shouldReturn('Stephen');
    }

    function it_has_no_last_name_by_default(): void
    {
        $this->getLastName()->shouldReturn(null);
    }

    function its_last_name_is_mutable(): void
    {
        $this->setLastName('King');

        $this->getLastName()->shouldReturn('King');
    }
}
```

```php
<?php

declare(strict_types=1);

namespace App\Entity\Author;

use App\Entity\IdentifiableTrait;
use Sylius\Component\Resource\Model\ResourceInterface;

class Author implements ResourceInterface
{
    use IdentifiableTrait;

    private ?string $firstName = null;
    private ?string $lastName = null;

    public function getFirstName(): ?string
    {
        return $this->firstName;
    }

    public function setFirstName(?string $firstName): void
    {
        $this->firstName = $firstName;
    }

    public function getLastName(): ?string
    {
        return $this->lastName;
    }

    public function setLastName(?string $lastName): void
    {
        $this->lastName = $lastName;
    }
}
```

## Specify author of the book {#specify-book-author}

`spec/src/App/Entity/Book/BookSpec.php`
```php
use Sylius\Component\Customer\Model\CustomerInterface;

// [...]

function its_author_is_mutable(CustomerInterface $author): void
{
    $this->setAuthor($author);
    $this->getAuthor()->shouldReturn($author);
}
```

## Add author on your entity {#add-author}

`src/App/Entity/Book/Book.php`
```php
// [...]

private ?CustomerInterface $author = null;

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
