# Chapter 3 - Routes

To configure admin routes for your resource, you have to create a new file on backend routes folder ``config/routes/backend``.

Let’s configure our “Article” routes as an example.

<div markdown="1" class="block-note">
If you haven’t already created your first resource, check out [Creating resources](resources.md) in Monofony and then come back!
</div>

```yaml
# config/routes/backend/article.yaml
app_backend_article:
    resource: |
        alias: app.article
        section: backend
        except: ['show']
        redirect: update
        grid: app_backend_article
        vars:
            all:
                subheader: app.ui.manage_articles
            index:
                icon: newspaper
        templates: backend/crud
    type: sylius.resource
```

And add it on backend routes configuration.

```yaml
# config/routes/backend/_main.yaml
[...]

app_backend_article:
    resource: "article.yaml"
```

And that’s all!

All the routes are now available:

|  Name                                  |   Method        |      Scheme   | Host  | Path                             |                                     
|----------------------------------------|-----------------|---------------|-------|----------------------------------|
| app_backend_article_index              | GET             |  ANY          |  ANY  |  /admin/articles/                |                             
| app_backend_article_create             | GET\|POST       |  ANY          |  ANY  |  /admin/articles/new             |                
| app_backend_article_update             | GET\|PUT\|PATCH |  ANY          |  ANY  |  /admin/articles/{id}/edit       |                
| app_backend_article_bulk_delete        | DELETE          |  ANY          |  ANY  |  /admin/articles/bulk-delete     |                
| app_backend_article_delete             | DELETE          |  ANY          |  ANY  |  /admin/articles/{id}            |

<div markdown="1" class="block-note">
These results can be dumped with the following command.
```bash
$ bin/console debug:router
$ bin/console debug:router | grep article # to filters only on article routes
```
</div>

## Learn More

* [Configuring routes in sylius documentation](https://github.com/Sylius/SyliusResourceBundle/blob/master/docs/routing.md)
