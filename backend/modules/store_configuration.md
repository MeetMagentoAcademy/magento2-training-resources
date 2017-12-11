# Store configuration

You already know how to [access and modify base store configuration](/administration/part1.md).
Now lets look at how we can create new fields in store configuration section and how to access configuration values from code.

## Defining store configuration fields

Store configuration is defined as tree like structure using `etc/adminhtml/system.xml` files.

Tree structure is as follows:
First level nodes are `sections` - those are left menu items that can be grouped in `tags`.
Second level nodes are `groups` with third level nodes `values`.

Let's improve on our example template by providing message based on configuration - not relying on translations.

We will create 3 fields:
* `message` - this will contain message where no user supplied value is provided
* `can_use_user` - this will be yes/no switch that will allow us to ignore user supplied value
* `message_with_user` - this will contain message with placeholder for user provided value.

```xml
<?xml version="1.0"?>
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:module:Magento_Config:etc/system_file.xsd">
    <system>
        <section id="customer">
            <group id="hello" translate="label" type="text" sortOrder="500" showInDefault="1" showInWebsite="1" showInStore="1">
                <label>Hello</label>
                <field id="message" translate="label comment" type="text" sortOrder="40" showInDefault="1" showInWebsite="1" showInStore="1" canRestore="1">
                    <label>Hello message</label>
                    <comment>This will be shown when no name were provided</comment>
                </field>
                <field id="can_use_user" translate="label comment" type="select" sortOrder="70" showInDefault="1" showInWebsite="1" showInStore="1">
                    <label>Can use user name</label>
                    <comment>If enabled will show user supplied name in message.</comment>
                    <source_model>Magento\Config\Model\Config\Source\Yesno</source_model>
                </field>
                <field id="message_with_name" translate="label comment" type="text" sortOrder="80" showInDefault="1" showInWebsite="1" showInStore="1">
                    <label>Hello message with name</label>
                    <comment>This will be shown when customer provided his name. Type {name} in place where customer name should be placed.</comment>
                </field>
            </group>
        </section>
    </system>
</config>
```

We can see that in existing section `customer` new group `hello` is created.
New group will be visible with label `Hello` but note that we did use attribute `translate` in `group` definition so value of node `label` will be translatable using standard translation file.
Note that every section, group and field can be hidden in given configuration scopes (`showInDefault`, `showInStore`, `showInWebsite`) as well as have sort order assigned.
With fields we can see that attribute `translate` can specify multiple underlying nodes - where apart of label we are also translating comment displayed under configuration field.

Every field has defined `type` for `text` story is quite simple as we only show input field.
With `select` type You can see that we have defined another node named `source_model` that point to a class name that will provide all available options to out select input field - in this case it is simple yes/no.

More over we can define default values using `etc/config.xml` file.
```xml
<?xml version="1.0"?>
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:module:Magento_Store:etc/config.xsd">
    <default>
        <customer>
            <hello>
                <message>Hello World!</message>
                <message_with_name>Hello {name}!</message_with_name>
                <can_use_user>1</can_use_user>
            </hello>
        </customer>
    </default>
</config>
```

As You can see under `default` node we just have to specify outline of configuration tree structure (add node representing section ID than group ID and than field ID) and provide values for given fields.

## Accessing values from code

Below we can see interface to access store configuration.
There are two methods but only difference between them is that `isSetFlag` will always return boolean value.
By default all configuration queries are resolved to current scope, but using second and third parameter we can query inactive configuration scope.

```php
<?php
/**
 * Configuration interface
 *
 * Copyright © 2013-2017 Magento, Inc. All rights reserved.
 * See COPYING.txt for license details.
 */

namespace Magento\Framework\App\Config;

/**
 * @api
 */
interface ScopeConfigInterface
{
    /**
     * Default scope type
     */
    const SCOPE_TYPE_DEFAULT = 'default';

    /**
     * Retrieve config value by path and scope.
     *
     * @param string $path The path through the tree of configuration values, e.g., 'general/store_information/name'
     * @param string $scopeType The scope to use to determine config value, e.g., 'store' or 'default'
     * @param null|string $scopeCode
     * @return mixed
     */
    public function getValue($path, $scopeType = ScopeConfigInterface::SCOPE_TYPE_DEFAULT, $scopeCode = null);

    /**
     * Retrieve config flag by path and scope
     *
     * @param string $path The path through the tree of configuration values, e.g., 'general/store_information/name'
     * @param string $scopeType The scope to use to determine config value, e.g., 'store' or 'default'
     * @param null|string $scopeCode
     * @return bool
     */
    public function isSetFlag($path, $scopeType = ScopeConfigInterface::SCOPE_TYPE_DEFAULT, $scopeCode = null);
}
```

Going back to our example: let's move message related logic from template to block.
We will create new method called `getMessage` that based on configuration and request parameters will return proper message to display.
Fortunately `Magento\Framework\App\ConfigScopeConfigInterface` already is defined as dependency on abstract block class so we can use by accessing property `_scopeConfig`.

```php
<?php

namespace MMAcademy\Hello\Block;

use Magento\Framework\View\Element\Template;

class Hello extends Template
{
    const MESSAGE = 'customer/hello/message';
    const CAN_USE_USER = 'customer/hello/can_use_user';
    const MESSAGE_WITH_USER = 'customer/hello/message_with_name';
    protected $_template = 'MMAcademy_Hello::hello.phtml';

    public function getMessage()
    {
        $name = $this->_request->getParam('name');
        if (!empty($name) && $this->_scopeConfig->isSetFlag(self::CAN_USE_USER)) {
            $message = $this->_scopeConfig->getValue(self::MESSAGE_WITH_USER);
            $message = str_replace('{name}', $name, $message);

            return $message;
        }

        return $this->_scopeConfig->getValue(self::MESSAGE);
    }
}
```

As You can see to improve readability paths for configuration tree were exposed as code constants.
We have also used `str_replace` function to implement simple placeholder replacement functionality.

Now we can see simplified template.
Note that for security reasons we are still performing `escapeHtml` on resulting text.

```php
<?php /** @var \MMAcademy\Hello\Block\Hello $block */ ?>
<h1><?= $block->escapeHtml(__('Hello there')) ?></h1>
<p><?= $block->escapeHtml($block->getMessage()) ?></p>
```

## Next steps

* [Flash messages](flash_message.md)
