# Basic Usage

The best way of understanding how things work in detail is showing and analyzing examples, that is why this section gathers all the knowledge from the previous chapters.
Let's assume that we are going to implement the functionality of managing countries in our system.
Now let us show you the flow.

## Describing features

Let's start with writing our feature file, which will contain answers to the most important questions:
Why (benefit, business value), who (actor using the feature) and what (the feature itself).
It should also include scenarios, which serve as examples of how things supposed to work.
Let's have a look at the ``features/addressing/managing_countries/adding_country.feature`` file.

`features/addressing/managing_countries/adding_country.feature`
```gherkin
@managing_countries
Feature: Adding a new country
    In order to sell my goods to different countries
    As an Administrator
    I want to add a new country to the store

    Background:
        Given I am logged in as an administrator

    @ui
    Scenario: Adding country
        When I want to add a new country
        And I choose "United States"
        And I add it
        Then I should be notified that it has been successfully created
        And the country "United States" should appear in the store
```

Pay attention to the form of these sentences. From the developer point of view they are hiding the details of the feature's implementation.
Instead of describing "When I click on the select box And I choose United States from the dropdown Then I should see the United States country in the table"
- we are using sentences that are less connected with the implementation, but more focused on the effects of our actions.
A side effect of such approach is that it results in steps being really generic, therefore if we want to add another way of testing this feature for instance in the domain or api context,
it will be extremely easy to apply. We just need to add a different tag (in this case "@domain") and of course implement the proper steps in the domain context of our system.
To be more descriptive let's imagine that we want to check if a country is added properly in two ways.
First we are checking if the adding works via frontend, so we are implementing steps that are clicking, opening pages,
filling fields on forms and similar, but also we want to check this action regardlessly of the frontend, for that we need the domain, which allows us to perform actions only on objects.

Choosing a correct suite
------------------------

After we are done with a feature file, we have to create a new suite for it. At the beginning we have decided that it will be a frontend/user interface feature, that is why we are placing it in "config/suites/ui/addressing/managing_countries.yaml".

`config/suites/ui/addressing/managing_countries.yaml`
```yaml
default:
    suites:
        ui_managing_countries:
            contexts:
                # This service is responsible for clearing database before each scenario,
                # so that only data from the current and its background is available.
                - App\Tests\Behat\Context\Hook\DoctrineORMContext

                # The transformer contexts services are responsible for all the transformations of data in steps:
                # For instance "And the country "France" should appear in the store" transforms "(the country "France")" to a proper Country object, which is from now on available in the scope of the step.
                - App\Tests\Behat\Context\Transform\CountryContext
                - App\Tests\Behat\Context\Transform\SharedStorageContext

                # The setup contexts here are preparing the background, adding available countries and users or administrators.
                # These contexts have steps like "I am logged in as an administrator" already implemented.
                - App\Tests\Behat\Context\Setup\GeographicalContext
                - App\Tests\Behat\Context\Setup\SecurityContext

                # Lights, Camera, Action!
                # Those contexts are essential here we are placing all action steps like "When I choose "France" and I add it Then I should ne notified that...".
                - App\Tests\Behat\Context\Ui\Backend\ManagingCountriesContext
                - App\Tests\Behat\Context\Ui\Backend\NotificationContext
            filters:
                tags: "@managing_countries && @ui"
```

A very important thing that is done here is the configuration of tags, from now on Behat will be searching for all your features tagged with ``@managing_countries`` and your scenarios tagged with ``@ui``.

We have mentioned with the generic steps we can easily switch our testing context to @domain. Have a look how it looks:

`config/suites/domain/addressing/managing_countries.yaml`
```yaml
default:
    suites:
        domain_managing_countries:
            contexts:
                - App\Tests\Behat\Context\Hook\DoctrineORMContext

                - App\Tests\Behat\Context\Transform\CountryContext
                - App\Tests\Behat\Context\Transform\SharedStorageContext

                - App\Tests\Behat\Context\Setup\GeographicalContext
                - App\Tests\Behat\Context\Setup\SecurityContext

                # Domain step implementation.
                - App\Tests\Behat\Context\Domain\Backend\ManagingCountriesContext
            filters:
                tags: "@managing_countries && @domain"
```

We are almost finished with the suite configuration.

## Registering Pages

The page object approach allows us to hide all the detailed interaction with ui (html, javascript, css) inside.

