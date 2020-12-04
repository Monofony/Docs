# How to configure phpspec with code coverage

By default, phpspec on Monofony is configured with code coverage which needs xdebug or phpdbg installed.
Thus you have two options:
* install xdebug
* install phpdbg (easier and faster)


> But if you don't need that feature, [disable code coverage](how-to-disable-phpspec-code-coverage.md).


## Install phpdbg

```bash
$ # on linux
$ sudo apt-get install php7.2-phpdbg
$
$ # on max
$ brew install php72-phpdbg
```
