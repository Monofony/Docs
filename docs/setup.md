# Chapter 1 - Setup

* [Setup the requirements](setup/requirements.md)

## Setup the application

```bash
composer install
bin/console app:install -n
bin/console sylius:fixtures:load -n
yarn install && yarn build
```

## Start the web local server

```bash
symfony server:start --no-tls
```
