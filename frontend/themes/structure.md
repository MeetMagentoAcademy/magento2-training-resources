# Magento theme structure

## Magento theme location

Storefront themes are conventionally located under `app/design/frontend/<Vendor>/`. Though technically they can reside in other directories. For example Magento built-in themes can be located under `vendor/magento/theme-frontend-<theme_code>` when a Magento instance is deployed from the Composer repository.


## Theme components

```
<theme_dir>/
├── <Vendor>_<Module>/
│ ├── web/
│ │ ├── css/
│ │ │ ├── source/
│ ├── layout/
│ │ ├── override/
│ ├── templates/
├── etc/
├── i18n/
├── media/
├── web/
│ ├── css/
│ │	├── source/
│ ├── fonts/
│ ├── images/
│ ├── js/
├── composer.json
├── registration.php
├── theme.xml
```

## Required files
- `theme.xml` - The file is mandatory as it declares a theme as a system component. It contains the basic meta-information, like the theme name and the parent theme name, if the theme is inherited from an existing theme. The file is used by the Magento system to recognize the theme.
- `registration.php` - 	Required to register your theme in the system.
- `media/<some_screenshot>` - 	This directory contains a theme preview (a screenshot of your theme).

## Required for a theme, but optional if exists in the parent theme
- `etc/view.xml` - This file contains images configuration for all storefront product images and thumbnails.

## Unoficaly required files
- `composer.json` - Describes the theme dependencies and some meta-information. Will be here if your theme is a Composer package.

---

## Static view files

A set of theme files that are returned by the server to a browser as is, without any processing, are called the static files of a theme.

The key difference between static files and other theme files is that static files appear on a web page as references to the files, while other theme files take part in the page generation, but are not explicitly referenced on a web page as files.

Static view files that can be accessed by a direct link from the store front, are distinguished as public theme files.


## Dynamic view files

View files that are processed or executed by the server in order to provide result to the client. These are: .less files, templates, and layouts.
