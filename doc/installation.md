## Installation

### Requirements

We work on stable, supported and up-to-date versions of packages. We recommend you to do the same.

| Package       | Version |
|---------------|---------|
| PHP           | \>8.0   |
| sylius/sylius | 1.12.x  |
| MySQL         | \>= 5.7 |


```bash
composer require sylius/plus-marketplace-suite-plugin
```

1. Add plugin dependencies to your `config/bundles.php` file:
```php
// config/bundles.php

return [
    ...
    Sylius\MarketplaceSuite\SyliusMarketplaceSuitePlugin::class => ['all' => true],
];
```

2. Import required config in your `config/packages/_sylius.yaml` file:
```yaml
# config/packages/_sylius.yaml

imports:
    ...
    - { resource: "@SyliusMarketplaceSuitePlugin/config/config.yaml" }
```

3. Import routing in your `config/routes.yaml` file:

```yaml
# config/routes.yaml

sylius_marketplace_suite:
    resource: "@SyliusMarketplaceSuitePlugin/config/routes.yaml"
```

4. Configure security in `config/packages/security.yaml`:

```yaml
parameters:
    sylius.security.new_api_user_account_vendor_route: "%sylius.security.new_api_user_account_route%/vendor"
    sylius.security.new_api_user_account_vendor_regex: "^%sylius.security.new_api_user_account_vendor_route%"

#    ...

access_control:
#    ...
    - { path: "%sylius.security.shop_regex%/account/vendor/conversations/create", role: ROLE_USER }
    - { path: "%sylius.security.shop_regex%/account/vendor/conversations", role: ROLE_USER }
    - { path: "%sylius.security.shop_regex%/account/vendor/register", role: ROLE_USER }
    - { path: "%sylius.security.shop_regex%/account/vendor", role: ROLE_VENDOR }
    - { path: "%sylius.security.shop_regex%/account", role: ROLE_USER }

    - { path: "%sylius.security.new_api_user_account_vendor_regex%/register", role: ROLE_USER }
    - { path: "%sylius.security.new_api_user_account_vendor_regex%", role: ROLE_VENDOR }
```

5. Add traits to the entities. Don't change anything if you have set `attribute` as default mapping type
```yaml
doctrine:
    orm:
        entity_managers:
            default:
                mappings:
                    App:
                        type: attribute
```

Otherwise change mapping configuration for annotations or orm.xml.

```php
<?php

declare(strict_types=1);

namespace App\Entity\Order;

use Doctrine\Common\Collections\Collection;
use Doctrine\ORM\Mapping as ORM;
use Sylius\Component\Core\Model\Order as BaseOrder;
use Sylius\MarketplaceSuite\Component\Order\Model\OrderInterface;
use Sylius\MarketplaceSuite\Component\Order\Model\OrderTrait;

/**
 * @ORM\Entity
 * @ORM\Table(name="sylius_order")
 */
#[ORM\Entity]
#[ORM\Table(name: 'sylius_order')]
class Order extends BaseOrder implements OrderInterface
{
    use OrderTrait;

    #[ORM\ManyToOne(targetEntity: self::class, inversedBy: 'secondaryOrders')]
    #[ORM\JoinColumn(onDelete: 'SET NULL')]
    protected ?OrderInterface $primaryOrder = null;

    /** @var Collection<int, OrderInterface> */
    #[ORM\OneToMany(mappedBy: 'primaryOrder', targetEntity: self::class)]
    protected Collection $secondaryOrders;

}
```
```php
<?php

declare(strict_types=1);

namespace App\Entity\Order;

use Doctrine\ORM\Mapping as ORM;
use Sylius\Component\Core\Model\OrderItem as BaseOrderItem;
use Sylius\MarketplaceSuite\Component\Order\Model\OrderItemInterface;
use Sylius\MarketplaceSuite\Component\Order\Model\OrderItemTrait;

/**
 * @ORM\Entity
 * @ORM\Table(name="sylius_order_item")
 */
#[ORM\Entity]
#[ORM\Table(name: 'sylius_order_item')]
class OrderItem extends BaseOrderItem implements OrderItemInterface
{
    use OrderItemTrait;
}
```
```php
<?php

declare(strict_types=1);

namespace App\Entity\Product;

use Doctrine\ORM\Mapping as ORM;
use Sylius\Component\Core\Model\Product as BaseProduct;
use Sylius\Component\Product\Model\ProductTranslationInterface;
use Sylius\MarketplaceSuite\Component\Product\Entity\ProductInterface;
use Sylius\MarketplaceSuite\Component\Product\Model\ProductTrait;

/**
 * @ORM\Entity
 * @ORM\Table(name="sylius_product")
 */
#[ORM\Entity]
#[ORM\Table(name: 'sylius_product')]
class Product extends BaseProduct implements ProductInterface
{
    use ProductTrait;

    protected function createTranslation(): ProductTranslationInterface
    {
        return new ProductTranslation();
    }
}
```

