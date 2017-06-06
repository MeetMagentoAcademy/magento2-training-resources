# Common layout customization tasks

## Set the page layout

The type of page layout to be used for a certain page is defined in the page configuration file, in the layout attribute of the root `<page>` node.

Example: Change the layout of Advanced Search page from default `1-column` to `2-column with left bar`. To do this, extend `catalogsearch_advanced_index.xml` in your theme by adding the following layout:

`app/design/frontend/<Vendor>/<theme>/Magento_CatalogSearch/layout/catalogsearch_advanced_index.xml`

```xml
<page layout="2columns-left" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:View/Layout/etc/page_configuration.xsd">
...
</page>
```

## Include static resources (JavaScript, CSS, fonts)

JavaScript, CSS and other static assets are added in the `<head>` section of a page configuration file. The default look of a Magento store page `<head>` is defined by `app/code/Magento/Theme/view/frontend/layout/default_head_blocks.xml`.

The recommended way to add CSS and JavaScript is to extend this file in your custom theme, and add the assets there. The following file is a sample of a file you must add:

`<theme_dir>/Magento_Theme/layout/default_head_blocks.xml`

```xml
<page xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:View/Layout/etc/page_configuration.xsd">
    <head>
        <!-- Add local resources -->
        <css src="css/my-styles.css"/>

        <!-- The following two ways to add local JavaScript files are equal -->
        <script src="Magento_Catalog::js/sample1.js"/>
        <link src="js/sample.js"/>

        <!-- Add external resources -->
        <css src="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.4/css/bootstrap-theme.min.css" src_type="url" />
        <script src="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.4/js/bootstrap.min.js" src_type="url" />
        <link rel="stylesheet" type="text/css" src="http://fonts.googleapis.com/css?family=Montserrat" src_type="url" />
    </head>
</page>
```

When adding external resources, specifying the `src_type="url"` argument value is a must.

If you’d like to include a google webfont, you have to add the `rel="stylesheet" type="text/css"` to the tag, otherwise it won’t work.

You can use either `<link src="js/sample.js"/> or <script src="js/sample.js"/>` instruction to add a locally stored JavaScript file to your theme.

The path to assets is specified relatively to one of the following locations:

- `<theme_dir>/web`
- `<theme_dir>/<Namespace>_<Module>/web`

## Remove static resources (JavaScript, CSS, fonts)

To remove the static resources, linked in a page `<head>`, make a change similar to the following in a theme extending file:

`app/design/frontend/<Vendor>/<theme>/Magento_Theme/layout/default_head_blocks.xml`

```xml
<page xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:View/Layout/etc/page_configuration.xsd">
   <head>
        <!-- Remove local resources -->
        <remove src="css/styles-m.css" />
        <remove src="my-js.js"/>
        <remove src="Magento_Catalog::js/compare.js" />

        <!-- Remove external resources -->
        <remove src="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.4/css/bootstrap-theme.min.css"/>
        <remove src="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.4/js/bootstrap.min.js"/>
        <remove src="http://fonts.googleapis.com/css?family=Montserrat" />
   </head>
</page>
```

Note, that if a static asset is added with a module path (for example `Magento_Catalog::js/sample.js`) in the initial layout, you need to specify the module path as well when removing the asset.

## Create a container

Use the following sample to create (declare) a container:

```xml
<container name="some.container"
           as="someContainer"
           label="Some Container"
           htmlTag="div"
           htmlClass="some-container"
/>
```

## Reference a container

To update a container use the `<referenceContainer>` instruction.

Example: add links to the page header panel.

```xml
<referenceContainer name="header.panel">
    <block class="Magento\Framework\View\Element\Html\Links" name="header.links">
        <arguments>
            <argument name="css_class" xsi:type="string">header links</argument>
        </arguments>
    </block>
</referenceContainer>
```

## Create a block

Blocks are created (declared) using the `<block>`instruction.

Example: add a block with a product SKU information.

```xml
<block class="Magento\Catalog\Block\Product\View\Description" name="product.info.sku" template="product/view/attribute.phtml" after="product.info.type">
    <arguments>
        <argument name="at_call" xsi:type="string">getSku</argument>
        <argument name="at_code" xsi:type="string">sku</argument>
        <argument name="css_class" xsi:type="string">sku</argument>
    </arguments>
</block>
```

## Reference a block

To update a block use the `<referenceBlock>` instruction.

Example: pass the image to the logo block.

