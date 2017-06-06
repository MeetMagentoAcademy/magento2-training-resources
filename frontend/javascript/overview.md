# JavaScript Developer Guide

By default, the Magento application uses the RequireJS file and module loader to optimize the time of loading pages with included JavaScript files, and to manage dependencies of JavaScript resources.

## Terms used

### JavaScript component (JS component)
Any separate .js file decorated as AMD module.
### Ui component
JS component located in the Magento_Ui module, in the app/code/Magento/Ui/view directory.
### jQuery UI widget
A JS component/widget provided by jQuery UI library used in Magento.
### jQuery widget
Custom widget created using jQuery UI Widget Factory and decorated as AMD module. Many Magento JS components are jQuery widget.


## JS resources location

In Magento, you can find the JS components on the following levels:

- Library level (`lib/web`). Resources located here are available in any place in Magento.
- Module level (`<module_dir>/view/<areaname>/web`). If the module is enabled, resources added here are available in other modules and themes.
- Theme level, for a particular module (`<theme_dir>/<VendorName>_<ModuleName>/web`). Resources added here are available for inheriting themes.
- Theme level (`<theme_dir>/web`). Resources added here are available for inheriting themes.

**Library level can only contain core Magento resources. Do not put custom JS files in the `lib/web` directory.**
