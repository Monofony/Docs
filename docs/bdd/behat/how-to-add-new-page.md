How to add a new page object?
=============================

Sylius uses a solution inspired by ``SensioLabs/PageObjectExtension``, which provides an infrastructure to create
pages that encapsulates all the user interface manipulation in page objects.

To create a new page object it is needed to add a service.

The simplest Symfony-based page looks like:

```php
<?php

declare(strict_types=1);

namespace App\Tests\Behat\Page\Frontend;

use FriendsOfBehat\PageObjectExtension\Page\SymfonyPage;

class LoginPage extends SymfonyPage
{
    public function getRouteName(): string
    {
        return 'app_frontend_security_login';
    }
}
```

<div markdown="1" class="block-note">
There are some boilerplates for common pages, which you may use. The available parents are ``FriendsOfBehat\PageObjectExtension\Page\Page``
and ``FriendsOfBehat\PageObjectExtension\Page\SymfonyPage``. It is not required for a page to extend any class as
pages are POPOs (Plain Old PHP Objects).
</div>
