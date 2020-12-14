## Setup the application

Install Monofony using composer
```bash
export SYMFONY_ENDPOINT=https://flex.symfony.com/r/github.com/symfony/recipes-contrib/1022
composer create-project monofony/skeleton acme
```

Install project :
```bash
$ bin/console app:install
$ yarn install && yarn build (or "yarn dev" for development usage)
$ symfony server:start --no-tls
```

### Api

The Monofony skeleton is built with the admin panel only.
You can install our API package to use our default endpoints using `Api Platform`

![alt text](https://github.com/Monofony/Monofony/raw/0.x/docs/_images/api.png "Logo Title Text 1")

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

### Front

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
