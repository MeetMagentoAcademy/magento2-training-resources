# Theme inheritance

Theme inheritance enables you to easily extend themes and minimize the maintenance efforts. You can use an existing theme as a basis for customizations, or minor store design updates, like holidays decoration. Rather than copy extensive theme files and modify what you want to change, you can add overriding and extending files.

Theme inheritance is based on the fallback mechanism, which guarantees that if a view file is not found in the current theme, the system searches in the ancestor themes, module view files or library.

The fallback order is slightly different for static assets (CSS, JavaScript, fonts and images) and other theme files, layouts and templates.

## Set a parent theme

```xml
<theme xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:Config/etc/theme.xsd">
     <title>MMA Theme</title>
     <parent>Magento/blank</parent>
     <media>
         <preview_image>media/preview.jpg</preview_image>
     </media>
 </theme>
 ```

## Override static assets and templates

Static assets, or static view files, are styles, JavaScript, images, and fonts.

To customize static view files defined in the parent theme, module view, or library files, you can override them by adding a file with the same name in the relevant location according to the fallback schemes described further. This also refers to the `.less` files, which technically are not static assets.


## Extend layouts

The layouts processing mechanism does not involve fallback. The system collects layout files in the following order:

- Current theme layouts: `<theme_dir>/<Vendor>_<Module>/layout/`
- Ancestor themes layouts, starting from the most distant ancestor, recursively until a theme with no parent is reached: `<parent_theme_dir>/<Vendor>_<Module>/layout/`
- Module layouts for the frontend area: `<module_dir>/view/frontend/layout/`
- Module layouts for the base area: `<module_dir>/view/base/layout/`


## Override layouts

Though overriding layouts is not recommended, it is still possible, and might be a solution for certain customization tasks.

### To override the instructions from an ancestor theme layout file:
Create a layout file with the same name in the `<theme_dir>/<Vendor>_<Module>/layout/override/theme/<Vendor>/<ancestor_theme>` directory.

### To override module layout instructions (base layout):
Create a layout file with the same name in the `<theme_dir>/<Vendor>_<Module>/layout/override/base` directory.
