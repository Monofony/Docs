# Menu

To configure your admin menu for your new resource, you have to edit `src/Menu/AdminMenuBuilder.php`.

```php
<?php

// [***]

public function createMenu(array $options): ItemInterface
{
    // add method ...
    $this->addContentSubMenu($menu);
    // rest of the code

    return $menu;
}

private function addContentSubMenu(ItemInterface $menu): ItemInterface
{
    $content = $menu
        ->addChild('content')
        ->setLabel('sylius.ui.content')
    ;

    $content->addChild('backend_article', ['route' => 'app_backend_article_index'])
        ->setLabel('app.ui.articles')
        ->setLabelAttribute('icon', 'newspaper');

    return $content;
}
```

## Learn More

* [Configuring menu in KnpMenuBundle documentation](https://symfony.com/doc/current/bundles/KnpMenuBundle/index.html)