```php
<?php

declare(strict_types=1);

namespace App\Entity\Shipping;

use Doctrine\ORM\Mapping as ORM;
use Sylius\Component\Core\Model\Shipment as BaseShipment;
use Sylius\MarketplaceSuite\Component\Order\Model\ShipmentInterface;
use Sylius\MarketplaceSuite\Component\Order\Model\ShipmentTrait;

/**
 * @ORM\Entity
 * @ORM\Table(name="sylius_shipment")
 */
#[ORM\Entity]
#[ORM\Table(name: 'sylius_shipment')]
class Shipment extends BaseShipment implements ShipmentInterface
{
    use ShipmentTrait;
}
```

```php
<?php

declare(strict_types=1);

namespace App\Entity\User;

use Doctrine\ORM\Mapping as ORM;
use Sylius\Component\Core\Model\ShopUser as BaseShopUser;
use Sylius\MarketplaceSuite\Component\Vendor\Model\ShopUserInterface;
use Sylius\MarketplaceSuite\Component\Vendor\Model\ShopUserTrait;

/**
 * @ORM\Entity
 * @ORM\Table(name="sylius_shop_user")
 */
#[ORM\Entity]
#[ORM\Table(name: 'sylius_shop_user')]
class ShopUser extends BaseShopUser implements ShopUserInterface
{
    use ShopUserTrait;
}
```

6. Override sylius repositories
```php
<?php

declare(strict_types=1);

namespace App\Repository;

use Sylius\Bundle\ChannelBundle\Doctrine\ORM\ChannelRepository as BaseChannelRepository;
use Sylius\MarketplaceSuite\Component\Settlement\Repository\ChannelRepositoryInterface;
use Sylius\MarketplaceSuite\Component\Settlement\Repository\ChannelRepositoryTrait;

class ChannelRepository extends BaseChannelRepository implements ChannelRepositoryInterface
{
    use ChannelRepositoryTrait;
}
```

```php
<?php

declare(strict_types=1);

namespace App\Repository;

use Sylius\Bundle\CoreBundle\Doctrine\ORM\CustomerRepository as BaseCustomerRepository;
use Sylius\MarketplaceSuite\Component\Vendor\Repository\CustomerRepositoryInterface;
use Sylius\MarketplaceSuite\Component\Vendor\Repository\CustomerRepositoryTrait;

class CustomerRepository extends BaseCustomerRepository implements CustomerRepositoryInterface
{
    use CustomerRepositoryTrait;
}
```

```php
<?php

declare(strict_types=1);

namespace App\Repository;

use Sylius\Bundle\CoreBundle\Doctrine\ORM\OrderRepository as BaseOrderRepository;
use Sylius\MarketplaceSuite\Component\Order\Repository\OrderRepositoryInterface as OrderComponentRepositoryInterface;
use Sylius\MarketplaceSuite\Component\Order\Repository\OrderRepositoryTrait as OrderComponentRepositoryTrait;
use Sylius\MarketplaceSuite\Component\Product\Repository\OrderRepositoryInterface as ProductComponentRepositoryInterface;
use Sylius\MarketplaceSuite\Component\Product\Repository\OrderRepositoryTrait as ProductComponentRepositoryTrait;

class OrderRepository extends BaseOrderRepository implements OrderComponentRepositoryInterface, ProductComponentRepositoryInterface
{
    use OrderComponentRepositoryTrait;
    use ProductComponentRepositoryTrait;
}
```