We have three kinds of pages:
    - Page - First layer of our pages it knows how to interact with DOM objects. It has a method ``getUrl(array $urlParameters)`` where you can define a raw url to open it.
    - SymfonyPage - This page extends the Page. It has a router injected so that the ``getUrl()`` method generates a url from the route name which it gets from the ``getRouteName()`` method.
    - Base Crud Pages (IndexPage, CreatePage, UpdatePage) - These pages extend SymfonyPage and they are specific to the Sylius resources. They have a resource name injected and therefore they know about the route name.

There are two ways to manipulate UI - by using ``getDocument()`` or ``getElement('your_element')``.
First method will return a ``DocumentElement`` which represents an html structure of the currently opened page,
second one is a bit more tricky because it uses the ``->getDefinedElements(): array`` method and it will return a ``NodeElement`` which represents only the restricted html structure.

Usage example of ``getElement('your_element')`` and ``getDefinedElements()`` methods.

```php
final class CreatePage extends SymfonyPage implements CreatePageInterface
{
    // This method returns a simple associative array, where the key is the name of your element and the value is its locator.
    protected function getDefinedElements(): array
    {
        return array_merge(parent::getDefinedElements(): array, [
            'provinces' => '#sylius_country_provinces',
        ]);
    }

    // By default it will assume that your locator is css.
    // Example with xpath.
    protected function getDefinedElements(): array
    {
        return array_merge(parent::getDefinedElements(): array, [
            'provinces_css' => '.provinces',
            'provinces_xpath' => ['xpath' => '//*[contains(@class, "provinces")]'], // Now your value is an array where key is your locator type.
        ]);
    }

    // Like that you can easily manipulate your page elements.
    public function addProvince(ProvinceInterface $province): void
    {
        $provinceSelectBox = $this->getElement('provinces');

        $provinceSelectBox->selectOption($province->getName());
    }
}
```

Let's get back to our main example and analyze our scenario. We have steps like:

```gherkin
When I choose "France"
And I add it
Then I should be notified that it has been successfully created
And the country "France" should appear in the store
```

```php
<?php

declare(strict_types=1);

namespace App\Tests\Behat\Page\Backend\Country;

use Monofony\Bridge\Behat\Crud\AbstractCreatePage;

final class CreatePage extends AbstractCreatePage
{
    /**
     * {@inheritdoc}
     */
    public function getRouteName(): string
    {
        return 'app_backend_country_create';
    }

    public function chooseName(string $name): void
    {
        $this->getDocument()->selectFieldOption('Name', $name);
    }
}
```

```php
<?php

declare(strict_types=1);

namespace App\Tests\Behat\Page\Backend\Country;

use Monofony\Bridge\Behat\Crud\AbstractIndexPage;

final class IndexPage extends AbstractIndexPage
{
    /**
     * {@inheritdoc}
     */
    public function getRouteName(): string
    {
        return 'app_backend_country_index';
    }
}
```

<div markdown="1" class="block-note">
There is one small gap in this concept - PageObjects is not a concrete instance of the currently opened page, they only mimic its behaviour (dummy pages).
This gap will be more understandable on the below code example.
</div>

```php

// Of course this is only to illustrate this gap.

class HomePage extends Page
{
    // In this context on home page sidebar you have for example weather information in selected countries.
    public function readWeather()
    {
        return $this->getElement('sidebar')->getText();
    }

    protected function getDefinedElements(): array
    {
        return ['sidebar' => ['css' => '.sidebar']];
    }

    protected function getUrl()
    {
        return 'http://your_domain.com';
    }
}

class LeagueIndexPage extends Page
{
    // In this context you have for example football match results.
    public function readMatchResults()
    {
        return $this->getElement('sidebar')->getText();
    }

    protected function getDefinedElements(): array
    {
        return ['sidebar' => ['css' => '.sidebar']];
    }

    protected function getUrl()
    {
        return 'http://your_domain.com/leagues/';
    }
}

final class GapContext implements Context
{
    private $homePage;
    private $leagueIndexPage;

    public function __construct(HomePage $homePage, LeagueIndexPage $leagueIndexPage)
    {
        $this->homePage = $homePage;
        $this->leagueIndexPage = $leagueIndexPage;
    }

    /**
     * @Given I want to be on Homepage
     */
    public function iWantToBeOnHomePage() // After this method call we will be on "http://your_domain.com".
    {
        $this->homePage->open(); //When we add @javascript tag we can actually see this thanks to selenium.
    }

    /**
     * @Then I want to see the sidebar and get information about the weather in France
     */
    public function iWantToReadSideBarOnHomePage($someInformation) // Still "http://your_domain.com".
    {
        $someInformation === $this->leagueIndexPage->readMatchResults(); // This returns true, but wait a second we are on home page (dummy pages).

        $someInformation === $this->homePage->readWeather(); // This also returns true.
    }
}
```

