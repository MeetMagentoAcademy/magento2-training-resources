# Profiler

Magento2 comes with build-in profiler that could be easily used to track performance issues.

## Enabling profiler

Easiest way to enable profiler is to set enviromental variable `MAGE_PROFILE`.
In orer to achieve that edit and reload nginx configuration.
Adding `fastcgi_param MAGE_PROFILER 1;` in location matching `index.php` path.
With profiler enabled You will see large table appended to end of every page.

## Bad code example

Now let us add some bad code example to our system so we can see how to use profiler to pin-point possible performance problems.

Following product reviews example let's add to the main page block displaying five recent reviews and corresponding products.

Start with creating block class that will use previosly implemented service contract to fetch recent reviews and acquire products related to those reviews.

```php
<?php

namespace MMAcademy\MicroReview\Block;

use Magento\Catalog\Api\Data\ProductInterface;
use Magento\Catalog\Api\ProductRepositoryInterface;
use Magento\Framework\View\Element\Template;
use MMAcademy\MicroReview\Api\ReviewRepositoryInterface;

class Recent extends Template
{
    /**
     * @var ReviewRepositoryInterface
     */
    private $reviewRepository;
    /**
     * @var ProductRepositoryInterface
     */
    private $productRepository;

    public function __construct(
        Template\Context $context,
        ReviewRepositoryInterface $reviewRepository,
        ProductRepositoryInterface $productRepository,
        array $data = []
    ) {
        parent::__construct($context, $data);
        $this->reviewRepository = $reviewRepository;
        $this->productRepository = $productRepository;
    }

    protected $_template = 'MMAcademy_MicroReview::recent.phtml';

    public function getRecent()
    {
        return $this->reviewRepository->getRecent();
    }

    /**
     * @param int $productId
     * @return ProductInterface
     */
    public function getProduct($productId)
    {
        return $this->productRepository->getById($productId);
    }
}
```

As You can see we used constructor dependency injection to require two service contracts: review repository and product repository.
Note that we call parent constructor of class we are extending.
This comes with additional constructor parameters that have to be provided based on parent class definition.
As You may remember all prevoius cases when we did declare block class we did not had to override constructor method because all dependencies were provided.
There are a lot of dependencies provided for template block - in order to simplify extending those all dependencies are passed to parent block via context object.

> Be extra careful with Context objects as You will encounter quite a lot of them thorough the system.
> Always verify if namespaces and `use` statements point to correct Context object.

Now let's add simple template that for this block.

```html
<?php /** @var \MMAcademy\MicroReview\Block\Recent $block */ ?>
<h3><?= __('Recent reviews') ?></h3>
<?php foreach ($block->getRecent() as $review): ?>
    <?php /** @var \MMAcademy\MicroReview\Model\Review $review */
    $product = $block->getProduct($review->getProductId())
    ?>
    <div class="review">
        <span class="review-product">
            <a href="<?= $product->getProductUrl() ?>"><?= $product->getName() ?></a>
        </span>
        <span class="review-message">
            <blockquote>
            <?= $block->escapeHtml($review->getMessage()) ?>
            </blockquote>
        </span>
        <span class="review-created_at">
            <?= $block->formatTime($review->getCreatedAt(), \IntlDateFormatter::LONG, true) ?>
        </span>
    </div>
<?php endforeach ?>
```

And don't forget about layout xml file that will enable us to show this block on main page - `view/frontend/layout/cms_index_index.xml`
```xml
<?xml version="1.0"?>

<page xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      layout="3columns"
      xsi:noNamespaceSchemaLocation="urn:magento:framework:View/Layout/etc/page_configuration.xsd">
    <body>
        <referenceContainer name="main">
            <block name="micro_review.recent" class="MMAcademy\MicroReview\Block\Recent"/>
        </referenceContainer>
    </body>
</page>
```

After flushing all caches and ensuring our block displays properly (and displays 5 products - be sure to add some reviews to see similar results as described below) flush only `full_page` cache - this will allow us to analysis of page rendering with warm magento cache and with cold full page cache.

By default profiler shows automatically track all rendered templates, all observer events and some trackers added by core developers.
Let's look at two tracked entries:
* `magento` this tracks almost full execution of magento activity and page rendering - in my case it was almost a second
* `TEMPLATE:/(...)/view/frontend/templates/recent.phtml` this tracks our template rendering - in my case it was almost half a second, so our "bad code" made request to main page almost twice as long!

## Using profiler

Profiling results are displayed as hierarhy so we can see that events or subtemplates were executed during rendering of our block.
We can see `EVENT:magento_catalog_api_data_productinterface_load_after` executed five times while rendering our block - in my case this took about a tenth of a second.
Let's add custom profiler tracker around product details loading as it may be the source of our bad performance.

All we have to do is to add constructor paremter with type of `\Magento\Framework\Profiler` and call on it methods `start($timerName)` and `stop($timerName)` around code we want to measure.
Here is full class code.

