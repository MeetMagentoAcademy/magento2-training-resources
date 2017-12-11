# Flash message

Not all controller actions end up rendering page for customer.
Some of them are responsible for performing actions like saving values in database.
Commonly after performing those operation use redirect is performed - for example to page listing details of newly created entry.
In order to communicate result of that action flash messages are often used in wide variety of systems.
Those messages are displayed once to specific client and usually stored within users session data on server.

In Magento messages are managed by `Magento\Framework\Message\ManagerInterface`.
Notable methods that we will be using in this example are:
```php
    /**
     * Adds new error message
     *
     * @param string $message
     * @param string|null $group
     * @return ManagerInterface
     */
    public function addErrorMessage($message, $group = null);

    /**
     * Adds new success message
     *
     * @param string $message
     * @param string|null $group
     * @return ManagerInterface
     */
    public function addSuccessMessage($message, $group = null);
```

Going back to our example - let's add new action controller that will display greeting message with our name as flash message.

```php
<?php

namespace MMAcademy\Hello\Controller\Index;

use Magento\Framework\App\Action\Action;
use Magento\Framework\App\ResponseInterface;
use Magento\Framework\Controller\ResultFactory;

class Message extends Action
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
        /** @var \Magento\Framework\Controller\Result\Redirect $result */
        $result = $this->resultFactory->create(ResultFactory::TYPE_REDIRECT);
        $name = $this->_request->getParam('name');
        if (empty($name)) {
            $result->setUrl('/');
            $this->messageManager->addErrorMessage(__('No name provided'));
        } else {
            $result->setPath('*/index/index', ['name' => $name]);
            $this->messageManager->addSuccessMessage(__('Hello %1', $name));
        }

        return $result;
    }
}
```

We used another type of result for action controller - `Magento\Framework\Controller\Result\Redirect` allows us to return 30x class HTTP code.
As You can see we can set redirection target in two ways:
* explicit redirect URL using `setUrl($url)`
* implicit redirect path using `setPath($path, $params)` - this will use magento routing rules to build redirect URL

Note that `*` in redirect path indicates current scope - setting path `*/*/*` would redirect to exactly the same action controller.

In this example we show different type of message depending on passed argument `name` value.
Error message is displayed if value was empty and success message is shown otherwise.

You should have noticed that we did pass unescaped user supplied value to `messageManager`, but it is safe because messages are escaped automatically when displayed.

