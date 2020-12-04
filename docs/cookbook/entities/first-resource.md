# How to configure your first resource

As an example we will take an **Article entity**.

`src/Entity/Article.php`
```php
<?php

declare(strict_types=1);

namespace App\Entity;

use Doctrine\ORM\Mapping as ORM;
use Sylius\Component\Resource\Model\ResourceInterface;
use Symfony\Component\Validator\Constraints as Assert;

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
     *
     * @Assert\NotBlank()
     */
    private $title;

    /**
     * @return string|null
     */
    public function getTitle(): ?string
    {
        return $this->title;
    }

    /**
     * @param string|null $title
     */
    public function setTitle(?string $title): void
    {
        $this->title = $title;
    }
}
```

If you don't add a form type, it uses a [default form type](https://github.com/Sylius/SyliusResourceBundle/blob/master/src/Bundle/Form/Type/DefaultResourceType.php). But it is a good practice to have one.

`src/Form/Type/ArticleType.php`
```php
<?php

declare(strict_types=1);

namespace App\Form\Type;

use App\Entity\Article;
use Symfony\Component\Form\AbstractType;
use Symfony\Component\Form\Extension\Core\Type\TextType;
use Symfony\Component\Form\FormBuilderInterface;
use Symfony\Component\OptionsResolver\OptionsResolver;

class ArticleType extends AbstractType
{
    /**
    * {@inheritdoc}
    */
    public function buildForm(FormBuilderInterface $builder, array $options)
    {
        parent::buildForm($builder, $options);
        $builder
            ->add('title', TextType::class, [
                'label' => 'sylius.ui.title',
            ]);
    }

    /**
    * {@inheritdoc}
    */
    public function getBlockPrefix(): string
    {
        return 'app_article';
    }

    /**
     * {@inheritdoc}
     */
    public function configureOptions(OptionsResolver $resolver): void
    {
        $resolver->setDefaults([
            'data_class' => Article::class
        ]);
    }
}
```

You now have to add it on Sylius Resource configuration.

`config/sylius/resources.yaml`
```yaml
sylius_resource:
    resources:
        app.article:
            classes:
                model: App\Entity\Article
                form: App\Form\Type\ArticleType
```

> Don't forget to synchronize your database using Doctrine Migrations.

You can use these two commands to generate and synchronize your database.

```bash
$ bin/console doctrine:migrations:diff
$ bin/console doctrine:migrations:migrate
```

## Learn More

* [Sylius Resource Bundle](https://github.com/Sylius/SyliusResourceBundle/blob/master/docs/index.md) documentation
* [Doctrine migrations](https://symfony.com/doc/master/bundles/DoctrineMigrationsBundle/index.html) documentation
