# 9.1 - Basics

## Update your composer configuration

We'll need Doctrine extensions
```bash
composer require stof/doctrine-extensions-bundle
```

## Configure new package

```yaml
stof_doctrine_extensions:
  default_locale: '%kernel.default_locale%'
  orm:
    default:
    sortable: true
```

# 9.2 - Configure your entity

## Add new property in entity model

```php
# src/Entity/Book.php
    #[ORM\Column(type: 'integer', options: ['default' => 0])]
    #[SortablePosition]
    private int $position = 0;

    public function getPosition(): int
    {
        return $this->position;
    }

    public function setPosition(int $position): void
    {
        $this->position = $position;
    }
```

Do not forget to generate migration for the new entity property.

```bash
$ bin/console doctrine:migration:diff
```

# 9.3 - Bringing the sorting ability to backend grid

## Add field to entity grid

```php
# src/Grid/BookGrid.php
$gridBuilder
    ->orderBy('position', 'asc')
    ->addField(
        TwigField::create('position', 'backend/book/grid/field/position_with_buttons.html.twig')
            ->setLabel('sylius.ui.position')
            ->setPath('.')
            ->setSortable(true),
    )
```

## Add new grid field template

```html
<!-- template/backend/book/grid/field/position_with_buttons.html.twig -->
<div style="text-align: center;"><span class="ui circular label">{{ data.position }}</span></div>

<div class="ui vertical menu">
    <div class="item button app-book-move-up" data-url="{{ path('app_backend_ajax_book_move', { id: data.id }) }}" data-id="{{ data.id }}" data-position="{{ data.position }}">
        <i class="arrow up icon grey"></i>{{ 'sylius.ui.move_up'|trans }}
    </div>
    {% if data.position > 0 %}
    <div class="item button app-book-move-down" data-url="{{ path('app_backend_ajax_book_move', { id: data.id }) }}" data-id="{{ data.id }}" data-position="{{ data.position }}">
        <i class="arrow down icon grey"></i>{{ 'sylius.ui.move_down'|trans }}
    </div>
    {% endif %}
</div>
```
## Create a form specifically for updating position entity property

```php
# src/Form/Type/Book/BookPositionType
declare(strict_types=1);

namespace App\Form\Type\Book;

use App\Entity\Book\BookType;
use Symfony\Component\Form\AbstractType;
use Symfony\Component\Form\Extension\Core\Type\IntegerType;
use Symfony\Component\Form\FormBuilderInterface;
use Symfony\Component\OptionsResolver\OptionsResolver;

final class BookPositionType extends AbstractType
{
    public function buildForm(FormBuilderInterface $builder, array $options): void
    {
        $builder
            ->add('position', IntegerType::class, [
                'label' => 'sylius.ui.position',
            ])
        ;
    }

    public function configureOptions(OptionsResolver $resolver): void
    {
        $resolver->setDefaults([
            'data_class' => Book::class,
            'csrf_protection' => false,
        ]);
    }

    public function getBlockPrefix(): string
    {
        return 'app_book_position';
    }
}
```

## Build a new route to handle position in entity

```php
#[SyliusRoute(
    name: 'app_backend_ajax_book_move',
    path: '/admin/ajax/book/{id}/move',
    controller: 'app.controller.book::updateAction',
    form: BookPositionType::class,
)]
```

## Link everything with some Ajax

```javascript
// assets/backend/js/app-move-book.js
import 'semantic-ui-css/components/api';
import $ from 'jquery';

$.fn.extend({
  bookMoveUp() {
    const element = this;

    element.api({
      method: 'PUT',
      on: 'click',
      beforeSend(settings) {
        /* eslint-disable-next-line no-param-reassign */
        settings.data = {
          position: $(this).data('position') - 1,
        };

        return settings;
      },
      onSuccess() {
        window.location.reload();
      },
    });
  },

  bookMoveDown() {
    const element = this;

    element.api({
      method: 'PUT',
      on: 'click',
      beforeSend(settings) {
        /* eslint-disable-next-line no-param-reassign */
        settings.data = {
          position: $(this).data('position') + 1,
        };

        return settings;
      },
      onSuccess() {
        window.location.reload();
      },
    });
  },
});
```
And add these lines to main javascript file `app.js`.
```javascript
// assets/backend/js/main.js
import './app-move-book';
[...]
    $('.app-book-move-up').bookMoveUp();
    $('.app-book-move-down').bookMoveDown();

```

# 9.5 API Platform

If you're using api-pack, you might want to sort your entities with the new position field : add the `order` property.
```php

#[ApiResource(
    normalizationContext: ['groups' => ['book:read']],
    order: ['position' => 'ASC'],
)]
```


# 9.4 Updating tests

- Update your `BookFactory` class to add position in your fixtures.
- Add functional tests according to the purpose of sorting in your app.
- If you use ApiPlatform, update your previous tests results with the new entity sorting.