Registering contexts
--------------------

As it was shown in the previous section we have registered a lot of contexts, so we will show you only some of the steps implementation.

```gherkin
Given I want to add a new country
And I choose "United States"
And I add it
Then I should be notified that it has been successfully created
And the country "United States" should appear in the store
```

Let's start with essential one ManagingCountriesContext

### Ui contexts

```php
<?php

declare(strict_types=1);

namespace App\Tests\Behat\Context\Ui\Backend;

use Behat\Behat\Context\Context;
use App\Tests\Behat\Page\Backend\Country\CreatePage;
use App\Tests\Behat\Page\Backend\Country\IndexPage;
use App\Tests\Behat\Page\Backend\Country\UpdatePage;

final class ManagingCountriesContext implements Context
{
    private $indexPage;
    private $createPage;
    private $updatePage;

    public function __construct(
        IndexPage $indexPage,
        CreatePage $createPage,
        UpdatePage $updatePage
    ) {
        $this->indexPage = $indexPage;
        $this->createPage = $createPage;
        $this->updatePage = $updatePage;
    }

    /**
     * @Given I want to add a new country
     */
    public function iWantToAddNewCountry(): void
    {
        $this->createPage->open(); // This method will send request.
    }

    /**
     * @When I choose :countryName
     */
    public function iChoose($countryName): void
    {
        $this->createPage->chooseName($countryName);
        // Great benefit of using page objects is that we hide html manipulation behind a interfaces so we can inject different CreatePage which implements CreatePageInterface
        // And have different html elements which allows for example chooseName($countryName).
    }

    /**
     * @When I add it
     */
    public function iAddIt(): void
    {
        $this->createPage->create();
    }

    /**
     * @Then /^the (country "([^"]+)") should appear in the store$/
     */
    public function countryShouldAppearInTheStore(CountryInterface $country): void // This step use Country transformer to get Country object.
    {
        $this->indexPage->open();

        //Webmozart assert library.
        Assert::true(
            $this->indexPage->isSingleResourceOnPage(['code' => $country->getCode()]),
            sprintf('Country %s should exist but it does not', $country->getCode())
        );
    }
}
```

```php
<?php

declare(strict_types=1);

namespace App\Tests\Behat\Context\Ui\Backend;

use Behat\Behat\Context\Context;
use Monofony\Bridge\Behat\Service\NotificationCheckerInterface;

final class NotificationContext implements Context
{
    /**
     * This is a helper service which give access to proper notification elements.
     */
    private $notificationChecker;

    public function __construct(NotificationCheckerInterface $notificationChecker)
    {
        $this->notificationChecker = $notificationChecker;
    }

    /**
     * @Then I should be notified that it has been successfully created
     */
    public function iShouldBeNotifiedItHasBeenSuccessfullyCreated(): void
    {
        $this->notificationChecker->checkNotification('has been successfully created.', NotificationType::success());
    }
}
```

### Transformer contexts

```php
<?php

declare(strict_types=1);

namespace App\Tests\Behat\Context\Transform;

use Behat\Behat\Context\Context;

final class CountryContext implements Context
{
    /** @var CountryNameConverterInterface */
    private $countryNameConverter;

    /** @var RepositoryInterface */
    private $countryRepository;

    public function __construct(
        CountryNameConverterInterface $countryNameConverter,
        RepositoryInterface $countryRepository
    ) {
        $this->countryNameConverter = $countryNameConverter;
        $this->countryRepository = $countryRepository;
    }

    /**
     * @Transform /^country "([^"]+)"$/
     * @Transform /^"([^"]+)" country$/
     */
    public function getCountryByName(string $countryName): Country // Thanks to this method we got in our ManagingCountries an Country object.
    {
        $countryCode = $this->countryNameConverter->convertToCode($countryName);
        $country = $this->countryRepository->findOneBy(['code' => $countryCode]);

        Assert::notNull(
            $country,
            'Country with name %s does not exist'
        );

        return $country;
    }
}
```

