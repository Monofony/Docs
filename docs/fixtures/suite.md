# 7.3 - How to use it on your suite

This defines options you can use on [fixtures bundle configurations yaml files](https://github.com/Monofony/Monofony/blob/0.x/src/Monofony/Pack/CorePack/.recipe/config/sylius/fixtures.yaml).

`config/sylius/fixtures.yaml`
```yaml
sylius_fixtures:
    suites:
        default:
            listeners:
                orm_purger: ~
                logger: ~

            fixtures:
                # [...]

                article:
                    options:
                        random: 10
                        custom:
                            -
                                title: "Awesome article"
```

It will generates 10 random articles and one custom with title ``Awesome article``.
