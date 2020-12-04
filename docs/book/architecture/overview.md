# Architecture Overview

Before we dive separately into every Monofony concept, you need to have an overview of how our main application is structured.

Fullstack Symfony
-----------------

.. image:: ../../_images/symfonyfs.png
    :scale: 15%
    :align: center

**Monofony** is based on Symfony, which is a leading PHP framework to create web applications. Using Symfony allows
developers to work better and faster by providing them with certainty of developing an application that is fully compatible
with the business rules, that is structured, maintainable and upgradable, but also it allows to save time by providing generic re-usable modules.

[Learn more about Symfony](http://symfony.com/what-is-symfony).

Doctrine
--------

.. image:: ../../_images/doctrine.png
    :align: center

**Doctrine** is a family of PHP libraries focused on providing data persistence layer.
The most important are the object-relational mapper (ORM) and the database abstraction layer (DBAL).
One of Doctrine's key features is the possibility to write database queries in Doctrine Query Language (DQL) - an object-oriented dialect of SQL.

To learn more about Doctrine - see [their documentation](http://www.doctrine-project.org/about.html).

Twig
----

.. image:: ../../_images/twig.png
    :scale: 30%
    :align: center

**Twig** is a modern template engine for PHP that is really fast, secure and flexible. Twig is being used by Symfony.

To read more about Twig, [go here](http://twig.sensiolabs.org/).

Third Party Libraries
---------------------

Monofony uses a lot of libraries for various tasks:

* [Sylius](https://docs.sylius.com/en/latest/) for routing, controllers, data fixtures, grids
* [KnpMenu](http://symfony.com/doc/current/bundles/KnpMenuBundle/index.html) - for backend menus
* [Imagine](https://github.com/liip/LiipImagineBundle) for images processing, generating thumbnails and cropping
* [Pagerfanta](https://github.com/whiteoctober/Pagerfanta) for pagination
* [Winzou State Machine](https://github.com/winzou/StateMachineBundle) -  for the state machines handling
