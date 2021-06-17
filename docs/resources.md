# Chapter 2 - Creating resources 

## A Basic Resource {#basic-resource}

`src/Entity/Article.php`
```php
<?php

declare(strict_types=1);

namespace App\Entity;

use Doctrine\ORM\Mapping as ORM;
use Sylius\Component\Resource\Model\ResourceInterface;

/**
 * @ORM\Entity
 * @ORM\Table(name="app_article")
 */
class Article implements ResourceInterface
{
    use IdentifiableTrait;

    /**
     * @var string|null
     *
     * @ORM\Column(type="string")
     */
    private $title;

    public function getTitle(): ?string
    {
        return $this->title;
    }

    public function setTitle(?string $title): void
    {
        $this->title = $title;
    }
}
```

The resource is an entity, which implements `ResourceInterface`. This interface only requires a `getId()` method.

The IdentifiableTrait contains:
 * the required getId() method.
 * the Doctrine mapping for the id property.

<div class="block-note">
The IdentifiableTrait has been created with Flex and this file is yours. You can edit it to fit your needs.
</div>

## Register the new Resource on Sylius {#register-new-resource}
You now have to register it on Sylius Resource configuration.

`config/sylius/resources.yaml`
```yaml
sylius_resource:
    resources:
        app.article:
            classes:
                model: App\Entity\Article
```

Sylius has registered some services for you.

| Service ID                                                                            |  Class name |
|---------------------------------------------------------------------------------------|------------------------------------------------------------|
| app.controller.article                                                                | Sylius\Bundle\ResourceBundle\Controller\ResourceController |                                                                     
| app.factory.article                                                                   | Sylius\Component\Resource\Factory\Factory                  |                                                                     
| app.manager.article                                                                   | alias for "doctrine.orm.default_entity_manager"            |                                                                     
| app.repository.article                                                                | Sylius\Bundle\ResourceBundle\Doctrine\ORM\EntityRepository |

<div markdown="1" class="block-note">
These results can be dumped with the following command.
```bash
$ bin/console debug:container
$ bin/console debug:container | grep article # to filters only on article services
```
</div>

The most interesting one is the `app.controller.article` which uses the ResourceController.

This controller can have many operations for your resource.

## Use PHP 8 attributes for Doctrine mapping {#php-8-attributes}

First, update the Doctrine configuration.

```yaml
# config/packages/doctrine.yaml
doctrine:
    # [...]
    orm:
        # [...]
        mappings:
        App:
            # [...]
            type: attribute
```

`src/Entity/Customer/Customer.php`
```php
<?php

// [...]

#[ORM\Entity]
#[ORM\Table(name: 'sylius_customer')]
class Customer extends BaseCustomer implements CustomerInterface
{
    /**
     * @Assert\Valid
     */
    #[ORM\OneToOne(targetEntity: AppUser::class, mappedBy: 'customer', cascade: ['persist'])]
    private ?UserInterface $user = null;

    // [...]
}
```

`src/Entity/Media/File.php`
```php
<?php

// [...]

#[ORM\MappedSuperclass]
abstract class File implements FileInterface, ResourceInterface
{
    // [...]
    
    /**
     * @Serializer\Groups({"Default", "Detailed"})
     */
    #[ORM\Column(type: 'string')]
    protected ?string $path = null;
    
    /**
     * @var \DateTimeInterface|null
     */
    #[ORM\Column(type: 'datetime')]
    protected $createdAt;
    
    /**
     * @var \DateTimeInterface|null
     */
    #[ORM\Column(type: 'datetime', nullable: true)]
    protected $updatedAt;
    
    // [...]
}
```

`src/Entity/OAuth/AccessToken.php`
```php
<?php

// [...]

use App\Entity\User\AppUser;

// [...]

#[ORM\Entity]
#[ORM\Table(name: 'oauth_access_token')]
class AccessToken extends BaseAccessToken
{
    #[ORM\Id]
    #[ORM\Column(type: 'integer')]
    #[ORM\GeneratedValue]
    protected $id;

    #[ORM\ManyToOne(targetEntity: Client::class)]
    #[ORM\JoinColumn(nullable: false)]
    protected $client;

    #[ORM\ManyToOne(targetEntity: AppUser::class)]
    protected $user;
}
```

`src/Entity/OAuth/AuthCode.php`
```php
<?php

// [...]

use App\Entity\User\AppUser;

// [...]

#[ORM\Entity]
#[ORM\Table(name: 'oauth_auth_code')]
class AuthCode extends BaseAuthCode
{
    #[ORM\Id]
    #[ORM\Column(type: 'integer')]
    #[ORM\GeneratedValue]
    protected $id;

    #[ORM\ManyToOne(targetEntity: Client::class)]
    #[ORM\JoinColumn(nullable: false)]
    protected $client;

    #[ORM\ManyToOne(targetEntity: AppUser::class)]
    protected $user;
}
```

`src/Entity/OAuth/Client.php`
```php
<?php

// [...]

#[ORM\Entity]
#[ORM\Table(name: 'oauth_client')]
class Client extends BaseClient implements ClientInterface
{
    #[ORM\Id]
    #[ORM\Column(type: 'integer')]
    #[ORM\GeneratedValue]
    protected $id;

    /**
     * {@inheritdoc}
     */
    public function getPublicId(): string
    {
        return $this->getRandomId();
    }
}
```

`src/Entity/User/AdminAvatar.php`
```php
<?php

// [...]

/**
 * @Vich\Uploadable
 */
#[ORM\Entity]
#[ORM\Table(name: 'app_admin_avatar')]
class AdminAvatar extends File implements AdminAvatarInterface
{
   // [..]
}
```

`src/Entity/User/AdminUser.php`
```php
<?php

// [...]

#[ORM\Entity]
#[ORM\Table(name: 'sylius_admin_user')]
class AdminUser extends BaseUser implements AdminUserInterface
{
    #[ORM\Column(type: 'string', nullable: true)]
    private ?string $lastName = null;

    #[ORM\Column(type: 'string', nullable: true)]
    private ?string $firstName = null;

    #[ORM\OneToOne(targetEntity: AdminAvatar::class, cascade: ['persist'])]
    private ?AdminAvatarInterface $avatar = null;
    
    // [...]
}
```

`src/Entity/User/AppUser.php`
```php
<?php

// [...]

#[ORM\Entity]
#[ORM\Table(name: 'sylius_app_user')]
class AppUser extends BaseUser implements AppUserInterface
{
    #[ORM\OneToOne(targetEntity: CustomerInterface::class, inversedBy: 'user', cascade: ['persist'])]
    #[ORM\JoinColumn(nullable: false)]
    private ?CustomerInterface $customer = null;
    
    // [...]
}
```