```php
<?php

declare(strict_types=1);

namespace App\Repository;

use Sylius\Bundle\CoreBundle\Doctrine\ORM\PaymentRepository as BasePaymentRepository;
use Sylius\MarketplaceSuite\Component\Order\Repository\PaymentRepositoryInterface;
use Sylius\MarketplaceSuite\Component\Order\Repository\PaymentRepositoryTrait;

class PaymentRepository extends BasePaymentRepository implements PaymentRepositoryInterface
{
    use PaymentRepositoryTrait;
}
```

```php
<?php

declare(strict_types=1);

namespace App\Repository;

use Sylius\Bundle\CoreBundle\Doctrine\ORM\ProductRepository as BaseProductRepository;
use Sylius\MarketplaceSuite\Component\Product\Repository\ProductRepositoryInterface;
use Sylius\MarketplaceSuite\Component\Product\Repository\ProductRepositoryTrait;

class ProductRepository extends BaseProductRepository implements ProductRepositoryInterface
{
    use ProductRepositoryTrait;
}
```

```php
<?php

declare(strict_types=1);

namespace App\Repository;

use Sylius\Bundle\CoreBundle\Doctrine\ORM\ProductReviewRepository as BaseProductReviewRepository;
use Sylius\MarketplaceSuite\Component\Product\Repository\ProductReviewRepositoryInterface;
use Sylius\MarketplaceSuite\Component\Product\Repository\ProductReviewRepositoryTrait;

class ProductReviewRepository extends BaseProductReviewRepository implements ProductReviewRepositoryInterface
{
    use ProductReviewRepositoryTrait;
}
```

```php
<?php

declare(strict_types=1);

namespace App\Repository;

use Sylius\Bundle\CoreBundle\Doctrine\ORM\ProductVariantRepository as BaseProductVariantRepository;
use Sylius\MarketplaceSuite\Component\Product\Repository\ProductVariantRepositoryInterface;
use Sylius\MarketplaceSuite\Component\Product\Repository\ProductVariantRepositoryTrait;

class ProductVariantRepository extends BaseProductVariantRepository implements ProductVariantRepositoryInterface
{
    use ProductVariantRepositoryTrait;
}
```

```php
<?php

declare(strict_types=1);

namespace App\Repository;

use Sylius\Bundle\CoreBundle\Doctrine\ORM\ShipmentRepository as BaseShipmentRepository;
use Sylius\MarketplaceSuite\Component\Order\Repository\ShipmentRepositoryInterface;
use Sylius\MarketplaceSuite\Component\Order\Repository\ShipmentRepositoryTrait;

class ShipmentRepository extends BaseShipmentRepository implements ShipmentRepositoryInterface
{
    use ShipmentRepositoryTrait;
}
```

```php
<?php

declare(strict_types=1);

namespace App\Repository;

use Sylius\Bundle\TaxonomyBundle\Doctrine\ORM\TaxonRepository as BaseTaxonRepository;
use Sylius\MarketplaceSuite\Component\Vendor\Repository\TaxonRepositoryInterface;
use Sylius\MarketplaceSuite\Component\Vendor\Repository\TaxonRepositoryTrait;

class TaxonRepository extends BaseTaxonRepository implements TaxonRepositoryInterface
{
    use TaxonRepositoryTrait;
}
```

7. Override _sylius configuration
```yaml
sylius_channel:
    resources:
        channel:
            classes:
                model: App\Entity\Channel\Channel
                repository: App\Repository\ChannelRepository

sylius_customer:
    resources:
        customer:
            classes:
                model: App\Entity\Customer\Customer
                repository: App\Repository\CustomerRepository

sylius_order:
    resources:
        order:
            classes:
                model: App\Entity\Order\Order
                repository: App\Repository\OrderRepository

sylius_payment:
    resources:
        payment:
            classes:
                model: App\Entity\Payment\Payment
                repository: App\Repository\PaymentRepository
                
sylius_product:
    resources:
        product:
            classes:
                model: App\Entity\Product\Product
                repository: App\Repository\ProductRepository
            translation:
                classes:
                    model: App\Entity\Product\ProductTranslation
        product_variant:
            classes:
                model: App\Entity\Product\ProductVariant
                repository: App\Repository\ProductVariantRepository

sylius_review:
    resources:
        product:
            review:
                classes:
                    model: App\Entity\Product\ProductReview
                    repository: App\Repository\ProductReviewRepository

sylius_shipping:
    resources:
        shipment:
            classes:
                model: App\Entity\Shipping\Shipment
                repository: App\Repository\ShipmentRepository
                
sylius_taxonomy:
    resources:
        taxon:
            classes:
                model: App\Entity\Taxonomy\Taxon
                repository: App\Repository\TaxonRepository

```

