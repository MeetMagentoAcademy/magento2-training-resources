# Creating Magento2 modules

## `composer.json`

All modern PHP applications should use [composer](https://getcomposer.org).
Composer is package manager for PHP.
It allows easy management and installation of all required dependencies.

Good point to start with module creation is to create composer description for module we are creating - `composer.json` file.

> Although this step is not required when modules are instsalled by putting them in `app/code` directory it is good practice to prepare those files beforehand.

Here is minimalistic `composer.json` file:

```
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