```php
<?php

namespace MMAcademy\MicroReview\Block;

use Magento\Catalog\Api\Data\ProductInterface;
use Magento\Catalog\Api\ProductRepositoryInterface;
use Magento\Framework\Profiler;
use Magento\Framework\View\Element\Template;
use MMAcademy\MicroReview\Api\ReviewRepositoryInterface;

class Recent extends Template
{
    /**
     * @var ReviewRepositoryInterface
     */
    private $reviewRepository;
    /**
     * @var ProductRepositoryInterface
     */
    private $productRepository;
    /**
     * @var Profiler
     */
    private $profiler;

    public function __construct(
        Template\Context $context,
        ReviewRepositoryInterface $reviewRepository,
        ProductRepositoryInterface $productRepository,
        Profiler $profiler,
        array $data = []
    ) {
        parent::__construct($context, $data);
        $this->reviewRepository = $reviewRepository;
        $this->productRepository = $productRepository;
        $this->profiler = $profiler;
    }

    protected $_template = 'MMAcademy_MicroReview::recent.phtml';

    public function getRecent()
    {
        return $this->reviewRepository->getRecent();
    }

    /**
     * @param int $productId
     * @return ProductInterface
     */
    public function getProduct($productId)
    {
        $this->profiler->start(__METHOD__);
        $product = $this->productRepository->getById($productId);
        $this->profiler->stop(__METHOD__);

        return $product;
    }
}
```

As we can see in the results we were right.
Calling five time method `getProduct($productId)` is responsible for 99% of block rendering time.

We did find most common Magento perfomance problem - loading database objects in a loop.
This sould be addressed by loading all products before we enter loop using database collection objects.

First let's try using different method of product repository - `getList` to fetch all required products based on their IDs.
In order to do that we need to populate search criteria object with appropriate filter.
To do so, we will iterate over loaded reviews when we load them for the first time and gather all product IDs.
Then we will have to add requirement for search criteria builder factory object (`\Magento\Framework\Api\SearchCriteriaBuilderFactory`) in our construtor - this will allow us to get unique instance of search critera builder that will create final immutable search criteria object.
Next we will call `getList` on product repository interface and store results in an associative array using product IDs as array keys.
Full example code below.

```php
<?php

namespace MMAcademy\MicroReview\Block;

use Magento\Catalog\Api\Data\ProductInterface;
use Magento\Catalog\Api\ProductRepositoryInterface;
use Magento\Framework\Api\SearchCriteriaBuilder;
use Magento\Framework\Api\SearchCriteriaBuilderFactory;
use Magento\Framework\Profiler;
use Magento\Framework\View\Element\Template;
use MMAcademy\MicroReview\Api\Data\ReviewInterface;
use MMAcademy\MicroReview\Api\ReviewRepositoryInterface;

class Recent extends Template
{
    /**
     * @var ReviewRepositoryInterface
     */
    private $reviewRepository;
    /**
     * @var ProductRepositoryInterface
     */
    private $productRepository;
    /**
     * @var Profiler
     */
    private $profiler;
    /**
     * @var ProductInterface[]
     */
    private $products = [];
    /**
     * @var SearchCriteriaBuilderFactory
     */
    private $searchCriteriaBuilderFactory;

    public function __construct(
        Template\Context $context,
        ReviewRepositoryInterface $reviewRepository,
        ProductRepositoryInterface $productRepository,
        SearchCriteriaBuilderFactory $searchCriteriaBuilderFactory,
        Profiler $profiler,
        array $data = []
    ) {
        parent::__construct($context, $data);
        $this->reviewRepository = $reviewRepository;
        $this->productRepository = $productRepository;
        $this->profiler = $profiler;
        $this->searchCriteriaBuilderFactory = $searchCriteriaBuilderFactory;
    }

    protected $_template = 'MMAcademy_MicroReview::recent.phtml';

    public function getRecent()
    {
        $recent = $this->reviewRepository->getRecent();
        if (empty($this->products)) {
            $this->preloadProducts($recent);
        }

        return $recent;
    }

    /**
     * @param int $productId
     * @return ProductInterface
     */
    public function getProduct($productId)
    {
        return $this->products[$productId];
    }

    /**
     * @param ReviewInterface[] $recent
     */
    protected function preloadProducts($recent)
    {
        $this->profiler->start(__METHOD__);
        $productIds = [];
        foreach ($recent as $review) {
            $productIds[] = $review->getProductId();
        }
        /** @var SearchCriteriaBuilder $searchCriteriaBuilder */
        $searchCriteriaBuilder = $this->searchCriteriaBuilderFactory->create();
        $searchCriteria = $searchCriteriaBuilder->addFilter('entity_id', $productIds, 'in')->create();
        $products = $this->productRepository->getList($searchCriteria)->getItems();

        foreach ($products as $product) {
            $this->products[$product->getId()] = $product;
        }
        $this->profiler->stop(__METHOD__);
    }
}
```

This gave us nice improvement - in my case it was 30% faster.

## Next steps

* [Entity Attribute Value explained](eav.md)
* [xhprof - function level profiler for PHP](https://github.com/phacility/xhprof)
* [XDebug - debugger and profiler tool for PHP](https://xdebug.org/)


