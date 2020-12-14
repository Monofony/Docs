# How to manage your new entity on the admin panel

To add a new grid, create a new grid configuration file in ``config/packages/grids/backend/`` and import this to sylius_grid configuration file

## Create a new grid configuration file

`config/sylius/grids/backend/article.yaml`
```yaml
sylius_grid:
    grids:
        app_backend_article:
            driver:
                name: doctrine/orm
                options:
                    class: "%app.model.article.class%"
            sorting:
                title: asc
            fields:
                title:
                    type: string
                    label: sylius.ui.title
                    sortable: null
            filters:
                search:
                    type: string
                    label: sylius.ui.search
                    options:
                        fields: [title]
            actions:
                main:
                    create:
                        type: create
                item:
                    update:
                        type: update
                    delete:
                        type: delete
```

<div class="block-warning">
You need to clear the Symfony cache when creating a new sylius grid configuration file.
</div>

## Manually importing your Sylius grid's configuration (optional)

Grids configuration files are automatically detected when clearing the cache.
You can manually add them if you prefer.

`config/sylius/grids.yaml`

```yaml
imports:
    - { resource: 'grids/backend/article.yaml' }
    - { resource: 'grids/backend/admin_user.yaml' }
    - { resource: 'grids/backend/customer.yaml' }`
```

## Learn More

* [Configuring grid in sylius documentation](https://github.com/Sylius/SyliusGridBundle/blob/master/docs/index.md)
* [The whole configuration reference in sylius documentation](https://github.com/Sylius/SyliusGridBundle/blob/master/docs/configuration.md)
