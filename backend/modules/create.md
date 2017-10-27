# Creating Magento2 modules

## `composer.json`

All modern PHP applications should use [composer](https://getcomposer.org).
Composer is package manager for PHP.
It allows easy management and installation of all required dependencies.

Good point to start with module creation is to create composer description for module we are creating - `composer.json` file.

> Although this step is not required when modules are instsalled by putting them in `app/code` directory it is good practice to prepare those files beforehand.

Here is minimalistic `composer.json` file:

```json
{
    "name": "mmacademy/module-hello",
    "description": "Shows Hello World!",
    "require": {
        "magento/framework": "100.1.*"
    },
    "autoload": {
        "files": [
            "registration.php"
        ],
        "psr-4": {
            "MMAcademy\\Hello\\": ""
        }
    }
}
```

Minimal configuration contains:
* module name - this moudle name specifies name of composer package, You should follow convention when naming composer packages with Magento2 modules: `<vendor name>/module-<moudle name>` where Magento2 module name would follow convention: `<Vendor Name>_<Module_Name>`.
* module decription - this is optional step but it is always nice to know what this moudle is supposed to do
* deprendencies in `require` section - here we specified that `magento/framework` in at least version `100.1.0` - this version is shipped with Magento 2.1
* autoload definitions

### Autoload explained

Since PHP 5.2 developer can register methods that will handle discovery and inclusion of class definition files when class not present in current processing context is refrenced.
While autoloading strategies can be implemented easly in modern PHP applications it is common to delegate this task to code generated by composer.
Within `composer.json` files we can defilne few types of autoloading to be performend:
* PSR-0 - where directory structure is mapped to PHP namespace
* PSR-4 - where directory strucutre is mapped to PHP sub-namespace (this standard is aplying composer package constraints onto PSR-0 standard)
* files - where given files are always included at the begining of every dispatched request
* classmap - where given files are analysed by composer tool and mapping of classnames and coresponding filenames is created

Note that composer has options for providing "optimized" autoloaders - this boils down to builing classmap based on PSR-0 and PSR-4 definitions allowing to avoid costly access to disk resources.

## `registration.php`

Magento platform modularity in composer enabled enviroment is fueled by `registration.php` files that are used to register given extension with framework code.
In case of composer installed packages `registartion.php` is executed by composer autoloader (as they are defined in `file` section of composer autoloader definition).
For non-composer installed packages file `app/etc/NonComposerComponentRegistration.php` will attempt to discover all `registartion.php` files in `app/code` directory.

Typical content of `registtration.php` file is as follows:
```php
<?php

\Magento\Framework\Component\ComponentRegistrar::register(
    \Magento\Framework\Component\ComponentRegistrar::MODULE,
    'MMAcademy_Hello',
    __DIR__
);
```

This binds component type (module, theme, language pack or library) its name and its root directory (in order to locate related files).

## `etc/module.xml`

Another required file for module definiition is `etc/modules.xml` file that declares our module depenencies on magento module level and declares setup version.

Node `sequence` defines module dependencies that have to be loaded before our module. Module loading order may impact final configuration state as modules loaded layter may impact configirations and behaviours of modules loaded before them.

Node `setup_version` defines module setup version number - this number is used to detect if changes in database scheme are required.
> There are several strategies for maintaiing version numbers.
> I recomend to keep this version only for database changes purposes and not keeping it in sync with composer package version number.
> Every time Magento detects change in setup version number maintanance break will be required to perform database scheme upgrade.
> If changes introduced to module do not impact database state do not increment setup version number to avoid this costly opration, but be sure that composer version number was incremented accordingly to semantic versioning to indicate code changes made.

```xml
<?xml version="1.0"?>

<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:Module/etc/module.xsd">
    <module name="MMAcademy_Hello" setup_version="1.0.0">
        <sequence>
        </sequence>
    </module>
</config>
```

## Verify Your work

Validity of `composer.json` file can be checkd by composer itself.
Run `php composer.phar validate path/to/composer.json` to check specified composer.json file.

In order to check if our module was properly definied we can run Magento CLI command in order to list all modules found - `php bin/magento module:status`.
We should see that our module was found but is not enabled:
```
...

List of disabled modules:
MMAcademy_Hello
```

Now we can enable our module executing `php bin/magento module:enable MMAcademy_Hello`

## Next Steps 

* [Define controllers and routes to handle HTTP requests](controller.md)