```php
<?php

declare(strict_types=1);

namespace App\Tests\Behat\Context\Ui\Backend;

use App\Entity\Country;
use App\Tests\Behat\Page\Backend\Country\UpdatePage;
use Behat\Behat\Context\Context;

final class ManagingCountriesContext implements Context
{
    private $updatePage;

    public function __construct(UpdatePage $updatePage)
    {
        $this->updatePage = $updatePage;
    }

    /**
     * @Given /^I want to create a new province in (country "[^"]+")$/
     */
    public function iWantToCreateANewProvinceInCountry(Country $country)
    {
        $this->updatePage->open(['id' => $country->getId()]);

        $this->updatePage->clickAddProvinceButton();
    }
}
```

```php
<?php

declare(strict_types=1);

namespace App\Tests\Behat\Context\Transform;

use Behat\Behat\Context\Context;

final class ShippingMethodContext implements Context
{
    /** @var ShippingMethodRepositoryInterface */
    private $shippingMethodRepository;

    public function __construct(ShippingMethodRepositoryInterface $shippingMethodRepository)
    {
        $this->shippingMethodRepository = $shippingMethodRepository;
    }

    /**
     * @Transform :shippingMethod
     */
    public function getShippingMethodByName($shippingMethodName)
    {
        $shippingMethod = $this->shippingMethodRepository->findOneByName($shippingMethodName);
        if (null === $shippingMethod) {
            throw new \Exception('Shipping method with name "'.$shippingMethodName.'" does not exist');
        }

        return $shippingMethod;
    }
}
```

```php
<?php

declare(strict_types=1);

namespace App\Tests\Behat\Context\Ui\Admin;

use App\Tests\Behat\Page\Admin\ShippingMethod\UpdatePageInterface;
use Behat\Behat\Context\Context;

final class ShippingMethodContext implements Context
{
    /** @var UpdatePageInterface */
    private $updatePage;

    public function __construct(UpdatePageInterface $updatePage)
    {
        $this->updatePage = $updatePage;
    }

    /**
     * @Given I want to modify a shipping method :shippingMethod
     */
    public function iWantToModifyAShippingMethod(ShippingMethodInterface $shippingMethod)
    {
        $this->updatePage->open(['id' => $shippingMethod->getId()]);
    }
}
```

<div class="block-warning">
Contexts should have single responsibility and this segregation (Setup, Transformer, Ui, etc...) is not accidental.
We shouldn't create objects in transformer contexts.
</div>    

### Setup contexts

For setup context we need different scenario with more background steps and all preparing scene steps.
Editing scenario will be great for this example:

```gherkin
Scenario:
    Given the store has disabled country "France"
    And I want to edit this country
    When I enable it
    And I save my changes
    Then I should be notified that it has been successfully edited
    And this country should be enabled
```    

```php
<?php

declare(strict_types=1);

namespace App\Tests\Behat\Context\Setup;

use Behat\Behat\Context\Context;
use Monofony\Bridge\Behat\Service\SharedStorageInterface;

final class GeographicalContext implements Context
{
    private $sharedStorage;
    private $countryFactory;
    private $countryRepository;
    private $countryNameConverter;

    public function __construct(
        SharedStorageInterface $sharedStorage,
        FactoryInterface $countryFactory,
        RepositoryInterface $countryRepository,
        CountryNameConverterInterface $countryNameConverter
    ) {
        $this->sharedStorage = $sharedStorage;
        $this->countryFactory = $countryFactory;
        $this->countryRepository = $countryRepository;
        $this->countryNameConverter = $countryNameConverter;
    }

    /**
     * @Given /^the store has disabled country "([^"]*)"$/
     */
    public function theStoreHasDisabledCountry($countryName) // This method save country in data base.
    {
        $country = $this->createCountryNamed(trim($countryName));
        $country->disable();

        $this->sharedStorage->set('country', $country);
        // Shared storage is an helper service for transferring objects between steps.
        // There is also SharedStorageContext which use this helper service to transform sentences like "(this country), (it), (its), (theirs)" into Country Object.

        $this->countryRepository->add($country);
    }

    private function createCountryNamed(string $name): CountryInterface
    {
        /** @var CountryInterface $country */
        $country = $this->countryFactory->createNew();
        $country->setCode($this->countryNameConverter->convertToCode($name));

        return $country;
    }
}
```
