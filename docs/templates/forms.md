# 6.1 - Customize your forms

## Create your own template {#custom}

```twig
<!-- templates/backend/article/_form.html.twig -->
<div class="two fields">
    {{ form_row(form.title) }}
    {{ form_row(form.slug) }}
</div>

```

This is just an example. You can do whatever you want.

## Add your template to the configuration {#configuration}

```yaml
# config/sylius/routes/backend/article.yaml
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
                templates:
                    form: backend/article/_form.html.twig
            index:
                icon: newspaper
        templates: backend/crud
    type: sylius.resource
```

This template will be used for both create and update pages.

## How does it work? {#how-does-it-work}

```twig
<!-- templates/backend/crud/update/_content.html.twig -->
<div class="ui segment">
    <!-- [...] some code here -->
    {% if configuration.vars.templates.form is defined %}
        {% include configuration.vars.templates.form %}
    {% else %}
        {{ form_widget(form) }}
    {% endif %}
    <!-- [...] some code there -->
</div>
```

<div markdown="1" class="block-note">
The `vars` property on your route's configuration is available on `configuration.vars` on your twig template.
</div>
