# How to use transformers?

Behat provides many awesome features, and one of them is definitely **transformers**. They can be used to transform (usually widely used) parts of steps and return some values from them,
to prevent unnecessary duplication in many steps' definitions.

## Basic transformer

Example is always the best way to clarify, so let's look at this:

```php
<?php

declare(strict_types=1);

namespace App\Tests\Behat\Transform;

use App\Entity\ShippingMethod;
use Behat\Behat\Context\Context;
use Doctrine\ORM\EntityManagerInterface;
use Webmozart\Assert\Assert;

class ShippingMethodContext implements Context
{
    private $entityManager;
    
    public function __construct(EntityManagerInterface $entityManager) 
    {
        $this->entityManager = $entityManager;
    }

    /**
     * @Transform /^"([^"]+)" shipping method$/
     * @Transform /^shipping method "([^"]+)"$/
     * @Transform :shippingMethod
     */
    public function getShippingMethodByName($shippingMethodName): ShippingMethod
    {
        $shippingMethod = $this->entityManager
            ->getRepository(ShippingMethod::class)
            ->findOneByName($shippingMethodName);

        Assert::notNull(
            $shippingMethod,
            sprintf('Shipping method with name "%s" does not exist', $shippingMethodName)
        );

        return $shippingMethod;
    }
}    
```    

This transformer is used to return ``ShippingMethod`` object from proper repository using it's name. It also throws exception if such a method does not exist. It can be used in plenty of steps,
that have shipping method name in it.

<div markdown="1" class="block-note">
In the example above a [Webmozart assertion](https://github.com/webmozart/assert) library was used, to assert a value and throw an exception if needed.
</div>

But how to use it? It is as simple as that:

```php
/**
 * @Given /^(shipping method "[^"]+") belongs to ("[^"]+" tax category)$/
 */
public function shippingMethodBelongsToTaxCategory(
    ShippingMethodInterface $shippingMethod,
    TaxCategoryInterface $taxCategory
) {
    // some logic here
}
```

If part of step matches transformer definition, it should be surrounded by parenthesis to be handled as whole expression. That's it! As it is shown in the example, many transformers can be
used in the same step definition. Is it all? No! The following example will also work like charm:

```php
/**
 * @When I delete shipping method :shippingMethod
 * @When I try to delete shipping method :shippingMethod
 */
public function iDeleteShippingMethod(ShippingMethodInterface $shippingMethod)
{
    // some logic here
}
```    

It is worth to mention, that in such a case, transformer would be matched depending on a name after ':' sign. So many transformes could be used when using this signature also.
This style gives an opportunity to write simple steps with transformers, without any regex, which would boost context readability.

<div markdown="1" class="block-note">
Transformer definition does not have to be implemented in the same context, where it is used. It allows to share them between many different contexts.
</div>

## Transformers implemented in Monofony

Moreover, there can be more generic transformers, that could be useful in many different cases.

### SharedStorageContext

``SharedStorageContext`` is kind of container used to keep objects, which can be shared between steps. It can be used, for example, to keep newly created promotion,
to use its name in checking existence step.

* ``@Transform /^(it|its|theirs)$/`` -> amazingly useful transformer, that returns last resource saved in ``SharedStorage``. It allows to simplify many steps used after creation/update (and so on) actions. Example: instead of writing ``When I create "Wade Wilson" customer/Then customer "Wade Wilson" should be registered`` just write ``When I create "Wade Wilson" customer/Then it should be registered``
* ``@Transform /^(?:this|that|the) ([^"]+)$/`` -> similar to previous one, but returns resource saved with specific key, for example ``this promotion`` will return resource saved with ``promotion`` key in ``SharedStorage``
