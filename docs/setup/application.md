# 1.2 - Setup the application

## Skeleton {#skeleton}

### Install Monofony using composer {#composer}
```bash
composer create-project monofony/skeleton project_name  # replace project_name by your project name
cd project_name                                         # move to your project directory
```

### Configure your database {#database}

Uncomment and configure your MySQL on `.env`.
```dotenv
DATABASE_URL="mysql://db_user:db_password@127.0.0.1:3306/db_name?serverVersion=5.7"
```

<div class="block-warning">
Don't forgot to replace db_user and db_password by yours.
</div>

<div class="block-note">
PostgreSQL can be used, and it is now recommended, but Doctrine migrations have been generated on MySQL, you have to delete them first.
</div>

### Install project {#project}
```bash
$ bin/console app:install -n            # install the application with non-interactive mode
$ bin/console doctrine:fixtures:load -n # load data fixtures
$ yarn install                          # install node packages
$ yarn build                            # or yarn dev for development
$ symfony serve -d                      # start a local web server
```

## Api {#api}

The Monofony skeleton is built with the admin panel only.
You can install our API package to use our default endpoints using `Api Platform`

![API Example](../_images/api.png "Image API example")

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

Uncomment `success_handler`, `failure_handler`, `jwt` and `refresh_jwt` on `config/packages/security.yaml`

```yaml
# config/packages/security.yaml
security:
    # ...
    firewalls:
        # ...        
        api_login:
        pattern: ^/api/authentication_token
        provider: sylius_app_user_provider
        stateless: true
        json_login:
            check_path: /api/authentication_token
            success_handler: lexik_jwt_authentication.handler.authentication_success
            failure_handler: lexik_jwt_authentication.handler.authentication_failure

        api:
            pattern: ^/api
            provider: sylius_app_user_provider
            stateless: true
            jwt: true
            refresh_jwt:
                check_path: /api/token/refresh

    
```

And execute the following commands:

```bash
composer require monofony/api-pack "^0.9"
```

If you see an exception as shown below, run the migration commands.

> Class "App\Entity\RefreshToken" sub class of "Gesdinet\JWTRefreshTokenBundle\Entity\RefreshToken" is not a valid entity or mapped super class.

```bash
bin/console doctrine:migrations:diff

bin/console doctrine:migrations:migrate
```

## Front {#front}

To build a frontend, you can use our front-pack with default features:
* login
* register
* forgotten password
* user profile

You can install it using the following commands:

```bash
composer require monofony/front-pack "^0.9"
```
