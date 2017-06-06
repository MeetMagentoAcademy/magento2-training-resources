# Use custom JavaScript

This topic discusses how to use custom JavaScript components with the components provided by Magento or having replaced them with custom implementations.

We strongly recommend not changing the source code of default Magento components and widgets. All customizations must be implemented in custom modules or themes.

## Add a custom JS component

1. Place the custom component source file in one of the following locations:
   Your theme JS files: `<theme_dir>/web/js` or `<theme_dir>/<VendorName>_<ModuleName>/web/js`. In this case the component is available in your theme and its child themes.
   Your module view JS files: `<module_dir>/view/frontend/web/js`. In this case the component is available in all modules and themes (if your module is enabled).
2. Optionally, in the corresponding module or theme, create a `requirejs-config.js` configuration file, if it does not yet exist there and set path for your resource.


## Replace a default JS component

To use a custom implementation of an existing Magento JS component:

1. Place the custom component source file
2. Create a RequireJS configuration file `requirejs-config.js`
   ```js
    var config = {
      "map": {
        "*": {
          "menu": "js/navigation-menu",
          "mage/backend/menu": "js/navigation-menu",
        }
      }
    };
   ```
