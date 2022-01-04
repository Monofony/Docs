# Chapter 7 - Fixtures

Monofony uses the official Symfony package [zenstruck/foundry](https://symfony.com/bundles/ZenstruckFoundryBundle/current/index.html).

## Load all data fixtures {#all-fixtures}

It loads fixtures from both `default` and `fake` groups.
It allows you to load random data for your dev environment.

```bash
$ bin/console doctrine:fixtures:load -n
```

To add more data on `default` group, edit the `App\DataFixtures\DefaultFixtures`
To add more data on `fake` group, edit the `App\DataFixtures\FakeFixtures`

## Load minimal data fixtures {#minimal-fixtures}

It only loads fixtures from both `default` group.
It allows you to load initial data for your prod environment.

```bash
$ bin/console doctrine:fixtures:load --group=default -n
```

To add more data on this group, edit the `App\DataFixtures\DefaultFixtures`

## Learn More {#learn-more}
* [Official Zenstruck Foundry bundle documentation](https://symfony.com/bundles/ZenstruckFoundryBundle/current/index.html)
