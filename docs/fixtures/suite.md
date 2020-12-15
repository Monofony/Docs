# 7.3 - How to use it on your suite

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
