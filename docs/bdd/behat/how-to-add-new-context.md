# How to add a new context?

Thanks to symfony autowiring, most of your contexts are ready to use.

But if you need to manually route an argument, it is needed to add a service in ``config/services_test.yaml`` file.

```yaml
App\Tests\Behat\Context\CONTEXT_CATEGORY\CONTEXT_NAME:
    arguments:
        $specificArgument: App\SpecificArgument
```

Then you can use it in your suite configuration:

````yaml
default:
    suites:
        SUITE_NAME:
            contexts:
                - 'App\Tests\Behat\Context\CONTEXT_CATEGORY\CONTEXT_NAME'
            filters:
                tags: "@SUITE_TAG"
````    

<div markdown="1" class="block-note">
The context categories are usually one of ``cli``, ``hook``, ``setup``, ``transform``, ``ui``.
</div>