```xml
<referenceBlock name="logo">
    <arguments>
        <argument name="logo_file" xsi:type="string">images/logo.png</argument>
    </arguments>
</referenceBlock>
```

## Set a block template

There are two ways to set the template for a block:

1. Using the template attribute
2. Using the `<argument>` instruction

Both approaches are demonstrated in the following examples of changing the template of the page title block.

### Example 1:
```xml
<referenceBlock name="page.main.title"
                template="Namespace_Module::new_template.phtml"
/>
```
### Example 2:
```xml
<referenceBlock name="page.main.title">
    <arguments>
        <argument name="template" xsi:type="string">
            Namespace_Module::new_template.phtml
        </argument>
    </arguments>
</referenceBlock>
```

In both example, the template is specified according to the following:

- `Namespace_Module` defines the module the template belongs to. For example, Magento_Catalog.
- `new_template.phtml`: the path to the template relatively to the templates directory. It might be `<module_dir>/view/<area>/templates` or `<theme_dir>/<Namespace_Module>/templates`.

Template values specified as attributes have higher priority during layout generation, than the ones specified using `<argument>`. It means, that if for a certain block, a template is set as attribute, it will override the value you specify in `<argument>` for the same block.

## Modify block arguments

To modify block arguments, use the `<referenceBlock>` instruction.

Example: change the value of the existing block argument and add a new argument.

### Initial block declaration:

```xml
<block class="Namespace_Module_Block_Type" name="block.example">
    <arguments>
        <argument name="label" xsi:type="string">Block Label</argument>
    </arguments>
</block>
```
### Extending layout:

```xml
<referenceBlock name="block.example">
    <arguments>
        <!-- Modified block argument -->
        <argument name="label" xsi:type="string">New Block Label</argument>

        <!--- Newly added block argument -->
        <argument name="custom_label" xsi:type="string">Custom Block Label</argument>
    </arguments>
</referenceBlock>
```

## Use block object methods to set block properties

Use the `<argument>` instruction for `<block>` or `<referenceBlock>` to set a CSS class and add an attribute for the product page using `<argument>`.

```xml
<referenceBlock name="page.main.title">
    <arguments>
        <argument name="css_class" xsi:type="string">product</argument>
        <argument name="add_base_attribute" xsi:type="string">itemprop="name"</argument>
    </arguments>
</referenceBlock>
```

## Rearrange elements

In layout files you can change the elements order on a page. This can be done using one of the following:

- `<move>` instruction: allows changing elements’ order and parent.
- `before` and `after` attributes of `<block>`: allows changing elements’ order within one parent.
Example of <move> usage: put the stock availability and SKU blocks next to the product price on a product page.

In the Magento Blank theme these elements are located as follows:
![](http://devdocs.magento.com/common/images/layout_image1.png)

Let’s place the stock availability and SKU blocks after product price block on a product page, and move the review block out of the product-info-price container. To do this, add the extending `catalog_product_view.xml` in the `app/design/frontend/vendor_name/theme_name/Magento_Catalog/layout/` directory:

```xml
<page layout="1column" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:View/Layout/etc/page_configuration.xsd">
    <body>
        <move element="product.info.stock.sku" destination="product.info.price" after="product.price.final"/>
        <move element="product.info.review" destination="product.info.main" before="product.info.price"/>
    </body>
</page>
```

This would make the product page look like following:
![](http://devdocs.magento.com/common/images/layout_image2.png)

## Remove elements

Elements are removed using the remove attribute for the `<referenceBlock>` and `<referenceContainer>`.

Example: remove the Compare Products sidebar block from all store pages.

This block is declared in `app/code/Magento/Catalog/view/frontend/layout/default.xml`:

```xml
<page xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:View/Layout/etc/page_configuration.xsd">
    <body>
        <referenceContainer name="sidebar.additional">
            <block class="Magento\Catalog\Block\Product\Compare\Sidebar" name="catalog.compare.sidebar" template="product/compare/sidebar.phtml"/>
        </referenceContainer>
    </body>
</page>
```

To remove the block, add the extending default.xml in your theme: `<theme_dir>/Magento_Catalog/layout/default.xml`

In this file, reference the element having added the remove attribute:

```xml
<page xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:View/Layout/etc/page_configuration.xsd">
    <body>
...
        <referenceBlock name="catalog.compare.sidebar" remove="true" />
...
    </body>
</page>
```

## Replace elements

To replace an element, remove it and add a new one.