7*. Remove Country entity definition from _sylius configuration
```yaml
sylius_addressing:
    resources:
#       ...
        country:
            classes:
                model: App\Entity\Addressing\Country
```

8. Clear application cache by using command:

```bash
bin/console cache:clear
```

9. Update your database
```bash
bin/console doctrine:migrations:migrate
```

10. Copy required templates into correct directories in your project.

```
- vendor/sylius/plus-marketplace-suite-plugin/templates/bundles/SyliusAdminBundle/Order/Show/Summary/_totals.html.twig
- vendor/sylius/plus-marketplace-suite-plugin/templates/bundles/SyliusAdminBundle/Product/Show/_header.html.twig
- vendor/sylius/plus-marketplace-suite-plugin/templates/bundles/SyliusCoreBundle/Email/Blocks/OrderConfirmation/_content.html.twig
- vendor/sylius/plus-marketplace-suite-plugin/templates/bundles/SyliusUiBundle/Modal/_confirmation.html.twig
- vendor/sylius/plus-marketplace-suite-plugin/templates/bundles/SyliusUiBundle/_flashes.html.twig
- vendor/sylius/plus-marketplace-suite-plugin/templates/bundles/SyliusShopBundle/Taxon/_horizontalMenu.html.twig
- vendor/sylius/plus-marketplace-suite-plugin/templates/bundles/SyliusShopBundle/Register/_header.html.twig
- vendor/sylius/plus-marketplace-suite-plugin/templates/bundles/SyliusShopBundle/ProductReview/create.html.twig
- vendor/sylius/plus-marketplace-suite-plugin/templates/bundles/SyliusShopBundle/Product/_box.html.twig
- vendor/sylius/plus-marketplace-suite-plugin/templates/bundles/SyliusShopBundle/Product/Show/_reviews.html.twig
- vendor/sylius/plus-marketplace-suite-plugin/templates/bundles/SyliusShopBundle/Order/_summary.html.twig
- vendor/sylius/plus-marketplace-suite-plugin/templates/bundles/SyliusShopBundle/Common/Form/_login.html.twig
- vendor/sylius/plus-marketplace-suite-plugin/templates/bundles/SyliusShopBundle/Checkout/_header.html.twig
- vendor/sylius/plus-marketplace-suite-plugin/templates/bundles/SyliusShopBundle/Account/Order/Show/_header.html.twig
```

Optional templates - marketplace instead sylius names, marketplace logos...
```
- vendor/sylius/plus-marketplace-suite-plugin/templates/bundles/SyliusAdminBundle/Layout/_logo.html.twig
- vendor/sylius/plus-marketplace-suite-plugin/templates/bundles/SyliusAdminBundle/Layout/_notification.html.twig
- vendor/sylius/plus-marketplace-suite-plugin/templates/bundles/SyliusAdminBundle/Security/login.html.twig
- vendor/sylius/plus-marketplace-suite-plugin/templates/bundles/SyliusAdminBundle/layout.html.twig
- vendor/sylius/plus-marketplace-suite-plugin/templates/bundles/SyliusCoreBundle/Email/layout.html.twig
- vendor/sylius/plus-marketplace-suite-plugin/templates/bundles/SyliusUiBundle/Layout/centered.html.twig
- vendor/sylius/plus-marketplace-suite-plugin/templates/bundles/SyliusUiBundle/Security/_logo.html.twig
- vendor/sylius/plus-marketplace-suite-plugin/templates/bundles/TwigBundle/Exception
- vendor/sylius/plus-marketplace-suite-plugin/templates/bundles/SyliusShopBundle/Layout/Header/_logo.html.twig
- vendor/sylius/plus-marketplace-suite-plugin/templates/bundles/SyliusShopBundle/Homepage/_banner.html.twig
```

