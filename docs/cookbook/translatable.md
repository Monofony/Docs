# 9.2 - How to convert en entity to Translatable

We'll see how to transform a string property of an entity to a translatable string property.

## Basics : turn your entity to translatable {#basics}

Declare your future translation entity which will contain all the translations for every translatable field.

```yaml
# config/sylius/resources.yaml
sylius_resource:
    resources:
        app.book:
            classes:
                model: App\Entity\Book
            translation:
                classes:
                    model: App\Entity\BookTranslation
```

Let's localize the book property `title`.
- Create a Book translation class which will contain all the translatable fields

```php
// src/Entity/BookTranslation.php

declare(strict_types=1);

namespace App\Entity;

use App\Entity\IdentifiableTrait;
use Doctrine\ORM\Mapping as ORM;
use Sylius\Component\Resource\Model\AbstractTranslation;
use Sylius\Component\Resource\Model\ResourceInterface;

#[ORM\Entity]
#[ORM\Table(name: 'app_book_translation')]
class BookTranslation extends AbstractTranslation implements ResourceInterface
{
    use IdentifiableTrait;

    #[ORM\Column(type: 'string')]
    private string $title;

    public function getTitle(): string
    {
        return $this->title;
    }

    public function setTitle(string $title): void
    {
        $this->title = $title;
    }
}
```

And update your `Book` entity class so :

- it implements `TranslatableInterface` and uses `TranslatableTrait`
- the translatable property is not mapped in ORM anymore
- if using `api-pack`, the `read` serialization group is moved from the property to the getter

```php
// src/Entity/Book.php

declare(strict_types=1);

namespace App\Entity;

use App\Entity\BookTranslation;
use Sylius\Component\Resource\Model\TranslatableInterface;
use Sylius\Component\Resource\Model\TranslatableTrait;
use Sylius\Component\Resource\Model\TranslationInterface;

class Book implements ResourceInterface, TranslatableInterface
{
    use TranslatableTrait {
        __construct as private initializeTranslationsCollection;
    }

    public function __construct()
    {
        $this->initializeTranslationsCollection();
    }

    #[Assert\NotBlank]
    private string $title;

    #[Groups([
        'book:read',
    ])]
    public function getTitle(): string
    {
        return $this->getTranslation()->getTitle();
    }

    public function setTitle(string $title): void
    {
        $this->getTranslation()->setTitle($title);
    }

    /**
     * {@inheritdoc}
     */
    protected function createTranslation(): TranslationInterface
    {
        return new BookTranslation();
    }
}
```

The first entity, containing translations for the translatable fields, will automatically be linked to this base translatable entity.

### Create and tune your migrations

Create the migration file

````bash
$ bin/console doctrine:migrations:diff
````

If you do not want to lose data in the process, update it by adding some data migration with these `INSERT` and `UPDATE` SQL commands.
This way, you will keep the original translation of the targeted property. Choose your default locale accordingly in the following migration additions.

(this example is for PostgreSQL migration)

```php
     public function up(Schema $schema): void
     {
     ...
         $this->addSql('ALTER TABLE app_book_translation ADD CONSTRAINT FK_DD7AA6B52C2AC5D3 FOREIGN KEY (translatable_id) REFERENCES app_book (id) ON DELETE CASCADE NOT DEFERRABLE INITIALLY IMMEDIATE');
+        $this->addSql('INSERT INTO app_book_translation (SELECT nextval(\'app_book_translation_id_seq\'), id, title, \'fr_FR\' FROM app_book)');
         $this->addSql('ALTER TABLE app_book DROP title');
     }
     
     public function down(Schema $schema): void
     {
     ...
         $this->addSql('ALTER TABLE app_book ADD title VARCHAR(255) NOT NULL');
+        $this->addSql('UPDATE app_book SET title=(SELECT title FROM app_book_translation WHERE translatable_id=id and locale=\'fr_FR\')');
         $this->addSql('DROP TABLE app_book_translation');
     }
```
Be sure to always test both up and down migration to ensure no data will be lost in case of rollback of this feature.
Note also that other locale translations than the default one (here `'fr_FR'`) will be lost in the down migration process.

## Update your backend grid and form to manage translations {#backend-grid}

You need to use a specific repository method to join translations in your query. So let's begin with adding this repository.

```php
// src/Repository/BookRepository.php

declare(strict_types=1);

namespace App\Repository;

use Doctrine\ORM\QueryBuilder;
use Sylius\Bundle\ResourceBundle\Doctrine\ORM\EntityRepository;

class BookRepository extends EntityRepository
{
    public function createListQueryBuilder(string $locale): QueryBuilder
    {
        return $this->createQueryBuilder('o')
            ->innerJoin('o.translations', 'translation')
            ->andWhere('translation.locale = :locale')
            ->setParameter('locale', $locale)
        ;
    }
}
```
and declare it in resources.yaml
```yaml
# config/sylius/resources.yaml
sylius_resource:
    resources:
        app.book:
            classes:
                model: App\Entity\Book
+               repository: App\Repository\BookRepository
            ...
```
Then update the grid to view the translated `title` field.
```php
// src/Grid/BookGrid.php

    // [...]

    public function buildGrid(GridBuilderInterface $gridBuilder): void
    {
        $gridBuilder
            ->setRepositoryMethod('createListQueryBuilder', ['%locale%'])
            ->orderBy('translation.title', 'desc')
            ->addField(
                StringField::create('translation.title')
                    ->setLabel('app.ui.title')
                    ->setSortable(true)
            )
            ->addFilter(StringFilter::create('search', ['translation.title']))

        ...
```
