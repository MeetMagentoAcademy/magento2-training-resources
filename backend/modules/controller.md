# Handling HTTP requests

Every application request in Magento is handled by front controller.
(There is posibility to define multiple front controllers - for example admin panel uses dedicated front controller, but in common usecases this is not necessery)
It will parse request URI and decide how it will be processed.

To simplify we can assume that every request URI has the same strucutre:
```
/route/controller/action
```

Based on those three variables front controller locates corresponding action class that wil handle that request.

## Defining route

Routes definitions can be places in file `etc/frontend/routes.xml` 

```xml
<?xml version="1.0"?>

<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:App/etc/routes.xsd">
    <router id="standard">
        <route id="hello" frontName="hello">
            <module name="MMAcademy_Hello" />
        </route>
    </router>
</config>
```

In `rotuer` node we see reference to `standard` front controller.
In node `route` we can see attribute `frontName` - value of this attribute will be matched against `route` variable in request URI.
Within `route` node we can define multiple `module` nodes and their prority.
For every module standard router will attempt to initialize class `\{ModuleVendor}\{ModuleName}\Controller\{{ContollerVariable}\{ActionVariable}Action`.
Based on our module example for URI `/hello/index/index` front controller will attempt to load folowing action class: `\MMAcademy\Hello\Controller\Index\IndexAction`.

### But not all URI in Magento are build from three words.

We can see standard controller routes that have less than three variables in URI.
Every omitted variable defaults to value `index`.
So action corresponding with URI `/hello` is identical with `/hello/index/index`.
And on the same basis stores home page is special case of route `/index/index/index`.

But there can also be more three secrions in URI and it will still be valid standard controller route.
Additional parameters can be added to URI as pair of `key/value`.
This is practically identical to standard GET params but allow more readability.
Common example is path to specified product page: `/catalog/product/view/id/3231`.

But we have seen that product URL can be "nice" Search Engine Optimized values.
URL rewrite mechanisms allow to define pairs of "nice" URLs and internal URIs.
Apart from `standard` controller or `adminhtml` controller there is `rewrite` controller.
Every request attempts to mach with every controller defined in system in order they are defined.
When request is processed by `rewrite` controller and it matches one of definced URL pair request is internally modified to point to coresponding internal address.

## Finaly `Hello world!`