Don't copy
```
- vendor/sylius/plus-marketplace-suite-plugin/templates/bundles/SyliusAdminBundle/_scripts.html.twig
- vendor/sylius/plus-marketplace-suite-plugin/templates/bundles/SyliusAdminBundle/_styles.html.twig
- vendor/sylius/plus-marketplace-suite-plugin/templates/bundles/SyliusShopBundle/_styles.html.twig
- vendor/sylius/plus-marketplace-suite-plugin/templates/bundles/SyliusShopBundle/_scripts.html.twig
- vendor/sylius/plus-marketplace-suite-plugin/templates/bundles/SyliusAdminBundle/Security/_content.html.twig
- vendor/sylius/plus-marketplace-suite-plugin/templates/bundles/SyliusShopBundle/Product/Show/_outOfStock.html.twig
- vendor/sylius/plus-marketplace-suite-plugin/templates/bundles/SyliusShopBundle/Cart/Summary/_items.html.twig
- vendor/sylius/plus-marketplace-suite-plugin/templates/bundles/SyliusShopBundle/login.html.twig
```

11. Add plugin assets to your project

Import plugin's `webpack.config.js` file

```js
// webpack.config.js
const [syliusMarketplaceSuiteShop, syliusMarketplaceSuiteAdmin] = require('./vendor/sylius/plus-marketplace-suite-plugin/webpack.config');
...

module.exports = [..., syliusMarketplaceSuiteShop, syliusMarketplaceSuiteAdmin];
```

Add new packages in `./config/packages/assets.yaml`

```yaml
# config/packages/assets.yaml

framework:
    assets:
        packages:
            # ...
            marketplace_suite_shop:
                json_manifest_path: '%kernel.project_dir%/public/build/sylius/marketplacesuite/shop/manifest.json'
            marketplace_suite_admin:
                json_manifest_path: '%kernel.project_dir%/public/build/sylius/marketplacesuite/admin/manifest.json'
```

Add new build paths in `./config/packages/webpack_encore.yml`

```yaml
# config/packages/webpack_encore.yml

webpack_encore:
    builds:
        # ...
        marketplace_suite_shop: '%kernel.project_dir%/public/build/sylius/marketplacesuite/shop'
        marketplace_suite_admin: '%kernel.project_dir%/public/build/sylius/marketplacesuite/admin'
```

Add encore functions to your templates

```twig
{# @SyliusShopBundle/_scripts.html.twig #}
{{ encore_entry_script_tags('sylius-marketplacesuite-shop', null, 'marketplace_suite_shop') }}

{# @SyliusShopBundle/_styles.html.twig #}
{{ encore_entry_link_tags('sylius-marketplacesuite-shop', null, 'marketplace_suite_shop') }}

{# @SyliusAdminBundle/_scripts.html.twig #}
{{ encore_entry_script_tags('sylius-marketplacesuite-admin', null, 'marketplace_suite_admin') }}

{# @SyliusAdminBundle/_styles.html.twig #}
{{ encore_entry_link_tags('sylius-marketplacesuite-admin', null, 'marketplace_suite_admin') }}
```

Run `bin/console assets:install` and  `yarn encore dev` or `yarn encore production`


### 12. Run the app

```bash
symfony server:start // or symfony serve -d --no-tls
```

### 13. Load fixtures

```bash
vendor/bin/marketplace-fixtures
```

> **NOTE!** If you install the plugin together with other plugins, the above command may not work, use then: 
> `bin/console sylius:fixtures:load`

## Optional steps

### 14. Run tests

Creating database for your test environment.

```bash
bin/console doctrine:database:create --env=test
bin/console doctrine:schema:create --env=test
```

**a)** PHPUnit

```bash
vendor/bin/phpunit --colors=always tests/
```
**b)** PHPSpec

```bash
vendor/bin/phpspec run
```

**c)** PHPStan

```bash
vendor/bin/phpstan analyse -c phpstan.neon -l 8 src/
```

**d)** Behat

```bash
vendor/bin/behat 
```

**e)** Coding Standard

```bash
vendor/bin/ecs check src
```
