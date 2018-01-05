# APIs and Service Contracts

Magento is highly modular software and often to achieve some functionality multiple modules need to be involved.
In order to decrease relying on specific implementations of such modules service contracts were introduced in Magento 2.

Service contracts are expressed as PHP interfaces that can be depend upon.
This way implementations can be swapped easily as long as they do not brake service contract.
This allows module creators to offer different implementations of base modules that offer different performance - often used example is storing product data in MongoDB instead of MySQL compatible database or using search engines like Apache Solr or ElasticSearch to implement better catalog search experience.

All module functionality that could be accessed by other modules (or even external systems) should be exposed as service contract.
Service contracts should be defined within `Api` directory so they could be easily located by other developers.
Take note that data structures used by those service contracts should be service contracts as well - we define them in `Api/Data` directory and since those are "data contracts" they expose only getters and setters.

## Service Contract Example

First example is our data service contract.
It will provide exactly the same structure as our database table (note that we do not allow to set creation date or internal ID as this should not change).

Apart of getters and setters it is good idea to define constants that will represent column names in database for future use in our code internals.

```php
<?php

namespace MMAcademy\MicroReview\Api\Data;

interface ReviewInterface
{
    const REVIEW_ID = 'review_id';
    const PRODUCT_ID = 'product_id';
    const MESSAGE = 'message';
    const CREATED_AT = 'created_at';

    /**
     * @return int
     */
    public function getReviewId();

    /**
     * @return int
     */
    public function getProductId();

    /**
     * @param int $productId
     * @return self
     */
    public function setProductId($productId);

    /**
     * @return string
     */
    public function getMessage();

    /**
     * @param string $message
     * @return self
     */
    public function setMessage($message);

    /**
     * @return string
     */
    public function getCreatedAt();
}
```

In this example we will define three simple methods:
* `add` to create new reviews based on product ID and review body
* `delete` to allow removal of reviews
* `getRecent` to access five most recent reviews 

> Note that we did not import any namespaces using `use` operator here in order to simply documentation comments.
> PHP can access documentation comments using reflection mechanism and read them as string.
> This allows Magento to parse those comments - later on we can expose service contracts as SOAP or REST webservices and annotations like `@param` or `@return` will be used to describe those services.
> Do not use `use` operator in service contracts because those will not be available to Magento when parsing docblocks leading to problems with exposing new API (Magento will not be able to find proper type of argument or return type)

```php
<?php

namespace MMAcademy\MicroReview\Api;

interface ReviewRepositoryInterface
{
    /**
     * @param int $productId
     * @param string $message
     * @return boolean
     */
    public function add($productId, $message);

    /**
     * @param int $reviewId
     * @return boolean
     */
    public function delete($reviewId);

    /**
     * @return \MMAcademy\MicroReview\Api\Data\ReviewInterface[]
     */
    public function getRecent();
}

```

## Service Contract Implementation

Obvious implementation of our data service contract is our model that is already capable of holding review data.
In order to be fully compliant with interface we have to explicitly implement all getters and setters, but our model already had methods to access any data - `getData($attribute)` and `setData($attribute, $value)` methods.
Here we can also use previously declared constants.

In order to propagate creation date into our object we have overloaded `beforeSave` method - it is called automatically by resource model before persiting object in database.

```php
<?php

namespace MMAcademy\MicroReview\Model;

use Magento\Framework\Model\AbstractModel;
use MMAcademy\MicroReview\Api\Data\ReviewInterface;

class Review extends AbstractModel implements ReviewInterface
{
    public function getReviewId()
    {
        return $this->getData(ReviewInterface::REVIEW_ID);
    }

    public function getProductId()
    {
        return $this->getData(ReviewInterface::PRODUCT_ID);
    }

    public function setProductId($productId)
    {
        $this->setData(ReviewInterface::PRODUCT_ID, $productId);

        return $this;
    }

    public function getMessage()
    {
        return $this->getData(ReviewInterface::MESSAGE);
    }

    public function setMessage($message)
    {
        $this->setData(ReviewInterface::MESSAGE, $message);

        return $this;
    }

    public function getCreatedAt()
    {
        return $this->getData(ReviewInterface::CREATED_AT);
    }

    protected function _construct()
    {
        $this->_init(ResourceModel\Review::class);
    }

    public function beforeSave()
    {
        parent::beforeSave();
        if ($this->isObjectNew()) {
            $this->setData(ReviewInterface::CREATED_AT, time());
        }

        return $this;
    }
}
```

With implementation of review repository our first step is require using constructor dependency injection factory objects related to our model and collection.
Note that we never implemented such classes - Magento will autogenerate factory classes for You.
Use of factory class in this place is important because we will operate on object that have their state tied to database entry, but all classes injected via constructor dependency are singletons (You can configure an exception, but this will be less readable solution).
Autogenerated factory classes will have `create` method that will return new instance of given class.

Underlying implementations are quite easy and straigh forward.

For new review creation we create new instance of review object, assign it all required information and persist it using it's resource model.

For deletion we acquire new instance of review object, load it from database to ensure it exists and then use resource model to delete it from database.

