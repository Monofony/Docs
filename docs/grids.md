# Chapter 4 - Grids

![Grids](_images/grids.png)

## Basic example {#basic-example}

```php
<?php
// src/Grid/ArticleGrid.php

declare(strict_types=1);

use App\Entity\Article\Article;
use Sylius\Bundle\GridBundle\Builder\Action\CreateAction;
use Sylius\Bundle\GridBundle\Builder\Action\DeleteAction;
use Sylius\Bundle\GridBundle\Builder\Action\UpdateAction;
use Sylius\Bundle\GridBundle\Builder\ActionGroup\ItemActionGroup;
use Sylius\Bundle\GridBundle\Builder\ActionGroup\MainActionGroup;
use Sylius\Bundle\GridBundle\Builder\Field\StringField;
use Sylius\Bundle\GridBundle\Builder\Filter\StringFilter;
use Sylius\Bundle\GridBundle\Builder\GridBuilderInterface;
use Sylius\Bundle\GridBundle\Grid\AbstractGrid;
use Sylius\Bundle\GridBundle\Grid\ResourceAwareGridInterface;

final class ArticleGrid extends AbstractGrid implements ResourceAwareGridInterface 
{
    public static function getName(): string
    {
        return 'app_backend_article';
    }
    
    public function buildGrid(GridBuilderInterface $gridBuilder): void
    {
        $gridBuilder
            ->orderBy('title', 'asc')
            ->addField(
                StringField::create('title')
                    ->setLabel('sylius.ui.title')
                    ->setSortable(true)
            )
            ->addFilter(StringFilter::create('search', ['title']))
            ->addActionGroup(
                MainActionGroup::create(
                    CreateAction::create(),
                )
            )
            ->addActionGroup(
                ItemActionGroup::create(
                    UpdateAction::create(),
                    DeleteAction::create(),
                )
            )
        ;
    }
    
    public function getResourceClass(): string
    {
        return Article::class;
    }
}
```

## Advanced example {#advanced-example}

This configuration is the one used on our demo to show pets list on admin panel.

```php
<?php
// src/Grid/PetGrid.php

declare(strict_types=1);

use App\Entity\Animal\Pet;
use Sylius\Bundle\GridBundle\Builder\Action\CreateAction;
use Sylius\Bundle\GridBundle\Builder\Action\DeleteAction;
use Sylius\Bundle\GridBundle\Builder\Action\UpdateAction;
use Sylius\Bundle\GridBundle\Builder\ActionGroup\ItemActionGroup;
use Sylius\Bundle\GridBundle\Builder\ActionGroup\MainActionGroup;
use Sylius\Bundle\GridBundle\Builder\Field\StringField;
use Sylius\Bundle\GridBundle\Builder\Filter\StringFilter;
use Sylius\Bundle\GridBundle\Builder\GridBuilderInterface;
use Sylius\Bundle\GridBundle\Grid\AbstractGrid;
use Sylius\Bundle\GridBundle\Grid\ResourceAwareGridInterface;

final class PetGrid extends AbstractGrid implements ResourceAwareGridInterface 
{
    public static function getName(): string
    {
        return 'app_backend_pet';
    }
    
    public function buildGrid(GridBuilderInterface $gridBuilder): void
    {
        $gridBuilder
            ->setRepositoryMethod('createListQueryBuilder', ['$taxonId', '%locale%'])
            ->orderBy('name', 'asc')
            ->addField(
                TwigField::create('image', 'backend/pet/grid/field/image.html.twig')
                    ->setLabel('sylius.ui.image')
                    ->setPath('.')
            )
            ->addField(
                StringField::create('name')
                    ->setLabel('sylius.ui.name')
                    ->setSortable(true)
            )
           ->addField(
                StringField::create('taxon')
                    ->setLabel('app.ui.taxon')
                    ->setPath('taxon.translation.name')
                    ->setSortable(true, 'taxon.translation.name')
            )
            ->addField(
                TwigField::create('status', '@SyliusUi/Grid/Field/state.html.twig')
                    ->setLabel('app.ui.status')
                    ->setOption('vars' => ['labels' => 'backend/pet/label/state'])
            )
            ->addFilter(StringFilter::create('search', ['name', 'slug']))
            ->addActionGroup(
                MainActionGroup::create(
                    CreateAction::create(),
                )
            )
            ->addActionGroup(
                ItemActionGroup::create(
                    UpdateAction::create(),
                    DeleteAction::create(),
                )
            )
        ;
    }
    
    public function getResourceClass(): string
    {
        return Pet::class;
    }
}
```

## Learn More {#learn-more}
* [Configuring grid in sylius documentation](https://github.com/Sylius/SyliusGridBundle/blob/master/docs/index.md)
* [The whole configuration reference in sylius documentation](https://github.com/Sylius/SyliusGridBundle/blob/master/docs/configuration.md)
