# 1.1 - Setup the requirements

Running Symfony on local machines are faster without using Docker for both PHP and Apache/Nginx.
We recommend using the Symfony web local server instead.

## Install Symfony CLI {#symfony-cli}
```bash
curl -sS https://get.symfony.com/cli/installer | bash
```

## PHP 7.3 or later {#php}
Ensure you have php 7.3 or later locally installed

```bash
php -v
```

If you do not have php yet...

On macOS, you can use brew:
```bash
brew install php@7.3
```

On linux, you can use apt-get:
```bash
sudo apt-get install php7.3
```

## MySQL 5.7 {#mysql}

We'll provide a docker-compose to setup mysql5.7.

But you can also install it yourself.

On macOS, you can use brew:
```bash
brew install mysql@5.7
```
