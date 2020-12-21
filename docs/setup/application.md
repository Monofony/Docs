# 1.2 - Setup the application

Install Monofony using composer
```bash
export SYMFONY_ENDPOINT=https://flex.symfony.com/r/github.com/symfony/recipes-contrib/1022
composer create-project monofony/skeleton acme  # replace acme by your project name
cd acme                                         # move to your project directory
```

Install project :
```bash
$ bin/console app:install -n            # install the application with non-interactive mode
$ bin/console sylius:fixtures:load -n   # load data fixtures
$ yarn install                          # install node packages
$ yarn build                            # or yarn dev for development
$ symfony server:start --no-tls         # start a local web server
```

## Api

The Monofony skeleton is built with the admin panel only.
You can install our API package to use our default endpoints using `Api Platform`

![API Example](/build/images/api.png "Image API example")

Uncomment `$syliusResources` binding on `config/services.yaml`

```yaml
# config/services.yaml
services:
    # ...
    _defaults:
        # ...
        bind:
            # ...
            $syliusResources: '%sylius.resources%' # for api
```

And execute the following commands:

```bash
export SYMFONY_ENDPOINT=https://flex.symfony.com/r/github.com/symfony/recipes-contrib/1022
composer require monofony/api-pack
```

## Front

To build a frontend, you can use our front-pack with default features:
* login
* register
* forgotten password
* user profile

You can install it using the following commands:

```bash
export SYMFONY_ENDPOINT=https://flex.symfony.com/r/github.com/symfony/recipes-contrib/1022
composer require monofony/front-pack
```