With recent reviews we acquire new instance of collection object.
Then we have to apply on it our filters, in this case it is only sort order and limiting number of results to five.
But we could use methods like `addFieldToFilter` to pass more complicated constraints.
As soon as we call `getItems` all constraints are gathered, database query is composed and executed.

```php
<?php

namespace MMAcademy\MicroReview\Model\ResourceModel;

use MMAcademy\MicroReview\Api\Data\ReviewInterface;
use MMAcademy\MicroReview\Api\ReviewRepositoryInterface;
use MMAcademy\MicroReview\Model\ResourceModel\Review\Collection;
use MMAcademy\MicroReview\Model\ResourceModel\Review\CollectionFactory;
use MMAcademy\MicroReview\Model\Review;
use MMAcademy\MicroReview\Model\ReviewFactory;

class ReviewRepository implements ReviewRepositoryInterface
{
    /**
     * @var ReviewFactory
     */
    private $reviewFactory;
    /**
     * @var CollectionFactory
     */
    private $collectionFactory;

    public function __construct(ReviewFactory $reviewFactory, CollectionFactory $collectionFactory)
    {
        $this->reviewFactory = $reviewFactory;
        $this->collectionFactory = $collectionFactory;
    }

    /**
     * @param int    $productId
     * @param string $message
     * @return boolean
     * @throws \Exception
     */
    public function add($productId, $message)
    {
        /** @var Review $review */
        $review = $this->reviewFactory->create();
        $review->setMessage($message);
        $review->setProductId($productId);
        $review->getResource()->save($review);

        return true;
    }

    /**
     * @param int $reviewId
     * @return boolean
     * @throws \Exception
     */
    public function delete($reviewId)
    {
        /** @var Review $review */
        $review = $this->reviewFactory->create();
        $review->getResource()->load($review, $reviewId);
        $review->getResource()->delete($review);

        return true;
    }

    /**
     * @return \MMAcademy\MicroReview\Api\Data\ReviewInterface[]
     */
    public function getRecent()
    {
        /** @var Collection $collection */
        $collection = $this->collectionFactory->create();
        $collection->setOrder(ReviewInterface::CREATED_AT);
        $collection->setPageSize(5);

        return $collection->getItems();
    }
}
```

Final step is to register our implementations as proper implementations of our service contract.
In order to do so we need to create file `etc/di.xml` with following content.

Note that preference node for given service contract could be defined in multiple modules.
Definition provided in last loaded module will be used (this is how we swap service contract implementation without altering other modules).


```xml
<?xml version="1.0"?>

<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:ObjectManager/etc/config.xsd">
    <preference
            for="MMAcademy\MicroReview\Api\ReviewRepositoryInterface"
            type="MMAcademy\MicroReview\Model\ResourceModel\ReviewRepository"/>
    <preference
            for="MMAcademy\MicroReview\Api\Data\ReviewInterface"
            type="MMAcademy\MicroReview\Model\Review"/>
</config>
```

## Exposing service contracts as SOAP/REST APIs

As stated before service contracts can be exposed as SOAP or REST APIs very easily.
In order to do so we only need to assign RESTful request paths to proper methods in `etc/webapi.xml` file.

In this example we exposed all methods of our service contracts.
In order to simply this example we left them accessible for everyone (by assigning them resource `anonymous`), but there are methods for fine controlling access to specific methods.

```xml
<?xml version="1.0"?>
<routes xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:noNamespaceSchemaLocation="urn:magento:module:Magento_Webapi:etc/webapi.xsd">

    <route url="/V1/micro_review" method="PUT">
        <service class="MMAcademy\MicroReview\Api\ReviewRepositoryInterface" method="add"/>
        <resources>
            <resource ref="anonymous"/>
        </resources>
    </route>
    <route url="/V1/micro_review/recent" method="GET">
        <service class="MMAcademy\MicroReview\Api\ReviewRepositoryInterface" method="getRecent"/>
        <resources>
            <resource ref="anonymous"/>
        </resources>
    </route>
    <route url="/V1/micro_review/:reviewId" method="DELETE">
        <service class="MMAcademy\MicroReview\Api\ReviewRepositoryInterface" method="delete"/>
        <resources>
            <resource ref="anonymous"/>
        </resources>
    </route>
</routes>

```

Magento comes bundled with Swagger UI - REST API visualisation tool.
We can use it to see all REST API endpoint end even more - execute them!
Go to `/swagger` page of Your magento store.
If You search by `review` You should find `mMAcademyMicroReviewReviewRepositoryV1` service.
You can see there all methods exposed by this service contract and execute them by pressing "Try it out!".
This will show You request target, body, response and even `curl` call example that will execute the same request.

If You want to access SOAP API name of service we found will be required.
Because there are a lot of services that are exposed one WSDL file to describe them all would be gigantic.
To avoid problems with big WSDL files we can specify what services we need when requesting WSDL file.
Now try to download WSDL file prom Your magento store installation at `/soap?services=mMAcademyMicroReviewReviewRepositoryV1&wsdl`.
(You can specify more than one service by just separating their names with comma)

