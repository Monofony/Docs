We'll see how to transform a string property of an entity to a translatable string property.

# 10.1 - Basics

## Update your composer configuration

We'll need doctrine extensions and sylius/locale-bundle
```bash
composer require stof/doctrine-extensions-bundle
composer require sylius/locale-bundle
```

Update framework and services configuration
```yaml
# config/packages/framework.yaml
framework:
  default_locale: '%locale%'
```

```yaml
# config/services.yaml
framework:
  default_locale: '%locale%'
```
