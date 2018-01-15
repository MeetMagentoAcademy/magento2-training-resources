# Fun with Dependency Injection

Magento 2 uses constructor based dependency injection.
I also comes with it's own implementation of dependency injection container.
In `etc/di.xml` file we can not only specify preferences for specific interfaces:
* we can define plugin methods that can intercept public method executions (altering parameters, return values or even completely skip method execution)
* alter parameters injected to given types, more over define new types that can be injected using existing types with custom attribute configuration 

### Plugin Example

In plugin/interceptor example let assume that we need to capitalize all product names without changing data in our database.
We can use after interceptor to change result of `\Magento\Catalog\Api\Data\ProductInterface::getName` in order to capitalize product names.
This will affect all implementation of `\Magento\Catalog\Api\Data\ProductInterface` - assuming all modules installed in our store follow guidelines this should cover all cases of accessing product name throughout the store.


In `etc/di.xml` we will add new node called `type` to indicate class we want to intercept.
Using `plugin` node we will define out interceptor class.

```xml
    <type name="Magento\Catalog\Api\Data\ProductInterface">
        <plugin name="capitalizeName" type="MMAcademy\MicroReview\Plugin\Product" />
    </type>
 ```
 
Body of our class is very simple: we define one method that accepts instance of type we are intercepting - all types of plugin take instance of currently intercepted object as first parameter - and result of intercepted method as second parameter.
All we have to do is return modified result.

```php
<?php

namespace MMAcademy\MicroReview\Plugin;

class Product
{
    /**
     * @param \Magento\Catalog\Api\Data\ProductInterface $product
     * @param                                            $name
     * @SuppressWarnings(PHPMD.UnusedFormalParameter)
     * @return array
     */
    public function afterGetName(\Magento\Catalog\Api\Data\ProductInterface $product, $name)
    {
        return mb_strtoupper($name, 'UTF-8');
    }
}
```

This was example of `after` plugin that changes method result.
We can also see `before` plugin that changes method parameters (all parameters are passed as parameters to our method, and we must return modified parameters as array).
And `around` plugin that is called instead of executing method, we can still execute this method as closure is passed as second parameter and original parameters follows.
With around plugins we can replicate `before` and `after` plugins but also completely change execution chain.

[See official documentation](http://devdocs.magento.com/guides/v2.2/extension-dev-guide/plugins.html)

### Configurable object via dependency injection

Let us consider how to design highly optimized product data fetcher.
Based on what we have seen in [EAV](performance/eav.md) section - we would need an option to provide list of attributes we would like to be fetched.
More over we would like to bulk fetch some information like product URL redirects or stock status that are not part of EAV data storage.

Let me suggest following class structure
```php
<?php

class ProductDataFetcher {
    
    public function __construct(\Magento\Framework\App\ResourceConnection $resourceConnection, $attributes = [], $extra = []) {}
    
    public function fetch($productIds) {}
}
```
What we have here is class with method expecting array of product IDs in order to fetch them from database.
But take closer look on constructor parameters:
* we have `\Magento\Framework\App\ResourceConnection` connection manager object that will allow us to obtain database connection object for direct database access
* we have `attributes` array of attribute names that we want to fetch
* we have `extra` array of object that will be responsible for fetching extra data

Having such class definition allows us to define what attributes will be processed by given product data fetcher instance and what enhancers we need to use (not all cases require us to fetch stock status, product URL or product prices)

Now we can use `virtualType` in dependency injection configuration to define cases we want to use product data fetcher for.
```xml
    <virtualType name="reviewProductFetcher" type="MMAcademy\MicroReview\Fetcher\Product">
        <arguments>
            <argument name="attributes" xsi:type="array">
                <item name="name" xsi:type="string">name</item>
                <item name="description" xsi:type="string">description</item>
                <item name="name" xsi:type="string">name</item>
            </argument>
            <argument name="extra" xsi:type="array">
                <item name="price" xsi:type="object">MMAcademy\MicroReview\Fetcher\Product\PriceEnhancer</item>
                <item name="url" xsi:type="object">MMAcademy\MicroReview\Fetcher\Product\UrlEnhancer</item>
            </argument>
        </arguments>
    </virtualType>
```

Here we did declare virtual instance of our class.
Now we have a names instance with specific constructor parameters.
Note that using `xsi:type` attribute we can control what type is being injected - especially with argument `extra` and type `object` we can specify class name and object of that class will be injected in this place.

Now we can specify that block that is expecting instance of our product data fetcher should be given specific pre configured instance.
```xml
    <type name="MMAcademy\MicroReview\Block\Recent">
        <arguments>
            <argument name="fetcher" xsi:type="object">reviewProductFetcher</argument>
        </arguments>
    </type>
```

[See official documentation](http://devdocs.magento.com/guides/v2.2/extension-dev-guide/build/di-xml-file.html)
