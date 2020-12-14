# Chapter 2 - Creating resources 

## A Basic Resource

`src/Entity/Article.php`
```php
<?php

declare(strict_types=1);

namespace App\Entity;

use Doctrine\ORM\Mapping as ORM;
use Sylius\Component\Resource\Model\ResourceInterface;

/**
 * @ORM\Entity
 * @ORM\Table(name="app_article")
 */
class Article implements ResourceInterface
{
    use IdentifiableTrait;

    /**
     * @var string|null
     *
     * @ORM\Column(type="string")
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
}
```

The resource is an entity, which implements `ResourceInterface`. This interface only requires a `getId()` method.

The IdentifiableTrait contains:
 * the required getId() method.
 * the Doctrine mapping for the id property.

<div class="block-note">
The IdentifiableTrait has been created with Flex and this file is yours. You can edit it to fit your needs.
</div>

## Register the new Resource on Sylius
You now have to register it on Sylius Resource configuration.

`config/sylius/resources.yaml`
```yaml
sylius_resource:
    resources:
        app.article:
            classes:
                model: App\Entity\Article
```

Sylius has registered some services for you.

| Service ID                                                                            |  Class name |
|---------------------------------------------------------------------------------------|------------------------------------------------------------|
| app.controller.article                                                                | Sylius\Bundle\ResourceBundle\Controller\ResourceController |                                                                     
| app.factory.article                                                                   | Sylius\Component\Resource\Factory\Factory                  |                                                                     
| app.manager.article                                                                   | alias for "doctrine.orm.default_entity_manager"            |                                                                     
| app.repository.article                                                                | Sylius\Bundle\ResourceBundle\Doctrine\ORM\EntityRepository |

The most interesting one is the `app.controller.article` which uses the ResourceController.

This controller can have many operations for your resource.
