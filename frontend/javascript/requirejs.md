# AMD modules and RequireJS

Magento uses AMD (asynchronous module definition) approach for JavaScript modules loading. Namely, Magento uses RequireJS and its standard syntax.

## RequireJS configuration location

As Magento has a modular architecture we have an ability to define requirejs-config.js for each module, separately for each area: frontend or admin. (Or base if it is same for both, frontend and admin).

Following is the conventional location of requirejs-config.js (RequireJS configuration file):

- For modules: `<Module_dir>/view/<area>/requirejs-config.js`
- For themes: `<theme_dir>/requirejs-config.js`
- For modules in themes: `<theme_dir>/<module_namespace>/requirejs-config.js` - recommended way of doing any custiomization


## Example of usage
Lets look at an example, the Catalog module. In the `<Magento_Catalog_module_dir>/view/base/requirejs-config.js` file we see the configuration variable:

```js
var config = {
    map: {
        '*': {
            categoryForm:       'Magento_Catalog/catalog/category/form',
            newCategoryDialog:  'Magento_Catalog/js/new-category-dialog',
            categoryTree:       'Magento_Catalog/js/category-tree',
            productGallery:     'Magento_Catalog/js/product-gallery',
            baseImage:          'Magento_Catalog/catalog/base-image-uploader',
            productAttributes:  'Magento_Catalog/catalog/product-attributes'
        }
    },
    deps: [
        'Magento_Catalog/catalog/product'
    ]
};
```

The config variable contains properties with the `map` and `deps` keys. These properties are equivalent to the native RequireJS properties. For example, in this case the map property contains an object with the keys that are aliases to files and values that are real paths to files.
