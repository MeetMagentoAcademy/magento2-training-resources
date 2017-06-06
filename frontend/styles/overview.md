# Cascading style sheets (CSS)

Magento 2 incorporates LESS, a CSS pre-processor that simplifies the management of complex CSS files. To define styles of a Magento store, you can use both - CSS and LESS stylesheets.

Magento application provides a built-in LESS UI library, which you can optionally extend.

To customize storefront styles, you need to create a custom design theme. Then you can use one of the following approaches:
- If your theme inherits from the Magento out-of-the-box Blank or Luma theme, you can override the default LESS files; for example to change the values of the variables used in the default files.
- Create your own LESS files using the built-in LESS preprocessor.
- Create your own CSS files, optionally having compiled them using third-party CSS preprocessor.

## Warning
*The CSS class names can be assigned in both templates and layouts.*


## LESS compilation modes

1. Server-side LESS compilation.

   This is the default compilation mode, and is the only option in production application mode. In this case the compilation is performed on the server, using the LESS PHP library.

2. Client-side LESS compilation.

   **DO NOT USE IT!**

   When your application is not in the production mode, you can set Magento to compile `.less` files in a browser, using the native `less.js` library

## The `@magento_import` directive

`@magento_import` is a Magento-specific LESS directive that allows including multiple files by a name pattern. It is used to include files with the same name from the different locations, for example, different modules. The standard @import directive includes a single file, which is found according to the static files fallback.

`@magento_import` can be used in the root source files of a theme only.

```less
//  Comment in a LESS document

//  Standard LESS import directive
//  ---------------------------------------------

@import 'source/_reset';
@import '_styles';

//
//  Custom Magento LESS import directives
//  ---------------------------------------------

//@magento_import 'source/_module.less'; // Theme modules
//@magento_import 'source/_widgets.less'; // Theme widgets
//@magento_import 'source/_extend.less'; // Extend for minor customization
```

## `@magento_import` processing

In the scope of static resources preprocessing, the built-in LESS preprocessor does the following:

Searches for all `@magento_import` directives.
Replaces the original `@magento_import` directives with the standard `@import` directives. The latter specify the paths to the particular files that correspond to the pattern specified in `@magento_import`.

Before:
```less
//@magento_import 'source/_widgets.less'; // Theme widgets
```

After:
```less
@import '../Magento_Catalog/css/source/_widgets.less';
@import '../Magento_Cms/css/source/_widgets.less';
@import '../Magento_Reports/css/source/_widgets.less';
@import '../Magento_Sales/css/source/_widgets.less';
 // Theme widgets
```
